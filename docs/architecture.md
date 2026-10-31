# Lab Architecture

## Network Topology

![Enterprise security homelab topology](../assets/topology.png)

This homelab uses VMware Workstation to simulate an enterprise network.
FortiGate connects the internal VMnet2 LAN to the VMnet8 NAT network and
provides firewalling, routing, NAT and DHCP services.

## VMware Virtual Machines

![VMware lab virtual machines](../screenshots/vmware/vmware-lab-vms.png)

The lab contains FortiGate, Windows Server, Windows 11, Wazuh, Splunk,
Ubuntu Linux and Kali Linux virtual machines.
