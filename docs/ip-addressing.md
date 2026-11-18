# VLAN and IP addressing plan

[← Back to main README](../README.md)

## VLANs and gateways

| VLAN | Purpose | Subnet | DIST-01 SVI | DIST-02 SVI | HSRP VIP |
| ---: | --- | --- | --- | --- | --- |
| 10 | Users | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.2` | `10.10.10.254` |
| 20 | Staff | `10.10.20.0/24` | `10.10.20.1` | `10.10.20.2` | `10.10.20.254` |
| 90 | Management | `10.10.90.0/24` | `10.10.90.1` | `10.10.90.2` | `10.10.90.254` |
| 999 | Native / unused | None | None | None | None |

**Reference management IPs:** `ACC-01 = 10.10.90.11/24` and `ACC-02 = 10.10.90.12/24`, with `ip default-gateway 10.10.90.254` on Layer 2 switches.

## Distribution ↔ edge routed links

| Link | Subnet | Distribution endpoint | Edge endpoint |
| --- | --- | --- | --- |
| DIST-01 ↔ EDGE-01 | `10.0.0.0/30` | DIST-01 Gi1/1 `10.0.0.1` | EDGE-01 Gi0/0 `10.0.0.2` |
| DIST-01 ↔ EDGE-02 | `10.0.0.4/30` | DIST-01 Gi1/2 `10.0.0.5` | EDGE-02 Gi0/3 `10.0.0.6` |
| DIST-02 ↔ EDGE-02 | `10.0.0.8/30` | DIST-02 Gi1/1 `10.0.0.9` | EDGE-02 Gi0/0 `10.0.0.10` |
| DIST-02 ↔ EDGE-01 | `10.0.0.12/30` | DIST-02 Gi1/2 `10.0.0.13` | EDGE-01 Gi0/3 `10.0.0.14` |

## Edge ↔ ISP links

| Connection | Enterprise edge | ISP peer |
| --- | --- | --- |
| EDGE-01 ↔ ISP-01 `198.51.100.0/30` | EDGE-01 Gi0/1 `198.51.100.2` | ISP-01 Gi0/1 `198.51.100.1` |
| EDGE-02 ↔ ISP-01 `198.51.100.4/30` | EDGE-02 Gi0/2 `198.51.100.6` | ISP-01 Gi0/2 `198.51.100.5` |
| EDGE-01 ↔ ISP-02 `192.0.2.4/30` | EDGE-01 Gi0/2 `192.0.2.6` | ISP-02 Gi0/2 `192.0.2.5` |
| EDGE-02 ↔ ISP-02 `192.0.2.0/30` | EDGE-02 Gi0/1 `192.0.2.2` | ISP-02 Gi0/1 `192.0.2.1` |
| ISP-01 ↔ ISP-02 `203.0.113.0/30` | ISP-01 Gi0/3 `203.0.113.1` | ISP-02 Gi0/3 `203.0.113.2` |

Both ISP Gi0/0 interfaces acquire upstream addresses from the GNS3 NAT/Internet network via DHCP.

## Enterprise routing loopbacks

| Device | Loopback1 | OSPF router ID |
| --- | --- | --- |
| EDGE-01 | `1.1.1.1/32` | `3.3.3.3` |
| EDGE-02 | `2.2.2.2/32` | `4.4.4.4` |

OSPF IDs on DIST-01 and DIST-02 are `1.1.1.1` and `2.2.2.2`. Those same numeric values also appear as the edge loopback **IP addresses**, not the edge OSPF router IDs. Verify the live routing table and avoid confusion during troubleshooting.

## DHCP design

EDGE-01 includes the original campus DHCP pools for VLANs 10, 20 and 90. Each DHCP `default-router` is the applicable HSRP virtual IP. Distribution SVIs relay requests toward EDGE-01's routed interfaces using `ip helper-address`.

The original source uses VLAN **99** in BGP `network` statements; the merged edge references change that prefix to VLAN **90**, matching the current campus. The combined configurations need live testing.

The `198.51.100.0/24`, `192.0.2.0/24` and `203.0.113.0/24` blocks are **reserved documentation prefixes** used only inside this isolated lab.
