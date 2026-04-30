# Loki + Grafana local log-analysis stack

## Folder layout

```text
.
├── docker-compose.yaml
├── loki-config.yaml
├── alloy-config.alloy
├── grafana/
│   └── provisioning/
│       └── datasources/
│           └── loki.yaml
└── logs/
    └── put-your-log-files-here.log
```

## Start

```bash
mkdir -p logs
# Copy your .log, .txt, .json, or rotated .log.* files into ./logs
# Example: cp -r /path/to/logs/* ./logs/

docker compose up -d
```

Open Grafana:

```text
http://localhost:3000
```

Default login:

```text
admin / admin
```

Useful health/debug endpoints:

```text
Loki ready:  http://localhost:3100/ready
Alloy UI:    http://localhost:12345
```

## Query examples in Grafana Explore

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

## Re-ingest from the beginning

Alloy stores file positions in the `alloy-data` Docker volume. If you want to re-read the same files from the beginning, remove the volumes:

```bash
docker compose down -v
docker compose up -d
```

This also deletes Loki and Grafana persisted data. To reset only Alloy positions, remove only the Alloy volume:

```bash
docker compose down
docker volume rm loki-grafana-log-analysis_alloy-data
docker compose up -d
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
