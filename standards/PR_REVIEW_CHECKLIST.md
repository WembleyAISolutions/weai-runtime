# Standard — PR Review Checklist (Runtime) (Public)

Status: ACTIVE  
Audience: Human reviewers + Review Agents  
Scope: Public checklist (no internal endpoints, no secrets)

## 0. Rule

A PR that claims “fixed” MUST include an Evidence Pack or a link to it.

See: `standards/EVIDENCE_PACK_STANDARD.md`

---

## 1. Scope & Boundaries

- [ ] Change stays within Runtime scope (no Kernel/AAA contamination)
- [ ] Public repo contains no secrets or internal endpoints
- [ ] Interfaces/contracts remain stable (or breaking change is explicit)

## 2. Correctness

- [ ] Reproduction steps are clear and minimal
- [ ] Root cause is stated in one sentence
- [ ] Fix addresses the cause (not only symptoms)
- [ ] Edge cases considered (inputs, retries, idempotency)

## 3. Testing & Verification

- [ ] Tests added/updated and pass
- [ ] If UI-related: stable `data-testid` / accessibility labels exist
- [ ] Repro script (preferred) or repeatability proof exists (≥ 3 runs)
- [ ] Evidence Pack attached/linked if claiming “fixed”

## 4. Observability

- [ ] Correlation fields present in logs (`trace_id`/`correlation_id`)
- [ ] Trace spans named consistently (see OTEL conventions)
- [ ] Metrics reflect success/failure states where appropriate

## 5. Rollback

- [ ] Rollback steps included (even if trivial)
- [ ] Safe revert path noted (flag/commit/release)

## 6. Risk Notes (Recommended)

- [ ] Risk level declared: low/medium/high
- [ ] If medium/high: monitoring signals and rollback trigger thresholds listed
