# Day 2 — Security Group in the Default VPC

**Objective:** Create a security group in the default VPC, add common ingress rules, and verify configuration.

**Prerequisites**
- AWS CLI configured for the target region
- IAM permissions: `ec2:CreateSecurityGroup`, `ec2:AuthorizeSecurityGroupIngress`, `ec2:DescribeSecurityGroups`

## Quick Steps

1. Create the security group
2. Add inbound rules (HTTP/SSH examples)
3. Verify the security group

## Commands (copy & paste — replace placeholders)

Create a security group (replace `<group-name>` and `<vpc-id>`):

```bash
aws ec2 create-security-group \
  --group-name <group-name> \
  --description "Security group for <purpose>" \
  --vpc-id <vpc-id>
```

Authorize HTTP (port 80) from anywhere:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <group-id> \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
```

Authorize SSH (port 22) from a single IP (recommended):

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <group-id> \
  --protocol tcp --port 22 --cidr 203.0.113.0/32
```

Verify the security group details:

```bash
aws ec2 describe-security-groups --group-ids <group-id>
```

## Example (illustrative output)

After `create-security-group` you may see:

```json
{
  "GroupId": "sg-04c7eb3b7c64778d0",
  "SecurityGroupArn": "arn:aws:ec2:us-east-1:123456789012:security-group/sg-04c7eb3b7c64778d0"
}
```

After adding rules, `describe-security-groups` will show `IpPermissions` for ports you opened.

## Security recommendations
- Prefer restricting SSH (`22`) to a known IP range instead of `0.0.0.0/0`.
- Use separate security groups for different application tiers.

## Previewing this file in VS Code
- Open the file `day2.md`.
- Quick preview: Press `Ctrl+Shift+V`.
- Preview to the side: `Ctrl+K` then `V` (press `Ctrl+K`, release, then `V`).

If you want, I can add a small interactive checklist to track completion of each step.