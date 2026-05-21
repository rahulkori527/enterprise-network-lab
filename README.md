# Enterprise Network Lab — Cisco Packet Tracer

A multi-site enterprise network simulation demonstrating BGP inter-site routing, VLAN segmentation, inter-VLAN routing, and ACL-based guest isolation across two sites connected via a simulated ISP.

## Topology

```
[HQ Site — AS 65001]              [Branch Site — AS 65002]
  VLAN 10: IT (192.168.10.0/24)      VLAN 30: Staff (10.30.0.0/24)
  VLAN 20: Mgmt (192.168.20.0/24)    VLAN 40: Guest (10.40.0.0/24)
       |                                     |
  L3 Switch (SW-HQ)                  L3 Switch (SW-Branch)
  3560-24PS                          3560-24PS
       |                                     |
  Router-HQ (AS 65001)           Router-Branch (AS 65002)
  Cisco 2911                         Cisco 2911
       |                                     |
       └──────── Router-ISP (AS 65000) ──────┘
                     Cisco 2911
```

## What's Implemented

- **VLANs** — HQ: VLAN 10 (IT), VLAN 20 (Management) / Branch: VLAN 30 (Staff), VLAN 40 (Guest)
- **Inter-VLAN Routing** — Layer 3 switching via SVIs on Cisco 3560 switches at both sites
- **Routed Uplinks** — `no switchport` on L3 switch uplink ports creates routed Layer 3 links to each router
- **eBGP** — External BGP between HQ (AS 65001), ISP (AS 65000), and Branch (AS 65002) for site-to-site routing
- **next-hop-self** — Configured on Router-ISP so BGP routes are reachable by both sites
- **Static Routes** — On routers pointing to L3 switch SVIs; on switches as default routes toward routers
- **ACL Guest Isolation** — VLAN 40 (Guest) blocked from VLAN 10 (IT) and VLAN 20 (Management), enforced inbound on SW-Branch VLAN 40 interface

## IP Addressing

| Device | Interface | IP Address | Purpose |
|--------|-----------|------------|---------|
| SW-HQ | Vlan10 | 192.168.10.1/24 | IT VLAN gateway |
| SW-HQ | Vlan20 | 192.168.20.1/24 | Mgmt VLAN gateway |
| SW-HQ | Gi0/1 | 192.168.1.2/24 | Routed uplink to Router-HQ |
| Router-HQ | Gi0/0 | 192.168.1.1/24 | Link to SW-HQ |
| Router-HQ | Gi0/1 | 203.0.113.1/30 | WAN link to ISP |
| Router-ISP | Gi0/0 | 203.0.113.2/30 | Link to Router-HQ |
| Router-ISP | Gi0/1 | 203.0.113.5/30 | Link to Router-Branch |
| Router-Branch | Gi0/0 | 10.0.0.1/24 | Link to SW-Branch |
| Router-Branch | Gi0/1 | 203.0.113.6/30 | WAN link to ISP |
| SW-Branch | Gi0/1 | 10.0.0.2/24 | Routed uplink to Router-Branch |
| SW-Branch | Vlan30 | 10.30.0.1/24 | Staff VLAN gateway |
| SW-Branch | Vlan40 | 10.40.0.1/24 | Guest VLAN gateway |

## Files

| File | Description |
|------|-------------|
| `enterprise-network-lab.pkt` | Cisco Packet Tracer topology file |
| `configs/router-hq.txt` | Running config for Router-HQ |
| `configs/router-branch.txt` | Running config for Router-Branch |
| `configs/router-isp.txt` | Running config for Router-ISP |
| `configs/sw-hq.txt` | Running config for SW-HQ |
| `configs/sw-branch.txt` | Running config for SW-Branch |
| `topology-diagram.png` | Network topology screenshot |
| `tests/ping-hq-to-branch.png` | PC-IT pinging PC-Staff — BGP routing verified |
| `tests/acl-guest-blocked.png` | PC-Guest blocked from IT subnet — ACL verified |
| `tests/inter-vlan-hq.png` | PC-IT pinging PC-Mgmt — inter-VLAN routing verified |
| `tests/inter-vlan-branch.png` | PC-Staff pinging PC-Guest — inter-VLAN routing verified |
| `tests/bgp-routes-router-hq.png` | show ip route on Router-HQ showing BGP routes |
| `tests/acl-guest-allowed.png` | PC-Guest pinging PC-Staff — partial access verified |

## Key Verification Commands

```bash
# Check BGP neighbors and routes
show ip bgp
show ip bgp summary
show ip route bgp

# Check VLANs and trunking
show vlan brief
show interfaces trunk
show ip interface brief

# Check routing table
show ip route

# Test connectivity
ping 10.30.0.10              # HQ to Branch Staff — should succeed
ping 192.168.10.10           # Branch to HQ IT — should succeed

# Test ACL
ping 192.168.10.10           # From PC-Guest — should fail (blocked)
ping 10.30.0.10              # From PC-Guest — should succeed (allowed)
```

## Design Decisions

- **BGP over static WAN routes** — scales easily when adding more sites; each site just peers with the ISP without touching other routers
- **next-hop-self on Router-ISP** — without this Router-HQ receives BGP routes but the next-hop IP is unreachable; next-hop-self replaces it with the ISP's own address
- **Routed uplinks with no switchport** — L3 switch uplink ports converted to routed interfaces so they can participate in IP routing between switch and router
- **Static routes on L3 switches** — default routes pointing to local routers so VLANs can forward traffic beyond their own site
- **ACL applied inbound on VLAN 40** — traffic checked as it enters from Guest devices; blocks IT and Management subnets while permitting everything else
- **/30 WAN subnets** — only 2 usable IPs needed per point-to-point WAN link; standard practice to avoid wasting address space

## Challenges and Lessons Learned

- BGP requires the advertised network to exist in the routing table first. Static routes on both L3 switches were needed to inject VLAN networks into the routers before BGP could advertise them.
- next-hop-self on Router-ISP was critical. Without it Router-HQ received BGP routes but traffic dropped silently because the next-hop was unreachable.
- The 3560 requires explicit `switchport trunk encapsulation dot1q` before trunk mode can be set, unlike newer switches that default to dot1q automatically.
- OSPF was initially configured on edge routers but not Router-ISP, so neighbor relationships never formed. BGP with next-hop-self and static routes handled all routing instead. In production, OSPF would run within each site between multiple internal routers with BGP only on WAN-facing edge routers.
- Routed ports (no switchport) on L3 switches were required for router-facing uplinks. Without this the switches had no IP-reachable path to the routers.
- Physical port assignments matter — a PC connected to the wrong switch port will not be in the correct VLAN regardless of how the config looks.

## Tools Used

- Cisco Packet Tracer 8.x
- Cisco IOS — 2911 Router, 3560-24PS Layer 3 Switch

## Related Skills

`TCP/IP` `BGP` `eBGP` `VLANs` `ACLs` `Inter-VLAN Routing` `Cisco IOS` `Layer 3 Switching` `Network Security` `Network Design` `Subnetting` `Static Routing` `WAN` `CCNA`
