# Day 24: Git Branching and Troubleshooting Permissions

## Task Overview

The Nautilus developers need a new branch named `xfusioncorp_demo` created from the `master` branch in the repository located at `/usr/src/kodekloudrepos/demo` on the **Storage Server** (`ststor01`).

## Infrastructure Details

| Server Name    | Hostname  | User    | Password   | Purpose       |
| -------------- | --------- | ------- | ---------- | ------------- |
| Jump Host      | jump-host | thor    | mjolnir123 | Secure Access |
| Storage Server | ststor01  | natasha | Bl@kW      | Data Storage  |

## Real-world Terminal Output & Process

```bash
# 1. Connect to Storage Server from Jump Host
thor@jumphost ~$ ssh natasha@ststor01
natasha@ststor01's password:

# 2. Navigate to the repository
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/demo

# 3. First hurdle: Dubious ownership error
[natasha@ststor01 demo]$ git branch
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/demo'
# Fix: Add directory to safe.directory
[natasha@ststor01 demo]$ git config --global --add safe.directory /usr/src/kodekloudrepos/demo

# 4. Check current branches
[natasha@ststor01 demo]$ git branch
* kodekloud_demo
  master

# 5. Second hurdle: Permission Denied when switching to master
[natasha@ststor01 demo]$ git checkout master
fatal: Unable to create '/usr/src/kodekloudrepos/demo/.git/index.lock': Permission denied
# Fix: Use sudo to perform git operations due to root ownership of the repo
[natasha@ststor01 demo]$ sudo git checkout master
Switched to branch 'master'

# 6. Create the new branch from master
[natasha@ststor01 demo]$ sudo git branch xfusioncorp_demo

# 7. Verify branch creation
[natasha@ststor01 demo]$ git branch
  kodekloud_demo
* master
  xfusioncorp_demo

# 8. Switch to the new branch (requires sudo)
[natasha@ststor01 demo]$ sudo git checkout xfusioncorp_demo
Switched to branch 'xfusioncorp_demo'
```

## Mistakes Made & Solutions

### 1. Dubious Ownership Error

- **Problem:** Git flagged the repository as having "dubious ownership" because the current user (`natasha`) didn't own the directory created by another user (likely `root`).
- **Why it happened:** Security features in newer Git versions prevent interacting with repositories owned by others to avoid execution of malicious code.
- **Fix:** Ran `git config --global --add safe.directory /usr/src/kodekloudrepos/demo` to tell Git this specific path is trusted.

### 2. Permission Denied (`index.lock`)

- **Problem:** Attempting to `git checkout` or `git branch` failed with "Permission denied".
- **Why it happened:** The `.git` directory and its files were owned by `root`, preventing the `natasha` user from creating the necessary lock files to modify the repository state.
- **Fix:** Used `sudo` for Git commands that modify the repository infrastructure (like creating branches or switching them).

## Core Theories

- **Git Branching:** A way to diverge from the main line of development and continue work without affecting the main line (master).
- **Safe Directory:** A Git configuration to bypass ownership checks on specific local paths.
- **Lock Files:** Files like `index.lock` used by Git to ensure only one process modifies the repo at a time.
