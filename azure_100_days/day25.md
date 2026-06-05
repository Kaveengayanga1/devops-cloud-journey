# Day 25: Resolving Storage Expansion and Mounting Issues in Azure VMs

## Scenario
The Nautilus DevOps team needed to expand the storage capacity of an existing virtual machine and add an additional data disk to support increased workloads. The task required resizing the existing VM disk and mounting a new data disk.

## Details and Credentials Used
- **Resource Group:** `kml_rg_main-759a6ab875fe421c`
- **VM Name:** `xfusion-vm`
- **Credentials:** No SSH credentials (username/password/key) were provided. 
- **Access Method:** Azure Portal and Azure CLI using the `az vm run-command invoke` feature, which leverages the Azure VM Agent to run non-interactive scripts directly on the VM without requiring SSH access.

## Tasks
1. Expand the existing VM `xfusion-vm` disk from `32Gi` to `64Gi`.
2. Create a new standard HDD data disk named `xfusion-disk` of `64Gi` and mount the disk to VM `xfusion-vm` at location `/mnt/xfusion-disk`.

## Mistakes Made and Why They Happened

### The Mistake
An attempt was initially made to format and mount `/dev/sdc` as the new data disk using the following commands:
```bash
sudo mkfs -t ext4 /dev/sdc
sudo mount /dev/sdc /mnt/xfusion-disk
```
This resulted in errors stating that `/dev/sdc` was already in use and mounted.

### Why it Happened
In Azure Linux VMs, the device naming convention (like `sda`, `sdb`, `sdc`) depends on the order disks are attached. Often, `sdb` or `sdc` is reserved by Azure as the **Temporary/Resource Disk**. 
By assuming the new disk was automatically `/dev/sdc` without verifying, the script attempted to overwrite a disk that was already mounted and in use by the system for temporary storage.

### The Fix
To identify the correct disk, the `lsblk` command was executed:
```bash
lsblk -o NAME,SIZE,MOUNTPOINT,TYPE
```
The output revealed that:
- `sdb` was the OS disk (already at 64G).
- `sdc` was the 4G temporary disk.
- `sda` was the newly attached 64G unmounted disk.

Once the correct device (`/dev/sda`) was identified, the formatting and mounting were performed on it successfully.

## Theories and Concepts

### 1. Disk Resizing vs. Partition Expansion
In cloud platforms, increasing the disk size in the portal only expands the physical block storage. The OS still operates within the old logical boundaries. You must use tools like `growpart` to expand the partition and `resize2fs` (for EXT4) to expand the filesystem to recognize the new space.

### 2. Device Naming in Azure
Azure dynamically assigns names to disks (sda, sdb, sdc). `sdb` or `sdc` is usually the temporary storage. Never assume a disk name; always use `lsblk` to verify unmounted disks before formatting.

### 3. File Systems and Formatting
A new "raw" disk cannot store data until a filesystem (like EXT4 or XFS) is created on it using `mkfs`. This organizes the disk into readable blocks and inodes.

### 4. Mounting and Persistence (/etc/fstab)
Mounting standardizes disk access by linking a storage device to a directory (mount point). However, these links are lost upon reboot. To make them persistent, the device's UUID must be added to `/etc/fstab`.

### 5. UUIDs and Mount Flags
- **UUID:** Using UUIDs instead of device names (`/dev/sda`) in `fstab` is much safer because device names can change if you add/remove disks.
- **nofail flag:** Adding `nofail` in `fstab` ensures that if the secondary disk fails to mount during reboot, the OS will still boot up gracefully instead of halting.

## Output of Successful Execution

```json
{
  "value": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      "level": "Info",
      "message": "Enable succeeded: \n[stdout]\nDiscarding device blocks:        0/16777216\b\b\b\b\b\b\b\b\b\b\b\b\b\b\b\b\b                 \b\b\b\b\b\b\b\b\b\b\b\b\b\b\b\b\bdone                            \nCreating filesystem with 16777216 4k blocks and 4194304 inodes\nFilesystem UUID: 39027935-f142-4f4e-a93a-f93c32284548\nSuperblock backups stored on blocks: \n\t32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, \n\t4096000, 7962624, 11239424\n\nAllocating group tables:   0/512\b\b\b\b\b\b\b       \b\b\b\b\b\b\bdone                            \nWriting inode tables:   0/512\b\b\b\b\b\b\b       \b\b\b\b\b\b\bdone                            \nCreating journal (131072 blocks): done\nWriting superblocks and filesystem accounting information:   0/512\b\b\b\b\b\b\b       \b\b\b\b\b\b\bdone\n\nUUID=39027935-f142-4f4e-a93a-f93c32284548 /mnt/xfusion-disk ext4 defaults,nofail 0 2\nFilesystem      Size  Used Avail Use% Mounted on\n/dev/sda         63G   24K   60G   1% /mnt/xfusion-disk\n\n[stderr]\nmke2fs 1.46.5 (30-Dec-2021)\n",
      "time": null
    }
  ]
}
```
