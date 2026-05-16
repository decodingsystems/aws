# Lab 06: EBS Volume — Create, Attach, Format, and Snapshot

## Objective
Create a standalone EBS volume, attach it to a running EC2 instance, format and mount it as a filesystem, write data, and create a point-in-time snapshot.

## Steps

### Part 1: Launch an EC2 Instance
Launch a basic `t2.micro` Amazon Linux instance in `us-east-1a` (note the AZ).

### Part 2: Create an EBS Volume
1. EC2 → **Volumes** → Create volume
   - Volume type: `gp3`
   - Size: `10 GiB`
   - **Availability Zone: `us-east-1a`** (must match the EC2 instance!)
   - Create.

### Part 3: Attach to EC2
1. Select the new volume → Actions → **Attach volume**
   - Instance: select your running EC2
   - Device name: `/dev/sdf`
   - Attach.

### Part 4: Format and Mount (SSH into EC2)
```bash
# List all available disks
lsblk
# You'll see xvdf (the new volume) without a mount point

# Check if it has a filesystem (it shouldn't yet)
sudo file -s /dev/xvdf

# Format with ext4
sudo mkfs -t ext4 /dev/xvdf

# Create a mount point and mount
sudo mkdir /data
sudo mount /dev/xvdf /data

# Verify
df -h | grep data      # Shows ~9.8G available

# Write a file
echo "important data" | sudo tee /data/testfile.txt
cat /data/testfile.txt
```

### Part 5: Make Mount Persistent (survives reboots)
```bash
# Get the UUID of the device
sudo blkid /dev/xvdf

# Add to /etc/fstab
echo "UUID=<your-uuid>  /data  ext4  defaults,nofail  0  2" | sudo tee -a /etc/fstab

sudo reboot
# After reconnecting:
df -h   # /data still mounted!
```

### Part 6: Create a Snapshot
1. EC2 → Volumes → Select your volume → Actions → **Create snapshot**
   - Description: `my-10gb-data-snapshot`
   - Create.
2. EC2 → **Snapshots** → see your snapshot! You can create a new volume from it in any AZ.

## Cleanup
Unmount (`sudo umount /data`), detach volume in console, delete volume, delete snapshot, terminate EC2.
