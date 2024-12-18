# Removing Degraded RAID (DEID) Configuration and Repurposing Disk

## Overview
This document outlines the steps to stop and remove a degraded RAID (DEID) configuration on a server and repurpose the disk for general use.

---

## Steps Performed

### 1. Check the RAID Status
- Command:
  ```bash
  cat /proc/mdstat
  ```
- Purpose: Verify the existing RAID setup and confirm which arrays are degraded.

### 2. Stop the RAID Devices
- Command (for each RAID device):
  ```bash
  mdadm --stop /dev/mdX
  ```
  Replace `/dev/mdX` with the actual RAID device names (e.g., `/dev/md0`, `/dev/md1`).

### 3. Clear RAID Metadata from Disks
- Command (for each disk or partition):
  ```bash
  mdadm --zero-superblock /dev/nvme1n1pX
  ```
  Replace `/dev/nvme1n1pX` with the specific partitions (e.g., `/dev/nvme1n1p1`, `/dev/nvme1n1p2`).

### 4. Delete Old Partitions
- Command:
  ```bash
  fdisk /dev/nvme1n1
  ```
- Steps in `fdisk`:
  1. Press `d` to delete existing partitions.
  2. Repeat for all partitions on the disk.
  3. Press `w` to write changes and exit.

### 5. Create a New Partition
- Command:
  ```bash
  fdisk /dev/nvme1n1
  ```
- Steps in `fdisk`:
  1. Press `n` to create a new partition.
  2. Choose the default values to use the entire disk.
  3. Press `w` to write changes and exit.

### 6. Format the New Partition
- Command:
  ```bash
  mkfs.ext4 /dev/nvme1n1p1
  ```
- Purpose: Create an ext4 filesystem on the newly created partition.

### 7. Mount the New Partition
1. Create a mount point:
   ```bash
   mkdir /mnt/data
   ```
2. Mount the partition:
   ```bash
   mount /dev/nvme1n1p1 /mnt/data
   ```
3. Add to `/etc/fstab` for automatic mounting on boot:
   ```bash
   echo "/dev/nvme1n1p1 /mnt/data ext4 defaults 0 0" >> /etc/fstab
   ```

### 8. Verify the Mount
- Command:
  ```bash
  mount -a
  df -h
  ```
- Purpose: Ensure the partition is mounted and check the available disk space.

---

## Results
- The degraded RAID configuration was successfully stopped and removed.
- The disk `/dev/nvme1n1` was repurposed with a single ext4 partition mounted at `/mnt/data`.

---

## Notes
- Be cautious when performing disk operations to avoid data loss.
- Always back up important data before modifying partitions or RAID configurations.

---

## References
- `man mdadm`
- `man fdisk`
- Linux Filesystem and Partition Management

