# Day 20: Setting Up Nginx with PHP-FPM 8.3 on Application Server 1

**Date:** April 13, 2026  
**Task:** Install and configure Nginx web server with PHP-FPM 8.3 on Application Server 1 (stapp01)

---

## Infrastructure Details

| Server Name | IP Address | Hostname | Username | Password | Purpose |
|---|---|---|---|---|---|
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | Dynamic | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| Application Server 3 | Dynamic | stapp03 | banner | BigGr33n | Hosts Nautilus Application 3 |
| Load Balancer Server | Dynamic | stlb01 | loki | Mischi3f | Distributes traffic for Nautilus HTTP |
| Database Server | Dynamic | stdb01 | peter | Sp!dy | Hosts Nautilus Database |
| Storage Server | Dynamic | sstor01 | natasha | Bl@kW | Stores data for Nautilus Servers |
| Backup Server | Dynamic | stbkp01 | clint | H@wk3y3 | Manages backups for Nautilus Servers |
| Mail Server | Dynamic | stmail01 | groot | Gr00T123 | Manages email services for Nautilus Servers |
| Jump Host Server | Dynamic | jump-host | thor | jolnir123 | Provides secure access to Stork DC |
| Jenkins Server | Dynamic | jenkins | jenkins | j@rv!s | Runs Jenkins for CI/CD pipeline |

**⚠️ Note:** The credentials listed above are no longer in use and are provided for historical documentation purposes only.

---

## Task Requirements

### a. Install Nginx
- Install Nginx web server on Application Server 1 (stapp01)
- Configure it to use **Port 8093**
- Set document root to **/var/www/html**

### b. Install PHP-FPM 8.3
- Install PHP-FPM version 8.3 on Application Server 1
- Configure it to use Unix socket: **/var/run/php-fpm/default.sock**
- Create parent directories if they don't exist

### c. Configure Nginx and PHP-FPM Integration
- Establish proper communication between Nginx and PHP-FPM using Unix sockets
- Ensure FastCGI parameters are correctly configured

### d. Testing
- Verify the setup using: `curl http://stapp01:8093/index.php`
- Pre-existing PHP files (index.php and info.php) must not be modified

---

## Theory and Concepts

### 1. What is Nginx?
Nginx is a lightweight, high-performance web server that efficiently handles HTTP requests and responses. Unlike Apache with its multi-process model, Nginx uses an event-driven architecture, making it more scalable and resource-efficient.

**Key features:**
- Asynchronous event-driven architecture
- Reverse proxy capabilities
- Load balancing support
- Low memory footprint
- Handles thousands of concurrent connections

### 2. PHP-FPM (FastCGI Process Manager)
PHP-FPM is a FastCGI server implementation that runs PHP code on the server side. While Nginx cannot directly execute PHP scripts, PHP-FPM handles the execution and returns results to Nginx.

**Why PHP-FPM is needed:**
- Nginx cannot natively execute PHP files
- FastCGI protocol bridges Nginx and PHP execution
- Manages PHP worker processes efficiently
- Allows multiple PHP versions on the same server

### 3. Unix Sockets vs TCP Sockets
There are two methods for Nginx and PHP-FPM communication:

**TCP Socket:**
- Uses IP address and port (e.g., 127.0.0.1:9000)
- Can be used for remote servers
- Slightly more overhead

**Unix Socket:**
- Uses a file-based socket (e.g., /var/run/php-fpm/default.sock)
- Only for local server communication
- Faster for local communication
- Better performance for single-server setups
- **Used in this task**

### 4. Socket Permissions and Ownership
For proper communication:
- Socket file must be readable/writable by both processes
- Nginx (web server) must have permission to access the socket
- `listen.mode = 0660` sets permissions (owner and group readable/writable)
- `listen.owner` and `listen.group` must be set to `nginx`

### 5. Remi Repository
Remi Repository is a reliable source for newer PHP versions on CentOS/RHEL systems. It provides SCL (Software Collections) which allow multiple PHP versions to coexist.

**Benefits:**
- Access to latest PHP versions
- Security updates
- Multiple PHP versions support
- Enterprise-grade reliability

