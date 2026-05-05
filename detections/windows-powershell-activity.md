# Detection: Windows PowerShell Activity

## Objective

Detect PowerShell activity on the Windows endpoint using PowerShell logs ingested by Wazuh.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11
- SIEM: Wazuh

## What This Detects

This detection checks whether PowerShell activity from the Windows endpoint is visible in Wazuh.

PowerShell is normal for administration, but it is also heavily used by attackers for discovery, execution, persistence, and defense evasion. For this test, I used safe enumeration commands. The goal was visibility, not a full incident story.

## Relevant Events

- PowerShell Operational Log
- Windows PowerShell Event ID `4104`: Script block logging
- Wazuh Rule ID `91816`: `Powershell script querying system environment variables`
- Wazuh Rule Level: `4`

## Detection Logic

Trigger when PowerShell script block activity is collected from the endpoint and appears in Wazuh Threat Hunting.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND rule.id: 91816
```

Fields I would review:

- Endpoint name
- Agent ID
- PowerShell command or script block content
- Windows Event ID
- Wazuh rule ID
- Rule description
- User context
- Time of execution

## Test Method

I enabled PowerShell script block logging on the Windows endpoint. Then I ran benign PowerShell commands to generate safe test activity.

## Commands Used

```powershell
Get-Process | Where-Object {$_.CPU -gt 10}
Get-LocalUser
Get-LocalGroupMember Administrators
```

## Wazuh Agent Configuration Added

The Windows Wazuh agent was updated to collect the PowerShell Operational event channel.

The agent configuration file updated was:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

After updating the agent configuration, I restarted the Wazuh agent service.

```powershell
Restart-Service WazuhSvc
Get-Service WazuhSvc
```

## Collection Issue Encountered

During testing, PowerShell activity was visible locally in Windows Event Viewer but did not appear in Wazuh at first.

Script block logging was enabled successfully, and Windows generated Event ID `4104` under:

```text
Applications and Services Logs > Microsoft > Windows > PowerShell > Operational
```

This confirmed that the endpoint was producing PowerShell telemetry. The issue was not local Windows logging. The issue was that Wazuh was not ingesting those events yet because the Windows agent was not configured to collect the `Microsoft-Windows-PowerShell/Operational` event channel.

To fix this, I added the PowerShell Operational event channel to:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

After the configuration was added and the Wazuh agent service was restarted, PowerShell events appeared in Wazuh Threat Hunting.

## Validation

Windows validation:

- PowerShell script block logging was enabled.
- Windows Event Viewer showed PowerShell Operational events.
- Event ID `4104` was present.

Wazuh validation:

- Wazuh Threat Hunting showed PowerShell-related events from `Windows-11-Lab`.
- Wazuh displayed rule ID `91816`.
- Wazuh displayed rule level `4`.
- Wazuh displayed the rule description `Powershell script querying system environment variables`.

## Evidence

- `screenshots/22-windows-powershell-logging-enabled.png`
- `screenshots/23a-powershell-process-query.png`
- `screenshots/23b-powershell-local-users.png`
- `screenshots/23c-powershell-admin-group-query.png`
- `screenshots/24-windows-powershell-eventviewer-4104.png`
- `screenshots/25-wazuh-powershell-event.png`

## Result

Wazuh ingested and displayed PowerShell activity from the Windows endpoint after the PowerShell Operational event channel was added to the agent configuration.

This detection proves visibility. I am keeping it as a detection note only for now because the commands were benign enumeration commands. It does not need a separate incident report unless it becomes part of a stronger activity chain later.

## Detection Considerations

- PowerShell is used for both normal administration and attacker activity.
- Enumeration commands are not automatically malicious, but they matter in context.
- This detection gets stronger when PowerShell activity follows failed logins, successful logins, local user creation, or privilege changes.
- Script block logging gives better visibility than process names alone.
- The SIEM must collect the PowerShell Operational channel. Local logging by itself is not enough.

## Recommendation

- Enable PowerShell script block logging on Windows endpoints.
- Collect the PowerShell Operational log channel through the SIEM agent.
- Monitor PowerShell commands used for user, group, process, and system discovery.
- Review suspicious PowerShell activity alongside authentication and privilege-change events.
- Alert on encoded commands, download cradles, suspicious remote execution, and unusual administrative enumeration.
