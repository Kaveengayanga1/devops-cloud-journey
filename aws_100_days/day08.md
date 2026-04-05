# Day 08: EC2 Stop Protection

Stop protection prevents an instance from being stopped via Console/CLI/API. Useful to avoid accidental downtime on production instances.

## Enable stop protection
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --disable-api-stop
```

## Disable stop protection (to allow stop)
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --no-disable-api-stop
```

## Verify status
```bash
aws ec2 describe-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --attribute disableApiStop \
  --query "DisableApiStop.Value"
```
- `true`: protection ON
- `false`: protection OFF

## Stop vs termination protection
- Stop protection: `--disable-api-stop` blocks stopping.
- Termination protection: `--disable-api-termination` blocks termination.

## Key limitations
- Does not block OS-initiated shutdown inside the instance.
- Auto Scaling may still stop/terminate per policies and health checks.

## Example session (us-east-1)
```text
~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances --query 'Reservations[*].Instances[*].{Name:Tags[?Key==`Name`].Value | [0], ID:InstanceId, Type:InstanceType, State:State.Name}' --output table
------------------------------------------------------------------
|                        DescribeInstances                       |
+----------------------+------------------+----------+-----------+
|          ID          |      Name        |  State   |   Type    |
+----------------------+------------------+----------+-----------+
|  i-0576b601548312aed |  datacenter-ec2  |  running |  t2.micro |
+----------------------+------------------+----------+-----------+

~ on ☁️  (us-east-1) ➜  aws ec2 modify-instance-attribute --instance-id i-0576b601548312aed --disable-api-stop

~ on ☁️  (us-east-1) ➜  ^C
```
