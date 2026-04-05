# Day 15 — Create snapshot from existing EBS volume (`datacenter-vol`)

Task: Create a snapshot from the EBS volume named `datacenter-vol` in `us-east-1`, tag the snapshot as `datacenter-vol-ss`, wait until the snapshot completes, and verify its completed state.

## Procedure

1. Get the Volume ID

```bash
VOL_ID=$(aws ec2 describe-volumes \
    --region us-east-1 \
    --filters "Name=tag:Name,Values=datacenter-vol" \
    --query "Volumes[0].VolumeId" \
    --output text)
```

2. Create the Snapshot

```bash
SNAP_ID=$(aws ec2 create-snapshot \
    --region us-east-1 \
    --volume-id $VOL_ID \
    --description "datacenter Snapshot" \
    --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=datacenter-vol-ss}]' \
    --query "SnapshotId" \
    --output text)

echo "Snapshot creation started: $SNAP_ID"
```

3. Check the Status

- One-time status check:

```bash
aws ec2 describe-snapshots \
    --region us-east-1 \
    --snapshot-ids $SNAP_ID \
    --query "Snapshots[0].{State:State,Progress:Progress}" \
    --output table
```

- Recommended (wait until completion):

```bash
aws ec2 wait snapshot-completed \
    --region us-east-1 \
    --snapshot-ids $SNAP_ID

echo "Snapshot $SNAP_ID is now COMPLETED."
```

## Example session (recorded outputs)

```
~ on ☁️  (us-east-1) ➜  aws ec2 describe-volumes
{
    "Volumes": [
        {
            "Iops": 100,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "datacenter-vol"
                }
            ],
            "VolumeType": "gp2",
            "MultiAttachEnabled": false,
            "Operator": {
                "Managed": false
            },
            "VolumeId": "vol-0707c80569fb67e10",
            "Size": 5,
            "SnapshotId": "",
            "AvailabilityZone": "us-east-1a",
            "State": "available",
            "CreateTime": "2026-01-21T03:37:45.077Z",
            "Attachments": [],
            "Encrypted": false
        }
    ]
}

~ on ☁️  (us-east-1) ➜  H
-bash: H: command not found

~ on ☁️  (us-east-1) ✖ VOL_ID=$(aws ec2 describe-volumes \
    --region us-east-1 \
    --filters "Name=tag:Name,Values=datacenter-vol" \
    --query "Volumes[0].VolumeId" \
    --output text)

~ on ☁️  (us-east-1) ➜  VOL_ID
-bash: VOL_ID: command not found

~ on ☁️  (us-east-1) ✖ $VOL_ID
-bash: vol-0707c80569fb67e10: command not found

~ on ☁️  (us-east-1) ✖ SNAP_ID=$(aws ec2 create-snapshot \
    --region us-east-1 \
    --volume-id $VOL_ID \
    --description "datacenter Snapshot" \
    --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=datacenter-vol-ss}]' \
    --query "SnapshotId" \
    --output text)

echo "Snapshot creation started: $SNAP_ID"
Snapshot creation started: snap-00207fab2e83a0ea7

~ on ☁️  (us-east-1) ➜  aws ec2 describe-snapshots \
    --region us-east-1 \
    --snapshot-ids $SNAP_ID \
    --query "Snapshots[0].{State:State,Progress:Progress}" \
    --output table
-------------------------
|   DescribeSnapshots   |
+-----------+-----------+
| Progress  |   State   |
+-----------+-----------+
|  0%       |  pending  |
+-----------+-----------+

~ on ☁️  (us-east-1) ➜  aws ec2 wait snapshot-completed \
    --region us-east-1 \
    --snapshot-ids $SNAP_ID

echo "Snapshot $SNAP_ID is now COMPLETED."
Snapshot snap-00207fab2e83a0ea7 is now COMPLETED.

~ on ☁️  (us-east-1) ➜  aws ec2 describe-snapshots \
    --region us-east-1 \
    --snapshot-ids $SNAP_ID \
    --query "Snapshots[0].{State:State,Progress:Progress}" \
    --output table
---------------------------
|    DescribeSnapshots    |
+-----------+-------------+
| Progress  |    State    |
+-----------+-------------+
|  100%     |  completed  |
+-----------+-------------+

~ on ☁️  (us-east-1) ➜  
```

## Summary Table for Verification

- Region: `us-east-1`
- Snapshot Name: `datacenter-vol-ss`
- Description: `datacenter Snapshot`
- Desired State: `completed`
