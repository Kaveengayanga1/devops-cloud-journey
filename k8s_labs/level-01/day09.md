# Day 09: Kubernetes Job Template

## Task Overview
The Nautilus DevOps team is building a Kubernetes Job template for a one-time dummy workload. The objective was to create a Job that runs a short command, completes successfully, and follows the exact naming and container requirements.

### Requirements
- **Job name**: `countdown-xfusion`
- **Job template name**: `countdown-xfusion`
- **Container name**: `container-countdown-xfusion`
- **Image**: `debian:latest`
- **Restart policy**: `Never`
- **Command**: `sleep 5`

## Server Credentials
- **Server**: Jump-host
- **User**: `thor`
- **Additional credentials**: None were provided or required for this lab task.

## Theory: What is a Kubernetes Job?
A **Kubernetes Job** is used for workloads that should run to completion. Unlike a Deployment, which keeps replicas running continuously, a Job starts Pods, waits for them to finish, and then marks the task as completed.

### Why Jobs are useful
- Batch processing
- Backup tasks
- Script execution
- Data migration or cleanup tasks

### Key fields in this task
- `apiVersion: batch/v1` defines the stable Job API.
- `kind: Job` tells Kubernetes that this manifest creates a Job.
- `metadata.name` gives the Job its cluster-level name.
- `spec.template.metadata.name` names the Pod template as requested.
- `spec.template.spec.containers` defines the container that will run the command.
- `restartPolicy: Never` ensures the Pod is not restarted after the command finishes.

### Why `sleep 5` matters
The command `sleep 5` creates a short-lived workload. This is useful for testing Job behavior because the Pod will stay active briefly and then finish cleanly with no output.

## Implementation Steps

### 1. Create the manifest file
Create a file named `job.yaml` on the jump-host:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-xfusion
spec:
  template:
    metadata:
      name: countdown-xfusion
    spec:
      containers:
      - name: container-countdown-xfusion
        image: debian:latest
        command: ["sleep", "5"]
      restartPolicy: Never
```

### 2. Apply the manifest
```bash
thor@jump-host ~$ kubectl apply -f job.yaml
job.batch/countdown-xfusion created
```

### 3. Verify the Job and Pod
```bash
thor@jump-host ~$ kubectl get jobs
NAME                STATUS    COMPLETIONS   DURATION   AGE
countdown-xfusion   Running   0/1           9s         9s
```

```bash
thor@jump-host ~$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
countdown-xfusion-6mzww   0/1     Completed   0          21s
```

### 4. Check logs
```bash
thor@jump-host ~$ kubectl logs countdown-xfusion-6mzww
thor@jump-host ~$ 
```

## Terminal Output Reference
The final observed flow was:

```bash
thor@jump-host ~$ vi job.yaml
thor@jump-host ~$ kubectl apply -f job.yaml 
job.batch/countdown-xfusion created
thor@jump-host ~$ kubectl get jobs
NAME                STATUS    COMPLETIONS   DURATION   AGE
countdown-xfusion   Running   0/1           9s         9s
thor@jump-host ~$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
countdown-xfusion-6mzww   0/1     Completed   0          21s
thor@jump-host ~$ kubectl logs countdown-xfusion-6mzww
thor@jump-host ~$ 
```

## Mistakes Made, Why They Happened, and How They Were Fixed

### 1. Expecting log output from `sleep 5`
**Mistake**: Running `kubectl logs` and expecting visible output.
**Why it happened**: The command only sleeps for 5 seconds and does not print anything.
**Fix**: Confirmed that an empty log result is expected and validated the workload through Pod completion status instead.

### 2. Using the wrong workload mindset
**Mistake**: Treating the Job like a long-running service.
**Why it happened**: Jobs and Deployments both use Pods, so it is easy to assume they behave the same way.
**Fix**: Focused on the Job lifecycle: create Pod, run once, complete, and stop.

### 3. Misnaming the template or container
**Mistake**: Using a generic Pod name or container name instead of the exact required values.
**Why it happened**: Small naming differences are easy to overlook in YAML.
**Fix**: Set `spec.template.metadata.name` to `countdown-xfusion` and the container name to `container-countdown-xfusion` exactly as required.

## Professional Steps for a DevOps/Cloud Engineer
1. Read the requirement carefully and identify whether the workload is long-running or run-to-completion.
2. Choose the correct Kubernetes resource type, which is `Job` for this task.
3. Build the YAML manifest with exact metadata, container name, image, command, and restart policy.
4. Apply the manifest with `kubectl apply -f job.yaml`.
5. Verify the Job, then inspect the Pod status to confirm completion.
6. Use logs only to confirm runtime behavior, not to expect output from a silent command like `sleep`.
7. Recheck field names and indentation if the manifest is rejected or the Pod does not behave as expected.

## Additional Theory to Know
### Job vs Deployment
- A **Job** is for finite tasks.
- A **Deployment** is for continuously running applications.

### restartPolicy
- `Never` is commonly used for Jobs that should not restart the same Pod.
- `OnFailure` can be used in some batch scenarios, but this task explicitly required `Never`.

### image: latest tag
- Using `debian:latest` satisfies the requirement.
- In production, pinned versions are usually safer than `latest` because they are more predictable.

### Pod template metadata
- `spec.template.metadata.name` is part of the pod template, not the top-level Job name.
- Exact naming matters in lab tasks because validation often checks the manifest fields directly.
