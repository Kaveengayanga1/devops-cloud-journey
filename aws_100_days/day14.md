# Day 14 — List EC2 instances, terminate an instance, and verify

This document explains how to list EC2 instances using the AWS CLI, terminate a chosen instance, and verify the instance reaches the `terminated` state.

## 1. List EC2 instances (find your target)

Use `describe-instances` and a `--query` to show the Instance ID, Name tag, and current state in a readable table:

```bash
aws ec2 describe-instances \
    --region us-east-1 \
    --query "Reservations[*].Instances[*].{ID:InstanceId,Name:Tags[?Key=='Name']|[0].Value,State:State.Name}" \
    --output table
```

Example output:

```
~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --region us-east-1 \
    --query "Reservations[*].Instances[*].{ID:InstanceId,Name:Tags[?Key=='Name']|[0].Value,State:State.Name}" \
    --output table
---------------------------------------------------
|                DescribeInstances                |
+----------------------+---------------+----------+
|          ID          |     Name      |  State   |
+----------------------+---------------+----------+
|  i-029f8bce946024946 |  xfusion-ec2  |  running |
+----------------------+---------------+----------+
```

Copy the Instance ID (e.g., `i-029f8bce946024946`) for the termination step.

## 2. Terminate the instance

Use `terminate-instances` with the chosen Instance ID. Termination is permanent.

```bash
aws ec2 terminate-instances \
    --region us-east-1 \
    --instance-ids <YOUR_INSTANCE_ID>
```

Example response for `i-029f8bce946024946`:

```
~ on ☁️  (us-east-1) ➜  aws ec2 terminate-instances \
    --region us-east-1 \
    --instance-ids i-029f8bce946024946
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-029f8bce946024946",
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```

## 3. Verify termination

A. Recommended (scripted): use the `wait` command to pause until the instance is fully `terminated`:

```bash
aws ec2 wait instance-terminated \
    --region us-east-1 \
    --instance-ids <YOUR_INSTANCE_ID>

echo "Instance is now fully terminated."
```

B. Manual check with `describe-instances`:

```bash
aws ec2 describe-instances \
    --region us-east-1 \
    --instance-ids <YOUR_INSTANCE_ID> \
    --query "Reservations[*].Instances[*].State.Name" \
    --output text
```

Example flow showing successful verification:

```
~ on ☁️  (us-east-1) ➜  aws ec2 wait instance-terminated \
    --region us-east-1 \
    --instance-ids i-029f8bce946024946

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --region us-east-1 \
    --instance-ids i-029f8bce946024946 \
    --query "Reservations[*].Instances[*].State.Name" \
    --output text
terminated
```

Notes and important reminders

- Termination is irreversible. Confirm you selected the correct Instance ID before running `terminate-instances`.
- Root EBS volumes are deleted at termination by default if the instance was launched with the typical `DeleteOnTermination` flag; additional attached volumes are not deleted unless configured, and will incur charges until removed.
- Use IAM least privilege: only grant `ec2:DescribeInstances`, `ec2:TerminateInstances`, and `ec2:DescribeInstances` to principals that require them.

Would you like a one-line script that selects an instance by `Name` tag and terminates it automatically (with a confirmation prompt)?