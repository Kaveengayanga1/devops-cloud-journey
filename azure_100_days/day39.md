Day 39: Azure Storage Static Website Hosting
Date: 2026-06-08
Topic: Enable Azure Storage static website hosting and upload the index page
Project: Azure 100 Days — storage static website example
Version: Azure CLI / Azure Storage static website configuration

## Server Credentials (Historical / Example Values)

| Item                     | Value                                                                       |
| ------------------------ | --------------------------------------------------------------------------- |
| Storage account endpoint | https://datacenterwebst14415.z13.web.core.windows.net/ (historical/example) |
| Storage account name     | datacenterwebst14415 (historical/example)                                   |
| Username                 | admin (historical/example; not active)                                      |
| Password                 | P@ssw0rd! (historical/example; not active)                                  |
| SSH port                 | 22 (historical/example)                                                     |
| API key                  | API_KEY_EXAMPLE_XXXXXXXXXXXXXXXX                                            |
| Database password        | DB_PASS_EXAMPLE                                                             |

> Note: The credentials above are historical/example values for documentation purposes only. They are not active and should not be used for access.

## Task Output

```bash
~ ➜  ls
index.html

~ ➜  az storage blob service-properties update \
    --account-name datacenterwebst14415 \
    --static-website true \
    --index-document index.html \
    --auth-mode login
{
  "cors": [],
  "deleteRetentionPolicy": {
    "allowPermanentDelete": null,
    "days": 7,
    "enabled": true
  },
  "hourMetrics": {
    "enabled": false,
    "includeApis": null,
    "retentionPolicy": {
      "allowPermanentDelete": null,
      "days": null,
      "enabled": false
    },
    "version": "1.0"
  },
  "logging": {
    "delete": false,
    "read": false,
    "retentionPolicy": {
      "allowPermanentDelete": null,
      "days": null,
      "enabled": false
    },
    "version": "1.0",
    "write": false
  },
  "minuteMetrics": {
    "enabled": false,
    "includeApis": null,
    "retentionPolicy": {
      "allowPermanentDelete": null,
      "days": null,
      "enabled": false
    },
    "version": "1.0"
  },
  "staticWebsite": {
    "defaultIndexDocumentPath": null,
    "enabled": true,
    "errorDocument_404Path": null,
    "indexDocument": "index.html"
  },
  "target_version": null
}

~ ➜  az storage blob upload \
    --account-name datacenterwebst14415 \
    --container-name '$web' \
    --file /index.html \
    --name index.html \
    --auth-mode login
[Errno 2] No such file or directory: '/index.html'
Please check the file path.

~ ✖ az storage blob upload     --account-name datacenterwebst14415     --container-name '$web'     --file /root/i
ndex.html     --name index.html     --auth-mode login
Finished[#############################################################]  100.0000%
{
  "client_request_id": "715a2200-6343-11f1-8eb0-aa36afe49264",
  "content_md5": "ipEexLOfOlV+W2sG+4JAYQ==",
  "date": "2026-06-08T14:07:54+00:00",
  "encryption_key_sha256": null,
  "encryption_scope": null,
  "etag": "\"0x8DEC567562DAC21\"",
  "lastModified": "2026-06-08T14:07:55+00:00",
  "request_id": "c0db9cb1-301e-0028-3150-f7e434000000",
  "request_server_encrypted": true,
  "version": "2022-11-02",
  "version_id": null
}

~ ➜  az storage account show \
    --name datacenterwebst14415 \
    --query "primaryEndpoints.web" \
    --output tsv
https://datacenterwebst14415.z13.web.core.windows.net/
```

## Theories

### Azure Storage Static Website Hosting

- Azure Storage can host static web content directly from a special blob container named `$web`.
- When static website hosting is enabled, Azure exposes a web endpoint such as `https://<storage-account>.z13.web.core.windows.net/`.
- The `indexDocument` setting defines the default file served when a user accesses the root of the website. In this task, that file was `index.html`.

### `$web` Container Behavior

- `$web` is a special-purpose container created for static website hosting.
- It must be referenced with quotes in the shell because the `$` character otherwise triggers variable expansion.
- Files uploaded to `$web` are publicly served through the static website endpoint rather than through the standard blob endpoint.
- Server-side code does not run in Azure Storage static website hosting; only static assets such as HTML, CSS, JavaScript, and images are served.

### Azure CLI and Authentication

- `az storage blob service-properties update` enables the static website feature and configures the index document.
- `az storage blob upload` copies the local file into the `$web` container.
- `--auth-mode login` uses the authenticated Azure CLI session instead of storage account keys.
- `az storage account show --query "primaryEndpoints.web" --output tsv` retrieves the static website URL for verification.

### Operational Considerations

- The file path used for upload must exist on the local machine.
- The storage account name must be globally unique within Azure.
- Public static hosting should be used only for content intended for anonymous access.

## Mistakes and Fixes

### Mistake 1: Incorrect file path used during upload

- What happened: The upload command used `--file /index.html`, and Azure CLI returned `[Errno 2] No such file or directory: '/index.html'`.
- Why it happened: The file was not present at the absolute root path `/index.html`. The intended file was located in the working directory or another valid local path.
- How it was fixed:
  1. Verified the file location with `ls`.
  2. Re-ran the upload using the correct path, `/root/index.html`.
  3. Confirmed success from the `Finished` progress indicator and the returned blob metadata JSON.

### Mistake 2: Static website configuration had to be enabled before hosting content

- What happened: The website content could only be uploaded after the storage account static website feature was enabled.
- Why it happened: The `$web` container is created by Azure when static website hosting is enabled, not during normal storage account creation.
- How it was fixed:
  1. Enabled static website hosting with `az storage blob service-properties update`.
  2. Confirmed that `staticWebsite.enabled` was `true` and that `indexDocument` was set to `index.html`.
  3. Uploaded the content to the `$web` container after the feature was active.

### Mistake 3: The `$web` container name required shell-safe quoting

- What happened: The container name was passed as `$web` in the CLI command.
- Why it happened: Shells interpret `$` as the start of a variable reference unless the value is quoted.
- How it was fixed:
  1. Wrapped the container name in single quotes: `--container-name '$web'`.
  2. Re-ran the upload command.
  3. Verified that the upload completed successfully.

## Recommended Procedure

1. Confirm the presence of the local `index.html` file.
2. Enable static website hosting on the Azure Storage account.
3. Upload `index.html` into the quoted `$web` container.
4. Query the `primaryEndpoints.web` property to confirm the public URL.
5. Open the endpoint in a browser and verify that the page renders correctly.
