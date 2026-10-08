# web01 — Ubuntu Server Baseline

## Server identity

- Hostname: web01
- Operating system: Ubuntu Server 26.04.1 LTS
- Virtualization: Microsoft Hyper-V
- Kernel: 7.0.0-38-generic
- Administrator account: haxz
- Administrative group: sudo

## Hardware and storage

- Virtual CPUs: 2
- Guest-visible RAM: approximately 3.3 GiB
- Swap: approximately 3.8 GiB
- Virtual disk: approximately 127 GiB
- Root filesystem: ext4 on LVM
- Root filesystem size: approximately 61 GiB
- Root usage during baseline: approximately 13%

## Network

- Address observed: 172.19.186.83
- SSH access: Verified using Windows OpenSSH
- Network: Hyper-V Default Switch; address may change

## Administration verification

Commands used:

    hostnamectl
    id
    ip -br addr
    sudo whoami
    sudo -l
    uptime
    free -h
    nproc
    df -hT
    systemctl --failed --no-pager

## Results

- Administrator privileges verified.
- Package updates completed.
- Two virtual CPUs available.
- Root filesystem has free space.
- No failed systemd units detected.

## Learning outcomes

- Verified Linux distribution and kernel.
- Checked sudo privileges and server health.
- Inspected CPU, memory, swap, disk, and network.
- Distinguished WSL from the Hyper-V server VM.

## Notes

This is a baseline snapshot. IP addresses and resource usage may change.
