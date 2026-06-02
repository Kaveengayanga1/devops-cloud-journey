# Day 22: Clone an Empty Git Repository on the Storage Server

**Date:** May 1, 2026  
**Task:** Clone the bare repository `/opt/blog.git` to `/usr/src/kodekloudrepos` using the `natasha` user on the Storage Server

## Infrastructure Details

| Server Name | IP Address | Hostname | User | Password | Purpose |
|---|---|---|---|---|---|
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | Dynamic | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| Application Server 3 | Dynamic | stapp03 | banner | BigGr33n | Hosts Nautilus Application 3 |
| LoadBalancer Server | Dynamic | stlb01 | loki | Mischi3f | Distributes traffic for Nautilus HTTP |
| Database Server | Dynamic | stdb01 | peter | Sp!dy | Hosts Nautilus Database |
| Storage Server | Dynamic | ststor01 | natasha | Bl@kW | Stores data for Nautilus Servers |
| Backup Server | Dynamic | stbkp01 | clint | H@wk3y3 | Manages backups for Nautilus Servers |
| Mail Server | Dynamic | stmail01 | groot | Gr00T123 | Manages email services for Nautilus Servers |
| Jump Host Server | Dynamic | jumphost / jump-host | thor | mjolnir123 | Provides secure access to Stork DC |
| Jenkins Server | Dynamic | jenkins | jenkins | j@rv!s | Runs Jenkins for CI/CD pipeline |

**Note:** These credentials were part of the original lab environment and are preserved here for documentation because they are no longer active.

## Task Requirements

- Use the `natasha` user on the Storage Server.
- Clone the repository from `/opt/blog.git`.
- Place the clone under `/usr/src/kodekloudrepos`.
- Do not change permissions or make unauthorized directory modifications.

## Theory and Concepts

### 1. Git Repository
Git stores source code history, references, and metadata so changes can be tracked, shared, and recovered.

### 2. Bare Repository
A bare repository contains Git data only. It does not have a checked-out working tree, so it is commonly used as a central repository for pushes and clones.

### 3. `git clone`
`git clone` creates a local copy of a remote repository, copies the remote reference, and initializes the destination directory automatically.

### 4. Empty Repository Behavior
If the source bare repository has no commits, the clone will still succeed, but commands such as `git log` will report that the current branch has no commits yet.

### 5. SSH Access and Lab Hostnames
Lab environments often expose the same host through slightly different naming conventions, such as `jumphost` and `jump-host`. The actual target should be followed exactly as shown in the session or lab instructions.

### 6. Linux Directory Structure and Ownership
The task target path `/usr/src/kodekloudrepos` is under the standard source-code storage area. Since the task explicitly says not to alter permissions, the repository should be cloned using the existing ownership and access model.

## Step-by-Step Execution

### Step 1: Connect to the Storage Server

```bash
ssh natasha@ststor01
```

Connection prompt shown during login:

```text
The authenticity of host 'ststor01 (10.244.49.46)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
```

### Step 2: Verify the Source Directory

```bash
ls -la /usr/src/
```

Observed output:

```text
total 44
drwxr-xr-x  1 root    root    4096 May  1 02:27 .
drwxr-xr-x  1 root    root    4096 Mar  2 04:13 ..
drwxr-xr-x  2 root    root    4096 Jun 25  2024 debug
drwxr-xr-x  1 root    root    4096 May  1 00:42 kernels
drwxr-xr-x  2 natasha natasha 4096 May  1 02:27 kodekloudrepos
drwxr-xr-x  2 root    root    4096 Mar  5 14:49 linux-gcp-6.8-headers-6.8.0-1026
drwxr-xr-x  2 root    root    4096 Mar  5 14:49 linux-headers-6.8.0-1026-gcp
drwxr-xr-x  7 root    root    4096 Feb  6 05:22 linux-headers-6.8.0-94-generic
drwxr-xr-x 26 root    root    4096 Feb  6 05:22 linux-hwe-6.8-headers-6.8.0-94
```

### Step 3: Move to the Target Directory

```bash
cd /usr/src/kodekloudrepos
ls
```

The directory was empty before cloning.

### Step 4: Clone the Repository

```bash
git clone /opt/blog.git
```

Observed output:

```text
Cloning into 'blog'...
warning: You appear to have cloned an empty repository.
done.
```

### Step 5: Confirm the Clone Result

```bash
ls -la /usr/src/kodekloudrepos/
```

Observed output:

```text
total 16
drwxr-xr-x 3 natasha natasha 4096 May  1 02:33 .
drwxr-xr-x 1 root    root    4096 May  1 02:27 ..
drwxr-xr-x 3 natasha natasha 4096 May  1 02:33 blog
```

### Step 6: Enter the Repository and Inspect It

```bash
cd /usr/src/kodekloudrepos/blog
git log --oneline
git status
git remote -v
ls -la /usr/src/kodekloudrepos/blog/
```

Observed output:

```text
fatal: your current branch 'master' does not have any commits yet

On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)

origin  /opt/blog.git (fetch)
origin  /opt/blog.git (push)

total 12
drwxr-xr-x 3 natasha natasha 4096 May  1 02:33 .
drwxr-xr-x 3 natasha natasha 4096 May  1 02:33 ..
drwxr-xr-x 6 natasha natasha 4096 May  1 02:34 .git
```

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. `git log` Failed Because the Repository Had No Commits
The command `git log --oneline` returned a fatal message stating that the current branch had no commits yet.

Why it happened:
- The source repository `/opt/blog.git` was empty.
- An empty bare repository can be cloned successfully, but it does not contain any commit history.

How it was fixed:
- The situation was validated with `git status` and `git remote -v` instead of expecting commit history.
- No repository modification was needed because the empty state was the correct lab condition.

### 2. Host Naming Was Slightly Inconsistent
The lab text used both `jumphost` and `jump-host` for the jump server.

Why it happened:
- This was a documentation inconsistency in the lab material.
- It was not a system failure.

How it was fixed:
- The connection flow followed the working lab context and the actual server used in the session.

### 3. No Permission Changes Were Made
The task required that no unauthorized changes be made.

Why it mattered:
- Cloning should happen with the existing directory ownership and permissions.
- Changing ownership or permissions could break the lab requirement.

How it was handled:
- No `chmod`, `chown`, or manual file edits were performed.
- The clone was left exactly as Git created it.

## Professional DevOps Execution Notes

- Always verify the target directory before cloning.
- Treat an empty bare repository as a valid source if the task says to clone it as-is.
- Use `git status` and `git remote -v` when history is absent.
- Keep the task scope minimal: clone only, do not modify content or permissions.

## Validation Checklist

- SSH login to `ststor01` as `natasha` succeeded.
- `/usr/src/kodekloudrepos` existed before cloning.
- `git clone /opt/blog.git` created the `blog` directory.
- `git remote -v` showed `/opt/blog.git` as the origin.
- `git status` showed a clean repository with no commits yet.
- No unauthorized permission or ownership changes were made.

## Final Outcome

The bare repository `/opt/blog.git` was successfully cloned into `/usr/src/kodekloudrepos/blog` on the Storage Server using the `natasha` user. The repository is empty, which explains the `git log` message, but the clone and remote configuration are correct.