# Day 36: AWS CLI - Deploying Nginx with Application Load Balancer (ALB)

## Task Overview
The Nautilus Development Team required a web server setup on an EC2 instance, placed behind an Application Load Balancer (ALB) for high availability and traffic management.

### Requirements:
1.  **Security Group**: Create `xfusion-sg` allowing Port 80 from the default security group (ALB).
2.  **EC2 Instance**: Launch an Ubuntu instance named `xfusion-ec2` with a User Data script to install and start Nginx.
3.  **Target Group**: Create `xfusion-tg` and register the instance.
4.  **Application Load Balancer**: Create `xfusion-alb` using the default security group and route traffic to the target group.
5.  **Verification**: Access the Nginx server via the ALB DNS name.

---

## 🛠 Project Credentials & Environment Details (Historical)
> **Note**: These credentials and resource IDs are from a past session and no longer exist.
- **VPC ID**: `vpc-02cabe789239af6f2`
- **Default SG ID**: `sg-0c4831c8dcb5a0b20`
- **Xfusion SG ID**: `sg-0a6ef0e9fec8f53f1`
- **Instance ID**: `i-011c298fb59668d71` (Final Healthy Instance)
- **Target Group ARN**: `arn:aws:elasticloadbalancing:us-east-1:605399109188:targetgroup/xfusion-tg/6a881ef42c5a6128`
- **ALB ARN**: `arn:aws:elasticloadbalancing:us-east-1:605399109188:loadbalancer/app/xfusion-alb/34e47db454b7f923`
- **ALB DNS**: `xfusion-alb-2006808558.us-east-1.elb.amazonaws.com`
- **Key Pair**: `xfusion-kp` (Private key: `xfusion-kp.pem`)
- **Public IP**: `98.93.71.238`

---

## 🚀 Step-by-Step Execution

### 1. Networking Setup
First, identify the default VPC and Security Group:
```bash
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)
DEFAULT_SG_ID=$(aws ec2 describe-security-groups --filters "Name=group-name,Values=default" --query "SecurityGroups[0].GroupId" --output text)
```

Create the instance Security Group and allow Port 80 from the ALB's Security Group:
```bash
XFUSION_SG_ID=$(aws ec2 create-security-group --group-name xfusion-sg --description "SG for EC2 instance" --vpc-id $VPC_ID --query "GroupId" --output text)

aws ec2 authorize-security-group-ingress --group-id $XFUSION_SG_ID --protocol tcp --port 80 --source-group $DEFAULT_SG_ID
```

### 2. EC2 Provisioning
Create the User Data script:
```bash
cat <<EOF > userdata.sh
#!/bin/bash
sudo apt update -y
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
EOF
```

Create Key Pair and Launch Instance:
```bash
aws ec2 create-key-pair --key-name xfusion-kp --query 'KeyMaterial' --output text > xfusion-kp.pem
chmod 400 xfusion-kp.pem

INSTANCE_ID=$(aws ec2 run-instances \
    --image-id ami-04680790a315cd58d \
    --count 1 \
    --instance-type t2.micro \
    --key-name xfusion-kp \
    --security-group-ids $XFUSION_SG_ID \
    --user-data file://userdata.sh \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
    --query "Instances[0].InstanceId" --output text)
```

### 3. Load Balancer Configuration
Create Target Group and Register Instance:
```bash
TG_ARN=$(aws elbv2 create-target-group --name xfusion-tg --protocol HTTP --port 80 --vpc-id $VPC_ID --target-type instance --query "TargetGroups[0].TargetGroupArn" --output text)

aws elbv2 register-targets --target-group-arn $TG_ARN --targets Id=$INSTANCE_ID
```

Create the ALB (requires at least two subnets):
```bash
SUBNET_1=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query "Subnets[0].SubnetId" --output text)
SUBNET_2=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query "Subnets[1].SubnetId" --output text)

ALB_ARN=$(aws elbv2 create-load-balancer --name xfusion-alb --subnets $SUBNET_1 $SUBNET_2 --security-groups $DEFAULT_SG_ID --query "LoadBalancers[0].LoadBalancerArn" --output text)
```

Create Listener:
```bash
aws elbv2 create-listener --load-balancer-arn $ALB_ARN --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=$TG_ARN
```

Final Security Group update for Public Access:
```bash
aws ec2 authorize-security-group-ingress --group-id $DEFAULT_SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0
```

---

## ⚠️ Mistakes Made & Troubleshooting

### 1. Invalid Key Pair Name
- **Mistake**: Using the filename `xfusion-kp.pem` in the `--key-name` parameter.
- **Why**: AWS expects the logical name defined in EC2, not the local file extension.
- **Fix**: Changed `--key-name xfusion-kp.pem` to `--key-name xfusion-kp`.

### 2. SSH Connection Timeout
- **Mistake**: Attempting to SSH into the instance without opening Port 22.
- **Why**: Security Groups act as a virtual firewall; if Port 22 isn't explicitly opened, traffic is dropped.
- **Fix**: Added an ingress rule to `xfusion-sg` for Port 22 from `0.0.0.0/0`.

### 3. SSH Permission Denied
- **Mistake**: Using `ec2-user` for an Ubuntu AMI or incorrect key file permissions.
- **Why**: Ubuntu default user is `ubuntu`. Also, SSH requires private keys to have restrictive permissions (`400`).
- **Fix**: Used `ssh -i xfusion-kp.pem ubuntu@<IP>` and ensured `chmod 400` was applied.

### 4. Nginx "Unable to locate package"
- **Mistake**: Running `apt install nginx` without updating the repository index.
- **Why**: Ubuntu needs to sync its package list with the server before it knows where to find Nginx.
- **Fix**: Ran `sudo apt update -y` before `sudo apt install nginx -y`.

### 5. Target Health "unhealthy"
- **Mistake**: Targets remained unhealthy for several minutes.
- **Why**: Health checks take time (Initial -> Healthy). Also, some instances were launched with Amazon Linux by mistake, missing Nginx.
- **Fix**: Verified Nginx service status with `sudo systemctl status nginx` and waited for the ALB to complete health checks.

---

## ✅ Final Result Verification
```bash
ALB_DNS=$(aws elbv2 describe-load-balancers --load-balancer-arns $ALB_ARN --query "LoadBalancers[0].DNSName" --output text)
curl -I http://$ALB_DNS
```
**Output**: `HTTP/1.1 200 OK` (Indicates the Load Balancer successfully routed traffic to the healthy Nginx instance).
