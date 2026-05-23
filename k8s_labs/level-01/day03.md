# Day 03: Kubernetes Namespaces and Pod Deployment

## Task Overview
The Nautilus DevOps team needed to deploy an Nginx pod within a specific namespace named `dev`.

### Requirements:
- Create a namespace named `dev`.
- Deploy a pod named `dev-nginx-pod`.
- Use the `nginx:latest` image.
- Ensure the pod is running in the `dev` namespace.

---

## Server Credentials
- **Server**: `jump-host`
- **User**: `thor`
- **Kubeconfig**: Pre-configured on `jump-host`

---

## Theory

### 1. Namespace
Namespaces are virtual clusters backed by the same physical cluster. They are used to divide cluster resources between multiple users or projects (e.g., dev, staging, production).

### 2. Pod
A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster and can contain one or more containers.

### 3. Kubectl
The command-line tool for communicating with a Kubernetes cluster's control plane, using the Kubernetes API.

### 4. YAML Configuration
Defining Kubernetes objects via YAML files is the industry standard (declarative approach) as it allows version control and easier management of complex configurations.

---

## Execution and Troubleshooting

### Step 1: Check existing resources
```bash
thor@jump-host ~$ kubectl get all
```

### Step 2: Create the namespace
```bash
thor@jump-host ~$ kubectl create namespace dev
```

### Step 3: Create the Pod YAML file
Initially, some mistakes were made in the YAML syntax.

**Mistake 1: Typo in command**
- **Command**: `ci dev-nginx-pod.yml`
- **Error**: `bash: ci: command not found`
- **Reason**: Intended to use `vi` or `vim` but typed `ci`.
- **Fix**: Used `vi dev-nginx-pod.yml`.

**Mistake 2: Syntax errors in YAML**
```yaml
piVersion: v1
kind: Pos
metadate:
# ...
```
- **Errors**: `piVersion` instead of `apiVersion`, `Pos` instead of `Pod`, `metadate` instead of `metadata`.
- **Reason**: Fast typing/manual entry errors.
- **Fix**: Corrected the YAML content.

**Corrected YAML (`dev-nginx-pod.yml`):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx-pod
  namespace: dev
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Step 4: Apply the configuration
**Mistake 3: File extension mismatch**
- **Command**: `kubectl apply -f dev-nginx-pod.yaml`
- **Error**: `error: the path "dev-nginx-pod.yaml" does not exist`
- **Reason**: The file was created as `.yml` but requested as `.yaml`.
- **Fix**: Used the correct filename.

```bash
thor@jump-host ~$ kubectl apply -f dev-nginx-pod.yml
# Output: pod/dev-nginx-pod created
```

### Step 5: Verification
Check if the namespace and pod are active.
```bash
thor@jump-host ~$ kubectl get namespaces
thor@jump-host ~$ kubectl get pods -n dev
thor@jump-host ~$ kubectl describe pods dev-nginx-pod -n dev
```

---

## Summary of Mistakes & Fixes
1. **Wrong Command**: Tried `ci` instead of `vi`. Fixed by using `vi`.
2. **YAML Typos**: Used `piVersion`, `Pos`, and `metadate`. Fixed by correcting to `apiVersion`, `Pod`, and `metadata`.
3. **File Path Error**: Tried to apply `dev-nginx-pod.yaml` when the file was `dev-nginx-pod.yml`. Fixed by using the correct extension.
