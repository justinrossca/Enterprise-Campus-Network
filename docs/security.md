# Access-layer security

[← Back to main README](../README.md)

## Controls in the baseline configuration

| Control | Where | Purpose |
| --- | --- | --- |
| VLAN 10/20 separation | PC-facing access interfaces | Separate broadcast domains for users and staff |
| VLAN 90 management | Access switch management SVIs | Avoid placing switch management IPs on user VLANs |
| VLAN 999 | Unused ports and trunk native VLAN | Keep unused access ports outside production VLANs |
| DHCP snooping | Access switches | Identify trusted DHCP server paths and learn client bindings |
| Dynamic ARP Inspection | Access switches | Validate ARP on untrusted ports, using DHCP binding information |
| Port security | `Gi0/2` on both access switches | Sticky MAC learning, one MAC maximum, violation restrict |
| SSH | Infrastructure devices | Remote CLI management without Telnet |

## Trusted versus untrusted interfaces

The access-switch uplinks to distribution are trusted for DHCP snooping and DAI in the baseline. The PC-facing interface remains untrusted. This placement matters: trusting the PC port would bypass the checks, while failing to trust the upstream relay/server path could interrupt DHCP operation.

For DHCP clients, verify that a binding appears **before** interpreting DAI drops. Static-IP clients might need separate ARP ACLs or static binding handling.

## Verification commands

```cisco
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection statistics
show port-security interface gigabitEthernet0/2
show interfaces trunk
```

## Scope limits

- VLANs are not an inter-VLAN firewall. The baseline documentation does not claim inbound SVI ACL segmentation.
- The switch management VLAN is separated at Layer 2, but administrative access should eventually be restricted with VTY management ACLs.
- Archived public configs omit the local administrator entries, so they are **not** deployable standalone templates.
- PortFast/BPDU Guard and final STP security should be checked against the live switch configs; older archived configs omit later changes.
