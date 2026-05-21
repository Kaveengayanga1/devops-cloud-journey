# Day 04: Azure Virtual Network (VNet)

To create a Virtual Network (VNet) in Azure, you must first understand its role as the fundamental building block of your private network in the cloud. It provides isolation, segmentation, and communication for your resources.

## Necessary Theory: Core Concepts

- Address Space (CIDR): When you create a VNet, you assign a custom private IP address space using CIDR notation (e.g., 10.0.0.0/16).
- Prefix (/16): This determines the number of available IP addresses. A /16 provides 65,536 addresses, while a /24 provides 256.
- Overlapping: Never use an address range that overlaps with your on-premises network or other VNets you plan to connect to (peer) in the future.
- Subnets: You divide your VNet into one or more subnets. This allows you to secure and group resources (e.g., a "Web" subnet and a "Database" subnet).
- Azure Reserved IPs: In every subnet, Azure reserves 5 IP addresses (the first four and the last one) for protocol and management purposes.
- Regions: A VNet is scoped to a single Azure region (like eastus). While the VNet stays in one region, you can connect it to VNets in other regions via Global VNet Peering.

## Implementation Steps

You can create the `devops-vnet` using various tools. Below is the command for the Azure CLI, which is the standard for DevOps automation.

### 1. Create a Resource Group
Resources in Azure must reside in a logical container called a Resource Group.

```bash
az group create --name devops-rg --location eastus
```

### 2. Create the Virtual Network
This command creates the VNet with a common private address space (10.0.0.0/16).

```bash
az network vnet create \
  --name devops-vnet \
  --resource-group devops-rg \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name default-subnet \
  --subnet-prefix 10.0.1.0/24
```

## Summary Table: Configuration Details

| Property     | Value         | Description                                                     |
|--------------|---------------|-----------------------------------------------------------------|
| Name         | devops-vnet   | The unique identifier for your network.                         |
| Region       | eastus        | The physical location of the network infrastructure.            |
| IPv4 CIDR    | 10.0.0.0/16   | The total range of private IPs for the entire VNet.             |
| Subnet CIDR  | 10.0.1.0/24   | A segment of the VNet for actual resource deployment.           |

## Find Existing Resource Groups

You can use the Azure Portal for a visual overview or command-line tools for automation.

### Method 1: Azure Portal (Visual)
The portal is best for a quick check or when you need to see associated resources visually.
- Log in to the Azure Portal.
- In the top search bar, type "Resource groups" and select it from the services list.
- You will see a list of all resource groups you have access to. You can filter this list by subscription or location using the dropdowns at the top.

### Method 2: Azure CLI (DevOps/Automation)
The CLI is highly efficient for listing resources in a script-friendly format.

List all groups:
```bash
az group list --output table
```

Search for a specific group by name:
```bash
az group show --name devops-vnet-rg
```

Filter by location:
```bash
az group list --query "[?location=='eastus']" --output table
```

### Method 3: Azure PowerShell
If you are working in a Windows environment or using the Az module:

List all groups:
```powershell
Get-AzResourceGroup
```

Find a group by a specific name pattern:
```powershell
Get-AzResourceGroup -Name "devops*"
```

## Comparison Summary

| Feature         | Azure Portal            | Azure CLI           | PowerShell            |
|-----------------|-------------------------|---------------------|-----------------------|
| Interface       | GUI                     | Command Line        | Command Line          |
| Best For        | Manual inspection       | Automation/Linux/macOS | Windows Automation |
| Search Command  | Search Bar              | az group list       | Get-AzResourceGroup   |
