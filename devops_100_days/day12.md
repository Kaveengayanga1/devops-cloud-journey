# Day 12 - Apache not reachable on port 6200 (Stratos DC)

## Task Summary

Monitoring reported that one of the application servers in **Stratos Datacenter** has its **Apache HTTPD** service not reachable on **TCP port 6200** from the **jump-host**. We had to:

- Identify which app server is broken.
- Fix Apache so that it listens correctly on port **6200**.
- Ensure the port is reachable from the jump host **without weakening security**.
- Verify from the jump host using tools like `telnet` / `curl`.

> Note: The lab explicitly says **do not change `index.html`**.

---

## Infrastructure details (from the lab)

These credentials are now invalid in real life, but were used in this lab and are recorded here for reference:

| Role / Server Type      | Hostname   | User   | Password      | Purpose                                      |
|-------------------------|-----------|--------|--------------|----------------------------------------------|
| Application Server 1    | stapp01   | tony   | `Ir0nM@n`    | Hosts Nautilus Application 1 (Apache 6200)   |
| Application Server 2    | stapp02   | steve  | `Am3ric@`    | Hosts Nautilus Application 2                 |
| Application Server 3    | stapp03   | banner | `BigGr33n`   | Hosts Nautilus Application 3                 |
| Load Balancer Server    | stlb01    | loki   | `Mischi3f`   | HTTP load balancer                           |
| Database Server         | stdb01    | peter  | `Sp!dy`      | Nautilus DB                                  |
| Storage Server          | ststor01  | natasha| `Bl@kW`      | Storage for Nautilus                          |
| Backup Server           | stbkp01   | clint  | `H@wk3y3`    | Backup management                            |
| Mail Server             | stmail01  | groot  | `Gr00T123`   | Email services                               |
| Jump Host               | jump-host | thor   | `mjolnir123` | Secure access into Stork/Stratos DC          |
| Jenkins Server          | jenkins   | jenkins| `j@rv!s`     | CI/CD                                        |

---

## Raw troubleshooting output (chronological)

### 1. Initial connectivity tests from jump host

```bash
thor@jump-host ~$ telnet stapp01 6200
Trying 10.244.234.215...
telnet: connect to address 10.244.234.215: No route to host

thor@jump-host ~$ telnet stapp02 6200
Trying 10.244.244.163...
Connected to stapp02.
Escape character is '^]'.
^]
HTTP/1.1 400 Bad Request
Date: Tue, 31 Mar 2026 11:31:18 GMT
Server: Apache/2.4.62 (CentOS Stream)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
Connection closed by foreign host.

thor@jump-host ~$ telnet stapp03 6200
Trying 10.244.81.29...
Connected to stapp03.
Escape character is '^]'.
^]
HTTP/1.1 400 Bad Request
Date: Tue, 31 Mar 2026 11:31:28 GMT
Server: Apache/2.4.62 (CentOS Stream)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
Connection closed by foreign host.
```

**Interpretation:**

- `stapp02:6200` and `stapp03:6200` are reachable and return HTTP 400 (expected when speaking raw telnet to HTTP).
- `stapp01:6200` returns **`No route to host`**, which indicates a **network / firewall level** problem to that host/port, not just Apache being down.

So the problematic app server is **stapp01**.

---

### 2. SSH into stapp01 and check Apache

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.234.215)' can't be established.
ED25519 key fingerprint is SHA256:HIVBBR7MsTMXjCOhviLdDq8qrwCpJ9NcbtfweeClGm4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password: 

[tony@stapp01 ~]$ sudo systemctl status httpd

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2026-03-31 11:22:07 UTC; 10min ago
       Docs: man:httpd.service(8)
    Process: 10618 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 10618 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 36ms

Mar 31 11:22:07 stapp01 systemd[1]: Starting The Apache HTTP Server...
Mar 31 11:22:07 stapp01 httpd[10618]: AH00558: httpd: Could not reliably determine the server's fully qualified >
Mar 31 11:22:07 stapp01 httpd[10618]: (98)Address already in use: AH00072: make_sock: could not bind to address >
Mar 31 11:22:07 stapp01 httpd[10618]: (98)Address already in use: AH00072: make_sock: could not bind to address >
Mar 31 11:22:07 stapp01 httpd[10618]: no listening sockets available, shutting down
Mar 31 11:22:07 stapp01 httpd[10618]: AH00015: Unable to open logs
Mar 31 11:22:07 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Mar 31 11:22:07 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Mar 31 11:22:07 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
```

Later retries show the same error, with more detail:

```bash
[tony@stapp01 ~]$ sudo systemctl start httpd
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.

