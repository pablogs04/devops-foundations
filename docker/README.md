# Docker (Sprint 1)

## Lab 1 — Nginx serving a bind-mounted static site

### Goal
Serve a local `index.html` with Nginx using a read-only bind mount.

### Run (from repo root)
```bash
volume path wrong if you run from ~
docker run -d --name web -p 8082:80 \
  -v "$(pwd)/docker/nginx-static:/usr/share/nginx/html:ro" \
  nginx:alpine

## Lab 2 — Same setup using Docker Compose

### Run
From repo root:
- `cd docker`
- `docker compose up -d`

### Status / logs
- `docker compose ps`
- `docker compose logs --tail 20 web`

### Debug (exec)
- `docker compose exec web sh`
  - `ls -la /usr/share/nginx/html`
  - `echo "test" >> /usr/share/nginx/html/index.html`  # should fail (read-only)

### Stop & clean
- `docker compose down`

