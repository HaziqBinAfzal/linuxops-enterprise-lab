<p align="center">
<img src="../../../docs/assets/linuxops-enterprise-lab-logo.png" alt="LinuxOps Enterprise Lab logo" width="280">
</p>

# Phase 01 — Screenshot Evidence

[← Phase 01 overview](../README.md)

Six selected real Phase 1 screenshots are available below. These are point-in-time records from the lab and GitHub collaboration.

| Evidence | What it shows |
|---|---|
| [haziq-web01-health.png](haziq-web01-health.png) | Ubuntu server uptime, CPU, memory, filesystems, and systemd health |
| [ruveeha-memory-before.png](ruveeha-memory-before.png) | Rocky Linux guest showing 609 MiB RAM and Hyper-V balloon warnings |
| [ruveeha-hyperv-before.png](ruveeha-hyperv-before.png) | Hyper-V Dynamic Memory enabled |
| [ruveeha-hyperv-after.png](ruveeha-hyperv-after.png) | PowerShell change disabling Dynamic Memory |
| [ruveeha-memory-after.png](ruveeha-memory-after.png) | Guest showing approximately 3.6 GiB RAM and zero failed systemd units |
| [github-merged-prs.png](github-merged-prs.png) | Both Phase 1 documentation PRs approved and merged |

See the [memory troubleshooting report](../docs/memory-troubleshooting.md) for the detailed incident record.

## Coverage limits

The screenshots directly support selected resource readings, Hyper-V memory settings, and GitHub review/merge outcomes. They do not independently establish every OS/kernel, account, storage, SSH configuration, or package-update detail in the baselines. See the [evidence coverage matrix](../docs/evidence-coverage.md). No additional evidence has been fabricated.

## Screenshot previews

Previews are scaled to 640 pixels wide, preserving each image's proportions. Open the original for small text or a full-resolution view.

<details>
<summary>Haziq — Ubuntu server health</summary>

<a href="haziq-web01-health.png"><img src="haziq-web01-health.png" alt="Haziq — Ubuntu server health" width="640"></a>

[Open full-size screenshot](haziq-web01-health.png)

</details>

<details>
<summary>Ruveeha — guest memory before the fix</summary>

<a href="ruveeha-memory-before.png"><img src="ruveeha-memory-before.png" alt="Ruveeha — guest memory before the fix" width="640"></a>

[Open full-size screenshot](ruveeha-memory-before.png)

</details>

<details>
<summary>Ruveeha — Hyper-V settings before the fix</summary>

<a href="ruveeha-hyperv-before.png"><img src="ruveeha-hyperv-before.png" alt="Ruveeha — Hyper-V settings before the fix" width="640"></a>

[Open full-size screenshot](ruveeha-hyperv-before.png)

</details>

<details>
<summary>Ruveeha — Hyper-V settings after the fix</summary>

<a href="ruveeha-hyperv-after.png"><img src="ruveeha-hyperv-after.png" alt="Ruveeha — Hyper-V settings after the fix" width="640"></a>

[Open full-size screenshot](ruveeha-hyperv-after.png)

</details>

<details>
<summary>Ruveeha — guest memory and health after the fix</summary>

<a href="ruveeha-memory-after.png"><img src="ruveeha-memory-after.png" alt="Ruveeha — guest memory and health after the fix" width="640"></a>

[Open full-size screenshot](ruveeha-memory-after.png)

</details>

<details>
<summary>GitHub — approved and merged documentation PRs</summary>

<a href="github-merged-prs.png"><img src="github-merged-prs.png" alt="GitHub — approved and merged documentation PRs" width="640"></a>

[Open full-size screenshot](github-merged-prs.png)

</details>
