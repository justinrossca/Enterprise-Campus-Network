# Dual-ISP WAN verification log

All results are **pending fresh CLI evidence**. Do not interpret configuration presence as proof of protocol state.

| Test | Command(s) | Result / evidence |
| --- | --- | --- |
| Four enterprise eBGP peers | `show ip bgp summary` on each edge | Pending |
| Edge-to-edge iBGP | `show ip bgp summary`, `show ip route 1.1.1.1`, `show ip route 2.2.2.2` | Pending |
| OSPF adjacency / unique router IDs | `show ip ospf neighbor` | Pending |
| ISP-1 local preference | `show ip bgp 0.0.0.0` | Pending |
| Summary route advertisement | `show ip bgp neighbors <peer> advertised-routes` | Pending |
| PAT via ISP-1 | Client reachability + `show ip nat translations` on edge and ISP-1 | Pending |
| ISP-1 withdrawal / failure | Shut selected links, collect routes and client tests | Pending |
| ISP-2 PAT/failover | Configure and validate both translation stages | Not configured in the supplied reference |


**Evidence note:** The `EDGE-02` NAT screenshot documents the earlier direct GNS3 NAT experiment, **not** this site's dual-ISP BGP/NAT path. Capture fresh screenshots from the consolidated GNS3 design before marking tests complete.
