# Date: May 21, 2026

# Topic: Troubleshooting Nginx & PHP-FPM Multi-Container Pod Configuration

## User Credentials & Details

- **User/Host:** `thor@jump-host`
- **Pod Name:** `nginx-phpfpm`
- **ConfigMap Name:** `nginx-config`

---

## Task Description

We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue based on the provided pod and configmap.

Once resolved, copy the `/home/thor/index.php` file from the jump host to the `nginx-container` within the nginx document root. After this, you should be able to access the website.

---

## Root Cause Analysis (What were the mistakes and why they happened)

**The Mistake:**
The `nginx-config` ConfigMap defined the Nginx document root as `/var/www/html`. However, the deployment configuration for the `nginx-container` inside the `nginx-phpfpm` pod mounted the `shared-files` volume to `/usr/share/nginx/html` instead of `/var/www/html`.

**Why it happened:**
In multi-container Pods, shared volumes must be mounted to the exact paths expected by each respective container's configuration. Because the Nginx root directory mismatched the actual volume mount path, Nginx could not locate the PHP files passed by PHP-FPM, leading to the application halting/failing to serve the website.

---

## How it was fixed (Step-by-step DevOps Instructions)

### Step 1: Investigate the current state

Check the pod status and the `nginx-config` ConfigMap.

```bash
kubectl get pods
kubectl get configmap nginx-config -o yaml
```

Extract the existing pod's YAML configuration to a temporary file:

```bash
kubectl get pod nginx-phpfpm -o yaml > /tmp/nginx-phpfpm.yaml
```

### Step 2: Identify the Issue

Inspecting the ConfigMap reveals `root /var/www/html;`. However, inside `nginx-phpfpm.yaml`, the `nginx-container` volume mount is configured as:

```yaml
- mountPath: /usr/share/nginx/html
  name: shared-files
```

### Step 3: Rectify the Pod Configuration

Edit the `/tmp/nginx-phpfpm.yaml` file to correct the mount path mismatch.

```bash
vi /tmp/nginx-phpfpm.yaml
```

Change the `mountPath` for `shared-files` in the `nginx-container` block to `/var/www/html`:

```yaml
volumeMounts:
  - mountPath: /var/www/html
    name: shared-files
```

### Step 4: Recreate the Pod

Since volume mounts cannot be updated dynamically on a running Pod, force delete the existing pod and recreate it using the corrected YAML file.

```bash
kubectl delete pod nginx-phpfpm --force --grace-period=0
kubectl apply -f /tmp/nginx-phpfpm.yaml
```

Wait until the pod is fully running (2/2 containers ready):

```bash
kubectl get pods -w
```

### Step 5: Copy the Source Code to the Container

Copy the required `index.php` file from the jump host to the newly corrected document root inside the `nginx-container`.

```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container
```

### Step 6: Verification

Exec into the container to verify the file was placed successfully:

```bash
kubectl exec -it nginx-phpfpm -c nginx-container -- ls -l /var/www/html
```

Check if the web application is accessible and serving correctly on port 8099:

```bash
kubectl exec -it nginx-phpfpm -c nginx-container -- curl -I http://localhost:8099
```

_Expected Result: HTTP/1.1 200 OK_

---

## Execution Output Log

```text
thor@jump-host ~$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          113s

thor@jump-host ~$ kubectl get configmap nginx-config -o yaml
apiVersion: v1
data:
  nginx.conf: |
    events {
    }
    http {
      server {
        listen 8099 default_server;
        listen [::]:8099 default_server;

        # Set nginx to serve files from the shared volume!
        root /var/www/html;
        index  index.html index.htm index.php;
        server_name _;
        location / {
          try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          include fastcgi_params;
          fastcgi_param REQUEST_METHOD $request_method;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_pass 127.0.0.1:9000;
        }
      }
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2026-05-21T03:07:53Z"
  name: nginx-config
  namespace: default

thor@jump-host ~$ kubectl get pod nginx-phpfpm -o yaml > /tmp/nginx-phpfpm.yaml
thor@jump-host ~$ cat /tmp/nginx-phpfpm.yaml
# [Output abbreviated: Pod YAML Dump with the invalid /usr/share/nginx/html path]

thor@jump-host ~$ vi /tmp/nginx-phpfpm.yaml
# [Edited the mountPath to /var/www/html]

thor@jump-host ~$ kubectl delete pod nginx-phpfpm --force --grace-period=0
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "nginx-phpfpm" force deleted from default namespace

thor@jump-host ~$ kubectl apply -f /tmp/nginx-phpfpm.yaml
pod/nginx-phpfpm created

thor@jump-host ~$ kubectl get pods -w
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          6s
^C

thor@jump-host ~$ kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container

thor@jump-host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- ls -l /var/www/html
total 4
-rw-r--r-- 1 1000 1000 19 May 21 03:19 index.php

thor@jump-host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- curl -I http://localhost:8099
HTTP/1.1 200 OK
Server: nginx/1.31.0
Date: Thu, 21 May 2026 03:19:32 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.2.34
```
