# Day 16: Azure Storage Account & Blob Storage Operations

## Overview
Today's focused task is to provision an Azure Storage Account and upload a data file as a blob using the Azure CLI. This is a foundational operation for any Cloud or DevOps Engineer managing data persistence in Azure.

## 1. Create an Azure Storage Account

### Theory
An **Azure Storage Account** acts as a top-level container that provides a unique namespace for your Azure Storage data (blobs, files, queues, tables, and disks).

*   **Resource Group**: A logical container that holds related resources for an Azure solution.
*   **SKU (`Standard_LRS`)**: 
    *   **Standard**: Standard performance tier (magnetic drives), suitable for bulk storage.
    *   **LRS (Locally-Redundant Storage)**: Replicates your data three times within a single physical location in the primary region. Low cost, but lower resilience than Zone-Redundant (ZRS).
*   **Kind (`StorageV2`)**: The latest general-purpose storage account type, supporting all storage features.

### Execution
The following command provisions a new storage account in the `eastus` region.

```bash
# Variables
RESOURCE_GROUP="kml_rg_main-a24f285f292c43a6"
STORAGE_ACCOUNT_NAME="devopsst24404"
LOCATION="eastus"

# Create Storage Account
az storage account create \
    --name $STORAGE_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku Standard_LRS \
    --kind StorageV2
```

## 2. Upload a File to Blob Storage

### Theory
**Blob Storage** is Azure's object storage solution, optimized for storing massive amounts of unstructured data.

*   **Container**: Similar to a folder in a file system, a container organizes a set of blobs. You must create a container before you can upload blobs.
*   **Auth Mode (`--auth-mode login`)**: This flag ensures you authorize the operation using your Azure Active Directory (Entra ID) credentials currently logged in, rather than using a shared access key. This is a security best practice.

### Execution
The command below uploads a local file named `hello.txt` to a specific container, renaming it to `uploaded-file-name.txt` in the cloud.

> **Pre-requisite**: Ensure the container (`devops-blob-8389`) exists. You can create it using `az storage container create --name devops-blob-8389 --account-name devopsst24404`.

```bash
CONTAINER_NAME="devops-blob-8389"
FILE_TO_UPLOAD="hello.txt"
BLOB_NAME="uploaded-file-name.txt"

# Upload File
az storage blob upload \
    --account-name $STORAGE_ACCOUNT_NAME \
    --container-name $CONTAINER_NAME \
    --name $BLOB_NAME \
    --file $FILE_TO_UPLOAD \
    --auth-mode login
```
