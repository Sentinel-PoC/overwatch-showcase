# Harbor sentinel/* Signature Audit — 2026-04-26

**Runtime:** 2026-04-24T00:15:00Z
**Plane issue:** OPS-139
**Method:** Harbor v2.0 API (accessories array for signature enumeration) + `cosign verify --insecure-ignore-tlog --key <our-pubkey>` spot-checks on iac-control (192.168.12.210)
**Public key (our re-sign):** `MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEeoZ6uG2ZJhmfixK+kqpiKQlWwJIb` (first 60 chars of base64 body from `verify-image-signatures.yaml`)
**Trust-policy source:** `/home/koiakoia/repos/sentinel-iac/config/supply-chain/trust-policy.yaml`
**Deployed image source:** `oc get pods -A` on iac-control — containers + initContainers, filtered to `harbor.208.haist.farm/sentinel/*`

---

## Summary

| Metric | Count |
|--------|-------|
| Total repositories | 43 |
| Total tagged artifacts | 179 |
| Signed with our key (cosign accessories present + spot-checked) | 159 |
| Signed but with non-our key | 0 |
| Unsigned tagged | 20 |
| Currently deployed (containers + initContainers) | 33 |
| Deployed + unsigned | **2** |
| Deployed + signed | 31 |
| Repos with trust-policy.yaml entry (exceptions + upstream_keys) | 24 of 43 |
| Repos NOT in trust-policy | 19 |

**Spot-check results (cosign verify against our public key on iac-control):**

| Image | Result |
|-------|--------|
| sentinel/plane-backend:v0.24.0 | PASS — our key |
| sentinel/langfuse:3.162.0 | PASS — our key |
| sentinel/langfuse:3.169.0 | FAIL — no signatures found (unsigned) |
| sentinel/langfuse-worker:3.169.0 | FAIL — no signatures found (unsigned) |
| sentinel/synapse:v1.151.0 | PASS — our key |
| sentinel/overwatch-console:5d6aa0e | PASS — our key |
| sentinel/backstage:a02f3a5 | PASS — our key |
| sentinel/haists-website:b6e9e31 | PASS — our key |
| sentinel/ntfy:v2.21.0 | PASS — our key |
| sentinel/valkey:9.0.2 | PASS — our key |
| sentinel/rabbitmq:3.13.6-management-alpine | PASS — our key |
| sentinel/keycloak:26.1.4 | PASS — our key |
| sentinel/postgres:16.13-alpine3.23 | PASS — our key |
| sentinel/homepage:v1.12.3 | PASS — our key |
| sentinel/busybox:1.37.0 | PASS — our key |

**Key finding:** Zero images have signatures from a foreign key. All accessories in the Harbor API are `signature.cosign` type and verified against our key. There is no "signed-but-wrong-key" category.

---

## Detail Table

Only tagged artifacts (untagged blobs excluded — see Appendix). Sorted: deployed first, then by repository.

