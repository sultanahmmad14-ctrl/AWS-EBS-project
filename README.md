# AWS EBS Lab: Volume, Snapshot and Cross-AZ Restore

A hands-on lab on Amazon EBS (Elastic Block Store). I created a volume, attached it to an EC2 instance, took a snapshot, and restored that snapshot as a new volume in a **different Availability Zone**, then attached it to a second EC2 instance.

## Objective

Understand that an EBS volume lives in a single Availability Zone (AZ), and learn how snapshots let you recreate a volume in another AZ.

## Environment

| Item | Value |
|---|---|
| Region | US East (N. Virginia) `us-east-1` |
| Instance type | t3.micro |
| AMI | Amazon Linux |
| Volume type | General Purpose SSD (gp2) |
| Volume size | 1 GiB |
| Key pair | None (all steps done from the AWS Console) |

## Architecture

```
us-east-1c                          us-east-1b
+---------------------+             +---------------------+
| EC2: AWS EBS Lab    |             | EC2: AWS EBS lab 2  |
|   |                 |             |   |                 |
| ebs-vol-1c (1 GiB)  |             | ebs-vol-1b (1 GiB)  |
+---------------------+             +---------------------+
          |                                   ^
          |  create snapshot                  |  create volume from snapshot
          +------------>  Snapshot  ----------+
```

## Steps performed

### Part 1: Create and attach a volume
1. Launched the first EC2 instance, `AWS EBS Lab`, in `us-east-1c`.
2. Went to **EC2 > Elastic Block Store > Volumes > Create volume**.
3. Chose **gp2**, size **1 GiB**, Availability Zone **us-east-1c**, and tagged it `ebs-vol-1c`.
4. Selected the volume, then **Actions > Attach volume**, picked `AWS EBS Lab`, device name `/dev/sdf`.
5. Verified under **Instances > Storage > Block devices** that the new 1 GiB volume is attached.

### Part 2: Create a snapshot
1. In **Volumes**, selected `ebs-vol-1c`.
2. **Actions > Create snapshot**.
3. Checked **Snapshots** until the status changed to `completed`.

### Part 3: Restore the snapshot in a different AZ
1. Selected the snapshot, then **Actions > Create volume from snapshot**.
2. Chose **gp2** and Availability Zone **us-east-1b** (different from the first instance's AZ).
3. Tagged the new volume `ebs-vol-1b`.

### Part 4: Attach to a second instance
1. Launched a second EC2 instance, `AWS EBS lab 2`, in **us-east-1b**.
2. Attached `ebs-vol-1b` to it as `/dev/sdf`.
3. Verified the volume under the instance's **Storage** tab.

## Screenshots

### Volume settings (gp2, 1 GiB, us-east-1c)
![Create volume](screenshots/01-create-volume-1c.png)

### Name tag on the volume
![Name tag](screenshots/02-volume-name-tag.png)

### Volumes list
![Volumes list](screenshots/03-volumes-list.png)

### Restored volume `ebs-vol-1b` (us-east-1b) attached to the second instance
![Volume 1b in use](screenshots/04-volume-1b-in-use.png)

### Second instance running in us-east-1b
![Instance in us-east-1b](screenshots/05-instance2-az-1b.png)

## What I learned

- An EBS volume can only be attached to an EC2 instance in the **same Availability Zone**.
- A snapshot is a backup of a volume and is **not tied to one AZ**, so a new volume can be created from it in any AZ.
- The AZ of an EC2 instance cannot be changed after launch. My second instance was first launched in `us-east-1c` by mistake, so I terminated it and relaunched it in `us-east-1b`.
- Snapshot-based restore is the basis for backups, disaster recovery and moving data between AZs.

## Cleanup

To avoid charges, after the lab I:
1. Terminated both EC2 instances.
2. Deleted both EBS volumes.
3. Deleted the snapshot.
