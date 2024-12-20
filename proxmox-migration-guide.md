# Proxmox VM Migration Guide Between Non-Clustered Hosts

This guide explains how to migrate VMs between Proxmox hosts that are not in the same cluster.

## Prerequisites

- Source Proxmox server
- Target Proxmox server
- SSH access to both servers
- Sufficient storage space on target server
- Root access on both servers

## Step-by-Step Migration Process

### 1. List VMs on Source Server

First, identify the VMID of the machine you want to migrate:

```bash
qm list
```

### 2. Create Backup on Source Server

Stop the VM and create a compressed backup:

```bash
# Stop the VM
qm stop <VMID>

# Create backup
vzdump <VMID> --compress zstd --mode stop
```

### 3. Transfer to Target Server

Transfer the backup file to the target server:

```bash
scp /var/lib/vz/dump/vzdump-qemu-<VMID>-*.vma.zst root@target-ip:/var/lib/vz/dump/
```

Note: On first connection, you'll need to accept the SSH fingerprint:
```
The authenticity of host 'target-ip' can't be established.
ED25519 key fingerprint is SHA256:xxx
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
Type 'yes' to continue.

### 4. Check Storage on Target Server

Before restoring, verify available storage on the target:

```bash
pvesm status
```

### 5. Restore VM on Target Server

If this is the first restore attempt:

```bash
qmrestore /var/lib/vz/dump/vzdump-qemu-<VMID>-*.vma.zst <VMID> --storage <storage_name>
```

If you get an error about VM already existing:

```bash
# Remove failed/partial VM
qm destroy <VMID>

# Try restore again
qmrestore /var/lib/vz/dump/vzdump-qemu-<VMID>-*.vma.zst <VMID> --storage <storage_name>
```

### 6. Verify Migration

Check if the VM is properly restored:

```bash
qm list
```

## Common Issues and Solutions

### Storage Doesn't Exist Error

If you get an error about storage not existing, make sure to:
1. Check available storage with `pvesm status`
2. Use the correct storage name in the restore command
3. Create required storage if needed

### VM Already Exists Error

If the VM already exists on target:
1. Remove the partial VM with `qm destroy <VMID>`
2. Attempt restore again

## Tips

- Use `--compress zstd-fast` for faster compression if needed
- For large VMs, consider using `rsync` instead of `scp`
- Make sure target server has enough free storage
- Consider using a different VMID on target if needed

## Example Migration

Here's a complete example migrating VM 2001:

```bash
# On source server
qm stop 2001
vzdump 2001 --compress zstd --mode stop
scp /var/lib/vz/dump/vzdump-qemu-2001-*.vma.zst root@target-ip:/var/lib/vz/dump/

# On target server
pvesm status
qmrestore /var/lib/vz/dump/vzdump-qemu-2001-*.vma.zst 2001 --storage local
```

## Additional Resources

- [Proxmox VE Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [Proxmox Backup & Restore Documentation]([https://pve.proxmox.com/wiki/Backup_and_Restore](https://cyanlabs.net/tutorials/install-proxmox-on-a-hetzner-dedicated-server-with-1-ip-using-sdn-and-without-kvm-using-qemu/)
- [Proxmox Backup Documentation](https://pve.proxmox.com/wiki/Backup_and_Restore)
