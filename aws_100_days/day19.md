# Day 19: Attaching IAM Policies to Users with AWS CLI

## Part 1: Theories You Need to Know

Before running commands, it is important to understand three core concepts to avoid permission errors or security gaps.

### 1. Identity vs. Permissions

**IAM User (Identity):** The "Who." This is the entity attempting to access AWS (e.g., jdoe). By default, a new user has zero permissions.

**IAM Policy (Permissions):** The "What." A JSON document that defines what actions are allowed or denied (e.g., "Allow reading files from S3").

**The Attachment:** You must link the Policy to the User to grant access.

### 2. Managed vs. Inline Policies (Crucial Distinction)

The AWS CLI uses different commands depending on the type of policy you are using.

**Managed Policies (Recommended):** Standalone policies that can be attached to multiple users, groups, or roles (e.g., AdministratorAccess or a custom policy you created).
- Command used: `attach-user-policy`

**Inline Policies:** Policies embedded directly into a single user. They cannot be reused.
- Command used: `put-user-policy`

### 3. The ARN (Amazon Resource Name)

When using the web console, you select policies by their checkbox. When using the CLI, you must identify resources by their ARN.

Format: `arn:aws:iam::aws:policy/PolicyName`

## Part 2: The Process (AWS CLI)

**Prerequisite:** Ensure your AWS CLI is installed and configured with valid credentials (`aws configure`).

### Step 1: Identify the IAM User

First, find the exact username you want to target.

```bash
aws iam list-users --output table
```

### Step 2: Find the Policy ARN

You need the ARN of the policy you want to attach.

**Option A: Find an AWS Managed Policy** (e.g., S3 Read Only)
Use grep (Linux/Mac) or findstr (Windows) to filter the massive list of default policies.

```bash
# Linux/Mac
aws iam list-policies --query 'Policies[*].[PolicyName, Arn]' --output text | grep "S3ReadOnly"

# Windows (PowerShell)
aws iam list-policies --query 'Policies[*].[PolicyName, Arn]' --output text | Select-String "S3ReadOnly"
```

Result Example: `AmazonS3ReadOnlyAccess arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess`

**Option B: Find a Customer Managed Policy** (Created by you)
Add `--scope Local` to list only policies created in your account.

```bash
aws iam list-policies --scope Local --output table
```

### Step 3: Attach the Policy

Use the `attach-user-policy` command. This links the specific ARN to the specific Username.

**Syntax:**
```bash
aws iam attach-user-policy --user-name <username> --policy-arn <policy_arn>
```

**Example:** To give user jdoe read-only access to S3:

```bash
aws iam attach-user-policy \
    --user-name jdoe \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

*(Note: If the command is successful, there is no output.)*

### Step 4: Verify the Attachment

Always confirm that the policy was successfully attached.

```bash
aws iam list-attached-user-policies --user-name jdoe
```

Output should look like this:

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonS3ReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
        }
    ]
}
```

## Summary Table: CLI Commands

| Action | Command |
|--------|---------|
| List Users | `aws iam list-users` |
| Find Policy ARN | `aws iam list-policies` |
| Attach Managed Policy | `aws iam attach-user-policy` |
| Embed Inline Policy | `aws iam put-user-policy` |
| Check Permissions | `aws iam list-attached-user-policies` |

### 💡 Best Practice Note

While attaching policies directly to users works, the AWS Best Practice is to attach policies to Groups (e.g., a "Developers" group) and then add users to that group. This makes managing permissions for many users significantly easier.

```bash
aws iam attach-group-policy --group-name Developers --policy-arn ...
```

## Practical Output Example

### Get AWS Account ID

```bash
~ on ☁️  (us-east-1) ➜  aws sts get-caller-identity --query Account --output text
045946725843
```

### Attach Policy to User

```bash
~ on ☁️  (us-east-1) ➜  aws iam attach-user-policy \
    --user-name iamuser_kirsty  \
    --policy-arn arn:aws:iam::045946725843:policy/iampolicy_kirsty 
```

### Verify Attachment

```bash
~ on ☁️  (us-east-1) ➜  aws iam list-attached-user-policies --user-name iamuser_kirsty
{
    "AttachedPolicies": [
        {
            "PolicyName": "iampolicy_kirsty",
            "PolicyArn": "arn:aws:iam::045946725843:policy/iampolicy_kirsty"
        }
    ]
}

~ on ☁️  (us-east-1) ➜  
```

---

**Key Takeaway:** The attach-user-policy command is straightforward—provide the username and policy ARN, then verify with list-attached-user-policies to confirm success.
