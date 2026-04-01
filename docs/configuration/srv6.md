# SRv6 Configuration

This section covers Segment Routing over IPv6 (SRv6) configuration in FloofOS.

## Overview

SRv6 is a modern source routing architecture that leverages IPv6 extension headers to encode routing information in the packet. Benefits include:

- Native IPv6 operation
- No additional encapsulation overhead
- Network programming with SRv6 functions
- Traffic Engineering
- Network slicing

---

## SRv6 Components

| Component | Description |
|-----------|-------------|
| **SID** | Segment Identifier (IPv6 address) |
| **Local SID** | SID processed locally by this node |
| **SRv6 Policy** | Ordered list of segments |
| **Steering** | Directing traffic into SRv6 policies |

---

## View SRv6 Configuration

### Show Local SIDs

```
show sr localsids
```

### Show SRv6 Policies

```
show sr policies
```

### Show Steering Policies

```
show sr steering-policies
```

---

## Configure Local SID

Local SIDs define how packets with matching destination SID are processed.

### Syntax

```
sr localsid address <sid> behavior <behavior> [parameters]
```

### SRv6 Behaviors

| Behavior | Description |
|----------|-------------|
| `end` | Endpoint (pop segment, forward) |
| `end.x <next-hop> <if>` | Endpoint with Layer 3 cross-connect |
| `end.dx4 <next-hop>` | Endpoint with decapsulation (IPv4) |
| `end.dx6 <next-hop>` | Endpoint with decapsulation (IPv6) |
| `end.dt4 <vrf>` | Endpoint with decap and VRF lookup (IPv4) |
| `end.dt6 <vrf>` | Endpoint with decap and VRF lookup (IPv6) |
| `end.dx2 <if>` | Endpoint with L2 cross-connect |

### Example: End (Transit Node)

```
sr localsid address fc00:1::1 behavior end
commit
```

### Example: End.DX4 (Decapsulate to IPv4)

```
sr localsid address fc00:1::100 behavior end.dx4 192.168.1.1 GE12/0/0
commit
```

### Example: End.DT4 (Decapsulate to IPv4 VRF)

```
sr localsid address fc00:1::200 behavior end.dt4 vrf-id 100
commit
```

### Delete Local SID

```
sr localsid del address fc00:1::1
commit
```

---

## Configure SRv6 Policy

SRv6 policies define a path through the network as a list of segments.

### Syntax

```
sr policy add bsid <bsid> next <sid> [next <sid> ...] [weight <n>] [fib-table <id>]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `bsid` | Binding SID (local identifier for policy) |
| `next` | Segment in the path (can be repeated) |
| `weight` | ECMP weight |
| `fib-table` | VRF table ID |

### Example: Simple Policy

```
sr policy add bsid fc00:1::999 next fc00:2::1 next fc00:3::1
commit
```

### Example: Multi-Path Policy

```
sr policy add bsid fc00:1::999 next fc00:2::1 next fc00:3::1 weight 1
sr policy mod bsid fc00:1::999 add sl next fc00:4::1 next fc00:3::1 weight 1
commit
```

### Delete Policy

```
sr policy del bsid fc00:1::999
commit
```

---

## Traffic Steering

Steer traffic into SRv6 policies based on destination prefix.

### Syntax

```
sr steer l3 <prefix> via bsid <bsid> [fib-table <id>]
```

### Example: Steer IPv6 Traffic

```
sr steer l3 2001:db8:100::/48 via bsid fc00:1::999
commit
```

### Example: Steer IPv4 Traffic

```
sr steer l3 192.168.100.0/24 via bsid fc00:1::999
commit
```

### Delete Steering

```
sr steer del l3 2001:db8:100::/48 via bsid fc00:1::999
commit
```

---

## SRv6 Network Example

### Topology

```
                   SID: fc00:2::1
                       +----+
          +------------|  R2  |-------------+
          |            +----+              |
          |                                 |
      +---+---+                         +---+---+
      |  R1   |                         |  R3   |
      |fc00:1::1                       |fc00:3::1
      +---+---+                         +---+---+
          |                                 |
     192.168.1.0/24                   192.168.100.0/24
```

### R1 Configuration (Ingress)

```
# Configure local SID
sr localsid address fc00:1::1 behavior end

# Create policy to reach R3 via R2
sr policy add bsid fc00:1::999 next fc00:2::1 next fc00:3::100

# Steer traffic to 192.168.100.0/24 into policy
sr steer l3 192.168.100.0/24 via bsid fc00:1::999

commit
```

### R2 Configuration (Transit)

```
# Configure local SID for transit
sr localsid address fc00:2::1 behavior end

commit
```

### R3 Configuration (Egress)

```
# Configure local SID with decapsulation
sr localsid address fc00:3::1 behavior end
sr localsid address fc00:3::100 behavior end.dx4 192.168.100.1 GE12/0/1

commit
```

---

## SRv6 Encapsulation Settings

### Set Encapsulation Source Address

```
set sr encaps source addr fc00:1::1
commit
```

### Show Encapsulation Settings

```
show sr encaps
```

---

## Troubleshooting

### Verify Local SIDs

```
show sr localsids
```

### Verify Policies

```
show sr policies
```

### Verify Steering

```
show sr steering-policies
```

### Check IPv6 FIB for SIDs

```
show ip6 fib fc00:1::1
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show sr localsids` | Display local SIDs |
| `show sr policies` | Display SRv6 policies |
| `show sr steering-policies` | Display steering policies |
| `sr localsid address <sid> behavior <b>` | Configure local SID |
| `sr localsid del address <sid>` | Delete local SID |
| `sr policy add bsid <bsid> next <sid>` | Create SRv6 policy |
| `sr policy del bsid <bsid>` | Delete SRv6 policy |
| `sr steer l3 <prefix> via bsid <bsid>` | Steer traffic to policy |
| `sr steer del l3 <prefix>` | Delete steering |
| `set sr encaps source addr <ip>` | Set encap source address |
