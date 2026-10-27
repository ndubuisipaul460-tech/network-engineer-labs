# OSPF

## How it works

- **Link-state** protocol: every router floods link-state advertisements (LSAs), builds the same map of the area and runs **SPF (Dijkstra)** to pick the lowest-**cost** path.
- **Cost** = reference bandwidth ÷ interface bandwidth. Default reference is 100 Mbps, so FastEthernet and Gigabit both cost 1: raise it with `auto-cost reference-bandwidth 1000` on every router.
- **Areas**: area 0 is the backbone; other areas must connect to it. Single-area is fine for small networks.
- **Router ID**: highest loopback IP, else highest interface IP, unless set with `router-id`.
- **DR/BDR** are elected on multi-access segments (Ethernet) to reduce flooding; point-to-point links don't need them.

## Neighbour states

`Down → Init → 2-Way → ExStart → Exchange → Loading → Full`

Stuck in **ExStart/Exchange** often points to an MTU mismatch. Never getting past **Init** usually means hellos only arrive one way.

## Neighbours only form when these match

- Area ID
- Subnet and mask on the link
- Hello and dead timers
- Authentication (if used)
- MTU (for full adjacency)
- Area type (stub flags)

## Verify

```
show ip ospf neighbor
show ip ospf interface brief
show ip ospf interface g0/0        ! timers, cost, network type, area
show ip route ospf
show ip protocols                  ! router ID, networks, passive interfaces
```

## OSPF vs RIP vs BGP

| | RIP | OSPF | BGP |
|--|-----|------|-----|
| Type | Distance vector | Link state | Path vector |
| Metric | Hop count (max 15) | Cost | Path attributes (AS path, local pref…) |
| Convergence | Slow | Fast | Policy driven, slower |
| Used for | Small/legacy networks | Enterprise internal routing | Between organisations and ISPs |
