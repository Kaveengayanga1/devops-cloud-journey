# Day 17: Configuring Nginx as a Load Balancer for Apache App Servers

## Task Overview

The Nautilus production support team observed performance degradation on their website due to increasing traffic. To ensure high availability and better performance, the team decided to migrate the application to a high availability stack. The goal is to configure an Nginx Load Balancer (LBR) to distribute traffic across three Apache Application Servers.

### Requirements:
1.  **Install Nginx** on the LBR server (`stlb01`) if not already installed.
2.  **Configure Load Balancing** in the `http` context of `/etc/nginx/nginx.conf` using all application servers.
3.  **Ensure Apache is running** on all app servers without changing their pre-defined ports.
4.  **Verify the setup** by accessing the website via `curl http://stlb01:80`.

---

## Infrastructure Details

| Server Name | Hostname | User | Password | Purpose | Port |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Application Server 1 | `stapp01` | `tony` | `Ir0nM@n` | Hosts Nautilus Application 1 | `8084` |
| Application Server 2 | `stapp02` | `steve` | `Am3ric@` | Hosts Nautilus Application 2 | `8084` |
| Application Server 3 | `stapp03` | `banner` | `BigGr33n` | Hosts Nautilus Application 3 | `8084` |
| LoadBalancer Server | `stlb01` | `loki` | `Mischi3f` | Distributes traffic | `80` |
| Jump Host | `jump-host` | `thor` | `mjolnir123` | Secure access to DC | - |

---

## Key Theories

### 1. Load Balancing
A Load Balancer acts as a reverse proxy and distributes network or application traffic across a number of servers. This increases the concurrent user capacity and overall reliability of applications.

### 2. High Availability (HA)
HA ensures that a system remains operational even if one or more components fail. By using multiple application servers behind a load balancer, if one server goes down, others can still serve the traffic.

### 3. Nginx Upstream Module
The `upstream` directive in Nginx defines a group of servers that can be referenced by the `proxy_pass` directive. It allows Nginx to balance requests among the defined servers.

### 4. Proxy Headers
*   `Host`: Passes the original host header from the client to the backend.
*   `X-Real-IP`: Passes the real client IP to the backend server.
*   `X-Forwarded-For`: Tracks the chain of proxies the request has passed through.

---

## Implementation Steps

### Step 1: Verify and Start Apache on App Servers
Connect to each application server and ensure `httpd` is running and enabled.

```bash
# Example for stapp01
ssh tony@stapp01
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd

# Check listening port
sudo ss -tulpn | grep httpd
# Output: tcp LISTEN 0 511 *:8084 ...
```
*Repeat for `stapp02` and `stapp03`.*

### Step 2: Configure Load Balancer (stlb01)
Access the LBR server and edit the Nginx configuration.

```bash
ssh loki@stlb01
sudo vi /etc/nginx/nginx.conf
```

**Configuration Changes:**

1.  **Define Upstream Group:**
    Inside the `http` block, add:
    ```nginx
    upstream nautilus_apps {
        server stapp01:8084;
        server stapp02:8084;
        server stapp03:8084;
    }
    ```

2.  **Update Server Block:**
    Modify the `location /` block:
    ```nginx
    location / {
        proxy_pass http://nautilus_apps;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    ```

3.  **Fix Syntax Errors:**
    Ensure the `log_format` line ends with a semicolon (`;`).

### Step 3: Validate and Restart Nginx
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Step 4: Verification
```bash
curl http://stlb01:80
# Output: Welcome to xFusionCorp Industries!
```

---

## Mistakes Made & Solutions

| Mistake | Reason | Fix |
| :--- | :--- | :--- |
| **Missing Semicolon** | A syntax error in `nginx.conf` (specifically `log_format`). | Added `;` at the end of the `log_format` directive. |
| **Apache Service Down** | Apache was disabled/inactive on some app servers by default. | Ran `sudo systemctl enable --now httpd` on all app servers. |
| **Default Root Page** | The default `root /usr/share/nginx/html;` was overriding the proxy. | Commented out the `root` directive to let `proxy_pass` handle all root requests. |
| **Unknown Port** | Attempted to use port 80/8080 initially without checking. | Used `ss -tulpn` to identify that Apache was actually listening on `8084`. |
| **Command Not Found** | Tried using `netstat` which wasn't installed. | Used the modern alternative `ss` to check network sockets. |