### 6. FastCGI Protocol
FastCGI is a protocol for interfacing web servers with application servers. It passes HTTP request information to PHP-FPM and receives the processed output.

**In this setup:**
- Nginx receives HTTP requests
- Converts them to FastCGI format
- Sends via Unix socket to PHP-FPM
- PHP-FPM processes and returns response
- Nginx sends response back to client

---

## Step-by-Step Implementation Guide

### Prerequisites
- SSH access to stapp01 (SSH credentials: tony / Ir0nM@n)
- Sudo privileges for the `tony` user
- Internet connectivity for package downloads

### Step 1: Connect to Application Server 1

```bash
ssh tony@stapp01
# Password: Ir0nM@n
```

### Step 2: Install Nginx

```bash
sudo yum install nginx -y
```

**Output:**
```
Last metadata expiration check: 0:01:06 ago on Mon Apr 13 17:03:03 2026.
Dependencies resolved.
=================================================================================================================
 Package                         Architecture        Version                        Repository              Size
=================================================================================================================
Installing:
 nginx                           x86_64              2:1.20.1-28.el9                appstream               37 k
Installing dependencies:
 centos-logos-httpd              noarch              90.9-1.el9                     appstream              1.5 M
 nginx-core                      x86_64              2:1.20.1-28.el9                appstream              571 k
 nginx-filesystem                noarch              2:1.20.1-28.el9                appstream              9.6 k
Installing weak dependencies:
 logrotate                       x86_64              3.18.0-12.el9                  baseos                  74 k

Transaction Summary
=================================================================================================================
Install  5 Packages

Total download size: 2.2 M
Installed size: 4.5 M
...
Installed:
  centos-logos-httpd-90.9-1.el9.noarch   logrotate-3.18.0-12.el9.x86_64            nginx-2:1.20.1-28.el9.x86_64
  nginx-core-2:1.20.1-28.el9.x86_64      nginx-filesystem-2:1.20.1-28.el9.noarch

Complete!
```

### Step 3: Start and Enable Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

**Output:**
```
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
```

### Step 4: Add Remi Repository for PHP 8.3

#### Mistake #1: Initial Remi Repository URL Issue
The first command used an incompatible EL7 repository:
```bash
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm -y
```

**Error:**
```
Error:
 Problem: conflicting requests
  - nothing provides epel-release = 7 needed by remi-release-7.9-6.el7.remi.noarch from @commandline
```

**Root Cause:** The system is running EL9 (Enterprise Linux 9), not EL7. Repository versions must match the OS version.

**Fix:** Use dynamic variable substitution to detect the correct EL version:

```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(cut -d ' ' -f 4 /etc/redhat-release | cut -d '.' -f 1).noarch.rpm -y
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-$(cut -d ' ' -f 4 /etc/redhat-release | cut -d '.' -f 1).rpm -y
```

**Output:**
```
Last metadata expiration check: 0:05:59 ago on Mon Apr 13 17:03:03 2026.
epel-release-latest-9.noarch.rpm                                                  36 kB/s |  19 kB     00:00
Package epel-release-9-10.el9.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!

Last metadata expiration check: 0:06:01 ago on Mon Apr 13 17:03:03 2026.
remi-release-9.rpm                                                               138 kB/s |  33 kB     00:00
Dependencies resolved.
=================================================================================================================
 Package                    Architecture         Version                        Repository                  Size
=================================================================================================================
Installing:
 remi-release               noarch               9.7-4.el9.remi                 @commandline                33 k

Installed:
  remi-release-9.7-4.el9.remi.noarch

Complete!
```

### Step 5: Install PHP 8.3 and PHP-FPM

```bash
sudo dnf install php83 php83-php-fpm
```

