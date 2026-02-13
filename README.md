# Observability Stack

A unified, Docker Compose-based observability stack providing metrics collection, log aggregation, and dashboarding out of the box.

## Components

| Service | Role | Default Port |
|---------|------|-------------|
| **Prometheus** | Metrics collection & storage | `9090` |
| **Grafana** | Visualization & dashboards | `3000` |
| **Loki** | Log aggregation & querying | `3100` |
| **Promtail** | Log shipping (Docker logs → Loki) | — |

## Quick Start

```bash
cp .env.example .env   # adjust ports/passwords as needed
docker compose up -d
```

- Grafana: [http://localhost:3000](http://localhost:3000) (default login: `admin` / value of `GF_SECURITY_ADMIN_PASSWORD`)
- Prometheus: [http://localhost:9090](http://localhost:9090)

Datasources for Prometheus and Loki are auto-provisioned in Grafana.

## Configuration

| File | Purpose |
|------|---------|
| `prometheus/prometheus.yml` | Scrape targets |
| `promtail/config.yml` | Log discovery & pipeline |
| `grafana/provisioning/datasources/` | Auto-provisioned datasources |
| `.env` | Ports, passwords, tunables |

## Adding Your App

1. Expose a `/metrics` endpoint in your service.
2. Add a scrape job in `prometheus/prometheus.yml` under the `app-metrics` placeholder.
3. `docker compose restart prometheus`

## License

MIT
