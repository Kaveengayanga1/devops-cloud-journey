# Day 18: Uploading Files to Azure Blob Storage

## Core Concepts Required (Theory)
Azure Blob Storage is Microsoft's cloud-based object storage solution. There are three main concepts to understand:

1.  **Storage Account**: This provides a unique namespace for your Azure data. All your data objects are stored within this account.
2.  **Container**: Similar to a folder on your computer. Blobs are stored inside containers, and a Storage Account can have an unlimited number of containers.
3.  **Blob**: A file of any type and size (e.g., text files, images, videos).

For this task, we will use the **Azure CLI** (Command Line Interface), a tool to manage Azure resources via the command prompt.

## Step-by-Step Instructions using Azure CLI
As a DevOps or Cloud engineer, follow these steps to upload a file:

### Step 1: Login to Azure Account
First, authenticate to your Azure account via the terminal.

```bash
az login
```
This command will redirect you to a browser to enter your credentials.

### Step 2: Uploading the File
Use the `az storage blob upload` command to upload the local file `/tmp/nautilus.txt` to the specified container.

```bash
az storage blob upload \
    --account-name nautilusst18870 \
    --container-name nautilus-blob-846 \
    --name nautilus.txt \
    --file /tmp/nautilus.txt \
    --auth-mode login
```

**Parameter Explanation:**
*   `--account-name`: The name of your storage account.
*   `--container-name`: The destination container name.
*   `--name`: The name of the file inside the container (Destination name).
*   `--file`: The path to the local file (Source path).
*   `--auth-mode login`: Uses your current Azure AD login credentials for authorization.

**Output:**
```
Finished[#############################################################]  100.0000%
{
  "client_request_id": "ff10801a-2466-11f1-b4ad-7e40e0e4f608",
  "content_md5": "Lu7zilatbGguzSz2Ecn5IQ==",
  "date": "2026-03-20T14:13:42+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DE868AE464DD9F\"",
  "lastModified": "2026-03-20T14:13:42+00:00",
  "request_id": "3ac1c072-801e-006e-0273-b83e90000000",
  "request_server_encrypted": true,
  "version": "2022-11-02",
  "version_id": null
}
```

### Step 3: Verification
Verify that the file has been successfully uploaded by listing the blobs in the container.

```bash
az storage blob list \
    --account-name nautilusst18870 \
    --container-name nautilus-blob-846 \
    --output table
```

This will display a list of all files within the relevant container.

**Output:**
```
There are no credentials provided in your command and environment, we will query for account key for your storage account.
It is recommended to provide --connection-string, --account-key or --sas-token in your command as credentials.

You also can add `--auth-mode login` in your command to use Azure Active Directory (Azure AD) for authorization if your login account is assigned required RBAC roles.
For more information about RBAC roles in storage, visit https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-cli.

In addition, setting the corresponding environment variables can avoid inputting credentials in your command. Please use --help to get more information about environment variable usage.
Name          Blob Type    Blob Tier    Length    Content Type    Last Modified              Snapshot
------------  -----------  -----------  --------  --------------  -------------------------  ----------
nautilus.txt  BlockBlob    Hot          33        text/plain      2026-03-20T14:13:42+00:00
```
