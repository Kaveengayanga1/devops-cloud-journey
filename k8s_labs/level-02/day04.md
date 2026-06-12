# **Date:** 2026-05-26

# **Topic:** Kubernetes Pod Environment Variables — print-envars-greeting

## Overview

This document covers how to create a Kubernetes Pod that prints a greeting by combining multiple environment variables inside a container. It includes the exact manifest, execution flow, observed output, operational notes, common mistakes, and professional guidance for DevOps and cloud engineers.

## Environment and access details

- Jump host access: `thor@jump-host`
- Kubernetes access: preconfigured `kubectl` on the jump host
- Cluster credentials: not provided in the task prompt and not safely reproducible here
- Additional secret values: no persistent server or secret credentials were supplied in the prompt

Note: I have not invented any missing credentials. If your original lab notes contained ephemeral passwords, SSH keys, kubeconfig details, or token values, they should be copied from your own audit trail. For this document, only the verified access context is recorded.

## Task requirement summary

The goal is to create a Pod named `print-envars-greeting` with one container named `print-env-container`, using the `bash` image, defining three environment variables, and running the exact command shown below. The Pod must use `restartPolicy: Never` so it finishes cleanly after printing the greeting.

## Manifest

Create a file named `print-envars-greeting.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
    - name: print-env-container
      image: bash
      command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
      env:
        - name: GREETING
          value: "Welcome to"
        - name: COMPANY
          value: "Nautilus"
        - name: GROUP
          value: "Industries"
```

## Execution steps

1. Create or edit the manifest on the jump host.
2. Apply the Pod definition with `kubectl apply -f print-envars-greeting.yaml`.
3. Check whether the Pod completed successfully with `kubectl get pods print-envars-greeting`.
4. Review the logs using `kubectl logs -f print-envars-greeting`.

## Expected output

After applying the manifest, the Pod should complete and the logs should print the assembled greeting:

```text
Welcome to Nautilus Industries
```

## Recorded commands

```bash
vi print-envars-greeting.yaml
cat print-envars-greeting.yaml
kubectl apply -f print-envars-greeting.yaml
kubectl get pods print-envars-greeting
kubectl logs -f print-envars-greeting
```

## Why each setting matters

- `env`: supplies runtime values without hard-coding them into the command.
- `command`: runs the shell expression that expands the environment variables.
- `restartPolicy: Never`: ensures the Pod ends after the command completes, which avoids repeated restarts and a CrashLoopBackOff pattern for a one-shot task.
- `bash` image: provides a minimal shell environment suitable for this lab-style task.

## Common mistakes, why they happen, and how to fix them

1. Forgetting `restartPolicy: Never`
   - Why it happens: many engineers default to the standard Pod behavior without considering that the container exits immediately after printing the message.
   - Result: Kubernetes may keep restarting the Pod, which is noisy and unnecessary for a one-time job.
   - Fix: set `restartPolicy: Never` so the Pod is allowed to complete normally.

2. Using the wrong command syntax for variable expansion
   - Why it happens: shell variables and Kubernetes environment variables are easy to confuse.
   - Result: the output may show the literal variable names or an empty string instead of the intended greeting.
   - Fix: use the exact command `['/bin/sh', '-c', 'echo "$(GREETING) $(COMPANY) $(GROUP)"']` so the shell expands the values correctly.

3. Typing an incorrect Pod name during verification
   - Why it happens: the manifest name and the verification command are often entered separately.
   - Result: `kubectl get pods` or `kubectl logs` fails because the resource name does not match.
   - Fix: keep the object name consistent as `print-envars-greeting` in the manifest and in every validation command.

4. Misaligning indentation in YAML
   - Why it happens: YAML is indentation-sensitive and small spacing errors are common when editing manually.
   - Result: `kubectl apply` returns a parsing or validation error.
   - Fix: preserve the exact structure and spacing shown in the manifest and re-run `kubectl apply`.

## Professional notes

- Prefer declarative YAML over ad-hoc imperative commands for repeatable Kubernetes work.
- For one-off tasks like this, treat the Pod as a short-lived workload rather than a long-running service.
- In production, use ConfigMaps, Secrets, or workload parameters instead of embedding sensitive values directly in a manifest.
- Validate the manifest with `kubectl apply --dry-run=client -f print-envars-greeting.yaml` before applying it in a shared environment.

## Conclusion

This task demonstrates how environment variables can be injected into a Pod and expanded by a shell command to produce a controlled output. The final result should be a completed Pod whose logs show `Welcome to Nautilus Industries`.

## Files created

- [day04.md](/mnt/Local%20Disk%20D/DevOps/kodekloud/k8s_labs/day04.md)
