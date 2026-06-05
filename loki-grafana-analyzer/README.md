# Switchable Loki / VictoriaLogs log-analysis stack

One active backend/parser profile at a time:

- Loki: Loki + Alloy + Grafana
- VictoriaLogs raw/non-JSON: VictoriaLogs + Alloy + Grafana, preserving each file line as `_msg`
- VictoriaLogs JSON: VictoriaLogs + Alloy + Grafana, parsing JSON fields at ingestion time

## Setup

```bash
mkdir -p logs
cp .env.example .env
```

Put `.log`, `.txt`, `.json`, or rotated `.log.*` files in `./logs`. Compressed archives are ignored by default.

Choose a VictoriaLogs parser profile before ingesting:

- Raw/non-JSON mode is safest for plaintext logs, WildFly/access logs, mixed folders, and JSON logs without a stable message field. It stores every file line as `_msg`.
- JSON mode is for JSON-lines logs. It parses JSON fields at ingestion time, reads timestamps from `@timestamp`, `timestamp`, `time`, or `ts`, and uses the first non-empty field from `_msg`, `message`, `msg`, `log`, `event.original`, `body`, or `text` as `_msg`.

If you switch parser profiles for the same files, reset VictoriaLogs data and the matching Alloy positions before re-ingesting.

## Start or switch backend

```bash
./log-stack loki
./log-stack victorialogs      # raw/non-JSON mode
./log-stack victorialogs-json # JSON parser mode
```

`./log-stack vl` is an alias for VictoriaLogs raw/non-JSON mode. `./log-stack vlj` is an alias for VictoriaLogs JSON mode. The script uses Docker Compose overrides with `--remove-orphans`, so switching removes the previous backend container.

## Open

- Grafana: http://localhost:3000
- Login: `admin` / `GF_SECURITY_ADMIN_PASSWORD` from `.env`, or `admin`
- Loki ready: http://localhost:3100/ready
- VictoriaLogs: http://localhost:9428
- VictoriaLogs UI: http://localhost:9428/select/vmui/
- Alloy UI: http://localhost:12345

## Query examples

Loki:

```logql
{job="local_logs"}
{job="local_logs"} |= "ERROR"
```

VictoriaLogs:

```text
*
{job="local_logs"}
{job="local_logs"} i(error)
* | unpack_json  # useful in raw/non-JSON mode for JSON lines
```

## Useful commands

```bash
./log-stack help
./log-stack ps-loki
./log-stack ps-victorialogs
./log-stack ps-victorialogs-json
./log-stack down
./log-stack config-loki
./log-stack config-victorialogs
./log-stack config-victorialogs-json
```

## Reset

Alloy positions are backend-specific, so VictoriaLogs can ingest the same local files after Loki has already read them.
After changing parser behavior or replacing log files, reset both the backend data and the matching Alloy positions before re-ingesting.

```bash
./log-stack reset-loki-data
./log-stack reset-loki-positions
./log-stack reset-victorialogs-data
./log-stack reset-victorialogs-positions
./log-stack reset-victorialogs-json-positions
./log-stack reset-victorialogs-all-positions
```

Loki retention is disabled in `loki-config.yaml`. VictoriaLogs runs with `-retentionPeriod=100y`.
