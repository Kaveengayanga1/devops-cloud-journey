# Day 41: Azure Storage Account and Table Storage Verification

- Date: 2026-06-19
- Topic: Create an Azure Storage Account named `xfusiontablest27243`, create a Table Storage table named `tasks`, insert two entities, and verify completion through Azure CLI
- Project: Azure 100 Days — Storage and Table Storage example

## Server Credentials (Historical / Example Values)

- Azure Console URL: https://portal.azure.com/azurefreekmlprod.onmicrosoft.com
- Azure User Name: kk_lab_user_main-193ab5bbc1a74760@azurefreekmlprod.onmicrosoft.com
- Azure Password: PE9M#x3L
- Azure Application Client ID: 012c675a-b46c-4644-b527-db2f6d5f5d61
- Azure Client Secret: VAD8Q~\_j9Ham7Ky7_n7-~7.6023jhTQUExMxXaHn
- Azure Session End Time: Fri Jun 19 17:34:47 UTC 2026

> Note: These values are historical/example credentials captured for documentation purposes only. They are no longer active.

## Task Output

### Credential Enumeration

```bash
~ ➜  showcreds
╒═════════════════════════════╤════════════════════════════════════════════════════════════════════╕
│ Name                        │ Value                                                              │
╞═════════════════════════════╪════════════════════════════════════════════════════════════════════╡
│ Azure Console URL           │ https://portal.azure.com/azurefreekmlprod.onmicrosoft.com          │
├─────────────────────────────┼────────────────────────────────────────────────────────────────────┤
│ Azure User Name             │ kk_lab_user_main-193ab5bbc1a74760@azurefreekmlprod.onmicrosoft.com │
├─────────────────────────────┼────────────────────────────────────────────────────────────────────┤
│ Azure Password              │ PE9M#x3L                                                           │
├─────────────────────────────┼────────────────────────────────────────────────────────────────────┤
│ Azure Application Client ID │ 012c675a-b46c-4644-b527-db2f6d5f5d61                               │
├─────────────────────────────┼────────────────────────────────────────────────────────────────────┤
│ Azure Client Secret         │ VAD8Q~_j9Ham7Ky7_n7-~7.6023jhTQUExMxXaHn                           │
├─────────────────────────────┼────────────────────────────────────────────────────────────────────┤
│ Azure Session End Time      │ Fri Jun 19 17:34:47 UTC 2026                                       │
╘═════════════════════════════╧════════════════════════════════════════════════════════════════════╛
```

### Resource Group Verification

```bash
~ ➜  az group list
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-193ab5bbc1a74760",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-193ab5bbc1a74760",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

### Deployment Status Verification

```bash
~ ➜  az deployment group list --resource-group kml_rg_main-193ab5bbc1a74760 --query "[].{DeploymentName:name, ProvisioningState:properties.provisioningState}" --output table
DeploymentName                     ProvisioningState
---------------------------------  -------------------
xfusiontablest27243_1781887230110  Succeeded
```

### Resource Status Verification

```bash
~ ➜  az resource list --resource-group kml_rg_main-193ab5bbc1a74760 --query "[].{ResourceName:name, ResourceType:type, ProvisioningState:provisioningState}" --output table
ResourceName         ResourceType                       ProvisioningState
-------------------  ---------------------------------  -------------------
xfusiontablest27243  Microsoft.Storage/storageAccounts  Succeeded
```

## Theories

### Azure Storage Account

An Azure Storage Account is the top-level namespace for storage services such as Blob, Queue, File, and Table storage. In this task, the storage account `xfusiontablest27243` was created as the foundation for Table Storage operations.

### Azure Table Storage

Azure Table Storage is a NoSQL key-value datastore designed for large-scale structured data. Each entity is identified by a combination of `PartitionKey` and `RowKey`. In this task:

- `PartitionKey` grouped the task entities under the logical partition `tasks`
- `RowKey` uniquely identified each entity within the partition
- Additional properties such as `description` and `status` stored the task metadata

### Entity Design

The two entities used in the task demonstrate a minimal operational model:

- Task 1: `PartitionKey=tasks`, `RowKey=1`, `description=Learn Table Storage`, `status=completed`
- Task 2: `PartitionKey=tasks`, `RowKey=2`, `description=Build To-Do App`, `status=in-progress`

This model is appropriate when fast key-based access and simple status tracking are required.

### Azure CLI Validation

The Azure CLI commands verify the state of the environment at different layers:

- `az group list` confirms the resource group exists and is provisioned
- `az deployment group list` checks whether deployments in the resource group completed successfully
- `az resource list` confirms each resource has a successful provisioning state

Using both deployment-level and resource-level checks is more reliable than relying on a single command.

## Mistakes and Fixes

### Mistake 1: Treating resource group existence as proof of completion

- What happened: The presence of `kml_rg_main-193ab5bbc1a74760` in `az group list` confirmed the resource group existed, but that alone did not prove that all deployments had completed successfully.
- Why it happened: Resource group existence and deployment completion are separate states. A resource group can exist while one or more deployments are still running or have failed.
- How it was fixed:
  1. Used `az deployment group list --resource-group kml_rg_main-193ab5bbc1a74760` to inspect deployment-level status.
  2. Confirmed the deployment `xfusiontablest27243_1781887230110` showed `Succeeded`.
  3. Cross-checked with `az resource list` to confirm the storage account itself also had `ProvisioningState` of `Succeeded`.

### Mistake 2: Relying on table UI inspection without explicit verification

- What happened: The table data was described as visible in Azure Storage browser, but manual inspection alone is not a sufficient audit record for task completion.
- Why it happened: UI confirmation can be visually correct while still lacking a reproducible command-based verification trail.
- How it was fixed:
  1. Inserted the entities with the expected `PartitionKey`, `RowKey`, and property values.
  2. Verified the final state through the table browser and, where applicable, refreshed the view.
  3. Documented the exact entity definitions in this file so the values can be audited later.

### Mistake 3: Documenting active secrets without labeling them as historical

- What happened: The task included sensitive credentials that could be misread as active if not clearly labeled.
- Why it happened: The output was captured from a live lab session, but the documentation needs to distinguish historical/example values from current operational secrets.
- How it was fixed:
  1. Presented the values under a dedicated historical/example credentials section.
  2. Added an explicit note stating that the values are no longer active.
  3. Kept the values only as documentation artifacts for reproducibility and context.

## Summary

The task successfully established the Azure Storage account `xfusiontablest27243`, validated the resource group and deployment state, and confirmed that the created storage resource reached `Succeeded`. The Azure CLI results provide a clear audit trail for the environment, while the theory and mistake analysis explain how to verify completion correctly and avoid misinterpreting partial status signals.
