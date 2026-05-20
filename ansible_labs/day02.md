# Day 02: Ansible Inventory and Basic Playbook Execution

## Task Overview

The Nautilus DevOps team is testing Ansible playbooks on various servers within their stack. The goal was to create an Ansible inventory file on the jump host to manage **App Server 1** in the Stratos DC and execute a playbook to install and start the `httpd` service.

### Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| Application Server 1 | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Jump Host | jump-host | thor | mjolnir123 | Provides secure access |

## Theories and Concepts

### 1. Ansible Inventory
An inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. It can be in INI or YAML format. In this task, we used the **INI format**.

### 2. Ansible Variables in Inventory
To enable Ansible to connect to managed nodes, we can define connection variables directly in the inventory:
- `ansible_host`: The IP address or hostname of the target server.
- `ansible_user`: The SSH username.
- `ansible_ssh_pass`: The SSH password.
- `ansible_become_pass`: The password for privilege escalation (sudo).
- `ansible_ssh_common_args`: Used here to disable strict host key checking for automated environments.

## Implementation Steps

1.  **Navigate to the project directory**:
    The playbooks and configuration files are located in `/home/thor/playbook/`.

2.  **Create the Inventory File**:
    Create a file named `inventory` (or `inventory.ini`) with the server details.

    ```ini
    [app_servers]
    stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n

    [app_servers:vars]
    ansible_ssh_common_args='-o StrictHostKeyChecking=no'
    ```

3.  **Verify Connection**:
    Use the `ping` module to check if Ansible can communicate with the server.
    ```bash
    ansible all -m ping -i inventory
    ```

4.  **Run the Playbook**:
    Execute the playbook using the created inventory.
    ```bash
    ansible-playbook -i inventory playbook.yml
    ```

## Mistakes Made and Fixes

| Problem | Cause | Fix |
| :--- | :--- | :--- |
| **SSH Host Key Verification Failed** | The jump host didn't recognize the host key of `stapp01`, causing the connection to hang or fail in non-interactive mode. | Added `ansible_ssh_common_args='-o StrictHostKeyChecking=no'` to the inventory variables to bypass manual confirmation. |
| **Authentication Denied for Sudo** | The playbook required `become: yes` to install `httpd`, but the sudo password wasn't provided. | Defined `ansible_become_pass=Ir0nM@n` in the inventory to automate privilege escalation. |
| **Inventory File Name Mismatch** | The requirement specified the file name as `inventory`, but sometimes users default to `hosts` or `inventory.ini`. | Renamed/created the file exactly as `inventory` to match the validation script requirements. |

## Terminal Output Reference

```bash
thor@jump-host ~/playbook$ cat inventory
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

thor@jump-host ~/playbook$ ansible all -m ping -i inventory
stapp01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

thor@jump-host ~/playbook$ ansible-playbook -i inventory playbook.yml 

PLAY [all] ******************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [stapp01]

TASK [Install httpd package] ************************************************************************************
changed: [stapp01]

TASK [Start service httpd] **************************************************************************************
changed: [stapp01]

PLAY RECAP ******************************************************************************************************
stapp01                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
