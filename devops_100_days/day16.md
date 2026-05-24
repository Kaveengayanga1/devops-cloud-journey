# Day 16: Configuring Nginx Load Balancer for High Availability

## Task Overview
The objective was to set up a High Availability (HA) stack for a website experiencing high traffic. This involved installing and configuring an Nginx Load Balancer (LBR) on `stlb01` to distribute traffic across three backend Application Servers running Apache.

## Infrastructure Details (Static Credentials)
| Server Name | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| App Server 1 | `stapp01` | tony | `Ir0nM@n` | Nautilus Application 1 |
| App Server 2 | `stapp02` | steve | `Am3ric@` | Nautilus Application 2 |
| App Server 3 | `stapp03` | banner | `BigGr33n` | Nautilus Application 3 |
| LoadBalancer | `stlb01` | loki | `Mischi3f` | Nginx LBR |
| Jump Host | `jump-host` | thor | `mjolnir123` | Secure access gateway |

## Theoretical Concepts
*   **Load Balancing:** Distributing traffic across multiple servers to ensure no single server is overloaded.
*   **Reverse Proxy:** Nginx acts as an intermediary, taking requests from clients and forwarding them to backend Apache servers.
*   **Upstream Module:** A block in Nginx used to define the group of backend servers (`stapp01-03`).
*   **Health Checks:** Ensuring the backend Apache service is active so the Load Balancer doesn't route traffic to a dead node.

## Troubleshooting: Mistakes & Fixes
1.  **Incorrect Port Assumption:**
    *   *Mistake:* Initially used port `8080` for backend servers in the config.
    *   *Why:* Common default assumption without verification.
    *   *Fix:* Ran `sudo ss -tulpn | grep httpd` on `stapp01` and found Apache was listening on port **6300**. Updated `nginx.conf` accordingly.
2.  **Syntax Error (Missing Braces):**
    *   *Mistake:* Received `nginx: [emerg] unexpected end of file, expecting "}"`.
    *   *Why:* When manually editing `nginx.conf`, the closing brace for the `http {}` block was omitted.
    *   *Fix:* Added the missing `}` at the end of the file and verified with `sudo nginx -t`.
3.  **Typo Errors:**
    *   *Mistake:* Typed `stataus` instead of `status` during service checks.
    *   *Fix:* Corrected the spelling in the command line.

## Implementation Steps
1.  **Start Apache on App Servers:**
    ```bash
    ssh tony@stapp01 # repeat for all app servers
    sudo systemctl enable --now httpd
    ```
2.  **Configure Nginx on `stlb01`:**
    Edit `/etc/nginx/nginx.conf`:
    ```nginx
    events {
        worker_connections 1024;
    }
    http {
        upstream backend_servers {
            server stapp01:6300;
            server stapp02:6300;
            server stapp03:6300;
        }
        server {
            listen 80;
            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
    }
    ```
3.  **Verify:** `sudo nginx -t` followed by `sudo systemctl reload nginx`. Test with `curl http://stlb01:80`.