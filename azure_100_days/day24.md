# Day 24: Create Azure VM with SSH Key Authentication (Policy-Aware Troubleshooting)

## 1. Task Description
The Nautilus DevOps team must create a new Azure VM that can be securely accessed from the `azure-client` host using SSH key-based authentication.

Target outcome:
- VM name: `xfusion-vm`
- Region: `westus`
- Size: `Standard_B1s`
- Admin user: `azureuser`
- Access method: password-less SSH using generated key pair
- Tooling: Azure CLI

---

## 2. Requirements
- Check whether SSH public key exists on landing host (`azure-client`).
- If missing, generate a new SSH key pair.
- Create VM `xfusion-vm` in `westus` using `azureuser` and SSH public key.
- Handle policy-related deployment failures and retry with compliant storage SKU.
- Verify VM IP address and SSH connectivity.

Environment/resource values used in this execution:
- Subscription ID: `f0c3bcdd-5ce2-4fa0-8cf3-41559747512b`
- Resource Group: `kml_rg_main-1770b4b549c34f41`
- VM: `xfusion-vm`
- Final Public IP: `20.245.200.52`
- Final Private IP: `10.0.0.4`

---

## 3. Core Theory (Must Know Before Execution)

### 3.1 SSH Key-Based Authentication
SSH uses asymmetric cryptography:
- **Private key** stays on client (`~/.ssh/id_rsa`) and must remain secret.
- **Public key** (`~/.ssh/id_rsa.pub`) is placed in VM user’s `authorized_keys`.

During login, server validates challenge-response with the private key, enabling secure password-less login.

### 3.2 Azure Resource Group Scope
All resources for this task must be created in the correct resource group. Using wrong RG names causes lookup failures for VM/IP commands.

### 3.3 Azure Policy Enforcement
Enterprise/lab Azure subscriptions often enforce Azure Policy. In this task, policy blocked non-compliant disk configuration and required:
- Disk size `<= 128 GB`
- Non-premium disk for compute disks
- Storage SKUs such as `Standard_LRS` / `Standard_RAGRS`

If `az vm create` fails with `RequestDisallowedByPolicy`, read policy details and retry with explicit compliant flags.

### 3.4 VM Provisioning and Identity Injection
`az vm create` with `--ssh-key-values ~/.ssh/id_rsa.pub` automatically injects the key into `azureuser` account inside VM.

### 3.5 Verification Layers
Professional validation should include:
1. Infra state (VM created, running)
2. Network state (public/private IP)
3. Access state (SSH login works without password)
4. Guest OS sanity (`apt update` works)

---

## 4. Failed Attempts Log (Explicit)

### Failed Attempt 1: VM create without explicit storage SKU
Command:
```bash
az vm create \
  --resource-group kml_rg_main-1770b4b549c34f41 \
  --name xfusion-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --location westus \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --public-ip-sku Standard
```
Observed failure:
- `DeploymentFailed`
- Nested error: `RequestDisallowedByPolicy`
- Policy message: storage config not allowed; requires non-premium disk configuration.

### Failed Attempt 2: Standard VM delete after failed deployment
Command:
```bash
az vm delete \
  --resource-group kml_rg_main-1770b4b549c34f41 \
  --name xfusion-vm \
  --yes
```
Observed failure:
- `RequestDisallowedByPolicy` (disk-related policy conflict still reported).

### Failed Attempt 3: Wrong resource group during cleanup
Command:
```bash
az resource delete \
  --resource-group kml_rg_main-b934e1c305354a8d \
  --resource-type "Microsoft.Compute/virtualMachines" \
  --name nautilus-vm
```
Observed failure:
- Resource deletion unsuccessful due to wrong RG/target context.

### Failed Attempt 4: Unsupported `az vm delete` arguments
Command:
```bash
az vm delete \
  --resource-group kml_rg_main-1770b4b549c34f41 \
  --name xfusion-vm \
  --yes \
  --delete-disks \
  --delete-vnet
```
Observed failure:
- `unrecognized arguments: --delete-disks --delete-vnet`
- Conclusion: current CLI version does not support those flags in this environment.

