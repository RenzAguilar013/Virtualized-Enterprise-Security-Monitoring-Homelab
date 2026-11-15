# Web Application Security Lab

## Purpose

This lab adds a contained web application security workflow to the enterprise monitoring homelab. OWASP Juice Shop provides an intentionally vulnerable target, Burp Suite supports authorized HTTP inspection, and the existing Wazuh and Splunk platforms provide detection and investigation coverage.

## Components

| System | Address | Role |
|---|---:|---|
| FortiGate | `10.10.10.1` | Internal routing and segmentation |
| Kali Linux | `10.10.10.40` | Burp Suite Community Edition and Firefox |
| WEB01 Ubuntu | `10.10.10.70` | Docker, Juice Shop, Nginx, Wazuh agent, Splunk forwarder |
| Wazuh Manager | `10.10.10.30` | Web-log analytics and custom detection |
| Splunk Enterprise | `10.10.10.60` | Search, dashboards, alerting, and email notification |

## Service design

- Juice Shop listens only on `127.0.0.1:3000` on WEB01.
- Nginx listens on `10.10.10.70:80` and reverse-proxies requests to Juice Shop.
- No WAN port-forward exposes the target to the public internet.
- Docker and Nginx are enabled at boot; the container uses `--restart unless-stopped`.
- WEB01 sends dedicated Nginx access and error logs to both Wazuh and Splunk.

## Monitoring paths

| Data | Local path | Destination |
|---|---|---|
| Nginx access | `/var/log/nginx/juice-shop-access.log` | Wazuh and Splunk `web` index |
| Nginx error | `/var/log/nginx/juice-shop-error.log` | Wazuh and Splunk `web` index |

## Burp configuration

Burp Suite Community Edition runs on Kali with a proxy listener on `127.0.0.1:8080`. If Burp's embedded browser cannot start, Firefox can be configured to use that HTTP proxy. Only the homelab target should be included in scope.

## Operational checks

```bash
sudo docker ps --filter name=juice-shop
curl -I http://127.0.0.1:3000
curl -I http://10.10.10.70
sudo nginx -t
sudo systemctl is-active nginx docker wazuh-agent SplunkForwarder
sudo tail -n 20 /var/log/nginx/juice-shop-access.log
```

## Safe-use boundaries

Perform testing only against systems you own or have explicit authorization to assess. Do not publish credentials, session tokens, private email addresses, or full confidential-looking sample content in screenshots.
