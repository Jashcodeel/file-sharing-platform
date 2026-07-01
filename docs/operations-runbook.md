# Operations Runbook

This runbook contains the basic operational commands for the self-hosted file sharing platform running on AWS EC2 with Docker Compose.

---

## Project Location

```bash
cd /root/file-sharing-platform
```

---

## Start / Stop Services

Start all containers:

```bash
docker compose up -d
```

Stop all containers:

```bash
docker compose down
```

Restart all containers:

```bash
docker compose restart
```

Validate Compose file:

```bash
docker compose config
```

---

## Check Container Status

```bash
docker ps
```

Expected containers:

```text
nginx
nextcloud
postgres
redis
prometheus
grafana
cadvisor
node-exporter
```

---

## Check Logs

```bash
docker logs nginx --tail 100
docker logs nextcloud --tail 100
docker logs postgres --tail 100
docker logs redis --tail 100
docker logs prometheus --tail 100
docker logs grafana --tail 100
```

Follow live logs:

```bash
docker logs -f nginx
docker logs -f nextcloud
```

---

## Application Access

Nextcloud:

```text
https://EC2_PUBLIC_IP
```

Grafana:

```text
http://EC2_PUBLIC_IP:3000
```

Note: Nextcloud uses a self-signed certificate, so browser warning is expected.

---

## Nextcloud Commands

Check status:

```bash
docker exec -u www-data nextcloud php occ status
```

Enable maintenance mode:

```bash
docker exec -u www-data nextcloud php occ maintenance:mode --on
```

Disable maintenance mode:

```bash
docker exec -u www-data nextcloud php occ maintenance:mode --off
```

Check trusted domains:

```bash
docker exec -u www-data nextcloud php occ config:system:get trusted_domains
```

---

## Nginx Commands

Test Nginx config:

```bash
docker exec nginx nginx -t
```

Reload Nginx:

```bash
docker exec nginx nginx -s reload
```

Restart Nginx:

```bash
docker restart nginx
```

---

## Storage Checks

Check disk usage:

```bash
df -h
docker system df
```

List Docker volumes:

```bash
docker volume ls
```

Check volume size:

```bash
sudo du -sh /var/snap/docker/common/var-lib-docker/volumes/*
```

---

## Backup

Run backup:

```bash
./scripts/backup.sh
```

Backups are stored under:

```text
backups/<timestamp>/
```

Verify backup:

```bash
ls -lh backups/
tar -tzf backups/<timestamp>/nextcloud_data.tar.gz | head
head -n 20 backups/<timestamp>/postgres.sql
```

---

## Restore Summary

Restore should follow this order:

```text
1. Stop stack
2. Restore PostgreSQL database
3. Restore Nextcloud data volume
4. Restore Nginx certificates
5. Start stack
6. Disable maintenance mode
7. Validate login and uploaded files
```

Important: Nextcloud `config.php` database credentials must match PostgreSQL credentials after restore.

---

## Monitoring

Restart monitoring stack:

```bash
docker restart prometheus grafana cadvisor node-exporter
```

Check monitoring logs:

```bash
docker logs prometheus --tail 100
docker logs grafana --tail 100
```

---

## Network Checks

List networks:

```bash
docker network ls
```

Inspect networks:

```bash
docker network inspect file-sharing-platform_frontend
docker network inspect file-sharing-platform_backend
docker network inspect file-sharing-platform_monitoring
```

Expected:

```text
frontend: nginx, nextcloud
backend: nextcloud, postgres, redis
monitoring: prometheus, grafana, cadvisor, node-exporter
```

---

## Common Issues

### 502 Bad Gateway

Check:

```bash
docker ps
docker logs nginx --tail 100
docker logs nextcloud --tail 100
```

### Database Error

Check:

```bash
docker logs postgres --tail 100
docker logs nextcloud --tail 100
```

### Large Upload Error

Check Nginx config:

```text
client_max_body_size 10G;
```

Reload Nginx:

```bash
docker exec nginx nginx -s reload
```

### Low Disk Space

Check:

```bash
df -h
docker system df
```

---

## Commands to Avoid

Do not run unless you intentionally want to delete persistent data:

```bash
docker compose down -v
```

This can delete uploaded files, database data, Grafana dashboards, and Prometheus metrics.

---

## Daily Health Check

```text
[ ] All containers are running
[ ] Nextcloud is accessible
[ ] Grafana is accessible
[ ] PostgreSQL is healthy
[ ] Redis is healthy
[ ] Disk usage is safe
[ ] Recent backup exists
[ ] No major errors in logs
```

Useful commands:

```bash
docker ps
df -h
docker logs nginx --tail 50
docker logs nextcloud --tail 50
ls -lh backups/
```
