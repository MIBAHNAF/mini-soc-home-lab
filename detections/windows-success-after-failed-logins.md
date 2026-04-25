# Detection: Windows Successful Login After Failed Attempts

## Objective

Identify a successful Windows logon sequence that occurred shortly after multiple failed login attempts on the same endpoint.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## Detection Summary

This detection looks for a suspicious authentication pattern where repeated Windows failed logon events are followed shortly afterward by successful local logon activity from the same endpoint. In this lab, the activity was generated intentionally to validate that Wazuh provides enough event data to investigate the sequence.

## Relevant Events

- Windows Event ID `4625`: Failed logon
- Windows Event ID `4624`: Successful logon

## Related Wazuh Rules Observed

- Rule ID `60122`: `Logon Failure - Unknown user or bad password`
- Rule ID `67022`: `Non network or service local logon.`
- Rule ID `67028`: `Special privileges assigned to new logon.`

## Detection Logic

Trigger an investigation when multiple failed logon events are followed by successful local logon activity on the same endpoint within a short time window.

Observed sequence:

- Four failed logon events from `Windows-11-Lab`
- Followed by local logon-related events from `Windows-11-Lab`
- Same Wazuh agent ID: `001`
- Same monitored endpoint: `192.168.152.129`

Suggested Wazuh dashboard filters:

```text
agent.id: 001 AND (rule.id: 60122 OR rule.id: 67022 OR rule.id: 67028)
```

For a production detection, the logic should correlate by username, endpoint, source address, and time window.

## Test Method

The Windows endpoint was locked, and several incorrect PIN or password attempts were entered. After the failed attempts, the correct credentials were used to log back into the endpoint.

## Validation

The failed logon attempts were first confirmed in Windows Event Viewer using Event ID `4625`. The successful logon was confirmed in Windows Event Viewer using Event ID `4624`. Wazuh then showed the failed logon events followed by local logon-related events from the same endpoint.

Windows Event Viewer validation:

- Event ID `4625` showed failed logon attempts.
- Event ID `4624` showed successful logon activity.
- Successful logon events were visible around `1:15 PM` on April 25, 2026.

Wazuh validation:

- Four failed logon events were observed with rule ID `60122`.
- Failed logon events occurred between `16:14:37` and `16:14:44`.
- Local logon-related events followed around `16:15:30`.
- Wazuh showed rule IDs `67022` and `67028` after the failed attempts.

## Evidence

- `screenshots/14b-windows-failed-login-attempts.png`
- `screenshots/15-wazuh-failed-login-events.png`
- `screenshots/16-windows-success-after-failures.png`
- `screenshots/17-wazuh-success-after-failures.png`

## Result

Wazuh successfully provided the event data needed to investigate a sequence of failed logins followed by successful local logon activity. The lab confirmed that authentication failures and later successful logon-related events can be reviewed together from the same Windows endpoint.

## Analyst Notes

In a production environment, this pattern would require review because a successful logon after repeated failures could indicate normal user error, credential guessing, password spraying, or successful unauthorized access.

The next investigation step would be to check:

- Account name associated with the failed and successful logons
- Logon type
- Source address or workstation name
- Whether the behavior matches normal user activity
- Whether the same account had failures across other endpoints
- Whether privileged access was assigned after the successful logon

## Recommendation

- Correlate failed and successful logons by username, endpoint, and source address.
- Alert when a successful login follows multiple failed login attempts within a short time window.
- Review logon type and source IP.
- Investigate whether special privileges were assigned after the successful logon.
- Apply account lockout policies where appropriate.
- Require MFA where possible.
- Tune thresholds to avoid alerting on normal user mistakes while still detecting suspicious authentication patterns.
