# Day 05: Ansible File Module and Inventory Variables

## Task Overview
The Nautilus DevOps team is testing various Ansible modules on servers in Stratos DC. The objective is to create a blank file on remote hosts with specific permissions and ownership using Ansible.

### Requirements:
1. Create an inventory file `~/playbook/inventory` on the jump host including all app servers.
2. Create a playbook `~/playbook/playbook.yml` to create a blank file `/opt/nfsdata.txt` on all app servers.
3. Set permissions of `/opt/nfsdata.txt` to `0655`.
4. Ensure the owner (user and group) of `/opt/nfsdata.txt` is:
   - `tony` on **stapp01**
   - `steve` on **stapp02**
   - `banner` on **stapp03**

---

## Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Application Server 1 | stapp01 | tony | Ir0nM@n | Nautilus Application 1 |
| Application Server 2 | stapp02 | steve | Am3ric@ | Nautilus Application 2 |
| Application Server 3 | stapp03 | banner | BigGr33n | Nautilus Application 3 |
| Jump Host | jump-host | thor | mjolnir123 | Secure Access |

---

## Initial Attempt & Mistakes Made

### 1. Inventory without Variables
Initially, the inventory file was created with connection details but lacked the specific ownership variables required by the playbook.

**Inventory File (`~/playbook/inventory`):**
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_become_pass=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 2. Path Error
The user tried to run the playbook from the home directory while the files were inside the `playbook` folder.
**Command:** `ansible-playbook -i inventory playbook.yml`
**Error:** `ERROR! the playbook: playbook.yml could not be found`
**Fix:** Navigate to the correct directory: `cd ~/playbook`

### 3. Undefined Variable Error
When running the playbook, it failed because the variable `file_owner` was used in the playbook but not defined in the inventory.

**Error Output:**
```text
fatal: [stapp01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'file_owner' is undefined..."}
```

---

## The Solution

### Step 1: Update the Inventory
To fix the undefined variable error, the `file_owner` variable was added to each host in the inventory file.

**Updated Inventory (`~/playbook/inventory`):**
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n file_owner=tony
stapp02 ansible_host=stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become_pass=Am3ric@ file_owner=steve
stapp03 ansible_host=stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_become_pass=BigGr33n file_owner=banner

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### Step 2: Create the Playbook
The playbook uses the `ansible.builtin.file` module with `state: touch` to create the blank file.

**Playbook (`~/playbook/playbook.yml`):**
```yaml
---
- name: Create a blank file on all app servers
  hosts: app_servers
  become: true
  tasks:
    - name: Ensure the file /opt/nfsdata.txt exists with correct attributes
      ansible.builtin.file:
        path: /opt/nfsdata.txt
        state: touch
        mode: '0655'
        owner: "{{ file_owner }}"
        group: "{{ file_owner }}"
```

### Step 3: Execution
Run the playbook from the `~/playbook` directory:
```bash
ansible-playbook -i inventory playbook.yml
```

---

## Theory and Concepts

### Ansible Inventory
A file that contains information about the hosts you want to manage. You can define host-specific variables (like `file_owner`) directly in this file.

### Ansible File Module
Used for managing files, directories, and symlinks.
- `state: touch`: Creates an empty file if it doesn't exist.
- `mode`: Sets permissions (e.g., `0655`).
- `owner/group`: Sets the ownership of the file.

### Privilege Escalation (`become: true`)
Since `/opt` is a system-protected directory, Ansible needs `sudo` privileges to create files there. `become: true` instructs Ansible to execute the task as root.
