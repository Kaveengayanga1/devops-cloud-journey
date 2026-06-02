# Day 21: Create a Bare Git Repository on the Storage Server

**Date:** April 16, 2026  
**Task:** Install Git on the Storage Server and create the bare repository `/opt/ecommerce.git`

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

- Use `yum` to install `git` on the Storage Server.
- Create a bare repository named `/opt/ecommerce.git`.

## Theory and Concepts

### 1. Git and Version Control
Git is a distributed version control system used to track changes in source code, coordinate work across teams, and maintain a history of commits.

### 2. Bare Repository
A bare repository contains only Git metadata and object data. It does not have a working tree, so there are no editable project files checked out in that directory.

### 3. Why Bare Repositories Are Used
Bare repositories are usually used as central repositories on servers. Developers clone from them and push code back to them, while the server itself does not need a local working copy.

### 4. Yum Package Manager
`yum` is the package manager used on RHEL-based systems to install, remove, and update software packages.

### 5. SSH and Jump Host Access
SSH is used to connect securely to remote systems. In this task, access is performed through the jump host before connecting to the Storage Server.

### 6. Ownership and Permissions
For team use, the repository may need ownership and permissions adjusted later. A bare repository created with `sudo` will typically be owned by `root` unless ownership is changed deliberately.

## Step-by-Step Execution

### Step 1: Connect to the Storage Server

```bash
ssh natasha@ststor01
```

Prompt shown during connection:

```text
The authenticity of host 'ststor01 (10.244.81.22)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
```

### Step 2: Install Git

```bash
sudo yum install git -y
```

Key result:

```text
Installed:
  git-2.52.0-1.el9.x86_64                git-core-2.52.0-1.el9.x86_64         git-core-doc-2.52.0-1.el9.noarch
  less-590-6.el9.x86_64                  perl-Error-1:0.17029-7.el9.noarch    perl-Git-2.52.0-1.el9.noarch
  perl-TermReadKey-2.38-11.el9.x86_64    perl-lib-0.65-483.el9.x86_64

Complete!
```

### Step 3: Create the Bare Repository

```bash
sudo git init --bare /opt/ecommerce.git
```

Key result:

```text
Initialized empty Git repository in /opt/ecommerce.git/
```

### Step 4: Verify the Repository Directory

```bash
ls -ld /opt/ecommerce.git
```

Output:

```text
drwxr-xr-x 6 root root 4096 Apr 16 14:06 /opt/ecommerce.git
```

### Step 5: Exit the Sessions

```bash
exit
exit
```

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. Repository Ownership Was Root-Owned
The repository was created with `sudo`, so the resulting ownership was `root:root`.

Why it happened:
- `sudo` runs the command with administrative privileges.
- Git created the repository as `root` because the command was executed by `root` through privilege escalation.

How it was fixed:
- The repository creation itself succeeded because that was the task requirement.
- If developers need write access later, the ownership can be changed with a command such as `sudo chown -R natasha:natasha /opt/ecommerce.git` or by assigning the correct shared group.

### 2. Hostname Naming Can Be Confusing
The lab material refers to the jump host in two slightly different forms: `jumphost` in the prompt and `jump-host` in the infrastructure table.

Why it happened:
- The environment documentation uses inconsistent naming.
- This is a common lab-documentation mismatch rather than a runtime failure.

How it was fixed:
- The actual SSH target used in the session was followed exactly as shown in the working output: `ssh natasha@ststor01` from the jump host.

## Bare Repository Theory: More Detail

### What It Is
A bare repository is a Git repository that stores version history without a working directory. It contains the repository database, branches, tags, refs, and objects, but no checked-out files.

### Typical Applications
- Central Git servers for teams
- Remote push targets for developers
- Git hosting platforms such as GitHub, GitLab, and Bitbucket internally use repository storage patterns similar to this model
- Deployment automation and CI/CD pipelines
- Server-side Git hooks for automated tasks after pushes

### Advantages
- No working tree conflicts on the server
- Smaller footprint than a full cloned repository with files checked out
- Better suited for central storage and collaboration
- Clean separation between code storage and code editing
- Easy to use as a source for automated deployments

### Limitations
- You cannot directly edit files inside a bare repository
- It is not meant for normal development work on the server itself
- Proper permissions must be managed carefully if multiple users will push to it

## Validation Checklist

- Git package installed successfully.
- Bare repository created at `/opt/ecommerce.git`.
- Repository path verified with `ls -ld`.
- Session exited cleanly.

## Final Outcome

The Storage Server now has Git installed and a bare repository created at `/opt/ecommerce.git`. This repository is ready to serve as a central Git remote for development workflows.