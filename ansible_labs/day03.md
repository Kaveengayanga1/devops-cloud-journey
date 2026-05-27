# Day 03: Configuring Ansible Default Remote User and Inventory

## Task Overview
The Nautilus DevOps team needs to manage all servers in their infrastructure using Ansible. The requirement is to configure a common default SSH user (`john`) for all managed hosts within the global Ansible configuration and set up an inventory file with specific connection arguments.

## Infrastructure Details
| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Application Server 1 | stapp01 | tony | Ir0nM@n | Nautilus Application 1 |
| Application Server 2 | stapp02 | steve | Am3ric@ | Nautilus Application 2 |
| Application Server 3 | stapp03 | banner | BigGr33n | Nautilus Application 3 |
| LoadBalancer Server | stlb01 | loki | Mischi3f | Nautilus HTTP Load Balancer |
| Database Server | stdb01 | peter | Sp!dy | Nautilus Database |
| Storage Server | ststor01 | natasha | Bl@kW | Nautilus Storage |
| Backup Server | stbkp01 | clint | H@wk3y3 | Nautilus Backups |
| Mail Server | stmail01 | groot | Gr00T123 | Nautilus Mail Services |
| Jump Host | jump-host | thor | mjolnir123 | Secure Access Node |
| Jenkins Server | jenkins | jenkins | j@rv!s | CI/CD Pipeline |

## Phase 1: Modifying Ansible Configuration

### 1. Verification of Initial State
First, we check the current Ansible version and the location of the active configuration file.
```bash
thor@jump-host ~$ ansible --version
ansible [core 2.14.18]
  config file = /etc/ansible/ansible.cfg
  ...
```

### 2. Mistakes Made & Troubleshooting
During the task, an attempt was made to change permissions of the configuration file to write to it without elevated privileges.

**Mistake:**
```bash
thor@jump-host ~$ chmod u+w /etc/ansible/ansible.cfg 
chmod: changing permissions of '/etc/ansible/ansible.cfg': Operation not permitted
```
**Why it happened:**
The `/etc/ansible/ansible.cfg` file is a system-level configuration file owned by the `root` user. The current user `thor` does not have the authority to change the file permissions directly.

**How it was fixed:**
Instead of changing permissions, use `sudo` to edit the file with root privileges.
```bash
thor@jump-host ~$ sudo vi /etc/ansible/ansible.cfg
```

### 3. Implementation
Edit the `[defaults]` section in `/etc/ansible/ansible.cfg` to set the `remote_user`.

```ini
[defaults]
remote_user = john
```

### 4. Verification
Verify that the configuration change is recognized by Ansible.
```bash
thor@jump-host ~$ ansible-config dump | grep REMOTE_USER
DEFAULT_REMOTE_USER(/etc/ansible/ansible.cfg) = john
```

---

## Phase 2: Creating the Inventory File

### 1. Requirements
Create a host list (inventory) and ensure that for the `app_servers` group, SSH strict host key checking is disabled to allow automated connections.

### 2. Implementation
Edit `/etc/ansible/hosts`:
```bash
sudo vi /etc/ansible/hosts
```

Add the following content:
```ini
[app_servers]
stapp01
stapp02
stapp03

[lb_servers]
stlb01

[db_servers]
stdb01

[storage_servers]
ststor01

[backup_servers]
stbkp01

[mail_servers]
stmail01

[jenkins_servers]
jenkins

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 3. Final Verification
Check if all hosts are listed correctly:
```bash
ansible all --list-hosts
```

Test connectivity:
```bash
ansible all -m ping
```

## Summary
- **Primary Configuration:** `/etc/ansible/ansible.cfg` updated with `remote_user = john`.
- **Inventory Management:** `/etc/ansible/hosts` populated with categorized server groups.
- **Security Bypass:** `StrictHostKeyChecking=no` applied to `app_servers` to streamline initial automation.
