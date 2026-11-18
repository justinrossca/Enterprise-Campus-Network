# Enterprise Campus Network

**Two-tier campus networking, redundant edge routing, dual-ISP BGP, and NAT/PAT in GNS3**

A Cisco networking lab I built to practice designing, configuring, and troubleshooting a small enterprise campus.

> **Lab status:** Consolidated single-site configuration references for campus switching, OSPF, dual-multihomed BGP and ISP-1 NAT/PAT.

## Overview

The two-tier campus uses Layer 2 access switches and Layer 3 distribution switches. Distribution provides HSRP gateways, a three-link LACP Port-channel and four routed OSPF links to the enterprise edges. EDGE-01 retains the campus DHCP pools for VLANs 10, 20 and 90.

Both edge routers connect to two simulated ISPs through four eBGP sessions within Enterprise AS 65000. The network also implements iBGP, BGP local preference to prioritize ISP-1, and NAT/PAT for Internet connectivity.

## Network topology

![Single-site enterprise campus GNS3 topology](topology/campus-topology.png)

A two-tier campus design with redundant uplinks, a three-link LACP EtherChannel, dual-multihomed ISP connections, and an inter-ISP peering link.

## What this network is designed to do

| Capability | Implementation | Verification status |
| --- | --- | --- |
| VLAN segmentation | VLAN 10 (Users), VLAN 20 (Staff), VLAN 90 (Management) | Addressing documented; live VLAN output pending |
| First-hop redundancy | HSRP v2 with distribution switches alternating active roles | Configured in reference; live HSRP failover test pending |
| Link aggregation | Three-link LACP EtherChannel between distribution switches | Configured in reference; member-state output pending |
| Routing | OSPF area 0 between distribution and edge | Configured in reference; final neighbor capture pending |
| Spanning Tree | Rapid PVST+ planned/implemented in later lab iteration | Live mode/root/port-state evidence pending |
| DHCP | DHCP pools on EDGE-01, relay on distribution SVIs | Configuration documented; lease capture pending |
| Access security | DHCP snooping, DAI, sticky MAC port security | Configuration documented; binding/violation checks pending |
| Earlier direct NAT test | EDGE-02 direct GNS3 NAT baseline | Historic NAT counters captured; not verification of the consolidated BGP design |
| Dual-ISP eBGP | Two enterprise edges peer with both ISP-1 (AS 65001) and ISP-2 (AS 65002) | Four peerings configured in the merged design; current neighbor states need capture |
| iBGP and routing policy | Edge loopback peering, `next-hop-self`, ISP-1 default preference 200, `10.10.0.0/16` BGP aggregate | Configured in reference; actual neighbor/policy outcomes pending validation |
| ISP-1 PAT | Enterprise-edge PAT to ISP-1 plus ISP-1 PAT to GNS3 Internet | Both stages configured in source snapshots; earlier lab reachability reported |
| ISP-2 failover | Second provider path with inter-ISP eBGP adjacency | ISP-2 PAT not configured in the supplied snapshot; Internet failover not confirmed |

## Architecture

### Access layer

- `ACC-01` carries the PC in VLAN 10; `ACC-02` carries the PC in VLAN 20.
- Each access switch connects to **both** distribution switches by independent 802.1Q trunks.
- VLAN 90 provides a separate IP subnet for switch management.
- PC-facing ports use port security; DHCP snooping and DAI are configured to protect client traffic.

### Distribution layer

- `DIST-01` and `DIST-02` perform inter-VLAN routing using SVIs.
- HSRP supplies one stable virtual gateway per VLAN, even when the preferred distribution switch changes.
- A three-member LACP Port-channel provides an inter-distribution Layer 2 connection.
- Each distribution switch has a routed OSPF connection to **each** edge router.

### Edge and Internet/WAN

- `EDGE-01` provides the campus DHCP pools and also connects to both simulated ISPs through eBGP.
- `EDGE-02` provides a second routed Internet edge with both ISP connections.
- The two enterprise edges have iBGP configured between `Loopback1` interfaces; OSPF carries the campus and loopback routes.
- ISP-1 (AS 65001) and ISP-2 (AS 65002) peer over `203.0.113.0/30`, and each has a GNS3 NAT-facing interface.
- Two-stage ISP-1 NAT/PAT is configured. ISP-2 NAT and tested end-to-end Internet failover are still outstanding.

