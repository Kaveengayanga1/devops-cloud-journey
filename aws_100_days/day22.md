# Day 22: AWS EC2 Passwordless SSH Setup via CLI

## Theory & Guide

### Overview
Setting up secure, passwordless access between a management host (like an `aws-client`) and an EC2 instance is a fundamental DevOps task. This process involves generating an asymmetric key pair and ensuring the public portion is recognized by the target server.

### Core Concepts

**1. SSH Key-Based Authentication**
Instead of a password, SSH uses Public Key Infrastructure (PKI).
*   **Private Key (`id_rsa`)**: Stays on the client (your machine). It must be kept secret.
*   **Public Key (`id_rsa.pub`)**: Uploaded to the EC2 instance.
*   **Connection**: When you connect, the EC2 instance sends a "challenge" that can only be signed by the matching private key.

**2. The `.ssh` Directory Permissions**
SSH is extremely strict about file permissions. If they are too open, SSH will reject the connection.
*   `.ssh` directory: Must be `700` (`drwx------`).
*   `authorized_keys`: Must be `600` (`-rw-------`).

### Implementation Steps

#### Step 1: Generate SSH Key Pair
Check if the key exists locally; if not, create it without a passphrase (`-N ""`).

```bash
if [ ! -f /root/.ssh/id_rsa ]; then
    ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
fi
```

#### Step 2: Launch EC2 with User Data (The "Here-Doc" Method)
Directly passing scripts with `#!` in the CLI can cause Bash "event not found" errors due to the exclamation mark. The best practice is to write the script to a file first.

1.  **Prepare the script locally**:
    ```bash
    PUB_KEY=$(cat /root/.ssh/id_rsa.pub)

    cat <<EOF > user_data.sh
    #!/bin/bash
    mkdir -p /root/.ssh
    echo "$PUB_KEY" >> /root/.ssh/authorized_keys
    chmod 700 /root/.ssh
    chmod 600 /root/.ssh/authorized_keys
    chown -R root:root /root/.ssh
    sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
    sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
    systemctl restart sshd
    EOF
    ```

2.  **Launch the instance**:
    ```bash
    aws ec2 run-instances \
        --image-id ami-0b0ea68c435eb488d \
        --count 1 \
        --instance-type t2.micro \
        --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
        --user-data file://user_data.sh
    ```

#### Step 3: Configure Security Group
If the connection times out, the default Security Group likely blocks Port 22.

```bash
# Get Group ID
SG_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].SecurityGroups[0].GroupId" \
    --output text)

# Open Port 22
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

#### Step 4: Verify Connection
```bash
INSTANCE_IP=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text)

ssh -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa root@$INSTANCE_IP
```

---

## Practical Output Log

```bash
~ on ☁️  (us-east-1) ➜  # Check if key exists, if not, generate it
if [ ! -f /root/.ssh/id_rsa ]; then
    ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
fi
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Ulo0We03Ll7xb8SecUrKAEaPkWAFirfWVfh1HBqWLoY root@aws-client
The key's randomart image is:
+---[RSA 2048]----+
|      +++=o.oo.. |
|   . o .*o .+oo  |
|  . o  .+* +..   |
|   . o =E = o +  |
|    o +.So . o = |
|   .   .  . . +.=|
|           + = +=|
|            + ..+|
|               . |
+----[SHA256]-----+

~ on ☁️  (us-east-1) ➜  ls

~ on ☁️  (us-east-1) ➜  aws ec2 run-instances \
    --image-id ami-0b0ea68c435eb488d \
    --count 1 \
    --instance-type t2.micro \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]'
{
    "ReservationId": "r-04edffc6362542854",
    "OwnerId": "602928953484",
    "Groups": [],
    "Instances": [
        {
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "741cf0b5-f79f-4a60-89d1-2b40a1ec3fbf",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2026-03-03T02:06:22.000Z",
                        "AttachmentId": "eni-attach-0d603b2217ffa793a",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupId": "sg-00fcf5e932850541c",
                            "GroupName": "default"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0a:ff:e0:71:86:11",
                    "NetworkInterfaceId": "eni-09d4e84142dba443a",
                    "OwnerId": "602928953484",
                    "PrivateDnsName": "ip-172-31-31-79.ec2.internal",
                    "PrivateIpAddress": "172.31.31.79",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-31-79.ec2.internal",
                            "PrivateIpAddress": "172.31.31.79"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0e1ad1e6e6da40835",
                    "VpcId": "vpc-0d93c506dc850a358",
                    "InterfaceType": "interface",
                    "Operator": {
                        "Managed": false
                    }
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupId": "sg-00fcf5e932850541c",
                    "GroupName": "default"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
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
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default",
                "RebootMigration": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios",
            "Operator": {
                "Managed": false
            },
            "InstanceId": "i-0bd6ef65b7e64d94e",
            "ImageId": "ami-0b0ea68c435eb488d",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "PrivateDnsName": "ip-172-31-31-79.ec2.internal",
            "PublicDnsName": "",
            "StateTransitionReason": "",
            "AmiLaunchIndex": 0,
            "ProductCodes": [],
            "InstanceType": "t2.micro",
            "LaunchTime": "2026-03-03T02:06:22.000Z",
            "Placement": {
                "GroupName": "",
                "Tenancy": "default",
                "AvailabilityZone": "us-east-1a"
            },
            "Monitoring": {
                "State": "disabled"
            },
            "SubnetId": "subnet-0e1ad1e6e6da40835",
            "VpcId": "vpc-0d93c506dc850a358",
            "PrivateIpAddress": "172.31.31.79"
        }
    ]
}

