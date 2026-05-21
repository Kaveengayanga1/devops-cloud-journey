# Day 32 - AWS RDS: Snapshot and Restore

## 📝 Task Description
The **Nautilus Development Team** is preparing for a major update to their database infrastructure. To ensure a smooth transition and to safeguard data, the team has requested the DevOps team to take a snapshot of the current RDS instance and restore it to a new instance. 

This process is crucial for testing and validation purposes before the update is rolled out to the production environment. The snapshot will serve as a backup, and the new instance will be used to verify that the backup process works correctly and that the application can function seamlessly with the restored data.

### 📋 Requirements
As a member of the Nautilus DevOps Team, your task is to:
1.  **Take a Snapshot:** Take a snapshot of the `nautilus-rds` RDS instance and name it `nautilus-snapshot` (ensure `nautilus-rds` is available first).
2.  **Restore the Snapshot:** Restore the snapshot to a new RDS instance named `nautilus-snapshot-restore`.
3.  **Configure the New Instance:** Ensure that the new RDS instance has a class of `db.t3.micro`.
4.  **Verify:** The new RDS instance must be in the `Available` state upon completion.

---

## 📚 Core Theories
To complete this task successfully, it's essential to understand the following concepts regarding AWS RDS and backup strategies:

1.  **AWS RDS (Relational Database Service):**
    *   A managed database service provided by AWS. AWS handles the underlying infrastructure, patching, and backups, while you manage the database optimization and data.

2.  **Database Snapshot:**
    *   A snapshot is a storage volume backup of your DB instance at a specific point in time. It creates a complete backup of the entire DB instance, not just individual databases.
    *   Snapshots are crucial for Disaster Recovery (DR) and creating test environments.

3.  **Snapshot Restoration:**
    *   A key feature of RDS is that restoring a snapshot **creates a new DB instance**. It does not overwrite the existing one. This allows you to test updates or data integrity on a clone of your production database without risking the live data.

4.  **Instance Class Modification:**
    *   When restoring from a snapshot, you can change the DB instance class (e.g., CPU, RAM). For example, creating a `db.t3.micro` instance from a snapshot of a larger production server allows for cost-effective testing.

5.  **States & Waiters:**
    *   Database operations (creating, backing up, restoring) are asynchronous and take time. They go through states like `creating`, `modifying`, `backing-up`, and finally `available`.
    *   In automation scripts, we use **AWS CLI Waiters** (`aws rds wait ...`) to pause execution until a resource reaches a specific state, ensuring the next command doesn't fail.

---

## 🚀 Execution Steps (AWS CLI)

Below is the professional approach to executing this task using the AWS CLI, including waiters to ensure robust automation.

### 1. Wait for the Instance to be Available
Before taking a snapshot, ensure the source database is in a stable state.

```bash
aws rds wait db-instance-available \
    --db-instance-identifier nautilus-rds
```

### 2. Create the DB Snapshot
Initiate the snapshot process for `nautilus-rds`.

```bash
aws rds create-db-snapshot \
    --db-instance-identifier nautilus-rds \
    --db-snapshot-identifier nautilus-snapshot
```

### 3. Wait for the Snapshot to Complete
Snapshot creation takes time. Wait until it is fully completed before trying to restore.

```bash
aws rds wait db-snapshot-available \
    --db-snapshot-identifier nautilus-snapshot
```

### 4. Restore the DB Instance from Snapshot
Create the new instance (`nautilus-snapshot-restore`) from the snapshot. Note the `--db-instance-class` parameter to resize it to `db.t3.micro`.

```bash
aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier nautilus-snapshot-restore \
    --db-snapshot-identifier nautilus-snapshot \
    --db-instance-class db.t3.micro
```

### 5. Wait for the New Instance to be Available
The restoration process creates a new instance. Wait for it to become available for use.

```bash
aws rds wait db-instance-available \
    --db-instance-identifier nautilus-snapshot-restore
```

### 6. Verify the New Instance Status
Confirm the details of the new instance to ensure it matches requirements (Name, Status, Class).

```bash
aws rds describe-db-instances \
    --db-instance-identifier nautilus-snapshot-restore \
    --query "DBInstances[*].[DBInstanceIdentifier,DBInstanceStatus,DBInstanceClass]" \
    --output table
```

#### Expected Output
```text
-----------------------------------------------------------
|                   DescribeDBInstances                   |
+----------------------------+------------+---------------+
|  nautilus-snapshot-restore |  available |  db.t3.micro  |
+----------------------------+------------+---------------+
```
