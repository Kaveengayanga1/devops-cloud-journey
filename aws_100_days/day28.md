# Day 28: Creating and Pushing Docker Images to Amazon ECR

In today's task, we will walk through the process of creating a private Amazon Elastic Container Registry (ECR) repository and pushing a Docker image to it using the AWS CLI. This is a fundamental skill for any DevOps or Cloud Engineer working with containerized applications on AWS.

## Theoretical Concepts

Before executing the commands, let's understand the key components:

1.  **Amazon ECR (Elastic Container Registry):** A fully managed container registry that makes it easy to store, manage, share, and deploy your container images and artifacts anywhere. It is secure, scalable, and reliable.
2.  **Private Repository:** A repository where you can store your Docker images. Access is controlled via IAM policies, ensuring only authorized users or services can pull/push images.
3.  **Repository URI:** Each ECR repository has a unique Uniform Resource Identifier (URI) used to tag and push images (e.g., `[aws_account_id].dkr.ecr.[region].amazonaws.com/repo-name`).
4.  **Docker Build and Tag:**
    *   **Dockerfile:** A text document that contains all the commands a user could call on the command line to assemble an image.
    *   **Tagging:** To push an image to ECR, the local Docker image must be "tagged" with the remote repository's URI. This tells the Docker client where to push the image.
5.  **Authentication:** AWS ECR requires authentication. The `aws ecr get-login-password` command retrieves a password to authenticate your Docker client to your registry.

## Step-by-Step Implementation

Follow these steps to create a repository, build an image, and push it to ECR.

### Prerequisites
*   AWS CLI installed and configured.
*   Docker installed and running.
*   A sample application with a `Dockerfile` (e.g., in `/root/pyapp`).

### 1. Create an ECR Repository
First, create a new private repository named `datacenter-ecr` in the `us-east-1` region.

```bash
aws ecr create-repository \
    --repository-name datacenter-ecr \
    --region us-east-1
```

*Identify the `repositoryUri` from the output (e.g., `826068004857.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr`).*

### 2. Authenticate Docker to ECR
Retrieve an authentication token and authenticate your Docker client to your registry. Replace `[aws_account_id]` with your actual AWS account ID.

```bash
# Get Account ID (Optional, for verification)
aws sts get-caller-identity --query Account --output text

# Login
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [aws_account_id].dkr.ecr.us-east-1.amazonaws.com
```

### 3. Build the Docker Image
Navigate to your application directory containing the `Dockerfile` and build the image.

```bash
cd /root/pyapp
ls
# Output: app.py  Dockerfile  requirements.txt

docker build -t datacenter-ecr .
```

### 4. Tag the Image
Tag your local image with the ECR repository URI so it can be pushed to the correct destination.

```bash
docker tag datacenter-ecr:latest [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
```

### 5. Push the Image to ECR
Finally, push the tagged image to your private ECR repository.

```bash
docker push [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
```

## Summary Checklist
*   **Create:** `aws ecr create-repository`
*   **Auth:** `aws ecr get-login-password | docker login`
*   **Build:** `docker build`
*   **Tag:** `docker tag`
*   **Push:** `docker push`
