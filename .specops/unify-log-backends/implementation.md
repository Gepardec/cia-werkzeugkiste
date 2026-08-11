# Implementation Journal: Unified local log backends

## Summary

Unified the supported inputs, timestamps, display behavior, and diagnostics across Loki and VictoriaLogs; upgraded every pinned service; removed Grafana credentials in favor of anonymous localhost access; and reduced the component README from 96 to 51 lines.

## Phase 1 Context Summary

- Config: defaults; no `.specops.json`; task tracking and git checkpointing disabled
- Context recovery: none; prior VictoriaLogs bugfix used as relevant history
- Steering files: loaded product, tech, structure, repo-map, dependencies
- Repo map: stale by age; refreshed with unchanged source hash
- Memory: loaded 3 decisions from 1 completed spec; 0 recurring patterns
- Vertical: infrastructure
- Affected files: Alloy configs, Compose profiles, `.env.example`, component README
- Project state: brownfield
- Vocabulary check: pass
- Plan validation: pass — 9 file references resolved

## Phase 2 Completion Summary

- Loki and VictoriaLogs will share file and timestamp semantics while retaining native query behavior.
- Stable release pins verified from official upstream release pages on 2026-08-11.
- Three tasks cover ingestion parity, versions/anonymous access, and concise documentation.
- No new dependencies introduced.

## Decision Log

| # | Decision | Rationale | Task | Timestamp |
|---|----------|-----------|------|-----------|
| 1 | Flush idle historical Loki chunks after five seconds | Loki accepted the 350-line fixture but skipped its old in-memory chunks during queries; the documented ingester lookback override did not alter single-binary query routing in the tested 3.7 release, while automatic idle flushing reliably made records visible. | Task 1 | 2026-08-11 |
| 2 | Give Loki a health-plus-query diagnostic | VictoriaLogs already verified both backend health and its query path; adding the equivalent Loki command makes the operator workflow backend-neutral. | Task 1 | 2026-08-11 |
| 3 | Check Loki's service endpoint instead of startup-gated readiness | Loki 3.7.6 returned successful historical queries while `/ready` still reported its post-ring startup delay; `/services` plus a real LogQL query detects an available local backend without that false negative. | Task 1 | 2026-08-11 |

## Deviations from Design

| Planned | Actual | Reason | Task |
|---------|--------|--------|------|
| Parser-only Loki parity | Also tune Loki idle flush timing | Live testing showed historical records were accepted but invisible until their chunks reached the store. | Task 1 |
| Direct Loki `/ready` diagnostic | Use `/services` plus a real LogQL query | Loki 3.7.6 intentionally returned 503 during its post-ring readiness delay even while historical queries already succeeded. | Task 1 |

## Blockers Encountered

| Blocker | Resolution | Impact | Task |
|---------|------------|--------|------|
| Minimal Loki and VictoriaLogs images lacked `wget` during isolated datasource probing | Published temporary localhost-only test ports and probed from the host | Test-harness-only retry; no product change | Task 2 |

## Documentation Review

- `loki-grafana-analyzer/README.md` — updated: replaced repeated setup, alias, and troubleshooting sections with a 51-line operator reference covering no-login Grafana, event time, fields, diagnostics, and universal reset.
- Root and image READMEs — up-to-date: none reference this optional log-analysis component or its credentials, versions, commands, or endpoints.
- `CLAUDE.md` and `docs/` — not present; no additional documentation target required an update.

## Session Log

- 2026-08-11: Created the refactor spec after inspecting the existing stack and verifying official stable releases.
- 2026-08-11: Spec evaluation passed at 8/10 or higher in every dimension; dependency and reference gates passed.
- Task 1 scope: align Loki with the tested VictoriaLogs JSON, Unix, ISO, and access timestamp stages; retain raw lines and verify field queries plus historical timestamps in both backends.
- Task 1 parity addition: add `check-loki` beside `check-victorialogs` so either backend can be checked through a real query.
- Task 1 complete: all Alloy profiles validate, file globs match, and live 350-line historical fixtures passed in both backends; Loki made old ranges visible after seven seconds without a forced flush.
- Task 2 scope: pin the four verified current stable images, use the current Grafana plugin preload variable, remove password configuration, and prove anonymous local API access.
- Task 2 complete: all Compose profiles render; current images load; the shared 350-line fixture passes on Loki 3.7.6 and VictoriaLogs 1.52.0 through Alloy 1.18.1; Grafana 13.1.3 returns HTTP 200 anonymously with its login form disabled.
- Task 3 scope: replace the profile-heavy README with a compact start, timestamp/field behavior, query, diagnostics, stop, and one-command reset reference.
- Task 3 complete: the README is 51 lines, all documented commands resolve in `log-stack`, and no credential or obsolete `.env.example` reference remains.
- Final verification: shell syntax, all three Compose renders, all three Alloy loads, 350-line Loki and VictoriaLogs round-trips, anonymous Grafana API access, and both provisioned Grafana datasource health APIs passed on the final versions.
