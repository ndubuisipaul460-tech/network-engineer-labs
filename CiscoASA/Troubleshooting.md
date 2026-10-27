# Cisco ASA Troubleshooting Method

"Application X doesn't work through the firewall." Work through these steps in order and record the output of each in the ticket.

1. **Get the facts**: source IP, destination IP/FQDN, port, start time, error message. One user or many? Anything changed recently?
2. **Does the ASA know the route?**
   `show route <destination>`: the egress interface should be the one you expect.
3. **Simulate the packet**
   `packet-tracer input <ingress-nameif> tcp <src> <sport> <dst> <dport> detailed`
   It shows ALLOW or DROP and the exact phase: ACL, NAT, route lookup, inspection or VPN.
4. **Check the access-list**
   `show access-list <ACL>`: do hit counts increase while the user retries? Look for a deny above the permit (rule order).
5. **Check NAT**
   `show nat detail`, `show xlate`: is the traffic translated as expected? Is VPN traffic exempt from NAT?
6. **Is there a connection?**
   `show conn address <src>`: no connection means traffic isn't arriving or is dropped; a connection with no return bytes points to the server or the return path.
7. **Capture to prove it**
   A capture on the ingress **and** egress interfaces, plus `capture ASP type asp-drop all` and `show asp drop`.
8. **Read the logs**
   `show logging | include <src>`: look for 106023 (ACL deny), 305006 (NAT failed) and 313001 (ICMP denied).
9. **Fix through change control**, retest with the user, document the root cause.

## Common root causes

| Symptom | Usual cause |
|---------|-------------|
| `packet-tracer` DROP at ACCESS-LIST | Missing permit, wrong ACL applied, or a deny above the permit |
| Outside → inside fails, inside → outside works | Security levels: lower → higher needs an explicit ACL (and NAT for the public address) |
| Works by IP, not by name | DNS, not the firewall |
| Connection built, then times out | Server not answering, or asymmetric return path bypassing the ASA |
| VPN phase 1 not up | IKE policy or pre-shared key mismatch, peer IP changed, UDP 500/4500 blocked |
| VPN up, no traffic | Crypto ACLs not mirrored, or missing NAT exemption |
| Interfaces on the same security level can't talk | Needs `same-security-traffic permit inter-interface` |
