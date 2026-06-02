# Day 19: Hosting Two Static Websites on Apache at Port 5002

## Task Overview
xFusionCorp Industries needed App Server 3 ready to host two static websites from backups available on the Jump Host. The requirement was to install and configure Apache HTTP Server so that the `official` site works at `http://localhost:5002/official/` and the `apps` site works at `http://localhost:5002/apps/`.

## Infrastructure Details
| Server Name | Hostname | User | Password | Purpose |
|-------------|----------|------|----------|---------|
| Application Server 3 | `stapp03` | `banner` | `BigGr33n` | Host Apache and serve the websites |
| Jump Host | `jump-host` | `thor` | `mjolnir123` | Source of website backups |

## Requirements
1. Install `httpd` and its dependencies on App Server 3.
2. Configure Apache to listen on port `5002`.
3. Copy the website backups from `/home/thor/official` and `/home/thor/apps` on the Jump Host.
4. Serve `official` on `http://localhost:5002/official/` and `apps` on `http://localhost:5002/apps/`.
5. Verify both sites using `curl` from App Server 3.

## Theories You Need To Know

### Apache HTTP Server
Apache HTTP Server (`httpd`) is a widely used open-source web server on Linux. It serves web content from a configured document root and can host multiple static sites from different subdirectories.

### Port Binding
Web servers normally listen on port 80 for HTTP. In this task, Apache had to be moved to port `5002` by changing the `Listen` directive in the configuration.

### Document Root and Subdirectories
Apache’s default document root is usually `/var/www/html`. By creating subdirectories such as `/var/www/html/official` and `/var/www/html/apps`, multiple websites can be hosted on the same server and same port.

### SCP and File Copying
`scp` is used to securely copy files between servers. Here, the content from the Jump Host backups had to be copied to App Server 3 before placing it under Apache’s document root.

### Permissions and Ownership
Apache must be able to read the web files. Correct ownership and permissions are important so that `httpd` can serve the pages without `Forbidden` errors.

### Troubleshooting Apache
If `curl` shows `Connection refused`, the service is usually not running or not listening on the expected port. If `curl` shows `Forbidden`, the problem is usually permissions or SELinux. Apache configuration should always be checked with `apachectl configtest`.

## Professional Implementation Guide

### 1. Access App Server 3
```bash
ssh banner@stapp03
# Password: BigGr33n
```

### 2. Install Apache
```bash
sudo yum install httpd -y
```

### 3. Configure Apache to Listen on Port 5002
```bash
sudo sed -i 's/Listen 80/Listen 5002/g' /etc/httpd/conf/httpd.conf
```

### 4. Create Temporary Backup Directories
Before copying files from the Jump Host, create temporary local directories on App Server 3:
```bash
sudo mkdir -p /tmp/official_backup
sudo mkdir -p /tmp/apps_backup
```

### 5. Copy the Backups from the Jump Host
The correct hostname is `jump-host`, not `jump_host`.
```bash
scp -r thor@jump-host:/home/thor/official/* /tmp/official_backup/
scp -r thor@jump-host:/home/thor/apps/* /tmp/apps_backup/
```

### 6. Create Apache Website Directories
```bash
sudo mkdir -p /var/www/html/official
sudo mkdir -p /var/www/html/apps
```

### 7. Copy Website Files into the Apache Document Root
```bash
sudo cp -r /tmp/official_backup/* /var/www/html/official/
sudo cp -r /tmp/apps_backup/* /var/www/html/apps/
```

### 8. Set Ownership and Permissions
```bash
sudo chown -R apache:apache /var/www/html/official
sudo chown -R apache:apache /var/www/html/apps
sudo chmod -R 755 /var/www/html/official
sudo chmod -R 755 /var/www/html/apps
```

### 9. Start and Enable Apache
```bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

### 10. Verify the Configuration
```bash
sudo apachectl configtest
sudo systemctl status httpd
sudo ss -tulnp | grep 5002
curl http://localhost:5002/official/
curl http://localhost:5002/apps/
```

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. Wrong Hostname Used in SCP
**Mistake:** `jump_host` was used instead of `jump-host`.

**Why it happened:** The hostname was typed with an underscore instead of the actual hyphenated hostname.

**Fix:** Use the correct hostname `jump-host` in the SCP command.

### 2. Missing Local Destination Directory for SCP
**Mistake:** `scp` failed with `open local "/tmp/official_backup/": Is a directory`.

**Why it happened:** The destination path was not prepared correctly for the copy operation, and the backup directory needed to exist before copying files into it.

**Fix:** Create the destination folders first with `mkdir -p` before running SCP.

### 3. Apache Still Listening on Port 80
**Mistake:** `systemctl status httpd` showed Apache listening on port 80 even after editing the config.

**Why it happened:** Apache had not been restarted after the configuration change, or the running service had not reloaded the updated configuration.

**Fix:** Restart Apache and confirm the active port with `systemctl status httpd` and `ss -tulnp | grep 5002`.

### 4. `Connection refused` on Curl
**Mistake:** `curl http://localhost:5002/official/` returned `Connection refused`.

**Why it happened:** Apache was not actively listening on port 5002 at that moment.

**Fix:** Confirm the `Listen 5002` directive, run `apachectl configtest`, restart `httpd`, and verify that the service is bound to port 5002.

### 5. Extra Shell Text Typed Accidentally
**Mistake:** Commands like `al` and copied prompt text such as `[banner@stapp03 ...]` were entered into the shell.

**Why it happened:** Terminal prompt text was accidentally copied along with the command output.

**Fix:** Ignore those input mistakes and re-run only valid shell commands.

## Full Terminal Output
```text
[banner@stapp03 apps]$ curl http://localhost:5002/official/
<!DOCTYPE html>
<html>
<body>

<h1>KodeKloud</h1>

<p>This is a sample page for our official website</p>

</body>
</html>
[banner@stapp03 alps]$ al
[banner@stapp03 apps]$
[banner@stapp03 ~]$ cd /var/www/html/official/
-bash: [banner@stapp03: command not found
-bash: al: command not found
-bash: [banner@stapp03: command not found
-bash: [banner@stapp03: command not found
-bash: -bash:: command not found
[banner@stapp03 apps]$ curl http://localhost:5002/apps/
<!DOCTYPE html>
<html>
<body>

<h1>KodeKloud</h1>

<p>This is a sample page for our apps website</p>

</body>
</html>
[banner@stapp03 apps]$
```

## Final Result
Both websites were successfully served through Apache on port `5002`. The final `curl` output confirmed that the `official` and `apps` sites were accessible from App Server 3.
