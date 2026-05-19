## Day 1 — Azure RG + SSH Key

### What happened
- Listed current resource groups.
- Failed to create `NautilusRG` due to insufficient permissions.
- Generated SSH key pair `nautilus-kp` in existing group `kml_rg_main-2c9701003f924bd8`.

### Commands and outputs
1) List groups

```
az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-2c9701003f924bd8  westus      Succeeded
```

2) Attempt to create resource group (failed: AuthorizationFailed)

```
az group create --name NautilusRG --location eastus
(AuthorizationFailed) The client '51694ae9-336a-485d-bb85-a65b84aacf49' with object id '2be31796-54f2-4e4d-acc8-dc65685f46c5' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourcegroups/write' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourcegroups/NautilusRG' or the scope is invalid. If access was recently granted, please refresh your credentials.
```

3) Create SSH key pair in existing RG

```
az sshkey create \
  --name nautilus-kp \
  --resource-group kml_rg_main-2c9701003f924bd8 \
  --location westus
No public key is provided. A key pair is being generated for you.
Private key is saved to "/root/.ssh/1767889627_9985514".
Public key is saved to "/root/.ssh/1767889627_9985514.pub".
{
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/KML_RG_MAIN-2C9701003F924BD8/providers/Microsoft.Compute/sshPublicKeys/nautilus-kp",
  "location": "westus",
  "name": "nautilus-kp",
  "publicKey": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC5MYxH4L/Rs384zNSNATDnQ+c/X/PY9H3Su/IuzLFoSJVaqNUAtjgkXx9K9wQncnHpfKjEqA0WzMiJGbggvIGYWytVGhTOCd6Pg/6N09D3innvfqdl5LbBtchWkW6A2estwYN7dp/19/DBGka/v+6RwNNFIv8mhvjVKPDjfI6o8BYQCbDaVTIEkrnPeqzGarOe6nFUYh0cDyOmfK46aQL38ztN6WMWVBzdtIWQqxlD++dWi4LUpuRNbT0BkIkfQjgKtqXV1jnswDJJ+yLc4y7psLwjCiiu77EF5ahiWkr5wrG7weZp79cV3VlSzBK1hdJv2Jz7uTgXa3eAW1MOfLn203tlqrFJXaWkRuCp4DjtIoOZRDhWIjQx8kN+iaLt9s1jNGd92w8l3uVEuN3v3MC/Aa7I8D7GXFn0/dYLHNY5dPatCi0VI9d+4/CmE7Kd5xLeM4TUM+rCjrlwAWtiyeBQ4fxn8smIlNKb1sOcDpxUa1GxnYrBSXe/hDgTsiQ5Y8k= generated-by-azure",
  "resourceGroup": "KML_RG_MAIN-2C9701003F924BD8",
  "tags": null,
  "type": null
}
```

### Guided steps to rerun successfully
1) Ensure access
	- You need a role with write rights on the subscription (e.g., Owner or Contributor). If access was just granted, run `az login` again.
2) Set subscription
	- `az account set --subscription f0c3bcdd-5ce2-4fa0-8cf3-41559747512b`
3) Create resource group
	- `az group create --name NautilusRG --location eastus`
4) Create SSH key in that group
	- `az sshkey create --name nautilus-kp --resource-group NautilusRG --location eastus`
5) Optional: local-only key generation
	- `ssh-keygen -t rsa -f ~/.ssh/nautilus-kp -N ""`

Notes
- Replace names/locations as needed for your project.
- If no RG exists yet, run step 3 before step 4.
