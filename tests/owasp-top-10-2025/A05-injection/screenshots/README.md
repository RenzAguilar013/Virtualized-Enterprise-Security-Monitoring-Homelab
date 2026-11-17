# A05 Screenshot Manifest

The twelve PNGs below are uploaded and were reviewed. The lab activity is
complete with the limitations documented in the [report](../OWASP-A05-SQL-Injection-Validation.md).
Screenshots 03, 04, and 06 require replacement with copies that fully redact
authorization/cookie values. Do not change the payloads, responses or log results.

| Filename | Reviewed evidence |
|---|---|
| `01-a05-web01-search-baseline.png` | Container/loopback baseline; service checks and separate running Splunk process status |
| `02-a05-burp-normal-search.png` | Normal orange search: HTTP 200 and product data |
| `03-burp-single-quote-response.png` | Single quote: HTTP 200, empty array; authorization token needs redaction |
| `04-burp-boolean-true.png` | True condition: HTTP 200 and product data; authorization/cookie values need redaction |
| `05-burp-boolean-false.png` | False condition: HTTP 200 and empty array |
| `06-nginx-search-test-timeline.png` | Both Boolean requests in Nginx; Burp authorization token needs redaction |
| `07-splunk-search-test-timeline.png` | Decoded requests, timestamps, source and HTTP statuses |
| `08-splunk-triggered-alert.png` | Scheduled A05 alert fired |
| `09-email-notification.png` | A05 email received with both conditions and source IP |
| `10-wazuh-sql-injection-alert.png` | Quote probe detected by rule 31164, level 6 |
| `11-recovery-validation.png` | Normal search works; nginx -t succeeds; other post-test service output not demonstrated |
| `11-wazuh-logtest-before-custom-rule.png` | Boolean true logtest: decoder web-accesslog; rule 31100, level 0 |

The two files prefixed 11 are distinct recovery and diagnostic captures.
No custom A05 rule or implemented application fix is claimed.
