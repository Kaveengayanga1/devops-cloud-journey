# Day 04: Kubernetes Resource Limits and Requests

## Task Description
The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization for a specific pod.

### Requirements:
- **Pod Name:** `httpd-pod`
- **Container Name:** `httpd-container`
- **Image:** `httpd:latest`
- **Resource Requests:** Memory: `15Mi`, CPU: `100m`
- **Resource Limits:** Memory: `20Mi`, CPU: `100m`

## Server Credentials
- **Server:** `jump-host`
- **User:** `thor`
- **Cluster Management:** `kubectl` (already configured)

## Theory: Resource Management in Kubernetes

### 1. Resource Requests vs Limits
- **Requests:** The minimum amount of resources (CPU/Memory) that Kubernetes guarantees to a container. The scheduler uses this value to decide which node to place the pod on.
- **Limits:** The maximum amount of resources a container is allowed to use. 
    - If a container exceeds its **CPU limit**, it is throttled (slowed down).
    - If a container exceeds its **Memory limit**, it may be terminated with an **OOMKilled** (Out Of Memory) error.

### 2. Units of Measurement
- **CPU:** Measured in millicores (`m`). `1000m` is equal to 1 vCPU/Core. So, `100m` is 0.1 of a CPU core.
- **Memory:** Measured in bytes. Common suffixes are `Mi` (Mebibytes) and `Gi` (Gibibytes). Note that `1Mi = 1,024 * 1,024` bytes.

## Implementation Steps

### Step 1: Create the Pod Manifest
Create a file named `httpd-pod.yaml`:
```bash
vi httpd-pod.yaml
```

**Manifest Content:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"
```

### Step 2: Apply the Configuration
```bash
kubectl apply -f httpd-pod.yaml
```

### Step 3: Verify the Pod and Resources
Check if the pod is running:
```bash
kubectl get pods
```

Detailed verification of resource limits:
```bash
kubectl describe pod httpd-pod
```

## Troubleshooting and Mistakes
1. **Mistake:** Setting `limits` lower than `requests`.
   - **Why:** Kubernetes will not allow a pod to request more than its limit.
   - **Fix:** Always ensure `limits >= requests`.
2. **Mistake:** Incorrect Unit Suffixes (e.g., using `15M` instead of `15Mi`).
   - **Why:** `M` is Megabytes (decimal), while `Mi` is Mebibytes (binary). Kubernetes prefers binary units for memory.
   - **Fix:** Use `Mi` for precise memory allocation.
3. **Internal Error (Hypothetical during task):** Container stuck in `ContainerCreating`.
   - **Reason:** Usually due to image pulling or insufficient resources on nodes.
   - **Fix:** Use `kubectl describe` to check events and verify node capacity.

---

## Terminal Output Reference
```text
thor@jump-host ~$ cat httpd-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    app: httpd
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
    resources:
      requests:
        memory: "15Mi"
        cpu: "100m"
      limits:
        memory: "20Mi"
        cpu: "100m"

thor@jump-host ~$ kubectl apply -f httpd-pod.yaml
pod/httpd-pod created

thor@jump-host ~$ kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
httpd-pod   0/1     ContainerCreating   0          5s

thor@jump-host ~$ kubectl describe pod httpd-pod
...
    Limits:
      cpu:     100m
      memory:  20Mi
    Requests:
      cpu:        100m
      memory:     15Mi
...
Events:
  Normal  Scheduled  11s   default-scheduler  Successfully assigned default/httpd-pod to jump-host
  Normal  Pulling    10s   kubelet            Pulling image "httpd:latest"
  Normal  Pulled     7s    kubelet            Successfully pulled image "httpd:latest" in 3.133s
  Normal  Created    7s    kubelet            Created container: httpd-container
  Normal  Started    7s    kubelet            Started container httpd-container
```
