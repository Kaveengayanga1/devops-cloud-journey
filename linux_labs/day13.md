# Restrict Crontab Access - Linux Cron

## Scenario
In alignment with security compliance standards, the Nautilus project team has opted to impose restrictions on crontab access. Specifically, only designated users will be permitted to create or update cron jobs.

### Task
Configure crontab access on App Server 1 (`stapp01`) as follows: 
Allow crontab access to `yousuf` user while denying access to the `garrett` user.

### Infrastructure Details
* **Source Server**: Jump Host (`jump-host`)
* **Target Server**: App Server 1 (`stapp01`)
* **Target User**: `tony`
* **Target Password**: `Ir0nM@n`

## Theory Knowledge
To control who can use the `crontab` command in Linux, two main files are used:
* `/etc/cron.allow`: If this file exists, only the users listed in it are allowed to use `crontab`. The `cron.deny` file is ignored.
* `/etc/cron.deny`: If `cron.allow` does not exist, the system checks this file. Any user not listed here is allowed to use `crontab`.
* If neither file exists, depending on system configuration, either all users or only the `root` user can use `crontab`.

*(Note: Per the task's rigid requirements, we will explicitly populate both files).*

## Step-by-Step Guide

**Step 1: Connect to Jump Host**
Ensure you are logged into the Jump Host.
```bash
ssh thor@jump-host
# Password: mjolnir123
```

**Step 2: Connect to App Server 1**
SSH into `stapp01` from the Jump Host.
```bash
ssh tony@stapp01
# Password: Ir0nM@n
```

**Step 3: Switch to Root**
You need root privileges to modify the cron control files.
```bash
sudo -i
# Password: Ir0nM@n
```

**Step 4: Configure `cron.allow`**
Allow the user `yousuf` by adding their name to `/etc/cron.allow`.
```bash
echo "yousuf" > /etc/cron.allow
```

**Step 5: Configure `cron.deny`**
Deny the user `garrett` by adding their name to `/etc/cron.deny`.
```bash
echo "garrett" > /etc/cron.deny
```

**Step 6: Verify Changes**
Check the contents of both files to ensure they were updated correctly.
```bash
cat /etc/cron.allow
cat /etc/cron.deny
```

## Output
```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.97.191)' can't be established.
ED25519 key fingerprint is SHA256:2IdhCpbnAivqD/t1RuM9bEc8DZsfoBUuz+NfFCBbERM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 
[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ sudo -i

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
[root@stapp01 ~]# echo "yousuf" > /etc/cron.allow
[root@stapp01 ~]# echo "garrett" > /etc/cron.deny
[root@stapp01 ~]# cat /etc/cron.allow
yousuf
[root@stapp01 ~]# cat /etc/cron.deny
garrett
[root@stapp01 ~]# 
```
