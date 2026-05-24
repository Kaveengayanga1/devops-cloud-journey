# Day 11: Resizing an Azure Virtual Machine

## Overview
Resizing an Azure VM is a fundamental operation in cloud cost management and performance optimization. This process allows you to change the VM's compute capacity (CPU, RAM) to match changing workload requirements without recreating the entire VM.

## 1. Theoretical Concepts

### A. Understanding VM Resizing
When you resize a VM, you're changing its hardware profile - specifically the VM Size (SKU). This affects:
- **CPU cores**: Number of virtual CPUs available
- **Memory (RAM)**: Amount of memory allocated
- **Network bandwidth**: Maximum network throughput
- **Storage options**: Temporary disk size and IOPS limits
- **NIC support**: Maximum number of network interfaces

**Example:** Resizing from `Standard_B1s` (1 vCPU, 1 GB RAM) to `Standard_B2s` (2 vCPUs, 4 GB RAM) doubles your compute capacity.

### B. VM States During Resize
The resize operation typically follows this sequence:

1. **Running → Stopping**: VM gracefully shuts down
2. **Deallocating**: VM releases compute resources (still retains storage)
3. **Resizing**: Azure allocates new hardware resources
4. **Starting**: VM boots with new size
5. **Running**: VM is operational with new capacity

**Important:** During resize, the VM experiences downtime (typically 2-10 minutes).

### C. Hardware Availability
Not all VM sizes are available in every hardware cluster:

- **Same Cluster Resize**: If the new size is available on the current hardware cluster, resize happens quickly
- **Cross-Cluster Resize**: If unavailable, VM must deallocate and move to a different cluster (longer downtime)
- **Region Constraints**: Some sizes are only available in specific Azure regions

**Pro Tip:** Use `az vm list-sizes` to check available sizes in your region before resizing.

### D. Data Persistence Rules

| Resource | Persistent? | Notes |
|----------|------------|-------|
| OS Disk | ✅ Yes | Fully preserved |
| Data Disks | ✅ Yes | Fully preserved |
| Temporary Disk (D:) | ❌ No | **Data will be lost** |
| Static Public IP | ✅ Yes | If configured as static |
| Dynamic Public IP | ⚠️ Maybe | May change after resize |
| Private IP | ✅ Yes | Preserved |

**Critical:** Always backup data on temporary disks before resizing!

### E. Cost Implications
Resizing directly affects your Azure costs:

- **Upsize**: Increased capacity = higher hourly rate
- **Downsize**: Reduced capacity = lower hourly rate
- **Billing**: Charged for the new size once resize completes

**Best Practice:** Use Azure Calculator to estimate cost changes before resizing.

## 2. Practical Guide: Resizing VM via Azure CLI

### Step 1: Verify Current VM Configuration

First, identify your VM's current size and resource group:

```bash
az vm list --query "[?name=='nautilus-vm'].{Name:name, ResourceGroup:resourceGroup, Size:hardwareProfile.vmSize}" -o table
```

**Output:**
```
Name         ResourceGroup                 Size
-----------  ----------------------------  ------------
nautilus-vm  KML_RG_MAIN-87CCEF343BD141E1  Standard_B1s
```

### Step 2: Check Available Sizes (Optional)

To see all sizes available for your VM in its current location:

```bash
az vm list-vm-resize-options \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --name nautilus-vm \
  --query "[].name" -o table
```

This helps verify that `Standard_B2s` is available before attempting the resize.

### Step 3: Resize the VM

Execute the resize command:

```bash
az vm resize \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --name nautilus-vm \
  --size Standard_B2s
```

**What Happens:**
- Azure CLI sends resize request
- VM automatically stops and deallocates
- New hardware resources allocated
- VM restarts with new size

**Output:** (abbreviated)
```json
{
  "hardwareProfile": {
    "vmSize": "Standard_B2s"
  },
  "name": "nautilus-vm",
  "provisioningState": "Succeeded",
  "resourceGroup": "KML_RG_MAIN-87CCEF343BD141E1"
}
```

### Step 4: Verify New Size and Status

Confirm the resize succeeded and VM is running:

```bash
az vm get-instance-view \
  --name nautilus-vm \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --query "{Status:instanceView.statuses[1].displayStatus, Size:hardwareProfile.vmSize}"
```

**Output:**
```json
{
  "Size": "Standard_B2s",
  "Status": "VM running"
}
```

## 3. Common Scenarios and Solutions

### Scenario 1: "Size Not Available" Error

**Error Message:**
```
Requested size Standard_B2s is not available in the current hardware cluster
```

**Solution:**
Deallocate the VM first, then resize:

```bash
# Deallocate VM
az vm deallocate \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --name nautilus-vm

# Resize
az vm resize \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --name nautilus-vm \
  --size Standard_B2s

# Start VM
az vm start \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --name nautilus-vm
```

### Scenario 2: Downsize to Reduce Costs

To reduce costs during off-peak hours:

```bash
# Resize to smaller instance
az vm resize \
  --resource-group KML_RG_MAIN-87CCEF343BD141E1 \
  --name nautilus-vm \
  --size Standard_B1s
```

### Scenario 3: Temporary Scale-Up for Heavy Workload

1. **Scale Up** before batch job:
   ```bash
   az vm resize -g KML_RG_MAIN-87CCEF343BD141E1 -n nautilus-vm --size Standard_D4s_v3
   ```

2. **Run workload** (via SSH or automation)

3. **Scale Down** after completion:
   ```bash
   az vm resize -g KML_RG_MAIN-87CCEF343BD141E1 -n nautilus-vm --size Standard_B2s
   ```

## 4. Best Practices

### ✅ Do's
- **Monitor Performance**: Use Azure Monitor to identify when resize is needed
- **Test in Non-Prod**: Test resize operations in dev/test environments first
- **Schedule Downtime**: Plan resizes during maintenance windows
- **Automate**: Use Azure Automation or Logic Apps for scheduled scaling
- **Document**: Keep records of size changes and reasons

### ❌ Don'ts
- **Don't resize during peak hours**: Avoid downtime during business-critical periods
- **Don't store critical data on temporary disks**: Always use managed disks
- **Don't resize without capacity planning**: Understand your actual resource needs
- **Don't forget cost analysis**: Calculate cost impact before upscaling

## 5. Quick Reference Commands

```bash
# List current VM details
az vm show -g <ResourceGroup> -n <VMName> -d --query "{Size:hardwareProfile.vmSize, Status:powerState}"

# Check available sizes in location
az vm list-sizes --location westus -o table

# Resize VM
az vm resize -g <ResourceGroup> -n <VMName> --size <NewSize>

# Deallocate, resize, and start (for stubborn cases)
az vm deallocate -g <ResourceGroup> -n <VMName>
az vm resize -g <ResourceGroup> -n <VMName> --size <NewSize>
az vm start -g <ResourceGroup> -n <VMName>

# Check VM status after resize
az vm get-instance-view -g <ResourceGroup> -n <VMName> --query "instanceView.statuses[1].displayStatus"
```

## 6. Summary

Resizing VMs in Azure is a powerful tool for:
- **Cost Optimization**: Scale down when not needed
- **Performance Tuning**: Scale up for demanding workloads
- **Right-Sizing**: Align resources with actual requirements

**Key Takeaway:** Resize operations cause brief downtime but preserve all managed disk data, making it a safe operation for most scenarios.

---

**Next Steps:** Learn about VM Auto-Shutdown schedules and Azure Automation for hands-off cost management.
