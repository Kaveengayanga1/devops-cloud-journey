# Day 17: Public Access to Azure Blob Storage

## Overview
Today's task involves configuring an Azure Storage Account to allow public access and creating a container that permits anonymous read access to its blobs. This is crucial for hosting public assets like images or static files, but must be done extensively carefully to avoid data leaks.

## 1. Create a Storage Account with Public Access

### Theory
By default, Azure Storage accounts are secure and deny public access. To allow anonymous users to read data, you must explicitly enable the `AllowBlobPublicAccess` property on the storage account.

*   **Public Access**: Controls whether containers in the account *can* be configured for public access. It does not automatically make data public; it just unlocks the capability.
*   **Security Risk**: Enabling this feature potentially exposes data to the internet. Always ensure sensitive data is not stored in public containers.

### Execution
First, we create the storage account. Then, we update it to allow public blob access.

```bash
# Variables
RESOURCE_GROUP="kml_rg_main-d800000fbead47b7"
STORAGE_ACCOUNT="datacenterst18507"
LOCATION="eastus"

# Create Storage Account
az storage account create \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku Standard_LRS \
    --kind StorageV2

# Enable Public Access
az storage account update \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --allow-blob-public-access true
```

## 2. Create a Public Container

### Theory
Enabling public access on the account is step one. Step two is configuring the specific **Container** to accept anonymous requests.

*   **Access Level (`blob`)**: Allows anonymous read access for blobs only. Clients can read blob data but cannot enumerate (list) the blobs within the container.
*   **Access Level (`container`)**: Allows listing and reading.

### Execution
Create a container named `devops-blob-18064` with blob-level public access.

```bash
CONTAINER_NAME="datacenter-blob-4086"

az storage container create \
    --name $CONTAINER_NAME \
    --account-name $STORAGE_ACCOUNT \
    --public-access container \
    --auth-mode login
```

## 3. Upload & Verify

### Theory
Once the container is public, any blob uploaded to it can be accessed via a direct URL without authentication tokens (SAS or Keys).

### Execution
Upload a text file and verify access using `curl`.

```bash
# Create a test file
echo "Hello, this is a public data migration test file for Nautilus." > test-file.txt

# Upload the file
az storage blob upload \
    --account-name $STORAGE_ACCOUNT \
    --container-name $CONTAINER_NAME \
    --name my-test-blob.txt \
    --file test-file.txt \
    --auth-mode login

# Verify access (Result should be the query content)
curl https://$STORAGE_ACCOUNT.blob.core.windows.net/$CONTAINER_NAME/my-test-blob.txt
```

## Output

