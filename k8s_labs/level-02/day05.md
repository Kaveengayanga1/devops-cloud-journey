# **Date:** 2026-05-27

# **Topic:** Kubernetes Deployment Rolling Update and Rollback Validation — `httpd-deploy`

## Overview

This document captures the complete Dev environment validation for a planned production deployment update. It includes namespace creation, deployment and service setup, rolling update execution, rollback verification, command evidence, operational theory, and mistakes encountered during execution.

## Environment and credential details

- Access user: `thor`
- Access host: `jump-host`
- Login context used in terminal: `thor@jump-host`
- Kubernetes CLI: `kubectl` (already configured on jump-host as per task note)
- Target namespace: `devops`
- Workload name: `httpd-deploy`
- Service name: `httpd-service`
- Initial image: `httpd:2.4.27`
- Updated image: `httpd:2.4.43`
- Service exposure: `NodePort 30008`

Credential and secret note:

- No SSH password, private key material, kubeconfig file content, token value, or cloud credential value was provided in the recorded output.
- This document includes all credentials and access identifiers that were explicitly available in the task evidence.

## Task objective

The Nautilus DevOps team needed to validate update and rollback behavior in Dev before next week’s production deployment.

Required execution sequence:

1. Create namespace `devops`.
2. Create deployment `httpd-deploy` with 3 replicas using image `httpd:2.4.27`.
3. Use `RollingUpdate` with `maxSurge=1` and `maxUnavailable=2`.
4. Create NodePort service `httpd-service` on port `30008`.
5. Update deployment image to `httpd:2.4.43`.
6. Roll back the deployment to previous image (`httpd:2.4.27`).

## Core theory

### 1. Why RollingUpdate is used

`RollingUpdate` replaces Pods gradually instead of stopping all old Pods first. This reduces service interruption risk and allows controlled transition between revisions.

### 2. `maxSurge` and `maxUnavailable`

- `maxSurge: 1` allows one extra Pod above desired replicas during update.
- `maxUnavailable: 2` allows up to two Pods to be temporarily unavailable while rollout proceeds.

For desired replicas = 3, Kubernetes can temporarily run up to 4 Pods, and can tolerate temporary unavailability up to 2 Pods while preserving rollout progress.

### 3. Deployment revisions and rollback behavior

Each meaningful deployment template change creates a new revision. `kubectl rollout undo` promotes the previous revision as current desired state. Revision numbering may continue forward during rollback operations because rollback itself is a deployment change event.

### 4. Why strict image sequence matters

Using a wrong intermediate image can create extra revisions and distort rollout history, which may invalidate lab/assessment criteria and complicate production audit trails.

## Implementation (professional execution steps)

### Step 1: Create namespace

```bash
kubectl create namespace devops
kubectl get ns
```

### Step 2: Create deployment and service manifest

```bash
cat <<EOF > httpd-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
  namespace: devops
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  selector:
    matchLabels:
      app: httpd-app
  template:
    metadata:
      labels:
        app: httpd-app
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.27
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: devops
spec:
  type: NodePort
  selector:
    app: httpd-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
EOF
```

Apply manifest:

```bash
kubectl apply -f httpd-deployment.yaml
```

### Step 3: Verify baseline deployment

```bash
kubectl get deployments -n devops
kubectl get pods -n devops
kubectl get svc httpd-service -n devops
kubectl rollout history deployment/httpd-deploy -n devops
```

### Step 4: Perform rolling update to `httpd:2.4.43`

```bash
kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n devops
kubectl rollout history deployment/httpd-deploy -n devops
kubectl rollout status deployment/httpd-deploy -n devops
kubectl get pods -n devops -o wide
```

### Step 5: Inspect updated Pod and validate namespace context

```bash
kubectl describe pod httpd-deploy-654778b674-6886l
kubectl describe pod httpd-deploy-654778b674-6886l -n devops
```

### Step 6: Roll back to previous version

```bash
kubectl rollout undo deployment/httpd-deploy -n devops
kubectl get pods -n devops -o wide
kubectl describe deployment httpd-deploy -n devops | grep -i image
kubectl rollout history deployment/httpd-deploy -n devops
```

## Mistakes made, why they happened, and fixes

1. Mistake: Typo in command (`kubectlk` instead of `kubectl`)

- Why it happened: Manual typing error.
- Impact: Shell returned `command not found`; deployment was not applied in that attempt.
- Fix: Re-ran the exact command with correct binary name: `kubectl apply -f httpd-deployment.yaml`.

2. Mistake: Pod describe command run without namespace

- Why it happened: Default namespace context was used implicitly, while Pod existed in `devops` namespace.
- Impact: `Error from server (NotFound)` for the Pod.
- Fix: Added explicit namespace flag: `kubectl describe pod httpd-deploy-654778b674-6886l -n devops`.

## Complete recorded terminal evidence

