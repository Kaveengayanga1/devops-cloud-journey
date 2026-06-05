# Linux Access Control Lists (ACL)

## Scenario
The sysadmin team needs to secure the DNS resolution configuration file (`/etc/resolv.conf`) on App Server 3. Specific users require specialized access control that cannot be achieved with standard Linux file permissions alone.

### Tasks
On App Server 3 (`stapp03`):
1. Ensure the owner and group owner of `/etc/resolv.conf` are both set to `root`.
2. Set standard permissions so that the owner has read/write access, while group and others have read-only access (`644`).
3. Use Access Control Lists (ACL) to deny all permissions to the user `mariyam`.
4. Use ACL to grant read-only permission to the user `jerome`.

### Infrastructure Details
* **Server**: App Server 3 (`stapp03`)
* **User**: `banner`
* **Target File**: `/etc/resolv.conf`

## Theory & Key Concepts

* **File Ownership**: The `chown` command sets the user and group ownership of a file.
* **Standard Permissions**: `chmod 644` corresponds to `rw-r--r--`, which is the standard secure permission line for system configuration files.
* **Access Control Lists (ACL)**: Standard permissions only cover Owner, Group, and Others. ACLs allow you to define permissions for specific users outside of these three categories.
  * `setfacl -m u:username:perms file`: Modifies (`-m`) the ACL for a specific user (`u`).
  * `getfacl file`: Reads and displays the current ACL settings of a file.
  * **Mask**: Defines the maximum allowed permissions for ACL entries.

## Practical Steps (DevOps Approach)

1. **Log in to App Server 3**
   ```bash
   ssh banner@stapp03
   ```

2. **Check initial permissions and fix ownership**
   ```bash
   ls -l /etc/resolv.conf 
   sudo chown root:root /etc/resolv.conf
   ```

3. **Set standard permissions (644)**
   ```bash
   sudo chmod 644 /etc/resolv.conf
   ```

4. **Apply ACL rules for specific users**
   ```bash
   sudo setfacl -m u:mariyam:--- /etc/resolv.conf
   sudo setfacl -m u:jerome:r-- /etc/resolv.conf
   ```

5. **Verify the configuration**
   ```bash
   getfacl /etc/resolv.conf
   ```

## Output Log
```bash
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.49.18)' can't be established.
ED25519 key fingerprint is SHA256:G4op90ZMGD2r6eiU8v1sZfRF9E8mv/XD99VYMJalC20.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password: 
Permission denied, please try again.
banner@stapp03's password: 
Last failed login: Sat Mar 28 16:09:50 UTC 2026 from 10.244.195.114 on ssh:notty
There was 1 failed login attempt since the last successful login.
[banner@stapp03 ~]$ ls -l /etc/resolv.conf 
-rw-r--r-- 1 root root 112 Mar 28 15:53 /etc/resolv.conf
[banner@stapp03 ~]$ sudo chown root:root /etc/resolv.conf

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner: 
[banner@stapp03 ~]$ sudo chmod 644 /etc/resolv.conf
[banner@stapp03 ~]$ sudo setfacl -m u:mariyam:--- /etc/resolv.conf
[banner@stapp03 ~]$ sudo setfacl -m u:jerome:r-- /etc/resolv.conf
[banner@stapp03 ~]$ getfacl /etc/resolv.conf
getfacl: Removing leading '/' from absolute path names
# file: etc/resolv.conf
# owner: root
# group: root
user::rw-
user:mariyam:---
user:jerome:r--
group::r--
mask::r--
other::r--

[banner@stapp03 ~]$ 
```