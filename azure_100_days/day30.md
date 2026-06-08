# Day 30: Azure SQL Database Provisioning and Troubleshooting

## Task Overview

The goal of this task was to create and configure a publicly accessible Azure SQL Database instance with specific requirements.

### Requirements:
1.  **Database Name:** `datacenter-sqldb`
2.  **Server Name:** `datacenter-server-19484`
3.  **Location:** `centralus`
4.  **Compute + Storage:** `Basic`
5.  **Backup Storage Redundancy:** `Locally-redundant backup storage` (LRS)
6.  **Administrator Credentials:**
    *   Username: `datacenter-admin`
    *   Password: `P@ssw0rd19484`
7.  **Database Size:** `2 GiB`
8.  **State:** Must be in `Ready` (Online) state.

### Azure Lab Credentials (Expired):
*   **Portal URL:** https://portal.azure.com
*   **Username:** `kk_lab_user_main-81237fdd031f4ab1@azurefreekmlprod.onmicrosoft.com`
*   **Password:** `9bDf2MaY`

---

## Core Theories

1.  **Azure SQL Database (PaaS):** A managed relational database service provided by Microsoft Azure. It handles infrastructure management, allowing users to focus on data.
2.  **Logical Server:** A central administrative point for multiple databases. You cannot create a database without a logical server.
3.  **Service Tiers (Basic):** Designed for low-demand workloads. It is cost-effective but has limited performance.
4.  **Backup Storage Redundancy (LRS):** Maintains three copies of your data within a single data center.
5.  **Firewall Rules:** By default, Azure SQL Servers are secure. Firewall rules must be configured to allow public access.

---

## Step-by-Step Implementation

### 1. Identify Resource Group
First, we list the available resource groups to find the correct one assigned to the lab.
```bash
az group list
```

### 2. Create Azure SQL Logical Server
```bash
az sql server create \
    --name datacenter-server-19484 \
    --resource-group kml_rg_main-81237fdd031f4ab1 \
    --location centralus \
    --admin-user datacenter-admin \
    --admin-password P@ssw0rd19484
```

### 3. Configure Firewall Rules for Public Access
To make the database publicly accessible (allowing all IPs):
```bash
az sql server firewall-rule create \
    --resource-group kml_rg_main-81237fdd031f4ab1 \
    --server datacenter-server-19484 \
    --name AllowAll \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 255.255.255.255
```

### 4. Create Azure SQL Database
Initial attempt with hardware family specification:
```bash
az sql db create \
    --resource-group kml_rg_main-81237fdd031f4ab1 \
    --server datacenter-server-19484 \
    --name datacenter-sqldb \
    --edition Basic \
    --family Gen5 \
    --capacity 5 \
    --max-size 2GB \
    --backup-storage-redundancy Local
```

### 5. Verify Database Status
```bash
az sql db show \
    --resource-group kml_rg_main-81237fdd031f4ab1 \
    --server datacenter-server-19484 \
    --name datacenter-sqldb \
    --query "status"
```

---

## Mistakes Made, Reasons, and Fixes

### 1. Incompatible SKU and Hardware Family
*   **Mistake:** Attempting to use `--family Gen5` with the `Basic` edition.
*   **Reason:** The `Basic`, `Standard`, and `Premium` tiers are DTU-based (Database Transaction Units). Hardware generations like `Gen5` are specific to the vCore-based purchasing model. In the DTU model, the hardware family is managed by Azure and shouldn't be specified.
*   **Fix:** Removed the `--family Gen5` parameter from the command.

### 2. Authorization Failure during Verification
*   **Mistake:** Receiving `(AuthorizationFailed)` when running the `show` command.
*   **Reason:** Used the wrong Resource Group name (`datacenter-rg`) in the command. The identity only had permissions for the specific lab resource group `kml_rg_main-81237fdd031f4ab1`.
*   **Fix:** Updated the command with the correct resource group name.

---

## Final Verification Result
| Requirement | Value | Status |
| :--- | :--- | :--- |
| **Database Name** | `datacenter-sqldb` | ✅ Success |
| **Server Name** | `datacenter-server-19484` | ✅ Success |
| **Location** | `centralus` | ✅ Success |
| **Edition** | `Basic` | ✅ Success |
| **Max Size** | `2 GiB` | ✅ Success |
| **Status** | `"Online"` | ✅ Success |
