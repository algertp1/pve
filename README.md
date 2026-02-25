# Veeam Backup Repository - RAID6 + XFS Setup

## What this does
Configures 6 HDDs as mdadm RAID6 with XFS filesystem for use as a Veeam v13 backup repository.

- **RAID**: mdadm RAID6 (4+2 parity), 256K chunk size
- **Filesystem**: XFS with optimized mount options for large sequential I/O
- **Mount**: `/mnt/veeam-repo`

## Before running

1. Create the Ubuntu 24.04 VM in Proxmox and pass through 6 HDD disks
2. Verify disk devices with `lsblk` on the VM
3. Update `inventory.ini` with the VM's IP and SSH user
4. Update `raid_disks` in `playbook.yml` if your devices differ from sdb-sdg

## Usage

```bash
# Dry run first
ansible-playbook -i inventory.ini playbook.yml --check

# Run for real
ansible-playbook -i inventory.ini playbook.yml
```

## XFS mount options explained
- `noatime,nodiratime` - skip access time updates (reduces writes)
- `logbufs=8,logbsize=256k` - larger journal buffers for write performance
- `allocsize=1g` - large preallocation for sequential writes (backup workloads)

## XFS stripe alignment
- `su=256k` matches the RAID chunk size
- `sw=4` = number of data disks (6 total minus 2 parity)
- This ensures XFS writes align with the RAID stripe for optimal performance

## After running
1. Add the VM as a Managed Linux Server in Veeam console (SSH credentials)
2. Veeam auto-installs its transport agent
3. Add a new Backup Repository pointing to `/mnt/veeam-repo`
