# Implementation Tasks: Unified local log backends

## Task Breakdown

### Task 1: Align ingestion behavior

**Status:** Completed
**Estimated Effort:** M
**Dependencies:** None
**Priority:** High
**IssueID:** None
**Blocker:** None

**Description:** Give Loki and both VictoriaLogs profiles the same discovery and timestamp coverage while preserving backend-native field behavior.

**Implementation Steps:**

1. Add the tested JSON, Unix, compact-offset, line-leading ISO, and access timestamp stages to Loki.
2. Flush completed historical chunks automatically within the acceptance window.
3. Keep discovery globs identical across all profiles.
4. Verify raw lines remain visible and arbitrary JSON fields remain queryable.
5. Add the same service-health plus real-query diagnostic for Loki that VictoriaLogs provides.

**Acceptance Criteria:**

- [x] All profiles discover the same supported files.
- [x] Both backends store `@timestamp` with compact offsets as event time.
- [x] Both backends store access-log event time.
- [x] Historical Loki imports are queryable within 10 seconds without a forced flush.
- [x] JSON fields are queryable and original lines are non-empty.
- [x] Both backends expose a health-plus-query diagnostic command.

**Files to Modify:**

- `loki-grafana-analyzer/alloy-config.alloy`
- `loki-grafana-analyzer/alloy-config.victorialogs.alloy`
- `loki-grafana-analyzer/alloy-config.victorialogs-json.alloy`
- `loki-grafana-analyzer/loki-config.yaml`
- `loki-grafana-analyzer/log-stack`

**Tests Required:**

- [x] Alloy configuration load checks pass.
- [x] Live Loki and VictoriaLogs fixture tests pass.
- [x] Shell syntax and diagnostic command checks pass.

---

### Task 2: Upgrade and remove Grafana credentials

**Status:** Completed
**Estimated Effort:** M
**Dependencies:** Task 1
**Priority:** High
**IssueID:** None
**Blocker:** None

**Description:** Upgrade the stack to current stable releases and enable credential-free local Grafana access.

**Implementation Steps:**

1. Update the four image pins and plugin preload variable.
2. Enable anonymous Admin and disable basic authentication and login UI.
3. Remove the obsolete password example.

**Acceptance Criteria:**

- [x] Compose renders the four verified stable image versions.
- [x] Grafana accepts unauthenticated local API requests.
- [x] No Grafana username or password setting remains.

**Files to Modify:**

- `loki-grafana-analyzer/docker-compose.yaml`
- `loki-grafana-analyzer/docker-compose.loki.yaml`
- `loki-grafana-analyzer/docker-compose.victorialogs.yaml`
- `loki-grafana-analyzer/docker-compose.victorialogs-json.yaml`
- `loki-grafana-analyzer/.env.example`

**Tests Required:**

- [x] Every Compose profile validates.
- [x] Anonymous Grafana smoke test passes.

---

### Task 3: Condense operator documentation

**Status:** Completed
**Estimated Effort:** S
**Dependencies:** Task 2
**Priority:** Medium
**IssueID:** None
**Blocker:** None

**Description:** Replace the long profile guide with a short start-query-reset reference.

**Implementation Steps:**

1. Document the two primary backend commands and optional raw mode.
2. State timestamp and JSON behavior once.
3. Keep only essential URLs, queries, checks, stop, and reset commands.

**Acceptance Criteria:**

- [x] README is no longer than 60 lines.
- [x] README states Grafana needs no login.
- [x] README documents the universal reset command.

**Files to Modify:**

- `loki-grafana-analyzer/README.md`

**Tests Required:**

- [x] Documented commands exist in `log-stack`.
- [x] README line-count check passes.

## Implementation Order

1. Task 1
2. Task 2
3. Task 3

## Progress Tracking

- Total Tasks: 3
- Completed: 3
- In Progress: 0
- Blocked: 0
- Pending: 0
