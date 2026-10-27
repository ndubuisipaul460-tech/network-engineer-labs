# Network Engineer Labs

Hands-on networking labs and notes from my work in network and infrastructure support: VLANs, routing, site-to-site VPNs, firewalls and structured troubleshooting. Every Packet Tracer lab has full configs, verification commands and **break-and-fix tickets** written the way incidents are handled in an operations centre.

## Repository Structure

```
network-engineer-labs/
├── Packet-Tracer/          # build-it-yourself labs with configs and break/fix tickets
│   ├── 01-vlans-intervlan-dhcp/
│   ├── 02-ospf-three-sites/
│   └── 03-site-to-site-ipsec-vpn/
├── Networking/             # TCP/IP, VLAN/STP, OSPF, DHCP/DNS, IPsec notes
├── PaloAlto/               # CLI cheat sheet and troubleshooting method
└── Interview-Notes/        # networking interview Q&A
```

## Packet Tracer Labs

| # | Lab | What it covers | Break/fix tickets |
|---|-----|----------------|-------------------|
| 1 | [VLANs, inter-VLAN routing and DHCP](Packet-Tracer/01-vlans-intervlan-dhcp/) | 802.1Q trunks, router-on-a-stick, DHCP pools, unused-port hardening | APIPA addresses, pruned trunk, wrong access VLAN, native VLAN mismatch |
| 2 | [OSPF across three sites](Packet-Tracer/02-ospf-three-sites/) | Single-area OSPF, router IDs, passive interfaces, failover | Missing network statements, timer mismatch, link failure reroute, wrong mask |
| 3 | [Site-to-site IPsec VPN](Packet-Tracer/03-site-to-site-ipsec-vpn/) | IKE phase 1/2, crypto maps, interesting traffic | Key mismatch, transform-set mismatch, non-mirrored ACLs, underlay down |

Each lab follows the same layout: **Goal → Topology → Addressing → Build → Verify → Break and fix → My notes**.

## Troubleshooting Method

The same order I use on every ticket:

1. **Scope**: one user, one site, or everyone? When did it start? What changed?
2. **Bottom up** when everything is down (link, VLAN, IP, routing, firewall, app); **top down** when one application fails.
3. **Prove it** with output (`show`, `ping`, `traceroute`, packet capture), not assumptions.
4. **Fix within the process** (change or incident procedure), then **verify** and **document** the root cause.

## Skills Demonstrated

| Area | Topics |
|------|--------|
| Switching | VLANs, 802.1Q trunks, native VLAN, STP, port hardening |
| Routing | Static routing, OSPF, RIP to dynamic migration, default routes |
| Services | DHCP, DNS, NAT |
| Security | Site-to-site IPsec VPN, ACLs, Palo Alto NGFW policies |
| Troubleshooting | Structured fault isolation, `show` command analysis, Wireshark |

## Environment

- Cisco Packet Tracer 8.2.2 (Cisco 2911 routers, 2960 switches)
- Windows 11 with PowerShell

## Related Repositories

- [it-operations-labs](https://github.com/ndubuisipaul460-tech/it-operations-labs): monitoring scripts, ITIL runbooks and incident templates
- [linux-admin-labs](https://github.com/ndubuisipaul460-tech/linux-admin-labs): Linux administration notes and Bash scripts
- [wireshark-labs](https://github.com/ndubuisipaul460-tech/wireshark-labs): packet analysis labs
