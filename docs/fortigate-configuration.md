# FortiGate Configuration

## Dashboard

![FortiGate dashboard](../screenshots/Fortigate/fortigate-dashboard.png)

## Network Interfaces

![FortiGate interfaces](../screenshots/Fortigate/fortigate-interfaces.png)

- WAN interface: `port1`
- WAN address: `192.168.100.10/24`
- LAN interface: `port2`
- LAN address: `10.10.10.1/24`

## Default Route

![FortiGate static route](../screenshots/Fortigate/fortigate-static-route.png)

The default route sends outbound traffic through the VMware NAT gateway at
`192.168.100.2`.

## DHCP Server

![FortiGate DHCP settings](../screenshots/Fortigate/fortigate-dhcp-settings.png)

FortiGate provides DHCP services to the internal `10.10.10.0/24` network.

## DHCP Reservations

![FortiGate DHCP reservations](../screenshots/Fortigate/fortigate-dhcp-reservations.png)
