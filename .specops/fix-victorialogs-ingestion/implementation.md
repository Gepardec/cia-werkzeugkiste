# Implementation Journal: Reliable VictoriaLogs ingestion and Grafana queries

## Summary

Completed both tasks. VictoriaLogs structured mode now follows the native Loki ingestion contract for mixed JSON and access logs, raw mode remains lossless, file labels remain stream fields, and operators have a layered health/query diagnostic. Shell syntax, URL assertions, and every Compose profile passed; live backend verification could not run because the local Docker daemon is stopped.

## Phase 1 Context Summary

- Config: defaults; no `.specops.json`; specsDir `.specops`; taskTracking `none`
- Context recovery: no incomplete specs
- Steering files: created and loaded `product.md`, `tech.md`, `structure.md`, `dependencies.md`, `repo-map.md`
- Repo map: generated from 22 tracked files
- Memory: no memory files found
- Vertical: infrastructure
- Affected files: `loki-grafana-analyzer/alloy-config.victorialogs*.alloy`, datasource provisioning, `log-stack`, and README
- Project state: brownfield
- Vocabulary check: pass
- Plan validation: pass — all five affected paths exist

## Decision Log

| # | Decision | Rationale | Task | Timestamp |
|---|----------|-----------|------|-----------|
| 1 | Preserve default Loki stream fields and explicitly select common parsed JSON message fields | This retains `filename` while avoiding VictoriaLogs v1.50's missing-message default after JSON expansion. | 1 | 2026-07-31 |
| 2 | Query `*` from epoch start in the diagnostic | This verifies LogsQL independently of Grafana's current time picker and historical source timestamps. | 2 | 2026-07-31 |
| 3 | Parse Apache/Nginx timestamps in both VictoriaLogs profiles | Access-log event time must remain independent of ingestion time. | Follow-up | 2026-07-31 |
| 4 | Append supported timestamp fields to the structured `_msg_field` list | Message-less JSON events still need a deterministic `_msg`; their source timestamp is present and remains independently stored as event time. | Follow-up | 2026-08-11 |

## Deviations from Design

| Planned | Actual | Reason | Task |
|---------|--------|--------|------|

## Blockers Encountered

| Blocker | Resolution | Impact | Task |
|---------|------------|--------|------|
| Docker daemon unavailable | Ran all static checks and exercised the diagnostic failure path; documented the user-run live command. | Live ingestion round-trip not executed locally. | 2 |

## Documentation Review

- `loki-grafana-analyzer/README.md`: updated with structured/mixed behavior, reset/start/check workflow, filename filtering, and historical time-range guidance.
- Root `README.md`: checked; it does not describe this component's runtime behavior and needs no change.

## Session Log

- Phase 1/2: inspected the existing stack, confirmed Compose merging, and compared the VictoriaLogs URLs with the official Loki-ingestion contract.
- Task 1 scope: remove schema-coupled Loki query parameters, preserve raw-mode line fidelity, and retain filename labels as stream fields.
- Task 1 completed: both profiles preserve default Loki stream fields; raw mode remains lossless and structured mode selects common parsed JSON message fields.
- Task 2 scope: add a layered VictoriaLogs health/query diagnostic, increase the provisioned display limit, and document deterministic mixed-log startup and recovery.
- Task 2 completed: added backend and LogsQL checks with failure diagnostics, provisioned a 1000-line Grafana limit, and updated operator guidance. All static checks pass; the live check correctly failed with a stopped Docker daemon.
- Follow-up verification: started isolated VictoriaLogs v1.50, Alloy v1.16, and Grafana v13.0.1 containers; ingested 300 JSON and 300 access records; verified exact counts, source-time boundaries, message bodies, zero Alloy errors, and Grafana datasource status `OK`. Added access-log timestamp parsing after the first live run exposed ingestion-time fallback.
- Message-less JSON follow-up: added timestamp candidates as the final structured `_msg` fallback and verified the behavior with two 300-line fixtures containing timestamps and arbitrary fields but no message-like property. The variant fixture evenly covered `@timestamp`, `timestamp`, `time`, and `ts`.

## Phase 3 Completion Summary

- Tasks completed: 2/2.
- Files modified: two Alloy profiles, datasource provisioning, `log-stack`, and component README.
- Deviations: none.
- Tests: shell syntax, three Compose configurations, URL assertions, help output, datasource assertion, `git diff --check`, 600-line mixed-log ingestion, a 300-line message-less JSON ingestion, timestamp boundaries, messages, structured fields, Alloy logs, and Grafana datasource health passed.

## Phase 2 Completion Summary

- Requirements: mixed JSON and access logs must both remain queryable; raw mode remains lossless.
- Design: preserve all Loki stream labels, select common structured message fields, parse JSON/ISO/access timestamps, and add layered diagnostics.
- Tasks: two small sequential changes covering ingestion then operations/docs.
- Dependencies: no new dependencies.
