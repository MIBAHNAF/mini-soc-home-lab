# Lab Notes

These notes are the working record for the Mini-SOC lab. The README explains the project at a higher level. This file tracks what I did, what I validated, what broke or mattered, and what still needs work.

## Current Status

As of 2026-04-25, Phase 1 and Phase 2 are complete. Phase 3 is in progress.

Current state:

- Wazuh server is deployed and reachable.
- Windows endpoint `Windows-11-Lab` is onboarded and active.
- Windows events are reaching Wazuh Threat Hunting.
- Four of eight planned detections are complete.
- Two incident-style reports are started.
- Remaining Phase 3 work: four detections plus any follow-up incident reports that make sense.

Lab systems:

- Wazuh server IP: `192.168.152.128`
- Windows endpoint IP: `192.168.152.129`
- Agent ID: `001`
- Agent name: `Windows-11-Lab`
- Agent group: `default`

---

## Phase 1 - Wazuh SIEM Deployment

## Date

2026-04-21

## What I Did

I deployed the Wazuh SIEM server on Ubuntu, verified the core services, checked the required ports, and confirmed dashboard access.

The install needed one important adjustment: Ubuntu 24.04 was not officially accepted by the Wazuh installer check, so I used the `-i` flag to bypass OS validation for this lab.

## Commands Used

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl unzip net-tools -y
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a -i
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
sudo netstat -tulnp | grep -E "1514|1515|55000|443"
```

## What I Validated

- `wazuh-manager` was running.
- `wazuh-indexer` was running.
- `wazuh-dashboard` was running.
- Required ports were listening.
- Dashboard was reachable at `https://192.168.152.128`.
- Initial dashboard state showed no agents yet.

## Issue Encountered

Dashboard access was interrupted by a routing or network path issue during initial validation.

## What I Checked

Start simple, work up:

- Confirmed the Wazuh services were running locally.
- Confirmed the expected ports were listening.
- Checked NAT networking and reachability between the host and VM.
- Retested dashboard access after correcting the network path.

## Result

Phase 1 was successful. The Wazuh server was operational, the dashboard was reachable, and the lab was ready for endpoint onboarding.

## Evidence Captured

- `screenshots/01-vm-specifications.png`
- `screenshots/02-ubuntu-ip-address.png`
- `screenshots/03-wazuh-install-success.png`
- `screenshots/04a-wazuh-indexer-running.png`
- `screenshots/04b-wazuh-manager-running.png`
- `screenshots/04c-wazuh-dashboard-running.png`
- `screenshots/05-wazuh-open-ports.png`
- `screenshots/06-dashboard-login.png`
- `screenshots/07-dashboard-overview.png`

## Notes

The main lesson was that service health alone is not enough. I had to check services, ports, network reachability, and dashboard access together.

---

## Phase 2 - Windows Endpoint Onboarding

## Date

2026-04-22

## What I Did

I created a Windows 11 endpoint VM, placed it on the same VMware NAT network as the Wazuh server, installed the Wazuh Windows agent, and confirmed that the endpoint reported into Wazuh.

## Endpoint Details

- Endpoint OS: Windows 11
- VM name: `Windows-11-Lab`
- Network: NAT / VMnet8
- Windows IP: `192.168.152.129`
- Wazuh server IP: `192.168.152.128`
- Endpoint VM resources: 64 GB disk, 4 GB RAM, 2 CPU cores

## Commands Used

```bat
ipconfig
ping 192.168.152.128
```

```powershell
NET START Wazuh
```

## What I Validated

- Windows VM could reach the Wazuh server.
- Wazuh dashboard opened from the Windows VM.
- Wazuh agent installed successfully.
- Wazuh service started successfully.
- Agent `001` registered as `Windows-11-Lab`.
- Agent showed as active in the Wazuh dashboard.
- First Windows events appeared in Threat Hunting.

## Result

Phase 2 was successful. The endpoint was onboarded, connected, and sending Windows event data to Wazuh.

## Evidence Captured

- `screenshots/08-windows-vm-specifications.png`
- `screenshots/09-windows-network-connectivity.png`
- `screenshots/10-windows-agent-deploy-page.png`
- `screenshots/11-windows-agent-installed.png`
- `screenshots/12-agent-visible-in-dashboard.png`
- `screenshots/13-first-windows-events.png`

## Notes

The important part was validating the full chain: endpoint network access, agent service startup, dashboard registration, and actual event ingestion. The phase was not done until events showed up in Wazuh.

---

## Phase 3 - Detection Engineering

## Date Started

2026-04-25

## Goal

Generate controlled Windows security activity, confirm it locally in Windows, confirm it centrally in Wazuh, then document the detection and any incident-style findings.

Same methodology each time:

