# Kyverno + Istio Compliance Check Rewrite Proposal — 2026-04-25

**Trigger:** operator question "is kyverno istio all the hardening working or is it's fake like before". Live cluster probe done from iac-control 2026-04-26T00:35Z confirmed: **the platform's enforcement is real, but the compliance script's checks are installation-level not enforcement-level.**

**Source script:** `/home/koiakoia/repos/sentinel-iac/scripts/nist-compliance-check.sh` (READ-ONLY for all agents). Proposed replacement bash for L547-L558 (`check_istio_mtls`) and L1719-L1730 (`check_kyverno_policies`).

**Method:** SSH'd to iac-control with Vault-signed JIT cert, ran `oc` probes against the live cluster, observed actual admission rejections in Kyverno logs and per-namespace `PeerAuthentication` state.

---

## Live state observed (2026-04-26T00:35Z)

### Kyverno

- 8 ClusterPolicies, all `status.conditions[Ready].status == True`
- All 9 Kyverno pods Running, ready, low restart counts
- 5 of 8 policies `validationFailureAction: Enforce`; 3 are `Audit` (`inject-harbor-pull-secret` is mutating so Audit is fine; `require-labels` and `verify-attestations` are validation-Audit and worth a look)
- `kyverno-resource-validating-webhook-cfg` correctly splits Fail-mode and Ignore-mode policies into separate webhook endpoints (this is by-design)
- Functional probe: applying a privileged + no-limits Pod to `media` namespace as cluster-admin got rejected by `validate.kyverno.svc-fail` with both `deny-privileged` and `require-limits` rules firing — **enforcement is real**
- Production admission logs show `verify-image-signatures` actively blocking `harbor.208.haist.farm/sentinel/plane-backend:v0.24.0` (unsigned image) — **policy is doing its job in prod right now**

### Istio

- `istio-system/default` PA mode = STRICT (the only object the current script reads)
- 9 namespaces have local PA overrides; one has `mode: PERMISSIVE`:
  - `harbor` namespace — full mTLS bypass for in-mesh traffic to harbor workloads
- Other PA overrides are STRICT (despite some named `*-permissive` — likely renamed in place)
- 9 namespaces have AuthorizationPolicies; `istio-ingress` and `overwatch-console` have sidecars but no AuthorizationPolicy (open within mesh)
- `istiod` running 2 replicas, 2-3 day pod ages

---

## Why the current checks miss everything

### `check_istio_mtls` (L547-L558)

```bash
mode=$(oc_cmd get peerauthentication -n istio-system default -o jsonpath='{.spec.mtls.mode}')
[ "$mode" = "STRICT" ] && pass || warn/fail
```

Reads exactly one PA object. Misses:
- All per-namespace PAs (the harbor PERMISSIVE override is invisible)
- AuthorizationPolicy presence
- Sidecar coverage gaps
- istiod readiness

If someone tomorrow drops `mode: DISABLE` PAs in 50 namespaces, the headline still PASSes.

### `check_kyverno_policies` (L1719-L1730)

```bash
count=$(oc_cmd get clusterpolicy --no-headers | wc -l)
[ "$count" -ge 3 ] && pass
```

Counts ClusterPolicies. Misses:
- Whether any policy is `Ready=True` (would have shown the OPS-108 background-controller CrashLoop as "all installed" while broken)
- Whether `validationFailureAction` is `Enforce` vs `Audit`
- Whether the ValidatingWebhookConfiguration is registered with the kube-apiserver
- Whether the webhooks have `failurePolicy: Fail` (the OPS-88 fail-open class of bug, but per-policy)
- Whether a known-bad resource is actually blocked (functional probe)

If tomorrow someone flips every policy to `Audit` and disables enforcement entirely, this check still PASSes.

---

## Proposed replacement bash

### `check_istio_mtls` rewrite — runtime + override-aware

