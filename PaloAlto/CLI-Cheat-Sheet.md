# Palo Alto NGFW CLI Cheat Sheet

Operational commands for troubleshooting traffic on PAN-OS. Run in operational mode unless noted.

## System and interfaces

```
show system info                               # model, PAN-OS version, uptime, serial
show interface all                             # link state and IPs
show interface ethernet1/1                     # counters, errors for one port
show routing route                             # routing table (virtual router)
show arp all
show high-availability state                   # HA role, peer status
```

## Sessions and policy

```
show session all filter source 192.168.1.10 destination 10.10.10.5
show session id <id>                           # rule matched, NAT, app, state
show counter global filter delta yes severity drop   # why packets are dropped
test security-policy-match from trust to untrust source 192.168.1.10 destination 8.8.8.8 protocol 6 destination-port 443
test nat-policy-match from trust to untrust source 192.168.1.10 destination 8.8.8.8 protocol 6 destination-port 443
```

## Logs

```
show log traffic direction equal backward      # newest first
show log traffic src in 192.168.1.10
show log system direction equal backward
```

## IPsec VPN

```
show vpn ike-sa                                 # phase 1
show vpn ipsec-sa                               # phase 2
show vpn flow                                   # tunnel IDs and state
test vpn ike-sa gateway <gw-name>               # try to bring phase 1 up
test vpn ipsec-sa tunnel <tunnel-name>
less mp-log ikemgr.log                          # IKE negotiation details
```

## Packet capture (filters first, then stages)

```
debug dataplane packet-diag set filter match source 192.168.1.10 destination 10.10.10.5
debug dataplane packet-diag set filter on
debug dataplane packet-diag set capture stage receive file rx.pcap
debug dataplane packet-diag set capture stage drop file drop.pcap
debug dataplane packet-diag set capture on
...reproduce the issue...
debug dataplane packet-diag set capture off
debug dataplane packet-diag clear all          # always clean up
```
