# DHCP and DNS

## DHCP: DORA

1. **Discover**: client broadcasts looking for a server.
2. **Offer**: server offers an address.
3. **Request**: client asks for the offered address.
4. **Acknowledge**: server confirms the lease (address, mask, gateway, DNS, lease time).

If the server is in another subnet, the router forwards the broadcast with `ip helper-address <server-ip>` (DHCP relay).

### DHCP troubleshooting

| Symptom | Check |
|---------|-------|
| Client has 169.254.x.x (APIPA) | Server reachable? Pool exists and isn't full? Relay configured on that VLAN? |
| Wrong gateway or DNS | Pool options (`default-router`, `dns-server`) |
| Address conflict | `show ip dhcp conflict`; static device inside the pool; add an exclusion |

```
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
ipconfig /release   then   ipconfig /renew      (Windows client)
```

## DNS

| Record | Purpose |
|--------|---------|
| A / AAAA | Name to IPv4 / IPv6 |
| CNAME | Alias to another name |
| MX | Mail server for a domain |
| PTR | Reverse lookup (IP to name) |
| NS | Authoritative servers for a zone |
| TXT | Text, e.g. SPF, verification |

### DNS troubleshooting

- **Ping by IP works, by name fails** → DNS.
- `nslookup name` (and `nslookup name <other-dns-server>` to compare servers).
- `ipconfig /displaydns`, `ipconfig /flushdns` for client cache problems.
- `Resolve-DnsName name -Type A` in PowerShell.
- Recently changed record still wrong? Check the **TTL**: old answers stay cached until it expires.
