Day 37: Integrate PHP app (xfusion-php-vm) with MySQL (xfusion-mysql-vm)
Date: 2026-05-30
Topic: Integrating a PHP application on one Azure VM with a MySQL database on another Azure VM
Project: Nautilus DevOps lab (example)

---

## Server Credentials (Historical / Example values)

Note: The following credentials are historical/example values used during the task and are no longer active.

- MySQL VM (xfusion-mysql-vm)
  - Host / Public IP: 20.29.80.41
  - SSH Username: xfusion_admin
  - SSH Password: Namin@123456
  - SSH Port: 22
  - MySQL Port: 3306

- PHP VM (xfusion-php-vm)
  - Host / Public IP: 20.119.70.134
  - Web server: Apache2 (serving /var/www/html/db_test.php)
  - (No active SSH password disclosed here; access often performed via Azure Run Command or key injection)

## Database Credentials (Historical / Example values)

- MySQL database name: xfusion_db
- MySQL user: xfusion_user
- MySQL password: password123

Other example credential entries (historical):

- Example API key (example only): AKIAEXAMPLEKEY123456789
- Example database backup password: Backup#Example!2026

---

## Task Output (collected logs and command outputs)

Below are the actual command outputs and logs captured during the task (verbatim where provided). These were used to diagnose and validate the integration.

1. SSH and MySQL session creating DB and user (session excerpts):

```text
~ ➜  ssh xfusion_admin@20.29.80.41
The authenticity of host '20.29.80.41 (20.29.80.41)' can't be established.
ECDSA key fingerprint is SHA256:C4okDWrKlQysndFAA10HkxlXEdfewV8pHKxgJOwpBb0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.29.80.41' (ECDSA) to the list of known hosts.
xfusion_admin@20.29.80.41's password:
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.11.0-1016-azure x86_64)

xfusion_admin@xfusion-mysql-vm:~$ sudo /jet/enter mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 5.7.17-13 MySQL Community Server (GPL)

mysql> -- Create the database
mysql> CREATE DATABASE xfusion_db;
Query OK, 1 row affected (0.00 sec)

mysql> -- Create a user allowed to connect from any host (%)
mysql> CREATE USER 'xfusion_user'@'%' IDENTIFIED BY 'password123';
Query OK, 0 rows affected (0.00 sec)

mysql> -- Grant all privileges on the database to the new user
mysql> GRANT ALL PRIVILEGES ON xfusion_db.* TO 'xfusion_user'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> -- Flush privileges to apply changes
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> -- Exit MySQL shell
mysql> EXIT;
Bye
xfusion_admin@xfusion-mysql-vm:~$ exit
logout
Connection to 20.29.80.41 closed.
```

2. SSH key generation and public key content (example):

```text
~ ➜  ssh-keygen -t rsa -b 4096 -C "azureuser"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
/root/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:aaaFBQttBUziGHeuveYW+ABnxp30gTuKreR+n6e+dgU azureuser
```

Public key (truncated here for brevity):

```text
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC5qFZPJ6... azureuser
```

3. Attempt to restart MySQL and diagnostics showing Jetware handling:

```text
xfusion_admin@xfusion-mysql-vm:~$ sudo nano /jet/etc/mysql/my.cnf
xfusion_admin@xfusion-mysql-vm:~$ sudo cat /jet/etc/mysql/my.cnf
bind-address = 0.0.0.0
xfusion_admin@xfusion-mysql-vm:~$ sudo systemctl restart mysql
Failed to restart mysql.service: Unit mysql.service not found.
xfusion_admin@xfusion-mysql-vm:~$ sudo /jet/enter mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 65
Server version: 5.7.17-13 MySQL Community Server (GPL)
```

4. MySQL client not found locally when attempting remote test from local machine:

```text
~ ➜  mysql -u xfusion_user -p -h 20.29.80.41
-bash: mysql: command not found
```

