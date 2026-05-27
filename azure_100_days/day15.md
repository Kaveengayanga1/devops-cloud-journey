# Day 15: Azure Network Security Groups (NSG)

## Theory

### 1. Core Theory (මූලික සිද්ධාන්ත)
An **Azure Network Security Group (NSG)** is essentially a Virtual Firewall for your Azure Virtual Network (VNet). It is used to control **Inbound** (incoming) and **Outbound** (outgoing) Network Traffic to your Azure resources.

### Key Concepts:
*   **NSG Rules:** Every rule has a Name, Priority, Source, Destination, Protocol, and Port.
*   **Inbound Rules:** Controls traffic coming from the outside into your server (e.g., opening Port 80 for a web server).
*   **Outbound Rules:** Controls traffic going from your server to the outside.
*   **Priority:** A number between 100 and 4096. Rules with lower numbers (e.g., 100) are processed before higher numbers.
*   **CIDR (0.0.0.0/0):** This notation represents "Any IP address" (Access from anywhere in the world).

### 2. Steps to Create an NSG via Azure CLI
Before executing these steps, ensure you have your Resource Group name. In this guide, we use `kml_rg_main-a3bedc06131f4194`.

#### Step 01: Create the Network Security Group
We create an NSG named `xfusion-nsg`.

#### Step 02: Add HTTP (Port 80) Inbound Rule
We create a rule named `Allow-HTTP` to allow traffic from `0.0.0.0/0` (anywhere) to Port 80.

#### Step 03: Add SSH (Port 22) Inbound Rule
We create a rule named `Allow-SSH` to allow remote connection to the server on Port 22.

### 3. Verification
Use the list command to verify the rules were created correctly.

> **Important:** For this NSG to be effective, it must be **Associated** with a Virtual Machine's network interface or a Subnet.

---

## Practical Execution & Output

### 1. Check Resource Group
```bash
az group list
```
**Output:**
```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-a3bedc06131f4194",
    "location": "eastus",
    "name": "kml_rg_main-a3bedc06131f4194",
    "properties": {
      "provisioningState": "Succeeded"
    }
  }
]
```

### 2. Create the NSG
```bash
az network nsg create \
  --resource-group kml_rg_main-a3bedc06131f4194 \
  --name xfusion-nsg
```
**Output:**
```json
{
  "NewNSG": {
    "defaultSecurityRules": [
      {
        "access": "Allow",
        "description": "Allow inbound traffic from all VMs in VNET",
        "direction": "Inbound",
        "name": "AllowVnetInBound",
        "priority": 65000,
        "protocol": "*"
      },
      {
        "access": "Deny",
        "description": "Deny all inbound traffic",
        "direction": "Inbound",
        "name": "DenyAllInBound",
        "priority": 65500,
        "protocol": "*"
      }
    ],
    "name": "xfusion-nsg",
    "provisioningState": "Succeeded"
  }
}
```

### 3. Create HTTP Rule (Port 80)
```bash
az network nsg rule create \
  --resource-group kml_rg_main-a3bedc06131f4194 \
  --nsg-name xfusion-nsg \
  --name Allow-HTTP \
  --protocol tcp \
  --priority 100 \
  --destination-port-range 80 \
  --source-address-prefix '0.0.0.0/0' \
  --access allow
```
**Output:**
```json
{
  "access": "Allow",
  "name": "Allow-HTTP",
  "priority": 100,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "destinationPortRange": "80",
  "sourceAddressPrefix": "0.0.0.0/0"
}
```

### 4. Create SSH Rule (Port 22)
```bash
az network nsg rule create \
  --resource-group kml_rg_main-a3bedc06131f4194 \
  --nsg-name xfusion-nsg \
  --name Allow-SSH \
  --protocol tcp \
  --priority 110 \
  --destination-port-range 22 \
  --source-address-prefix '0.0.0.0/0' \
  --access allow
```
**Output:**
```json
{
  "access": "Allow",
  "name": "Allow-SSH",
  "priority": 110,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "destinationPortRange": "22",
  "sourceAddressPrefix": "0.0.0.0/0"
}
```

### 5. Verify Rules
```bash
az network nsg rule list --resource-group kml_rg_main-a3bedc06131f4194 --nsg-name xfusion-nsg --output table
```
**Output:**
```
Name        ResourceGroup                 Priority    SourcePortRanges    SourceAddressPrefixes    SourceASG    Access    Protocol    Direction    DestinationPortRanges    DestinationAddressPrefixes    DestinationASG
----------  ----------------------------  ----------  ------------------  -----------------------  -----------  --------  ----------  -----------  -----------------------  ----------------------------  ----------------
Allow-HTTP  kml_rg_main-a3bedc06131f4194  100         *                   0.0.0.0/0                None         Allow     Tcp         Inbound      80                       *                             None
Allow-SSH   kml_rg_main-a3bedc06131f4194  110         *                   0.0.0.0/0                None         Allow     Tcp         Inbound      22                       *                             None
```

---

## Professional DevOps Engineer Instructions

When performing this task in a real-world environment, a professional DevOps engineer would follow these practices:

1.  **Use Variables**: Avoid hardcoding Resource Group names. Use variables to reduce errors and improve reusability.
    ```bash
    RG_NAME="kml_rg_main-a3bedc06131f4194"
    NSG_NAME="xfusion-nsg"
    
    az network nsg create --resource-group $RG_NAME --name $NSG_NAME
    ```

2.  **Security Best Practice (SSH)**: In a production environment, **NEVER** expose Port 22 (SSH) to `0.0.0.0/0` (the whole internet).
    *   **Correct approach:** Limit the standard `--source-address-prefix` to your company's VPN IP or your specific public IP address.
    *   **Alternative:** Use **Azure Bastion** for secure connectivity without exposing public ports.

3.  **Tags**: Always apply tags (e.g., `Environment=Dev`, `Owner=TeamX`) when creating resources for cost tracking and governance.
    ```bash
    az network nsg create ... --tags Environment=Dev Project=XFusion
    ```

4.  **Infrastructure as Code (IaC)**: While CLI is great for learning and ad-hoc tasks, production resources are typically deployed using **Terraform**, **Bicep**, or **ARM Templates**. This ensures the infrastructure states are managed and version controlled.
