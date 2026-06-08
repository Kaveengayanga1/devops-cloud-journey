# Day 31: Azure App Service (Python on Linux) Deployment with Troubleshooting

## Task Overview

The Nautilus DevOps task was to deploy a Python-based Azure Web App with specific requirements.

### Requirements
1. **Web App Name:** `datacenter-webapp`
2. **Region:** `West US`
3. **Resource Group:** Initially asked as `default` (but actual working RG was lab-assigned)
4. **Publish Option:** `Code`
5. **Runtime Stack:** `Python` on `Linux`
6. **App Service Plan Name:** `datacenter-learn-python`
7. **SKU:** `Basic B1`
8. **Application Insights:** Disabled
9. **Tags:**
   - `Name=WebAppLearning`
   - `Environment=Dev`
10. **Final State:** Web App must be `Running`

---

## Lab Credentials and Access Details (Expired)

These credentials are included for complete historical record.

- **Portal URL:** https://portal.azure.com  
- **Username:** `kk_lab_user_main-2ee71e05627b4c2b@azurefreekmlprod.onmicrosoft.com`  
- **Password:** `Ysan686D`  
- **Start Time:** `Wed Apr 22 03:26:55 UTC 2026`  
- **End Time:** `Wed Apr 22 04:26:55 UTC 2026`

Other identity details captured from Azure error logs:

- **Subscription ID:** `f0c3bcdd-5ce2-4fa0-8cf3-41559747512b`
- **Client ID:** `1b23d4e6-42af-454b-a5c4-cf1219b749e4`
- **Object ID:** `0fa64c8e-7c5f-4bea-b26b-6029050950d3`

---

## Raw Execution Output Log

### 1) Check CLI defaults
```bash
az configure --list-defaults
```
Output:
```json
[]
```

### 2) Check available resource groups
```bash
az group list
```
Output:
```json
[
  {
    "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-2ee71e05627b4c2b",
    "location": "eastus",
    "managedBy": null,
    "name": "kml_rg_main-2ee71e05627b4c2b",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": null,
    "type": "Microsoft.Resources/resourceGroups"
  }
]
```

### 3) First attempt to create App Service Plan in `default` RG (Failed)
```bash
az appservice plan create \
    --name datacenter-learn-python \
    --resource-group default \
    --location "West US" \
    --sku B1 \
    --is-linux
```
Output:
```text
Readonly attribute name will be ignored in class <class 'azure.mgmt.web.v2023_01_01.models._models_py3.AppServicePlan'>
(AuthorizationFailed) The client '1b23d4e6-42af-454b-a5c4-cf1219b749e4' with object id '0fa64c8e-7c5f-4bea-b26b-6029050950d3' does not have authorization to perform action 'Microsoft.Web/serverfarms/write' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/default/providers/Microsoft.Web/serverfarms/datacenter-learn-python' or the scope is invalid. If access was recently granted, please refresh your credentials.
Code: AuthorizationFailed
Message: The client '1b23d4e6-42af-454b-a5c4-cf1219b749e4' with object id '0fa64c8e-7c5f-4bea-b26b-6029050950d3' does not have authorization to perform action 'Microsoft.Web/serverfarms/write' over scope '/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/default/providers/Microsoft.Web/serverfarms/datacenter-learn-python' or the scope is invalid. If access was recently granted, please refresh your credentials.
```

### 4) Corrected App Service Plan creation in actual allowed RG (Success)
```bash
az appservice plan create \
    --name datacenter-learn-python \
    --resource-group kml_rg_main-2ee71e05627b4c2b \
    --location "West US" \
    --sku B1 \
    --is-linux
```
Output:
```json
{
  "elasticScaleEnabled": false,
  "extendedLocation": null,
  "freeOfferExpirationTime": "2026-05-22T03:32:20.063333",
  "geoRegion": "West US",
  "hostingEnvironmentProfile": null,
  "hyperV": false,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-2ee71e05627b4c2b/providers/Microsoft.Web/serverfarms/datacenter-learn-python",
  "isSpot": false,
  "isXenon": false,
  "kind": "linux",
  "kubeEnvironmentProfile": null,
  "location": "westus",
  "maximumElasticWorkerCount": 1,
  "maximumNumberOfWorkers": 0,
  "name": "datacenter-learn-python",
  "numberOfSites": 0,
  "numberOfWorkers": 1,
  "perSiteScaling": false,
  "provisioningState": "Succeeded",
  "reserved": true,
  "resourceGroup": "kml_rg_main-2ee71e05627b4c2b",
  "sku": {
    "capabilities": null,
    "capacity": 1,
    "family": "B",
    "locations": null,
    "name": "B1",
    "size": "B1",
    "skuCapacity": null,
    "tier": "Basic"
  },
  "spotExpirationTime": null,
  "status": "Ready",
  "subscription": "f0c3bcdd-5ce2-4fa0-8cf3-41559747512b",
  "tags": null,
  "targetWorkerCount": 0,
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null,
  "zoneRedundant": false
}
```

