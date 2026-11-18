# Troubleshooting log

[← Back to main README](../README.md)

These are issues discussed and investigated during the build. They are explanations and observations, **not** claims of completed remediation unless indicated.

## 1. Traceroute showed `10.10.10.1`, not `10.10.10.254`

**Observation:** PC traceroute displayed `10.10.10.1` as the first hop, although VLAN 10 uses `10.10.10.254` as its HSRP gateway.

**Explanation:** The active distribution switch can source the ICMP Time Exceeded reply from its physical SVI address, while the client forwards toward the HSRP virtual IP/MAC.

**How to verify HSRP directly:** `show standby brief` on both distribution switches, plus the PC's default gateway from `ipconfig`.

## 2. NAT counters increased, but Internet ping was failing

**Observation:** EDGE-02 reported NAT hits and expired translations, yet a ping sourced from DIST-01 to `8.8.8.8` was failing.

**Likely contributing factor:** The NAT ACL matched `10.10.0.0/16` campus LANs, while a ping initiated by a distribution switch might use its `10.0.0.x` routed interface as the source. That traffic may not match the NAT rule.

**Next diagnostic test:** `ping 8.8.8.8 source Vlan10`, then examine the actual egress edge and NAT translations. Confirm upstream default route and return traffic separately.

**Evidence status:** No confirmed successful public Internet ping is included here.

## 3. DHCP snooping and Dynamic ARP Inspection interaction

**Lesson:** DAI on untrusted PC ports relies on valid DHCP snooping bindings for DHCP clients. An empty binding table can cause legitimate ARP traffic to be rejected.

**Check:** Obtain a fresh DHCP lease, then inspect `show ip dhcp snooping binding` and DAI statistics.

## 4. EtherChannel and command placement

**Lesson:** The protocol keyword is `lacp`; all intended member links should use compatible channel-group and trunk settings. Global DHCP snooping/DAI commands must be entered from global configuration mode rather than under an interface.

## Open verification items

- Final live running-config capture with credentials redacted.
- HSRP state and failover result.
- LACP member state.
- Rapid PVST+ root placement.
- End-to-end client Internet test.

## 5. HQ BGP policy typo

**Observed in EDGE-01 `show running-config`:** An inbound route-map is applied as `PREFER-ISP01`, but the intended no-match permit clause is spelled `PREFER-IPS01 permit 20`. It therefore belongs to a different route-map and does not permit other routes under `PREFER-ISP01`. EDGE-02 spells the clause correctly.

**Follow-up:** Back up the running config, correct the name, and verify received routes and `LOCAL_PREF` using `show ip bgp`. A configuration typo may affect routes other than the default.

## 6. ISP-2 advertises a default without complete NAT

**Observed in ISP-2 `show running-config`:** Both HQ peers receive `default-originate`, but ISP-2 has no configured PAT rule and the two HQ edges have PAT route-maps for ISP-1 only.

**Consequence:** A BGP default route does not by itself demonstrate functional failover. Configure and validate both enterprise-edge and ISP-2 PAT, as well as upstream routing and reply traffic.

## 7. Defined but unused BGP filters

**Observed:** `ISP-IN` and `ISP-OUT` prefix lists are defined on HQ edges but are not applied in the supplied BGP address-family neighbor configuration. `ISP-OUT` as written denies all IPv4 routes.

**Follow-up:** Define deliberate import/export policies and verify route advertisements. Never connect a documentation-prefix lab to real Internet BGP without a controlled isolation boundary.
