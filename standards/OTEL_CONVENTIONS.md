# Standard — OpenTelemetry Conventions (Public)

Status: ACTIVE  
Scope: Public standard (no internal endpoints, no secrets)

## 0. Purpose

Provide deterministic correlation across:
- UI events
- API requests
- service traces
- structured logs
- metrics

Agents must be able to traverse:
UI step → request → trace → log lines → metric anomalies.

## 1. Correlation Fields (Minimum)

Structured logs MUST include (where applicable):
- `trace_id`
- `span_id`
- `correlation_id` (or `request_id`)
- `service`
- `env`
- `severity`
- `event_name`

Recommended:
- `user_id_hash` (never raw PII)
- `tenant_id` (if multi-tenant)
- `build_sha` / `version`

## 2. Trace / Span Naming

### 2.1 Span name format (recommended)
`<layer>.<operation>.<resource>`

Examples:
- `api.request.POST_/orders`
- `runtime.execute.skill_resolve`
- `gateway.validate.execution_object`
- `db.query.orders_by_id`

### 2.2 Attributes (recommended)
- `http.method`, `http.route`, `http.status_code`
- `error.type`, `error.message` (redacted)
- `component` (db/cache/queue/etc.)
- `peer.service` (downstream service)

## 3. Error Recording Rules

When an error occurs:
- span MUST be marked error
- log entry MUST include correlation fields + error summary
- metric SHOULD increment an error counter (e.g. `http_server_errors_total`)

Avoid logging secrets. Redact tokens and user PII.

## 4. Metrics Naming (Guidance)

Use conventional names:
- `*_total` counters
- `*_seconds` duration histograms
- `*_current` gauges

Examples:
- `runtime_jobs_total`
- `runtime_job_duration_seconds`
- `gateway_validation_failures_total`

## 5. Sampling Guidance (Public)

- Prefer head sampling in low-volume environments
- Ensure errors are sampled at higher priority
- Provide deterministic correlation even when traces are sampled (via correlation_id)

## 6. Compliance

A PR that introduces new runtime behavior MUST:
- include trace/log correlation for critical paths
- ensure errors are observable via at least logs + metrics
