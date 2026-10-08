# backup01 — Rocky Linux Server Baseline

[← Phase 01](../README.md) · [Setup guide](setup-guide.md) · [Evidence coverage](evidence-coverage.md)

> Historical baseline recorded by Ruveeha. OS/kernel, account, storage, SSH, and package-update details are documented observations; selected screenshots directly support the memory investigation and post-fix health checks.

## On this page

[Server identity](#server-identity) · [Hardware and memory](#hardware-and-memory) · [Storage configuration](#storage-configuration) · [Network](#network) · [Administration verification](#administration-verification) · [Results](#results) · [Learning outcomes](#learning-outcomes) · [Supporting evidence](#supporting-evidence) · [Notes](#notes)

## Server identity

| Field | Recorded value |
|---|---|
| Hostname | `backup01` |
| Operating system | Rocky Linux 9.8 Minimal |
| Virtualization | Microsoft Hyper-V |
| Kernel | `5.14.0-687.54.1.el9_8.x86_64` |
| Administrator account | `ruveeha` |
| Administrative group | `wheel` |

## Hardware and memory

| Field | Recorded value |
|---|---|
| Virtual CPUs | 2 |
| Configured RAM | 4 GiB |
| Guest-visible RAM after the fix | Approximately 3.6 GiB |
| Swap | 2 GiB |
| Hyper-V Dynamic Memory after the fix | Disabled |

## Storage configuration

| Field | Recorded value |
|---|---|
| Virtual disk capacity | Approximately 127 GiB |
| Storage management | LVM |
| Root logical volume | `rlm-root` |
| Root filesystem | XFS; approximately 70 GiB |
| Home logical volume | `rlm-home` |
| Home filesystem | XFS; approximately 53.4 GiB |
| Boot filesystem | XFS |
| EFI filesystem | VFAT |

These storage values come from the existing baseline record; the six selected screenshots do not independently show the full storage layout.

## Network

- Observed address: `172.17.51.51`.
- Virtual network: Hyper-V Default Switch.
- SSH service: `sshd`.
- Remote access: recorded as verified using Windows OpenSSH.
- Address assignment may change.

## Administration verification

| Command | Purpose | How to interpret it |
|---|---|---|
| `hostnamectl` | Inspect hostname, OS, kernel, and virtualization. | Confirm the intended VM name and recorded OS. This distinguishes the server VM from Ubuntu WSL. |
| `whoami` | Print the current effective username. | Expect the normal administrator account before using sudo; this alone does not prove sudo access. |
| `id` | Inspect UID, primary group, and supplementary groups. | Check the administrator account and sudo/wheel membership; group membership alone does not prove the full sudo policy. |
| `sudo whoami` | Run a small command with elevated privileges. | Successful authorized elevation prints root. A denial or authentication failure requires investigation. |
| `sudo -l` | List the current account's permitted sudo commands. | Inspect the actual policy rather than assuming all group members have unrestricted privileges. |
| `uname -r` | Print the running kernel release. | Compare with the recorded baseline after reboot; an installed kernel may differ from the running one. |
| `uptime` | Inspect uptime, logged-in sessions, and 1/5/15-minute load averages. | Interpret load with CPU count and workload context. A snapshot alone is not a performance diagnosis. |
| `free -h` | Display memory and swap in human-readable units. | Compare guest total with configured RAM and inspect available memory; free memory alone excludes reclaimable caches. |
| `nproc` | Print processing units available to this process. | The recorded result is 2; this is available CPU capacity, not a full hardware inventory. |
| `lsblk -f` | Inspect block devices, filesystem types, UUIDs, and mount points. | Map the guest disk and logical volumes; do not confuse whole-disk capacity with root filesystem size. |
| `df -hT` | Show mounted filesystem capacity, usage, and filesystem type. | Inspect the root mount and available space. This does not show all unallocated disk or volume-group space. |
| `systemctl --failed --no-pager` | List units in the failed state without opening a pager. | 0 loaded units listed means no units are currently failed; it does not prove that all applications are healthy. |

These commands inspect state; package updates and SSH setup require separate actions. See the setup guide for a new lab build.

## Results

- Package updates and reboot into the updated kernel were recorded.
- sudo access, SSH access, and two virtual CPUs were recorded as verified.
- Hyper-V memory configuration was corrected.
- Guest-visible RAM returned to approximately 3.6 GiB.
- No failed systemd units were listed during post-fix verification.

## Learning outcomes

- Distinguished the Rocky Linux VM from Ubuntu WSL.
- Inspected LVM logical volumes and XFS filesystems.
- Reviewed the wheel group and actual sudo permissions.
- Compared host memory configuration with guest observations.
- Verified state after a configuration change.

## Supporting evidence

- [Memory before the fix](../evidence/ruveeha-memory-before.png).
- [Host configuration before the fix](../evidence/ruveeha-hyperv-before.png).
- [Host configuration after the fix](../evidence/ruveeha-hyperv-after.png).
- [Guest memory and service health after the fix](../evidence/ruveeha-memory-after.png).
- [Detailed incident report](memory-troubleshooting.md).

## Notes

This is a point-in-time baseline. IP addresses, kernel versions, uptime, and utilization may change. Additional dated verification output would be needed to independently corroborate the fields outside the selected evidence.

---

[← Phase 01 overview](../README.md) · [Project overview](../../../README.md) · [Evidence index](../evidence/README.md)
