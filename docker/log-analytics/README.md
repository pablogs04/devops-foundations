
# Nginx Log Analytics Stack (Sprint 1)

Mini-stack DevOps reproducible para **generar, persistir y analizar access logs**.

## What you get
- **Nginx** serves a static site and writes access/error logs to files.
- **GoAccess** reads the access log and generates a dashboard (`report.html`).
- The output can be consumed by a **Python CLI** (repo `python-devops-basics`) to produce JSON metrics.

## Architecture
Ports:
- Nginx: `http://localhost:8085`
- Report served by Nginx: `http://localhost:8085/reports/report.html`
- GoAccess realtime websocket server: `ws://localhost:7890`

Volumes (host -> container):
- `./public` -> `/usr/share/nginx/html` (read-only)
- `./configs/nginx.conf` -> `/etc/nginx/nginx.conf` (read-only)
- `./logs` -> `/var/log/nginx` (Nginx writes `access.log` and `error.log`)
- `./reports` -> `/reports` (GoAccess writes `report.html`)

## Quickstart
From `devops-foundations/docker/log-analytics`:

1) Start the stack
```bash
docker compose up -d
docker compose ps