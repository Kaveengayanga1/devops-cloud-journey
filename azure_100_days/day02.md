# Creating an Azure Virtual Machine with Azure CLI

This task can be accomplished efficiently using the Azure Command-Line Interface (CLI). Below are the steps and the specific command to create the VM according to your requirements.

## **Prerequisites**

Ensure you are logged into Azure (`az login`) and have identified the name of your **existing resource group**.

## **Step 1: Create the Virtual Machine**

Run the following command. **Note:** Replace `<Existing_Resource_Group>` with the actual name of your resource group.

```bash
az vm create \
  --resource-group <Existing_Resource_Group> \
  --name xfusion-vm \
  --location westus \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --nsg-rule ssh
```

**Breakdown of the flags used:**

* `--image Ubuntu2204`: Selects the Ubuntu 22.04 LTS image.
* `--size Standard_B1s`: Sets the specific VM size required.
* `--storage-sku Standard_LRS`: specifices "Standard HDD" (Locally Redundant Storage).
* `--os-disk-size-gb 30`: Sets the OS disk size to 30 GB.
* `--nsg-rule ssh`: Automatically configures the Network Security Group to allow inbound traffic on port 22.
* `--generate-ssh-keys`: Creates SSH keys in `~/.ssh` if they don't already exist, allowing passwordless login.

---

## **Step 2: Verify and Connect**

Once the command completes, you will see a JSON output containing the `publicIpAddress`. You can also retrieve the IP explicitly with this command:

```bash
az vm show \
  --resource-group <Existing_Resource_Group> \
  --name xfusion-vm \
  --show-details \
  --query publicIps \
  --output tsv
```

**To SSH into the machine:**
Use the Public IP you just retrieved:

```bash
ssh azureuser@<Public_IP_Address>
```

To list all resource groups in your subscription, use the following Azure CLI command:

```bash
az group list --output table
```

## **Command Options**

* **`az group list`**: This is the base command. By default, it returns a long JSON list.
* **`--output table`**: This formats the output into a clean, readable table (recommended for quick scanning).
* **`--output tsv`**: Useful if you need to pipe the output into another script or command.
