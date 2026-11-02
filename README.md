# EC2 House Manager – Linux & AWS EC2 System Administration Lab

## Project Overview

EC2 House Manager is a hands-on Linux system administration and AWS EC2 learning project.

It demonstrates practical server inspection, Linux storage management, filesystem operations, permissions, and safe troubleshooting.

The project combines read-only inspection of an Ubuntu EC2 server with hands-on storage exercises using a local virtual disk in Ubuntu on WSL2.

## Learning Objectives

- Understand Linux disks and partitions.
- Inspect filesystems and mount points.
- Monitor disk capacity.
- Create and manage a safe virtual disk.
- Practise mounting and unmounting filesystems.
- Understand Linux ownership and permissions.
- Learn AWS EC2 and EBS storage concepts.
- Develop safe server inspection and troubleshooting habits.

## Project Structure

```text
ec2-house-manager/
├── README.md
├── .gitignore
├── docs/
│   └── 01-disk-management.md
└── lab-storage/
    ├── mount-point/
    └── test-disk.img  (excluded from Git)
```

## Practical Learning Progress

| Exercise | Status |
|---|---|
| Disk capacity inspection using df | Completed |
| Disk inspection using lsblk | Completed |
| Filesystem inspection using lsblk -f | Completed |
| GPT partition table identification | Completed |
| Local virtual disk creation | Completed |
| ext4 filesystem creation | Completed |
| Filesystem mounting and unmounting | Completed |
| Linux ownership and permissions | Completed |
| Virtual disk data persistence test | Completed |
| GPT and MBR comparison | Pending |
| RAM and CPU monitoring | Pending |
| Linux user and group administration | Pending |
| Linux networking | Pending |
| System service management | Pending |
| Dedicated AWS EBS test-volume operations | Pending |

## Tools and Technologies

- Ubuntu Linux
- Ubuntu on WSL2
- AWS EC2
- Amazon Elastic Block Store (EBS)
- Linux command line
- Git and GitHub

## Learning Method

Each exercise aims to include:

1. Technical explanation.
2. Beginner-friendly explanation.
3. Practical house analogy.
4. Linux commands used.
5. Sanitised command output.
6. Lessons learned.

## Safety and Security

- EC2 server inspection is performed with appropriate permission.
- Destructive storage operations are restricted to dedicated test storage.
- The local virtual disk image is excluded from Git.
- Private keys, passwords, access tokens and sensitive server details must never be published.

## Project Goal

Develop practical Linux and AWS system administration skills through hands-on exercises, troubleshooting, technical documentation and reproducible demonstrations.

