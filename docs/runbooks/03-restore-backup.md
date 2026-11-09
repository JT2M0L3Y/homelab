# Runbook: Restore Backup

**ID:** RB-003  
**Last Updated:** 2026-11-09  
**Owner:** Project owner  
**Last Tested:** Not yet tested

## Purpose

Restore data or system state from a known-good backup. Covers both Proxmox VM-level backups and application-level (PostgreSQL) logical backups.

## When to Use

- Accidental data deletion or corruption
- Failed migration or schema change
- Need to return a service to a previous known state
- Disaster recovery testing

## Prerequisites

- Access to the Proxmox host (or backup storage location)
- Access to the `apps` VM for database restores
- Knowledge of the backup schedule and retention policy
- Sufficient disk space for the restore operation
- Tailscale connectivity

## Procedure

### A. Restore an entire VM from Proxmox backup

1. Identify the correct backup (date, VMID, storage).
2. In Proxmox UI or CLI, select the backup and choose “Restore”.
3. Decide whether to overwrite the existing VM or restore to a new VMID.
4. Start the restored VM.
5. Verify Tailscale connectivity and service health.
6. Update any internal DNS or configuration that referenced the old VM if a new VMID was used.

### B. Restore PostgreSQL from logical backup (pg_dump)

1. Locate the desired dump file (local storage, object storage, or backup volume).
2. Connect to the `apps` VM.
3. Stop or quiesce the application that uses the database (to avoid writes during restore).
4. Restore the dump:
   ```bash
   # Example – adjust for actual tooling
   pg_restore --clean --if-exists -d <database> <backup-file>
   # or
   psql -d <database> < backup.sql
   ```
5. Restart the application.
6. Verify data integrity and application functionality.

### C. Restore OpenBao data (if applicable)

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

- Proxmox backup schedule (document concrete storage and retention here once defined)
- PostgreSQL backup scripts / cron jobs
- RB-004 Recover VM
- `docs/design.md` §8 Backup and Recovery