~ on ☁️  (us-east-1) ➜  PUB_KEY=$(cat /root/.ssh/id_rsa.pub)

~ on ☁️  (us-east-1) ➜  aws ec2 run-instances \
    --image-id ami-0b0ea68c435eb488d \
    --count 1 \
    --instance-type t2.micro \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
    --user-data "#!/bin/bash
                 mkdir -p /root/.ssh
                 echo '$PUB_KEY' >> /root/.ssh/authorized_keys
                 chmod 700 /root/.ssh
                 chmod 600 /root/.ssh/authorized_keys
                 chown -R root:root /root/.ssh"
-bash: !/bin/bash: event not found
Note: AWS CLI version 2, the latest major version of the AWS CLI, is now stable and recommended for general use. For more information, see the AWS CLI version 2 installation instructions at: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help

Unknown options: -p, /root/.ssh
> ^C

~ on ☁️  (us-east-1) ✖ aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
{
    "Reservations": [
        {
            "ReservationId": "r-04edffc6362542854",
            "OwnerId": "602928953484",
            "Groups": [],
            "Instances": [
                {
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/sda1",
                            "Ebs": {
                                "AttachTime": "2026-03-03T02:06:23.000Z",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-0720eba17627bbf90"
                            }
                        }
                    ],
                    "ClientToken": "741cf0b5-f79f-4a60-89d1-2b40a1ec3fbf",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "amazon",
                                "PublicDnsName": "ec2-54-227-122-58.compute-1.amazonaws.com",
                                "PublicIp": "54.227.122.58"
                            },
                            "Attachment": {
                                "AttachTime": "2026-03-03T02:06:22.000Z",
                                "AttachmentId": "eni-attach-0d603b2217ffa793a",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupId": "sg-00fcf5e932850541c",
                                    "GroupName": "default"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "0a:ff:e0:71:86:11",
                            "NetworkInterfaceId": "eni-09d4e84142dba443a",
                            "OwnerId": "602928953484",
                            "PrivateDnsName": "ip-172-31-31-79.ec2.internal",
                            "PrivateIpAddress": "172.31.31.79",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "amazon",
                                        "PublicDnsName": "ec2-54-227-122-58.compute-1.amazonaws.com",
                                        "PublicIp": "54.227.122.58"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-31-79.ec2.internal",
                                    "PrivateIpAddress": "172.31.31.79"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-0e1ad1e6e6da40835",
                            "VpcId": "vpc-0d93c506dc850a358",
                            "InterfaceType": "interface",
                            "Operator": {
                                "Managed": false
                            }
                        }
                    ],
                    "RootDeviceName": "/dev/sda1",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupId": "sg-00fcf5e932850541c",
                            "GroupName": "default"
                        }
                    ],
                    "SourceDestCheck": true,
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
                    },
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1,
                        "HttpEndpoint": "enabled",
                        "HttpProtocolIpv6": "disabled",
                        "InstanceMetadataTags": "disabled"
                    },
                    "EnclaveOptions": {
                        "Enabled": false
                    },
                    "PlatformDetails": "Linux/UNIX",
                    "UsageOperation": "RunInstances",
                    "UsageOperationUpdateTime": "2026-03-03T02:06:22.000Z",
                    "PrivateDnsNameOptions": {
                        "HostnameType": "ip-name",
                        "EnableResourceNameDnsARecord": false,
                        "EnableResourceNameDnsAAAARecord": false
                    },
                    "MaintenanceOptions": {
                        "AutoRecovery": "default",
                        "RebootMigration": "default"
                    },
                    "CurrentInstanceBootMode": "legacy-bios",
                    "NetworkPerformanceOptions": {
                        "BandwidthWeighting": "default"
                    },
                    "Operator": {
                        "Managed": false
                    },
                    "InstanceId": "i-0bd6ef65b7e64d94e",
                    "ImageId": "ami-0b0ea68c435eb488d",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "PrivateDnsName": "ip-172-31-31-79.ec2.internal",
                    "PublicDnsName": "ec2-54-227-122-58.compute-1.amazonaws.com",
                    "StateTransitionReason": "",
                    "AmiLaunchIndex": 0,
                    "ProductCodes": [],
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2026-03-03T02:06:22.000Z",
                    "Placement": {
                        "GroupName": "",
                        "Tenancy": "default",
                        "AvailabilityZone": "us-east-1a"
                    },
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "SubnetId": "subnet-0e1ad1e6e6da40835",
                    "VpcId": "vpc-0d93c506dc850a358",
                    "PrivateIpAddress": "172.31.31.79",
                    "PublicIpAddress": "54.227.122.58"
                }
            ]
        }
    ]
}

