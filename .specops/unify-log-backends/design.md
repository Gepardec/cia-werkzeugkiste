# Design: Unified local log backends

## Architecture Overview

Alloy remains the single file reader. Each backend profile uses the same discovery and timestamp pipeline, while only the write endpoint and VictoriaLogs JSON message shaping differ. Loki preserves the raw line and exposes arbitrary JSON fields with LogQL `| json`; VictoriaLogs structured mode parses the object and injects a complete `_msg` fallback.

## Technical Decisions

### Keep backend-native field queries

**Decision:** Preserve full JSON lines in Loki and parse arbitrary fields at query time instead of promoting a fixed field list during ingestion.

**Rationale:** JSON schemas vary by file. LogQL `| json` discovers arbitrary keys without creating high-cardinality labels or discarding fields.

### Share timestamp semantics

**Decision:** Copy the tested VictoriaLogs timestamp stages into the Loki pipeline and use the same file globs in every profile.

**Rationale:** Backend selection must not change source event time. The pipeline tries JSON time first, then line-leading ISO time, then access-log time; failed parsing safely falls back to scrape time.

### Flush historical Loki chunks promptly

**Decision:** Check for flushes every second and flush chunks after five idle seconds.

**Rationale:** Loki skips ingesters for query ranges older than three hours. A completed historical file becomes idle immediately, so a five-second idle flush makes it queryable from the chunk store without an operator-only `/flush` call. Active current streams remain queryable from ingester memory.

### Use stable, explicit image pins

**Decision:** Pin Alloy 1.18.1, Grafana 13.1.3, Loki 3.7.6, and VictoriaLogs 1.52.0, verified from their official release pages on 2026-08-11.

**Rationale:** Explicit pins are reproducible while still meeting the request to update to the current stable releases.

### Make Grafana anonymous only on localhost

**Decision:** Enable anonymous Admin, disable basic authentication and the login form, and keep the host bind at `127.0.0.1`.

**Rationale:** This removes credentials for the local analysis tool while retaining access to Explore and datasource administration. Remote exposure remains out of scope.

## Infrastructure Topology

- `alloy`: discovers and tails local files, parses source event time, forwards raw Loki batches.
- `loki` or `victorialogs`: stores one active backend's logs.
- `grafana`: anonymous local UI with one provisioned datasource.
- `log-stack`: offers equivalent start, status, service-health/query check, stop, config, and reset operations for both backends.

## Failure Modes

- Invalid or missing timestamps use ingestion time and keep the original line.
- Invalid JSON remains a plain log line.
- A missing backend fails Alloy delivery and is visible in container logs.
- A missing VictoriaLogs datasource plugin prevents datasource provisioning; Grafana preloads the latest catalog release during startup.

## Validation Strategy

- Run shell syntax and Docker Compose rendering checks.
- Run both backend service-health-plus-query commands against live services.
- Load every Alloy configuration with Alloy 1.18.1.
- Ingest 300 historical JSON lines plus access-log lines into isolated Loki and VictoriaLogs stacks.
- Query each backend and assert stored timestamp bounds, line count, field filtering, and non-empty displayed content.
- Query Loki without forcing a chunk flush and assert historical records appear within 10 seconds.
- Start Grafana 13.1.3 and confirm an unauthenticated API request succeeds.

### Dependency Decisions

No new dependencies introduced. Existing container images are upgraded to verified stable releases.
