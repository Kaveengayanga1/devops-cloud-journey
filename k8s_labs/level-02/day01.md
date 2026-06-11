# Kubernetes Multi-Container Pod with Shared Volume

**Date:** May 23, 2026  
**Topic:** Implementing emptyDir Volume Sharing Between Multiple Containers in a Kubernetes Pod  
**Lab Environment:** Nautilus DevOps Infrastructure  
**Jump Host:** thor@jump-host  
**Kubernetes Cluster:** Pre-configured and accessible via kubectl

---

## Executive Summary

This document details the successful implementation of a multi-container Kubernetes pod with shared volume configuration. The task demonstrates the fundamental concept of volume sharing between two containers within a single pod using the `emptyDir` volume type. This is a critical pattern in Kubernetes for scenarios where tightly coupled applications need to exchange temporary data.

---

## Objective

Create a Kubernetes pod named `volume-share-devops` with two containers that share a common volume, allowing both containers to read and write data to the same storage location despite having different mount paths.

### Task Requirements:

- Pod name: `volume-share-devops`
- Two containers running Debian Linux (latest)
- Shared volume of type `emptyDir`
- First container mount path: `/tmp/ecommerce`
- Second container mount path: `/tmp/demo`
- Data persistence and accessibility verification across containers

---

## Theoretical Foundation

### 1. Multi-Container Pods in Kubernetes

While Kubernetes traditionally runs a single application container per pod, the architecture supports multiple containers within one pod for tightly coupled applications. Key characteristics:

- **Shared Network Namespace:** All containers in a pod share the same IP address and network interfaces, enabling localhost communication
- **Shared IPC Namespace:** Inter-process communication mechanism between containers
- **Shared Volume Access:** Multiple containers can mount the same volume at different paths
- **Co-location:** Containers are deployed together and share the pod's lifecycle

### 2. Kubernetes Volumes Explained

Volumes in Kubernetes address the ephemeral nature of container filesystems:

- **Ephemeral Storage Problem:** Container filesystems are temporary. When a container restarts or terminates, all data is lost
- **Volume Solution:** Persistent storage that outlives individual container restarts
- **Volume Types:** Kubernetes offers multiple volume types (emptyDir, hostPath, persistentVolumeClaim, configMap, secret, etc.)

### 3. emptyDir Volume Type

The `emptyDir` volume type is specifically designed for temporary, scratch-space storage:

- **Creation:** An empty directory is created automatically when a pod is assigned to a worker node
- **Lifecycle:** The volume exists as long as the pod is running on that specific node
- **Persistence Scope:** Limited to the pod's lifespan; data is lost when the pod terminates
- **Performance:** Backed by the worker node's storage (typically faster than network-based storage)
- **Shared Access:** Multiple containers within the same pod can mount the same emptyDir volume at different paths
- **Use Cases:** Temporary caching, inter-container communication, scratch space for batch processing

### 4. Volume Mount Mechanics (Addressing Common Misconceptions)

A frequent point of confusion: Although containers mount the same volume at different paths, they access the identical underlying storage.

**Analogy:** Imagine a USB drive connected to two different computers:

- Windows system mounts it as `E:\Drive`
- macOS system mounts it as `/Volumes/MyUSB`
- The paths differ, but they represent the same physical storage

**In Kubernetes:**

- Container 1 mounts `volume-share` at `/tmp/ecommerce` (its perspective)
- Container 2 mounts `volume-share` at `/tmp/demo` (its perspective)
- Both mount paths reference the identical underlying storage on the worker node
- Linux bind mounting ensures that both containers, despite different paths, access the same inode

---

## Task Output

```bash
thor@jump-host ~$ vi pod1.yaml
thor@jump-host ~$ mv pod1.yaml pod.yaml
thor@jump-host ~$ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
spec:
  volumes:
  - name: volume-share
    emptyDir: {}
  containers:
  - name: volume-container-devops-1
    image: debian:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/ecommerce
  - name: volume-container-devops-2
    image: debian:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/demo
thor@jump-host ~$ kubectl apply -f pod.yaml
pod/volume-share-devops created
thor@jump-host ~$ kubectl get pods volume-share-devops
NAME                  READY   STATUS    RESTARTS   AGE
volume-share-devops   2/2     Running   0          32s
thor@jump-host ~$ kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/sh
# echo "Welcome to xFusionCorp Industries" > /tmp/ecommerce/ecommerce.txt
# cat /tmp/ecommerce/ecommerce.txt
Welcome to xFusionCorp Industries
# exit
thor@jump-host ~$ kubectl exec -it volume-share-devops -c volume-container-devops-2 -- cat /tmp/demo/ecommerce.txt
Welcome to xFusionCorp Industries
thor@jump-host ~$
```

