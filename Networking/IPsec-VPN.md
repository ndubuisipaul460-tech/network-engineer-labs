# Site-to-site IPsec VPN

## Two phases

| Phase | Builds | Must match on both peers |
|-------|--------|--------------------------|
| Phase 1 (IKE / ISAKMP) | A secure management channel between the peers | Encryption, hash, authentication (pre-shared key or certificates), DH group, lifetime, peer address |
| Phase 2 (IPsec SA) | The tunnel that carries user traffic (ESP) | Transform set (encryption + integrity), protected networks (proxy IDs / crypto ACL), PFS group if used |

**Interesting traffic** is what the crypto ACL matches. It must be a **mirror image** on each side, or phase 2 fails.

## Verify (Cisco IOS)

```
show crypto isakmp sa          ! QM_IDLE = phase 1 up
show crypto ipsec sa           ! encaps/decaps counters rising = traffic flowing
show crypto map                ! peer, ACL, transform set, applied interface
show access-lists              ! crypto ACL hit counts
debug crypto isakmp            ! lab only; very noisy in production
```

## Troubleshooting order

1. **Underlay first**: can the peers ping each other's public IP? Is UDP 500/4500 and ESP allowed?
2. **Phase 1**: policy and pre-shared key match? Peer IP correct?
3. **Phase 2**: transform sets match? ACLs mirrored? Same subnets both sides?
4. **Traffic**: are packets encapsulated (encaps) but nothing decrypted on the far side? Check routing and NAT (NAT must exempt VPN traffic).
5. **Recent changes**: key rotation, firewall rule change, ISP change of public IP.
