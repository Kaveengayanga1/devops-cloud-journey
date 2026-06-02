# Day 06: Disable Direct Root SSH Login

## Theory
Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including the restriction of direct root SSH login. Disabling direct SSH root login on servers is a crucial security measure. Leaving it enabled increases the risk of brute-force attacks. By disabling it, users must first log in with their personal accounts and then switch to root (using `sudo` or `su`), which improves accountability and security.

### Infrastructure Details
| Server Name | Hostname | User | Password | Purpose |
| ----------- | -------- | ---- | -------- | ------- |
| Application Server 1 | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| Application Server 3 | stapp03 | banner | BigGr33n | Hosts Nautilus Application 3 |

## Task
Your task is to disable direct SSH root login on all app servers within the Stratos Datacenter using an Ansible playbook.

## Steps

### 1. Create the Inventory File (`inventory.ini`)
Create an inventory file with the connection details and credentials for the app servers. This includes the variables to bypass strict host key checking.

```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password=BigGr33n ansible_become_pass=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### 2. Create the Ansible Playbook (`disable_root_login.yml`)
Write a playbook to modify the `/etc/ssh/sshd_config` file and restart the SSH service so the changes take effect.

```yaml
---
- name: Disable Direct Root SSH Login on App Servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Ensure PermitRootLogin is set to no
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'  
        line: 'PermitRootLogin no'
        state: present
      notify: Restart SSH service  

  handlers:
    - name: Restart SSH service
      service:
        name: sshd
        state: restarted
```

### 3. Execute the Playbook
Run the playbook using the `ansible-playbook` command:
```bash
ansible-playbook -i inventory.ini disable_root_login.yml
```

### 4. Verification
Verify that root login is blocked by trying to SSH directly as the `root` user. It should fail and show a "Permission denied" message:
```bash
ssh root@stapp01
# Expected output: Permission denied, please try again.
```
You can also verify by logging in as a normal user and checking the file:
```bash
ssh tony@stapp01
sudo sshd -T | grep permitrootlogin
# Expected output: permitrootlogin no
```
