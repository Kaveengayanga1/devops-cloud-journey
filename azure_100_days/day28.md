# Day 28: Deploying and Troubleshooting Nginx on Azure VM

## Task Overview
The goal was to deploy an Nginx web server on an Azure VM named `nautilus-vm` within a public VNet `nautilus-vnet`. The task involved verifying network configuration, attaching a public IP, ensuring accessibility on port 80, and installing Nginx.

## Core Theories

### 1. Azure Virtual Network (VNet)
A VNet is a representation of your own network in the cloud. It is a logical isolation of the Azure cloud dedicated to your subscription. For internet communication, a subnet within the VNet must have a path to an Internet Gateway.

### 2. Public IP Address (PIP)
A Public IP address allows Azure resources to communicate with the internet and other public-facing Azure services. It's essential for accessing the VM from outside the Azure network.

### 3. Network Interface Card (NIC)
The NIC is the interconnection between a VM and the underlying software network. A Public IP is associated with a NIC's IP configuration.

### 4. Network Security Group (NSG)
An NSG contains security rules that allow or deny inbound or outbound network traffic. To allow web traffic, an inbound rule for Port 80 (HTTP) must be configured.

### 5. Route Table & User Defined Routes (UDR)
Route Tables control where network traffic is directed. A default route `0.0.0.0/0` with a Next Hop of `Internet` is required for outbound internet access (e.g., for installing packages via `apt`).

### 6. Nginx Web Server
Nginx is a popular open-source web server used for hosting websites, acting as a reverse proxy, and handles HTTP traffic on port 80 by default.

---

## Used Credentials

| Name | Value |
| :--- | :--- |
| Azure Portal URL | https://portal.azure.com/azurefreekmlprod.onmicrosoft.com |
| Azure User Name | kk_lab_user_main-60023e5050a94d2b@azurefreekmlprod.onmicrosoft.com |
| Azure Password | 26FYE&$= |
| Azure Client ID | 22ffe63e-886a-4553-bf38-785815f10616 |
| Azure Client Secret | m0i8Q~45jA_OuEqIilqEmtfXFIhuZla_73Fhubdk |
| VM Username | devopsadmin |
| VM Password | Nautilus@Cloud2026 |

---

## Mistakes Made & Fixes

### 1. Resource Not Found when Updating NIC
- **Mistake:** Attempted to update the NIC IP configuration using a full resource ID instead of the simple name, and used an incorrect IP configuration name (`ipconfig1`).
- **Error:** `(ResourceNotFound) The Resource 'Microsoft.Network/networkInterfaces/subscriptions' under resource group 'kml_rg_main-60023e5050a94d2b' was not found.`
- **Fix:** Used the correct NIC name `nautilus-vmVMNic` and the actual IP configuration name `ipconfignautilus-vm` found via inspection.

### 2. NSG Rule Priority Conflict
- **Mistake:** Tried to create an inbound rule `AllowHTTPInbound` with priority 100, which was already assigned to another rule.
- **Error:** `(SecurityRuleConflict) Security rule set-port-80 conflicts with rule AllowHTTPInbound. Rules cannot have the same Priority and Direction.`
- **Fix:** Identified that priority 100 was taken. (Note: In practice, one would use a different priority like 101 or modify the existing rule).

### 3. Outbound Internet Access Blocked by Route Table
- **Mistake:** The subnet had a Route Table `nautilus-rtb` with a route `Block-Internet` that set the next hop for `0.0.0.0/0` to `None`.
- **Reason:** This prevented the VM from reaching the internet to download Nginx.
- **Fix:** Updated the existing `Block-Internet` route to change its `next-hop-type` to `Internet`.

---

## Implementation Output

```bash
~ ➜ az group list
[
{
"id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-60023e5050a94d2b",
"location": "eastus",
"managedBy": null,
"name": "kml_rg_main-60023e5050a94d2b",
"properties": {
"provisioningState": "Succeeded"
},
"tags": null,
"type": "Microsoft.Resources/resourceGroups"
}
]

~ ➜ az login --service-principal \
-u "22ffe63e-886a-4553-bf38-785815f10616" \
-p "m0i8Q~45jA_OuEqIilqEmtfXFIhuZla_73Fhubdk" \
--tenant "azurefreekmlprod.onmicrosoft.com"

~ ➜ az vm user update \
--resource-group kml_rg_main-60023e5050a94d2b \
--name nautilus-vm \
--username devopsadmin \
--password "Nautilus@Cloud2026"

~ ➜ az network nic ip-config update \
--resource-group kml_rg_main-60023e5050a94d2b \
--nic-name nautilus-vmVMNic \
--name ipconfignautilus-vm \
--public-ip-address nautilus-pip

~ ➜ az network route-table route update \
--resource-group kml_rg_main-60023e5050a94d2b \
--route-table-name nautilus-rtb \
--name Block-Internet \
--next-hop-type Internet

~ ➜ ssh devopsadmin@20.184.147.146
devopsadmin@nautilus-vm:~$ sudo apt-get update
devopsadmin@nautilus-vm:~$ sudo apt-get install nginx -y
devopsadmin@nautilus-vm:~$ sudo systemctl start nginx
devopsadmin@nautilus-vm:~$ sudo systemctl enable nginx
```
