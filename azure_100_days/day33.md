# Day 33: Azure Load Balancer for an Nginx VM

## Task Overview

The goal of this task was to place an Azure Load Balancer in front of an existing virtual machine running Nginx, then open HTTP traffic so the sample page could be reached through the load balancer public IP.

### Requirements
1. Create an Azure Load Balancer named `devops-lb`.
2. Create a frontend IP configuration named `devops-lb-ip` and assign a public IP with the same name.
3. Create a backend pool named `devops-backend-pool` and attach the Nginx VM to it.
4. Create a health probe named `devops-health-probe` on port 80.
5. Create a load balancing rule named `devops-lb-rule` for port 80 to port 80.
6. Add an inbound NSG rule to allow HTTP traffic on port 80.
7. Use the eastus region only.

### Portal and Lab Credentials
These were the credentials available in the lab and they were part of the task context.

| Item | Value |
| :--- | :--- |
| Console URL | https://portal.azure.com/azurefreekmlprod.onmicrosoft.com |
| Username | kk_lab_user_main-f7fda99c46274476@azurefreekmlprod.onmicrosoft.com |
| Password | +T%9LNEQ |
| Start Time | Mon May 04 06:06:18 UTC 2026 |
| End Time | Mon May 04 07:06:18 UTC 2026 |

---

## Core Theories

1. **Azure Load Balancer is Layer 4:** It works at the transport layer and distributes TCP or UDP traffic based on the configured rules.
2. **Frontend IP Configuration:** This is the public entry point of the load balancer. Users connect to the public IP associated with this frontend.
3. **Public IP and Frontend Name Matching:** In this task the public IP and frontend IP configuration used the same name, `devops-lb-ip`, which makes the architecture easier to follow.
4. **Backend Pool:** The backend pool is the set of VMs that should receive traffic. Here, the VM running Nginx was added to `devops-backend-pool`.
5. **Health Probe:** The probe checks whether the backend VM is healthy. If the probe fails, the load balancer stops sending traffic to that backend.
6. **Load Balancing Rule:** A rule maps frontend port 80 to backend port 80 and ties together the frontend, backend pool, and probe.
7. **NSG as a Virtual Firewall:** Even if the load balancer is configured correctly, the VM still needs an NSG rule to allow inbound HTTP traffic.
8. **Resource Name vs Resource ID:** Azure CLI commands often expect a resource name, not the full ARM resource ID. Mixing those up is a common source of errors.
9. **IP Configuration Name Discovery:** NICs sometimes use a different internal IP configuration name than expected, so it is safer to query the NIC first instead of assuming `ipconfig1`.
10. **Asymmetric Troubleshooting:** When a command fails, check the exact object name, the resource group, and the internal child resource name before retrying.

---

## Step-by-Step Azure CLI Procedure

### 1. Sign in to Azure
```bash
az login -u kk_lab_user_main-f7fda99c46274476@azurefreekmlprod.onmicrosoft.com -p +T%9LNEQ
```

### 2. Confirm the resource group
```bash
az group list
```

Expected resource group:
```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-f7fda99c46274476",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

### 3. Create the public IP
```bash
az network public-ip create \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --name devops-lb-ip \
    --sku Standard \
    --location eastus
```

Observed output:
```json
{
  "publicIp": {
    "ddosSettings": {
      "protectionMode": "VirtualNetworkInherited"
    },
    "etag": "W/\"f1ad07a8-28df-4606-a8f5-9b7192b2ee07\"",
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/publicIPAddresses/devops-lb-ip",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "20.25.64.245",
    "ipTags": [],
    "location": "eastus",
    "name": "devops-lb-ip",
    "provisioningState": "Succeeded",
    "publicIPAddressVersion": "IPv4",
    "publicIPAllocationMethod": "Static",
    "resourceGroup": "kml_rg_main-f7fda99c46274476",
    "resourceGuid": "16acf890-21c9-422a-a569-d56eb69b0474",
    "sku": {
      "name": "Standard",
      "tier": "Regional"
    },
    "type": "Microsoft.Network/publicIPAddresses"
  }
}
```

### 4. Create the load balancer
```bash
az network lb create \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --name devops-lb \
    --sku Standard \
    --location eastus \
    --public-ip-address devops-lb-ip \
    --frontend-ip-name devops-lb-ip
