# T1070.004 Evidence Index

Upload the sanitized screenshots below to this directory.

| File | Evidence shown |
|---|---|
| `01-windows-endpoint-identity.png` | Windows 11 endpoint identity and address. |
| `02-monitoring-services-baseline.png` | Sysmon, Wazuh and Splunk Forwarder service baseline. |
| `03-event-channels-baseline.png` | Sysmon and PowerShell Operational channels enabled. |
| `04-local-sysmon-baseline.png` | Recent local Sysmon activity. |
| `05-splunk-sysmon-baseline.png` | Sysmon telemetry received by Splunk. |
| `06-windows-file-auditing-config.png` | File System audit policy and scoped deletion SACL. |
| `07-test-file-created.png` | Harmless marker-file creation and metadata. |
| `08-test-file-hash-baseline.png` | SHA-256 integrity baseline and existing-file confirmation. |
| `09-sysmon-file-create-event-11.png` | Sysmon Event `11` for the marker. |
| `10-controlled-file-deletion.png` | Exact-path deletion, timestamp and `Test-Path=False`. |
| `11-local-security-file-deletion-events.png` | Local Security Event `4663` and related `4660` activity. |
| `12-splunk-t1070-004-detection.png` | Splunk correlation of the user, process, object and DELETE access. |
| `13-wazuh-t1070-004-investigation.png` | Wazuh investigation or documented alert-index visibility gap. |
| `14-splunk-t1070-004-dashboard.png` | T1070.004 dashboard panel. |
| `15-splunk-t1070-004-alert-config.png` | Scheduled high-severity alert and actions. |
| `16-splunk-t1070-004-alert-triggered.png` | Triggered-alert record. |
| `17-t1070-004-email-alert.png` | SMTP notification received. |
| `18-endpoint-recovery-validated.png` | Test artifacts removed and monitoring services healthy. |

## Publication checklist

- Redact passwords, application passwords, tokens, cookies and unrelated personal information.
- Confirm that the screenshots are PNG files with the exact names above.
- Do not append duplicate extensions such as `.png.png`.
- Keep the full query, time range and important result columns visible where applicable.

