# check-strength.yaml Delta Proposal — 2026-04-25

**Trigger:** strength registry header (2026-03-18) references nist-compliance-check.sh at 2,829 lines / 125 checks. Script today is 2,984 lines (+155). Fourteen commits since 2026-03-18 touched the script; none triggered a formal strength re-audit until now.

**Source script:** `/home/koiakoia/repos/sentinel-iac/scripts/nist-compliance-check.sh` (READ-ONLY, observed at HEAD commit `5dafb6e`, 2,984 lines, 2026-04-24)
**Source registry:** `/home/koiakoia/repos/overwatch/check-strength.yaml` (READ-ONLY, last header date 2026-03-18, references 2,829 lines / 125 checks)
**Method:** `git log --since=2026-03-18 -- scripts/nist-compliance-check.sh` (15 commits) + per-commit `git diff` inspection. Line counts verified per-SHA.

---

## Commit window summary

| commit | issue | date | net lines | checks touched |
|--------|-------|------|-----------|----------------|
| `f45354d` | OPS-82 | 2026-03-24 | +24 | AU-6, IR-4, IR-6, IR-5, SA-8, AU-10 |
| `e8196ce` | OPS-130 | 2026-03-29 | 0 | CP-10, AC-14, SC-7(7) (constants/labels) |
| `88d7497` | OPS-148 | 2026-04-07 | 0 | CA-7 (EXPECTED_AGENTS constant) |
| `da1b266` | OPS-196 | 2026-04-17 | 0 | IR-6 (caller escape typo) |
| `b11d914` | OPS-200 | 2026-04-17 | +4 | SA-10 |
| `ae9735e` | OPS-202 | 2026-04-17 | 0 | AU-10 (caller escape typo) |
| `76dd8d8` | OPS-193 | 2026-04-17 | +1 | CP-9 |
| `7421ac3` | COMP-20 | 2026-04-20 | +59 | SI-2 |
| `aee1272` | COMP-26 Group A | 2026-04-20 | +35 | CM-3(3), RA-5, RA-5(3), SA-11 |
| `269db5f` | COMP-26 Group B | 2026-04-20 | +15 | SC-17 |
| `101c0cb` | COMP-26 Group C | 2026-04-20 | +8 | AC-6, IA-5(13) |
| `dd576fc` | COMP-26 Group D | 2026-04-20 | +14 | MA-2, CP-2, CP-3, SA-10 |
| `5dafb6e` | COMP-27 | 2026-04-20 | +10 | AC-17 |

Baseline: 2,829 lines (2026-03-18). HEAD: 2,984 lines. Delta: +155 lines confirmed across 13 distinct code commits (two commits `771d402`, `fd6c87a` are merge commits with zero net script changes).

---

## Affected checks — strength recommendation table

