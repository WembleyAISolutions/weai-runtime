# Runbook — Bug Reproduce → Fix → Verify (Public)

Status: ACTIVE  
Scope: Public process only (no internal URLs/secrets)

---

## 1. Goal

Produce a deterministic, reviewable workflow that:
1) Reproduces a bug reliably
2) Identifies the cause with evidence
3) Implements a minimal fix
4) Verifies the fix with repeatable checks
5) Generates an Evidence Pack for PR review

---

## 2. Inputs

- Bug report (issue text or incident summary)
- Affected surface:
  - UI flow / API endpoint / background job
- Expected vs actual behavior
- Any available identifiers:
  - correlationId / requestId / traceId
  - timestamps and user journey steps

---

## 3. Reproduction (must become scriptable)

### 3.1 Create a minimal reproduction
- Prefer **Playwright/Appium** for UI
- Prefer **curl/HTTP script** for API
- Prefer **seeded fixture** for data-layer reproduction

### 3.2 Acceptance for “Reproduced”
- Repro succeeds ≥ 3 consecutive runs
- Repro output is captured:
  - screenshots/trace (UI) or request/response logs (API)
  - relevant error logs
  - trace ID or correlation ID

---

## 4. Diagnosis

### 4.1 Evidence-first
- Start from trace/correlationId if possible
- Link:
  UI action → request → service → error → log lines → metric anomaly

### 4.2 Define hypothesis
- Single-sentence cause hypothesis:
  “X fails because Y under condition Z.”

---

## 5. Fix

- Keep fix minimal and localized
- Update or add tests that fail before the fix and pass after
- Avoid refactors unless required for correctness

---

## 6. Verification

Minimum verification set:
- ✅ Unit/integration tests (as applicable)
- ✅ Repro script passes repeatedly (≥ 3 runs)
- ✅ UI automation / API script shows expected behavior
- ✅ Observability signals: error disappears or is reduced (where measurable)

---

## 7. Evidence Pack (mandatory)

Attach artifacts as defined in:
- `standards/EVIDENCE_PACK_STANDARD.md`

---

## 8. PR Requirements

PR must include:
- What broke + why (summary)
- How to reproduce (link to script/steps)
- What changed (diff summary)
- How verified (tests + evidence pack)
- Rollback notes (how to revert safely)