~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,Name:Tags[?Key=='Name']|[0].Value,IP:PublicIpAddress}" \
    --output table
---------------------------------------------------------
|                   DescribeInstances                   |
+---------------------+-----------------+---------------+
|         ID          |       IP        |     Name      |
+---------------------+-----------------+---------------+
|  i-0bd6ef65b7e64d94e|  54.227.122.58  |  xfusion-ec2  |
+---------------------+-----------------+---------------+

~ on ☁️  (us-east-1) ➜  aws ec2 terminate-instances --instance-ids i-0abcdef1234567890

An error occurred (InvalidInstanceID.Malformed) when calling the TerminateInstances operation: The instance ID 'i-0abcdef1234567890' is malformed

~ on ☁️  (us-east-1) ✖ aws ec2 terminate-instances --instance-ids i-0bd6ef65b7e64d94e|
> ^C

~ on ☁️  (us-east-1) ✖ aws ec2 terminate-instances --instance-ids i-0bd6ef65b7e64d94e
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-0bd6ef65b7e64d94e",
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

~ on ☁️  (us-east-1) ➜  cd /root/.ssh/

~/.ssh on ☁️  (us-east-1) ➜  ls
agent-environment  authorized_keys  id_rsa  id_rsa.pub

~/.ssh on ☁️  (us-east-1) ➜  chmod 700 /root/.ssh

~/.ssh on ☁️  (us-east-1) ➜  echo $PUB_KEY
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3bjuesPoNbxudZpdhn7C3THinNH6N9CzZHAj1u0H33i43z+Sf7y2d5UgQo98da0wLvOczc1dVJn6eBBhSH2KyQdMcUK6w0wRVSjXdTqAcdqDuvg1UME0Fj24P9EwViPlcvNbYcqOuwdJOsIFixxqbrNgk948295E0rewEHCQwG2O5HaTL1nf9qIdo7QbYX/HGpXBdAb4fVEAORPDZFFNEMCHPCyT7tMZXeX7i4USYSoE4iwUyWL5WQRp/i6nk1zkYFlLkwQntgDq580P/2UAogsF+I+3D9Q2DVBmDeVqk4WNEVW7hzH5697eJfNANsGiglKFSDMppiXjlk7l4CSsT root@aws-client

~/.ssh on ☁️  (us-east-1) ➜  aws ec2 run-instances \
    --image-id ami-0b0ea68c435eb488d \
    --count 1 \
    --instance-type t2.micro \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
    --user-data "#!/bin/bash
                 mkdir -p /root/.ssh
                 echo '$PUB_KEY' >> /root/.ssh/authorized_keys
                 chmod 700 /root/.ssh
                 chmod 600 /root/.ssh/authorized_keys
                 chown -R root:root /root/.ssh"
-bash: !/bin/bash: event not found
Note: AWS CLI version 2, the latest major version of the AWS CLI, is now stable and recommended for general use. For more information, see the AWS CLI version 2 installation instructions at: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help

Unknown options: -p, /root/.ssh
> ^C

~/.ssh on ☁️  (us-east-1) ✖ PUB_KEY=$(cat /root/.ssh/id_rsa.pub)

~/.ssh on ☁️  (us-east-1) ➜  aws ec2 run-instances \
    --image-id ami-0b0ea68c435eb488d \
    --count 1 \
    --instance-type t2.micro \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
    --user-data "#!/bin/bash
