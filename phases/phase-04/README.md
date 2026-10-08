<div align="center">

<img src="../../docs/assets/linuxops-enterprise-lab-logo.png" alt="LinuxOps Enterprise Lab logo" width="360">

# Phase 04 — Services & systemd

**Status: Planned**

[Project overview](../../README.md) · [Previous: Phase 03](../phase-03/README.md) · [Next: Phase 05](../phase-05/README.md)

</div>

---

## Overview

Manage Linux services, startup behavior, logs, and a practice web service.

## Objectives and planned tasks

| Area | Planned task | Intended verification |
|---|---|---|
| Service lifecycle | Practice starting, stopping, and enabling services. | Check status and boot behavior. |
| systemd configuration | Inspect units and practice a lab service. | Validate unit configuration. |
| Service troubleshooting | Review logs and service failures. | Record a meaningful diagnosis and verification. |

These activities describe intended work. Commands and detailed procedures will be added when this phase begins.

## Acceptance criteria — planned

These are requirements for future verification, not completed test results.

| Check | Required evidence for completion |
|---|---|
| Lifecycle | Record active/inactive state after start/stop and enabled/disabled state after the relevant change. |
| Boot behavior | Reboot a disposable lab VM and verify the intended service starts. |
| Service function | For the chosen web service, show a successful local request with the expected response, plus a remote test only if connectivity is established. |
| Failure diagnosis | Capture a controlled failure, relevant journal entries, the fix, and a successful repeat check. |

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
