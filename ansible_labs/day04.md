# Day 04: Ansible File Distribution and Inventory Management

## Task Overview
The Nautilus DevOps team needed to copy a file (`index.html`) from the jump host to all application servers (`stapp01`, `stapp02`, `stapp03`) in the Stratos DC using Ansible.

### Requirements:
1. Create an inventory file at `/home/thor/ansible/inventory`.
2. Create an Ansible playbook at `/home/thor/ansible/playbook.yml`.
3. The playbook should:
    - Ensure the destination directory `/opt/dba` exists.
    - Copy `/usr/src/dba/index.html` to `/opt/dba/index.html` on all app servers.
4. Use the provided server credentials.

## Infrastructure & Credentials
| Server Name | Hostname | User | Password | Become Password |
|-------------|----------|------|----------|-----------------|
| App Server 1| stapp01 | tony | Ir0nM@n | Ir0nM@n |
| App Server 2| stapp02 | steve| Am3ric@ | Am3ric@ |
| App Server 3| stapp03 | banner| BigGr33n| BigGr33n|
| Jump Host   | jump-host| thor | mjolnir123| - |

## Step-by-Step Implementation

### 1. Prepare Workspace
```bash
mkdir -p /home/thor/ansible
cd /home/thor/ansible
```

### 2. Create Inventory File
**File Path:** `/home/thor/ansible/inventory`
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_become_pass=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 3. Create Playbook
**File Path:** `/home/thor/ansible/playbook.yml`
```yaml
- name: Copy index.html to Application Servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Ensure the destination directory exists
      file:
        path: /opt/dba
        state: directory
        mode: '0755'

    - name: Copy index.html file to app servers
      copy:
        src: /usr/src/dba/index.html
        dest: /opt/dba/index.html
        mode: '0644'
```

### 4. Execute the Playbook
```bash
ansible-playbook -i inventory playbook.yml
```

---

## Troubleshooting & Mistakes Made

During the process, several mistakes were identified and resolved:

### Mistake 1: Duplicate Host Aliases in Inventory
**Issue:** When running `ansible all -i inventory -m ping`, only `stapp01` and `stapp02` responded. `stapp03` was missing.
**Reason:** In the initial inventory file, the third line used the alias `stapp01` for the host `stapp03`. 
```ini
stapp01 ansible_host=stapp01 ...
stapp02 ansible_host=stapp02 ...
stapp01 ansible_host=stapp03 ...  # ERROR: Duplicate alias 'stapp01'
```
Ansible overwrote the first `stapp01` entry with the last one because they shared the same name.
**Fix:** Corrected the alias to `stapp03`.

### Mistake 2: Missing Inventory Specification
**Issue:** `ansible all -m ping` returned a warning: `[WARNING]: provided hosts list is empty`.
**Reason:** Ansible looks for a default inventory file (usually `/etc/ansible/hosts`) if `-i` is not provided. Since we used a custom path, we must specify it.
**Fix:** Use the `-i inventory` flag.

### Mistake 3: Incorrect Playbook Path
**Issue:** `ansible-playbook -i inventory /ansible/playbook.yml` failed with `ERROR! could not be found`.
**Reason:** The user tried to use an absolute path `/ansible/playbook.yml` which didn't exist (it should have been `/home/thor/ansible/playbook.yml` or just `playbook.yml` while inside the directory).
**Fix:** Used the relative path `playbook.yml` after confirming the file location with `ls`.

## Summary of Success
After fixing the inventory typo, the `ping` module confirmed all three servers were reachable. The final playbook execution showed `changed: [stapp01]`, `changed: [stapp02]`, and `changed: [stapp03]`, indicating successful file distribution.
