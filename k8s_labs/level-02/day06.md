# Day 06: Jenkins CI Server Deployment on Kubernetes

**Date:** 2026-06-02  
**Topic:** Jenkins CI/CD Server Deployment on Kubernetes Cluster  
**Project:** Nautilus DevOps Infrastructure  
**Environment:** Kubernetes Cluster (K3s v1.34.1)

---

## Server Credentials (Historical)

**Kubernetes Cluster:**

- Host: jump-host (Control-plane)
- Internal IP: 10.244.195.234
- Kubernetes Version: v1.34.1+k3s1
- Container Runtime: containerd v1.6.8

**Jenkins Service:**

- Service Name: jenkins-service
- Service Type: NodePort
- Internal Port: 8080
- External Port (NodePort): 30008
- Namespace: jenkins
- Access URL: http://10.244.195.234:30008

**Jenkins Deployment Configuration:**

- Deployment Name: jenkins-deployment
- Container Name: jenkins-container
- Container Image: jenkins/jenkins:latest
- Container Port: 8080
- Replicas: 1
- Java Options: -Djenkins.install.runSetupWizard=false (Setup Wizard disabled)

---

## Task Description

The Nautilus DevOps team required setting up a Jenkins CI/CD server on a Kubernetes cluster to manage deployment pipelines for multiple projects. The task involved:

1. Creating a dedicated `jenkins` namespace for resource isolation
2. Deploying a Jenkins Service of type NodePort with port 30008 for external access
3. Creating a Jenkins Deployment with proper configuration and environment variables
4. Verifying that the Jenkins UI is accessible and running correctly

---

## Task Output

### Step 1: Create Namespace

```bash
$ kubectl create namespace jenkins
namespace/jenkins created
```

### Step 2: Create Service YAML Configuration File

```bash
$ vi jenkins-service.yaml
$ cat jenkins-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: jenkins # Deployed under the 'jenkins' namespace
spec:
  type: NodePort
  ports:
    - port: 8080       # Port exposed inside the cluster
      targetPort: 8080 # Port the container is listening on
      nodePort: 30008  # Port opened on all Kubernetes nodes for external access
  selector:
    app: jenkins       # Routes traffic to pods matching this label
```

### Step 3: Create Deployment YAML Configuration File

```bash
$ vi jenkins-deployment.yaml
$ cat jenkins-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  namespace: jenkins # Deployed under the 'jenkins' namespace
  labels:
    app: jenkins
spec:
  replicas: 1 # Number of Jenkins instances to run
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins # Label used by the Service to find this pod
    spec:
      containers:
        - name: jenkins-container
          image: jenkins/jenkins:latest # Using the specified Jenkins image
          ports:
            - containerPort: 8080 # Port where Jenkins web UI is accessible inside container
          env:
            - name: JAVA_OPTS
              value: "-Djenkins.install.runSetupWizard=false" # Skips the initial wizard
```

### Step 4: Apply Configuration to Kubernetes Cluster

```bash
$ kubectl apply -f jenkins-service.yaml
service/jenkins-service created

$ kubectl apply -f jenkins-deployment.yaml
deployment.apps/jenkins-deployment created
```

### Step 5: Verify Service Creation

```bash
$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   36m
```

### Step 6: Verify All Resources in Jenkins Namespace

```bash
$ kubectl get all -n jenkins
NAME                                      READY   STATUS    RESTARTS   AGE
pod/jenkins-deployment-59bd7d5977-98xkq   1/1     Running   0          17s

NAME                      TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
service/jenkins-service   NodePort   10.43.116.4   <none>        8080:30008/TCP   27s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins-deployment   1/1     1            1           17s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-deployment-59bd7d5977   1         1         1       17s
```

### Step 7: Check Storage Classes

```bash
$ kubectl get sc
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  36m
```

### Step 8: Verify Default Namespace (No Resources)

```bash
$ kubectl get deployments
No resources found in default namespace.

$ kubectl get deployments jenkins
Error from server (NotFound): deployments.apps "jenkins" not found
```

### Step 9: Verify Jenkins Deployment in Correct Namespace

```bash
$ kubectl get deployments -n jenkins
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
jenkins-deployment   1/1     1            1           84s

$ kubectl get svc -n jenkins
NAME              TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
jenkins-service   NodePort   10.43.116.4   <none>        8080:30008/TCP   105s
```

