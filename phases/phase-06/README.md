<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../docs/assets/linuxops-logo-dark.png">
  <img src="../../docs/assets/linuxops-logo-light.png" alt="LinuxOps Enterprise Lab" width="260">
</picture>

# Phase 06 — Backups & Restore Testing

**Status: Planned**

[Project overview](../../README.md) · [Previous: Phase 05](../phase-05/README.md) · [Next: Phase 07](../phase-07/README.md)

</div>

---

## Overview

Develop and test backup and restore procedures for lab data.

## Objectives and planned tasks

| Area | Planned task | Intended verification |
|---|---|---|
| Backup design | Define lab data, destinations, and backup scope. | Document what is included. |
| Backup execution | Practice a repeatable backup procedure. | Inspect backup output and errors. |
| Restore testing | Restore selected lab data. | Verify recovered contents. |

These activities describe intended work. Commands and detailed procedures will be added when this phase begins.

## Acceptance criteria — planned

These are requirements for future verification, not completed test results.

| Check | Required evidence for completion |
|---|---|
| Backup scope | Record source paths, exclusions, destination, timestamp, and the backup command's exit status. |
| Restore contents | Restore to a separate test directory; compare relative path sets and SHA-256 hashes of every included regular test file. |
| Metadata | Where metadata preservation is in scope, compare ownership, modes, and symbolic-link targets. |
| Recovery result | Record restore duration and investigate every missing/mismatched item; do not mark success from backup creation alone. |

## Contributor responsibilities

| Contributor | Lab server | Planned responsibility |
|---|---|---|
| [Haziq](https://github.com/HaziqBinAfzal) | Ubuntu `web01` | Perform the phase activities, document results, and review Ruveeha's contribution. |
| [Ruveeha](https://github.com/ruveeha33) | Rocky Linux `backup01` | Perform the phase activities, document results, and review Haziq's contribution. |

## Results

Not started. No completion or validation claims are made for this phase.

## Documentation and evidence

Technical notes and selected screenshots will be added here after work is performed and verified. No evidence is available yet.

## Completion checklist

- [ ] Perform the planned lab activities.
- [ ] Verify and document actual results.
- [ ] Record meaningful troubleshooting, if needed.
- [ ] Add selected evidence with sensitive information redacted.
- [ ] Submit a pull request and complete peer review.
