# OWASP Top 10:2025 Validation Program

This directory tracks authorized web application security exercises performed against the owner-controlled OWASP Juice Shop instance on WEB01 (`10.10.10.70`). Each activity combines Burp Suite evidence with Nginx, Wazuh, and Splunk telemetry.

## Progress

| Category | Activity | Status |
|---|---|---|
| A01:2025 Broken Access Control | Unauthenticated access to exposed `/ftp/` resources | Completed |
| A02:2025 Security Misconfiguration | Directory listing and exposed document access | Completed |
| A03:2025 Software Supply Chain Failures | Container and dependency assessment | Planned |
| A04:2025 Cryptographic Failures | Transport, token, and sensitive-data review | Planned |
| A05:2025 Injection | Product-search SQL injection and SIEM detection review | In progress; single-quote Wazuh alert reported |
| A06:2025 Insecure Design | Business-logic and abuse-case analysis | Planned |
| A07:2025 Authentication Failures | Login and rate-limit validation | Completed |
| A08:2025 Software or Data Integrity Failures | Image and deployment integrity validation | Planned |
| A09:2025 Security Logging and Alerting Failures | Detection coverage and logging-gap analysis | Planned |
| A10:2025 Mishandling of Exceptional Conditions | Controlled malformed-request and error-handling review | Planned |

## Completed activity

- [A01/A02 Exposed Resources](A01-A02-exposed-resources/OWASP-A01-A02-Exposed-Resources-Validation.md)
- [A07 Authentication Failures](A07-authentication-failures/OWASP-A07-Authentication-Failures-Validation.md)

## Activity in progress

- [A05 Product Search SQL Injection](A05-injection/OWASP-A05-SQL-Injection-Validation.md)
  records the confirmed single-quote Wazuh alert separately from Boolean response,
  Splunk alert/email, and recovery results awaiting evidence confirmation.

## Shared configuration

The `shared-config/` directory contains the WEB01 Nginx, Splunk Universal Forwarder, and Wazuh log-collection configuration used across the web-security exercises.

## Evidence standard

Every activity should document scope, authorization, baseline, request and response evidence, server telemetry, Wazuh detection, Splunk investigation, alerting, remediation, cleanup, and retest results. Do not publish credentials, tokens, cookies, private email addresses, or unnecessary confidential-looking content.
