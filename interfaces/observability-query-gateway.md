# Interface — Observability Query Gateway (Public Contract)

Status: DRAFT  
Scope: Public contract only (no internal endpoints, no secrets)

## 0. Goal

Provide a least-privilege, environment-agnostic interface that agents and CI
can use to query logs/metrics/traces for Evidence Packs and debugging.

This document defines request/response shapes only.
Concrete endpoints, credentials, dashboards, and datasource mappings belong in:
`weai-runtime-internal`.

## 1. Principles

- Least privilege by default
- All queries require explicit time bounds
- Hard caps on result size
- Redaction of sensitive values
- Auditability: queries must be logged (internal implementation)

## 2. Common Types

### 2.1 TimeRange
- `from`: ISO-8601 timestamp
- `to`: ISO-8601 timestamp

### 2.2 QueryLimits
- `limit`: integer (capped)
- `cursor`: optional pagination cursor

## 3. Logs API (Conceptual)

### Request
- `service`: string
- `time_range`: TimeRange
- `filter`: string (structured filter or query language abstraction)
- `limits`: QueryLimits

### Response
- `entries`: list of log entries (structured)
- `next_cursor`: optional string
- `sample_trace_ids`: optional list of strings

## 4. Metrics API (Conceptual)

### Request
- `metric_name`: string
- `labels`: map<string,string> (optional)
- `time_range`: TimeRange
- `resolution`: string (e.g., "30s", "1m", "5m")

### Response
- `series`: list of { `ts`, `value`, `labels` }

## 5. Traces API (Conceptual)

### Request (preferred)
- `trace_id`: string

### Request (search)
- `service`: string
- `time_range`: TimeRange
- `where`: string (attribute filter abstraction)
- `limits`: QueryLimits

### Response
- `traces`: list of summarized traces
- `spans`: optional list (if requested)
- `sample_correlation_ids`: optional list

## 6. Security Requirements (Implementation)

Implementation MUST enforce:
- auth scoping per repo/workflow/agent role
- maximum time window (e.g., <= 2 hours by default)
- maximum results (e.g., <= 500 entries)
- redaction of secrets/PII
- audit logs of all queries

## 7. Evidence Pack Integration

Evidence Pack generation should:
- query logs/metrics/traces bounded to the repro window
- attach trace_ids/correlation_ids
- link the pack to the PR

See: `standards/EVIDENCE_PACK_STANDARD.md`
