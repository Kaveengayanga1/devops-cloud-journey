# Day 30: Cost-Optimized VPC Egress using NAT Instances (AWS CLI)

## 📌 The Scenario
The DevOps team is tasked with enabling internet access for an EC2 instance running in a `private subnet`. This instance needs to upload a test file via a cron job to a public S3 bucket (`xfusion-nat-1441`). To minimize costs, a **NAT Instance** is used instead of a managed NAT Gateway.

---

## 🏗️ Technical Components & Architecture Explained

To successfully route traffic from a private instance to the internet, multiple AWS networking components must connect securely.

1. **VPC (Virtual Private Cloud):** 
   Your isolated network boundary in the cloud. Think of it as the outer fence of your data center. The VPC defines the overarching IP address space (e.g., `10.1.0.0/16`).
2. **Subnet (Public & Private):**
   Subnets carve out smaller sections of the VPC. 
   - **Public Subnet:** A subnet is "public" *only* because its Route Table has a direct path to the Internet Gateway. 
   - **Private Subnet:** Has no direct route to the outside world. Resources here cannot be directly reached from the internet, keeping them secure.
3. **IGW (Internet Gateway):**
   The "front door" that connects your VPC to the wider internet. Without an IGW, your VPC is effectively an isolated island.
4. **Route Table (RT):**
   The "GPS map" for your subnet. It tells network traffic where to go. A public route table points `0.0.0.0/0` (any external IP) to the IGW. A private route table points `0.0.0.0/0` to a NAT device.
5. **Security Group (SG):**
   The instance-level firewall. For a NAT instance, the SG must explicitly allow inbound internet-bound traffic (ports 80 and 443) originating *from* the private subnet.
6. **NAT Instance (Network Address Translation):**
   A special EC2 instance living in the Public Subnet. It receives requests from the private instances, replaces their unroutable private IPs with its own Public IP, fetches the result from the internet, and sends it back to the private instance. 
   > *Note:* You must disable standard **Source/Destination Checks** on this instance because it acts as a router handling traffic meant for other machines.

---

## ❌ Mistakes Made & Lessons Learned (Troubleshooting Analysis)

During the deployment, several errors occurred. Understanding these is the hallmark of a Senior DevOps Engineer:

### 1. The CIDR Range Error (`InvalidSubnet.Range`)
* **What happened:** Attempted to create a subnet with `10.0.1.0/24` in a VPC that uses `10.1.0.0/16`.
* **Why it's wrong:** A subnet must mathematically fit entirely within its parent VPC. The `10.0.x.x` space is outside the `10.1.x.x` VPC range.
* **The fix:** Align the first two octets with the VPC: `10.1.x.x`.

### 2. The Subnet Conflict Error (`InvalidSubnet.Conflict`)
* **What happened:** Attempted to create a subnet with `10.1.1.0/24`, but it failed.
* **Why it's wrong:** The existing Private Subnet was already occupying the `10.1.1.0/24` block. Two subnets cannot overlap IP address ranges.
* **The fix:** Select the next available block, such as `10.1.2.0/24`.

### 3. The `eth0` vs `ens5` Interface Issue
* **What happened:** The NAT `iptables` rule was configured as `-o eth0 -j MASQUERADE`, but no internet connection was established.
* **Why it's wrong:** Modern AWS Nitro-based instances (like `t3.micro` on Amazon Linux 2023) use NVMe/ENA drivers which attach interfaces named `ens5` instead of the legacy `eth0`.
* **The fix:** Checked interface names using `ip link show` and updated the `iptables` rule to use `-o ens5`.

### 4. Admin Prohibited (The `FORWARD` Chain Drop)
* **What happened:** `tcpdump` showed `ICMP host unreachable - admin prohibited`.
* **Why it's wrong:** Even though IP Forwarding was enabled on the kernel (`sysctl`), the Amazon Linux 2023 OS-level firewall (`iptables`) defaults the `FORWARD` policy to `REJECT/DROP` for security.
* **The fix:** Explicity allow traffic forwarding: `iptables -P FORWARD ACCEPT` and flush rejecting rules `iptables -F FORWARD`.

