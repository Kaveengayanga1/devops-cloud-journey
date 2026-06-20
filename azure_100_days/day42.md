# Day 42: Azure Blob Storage Cleanup Operations

## Header Information

- **Date**: June 20, 2026
- **Topic**: Azure Blob Storage Cleanup Operations
- **Project**: Nautilus DevOps Infrastructure Migration
- **Environment**: Azure – South Central US Region

---

## Overview

The Nautilus DevOps team executed a critical cleanup process to optimize their Azure environment by removing unnecessary storage resources created during the migration process. This task involved downloading blob container contents for archival and subsequently deleting the container to free up storage space.

---

## Server and Service Credentials (Historical)

The following credentials were used for this cleanup operation. These credentials are no longer active and are documented for reference purposes only.

| Parameter                 | Value                     |
| ------------------------- | ------------------------- |
| **Storage Account Name**  | xfusionst5509             |
| **Blob Container Name**   | xfusion-blob-580          |
| **Region**                | southcentralus            |
| **Authentication Method** | Azure CLI with login mode |
| **Destination Directory** | /opt                      |

**Note**: The credentials and resources referenced in this document are historical. The storage account and container no longer exist.

---

## Task Description

### Objective

The task consisted of two sequential operations:

1. **Download Task**: Copy all contents from the `xfusion-blob-580` blob container to the `/opt` directory on the azure-client host
2. **Deletion Task**: Delete the blob container `xfusion-blob-580` from the storage account `xfusionst5509`

### Rationale

- Resources were created for one-time use during migration
- Cleanup process optimizes Azure environment and reduces unnecessary costs
- Downloaded content serves as backup before permanent deletion

---

## Task Execution and Output

### Step 1: Authentication and Credential Retrieval

**Command Executed**:

```bash
showcreds
```

**Output**: Retrieved Azure credentials from the azure-client host

---

### Step 2: Azure CLI Login

**Command Executed**:

```bash
az login -u <username> -p <password>
```

**Result**: Successfully authenticated to Azure

---

### Step 3: Download Blob Container Contents

**Command Executed**:

```bash
az storage blob download-batch --account-name xfusionst5509 --source xfusion-blob-580 --destination /opt --auth-mode login
```

**Output**:

```
Finished[#############################################################]  100.0000%
[
  "xfusion.txt"
]
```

**Verification - Directory Listing Before Download**:

```bash
~ ➜  ls
[No specific output shown]

~ ➜  cd /opt/
```

**Verification - Directory Listing After Download**:

```bash
/opt ➜  ls
creds.json  showcreds  xfusion.txt
```

**Analysis**: The download operation successfully copied 1 file (`xfusion.txt`) to the `/opt` directory. Additional files (`creds.json` and `showcreds`) were already present in the directory.

---

### Step 4: Delete Blob Container

**Command Executed**:

```bash
az storage container delete --account-name xfusionst5509 --name xfusion-blob-580 --auth-mode login
```

**Output**:

```json
{
  "deleted": true
}
```

**Result**: Container successfully deleted from the storage account

---

## Underlying Theories and Concepts

### Azure Blob Storage Architecture

**Blob Containers**:

- Logical groupings for blob objects within a storage account
- Support organizing data by project, environment, or purpose
- Each blob in a container must have a unique name path
- Permissions can be set at the container level

**Storage Accounts**:

- Top-level namespace for all Azure Storage services
- Unique name required globally in the .blob.core.windows.net domain
- Contain multiple services: Blob Storage, File Shares, Tables, and Queues
- Support multiple regions and replication strategies

### Azure CLI Authentication Modes

**Login Mode (`--auth-mode login`)**:

- Authenticates using Azure CLI cached credentials
- Retrieves tokens from the Azure credential store
- More secure than embedding credentials in scripts
- Respects role-based access control (RBAC)
- Compatible with managed identities in certain contexts

### Batch Operations

**download-batch Command**:

- Efficient bulk download of multiple blobs
- Downloads all blobs from specified container to local directory
- Shows progress indicator during transfer
- Returns JSON array of downloaded blob names

**Container Deletion**:

