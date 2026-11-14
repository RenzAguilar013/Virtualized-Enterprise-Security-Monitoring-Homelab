# T1087.002 Evidence Index

These screenshots document the authorized domain-account discovery exercise from baseline through recovery.

| File | Evidence shown |
|---|---|
| `01-windows-domain-identity.png` | Renz identity, address and domain membership. |
| `02-dc01-connectivity-baseline.png` | DNS resolution and LDAP/SMB connectivity to DC01. |
| `03-monitoring-services-baseline.png` | Sysmon, Wazuh and Splunk Forwarder health. |
| `04-event-channels-baseline.png` | Security, Sysmon and PowerShell channels enabled. |
| `05-splunk-telemetry-baseline.png` | Baseline Windows telemetry in Splunk. |
| `06-active-directory-tools-baseline.png` | Active Directory module and native `net.exe` availability. |
| `07-controlled-domain-account-discovery.png` | Read-only `net user /domain` execution. |
| `08-local-sysmon-domain-discovery.png` | Local Sysmon Event `1` with command-line evidence. |
| `09-splunk-t1087-002-sysmon-detection.png` | Splunk Sysmon correlation for native discovery. |
| `10-wazuh-t1087-002-investigation.png` | Wazuh process and command-line investigation. |
| `11-powershell-domain-account-discovery.png` | Read-only `Get-ADUser` or ADSI discovery. |
| `12-local-powershell-event-4104.png` | Local Event `4104` script-block evidence. |
| `13-splunk-powershell-discovery.png` | Splunk PowerShell discovery correlation. |
| `14-splunk-t1087-002-dashboard.png` | Combined discovery dashboard panel. |
| `15-splunk-t1087-002-alert-config.png` | High-severity scheduled alert configuration. |
| `16-splunk-t1087-002-alert-triggered.png` | Triggered-alert record. |
| `17-t1087-002-email-alert.png` | SMTP notification received. |
| `18-endpoint-siem-recovery-validated.png` | Domain trust, monitoring services and SIEM connectivity validated. |

## Publication checklist

- Redact real personal, administrative or service-account names when appropriate.
- Redact passwords, application passwords, tokens, cookies and unrelated personal information.
- Keep the complete query, time range and important result columns visible.
- Confirm that every file has the exact name above and no duplicated extension.

