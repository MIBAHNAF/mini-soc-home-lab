# Lab Notes

## Current Status

As of 2026-04-22, Phase 1 and Phase 2 are complete. The Wazuh SIEM server is deployed and operational, the Windows 11 endpoint is onboarded, the agent is active, and initial Windows event ingestion has been confirmed.

Current lab state:

- Wazuh server IP: `192.168.152.128`
- Windows endpoint IP: `192.168.152.129`
- Agent ID: `001`
- Agent name: `Windows-11-Lab`
- Agent status: active
- Agent group: `default`
- Registration date: `Apr 22, 2026 @ 16:34:09`
- Initial Windows event ingestion: confirmed
- Threat Hunting results observed: 494 hits during initial validation

---

## Phase 1 - Wazuh SIEM Deployment

## Date

2026-04-21

## Objective

Deploy the Wazuh SIEM, verify core service health, confirm required listening ports, and restore dashboard access after a routing issue.

## Environment

- Hypervisor: VMware
- SIEM platform: Wazuh 4.7.x
- Server OS: Ubuntu 24.04 LTS
- Network mode: NAT / VMnet8
- Server IP: `192.168.152.128`
- Server VM resources: 4 vCPUs, 4.9 GB RAM, 20 GB disk

## Actions Taken

- Updated Ubuntu packages and installed required dependencies.
- Downloaded the Wazuh installer and ran it with the `-i` flag to bypass the Ubuntu 24.04 OS check.
- Verified the `wazuh-manager`, `wazuh-indexer`, and `wazuh-dashboard` services.
- Confirmed required open ports for ingestion, registration, API access, and dashboard access.
- Tested dashboard connectivity at `https://192.168.152.128`.
- Recovered dashboard access after a routing or network path issue.

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

## Issue Encountered

Dashboard access was interrupted by a routing or network path issue during initial validation.

## Troubleshooting Performed

- Confirmed the Wazuh services were running locally on the server.
- Verified the expected ports were listening on the host.
- Reviewed NAT networking and reachability between the host and the VM.
- Retested dashboard access after correcting the routing problem.

## Result

The Wazuh server was operational, the dashboard was reachable, and the environment was ready for agent onboarding.

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

## Lessons Learned

Ubuntu 24.04 required bypassing the installer's operating system validation, and successful deployment required validating both service health and network reachability rather than checking only one layer.

---

## Phase 2 - Windows Endpoint Onboarding

## Date

2026-04-22

## Objective

Create a Windows endpoint VM, place it on the same VMware NAT network as the Wazuh server, install and start the Wazuh Windows agent, verify the agent registered successfully, and confirm that Windows event data was being ingested into Wazuh.

## Environment

- Endpoint OS: Windows 11
- VM name: `Windows-11-Lab`
- Network: NAT / VMnet8
- Windows IP: `192.168.152.129`
- Wazuh server IP: `192.168.152.128`
- Endpoint VM resources: 64 GB disk, 4 GB RAM, 2 CPU cores

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

## Evidence Captured

- `screenshots/08-windows-vm-specifications.png`
- `screenshots/09-windows-network-connectivity.png`
- `screenshots/10-windows-agent-deploy-page.png`
- `screenshots/11-windows-agent-installed.png`
- `screenshots/12-agent-visible-in-dashboard.png`
- `screenshots/13-first-windows-events.png`

## Lessons Learned

Successful endpoint onboarding required validating the full chain, not just installation: network reachability, agent service startup, dashboard registration, and actual event ingestion all had to be confirmed before the phase could be considered complete.

---

## Next Steps

- Generate security-relevant Windows events.
- Build first detections.
- Create and test alerts against the new endpoint telemetry.
- Document incident-style findings in the repository.
- Continue expanding detection and reporting coverage.
