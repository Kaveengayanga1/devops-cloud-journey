# Day 04: Create a User without a Home Directory

## Theory
To complete a specific task at xFusionCorp Industries, the system administration team needs to create a new user on App Server 2 without a home directory. This is often done for service accounts or users who do not need to store files on the server.

**Task:** Create a user named `mark` on App Server 2 without creating a home directory.

## Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Application Server 2 | `stapp02` | `steve` | `Am3ric@` | Hosts Nautilus Application 2 |
| Jump Host Server | `jump-host` | `thor` | `mjolnir123` | Provides secure access to Stork DC |

## Step-by-Step Guide

### Step 1: Access App Server 2
First, you need to log in to the server. From the Jump Host, use SSH to connect to `stapp02` using the credentials provided.

```bash
ssh steve@stapp02
```
Input password: `Am3ric@`

### Step 2: Create the User (No Home Directory)
Once logged into App Server 2, use the `useradd` command with the `-M` flag. This restriction is crucial as it creates the user but specifically instructs the system **not** to create the user's home directory.

```bash
sudo useradd -M mark
```

*   `sudo`: Execute as administrator.
*   `-M`: Prevents the creation of the home directory.
*   `mark`: The username to create.

### Step 3: Verify the Task
It is good practice to double-check that the user exists and that the home directory was skipped.

Check if the user exists:
```bash
id mark
```
This should return the UID and GID for `mark`.

Check for the home directory:
```bash
ls /home/mark
```
This should return an error "No such file or directory," confirming the requirement was met.

## Terminal Output

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.29.255)' can't be established.
ED25519 key fingerprint is SHA256:a3a/JVaiq7zpqXqBi+MQr4VX2M3Z85qfh3Hi8JEyxxk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
[steve@stapp02 ~]$ sudo useradd -M mark

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[steve@stapp02 ~]$ 
```
