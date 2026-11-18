# Architecture

[← Back to main README](../README.md)

## Design objective

Document a single-site two-tier enterprise campus with redundant switching, first-hop gateways, routed edge links, dual-ISP BGP, and Internet NAT/PAT. The SRv6 multi-site backbone is a separate project.

## Functional layers

| Layer | Devices | Responsibilities |
| --- | --- | --- |
| Clients | PC-01, PC-02 | DHCP clients, routing and failover testing |
| Access | ACC-01, ACC-02 | VLAN assignment, 802.1Q trunks, access-port protections |
| Distribution | DIST-01, DIST-02 | SVIs, HSRP, OSPF, LACP Port-channel, routed uplinks |
| Edge | EDGE-01, EDGE-02 | OSPF, eBGP, iBGP, NAT/PAT; DHCP pools on EDGE-01 |
| Simulated providers | ISP-1, ISP-2 | eBGP to both edges, inter-ISP peering, GNS3 Internet connection |

## Why the links are arranged this way

Both access switches connect to both distribution switches. This gives an alternative Layer 2 uplink when one is unavailable, with Spanning Tree preventing forwarding loops. HSRP supplies stable virtual IP gateways for VLANs 10, 20 and 90.

Three physical links between the distribution pair join LACP Port-channel1. The port-channel must be verified with operational command output before claiming resiliency measurements.

Each distribution switch connects to both edge routers using routed IPv4 /30 links. This is a Layer 3 boundary managed by OSPF, rather than Spanning Tree.

Each enterprise edge also connects to both simulated ISPs via eBGP, providing multiple external paths. ISP-1 and ISP-2 peer directly over `203.0.113.0/30`.

## Packet flow

1. The client in VLAN 10 uses the HSRP VIP `10.10.10.254` as its default gateway.
2. The active distribution switch routes the packet to an enterprise edge based on its routing table.
3. The edge selects an ISP-facing next hop using its installed route and BGP policy.
4. For ISP-1, route-map-based PAT translates the client source to the outgoing edge interface; ISP-1 performs PAT to the GNS3 Internet interface.
5. Return traffic requires corresponding routes and translation state. An alternative BGP path without NAT is not sufficient for working Internet failover.

The first hop in a traceroute can show the physical SVI IP (`10.10.10.1`) even when clients are configured with HSRP gateway `.254`.

## Boundaries and verification

- Campus VLAN 90 is the management subnet; VLAN 999 is native/unused.
- OSPF operates on the routed campus links; the enterprise BGP ASN is 65000.
- EDGE-01 provides DHCP for VLANs 10, 20 and 90 in the consolidated reference.
- **ISP-1 NAT/PAT** was configured in the source lab, but **ISP-2 NAT/PAT and full failover remain unverified**.
- These configs align two prior lab snapshots into a consistent reference and are **not a current verified running configuration**.

See [BGP and NAT detail](wan-bgp-nat.md) and [IP addressing](ip-addressing.md).
