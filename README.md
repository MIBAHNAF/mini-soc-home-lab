# Mini-SOC Home Lab

## Overview

This project is a small Security Operations Center (SOC) lab built with Wazuh. I built it to practice the full workflow: deploy a SIEM, onboard a Windows endpoint, generate security activity, validate detections, capture evidence, and write incident-style reports.

This README is the broad project overview. For the detailed build steps, exact commands, troubleshooting notes, and working journal, see [`lab-notes.md`](lab-notes.md).

## Why I Built This

I wanted hands-on practice with real security operations work, not just theory. This lab lets me test Windows telemetry, Wazuh rules, detection logic, and incident documentation in a controlled environment.

The goal is not just to make alerts fire. The goal is to prove the full chain:

```text
Endpoint activity -> local logs -> Wazuh collection -> Wazuh dashboard -> detection report -> incident report
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
| Phase 3 | Complete | Completed 8 Windows and Linux detections, captured evidence, and documented 3 incident-style investigation chains. |

## Detection Progress

Phase 3 progress: 8 of 8 planned detections complete.

| # | Detection | Status | Report |
|---|---|---|---|
| 1 | Windows failed login attempts | Complete | [`windows-failed-logins.md`](detections/windows-failed-logins.md) |
| 2 | Windows successful login after failed attempts | Complete | [`windows-success-after-failed-logins.md`](detections/windows-success-after-failed-logins.md) |
| 3 | Windows local user created | Complete | [`windows-local-user-created.md`](detections/windows-local-user-created.md) |
| 4 | Windows local administrator group change | Complete | [`windows-local-admin-group-change.md`](detections/windows-local-admin-group-change.md) |
| 5 | Windows PowerShell activity | Complete | [`windows-powershell-activity.md`](detections/windows-powershell-activity.md) |
| 6 | Linux failed login attempts | Complete | [`linux-failed-login-attempts.md`](detections/linux-failed-login-attempts.md) |
| 7 | Linux local user created | Complete | [`linux-local-user-created.md`](detections/linux-local-user-created.md) |
| 8 | Linux UFW / firewall change | Complete | [`linux-firewall-change.md`](detections/linux-firewall-change.md) |

## Incident Reports

| # | Incident Report | Status |
|---|---|---|
| 001 | [`Failed Logins Followed by Successful Local Logon`](incident-reports/incident-001-windows-failed-logins.md) | Closed - lab validation successful |
| 002 | [`Local Account Creation, PowerShell Enumeration, and Administrator Group Change`](incident-reports/incident-002-local-account-admin-change.md) | Closed - lab validation successful |
| 003 | [`Linux Failed Authentication, Local User Creation, and Firewall Configuration Change`](incident-reports/incident-003-linux-auth-user-firewall.md) | Closed - lab validation successful |

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
- Linux failed login attempts: `screenshots/26-*` and `screenshots/27-*`
- Linux local user creation: `screenshots/28-*` and `screenshots/29-*`
- Linux UFW firewall change: `screenshots/30-*` through `screenshots/32-*`

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
- Linux authentication and firewall-change monitoring
- Evidence collection
- Incident-style reporting
- Troubleshooting across network, endpoint, and SIEM layers

## Resume Summary

Built a Mini-SOC home lab using Wazuh, Ubuntu, VMware, and a Windows 11 endpoint. Deployed and validated SIEM infrastructure, onboarded a Windows agent, generated controlled Windows and Linux security events, created detection reports, and wrote SOC-style incident reports for authentication failures, account creation, PowerShell enumeration, privilege changes, and firewall configuration changes.

## Current Status

Core project status: complete.

Phase 1, Phase 2, and Phase 3 are complete. The lab includes 8 validated detections and 3 incident reports documenting the strongest activity chains.

Optional future work:

- Keep refining detections with better thresholds and context.
- Add future detections or incident chains if the lab is expanded.
