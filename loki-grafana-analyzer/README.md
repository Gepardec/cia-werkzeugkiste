# Local log analysis with Loki or VictoriaLogs

Alloy reads files from `./logs` and sends them to either backend. Grafana is available only on localhost and opens without a login.

## Run

```bash
mkdir -p logs
./log-stack loki
# or
./log-stack victorialogs
```

Open http://localhost:3000 and set Explore's time range to the dates inside your logs. Use `./log-stack victorialogs-raw` only when every line must remain completely unparsed.

## Ingestion behavior

- Every profile reads `.log`, `.txt`, `.json`, and rotated `.log.*` files; compressed archives are ignored.
- JSON timestamps come from `@timestamp`, `timestamp`, `time`, or `ts`. RFC3339/ISO-8601—including `+0100`—and Unix second, millisecond, microsecond, and nanosecond values are supported.
- ISO timestamps at the start of a line and Apache/Nginx timestamps such as `[03/Jan/2026:19:58:28 +0100]` are also supported. Missing or invalid timestamps fall back to ingestion time.
- Loki keeps the complete line and exposes arbitrary JSON fields with LogQL `| json`.
- VictoriaLogs structured mode exposes JSON fields and keeps the complete original object as the default `_msg`, so the initial log view is not empty. Non-JSON access logs remain visible.

## Query

Loki:

```logql
{job="local_logs"} | json
{job="local_logs"} | json | level="error"
```

VictoriaLogs:

```text
*
{job="local_logs"}
```

## Operate

```bash
./log-stack check-loki
./log-stack check-victorialogs
./log-stack down
./log-stack reset
```

`reset` is the single cleanup command for all Loki, VictoriaLogs, and Alloy data; Grafana data is preserved. Backend endpoints are Loki at http://localhost:3100, VictoriaLogs at http://localhost:9428, and Alloy at http://localhost:12345.

The stack pins Alloy 1.18.1, Grafana 13.1.3, Loki 3.7.6, and VictoriaLogs 1.52.0.
