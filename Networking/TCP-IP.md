# TCP/IP Essentials

## OSI vs TCP/IP

| OSI layer | TCP/IP layer | Examples | First check when it breaks |
|-----------|--------------|----------|----------------------------|
| 7 Application, 6 Presentation, 5 Session | Application | HTTP, DNS, SMTP, TLS | Does the service answer? Certificate errors? |
| 4 Transport | Transport | TCP, UDP, ports | Is the port open? Firewall deny? |
| 3 Network | Internet | IP, ICMP, routing | `ping`, `traceroute`, routing table |
| 2 Data link | Link | Ethernet, MAC, VLAN, ARP | `show mac address-table`, `arp -a`, VLAN membership |
| 1 Physical | Link | Cables, optics, Wi-Fi | Link lights, `show interfaces` errors |

## TCP vs UDP

| | TCP | UDP |
|--|-----|-----|
| Connection | Three-way handshake (SYN, SYN-ACK, ACK) | None |
| Reliability | Acknowledgements, retransmission, ordering | Best effort |
| Used by | HTTP/S, SSH, RDP, SMB, LDAP | DNS queries, DHCP, SNMP, syslog, NTP |

A **RST** in reply to a SYN means the host is reachable but nothing listens on the port (or a firewall rejects). **No reply at all** usually means a firewall silently drops it, or a routing problem.

## Common ports

| Port | Service | Port | Service |
|------|---------|------|---------|
| 22 | SSH | 161/162 | SNMP / SNMP traps |
| 25 | SMTP | 389 / 636 | LDAP / LDAPS |
| 53 | DNS | 443 | HTTPS |
| 67/68 | DHCP | 445 | SMB |
| 80 | HTTP | 514 | Syslog |
| 123 | NTP | 3389 | RDP |

## Subnetting quick table

| CIDR | Mask | Addresses | Usable hosts |
|------|------|-----------|--------------|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 (point-to-point links) |

Private ranges (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

## Windows diagnostics

```powershell
ipconfig /all                          # IP, gateway, DNS, DHCP lease
ping 192.168.1.1 -n 4                  # reachability
tracert 8.8.8.8                        # path, where it stops
nslookup app.example.com               # DNS resolution
Test-NetConnection app.example.com -Port 443   # TCP port test
netstat -ano | findstr :443            # listening/established sockets + PID
arp -a                                 # IP to MAC cache
route print                            # routing table
```
