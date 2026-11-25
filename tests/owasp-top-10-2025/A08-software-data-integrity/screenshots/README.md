# A08 portfolio screenshots

Keep the eight core captures below. Configuration, restarts and troubleshooting
are documented in the report without adding more portfolio screenshots.

The operator reported completing the activity and capture workflow. At this
publication, no A08 image files were attached or present on the repository's
current branch. The supplied gate output and full-record Wazuh rule-test output
were reviewed as text; live-alert, email and final recovery outcomes are recorded
as operator-confirmed. Upload the actual captures here using these exact names.

| Filename | Required visible evidence | File status |
|---|---|---|
| `01-a08-signed-artifact-baseline.png` | Kali: signature Verified OK and approved artifact/public-key SHA-256 baselines | Awaiting upload |
| `02-a08-tampered-artifact-blocked.png` | WEB01: failed / blocked, identical active hashes, staged_content_changed=false | Awaiting upload |
| `03-a08-splunk-integrity-telemetry.png` | Splunk: parsed verification records, outcomes and run IDs | Awaiting upload |
| `04-a08-splunk-alert-settings.png` | Alert title, minutely schedule, five-minute window, per-result trigger, suppression and actions | Awaiting upload |
| `05-a08-wazuh-integrity-alert.png` | Live Wazuh rule 119997, level 10, WEB01, reason and fresh post-fix run ID | Awaiting upload |
| `06-a08-splunk-triggered-alert.png` | Scheduled trigger and matching blocked-run results | Awaiting upload |
| `07-a08-email-notification.png` | Received contextual email, rejection reason and correlated results/run ID | Awaiting upload |
| `08-a08-recovery-validation.png` | Approved artifact passes/stages, hashes match, healthy WEB01 services/HTTP 200 and matching Splunk recovery row | Awaiting upload |

For Screenshot 08, place the successful Splunk recovery row and WEB01 terminal
results together. The known-good gate's new run ID must match the row. An
unchanged hash is expected because rejected input never replaced the approved
staged artifact. A normal pass does not need a blocked-artifact Wazuh alert.

Remove personal mail addresses or secrets from captures before publishing.
The private signing key remains on Kali and is never part of the portfolio.

[Validation report](../OWASP-A08-Software-Data-Integrity-Validation.md)
