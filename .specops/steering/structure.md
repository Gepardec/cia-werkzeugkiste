---
name: "Project Structure"
description: "Directory layout, key files, and module boundaries"
inclusion: always
---

## Directory Layout

- `loki-grafana-analyzer/`: switchable Loki/VictoriaLogs analysis stack.
- `images/`: reusable debugging container images.

## Key Files

- `loki-grafana-analyzer/docker-compose*.yaml`: base and backend-specific services.
- `loki-grafana-analyzer/alloy-config*.alloy`: file discovery, parsing, and delivery.
- `loki-grafana-analyzer/log-stack`: operator commands.

## Module Boundaries

Alloy tails mounted files and sends Loki-compatible batches to the active backend; Grafana queries that backend through a provisioned datasource.
