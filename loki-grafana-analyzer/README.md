# Switchable Loki / VictoriaLogs log-analysis stack

One active backend at a time:

- Loki: Loki + Alloy + Grafana
- VictoriaLogs: VictoriaLogs + Alloy + Grafana with the VictoriaLogs datasource plugin

## Setup

```bash
mkdir -p logs
cp .env.example .env
```

Put `.log`, `.txt`, `.json`, or rotated `.log.*` files in `./logs`. Compressed archives are ignored by default.

## Start or switch backend

```bash
./log-stack loki
./log-stack victorialogs
```

`./log-stack vl` is an alias for VictoriaLogs. The script uses Docker Compose overrides with `--remove-orphans`, so switching removes the previous backend container.

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
```

## Useful commands

```bash
./log-stack help
./log-stack ps-loki
./log-stack ps-victorialogs
./log-stack down
./log-stack config-loki
./log-stack config-victorialogs
```

## Reset

Alloy positions are backend-specific, so VictoriaLogs can ingest the same local files after Loki has already read them.

```bash
./log-stack reset-loki-data
./log-stack reset-loki-positions
./log-stack reset-victorialogs-data
./log-stack reset-victorialogs-positions
```

Loki retention is disabled in `loki-config.yaml`. VictoriaLogs runs with `-retentionPeriod=100y`.
