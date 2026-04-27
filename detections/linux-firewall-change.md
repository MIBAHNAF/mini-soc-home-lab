# Detection: Linux UFW Firewall Change

## Objective

Detect and document a controlled Linux firewall rule change on the Ubuntu host, then validate what was visible locally and what was ingested by Wazuh.

## Endpoint

- Hostname: mirahnafali-VMware-Virtual-Platform
- IP address: 192.168.152.128
- Platform: Ubuntu 24.04 LTS
- SIEM: Wazuh

## Relevant Events

| Source | Event / Rule | Description |
|---|---|---|
| Ubuntu auth log | `sudo ufw allow 2222/tcp` | Local proof that the firewall rule change command was executed |
| Ubuntu UFW status | UFW numbered rules | Local proof that the test firewall rule existed |
| Wazuh | Sudo activity from Ubuntu host | Related sudo command activity ingested from `mirahnafali-VMware-Virtual-Platform` |

## Test Method

A controlled UFW rule was added on the Ubuntu VM to allow TCP port `2222`. The rule was used only as a safe test case to validate firewall-change visibility.

## Commands Used

```bash
sudo ufw status
sudo ufw allow 2222/tcp
sudo ufw status numbered
```

Local log validation:

```bash
sudo grep -i "ufw.*2222\|COMMAND=.*ufw" /var/log/auth.log | tail -n 30
```

Cleanup command:

```bash
sudo ufw delete allow 2222/tcp
sudo ufw status numbered
```

## Validation

The exact UFW rule change was confirmed locally in `/var/log/auth.log`, while Wazuh confirmed related sudo command ingestion from the Ubuntu host.

This distinction matters. The local Ubuntu evidence proves the specific firewall command and rule change. The Wazuh evidence proves that Wazuh was receiving related Linux sudo activity from the same host during the test.

## Evidence

- `screenshots/30-linux-firewall-change.png`
- `screenshots/31-linux-ufw-log-proof.png`
- `screenshots/32-wazuh-linux-ufw-event.png`

## Result

The Linux firewall change test was completed successfully. Ubuntu showed the exact UFW rule change locally, and Wazuh showed related sudo activity from the Ubuntu host.

## Analyst Notes

Firewall changes can be normal administrative work, but they can also expose new services or create unexpected access paths. In a real environment, I would check who made the change, whether the port was approved, what service was listening, and whether the rule was later removed.

## Recommendation

- Monitor firewall rule changes on Linux systems.
- Review UFW changes against approved change records.
- Alert on new allowed inbound ports that are not expected.
- Correlate firewall changes with sudo activity and login activity.
- Confirm temporary test rules are removed after validation.
