<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../docs/assets/linuxops-logo-dark.png">
  <img src="../../docs/assets/linuxops-logo-light.png" alt="LinuxOps Enterprise Lab" width="260">
</picture>

# Phase 01 — Server Baseline & Initial Administration

**Status: Completed**

[Project overview](../../README.md) · [Objectives](#objectives) · [Results](#verified-results) · [Documentation](#documentation) · [Evidence](#evidence) · [Next: Phase 02](../phase-02/README.md)

</div>

---

> [!NOTE]
> The phase records completed baseline work. Screenshots independently support selected health and memory checks; other details are contributor observations. The reconstruction guide still awaits a fresh-build validation.

## Start here

| Goal | Document |
|---|---|
| Recreate the environment | [Setup guide](docs/setup-guide.md) · [Pending validation](docs/fresh-build-validation.md) |
| Compare the two servers | [Ubuntu baseline](docs/web01-baseline.md) · [Rocky Linux baseline](docs/backup01-baseline.md) |
| Follow the memory investigation | [Incident report](docs/memory-troubleshooting.md) |
| Check supporting proof | [Evidence index](evidence/README.md) · [Coverage limits](docs/evidence-coverage.md) |

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

## Verified results

| Area | Haziq — `web01` | Ruveeha — `backup01` |
|---|---|---|
| Virtualization | Hyper-V VM | Hyper-V VM |
| Virtual CPUs | 2 | 2 |
| Guest-visible memory | Approximately 3.3 GiB | Approximately 3.6 GiB after the fix |
| Root filesystem | ext4 on LVM | XFS on LVM |
| Administrative access | sudo and SSH verified | sudo and SSH verified |
| Service health | Zero failed systemd units | Zero failed systemd units |
| Troubleshooting | Baseline health documented | Investigated 609 MiB RAM and corrected Hyper-V Dynamic Memory |

Resource readings are point-in-time observations. The servers use separate Hyper-V Default Switch networks; inter-host VM connectivity is not yet established. Each contributor uses Ubuntu WSL for Git operations.

## Memory troubleshooting workflow

The investigation compared guest readings with host configuration and verified the result after changing Dynamic Memory.

```mermaid
flowchart TD
    A["Guest reports 609 MiB"] --> B["Inspect memory and hv_balloon warnings"]
    B --> C["Check Hyper-V memory configuration"]
    C --> D{"Dynamic Memory enabled"}
    D -->|"Observed: True"| E["Shut down VM and disable Dynamic Memory"]
    E --> F["Start VM and repeat health checks"]
    F --> G{"Expected memory restored"}
    G -->|"Observed: 3.6 GiB"| H["Document fix and zero failed units"]
    G -->|"If verification fails"| B
```

The successful path is recorded in the [incident report](docs/memory-troubleshooting.md). The retry branch describes how to continue investigation if a check fails; it is not an additional incident claim.

## Documentation

| Record | Purpose |
|---|---|
| [Setup and verification guide](docs/setup-guide.md) | Reconstruct a comparable lab with explained commands |
| [Fresh-build validation](docs/fresh-build-validation.md) | Unexecuted checklist; no successful rebuild claimed |
| [Ubuntu baseline](docs/web01-baseline.md) | Haziq's server observations and their interpretation |
| [Rocky Linux baseline](docs/backup01-baseline.md) | Ruveeha's server observations and their interpretation |
| [Memory troubleshooting](docs/memory-troubleshooting.md) | Symptoms, investigation, change, and post-fix checks |
| [Evidence coverage](docs/evidence-coverage.md) | Screenshot-backed checks and documented observations |

## Evidence

[Browse the Phase 1 evidence index](evidence/README.md). Six real screenshots document server health, the memory investigation and fix, and the merged documentation PRs.

## GitHub collaboration

- [PR #1 — Ubuntu baseline](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/1)
- [PR #2 — Rocky Linux baseline and troubleshooting](https://github.com/HaziqBinAfzal/linuxops-enterprise-lab/pull/2)

## Reproduction and evidence limits

The baseline documents record historical results. The setup guide describes how to construct a comparable new lab and validate it. Exact original ISO filenames/checksums, installer choices, VM generation, and installation/update transcripts were not preserved in the current selected evidence; those details must not be inferred from the screenshots.

## What we learned

Linux administration involves verifying system state, understanding how virtualization affects guests, troubleshooting with evidence, documenting changes, and peer-reviewing work.