### 5) Create Web App (Python/Linux) with tags (Success)
```bash
az webapp create \
    --name datacenter-webapp \
    --resource-group kml_rg_main-2ee71e05627b4c2b \
    --plan datacenter-learn-python \
    --runtime "PYTHON|3.9" \
    --tags Name=WebAppLearning Environment=Dev
```
Output:
```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "clientCertExclusionPaths": null,
  "clientCertMode": "Required",
  "cloningInfo": null,
  "containerSize": 0,
  "customDomainVerificationId": "55941048058256668DBB931AC313770F86D86BD6CE772FCC8D28B21AA32741C1",
  "dailyMemoryTimeQuota": 0,
  "daprConfig": null,
  "defaultHostName": "datacenter-webapp.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "datacenter-webapp.azurewebsites.net",
    "datacenter-webapp.scm.azurewebsites.net"
  ],
  "extendedLocation": null,
  "ftpPublishingUrl": "ftps://waws-prod-bay-271.ftp.azurewebsites.windows.net/site/wwwroot",
  "hostNameSslStates": [
    {
      "certificateResourceId": null,
      "hostType": "Standard",
      "ipBasedSslResult": null,
      "ipBasedSslState": "NotConfigured",
      "name": "datacenter-webapp.azurewebsites.net",
      "sslState": "Disabled",
      "thumbprint": null,
      "toUpdate": null,
      "toUpdateIpBasedSsl": null,
      "virtualIPv6": null,
      "virtualIp": null
    },
    {
      "certificateResourceId": null,
      "hostType": "Repository",
      "ipBasedSslResult": null,
      "ipBasedSslState": "NotConfigured",
      "name": "datacenter-webapp.scm.azurewebsites.net",
      "sslState": "Disabled",
      "thumbprint": null,
      "toUpdate": null,
      "toUpdateIpBasedSsl": null,
      "virtualIPv6": null,
      "virtualIp": null
    }
  ],
  "hostNames": [
    "datacenter-webapp.azurewebsites.net"
  ],
  "hostNamesDisabled": false,
  "hostingEnvironmentProfile": null,
  "httpsOnly": false,
  "hyperV": false,
  "id": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-2ee71e05627b4c2b/providers/Microsoft.Web/sites/datacenter-webapp",
  "identity": null,
  "inProgressOperationId": null,
  "isDefaultContainer": null,
  "isXenon": false,
  "keyVaultReferenceIdentity": "SystemAssigned",
  "kind": "app,linux",
  "lastModifiedTimeUtc": "2026-04-22T03:33:27.733333",
  "location": "West US",
  "managedEnvironmentId": null,
  "maxNumberOfWorkers": null,
  "name": "datacenter-webapp",
  "outboundIpAddresses": "13.91.71.221,13.87.130.15,13.87.133.42,13.64.111.216,13.64.106.162,13.64.108.222,40.78.84.34,13.93.139.194,40.85.156.188,40.78.81.155,13.64.50.238,13.91.66.242,40.112.243.111",
  "possibleOutboundIpAddresses": "13.91.71.221,13.87.130.15,13.87.133.42,13.64.111.216,13.64.106.162,13.64.108.222,40.78.84.34,13.93.139.194,40.85.156.188,40.78.81.155,13.64.50.238,13.91.66.242,13.91.64.158,13.88.172.155,13.88.168.174,13.88.174.151,13.88.155.42,13.88.184.96,13.88.153.146,13.88.152.43,13.88.169.69,13.64.227.206,13.64.212.122,13.64.229.154,13.64.104.77,13.64.104.212,13.91.83.46,13.88.154.116,13.64.108.230,13.64.141.52,40.112.243.111",
  "publicNetworkAccess": null,
  "redundancyMode": "None",
  "repositorySiteName": "datacenter-webapp",
  "reserved": true,
  "resourceConfig": null,
  "resourceGroup": "kml_rg_main-2ee71e05627b4c2b",
  "scmSiteAlsoStopped": false,
  "serverFarmId": "/subscriptions/f0c3bcdd-5ce2-4fa0-8cf3-41559747512b/resourceGroups/kml_rg_main-2ee71e05627b4c2b/providers/Microsoft.Web/serverfarms/datacenter-learn-python",
  "siteConfig": {
    "acrUseManagedIdentityCreds": false,
    "acrUserManagedIdentityId": null,
    "alwaysOn": false,
    "antivirusScanEnabled": null,
    "apiDefinition": null,
    "apiManagementConfig": null,
    "appCommandLine": null,
    "appSettings": null,
    "autoHealEnabled": null,
    "autoHealRules": null,
    "autoSwapSlotName": null,
    "azureMonitorLogCategories": null,
    "azureStorageAccounts": null,
    "clusteringEnabled": false,
    "connectionStrings": null,
    "cors": null,
    "customAppPoolIdentityAdminState": null,
    "customAppPoolIdentityTenantState": null,
    "defaultDocuments": null,
    "detailedErrorLoggingEnabled": null,
    "documentRoot": null,
    "elasticWebAppScaleLimit": 0,
    "experiments": null,
    "fileChangeAuditEnabled": null,
    "ftpsState": null,
    "functionAppScaleLimit": null,
    "functionsRuntimeScaleMonitoringEnabled": null,
    "handlerMappings": null,
    "healthCheckPath": null,
    "http20Enabled": false,
    "http20ProxyFlag": null,
    "httpLoggingEnabled": null,
    "ipSecurityRestrictions": [
      {
        "action": "Allow",
        "description": "Allow all access",
        "headers": null,
        "ipAddress": "Any",
        "name": "Allow all",
        "priority": 2147483647,
        "subnetMask": null,
        "subnetTrafficTag": null,
        "tag": null,
        "vnetSubnetResourceId": null,
        "vnetTrafficTag": null
      }
    ],
    "ipSecurityRestrictionsDefaultAction": null,
    "javaContainer": null,
    "javaContainerVersion": null,
    "javaVersion": null,
    "keyVaultReferenceIdentity": null,
    "limits": null,
    "linuxFxVersion": "",
    "loadBalancing": null,
    "localMySqlEnabled": null,
    "logsDirectorySizeLimit": null,
    "machineKey": null,
    "managedPipelineMode": null,
    "managedServiceIdentityId": null,
    "metadata": null,
    "minTlsCipherSuite": null,
    "minTlsVersion": null,
    "minimumElasticInstanceCount": 0,
    "netFrameworkVersion": null,
    "nodeVersion": null,
    "numberOfWorkers": 1,
    "phpVersion": null,
    "powerShellVersion": null,
    "preWarmedInstanceCount": null,
    "publicNetworkAccess": null,
    "publishingPassword": null,
    "publishingUsername": null,
    "push": null,
    "pythonVersion": null,
    "remoteDebuggingEnabled": null,
    "remoteDebuggingVersion": null,
    "requestTracingEnabled": null,
    "requestTracingExpirationTime": null,
    "routingRules": null,
    "runtimeADUser": null,
    "runtimeADUserPassword": null,
    "sandboxType": null,
    "scmIpSecurityRestrictions": [
      {
        "action": "Allow",
        "description": "Allow all access",
        "headers": null,
        "ipAddress": "Any",
        "name": "Allow all",
        "priority": 2147483647,
        "subnetMask": null,
        "subnetTrafficTag": null,
        "tag": null,
        "vnetSubnetResourceId": null,
        "vnetTrafficTag": null
      }
    ],
    "scmIpSecurityRestrictionsDefaultAction": null,
    "scmIpSecurityRestrictionsUseMain": null,
    "scmMinTlsCipherSuite": null,
    "scmMinTlsVersion": null,
    "scmSupportedTlsCipherSuites": null,
    "scmType": null,
    "sitePort": null,
    "sitePrivateLinkHostEnabled": null,
    "storageType": null,
    "supportedTlsCipherSuites": null,
    "tracingOptions": null,
    "use32BitWorkerProcess": null,
    "virtualApplications": null,
    "vnetName": null,
    "vnetPrivatePortsCount": null,
    "vnetRouteAllEnabled": null,
    "webJobsEnabled": false,
    "webSocketsEnabled": null,
    "websiteTimeZone": null,
    "winAuthAdminState": null,
    "winAuthTenantState": null,
    "windowsConfiguredStacks": null,
    "windowsFxVersion": null,
    "xManagedServiceIdentityId": null
  },
  "slotSwapStatus": null,
  "state": "Running",
  "storageAccountRequired": false,
  "suspendedTill": null,
  "tags": {
    "Environment": "Dev",
    "Name": "WebAppLearning"
  },
  "targetSwapSlot": null,
  "trafficManagerHostNames": null,
  "type": "Microsoft.Web/sites",
  "usageState": "Normal",
  "virtualNetworkSubnetId": null,
  "vnetContentShareEnabled": false,
  "vnetImagePullEnabled": false,
  "vnetRouteAllEnabled": false,
  "workloadProfileName": null
}
```

