# Lab 06: Four VLANs across three switches with router-on-a-stick

**My own build** (Packet Tracer 8.2.2): [`lab06-vlans.pkt`](lab06-vlans.pkt)

## Goal

Separate four departments into VLANs that span three switches, carry them over 802.1Q trunks, and route between them with one router using subinterfaces (router-on-a-stick). One /24 is split into four /26 subnets.

## Topology

```
 [SW0] G0/1 ==trunk== G0/1 [SW1] G0/2 ==trunk== G0/2 [SW2] G0/1 ==trunk== G0/1 [R1]
  PC0,1 (V10)               PC9 (V10)                PC13 (V10)        G0/1.10 .1
  PC2,3 (V20)               PC11 (V20)               PC14 (V20)        G0/1.20 .65
  PC5,6 (V30)               PC10 (V30)               PC15 (V30)        G0/1.30 .129
  PC7,8 (V40)               PC12 (V40)               PC16 (V40)        G0/1.40 .193
```

Devices: 1 × Cisco 2911, 3 × 2960, 16 × PC.

## VLAN and addressing plan

| VLAN | Name | Switch ports (all 3 switches) | Subnet | Gateway (R1 subinterface) |
|------|------|-------------------------------|--------|----------------------------|
| 10 | Sales | Fa0/1–5 | 192.168.10.0/26 | 192.168.10.1 (G0/1.10) |
| 20 | HR | Fa0/6–10 | 192.168.10.64/26 | 192.168.10.65 (G0/1.20) |
| 30 | IT | Fa0/11–15 | 192.168.10.128/26 | 192.168.10.129 (G0/1.30) |
| 40 | Accounting | Fa0/16–20 | 192.168.10.192/26 | 192.168.10.193 (G0/1.40) |

## How it works

- Each switch has the same VLANs and the same port-to-VLAN layout, so a PC's department depends only on which port it's in.
- Switch-to-switch and switch-to-router links are **trunks**; frames carry an 802.1Q tag with the VLAN ID.
- R1's G0/1 has one **subinterface per VLAN** (`encapsulation dot1Q <vlan>`), each being that VLAN's default gateway. Traffic between VLANs goes up the trunk to R1 and back down.

As-built router config: [configs/Router1-as-built.txt](configs/Router1-as-built.txt)

## Review findings

I decoded the saved file and checked every port, VLAN and address:

| # | Finding | Impact | Fix |
|---|---------|--------|-----|
| 1 | **Typo on R1**: G0/1.20 is `192.160.10.65` and G0/1.30 is `192.160.10.129` (160 instead of 168) | HR and IT PCs have no reachable gateway, so they can't talk to any other VLAN. Sales ↔ Accounting works | Set 192.168.10.65 and 192.168.10.129 |
| 2 | **PC6** is on SW0 Fa0/12 (VLAN 30 IT) but has 192.168.10.45, a VLAN 10 address | PC6 can't reach its gateway or anyone in VLAN 30 | Give it 192.168.10.136/26 |
| 3 | Trunks allow all VLANs | Works; best practice is to allow only VLANs in use | `switchport trunk allowed vlan 10,20,30,40` |
| 4 | Default hostnames, unused ports active | Harder to manage; unused ports are a security risk | Name devices; shut unused ports |

All fix commands: [configs/fixes.txt](configs/fixes.txt). Everything else checked out: all 16 PCs are in the right VLAN for their port, and VLAN names match on all switches.

## Verify

| Command | Where | Expect |
|---------|-------|--------|
| `show vlan brief` | each switch | VLANs 10/20/30/40 named, Fa0/1-20 assigned |
| `show interfaces trunk` | each switch | G0/1 (and G0/2) trunking, VLANs 10-40 allowed |
| `show ip interface brief` | R1 | G0/1.10/.20/.30/.40 up with 192.168.10.x addresses |
| `ping 192.168.10.65` | PC0 (Sales) | Gateway of HR replies (after fix 1) |
| `ping 192.168.10.180` | PC11 (HR) | PC15 in IT replies: inter-VLAN routing works |
| `tracert 192.168.10.243` | PC0 | First hop 192.168.10.1, then the PC |

## Break and fix

**Ticket 06-A: One department can't reach the others (the real bug in this lab)**
- Symptom: HR PCs ping each other but nothing outside HR.
- Diagnose: `ping` the gateway from an HR PC (fails); `show ip interface brief` on R1 shows 192.160.10.65.
- Fix: correct the address. Lesson: if one whole VLAN fails, check its gateway subinterface first.

**Ticket 06-B: A VLAN works on SW0 but not on SW2**
- Break: on SW1, `interface g0/2` then `switchport trunk allowed vlan 10,20,30`.
- Symptom: Accounting PCs on SW0 and SW1 can't reach the router; Accounting on SW2 still can.
- Diagnose: `show interfaces trunk` on SW1 shows VLAN 40 isn't allowed towards SW2/R1.
- Fix: add VLAN 40 back.

**Ticket 06-C: New PC in the wrong VLAN**
- Break: move PC0's cable to SW0 Fa0/8.
- Symptom: PC0 (Sales address) is now in HR's VLAN and reaches nothing.
- Diagnose: `show mac address-table` to find the port, `show vlan brief` for its VLAN.
- Fix: move the cable back or change the port's access VLAN.

## What I learned

- VLANs spanning several switches need consistent VLAN databases and trunks all the way to the router.
- Subnetting a /24 into four /26s, one per VLAN.
- One typo in a gateway address takes down a whole department, so verify each subinterface after configuring.

## My notes

- Date built: October 2026
- Problems I hit and how I solved them:
