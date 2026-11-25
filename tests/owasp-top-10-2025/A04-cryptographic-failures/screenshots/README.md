# A04 Portfolio Evidence

The operator uploaded the remaining captures after the initial report. This
checklist keeps the portfolio selection focused; the other uploaded setup images
remain available and have not been removed.

## Selected key captures

| File | Evidence |
|---|---|
| [02 — HTTP/HTTPS check](02-a04-kali-http-https-transport-check.png) | HTTP 200 and failed connection to port 443 |
| [04 — Packet capture](04-a04-kali-cleartext-login-packet-capture.png) | Readable fictional login body on the Kali-to-WEB01 HTTP connection |
| [12 — JSON rule test](12-a04-wazuh-http-login-rule-logtest.png) | Rule 119991, level 10, and decoded transport fields |
| [13 — Live Wazuh alert](13-a04-wazuh-live-http-login-alert.png) | Live WEB01 rule 119991, level 10, POST and status 401 |
| [18 — Triggered Splunk alert](18-a04-splunk-triggered-http-login-alert.png) | Scheduled High alert at 09:04 UTC; listing shows Digest mode |
| [19 — Received email](19-a04-email-alert-notification.png) | Contextual message, populated fields and inline results, including the fresh validation ID |
| [20 — Recovery](20-a04-recovery-validation.png) | Application/agent/container health and fresh GET indexed in Splunk |

The later packet, Wazuh, triggered-alert and inbox images were reviewed during
the 25 November portfolio update. The received email includes two result rows;
the trigger listing shows Digest mode, whereas the guided setup specified
per-result triggering. The report records that difference rather than claiming
that per-result suppression was independently verified.

Personal labels are masked in the received-email capture. Demonstration
credentials in the packet are fictional. HTTPS enforcement remains open.