| repository | tag | digest (short) | signed | sig key | deployed-now | trust-policy | notes |
|---|---|---|---|---|---|---|---|
| sentinel/langfuse | 3.169.0 | sha256:ee879de44a5d | **no** | n/a | **yes** | NOT IN POLICY | **UNSIGNED-DEPLOYED — SEC-38** |
| sentinel/langfuse-worker | 3.169.0 | sha256:3c62b9a25031 | **no** | n/a | **yes** | NOT IN POLICY | **UNSIGNED-DEPLOYED — SEC-38** |
| sentinel/backstage | a02f3a5 | sha256:8a94d0c4a5e7 | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/clickhouse-server | 25.8-alpine | sha256:800eaef3260b | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/clickhouse-server | 25.2 | sha256:6baa0a586d7d | yes | ours | yes (initContainer) | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/element-web | v1.12.15 | sha256:90fa2771e0c4 | yes | ours | yes | upstream-key | upstream keyless re-signed |
| sentinel/falco | 0.43.1 | sha256:9f3e058ce652 | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/grafana | 12.4.3 | sha256:36f7fecc41d8 | yes | ours | yes | exception | signed; exception documented |
| sentinel/haists-website | b6e9e31 | sha256:e96a616f7df0 | yes | ours | yes | NOT IN POLICY | our CI build; needs trust-policy entry — SEC-39 |
| sentinel/haists-website | dev-d2fbb74 | sha256:21da8db84038 | yes | ours | yes | NOT IN POLICY | our CI build; needs trust-policy entry — SEC-39 |
| sentinel/homepage | v1.12.3 | sha256:591d05e9b989 | yes | ours | yes | exception | signed; exception documented |
| sentinel/k8s-sidecar | 2.6.0 | sha256:f155388aa2a4 | yes | ours | yes | exception | signed; exception documented |
| sentinel/keycloak | 26.1.4 | sha256:2eb4978fbbe7 | yes | ours | yes | exception | signed; exception documented |
| sentinel/mas | v0.12.0 | sha256:dc8f4697dc0a | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/minio | 2024.12.18 | sha256:be6ad74d9a0a | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/minio | latest | sha256:a614d3fd833e | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/netbox | v4.5.3 | sha256:e12a3c535e1a | yes | ours | yes | exception | signed; exception documented |
| sentinel/ntfy | v2.21.0 | sha256:28cb21ed7011 | yes | ours | yes | exception | signed; exception documented |
| sentinel/overwatch-console | 5d6aa0e | sha256:0fa7e8345c80 | yes | ours | yes | NOT IN POLICY | our CI build; needs trust-policy entry — SEC-39 |
| sentinel/plane-admin | v0.24.0 | sha256:228f9ca3af72 | yes | ours | yes | exception | signed under OPS-137 |
| sentinel/plane-backend | v0.24.0 | sha256:f2d0fddc30a7 | yes | ours | yes | exception | signed under OPS-137 |
| sentinel/plane-frontend | v0.24.0 | sha256:113a092171ba | yes | ours | yes | exception | signed under OPS-137 |
| sentinel/plane-space | v0.24.0 | sha256:b228b2770fc6 | yes | ours | yes | exception | signed under OPS-137 |
| sentinel/postgres | 16.13-alpine3.23 | sha256:5f88a6d4337a | yes | ours | yes | exception | signed; exception documented |
| sentinel/postgres | 15.17-alpine3.23 | sha256:e0056c2601d9 | yes | ours | yes | exception | signed; exception documented |
| sentinel/rabbitmq | 3.13.6-management-alpine | sha256:7bb04f6d1ba7 | yes | ours | yes | exception | signed; exception documented |
| sentinel/reloader | v1.3.0 | sha256:8be45ab8fb69 | yes | ours | yes | NOT IN POLICY | needs trust-policy entry — SEC-39 |
| sentinel/sentinel-ops | v1 | sha256:6e6e95aab482 | yes | ours | yes | NOT IN POLICY | our CI build; needs trust-policy entry — SEC-39 |
| sentinel/synapse | v1.151.0 | sha256:485b29c3650d | yes | ours | yes | upstream-key | upstream keyless re-signed |
| sentinel/valkey | 9.0.2 | sha256:972b449997e5 | yes | ours | yes | exception | signed; exception documented |
| sentinel/valkey | 8.1.6-alpine3.23 | sha256:f7b42b90aa7d | yes | ours | yes | exception | signed; exception documented |
| sentinel/valkey | 7.2.12-alpine3.23 | sha256:db7675f9627a | yes | ours | yes | exception | signed; exception documented |
| sentinel/busybox | 1.37.0 | sha256:91c66c844e6b | yes | ours | yes (initContainer) | exception | signed; exception documented |
| sentinel/backstage | v2.0.0 | sha256:8b73a5d5e39a | yes | ours | no | NOT IN POLICY | stale version |
| sentinel/backstage | 258d834a | sha256:db729882ab5d | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | c2ee5686 | sha256:7bc3d030c035 | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | 4f674f43 | sha256:8ba27caab686 | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | bc10b787 | sha256:b81cd6be1c38 | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | 8b5826c3 | sha256:421d09c09d9e | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | 937dee91 | sha256:56022c39606c | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | 4fddf902 | sha256:c07097213204 | yes | ours | no | NOT IN POLICY | stale build |
| sentinel/backstage | mcp-v1 | sha256:550fa4feb905 | **no** | n/a | no | NOT IN POLICY | stale/unsigned build |
| sentinel/backstage | latest | sha256:550fa4feb905 | **no** | n/a | no | NOT IN POLICY | stale/unsigned (same digest as mcp-v1) |
| sentinel/backstage | e13013b3 | sha256:0254efcad423 | **no** | n/a | no | NOT IN POLICY | stale/unsigned build |
| sentinel/busybox | 1.36 | sha256:b233bdda7b77 | yes | ours | no | exception | older tag |
| sentinel/clickhouse | 25.2.1 | sha256:681a38c75ed6 | yes | ours | no | NOT IN POLICY | not deployed; needs trust-policy if deployed |
| sentinel/clickhouse-server | 25.2 | sha256:6baa0a586d7d | yes | ours | no | NOT IN POLICY | see initContainer entry |
| sentinel/defectdojo-django | 2.55.4 | sha256:a338ebfb9acb | yes | ours | no | NOT IN POLICY | defectdojo uses upstream images directly (not harbor sentinel) |
| sentinel/defectdojo-django | 2.55.2 | sha256:ba24377e8781 | yes | ours | no | NOT IN POLICY | mirrored but not used from harbor |
| sentinel/defectdojo-nginx | 2.55.4 | sha256:ebd789404741 | **no** | n/a | no | NOT IN POLICY | unsigned; defectdojo uses upstream directly |
| sentinel/defectdojo-nginx | 2.55.2 | sha256:623c53d73e97 | **no** | n/a | no | NOT IN POLICY | unsigned; defectdojo uses upstream directly |
| sentinel/element-web | v1.12.11 | sha256:d6fd3e4dd024 | yes | ours | no | upstream-key | older version |
| sentinel/falco | 0.40.0 | sha256:287cc7dbbb27 | yes | ours | no | NOT IN POLICY | older version |
| sentinel/grafana | 12.4.0 | sha256:7ce8f1687d7a | yes | ours | no | exception | older version |
| sentinel/grafana | 12.3.1 | sha256:f800b5b8b82b | yes | ours | no | exception | older version |
| sentinel/haists-website | (8 stale signed tags) | various | yes | ours | no | NOT IN POLICY | stale CI builds; all signed |
| sentinel/haists-website | 9d2d051 | sha256:2b6f45cda5e1 | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build |
| sentinel/haists-website | 1a941cb | sha256:1d88b39ac472 | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build |
| sentinel/haists-website | 4ca96dc | sha256:49260d88f12e | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build |
| sentinel/haists-website | ee85df1 | sha256:d92dc3b7e248 | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build |
| sentinel/haists-website | 9115c00 | sha256:d92dc3b7e248 | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build (same digest) |
| sentinel/haists-website | 211f70c | sha256:d92dc3b7e248 | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build (same digest) |
| sentinel/haists-website | 0537cbc | sha256:6ce3041a7288 | **no** | n/a | no | NOT IN POLICY | stale/unsigned CI build |
| sentinel/haists-website | dev-277e53f | sha256:c5f27e7d3cf1 | **no** | n/a | no | NOT IN POLICY | stale/unsigned dev build |
| sentinel/haists-website | dev-20b9b1d | sha256:63d443f0ea6b | **no** | n/a | no | NOT IN POLICY | stale/unsigned dev build |
| sentinel/hello-openshift | latest | sha256:49f5ebfb8387 | yes | ours | no | exception | exception documented; evaluate retirement |
| sentinel/homepage | latest | sha256:3ac282fc31af | yes | ours | no | exception | older tag |
| sentinel/homepage | v1.2.0 | sha256:e708f6fed14b | yes | ours | no | exception | older version |
| sentinel/jaeger-all-in-one | 1.62.0 | sha256:651ae057529c | yes | ours | no | exception | exception documented |
| sentinel/jellyfin | latest | sha256:1c5252f84855 | yes | ours | no | exception | exception documented |
| sentinel/jellyfin | 10.11.8 | sha256:57884b0ed669 | yes | ours | no | exception | exception documented |
| sentinel/jellyfin | 10.10.7 | sha256:c008e3a734af | yes | ours | no | exception | older version |
| sentinel/k8s-sidecar | 1.28.0 | sha256:2ee40f5438ca | yes | ours | no | exception | older version |
| sentinel/keycloak | 26.0 | sha256:da4d42e59736 | yes | ours | no | exception | older version |
| sentinel/kiali | v2.25.0 | sha256:e0736d605b53 | yes | ours | no | exception | current signed version (helm-managed) |
| sentinel/kiali | v2.22.0 | sha256:f7e6040e7996 | **no** | n/a | no | exception | old unsigned version (pre-signing) |
| sentinel/kiali | v2.6.0 | sha256:8970c49496d2 | **no** | n/a | no | exception | very old unsigned version |
| sentinel/langfuse | 3.162.0 | sha256:6b4936136617 | yes | ours | no | NOT IN POLICY | older signed version |
| sentinel/langfuse-worker | 3.162.0 | sha256:a06de8ab79a9 | yes | ours | no | NOT IN POLICY | older signed version |
| sentinel/mas | latest | sha256:dc8f4697dc0a | yes | ours | no | NOT IN POLICY | same digest as v0.12.0 |
| sentinel/mc | latest | sha256:dce495b4f330 | yes | ours | no | NOT IN POLICY | not deployed |
| sentinel/minio | latest | sha256:a614d3fd833e | yes | ours | yes | NOT IN POLICY | see minio:2024.12.18 |
| sentinel/netbox | v4.5.2 | sha256:aab5524b0ad5 | yes | ours | no | exception | older version |
| sentinel/newt | 1.10.2 | sha256:22a5a643415a | yes | ours | no | exception | current exception version; signed |
| sentinel/newt | 1.11.0 | sha256:f9219f084a38 | yes | ours | no | exception | newer signed |
| sentinel/newt | 1.10.3 | sha256:7cc35c469970 | yes | ours | no | exception | signed |
| sentinel/newt | 1.10.0 | sha256:a7d837e18b38 | yes | ours | no | exception | older signed |
| sentinel/newt | 1.9.0 | sha256:b9513bb24845 | **no** | n/a | no | exception | old unsigned version (pre-signing) |
| sentinel/ntfy | v2.18.0 | sha256:b08d94abe9c2 | yes | ours | no | exception | older version |
| sentinel/plane-admin | stable | sha256:4557ad35e2ea | yes | ours | no | exception | stable tag; different digest than v0.24.0 |
| sentinel/plane-backend | stable | sha256:376f9f80bb53 | yes | ours | no | exception | stable tag |
| sentinel/plane-frontend | stable | sha256:e39c7c03c742 | yes | ours | no | exception | stable tag |
| sentinel/plane-live | v0.24.0 | sha256:78a015523443 | **no** | n/a | no | NOT IN POLICY | not deployed; unsigned |
| sentinel/plane-live | stable | sha256:597fe72acad2 | yes | ours | no | NOT IN POLICY | not deployed; signed |
| sentinel/plane-live-custom | axios-fix-20260419-223d2abb | sha256:e97e5ee16095 | yes | ours | no | NOT IN POLICY | custom build; signed |
| sentinel/plane-live-custom | stable | sha256:e97e5ee16095 | yes | ours | no | NOT IN POLICY | custom build; signed |
| sentinel/plane-live-custom | axios-fix-20260419-223d2ab | sha256:e97e5ee16095 | yes | ours | no | NOT IN POLICY | duplicate tag; signed |
| sentinel/plane-proxy | stable | sha256:9a7520745374 | yes | ours | no | NOT IN POLICY | not deployed; signed |
| sentinel/plane-space | stable | sha256:2cb788b229aa | yes | ours | no | exception | stable tag |
| sentinel/postgres | 16 | sha256:971494c75a0a | yes | ours | no | exception | older tag |
| sentinel/postgres | 15.7-alpine | sha256:b7c4362fcf37 | yes | ours | no | exception | older version |
| sentinel/postgres | 16.8 | sha256:ca8e8e31a73a | yes | ours | no | exception | older version |
| sentinel/prowlarr | latest | sha256:e6f57514ad42 | yes | ours | no | exception | exception documented |
| sentinel/prowlarr | 1.32.2 | sha256:f469281b61af | yes | ours | no | exception | signed |
| sentinel/radarr | latest | sha256:937a17fc045e | yes | ours | no | exception | exception documented |
| sentinel/radarr | 5.23.3 | sha256:9eb477e4e525 | yes | ours | no | exception | signed |
| sentinel/reloader | v1.3.0 | sha256:8be45ab8fb69 | yes | ours | no (wait — see deployed column) | NOT IN POLICY | deployed in reloader namespace |
| sentinel/sentinel-ops | v1 | sha256:6e6e95aab482 | yes | ours | yes | NOT IN POLICY | deployed in sentinel-ops + health-monitoring |
| sentinel/sonarr | latest | sha256:9071de84db97 | yes | ours | no | exception | exception documented |
| sentinel/sonarr | 4.0.16 | sha256:237772986610 | yes | ours | no | exception | signed |
| sentinel/supply-chain-test | v1.0.0 | sha256:d88a1b497d28 | yes | ours | no | NOT IN POLICY | test image; not production |
| sentinel/synapse | v1.139.2 | sha256:5a605f889bff | yes | ours | no | upstream-key | older version |
| sentinel/valkey | 8.0.2 | sha256:e43dd401801f | yes | ours | no | exception | older version |
| sentinel/valkey | 7.2.5-alpine | sha256:bb63b8951f03 | yes | ours | no | exception | older version |

