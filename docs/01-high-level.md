# High-Level Architecture

```mermaid
flowchart TB
    Internet((Internet))
    CF[Cloudflare DNS]
    TS[Tailscale Overlay Network]

    subgraph Host["Proxmox Host"]
        direction TB

        subgraph Core["VM: platform-core"]
            OpenBao[OpenBao]
            Grafana[Grafana]
            Prom[Prometheus]
            Loki[Loki]
            AdGuard[AdGuard Home<br/>Internal DNS]
        end

        subgraph Apps["VM: apps"]
            Caddy[Caddy<br/>Reverse Proxy + HTTPS]
            PG[(PostgreSQL)]
            App[Portfolio Website<br/>jonathansmoley.com]
        end

        subgraph Dev["VM: dev"]
            Workstation[Developer Workstation]
        end
    end

    GH[GitHub Repository]
    GHA[GitHub Actions]

    Internet --> CF
    CF --> TS
    TS --> Host

    GH --> GHA
    GHA -->|Deploy via Tailscale SSH| Apps

    Caddy --> App
    App --> PG
    App -.->|Fetch secrets| OpenBao
    Prom -->|Scrape| Core
    Prom -->|Scrape| Apps
    Loki -->|Ingest logs| Core
    Loki -->|Ingest logs| Apps
```

## Notes

- `platform-core` hosts the control-plane and observability stack (OpenBao, Grafana, Prometheus, Loki, AdGuard Home).
- `apps` hosts the reverse proxy, database, and the portfolio website.
- `dev` is a full developer workstation joined to the same Tailscale network.
- All management traffic (SSH, OpenBao UI, Grafana, etc.) travels over Tailscale.
- Public HTTPS traffic for the portfolio site is terminated by Caddy after DNS resolution via Cloudflare.
- The portfolio site is the primary workload that proves the platform; it will evolve from the current static Cloudflare Worker into a dynamic application backed by PostgreSQL and additional APIs.
