# Design: Reliable VictoriaLogs ingestion and Grafana queries

## Architecture Overview

Alloy continues to send Loki-compatible batches. VictoriaLogs owns Loki-envelope decoding and structured-message parsing. Alloy extracts supported source timestamps and, for JSON objects, injects a copy of the complete original object as `_msg` so the default log view is useful without sacrificing indexed fields. The raw profile opts out of structured-message parsing, while non-JSON access lines pass through unchanged.

## Technical Decisions

### Decision 1: Preserve streams and select structured messages

**Decision:** Remove `_stream_fields` overrides from both URLs. Keep `disable_message_parsing=1` in raw mode. In structured mode, copy JSON objects into an injected `_msg`, prefer real message fields when present, and otherwise select the injected copy.

**Rationale:** VictoriaLogs treats Loki labels as stream fields by default, preserving `filename`. On v1.50, selecting common fields such as `message`, `msg`, `log`, and `body` prevents parsed JSON records from receiving the missing-message default. The injected `_msg` gives arbitrary message-less JSON a meaningful initial display while automatic parsing keeps every source field indexed.

### Decision 4: Make JSON re-ingestion atomic for operators

**Decision:** Add `reingest-victorialogs-json`, which stops both VictoriaLogs profiles, removes backend data and both VictoriaLogs Alloy position volumes, then starts structured mode.

**Rationale:** Stored `_time` and `_msg` values are immutable. A single explicit command prevents partial resets that leave stale data or consumed file positions behind.

### Decision 3: Parse access-log event time

**Decision:** Extract bracketed Apache/Nginx timestamps and parse them with `02/Jan/2006:15:04:05 -0700` in both VictoriaLogs profiles.

**Rationale:** Offline access logs must retain their event time rather than the later ingestion time.

### Decision 2: Diagnose each connection layer

**Decision:** Add a `check-victorialogs` command that checks `/-/healthy`, runs a minimal `*` LogsQL query, and emits service state/logs on failure.

**Rationale:** A single command separates backend reachability from ingestion/query problems and provides actionable output.

### Dependency Decisions

No new dependencies introduced. The diagnostic uses the existing Docker CLI and `curl` already expected for local HTTP checks.

## Testing Strategy

- Validate all Compose profiles with `docker compose config`.
- Validate `log-stack` with `sh -n`.
- Assert VictoriaLogs write URLs match the intended raw and structured contracts.
- Ingest a few hundred `@timestamp`-only structured records and verify count, exact `_time`, complete JSON `_msg`, and arbitrary nested fields.
- Run the live diagnostic when Docker is available.

## Risks & Mitigations

- **Risk:** Historic timestamps may fall outside Grafana's current time range. **Mitigation:** Document widening the Explore time range and expose direct query diagnostics.
- **Risk:** Existing Alloy positions prevent re-reading files after the config changes. **Mitigation:** Provide one explicit reset-and-reingest command while retaining the granular reset commands.
