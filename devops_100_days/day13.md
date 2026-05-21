# Day 13: Configure Iptables Using Ansible

## Task Description
The security team has raised a concern that Apache’s port `5004` is open for all since there is no firewall installed on the app hosts. The requirements are:
1. Install `iptables` and all its dependencies on each app host.
2. Block incoming port `5004` on all apps for everyone except for the LBR host.
3. Make sure the rules remain even after a system reboot.

## Infrastructure Details

| Server Name | IP | Hostname | User | Password | Purpose |
| --- | --- | --- | --- | --- | --- |
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | Dynamic | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| Application Server 3 | Dynamic | stapp03 | banner | BigGr33n | Hosts Nautilus Application 3 |
| LoadBalancer Server | Dynamic | stlb01 | loki | Mischi3f | Distributes traffic for Nautilus HTTP |
| Database Server | Dynamic | stdb01 | peter | Sp!dy | Hosts Nautilus Database |
| Storage Server | Dynamic | ststor01 | natasha | Bl@kW | Stores data for Nautilus Servers |
| Backup Server | Dynamic | stbkp01 | clint | H@wk3y3 | Manages backups for Nautilus Servers |
| Mail Server | Dynamic | stmail01 | groot | Gr00T123 | Manages email services for Nautilus Servers |
| Jump Host Server | Dynamic | jump-host | thor | mjolnir123 | Provides secure access to Stork DC |
| Jenkins Server | Dynamic | jenkins | jenkins | j@rv!s | Runs Jenkins for CI/CD pipeline |

## Mistakes Made and How They Were Fixed

### 1. YAML Syntax Error (`if` instead of `when`)
- **Mistake:** Used the `if` keyword to conditionally execute a task.
- **Why it happened:** Assuming standard programming language logic inside an Ansible playbook. 
- **Fix:** Ansible uses the `when` condition. Replaced `if:` with `when:`.

### 2. Missing Executable `iptables`
- **Mistake:** Installed only `iptables-services`, which left the system without the core `iptables` binary.
- **Why it happened:** On CentOS/RHEL, sometimes `iptables` and `iptables-services` are separate packages. Ansible's `iptables` module requires the `iptables` binary to be present.
- **Fix:** Specified both `iptables` and `iptables-services` in the `package` module to ensure both are installed.

### 3. Iptables Service Execution Failure
- **Mistake:** Tried to start the `iptables` service before saving any firewall configurations.
- **Why it happened:** The `iptables` service fails to start if `/etc/sysconfig/iptables` does not exist or lacks basic filter tables.
- **Fix:** Restructured the playbook to first push the rules directly into memory using the ansible `iptables` module, then save them using an `iptables-save > /etc/sysconfig/iptables` command, and ONLY THEN start and enable the `iptables` service. This completely bypassed the requirement for a pre-existing correctly formatted configuration file.

## Final Solution (Playbook)

### `inventory.ini`
```ini
[app_servers]
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n ansible_become_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@ ansible_become_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n ansible_become_pass=BigGr33n

[lbr]
stlb01 ansible_host=stlb01 ansible_user=loki ansible_ssh_pass=Mischi3f ansible_become_pass=Mischi3f

[all:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

### `setup_firewall.yml`
```yaml
---
- name: Configure iptables for Nautilus Infrastructure
  hosts: app_servers
  become: yes
  tasks:
    - name: Install iptables and iptables-services
      package:
        name:
          - iptables
          - iptables-services
        state: present

    - name: Allow incoming port 5004 from LBR host
      iptables:
        chain: INPUT
        protocol: tcp
        source: "{{ hostvars['stlb01']['ansible_host'] }}"
        destination_port: "5004"
        jump: ACCEPT

    - name: Block incoming port 5004 for everyone else
      iptables:
        chain: INPUT
        protocol: tcp
        destination_port: "5004"
        jump: DROP

    - name: Save iptables rules to file
      shell: iptables-save > /etc/sysconfig/iptables

    - name: Enable and start iptables service
      service:
        name: iptables
        state: started
        enabled: yes
```

### Execution
Run the specific playbook with inventory limits:
```bash
ansible-playbook -i inventory.ini setup_firewall.yml
```

### Verification (from any target server)
Login internally via SSH and verify settings:
```bash
sudo iptables -L INPUT -n --line-numbers
sudo cat /etc/sysconfig/iptables
sudo systemctl status iptables
```

## Theory

- **Iptables vs Iptables-services**: `iptables` is the package that provides the core binary to manipulate firewall rules in the Linux kernel. `iptables-services` is the package that provides the `systemd` service to ensure these rules automatically reload after a system reboot.
- **Order of Precedence**: Iptables processes rules top-to-bottom. You must explicitly `ACCEPT` traffic from specific sources before executing generic `DROP` or `REJECT` commands. 
- **Persistence (`The Save Command`)**: Iptables rules sit natively in system memory. If you reboot, all changes are wiped. You must save to `/etc/sysconfig/iptables` so the OS brings the firewall settings back up automatically.
- **Ansible Execution Context**: Utilizing variables like `{{ hostvars['stlb01']['ansible_host'] }}` ensures you maintain a clean dynamic logic where Ansible resolves the IP based off the `[lbr]` inventory entries dynamically rather than hardcoding static IPs that might change.
