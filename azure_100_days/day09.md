# Day 09: Attaching a Network Interface (NIC) to an Azure VM

## Overview
Attaching a network interface (NIC) is fundamentally different from attaching a disk. While a disk is a "hot-pluggable" storage device, a NIC represents the VM's identity on the network.

## 1. Theoretical Concepts
To understand why the process works the way it does, you must understand the "Chain of Dependencies" in Azure networking:

### A. The Separation of Identity
In physical computers, the network card is soldered to the motherboard. In Azure, the VM and the NIC are completely separate resources.

- The VM provides the CPU and RAM.
- The NIC holds the IP address, MAC address, and Network Security Group (NSG) rules.

**Key Takeaway:** If you delete a VM, the NIC (and its IP) usually survives unless you explicitly set it to delete automatically.

### B. The "Deallocation" Constraint
This is the most critical theory to know. Unlike disks, you typically cannot attach a new NIC to a running VM.

- The VM must be in a **Deallocated** state (completely stopped, not billing for compute).
- **Why?** The hypervisor needs to map new virtual hardware paths to the VM's bus, which usually requires a cold boot to register correctly.

### C. VM Size Limits
Not all VMs can handle multiple NICs.

- A small VM (e.g., `Standard_B1s`) supports only **1 NIC**.
- Larger VMs (e.g., `Standard_D2s_v3`) support **2 or more**.

**Theory Check:** If your VM size only supports 1 NIC and it already has one (the primary), the attachment command will fail.

## 2. Guide: Attaching the NIC (Azure CLI)
Because of the constraints above, this process requires downtime.

### Step 1: Verify VM Size Support
Before stopping anything, check if your VM size supports more than 1 NIC.

- Go to Azure Portal -> Your VM -> Size.
- Look for the column "Max NICs". If you are trying to add a second NIC and this number is 1, you must resize the VM first.

### Step 2: List Resource Groups and Resources

**List all resource groups:**
```bash
az group list --output table
```

**Output:**
```
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-7c88e309798443b3  westus      Succeeded
```

**List all VMs in the resource group:**
```bash
az vm list --resource-group kml_rg_main-7c88e309798443b3 --output table
```

**Output:**
```
Name         ResourceGroup                 Location    Zones
-----------  ----------------------------  ----------  -------
nautilus-vm  kml_rg_main-7c88e309798443b3  westus
```

### Step 3: Get Resource IDs
We need the "ID" of the NIC you want to attach and the VM name.

**List all NICs to find the name of the one you want to attach:**
```bash
az network nic list --resource-group <myResourceGroup> --output table
```

**Actual Command:**
```bash
az network nic list --resource-group kml_rg_main-7c88e309798443b3 --output table
```

**Output:**
```
AuxiliaryMode    AuxiliarySku    DisableTcpStateTracking    EnableIPForwarding    Location    Name              NicType    ProvisioningState    ResourceGroup                 ResourceGuid                          VnetEncryptionSupported    MacAddress         Primary
---------------  --------------  -------------------------  --------------------  ----------  ----------------  ---------  -------------------  ----------------------------  ------------------------------------  -------------------------  -----------------  ---------
None             None            False                      False                 westus      nautilus-nic      Standard   Succeeded            kml_rg_main-7c88e309798443b3  21672662-a22f-4d48-9e20-c917c29c58fe  False
None             None            False                      False                 westus      nautilus-vmVMNic  Standard   Succeeded            kml_rg_main-7c88e309798443b3  2b5ae94b-6186-4c6a-a1dc-65950d22f324  False                      60-45-BD-01-DD-57  True
```

**Store the NIC ID in a variable for easier use:**
```bash
nicId=$(az network nic show \
  --resource-group <myResourceGroup> \
  --name <myExistingNICName> \
  --query id \
  --output tsv)
```

**Actual Command:**
```bash
nicId=$(az network nic show \
  --resource-group kml_rg_main-7c88e309798443b3 \
  --name nautilus-nic \
  --query id \
  --output tsv)
```