### Step 10: Wait for Pod Readiness

```bash
$ kubectl wait --namespace jenkins --for=condition=ready pod -l app=jenkins --timeout=90s
pod/jenkins-deployment-59bd7d5977-98xkq condition met
```

### Step 11: Get Node Information

```bash
$ kubectl get nodes -o wide
NAME        STATUS   ROLES           AGE   VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
jump-host   Ready    control-plane   38m   v1.34.1+k3s1   10.244.195.234   <none>        Alpine Linux v3.16   6.8.0-94-generic   containerd://1.6.8
```

### Step 12: Extract Node Internal IP Address

```bash
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
10.244.195.234
```

### Step 13: Verify Jenkins Service Accessibility via curl

```bash
$ curl -I http://10.244.195.234:30008
HTTP/1.1 200 OK
Server: Jetty(12.1.8)
Date: Tue, 02 Jun 2026 03:27:23 GMT
X-Content-Type-Options: nosniff
Content-Security-Policy-Report-Only: base-uri 'none'; default-src 'self'; form-action 'self'; frame-ancestors 'self'; img-src 'self' data:; script-src 'report-sample' 'self' usage.jenkins.io; style-src 'report-sample' 'self' 'unsafe-inline'; report-to content-security-policy; report-uri http://10.244.195.234:30008/content-security-policy-reporting-endpoint/lxnlIpPzTMMlF4y_3cUmQHPjc1ZeBUeir9Ohy03q-D8=:YW5vbnltb3Vz:aHVkc29uLm1vZGVsLkFsbFZpZXc=:
Content-Type: text/html;charset=utf-8
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Cache-Control: no-cache,no-store,must-revalidate
X-Hudson-Theme: default
Referrer-Policy: same-origin
Cross-Origin-Opener-Policy: same-origin
Set-Cookie: JSESSIONID.d9bb38e2=node0l8pencxa9q0r1gl4vhjrj86mz2.node0; Path=/; HttpOnly; SameSite=Lax
X-Hudson: 1.395
X-Jenkins: 2.566
X-Jenkins-Session: 2d55e959
X-Frame-Options: sameorigin
Reporting-Endpoints: content-security-policy: http://10.244.195.234:30008/content-security-policy-reporting-endpoint/lxnlIpPzTMMlF4y_3cUmQHPjc1ZeBUeir9Ohy03q-D8=:YW5vbnltb3Vz:aHVkc29uLm1vZGVsLkFsbFZpZXc=:
Transfer-Encoding: chunked
```

---

## Theories

### 1. Kubernetes Namespace Concept

A **Namespace** in Kubernetes is a logical grouping mechanism that acts as a virtual cluster within a single physical Kubernetes cluster. Namespaces provide:

- **Resource Isolation:** Allows multiple teams or projects to share a cluster without conflicts
- **Resource Quotas:** Enable limiting CPU, memory, and storage per namespace
- **RBAC (Role-Based Access Control):** Facilitates fine-grained access control per namespace
- **Naming Scope:** Enables resource names to be unique only within a namespace, not globally

In this deployment, the `jenkins` namespace isolates all Jenkins-related resources from the default namespace and other applications.

### 2. Kubernetes Deployment Architecture

A **Deployment** is a declarative resource in Kubernetes that:

- Defines the desired state of application replicas
- Manages Replicasets automatically
- Enables rolling updates and rollbacks
- Provides self-healing capabilities

Key components:

- **Replicas:** Number of pod instances to maintain
- **Selector:** Labels used to identify managed pods
- **Pod Template:** Specification for creating pods
- **Strategy:** Defines how updates are rolled out

### 3. Kubernetes Service and Network Exposure

A **Service** is an abstraction layer that exposes application pods to network traffic. Three primary types exist:

#### NodePort Service (Used in this deployment)

- Allocates a port (30000-32767) on each node
- Routes external traffic to the service and subsequently to pods
- Simple but suitable for development/testing environments
- No cloud-provider-specific infrastructure required

#### Service Components:

- **Port:** Exposed port within the cluster
- **TargetPort:** Port on which the container listens
- **NodePort:** External port on all nodes
- **Selector:** Pod labels matching the service

### 4. Container Image and Jenkins Configuration

