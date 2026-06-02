# Day 19: Azure Blob Storage Security - Managing Container Access Levels

## 1. Theory: Azure Blob Storage Access (Azure Blob Storage ප්‍රවේශ න්‍යාය)

Understanding how to secure data in Azure is critical. Azure Storage creates a hierarchy:
1.  **Storage Account:** The top-level namespace (e.g., `nautilusst5247`).
2.  **Container:** Logic folders that hold blobs (e.g., `nautilus-container-21086`).
3.  **Blob:** The actual data files.

### Public Access Levels
*   **Container (Full public read access):** Anonymous users can list blobs and read content. (High Risk)
*   **Blob (Public read access for blobs only):** Anonymous users can read content if they have the exact URL, but cannot list blobs.
*   **Private (No public read access):** Default and most secure. Requires authentication via Microsoft Entra ID (login) or Access Keys.

---

## 2. Professional DevOps Criteria (වෘත්තීය DevOps නිර්ණායක)

A professional Cloud/DevOps Engineer follows these strict protocols when managing storage:

1.  **Verification First (තහවුරු කරගැනීම):**
    *   Never assume the state of a resource. Always query (`show`) before modifying.
    *   *Context:* We checked `nautilus-priv-1494` to ensure it was already private before focusing on the target.

2.  **Least Privilege Principle (අවම වරප්‍රසාද මූලධර්මය):**
    *   Storage containers should always be **Private** by default unless there is a specific requirement for public hosting.

3.  **Error Handling & Auth Modes (දෝෂ හඳුනාගැනීම සහ auth-mode):**
    *   CLI commands often support different authentication modes: `--auth-mode login` (RBAC/Entra ID) vs `--auth-mode key` (Access Keys).
    *   *Lesson:* If `login` fails with "Allowed values: key", switch to using the Storage Account Key method immediately.

4.  **CLI over GUI:**
    *   Using Azure CLI ensures the process is repeatable and scriptable, unlike clicking in the Azure Portal.

---

## 3. The Task: Secure the Nautilus Container

**Scenario:**
The Nautilus DevOps team identified a security risk. The container `nautilus-container-21086` is currently **Public**. It contains sensitive data and must be restricted to **Private**. The other container `nautilus-priv-1494` helps serve as a control—it is already private.

---

## 4. Execution Steps (ක්‍රියාත්මක කිරීම)

### Step 1: Verify Current Status (වත්මන් තත්වය පරීක්ෂා කිරීම)
Check the public access level of the target container.

```bash
az storage container show \
  --name nautilus-container-21086 \
  --account-name nautilusst5247 \
  --query "properties.publicAccess" \
  --auth-mode login \
  --output tsv
```
**Output:** `container` (This confirms it is currently Public).

### Step 2: The Remediation (ගැටළුව විසඳීම)
We need to remove public access.

**Initial Attempt (Failed due to Auth Mode):**
```bash
az storage container set-permission \
  --name nautilus-container-21086 \
  --account-name nautilusst5247 \
  --public-access off \
  --auth-mode login
```
*Error:* `az storage container set-permission: 'login' is not a valid value for '--auth-mode'. Allowed values: key.`

**Corrected Command (Using Key Auth):**
We switch to key-based authentication (default if `--auth-mode` is omitted, or explicit with `key`).

```bash
az storage container set-permission \
  --name nautilus-container-21086 \
  --account-name nautilusst5247 \
  --public-access off \
  --auth-mode key
```
*Result:* Command executes successfully. Public access is revoked.

### Step 3: Final Verification (අවසාන තහවුරු කිරීම)
Ensure the container returns no public access level (null).

```bash
az storage container show \
  --name nautilus-container-21086 \
  --account-name nautilusst5247 \
  --query "properties.publicAccess" \
  --auth-mode login \
  --output tsv
```
**Output:** *(Empty/Blank)* -> This confirms the container is now **Private**.
