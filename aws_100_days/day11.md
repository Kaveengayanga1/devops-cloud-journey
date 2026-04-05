# Day 11: Attach Network Interfaces to EC2 Instances

Link secondary network interfaces (ENIs) to running or stopped EC2 instances using the AWS CLI.

## 1. List Network Interfaces (ENIs)
```bash
aws ec2 describe-network-interfaces \
  --query 'NetworkInterfaces[*].{ID:NetworkInterfaceId, Status:Status, PrivateIP:PrivateIpAddress, Name:Tags[?Key==`Name`].Value | [0]}' \
  --output table
```
**Look for:** Interfaces with status `available` (not `in-use`).

## 2. List Instances
```bash
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].{ID:InstanceId, State:State.Name, Name:Tags[?Key==`Name`].Value | [0]}' \
  --output table
```
**Note:** Instance must be in `running` or `stopped` state.

## 3. Attach the Interface
```bash
aws ec2 attach-network-interface \
  --network-interface-id eni-1234567890abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device-index 1
```
**Device Index rules:**
- Index 0: Primary interface (eth0) — already taken
- Index 1: Second interface (eth1)
- Index 2+: Additional interfaces

## 4. Verify Attachment
```bash
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[*].Instances[*].NetworkInterfaces[*].{ID:NetworkInterfaceId, DeviceIndex:Attachment.DeviceIndex, Status:Status}' \
  --output table
```

## Example session (us-east-1)
```text
~ on ☁️  (us-east-1) ➜  aws ec2 describe-network-interfaces \
    --query 'NetworkInterfaces[*].{ID:NetworkInterfaceId, Status:Status, PrivateIP:PrivateIpAddress, Name:Tags[?Key==`Name`].Value | [0]}' \
    --output table
-----------------------------------------------------------------
|                   DescribeNetworkInterfaces                   |
+------------------------+-------+-----------------+------------+
|           ID           | Name  |    PrivateIP    |  Status    |
+------------------------+-------+-----------------+------------+
|  eni-0ecdc285bed3493f8 |  None |  172.31.82.236  |  available |
|  eni-0484f4bc1a46ffc36 |  None |  172.31.93.210  |  in-use    |
+------------------------+-------+-----------------+------------+

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --query 'Reservations[*].Instances[*].{ID:InstanceId, State:State.Name, Name:Tags[?Key==`Name`].Value | [0]}' \
    --output table
----------------------------------------------------
|                 DescribeInstances                |
+----------------------+----------------+----------+
|          ID          |     Name       |  State   |
+----------------------+----------------+----------+
|  i-038e97abfcd6b19f9 |  nautilus-ec2  |  running |
+----------------------+----------------+----------+

~ on ☁️  (us-east-1) ➜  aws ec2 describe-network-interfaces \
    --query 'NetworkInterfaces[*].{ID:NetworkInterfaceId, Name:Tags[?Key==`Name`].Value | [0], Status:Status, PrivateIP:PrivateIpAddress}' \
    --output table
-----------------------------------------------------------------
|                   DescribeNetworkInterfaces                   |
+------------------------+-------+-----------------+------------+
|           ID           | Name  |    PrivateIP    |  Status    |
+------------------------+-------+-----------------+------------+
|  eni-0ecdc285bed3493f8 |  None |  172.31.82.236  |  available |
|  eni-0484f4bc1a46ffc36 |  None |  172.31.93.210  |  in-use    |
+------------------------+-------+-----------------+------------+

~ on ☁️  (us-east-1) ➜  aws ec2 attach-network-interface \
    --network-interface-id eni-0ecdc285bed3493f8 \
    --instance-id i-038e97abfcd6b19f9 \
    --device-index 1
{
    "AttachmentId": "eni-attach-06ed3a65ebd406bb5",
    "NetworkCardIndex": 0
}

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --instance-ids i-038e97abfcd6b19f9 \
    --query 'Reservations[*].Instances[*].NetworkInterfaces[*].{ID:NetworkInterfaceId, DeviceIndex:Attachment.DeviceIndex, Status:Status}' \
    --output table
----------------------------------------------------
|                 DescribeInstances                |
+--------------+-------------------------+---------+
|  DeviceIndex |           ID            | Status  |
+--------------+-------------------------+---------+
|  0           |  eni-0484f4bc1a46ffc36  |  in-use |
|  1           |  eni-0ecdc285bed3493f8  |  in-use |
+--------------+-------------------------+---------+

~ on ☁️  (us-east-1) ➜  
```
