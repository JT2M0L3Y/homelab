# Observability Data Flow

## Metrics Path

```mermaid
flowchart LR
    subgraph Sources["Metric Sources"]
        NodePC[platform-core<br/>node_exporter]
        NodeApps[apps<br/>node_exporter]
        NodeDev[dev<br/>node_exporter]
        AppMetrics[Portfolio Site<br/>/metrics]
        CaddyMetrics[Caddy metrics]
    end

    Prom[Prometheus<br/>platform-core]
    Grafana[Grafana]

    NodePC --> Prom
    NodeApps --> Prom
    NodeDev --> Prom
    AppMetrics --> Prom
    CaddyMetrics --> Prom

    Prom --> Grafana
```

## Logging Path

```mermaid
flowchart LR
    subgraph LogSources["Log Sources"]
        AppLogs[Portfolio Website]
        CaddyLogs[Caddy]
        SystemLogs[systemd / journal]
        OpenBaoLogs[OpenBao]
    end

    Promtail[Promtail or<br/>equivalent agents]
    Loki[Loki<br/>platform-core]
    Grafana[Grafana]

    AppLogs --> Promtail
    CaddyLogs --> Promtail
    SystemLogs --> Promtail
    OpenBaoLogs --> Promtail

    Promtail --> Loki
    Loki --> Grafana
```

## End-to-End Observability View

```mermaid
flowchart TB
    subgraph Collect["Collection"]
        Exporters[Node + App Exporters]
        Agents[Log Agents]
    end

    subgraph Store["Storage"]
        Prom[Prometheus]
        Loki[Loki]
    end

    subgraph Present["Presentation"]
        Grafana[Grafana Dashboards]
    end

    Exporters --> Prom
    Agents --> Loki
    Prom --> Grafana
    Loki --> Grafana
```

## Required Dashboards (MVP)

1. **Infrastructure Health** – host + all VMs (CPU, memory, disk, network)
2. **VM / Node Detail** – per-VM drill-down
3. **Application** – request rate, latency, errors, saturation for the portfolio site

## Notes

- All observability components live on `platform-core` to keep the data plane separate from the control/observability plane where practical.
- Grafana is the single pane of glass for both metrics and logs in the MVP.
- The portfolio site should expose Prometheus metrics and structured logs from the first dynamic version.
