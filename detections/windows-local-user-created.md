# Detection: Windows Local User Created

## Objective

Detect local user account creation on the Windows endpoint using Windows Security logs ingested by Wazuh.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## Detection Summary

This detection identifies local account creation activity on a monitored Windows endpoint. In this lab, a test account named `soc_test_user` was created from an Administrator PowerShell session, then validated locally in Windows and centrally in Wazuh Threat Hunting.

## Relevant Events

- Windows Event ID `4720`: A user account was created
- Wazuh Rule ID `60109`: `User account enabled or created`
- Wazuh Rule ID `60110`: `User account changed`
- Wazuh Rule ID `60170`: `Users Group Changed`

## Detection Logic

Trigger when Windows account-management events show local user creation or related account changes on an endpoint.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND (rule.id: 60109 OR rule.id: 60110 OR rule.id: 60170)
```

Detection fields:

- Created username
- Creator account
- Endpoint name
- Agent ID
- Group membership changes
- Related account-change events

## Test Method

A test local user account was created on the Windows endpoint using an Administrator PowerShell session.

## Commands Used

```powershell
net user soc_test_user <test-password> /add
net user soc_test_user
```

The password value is intentionally omitted from the report. The second command confirmed that the account existed, was active, and belonged to the local `Users` group.

## Validation

Windows validation:

- The `net user soc_test_user` command completed successfully.
- The account name `soc_test_user` was visible.
- Account active status was `Yes`.
- Local group membership showed `Users`.
- Windows Event Viewer showed Event ID `4720`.

Wazuh validation:

- Wazuh displayed account-related events from the `Windows-11-Lab` endpoint.
- Wazuh showed `User account enabled or created` with rule ID `60109`.
- Wazuh showed `User account changed` with rule ID `60110`.
- Wazuh showed `Users Group Changed` with rule ID `60170`.
- The relevant Wazuh events were observed around `17:05:57` on April 25, 2026.

## Evidence

- `screenshots/18a-windows-user-created.png`
- `screenshots/18b-windows-user-created.png`
- `screenshots/19-wazuh-user-created-event.png`

## Result

Wazuh successfully ingested and displayed Windows local account creation activity from the endpoint. The Windows local validation and Wazuh event data confirmed that account-management telemetry is flowing from the Windows endpoint into the SIEM.

## Detection Considerations

- Local user creation can be legitimate administrative activity.
- Unexpected account creation can indicate persistence or unauthorized administrative access.
- Confidence improves when user creation is correlated with group membership changes, command-line activity, and later successful logons.
- Newly created accounts should be reviewed for privilege level and subsequent activity.

## Recommendation

- Monitor local user creation events.
- Alert on unauthorized or unexpected local account creation.
- Review the account that performed the action.
- Disable or remove unauthorized local users.
- Correlate user creation with privilege and group membership changes.
- Review successful logons by newly created local accounts.
- Use least privilege and restrict local administrator rights where possible.
