# Day 38: Deploying Containerized Application on AWS ECS with Fargate

**Date:** May 29, 2026

**Topic:** Amazon Elastic Container Service (ECS) Deployment with Fargate Launch Type

**Project:** Xfusion - Cloud-Native Application Deployment

**Region:** us-east-1

---

## Server Credentials (Historical/Example Values)

| Credential          | Value                                        | Status           |
| ------------------- | -------------------------------------------- | ---------------- |
| AWS Account ID      | 258124001149                                 | Historical       |
| IAM Username        | kk_labs_user_894508                          | Historical       |
| ECR Repository Name | xfusion-ecr                                  | Active (Example) |
| ECS Cluster Name    | xfusion-cluster                              | Active (Example) |
| Security Group ID   | sg-01d65516525a997aa                         | Historical       |
| IAM Execution Role  | xfusion-execution-role                       | Historical       |
| Docker Registry     | 258124001149.dkr.ecr.us-east-1.amazonaws.com | Historical       |

---

## Task Overview

The objective was to deploy a containerized Nginx application using AWS services. The complete workflow involved:

1. Creating a private Amazon ECR repository
2. Building a Docker image from a Dockerfile
3. Pushing the image to ECR
4. Creating an ECS cluster with Fargate launch type
5. Registering an ECS task definition
6. Deploying the application as an ECS service
7. Verifying accessibility and troubleshooting network connectivity

---

## Task Output

### Step 1: Create Private ECR Repository

```bash
aws ecr create-repository \
    --repository-name xfusion-ecr \
    --region us-east-1
```

**Output:**

```json
{
  "repository": {
    "repositoryArn": "arn:aws:ecr:us-east-1:258124001149:repository/xfusion-ecr",
    "registryId": "258124001149",
    "repositoryName": "xfusion-ecr",
    "repositoryUri": "258124001149.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr",
    "createdAt": 1780028576.604,
    "imageTagMutability": "MUTABLE",
    "imageScanningConfiguration": {
      "scanOnPush": false
    },
    "encryptionConfiguration": {
      "encryptionType": "AES256"
    }
  }
}
```

### Step 2: Authenticate Docker with ECR and Build Image

```bash
cd /root/pyapp

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com

docker build -t xfusion-ecr .

docker tag xfusion-ecr:latest $YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest

docker push $YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest
```

**Build Output:**

```
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM nginx:alpine
alpine: Pulling from library/nginx
[Multiple layers pulled successfully]
Digest: sha256:8b1e78743a03dbb2c95171cc58639fef29abc8816598e27fb910ed2e621e589a
Status: Downloaded newer image for nginx:alpine

Step 2/4 : COPY index.html /usr/share/nginx/html/index.html
 ---> ca6bb732423c

Step 3/4 : EXPOSE 80
 ---> 39b7ce437943

Step 4/4 : CMD ["nginx", "-g", "daemon off;"]
 ---> 3137ad3848a6

Successfully built 3137ad3848a6
Successfully tagged xfusion-ecr:latest
```

**Push Output:**

```
The push refers to repository [258124001149.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr]
9130c94e19f0: Pushed
5519239b187c: Pushed
[Multiple layers pushed]
latest: digest: sha256:c456613b906569c51553189bcf05fbfb29c03903e219563ae43c6e56dd4de8c4 size: 2196
```

### Step 3: Create ECS Cluster

```bash
aws ecs create-cluster \
    --cluster-name xfusion-cluster \
    --region us-east-1
```

**Output:**

```json
{
  "cluster": {
    "clusterArn": "arn:aws:ecs:us-east-1:258124001149:cluster/xfusion-cluster",
    "clusterName": "xfusion-cluster",
    "status": "ACTIVE",
    "registeredContainerInstancesCount": 0,
    "runningTasksCount": 0,
    "activeServicesCount": 0,
    "settings": [
      {
        "name": "containerInsights",
        "value": "disabled"
      }
    ]
  }
}
```

### Step 4: Register Task Definition

**Task Definition JSON:**

