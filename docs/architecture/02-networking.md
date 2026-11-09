# Networking & Traffic Flows

## Overlay and Trust Zones

```mermaid
flowchart LR
    subgraph Untrusted["Untrusted"]
        Internet((Internet))
        PublicClient[Public Clients]
    end

    subgraph Edge["Edge"]
        CF[Cloudflare DNS]
    end

    subgraph Trusted["Trusted - Tailscale"]
        TS[Tailscale Network]
        PC[platform-core]
        APPS[apps]
        DEV[dev]
    end

    PublicClient --> Internet
    Internet --> CF
    CF -->|Public HTTPS| APPS
    CF -.->|Optional tunnel / DNS only| TS

    TS --- PC
    TS --- APPS
    TS --- DEV
```

## Internal DNS & Service Discovery

```mermaid
flowchart TB
    Client[Any node on Tailscale]
    AGH[AdGuard Home<br/>*.internal]
    
    Client -->|DNS query| AGH
    AGH -->|openbao.internal| OpenBao[OpenBao]
    AGH -->|grafana.internal| Grafana[Grafana]
    AGH -->|prometheus.internal| Prom[Prometheus]
    AGH -->|app.internal / www.internal| App[Portfolio Site via Caddy]
    AGH -->|db.internal| PG[(PostgreSQL)]
```

## Public Request Path (Portfolio Site)

```mermaid
sequenceDiagram
    participant User as Public User
    participant CF as Cloudflare DNS
    participant Caddy as Caddy (apps VM)
    participant App as Portfolio Website
    participant OpenBao as OpenBao (platform-core)
    participant DB as PostgreSQL

    User->>CF: Resolve jonathansmoley.com
    CF-->>User: Public IP / route
    User->>Caddy: HTTPS request
    Caddy->>App: Proxy (internal)
    App->>OpenBao: Fetch secrets (if needed)
    App->>DB: Query / write
    DB-->>App: Result
    App-->>Caddy: Response
    Caddy-->>User: HTTPS response
```

## Management / SSH Path

```mermaid
sequenceDiagram
    participant Admin as Operator
    participant TS as Tailscale
    participant VM as Target VM

    Admin->>TS: Connect (authenticated)
    TS->>VM: Encrypted tunnel
    Admin->>VM: SSH (key-based, no public port)
```

## Notes

- No SSH (or other admin ports) are exposed to the public internet.
- Tailscale ACLs can be tightened later to further restrict which nodes can reach which services.
- Internal DNS removes the need to hard-code IP addresses in configuration.
- The portfolio site (`jonathansmoley.com`) is the primary public-facing application.
