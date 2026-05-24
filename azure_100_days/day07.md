# Day 07: Azure Public IP Address Allocation

## Output

```bash
~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-b4eade19d95f4450  westus      Succeeded

~ ➜  az network public-ip create \
    --resource-group kml_rg_main-b4eade19d95f4450 \
    --name xfusion-pip \
    --sku Standard \
    --allocation-method Static \
    --zone 1 2 3
(LocationNotSupportAvailabilityZones) The resource 'Microsoft.Network/publicIPAddresses/xfusion-pip' does not support availability zones at location 'westus'.
Code: LocationNotSupportAvailabilityZones
Message: The resource 'Microsoft.Network/publicIPAddresses/xfusion-pip' does not support availability zones at location 'westus'.

~ ✖ az network public-ip create \
    --resource-group kml_rg_main-b4eade19d95f4450 \
    --name xfusion-pip \
    --sku Standard \
    --allocation-method Static
[Coming breaking change] In the coming release, the default behavior will be changed as follows when sku is Standard and zone is not provided: For zonal regions, you will get a zone-redundant IP indicated by zones:["1","2","3"]; For non-zonal regions, you will get a non zone-redundant IP indicated by zones:null.
{
  "publicIp": {
    "ddosSettings": {
      "protectionMode": "VirtualNetworkInherited"
    },
    "etag": "W/\"7a6d2a1a-19a1-47ae-b026-35a39c36e9ce\"",
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-b4eade19d95f4450/providers/Microsoft.Network/publicIPAddresses/xfusion-pip",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "20.237.223.33",
    "ipTags": [],
    "location": "westus",
    "name": "xfusion-pip",
    "provisioningState": "Succeeded",
    "publicIPAddressVersion": "IPv4",
    "publicIPAllocationMethod": "Static",
    "resourceGroup": "kml_rg_main-b4eade19d95f4450",
    "resourceGuid": "c70acc79-885c-4ad0-95fa-8ab47a93a17c",
    "sku": {
      "name": "Standard",
      "tier": "Regional"
    },
    "type": "Microsoft.Network/publicIPAddresses"
  }
}

~ ➜  
```

## Theory

Allocation of a Public IP address in Azure involves two distinct parts: understanding the theory (why you choose certain options) and the practical execution (the CLI commands).

### Part 1: Theories You Need to Know

Before running any commands, you must understand three key concepts: SKUs, Allocation Methods, and Associations.

#### 1. SKUs: Basic vs. Standard (Crucial)

Azure offers two "tiers" of Public IPs. This is the most important decision you will make because Basic is being retired.

**Basic SKU (Retiring Sept 2025):**
- **Security:** Open to the internet by default (less secure).
- **Availability:** Does not support Availability Zones (no redundancy against data center failure).
- **Status:** Do not use this for new projects. Azure is retiring Basic Public IPs on September 30, 2025.

**Standard SKU (Recommended):**
- **Security:** "Secure by default." It is closed to all inbound traffic until you explicitly open a port using a Network Security Group (NSG).
- **Availability:** Supports Zone Redundancy (can survive a data center failure).
- **Requirement:** Required for modern resources like NAT Gateways and Standard Load Balancers.

#### 2. Allocation Methods: Static vs. Dynamic

This determines if your IP address stays the same or changes.

- **Dynamic:** The IP address is assigned when the resource starts and released (lost) when the resource stops (deallocated). If you stop a VM and start it again next week, you will get a different IP address.
- **Static:** The IP address is reserved immediately upon creation and stays with you until you explicitly delete the IP resource. Even if the VM is stopped for months, the IP remains yours.

**Note:** Standard SKU Public IPs almost always use Static allocation.

#### 3. Association (The Architecture)

A Public IP in Azure is a standalone resource. It is not "inside" a VM. You create the IP first, then "associate" (attach) it to a resource.

- **Network Interface (NIC):** To give a specific Virtual Machine internet access.
- **Load Balancer:** To distribute traffic to multiple VMs behind one IP.
- **VPN/Application Gateway:** For secure tunnel or web traffic management.

### Part 2: Practical - Allocating the IP via CLI

Here is how you translate the theories above into commands.

#### Step 1: Prerequisites

You need a Resource Group to hold the IP. If you don't have one:

```bash
az group create --name myResourceGroup --location eastus
```

#### Step 2: Create the Public IP (The Core Command)

We will create a Standard SKU IP with Static allocation (best practice).

```bash
az network public-ip create \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --sku Standard \
    --allocation-method Static \
    --zone 1 2 3
```

**Command Breakdown:**
- `--name`: The logical name of the resource (e.g., myWebserverIP).
- `--sku Standard`: Selects the modern, secure tier.
- `--allocation-method Static`: Ensures the IP address never changes.
- `--zone 1 2 3`: (Optional) Makes the IP Zone Redundant, meaning it is replicated across three physical data centers in that region for high availability.

#### Step 3: Verify the Allocation

Once created, you can see the details (including the actual IP address assigned) using the show command:

```bash
az network public-ip show \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --query "{ipAddress:ipAddress, publicIpAllocationMethod:publicIpAllocationMethod, sku:sku.name}" \
    --output table
```

**Expected Output:**

```
IpAddress      PublicIpAllocationMethod    Sku
-------------  --------------------------  --------
203.0.113.15   Static                      Standard
```

### Summary Table

| Feature | Basic SKU (Avoid) | Standard SKU (Use this) |
|---------|-------------------|-------------------------|
| Security | Open by default | Closed by default (Needs NSG) |
| IP Persistence | Lost on stop (Dynamic) | Kept on stop (Static) |
| Zone Redundancy | No | Yes |
| Pricing | Cheaper | Slightly more expensive |

## Checking Available Resource Groups

To check the available Resource Groups in your Azure subscription using the CLI, you use the list command.

### The Command

```bash
az group list --output table
```

### Understanding the Options

- `az group list`: This is the base command to retrieve all resource groups in your current subscription.
- `--output table`: (Recommended) This formats the output into a clean, readable text table instead of the default JSON format.

### Advanced Filtering (Optional)

If you have many resource groups and want to find a specific one (e.g., only groups in "eastus"), you can use the `--query` parameter:

```bash
az group list --query "[?location=='eastus']" --output table
```

## Troubleshooting: Availability Zones Error

This error occurred because the westus region is an older Azure region that does not support Availability Zones.

Availability Zones (physically separate data centers within a region) are only available in newer or major regions (like westus2, westus3, eastus, etc.).

### The Fix

You can still create a Standard SKU and Static IP in westus, but you must remove the `--zone` parameter. This will create a "Regional" IP (secure and static, but not zone-redundant).

Run this corrected command:

```bash
az network public-ip create \
    --resource-group kml_rg_main-b4eade19d95f4450 \
    --name xfusion-pip \
    --sku Standard \
    --allocation-method Static
```

### What Changed?

- **Removed:** `--zone 1 2 3`
- **Result:** The IP is created successfully in westus. It still has the benefits of the Standard SKU (static IP, secure by default), but it lives generally in the region rather than being replicated across specific data centers.