---

## Credential Details

### Jump Host Credentials

- **Hostname:** jump-host
- **User:** thor
- **Shell:** bash (~/.bashrc configured)
- **Access Method:** Direct SSH/terminal access within lab environment

### Kubernetes Cluster Configuration

- **Cluster Name:** Nautilus DevOps Cluster
- **Kubectl Context:** Pre-configured
- **Authentication:** Configured via kubeconfig (no additional credentials required for kubectl commands)
- **API Server:** Accessible from jump-host

### xFusionCorp Industries (Test Organization)

- **Company:** xFusionCorp Industries (Fictional lab organization)
- **Purpose:** Used as test string content in volume verification
- **Data Status:** Non-sensitive, for lab demonstration only

**Note:** All credentials and access credentials listed above were specific to the lab environment and are no longer active.

---

## Common Mistakes and Troubleshooting

### Mistake 1: Incorrect Container Reference in kubectl exec

**What Went Wrong:**

```bash
# INCORRECT - Missing container specification
kubectl exec -it volume-share-devops -- /bin/sh
```

**Why It Happened:**
Without specifying the `-c` flag, kubectl defaults to the first container. In a multi-container pod, this leads to ambiguity and potential execution failures.

**How It Was Fixed:**

```bash
# CORRECT - Explicit container specification
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/sh
```

**Lesson:** Always explicitly specify the container with `-c <container-name>` when working with multi-container pods.

---

### Mistake 2: File Created Initially Named "pod1.yaml"

**What Went Wrong:**

```bash
thor@jump-host ~$ vi pod1.yaml  # Created with incorrect name
```

**Why It Happened:**
Initial typo or inconsistency in file naming during editor invocation.

**How It Was Fixed:**

```bash
thor@jump-host ~$ mv pod1.yaml pod.yaml  # Renamed to correct name
```

**Lesson:** Always verify file names match the deployment specification before applying configurations. Use consistent naming conventions.

---

### Mistake 3: Misunderstanding Volume Mount Paths

**What Went Wrong:**
Initial confusion: "If both containers mount the same volume at different paths, how do they share data?"

**Why It Happened:**
Common misconception treating container mount paths as separate storage entities rather than pointers to underlying shared storage.

**How It Was Clarified:**
The volume is a single storage entity on the worker node. The `volumeMounts` simply define how each container accesses this unified storage:

- Container 1 accesses it via `/tmp/ecommerce`
- Container 2 accesses it via `/tmp/demo`
- Both paths reference the same underlying storage due to Linux bind mounting

**Lesson:** Container mount paths are virtual reference points, not independent storage locations.

---

### Mistake 4: Forgetting the "latest" Tag in Container Image

**What Went Wrong:**

```yaml
# POTENTIALLY PROBLEMATIC
image: debian
```

**Why It Matters:**
Without explicit tag specification, Kubernetes defaults to "latest", which is implicit and unpredictable (latest tag can change). Production configurations should be explicit.

**How It Was Corrected:**

```yaml
# CORRECT
image: debian:latest
```

**Lesson:** Always specify explicit image tags for reproducibility and debugging purposes.

---

### Mistake 5: Insufficient Sleep Duration in Container Command

**What Went Wrong:**

```bash
command: ["/bin/sh", "-c", "sleep 60"]  # Only 1 minute
```

**Why It's a Problem:**
Short sleep duration means container exits quickly, making it difficult to exec into and troubleshoot.

**How It Was Corrected:**

```bash
command: ["/bin/sh", "-c", "sleep 3600"]  # 1 hour (3600 seconds)
```

**Lesson:** Use sufficient sleep duration in development/testing containers to allow adequate time for inspection and troubleshooting.

---

## Step-by-Step Implementation Guide

### Prerequisites Verification

- Access to jump-host with proper SSH credentials
- kubectl CLI installed and configured
- Access to Nautilus DevOps Kubernetes cluster
- Adequate permissions to create pods

### Step 1: Create YAML Configuration File

Navigate to the working directory and create the pod configuration:

```bash
thor@jump-host ~$ vi pod.yaml
```

