# Phase 1 — Selected evidence

These are screenshots from the LinuxOps Enterprise Lab training environment. The related reports are in [`docs/phase-01`](../../docs/phase-01/).

| Screenshot | Verification |
|---|---|
| [haziq-web01-health.png](haziq-web01-health.png) | Ubuntu `web01` uptime, RAM, CPU, filesystem usage, and systemd health. |
| [ruveeha-memory-before.png](ruveeha-memory-before.png) | Rocky Linux initially reporting 609 MiB guest-visible RAM and `hv_balloon` warnings. |
| [ruveeha-hyperv-before.png](ruveeha-hyperv-before.png) | Hyper-V Dynamic Memory enabled before the fix. |
| [ruveeha-hyperv-after.png](ruveeha-hyperv-after.png) | PowerShell change disabling Dynamic Memory and confirming the setting. |
| [ruveeha-memory-after.png](ruveeha-memory-after.png) | Rocky Linux reporting approximately 3.6 GiB RAM and zero failed systemd units after the fix. |
| [github-merged-prs.png](github-merged-prs.png) | Phase 1 documentation PRs approved and merged. |

**Note:** Screenshot images are being added separately. Links above will work once the PNG files are committed.

Read the [Hyper-V memory troubleshooting report](../../docs/phase-01/memory-troubleshooting.md) for the investigation and corrective action.
