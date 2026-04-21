# Lab Notes

## Date

2026-04-21

## Phase

Phase 1 - Wazuh SIEM Deployment

## Objective

Deploy the Wazuh SIEM, verify core service health, confirm required listening ports, and restore dashboard access after a routing issue.

## Actions Taken

- Updated Ubuntu packages and installed required dependencies.
- Downloaded the Wazuh installer and ran it with the `-i` flag to bypass the Ubuntu 24.04 OS check.
- Verified the `wazuh-manager`, `wazuh-indexer`, and `wazuh-dashboard` services.
- Confirmed required open ports for ingestion, registration, API access, and dashboard access.
- Tested dashboard connectivity at `https://192.168.152.128`.

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

The Wazuh server is operational, the dashboard is reachable, and the environment is ready for agent onboarding in the next phase.

## Current State

- Wazuh manager: running
- Wazuh indexer: running
- Wazuh dashboard: running
- Dashboard URL: `https://192.168.152.128`
- Total agents: 0
- Active agents: 0
- Disconnected agents: 0

## Evidence to Capture

- Screenshot of service status checks
- Screenshot of listening ports
- Screenshot of Wazuh dashboard login or home page
- Screenshot showing healthy service state after the routing fix

## Next Steps

- Deploy a Windows endpoint VM
- Install and register the Wazuh agent
- Validate Windows Event Log ingestion
- Create initial detections and test alerts
- Begin writing incident report templates for future exercises

## Lessons Learned

Ubuntu 24.04 required bypassing the installer's operating system validation, and successful deployment required validating both service health and network reachability rather than checking only one layer.
