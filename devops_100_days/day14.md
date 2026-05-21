# Day 14: Apache Service Troubleshooting

## Task Description
The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.

Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. Also, make sure Apache is running on port 3004 on all app servers.

## Infrastructure Details

| Server Name | IP | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
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

## Theory
- **Apache HTTP Server (httpd):** A widely used web server software. In Linux (CentOS/RHEL), it is known as `httpd`.
- **Systemd & Systemctl:** Used to manage services in Linux (start, stop, check status).
- **Apache Configuration Files:** Located at `/etc/httpd/conf/httpd.conf`.
- **Network Ports:** Apache runs on Port 80 by default. It can be changed in the configuration file.
- **Port Conflict:** If a port is occupied by another service, Apache fails to start. We can use commands like `lsof` or `netstat` to find the conflicting process and `kill` to forcefully stop it.

## Mistakes Made and Fixes
- **Mistake**: Initially, the `httpd` service on `stapp01` failed to start because the port `3004` (or 80) was already in use by another process (`sendmail`). 
- **Why it happened**: Usually, another background service was accidentally configured to listen on the target port, creating a port conflict.
- **Fix**: Used `sudo lsof -i :3004` to identify the process (`sendmail` with PID `24715`). Terminated it forcefully with `sudo kill -9 24715`. After freeing the port, updated the Apache config and started the service successfully.

## Step-by-step Instructions

1. Identify the faulty app host by running `curl` from the jump host:
   ```bash
   curl -I http://stapp01:3004
   # Result: curl: (7) Failed to connect to stapp01 port 3004: Connection refused
   curl -I http://stapp02:3004
   # Result: HTTP/1.1 403 Forbidden
   curl -I http://stapp03:3004
   # Result: HTTP/1.1 403 Forbidden
   ```
   *This indicates `stapp01` is the faulty one since connections are refused.*

2. SSH into `stapp01`:
   ```bash
   ssh tony@stapp01
   # Use password: Ir0nM@n
   ```

3. Check the Apache service status:
   ```bash
   sudo systemctl status httpd
   ```

4. Address the Port Conflict:
   ```bash
   sudo lsof -i :3004
   # Sendmail was found using the port.
   sudo kill -9 24715
   ```

5. Change the port in Apache configuration:
   ```bash
   sudo sed -i 's/Listen 80/Listen 3004/g' /etc/httpd/conf/httpd.conf
   ```

6. Start and Enable Apache:
   ```bash
   sudo systemctl start httpd
   sudo systemctl enable httpd
   sudo systemctl status httpd
   ```

## Task Output Execution
```bash
thor@jump-host ~$ curl -I http://stapp02:3004
HTTP/1.1 403 Forbidden

thor@jump-host ~$ curl -I http://stapp01:3004
curl: (7) Failed to connect to stapp01 port 3004: Connection refused

thor@jump-host ~$ curl -I http://stapp03:3004
HTTP/1.1 403 Forbidden

thor@jump-host ~$ ssh tony@stapp01
tony@stapp01's password: 
[tony@stapp01 ~]$ sudo systemctl status httpd
× httpd.service - The Apache HTTP Server
     Active: failed (Result: exit-code) 

[tony@stapp01 ~]$ sudo lsof -i :3004
COMMAND    PID USER   FD   TYPE     DEVICE SIZE/OFF NODE NAME
sendmail 24715 root    4u  IPv4 1525935642      0t0  TCP localhost:csoftragent (LISTEN)

[tony@stapp01 ~]$ sudo kill -9 24715
[tony@stapp01 ~]$ sudo sed -i 's/Listen 80/Listen 3004/g' /etc/httpd/conf/httpd.conf
[tony@stapp01 ~]$ sudo systemctl start httpd
[tony@stapp01 ~]$ sudo systemctl enable httpd
[tony@stapp01 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Active: active (running) 
```