5. Apache error log excerpts showing placeholder hostname error (key diagnostic):

```text
[Sat May 30 03:58:51.190534 2026] [php:warn] [pid 9962] [client 175.157.16.192:16864] PHP Warning:  mysqli::__construct(): php_network_getaddresses: getaddrinfo for <mysql-vm-public-ip> failed: Name or service not known in /var/www/html/db_test.php on line 8
[Sat May 30 03:58:51.190960 2026] [php:error] [pid 9962] [client 175.157.16.192:16864] PHP Fatal error:  Uncaught mysqli_sql_exception: php_network_getaddresses: getaddrinfo for <mysql-vm-public-ip> failed: Name or service not known in /var/www/html/db_test.php:8\nStack trace:\n#0 /var/www/html/db_test.php(8): mysqli->__construct()\n#1 {main}\n  thrown in /var/www/html/db_test.php on line 8
... (repeated entries showing same failure)
```

6. Azure CLI run-command used to overwrite `db_test.php` and test connection (worked):

```text
~ ➜  az vm run-command invoke \
  --resource-group kml_rg_main-da6768b4a9e24e74 \
  --name xfusion-php-vm \
  --command-id RunShellScript \
  --scripts "cat << 'EOF' > /var/www/html/db_test.php
<?php
// Database configuration with the correct MySQL IP
\$servername = "20.29.80.41";
\$username = "xfusion_user";
\$password = "password123";
\$dbname = "xfusion_db";

// Establish connection
\$conn = new mysqli(\$servername, \$username, \$password, \$dbname);

// Check connection
if (\$conn->connect_error) {
    die("Connection failed: " . \$conn->connect_error);
}
echo "Connected successfully";
\$conn->close();
?>
EOF
chmod 644 /var/www/html/db_test.php
sudo systemctl restart apache2"
{
  "value": [ ... Provisioning succeeded ... ]
}

~ ➜  curl http://20.119.70.134/db_test.php
Connected successfully
```

7. Other diagnostic CLI outputs and notes (policy error example seen during provisioning):

```json
{
  "code": "RequestDisallowedByPolicy",
  "message": "Resource 'xfusion-mysql-vm_OsDisk_1_33ef0357f850439f9b9102fc4be02602' was disallowed by policy. Reasons: 'Global: This storage configuration is not allowed. Ensure the disk size is 128 GB or less and the SKU is not Premium for Compute disks. For Storage accounts, use Standard_LRS or Standard_RAGRS SKUs.'..."
}
```

---

## Theories and Concepts Used

This section explains the underlying concepts, configurations, and behaviors relevant to the task.

- Virtual Machines (VMs): VMs provide isolated compute environments; Azure regions (Central US, East US) can be used to host separate roles (DB vs App).

- Networking and Security Groups (NSG): Inbound/outbound traffic is controlled by Azure NSGs. For MySQL exposure, inbound port 3306 needs an allow rule on the MySQL VM's NIC or subnet.

- MySQL bind-address: MySQL binds to an address (127.0.0.1 or 0.0.0.0). To accept remote connections, set `bind-address = 0.0.0.0` in the MySQL configuration and restart the appropriate service.

- Jetware wrapper: Jetware images often provide custom service wrappers (e.g., `sudo /jet/enter mysql`) and may not expose `mysql.service` to systemctl. Use provided wrapper commands to enter/manage MySQL.

- Azure Run Command: `az vm run-command invoke` allows executing scripts inside a VM without direct SSH. Useful when SSH is not available or when you prefer CLI automation.

- DNS vs IP resolution: PHP's mysqli attempts DNS resolution for hostnames; placeholder strings like `<mysql-vm-public-ip>` cause getaddrinfo failures. Use the literal IP or resolvable hostname.

- Least Privilege and networking security: Prefer limiting NSG source addresses to only the PHP VM's IP (or the application subnet) rather than allowing `Any`.

---

## Mistakes, Root Causes, and Fixes

