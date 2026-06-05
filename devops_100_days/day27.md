# Day 27: Git Revert - Reverting Changes Safely

**Date:** May 15, 2026
**Topic:** Reverting latest commit in a Git repository
**Server:** Storage Server (`ststor01`)
**Location:** Stratos DC

## Task Overview

The Nautilus application development team reported an issue with the recent commits pushed to the `ecommerce` repository. The goal is to revert the repo's `HEAD` to the previous commit while maintaining a clean history.

## Infrastructure Details

| Server Name     | Hostname    | User      | Password     | Purpose                |
| --------------- | ----------- | --------- | ------------ | ---------------------- |
| Jump Host       | `jump-host` | `thor`    | `mjolnir123` | Secure access          |
| Storage Server  | `ststor01`  | `natasha` | `Bl@kW`      | Data storage/Git repos |
| App Server 1    | `stapp01`   | `tony`    | `Ir0nM@n`    | Application 1          |
| App Server 2    | `stapp02`   | `steve`   | `Am3ric@`    | Application 2          |
| App Server 3    | `stapp03`   | `banner`  | `BigGr33n`   | Application 3          |
| Database Server | `stdb01`    | `peter`   | `Sp!dy`      | Database               |
| Backup Server   | `stbkp01`   | `clint`   | `H@wk3y3`    | Backups                |
| Mail Server     | `stmail01`  | `groot`   | `Gr00T123`   | Emails                 |
| LoadBalancer    | `stlb01`    | `loki`    | `Mischi3f`   | Traffic                |
| Jenkins Server  | `jenkins`   | `jenkins` | `j@rv!s`     | CI/CD                  |

---

## Technical Theories

### 1. What is Git Revert?

`git revert` is a command used to undo changes by creating a **new commit** that is the inverse of the targeted commit. Unlike `reset`, it does not delete history.

### 2. Git Revert vs. Git Reset

- **Git Reset:** Moves the branch pointer backward, effectively "deleting" commits. Dangerous for shared repositories as it desyncs other developers' histories.
- **Git Revert:** Best for public/shared branches. It adds a "fix" commit, showing exactly what was undone and why.

### 3. Understanding HEAD

`HEAD` is a pointer to the last commit in your current active branch.

---

## Mistakes Made & Fixes

### Mistake 1: Incorrect Hostname

**Issue:** Attempted to SSH using `ststor` instead of `ststor01`.

- **Error:** `ssh: Could not resolve hostname ststor: Name or service not known`
- **Why:** The hostname was incomplete or incorrect in the local DNS/hosts file.
- **Fix:** Used the correct hostname `ststor01`.

### Mistake 2: Dubious Ownership

**Issue:** Git refused to operate on the repository due to security checks implemented in newer Git versions.

- **Error:** `fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/ecommerce'`
- **Why:** The folder belongs to a different user/group than the one executing the command (natasha vs root/other).
- **Fix:** Switched to the root user using `sudo su` to gain the necessary permissions to manage the repository.

---

## Step-by-Step Execution

### Step 1: Access the Storage Server

```bash
ssh natasha@ststor01
# Password: Bl@kW
```

### Step 2: Navigate to Repo and Handle Permissions

```bash
cd /usr/src/kodekloudrepos/ecommerce
sudo su
# Switched to root to avoid "dubious ownership" errors
```

### Step 3: Check Commit History

```bash
git log --oneline
# Identified HEAD at 8f40e48
```

### Step 4: Revert the Latest Commit

The requirement was to use a specific message: `revert ecommerce`.

```bash
git revert HEAD --no-commit
git commit -m "revert ecommerce"
```

### Step 5: Verify the Result

```bash
git log --oneline
# Verified that a new commit 09839d4 was created on top of the history.
```

---

**Status:** Completed Successfully
