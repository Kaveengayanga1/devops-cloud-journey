# Secure File Transfer (SCP)

## Scenario
A Nautilus developer has stored confidential data on the jump host within Stratos DC. To ensure security and compliance, this data must be transferred to one of the app servers. Given developers lack direct access to these servers, the system admin team has been enlisted for assistance.

### Task
Copy the `/tmp/nautilus.txt.gpg` file from the Jump Server to App Server 2 (`stapp02`), placing it in the directory `/home/data`.

### Infrastructure Details
* **Source Server**: Jump Host (`jump-host`)
* **Target Server**: App Server 2 (`stapp02`)
* **Target User**: `steve`
* **Target Password**: `Am3ric@`
* **Source File**: `/tmp/nautilus.txt.gpg`
* **Destination Path**: `/home/data`

## Theory & Key Concepts

* **Jump Host (Bastion Host)**: An intermediate server used as a "gateway" to access secure internal networks. Direct internet access to application servers is normally restricted; you first connect to the Jump Host and orchestrate internal tasks from there.
* **SSH (Secure Shell)**: A cryptographic network protocol for operating network services securely over an unsecured network. It is used to run remote commands and authenticate securely.
* **SCP (Secure Copy Protocol)**: A command-line tool based on SSH used to copy files securely across a network between different hosts. Since it runs over SSH, it provides the same authentication and security.

### Syntax for SCP
```bash
scp <source_file_path> <username>@<destination_host>:<destination_directory>
```

## Practical Steps (DevOps Approach)

1. **Log in to the Jump Host**
   Start by navigating into the bastion host.
   ```bash
   ssh thor@jump-host
   # Password: mjolnir123
   ```

2. **Verify the Source File exists**
   Check that the file is present in the specified directory before attempting the transfer.
   ```bash
   ls -l /tmp/nautilus.txt.gpg
   ```

3. **Copy the File to App Server 2 using SCP**
   Use `scp` to send the file securely to `stapp02`. Providing the destination path is crucial.
   ```bash
   scp /tmp/nautilus.txt.gpg steve@stapp02:/home/data
   # Enter password when prompted: Am3ric@
   ```

4. **Verify the Transfer**
   Log into the destination server and ensure the file arrived securely and intact.
   ```bash
   ssh steve@stapp02
   # Enter password: Am3ric@
   ls -l /home/data/nautilus.txt.gpg
   ```

*(DevOps Pro-Tip: After confirming the confidential data transfer, it's good practice to securely clean up the leftover file on the temporary layer: `rm /tmp/nautilus.txt.gpg`.)*

## Output Log
```bash
thor@jump-host ~$ ls -l /tmp/nautilus.txt.gpg
-rw-r--r-- 1 root root 105 Mar 29 02:06 /tmp/nautilus.txt.gpg
thor@jump-host ~$ scp /tmp/nautilus.txt.gpg steve@stapp02:/home/data
The authenticity of host 'stapp02 (10.244.244.163)' can't be established.
ED25519 key fingerprint is SHA256:G9nVEMFTRhISU98zbN3D/lu2CkDxhntVonMLIP6fuHA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
nautilus.txt.gpg                                                               100%  105   293.1KB/s   00:00    
thor@jump-host ~$ ssh steve@stapp02
steve@stapp02's password: 
[steve@stapp02 ~]$ ls -l /home/data/nautilus.txt.gpg
-rw-r--r-- 1 steve steve 105 Mar 29 02:11 /home/data/nautilus.txt.gpg
[steve@stapp02 ~]$ 
```
