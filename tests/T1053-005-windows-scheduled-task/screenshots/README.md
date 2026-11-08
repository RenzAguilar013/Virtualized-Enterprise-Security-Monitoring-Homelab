# T1053.005 Evidence Index

The screenshots are ordered to show preparation, execution, detection, alerting, cleanup and recovery.

| # | Screenshot | Evidence |
|---:|---|---|
| 01 | `01-endpoint-identity.png` | Windows endpoint identity and address |
| 02 | `02-monitoring-services-running.png` | Sysmon, Wazuh Agent and Splunk Forwarder health |
| 03 | `03-scheduled-task-auditing-enabled.png` | Other Object Access Events auditing |
| 04 | `04-task-scheduler-log-enabled.png` | Task Scheduler Operational channel enabled |
| 05 | `05-required-event-logs.png` | Security, Task Scheduler and Sysmon channels |
| 06 | `06-splunk-event-inputs.png` | Splunk Windows event-log inputs |
| 07 | `07-scheduled-tasks-before-test.png` | Scheduled-task baseline |
| 08 | `08-controlled-task-creation.png` | Harmless task creation |
| 09 | `09-created-task-details.png` | Created task properties |
| 10 | `10-local-security-event-4698.png` | Windows Security task-creation event |
| 11 | `11-local-task-scheduler-event-106.png` | Task Scheduler registration event |
| 12 | `12-local-sysmon-schtasks-event.png` | Sysmon `schtasks.exe` process creation |
| 13 | `13-splunk-scheduled-task-events.png` | Splunk event correlation |
| 14 | `14-wazuh-scheduled-task-alert.png` | Wazuh scheduled-task evidence |
| 15 | `15-splunk-t1053-005-dashboard.png` | Splunk dashboard panel |
| 16 | `16-splunk-t1053-005-alert-config.png` | High-severity alert configuration |
| 17 | `17-splunk-t1053-005-alert-triggered.png` | Triggered Splunk alert |
| 18 | `18-t1053-005-email-alert.png` | SMTP email notification |
| 19 | `19-scheduled-task-cleanup.png` | Task deletion and deletion events |
| 20 | `20-scheduled-task-recovery.png` | Task absence, service health and SIEM connectivity |

All screenshots must be sanitized before publication. Passwords, application passwords, tokens, cookies and personal email addresses must not be visible.

