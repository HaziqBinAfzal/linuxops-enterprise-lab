# Hyper-V Memory Troubleshooting — backup01

[← Phase 01](../README.md) · [Setup guide](setup-guide.md) · [Evidence coverage](evidence-coverage.md)

## Incident Overview

Server: backup01

Operating System: Rocky Linux 9.8 Minimal

Virtualization Platform: Microsoft Hyper-V

Incident Type: Virtual Machine Memory Configuration

Status: Resolved

## Problem Description

The Rocky Linux virtual machine was configured with 4 GiB of startup memory in Hyper-V.

However, the guest operating system reported approximately 609 MiB of total RAM.

This was significantly lower than expected and could affect server performance, package management, and system services.

## Investigation

The `free -h` command was used to inspect available system memory.

Kernel messages showed repeated `hv_balloon` warnings, including messages indicating that the balloon floor had been reached.

The Hyper-V virtual machine configuration was then examined.

Dynamic Memory was found to be enabled, allowing Hyper-V to adjust the amount of physical memory assigned to the guest.

The configuration was identified as the likely cause of the unexpectedly low guest-visible memory.

## Investigation commands explained

| Environment | Command | Purpose and interpretation |
|---|---|---|
| Rocky Linux guest | `free -h` | Compare total guest RAM with the 4 GiB startup setting; the screenshot shows 609 MiB before the fix. |
| Rocky Linux guest | `grep -E 'MemTotal\|MemAvailable\|SwapTotal' /proc/meminfo` | Inspect detailed memory counters in kB; this corroborates the human-readable memory output. |
| Rocky Linux guest | `sudo dmesg \| grep -iE 'balloon\|memory hotplug\|hot-add' \| tail -20` | Filter the kernel ring buffer for relevant memory messages; repeated balloon-floor warnings are evidence of memory pressure/balloon behavior, not proof of every underlying cause. |
| Windows PowerShell (Administrator) | `Get-VMMemory -VMName "backup01" \| Format-List DynamicMemoryEnabled,Startup,Minimum,Maximum,Assigned,MemoryDemand` | Inspect host-side settings. The before screenshot shows Dynamic Memory enabled and startup memory of 4294967296 bytes. |

## Corrective Action

The `backup01` virtual machine was shut down.

The following command was executed in Windows PowerShell with administrative privileges:

    Set-VMMemory -VMName "backup01" -DynamicMemoryEnabled $false -StartupBytes 4GB

This disabled Dynamic Memory and configured the VM to use 4 GiB of startup memory.

The configuration was verified using:

    Get-VMMemory -VMName "backup01" | Format-List DynamicMemoryEnabled,Startup,Assigned

Hyper-V reported:

    DynamicMemoryEnabled : False
    Startup : 4294967296

The virtual machine was then started again.

## Change and verification commands explained

| Environment | Command | Purpose and expected result |
|---|---|---|
| Windows PowerShell (Administrator) | `Get-VM -Name "backup01" \| Select-Object Name,State` | Check VM state; the change screenshot shows Off before the memory change. |
| Windows PowerShell (Administrator) | `Set-VMMemory -VMName "backup01" -DynamicMemoryEnabled $false -StartupBytes 4GB` | Change the stopped lab VM to fixed 4 GiB startup memory. This modifies configuration; it is not a read-only check. |
| Windows PowerShell (Administrator) | `Get-VMMemory -VMName "backup01" \| Format-List DynamicMemoryEnabled,Startup,Assigned` | Confirm DynamicMemoryEnabled is False and Startup is 4294967296 bytes. Assigned may be unavailable while stopped. |
| Rocky Linux guest | `systemctl --failed --no-pager` | After boot, inspect failed units; zero listed is a current unit-state observation, not a complete application test. |

## Post-Fix Verification

After restarting Rocky Linux, the following commands were executed:

    free -h
    systemctl --failed --no-pager

The operating system reported approximately 3.6 GiB of total memory.

Approximately 3.1 GiB was available during verification.

Swap usage was zero.

No failed systemd units were reported.

The expected memory capacity was restored.

## Root Cause Assessment

The investigation linked the reduced guest-visible memory to Hyper-V Dynamic Memory behavior.

Disabling Dynamic Memory and restarting the virtual machine restored the expected memory allocation.

The observed before-and-after results support this assessment.

## Lessons Learned

Hypervisor memory settings can directly affect guest operating system performance.

Memory allocation should be verified from both the host and the guest.

Kernel warnings can provide useful troubleshooting evidence.

Configuration changes should be followed by verification.

Documenting the original problem and final result makes troubleshooting reproducible.

## Evidence

- [Guest memory before the fix](../evidence/ruveeha-memory-before.png)
- [Hyper-V memory configuration before the fix](../evidence/ruveeha-hyperv-before.png)
- [Hyper-V configuration after disabling Dynamic Memory](../evidence/ruveeha-hyperv-after.png)
- [Guest memory and service health after the fix](../evidence/ruveeha-memory-after.png)

[All Phase 1 screenshots](../evidence/README.md)
