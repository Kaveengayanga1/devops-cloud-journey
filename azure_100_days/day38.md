Day 38: Azure VM → Azure Blob Storage
Date: 2026-06-05
Topic: Create private Azure Storage Account and upload a test file from an existing VM
Project: Nautilus DevOps — Storage integration (example)

## Overview

This document records the step-by-step actions taken to create a private Azure Storage account (`devopsstor10876`), a private blob container (`devops-container10876`), retrieve the account key, create a test file on the VM `devops-vm`, upload that file to the blob container, and verify the upload. All credentials and outputs below are historical/example values gathered during the exercise.

## Resources & Historical Credentials (Examples)

- Resource Group: `kml_rg_main-238961b4df974207`
- Storage account: `devopsstor10876` (East US, Standard_LRS)
- Container: `devops-container10876` (private)
- VM name: `devops-vm` (public IP: `20.169.211.65`)
- VM user: `azureuser`
- Example storage account key (historical/example, no longer active):

```
OSlzSBzQzmXqg5wecKOgyRenRebSqAq6dAK4dWMni2sQlBMRAuU8U2RBESf672DzL5BgMA0ePRDe+AStiOUhHQ==
```

> Note: Treat any keys shown here as historical/example values only. Do not publish active credentials in source control or public channels.

## Commands Executed (recorded)

1. Set default location

```
az configure --defaults location=eastus
```

2. Verify resource groups

```
az group list
```

Sample output (truncated):

```
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-238961b4df974207",
    "location": "eastus",
    "name": "kml_rg_main-238961b4df974207",
    "properties": {"provisioningState": "Succeeded"}
  }
]
```

3. Create storage account (private, LRS)

```
az storage account create \
  --name devopsstor10876 \
  --resource-group kml_rg_main-238961b4df974207 \
  --location eastus \
  --sku Standard_LRS \
  --allow-blob-public-access false
```

Sample creation result (truncated):

```
{
  "allowBlobPublicAccess": false,
  "creationTime": "2026-06-05T12:27:41.049118+00:00",
  "enableHttpsTrafficOnly": true,
  "encryption": {"services": {"blob": {"enabled": true}}},
  "id": "/subscriptions/.../storageAccounts/devopsstor10876",
  "location": "eastus",
  "name": "devopsstor10876",
  "provisioningState": "Succeeded",
  "sku": {"name": "Standard_LRS"}
}
```

4. Create private blob container

```
az storage container create \
    --name devops-container10876 \
    --account-name devopsstor10876 \
    --public-access off
```

CLI printed a message indicating it would query for account key (if not provided) and responded with `{ "created": true }`.

5. Retrieve storage account key

```
az storage account keys list \
    --account-name devopsstor10876 \
    --resource-group kml_rg_main-238961b4df974207 \
    --query "[0].value" \
    --output tsv
```

Returned (example/historical):

```
OSlzSBzQzmXqg5wecKOgyRenRebSqAq6dAK4dWMni2sQlBMRAuU8U2RBESf672DzL5BgMA0ePRDe+AStiOUhHQ==
```

6. Get VM public IP (example)

```
az vm list-ip-addresses --resource-group kml_rg_main-238961b4df974207 --name "devops-vm" --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" --output tsv
```

Returned:

```
20.169.211.65
```

7. SSH into VM and create the test file

```
ssh azureuser@20.169.211.65
cd /home/azureuser && echo "this is a test file." > testfile.txt
cat /home/azureuser/testfile.txt
```

Output from `cat`:

```
this is a test file.
```

8. Upload the test file to blob storage using account key

```
az storage blob upload \
    --account-name devopsstor10876 \
    --account-key "OSlzSBzQzmXqg5wecKOgyRenRebSqAq6dAK4dWMni2sQlBMRAuU8U2RBESf672DzL5BgMA0ePRDe+AStiOUhHQ==" \
    --container-name devops-container10876 \
    --name testfile.txt \
    --file /home/azureuser/testfile.txt
```

Sample upload output (successful):

