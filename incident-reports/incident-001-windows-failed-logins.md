# Incident Report 001: Failed Logins Followed by Successful Local Logon

## 1. Executive Summary

On April 25, 2026, the analyst observed multiple failed login attempts on the Windows endpoint `Windows-11-Lab`, followed shortly afterward by successful local logon-related activity from the same endpoint. This activity was generated intentionally in a controlled lab environment to validate Windows authentication monitoring through Wazuh SIEM.

The case was confirmed using Windows Event Viewer and Wazuh Threat Hunting. Wazuh showed four failed authentication events using rule ID `60122`, followed by local logon-related events using rule IDs `67022` and `67028`.

This incident was classified as a lab validation event, not a real compromise.

---

## 2. Environment

| Field | Value |
|---|---|
| SIEM | Wazuh |
| Manager | mirahnafali-VMware-Virtual-Platform |
| Endpoint | Windows-11-Lab |
| Agent ID | 001 |
| Endpoint IP | 192.168.152.129 |
| Operating System | Microsoft Windows 11 Enterprise |
| Agent Version | Wazuh v4.14.3 |
| Group | default |

---

## 3. Alert and Detection Source

The activity was detected through Windows authentication logs collected by the Wazuh agent and displayed in Wazuh Threat Hunting.

Relevant detection details:

| Field | Value |
|---|---|
| Failed Logon Event ID | 4625 |
| Successful Logon Event ID | 4624 |
| Failed Logon Wazuh Rule ID | 60122 |
| Local Logon Wazuh Rule ID | 67022 |
| Special Privileges Wazuh Rule ID | 67028 |
| Failed Logon Rule Description | Logon Failure - Unknown user or bad password |
| Alert Count for Failed Logons | 4 |

---

## 4. Timeline

| Time | Event |
|---|---|
| 16:14:37 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| 16:14:40 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| 16:14:42 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| 16:14:44 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| Around 16:15:30 | Local logon-related events recorded in Wazuh with rule IDs `67022` and `67028` |
| After failed attempts | User authenticated normally after the controlled test |

Timestamp note:

Windows Event Viewer and Wazuh displayed timestamps differently during testing. The analyst correlated the activity using event sequence, endpoint name, agent ID, rule descriptions, Windows Event IDs, and Wazuh rule IDs.

---

## 5. What Happened

The Windows endpoint was locked, and several incorrect PIN or password attempts were entered to simulate repeated failed authentication attempts. Windows generated failed logon events under Event ID `4625`.

After the failed attempts, the correct credentials were used to log back into the endpoint. Windows Event Viewer showed successful logon activity under Event ID `4624`. Wazuh then showed local logon-related activity from the same endpoint, including `Non network or service local logon` and `Special privileges assigned to new logon`.

The analyst treated the sequence as one investigation because the successful local logon happened shortly after the failed authentication activity on the same endpoint.

---

## 6. Analyst Observations

- Four failed logon events were visible in Wazuh for `Windows-11-Lab`.
- The failed logon events used Wazuh rule ID `60122`.
- Windows Event Viewer confirmed matching failed logon activity with Event ID `4625`.
- Windows Event Viewer confirmed successful logon activity with Event ID `4624`.
- Wazuh showed local logon-related events shortly after the failed attempts.
- Wazuh displayed rule ID `67022` for local logon activity.
- Wazuh displayed rule ID `67028` for special privileges assigned to the new logon.
- The activity was expected because it was generated intentionally for lab validation.

---

## 7. Actions Taken

1. Generated controlled failed login attempts on the Windows endpoint.
2. Logged in successfully after the failed attempts.
3. Confirmed failed logon events locally in Windows Event Viewer.
4. Confirmed successful logon activity locally in Windows Event Viewer.
5. Confirmed Wazuh received the failed authentication events.
6. Confirmed Wazuh displayed local logon-related events after the failures.
7. Captured screenshots and exported Wazuh evidence.
8. Documented the detection and incident workflow.

---

## 8. Evidence Reviewed

### Screenshots

- `../screenshots/14a-windows-audit-policy-logon.png`
- `../screenshots/14b-windows-failed-login-attempts.png`
- `../screenshots/15-wazuh-failed-login-events.png`
- `../screenshots/16-windows-success-after-failures.png`
- `../screenshots/17-wazuh-success-after-failures.png`

### Wazuh Report

- `evidence/incident-001-wazuh-threat-hunting-report.pdf.pdf`

---

## 9. Impact

This was a controlled lab test with no real unauthorized access.

Potential production impact of similar activity could include:

- Account lockout
- Successful unauthorized access after credential guessing
- Brute-force attempts against local or domain accounts
- Password spraying activity
- Privileged session creation after a successful login

---

## 10. Recommended Next Steps

For a production environment, the following actions would be recommended:

1. Identify the username involved in the failed and successful logons.
2. Review the source address or workstation name.
3. Confirm whether the successful logon was expected.
4. Review logon type and privilege assignment.
5. Check whether additional failures occurred against the same account or endpoint.
6. Alert when successful login follows repeated failures within a short time window.
7. Enforce account lockout policies where appropriate.
8. Require MFA where possible.
9. Correlate endpoint, VPN, and identity provider logs.

---

## 11. Final Determination

Closed - Lab validation successful.

The analyst determined that the activity was expected and intentionally generated. The case confirmed that Wazuh can ingest and display failed Windows authentication events, successful local logon-related events, and special privilege assignment events from the onboarded Windows endpoint.
