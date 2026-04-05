# Day 10: Allocate and Associate Elastic IP

An Elastic IP must be allocated before it can be associated with an EC2 instance. This guide shows the two-step process.

## Check existing Elastic IPs
```bash
aws ec2 describe-addresses --region us-east-1 --output table
```

## Step 1: Allocate (create) a new Elastic IP
```bash
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=nautilus-ec2-eip}]'
```
Note the `AllocationId` from the output (e.g., `eipalloc-xxxxxxxx`).

## Step 2: Associate the Elastic IP with an instance
```bash
aws ec2 associate-address \
  --instance-id i-0e739b60863620430 \
  --allocation-id eipalloc-xxxxxxxx
```

## Verify the association
```bash
aws ec2 describe-addresses \
  --region us-east-1 \
  --query 'Addresses[*].{Name:Tags[?Key==`Name`].Value | [0], IP:PublicIp, Instance:InstanceId}' \
  --output table
```

## Example session (us-east-1)
```text
~ on ☁️  (us-east-1) ➜  aws ec2 describe-addresses --output table
-------------------
|DescribeAddresses|
+-----------------+

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType}' \
  --output table
-------------------------------------
|         DescribeInstances         |
+----------------------+------------+
|          ID          |   Type     |
+----------------------+------------+
|  i-0e739b60863620430 |  t2.micro  |
+----------------------+------------+

~ on ☁️  (us-east-1) ➜  aws ec2 describe-addresses --region us-east-1 --output table
-------------------
|DescribeAddresses|
+-----------------+

~ on ☁️  (us-east-1) ➜  aws ec2 allocate-address \
    --domain vpc \
    --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=nautilus-ec2-eip}]'
{
    "AllocationId": "eipalloc-04e229e0f56965877",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "us-east-1",
    "Domain": "vpc",
    "PublicIp": "3.227.38.111"
}

~ on ☁️  (us-east-1) ➜  aws ec2 describe-addresses --region us-east-1 --output table
--------------------------------------------------------
|                   DescribeAddresses                  |
+------------------------------------------------------+
||                      Addresses                     ||
|+---------------------+------------------------------+|
||  AllocationId       |  eipalloc-04e229e0f56965877  ||
||  Domain             |  vpc                         ||
||  NetworkBorderGroup |  us-east-1                   ||
||  PublicIp           |  3.227.38.111                ||
||  PublicIpv4Pool     |  amazon                      ||
|+---------------------+------------------------------+|
|||                       Tags                       |||
||+--------------+-----------------------------------+||
|||  Key         |  Name                             |||
|||  Value       |  nautilus-ec2-eip                 |||
||+--------------+-----------------------------------+||

~ on ☁️  (us-east-1) ➜  aws ec2 associate-address \
    --instance-id i-0e739b60863620430 \
    --allocation-id eipalloc-04e229e0f56965877
{
    "AssociationId": "eipassoc-0ce602081ea0e1755"
}

~ on ☁️  (us-east-1) ➜  aws ec2 describe-addresses \
    --region us-east-1 \
    --query 'Addresses[*].{Name:Tags[?Key==`Name`].Value | [0], IP:PublicIp, Instance:InstanceId}' \
    --output table
-------------------------------------------------------------
|                     DescribeAddresses                     |
+--------------+-----------------------+--------------------+
|      IP      |       Instance        |       Name         |
+--------------+-----------------------+--------------------+
|  3.227.38.111|  i-0e739b60863620430  |  nautilus-ec2-eip  |
+--------------+-----------------------+--------------------+

~ on ☁️  (us-east-1) ➜  
```
