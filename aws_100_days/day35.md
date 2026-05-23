# Day 35: AWS RDS Private Instance & EC2 Connectivity

## Task Overview
The goal was to set up a private Amazon RDS (MySQL) instance and configure an existing EC2 instance (`datacenter-ec2`) to connect to it. This involved configuring networking (Security Groups), handling SSH access, and deploying a PHP application with correct database credentials.

### Environment Credentials (Archived)
**Note:** These credentials were valid during the session and no longer exist.
- **Console URL:** https://046473746767.signin.aws.amazon.com/console?region=us-east-1
- **Username:** `kk_labs_user_678717`
- **Session Password:** `U%t6Htv3OnqY`
- **RDS Master Username:** `datacenter_admin`
- **RDS Master Password:** `rdsInstance123` (later updated in PHP)
- **Database Name:** `datacenter_db`
- **EC2 Public IP:** `3.91.213.249`
- **RDS Endpoint:** `datacenter-rds.ccnghivlicsm.us-east-1.rds.amazonaws.com`

---

## 1. Core Theories

### Amazon RDS (Relational Database Service)
A managed service that makes it easy to set up, operate, and scale a relational database in the cloud. We used the **MySQL 8.4.5** engine.

### Security Groups (Virtual Firewalls)
Security groups act as a firewall for associated Amazon EC2 instances, controlling both inbound and outbound traffic. 
- **Port 3306:** Required for MySQL/RDS communication.
- **Port 22:** Required for SSH access.
- **Port 80:** Required for HTTP web traffic.

### Private vs. Public Accessibility
A **Private RDS instance** does not have a public IP address and cannot be accessed from the internet. It is only accessible within the VPC or via authorized Security Groups, increasing security for sensitive data.

---

## 2. Implementation Steps (AWS CLI)

### Step 1: Security Group Configuration
```bash
# Get VPC and EC2 Security Group IDs
VPC_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=datacenter-ec2" --query "Reservations[0].Instances[0].VpcId" --output text)
EC2_SG_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=datacenter-ec2" --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" --output text)

# Create RDS Security Group
RDS_SG_ID=$(aws ec2 create-security-group --group-name datacenter-rds-sg --description "SG for RDS" --vpc-id $VPC_ID --query "GroupId" --output text)

# Allow Inbound Traffic
aws ec2 authorize-security-group-ingress --group-id $RDS_SG_ID --protocol tcp --port 3306 --source-group $EC2_SG_ID
aws ec2 authorize-security-group-ingress --group-id $EC2_SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $EC2_SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
```

### Step 2: Create RDS Instance
```bash
aws rds create-db-instance \
    --db-instance-identifier datacenter-rds \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --engine-version 8.4.5 \
    --allocated-storage 5 \
    --storage-type gp2 \
    --master-username datacenter_admin \
    --master-user-password "rdsInstance123" \
    --db-name datacenter_db \
    --vpc-security-group-ids $RDS_SG_ID \
    --no-publicly-accessible \
    --region us-east-1
```

### Step 3: Configure PHP Application
```bash
RDS_ENDPOINT=$(aws rds describe-db-instances --db-instance-identifier datacenter-rds --query "DBInstances[0].Endpoint.Address" --output text)

# Update placeholders in index.php
sed -i "s/<dbhost>/$RDS_ENDPOINT/g" /root/index.php
sed -i "s/<dbuser>/datacenter_admin/g" /root/index.php
sed -i "s/<dbpass>/rdsInstance123/g" /root/index.php
sed -i "s/<dbname>/datacenter_db/g" /root/index.php

# Deploy to EC2
scp -o StrictHostKeyChecking=no /root/index.php root@3.91.213.249:/var/www/html/
```

---

## 3. Troubleshooting & Mistake Analysis

### Mistake 1: SSH Connection Timeout
**Symptom:** `ssh: connect to host 3.91.213.249 port 22: Connection timed out`
- **Cause:** Port 22 was not opened in the EC2 Security Group.
- **Fix:** Ran `aws ec2 authorize-security-group-ingress` for port 22.

### Mistake 2: Permission Denied (publickey)
**Symptom:** `root@3.91.213.249: Permission denied (publickey)`
- **Cause:** The `aws-client` public key was not in the EC2 instance's `/root/.ssh/authorized_keys`.
- **Fix:** Manually added the public key to the EC2 instance using the AWS Console Terminal and corrected folder permissions (700 for `.ssh`, 600 for `authorized_keys`).

### Mistake 3: Placeholder Mismatch in `sed`
**Symptom:** `index.php` still showed `<dbhost>` etc. after running `sed`.
- **Cause:** The `sed` command was looking for `db_host_placeholder` but the file actually contained `<dbhost>`.
- **Fix:** Updated the `sed` command to match the actual placeholders: `sed -i "s/<dbhost>/$RDS_ENDPOINT/g"`.

### Mistake 4: Shell Typos (The "Elephant" in the Room)
**Symptom:** `sssh`, `ssed`, `sscp`, `EEC2_PUBLIC_IP`
- **Cause:** Accidental double characters during command entry.
- **Fix:** Carefully re-entered commands and verified environment variables using `echo`.

---

## Conclusion
The task was successfully completed when the browser displayed **"Connected successfully"** at `http://3.91.213.249/index.php`. This exercise demonstrated the importance of Security Group rules, proper SSH key management, and precise configuration management using shell tools.
