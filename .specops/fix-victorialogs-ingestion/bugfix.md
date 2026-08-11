# Bug Fix: Reliable VictoriaLogs ingestion and Grafana queries

## Problem Statement

VictoriaLogs ingestion is unreliable for the two real input classes: JSON-lines application logs and plain HTTP access logs. Grafana often shows no useful records even though Alloy has consumed the files.

## Root Cause Analysis

Both VictoriaLogs profiles force `_stream_fields=job`, collapsing all files into one stream and discarding Alloy's useful `filename` stream label. The pipeline parses JSON and ISO-8601 timestamps but does not parse Apache/Nginx access timestamps, so access records receive ingestion time instead of event time. Live verification also showed that VictoriaLogs v1.50 needs an explicit parsed JSON message-field list after automatic JSON expansion; without it, `_msg` becomes the backend's default missing-message value.

The stack has no command that distinguishes backend reachability, Alloy delivery failure, and Grafana query failure, so a configuration error presents as an empty Explore result.

**Affected Components:**

- VictoriaLogs Alloy write endpoints
- VictoriaLogs Grafana datasource provisioning
- `log-stack` operational commands

**Error Symptoms:**

- JSON records may not have a useful `_msg` value.
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

Preserve all Alloy stream labels by removing `_stream_fields=job`, explicitly select common parsed JSON message fields for VictoriaLogs v1.50, and only disable JSON parsing in the raw profile. Parse Apache/Nginx access timestamps with Alloy's Go timestamp format. Add a diagnostic command that checks VictoriaLogs health, verifies the LogsQL query endpoint, and reports container state and recent logs when a check fails.

## Unchanged Behavior

- WHEN raw mode ingests a plaintext line THE SYSTEM SHALL CONTINUE TO store the complete line as `_msg`.
- WHEN Loki mode is selected THE SYSTEM SHALL CONTINUE TO use the existing Loki datasource and ingestion endpoint.

## Testing Plan

### Current Behavior (verify the bug exists)

- WHEN the original access-log pipeline ingests bracketed timestamps THE SYSTEM CURRENTLY assigns ingestion time, and `_stream_fields=job` collapses distinct files into one stream.

### Expected Behavior (verify the fix works)

- WHEN JSON and access logs are sent through the structured profile THE SYSTEM SHALL retain a non-empty `_msg`, parse valid JSON fields, and keep the `filename` stream label.
- WHEN diagnostics run against a started stack THE SYSTEM SHALL verify both the health and LogsQL query endpoints.

### Unchanged Behavior (verify no regressions)

- WHEN raw mode ingests JSON or plaintext THE SYSTEM SHALL CONTINUE TO preserve the original line.

## Acceptance Criteria

- [x] Regression Risk Analysis completed to the required depth for Medium severity
- [x] Bug reproduction confirmed by configuration inspection
- [x] Fix verified through static checks; the live smoke test was attempted and blocked by the unavailable Docker daemon
- [x] Raw-mode preservation behavior remains configured
