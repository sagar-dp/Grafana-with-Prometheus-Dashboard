# Grafana + Prometheus Monitoring Stack

## Quick start

```bash
docker compose up -d
```

- Prometheus: http://localhost:9090
- Alertmanager: http://localhost:9093
- Grafana: http://localhost:3000 (login: admin / admin — change on first login)
- cAdvisor UI: http://localhost:8080
- node-exporter metrics: http://localhost:9100/metrics

Grafana auto-provisions the Prometheus data source and loads
`dashboards/host-overview.json` into a "Monitoring" folder — no manual
data source setup needed.

## Before you start

1. Edit `prometheus/prometheus.yml` — replace the `myapp` job's target
   (`myapp:PORT`) with your actual application's host:port, and make sure
   your app exposes a Prometheus `/metrics` endpoint (use a client library
   like `prom-client` for Node.js or `prometheus_client` for Python).
2. Edit `alertmanager/alertmanager.yml` — uncomment and fill in the
   `slack_configs` (webhook URL) or `email_configs` block(s) you want to
   use for notifications.

## Verify the alert rules loaded

```bash
curl http://localhost:9090/api/v1/rules | jq
```

## Fire drill (test alerting end-to-end)

Simulate load to trigger `HighCPUUsage` / `HighMemoryUsage`:

```bash
docker run --rm -it polinux/stress stress --cpu 4 --timeout 300s
```

Or stop the demo app / kill a container to trigger `AppDown` or
`ContainerRestartingFrequently`, then watch:
- Prometheus → Alerts tab (pending → firing)
- Alertmanager UI (http://localhost:9093)
- Your Slack/email channel

## Files

| Path | Purpose |
|---|---|
| `docker-compose.yml` | Full stack: Prometheus, Alertmanager, Grafana, node-exporter, cAdvisor |
| `prometheus/prometheus.yml` | Scrape configs |
| `prometheus/alerts.yml` | Alert rules (CPU, memory, disk, app down, container health) |
| `alertmanager/alertmanager.yml` | Notification routing (Slack/email templates included, commented out) |
| `grafana/provisioning/` | Auto-configures Grafana's data source + dashboard folder |
| `dashboards/host-overview.json` | Pre-built dashboard: CPU, memory, disk, network, container stats, app up/down, active alerts |

## Next steps for interns

- Instrument the target application with a Prometheus client library and expose custom business metrics (request rate, error rate, latency).
- Import community dashboards for deeper detail: Node Exporter Full (ID `1860`) and cAdvisor (ID `893` or `14282`) via Grafana → Dashboards → Import.
- Write a short runbook per alert and link it in the alert annotation (`runbook_url`).
- Practice incident response: trigger an alert, acknowledge it, resolve the underlying issue, confirm the alert auto-resolves.
