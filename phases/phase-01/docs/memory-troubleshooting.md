# Hyper-V Memory Troubleshooting — backup01

[← Phase 01](../README.md) · [Setup guide](setup-guide.md) · [Evidence coverage](evidence-coverage.md)

## Incident summary

| Field | Recorded observation |
|---|---|
| Server | `backup01` — Rocky Linux 9.8 Minimal on Hyper-V |
| Symptom | Approximately 609 MiB RAM visible in the guest despite 4 GiB startup memory |
| Related evidence | Repeated `hv_balloon` balloon-floor warnings; Dynamic Memory enabled |
| Corrective action | Shut down the VM, disable Dynamic Memory, retain 4 GiB startup memory, and restart |
| Verified result | Approximately 3.6 GiB RAM, approximately 3.1 GiB available, zero swap usage, and zero failed systemd units |
| Status | Resolved for the observed guest-memory symptom |

The reduced memory could affect workloads and package management. No measured application outage or performance loss was preserved, so this report does not claim one.

## Investigation

| Check | Observation | Interpretation |
|---|---|---|
| Guest memory | 609 MiB total | Large shortfall compared with the configured startup allocation |
| Kernel messages | Repeated balloon-floor warnings | Relevant Hyper-V balloon behavior; not a complete explanation of host memory decisions |
| Host memory settings | Dynamic Memory True; Startup 4294967296 bytes | The host could adjust the guest allocation |
| Post-change guest check | Approximately 3.6 GiB total | Expected allocation restored after using fixed memory |

### Guest commands — Rocky Linux

```bash
free -h
grep -E 'MemTotal|MemAvailable|SwapTotal' /proc/meminfo
sudo dmesg | grep -iE 'balloon|memory hotplug|hot-add' | tail -20
```

- `free -h` summarizes RAM and swap; inspect total and available memory.
- The `grep` command selects detailed memory counters from `/proc/meminfo`.
- `sudo dmesg` reads the kernel ring buffer; the filters select memory-related messages and retain the last 20 matching lines. A filtered view can omit context, so review surrounding messages when needed.

### Host command — Windows PowerShell (Administrator)

```powershell
Get-VMMemory -VMName "backup01" |
    Format-List DynamicMemoryEnabled,Startup,Minimum,Maximum,Assigned,MemoryDemand
```

`Get-VMMemory` inspects VM memory settings; `Format-List` makes the selected properties readable. The before screenshot establishes Dynamic Memory and startup settings, but does not preserve a full host memory-pressure history.

## Corrective action

The VM was shut down before changing memory settings.

```powershell
Get-VM -Name "backup01" | Select-Object Name,State
Set-VMMemory -VMName "backup01" -DynamicMemoryEnabled $false -StartupBytes 4GB
Get-VMMemory -VMName "backup01" |
    Format-List DynamicMemoryEnabled,Startup,Assigned
```

| Command | Purpose | Recorded outcome |
|---|---|---|
| `Get-VM` with `Select-Object` | Inspect VM power state | Off before the change |
| `Set-VMMemory` | Change the stopped VM to fixed 4 GiB startup memory | Configuration changed |
| `Get-VMMemory` | Verify the resulting settings | DynamicMemoryEnabled False; Startup 4294967296 bytes |

These commands run on the Windows Hyper-V host. The change command modifies configuration; it is not an inspection command.

The VM was then started again. Its start command and boot timeline are not preserved in the six-image evidence set.

## Post-fix verification

Run inside the Rocky Linux guest:

```bash
free -h
systemctl --failed --no-pager
```

`free -h` checks the resulting memory allocation. `systemctl --failed --no-pager` inspects current failed-unit state without a pager.

| Result | Recorded reading |
|---|---|
| Total RAM | Approximately 3.6 GiB |
| Available RAM | Approximately 3.1 GiB |
| Swap | 2 GiB total; zero used |
| Failed units | 0 loaded units listed |

This verifies the observed memory symptom and current systemd failure state. It is not a complete application, performance, or security acceptance test.

## Cause assessment and remaining unknowns

The before/after evidence supports an association between Hyper-V Dynamic Memory behavior and the reduced allocation. Disabling Dynamic Memory and restarting restored the expected guest-visible memory.

The evidence does **not** establish why the host reduced allocation so far. Host memory pressure, demand readings over time, and other host-side factors were not captured. The change and restart occurred together, so this record does not isolate their individual effects through a controlled comparison.

### Optional follow-up investigation — not performed

For a future recurrence, capture a timestamped guest reading, full relevant kernel context, host available memory, and VM assigned/demand/minimum/maximum settings **before** changing configuration. Repeat after the change and correlate the times. Do not re-enable Dynamic Memory on the working VM solely to recreate the incident.

## Lessons learned

- Compare guest observations with host configuration.
- Treat a successful fix as evidence of recovery while keeping cause claims proportionate.
- Distinguish configured memory, assigned memory, and guest-visible memory.
- Verify a change and preserve its outcome.
- Keep useful incident context rather than only the final healthy state.

## Evidence

| Stage | Original screenshot |
|---|---|
| Guest before | [Memory and balloon warnings](../evidence/ruveeha-memory-before.png) |
| Host before | [Dynamic Memory settings](../evidence/ruveeha-hyperv-before.png) |
| Host change and check | [Dynamic Memory disabled](../evidence/ruveeha-hyperv-after.png) |
| Guest after | [Restored memory and zero failed units](../evidence/ruveeha-memory-after.png) |

[Browse scaled previews and full-size originals](../evidence/README.md).
