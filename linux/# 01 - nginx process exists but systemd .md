# 01 - nginx process exists but systemd service does not

## Symptom

`systemctl status nginx --no-pager` returned:

`Unit nginx.service could not be found.`

At the same time, `ps aux | grep nginx` showed an active nginx master process and worker process.

## Why this matters in real-world DevOps

A process can be running without being managed by `systemd`. If engineers assume the wrong runtime layer, they may waste time, restart the wrong component, or miss the actual source of logs and configuration.

## Environment

- Host OS: AlmaLinux 9.7
- Context: local Linux drill on a test server
- Runtime discovered: Docker
- Relevant containers:
  - `la-nginx`
  - `la-goaccess`

## Diagnostic flow used

1. Confirm host context
2. Check service status
3. Check listening ports
4. Check running processes
5. Check service logs
6. Verify container runtime
7. Validate published port and HTTP response

## Commands used

```bash
hostnamectl
systemctl status nginx --no-pager
ss -lntp
ps aux | grep nginx
sudo journalctl -u nginx -n 20 --no-pager
docker ps
docker logs --tail 20 la-nginx
docker inspect --format '{{json .NetworkSettings.Ports}}' la-nginx
curl -I http://127.0.0.1:8085
curl -I http://localhost:8085