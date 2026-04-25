# Detection: Windows Successful Login After Failed Attempts

## Objective

Identify a successful Windows logon sequence that occurred shortly after multiple failed login attempts on the same endpoint.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## Detection Summary

This detection identifies a correlation pattern: repeated failed Windows logons followed by successful local logon activity from the same endpoint. The detection is intended to support investigation of possible credential guessing, password spraying, or successful unauthorized access.

## Relevant Events

- Windows Event ID `4625`: Failed logon
- Windows Event ID `4624`: Successful logon
- Wazuh Rule ID `60122`: `Logon Failure - Unknown user or bad password`
- Wazuh Rule ID `67022`: `Non network or service local logon.`
- Wazuh Rule ID `67028`: `Special privileges assigned to new logon.`

## Detection Logic

Trigger an investigation when multiple failed logon events are followed by successful local logon activity on the same endpoint within a short time window.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND (rule.id: 60122 OR rule.id: 67022 OR rule.id: 67028)
```

Correlation fields:

- Endpoint name
- Agent ID
- Username
- Source address or workstation name
- Event sequence
- Time window
- Wazuh rule IDs
- Windows Event IDs

## Test Method

The Windows endpoint was locked, and several incorrect PIN or password attempts were entered. After the failed attempts, the correct credentials were used to log back into the endpoint.

## Validation

Windows Event Viewer:

- Event ID `4625` showed failed logon attempts.
- Event ID `4624` showed successful logon activity.
- Successful logon events were visible around `1:15 PM` on April 25, 2026.

Wazuh Dashboard:

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

Wazuh provided the event data needed to investigate a sequence of failed logins followed by successful local logon activity. The lab confirmed that authentication failures and later successful logon-related events can be reviewed together from the same Windows endpoint.

## Detection Considerations

- A successful login after failures can be normal user error or suspicious authentication behavior.
- This detection should use a threshold and time window instead of alerting on isolated failures.
- Confidence improves when the same username and source are present in both failed and successful events.
- Special privilege assignment after logon should increase investigation priority.

## Recommendation

- Correlate failed and successful logons by username, endpoint, and source address.
- Alert when a successful login follows multiple failed login attempts within a short time window.
- Review logon type and source IP.
- Investigate whether special privileges were assigned after the successful logon.
- Apply account lockout policies where appropriate.
- Require MFA where possible.
- Tune thresholds to avoid alerting on normal user mistakes while still detecting suspicious authentication patterns.
