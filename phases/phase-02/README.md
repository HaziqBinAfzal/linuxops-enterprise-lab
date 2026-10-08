<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../docs/assets/linuxops-logo-dark.png">
  <img src="../../docs/assets/linuxops-logo-light.png" alt="LinuxOps Enterprise Lab" width="260">
</picture>

# Phase 02 — Users, Groups & Permissions

**Status: Planned**

[Project overview](../../README.md) · [Previous: Phase 01](../phase-01/README.md) · [Next: Phase 03](../phase-03/README.md)

</div>

---

## Overview

Practice Linux accounts, group membership, file ownership, chmod, umask, sudo policies, and ACLs.

## Objectives and planned tasks

| Area | Planned task | Intended verification |
|---|---|---|
| Accounts and groups | Create lab users and groups; inspect membership and account information. | Verify identity and group membership. |
| Ownership and permissions | Practice ownership, chmod, umask, and ACLs on lab files. | Compare expected and actual access. |
| Administrative access | Review sudo policies using safe configuration checks. | Verify authorized access and denied actions. |

These activities describe intended work. Commands and detailed procedures will be added when this phase begins.

## Acceptance criteria — planned

These are requirements for future verification, not completed test results.

| Check | Required evidence for completion |
|---|---|
| Identity and membership | For a new lab user, record id output and confirm the intended primary/supplementary groups. |
| Permissions and ACLs | Test the same file as an allowed and a denied lab user; record successful access, permission-denied output, and exit status. |
| umask | Create a file and directory under a chosen umask and compare resulting modes with the intended values. |
| sudo policy | Validate any edited policy with visudo; record an explicitly permitted command and a disallowed command for the test account. |

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
