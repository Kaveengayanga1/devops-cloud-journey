Day 08: Deploy Tomcat application on Kubernetes
Date: 2026-06-04
Topic: Deploy a Java (Tomcat) application with Deployment + NodePort Service
Project: KodeKloud Kubernetes labs (example)

---

## Header Information

- Date: 2026-06-04
- Topic: Deploy Tomcat app (kodekloud/centos-ssh-enabled:tomcat) to Kubernetes
- Project: KodeKloud lab / training cluster (example)

## Server Credentials (Historical / Example values)

- Host: jump-host (example)
- Host IP (Node internal IP): 10.244.244.139 (example)
- SSH Username: thor (example)
- SSH Password: ExamplePass! (no longer active)
- SSH Port: 22

Other example credentials (historical / no longer active):

- DB user: db_user_example
- DB password: db_pass_example (no longer active)
- API key: APIKEY_EXAMPLE_ABC123 (example)

Note: All credentials above are example/historical values and are not active.

---

## Task Description

Create the following resources in the existing Kubernetes cluster (using `kubectl` from the jump-host):

- Namespace: `tomcat-namespace-devops`
- Deployment: `tomcat-deployment-devops` (replicas: 1)
  - Container name: `tomcat-container-devops`
  - Image: `kodekloud/centos-ssh-enabled:tomcat`
  - Container port: `8080`
- Service: `tomcat-service-devops` (type: `NodePort`, nodePort: `32227`)

You must ensure the application is up and running before verifying with the lab "Check" button.

---

## Manifest used (correct version)

```yaml
# Deployment and Service manifest for Tomcat
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment-devops
  namespace: tomcat-namespace-devops
  labels:
    app: tomcat-app # Custom label for identification
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat-app # Must match the pod template label
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      containers:
        - name: tomcat-container-devops
          image: kodekloud/centos-ssh-enabled:tomcat
          ports:
            - containerPort: 8080 # The port inside the container
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service-devops
  namespace: tomcat-namespace-devops
spec:
  type: NodePort
  selector:
    app: tomcat-app # Routes traffic to pods with this label
  ports:
    - port: 8080 # Port exposed inside the cluster
      targetPort: 8080 # Port the container is listening on
      nodePort: 32227 # Explicitly defined external node port
```

All inline comments inside code blocks are in English (per requirement).

---

## Task Output (shell session logs)

The following is the recorded terminal session output from the jump-host showing the sequence of commands, errors, fixes, and verification (kept verbatim):

