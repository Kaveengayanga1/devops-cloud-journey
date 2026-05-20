# Day 04: Update File Permissions for sysadmin Team

## Objective
Jump from the current host to App Server 2 (stapp02) and update file permissions for the sysadmin team on a script file.

## Output

```
thor@jumphost ~$ cd /
thor@jumphost /$ ssh steve@172.16.238.11
The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:QOvQnqYHm/gPEtvs1gVKx9wb6RdPzD1e5Pk3QKp7k9o.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password: 
[steve@stapp02 ~]$ 
[steve@stapp02 ~]$ 
[steve@stapp02 ~]$ ls
[steve@stapp02 ~]$ cd /
[steve@stapp02 /]$ ls
afs  boot  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
bin  dev   home  lib64  media       opt  root  sbin  sys  usr
[steve@stapp02 /]$ cd tmp/
[steve@stapp02 tmp]$ ls
systemd-private-239d470067814f5abf4a1baf5539a1d7-dbus-broker.service-PFsWVu
systemd-private-239d470067814f5abf4a1baf5539a1d7-systemd-hostnamed.service-U8TI56
systemd-private-239d470067814f5abf4a1baf5539a1d7-systemd-logind.service-cxhv5C
xfusioncorp.sh
[steve@stapp02 tmp]$ chmod 755 xfusioncorp.sh 
chmod: changing permissions of 'xfusioncorp.sh': Operation not permitted
[steve@stapp02 tmp]$ sudo chmod 755 xfusioncorp.sh 

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[steve@stapp02 tmp]$ ls -la
total 36
drwxrwxrwt 1 root root 4096 Jan 22 02:27 .
drwxr-xr-x 1 root root 4096 Jan 22 02:27 ..
drwxrwxrwt 2 root root 4096 Jan 22 02:25 .ICE-unix
drwxrwxrwt 2 root root 4096 Jan 22 02:25 .X11-unix
drwxrwxrwt 2 root root 4096 Jan 22 02:25 .XIM-unix
drwxrwxrwt 2 root root 4096 Jan 22 02:25 .font-unix
drwx------ 3 root root 4096 Jan 22 02:25 systemd-private-239d470067814f5abf4a1baf5539a1d7-dbus-broker.service-PFsWVu
drwx------ 3 root root 4096 Jan 22 02:25 systemd-private-239d470067814f5abf4a1baf5539a1d7-systemd-logind.service-cxhv5C
-rwxr-xr-x 1 root root   40 Jan 22 02:24 xfusioncorp.sh
[steve@stapp02 tmp]$ 
```

## Theory

To complete this task, you will need to jump from your current host to App Server 2 (stapp02) and update the file permissions for the sysadmin team. Based on the infrastructure details provided, here is the step-by-step execution:

### 1. Access App Server 2

From the jump host, SSH into the second app server using the credentials provided in your table:

```bash
ssh steve@172.16.238.11
```

When prompted for the password, enter: `Am3ric@`

### 2. Grant Executable Permissions

Once logged in, use the chmod command. The requirement is that all users (Owner, Group, and Others) must have the capability to execute the script.

The most efficient way to do this is using the symbolic mode `a+x` (all plus execute):

```bash
sudo chmod a+x /tmp/xfusioncorp.sh
```

Alternatively, you can use the absolute numeric mode `755`, which grants full permissions to the owner and read/execute permissions to everyone else:

```bash
sudo chmod 755 /tmp/xfusioncorp.sh
```

### 3. Verify the Change

Run a long listing on the file to ensure the execution bits (x) are now present for all categories:

```bash
ls -l /tmp/xfusioncorp.sh
```

**Expected Output:**

The output should start with `-rwxr-xr-x`, indicating that the owner, the group, and others all have execute permissions.

### Permission Breakdown

| Permission | Character | Numeric Value | Description |
|-----------|-----------|---------------|-------------|
| Read | r | 4 | Allows viewing file content |
| Write | w | 2 | Allows modifying file content |
| Execute | x | 1 | Allows running the file as a script |

