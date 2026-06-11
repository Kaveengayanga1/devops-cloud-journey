# Day 18 - SELinux Configuration

## Task Requirements
Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 2 in the Stratos Datacenter:

1. Install the required SELinux packages.
2. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.
3. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.
4. Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.

### Resources & Credentials
*   **Server:** Application Server 2 (App server 2)
*   **Host:** `stapp02`
*   **Username:** `steve`
*   **Password:** `Am3ric@`

## Theory

### 1. What is SELinux (Security-Enhanced Linux)?
SELinux is a "Mandatory Access Control" (MAC) mechanism that provides an extra layer of security to the Linux kernel. It offers more advanced security to control permissions for files and processes than a standard Linux system.

### 2. The 3 Main Modes of SELinux
A SELinux system can operate in 3 modes:
*   **Enforcing:** All security policies are enforced. Prevents unauthorized access.
*   **Permissive:** Security policies are not enforced, but warnings (logs) regarding policy violations are recorded. This is used for troubleshooting.
*   **Disabled:** SELinux is completely turned off. The system does not check any security policies.

### 3. Temporary vs Permanent Changes
*   **Temporary:** When using a command like `setenforce 0`, it only applies to the current session. When the server reboots, it reverts to its previous state.
*   **Permanent:** When changing the configuration file `/etc/selinux/config`, the changes remain permanently even after the server reboots.

## Execution Steps (Professional DevOps Approach)

**Step 1: Login to the Server**
First, log in to App server 2 using the provided credentials via the terminal.
```bash
ssh steve@stapp02
# Enter password: Am3ric@
```

**Step 2: Privilege Escalation (Gain Root Access)**
Root privileges are required to change system configurations.
```bash
sudo -i
# Enter password for steve
```

**Step 3: Install Required SELinux Packages**
Use the following command to install the required packages (commonly `yum` is used in RHEL/CentOS systems).
```bash
yum install selinux-policy selinux-policy-targeted libselinux-utils -y
```

**Step 4: Permanently Disable SELinux**
Here we need to edit the configuration file. You can use the `vi` or `nano` editor.
```bash
vi /etc/selinux/config
```
Once the file is open, find the line that says `SELINUX=enforcing` or `SELINUX=permissive` and change it to:
```plaintext
SELINUX=disabled
```
*(Note: Do not change the `SELINUXTYPE` line)*
Save the changes (type `:wq` in `vi`) and exit.

**Step 5: Verification**
According to the instructions, there is no need to reboot now. However, to verify if the config file has been updated correctly, use the following command:
```bash
cat /etc/selinux/config | grep SELINUX=
```
The output should show `SELINUX=disabled`.

## Task Output
```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.13.35)' can't be established.
ED25519 key fingerprint is SHA256:otUeJb6Nak8gslBiJx1FNrNAcL7rNAk841wfK4nTdNM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
[steve@stapp02 ~]$ sudo -i

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[root@stapp02 ~]# yum install selinux-policy selinux-policy-targeted libselinux-utils -y
CentOS Stream 9 - BaseOS                                                         5.0 MB/s | 8.9 MB     00:01    
CentOS Stream 9 - AppStream                                                       14 MB/s |  27 MB     00:01    
CentOS Stream 9 - Extras packages                                                 25 kB/s |  21 kB     00:00    
Extra Packages for Enterprise Linux 9 - x86_64                                   9.8 MB/s |  20 MB     00:02    
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64             4.0 kB/s | 2.5 kB     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64                            436 kB/s | 260 kB     00:00    
Package libselinux-utils-3.6-1.el9.x86_64 is already installed.
Dependencies resolved.
=================================================================================================================
 Package                                 Architecture      Version                    Repository            Size
=================================================================================================================
Installing:
 selinux-policy                          noarch            38.1.76-1.el9              baseos                40 k
 selinux-policy-targeted                 noarch            38.1.76-1.el9              baseos               6.9 M
Upgrading:
 audit-libs                              x86_64            3.1.5-8.el9                baseos               121 k
 libselinux                              x86_64            3.6-3.el9                  baseos                86 k
 libselinux-utils                        x86_64            3.6-3.el9                  baseos               190 k
 python3-rpm                             x86_64            4.16.1.3-37.el9            baseos                65 k
 rpm                                     x86_64            4.16.1.3-37.el9            baseos               536 k
 rpm-build-libs                          x86_64            4.16.1.3-37.el9            baseos                89 k
 rpm-libs                                x86_64            4.16.1.3-37.el9            baseos               308 k
 rpm-sign-libs                           x86_64            4.16.1.3-37.el9            baseos                21 k
Installing dependencies:
 checkpolicy                             x86_64            3.6-1.el9                  appstream            353 k
 flatpak-selinux                         noarch            1.12.9-1.el9               appstream             22 k
 policycoreutils-python-utils            noarch            3.6-2.1.el9                appstream             77 k
 python3-audit                           x86_64            3.1.5-8.el9                appstream             82 k
 python3-distro                          noarch            1.5.0-7.el9                appstream             37 k
 python3-libselinux                      x86_64            3.6-3.el9                  appstream            188 k
 python3-libsemanage                     x86_64            3.6-1.el9                  appstream             80 k
 python3-policycoreutils                 noarch            3.6-2.1.el9                appstream            2.1 M
 python3-setools                         x86_64            4.4.4-1.el9                baseos               605 k
 python3-setuptools                      noarch            53.0.0-15.el9              baseos               936 k
 rpm-plugin-selinux                      x86_64            4.16.1.3-37.el9            baseos                17 k

Transaction Summary
=================================================================================================================
Install  13 Packages
Upgrade   8 Packages

Complete!
[root@stapp02 ~]# vi /etc/selinux/config
[root@stapp02 ~]# cat /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted


[root@stapp02 ~]# cat /etc/selinux/config | grep SELINUX=
# SELINUX= can take one of these three values:
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
SELINUX=disabled
[root@stapp02 ~]# 
```
