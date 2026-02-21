# WeAI Runtime — Runbooks & Standards Index (Public SSoT)

Status: ACTIVE  
Last updated: 2026-02-22  
Repository: `weai-runtime` (Public)  
Scope: **Standards + Runbooks + Public Interfaces (No secrets, no internal URLs, no production implementation)**

---

## 0. Purpose

This repository is the **public, versioned source of truth** for:

- Runtime operational **runbooks** (SOPs)
- Cross-repo **standards** (observability, evidence, UI testability)
- Public **interface contracts** for agent tooling and observability query access

**Production implementations, credentials, dashboards, and automation code live in**: `weai-runtime-internal` (Private).

---

## 1. What belongs here (Public)

✅ Allowed:
- Runbooks (process, steps, acceptance criteria)
- Standard schemas and conventions (field names, trace conventions, test-id rules)
- Interface definitions (request/response shapes, not endpoints)
- PR templates/checklists (review gates, evidence requirements)

❌ Not allowed:
- Secrets, tokens, API keys
- Internal URLs, dashboard IDs, on-call phone numbers
- Proprietary production logic or sensitive infrastructure topology
- Customer data or incident details that reveal private environment information

---

## 2. Runbooks

- `runbooks/BUG_REPRO_FIX_VERIFY.md`  
  Reproduce → Diagnose → Fix → Verify → Evidence Pack → PR.
- `runbooks/INCIDENT_TRIAGE.md`  
  Triage and stabilize production signals (public process, no internal routing details).
- `runbooks/RELEASE_MONITOR_ROLLBACK.md`  
  Release guardrails, monitoring windows, rollback thresholds.

---

## 3. Standards

- `standards/EVIDENCE_PACK_STANDARD.md`  
  Defines what artifacts must be produced to claim a bug is reproduced and a fix is verified.
- `standards/UI_TESTID_A11Y_STANDARD.md`  
  Rules for stable UI automation (Playwright/Appium) using `data-testid` and accessibility labels.
- `standards/OTEL_CONVENTIONS.md`  
  OpenTelemetry naming and correlation conventions (logs/metrics/traces).
- `standards/PR_REVIEW_CHECKLIST.md`  
  Review checklist for runtime changes and agent-produced PRs.

---

## 4. Public Interfaces

- `interfaces/observability-query-gateway.md`  
  Public contract for querying logs/metrics/traces in a least-privilege manner.

> Note: Interface contracts here must be **environment-agnostic**.
> All real endpoints and credentials belong in `weai-runtime-internal`.

---

## 5. Evidence Pack Rule (Non-negotiable)

Any bug fix PR that claims “fixed” MUST attach an **Evidence Pack** that satisfies:
- Minimal reproduction steps (scripted where possible)
- Before/after proof (tests + UI + observability)
- Traceability identifiers (correlation IDs / trace IDs)
- Rollback notes (even if trivial)

See: `standards/EVIDENCE_PACK_STANDARD.md`.

---

## 6. Suggested PR Order (to build this system)

1) Add index + standards skeleton (this pack)  
2) Add first runbook: BUG_REPRO_FIX_VERIFY  
3) Add interfaces: observability query contract  
4) Implement automation + evidence workflows in `weai-runtime-internal`

---

## 7. Governance

- All changes: **PR-only**
- Keep **public** content free of environment secrets.
- If you’re unsure: place the detail into `weai-runtime-internal` and reference it here at a high level.
