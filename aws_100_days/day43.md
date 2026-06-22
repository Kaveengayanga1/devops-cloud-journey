Day 43: EKS cluster creation — datacenter-eks
Date: 2026-06-21
Project: Nautilus DevOps
Topic: Create Amazon EKS cluster with Custom configuration, Auto Mode disabled, private endpoint, using Default VPC (AZs: a, b, c)
Kubernetes version: latest stable (selected at creation time)

## Overview

This document records the steps, outputs, errors, root-cause analysis, and fixes performed while creating an Amazon EKS cluster named `datacenter-eks` with the following constraints:

- Use IAM role `eksClusterRole` as the cluster service role.
- Use Custom configuration with EKS Auto Mode disabled (Standard mode).
- Configure cluster endpoint access as Private only.
- Use the account's Default VPC and three subnets that correspond to availability zones a, b and c.

All credentials shown below are historical/example values and are no longer active.

---

## Server Credentials (Historical / Example)

| Item                      |                    Example / Historical value | Notes                                           |
| ------------------------- | --------------------------------------------: | ----------------------------------------------- |
| Cluster name              |                                datacenter-eks | Created via AWS Console (Custom config)         |
| Cluster role (IAM)        | arn:aws:iam::704152243091:role/eksClusterRole | Role used as cluster service role               |
| Example server (SSH) host |                                    192.0.2.10 | Example only — not active                       |
| Example server username   |                                      ec2-user | Example only                                    |
| Example server password   |                  P@ssw0rd! (no longer active) | Example only — never store real secrets in docs |
| Example SSH port          |                                            22 | Example only                                    |

## Other credentials (Historical / Example)

- Example API key: AKIAIOSFODNN7EXAMPLE (example only)
- Example DB password: db-P@ssw0rd! (example only)

> Note: Remove or rotate any real credentials before storing documentation in a shared repository.

---

## Creation Summary (what was performed)

1. In AWS Console -> Amazon EKS -> Create cluster.
2. Selected `Custom configuration` (to disable Auto Mode / Quick Create flow).
3. Set Name: `datacenter-eks`.
4. Selected cluster service role: `eksClusterRole` (existing IAM role).
5. Ensured EKS Auto Mode / Quick Create was disabled (selected Standard / Custom options).
6. Networking: selected Default VPC and the three subnets in availability zones a, b, c.
7. Set Cluster endpoint access to Private only.
8. Reviewed settings and clicked Create.

Cluster status at creation: `Creating` (console shows progress; allow ~10–15 minutes for Active).

---

## Task outputs (console messages, errors observed)

During attempts to create the cluster or related Node Group actions the following outputs / errors were recorded.

1. iam:PassRole error (observed when console attempted to use an auto-managed node role):

```
User: arn:aws:iam::704152243091:user/kk_labs_user_551145 is not authorized to perform: iam:PassRole on resource: arn:aws:iam::704152243091:role/AmazonEKSAutoNodeRole because no identity-based policy allows the iam:PassRole action
```

2. Subsequent variant when attaching a custom node role (example `eks-node-role`):

```
User: arn:aws:iam::704152243091:user/kk_labs_user_551145 is not authorized to perform: iam:PassRole on resource: arn:aws:iam::704152243091:role/eks-node-role because no identity-based policy allows the iam:PassRole action
```

3. Fargate profiles list error (observed when console tried to enumerate Fargate profiles):

```
Error loading Fargate profiles
User: arn:aws:iam::704152243091:user/kk_labs_user_551145 is not authorized to perform: eks:ListFargateProfiles on resource: arn:aws:eks:us-east-1:704152243091:cluster/datacenter-eks with an explicit deny in a service control policy: arn:aws:organizations::487349550619:policy/o-0mgf8ua334/service_control_policy/p-icntgjha
```

4. Console status messages during normal progress:

```
Cluster datacenter-eks status: Creating
...
Cluster datacenter-eks status: Active
```

(Allow several minutes for transition; verify `Active` before proceeding to add Node Groups or workloads.)

---

## Theories and technical background

1. EKS Auto Mode vs Custom (Standard)

- EKS Auto Mode (Quick Create) attempts to simplify cluster provisioning by selecting sensible defaults and creating an auto-managed node role (e.g., `AmazonEKSAutoNodeRole`) or auto-provisioned resources. In locked lab environments, passing an auto-managed role may be blocked because the user cannot perform `iam:PassRole` for that role.

2. IAM `iam:PassRole` action

