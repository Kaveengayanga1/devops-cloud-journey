# Day 03: Create a User with a Non-Interactive Shell

## Theory
To accommodate the backup agent tool's specifications, the system admin team at xFusionCorp Industries requires the creation of a user with a non-interactive shell.

**Task:** Create a user named `rose` with a non-interactive shell on App Server 1.

## Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Application Server 1 | `stapp01` | `tony` | `Ir0nM@n` | Hosts Nautilus Application 1 |
| Jump Host Server | `jump-host` | `thor` | `mjolnir123` | Provides secure access to Stork DC |

## Step-by-Step Guide

### Step 1: Connect to App Server 1
First, SSH into `stapp01` using the credentials provided for App Server 1.

```bash
ssh tony@stapp01
```

When prompted for the password, enter: `Ir0nM@n`

### Step 2: Create the User with Non-Interactive Shell
Creating a user requires root permissions. Use `sudo` to execute the command. To create a user with a non-interactive shell, we specify `/sbin/nologin` (or `/bin/false`) as the shell using the `-s` flag.

```bash
sudo useradd -s /sbin/nologin rose
```

You may be prompted to enter Tony's password again (`Ir0nM@n`) for the `sudo` command.

### Step 3: Verify the Configuration
Check the `/etc/passwd` file to confirm the user was created with the correct shell.

```bash
grep rose /etc/passwd
```

You should see output similar to `rose:x:1001:1001::/home/rose:/sbin/nologin`.

### Step 4: Exit
Cleanly terminate your SSH session to return to the Jump Host.

```bash
exit
```

## Terminal Output
```
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.195.20)' can't be established.
ED25519 key fingerprint is SHA256:ji/1qthcQ0SZPszrbw5B1fGUP9Y7aI49Cvyaal8WUuo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 
[tony@stapp01 ~]$ sudo useradd -s /sbin/nologin rose

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
[tony@stapp01 ~]$ grep rose /etc/passwd
rose:x:1001:1001::/home/rose:/sbin/nologin
[tony@stapp01 ~]$ exit
logout
Connection to stapp01 closed.
thor@jump-host ~$ 
```
