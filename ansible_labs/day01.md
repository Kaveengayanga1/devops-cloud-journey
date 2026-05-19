# Ansible Day 01: Creating a File on a Remote Server

## Task Overview
The objective was to complete an Ansible configuration and playbook started by a team member on the `jump-host`. The goal was to create an empty file at `/tmp/file.txt` on **App Server 2** (`stapp02`) within the Stratos DC.

## Infrastructure & Credentials
The following credentials and server details were used for this task:

| Server Name | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| Jump Host | `jump-host` | `thor` | `mjolnir123` | Control Node |
| App Server 1 | `stapp01` | `tony` | `Ir0nM@n` | Managed Node |
| App Server 2 | `stapp02` | `steve` | `Am3ric@` | Managed Node (Target) |
| App Server 3 | `stapp03` | `banner` | `BigGr33n` | Managed Node |
| DB Server | `stdb01` | `peter` | `Sp!dy` | Database |
| Storage Server | `ststor01` | `natasha` | `Bl@kW` | Storage |

## Theoretical Concepts
1.  **Ansible Inventory**: A file that lists the hosts and groups of hosts upon which commands, modules, and playbooks operate. It includes connection details like `ansible_user` and `ansible_ssh_pass`.
2.  **Ansible Playbook**: A YAML file containing a list of "plays" that map groups of hosts to well-defined roles/tasks.
3.  **File Module**: Used for managing files and directories. The `state: touch` parameter ensures a file is created if it doesn't exist.
4.  **Become**: Ansible's privilege escalation system (similar to `sudo`).
5.  **SSH Common Args**: Used to bypass manual SSH fingerprint confirmation (`StrictHostKeyChecking=no`).

## Implementation Steps

### 1. Inventory Configuration
The inventory file located at `/home/thor/ansible/inventory` was updated to include connection details for `stapp02`.
```ini
stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become_pass=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 2. Playbook Creation
A playbook named `/home/thor/ansible/playbook.yml` was created with the following content:
```yaml
- name: Create a file on App Server 2
  hosts: stapp02
  become: yes
  tasks:
    - name: Create an empty file at /tmp/file.txt
      file:
        path: /tmp/file.txt
        state: touch
```

### 3. Execution
The playbook was executed using the command:
```bash
ansible-playbook -i inventory playbook.yml
```

---

## Mistakes Made & Corrections

### Mistake 1: Incorrect Directory Navigation
-   **What Happened**: Attempted to `cd /home/thor/inv` and `cd ansible/inventory`.
-   **Why**: Confusion regarding the directory structure. `inventory` was a file inside the `ansible` directory, not a directory itself.
-   **Fix**: Listed the directory contents using `ls` to identify the correct path and navigated to `/home/thor/ansible/`.

### Mistake 2: Missing Privilege Escalation Password
-   **What Happened**: Initially, the inventory only had `ansible_ssh_pass`.
-   **Why**: While `become: yes` was in the playbook, Ansible didn't have the password to perform `sudo` operations on the remote host.
-   **Fix**: Added `ansible_become_pass=Am3ric@` to the inventory file so Ansible could provide the password when escalating privileges.

## Final Output Verification
After running the playbook, the file was verified on `stapp02`:
```bash
ssh steve@stapp02
ls /tmp/file.txt
# Output: /tmp/file.txt (Success)
```