```json
{
  "family": "xfusion-taskdefinition",
  "networkMode": "awsvpc",
  "executionRoleArn": "arn:aws:iam::258124001149:role/xfusion-execution-role",
  "containerDefinitions": [
    {
      "name": "pyapp-container",
      "image": "258124001149.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 80
        }
      ]
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

**Registration Output:**

```json
{
  "taskDefinition": {
    "taskDefinitionArn": "arn:aws:ecs:us-east-1:258124001149:task-definition/xfusion-taskdefinition:2",
    "family": "xfusion-taskdefinition",
    "revision": 2,
    "executionRoleArn": "arn:aws:iam::258124001149:role/xfusion-execution-role",
    "containerDefinitions": [
      {
        "name": "pyapp-container",
        "image": "258124001149.dkr.ecr.us-east-1.amazonaws.com/xfusion-ecr:latest",
        "portMappings": [
          {
            "containerPort": 80,
            "hostPort": 80,
            "protocol": "tcp"
          }
        ],
        "essential": true
      }
    ],
    "status": "ACTIVE",
    "requiresCompatibilities": ["FARGATE"],
    "cpu": "256",
    "memory": "512"
  }
}
```

### Step 5: Retrieve VPC Network Configuration

```bash
SUBNETS=$(aws ec2 describe-subnets --region us-east-1 --query "Subnets[*].SubnetId" --output text | tr '\t' ',')

SG=$(aws ec2 describe-security-groups --filters "Name=group-name,Values=default" --region us-east-1 --query "SecurityGroups[0].GroupId" --output text)

echo "Subnets: $SUBNETS"
echo "Security Group: $SG"
```

**Output:**

```
Subnets: subnet-082096681de3c7c13,subnet-042ac4e3104939dce,subnet-01b6d4b036fb0033e,subnet-019f72d7a6c375b02,subnet-0dd0647ec1c65c9e7,subnet-0443e01ef51e9cd8b
Security Group: sg-01d65516525a997aa
```

### Step 6: Create ECS Service

```bash
aws ecs create-service \
    --cluster xfusion-cluster \
    --service-name xfusion-service \
    --task-definition xfusion-taskdefinition \
    --desired-count 1 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[${SUBNETS}],securityGroups=[sg-01d65516525a997aa],assignPublicIp=ENABLED}" \
    --region us-east-1
```

**Output:**

```json
{
  "service": {
    "serviceArn": "arn:aws:ecs:us-east-1:258124001149:service/xfusion-cluster/xfusion-service",
    "serviceName": "xfusion-service",
    "status": "ACTIVE",
    "desiredCount": 1,
    "runningCount": 0,
    "launchType": "FARGATE",
    "taskDefinition": "arn:aws:ecs:us-east-1:258124001149:task-definition/xfusion-taskdefinition:2",
    "networkConfiguration": {
      "awsvpcConfiguration": {
        "subnets": [
          "subnet-082096681de3c7c13",
          "subnet-042ac4e3104939dce",
          "subnet-01b6d4b036fb0033e",
          "subnet-019f72d7a6c375b02",
          "subnet-0dd0647ec1c65c9e7",
          "subnet-0443e01ef51e9cd8b"
        ],
        "securityGroups": ["sg-01d65516525a997aa"],
        "assignPublicIp": "ENABLED"
      }
    }
  }
}
```

### Step 7: Retrieve Public IP Address

```bash
TASK_ARN=$(aws ecs list-tasks --cluster xfusion-cluster --region us-east-1 --query "taskArns[0]" --output text)

ENI_ID=$(aws ecs describe-tasks --cluster xfusion-cluster --tasks $TASK_ARN --region us-east-1 --query "tasks[0].attachments[0].details[?name=='networkInterfaceId'].value" --output text)

PUBLIC_IP=$(aws ec2 describe-network-interfaces --network-interface-ids $ENI_ID --region us-east-1 --query "NetworkInterfaces[0].Association.PublicIp" --output text)

echo "Your Fargate Task Public IP is: $PUBLIC_IP"
```

**Output:**

```
Your Fargate Task Public IP is: 54.91.48.249
```

### Step 8: Authorize Security Group Ingress

```bash
aws ec2 authorize-security-group-ingress \
    --group-id sg-01d65516525a997aa \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0 \
    --region us-east-1
