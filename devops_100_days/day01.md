# Day 1: Create User with Non-Interactive Shell

## Task Description
The system admin team at xFusionCorp Industries requires the creation of a user with a non-interactive shell to accommodate the backup agent tool's specifications.

**Objective:** Create a user named `javed` with a non-interactive shell on App Server 2.

---

## Infrastructure Details

| Server Name | IP | Hostname | User | Password | Purpose |
|------------|------------|--------------------------------|---------|----------|---------------------|
| stapp01 | 172.16.238.10 | stapp01.stratos.xfusioncorp.com | tony | Ir0nM@n | Nautilus App 1 |
| **stapp02** | **172.16.238.11** | **stapp02.stratos.xfusioncorp.com** | **steve** | **Am3ric@** | **Nautilus App 2** |
| stapp03 | 172.16.238.12 | stapp03.stratos.xfusioncorp.com | banner | BigGr33n | Nautilus App 3 |
| stlb01 | 172.16.238.14 | stlb01.stratos.xfusioncorp.com | loki | Mischi3f | Nautilus HTTP LB |
| stdb01 | 172.16.239.10 | stdb01.stratos.xfusioncorp.com | peter | Sp!dy | Nautilus DB Server |
| ststor01 | 172.16.238.15 | ststor01.stratos.xfusioncorp.com | natasha | Bl@kW | Nautilus Storage Server |
| stbkp01 | 172.16.238.16 | stbkp01.stratos.xfusioncorp.com | clint | H@wk3y3 | Nautilus Backup Server |
| stmail01 | 172.16.238.17 | stmail01.stratos.xfusioncorp.com | groot | Gr00T123 | Nautilus Mail Server |
| jump_host | Dynamic | jump_host.stratos.xfusioncorp.com | thor | mjolnir123 | Jump Server |
| jenkins | 172.16.238.19 | jenkins.stratos.xfusioncorp.com | jenkins | j@rv!s | Jenkins Server for CI/CD |

---

## Solution Steps

### Step 1: Connect to App Server 2
From the jump host, SSH into App Server 2 using the correct credentials:

```bash
ssh steve@stapp02
```

**Password:** `Am3ric@`

**Expected Output:**
```bash
thor@jumphost ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:HLU7RtC350VmlsQ4CxKo4tcLT9LlC95qS+D/qhfDdn0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
[steve@stapp02 ~]$
```

### Step 2: Create the User with Non-Interactive Shell
Run the following command to create the user `javed` with `/sbin/nologin` as the shell:

```bash
sudo useradd -s /sbin/nologin javed
```

**Password:** `Am3ric@` (when prompted for sudo password)

**Expected Output:**
```bash
[steve@stapp02 ~]$ sudo useradd -s /sbin/nologin javed

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve:
[steve@stapp02 ~]$
```

### Step 3: Verify User Creation
Confirm the user was created successfully with the correct shell:

**Method 1: Using `id` command**
```bash
id javed
```

**Expected Output:**
```bash
[steve@stapp02 ~]$ id javed
uid=1002(javed) gid=1002(javed) groups=1002(javed)
```

**Method 2: Using `grep` on /etc/passwd**
```bash
grep javed /etc/passwd
```

**Expected Output:**
```bash
javed:x:1002:1002::/home/javed:/sbin/nologin
```

---

## Key Learnings

1. **Non-Interactive Shell:** The `/sbin/nologin` shell prevents users from logging in interactively, which is useful for service accounts and backup agents.

2. **Server-Specific Credentials:** Each application server has different user credentials. Always verify the correct server and credentials from the infrastructure table.

3. **useradd Command Options:**
   - `-s` or `--shell`: Specifies the login shell for the user
   - Common non-interactive shells: `/sbin/nologin` or `/bin/false`

4. **Verification Methods:** Always verify user creation using `id` command or by checking `/etc/passwd` file.

---

## Common Mistakes to Avoid

- ❌ Using wrong server credentials (e.g., using `tony` instead of `steve` for stapp02)
- ❌ Attempting to run commands from the jump host instead of the target server
- ❌ Forgetting to specify the `-s` flag to set the shell
- ❌ Not verifying the user creation after running the command

---

## Status
✅ **Completed Successfully**
