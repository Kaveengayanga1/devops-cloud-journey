# Day 36: Azure Blob Lifecycle Management for Cost Optimization

**Date:** May 29, 2026
**Topic:** Azure Storage - Blob Lifecycle Management
**Task:** Create an Azure Storage account, create a blob container, upload a file, and configure a lifecycle policy to delete blobs after 7 days of last modification.

---

## 1. Task Overview

The objective of this exercise was to implement Azure Blob Lifecycle Management for a specific container so that old blobs are deleted automatically after 7 days. This helps reduce storage cost and keeps the storage account aligned with retention requirements.

### Resource Details

| Item            | Value                                  |
| :-------------- | :------------------------------------- |
| Subscription ID | `f0c3bcdd-5ce2-4fa0-8cf3-41559747512b` |
| Resource Group  | `kml_rg_main-cf29307c3126407a`         |
| Region          | `eastus`                               |
| Storage Account | `datacenterstor20756`                  |
| Container       | `datacenter-container20756`            |
| Lifecycle Rule  | `datacenter-del-rule`                  |
| Uploaded File   | `/root/tempfile.txt`                   |

---

## 2. Credentials and Access Context

_This section records the operational context that was available during the task. No manual storage account key, SAS token, or password was exposed in the session output._

| Item                  | Value                                                                                                                    |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------- |
| Azure Portal          | https://portal.azure.com                                                                                                 |
| Authentication Method | Azure CLI login session (`az login`)                                                                                     |
| Storage Access        | Azure CLI warned that no explicit credentials were provided and then resolved access through the signed-in Azure context |

### Important Note on Credentials

The CLI output showed the following warning during container creation and blob upload:

> There are no credentials provided in your command and environment, we will query for account key for your storage account.

This means no account key or SAS token was manually supplied in the command. The task still succeeded because the Azure CLI could retrieve the necessary access information using the authenticated Azure session.

---

## 3. Core Theory

### Azure Storage Account

An Azure Storage Account is the top-level namespace for blob, file, queue, and table storage resources. It defines the storage location, redundancy model, performance tier, and security posture for the data that will be stored.

### Blob Container

A blob container is a logical grouping of blobs inside a storage account. It acts as the access boundary for unstructured data such as text files, logs, images, and backups.

### Locally Redundant Storage (LRS)

LRS replicates data three times within a single datacenter in the selected Azure region. It is the lowest-cost redundancy option and protects against local hardware failure, but not regional failure.

### Blob Lifecycle Management

Blob Lifecycle Management uses policy rules to automatically transition or delete blobs based on age, last access, or last modification. It is commonly used for cost control, compliance, and automated data retention.

### Prefix Matching in Lifecycle Rules

The lifecycle policy in this task uses a `prefixMatch` filter. This limits the rule to blobs under the specified container path, ensuring the policy is applied only to the intended scope.

---

## 4. Step-by-Step Implementation

### Step 1: Inspect the Resource Group

```bash
az group list
```

Observed output:

```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-cf29307c3126407a",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-cf29307c3126407a",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

### Step 2: Set the Working Variables

```bash
RESOURCE_GROUP="kml_rg_main-cf29307c3126407a"
LOCATION="eastus"
STORAGE_ACCOUNT="datacenterstor20756"
CONTAINER_NAME="datacenter-container20756"
RULE_NAME="datacenter-del-rule"
```

```bash
echo $LOCATION
```

Observed output:

```text
eastus
```

### Step 3: Create the Storage Account

```bash
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
```

Observed output:

```json
{
  "accessTier": "Hot",
  "creationTime": "2026-05-29T03:50:27.143870+00:00",
  "enableHttpsTrafficOnly": true,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-cf29307c3126407a/providers/Microsoft.Storage/storageAccounts/datacenterstor20756",
  "kind": "StorageV2",
  "location": "eastus",
  "name": "datacenterstor20756",
  "primaryEndpoints": {
    "blob": "https://datacenterstor20756.blob.core.windows.net/",
    "dfs": "https://datacenterstor20756.dfs.core.windows.net/",
    "file": "https://datacenterstor20756.file.core.windows.net/",
    "queue": "https://datacenterstor20756.queue.core.windows.net/",
    "table": "https://datacenterstor20756.table.core.windows.net/",
    "web": "https://datacenterstor20756.z13.web.core.windows.net/"
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "kml_rg_main-cf29307c3126407a",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "type": "Microsoft.Storage/storageAccounts"
}
```

### Step 4: Create the Blob Container

```bash
az storage container create \
  --name $CONTAINER_NAME \
  --account-name $STORAGE_ACCOUNT
```

Observed output:

```text
There are no credentials provided in your command and environment, we will query for account key for your storage account.
It is recommended to provide --connection-string, --account-key or --sas-token in your command as credentials.
You also can add --auth-mode login in your command to use Azure Active Directory (Azure AD) for authorization if your login account is assigned required RBAC roles.
{
  "created": true
}
```

### Step 5: Upload the File

```bash
az storage blob upload \
  --account-name $STORAGE_ACCOUNT \
  --container-name $CONTAINER_NAME \
  --name tempfile.txt \
  --file /root/tempfile.txt
```

Observed output:

```text
There are no credentials provided in your command and environment, we will query for account key for your storage account.
It is recommended to provide --connection-string, --account-key or --sas-token in your command as credentials.
You also can add --auth-mode login in your command to use Azure Active Directory (Azure AD) for authorization if your login account is assigned required RBAC roles.
Finished[#############################################################]  100.0000%
```

```json
{
  "client_request_id": "b8c5f676-5b11-11f1-ac46-6a2724fb1c87",
  "content_md5": "KK8IdCQ+hAnGosayoGGjvA==",
  "date": "2026-05-29T03:51:49+00:00",
  "etag": "\"0x8DEBD359D4A9E29\"",
  "lastModified": "2026-05-29T03:51:50+00:00",
  "request_id": "4080ecde-e01e-003b-271e-efec1d000000",
  "request_server_encrypted": true,
  "version": "2022-11-02"
}
```

### Step 6: Create the Lifecycle Policy File

```bash
cat << EOF > /tmp/lifecycle-policy.json
{
  "rules": [
    {
      "enabled": true,
      "name": "datacenter-del-rule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "delete": {
              "daysAfterModificationGreaterThan": 7
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "datacenter-container20756/"
          ]
        }
      }
    }
  ]
}
EOF
```

### Step 7: Apply the Lifecycle Management Policy

```bash
az storage account management-policy create \
  --account-name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --policy @/tmp/lifecycle-policy.json
