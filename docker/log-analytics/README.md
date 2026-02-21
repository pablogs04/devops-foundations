# Nginx Log Analytics Stack (Sprint 1)

Mini-stack DevOps reproducible para **generar, persistir y analizar access logs**:
- **Nginx** sirve contenido estático y escribe logs a fichero.
- **GoAccess** genera un dashboard HTML en tiempo real a partir del access log.
- (Sprint 1) Este stack se conecta con un **CLI Python** (repo `python-devops-basics`) para sacar métricas JSON.

## Architecture
Host ports:
- Nginx: http://localhost:8085
- GoAccess dashboard: http://localhost:7890/report.html

Volumes:
- `./public` -> site content (read-only)
- `./logs` -> persistent nginx logs (host filesystem)
- `./reports` -> generated GoAccess report (host filesystem)

## Quickstart
From `devops-foundations/docker/log-analytics`:
1) `docker compose up -d`
2) Generate traffic:
   - `curl -s http://localhost:8085 >/dev/null`
   - `curl -s http://localhost:8085/NOEXISTE >/dev/null`
3) Verify logs:
   - `tail -n 5 logs/access.log`
4) Open dashboard:
   - http://localhost:7890/report.html

## Debugging
- Container status: `docker compose ps`
- Nginx logs (stdout/stderr): `docker compose logs --tail 50 nginx`
- GoAccess logs: `docker compose logs --tail 50 goaccess`
- Exec into nginx: `docker compose exec nginx sh`
  - Check config: `nginx -T | head`
  - Check logs dir: `ls -la /var/log/nginx`

## Notes
- In many Docker images, Nginx logs are redirected to stdout/stderr. Here we override it to write to real files using `configs/nginx.conf`.
- Access logs can be created by root inside the container; reading them from the host is enough for analytics.

## Stop & clean
- `docker compose down`