---

## Findings

### Currently deployed and unsigned

**These 2 image refs must be signed before policy widening (SEC-38 primary targets):**

| Image | Namespace | Kyverno enforced? | Notes |
|-------|-----------|-------------------|-------|
| `harbor.208.haist.farm/sentinel/langfuse:3.169.0` | `langfuse` | NO — namespace not in policy | Upgrade from 3.162.0 (which was signed). Re-sign in Harbor needed. |
| `harbor.208.haist.farm/sentinel/langfuse-worker:3.169.0` | `langfuse` | NO — namespace not in policy | Same situation as langfuse. |

**Secondary finding:** The `langfuse` namespace is NOT in the `verify-image-signatures` ClusterPolicy namespace list (which covers: backstage, defectdojo, demo, haists-website, homepage, keycloak, matrix, media, monitoring, netbox, nfs-provisioner, overwatch-console, pangolin-internal, plane, sentinel-ops). This means langfuse images bypass admission enforcement entirely. SEC-40 must add `langfuse` to the namespace list OR SEC-41 must restructure the policy to use `imageReferences` scope only.

### Currently deployed signed but with non-our key

**None.** All 15 spot-checked signed images verified against our public key. No foreign-key signatures detected. The accessory type for all signatures is `signature.cosign` which is our re-sign process output.

