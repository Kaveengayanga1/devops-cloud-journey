# Day 35: Implementing Azure VNet Peering

**Date:** May 16, 2026
**Topic:** Azure Networking - Virtual Network (VNet) Peering
**Task:** Configure VNet Peering between a public and a private VNet to allow communication between VMs using private IPs.

---

## 1. Task Overview

The goal is to establish a secure connection between two separate Virtual Networks (`datacenter-pub-vnet` and `datacenter-priv-vnet`) using Azure VNet Peering. This allows a VM in the public network to communicate with a VM in the private network via the Microsoft backbone network.

### Resource Details

| Resource Name          | Type       | IP Address                                   | VNet/Subnet                                   |
| :--------------------- | :--------- | :------------------------------------------- | :-------------------------------------------- |
| **datacenter-pub-vm**  | Public VM  | Public: `20.124.180.22`, Private: `10.2.1.4` | `datacenter-pub-vnet/datacenter-pub-subnet`   |
| **datacenter-priv-vm** | Private VM | Private: `10.1.1.4`                          | `datacenter-priv-vnet/datacenter-priv-subnet` |

---

## 2. Credentials (Expired)

_These credentials are recorded for documentation purposes only._

- **Portal URL:** [https://portal.azure.com](https://portal.azure.com)
- **Username:** `kk_lab_user_main-b64a0071b0ec4a1d@azurefreekmlprod.onmicrosoft.com`
- **Password:** `=ZD43#W=`
- **Azure Region:** East US

---

## 3. Implementation Steps

### Step 1: Identify Networking Context

Identify the VNets associated with both VMs. In this case, `datacenter-pub-vm` is in `datacenter-pub-vnet` and `datacenter-priv-vm` is in `datacenter-priv-vnet`.

### Step 2: Configure VNet Peering

1. Navigate to **Virtual networks** > **datacenter-pub-vnet**.
2. Select **Peerings** under Settings and click **+ Add**.
3. Configure the following:
   - **Local VNet Peering Link Name:** `datacenter-pub-to-priv-peering`
   - **Remote VNet Peering Link Name:** `datacenter-priv-to-pub-peering`
   - **Remote Virtual Network:** `datacenter-priv-vnet`
   - **Traffic to remote virtual network:** Allow
   - **Traffic forwarded from remote virtual network:** Allow

### Step 3: Verify Connection

1. SSH into the Public VM.
2. Ping the Private VM's private IP (`10.1.1.4`).

---

## 4. Mistakes Made & Troubleshooting

### Issue 1: Peering with "Self"

- **Mistake:** Attempting to select `datacenter-pub-vnet` as the remote network while currently inside the `datacenter-pub-vnet` settings.
- **Error:** _"Select a different virtual network. A virtual network cannot be peered with itself."_
- **Fix:** Properly selecting the intended destination network (`datacenter-priv-vnet`) from the dropdown.

### Issue 2: Peering Status "Initiated / Remote sync required"

- **Mistake:** The peering was initiated from one side but didn't automatically synchronize with the other side due to lab permission limitations or timing.
- **Reason:** For a peering to be "Connected," both sides must have a corresponding peering link.
- **Fix:** Navigated to the `datacenter-priv-vnet`, went to the Peerings section, and clicked **Sync** on the existing link or manually added the return link from Private to Public.

---

## 5. Verification Output

```bash
azureuser@datacenter-pub-vm:~$ ping -c 4 10.1.1.4
PING 10.1.1.4 (10.1.1.4) 56(84) bytes of data.
64 bytes from 10.1.1.4: icmp_seq=1 ttl=64 time=2.09 ms
64 bytes from 10.1.1.4: icmp_seq=2 ttl=64 time=0.649 ms
64 bytes from 10.1.1.4: icmp_seq=3 ttl=64 time=1.09 ms
64 bytes from 10.1.1.4: icmp_seq=4 ttl=64 time=0.845 ms

--- 10.1.1.4 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3045ms
rtt min/avg/max/mdev = 0.649/1.168/2.086/0.552 ms
```
