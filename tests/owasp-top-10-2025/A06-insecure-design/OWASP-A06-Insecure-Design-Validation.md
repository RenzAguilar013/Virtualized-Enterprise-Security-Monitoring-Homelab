# OWASP 2025 A06 Insecure Design Validation

Activity date: **25 November 2026 (Asia/Manila)**. The supplied terminal/log
timestamps are UTC; Splunk displays time according to its user configuration.

## Objective

Define and test a shopping-basket business rule, observe server behavior when
a request bypasses the normal UI workflow, and detect negative-quantity
submissions using Wazuh and Splunk.

## Scope and authorization

Testing used the owner-controlled Juice Shop homelab and the operator's fictional
lab account. Requests targeted the account's own basket item, observed as
`/api/BasketItems/9`. The item ID is an example from this run, not a universal
Juice Shop identifier. Each negative update was followed by restoration to `1`.
No checkout, financial transaction, other user's basket or account takeover is claimed.

**Completion:** The operator reported the activity complete. Directly reviewed
evidence establishes quantity-aware JSON ingestion and successful scheduled
search runs with matching results and email-action selection. The operator
confirmed that the basket displayed `-1`, returned to `1` after restoration,
and later reported the notification appearing. The finding, live alert, received
email and final recovery screenshots remain to be uploaded for independent review.

**Open finding:** restoring the basket is cleanup; server-side quantity validation
was not patched or remediation-retested.

## Environment

| Component | Address / setting | Purpose |
|---|---|---|
| Kali | `10.10.10.40` | Burp browser, HTTP history and authenticated Repeater tests |
| WEB01 | `10.10.10.70` | Nginx, Docker, Juice Shop, Wazuh Agent and Splunk forwarder |
| Juice Shop | `127.0.0.1:3000` on WEB01 | Owner-controlled application behind Nginx |
| Wazuh | `10.10.10.30` | JSON collection and custom negative-quantity rule |
| Splunk | `10.10.10.60` | Index `web`, sourcetype `nginx:a06:json`, scheduled alert/email |
| Endpoint observed | `PUT /api/BasketItems/9` | Quantity update for the operator's test item |

## Security mapping and design requirement

- **OWASP A06:2025 — Insecure Design:** abuse-case and business-rule assessment.
- **Stated invariant:** a retained basket item must have a positive integer quantity;
  the server must enforce the rule independently of UI controls.
- **Threat case:** an authenticated user submits a negative quantity directly to
  the update API, causing an unintended basket state or total.
- **NIST CSF 2.0:** Identify the business risk, Protect through server-side controls,
  Detect through relevant telemetry, and investigate and clean up test changes.

An observed validation gap does not by itself establish whether its root cause
was a design omission or an implementation defect. This is a bounded assessment
of the A06 business-rule requirement, not comprehensive proof about the entire design.

## Test method

1. Established a normal basket containing one test product with quantity `1`.
2. Used the UI to change the quantity and captured the actual update request in Burp.
3. Replayed the authenticated update to `/api/BasketItems/9`; the operator reported HTTP `200`.
4. Changed only the JSON quantity to `-1`, observed the negative value in the basket,
   then restored `1` and refreshed to verify restoration.
5. Reviewed WEB01 access records showing the basket PUTs, source and status.
6. Added a separate quantity-aware JSON log, retaining earlier access/transport logs.
7. Verified extraction using a normal authenticated quantity-`1` update and
   unauthenticated WEB01 diagnostic requests returning `401`.
8. Configured Wazuh collection and custom rules; the operator confirmed valid
   configuration and active agent/manager status.
9. Configured Splunk single-line parsing and collection, then reviewed actual indexed records.
10. Created a minutely negative-quantity alert with an investigation message and inline email results.
11. Generated a fresh authenticated negative update after detection configuration,
    restored the item, and checked the live-alert workflow.
12. Diagnosed an email failure from scheduler and mail-action logs; a later
    notification was reported as visible.
13. Completed the recovery procedure per operator report; the final capture is pending upload.

## Finding

**Title:** Negative quantity accepted into the operator's shopping basket

**Training-scenario risk:** High for a business rule that governs quantities and totals.
Actual financial impact was not tested.

### Results and evidence

