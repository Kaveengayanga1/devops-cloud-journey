# Day 05: Create a VNet with 192.168.0.0/24 (southcentralus)

## Output (example session)

```console
~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-5b9a1a0b7e774df5  westus      Succeeded

~ ➜  az network vnet create \
  --name datacenter-vnet \
  --resource-group kml_rg_main-5b9a1a0b7e774df5 \
  --location southcentralus \
  --address-prefix 192.168.0.0/24 \
  --subnet-name front-end-subnet \
  --subnet-prefix 192.168.0.0/26
```

```json
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "192.168.0.0/24"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"724b9449-3411-4ce2-adda-e6de1b411873\"",
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-5b9a1a0b7e774df5/providers/Microsoft.Network/virtualNetworks/datacenter-vnet",
    "location": "southcentralus",
    "name": "datacenter-vnet",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "kml_rg_main-5b9a1a0b7e774df5",
    "resourceGuid": "bf833762-69aa-4613-96a6-b0ead0a486c5",
    "subnets": [
      {
        "addressPrefix": "192.168.0.0/26",
        "delegations": [],
        "etag": "W/\"724b9449-3411-4ce2-adda-e6de1b411873\"",
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-5b9a1a0b7e774df5/providers/Microsoft.Network/virtualNetworks/datacenter-vnet/subnets/front-end-subnet",
        "name": "front-end-subnet",
        "privateEndpointNetworkPolicies": "Disabled",
        "privateLinkServiceNetworkPolicies": "Enabled",
        "provisioningState": "Succeeded",
        "resourceGroup": "kml_rg_main-5b9a1a0b7e774df5",
        "type": "Microsoft.Network/virtualNetworks/subnets"
      }
    ],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}
```

## Theory: Azure IP Addressing

- CIDR Notation (/24): A /24 provides 256 total IPs. Subnets inside a /24 must be the same size (/24) or smaller (e.g., /25, /26, /27).
- Azure Reserved IPs: In every subnet, Azure reserves 5 IPs — the first four and the last IP of the subnet (network address, default gateway, Azure infrastructure, and the last address).
- Regional Scope: A VNet is bound to a single region. With `southcentralus`, resources you place in this VNet (VMs, Load Balancers, etc.) must also be in `southcentralus`.

## Implementation (Azure CLI)

1) Create the Resource Group

```bash
az group create --name datacenter-rg --location southcentralus
```

2) Create the Virtual Network and Subnet

```bash
az network vnet create \
  --name datacenter-vnet \
  --resource-group datacenter-rg \
  --location southcentralus \
  --address-prefix 192.168.0.0/24 \
  --subnet-name front-end-subnet \
  --subnet-prefix 192.168.0.0/26
```

## Configuration Summary

| Parameter               | Value               | Details                                                     |
|-------------------------|---------------------|-------------------------------------------------------------|
| VNet Name               | datacenter-vnet     | Logical name for your virtual data center.                  |
| Region                  | southcentralus      | Region where the VNet is created.                           |
| VNet CIDR               | 192.168.0.0/24      | Total range: 192.168.0.0 – 192.168.0.255.                   |
| Subnet Name             | front-end-subnet    | Default subnet for workloads.                               |
| Subnet CIDR             | 192.168.0.0/26      | 64 total IPs; 59 usable (5 reserved by Azure).              |
| Usable IPs (single /24) | 251                 | If using a single /24 subnet: 256 total − 5 reserved.       |
