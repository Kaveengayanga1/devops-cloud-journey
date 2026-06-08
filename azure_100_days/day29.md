# Day 29: Azure Container Registry (ACR) and Docker Integration

## Task Overview
The Nautilus DevOps team needed to set up a containerized application environment. This involved creating an Azure Container Registry (ACR), building a Docker image from a local source on the `azure-client` host, and pushing that image to the newly created registry.

### Requirements:
1.  **Registry Name:** `datacenteracr9043`
2.  **Region:** `eastus`
3.  **SKU:** `Basic`
4.  **Source Code:** Located at `/root/pyapp` on the `azure-client` host.
5.  **Image Tag:** `latest` (Full path: `datacenteracr9043.azurecr.io/datacenteracr9043:latest`)

---

## Step-by-Step Implementation

### 1. Azure Authentication
Ensure you are logged into the Azure CLI.
```bash
az login -u <username> -p <password>
```

### 2. Resource Group Identification
List the available resource groups to find the correct one for deployment.
```bash
az group list
```
**Output Context:**
*   **Resource Group Found:** `kml_rg_main-0c0c748338aa43a1`
*   **Subscription ID:** `f0c3bcdd-5ce2-4fa0-8cf3-41559747512b`

### 3. Create Azure Container Registry (ACR)
```bash
az acr create --resource-group kml_rg_main-0c0c748338aa43a1 --name datacenteracr9043 --sku Basic --location eastus
```

### 4. Authenticate Docker with ACR
```bash
az acr login --name datacenteracr9043
```
*Status: Login Succeeded*

### 5. Build the Docker Image
Navigate to the application directory and build the image.
```bash
cd /root/pyapp
docker build -t datacenteracr9043:latest .
```

### 6. Tag and Push the Image
Tag the image with the ACR login server address and push it.
```bash
# Tagging
docker tag datacenteracr9043:latest datacenteracr9043.azurecr.io/datacenteracr9043:latest

# Pushing
docker push datacenteracr9043.azurecr.io/datacenteracr9043:latest
```

### 7. Verification
Verify the image exists in the repository.
```bash
az acr repository list --name datacenteracr9043 --output table
az acr repository show-tags --name datacenteracr9043 --repository datacenteracr9043 --output table
```

---

## Mistakes Made & Lessons Learned

### Mistake 1: Registry Name Confusion
*   **What happened:** An initial attempt was made to create a registry with the name `devopsacr29817`.
*   **Why it happened:** Lack of strict adherence to the specific naming requirement (`datacenteracr9043`) provided in the task description.
*   **How it was fixed:** A second `az acr create` command was run with the correct name `datacenteracr9043`.

### Mistake 2: Missing Fully Qualified Name for Push
*   **Theory Note:** Docker cannot push to ACR simply using the local tag `datacenteracr9043:latest`.
*   **Why it happens:** ACR requires the registry's login server URI (`<registry-name>.azurecr.io`) as part of the image tag to route the push correctly.
*   **How it was fixed:** Used `docker tag` to create a pointer from the local image to the fully qualified ACR path before running `docker push`.

---

## Technical Theories

### Azure Container Registry (ACR) SKUs
*   **Basic:** Optimized for cost. Small storage (5GB) and standard throughput. Ideal for dev/test environments.
*   **Standard/Premium:** Offer higher storage, geo-replication, and private links for production workloads.

### Build Context
The `.` at the end of `docker build -t ... .` specifies the **Build Context**. Docker sends all files in this directory to the Docker daemon. If the directory is massive, it can slow down builds, which is why `.dockerignore` is important.

### ACR Login Server
The login server for any Azure Container Registry follows the pattern `registryname.azurecr.io`. This is the endpoint used for all `docker login`, `docker tag`, and `docker push/pull` operations.