| Check | Observed / reported outcome |
|---|---|
| Normal authenticated update | HTTP `200`; reviewed JSON/Splunk records contain quantity `1` |
| Negative update | Operator confirmed `-1` appeared in the basket; finding image not supplied |
| Restoration | Operator explicitly confirmed changing back to `1` and refreshing restored `1` |
| Normal JSON telemetry | Source `10.10.10.40`, method `PUT`, URI `/api/BasketItems/9`, quantity `1`, status `200` |
| Unauthenticated logging diagnostics | Source `10.10.10.70`, quantity `1`, status `401`; collection still recorded the request |
| Scheduled detection | Reviewed scheduler rows show `success`, `result_count=1`, and `email` on several runs |
| Email delivery issue | Supplied logs show temporary name-resolution failure for the configured SMTP host |
| Later notification | Operator reported it appeared; received-email image pending upload |

A successful status alone is not proof of a persisted business-logic change.
The operator's basket-state observation is separate from the response code.
The detection below flags a submitted negative integer and preserves status
for investigation; it does not automatically prove persistence or financial loss.

## Detection implementation

### Nginx: quantity-specific metadata

The active site is `/etc/nginx/sites-enabled/juice-shop`. An additional log was
added inside the existing server block:

```nginx
access_log /var/log/nginx/juice-shop-a06-basket.log a06_basket if=$a06_log_basket;
```

The existing combined and A04 transport logs were retained. Separate directives
on the same line remain valid when separated by semicolons.

`/etc/nginx/conf.d/a06-basket-telemetry.conf`:

```nginx
map "$request_method:$uri" $a06_log_basket {
    default 0;
    "~^PUT:/api/BasketItems/[0-9]+$" 1;
}

map $request_body $a06_requested_quantity {
    default "unparsed";
    '~"quantity"\s*:\s*(?<a06_quantity_capture>-?[0-9]+)\s*[,}]' $a06_quantity_capture;
}

log_format a06_basket escape=json
  '{"integration":"a06_basket","asset":"WEB01",'
  '"timestamp":"$time_iso8601","src_ip":"$remote_addr",'
  '"scheme":"$scheme","method":"$request_method",'
  '"uri":"$uri","request_uri":"$request_uri",'
  '"requested_quantity":"$a06_requested_quantity",'
  '"status":"$status"}';
```

This extraction is scoped to the small integer-quantity updates used in the lab.
It is not a complete JSON parser or schema validator. An unavailable/unrecognized
body produces `unparsed`, which must not be interpreted as a valid quantity.
The emitted record includes the derived quantity and metadata, not authorization
headers, cookies or the full request body.

An early filtered command produced `Expecting value: line 1 column 1` because
no JSON reached the formatter. Raw log inspection showed that normal Burp
requests lacked the requested URL marker. Logging and quantity extraction were
working; the final investigation and alert do not require markers.

### Wazuh collection and rule

WEB01 local configuration:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/nginx/juice-shop-a06-basket.log</location>
  <only-future-events>yes</only-future-events>
