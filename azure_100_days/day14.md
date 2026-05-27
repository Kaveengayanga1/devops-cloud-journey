# Day 14: Azure Managed Disks

## Cloud Theory

### Azure Managed Disks
Azure Managed Disks are block-level storage volumes that are managed by Azure and used with Azure Virtual Machines. Managed disks are like a physical disk in on-premises server but virtualized. With managed disks, all you have to do is specify the disk size, the disk type, and provision the disk. Once you provision the disk, Azure handles the rest.

### Standard_LRS (Locally Redundant Storage)
*   **Standard**: This refers to Standard HDD (Hard Disk Drive) based storage. It is a low-cost option suitable for bulk storage or non-critical workloads.
*   **LRS**: Locally Redundant Storage replicates your data three times within a single physical location in the primary region. LRS provides at least 99.999999999% (11 nines) durability of objects over a given year.

### Azure Resource Group
A Resource Group is a container that holds related resources for an Azure solution. The resource group can include all the resources for the solution, or only those resources that you want to manage as a group. You decide how you want to allocate resources to resource groups based on what makes the most sense for your organization.

### Azure CLI
The Azure Command-Line Interface (CLI) is a cross-platform command-line tool to connect to Azure and execute administrative commands on Azure resources. It allows the execution of commands through a terminal using interactive command-line prompts or a script.

## Task
Create a Standard_LRS managed disk named `xfusion-disk` with a size of 2GB inside the existing resource group.

## Steps

### Step 1: Login to Azure
Connect to your Azure account using the CLI.
```bash
az login
```

### Step 2: Identify Resource Group
List the existing resource groups to find the correct one for your assignment. In this lab environment, the resource group name is dynamically generated.
```bash
az group list --output table
```
*Take note of the resource group name (e.g., `kml_rg_main-...`) from the output.*

### Step 3: Create Managed Disk
Create the disk using the `az disk create` command.
```bash
az disk create \
  --resource-group <Your-Resource-Group-Name> \
  --name xfusion-disk \
  --sku Standard_LRS \
  --size-gb 2
```
*   `--resource-group`: The name of the resource group found in Step 2.
*   `--name`: The name of the disk (`xfusion-disk`).
*   `--sku`: The storage type (`Standard_LRS`).
*   `--size-gb`: The size of the disk in gigabytes (`2`).

### Step 4: Verification
Verify that the disk has been created successfully.
```bash
az disk show --resource-group <Your-Resource-Group-Name> --name xfusion-disk
```

## Execution & Output

```json
~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-309265ad2bbc4bf7  eastus      Succeeded

~ ➜  az disk create \
  --resource-group kml_rg_main-309265ad2bbc4bf7 \
  --name xfusion-disk \
  --sku Standard_LRS \
  --size-gb 2
{
  "burstingEnabled": null,
  "burstingEnabledTime": null,
  "completionPercent": null,
  "creationData": {
    "createOption": "Empty",
    "elasticSanResourceId": null,
    "galleryImageReference": null,
    "imageReference": null,
    "logicalSectorSize": null,
    "performancePlus": null,
    "securityDataUri": null,
    "sourceResourceId": null,
    "sourceUniqueId": null,
    "sourceUri": null,
    "storageAccountId": null,
    "uploadSizeBytes": null
  },
  "dataAccessAuthMode": null,
  "diskAccessId": null,
  "diskIopsReadOnly": null,
  "diskIopsReadWrite": 500,
  "diskMBpsReadOnly": null,
  "diskMBpsReadWrite": 60,
  "diskSizeBytes": 2147483648,
  "diskSizeGb": 2,
  "diskState": "Unattached",
  "encryption": {
    "diskEncryptionSetId": null,
    "type": "EncryptionAtRestWithPlatformKey"
  },
  "encryptionSettingsCollection": null,
  "extendedLocation": null,
  "hyperVGeneration": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-309265ad2bbc4bf7/providers/Microsoft.Compute/disks/xfusion-disk",
  "lastOwnershipUpdateTime": null,
  "location": "eastus",
  "managedBy": null,
  "managedByExtended": null,
  "maxShares": null,
  "name": "xfusion-disk",
  "networkAccessPolicy": "AllowAll",
  "optimizedForFrequentAttach": null,
  "osType": null,
  "propertyUpdatesInProgress": null,
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Enabled",
  "purchasePlan": null,
  "resourceGroup": "kml_rg_main-309265ad2bbc4bf7",
  "securityProfile": null,
  "shareInfo": null,
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "supportedCapabilities": null,
  "supportsHibernation": null,
  "tags": {},
  "tier": null,
  "timeCreated": "2026-03-10T02:49:12.988883+00:00",
  "type": "Microsoft.Compute/disks",
  "uniqueId": "e8726517-5fed-4a87-b60e-3b75a079e0f4",
  "zones": null
}

~ ➜  az disk show --resource-group nautilus-rg --name xfusion-disk
(AuthorizationFailed) The client '181c1a1e-0f9e-4881-8edf-41e8e08440da' with object id '8d998f7b-32e6-429d-9e8e-9243e5b45a4e' does not have authorization to perform action 'Microsoft.Compute/disks/read' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/nautilus-rg/providers/Microsoft.Compute/disks/xfusion-disk' or the scope is invalid. If access was recently granted, please refresh your credentials.
Code: AuthorizationFailed
Message: The client '181c1a1e-0f9e-4881-8edf-41e8e08440da' with object id '8d998f7b-32e6-429d-9e8e-9243e5b45a4e' does not have authorization to perform action 'Microsoft.Compute/disks/read' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/nautilus-rg/providers/Microsoft.Compute/disks/xfusion-disk' or the scope is invalid. If access was recently granted, please refresh your credentials.

~ ✖ az disk show --resource-group kml_rg_main-309265ad2bbc4bf7 --name xfusion-disk
{
  "creationData": {
    "createOption": "Empty"
  },
  "diskIOPSReadWrite": 500,
  "diskMBpsReadWrite": 60,
  "diskSizeBytes": 2147483648,
  "diskSizeGB": 2,
  "diskState": "Unattached",
  "encryption": {
    "type": "EncryptionAtRestWithPlatformKey"
  },
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-309265ad2bbc4bf7/providers/Microsoft.Compute/disks/xfusion-disk",
  "location": "eastus",
  "name": "xfusion-disk",
  "networkAccessPolicy": "AllowAll",
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Enabled",
  "resourceGroup": "kml_rg_main-309265ad2bbc4bf7",
  "sku": {
    "name": "Standard_LRS",
    "tier": "Standard"
  },
  "tags": {},
  "timeCreated": "2026-03-10T02:49:12.9888832+00:00",
  "type": "Microsoft.Compute/disks",
  "uniqueId": "e8726517-5fed-4a87-b60e-3b75a079e0f4"
}
```
