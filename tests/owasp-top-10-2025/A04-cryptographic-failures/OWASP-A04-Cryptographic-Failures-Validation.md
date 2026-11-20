# OWASP 2025 A04 Cryptographic Failures Validation

Activity date: **20 November 2026 (Asia/Manila)**. Capture timestamps shown in the
terminal are UTC; Splunk's displayed time follows its configured user timezone.

## Objective

Assess authentication transport protection on the owner-controlled Juice Shop
instance, add scheme-aware telemetry, and build Wazuh and Splunk detections for
login requests sent over HTTP.

## Scope and authorization

The exercise used Kali and WEB01 in the private homelab. Login tests used the
fictional address `a04-transport@example.invalid` and a demonstration password.
No real account password was required. This is a transport-protection assessment
within A04; it does not claim comprehensive testing of encryption at rest,
password storage, token cryptography, or every application cryptographic control.

**Status:** HTTP/HTTPS checks, JSON decoding and the A04 rule match were reviewed.
Post-test application availability and fresh Splunk collection passed. The live
alert and notification workflow was configured during the guided activity;
the corresponding dashboard, triggered-alert and received-email captures have
not been supplied for review. **HTTPS enforcement remains an open finding.**

## Environment

| Component | Address / setting | Purpose |
|---|---|---|
| Kali | `10.10.10.40` | curl, Burp Repeater and capture of its own test traffic |
| WEB01 | `10.10.10.70` | Nginx reverse proxy, Docker, Juice Shop and SIEM agents |
| Juice Shop | `127.0.0.1:3000` on WEB01 | Training application behind Nginx |
| Wazuh | `10.10.10.30` | JSON decoding and custom transport rule |
| Splunk | `10.10.10.60` | Index `web`, sourcetype `nginx:a04:json`, alert/email workflow |
| Nginx site | `/etc/nginx/sites-enabled/juice-shop` | Active site listening on `10.10.10.70:80` |

## Security mapping

- **OWASP A04:2025 — Cryptographic Failures**
- **CWE-319 — Cleartext Transmission of Sensitive Information** and
  **CWE-523 — Unprotected Transport of Credentials** as transport-risk mappings.
- **NIST CSF 2.0:** risk identification, data protection, security monitoring
  and investigation of observed events.
- No credential theft, account takeover or ATT&CK technique execution is claimed.

## Test method

1. Established WEB01 application, container and monitoring-service baselines.
2. Checked HTTP and HTTPS connectivity from Kali.
3. Sent a fictional login POST using Burp Repeater and curl.
4. Captured the test connection on Kali. The operator reported Screenshot 04
   complete; that packet-capture image has not been supplied for review.
5. Correlated the request through Nginx and Splunk using a query-string marker.
6. Tested the original access record in Wazuh; the existing A07 authentication
   rule matched before a dedicated A04 rule was added.
7. Added a separate JSON access log containing scheme, port, method and URI.
8. Configured Wazuh JSON collection and rule `119991` for HTTP login POSTs.
9. Configured Splunk event splitting, JSON extraction, collection and a
   scheduled per-result alert with an email message.
10. Used fresh validation IDs for scheduled-notification tests.
11. Verified post-test application health and ingestion of a fresh recovery GET.

## Finding

**Title:** Login endpoint reachable over unencrypted HTTP without verified HTTPS enforcement

**Training-scenario risk:** High when real credentials are submitted over this
transport. The demonstration used fictional credentials and did not show account compromise.

**Affected endpoint:** `POST /rest/user/login`

### Reviewed evidence

- Kali received HTTP `200` for the normal product-search request without an
  HTTPS redirect.
- The HTTPS connection to `10.10.10.70:443` failed before a TLS handshake.
  This establishes failed connectivity from the test path, not a certificate
  error or a completed TLS/cipher assessment.
- The reviewed active Nginx site listened on `10.10.10.70:80` and proxied to
  `127.0.0.1:3000`.
- An actual JSON record for `A04-RULE-CHECK-01` contained source
  `10.10.10.40`, scheme `http`, port `80`, method `POST`, URI
  `/rest/user/login`, and response status `401`.
- Wazuh logtest decoded that record as JSON and matched rule `119991`, level `10`.

A `401` rejects the credentials after the HTTP request is received; it does not
provide transport encryption. The readable request body in a wire capture is
the intended direct cleartext evidence. Readable content in Burp alone would
not establish cleartext transport, since Burp can also inspect proxied HTTPS.

