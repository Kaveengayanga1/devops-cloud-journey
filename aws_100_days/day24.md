# Day 24: AWS Application Load Balancer (ALB) Setup

## Theory

### What is an Application Load Balancer (ALB)?

An **Application Load Balancer (ALB)** operates at Layer 7 (Application Layer) of the OSI model. It intelligently routes HTTP and HTTPS traffic to different target groups based on content, making it ideal for modern application architectures.

**Key Features:**
- Content-based routing (URL path, host headers, HTTP methods)
- Support for containerized applications
- WebSocket and HTTP/2 support
- Advanced health checks
- Integration with AWS services (ECS, EKS, Lambda)

### Target Groups

A **Target Group** is a logical grouping of targets (EC2 instances, containers, IP addresses, or Lambda functions) that receive traffic from the load balancer.

**Key Concepts:**
- **Health Checks**: Continuously monitor target health
- **Target Types**: Instance, IP, Lambda
- **Routing**: Based on listener rules
- **Sticky Sessions**: Route users to the same target

### Security Groups

**Security Groups** act as virtual firewalls, controlling inbound and outbound traffic at the instance or load balancer level.

**Best Practices:**
- Principle of least privilege
- Separate security groups for ALB and EC2 instances
- Use security group references instead of IP ranges

### Listeners

**Listeners** check for connection requests from clients using the protocol and port you configure, then forward requests to target groups based on rules.

**Components:**
- **Protocol**: HTTP, HTTPS
- **Port**: 80, 443, custom
- **Default Action**: Forward, redirect, fixed response
- **Rules**: Path-based, host-based routing

---

## Practical Implementation

### Objective

Set up an Application Load Balancer named `xfusion-alb` that routes traffic to an EC2 instance running Nginx:

- Create a target group named `xfusion-tg`
- Create a security group named `xfusion-sg` (port 80 open to public)
- Attach security group to ALB
- Route traffic from ALB port 80 to EC2 instance port 80
- Configure EC2 security group to accept traffic only from ALB

---

## Step-by-Step Implementation

### Step 1: Find Your VPC ID

Before creating any resources, identify your VPC:

```bash
aws ec2 describe-vpcs --query "Vpcs[*].{VpcId:VpcId,Name:Tags[?Key=='Name']|[0].Value}" --output table
```

**Output:**
```
-----------------------------------
|          DescribeVpcs           |
+------+--------------------------+
| Name |          VpcId           |
+------+--------------------------+
|  None|  vpc-07cbd64b12f1f66d3   |
+------+--------------------------+
```

**Note:** Save the VPC ID (`vpc-07cbd64b12f1f66d3`)

---

### Step 2: Find Your Subnet IDs

ALBs require at least 2 subnets in different Availability Zones:

```bash
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-07cbd64b12f1f66d3" --query "Subnets[*].{SubnetId:SubnetId,AZ:AvailabilityZone}" --output table
```

**For subnet IDs only:**
```bash
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-07cbd64b12f1f66d3" --query "Subnets[*].SubnetId" --output text
```

**Output:**
```
subnet-0e5cfccd083fee319  subnet-0025dfb8e18859a67  subnet-0b6b95086554d75d6
subnet-044826bb7df6a0f51  subnet-071dfa1a672d84fef  subnet-095a9008ebdb40664
```

**Note:** Select at least 2 subnets from different AZs

---

### Step 3: Find Your EC2 Instance ID

Locate the instance you want to place behind the ALB:

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=xfusion-ec2" --query "Reservations[*].Instances[*].InstanceId" --output text
```

**Output:**
```
i-0d3e389a6e72e80e6
```

---

### Step 4: Create Security Group for ALB

Create a security group that allows HTTP traffic from the internet:

```bash
aws ec2 create-security-group \
    --group-name xfusion-sg \
    --description "Security group for xfusion-alb" \
    --vpc-id vpc-07cbd64b12f1f66d3
```

**Output:**
```json
{
    "GroupId": "sg-00be083a772d48ba7",
    "SecurityGroupArn": "arn:aws:ec2:us-east-1:854993966332:security-group/sg-00be083a772d48ba7"
}
```

**Note:** Save the GroupId (`sg-00be083a772d48ba7`)

---

### Step 5: Authorize Inbound Traffic on Port 80

Allow HTTP traffic from anywhere (0.0.0.0/0):

```bash
aws ec2 authorize-security-group-ingress \
    --group-name xfusion-sg \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

**Output:**
```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0e40f9707e18dbb8b",
            "GroupId": "sg-00be083a772d48ba7",
            "GroupOwnerId": "854993966332",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

---

### Step 6: Create Target Group

Create a target group to register your EC2 instances:

```bash
aws elbv2 create-target-group \
    --name xfusion-tg \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-07cbd64b12f1f66d3 \
    --target-type instance
```

**Output:**
```json
{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:854993966332:targetgroup/xfusion-tg/b60542d5ccc9d60d",
            "TargetGroupName": "xfusion-tg",
            "Protocol": "HTTP",
            "Port": 80,
            "VpcId": "vpc-07cbd64b12f1f66d3",
            "HealthCheckProtocol": "HTTP",
            "HealthCheckPort": "traffic-port",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 30,
            "HealthCheckTimeoutSeconds": 5,
            "HealthyThresholdCount": 5,
            "UnhealthyThresholdCount": 2,
            "HealthCheckPath": "/",
            "TargetType": "instance"
        }
    ]
}
```

**Note:** Save the TargetGroupArn

---

### Step 7: Register EC2 Instance to Target Group

Add your EC2 instance to the target group:

```bash
aws elbv2 register-targets \
    --target-group-arn arn:aws:elasticloadbalancing:us-east-1:854993966332:targetgroup/xfusion-tg/b60542d5ccc9d60d \
    --targets Id=i-0d3e389a6e72e80e6
