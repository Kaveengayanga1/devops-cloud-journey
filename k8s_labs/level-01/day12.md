# Day 12: Updating Kubernetes Deployments and Services

**Date:** May 13, 2026
**Topic:** Kubernetes Deployment Updates, Scaling, and Service Patching
**Server:** thor@jump-host

## Task Overview

The Nautilus application development team required updates to an existing application deployed on the Kubernetes cluster. The task involved modifying a service's NodePort, scaling a deployment, and updating a container image without deleting the existing resources.

### Objectives:

1.  Modify `nginx-service` NodePort from `30008` to `32165`.
2.  Scale `nginx-deployment` replicas from `1` to `5`.
3.  Update the image from `nginx:1.18` to `nginx:latest`.

---

## Core Theories

### 1. Kubernetes Deployment

A Deployment provides declarative updates for Pods and ReplicaSets. It allows you to describe a desired state, and the Deployment Controller changes the actual state to the desired state at a controlled rate. This is ideal for rolling updates where new pods are gradually deployed while old ones are terminated, ensuring zero downtime.

### 2. Scaling (Replicas)

Scaling is the process of increasing or decreasing the number of replicas (running instances of a Pod). Increasing replicas helps in handling higher traffic and ensures high availability/resiliency of the application.

### 3. Kubernetes Service (NodePort)

A Service is an abstract way to expose an application running on a set of Pods as a network service. A `NodePort` service exposes the Service on each Node's IP at a static port. Traffic sent to the NodePort is automatically routed to the Service.

---

## Implementation Steps

### Step 1: Initial Status Check

Verify the current state of the deployment and service.

```bash
kubectl get deployment nginx-deployment
kubectl get svc nginx-service
```

### Step 2: Patch Service NodePort (30008 → 32165)

Using `kubectl patch` to update the specific field in the service specification.

```bash
kubectl patch svc nginx-service -p '{"spec":{"ports":[{"nodePort":32165,"port":80,"targetPort":80}]}}'
```

### Step 3: Scale Deployment Replicas (1 → 5)

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

### Step 4: Update Container Image (nginx:1.18 → nginx:latest)

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:latest
```

### Step 5: Verification

```bash
kubectl describe deployment nginx-deployment
kubectl describe svc nginx-service
```

---

## Troubleshooting: Mistakes Made & Fixes

### 1. Incorrect Container Name during Image Update

- **Mistake:** Executed `kubectl set image deployment/nginx-deployment nginx=nginx:latest`.
- **Reason:** Assumed the container name was `nginx`, but it was actually `nginx-container`.
- **Error Message:** `error: unable to find container named "nginx"`.
- **Fix:** Checked the actual container name using `describe` or `jsonpath`:
  ```bash
  kubectl describe deployment nginx-deployment | grep -i container -A 2
  ```
  Identified name as `nginx-container` and re-ran:
  ```bash
  kubectl set image deployment/nginx-deployment nginx-container=nginx:latest
  ```

### 2. Incorrect Resource Specification in Describe

- **Mistake:** Executed `kubectl describe nginx-deployment`.
- **Reason:** Forgot to specify the resource type (`deployment`).
- **Error Message:** `error: the server doesn't have a resource type "nginx-deployment"`.
- **Fix:** Explicitly stated the resource type:
  ```bash
  kubectl describe deployment nginx-deployment
  ```

---

## Final Verification Summary

- **Service:** NodePort successfully updated to `32165`.
- **Scaling:** 5 replicas are desired and available.
- **Image:** Container `nginx-container` is running `nginx:latest`.
- **Rollout:** History shows revision 2 as the latest update.

```bash
kubectl rollout history deployment/nginx-deployment
```
