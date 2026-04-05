# Day 6: Creating an EC2 Key Pair and Launching an Instance

## Overview
This guide demonstrates how to create an RSA key pair and launch an EC2 instance using the AWS CLI. The key pair is used for secure SSH access to the instance.

## Prerequisites
- AWS CLI installed and configured
- Appropriate IAM permissions to create key pairs and launch EC2 instances
- Valid Amazon Linux AMI ID for your region

## Task Requirements

| Requirement | Implementation |
|-------------|----------------|
| Instance Name | `xfusion-ec2` |
| AMI | Amazon Linux (region-specific) |
| Instance Type | `t2.micro` |
| Key Pair | `xfusion-kp` (RSA type) |
| Security Group | Default (if not specified) |

## Step 1: Create the RSA Key Pair

You must create the key pair before launching the instance so it can be referenced during instance creation.

### Command

```bash
aws ec2 create-key-pair \
    --key-name xfusion-kp \
    --key-type rsa \
    --query 'KeyMaterial' \
    --output text > xfusion-kp.pem
```

### What This Does

- Creates an RSA key pair named `xfusion-kp` in AWS
- Extracts the private key material using `--query 'KeyMaterial'`
- Saves the private key to a file named `xfusion-kp.pem` in your current directory

### Verify the Key File

```bash
ls
```

**Output:**
```
xfusion-kp.pem
```

### Secure the Private Key

On Linux/Mac, set the correct permissions to secure the private key file:

```bash
chmod 400 xfusion-kp.pem
```

This ensures only the file owner can read the private key, which is required for SSH authentication.

## Step 2: Launch the EC2 Instance

Once the key pair exists, launch the EC2 instance using the following command.

### Command

```bash
aws ec2 run-instances \
    --image-id ami-0ff8a91507f77f867 \
    --count 1 \
    --instance-type t2.micro \
    --key-name xfusion-kp \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]'
```

### Breakdown of Flags

| Flag | Value | Description |
|------|-------|-------------|
| `--image-id` | ami-0ff8a91507f77f867 | Amazon Linux AMI ID (region-specific) |
| `--count` | 1 | Number of instances to launch |
| `--instance-type` | t2.micro | Instance size (eligible for free tier) |
| `--key-name` | xfusion-kp | The key pair name created in Step 1 |
| `--tag-specifications` | Key=Name,Value=xfusion-ec2 | Assigns the name "xfusion-ec2" to the instance |

**Note:** Replace `ami-0ff8a91507f77f867` with a valid Amazon Linux AMI ID for your specific region if needed.

## Expected Output

When the instance is successfully launched, you'll receive detailed JSON output:

```json
{
    "ReservationId": "r-0d48d40bf5bef7f84",
    "OwnerId": "440900363606",
    "Groups": [],
    "Instances": [
        {
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "61abf5b5-04d8-4e1e-9c6d-cbddb607dba7",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "InstanceId": "i-0bc4969885d4eb8f6",
            "ImageId": "ami-0ff8a91507f77f867",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "PrivateDnsName": "ip-172-31-23-32.ec2.internal",
            "PublicDnsName": "",
            "StateTransitionReason": "",
            "KeyName": "xfusion-kp",
            "AmiLaunchIndex": 0,
            "ProductCodes": [],
            "InstanceType": "t2.micro",
            "LaunchTime": "2026-01-13T03:06:53.000Z",
            "Placement": {
                "GroupName": "",
                "Tenancy": "default",
                "AvailabilityZone": "us-east-1c"
            },
            "Monitoring": {
                "State": "disabled"
            },
            "SubnetId": "subnet-0d2218925afdca90f",
            "VpcId": "vpc-05545b7be770504f3",
            "PrivateIpAddress": "172.31.23.32",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2026-01-13T03:06:53.000Z",
                        "AttachmentId": "eni-attach-000bf5573f48220f8",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupId": "sg-02811ebf7cc3a0494",
                            "GroupName": "default"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0a:ff:e1:f6:f4:cd",
                    "NetworkInterfaceId": "eni-01e6937be7a939cdf",
                    "OwnerId": "440900363606",
                    "PrivateDnsName": "ip-172-31-23-32.ec2.internal",
                    "PrivateIpAddress": "172.31.23.32",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-23-32.ec2.internal",
                            "PrivateIpAddress": "172.31.23.32"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0d2218925afdca90f",
                    "VpcId": "vpc-05545b7be770504f3",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupId": "sg-02811ebf7cc3a0494",
                    "GroupName": "default"
                }
            ],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "xfusion-ec2"
                }
            ],
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            }
        }
    ]
}
```

## Key Output Fields

- **InstanceId**: `i-0bc4969885d4eb8f6` - Unique identifier for your instance
- **State**: `pending` - Instance is being provisioned
- **KeyName**: `xfusion-kp` - The key pair associated with this instance
- **PrivateIpAddress**: `172.31.23.32` - Private IP within the VPC
- **AvailabilityZone**: `us-east-1c` - Where the instance is located
- **SecurityGroups**: `default` - Default security group is applied
- **InstanceType**: `t2.micro` - Instance size

## Important Notes

1. **Key Pair Order**: The key pair must be created before launching the instance.
2. **File Permissions**: The private key file must have `400` permissions for SSH to accept it.
3. **AMI ID**: The AMI ID is region-specific. Use `aws ec2 describe-images` to find valid AMIs for your region.
4. **Security Group**: When not specified, AWS uses the default security group, which may need modification for SSH access.
5. **Key Storage**: Keep the `.pem` file secure. If lost, you cannot recover it and will lose SSH access to the instance.

## Next Steps

After launching the instance:
1. Wait for the instance state to change from `pending` to `running`
2. Obtain the public IP address using `aws ec2 describe-instances`
3. Configure security group to allow SSH (port 22) access
4. Connect to the instance using: `ssh -i xfusion-kp.pem ec2-user@<public-ip>`

## Common Commands

### Check Instance Status
```bash
aws ec2 describe-instances --instance-ids i-0bc4969885d4eb8f6
```

### Get Public IP
```bash
aws ec2 describe-instances \
    --instance-ids i-0bc4969885d4eb8f6 \
    --query 'Reservations[0].Instances[0].PublicIpAddress' \
    --output text
```

### Connect via SSH
```bash
ssh -i xfusion-kp.pem ec2-user@<public-ip>
```