[tony@stapp01 ~]$ sudo systemctl status httpd
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2026-03-31 11:32:44 UTC; 1min 44s ago
       Docs: man:httpd.service(8)
    Process: 36605 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 36605 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 36ms

Mar 31 11:32:43 stapp01 systemd[1]: Starting The Apache HTTP Server...
Mar 31 11:32:44 stapp01 httpd[36605]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.234.215. Set t>
Mar 31 11:32:44 stapp01 httpd[36605]: (98)Address already in use: AH00072: make_sock: could not bind to address [::]:6200
Mar 31 11:32:44 stapp01 httpd[36605]: (98)Address already in use: AH00072: make_sock: could not bind to address 0.0.0.0:6200
Mar 31 11:32:44 stapp01 httpd[36605]: no listening sockets available, shutting down
Mar 31 11:32:44 stapp01 httpd[36605]: AH00015: Unable to open logs
Mar 31 11:32:44 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Mar 31 11:32:44 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Mar 31 11:32:44 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
```

**Key message:**

> `(98)Address already in use: ... could not bind to address [::]:6200 / 0.0.0.0:6200`

So **some other process is already listening on port 6200**, blocking Apache.

---

### 3. First attempts with non‑existent tools

```bash
[tony@stapp01 ~]$ sudo netstat -tunlp | grep 6200
sudo: netstat: command not found

[tony@stapp01 ~]$ sudo firewall-cmd --permanent --add-port=6200/tcp
sudo: firewall-cmd: command not found

