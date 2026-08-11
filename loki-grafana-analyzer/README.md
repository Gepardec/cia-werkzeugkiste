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
- Structured/mixed mode is recommended when a folder contains JSON-lines application logs and access logs. VictoriaLogs parses valid JSON messages into fields and keeps non-JSON lines unchanged. A common message field such as `message`, `msg`, `log`, or `body` is displayed when present; otherwise `_msg` contains the complete original JSON object, so the initial log view is never blank while the other fields remain queryable. Alloy stores `@timestamp`, `timestamp`, `time`, or `ts` as event time, recognizes ISO-8601 timestamps at the start of a line, and parses Apache/Nginx access timestamps such as `[31/Jul/2026:12:30:00 +0200]`.

Both VictoriaLogs modes extract JSON event time. Supported values include RFC3339/ISO-8601 (including space or comma variants and timezone offsets such as `+01:00` or `+0100`) and Unix seconds, milliseconds, microseconds, or nanoseconds. The plain `victorialogs` command selects structured/mixed mode by default.

If you switch parser profiles for the same files, reset VictoriaLogs data and the matching Alloy positions before re-ingesting.

## Start or switch backend

```bash
./log-stack loki
./log-stack victorialogs      # structured/mixed mode (JSON + access logs)
./log-stack victorialogs-raw  # raw/non-JSON mode
```

`./log-stack vl`, `./log-stack vlj`, and `./log-stack victorialogs-json` are aliases for structured/mixed mode. Raw mode must be selected explicitly with `./log-stack victorialogs-raw`. The script uses Docker Compose overrides with `--remove-orphans`, so switching removes the previous backend container.

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

Reset every Loki/VictoriaLogs backend data volume and every Alloy positions volume with one command. Grafana data is preserved:

```bash
./log-stack reset
```

## VictoriaLogs troubleshooting

For a mixed folder of JSON application logs and access logs, reset all stored backend data and positions, then start JSON mode:

```bash
./log-stack reset
./log-stack victorialogs-json
```

After startup, `./log-stack check-victorialogs` optionally verifies backend health and the LogsQL query endpoint.

Then query `*` in Grafana Explore. If the source files contain old timestamps, widen Explore's time range to cover those timestamps; Alloy deliberately preserves recognized source timestamps. Filter a particular file with `{filename="/logs/path/to/file.log"}` or all local files with `{job="local_logs"}`.

`check-victorialogs` verifies the backend health endpoint and a real LogsQL query. On failure it prints the service state plus recent VictoriaLogs and Alloy logs, which distinguishes a datasource connection problem from an ingestion or positions problem.

Loki retention is disabled in `loki-config.yaml`. VictoriaLogs runs with `-retentionPeriod=100y`.
