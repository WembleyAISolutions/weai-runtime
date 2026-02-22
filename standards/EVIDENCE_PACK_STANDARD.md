# Standard — Evidence Pack (Public)

Status: ACTIVE  
Purpose: Define the minimum artifacts required to claim (a) a bug is reproduced and (b) a fix is verified.

---

## 1. Definition

An **Evidence Pack** is a structured collection of artifacts attached to a PR (or linked from it) that proves:

- Reproduction is real and repeatable
- Fix addresses the cause
- Fix is verified by repeatable checks
- Outcome is observable in logs/metrics/traces where applicable

---

## 2. Minimal Contents (Required)

### A) Reproduction
- Repro steps (human readable)
- Repro script location (preferred)
- Before-state evidence:
  - UI trace/screenshot/recording OR request/response dump
  - Error message / stack trace snippet
  - Correlation ID / trace ID (if available)

### B) Fix & Verification
- Test evidence:
  - failing test (before) and passing test (after), or equivalent proof
- After-state evidence:
  - UI trace/screenshot/recording OR request/response dump showing expected behavior
- Repeatability:
  - “Repro passes ≥ 3 runs” statement + automation output (if available)

### C) Observability (when applicable)
- Logs sample (structured)
- Metrics snapshot (before/after, or error rate trend)
- Trace link/reference (trace ID + service path)

### D) Rollback Notes
- One-paragraph rollback instruction:
  - revert commit / disable flag / restore config

---

## 3. Naming Convention (Recommended)

When uploaded as a build artifact or linked bundle, use:

`evidence-pack_<repo>_<pr#>_<yyyymmdd>_<shortdesc>.zip`

---

## 4. Storage

Public repo should only store:
- The **standard**, not sensitive artifacts.

Actual evidence artifacts should be stored:
- As CI artifacts (private)
- Or in `weai-runtime-internal` secure storage
- PR should link to the artifact location.

---

## 5. Non-Compliance

A PR claiming “fixed” without an Evidence Pack is **incomplete** and should not be merged.
