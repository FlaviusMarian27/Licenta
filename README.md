# Design, Implementation and Automation of an Enterprise Network in GNS3

> Bachelor's thesis project (working title) · Marian Flavius-Andrei

## About

This project builds the complete network of a medium-sized company in **GNS3**, treated as a **product delivered to a client**: design, implementation, hardening and administration. Network configuration and administration are automated with a **Python** application.

The topology is designed from scratch. It started from the network built during a summer internship, but the design here is far more complex: multiple buildings, redundancy, per-department segmentation and management services.

## Scenario

The company (fictional, name TBD) is an **automotive supplier**, with a headquarters made of several buildings on the same site and, optionally, production halls at remote locations.

|Zone|Contents|
|---|---|
|**Building A** (technical HQ)|Network core, servers, DMZ, Internet edge, IT, R&D|
|**Building B** (administrative)|Management, HR, Finance, Purchasing/Sales, kitchen (IoT)|
|**Building C**|Quality, Guest (visitors)|
|**Halls D and E** _(optional)_|Production, Logistics, OT network, connected via VPN|
|**Admin** _(optional)_|The integrator's NOC, managing the network remotely via VPN|
|**ISP**|Simulated Internet service provider|

## Architecture

- **Hierarchical campus:** Building A hosts a redundant _collapsed core_ (two L3 switches); Buildings B and C each have an L3 distribution switch connected to the core over routed links.
- **Each department has its own VLAN and subnet**, and inter-department traffic is controlled with ACLs.
- **The edge router (R-EDGE)** handles Internet access, NAT/PAT, static NAT for the DMZ and perimeter filtering.
- **Remote sites** (halls, Admin) connect through the ISP using VPN tunnels.

## Scope (CCNA modules 1–3)

|Stage|Contents|
|---|---|
|**ITN**|IPv4 addressing plan with VLSM, basic configuration and hardening (SSH), static and default routes, DHCPv4 (server and relay)|
|**SRWE**|VLANs and 802.1Q trunks, inter-VLAN routing (SVIs), Rapid PVST+, PortFast, BPDU guard, EtherChannel (LACP), HSRP, port security, DHCP snooping|
|**ENSA**|OSPF (possibly multi-area), standard/extended/named ACLs, NAT/PAT and static NAT, NTP, syslog, SNMP, CDP/LLDP|
|**Optional**|Site-to-site GRE/IPsec VPN, dedicated DMZ firewall, BGP towards the ISP, QoS (theoretical)|
|**Extension**|IPv6 dual-stack and an analysis of IPv6-specific security risks|

Wireless is out of scope, since GNS3 does not simulate it well.

## Python automation

An application that:

- **generates** device configurations from templates;
- **pushes** configurations to devices over SSH;
- **backs up** configurations periodically;
- **verifies** network state (interfaces, OSPF neighbors, connectivity).

Details will be defined once the topology is finalized.

## Tech stack

- **GNS3** for simulation
- **Cisco IOSvL2** (L2/L3 switches) and **IOSv** (routers); IOL for the final, lighter build
- **VPCS** for end-user hosts, **Debian 12** for servers and testing
- **Python** for automation

## Status

- [x] Project proposal approved
- [x] Overall architecture defined
- [x] Buildings B and C placed in GNS3
- [ ] Building A (core, servers, DMZ, R-EDGE)
- [ ] Addressing plan and VLANs
- [ ] Staged configuration (ITN → SRWE → ENSA)
- [ ] Python application
- [ ] Optional extensions and IPv6
- [ ] Thesis write-up