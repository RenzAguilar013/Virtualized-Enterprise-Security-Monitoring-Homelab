# OWASP A05 Product Search SQL Injection Validation

Exercise 3 uses controlled SQL-injection requests against the owner-operated
Juice Shop product-search API. The objective is to compare application responses
and investigate the resulting Nginx, Splunk, and Wazuh telemetry.

Activity date: 17 November 2026 (Asia/Manila).

## Scope and environment

| Component | Address or setting |
|---|---|
| Authorized test source | Kali, `10.10.10.40`, Burp Suite Community Edition |
| Target | WEB01, `http://10.10.10.70` |
| Endpoint | `GET /rest/products/search?q=...` |
| Application path | Nginx port 80 to Docker loopback `127.0.0.1:3000` |
| Wazuh manager | `10.10.10.30` |
| Registered Wazuh agent | `WEB01-10.10.10.70` |
| Splunk | `10.10.10.60`, index `web` |
| Access log | `/var/log/nginx/juice-shop-access.log` |

Only the isolated homelab target is in scope. Tests use bounded, read-only
requests; no user-table extraction, destructive statements, or data modification
is required.

## Evidence and current status

| Item | Recorded status |
|---|---|
| Baseline and normal-search screenshots | Capture reported; sanitized originals not supplied for this update |
| Single-quote probe | Procedure supplied; exact request/response evidence awaits attachment |
| Wazuh alert | User reports a SQL-injection description and `%27` in the WEB01 Nginx event |
| Wazuh rule ID and level | Not yet recorded |
| Boolean true/false application results | Not yet confirmed |
| Boolean payload visibility in Wazuh | User could not locate `1%3D1` or `1%3D0`; cause not established |
| A05 Splunk timeline, scheduled alert, and email | Procedures supplied; outcomes not yet confirmed |
| Recovery checks | Procedure supplied; outcomes not yet confirmed |

This activity remains in progress. A detected single-quote request is evidence
of alerting on that probe. HTTP 200, a SQL-error response, or a SIEM rule match
alone does not establish successful SQL injection. No custom A05 Wazuh rule or
implemented application fix is claimed.

## Method

1. Verify WEB01 services and capture a normal product-search baseline in Burp.
2. Send the encoded single-quote search request once and record its actual
   status and body; do not assume that an error must occur.
3. Send the Boolean true/false pair from [the test cases](burp-test-cases.md).
   Record both response bodies and product counts. Confirm the query structure
   matches the deployed Juice Shop version before interpreting results.
4. Locate each request in the dedicated Nginx access log and record source,
   timestamp/timezone, URI, and status.
5. Run the searches in [splunk-searches.spl](splunk-searches.spl). Confirm field
   extraction against actual logs before saving the scheduled alert.
6. Use [wazuh-queries.txt](wazuh-queries.txt) to locate WEB01 Nginx alerts. Record
   the existing rule ID, level, description, and complete request.
7. Validate alert/email behavior and recovery separately. Update the status
   table only after the corresponding evidence is available.

## Wazuh observation and investigation

The successful dashboard filter was:

```text
agent.name: "WEB01-10.10.10.70" AND location: "/var/log/nginx/juice-shop-access.log"
```

The user reported a SQL-injection rule description for an event containing
`%27`. `%27` is a single quote. It does not identify the Boolean test by itself,
because the Boolean payload also contains an encoded quote. Inspect the full
request before attributing an event to a particular test.

Capture that alert as Screenshot 10. Do not state that the Boolean tests were
detected unless their complete request URIs are present. Search-field behavior,
encoding, time range, ingestion, and rule matching remain possible explanations
for the missing Boolean events.

If needed, copy a complete Boolean request line from the Nginx log on WEB01
and paste it into `sudo /var/ossec/bin/wazuh-logtest` on the Wazuh manager.
Record Phases 2 and 3 to inspect decoding and matching. Logtest is diagnostic;
it does not verify live forwarding or generate the production dashboard event.

## Splunk alert configuration

Use search 2 from `splunk-searches.spl` after confirming it matches the actual
test requests. Run every minute (`* * * * *`) over the last five minutes;
trigger once when the result count is greater than zero. Use a five-minute
suppression interval to limit duplicate notifications during the exercise.

Keep Add to Triggered Alerts enabled. If testing email, use the established
lab SMTP configuration and recipient. Replay one request after saving and
verify both the triggered alert and email receipt. An alert firing does not
establish email delivery. The search detects specific exercise strings and is
not a comprehensive SQL-injection detector.

## Recovery validation

Repeat the normal `apple` search and record the HTTP status and body. On WEB01:

```bash
sudo nginx -t
sudo systemctl is-active nginx docker wazuh-agent SplunkForwarder
sudo docker ps --filter name=juice-shop
```

Verify that the container is running and port 3000 remains loopback-bound.
Retain monitoring and security controls. No database cleanup should be required
for these read-only requests. Record any unexpected effect before concluding
that recovery passed.

## Remediation guidance

For a production application, use parameterized SQL queries with bound values,
appropriate database privileges, and generic client-facing errors with useful
server-side logging. Validate search length and type, and use allowlists for
dynamic SQL identifiers where applicable. Escaping input or adding an alert
does not replace parameterization. Juice Shop is intentionally vulnerable;
these recommendations have not been implemented or retested in this activity.

## Evidence layout

See [the screenshot manifest](screenshots/README.md). It lists intended filenames
and what each capture must establish. Image files are not present in this update.
Upload sanitized original captures; preserve request and response details, and
redact authorization headers, cookies, tokens, and personal email addresses.
Do not publish AI-generated recreations as test evidence.

## References

- [OWASP A05 2025 Injection](https://top10.owasp.org/2025/A05_2025-Injection/)
- [Juice Shop product-search implementation](https://github.com/juice-shop/juice-shop/blob/master/routes/search.ts)
- [Wazuh decoder and rule testing](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html)
