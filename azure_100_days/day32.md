# Day 32: Azure Blob Container Migration and Verification

## Task Overview

The goal of this task was to create a new private Azure Blob container and migrate a blob from an existing source container into it, then verify that the copied data matched the original.

### Requirements:
1. Create a new private container named `datacenter-dest-29567`.
2. Use the storage account `datacenterst10532`.
3. Copy the blob `datacenter.txt` from `datacenter-source-12318` to `datacenter-dest-29567`.
4. Confirm the copy completed successfully.
5. Confirm the MD5 hash of the blob in both containers is identical.

### Credential-Related Notes
The command output did not include any explicit storage account key, SAS token, or connection string. Azure CLI therefore attempted to query the account key automatically and printed authentication guidance. The only visible values in the session were the storage account, container names, and blob name.

---

## Core Theories

1. **Azure Blob Storage Hierarchy:** Azure Blob Storage is organized as Storage Account -> Container -> Blob. The storage account is the top-level namespace, containers group blobs, and blobs hold the actual content.
2. **Private Container Access:** A private container with `--public-access off` blocks anonymous access and is the correct default for production data.
3. **Server-Side Blob Copy:** `az storage blob copy start` triggers an asynchronous server-side copy within Azure. The transfer is handled by Azure rather than by downloading and re-uploading the file locally.
4. **Asynchronous Operations:** Copy operations may complete after the command returns. That is why the copy status must be checked separately.
5. **Data Integrity Verification:** MD5 hashes are a reliable way to verify that source and destination contents are identical after migration.
6. **Azure CLI Authentication:** Azure CLI storage commands can use account keys, SAS tokens, connection strings, or Azure AD login with RBAC permissions. If none are provided, CLI tries to discover credentials automatically and warns the user.
7. **Metadata Validation:** Checking fields such as copy status, content MD5, and last modified time helps confirm the transfer completed correctly.

---

## Step-by-Step Implementation

### 1. Define reusable environment variables
```bash
export STORAGE_ACCOUNT="datacenterst10532"
export SOURCE_CONTAINER="datacenter-source-12318"
export DEST_CONTAINER="datacenter-dest-29567"
export BLOB_NAME="datacenter.txt"
```

### 2. Confirm the storage account value
```bash
echo $STORAGE_ACCOUNT
```

### 3. Create the new private destination container
```bash
az storage container create \
    --name $DEST_CONTAINER \
    --account-name $STORAGE_ACCOUNT \
    --public-access off
```

### 4. Copy the blob from the source container to the destination container
```bash
az storage blob copy start \
    --account-name $STORAGE_ACCOUNT \
    --destination-blob $BLOB_NAME \
    --destination-container $DEST_CONTAINER \
    --source-blob $BLOB_NAME \
    --source-container $SOURCE_CONTAINER
```

### 5. Verify the copy status
```bash
az storage blob show \
    --account-name $STORAGE_ACCOUNT \
    --container-name $DEST_CONTAINER \
    --name $BLOB_NAME \
    --query "properties.copy.status"
```

### 6. Verify the source blob MD5
```bash
az storage blob show \
    --account-name $STORAGE_ACCOUNT \
    --container-name $SOURCE_CONTAINER \
    --name $BLOB_NAME \
    --query "properties.contentSettings.contentMd5"
```

### 7. Verify the destination blob MD5
```bash
az storage blob show \
    --account-name $STORAGE_ACCOUNT \
    --container-name $DEST_CONTAINER \
    --name $BLOB_NAME \
    --query "properties.contentSettings.contentMd5"
```

---

## Command Output and Verification

### Environment Variable Check
```text
datacenterst10532
```

### Container Creation Output
```json
{
  "created": true
}
```

### Copy Operation Output
```json
{
  "client_request_id": "d0aecbfe-41db-11f1-9eb0-eeee18f55ddd",
  "copy_id": "6c3fd19d-c952-40a9-82db-167f1ce7da38",
  "copy_status": "success",
  "date": "2026-04-27T01:52:57+00:00",
  "etag": "\"0x8DEA3FFB545A55B\"",
  "last_modified": "2026-04-27T01:52:58+00:00",
  "request_id": "96f767af-701e-0076-67e8-d5e163000000",
  "version": "2022-11-02",
  "version_id": null
}
```

### Copy Status
```text
"success"
```

### Source Blob MD5
```text
"Lu7zilatbGguzSz2Ecn5IQ=="
```

### Destination Blob MD5
```text
"Lu7zilatbGguzSz2Ecn5IQ=="
```

---

## Mistakes Made, Reasons, and Fixes

### 1. No explicit storage credentials were provided
* **Mistake:** The commands were executed without `--connection-string`, `--account-key`, or `--sas-token`.
* **Reason:** Azure CLI storage commands need authorization details. Without explicit credentials, the CLI tries to discover them and prints warnings.
* **Fix:** The copy still succeeded because Azure CLI was able to proceed in the lab context, but the proper professional approach is to supply credentials explicitly or use `--auth-mode login` with the correct RBAC role.

### 2. Authentication guidance appeared during both create and copy operations
* **Mistake:** The commands relied on implicit authentication behavior instead of a clearly defined auth method.
* **Reason:** Azure Storage CLI prefers account key, SAS token, connection string, or Azure AD login. When none are passed, it reports the missing credentials message.
* **Fix:** For production or repeatable automation, choose one auth mode and standardize it in the script.

### 3. Asynchronous copy required follow-up verification
* **Mistake:** Assuming the copy was complete immediately after `az storage blob copy start`.
* **Reason:** Blob copy is asynchronous, so the initial command only starts the operation.
* **Fix:** Checked `properties.copy.status` until it reported `success`, then compared MD5 hashes.

---

## Final Verification Result
| Requirement | Value | Status |
| :--- | :--- | :--- |
| **Storage Account** | `datacenterst10532` | ✅ Success |
| **Source Container** | `datacenter-source-12318` | ✅ Success |
| **Destination Container** | `datacenter-dest-29567` | ✅ Success |
| **Blob Name** | `datacenter.txt` | ✅ Success |
| **Copy Status** | `success` | ✅ Success |
| **Source MD5** | `Lu7zilatbGguzSz2Ecn5IQ==` | ✅ Success |
| **Destination MD5** | `Lu7zilatbGguzSz2Ecn5IQ==` | ✅ Success |
