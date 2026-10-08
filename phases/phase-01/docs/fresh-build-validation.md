# Phase 01 — Fresh-build Validation Record

[← Phase 01](../README.md) · [Setup guide](setup-guide.md) · [Evidence coverage](evidence-coverage.md)

**Status: Not executed**

This is a checklist and blank record for a future test. No execution, pass result, peer review, or new technical accomplishment is claimed. Complete it only after following the setup guide on a new disposable VM.

## Run identity

| Field | Value to record |
|---|---|
| Contributor and reviewer | Pending |
| Start/end time and timezone | Pending |
| Guide commit tested | Pending |
| Windows edition and Hyper-V availability | Pending |
| ISO filename, release, official source | Pending |
| SHA-256 and authenticated checksum source | Pending |
| VM name, generation, Secure Boot template | Pending |
| CPU/RAM/Dynamic Memory/disk/switch settings | Pending |
| Actual guest OS and running kernel | Pending |
| Account and effective sudo policy | Pending |
| Storage layout and deviations | Pending |

## Execution checks

| Check | Required observation | Current result |
|---|---|---|
| Media | SHA-256 matches the exact ISO's published checksum; checksum authenticity handled using distribution instructions | Not run |
| Boot | Selected compatible ISO boots in the new VM | Not run |
| Installation | Guest boots from its VHDX after ISO detachment | Not run |
| Identity | Hostname, account, OS, kernel, and virtualization match the intended new build | Not run |
| Privileges | sudo whoami prints root and sudo -l shows the intended policy | Not run |
| Updates | Package transaction outcome retained; errors/deferred items recorded | Not run |
| Reboot | Guest restarts and running kernel is recorded | Not run |
| SSH | Intended host authenticates to the intended guest; public host-key fingerprint checked | Not run |
| Resources | CPU, RAM, swap, storage, and mount output recorded and compared with settings | Not run |
| systemd | Failed-unit check recorded; any listed failure investigated | Not run |
| Git handoff | Reviewed record committed/pushed to the confirmed destination and PR opened | Not run |
| Peer reproduction | Reviewer follows the relevant steps and records ambiguities/deviations | Not run |

## Retained output

Add dated, redacted text output or links after execution. At minimum retain identity, effective sudo policy, addresses/listeners, storage, health checks, and package transaction outcome. Do not copy expected examples into this section as if they were observed.

No output recorded yet.

## Deviations and corrections

| Guide step | Actual behavior or error | Resolution | Repeat-check outcome |
|---|---|---|---|
| Pending | No execution recorded | Pending | Not run |

## Selected screenshot evidence

Choose one useful health capture per new server and any meaningful troubleshooting before/after. Link new images only after they exist, and distinguish their date/build from the historical screenshots.

No new screenshots recorded.

## Final decision

- [ ] All required checks were executed and actual outcomes recorded.
- [ ] Any unresolved item is explicitly identified.
- [ ] Guide corrections are documented.
- [ ] Reviewer confirmed the evidence and scope.

**Decision: Pending.** Documentation review alone does not turn this into a passed build.