```bash
check_istio_mtls() {
    log "Checking Istio mTLS [AC-4]..."

    # 1. mesh-wide PA must exist and be STRICT
    local mesh_mode
    mesh_mode=$(oc_cmd get peerauthentication -n istio-system default -o jsonpath='{.spec.mtls.mode}' 2>/dev/null || echo "")
    if [ "$mesh_mode" != "STRICT" ]; then
        if [ "$mesh_mode" = "PERMISSIVE" ]; then
            warn "AC-4" "istio_mtls" "Istio mesh-wide PeerAuthentication is PERMISSIVE not STRICT"
        else
            fail "AC-4" "istio_mtls" "Istio mesh-wide PeerAuthentication not STRICT (mode=${mesh_mode:-missing})"
        fi
        return
    fi

    # 2. enumerate per-namespace PAs; flag any non-STRICT (those override mesh-wide)
    local override_report
    override_report=$(oc_cmd get peerauthentication -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}={.spec.mtls.mode}{"\n"}{end}' 2>/dev/null \
        | grep -v "=STRICT$" \
        | grep -v "^istio-system/default=$" \
        | grep -v "^$")
    if [ -n "$override_report" ]; then
        local count
        count=$(echo "$override_report" | wc -l)
        warn "AC-4" "istio_mtls" "Mesh STRICT but ${count} per-namespace PAs override (non-STRICT): $(echo "$override_report" | tr '\n' ',' | sed 's/,$//')"
        return
    fi

    # 3. istiod must be Ready (otherwise mTLS is enforced by stale Envoys until they die)
    local istiod_ready
    istiod_ready=$(oc_cmd -n istio-system get deployment istiod -o jsonpath='{.status.readyReplicas}' 2>/dev/null || echo 0)
    if [ "${istiod_ready:-0}" -lt 1 ]; then
        warn "AC-4" "istio_mtls" "Mesh PAs all STRICT but istiod has zero Ready replicas"
        return
    fi

    pass "AC-4" "istio_mtls" "Mesh STRICT; no per-namespace overrides; istiod ${istiod_ready} ready"
}
```

### `check_kyverno_policies` rewrite — ready + enforce + functional

```bash
check_kyverno_policies() {
    log "Checking Kyverno policies [SI-7(1)]..."

    # 1. policies must exist
    local total
    total=$(oc_cmd get clusterpolicy --no-headers 2>/dev/null | wc -l)
    if [ "$total" = 0 ]; then
        fail "SI-7(1)" "kyverno_policies" "No Kyverno ClusterPolicies found"
        return
    fi

    # 2. all policies Ready
    local not_ready
    not_ready=$(oc_cmd get clusterpolicy -o jsonpath='{range .items[*]}{.metadata.name}={.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}' 2>/dev/null \
        | grep -v "=True$" | grep -v "^$" | wc -l)
    if [ "$not_ready" -gt 0 ]; then
        local list
        list=$(oc_cmd get clusterpolicy -o jsonpath='{range .items[*]}{.metadata.name}={.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}' 2>/dev/null \
            | grep -v "=True$" | grep -v "^$" | head -5 | tr '\n' ',' | sed 's/,$//')
        fail "SI-7(1)" "kyverno_policies" "${not_ready}/${total} policies not Ready: ${list}"
        return
    fi

    # 3. count Enforce vs Audit; require at least one Enforce
    local enforce_count audit_count
    enforce_count=$(oc_cmd get clusterpolicy -o jsonpath='{range .items[*]}{.spec.validationFailureAction}{"\n"}{end}' 2>/dev/null | grep -ic "^Enforce$" || true)
    audit_count=$(oc_cmd get clusterpolicy -o jsonpath='{range .items[*]}{.spec.validationFailureAction}{"\n"}{end}' 2>/dev/null | grep -ic "^Audit$" || true)
    if [ "${enforce_count:-0}" -lt 1 ]; then
        fail "SI-7(1)" "kyverno_policies" "${total} policies installed but ZERO Enforce-mode (all Audit-only — no admission blocking)"
        return
    fi

    # 4. ValidatingWebhookConfiguration must be registered
    local wh_count
    wh_count=$(oc_cmd get validatingwebhookconfigurations -o jsonpath='{range .items[?(@.metadata.name=="kyverno-resource-validating-webhook-cfg")]}{.metadata.name}{"\n"}{end}' 2>/dev/null | wc -l)
    if [ "$wh_count" = 0 ]; then
        fail "SI-7(1)" "kyverno_policies" "${total} policies installed but kyverno-resource-validating-webhook-cfg missing — admission disconnected"
        return
    fi

    # 5. functional probe: Pod with privileged + no limits in ns covered by Enforce policy
    #    If admission accepts it, enforcement is broken even though config looks right.
    local probe_ns="media"  # any ns in the Enforce policies' namespace list
    local probe_yaml="apiVersion: v1
kind: Pod
metadata:
  name: nist-compliance-probe
  namespace: ${probe_ns}
spec:
  containers:
  - name: c
    image: nginx:latest
    securityContext:
      privileged: true
      runAsUser: 0"
    local probe_out
    probe_out=$(echo "$probe_yaml" | oc_cmd apply --dry-run=server -f - 2>&1 || true)
    if echo "$probe_out" | grep -q "denied the request"; then
        pass "SI-7(1)" "kyverno_policies" "${total} policies all Ready; ${enforce_count} Enforce / ${audit_count} Audit; webhook registered; functional probe DENIED privileged Pod"
    else
        fail "SI-7(1)" "kyverno_policies" "${total} policies all Ready and webhook registered, but FUNCTIONAL PROBE BYPASSED (privileged Pod accepted in ${probe_ns})"
    fi
}
```

