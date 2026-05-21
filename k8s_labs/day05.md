# Day 05: Kubernetes Rolling Updates and Image Management

## Task Overview

The Nautilus application development team introduced changes requiring an update to the `nginx-deployment`. The task was to perform a rolling update to change the container image from `nginx:1.16` to `nginx:1.19` without causing downtime.

### Server Credentials

- **Jump-host**: `thor@jump-host`
- **Kubernetes Cluster**: Accessible via `kubectl` on jump-host.
- **Deployment Name**: `nginx-deployment`
- **Target Image**: `nginx:1.19`

---

## Technical Theories

### 1. Kubernetes Deployment

A Deployment provides declarative updates for Pods and ReplicaSets. You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate.

### 2. Rolling Update

Rolling updates allow a Deployment's update to take place with zero downtime by incrementally updating Pods instances with new ones. New Pods will be scheduled on Nodes with available resources before old Pods are terminated.

### 3. Rolling Update Parameters

- **Max Surge**: The maximum number of Pods that can be created over the desired number of Pods.
- **Max Unavailable**: The maximum number of Pods that can be unavailable during the update process.

### 4. Container Name vs. Image Name

In a Kubernetes manifest, a container has a `name` (a unique identifier within the Pod) and an `image` (the software being run). When updating an image via CLI, you must specify the container name, not just the image name or the deployment name.

---

## Implementation Steps

### 1. Initial Status Check

Verify the deployment and the current image being used.

```bash
kubectl get deployments
kubectl get pods
kubectl describe deployment nginx-deployment
```

### 2. Update the Image

Update the container image using the `kubectl set image` command. Note that the container name is `nginx-container`.

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.19
```

### 3. Monitor the Rollout

Check the progress to ensure the update completes successfully.

```bash
kubectl rollout status deployment/nginx-deployment
```

### 4. Post-Update Verification

Confirm all pods are running with the new image.

```bash
kubectl get pods
kubectl describe deployment nginx-deployment | grep Image
```

---

## Retrospective: Mistakes and Fixes

### The Mistake

The initial attempts to update the image failed with the following error:
`error: unable to find container named "nginx"`

- **Symptom**: The command `kubectl set image deployment/nginx-deployment nginx=nginx:1.19` failed repeatedly.
- **Cause**: The command assumed the container name inside the deployment was `nginx`. However, checking the deployment details showed that the container was actually named **`nginx-container`**.

### The Fix

1. **Investigation**: Used `kubectl describe deployment nginx-deployment` to inspect the Pod Template.
2. **Identification**: Found the correct container name under the `Containers:` section:
   ```text
   Containers:
    nginx-container:
     Image: nginx:1.16
   ```
3. **Correction**: Re-ran the update command using the specific container name:
   ```bash
   kubectl set image deployment/nginx-deployment nginx-container=nginx:1.19
   ```

---

## Final Output Log

```text
deployment.apps/nginx-deployment image updated
deployment "nginx-deployment" successfully rolled out
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6655dc8cfb-b2n8r   1/1     Running   0          15s
nginx-deployment-6655dc8cfb-vggt2   1/1     Running   0          16s
nginx-deployment-6655dc8cfb-z6kvr   1/1     Running   0          20s
```

The application was successfully updated to `nginx:1.19` with zero downtime.
