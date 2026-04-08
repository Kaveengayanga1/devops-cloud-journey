# Day 17: AWS IAM Group Creation via CLI

## Output

```
~ on ☁️  (us-east-1) ➜  aws iam create-group --group-name iamgroup_john
{
    "Group": {
        "Path": "/",
        "GroupName": "iamgroup_john",
        "GroupId": "AGPAWRGJ7YPLE6B4J42IG",
        "Arn": "arn:aws:iam::449247691734:group/iamgroup_john",
        "CreateDate": "2026-01-22T02:13:23Z"
    }
}

~ on ☁️  (us-east-1) ➜  
```

## Theory

Creating an IAM group is the professional way to manage permissions. Instead of attaching policies to every individual user, you manage the group, and users "inherit" those permissions.

### Core Theories of IAM Groups

#### 1. Permission Inheritance

Think of a group as a "folder" for permissions. When you add a user to the group, they automatically gain every permission attached to that group. When you remove them, those permissions are instantly revoked.

**Best Practice:** Never attach a policy directly to a user. Always attach it to a group and move the user into the group.

#### 2. Group Membership

An IAM user can belong to multiple groups (up to 10 by default). Their total permissions are the sum of all policies attached to every group they are in, plus any policies attached directly to them.

#### 3. No Nesting

Unlike folders on your computer, IAM groups cannot be nested. You cannot put a "Junior Devs" group inside a "Developers" group. All groups exist at the same level.

#### 4. Groups are not "Identities"

Unlike users or roles, groups cannot "sign in." They don't have passwords or access keys. They are purely an organizational tool for managing user permissions.

## Step-by-Step: Creating a Group via CLI

Since you are preparing for an internship (starting around May!), mastering the CLI is a great way to show you're ready for DevOps or Cloud Engineering tasks.

### Step 1: Create the Group

This command creates the group container.

```bash
aws iam create-group --group-name DevTeam
```

### Step 2: Attach a Policy to the Group

You need to tell AWS what this group is allowed to do. Let's give this group full access to EC2 (virtual servers).

```bash
aws iam attach-group-policy --group-name DevTeam --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

### Step 3: Add Your User to the Group

Now, move your previously created user (e.g., "Alice") into this group.

```bash
aws iam add-user-to-group --user-name Alice --group-name DevTeam
```

### Step 4: Verify Membership

Check that the user is successfully in the group:

```bash
aws iam get-group --group-name DevTeam
```

## Pro-Tip for your Portfolio

When you start your internship, you'll likely see "Job Function" policies. AWS provides pre-made policies like ReadOnlyAccess, PowerUserAccess, and AdministratorAccess. Using these standard policies for your groups makes your infrastructure much easier to audit.
