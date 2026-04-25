# Mini-SOC Architecture

## Overview

This lab is a small SOC environment built in VMware. The goal is simple: generate Windows security activity, send it to Wazuh, and validate detections from the SIEM side.

The setup has one Wazuh server and one Windows endpoint. The Windows endpoint sends logs to the Wazuh server, and I review the events in the Wazuh dashboard under Threat Hunting.

## Lab Components

| Component | Role | IP Address |
|---|---|---|
| Ubuntu Server | Wazuh manager, indexer, and dashboard | `192.168.152.128` |
| Windows-11-Lab | Monitored Windows endpoint | `192.168.152.129` |
| VMware NAT / VMnet8 | Private lab network | `192.168.152.0/24` |

## Wazuh Server

The Wazuh server runs the main SIEM components:

- Wazuh manager
- Wazuh indexer
- Wazuh dashboard
- Wazuh API

Key validation from Phase 1:

- Services were running.
- Required ports were listening.
- Dashboard was reachable at `https://192.168.152.128`.
- Dashboard access was recovered after a network path issue.

## Windows Endpoint

The endpoint is a Windows 11 VM named `Windows-11-Lab`.

Endpoint details:

- Agent ID: `001`
- Agent group: `default`
- IP address: `192.168.152.129`
- Agent status: active
- Wazuh agent version observed: `v4.14.3`

This endpoint is where I generate controlled security activity for Phase 3 detections.

## Key Ports

| Port | Purpose |
|---|---|
| `1514` | Agent event ingestion |
| `1515` | Agent registration |
| `55000` | Wazuh API |
| `443` | Wazuh dashboard |

## Current Scope

Current lab scope:

- One Wazuh server
- One Windows 11 endpoint
- One active Wazuh agent
- Windows authentication and account-management detections
- Incident-style writeups for activity that tells a useful investigation story

Current Phase 3 status:

- 4 of 8 planned detections complete
- 2 incident reports started
- Remaining work focuses on more Windows activity, evidence capture, and detection tuning

## Assumptions

- This is a controlled lab, not production.
- Both VMs are on VMware NAT / VMnet8.
- The Windows endpoint can reach the Wazuh server.
- Wazuh dashboard access happens over HTTPS.
- Events are validated using endpoint name, agent ID, event sequence, Windows Event IDs, and Wazuh rule IDs.
- Windows and Wazuh may show different timestamp formats or time zones, so timestamps are not the only validation method.

## Why This Architecture Works

This setup is small, but it is enough to practice the core SOC workflow:

1. Generate activity on an endpoint.
2. Confirm the event locally.
3. Confirm the event centrally in Wazuh.
4. Capture evidence.
5. Write the detection.
6. Write an incident report when the activity has a meaningful story.

Start simple, work up. This architecture keeps the lab focused while still giving enough telemetry to practice real detection and triage habits.
