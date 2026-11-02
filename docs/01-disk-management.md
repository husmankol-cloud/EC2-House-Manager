# Chapter 1: Linux Disk Management

## 1. Project Objective

The objective of this exercise was to develop practical Linux storage administration skills using an Ubuntu server inspection exercise and a safe local virtual-disk laboratory.

The exercise covered:

- Disk capacity inspection.
- Disk and partition identification.
- Filesystem inspection.
- GPT partition table identification.
- Virtual disk creation.
- Filesystem creation and mounting.
- Linux ownership and permissions.
- Storage verification and data persistence.

## 2. House Analogy

Imagine an EC2 instance as a house.

- EC2 instance = The house.
- EBS volume = The storage device attached to the house.
- Disk = The entire storage device.
- Partition = A section of the disk.
- Filesystem = The organisation system used to store files.
- Mount point = The doorway through which Linux accesses a filesystem.

## 3. Checking Disk Capacity

### Command

```bash
df -h
```

### Explanation

- `df` displays filesystem disk space.
- `-h` means human-readable output.
- The command displays filesystem capacity, used space and available space.

### Observed Result

| Filesystem | Size | Used | Available | Usage |
|---|---:|---:|---:|---:|
| /dev/root | 28G | 7.3G | 21G | 26% |
| /dev/nvme0n1p13 | 989M | 163M | 759M | 18% |
| /dev/nvme0n1p15 | 105M | 6.3M | 99M | 7% |

### What I Learned

I learned how to inspect filesystem capacity and identify how much storage is available on a Linux server.

## 4. Inspecting Disks and Partitions

### Command

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

### Explanation

- `lsblk` means List Block Devices.
- `NAME` displays device names.
- `SIZE` displays storage capacity.
- `TYPE` identifies disks, partitions and loop devices.
- `FSTYPE` displays filesystem types.
- `MOUNTPOINTS` shows where filesystems are accessible.

### Observed Disk Layout

| Device | Size | Type | Filesystem | Mount point |
|---|---:|---|---|---|
| nvme0n1 | 30G | Disk | - | - |
| nvme0n1p1 | 28.9G | Partition | ext4 | / |
| nvme0n1p13 | 1023M | Partition | ext4 | /boot |
| nvme0n1p14 | 4M | Partition | - | - |
| nvme0n1p15 | 106M | Partition | vfat | /boot/efi |

### What I Learned

I learned that one virtual disk can contain multiple partitions, each serving a different purpose.

## 5. Inspecting Filesystems

### Command

```bash
lsblk -f
```

### Explanation

The `-f` option displays filesystem information, including:

- Filesystem type.
- Filesystem version.
- Filesystem label.
- UUID.
- Available storage.
- Filesystem usage.
- Mount point.

### Observed Filesystems

| Partition | Filesystem | Label | Mount point |
|---|---|---|---|
| nvme0n1p1 | ext4 | cloudimg-rootfs | / |
| nvme0n1p13 | ext4 | BOOT | /boot |
| nvme0n1p15 | vfat | UEFI | /boot/efi |

### What I Learned

I learned that ext4 is a common Linux filesystem and that vfat is commonly used for EFI system partitions.

I also learned that UUIDs provide unique identifiers for filesystems.

## 6. Inspecting the Partition Table

### Command

```bash
sudo fdisk -l /dev/nvme0n1
```

### Explanation

- `sudo` runs the command with administrator privileges.
- `fdisk` is a Linux disk partitioning utility.
- `-l` lists partition information.
- `/dev/nvme0n1` specifies the disk to inspect.

### Observed Result

```text
Disklabel type: gpt
Disk identifier: <REDACTED>
```

### What I Learned

The inspected EC2 disk uses GPT (GUID Partition Table).

GPT is a modern partition table format commonly used with UEFI-based systems.

## 7. AWS Storage Understanding

The inspected disk was identified as an Amazon Elastic Block Store device.

Amazon EBS provides persistent block storage that can be attached to EC2 instances.

### House Analogy

- EC2 is the house.
- EBS is the storage device attached to the house.
- Partitions are sections of the storage device.
- Filesystems organise the information stored inside those sections.

## 8. Practical Lab: Creating a Virtual Disk

### Objective

To safely practise Linux storage administration without modifying a real disk or the shared EC2 server.

A 100 MB disk-image file was created inside the local project directory.

### Step 1: Create a storage directory

```bash
mkdir -p lab-storage
```

### Step 2: Create a 100 MB disk image

```bash
truncate -s 100M lab-storage/test-disk.img
```

The `truncate` command created a 100 MB image file.

The image may be sparse, meaning its logical size does not necessarily equal the physical storage space initially occupied.

### Step 3: Identify an available loop device

```bash
sudo losetup -f
```

Linux returned an available loop device.

### Step 4: Connect the image to Linux

```bash
sudo losetup --find --show lab-storage/test-disk.img
```

The image was connected as `/dev/loop0`.

A loop device allows Linux to treat an ordinary file as a block device.

### Step 5: Verify the connection

```bash
sudo losetup -l
```

The output confirmed that `/dev/loop0` was associated with the project image file.

### Step 6: Create an ext4 filesystem

