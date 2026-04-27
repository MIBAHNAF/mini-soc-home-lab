# Incident Report 001: Failed Logins Followed by Successful Local Logon

## 1. Executive Summary

On April 25, 2026, I observed four failed login attempts on `Windows-11-Lab`, followed shortly after by successful local logon-related activity from the same endpoint.

This was a controlled lab test. The goal was to validate that Wazuh could show both sides of the authentication story: the failed attempts and the later successful logon activity.

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
| Operating System | Microsoft Windows 11 Enterprise |
| Agent Version | Wazuh v4.14.3 |
| Group | default |

---

## 3. Alert and Detection Source

Wazuh received Windows authentication logs from the Wazuh agent installed on `Windows-11-Lab`.

Key detection details:

| Field | Value |
|---|---|
| Failed Logon Event ID | 4625 |
| Successful Logon Event ID | 4624 |
| Failed Logon Wazuh Rule ID | 60122 |
| Local Logon Wazuh Rule ID | 67022 |
| Special Privileges Wazuh Rule ID | 67028 |
| Failed Logon Rule Description | Logon Failure - Unknown user or bad password |
| Failed Logon Count | 4 |

---

## 4. Timeline

| Time | Event |
|---|---|
| 16:14:37 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| 16:14:40 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| 16:14:42 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| 16:14:44 | Failed login attempt recorded in Wazuh with rule ID `60122` |
| Around 16:15:30 | Local logon-related events recorded in Wazuh with rule IDs `67022` and `67028` |
| After failed attempts | User logged in normally after the controlled test |

Timestamp note:

Windows Event Viewer and Wazuh displayed timestamps differently during testing. I did not rely on the clock display alone. I correlated the events by sequence, endpoint name, agent ID, rule description, Windows Event ID, and Wazuh rule ID.

---

## 5. What Happened

I locked the Windows endpoint and entered the wrong PIN or password several times. Windows generated failed logon events under Event ID `4625`.

After that, I logged in with the correct credentials. Windows Event Viewer showed successful logon activity under Event ID `4624`. Wazuh showed related local logon events from the same endpoint, including:

- `Non network or service local logon`
- `Special privileges assigned to new logon`

I treated this as one incident because the successful logon happened right after the failed attempts on the same machine.

---

## 6. What I Observed

- Wazuh showed four failed logon events for `Windows-11-Lab`.
- The failed logon events used Wazuh rule ID `60122`.
- Windows Event Viewer confirmed failed logons with Event ID `4625`.
- Windows Event Viewer confirmed successful logon activity with Event ID `4624`.
- Wazuh showed local logon-related events shortly after the failures.
- Wazuh showed rule ID `67022` for local logon activity.
- Wazuh showed rule ID `67028` for special privileges assigned to the new logon.
- The activity was expected because I generated it for lab validation.

---

## 7. What I Did

1. Generated controlled failed login attempts on the Windows endpoint.
2. Logged in successfully after the failed attempts.
3. Checked Windows Event Viewer for Event ID `4625`.
4. Checked Windows Event Viewer for Event ID `4624`.
5. Checked Wazuh Threat Hunting for the failed authentication events.
6. Checked Wazuh Threat Hunting for the local logon-related events.
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

- `evidence/incident-001-wazuh-threat-hunting-report.pdf`

---

## 9. Impact

This was a controlled lab test. No unauthorized access occurred.

In production, the same pattern would matter because it could mean:

- A user made several mistakes and then logged in normally
- Someone guessed a password and eventually succeeded
- Password spraying or brute-force activity targeted the account
- A privileged session started after suspicious authentication activity

The key question would be: is it real or noise?

---

## 10. Recommended Next Steps

If this happened in production, I would:

1. Identify the username tied to the failed and successful logons.
2. Check the source address or workstation name.
3. Confirm whether the successful logon was expected.
4. Review the logon type and privilege assignment.
5. Check whether the same account had failures on other endpoints.
6. Alert when a successful login follows repeated failures in a short time window.
7. Enforce account lockout policy where appropriate.
8. Require MFA where possible.
9. Correlate endpoint, VPN, and identity provider logs.

---

## 11. Final Determination

Closed - lab validation successful.

This was expected activity. I confirmed that Wazuh can ingest and display failed Windows authentication events, successful local logon-related events, and special privilege assignment events from the onboarded Windows endpoint.
