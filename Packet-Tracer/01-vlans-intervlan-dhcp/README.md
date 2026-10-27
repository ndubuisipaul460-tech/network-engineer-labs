# Lab 01: VLANs, inter-VLAN routing and DHCP

## Goal

Split one office into separate VLANs for users, servers and management, route between them with a single router (router-on-a-stick), and hand out addresses with DHCP from the router.

## Topology

```
                 [R1 2911]
                  G0/0 (trunk, subinterfaces .10 .20 .99)
                    |
                 G0/1
                 [S1 2960]
        Fa0/1-10   Fa0/11-20    VLAN 99 SVI
           |           |
       PC-U1, PC-U2   SRV1 (static)
       (VLAN 10)     (VLAN 20)
```

Devices: 1 × 2911 router, 1 × 2960 switch, 2 × PC, 1 × Server.

## Addressing

| VLAN | Name | Subnet | Gateway (R1) | Hosts |
|------|------|--------|--------------|-------|
| 10 | USERS | 192.168.10.0/24 | 192.168.10.1 | DHCP .100 to .199 |
| 20 | SERVERS | 192.168.20.0/24 | 192.168.20.1 | SRV1 192.168.20.10 (static) |
| 99 | MGMT | 192.168.99.0/24 | 192.168.99.1 | S1 192.168.99.2 |
| 999 | PARKING | (unused ports, shut down) | | |

## Build

Full configs are in [configs/](configs/). Paste them in global configuration mode, then `copy running-config startup-config`.

Key ideas:
- **S1** carries all VLANs to R1 on one 802.1Q trunk (G0/1); native VLAN changed to 99 so no user traffic is untagged.
- **R1** has one subinterface per VLAN with `encapsulation dot1Q <vlan>`; each subinterface is that VLAN's default gateway.
- **DHCP** runs on R1 for VLAN 10; `.1` to `.99` are excluded for static devices.
- Unused switch ports go to VLAN 999 and are shut down.

## Verify

| Command | Device | What to look for |
|---------|--------|------------------|
| `show vlan brief` | S1 | Fa0/1-10 in VLAN 10, Fa0/11-20 in VLAN 20, rest in 999 |
| `show interfaces trunk` | S1 | G0/1 trunking, native VLAN 99, VLANs 10,20,99 allowed |
| `show ip interface brief` | R1 | G0/0.10, .20, .99 up/up |
| `show ip dhcp binding` | R1 | PC-U1 and PC-U2 have leases from .100 up |
| `ipconfig` | PC-U1 | 192.168.10.1xx, gateway 192.168.10.1 |
| `ping 192.168.20.10` | PC-U1 | Replies: inter-VLAN routing works |
| `tracert 192.168.20.10` | PC-U1 | First hop 192.168.10.1, then the server |

## Break and fix

Do each fault, then fix it using only show commands to find the cause. Write what you saw in "My notes".

**Ticket 01-A: Users get 169.254.x.x addresses**
- Break: on R1, `no ip dhcp pool USERS` (save the pool commands first).
- Symptom: PCs in VLAN 10 show an APIPA address; no network access.
- Diagnose: `ipconfig` on the PC, `show ip dhcp pool` and `show ip dhcp binding` on R1.
- Fix: recreate the pool, then `ipconfig /renew` on the PCs.

**Ticket 01-B: Users can't reach the server, but can reach each other**
- Break: on S1, `interface g0/1` then `switchport trunk allowed vlan 10,99`.
- Symptom: PC to PC works; PC to SRV1 times out.
- Diagnose: `show interfaces trunk` on S1 shows VLAN 20 missing from the allowed list.
- Fix: `switchport trunk allowed vlan 10,20,99`.

**Ticket 01-C: One user is offline after a desk move**
- Break: on S1, put Fa0/2 into VLAN 20.
- Symptom: PC-U2 gets no DHCP address (or the wrong subnet if set statically).
- Diagnose: `show vlan brief`, `show interfaces fa0/2 switchport`.
- Fix: `switchport access vlan 10`.

**Ticket 01-D: Native VLAN mismatch messages on the console**
- Break: on S1, `switchport trunk native vlan 1` on G0/1.
- Symptom: CDP native VLAN mismatch logs; management traffic unreliable.
- Diagnose: console logs, `show interfaces trunk`.
- Fix: set native VLAN 99 again, matching R1's `G0/0.99 native` subinterface.

## My notes

- Date built:
- What worked first time:
- Problems I hit and how I solved them:
- What I'd explain in an interview:
