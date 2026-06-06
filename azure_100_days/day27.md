# Day 27: Configuring Private Virtual Network and VM in Azure

## Tasks and Objective
The Nautilus DevOps team is expanding their Azure infrastructure and requires the setup of a private Virtual Network (VNet) along with a subnet. This VNet and subnet configuration ensures that resources remains isolated from external networks and can only communicate within the VNet. 

### Requirements:
- **VNet Name**: `devops-priv-vnet`
- **Subnet Name**: `devops-priv-subnet`
- **VM Name**: `devops-priv-vm` (Ubuntu 22.04, Size: Standard_B1s)
- **NSG Name**: `devops-priv-nsg`
- **Region**: `centralus`
- **SSH Rule**: Allow inbound SSH (Port 22) only from within the VNet CIDR (`10.0.0.0/16`).

---

## Theory and Concepts

### 1. Azure Virtual Network (VNet)
A VNet is a representation of your own network in the cloud. It is a logical isolation of the Azure cloud dedicated to your subscription. You can use VNets to provision and manage virtual private networks (VPNs) in Azure.

### 2. Subnets
Subnets allow you to segment the virtual network into one or more sub-networks and allocate a portion of the virtual network's address space to each subnet. This improves security and efficiency.

### 3. Network Security Group (NSG)
An NSG contains security rules that allow or deny inbound network traffic to, or outbound network traffic from, several types of Azure resources. For each rule, you can specify source and destination, port, and protocol.

### 4. CIDR Block (10.0.0.0/16)
Classless Inter-Domain Routing (CIDR) is a method for allocating IP addresses and IP routing. `10.0.0.0/16` means the first 16 bits are fixed, allowing for 65,536 IP addresses.

### 5. Private VM Isolation
By not assigning a Public IP address to the VM, it becomes "private." It cannot be reached directly from the internet. Access is typically gained through a Bastion host, a VPN, or another VM within the same VNet.

---

## Credentials Used
- **Portal URL**: https://portal.azure.com
- **Username**: `kk_lab_user_main-719ce811e86f44be@azurefreekmlprod.onmicrosoft.com`
- **Password**: `VLUt3a^2`
- **Resource Group**: `kml_rg_main-719ce811e86f44be`
- **Subscription ID**: `f0c3bcdd-5ce2-4fa0-8cf3-41559747512b`

---

## Implementation (Azure CLI)

### 1. Create the Virtual Network and Subnet
```bash
az network vnet create \
    --resource-group kml_rg_main-719ce811e86f44be \
    --name devops-priv-vnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name devops-priv-subnet \
    --subnet-prefix 10.0.1.0/24 \
    --location centralus
```

### 2. Create the Network Security Group (NSG)
```bash
az network nsg create \
    --resource-group kml_rg_main-719ce811e86f44be \
    --name devops-priv-nsg \
    --location centralus
```

### 3. Create the NSG Rule for Internal SSH
```bash
az network nsg rule create \
    --resource-group kml_rg_main-719ce811e86f44be \
    --nsg-name devops-priv-nsg \
    --name AllowVnetSSH \
    --priority 100 \
    --source-address-prefixes 10.0.0.0/16 \
    --source-port-ranges '*' \
    --destination-address-prefixes 10.0.0.0/16 \
    --destination-port-ranges 22 \
    --access Allow \
    --protocol Tcp \
    --direction Inbound
```

### 4. Create the Virtual Machine (Private)
```bash
az vm create \
  --resource-group kml_rg_main-719ce811e86f44be \
  --name devops-priv-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --location centralus \
  --vnet-name devops-priv-vnet \
  --subnet devops-priv-subnet \
  --nsg devops-priv-nsg \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address ""
```

---

## Mistakes Made and Fixes

### 1. Resource Group Ambiguity
- **Mistake**: In the initial planning, a generic name like `DevOps-RG` was used in thoughts, but the actual lab environment required a specific pre-existing resource group `kml_rg_main-719ce811e86f44be`.
- **Reason**: Lab environments usually have locked-down permissions where you must use the provided RG.
- **Fix**: Used `az group list` to identify the correct Resource Group name and updated the commands accordingly.

### 2. VNet Location Mismatch
- **Mistake**: The Resource Group was in `eastus`, but the requirement strictly mentioned `centralus`.
- **Reason**: Overlooking the requirement notes while checking the environment.
- **Fix**: Explicitly added `--location centralus` to every resource creation command to ensure compliance.

### 3. SSH Connectivity Logic
- **Mistake**: Forgetting that a VM without a Public IP cannot be reached via SSH from a local machine/terminal outside Azure.
- **Reason**: Standard VM creation often defaults to having a public IP for easy access.
- **Fix**: Set `--public-ip-address ""` and recognized that testing must be done from another resource within the same VNet.

---

## Verification Output
```json
{
  "fqdns": "",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-719ce811e86f44be/providers/Microsoft.Compute/virtualMachines/devops-priv-vm",
  "location": "centralus",
  "privateIpAddress": "10.0.1.4",
  "publicIpAddress": "",
  "powerState": "VM running",
  "resourceGroup": "kml_rg_main-719ce811e86f44be"
}
```