### Step 4: Attempt to Attach NIC (Learning from Error)

**First attempt (without deallocating):**
```bash
az vm nic add \
  --resource-group kml_rg_main-7c88e309798443b3 \
  --vm-name nautilus-vm \
  --nics $nicId
```

**Error Received:**
```
(AddingOrDeletingNetworkInterfacesOnARunningVirtualMachineNotSupported) Virtual machine /subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-7c88e309798443b3/providers/Microsoft.Compute/virtualMachines/nautilus-vm with a single network interface must be stop-deallocated before it can be updated to have multiple network interfaces and vice-versa.
Code: AddingOrDeletingNetworkInterfacesOnARunningVirtualMachineNotSupported
Message: Virtual machine /subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-7c88e309798443b3/providers/Microsoft.Compute/virtualMachines/nautilus-vm with a single network interface must be stop-deallocated before it can be updated to have multiple network interfaces and vice-versa.
```

**Key Learning:** This error confirms the theoretical constraint - you CANNOT attach/detach NICs on a running VM. Deallocation is mandatory.

### Step 5: Stop (Deallocate) the VM
You must release the compute resources to update the hardware profile.

```bash
# Replace with your actual names
az vm deallocate \
  --resource-group <myResourceGroup> \
  --name <myVMName>
```

**Actual Command:**
```bash
az vm deallocate \
  --resource-group kml_rg_main-7c88e309798443b3 \
  --name nautilus-vm
```

### Step 6: Attach the NIC
This command links the existing NIC resource to the stopped VM.

```bash
az vm nic add \
  --resource-group <myResourceGroup> \
  --vm-name <myVMName> \
  --nics $nicId
```

**Actual Command:**
```bash
az vm nic add \
  --resource-group kml_rg_main-7c88e309798443b3 \
  --vm-name nautilus-vm \
  --nics $nicId
```

**Output:**
```json
[
  {
    "deleteOption": null,
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-7c88e309798443b3/providers/Microsoft.Network/networkInterfaces/nautilus-vmVMNic",
    "primary": true,
    "resourceGroup": "kml_rg_main-7c88e309798443b3"
  },
  {
    "deleteOption": null,
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-7c88e309798443b3/providers/Microsoft.Network/networkInterfaces/nautilus-nic",
    "primary": false,
    "resourceGroup": "kml_rg_main-7c88e309798443b3"
  }
]
```

**Analysis:** The output shows both NICs attached:
- `nautilus-vmVMNic` (Primary NIC - original)
- `nautilus-nic` (Secondary NIC - newly attached)

### Step 7: Restart the VM
Now that the hardware profile is updated, bring the VM back online.

```bash
az vm start \
  --resource-group <myResourceGroup> \
  --name <myVMName>
```

**Actual Command:**
```bash
az vm start \
  --resource-group kml_rg_main-7c88e309798443b3 \
  --name nautilus-vm
```

## 3. Post-Attachment Configuration (Inside the OS)
Just because Azure attached the NIC doesn't mean the OS uses it yet.

### Windows
Usually detects the new NIC automatically via DHCP. You might see it as "Ethernet 2".

### Linux
Often requires manual configuration if you want it to persist.

1. Run `ip link` to see the new interface (e.g., `eth1`).
2. You may need to create a configuration file:
   - **Ubuntu:** `/etc/netplan`
   - **RHEL/CentOS:** `/etc/sysconfig/network-scripts`
3. Assign it an IP or enable DHCP.

## Summary
- **Resource Group:** `kml_rg_main-7c88e309798443b3`
- **VM Name:** `nautilus-vm`
- **Primary NIC:** `nautilus-vmVMNic` (MAC: 60-45-BD-01-DD-57)
- **Secondary NIC:** `nautilus-nic` (Newly attached)
- **Status:** ✅ Successfully attached after deallocation
- **Important Lesson:** NICs cannot be hot-plugged; VM must be deallocated first (unlike disks)
