# Day 07: Listing EC2 Instance Types

Below are useful AWS CLI and in-instance methods to find EC2 instance types.

## 1) Basic list: ID and type
```bash
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType}' \
  --output table
```

## 2) Comprehensive list: Name, ID, type, state
```bash
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].{Name:Tags[?Key==`Name`].Value | [0], ID:InstanceId, Type:InstanceType, State:State.Name}' \
  --output table
```

## 3) Filter by type
- Specific type (e.g., t3.micro):
```bash
aws ec2 describe-instances \
  --filters "Name=instance-type,Values=t3.micro" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text
```
- Wildcard family (e.g., all t3):
```bash
aws ec2 describe-instances \
  --filters "Name=instance-type,Values=t3.*" \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType]' \
  --output table
```

## 4) From within an EC2 instance (IMDSv2)
```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-type
```

Notes:
- IMDS calls require network access to 169.254.169.254 from the instance.
- Ensure your AWS CLI is configured with credentials/region before running the CLI commands.

## 5) Change an EC2 instance type (stop → modify → start)
Prereqs: instance must be EBS-backed; AMI architecture must support the target type (e.g., x86_64 vs ARM).

1) Stop the instance (wait until stopped):
```bash
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 wait instance-stopped --instance-ids i-1234567890abcdef0
```

2) Modify the instance type:
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --instance-type "{\"Value\": \"t3.medium\"}"
```

3) Start the instance:
```bash
aws ec2 start-instances --instance-ids i-1234567890abcdef0
```

Important considerations:
- Public IPv4 changes on stop/start unless using an Elastic IP.
- Nitro families (t3/m5/c5/…) need NVMe and ENA drivers installed in the OS.
- Expect downtime during the stop/modify/start sequence.
