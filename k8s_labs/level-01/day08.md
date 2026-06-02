# Day 08: Kubernetes CronJobs

## Task Overview
The Nautilus DevOps team is setting up recurring tasks on different schedules. The goal was to create a Kubernetes CronJob to execute a dummy command periodically.

### Requirements:
- **CronJob Name**: `devops`
- **Schedule**: `*/7 * * * *` (Every 7 minutes)
- **Container Name**: `cron-devops`
- **Image**: `httpd:latest`
- **Command**: `echo Welcome to xfusioncorp!`
- **Restart Policy**: `OnFailure`

## Server Credentials
- **Server**: Jump-host
- **User**: `thor`

## Theory: What is a CronJob?
A **CronJob** in Kubernetes creates **Jobs** on a repeating schedule. It is similar to the traditional crontab in Linux. It is useful for periodic and recurring tasks like backups, report generation, and automated scripts.

### Cron Schedule Format
The schedule is defined using five fields:
`* * * * *`
(Minute) (Hour) (Day of month) (Month) (Day of week)

- `*/7 * * * *` means "every 7 minutes".

## Implementation Steps

### 1. Create the Manifest File
Create a file named `devops-cronjob.yaml`:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: devops
spec:
  schedule: "*/7 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cron-devops
            image: httpd:latest
            command:
            - /bin/sh
            - -c
            - echo Welcome to xfusioncorp!
          restartPolicy: OnFailure
```

### 2. Apply the Manifest
```bash
thor@jump-host ~$ kubectl apply -f devops-cronjob.yaml
cronjob.batch/devops created
```

### 3. Verification
Check the CronJob status:
```bash
thor@jump-host ~$ kubectl get cronjob devops
```

Wait for the schedule to trigger (up to 7 minutes) and check for Jobs/Pods:
```bash
thor@jump-host ~$ kubectl get jobs
thor@jump-host ~$ kubectl get pods
```

Check the logs of the completed pod:
```bash
thor@jump-host ~$ kubectl logs <pod-name>
Welcome to xfusioncorp!
```

## Troubleshooting & Mistakes Made

### 1. Waiting for the First Execution
**Mistake**: Immediately running `kubectl get pods` or `kubectl get jobs` and seeing "No resources found".
**Reason**: Unlike a standard Deployment or Pod, a CronJob does not start immediately unless the schedule matches the current time. It waits for the defined interval (in this case, every 7 minutes).
**Fix**: Be patient and monitor the `LAST SCHEDULE` column in `kubectl get cronjob`. Once a job is triggered, the pods will appear.

### 2. Resource Cleanup
**Mistake**: Not realizing that Jobs and Pods created by CronJobs stay in the system in a "Completed" state.
**Reason**: Kubernetes keeps them for log inspection.
**Fix**: If you need to limit history, use `successfulJobsHistoryLimit` and `failedJobsHistoryLimit` in the spec.
