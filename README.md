# Observability Stack

Self-hosted, per-node monitoring stack: Prometheus, Grafana, Loki, and Promtail. Deploys alongside your applications. Grafana is served at `/telemetry` via nginx reverse proxy.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Node                                               │
│                                                     │
│  ┌──────────┐  scrape   ┌────────────┐              │
│  │Prometheus│◄──────────│node-exporter│  host metrics│
│  │  :9090   │◄──┐       └────────────┘              │
│  └────┬─────┘   │       ┌────────────┐              │
│       │         └───────│  cadvisor  │  containers  │
│       │                 └────────────┘              │
│       ▼                                             │
│  ┌─────────┐    ┌──────┐    ┌────────┐              │
│  │ Grafana │◄───│ Loki │◄───│Promtail│              │
│  │  :3001  │    │:3100 │    │  logs  │              │
│  └────┬────┘    └──────┘    └────────┘              │
│       │                                             │
└───────┼─────────────────────────────────────────────┘
        │
   nginx /telemetry  →  Grafana
```

## Quick Start

```bash
# 1. Configure
cp .env.example .env
# Edit .env with your admin password

# 2. Launch
docker compose up -d

# 3. Access
# Direct: http://localhost:3001
# Via nginx: http://yourhost/telemetry (after nginx config)
```

## Adding Application Targets

Edit `prometheus/prometheus.yml` and add a scrape config:

```yaml
- job_name: "my-app"
  metrics_path: /metrics
  static_configs:
    - targets: ["host.docker.internal:8000"]
      labels:
        app: "my-app"
```

Then reload Prometheus:

```bash
docker compose restart prometheus
```

## Nginx Setup

Include the provided snippet in your nginx server block:

```nginx
include /path/to/observability-stack/nginx/telemetry.conf;
```

Or copy the location block from `nginx/telemetry.conf` into your existing config. Reload nginx after.

## Services

| Service        | Port | Purpose              |
|----------------|------|----------------------|
| Prometheus     | 9090 | Metrics storage      |
| Grafana        | 3001 | Dashboards & alerts  |
| Loki           | 3100 | Log aggregation      |
| Promtail       | —    | Log shipping         |
| Node Exporter  | 9100 | Host metrics         |
| cAdvisor       | 8080 | Container metrics    |

## Prometheus Federation (Future)

For multi-node setups, a central Prometheus can federate from each node:

```yaml
# On central Prometheus
- job_name: "federation-node-1"
  honor_labels: true
  metrics_path: /federate
  params:
    match[]:
      - '{__name__=~".+"}'
  static_configs:
    - targets: ["node1:9090"]
```

This enables a central Grafana to query metrics across all nodes while each node retains local observability.
