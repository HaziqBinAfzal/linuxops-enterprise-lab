# Phase 01 — Server Baseline & Initial Administration

**Status: Completed** · [← Project overview](../../README.md) · [Next: Phase 02 →](../phase-02/README.md)

## Objectives

Set up two Linux server VMs, establish administrative access, inspect baseline resource and service health, troubleshoot observed issues, and document the results.

## Lab servers and contributors

| Contributor | Server | OS | Contribution |
|---|---|---|---|
| [Haziq](https://github.com/HaziqBinAfzal) | `web01` | Ubuntu Server 26.04.1 LTS | Server identity, sudo/SSH, CPU, memory, disk, network, and systemd verification |
| [Ruveeha](https://github.com/ruveeha33) | `backup01` | Rocky Linux 9.8 Minimal | Rocky Linux baseline, LVM/XFS inspection, and Hyper-V memory troubleshooting |

## Completed tasks

- Installed and updated the server operating systems.
- Verified administrative privileges and SSH access.
- Recorded kernel, CPU, RAM, swap, storage, filesystem, and network details.
- Checked systemd for failed units.
- Investigated Rocky Linux showing 609 MiB of guest-visible RAM with `hv_balloon` warnings.
- Disabled Hyper-V Dynamic Memory and verified approximately 3.6 GiB of guest-visible RAM.
- Submitted, peer-reviewed, and merged the Phase 1 documentation through two pull requests.

## Documentation

- [Haziq: Ubuntu `web01` baseline](docs/web01-baseline.md)
- [Ruveeha: Rocky Linux `backup01` baseline](docs/backup01-baseline.md)
- [Ruveeha: Hyper-V memory troubleshooting](docs/memory-troubleshooting.md)

## Evidence

[Browse the Phase 1 evidence index](evidence/README.md). Six real screenshots document server health, the memory investigation and fix, and the merged documentation PRs.

## GitHub collaboration

- [PR #1 — Ubuntu baseline](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/1)
- [PR #2 — Rocky Linux baseline and troubleshooting](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/2)

## What we learned

Linux administration involves verifying system state, understanding how virtualization affects guests, troubleshooting with evidence, documenting changes, and peer-reviewing work.
