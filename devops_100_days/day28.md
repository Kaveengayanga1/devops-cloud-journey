# Day 28: Git Cherry-pick Task

**Date:** May 20, 2026
**Topic:** Merging Specific Commits using Git Cherry-pick
**Server:** Storage Server (`ststor01`)

---

## Task Description

The Nautilus application development team has been working on a project repository `/opt/beta.git`. This repo is cloned at `/usr/src/kodekloudrepos` on the storage server in Stratos DC.

**Requirements:**

- There are two branches: `master` and `feature`.
- A developer is working on the `feature` branch.
- We need to merge a specific commit from the `feature` branch into the `master` branch.
- The commit message to be merged is "Update info.txt".
- Changes must be pushed to the remote repository.

---

## Infrastructure Details

| Server Name          | Hostname    | User      | Password     | Purpose                                     |
| :------------------- | :---------- | :-------- | :----------- | :------------------------------------------ |
| Application Server 1 | `stapp01`   | `tony`    | `Ir0nM@n`    | Hosts Nautilus Application 1                |
| Application Server 2 | `stapp02`   | `steve`   | `Am3ric@`    | Hosts Nautilus Application 2                |
| Application Server 3 | `stapp03`   | `banner`  | `BigGr33n`   | Hosts Nautilus Application 3                |
| LoadBalancer Server  | `stlb01`    | `loki`    | `Mischi3f`   | Distributes traffic for Nautilus HTTP       |
| Database Server      | `stdb01`    | `peter`   | `Sp!dy`      | Hosts Nautilus Database                     |
| Storage Server       | `ststor01`  | `natasha` | `Bl@kW`      | Stores data for Nautilus Servers            |
| Backup Server        | `stbkp01`   | `clint`   | `H@wk3y3`    | Manages backups for Nautilus Servers        |
| Mail Server          | `stmail01`  | `groot`   | `Gr00T123`   | Manages email services for Nautilus Servers |
| Jump Host Server     | `jump-host` | `thor`    | `mjolnir123` | Provides secure access to Stratos DC        |
| Jenkins Server       | `jenkins`   | `jenkins` | `j@rv!s`     | Runs Jenkins for CI/CD pipeline             |

---

## Mistakes Made and Fixes

### 1. Executing Git Commands outside the Repository

- **Mistake:** Running `git log feature --oneline` inside `/usr/src/kodekloudrepos`.
- **Error:** `fatal: not a git repository (or any of the parent directories): .git`
- **Why it happened:** The user was in the parent directory of the cloned repository, not inside the `beta` folder where the `.git` directory resides.
- **Fix:** Changed directory to the actual repo using `cd beta/`.

### 2. Redundant Directory Navigation

- **Mistake:** Attempting to `cd beta/` while already inside the `beta` directory.
- **Error:** `bash: cd: beta/: No such file or directory`
- **Why it happened:** The user forgot they had already entered the repository folder.
- **Fix:** Stay in the current directory and proceed with git commands.

---

## Step-by-Step Execution Output

```bash
# Connect to Storage Server
thor@jumphost ~$ ssh natasha@ststor01
natasha@ststor01's password: Bl@kW

# Navigate to the repository location
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos
[natasha@ststor01 kodekloudrepos]$ ls
beta

# Switch to root for permissions (optional depending on setup)
[natasha@ststor01 kodekloudrepos]$ sudo su

# Attempting log in wrong directory (Mistake 1)
[root@ststor01 kodekloudrepos]# git log feature --oneline
fatal: not a git repository (or any of the parent directories): .git

# Moving into the repo
[root@ststor01 kodekloudrepos]# cd beta/

# Accidental redundant cd (Mistake 2)
[root@ststor01 beta]# cd beta/
bash: cd: beta/: No such file or directory

# Identify the commit hash for "Update info.txt"
[root@ststor01 beta]# git log feature --oneline
6f20c62 (HEAD -> feature, origin/feature) Update welcome.txt
9f6959a Update info.txt
91f82cd (origin/master, master) Add welcome.txt
05ed068 initial commit

# Switch to master branch
[root@ststor01 beta]# git checkout master
Switched to branch 'master'

# Cherry-pick the required commit (9f6959a)
[root@ststor01 beta]# git cherry-pick 9f6959a
[master d7e2fb8] Update info.txt
 Date: Wed May 20 13:53:27 2026 +0000
 1 file changed, 1 insertion(+), 1 deletion(-)

# Push changes to the remote origin
[root@ststor01 beta]# git push origin master
To /opt/beta.git
   91f82cd..d7e2fb8  master -> master

# Verify status
[root@ststor01 beta]# git status
On branch master
Your branch is up to date with 'origin/master'.
nothing to commit, working tree clean
```

---

## Theory

### What is Git Cherry-pick?

Git Cherry-pick allows you to pick a single commit from one branch and apply it to another. This is useful when you don't want to merge an entire branch but only need a specific fix or feature that exists in a particular commit.

### What is a Commit Hash?

Every commit in Git is identified by a unique SHA-1 hash (a string of characters). To cherry-pick, you must identify this hash using `git log` so Git knows exactly which changes to copy.
