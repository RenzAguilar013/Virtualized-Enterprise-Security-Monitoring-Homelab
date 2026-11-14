# T1021.002 Evidence Index

These screenshots document the authorized SMB/Windows Admin Shares exercise from baseline through recovery.

| File | Evidence shown |
|---|---|
| `01-source-endpoint-identity.png` | Renz identity, address and domain membership. |
| `02-dc01-smb-connectivity.png` | DC01 name resolution and TCP `445` connectivity. |
| `03-smb-connections-baseline.png` | SMB mappings before the test. |
| `04-renz-monitoring-services.png` | Renz Sysmon, Wazuh and Splunk Forwarder health. |
| `05-dc01-monitoring-services.png` | DC01 monitoring-service health. |
| `06-splunk-hosts-baseline.png` | Recent Renz and DC01 events in Splunk. |
| `07-dc01-smb-audit-policy.png` | File Share, Detailed File Share and Logon audit policy. |
| `08-controlled-ipc-share-connection.png` | Controlled read-only `IPC$` connection. |
| `09-dc01-network-logon-event-4624.png` | DC01 Event `4624`, Logon Type `3` and source address. |
| `10-dc01-share-events-5140-5145.png` | IPC$ share-access events on DC01. |
| `11-renz-sysmon-smb-network-connection.png` | Renz Sysmon Event `3` to DC01 TCP `445`. |
| `12-splunk-t1021-002-dc01-correlation.png` | Splunk source/destination, account and share correlation. |
| `13-wazuh-t1021-002-investigation.png` | Wazuh SMB investigation. |
| `14-splunk-t1021-002-dashboard.png` | T1021.002 dashboard panel. |
| `15-splunk-t1021-002-alert-config.png` | High-severity scheduled alert configuration. |
| `16-splunk-t1021-002-alert-triggered.png` | Triggered-alert record. |
| `17-t1021-002-email-alert.png` | SMTP notification received. |
| `18-smb-session-cleanup-recovery.png` | IPC$ session removed and endpoint/SIEM health validated. |

## Publication checklist

- Redact passwords, application passwords, tokens, cookies and unrelated personal information.
- Confirm no credential was embedded in a command or screenshot.
- Keep the complete query, time range and important result columns visible.
- Confirm every screenshot has the exact name above and no duplicated extension.