- When AWS services create or assign roles on behalf of a user (for example, assigning a node role to a Node Group), the user who requests the operation must have the `iam:PassRole` permission on the role resource. Without it, the console/API operation fails with the exact `iam:PassRole` error recorded above.

3. Service Control Policies (SCPs)

- SCPs applied to an AWS Organization can explicitly Deny actions (for example `eks:ListFargateProfiles`) for member accounts. An explicit deny via an SCP cannot be overridden by identity policies in the account. That is why the Fargate listing failed with an SCP-deny message.

4. Private cluster endpoints

- When a cluster's endpoint access is configured as Private, the Kubernetes API server endpoint is accessible only from within the VPC (or via VPC peering/Transit Gateway/privatelink patterns). Public access is disabled to reduce external exposure.

5. Default VPC and Availability Zones

- Using the Default VPC simplifies networking for labs. Ensure three subnets mapped to zones a, b, c exist and are selected so the control plane and future node groups have multi-AZ redundancy.

---

## Mistakes, root causes, and fixes

Mistake 1: Attempting Quick Create / Auto Mode which required an auto-managed node role.

- Why it happened: Console default or Quick Create option attempts to create/attach `AmazonEKSAutoNodeRole` or similar; the lab account restricts `iam:PassRole` for that role.
- Fix (how it was resolved): Switch to `Custom configuration` / Standard flow in the EKS Create Cluster UI and explicitly disable Auto Mode; select the pre-existing cluster service role `eksClusterRole`. This prevents the console from trying to auto-assign `AmazonEKSAutoNodeRole`.

Mistake 2: Creating a custom node role but selecting incorrect Trusted Entity or expecting it to be passable.

- Why it happened: When a user-created role was made with a non-EC2 trusted entity (or when organization SCPs restrict pass actions), the console either didn't show the role in the Node role dropdown or the account lacked `iam:PassRole` privilege for that role.
- Fix: Ensure any custom node role is created with Trusted entity = `EC2` (use case EC2) and has the required managed policies attached: `AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`. Prefer using the lab-provided role name if the lab requires an exact name. If `iam:PassRole` is denied by policy, request administrator to grant `iam:PassRole` for the exact role ARN or use the lab-provided role.

Mistake 3: Misinterpreting the Fargate profiles error as an EKS creation failure.

- Why it happened: Console automatically queries Fargate profiles for the cluster; the organization SCP explicitly denies `eks:ListFargateProfiles` causing a red error message in the UI.
- Fix: This is not fatal. Confirm cluster `Status: Active` and ignore the Fargate profiles permission error if Fargate is not required. If Fargate is required, contact account administrators to remove the SCP deny or to grant the specific permission.

---

## Step-by-step resolution summary (actions taken)

1. Re-opened EKS Create Cluster flow and selected `Custom configuration`.
2. Entered cluster name `datacenter-eks` and selected `eksClusterRole` for Cluster service role.
3. Ensured Auto Mode / Quick Create was disabled (selected Standard/Custom options).
4. Selected Default VPC and three subnets belonging to AZs a, b, c.
5. Set Cluster endpoint access to `Private` only.
6. Clicked Create and monitored console until cluster transitioned from `Creating` to `Active`.
7. If `iam:PassRole` errors occur for node role assignment, either:
   - Use the lab-provided node role name (recommended in locked lab environments), or
   - Request an admin to attach an identity policy granting `iam:PassRole` for the specific node role ARN.

---

## Recommended next steps (to make cluster ready for workloads)

1. Verify cluster Status is `Active` in Console or via `aws eks describe-cluster`.
2. Create or attach Node Group(s):
   - Prefer using the lab-provided node role name (if specified in lab instructions) to avoid `iam:PassRole` denies.
   - If creating a role, set Trusted entity = `EC2` and attach: `AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`.
3. Confirm `kubectl` connectivity via a bastion or from within the VPC (private endpoint requires VPC-internal access). Example `kubectl` check (from VPC):

```bash
# after configuring kubeconfig for datacenter-eks (from VPC)
kubectl get nodes
kubectl get pods -A
```

4. If Fargate is required, request admin to adjust SCP that denies `eks:ListFargateProfiles`.

---

## References

- AWS EKS docs: Cluster endpoint access, node IAM policies, and `iam:PassRole` requirements.
- Organization admin / lab instructions (follow lab exact role name if provided).

---

Document prepared by: Nautilus DevOps (lab session notes) — historical/example credentials included only for completeness and learning.
