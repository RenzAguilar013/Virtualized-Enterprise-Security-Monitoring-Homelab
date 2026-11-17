# OWASP A05 Product Search SQL Injection Validation

**Status: Completed — test and detection evidence reviewed.**

Activity date: 17 November 2026 (Asia/Manila). The controlled product-search
tests demonstrated Boolean SQL injection. This exercise also validated Nginx
and Splunk visibility, a scheduled Splunk alert and delivered email, and Wazuh
detection of the single-quote probe. Detection coverage limitations are recorded
below. Publication cleanup remains: redact authorization/cookie values in
screenshots 03, 04, and 06 before presenting those copies as sanitized evidence.

## Scope and environment

| Component | Address or setting |
|---|---|
| Test source | Kali, `10.10.10.40`, Burp Suite Community Edition |
| Target | WEB01, `http://10.10.10.70` |
| Endpoint | `GET /rest/products/search?q=...` |
| Application path | Nginx port 80 to Docker loopback `127.0.0.1:3000` |
| Wazuh manager | `10.10.10.30` |
| Registered Wazuh agent | `WEB01-10.10.10.70` |
| Splunk | `10.10.10.60`, index `web` |
| Access log | `/var/log/nginx/juice-shop-access.log` |

Tests were limited to the owner-operated homelab. No database modification
or user-table extraction was needed.

## Observed results

| Test or validation | Observed result | Evidence |
|---|---|---|
| Infrastructure baseline | Juice Shop running, port 3000 bound to 127.0.0.1; Docker, Nginx and Wazuh agent active | 01 |
| Splunk forwarder baseline | systemd reported inactive, but the separate Splunk status command showed splunkd and helpers running | 01 |
| Normal search | `orange` returned HTTP 200 and Orange Juice product data | 02 |
| Single quote | `q=%27` returned HTTP 200 with `"data":[]`; no SQL error shown | 03 |
| Boolean true | `apple%')) AND 1=1-- ` returned HTTP 200 and product data at 08:06:51 UTC | 04 |
| Boolean false | `apple%')) AND 1=0-- ` returned HTTP 200 and `"data":[]` at 08:08:02 UTC | 05 |
| Nginx visibility | Access entries contain both Boolean conditions from 10.10.10.40 with status 200 | 06 |
| Splunk investigation | Decoded true/false requests, source IP, host and statuses visible in timeline | 07 |
| Scheduled alert | A05 alert entries visible at 08:19 and 08:20 UTC | 08 |
| Email delivery | Received A05 email includes WEB01, 10.10.10.40, both Boolean conditions and HTTP 200 | 09 |
| Wazuh quote detection | Rule 31164, level 6, description "SQL injection attempt.", URL containing `q=%27` | 10 |
| Post-test recovery | Normal `apple` search returned HTTP 200 and product data; Nginx configuration test succeeded | 11 |
| Boolean rule diagnostic | web-accesslog decoder extracted the encoded Boolean URL; rule 31100, level 0, "Access log messages grouped." | Additional logtest image |

The true/false response difference demonstrates control of SQL query behavior
in this test. No exact product count is claimed because the full true-condition
array is not visible in the capture. An HTTP status or SIEM alert by itself
would not establish this finding.

## Test method and request evidence

Use [burp-test-cases.md](burp-test-cases.md) for exact encoded request lines.
The baseline screenshot uses `orange`; the Boolean pair and recovery search
use `apple`.

1. Record the container, services and normal product-search response.
2. Send the encoded single quote and record the actual status and body.
3. Send the matching Boolean true and false conditions in Burp Repeater.
4. Compare the response bodies and correlate requests with Nginx and Splunk.
5. Validate the scheduled alert, email delivery and Wazuh detection.
6. Restore a normal search and check Nginx configuration.

## Wazuh detection and coverage gap

The working filter was:

```text
agent.name: "WEB01-10.10.10.70" AND location: "/var/log/nginx/juice-shop-access.log"
```

Screenshot 10 shows built-in rule `31164`, level `6`, detecting the
single-quote probe. It does not show a Boolean-condition alert.

The additional logtest screenshot uses a complete Boolean true-condition
Nginx entry. Phase 2 decodes it using `web-accesslog`, including source
`10.10.10.40`, HTTP status `200`, and the encoded URL. Phase 3 matches
`31100`, level `0`, "Access log messages grouped." This explains the tested
line's lack of a SQL-injection alert in that rule test; it does not independently
verify live forwarding. No custom A05 Wazuh rule was implemented. Extending and
retesting Boolean alert coverage is an optional follow-up improvement.

## Splunk investigation and alerting

[splunk-searches.spl](splunk-searches.spl) contains the investigation and
exercise-specific detection searches. Screenshot 07 shows both conditions.
Screenshot 08 establishes that the scheduled A05 alert fired. Screenshot 09
separately establishes email receipt with the matching source and requests.

The intended schedule is every minute over five minutes, with suppression to
limit duplicate notifications. The screenshots do not verify every saved
schedule or suppression setting. This search detects the exercise strings;
it is not a comprehensive SQL-injection detector.

## Recovery and limitations

Screenshot 11 shows a successful normal product search and successful
`nginx -t`. It does not show usable output from all service-status and
container-listing commands, so complete post-test service health and port binding
are not claimed. The baseline separately shows the loopback binding and a running
Splunk forwarder despite its inactive systemd unit.

No application fix or remediation retest was performed. Juice Shop remains
an intentionally vulnerable training target.

## Remediation recommendations

Use parameterized queries with bound values, least-privileged database access,
and generic client-facing errors with useful server-side logs. Validate search
types and lengths. Alerting does not replace query parameterization.

## Uploaded evidence

See [the screenshot manifest](screenshots/README.md) for the actual filenames,
reviewed results and publication cleanup. Twelve PNGs are present: screenshots
01–11 plus a separate logtest capture that also uses prefix 11. Screenshot names
alone were not used to establish completion; the displayed results were reviewed.

## References

- [OWASP A05 2025 Injection](https://top10.owasp.org/2025/A05_2025-Injection/)
- [Juice Shop product-search implementation](https://github.com/juice-shop/juice-shop/blob/master/routes/search.ts)
- [Wazuh decoder and rule testing](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html)
