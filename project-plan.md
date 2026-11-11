# Self-Hosted Developer Platform

## Project Overview

### Objective

Build a self-hosted developer platform on a Debian/Proxmox host that provides
core services and a quick, disposable environment for developing, testing, and
running projects.

### Primary Application

The portfolio website proves the platform end-to-end.

- **Current state:** Static site served by a Cloudflare Worker.
- **Target state:** Dynamic application backed by the platform PostgreSQL
  service and published through Cloudflare Tunnel and Caddy.
- **Placement:** A Proxmox-managed project environment, separate from the
  host-level core services.

---

## Success Criteria

I should be able to demonstrate:

- Secure remote access using NetBird, with no public SSH
- Core services running as Docker containers on the Debian host OS
- Internal DNS through AdGuard Home
- Internal routing and HTTPS termination through Caddy
- Public access through Cloudflare Tunnel
- Proxmox environments that can be quickly created for projects, development,
  and testing
- A Docker-based portfolio deployment in a Proxmox project environment
- CI/CD automation over NetBird and the Proxmox API/management path
- Metrics collection (Prometheus)
- Centralized logging (Loki)
- Documentation and operational runbooks

---

## Target Architecture

```text
Local LAN (vmbr0) ── Debian / Proxmox host (`<LAN_HOST_IP>/<LAN_PREFIX>`)
                       ├── Host-level Docker core services
                       │   ├── AdGuard Home
                       │   ├── OpenBao
                       │   ├── Caddy
                       │   ├── PostgreSQL
                       │   ├── Prometheus
                       │   ├── Grafana
                       │   ├── Loki
                       │   └── Cloudflare Tunnel
                       ├── NetBird peer / subnet router
                       └── Proxmox
                           └── vmbr1 project subnet
                               ├── Portfolio project environment
                               ├── Development environments
                               └── Short-lived test environments

NetBird hosted management coordinates authorized peers; the Debian host
advertises the project subnet for private routing. AdGuard handles internal
DNS; Caddy handles internal routing and public HTTPS ingress from Cloudflare
Tunnel.

GitHub Actions ── NetBird ── Debian/Proxmox host and project environments
```

Core service containers run directly on the Debian host OS, not inside
Proxmox-managed guests. Proxmox remains the fast provisioning layer for
isolated project workloads. Docker is the initial container runtime; project
Docker workloads should run in Proxmox VMs rather than nesting Docker in LXCs.

Detailed diagrams and the full technical design live in `docs/`.

---

## MVP Scope

### Infrastructure

#### Debian / Proxmox

**Responsibilities:** Run core Docker services on the host OS and provide
Proxmox-managed environments for projects, development, and testing.

**Deliverables:**
- Stable Debian/Proxmox host
- Dedicated `vmbr1` project subnet, isolated from the local LAN
- Host `nftables` policy and outbound NAT for project environments
- Reusable Proxmox templates and documented quick-provisioning workflow
- Proxmox backups for project environments
- Protected backups for host Docker configuration, persistent volumes, and data

#### Docker on the Debian host

Run the platform/core services as Docker containers on the host OS:

- AdGuard Home
- OpenBao
- Caddy
- PostgreSQL
- Prometheus
- Grafana
- Loki
- Cloudflare Tunnel

Use persistent storage and a documented startup/backup procedure. These
containers are separate from Proxmox guests.

### Remote Access

#### NetBird Community Edition

**Responsibilities:** Private remote administration and access to the project
subnet.

**Requirements:**
- Use NetBird-hosted management/control plane
- Install the NetBird peer on the Debian host
- Advertise the `vmbr1` project subnet through the host
- Authorize remote peers and restrict routes/access with NetBird policies
- SSH and management access remain private; no public SSH

### Internal DNS

#### AdGuard Home

**Responsibilities:** Internal DNS and service discovery for host services
and project environments.

Example records:
```text
openbao.internal
grafana.internal
prometheus.internal
app.internal / www.internal
db.internal
```

Local LAN clients may query AdGuard Home only on TCP/UDP port 53. Other
local-LAN-initiated access to host services and the project subnet is blocked.

### Reverse Proxy and Public Access

#### Caddy

**Responsibilities:** Internal reverse proxy/routing and HTTPS termination.
Public traffic arrives through Cloudflare Tunnel; no public host ports are
opened.

#### Cloudflare Tunnel

Runs as a Docker container on the Debian host and forwards public requests to
Caddy. Caddy routes to the portfolio application in its Proxmox project
environment.

### Secrets Management

#### OpenBao

**Responsibilities:** Secret storage, credential management, and secure
application configuration. OpenBao is Vault-compatible and open source.

Examples include database credentials, Cloudflare credentials, and deployment
credentials. The portfolio application retrieves its secrets from OpenBao at
runtime.

