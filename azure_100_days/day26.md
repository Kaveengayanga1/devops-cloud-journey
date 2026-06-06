# Day 26 - Azure VNet, Subnet, and Public VM Setup

## 1. Theory & Task Description
The Nautilus DevOps Team requested the Networking Team to set up a new public VNet to support public-facing services. The VNet required public subnets with automatic public IP assignments. Additionally, a new VM needed to be launched to host public applications with SSH access (port 22) open to the internet. 

**Requirements:**
- VNet: `devops-pub-vnet`
- Subnet: `devops-pub-subnet`
- VM: `devops-pub-vm` (Ubuntu 22.04 LTS)
- Open SSH (Port 22)
- All resources must be in the same region (`eastus`).

## 2. Detailed Execution Journey (Mistakes & Management)

During the execution, multiple errors were encountered. Here is the chronological breakdown of every mistake made, the reason, and how it was resolved:

### Mistake 1: Incorrect Azure Command
- **Command Used:** `azure group list`
- **Error:** `-bash: azure: command not found`
- **Reason:** 'azure' is deprecated/invalid or an old syntax. The proper CLI command is `az`.
- **Management:** Replaced with `az group list` and confirmed the Resource Group: `kml_rg_main-a24afe48309b468d` in `eastus`.

*(Creation of VNet, Subnet, NSG, and SSH Rule succeeded here)*

### Mistake 2: Missing VM Size resulting in SkuNotAvailable
- **Command Used:** `az vm create` (without `--size`)
- **Error:** `SkuNotAvailable` - The requested VM size `Standard_DS1_v2` (default) is not available in location `eastus`.
- **Reason:** The default compute size was not available or restricted for this subscription in `eastus`.
- **Management:** Attempted to delete the failed deployment and manually specified `--size Standard_B1s` in later commands.

### Mistake 3: Copy-Paste Error
- **Command Used:** `safe location.`
- **Error:** `-bash: safe: command not found`
- **Reason:** Accidentally pasted terminal output text into the input line.
- **Management:** Terminated the command with `Ctrl+C`.

### Mistake 4: Wrong Resource Group (Authorization Failed)
- **Command Used:** `az vm create` with `--resource-group kml_rg_main-1770b4b549c34f41` and `--name xfusion-vm`
- **Error:** `AuthorizationFailed`
- **Reason:** A wrong resource group name was specified by accident, for which the current user account had no write permissions.
- **Management:** Reverted to using the correct Resource Group: `kml_rg_main-a24afe48309b468d`.

### Mistake 5: Public IP Allocation Method Conflict
- **Command Used:** `az vm create` with `--public-ip-address-allocation dynamic` and `--public-ip-sku Standard`
- **Error:** `StandardAndStandardV2SkuPublicIPAddressesMustBeStatic`
- **Reason:** Azure restricts Standard Public IP SKUs to strictly use `Static` allocation. Dynamic allocation is only supported for the `Basic` SKU.
- **Management:** Changed `--public-ip-address-allocation` from `dynamic` to `static` in subsequent attempts.

### Mistake 6: Cross-Region Reference Error
- **Command Used:** `az vm create` specifying `--location westus` while referencing VNet and NSG.
- **Error:** `InvalidResourceReference`
- **Reason:** Azure does not allow cross-region resource attachments for VNets and NSGs. The networking resources were natively created in `eastus`, but the VM was being requested in `westus`.
- **Management:** Checked the VNet location using `az network vnet show`, confirmed it was `eastus`, and changed the VM `--location` to `eastus`.

### Mistake 7: Stray Typo 
- **Command Used:** `n`
- **Error:** `-bash: n: command not found`
- **Reason:** Accidental keystroke.
- **Management:** Ignored and proceeded.

### Mistake 8: Repeating the Dynamic IP mistake
- **Command Used:** `az vm create` trying location `eastus` but forgot to keep the IP allocation as static, leaving it `dynamic`.
- **Error:** `StandardAndStandardV2SkuPublicIPAddressesMustBeStatic`
- **Reason:** Forgot the previous lesson about Standard SKUs requiring static IP allocation.
- **Management:** Corrected the flag back to `--public-ip-address-allocation static`.

### Mistake 9: Orphaned Resources / Name Conflicts
- **Command Used:** `az vm create` in location `eastus` with all correct flags.
- **Error:** `InvalidResourceLocation` - The resource `devops-pub-vmPublicIP` already exists in location `westus` in the resource group.
- **Reason:** During the earlier failed attempt in `westus` (Mistake 6), Azure actually created the Public IP resource behind the scenes before failing on the VNet reference. Because resources with the same name cannot have different locations in the same RG, creating it in `eastus` failed.
- **Management:** Deleted the orphaned Public IP block in `westus` using `az network public-ip delete --name devops-pub-vmPublicIP`. Additionally, specified a new unique Public IP name explicitly `--public-ip-address devops-pub-vm-ip-new` to entirely avoid namespace collision.

## 3. Final Successful Infrastructure Commands

```bash
# 1. Verify Resource Group
az group list

# 2. Create VNet and Subnet
az network vnet create \
  --resource-group kml_rg_main-a24afe48309b468d \
  --name devops-pub-vnet \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name devops-pub-subnet \
  --subnet-prefix 10.0.1.0/24

# 3. Create NSG and Open SSH Port
az network nsg create \
  --resource-group kml_rg_main-a24afe48309b468d \
  --location eastus \
  --name devops-nsg

az network nsg rule create \
  --resource-group kml_rg_main-a24afe48309b468d \
  --nsg-name devops-nsg \
  --name AllowSSH \
  --protocol tcp \
  --priority 1000 \
  --destination-port-range 22 \
  --access allow

# 4. Create VM successfully
az vm create \
  --resource-group kml_rg_main-a24afe48309b468d \
  --name devops-pub-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --location eastus \
  --vnet-name devops-pub-vnet \
  --subnet devops-pub-subnet \
  --nsg devops-nsg \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address devops-pub-vm-ip-new \
  --public-ip-address-allocation static \
  --public-ip-sku Standard \
  --storage-sku Standard_LRS
```

## 4. Server Credentials & Final State

- **Resource Group:** `kml_rg_main-a24afe48309b468d`
- **Location:** `eastus`
- **Admin Username:** `azureuser`
- **VM Private IP:** `10.0.1.4`
- **VM Public IP:** `168.62.57.1`
- **MAC Address:** `70-A8-A5-3E-24-E9`
- **SSH Key Location:** `/root/.ssh/id_rsa`
- **OS:** Ubuntu 22.04.5 LTS

**Verification Command executed successfully:**
```bash
ssh azureuser@168.62.57.1
# Output: Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)
```
