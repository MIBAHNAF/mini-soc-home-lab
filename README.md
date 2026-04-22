# Mini-SOC Home Lab

## Overview

This project simulates a small-scale Security Operations Center (SOC) environment using Wazuh SIEM. The goal is to centralize log collection, validate detection capabilities, and document incident response workflows in a controlled virtual lab.

Phase 1 focused on deploying the SIEM infrastructure, validating service health, confirming open communication ports, and restoring dashboard access after a routing issue. Phase 2 extended the lab by onboarding a Windows 11 endpoint, validating agent connectivity, and confirming that Windows event data was successfully reaching the Wazuh dashboard.

---

## Lab Environment

- **Hypervisor:** VMware
- **SIEM Platform:** Wazuh 4.7.x
- **Server OS:** Ubuntu 24.04 LTS
- **Network Mode:** NAT (VMnet8)
- **Server IP:** `192.168.152.128`

### Wazuh Server VM Configuration

- 4 vCPUs
- 4.9 GB RAM
- 20 GB disk

---

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

---

## Phase 1 - Wazuh SIEM Deployment

### 1. System Preparation

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl unzip net-tools -y
```

---

### 2. Wazuh Installation

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a -i
```

Note:
Ubuntu 24.04 was not officially listed in the installer's supported OS check. The `-i` flag was used to bypass OS validation in this lab environment.

---

### 3. Service Verification

Verified that core services were active:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All services confirmed running.

---

### 4. Port Validation

Confirmed listening ports:

- 1514 - Agent log ingestion
- 1515 - Agent registration
- 55000 - Wazuh API
- 443 - Wazuh Dashboard

Verified using:

```bash
sudo netstat -tulnp | grep -E "1514|1515|55000|443"
```

---

### 5. Dashboard Access

Dashboard accessed via:

```text
https://192.168.152.128
```

Successfully authenticated using generated admin credentials.

Initial state:

- Total agents: 0
- Active agents: 0
- Disconnected agents: 0

---

## Current Status

Wazuh SIEM infrastructure is successfully deployed and operational, and the first Windows endpoint is onboarded and reporting.

Next Phase:

- Generate security-relevant Windows events
- Build first detections
- Document incident-style findings

---

## What This Demonstrates

- Enterprise SIEM deployment
- Linux service management
- Network port verification
- TLS dashboard configuration
- Virtualization best practices
- Troubleshooting OS compatibility issues
- Windows endpoint onboarding
- Agent registration and connectivity validation
- Initial Windows event ingestion into Wazuh

---

## Phase 2 - Windows Endpoint Onboarding

### Objective

The goal of Phase 2 was to create a Windows endpoint VM, connect it to the same VMware network as the Wazuh server, install the Wazuh Windows agent, verify that the endpoint appeared in the dashboard, and confirm that Windows event data was being ingested successfully.

### Environment

- **Endpoint OS:** Windows 11
- **VM Name:** Windows-11-Lab
- **Network:** NAT / VMnet8
- **Windows IP:** `192.168.152.129`
- **Wazuh Server IP:** `192.168.152.128`

### Work Completed

- Created a Windows 11 VM in VMware
- Configured the VM with 64 GB disk, 4 GB RAM, and 2 CPU cores
- Set the network adapter to NAT / VMnet8
- Verified Windows-to-Wazuh connectivity using `ping`
- Opened the Wazuh dashboard from the Windows VM
- Used the Wazuh `Deploy new agent` workflow for Windows
- Installed the Wazuh Windows agent
- Started the Wazuh service successfully
- Confirmed the endpoint appeared as active in the Wazuh dashboard
- Confirmed Windows events were arriving in the Threat Hunting `Events` view

### Commands Used

#### Windows network validation

```bat
ipconfig
ping 192.168.152.128
```

#### Agent installation and startup

The Windows agent installer command was generated from the Wazuh dashboard's `Deploy new agent` workflow using the lab server IP, default group, and agent name `Windows-11-Lab`.

```powershell
NET START Wazuh
```

### Validation Results

- Agent `001` registered as `Windows-11-Lab`
- Agent status showed as `active` in the Wazuh dashboard
- The endpoint appeared under the `default` group with OS `Microsoft Windows 11 Enterprise`
- The first Windows events were visible in Threat Hunting with the agent filter applied
- Initial event ingestion was confirmed on April 22, 2026
