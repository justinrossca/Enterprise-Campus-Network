# Design decisions and trade-offs

[← Back to main README](../README.md)

## Why dual-home the access switches?

A single distribution switch failure should not automatically isolate every endpoint. Each access switch has two independent trunks. The trade-off is that Layer 2 loop prevention matters; not every physical link forwards for every VLAN at all times.

## Why use HSRP instead of static gateway addresses?

A fixed HSRP VIP decouples the client configuration from whichever distribution switch is forwarding. Splitting active gateway roles between VLANs gives practice with load sharing and aligned STP root selection. It does not provide arbitrary traffic load balancing within a single VLAN.

## Why put the EtherChannel between distribution switches?

It provides a logical Layer 2 connection between the SVI/HSRP peers and an opportunity to learn LACP negotiation and member failure behavior. Access-to-distribution connections stay separate trunks because the two distribution devices are independent switches, not a configured multi-chassis EtherChannel system.

## Why use /30 routed links to the edge?

Point-to-point routed links keep the broadcast domains from extending into the edge routing layer. OSPF can select alternative paths independently of Spanning Tree.

## Why extend the campus with two simulated ISPs?

Dual-multihoming gives practice with four enterprise-to-ISP eBGP sessions, loopback iBGP, local preference, NAT/PAT, and return-path troubleshooting. The two ISPs also peer with each other. It is important to distinguish BGP reachability from working public Internet access: the supplied configuration has PAT through ISP-1, but ISP-2 NAT and a complete failover demonstration are not present.

The previous direct-GNS3-NAT test is kept only as historical validation evidence. Edge configurations have now been consolidated into one reference file per device, using VLAN 90 and the dual-ISP interface layout. This integrated reference still requires live verification.

Private SRv6 site-to-site transport stays in its own project, separate from Internet NAT.

## What would make the design stronger?

1. HSRP tracking of upstream reachability so an isolated distribution switch relinquishes active gateway roles.
2. Measured failover tests instead of assumed convergence behavior.
3. Verified Rapid PVST+ root/port placement and access protections.
4. A management access ACL for infrastructure SSH.
5. BGP prefix/route-map audit, including the corrected permit clause on EDGE-01.
6. Two-sided ISP-2 NAT/PAT and controlled client failover test.
7. Clear separation between the consolidated single-site campus and separate SRv6 work.
