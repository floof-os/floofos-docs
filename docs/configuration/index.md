# Configuration

This section covers system and network configuration in FloofOS.

## Overview

FloofOS uses a commit-based configuration model. Changes are staged until committed, and up to 50 commit points are retained for rollback.

---
## Configuration Workflow

1. Enter configuration mode: `conf` or `configure`
2. Make changes using `set`, `create`, or `delete` commands
3. Review changes: `show configuration`
4. Apply changes: `commit`
---

## Documentation

### System

| Topic | Description |
|-------|-------------|
| [System](system.md) | Hostname, time, logging, resources |

### Layer 2

| Topic | Description |
|-------|-------------|
| [Interfaces](interfaces.md) | Physical interface configuration |
| [VLAN](vlan.md) | 802.1Q sub-interfaces and Q-in-Q |
| [Bridge](bridge.md) | Layer 2 bridge domains |
| [Cross-Connect](xconnect.md) | L2 xconnect and L3XC |
| [Bonding](bonding.md) | Link aggregation (LAG/LACP) |

### Layer 3

| Topic | Description |
|-------|-------------|
| [Routing](routing.md) | Static routes, FIB, ARP, VRF |
| [BGP](bgp.md) | BGP routing with Pathvector |

### Tunneling

| Topic | Description |
|-------|-------------|
| [VXLAN](vxlan.md) | VXLAN overlay tunnels |
| [EVPN-VXLAN](evpn-vxlan.md) | EVPN control plane with VXLAN data plane |
| [GRE](gre.md) | GRE tunnels |
| [IPIP](ipip.md) | IP-in-IP tunnels |
| [IPsec](ipsec.md) | IPsec VPN (manual keying) |
| [IKEv2](ikev2.md) | IKEv2 automatic key exchange |
| [WireGuard](wireguard.md) | Modern VPN with WireGuard |

### Services

| Topic | Description |
|-------|-------------|
| [DHCP](dhcp.md) | DHCP client, relay, and DHCPv6 |

### MPLS & Segment Routing

| Topic | Description |
|-------|-------------|
| [MPLS](mpls.md) | MPLS label switching |
| [SRv6](srv6.md) | Segment Routing over IPv6 |

### Security & QoS

| Topic | Description |
|-------|-------------|
| [ACL](acl.md) | Access Control Lists |
| [QoS](qos.md) | Quality of Service and Policing |

---

## Interface Naming Convention

FloofOS uses structured interface names based on speed and slot position:

| Speed | VPP Interface | LCP Interface |
|-------|---------------|---------------|
| 1 Gbps | `GEX/Y/Z` | `ge-x-y-z` |
| 10 Gbps | `10GEX/Y/Z` | `10ge-x-y-z` |
| 25 Gbps | `25GEX/Y/Z` | `25ge-x-y-z` |
| 40 Gbps | `40GEX/Y/Z` | `40ge-x-y-z` |
| 100 Gbps | `100GEX/Y/Z` | `100ge-x-y-z` |
| 400 Gbps | `400GEX/Y/Z` | `400ge-x-y-z` |

**Naming Components:**

- **X** = Card/Slot ID
- **Y** = NUMA Node (0 or 1)
- **Z** = Port Index

**Example:** `GE12/0/0` = 1 Gbps interface on slot 12, NUMA node 0, port 0

---

## Quick Reference

### System

```
set hostname <name>
set system time-zone <timezone>
show system
show configuration
```

### Interfaces

```
show interface
show interface addr
set interface state GE12/0/0 up
set interface ip address GE12/0/0 172.16.32.32/24
```

### Routing

```
show ip fib
ip route add 10.0.0.0/8 via 172.16.32.1
show ip neighbor
```

### L2

```
show bridge-domain
set interface l2 bridge GE12/0/0 100
show l2fib all
```

### Configuration Management

```
conf
commit
commit comment <description>
rollback <n>
```