```text
thor@jump-host ~$ kubectl create namespace devops
namespace/devops created
thor@jump-host ~$ kubectl get ns
NAME              STATUS   AGE
default           Active   59m
devops            Active   6s
kube-node-lease   Active   59m
kube-public       Active   59m
kube-system       Active   59m
thor@jump-host ~$ cat <<EOF > httpd-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
  namespace: devops
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  selector:
    matchLabels:
      app: httpd-app
  template:
    metadata:
      labels:
        app: httpd-app
    spec:
      containers:
      - name: httpd
        image: httpd:2.4.27
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: devops
spec:
  type: NodePort
  selector:
    app: httpd-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
EOF
thor@jump-host ~$ kubectlk apply -f httpd-deployment.yaml
bash: kubectlk: command not found
thor@jump-host ~$ kubectl apply -f httpd-deployment.yaml
deployment.apps/httpd-deploy created
service/httpd-service created
thor@jump-host ~$ kubectl get deployments -n devops
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
httpd-deploy   3/3     3            3           10s
thor@jump-host ~$ kubectl get pods -n devops
NAME                            READY   STATUS    RESTARTS   AGE
httpd-deploy-5c6cbbbb56-h6hhw   1/1     Running   0          18s
httpd-deploy-5c6cbbbb56-kxzvn   1/1     Running   0          18s
httpd-deploy-5c6cbbbb56-t66rd   1/1     Running   0          18s
thor@jump-host ~$ kubectl get svc httpd-service -n devops
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
httpd-service   NodePort   10.43.79.50   <none>        80:30008/TCP   24s
thor@jump-host ~$ kubectl rollout history deployment/httpd-deploy -n devops
deployment.apps/httpd-deploy
REVISION  CHANGE-CAUSE
1         <none>

thor@jump-host ~$ kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n devops
deployment.apps/httpd-deploy image updated
thor@jump-host ~$ kubectl rollout history deployment/httpd-deploy -n devops
deployment.apps/httpd-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

thor@jump-host ~$ kubectl rollout status deployment/httpd-deploy -n devops
deployment "httpd-deploy" successfully rolled out
thor@jump-host ~$ kubectl get pods -n devops -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE   READINESS GATES
httpd-deploy-654778b674-6886l   1/1     Running   0          18s   10.22.0.14   jump-host   <none>           <none>
httpd-deploy-654778b674-fqn2s   1/1     Running   0          18s   10.22.0.13   jump-host   <none>           <none>
httpd-deploy-654778b674-hk74b   1/1     Running   0          18s   10.22.0.12   jump-host   <none>           <none>
thor@jump-host ~$ kubectl describe pod httpd-deploy-654778b674-6886l
Error from server (NotFound): pods "httpd-deploy-654778b674-6886l" not found
thor@jump-host ~$ kubectl describe pod httpd-deploy-654778b674-6886l -n devops
Name:             httpd-deploy-654778b674-6886l
Namespace:        devops
Priority:         0
Service Account:  default
Node:             jump-host/10.244.240.174
Start Time:       Wed, 27 May 2026 02:33:06 +0000
Labels:           app=httpd-app
                  pod-template-hash=654778b674
Annotations:      <none>
Status:           Running
IP:               10.22.0.14
IPs:
  IP:           10.22.0.14
Controlled By:  ReplicaSet/httpd-deploy-654778b674
Containers:
  httpd:
    Container ID:   containerd://9b8409a414b498daeb215040d4528490e18a1beb1630561e87abb425467bdec0
    Image:          httpd:2.4.43
    Image ID:       docker.io/library/httpd@sha256:cd88fee4eab37f0d8cd04b06ef97285ca981c27b4d685f0321e65c5d4fd49357
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 27 May 2026 02:33:11 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-svw7h (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-svw7h:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  113s  default-scheduler  Successfully assigned devops/httpd-deploy-654778b674-6886l to jump-host
  Normal  Pulling    112s  kubelet            Pulling image "httpd:2.4.43"
  Normal  Pulled     108s  kubelet            Successfully pulled image "httpd:2.4.43" in 4.408s (4.408s including waiting). Image size: 61945609 bytes.
  Normal  Created    108s  kubelet            Created container: httpd
  Normal  Started    108s  kubelet            Started container httpd
thor@jump-host ~$ kubectl rollout undo deployment/httpd-deploy -n devops
deployment.apps/httpd-deploy rolled back
thor@jump-host ~$ kubectl get pods -n devops -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE        NOMINATED NODE   READINESS GATES
httpd-deploy-5c6cbbbb56-664rz   1/1     Running   0          23s   10.22.0.16   jump-host   <none>           <none>
httpd-deploy-5c6cbbbb56-cnl9w   1/1     Running   0          23s   10.22.0.15   jump-host   <none>           <none>
httpd-deploy-5c6cbbbb56-p5qrc   1/1     Running   0          23s   10.22.0.17   jump-host   <none>           <none>
thor@jump-host ~$ kubectl describe deployment httpd-deploy -n devops | grep -i image
    Image:         httpd:2.4.27
thor@jump-host ~$ kubectl rollout history deployment/httpd-deploy -n devops
deployment.apps/httpd-deploy
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

thor@jump-host ~$
```

## Final validation result

- Namespace `devops` created successfully.
- Deployment `httpd-deploy` created with 3 replicas and rolling update settings as required.
- Service `httpd-service` exposed on NodePort `30008`.
- Update to `httpd:2.4.43` completed successfully.
- Rollback completed successfully.
- Final active deployment image confirmed as `httpd:2.4.27`.

## File created

- `day05.md`
