# Phase 01: Follow Along Linux Server Lab

[Phase 01 overview](../README.md) | [Original Ubuntu results](web01-baseline.md) | [Original Rocky results](backup01-baseline.md) | [Memory incident](memory-troubleshooting.md) | [Evidence](../evidence/README.md)

This tutorial is for anyone who wants to repeat our **server baseline and initial administration** exercises. Follow the steps in order. You may choose **Ubuntu Server** or **Rocky Linux**. The original team used separate Hyper-V hosts and one VM per person. You only need one VM to begin.

**What you will learn:** create a VM, install Linux, verify your administrator account, update packages, enable SSH, inspect network and storage, check server health, and document your results.

**Safety:** Use a new disposable virtual disk, not an existing drive containing important data. Run Linux commands inside the **VM**, not in Ubuntu WSL. Do not copy our old IP addresses or passwords. A clean installation using this exact tutorial has not yet been independently validated. Record your own results.

## Step 1: Prepare your Windows host

**Where:** Windows PowerShell.

```powershell
Get-ComputerInfo | Select-Object WindowsProductName,WindowsVersion
```

**Purpose:** Identify your Windows edition. **Expected:** Your Windows edition and version. Hyper-V requires a supported Windows edition, enabled hardware virtualization, and sufficient RAM and storage.

Open **Turn Windows features on or off**, enable **Hyper-V**, restart if prompted, then open **Hyper-V Manager**. If Hyper-V is missing, check your Windows edition and firmware virtualization settings before proceeding.

## Step 2: Download and check your Linux ISO

**Where:** Windows browser and PowerShell.

