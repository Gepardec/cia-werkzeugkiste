# Implementation Journal: Reliable VictoriaLogs ingestion and Grafana queries

## Summary

Completed all tasks. VictoriaLogs structured mode now gives arbitrary JSON a useful default line, preserves source event time and indexed fields, raw mode remains lossless, file labels remain stream fields, and operators have layered diagnostics plus one universal backend reset. Static and isolated live verification pass.

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
| 5 | Inject the complete JSON object as `_msg` and add one universal reset | A timestamp-only `_msg` is not a useful default display, and separate reset commands make it easy to leave stale data or positions behind. | Follow-up | 2026-08-11 |
| 6 | Parse JSON timestamps in both profiles and default `victorialogs` to structured mode | The retained container proved raw mode was active; Loki requires Alloy to put the source time in its envelope, and numeric Unix units need length-specific parsing. | Follow-up | 2026-08-11 |

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
- Default-display follow-up: inspected the retained backend data, confirmed stale records kept ingestion `_time`, injected the complete JSON object into `_msg`, and consolidated reset operations into `./log-stack reset`. An isolated Alloy v1.16/VictoriaLogs v1.50 run verified 300 JSON records and 100 access records with exact source-time bounds, complete non-empty messages, indexed nested fields, and zero Alloy errors.
- Timestamp follow-up: inspected the stopped container mounts and found the raw profile from an older worktree was active. Added JSON timestamp processing to raw mode, made structured mode the plain-command default, and routed Unix integers by length after a live test exposed nanoseconds being misread as seconds. Raw and structured profiles each ingested all 350 seven-format records into the 2024 time range with zero Alloy errors.

## Phase 3 Completion Summary

- Tasks completed: 3/3.
- Files modified: two Alloy profiles, datasource provisioning, `log-stack`, and component README.
- Deviations: none.
- Tests: shell syntax, three Compose configurations, pinned Alloy runtime loading, URL/help/datasource assertions, `git diff --check`, mixed-log fixtures, a 300-line message-less JSON plus 100-line access regression, two 350-line seven-format timestamp profile runs, timestamp boundaries, complete messages, structured fields, Alloy logs, command routing, and Grafana datasource health passed.

## Phase 2 Completion Summary

- Requirements: mixed JSON and access logs must both remain queryable; raw mode remains lossless.
- Design: preserve all Loki stream labels, select common structured message fields, parse JSON/ISO/access timestamps, and add layered diagnostics.
- Tasks: two small sequential changes covering ingestion then operations/docs.
- Dependencies: no new dependencies.
