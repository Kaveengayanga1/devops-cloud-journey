# Day 11: Troubleshooting Kubernetes Pods (ImagePullBackOff & Sidecar Processes)

## Task Overview

A junior DevOps team member encountered issues deploying a multi-container stack on a Kubernetes cluster. The pod, named `webserver`, failed to start correctly. The goal was to identify the root causes (specifically related to image naming and container lifecycle) and rectify them to ensure the application is running and accessible.

### Server Credentials

- **Jump-host**: `thor@jump-host`
- **Kubernetes Cluster**: Configured on `jump-host`.
- **Node**: `jump-host/10.244.73.244`
- **Pod IP**: `10.22.0.9`

---

## Technical Theories

### 1. Pods and Multi-Container Design

A Pod is the smallest execution unit in Kubernetes. It can host multiple containers that share the same network namespace and storage volumes. In this task, `httpd-container` serves as the primary web server, while `sidecar-container` acts as a helper for logging.

### 2. Sidecar Container Pattern

Sidecar containers extend or enhance the functionality of a main container (e.g., log forwarding, monitoring, or networking proxies). They run in parallel within the same Pod.

### 3. Container Lifecycle and Foreground Processes

For a container to remain in a `Running` state, it must execute a continuous foreground process. If a container (like a basic Ubuntu image) starts and finishes its command (or has no command), it exits. Kubernetes then attempts to restart it, often leading to a `CrashLoopBackOff`.

### 4. ImagePullBackOff Error

This error occurs when Kubernetes cannot retrieve the specified container image from the registry. Common reasons include:

- Incorrect image name or tag.
- Private registry authentication issues.
- Network connectivity problems.

---

## Troubleshooting and Mistakes Identified

### Mistake 1: Incorrect Image Tag

- **Issue**: The `httpd-container` was configured with the image `httpd:latests`.
- **Why**: A typo was made in the tag (`latests` instead of `latest`).
- **Symptom**: `kubectl get pods` showed `ImagePullBackOff`. `kubectl describe pod` confirmed: `Failed to pull image "httpd:latests": ... not found`.

### Mistake 2: Missing Foreground Process in Sidecar

- **Observation**: While the sidecar in the output was running a specific command, typically in these scenarios, if an OS image like `ubuntu:latest` is used without a long-running command, it exits immediately.
- **Fix**: Ensuring the sidecar has a command like `while true; do sleep 30; done` or a log-tailing script keeps it alive.

---

## Implementation Steps

### 1. Inspect the Pod Status

```bash
thor@jump-host ~$ kubectl get pods
NAME        READY   STATUS             RESTARTS   AGE
webserver   1/2     ImagePullBackOff   0          2m8s
```

### 2. Describe the Pod to Identify Errors

```bash
thor@jump-host ~$ kubectl describe pods webserver
# Output revealed:
# Warning  Failed  ...  Failed to pull image "httpd:latests": ... not found
```

### 3. Rectify the Configuration

Edit the pod directly to fix the image tag and ensure the sidecar command is correct.

```bash
thor@jump-host ~$ kubectl edit pods webserver
```

**Corrected YAML section:**

```yaml
spec:
  containers:
  - name: httpd-container
    image: httpd:latest  # Fixed typo from 'latests' to 'latest'
    ...
  - name: sidecar-container
    image: ubuntu:latest
    command:
      - sh
      - -c
      - "while true; do cat /var/log/httpd/access.log /var/log/httpd/error.log; sleep 30; done"
```

### 4. Verify Final State

```bash
thor@jump-host ~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          5m58s
```

### 5. Validate Application Access

Use port-forwarding to test the web server.

```bash
thor@jump-host ~$ kubectl port-forward pod/webserver 8080:80
# In another terminal or after backgrounding:
thor@jump-host ~$ curl http://localhost:8080
# Output: <html><body><h1>It works!</h1></body></html>
```
