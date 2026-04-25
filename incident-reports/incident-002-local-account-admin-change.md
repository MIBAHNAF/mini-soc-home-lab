# Incident Report 002: Local Account Creation and Administrator Group Change

## 1. Executive Summary

On April 25, 2026, I created a local test user account named `soc_test_user` on `Windows-11-Lab`, then added that account to the local Administrators group.

This was a controlled lab test. The goal was to validate whether Wazuh could show both parts of the sequence: local account creation and privileged group membership change.

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
| Agent Group | default |

---

## 3. Alert and Detection Sources

Wazuh received Windows account-management logs from the Wazuh agent installed on `Windows-11-Lab`.

Key detection details:

| Source | Event / Rule | Description |
|---|---|---|
| Windows Security Log | Event ID `4720` | A user account was created |
| Windows Security Log | Event ID `4732` | A member was added to a security-enabled local group |
| Wazuh | Rule ID `60109` | User account enabled or created |
| Wazuh | Rule ID `60110` | User account changed |
| Wazuh | Rule ID `60154` | Administrators Group Changed |
| Wazuh | Rule Level `12` | Higher severity alert for administrator group change |

---

## 4. Timeline

| Time | Event |
|---|---|
| Around 17:05:57 | Wazuh displayed account creation and account change events for `soc_test_user` |
| 19:14:49 | Windows Event Viewer showed Event ID `4732` for the Administrators group change |
| 19:14:45 | Wazuh displayed `Administrators Group Changed` with rule ID `60154` and level `12` |

Timestamp note:

Windows Event Viewer and Wazuh displayed timestamps differently during testing. I did not rely on the clock display alone. I correlated the events by sequence, endpoint name, agent ID, event description, Windows Event ID, and Wazuh rule ID.

---

## 5. What Happened

I created a local user account named `soc_test_user` on the Windows endpoint. After that, I added the account to the local Administrators group.

Windows logged the account creation and the group membership change. Wazuh ingested the events and displayed account-related activity from `Windows-11-Lab`.

The most important alert was `Administrators Group Changed`. Wazuh mapped that to rule ID `60154` and marked it as rule level `12`.

I treated this as one incident because the account creation and privilege assignment tell one story. A new account by itself matters. A new account added to Administrators matters more.

---

## 6. What I Observed

- `soc_test_user` existed on the Windows endpoint.
- `soc_test_user` was added to the local Administrators group.
- Windows Event Viewer showed Event ID `4720` for user creation.
- Windows Event Viewer showed Event ID `4732` for group membership change.
- Wazuh showed `User account enabled or created` with rule ID `60109`.
- Wazuh showed `User account changed` with rule ID `60110`.
- Wazuh showed `Administrators Group Changed` with rule ID `60154`.
- Wazuh marked the Administrators group change as level `12`.
- The activity was expected because I generated it for lab validation.

---

## 7. What I Did

1. Created the local test account `soc_test_user`.
2. Confirmed the account existed on the endpoint.
3. Added `soc_test_user` to the local Administrators group.
4. Confirmed the account appeared in the Administrators group.
5. Checked Windows Event Viewer for account-management events.
6. Checked Wazuh Threat Hunting for account creation and group change events.
7. Captured screenshots and exported the Wazuh report.
8. Documented the detection and incident workflow.

---

## 8. Evidence Reviewed

### Screenshots

- `../screenshots/18a-windows-user-created.png`
- `../screenshots/18b-windows-user-created.png`
- `../screenshots/19-wazuh-user-created-event.png`
- `../screenshots/20a-windows-admin-group-change.png`
- `../screenshots/20b-windows-admin-group-change.png`
- `../screenshots/21-wazuh-privilege-change-event.png`

### Wazuh Report

- `evidence/incident-002-local-account-admin-change-report.pdf`

---

## 9. Impact

This was a controlled lab test. No unauthorized access occurred.

In production, the same pattern would matter because it could mean:

- A new local account was created for persistence
- A user was given unauthorized administrator access
- Privilege escalation happened on the endpoint
- The account could install tools or change system settings
- The account could support lateral movement if credentials are reused elsewhere

The key question would be: was this approved admin work, or is it real risk?

---

## 10. Recommended Next Steps

If this happened in production, I would:

1. Identify who created the account.
2. Confirm whether the account creation was approved.
3. Identify who added the account to Administrators.
4. Confirm whether the privilege change was approved.
5. Check whether the account logged in after being created.
6. Review PowerShell and command-line activity around the same time.
7. Remove unauthorized users from local administrator groups.
8. Disable or delete unauthorized local users.
9. Correlate this activity with endpoint, identity, and VPN logs.
10. Apply least privilege so local administrator access stays limited.

---

## 11. Final Determination

Closed - lab validation successful.

This was expected activity. I confirmed that Wazuh can ingest and display Windows local account creation, account changes, and local Administrators group membership changes from the onboarded Windows endpoint.
