# Detection: Windows Failed Login Attempts

## Objective

Detect failed Windows login attempts from the Windows endpoint and confirm that Wazuh ingests the events correctly.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## What This Detects

This detection looks for failed Windows logons. I tested it by entering the wrong PIN or password several times on the Windows VM.

The important signal is simple:

- Windows generated Event ID `4625`.
- Wazuh received the events from `Windows-11-Lab`.
- Wazuh mapped the activity to rule ID `60122`.

## Relevant Events

- Windows Event ID `4625`: Failed logon
- Wazuh Rule ID `60122`: `Logon Failure - Unknown user or bad password`
- Wazuh rule level: `5`
- Observed alert count: `4`

## Detection Logic

Start simple: look for failed authentication events from the Windows endpoint.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND rule.id: 60122
```

Fields I used to validate the detection:

- `agent.name: Windows-11-Lab`
- `agent.id: 001`
- `rule.id: 60122`
- `rule.description: Logon Failure - Unknown user or bad password`

## Test Method

I locked the Windows endpoint and entered the wrong PIN or password multiple times. After the failed attempts, I logged in with the correct credentials.

Before validating the detection, I checked the audit policy:

```powershell
auditpol /get /category:"Logon/Logoff"
```

The `Logon` setting showed `Success and Failure`, so Windows was configured to record failed logon activity.

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

Wazuh ingested and displayed the failed Windows login attempts. The same activity showed up locally in Windows Event Viewer and centrally in Wazuh, so the pipeline worked end to end.

## Detection Considerations

- One failed login can be normal user error.
- Multiple failures in a short window are more useful.
- This detection gets stronger when it is tied to username, source address, endpoint, and any later successful login.
- Alerting on every failed attempt would create noise.

## Recommendation

- Monitor repeated failed login attempts by user and source host.
- Alert when failed attempts cross a clear threshold.
- Correlate failed logons with successful logons that happen shortly afterward.
- Enforce account lockout policy where it makes sense.
- Use MFA for remote access and privileged accounts.
- Tune the threshold so normal mistakes do not drown out real credential attacks.