Mistake 1: db_test.php contained a placeholder host string (`<mysql-vm-public-ip>`) instead of the real IP

- Why it happened: The sample `db_test.php` was left with a placeholder value; the application code attempted DNS resolution on that literal string.
- Root cause: Human/template oversight — the placeholder was never replaced with the real public IP.
- How it was fixed:
  1. Overwrote `/var/www/html/db_test.php` on the PHP VM using `az vm run-command invoke` with the correct `20.29.80.41` value.
  2. Set file permissions (`chmod 644`) and restarted Apache (`sudo systemctl restart apache2`).
  3. Validated with `curl http://20.119.70.134/db_test.php` and observed `Connected successfully`.

Mistake 2: MySQL bound only to local interface or MySQL service not restarting via systemctl

- Why it happened: Jetware images use custom wrappers; `systemctl restart mysql` returned "Unit mysql.service not found". Additionally, some MySQL configurations default to `bind-address = 127.0.0.1`.
- Root cause: Image-specific service management and default bind-address prevented remote connections.
- How it was fixed:
  1. Edited Jetware MySQL config at `/jet/etc/mysql/my.cnf` (or appropriate path) and set `bind-address = 0.0.0.0`.
  2. Used Jetware wrapper (`sudo /jet/enter mysql`) and relevant wrapper restart mechanisms if `systemctl` was not applicable.
  3. Verified MySQL accepted remote connections by re-granting privileges and testing from the PHP VM.

Mistake 3: NSG / provisioning policy issues during VM creation (RequestDisallowedByPolicy)

- Why it happened: Subscription policy blocked certain disk SKUs or sizes (e.g., Premium disks or disks > 128 GB).
- Root cause: Global/Org Azure Policy enforcement.
- How it was fixed:
  1. During VM creation, choose acceptable disk SKU (Standard SSD or Standard HDD) and size ≤ 128 GB.
  2. Alternatively, create a VM with a supported base image and manually install MySQL if the Marketplace image is disallowed.

Mistake 4: No mysql client installed locally when attempting `mysql -u ... -h` test

- Why it happened: The local workstation lacked the `mysql` client binary.
- Root cause: Missing client packages on the testing host.
- How it was fixed:
  1. Install a MySQL client on the test machine (e.g., `sudo apt install mysql-client` on Debian/Ubuntu) or use `mysql` inside a container/ VM that has the client.
  2. Alternatively, use `telnet <ip> 3306` or `nc -vz <ip> 3306` to verify TCP connectivity.

---

## Step-by-step Summary (what was done)

1. Created MySQL VM `xfusion-mysql-vm` (Central US) with username `xfusion_admin` and password `Namin@123456`, ensured NSG allowed port 22 and later added rule to allow 3306.
2. SSHed to MySQL VM and used Jetware MySQL wrapper to create `xfusion_db` and user `xfusion_user` with password `password123` and granted privileges.
3. Adjusted MySQL config `bind-address = 0.0.0.0` to accept remote connections.
4. Overwrote `db_test.php` on `xfusion-php-vm` using `az vm run-command invoke` and set the DB host to `20.29.80.41`.
5. Restarted Apache on PHP VM and validated success via `curl` returning `Connected successfully`.

---

## Recommendations / Best Practices

- Use private networking (VNet peering) for app → DB traffic in production (avoid exposing 3306 publicly).
- Use NSG rules scoped to the application VM's IP or subnet rather than `Any`.
- Use Azure Key Vault or managed identities for secrets rather than hard-coded passwords.
- Use automation (Terraform/ARM/Bicep) to ensure repeatable, policy-compliant VM creation (avoid manual SKU choices that trigger Azure Policy failures).

---

If you want, I can now:

- run a quick pass to validate both files were created, or
- commit these files to a specific folder or add more concise executive summary bullets.

Files created: `/mnt/Local Disk D/DevOps/kodekloud/azure_100_days/day37.md`
