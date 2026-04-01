# Routing Configuration

This section covers static routing, routing tables, and IP configuration in FloofOS.

---

## Overview

FloofOS provides a dual routing architecture:

- **VPP FIB** — High-performance forwarding information base for data plane
- **BIRD** — Routing daemon for dynamic routing protocols (BGP, OSPF, etc.)

Routes learned by BIRD are automatically synced to VPP's FIB via the Linux Control Plane (LCP).

---

## Viewing Routes

### VPP FIB (Forwarding Table)

#### Show IPv4 FIB

```
show ip fib
```

#### Show Specific Route

```
show ip fib 10.0.0.0/8
```

#### Show IPv6 FIB

```
show ip6 fib
```

#### Show FIB Summary

```
show ip fib summary
```

---

## Static Routes

### Add IPv4 Static Route

#### Syntax

```
ip route add <prefix>/<length> via <next-hop> [<interface>] [table <vrf-id>]
```

#### Example

```
ip route add 10.0.0.0/8 via 172.16.32.1
commit
```

#### Via Specific Interface

```
ip route add 10.0.0.0/8 via 172.16.32.1 GE12/0/0
commit
```

### Add IPv6 Static Route

```
ip route add 2001:db8:100::/48 via 2001:db8:1::1
commit
```

### Delete Static Route

```
ip route del 10.0.0.0/8 via 172.16.32.1
commit
```

---

## Default Gateway

### IPv4 Default Route

```
ip route add 0.0.0.0/0 via 172.16.32.1
commit
```

### IPv6 Default Route

```
ip route add ::/0 via 2001:db8:1::1
commit
```

---

## IP Neighbor (ARP/NDP)

### Show ARP Table

```
show ip neighbor
```

#### Flags

| Flag | Description |
|------|-------------|
| D | Dynamic (learned) |
| S | Static |

### Show IPv6 Neighbors

```
show ip6 neighbor
```

### Add Static ARP Entry

```
set ip neighbor GE12/0/0 172.16.32.100 00:aa:bb:cc:dd:ee
commit
```

### Delete ARP Entry

```
set ip neighbor del GE12/0/0 172.16.32.100
commit
```

### Flush Dynamic Entries

```
ip neighbor flush
```

---

## ECMP (Equal-Cost Multi-Path)

### Add ECMP Routes

Add multiple next-hops for load balancing:

```
ip route add 10.0.0.0/8 via 172.16.32.2
ip route add 10.0.0.0/8 via 172.16.32.3
commit
```

### View ECMP Configuration

```
show ip fib 10.0.0.0/8
```

VPP displays multiple buckets in the load-balance DPO for ECMP routes.

---

## VRF (Virtual Routing and Forwarding)

### Create VRF Table

```
ip table add 100
```

### Assign Interface to VRF

```
set interface ip table GE12/0/1 100
commit
```

### Add Route to VRF

```
ip route add 10.0.0.0/8 via 192.168.2.1 table 100
commit
```

### View VRF FIB

```
show ip fib table 100
```

---

## BIRD Routing Integration

FloofOS uses BIRD for dynamic routing protocols. Routes from BIRD are automatically installed in VPP's FIB through Linux kernel route synchronization.

### View BIRD Routes

```
show route
```

### View Specific Route

```
show route for 10.0.0.0/8
```

### View BIRD Protocols

```
show protocols
```

### Reload BIRD Configuration

```
configure soft
```

---

## IPv6 Neighbor Discovery

### Show ND Configuration

```
show ip6 nd
```

### Configure RA Interval

```
ip6 nd GE12/0/0 ra-interval 60
```

### Configure ND Prefix

```
ip6 nd GE12/0/0 prefix 2001:db8:1::/64
```

### Suppress RA

```
ip6 nd GE12/0/0 ra-suppress
```

---

## Troubleshooting

### Route Not in FIB

1. Check if route exists in BIRD:
   ```
   show route for <prefix>
   ```

2. Verify LCP is operational:
   ```
   show lcp
   ```

3. Check BIRD kernel protocol:
   ```
   show protocols all kernel1
   ```

### Packet Drops

1. Check interface counters:
   ```
   show interface
   ```

2. Check for errors:
   ```
   show errors
   ```

3. Verify adjacency:
   ```
   show ip neighbor
   ```

### No ARP Resolution

1. Check interface state:
   ```
   show interface GE12/0/0
   ```

2. Check IP configuration:
   ```
   show interface addr
   ```

3. Verify physical connectivity

---

## Architecture

When you configure routes in FloofOS:

1. **VPP FIB** — Direct route adds go to VPP's forwarding table
2. **BIRD Integration** — Dynamic routes from BGP/OSPF go to BIRD, which updates Linux kernel routes
3. **LCP Sync** — Linux kernel routes are synchronized to VPP FIB via Linux Control Plane plugin

```
                    +------------------+
                    |      BIRD        |
                    |  (BGP, OSPF)     |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    |   Linux Kernel   |
                    |  Routing Table   |
                    +--------+---------+
                             |
                             v (LCP Sync)
                    +------------------+
                    |     VPP FIB      |
                    | (Data Plane)     |
                    +------------------+
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show ip fib` | Display IPv4 FIB |
| `show ip fib <prefix>` | Display specific route |
| `show ip fib table <id>` | Display VRF FIB |
| `show ip fib summary` | Display FIB summary |
| `show ip6 fib` | Display IPv6 FIB |
| `ip route add <prefix> via <nh>` | Add static route |
| `ip route del <prefix> via <nh>` | Delete static route |
| `ip table add <id>` | Create VRF table |
| `set interface ip table <if> <id>` | Assign interface to VRF |
| `show ip neighbor` | Display ARP table |
| `show ip6 neighbor` | Display NDP table |
| `set ip neighbor <if> <ip> <mac>` | Add static ARP |
| `set ip neighbor del <if> <ip>` | Delete ARP entry |
| `show route` | Display BIRD routes |
| `show protocols` | Display BIRD protocols |

---

## Additional Resources

- [VPP Layer 3 CLI](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_vnet_ip.html) - Complete IP CLI reference
- [VPP IP Neighbor CLI](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_vnet_ip-neighbor.html) - ARP/NDP commands
- [BIRD 2 Documentation](https://bird.network.cz/?get_doc) - BIRD routing daemon
- [BGP Configuration](bgp.md) - BGP with Pathvector
