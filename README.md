Enterprise Multi-Site 3-Tier Network

This is a Cisco Packet Tracer lab (`Enterprise MultiSite 3Tier Network.pkt`) I built to practice designing and configuring a full three-tier enterprise campus network. It covers Core, Distribution, and Access layers along with dual-ISP failover, Layer 3 EtherChannels, OSPF, eBGP, HSRP, and Layer 2 security.

Network Layout

### Edge and WAN (Edge-Router-1, ISP-1, ISP-2, Router1)
* Configured eBGP peering with ISP-1 and ISP-2 over `/30` transit links (`203.0.113.0/30`), alongside internal OSPF routing for the campus.
* Set up floating static routes for ISP failover, keeping ISP-1 as the primary gateway of last resort.
* Configured PAT on Edge-Router-1 for outbound internet access.
* Added a Loopback interface (`1.1.1.1`) for router identification and stability.

### Core Layer (Core-SW1, Core-SW2)
* Configured strictly as Layer 3 routed switches with dedicated Loopback interfaces (`10.10.10.10` and `20.20.20.20`).
* Bundled inter-switch and downlink connections using both PAgP and LACP EtherChannels (`Po1` and `Po2`).
* Running OSPF across all core point-to-point links with Edge-Router-1 set as the gateway of last resort.

### Distribution Layer (DSW-1, DSW-2)
* Divided hosts by device type and placed management traffic on its own separate VLAN (`VLAN 99`), with SVIs configured for each VLAN.
* Configured per-VLAN HSRP with preemption enabled, using the last usable IP address in each subnet as the virtual IP.
* Aligned Spanning Tree so whichever distribution switch is the active HSRP router for a VLAN is also its STP Root Bridge.
* Set VTP mode to transparent, disabled DTP on trunks, and assigned an unused native VLAN.

### Access Layer (ASW-1, ASW-2, ASW-3)
* Enabled PortFast and BPDU Guard on all access ports.
* Configured Port Security using sticky MAC learning and `restrict` violation mode.
* Enabled DHCP Snooping and Dynamic ARP Inspection (DAI), setting `g0/1` and `g0/2` as trusted uplink ports.
* Configured trunks on `g0/1` and `g0/2` allowing only the required VLANs, with DTP disabled and an unused native VLAN.

### Infrastructure Services (DNS/DHCP/AAA, Syslog/NTP)
* Centralized DNS, DHCP, and AAA authentication (with local credentials configured as fallback) under the `officeA.com` domain.
* Configured Syslog and authenticated NTP across all routers and switches so logs and timestamps stay synced.
* Enabled SSH across network devices for secure remote management.

IP Addressing Table

| Device | Interface / VLAN | IP Address / Subnet | Notes |
| :--- | :--- | :--- | :--- |
| Edge-Router-1 | g0/0 | 192.168.0.1/30 | Link to Core-SW1 |
| | g0/1 | 192.168.0.5/30 | Link to Core-SW2 |
| | s0/2/0 | 203.0.113.1/30 | Link to ISP-1 |
| | Loopback0 | 1.1.1.1 | Router ID |
| Core-SW1 | Po2 | 192.168.0.9/30 | Link to Core-SW2 |
| | Po1 | 192.168.0.13/30 | Link to DSW-1 |
| | g1/0/4 | 192.168.0.21/30 | Link to DSW-2 |
| | g1/0/5 | 192.168.0.2/30 | Link to Edge-Router-1 |
| | Loopback0 | 10.10.10.10 | Router ID |
| Core-SW2 | Po2 | 192.168.0.10/30 | Link to Core-SW1 |
| | Po1 | 192.168.0.17/30 | Link to DSW-2 |
| | g1/0/4 | 192.168.0.25/30 | Link to DSW-1 |
| | g1/0/5 | 192.168.0.6/30 | Link to Edge-Router-1 |
| | Loopback0 | 20.20.20.20 | Router ID |
| DSW-1 | g1/0/1 | 192.168.0.26/30 | Link to Core-SW2 |
| | Po1 | 192.168.0.14/30 | Link to Core-SW1 |
| | Po2 | 192.168.0.29/30 | Link to DSW-2 |
| | VLAN 10 | 192.168.10.253/24 | HSRP VIP: 192.168.10.254 |
| | VLAN 20 | 192.168.20.253/24 | HSRP VIP: 192.168.20.254 |
| | VLAN 30 | 192.168.30.252/24 | HSRP VIP: 192.168.30.254 |
| | VLAN 40 | 192.168.40.4/29 | HSRP VIP: 192.168.40.6 |
| | VLAN 99 | 192.168.99.4/28 | Management (HSRP VIP: 192.168.99.14) |
| DSW-2 | g1/0/1 | 192.168.0.22/30 | Link to Core-SW1 |
| | Po1 | 192.168.0.18/30 | Link to Core-SW2 |
| | Po2 | 192.168.0.30/30 | Link to DSW-1 |
| | VLAN 10 | 192.168.10.252/24 | HSRP VIP: 192.168.10.254 |
| | VLAN 20 | 192.168.20.252/24 | HSRP VIP: 192.168.20.254 |
| | VLAN 30 | 192.168.30.253/24 | HSRP VIP: 192.168.30.254 |
| | VLAN 40 | 192.168.40.5/29 | HSRP VIP: 192.168.40.6 |
| | VLAN 99 | 192.168.99.5/28 | Management (HSRP VIP: 192.168.99.14) |
