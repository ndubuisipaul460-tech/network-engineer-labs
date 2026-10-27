# Lab 03: Site-to-site IPsec VPN

## Goal

Join a head office and a branch over an untrusted "internet" router with an encrypted IPsec tunnel, so the two LANs talk privately, then troubleshoot the most common VPN faults.

## Topology

```
 HQ LAN                                                     BRANCH LAN
192.168.1.0/24                                            192.168.3.0/24
   PC-HQ --- [R-HQ] G0/1 ---- [ISP] ---- G0/1 [R-BR] --- PC-BR
            10.1.1.1/30   10.1.1.2 | 10.2.2.2   10.2.2.1/30
                                   (no routes to private LANs)
```

Devices: 3 × 2911 router, 2 × PC (connect each PC to its router's G0/0 directly, or through a switch).

## Addressing

| Device | Interface | Address |
|--------|-----------|---------|
| R-HQ | G0/0 | 192.168.1.1/24 |
| R-HQ | G0/1 | 10.1.1.1/30 |
| ISP | G0/0 | 10.1.1.2/30 |
| ISP | G0/1 | 10.2.2.2/30 |
| R-BR | G0/1 | 10.2.2.1/30 |
| R-BR | G0/0 | 192.168.3.1/24 |
| PC-HQ | | 192.168.1.10, gateway .1 |
| PC-BR | | 192.168.3.10, gateway .1 |

The ISP router knows nothing about 192.168.x.x, just like the real internet. The only way the LANs can reach each other is through the tunnel.

## Before you start: enable the security licence

The 2911 needs the security package for `crypto` commands. On R-HQ and R-BR:

```
enable
configure terminal
license boot module c2900 technology-package securityk9
(accept with yes)
end
copy running-config startup-config
reload
```

Check with `show version`: `securityk9` should show as enabled.

## Build

Configs are in [configs/](configs/). The tunnel settings must **match on both ends**:

| Setting | Value |
|---------|-------|
| Phase 1 (ISAKMP) | AES-256, SHA, pre-shared key, DH group 5, lifetime 86400 |
| Pre-shared key | `vpnLab01` (lab only) |
| Phase 2 transform set | `esp-aes esp-sha-hmac` |
| Interesting traffic | HQ: 192.168.1.0/24 → 192.168.3.0/24; Branch: the mirror |

## Verify

1. From PC-HQ, `ping 192.168.3.10`. The first ping or two may time out while the tunnel comes up. That's normal.
2. On R-HQ:

| Command | Expect |
|---------|--------|
| `show crypto isakmp sa` | State `QM_IDLE` with peer 10.2.2.1 (phase 1 up) |
| `show crypto ipsec sa` | `#pkts encaps` and `#pkts decaps` increasing (phase 2 passing traffic) |
| `show crypto map` | Peer 10.2.2.1, ACL 110, transform set VPN-SET, applied to G0/1 |

3. Ping 10.2.2.1 from PC-HQ instead: it isn't in the interesting traffic, so it doesn't go through the tunnel. Notice that the encaps counter doesn't change.

## Break and fix

**Ticket 03-A: Tunnel won't come up, phase 1 fails**
- Break: on R-BR, change the key: `no crypto isakmp key vpnLab01 address 10.1.1.1` then `crypto isakmp key wrongKey address 10.1.1.1`.
- Symptom: LAN-to-LAN pings fail; `show crypto isakmp sa` shows no `QM_IDLE`.
- Diagnose: phase 1 never completes, so check the policy and the key on both sides.
- Fix: set the same pre-shared key on both peers.

**Ticket 03-B: Phase 1 up, but no traffic passes**
- Break: on R-BR, `crypto ipsec transform-set VPN-SET esp-3des esp-sha-hmac` (doesn't match HQ).
- Symptom: `show crypto isakmp sa` may show phase 1, but `show crypto ipsec sa` counters stay at 0.
- Diagnose: phase 2 proposals differ; compare `show crypto ipsec transform-set` on both sides.
- Fix: put back `esp-aes esp-sha-hmac`.

**Ticket 03-C: Only some traffic goes through the tunnel**
- Break: on R-HQ, change ACL 110 to `permit ip 192.168.1.0 0.0.0.255 192.168.30.0 0.0.0.255`.
- Symptom: HQ to Branch fails; the ACLs are no longer mirror images.
- Diagnose: `show access-lists` on both routers.
- Fix: correct the destination to 192.168.3.0.

**Ticket 03-D: Peer unreachable**
- Break: on ISP, shut down G0/1.
- Symptom: tunnel down; `ping 10.2.2.1` from R-HQ fails.
- Diagnose: always test basic reachability between the peers before blaming the VPN.
- Fix: `no shutdown`. Lesson: check the underlay (the path between the peers) first.

## My notes

- Date built:
- What worked first time:
- Problems I hit and how I solved them:
- What I'd explain in an interview:
