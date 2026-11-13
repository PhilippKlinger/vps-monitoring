# vps-monitoring runbook

Operating notes for the monitoring stack. The [incident assistant](https://github.com/PhilippKlinger/vps-incident-assistant) reads the Troubleshooting section below and links its cases in incident issues.

## Troubleshooting

Commands run in `/srv/monitoring`. Problems of the proxy itself are in the runbook of [vps-edge](https://github.com/PhilippKlinger/vps-edge/blob/main/RUNBOOK.md#troubleshooting).

### Status page down

**Symptom:** the status page returns 404 or 502, the monitored sites work.

**Check:** `docker compose ps` shows whether `uptime-kuma` is healthy; `docker network inspect monitoring --format '{{range .Containers}}{{.Name}} {{end}}'` must list Caddy and Kuma. A 404 on a path that should be public points to the allowlist in `sites/status.caddy` of vps-edge.

**Fix:** Kuma not healthy → `docker compose logs --tail 50 uptime-kuma`, then `docker compose up -d`. Network missing → start or update vps-edge first, then `docker compose up -d` here.

**Way back:** revert the image update or the allowlist change through a pull request.

### All monitors red

**Symptom:** all monitors report down at the same moment, but the sites open from outside.

**Check:** Kuma reaches nothing outside: `docker compose exec uptime-kuma getent hosts github.com` fails, or the network `outbound` is missing in `docker compose ps --format '{{.Networks}}'`. On the host, `resolvectl status` shows whether DNS works.

**Fix:** `docker compose up -d --force-recreate` attaches the networks again; a DNS problem of the host is fixed on the host. No change in this repository causes this case, so there is nothing to roll back.

**Way back:** none needed.
