```markdown
# Day 14: System Configuration and Management

## Common Configuration

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

---

## Task 1: Wait and Switch to the Correct Runlevel

### Theory
In systemd-based systems, the concept of runlevels has been replaced with targets. The graphical.target is the equivalent of the traditional runlevel 5, which starts the system with a graphical user interface. It's important to ensure that the system is set to boot into the correct target, especially for servers that require a GUI for specific applications.

**Task:** Verify the current target and switch to graphical.target if it is not already set.

### Professional DevOps Approach
A professional DevOps engineer knows that directly using `systemctl set-default graphical.target` might not be the best approach if there are existing customizations or if the system is not in a state to change the default target. Instead, checking the current state and switching targets carefully is essential.

### `set_runlevel.yml`
The Ansible playbook used to verify and switch the runlevel (target) to graphical.target:
```yaml
---
- name: Verify and Switch to the Correct Runlevel
  hosts: app_servers
  become: yes
  tasks:
    - name: Check current default target
      command: systemctl get-default
      register: current_target

    - name: Set default target to graphical.target
      command: systemctl set-default graphical.target
      when: current_target.stdout != "graphical.target"
      notify: reboot

    - name: Verify the change
      command: systemctl get-default
      register: new_target
      when: current_target.stdout != "graphical.target"

    - name: Display current status
      debug:
        msg: "Default target has been changed to {{ new_target.stdout | default('graphical.target') }}"
      when: new_target.changed

  handlers:
    - name: reboot
      reboot:
```

### Execution and Output

```bash
thor@jump-host ~$ ansible -i inventory.ini app_servers -a "systemctl get-default"
stapp02 | SUCCESS | rc=0 >>
multi-user.target
stapp03 | SUCCESS | rc=0 >>
multi-user.target
stapp01 | SUCCESS | rc=0 >>
multi-user.target
```

*(Note: The current default target is `multi-user.target`, which is equivalent to the traditional runlevel 3.)*

---

## Task 2: Synchronize Timezone Across Application Servers

### Theory
In the daily standup, it was noted that the timezone settings across the Nautilus Application Servers in the Stratos Datacenter are inconsistent with the local datacenter's timezone, currently set to `America/Los_Angeles`. Over the lifecycle of systems management setting the correct timezone helps to maintain logs consistency, reliable execution of chron jobs and accurate database transaction timestamps. 

**Task:** Synchronize the timezone settings to match the local datacenter's timezone (`America/Los_Angeles`).

### Professional DevOps Approach
Instead of ssh'ing into each individual server and running `sudo timedatectl set-timezone America/Los_Angeles`, a professional DevOps engineer uses configuration management systems like Ansible. This guarantees consistency and idempotency across all servers simultaneously.

### `set_timzone.yml`
The Ansible playbook used to synchronize the timezones (with the fixed syntax avoiding `if` condition task errors and variable access errors):
```yaml
---
- name: Synchronize Timezone Across Application Servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Set timezone to America/Los_Angeles
      community.general.timezone:
        name: America/Los_Angeles
      register: tz_status

    - name: Display current status
      debug:
        msg: "Timezone has been changed to {{ tz_status.name | default('America/Los_Angeles') }}"
      when: tz_status.changed

    - name: Restart Crond to apply changes (Optional but Recommended)
      ansible.builtin.service:
        name: crond
        state: restarted
      ignore_errors: yes
```

### Execution and Output

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.97.141)' can't be established.
ED25519 key fingerprint is SHA256:afVaEjHBytYeMI0PToBQ1m1bHe89s+iQhIz0/qkfWT8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 
[tony@stapp01 ~]$ timedatectl
               Local time: Sun 2026-03-29 02:53:11 UTC
           Universal time: Sun 2026-03-29 02:53:11 UTC
                 RTC time: n/a
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: n/a
          RTC in local TZ: no
[tony@stapp01 ~]$ exit
logout
Connection to stapp01 closed.

thor@jump-host ~$ vi inventory.ini
thor@jump-host ~$ cat inventory.ini 
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_password=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_password=BigGr33n ansible_become_pass=BigGr33n

[app_servers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

thor@jump-host ~$ vi set_timzone.yml 
thor@jump-host ~$ ansible-playbook -i inventory.ini set_timzone.yml 

PLAY [Synchronize Timezone Across Application Servers] *********************************************************************

TASK [Gathering Facts] *****************************************************************************************************
ok: [stapp03]
ok: [stapp01]
ok: [stapp02]

TASK [Set timezone to America/Los_Angeles] *********************************************************************************
ok: [stapp01]
ok: [stapp03]
ok: [stapp02]

TASK [Display current status] **********************************************************************************************
skipping: [stapp01]
skipping: [stapp02]
skipping: [stapp03]

TASK [Restart Crond to apply changes] **************************************************************************************
fatal: [stapp01]: FAILED! => {"changed": false, "msg": "Could not find the requested service crond: host"}
...ignoring
fatal: [stapp02]: FAILED! => {"changed": false, "msg": "Could not find the requested service crond: host"}
...ignoring
fatal: [stapp03]: FAILED! => {"changed": false, "msg": "Could not find the requested service crond: host"}
...ignoring

PLAY RECAP *****************************************************************************************************************
stapp01                    : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=1   
stapp02                    : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=1   
stapp03                    : ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=1   

thor@jump-host ~$ 
```