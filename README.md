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
|**Building B** (administrative)|Management, HR, Finance, Sales, kitchen (IoT)|
|**Building C**|Quality, Guest (visitors)|
|**Halls D and E** _(optional)_|Production, Logistics, OT network, connected via VPN|
|**Admin** _(optional)_|The integrator's NOC, managing the network remotely via VPN|
|**ISP**|Simulated Internet service provider|

## Architecture

- **Hierarchical campus:** Building A hosts a redundant _collapsed core_ (two L3 switches); Buildings B and C each have an L3 distribution switch connected to both core switches over routed links.
- **Each department has its own VLAN and subnet**, and inter-department traffic is controlled with ACLs.
- **The edge router (R-EDGE)** handles Internet access, NAT/PAT, static NAT for the DMZ and perimeter filtering.
- **Remote sites** (halls, Admin) connect through the ISP using VPN tunnels.

## Current topology

![Current topology](topologieCurenta.png)

### Devices

| Role                                                    | Platform                 | Count |
| ------------------------------------------------------- | ------------------------ | ----- |
| Routers (R-EDGE, ISP)                                   | Cisco IOSv 15.9(3)M6     | 2     |
| Switches (core, distribution, access)                   | Cisco IOSvL2 15.2        | 10    |
| Servers (SRV-DNS, SRV-LOG, SRV-MGMT, SRV-FILE, WEB-DMZ) | Docker `debian:bookworm` | 5     |
| Full Linux hosts (ADMIN-IT, CalculatorPrezentare)       | Debian 12.6 (QEMU)       | 2     |
| End hosts (PCs, laptops, printers, IoT, test bench)     | VPCS                     | 22    |
| Internet access                                         | GNS3 NAT node            | 1     |

## Port mapping

Generated from the GNS3 project file. Each switch uplinks to **both** core switches for redundancy.

### Internet edge

| Device | Port          | Connected to              | Remote port |
| ------ | ------------- | ------------------------- | ----------- |
| ISP    | Gi0/0         | R-EDGE                    | Gi0/0       |
| ISP    | Gi0/1         | NAT1 (real Internet)      | nat0        |
| ISP    | Gi0/2         | INTERNET-USER             | e0          |
| ISP    | Gi0/3 – Gi0/7 | _free (Halls D/E, Admin)_ |             |

### Building A

#### Edge and core

| Device   | Port  | Connected to | Remote port            |
| -------- | ----- | ------------ | ---------------------- |
| R-EDGE   | Gi0/0 | ISP          | Gi0/0                  |
| R-EDGE   | Gi0/1 | SW-CORE1     | Gi0/0                  |
| R-EDGE   | Gi0/2 | SW-CORE2     | Gi0/0                  |
| R-EDGE   | Gi0/3 | WEB-DMZ      | eth0                   |
| SW-CORE1 | Gi0/0 | R-EDGE       | Gi0/1                  |
| SW-CORE1 | Gi0/1 | SW-CORE2     | Gi0/1 _(EtherChannel)_ |
| SW-CORE1 | Gi0/2 | SW-CORE2     | Gi0/2 _(EtherChannel)_ |
| SW-CORE1 | Gi0/3 | SW-MAIN-B    | Gi0/0                  |
| SW-CORE1 | Gi1/0 | SW-MAIN-C    | Gi0/0                  |
| SW-CORE1 | Gi1/1 | SW-TEHNIC    | Gi0/0                  |
| SW-CORE1 | Gi1/2 | SW-SERVER    | Gi0/0                  |
| SW-CORE2 | Gi0/0 | R-EDGE       | Gi0/2                  |
| SW-CORE2 | Gi0/1 | SW-CORE1     | Gi0/1 _(EtherChannel)_ |
| SW-CORE2 | Gi0/2 | SW-CORE1     | Gi0/2 _(EtherChannel)_ |
| SW-CORE2 | Gi0/3 | SW-MAIN-B    | Gi0/3                  |
| SW-CORE2 | Gi1/0 | SW-MAIN-C    | Gi0/3                  |
| SW-CORE2 | Gi1/1 | SW-TEHNIC    | Gi0/1                  |
| SW-CORE2 | Gi1/2 | SW-SERVER    | Gi0/1                  |

