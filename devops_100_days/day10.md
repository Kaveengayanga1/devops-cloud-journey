# Day 10: Automating Website Backups with Bash and Ansible

## Task Description
The production support team of xFusionCorp Industries is working on developing some bash scripts to automate different day-to-day tasks. One is to create a bash script for taking website backups. They have a static website running on App Server 1 in Stratos Datacenter, and they need to create a bash script named `ecommerce_backup.sh` under the `/scripts` directory on App Server 1.

Tasks to accomplish:
a. Create a zip archive named `xfusioncorp_ecommerce.zip` of `/var/www/html/ecommerce` directory.
b. Save the archive in `/backup/` on App Server 1.
c. Copy the created archive to Nautilus Storage Server in `/backup/` location.
d. Ensure the script won't ask for a password while copying the archive file. The respective server user (e.g., `tony`) must be able to run it.
e. Do not use `sudo` inside the script.

## Infrastructure Details & Credentials
| Server Name | Hostname | User | Password |
| :--- | :--- | :--- | :--- |
| Application Server 1 | stapp01 | tony | Ir0nM@n |
| Application Server 2 | stapp02 | steve | Am3ric@ |
| Application Server 3 | stapp03 | banner | BigGr33n |
| Storage Server | ststor01 | natasha | Bl@kW |
| Jump Host Server | jump-host | thor | mjolnir123 |
| DB Server | stdb01 | peter | Sp!dy |
| Load Balancer | stlb01 | loki | Mischi3f |
| Backup Server | stbkp01 | clint | H@wk3y3 |
| Mail Server | stmail01 | groot | Gr00T123 |
| Jenkins Server | jenkins | jenkins | j@rv!s |

## Theory & Concepts

### Bash Scripting & Variables
Using variables for directories and filenames makes scripts reusable and easier to maintain.

### Archiving (Zip Utility)
The `zip` command is used to compress files. The `-r` flag allows recursive compression of directories, and `-q` suppresses the output.

### Secure Copy (SCP) & SSH
SCP transfers files between servers over SSH. To prevent password prompts, SSH key-based authentication (passwordless SSH) must be established between the source and destination users.

### Ansible Playbooks for Automation
Ansible allows us to automate the entire setup process. We can use a playbook to:
- Create required directories.
- Copy the backup script to the target server.
- Execute the script as the correct user without requiring manual intervention.

## Execution Output

**1. Setting up Passwordless SSH:**
```bash
thor@jump-host ~$ ssh-keygen -t rsa
# (Key generated)
thor@jump-host ~$ ssh-copy-id tony@stapp01
# (Key copied to stapp01)

[tony@stapp01 ~]$ ssh-keygen -t rsa
[tony@stapp01 ~]$ ssh-copy-id natasha@ststor01
```

**2. Verifying Backup Directory Permissions:**
```bash
[tony@stapp01 ~]$ ls -ld /backup
drwxrwxrwx 2 root root 4096 Mar 28 03:24 /backup
```

**3. Running the Ansible Playbook:**
```bash
thor@jump-host ~$ ansible-playbook -i hosts.ini backup_playbook2.yml 

PLAY [Setup and Run Ecommerce Backup Script] ********************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [stapp01]

TASK [Create /scripts directory] ********************************************************************************
changed: [stapp01]

TASK [Copy backup script to App Server 1] ***********************************************************************
changed: [stapp01]

TASK [Execute the backup script as tony] ************************************************************************
changed: [stapp01]

TASK [Show script output] ***************************************************************************************
ok: [stapp01] => {
    "script_output.stdout": "Backup completed successfully."
}

PLAY RECAP ******************************************************************************************************
stapp01                    : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**4. Verifying the Backup on the Storage Server:**
```bash
[tony@stapp01 ~]$ ssh natasha@ststor01 "ls -l /backup/xfusioncorp_ecommerce.zip"
-rw-r--r-- 1 natasha natasha 433 Mar 28 03:24 /backup/xfusioncorp_ecommerce.zip
```
