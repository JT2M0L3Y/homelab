# Runbook: Portfolio Cut-over / Rollback

**ID:** RB-006  
**Last Updated:** 2026-11-11
**Owner:** Platform operator
**Last Tested:** Not yet tested

## Purpose

Safely move public traffic for the portfolio domain from the current Cloudflare Worker (static) to the self-hosted dynamic version running on the platform, and reverse the change if necessary.

## When to Use

- Initial cut-over once the self-hosted portfolio site is stable
- Emergency rollback if the self-hosted version misbehaves after cut-over
- Planned maintenance that requires temporarily returning traffic to Cloudflare

## Prerequisites

- Self-hosted portfolio site is deployed as a Docker workload in its Proxmox
  project environment, healthy, and routed through host Caddy
- Database migrations (if any) have been applied and verified
- Cloudflare DNS access
- Ability to update Caddy and OpenBao if needed
- Monitoring (Grafana / Loki) is watching the self-hosted instance
- A tested rollback path exists

## Procedure

### A. Cut-over (Cloudflare Worker → Self-hosted)

1. **Final pre-flight checks**
   - [ ] Self-hosted site responds correctly through host Caddy and the
         Cloudflare Tunnel
   - [ ] TLS certificate is valid
   - [ ] Database connectivity and critical pages work
   - [ ] Logs and metrics are flowing
   - [ ] Recent backup of PostgreSQL exists

2. **Lower DNS TTL** (optional but recommended)
   - No DNS origin cut-over is required for tunnel-based ingress. If Cloudflare
     DNS records or Worker routes need to change, temporarily lower their TTL
     where applicable.

3. **Update DNS**
   - Configure Cloudflare Tunnel to forward `<PORTFOLIO_DOMAIN>` (and its
     `www` hostname if used) to host Caddy.
   - Ensure Caddy proxies to the portfolio project environment. Do not expose
     public ports on the host.

4. **Monitor**
   - Watch Grafana, Loki, and real-user checks for errors, elevated latency, or 5xx responses.
   - Keep the Cloudflare Worker intact (do not delete it yet).

5. **Stabilize**
   - Once traffic looks healthy for an agreed period (e.g., a few hours or a day), raise TTL back to normal.
   - Document the exact time of cut-over and the commit/image that is live.

### B. Rollback (Self-hosted → Cloudflare Worker)

1. **Decide to roll back** (error budget, data issues, or major regression).
2. **Revert DNS**
   - Point the domain back to the Cloudflare Worker.
3. **Confirm**
   - Public traffic is again served by the Worker.
   - Self-hosted instance can remain running for debugging or be scaled down.
4. **Post-incident**
   - Capture logs and metrics from the failed period.
   - Fix the underlying issue before attempting cut-over again.

## Verification (after cut-over)

- [ ] `https://<PORTFOLIO_DOMAIN>` resolves and serves the self-hosted version
- [ ] Certificate is valid and trusted
- [ ] Key dynamic features (DB-backed pages, any APIs) function
- [ ] No spike in 5xx or unusual error rates in Loki/Grafana
- [ ] Cloudflare analytics (if still proxying) show expected traffic

## Verification (after rollback)

- [ ] Domain again serves the static Cloudflare Worker
- [ ] Self-hosted site is no longer receiving public traffic (or is intentionally still reachable for debugging)

## Related

- RB-001 Deploy Service
- RB-003 Restore Backup (if data issues are involved)
- Cloudflare DNS dashboard
- Caddy configuration for the portfolio site
- `docs/design.md` §1 (Primary Application) and §6
- Current Cloudflare Worker project
