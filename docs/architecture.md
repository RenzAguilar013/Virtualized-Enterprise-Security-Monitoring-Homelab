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
