# overwatch-showcase

Public-facing showcase for the Overwatch platform, built for the Codex
Creator Challenge / project gallery submission.

- **Thesis:** verification over trust. AI agents cannot reliably self-report
  compliance state; ground truth comes from deterministic tooling outside
  the AI trust chain.
- **What this repo is:** a static HTML representation of the platform,
  plus the evidence artifacts each claim on that page is grounded in.
  It is deliberately separate from the platform repos (which are private)
  — the repo itself is part of the thesis.
- **What this repo is NOT:** the platform. No live data. No iframes into
  `*.208.haist.farm`. No backend. No fetches. Static only.

## Layout

- [showcase-ground-truth.md](showcase-ground-truth.md) — single source of
  truth for every claim that appears in `index.html`. One claim = one row.
- [evidence/](evidence/) — raw artifacts (compliance JSON, derivations,
  redacted screenshots). Judges can click through to verify.
- `index.html` *(pending HAIST-28)* — the submitted showcase page.

## Tracking

- Plane parent: HAIST-26 — *Public showcase page for Overwatch platform*
- HAIST-27 — Phase 1, ground-truth collection (this commit range)
- HAIST-28 — Phase 2, HTML build (blocked on operator sign-off of ground truth)
- HAIST-29 — drift fix-ups found while building (non-blocking)

## Branch convention

Worker branches follow the platform convention: `worker/issue-{ID}-{short}`.
PRs open against this repo's `main` after Jim's review of the artifact each
branch produces.
