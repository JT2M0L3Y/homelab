# Runbook: Recover Project Environment

**ID:** RB-004  
**Last Updated:** 2026-11-11  
**Owner:** Platform operator
**Last Tested:** Not yet tested

## Purpose

Recover a failed or corrupted Proxmox-managed project VM or LXC. Core Docker
services run on the Debian host and have a separate restore procedure in RB-003.

## When to Use

- Project VM or LXC fails to start
- File-system or disk corruption
- Accidental deletion or severe misconfiguration of a project environment
- Host-level storage issues affecting a project guest

## Prerequisites

- Access to the Proxmox web UI or CLI (via NetBird or local console)
- Knowledge of the affected VM or LXC and its workload
- Recent Proxmox backups or snapshots available
- Understanding of service dependencies (for example, the portfolio guest
  uses host PostgreSQL and OpenBao)

## Procedure

### 1. Assess the failure

- Check Proxmox task history and guest status.
- Attempt to view the guest console for startup errors.
- Determine whether the problem is boot-related, disk-related, or network-related.

### 2. Prefer non-destructive recovery first

- Try a clean reboot.
- If a recent snapshot exists, roll back to it.
- Check storage status (ZFS, LVM, directory) for the guest disks and attached
  mount points.

### 3. Restore from backup (if needed)

1. Identify the latest known-good Proxmox backup for the guest.
2. Restore the VM or LXC (see RB-003 for details).
3. Prefer restoring with a new VMID/CTID if the broken environment is needed
   for forensics.
4. Confirm the restored guest uses the project subnet, start it, and verify
   service reachability. NetBird subnet routing is provided by the host.

### 4. Re-join services

- Confirm NetBird is connected on the host and the advertised route is
  available to authorized peers.
- Verify internal DNS still resolves correctly.
- Start the project workload and confirm connectivity to host core services.
- If the portfolio environment was recovered, verify the site, PostgreSQL, and
  OpenBao connectivity.

### 5. Clean up

- Once the recovered environment is confirmed healthy, remove or archive the
  broken guest if it is no longer needed.
- Update any documentation that referenced the old VMID/CTID or IP.

## Verification

- [ ] Project VM or LXC is running and reachable over the authorized route
- [ ] Proxmox management works
- [ ] Workload is healthy and can reach required host services
- [ ] Caddy routes to the workload where applicable
- [ ] Backups and monitoring resume

## Rollback

- If the restored guest is worse than the original, stop it and re-evaluate from
  the previous backup or snapshot.
- Keep the original guest storage until recovery is fully validated.

## Related

- RB-003 Restore Backup
- Proxmox guest backup and snapshot configuration
- [Technical design](../design.md) §3 System Architecture and §8 Backup and Recovery
- [RB-003 Restore Backup](03-restore-backup.md) for host core services and data