Insert the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
spec:
  volumes:
    - name: volume-share
      emptyDir: {}
  containers:
    - name: volume-container-devops-1
      image: debian:latest
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/ecommerce
    - name: volume-container-devops-2
      image: debian:latest
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/demo
```

Save and exit the editor (`:wq` in vi).

### Step 2: Deploy the Pod to Kubernetes Cluster

Apply the configuration to the cluster:

```bash
thor@jump-host ~$ kubectl apply -f pod.yaml
pod/volume-share-devops created
```

### Step 3: Verify Pod Creation and Status

Confirm both containers are running successfully:

```bash
thor@jump-host ~$ kubectl get pods volume-share-devops
NAME                  READY   STATUS    RESTARTS   AGE
volume-share-devops   2/2     Running   0          32s
```

**Expected Output Interpretation:**

- `READY 2/2`: Both containers are running and ready
- `STATUS Running`: Pod is in operational state
- `RESTARTS 0`: No container failures have occurred
- `AGE 32s`: Pod has been running for 32 seconds (varies based on timing)

### Step 4: Execute Commands in First Container

Access the first container's shell:

```bash
thor@jump-host ~$ kubectl exec -it volume-share-devops -c volume-container-devops-1 -- /bin/sh
#
```

You are now inside the `volume-container-devops-1` container.

### Step 5: Create Test File in First Container

Within the first container's shell, create a test file in the mounted volume:

```bash
# echo "Welcome to xFusionCorp Industries" > /tmp/ecommerce/ecommerce.txt
# cat /tmp/ecommerce/ecommerce.txt
Welcome to xFusionCorp Industries
# exit
```

**Explanation:**

- The file is written to `/tmp/ecommerce`, which is mounted from the shared `volume-share` volume
- Verification with `cat` confirms successful file creation
- Exit returns to the jump-host shell

### Step 6: Verify Data Sharing in Second Container

Test that the second container can access the file created by the first container:

```bash
thor@jump-host ~$ kubectl exec -it volume-share-devops -c volume-container-devops-2 -- cat /tmp/demo/ecommerce.txt
Welcome to xFusionCorp Industries
```

**Critical Observation:**

- File was created in Container 1 at `/tmp/ecommerce/ecommerce.txt`
- Container 2 reads it from `/tmp/demo/ecommerce.txt`
- Despite different paths, the same file content is accessible
- This confirms successful volume sharing

---

## Verification Checklist

- [ ] pod.yaml file created with correct specifications
- [ ] Container images specified as `debian:latest`
- [ ] Volume name defined as `volume-share` with `emptyDir` type
- [ ] Container 1 configured with correct name and mount path
- [ ] Container 2 configured with correct name and mount path
- [ ] Pod deployed successfully with `kubectl apply`
- [ ] Pod status shows `READY 2/2` and `STATUS Running`
- [ ] File created in Container 1 at `/tmp/ecommerce/`
- [ ] File verified as readable in Container 2 at `/tmp/demo/`
- [ ] Content verified as identical across both containers

---

## Key Takeaways

1. **Multi-Container Architecture:** Kubernetes pods can host multiple containers for tightly coupled applications sharing resources and network namespace.

2. **Volume Sharing Mechanism:** The `emptyDir` volume provides temporary shared storage accessible by multiple containers regardless of mount path differences.

3. **Mount Path Abstraction:** Container mount paths are virtual access points to underlying storage. Different paths can reference identical storage via Linux bind mounting.

4. **Data Persistence Scope:** emptyDir volumes persist for the pod's lifetime on a specific node but are deleted when the pod terminates or migrates.

5. **kubectl exec with Multi-Container Pods:** Always explicitly specify the target container with `-c <container-name>` flag.

6. **Debugging Multi-Container Pods:** Verify pod status before attempting to exec into containers; ensure all containers are in the Running state.

---

## Conclusion

This task successfully demonstrates the core concepts of Kubernetes volume sharing in a multi-container pod context. The implementation shows how two containers with different mount paths can seamlessly access shared data, a pattern essential for microservice architectures utilizing sidecar containers, log shippers, and inter-application communication.

The verification of file creation in Container 1 and successful reading in Container 2 confirms the proper functioning of the emptyDir volume mechanism and validates the Kubernetes cluster configuration.

---

**Document Status:** Complete  
**Verification Status:** Successful  
**Lab Environment Status:** Active (credentials no longer valid)
