# Day 20: ARM Template VNet Deployment - CIDR Error Troubleshooting and Fix

## 1. Theory: What You Must Know Before Deploying ARM Templates

### Infrastructure as Code (IaC) with ARM
Azure Resource Manager (ARM) templates let you define infrastructure in JSON and deploy it repeatedly in a consistent way. This is core DevOps practice because it reduces manual mistakes and supports automation.

### ARM Template Core Sections
- **$schema / contentVersion**: Template contract and version metadata.
- **parameters**: Values passed at deployment time.
- **variables**: Reusable internal values.
- **resources**: Actual Azure resources to create (here: Virtual Network).
- **outputs**: Values returned after deployment.

### CIDR Basics (Very Important)
CIDR blocks must be aligned with prefix size:
- For `/16`, valid network base looks like `x.y.0.0/16`.
- `10.10.10.0/16` is invalid, because for `/16` the third and fourth octets must start at `0.0`.
- Correct examples: `10.10.0.0/16`, `192.168.0.0/16`.

### Why Tags Matter
Tags such as `displayName` and `Environment` are metadata used for:
- Resource organization
- Cost tracking
- Policy/compliance filtering
- Operational ownership

### Real-World Applications
- Standardized network provisioning across Dev/Test/Prod.
- Fast environment rebuild during incidents.
- Governance with naming/tagging policies.
- CI/CD-based infrastructure rollout.

---

## 2. Task Objective
Update `/root/arm-templates/vnet-deployment-template.json` with:
1. VNet name: `arm-vnet-devops`
2. Tag `displayName`: `arm-vnet-devops`
3. Address prefix: `192.168.0.0/16`
4. Tag `Environment`: `KKE-devops`

Then deploy using Azure CLI and verify the VNet exists.

---

## 3. Output Walkthrough (As Executed)

### Step 1: Find the correct resource group
```bash
az group list --query '[].name' --output table | grep 'kml'
```
Output:
```text
kml_rg_main-0425d24cd9804b51
```

### Step 2: Go to template location and review file
```bash
cd /root/arm-templates/
ls
cat vnet-deployment-template.json
```

### Step 3: First deployment attempt (failed)
```bash
az deployment group create \
  --resource-group kml_rg_main-0425d24cd9804b51 \
  --template-file /root/arm-templates/vnet-deployment-template.json
```
Error:
```text
InvalidTemplateDeployment
Inner Error: InvalidCIDRNotation
The address prefix 10.10.10.0/16 ... should be 10.10.0.0/16
```

### Step 4: Root cause and fix
- Root cause: Invalid CIDR network boundary.
- Also align with task requirement: use `192.168.0.0/16`.

Corrected resource section:
```json
{
  "name": "arm-vnet-devops",
  "type": "Microsoft.Network/virtualNetworks",
  "apiVersion": "2023-11-01",
  "location": "[resourceGroup().location]",
  "tags": {
    "displayName": "arm-vnet-devops",
    "Environment": "KKE-devops"
  },
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "192.168.0.0/16"
      ]
    }
  }
}
```

### Step 5: Re-deploy after correction (succeeded)
```bash
az deployment group create \
  --resource-group kml_rg_main-0425d24cd9804b51 \
  --template-file /root/arm-templates/vnet-deployment-template.json
```
Result:
- `provisioningState: Succeeded`
- Output resource includes `Microsoft.Network/virtualNetworks/arm-vnet-devops`

### Step 6: Verify VNet
Wrong placeholder usage:
```bash
az network vnet show --name arm-vnet-devops --resource-group <RG_NAME> --output table
```
Error:
```text
-bash: RG_NAME: No such file or directory
```

Correct command:
```bash
az network vnet show --name arm-vnet-devops --resource-group kml_rg_main-0425d24cd9804b51 --output table
```
Successful output shows:
- Name: `arm-vnet-devops`
- Location: `eastus`
- ProvisioningState: `Succeeded`

---

## 4. Cloud/DevOps Engineer Execution Flow (Recommended)

1. **Identify target RG** using filtered CLI query.
2. **Inspect template before deploy** (`cat` / editor review).
3. **Deploy and capture errors** (never ignore inner errors).
4. **Fix root cause in template** (not workaround in command).
5. **Re-deploy in incremental mode**.
6. **Verify created resource** using direct resource show command.
7. **Avoid placeholder mistakes** (`<RG_NAME>` must be replaced with actual value).

---

## 5. Best Practices

- Run validation before full deploy:
```bash
az deployment group validate \
  --resource-group kml_rg_main-0425d24cd9804b51 \
  --template-file /root/arm-templates/vnet-deployment-template.json
```
- Parameterize reusable values (name, address prefix, environment tag).
- Keep consistent naming and tagging for governance.
- Commit template changes to version control for auditability.