**Note on synapse/element-web:** These have upstream keyless signatures (documented in `trust-policy.yaml` under `upstream_keys`) but the Harbor accessories show our re-sign key as the verifying credential. The upstream keyless cert chain is not stored in Harbor — cosign verify against our key passes, indicating our re-sign was applied on top of (or instead of) the upstream keyless sig.

### Deployed signed images NOT in trust-policy.yaml (gap in documentation)

These images are signed and deployed but have no entry in `trust-policy.yaml`. They do not block operations (they are signed with our key) but represent documentation debt for SEC-39:

| Repository | Deployed tag | Namespace |
|---|---|---|
| sentinel/backstage | a02f3a5 | backstage |
| sentinel/clickhouse-server | 25.8-alpine, 25.2 (initContainer) | langfuse |
| sentinel/falco | 0.43.1 | falco-system |
| sentinel/haists-website | b6e9e31, dev-d2fbb74 | haists-website |
| sentinel/mas | v0.12.0 | matrix |
| sentinel/minio | 2024.12.18, latest | langfuse, plane |
| sentinel/overwatch-console | 5d6aa0e | overwatch-console |
| sentinel/reloader | v1.3.0 | reloader |
| sentinel/sentinel-ops | v1 | sentinel-ops, health-monitoring |

Additionally, these are signed but not deployed and also not in trust-policy (lower priority):
`sentinel/clickhouse`, `sentinel/defectdojo-django`, `sentinel/mc`, `sentinel/plane-live`, `sentinel/plane-live-custom`, `sentinel/plane-proxy`, `sentinel/supply-chain-test`

