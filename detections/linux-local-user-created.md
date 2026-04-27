# Detection: Linux Local User Created

## Objective

Detect local Linux user account creation using Ubuntu authentication and system logs ingested by Wazuh.

## Endpoint

- Hostname: mirahnafali-VMware-Virtual-Platform
- IP address: 192.168.152.128
- Platform: Ubuntu 24.04 LTS
- SIEM: Wazuh

## Relevant Events

| Source | Event / Rule | Description |
|---|---|---|
| Linux system logs | Account creation activity | Local Linux user was created |
| Wazuh | Rule ID `5901` | New group added to the system |
| Wazuh | Rule ID `5902` | New user added to the system |
| Wazuh | Rule ID `5402` | Successful sudo to ROOT executed |
| Wazuh | Rule ID `5501` | PAM: Login session opened |
| Wazuh | Rule ID `5502` | PAM: Login session closed |

## Test Method

A test Linux user named `soclinuxuser` was created on the Ubuntu VM using the `useradd` command.

## Commands Used

```bash
sudo useradd -m soclinuxuser
id soclinuxuser
getent passwd soclinuxuser
```

## Validation

The user creation was first validated locally on Ubuntu using `id` and `getent passwd`. Wazuh then displayed related Linux account activity from `mirahnafali-VMware-Virtual-Platform`, including `New user added to the system` and `New group added to the system`.

## Evidence

- `screenshots/28-linux-user-created.png`
- `screenshots/29-wazuh-linux-user-created-event.png`

## Result

Wazuh successfully ingested and displayed Linux local user creation activity from the Ubuntu system.

## Analyst Notes

Unexpected Linux user creation can indicate persistence or unauthorized administrative activity. In a real environment, I would review the account that performed the action, the target username, the timestamp, and the related sudo activity around the same time.

## Recommendation

- Monitor local Linux user creation events.
- Alert on unexpected accounts created outside approved workflows.
- Review the command history and user responsible for the change.
- Disable or remove unauthorized accounts.
- Correlate user creation with sudo activity and login attempts.
