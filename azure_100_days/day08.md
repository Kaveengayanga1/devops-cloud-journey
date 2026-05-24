# Day 08: Attaching Existing Azure Disk to a VM

## Overview
This guide provides the specific Azure CLI commands to gather your resource details and attach your existing volume.

## Prerequisites
Ensure you are logged in to the Azure CLI.

```bash
az login
```

## Step 1: Get Resource Group Details
You need the exact name of the Resource Group (RG) where your VM and Disk are located.

**List all resource groups:**

```bash
az group list --output table
```

**Output:**
```
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-18e6b0f7f27648cc  westus      Succeeded
```

Copy the relevant name from the first column (e.g., `myResourceGroup`).

## Step 2: Get VM Details
Once you have the Resource Group name, find your Virtual Machine's name.

**List VMs in that Resource Group:**

```bash
# Replace <myResourceGroup> with the name from Step 1
az vm list --resource-group <myResourceGroup> --output table
```

**Actual Command:**
```bash
az vm list --resource-group kml_rg_main-18e6b0f7f27648cc --output table
```

**Output:**
```
Name           ResourceGroup                 Location    Zones
-------------  ----------------------------  ----------  -------
datacenter-vm  kml_rg_main-18e6b0f7f27648cc  eastus
```

Copy the name of your VM (e.g., `my-linux-vm`).

## Step 3: Get Storage (Disk) Details
Now, find the name (or ID) of the existing disk you want to attach.

**List all Managed Disks in the Resource Group:**

```bash
az disk list --resource-group <myResourceGroup> --output table
```

**Actual Command:**
```bash
az disk list --resource-group kml_rg_main-18e6b0f7f27648cc --output table
```

**Output:**
```
Name                                                  ResourceGroup                 Location    Zones    Sku           SizeGb    ProvisioningState    OsType
----------------------------------------------------  ----------------------------  ----------  -------  ------------  --------  -------------------  --------
datacenter-disk                                       kml_rg_main-18e6b0f7f27648cc  eastus               Standard_LRS  30        Succeeded
datacenter-vm_disk1_95d56067e9a74a688c0cd23aebd4110e  kml_rg_main-18e6b0f7f27648cc  eastus               Standard_LRS  30        Succeeded            Linux
```

Copy the name of the disk (e.g., `myDataDisk`).

**Note:** If your disk is in a different Resource Group than your VM, you will need the disk's full ID instead of just the name. You can get the ID by adding `--query '[].id'` to the command above.

## Step 4: Attach the Volume
Use the `az vm disk attach` command to connect the disk to the VM.

**Command:**

```bash
az vm disk attach \
  --resource-group <myResourceGroup> \
  --vm-name <myVMName> \
  --name <myDataDiskName>
```

**Actual Command:**
```bash
az vm disk attach \
  --resource-group kml_rg_main-18e6b0f7f27648cc \
  --vm-name datacenter-vm \
  --name datacenter-disk
```

**If the disk is in a DIFFERENT Resource Group:**

```bash
az vm disk attach \
  --resource-group <vmResourceGroup> \
  --vm-name <myVMName> \
  --ids <full_disk_resource_id>
```

## Step 5: Verify the Attachment
Confirm that Azure sees the disk attached to the VM.

```bash
az vm show \
  --resource-group <myResourceGroup> \
  --name <myVMName> \
  --query "storageProfile.dataDisks" \
  --output table
```

**Actual Command:**
```bash
az vm show \
  --resource-group kml_rg_main-18e6b0f7f27648cc \
  --name datacenter-vm \
  --query "storageProfile.dataDisks" \
  --output table
```

**Output:**
```
Lun    Name             Caching    CreateOption    DiskSizeGb    ToBeDetached    DeleteOption
-----  ---------------  ---------  --------------  ------------  --------------  --------------
0      datacenter-disk  None       Attach          30            False           Detach
```

## Important Next Step: OS Initialization
The disk is now physically "plugged in," but your VM's operating system (Windows/Linux) doesn't know how to use it yet. You typically need to Remote Desktop (RDP) or SSH into the VM to initialize and format the disk:
- Create Partition
- Format (NTFS/ext4)
- Mount

## Summary
- Resource Group: `kml_rg_main-18e6b0f7f27648cc`
- VM Name: `datacenter-vm`
- Data Disk Attached: `datacenter-disk` (30 GB, Standard_LRS)
- Attachment Status: ✅ Successfully attached at LUN 0