```bash
thor@jump-host ~$ kubectl create namespace tomcat-namespace-devops
namespace/tomcat-namespace-devops created
thor@jump-host ~$ vi tomcat-stack.yaml
thor@jump-host ~$ cat tomcat-stack.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment-devops
  namespace: tomcat-namespace-devops
  labels:
    app: tomcat-app # Custom label for identification
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: tomcat-app #must mapp pod template label
      template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      containers:
      - name: tomcat-container-devops
        image: kodekloud/centos-ssh-enabled:tomcat
        ports:
        - containerPort: 8080 # The port inside the container

---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service-devops
  namespace: tomcat-namespace-devops
spec:
  type: NodePort
  selector:
    app: tomcat-app # Routes traffic to pods with this label
  ports:
    - port: 8080       # Port exposed inside the cluster
      targetPort: 8080 # Port the container is listening on
      nodePort: 32227  # Explicitly defined external node port
thor@jump-host ~$ kubectl apply -f tomcat-stack.yaml
service/tomcat-service-devops created
Error from server (BadRequest): error when creating "tomcat-stack.yaml": Deployment in version "v1" cannot be handled as a Deployment: strict decoding error: unknown field "metadata.spec"
thor@jump-host ~$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   26m
thor@jump-host ~$ kubectl delete
error: You must provide one or more resources by argument or filename.
Example resource specifications include:
   '-f rsrc.yaml'
   '--filename=rsrc.json'
   '<resource> <name>'
   '<resource>'
thor@jump-host ~$ kubectl get svc -n tomcat-namespace-devops
NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
tomcat-service-devops   NodePort   10.43.173.150   <none>        8080:32227/TCP   2m16s
thor@jump-host ~$ kubectl delete tomcat-service-devops
error: the server doesn't have a resource type "tomcat-service-devops"
thor@jump-host ~$ kubectl delete svc tomcat-service-devops -n tomcat-namespace-devops
service "tomcat-service-devops" deleted from tomcat-namespace-devops namespace
thor@jump-host ~$
thor@jump-host ~$
thor@jump-host ~$ rm tomcat-stack.yaml
thor@jump-host ~$ vi tomcat-stack.yaml
thor@jump-host ~$ cat tomcat-stack.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment-devops
  namespace: tomcat-namespace-devops
  labels:
    app: tomcat-app # Custom label for identification
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat-app # Must match the pod template label
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      containers:
      - name: tomcat-container-devops
        image: kodekloud/centos-ssh-enabled:tomcat
        ports:
        - containerPort: 8080 # The port inside the container
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service-devops
  namespace: tomcat-namespace-devops
spec:
  type: NodePort
  selector:
    app: tomcat-app # Routes traffic to pods with this label
  ports:
    - port: 8080       # Port exposed inside the cluster
      targetPort: 8080 # Port the container is listening on
      nodePort: 32227  # Explicitly defined external node port
thor@jump-host ~$ kubectl apply -f tomcat-stack.yaml
deployment.apps/tomcat-deployment-devops created
service/tomcat-service-devops created
thor@jump-host ~$ kubectl get all -n tomcat-namespace-devops
NAME                                            READY   STATUS              RESTARTS   AGE
pod/tomcat-deployment-devops-6cd47cb899-dzk78   0/1     ContainerCreating   0          20s

NAME                            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/tomcat-service-devops   NodePort   10.43.28.184   <none>        8080:32227/TCP   20s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat-deployment-devops   0/1     1            0           20s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/tomcat-deployment-devops-6cd47cb899   1         1         0       20s
thor@jump-host ~$ kubectl get pods -n tomcat-namespace-devops
NAME                                        READY   STATUS    RESTARTS   AGE
tomcat-deployment-devops-6cd47cb899-dzk78   1/1     Running   0          38s
thor@jump-host ~$ kubectl get all -n tomcat-namespace-devops
NAME                                            READY   STATUS    RESTARTS   AGE
pod/tomcat-deployment-devops-6cd47cb899-dzk78   1/1     Running   0          42s

NAME                            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/tomcat-service-devops   NodePort   10.43.28.184   <none>        8080:32227/TCP   42s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat-deployment-devops   1/1     1            1           42s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/tomcat-deployment-devops-6cd47cb899   1         1         1       42s
thor@jump-host ~$ kubectl get svc -n tomcat-namespace-devops
NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
tomcat-service-devops   NodePort   10.43.28.184   <none>        8080:32227/TCP   89s
thor@jump-host ~$ kubectl get svc tomcat-service-devops -n tomcat-namespace-devops
NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
tomcat-service-devops   NodePort   10.43.28.184   <none>        8080:32227/TCP   94s
thor@jump-host ~$ kubectl get nodes -o wide
NAME        STATUS   ROLES           AGE   VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
jump-host   Ready    control-plane   31m   v1.34.1+k3s1   10.244.244.139   <none>        Alpine Linux v3.16   6.8.0-90-generic   containerd://1.6.8
thor@jump-host ~$ kubectl get nodes -o wide -n tomcat-namespace-devops
NAME        STATUS   ROLES           AGE   VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
jump-host   Ready    control-plane   31m   v1.34.1+k3s1   10.244.244.139   <none>        Alpine Linux v3.16   6.8.0-90-generic   containerd://1.6.8
thor@jump-host ~$ curl http://10.244.244.139:32227
<!DOCTYPE html>
<!--
To change this license header, choose License Headers in Project Properties.
To change this template file, choose Tools | Templates
and open the template in the editor.
-->
<html>
    <head>
        <title>SampleWebApp</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h2>Welcome to xFusionCorp Industries!</h2>
        <br>

    </body>
</html>
thor@jump-host ~$
```

