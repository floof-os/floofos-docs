# Bridge Configuration

This section covers Layer 2 bridge domain configuration in FloofOS.

## Overview

Bridge domains in FloofOS provide Layer 2 switching functionality, allowing multiple interfaces to communicate at Layer 2. This is commonly used for:

- VXLAN overlay networks
- Transparent bridging between interfaces
- Layer 2 VPN services

---

## Show Bridge-Domain

Displays a summary of all bridge-domain instances or detailed view of a single bridge-domain.

### Syntax

```
show bridge-domain [<bridge-domain-id> [detail|int|arp]]
```

### Show All Bridge Domains

```
show bridge-domain
```

### Show Bridge Domain Details

```
show bridge-domain 100 detail
```

### Output Fields

| Field | Description |
|-------|-------------|
| ID | Bridge domain ID |
| Learning | MAC address learning enabled |
| U-Forwrd | Unknown unicast forwarding |
| UU-Flood | Unknown unicast flooding |
| Flooding | Broadcast/multicast flooding |
| ARP-Term | ARP termination enabled |
| BVI-Intf | Bridge Virtual Interface (if configured) |
| SHG | Split Horizon Group |

---

## Create Bridge-Domain

### Syntax

```
create bridge-domain <bridge-domain-id> [learn <0|1>] [forward <0|1>] [uu-flood <0|1>] [flood <0|1>] [arp-term <0|1>]
```

### Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `bridge-domain-id` | Unique bridge domain identifier (1-16777215) | Required |
| `learn` | Enable MAC learning | 1 (on) |
| `forward` | Enable unknown unicast forwarding | 1 (on) |
| `uu-flood` | Enable unknown unicast flooding | 1 (on) |
| `flood` | Enable broadcast/multicast flooding | 1 (on) |
| `arp-term` | Enable ARP termination | 0 (off) |

### Example

```
create bridge-domain 100
commit
```

### Example with Options

```
create bridge-domain 100 learn 1 flood 1 arp-term 0
commit
```

---

## Set Interface L2 Bridge

Adds an interface to a bridge domain for Layer 2 switching.

### Syntax

```
set interface l2 bridge <interface> <bridge-domain-id> [bvi] [shg <n>]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `interface` | Interface to add (e.g., GE12/0/0, GE12/0/0.100, vxlan_tunnel0) |
| `bridge-domain-id` | Bridge domain to join |
| `bvi` | Configure as Bridge Virtual Interface (for routing) |
| `shg` | Split Horizon Group number |

### Example: Add Physical Interface

```
set interface l2 bridge GE12/0/0 100
commit
```

### Example: Add VLAN Sub-Interface

```
set interface l2 bridge GE12/0/0.100 100
commit
```

### Example: Add VXLAN Tunnel

```
set interface l2 bridge vxlan_tunnel0 100
commit
```

---

## Remove Interface from Bridge

### Syntax

```
set interface l3 <interface>
```

This returns the interface to Layer 3 mode, removing it from the bridge domain.

### Example

```
set interface l3 GE12/0/0
commit
```

---

## Bridge Virtual Interface (BVI)

A BVI provides Layer 3 connectivity (IP routing) to a bridge domain.

### Create BVI

```
create loopback interface
set interface l2 bridge loop0 100 bvi
set interface state loop0 up
set interface ip address loop0 192.168.100.1/24
commit
```

### BVI Use Case

BVI allows the router to participate in a Layer 2 domain while also providing routing:

```
                    +------------------+
                    |    FloofOS       |
                    |                  |
                    |  BVI (loop0)     |
                    |  192.168.100.1   |
                    |        |         |
                    |  Bridge Domain   |
                    |      100         |
                    |   /        \     |
                    +--/----------\----+
                      /            \
               GE12/0/0.100    GE12/0/1.100
                   |                |
               Host A            Host B
            192.168.100.10    192.168.100.20
```

---

## L2 FIB (MAC Address Table)

View learned MAC addresses in bridge domains.

### Syntax

```
show l2fib all
show l2fib bd <bridge-domain-id>
```

### Clear L2 FIB

```
clear l2fib all
```

---

## Split Horizon Groups

Split Horizon Groups (SHG) prevent traffic received on one interface from being forwarded to other interfaces in the same group. This is useful for preventing loops in certain topologies.

### Example

```
set interface l2 bridge GE12/0/0.100 100 shg 1
set interface l2 bridge GE12/0/1.100 100 shg 1
set interface l2 bridge vxlan_tunnel0 100 shg 2
commit
```

In this example, GE12/0/0.100 and GE12/0/1.100 are in SHG 1, so traffic between them is blocked. Traffic to/from vxlan_tunnel0 (SHG 2) is allowed.

---

## Bridge Domain with VXLAN Example

### Topology

```
        Site A                              Site B
    +-----------+                      +-----------+
    |  FloofOS  |                      |  FloofOS  |
    |  10.0.0.1 |======================|  10.0.0.2 |
    +-----+-----+    VXLAN VNI 10000   +-----+-----+
          |                                  |
    +-----+-----+                      +-----+-----+
    |  Servers  |                      |  Servers  |
    | 10.1.0.x  |                      | 10.1.0.x  |
    +-----------+                      +-----------+
```

### Site A Configuration

```
# Create VXLAN tunnel
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 10000
set interface state vxlan_tunnel0 up

# Create bridge domain
create bridge-domain 100

# Add local interface
set interface l2 bridge GE12/0/1 100

# Add VXLAN tunnel
set interface l2 bridge vxlan_tunnel0 100

commit
```

---

## Delete Bridge-Domain

### Syntax

```
delete bridge-domain <bridge-domain-id>
```

!!! warning
    Remove all interfaces from the bridge domain before deleting it.

### Example

```
set interface l3 GE12/0/0.100
set interface l3 vxlan_tunnel0
delete bridge-domain 100
commit
```

---

## Troubleshooting

### No MAC Learning

1. Check bridge domain learning is enabled:
   ```
   show bridge-domain 100 detail
   ```

2. Verify interface is in bridge domain:
   ```
   show bridge-domain 100 detail
   ```

### Traffic Not Forwarding

1. Check L2 FIB for MAC entries:
   ```
   show l2fib bd 100
   ```

2. Verify flooding is enabled:
   ```
   show bridge-domain
   ```

3. Check interface states:
   ```
   show interface
   ```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show bridge-domain` | Display all bridge domains |
| `show bridge-domain <id> detail` | Display bridge domain details |
| `create bridge-domain <id>` | Create bridge domain |
| `delete bridge-domain <id>` | Delete bridge domain |
| `set interface l2 bridge <if> <bd>` | Add interface to bridge |
| `set interface l2 bridge <if> <bd> bvi` | Add BVI interface |
| `set interface l2 bridge <if> <bd> shg <n>` | Set split horizon group |
| `set interface l3 <if>` | Remove interface from bridge |
| `show l2fib all` | Display MAC address table |
| `clear l2fib all` | Clear MAC address table |
