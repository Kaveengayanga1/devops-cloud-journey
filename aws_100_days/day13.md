# Day 13 — Create AMI from existing EC2 instance `datacenter-ec2`

Task: Create an AMI from the existing instance named `datacenter-ec2` with the AMI name `datacenter-ec2-ami`, and ensure the AMI reaches the `available` state.

This document shows how to perform the operation using the AWS CLI and the Management Console, plus explanations and troubleshooting notes.

## Using the AWS CLI

1. Identify the Instance ID (filter by Name tag):

```bash
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=datacenter-ec2" \
    --query "Reservations[*].Instances[*].InstanceId" \
    --output text)
```

2. Create the AMI (by default AWS reboots the instance for consistency):

```bash
AMI_ID=$(aws ec2 create-image \
    --instance-id $INSTANCE_ID \
    --name "datacenter-ec2-ami" \
    --description "AMI for datacenter-ec2" \
    --query "ImageId" \
    --output text)

echo "AMI Creation initiated. Image ID: $AMI_ID"
```

3. Wait until the AMI becomes `available` (recommended for automation):

```bash
aws ec2 wait image-available --image-ids $AMI_ID
echo "AMI $AMI_ID is now in 'available' state."
```

Notes:
- By default, `create-image` reboots the instance to ensure filesystem integrity. Use `--no-reboot` only if you understand the consistency implications.
- The `wait` command polls the API until the image reaches the desired state (or times out), which is useful in CI/CD pipelines.

## Using the AWS Management Console

- Go to EC2 Dashboard → Instances and select `datacenter-ec2`.
- Actions → Image and templates → Create image.
- Enter Image name: `datacenter-ec2-ami` and leave other settings as default.
- Click `Create image` and monitor the Images → AMIs view until Status becomes `available`.

## What creating an AMI means

Creating an AMI is like taking a point-in-time clone of the instance's storage and metadata.

- It snapshots the root EBS volume and any additional volumes included.
- It includes OS, installed applications, configuration files, and any personal files stored on those volumes.
- Files only in memory or in ephemeral non-persistent storage may not be captured unless flushed to disk; the default reboot helps ensure data consistency.

## Troubleshooting and considerations

- Permissions: Ensure your IAM principal has `ec2:CreateImage`, `ec2:DescribeInstances`, and `ec2:DescribeImages` permissions.
- Duplicate names: AMI names must be unique within the account and region; duplicate names may still succeed but can complicate automation that relies on name-based lookups.
- Cost: You pay for the snapshots that back the AMI.

## Example session (sample outputs)

The following is an example sequence demonstrating the commands and outputs.

```bash
~ on ☁️  (us-east-1) ➜  INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=datacenter-ec2" \
    --query "Reservations[*].Instances[*].InstanceId" \
    --output text)

~ on ☁️  (us-east-1) ➜  echo INSTANCE_ID
INSTANCE_ID

~ on ☁️  (us-east-1) ➜  echo $INSTANCE_ID
i-073fbdb5255fce0af

~ on ☁️  (us-east-1) ➜  AMI_ID=$(aws ec2 create-image \
    --instance-id $INSTANCE_ID \
    --name "datacenter-ec2-ami" \
    --description "AMI for datacenter-ec2" \
    --query "ImageId" \
    --output text)

echo "AMI Creation initiated. Image ID: $AMI_ID"
AMI Creation initiated. Image ID: ami-042088ee09465e018

~ on ☁️  (us-east-1) ➜  aws ec2 wait image-available --image-ids $AMI_ID
echo "AMI $AMI_ID is now in 'available' state."
AMI ami-042088ee09465e018 is now in 'available' state.
```

If you want, I can provide:
- A one-line script that wraps the `describe-instances`, `create-image`, and `wait` commands end-to-end, or
- An explanation of how to launch a new EC2 instance from this AMI and verify its volumes.