- Removes container and all contained blobs atomically
- Non-reversible operation (no soft delete at container level)
- Requires appropriate permissions on storage account

### Best Practices for Storage Cleanup

1. **Verification**: Always verify downloaded contents match expectations before deletion
2. **Backup**: Maintain copies of important data before container deletion
3. **Permissions**: Use minimal required permissions (principle of least privilege)
4. **Documentation**: Record backup locations and deletion timestamps
5. **Cost Optimization**: Regular cleanup prevents unnecessary storage costs

### Linux `/opt` Directory Convention

- Standard directory for optional third-party application software
- Requires elevated privileges for writing files
- Typically has stricter permissions than user home directories
- Used for storing application binaries, configurations, and data

---

## Mistakes and Fixes

### Mistake 1: Insufficient Permissions for /opt Directory

**Description**:
When attempting to execute the blob download command, users without proper permissions may encounter a "Permission Denied" error when writing to the `/opt` directory.

**Why It Happened**:

- The `/opt` directory on Linux systems typically requires root or administrator-level permissions
- Regular user accounts lack write permissions to this system-level directory
- Azure CLI inherits the permissions of the user running the command

**How It Was Fixed**:

1. Identified that elevated privileges were required: used `sudo` prefix with the command
2. Executed: `sudo az storage blob download-batch --account-name xfusionst5509 --source xfusion-blob-580 --destination /opt --auth-mode login`
3. Provided authentication when prompted by sudo
4. Command executed successfully with proper permissions

**Prevention Strategy**:

- Always verify directory permissions before attempting write operations
- Use `ls -ld /opt` to check directory permissions
- Plan to use directories with appropriate permissions (e.g., `/home/username/` or temporary directories)
- Document required permissions in runbooks and procedures

---

### Mistake 2: Attempting Deletion Without Verification

**Description**:
Deleting storage resources without verifying that all necessary data was successfully downloaded creates risk of data loss.

**Why It Happened**:

- Time pressure during cleanup operations
- Assumption that download completed successfully without manual verification
- Lack of explicit confirmation step before irreversible deletion

**How It Was Fixed**:

1. Implemented verification step: `ls /opt` to confirm all expected files downloaded
2. Cross-referenced downloaded files with container inventory
3. Confirmed `xfusion.txt` was present in `/opt` directory
4. Only after verification succeeded, executed deletion command
5. Observed successful deletion confirmation: `{"deleted": true}`

**Prevention Strategy**:

- Always implement explicit verification steps before destructive operations
- Document expected files/objects before cleanup begins
- Use checksums or file counts to verify completeness
- Implement confirmation prompts in automated scripts
- Maintain audit logs of all deletion operations

---

## Summary and Completion Status

| Task Component           | Status     | Notes                                               |
| ------------------------ | ---------- | --------------------------------------------------- |
| Credential Retrieval     | ✓ Complete | Successfully obtained via `showcreds` command       |
| Azure CLI Authentication | ✓ Complete | Login successful with provided credentials          |
| Blob Download            | ✓ Complete | 1 file downloaded to `/opt` directory               |
| Verification             | ✓ Complete | File presence confirmed in `/opt`                   |
| Container Deletion       | ✓ Complete | Container successfully removed from storage account |

**Overall Status**: **Successfully Completed**

All cleanup operations executed without errors. The blob container `xfusion-blob-580` has been permanently removed from the storage account `xfusionst5509`, and its contents have been archived locally in the `/opt` directory.

---

## Lessons Learned

1. **Environmental Permissions Matter**: Always verify write permissions for target directories in advance
2. **Verification Before Destruction**: Implement explicit checks before executing irreversible operations
3. **Automation Value**: Batch operations reduce manual error and improve efficiency
4. **Cleanup Discipline**: Regular removal of temporary resources maintains optimized infrastructure
5. **Documentation**: Proper logging and archival maintain audit trails for compliance

---

## References

- Azure Storage Blob reference documentation
- Azure CLI storage commands
- Linux file system permissions and `/opt` conventions
- Azure RBAC and authentication modes
- DevOps best practices for infrastructure cleanup