```

Observed output:

```json
{
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-cf29307c3126407a/providers/Microsoft.Storage/storageAccounts/datacenterstor20756/managementPolicies/default",
  "lastModifiedTime": "2026-05-29T03:52:13.705875+00:00",
  "name": "DefaultManagementPolicy",
  "policy": {
    "rules": [
      {
        "definition": {
          "actions": {
            "baseBlob": {
              "delete": {
                "daysAfterModificationGreaterThan": 7.0
              }
            }
          },
          "filters": {
            "blobTypes": ["blockBlob"],
            "prefixMatch": ["datacenter-container20756/"]
          }
        },
        "enabled": true,
        "name": "datacenter-del-rule",
        "type": "Lifecycle"
      }
    ]
  },
  "resourceGroup": "kml_rg_main-cf29307c3126407a",
  "type": "Microsoft.Storage/storageAccounts/managementPolicies"
}
```

### Step 8: Verify the Policy

```bash
az storage account management-policy show \
  --account-name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP
```

Observed output:

```json
{
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-cf29307c3126407a/providers/Microsoft.Storage/storageAccounts/datacenterstor20756/managementPolicies/default",
  "lastModifiedTime": "2026-05-29T03:52:13.705875+00:00",
  "name": "DefaultManagementPolicy",
  "policy": {
    "rules": [
      {
        "definition": {
          "actions": {
            "baseBlob": {
              "delete": {
                "daysAfterModificationGreaterThan": 7.0
              }
            }
          },
          "filters": {
            "blobTypes": ["blockBlob"],
            "prefixMatch": ["datacenter-container20756/"]
          }
        },
        "enabled": true,
        "name": "datacenter-del-rule",
        "type": "Lifecycle"
      }
    ]
  },
  "resourceGroup": "kml_rg_main-cf29307c3126407a",
  "type": "Microsoft.Storage/storageAccounts/managementPolicies"
}
```

---

## 5. Mistakes Made, Why They Happened, and How They Were Fixed

### Mistake 1: Assuming the CLI would receive explicit credentials

- **What happened:** The container and blob commands were executed without `--account-key`, `--connection-string`, or `--sas-token`.
- **Why it happened:** The lab workflow used the signed-in Azure session and did not require manually typed storage secrets.
- **How it was fixed:** The task was completed using the authenticated Azure CLI context, and the policy was applied successfully after the CLI resolved access through the session.

### Mistake 2: Treating the warning as a failure

- **What happened:** The CLI warned that no credentials were provided, which can look like an error at first glance.
- **Why it happened:** Azure Storage commands often display this warning when they need to resolve access automatically.
- **How it was fixed:** The warning was interpreted correctly as informational because the commands completed successfully and returned valid JSON outputs.

### Mistake 3: Using the container path in the lifecycle scope without validating the filter

- **What happened:** The lifecycle rule was scoped with `prefixMatch` set to `datacenter-container20756/`.
- **Why it happened:** The policy needed to target blobs inside one container only, and prefix matching is the correct method for that scope.
- **How it was fixed:** The policy was validated with `az storage account management-policy show`, which confirmed the rule name and the 7-day delete condition.

---

## 6. Final Validation Summary

The lifecycle management rule was successfully created and confirmed with the expected values:

- Rule name: `datacenter-del-rule`
- Delete condition: `daysAfterModificationGreaterThan = 7`
- Blob type: `blockBlob`
- Container prefix: `datacenter-container20756/`

This satisfies the requirement to automatically delete blobs after 7 days of last modification.
