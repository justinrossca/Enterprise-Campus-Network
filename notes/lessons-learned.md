# Lessons learned

[← Back to main README](../README.md)

## 1. Configuration and verification are different

A syntactically accepted command isn't a functional test. Neighbors, routes, bindings, standby roles, port-channel members, and client traffic should be checked separately.

## 2. A virtual gateway does not need to appear in traceroute

For HSRP, the end host forwards to the `.254` virtual address, but the responding router can source traceroute replies from `.1` or `.2` depending on which distribution switch is active.

## 3. Routing redundancy and Layer 2 redundancy are independent

Rapid PVST+ handles redundant trunks inside the switched campus. OSPF handles alternate routed paths between distribution and edge. HSRP preserves a stable first-hop gateway for clients. None replaces the others.

## 4. NAT counters don't prove application reachability

Translation hits tell you matching traffic was processed. They don't confirm DNS, outbound routing, firewall policy, or reply traffic is correct. Troubleshooting should check the complete forward and return path.

## 5. A good lab record contains failures too

A troubleshooting journal with the symptom, a hypothesis, the exact command used, and the result is more informative than a list of successful protocol names. Future updates will add actual packet loss and convergence measurements once collected.

## 6. BGP control-plane redundancy is not Internet failover

Two ISPs and multiple eBGP paths improve routing options, but working failover still depends on default-route validity, NAT state, return reachability, and client tests. In this lab, ISP-1 has PAT configured while ISP-2 does not.

## 7. Merging references requires validation

The original BGP snapshot advertised management VLAN 99, whereas this site uses VLAN 90. The consolidated EDGE files now use VLAN 90 and retain the original BGP/NAT policies. These are documentation references, not proof that the merged configurations have been tested on running devices.
