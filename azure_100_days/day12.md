# Day 12: Azure VM Tagging

## Theory

The Nautilus DevOps team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is not tagged properly so they decided to tag it as needed.

Add the tag `Environment=dev` to the virtual machine named `datacenter-vm`.

### Why Use Tags in Azure?

**Tags** are key-value pairs used to organize, track, and manage cloud resources. They are essential for:

- **Resource Organization**: Categorize VMs by project, environment, or department
- **Cost Management**: Track expenses by environment (dev, prod, staging)
- **Automation**: Trigger automated actions based on specific tags
- **Compliance**: Ensure resources meet organizational policies

## Output

```bash
~ ➜  az vm list --query "[?name=='datacenter-vm'].{ResourceGroup:resourceGroup}" -o table
ResourceGroup
----------------------------
KML_RG_MAIN-A87925844D504658
```

```bash
~ ➜  az vm update \
  --resource-group KML_RG_MAIN-A87925844D504658 \
  --name datacenter-vm \
  --set tags.Environment=dev
{
  "additionalCapabilities": null,
  "applicationProfile": null,
  "availabilitySet": null,
  "billingProfile": null,
  "capacityReservation": null,
  "diagnosticsProfile": null,
  "etag": "\"2\"",
  "evictionPolicy": null,
  "extendedLocation": null,
  "extensionsTimeBudget": null,
  "hardwareProfile": {
    "vmSize": "Standard_B1s",
    "vmSizeProperties": null
  },
  "host": null,
  "hostGroup": null,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/KML_RG_MAIN-A87925844D504658/providers/Microsoft.Compute/virtualMachines/datacenter-vm",
  "identity": null,
  "instanceView": null,
  "licenseType": null,
  "location": "westus",
  "managedBy": null,
  "name": "datacenter-vm",
  "networkProfile": {
    "networkApiVersion": null,
    "networkInterfaceConfigurations": null,
    "networkInterfaces": [
      {
        "deleteOption": null,
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-a87925844d504658/providers/Microsoft.Network/networkInterfaces/datacenter-vmVMNic",
        "primary": null,
        "resourceGroup": "kml_rg_main-a87925844d504658"
      }
    ]
  },
  "osProfile": {
    "adminPassword": null,
    "adminUsername": "azureuser",
    "allowExtensionOperations": true,
    "computerName": "datacenter-vm",
    "customData": null,
    "linuxConfiguration": {
      "disablePasswordAuthentication": true,
      "enableVmAgentPlatformUpdates": null,
      "patchSettings": {
        "assessmentMode": "ImageDefault",
        "automaticByPlatformSettings": null,
        "patchMode": "ImageDefault"
      },
      "provisionVmAgent": true,
      "ssh": {
        "publicKeys": [
          {
            "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD3xQd1BaYCfU1H8RQfeHeNCnUjrC/lyi0ZgOafHVELgxbwgPH5K+ImptO80Aglkh5skDz3ythpPBsN2+eugElK28IMqTFWaxpkjE1qq3EQaO0tn61lhRWjondw0FZrebSzRQQwH0kB1inmKSAWJtZzkIS/WEEnXa6P9FExakfm+3jPjkhEYtWQm8BwgV8GGsGMM8RnoaaYXouZvtVpoGygiaHHeceUNRK5ZZZXoEf/O9KVs4TiwPW9Y9XNswaRvNdqDYblYOWLrkpk1CoP7d8TwbYJyX+t1v3MaOd0nVzj3CgTncv7RbBEcPyo3hqYQ+N/H8oFCTRkiGRKhOlntpXT root@azure-client\n",
            "path": "/home/azureuser/.ssh/authorized_keys"
          }
        ]
      }
    },
    "requireGuestProvisionSignal": true,
    "secrets": [],
    "windowsConfiguration": null
  },
  "plan": null,
  "platformFaultDomain": null,
  "priority": null,
  "provisioningState": "Succeeded",
  "proximityPlacementGroup": null,
  "resourceGroup": "KML_RG_MAIN-A87925844D504658",
  "resources": null,
  "scheduledEventsPolicy": null,
  "scheduledEventsProfile": null,
  "securityProfile": {
    "encryptionAtHost": null,
    "encryptionIdentity": null,
    "proxyAgentSettings": null,
    "securityType": "TrustedLaunch",
    "uefiSettings": {
      "secureBootEnabled": true,
      "vTpmEnabled": true
    }
  },
  "storageProfile": {
    "dataDisks": [],
    "diskControllerType": "SCSI",
    "imageReference": {
      "communityGalleryImageId": null,
      "exactVersion": "22.04.202601310",
      "id": null,
      "offer": "0001-com-ubuntu-server-jammy",
      "publisher": "Canonical",
      "sharedGalleryImageId": null,
      "sku": "22_04-lts-gen2",
      "version": "latest"
    },
    "osDisk": {
      "caching": "ReadWrite",
      "createOption": "FromImage",
      "deleteOption": "Detach",
      "diffDiskSettings": null,
      "diskSizeGb": 30,
      "encryptionSettings": null,
      "image": null,
      "managedDisk": {
        "diskEncryptionSet": null,
        "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-a87925844d504658/providers/Microsoft.Compute/disks/datacenter-vm_disk1_90d6d5875a784413a28ffe9e0cfd2788",
        "resourceGroup": "kml_rg_main-a87925844d504658",
        "securityProfile": null,
        "storageAccountType": "Standard_LRS"
      },
      "name": "datacenter-vm_disk1_90d6d5875a784413a28ffe9e0cfd2788",
      "osType": "Linux",
      "vhd": null,
      "writeAcceleratorEnabled": null
    }
  },
  "tags": {
    "Environment": "dev"
  },
  "timeCreated": "2026-03-05T05:40:20.132736+00:00",
  "type": "Microsoft.Compute/virtualMachines",
  "userData": null,
  "virtualMachineScaleSet": null,
  "vmId": "eb92a2c6-e759-4eff-a387-eb62023d60e6",
  "zones": null
}
```

```bash
~ ➜  az vm show \
  --resource-group KML_RG_MAIN-A87925844D504658 \
  --name datacenter-vm \
  --query tags
{
  "Environment": "dev"
}

~ ➜  
```

## Commands Used

| Command | Description |
|---------|-------------|
| `az vm list` | List VMs with filtering |
| `az vm update` | Update VM configuration with new tags |
| `--set tags.Key=Value` | Add or update a tag on the resource |
| `az vm show --query tags` | Display only the tags of a VM |
