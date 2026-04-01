# MPLS Configuration

This section covers Multiprotocol Label Switching (MPLS) configuration in FloofOS.

## Overview

MPLS provides high-performance packet forwarding using short fixed-length labels instead of IP addresses. Common use cases:

- Traffic Engineering
- VPN services (L3VPN, L2VPN)
- Fast reroute protection
- QoS through EXP bits

---

## Enable MPLS on Interface

### Syntax

```
set interface mpls <interface> enable
```

### Example

```
set interface mpls GE12/0/0 enable
commit
```

### Disable MPLS

```
set interface mpls GE12/0/0 disable
commit
```

---

## View MPLS Configuration

### Show MPLS FIB

```
show mpls fib
```

### Show MPLS Tunnels

```
show mpls tunnel
```

### Show MPLS Interface

```
show mpls interface
```

---

## Local Label Binding

### Add Local Label

#### Syntax

```
mpls local-label add <label> via <next-hop> [<interface>] [out-labels <label> ...]
```

### Example: Simple Label Binding

```
mpls local-label add 16000 via 10.0.0.2 GE12/0/0
commit
```

### Example: Label Swap

```
mpls local-label add 16000 via 10.0.0.2 GE12/0/0 out-labels 16001
commit
```

### Example: Label Stack (Push)

```
mpls local-label add 16000 via 10.0.0.2 GE12/0/0 out-labels 16001 16002
commit
```

### Delete Local Label

```
mpls local-label del 16000
commit
```

---

## MPLS Tunnel

### Create MPLS Tunnel

```
create mpls tunnel via 10.0.0.2 GE12/0/0 out-labels 16000
set interface state mpls_tunnel0 up
commit
```

### Delete MPLS Tunnel

```
delete mpls tunnel mpls_tunnel0
commit
```

---

## IP to MPLS (Label Imposition)

Route IP traffic into MPLS tunnel:

```
# Create MPLS tunnel
create mpls tunnel via 10.0.0.2 GE12/0/0 out-labels 16000 16001
set interface state mpls_tunnel0 up

# Route traffic through tunnel
ip route add 192.168.100.0/24 via mpls_tunnel0

commit
```

---

## MPLS Label Ranges

| Range | Usage |
|-------|-------|
| 0-15 | Reserved (special labels) |
| 16-1048575 | Dynamic/Static labels |

### Special Labels

| Label | Name | Description |
|-------|------|-------------|
| 0 | IPv4 Explicit NULL | Pop and forward as IPv4 |
| 1 | Router Alert | Needs attention |
| 2 | IPv6 Explicit NULL | Pop and forward as IPv6 |
| 3 | Implicit NULL (PHP) | Penultimate Hop Popping |

---

## MPLS Static LSP Example

### Topology

```
    R1 (10.0.0.1)                           R2 (10.0.0.2)
        |                                       |
    +---+---+      MPLS Label: 16000       +---+---+
    |FloofOS|===========================|FloofOS|
    +---+---+                              +---+---+
        |                                       |
   192.168.1.0/24                         192.168.2.0/24
```

### R1 Configuration (Ingress)

```
# Enable MPLS on interface
set interface mpls GE12/0/0 enable

# Create tunnel with label
create mpls tunnel via 10.0.0.2 GE12/0/0 out-labels 16000
set interface state mpls_tunnel0 up

# Route to remote network via tunnel
ip route add 192.168.2.0/24 via mpls_tunnel0

commit
```

### R2 Configuration (Egress)

```
# Enable MPLS on interface
set interface mpls GE12/0/0 enable

# Bind local label (pop and forward)
mpls local-label add 16000 via ip4-lookup

commit
```

---

## Troubleshooting

### Check MPLS Interface Status

```
show mpls interface
```

### Check MPLS FIB

```
show mpls fib
```

### Check Packet Counters

```
show interface
show errors
```

### Trace MPLS Path

```
traceroute mpls 16000
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `set interface mpls <if> enable` | Enable MPLS on interface |
| `set interface mpls <if> disable` | Disable MPLS on interface |
| `show mpls fib` | Display MPLS FIB |
| `show mpls tunnel` | Display MPLS tunnels |
| `show mpls interface` | Display MPLS interfaces |
| `mpls local-label add <label> via <nh>` | Add local label binding |
| `mpls local-label add <label> via <nh> out-labels <label>` | Add label swap |
| `mpls local-label del <label>` | Delete local label |
| `create mpls tunnel via <nh> out-labels <labels>` | Create MPLS tunnel |
| `delete mpls tunnel <if>` | Delete MPLS tunnel |
