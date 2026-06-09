# Day 32: Git Rebase - Feature Branch Integration with Master Branch

**Date:** May 27, 2026  
**Topic:** Git Rebase Operations and Feature Branch Synchronization  
**Environment:** Stratos Data Center - Nautilus Application Project  
**Objective:** Rebase feature branch with master branch while preserving commits and avoiding merge commits  
**Status:** Completed Successfully

---

## Executive Summary

This documentation covers the successful execution of a Git rebase operation on the Nautilus application development project. A developer required synchronization of their feature branch with the latest changes in the master branch without creating merge commits or losing any committed work. The task was accomplished using Git rebase operations on the Storage Server (ststor01) in the Stratos Data Center.

---

## Infrastructure Details

| Server Name          | Hostname  | IP Address   | Username | Password   | Purpose                                     |
| -------------------- | --------- | ------------ | -------- | ---------- | ------------------------------------------- |
| Storage Server       | ststor01  | 10.244.81.28 | natasha  | Bl@kW      | Stores data for Nautilus Servers            |
| Jump Host            | jump-host | Dynamic      | thor     | mjolnir123 | Provides secure access to Stratos DC        |
| Application Server 1 | stapp01   | Dynamic      | tony     | Ir0nM@n    | Hosts Nautilus Application 1                |
| Application Server 2 | stapp02   | Dynamic      | steve    | Am3ric@    | Hosts Nautilus Application 2                |
| Application Server 3 | stapp03   | Dynamic      | banner   | BigGr33n   | Hosts Nautilus Application 3                |
| Load Balancer        | stlb01    | Dynamic      | loki     | Mischi3f   | Distributes traffic for Nautilus HTTP       |
| Database Server      | stdb01    | Dynamic      | peter    | Sp!dy      | Hosts Nautilus Database                     |
| Backup Server        | stbkp01   | Dynamic      | clint    | H@wk3y3    | Manages backups for Nautilus Servers        |
| Mail Server          | stmail01  | Dynamic      | groot    | Gr00T123   | Manages email services for Nautilus Servers |
| Jenkins Server       | jenkins   | Dynamic      | jenkins  | j@rv!s     | Runs Jenkins for CI/CD pipeline             |

---

## Task Requirements

The Nautilus application development team required the following objectives to be accomplished:

1. **Feature Branch Synchronization:** Synchronize the feature branch with the latest changes from the master branch
2. **No Merge Commits:** Avoid creating additional merge commits that would complicate the Git history
3. **Data Preservation:** Ensure all commits and changes in the feature branch are preserved
4. **Linear History:** Maintain a clean, linear commit history
5. **Remote Push:** Update the remote repository with the rebased changes

---

## Technical Background: Git Merge vs. Git Rebase

### Git Merge Approach

Git merge combines the feature branch with the master branch by creating a new merge commit that ties the two branches together. While this preserves the complete history including the point at which branches diverged, it can result in a more complex commit history, especially in projects with frequent branching and merging.

**Advantages:**

- Complete historical record of when branches merged
- Non-destructive operation on the feature branch

**Disadvantages:**

- Creates additional merge commits
- Results in a less linear history
- Makes it harder to understand the actual development flow

### Git Rebase Approach

Git rebase "replays" the feature branch commits on top of the master branch. This effectively "cuts" the feature branch from its original point and reattaches it to the tip of the master branch, creating a linear history.

**How Git Rebase Works:**

1. **Identify Common Ancestor:** Git finds the last common commit between the feature branch and master branch
2. **Temporary Storage:** Git saves all commits made in the feature branch (but not in master) to a temporary location
3. **Reset Feature Branch:** The feature branch is reset to point to the same commit as master
4. **Replay Commits:** The saved commits are reapplied one by one on top of the master branch commits
5. **Update References:** The feature branch reference is updated to point to the new tip

**Advantages:**

- Clean, linear commit history
- Easier to understand the sequence of changes
- No additional merge commits
- Preserves all original commits with their messages and authors

**Disadvantages:**

- Rewrites commit history (requires force push)
- Should not be used on public/shared branches without careful consideration

---

## Task Execution

### Session Output

```bash
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.81.28)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos
[natasha@ststor01 kodekloudrepos]$ sudo su
[root@ststor01 kodekloudrepos]# git branch -a
fatal: not a git repository (or any of the parent directories): .git
[root@ststor01 kodekloudrepos]# ls
media
[root@ststor01 kodekloudrepos]# cd media/
[root@ststor01 media]# ls
feature.txt  info.txt
[root@ststor01 media]# git branch -a
* feature
  master
  remotes/origin/feature
  remotes/origin/master
[root@ststor01 media]# git rebase master
Successfully rebased and updated refs/heads/feature.
[root@ststor01 media]# git push origin feature --force
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 296 bytes | 296.00 KiB/s, done.
Total 3 (delta 0), reused 0, pack-reused 0 (from 0)
To /opt/media.git
 + a486908...123581f feature -> feature (forced update)
[root@ststor01 media]# git status
On branch feature
nothing to commit, working tree clean
[root@ststor01 media]#
```

---

## Issues Encountered and Resolutions

### Issue 1: Incorrect Directory Navigation

**What Happened:**
When the user first executed `git branch -a` after changing to `/usr/src/kodekloudrepos`, the system returned an error: "fatal: not a git repository (or any of the parent directories): .git"

**Why It Happened:**
The cloned repository is not directly in the `/usr/src/kodekloudrepos` directory. Instead, it resides in a subdirectory called `media/`. The command was executed at the wrong location.

