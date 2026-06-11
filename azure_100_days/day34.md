# Day 34: Fixing Azure VM Package Installation Connectivity

## Task Overview

The goal of this task was to investigate why the Azure VM `devops-vm` could not install packages, identify the real connectivity block, and restore normal package installation behavior.

The lab initially looked like a general internet problem, but the actual root cause was an outbound NSG deny rule blocking HTTP and HTTPS traffic from the VM.

### Requirements
1. Identify the resource group and the VM network details.
2. Find the NIC attached to `devops-vm`.
3. Discover which NSG is attached to that NIC.
4. Inspect outbound NSG rules.
5. Add an outbound allow rule for internet access on ports 80 and 443.
6. Verify package retrieval works from inside the VM.
7. Record the mistakes made during troubleshooting and explain why they happened.
8. Use the East US region only.

### Portal and Lab Credentials

These were the lab access details available during the task.

| Item | Value |
| :--- | :--- |
| Portal URL | https://portal.azure.com |
| Username | kk_lab_user_main-e39e48d44ad64ce8@azurefreekmlprod.onmicrosoft.com |
| Password | gEa6zxUc |
| Start Time | Wed May 06 06:56:51 UTC 2026 |
| End Time | Wed May 06 07:56:51 UTC 2026 |
| Resource Group | kml_rg_main-e39e48d44ad64ce8 |
| VM Name | devops-vm |
| VM Public IP | 20.127.43.227 |
| VM Admin Username | azureuser |
| SSH Key Path | /root/.ssh/id_rsa |

---

## Core Theories

1. **NSG is a virtual firewall:** Azure Network Security Groups control inbound and outbound traffic at the subnet or NIC level.
2. **Outbound rules matter for package installation:** `apt update` and package installs need outbound access to HTTP/HTTPS endpoints.
3. **NIC and NSG are different resources:** A NIC name is not automatically the NSG name. The NSG must be discovered from the NIC or subnet association.
4. **Rule priority decides which rule wins:** A lower priority number takes precedence over a higher number. An allow rule must be placed above the blocking rule.
5. **Stateful firewall behavior:** Once outbound traffic is allowed, the return traffic is normally allowed automatically by Azure NSG state tracking.
6. **Ping is not the same as package download traffic:** `ping` uses ICMP, while package repositories use TCP. A blocked ping does not automatically mean package installation is broken.
7. **DNS was not the root cause here:** The VM successfully reached Ubuntu package repositories after the outbound HTTP/HTTPS rule was added, so DNS was not the blocker.
8. **Resource lookup must be exact:** Azure CLI commands often fail when the resource name, resource ID, or target type is confused.

---

## Step-by-Step Azure CLI Procedure

### 1. List the resource groups
```bash
az group list
```

Observed output:
```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-e39e48d44ad64ce8",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-e39e48d44ad64ce8",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

### 2. Inspect the VM NIC
```bash
az vm show -d -g kml_rg_main-e39e48d44ad64ce8 -n devops-vm --query "networkProfile.networkInterfaces"
```

Observed output:
```json
[
  {
    "deleteOption": null,
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-e39e48d44ad64ce8/providers/Microsoft.Network/networkInterfaces/devops-vmVMNic",
    "primary": null,
    "resourceGroup": "kml_rg_main-e39e48d44ad64ce8"
  }
]
```

### 3. First mistake: using the NIC name as if it were the NSG name
```bash
az network nsg rule list --resource-group kml_rg_main-e39e48d44ad64ce8 --nsg-name devops-vmVMNic --query "[?direction=='Outbound']"
```

Observed error:
```text
(ResourceNotFound) The Resource 'Microsoft.Network/networkSecurityGroups/devops-vmVMNic' under resource group 'kml_rg_main-e39e48d44ad64ce8' was not found.
```

### 4. Find the real NSG attached to the NIC
```bash
az network nic show \
  --resource-group kml_rg_main-e39e48d44ad64ce8 \
  --name devops-vmVMNic \
  --query "networkSecurityGroup.id" \
  --output tsv
