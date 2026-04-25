# Detection: Windows Failed Login Attempts

## Objective

Detect failed Windows login attempts from the Windows endpoint using Windows Security logs ingested by Wazuh.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## Detection Summary

This detection identifies failed Windows logon attempts. In this lab, repeated incorrect PIN or password attempts generated Windows Event ID `4625`, and Wazuh displayed the activity as authentication failure events using rule ID `60122`.

## Relevant Events

- Windows Event ID `4625`: Failed logon
- Wazuh Rule ID `60122`: `Logon Failure - Unknown user or bad password`
- Wazuh rule level: `5`
- Observed alert count: `4`

## Detection Logic

Trigger when Windows Security logs show failed authentication activity from the monitored Windows endpoint.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND rule.id: 60122
```

Detection fields:

- `agent.name: Windows-11-Lab`
- `agent.id: 001`
- `rule.id: 60122`
- `rule.description: Logon Failure - Unknown user or bad password`

## Test Method

The Windows endpoint was locked, and multiple incorrect PIN or password attempts were entered. After several failed attempts, the correct credentials were used to log back into the system.

Audit policy was checked before validation to confirm that logon auditing was enabled.

```powershell
auditpol /get /category:"Logon/Logoff"
```

The observed audit policy showed `Logon` configured for `Success and Failure`, allowing failed login attempts to appear in the Windows Security log.

## Validation

Windows Event Viewer:

- Path: `Windows Logs > Security`
- Event ID `4625` was present.
- Four failed logon events were observed.
- Event times were approximately `1:14:40 PM` through `1:14:47 PM` on April 25, 2026.

Wazuh Dashboard:

- Path: `Threat Hunting > Events`
- Filtered by agent `Windows-11-Lab`
- Four failed logon events were detected.
- Wazuh displayed rule ID `60122`.
- Wazuh displayed rule level `5`.

## Evidence

- `screenshots/14a-windows-audit-policy-logon.png`
- `screenshots/14b-windows-failed-login-attempts.png`
- `screenshots/15-wazuh-failed-login-events.png`

## Result

Wazuh successfully ingested and displayed failed Windows login attempts from the Windows endpoint. The Windows Event Viewer evidence and Wazuh dashboard evidence matched the same test activity, confirming end-to-end visibility from endpoint event generation to SIEM display.

## Detection Considerations

- Failed logons can be benign when caused by normal user mistakes.
- Repeated failures in a short time window increase suspicion.
- Confidence improves when failures are correlated with username, source address, endpoint, and later successful logons.
- This detection can be noisy if alerting on every single failed attempt.

## Recommendation

- Monitor repeated failed login attempts by user and source host.
- Alert when failed attempts exceed a defined threshold.
- Correlate failed logons with successful logons that occur shortly afterward.
- Enforce account lockout policy where appropriate.
- Use MFA for accounts with remote access or administrative privileges.
- Tune thresholds to reduce noise from normal user mistakes while preserving brute-force visibility.
