# Palo Alto Troubleshooting Method

"Application X doesn't work through the firewall." Work through these steps in order and record the output of each in the ticket.

1. **Get the facts**: source IP, destination IP/FQDN, port or application, time it started, error the user sees. One user or many?
2. **Is the traffic reaching the firewall?**
   `show session all filter source <src> destination <dst>`. No session at all: look upstream (routing, switch, client).
3. **Which rule matches?**
   `test security-policy-match ...` and check the traffic log for `allow` / `deny` and the rule name.
4. **Is it being dropped, and why?**
   `show counter global filter delta yes severity drop` while reproducing the issue.
5. **NAT correct?**
   `test nat-policy-match ...` and `show session id <id>` (pre-NAT vs post-NAT addresses).
6. **App-ID or security profiles?**
   The session may start as `incomplete` or `insufficient-data`; check if the rule allows the identified app, and whether a threat or URL profile blocked it (threat/URL logs).
7. **Return path**: asymmetric routing makes sessions time out. Check the route back to the source.
8. **Capture it**: `packet-diag` with receive, transmit and drop stages to prove where the packet dies.
9. **Fix through change control**, verify with the user, document the root cause.

## Common root causes

| Symptom | Usual cause |
|---------|-------------|
| Deny in traffic log | Missing rule, wrong zone, rule order (a deny above the allow) |
| Session `incomplete` | Server not replying, or the return path doesn't go through the firewall |
| Works by IP, not by FQDN | DNS, or FQDN address object not resolving on the firewall |
| Tunnel down after a change | IKE crypto profile or pre-shared key mismatch, peer IP changed |
| Intermittent drops | Zone protection or DoS profile thresholds, session table full, HA failover |