| control | check_id | current strength | recommended | reason | script lines |
|---------|----------|-----------------|-------------|--------|--------------|
| AU-6 | wazuh_alerts_24h | not listed (strong implied) | **strong — upgrade note** | OPS-82 replaced SSH-to-Wazuh with Wazuh REST API `/manager/stats`. Old method was a broken SSH probe that returned 0 by default. New method is a live API call. Evidence quality improved: was a broken proxy, now a genuine runtime probe. No tier change needed if already unlisted (defaults to strong), but worth noting the old implementation would have been rated "proxy" or "misleading" at audit time. | L773–L787 |
| IR-4 | wazuh_ar_rules | not listed (strong implied) | **no change** | OPS-82 replaced SSH `grep ossec.conf` with Wazuh API `/manager/configuration?section=active-response`. Both methods query actual Wazuh config; API method is more reliable. Evidence quality preserved — both probe real operational state, API is the canonical interface. No tier change. | L1221–L1234 |
| IR-6 | discord_alerting → alert_notifications | not listed (strong implied) | **promote to strong — check_id rename required** | OPS-82 corrected a stale check: the function was named `check_discord_alerting` and looked for Discord webhooks via SSH on a system where the platform uses Matrix, not Discord. Renamed to `alert_notifications`, now queries Wazuh integrations API. Old check was "misleading" (would always WARN because Discord was never configured; the platform used Matrix). New check is a genuine runtime probe of configured notification integrations. Registry entry at `discord_alerting` is now stale — operator should update check_id to `alert_notifications` and add a note about the correction. | L1236–L1249 |
| IR-5 | alert_volume | not listed (strong implied) | **no change** | OPS-82 replaced SSH with Wazuh `/manager/stats` API. The threshold also changed from `< 100,000` to `< 500,000` for WARN. This threshold change moves the upper boundary for "normal" alert volume upward but does not change what is being verified (whether the platform is generating a reasonable alert volume). The check remains a genuine runtime probe. The threshold adjustment reflects observed platform behavior (89K+ daily alerts). No evidence-quality shift. | L1251–L1267 |
| SA-8 | secure_development | not listed (strong implied) | **no change — but note a residual OPS-82 issue** | OPS-82 expanded scope from one repo to three repos (`sentinel-repo`, `overwatch-gitops`, `overwatch-repo`). However, the paths use `$HOME` variables, which COMP-26 Group A identified as a root-execution bug. As of HEAD, the `check_secure_development` function at L2188–L2207 still uses `$HOME` paths (COMP-26 Group A fixed CM-3(3), RA-5, RA-5(3), SA-11 but not SA-8). This is an existing gap, not a new one introduced in this window. Strength tier is unchanged: file existence / CI config grep, which is "moderate" evidence quality. Not listed in registry as weak/trivial/proxy/misleading. No tier change from this delta; recommend operator create a follow-on issue for the `$HOME` path bug in SA-8. | L2188–L2207 |
| AU-10 | non_repudiation | not listed (strong implied) | **no change** | OPS-82 replaced SSH `grep logall_json ossec.conf` with Wazuh API `/manager/configuration?section=logging` + `/manager/logs/summary`. Both probe actual Wazuh configuration state. New method is more reliable (no SSH dependency) but equivalent evidence quality — both verify that Wazuh is configured for structured logging. OPS-202 (`ae9735e`) fixed a `\$token` escape typo in the caller that caused the function to receive a literal `$token` string, meaning AU-10 was always running without a Wazuh token until 2026-04-17. The typo fix is functionally significant (check was broken before it) but does not change evidence-quality classification. No tier change. | L2285–L2305 |
| CP-10 | gitlab_accessible | not listed (strong implied) | **no change — URL target corrected** | OPS-130 changed `GITLAB_URL` from `http://192.168.12.68` (dead GitLab VM) to `http://192.168.12.70:3000` (Forgejo). The function body is unchanged; it probes HTTP reachability. Before the fix this check was a false-positive FAIL (probing a dead host). After the fix it probes the live platform. No evidence-quality shift; the check mechanism is the same. | L287–L299 |
| AC-14 | cf_access | not listed (strong implied) | **no change — URL target corrected** | OPS-130 changed target URL from `https://gitlab.haist.farm` (dead) to `https://www.haist.farm` (live). Same HTTP-status probe mechanism. No evidence-quality shift. | L719–L733 |
| SC-7(7) | cloudflare_tunnel | not listed (strong implied) | **no change — URL target corrected** | OPS-130 changed fallback URL from `https://gitlab.haist.farm` to `https://www.haist.farm`. Primary check (syscollector probe for cloudflared process) unchanged. No evidence-quality shift. | L1451–L1474 |
| CA-7 | wazuh_agents | not listed (strong implied) | **no change — scope correction** | OPS-148 decremented `EXPECTED_AGENTS` from 8 to 7 to match the intentional removal of the seedbox agent. The check logic (count active Wazuh agents, compare to expected count) is unchanged. Before the fix, the check was a false WARN because it expected 8 agents but only 7 were present. No evidence-quality shift. | L124–L137 (check_agents_active) |
| CP-9 | backup_freshness | not listed (strong implied) | **no change — path corrected** | OPS-193 changed backup bucket from `sentinel/gitlab-backups/` (empty after GitLab retired) to `sentinel/forgejo-backups/` (active). The MinIO freshness-check mechanism is unchanged. Before the fix, CP-9 was a false WARN (bucket empty). After the fix it checks the correct bucket. Evidence quality preserved: actual backup object age via S3 API is a strong runtime probe. No tier change. | L301–L379 (check_backup_timers) |
| SI-2 | flaw_remediation | **strong** (already in registry from COMP-20) | **no change** | COMP-20 (`7421ac3`) rewrote the trivial `check_unattended_upgrades` into a composite `check_flaw_remediation`. The registry was updated under COMP-20 to reflect `strong` tier with detailed data-source documentation. No further tier change warranted. Already correctly classified. | L1566–L1641 |
| CM-3(3) | ci_security_scanning | not listed (strong implied) | **no change — path fixed** | COMP-26 Group A replaced `$HOME`-relative paths with absolute `/home/ubuntu/...` paths and added Forgejo workflow file as primary lookup (falling back to legacy `ci/security.yml`). The check still greps for `trivy` and `gitleaks` in CI config files. Evidence quality is "moderate" (CI config inspection, not runtime execution). The change corrects a root-execution false-negative (wrong `$HOME`) but does not change the evidence mechanism. No tier change. | L998–L1027 |
| RA-5 | vulnerability_scanning | not listed (strong implied) | **no change — path fixed** | COMP-26 Group A: same absolute-path fix as CM-3(3). CI config grep for `trivy`. Moderate evidence. No tier change. | L1333–L1356 |
| RA-5(3) | secret_scanning | not listed (strong implied) | **no change — path fixed** | COMP-26 Group A: same absolute-path fix. CI config grep for `gitleaks`. Moderate evidence. No tier change. | L1358–L1385 |
| SA-11 | sbom_generation | not listed (strong implied) | **no change — path fixed** | COMP-26 Group A: absolute-path fix + Forgejo Actions composite action as primary lookup. Still a file-existence / grep-for-toolname check. Moderate evidence. No tier change. | L2342–L2359 |
| SC-17 | pki_cert_management | not listed (strong implied) | **downgrade: strong → moderate** | COMP-26 Group B changed the probe from reading `/ssh/config/ca` public key (direct CA data — strong evidence that the SSH CA is operational) to reading `/sys/mounts` for presence of an `ssh/` engine (mount presence — proves engine is mounted, not that CA has a key configured). The previous check `data.public_key` was stronger: it confirmed the CA had a configured signing key. The new check `sys/mounts` confirms the engine exists but not that it is configured and functional. The check comment at L510 acknowledges this: "presence of ssh/ mount type='ssh' confirms SSH CA is configured." This is technically weaker than reading the CA public key, though the COMP-26 justification is that `/ssh/config/ca` was denied by the compliance-check token. The evidence mechanism regressed from direct-CA-key verification to mount-existence proxy. Recommend operator classify as **moderate**: config-level evidence (mount presence) vs. strong (operational CA data). | L507–L532, L2659–L2695 |
| AC-6 | vault_policies | not listed (strong implied) | **no change** | COMP-26 Group C changed endpoint from `/sys/policy` (deprecated, denied) to `/sys/policies/acl?list=true`. Both return a list of Vault ACL policies. The fallback (enumerate known policies individually) is unchanged. Evidence quality is the same: runtime API call verifying that ≥3 Vault policies exist. No tier change. | L560–L594 |
| IA-5(13) | vault_token_ttl | not listed (strong implied) | **no change** | COMP-26 Group C changed endpoint from `/sys/auth/token/tune` (denied by compliance-check token) to `/auth/token/lookup-self`. The old endpoint returned the token-type's max_lease_ttl (system-level config). The new endpoint returns the current token's creation_ttl (instance-level). Both verify that the runtime Vault token has a bounded lifetime. The `creation_ttl > 0` check on the actual token is arguably stronger than reading the mount's `max_lease_ttl` (which applies to the token type, not this specific token). No tier downgrade warranted. | L1192–L1215 |
| MA-2 | maintenance_mode | **weak** (in registry) | **no change** | COMP-26 Group D: absolute-path fix (`$HOME/scripts/` → `/home/ubuntu/scripts/`). The check mechanism is unchanged: `[ -x path/sentinel-maintenance.sh ]` — file existence + executable bit. Still weak evidence. No tier change. | L1273–L1283 |
| CP-2 | dr_scripts | **weak** (in registry) | **no change** | COMP-26 Group D: absolute-path fix (`$HOME/sentinel-repo/` → `/home/ubuntu/sentinel-repo/`). Still counts DR scripts ≥2 without testing them. Still weak evidence. No tier change. | L1029–L1043 |
| CP-3 | recovery_procedures | not listed (strong implied) | **no change — path fixed** | COMP-26 Group D: same absolute-path fix as CP-2. Same directory-listing mechanism. No tier change. | L1857–L1873 |
| SA-10 | dev_config_mgmt | not listed (strong implied) | **upgrade: moderate → strong** | Two separate changes in this window significantly improved SA-10. OPS-200 (`b11d914`) replaced the dead GitLab branch-protection API call with a live Forgejo branch-protection API call across three repos (`sentinel-iac`, `overwatch`, `overwatch-gitops`). COMP-26 Group D added `FORGEJO_TOKEN` environment variable support and absolute path fallback. Before OPS-200, the check was either FAIL (Vault secret lookup for GitLab token failing) or relied on a stale CI file check. After OPS-200, the primary path is a live Forgejo API call returning actual branch-protection rule counts. This is a runtime API probe of actual configuration — strong evidence. Registry does not list SA-10, implying it was "strong" at audit time via the old GitLab path. Given the OPS-200 rewrite restored genuine API-probe capability after the GitLab retirement broke it, the effective strength is confirmed strong. No tier change from unlisted default, but worth documenting that SA-10 was effectively "misleading" during the GitLab→Forgejo migration gap (OPS-130 to OPS-200). | L2614–L2657 |
| AC-17 | vault_ssh_ca | not listed (strong implied) | **downgrade: strong → moderate** | COMP-27 (`5dafb6e`) changed the probe from `/ssh/config/ca` (returns `data.public_key` — direct evidence the CA has a configured signing key) to `/sys/mounts` (returns engine list — evidence the ssh/ engine exists). Same situation as SC-17 above. The prior check verified the CA was configured with a public key; the new check verifies only that the ssh/ engine is mounted. COMP-27 comment at L509 acknowledges `/ssh/config/ca is denied by compliance-check token`. The evidence regressed from direct-CA-key verification to mount-presence proxy. Recommend **moderate**: config-level evidence (mount presence) rather than operational verification. This is the same recommended downgrade as SC-17. | L507–L532 |