```

Observed output:
```json
{
  "loadBalancer": {
    "backendAddressPools": [
      {
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/loadBalancers/devops-lb/backendAddressPools/devops-lbbepool",
        "name": "devops-lbbepool",
        "properties": {
          "loadBalancerBackendAddresses": [],
          "provisioningState": "Succeeded"
        },
        "resourceGroup": "kml_rg_main-f7fda99c46274476",
        "type": "Microsoft.Network/loadBalancers/backendAddressPools"
      }
    ],
    "frontendIPConfigurations": [
      {
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/loadBalancers/devops-lb/frontendIPConfigurations/devops-lb-ip",
        "name": "devops-lb-ip",
        "properties": {
          "privateIPAllocationMethod": "Dynamic",
          "provisioningState": "Succeeded",
          "publicIPAddress": {
            "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/publicIPAddresses/devops-lb-ip",
            "resourceGroup": "kml_rg_main-f7fda99c46274476"
          }
        },
        "resourceGroup": "kml_rg_main-f7fda99c46274476",
        "type": "Microsoft.Network/loadBalancers/frontendIPConfigurations"
      }
    ],
    "inboundNatPools": [],
    "inboundNatRules": [],
    "loadBalancingRules": [],
    "outboundRules": [],
    "probes": [],
    "provisioningState": "Succeeded"
  }
}
```

### 5. Create the backend pool
```bash
az network lb address-pool create \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --lb-name devops-lb \
    --name devops-backend-pool
```

### 6. Create the health probe
```bash
az network lb probe create \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --lb-name devops-lb \
    --name devops-health-probe \
    --protocol http \
    --port 80 \
    --path /
```

### 7. Create the load balancing rule
```bash
az network lb rule create \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --lb-name devops-lb \
    --name devops-lb-rule \
    --protocol tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name devops-lb-ip \
    --backend-pool-name devops-backend-pool \
    --probe-name devops-health-probe
```

### 8. Find the VM and NIC details
```bash
az vm list --query "[].{VM_Name:name, ResourceGroup:resourceGroup}" --output table
```

Observed result:
```text
VM_Name    ResourceGroup
---------  ----------------------------
devops-vm  KML_RG_MAIN-F7FDA99C46274476
```

```bash
az vm list --query "[].{VM:name, RG:resourceGroup, NIC:networkProfile.networkInterfaces[0].id}" --output table
```

Observed result:
```text
VM         RG                            NIC
---------  ----------------------------  ------------------------------------------------------------------------------------------------------------------------------------------------------------
devops-vm  KML_RG_MAIN-F7FDA99C46274476  /subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/networkInterfaces/devops-vmVMNic
```

### 9. First attempt to add the NIC to the backend pool
```bash
az network nic ip-config address-pool add \
    --address-pool devops-backend-pool \
    --ip-config-name ipconfig1 \
    --nic-name /subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/networkInterfaces/devops-vmVMNic \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --lb-name devops-lb
```

Error received:
```text
(ResourceNotFound) The Resource 'Microsoft.Network/networkInterfaces/subscriptions' under resource group 'kml_rg_main-f7fda99c46274476' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix
Code: ResourceNotFound
Message: The Resource 'Microsoft.Network/networkInterfaces/subscriptions' under resource group 'kml_rg_main-f7fda99c46274476' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix
```

### 10. Second attempt with NIC name only
```bash
az network nic ip-config address-pool add \
    --address-pool devops-backend-pool \
    --ip-config-name ipconfig1 \
    --nic-name devops-vmVMNic \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --lb-name devops-lb
```

Error received:
```text
ResourceNotFoundError
```

### 11. Query the NIC to find the real IP configuration name
```bash
az network nic show \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --name devops-vmVMNic \
    --query "ipConfigurations[0].name" \
    --output tsv
```

Observed result:
```text
ipconfigdevops-vm
```

### 12. Add the NIC using the correct IP configuration name
```bash
az network nic ip-config address-pool add \
    --address-pool devops-backend-pool \
    --ip-config-name ipconfigdevops-vm \
    --nic-name devops-vmVMNic \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --lb-name devops-lb
```

Successful output:
```json
{
  "etag": "W/\"593a4607-726b-4d3b-a28f-331fd72eaab5\"",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/networkInterfaces/devops-vmVMNic/ipConfigurations/ipconfigdevops-vm",
  "loadBalancerBackendAddressPools": [
    {
      "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/loadBalancers/devops-lb/backendAddressPools/devops-backend-pool",
      "resourceGroup": "kml_rg_main-f7fda99c46274476"
    }
  ],
  "name": "ipconfigdevops-vm",
  "primary": true,
  "privateIPAddress": "10.0.0.4",
  "privateIPAddressVersion": "IPv4",
  "privateIPAllocationMethod": "Dynamic",
  "provisioningState": "Succeeded",
  "publicIPAddress": {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/publicIPAddresses/devops-vmPublicIP",
    "resourceGroup": "kml_rg_main-f7fda99c46274476"
  },
  "resourceGroup": "kml_rg_main-f7fda99c46274476",
  "subnet": {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/virtualNetworks/devops-vmVNET/subnets/devops-vmSubnet",
    "resourceGroup": "kml_rg_main-f7fda99c46274476"
  },
  "type": "Microsoft.Network/networkInterfaces/ipConfigurations"
}
```

### 13. Find the NSG and add the inbound HTTP rule
```bash
az network nsg list \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --query "[].name" \
    --output tsv
