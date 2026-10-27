# Lab 05: Static routing across a four-router ring

**My own build** (Packet Tracer 8.2.2): [`lab05-static-routing.pkt`](lab05-static-routing.pkt)

## Goal

Connect four LANs through four routers in a ring using **static routes only**, subnetting a single /24 (200.170.95.0) into /27s. Then review the build, find what breaks reachability, and fix it.

## Topology

```
     LAN A (Switch0, PC25-27)                 LAN B (Switch1, PC31-33)
     200.170.95.0/27                          200.170.95.32/27
            | G0/0 .1                                | G0/2 .33
          [R0] G0/1 .129 ---- .128/27 ---- G0/0 .130 [R1]
            | G0/2 .225                              | G0/1 .161
         .224/27                                  .160/27
            | G0/1 .226                              | G0/1 .162
          [R2] G0/2 .194 ---- .192/27 ---- G0/2 .193 [R3]
            | G0/0 .97                               | G0/0 .65
     LAN D (Switch3, PC28-30)                 LAN C (Switch2, PC34-36)
     200.170.95.96/27                         200.170.95.64/27
```

Devices: 4 × Cisco 2911, 4 × 2960, 12 × PC.

## Addressing plan (/24 split into eight /27s)

| Subnet | Use | Addresses |
|--------|-----|-----------|
| 200.170.95.0/27 | LAN A | R0 .1, Switch0 .5, PCs .2–.4 |
| 200.170.95.32/27 | LAN B | R1 .33, Switch1 .37, PCs .34–.36 |
| 200.170.95.64/27 | LAN C | R3 .65, Switch2 .69, PCs .66–.68 |
| 200.170.95.96/27 | LAN D | R2 .97, PCs .98–.100 |
| 200.170.95.128/27 | R0–R1 link | .129 / .130 |
| 200.170.95.160/27 | R1–R3 link | .161 / .162 |
| 200.170.95.192/27 | R3–R2 link | .193 / .194 |
| 200.170.95.224/27 | R2–R0 link | .225 / .226 |

## Review findings

I decoded the saved file and compared every router's configuration with the cabling:

| # | Finding | Impact |
|---|---------|--------|
| 1 | **R1 has R2's addresses** (.97, .226, .194) instead of .130 / .161 / .33 | R1 shares no subnet with R0 or R3 and isn't a valid gateway for LAN B, so LAN B is cut off and the ring is broken on both of R1's links |
| 2 | **R0 and R3 have no static routes** (R0 has an empty `router rip`) | They only know their directly connected networks, so most traffic has nowhere to go |
| 3 | R2 is the only router with a complete, correct set of static routes | Its routes work once R1 and R3 are fixed |
| 4 | Switch0 and Switch1 management IPs are **swapped** (each has the other LAN's address); Switch3 has none | Switches can't be managed remotely |
| 5 | /27s on point-to-point links | Works, but wastes 28 addresses per link; /30 is the usual choice |
| 6 | Default hostnames (`Router`, `Switch`) | Hard to tell devices apart |

As-built configs: [R0](configs/Router0-as-built.txt) · [R1](configs/Router1-as-built.txt) · [R2](configs/Router2-as-built.txt) · [R3](configs/Router3-as-built.txt)

## Corrected design

Full configs: [configs/corrected-all-routers.txt](configs/corrected-all-routers.txt)

- Each router has a **primary static route** to every remote network along the shortest path.
- LANs on the opposite side of the ring (A↔C, B↔D) are two hops either way; one direction is chosen.
- **Floating static routes** (administrative distance 10) point the other way round the ring. If a link goes down, its primary route is removed and the backup is installed automatically.

Example (R0):
```
ip route 200.170.95.32 255.255.255.224 200.170.95.130        ! LAN B via R1
ip route 200.170.95.32 255.255.255.224 200.170.95.226 10     ! backup via R2
```

## Verify

| Command | Where | Expect |
|---------|-------|--------|
| `show ip interface brief` | each router | 3 interfaces up/up with the planned IPs |
| `show ip route static` | each router | 5 primary `S` routes (backups don't show while primaries are up) |
| `ping 200.170.95.66` | PC25 (LAN A) | Replies from LAN C |
| `tracert 200.170.95.98` | PC31 (LAN B) | R1 → R0 → R2 → PC |
| `show ip route 200.170.95.64` | R0 | Next hop 200.170.95.130 |

## Break and fix

**Ticket 05-A: Link failure, does traffic survive?**
- Break: on R0, `interface g0/1` then `shutdown`.
- Expect: `show ip route` on R0 now shows LAN B and LAN C via 200.170.95.226 (the floating routes). PC25 can still reach LAN B, the long way round.
- Restore: `no shutdown`; the primary routes return.

**Ticket 05-B: One LAN unreachable after a "copy-paste" change**
- Break: give R3 G0/2 the address 200.170.95.194 (duplicate of R2).
- Symptom: `%IP-4-DUPADDR` messages; LAN C unreachable from LAN D.
- Diagnose: `show ip interface brief` on both ends of the link. This is the same mistake as finding 1.
- Fix: set .193 back.

**Ticket 05-C: Return path missing**
- Break: on R3, remove `ip route 200.170.95.0 255.255.255.224 200.170.95.161` and its floating backup.
- Symptom: ping from PC25 to PC34 times out, though `show ip route` on R0 looks fine.
- Diagnose: traffic gets there; replies have no route back. Check the route table on the **far** router.
- Fix: add the routes back. Lesson: static routing must be correct in both directions.

## What I learned

- Subnetting a /24 into /27s and planning addresses per link.
- Static routes need a reachable next hop on a directly connected subnet, in both directions.
- Floating static routes give simple failover without a routing protocol.
- Reviewing a config against the physical cabling catches copy-paste errors fast.

## My notes

- Date built: October 2026
- Problems I hit and how I solved them:
