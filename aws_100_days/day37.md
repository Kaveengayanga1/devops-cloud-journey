# Day 37 — AWS EC2 + S3 Integration with IAM Roles & SSH Key Setup

**Date:** Sunday, May 17, 2026
**Topic:** AWS EC2, S3, IAM Policy/Role, Instance Profile, SSH Key Authentication
**Platform:** KodeKloud Lab (Nautilus DevOps Series)
**Region:** us-east-1

---

## Lab Credentials & Environment Info

### Attempt 1 (Failed)
| Field | Value |
|---|---|
| Console URL | https://113774153327.signin.aws.amazon.com/console?region=us-east-1 |
| Username | kk_labs_user_995580 |
| Password | eHx6Xyg@I9Ei |
| Start Time | Sun May 17 03:53:40 UTC 2026 |
| End Time | Sun May 17 04:53:40 UTC 2026 |
| EC2 Instance Name | datacenter-ec2 |
| EC2 Public IP | 52.90.136.149 |
| EC2 Instance ID | i-05028b7f4dbb18202 |
| S3 Bucket | datacenter-s3-113774153327 |
| IAM Role | datacenter-role |
| AWS Account ID | 113774153327 |

### Attempt 2 (Passed)
| Field | Value |
|---|---|
| Console URL | https://566733028177.signin.aws.amazon.com/console?region=us-east-1 |
| Username | kk_labs_user_702772 |
| Password | 3^1VpG0isIH2 |
| Start Time | Sun May 17 04:11:35 UTC 2026 |
| End Time | Sun May 17 05:11:35 UTC 2026 |
| EC2 Instance Name | nautilus-ec2 |
| EC2 Public IP | 44.215.105.82 |
| EC2 Instance ID | i-09b7fab714c788267 |
| S3 Bucket | nautilus-s3-566733028177 |
| IAM Role | nautilus-role |
| AWS Account ID | 566733028177 |

---

## Task Overview

The Nautilus DevOps team needed to:

1. Use an existing EC2 instance
2. Generate an SSH key pair on the aws-client host and add the public key to the EC2 instance's root user
3. Create a private S3 bucket
4. Create an IAM policy granting `s3:PutObject`, `s3:GetObject`, and `s3:ListBucket` access
5. Create an IAM role and attach the policy to it
6. Attach the IAM role to the EC2 instance via an Instance Profile
7. SSH into EC2 and test file upload/list to S3

---

## Theory

### AWS IAM (Identity and Access Management)

IAM is the AWS service that controls who can access which AWS resources and what they can do with them.

**Key Components:**

- **IAM Policy** — A JSON document that defines permissions (allow/deny) for specific AWS actions on specific resources. Example actions: `s3:PutObject`, `s3:GetObject`, `s3:ListBucket`
- **IAM Role** — An identity that AWS services (like EC2) can assume to gain permissions defined in attached policies. Unlike users, roles don't have permanent credentials — they issue temporary credentials via AWS STS (Security Token Service)
- **Instance Profile** — A container that holds an IAM role and is used to pass that role to an EC2 instance. When you create a role via the AWS Console, an Instance Profile with the same name is automatically created. When using the CLI, you must create it manually

### IAM Instance Profile vs Hardcoded Credentials

| Method | Security | Recommended |
|---|---|---|
| Hardcoded Access Keys in EC2 | Low — Keys can leak via code, logs, or metadata | No |
| IAM Instance Profile (Role) | High — Temporary credentials via IMDS, auto-rotated | Yes |

When EC2 uses an Instance Profile, it fetches temporary credentials from the **EC2 Instance Metadata Service (IMDS)** at `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`. These credentials expire and rotate automatically.

### AWS S3 (Simple Storage Service)

S3 is AWS's object storage service used to store and retrieve files (objects) in containers called **buckets**.

- **Bucket** — The top-level container for storing objects
- **Private Bucket** — A bucket with all public access blocked. Access is controlled entirely through IAM policies or bucket policies
- **Block Public Access** — A set of 4 settings that prevent any public ACL or policy from making objects publicly accessible:
  - `BlockPublicAcls`
  - `IgnorePublicAcls`
  - `BlockPublicPolicy`
  - `RestrictPublicBuckets`

### IAM Policy Resource ARN Structure for S3

When writing an S3 IAM policy, two resource ARN formats are needed:

```
arn:aws:s3:::bucket-name        → For bucket-level actions (ListBucket)
arn:aws:s3:::bucket-name/*      → For object-level actions (PutObject, GetObject)
```

Both must be included in the `Resource` array. Omitting either one causes permission errors.

### SSH Key Authentication

SSH (Secure Shell) uses asymmetric cryptography for passwordless authentication.

- **Private Key (`id_rsa`)** — Kept secret on the client machine. Never share this file
- **Public Key (`id_rsa.pub`)** — Added to the server's `~/.ssh/authorized_keys` file. Safe to share
- **Authentication Flow:**
  1. Client requests login
  2. Server sends a challenge encrypted with the public key
  3. Client decrypts the challenge using the private key and sends back the response
  4. Server verifies and grants access

---

## Architecture Diagram

```
[aws-client host]
      |
      | SSH using ~/.ssh/id_rsa (private key)
      ↓
[nautilus-ec2 (EC2 Instance)]
      |
      | Assumes IAM Role via Instance Profile
      ↓
[AWS STS → Temporary Credentials]
      |
      ↓
[nautilus-s3-566733028177 (Private S3 Bucket)]

IAM Chain:
nautilus-s3-policy → nautilus-role → nautilus-role (Instance Profile) → nautilus-ec2
```

---

## Step-by-Step Execution (Attempt 2 — Successful)

### Step 1: Get EC2 Public IP

```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[0].Instances[0].PublicIpAddress" \
  --output text
```

Output: `44.215.105.82`

### Step 2: Generate SSH Key Pair on aws-client

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

View the generated public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Public key generated:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCxAteLGayptnKqAgKmKF0eQbwUropxJ5rp5Dz9N2SdhbZ/+vyUu0YjICL9oAvqyYi/rzXHUpJ5lW1HRDVKuD3/C4oRp5mexPE1eI0KPSlv5i7w2KYgX4D4ldboXt50tI+30UUkQNP+daYBbHGbuYkE/pn0hiUS/be9B/jut8j10VFAEuUZ5+8uoqwrTmxoJQQzCL84m/5Kv60I/klCLuk68KIzFkuiFWlCVXjmNv1kCwujQt25ujKv78LX8NnUKAIALp16Fjt2Vab4XKLl/s9Ykq80LfTcQiJiAaidv/+bxlKxGsMmNAYEsdEWb3digOs4VHRi727H7Fb1iR8mbpgpjvWdSkkrq7moyAEEPWUBBx1EoITl59ZFsS+PGu0Rea5N+pYH5xXO7UVVXfj7bOs4WM4uUJdBxVyG6AJGfHj0zR18A6Y8umKbNMaRhMWyPY8M3XuAz7lhOveEiSU64nD/Ha6rhnfDk+66nr0npzqUl6XiWDevNfbD1SsKTrBW1LAszGanHiu4iIZij8K2uoNBG7+kaC2htn2EuCPUq6sl3U+LN2iu9IoLTYXz6B7p2rmOOnWLVvVXBnwyxK529LEFewjPkGEDzD3/hslwAtOlOV6XZnfrXYBAvqCgoI8EWPYwdzMH2vaP40NDLcbrPryB+Dc36sHzQIsG0cSpy+G8nw== root@aws-client
```

### Step 3: Add Public Key to EC2 via AWS Console (EC2 Instance Connect)

Since no PEM key existed to initially SSH into the instance, the public key was added using AWS Console's browser-based terminal:

1. Logged into AWS Console at `https://566733028177.signin.aws.amazon.com/console?region=us-east-1`
2. Navigated to EC2 → Instances → `nautilus-ec2` → Connect → EC2 Instance Connect → Connect
3. In the browser terminal, ran:

