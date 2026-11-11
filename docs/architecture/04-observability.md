# Observability Data Flow

## Metrics Path

```mermaid
flowchart LR
    subgraph Sources["Metric Sources"]
        HostMetrics[Debian host metrics]
        DockerMetrics[Core Docker service metrics]
        GuestMetrics[Proxmox project VM metrics]
        AppMetrics[Portfolio application metrics]
        CaddyMetrics[Host Caddy metrics]
    end

    Prom[Host Docker<br/>Prometheus]
    Grafana[Host Docker<br/>Grafana]

    HostMetrics --> Prom
    DockerMetrics --> Prom
    GuestMetrics --> Prom
    AppMetrics --> Prom
    CaddyMetrics --> Prom
    Prom --> Grafana
```

## Logging Path

```mermaid
flowchart LR
    subgraph Sources["Log Sources"]
        HostLogs[Debian host / systemd]
        CoreLogs[Core Docker services]
        ProjectLogs[Project VM / Docker workloads]
    end

    Agent[Log shipping agent]
    Loki[Host Docker<br/>Loki]
    Grafana[Host Docker<br/>Grafana]

    HostLogs --> Agent
    CoreLogs --> Agent
    ProjectLogs --> Agent
    Agent --> Loki
    Loki --> Grafana
```

## End-to-End Observability View

```mermaid
flowchart TB
    subgraph Collect["Collection"]
        Exporters[Host / Docker / project / app metrics]
        Agents[Host and project log agents]
    end

    subgraph Store["Host Docker Services"]
        Prom[Prometheus]
        Loki[Loki]
    end

    Grafana[Grafana]

    Exporters --> Prom
    Agents --> Loki
    Prom --> Grafana
    Loki --> Grafana
```

## Required Dashboards (MVP)

1. **Infrastructure Health** – Debian host, Docker core services, and Proxmox
   project environments
2. **Project Environment Detail** – per-VM and workload drill-down
3. **Application** – portfolio request rate, latency, errors, and saturation

## Notes

- Prometheus, Grafana, and Loki run as Docker containers on the Debian host OS.
- Collect metrics and logs from the host, core Docker services, and project
  workloads.
- Grafana is the single pane of glass for metrics and logs.
- Avoid collecting secrets or sensitive payloads in logs.
