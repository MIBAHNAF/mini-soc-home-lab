# Mini-SOC Home Lab

## Overview

This project is a small Security Operations Center (SOC) lab built with Wazuh. I built it to practice the full workflow: deploy a SIEM, onboard a Windows endpoint, generate security activity, validate detections, capture evidence, and write incident-style reports.

This README is the broad project overview. For the detailed build steps, exact commands, troubleshooting notes, and working journal, see [`lab-notes.md`](lab-notes.md).

## Why I Built This

I wanted hands-on practice with real security operations work, not just theory. This lab lets me test Windows telemetry, Wazuh rules, detection logic, and incident documentation in a controlled environment.

The goal is not just to make alerts fire. The goal is to prove the full chain:

```text
Windows activity -> Windows Event Log -> Wazuh agent -> Wazuh dashboard -> detection report -> incident report
```

## Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware |
| SIEM | Wazuh |
| Server OS | Ubuntu 24.04 LTS |
| Endpoint OS | Windows 11 |
| Network | NAT / VMnet8 |
| Wazuh server IP | `192.168.152.128` |
| Windows endpoint IP | `192.168.152.129` |
| Windows agent name | `Windows-11-Lab` |
| Windows agent ID | `001` |

## Repository Structure

```text
mini-soc-home-lab/
|-- architecture/
|-- detections/
|-- incident-reports/
|-- screenshots/
|-- lab-notes.md
`-- README.md
```

## Architecture

Architecture docs:

- [`architecture/README.md`](architecture/README.md)
- [`architecture/network-and-data-flow.md`](architecture/network-and-data-flow.md)

High-level flow:

```text
Windows endpoint
    -> Wazuh agent
    -> Wazuh manager / indexer
    -> Wazuh dashboard
    -> Detection and incident documentation
```

Key Wazuh ports:

| Port | Purpose |
|---|---|
| `1514` | Agent event ingestion |
| `1515` | Agent registration |
| `55000` | Wazuh API |
| `443` | Wazuh dashboard |

## Phase Summary

| Phase | Status | Summary |
|---|---|---|
| Phase 1 | Complete | Deployed Wazuh, validated services, confirmed ports, and recovered dashboard access after a network path issue. |
| Phase 2 | Complete | Built and onboarded the Windows 11 endpoint, confirmed active agent status, and validated first Windows event ingestion. |
| Phase 3 | In progress | Generating Windows security activity, writing detection reports, and documenting incident-style findings. |

## Detection Progress

Phase 3 progress: 5 of 8 planned detections complete.

| # | Detection | Status | Report |
|---|---|---|---|
| 1 | Windows failed login attempts | Complete | [`windows-failed-logins.md`](detections/windows-failed-logins.md) |
| 2 | Windows successful login after failed attempts | Complete | [`windows-success-after-failed-logins.md`](detections/windows-success-after-failed-logins.md) |
| 3 | Windows local user created | Complete | [`windows-local-user-created.md`](detections/windows-local-user-created.md) |
| 4 | Windows local administrator group change | Complete | [`windows-local-admin-group-change.md`](detections/windows-local-admin-group-change.md) |
| 5 | Windows PowerShell activity | Complete | [`windows-powershell-activity.md`](detections/windows-powershell-activity.md) |
| 6 | Linux failed login attempts | Planned | `detections/linux-failed-login-attempts.md` |
| 7 | Linux local user created | Planned | `detections/linux-local-user-created.md` |
| 8 | Linux UFW / firewall change | Planned | `detections/linux-firewall-change.md` |

## Incident Reports

| # | Incident Report | Status |
|---|---|---|
| 001 | [`Failed Logins Followed by Successful Local Logon`](incident-reports/incident-001-windows-failed-logins.md) | Closed - lab validation successful |
| 002 | [`Local Account Creation and Administrator Group Change`](incident-reports/incident-002-local-account-admin-change.md) | Closed - lab validation successful |

## Evidence Index

Evidence is stored in the `screenshots/` directory and linked directly from each detection or incident report.

Key evidence groups:

- Phase 1 Wazuh deployment: `screenshots/01-*` through `screenshots/07-*`
- Phase 2 Windows onboarding: `screenshots/08-*` through `screenshots/13-*`
- Failed login detection: `screenshots/14*` and `screenshots/15-*`
- Successful login after failures: `screenshots/16-*` and `screenshots/17-*`
- Local user creation: `screenshots/18*` and `screenshots/19-*`
- Local administrator group change: `screenshots/20*` and `screenshots/21-*`
- PowerShell activity: `screenshots/22-*` through `screenshots/25-*`
- Planned Linux detections: `screenshots/26-*` through `screenshots/31-*`

Exported Wazuh reports are stored in:

```text
incident-reports/evidence/
```

## Time Zone Note

During Phase 3 testing, the Windows VM and Wazuh dashboard displayed timestamps in different formats and time zones. I validated events using event sequence, endpoint name, agent ID, event descriptions, Windows Event IDs, and Wazuh rule IDs.

This kept validation clean even when timestamps did not visually match one-to-one.

## Skills Demonstrated

- SIEM deployment and validation
- Wazuh manager, indexer, dashboard, and agent workflow
- Linux service and port validation
- Windows endpoint onboarding
- Windows Event Log analysis
- Detection engineering
- Authentication and account-management monitoring
- Evidence collection
- Incident-style reporting
- Troubleshooting across network, endpoint, and SIEM layers

## Resume Summary

Built a Mini-SOC home lab using Wazuh, Ubuntu, VMware, and a Windows 11 endpoint. Deployed and validated SIEM infrastructure, onboarded a Windows agent, generated controlled security events, created detection reports, and wrote SOC-style incident reports for failed logins, successful logon after failures, local user creation, and administrator group changes.

## Current Status

Phase 1 and Phase 2 are complete. Phase 3 is in progress with 5 of 8 planned detections complete.

Next work:

- Complete Linux failed login attempts.
- Complete Linux local user creation.
- Complete Linux UFW / firewall change.
- Continue capturing screenshots and Wazuh evidence.
- Add incident reports when the activity tells a useful investigation story.
- Keep refining detections with better thresholds and context.