```bash
sudo su -
mkdir -p /root/.ssh
chmod 700 /root/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB...== root@aws-client" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
cat /root/.ssh/authorized_keys
```

### Step 4: Verify SSH from aws-client

```bash
ssh -i ~/.ssh/id_rsa root@44.215.105.82
```

Successfully connected. Then exited back to aws-client.

### Step 5: Create Private S3 Bucket

```bash
aws s3api create-bucket \
  --bucket nautilus-s3-566733028177 \
  --region us-east-1
```

Block all public access:

```bash
aws s3api put-public-access-block \
  --bucket nautilus-s3-566733028177 \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

Verify:

```bash
aws s3api get-public-access-block --bucket nautilus-s3-566733028177
```

Output confirmed all four settings as `true`.

### Step 6: Create IAM Policy

```bash
cat > /tmp/s3-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::nautilus-s3-566733028177",
        "arn:aws:s3:::nautilus-s3-566733028177/*"
      ]
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name nautilus-s3-policy \
  --policy-document file:///tmp/s3-policy.json
```

Policy ARN: `arn:aws:iam::566733028177:policy/nautilus-s3-policy`

### Step 7: Create IAM Role

```bash
cat > /tmp/trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name nautilus-role \
  --assume-role-policy-document file:///tmp/trust-policy.json
```

Role ARN: `arn:aws:iam::566733028177:role/nautilus-role`

### Step 8: Attach Policy to Role

```bash
aws iam attach-role-policy \
  --role-name nautilus-role \
  --policy-arn arn:aws:iam::566733028177:policy/nautilus-s3-policy
```

Verified with:

```bash
aws iam list-attached-role-policies --role-name nautilus-role
```

### Step 9: Create Instance Profile (same name as role)

```bash
aws iam create-instance-profile \
  --instance-profile-name nautilus-role

aws iam add-role-to-instance-profile \
  --instance-profile-name nautilus-role \
  --role-name nautilus-role
```

Instance Profile ARN: `arn:aws:iam::566733028177:instance-profile/nautilus-role`

### Step 10: Attach Instance Profile to EC2

```bash
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)

aws ec2 associate-iam-instance-profile \
  --instance-id $INSTANCE_ID \
  --iam-instance-profile Name=nautilus-role
```

Verified State: `associated`

### Step 11: Test S3 Access from EC2

SSH into EC2:

```bash
ssh -i ~/.ssh/id_rsa root@44.215.105.82
```

Verify IAM role assumption:

```bash
aws sts get-caller-identity
```

Output:
```json
{
    "UserId": "AROAYH47LXNI4GTMK2BSV:i-09b7fab714c788267",
    "Account": "566733028177",
    "Arn": "arn:aws:sts::566733028177:assumed-role/nautilus-role/i-09b7fab714c788267"
}
```

Create and upload a test file:

```bash
echo "Nautilus DevOps Test - $(date)" > /tmp/testfile.txt
aws s3 cp /tmp/testfile.txt s3://nautilus-s3-566733028177/
```

Output: `upload: ../tmp/testfile.txt to s3://nautilus-s3-566733028177/testfile.txt`

List the bucket:

```bash
aws s3 ls s3://nautilus-s3-566733028177/
```

Output: `2026-05-17 04:22:57    52 testfile.txt`

---

## Mistakes Made, Why They Happened & How They Were Fixed

### Mistake 1: SSH Key — No Existing PEM Key to Bootstrap

**What happened:**
The initial guide assumed an existing PEM key was available to SSH into the EC2 instance to add the new public key. In reality, the lab environment did not provide one. `ssh-copy-id` failed with `Permission denied (publickey)` because there was no way to authenticate initially.