#### Servers (SW-SERVER)

|Port|Connected to|Remote port|
|---|---|---|
|Gi0/0|SW-CORE1|Gi1/2|
|Gi0/1|SW-CORE2|Gi1/2|
|Gi1/0|SRV-DNS|eth0|
|Gi1/1|SRV-LOG|eth0|
|Gi1/2|SRV-MGMT|eth0|
|Gi1/3|SRV-FILE|eth0|

#### IT and R&D (SW-TEHNIC)

|Port|Connected to|Remote port|Zone|
|---|---|---|---|
|Gi0/0|SW-CORE1|Gi1/1|uplink|
|Gi0/1|SW-CORE2|Gi1/1|uplink|
|Gi1/0|PC1-IT|e0|IT|
|Gi1/1|ADMIN-IT|ens4|IT|
|Gi2/0|PC1-RD|e0|R&D|
|Gi2/1|PC2-RD|e0|R&D|
|Gi2/2|BANC-TEST|e0|R&D|

#### DMZ

|Device|Port|Connected to|Remote port|
|---|---|---|---|
|WEB-DMZ|eth0|R-EDGE|Gi0/3|

### Building B

#### Distribution (SW-MAIN-B)

|Port|Connected to|Remote port|
|---|---|---|
|Gi0/0|SW-CORE1|Gi0/3|
|Gi0/1|SW-CONDUCERE|Gi0/0|
|Gi0/2|SW-BIROU|Gi0/0|
|Gi0/3|SW-CORE2|Gi0/3|

#### Management (SW-CONDUCERE)

|Port|Connected to|Remote port|
|---|---|---|
|Gi0/0|SW-MAIN-B|Gi0/1|
|Gi0/1|PC-Director|e0|
|Gi0/2|PC-asistent/secretar|e0|
|Gi0/3|Imprimanta|e0|

#### Offices (SW-BIROU)

|Port|Connected to|Remote port|Zone|
|---|---|---|---|
|Gi0/0|SW-MAIN-B|Gi0/2|uplink|
|Gi0/1|ESPRESSOR|e0|Kitchen (IoT)|
|Gi0/2|TV|e0|Kitchen (IoT)|
|Gi0/3|PC1-HR|e0|HR|
|Gi1/0|PC2-HR|e0|HR|
|Gi1/1|PC1-FIN|e0|Finance|
|Gi1/2|PC2-FIN|e0|Finance|
|Gi1/3|LAPTOP-VNZ|e0|Sales|
|Gi2/0|PC1-VNZ|e0|Sales|

### Building C

#### Distribution (SW-MAIN-C)

|Port|Connected to|Remote port|
|---|---|---|
|Gi0/0|SW-CORE1|Gi1/0|
|Gi0/1|SW-CALITATE|Gi0/0|
|Gi0/2|SW-GUEST|Gi0/0|
|Gi0/3|SW-CORE2|Gi1/0|

#### Quality (SW-CALITATE)

|Port|Connected to|Remote port|
|---|---|---|
|Gi0/0|SW-MAIN-C|Gi0/1|
|Gi0/1|PC1-CAL|e0|
|Gi0/2|PC2-CAL|e0|
|Gi0/3|PC3-CAL|e0|
|Gi1/0|PRINTER-CAL|e0|

#### Guest (SW-GUEST)

|Port|Connected to|Remote port|
|---|---|---|
|Gi0/0|SW-MAIN-C|Gi0/2|
|Gi0/1|CalculatorPrezentare|ens4|
|Gi0/2|LAPTOP-G1|e0|
|Gi0/3|LAPTOP-G2|e0|
|Gi1/0|LAPTOP-G3|e0|

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
- **VPCS** for end-user hosts, **Debian 12** for servers and testing (Docker and QEMU)
- **Python** for automation

## Status

- [x] Project proposal approved
- [x] Overall architecture defined
- [x] Buildings A, B and C placed and cabled in GNS3
- [x] Internet edge (R-EDGE, ISP, NAT) and DMZ placed
- [ ] Addressing plan and VLANs
- [ ] Staged configuration (ITN → SRWE → ENSA)
- [ ] Python application
- [ ] Optional extensions (Halls D/E, Admin, VPN) and IPv6
- [ ] Thesis write-up