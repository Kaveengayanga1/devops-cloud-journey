# Day 12 — Attach existing EBS volume to EC2 instance

An instance named `xfusion-ec2` and a volume named `xfusion-volume` already exist in the `us-east-1` region. This document shows how to attach the `xfusion-volume` volume to the `xfusion-ec2` instance with device name `/dev/sdb` using both the AWS Management Console and the AWS CLI, plus verification and common post-attach steps.

## Using the AWS Management Console

- Open the EC2 Dashboard in the AWS Console.
- Under Elastic Block Store → Volumes, find the volume named `xfusion-volume`. Ensure its state is `Available`.
- Select the volume → Actions → Attach volume.
- In the Instance field select `xfusion-ec2`.
- Replace the default device name with `/dev/sdb`.
- Click `Attach volume`.

## Using the AWS CLI

1. Get the Instance ID (filter by Name tag):

```bash
aws ec2 describe-instances \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text
```

2. Get the Volume ID (filter by Name tag):

```bash
aws ec2 describe-volumes \
  --region us-east-1 \
  --filters "Name=tag:Name,Values=xfusion-volume" \
  --query "Volumes[*].VolumeId" \
  --output text
```

3. Attach the volume (replace IDs with the values from steps 1–2):

```bash
aws ec2 attach-volume \
  --region us-east-1 \
  --volume-id <INSERT_VOLUME_ID_HERE> \
  --instance-id <INSERT_INSTANCE_ID_HERE> \
  --device /dev/sdb
```

## Important considerations

- Availability Zone: EBS volumes must be in the same AZ as the instance. If not, snapshot + restore in correct AZ is required.
- Mounting: After attaching, SSH into the instance, format (if new) and mount the device to use it.

Would you like the Linux commands to format and mount this volume once it's attached?

## How to check attached devices

From inside the instance (Linux):

- `lsblk` — shows block devices and mount points (device may appear as `/dev/xvdb` or `/dev/nvme1n1`).
- `lsblk --output NAME,SERIAL,SIZE,MOUNTPOINT` — SERIAL often contains the Volume ID for EBS devices.
- `df -h` — shows mounted filesystems only.

From the AWS CLI (external):

```bash
aws ec2 describe-volumes \
  --region us-east-1 \
  --filters Name=attachment.instance-id,Values=<INSTANCE_ID> \
  --query "Volumes[*].{ID:VolumeId,Device:Attachments[0].Device,State:State}" \
  --output table
```

From the AWS Console: select the instance `xfusion-ec2` → Storage tab → Block devices.

## Example session (IDs and outputs)

The following is an example interactive session that demonstrates retrieving IDs, attaching the volume, and verifying attachment.

1) Get the instance ID (output):

```
~ on ☁️  (us-east-1) ➜  aws ec2 describe-instances \
    --region us-east-1 \
    --filters "Name=tag:Name,Values=xfusion-ec2" \
    --query "Reservations[*].Instances[*].InstanceId" \
    --output text
i-09300107d104051a1
```

2) Describe volumes (shows `xfusion-volume` as available):

```
~ on ☁️  (us-east-1) ➜  aws ec2 describe-volumes
{
    "Volumes": [
        {
            "Iops": 3000,
            "VolumeType": "gp3",
            "MultiAttachEnabled": false,
            "Throughput": 125,
            "Operator": {
                "Managed": false
            },
            "VolumeId": "vol-056369b5a533e2fc8",
            "Size": 8,
            "SnapshotId": "snap-04819fa0d2620618b",
            "AvailabilityZone": "us-east-1a",
            "State": "in-use",
            "CreateTime": "2026-01-21T02:43:22.258Z",
            "Attachments": [
                {
                    "DeleteOnTermination": true,
                    "VolumeId": "vol-056369b5a533e2fc8",
                    "InstanceId": "i-09300107d104051a1",
                    "Device": "/dev/xvda",
                    "State": "attached",
                    "AttachTime": "2026-01-21T02:43:22.000Z"
                }
            ],
            "Encrypted": false
        },
        {
            "Iops": 100,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "xfusion-volume"
                }
            ],
            "VolumeType": "gp2",
            "MultiAttachEnabled": false,
            "Operator": {
                "Managed": false
            },
            "VolumeId": "vol-0c8cd5d4d3b142535",
            "Size": 5,
            "SnapshotId": "",
            "AvailabilityZone": "us-east-1a",
            "State": "available",
            "CreateTime": "2026-01-21T02:43:22.761Z",
            "Attachments": [],
            "Encrypted": false
        }
    ]
}
```

3) Attach the volume (command and response):

```bash
~ on ☁️  (us-east-1) ✖ aws ec2 attach-volume \
--region us-east-1 \
--volume-id vol-0c8cd5d4d3b142535 \
--instance-id i-09300107d104051a1 \
--device /dev/sdb
{
    "VolumeId": "vol-0c8cd5d4d3b142535",
    "InstanceId": "i-09300107d104051a1",
    "Device": "/dev/sdb",
    "State": "attaching",
    "AttachTime": "2026-01-21T02:49:57.424Z"
}
```

4) Verify attachments via CLI (table output):

```bash
~ on ☁️  (us-east-1) ➜  aws ec2 describe-volumes \
    --region us-east-1 \
    --filters Name=attachment.instance-id,Values=i-09300107d104051a1 \
    --query "Volumes[*].{ID:VolumeId,Device:Attachments[0].Device,State:State}" \
    --output table
--------------------------------------------------
|                 DescribeVolumes                |
+------------+-------------------------+---------+
|   Device   |           ID            |  State  |
+------------+-------------------------+---------+
|  /dev/xvda |  vol-056369b5a533e2fc8  |  in-use |
|  /dev/sdb  |  vol-0c8cd5d4d3b142535  |  in-use |
+------------+-------------------------+---------+
```

## Next steps (post-attach, on the instance)

- SSH into the instance: `ssh ec2-user@<public-ip>` (adjust user by distro).
- Run `lsblk` to find the new device name (may be shown as `/dev/xvdb` or `/dev/nvme1n1`).
- If the volume is new: create filesystem, e.g., `sudo mkfs -t ext4 /dev/xvdb` (confirm device name first).
- Create a mount point and mount: `sudo mkdir -p /mnt/xfusion && sudo mount /dev/xvdb /mnt/xfusion`.
- To persist across reboots, add an `/etc/fstab` entry (use UUID from `blkid`).

---

If you want, I can:

- Provide the exact Linux commands to format and mount the device (I can tailor them to the detected device name), or
- Provide a small script that finds the instance/volume IDs and attaches the volume automatically.

Please tell me which option you'd prefer.
