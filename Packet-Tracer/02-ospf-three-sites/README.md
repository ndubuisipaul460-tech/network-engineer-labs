# Lab 02: OSPF across three sites

## Goal

Connect three sites with dynamic routing (single-area OSPF), so each site learns the others' networks automatically and traffic reroutes when a link fails.

## Topology

```
              [R1 HQ]
           G0/0     G0/1
   10.0.12.0/30      10.0.13.0/30
         /                 \
   G0/0 [R2 BRANCH-A] G0/1 --- G0/1 [R3 BRANCH-B] G0/0
                   10.0.23.0/30
  LANs: R1 G0/2 192.168.1.0/24   R2 G0/2 192.168.2.0/24   R3 G0/2 192.168.3.0/24
```

Devices: 3 × 2911 router, 3 × 2960 switch (one per LAN), one PC per LAN.

## Addressing

| Router | Interface | Address | Link to |
|--------|-----------|---------|---------|
| R1 | G0/0 | 10.0.12.1/30 | R2 G0/0 |
| R1 | G0/1 | 10.0.13.1/30 | R3 G0/0 |
| R1 | G0/2 | 192.168.1.1/24 | HQ LAN |
| R1 | Lo0 | 1.1.1.1/32 | router ID |
| R2 | G0/0 | 10.0.12.2/30 | R1 |
| R2 | G0/1 | 10.0.23.1/30 | R3 |
| R2 | G0/2 | 192.168.2.1/24 | Branch A LAN |
| R2 | Lo0 | 2.2.2.2/32 | router ID |
| R3 | G0/0 | 10.0.13.2/30 | R1 |
| R3 | G0/1 | 10.0.23.2/30 | R2 |
| R3 | G0/2 | 192.168.3.1/24 | Branch B LAN |
| R3 | Lo0 | 3.3.3.3/32 | router ID |

Note: cable R1 G0/1 to R3 **G0/0**, and R2 G0/1 to R3 **G0/1**. Use copper cross-over or let Packet Tracer pick the cable automatically.

PCs: PC1 192.168.1.10, PC2 192.168.2.10, PC3 192.168.3.10, gateway `.1` of their LAN.

## Build

Configs are in [configs/](configs/). Key ideas:
- `router-id` set from the loopback, so neighbours are easy to read.
- `network ... area 0` statements with wildcard masks.
- `passive-interface g0/2` stops hellos going out to the LANs, where there are no routers.
- `auto-cost reference-bandwidth 1000` so gigabit links get a sensible cost.

## Verify

| Command | Where | Expect |
|---------|-------|--------|
| `show ip ospf neighbor` | each router | 2 neighbours in FULL state |
| `show ip route ospf` | R1 | 192.168.2.0 and 192.168.3.0 learned with `O` |
| `show ip ospf interface brief` | R1 | G0/0, G0/1 active; G0/2 passive |
| `ping 192.168.3.10` | PC1 | Replies |
| `tracert 192.168.3.10` | PC1 | Via R1 then R3 (direct path) |

## Break and fix

**Ticket 02-A: Branch B unreachable after a "small change"**
- Break: on R3, `router ospf 1` then `no network 10.0.13.0 0.0.0.3 area 0` and `no network 10.0.23.0 0.0.0.3 area 0`.
- Symptom: PC1 can't reach PC3; R3 has no neighbours.
- Diagnose: `show ip ospf neighbor` (empty on R3), `show running-config | section ospf`.
- Fix: add the network statements back. Root cause: a change removed the network statements. Lesson: check recent changes first.

**Ticket 02-B: Neighbour stuck, never reaches FULL**
- Break: on R2 G0/0, `ip ospf hello-interval 5`.
- Symptom: R1 and R2 are not neighbours; traffic between HQ and Branch A takes the long way through R3.
- Diagnose: `show ip ospf interface g0/0` on both routers: hello and dead timers don't match.
- Fix: `no ip ospf hello-interval 5`.

**Ticket 02-C: Link failure and automatic reroute**
- Break: on R1, `interface g0/1` then `shutdown`.
- Symptom: none for users, which is the point. Traffic from HQ to Branch B now goes R1 → R2 → R3.
- Verify: `tracert 192.168.3.10` from PC1 before and after; `show ip route` on R1 shows the new path.
- Fix: `no shutdown`; watch the route change back.

**Ticket 02-D: Wrong subnet mask on a link**
- Break: on R2 G0/1, `ip address 10.0.23.1 255.255.255.0`.
- Symptom: R2 and R3 don't form a neighbourship.
- Diagnose: `show ip interface brief`, `show ip ospf interface`: the network/mask differs on each side.
- Fix: set the /30 mask back.

## My notes

- Date built:
- What worked first time:
- Problems I hit and how I solved them:
- What I'd explain in an interview:
