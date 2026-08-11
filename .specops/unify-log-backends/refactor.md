# Refactor: Unify the Loki and VictoriaLogs stack

## Motivation

The local log stack currently gives VictoriaLogs broader timestamp parsing than Loki, uses outdated container pins, requires Grafana credentials, and documents too many aliases and profile details.

## Current State

- VictoriaLogs parses JSON `@timestamp`, alternate JSON timestamp fields, Unix timestamps, line-leading ISO timestamps, and Apache/Nginx access timestamps.
- Loki only parses line-leading RFC3339 timestamps, and newly ingested historical chunks are not queried until they flush because the default ingester query window is three hours.
- Grafana requires an admin password even though every published port is bound to `127.0.0.1`.
- The README is 96 lines and repeats profile and troubleshooting information.

## Target State

Both backends accept the same files and source timestamp formats, preserve the original log line, expose JSON fields through their native query language, and provide a health-plus-query diagnostic. The stack uses current stable releases, Grafana opens locally without credentials, and the README documents only the main workflow.

## Scope & Boundaries

- **In scope:** Alloy ingestion and diagnostic parity, current stable image pins, current Grafana plugin preload syntax, anonymous local Grafana access, and concise documentation.
- **Out of scope:** Production authentication, remote exposure, dashboards, compressed-file ingestion, and making Loki and LogsQL query syntax identical.
- **Behavioral changes:** Grafana becomes anonymous local Admin; Loki gains source timestamp parsing; documented aliases are reduced without removing script compatibility.

## Migration Strategy

Apply the config changes together, validate every Compose profile, then ingest representative historical JSON and access logs into both backends. Existing stored samples remain readable; `./log-stack reset` provides a clean re-ingest when needed.

## Risk Assessment

- **Regression risk:** Medium because timestamps, image versions, datasource loading, and authentication all affect startup or query behavior.
- **Rollback plan:** Revert the Compose and Alloy config changes; no data migration is performed.

## Success Metrics

- A 300-line JSON fixture using `{"@timestamp":"2026-01-03T19:58:28.891+0100"}` stores event time, not ingestion time, in Loki and VictoriaLogs.
- Apache/Nginx access timestamps store the source event time in both backends.
- All Compose profiles validate and Grafana APIs respond without authentication.
- The component README is no longer than 60 lines.

## Acceptance Criteria

- [x] Loki and VictoriaLogs cover the same supported file extensions and timestamp formats.
- [x] Newly ingested historical Loki records become queryable automatically within 10 seconds without a manual flush.
- [x] JSON fields remain queryable and every entry has a useful default displayed line.
- [x] Both backends provide a health-plus-query diagnostic command.
- [x] Container images use the stable upstream releases current on 2026-08-11.
- [x] Grafana requires no username or password on the local-only endpoint.
- [x] The README is at most 60 lines and documents start, query, check, stop, and universal reset.
- [x] Static checks and live sample ingestion pass.

## Dependencies & Blockers

No spec dependencies or blockers.
