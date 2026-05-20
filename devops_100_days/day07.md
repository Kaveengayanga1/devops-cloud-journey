# Day 07: Password-less SSH Access with Ansible

## Task Overview
The system admins at xFusionCorp need `thor` on `jump-host` to access all app servers without entering passwords, using each server's sudo user:
- `stapp01` as `tony`
- `stapp02` as `steve`
- `stapp03` as `banner`

This is required so scheduled scripts on the jump host can run non-interactively across app servers.

## Theory

### 1. Why Password-less SSH Is Needed
- Automation jobs (cron/scripts/Ansible) cannot reliably wait for manual password input.
- SSH key-based authentication is more secure and scalable than plain password-based SSH.
- It enables repeatable operations across multiple servers.

### 2. SSH Key Pair Basics
- **Private key (`id_rsa`)**: Stays on jump host; never shared.
- **Public key (`id_rsa.pub`)**: Copied to remote users' `~/.ssh/authorized_keys`.
- When `thor` connects, the server verifies the private/public key match and allows login without password prompts.

### 3. Why Use Ansible for This
- Manual setup (`ssh-copy-id`) works for a few servers, but Ansible scales to many hosts.
- `authorized_key` module is idempotent: running multiple times does not duplicate keys.
- Playbook-based setup is documented, repeatable, and team-friendly.

### 4. Delegation Concept in This Task
- SSH key generation must happen on jump host, not on each app server.
- `delegate_to: localhost` + `run_once: true` ensures key generation runs once on jump host.

## Infrastructure Reference
Only app servers are targeted for this task:
- `stapp01` (`tony` / `Ir0nM@n`)
- `stapp02` (`steve` / `Am3ric@`)
- `stapp03` (`banner` / `BigGr33n`)

## Implementation

### 1. Generate SSH Key on Jump Host
```bash
thor@jump-host ~$ ssh-keygen -t rsa
```
Press Enter for default file path and empty passphrase in this lab.

### 2. Create Inventory (`inventory.ini`)
```ini
[app_servers]
stapp01 ansible_user=tony ansible_password=Ir0nM@n
stapp02 ansible_user=steve ansible_password=Am3ric@
stapp03 ansible_user=banner ansible_password=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 3. Create Playbook (`setup_ssh.yml`)
```yaml
---
- name: Setup Passwordless SSH Access
  hosts: app_servers
  gather_facts: false
  tasks:
    - name: Generate SSH keypair on Jump Host (if not exists)
      delegate_to: localhost
      run_once: true
      openssh_keypair:
        path: "~/.ssh/id_rsa"
        type: rsa

    - name: Add Jump Host public key to authorized_keys
      authorized_key:
        user: "{{ ansible_user }}"
        state: present
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

### 4. Run Playbook
```bash
thor@jump-host ~$ ansible-playbook -i inventory.ini setup_ssh.yml
```

### 5. Verify Password-less Login
```bash
thor@jump-host ~$ ssh tony@stapp01
thor@jump-host ~$ ssh steve@stapp02
thor@jump-host ~$ ssh banner@stapp03
```

## Given Output (Execution Evidence)
```bash
thor@jump-host ~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/thor/.ssh/id_rsa): 
Created directory '/home/thor/.ssh'.
Enter passphrase for "/home/thor/.ssh/id_rsa" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/thor/.ssh/id_rsa
Your public key has been saved in /home/thor/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:qKFhPM1EdPIEEL1Cm1tCvzm+XjpagNOeTTsmy1ZrdRk thor@jump-host
The key's randomart image is:
+---[RSA 3072]----+
|  o==.o          |
|  o..=           |
| o +...          |
| +=++  .E        |
|o B==o. So       |
| +.O=+. o        |
|  =oB+..         |
| ..==+           |
| .+++o           |
+----[SHA256]-----+
thor@jump-host ~$ vi inventory.ini
thor@jump-host ~$ cat inventory.ini 
[app_servers]
stapp01 ansible_user=tony ansible_password=Ir0nM@n
stapp02 ansible_user=steve ansible_password=Am3ric@
stapp03 ansible_user=banner ansible_password=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
thor@jump-host ~$ vi setup_ssh.yml
thor@jump-host ~$ ansible-playbook -i inventory.ini setup_ssh.yml 

PLAY [Setup Passwordless SSH Access] *************************************************************************

TASK [Generate SSH keypair on Jump Host (if not exists)] *****************************************************
changed: [stapp01 -> localhost]

TASK [Add Jump Host public key to authorized_keys] ***********************************************************
changed: [stapp03]
changed: [stapp02]
changed: [stapp01]

PLAY RECAP ***************************************************************************************************
stapp01                    : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp02                    : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp03                    : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

thor@jump-host ~$ ssh tony@stapp01
Last login: Sun Mar 22 03:59:58 2026 from 10.244.29.206
[tony@stapp01 ~]$ exit
logout
Connection to stapp01 closed.
thor@jump-host ~$ ssh steve@stapp02
Last login: Sun Mar 22 03:59:58 2026 from 10.244.29.206
[steve@stapp02 ~]$ exit
logout
Connection to stapp02 closed.
thor@jump-host ~$ ssh banner@stapp03
Last login: Sun Mar 22 03:59:58 2026 from 10.244.29.206
[banner@stapp03 ~]$ exit
logout
Connection to stapp03 closed.
thor@jump-host ~$
```

## Output Analysis
- `changed: [stapp01 -> localhost]` confirms key generation happened on jump host via delegation.
- `changed` on all app servers confirms public key insertion to each user's `authorized_keys`.
- `failed=0` and `unreachable=0` confirms successful automation.
- Direct SSH verification without password confirms the final requirement is fully met.

## Professional DevOps Best Practices
- Replace plain-text `ansible_password` with Ansible Vault in real environments.
- After successful key-based setup, remove password variables from inventory.
- Keep secure permissions:
  - `~/.ssh` -> `700`
  - `~/.ssh/authorized_keys` -> `600`
- Prefer dedicated automation keys over personal keys in production.
- Keep playbooks idempotent and rerunnable for drift correction.

## Conclusion
Password-less SSH from `thor@jump-host` to all app servers through their sudo users was set up successfully using Ansible, and verified by direct SSH logins.
