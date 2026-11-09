# Runbook: Recover VM

**ID:** RB-004  
**Last Updated:** 2026-11-09  
**Owner:** Project owner  
**Last Tested:** Not yet tested

## Purpose

Recover a failed or corrupted virtual machine on the Proxmox host so that platform services or the portfolio application return to a healthy state.

## When to Use

- VM fails to boot
- File-system or disk corruption
- Accidental deletion or severe misconfiguration of a VM
- Host-level storage issues affecting a single VM

## Prerequisites

- Access to the Proxmox web UI or CLI (via Tailscale or local console)
- Knowledge of which VM is affected (`platform-core`, `apps`, or `dev`)
- Recent Proxmox backups or snapshots available
- Understanding of dependencies (e.g., if `platform-core` is down, OpenBao/Grafana/Loki are unavailable)

## Procedure

### 1. Assess the failure

- Check Proxmox task history and VM status.
- Attempt to view the console / serial output for boot errors.
- Determine whether the problem is boot-related, disk-related, or network-related.

### 2. Prefer non-destructive recovery first

- Try a clean reboot.
- If a recent snapshot exists, roll back to it.
- Check storage status (ZFS, LVM, directory) for the VM’s disks.

### 3. Restore from backup (if needed)

1. Identify the latest known-good backup for the VM.
2. Restore the VM (see RB-003 for details).
3. Prefer restoring to a new VMID first if you want to keep the broken VM for forensics.
4. Start the restored VM and verify Tailscale comes up.

### 4. Re-join services

- Confirm Tailscale is connected.
- Verify internal DNS still resolves correctly.
- Start or restart platform services as required.
- If the `apps` VM was recovered, re-verify the portfolio site and database connectivity.
- If `platform-core` was recovered, re-unseal OpenBao if necessary and confirm observability stack health.

### 5. Clean up

- Once the recovered VM is confirmed healthy, remove or archive the broken VM if it is no longer needed.
- Update any documentation that referenced the old VMID or IP.

## Verification

- [ ] VM is running and reachable over Tailscale
- [ ] SSH access works
- [ ] Critical services on that VM are healthy
- [ ] Dependent services on other VMs can still reach it
- [ ] Backups and monitoring resume

## Rollback

- If the restored VM is worse than the original, power it off and re-evaluate from the previous backup or snapshot.
- Keep the original disk images until the recovery is fully validated.

## Related

- RB-003 Restore Backup
- Proxmox snapshot and backup configuration
- `docs/design.md` §3 System Architecture and §8 Backup and Recovery
- OpenBao unseal procedure (if `platform-core` is involved)
