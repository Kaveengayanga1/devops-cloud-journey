# Day 16: Configure Firewalld and Allow Ports

## Infrastructure and Security Credentials

Since the servers are no longer available directly, here are the infrastructure details and credentials for your reference:

| Server Name | IP | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Application Server 2 | Dynamic | stapp02 | steve | Am3ric@ | Hosts Nautilus Application 2 |
| **Application Server 3** | **Dynamic** | **stapp03** | **banner** | **BigGr33n** | **Hosts Nautilus Application 3** |
| LoadBalancer Server | Dynamic | stlb01 | loki | Mischi3f | Distributes traffic for Nautilus HTTP |
| Database Server | Dynamic | stdb01 | peter | Sp!dy | Hosts Nautilus Database |
| Storage Server | Dynamic | ststor01 | natasha | Bl@kW | Stores data for Nautilus Servers |
| Backup Server | Dynamic | stbkp01 | clint | H@wk3y3 | Manages backups for Nautilus Servers |
| Mail Server | Dynamic | stmail01 | groot | Gr00T123 | Manages email services for Nautilus Servers |
| **Jump Host Server** | **Dynamic** | **jump-host** | **thor** | **mjolnir123** | **Provides secure access to Stork DC** |
| Jenkins Server | Dynamic | jenkins | jenkins | j@rv!s | Runs Jenkins for CI/CD pipeline |

---

## Theory
The Nautilus system administrators team has rolled out a web UI application for their backup utility on the Nautilus application server 3 (stapp03) within the Stratos Datacenter. This application runs on port `6100` and appropriate firewall rules must be configured to allow incoming traffic. To achieve this, `firewalld` needs to be installed and configured on the application server. 

### Why Firewalld?
**Firewalld** is a dynamic firewall manager primarily used in RHEL/CentOS distributions. It handles network traffic restrictions using `zones` and services/ports configurations.

### Key Concepts
- **Zones:** They dictate the trust level of the network connections. For general incoming application traffic, the `public` zone is commonly used.
- **Runtime vs Permanent Rules:** 
  - *Runtime rules* are active immediately but lost upon a reload or restart.
  - *Permanent rules* persist through restarts but require a `--reload` to become active.
- **Port `6100/tcp`:** The door through which the backup UI receives HTTP requests.

### Task Requirements
1. Install and enable the `firewalld` service on App Server 3.
2. Ensure the firewall zone is set to `public`.
3. Allow all incoming connections on port `6100/tcp`.

---

## Professional DevOps Steps

1. **SSH into Application Server 3 (stapp03)** from the Jump Host.
2. **Install Firewalld** using `yum`/`dnf`.
3. **Enable and Start the Service** using `systemctl` to ensure it runs even after reboots.
4. **Configure the Firewall Rules** by adding port `6100/tcp` permanently to the `public` zone.
5. **Reload the Firewall** to enforce the permanent change.
6. **Verify the rules** from inside the server and perform an external connection test (e.g. `nc` or `telnet`) from the Jump Host.

---

## Execution and Output

```bash
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.29.210)' can't be established.
ED25519 key fingerprint is SHA256:AO99MYFvb0NsgJ7N9l4qpzg+x5F35E6ju04XC+fH9e8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password: 

[banner@stapp03 ~]$ sudo yum install -y firewalld
[sudo] password for banner: 
Last metadata expiration check: 0:03:29 ago on Sun Mar 29 03:12:31 2026.
Dependencies resolved.
# ... [Installation details omitted for brevity. Total download size: 2.7 M] ...
Complete!

[banner@stapp03 ~]$ sudo systemctl enable firewalld
[banner@stapp03 ~]$ sudo systemctl start firewalld
[banner@stapp03 ~]$ sudo systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-03-29 03:16:14 UTC; 11s ago
       Docs: man:firewalld(1)
   Main PID: 35023 (firewalld)
      Tasks: 2 (limit: 404712)
     Memory: 22.1M
        CPU: 186ms
     CGroup: /system.slice/firewalld.service
             └─35023 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

[banner@stapp03 ~]$ sudo firewall-cmd --get-default-zone
public

[banner@stapp03 ~]$ sudo firewall-cmd --zone=public --add-port=6100/tcp --permanent
success

[banner@stapp03 ~]$ sudo firewall-cmd --reload
success

[banner@stapp03 ~]$ sudo firewall-cmd --zone=public --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 6100/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

[banner@stapp03 ~]$ exit
logout
Connection to stapp03 closed.
```

### Verification from Jump Host

```bash
thor@jump-host ~$ nc -zv stapp03 6100
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Connected to 10.244.29.210:6100.
Ncat: 0 bytes sent, 0 bytes received in 0.01 seconds.

thor@jump-host ~$ telnet stapp03 6100
Trying 10.244.29.210...
Connected to stapp03.
Escape character is '^]'.
^[
HTTP/1.1 400 Bad Request
Date: Sun, 29 Mar 2026 03:18:06 GMT
Server: Apache/2.4.62 (CentOS Stream)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
...
Connection closed by foreign host.
```
*(The HTTP `400 Bad Request` simply means the web server at port `6100` received the raw telnet connection but didn't get proper HTTP headers yet, confirming that the port is open and networking works!)*
