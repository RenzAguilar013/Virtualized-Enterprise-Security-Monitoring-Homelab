# A05 Screenshot Manifest

The filenames below define the final evidence layout. No A05 screenshots are
included in this update. Baseline capture was reported, and a Wazuh SQL-injection
alert was reported; other outcomes need confirmation from original evidence.

| Filename | Required evidence |
|---|---|
| `01-baseline.png` | WEB01 service or application baseline |
| `02-burp-normal-search.png` | Normal search request and response |
| `03-burp-single-quote-response.png` | Exact single-quote request and actual status/body |
| `04-burp-boolean-true.png` | True-condition request and actual returned products |
| `05-burp-boolean-false.png` | False-condition request and actual returned products |
| `06-nginx-search-test-timeline.png` | Nginx entries with source, URI, timestamp and status |
| `07-splunk-search-test-timeline.png` | Splunk timeline with decoded test URIs |
| `08-splunk-triggered-alert.png` | A05 triggered alert and result |
| `09-email-notification.png` | Received A05 alert email, if verified |
| `10-wazuh-sql-injection-alert.png` | Wazuh alert for the observed %27 request, with rule and full log |
| `11-recovery-validation.png` | Normal search and/or WEB01 service/container health |

Screenshot 10 may document the single-quote alert; Boolean payload detection is
not required for that capture. Include rule.id, rule.description, timestamp,
agent.name, location, and the complete relevant request. Do not infer Boolean
coverage from the presence of %27 alone.

Screenshot 11 is recovery validation. A logtest screenshot is optional diagnostic
evidence and can use `12-wazuh-logtest-diagnostic.png` if performed.

Use sanitized original screenshots. Fully redact authorization and cookie values,
tokens, passwords, and personal email addresses. Do not alter payloads, statuses,
response bodies, or rule details. Do not use generated recreations as evidence.
