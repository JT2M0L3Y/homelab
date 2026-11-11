# Networking & Traffic Flows

## Interfaces and Trust Zones

```mermaid
flowchart LR
    Internet((Internet))
    CF[Cloudflare]
    LAN[Local LAN<br/>vmbr0]
    NBPeers[Authorized NetBird peers]
    NBMgmt[NetBird hosted management]

    subgraph Host["Debian / Proxmox Host"]
        Firewall[nftables<br/>LAN access default drop]
        NAT[Outbound NAT<br/>vmbr1 → internet]
        Peer[NetBird peer / subnet router]
        Tunnel[Cloudflare Tunnel container]
        Caddy[Caddy container]
        AdGuard[AdGuard Home container]

        subgraph Core["Other host Docker services"]
            Bao[OpenBao]
            PG[PostgreSQL]
            Observability[Prometheus / Grafana / Loki]
        end

        subgraph ProjectNet["vmbr1 · project subnet"]
            Project[Project / portfolio VMs]
            Dev[Development VMs]
            Test[Temporary test environments]
        end
    end

    LAN -->|TCP/UDP 53 only| AdGuard
    LAN --- Firewall
    NBPeers <--> NBMgmt
    NBMgmt -. peer coordination .-> Peer
    NBPeers -->|Authorized overlay route| Peer
    Peer --> ProjectNet
    ProjectNet --> NAT
    NAT --> Internet
    Internet --> CF
    CF -->|Tunnel ingress| Tunnel
    Tunnel --> Caddy
```

- The Debian/Proxmox host uses `vmbr0` for the local LAN. Its address is
  deployment-specific (`<LAN_HOST_IP>/<LAN_PREFIX>`).
- Proxmox guests use the dedicated `vmbr1` project subnet. Its CIDR, gateway,
  and guest addresses are deployment-specific (`<PROJECT_SUBNET>`).
- Host `nftables` defaults to drop for local-LAN-initiated access to host
  services and `vmbr1`. The only exception is TCP/UDP port 53 from the LAN to
  AdGuard Home.
- Host NAT allows project guests to make outbound internet connections for
  package and image downloads; it does not allow LAN-initiated access.
- The Debian host runs a NetBird peer using NetBird-hosted management and
  advertises the project subnet. NetBird policies restrict which peers can
  reach the host and guest addresses.
- No public SSH or other host ports are exposed.

## Internal DNS and Routing

```mermaid
flowchart TB
    LAN[Local LAN client]
    Remote[Authorized NetBird peer]
    Project[Project environment]
    AGH[AdGuard Home<br/>host Docker · *.internal]
    Caddy[Caddy<br/>host Docker · internal routing]

    LAN -->|DNS TCP/UDP 53 only| AGH
    Remote -->|NetBird route + DNS| AGH
    Project -->|DNS| AGH
    AGH -->|openbao.internal| Bao[OpenBao]
    AGH -->|grafana.internal| Grafana[Grafana]
    AGH -->|prometheus.internal| Prom[Prometheus]
    AGH -->|app.internal / www.internal| Caddy
    AGH -->|db.internal| PG[(PostgreSQL)]
    Caddy -->|Internal routes| Project
```

AdGuard Home resolves internal names for local DNS clients, authorized
NetBird peers, and project environments. LAN clients may query DNS but cannot
initiate connections to other host services or project guests. Caddy provides
internal service routing; PostgreSQL should only be reachable by authorized
clients.

## Public Request Path

```mermaid
sequenceDiagram
    participant User as Public User
    participant CF as Cloudflare
    participant Tunnel as Cloudflare Tunnel container
    participant Caddy as Host Caddy container
    participant App as Portfolio project VM
    participant OpenBao as Host OpenBao container
    participant DB as Host PostgreSQL container

    User->>CF: Request PORTFOLIO_DOMAIN
    CF->>Tunnel: Route through established tunnel
    Tunnel->>Caddy: Forward request
    Caddy->>Caddy: Terminate public HTTPS
    Caddy->>App: Proxy over project network
    App->>OpenBao: Fetch secrets (if needed)
    App->>DB: Query / write
    DB-->>App: Result
    App-->>Caddy: Response
    Caddy-->>Tunnel: Response
    Tunnel-->>CF: Return response
    CF-->>User: Public response
```

The Tunnel originates from the host and forwards requests to Caddy. Caddy
routes the portfolio request to its Proxmox project environment. No public
ports are opened on the host.

## Management and Deployment Path

```mermaid
sequenceDiagram
    participant Admin as Operator / GitHub Actions
    participant NB as NetBird hosted management
    participant Host as Debian / Proxmox Host
    participant VM as Project VM

    Admin->>NB: Authenticate / obtain peer configuration
    Admin->>Host: Connect through NetBird overlay / subnet route
    Admin->>Host: SSH or scoped Proxmox API
    Host->>VM: Provision/manage via Proxmox
    Admin->>VM: Deploy over authorized private route
```

## Notes

- NetBird replaces Tailscale for private remote access and subnet routing.
- NetBird management is hosted; no self-hosted NetBird control-plane service is
  planned.
- Local LAN access to platform services is limited to DNS queries to AdGuard
  Home; public access is through Cloudflare Tunnel and Caddy.
