# Design: Reliable VictoriaLogs ingestion and Grafana queries

## Architecture Overview

Alloy continues to send Loki-compatible batches. VictoriaLogs owns Loki-envelope decoding and structured-message parsing; Alloy only extracts supported source timestamps. The raw profile opts out of structured-message parsing, while the JSON profile uses the default parser and falls back naturally to the original message for non-JSON access lines.

## Technical Decisions

### Decision 1: Preserve streams and select structured messages

**Decision:** Remove `_stream_fields` overrides from both URLs. Keep `disable_message_parsing=1` in raw mode and use an explicit list of common message fields in structured mode.

**Rationale:** VictoriaLogs treats Loki labels as stream fields by default, preserving `filename`. On v1.50, selecting common fields such as `message`, `msg`, `log`, and `body` prevents parsed JSON records from receiving the missing-message default.

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
- Run the live diagnostic when Docker is available.

## Risks & Mitigations

- **Risk:** Historic timestamps may fall outside Grafana's current time range. **Mitigation:** Document widening the Explore time range and expose direct query diagnostics.
- **Risk:** Existing Alloy positions prevent re-reading files after the config changes. **Mitigation:** Keep backend-specific reset commands and document the exact reset/start sequence.
