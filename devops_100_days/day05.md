# Day 05: Disable SELinux on App Server 2

## Objective
Install SELinux packages and permanently disable SELinux on App Server 2 (stapp02) to prepare for security enhancement at xFusionCorp Industries.

## Output

```
thor@jumphost ~$ ssh steve@172.16.238.11
The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:lIyVc512XRS3qVS4AylsXmR1tiZJtcGAK9O4mCeeWYA.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password: 
[steve@stapp02 ~]$ sudo yum install -y policycoreutils selinux-policy selinux-policy-targeted libselinux-utils

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
CentOS Stream 9 - BaseOS                                  38 kB/s | 6.7 kB     00:00    
CentOS Stream 9 - BaseOS                                  12 MB/s | 8.9 MB     00:00    
CentOS Stream 9 - AppStream                               41 kB/s | 6.8 kB     00:00    
CentOS Stream 9 - AppStream                               37 MB/s |  26 MB     00:00    
CentOS Stream 9 - Extras packages                         58 kB/s | 7.3 kB     00:00    
CentOS Stream 9 - Extras packages                        2.8 kB/s |  20 kB     00:07    
Extra Packages for Enterprise Linux 9 - x86_64           144 kB/s |  32 kB     00:00    
Extra Packages for Enterprise Linux 9 - x86_64            17 MB/s |  20 MB     00:01    
Extra Packages for Enterprise Linux 9 openh264 (From Cis 5.4 kB/s | 993  B     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64     97 kB/s |  21 kB     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64    591 kB/s | 259 kB     00:00    
Dependencies resolved.
=========================================================================================
 Package                        Architecture  Version                Repository     Size
=========================================================================================
Installing:
 libselinux-utils               x86_64        3.6-3.el9              baseos        190 k
 policycoreutils                x86_64        3.6-4.el9              baseos        238 k
 selinux-policy                 noarch        38.1.71-1.el9          baseos         40 k
 selinux-policy-targeted        noarch        38.1.71-1.el9          baseos        6.9 M
Installing dependencies:
 diffutils                      x86_64        3.7-12.el9             baseos        397 k
 rpm-plugin-selinux             x86_64        4.16.1.3-38.el9        baseos         16 k

Transaction Summary
=========================================================================================
Install  6 Packages

Total download size: 7.8 M
Installed size: 21 M
Downloading Packages:
(1/6): policycoreutils-3.6-4.el9.x86_64.rpm              488 kB/s | 238 kB     00:00    
(2/6): libselinux-utils-3.6-3.el9.x86_64.rpm             379 kB/s | 190 kB     00:00    
(3/6): diffutils-3.7-12.el9.x86_64.rpm                   751 kB/s | 397 kB     00:00    
(4/6): rpm-plugin-selinux-4.16.1.3-38.el9.x86_64.rpm      59 kB/s |  16 kB     00:00    
(5/6): selinux-policy-38.1.71-1.el9.noarch.rpm           128 kB/s |  40 kB     00:00    
(6/6): selinux-policy-targeted-38.1.71-1.el9.noarch.rpm   11 MB/s | 6.9 MB     00:00    
-----------------------------------------------------------------------------------------
Total                                                    5.5 MB/s | 7.8 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: selinux-policy-targeted-38.1.71-1.el9.noarch                    1/1 
  Preparing        :                                                                 1/1 
  Installing       : libselinux-utils-3.6-3.el9.x86_64                               1/6 
  Installing       : diffutils-3.7-12.el9.x86_64                                     2/6 
  Installing       : policycoreutils-3.6-4.el9.x86_64                                3/6 
  Running scriptlet: policycoreutils-3.6-4.el9.x86_64                                3/6 
Created symlink /etc/systemd/system/sysinit.target.wants/selinux-autorelabel-mark.service → /usr/lib/systemd/system/selinux-autorelabel-mark.service.

  Installing       : selinux-policy-38.1.71-1.el9.noarch                             4/6 
  Running scriptlet: selinux-policy-38.1.71-1.el9.noarch                             4/6 
  Running scriptlet: selinux-policy-targeted-38.1.71-1.el9.noarch                    5/6 
  Installing       : selinux-policy-targeted-38.1.71-1.el9.noarch                    5/6 
  Running scriptlet: selinux-policy-targeted-38.1.71-1.el9.noarch                    5/6 
  Installing       : rpm-plugin-selinux-4.16.1.3-38.el9.x86_64                       6/6 
  Running scriptlet: rpm-plugin-selinux-4.16.1.3-38.el9.x86_64                       6/6 
  Verifying        : diffutils-3.7-12.el9.x86_64                                     1/6 
  Verifying        : libselinux-utils-3.6-3.el9.x86_64                               2/6 
  Verifying        : policycoreutils-3.6-4.el9.x86_64                                3/6 
  Verifying        : rpm-plugin-selinux-4.16.1.3-38.el9.x86_64                       4/6 
  Verifying        : selinux-policy-38.1.71-1.el9.noarch                             5/6 
  Verifying        : selinux-policy-targeted-38.1.71-1.el9.noarch                    6/6 

Installed:
  diffutils-3.7-12.el9.x86_64            libselinux-utils-3.6-3.el9.x86_64              
  policycoreutils-3.6-4.el9.x86_64       rpm-plugin-selinux-4.16.1.3-38.el9.x86_64      
  selinux-policy-38.1.71-1.el9.noarch    selinux-policy-targeted-38.1.71-1.el9.noarch   

Complete!
[steve@stapp02 ~]$ sudo vi /etc/selinux/config
[steve@stapp02 ~]$ grep "SELINUX=" /etc/selinux/config
# SELINUX= can take one of these three values:
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
SELINUX=disabled
[steve@stapp02 ~]$ 
```

