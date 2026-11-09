# Runbook: Rotate Secrets

**ID:** RB-002  
**Last Updated:** 2026-11-09  
**Owner:** Project owner  
**Last Tested:** Not yet tested

## Purpose

Safely rotate secrets stored in OpenBao and ensure dependent services pick up the new values without prolonged downtime.

## When to Use

- Scheduled rotation (e.g., database passwords, API tokens)
- Suspected or confirmed credential compromise
- After an employee / access change (even for a single-operator system, good hygiene)
- Before decommissioning an old secret

## Prerequisites

- Access to OpenBao (UI or CLI) via Tailscale
- Appropriate OpenBao policy / token to read and write the target secret paths
- Ability to restart or signal the dependent services (or trigger a redeploy)
- Knowledge of which services consume the secret

## Procedure

1. **Identify the secret and its consumers**
   - Note the OpenBao path (e.g., `secret/data/portfolio/db`)
   - List every service or job that reads it

2. **Generate the new secret value**
   - Prefer OpenBao’s dynamic secrets or transit engine where applicable
   - For static secrets, generate a strong value offline or via a password manager

3. **Write the new value to OpenBao**
   - Update the secret at the existing path (or write to a versioned path if using KV v2 carefully)
   - Confirm the write succeeded

4. **Propagate the new value**
   - Preferred: services that fetch secrets at startup → restart / redeploy them
   - If a service caches secrets, force a refresh or restart
   - For database password rotation:
     a. Create the new DB role/user or alter the password
     b. Update OpenBao
     c. Restart the application
     d. Verify connectivity
     e. Drop or disable the old credential

5. **Revoke or delete the old secret** (once all consumers are confirmed on the new value)

## Verification

- [ ] New secret is present and readable in OpenBao
- [ ] Dependent services are running and functional
- [ ] Application can still authenticate to PostgreSQL / external APIs
- [ ] Old secret no longer works (where applicable)
- [ ] Audit log in OpenBao shows the rotation events

## Rollback

1. Restore the previous secret value in OpenBao (KV v2 versioning makes this straightforward).
2. Restart / redeploy dependent services so they pick up the old value.
3. If a database password was changed, revert the database credential as well.
4. Re-run verification.

## Related

- OpenBao documentation / policies (add concrete paths once defined)
- RB-001 Deploy Service (for restarting applications)
- PostgreSQL backup runbook if rotation involves schema or user changes
- `docs/design.md` §5 Security Model
