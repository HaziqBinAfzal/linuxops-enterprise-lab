# Hyper-V Memory Troubleshooting - backup01

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

SS-01-Ruveeha-Rocky-Memory.png: Guest memory before the fix.

SS-02-Ruveeha-HyperV-Memory.png: Hyper-V memory configuration before the fix.

SS-03-Ruveeha-Fixed-Memory.png: Hyper-V configuration after disabling Dynamic Memory.

SS-04-Ruveeha-Memory-Verified.png: Guest memory and service health after the fix.