### 6) Final state verification
```bash
az webapp show \
    --name datacenter-webapp \
    --resource-group kml_rg_main-2ee71e05627b4c2b \
    --query state \
    --output tsv
```
Output:
```text
Running
```

---

## Core Theories

### 1. What Azure App Service is
Azure App Service is a Platform as a Service (PaaS) that lets you deploy web applications without managing OS patching, web server setup, or VM lifecycle.

### 2. App Service Plan vs Web App
- **App Service Plan** defines compute, pricing tier, region, and OS.
- **Web App** is the application instance that runs inside that plan.

### 3. Publish Option: `Code` (What it means)
`Publish: Code` means you deploy source code directly (for example Python files), and Azure provides the runtime environment.

- You provide app code.
- Azure provides and manages runtime host.
- Faster for standard web app deployments.

Compared with `Container` publish mode:
- You provide a Docker image.
- You manage packaging and image lifecycle.

### 4. Runtime Stack: Python on Linux
Using `--runtime "PYTHON|3.9"` with Linux creates a Linux-based Python web app. This aligns with the requirement for Python runtime and Linux OS.

### 5. Application Insights behavior
For this CLI flow, Application Insights was not enabled during creation, which satisfies the task requirement to keep it disabled.

### 6. Tags and governance
Tags like `Name` and `Environment` are important for cost tracking, filtering, and resource governance.

