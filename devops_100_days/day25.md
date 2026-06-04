# Day 25: Git Branching, Merging, and Remote Push

## Task Overview

The Nautilus application development team has a project repository located at `/opt/games.git`. This repository is cloned at `/usr/src/kodekloudrepos` on the storage server in the Stratos DC. The task is to create a new branch named `datacenter`, add a specific file, merge it back to `master`, and push both branches to the origin.

### Infrastructure Details

| Server Name    | Hostname    | User      | Password     | Purpose                          |
| :------------- | :---------- | :-------- | :----------- | :------------------------------- |
| Storage Server | `ststor01`  | `natasha` | `Bl@kW`      | Stores data for Nautilus Servers |
| Jump Host      | `jump-host` | `thor`    | `mjolnir123` | Provides secure access           |

## Theories & Concepts

1.  **Git Branching**: Allows you to diverge from the main line of development and continue to do work without messing with that main line.
2.  **Staging and Committing**: `git add` moves changes to the staging area, and `git commit` records those changes in the repository history.
3.  **Merging**: The process of joining two or more development histories together (e.g., merging `datacenter` into `master`).
4.  **Remote Push**: Sending local repository commits to a remote server (`origin`) so other team members can access them.

## Step-by-Step Implementation

### 1. Access the Storage Server

Connect via SSH from the jump host.

```bash
ssh natasha@ststor01
# Password: Bl@kW
```

### 2. Switch to Root and Navigate to Repository

```bash
sudo su
cd /usr/src/kodekloudrepos/games
```

### 3. Create and Switch to a New Branch

```bash
git checkout -b datacenter
```

### 4. Copy the Required File and Commit

```bash
cp /tmp/index.html .
git add .
git commit -m "Add index.html to datacenter branch"
```

### 5. Merge the New Branch into Master

```bash
git checkout master
git merge datacenter
```

### 6. Push Changes to Origin

```bash
git push origin master
git push origin datacenter
```

## Mistakes Made & Troubleshooting

1.  **Missing Destination in `cp` Command**:
    - **Mistake**: `cp /tmp/index.html`
    - **Why it happened**: Forgot to specify the destination directory (current directory `.`).
    - **Fix**: Used `cp /tmp/index.html .`

2.  **Typo in Git Command**:
    - **Mistake**: `gid add .`
    - **Why it happened**: Rapid typing lead to "gid" instead of "git".
    - **Fix**: Re-entered the correct command `git add .`.

3.  **Direct Push without Merging (Hypothetical Risk)**:
    - **Observation**: Always ensure you checkout `master` before merging. Forgetting to switch back would result in merging into the wrong branch.

## Final State Verification

Executing `ls` in the `master` branch shows `index.html`, `info.txt`, and `welcome.txt`. Both local branches are synced with `origin`.