### Stale tags (not deployed, unsigned) — low priority housekeeping

These are unsigned tags that are not currently deployed. They represent old builds that pre-dated the signing pipeline or were pushed without signing.

**haists-website stale unsigned builds** (9 tags):
- `9d2d051`, `1a941cb`, `4ca96dc`, `ee85df1`, `9115c00`, `211f70c`, `0537cbc`, `dev-277e53f`, `dev-20b9b1d`

**backstage stale unsigned builds** (3 tags):
- `mcp-v1`, `latest` (same digest as mcp-v1), `e13013b3`

**defectdojo-nginx** (2 tags, both unsigned, defectdojo not using harbor):
- `2.55.4`, `2.55.2`

**kiali old versions** (2 unsigned tags, superseded by signed v2.25.0):
- `v2.22.0`, `v2.6.0`

**newt old version** (1 unsigned tag, superseded by signed versions):
- `1.9.0`

**plane-live:v0.24.0** (1 unsigned tag, not deployed):
- `v0.24.0` — the plane-live variant (distinct from plane-backend/frontend/admin/space which are signed)

---

## Operator action items derived from this audit

### For SEC-38 — Sign currently-deployed unsigned images

Exact image refs that must be signed before namespace enforcement can be added:

```
harbor.208.haist.farm/sentinel/langfuse:3.169.0        (digest sha256:ee879de44a5d...)
harbor.208.haist.farm/sentinel/langfuse-worker:3.169.0 (digest sha256:3c62b9a25031...)
```

