**Date:** 2026-05-25
**Topic:** Kubernetes — Deploying a Highly Available Nginx Static Website (Deployment + NodePort Service)

## Overview

This document describes how to deploy a static website on a Kubernetes cluster using an Nginx Deployment with 3 replicas and a NodePort Service (nodePort 30011). It includes the manifest, commands executed, observed outputs, credentials and environment details used during the exercise (marked as no-longer-valid), common mistakes encountered, root causes, and how they were fixed. The content is written for professional DevOps/cloud engineers.

## Environment & used credentials (no longer valid)

- Jump host SSH user: thor (example): thor@jump-host
- Jump host internal IP: 10.244.97.143
- Kubernetes: k3s, version: v1.34.1+k3s1
- Cluster node (example): jump-host (control-plane)
- Cluster service ClusterIP used in test: 10.43.112.161
- NodePort used: 30011
- kubeconfig context: /home/thor/.kube/config (example path)
- SSH key: id_rsa_thor (listed for audit; keys/passwords are expired and not valid now)

Note: All credentials and keys listed above are for documentation purposes and are no longer valid.

## Manifest (nginx-deployment.yaml)

The single-file manifest used to create the Deployment and Service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30011
```

## Commands executed (recorded session)

vi nginx-deployment.yaml
cat nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml

Observed kubectl output (example):

```
deployment.apps/nginx-deployment created
service/nginx-service created
```

## Verification commands and sample outputs

kubectl get deployments nginx-deployment

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           15s
```

kubectl get pods -l app=nginx-app

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5bdb4f8cbb-7cvd7   1/1     Running   0          42s
nginx-deployment-5bdb4f8cbb-d5m54   1/1     Running   0          42s
nginx-deployment-5bdb4f8cbb-pfnn5   1/1     Running   0          42s
```

kubectl get svc nginx-service

```
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.43.112.161   <none>        80:30011/TCP   48s
```

curl http://localhost:30011 (on node)

```
<!DOCTYPE html>
<html>
  ... Welcome to nginx! ...
</html>
```

curl http://10.43.112.161:80

```
<!DOCTYPE html>
<html>
  ... Welcome to nginx! ...
</html>
```

curl -I http://localhost:30011

```
HTTP/1.1 200 OK
Server: nginx/1.31.1
Date: Mon, 25 May 2026 04:19:08 GMT
Content-Type: text/html
Content-Length: 896
```

## Key considerations & professional notes

- Image tag: `nginx:latest` was requested. For production environments prefer an explicit, immutable tag (for example `nginx:1.31.1`) and use an image-pull policy appropriate for your release process.
- NodePort range: NodePort 30011 is within the default allowed range (30000-32767). Ensure it does not collide with other services.
- Security: Exposing NodePort on control-plane nodes may expose services to networks; consider using an Ingress, LoadBalancer or ClusterIP + ingress controller for production.
- Readiness and liveness probes: Add them for safer rolling updates and traffic control.
- Resource requests/limits: Define requests/limits to allow proper bin-packing and QoS classification.
- Network policies, RBAC, and PodSecurity admission: evaluate before exposing services externally.

## Common mistakes observed, root causes, and fixes

1. Service not routing to Pods (symptom): Service shows as created but curl returns connection refused or 503.
   - Root cause: Label selector mismatch between Service `selector` and Deployment pod labels (e.g., typo in label key or value).
   - Fix: Ensure `metadata.labels` on the Deployment template exactly match the Service `selector`. Example: both use `app: nginx-app`.

2. Manifest YAML syntax or stray keys (symptom): `kubectl apply` fails with a validation error or unexpected object fields.
   - Root cause: Incorrect indentation or stray keys (for example an accidental `source:` line before `spec:`) or mixing tabs and spaces.
   - Fix: Validate YAML indentation and remove stray keys. Use `kubectl apply --dry-run=client -f file.yaml` and `kubectl explain <resource>` to check.

3. NodePort collisions or out-of-range (symptom): `kubectl apply` fails or service can't allocate requested nodePort.
   - Root cause: Requested nodePort already in use or outside allowed range.
   - Fix: Either omit `nodePort` and let Kubernetes assign one, or choose a free port within the default range 30000-32767 and coordinate with other services.

4. Using `nginx:latest` in production (issue): Unpredictable upgrades when base image changes.
   - Root cause: `latest` is mutable and will change when image is rebuilt.
   - Fix: Pin to a specific tag and use an image promotion pipeline; for emergency fixes you can temporarily use `:latest` but document and revert.

5. No readiness probe (symptom): During rolling updates, pods receive traffic before they are ready causing errors.
   - Fix: Add a readiness probe (HTTP GET /) and a liveness probe to the container spec.

## Recommendations (professional)

- Use declarative manifests stored in Git and apply via CI/CD (gitops) rather than ad-hoc kubectl on jump-host.
- Add resource requests/limits and probes to the deployment.
- For production, prefer `ClusterIP + Ingress` or a cloud `LoadBalancer` service rather than NodePort for external traffic.
- Implement health checks, monitoring (Prometheus), and logging (ELK/Fluentd) for the deployed application.

Example: improved Deployment snippet (add probes and resources)

```yaml
containers:
  - name: nginx-container
    image: nginx:1.31.1
    ports:
      - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 20
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

## Appendix: Quick commands to re-run the example

```bash
# Apply manifest
kubectl apply -f nginx-deployment.yaml

# Verify
kubectl get deployments nginx-deployment
kubectl get pods -l app=nginx-app
kubectl get svc nginx-service

# From a node (or jump-host) test NodePort
curl http://localhost:30011

# Test ClusterIP (internal)
curl http://10.43.112.161:80

# Inspect headers
curl -I http://localhost:30011
```

## Files created

- [day03.md](/mnt/Local Disk D/DevOps/kodekloud/k8s_labs/day03.md)

If you want, I can: (a) add the recommended probes/resources directly into the original `nginx-deployment.yaml` and re-run a dry-run check, or (b) commit these files to your repo. Which would you like next?
