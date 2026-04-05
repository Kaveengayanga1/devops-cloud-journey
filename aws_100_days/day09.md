# Day 09: EC2 Termination Protection

Termination protection blocks an EC2 instance from being terminated via Console/CLI/API. Use it to prevent accidental deletion.

## Enable termination protection
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --disable-api-termination
```

## Disable termination protection (to allow terminate)
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --no-disable-api-termination
```

## Verify status
```bash
aws ec2 describe-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --attribute disableApiTermination \
  --query "DisableApiTermination.Value"
```
- `true`: protection active (cannot be terminated)
- `false`: protection off (can be terminated)

## Important notes
- Not supported on Spot Instances (Spot may terminate anytime).
- In an Auto Scaling Group, ASG can still terminate on scale-in unless ASG instance protection is also enabled.
- If instance-initiated shutdown behavior is set to Terminate, an in-OS shutdown can delete the instance regardless of this setting.

## Example session (us-east-1)
```text
~ on ☁️  (us-east-1) ➜  aws ec2 describe-instance-status --query 'InstanceStatuses[*].{ID:InstanceId,Health:InstanceStatus.Status,System:SystemStatus.Status}' --output table
---------------------------------------------------------
|                DescribeInstanceStatus                 |
+--------------+-----------------------+----------------+
|    Health    |          ID           |    System      |
+--------------+-----------------------+----------------+
|  initializing|  i-0b7e6ed9dc4be90a8  |  initializing  |
+--------------+-----------------------+----------------+

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType}' --output table
-------------------------------------
|         DescribeInstances         |
+----------------------+------------+
|          ID          |   Type     |
+----------------------+------------+
|  i-0b7e6ed9dc4be90a8 |  t2.micro  |
+----------------------+------------+

~ on ☁️  (us-east-1) ➜  aws ec2 modify-instance-attribute --instance-id i-0b7e6ed9dc4be90a8 --disable-api-termination

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instance-attribute \
    --instance-id i-0b7e6ed9dc4be90a8 \
    --attribute disableApiTermination \
    --query "DisableApiTermination.Value"
true

~ on ☁️  (us-east-1) ➜  
```
