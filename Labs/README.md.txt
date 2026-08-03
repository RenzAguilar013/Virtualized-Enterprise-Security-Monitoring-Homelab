# 🏠 Homelab Network Topology

## Overview
This homelab simulates a small enterprise network using VMware Workstation and a FortiGate firewall.

- **Hypervisor:** VMware Workstation
- **Firewall:** FortiGate VM
- **WAN Network:** VMware NAT (VMnet8)
- **LAN Network:** VMware Host-only (VMnet2)
- **DHCP Server:** FortiGate
- **DNS / Active Directory:** Windows Server 2022 (DC01)

---

## Network Topology

```text
                           Internet
                               │
                    VMware NAT (VMnet8)
                  Gateway: 192.168.100.2
                               │
                    FortiGate port1 (WAN)
                    192.168.100.10/24
                               │
                        Firewall + NAT
                               │
                    FortiGate port2 (LAN)
                        10.10.10.1/24
                 DHCP Server (FortiGate)
                               │
               VMware VMnet2 (Host-only LAN)
                      Network: 10.10.10.0/24
                               │
              ┌────────────────┴────────────────┐
              │                                 │
      Windows Server 2022                 Windows 11 Pro
      DC01 (AD DS + DNS)                  Domain Client
      DHCP Reservation                    DHCP Reservation
      10.10.10.10                         10.10.10.20
```

---

## IP Addressing

| Device | Interface | IP Address | Assignment |
|---------|-----------|------------|------------|
| VMware NAT Gateway | VMnet8 | 192.168.100.2 | Static |
| FortiGate | port1 (WAN) | 192.168.100.10/24 | Static |
| FortiGate | port2 (LAN) | 10.10.10.1/24 | Static |
| Windows Server 2022 (DC01) | Ethernet | 10.10.10.10 | DHCP Reservation |
| Windows 11 Pro | Ethernet | 10.10.10.20 | DHCP Reservation |

---

## DHCP Configuration (FortiGate)

| Setting | Value |
|---------|-------|
| DHCP Server | Enabled |
| Interface | port2 |
| Scope | 10.10.10.100 - 10.10.10.200 |
| Default Gateway | 10.10.10.1 |
| DNS Server | 10.10.10.10 |
| Lease Time | 86400 seconds |

### DHCP Reservations

| Device | MAC Address | Reserved IP |
|---------|-------------|-------------|
| DC01 | 00:0C:29:08:CA:A6 | 10.10.10.10 |
| Windows 11 Pro | 00:0C:29:2D:EE:94 | 10.10.10.20 |

---

## Network Flow

Internet
→ VMware NAT (VMnet8)
→ FortiGate WAN (port1)
→ Firewall Policies & NAT
→ FortiGate LAN (port2)
→ FortiGate DHCP Server
→ VMnet2 Host-only Network
→ Windows Server 2022 (AD DS + DNS)
→ Windows 11 Pro Client

---

## Features

- VMware Workstation virtual networking
- FortiGate Firewall & NAT
- FortiGate DHCP Server
- DHCP Reservations
- Active Directory Domain Services
- Internal DNS (DC01)
- Domain-joined Windows 11 Client
- Secure LAN segmentation

---

## Future Improvements

- Deploy Kali Linux VM
- Deploy Ubuntu Server
- Configure FortiGate SSL VPN
- Implement VLANs
- Configure FortiSwitch
- Add FortiAnalyzer
- Deploy a Certificate Authority (AD CS)