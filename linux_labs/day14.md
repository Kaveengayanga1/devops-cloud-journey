# Day 14: Configure App Servers to Boot into GUI by Default

## Theory
With the installation of new tools on the app servers within the Stratos Datacenter, certain functionalities now necessitate graphical user interface (GUI) access.

Linux systems manage the execution state using Runlevels (in SystemV init) or Targets (in modern systemd).
- **multi-user.target (Runlevel 3):** Standard CLI (Command Line Interface) mode typically used for servers to save system resources.
- **graphical.target (Runlevel 5):** GUI mode with network services.

**Task:** Adjust the default runlevel on all App servers in Stratos Datacenter to enable GUI booting by default. It's imperative not to initiate a server reboot after completing this task.

### Infrastructure Details
| Server Name | IP | Hostname | User | Password | Purpose |
|---|---|---|---|---|---|
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | Dynamic | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| Application Server 3 | Dynamic | stapp03 | banner | BigGr33n | Hosts Nautilus Application 3 |

### Professional DevOps Approach (Using Ansible)
To ensure efficiency and idempotency, we use Ansible instead of logging into servers manually.
1. Use `systemctl set-default graphical.target` to permanently change the default runlevel.
2. Use `systemctl isolate graphical.target` to activate the GUI mode immediately without requiring a reboot.

### `inventory.ini`
Since the servers are no longer available for direct testing, you must create and use the following `inventory.ini` file for Ansible:
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password=BigGr33n ansible_become_pass=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### `set_gui.yml`
The Ansible playbook used to make the changes:
```yaml
---
- name: Configure and Start GUI on App Servers without Reboot
  hosts: app_servers
  become: yes
  tasks:
    - name: Set default boot target to graphical.target (Permanent change)
      command: systemctl set-default graphical.target
      register: set_default_output
      changed_when: "'set-default' in set_default_output.stdout or set_default_output.rc == 0"

    - name: Start GUI immediately in the current session (No Reboot)
      command: systemctl isolate graphical.target
      async: 60 
      poll: 0  
      tags: start_gui

    - name: Verify the change
      command: systemctl get-default
      register: current_target
      changed_when: false

    - name: Show status message
      debug:
        msg: "Current default boot target is now {{ current_target.stdout }}. GUI has been triggered."
```

## Execution and Output

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.240.182)' can't be established.
ED25519 key fingerprint is SHA256:XY9JnlYYXcmDTrWxtxDWXC3lVo1m7+ynjv82Wv+YpWo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 
Permission denied, please try again.
tony@stapp01's password: 
Last failed login: Sun Mar 29 02:27:59 UTC 2026 from 10.244.240.137 on ssh:notty
There was 1 failed login attempt since the last successful login.
[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ systemctl get-default
multi-user.target
[tony@stapp01 ~]$ exit
logout
Connection to stapp01 closed.

thor@jump-host ~$ vi inventory.ini
thor@jump-host ~$ vi set_gui.yml

thor@jump-host ~$ cat inventory.ini 
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password=BigGr33n ansible_become_pass=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

thor@jump-host ~$ cat set_gui.yml 
---
- name: Configure and Start GUI on App Servers without Reboot
  hosts: app_servers
  become: yes
  tasks:
    - name: Set default boot target to graphical.target (Permanent change)
      command: systemctl set-default graphical.target
      register: set_default_output
      changed_when: "'set-default' in set_default_output.stdout or set_default_output.rc == 0"

    - name: Start GUI immediately in the current session (No Reboot)
      command: systemctl isolate graphical.target
      async: 60 
      poll: 0  
      tags: start_gui

    - name: Verify the change
      command: systemctl get-default
      register: current_target
      changed_when: false

    - name: Show status message
      debug:
        msg: "Current default boot target is now {{ current_target.stdout }}. GUI has been triggered."

thor@jump-host ~$ ansible-playbook -i inventory.ini set_gui.yml 

PLAY [Configure and Start GUI on App Servers without Reboot] **********************************************************************

TASK [Gathering Facts] ************************************************************************************************************
ok: [stapp02]
ok: [stapp03]
ok: [stapp01]

TASK [Set default boot target to graphical.target (Permanent change)] *************************************************************
changed: [stapp03]
changed: [stapp01]
changed: [stapp02]

TASK [Start GUI immediately in the current session (No Reboot)] *******************************************************************
changed: [stapp02]
changed: [stapp03]
changed: [stapp01]

TASK [Verify the change] **********************************************************************************************************
ok: [stapp03]
ok: [stapp02]
ok: [stapp01]

TASK [Show status message] ********************************************************************************************************
ok: [stapp01] => {
    "msg": "Current default boot target is now graphical.target. GUI has been triggered."
}
ok: [stapp02] => {
    "msg": "Current default boot target is now graphical.target. GUI has been triggered."
}
ok: [stapp03] => {
    "msg": "Current default boot target is now graphical.target. GUI has been triggered."
}

PLAY RECAP ************************************************************************************************************************
stapp01                    : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp02                    : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
stapp03                    : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

thor@jump-host ~$ ansible -i inventory.ini app_servers -a "systemctl is-active graphical.target"
stapp03 | CHANGED | rc=0 >>
active
stapp01 | CHANGED | rc=0 >>
active
stapp02 | CHANGED | rc=0 >>
active

thor@jump-host ~$ ansible -i inventory.ini app_servers -a "systemctl status gdm"
stapp02 | FAILED | rc=4 >>
Unit gdm.service could not be found.non-zero return code
stapp03 | FAILED | rc=4 >>
Unit gdm.service could not be found.non-zero return code
stapp01 | FAILED | rc=4 >>
Unit gdm.service could not be found.non-zero return code
```

*(Note: Although `gdm.service` was not found which throws an error when checking status, returning `active` for `systemctl is-active graphical.target` confirms that the runlevel was successfully switched in systemd.)*
