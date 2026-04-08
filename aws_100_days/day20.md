# Day 20: Creating an IAM Role and Attaching Policies via AWS CLI

Today's task involved creating an IAM role named `iamrole_mariyam`, configuring it to be assumable by EC2 instances (Trust Policy), and attaching an existing custom policy `iampolicy_mariyam` (Permissions Policy).

## Prerequisite: Verify Policy Existence

Before creating the role, we verified that the custom policy `iampolicy_mariyam` exists and retrieved its ARN.

```bash
aws iam list-policies --scope Local --query "Policies[?PolicyName=='iampolicy_mariyam']"
```

**Output:**
```json
[
    {
        "PolicyName": "iampolicy_mariyam",
        "PolicyId": "ANPAQIGQBZ7JBLM7GJUIH",
        "Arn": "arn:aws:iam::017616195538:policy/iampolicy_mariyam",
        ...
    }
]
```

## Step 1: Create the Trust Policy

The Trust Policy defines *who* can assume the role. For this task, the entity type is **AWS Service** and the use case is **EC2**.

We created a file named `trust-policy.json`:

```json
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
```

## Step 2: Create the IAM Role

Using the trust policy file, we created the role `iamrole_mariyam`.

```bash
aws iam create-role \
    --role-name iamrole_mariyam \
    --assume-role-policy-document file://trust-policy.json
```

**Output:**
```json
{
    "Role": {
        "Path": "/",
        "RoleName": "iamrole_mariyam",
        "RoleId": "AROAQIGQBZ7JCUTQQBIMX",
        "Arn": "arn:aws:iam::017616195538:role/iamrole_mariyam",
        "CreateDate": "2026-02-12T13:27:33Z",
        "AssumeRolePolicyDocument": { ... }
    }
}
```

## Step 3: Attach the Permissions Policy

We attached the existing policy `iampolicy_mariyam` to the new role using its ARN.

```bash
aws iam attach-role-policy \
    --role-name iamrole_mariyam \
    --policy-arn arn:aws:iam::017616195538:policy/iampolicy_mariyam
```

## Step 4: Verification

We verified the policy was correctly attached.

```bash
aws iam list-attached-role-policies --role-name iamrole_mariyam
```

**Output:**
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "iampolicy_mariyam",
            "PolicyArn": "arn:aws:iam::017616195538:policy/iampolicy_mariyam"
        }
    ]
}
```

## Theory: Key Concepts

1.  **IAM Role (The Identity)**: A role is an identity with specific permissions that is not associated with a specific person. It is "assumed" by trusted entities (like an EC2 instance).
2.  **Trust Policy**: Defined in `trust-policy.json`, this answers "Who is allowed to use this role?". We specified `ec2.amazonaws.com`.
3.  **Permissions Policy**: The attached policy (`iampolicy_mariyam`) answers "What can this role do?".
4.  **Instance Profile**: When using the CLI, an instance profile must be manually created to pass the role information to an EC2 instance (unlike the Console which does this automatically).