### Risk and limitations

An observer able to access the network path could read or modify unencrypted
traffic. The exercise captured the test client's own connection, not another
user's traffic. The JSON detection establishes an HTTP POST to the login route;
it does not inspect the body or prove that every matched request carried credentials.

## Detection implementation

### Nginx transport telemetry

The original combined access log was retained for the existing web exercises.
A second log was added at the same server level:

```nginx
access_log /var/log/nginx/juice-shop-access.log combined;
access_log /var/log/nginx/juice-shop-a04-transport.log a04_transport;
```

The HTTP-level format in `/etc/nginx/conf.d/a04-transport-format.conf` is:

```nginx
log_format a04_transport escape=json
  '{"integration":"a04_transport","asset":"WEB01",'
  '"timestamp":"$time_iso8601","src_ip":"$remote_addr",'
  '"scheme":"$scheme","server_port":"$server_port",'
  '"method":"$request_method","uri":"$uri",'
  '"request_uri":"$request_uri","status":"$status",'
  '"user_agent":"$http_user_agent"}';
```

`$uri` supplies the path used by the detection; `$request_uri` also retains the
query-string validation ID. The format collects request metadata, without
passwords or request bodies. The existing combined format does not establish
HTTP versus HTTPS from the `HTTP/1.1` text alone.

### Wazuh collection and rules

The following block was added inside an existing WEB01 `ossec_config` section:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/nginx/juice-shop-a04-transport.log</location>
  <only-future-events>yes</only-future-events>
