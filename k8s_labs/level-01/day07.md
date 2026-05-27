# Day 07: Kubernetes ReplicaSet Management

## Task Overview
The Nautilus DevOps team objective was to deploy a `ReplicaSet` on a Kubernetes cluster to ensure application availability and scaling.

### Requirements:
- **ReplicaSet Name**: `nginx-replicaset`
- **Image**: `nginx:latest`
- **Container Name**: `nginx-container`
- **Replica Count**: 4
- **Labels**: 
  - `app: nginx_app`
  - `type: front-end`

## Technical Theory
### 1. What is a ReplicaSet?
A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. It is often used to guarantee the availability of a specified number of identical Pods.

### 2. Labels and Selectors
Kubernetes uses labels as key-value pairs attached to objects. The ReplicaSet uses a **selector** to identify which Pods it acquired. These labels must match the ones defined in the Pod template.

### 3. Pod Template
This is the specification for the Pods that the ReplicaSet will create if the current count doesn't match the desired state.

## Implementation Details

### Server Credentials (Reference)
- **Host**: `thor@jump-host`
- **Tool**: `kubectl` (pre-configured)

### Manifest File: `nginx-rs.yaml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx_app
      type: front-end
  template:
    metadata:
      labels:
        app: nginx_app
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

## Execution Steps & Output
1. **Create the manifest**:
   ```bash
   vi nginx-rs.yaml
   ```
2. **Apply the configuration**:
   ```bash
   kubectl apply -f nginx-rs.yaml
   # Output: replicaset.apps/nginx-replicaset created
   ```
3. **Verify the status**:
   ```bash
   kubectl get rs
   ```

## Mistakes and Troubleshooting
During the execution, the following error occurred:
- **Mistake**: `kubectl get re`
- **Error**: `error: the server doesn't have a resource type "re"`
- **Why it happened**: Attempted to use an invalid shorthand for `replicasets`. Kubernetes supports `rs` but not `re`.
- **Fix**: Used the correct shorthand `rs`:
  ```bash
  kubectl get rs
  ```

## Final Verification
```bash
thor@jump-host ~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-96dhr   1/1     Running   0          15s
nginx-replicaset-ldb52   1/1     Running   0          15s
nginx-replicaset-pts6w   1/1     Running   0          15s
nginx-replicaset-rdmxm   1/1     Running   0          15s
```
All 4 replicas are successfully running.
```
