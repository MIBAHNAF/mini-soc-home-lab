# Detection: Linux Failed Login Attempts

## Objective

Detect failed Linux authentication attempts using local Linux authentication logs ingested by Wazuh.

## Endpoint

- Hostname: `mirahnafali-VMware-Virtual-Platform`
- IP address: `192.168.152.128`
- Platform: Ubuntu 24.04 LTS
- SIEM: Wazuh

## What This Detects

This detection looks for failed Linux authentication activity on the Ubuntu Wazuh server.

I tested this with failed `su` and `sudo` activity. The useful signal was not just one bad password. Wazuh also showed the stronger event: three failed attempts to run `sudo`.

## Relevant Events

- PAM authentication failure
- Failed `sudo` authentication
- Wazuh Rule ID `5404`: `Three failed attempts to run sudo`
- Wazuh Rule Level `10`
- Wazuh Rule ID `5503`: `PAM: User login failed`
- Wazuh Rule Level `5`

## Detection Logic

Trigger when Linux authentication logs show failed login or failed privilege escalation attempts.

Suggested Wazuh dashboard filters:

```text
manager.name: mirahnafali-VMware-Virtual-Platform AND (rule.id: 5404 OR rule.id: 5503)
```

Fields I would review:

- Hostname
- User account
- Authentication method
- Source terminal or source address
- Rule ID
- Rule level
- Whether the failures were followed by successful root access

## Test Method

I generated failed authentication activity on the Ubuntu VM by trying to switch to a nonexistent user and by entering incorrect passwords for a `sudo` command.

## Commands Used

```bash
su fakeuser
sudo -k
sudo ls /root
```

Local validation command:

```bash
sudo grep -i "authentication failure\|incorrect password\|FAILED su\|3 incorrect password attempts" /var/log/auth.log | tail -n 30
```

## Validation

Local Linux validation:

- Failed authentication activity appeared in `/var/log/auth.log`.
- The local logs showed failed `su` or `sudo` authentication activity.

Wazuh validation:

- Wazuh Threat Hunting showed Linux authentication events from `mirahnafali-VMware-Virtual-Platform`.
- Wazuh displayed `Three failed attempts to run sudo`.
- Wazuh displayed rule ID `5404`.
- Wazuh displayed rule level `10`.
- Wazuh displayed related `PAM: User login failed` activity.
- Wazuh displayed rule ID `5503`.
- Wazuh displayed rule level `5`.

## Evidence

- `screenshots/26-linux-failed-login-attempts.png`
- `screenshots/27-wazuh-linux-failed-login-events.png`

## Result

Wazuh ingested and displayed Linux failed authentication activity from the Ubuntu system.

This detection adds Linux authentication monitoring to the lab. It also shows that Wazuh can alert on failed privilege escalation attempts, not just Windows endpoint activity.

## Detection Considerations

- One failed login can be normal user error.
- Repeated failed `sudo` attempts are more important because they point to failed privilege escalation.
- This detection gets stronger when failures are tied to username, source, command, and any later successful privileged access.
- The Wazuh rule level `10` for repeated failed `sudo` attempts makes this worth reviewing quickly.

## Recommendation

- Monitor repeated `sudo` and PAM authentication failures.
- Alert when multiple failed `sudo` attempts occur within a short time window.
- Review whether failures are followed by successful privileged access.
- Enforce least privilege and strong password policies.
- Restrict SSH access and administrative access where possible.
