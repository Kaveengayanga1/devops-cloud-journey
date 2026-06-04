# Day 21: Azure VM with Static Public IP - Policy Error Troubleshooting End-to-End

## 1. Task Description
The Development Team requested a new Azure VM to host an application that needs a stable public endpoint. The VM must be reachable through a static public IP and SSH key authentication.

---

## 2. Requirements
- Create VM: `nautilus-vm`
- OS image: any Ubuntu image (used: `Ubuntu2204`)
- VM size: `Standard_B1s`
- Public IP: static, name `nautilus-pip`
- Generate SSH key pair on `azure-client`
- Associate public key to VM
- Verify SSH access using private key

---

## 3. Core Theory (Before Execution)

### Why Static Public IP?
A static public IP stays unchanged across stop/start cycles (subject to Azure behavior and resource lifecycle), so application endpoints remain stable for users and integrations.

### SSH Key-Based Access
Azure Linux VMs should use SSH public/private key authentication instead of passwords for stronger security and automation-friendly login.

### Azure Policy Impact on Deployments
In restricted/lab subscriptions, organization policies can block disallowed storage configurations. VM creation may fail if OS disk SKU/size violates policy.

### Idempotency and Partial Failures
When `az vm create` fails midway, some resources can remain in a failed/incomplete state. Retrying with changed options can trigger update/immutability errors unless stale resources are cleaned up.

---

## 4. Step-by-Step Execution (Azure CLI)

### Step 1: Generate SSH key pair on `azure-client`
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```
Result: `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub` created successfully.

### Step 2: Confirm target resource group
```bash
az group list --query "[].id" -o tsv
```
Used RG: `kml_rg_main-b934e1c305354a8d`

### Step 3: Create static public IP (`nautilus-pip`)
```bash
az network public-ip create \
  --resource-group kml_rg_main-b934e1c305354a8d \
  --name nautilus-pip \
  --allocation-method Static \
  --sku Standard
```
Succeeded. Public IP assigned: `13.90.37.55`.

### Step 4: First VM creation attempt (failed)
```bash
az vm create \
  --resource-group kml_rg_main-b934e1c305354a8d \
  --name nautilus-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --public-ip-address nautilus-pip \
  --ssh-key-values ~/.ssh/id_rsa.pub
```
Failed with `RequestDisallowedByPolicy` due to disallowed disk configuration.

### Step 5: Second attempt with Standard disk (still failed)
```bash
az vm create \
  --resource-group kml_rg_main-b934e1c305354a8d \
  --name nautilus-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --public-ip-address nautilus-pip \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --storage-sku Standard_LRS
```
Failed with `OperationNotAllowed` (`osDisk.managedDisk.storageAccountType`) because Azure tried to modify an already-created/partial disk path from previous failed deployment.

### Step 6: Cleanup failed resources
`az vm delete` was blocked by policy, so low-level resource cleanup was used.

```bash
az resource delete \
  --resource-group kml_rg_main-b934e1c305354a8d \
  --resource-type "Microsoft.Compute/virtualMachines" \
  --name nautilus-vm
```

```bash
az disk delete \
  --ids /subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/KML_RG_MAIN-B934E1C305354A8D/providers/Microsoft.Compute/disks/nautilus-vm_disk1_a3b7b673a62249338f88639094fd7409 \
  --yes
```

### Step 7: Final successful VM creation with policy-compliant disk settings
```bash
az vm create \
  --resource-group kml_rg_main-b934e1c305354a8d \
  --name nautilus-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --public-ip-address nautilus-pip \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30
```
Succeeded. VM state: `VM running`. Public IP remained `13.90.37.55`.

### Step 8: SSH verification
```bash
ssh -i ~/.ssh/id_rsa azureuser@13.90.37.55
```
- First login succeeded after host key confirmation.
- Re-login succeeded.
- `sudo apt update` ran successfully inside VM.

---

## 5. Errors and Mistakes - What Happened and How It Was Handled

1. **Error:** `RequestDisallowedByPolicy`
   - **Cause:** Default disk configuration from initial `az vm create` violated org policy (non-compliant compute disk settings).
   - **Fix:** Added `--storage-sku Standard_LRS` and explicit compliant disk size (`--os-disk-size-gb 30`).

2. **Error:** `OperationNotAllowed` on `osDisk.managedDisk.storageAccountType`
   - **Cause:** Retrying after partial failure attempted an unsupported disk-type change on a stale deployment artifact.
   - **Fix:** Deleted failed VM resource and orphaned disk, then recreated VM cleanly.

3. **Error during cleanup:** `az vm delete` also triggered policy block
   - **Cause:** Delete path still evaluated against problematic disk state/policy.
   - **Fix:** Used `az resource delete` for VM and `az disk delete --ids ...` for direct disk removal.

4. **Non-error warning:** `[Coming breaking change]` for Public IP zone defaults
   - **Meaning:** Informational warning about future CLI default behavior.
   - **Action:** No immediate fix needed; in production, explicitly set `--zone` when required.

---

## 6. Validation Checklist
- [x] SSH key pair generated on `azure-client`
- [x] Static Public IP `nautilus-pip` created
- [x] VM `nautilus-vm` created with `Standard_B1s`
- [x] OS disk made policy-compliant (`Standard_LRS`, `30 GB`)
- [x] VM reachable via SSH using generated key
- [x] Basic post-login command validation (`sudo apt update`) completed

---

## 7. DevOps/Cloud Engineer Notes
- Always read full Azure error JSON (`code`, `target`, nested `details`).
- In policy-constrained labs, declare disk SKU and disk size explicitly in first attempt.
- After failed provisioning, clean stale resources before retrying to avoid immutability conflicts.
- Keep CLI commands idempotent and auditable for future automation (Bash/Ansible/Terraform).