---

## Strength registry implications

If the existing entries (`AC-4 istio_mtls` and `SI-7(1) kyverno_policies`) are kept as-is, recommended tags in `check-strength.yaml` (operator-only file):

```yaml
- control: AC-4
  check_id: istio_mtls
  strength: misleading
  reason: "Reads only istio-system/default PeerAuthentication. Per-namespace PERMISSIVE overrides invisible (e.g., harbor namespace today). Claims mesh STRICT when one namespace bypasses mTLS."

- control: SI-7(1)
  check_id: kyverno_policies
  strength: trivial
  reason: "Counts ClusterPolicies. Default Kyverno install ships with ≥3 sample policies. Says nothing about Ready status, Enforce vs Audit mode, webhook registration, or functional admission. Identical structural pattern to the trivial AC-3 RBAC count check."
```

If the rewrites land, both move to `strong`.

---

## Operator (Jim) action

This file is the proposal. Apply path:

1. **Optional first:** add the registry tags above to `check-strength.yaml` so the current strength of these checks is honestly documented before any rewrite.
2. **Rewrite:** in `sentinel-iac/scripts/nist-compliance-check.sh`, replace L547-L558 and L1719-L1730 with the bash above. Branch suggestion: `OPS-NNN-kyverno-istio-runtime-checks`.
3. **Run on iac-control:** `~/sentinel-iac/scripts/nist-compliance-check.sh` to confirm both checks still PASS with rewritten logic. Today's run should return PASS for Kyverno (functional probe denies privileged Pod) and **WARN for Istio** (the harbor PERMISSIVE override will surface — that's the point).
4. **If harbor's PERMISSIVE is intentional** (kubelet plaintext image pulls), document the exception in the SSP (compliance-scribe issue) and either:
   - Tighten to STRICT with `portLevelMtls` carving out the registry port for plaintext, OR
   - Accept the exception and code the check to permit the documented namespace allowlist

---

## Files of interest (live state, reproducible)

```
ssh -i ~/.ssh/claude_jit ubuntu@192.168.12.210
oc get clusterpolicy -o jsonpath='{range .items[*]}{.metadata.name}\t{.spec.validationFailureAction}\t{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'
oc get peerauthentication -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}={.spec.mtls.mode}{"\n"}{end}'
```

The JIT cert is signed via Vault `ssh/sign/claude-automation` role and expires 6h after issuance.
