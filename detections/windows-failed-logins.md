# Detection: Windows Failed Login Attempts

## Objective

Detect failed Windows login attempts from the Windows endpoint using Wazuh event ingestion.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## Detection Summary

This detection identifies failed Windows logon attempts collected from the Windows Security event log and ingested into Wazuh. In this test, repeated incorrect PIN or password attempts generated Windows Event ID `4625`, which Wazuh mapped to rule `60122`.

## Relevant Event Details

- Windows Event ID: `4625`
- Windows log source: Security
- Wazuh rule description: `Logon Failure - Unknown user or bad password`
- Wazuh rule ID: `60122`
- Wazuh rule level: `5`
- Observed result count: `4`
- Observed event time window: April 25, 2026 around 4:14 PM

## Detection Logic

Trigger when Windows Security log events show failed authentication activity from the monitored Windows endpoint.

Detection fields:

- `agent.name: Windows-11-Lab`
- `agent.id: 001`
- `rule.id: 60122`
- `rule.description: Logon Failure - Unknown user or bad password`

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND rule.id: 60122
```

## Test Method

The Windows endpoint was locked and multiple incorrect PIN or password attempts were entered. After several failed attempts, the correct credentials were used to log back into the system.

Audit policy was checked before validation to confirm that logon auditing was enabled.

```powershell
auditpol /get /category:"Logon/Logoff"
```

The observed audit policy showed `Logon` configured for `Success and Failure`, which allowed the failed login attempts to appear in the Windows Security log.

## Validation

The failed logon attempts were confirmed in two places.

1. Windows Event Viewer

Path:

```text
Windows Logs > Security
```

Result:

- Event ID `4625` was present.
- Four failed logon events were observed.
- Event times were approximately `1:14:40 PM` through `1:14:47 PM` on April 25, 2026.

2. Wazuh Dashboard

Path:

```text
Threat Hunting > Events
```

Filters:

- Agent: `Windows-11-Lab`
- Agent ID: `001`
- Event type: Authentication failure

Result:

- Four failed logon events were detected.
- Wazuh displayed rule ID `60122`.
- Wazuh displayed rule level `5`.
- Wazuh displayed the rule description `Logon Failure - Unknown user or bad password`.

## Evidence

- `screenshots/14a-windows-audit-policy-logon.png`
- `screenshots/14b-windows-failed-login-attempts.png`
- `screenshots/15-wazuh-failed-login-events.png`

## Result

Wazuh successfully ingested and displayed failed Windows login attempts from the Windows endpoint. The Windows Event Viewer evidence and Wazuh dashboard evidence matched the same test activity, confirming end-to-end detection from endpoint event generation to SIEM visibility.

## Analyst Notes

This activity simulates a basic credential guessing pattern. In a real SOC environment, repeated failed logons against a single user account could indicate mistyped credentials, brute-force activity, password spraying, or unauthorized access attempts.

Follow-up investigation should include:

- Source host or source IP associated with the failed attempts
- Target username
- Time pattern and frequency of failures
- Whether a successful login occurred shortly after the failures
- Whether the same account failed authentication across multiple systems

## Recommendation

- Monitor repeated failed login attempts by user and source host.
- Alert when failed attempts exceed a defined threshold.
- Review whether a successful login occurs shortly after repeated failures.
- Enforce account lockout policy where appropriate.
- Use MFA for accounts with remote access or administrative privileges.
- Tune thresholds to reduce noise from normal user mistakes while preserving brute-force visibility.
