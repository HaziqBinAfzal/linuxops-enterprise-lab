<p align="center">
<img src="../../../docs/assets/linuxops-enterprise-lab-logo.png" alt="LinuxOps Enterprise Lab logo" width="280">
</p>

# Phase 01 — Evidence Coverage

[← Phase 01](../README.md) · [Screenshot index](../evidence/README.md) · [Setup guide](setup-guide.md)

## Evidence categories

**Screenshot-backed** means the selected image directly shows the stated observation. **Documented observation** means a contributor's baseline records it, but the six selected screenshots do not independently show it. **Reconstruction guidance** describes a new-build procedure and is not a historical accomplishment.

## Coverage matrix

| Observation | Supporting record | Coverage |
|---|---|---|
| web01: 2 available CPUs, approximately 3.3 GiB RAM, 3.8 GiB swap, ext4 root of approximately 61 GiB and 13% usage, zero failed units | [Ubuntu health](../evidence/haziq-web01-health.png) | Screenshot-backed |
| backup01: 609 MiB RAM and repeated hv_balloon floor warnings | [Guest before](../evidence/ruveeha-memory-before.png) | Screenshot-backed |
| Hyper-V Dynamic Memory enabled; startup setting 4294967296 bytes | [Host before](../evidence/ruveeha-hyperv-before.png) | Screenshot-backed |
| VM Off before change; Dynamic Memory disabled; startup setting 4294967296 bytes | [Host after](../evidence/ruveeha-hyperv-after.png) | Screenshot-backed |
| backup01: approximately 3.6 GiB RAM, 2 GiB swap, zero failed units | [Guest after](../evidence/ruveeha-memory-after.png) | Screenshot-backed |
| Documentation PRs #1 and #2 approved and merged | [GitHub screenshot](../evidence/github-merged-prs.png) | Screenshot-backed; approvals/merges also checked through GitHub |
| Exact OS releases and kernel strings | [Ubuntu baseline](web01-baseline.md), [Rocky baseline](backup01-baseline.md) | Documented observations; not independently visible in these selected images |
| sudo/wheel membership and effective sudo policy | Both baseline records | Documented observations |
| Full SSH service configuration and access procedure | Both baseline records | Documented observations; the Rocky after screenshot contains login context, but is not a configuration audit |
| Package updates and running updated kernel | Both baseline records | Documented observations; update transcripts are not included |
| Rocky CPU count and detailed LVM/XFS layout | [Rocky baseline](backup01-baseline.md) | Documented observations |
| Exact ISO/checksum, original VM generation, and installer choices | No preserved record in this evidence set | Unknown; do not infer |
| Steps in the setup guide | [Setup guide](setup-guide.md) | Reconstruction guidance, not a verified new run |

## Interpretation limits

Zero failed systemd units means no units were in the failed state during that check. It does not establish complete service functionality, security, or performance.

The memory before/after evidence supports the assessment that changing Hyper-V Dynamic Memory restored the expected allocation. It does not establish every underlying host scheduling or memory-pressure factor.

Kernel versions, network addresses, uptime, utilization, and package state may change after the baseline. New output should be dated and clearly distinguished from historical observations.

## Closing evidence gaps

For a future verification run, retain selected text output for identity, sudo policy, network/SSH checks, storage, and package transaction outcomes. Record collection time and contributor, redact sensitive details, and submit the record for peer review. These are follow-up verification tasks; they have not been completed by this documentation change.