Download from [Ubuntu Server](https://ubuntu.com/download/server) or [Rocky Linux](https://rockylinux.org/download). Note the exact ISO filename and find its official SHA256 checksum. Follow the vendor's signature verification instructions when available.

```powershell
$iso = Read-Host "Full path to your ISO"
$expected = (Read-Host "Official SHA256 for this exact ISO").Trim()
if ($expected -notmatch '^[0-9a-fA-F]{64}$') {
    throw "SHA256 must have 64 hexadecimal characters."
}
$actual = (Get-FileHash -LiteralPath $iso -Algorithm SHA256).Hash
if ($actual -ine $expected) {
    throw "Checksum mismatch. Stop and verify the download."
}
"Checksum matched."
```

**Purpose:** `Get-FileHash` calculates the file checksum and compares it with the official value. **Expected:** `Checksum matched.` If not, stop and investigate the source and filename.

## Step 3: Create your Hyper-V VM

**Where:** Windows Hyper-V Manager.

1. Select **New**, then **Virtual Machine**.
2. Name the VM `web01` for Ubuntu or `backup01` for Rocky, or use your own name.
3. Choose **Generation 2** when compatible with your installer.
4. Set startup RAM to **4096 MB** and turn **Dynamic Memory off**.
5. Choose **Default Switch**.
6. Create a **new** dynamically expanding virtual disk. **128 GB** is one possible practice allocation, not a requirement.
7. Attach your verified ISO.
8. In VM **Settings**, choose **2 virtual processors**.
9. If Secure Boot needs adjustment for Linux, select **Microsoft UEFI Certificate Authority** when supported.
10. Select **Connect** and **Start**.

**Expected:** The Linux installer boots. If not, inspect the ISO, boot order, generation, Secure Boot template, and available host resources.

## Step 4: Install Linux

**Where:** VM console in Hyper-V.

**Ubuntu:** Choose language, keyboard and DHCP networking. Select **only the new virtual disk**. Choose LVM if you want a layout like our baseline. Set hostname, create your own user and password, select OpenSSH server if offered, finish installation, detach the ISO and reboot.

**Rocky:** Select Minimal Install. Enable networking, set hostname, select **only the new virtual disk**, choose LVM and XFS if reproducing our storage approach, create your user with administrator privileges, finish installation, detach ISO and reboot.

**Expected:** A login prompt and successful login. Installer screens and partition sizes may vary. Review the storage summary before confirming any write.

## Step 5: Verify Linux identity and access

**Where:** Linux VM terminal.

```bash
hostnamectl
whoami
id
cat /etc/os-release
uname -r
sudo whoami
sudo -l
```

| Command | What it does | What to check |
|---|---|---|
| `hostnamectl` | Displays hostname, OS and virtualization | You are in the intended VM |
| `whoami` | Prints current username | Your newly created user |
| `id` | Shows account IDs and groups | Ubuntu often uses `sudo`; Rocky often uses `wheel` |
| `cat /etc/os-release` | Shows Linux distribution details | Ubuntu or Rocky |
| `uname -r` | Prints running kernel | A valid kernel version |
| `sudo whoami` | Tests privileged access | Prints `root` |
| `sudo -l` | Displays permitted sudo commands | Appropriate authorization |

**If it fails:** Use the VM console to check the account and installation. Do not assume group membership alone proves sudo authorization.

## Step 6: Update your server

**Where:** Ubuntu VM only.

```bash
sudo apt update
sudo apt upgrade
sudo reboot
```

`apt update` refreshes package lists, `apt upgrade` applies eligible updates, and `reboot` restarts the VM.

**Where:** Rocky VM only.

```bash
sudo dnf upgrade
sudo reboot
```

`dnf upgrade` applies available updates; `reboot` restarts the VM.

**Expected:** No unresolved package errors. Log in again, then run `uname -r` and `uptime` to record the running kernel and uptime. If updates fail, check the error and network before continuing.

## Step 7: Install and start SSH

**Where:** Ubuntu VM only.

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl status ssh --no-pager
```

**Where:** Rocky VM only.

```bash
sudo dnf install openssh-server
sudo systemctl enable --now sshd
systemctl status sshd --no-pager
```

The first command installs SSH, the second starts it and enables startup, and the third checks service state. **Expected:** Active SSH service or, on some Ubuntu installations, an active `ssh.socket`.

If the service fails, inspect logs using `sudo journalctl -u ssh -n 30 --no-pager` on Ubuntu or `sudo journalctl -u sshd -n 30 --no-pager` on Rocky.

## Step 8: Find the VM address

**Where:** Linux VM.

```bash
ip -br addr
sudo ss -lntp
```

`ip -br addr` shows interface state and IP addresses. `ss -lntp` shows listening TCP services. **Expected:** An active interface with a usable IP and SSH listening, usually on TCP 22. Do not use `127.0.0.1` for remote access.

If the VM has no IP, check Hyper-V Default Switch and guest networking. If SSH is not listening, return to Step 7.

## Step 9: Connect from Windows

**Where:** Windows PowerShell on the same host.

```powershell
ssh YOUR_USERNAME@YOUR_VM_IP
```

Replace both placeholders. Before accepting an unfamiliar host key, check the fingerprint **inside the VM console**:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Compare the displayed fingerprint with the SSH client's prompt. **Expected:** A Linux shell after authentication. Run `hostnamectl` and `whoami` to confirm the destination.

If connection fails, verify IP, SSH service, firewall and account. On Ubuntu inspect `sudo ufw status`. On Rocky inspect `sudo firewall-cmd --state` and `sudo firewall-cmd --get-active-zones`. Only adjust rules when you confirm they block the intended connection. Do not disable the firewall blindly.

## Step 10: Inspect CPU, memory and uptime

**Where:** Linux VM.

```bash
nproc
free -h
uptime
```

`nproc` reports available CPUs. `free -h` reports total and available RAM plus swap. `uptime` shows uptime and load averages.

**Expected:** About 2 CPUs with the suggested VM settings. Guest visible memory should be broadly consistent with the Hyper-V allocation; it may not show exactly 4.0 GiB. A dramatic memory shortfall needs investigation, not guesswork.

## Step 11: Inspect disks and LVM

**Where:** Linux VM.

```bash
lsblk -f
df -hT
sudo pvs
sudo vgs
sudo lvs
```

`lsblk -f` displays block devices, filesystems and mounts. `df -hT` displays filesystem usage. `pvs`, `vgs` and `lvs` inspect LVM physical volumes, volume groups and logical volumes.

**Expected:** A mounted root filesystem with free space. If you did not select LVM, the LVM output may be empty or the utilities may be absent. Do not repartition your VM merely to match our screenshots.

## Step 12: Check service health

**Where:** Linux VM.

```bash
systemctl --failed --no-pager
```

This lists failed systemd units. **Expected:** Ideally zero. If a unit is listed, substitute its actual name:

```bash
systemctl status UNIT_NAME --no-pager
sudo journalctl -u UNIT_NAME -n 30 --no-pager
```

These commands display the unit status and recent logs. A zero failed unit count is useful baseline evidence, not proof that every application is healthy.

## Step 13: Learn from our memory troubleshooting incident

Our Rocky Linux VM initially showed approximately **609 MiB** of RAM. We compared guest memory readings with Hyper-V settings and kernel messages, disabled Dynamic Memory while the VM was stopped, restarted, and verified approximately **3.6 GiB** guest-visible memory.

Read the [complete incident report](memory-troubleshooting.md) for the original commands and screenshots. **Do not deliberately create the problem on a healthy VM.** If you encounter a mismatch, record readings before making changes.

## Step 14: Save your own evidence

Record the VM name, OS, account, kernel, IP, CPU, RAM, swap, storage layout, SSH result, package update result and systemd status. Save clear screenshots without credentials or secrets.

Use the [fresh build validation form](fresh-build-validation.md) to document what you personally executed. Our [Ubuntu baseline](web01-baseline.md) and [Rocky baseline](backup01-baseline.md) are historical comparisons, not expected output to copy.

## Step 15: Optional GitHub contribution

You can complete this lab without GitHub. To contribute documentation, fork the [project repository](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab), clone your own fork, make a branch, and open a pull request. The original [setup guide](setup-guide.md) also describes the collaborators' WSL Git workflow.

**Finished:** You have completed Phase 01 when you can identify your server, verify sudo and SSH, inspect CPU and memory, understand your disk layout, check service health, and document your actual results. This does not make the VM production ready.