[tony@stapp01 ~]$ sudo firewall-cmd --reload
sudo: firewall-cmd: command not found
```

These show that:

- `netstat` is not installed.
- `firewalld` / `firewall-cmd` is not present; the host is using **iptables** instead.

---

### 4. Find and kill the conflicting process on port 6200

Since `netstat` is unavailable, use `ss`:

```bash
[tony@stapp01 ~]$ sudo ss -tunlp | grep 6200
tcp   LISTEN 0      10         127.0.0.1:6200      0.0.0.0:*    users:(("sendmail",pid=9962,fd=4))
```

So **sendmail** is listening on `127.0.0.1:6200`.

Fix:

```bash
[tony@stapp01 ~]$ sudo kill -9 9962
[tony@stapp01 ~]$ sudo systemctl start httpd
[tony@stapp01 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-03-31 11:36:45 UTC; 4s ago
       Docs: man:httpd.service(8)
   Main PID: 37666 (httpd)
     Status: "Started, listening on: port 6200"
      Tasks: 177 (limit: 404712)
     Memory: 15.0M
        CPU: 64ms
     CGroup: /system.slice/httpd.service
             ├─37666 /usr/sbin/httpd -DFOREGROUND
             ├─37673 /usr/sbin/httpd -DFOREGROUND
             ├─37674 /usr/sbin/httpd -DFOREGROUND
             ├─37675 /usr/sbin/httpd -DFOREGROUND
             └─37676 /usr/sbin/httpd -DFOREGROUND

Mar 31 11:36:44 stapp01 systemd[1]: Starting The Apache HTTP Server...
Mar 31 11:36:45 stapp01 httpd[37666]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.234.215. Set t>
Mar 31 11:36:45 stapp01 httpd[37666]: Server configured, listening on: port 6200
Mar 31 11:36:45 stapp01 systemd[1]: Started The Apache HTTP Server.

[tony@stapp01 ~]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```

Now Apache is up and bound to port 6200.

Verify the bind:

```bash
[tony@stapp01 ~]$ sudo ss -tunlp | grep 6200
[sudo] password for tony: 
tcp   LISTEN 0      511                *:6200            *:*    users:(("httpd",pid=37676,fd=4),("httpd",pid=37675,fd=4),("httpd",pid=37674,fd=4),("httpd",pid=37666,fd=4))
```

So now **Apache is listening on all interfaces on port 6200**.

---

### 5. Still not reachable from jump host: firewall (iptables) issue

From jump host, it still fails:

```bash
thor@jump-host ~$ telnet stapp01 6200
Trying 10.244.234.215...
telnet: connect to address 10.244.234.215: No route to host
```

Given Apache is up and listening, this strongly suggests a **host firewall** rule rejecting packets.

Check iptables on stapp01:

```bash
[tony@stapp01 ~]$ sudo iptables -L INPUT -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
4    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
5    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
6    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
7    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
```

The critical rule is line 7:

- `REJECT all ... reject-with icmp-host-prohibited`

This exactly causes the `No route to host` message on the client.

Add rules to allow port 6200 before that REJECT:

```bash
[tony@stapp01 ~]$ sudo iptables -I INPUT -p tcp --dport 6200 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -I INPUT 1 -p tcp --dport 6200 -j ACCEPT

[tony@stapp01 ~]$ sudo iptables -L INPUT -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:6200
2    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:6200
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
4    ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
5    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
6    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
7    REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
```

Attempt to persist rules (command not available, but recorded for completeness):

```bash
[tony@stapp01 ~]$ sudo service iptables save
sudo: service: command not found
```

Then exit and test again from jump host:

```bash
thor@jump-host ~$ telnet stapp01 6200
Trying 10.244.234.215...
Connected to stapp01.
Escape character is '^]'.
^]
HTTP/1.1 400 Bad Request
Date: Tue, 31 Mar 2026 11:40:08 GMT
Server: Apache/2.4.62 (CentOS Stream)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
Connection closed by foreign host.
thor@jump-host ~$ 
```

Now the port is reachable and Apache responds as expected.

---

## Theory (what you need to know)

### 1. Apache HTTP Server & non‑standard ports

- Apache (httpd) is a widely used web server.
- Default ports are **80** (HTTP) and **443** (HTTPS).
- In this lab Apache is configured to listen on **port 6200**.
- If another process binds to the same port, httpd fails to start with `Address already in use`.

### 2. Port binding and conflicts

- A TCP/UDP port can only be bound by **one** listening socket per address.
- When Apache starts, it tries to `bind()` to `[::]:6200` and `0.0.0.0:6200`.
- If something else is already listening there, you see:
  - `(98)Address already in use: make_sock: could not bind to address ...`
  - `no listening sockets available, shutting down`.
- Tools to see port usage:
  - `ss -tunlp` (modern replacement for `netstat`).
  - `netstat -tunlp` (if installed).

### 3. Host firewall: iptables vs firewalld

- Linux hosts often protect incoming traffic with:
  - `firewalld` (+ `firewall-cmd` CLI), or
  - raw **iptables** rules.
- In this lab, **firewalld was not installed**, but **iptables** rules were present.
- iptables chains:
  - `INPUT` – incoming traffic to the local host.
  - `OUTPUT` – outgoing.
  - `FORWARD` – routed traffic.
- Rule order matters; the first matching rule wins.
- Rule: `REJECT all ... reject-with icmp-host-prohibited` causes the remote client to see `No route to host`.

### 4. SELinux (not the main issue here)

- SELinux can also block httpd from binding to non‑standard ports unless the port has type `http_port_t`.
- Typical commands:
  - `semanage port -l | grep http_port_t`
  - `semanage port -a -t http_port_t -p tcp 6200`
- In this particular lab the failure was due to **port conflict + iptables**, not SELinux.

### 5. Troubleshooting with telnet and curl

- `telnet host port` is useful to confirm **TCP-level connectivity**.
  - `Connected` → TCP handshake OK; application may still error.
  - `No route to host` → routing / firewall / iptables REJECT.
  - `Connection refused` → host reachable, but nothing listening on that port.
- `curl http://host:port` verifies **HTTP** behaviour and content from Apache.

---

## What went wrong (root causes)

There were **two independent problems**:

1. **Port conflict on stapp01**
   - Process `sendmail` was already listening on `127.0.0.1:6200`.
   - Apache httpd was also configured to listen on port 6200.
   - On start, httpd failed with `(98)Address already in use` and shut down.

2. **Host firewall (iptables) blocking port 6200**
   - INPUT chain had a final rule:
     - `REJECT all ... reject-with icmp-host-prohibited`.
   - No rule existed to allow TCP dport 6200.
   - Incoming packets to 6200 from the jump host were rejected before reaching Apache.
   - On the client, this appeared as `No route to host`.

Only after fixing **both** issues did Apache become reachable from the jump host.

---

## Mistakes made during debugging and why they happened

1. **Assuming the problematic server was stapp03 (based only on the lab text)**
   - The lab statement: *"Once fixed, you can test the same using command curl http://stapp03:6200"* was misread as implying that **stapp03** was broken.
   - In reality, `telnet` tests clearly showed:
     - `stapp02:6200` → Connected (OK)
     - `stapp03:6200` → Connected (OK)
     - `stapp01:6200` → `No route to host` (problem)
   - Lesson: **Always trust actual observations over assumptions**; test all app servers before deciding.

