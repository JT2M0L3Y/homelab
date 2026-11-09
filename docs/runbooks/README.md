# Operational Runbooks

This directory contains the operational procedures for the self-hosted developer platform.

## Standard Runbook Structure

Every runbook follows the same template so procedures are predictable and easy to execute under pressure.

```markdown
# Runbook: <Short Title>

**ID:** RB-XXX  
**Last Updated:** YYYY-MM-DD  
**Owner:** <name>  
**Last Tested:** YYYY-MM-DD (or "Not yet tested")

## Purpose
One or two sentences explaining why this procedure exists.

## When to Use
Bullet list of trigger conditions.

## Prerequisites
- Access requirements
- Tools / credentials needed
- Assumed starting state

## Procedure
Numbered steps. Keep each step atomic and verifiable.

## Verification
How to confirm the procedure succeeded.

## Rollback
How to undo or recover if something goes wrong.

## Related
- Links to design docs, other runbooks, secrets paths, dashboards, etc.
```

## Core Runbooks (MVP)

| ID | Title | File |
|----|-------|------|
| RB-001 | Deploy Service | [01-deploy-service.md](01-deploy-service.md) |
| RB-002 | Rotate Secrets | [02-rotate-secrets.md](02-rotate-secrets.md) |
| RB-003 | Restore Backup | [03-restore-backup.md](03-restore-backup.md) |
| RB-004 | Recover VM | [04-recover-vm.md](04-recover-vm.md) |
| RB-005 | Add New Application | [05-add-new-application.md](05-add-new-application.md) |
| RB-006 | Portfolio Cut-over / Rollback | [06-portfolio-cutover-rollback.md](06-portfolio-cutover-rollback.md) |

## Principles

- Prefer runbooks over tribal knowledge.
- Every procedure that can break production must have a tested rollback path.
- Update the “Last Tested” date whenever the procedure is successfully executed.
- Keep steps concrete; avoid “configure the thing appropriately.”
