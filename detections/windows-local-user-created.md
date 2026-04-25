# Detection: Windows Local User Created

## Objective

Detect local user account creation on the Windows endpoint using Windows Security logs ingested by Wazuh.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11

## What This Detects

This detection looks for local account creation on a Windows endpoint. In this lab, I created a test account named `soc_test_user` from an Administrator PowerShell session.

This is a useful detection because unexpected local account creation can be a persistence technique. If an attacker gets access, one simple way to come back later is to create another local user.

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

Fields I would review:

- Created username
- Creator account
- Endpoint name
- Agent ID
- Group membership changes
- Related account-change events

## Test Method

I created a test local user account on the Windows endpoint using an Administrator PowerShell session.

## Commands Used

```powershell
net user soc_test_user <test-password> /add
net user soc_test_user
```

I intentionally left the test password out of the report. The second command confirmed that the account existed, was active, and belonged to the local `Users` group.

## Validation

Windows validation:

- The `net user soc_test_user` command completed successfully.
- The account name `soc_test_user` was visible.
- Account active status was `Yes`.
- Local group membership showed `Users`.
- Windows Event Viewer showed Event ID `4720`.

Wazuh validation:

- Wazuh displayed account-related events from `Windows-11-Lab`.
- Wazuh showed `User account enabled or created` with rule ID `60109`.
- Wazuh showed `User account changed` with rule ID `60110`.
- Wazuh showed `Users Group Changed` with rule ID `60170`.
- The relevant Wazuh events were observed around `17:05:57` on April 25, 2026.

## Evidence

- `screenshots/18a-windows-user-created.png`
- `screenshots/18b-windows-user-created.png`
- `screenshots/19-wazuh-user-created-event.png`

## Result

Wazuh ingested and displayed the local account creation activity from the Windows endpoint. The account existed locally, Windows logged it, and Wazuh showed the related account-management events.

## Detection Considerations

- Local user creation can be normal admin work.
- Unexpected account creation is worth reviewing.
- The detection is stronger when tied to the creator account, command-line activity, group changes, and later logons.
- Newly created accounts should be checked for privilege level and follow-up activity.

## Recommendation

- Monitor local user creation events.
- Alert on unauthorized or unexpected local account creation.
- Review the account that performed the action.
- Disable or remove unauthorized local users.
- Correlate user creation with privilege and group membership changes.
- Review successful logons by newly created local accounts.
- Keep local administrator rights limited.
