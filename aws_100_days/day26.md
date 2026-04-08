# Day 26: Launching an Nginx Web Server on AWS EC2 (CLI)

## 1. Theoretical Concepts

As a Cloud Engineer, understanding the core components of AWS compute is essential before provisioning resources.

*   **EC2 (Elastic Compute Cloud):** A web service that provides resizable compute capacity in the cloud. It serves as the virtual server for our application.
*   **AMI (Amazon Machine Image):** A template that contains the software configuration (operating system, application server, and applications) required to launch your instance.
*   **Security Groups:** Acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic.
*   **User Data:** A script passed to the instance at launch time to perform automated configuration tasks (bootstrapping), such as installing software.
*   **Key Pairs:** AWS uses public-key cryptography to secure the login information for your instance. You use the private key to SSH into the instance.

---

## 2. Practical Implementation Steps

### Step 1: Preliminary Setup & Variables
To make our commands reusable and cleaner, we set up variables for our VPC and Subnet IDs.

```bash
# Set Variables (Replace with your specific IDs)
export VPC_ID="vpc-0d50c85b41498209f"
export SUBNET_ID="subnet-0fc6633e54c3a230b"
```

### Step 2: Security Group Configuration
We need a security group that allows HTTP traffic (for the web server) and SSH traffic (for management).

```bash
# 1. Create the Security Group
SG_ID=$(aws ec2 create-security-group \
    --group-name nautilus-sg \
    --description "Security group for Nautilus Web Server" \
    --vpc-id $VPC_ID \
    --query 'GroupId' --output text)

# 2. Allow HTTP (Port 80) from anywhere
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# 3. Allow SSH (Port 22) only from your IP (Security Best Practice)
MY_IP=$(curl -s https://checkip.amazonaws.com)
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr ${MY_IP}/32
```

### Step 3: Create SSH Key Pair
**Crucial Step:** You must create the key pair *before* launching the instance.

```bash
aws ec2 create-key-pair --key-name nautilus-key --query 'KeyMaterial' --output text > nautilus-key.pem
chmod 400 nautilus-key.pem
```

### Step 4: Prepare User Data Script
Create the bootstrapping script to install Nginx automatically.

```bash
cat <<EOF > install_nginx.sh
#!/bin/bash
sudo apt update -y
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
EOF
```

### Step 5: Select the Correct AMI
Avoid using Marketplace AMIs to prevent subscription errors. Use the official Canonical owner ID (`099720109477`).

```bash
AMI_ID=$(aws ec2 describe-images \
    --owners 099720109477 \
    --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
    --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
    --output text)
```

### Step 6: Launch the Instance
Combine all components to launch the instance.

```bash
aws ec2 run-instances \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type t2.micro \
    --key-name nautilus-key \
    --security-group-ids $SG_ID \
    --subnet-id $SUBNET_ID \
    --user-data file://install_nginx.sh \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nautilus-ec2}]'
```

### Step 7: Verification
Once running, verify the installation.

```bash
# Get Public IP
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=nautilus-ec2" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text

# SSH into the instance
ssh -i "nautilus-key.pem" ubuntu@<PUBLIC_IP>

# Inside the instance:
curl -I localhost
sudo systemctl status nginx
```

---

## 3. Post-Mortem: Mistakes & Improvements

During this session, we encountered two common "gotchas" in AWS operations.

### Mistake 1: OptInRequired Error
*   **Issue:** Tried to launch an instance using a random or search-result AMI that belonged to the AWS Marketplace.
*   **Error:** `An error occurred (OptInRequired)... In order to use this AWS Marketplace product you need to accept terms...`
*   **Solution:** Filter AMIs specifically by the **Owner ID**. For Ubuntu, always use Canonical's ID `099720109477` to get the official, free-tier eligible image.

### Mistake 2: SSH Permission Denied
*   **Issue:** Launched the instance *without* specifying a Key Pair, or created the Key Pair *after* the instance was already launched.
*   **Error:** `Permission denied (publickey).`
*   **Theory:** The Public Key is injected into the instance **only at boot time**. If you create a key pair later, the running instance knows nothing about it.
*   **Solution:** Terminate the instance. Create the Key Pair first. Launch a new instance explicitly adding the `--key-name` parameter.

### Improvements for Professional Production Envs
1.  **Infrastructure as Code (IaC):** Instead of manual CLI commands, use **Terraform** or **CloudFormation** to manage this state. It prevents ordering mistakes (like missing keys).
2.  **Elastic IP:** Use an Elastic IP if you need a static address, as stopping/starting the instance changes the Public IP.
3.  **Logs:** Always check `/var/log/cloud-init-output.log` if things don't install correctly on startup.
