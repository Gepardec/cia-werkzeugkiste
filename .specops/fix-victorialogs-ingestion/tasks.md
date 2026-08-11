# Implementation Tasks: Reliable VictoriaLogs ingestion and Grafana queries

## Task Breakdown

### Task 1: Correct the VictoriaLogs ingestion contract

**Status:** Completed
**Estimated Effort:** S
**Dependencies:** None
**Priority:** High
**IssueID:** None
**Blocker:** None

**Description:**
Remove redundant Loki-ingestion overrides while retaining the raw profile's explicit message-parsing opt-out.

**Implementation Steps:**

1. Update the raw and JSON Alloy write endpoints.
2. Preserve automatic Loki stream labels.
3. Clarify comments around mixed JSON/access-log behavior.

**Acceptance Criteria:**

- [x] JSON profile uses VictoriaLogs automatic Loki message parsing.
- [x] JSON profile falls back to a supported timestamp when no message-like field exists.
- [x] Raw profile preserves original lines.
- [x] Both profiles retain Alloy's filename stream label.

**Files to Modify:**

- `loki-grafana-analyzer/alloy-config.victorialogs.alloy`
- `loki-grafana-analyzer/alloy-config.victorialogs-json.alloy`

**Tests Required:**

- [x] Static endpoint assertions pass.
- [x] A live timestamp-only JSON fixture preserves source time and structured fields.

### Task 2: Add connection diagnostics and operator guidance

**Status:** Completed
**Estimated Effort:** S
**Dependencies:** Task 1
**Priority:** Medium
**IssueID:** None
**Blocker:** None

**Description:**
Add an actionable health/query check and document mixed-log ingestion, time-range behavior, and positions resets.

**Implementation Steps:**

1. Add `check-victorialogs` to `log-stack`.
2. Configure a practical Grafana line limit.
3. Update README startup, query, and troubleshooting guidance.

**Acceptance Criteria:**

- [x] Diagnostic checks health and LogsQL query endpoints.
- [x] Diagnostic prints container state and recent logs on failure.
- [x] README gives a deterministic reset/start/check workflow.

**Files to Modify:**

- `loki-grafana-analyzer/log-stack`
- `loki-grafana-analyzer/grafana/provisioning-victorialogs/datasources/victorialogs.yaml`
- `loki-grafana-analyzer/README.md`

**Tests Required:**

- [x] Shell syntax and Compose configuration checks pass.
- [x] Live diagnostic is run or the missing Docker daemon is recorded.

## Progress Tracking

- Total Tasks: 2
- Completed: 2
- In Progress: 0
- Blocked: 0
- Pending: 0
