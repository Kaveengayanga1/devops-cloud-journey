# Day 34 - AWS Lambda via CLI: Create and Invoke Function from ZIP

## 📝 Task Description
The Nautilus DevOps team continues serverless practice with AWS Lambda. In this task, we create a Python Lambda function package, deploy it using AWS CLI, and verify the output response.

### 📋 Requirements
1. Create `lambda_function.py` with a handler returning:
   - `statusCode`: `200`
   - `body`: `Welcome to KKE AWS Labs!`
2. Zip the script as `function.zip`.
3. Create Lambda function `nautilus-lambda-cli` using Python runtime.
4. Use IAM role `lambda_execution_role`.
5. Perform all actions from `aws-client` using already configured AWS CLI.

---

## 📚 Core Theories

1. **Serverless and AWS Lambda**
   - AWS Lambda runs code without managing servers.
   - AWS handles provisioning, scaling, and availability.
   - Billing is based on requests and execution duration.

2. **Lambda Handler Contract**
   - Python entry point format: `lambda_function.lambda_handler`.
   - File name (`lambda_function.py`) + function name (`lambda_handler`) must match the handler configuration.
   - The handler usually returns a JSON-serializable dictionary.

3. **Deployment Package (ZIP)**
   - For `Zip` package type, code must be uploaded as a `.zip` archive.
   - For simple functions, the ZIP can contain just one file.

4. **IAM Execution Role**
   - Lambda must assume an IAM role to run and write logs.
   - `lambda_execution_role` should include at least basic Lambda execution permissions (for CloudWatch logs).
   - `create-function` requires full role ARN, not only role name.

5. **CLI Invocation vs Function Response**
   - `aws lambda invoke` output `StatusCode: 200` means invoke API call succeeded.
   - Actual function return payload is written to the output file (for example `response.json`).
   - Validate both invoke status and payload content.

6. **Function State Awareness**
   - Right after creation, Lambda may show `State: Pending` briefly.
   - Invocation can still succeed once function becomes active.

---

## 🚀 Step-by-Step Execution (AWS CLI)

### 1. Create Python script
```bash
cat <<EOF > lambda_function.py
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Welcome to KKE AWS Labs!'
    }
EOF
```

### 2. Zip the script
```bash
zip function.zip lambda_function.py
```

### 3. Confirm artifacts
```bash
ls
```
Expected files include:
- `lambda_function.py`
- `function.zip`

### 4. Get IAM role ARN
```bash
ROLE_ARN=$(aws iam get-role --role-name lambda_execution_role --query 'Role.Arn' --output text)
echo $ROLE_ARN
```
Expected format:
- `arn:aws:iam::<account-id>:role/lambda_execution_role`

### 5. Create Lambda function
```bash
aws lambda create-function \
    --function-name nautilus-lambda-cli \
    --runtime python3.9 \
    --handler lambda_function.lambda_handler \
    --role $ROLE_ARN \
    --zip-file fileb://function.zip
```
Check response fields such as:
- `FunctionName: nautilus-lambda-cli`
- `Runtime: python3.9`
- `Handler: lambda_function.lambda_handler`
- `Role: ...:role/lambda_execution_role`

### 6. Invoke Lambda
```bash
aws lambda invoke \
    --function-name nautilus-lambda-cli \
    response.json
```
Expected CLI response:
- `StatusCode: 200`
- `ExecutedVersion: $LATEST`

### 7. Verify function output payload
```bash
cat response.json
```
Expected payload:
```json
{"statusCode": 200, "body": "Welcome to KKE AWS Labs!"}
```

---

## ✅ Final Validation Checklist
- `lambda_function.py` was created with correct handler response.
- `function.zip` was created and used in deployment.
- IAM role ARN was fetched from `lambda_execution_role`.
- Lambda `nautilus-lambda-cli` was created successfully.
- `aws lambda invoke` returned `StatusCode: 200`.
- `response.json` contains `{"statusCode": 200, "body": "Welcome to KKE AWS Labs!"}`.
- Files present after test: `lambda_function.py`, `function.zip`, `response.json`.
