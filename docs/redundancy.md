# Redundancy mechanisms

[← Back to main README](../README.md)

## HSRP: first-hop redundancy

Distribution SVIs share a virtual IP for each client/management subnet. The design uses priorities `110` and `90` and preemption to prefer different active switches per VLAN.

| VLAN | Preferred active | Backup | Virtual IP |
| --- | --- | --- | --- |
| 10 | DIST-01 | DIST-02 | `10.10.10.254` |
| 20 | DIST-02 | DIST-01 | `10.10.20.254` |
| 90 | DIST-01 | DIST-02 | `10.10.90.254` |

Inspect using `show standby brief` on both distribution switches. Shut down the preferred distribution SVI in a controlled test, then confirm that the peer becomes Active while the PC's default gateway remains unchanged.

## LACP inter-distribution bundle

The baseline uses Port-channel1 with three member links: `Gi1/0`, `Gi0/2`, and `Gi0/3` on both distribution switches. Each side uses LACP active mode.

Check `show etherchannel summary` for the operational Port-channel and bundled members. Test one member at a time. Verify that the remaining links still forward across the Port-channel.

## Spanning Tree

The intended mode is Rapid PVST+, with root bridge placement aligned to HSRP: DIST-01 primary for VLANs 10/90 and DIST-02 primary for VLAN 20.

Because each access switch connects to both distribution switches, expect some redundant trunk ports to be in Alternate/Discarding state for a given VLAN. Do not treat this as a broken physical link.

**Validation pending:** Capture `show spanning-tree summary`, `show spanning-tree vlan 10`, and equivalent VLAN 20/90 outputs. The archived baseline scripts do not include the later STP additions.

## OSPF at the Layer 3 boundary

Both distribution switches have one routed link to each edge router. OSPF area 0 advertises these routed interfaces and campus VLAN SVIs. The SVIs are passive in the baseline to prevent adjacencies with hosts while advertising connected VLAN routes.

OSPF gives alternative routed paths but doesn't, by itself, guarantee Internet-session continuity if NAT state moves between edge routers.

## Recommended failover testing

| Scenario | Expected mechanism | Record |
| --- | --- | --- |
| One access uplink disconnected | Rapid PVST+ reconvergence | Port roles and client reachability |
| One LACP member disabled | Port-channel continues with fewer members | Member flags, traffic impact |
| One distribution SVI disabled | HSRP peer takes active role | Active state, lost pings |
| One routed link disabled | OSPF recomputes path | Neighbors, route next hops |
| One edge router disabled | Alternate routing may remain | Return path and NAT behavior |

## Dual-multihomed BGP redundancy

The consolidated edge references configure **two eBGP upstreams per enterprise edge router**, iBGP between enterprise edges, and an eBGP link between the two ISPs. The ISP-1 default route is assigned local preference 200 on the enterprise edges. Because no successful ISP-2 Internet test or NAT/PAT configuration was supplied, physical/BGP redundancy should not be described as proven end-to-end Internet failover.

Suggested evidence: `show ip bgp summary`, `show ip bgp 0.0.0.0`, `show ip route`, client traceroutes, and NAT translations captured before and after isolating ISP-1. See [WAN validation log](../validation-tests/wan-test-log.md).
