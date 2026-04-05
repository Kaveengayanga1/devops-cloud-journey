# Day 3 — Create a Security Group (Default VPC/Subnet)

**Objective:** Create and configure a security group in the default VPC/subnet and show commands you can copy-paste.

**Prerequisites**
- AWS CLI configured for the target region
- IAM permissions: `ec2:Describe*`, `ec2:CreateSecurityGroup`, `ec2:AuthorizeSecurityGroupIngress`, `ec2:CreateSubnet` (if needed)

## Quick Steps

1. Find the default VPC ID
2. Inspect subnets in the VPC
3. (Optional) Create a subnet
4. Create a security group
5. Add ingress rules (SSH/HTTP example)

## Commands (copy & paste — replace placeholders)

Find the default VPC ID:

```bash
aws ec2 describe-vpcs \
    --filters "Name=isDefault,Values=true" \
    --query "Vpcs[0].VpcId" --output text
```

List subnets in that VPC (replace <vpc-id>):

```bash
aws ec2 describe-subnets \
    --filters "Name=vpc-id,Values=<vpc-id>" \
    --query "Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]" --output table
```

Create a subnet (optional):

```bash
aws ec2 create-subnet \
    --vpc-id <vpc-id> \
    --cidr-block 172.31.96.0/24 \
    --availability-zone us-east-1a
```

Create a tagged subnet (example)

```bash
aws ec2 create-subnet \
    --vpc-id vpc-0ef52f7a59409247d \
    --cidr-block 172.31.100.0/24 \
    --availability-zone us-east-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=devops-subnet}]'
```

What this does

- Creates a new subnet in the specified VPC and AZ with CIDR `172.31.100.0/24`.
- Tags the subnet with `Name=devops-subnet` so it is easy to find in the console and via CLI filters.

Verify the subnet was created and tagged (replace `<vpc-id>` or use the example VPC ID):

```bash
aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=<vpc-id>" "Name=tag:Name,Values=devops-subnet" \
  --query "Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,Tags]" --output table
```

Or describe by tag directly:

```bash
aws ec2 describe-subnets --filters "Name=tag:Name,Values=devops-subnet" --query "Subnets[*].[SubnetId,VpcId,CidrBlock,AvailabilityZone]" --output table
```

Notes on quoting (Windows `cmd.exe` vs Bash):
- In Bash (Linux/macOS) the single-quoted `--tag-specifications '...` works as shown.
- In Windows `cmd.exe` you may need to use double quotes and escape inner quotes or use PowerShell; for example (PowerShell):

```powershell
aws ec2 create-subnet --vpc-id vpc-0ef52f7a59409247d --cidr-block 172.31.100.0/24 --availability-zone us-east-1a --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=devops-subnet}]"
```

Create a security group (replace `<vpc-id>` and `<group-name>`):

```bash
aws ec2 create-security-group \
    --group-name <group-name> \
    --description "Day 3 security group" \
    --vpc-id <vpc-id>
```

Authorize ingress (SSH + HTTP examples — replace `<group-id>`):

```bash
# SSH from specific IP
aws ec2 authorize-security-group-ingress \
    --group-id <group-id> \
    --protocol tcp --port 22 --cidr 203.0.113.0/32

# HTTP from anywhere
aws ec2 authorize-security-group-ingress \
    --group-id <group-id> \
    --protocol tcp --port 80 --cidr 0.0.0.0/0
```

Launch an instance into the subnet and attach the security group (example):

```bash
aws ec2 run-instances \
    --image-id ami-0123456789abcdef0 \
    --count 1 \
    --instance-type t2.micro \
    --subnet-id <subnet-id> \
    --security-group-ids <group-id> \
    --key-name <your-key>
```

## Notes
- Always replace `<vpc-id>`, `<subnet-id>`, `<group-id>`, `<group-name>` and IP placeholders with your values.
- Be careful opening ports to `0.0.0.0/0` — limit to required sources when possible.

## Previewing this file in VS Code
- Open the file [day3.md](day3.md).
- Quick preview: Press `Ctrl+Shift+V`.
- Preview to the side: `Ctrl+K` then `V` (press `Ctrl+K`, release, then `V`).

If you'd like, I can also add a short table of example commands with outputs or a checklist you can mark as you complete each step.