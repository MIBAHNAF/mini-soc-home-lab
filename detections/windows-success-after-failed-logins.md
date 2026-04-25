# Detection: Windows Successful Login After Failed Attempts

## Objective

Identify a successful Windows logon that happened shortly after multiple failed login attempts on the same endpoint.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## What This Detects

This detection looks for a sequence, not just one event. The pattern is failed logins first, then successful local logon activity shortly after.

That matters because the activity could be normal user error, or it could be credential guessing that eventually worked. The detection does not prove compromise by itself. It tells the analyst where to look.

## Relevant Events

- Windows Event ID `4625`: Failed logon
- Windows Event ID `4624`: Successful logon
- Wazuh Rule ID `60122`: `Logon Failure - Unknown user or bad password`
- Wazuh Rule ID `67022`: `Non network or service local logon.`
- Wazuh Rule ID `67028`: `Special privileges assigned to new logon.`

## Detection Logic

Trigger an investigation when repeated failed logons are followed by successful local logon activity on the same endpoint within a short time window.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND (rule.id: 60122 OR rule.id: 67022 OR rule.id: 67028)
```

Fields I would correlate:

- Endpoint name
- Agent ID
- Username
- Source address or workstation name
- Event sequence
- Time window
- Wazuh rule IDs
- Windows Event IDs

## Test Method

I locked the Windows endpoint and entered the wrong PIN or password several times. Then I used the correct credentials to log back into the endpoint.

Same methodology as the failed-login test: generate the activity, confirm it locally, then confirm it in Wazuh.

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

Wazuh provided enough data to investigate the sequence. I could see the failed logons and the later local logon activity from the same endpoint.

## Detection Considerations

- This should not fire on one failed login.
- A short time window matters.
- The strongest version of this detection would match the same username and source.
- Special privilege assignment after the successful logon should raise priority.
- Timestamp display differences between Windows and Wazuh should not block validation. Use event sequence, endpoint, rule IDs, and Event IDs.

## Recommendation

- Correlate failed and successful logons by username, endpoint, and source address.
- Alert when a successful login follows multiple failed attempts in a short time window.
- Review logon type and source IP.
- Check whether special privileges were assigned after logon.
- Apply account lockout policies where appropriate.
- Require MFA where possible.
- Tune thresholds so the alert catches real risk without creating noise.