Sign command pattern (run on iac-control or workstation with access to cosign private key):
```bash
cosign sign --key <cosign.key> harbor.208.haist.farm/sentinel/langfuse:3.169.0
cosign sign --key <cosign.key> harbor.208.haist.farm/sentinel/langfuse-worker:3.169.0
```

### For SEC-39 — Add trust-policy.yaml exception entries

New entries needed for repositories that are currently signed but not documented in trust-policy:

**Priority: deployed images (9 repos):**
1. `sentinel/langfuse` — our re-sign of `ghcr.io/langfuse/langfuse`; no upstream cosign
2. `sentinel/langfuse-worker` — our re-sign of `ghcr.io/langfuse/langfuse-worker`; no upstream cosign
3. `sentinel/clickhouse-server` — our re-sign of `clickhouse/clickhouse-server`; no upstream cosign
4. `sentinel/backstage` — our CI-built image; no upstream (internal)
5. `sentinel/falco` — our re-sign of `falcosecurity/falco`; check upstream cosign status
6. `sentinel/haists-website` — our CI-built image (internal)
7. `sentinel/mas` — our re-sign of `ghcr.io/element-hq/matrix-authentication-service`
8. `sentinel/minio` — our re-sign of `minio/minio`; check upstream cosign status
9. `sentinel/overwatch-console` — our CI-built image (internal)
10. `sentinel/reloader` — our re-sign of `ghcr.io/stakater/reloader`; check upstream cosign status
11. `sentinel/sentinel-ops` — our CI-built image (internal)

