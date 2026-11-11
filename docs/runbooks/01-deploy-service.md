# Runbook: Deploy Service

**ID:** RB-001  
**Last Updated:** 2026-11-11
**Owner:** Platform operator
**Last Tested:** Not yet tested

## Purpose

Deploy or update an application in its Proxmox project environment (primarily
the portfolio Docker workload) via CI/CD or manually when necessary. Core
platform services are updated separately on the Debian host with their Docker
configuration.

## When to Use

- Normal continuous deployment after a merge to the main branch
- Manual redeploy after a failed automated run
- Rolling back to a previous known-good application version
- First-time deployment of a new service version

## Prerequisites

- NetBird connectivity to the Debian host and, for project workloads, the
  authorized project environment
- SSH key access to the host/project VM or authorized Proxmox API credentials
- Docker image available from its registry or another approved source
- OpenBao (or GitHub Secrets) containing any required runtime secrets
- Caddy configuration already present on the host for the target hostname
  (for public services)
- For project workloads, a VM exists on the Proxmox project network

## Procedure

### Automated path (preferred)

1. Push or merge the desired commit to the main branch of the application repository.
2. Confirm the GitHub Actions workflow starts.
3. Monitor the workflow:
   - Lint and tests pass
   - Docker image builds successfully
   - Deploy step connects through NetBird and updates the project workload
4. Wait for the health-check step to succeed.

### Manual path (break-glass)

1. Select the target:
   - **Core service:** connect over NetBird to the Debian host.
   - **Project workload:** connect over NetBird to its authorized project VM.
2. Connect with SSH:
   ```bash
   ssh <deploy-user>@<target-netbird-address>
   ```
3. Pull or load the desired Docker image.
4. Update or restart the service using its Docker Compose configuration on the
   host or project VM, as appropriate.
5. Confirm the service is healthy.

## Verification

- [ ] Target Docker service is running the expected image version
- [ ] Application health endpoint returns success (or site loads)
- [ ] Caddy is routing correctly (HTTPS certificate valid)
- [ ] Recent logs appear in Loki
- [ ] Metrics are being scraped (if instrumented)

## Rollback

1. Identify the previous known-good image version from Git history or
   deployment notes.
2. Redeploy that tag using either the automated workflow (revert commit) or the manual path above.
3. Re-run verification steps.
4. If database migrations were involved, consult the database rollback notes before proceeding.

## Related

- `docs/design.md` §6 Deployment Strategy
- [Deployment & CI/CD flow](../architecture/03-deployment.md)
- OpenBao paths used by the service (document concrete paths here once created)
- Grafana application dashboard
- RB-006 (Portfolio Cut-over / Rollback) for DNS/traffic changes