---

## Mistakes Made, Why They Happened, and How They Were Fixed

### Mistake 1: Using wrong resource group (`default`)
- **What happened:** `az appservice plan create` failed with `AuthorizationFailed` when targeting `default`.
- **Why it happened:** The task statement said default resource group, but this lab identity had permission only on the assigned group `kml_rg_main-2ee71e05627b4c2b`.
- **How it was fixed:** Switched `--resource-group` to `kml_rg_main-2ee71e05627b4c2b` and retried. Creation succeeded.
- **Prevention:** Always run `az group list` and verify the permitted RG before creating resources in lab environments.

### Mistake 2: Misinterpreting warning as failure risk
- **What happened:** CLI printed: `Readonly attribute name will be ignored...`
- **Why it happened:** Azure SDK warning text appears in some CLI operations and can look like an error.
- **How it was fixed:** Verified actual result by checking JSON output fields such as `provisioningState: Succeeded` and `status: Ready`.
- **Prevention:** Distinguish warning vs fatal error by checking command exit behavior and provisioning state.

### Mistake 3: Potential confusion around `Publish: Code`
- **What happened:** Requirement wording caused confusion between Code and Container publishing models.
- **Why it happened:** In Azure Portal, Publish is an explicit dropdown; in CLI, it is implied by command type and runtime arguments.
- **How it was fixed:** Used `az webapp create` with runtime `PYTHON|3.9` and no container image options, which maps to Publish as Code.
- **Prevention:** In CLI, treat runtime-based app creation as Code publish unless a container image is specified.

---

## Professional Azure CLI Workflow (Recommended)

Use this checklist for similar tasks:

1. Login and verify subscription context.
2. Check default settings with `az configure --list-defaults`.
3. List resource groups and identify authorized RG.
4. Create App Service Plan with correct region, SKU, and `--is-linux`.
5. Create Web App with correct runtime and required tags.
6. Validate state using `az webapp show --query state`.
7. Optionally validate host reachability using the app URL.

---

## Final Validation Matrix

| Requirement | Expected | Actual | Status |
| :--- | :--- | :--- | :--- |
| Web App Name | `datacenter-webapp` | `datacenter-webapp` | ✅ |
| Plan Name | `datacenter-learn-python` | `datacenter-learn-python` | ✅ |
| Region | `West US` | `West US` | ✅ |
| SKU | `Basic B1` | `B1 / Basic` | ✅ |
| Runtime | Python on Linux | `PYTHON|3.9`, `kind: app,linux` | ✅ |
| Publish Option | Code | Runtime-based code deployment | ✅ |
| Tags | Name + Environment | Both present | ✅ |
| App Insights | Disabled | Not enabled in this flow | ✅ |
| App State | Running | `Running` | ✅ |

---

## Key Takeaway

The task was completed successfully after correcting the resource group scope. The biggest operational lesson is to validate authorization boundaries first in lab subscriptions, even when the task statement names a default resource group.