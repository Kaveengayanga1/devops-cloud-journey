# Day 15: Nginx SSL Configuration on App Server 1

## Task Overview
The xFusionCorp Industries system admin team needs to deploy Nginx on App Server 1, configure it with a self-signed SSL certificate, and set up a basic index page.

## Infrastructure Details
| Server Name | IP | Hostname | User | Password | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Application Server 1 | Dynamic | stapp01 | tony | Ir0nM@n | Hosts Nautilus Application 1 |
| Jump Host Server | Dynamic | jump-host | thor | mjolnir123 | Secure Access to Stratos DC |

## Theory
1.  **Nginx**: A high-performance web server, reverse proxy, and load balancer.
2.  **SSL/TLS**: Protocols for encrypting data in transit. A `.crt` file is the public certificate, and a `.key` file is the private key.
3.  **Document Root**: The directory where web files (like `index.html`) are stored (e.g., `/usr/share/nginx/html`).

## Step-by-Step Instructions

### 1. Access App Server 1
```bash
ssh tony@stapp01
```

### 2. Install Nginx
```bash
sudo yum update -y
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 3. Handle SSL Certificates
Move existing certificates from `/tmp` to a secure location.
```bash
sudo mkdir -p /etc/nginx/ssl
sudo mv /tmp/nautilus.crt /etc/nginx/ssl/
sudo mv /tmp/nautilus.key /etc/nginx/ssl/
```

### 4. Create Index File
```bash
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```

### 5. Configure Nginx for SSL
Edit `/etc/nginx/conf.d/default.conf` or the main config:
```nginx
server {
    listen 80;
    server_name stapp01;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name stapp01;

    ssl_certificate /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 6. Verify and Restart
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### 7. Final Testing
From the Jump Host:
```bash
curl -Ik https://stapp01/
```

## Mistakes, Reasons, and Fixes

1.  **Mistake: Permission Denied during move.**
    *   **Reason:** Moving files into `/etc/nginx/` requires root privileges.
    *   **Fix:** Use `sudo` for `mkdir` and `mv` commands.

2.  **Mistake: Nginx fails to start after config change.**
    *   **Reason:** Syntax error in the configuration file or wrong paths to SSL certificates.
    *   **Fix:** Run `sudo nginx -t` to identify the line with the error and correct it.

3.  **Mistake: Curl fails with "Certificate Expired/Invalid".**
    *   **Reason:** Since it's a self-signed certificate, `curl` won't trust it by default.
    *   **Fix:** Use the `-k` or `--insecure` flag in the `curl` command to bypass certificate validation.
