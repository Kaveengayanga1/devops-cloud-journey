# Day 31: Provisioning a Private Amazon RDS Instance with Storage Autoscaling (AWS CLI)

## 📌 The Scenario
The Nautilus Development Team is working on a new application feature that requires a reliable and scalable database solution. To facilitate development and testing, they need a new **private RDS instance**. This instance will be used to store critical application data and must be provisioned using the AWS free tier (`db.t3.micro`) to minimize costs. The team has selected **MySQL 8.4.x** as the engine.

The task is to provision this instance named `xfusion-rds`, ensure it has **Storage Autoscaling** enabled (Max 50GB), and verify it is in the `available` state.

---

## 🏗️ Technical Components & Architecture Explained

1.  **Amazon RDS (Relational Database Service):**
    A managed service that makes it easy to set up, operate, and scale a relational database in the cloud. AWS handles time-consuming administrative tasks like hardware provisioning, database setup, patching, and backups.

2.  **Private RDS Instance:**
    A database instance is considered "private" when it is placed in a private subnet and is not accessible from the public internet. This enhances security by restricting access only to resources within the VPC (like application servers).

3.  **DB Subnet Group:**
    A collection of subnets (typically private) that you designate for your RDS DB instances in a VPC. An RDS DB subnet group must contain subnets from at least two Availability Zones in the region to ensure high availability.

4.  **Storage Autoscaling:**
    A feature that allows the RDS instance to automatically increase its storage capacity when free database space runs low. You define a maximum storage threshold (e.g., 50GB), and AWS handles the scaling without downtime.

---

## ☁️ DevOps Step-by-Step Execution Guide (AWS CLI)

To complete this task securely and efficiently, follow these AWS CLI steps.

### Step 1: Create a Security Group
First, create a security group that will control traffic to the RDS instance.
*(Replace `vpc-xxxxxxx` with your legitimate VPC ID)*

```bash
aws ec2 create-security-group \
    --group-name rds-private-sg \
    --description "SG for xfusion-rds" \
    --vpc-id vpc-xxxxxxx
```

### Step 2: Create a DB Subnet Group
RDS requires a subnet group to know which subnets to place the database in. Ensure you select subnets from at least two different Availability Zones.
*(Replace `subnet-xxxxxxx` and `subnet-yyyyyyy` with your actual private subnet IDs)*

```bash
aws rds create-db-subnet-group \
    --db-subnet-group-name xfusion-subnet-group \
    --db-subnet-group-description "Subnet group for private rds" \
    --subnet-ids "subnet-xxxxxxx" "subnet-yyyyyyy"
```

### Step 3: Provision the RDS Instance
Create the private MySQL instance with the specified configurations, including storage autoscaling.

```bash
aws rds create-db-instance \
    --db-instance-identifier xfusion-rds \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --engine-version 8.4.0 \
    --allocated-storage 20 \
    --max-allocated-storage 50 \
    --no-publicly-accessible \
    --db-subnet-group-name xfusion-subnet-group \
    --vpc-security-group-ids sg-xxxxxxx \
    --master-username admin \
    --master-user-password YourSecurePassword123
```

**Key Parameters:**
*   `--no-publicly-accessible`: Ensures the instance is not assigned a public IP address.
*   `--max-allocated-storage 50`: Enables storage autoscaling up to 50GB.
*   `--engine-version 8.4.0`: Specifies the requested MySQL version.

### Step 4: Verification
The instance creation process takes a few minutes. You can monitor the status using the following command. Wait until the status changes to `available`.

```bash
aws rds describe-db-instances \
    --db-instance-identifier xfusion-rds \
    --query "DBInstances[*].DBInstanceStatus"
```