## Theory

To prepare for the security enhancement at xFusionCorp Industries, it is essential to understand how SELinux manages security and how its state is controlled within the Linux kernel.

### 1. Core Theories of SELinux

#### What is SELinux?

Security-Enhanced Linux (SELinux) is a kernel security module that provides a mechanism for supporting Mandatory Access Control (MAC). Unlike standard Linux security (Discretionary Access Control - DAC), where a file owner can grant permissions at will, SELinux allows the system administrator to define a centralized security policy. Even if a process is running as root, SELinux can block it from accessing files it doesn't need for its specific job.

#### SELinux Operational States

SELinux can exist in one of three states:

- **Enforcing**: The default mode. Policies are enforced, and unauthorized access attempts are blocked and logged.

- **Permissive**: SELinux is active but does not block any actions. Instead, it logs "denials" that would have occurred in Enforcing mode. This is used for debugging.

- **Disabled**: The SELinux infrastructure is completely turned off in the kernel. No logs are generated, and no policies are loaded.

#### Temporary vs. Permanent Configuration

- **Runtime Change**: Using `setenforce 0` or `1` changes the state immediately but does not persist after a reboot.

- **Persistent Change**: Modifying the configuration file at `/etc/selinux/config` ensures the setting is applied during the boot sequence.

### 2. Implementation Steps for App Server 2

#### Step 1: Access the Server

Log in to App Server 2 using the provided credentials:

```bash
ssh steve@172.16.238.11
```

Password: `Am3ric@`

#### Step 2: Install Required Packages

On Red-Hat-based systems (commonly used in Stratos DC), the core packages include `policycoreutils`, `selinux-policy`, and `selinux-policy-targeted`.

```bash
sudo yum install -y policycoreutils selinux-policy selinux-policy-targeted libselinux-utils
```

#### Step 3: Permanently Disable SELinux

To meet the requirement that the status remains disabled after the scheduled maintenance reboot tonight, you must edit the configuration file.

Open the config file:

```bash
sudo vi /etc/selinux/config
```

Locate the line starting with `SELINUX=`.

Change the value to `disabled`:

```
SELINUX=disabled
```

Save and exit (`:wq`).

#### Step 4: Verification (Optional)

Since the requirement states no reboot is needed now, you can verify that the file is saved correctly without changing the current running state:

```bash
grep "SELINUX=" /etc/selinux/config
```

**Note**: Because you are not rebooting now, `sestatus` will still show the previous state of the kernel. However, upon the next boot, the kernel will read your change and keep SELinux disabled.