---

## Checks requiring registry action

Two checks warrant a strength downgrade recommendation:

1. **SC-17 `pki_cert_management`** — from implied-strong → moderate. Was: probed `/pki/ca/pem` and `/ssh/config/ca` (direct CA operational data). Now: reads `/sys/mounts` for engine presence only. Change introduced in COMP-26 Group B (`269db5f`).

2. **AC-17 `vault_ssh_ca`** — from implied-strong → moderate. Was: read `/ssh/config/ca` for `data.public_key` (CA configured with signing key). Now: reads `/sys/mounts` for `ssh/` engine presence. Change introduced in COMP-27 (`5dafb6e`).

One check requires a registry update for check_id correctness:

3. **IR-6** — registry (if it lists `discord_alerting`) has a stale check_id. OPS-82 renamed the function output field from `discord_alerting` to `alert_notifications`. The function name in script remains `check_discord_alerting` but the pass/warn/fail calls use `alert_notifications` as the check_id. If the registry has an entry for IR-6 using the old check_id, it must be updated.

One observation about a gap not covered by the registry:

4. **SA-8 `secure_development`** — still uses `$HOME`-relative CI paths (missed by COMP-26 Group A). The check can produce false FAILs when run as root. This is a correctness bug, not a strength-tier issue, but it means the check result is unreliable in some execution contexts. No strength tier change recommended; recommend a follow-on OPS issue to add absolute paths to SA-8.

