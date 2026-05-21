# Day 33 - AWS Lambda: Create a Basic Serverless Function

## 📝 Task Description
The Nautilus DevOps team is adopting serverless architecture and needs a simple AWS Lambda function to demonstrate rapid deployment and auto-scaling capabilities.

### 📋 Requirements
As a member of the Nautilus DevOps Team, complete the following using the **AWS Console**:
1. Create a Lambda function named `nautilus-lambda`.
2. Use **Python** runtime.
3. Ensure the function returns status code `200`.
4. Ensure the response body is `Welcome to KKE AWS Labs!`.
5. Create and use IAM role `lambda_execution_role`.

---

## 📚 Core Theories

1. **Serverless with AWS Lambda**
   - AWS Lambda lets you run code without managing servers.
   - You pay for execution time, and scaling is handled automatically by AWS.

2. **Why IAM Role is Required**
   - Lambda needs permissions to interact with AWS services (especially CloudWatch Logs).
   - An execution role (`lambda_execution_role`) is attached to the function so Lambda can run securely with least privilege.

3. **Lambda Handler (Python)**
   - The handler is the function entry point, usually `lambda_handler(event, context)`.
   - It receives request/event data and returns a response object.

4. **Status Code and Body**
   - Returning `statusCode: 200` indicates success.
   - Returning the expected body confirms the business output is correct.

5. **CloudWatch Logs for Verification**
   - `print()` outputs are written to CloudWatch Logs.
   - This helps verify that the function executed and printed the expected message.

---

## 🚀 Execution Steps (AWS Console)

### 1. Get AWS Credentials
From `aws-client`, run `showcreds` and use those credentials to sign in to the AWS Console for this lab.

### 2. Create IAM Role
1. Open **IAM** in AWS Console.
2. Go to **Roles** → **Create role**.
3. Select **AWS service** as trusted entity.
4. Choose **Lambda** use case.
5. Attach policy: `AWSLambdaBasicExecutionRole`.
6. Role name: `lambda_execution_role`.
7. Click **Create role**.

### 3. Create Lambda Function
1. Open **Lambda** service.
2. Click **Create function**.
3. Select **Author from scratch**.
4. Set:
   - Function name: `nautilus-lambda`
   - Runtime: `Python` (latest available, e.g., Python 3.14)
5. Under **Permissions**, expand **Change default execution role**.
6. Select **Use an existing role**.
7. Choose `lambda_execution_role`.
8. Click **Create function**.

### 4. Add Function Code and Deploy
In `lambda_function.py`, replace existing code with:

```python
def lambda_handler(event, context):
    message = "Welcome to KKE AWS Labs!"
    print(message)
    return {
        'statusCode': 200,
        'body': message
    }
```

Then click **Deploy**.

### 5. Test the Function
1. Click **Test**.
2. Create a test event (any name, e.g., `TestEvent`) and save.
3. Run **Test** again.

### 6. Verify Output
Confirm the following:
- Execution status is **Succeeded**.
- Response contains:
  - `statusCode`: `200`
  - `body`: `Welcome to KKE AWS Labs!`
- CloudWatch logs show printed line: `Welcome to KKE AWS Labs!`

---

## ✅ Final Validation Checklist
- Lambda function name is `nautilus-lambda`.
- Runtime is Python.
- IAM role `lambda_execution_role` is attached.
- Function is deployed successfully.
- Test output returns status code `200` and body `Welcome to KKE AWS Labs!`.