**Output (showing key parts):**
```
Last metadata expiration check: 0:00:17 ago on Mon Apr 13 17:09:17 2026.
Dependencies resolved.
=================================================================================================================
 Package                                Architecture     Version                       Repository           Size
=================================================================================================================
Installing:
 php83                                  x86_64           8.3-1.el9.remi                remi-safe           7.2 k
 php83-php-fpm                          x86_64           8.3.30-1.el9.remi             remi-safe           1.9 M
Installing dependencies:
 [26 additional packages downloaded...]

Transaction Summary
=================================================================================================================
Install  26 Packages

Total download size: 16 M
Installed size: 70 M

Installed:
  [26 packages successfully installed]
```

### Step 6: Install PHP Syspaths

This package creates symlinks for easy command-line access to PHP tools:

```bash
sudo dnf install php83-syspaths -y
```

**Output:**
```
Last metadata expiration check: 0:00:44 ago on Mon Apr 13 17:10:06 2026.
Dependencies resolved.
=================================================================================================================
 Package                      Architecture         Version                         Repository               Size
=================================================================================================================
Installing:
 php83-syspaths               x86_64               8.3-1.el9.remi                  remi-safe               8.8 k

Installed:
  php83-syspaths-8.3-1.el9.remi.x86_64

Complete!
```

### Step 7: Create Socket Directory

```bash
sudo mkdir -p /var/run/php-fpm
```

#### Mistake #2: Wrong User Name for Socket Directory Ownership

Initial (incorrect) attempt:
```bash
sudo chown -R php-fpm:php-fpm /var/run/php-fpm
```

**Error:**
```
chown: invalid user: 'php-fpm:php-fpm'
```

**Root Cause:** When using Remi's Software Collection (SCL) for PHP 8.3, the user doesn't exist as a system user named `php-fpm`. Instead, we need to use the `nginx` user since Nginx will be accessing the socket.

**Fix:** Set ownership to `nginx`:
```bash
sudo chown -R nginx:nginx /var/run/php-fpm
```

### Step 8: Configure PHP-FPM

Edit the PHP-FPM configuration file:

```bash
sudo vi /etc/opt/remi/php83/php-fpm.d/www.conf
```

**Key Configuration Changes Required:**

| Parameter | Change to | Reason |
|---|---|---|
| `user = apache` | `user = nginx` | PHP-FPM processes run as nginx user |
| `group = apache` | `group = nginx` | PHP-FPM group matches nginx |
| `listen = /var/opt/remi/php83/run/php-fpm/www.sock` | `listen = /var/run/php-fpm/default.sock` | Use standard socket location |
| `;listen.owner = nobody` | `listen.owner = nginx` | Socket owner is nginx |
| `;listen.group = nobody` | `listen.group = nginx` | Socket group is nginx |
| `;listen.mode = 0660` | `listen.mode = 0660` | Permissions for socket |
| `listen.acl_users = apache` | `;listen.acl_users = apache` | Comment out (use owner/group instead) |

**Configured Section:**
```ini
[www]

user = nginx
group = nginx

listen = /var/run/php-fpm/default.sock

listen.owner = nginx
listen.group = nginx
listen.mode = 0660

;listen.acl_users = apache
;listen.acl_groups =
```

### Step 9: Configure Nginx Virtual Server Block

Create Nginx configuration for the PHP application:

```bash
sudo vi /etc/nginx/conf.d/php_app.conf
```

**Content:**
```nginx
server {
    listen 8093;
    server_name _;
    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
    }
}
```

**Key Configuration Details:**
- `listen 8093` - Listens on port 8093 as required
- `root /var/www/html` - Document root directory
- `server_name _` - Accepts all server names
- `fastcgi_pass unix:/var/run/php-fpm/default.sock` - FastCGI backend using Unix socket (must match PHP-FPM listen path)

### Step 10: Restart Services

#### Mistake #3: Service Name for PHP-FPM
Initial attempts used generic service names. The correct service name for Remi PHP 8.3 is `php83-php-fpm`, not `php-fpm`.

**Correct command:**
```bash
sudo systemctl restart php83-php-fpm
sudo systemctl restart nginx
```

**Output:**
```
[no output indicates success]
```

### Step 11: Enable Services at Boot

```bash
sudo systemctl enable php83-php-fpm
sudo systemctl enable nginx
```

