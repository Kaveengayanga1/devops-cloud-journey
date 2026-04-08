# Day 25: EC2 Monitoring with CloudWatch Alarm and SNS

## Theory

### What is EC2?
Amazon EC2 (Elastic Compute Cloud) provides virtual servers in AWS. In this task, we launched an Ubuntu-based EC2 instance named `devops-ec2` to host the application.

### What is CloudWatch?
Amazon CloudWatch is AWS's monitoring and observability service. It collects metrics such as CPU utilization, memory-related data (with agents), and logs.

### What is a CloudWatch Alarm?
A CloudWatch alarm watches a metric and changes state based on configured thresholds.

For this lab:
- Metric: `CPUUtilization`
- Namespace: `AWS/EC2`
- Statistic: `Average`
- Period: `300` seconds (5 minutes)
- Threshold: `>= 90`
- Evaluation periods: `1`

This means the alarm enters `ALARM` if CPU average is 90% or more for one 5-minute period.

### What is SNS?
Amazon SNS (Simple Notification Service) is used for notifications. CloudWatch alarm actions can publish messages to an SNS topic. Here we used `devops-sns-topic` to send an email alert.

### Why `t3.micro` instead of `t2.micro`?
The selected Ubuntu AMI used UEFI-compatible boot mode. `t2.micro` failed for this AMI in your session, and `t3.micro` (Nitro-based) worked.

---

## Task Objective
1. Launch EC2 instance `devops-ec2` using Ubuntu AMI.
2. Create CloudWatch alarm `devops-alarm` for high CPU.
3. Send alarm notification through existing SNS topic `devops-sns-topic`.
4. Validate alarm by generating CPU load.

---

## Practical Steps (AWS CLI)

### Step 1: Find latest Ubuntu AMI (us-east-1)
```bash
aws ec2 describe-images \
    --owners 099720109477 \
    --filters "Name=name,Values=ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*" \
    --query 'Images | sort_by(@, &CreationDate) | [-1].{Name:Name, ImageId:ImageId, Date:CreationDate}' \
    --output table
```

Output used:
```text
ImageId: ami-0071174ad8cbb9e17
Name: ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-20260218
```

### Step 2: Create key pair
```bash
aws ec2 create-key-pair \
    --key-name devops-key \
    --query 'KeyMaterial' \
    --output text > devops-key.pem
```

### Step 3: Get default VPC and subnet
```bash
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text
aws ec2 describe-subnets --filters "Name=default-for-az,Values=true" --query "Subnets[0].SubnetId" --output text
```

Output used:
```text
vpc-0f69ab9df4e4268b7
subnet-038e044d6fea9183b
```

### Step 4: Create security group and allow SSH
```bash
aws ec2 create-security-group \
    --group-name devops-sg \
    --description "Security group for devops application" \
    --vpc-id vpc-0f69ab9df4e4268b7
```

Output used:
```text
GroupId: sg-0c21389b957f9928a
```

```bash
aws ec2 authorize-security-group-ingress \
    --group-id sg-0c21389b957f9928a \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

### Step 5: Launch EC2 instance `devops-ec2`
```bash
aws ec2 run-instances \
    --image-id ami-0071174ad8cbb9e17 \
    --count 1 \
    --instance-type t3.micro \
    --key-name devops-key \
    --security-group-ids sg-0c21389b957f9928a \
    --subnet-id subnet-038e044d6fea9183b \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=devops-ec2}]'
```

Output used:
```text
InstanceId: i-09c0ddada5c475423
PrivateIpAddress: 172.31.37.221
State: pending -> running
```

### Step 6: Get SNS topic ARN
```bash
aws sns list-topics --query "Topics[?contains(TopicArn, 'devops-sns-topic')].TopicArn" --output text
```

Output used:
```text
arn:aws:sns:us-east-1:419341726142:devops-sns-topic
```

### Step 7: Create CloudWatch alarm `devops-alarm`
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name "devops-alarm" \
    --alarm-description "Alarm when CPU exceeds 90% for 5 minutes" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 90 \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --evaluation-periods 1 \
    --dimensions Name=InstanceId,Value=i-09c0ddada5c475423 \
    --alarm-actions arn:aws:sns:us-east-1:419341726142:devops-sns-topic
```

Note: No output means the alarm command succeeded.

### Step 8: Subscribe email to SNS topic
```bash
aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:419341726142:devops-sns-topic \
    --protocol email \
    --notification-endpoint tharindupasintha@gmail.com
```

Initial output:
```text
"SubscriptionArn": "pending confirmation"
```

After confirming from email:
```bash
aws sns list-subscriptions-by-topic --topic-arn arn:aws:sns:us-east-1:419341726142:devops-sns-topic
```

Output used:
```text
SubscriptionArn: arn:aws:sns:us-east-1:419341726142:devops-sns-topic:e03ee9b7-6666-45e0-ba1f-484d142e344b
Protocol: email
Endpoint: tharindupasintha@gmail.com
```

### Step 9: Get public IP and SSH into instance
```bash
aws ec2 describe-instances \
    --instance-ids i-09c0ddada5c475423 \
    --query 'Reservations[*].Instances[*].PublicIpAddress' \
    --output text
```

Output used:
```text
54.166.173.150
```

SSH command:
```bash
ssh -i devops-key.pem ubuntu@54.166.173.150
```

### Step 10: Fix private key permission issue (important)
You got this error first:
```text
WARNING: UNPROTECTED PRIVATE KEY FILE!
Permissions 0644 for 'devops-key.pem' are too open.
Load key "devops-key.pem": bad permissions
```

Correct fix:
```bash
chmod 400 devops-key.pem
```

Then SSH worked successfully.

### Step 11: Install stress tool and trigger high CPU
Inside EC2:
```bash
sudo apt update && sudo apt install stress -y
nproc
```

Output showed:
```text
2
```

Since `t3.micro` has 2 vCPUs, use:
```bash
stress --cpu 2 --timeout 400s
```

### Step 12: Verify alarm state
From local AWS CLI terminal:
```bash
aws cloudwatch describe-alarms \
    --alarm-names "devops-alarm" \
    --query "MetricAlarms[*].StateValue" \
    --output text
```

Observed transition:
```text
OK ... OK ... ALARM
```

Email notification received:
```text
State Change: OK -> ALARM
Reason: 1 datapoint [99.9966...] was >= threshold (90.0)
```

---

## Errors Faced and Fixes

1. `InvalidKeyPair.NotFound`
- Cause: `MyKeyPair` did not exist.
- Fix: Created `devops-key` and used `--key-name devops-key`.

2. Private key permission denied in SSH
- Cause: PEM file mode was too open (`0644`).
- Fix: `chmod 400 devops-key.pem`.

3. Alarm not triggering with `stress --cpu 1`
- Cause: Instance had 2 vCPUs, so overall CPU was around 50%.
- Fix: Used `stress --cpu 2 --timeout 400s`.

---

## Verification Checklist
- [x] Ubuntu EC2 instance `devops-ec2` launched
- [x] Security group created and SSH allowed
- [x] CloudWatch alarm `devops-alarm` created
- [x] SNS topic ARN attached to alarm action
- [x] Email subscription confirmed
- [x] Alarm transitioned from `OK` to `ALARM`
- [x] Alert email received

---

## Key Takeaways
- Always validate key pair availability before launching EC2.
- For Ubuntu Noble AMIs, instance type compatibility matters.
- CloudWatch alarm actions work only when SNS subscriptions are confirmed.
- CPU stress tests must match vCPU count for reliable alarm triggering.
