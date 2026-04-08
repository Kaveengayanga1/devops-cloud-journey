# Day 23: AWS S3 Data Migration

## Scenario

As part of a data migration project, the team lead has tasked the team with migrating data from an existing S3 bucket to a new S3 bucket. The existing bucket contains a substantial amount of data that must be accurately transferred to the new bucket. The team is responsible for creating the new S3 bucket and ensuring that all data from the existing bucket is copied or synced to the new bucket completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new bucket without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

1.  **Create a New Private S3 Bucket**: Name the bucket `devops-sync-31802`.
2.  **Data Migration**: Migrate the entire data from the existing `devops-s3-2123` bucket to the new `devops-sync-31802` bucket.
3.  **Ensure Data Consistency**: Ensure that both buckets have the same data.
4.  **Use AWS CLI**: Use the AWS CLI to perform the creation and data migration tasks.

---

## Theory & Concepts

To successfully complete this data migration task, it's important to understand the following concepts:

### 1. What is AWS S3?
Amazon S3 (Simple Storage Service) is a cloud storage service that allows you to store and retrieve any amount of data from anywhere on the internet. In S3, data is stored as **Objects** inside containers called **Buckets**.

### 2. AWS CLI (Command Line Interface)
The AWS CLI is a unified tool to manage your AWS services. It enables you to control multiple AWS services from the command line and automate them through scripts, making tasks like data migration much easier and faster than using the web console.

### 3. S3 `sync` vs `cp`
*   **`cp` (Copy)**: Generally used to copy a file or object from source to destination. It is suitable for one-time copies of specific files.
*   **`sync` (Synchronize)**: This command is highly efficient for migration. It compares the files in the source and destination and only copies new or modified files. For large data migrations, `sync` is preferred as it ensures the destination matches the source recursively.

---

## Implementation Steps

Ensure your AWS CLI is configured before proceeding.

### Step 1: Create the New S3 Bucket
First, create the new bucket with the specified name (`devops-sync-31802`) using the `mb` (make bucket) command.

```bash
aws s3 mb s3://devops-sync-31802
```
*Note: When you create an S3 bucket, it is private by default.*

### Step 2: Data Migration (Sync)
Migrate all data from the old bucket (`devops-s3-2123`) to the new one using the `sync` command. This will copy all files and folders recursively.

```bash
aws s3 sync s3://devops-s3-2123 s3://devops-sync-31802
```

### Step 3: Verify Data Consistency
To ensure the migration was successful and accurate, use the following verification methods:

**A) List and Summarize Data**
Run the `ls` command with `--summarize` on both buckets to compare the total object count and size.

```bash
# Check source bucket
aws s3 ls s3://devops-s3-2123 --recursive --human-readable --summarize

# Check destination bucket
aws s3 ls s3://devops-sync-31802 --recursive --human-readable --summarize
```

**B) Dry Run Check**
You can run a `sync` with the `--dryrun` flag. If the buckets are identical, this command should produce no output (indicating no files need to be transferred).

```bash
aws s3 sync s3://devops-s3-2123 s3://devops-sync-31802 --dryrun
```

## Summary Table

| Task | Command |
| :--- | :--- |
| **Create Bucket** | `aws s3 mb s3://devops-sync-31802` |
| **Sync Data** | `aws s3 sync s3://devops-s3-2123 s3://devops-sync-31802` |
| **Verify** | `aws s3 ls s3://devops-sync-31802 --recursive` |