```
Finished[#############################################################]  100.0000%
{
  "client_request_id": "fe43e091-60da-11f1-be43-a76f914d446a",
  "content_md5": "QrEn69XW1rKvT7d6PrY9dA==",
  "lastModified": "2026-06-05T12:35:12+00:00",
  "request_server_encrypted": true,
  "version": "2026-04-06"
}
```

9. Verify container contents

```
az storage blob list \
    --account-name devopsstor10876 \
    --account-key "OSlzSBzQzmXqg5wecKOgyRenRebSqAq6dAK4dWMni2sQlBMRAuU8U2RBESf672DzL5BgMA0ePRDe+AStiOUhHQ==" \
    --container-name devops-container10876 \
    --output table
```

Sample `table` output:

```
Name          Blob Type    Blob Tier    Length    Content Type    Last Modified
------------  -----------  -----------  --------  --------------  -------------------------
testfile.txt  BlockBlob    Hot          21        text/plain      2026-06-05T12:35:12+00:00
```

## Theories (concise)

- Azure Storage Account: a namespace providing access to Blob/File/Queue/Table services. Accounts have keys and support RBAC, SAS, and Managed Identity options.
- Blob Container: logical grouping for blobs. Access can be public, blob-level, or private. For production, keep containers private.
- Redundancy (LRS): stores three copies locally within the region for durability.
- Authentication: Shared Key (account key), SAS tokens, or Azure AD (preferred when possible). Managed Identity + RBAC is recommended for VMs running applications.
- Network restrictions: storage accounts support firewall and private endpoint configuration to limit network access.

## Mistakes Observed, Root Cause Analysis, and Fixes

Mistake 1: CLI printed a warning when creating the container indicating no credentials were provided.

- Why it happened: `az storage container create` was run without an explicit `--account-key` or `--connection-string` and the CLI falls back to querying keys or using login-based auth. The environment did not supply credentials.
- How it was fixed: Retrieved the account key via `az storage account keys list` and then re-ran commands with `--account-key`, or use `--auth-mode login` after ensuring the CLI user has proper RBAC role.

Mistake 2: A broken typed command produced `-bash: /root: Is a directory`.

- Why it happened: A stray character or accidental redirection caused bash to treat `/root` incorrectly. This is a typing error rather than a platform error.
- How it was fixed: Re-entered the intended command properly and avoided accidental trailing characters.

Mistake 3: Partial command copy produced truncated `virtua lMachine` field while querying VM IP.

- Why it happened: Line breaks or copy-paste errors in the CLI command caused it to be malformed. The query string must match the exact JSON path.
- How it was fixed: Re-run the full `az vm list-ip-addresses` command with correct quoting and query expression.

Recommended Fixes / Best Practices (prevention):

- Avoid using account keys in long-term automation. Use Managed Identity and grant `Storage Blob Data Contributor` role to the VM's identity.
- Use `--auth-mode login` in CLI when signed in and appropriate RBAC is configured.
- For production, enable network restrictions and private endpoints and use RBAC+Managed Identity.
- Do not commit keys into repositories; store them in Key Vault and reference securely.

## Verification Steps (repeatable)

1. Retrieve account keys or configure Managed Identity.
2. Upload a test file from the VM using `az storage blob upload` with the correct `--account-key` or connection method.
3. Run `az storage blob list --output table` and confirm `testfile.txt` appears.

## Appendix — Full recorded outputs (selected)

Storage account key (example):

```
OSlzSBzQzmXqg5wecKOgyRenRebSqAq6dAK4dWMni2sQlBMRAuU8U2RBESf672DzL5BgMA0ePRDe+AStiOUhHQ==
```

Blob list table (example):

```
Name          Blob Type    Blob Tier    Length    Content Type    Last Modified
------------  -----------  -----------  --------  --------------  -------------------------
testfile.txt  BlockBlob    Hot          21        text/plain      2026-06-05T12:35:12+00:00
```

---

If you want, I can:

- Convert access to a Managed Identity + RBAC example and show Terraform/ARM/Bicep for these resources.
- Commit these files to a git branch and run a quick lint/format.
