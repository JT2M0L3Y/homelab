# Self-Hosted Developer Platform (30-Day MVP)

## Project Overview

### Objective

Build a self-hosted developer platform on Proxmox that provides a production-like environment for developing, deploying, monitoring, and operating software projects.

The platform serves two purposes:

1. Enable rapid development of future personal projects.
2. Demonstrate platform engineering, infrastructure, and software delivery skills to employers.

### Project Narrative

Instead of:

> "I built a homelab."

The goal is to demonstrate:

> "I designed and operated a secure self-hosted developer platform that supports application development, deployment, observability, secrets management, and remote administration."

### Primary Application

The workload that proves the platform is the portfolio website at [https://jonathansmoley.com](https://jonathansmoley.com).

- **Current state:** Static site served by a Cloudflare Worker.
- **Target state:** Self-hosted, dynamic application that uses the platform’s PostgreSQL instance and can call additional internal/external APIs.

The platform itself remains the primary project; the portfolio site is the concrete proof that the platform works end-to-end.

---

## Success Criteria

At the end of 30 days, I should be able to demonstrate:

- Secure remote access (Tailscale, no public SSH)
- Centralized secrets management (OpenBao)
- Internal DNS (AdGuard Home)
- Reverse proxy with HTTPS (Caddy)
- Containerized deployment of the portfolio website
- CI/CD automation (GitHub Actions → SSH over Tailscale)
- Metrics collection (Prometheus)
- Centralized logging (Loki)
- Documentation and operational runbooks

---

## Final Architecture

```text
Internet
    |
Cloudflare DNS
    |
    V
Tailscale
    |
+-----------------------+
|   Proxmox Host        |
+-----------------------+

  VM: platform-core
  ----------------
  OpenBao
  Grafana
  Prometheus
  Loki
  AdGuard Home

  VM: apps
  --------
  Caddy
  PostgreSQL
  Portfolio Website (jonathansmoley.com)

  VM: dev
  -------
  Developer Workstation

GitHub
   |
   V
GitHub Actions
   |
   V
Deploy via SSH over Tailscale → apps VM
```

Detailed diagrams and the full technical design live in `docs/`.

---

## MVP Scope

### Infrastructure

#### Proxmox

**Responsibilities:** Virtualization, VM management, snapshot management, backup management.

**Deliverables:**
- Stable Proxmox installation
- VM templates
- Backup schedule
- Snapshot strategy

---

### Remote Access

#### Tailscale

**Responsibilities:** Remote administration, secure networking, private access.

**Requirements:**
- Installed on every VM (and preferably the host)
- SSH access via Tailscale only
- No public SSH exposure

---

### Internal DNS

#### AdGuard Home

**Responsibilities:** Internal DNS, service discovery, optional DNS filtering.

**Example records:**
```text
openbao.internal
grafana.internal
prometheus.internal
app.internal / www.internal
db.internal
```

---

### Reverse Proxy

#### Caddy

**Responsibilities:** HTTPS termination, reverse proxying, automatic TLS.

**Primary public service:** `jonathansmoley.com` (and any required subdomains).

---

### Secrets Management

#### OpenBao

**Responsibilities:** Secret storage, credential management, secure application configuration.

OpenBao is Vault-compatible (same API/CLI surface) while remaining fully open-source (MPL-2.0) under Linux Foundation governance.

**Examples of secrets:**
```text
github-token
cloudflare-token
database-password
deploy keys / registry credentials
```

**Demonstration:** Portfolio application retrieves secrets from OpenBao at runtime.

---

### Database

#### PostgreSQL

**Responsibilities:** Persistent storage for the portfolio website.

**Requirements:**
- Dedicated database instance on the `apps` VM
- Backup strategy documented and tested

---

### Observability

#### Prometheus
- Metrics collection (CPU, memory, network, disk, container, and application metrics)

#### Grafana
- Dashboards: Infrastructure Health, VM Metrics, Application Metrics (portfolio site)

#### Loki
- Centralized logging of application and infrastructure logs

---

## Application

### Portfolio Website

**Purpose:** Prove the platform works end-to-end while giving a real personal asset a production-like home.

**Migration path:**
1. Keep current Cloudflare Worker live during development.
2. Build and containerize the dynamic version on the platform.
3. Cut over DNS / traffic once the self-hosted version is stable.
4. Document rollback procedure.

**Capabilities enabled by the platform:**
- Dynamic content backed by PostgreSQL
- Ability to call additional internal or external APIs
- Secure secret injection via OpenBao
- Automated deployment and observability

### Non-Goals (for the application)

Avoid turning the portfolio site into a large product during the 30-day window:
- Heavy authentication systems
- Multi-tenancy
- Complex SPA frameworks beyond what is needed
- AI features

Keep the application scope tight so the platform remains the focus.

---

## CI/CD Pipeline

### Goal

Automate deployments of the portfolio website to the platform.

### GitHub Actions Workflow

```text
Push / PR
    |
Lint
    |
Test
    |
Build Container
    |
Deploy (SSH over Tailscale)
    |
Health check / Restart Service
```

### Deployment Decision (MVP)

**Primary path:** GitHub-hosted runners + SSH over Tailscale.

**Rationale:** Fastest to implement, still demonstrates secure private deployment, no extra runner infrastructure to maintain during the 30-day window.

**Future improvement:** Self-hosted GitHub runner on the platform.

**Deployment target:** `apps` VM.

---

## 30-Day Schedule

### Week 1: Foundation

**Goal:** Establish core infrastructure and network topology.

#### Proxmox
- [ ] Install/configure host
- [ ] Create VM templates
- [ ] Configure storage
- [ ] Configure backups

#### Create VMs
- [ ] platform-core
- [ ] apps
- [ ] dev

#### Tailscale
- [ ] Install on all systems
- [ ] Verify remote access
- [ ] Configure SSH (key-only, no public exposure)

#### DNS
- [ ] Deploy AdGuard Home
- [ ] Configure internal records (`openbao.internal`, etc.)

**Week 1 Deliverable:** Working infrastructure and network topology.

---

### Week 2: Core Platform Services

**Goal:** Deploy foundational platform services.

#### OpenBao
- [ ] Install
- [ ] Configure secret storage
- [ ] Create sample secrets
- [ ] Verify application can retrieve secrets

#### PostgreSQL
- [ ] Install
- [ ] Configure backups
- [ ] Verify connectivity from apps VM

#### Caddy
- [ ] Configure HTTPS
- [ ] Configure reverse proxy for the portfolio domain

#### Prometheus
- [ ] Install
- [ ] Add node exporters on all VMs

#### Grafana
- [ ] Install
- [ ] Create initial dashboards

#### Loki
- [ ] Install
- [ ] Aggregate logs from platform services

**Week 2 Deliverable:** Operational platform services.

---

### Week 3: Application and CI/CD

**Goal:** Complete the software delivery workflow for the portfolio site.

#### GitHub Repository
- [ ] Ensure repository is ready (or create if needed)
- [ ] Configure GitHub Actions workflow

#### Application
- [ ] Evolve portfolio site toward dynamic version
- [ ] Containerize application
- [ ] Connect to PostgreSQL
- [ ] Integrate OpenBao for secrets

#### Deployment
- [ ] Automate deployment via SSH over Tailscale
- [ ] Verify updates land correctly
- [ ] Document cut-over and rollback from Cloudflare Worker

#### Monitoring
- [ ] Add application metrics
- [ ] Add application logging to Loki

**Week 3 Deliverable:** Working portfolio website deployed through CI/CD.

---

### Week 4: Documentation and Presentation

**Goal:** Transform implementation into a polished portfolio project.

#### Architecture Diagrams
- [ ] High-level topology (already drafted in `docs/architecture/`)
- [ ] Networking & traffic flows
- [ ] Deployment flow
- [ ] Observability data flow
- [ ] Final polish / export as needed

#### Technical Design Document
- [x] `docs/design.md` created and aligned with current decisions
- [ ] Keep updated as implementation decisions are finalized

#### Runbooks
Create under `docs/runbooks/`:
- [ ] Deploy Service
- [ ] Rotate Secrets
- [ ] Restore Backup
- [ ] Recover VM
- [ ] Add New Application
- [ ] Portfolio cut-over / rollback

#### Demo Video
**Length:** 5–10 minutes

**Suggested flow:**
1. Architecture overview
2. OpenBao secrets
3. Git push
4. CI/CD pipeline
5. Deployment over Tailscale
6. Grafana dashboard
7. Loki logs
8. Live portfolio site (dynamic + DB)

**Week 4 Deliverable:** Publicly presentable engineering portfolio project.

---

## Explicitly Out of Scope (v2)

The following are postponed until Version 2:

- Kubernetes / Nomad
- Consul / Service Mesh
- ArgoCD
- Full Terraform / declarative IaC
- OpenTelemetry distributed tracing
- Distributed databases
- Multi-node clusters / High Availability
- AI agents
- Microservice architecture beyond the single portfolio application
- Self-hosted GitHub runner as the primary deploy path

**Reason:** The objective is a shippable, demonstrable project within 30 days.

---

## Resume Outcomes

**Resume Bullet #1**  
Designed and operated a self-hosted developer platform on Proxmox supporting secure remote access, secrets management (OpenBao), automated deployments, centralized logging, metrics collection, and HTTPS service delivery.

**Resume Bullet #2**  
Built CI/CD pipelines using GitHub Actions that deployed a containerized portfolio application to a production-like environment utilizing Tailscale, OpenBao, Caddy, PostgreSQL, Grafana, Prometheus, and Loki.

**Resume Bullet #3**  
Implemented observability and operational tooling including centralized logging, infrastructure monitoring, backup strategies, deployment runbooks, and service recovery procedures.

---

## Definition of Done

The project is considered complete when:

- [ ] Infrastructure is stable
- [ ] Services run continuously
- [ ] Secrets are stored in and retrieved from OpenBao
- [ ] HTTPS is operational via Caddy
- [ ] Metrics appear in Grafana
- [ ] Logs appear in Loki
- [ ] GitHub Actions deploys the portfolio website automatically via SSH over Tailscale
- [ ] Portfolio website is reachable, functional, and using PostgreSQL
- [ ] Documentation is finished (`docs/design.md` + runbooks + diagrams)
- [ ] Architecture diagrams are completed
- [ ] Demo video is recorded

At that point the project shifts from “building infrastructure” to “demonstrating engineering capability.”

---

## Related Documents

- `docs/design.md` — Technical Design Document (source of truth for architecture decisions)
- `docs/architecture/` — Mermaid diagrams
- `docs/runbooks/` — Operational procedures (to be populated)
- `docs/decisions/` — Architecture Decision Records (to be populated)
- `docs/demo/` — Demo video script