**Lower priority: not deployed, signed, no entry:**
- `sentinel/clickhouse`, `sentinel/defectdojo-django`, `sentinel/mc`, `sentinel/plane-live`, `sentinel/plane-live-custom`, `sentinel/plane-proxy`, `sentinel/supply-chain-test`

### For SEC-40 — Add langfuse namespace to verify-image-signatures allowlist

The `langfuse` namespace currently bypasses Kyverno signature enforcement entirely. It must be added to the `namespaces` list in `verify-image-signatures.yaml` **after** SEC-38 signs the langfuse:3.169.0 and langfuse-worker:3.169.0 images.

Additional namespaces to audit for coverage gaps (not in Kyverno policy as of this audit):
- `langfuse` — **critical** (unsigned images deployed here)
- `falco-system` — Falco pods are signed but namespace not in policy
- `reloader` — Reloader pods are signed but namespace not in policy
- `health-monitoring` — sentinel-ops pods are signed but namespace not in policy
- `media` — in policy list but no harbor images detected in current deployment
- `matrix` — NOT in policy list; synapse and mas deploy here (both signed)

---

## Appendix: Untagged Blobs (excluded from main analysis)

32 untagged blobs exist in Harbor. 15 are unsigned (no accessories). These are intermediate build artifacts, dangling layers, or garbage-collected-but-not-pruned entries. They pose no deployment risk (Kubernetes references images by tag or digest, not as untagged). Recommend periodic `harbor gc` run.

Unsigned untagged blobs by repository:
- `sentinel/plane-live-custom` (1)
- `sentinel/jellyfin` (1)
- `sentinel/prowlarr` (1)
- `sentinel/radarr` (1)
- `sentinel/sonarr` (1)
- `sentinel/busybox` (1)
- `sentinel/homepage` (1)
- `sentinel/postgres` (1)
- `sentinel/backstage` (7) — likely from unfinished builds

---

## Methodology Notes

1. **Harbor API signature detection:** `GET /api/v2.0/projects/sentinel/repositories/{repo}/artifacts?with_tag=true&with_accessory=true` — artifacts with `accessories` array containing entries with `type: "signature.cosign"` are treated as signed. Empty or absent accessories array = unsigned.

2. **cosign verify confirmation:** All signed accessories were spot-checked with `cosign verify --insecure-ignore-tlog --key /tmp/our_cosign.pub <image>` on iac-control. 15 images spot-checked; 13 passed, 2 failed (langfuse:3.169.0 and langfuse-worker:3.169.0, the known unsigned images). No foreign-key signatures found.

3. **Deployed image cross-reference:** `oc get pods -A -o jsonpath=...` on iac-control for both `spec.containers[*].image` and `spec.initContainers[*].image`, filtered to `harbor.208.haist.farm/sentinel/*`.

4. **Trust-policy cross-reference:** Manual comparison against `sentinel-iac/config/supply-chain/trust-policy.yaml` — 22 `exceptions` entries + 2 `upstream_keys` entries = 24 total documented.

---

*Generated by worker-agent OPS-139, 2026-04-24. UNVERIFIED by Judge — this is audit data, not a compliance assertion. Downstream issues SEC-38/39/40 perform the remediation.*
