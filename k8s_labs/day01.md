# Day 01: Creating a Kubernetes Pod

## Task

The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:

- Create a pod named `pod-httpd` using the `httpd` image with the latest tag. Ensure to specify the tag as `httpd:latest`.
- Set the app label to `httpd_app`.
- Name the container as `httpd-container`.

> **Note:** The `kubectl` utility on the jump-host has been configured to work with the Kubernetes cluster.

---

## Guide

Getting comfortable with Pods is a foundational step in mastering Kubernetes. Let's break down the theory and then look at the professional way to deploy this.

### The Theory: Pods and Labels

In Kubernetes, a **Pod** is the smallest and simplest unit in the object model that you create or deploy. It represents a single instance of a running process in your cluster. While a Pod can hold multiple containers that share resources, such as network and storage, the most common use case is the *one-container-per-Pod* model. In this scenario, the Pod acts as a wrapper around your single `httpd` container.

**Labels** are key/value pairs attached to Kubernetes objects. They are used to specify identifying attributes of objects that are meaningful and relevant to users. By tagging your Pod with `app: httpd_app`, you make it easy to manage, filter, and connect this Pod to other Kubernetes resources like Services or Deployments later on.

### The Professional Approach: Declarative Configuration

While you could use a single command-line instruction to create this Pod, professional DevOps engineers rely on declarative configuration using YAML files. This practice aligns with **Infrastructure as Code** principles, allowing your cluster configurations to be reviewed, version-controlled, and easily reproduced.

#### Step 1: Create the YAML Definition

Create a file named `httpd-pod.yaml` on your jump-host using your preferred text editor, such as `vim` or `nano`.

Paste the following configuration into the file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
  - name: httpd-container
    image: httpd:latest
```

Here is how the configuration breaks down:

- `apiVersion` and `kind` tell Kubernetes what type of resource you are declaring.
- `metadata` holds the specific name of the Pod and the labels you need to apply.
- `spec` describes the desired state for the Pod, defining the container name and the exact image to pull.

#### Step 2: Apply the Configuration

Once the file is saved, use the `kubectl` utility to apply this configuration to your cluster.

```bash
kubectl apply -f httpd-pod.yaml
```

The `apply` command is standard practice because it creates the resource if it doesn't exist, and gracefully updates it if you make changes to the YAML file later.

#### Step 3: Verify the Deployment

It is standard procedure to verify that your resource was created successfully and is running without issues.

To check the current status of your Pod:

```bash
kubectl get pods
```

To see detailed information about the Pod, including verifying its labels and container image:

```bash
kubectl describe pod pod-httpd
```
