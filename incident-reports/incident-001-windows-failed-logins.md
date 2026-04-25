# Incident Report 001: Windows Failed Login Attempts

## 1. Executive Summary

On April 25, 2026, multiple failed login attempts were detected on the Windows endpoint `Windows-11-Lab` in the Mini-SOC Home Lab. This activity was generated intentionally in a controlled lab environment to validate Windows authentication monitoring through Wazuh SIEM.

Wazuh successfully ingested the Windows failed logon events from the endpoint and generated authentication failure alerts. The activity was confirmed in both Windows Event Viewer and the Wazuh Threat Hunting interface.

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

## 3. Detection Source

The activity was detected through Windows authentication logs collected by the Wazuh agent and displayed in Wazuh Threat Hunting.

Relevant detection details:

| Field | Value |
|---|---|
| Windows Event ID | 4625 |
| Wazuh Rule ID | 60122 |
| Rule Description | Logon Failure - Unknown user or bad password |
| Rule Level | 5 |
| Alert Count | 4 |
| Rule Groups | authentication_failed, windows, windows_security |

---

## 4. Timeline

| Time | Event |
|---|---|
| 16:14:37 | Failed login attempt recorded in Wazuh |
| 16:14:40 | Failed login attempt recorded in Wazuh |
| 16:14:42 | Failed login attempt recorded in Wazuh |
| 16:14:44 | Failed login attempt recorded in Wazuh |
| After failed attempts | User authenticated normally after controlled test |

Windows Event Viewer also showed four matching Event ID `4625` entries around `1:14 PM` local Windows time.

---

## 5. What Happened

The Windows endpoint was locked, and several incorrect PIN or password attempts were entered to simulate repeated failed authentication attempts. This caused Windows to generate failed logon events under Event ID `4625`.

The Wazuh agent installed on `Windows-11-Lab` forwarded these Windows security events to the Wazuh manager. In the Wazuh dashboard, the events appeared under Threat Hunting with the rule description `Logon Failure - Unknown user or bad password`.

The Wazuh dashboard showed four authentication failure alerts for the Windows endpoint. The Wazuh-generated report also confirmed the endpoint information, including agent ID `001`, endpoint name `Windows-11-Lab`, IP address `192.168.152.129`, and Windows 11 operating system details. The report summary listed rule ID `60122`, rule level `5`, and alert count `4` for the failed logon activity.

---

## 6. Evidence

### Screenshots

- `../screenshots/14a-windows-audit-policy-logon.png`
- `../screenshots/14b-windows-failed-login-attempts.png`
- `../screenshots/15-wazuh-failed-login-events.png`

### Wazuh Report

- `evidence/incident-001-wazuh-threat-hunting-report.pdf.pdf`

---

## 7. Analysis

The failed login attempts were successfully detected and correlated to the Windows endpoint. In a real SOC environment, this pattern could represent several possibilities:

- A user mistyping their password or PIN
- A brute-force attempt against a local account
- Password spraying activity
- Unauthorized access attempts
- Reconnaissance against a valid user account

Because the activity occurred in a lab environment and was intentionally generated, there was no real security impact. However, the test confirms that the Wazuh pipeline can collect and display authentication failure events from a Windows endpoint.

The most important validation point is that the activity was visible in both the local Windows logs and the centralized SIEM dashboard. This confirms that endpoint telemetry is flowing correctly from Windows to Wazuh.

---

## 8. Impact

This was a controlled test with no real unauthorized access.

Potential production impact of similar activity could include:

- Account lockout
- Unauthorized access if followed by a successful login
- Credential guessing against local or domain accounts
- Early-stage brute-force or password spraying activity

---

## 9. Recommendations

For a production environment, the following actions would be recommended:

1. Monitor repeated failed login attempts by username, endpoint, and source address.
2. Alert when failed login attempts exceed a defined threshold within a short time window.
3. Investigate whether a successful login occurs shortly after repeated failures.
4. Enforce account lockout policies.
5. Require MFA where possible.
6. Review authentication attempts outside normal business hours.
7. Correlate failed logins with endpoint, VPN, and identity provider logs.
8. Tune SIEM rules to reduce false positives from normal user mistakes while still detecting suspicious patterns.

---

## 10. Status

Closed - Lab validation successful.

---

## 11. Analyst Notes

This incident confirmed that the Windows endpoint was properly onboarded into Wazuh and that authentication failure events were being ingested into the SIEM. The test also created a baseline workflow for future detection validation:

1. Generate controlled activity on the endpoint.
2. Confirm the event locally in Windows Event Viewer.
3. Confirm centralized ingestion in Wazuh.
4. Capture screenshots and exported reports.
5. Write a short analyst-style incident report.

This workflow will be reused for future detections involving successful login after failures, local user creation, privilege changes, and PowerShell activity.
