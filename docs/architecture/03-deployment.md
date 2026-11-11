# Deployment & CI/CD Flow

## Pipeline Overview

```mermaid
flowchart LR
    Dev[Developer] -->|git push| GH[GitHub]
    GH --> GHA[GitHub Actions]

    subgraph Pipeline["GitHub Actions Workflow"]
        Lint[Lint]
        Test[Test]
        Build[Build Docker image]
        Connect[Authenticate to NetBird]
        Provision[Provision / select Proxmox project VM]
        Deploy[Deploy application]
        Verify[Health check]
    end

    GHA --> Lint --> Test --> Build --> Connect --> Provision --> Deploy --> Verify
    Connect -->|Private network| NB[NetBird hosted network]
    NB --> Host[Debian / Proxmox Host]
    Provision -->|Scoped API| PVE[Proxmox API]
    Deploy -->|SSH / deployment mechanism| VM[Project VM]
```

## Portfolio Deployment Sequence

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant GHA as GitHub Actions
    participant NB as NetBird hosted network
    participant Host as Debian / Proxmox Host
    participant PVE as Proxmox
    participant VM as Portfolio project VM
    participant Caddy as Host Caddy
    participant App as Portfolio Docker workload

    Dev->>GH: git push
    GH->>GHA: Trigger workflow
    GHA->>GHA: Lint, test, build image
    GHA->>NB: Authenticate as authorized peer
    GHA->>Host: Connect over private network
    GHA->>PVE: Provision/select project VM (scoped API)
    GHA->>VM: Deploy Docker workload over private route
    VM->>App: Start updated workload
    App->>Caddy: Ready for internal proxy traffic
    GHA->>VM: Health check
    GHA-->>GH: Workflow success / failure
```

## Notes

- Core services are Docker containers on the Debian host OS; project workloads
  run in Proxmox-managed environments.
- Use VMs for Docker-based project workloads rather than nesting Docker inside
  LXCs.
- GitHub Actions uses NetBird for private connectivity and narrowly scoped
  Proxmox/deployment credentials.
- Proxmox provides quick provisioning for projects, development, and tests;
  persistent project data must be backed up separately from disposable
  environments.
- The portfolio cut-over from its Cloudflare Worker has a separate rollback
  procedure.
