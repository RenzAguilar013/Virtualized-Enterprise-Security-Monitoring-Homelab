# OWASP 2025 A07 Authentication Failures Validation

## Objective

Determine whether OWASP Juice Shop limits repeated failed authentication attempts, locks the tested account, or records enough telemetry for detection and response.

## Scope and authorization

Testing was limited to the owner-controlled Juice Shop instance on WEB01 at `10.10.10.70`. A fictional lab-only account was used. The activity was intentionally bounded to five failed attempts per validation sequence.

## Environment

| Component | Address | Purpose |
|---|---:|---|
| Kali Linux | `10.10.10.40` | Burp Suite request capture and replay |
| WEB01 Ubuntu | `10.10.10.70` | Nginx reverse proxy and Juice Shop Docker host |
| OWASP Juice Shop | `127.0.0.1:3000` on WEB01 | Authorized authentication target |
| Wazuh Manager | `10.10.10.30` | Custom single-event and correlation rules |
| Splunk Enterprise | `10.10.10.60` | Investigation, threshold alerting and email notification |

## Security mapping

- **OWASP Top 10 A07:2025 — Authentication Failures**
- **MITRE ATT&CK T1110 — Brute Force**, used only as a detection mapping for repeated authentication failures
- **NIST CSF 2.0:** Protect, Detect and Respond functions through authentication-control review, log correlation, alerting and recovery validation

## Test method

1. Verified Docker, Nginx, Wazuh Agent and Splunk Universal Forwarder health on WEB01.
2. Created a fictional Juice Shop account and captured a successful `POST /rest/user/login` request in Burp.
3. Replayed the request once with an invalid password and recorded HTTP `401 Unauthorized`.
4. Sent five controlled invalid requests within one minute.
5. Confirmed that no HTTP `429` response or temporary account lockout occurred.
6. Immediately replayed the correct fictional credentials and received HTTP `200 OK`.
7. Correlated the HTTP method, URI, source IP and status codes in Nginx and Splunk.
8. Triggered a high-severity Splunk alert and validated the SMTP notification workflow.
9. Added Wazuh rules `119972` and `119973` for individual and repeated failures.
10. Confirmed post-test application, monitoring-agent and SIEM service health.

## Finding

**Title:** Repeated authentication failures accepted without rate limiting or temporary lockout  
**Severity:** High in the training scenario  
**Affected endpoint:** `POST /rest/user/login`

### Evidence

- Five consecutive invalid requests returned HTTP `401`.
- No request returned HTTP `429 Too Many Requests`.
- No temporary lockout or progressive delay was observed.
- A valid login succeeded immediately after the failed sequence.
- Nginx retained the relevant access events.
- Splunk correlated at least five failures from `10.10.10.40`.
- Wazuh rule `119973` generated a level 10 correlation alert.

### Risk

An application that does not throttle repeated authentication failures increases exposure to password guessing, credential stuffing and automated account compromise attempts.

### Recommended remediation

- Apply per-account and per-source rate limiting.
- Introduce progressive delays after repeated failures.
- Use temporary lockout controls that resist denial-of-service abuse.
- Require multi-factor authentication for sensitive accounts.
- Detect distributed attempts across multiple source addresses.
- Avoid revealing whether an email address or account exists.
- Retain alerting for repeated failures followed by a successful login.

## Detection outcome

### Splunk

The detection aggregates HTTP `401` responses for `/rest/user/login` by source address. Five or more results inside the scheduled five-minute window generate a high-severity alert.

### Wazuh

- Rule `119972`: individual `POST /rest/user/login` response with status `401`
- Rule `119973`: five matches of rule `119972` from the same source within 300 seconds

## Recovery validation

The fictional account remained available after testing. Juice Shop, Nginx, Docker, the Wazuh Agent, the Splunk Universal Forwarder and the Wazuh Manager were verified as operational.

## Related files

- [Splunk searches](splunk-searches.spl)
- [Wazuh rules](wazuh-rules.xml)
- [Wazuh queries](wazuh-queries.txt)
- [Evidence checklist](screenshots/README.md)
- [Shared WEB01 configuration](../shared-config/)
