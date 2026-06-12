# Day 40: EC2 Internet Connectivity Validation with Internet Gateway and Route Table

- **Date:** 2026-06-11
- **Topic:** Validating public access to an EC2-hosted Nginx application
- **Project:** AWS 100 Days DevOps Practice
- **Region:** us-east-1

## Server Credentials and Environment Details

| Item                 | Value                                   |
| -------------------- | --------------------------------------- |
| Instance name        | `devops-ec2`                            |
| Historical public IP | `32.194.175.93`                         |
| Username             | `ubuntu`                                |
| Password             | Not used; historical/example value only |
| SSH port             | `22`                                    |
| Application port     | `80`                                    |
| VPC ID               | `vpc-08f5bc4e248519a38`                 |
| Internet Gateway ID  | `igw-02775c8d9f7cae969`                 |

## Task Summary

The objective of this task was to make an EC2 instance running Nginx reachable from the public internet without logging into the instance. The main troubleshooting steps were to attach an Internet Gateway to the VPC, correct the route table, and verify the application from the outside using AWS CLI and `curl`.

## Task Output

### 1. Attach the Internet Gateway

```bash
aws ec2 attach-internet-gateway --vpc-id "vpc-08f5bc4e248519a38" --internet-gateway-id "igw-02775c8d9f7cae969" --region us-east-1
```

### 2. Retrieve the public IP of the running instance

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=devops-ec2" "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text
```

Output:

```text
32.194.175.93
```

### 3. Verify external access to Nginx

```bash
curl -I http://32.194.175.93
```

Output:

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Thu, 11 Jun 2026 01:57:38 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Fri, 13 Oct 2023 13:33:26 GMT
Connection: keep-alive
ETag: "65294726-267"
Accept-Ranges: bytes
```

The `200 OK` response confirmed that the web server was reachable on port 80 from outside the instance.

## Theory

### How an EC2 instance reaches the internet

An EC2 instance does not become publicly reachable by itself. Internet connectivity depends on a chain of networking components working together in the correct order.

1. **VPC**: The Virtual Private Cloud defines the isolated network boundary for AWS resources.
2. **CIDR block**: The VPC CIDR defines the private IP range available inside the network.
3. **Subnet**: A subnet is a smaller segment of the VPC. A public subnet is one whose route table points to an Internet Gateway.
4. **Internet Gateway (IGW)**: The IGW is the edge connection between the VPC and the internet.
5. **Route table**: The route table decides where traffic is sent. A public subnet needs a route such as `0.0.0.0/0 -> IGW`.
6. **Public IP / Elastic IP**: The instance needs a public address so internet users can reach it.
7. **Security Group**: The instance firewall must allow inbound HTTP traffic on port 80 and outbound return traffic.
8. **Network ACL**: The subnet-level network ACL must also permit the traffic path.

### Required order of operation

The connectivity path works in this order:

EC2 instance with public IP -> Public subnet -> Route table `0.0.0.0/0` -> Internet Gateway -> Internet

### Why `curl -I` was useful

The `-I` option requests only the HTTP headers. This is a fast way to validate that the application responds on port 80 without downloading the full page.

### Meaning of a blackhole route

A route table entry becomes **blackhole** when the target resource no longer exists, is detached, or is otherwise invalid. In practice, this means the route exists on paper, but traffic cannot be forwarded through it.

## Mistakes and Fixes

### Mistake 1: The Internet Gateway was not attached to the VPC

**Why it happened:** The VPC had no active internet edge connection, so the subnet could not send traffic to the public internet.

**How it was fixed:** The Internet Gateway `igw-02775c8d9f7cae969` was attached to `vpc-08f5bc4e248519a38` using the AWS CLI.

### Mistake 2: The route table did not have a valid public route

**Why it happened:** The subnet was missing a working `0.0.0.0/0` route pointing to the active Internet Gateway. This caused the route status to appear as blackhole.

**How it was fixed:** A new route was created and associated with the correct, attached Internet Gateway. After this, the route status changed to active.

### Mistake 3: The application had to be validated without instance login

**Why it happened:** The goal was to confirm public accessibility externally rather than using SSH access to inspect the service locally.

**How it was fixed:** The instance was tested from outside with AWS CLI and `curl -I http://32.194.175.93`, which returned `HTTP/1.1 200 OK`.

## Result

The EC2-hosted Nginx application became publicly reachable on port 80 after the Internet Gateway and route table were corrected. External validation through `curl` confirmed successful access.