**Output:**
```
Created symlink /etc/systemd/system/multi-user.target.wants/php83-php-fpm.service → /usr/lib/systemd/system/php83-php-fpm.service.
```

### Step 12: Verify Configuration

Test the PHP application via curl:

```bash
curl http://stapp01:8093/index.php
```

**Successful Output:**
```
Welcome to xFusionCorp Industries!
```

### Step 13: Additional Verification (Optional)

Test the info.php file:

```bash
curl http://stapp01:8093/info.php
```

This will output detailed PHP configuration and environment information in HTML format.

---

## Summary of Mistakes and Fixes

| # | Mistake | Error Message | Root Cause | Fix |
|---|---|---|---|---|
| 1 | Installed EL7 Remi repo on EL9 system | `conflicting requests - nothing provides epel-release = 7` | Repository version mismatch | Use dynamic version detection: `$(cut -d ' ' -f 4 /etc/redhat-release | cut -d '.' -f 1)` |
| 2 | Used non-existent `php-fpm:php-fpm` user | `chown: invalid user: 'php-fpm:php-fpm'` | SCL PHP doesn't create system user `php-fpm` | Use `nginx:nginx` as socket owner |
| 3 | Used generic `php-fpm` service name | Service failed to exist/start | Remi SCL creates service as `php83-php-fpm` | Use service name: `php83-php-fpm` |
| 4 | Socket path mismatch (initial thought) | N/A | Default SCL path differs from requirement | Explicitly set `listen = /var/run/php-fpm/default.sock` in config |

---

## Key Learning Points for DevOps Engineers

### 1. Repository Management
- Always match repository versions with OS versions
- Use dynamic detection for version numbers when possible
- Test repository commands in non-production first

### 2. Service Naming in SCL
- Software Collections modify standard package names
- PHP 8.3 from Remi becomes `php83-*` not `php*`
- Configuration files are under `/etc/opt/remi/php83/`

### 3. Unix Socket Permissions
- Both processes (Nginx and PHP-FPM) must have socket access
- Ownership and mode are critical: `listen.owner` and `listen.mode`
- Use ACL comments to allow owner/group permissions

### 4. FastCGI Path Consistency
- PHP-FPM `listen` path must exactly match Nginx `fastcgi_pass` path
- Path mismatch is a common 502 Bad Gateway error cause
- Verify paths match using grep or cat commands

### 5. Testing Strategy
- Start services individually before integration testing
- Use curl for HTTP testing from different hosts
- Check service logs: `sudo journalctl -u php83-php-fpm -n 50`

---

## Troubleshooting Commands

If issues occur during or after setup, these commands are helpful:

```bash
# Check service status
sudo systemctl status php83-php-fpm
sudo systemctl status nginx

# View service logs
sudo journalctl -u php83-php-fpm -n 50
sudo journalctl -u nginx -n 50

# Verify socket exists
ls -la /var/run/php-fpm/default.sock

# Test nginx configuration
sudo nginx -t

# Check PHP-FPM process
sudo ps aux | grep php83

# Verify listening ports
sudo netstat -tulpn | grep :8093
# or
sudo ss -tulpn | grep :8093

# Check socket permissions
stat /var/run/php-fpm/default.sock

# Test FastCGI connection
sudo -u nginx curl --unix-socket /var/run/php-fpm/default.sock http://localhost/index.php
```

---

## Task Completion Status

✅ **COMPLETE** - All requirements successfully implemented and verified

- Nginx installed and configured on port 8093 with document root /var/www/html
- PHP-FPM 8.3 installed and configured with Unix socket /var/run/php-fpm/default.sock
- Nginx and PHP-FPM properly integrated and communicating
- Verification successful: `curl http://stapp01:8093/index.php` returns expected output
- Services enabled to start automatically on server reboot

---

**Date of Completion:** April 13, 2026  
**Server:** stapp01 (Application Server 1)  
**CentOS/RHEL Version:** 9  
**Nginx Version:** 1.20.1-28.el9  
**PHP Version:** 8.3.30  
**PHP-FPM Version:** 8.3.30-1.el9.remi
