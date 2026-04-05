# Day 5: Creating an AWS EBS Volume

## Overview
This guide demonstrates how to create an Amazon Elastic Block Store (EBS) volume using the AWS CLI with specific configurations.

## Prerequisites
- AWS CLI installed and configured
- Appropriate IAM permissions to create EBS volumes
- Knowledge of your target Availability Zone

## Creating the EBS Volume

### AWS CLI Command

Run the following command in your terminal:

```bash
aws ec2 create-volume \
    --availability-zone us-east-1a \
    --size 2 \
    --volume-type gp3 \
    --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=nautilus-volume}]'
```

### Breakdown of Flags

| Flag | Value | Description |
|------|-------|-------------|
| `--availability-zone` | us-east-1a | Specifies the AWS Availability Zone where the volume will be created. **Note:** Change this to match the zone where your EC2 instance resides. |
| `--size` | 2 | Sets the size of the volume to 2 GiB. |
| `--volume-type` | gp3 | Sets the volume type to General Purpose SSD (gp3), which provides a balance of price and performance. |
| `--tag-specifications` | Key=Name,Value=nautilus-volume | Assigns the specific name "nautilus-volume" to the resource for easy identification. |

## Expected Output

When the command executes successfully, you'll receive output similar to:

```json
{
    "Iops": 3000,
    "Tags": [
        {
            "Key": "Name",
            "Value": "nautilus-volume"
        }
    ],
    "VolumeType": "gp3",
    "MultiAttachEnabled": false,
    "Throughput": 125,
    "VolumeId": "vol-0b8850fdf7444b247",
    "State": "creating",
    "SnapshotId": "",
    "AvailabilityZone": "us-east-1a",
    "Size": 2,
    "CreateTime": "2026-01-13T02:56:16.000Z",
    "Encrypted": false
}
```

## Key Output Fields

- **VolumeId**: `vol-0b8850fdf7444b247` - The unique identifier for your volume
- **State**: `creating` - The volume is being provisioned
- **Iops**: 3000 - Default IOPS for gp3 volumes
- **Throughput**: 125 MB/s - Default throughput for gp3 volumes
- **Encrypted**: false - Volume is not encrypted by default

## Important Notes

1. **Availability Zone**: Ensure the volume is created in the same Availability Zone as your EC2 instance for attachment.
2. **Default Settings**: gp3 volumes come with 3000 IOPS and 125 MB/s throughput by default.
3. **Encryption**: The volume is not encrypted by default. Add `--encrypted` flag if encryption is required.
4. **Billing**: You will be charged for the volume from the creation time, even if not attached to an instance.

## Next Steps

After creating the volume, you can:
- Attach it to an EC2 instance using `aws ec2 attach-volume`
- Format and mount the volume within your instance
- Create snapshots for backup purposes
- Modify volume properties (size, IOPS, throughput) as needed
