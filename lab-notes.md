# Lab Notes

## Date

2026-04-22

## Phase

Phase 2 - Windows Endpoint Onboarding

## Objective

Create a Windows endpoint VM, place it on the same VMware NAT network as the Wazuh server, install and start the Wazuh Windows agent, verify the agent registered successfully, and confirm that Windows event data was being ingested into Wazuh.

## Actions Taken

- Created a Windows 11 VM in VMware for endpoint monitoring.
- Configured the VM with 64 GB disk, 4 GB RAM, and 2 CPU cores.
- Connected the Windows VM to the same NAT / VMnet8 network as the Wazuh server.
- Verified network reachability from Windows to the Wazuh server using `ping`.
- Opened the Wazuh dashboard from the Windows VM to confirm connectivity.
- Used the Wazuh dashboard `Deploy new agent` workflow to generate the Windows onboarding steps.
- Installed the Wazuh Windows agent and assigned the agent name `Windows-11-Lab`.
- Started the Wazuh service successfully on the Windows endpoint.
- Confirmed the agent appeared in the Wazuh dashboard as active.
- Confirmed Windows events were visible in the Threat Hunting `Events` view.

## Commands Used

```bat
ipconfig
ping 192.168.152.128
```

```powershell
NET START Wazuh
```

## Issue Encountered

No blocking issue occurred during onboarding. Validation focused on confirming network reachability, agent registration, and event flow from the Windows endpoint to the Wazuh server.

## Troubleshooting Performed

- Confirmed the Windows endpoint could reach the Wazuh server over the VMware NAT network.
- Verified that the agent service started successfully on the Windows VM.
- Checked the Wazuh dashboard to confirm the endpoint appeared as active.
- Reviewed the Threat Hunting `Events` view to ensure logs were being ingested from agent `001`.

## Result

The Windows endpoint was successfully onboarded to Wazuh. The agent registered as `Windows-11-Lab`, showed as active in the dashboard, and began sending Windows event data that appeared in the Threat Hunting `Events` view.

## Current State

- Wazuh server IP: `192.168.152.128`
- Windows endpoint IP: `192.168.152.129`
- Agent ID: `001`
- Agent name: `Windows-11-Lab`
- Agent status: active
- Group: `default`
- Registration date: `Apr 22, 2026 @ 16:34:09`
- Initial Windows event ingestion: confirmed
- Threat Hunting results observed: 494 hits during initial validation

## Evidence to Capture

- Screenshot of Windows VM specifications
- Screenshot of Windows-to-Wazuh network connectivity
- Screenshot of Wazuh `Deploy new agent` workflow for Windows
- Screenshot of successful Windows agent installation and service startup
- Screenshot showing the Windows endpoint active in the Wazuh dashboard
- Screenshot showing the first Windows events in Threat Hunting

## Next Steps

- Generate security-relevant Windows events
- Build first detections
- Create and test alerts against the new endpoint telemetry
- Document incident-style findings in the repository
- Continue expanding detection and reporting coverage

## Lessons Learned

Successful endpoint onboarding required validating the full chain, not just installation: network reachability, agent service startup, dashboard registration, and actual event ingestion all had to be confirmed before the phase could be considered complete.
