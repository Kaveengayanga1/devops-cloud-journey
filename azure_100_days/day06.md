# Day 06: Create VNet 10.0.0.0/16 (eastus)

## Output (example session)

```console
~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-bec9cf5222e2418f  westus      Succeeded

~ ➜  az network vnet create \
  --name datacenter-vnet \
  --resource-group kml_rg_main-bec9cf5222e2418f \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name datacenter-subnet \
  --subnet-prefix 10.0.0.0/24
```

```json
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"70f427bb-b6e2-4bb7-b8b8-e50d5ded2072\"",
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-bec9cf5222e2418f/providers/Microsoft.Network/virtualNetworks/datacenter-vnet",
    "location": "eastus",
    "name": "datacenter-vnet",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "kml_rg_main-bec9cf5222e2418f",
    "resourceGuid": "a8724a25-acca-4072-a2d6-19fac384b336",
    "subnets": [
      {
        "addressPrefix": "10.0.0.0/24",
        "delegations": [],
        "etag": "W/\"70f427bb-b6e2-4bb7-b8b8-e50d5ded2072\"",
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-bec9cf5222e2418f/providers/Microsoft.Network/virtualNetworks/datacenter-vnet/subnets/datacenter-subnet",
        "name": "datacenter-subnet",
        "privateEndpointNetworkPolicies": "Disabled",
        "privateLinkServiceNetworkPolicies": "Enabled",
        "provisioningState": "Succeeded",
        "resourceGroup": "kml_rg_main-bec9cf5222e2418f",
        "type": "Microsoft.Network/virtualNetworks/subnets"
      }
    ],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}
```

## Theory: Azure Networking Essentials

1) Core Theories of Azure Networking
- Azure Virtual Network (VNet): The foundational, logically isolated private network in Azure enabling secure communication among resources, the internet, and on-premises.
- Address Space and CIDR: VNets use custom private IP ranges in CIDR. Example 10.0.0.0/16 fixes the first 16 bits, covering 10.0.0.0–10.0.255.255 (65,536 addresses).
- Subnets: Segments within a VNet for organization and security (e.g., separate web and database tiers). Note: Azure reserves 5 IPs in every subnet (first three and last two).

2) Implementation Steps using Azure CLI
- Step 1: Log in to Azure

```bash
az login -u <username> -p <password>
```

- Step 2: Create a Resource Group (Optional)

```bash
az group create --name Nautilus-Migration-RG --location eastus
```

- Step 3: Create the Virtual Network and Subnet

```bash
az network vnet create \
  --name datacenter-vnet \
  --resource-group Nautilus-Migration-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name datacenter-subnet \
  --subnet-prefix 10.0.0.0/24
```

3) Verification

- Check VNet details

```bash
az network vnet show --name datacenter-vnet --resource-group Nautilus-Migration-RG --output table
```

- Check Subnet details

```bash
az network vnet subnet list --vnet-name datacenter-vnet --resource-group Nautilus-Migration-RG --output table
```
