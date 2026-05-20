# Day 06: Ansible Cron Job Deployment

## Task Overview
The goal was to automate the deployment of a specific cron job to all app servers in the Stratos DC.
**Cron Job Details:**
- **Schedule:** `*/5 * * * *` (Every 5 minutes)
- **Command:** `echo hello > /tmp/cron_text`
- **User:** `root`

## Theory
### 1. Cron and Cronie
- **Cron Job:** A time-based job scheduler in Unix-like computer operating systems. Users that set up and maintain software environments use cron to schedule jobs (commands or shell scripts) to run periodically at fixed times, dates, or intervals.
- **Cronie:** The standard package in Red Hat/CentOS distributions that provides the cron daemon (`crond`).
- **Syntax:** `Minute Hour Day Month Weekday Command`
    - `*/5` in the minute field means "every 5 minutes".

### 2. Ansible Concepts
- **Inventory:** A file defining the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. It can also hold variables like SSH passwords.
- **Playbook:** A YAML file containing a list of tasks to be executed on the managed nodes.
- **Modules Used:**
    - `yum`: To install packages (e.g., `cronie`).
    - `service`: To manage services (start and enable `crond`).
    - `cron`: To manage crontab entries efficiently.
- **Idempotency:** The property that executing an operation multiple times has the same effect as executing it once. Ansible ensures that if a package is already installed or a service is already running, it won't try to accept the change again, reporting "ok" instead of "changed".

## Execution Steps & Output

### 1. Inventory Setup (`inventory`)
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_become_pass=BigGr33n

[all:vars]
ansible_become_pass=Ir0nM@n
```

### 2. Playbook Creation (`deployment.yml`)
```yaml
---
- name: Deploy Cron Job on Nautilus App Servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Install cronie package
      yum:
        name: cronie
        state: present

    - name: Ensure crond service is started and enabled
      service:
        name: crond
        state: started
        enabled: yes

    - name: Add cron job for root user
      ansible.builtin.cron:
        name: "test cron job"
        user: root
        minute: "*/5"
        job: "echo hello > /tmp/cron_text"
```

### 3. Execution Logs

**First Attempt (Failed):**
```bash
thor@jump-host ~$ ansible-playbook -i inventory deployment.yml
...
fatal: [stapp02]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled..."}
fatal: [stapp03]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled..."}
```

**Second Attempt (Success with Host Key Check Disabled):**
```bash
thor@jump-host ~$ ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory deployment.yml
...
PLAY RECAP ***************************************************************************************************
stapp01: ok=4    changed=0    unreachable=0    failed=0
stapp02: ok=4    changed=3    unreachable=0    failed=0
stapp03: ok=4    changed=3    unreachable=0    failed=0
```

**Verification:**
```bash
thor@jump-host ~$ ANSIBLE_HOST_KEY_CHECKING=False ansible -i inventory app_servers -m shell -a "crontab -l" -b
stapp02 | CHANGED | rc=0 >>
#Ansible: test cron job
*/5 * * * * echo hello > /tmp/cron_text
...
```

## Errors and Reasons

### Error 1: Host Key Checking
**Error Message:**
> `Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.`

**Reason:**
When connecting to a new SSH server, the client is prompted to accept the host's fingerprint ("Are you sure you want to continue connecting (yes/no)?"). Ansible uses `sshpass` for password authentication, which is non-interactive and cannot handle this prompt, causing the connection to fail for servers (`stapp02`, `stapp03`) that hadn't been accessed manually before.

**Solution:**
Disable host key checking temporarily using the environment variable:
`ANSIBLE_HOST_KEY_CHECKING=False`

---

### Error 2: Sudo Password Requirement
**Error Message:**
> `sudo: a password is required` / `sudo: timed out reading password`

**Reason:**
This occurred during verification when running an ad-hoc command like `ansible ... -a "sudo crontab -l"`.
When `sudo` is included inside the command string, it expects an interactive TTY to input the password. Ansible runs commands non-interactively.

**Solution:**
Remove `sudo` from the command string and use the Ansible `-b` (become) flag. This tells Ansible to use the configured `ansible_become_pass` to handle privilege escalation properly.
Correct command: `ansible ... -a "crontab -l" -b`

---

### Error 3: File Not Found (Timing Issue)
**Error Message:**
> `ls: cannot access '/tmp/cron_text': No such file or directory`

**Reason:**
The cron job was configured to run every 5 minutes (`*/5`). The verification check (looking for the output file) was run immediately after the playbook finished. Since the next 5-minute interval (e.g., :00, :05, :10) hadn't occurred yet, the cron job hadn't executed its first run, so the file didn't exist.

**Solution:**
Wait for the next 5-minute interval to pass before checking for the output file.