---

## ☁️ DevOps Step-by-Step Execution Guide (AWS CLI)

Here are the polished steps a Cloud Engineer follows to perform this setup autonomously:

### Step 1: Network Scaffolding (Subnet & IGW)
```bash
# 1. Identify VPC ID
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=xfusion-priv-vpc" --query 'Vpcs[0].VpcId' --output text)

# 2. Create Public Subnet (Using an available CIDR)
PUB_SUB_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.1.2.0/24 --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-pub-subnet}]' --query 'Subnet.SubnetId' --output text)

# 3. Create & Attach Internet Gateway 
IGW_ID=$(aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=xfusion-igw}]' --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

# 4. Configure Public Route Table
RT_PUB_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=xfusion-pub-rt}]' --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $RT_PUB_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
aws ec2 associate-route-table --route-table-id $RT_PUB_ID --subnet-id $PUB_SUB_ID
```

### Step 2: NAT Security Setup
```bash
NAT_SG_ID=$(aws ec2 create-security-group --group-name xfusion-nat-sg --description "NAT SG" --vpc-id $VPC_ID --query 'GroupId' --output text)

# Allow HTTP/HTTPS exclusively from the Private Subnet
aws ec2 authorize-security-group-ingress --group-id $NAT_SG_ID --protocol tcp --port 443 --cidr 10.1.1.0/24
aws ec2 authorize-security-group-ingress --group-id $NAT_SG_ID --protocol tcp --port 80 --cidr 10.1.1.0/24
# Allow personal SSH access
aws ec2 authorize-security-group-ingress --group-id $NAT_SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
```

### Step 3: Launch NAT Instance
```bash
# Create Keypair
aws ec2 create-key-pair --key-name xfusion-nat-key --query 'KeyMaterial' --output text > xfusion-nat-key.pem && chmod 400 xfusion-nat-key.pem

# Get AL2023 AMI
AMI_ID=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query 'Parameters[0].Value' --output text)

# Launch
NAT_INSTANCE_ID=$(aws ec2 run-instances --image-id $AMI_ID --count 1 --instance-type t3.micro --key-name xfusion-nat-key --security-group-ids $NAT_SG_ID --subnet-id $PUB_SUB_ID --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-nat-instance}]' --query 'Instances[0].InstanceId' --output text)

# CRITICAL: Disable Source/Dest Checks
aws ec2 modify-instance-attribute --instance-id $NAT_INSTANCE_ID --no-source-dest-check
```

### Step 4: Route Private Subnet details to NAT
```bash
PRIV_SUB_ID=$(aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" "Name=tag:Name,Values=xfusion-priv-subnet" --query 'Subnets[0].SubnetId' --output text)
PRIV_RT_ID=$(aws ec2 describe-route-tables --filters "Name=association.subnet-id,Values=$PRIV_SUB_ID" --query 'RouteTables[0].RouteTableId' --output text)

# Forward '0.0.0.0/0' to the NAT Instance
aws ec2 create-route --route-table-id $PRIV_RT_ID --destination-cidr-block 0.0.0.0/0 --instance-id $NAT_INSTANCE_ID
```

### Step 5: OS Networking (Inside NAT Instance)
*SSH into the NAT instance, then configure IPTables natively:*
```bash
sudo dnf install iptables-services -y
sudo systemctl enable iptables --now

# Enable OS-Level IP Forwarding
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Route configuration (Fixing the interface to ens5 and allowing FORWARD chain)
sudo iptables -t nat -F
sudo iptables -F FORWARD
sudo iptables -P FORWARD ACCEPT
sudo iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE
sudo service iptables save
```

### Step 6: Final Verification
Running this command locally confirms that the private instance successfully reached the internet and executed its S3 upload cron job:
```bash
aws s3 ls s3://xfusion-nat-1441/
# Expected Output:
# 2026-03-19 03:16:36   18 xfusion-test.txt
```