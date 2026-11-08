# T1547.001 Evidence Index

The screenshots are ordered to show preparation, controlled execution, detection, alerting, cleanup and recovery.

| # | Screenshot | Evidence |
|---:|---|---|
| 01 | `01-endpoint-identity.png` | Windows endpoint identity and address |
| 02 | `02-monitoring-services-running.png` | Sysmon, Wazuh Agent and Splunk Forwarder health |
| 03 | `03-registry-baseline.png` | Existing current-user Run values |
| 04 | `04-sysmon-registry-baseline.png` | Sysmon registry telemetry validation |
| 05 | `05-controlled-run-key-creation.png` | Harmless Run-value creation |
| 06 | `06-created-run-key-details.png` | Registry path, name, type and data |
| 07 | `07-local-sysmon-event-13.png` | Local Sysmon registry value-set event |
| 08 | `08-splunk-registry-events.png` | Splunk registry-event investigation |
| 09 | `09-wazuh-registry-alert.png` | Wazuh registry evidence |
| 10 | `10-splunk-dashboard.png` | T1547.001 dashboard panel |
| 11 | `11-splunk-alert-config.png` | High-severity alert configuration |
| 12 | `12-splunk-alert-triggered.png` | Triggered Splunk alert |
| 13 | `13-email-alert.png` | SMTP email notification |
| 14 | `14-run-key-cleanup.png` | Value removal and Sysmon deletion event |
| 15 | `15-recovery-validated.png` | Value absence, service health and SIEM connectivity |

All screenshots must be sanitized before publication. Passwords, application passwords, tokens, cookies and personal email addresses must not be visible.

