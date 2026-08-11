# Switchable Loki / VictoriaLogs log-analysis stack

One active backend/parser profile at a time:

- Loki: Loki + Alloy + Grafana
- VictoriaLogs raw/non-JSON: VictoriaLogs + Alloy + Grafana, preserving each file line as `_msg`
- VictoriaLogs structured/mixed: VictoriaLogs + Alloy + Grafana, parsing JSON fields while preserving plain access-log lines

## Setup

```bash
mkdir -p logs
cp .env.example .env
```

Put `.log`, `.txt`, `.json`, or rotated `.log.*` files in `./logs`. Compressed archives are ignored by default.

Choose a VictoriaLogs parser profile before ingesting:

- Raw/non-JSON mode stores every file line unchanged as `_msg`. Use it when exact line fidelity matters and parse JSON later with `unpack_json`.
- Structured/mixed mode is recommended when a folder contains JSON-lines application logs and access logs. VictoriaLogs parses valid JSON messages into fields, selects common message fields such as `message`, `msg`, `log`, or `body` for `_msg`, and keeps non-JSON lines unchanged. If a JSON event has no message-like field, its first available timestamp field (`@timestamp`, `timestamp`, `time`, or `ts`) becomes `_msg` instead, while its other fields remain queryable. Alloy also stores that source timestamp as event time, recognizes ISO-8601 timestamps at the start of a line, and parses Apache/Nginx access timestamps such as `[31/Jul/2026:12:30:00 +0200]`.

If you switch parser profiles for the same files, reset VictoriaLogs data and the matching Alloy positions before re-ingesting.

## Start or switch backend

```bash
./log-stack loki
./log-stack victorialogs      # raw/non-JSON mode
./log-stack victorialogs-json # structured/mixed mode (JSON + access logs)
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
./log-stack check-victorialogs
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

## VictoriaLogs troubleshooting

For a mixed folder of JSON application logs and access logs, start from clean backend and position volumes once after this configuration change:

```bash
./log-stack reset-victorialogs-data
./log-stack reset-victorialogs-all-positions
./log-stack victorialogs-json
./log-stack check-victorialogs
```

Then query `*` in Grafana Explore. If the source files contain old timestamps, widen Explore's time range to cover those timestamps; Alloy deliberately preserves recognized source timestamps. Filter a particular file with `{filename="/logs/path/to/file.log"}` or all local files with `{job="local_logs"}`.

`check-victorialogs` verifies the backend health endpoint and a real LogsQL query. On failure it prints the service state plus recent VictoriaLogs and Alloy logs, which distinguishes a datasource connection problem from an ingestion or positions problem.

Loki retention is disabled in `loki-config.yaml`. VictoriaLogs runs with `-retentionPeriod=100y`.
