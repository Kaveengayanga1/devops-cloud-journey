# Kubernetes Day 02: Creating a Deployment

## Theory

Before creating a Kubernetes Deployment, it is important to understand the fundamental concepts involved.

### Key Concepts

*   **Deployment**: This is a Kubernetes "Resource object". Its main function is to maintain the **Desired State** of your application. For example, if a Pod fails, the Deployment controller automatically creates a new one (Self-healing).
*   **Pod**: The smallest deployable unit in Kubernetes. Your `nginx` container runs inside a Pod.
*   **Container Image**: This is the "Blueprint" containing all the files and configurations necessary to run the application. In this task, we use `nginx:latest`, where `latest` is the image tag.
*   **kubectl**: This is the Command Line Interface (CLI) tool used to interact with the Kubernetes cluster.

## Task

The Nautilus DevOps team is delving into Kubernetes for app management. One team member needs to create a deployment following these details:

1.  Create a deployment named `nginx`.
2.  Deploy the application `nginx`.
3.  Use the image `nginx:latest` (ensure to specify the tag).

> **Note**: The `kubectl` utility on the jump-host has been configured to work with the Kubernetes cluster.

## Steps to Perform the Task

1.  **Connect to the Jump-host and Verify Connectivity**:
    Check if `kubectl` is working correctly and lists the nodes.
    ```bash
    kubectl get nodes
    ```

2.  **Create the Deployment**:
    Use the imperative command to create the deployment.
    ```bash
    kubectl create deployment nginx --image=nginx:latest
    ```
    *   `nginx`: The name of the deployment.
    *   `--image=nginx:latest`: Specifies the image and tag clearly.

3.  **Verify the Creation**:
    Check if the deployment was created successfully and if the pods are running.

    Check Deployments:
    ```bash
    kubectl get deploy
    ```

    Check Pods:
    ```bash
    kubectl get pods
    ```

4.  **Detailed Inspection (Optional)**:
    If you need to see more details about the deployment (e.g., events, strategy), use the describe command.
    ```bash
    kubectl describe deployment nginx
    ```

## Output

```bash
thor@jump-host ~$ kubectl get nodes
NAME        STATUS   ROLES           AGE   VERSION
jump-host   Ready    control-plane   44m   v1.34.1+k3s1
thor@jump-host ~$ kubectl get pods
No resources found in default namespace.
thor@jump-host ~$ kubectl create deployment nginx --image=nginx:latest
deployment.apps/nginx created
thor@jump-host ~$ kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           14s
thor@jump-host ~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7c5d8bf9f7-k8ltt   1/1     Running   0          26s
thor@jump-host ~$ docker ps
bash: docker: command not found
thor@jump-host ~$ kubectl describe deployment nginx
Name:                   nginx
Namespace:              default
CreationTimestamp:      Sun, 22 Mar 2026 03:47:47 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:         nginx:latest
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-7c5d8bf9f7 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  45s   deployment-controller  Scaled up replica set nginx-7c5d8bf9f7 from 0 to 1
thor@jump-host ~$ 
```
