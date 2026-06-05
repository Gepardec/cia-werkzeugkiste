# Switchable Loki / VictoriaLogs local log-analysis stack

## Folder layout

```text
.
├── docker-compose.yaml
├── docker-compose.loki.yaml
├── docker-compose.victorialogs.yaml
├── log-stack
├── loki-config.yaml
├── alloy-config.alloy
├── alloy-config.victorialogs.alloy
├── grafana/
│   ├── provisioning/
│   │   └── datasources/
│   │       └── loki.yaml
│   └── provisioning-victorialogs/
│       └── datasources/
│           └── victorialogs.yaml
└── logs/
    └── put-your-log-files-here.log
```

## Start a backend

```bash
mkdir -p logs
# Copy your .log, .txt, .json, or rotated .log.* files into ./logs.
# Compressed archives are ignored by default.
# Example: cp -r /path/to/logs/* ./logs/

cp .env.example .env
# Optional: edit .env and set GF_SECURITY_ADMIN_PASSWORD to a non-default value.
```

Start Loki mode:

```bash
./log-stack loki
```

Start VictoriaLogs mode:

```bash
./log-stack victorialogs
```

`./log-stack vl` is a shorter alias for VictoriaLogs mode. The script wraps the explicit Docker Compose override commands and keeps `--remove-orphans` enabled for switching backends.

Open Grafana:

```text
http://localhost:3000
```

Default login:

```text
User: admin
Password: value of GF_SECURITY_ADMIN_PASSWORD in .env, or admin if unchanged
```

Useful health/debug endpoints:

```text
Loki ready endpoint:       http://localhost:3100/ready
VictoriaLogs endpoint:     http://localhost:9428
VictoriaLogs built-in UI:  http://localhost:9428/select/vmui/
Alloy UI:                  http://localhost:12345
```

## Loki query examples in Grafana Explore

```logql
{job="local_logs"}
```

```logql
{job="local_logs"} |= "ERROR"
```

```logql
{job="local_logs"} | json
```

```logql
{job="local_logs"} | logfmt
```

## VictoriaLogs query examples in Grafana Explore

```text
*
```

```text
{job="local_logs"}
```

```text
{job="local_logs"} i(error)
```

## Reset data or file positions

Alloy stores file positions in backend-specific Docker volumes. This lets VictoriaLogs ingest the same local files even after Loki has already read them.

Reset Loki data:

```bash
./log-stack reset-loki-data
```

Reset Loki Alloy positions:

```bash
./log-stack reset-loki-positions
```

Reset VictoriaLogs data:

```bash
./log-stack reset-victorialogs-data
```

Reset VictoriaLogs Alloy positions:

```bash
./log-stack reset-victorialogs-positions
```

## Retention

Retention is disabled by default so your imported log data is not deleted unexpectedly. To enable automatic deletion, set `limits_config.retention_period` in `loki-config.yaml` and uncomment the `compactor` block at the bottom of that file.

Example:

```yaml
limits_config:
  retention_period: 720h  # 30 days

compactor:
  working_directory: /loki/compactor
  retention_enabled: true
  retention_delete_delay: 2h
  delete_request_store: filesystem
```
