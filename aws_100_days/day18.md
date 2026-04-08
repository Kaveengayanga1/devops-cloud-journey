# Day 18: Creating IAM Policies with AWS CLI

## Output

```
~ on ☁️  (us-east-1) ➜  aws iam create-policy \
    --policy-name iampolicy_mariyam \
    --policy-document file://policy.json \
    --region us-east-1

Error parsing parameter '--policy-document': Unable to load paramfile file://policy.json: [Errno 2] No such file or directory: 'policy.json'

~ on ☁️  (us-east-1) ✖ cat <<EOF > policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:DescribeSnapshots"
            ],
            "Resource": "*"
        }
    ]
}
EOF

~ on ☁️  (us-east-1) ➜  aws iam create-policy \
    --policy-name iampolicy_mariyam \
    --policy-document file://policy.json \
    --region us-east-1
{
    "Policy": {
        "PolicyName": "iampolicy_mariyam",
        "PolicyId": "ANPA3SJDSPV2WHR5CRKT5",
        "Arn": "arn:aws:iam::795180432757:policy/iampolicy_mariyam",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2026-01-27T15:21:30Z",
        "UpdateDate": "2026-01-27T15:21:30Z"
    }
}

~ on ☁️  (us-east-1) ➜  
```

## Theory

### Understanding IAM Policies & CLI

Before running commands, it is crucial to understand three core concepts: JSON Policy Structure, ARN, and CLI Command Syntax.

#### 1. JSON Policy Structure

AWS IAM policies are JSON documents that define permissions. To create a policy, you must construct a JSON object containing a Statement array. Each object in that array needs:

- **Effect**: Either Allow or Deny.
- **Action**: The specific API call you are allowing (e.g., ec2:DescribeInstances).
- **Resource**: The specific AWS infrastructure the action applies to. For "view all" permissions, we typically use the wildcard `*`.

#### 2. Read-Only Access (The "Describe" Actions)

In AWS, "Read-Only" access usually maps to API actions that start with Describe or List. To view instances, AMIs, and snapshots, you specifically need:

- `ec2:DescribeInstances`
- `ec2:DescribeImages` (for AMIs)
- `ec2:DescribeSnapshots`

#### 3. The aws iam create-policy Command

The CLI command requires a name for the policy and the policy document itself. The document can be passed as a string or a file. Using a file is generally safer and less prone to syntax errors.

## Step-by-Step Implementation

Here is how to create the policy `iampolicy_mariyam` in us-east-1 with the required read-only permissions.

### Step 1: Create the Policy Document File

First, create a file named `policy.json` on your local machine. This keeps your command clean and makes it easier to edit permissions later.

**File Content (policy.json):**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:DescribeSnapshots"
            ],
            "Resource": "*"
        }
    ]
}
```

**Note:** The `Resource: "*"` is necessary here because "Describe" actions usually cannot be restricted to specific resources (you list all instances or none).

### Step 2: Execute the CLI Command

Run the following command in your terminal. This command tells AWS to read your JSON file and create the policy object in IAM.

```bash
aws iam create-policy \
    --policy-name iampolicy_mariyam \
    --policy-document file://policy.json \
    --region us-east-1
```

**Parameter Breakdown:**

- `--policy-name`: Sets the name to `iampolicy_mariyam`.
- `--policy-document`: Points to the local file (prefix with `file://`).
- `--region`: Specifies us-east-1 (though IAM is global, specifying the region ensures your CLI session connects to the correct endpoint).

### Step 3: Verify the Policy Creation

After running the command, you should receive a JSON output confirming the creation. You can verify it exists by running:

```bash
aws iam get-policy --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/iampolicy_mariyam
```

(Replace `YOUR_ACCOUNT_ID` with your actual 12-digit AWS account ID).

## Summary of Permissions

| Permission | CLI Action | Description |
|------------|-----------|-------------|
| View Instances | `ec2:DescribeInstances` | Allows the user to see the list of EC2 servers. |
| View AMIs | `ec2:DescribeImages` | Allows the user to see available Amazon Machine Images. |
| View Snapshots | `ec2:DescribeSnapshots` | Allows the user to see EBS volume backups. |

## Troubleshooting: Missing policy.json File

This error happened because the file `policy.json` does not exist in your current folder. The command `--policy-document file://policy.json` tells the AWS CLI to look for a specific file on your disk, but you haven't created it yet.

You have two ways to fix this:

### Option 1: Create the file first (Recommended)

Run this command in your terminal. It uses `cat` to create the file `policy.json` with the correct content right where you are standing.

```bash
cat <<EOF > policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:DescribeSnapshots"
            ],
            "Resource": "*"
        }
    ]
}
EOF
```

Now, run your original command again:

```bash
aws iam create-policy \
    --policy-name iampolicy_mariyam \
    --policy-document file://policy.json \
    --region us-east-1
```

### Option 2: Run without a file (Inline JSON)

If you don't want to create a file, you can pass the JSON directly inside the command using single quotes. This is faster for simple policies.

```bash
aws iam create-policy \
    --policy-name iampolicy_mariyam \
    --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":["ec2:DescribeInstances","ec2:DescribeImages","ec2:DescribeSnapshots"],"Resource":"*"}]}' \
    --region us-east-1
```

**Key Note:** I used single quotes `'...'` around the policy document string. This is crucial on Linux/Mac terminals to prevent the shell from misinterpreting the JSON characters.
