# Day 13: Configure Root SSH Access on Azure VM

## Cloud Theory

### SSH Public Key Authentication
SSH uses a pair of keys: a private key (kept secret on your local machine) and a public key (placed on the remote server). When you connect, the server verifies your identity using these keys, allowing for password-less login.

### Root User Access in Azure
By default, Azure Linux VMs disable direct login as the `root` user for security reasons. To enable it, you typically need to:
1.  Add your public key to the root user's `authorized_keys` file.
2.  Modify the SSH daemon configuration (`sshd_config`) to allow root login.
3.  Remove any Azure-specific restrictions (forced commands) from the `authorized_keys` file.

### File Permissions
Secure permissions are critical for SSH to work:
*   `.ssh` directory: `700` (rwx------) - Only owner can read/write/execute.
*   `authorized_keys` file: `600` (rw-------) - Only owner can read/write.

## Task
Set up secure SSH access for the `root` user on the `devops-vm` Azure VM. You need to verify you can SSH as root from the Azure Client host without a password.

## Steps

1.  **Retrieve Public Key**: View the content of the public key on the client host (`/root/.ssh/id_rsa.pub`).
2.  **Get VM IP**: Find the public IP address of `devops-vm`.
3.  **Login as Azure User**: SSH into the VM using the default `azureuser` account using the IP.
4.  **Configure Root Access**:
    *   Switch to the root user (`sudo su -`).
    *   Create the `/root/.ssh` directory and set permissions to `700`.
    *   Add the client's public key to `/root/.ssh/authorized_keys` and set permissions to `600`.
5.  **Enable Root Login**:
    *   Edit `/etc/ssh/sshd_config` to ensure `PermitRootLogin yes` is set (change from `prohibit-password` if necessary).
    *   Remove default Azure restrictions in `authorized_keys` (lines starting with `command="..."`).
    *   Restart the SSH service (`systemctl restart ssh`).
6.  **Verify**: Log out of the VM and verify you can SSH directly as `root` from the Azure Client.

## Execution & Output

```bash
~ ➜  ls

~ ➜  cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCeUJ0QjY32q/SvPV+We0TT365y0YGIOX5ps1EO2hcCMCUcstKMdIZgPGaa+PPKmoUIER6RPNi4An+l2tSLR5qHy+FtabeqHalQciOYaIpDIc4huno9DPDcP/+7W09G6MvoMp6wvovhr+0gI3zd49nFpwP5NtRvjuy8xLKBFnUYyRlhr7OpKt2YCWfs3qUSsB2sq/1SsT6Mn5iXZHerlQPQpfU1w0aDPKRm1aeLdgwJmp5mhJ5n2pY8hPB8imfq42MnWAGWKhj7rS4TrmK4WvXBKZc5kOPi3lYBfxnmAbYDWWVqnehEfNsq82tprsef7Z8fEJh4qyYv4V7bzoFwpHxz root@azure-client

~ ➜  az group list --output table
Name                          Location    Status
----------------------------  ----------  ---------
kml_rg_main-e416f16dfa674eef  eastus      Succeeded

~ ➜  az vm list-ip-addresses -g kml_rg_main-e416f16dfa674eef -n devops-vm --output table
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
devops-vm         20.85.210.242        10.0.0.4

~ ➜  ssh azureuser@20.85.210.242
The authenticity of host '20.85.210.242 (20.85.210.242)' can't be established.
ECDSA key fingerprint is SHA256:Ci5Ay7vc/WFCkQSzxVMc6I0o5kSQhTrQmtcKpuQKTWA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.85.210.242' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun Mar  8 02:04:11 UTC 2026

  System load:  0.09              Processes:             107
  Usage of /:   5.5% of 28.89GB   Users logged in:       0
  Memory usage: 30%               IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@devops-vm:~$ sudo su -
root@devops-vm:~# mkdir -p /root/.ssh
root@devops-vm:~# chmod 700 /root/.ssh
root@devops-vm:~# echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCeUJ0QjY32q/SvPV+We0TT365y0YGIOX5ps1EO2hcCMCUcstKMdIZgPGaa+PPKmoUIER6RPNi4An+l2tSLR5qHy+FtabeqHalQciOYaIpDIc4huno9DPDcP/+7W09G6MvoMp6wvovhr+0gI3zd49nFpwP5NtRvjuy8xLKBFnUYyRlhr7OpKt2YCWfs3qUSsB2sq/1SsT6Mn5iXZHerlQPQpfU1w0aDPKRm1aeLdgwJmp5mhJ5n2pY8hPB8imfq42MnWAGWKhj7rS4TrmK4WvXBKZc5kOPi3lYBfxnmAbYDWWVqnehEfNsq82tprsef7Z8fEJh4qyYv4V7bzoFwpHxz root@azure-client >> /root/.ssh/authorized_keys 
root@devops-vm:~# chmod 600 /root/.ssh/authorized_keys
root@devops-vm:~# PUB_KEY=$(cat /root/.ssh/id_rsa.pub)
cat: /root/.ssh/id_rsa.pub: No such file or directory
root@devops-vm:~# exit
logout
azureuser@devops-vm:~$ exit
logout
Connection to 20.85.210.242 closed.

~ ✖ ssh root@20.85.210.242
Please login as the user "azureuser" rather than the user "root".

Connection to 20.85.210.242 closed.

~ ➜  ssh azureuser@20.85.210.242
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun Mar  8 02:08:33 UTC 2026

  System load:  0.0               Processes:             105
  Usage of /:   5.7% of 28.89GB   Users logged in:       0
  Memory usage: 32%               IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '24.04.4 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Sun Mar  8 02:04:15 2026 from 65.108.255.62
azureuser@devops-vm:~$ sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
azureuser@devops-vm:~$ sudo systemctl restart ssh
azureuser@devops-vm:~$ sudo sed -i 's/.*ssh-rsa/ssh-rsa/' /root/.ssh/authorized_keys
azureuser@devops-vm:~$ sudo nano /etc/ssh/sshd_config
azureuser@devops-vm:~$ sudo systemctl restart ssh
azureuser@devops-vm:~$ ssh root@20.85.210.242
...
root@20.85.210.242: Permission denied (publickey).
azureuser@devops-vm:~$ exit
logout
Connection to 20.85.210.242 closed.

~ ➜  ssh root@20.85.210.242
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1044-azure x86_64)
...
root@devops-vm:~# 
```