</localfile>
```

The operator confirmed `wazuh-logcollector -t` succeeded and the agent was active.
The [custom rule file](wazuh-rules.xml) was validated with `wazuh-analysisd -t`
before restarting the manager; valid/active status was also confirmed.

| Rule | Level | Purpose |
|---|---:|---|
| `119994` | 0 | JSON integration `a06_basket`, asset `WEB01` |
| `119995` | 10 | Parent match, basket-item PUT path and canonical negative integer quantity |

The child rule does not require a `200` status. It also detects rejected negative
submissions; investigate the preserved status and actual basket state. A normal
quantity-`1` event does not meet the negative-quantity condition. Parent rule
`119994` is not expected in the normal alert index because its level is zero.

### Splunk parsing and collection

Server app `/opt/splunk/etc/apps/a06_basket/local/props.conf`:

```ini
[nginx:a06:json]
SHOULD_LINEMERGE = false
LINE_BREAKER = ([\r\n]+)
DATETIME_CONFIG = CURRENT
KV_MODE = json
```

Server app `metadata/default.meta`:

```ini
[props]
export = system
```

WEB01 forwarder input:

```ini
[monitor:///var/log/nginx/juice-shop-a06-basket.log]
disabled = false
index = web
sourcetype = nginx:a06:json
```

The reviewed screenshot confirms the actual index, source, sourcetype, extracted
quantity and status. `_time` uses receipt time while the Nginx `timestamp` remains
available. Initial backfill records shared receipt times; subsequent records
arrived with later receipt times. Searches use automatic JSON extraction,
without redundant `spath`.

## Scheduled alert and email

The exact search is block 3 of [splunk-searches.spl](splunk-searches.spl).
It converts `requested_quantity` to a number and filters values below zero.

| Setting | Documented workflow |
|---|---|
| Title | `OWASP A06 - Negative Basket Quantity Submission` |
| Schedule | Every minute: `* * * * *` |
| Earliest / latest | `-5m` / `now` |
| Condition | Number of results greater than zero |
| Trigger mode | Per result, as instructed; saved-settings image pending |
| Intended suppression | 5 minutes by `asset,src_ip,uri,quantity` |
| Actions | Add to Triggered Alerts (High); Send Email |
| Recipient | Existing lab notification recipient; personal address omitted |
| Results | Inline table |

Throttling was temporarily disabled during troubleshooting. The operator was
instructed to restore the documented five-minute suppression after delivery;
the final saved settings have not been independently reviewed.

Email subject:

```text
OWASP A06 - Negative Basket Quantity Submission
```

Email message:

```text
OWASP A06:2025 — Insecure Design

A basket-update request submitted a negative quantity.

Asset: $result.asset$
Source IP: $result.src_ip$
Endpoint: $result.uri$
Submitted quantity: $result.quantity$
Matching requests: $result.attempts$
HTTP status(es): $result.http_statuses$
First seen: $result.first_seen$
Last seen: $result.last_seen$

Investigation:
Correlate the request with Wazuh rule 119995 and inspect the basket state. A successful HTTP response alone does not prove that the negative quantity persisted.

For the controlled lab test, restore quantity 1 and verify the basket afterward.
```

### Controlled validation sequence

Use the actual authenticated request for the operator's own item. For this run:

1. `PUT /api/BasketItems/9` with `{"quantity":1}` establishes the normal control.
2. After collection and alerts are active, send one update with `{"quantity":-1}`.
3. Record the response and verify the actual basket state.
4. Restore `{"quantity":1}` and refresh the basket.
5. Correlate the negative event in Wazuh, Splunk, the scheduled alert and email.

Keep authentication in the working Burp request; exclude it from public captures.
No token, cookie or actual credential is stored in these repository files.

## Email troubleshooting outcome

The reviewed scheduler capture shows runs between approximately 05:24 and 05:29
UTC with `status=success` and `result_count=1`. Several rows list the `email`
action. This establishes matching scheduled results and action selection,
not SMTP delivery by itself.

Supplied mail-action logs identify the failure:

```text
[Errno -3] Temporary failure in name resolution
SMTP host: smtp.gmail.com:587
Unable to create SMTP Object
```

Personal recipient addresses and internal job identifiers are omitted here.
The failure occurred during hostname resolution before an SMTP connection;
changing alert suppression did not resolve that DNS lookup failure.
The operator subsequently reported the notification appearing. No specific DNS
configuration change or permanent network repair was recorded, so neither is claimed.

## Recovery validation and limits

The operator explicitly confirmed that changing back to `1` and refreshing
restored the basket. Normal authenticated quantity-`1` updates with HTTP `200`
are present in reviewed telemetry. The final guided checks covered:

- A fresh authenticated restoration and a refreshed basket showing `1`.
- Successful Nginx configuration validation and active Nginx/Docker/Wazuh-agent services.
- A running forwarder and Juice Shop container with loopback-only port binding.
- A fresh restored-quantity event in the Splunk browser search.

The operator declared the activity complete after those steps. Final service
outputs and Screenshot 08 were not supplied for review; do not substitute the
earlier normal-control screenshot for a reviewed final recovery capture.
Restoration is cleanup, not a fix to the server's validation logic.

## Recommended remediation

- Define valid quantity ranges, item-removal semantics and total-calculation rules.
- Enforce integer type and bounds server-side before persisting an update.
- Calculate prices and totals from trusted server-side values.
- Test negative, zero, fractional, oversized and malformed values according to
  the defined business rules, including concurrency and persistence behavior.
- Add application audit events that distinguish submitted, rejected and persisted changes.
- Retest the application and detections after an actual remediation.

No application patch, completed checkout or remediation retest occurred.

## Related files

- [Splunk searches](splunk-searches.spl)
- [Wazuh queries](wazuh-queries.txt)
- [Wazuh rules](wazuh-rules.xml)
- [Concise portfolio evidence checklist](screenshots/README.md)
- [Shared WEB01 configuration](../shared-config/)

## References

- [OWASP A06:2025 Insecure Design](https://top10.owasp.org/2025/A06_2025-Insecure_Design/)
- [Juice Shop companion guide: input validation](https://help.owasp-juice.shop/part2/improper-input-validation.html)
- [Nginx map module](https://nginx.org/en/docs/http/ngx_http_map_module.html)
- [Wazuh custom rules](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html)
- [Splunk email notification action](https://help.splunk.com/en/splunk-enterprise/alert-and-respond/alerting-manual/10.2/configure-alert-actions/email-notification-action)
