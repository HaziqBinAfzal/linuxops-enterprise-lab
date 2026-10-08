# backup01: Rocky Linux Server Baseline

## Server Identity

Hostname: backup01

Operating System: Rocky Linux 9.8 Minimal

Virtualization Platform: Microsoft Hyper-V

Kernel: 5.14.0-687.54.1.el9_8.x86_64

Administrator Account: ruveeha

Administrative Group: wheel

## Hardware and Memory

Virtual CPUs: 2

Configured RAM: 4 GiB

Guest-Visible RAM: Approximately 3.6 GiB

Swap: 2 GiB

Hyper-V Dynamic Memory: Disabled

## Storage Configuration

Virtual Disk Capacity: Approximately 127 GiB

Storage Management: LVM

Root Logical Volume: rlm-root

Root Filesystem: XFS

Root Filesystem Capacity: Approximately 70 GiB

Home Logical Volume: rlm-home

Home Filesystem: XFS

Home Filesystem Capacity: Approximately 53.4 GiB

Boot Filesystem: XFS

EFI Filesystem: VFAT

## Network Configuration

Observed IP Address: 172.17.51.51

Virtual Network: Hyper-V Default Switch

SSH Service: sshd

Remote Access: Verified using Windows OpenSSH

IP Address Assignment: May change

## Administration Verification

The following commands were used to inspect the server configuration:

    hostnamectl
    whoami
    id
    sudo whoami
    sudo -l
    uname -r
    uptime
    free -h
    nproc
    lsblk -f
    df -hT
    systemctl --failed --no-pager

## Verification Results

Rocky Linux packages were updated successfully.

The server was rebooted into the updated kernel.

Administrator privileges were verified using sudo.

Two virtual CPUs were available.

Hyper-V memory configuration was corrected.

The SSH service was verified.

No failed systemd units were detected.

## Learning Outcomes

Verified Rocky Linux operating system information and kernel version.

Inspected LVM logical volumes and XFS filesystems.

Understood the purpose of the wheel administrative group.

Investigated a Hyper-V Dynamic Memory configuration problem.

Verified system memory after troubleshooting.

Distinguished the Rocky Linux virtual machine from Ubuntu WSL.

## Notes

This document records the initial Rocky Linux server baseline.

IP addresses, uptime, and resource utilization may change between checks.