**How It Was Fixed:**
The user navigated to the correct directory by executing:

```bash
cd media/
```

The actual Git repository clone is located at `/usr/src/kodekloudrepos/media/`, which contains the `.git` directory and the tracked files. After changing to this directory, Git commands functioned correctly.

### Issue 2: Permission Requirements

**What Happened:**
To execute Git operations and particularly to perform force push operations, the user needed elevated privileges using `sudo su` to become the root user.

**Why It Happened:**
The operations required writing to the repository and interacting with remotes, which may require elevated permissions depending on the repository's ownership and access control settings.

**How It Was Fixed:**
The user executed `sudo su` to switch to the root user, which provided the necessary permissions to execute all Git operations without restrictions.

### Issue 3: Force Push Requirement

**What Happened:**
After the successful rebase operation, a standard `git push origin feature` would have failed because the local feature branch history had been rewritten.

**Why It Happened:**
Git rebase creates new commits for all the feature branch commits (even though the changes are identical). This changes the commit history, and Git prevents pushing branches with rewritten history by default for safety reasons.

**How It Was Fixed:**
The `--force` flag was used with the push command:

```bash
git push origin feature --force
```

The `--force` flag explicitly tells Git to overwrite the remote tracking branch with the local rewritten history. The output shows:

```
+ a486908...123581f feature -> feature (forced update)
```

The `+` symbol indicates a forced update where the commit hashes have changed.

---

## Step-by-Step Implementation Guide

### Step 1: Establish SSH Connection

Connect to the Storage Server from the Jump Host using SSH:

```bash
ssh natasha@ststor01
```

- **Username:** natasha
- **Password:** Bl@kW
- **Server IP:** 10.244.81.28
- **Purpose:** Access the storage server where the repository is cloned

### Step 2: Navigate to Repository Location

Change directory to the location where the repository is cloned:

```bash
cd /usr/src/kodekloudrepos
```

Verify the structure and identify the actual Git repository location:

```bash
ls
```

You should see a `media/` directory. Enter this directory:

```bash
cd media/
```

### Step 3: Elevate Privileges

Switch to the root user to ensure sufficient permissions:

```bash
sudo su
```

Enter the root user password if prompted.

### Step 4: Verify Current Branch Status

Check the current state of branches in the repository:

```bash
git branch -a
```

Expected output should show:

- Current branch: `* feature` (indicated by asterisk)
- Local branches: `feature` and `master`
- Remote branches: `remotes/origin/feature` and `remotes/origin/master`

### Step 5: Execute Rebase Operation

Perform the rebase operation to align the feature branch with the master branch:

```bash
git rebase master
```

Expected output:

```
Successfully rebased and updated refs/heads/feature.
```

**Important:** If conflicts occur during rebase:

1. Resolve the conflicting files manually
2. Stage the resolved files: `git add <filename>`
3. Continue the rebase: `git rebase --continue`
4. Repeat if necessary

### Step 6: Force Push Changes to Remote

Push the rebased feature branch to the remote repository with the `--force` flag:

```bash
git push origin feature --force
```

Expected output shows the forced update:

```
To /opt/media.git
 + a486908...123581f feature -> feature (forced update)
```

### Step 7: Verify Completion

Confirm that the rebase operation completed successfully:

```bash
git status
```

Expected output:

```
On branch feature
nothing to commit, working tree clean
```

This confirms:

- You are on the feature branch
- All rebased commits have been successfully applied
- There are no uncommitted changes
- The working directory is clean

---

## Technical Insights and Best Practices

### Commit Hashes and Rebase

When a rebase occurs, the commit hashes of the rebased commits change. In the output, you can see:

- Original feature branch tip: `a486908`
- New feature branch tip after rebase: `123581f`

Even though the actual file changes remain identical, the commit objects themselves are recreated with new hashes because they now have different parent commits.

### Force Push Safety Considerations

The `--force` flag is powerful but dangerous on shared branches. In this context:

- The feature branch is a developer's personal branch (not shared)
- The use of `--force` is appropriate
- On shared branches, consider using `--force-with-lease` which is safer

### Repository Structure

The repository `/opt/media.git` is a bare repository (indicated by the `.git` suffix), which is a standard practice for remote repositories. The clone at `/usr/src/kodekloudrepos/media/` is the working directory where developers make changes.

### Git Objects in Push Output

The push output provides useful information:

```
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 296 bytes | 296.00 KiB/s, done.
Total 3 (delta 0), reused 0 (from 0)
```

This shows:

- 4 total objects were examined
- 3 objects were written (new commits)
- 0 delta objects (objects that are stored as differences)
- The transfer was successful

---

## Verification Checklist

- ✅ Feature branch successfully rebased on top of master
- ✅ No merge commits created
- ✅ All feature branch commits preserved
- ✅ Linear Git history maintained
- ✅ Remote repository updated with new feature branch state
- ✅ Working directory is clean with no uncommitted changes
- ✅ Git status confirms successful completion

---

## Conclusion

The Git rebase operation was successfully completed, achieving all required objectives. The feature branch now contains all its original commits while being based on the latest master branch, resulting in a clean, linear Git history. The changes have been pushed to the remote repository, and the task is complete.

This approach provides several advantages:

- Cleaner commit history for future developers
- Easier to track the flow of development
- Avoids merge commit clutter
- Maintains code integrity with all changes preserved

The task demonstrates best practices in Git workflow management for feature branch development in a collaborative environment.
