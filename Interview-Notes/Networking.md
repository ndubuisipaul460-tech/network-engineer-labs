# Networking Interview Notes

**What happens when you type a URL into a browser?**
DNS resolves the name to an IP; the client ARPs for its gateway; a TCP handshake to port 443; TLS handshake (SNI, certificate check); the HTTP request; the response renders. Every step is a place where something can fail, so it's also a troubleshooting checklist.

**TCP vs UDP?**
TCP is connection-oriented and reliable (handshake, acknowledgements, retransmission). UDP has no connection and no guarantee; it's used where speed matters or the application handles loss (DNS, DHCP, SNMP, VoIP).

**How does a switch learn MAC addresses?**
From the source MAC of incoming frames. Unknown destinations are flooded in the VLAN; known ones are forwarded out one port.

**What's the purpose of STP?**
Stops Layer 2 loops by blocking redundant links, and unblocks them if the active path fails.

**What is a default gateway?**
The router a host sends traffic to when the destination is outside its own subnet.

**Static vs dynamic routing?**
Static is manual, predictable and doesn't adapt. Dynamic protocols (OSPF, BGP) share routes and adapt to failures automatically.

**What's an IPsec VPN phase 1 vs phase 2?**
Phase 1 (IKE) authenticates the peers and builds a secure channel. Phase 2 builds the IPsec SAs that encrypt user traffic. Both sides must agree on the settings.

**Stateful vs stateless firewall?**
A stateful firewall tracks connections and allows return traffic automatically. A stateless ACL checks every packet against rules on its own.

**What's NAT and why is it used?**
Translates private addresses to public ones (and back). It saves public IPv4 addresses and hides internal addressing.

**A user can't reach a server. Walk me through it.**
Scope (one user or many) → ping by IP → traceroute → DNS with nslookup → port test (`Test-NetConnection -Port`) → service on the server → firewall logs → recent changes. Escalate with the evidence.

**What's MTU, and what goes wrong with it?**
The largest packet a link carries (1500 bytes on Ethernet). Tunnels add headers, so large packets may need fragmenting; if ICMP "fragmentation needed" is blocked, large transfers hang while small pings work (a PMTUD black hole).

**How would you check if a firewall is blocking traffic?**
Traffic logs for denies, a policy-match test, drop counters, and a packet capture on both sides of the firewall.
