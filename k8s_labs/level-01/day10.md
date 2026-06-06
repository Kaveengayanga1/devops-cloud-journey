# Day 10: Kubernetes ConfigMaps, Volumes, and Namespaces

## Task Overview

The Nautilus DevOps team required a `time-check` pod in the `datacenter` namespace for logging purposes. This task involved creating a ConfigMap for configuration data, setting up a Pod with environment variables derived from that ConfigMap, and mounting a volume for persistent logging.

### Server Credentials

- **Jump-host**: `thor@jump-host`
- **Kubernetes Cluster**: Accessible via `kubectl` on jump-host.
- **Namespaces involved**: `nautilus` (initial mistake), `datacenter` (target).

---

## Technical Theories

### 1. Namespaces

Namespaces are virtual clusters backed by the same physical cluster. They provide a scope for names and a way to divide cluster resources between multiple users or projects.

### 2. ConfigMaps

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

### 3. Pods & Containers

A Pod is the smallest deployable unit in Kubernetes, which can contain one or more containers. In this task, we use the `busybox:latest` image.

### 4. Environment Variables

Environment variables are a way to pass configuration settings into the container runtime. Here, `TIME_FREQ` is injected into the container from a ConfigMap.

### 5. Volumes (emptyDir)

An `emptyDir` volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that node. It is initially empty and used here to store the log file `/opt/sysops/time/time-check.log`.

---

## Implementation Steps

### 1. Create the Namespace

```bash
kubectl create namespace datacenter
```

### 2. Create the ConfigMap

Create a file named `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: datacenter
data:
  TIME_FREQ: "2"
```

Apply it:

```bash
kubectl apply -f configmap.yaml
```

### 3. Create the Pod Manifest

Create a file named `manifest.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: datacenter
spec:
  containers:
    - name: time-check
      image: busybox:latest
      command: ["/bin/sh", "-c"]
      args:
        - "while true; do date; sleep $TIME_FREQ; done > /opt/sysops/time/time-check.log"
      env:
        - name: TIME_FREQ
          valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
      volumeMounts:
        - name: log-volume
          mountPath: /opt/sysops/time
  volumes:
    - name: log-volume
      emptyDir: {}
```

Apply it:

```bash
kubectl apply -f manifest.yaml
```

### 4. Verification

Check if the pod is running and verify the logs:

```bash
kubectl get pods -n datacenter
kubectl exec -it time-check -n datacenter -- tail -f /opt/sysops/time/time-check.log
```

---

## Retrospective: Mistakes and Fixes

### The Mistake

Initially, the resources (ConfigMap and Pod) were created in the **`nautilus`** namespace instead of the required **`datacenter`** namespace.

- **Symptom**: Running `kubectl exec -it time-check -n datacenter` resulted in a `NotFound` error because the pod existed in `nautilus`.
- **Cause**: The `metadata.namespace` field in the YAML files was hardcoded to `nautilus`.

### The Fix

1. **Cleanup**: Deleted the incorrect resources from the wrong namespace.
   ```bash
   kubectl delete pod time-check -n nautilus
   kubectl delete configmap time-config -n nautilus
   ```
2. **Correction**: Updated the `namespace` field in both `configmap.yaml` and `manifest.yaml` to `datacenter`.
3. **Redeploy**: Re-applied the corrected manifests.

---

## Final Output Log

```text
Mon May 11 02:52:37 UTC 2026
Mon May 11 02:52:39 UTC 2026
Mon May 11 02:52:41 UTC 2026
Mon May 11 02:52:43 UTC 2026
```

The logs confirm that the command is executing every 2 seconds as defined in the ConfigMap.
