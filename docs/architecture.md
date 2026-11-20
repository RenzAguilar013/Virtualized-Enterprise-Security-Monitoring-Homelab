# Lab Architecture

![Enterprise security homelab topology](../assets/topology.png)

[Editable SVG topology](../assets/topology.svg)

The existing VMware, FortiGate, Active Directory, endpoint and SIEM topology
retains its previously published hosts and addressing.

## Application and monitoring

WEB01 hosts the Nginx-fronted, Dockerized OWASP Juice Shop training application.
Kali supplies controlled web-test traffic. Wazuh agents and Splunk forwarders
provide endpoint and web telemetry.

## A03 assessment flow

Trivy runs locally on WEB01 to generate a CycloneDX SBOM and assess known
component vulnerabilities. Findings are normalized as JSON Lines for custom
Wazuh detection and Splunk investigation, scheduled alerting and email.

A controlled fresh assessment forwards a labeled priority sample to validate
the notification workflow. Scanner severity is distinct from demonstrated
application exploitability. No additional VM or inline security device was added.

See the [A03 summary](../tests/owasp-top-10-2025/A03-software-supply-chain/OWASP-A03-Software-Supply-Chain-Validation.md)
for results and evidence status.


## A04 authentication transport flow

Kali sends a controlled HTTP login POST to Nginx on WEB01. A separate JSON
access log records the server-observed scheme, port, method, URI and status.
WEB01's Wazuh Agent and Splunk Universal Forwarder collect the metadata.
The existing combined access log remains available for earlier web exercises.

| Component | A04 function |
|---|---|
| Kali | Burp/curl requests and scoped capture of its own connection |
| WEB01 | `juice-shop-a04-transport.log`, formatted as single-line JSON |
| Wazuh | JSON decoder, parent `119990`, HTTP-login rule `119991` at level 10 |
| Splunk | Index `web`, sourcetype `nginx:a04:json`, scheduled alert and contextual email |

Detection requires an HTTP scheme and a POST to the login route; neither a
user-agent marker nor an authentication failure alone establishes the finding.
Post-test application availability and fresh Splunk collection were reviewed.
HTTPS remains an open remediation item; no new VM or TLS endpoint was added.

See the [A04 report](../tests/owasp-top-10-2025/A04-cryptographic-failures/OWASP-A04-Cryptographic-Failures-Validation.md)
for configuration snapshots, evidence status and recovery results.