```bash
sudo mkfs.ext4 /dev/loop0
```

This created an ext4 filesystem on the verified virtual disk.

**Important:** Formatting destroys existing filesystem information on the selected target. The operation was performed only on the dedicated test image.

### Step 7: Inspect the filesystem

```bash
sudo lsblk -f /dev/loop0
```

The output confirmed the ext4 filesystem and its UUID.

### Step 8: Create a mount-point directory

```bash
mkdir -p lab-storage/mount-point
```

### Step 9: Mount the filesystem

```bash
sudo mount /dev/loop0 lab-storage/mount-point
```

The filesystem became accessible through the mount-point directory.

### Step 10: Verify the mount

```bash
findmnt lab-storage/mount-point
```

The output confirmed:

- Source: `/dev/loop0`
- Filesystem: `ext4`
- Target: The project's mount-point directory.

### Step 11: Inspect available capacity

```bash
df -h lab-storage/mount-point
```

Observed capacity:

| Metric | Result |
|---|---:|
| Filesystem | /dev/loop0 |
| Size | 90M |
| Used | 24K |
| Available | 83M |
| Usage | 1% |

The usable filesystem capacity was smaller than the 100 MB image because ext4 requires space for filesystem metadata and other structures.

## 9. Linux Ownership and Permissions

### Problem Encountered

An attempt to create a file inside the mounted filesystem initially returned:

```text
Permission denied
```

Inspection showed that the mounted filesystem's root directory belonged to `root`.

### Inspection Command

```bash
ls -ld lab-storage/mount-point
```

### Resolution

```bash
sudo chown husmankol:husmankol lab-storage/mount-point
```

This changed the ownership of the test filesystem's root directory to the local user and group.

The operation was restricted to the test mount point.

### Verification

```bash
echo "My first virtual disk test" > lab-storage/mount-point/hello.txt
cat lab-storage/mount-point/hello.txt
```

Observed result:

```text
My first virtual disk test
```

### What I Learned

Linux ownership determines who can access and modify files and directories.

I learned how to investigate a permission error and correct ownership using `chown`.

## 10. Unmounting and Detaching Virtual Storage

After testing file creation and reading, the filesystem was safely unmounted.

### Step 1: Inspect the mount directory

```bash
ls -l lab-storage/mount-point
```

The output showed:

- `hello.txt`
- `lost+found`

`lost+found` is a directory created by ext4 for certain filesystem recovery operations.

### Step 2: Unmount the filesystem

```bash
sudo umount lab-storage/mount-point
```

### Step 3: Verify unmounting

```bash
findmnt lab-storage/mount-point
```

No output confirmed that the filesystem was no longer mounted at that location.

### Step 4: Verify the loop-device association

```bash
sudo losetup -j lab-storage/test-disk.img
```

The output identified `/dev/loop0` as the associated device.

### Step 5: Detach the virtual disk

```bash
sudo losetup -d /dev/loop0
```

### Step 6: Verify detachment

```bash
sudo losetup -j lab-storage/test-disk.img
```

No output confirmed that the image was no longer associated with a loop device.

## 11. Data Persistence Test

The disk image was reconnected using:

```bash
sudo losetup --find --show lab-storage/test-disk.img
```

Filesystem inspection confirmed that the existing ext4 filesystem remained intact.

The filesystem was mounted again, and the original test file was retrieved:

```bash
cat lab-storage/mount-point/hello.txt
```

Observed result:

```text
My first virtual disk test
```

### What I Learned

The file survived the unmounting, detachment and reconnection process.

This demonstrated data persistence in the local virtual-disk lab.

## 12. Lessons Learned

Through these exercises, I learned how to:

1. Check disk capacity using `df -h`.
2. Identify disks and partitions using `lsblk`.
3. Inspect filesystem details using `lsblk -f`.
4. Understand ext4 and vfat filesystems.
5. Inspect partition tables using `fdisk`.
6. Understand GPT partitioning.
7. Create a virtual disk image.
8. Connect an image using a loop device.
9. Create an ext4 filesystem.
10. Mount and unmount filesystems.
11. Inspect and correct directory ownership.
12. Create and retrieve files from mounted storage.
13. Detach and reconnect virtual storage.
14. Verify data persistence.

## 13. Safety Practices

- The virtual-disk practical exercise was performed on a dedicated local image file.
- No real system disk was formatted or partitioned during the local lab.
- The shared EC2 server was used for inspection only.
- Formatting was restricted to the verified loop device backed by the test image.
- Filesystem unmounting was performed before detaching the loop device.
- Sensitive identifiers and server details must be redacted before public publication.

## 14. Future Improvements

Planned practical exercises include:

- Comparing GPT and MBR partition tables.
- Creating and managing partitions on a dedicated test image.
- Practising Linux user and group administration.
- Monitoring CPU, RAM and disk usage.
- Investigating disk-full scenarios.
- Managing dedicated AWS EBS test volumes.
- Automating system checks using Bash.

---

**Project:** EC2 House Manager  
**Chapter:** Linux Disk Management  
**Environment:** Ubuntu on WSL2 for local virtual-disk practice; Ubuntu on AWS EC2 for read-only inspection.

