# Phase 01 — Setup and Verification Guide

[← Phase 01](../README.md) · [Evidence coverage](evidence-coverage.md) · [Ubuntu baseline](web01-baseline.md) · [Rocky baseline](backup01-baseline.md)

## On this page

[Quick start](#quick-start-for-readers) · [Scope and provenance](#scope-and-provenance) · [1. Prepare each Windows host](#1-prepare-each-windows-host) · [2. Create the new VM in Hyper-V Manager](#2-create-the-new-vm-in-hyper-v-manager) · [3. Install the guest OS](#3-install-the-guest-os) · [4. Update the new VM](#4-update-the-new-vm) · [5. Establish SSH access](#5-establish-ssh-access) · [6. Verify resources and service health](#6-verify-resources-and-service-health) · [7. Record a reviewable result](#7-record-a-reviewable-result) · [Official references](#official-references)

## Scope and provenance

This is a manual reconstruction guide for a **new, disposable lab**. It is not a transcript of the original setup and does not claim that every command below was previously executed.

Historical baselines record Ubuntu Server 26.04.1 LTS and Rocky Linux 9.8 Minimal. Exact ISO names/checksums, VM generation, original installer partitioning choices, and update transcripts were not preserved in the selected evidence. Record these during a new build. Current installation media and updates may produce different versions and layouts.

The guide reproduces the baseline workflow, not an identical disk image. Existing working VMs do not need to be rebuilt to use the read-only verification sections.

## Quick start for readers

**Start with a disposable VM.** Do not run installer or disk-partitioning steps against a Windows host disk, an existing VM with important data, or a production server. The walkthrough is designed for Windows with Hyper-V; users on another hypervisor must adapt the VM creation steps.

1. Read [Phase 01 overview](../README.md) to understand the goal and the two reference servers.
2. Follow sections 1–5 below to prepare Hyper-V, create **one** new VM, install your chosen Linux distribution, update it, and test SSH. You can choose Ubuntu **or** Rocky Linux; you do not need both to start.
3. Run the read-only verification commands in section 6 **inside your Linux VM**, compare the output with the acceptance criteria, and investigate discrepancies.
4. Record **your own** values and deviations using the [fresh-build validation template](fresh-build-validation.md). Never copy the historical screenshot values as if they were your results.
5. Only after reviewing the output, optionally share your results through your own GitHub fork or a collaborator branch.

### Which terminal runs which commands?

| Terminal | Used for | Example |
|---|---|---|
| **Windows PowerShell (host)** | Check the ISO, configure or inspect Hyper-V, connect using Windows SSH | `Get-FileHash`, `ssh` |
| **Ubuntu VM terminal** | Ubuntu installation verification, packages, SSH service, health | `sudo apt update`, `systemctl status ssh` |
| **Rocky Linux VM terminal** | Rocky installation verification, packages, SSH service, health | `sudo dnf upgrade`, `systemctl status sshd` |
| **Ubuntu WSL terminal (optional)** | Git clone, branch, commit, push, pull request | `git status`, `git push` |

Commands marked for one Linux distribution must **not** be run on the other distribution without adaptation. Lines beginning with `sudo` can change system state; inspect the action before execution.

### Quick verification after installation

Run this block **inside your newly installed Linux VM**, not WSL:

```bash
hostnamectl
whoami
id
nproc
free -h
lsblk -f
df -hT
systemctl --failed --no-pager
```

| Command | What to look for |
|---|---|
| `hostnamectl` | Your intended VM hostname, Linux distribution, and kernel |
| `whoami` and `id` | The account you created and its UID/group memberships |
| `nproc` | Virtual CPU count available to the guest |
| `free -h` | Guest-visible RAM and swap; compare with Hyper-V settings |
| `lsblk -f` | Disk, partition, filesystem, and mount information |
| `df -hT` | Mounted filesystem types, capacity, and free space |
| `systemctl --failed --no-pager` | Failed systemd units; zero is the intended baseline, but investigate any failures |

These are **inspection commands**, not a complete security audit. The outputs will vary by installation, distribution, and VM settings. Record the actual results.

## 1. Prepare each Windows host

Use a Windows edition that supports Hyper-V, hardware virtualization enabled in firmware, and sufficient free RAM and disk space. Enable Hyper-V through **Turn Windows features on or off**, restart if requested, then open Hyper-V Manager. Ubuntu WSL is the Git workspace; it is separate from the server VM.

| Item | Haziq | Ruveeha |
|---|---|---|
| Server VM name | `web01` | `backup01` |
| Guest OS | Ubuntu Server | Rocky Linux Minimal |
| Guest administrator | `haxz` | `ruveeha` |
| CPU allocation for this reconstruction | 2 virtual processors | 2 virtual processors |
| RAM allocation for this reconstruction | Fixed 4 GiB | Fixed 4 GiB |
| New virtual disk for this reconstruction | 128 GiB dynamically expanding VHDX | 128 GiB dynamically expanding VHDX |
| Virtual switch | Local Hyper-V Default Switch | Local Hyper-V Default Switch |

These are reconstruction choices, not independently proven original host settings. A dynamically expanding disk grows as data is written; monitor host free space.

Download the selected installer ISO from [Ubuntu](https://ubuntu.com/download/server) or [Rocky Linux](https://rockylinux.org/download). Verify it using the distribution's published checksum/signature instructions. Record the exact filename, version, checksum, download source, and date.

### Check the downloaded ISO — Windows PowerShell

Obtain the expected SHA-256 value from the distribution's official checksum file for the **exact ISO filename**. Where a signed checksum file is supplied, follow the distribution's signature-verification instructions to authenticate that file; a hash comparison alone cannot authenticate an untrusted download source.

```powershell
$labIsoPath = Read-Host "Full path to the downloaded ISO"
$labExpectedHash = (Read-Host "Official SHA-256 for that exact ISO").Trim()
if ($labExpectedHash -notmatch '^[0-9a-fA-F]{64}$') {
    throw "Expected SHA-256 must contain exactly 64 hexadecimal characters."
}
$labActualHash = (Get-FileHash -LiteralPath $labIsoPath -Algorithm SHA256).Hash
if ($labActualHash -ine $labExpectedHash) {
    throw "ISO checksum mismatch. Do not use this ISO."
}
"SHA-256 matches the supplied official checksum."
```

| Step | Purpose |
|---|---|
| `Read-Host` | Collect the local ISO path and expected checksum without hard-coded personal paths |
| Hash format check | Reject a malformed expected SHA-256 |
| `Get-FileHash` | Calculate the downloaded file's SHA-256 |
| Case-insensitive comparison | Stop on a mismatch; matching case is irrelevant for hexadecimal hashes |

Record the ISO filename, actual checksum, official checksum source, and verification date in the [fresh-build validation record](fresh-build-validation.md).

## 2. Create the new VM in Hyper-V Manager

1. Select **New → Virtual Machine**. Use the VM name from the table and a storage location with enough free space.
2. For this reconstruction, choose **Generation 2** with compatible 64-bit installation media. The original VM generation is not established by the current evidence.
3. Assign 4096 MB startup memory and leave **Dynamic Memory disabled**.
4. Select the **Default Switch** for the network adapter.
5. Create a new 128 GiB VHDX. Do not attach a physical Windows disk or an existing disk containing important data.
6. Attach the verified ISO as installation media.
7. Open VM settings and set **2 virtual processors**.
8. For Linux Secure Boot, use **Microsoft UEFI Certificate Authority** when supported by the selected media. Investigate incompatible media/settings rather than assuming the original lab used the same configuration.
9. Start the VM and connect through the Hyper-V console.

Expected outcome: the installer boots and sees only the intended new virtual disk.

## 3. Install the guest OS

### Haziq — Ubuntu Server

Choose the language and keyboard, enable the VM network interface, and use the appropriate default package mirror. Install to the new virtual disk; select the installer LVM option if reproducing the baseline's storage approach. Set the hostname to `web01` and create `haxz` as the normal administrative account. Select OpenSSH server when offered. Finish installation, detach the ISO, and reboot into the installed disk.

### Ruveeha — Rocky Linux

Choose **Minimal Install**, enable the network interface, set hostname `backup01`, and select only the new virtual disk as the installation destination. Use an LVM layout with XFS for root/home if reproducing the recorded storage approach; review and record the actual allocation rather than assuming automatic partitioning matches the old baseline. Create `ruveeha` and select the installer option to make the user an administrator. Finish installation, detach the ISO, and reboot.

### Record the installer storage decision

Before accepting partitioning, record the selected virtual disk and proposed mount layout. The procedure deliberately does not prescribe the historical 70 GiB/53.4 GiB Rocky split because the original partitioning transcript is unavailable.

| Guest | Required layout characteristic | What to record |
|---|---|---|
| Ubuntu | Root on LVM with ext4 when following this lab's storage approach | EFI/boot partitions, VG/LV names, root size, and unallocated VG space |
| Rocky | Root/home on LVM with XFS when following this lab's storage approach | EFI/boot partitions, VG/LV names, root/home sizes, and swap |
| Both | Only the new disposable VHDX is selected | Installer's final storage summary before writing changes |

A different layout can still support the learning exercises, but must be documented as a deviation. After installation, verify it with `lsblk -f`, `df -hT`, and, where LVM is used, `sudo pvs`, `sudo vgs`, and `sudo lvs`. The latter three inspect physical volumes, volume groups, and logical volumes respectively.

Passwords are entered interactively. Do not put them into commands, repository files, or screenshots.

### Verify installation identity — inside each Linux VM

| Command | Purpose | Acceptance criterion |
|---|---|---|
| `hostnamectl` | Inspect host, OS, kernel, and virtualization | Intended hostname and chosen OS are displayed; record the actual version |
| `whoami` | Identify the current user | Haziq sees haxz; Ruveeha sees ruveeha |
| `id` | Inspect group membership | Expected administrator group is present |
| `sudo whoami` | Test effective administrative access | Authorized elevation prints root |
| `sudo -l` | Inspect the effective sudo policy | The policy permits the intended administration work |

If sudo is denied, use the VM console and the installation's administrator access to diagnose account/group/policy configuration. Do not assume group membership alone proves permission.

## 4. Update the new VM

Run only the commands for the correct distribution. Review each package transaction before accepting it.

### Haziq — Ubuntu guest

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo apt update` | Refresh package metadata; resolve repository/signature errors before upgrading |
| `sudo apt upgrade` | Apply available upgrades after reviewing the transaction; record successful completion or any deferred packages |
| `sudo reboot` | Restart the guest; the SSH session closes and must be re-established |

### Ruveeha — Rocky guest

```bash
sudo dnf upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo dnf upgrade` | Apply updates from enabled repositories after reviewing the transaction; record success or errors |
| `sudo reboot` | Restart the guest so a newly installed kernel can become active |

After reconnecting, use `uname -r` to record the running kernel and `uptime` to inspect uptime. Their outputs do not independently prove that the package transaction succeeded; preserve its outcome separately.

## 5. Establish SSH access

Run server commands in the Linux VM console. Keep console access available while testing SSH.

### Haziq — Ubuntu guest

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
```

The first command installs the SSH server if absent. The second enables boot startup and starts the service. The status command checks service state. On Ubuntu installations using socket activation, also inspect `systemctl status ssh.socket --no-pager`; an active socket may start the daemon on demand.

### Ruveeha — Rocky guest

```bash
sudo dnf install openssh-server
sudo systemctl enable --now sshd
systemctl status sshd --no-pager
```

The first command installs the package, the second enables/starts the daemon, and the third checks its state. Expect an active service or investigate the status and logs.

### Both guests — address and listener checks

| Command | Purpose and expected result |
|---|---|
| `ip -br addr` | Identify the active VM interface and current IP address |
| `sudo ss -lntp` | Inspect TCP listening sockets and owning processes; check SSH's configured port, normally 22 |
| `sudo journalctl -u ssh -n 30 --no-pager` | Ubuntu service diagnostics if needed |
| `sudo journalctl -u sshd -n 30 --no-pager` | Rocky service diagnostics if needed |

Inspect the existing firewall before changing it. On Ubuntu, `sudo ufw status` reports UFW state. If active and blocking the intended lab connection, `sudo ufw allow 22/tcp` permits the default SSH port. This modifies a rule; it does not itself enable UFW.

On Rocky, `sudo firewall-cmd --state` checks whether firewalld is running and `sudo firewall-cmd --get-active-zones` identifies the guest interface's zone. If running and SSH is blocked, use that actual zone:

```bash
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --permanent --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --query-service=ssh
```

Replace `YOUR_ACTIVE_ZONE` before execution. The first command changes runtime policy, the second persists it, and the third verifies runtime permission. This is new-build guidance, not evidence that firewall changes occurred in the original lab. Review restrictions and hardening in Phase 03.

### Windows PowerShell — connect from the same host

Replace the example addresses with each VM's current IP:

```powershell
ssh haxz@YOUR_WEB01_IP
ssh ruveeha@YOUR_BACKUP01_IP
```

Haziq runs the first command on Haziq's host; Ruveeha runs the second on Ruveeha's host. Each invokes the Windows SSH client and authenticates to the local guest. Verify a new host key against the guest console before accepting it. For example, `sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` prints the guest's public host-key fingerprint for comparison.

After login, `hostnamectl` and `whoami` must identify the intended guest/account. Do not blindly remove known-host entries when a key changes. These tests do not establish connectivity between Haziq's and Ruveeha's separate VM networks.

## 6. Verify resources and service health

The two baseline documents explain each inspection command. Run the relevant commands, record actual output, and compare it with the intended new-build settings.

| Check | Command | Acceptance criterion |
|---|---|---|
| CPU capacity | `nproc` | 2 processing units available for this reconstruction |
| Memory and swap | `free -h` | Guest memory is consistent with the configured allocation after overhead; investigate a large shortfall |
| Disk layout | `lsblk -f` | Expected new disk, logical volumes, filesystem types, and mounts are present |
| Filesystem space | `df -hT` | Root is mounted with the intended filesystem and adequate free space |
| Failed units | `systemctl --failed --no-pager` | Zero failed units, or each failure is explicitly investigated |
| Current kernel | `uname -r` | Running kernel recorded after reboot |
| Load and uptime | `uptime` | Snapshot recorded and interpreted with CPU/workload context |

A 4 GiB Hyper-V allocation does not necessarily appear as exactly 4.0 GiB in the guest. Use the [memory incident report](memory-troubleshooting.md) for the observed 609 MiB failure and its verified resolution.

## 7. Record a reviewable result

Record the build date, installer provenance, VM settings, actual OS/kernel, account, storage layout, SSH verification, package transaction outcome, and resource checks. Explain deviations from the historical baseline rather than editing old observations to match a new run.

Choose important evidence: one clear baseline health capture per server, meaningful troubleshooting before/after, and the reviewed PR. Use existing screenshots only for the historical observations they actually show. Redact credentials, keys, tokens, and confidential details.

### Git handoff — Ubuntu WSL

Use Ubuntu WSL for these Git operations, not Windows PowerShell or the server guest. This workflow assumes Git is installed and the contributor's GitHub SSH authentication already works. Never commit an SSH private key.

1. **Haziq:** use the existing checkout of `HaziqBinAfzal/linuxops-enterprise-lab`. For a new checkout, clone the repository:

   ```bash
   git clone git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   `git clone` downloads the repository and sets `origin`; `cd` enters it.

2. **Ruveeha:** as an existing collaborator, use her existing checkout of the canonical repository. A fork is optional, not required. For a new checkout, clone the canonical repository:

   ```bash
   git clone git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   Confirm that `origin` points to the canonical repository before pushing. A reader without write access should fork the repository, clone their own fork, and open a pull request from it.

3. Inspect the checkout before changing branches:

   ```bash
   git status
   git remote -v
   git branch --show-current
   ```

   These show pending edits, remote destinations, and the current branch. Preserve existing work before switching branches.

4. Fetch main from the canonical repository and create a **new validation branch**, so a fresh-build result stays distinct from the historical baseline records:

   ```bash
   git fetch git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git main
   git switch -c YOUR_NEW_VALIDATION_BRANCH FETCH_HEAD
   ```

   Replace `YOUR_NEW_VALIDATION_BRANCH` with a new contributor-specific name such as `haziq/phase-01-fresh-build-validation` or `ruveeha/phase-01-fresh-build-validation`. Fetch reads canonical main; switch creates a new local branch at that fetched commit.

5. After performing the new build, edit the validation record with actual observations and review the changes:

   ```bash
   git diff
   git add phases/phase-01/docs/fresh-build-validation.md
   git diff --cached
   git commit -m "docs: record verified Phase 01 fresh build"
   git push -u origin HEAD
   ```

   `git diff` reviews unstaged edits; `git add` stages the named record; `git diff --cached` shows exactly what will be committed; `git commit` saves the reviewed change; `git push -u origin HEAD` publishes the current branch to the already-confirmed origin and sets its upstream. Add any new evidence files individually after reviewing them.

6. Open a PR to the canonical repository's appropriate base branch, explain which build was executed, and request the other contributor's review. Confirm the base/head selection in GitHub. Keep the validation change scoped to the new build and its actual observations.

The validation record starts **Not executed**. Creating a branch or PR is not evidence that the build passed.

## Official references

- [Microsoft: Generation 1 and 2 VM guidance](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
- [Ubuntu: OpenSSH server installation](https://help.ubuntu.com/community/SSH)
- [Rocky Linux: DNF package management](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)
- [Rocky Linux: firewalld guide](https://docs.rockylinux.org/guides/security/firewalld-beginners/)

Follow the documentation for the installed release. This guide has been reviewed as documentation; its new-build procedure has not been executed against the participants' laptops in this change.
---

[← Phase 01 overview](../README.md) · [Project overview](../../../README.md) · [Evidence index](../evidence/README.md)
) {
    throw "Expected SHA-256 must contain exactly 64 hexadecimal characters."
}
$labActualHash = (Get-FileHash -LiteralPath $labIsoPath -Algorithm SHA256).Hash
if ($labActualHash -ine $labExpectedHash) {
    throw "ISO checksum mismatch. Do not use this ISO."
}
"SHA-256 matches the supplied official checksum."
```

| Step | Purpose |
|---|---|
| `Read-Host` | Collect the local ISO path and expected checksum without hard-coded personal paths |
| Hash format check | Reject a malformed expected SHA-256 |
| `Get-FileHash` | Calculate the downloaded file's SHA-256 |
| Case-insensitive comparison | Stop on a mismatch; matching case is irrelevant for hexadecimal hashes |

Record the ISO filename, actual checksum, official checksum source, and verification date in the [fresh-build validation record](fresh-build-validation.md).

## 2. Create the new VM in Hyper-V Manager

1. Select **New → Virtual Machine**. Use the VM name from the table and a storage location with enough free space.
2. For this reconstruction, choose **Generation 2** with compatible 64-bit installation media. The original VM generation is not established by the current evidence.
3. Assign 4096 MB startup memory and leave **Dynamic Memory disabled**.
4. Select the **Default Switch** for the network adapter.
5. Create a new 128 GiB VHDX. Do not attach a physical Windows disk or an existing disk containing important data.
6. Attach the verified ISO as installation media.
7. Open VM settings and set **2 virtual processors**.
8. For Linux Secure Boot, use **Microsoft UEFI Certificate Authority** when supported by the selected media. Investigate incompatible media/settings rather than assuming the original lab used the same configuration.
9. Start the VM and connect through the Hyper-V console.

Expected outcome: the installer boots and sees only the intended new virtual disk.

## 3. Install the guest OS

### Haziq — Ubuntu Server

Choose the language and keyboard, enable the VM network interface, and use the appropriate default package mirror. Install to the new virtual disk; select the installer LVM option if reproducing the baseline's storage approach. Set the hostname to `web01` and create `haxz` as the normal administrative account. Select OpenSSH server when offered. Finish installation, detach the ISO, and reboot into the installed disk.

### Ruveeha — Rocky Linux

Choose **Minimal Install**, enable the network interface, set hostname `backup01`, and select only the new virtual disk as the installation destination. Use an LVM layout with XFS for root/home if reproducing the recorded storage approach; review and record the actual allocation rather than assuming automatic partitioning matches the old baseline. Create `ruveeha` and select the installer option to make the user an administrator. Finish installation, detach the ISO, and reboot.

### Record the installer storage decision

Before accepting partitioning, record the selected virtual disk and proposed mount layout. The procedure deliberately does not prescribe the historical 70 GiB/53.4 GiB Rocky split because the original partitioning transcript is unavailable.

| Guest | Required layout characteristic | What to record |
|---|---|---|
| Ubuntu | Root on LVM with ext4 when following this lab's storage approach | EFI/boot partitions, VG/LV names, root size, and unallocated VG space |
| Rocky | Root/home on LVM with XFS when following this lab's storage approach | EFI/boot partitions, VG/LV names, root/home sizes, and swap |
| Both | Only the new disposable VHDX is selected | Installer's final storage summary before writing changes |

A different layout can still support the learning exercises, but must be documented as a deviation. After installation, verify it with `lsblk -f`, `df -hT`, and, where LVM is used, `sudo pvs`, `sudo vgs`, and `sudo lvs`. The latter three inspect physical volumes, volume groups, and logical volumes respectively.

Passwords are entered interactively. Do not put them into commands, repository files, or screenshots.

### Verify installation identity — inside each Linux VM

| Command | Purpose | Acceptance criterion |
|---|---|---|
| `hostnamectl` | Inspect host, OS, kernel, and virtualization | Intended hostname and chosen OS are displayed; record the actual version |
| `whoami` | Identify the current user | Haziq sees haxz; Ruveeha sees ruveeha |
| `id` | Inspect group membership | Expected administrator group is present |
| `sudo whoami` | Test effective administrative access | Authorized elevation prints root |
| `sudo -l` | Inspect the effective sudo policy | The policy permits the intended administration work |

If sudo is denied, use the VM console and the installation's administrator access to diagnose account/group/policy configuration. Do not assume group membership alone proves permission.

## 4. Update the new VM

Run only the commands for the correct distribution. Review each package transaction before accepting it.

### Haziq — Ubuntu guest

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo apt update` | Refresh package metadata; resolve repository/signature errors before upgrading |
| `sudo apt upgrade` | Apply available upgrades after reviewing the transaction; record successful completion or any deferred packages |
| `sudo reboot` | Restart the guest; the SSH session closes and must be re-established |

### Ruveeha — Rocky guest

```bash
sudo dnf upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo dnf upgrade` | Apply updates from enabled repositories after reviewing the transaction; record success or errors |
| `sudo reboot` | Restart the guest so a newly installed kernel can become active |

After reconnecting, use `uname -r` to record the running kernel and `uptime` to inspect uptime. Their outputs do not independently prove that the package transaction succeeded; preserve its outcome separately.

## 5. Establish SSH access

Run server commands in the Linux VM console. Keep console access available while testing SSH.

### Haziq — Ubuntu guest

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
```

The first command installs the SSH server if absent. The second enables boot startup and starts the service. The status command checks service state. On Ubuntu installations using socket activation, also inspect `systemctl status ssh.socket --no-pager`; an active socket may start the daemon on demand.

### Ruveeha — Rocky guest

```bash
sudo dnf install openssh-server
sudo systemctl enable --now sshd
systemctl status sshd --no-pager
```

The first command installs the package, the second enables/starts the daemon, and the third checks its state. Expect an active service or investigate the status and logs.

### Both guests — address and listener checks

| Command | Purpose and expected result |
|---|---|
| `ip -br addr` | Identify the active VM interface and current IP address |
| `sudo ss -lntp` | Inspect TCP listening sockets and owning processes; check SSH's configured port, normally 22 |
| `sudo journalctl -u ssh -n 30 --no-pager` | Ubuntu service diagnostics if needed |
| `sudo journalctl -u sshd -n 30 --no-pager` | Rocky service diagnostics if needed |

Inspect the existing firewall before changing it. On Ubuntu, `sudo ufw status` reports UFW state. If active and blocking the intended lab connection, `sudo ufw allow 22/tcp` permits the default SSH port. This modifies a rule; it does not itself enable UFW.

On Rocky, `sudo firewall-cmd --state` checks whether firewalld is running and `sudo firewall-cmd --get-active-zones` identifies the guest interface's zone. If running and SSH is blocked, use that actual zone:

```bash
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --permanent --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --query-service=ssh
```

Replace `YOUR_ACTIVE_ZONE` before execution. The first command changes runtime policy, the second persists it, and the third verifies runtime permission. This is new-build guidance, not evidence that firewall changes occurred in the original lab. Review restrictions and hardening in Phase 03.

### Windows PowerShell — connect from the same host

Replace the example addresses with each VM's current IP:

```powershell
ssh haxz@YOUR_WEB01_IP
ssh ruveeha@YOUR_BACKUP01_IP
```

Haziq runs the first command on Haziq's host; Ruveeha runs the second on Ruveeha's host. Each invokes the Windows SSH client and authenticates to the local guest. Verify a new host key against the guest console before accepting it. For example, `sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` prints the guest's public host-key fingerprint for comparison.

After login, `hostnamectl` and `whoami` must identify the intended guest/account. Do not blindly remove known-host entries when a key changes. These tests do not establish connectivity between Haziq's and Ruveeha's separate VM networks.

## 6. Verify resources and service health

The two baseline documents explain each inspection command. Run the relevant commands, record actual output, and compare it with the intended new-build settings.

| Check | Command | Acceptance criterion |
|---|---|---|
| CPU capacity | `nproc` | 2 processing units available for this reconstruction |
| Memory and swap | `free -h` | Guest memory is consistent with the configured allocation after overhead; investigate a large shortfall |
| Disk layout | `lsblk -f` | Expected new disk, logical volumes, filesystem types, and mounts are present |
| Filesystem space | `df -hT` | Root is mounted with the intended filesystem and adequate free space |
| Failed units | `systemctl --failed --no-pager` | Zero failed units, or each failure is explicitly investigated |
| Current kernel | `uname -r` | Running kernel recorded after reboot |
| Load and uptime | `uptime` | Snapshot recorded and interpreted with CPU/workload context |

A 4 GiB Hyper-V allocation does not necessarily appear as exactly 4.0 GiB in the guest. Use the [memory incident report](memory-troubleshooting.md) for the observed 609 MiB failure and its verified resolution.

## 7. Record a reviewable result

Record the build date, installer provenance, VM settings, actual OS/kernel, account, storage layout, SSH verification, package transaction outcome, and resource checks. Explain deviations from the historical baseline rather than editing old observations to match a new run.

Choose important evidence: one clear baseline health capture per server, meaningful troubleshooting before/after, and the reviewed PR. Use existing screenshots only for the historical observations they actually show. Redact credentials, keys, tokens, and confidential details.

### Git handoff — Ubuntu WSL

Use Ubuntu WSL for these Git operations, not Windows PowerShell or the server guest. This workflow assumes Git is installed and the contributor's GitHub SSH authentication already works. Never commit an SSH private key.

1. **Haziq:** use the existing checkout of `HaziqBinAfzal/linuxops-enterprise-lab`. For a new checkout, clone the repository:

   ```bash
   git clone git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   `git clone` downloads the repository and sets `origin`; `cd` enters it.

2. **Ruveeha:** use her existing fork checkout. If a fresh fork is needed, create it using GitHub's **Fork** action first. Then clone the confirmed fork; the example below applies only if its owner/name is `ruveeha33/linuxops-enterprise-lab`:

   ```bash
   git clone git@github.com:ruveeha33/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   Confirm the actual fork URL before executing; do not assume a fork exists from these instructions.

3. Inspect the checkout before changing branches:

   ```bash
   git status
   git remote -v
   git branch --show-current
   ```

   These show pending edits, remote destinations, and the current branch. Preserve existing work before switching branches.

4. Fetch main from the canonical repository and create a **new validation branch**, so a fresh-build result stays distinct from the historical baseline records:

   ```bash
   git fetch git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git main
   git switch -c YOUR_NEW_VALIDATION_BRANCH FETCH_HEAD
   ```

   Replace `YOUR_NEW_VALIDATION_BRANCH` with a new contributor-specific name such as `haziq/phase-01-fresh-build-validation` or `ruveeha/phase-01-fresh-build-validation`. Fetch reads canonical main; switch creates a new local branch at that fetched commit.

5. After performing the new build, edit the validation record with actual observations and review the changes:

   ```bash
   git diff
   git add phases/phase-01/docs/fresh-build-validation.md
   git diff --cached
   git commit -m "docs: record verified Phase 01 fresh build"
   git push -u origin HEAD
   ```

   `git diff` reviews unstaged edits; `git add` stages the named record; `git diff --cached` shows exactly what will be committed; `git commit` saves the reviewed change; `git push -u origin HEAD` publishes the current branch to the already-confirmed origin and sets its upstream. Add any new evidence files individually after reviewing them.

6. Open a PR to the canonical repository's appropriate base branch, explain which build was executed, and request the other contributor's review. Confirm the base/head selection in GitHub. Keep the validation change scoped to the new build and its actual observations.

The validation record starts **Not executed**. Creating a branch or PR is not evidence that the build passed.

## Official references

- [Microsoft: Generation 1 and 2 VM guidance](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
- [Ubuntu: OpenSSH server installation](https://help.ubuntu.com/community/SSH)
- [Rocky Linux: DNF package management](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)
- [Rocky Linux: firewalld guide](https://docs.rockylinux.org/guides/security/firewalld-beginners/)

Follow the documentation for the installed release. This guide has been reviewed as documentation; its new-build procedure has not been executed against the participants' laptops in this change.
---

[← Phase 01 overview](../README.md) · [Project overview](../../../README.md) · [Evidence index](../evidence/README.md)
) {
    throw "Expected SHA-256 must contain exactly 64 hexadecimal characters."
}
$labActualHash = (Get-FileHash -LiteralPath $labIsoPath -Algorithm SHA256).Hash
if ($labActualHash -ine $labExpectedHash) {
    throw "ISO checksum mismatch. Do not use this ISO."
}
"SHA-256 matches the supplied official checksum."
```

| Step | Purpose |
|---|---|
| `Read-Host` | Collect the local ISO path and expected checksum without hard-coded personal paths |
| Hash format check | Reject a malformed expected SHA-256 |
| `Get-FileHash` | Calculate the downloaded file's SHA-256 |
| Case-insensitive comparison | Stop on a mismatch; matching case is irrelevant for hexadecimal hashes |

Record the ISO filename, actual checksum, official checksum source, and verification date in the [fresh-build validation record](fresh-build-validation.md).

## 2. Create the new VM in Hyper-V Manager

1. Select **New → Virtual Machine**. Use the VM name from the table and a storage location with enough free space.
2. For this reconstruction, choose **Generation 2** with compatible 64-bit installation media. The original VM generation is not established by the current evidence.
3. Assign 4096 MB startup memory and leave **Dynamic Memory disabled**.
4. Select the **Default Switch** for the network adapter.
5. Create a new 128 GiB VHDX. Do not attach a physical Windows disk or an existing disk containing important data.
6. Attach the verified ISO as installation media.
7. Open VM settings and set **2 virtual processors**.
8. For Linux Secure Boot, use **Microsoft UEFI Certificate Authority** when supported by the selected media. Investigate incompatible media/settings rather than assuming the original lab used the same configuration.
9. Start the VM and connect through the Hyper-V console.

Expected outcome: the installer boots and sees only the intended new virtual disk.

## 3. Install the guest OS

### Haziq — Ubuntu Server

Choose the language and keyboard, enable the VM network interface, and use the appropriate default package mirror. Install to the new virtual disk; select the installer LVM option if reproducing the baseline's storage approach. Set the hostname to `web01` and create `haxz` as the normal administrative account. Select OpenSSH server when offered. Finish installation, detach the ISO, and reboot into the installed disk.

### Ruveeha — Rocky Linux

Choose **Minimal Install**, enable the network interface, set hostname `backup01`, and select only the new virtual disk as the installation destination. Use an LVM layout with XFS for root/home if reproducing the recorded storage approach; review and record the actual allocation rather than assuming automatic partitioning matches the old baseline. Create `ruveeha` and select the installer option to make the user an administrator. Finish installation, detach the ISO, and reboot.

### Record the installer storage decision

Before accepting partitioning, record the selected virtual disk and proposed mount layout. The procedure deliberately does not prescribe the historical 70 GiB/53.4 GiB Rocky split because the original partitioning transcript is unavailable.

| Guest | Required layout characteristic | What to record |
|---|---|---|
| Ubuntu | Root on LVM with ext4 when following this lab's storage approach | EFI/boot partitions, VG/LV names, root size, and unallocated VG space |
| Rocky | Root/home on LVM with XFS when following this lab's storage approach | EFI/boot partitions, VG/LV names, root/home sizes, and swap |
| Both | Only the new disposable VHDX is selected | Installer's final storage summary before writing changes |

A different layout can still support the learning exercises, but must be documented as a deviation. After installation, verify it with `lsblk -f`, `df -hT`, and, where LVM is used, `sudo pvs`, `sudo vgs`, and `sudo lvs`. The latter three inspect physical volumes, volume groups, and logical volumes respectively.

Passwords are entered interactively. Do not put them into commands, repository files, or screenshots.

### Verify installation identity — inside each Linux VM

| Command | Purpose | Acceptance criterion |
|---|---|---|
| `hostnamectl` | Inspect host, OS, kernel, and virtualization | Intended hostname and chosen OS are displayed; record the actual version |
| `whoami` | Identify the current user | Haziq sees haxz; Ruveeha sees ruveeha |
| `id` | Inspect group membership | Expected administrator group is present |
| `sudo whoami` | Test effective administrative access | Authorized elevation prints root |
| `sudo -l` | Inspect the effective sudo policy | The policy permits the intended administration work |

If sudo is denied, use the VM console and the installation's administrator access to diagnose account/group/policy configuration. Do not assume group membership alone proves permission.

## 4. Update the new VM

Run only the commands for the correct distribution. Review each package transaction before accepting it.

### Haziq — Ubuntu guest

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo apt update` | Refresh package metadata; resolve repository/signature errors before upgrading |
| `sudo apt upgrade` | Apply available upgrades after reviewing the transaction; record successful completion or any deferred packages |
| `sudo reboot` | Restart the guest; the SSH session closes and must be re-established |

### Ruveeha — Rocky guest

```bash
sudo dnf upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo dnf upgrade` | Apply updates from enabled repositories after reviewing the transaction; record success or errors |
| `sudo reboot` | Restart the guest so a newly installed kernel can become active |

After reconnecting, use `uname -r` to record the running kernel and `uptime` to inspect uptime. Their outputs do not independently prove that the package transaction succeeded; preserve its outcome separately.

## 5. Establish SSH access

Run server commands in the Linux VM console. Keep console access available while testing SSH.

### Haziq — Ubuntu guest

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
```

The first command installs the SSH server if absent. The second enables boot startup and starts the service. The status command checks service state. On Ubuntu installations using socket activation, also inspect `systemctl status ssh.socket --no-pager`; an active socket may start the daemon on demand.

### Ruveeha — Rocky guest

```bash
sudo dnf install openssh-server
sudo systemctl enable --now sshd
systemctl status sshd --no-pager
```

The first command installs the package, the second enables/starts the daemon, and the third checks its state. Expect an active service or investigate the status and logs.

### Both guests — address and listener checks

| Command | Purpose and expected result |
|---|---|
| `ip -br addr` | Identify the active VM interface and current IP address |
| `sudo ss -lntp` | Inspect TCP listening sockets and owning processes; check SSH's configured port, normally 22 |
| `sudo journalctl -u ssh -n 30 --no-pager` | Ubuntu service diagnostics if needed |
| `sudo journalctl -u sshd -n 30 --no-pager` | Rocky service diagnostics if needed |

Inspect the existing firewall before changing it. On Ubuntu, `sudo ufw status` reports UFW state. If active and blocking the intended lab connection, `sudo ufw allow 22/tcp` permits the default SSH port. This modifies a rule; it does not itself enable UFW.

On Rocky, `sudo firewall-cmd --state` checks whether firewalld is running and `sudo firewall-cmd --get-active-zones` identifies the guest interface's zone. If running and SSH is blocked, use that actual zone:

```bash
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --permanent --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --query-service=ssh
```

Replace `YOUR_ACTIVE_ZONE` before execution. The first command changes runtime policy, the second persists it, and the third verifies runtime permission. This is new-build guidance, not evidence that firewall changes occurred in the original lab. Review restrictions and hardening in Phase 03.

### Windows PowerShell — connect from the same host

Replace the example addresses with each VM's current IP:

```powershell
ssh haxz@YOUR_WEB01_IP
ssh ruveeha@YOUR_BACKUP01_IP
```

Haziq runs the first command on Haziq's host; Ruveeha runs the second on Ruveeha's host. Each invokes the Windows SSH client and authenticates to the local guest. Verify a new host key against the guest console before accepting it. For example, `sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` prints the guest's public host-key fingerprint for comparison.

After login, `hostnamectl` and `whoami` must identify the intended guest/account. Do not blindly remove known-host entries when a key changes. These tests do not establish connectivity between Haziq's and Ruveeha's separate VM networks.

## 6. Verify resources and service health

The two baseline documents explain each inspection command. Run the relevant commands, record actual output, and compare it with the intended new-build settings.

| Check | Command | Acceptance criterion |
|---|---|---|
| CPU capacity | `nproc` | 2 processing units available for this reconstruction |
| Memory and swap | `free -h` | Guest memory is consistent with the configured allocation after overhead; investigate a large shortfall |
| Disk layout | `lsblk -f` | Expected new disk, logical volumes, filesystem types, and mounts are present |
| Filesystem space | `df -hT` | Root is mounted with the intended filesystem and adequate free space |
| Failed units | `systemctl --failed --no-pager` | Zero failed units, or each failure is explicitly investigated |
| Current kernel | `uname -r` | Running kernel recorded after reboot |
| Load and uptime | `uptime` | Snapshot recorded and interpreted with CPU/workload context |

A 4 GiB Hyper-V allocation does not necessarily appear as exactly 4.0 GiB in the guest. Use the [memory incident report](memory-troubleshooting.md) for the observed 609 MiB failure and its verified resolution.

## 7. Record a reviewable result

Record the build date, installer provenance, VM settings, actual OS/kernel, account, storage layout, SSH verification, package transaction outcome, and resource checks. Explain deviations from the historical baseline rather than editing old observations to match a new run.

Choose important evidence: one clear baseline health capture per server, meaningful troubleshooting before/after, and the reviewed PR. Use existing screenshots only for the historical observations they actually show. Redact credentials, keys, tokens, and confidential details.

### Git handoff — Ubuntu WSL

Use Ubuntu WSL for these Git operations, not Windows PowerShell or the server guest. This workflow assumes Git is installed and the contributor's GitHub SSH authentication already works. Never commit an SSH private key.

1. **Haziq:** use the existing checkout of `HaziqBinAfzal/linuxops-enterprise-lab`. For a new checkout, clone the repository:

   ```bash
   git clone git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   `git clone` downloads the repository and sets `origin`; `cd` enters it.

2. **Ruveeha:** use her existing fork checkout. If a fresh fork is needed, create it using GitHub's **Fork** action first. Then clone the confirmed fork; the example below applies only if its owner/name is `ruveeha33/linuxops-enterprise-lab`:

   ```bash
   git clone git@github.com:ruveeha33/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   Confirm the actual fork URL before executing; do not assume a fork exists from these instructions.

3. Inspect the checkout before changing branches:

   ```bash
   git status
   git remote -v
   git branch --show-current
   ```

   These show pending edits, remote destinations, and the current branch. Preserve existing work before switching branches.

4. Fetch main from the canonical repository and create a **new validation branch**, so a fresh-build result stays distinct from the historical baseline records:

   ```bash
   git fetch git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git main
   git switch -c YOUR_NEW_VALIDATION_BRANCH FETCH_HEAD
   ```

   Replace `YOUR_NEW_VALIDATION_BRANCH` with a new contributor-specific name such as `haziq/phase-01-fresh-build-validation` or `ruveeha/phase-01-fresh-build-validation`. Fetch reads canonical main; switch creates a new local branch at that fetched commit.

5. After performing the new build, edit the validation record with actual observations and review the changes:

   ```bash
   git diff
   git add phases/phase-01/docs/fresh-build-validation.md
   git diff --cached
   git commit -m "docs: record verified Phase 01 fresh build"
   git push -u origin HEAD
   ```

   `git diff` reviews unstaged edits; `git add` stages the named record; `git diff --cached` shows exactly what will be committed; `git commit` saves the reviewed change; `git push -u origin HEAD` publishes the current branch to the already-confirmed origin and sets its upstream. Add any new evidence files individually after reviewing them.

6. Open a PR to the canonical repository's appropriate base branch, explain which build was executed, and request the other contributor's review. Confirm the base/head selection in GitHub. Keep the validation change scoped to the new build and its actual observations.

The validation record starts **Not executed**. Creating a branch or PR is not evidence that the build passed.

## Official references

- [Microsoft: Generation 1 and 2 VM guidance](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
- [Ubuntu: OpenSSH server installation](https://help.ubuntu.com/community/SSH)
- [Rocky Linux: DNF package management](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)
- [Rocky Linux: firewalld guide](https://docs.rockylinux.org/guides/security/firewalld-beginners/)

Follow the documentation for the installed release. This guide has been reviewed as documentation; its new-build procedure has not been executed against the participants' laptops in this change.
---

[← Phase 01 overview](../README.md) · [Project overview](../../../README.md) · [Evidence index](../evidence/README.md)
) {
    throw "Expected SHA-256 must contain exactly 64 hexadecimal characters."
}
$labActualHash = (Get-FileHash -LiteralPath $labIsoPath -Algorithm SHA256).Hash
if ($labActualHash -ine $labExpectedHash) {
    throw "ISO checksum mismatch. Do not use this ISO."
}
"SHA-256 matches the supplied official checksum."
```

| Step | Purpose |
|---|---|
| `Read-Host` | Collect the local ISO path and expected checksum without hard-coded personal paths |
| Hash format check | Reject a malformed expected SHA-256 |
| `Get-FileHash` | Calculate the downloaded file's SHA-256 |
| Case-insensitive comparison | Stop on a mismatch; matching case is irrelevant for hexadecimal hashes |

Record the ISO filename, actual checksum, official checksum source, and verification date in the [fresh-build validation record](fresh-build-validation.md).

## 2. Create the new VM in Hyper-V Manager

1. Select **New → Virtual Machine**. Use the VM name from the table and a storage location with enough free space.
2. For this reconstruction, choose **Generation 2** with compatible 64-bit installation media. The original VM generation is not established by the current evidence.
3. Assign 4096 MB startup memory and leave **Dynamic Memory disabled**.
4. Select the **Default Switch** for the network adapter.
5. Create a new 128 GiB VHDX. Do not attach a physical Windows disk or an existing disk containing important data.
6. Attach the verified ISO as installation media.
7. Open VM settings and set **2 virtual processors**.
8. For Linux Secure Boot, use **Microsoft UEFI Certificate Authority** when supported by the selected media. Investigate incompatible media/settings rather than assuming the original lab used the same configuration.
9. Start the VM and connect through the Hyper-V console.

Expected outcome: the installer boots and sees only the intended new virtual disk.

## 3. Install the guest OS

### Haziq — Ubuntu Server

Choose the language and keyboard, enable the VM network interface, and use the appropriate default package mirror. Install to the new virtual disk; select the installer LVM option if reproducing the baseline's storage approach. Set the hostname to `web01` and create `haxz` as the normal administrative account. Select OpenSSH server when offered. Finish installation, detach the ISO, and reboot into the installed disk.

### Ruveeha — Rocky Linux

Choose **Minimal Install**, enable the network interface, set hostname `backup01`, and select only the new virtual disk as the installation destination. Use an LVM layout with XFS for root/home if reproducing the recorded storage approach; review and record the actual allocation rather than assuming automatic partitioning matches the old baseline. Create `ruveeha` and select the installer option to make the user an administrator. Finish installation, detach the ISO, and reboot.

### Record the installer storage decision

Before accepting partitioning, record the selected virtual disk and proposed mount layout. The procedure deliberately does not prescribe the historical 70 GiB/53.4 GiB Rocky split because the original partitioning transcript is unavailable.

| Guest | Required layout characteristic | What to record |
|---|---|---|
| Ubuntu | Root on LVM with ext4 when following this lab's storage approach | EFI/boot partitions, VG/LV names, root size, and unallocated VG space |
| Rocky | Root/home on LVM with XFS when following this lab's storage approach | EFI/boot partitions, VG/LV names, root/home sizes, and swap |
| Both | Only the new disposable VHDX is selected | Installer's final storage summary before writing changes |

A different layout can still support the learning exercises, but must be documented as a deviation. After installation, verify it with `lsblk -f`, `df -hT`, and, where LVM is used, `sudo pvs`, `sudo vgs`, and `sudo lvs`. The latter three inspect physical volumes, volume groups, and logical volumes respectively.

Passwords are entered interactively. Do not put them into commands, repository files, or screenshots.

### Verify installation identity — inside each Linux VM

| Command | Purpose | Acceptance criterion |
|---|---|---|
| `hostnamectl` | Inspect host, OS, kernel, and virtualization | Intended hostname and chosen OS are displayed; record the actual version |
| `whoami` | Identify the current user | Haziq sees haxz; Ruveeha sees ruveeha |
| `id` | Inspect group membership | Expected administrator group is present |
| `sudo whoami` | Test effective administrative access | Authorized elevation prints root |
| `sudo -l` | Inspect the effective sudo policy | The policy permits the intended administration work |

If sudo is denied, use the VM console and the installation's administrator access to diagnose account/group/policy configuration. Do not assume group membership alone proves permission.

## 4. Update the new VM

Run only the commands for the correct distribution. Review each package transaction before accepting it.

### Haziq — Ubuntu guest

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo apt update` | Refresh package metadata; resolve repository/signature errors before upgrading |
| `sudo apt upgrade` | Apply available upgrades after reviewing the transaction; record successful completion or any deferred packages |
| `sudo reboot` | Restart the guest; the SSH session closes and must be re-established |

### Ruveeha — Rocky guest

```bash
sudo dnf upgrade
sudo reboot
```

| Command | Purpose and verification |
|---|---|
| `sudo dnf upgrade` | Apply updates from enabled repositories after reviewing the transaction; record success or errors |
| `sudo reboot` | Restart the guest so a newly installed kernel can become active |

After reconnecting, use `uname -r` to record the running kernel and `uptime` to inspect uptime. Their outputs do not independently prove that the package transaction succeeded; preserve its outcome separately.

## 5. Establish SSH access

Run server commands in the Linux VM console. Keep console access available while testing SSH.

### Haziq — Ubuntu guest

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
```

The first command installs the SSH server if absent. The second enables boot startup and starts the service. The status command checks service state. On Ubuntu installations using socket activation, also inspect `systemctl status ssh.socket --no-pager`; an active socket may start the daemon on demand.

### Ruveeha — Rocky guest

```bash
sudo dnf install openssh-server
sudo systemctl enable --now sshd
systemctl status sshd --no-pager
```

The first command installs the package, the second enables/starts the daemon, and the third checks its state. Expect an active service or investigate the status and logs.

### Both guests — address and listener checks

| Command | Purpose and expected result |
|---|---|
| `ip -br addr` | Identify the active VM interface and current IP address |
| `sudo ss -lntp` | Inspect TCP listening sockets and owning processes; check SSH's configured port, normally 22 |
| `sudo journalctl -u ssh -n 30 --no-pager` | Ubuntu service diagnostics if needed |
| `sudo journalctl -u sshd -n 30 --no-pager` | Rocky service diagnostics if needed |

Inspect the existing firewall before changing it. On Ubuntu, `sudo ufw status` reports UFW state. If active and blocking the intended lab connection, `sudo ufw allow 22/tcp` permits the default SSH port. This modifies a rule; it does not itself enable UFW.

On Rocky, `sudo firewall-cmd --state` checks whether firewalld is running and `sudo firewall-cmd --get-active-zones` identifies the guest interface's zone. If running and SSH is blocked, use that actual zone:

```bash
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --permanent --add-service=ssh
sudo firewall-cmd --zone=YOUR_ACTIVE_ZONE --query-service=ssh
```

Replace `YOUR_ACTIVE_ZONE` before execution. The first command changes runtime policy, the second persists it, and the third verifies runtime permission. This is new-build guidance, not evidence that firewall changes occurred in the original lab. Review restrictions and hardening in Phase 03.

### Windows PowerShell — connect from the same host

Replace the example addresses with each VM's current IP:

```powershell
ssh haxz@YOUR_WEB01_IP
ssh ruveeha@YOUR_BACKUP01_IP
```

Haziq runs the first command on Haziq's host; Ruveeha runs the second on Ruveeha's host. Each invokes the Windows SSH client and authenticates to the local guest. Verify a new host key against the guest console before accepting it. For example, `sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub` prints the guest's public host-key fingerprint for comparison.

After login, `hostnamectl` and `whoami` must identify the intended guest/account. Do not blindly remove known-host entries when a key changes. These tests do not establish connectivity between Haziq's and Ruveeha's separate VM networks.

## 6. Verify resources and service health

The two baseline documents explain each inspection command. Run the relevant commands, record actual output, and compare it with the intended new-build settings.

| Check | Command | Acceptance criterion |
|---|---|---|
| CPU capacity | `nproc` | 2 processing units available for this reconstruction |
| Memory and swap | `free -h` | Guest memory is consistent with the configured allocation after overhead; investigate a large shortfall |
| Disk layout | `lsblk -f` | Expected new disk, logical volumes, filesystem types, and mounts are present |
| Filesystem space | `df -hT` | Root is mounted with the intended filesystem and adequate free space |
| Failed units | `systemctl --failed --no-pager` | Zero failed units, or each failure is explicitly investigated |
| Current kernel | `uname -r` | Running kernel recorded after reboot |
| Load and uptime | `uptime` | Snapshot recorded and interpreted with CPU/workload context |

A 4 GiB Hyper-V allocation does not necessarily appear as exactly 4.0 GiB in the guest. Use the [memory incident report](memory-troubleshooting.md) for the observed 609 MiB failure and its verified resolution.

## 7. Record a reviewable result

Record the build date, installer provenance, VM settings, actual OS/kernel, account, storage layout, SSH verification, package transaction outcome, and resource checks. Explain deviations from the historical baseline rather than editing old observations to match a new run.

Choose important evidence: one clear baseline health capture per server, meaningful troubleshooting before/after, and the reviewed PR. Use existing screenshots only for the historical observations they actually show. Redact credentials, keys, tokens, and confidential details.

### Git handoff — Ubuntu WSL

Use Ubuntu WSL for these Git operations, not Windows PowerShell or the server guest. This workflow assumes Git is installed and the contributor's GitHub SSH authentication already works. Never commit an SSH private key.

1. **Haziq:** use the existing checkout of `HaziqBinAfzal/linuxops-enterprise-lab`. For a new checkout, clone the repository:

   ```bash
   git clone git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   `git clone` downloads the repository and sets `origin`; `cd` enters it.

2. **Ruveeha:** use her existing fork checkout. If a fresh fork is needed, create it using GitHub's **Fork** action first. Then clone the confirmed fork; the example below applies only if its owner/name is `ruveeha33/linuxops-enterprise-lab`:

   ```bash
   git clone git@github.com:ruveeha33/linuxops-enterprise-lab.git
   cd linuxops-enterprise-lab
   ```

   Confirm the actual fork URL before executing; do not assume a fork exists from these instructions.

3. Inspect the checkout before changing branches:

   ```bash
   git status
   git remote -v
   git branch --show-current
   ```

   These show pending edits, remote destinations, and the current branch. Preserve existing work before switching branches.

4. Fetch main from the canonical repository and create a **new validation branch**, so a fresh-build result stays distinct from the historical baseline records:

   ```bash
   git fetch git@github.com:HaziqBinAfzal/linuxops-enterprise-lab.git main
   git switch -c YOUR_NEW_VALIDATION_BRANCH FETCH_HEAD
   ```

   Replace `YOUR_NEW_VALIDATION_BRANCH` with a new contributor-specific name such as `haziq/phase-01-fresh-build-validation` or `ruveeha/phase-01-fresh-build-validation`. Fetch reads canonical main; switch creates a new local branch at that fetched commit.

5. After performing the new build, edit the validation record with actual observations and review the changes:

   ```bash
   git diff
   git add phases/phase-01/docs/fresh-build-validation.md
   git diff --cached
   git commit -m "docs: record verified Phase 01 fresh build"
   git push -u origin HEAD
   ```

   `git diff` reviews unstaged edits; `git add` stages the named record; `git diff --cached` shows exactly what will be committed; `git commit` saves the reviewed change; `git push -u origin HEAD` publishes the current branch to the already-confirmed origin and sets its upstream. Add any new evidence files individually after reviewing them.

6. Open a PR to the canonical repository's appropriate base branch, explain which build was executed, and request the other contributor's review. Confirm the base/head selection in GitHub. Keep the validation change scoped to the new build and its actual observations.

The validation record starts **Not executed**. Creating a branch or PR is not evidence that the build passed.

## Official references

- [Microsoft: Generation 1 and 2 VM guidance](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
- [Ubuntu: OpenSSH server installation](https://help.ubuntu.com/community/SSH)
- [Rocky Linux: DNF package management](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)
- [Rocky Linux: firewalld guide](https://docs.rockylinux.org/guides/security/firewalld-beginners/)

Follow the documentation for the installed release. This guide has been reviewed as documentation; its new-build procedure has not been executed against the participants' laptops in this change.
---

[← Phase 01 overview](../README.md) · [Project overview](../../../README.md) · [Evidence index](../evidence/README.md)