2. **Trying to use `netstat` when it is not installed**
   - Habit from older systems led to `netstat -tunlp | grep 6200`.
   - The host did not have `netstat` in PATH, causing `command not found`.
   - Correct alternative: `ss -tunlp | grep 6200`.
   - Lesson: Know the modern toolset (`ss`) and adapt when commands are missing.

3. **Assuming firewalld (`firewall-cmd`) instead of iptables**
   - Ran:
     - `sudo firewall-cmd --permanent --add-port=6200/tcp`
     - `sudo firewall-cmd --reload`
   - Both returned `command not found` because `firewalld` was not installed.
   - Actual firewall implementation was `iptables`, confirmed by `iptables -L`.
   - Lesson: **Check what firewall framework is present** instead of assuming firewalld.

4. **Initially focusing only on Apache status, ignoring firewall clues from telnet**
   - `telnet stapp01 6200` from jump host already showed `No route to host`.
   - This indicates a network / firewall problem, not just a down service.
   - Focusing exclusively on systemd status delayed firewall diagnostics.
   - Lesson: use **end‑to‑end tests (telnet/curl)** to infer where in the path things are broken.

5. **Duplicate iptables allow rules**
   - Two very similar rules were added:
     - `sudo iptables -I INPUT -p tcp --dport 6200 -j ACCEPT`
     - `sudo iptables -I INPUT 1 -p tcp --dport 6200 -j ACCEPT`
   - Result: two ACCEPT rules for port 6200 (lines 1 and 2).
   - Functionally harmless but unnecessary.
   - Lesson: Be deliberate with `-I` (insert) vs `-A` (append) and avoid duplicate rules.

6. **Attempting to use `service iptables save` on a system without that helper**
   - Command:
     - `sudo service iptables save`
   - Response: `service: command not found` (or iptables service not managed via that wrapper).
   - This step was not essential for lab success but reflects an assumption about how iptables persistence is managed.
   - Lesson: On modern systems, firewall persistence is often handled differently (e.g., `iptables-save` + config files, or firewalld zones).

---

## Professional DevOps / Cloud troubleshooting steps (English summary)

These are the steps a professional engineer should follow for this kind of issue:

1. **Identify the failing server**
   - From jump host, test all app servers on the target port:
     - `telnet stapp01 6200`
     - `telnet stapp02 6200`
     - `telnet stapp03 6200`
   - Mark which one fails (stapp01 here).

2. **Log into the affected host**
   - From jump host:
     - `ssh tony@stapp01` (password `Ir0nM@n`).

3. **Check Apache service state**
   - `sudo systemctl status httpd`.
   - If `failed` with `Address already in use`, suspect port conflict.

4. **Check which process is using the port**
   - `sudo ss -tunlp | grep 6200`.
   - Identify process (here `sendmail` with PID 9962).

5. **Resolve port conflict**
   - Stop/kill the offending process or reconfigure it to another port:
     - `sudo kill -9 9962`.
   - Restart Apache:
     - `sudo systemctl start httpd`.
   - Enable on boot:
     - `sudo systemctl enable httpd`.

6. **Confirm Apache is listening correctly**
   - `sudo ss -tunlp | grep httpd` or `grep 6200`.
   - Expect `*:6200` or `0.0.0.0:6200` / `[::]:6200` bound to `httpd`.

7. **Inspect firewall rules**
   - If from jump host you still see `No route to host`, check host firewall:
     - `sudo iptables -L INPUT -n --line-numbers`.
   - Look for `REJECT`/`DROP` rules and absence of an allow rule for port 6200.

8. **Open the required port safely**
   - Insert allow rule **above** the REJECT:
     - `sudo iptables -I INPUT 1 -p tcp --dport 6200 -j ACCEPT`.
   - Re‑list to confirm ordering.

9. **(Optional) Persist the rules**
   - Depending on distribution (not strictly needed for this lab).

10. **End‑to‑end verification from jump host**
    - `telnet stapp01 6200` → should connect.
    - `curl http://stapp01:6200` or the lab‑specified check (for stapp03 in the generic statement).

11. **Do not modify application content**
    - Respect the requirement: **do not change `index.html`**.

---

This completes the Day 12 lab write‑up for Apache not reachable on port 6200 on stapp01.
