# Bug Fix: Reliable VictoriaLogs ingestion and Grafana queries

## Problem Statement

VictoriaLogs ingestion is unreliable for the two real input classes: JSON-lines application logs and plain HTTP access logs. Grafana often shows no useful records even though Alloy has consumed the files.

## Root Cause Analysis

Both VictoriaLogs profiles force `_stream_fields=job`, collapsing all files into one stream and discarding Alloy's useful `filename` stream label. The pipeline parses JSON and ISO-8601 timestamps but does not parse Apache/Nginx access timestamps, so access records receive ingestion time instead of event time. Live verification also showed that VictoriaLogs v1.50 needs an explicit parsed JSON message-field list after automatic JSON expansion; without it, `_msg` becomes the backend's default missing-message value. Follow-up production payloads exposed two additional problems: valid JSON events may contain `@timestamp` and structured fields but no message-like field, and already-ingested records retain their old `_time` and `_msg` until both storage and Alloy positions are reset. Container inspection then confirmed the retained Alloy instance was mounted to the raw profile from an older worktree; raw mode had no JSON timestamp stages, and the plain `victorialogs` command selected that profile by default.

The stack has no command that distinguishes backend reachability, Alloy delivery failure, and Grafana query failure, so a configuration error presents as an empty Explore result.

**Affected Components:**

- VictoriaLogs Alloy write endpoints
- VictoriaLogs Grafana datasource provisioning
- `log-stack` operational commands

**Error Symptoms:**

- JSON records may not have a useful `_msg` value, especially when they contain no message-like field.
- The initial log view can be blank even though expanding a row exposes structured fields.
- Historic `@timestamp` values can remain ordinary fields while `_time` reflects ingestion time in stale records.
- Access and application logs share one indistinguishable stream.
- Operators cannot tell whether VictoriaLogs is unreachable or merely has no matching records.

## Impact Assessment

- **Severity:** Medium
- **Users Affected:** Users of the local VictoriaLogs mode
- **Frequency:** Often, especially with mixed input schemas and reused positions

## Reproduction Steps

1. Put JSON-lines and access-log files under `loki-grafana-analyzer/logs/`.
2. Start `./log-stack victorialogs-json`.
3. Query `*` or `{job="local_logs"}` in Grafana.
4. Expected: both payload classes are visible, with JSON fields available and files separated by stream labels.
5. Actual: results are often empty, missing useful messages, or impossible to distinguish by source file.

## Regression Risk Analysis

### Blast Radius

- Raw VictoriaLogs ingestion through `alloy-config.victorialogs.alloy`
- Structured VictoriaLogs ingestion through `alloy-config.victorialogs-json.alloy`
- Grafana datasource provisioning and helper commands

### Behavior Inventory

- Raw mode must preserve every physical file line as `_msg`.
- JSON mode must expose JSON fields without dropping plain access lines.
- Loki mode must remain unchanged.

### Risk Tier

| Behavior | Tier | Reason |
| --- | --- | --- |
| Raw lines remain queryable | Must-Test | The write URL changes directly affect ingestion semantics. |
| JSON fields remain queryable | Must-Test | The JSON profile relies on VictoriaLogs automatic parsing. |
| Loki mode remains unchanged | Low-Risk | No Loki files are modified. |

## Proposed Fix

Preserve all Alloy stream labels by removing `_stream_fields=job` and explicitly select common parsed JSON message fields for VictoriaLogs v1.50. In structured mode, copy the complete original JSON object into `_msg` before delivery so message-less events have a useful default line while VictoriaLogs still indexes every field. Parse `@timestamp` and the other supported source timestamps into the Loki envelope in both raw and structured modes, with length-specific Unix parsing to avoid treating nanoseconds as seconds. Make plain `victorialogs` select structured/mixed mode; raw mode remains an explicit opt-in. Add diagnostics and a single universal `reset` command that clears Loki and VictoriaLogs storage plus every Alloy positions volume while preserving Grafana.

## Unchanged Behavior

- WHEN raw mode ingests a plaintext line THE SYSTEM SHALL CONTINUE TO store the complete line as `_msg`.
- WHEN Loki mode is selected THE SYSTEM SHALL CONTINUE TO use the existing Loki datasource and ingestion endpoint.

## Testing Plan

### Current Behavior (verify the bug exists)

- WHEN the original access-log pipeline ingests bracketed timestamps THE SYSTEM CURRENTLY assigns ingestion time, and `_stream_fields=job` collapses distinct files into one stream.

### Expected Behavior (verify the fix works)

- WHEN JSON and access logs are sent through the structured profile THE SYSTEM SHALL retain a non-empty `_msg`, parse valid JSON fields, and keep the `filename` stream label.
- WHEN a JSON event has only `@timestamp` and arbitrary structured fields THE SYSTEM SHALL use the complete original JSON object as `_msg`, retain the parsed fields, and store `@timestamp` as event time.
- WHEN either VictoriaLogs profile ingests JSON with a supported `@timestamp` representation THE SYSTEM SHALL store the parsed source time in `_time` rather than ingestion time.
- WHEN the operator starts `./log-stack victorialogs` THE SYSTEM SHALL use structured/mixed mode; raw mode SHALL require `victorialogs-raw`.
- WHEN diagnostics run against a started stack THE SYSTEM SHALL verify both the health and LogsQL query endpoints.
- WHEN the operator runs `./log-stack reset` THE SYSTEM SHALL stop all profiles and remove Loki data, VictoriaLogs data, and every Alloy positions volume while preserving Grafana data.

### Unchanged Behavior (verify no regressions)

- WHEN raw mode ingests JSON or plaintext THE SYSTEM SHALL CONTINUE TO preserve the original line.

## Acceptance Criteria

- [x] Regression Risk Analysis completed to the required depth for Medium severity
- [x] Bug reproduction confirmed by configuration inspection
- [x] Fix verified through static checks and isolated live round-trips
- [x] Raw-mode preservation behavior remains configured
