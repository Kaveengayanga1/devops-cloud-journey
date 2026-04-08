# Day 29: Configuring VPC Peering Connectivity

In today's task, we will establish network connectivity between our Default VPC (acting as a public/management network) and our custom Private VPC using VPC Peering. This allows instances in different VPCs to communicate with each other securely using private IP addresses, which is a critical skill for managing multi-VPC architectures.

## Theoretical Concepts

Before executing the commands, let's understand the key components:

1.  **VPC Peering Connection:** A networking connection between two VPCs that enables you to route traffic between them using private IPv4 addresses or IPv6 addresses. Both VPCs can communicate with each other as if they are within the same network. One VPC acts as the **Requester**, and the other as the **Accepter**.
2.  **Route Tables:** Merely creating a peering connection does not enable traffic flow. You must explicitly update route tables in *both* VPCs to direct traffic destined for the peer VPC's CIDR block to the VPC peering connection (`pcx-xxxxx`).
3.  **Security Groups:** Instance-level firewalls (Security Groups) must be updated to allow traffic (such as ICMP or SSH) from the peer VPC's CIDR block.
4.  **Non-Overlapping CIDRs:** A fundamental requirement for VPC peering is that the VPCs cannot have matching or overlapping IPv4 CIDR blocks. In our case, the Default VPC uses `172.31.0.0/16` and our Private VPC uses `10.1.0.0/16`, so peering is possible.

## Step-by-Step Implementation

### Step 1: Retrieve VPC IDs

First, we need to identify the VPC IDs for both the Default VPC and our custom Private VPC. We use `aws ec2 describe-vpcs` with filters to capture these IDs into variables.

```bash
# Get Default VPC ID
export PUBLIC_VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)

# Get Private VPC ID
export PRIVATE_VPC_ID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=devops-private-vpc" --query "Vpcs[0].VpcId" --output text)

# Verify the IDs
echo $PUBLIC_VPC_ID
# Output: vpc-03cf2a34e41655eb7

echo $PRIVATE_VPC_ID
# Output: vpc-048273d7f58ae851d
```

### Step 2: Create and Accept VPC Peering Connection

We initiate the peering request from the Public VPC to the Private VPC. Since both VPCs are in the same account, we can immediately accept the request.

```bash
# Create the peering connection
export PEERING_CONN_ID=$(aws ec2 create-vpc-peering-connection \
    --vpc-id $PUBLIC_VPC_ID \
    --peer-vpc-id $PRIVATE_VPC_ID \
    --query "VpcPeeringConnection.VpcPeeringConnectionId" --output text)

# Tag the connection for easier identification
aws ec2 create-tags --resources $PEERING_CONN_ID --tags Key=Name,Value=devops-vpc-peering

# Accept the peering connection request
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id $PEERING_CONN_ID
```
**Output from acceptance:**
```json
{
    "VpcPeeringConnection": {
        "AccepterVpcInfo": {
            "CidrBlock": "10.1.0.0/16",
            "OwnerId": "435388912352",
            "VpcId": "vpc-048273d7f58ae851d",
            "Region": "us-east-1"
        },
        "RequesterVpcInfo": {
            "CidrBlock": "172.31.0.0/16",
            "OwnerId": "435388912352",
            "VpcId": "vpc-03cf2a34e41655eb7",
            "Region": "us-east-1"
        },
        "Status": {
            "Code": "provisioning",
            "Message": "Provisioning"
        },
        "VpcPeeringConnectionId": "pcx-065f4cdb2db741a50"
    }
}
```

### Step 3: Update Route Tables

Now that the bridge is built, we must tell the routers in each VPC how to cross it. We add routes pointing to the *other* VPC's CIDR block via the Peering Connection ID.

```bash
# Get Route Table ID for Public VPC
PUBLIC_RT_ID=$(aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$PUBLIC_VPC_ID" --query "RouteTables[0].RouteTableId" --output text)

# Get Route Table ID for Private VPC
PRIVATE_RT_ID=$(aws ec2 describe-route-tables --filters "Name=vpc-id,Values=$PRIVATE_VPC_ID" --query "RouteTables[0].RouteTableId" --output text)

echo $PUBLIC_RT_ID
# Output: rtb-0489f23e0e49f7efe

echo $PRIVATE_RT_ID
# Output: rtb-06e43a2e957b9efdf

# Add route in Public RT pointing to Private CIDR (10.1.0.0/16)
aws ec2 create-route --route-table-id $PUBLIC_RT_ID --destination-cidr-block 10.1.0.0/16 --vpc-peering-connection-id $PEERING_CONN_ID

# Add route in Private RT pointing to Public CIDR (172.31.0.0/16)
aws ec2 create-route --route-table-id $PRIVATE_RT_ID --destination-cidr-block 172.31.0.0/16 --vpc-peering-connection-id $PEERING_CONN_ID
```

### Step 4: Update Security Groups

We need to allow traffic into our Private instance. Specifically, we'll allow ICMP (Ping) traffic originating from the Public VPC's CIDR block.

```bash
# Get Security Group ID for the Private Instance
SG_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=devops-private-ec2" --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" --output text)

# Authorize ICMP ingress from 172.31.0.0/16
aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol icmp --port -1 --cidr 172.31.0.0/16
```

### Step 5: Verify Connectivity (SSH & Ping)

Finally, we test the connection. We will SSH into a public instance (jump host) in the Default VPC and ping the private instance in the Private VPC.

1.  **Prepare SSH Access:** Ensure your public key is authorized on the public instance.
    ```bash
    # (Optional) Verify key existence
    cat /root/.ssh/id_rsa.pub
    
    # Send SSH public key to the instance (if using EC2 Instance Connect)
    aws ec2-instance-connect send-ssh-public-key \
        --instance-id i-0feaae03de78c94de \
        --availability-zone us-east-1c \
        --instance-os-user ec2-user \
        --ssh-public-key file:///root/.ssh/id_rsa.pub
    ```

2.  **SSH into Public Instance:**
    ```bash
    ssh ec2-user@52.201.245.127
    ```

3.  **Ping Private Instance:**
    Once logged into the public instance, ping the private IP of the instance in the Private VPC (`10.1.1.79`).

    ```bash
    [ec2-user@ip-172-31-29-234 ~]$ ping 10.1.1.79
    PING 10.1.1.79 (10.1.1.79) 56(84) bytes of data.
    64 bytes from 10.1.1.79: icmp_seq=1 ttl=127 time=1.36 ms
    64 bytes from 10.1.1.79: icmp_seq=2 ttl=127 time=0.874 ms
    64 bytes from 10.1.1.79: icmp_seq=3 ttl=127 time=0.885 ms
    ...
    ```

If you receive reply packets, the VPC Peering connection is successfully established and actively routing traffic!
