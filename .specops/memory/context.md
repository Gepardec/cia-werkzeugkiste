# Project Memory

## Completed Specs

### fix-victorialogs-ingestion (bugfix) — 2026-07-31

Completed two tasks and a live follow-up to make mixed JSON/access-log ingestion preserve messages, filename streams, and source event timestamps while adding layered connection diagnostics. Static checks and an isolated 600-line VictoriaLogs/Alloy/Grafana round-trip pass.

### unify-log-backends (refactor) — 2026-08-11

Completed three tasks to align Loki and VictoriaLogs input, source-time, field-display, and diagnostic behavior; upgrade all four service pins; remove Grafana credentials; and condense the README. Static validation, matching 350-line historical round-trips, anonymous Grafana access, and both provisioned datasource health APIs pass on the final versions.