### Database

#### PostgreSQL

**Responsibilities:** Persistent storage for the portfolio and other projects.

**Requirements:**
- One PostgreSQL Docker service on the Debian host
- Multiple databases, schemas, and roles as needed
- `pg_dump` backups and a documented restore procedure

### Observability

#### Prometheus
- Host, Docker service, Proxmox guest, and application metrics

#### Grafana
- Infrastructure, project environment, and portfolio dashboards

#### Loki
- Centralized logs from host services, Docker workloads, and project
  environments

---

## Application

### Portfolio Website

**Migration path:**
1. Keep the Cloudflare Worker live during development.
2. Create a Proxmox project environment and deploy the dynamic application as
   a Docker workload.
3. Connect it to host PostgreSQL and OpenBao over the private network.
4. Route public traffic from Cloudflare Tunnel through Caddy to the project
   environment.
5. Document cut-over and rollback.

---

## CI/CD Pipeline

### GitHub Actions Workflow

```text
Push / PR → Lint → Test → Build Docker image
         → Authenticate/connect over NetBird
         → Deploy to the project environment
         → Health check
```

**Primary path:** GitHub Actions joins the NetBird network and reaches the
Proxmox host and authorized project subnet. Use the Proxmox API for environment
lifecycle and SSH/appropriate deployment tooling for application rollout.

**Development:** Use the local workstation over NetBird and spin up isolated
Proxmox environments as needed.

---

## Schedule

### Week 1: Foundation

- [ ] Install/configure Debian and Proxmox host
- [ ] Configure `vmbr1`, project subnet, host firewall, and outbound NAT
- [ ] Configure NetBird hosted management, host peer, subnet route, and access
      policies
- [ ] Configure host SSH (key-only, no public exposure)
- [ ] Create reusable Proxmox project VM template(s)
- [ ] Verify authorized remote access and local-LAN isolation

**Week 1 Deliverable:** Working host, private access, and quick project
environment provisioning.

### Week 2: Core Platform Services

- [ ] Set up Docker on the Debian host OS
- [ ] Deploy AdGuard Home and configure internal records
- [ ] Deploy OpenBao and configure secret storage/recovery
- [ ] Deploy PostgreSQL; configure multiple databases/roles and backups
- [ ] Deploy Caddy for internal routes and HTTPS
- [ ] Deploy Prometheus, Grafana, and Loki
- [ ] Deploy Cloudflare Tunnel to Caddy; verify no public host ports
- [ ] Configure persistent host data and backup/restore procedures

**Week 2 Deliverable:** Operational Docker-based core services.

### Week 3: Application and CI/CD

- [ ] Configure GitHub Actions workflow and NetBird access
- [ ] Create a Proxmox project environment for the portfolio
- [ ] Build and deploy the portfolio as a Docker workload
- [ ] Connect to PostgreSQL and OpenBao
- [ ] Route public traffic through Cloudflare Tunnel and Caddy
- [ ] Add application metrics and logs
- [ ] Document Cloudflare Worker cut-over and rollback

**Week 3 Deliverable:** Working portfolio deployment through the platform.

### Week 4: Documentation and Presentation

- [ ] Finalize high-level, networking, deployment, and observability diagrams
- [ ] Keep `docs/design.md` aligned with implementation decisions
- [ ] Complete deploy, secrets, backup/restore, project-environment recovery,
      application onboarding, and portfolio cut-over runbooks
- [ ] Prepare the project demonstration

**Week 4 Deliverable:** Publicly presentable engineering project.

---

## Explicitly Out of Scope (v2)

- Kubernetes / Nomad / Consul / service mesh
- Full Terraform / declarative IaC
- High availability or multi-node clusters
- Distributed databases
- OpenTelemetry distributed tracing
- Self-hosted GitHub runner as the primary deploy path

---

## Definition of Done

- [ ] Core services run reliably as Docker containers on the Debian host OS
- [ ] NetBird provides private access to the host and authorized project subnet
- [ ] Local LAN access is limited to the DNS exception
- [ ] Proxmox project environments can be quickly provisioned and recovered
- [ ] Cloudflare Tunnel publishes the portfolio through Caddy without public
      host ports
- [ ] Portfolio is functional, uses PostgreSQL, and retrieves secrets from
      OpenBao
- [ ] Metrics and logs appear in Grafana/Loki
- [ ] CI/CD deploys over NetBird
- [ ] Core service data and Proxmox project environments have documented
      backups and restore procedures
- [ ] Design docs, diagrams, and runbooks match the implementation

---

## Related Documents

- `docs/design.md` — Technical Design Document
- `docs/architecture/` — Mermaid diagrams
- `docs/runbooks/` — Operational procedures
