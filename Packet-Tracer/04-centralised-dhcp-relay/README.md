# Lab 04: Centralised DHCP with relay across four branches

**My own build** (Packet Tracer 8.2.2): [`lab04-centralised-dhcp.pkt`](lab04-centralised-dhcp.pkt)

## Goal

Serve IP addresses to four branch LANs from **one central DHCP server**, instead of a DHCP server per site. Branch routers forward DHCP broadcasts to the server with **DHCP relay** (`ip helper-address`), and static routes connect every network.

## Topology

```
 Branch 1 (Switch6, PC15-18)            Branch 3 (Switch7, PC27-30)
 100.150.50.0/27                        100.150.50.64/27
        |  G0/0 .1                             |  G0/1 .65
      [R3] G0/1 .33 -- Branch 2         [R4] G0/0 .97 -- Branch 4
        |       (Switch3, PC19-22)        |       (Switch5, PC23-26)
        |       100.150.50.32/27          |       100.150.50.96/27
   G0/2 .1                           G0/2 .10
   120.10.10.0/30                    120.10.10.8/30
   G0/0 .2                           G0/1 .9
        \_______________ [R5 core] _______________/
                            | G0/2 150.30.30.1
                        (Switch4)
                     DHCP server 150.30.30.2/29
```

Devices: 3 × Cisco 2911 routers, 5 × 2960 switches, 1 × Server-PT (DHCP), 16 × PC.

## Addressing

| Network | Subnet | Gateway | Where |
|---------|--------|---------|-------|
| Branch 1 | 100.150.50.0/27 | 100.150.50.1 | R3 G0/0 |
| Branch 2 | 100.150.50.32/27 | 100.150.50.33 | R3 G0/1 |
| Branch 3 | 100.150.50.64/27 | 100.150.50.65 | R4 G0/1 |
| Branch 4 | 100.150.50.96/27 | 100.150.50.97 | R4 G0/0 |
| Server LAN | 150.30.30.0/29 | 150.30.30.1 | R5 G0/2, server .2 |
| R3–R5 link | 120.10.10.0/30 | .1 / .2 | point-to-point |
| R4–R5 link | 120.10.10.8/30 | .10 / .9 | point-to-point |

One /24 (100.150.50.0) is split into four **/27** subnets (30 usable hosts each); the router links use **/30** (2 hosts), and the server LAN a **/29** (6 hosts).

## DHCP pools (on the server)

| Pool | Range | Default router | Max users |
|------|-------|----------------|-----------|
| branch1 | 100.150.50.3 – .30 | 100.150.50.1 | 28 |
| branch2 | 100.150.50.35 – .62 | 100.150.50.33 | 28 |
| branch3 | 100.150.50.67 – .94 | 100.150.50.65 | 28 |
| branch4 | 100.150.50.99 – .126 | 100.150.50.97 | 28 |

## How the relay works

1. A PC broadcasts **DHCP Discover**. Routers don't forward broadcasts, so it would never reach the server on its own.
2. The branch router interface has `ip helper-address 150.30.30.2`. It turns the broadcast into a **unicast** to the server and writes its own interface IP into the **giaddr** field (e.g. 100.150.50.65 for branch 3).
3. The server picks the pool whose network contains the giaddr, which is how one server serves four subnets.
4. The Offer/ACK goes back to the relay router, which delivers it to the PC.
5. This only works if the server has a **route back** to every giaddr. Here R5 has static routes to all four branch subnets, and the server's gateway is R5.

## Routing

Static routes on each router. Branch routers send everything for the other branches and the server LAN to R5; R5 knows where every branch lives. Configs as saved in the file: [R3](configs/R3-as-built.txt), [R4](configs/R4-as-built.txt), [R5](configs/R5-as-built.txt).

## Verify

| Where | Command / action | Expect |
|-------|------------------|--------|
| PC | Desktop → IP Configuration → DHCP | Address from its branch pool, correct gateway |
| PC | `ipconfig /all` | DHCP server 150.30.30.2 |
| PC (branch 1) | `ping 100.150.50.67` (a branch 3 PC) | Replies: routing works end to end |
| R3 / R4 | `show running-config \| include helper` | Relay on each LAN interface |
| R5 | `show ip route static` | Routes to all four /27 branches |
| Simulation mode | Filter DHCP, renew on a PC | Discover → relay unicast → Offer → Request → ACK, with giaddr set |

## Review findings

From reviewing the saved file:

| # | Finding | Impact | Fix |
|---|---------|--------|-----|
| 1 | 10 of 16 PCs not set to DHCP; PC19 and PC28 show APIPA (169.254.x.x) | Those PCs have no usable address; branch 4 had no leases at all | Set every PC to DHCP; retry the APIPA ones |
| 2 | R4 has two static routes via 120.10.10.1, which isn't on any R4 interface | Unusable routes, confusing during troubleshooting | Remove them (the correct routes via .9 are already there) |
| 3 | Branch pools give DNS .2/.34/.66/.98, where no DNS server exists | Name resolution would fail | Point DNS to 150.30.30.2 |
| 4 | `ip helper-address` on R3's transit link G0/2 | No clients there; harmless but unnecessary | Remove it |
| 5 | Default `serverPool` starts at 150.30.30.0 and overlaps the router and server | Could hand out .1/.2 if a client appeared on that LAN | Start at .3 |
| 6 | All routers named `Router`, all switches `Switch`; no passwords | Hard to tell devices apart in a real network | Set hostnames, console/enable passwords |

Commands for all fixes: [configs/fixes.txt](configs/fixes.txt).

## Break and fix

**Ticket 04-A: A whole branch gets 169.254.x.x**
- Break: on R4, `interface g0/0` then `no ip helper-address 150.30.30.2`.
- Symptom: branch 4 PCs get APIPA addresses; other branches are fine.
- Diagnose: only one branch affected → look at that branch's router; `show run interface g0/0` shows no relay.
- Fix: add the helper address back; renew on the PCs.

**Ticket 04-B: Relay configured, but a branch still gets no address**
- Break: on R5, `no ip route 100.150.50.64 255.255.255.224 120.10.10.10`.
- Symptom: branch 3 can't get addresses even though the relay is configured. Discover reaches the server, but the Offer has no route back to giaddr 100.150.50.65.
- Diagnose: simulation mode shows the Offer dropped at R5; `show ip route` on R5.
- Fix: restore the route. Lesson: DHCP relay needs routing in **both** directions.

**Ticket 04-C: Pool exhausted**
- Break: set branch1 Max Users to 1.
- Symptom: the first PC gets an address, the next ones get APIPA.
- Fix: restore the pool size. In production, watch pool utilisation in monitoring.

## What I learned

- Why DHCP needs a relay across routers, and how the server picks the right pool from giaddr.
- Subnetting one /24 into /27s and using /30s for router links.
- Static routing has to be right in both directions, and a route with an unreachable next hop just sits there unused.

## My notes

- Date built: October 2026
- Problems I hit and how I solved them:
- What I'd explain in an interview:
