# Day 13: Kubernetes Services - Exposing a ReplicaSet using NodePort

**Date:** May 20, 2026
**Topic:** Kubernetes Services: Exposing Applications with NodePort
**Server:** thor@jump-host

## Task Overview

The Nautilus DevOps team already deployed a ReplicaSet to host a highly available application. The goal was to expose this application using a Kubernetes `NodePort` Service named `httpd-service`, ensuring the pods are accessible on a specific port across the cluster.

### Objectives:

- Identify labels of the existing ReplicaSet `httpd-replicaset`.
- Create a `NodePort` Service named `httpd-service`.
- Target the application on port `80`.
- Expose the Service on node port `30080`.

---

## Core Theories

### 1. Kubernetes Service

A Service is an abstraction that defines a logical set of Pods and a policy by which to access them (determined by a selector). This provides a stable IP and DNS name for the ephemeral pods.

### 2. NodePort Service

A `NodePort` service is a way to expose a service to external traffic. It opens a specific port (30000-32767) on all nodes in the cluster. Traffic sent to `<NodeIP>:<NodePort>` is routed to the corresponding service.

### 3. Labels and Selectors

Labels are key/value pairs attached to objects (like Pods). Selectors are used by Services to identify which Pods should receive traffic. For the Service to work, its selector must match the Pod labels exactly.

---

## Terminal Output & Execution

```bash
thor@jump-host ~$ kubectl get replicaset httpd-replicaset -o wide
NAME               DESIRED   CURRENT   READY   AGE     CONTAINERS        IMAGES         SELECTOR
httpd-replicaset   3         3         3       5m56s   httpd-container   httpd:latest   app=httpd_app,type=front-end

thor@jump-host ~$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
httpd-replicaset-glqmv   1/1     Running   0          6m14s   app=httpd_app,type=front-end
httpd-replicaset-rsbgg   1/1     Running   0          6m14s   app=httpd_app,type=front-end
httpd-replicaset-twpqf   1/1     Running   0          6m14s   app=httpd_app,type=front-end

thor@jump-host ~$ vi httpd-service.yaml
# (Creating the service definition)

thor@jump-host ~$ cat httpd-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
spec:
  type: NodePort
  selector:
    app: httpd_app
    type: front-end
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
      protocol: TCP

thor@jump-host ~$ kubectl apply -f httpd-service.yaml
service/httpd-service created

thor@jump-host ~$ kubectl get svc httpd-service
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
httpd-service   NodePort   10.43.13.135   <none>        80:30080/TCP   7s

thor@jump-host ~$ kubectl get endpoints httpd-service
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME            ENDPOINTS                                  AGE
httpd-service   10.22.0.10:80,10.22.0.11:80,10.22.0.9:80   36s

thor@jump-host ~$ curl http://localhost:30080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<title>It works! Apache httpd</title>
</head>
<body>
<p>It works!</p>
</body>
</html>

thor@jump-host ~$ kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
httpd-service   NodePort    10.43.13.135   <none>        80:30080/TCP   57s
kubernetes      ClusterIP   10.43.0.1      <none>        443/TCP        38m

thor@jump-host ~$ curl http://localhost:30080
# (Success)

thor@jump-host ~$ curl http://10.43.13.135:80
# (Success via ClusterIP)
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<title>It works! Apache httpd</title>
</head>
<body>
<p>It works!</p>
</body>
</html>

thor@jump-host ~$ curl http://httpd-service:80
curl: (6) Could not resolve host: httpd-service

thor@jump-host ~$ curl http://10.22.0.10:80
# (Success via Pod IP)
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<title>It works! Apache httpd</title>
</head>
<body>
<p>It works!</p>
</body>
</html>
```

---

## Analysis of Mistakes & Fixes

### 1. DNS Resolution Failure

- **Mistake:** Running `curl http://httpd-service:80` resulted in `Could not resolve host: httpd-service`.
- **Reason:** The `jump-host` (where the command was run) is usually outside the Kubernetes Pod network. While it can reach IPs (ClusterIP, Pod IP) due to network routing, it is not configured to use the internal cluster DNS (CoreDNS) for hostname resolution.
- **Fix:**
  - Verify connectivity using the **ClusterIP** (`10.43.13.135`) or **NodePort** (`30080`).
  - If hostname resolution is required from outside the cluster, proper DNS forwarding or using an Ingress/ExternalDNS would be necessary. Within a Pod in the cluster, `httpd-service` would resolve correctly.

### 2. Deprecation Warning

- **Observation:** `kubectl get endpoints` showed a warning: `v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice`.
- **Fix:** In newer Kubernetes versions, it is recommended to use `kubectl get endpointslice` to view pod connectivity details.

---
