# Detection: Windows Local Administrator Group Change

## Objective

Detect when a local user account is added to the local Administrators group on a Windows endpoint.

This detection checks whether Wazuh can ingest and display privileged group membership changes from the Windows Security log.

## Endpoint

- Agent name: `Windows-11-Lab`
- Agent ID: `001`
- IP address: `192.168.152.129`
- Platform: Windows 11
- SIEM: Wazuh

## What This Detects

This detection looks for a local privilege change. In this test, I added `soc_test_user` to the local Administrators group.

That matters because adding a user to Administrators gives that account full control over the endpoint. In a real environment, this could be normal admin work, or it could be privilege escalation.

## Relevant Events

| Source | Event / Rule | Description |
|---|---|---|
| Windows Security Log | Event ID `4732` | A member was added to a security-enabled local group |
| Wazuh | Rule ID `60154` | Administrators Group Changed |
| Wazuh | Rule Level `12` | Higher severity alert |

## Detection Logic

Trigger when Windows account-management events show a member added to the local Administrators group.

Suggested Wazuh dashboard filter:

```text
agent.id: 001 AND rule.id: 60154
```

Fields I would review:

- Account added to Administrators
- Account that made the change
- Endpoint name
- Agent ID
- Windows Event ID
- Wazuh rule ID
- Group name
- Time of change

## Test Method

I added the test account `soc_test_user` to the local Administrators group from an Administrator PowerShell session.

## Commands Used

```powershell
net localgroup Administrators soc_test_user /add
net localgroup Administrators
```

The second command confirmed that `soc_test_user` was listed as a member of the local Administrators group.

## Validation

Windows validation:

- The command completed successfully.
- `soc_test_user` appeared in the local Administrators group.
- Windows Event Viewer showed Event ID `4732`.
- Event ID `4732` means a member was added to a security-enabled local group.

Wazuh validation:

- Wazuh Threat Hunting showed the event from `Windows-11-Lab`.
- Wazuh displayed `Administrators Group Changed`.
- Wazuh displayed rule ID `60154`.
- Wazuh displayed rule level `12`.
- The Wazuh event was observed around `19:14:45` on April 25, 2026.

## Evidence

- `screenshots/20a-windows-admin-group-change.png`
- `screenshots/20b-windows-admin-group-change.png`
- `screenshots/21-wazuh-privilege-change-event.png`

## Result

Wazuh detected and displayed the local Administrators group change from the Windows endpoint. The Windows endpoint showed the group membership change locally, and Wazuh showed the higher-severity alert centrally.

## Detection Considerations

- Administrator group changes are high-value events.
- This can be approved admin work, so context matters.
- This becomes more suspicious when it happens after a new local user is created.
- The strongest version of this detection correlates user creation, group change, logon activity, and PowerShell activity.
- Rule level `12` makes this worth reviewing quickly.

## Recommendation

- Monitor all local Administrators group membership changes.
- Alert on new users added to privileged local groups.
- Review the account that performed the change.
- Confirm whether the change was approved.
- Correlate this activity with recent account creation, failed logins, successful logins, and PowerShell activity.
- Remove unauthorized users from privileged groups.
- Keep local administrator access limited.
