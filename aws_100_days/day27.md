# Day 27: Provisioning a Public EC2 Instance with Custom VPC using AWS CLI

In today's task, we will simulate a real-world scenario where a Cloud Engineer needs to provision a secure, public-facing EC2 instance within a custom Virtual Private Cloud (VPC) using only the AWS Command Line Interface (CLI). This approach allows for automation and reproducibility, which are core tenets of DevOps.

## Theoretical Concepts

Before we dive into the commands, it's crucial to understand the components we are building:

1.  **VPC (Virtual Private Cloud):** This is your logically isolated "slice" of the AWS cloud. It defines the network boundaries (CIDR block) for your resources.
2.  **Subnet:** A subdivision of your VPC's IP range. To make a subnet "public," it must have a route to an Internet Gateway.
3.  **Internet Gateway (IGW):** A horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet.
4.  **Route Table:** A set of rules (routes) that determine where network traffic from your subnet or gateway is directed.
5.  **Security Group:** A stateful firewall that controls inbound and outbound traffic at the instance level. We will need to explicitly allow SSH traffic.

## Step-by-Step Implementation

### Prerequisites
*   AWS CLI installed and configured with appropriate permissions.
*   `jq` installed (optional, but helpful for parsing JSON).

### 1. Create a VPC
First, we create the isolated network environment.
```bash
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=devops-pub-vpc}]'
```
*Note the `VpcId` from the output (e.g., `vpc-067aafbf32eeb15b0`), as you will need it for subsequent steps.*

### 2. Create a Subnet
We create a subnet within the VPC.
```bash
aws ec2 create-subnet \
    --vpc-id <Your-VpcId> \
    --cidr-block 10.0.1.0/24 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=devops-pub-subnet}]'
```

### 3. Enable Auto-assign Public IP
For an instance to be reachable from the internet, it needs a public IP address. We configure the subnet to assign one automatically to new instances.
```bash
aws ec2 modify-subnet-attribute \
    --subnet-id <Your-SubnetId> \
    --map-public-ip-on-launch
```

### 4. Create and Attach Request Internet Gateway
The VPC needs a gateway to talk to the outside world.
```bash
# Create IGW
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=devops-pub-igw}]'

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id <Your-IgwId> \
    --vpc-id <Your-VpcId>
```

### 5. Configure Routing
We create a custom route table and add a route to the Internet Gateway (0.0.0.0/0), making the associated subnet "public".
```bash
# Create Route Table
aws ec2 create-route-table \
    --vpc-id <Your-VpcId> \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=devops-pub-rt}]'

# Add Route to Internet (0.0.0.0/0 -> IGW)
aws ec2 create-route \
    --route-table-id <Your-RouteTableId> \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id <Your-IgwId>

# Associate Route Table with Subnet
aws ec2 associate-route-table \
    --subnet-id <Your-SubnetId> \
    --route-table-id <Your-RouteTableId>
```

### 6. Setup Security Group
We create a security group to act as a firewall and allow SSH access (Port 22).
```bash
# Create Security Group
aws ec2 create-security-group \
    --group-name devops-pub-sg \
    --description "Allow SSH" \
    --vpc-id <Your-VpcId>

# Add Ingress Rule for SSH
aws ec2 authorize-security-group-ingress \
    --group-id <Your-SecurityGroupId> \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```
> **Professional Tip:** In a production environment, replace `0.0.0.0/0` with your specific IP address/CIDR range to restrict access only to your trusted network.

### 7. Create Key Pair
Generate a key pair to securely authenticate with the instance.
```bash
aws ec2 create-key-pair \
    --key-name devops-key \
    --query 'KeyMaterial' \
    --output text > devops-key.pem

# Secure the key file (crucial for SSH to work on Linux/Mac)
chmod 400 devops-key.pem
```

### 8. Launch the Instance
A professional engineer often automates the selection of the latest AMI ID rather than hardcoding it.
```bash
# Get the latest Ubuntu 22.04 AMI ID
AMI_ID=$(aws ec2 describe-images \
    --owners 099720109477 \
    --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
    --query 'sort_by(Images, &CreationDate)[-1].ImageId' \
    --output text)

echo "Using AMI: $AMI_ID"

# Launch the instance
aws ec2 run-instances \
    --image-id $AMI_ID \
    --count 1 \
    --instance-type t2.micro \
    --key-name devops-key \
    --security-group-ids <Your-SecurityGroupId> \
    --subnet-id <Your-SubnetId> \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=devops-pub-ec2}]'
```

### 9. Connect to the Instance
Finally, retrieve the public IP and connect to verify the setup.
```bash
# Get Public IP
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=devops-pub-ec2" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text

# SSH into the instance
ssh -i devops-key.pem ubuntu@<Public-IP>
```

If successful, you will see the Ubuntu welcome message!