[Full dual-ISP BGP and NAT design](docs/wan-bgp-nat.md).

See [Architecture and design decisions](docs/architecture.md) for more detail.

## VLAN and IP addressing

| VLAN | Name | IPv4 network | HSRP virtual IP | Preferred active switch |
| ---: | --- | --- | --- | --- |
| 10 | USERS | `10.10.10.0/24` | `10.10.10.254` | `DIST-01` |
| 20 | STAFF | `10.10.20.0/24` | `10.10.20.254` | `DIST-02` |
| 90 | MANAGEMENT | `10.10.90.0/24` | `10.10.90.254` | `DIST-01` |
| 999 | BLACKHOLE / native | No routed SVI | None | Not applicable |

Distribution SVI addresses use `.1` on `DIST-01` and `.2` on `DIST-02` for VLANs 10, 20, and 90. Access-switch management addresses are planned as `10.10.90.11` and `10.10.90.12`.

See [Full addressing plan](docs/ip-addressing.md) for physical routed links, management addresses, and gateway choices.

The WAN design uses four ISP /30s and an ISP-to-ISP link (`203.0.113.0/30`). [View the WAN addressing table](docs/wan-bgp-nat.md#wan-addressing-and-bgp-neighbors).

## How redundancy works

**HSRP:** VLANs 10 and 90 prefer `DIST-01`; VLAN 20 prefers `DIST-02`. End devices keep `.254` as their default gateway, even if the active router changes.

**LACP:** The distribution-to-distribution bundle acts as one logical Port-channel. Losing one member should leave the remaining bundle operational, subject to the live platform configuration.

**Rapid PVST+:** Both access switches have redundant trunks, so STP prevents Layer 2 loops. The intended per-VLAN root placement aligns with HSRP active roles. This still needs documented `show spanning-tree` evidence.

**OSPF:** Both distribution switches have links to both edge routers. OSPF should provide alternative paths when a routed connection fails; convergence behavior has not yet been measured.

**Dual-ISP BGP:** Each enterprise edge has eBGP configured toward both ISPs; the ISPs also have an eBGP interconnection. ISP-1 preference is configured with a route-map, but failover of client Internet traffic through ISP-2 is **not** verified. BGP sessions and NAT paths need independent testing.

[Redundancy mechanisms and test plan](docs/redundancy.md)

## Security design

The baseline lab uses VLAN separation, DHCP snooping, Dynamic ARP Inspection, sticky MAC learning, switch management over VLAN 90, and SSH. VLAN separation **does not automatically block routed communication** between VLANs; inter-VLAN ACL policy is not claimed as implemented in these files.

[Access-layer security notes](docs/security.md)

## Verification and troubleshooting

| Evidence or configuration | What it demonstrates | Limitation |
| --- | --- | --- |
| PC traceroute reaches `10.0.0.14` in two hops | Baseline campus IPv4 routing to an edge interface | Not an HSRP failover test |
| EDGE-02 NAT counters show hits and expired translations | Earlier **direct-NAT** experiment processed matching packets | Not proof of live Internet access over the BGP/ISP paths |
| Enterprise edge configuration contains four eBGP neighbors and loopback iBGP | BGP peers and policies are defined | Does not prove the sessions are Established |
| Enterprise edges and ISP-1 contain PAT rules | ISP-1 NAT path is defined | No completed ISP-2 NAT failover test |

## Configuration references

The published configuration files are **sanitized, consolidated reference configurations**, not verified live running-config exports. There is **one configuration per device**, with BGP and NAT incorporated directly into each edge file. Campus VLAN 90 and unique OSPF router IDs are kept consistent.

| Access switches | Distribution switches | Edge routers |
| --- | --- | --- |
| [ACC-01](configs/access/ACC-01.cfg) | [DIST-01](configs/distribution/DIST-01.cfg) | [EDGE-01](configs/edge/EDGE-01.cfg) |
| [ACC-02](configs/access/ACC-02.cfg) | [DIST-02](configs/distribution/DIST-02.cfg) | [EDGE-02](configs/edge/EDGE-02.cfg) |

**Simulated ISP configurations:** [ISP-1](configs/wan/ISP-1.cfg) · [ISP-2](configs/wan/ISP-2.cfg)

## Design trade-offs

- **Dual-homed Layer 2 access:** Keeps access switching straightforward but allows STP to place alternate uplinks into discarding state on some VLANs.
- **HSRP with asymmetric active roles:** Distributes gateway ownership across the distribution pair but requires consistent STP root planning.
- **Dual-multihomed WAN:** Provides four possible enterprise-to-ISP eBGP adjacencies and an inter-ISP peer link, but additional NAT and return-path work is required for validated backup Internet service.
- **Consolidated single-site design:** The edge files integrate the BGP/NAT settings with campus VLAN 90 and consistent interface assignments, but still need a live verification pass.
- **Separate SRv6 lab:** Keeps campus troubleshooting focused rather than introducing an unfinished WAN transport into every campus test.

More detail: [Design decisions](docs/design-decisions.md).

## Future Development & Learning Goals

This network is an ongoing learning project. As I continue developing my networking skills, I plan to expand the topology and implement additional technologies to better understand enterprise network design, security, and troubleshooting.

### Planned Implementations

- **OSPFv3 & IPv6:** Introduce IPv6 addressing and OSPFv3 routing alongside the existing IPv4 infrastructure.
- **SRv6 Backbone (separate repository):** Develop Segment Routing over IPv6 services for a separate multi-site lab.
- **Rapid PVST+ Optimization:** Fine-tune bridge priorities, root placement, and path selection to improve Layer 2 redundancy and convergence.
- **OSPF Authentication:** Implement routing protocol authentication to protect OSPF adjacencies from unauthorized participation.
- **Multi-Area OSPF:** Explore hierarchical OSPF designs, area border routers, and route summarization if future network expansion justifies multiple areas.
- **HSRP Interface Tracking:** Implement object tracking to improve gateway failover when upstream connections become unavailable.
- **Network Monitoring:** Explore centralized Syslog, NTP, and SNMPv3 for device monitoring and troubleshooting.
- **Dual-ISP Failover:** Configure and validate ISP-2 NAT/PAT, route policy, and client recovery after ISP-1 failures.
- **BGP Policy Hardening:** Validate the corrected `PREFER-ISP01` route-map on EDGE-01, audit outbound prefix controls, and test local preference and iBGP best-path selection.
- **Redundancy & Failover Testing:** Conduct controlled link and device failure tests to examine STP, HSRP, OSPF, and BGP convergence.

### Long-Term Goal

Continue refining this single-site enterprise campus while documenting new implementations, troubleshooting experiences, and lessons learned. Multi-site SRv6 networking will be developed separately.

## Repository contents

```text
Enterprise-Campus-Network/
├── README.md
├── topology/
│   └── campus-topology.png
├── configs/
│   ├── access/                  # campus access references
│   ├── distribution/            # campus distribution references
│   ├── edge/                    # campus edge + BGP/NAT references
│   └── wan/                     # ISP-1 and ISP-2 reference configurations
├── docs/
│   ├── architecture.md
│   ├── ip-addressing.md
│   ├── redundancy.md
│   ├── security.md
│   ├── design-decisions.md
│   └── wan-bgp-nat.md
└── notes/
    ├── troubleshooting-log.md
    └── lessons-learned.md
```

## Lessons from the lab

Having built a similar network in the past, I was already familiar with many of the technologies involved, so I encountered relatively few issues during implementation. This project helped reinforce my understanding of network design and troubleshooting, particularly interpreting what different verification commands actually demonstrate. For example, a traceroute displaying a physical SVI address does not contradict an HSRP virtual gateway, and increasing NAT counters do not necessarily confirm successful Internet connectivity. I plan to continue updating this repository as I expand the lab and conduct further redundancy and failover testing.

[More troubleshooting notes](notes/lessons-learned.md)