---

## Theories and Explanations

1. Kubernetes Service Types

- ClusterIP: internal virtual IP used to route traffic inside the cluster. Not directly reachable from outside cluster nodes unless routes or proxying exist.
- NodePort: opens the specified port on every Node; traffic to NodeIP:NodePort is forwarded to the Service and then to pod targetPort.

2. ClusterIP vs Internal (Node) IP

- ClusterIP (example 10.43.28.184) is part of the Service network and is a virtual, cluster-internal address. The node's network stack does not have a separate interface with this IP.
- Internal/Node IP (example 10.244.244.139) is the actual IP address of the Kubernetes node. NodePort exposes a port on this IP so external callers can reach the service.

3. Why the ClusterIP is not directly reachable from outside the cluster

- ClusterIP is a virtual service IP; the host network doesn't route to it directly. Access must be via other mechanisms: `kubectl port-forward`, `kubectl proxy`, NodePort, LoadBalancer, Ingress, or by being inside the cluster network (another pod/node with routing).

---

## Mistakes and Fixes (Root-cause analysis)

Mistake 1: YAML indentation / incorrect placement of `spec` under `metadata` in the initial file

- Symptom (error):

  ```
  Error from server (BadRequest): error when creating "tomcat-stack.yaml": Deployment in version "v1" cannot be handled as a Deployment: strict decoding error: unknown field "metadata.spec"
  ```

- Why it happened: The `spec:` block for the Deployment was accidentally indented under `metadata:` rather than at the Deployment top-level. That made the manifest invalid and Kubernetes couldn't decode it into a Deployment object.

- How it was fixed (steps taken):
  1. Remove the invalid file (`rm tomcat-stack.yaml`).
  2. Recreate the manifest with the correct structure, moving `spec:` out from under `metadata:` so both `metadata:` and `spec:` are top-level fields of the Deployment object.
  3. Re-apply the corrected manifest: `kubectl apply -f tomcat-stack.yaml`.

Result: The Deployment and Service were created successfully and pods reached `Running` state.

Mistake 2: Attempting to delete a resource without specifying the resource type or namespace

- Symptom:

  ```
  thor@jump-host ~$ kubectl delete
  error: You must provide one or more resources by argument or filename.
  ```

- Why it happened: The `kubectl delete` command requires either resource identifier(s) (e.g., `svc mysvc`) or `-f` filename; running it without arguments fails.

- How it was fixed: Use the correct form, e.g. `kubectl delete svc tomcat-service-devops -n tomcat-namespace-devops`.

---

## Verification & Post-check commands

Run these from the jump-host to verify resources:

```bash
# Check all resources in namespace
kubectl get all -n tomcat-namespace-devops

# Check service details
kubectl get svc tomcat-service-devops -n tomcat-namespace-devops -o wide

# Curl the NodePort from a machine that can reach the node IP
curl http://10.244.244.139:32227

# If you cannot reach the node IP from outside, use port-forward from the jump-host:
kubectl port-forward -n tomcat-namespace-devops svc/tomcat-service-devops 8080:8080
# Then access locally: http://localhost:8080
```

---

## Recommendations and best practices

- Keep manifests in Git and use a CI process to lint YAML (e.g., `kubeval`, `kube-linter`) before applying.
- Avoid hard-coding NodePorts in production; prefer LoadBalancer or Ingress for external exposure.
- Use readiness and liveness probes on real workloads to ensure traffic only goes to healthy pods.
- Use RBAC and least-privilege for service accounts and users.

---

If you want, I can:

- Run simple YAML linting on the manifest (if provided here).
- Commit these documents into the repo and/or run additional checks.

Files created: /mnt/Local Disk D/DevOps/kodekloud/k8s_labs/day25.md and /mnt/Local Disk D/DevOps/kodekloud/k8s_labs/day25-sinhala.md
