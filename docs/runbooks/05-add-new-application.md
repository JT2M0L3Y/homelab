# Runbook: Add New Application

**ID:** RB-005  
**Last Updated:** 2026-11-09  
**Owner:** Project owner  
**Last Tested:** Not yet tested

## Purpose

Onboard a new application onto the platform in a consistent, secure, and observable way.

## When to Use

- Introducing any new workload beyond the portfolio website
- Creating a staging or secondary service
- Migrating an existing external service onto the platform

## Prerequisites

- Access to the `apps` VM (or decision that the new app belongs on a different VM)
- Ability to create OpenBao secrets and policies
- Ability to update Caddy configuration and internal DNS
- GitHub repository (or decision to place the code somewhere)
- Understanding of the application’s runtime needs (ports, volumes, environment, database)

## Procedure

1. **Decide placement**
   - Default: `apps` VM unless there is a strong isolation reason.
   - Record the decision (future ADR or note in design.md).

2. **Prepare secrets**
   - Create the necessary secret paths in OpenBao.
   - Grant the application (or its deploy identity) the minimum required policy.
   - Document the paths in this runbook or the application’s README.

3. **Database (if required)**
   - Create a dedicated database and user in PostgreSQL.
   - Store the credentials in OpenBao.
   - Apply any required schema / migrations.

4. **Containerize and configure**
   - Ensure the application has a Dockerfile (or equivalent).
   - Define how configuration and secrets are injected.
   - Add health-check endpoint if missing.

5. **Networking & exposure**
   - Add internal DNS record(s) if useful (`newapp.internal`).
   - Update Caddy if the application should be publicly reachable.
   - Restrict ports with host firewall / Tailscale ACLs as appropriate.

6. **CI/CD**
   - Add or extend a GitHub Actions workflow following the existing SSH-over-Tailscale pattern.
   - Test a full deploy.

7. **Observability**
   - Ensure logs are shipped to Loki.
   - Expose Prometheus metrics if practical.
   - Add a basic Grafana panel or note the queries.

8. **Documentation**
   - Add a short entry to `docs/design.md` or a new architecture note.
   - Create or update any application-specific runbook steps.
   - Update this runbook’s “Related” section with concrete paths.

## Verification

- [ ] Application deploys cleanly via CI/CD
- [ ] Secrets are fetched from OpenBao (no plaintext in the image or repo)
- [ ] Database connectivity works (if applicable)
- [ ] Public or internal URL responds correctly
- [ ] Logs appear in Loki
- [ ] Metrics appear in Prometheus/Grafana (if instrumented)
- [ ] Restart after reboot works automatically

## Rollback

- Stop and remove the new containers.
- Remove Caddy site blocks and DNS records.
- Drop the database user/database if it was created solely for this app.
- Delete or disable the OpenBao policy and secrets.
- Revert CI/CD workflow changes.

## Related

- RB-001 Deploy Service
- RB-002 Rotate Secrets
- `docs/design.md` §3 and §6
- Existing Caddy and OpenBao configuration locations (fill in once known)
