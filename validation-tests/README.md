# Validation evidence and test plan

[← Main README](../README.md)

## Original campus evidence

- [PC traceroute to an edge interface](screenshots/traceroute-to-edge.png): `10.0.0.14` reached in two hops from a campus host. It does not demonstrate HSRP failover.
- [EDGE-02 `show ip nat statistics`](screenshots/edge02-nat-counters.png): 9,810 hits and 114 expired translations in a previous direct GNS3 NAT baseline. Translation counters do not confirm successful Internet access.

These are original user-supplied console screenshots, not generated recreations or new validation of the HQ dual-ISP topology.

## HQ BGP / NAT validation

See [WAN test log](wan-test-log.md) for checks still requiring live captures. The provided `show running-config` outputs confirm configuration, not actual neighbor establishment or operational failover.

## Next command captures

- `show standby brief` on both distribution switches
- `show etherchannel summary` on both distribution switches
- `show spanning-tree vlan 10,20,90` on distribution and access switches
- `show ip ospf neighbor` at campus edge/distribution
- `show ip bgp summary` and `show ip bgp 0.0.0.0` on both HQ edges
- Client Internet reachability through ISP-1 and through ISP-2 after a controlled failure
- `show ip nat statistics` and `show ip nat translations` from devices handling Internet traffic
