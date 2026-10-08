# LinuxOps Enterprise Lab

**Hands-on, two-person Linux system administration lab**

A collaborative portfolio project by **[Haziq](https://github.com/HaziqBinAfzal)** and **[Ruveeha](https://github.com/ruveeha33)**. We administer separate Ubuntu and Rocky Linux virtual machines, document verified work, and collaborate through peer-reviewed GitHub pull requests.

This is a **learning environment**, not a production deployment.

## Lab overview

| Server | Operating system | Host environment | Contributor | Role |
|---|---|---|---|---|
| `web01` | Ubuntu Server 26.04.1 LTS | Hyper-V VM | Haziq | Ubuntu administration; planned web-service lab |
| `backup01` | Rocky Linux 9.8 Minimal | Hyper-V VM | Ruveeha | Rocky Linux administration; planned backup lab |

Each contributor uses Ubuntu WSL for Git operations. The VMs currently run on separate Hyper-V Default Switch networks; inter-host VM connectivity is not yet established.

## Phases

| Phase | Topic | Status |
|---|---|---|
| 01 | [Server Baseline & Initial Administration](phases/phase-01/README.md) | Completed |
| 02 | [Users, Groups & Permissions](phases/phase-02/README.md) | Planned |
| 03 | [SSH & Network Security](phases/phase-03/README.md) | Planned |
| 04 | [Services & systemd](phases/phase-04/README.md) | Planned |
| 05 | [Storage & LVM](phases/phase-05/README.md) | Planned |
| 06 | [Backups & Restore Testing](phases/phase-06/README.md) | Planned |
| 07 | [Logging & Monitoring](phases/phase-07/README.md) | Planned |
| 08 | [Bash Automation & Scheduling](phases/phase-08/README.md) | Planned |
| 09 | [Troubleshooting Exercises](phases/phase-09/README.md) | Planned |
| 10 | [Final Validation & Portfolio](phases/phase-10/README.md) | Planned |

Select a phase above to view its own README, tasks, documentation, and available evidence.

## Phase 1 highlights

We verified OS/kernel identity, sudo and SSH access, networking, CPU, memory, LVM/filesystem layouts, and systemd health on both servers. On Rocky Linux `backup01`, a Hyper-V Dynamic Memory issue reduced guest-visible RAM to approximately **609 MiB**; after disabling Dynamic Memory, the guest reported approximately **3.6 GiB** and zero failed systemd units.

[Explore Phase 1 →](phases/phase-01/README.md)

## Collaboration

- [PR #1: Haziq's Ubuntu server baseline](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/1)
- [PR #2: Ruveeha's Rocky Linux baseline and troubleshooting](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/2)

**Security:** Never publish private keys, credentials, tokens, or sensitive host information. Lab IP addresses and resource readings are point-in-time observations.
