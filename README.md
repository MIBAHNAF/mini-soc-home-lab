# Mini-SOC Home Lab

## Overview

This project simulates a small-scale Security Operations Center (SOC) environment using Wazuh SIEM.
The goal is to centralize log collection, validate detection capabilities, and document incident response workflows in a controlled virtual lab.

Phase 1 focuses on deploying the SIEM infrastructure.

---

## Lab Environment

**Hypervisor:** VMware
**SIEM Platform:** Wazuh 4.7.x
**Server OS:** Ubuntu 24.04 LTS
**Network Mode:** NAT (VMnet8)
**Server IP:** 192.168.152.128

### Wazuh Server VM Configuration

* 4 vCPUs
* ~5 GB RAM
* 20 GB disk

---

## Phase 1 – Wazuh SIEM Deployment

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
Ubuntu 24.04 was not officially listed in the installer’s supported OS check.
The `-i` flag was used to bypass OS validation in this lab environment.

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

* 1514 – Agent log ingestion
* 1515 – Agent registration
* 55000 – Wazuh API
* 443 – Wazuh Dashboard

Verified using:

```bash
sudo netstat -tulnp | grep -E "1514|1515|55000|443"
```

---

### 5. Dashboard Access

Dashboard accessed via:

```
https://192.168.152.128
```

Successfully authenticated using generated admin credentials.

Initial state:

* Total agents: 0
* Active agents: 0
* Disconnected agents: 0

---

## Current Status

Wazuh SIEM infrastructure successfully deployed and operational.

Next Phase:

* Deploy Windows endpoint VM
* Install Wazuh agent
* Validate Windows Event Log ingestion
* Begin detection engineering

---

## What This Demonstrates

* Enterprise SIEM deployment
* Linux service management
* Network port verification
* TLS dashboard configuration
* Virtualization best practices
* Troubleshooting OS compatibility issues