**Jenkins Container Image (`jenkins/jenkins:latest`):**

- Official Jenkins Docker image with all necessary dependencies
- Pre-configured with Java runtime environment
- Uses Jetty web server (version 12.1.8)

**Environment Variables in Kubernetes:**

- Passed to containers at runtime via the `env` field
- Allow runtime configuration without modifying container images
- `JAVA_OPTS=-Djenkins.install.runSetupWizard=false` disables the initial setup wizard

### 5. Pod Lifecycle and Readiness

**Pod Lifecycle States:**

1. Pending - Awaiting resource allocation
2. Running - Container is operational
3. Succeeded - Completed container execution
4. Failed - Execution terminated with error
5. Unknown - State cannot be determined

**Readiness Probes:**

- Determine if a pod is ready to accept traffic
- Used by kubectl wait command to monitor pod status
- Ensures application is fully initialized before accepting requests

### 6. DNS and Internal Service Discovery

Kubernetes provides automated service discovery through DNS:

- **Service DNS Name:** `<service-name>.<namespace>.svc.cluster.local`
- Jenkins service internal DNS: `jenkins-service.jenkins.svc.cluster.local`
- Pods can communicate using these DNS names without knowing pod IPs

### 7. Load Balancing and Traffic Routing

**kube-proxy:** Daemon running on every node that:

- Manages service endpoints
- Implements load balancing algorithms
- Routes traffic from NodePort to pod internal ports
- Maintains connection state for UDP/TCP connections

---

## Mistakes and Fixes

### Outcome: SUCCESSFUL DEPLOYMENT

After careful execution of all steps and verification procedures, the Jenkins deployment was **successful on the first attempt**. The following validation confirms correct implementation:

#### Verification Checkpoints Met:

1. **Namespace Created Successfully**
   - Command executed without errors
   - Verified via `kubectl get namespace jenkins`

2. **Service Configured Correctly**
   - Service type: NodePort ✓
   - Service port: 8080 ✓
   - NodePort: 30008 ✓
   - Selector matching: app=jenkins ✓
   - Service CLUSTER-IP: 10.43.116.4 ✓

3. **Deployment Configured Correctly**
   - Deployment name: jenkins-deployment ✓
   - Container image: jenkins/jenkins:latest ✓
   - Container port: 8080 ✓
   - Replicas: 1/1 Running ✓
   - Labels: app=jenkins ✓
   - Environment variable set: JAVA_OPTS correctly configured ✓

4. **Pod Readiness**
   - Pod achieved Running state immediately (17 seconds after deployment)
   - Readiness probe confirmed pod condition met within timeout
   - No restart events observed (RESTARTS: 0)

5. **Service Accessibility**
   - HTTP/1.1 200 OK response received from external access
   - Jenkins headers present in HTTP response (X-Jenkins: 2.566)
   - Jetty web server responding correctly (Server: Jetty(12.1.8))
   - Service accessible via NodePort 30008 ✓

#### Potential Improvements (Not Issues):

While the deployment was successful, the following enhancements could be considered for production environments:

| Area                | Consideration                    | Recommendation                                                     |
| ------------------- | -------------------------------- | ------------------------------------------------------------------ |
| **Storage**         | No persistent storage configured | Add PersistentVolume for Jenkins configuration and build artifacts |
| **Resource Limits** | No CPU/memory limits set         | Define resource requests and limits to prevent resource starvation |
| **Security**        | No authentication enforced yet   | Configure Jenkins security and RBAC policies                       |
| **Access Method**   | NodePort used for simplicity     | Consider using Ingress for production multi-service deployments    |
| **Monitoring**      | No health checks configured      | Implement liveness and readiness probes for self-healing           |
| **Backup**          | No backup strategy               | Plan backup strategy for Jenkins configurations and jobs           |

---

## Summary

The Jenkins CI/CD server deployment on the Kubernetes cluster was completed successfully. All containerization principles, Kubernetes orchestration features, and DevOps best practices were properly implemented. The service is now accessible and ready for pipeline creation and management by the Nautilus DevOps team.

**Status:** ✅ DEPLOYMENT SUCCESSFUL

**Next Steps:**

- Access Jenkins UI at http://10.244.195.234:30008
- Configure Jenkins plugins and security
- Create deployment pipelines for projects
- Set up backup and monitoring strategies