```console
~ ➜  az group list
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-d800000fbead47b7",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-d800000fbead47b7",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]

~ ➜  az storage account create \
  --name datacenterst18507 \
  --resource-group kml_rg_main-d800000fbead47b7 \
  --location eastus \
  --sku Standard_LRS \
  --allow-blob-public-access true
{
  "accessTier": "Hot",
  "accountMigrationInProgress": null,
  "allowBlobPublicAccess": true,
  "allowCrossTenantReplication": false,
  "allowSharedKeyAccess": null,
  "allowedCopyScope": null,
  "azureFilesIdentityBasedAuthentication": null,
  "blobRestoreStatus": null,
  "creationTime": "2026-03-19T04:06:02.840500+00:00",
  "customDomain": null,
  "defaultToOAuthAuthentication": null,
  "dnsEndpointType": null,
  "enableExtendedGroups": null,
  "enableHttpsTrafficOnly": true,
  "enableNfsV3": null,
  "encryption": {
    "encryptionIdentity": null,
    "keySource": "Microsoft.Storage",
    "keyVaultProperties": null,
    "requireInfrastructureEncryption": null,
    "services": {
      "blob": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-03-19T04:06:03.199877+00:00"
      },
      "file": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-03-19T04:06:03.199877+00:00"
      },
      "queue": null,
      "table": null
    }
  },
  "extendedLocation": null,
  "failoverInProgress": null,
  "geoReplicationStats": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-d800000fbead47b7/providers/Microsoft.Storage/storageAccounts/datacenterst18507",
  "identity": null,
  "immutableStorageWithVersioning": null,
  "isHnsEnabled": null,
  "isLocalUserEnabled": null,
  "isSftpEnabled": null,
  "isSkuConversionBlocked": null,
  "keyCreationTime": {
    "key1": "2026-03-19T04:06:03.184243+00:00",
    "key2": "2026-03-19T04:06:03.184243+00:00"
  },
  "keyPolicy": null,
  "kind": "StorageV2",
  "largeFileSharesState": null,
  "lastGeoFailoverTime": null,
  "location": "eastus",
  "minimumTlsVersion": "TLS1_0",
  "name": "datacenterst18507",
  "networkRuleSet": {
    "bypass": "AzureServices",
    "defaultAction": "Allow",
    "ipRules": [],
    "ipv6Rules": [],
    "resourceAccessRules": null,
    "virtualNetworkRules": []
  },
  "primaryEndpoints": {
    "blob": "https://datacenterst18507.blob.core.windows.net/",
    "dfs": "https://datacenterst18507.dfs.core.windows.net/",
    "file": "https://datacenterst18507.file.core.windows.net/",
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://datacenterst18507.queue.core.windows.net/",
    "table": "https://datacenterst18507.table.core.windows.net/",
    "web": "https://datacenterst18507.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "privateEndpointConnections": [],
  "provisioningState": "Succeeded",
  "publicNetworkAccess": null,
  "resourceGroup": "kml_rg_main-d800000fbead47b7",
  "routingPreference": null,
  "sasPolicy": null,
  "secondaryEndpoints": null,
  "secondaryLocation": null,
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": null,
  "storageAccountSkuConversionStatus": null,
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}

~ ➜  az storage account update \
  --name datacenterst18507 \
  --resource-group kml_rg_main-d800000fbead47b7 \
  --allow-blob-public-access true
{
  "accessTier": "Hot",
  "accountMigrationInProgress": null,
  "allowBlobPublicAccess": true,
  "allowCrossTenantReplication": false,
  "allowSharedKeyAccess": null,
  "allowedCopyScope": null,
  "azureFilesIdentityBasedAuthentication": null,
  "blobRestoreStatus": null,
  "creationTime": "2026-03-19T04:06:02.840500+00:00",
  "customDomain": null,
  "defaultToOAuthAuthentication": null,
  "dnsEndpointType": null,
  "enableExtendedGroups": null,
  "enableHttpsTrafficOnly": true,
  "enableNfsV3": null,
  "encryption": {
    "encryptionIdentity": null,
    "keySource": "Microsoft.Storage",
    "keyVaultProperties": null,
    "requireInfrastructureEncryption": null,
    "services": {
      "blob": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-03-19T04:06:03.199877+00:00"
      },
      "file": {
        "enabled": true,
        "keyType": "Account",
        "lastEnabledTime": "2026-03-19T04:06:03.199877+00:00"
      },
      "queue": null,
      "table": null
    }
  },
  "extendedLocation": null,
  "failoverInProgress": null,
  "geoReplicationStats": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-d800000fbead47b7/providers/Microsoft.Storage/storageAccounts/datacenterst18507",
  "identity": null,
  "immutableStorageWithVersioning": null,
  "isHnsEnabled": null,
  "isLocalUserEnabled": null,
  "isSftpEnabled": null,
  "isSkuConversionBlocked": null,
  "keyCreationTime": {
    "key1": "2026-03-19T04:06:03.184243+00:00",
    "key2": "2026-03-19T04:06:03.184243+00:00"
  },
  "keyPolicy": null,
  "kind": "StorageV2",
  "largeFileSharesState": null,
  "lastGeoFailoverTime": null,
  "location": "eastus",
  "minimumTlsVersion": "TLS1_0",
  "name": "datacenterst18507",
  "networkRuleSet": {
    "bypass": "AzureServices",
    "defaultAction": "Allow",
    "ipRules": [],
    "ipv6Rules": [],
    "resourceAccessRules": null,
    "virtualNetworkRules": []
  },
  "primaryEndpoints": {
    "blob": "https://datacenterst18507.blob.core.windows.net/",
    "dfs": "https://datacenterst18507.dfs.core.windows.net/",
    "file": "https://datacenterst18507.file.core.windows.net/",
    "internetEndpoints": null,
    "microsoftEndpoints": null,
    "queue": "https://datacenterst18507.queue.core.windows.net/",
    "table": "https://datacenterst18507.table.core.windows.net/",
    "web": "https://datacenterst18507.z13.web.core.windows.net/"
  },
  "primaryLocation": "eastus",
  "privateEndpointConnections": [],
  "provisioningState": "Succeeded",
  "publicNetworkAccess": null,
  "resourceGroup": "kml_rg_main-d800000fbead47b7",
  "routingPreference": null,
  "sasPolicy": null,
  "secondaryEndpoints": null,
  "secondaryLocation": null,
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": null,
  "storageAccountSkuConversionStatus": null,
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}

~ ➜  az storage container create \
  --name datacenter-blob-4086 \
  --account-name datacenterst18507 \
  --public-access container

There are no credentials provided in your command and environment, we will query for account key for your storage account.
It is recommended to provide --connection-string, --account-key or --sas-token in your command as credentials.

You also can add `--auth-mode login` in your command to use Azure Active Directory (Azure AD) for authorization if your login account is assigned required RBAC roles.
For more information about RBAC roles in storage, visit https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-cli.

In addition, setting the corresponding environment variables can avoid inputting credentials in your command. Please use --help to get more information about environment variable usage.
{
  "created": true
}

~ ➜  az storage container show \
  --name datacenter-blob-4086 \
  --account-name datacenterst18507 \
  --query "{Name:name, PublicAccess:properties.publicAccess}" \
  --output table

There are no credentials provided in your command and environment, we will query for account key for your storage account.
It is recommended to provide --connection-string, --account-key or --sas-token in your command as credentials.

You also can add `--auth-mode login` in your command to use Azure Active Directory (Azure AD) for authorization if your login account is assigned required RBAC roles.
For more information about RBAC roles in storage, visit https://docs.microsoft.com/azure/storage/common/storage-auth-aad-rbac-cli.

In addition, setting the corresponding environment variables can avoid inputting credentials in your command. Please use --help to get more information about environment variable usage.
Name                  PublicAccess
--------------------  --------------
datacenter-blob-4086  container

~ ➜  echo "Hello, this is a public data migration test file for Nautilus." > test-file.txt

~ ➜  ls
test-file.txt

~ ➜  az storage blob upload \
  --account-name datacenterst18507 \
  --container-name datacenter-blob-4086 \
  --name my-test-blob.txt \
  --file test-file.txt \
  --auth-mode login\
> ^C

~ ✖ az storage blob upload   --account-name datacenterst18507   --container-name datacenter-blob-4086   --name my-test-blob.txt   --file test-file.txt   --auth-mode login
Finished[#############################################################]  100.0000%
{
  "client_request_id": "80872852-2349-11f1-93b2-3ac6216b6bc8",
  "content_md5": "eIpjXvpl+ciOl96M0NB/kg==",
  "date": "2026-03-19T04:10:03+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DE856D65F20E16\"",
  "lastModified": "2026-03-19T04:10:04+00:00",
  "request_id": "c0b6b294-901e-00e4-4c56-b76521000000",
  "request_server_encrypted": true,
  "version": "2022-11-02",
  "version_id": null
}

~ ➜  curl -I https://datacenterst18507.blob.core.windows.net/datacenter-blob-4086/my-test-blob.txt
HTTP/1.1 200 OK
Content-Length: 63
Content-Type: text/plain
Content-MD5: eIpjXvpl+ciOl96M0NB/kg==
Last-Modified: Thu, 19 Mar 2026 04:10:04 GMT
ETag: 0x8DE856D65F20E16
x-ms-request-id: 11456de8-401e-003c-2156-b74278000000
x-ms-version: 2009-09-19
x-ms-lease-status: unlocked
x-ms-blob-type: BlockBlob
Date: Thu, 19 Mar 2026 04:11:25 GMT


~ ➜  
```
