# web01 — Ubuntu Server Baseline

[← Phase 01](../README.md) · [Setup guide](setup-guide.md) · [Evidence coverage](evidence-coverage.md)

> Historical baseline recorded by Haziq. Exact OS/kernel, account, network, and update details are documented observations; the selected screenshot directly supports the resource and systemd readings. See the evidence coverage record.

## On this page

[Server identity](#server-identity) · [Hardware and storage](#hardware-and-storage) · [Network](#network) · [Administration verification](#administration-verification) · [Results](#results) · [Learning outcomes](#learning-outcomes) · [Notes](#notes) · [Supporting evidence](#supporting-evidence)

## Server identity

| Field | Recorded value |
|---|---|
| Hostname | web01 |
| Operating system | Ubuntu Server 26.04.1 LTS |
| Virtualization | Microsoft Hyper-V |
| Kernel | 7.0.0-38-generic |
| Administrator account | haxz |
| Administrative group | sudo |

## Hardware and storage

| Field | Recorded value |
|---|---|
| Virtual CPUs | 2 |
| Guest-visible RAM | approximately 3.3 GiB |
| Swap | approximately 3.8 GiB |
| Virtual disk | approximately 127 GiB |
| Root filesystem | ext4 on LVM |
| Root filesystem size | approximately 61 GiB |
| Root usage during baseline | approximately 13% |

## Network

| Field | Recorded value |
|---|---|
| Address observed | 172.19.186.83 |
| SSH access | Verified using Windows OpenSSH |
| Network | Hyper-V Default Switch; address may change |

## Administration verification

| Command | Purpose | How to interpret it |
|---|---|---|
| `hostnamectl` | Inspect hostname, OS, kernel, and virtualization. | Confirm the intended VM name and recorded OS. This distinguishes the server VM from Ubuntu WSL. |
| `id` | Inspect UID, primary group, and supplementary groups. | Check the administrator account and sudo/wheel membership; group membership alone does not prove the full sudo policy. |
| `ip -br addr` | Show interface state and assigned addresses concisely. | Use the active VM interface address for SSH. Loopback is not the remote-access address; DHCP addresses can change. |
| `sudo whoami` | Run a small command with elevated privileges. | Successful authorized elevation prints root. A denial or authentication failure requires investigation. |
| `sudo -l` | List the current account's permitted sudo commands. | Inspect the actual policy rather than assuming all group members have unrestricted privileges. |
| `uptime` | Inspect uptime, logged-in sessions, and 1/5/15-minute load averages. | Interpret load with CPU count and workload context. A snapshot alone is not a performance diagnosis. |
| `free -h` | Display memory and swap in human-readable units. | Compare guest total with configured RAM and inspect available memory; free memory alone excludes reclaimable caches. |
| `nproc` | Print processing units available to this process. | The recorded result is 2; this is available CPU capacity, not a full hardware inventory. |
| `df -hT` | Show mounted filesystem capacity, usage, and filesystem type. | Inspect the root mount and available space. This does not show all unallocated disk or volume-group space. |
| `systemctl --failed --no-pager` | List units in the failed state without opening a pager. | 0 loaded units listed means no units are currently failed; it does not prove that all applications are healthy. |

These commands inspect state; they do not install, update, or configure the server. Use the setup guide for a new lab build.

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

## Supporting evidence

[Server health screenshot](../evidence/haziq-web01-health.png) shows the CPU count, memory, swap, mounted filesystems, root usage, and failed-unit check. It does not show every field in this document.

---

[← Phase 01 overview](../README.md) · [Project overview](../../../README.md) · [Evidence index](../evidence/README.md)
