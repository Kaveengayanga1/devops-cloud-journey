# Day 16: AWS IAM User Creation via CLI

## Output

```
~ on ☁️  (us-east-1) ➜  aws iam create-user --user-name iamuser_kirsty
{
    "User": {
        "Path": "/",
        "UserName": "iamuser_kirsty",
        "UserId": "AIDAR7XTVPLU7C2CHFL5X",
        "Arn": "arn:aws:iam::136876227305:user/iamuser_kirsty",
        "CreateDate": "2026-01-22T02:08:34Z"
    }
}

~ on ☁️  (us-east-1) ➜  
```

## Theory

To create an Identity and Access Management (IAM) user via the AWS Command Line Interface (CLI), it helps to understand the "why" and "how" of the AWS security model first.

### Core Theories of IAM

Before running commands, you should be familiar with these three foundational concepts:

#### 1. The Principle of Least Privilege (PoLP)

This is the golden rule of AWS security. You should only grant a user the minimum permissions required to perform their specific task. Creating an IAM user with "AdministratorAccess" for a simple task is a major security risk.

#### 2. Authentication vs. Authorization

- **Authentication**: Proving who you are (Usernames, Passwords, Access Keys).
- **Authorization**: Proving what you are allowed to do (Policies).

When you create a user in the CLI, you are creating an identity, but that identity has zero permissions until you attach a policy.

#### 3. IAM Entities

- **User**: A person or application that needs to interact with AWS.
- **Group**: A collection of users. It's easier to manage permissions by adding users to groups (e.g., a "Developers" group).
- **Policy**: A JSON document that defines permissions.

## Step-by-Step: Creating a User via CLI

Ensure you have the AWS CLI installed and configured with administrative credentials before starting.

### Step 1: Create the User

This command creates the user entity within your AWS account.

```bash
aws iam create-user --user-name Alice
```

### Step 2: Create Access Keys (Optional)

If this user needs to use the CLI or API, they need access keys. Note: This will output the SecretAccessKey only once.

```bash
aws iam create-access-key --user-name Alice
```

### Step 3: Attach a Policy

Currently, "Alice" can't do anything. To give her permission (for example, Read-Only access to S3), attach a managed policy:

```bash
aws iam attach-user-policy --user-name Alice --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

## Summary of Useful Commands

| Action | CLI Command |
|--------|-------------|
| List Users | `aws iam list-users` |
| Add to Group | `aws iam add-user-to-group --user-name <name> --group-name <group>` |
| Create Login Profile | `aws iam create-login-profile --user-name <name> --password <pass>` |

## Pro-Tip

For production environments, it is often better to use IAM Roles instead of long-term IAM Users with access keys to reduce the risk of credential leaks.
