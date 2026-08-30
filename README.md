# 🏠 Virtualized Enterprise Security & Monitoring Homelab

A virtualized enterprise security homelab built with **VMware Workstation**, **FortiGate**, **Active Directory**, **Wazuh SIEM**, **Splunk Enterprise**, **Kali Linux**, **Nessus**, and Windows/Linux telemetry.

The lab is designed to simulate real-world **SOC monitoring, network security, vulnerability management, attack detection, log collection, SIEM correlation, and alerting**.

---

## 🏗️ Lab Architecture

### Core Technologies

- VMware Workstation
- FortiGate Firewall
- Windows Server 2022
- Active Directory Domain Services (AD DS)
- Internal DNS
- Windows 11 Pro
- Ubuntu 22.04 LTS
- Wazuh SIEM
- Wazuh Agents
- Splunk Enterprise
- Splunk Universal Forwarder
- Kali Linux
- Nmap
- Wireshark
- Nessus Vulnerability Scanner

---

# 🌐 Network Topology
![Enterprise security homelab topology](assets/topology.png)

```text
                              Internet
                                  │
                         VMware NAT (VMnet8)
                         Gateway: 192.168.100.2
                                  │
                         FortiGate port1 (WAN)
                           192.168.100.10/24
                                  │
                         Firewall + NAT + DHCP
                                  │
                         FortiGate port2 (LAN)
                            10.10.10.1/24
                                  │
                     VMware VMnet2 Host-only LAN
                         10.10.10.0/24
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        │                         │                         │
 Windows Server 2022       Windows 11 Pro             Ubuntu 22.04
      DC01                    Client                 Linux Telemetry
   10.10.10.10              10.10.10.20               10.10.10.50
        │                         │                         │
   AD DS + DNS             Sysmon + Windows Logs       Wazuh Agent
        │                   Splunk Forwarder          Linux Telemetry
        │                         │                         │
        │                         └──────────┐              │
        │                                    │              │
        │                                    ▼              │
        │                              Splunk Enterprise    │
        │                                10.10.10.60        │
        │                                    │              │
        │                                    │              │
        └──────────────────────┐             │              │
                               ▼             ▼              ▼
                         Wazuh Manager / SIEM
                              10.10.10.30
                                   │
                         Correlation & Detection
                                   │
                              Alerts / SOC
                                   │
                              Email Alert


                    Kali Linux
                    10.10.10.40
                         │
             ┌───────────┼────────────┐
             │           │            │
            Nmap      Wireshark     Nessus
             │           │            │
             ▼           ▼            ▼
       Reconnaissance  Traffic     Vulnerability
         & Scanning    Capture       Scanning