1. Generate the activity.
2. Confirm the event locally on Windows.
3. Confirm the event in Wazuh.
4. Capture screenshots or exported evidence.
5. Write the detection report.
6. Write an incident report when the activity tells a bigger story.

Credential note:

- I do not store test passwords in this repo.
- When a command needs to show a password value, I use `<test-password>` instead.
- Screenshots and reports should prove the activity without exposing credentials.

## Detection 1 - Failed Login Attempts

What I tested:

- Locked the Windows endpoint.
- Entered the wrong PIN or password multiple times.
- Confirmed failed logons locally and in Wazuh.

Validated events:

- Windows Event ID `4625`
- Wazuh rule ID `60122`
- Rule description: `Logon Failure - Unknown user or bad password`
- Observed count: 4 failed logon events

Evidence:

- `screenshots/14a-windows-audit-policy-logon.png`
- `screenshots/14b-windows-failed-login-attempts.png`
- `screenshots/15-wazuh-failed-login-events.png`

Report:

- `detections/windows-failed-logins.md`

Result:

Wazuh detected the failed login attempts. This validates basic Windows authentication failure monitoring.

---

## Detection 2 - Successful Login After Failed Attempts

What I tested:

- Generated failed login attempts.
- Logged in successfully afterward.
- Confirmed the sequence in Windows and Wazuh.

Validated events:

- Windows Event ID `4625`
- Windows Event ID `4624`
- Wazuh rule ID `60122`
- Wazuh rule ID `67022`
- Wazuh rule ID `67028`

Evidence:

- `screenshots/16-windows-success-after-failures.png`
- `screenshots/17-wazuh-success-after-failures.png`

Reports:

- `detections/windows-success-after-failed-logins.md`
- `incident-reports/incident-001-windows-failed-logins.md`

Result:

This was treated as one incident with the failed logins because the successful logon happened right after the failed attempts. The useful story is the sequence, not just the single event.

---

## Detection 3 - Local User Created

What I tested:

- Created a local test user named `soc_test_user`.
- Confirmed the account existed locally.
- Confirmed Wazuh showed the account-related events.

Commands used:

```powershell
net user soc_test_user <test-password> /add
net user soc_test_user
```

Validated events:

- Windows Event ID `4720`
- Wazuh rule ID `60109`
- Wazuh rule ID `60110`
- Wazuh rule ID `60170`

Evidence:

- `screenshots/18a-windows-user-created.png`
- `screenshots/18b-windows-user-created.png`
- `screenshots/19-wazuh-user-created-event.png`

Report:

- `detections/windows-local-user-created.md`

Result:

Wazuh detected the local user creation and related account changes. This matters because unexpected local account creation can be persistence.

---

## Detection 4 - Local Administrator Group Change

What I tested:

- Added `soc_test_user` to the local Administrators group.
- Confirmed membership locally.
- Confirmed Wazuh raised the higher-severity administrator group change alert.

Commands used:

```powershell
net localgroup Administrators soc_test_user /add
net localgroup Administrators
```

Validated events:

- Windows Event ID `4732`
- Wazuh rule ID `60154`
- Wazuh rule level `12`
- Rule description: `Administrators Group Changed`

Evidence:

- `screenshots/20a-windows-admin-group-change.png`
- `screenshots/20b-windows-admin-group-change.png`
- `screenshots/21-wazuh-privilege-change-event.png`
- `incident-reports/evidence/incident-002-local-account-admin-change-report.pdf`

Reports:

- `detections/windows-local-admin-group-change.md`
- `incident-reports/incident-002-local-account-admin-change.md`

Result:

This was treated as one incident with local account creation because the account was created and then given administrator rights. That is a stronger security story than either event by itself.

---

## Time Zone Note

During Phase 3 testing, the Windows VM and Wazuh dashboard displayed timestamps in different formats and time zones. I did not rely on timestamp display alone.

For validation, I correlated events using:

- Event sequence
- Endpoint name: `Windows-11-Lab`
- Agent ID: `001`
- Event or rule description
- Windows Event IDs
- Wazuh rule IDs

This kept the validation clean even when Windows Event Viewer and Wazuh did not visually match one-to-one.

---

## Current Phase 3 Progress

Completed detections:

- `detections/windows-failed-logins.md`
- `detections/windows-success-after-failed-logins.md`
- `detections/windows-local-user-created.md`
- `detections/windows-local-admin-group-change.md`

Incident reports:

- `incident-reports/incident-001-windows-failed-logins.md`
- `incident-reports/incident-002-local-account-admin-change.md`

Still left:

- Complete the remaining four planned detections.
- Capture screenshots and exported evidence for each test.
- Add incident reports only when the activity tells a useful investigation story.
- Keep detection reports focused on logic and validation.
- Keep incident reports focused on what happened, what I saw, what I did, what it means, and what comes next.