</localfile>
```

Collection configuration was tested using `wazuh-logcollector -t`. Manager rules
were validated using `wazuh-analysisd -t` before restart in the guided procedure.
The [rule definitions](wazuh-rules.xml) contain:

| Rule | Level | Condition |
|---|---:|---|
| `119990` | 0 | JSON integration `a04_transport`, asset `WEB01` |
| `119991` | 10 | Parent match plus `scheme=http`, `method=POST`, and `uri=/rest/user/login` |

The child rule applies independently of status or user-agent marker. An HTTP
GET or an event with `scheme=https` does not meet its configured conditions.
Those are rule conditions, not a claim that an HTTPS remediation retest occurred.

### Splunk collection and parsing

WEB01 forwarder input:

```ini
[monitor:///var/log/nginx/juice-shop-a04-transport.log]
disabled = false
index = web
sourcetype = nginx:a04:json
```

Splunk server `a04_crypto/local/props.conf`:

```ini
[nginx:a04:json]
SHOULD_LINEMERGE = false
LINE_BREAKER = ([\r\n]+)
DATETIME_CONFIG = CURRENT
KV_MODE = json
```

The app's `metadata/default.meta` exports the extraction settings:

```ini
[props]
export = system
```

`_time` uses receipt time; the Nginx `timestamp` field remains available for
correlation. Final queries use automatic JSON extraction and omit redundant
`spath`. Configuration was applied on the server before enabling the new input.

Early searches did not find the initial test. A fresh URL marker,
`A04-TRANSPORT-CHECK-01`, produced an identifiable event. The original search miss
was not conclusively attributed to an input or forwarding failure.

## Alert and email workflow

The exact scheduled search is block 3 of [splunk-searches.spl](splunk-searches.spl).
It detects HTTP login POSTs regardless of marker or response status. A marker
becomes `validation_id`; ordinary requests use `unmarked`.

| Setting | Configured value |
|---|---|
| Title | `OWASP A04 - Unencrypted HTTP Login` |
| Schedule | Every minute: `* * * * *` |
| Earliest / latest | `-5m` / `now` |
| Condition | Number of results greater than zero |
| Trigger | Per result |
| Suppression | 5 minutes by `asset,src_ip,uri,validation_id` |
| Actions | Add to Triggered Alerts, severity High; Send Email |
| Recipient | Existing lab notification recipient; no personal address published |
| Results | Inline table |

Email subject:

```text
OWASP A04 - Unencrypted HTTP Login
```

Email message:

```text
OWASP A04:2025 — Cryptographic Failures

An HTTP POST to the login endpoint was detected. Unencrypted authentication traffic can expose credentials in transit.

Asset: $result.asset$
Source IP: $result.src_ip$
Transport: $result.scheme$
Server port(s): $result.server_ports$
Endpoint: $result.uri$
HTTP status(es): $result.http_statuses$
Matching requests: $result.attempts$
Validation ID: $result.validation_id$
First seen: $result.first_seen$
Last seen: $result.last_seen$

Investigation:
Correlate the Nginx transport event with Wazuh rule 119991 and the packet-capture evidence. Verify HTTPS availability and enforcement.
```

### Repeatable controlled test

After collection, rules and the saved alert are active, run on Kali:

```bash
TEST_ID="A04-ALERT-$(date -u +%Y%m%dT%H%M%SZ)"
echo "Validation ID: $TEST_ID"
curl --noproxy '*' -i \
  --connect-timeout 5 --max-time 10 \
  -A 'A04-Crypto-Lab-Validation' \
  -H 'Content-Type: application/json' \
  --data-raw '{"email":"a04-transport@example.invalid","password":"A04-Demo-Only-2026!"}' \
  "http://10.10.10.70/rest/user/login?a04_test=$TEST_ID"
```

The fictional login is expected to fail authentication. Correlate the same
validation ID in the JSON record, Wazuh alert, Splunk result, triggered alert
and email. Each new ID separates test notifications from the suppression state
of earlier runs. The workflow uses normal collected traffic and the scheduled
search; manual email dispatch is not equivalent evidence.

## Detection outcome and evidence status

| Check | Evidence status |
|---|---|
| HTTP works; HTTPS connection fails | Reviewed and published Screenshot 02 |
| Existing A07 rule matches original combined-log request | Reviewed working capture; setup image omitted from portfolio |
| A04 JSON rule `119991`, level 10 | Reviewed and published Screenshot 12 |
| Packet capture with fictional cleartext body | Operator reported Screenshot 04 complete; image not supplied |
| Live Wazuh alert and login-field Splunk investigation | Guided steps completed in conversation; screenshots 13/16 not supplied |
| Triggered alert and received email | Workflow configured; screenshots 18/19 not supplied for review |
| Post-test recovery and fresh Splunk ingestion | Reviewed and published Screenshot 20 |

Logtest validates rule matching; it does not itself create a live alert. A
configured email action is distinct from reviewed evidence of received email.
The [screenshot checklist](screenshots/README.md) keeps these statuses explicit.

## Recovery validation

The reviewed capture on 20 November 2026 shows:

- `nginx -t`: syntax and configuration test successful.
- Nginx, Docker and Wazuh Agent: active.
- Splunk forwarder and helper processes: running.
- Juice Shop: running with `127.0.0.1:3000->3000/tcp` binding.
- Normal product search: HTTP `200`, application `success`, three products.
- A fresh `A04-RECOVERY-CHECK-01` GET indexed in Splunk as WEB01, scheme `http`,
  URI `/rest/products/search`, status `200`.

The terminal time was approximately 09:07:32 UTC and the Splunk recovery event
was displayed at 09:07:33 in the capture. This confirms post-test application
availability and continued transport-log collection. It does not establish
Wazuh manager/indexer/dashboard service health or HTTPS remediation.

## Recommended remediation

- Configure a trusted HTTPS endpoint with a certificate valid for the intended
  hostname and a supported TLS policy.
- Publish and use HTTPS login URLs; enforce HTTPS before users enter sensitive
  data. A redirect after a credential POST cannot undo an already exposed body.
- Apply HSTS after HTTPS and certificate trust are correctly deployed and tested.
- Review secure session-cookie handling and internal transport trust boundaries.
- Retest HTTPS trust, HTTP behavior, application function and both SIEM pipelines.

No TLS certificate, HTTPS listener, redirect or HSTS remediation was applied in
this exercise. The intentionally vulnerable target remains on HTTP inside the lab.

## Related files

- [Splunk searches](splunk-searches.spl)
- [Wazuh queries](wazuh-queries.txt)
- [Wazuh rules](wazuh-rules.xml)
- [Portfolio evidence checklist](screenshots/README.md)
- [Shared WEB01 configuration](../shared-config/)

## References

- [OWASP A04:2025 Cryptographic Failures](https://top10.owasp.org/2025/A04_2025-Cryptographic_Failures/)
- [Nginx HTTP log module](https://nginx.org/en/docs/http/ngx_http_log_module.html)
- [Wazuh JSON decoder](https://documentation.wazuh.com/current/user-manual/ruleset/decoders/json-decoder.html)
- [Wazuh custom rules](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html)
- [Splunk event line breaking](https://help.splunk.com/en/splunk-enterprise/get-started/get-data-in/10.6/configure-event-processing/configure-event-line-breaking)
