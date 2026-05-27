# Day 05: Create a Temporary User with Expiry using Ansible

## Task Description
As part of a temporary Nautilus project assignment, create a lowercase user `javed` on App Server 1 (`stapp01`) with account expiry date `2027-02-17`.

## Requirements
- User must be created as `javed` (lowercase only).
- Target server: App Server 1 (`stapp01`).
- Use Ansible from `jump-host`.
- Account expiry must be exactly `2027-02-17`.
- Verification must confirm user exists and account expiry is correct.

## Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Application Server 1 | `stapp01` | `tony` | `Ir0nM@n` | Hosts Nautilus Application 1 |
| Jump Host Server | `jump-host` | `thor` | `mjolnir123` | Provides secure access to Stork DC |

## Core Theory

### 1) Why this task matters
Temporary access control is part of access lifecycle management. Instead of manually disabling accounts later, setting an expiry at creation time enforces automatic cleanup.

### 2) Linux account expiry basics
- Linux stores user/account metadata in system files (`/etc/passwd`, `/etc/shadow`).
- Account expiry disables login after the configured date.
- `chage -l <user>` is the standard command to validate account aging and expiry.

### 3) Why Ansible for this
- **Idempotent:** Running the same playbook repeatedly keeps desired state.
- **Scalable:** Same playbook can target one server or many servers.
- **Auditable:** YAML files become version-controlled infrastructure records.

### 4) `ansible.builtin.user` and expiry format
In Ansible, the `expires` field requires Unix epoch time (seconds).

```bash
date -d "2027-02-17" +%s
```

Expected result:

```bash
1802822400
```

### 5) SSH host key checking issue (real-world)
When password authentication is used with `sshpass`, first-time host key prompts can fail non-interactive Ansible runs. In lab/sandbox setups, adding the inventory var below avoids that block:

```ini
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

## Step-by-Step Execution (Professional Flow)

### Step 1: Create playbook

```bash
vi create_temp_user.yml
```

`create_temp_user.yml`:

```yaml
---
- name: Manage Temporary Project Access
  hosts: app_servers
  become: yes
  tasks:
    - name: Create temporary user javed with expiry date
      ansible.builtin.user:
        name: javed
        shell: /bin/bash
        state: present
        # 2027-02-17 in Epoch time is 1802822400
        expires: 1802822400
        comment: "Temporary access for Nautilus Project"
```

### Step 2: Prepare inventory

```bash
vi inventory.ini
```

Initial inventory:

```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n
```

### Step 3: Run playbook (first attempt)

```bash
ansible-playbook -i inventory.ini create_temp_user.yml
```

Observed issue:
- Failure caused by SSH host key checking + password auth (`sshpass`).

### Step 4: Fix inventory for lab execution
Add:

```ini
[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### Step 5: Re-run playbook

```bash
ansible-playbook -i inventory.ini create_temp_user.yml
```

Expected result:
- `ok: [stapp01]`
- `changed: [stapp01]`

### Step 6: Verify through Ansible ad-hoc command
Use correct inventory file name (`inventory.ini`, not `hosts.ini`):

```bash
ansible app_servers -i inventory.ini -m shell -a "chage -l javed" -b
```

Expected key line:

```text
Account expires : Feb 17, 2027
```

## Validation Checklist
- [ ] `create_temp_user.yml` contains `name: javed` in lowercase.
- [ ] `expires` is set to `1802822400`.
- [ ] `ansible-playbook -i inventory.ini create_temp_user.yml` completes without failures.
- [ ] `chage -l javed` via Ansible shows `Account expires : Feb 17, 2027`.
- [ ] Correct inventory file (`inventory.ini`) is used for both playbook and ad-hoc commands.

## Actual Terminal Output (Reference)
```text
thor@jump-host ~$ ansible-playbook -i inventory.ini create_temp_user.yml
fatal: [stapp01]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled ..."}

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

thor@jump-host ~$ ansible-playbook -i inventory.ini create_temp_user.yml
ok: [stapp01]
changed: [stapp01]

thor@jump-host ~$ ansible app_servers -i hosts.ini -m shell -a "chage -l javed" -b
[WARNING]: Unable to parse /home/thor/hosts.ini as an inventory source

thor@jump-host ~$ ansible app_servers -i inventory.ini -m shell -a "chage -l javed" -b
Account expires                                         : Feb 17, 2027
```