```

Observed result:
```text
devops-vmNSG
```

```bash
az network nsg rule create \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --nsg-name devops-vmNSG \
    --name allow-http-80 \
    --priority 100 \
    --destination-port-ranges 80 \
    --access Allow \
    --protocol Tcp \
    --direction Inbound
```

Successful output:
```json
{
  "access": "Allow",
  "destinationAddressPrefix": "*",
  "destinationAddressPrefixes": [],
  "destinationPortRange": "80",
  "destinationPortRanges": [],
  "direction": "Inbound",
  "etag": "W/\"9f7493ec-9190-4c8b-9653-38179188feb9\"",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f7fda99c46274476/providers/Microsoft.Network/networkSecurityGroups/devops-vmNSG/securityRules/allow-http-80",
  "name": "allow-http-80",
  "priority": 100,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "kml_rg_main-f7fda99c46274476",
  "sourceAddressPrefix": "*",
  "sourceAddressPrefixes": [],
  "sourcePortRange": "*",
  "sourcePortRanges": [],
  "type": "Microsoft.Network/networkSecurityGroups/securityRules"
}
```

### 14. Verify the public IP and test HTTP access
```bash
az network public-ip show \
    --resource-group kml_rg_main-f7fda99c46274476 \
    --name devops-lb-ip \
    --query "ipAddress" \
    --output tsv
```

Observed result:
```text
20.25.64.245
```

```bash
curl -I 20.25.64.245
```

Observed result:
```text
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 04 May 2026 06:26:56 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Mon, 04 May 2026 06:08:50 GMT
Connection: keep-alive
ETag: "69f837f2-264"
Accept-Ranges: bytes
```

---

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. Full NIC resource ID was passed to `--nic-name`
* **Mistake:** The first `az network nic ip-config address-pool add` command used the full ARM resource ID instead of the NIC name.
* **Why it happened:** The CLI parameter `--nic-name` expects only the resource name when `--resource-group` is already supplied. Passing the full path caused Azure to interpret the input incorrectly.
* **Fix:** Re-ran the command with `--nic-name devops-vmVMNic`.

### 2. Wrong IP configuration name was assumed
* **Mistake:** The command used `--ip-config-name ipconfig1`.
* **Why it happened:** The VM NIC did not actually have that child resource name. The real IP configuration name was `ipconfigdevops-vm`.
* **Fix:** Queried the NIC with `az network nic show --query "ipConfigurations[0].name" --output tsv` and used the returned value.

### 3. Case and internal naming needed verification
* **Mistake:** The VM and resource group outputs showed names in different cases, which could be confusing during troubleshooting.
* **Why it happened:** Azure resources are often displayed with varying case in outputs and queries, and it is easy to focus on the visible VM name while missing a child resource name.
* **Fix:** Verified the exact NIC name and exact IP configuration name before retrying.

### 4. The load balancer rule alone was not enough
* **Mistake:** It would have been incorrect to assume that creating the load balancer and rule was sufficient.
* **Why it happened:** The VM still had to permit inbound port 80 traffic through its NSG.
* **Fix:** Created the `allow-http-80` inbound NSG rule on `devops-vmNSG`.

---

## Final Verification

| Item | Value | Status |
| :--- | :--- | :--- |
| Resource Group | `kml_rg_main-f7fda99c46274476` | Success |
| Load Balancer | `devops-lb` | Success |
| Public IP | `devops-lb-ip` | Success |
| Public IP Address | `20.25.64.245` | Success |
| Backend Pool | `devops-backend-pool` | Success |
| Probe | `devops-health-probe` | Success |
| Rule | `devops-lb-rule` | Success |
| NIC | `devops-vmVMNic` | Success |
| IP Configuration | `ipconfigdevops-vm` | Success |
| NSG | `devops-vmNSG` | Success |
| NSG Rule | `allow-http-80` | Success |

## Technical Summary

This task demonstrated how a standard Azure Load Balancer is assembled from a frontend public IP, a backend pool, a health probe, and a load balancing rule. The VM was then attached to the backend pool through its NIC IP configuration, and the NSG was updated to allow HTTP traffic. The final `curl -I` test returned `HTTP/1.1 200 OK`, confirming that traffic reached the Nginx server successfully.
