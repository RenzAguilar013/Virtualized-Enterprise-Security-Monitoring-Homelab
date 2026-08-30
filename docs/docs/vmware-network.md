
# VMware Network Configuration

## VMnet8 NAT Network

![VMnet8 NAT configuration](../screenshots/vmware/vmnet8-nat-overview.png)

VMnet8 provides upstream connectivity to the FortiGate WAN interface.

- Network type: NAT
- Subnet: `192.168.100.0/24`
- Gateway: `192.168.100.2`
- Connected interface: FortiGate `port1`

## VMnet2 Internal LAN

![VMnet2 host-only configuration](../screenshots/vmware/vmnet2-lan-overview.png)

VMnet2 provides the isolated internal network for the homelab.

- Network type: Host-only
- Subnet: `10.10.10.0/24`
- Gateway: `10.10.10.1`
- VMware DHCP: Disabled
- DHCP server: FortiGate