```

**Note:** No output means success

---

### Step 8: Create Application Load Balancer

Create the ALB with at least 2 subnets in different AZs:

```bash
aws elbv2 create-load-balancer \
    --name xfusion-alb \
    --subnets subnet-0e5cfccd083fee319 subnet-0025dfb8e18859a67 \
    --security-groups sg-00be083a772d48ba7 \
    --scheme internet-facing \
    --type application
```

**Output:**
```json
{
    "LoadBalancers": [
        {
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:854993966332:loadbalancer/app/xfusion-alb/97640f90bd27b1f7",
            "DNSName": "xfusion-alb-166442030.us-east-1.elb.amazonaws.com",
            "CanonicalHostedZoneId": "Z35SXDOTRQ7X7K",
            "CreatedTime": "2026-03-05T05:16:28.544Z",
            "LoadBalancerName": "xfusion-alb",
            "Scheme": "internet-facing",
            "VpcId": "vpc-07cbd64b12f1f66d3",
            "State": {
                "Code": "provisioning"
            },
            "Type": "application",
            "AvailabilityZones": [
                {
                    "ZoneName": "us-east-1c",
                    "SubnetId": "subnet-0e5cfccd083fee319"
                },
                {
                    "ZoneName": "us-east-1e",
                    "SubnetId": "subnet-0025dfb8e18859a67"
                }
            ]
        }
    ]
}
```

**Note:** Save the LoadBalancerArn and DNSName

---

### Step 9: Create Listener

Configure the ALB to listen on port 80 and forward to the target group:

```bash
aws elbv2 create-listener \
    --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:854993966332:loadbalancer/app/xfusion-alb/97640f90bd27b1f7 \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:854993966332:targetgroup/xfusion-tg/b60542d5ccc9d60d
```

**Output:**
```json
{
    "Listeners": [
        {
            "ListenerArn": "arn:aws:elasticloadbalancing:us-east-1:854993966332:listener/app/xfusion-alb/97640f90bd27b1f7/86243590ce835b11",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:854993966332:loadbalancer/app/xfusion-alb/97640f90bd27b1f7",
            "Port": 80,
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "Type": "forward",
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:854993966332:targetgroup/xfusion-tg/b60542d5ccc9d60d"
                }
            ]
        }
    ]
}
```

---

### Step 10: Update EC2 Instance Security Group

First, find the security group attached to your EC2 instance:

```bash
aws ec2 describe-instances --instance-ids i-0d3e389a6e72e80e6 --query "Reservations[*].Instances[*].SecurityGroups[0].GroupId" --output text
```

**Output:**
```
sg-0cb6f36abd4ddd107
```

Now, allow traffic from the ALB security group only:

```bash
aws ec2 authorize-security-group-ingress \
    --group-id sg-0cb6f36abd4ddd107 \
    --protocol tcp \
    --port 80 \
    --source-group sg-00be083a772d48ba7
```

**Output:**
```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-02452ed63e4561d22",
            "GroupId": "sg-0cb6f36abd4ddd107",
            "GroupOwnerId": "854993966332",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "ReferencedGroupInfo": {
                "GroupId": "sg-00be083a772d48ba7",
                "UserId": "854993966332"
            }
        }
    ]
}
```

**Security Best Practice:** This ensures traffic can only reach the EC2 instance through the ALB, not directly.

---

### Step 11: Get ALB DNS Name and Test

Retrieve the DNS name of your load balancer:

```bash
aws elbv2 describe-load-balancers --names xfusion-alb --query "LoadBalancers[*].DNSName" --output text
```

**Output:**
```
xfusion-alb-166442030.us-east-1.elb.amazonaws.com
```

**Testing:**
Open your browser and navigate to:
```
http://xfusion-alb-166442030.us-east-1.elb.amazonaws.com
```

You should see the Nginx welcome page!

---

## Architecture Overview

```
Internet
   ↓
   ↓ (HTTP:80)
   ↓
[Application Load Balancer]
(xfusion-alb)
Security Group: xfusion-sg (0.0.0.0/0:80)
   ↓
   ↓ (HTTP:80)
   ↓
[Target Group: xfusion-tg]
   ↓
   ↓
[EC2 Instance: xfusion-ec2]
Security Group: (ALB-SG-Only:80)
   ↓
[Nginx Server]
```

---

## Key Takeaways

1. **ALB requires at least 2 subnets** in different Availability Zones for high availability
2. **Security Groups act as firewalls** - configure them following the principle of least privilege
3. **Target Groups perform health checks** - ensure your application responds to health check requests
4. **Listeners define routing rules** - you can have multiple listeners with different rules
5. **Security best practice**: Allow traffic to EC2 instances only from the ALB security group
6. **DNS propagation**: The ALB DNS name may take a few minutes to become fully active

---

## Common Issues and Troubleshooting

### Issue: Target shows as "unhealthy"
- Check security group allows traffic from ALB
- Verify application is running on the specified port
- Check health check path returns HTTP 200

### Issue: Cannot access ALB DNS name
- Wait 2-3 minutes for ALB to become "active"
- Verify security group allows inbound traffic on port 80
- Check subnets are in different AZs

### Issue: 502 Bad Gateway
- Target instance is not responding
- Application is not running
- Security group blocking traffic

---

## Next Steps

- Add HTTPS listener with SSL/TLS certificate
- Configure path-based routing
- Set up CloudWatch alarms for monitoring
- Implement auto-scaling with the target group
- Add WAF (Web Application Firewall) for security

---

**Day 24 Complete!** ✅

You've successfully set up an Application Load Balancer with AWS CLI!
