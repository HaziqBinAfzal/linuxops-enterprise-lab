# LinuxOps Enterprise Lab

**A collaborative, hands-on Linux system administration portfolio project.**

LinuxOps Enterprise Lab is a two-person training environment for configuring, verifying, troubleshooting, securing, and documenting Linux servers. We work on separate Windows hosts using Microsoft Hyper-V virtual machines, Ubuntu WSL for Git workflows, and peer-reviewed GitHub pull requests.

> **Current status:** Phase 1 completed. This is a learning lab, not a production deployment.

## Lab architecture

| Server | Operating system | Virtualization | Contributor | Planned lab role |
|---|---|---|---|---|
| `web01` | Ubuntu Server 26.04.1 LTS | Microsoft Hyper-V | [Haziq](https://github.com/HaziqBinAfzal) | Linux administration and web-service practice |
| `backup01` | Rocky Linux 9.8 Minimal | Microsoft Hyper-V | [Ruveeha](https://github.com/ruveeha33) | Linux administration and backup practice |

Each VM currently uses its host's Hyper-V Default Switch. Direct connectivity between the two hosts' VMs has **not yet been established**. Server names describe intended roles; they do not imply web or backup services have already been deployed.

## Phase 1 — Completed: server baselines and troubleshooting

- Installed, updated, and verified Ubuntu and Rocky Linux server environments.
- Verified administrator privileges, SSH access, networking, CPU, RAM, disk usage, and systemd health.
- Documented Ubuntu ext4/LVM and Rocky Linux XFS/LVM storage configurations.
- Investigated Rocky Linux reporting only **609 MiB** of guest-visible RAM, alongside Hyper-V `hv_balloon` warnings.
- Disabled Hyper-V Dynamic Memory for `backup01`, then verified approximately **3.6 GiB** of guest-visible RAM and zero failed systemd units.
- Submitted, reviewed, approved, and merged two separate documentation pull requests.

### Documentation

- [Ubuntu `web01` baseline](docs/phase-01/web01-baseline.md)
- [Rocky Linux `backup01` baseline](docs/phase-01/backup01-baseline.md)
- [Hyper-V memory troubleshooting report](docs/phase-01/memory-troubleshooting.md)
- [Selected Phase 1 evidence](evidence/phase-01/README.md) *(images to be added in this presentation update)*

### Peer-reviewed contributions

- [PR #1 — Ubuntu `web01` server baseline](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/1)
- [PR #2 — Rocky Linux baseline and memory troubleshooting](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/2)

## Project roadmap

| Phase | Focus | Status |
|---|---|---|
| 1 | Server setup, baseline verification, and troubleshooting | Completed |
| 2 | Users, groups, permissions, sudo, and ACLs | Planned |
| 3 | SSH and network security | Planned |
| 4 | systemd and web-service administration | Planned |
| 5 | Storage and LVM administration | Planned |
| 6 | Backups and restore testing | Planned |
| 7 | Logging and monitoring | Planned |
| 8 | Bash automation and scheduling | Planned |
| 9 | Troubleshooting exercises | Planned |
| 10 | Final validation and portfolio documentation | Planned |

## Contributors

**[Haziq](https://github.com/HaziqBinAfzal):** Ubuntu `web01` server baseline, health verification, documentation, and peer review.

**[Ruveeha](https://github.com/ruveeha33):** Rocky Linux `backup01` server baseline, Hyper-V memory troubleshooting, documentation, and peer review.

## Scope and safety

This repository documents a training environment. Resource usage and private-network addresses are point-in-time observations. Never commit passwords, private SSH keys, authentication tokens, or other secrets.