**Why it happened:**
In standard AWS setups, a key pair is attached at instance launch time. KodeKloud lab instances sometimes use a different access method or have no key pair attached.

**How it was fixed:**
Used **AWS Console → EC2 Instance Connect** — a browser-based terminal that uses AWS's own IAM-authenticated SSH mechanism. This allowed access to the instance without any pre-existing key. From the browser terminal, the new public key was manually appended to `/root/.ssh/authorized_keys`.

---

### Mistake 2: Instance Profile Named Differently from IAM Role (Attempt 1 Failed)

**What happened:**
In Attempt 1, the Instance Profile was created with the name `datacenter-instance-profile` while the IAM Role was named `datacenter-role`. The KodeKloud grader checked for the role association by looking for an Instance Profile named `datacenter-role` and did not find it, causing the task to fail with:

```
IAM role datacenter-role is not associated with the instance datacenter-ec2
```

**Why it happened:**
When you create an IAM Role through the **AWS Console**, an Instance Profile is automatically created with the **exact same name** as the role. This is a Console convenience that is not replicated when using the CLI. Through the CLI, the Instance Profile must be created separately, and there is no enforcement that the names match. The incorrect assumption was that any name was acceptable.

**How it was fixed:**
In Attempt 2, the Instance Profile was created with the same name as the IAM role:

```bash
aws iam create-instance-profile --instance-profile-name nautilus-role
aws iam add-role-to-instance-profile --instance-profile-name nautilus-role --role-name nautilus-role
```

This matched what the grader expected and the task passed.

---

### Mistake 3: SSM Session Manager Failed (TargetNotConnected)

**What happened:**
During Attempt 1 troubleshooting, SSM Session Manager was tried as an alternative way to access the EC2 instance:

```
An error occurred (TargetNotConnected) when calling the StartSession operation: i-05028b7f4dbb18202 is not connected.
```

**Why it happened:**
SSM Session Manager requires the **SSM Agent** to be running on the EC2 instance and the instance must have a VPC endpoint or internet access to reach the SSM service endpoint. The lab instance either did not have SSM Agent running or lacked the necessary IAM permissions/network access.

**How it was fixed:**
SSM was abandoned and EC2 Instance Connect (browser terminal) was used instead, which worked without any additional agent or configuration.

---

## Key Lessons Learned

**Instance Profile naming is critical when using CLI:**
Always name the Instance Profile identically to the IAM Role when working via CLI. This matches the behavior of the AWS Console and is what most tools, graders, and automation systems expect.

**EC2 Instance Connect is a reliable bootstrap method:**
When no PEM key exists, AWS Console's EC2 Instance Connect provides IAM-authenticated browser terminal access. This can be used to add SSH public keys, run initial setup commands, or troubleshoot without any pre-existing credentials.

**S3 IAM Policy needs both ARN formats:**
`arn:aws:s3:::bucket-name` for `ListBucket` (bucket level) and `arn:aws:s3:::bucket-name/*` for `PutObject`/`GetObject` (object level). Both must be in the Resource array.

**IAM Role on EC2 eliminates need for stored credentials:**
`aws sts get-caller-identity` confirming `assumed-role/nautilus-role` proves the instance is using temporary credentials from IMDS — no Access Key or Secret Key is stored anywhere on the instance.

---

## Final Checklist

| Task | Status |
|---|---|
| SSH Key Pair generated on aws-client | Done |
| Public key added to EC2 root user via Console | Done |
| SSH connection verified from aws-client | Done |
| Private S3 bucket created | Done |
| All public access blocked on S3 bucket | Done |
| IAM Policy created with correct actions and resources | Done |
| IAM Role created with EC2 trust policy | Done |
| Policy attached to role | Done |
| Instance Profile created with same name as role | Done |
| Role added to Instance Profile | Done |
| Instance Profile attached to EC2 | Done |
| IAM role assumption verified via sts get-caller-identity | Done |
| Test file uploaded to S3 | Done |
| Uploaded file listed from S3 | Done |