### Non-working check due to wrong RG while listing IP
Command:
```bash
az vm list-ip-addresses --name xfusion-vm --resource-group Nautilus-RG --output table
```
Observed issue:
- No valid result for target VM because RG mismatch.

---

## 5. Step-by-Step Execution (Professional Azure CLI Flow)

### Step 1: Check SSH public key existence
```bash
ls -al ~/.ssh/id_rsa.pub
```
Output:
```text
ls: cannot access '/root/.ssh/id_rsa.pub': No such file or directory
```

### Step 2: Generate SSH key pair
```bash
ssh-keygen -t rsa -b 4096 -C "azureuser@xfusion" -f ~/.ssh/id_rsa -N ""
```
Key details from output:
- Public key: `/root/.ssh/id_rsa.pub`
- Fingerprint: `SHA256:XWp4JVUPCHEcRSZCi3xVLMRCGAfwQica/zFsZLA1hUA`

### Step 3: Confirm key generated
```bash
ls -al ~/.ssh/id_rsa.pub
```
Output confirms file exists:
```text
-rw-r--r-- 1 root root 743 Mar 26 03:07 /root/.ssh/id_rsa.pub
```

### Step 4: Validate target resource group
```bash
az group list
```
Confirmed RG exists:
- `kml_rg_main-1770b4b549c34f41`

### Step 5: Retry VM creation with policy-compliant storage
```bash
az vm create \
  --resource-group kml_rg_main-1770b4b549c34f41 \
  --name xfusion-vm \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --location westus \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --public-ip-sku Standard \
  --storage-sku Standard_LRS
```
Successful output highlights:
- `powerState`: `VM running`
- `publicIpAddress`: `20.245.200.52`
- `privateIpAddress`: `10.0.0.4`

### Step 6: Get VM IP information (correct RG)
```bash
az vm list-ip-addresses --name xfusion-vm --resource-group kml_rg_main-1770b4b549c34f41 --output table
```
Output:
```text
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
xfusion-vm        20.245.200.52        10.0.0.4
```

### Step 7: Verify SSH connectivity
```bash
ssh azureuser@20.245.200.52
```
Observed result:
- First-time host key acceptance prompt shown and accepted.
- Login successful to `Ubuntu 22.04.5 LTS` on `xfusion-vm`.

### Step 8: Post-login package index validation
Inside VM:
```bash
sudo apt update
```
Observed result:
- Package repositories updated successfully.
- System reports upgradable packages; VM is healthy and reachable.

---

## 6. Validation Checklist
- [x] SSH key absence detected and new key generated.
- [x] Public key file created on landing host.
- [x] Initial VM creation failure documented with root-cause (`RequestDisallowedByPolicy`).
- [x] Cleanup/deletion failed attempts documented.
- [x] Final VM creation succeeded with `--storage-sku Standard_LRS`.
- [x] VM IP addresses retrieved from correct RG.
- [x] Password-less SSH login to `azureuser` confirmed.
- [x] In-VM command (`sudo apt update`) executed successfully.

---

## 7. Troubleshooting Notes for Future Runs
- If VM create fails with policy error, inspect policy message and force compliant storage via `--storage-sku Standard_LRS`.
- Always verify the exact resource group before `create`, `delete`, and `list-ip-addresses` commands.
- CLI options can differ by version; if a flag is unrecognized, run `az vm delete -h` and adapt commands.
- When needed, investigate failed deployment operations for deeper diagnostics:

```bash
az deployment operation group list \
  --resource-group kml_rg_main-1770b4b549c34f41 \
  --name vm_deploy_fefgAgmdjPKDbDNm52CMGRf0MYH1JoB5 \
  --query "[?provisioningState=='Failed']"
```
