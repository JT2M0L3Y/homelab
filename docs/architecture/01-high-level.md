# High-Level Architecture

```mermaid
flowchart TB
    Internet((Internet))
    Cloudflare[Cloudflare]
    NetBirdMgmt[NetBird hosted management]
    Remote[Authorized remote peers]
    GH[GitHub]
    GHA[GitHub Actions]

    subgraph Host["Debian / Proxmox Host · LAN_HOST_IP/LAN_PREFIX"]
        Peer[NetBird peer / subnet router]
        subgraph Core["Host-level Docker services"]
            AdGuard[AdGuard Home]
            OpenBao[OpenBao]
            Caddy[Caddy]
            PostgreSQL[(PostgreSQL)]
            Prometheus[Prometheus]
            Grafana[Grafana]
            Loki[Loki]
            Tunnel[Cloudflare Tunnel]
        end
        Proxmox[Proxmox]
        subgraph Projects["vmbr1 · project environments"]
            Portfolio[Portfolio Docker workload]
            Dev[Development environments]
            Test[Short-lived test environments]
        end
    end

    NetBirdMgmt -. peer coordination .-> Peer
    NetBirdMgmt -. peer coordination .-> Remote
    Remote -->|Authorized overlay route| Peer
    Peer --> Projects
    GH --> GHA
    GHA -->|NetBird / Proxmox API / SSH| Host
    Internet --> Cloudflare
    Cloudflare -->|Tunnel ingress| Tunnel
    Tunnel --> Caddy
    Caddy --> Portfolio
    Portfolio --> PostgreSQL
    Portfolio -.->|Fetch secrets| OpenBao
    Prometheus --> Core
    Prometheus --> Projects
    Prometheus --> Grafana
    Loki --> Grafana
```

## Notes

- Core services run as Docker containers on the Debian host OS, separate from
  Proxmox-managed guests.
- NetBird uses hosted management. The Debian host runs the peer and advertises
  the authorized project subnet; remote peers access only what policy permits.
- Proxmox is the quick-provisioning environment for project, development, and
  test workloads. The portfolio is a project workload.
- Use Proxmox VMs for Docker-based project workloads; do not nest Docker inside
  an LXC.
- AdGuard Home provides internal DNS. Caddy handles internal routing and
  receives public requests from the host's Cloudflare Tunnel.
- The host firewall blocks local-LAN-initiated access to platform services and
  the project subnet, except DNS to AdGuard Home on TCP/UDP 53.
- No public ports are opened on the host.
