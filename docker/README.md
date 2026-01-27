# Docker (Sprint 1)

## Lab 1 — Nginx serving a bind-mounted static site

### Goal
Serve a local `index.html` with Nginx using a read-only bind mount.

### Run (from repo root)
```bash
docker run -d --name web -p 8082:80 \
  -v "$(pwd)/docker/nginx-static:/usr/share/nginx/html:ro" \
  nginx:alpine

