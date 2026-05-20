# Day 08: Install Ansible 4.8.0 on Jump Host (Global Access)

## Task Description
The Nautilus DevOps team selected Ansible as their current automation and configuration management tool because it is agentless, simple to set up, and has minimal prerequisites.

For initial testing, the team uses `jump-host` as the Ansible controller node.

Your task is to install **Ansible version `4.8.0`** on `jump-host` using **`pip3` only**, and ensure the `ansible` binary is globally available so **all users** on the system can run Ansible commands.

---

## Requirements
- Install Ansible with `pip3` (not `apt`/`yum`).
- Install the exact version: `ansible==4.8.0`.
- Perform installation system-wide so all users can access the binary.
- Validate installation with `ansible --version`.
- Ensure executable path is global (for example `/usr/local/bin/ansible`).

---

## Core Theory

### 1) Why Jump Host is the Ansible Controller
A jump host (bastion) is a central machine used to access other infrastructure nodes securely. Since Ansible is agentless and relies primarily on SSH, installing Ansible on one control node is enough to manage many servers.

### 2) Why Use `pip3` for This Task
Linux repositories often provide older package versions. `pip3` allows strict version pinning (`ansible==4.8.0`), which is important for lab consistency and predictable behavior.

### 3) Why `sudo pip3 install` for Global Availability
- User-level pip installs usually place binaries under a user-local path (such as `~/.local/bin`).
- System-wide install using `sudo` places Ansible in global locations (commonly `/usr/local/bin`).
- This satisfies the requirement that all users on the machine can run `ansible`.

### 4) Important Operational Note
`pip` warns against running as root because it can conflict with system packages. In production, virtual environments are preferred. However, this specific task explicitly requires global installation and availability for all users.

---

## Step-by-Step Execution (DevOps Engineer Flow)

### Step 1: Confirm Ansible is Not Yet Installed
```bash
thor@jump-host ~$ ansible
bash: ansible: command not found
```

### Step 2: Verify `pip3` is Available
```bash
thor@jump-host ~$ pip3 --version
pip 21.3.1 from /usr/lib/python3.9/site-packages/pip (python 3.9)
```

### Step 3: Install Exact Ansible Version Globally
```bash
thor@jump-host ~$ sudo pip3 install ansible==4.8.0
```

This installs:
- `ansible-4.8.0`
- `ansible-core-2.11.12`
- required Python dependencies (`jinja2`, `PyYAML`, `cryptography`, etc.)

### Step 4: Verify Installation and Version
```bash
thor@jump-host ~$ ansible --version
ansible [core 2.11.12]
```

Key validation points from output:
- Ansible package installed successfully.
- `executable location = /usr/local/bin/ansible` (global path).
- Python and dependency stack loaded correctly.

### Step 5 (Recommended): Validate as Another User
Optional but recommended in real operations:
```bash
sudo su - another_user
ansible --version
exit
```
If this works, global accessibility requirement is fully confirmed.

---

## Given Output (Execution Evidence)
```bash
thor@jump-host ~$ ansible
bash: ansible: command not found
thor@jump-host ~$ pip3 --version
pip 21.3.1 from /usr/lib/python3.9/site-packages/pip (python 3.9)
thor@jump-host ~$ sudo pip3 install ansible==4.8.0
Collecting ansible==4.8.0
  Downloading ansible-4.8.0.tar.gz (36.1 MB)
     |████████████████████████████████| 36.1 MB 8.6 MB/s            
  Preparing metadata (setup.py) ... done
Collecting ansible-core<2.12,>=2.11.6
  Downloading ansible-core-2.11.12.tar.gz (7.1 MB)
     |████████████████████████████████| 7.1 MB 125.2 MB/s            
  Preparing metadata (setup.py) ... done
Collecting jinja2
  Downloading jinja2-3.1.6-py3-none-any.whl (134 kB)
     |████████████████████████████████| 134 kB 145.5 MB/s            
Collecting PyYAML
  Downloading pyyaml-6.0.3-cp39-cp39-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (750 kB)
     |████████████████████████████████| 750 kB 111.9 MB/s            
Collecting cryptography
  Downloading cryptography-46.0.5-cp38-abi3-manylinux_2_34_x86_64.whl (4.5 MB)
     |████████████████████████████████| 4.5 MB 139.9 MB/s            
Collecting packaging
  Downloading packaging-26.0-py3-none-any.whl (74 kB)
     |████████████████████████████████| 74 kB 32.3 MB/s             
Collecting resolvelib<0.6.0,>=0.5.3
  Downloading resolvelib-0.5.4-py2.py3-none-any.whl (12 kB)
Collecting cffi>=2.0.0
  Downloading cffi-2.0.0-cp39-cp39-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (216 kB)
     |████████████████████████████████| 216 kB 112.7 MB/s            
Requirement already satisfied: typing-extensions>=4.13.2 in /usr/local/lib/python3.9/site-packages (from cryptography->ansible-core<2.12,>=2.11.6->ansible==4.8.0) (4.15.0)
Collecting MarkupSafe>=2.0
  Downloading markupsafe-3.0.3-cp39-cp39-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (20 kB)
Collecting pycparser
  Downloading pycparser-2.23-py3-none-any.whl (118 kB)
     |████████████████████████████████| 118 kB 99.6 MB/s            
Using legacy 'setup.py install' for ansible, since package 'wheel' is not installed.
Using legacy 'setup.py install' for ansible-core, since package 'wheel' is not installed.
Installing collected packages: pycparser, MarkupSafe, cffi, resolvelib, PyYAML, packaging, jinja2, cryptography, ansible-core, ansible
    Running setup.py install for ansible-core ... done
    Running setup.py install for ansible ... done
Successfully installed MarkupSafe-3.0.3 PyYAML-6.0.3 ansible-4.8.0 ansible-core-2.11.12 cffi-2.0.0 cryptography-46.0.5 jinja2-3.1.6 packaging-26.0 pycparser-2.23 resolvelib-0.5.4
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
thor@jump-host ~$ ansible --version
ansible [core 2.11.12] 
  config file = None
  configured module search path = ['/home/thor/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/thor/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.19 (main, Jun 11 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)]
  jinja version = 3.1.6
  libyaml = True
thor@jump-host ~$ 
```

---

## Validation Checklist
- [x] `ansible` command was initially missing.
- [x] `pip3` was present and usable.
- [x] `sudo pip3 install ansible==4.8.0` completed successfully.
- [x] `ansible --version` shows installed version details.
- [x] Binary path shows global executable location (`/usr/local/bin/ansible`).
- [ ] Optional: Verified from a second user account.

---

## Best Practices Note (Real-World)
- Prefer virtual environments in production for Python tool isolation.
- For strict enterprise setups, track package install via internal artifact repositories.
- Record exact tested versions (`ansible`, `ansible-core`, Python) in team documentation for reproducible automation environments.
