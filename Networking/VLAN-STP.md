# VLANs, Trunks and Spanning Tree

## Concepts

- **VLAN**: a separate Layer 2 broadcast domain on the same switch hardware. Each VLAN is normally its own IP subnet.
- **Access port**: belongs to one VLAN; frames are untagged.
- **Trunk port**: carries many VLANs between switches or to a router, tagged with **802.1Q**. The **native VLAN** is sent untagged and must match on both ends.
- **Inter-VLAN routing**: a router (router-on-a-stick subinterfaces) or a Layer 3 switch (SVIs) routes between VLANs.
- **STP (802.1D / Rapid PVST+)**: blocks redundant links to stop Layer 2 loops. One root bridge per VLAN; other switches keep their best path to it.

## Verify

```
show vlan brief                      ! which ports are in which VLAN
show interfaces trunk                ! trunking ports, native VLAN, allowed VLANs
show interfaces fa0/2 switchport     ! mode and VLAN of one port
show mac address-table               ! where each MAC is learned
show spanning-tree vlan 10           ! root bridge, port roles and states
```

## Troubleshooting

| Symptom | Likely cause | Check |
|---------|--------------|-------|
| One PC can't reach its gateway | Port in the wrong VLAN, port shut down | `show vlan brief`, `show interfaces status` |
| A whole VLAN can't cross to another switch | VLAN not allowed on the trunk, or VLAN not created on the far switch | `show interfaces trunk` |
| "Native VLAN mismatch" logs | Different native VLAN at each end of a trunk | `show interfaces trunk` both sides |
| Network slow, high CPU, MAC flapping | Layer 2 loop, STP disabled or misconfigured | `show spanning-tree`, interface counters |
| Host gets 169.254.x.x | DHCP not reachable from that VLAN (pool, relay `ip helper-address`) | `show ip dhcp binding`, helper config |

## Hardening

- Put unused ports in a parking VLAN and shut them down.
- Change the native VLAN away from VLAN 1 and don't use it for users.
- Enable PortFast and BPDU Guard on access ports only.
- Restrict allowed VLANs on trunks.
