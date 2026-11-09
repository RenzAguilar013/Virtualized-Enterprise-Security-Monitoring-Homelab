# T1562.001 Evidence Index

These screenshots document the authorized Linux01 defense-impairment exercise from baseline through recovery. Sensitive values must be redacted before publication.

| File | Evidence shown |
|---|---|
| `01-linux01-endpoint-identity.png` | Linux01 identity, address and operating-system scope. |
| `02-monitoring-services-baseline.png` | Baseline state of Wazuh Agent, auditd and Splunk Forwarder. |
| `03-local-log-collection-baseline.png` | Local audit/auth/system logging baseline. |
| `04-splunk-log-baseline.png` | Linux01 telemetry visible in Splunk before the test. |
| `05-controlled-wazuh-agent-stop.png` | Authorized stop action and controlled marker. |
| `06-local-systemd-stop-restart.png` | Local systemd service interruption and recovery evidence. |
| `07-splunk-wazuh-agent-interruption.png` | Splunk correlation of the interruption timeline. |
| `08-wazuh-agent-recovered.png` | Wazuh Agent active and connected after recovery. |
| `08-wazuh-post-restart-detection.png` | Wazuh events observed after the agent restarted. |
| `09-splunk-t1562-001-dashboard.png` | T1562.001 dashboard panel. |
| `10-splunk-t1562-001-alert-config.png` | High-severity scheduled alert configuration. |
| `11-splunk-t1562-001-alert-triggered.png` | Triggered-alert record. |
| `12-t1562-001-email-alert.png` | SMTP email alert received. |
| `13-linux01-recovery-validated.png` | Final service, connectivity and system-health validation. |

## Publication checklist

- Confirm that all evidence comes from the private homelab.
- Redact passwords, application passwords, tokens, cookies and unrelated personal data.
- Keep both screenshots numbered `08`; they document different recovery views.
- Do not publish active credentials or secrets in screenshots, Markdown or query files.

