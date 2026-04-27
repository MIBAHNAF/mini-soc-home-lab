# Incident Report 002: Local Account Creation, PowerShell Enumeration, and Administrator Group Change

## 1. Executive Summary

On April 25 and April 26, 2026, I generated a controlled activity chain on `Windows-11-Lab`: local account creation, PowerShell enumeration, and local Administrators group modification.

This was expected lab activity. The goal was to validate whether Wazuh could show the full story across account-management, PowerShell, and privilege-change telemetry.

Final call: lab validation successful. No real compromise.

---

## 2. Environment

| Field | Value |
|---|---|
| SIEM | Wazuh |
| Manager | mirahnafali-VMware-Virtual-Platform |
| Endpoint | Windows-11-Lab |
| Agent ID | 001 |
| Endpoint IP | 192.168.152.129 |
| Operating System | Microsoft Windows 11 |
| Agent Version | Wazuh v4.14.3 |
| Agent Group | default |

---

## 3. Alert and Detection Sources

Wazuh received Windows account-management and PowerShell logs from the Wazuh agent installed on `Windows-11-Lab`.

Key detection details:

| Source | Event / Rule | Description |
|---|---|---|
| Windows Security Log | Event ID `4720` | A user account was created |
| Windows Security Log | Event ID `4732` | A member was added to a security-enabled local group |
| PowerShell Operational Log | Event ID `4104` | PowerShell script block logging |
| Wazuh | Rule ID `60109` | User account enabled or created |
| Wazuh | Rule ID `60110` | User account changed |
| Wazuh | Rule ID `91816` | Powershell script querying system environment variables |
| Wazuh | Rule ID `60154` | Administrators Group Changed |
| Wazuh | Rule Level `12` | Higher severity alert for administrator group change |

---

## 4. Timeline

| Time | Event |
|---|---|
| Around 17:05:57 on Apr 25 | Wazuh displayed account creation and account change events for `soc_test_user` |
| 19:14:49 on Apr 25 | Windows Event Viewer showed Event ID `4732` for the Administrators group change |
| 19:14:45 on Apr 25 | Wazuh displayed `Administrators Group Changed` with rule ID `60154` and level `12` |
| Around 19:39 on Apr 26 | Wazuh displayed PowerShell activity with rule ID `91816` and level `4` |

Timestamp note:

Windows Event Viewer and Wazuh displayed timestamps differently during testing. I did not rely on the clock display alone. I correlated the events by sequence, endpoint name, agent ID, event description, Windows Event ID, and Wazuh rule ID.

---

## 5. What Happened

I created a local user account named `soc_test_user` on the Windows endpoint. After that, I added the account to the local Administrators group.

After the local account was created, benign PowerShell enumeration commands were executed to simulate discovery activity. The commands included process enumeration, local user enumeration, and local Administrators group enumeration. This activity was expected in the lab, but in a production environment, similar PowerShell discovery after account creation could require investigation because it may indicate that an actor is checking users, groups, and privilege levels before making further changes.

Wazuh ingested the PowerShell activity and displayed events with rule ID `91816`, described as `Powershell script querying system environment variables`. This confirmed that PowerShell telemetry was being collected after the PowerShell Operational log channel was added to the Windows Wazuh agent configuration.

The most important alert remained `Administrators Group Changed`. Wazuh mapped that to rule ID `60154` and marked it as rule level `12`.

I treated this as one incident because the full sequence tells a stronger story than any single event by itself:

```text
local user created -> PowerShell enumeration -> local Administrators group change
```

---

## 6. Evidence

### Screenshots

- `../screenshots/18a-windows-user-created.png`
- `../screenshots/18b-windows-user-created.png`
- `../screenshots/19-wazuh-user-created-event.png`
- `../screenshots/20a-windows-admin-group-change.png`
- `../screenshots/20b-windows-admin-group-change.png`
- `../screenshots/21-wazuh-privilege-change-event.png`
- `../screenshots/22-windows-powershell-logging-enabled.png`
- `../screenshots/23a-powershell-process-query.png`
- `../screenshots/23b-powershell-local-users.png`
- `../screenshots/23c-powershell-admin-group-query.png`
- `../screenshots/24-windows-powershell-eventviewer-4104.png`
- `../screenshots/25-wazuh-powershell-event.png`

### Wazuh Report

- `evidence/incident-002-local-account-powershell-admin-change-report.pdf`

---

## 7. Analysis

The activity chain is security-relevant because it combines three behaviors that are stronger together than they are individually:

1. A local user account was created.
2. PowerShell was used to enumerate local users, groups, and running processes.
3. The local user was added to the Administrators group.

In a real SOC environment, this sequence could indicate persistence, discovery, and privilege escalation. An attacker who gains access to a system may create a local account to maintain access, use PowerShell to understand the system and account structure, and then add the account to a privileged group to gain elevated control.

In this lab, the activity was intentionally generated and authorized. The purpose was to validate that Wazuh could detect and display account creation, PowerShell activity, and administrator group modification from a Windows endpoint.

The detection worked as expected. Wazuh displayed account-related events, PowerShell-related events, and the administrator group change as a higher-severity alert.

---

## 8. What I Observed

- `soc_test_user` existed on the Windows endpoint.
- `soc_test_user` was added to the local Administrators group.
- Windows Event Viewer showed Event ID `4720` for user creation.
- Windows Event Viewer showed Event ID `4732` for group membership change.
- Windows Event Viewer showed PowerShell Event ID `4104`.
- Wazuh showed `User account enabled or created` with rule ID `60109`.
- Wazuh showed `User account changed` with rule ID `60110`.
- Wazuh showed PowerShell activity with rule ID `91816`, level `4`, and count `16` in the exported report.
- Wazuh showed `Administrators Group Changed` with rule ID `60154`.
- Wazuh marked the Administrators group change as level `12`.
- The activity was expected because I generated it for lab validation.

---

## 9. What I Did

1. Created the local test account `soc_test_user`.
2. Confirmed the account existed on the endpoint.
3. Ran benign PowerShell enumeration commands.
4. Added `soc_test_user` to the local Administrators group.
5. Confirmed the account appeared in the Administrators group.
6. Checked Windows Event Viewer for account-management and PowerShell events.
7. Checked Wazuh Threat Hunting for account creation, PowerShell activity, and group change events.
8. Captured screenshots and exported the Wazuh report.
9. Documented the detection and incident workflow.

---

## 10. Impact

This was a controlled lab test. No unauthorized access occurred.

In production, the same pattern would matter because it could mean:

- A new local account was created for persistence
- PowerShell was used for discovery
- A user was given unauthorized administrator access
- Privilege escalation happened on the endpoint
- The account could install tools or change system settings
- The account could support lateral movement if credentials are reused elsewhere

The key question would be: was this approved admin work, or is it real risk?

---

## 11. Recommendations

If this happened in production, I would:

1. Identify who created the account.
2. Confirm whether the account creation was approved.
3. Review the PowerShell commands and user context.
4. Identify who added the account to Administrators.
5. Confirm whether the privilege change was approved.
6. Check whether the account logged in after being created.
7. Correlate local account creation with PowerShell discovery activity and privileged group changes.
8. Remove unauthorized users from local administrator groups.
9. Disable or delete unauthorized local users.
10. Correlate this activity with endpoint, identity, and VPN logs.
11. Apply least privilege so local administrator access stays limited.

---

## 12. Final Determination

Closed - lab validation successful.

This was expected activity. I confirmed that Wazuh can ingest and display Windows local account creation, PowerShell enumeration, and local Administrators group membership changes from the onboarded Windows endpoint.