```

**Output:**

```json
{
  "Return": true,
  "SecurityGroupRules": [
    {
      "SecurityGroupRuleId": "sgr-063958d9be7d8ce38",
      "GroupId": "sg-01d65516525a997aa",
      "IsEgress": false,
      "IpProtocol": "tcp",
      "FromPort": 80,
      "ToPort": 80,
      "CidrIpv4": "0.0.0.0/0"
    }
  ]
}
```

### Step 9: Verify Application Accessibility

```bash
curl http://$PUBLIC_IP
```

**Output:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Welcome to Nautilus</title>
  </head>
  <body>
    <h1>Welcome to KKE AWS cloud labs!</h1>
  </body>
</html>
```

---

## Theories and Concepts

### 1. Containerization

Containerization is a lightweight virtualization technology that packages an application and all its dependencies (libraries, configuration files, runtime) into a single, portable unit called a container. Benefits include:

- **Consistency:** Ensures the application runs identically across different environments (development, testing, production)
- **Isolation:** Containers are isolated from each other and from the host system
- **Efficiency:** Containers share the host OS kernel, making them more efficient than virtual machines
- **Portability:** Containers can run on any system that has a container runtime installed (Docker, Containerd)

### 2. Docker and Images

A Docker image is a lightweight, standalone, executable package that contains:

- Application code
- Runtime environment
- System libraries and tools
- Environment variables and configuration

Docker images are built from a Dockerfile, which is a text file containing instructions to build the image. Each instruction creates a layer in the image, and these layers are cached for efficiency.

### 3. Amazon Elastic Container Registry (ECR)

Amazon ECR is a fully managed Docker container registry provided by AWS. It serves as:

- **Private Registry:** Stores Docker images in a private repository accessible only by authorized users
- **Integration:** Seamlessly integrates with other AWS services like ECS, Lambda, and CodePipeline
- **Security:** Supports image scanning for vulnerabilities and encryption at rest
- **Lifecycle Policies:** Allows automatic cleanup of old or unused images

### 4. Amazon Elastic Container Service (ECS)

Amazon ECS is a highly scalable container orchestration service that manages Docker containers on AWS infrastructure. It provides:

- **Task Management:** Defines and executes containerized applications
- **Service Management:** Maintains a desired number of running tasks
- **Cluster Management:** Groups EC2 instances or Fargate resources for running tasks
- **Load Balancing:** Integrates with Elastic Load Balancers for traffic distribution
- **Service Discovery:** Automatic DNS-based service discovery within the cluster

### 5. AWS Fargate

AWS Fargate is a serverless compute engine for containers that eliminates the need to manage underlying infrastructure:

- **No EC2 Management:** No need to provision, configure, or manage EC2 instances
- **Pay Per Use:** Billing is based on the actual resources (CPU/Memory) consumed
- **Automatic Scaling:** Scales up or down based on application demands
- **Security:** Provides task-level isolation and AWS managed security groups

**Fargate vs EC2 Launch Type:**

| Aspect                    | Fargate                  | EC2                           |
| ------------------------- | ------------------------ | ----------------------------- |
| Infrastructure Management | AWS managed (serverless) | You manage                    |
| Scaling                   | Automatic                | Manual or Auto Scaling Groups |
| Cost Model                | Pay per task             | Pay per instance              |
| Complexity                | Lower                    | Higher                        |

### 6. VPC and Networking

When using Fargate with the `awsvpc` network mode:

- Each task receives its own Elastic Network Interface (ENI)
- Tasks can have public or private IP addresses
- Security groups control inbound and outbound traffic
- Subnets determine the availability zones where tasks run

### 7. IAM Execution Role

An IAM Execution Role is required for Fargate tasks to:

- Pull Docker images from private ECR repositories
- Write logs to CloudWatch
- Access other AWS services required by the application
- The role must have a trust relationship with the ECS service principal (`ecs-tasks.amazonaws.com`)

### 8. Docker Port Mapping

In the container definition, port mapping specifies:

