# Day 33: Create an Azure Ubuntu VM with Nginx using cloud-init

## 1. Task Description
The Nautilus DevOps Team needs a new Azure VM to host a web server for a critical application. The VM must be provisioned with Ubuntu, bootstrap Nginx automatically at first launch, and allow HTTP access from the internet.

---

## 2. Requirements
- VM name: `xfusion-vm`
- Region: `eastus`
- Image: Ubuntu (`Ubuntu2204` used)
- User data / custom data: run script at first boot to:
  - install Nginx
  - start Nginx service
- NSG: allow inbound HTTP traffic on port `80`
- Deployment method: Azure CLI

---

## 3. Core Theory (Must Know Before Execution)

### Azure Resource Group
A Resource Group is a logical container for related Azure resources. In this task, all resources are created in:
- `kml_rg_main-30c3042d41454091`

### Virtual Machine Provisioning with Azure CLI
`az vm create` can create VM, NIC, NSG, public IP, and VNet defaults in one workflow. This is fast and reproducible for lab and operational tasks.

### cloud-init (`--custom-data`)
For Linux VMs, cloud-init runs on first boot and applies bootstrap instructions. This is a clean way to automate package installation and service startup without manual SSH setup.

### Network Security Group (NSG)
NSG controls inbound/outbound traffic. By default, HTTP may be blocked; `az vm open-port --port 80` creates an inbound allow rule so the web server is reachable from internet.

### Web Server Validation Logic
If Nginx installed correctly and NSG allows port 80:
1. VM must be in `VM running` state
2. Public IP must exist
3. Accessing the IP via browser/curl should return Nginx default page

---

## 4. Cloud-init File Used
Create `cloud-init.txt` locally:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
runcmd:
  - systemctl start nginx
  - systemctl enable nginx
```

---

## 5. Step-by-Step Execution (Professional Azure CLI Flow)

### Step 1: Confirm Resource Group and Region
```bash
az group list
```
Observed output includes:
- `name`: `kml_rg_main-30c3042d41454091`
- `location`: `eastus`

### Step 2: Create/Edit cloud-init file
```bash
vi cloud-init.txt
```
Add the cloud-init content shown above.

### Step 3: Create VM with explicit sizing and storage
```bash
az vm create \
--resource-group kml_rg_main-30c3042d41454091 \
--name xfusion-vm \
--image Ubuntu2204 \
--admin-username azureuser \
--generate-ssh-keys \
--custom-data cloud-init.txt \
--public-ip-sku Standard \
--size Standard_B1s \
--storage-sku Standard_LRS
```
Key output (successful):
- `powerState`: `VM running`
- `privateIpAddress`: `10.0.0.4`
- `publicIpAddress`: `20.127.104.219`
- `resourceGroup`: `kml_rg_main-30c3042d41454091`

### Step 4: Open HTTP port 80 in NSG
```bash
az vm open-port --port 80 --resource-group kml_rg_main-30c3042d41454091 --name xfusion-vm
```
Result confirms NSG update with custom security rule:
- `name`: `open-port-80`
- `direction`: `Inbound`
- `destinationPortRange`: `80`
- `access`: `Allow`

### Step 5: Verify VM IP addresses
```bash
az vm list-ip-addresses --resource-group kml_rg_main-30c3042d41454091 --name xfusion-vm --output table
```
Output:

```text
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
xfusion-vm        20.127.104.219       10.0.0.4
```

### Step 6: Functional check
Open in browser:
- `http://20.127.104.219`

Expected result:
- Nginx default page (`Welcome to nginx!`)

---

## 6. Validation Checklist
- [x] Resource Group exists in `eastus`
- [x] `cloud-init.txt` prepared with Nginx install/start commands
- [x] VM `xfusion-vm` created successfully
- [x] VM state is `VM running`
- [x] NSG allows inbound HTTP on port `80`
- [x] Public IP assigned: `20.127.104.219`
- [x] VM is internet-accessible on HTTP

---

## 7. Professional DevOps/Cloud Engineer Notes
- Prefer explicit flags (`--size`, `--storage-sku`, `--public-ip-sku`) for predictable builds.
- Keep bootstrap logic in version-controlled cloud-init files for repeatable provisioning.
- Validate both infrastructure state (CLI output) and application state (Nginx page response).
- For production, replace passwordless defaults with hardened NSG rules, SSH restrictions, monitoring, and IaC (Bicep/Terraform).
