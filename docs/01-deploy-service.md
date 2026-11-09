# Runbook: Deploy Service

**ID:** RB-001  
**Last Updated:** 2026-11-09  
**Owner:** Project owner  
**Last Tested:** Not yet tested

## Purpose

Deploy or update a containerized service (primarily the portfolio website) onto the `apps` VM via the CI/CD pipeline or manually when necessary.

## When to Use

- Normal continuous deployment after a merge to the main branch
- Manual redeploy after a failed automated run
- Rolling back to a previous known-good image tag
- First-time deployment of a new service version

## Prerequisites

- Tailscale connectivity to the `apps` VM
- SSH key access to the `apps` VM (deploy user or equivalent)
- Container image available (registry or pre-loaded)
- OpenBao (or GitHub Secrets) containing any required runtime secrets
- Caddy configuration already present for the target hostname (for public services)

## Procedure

### Automated path (preferred)

1. Push or merge the desired commit to the main branch of the application repository.
2. Confirm the GitHub Actions workflow starts.
3. Monitor the workflow:
   - Lint and tests pass
   - Image builds successfully
   - Deploy step connects over Tailscale and updates the container
4. Wait for the health-check step to succeed.

### Manual path (break-glass)

1. Connect to the `apps` VM over Tailscale:
   ```bash
   ssh <deploy-user>@apps.<tailscale-name>
   ```
2. Pull or load the desired image tag.
3. Update the running container (docker compose up -d, or equivalent systemd/podman command).
4. Confirm the container is healthy.

## Verification

- [ ] Container is running and has the expected image tag / digest
- [ ] Application health endpoint returns success (or site loads)
- [ ] Caddy is routing correctly (HTTPS certificate valid)
- [ ] Recent logs appear in Loki
- [ ] Metrics are being scraped (if instrumented)

## Rollback

1. Identify the previous known-good image tag (from Git history, container labels, or deployment notes).
2. Redeploy that tag using either the automated workflow (revert commit) or the manual path above.
3. Re-run verification steps.
4. If database migrations were involved, consult the database rollback notes before proceeding.

## Related

- `docs/design.md` §6 Deployment Strategy
- `docs/architecture/03-deployment.md`
- OpenBao paths used by the service (document concrete paths here once created)
- Grafana application dashboard
- RB-006 (Portfolio Cut-over / Rollback) for DNS/traffic changes
