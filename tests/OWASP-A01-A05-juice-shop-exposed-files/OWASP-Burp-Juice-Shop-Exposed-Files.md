# OWASP Juice Shop Exposed-File Detection Validation

## Objective

Build an intentionally vulnerable web target, inspect it through Burp Suite Community Edition, and validate that unauthenticated access to exposed files is visible in Nginx, Wazuh, and Splunk.

## Scope and authorization

This exercise was performed only against the owner-controlled OWASP Juice Shop instance at `10.10.10.70`. The target was isolated inside the homelab and was not exposed through a WAN port-forward.

## Environment

| Component | Address | Purpose |
|---|---:|---|
| Kali Linux | `10.10.10.40` | Burp Suite and test client |
| WEB01 Ubuntu | `10.10.10.70` | Nginx reverse proxy and Docker host |
| OWASP Juice Shop | `127.0.0.1:3000` on WEB01 | Intentionally vulnerable application |
| Wazuh Manager | `10.10.10.30` | Log analysis and custom rule alerting |
| Splunk Enterprise | `10.10.10.60` | Search, investigation, and alerting |

## Security mapping

- **OWASP Top 10 A01:2021 — Broken Access Control:** a user could retrieve content below `/ftp/` without authentication.
- **OWASP Top 10 A05:2021 — Security Misconfiguration:** directory and file exposure increased information-disclosure risk.
- **NIST CSF 2.0:** Identify, Protect, Detect, and Respond functions were exercised through asset definition, service containment, monitoring, alerting, and remediation planning.

## Architecture and telemetry flow

1. Burp Suite on Kali sent authorized requests to `http://10.10.10.70`.
2. Nginx proxied normal application traffic to Juice Shop on loopback port `3000`.
3. Nginx wrote access and error events to dedicated log files.
4. The Wazuh agent evaluated the access log with custom rules.
5. The Splunk Universal Forwarder sent the same logs to the `web` index for investigation and alerting.

## Procedure

### 1. Deploy and contain Juice Shop

Juice Shop was run in Docker with restart persistence and bound only to loopback:

```bash
sudo docker run -d \
  --name juice-shop \
  --restart unless-stopped \
  --memory 1g \
  --cpus 1 \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop
```

Nginx was configured as the only network-facing entry point on `10.10.10.70:80`.

### 2. Confirm telemetry

Requests were confirmed in:

- `/var/log/nginx/juice-shop-access.log`
- Wazuh alerts for agent `WEB01-10.10.10.70`
- Splunk index `web` with sourcetype `nginx:access`

### 3. Configure Burp Suite

Burp Suite Community Edition ran on Kali. Firefox used the proxy listener at `127.0.0.1:8080`, and the lab URL was added to scope. Proxy history and Repeater were used to inspect requests and responses.

### 4. Validate exposed resources

The following requests were tested:

```http
GET /robots.txt HTTP/1.1
Host: 10.10.10.70
Connection: close
```

The actual response was recorded as evidence; the test did not assume that `robots.txt` must exist.

```http
GET /ftp/ HTTP/1.1
Host: 10.10.10.70
Connection: close
```

The `/ftp/` request returned an HTTP `200` directory listing.

```http
GET /ftp/acquisitions.md HTTP/1.1
Host: 10.10.10.70
Connection: close
```

The document request returned HTTP `200` without authentication, confirming the finding.

### 5. Validate detections

Wazuh rule `119971` identifies requests whose decoded URL begins with `/ftp`. Splunk searches correlate the client, URI, status, user agent, and raw event. A scheduled Splunk alert was triggered and its email action was verified after DNS resolution was corrected on the Splunk server.

## Finding

**Title:** Unauthenticated access to exposed Juice Shop files  
**Severity:** High in the training scenario  
**Evidence:** `/ftp/` and `/ftp/acquisitions.md` returned HTTP `200` to an unauthenticated client.

### Risk

Exposed directories and documents can disclose internal information that assists reconnaissance, credential attacks, or follow-on exploitation.

### Recommended remediation

- Remove sensitive files from the web root and deployment image.
- Disable directory indexing unless explicitly required.
- Enforce authorization for protected resources.
- Return `403` or `404` for unauthorized file access.
- Add automated tests for accidental file exposure.
- Retain Nginx, Wazuh, and Splunk monitoring for repeated or unusual requests.

## Validation outcome

The activity demonstrated an end-to-end web security workflow: target deployment, proxy-assisted testing, log collection, custom detection, SIEM investigation, alert triggering, notification testing, and remediation planning.

## Related files

- [`nginx-juice-shop.conf`](nginx-juice-shop.conf)
- [`splunk-forwarder-inputs.conf`](splunk-forwarder-inputs.conf)
- [`splunk-searches.spl`](splunk-searches.spl)
- [`wazuh-agent-localfile.xml`](wazuh-agent-localfile.xml)
- [`wazuh-rules.xml`](wazuh-rules.xml)
- [`wazuh-queries.txt`](wazuh-queries.txt)
- [`screenshots/README.md`](screenshots/README.md)