```

Observed output:
```text
/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-e39e48d44ad64ce8/providers/Microsoft.Network/networkSecurityGroups/devops-nsg
```

### 5. Inspect outbound rules on the correct NSG
```bash
az network nsg rule list --resource-group kml_rg_main-e39e48d44ad64ce8 --nsg-name devops-nsg --query "[?direction=='Outbound']"
```

Observed output:
```json
[
  {
    "access": "Deny",
    "destinationAddressPrefix": "*",
    "destinationAddressPrefixes": [],
    "destinationPortRange": "*",
    "destinationPortRanges": [],
    "direction": "Outbound",
    "etag": "W/\"51b03fc5-93b4-423a-bfc5-98e67629df36\"",
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-e39e48d44ad64ce8/providers/Microsoft.Network/networkSecurityGroups/devops-nsg/securityRules/Block-All-Outbound",
    "name": "Block-All-Outbound",
    "priority": 200,
    "protocol": "*",
    "provisioningState": "Succeeded",
    "resourceGroup": "kml_rg_main-e39e48d44ad64ce8",
    "sourceAddressPrefix": "*",
    "sourceAddressPrefixes": [],
    "sourcePortRange": "*",
    "sourcePortRanges": [],
    "type": "Microsoft.Network/networkSecurityGroups/securityRules"
  }
]
```

### 6. Add the outbound allow rule for package downloads
```bash
az network nsg rule create \
  --resource-group kml_rg_main-e39e48d44ad64ce8 \
  --nsg-name devops-nsg \
  --name AllowInternetOutbound \
  --priority 101 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes Internet \
  --destination-port-ranges 80 443
```

Observed output:
```json
{
  "access": "Allow",
  "destinationAddressPrefix": "Internet",
  "destinationAddressPrefixes": [],
  "destinationPortRanges": [
    "80",
    "443"
  ],
  "direction": "Outbound",
  "etag": "W/\"43c42003-ac49-4eca-9b32-98295eb6049e\"",
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-e39e48d44ad64ce8/providers/Microsoft.Network/networkSecurityGroups/devops-nsg/securityRules/AllowInternetOutbound",
  "name": "AllowInternetOutbound",
  "priority": 101,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "kml_rg_main-e39e48d44ad64ce8",
  "sourceAddressPrefix": "*",
  "sourceAddressPrefixes": [],
  "sourcePortRange": "*",
  "sourcePortRanges": [],
  "type": "Microsoft.Network/networkSecurityGroups/securityRules"
}
```

### 7. Get the VM public IP
```bash
az vm show -d -g kml_rg_main-e39e48d44ad64ce8 -n devops-vm --query publicIps -o tsv
```

Observed output:
```text
20.127.43.227
```

### 8. Get the VM admin username
```bash
az vm show \
  --resource-group kml_rg_main-e39e48d44ad64ce8 \
  --name devops-vm \
  --query "osProfile.adminUsername" \
  --output tsv
```

Observed output:
```text
azureuser
```

### 9. SSH into the VM
```bash
ssh -i /root/.ssh/id_rsa azureuser@20.127.43.227
```

### 10. Verify package installation works
```bash
sudo apt update
```

Observed result after the fix:
```text
Hit:1 http://azure.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://azure.archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Get:3 http://azure.archive.ubuntu.com/ubuntu jammy-backports InRelease [127 kB]
Get:4 http://azure.archive.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
5 packages can be upgraded.
```

### 11. Optional check that still failed
```bash
ping -c 3 google.com
```

Observed result:
```text
3 packets transmitted, 0 received, 100% packet loss
```

This was expected because the NSG rule allowed TCP traffic for package downloads, not ICMP traffic for ping.

---

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. The NIC name was confused with the NSG name
* **Mistake:** `devops-vmVMNic` was used as the NSG name.
* **Why it happened:** The VM output showed the NIC name, and it was easy to assume that the NSG used the same name.
* **Fix:** Query the NIC directly with `az network nic show` and read `networkSecurityGroup.id`, which revealed the real NSG name: `devops-nsg`.

### 2. A placeholder was used literally in the shell command
* **Mistake:** `--nsg-name <nsg-name>` was entered exactly as written.
* **Why it happened:** The placeholder was not replaced with a real resource name before execution.
* **Fix:** Replace placeholders with actual values before running Azure CLI commands.

### 3. Ping failure was initially treated like a package-install failure
* **Mistake:** The failed `ping google.com` result could have been misread as proof that internet access still was broken.
* **Why it happened:** Ping and package download both look like connectivity tests, but they use different protocols.
* **Fix:** Recognize that package installation uses HTTP/HTTPS over TCP, while ping uses ICMP. `apt update` succeeding proved the relevant traffic was now allowed.

### 4. DNS was suspected before checking the actual package path
* **Mistake:** The issue was briefly interpreted as a DNS problem.
* **Why it happened:** `apt update` paused before the allow rule was added, which can resemble DNS or general internet failure.
* **Fix:** After the TCP 80/443 outbound rule was created, `apt update` completed successfully. That confirmed the root cause was the outbound NSG block, not DNS.

---

## Final Result

The VM could reach Ubuntu package repositories after the outbound NSG allow rule was added. Package installation capability was restored.

| Check | Result |
| :--- | :--- |
| Resource Group | `kml_rg_main-e39e48d44ad64ce8` |
| VM Name | `devops-vm` |
| NSG Name | `devops-nsg` |
| Fixed Rule | `AllowInternetOutbound` |
| Allowed Ports | `80`, `443` |
| Package Update | Successful |
| Ping to Google | Still blocked, expected |
