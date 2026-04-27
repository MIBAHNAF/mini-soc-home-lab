# Incident Report 003: Linux Failed Authentication, Local User Creation, and Firewall Configuration Change

## 1. Executive Summary

A sequence of Linux security-relevant events was generated on the Ubuntu Wazuh server to validate Linux log monitoring through Wazuh. The activity included failed sudo/authentication attempts, creation of a local Linux user, and a UFW firewall rule configuration change.

This was controlled lab activity, not a real compromise. However, in a production environment, this sequence would require review because it combines failed privilege access, local account creation, and firewall configuration activity.

Wazuh successfully ingested and displayed Linux authentication, sudo, user creation, and related system activity from the Ubuntu host.

Final call: lab validation successful.

---

## 2. Environment

| Field | Value |
|---|---|
| SIEM | Wazuh |
| Hostname | mirahnafali-VMware-Virtual-Platform |
| IP Address | 192.168.152.128 |
| Platform | Ubuntu 24.04 LTS |
| Log Sources | `/var/log/auth.log`, `/var/log/syslog`, Wazuh Threat Hunting |

---

## 3. Detection Sources

| Source | Rule/Event | Description |
|---|---|---|
| Wazuh | Rule ID `5404` | Three failed attempts to run sudo |
| Wazuh | Rule ID `5503` | PAM: User login failed |
| Wazuh | Rule ID `5901` | New group added to the system |
| Wazuh | Rule ID `5902` | New user added to the system |
| Wazuh | Rule ID `5402` | Successful sudo to ROOT executed |
| Ubuntu logs | `/var/log/auth.log` | Confirmed sudo and UFW command activity |
| Ubuntu logs | `/var/log/syslog` | Confirmed related UFW service activity |

---

## 4. Timeline

| Event | Description |
|---|---|
| Failed authentication test | Invalid `su` and incorrect `sudo` password attempts were generated |
| Local user creation | Test user `soclinuxuser` was created |
| Firewall configuration change | UFW rule for TCP port `2222` was added |
| Wazuh validation | Events appeared in Wazuh Threat Hunting under the Ubuntu host |

Timestamp note:

Some timestamps may differ slightly between Ubuntu terminal output and Wazuh because of display or time zone formatting. I correlated events using hostname, rule descriptions, command output, and local log entries.

---

## 5. What Happened

First, failed Linux authentication activity was generated using an invalid `su` attempt and incorrect `sudo` password attempts. Ubuntu recorded the activity in `/var/log/auth.log`, including PAM authentication failure messages and `3 incorrect password attempts`.

Next, a local Linux user named `soclinuxuser` was created using `useradd`. The account was confirmed locally with `id soclinuxuser` and `getent passwd soclinuxuser`. Wazuh displayed related account creation events, including `New user added to the system` and `New group added to the system`.

Finally, a temporary UFW rule was added for TCP port `2222`. UFW was inactive during the test, so this represented a firewall rule configuration change rather than active packet-filtering enforcement. The exact UFW command was confirmed locally in `/var/log/auth.log`, while Wazuh confirmed related sudo command ingestion from the Ubuntu host.

I treated these as one incident because the sequence tells a stronger Linux SOC story than the individual detections alone:

```text
failed sudo/authentication attempts -> local Linux user created -> firewall/UFW rule modified
```

---

## 6. Evidence

### Screenshots

- `../screenshots/26-linux-failed-login-attempts.png`
- `../screenshots/27-wazuh-linux-failed-login-events.png`
- `../screenshots/28-linux-user-created.png`
- `../screenshots/29-wazuh-linux-user-created-event.png`
- `../screenshots/30-linux-firewall-change.png`
- `../screenshots/31-linux-ufw-log-proof.png`
- `../screenshots/32-wazuh-linux-ufw-event.png`

### Wazuh Report

- `evidence/incident-003-linux-auth-user-firewall-report.pdf`

---

## 7. Analysis

This activity chain is stronger than the individual detections by themselves. Failed sudo attempts can indicate attempted privilege access. Local user creation can indicate persistence. Firewall rule modification can indicate an attempt to change host-level network exposure.

In this lab, all activity was intentionally generated. In production, the sequence would be suspicious because it could represent a user or attacker attempting to gain privileged access, create a new local account, and modify firewall behavior.

The Wazuh report supports this investigation by showing Linux authentication and sudo activity, including `Three failed attempts to run sudo`, `PAM: User login failed`, and `Successful sudo to ROOT executed`. It also confirms account creation activity through `New user added to the system` and `New group added to the system`.

The UFW test had one limitation: UFW was inactive. Because of that, this should be described as a firewall rule configuration change, not an active firewall enforcement change.

---

## 8. Impact

This was a controlled lab test with no real unauthorized access.

In production, similar activity could create risk by allowing:

- Attempted privilege escalation
- Unauthorized local account persistence
- Unauthorized firewall rule changes
- Increased exposure of local services
- Preparation for later access or lateral movement

---

## 9. Recommendations

1. Monitor repeated sudo and PAM authentication failures.
2. Alert when multiple failed sudo attempts occur within a short time window.
3. Monitor local Linux user and group creation.
4. Review the account responsible for user creation.
5. Monitor firewall/UFW configuration changes.
6. Alert on unexpected ports being opened or modified.
7. Correlate sudo activity with account creation and firewall changes.
8. Require approval for privileged account and firewall modifications.
9. Remove unauthorized local users and firewall rules.

---

## 10. Status

Closed - lab validation successful.

---

## 11. Analyst Notes

This incident validates Linux-side monitoring in the Mini-SOC lab. It shows that Wazuh can support investigation across authentication, account management, and firewall-related administrative activity.

The main lesson is that the strongest SOC value comes from correlation. A failed sudo event, a new user, and a firewall change may each be explainable alone. Together, they form a sequence that deserves investigation.