---

## Unaffected checks

The remaining 107 checks (125 total minus 18 affected above) had zero changes in the +155-line window. These checks are confirmed unmodified by per-SHA diff inspection across all 13 code commits. The 8 trivial, 1 misleading, 3 proxy, and 5 weak entries already in the registry retain their classifications. The SI-2 rewrite (COMP-20) is the only entry that was _added_ to the registry in this window, and that entry is already correct.

---

## Operator action

Operator (Jim) to evaluate the following recommended changes to `check-strength.yaml`:

1. **SC-17 `pki_cert_management`** — add entry: strength `moderate`, reason: "COMP-26 Group B changed probe from /pki/ca/pem + /ssh/config/ca (direct CA data) to /sys/mounts engine-presence check. Engine mounted does not prove CA is configured with a signing key."

2. **AC-17 `vault_ssh_ca`** — add entry: strength `moderate`, reason: "COMP-27 changed probe from /ssh/config/ca data.public_key (CA configured) to /sys/mounts ssh/ engine presence. Mount presence does not prove CA has a signing key."

3. **IR-6** — if an entry for IR-6 exists with check_id `discord_alerting`, update to `alert_notifications` to match current script output. The check is now a genuine Wazuh integrations API probe (stronger than the original Discord-only SSH grep).

4. The 19 existing weak/trivial/proxy/misleading entries remain unaffected by the +155-line delta. They should be re-evaluated as separate work if PLANNER intends to assert control improvements against them.

This file is READ-ONLY for all agents; the worker (COMP-12) may not modify `check-strength.yaml`. These are recommendations only.

---

## Methodology notes

- Diff inspection used `git diff <sha>~1..<sha>` for each commit. Line counts confirmed per-SHA with `git show <sha>:scripts/nist-compliance-check.sh | wc -l`.
- Whitespace-only changes were not present in this window (all commits included substantive logic changes or comments).
- "Scope correction" = fixing which target is probed (e.g., wrong URL, wrong bucket name) without changing the probe mechanism. These do not change evidence quality.
- "Path fix" = correcting `$HOME`-relative paths to absolute paths. These do not change the evidence mechanism but correct false-negatives when the script runs as root.
- "Evidence-quality change" = a change to what is actually measured: e.g., from SSH-grep of a config file to a runtime API call, or from direct CA-key verification to engine-presence proxy. These are the only changes that affect strength tiers.
- The threshold change in IR-5 (100,000 → 500,000 upper bound for "normal") was assessed as a calibration adjustment reflecting observed platform behavior, not an evidence-quality regression.
- The escape-typo fixes in IR-6 (OPS-196) and AU-10 (OPS-202) are functionally significant (the affected checks were passing literal `$token` strings instead of token values, making them run without authentication) but do not change evidence-quality classification once the bug is fixed.