- **containerPort:** The port the application listens on inside the container (80 for Nginx)
- **hostPort:** The port exposed on the host (80 for direct access)
- In Fargate with `awsvpc` mode, containerPort and hostPort are typically the same

---

## Mistakes and Fixes

### Mistake 1: Using Angle Brackets Instead of Environment Variable

**Description:** The command used angle brackets `<YOUR_AWS_ACCOUNT_ID>` literally instead of substituting the actual AWS Account ID.

**Error Message:**

```
bash: YOUR_AWS_ACCOUNT_ID: No such file or directory
Exception ignored in: <_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
BrokenPipeError: [Errno 32] Broken pipe
```

**Why It Happened:**

- Shell interprets angle brackets as input/output redirection operators
- The literal string `<YOUR_AWS_ACCOUNT_ID>` was treated as a file path for input redirection
- This is a common mistake when copying commands from documentation without proper variable substitution

**How It Was Fixed:**

1. Stored the AWS Account ID in an environment variable:

   ```bash
   YOUR_AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
   ```

2. Used the variable in subsequent commands with proper shell expansion:

   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $YOUR_AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
   ```

3. Verified the variable contained the correct value:
   ```bash
   echo $YOUR_AWS_ACCOUNT_ID
   ```

**Lesson Learned:** Always use proper shell variable expansion (`$VARIABLE` or `${VARIABLE}`) when substituting values in commands. Avoid copying documentation literally without understanding the placeholders.

---

### Mistake 2: Missing Execution Role ARN in Task Definition

**Description:** The task definition was created without specifying an `executionRoleArn` parameter, which is mandatory for Fargate tasks when pulling images from private ECR repositories.

**Error Message:**

```
An error occurred (ClientException) when calling the RegisterTaskDefinition operation: Fargate requires task definition to have execution role ARN to support ECR images.
```

**Why It Happened:**

- Fargate requires explicit permissions to pull Docker images from private repositories
- The execution role grants ECS service permissions to authenticate with ECR
- The task definition JSON template provided did not initially include this required field
- The AWS documentation requirement was overlooked during initial task definition creation

**How It Was Fixed:**

1. Created an IAM role with appropriate trust policy for ECS tasks:

   ```bash
   cat <<EOF > trust-policy.json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "ecs-tasks.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   EOF

   aws iam create-role \
       --role-name xfusion-execution-role \
       --assume-role-policy-document file://trust-policy.json
   ```

2. Attached the standard AWS managed policy for ECS task execution:

   ```bash
   aws iam attach-role-policy \
       --role-name xfusion-execution-role \
       --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
   ```

3. Updated the task definition to include the execution role ARN:

   ```json
   {
     "family": "xfusion-taskdefinition",
     "executionRoleArn": "arn:aws:iam::258124001149:role/xfusion-execution-role",
     ...
   }
   ```

4. Re-registered the task definition with the corrected JSON

**Lesson Learned:** For Fargate tasks pulling from private repositories, always include an `executionRoleArn` in the task definition. This role enables the ECS agent to authenticate with ECR and other AWS services on behalf of the task.

---

### Mistake 3: Invalid Subnet Filter Parameter

**Description:** Used incorrect filter name `default-for-zone` when querying for default VPC subnets, resulting in an invalid parameter error.

**Error Message:**

```
An error occurred (InvalidParameterValue) when calling the DescribeSubnets operation: The filter 'default-for-zone' is invalid
```

**Why It Happened:**

- AWS changed the valid filter names for EC2 subnet queries over time
- The filter name `default-for-zone` is not recognized by the current AWS CLI version
- The actual valid filter (if used) would be `default-for-az` (availability zone)
- Documentation on various sources may reference outdated filter names

**How It Was Fixed:**

1. Replaced the invalid filter with a more reliable approach that fetches all available subnets:

   ```bash
   SUBNETS=$(aws ec2 describe-subnets --region us-east-1 --query "Subnets[*].SubnetId" --output text | tr '\t' ',')
   ```

2. This command:
   - Queries all subnets in the us-east-1 region
   - Extracts only the SubnetIds
   - Converts tabs to commas for ECS service configuration format

3. Verified the subnets were correctly retrieved:
   ```bash
   echo $SUBNETS
   # Output: subnet-xxxxx,subnet-yyyyy,subnet-zzzzz,...
   ```

**Lesson Learned:** When using AWS CLI filters, verify the filter name in the current AWS CLI documentation. As an alternative, if a filter is uncertain, query all resources and the defaults will typically be suitable for Fargate deployments.

---

### Mistake 4: Security Group Ingress Rule Not Configured

**Description:** After deploying the ECS service and retrieving the public IP address, curl requests to the application hung indefinitely without returning any response.

**Error Message:**

```
curl http://54.91.48.249
# (command hangs/timeout occurs)
```

**Why It Happened:**

- The default security group (`sg-01d65516525a997aa`) used for the Fargate task did not have an inbound rule allowing HTTP traffic on port 80
- AWS security groups are stateful and deny all inbound traffic by default (whitelist-based)
- When no explicit inbound rule allows port 80, all HTTP requests are silently dropped
- The application was running correctly inside the container, but network access was blocked at the security group level

**How It Was Fixed:**

1. Added an inbound rule to the security group allowing HTTP traffic from anywhere:

   ```bash
   aws ec2 authorize-security-group-ingress \
       --group-id sg-01d65516525a997aa \
       --protocol tcp \
       --port 80 \
       --cidr 0.0.0.0/0 \
       --region us-east-1
   ```

2. Verified the rule was successfully added by checking the security group configuration

3. Retested the application accessibility:
   ```bash
   curl http://54.91.48.249
   # Successfully returned HTML content
   ```

**Important Security Consideration:** The `0.0.0.0/0` CIDR block allows traffic from any IP address globally. This is suitable for public-facing web applications but should be restricted to specific IP ranges or security groups for production environments requiring enhanced security.

**Lesson Learned:** In AWS, always verify that security groups have appropriate inbound rules configured for the ports and protocols that the application requires. Network connectivity issues often stem from misconfigured security groups rather than application failures. Test network connectivity explicitly before assuming the application has issues.

---

## Verification Steps

The following commands confirm successful deployment:

1. **Check Task Definition Registration:**

   ```bash
   aws ecs list-task-definitions --region us-east-1
   aws ecs describe-task-definition --task-definition xfusion-taskdefinition --region us-east-1
   ```

2. **Check Service Status:**

   ```bash
   aws ecs describe-services \
       --cluster xfusion-cluster \
       --services xfusion-service \
       --region us-east-1
   ```

3. **Check Running Tasks:**

   ```bash
   aws ecs list-tasks --cluster xfusion-cluster --region us-east-1
   aws ecs describe-tasks --cluster xfusion-cluster --tasks $TASK_ARN --region us-east-1
   ```

4. **Verify Application Response:**
   ```bash
   curl http://$PUBLIC_IP
   curl -I http://$PUBLIC_IP  # Show only headers
   ```

---

## Summary

This task successfully demonstrated a complete containerized application deployment workflow on AWS using modern cloud-native practices. The implementation covered:

- **Container Image Management:** Building and pushing Docker images to a private registry
- **Infrastructure as Code Concepts:** Infrastructure components defined and created via AWS CLI
- **Container Orchestration:** Using ECS with Fargate for simplified container management
- **Networking:** Configuring VPC, subnets, and security groups for container accessibility
- **Identity and Access Management:** Setting up IAM roles with proper trust relationships and permissions

The deployment is now accessible at the assigned public IP address, serving the containerized Nginx application with custom HTML content. All troubleshooting steps and lessons learned have been documented for future reference and similar deployments.

---

## Key Takeaways

1. **Environment Variables:** Always properly substitute variables in AWS CLI commands to avoid shell interpretation issues
2. **IAM Permissions:** Fargate tasks require explicit execution roles for accessing AWS services
3. **Security Groups:** Network access is blocked by default; explicitly configure inbound rules for required ports
4. **AWS CLI Filters:** Use current documentation for filter parameter names; query all resources when uncertain
5. **Testing:** Verify each deployment step independently to isolate issues quickly
