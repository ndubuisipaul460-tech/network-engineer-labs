# Cisco ASA CLI Cheat Sheet

Operational commands for troubleshooting traffic on a Cisco ASA firewall.

## Key concepts

- **Interfaces** have a `nameif` (inside, outside, dmz) and a **security level** (0–100). By default, traffic may flow from a higher to a lower level (inside 100 → outside 0); the reverse needs an access-list.
- The ASA is **stateful**: return traffic for an allowed connection is permitted automatically.
- **ACLs** are applied to an interface with `access-group <ACL> in interface <nameif>`.
- **NAT** is configured with network objects (`object network` … `nat (inside,outside) dynamic interface`).

## System and interfaces

```
show version                         ! model, ASA version, uptime, licence
show interface ip brief              ! interfaces, IPs, status
show nameif                          ! interface names and security levels
show route                           ! routing table
show running-config interface
show failover                        ! active/standby state (HA pairs)
```

## Connections, NAT and access-lists

```
show conn address 192.168.1.10       ! active connections for a host
show xlate                           ! current NAT translations
show nat detail                      ! NAT rules with hit counts
show access-list OUTSIDE_IN          ! ACL entries with hit counts
show running-config access-group     ! which ACL is applied where
```

## Packet tracer (simulates a packet through the ASA)

```
packet-tracer input inside tcp 192.168.1.10 51000 8.8.8.8 443 detailed
```
Shows every phase (route lookup, ACL, NAT, inspection, VPN) and whether the packet is **ALLOWED** or **DROPPED**, with the rule responsible. This is the first command to run for "is the firewall blocking it?"

## Packet capture

```
capture CAP interface inside match ip host 192.168.1.10 host 10.10.10.5
show capture CAP                     ! view packets
show capture CAP detail
copy /pcap capture:CAP tftp://10.0.0.5/cap.pcap   ! open later in Wireshark
no capture CAP                       ! always clean up
capture ASP type asp-drop all        ! packets the ASA itself dropped
show asp drop                        ! drop counters with reasons
```

## Site-to-site VPN

```
show crypto ikev1 sa                 ! phase 1 (IKEv1)  - MM_ACTIVE = up
show crypto ikev2 sa                 ! phase 1 (IKEv2)  - READY = up
show crypto ipsec sa                 ! phase 2: encaps/decaps counters
show vpn-sessiondb l2l               ! site-to-site sessions summary
debug crypto ikev1 127               ! lab only; undebug all afterwards
```

## Logging

```
show logging                         ! buffered syslog
show logging | include 192.168.1.10
```
Common messages: **%ASA-4-106023** (denied by access-group), **%ASA-6-302013/302014** (TCP connection built/torn down), **%ASA-3-710003** (management access denied).
