# Enterprise Network Lab — Cisco Packet Tracer

A multi-site enterprise network simulation demonstrating BGP inter-site routing, OSPF intra-site routing, VLAN segmentation, inter-VLAN routing, and ACL-based access control.

## Topology

```
[HQ Site — AS 65001]          [Branch Site — AS 65002]
  VLAN 10: IT (192.168.10.0/24)    VLAN 30: Staff (10.30.0.0/24)
  VLAN 20: Mgmt (192.168.20.0/24) VLAN 40: Guest (10.40.0.0/24)
       |                                  |
   L3 Switch (SW-HQ)             L3 Switch (SW-Branch)
       |                                  |
   Router-HQ ──── BGP (eBGP) ──── Router-Branch
       |                                  |
       └──────────── ISP ────────────────┘
```

## What's Implemented

- **VLANs** — HQ: VLAN 10 (IT), VLAN 20 (Management) / Branch: VLAN 30 (Staff), VLAN 40 (Guest)
- **Inter-VLAN Routing** — Layer 3 switching on both sites
- **OSPF** — Internal routing within each site (Area 0 at HQ, Area 1 at Branch)
- **eBGP** — External BGP between HQ (AS 65001) and Branch (AS 65002) for site-to-site connectivity
- **ACLs** — Guest VLAN (40) blocked from accessing IT VLAN (10) — guest isolation policy enforced at the router

## Files

| File | Description |
|------|-------------|
| `enterprise-network-lab.pkt` | Cisco Packet Tracer topology file |
| `configs/router-hq.txt` | Running config for Router-HQ |
| `configs/router-branch.txt` | Running config for Router-Branch |
| `configs/sw-hq.txt` | Running config for SW-HQ |
| `configs/sw-branch.txt` | Running config for SW-Branch |
| `topology-diagram.png` | Network topology diagram |

## Key Verification Commands

```bash
# Check BGP neighbors and routes
show ip bgp
show ip bgp summary
show ip route bgp

# Check OSPF
show ip ospf neighbor
show ip route ospf

# Check VLANs
show vlan brief
show interfaces trunk

# Test ACL
ping 192.168.10.x source 10.40.0.x  # Should fail — guest blocked from IT
ping 10.30.0.x source 192.168.10.x  # Should succeed — HQ to Branch staff
```

## Design Decisions

- **BGP chosen over a static WAN route** — demonstrates scalability for adding more sites without reconfiguring the core
- **OSPF per-site with area separation** — Area 0 at HQ, Area 1 at Branch, reflecting real enterprise multi-area OSPF design
- **Guest VLAN isolation via ACL** — simulates a real security requirement preventing guest users from reaching internal corporate resources
- **Layer 3 switching** — avoids routing all inter-VLAN traffic to the router, reducing latency and reflecting modern enterprise design

## Tools Used

- Cisco Packet Tracer 8.x
- Cisco IOS (2911 Router, 3650 Layer 3 Switch)

## Related Skills

`TCP/IP` `BGP` `OSPF` `VLANs` `ACLs` `Inter-VLAN Routing` `Cisco IOS` `Network Security` `Network Design`
