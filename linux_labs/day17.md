# Day 17: Linux User Resource Limits Configuration

## Infrastructure Details

*Note: Server credentials are provided below for documentation purposes as these specific resources no longer exist.*

| Server Name | IP | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |

## Theory

In the Stratos Datacenter, Application Server 1 (`stapp01`) has been encountering performance degradation due to an excessive number of processes held by the `nfsuser` user. To mitigate this issue, system administrators need to enforce limitations on its maximum processes.

The Linux operating system uses resource limits to restrict user resources (CPU, Memory, Processes), which preserves system stability. These are configured in `/etc/security/limits.conf`.

**Limit Types:**
* **Soft Limit:** The default limit enforced on the user. The user can temporarily increase this value up to the hard limit if needed.
* **Hard Limit:** The absolute maximum limit set by the system administrator (root). A regular user cannot exceed this limit under any circumstances.

**Task Requirements:**
1. Set the soft limit for `nfsuser` to **1027**
2. Set the hard limit for `nfsuser` to **2027**

## Professional DevOps Steps

### Step 1: Access Application Server 1
Log into `stapp01` via the Jump Host using the provided credentials.
```bash
ssh tony@stapp01
```

### Step 2: Check Existing Limits (Optional)
Switch to the `nfsuser` account and check current limitations.
```bash
sudo su - nfsuser
ulimit -Sn
ulimit -Hn
exit
```

### Step 3: Edit the Limits Configuration
Open `/etc/security/limits.conf` with a text editor using root privileges.
```bash
sudo vi /etc/security/limits.conf
```

### Step 4: Apply the New Limits
Append the following lines to the end of the config file to define the limits for `nfsuser`.
```text
nfsuser soft nproc 1027
nfsuser hard nproc 2027
```
*(Note: `nproc` specifies the maximum number of processes)*
Save and exit the file (e.g., `:wq` in vi).

### Step 5: Verify the Configuration
Switch back to `nfsuser` to verify that the soft and hard limits for user processes (`-u`) have been applied correctly:
```bash
sudo su - nfsuser
ulimit -Su
ulimit -Hu
```

## Real Output

```text
thor@jump-host ~$ sssh tony@stapp01
bash: sssh: command not found
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.244.164)' can't be established.
ED25519 key fingerprint is SHA256:466RAN7JeyRiOwcvqabKpPCT8zq92C2NEoQz3FlT8q8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 
Permission denied, please try again.
tony@stapp01's password: 
Last failed login: Sun Mar 29 03:23:50 UTC 2026 from 10.244.73.181 on ssh:notty
There was 1 failed login attempt since the last successful login.
[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ sudo su - nfsuser

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
[nfsuser@stapp01 ~]$ ulimit -Sn
1024
[nfsuser@stapp01 ~]$ ulimit
unlimited
[nfsuser@stapp01 ~]$ exit
logout
[tony@stapp01 ~]$ sudo vi /etc/security/limits.conf
[tony@stapp01 ~]$ cat /etc/security/limits.conf 
# /etc/security/limits.conf
#
#This file sets the resource limits for the users logged in via PAM.
#It does not affect resource limits of the system services.
#
#Also note that configuration files in /etc/security/limits.d directory,
#which are read in alphabetical order, override the settings in this
#file in case the domain is the same or more specific.
#That means, for example, that setting a limit for wildcard domain here
#can be overridden with a wildcard setting in a config file in the
#subdirectory, but a user specific setting here can be overridden only
#with a user specific setting in the subdirectory.
#
#Each line describes a limit for a user in the form:
#
#<domain>        <type>  <item>  <value>
#
#Where:
#<domain> can be:
#        - a user name
#        - a group name, with @group syntax
#        - the wildcard *, for default entry
#        - the wildcard %, can be also used with %group syntax,
#                 for maxlogin limit
#
#<type> can have the two values:
#        - "soft" for enforcing the soft limits
#        - "hard" for enforcing hard limits
#
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open file descriptors
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
#
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

nfsuser soft nproc 1027
nfsuser hard nproc 2027

# End of file
[tony@stapp01 ~]$ sudo su - nfsuser
Last login: Sun Mar 29 03:25:36 UTC 2026 on pts/0
[nfsuser@stapp01 ~]$ uilimit -Su
-bash: uilimit: command not found
[nfsuser@stapp01 ~]$ ulimit -Su
1027
[nfsuser@stapp01 ~]$ ulimit -Hu
2027
[nfsuser@stapp01 ~]$ 
```
