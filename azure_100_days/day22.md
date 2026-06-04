# Day 22: Azure VM + Nginx via cloud-init (Policy & CLI Troubleshooting)

## 1. Task Description
The Nautilus DevOps Team needed an Azure VM to host a web server for a critical application. The VM had to be provisioned with Ubuntu, auto-install and start Nginx during first boot, and allow HTTP traffic from the internet.

---

## 2. Requirements
- VM name: `devops-vm`
- Image: Ubuntu (`Ubuntu2204` used)
- Bootstrap script: use custom data (`cloud-init.txt`) to:
  - install Nginx
  - start/enable Nginx
- NSG rule: allow inbound HTTP on port `80`
- Validate public IP retrieval after deployment

---

## 3. Core Theory (Before Execution)

### Resource Group
Azure resources are grouped in a logical container (`kml_rg_main-89fee4eef6e5419c`) for lifecycle and policy management.

### cloud-init / Custom Data
`--custom-data cloud-init.txt` passes first-boot configuration to Linux VM. It enables immutable, repeatable server bootstrap without manual SSH steps.

### Azure Policy Enforcement
In restricted/lab subscriptions, policy can block resource creation if disk SKU/size/configuration violates organizational constraints.

### VM SKU Capacity Constraints
Region-specific capacity can reject default VM sizes (`SkuNotAvailable`). You must select another allowed size.

### Idempotency vs Partial State
When `az vm create` fails, partial resources can remain. Retrying with changed flags may trigger update/immutability errors unless state is handled correctly.

---

## 4. Step-by-Step Execution (Azure CLI)

### Step 1: Prepare cloud-init
```bash
vi cloud-init.txt
```
Used content:
```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
runcmd:
  - systemctl enable nginx
  - systemctl start nginx
```

### Step 2: Confirm target resource group
```bash
az group list
```
Confirmed RG:
- `kml_rg_main-89fee4eef6e5419c` in `eastus`

### Step 3: First VM create attempt (failed: capacity)
```bash
az vm create \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt \
  --public-ip-sku Standard
```
Result:
- SSH keys generated under `/root/.ssh/`
- Deployment failed with `SkuNotAvailable` for default size path (`Standard_DS1_v2`) in `eastus`

### Step 4: Retry with smaller size (failed: policy)
```bash
az vm create \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt \
  --public-ip-sku Standard \
  --size Standard_B1s
```
Result:
- Failed with `RequestDisallowedByPolicy`
- Policy required non-premium compliant storage configuration

### Step 5: Wrong flag attempt
```bash
az vm create \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt \
  --public-ip-sku Standard \
  --size Standard_B1s \
  --os-disk-sku Standard_LRS
```
Result:
- `unrecognized arguments: --os-disk-sku Standard_LRS`

### Step 6: Inspect existing resources
```bash
az resource list --resource-group kml_rg_main-89fee4eef6e5419c --output table
```
Observed existing partial resources:
- `devops-vmPublicIP`
- `devops-vmNSG`
- `devops-vmVNET`
- `devops-vmVMNic`
- `devops-vm`

### Step 7: Retry with correct storage flag (intermediate failure)
```bash
az vm create \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.txt \
  --public-ip-sku Standard \
  --size Standard_B1s \
  --storage-sku Standard_LRS
```
Result:
- Failed with `OperationNotAllowed` on `osDisk.managedDisk.storageAccountType`
- Cause: attempted disk storage type change on an existing failed VM/disk state

### Step 8: Deletion path troubleshooting
```bash
az vm delete \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --yes
```
Result:
- Delete was blocked by same policy (`RequestDisallowedByPolicy`)

Used fallback deletion:
```bash
az resource delete \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --resource-type "Microsoft.Compute/virtualMachines"
```

### Step 9: Final successful VM creation
```bash
az vm create --resource-group kml_rg_main-89fee4eef6e5419c --name devops-vm --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys --custom-data cloud-init.txt --public-ip-sku Standard --size Standard_B1s --storage-sku Standard_LRS
```
Result:
- VM created successfully
- Power state: `VM running`
- Public IP: `20.124.204.137`

### Step 10: Open HTTP port 80
```bash
az vm open-port \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --port 80 \
  --priority 1001
```
Result:
- NSG rule `open-port-80` created successfully

### Step 11: Verify public IP
```bash
az vm show \
  --resource-group kml_rg_main-89fee4eef6e5419c \
  --name devops-vm \
  --show-details \
  --query publicIps \
  --output tsv
```
Output:
- `20.124.204.137`

---

## 5. Errors and Mistakes (Clearly Documented)

1. **`SkuNotAvailable` on first `az vm create`**
   - **What happened:** Azure default size path used unavailable capacity (`Standard_DS1_v2`) in `eastus`.
   - **Fix:** Explicitly selected another size (`--size Standard_B1s`).

2. **`RequestDisallowedByPolicy` after adding size**
   - **What happened:** Subscription policy blocked disk/storage settings (non-compliant configuration).
   - **Fix:** Use policy-compliant storage option (`--storage-sku Standard_LRS`).

3. **Wrong CLI argument used (`--os-disk-sku`)**
   - **What happened:** Command failed immediately with `unrecognized arguments`.
   - **Mistake type:** Parameter mismatch with current Azure CLI command syntax.
   - **Fix:** Replaced with `--storage-sku Standard_LRS`.

4. **`OperationNotAllowed` while changing disk storage type**
   - **What happened:** Existing failed VM/disk state prevented in-place managed disk storage type transition.
   - **Fix:** Handle stale resource state, remove VM resource, and re-run create in clean state.

5. **`az vm delete` also blocked by policy**
   - **What happened:** Standard delete flow got blocked by policy evaluation against problematic disk state.
   - **Fix:** Used `az resource delete` to remove VM resource directly, then recreated VM.

6. **Process-level mistake: assuming retries always self-heal**
   - **What happened:** Multiple retries on partially failed resources produced different errors.
   - **Lesson:** In policy-constrained labs, inspect resource state and clean stale artifacts before retrying.

---

## 6. Validation Checklist
- [x] `cloud-init.txt` created with Nginx install/start commands
- [x] VM `devops-vm` created in RG `kml_rg_main-89fee4eef6e5419c`
- [x] VM size adjusted to `Standard_B1s` after capacity issue
- [x] Storage made policy-compliant with `--storage-sku Standard_LRS`
- [x] Port `80` opened via NSG rule (`open-port-80`)
- [x] Public IP retrieved successfully: `20.124.204.137`
- [x] cloud-init file content verified with `cat cloud-init.txt`

---

## 7. Professional DevOps/Cloud Engineer Notes
- Prefer explicit VM options in policy-controlled environments (`--size`, `--storage-sku`).
- Read nested error JSON (`code`, `target`, `details`) before retrying.
- Treat failed creates as stateful events; partial resources may need cleanup.
- Keep cloud-init minimal, idempotent, and version-controlled.
- For production, convert this flow to IaC (Terraform/Bicep) and pipeline-based deployment.
