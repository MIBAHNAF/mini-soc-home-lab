# Network and Data Flow

## Network Layout

Both VMs are connected through VMware NAT / VMnet8.

```text
Host Machine
    |
    | VMware NAT / VMnet8
    |
    +-- Ubuntu Wazuh Server
    |      IP: 192.168.152.128
    |      Roles: manager, indexer, dashboard, API
    |
    +-- Windows-11-Lab
           IP: 192.168.152.129
           Role: monitored endpoint
           Agent ID: 001
```

## Log Flow

This is the main path I care about:

```text
Windows activity
    -> Windows Event Log
    -> Wazuh Windows agent
    -> Wazuh manager
    -> Wazuh indexer
    -> Wazuh dashboard / Threat Hunting
    -> Detection report or incident report
```

For Linux-side tests on the Ubuntu Wazuh server, the flow is slightly shorter:

```text
Linux activity
    -> Linux authentication / system logs
    -> Wazuh manager local log collection
    -> Wazuh indexer
    -> Wazuh dashboard / Threat Hunting
    -> Detection report
```

## Windows Step-by-Step Flow

1. Activity happens on `Windows-11-Lab`.
2. Windows writes the event to Event Viewer.
3. The Wazuh agent reads the event.
4. The agent sends the event to the Wazuh manager.
5. Wazuh indexes and enriches the event.
6. I review the event in Threat Hunting.
7. I compare the Wazuh event with the local Windows evidence.
8. I document the detection or incident.

## Linux Step-by-Step Flow

1. Activity happens on the Ubuntu Wazuh server.
2. Ubuntu writes the activity to local logs like `/var/log/auth.log` or `/var/log/syslog`.
3. Wazuh collects the local Linux log activity.
4. Wazuh indexes and enriches the event.
5. I review the event in Threat Hunting.
6. I compare the Wazuh event with local Ubuntu evidence.
7. I document the detection or incident.

## Validation Points

I do not count a detection as validated just because one screen shows an alert. I check the full chain.

Validation checklist:

- Endpoint generated the activity.
- The local system showed the expected event or command evidence.
- Wazuh showed the same or related activity from the correct host.
- Agent ID or hostname matched the expected system.
- Event or rule description matched the test.
- Windows Event ID or Linux log evidence matched the expected behavior.
- Wazuh rule ID matched the expected behavior.
- Screenshots or exported evidence were captured.

## Current Detection Data Sources

| Activity | Windows Event ID | Wazuh Rule ID |
|---|---|---|
| Failed logon | `4625` | `60122` |
| Successful logon | `4624` | `67022`, `67028` |
| Local user created | `4720` | `60109`, `60110`, `60170` |
| Local Administrators group changed | `4732` | `60154` |
| PowerShell activity | `4104` | `91816` |
| Linux failed login attempts | Linux auth logs | `5404`, `5503` |
| Linux local user created | Linux account-management logs | `5901`, `5902` |
| Linux UFW / firewall change | Linux auth logs and related sudo activity | Local UFW proof plus Wazuh sudo ingestion |

## Collection Notes

PowerShell required one extra collection step. Windows generated Event ID `4104` locally under the PowerShell Operational log, but Wazuh did not ingest it until the Windows agent was configured to collect:

```text
Microsoft-Windows-PowerShell/Operational
```

That was added to the Windows agent configuration at:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

This is an important validation lesson for the lab: local event generation and SIEM ingestion are two separate checkpoints.

## Timestamp Handling

During Phase 3, Windows Event Viewer and Wazuh did not always display timestamps in the same format or time zone.

I handled that by correlating events with:

- Event sequence
- Endpoint name: `Windows-11-Lab`
- Agent ID: `001`
- Event description
- Windows Event ID
- Wazuh rule ID

The timestamp still matters, but it is not the only proof. The event sequence and IDs are what keep the validation clean.

## Trust Boundaries

This is a lab, but the trust boundaries still matter:

- Host machine controls the VMware environment.
- VMware NAT / VMnet8 isolates the lab network from a normal production network.
- Windows endpoint is the monitored system.
- Wazuh server is the central collection and analysis point.
- Dashboard access should be treated as sensitive because it exposes SIEM data and management views.

## Current Limitations

- Only one Windows endpoint is onboarded.
- No domain controller is present.
- No production identity provider is connected.
- No EDR tool is integrated.
- Detections are validated manually through screenshots and Wazuh Threat Hunting.

That is fine for this stage. The point right now is to build the workflow and prove that the log pipeline works.

## Next Architecture Improvements

Possible future improvements:

- Add a second endpoint for lateral movement testing.
- Add a domain controller for Active Directory detections.
- Add Sysmon for better process and command-line telemetry.
- Add a visual network diagram.
- Add documented firewall rules and allowed traffic paths.
- Add a detection coverage matrix tied to MITRE ATT&CK.
