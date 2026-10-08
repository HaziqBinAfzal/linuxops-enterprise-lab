<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../../docs/assets/linuxops-logo-dark.png">
  <img src="../../docs/assets/linuxops-logo-light.png" alt="LinuxOps Enterprise Lab" width="260">
</picture>

# Phase 03 — SSH & Network Security

**Status: Planned**

[Project overview](../../README.md) · [Previous: Phase 02](../phase-02/README.md) · [Next: Phase 04](../phase-04/README.md)

</div>

---

## Overview

Verify secure remote administration, SSH configuration, host firewalls, and network diagnostics.

## Objectives and planned tasks

| Area | Planned task | Intended verification |
|---|---|---|
| SSH administration | Review SSH access and configuration. | Verify authorized remote access. |
| Host firewall | Practice rules for intended lab services. | Check allowed and blocked connections. |
| Network diagnostics | Inspect addresses, routes, listeners, and connectivity. | Document connectivity checks and limitations. |

These activities describe intended work. Commands and detailed procedures will be added when this phase begins.

## Acceptance criteria — planned

These are requirements for future verification, not completed test results.

| Check | Required evidence for completion |
|---|---|
| SSH access | Show a successful login as an authorized test account and a failed login for a deliberately invalid credential; retain corresponding server log context without secrets. |
| Configuration | Validate the daemon configuration with sshd -t and record relevant effective settings with sshd -T. |
| Firewall | From a stated reachable client, demonstrate the intended allowed port and a denied test port; document firewall state and distinguish denial from a stopped service. |
| Network scope | Record source/destination addresses and routing assumptions; do not claim inter-host connectivity without a successful test. |

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
