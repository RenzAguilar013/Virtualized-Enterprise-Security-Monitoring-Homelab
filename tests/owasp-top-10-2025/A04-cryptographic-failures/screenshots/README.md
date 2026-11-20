# A04 Portfolio Evidence

Keep the evidence set concise. Configuration, editor, service-restart and repeated
search screenshots are working notes rather than additional portfolio requirements.
Original capture numbers are retained so existing files do not need renaming.

## Published, reviewed captures

| Capture | What it establishes |
|---|---|
| [02 — Kali HTTP/HTTPS check](02-a04-kali-http-https-transport-check.png) | HTTP 200 and failed connection to port 443; not a certificate/cipher assessment |
| [12 — Wazuh A04 logtest](12-a04-wazuh-http-login-rule-logtest.png) | Actual JSON fields and rule 119991, level 10 |
| [20 — Recovery validation](20-a04-recovery-validation.png) | WEB01 application/services/container health and fresh recovery GET indexed in Splunk |

## Important captures awaiting upload

| Existing filename | Status / purpose |
|---|---|
| `04-a04-kali-cleartext-login-packet-capture.png` | Operator reported completion; direct wire evidence with fictional credentials; image not supplied |
| `13-a04-wazuh-live-http-login-alert.png` | Live collection proof; image not supplied |
| `18-a04-splunk-triggered-http-login-alert.png` | Scheduled alert, trigger time and matching validation ID; image not supplied |
| `19-a04-email-alert-notification.png` | Received message and inline results for the same validation ID; image not supplied |

The login-field investigation image `16-a04-splunk-http-login-transport-fields.png`
can be retained as optional detail. The recovery capture already demonstrates
JSON field extraction and continued Splunk collection.

The live-alert and notification workflow was configured in the activity. The
absence of supplied images is an evidence-availability limitation, not a claim
that those actions failed. No synthetic screenshots substitute for missing captures.

## Publication notes

Use fictional test credentials in packet evidence. Personal email addresses,
real credentials and session tokens must be removed from portfolio copies.
The published recovery capture already masks terminal connection labels.

**Recovery passed. HTTPS enforcement remains open.**
