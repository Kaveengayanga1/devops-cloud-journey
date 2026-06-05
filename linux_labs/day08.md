# Linux File Permissions

## Scenario
In a bid to automate backup processes, the xFusionCorp Industries sysadmin team has developed a new bash script named `xfusioncorp.sh`. While the script has been distributed to all necessary servers, it lacks executable permissions on App Server 2 within the Stratos Datacenter.

### Task
Grant executable permissions to the `/tmp/xfusioncorp.sh` script on App Server 2 (`stapp02`). Additionally, ensure that all users have the capability to execute it.

### Infrastructure Details
* **Server**: App Server 2 (`stapp02`)
* **User**: `steve`
* **Password**: `Am3ric@`
* **Target File**: `/tmp/xfusioncorp.sh`

## Theory & Key Concepts

* **Linux File Permissions**:
  * **Read (r)**: View file contents (Numeric value: 4)
  * **Write (w)**: Modify file contents (Numeric value: 2)
  * **Execute (x)**: Run the file as a script/program (Numeric value: 1)
* **User Categories**:
  * **Owner (u)**: The user who owns the file.
  * **Group (g)**: Users in the file's assigned group.
  * **Others (o)**: Everyone else.
* **`chmod` Command**: Used to change access permissions. 
  * `chmod +x file`: Adds execute permission.
  * `chmod 755 file`: Sets full permissions (7=4+2+1) for the owner, and read+execute (5=4+1) for group and others.

## Practical Steps (DevOps Approach)

1. **Log in to App Server 2**
   ```bash
   ssh steve@stapp02
   # Password: Am3ric@
   ```

2. **Navigate to the target directory and check current permissions**
   ```bash
   cd /tmp/
   ls -l xfusioncorp.sh
   # Notice the lack of 'x' in the output (e.g., ----------)
   ```

3. **Grant executable permissions to all users**
   ```bash
   sudo chmod 755 xfusioncorp.sh
   # OR
   sudo chmod a+x xfusioncorp.sh
   ```

4. **Verify the changes**
   ```bash
   ls -l xfusioncorp.sh
   # Output should now look like: -rwxr-xr-x
   ```

## Output Log
```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.97.194)' can't be established.
ED25519 key fingerprint is SHA256:H/OPqguzZ0FRZHugdtOqmrQiJlgP626pK3z99HUVGMY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
[steve@stapp02 ~]$ cd /tmp/
[steve@stapp02 tmp]$ ls -l xfusioncorp.sh 
---------- 1 root root 40 Mar 28 15:49 xfusioncorp.sh
[steve@stapp02 tmp]$ sudo chmod 755 xfusioncorp.sh 

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[steve@stapp02 tmp]$ cat xfusioncorp.sh 
#!/bin/bash

echo "Welcome To KodeKloud"[steve@stapp02 tmp]$ 
[steve@stapp02 tmp]$ ls -l xfusioncorp.sh
-rwxr-xr-x 1 root root 40 Mar 28 15:49 xfusioncorp.sh
```