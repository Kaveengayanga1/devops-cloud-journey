# Day 06: Kubernetes Deployment Rollback (Undo a Bad Release)

## Task Description

Earlier today, the Nautilus DevOps team deployed a new application release. A customer reported a bug in the latest release, so the team needed to roll back to the previous working revision.

### Requirement

- Deployment name: `nginx-deployment`
- Action: Roll back to the previous revision
- Environment note: `kubectl` on `jump-host` is already configured to access the cluster

## Server and Access Credentials Used

- Server: `jump-host`
- User: `thor`
- Shell prompt observed: `thor@jump-host ~$`
- Kubernetes access: `kubectl` preconfigured on jump host
- Namespace used: `default` (implicit)

## Core Theory

### 1. What is a Kubernetes Deployment?

A Deployment manages the desired state for Pods through ReplicaSets. When a new image is set, Kubernetes performs a rolling update by creating new Pods and scaling down old Pods.

### 2. What is Rollout History?

Deployment updates are stored as revisions. `kubectl rollout history` lets you inspect those revisions and identify what changed.

### 3. What does `kubectl rollout undo` do?

It reverts a Deployment to the previous revision (or a specific revision if requested). Kubernetes then creates Pods that match the older ReplicaSet template.

### 4. Why verification is mandatory after rollback

A rollback command can complete successfully, but verification is still required to confirm:

- new Pods are healthy
- expected image is active
- desired replica count is preserved

## Professional Rollback Procedure

### Step 1: Check current Deployment health

```bash
kubectl get deployment nginx-deployment
kubectl get pods
```

### Step 2: Confirm currently running image

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

### Step 3: Inspect rollout history

```bash
kubectl rollout history deployment/nginx-deployment
```

### Step 4: Execute rollback

```bash
kubectl rollout undo deployment/nginx-deployment
```

### Step 5: Confirm rollout completion

```bash
kubectl rollout status deployment/nginx-deployment
```

### Step 6: Validate final state

```bash
kubectl get pods
kubectl describe deployment nginx-deployment | grep -i image
```

## Mistakes Made, Why They Happened, and How They Were Fixed

1. Mistake: A faulty image (`nginx:alpine-perl`) was promoted to production.

- Why it happened: Release validation was insufficient before rollout to all replicas.
- How it was fixed: Team checked rollout history and used `kubectl rollout undo deployment/nginx-deployment` to restore the previous stable revision.

2. Mistake: Change tracking was partially missing in old revision history.

- Evidence: Revision 1 shows `<none>` as change-cause.
- Why it happened: Earlier deployment update was done without proper change-cause recording.
- How it was fixed: New change included `--record=true`, and team should standardize release metadata capture going forward.

3. Mistake: No guarded rollout strategy for risky updates.

- Why it happened: Direct full rollout increases blast radius when a bug exists.
- How it was fixed: Immediate rollback resolved service risk; future releases should use canary/blue-green and smoke checks before full rollout.

## Additional Theory and Best Practices

- `kubectl rollout undo` without flags rolls back one revision.
- To roll back to a specific revision:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

- Use `kubectl rollout status` to wait for completion before closing incident.
- Keep release notes and incident timeline for RCA.
- Add automated checks in CI/CD:
  - image vulnerability scan
  - smoke test after deployment
  - rollback playbook and ownership

---

## Terminal Output Reference (Exact)

```text
thor@jump-host ~$ kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           99s
thor@jump-host ~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-55658c8544-d6zhm   1/1     Running   0          98s
nginx-deployment-55658c8544-kt264   1/1     Running   0          93s
nginx-deployment-55658c8544-sljsv   1/1     Running   0          94s
thor@jump-host ~$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           115s
thor@jump-host ~$ kubectl describe deployment nginx-deployment | grep -i image
                        kubernetes.io/change-cause: kubectl set image deployment nginx-deployment nginx-container=nginx:alpine-perl --record=true
    Image:         nginx:alpine-perl
thor@jump-host ~$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine-perl --record=true

thor@jump-host ~$ kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
thor@jump-host ~$ kubectl rollout status deployment/nginx-deployment
deployment "nginx-deployment" successfully rolled out
thor@jump-host ~$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-fc677cbc9-2b4x9   1/1     Running   0          20s
nginx-deployment-fc677cbc9-87qdg   1/1     Running   0          19s
nginx-deployment-fc677cbc9-bh9kr   1/1     Running   0          19s
thor@jump-host ~$ kubectl describe deployment nginx-deployment | grep -i image
    Image:         nginx:1.16
thor@jump-host ~$
```
