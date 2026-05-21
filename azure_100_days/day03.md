# Day 03 - Azure VM Creation

## Example Output

```bash
~ ➜  azute-client
-bash: azute-client: command not found

~ ✖ azure-client
-bash: azure-client: command not found

~ ✖ az group list
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-f4278f4a2a4b4c14",
    "location": "westus",
    "managedBy": null,
    "name": "kml_rg_main-f4278f4a2a4b4c14",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]

~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-f4278f4a2a4b4c14  westus      Succeeded

~ ➜  az vm create \
  --resource-group kml_rg_main-f4278f4a2a4b4c14 \
  --name devops-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30 \
  --no-wait
SSH key files '/root/.ssh/id_rsa' and '/root/.ssh/id_rsa.pub' have been generated under ~/.ssh to allow SSH access to the VM. If using machines without permanent storage, back up your keys to a safe location.

~ ➜  az vm get-instance-view \
  --name devops-vm \
  --resource-group YourResourceGroupName \
  --query "instanceView.statuses[1].displayStatus"
(AuthorizationFailed) The client 'cbd039aa-fdf3-486c-986e-f8c59802ac21' with object id '0b82e846-a025-4b32-b780-5595a658685a' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/read' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/YourResourceGroupName/providers/Microsoft.Compute/virtualMachines/devops-vm' or the scope is invalid. If access was recently granted, please refresh your credentials.
Code: AuthorizationFailed
Message: The client 'cbd039aa-fdf3-486c-986e-f8c59802ac21' with object id '0b82e846-a025-4b32-b780-5595a658685a' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/read' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/YourResourceGroupName/providers/Microsoft.Compute/virtualMachines/devops-vm' or the scope is invalid. If access was recently granted, please refresh your credentials.

~ ✖ az vm get-instance-view \
  --name devops-vm \
  --resource-group kml_rg_main-f4278f4a2a4b4c14 \
  --query "instanceView.statuses[1].displayStatus"
"VM running"

~ ➜  
```

## Procedure

To create this specific virtual machine using the Azure CLI, you can use the `az vm create` command. Below is the structured command that incorporates all your requirements, followed by a breakdown of what each parameter does.

### Azure CLI Command

```bash
az vm create \
  --resource-group YourResourceGroupName \
  --name devops-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30 \
  --no-wait
```

### Parameter Breakdown

| Parameter | Purpose |
|-----------|---------|
| `--resource-group` | Specifies the logical container where your VM will live. |
| `--name` | Sets the VM name to devops-vm. |
| `--image` | Uses the Ubuntu 22.04 LTS image. |
| `--size` | Sets the hardware profile to Standard_B2s (2 vCPUs, 4GiB RAM). |
| `--admin-username` | Configures the default user as azureuser. |
| `--generate-ssh-keys` | Creates RSA public/private key pairs if they don't exist. |
| `--storage-sku` | Ensures the disk uses Standard_LRS (Locally Redundant Storage). |
| `--os-disk-size-gb` | Sets the OS disk capacity to 30GB. |

## Ensuring the "Running" State

By default, Azure starts the VM immediately after the provisioning process is complete. If you want to verify the state after the command finishes, you can run:

```bash
az vm get-instance-view \
  --name devops-vm \
  --resource-group YourResourceGroupName \
  --query "instanceView.statuses[1].displayStatus"
```

**Note:** Ensure you replace `YourResourceGroupName` with the actual name of a resource group you have already created. If you haven't created one yet, use:

```bash
az group create --name MyRG --location eastus
```

## Key Takeaways

- Use `--generate-ssh-keys` to automatically create SSH keys for secure VM access
- The `--no-wait` flag allows the command to return immediately without waiting for VM provisioning to complete
- Always verify the resource group exists and you have proper permissions before creating VMs
- Use `az vm get-instance-view` to check VM status and verify it's running