mkdir -p /root/.ssh
echo '$PUB_KEY' >> /root/.ssh/authorized_keys
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
chown -R root:root /root/.ssh
sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
systemctl restart sshd"
-bash: !/bin/bash: event not found
Note: AWS CLI version 2, the latest major version of the AWS CLI, is now stable and recommended for general use. For more information, see the AWS CLI version 2 installation instructions at: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help

Unknown options: -p, /root/.ssh
> ^C

~/.ssh on ☁️  (us-east-1) ✖ cat <<EOF > user_data.sh
#!/bin/bash
mkdir -p /root/.ssh
echo "$PUB_KEY" >> /root/.ssh/authorized_keys
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
chown -R root:root /root/.ssh
sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart sshd
EOF

~/.ssh on ☁️  (us-east-1) ➜  aws ec2 run-instances \
    --image-id ami-0b0ea68c435eb488d \
    --count 1 \
    --instance-type t2.micro \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
    --user-data file://user_data.sh
{
    "ReservationId": "r-06be3cfc062baf046",
    "OwnerId": "602928953484",
    "Groups": [],
    "Instances": [
        {
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "62cd1bd0-da08-41e0-9788-1e4083acf30d",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2026-03-03T02:24:11.000Z",
                        "AttachmentId": "eni-attach-09deb5ee3e0a152c6",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupId": "sg-00fcf5e932850541c",
                            "GroupName": "default"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0a:ff:fd:f4:78:29",
                    "NetworkInterfaceId": "eni-0f4a69c93d734ebd8",
                    "OwnerId": "602928953484",
                    "PrivateDnsName": "ip-172-31-19-82.ec2.internal",
                    "PrivateIpAddress": "172.31.19.82",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-19-82.ec2.internal",
                            "PrivateIpAddress": "172.31.19.82"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0e1ad1e6e6da40835",
                    "VpcId": "vpc-0d93c506dc850a358",
                    "InterfaceType": "interface",
                    "Operator": {
                        "Managed": false
                    }
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupId": "sg-00fcf5e932850541c",
                    "GroupName": "default"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
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
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default",
                "RebootMigration": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios",
            "Operator": {
                "Managed": false
            },
            "InstanceId": "i-0931f53715c8ef6f4",
            "ImageId": "ami-0b0ea68c435eb488d",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "PrivateDnsName": "ip-172-31-19-82.ec2.internal",
            "PublicDnsName": "",
            "StateTransitionReason": "",
            "AmiLaunchIndex": 0,
            "ProductCodes": [],
            "InstanceType": "t2.micro",
            "LaunchTime": "2026-03-03T02:24:11.000Z",
            "Placement": {
                "GroupName": "",
                "Tenancy": "default",
                "AvailabilityZone": "us-east-1a"
            },
            "Monitoring": {
                "State": "disabled"
            },
            "SubnetId": "subnet-0e1ad1e6e6da40835",
            "VpcId": "vpc-0d93c506dc850a358",
            "PrivateIpAddress": "172.31.19.82"
        }
    ]
}

~/.ssh on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].{ID:InstanceId,IP:PublicIpAddress}" \
    --output table
-------------------------------------------
|            DescribeInstances            |
+----------------------+------------------+
|          ID          |       IP         |
+----------------------+------------------+
|  i-0931f53715c8ef6f4 |  54.242.242.121  |
+----------------------+------------------+

~/.ssh on ☁️  (us-east-1) ➜  INSTANCE_IP=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text)

~/.ssh on ☁️  (us-east-1) ➜  echo $INSTANCE_IP
54.242.242.121

~/.ssh on ☁️  (us-east-1) ➜  ssh -i /root/.ssh/id_rsa root@$INSTANCE_IP
ssh: connect to host 54.242.242.121 port 22: Connection timed out

~/.ssh on ☁️  (us-east-1) ✖ SG_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=xfusion-ec2" "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].SecurityGroups[0].GroupId" \
    --output text)

echo "Security Group ID: $SG_ID"
Security Group ID: sg-00fcf5e932850541c

~/.ssh on ☁️  (us-east-1) ➜  aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-04d45111963983606",
            "GroupId": "sg-00fcf5e932850541c",
            "GroupOwnerId": "602928953484",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-1:602928953484:security-group-rule/sgr-04d45111963983606"
        }
    ]
}

~/.ssh on ☁️  (us-east-1) ➜  ssh -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa root@$INSTANCE_IP
Warning: Permanently added '54.242.242.121' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-1128-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

UA Infra: Extended Security Maintenance (ESM) is not enabled.

0 updates can be applied immediately.

44 additional security updates can be applied with UA Infra: ESM
Learn more about enabling UA Infra: ESM service for Ubuntu 16.04 at
https://ubuntu.com/16-04



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@ip-172-31-19-82:~# 
```
