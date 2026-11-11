# Runbook: Restore Backup

**ID:** RB-003  
**Last Updated:** 2026-11-11
**Owner:** Platform operator
**Last Tested:** Not yet tested

## Purpose

Restore host Docker service data, Proxmox project environments, or PostgreSQL
logical backups from a known-good backup.

## When to Use

- Accidental data deletion or corruption
- Failed migration or schema change
- Need to return a service to a previous known state
- Disaster recovery testing

## Prerequisites

- Access to the Proxmox host (or backup storage location)
- Access to the Debian host and Proxmox management
- Knowledge of the backup schedule and retention policy
- Sufficient disk space for the restore operation
- NetBird connectivity or local-console access

## Procedure

### A. Create a PostgreSQL logical backup

1. Choose the database and the configured protected backup destination.
2. From the Debian host, run `pg_dump` in the PostgreSQL Docker container for
   each database and write the dumps to the configured backup destination.
3. Confirm the dump was created and is included in the backup retention policy.

### B. Restore a Proxmox project environment

1. Identify the correct Proxmox guest backup (date, VMID/CTID, storage).
2. In Proxmox UI or CLI, restore the VM or LXC. Prefer a new ID if the existing
   environment may be needed for investigation.
3. Confirm network attachment to the project subnet and start the guest.
4. Verify the workload and its connectivity to host core services.
5. Update AdGuard DNS or Caddy upstream configuration if the guest address
   changed.

### C. Restore host Docker core service data

1. Locate the protected backup of the service's configuration and persistent
   data.
2. Stop the affected Docker service on the Debian host.
3. Restore its configuration/data to the documented host paths and check
   ownership and permissions.
4. Recreate/start the service using its Docker Compose configuration.
5. Verify the service and its dependent workloads.

### D. Restore PostgreSQL from logical backup (pg_dump)

1. Locate the desired dump file (protected backup storage).
2. Stop or quiesce the application that uses the database.
3. Restore into the PostgreSQL Docker service:
   ```bash
   # Example – adjust for actual tooling
   pg_restore --clean --if-exists -d <database> <backup-file>
   # or
   psql -d <database> < backup.sql
   ```
4. Restart the dependent application service in its project environment.
5. Verify data integrity and application functionality.

### E. Restore OpenBao data (if applicable)

1. Follow OpenBao’s documented recovery procedure for the storage backend in use.
2. Have unseal / recovery keys available offline.
3. Verify secrets are readable after unseal.

## Verification

- [ ] Target system or database is online
- [ ] Application can read/write expected data
- [ ] Critical recent records are present (spot-check)
- [ ] Logs and metrics resume normally
- [ ] No obvious permission or ownership issues on restored files

## Rollback

- If the restore itself fails or produces worse results, re-restore from an earlier backup.
- Keep the pre-restore snapshot or export if possible so the “bad” state can be examined later.
- Document the exact backup identifier used.

## Related

- Proxmox project-environment backup schedule
- Host Docker configuration/volume backup procedure
- PostgreSQL `pg_dump` scripts / cron jobs
- [RB-004 Recover Project Environment](04-recover-project-environment.md)
- `docs/design.md` §8 Backup and Recovery
