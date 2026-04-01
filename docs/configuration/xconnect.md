# Cross-Connect Configuration

This section covers Layer 2 Cross-Connect (L2 xconnect) and Layer 3 Cross-Connect (L3XC) configuration in FloofOS for direct interface-to-interface forwarding.

---

## Overview

Cross-connect provides direct packet forwarding between interfaces, bypassing normal switching or routing decisions:

| Feature | Layer | Description |
|---------|-------|-------------|
| **L2 xconnect** | Layer 2 | Direct L2 forwarding between two interfaces |
| **L3XC** | Layer 3 | Force traffic to specific next-hop, bypass FIB |

---

## Layer 2 Cross-Connect (L2 xconnect)

L2 xconnect creates a point-to-point Layer 2 connection between two interfaces. All traffic received on one interface is forwarded directly to the other interface without MAC learning or bridge domain membership.

### Use Cases

- **Pseudowire services** - Transparent L2 connectivity for customers
- **MPLS L2VPN termination** - Connect customer ports to MPLS pseudowires
- **Transparent bridging** - Simple two-port bridging without MAC table
- **Traffic mirroring** - Forward traffic between interfaces
- **Bypass scenarios** - Direct forwarding for specific ports

### L2 xconnect vs Bridge Domain

| Aspect | L2 xconnect | Bridge Domain |
|--------|-------------|---------------|
| Interfaces | Exactly 2 | Multiple |
| MAC Learning | No | Yes |
| Flooding | No | Yes |
| Performance | Higher | Lower |
| Use Case | Point-to-point | Multi-access |

---

## Configure L2 Cross-Connect

### Create L2 xconnect

**Syntax:**

```
set interface l2 xconnect <interface1> <interface2>
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `<interface1>` | First interface in the cross-connect pair |
| `<interface2>` | Second interface in the cross-connect pair |

**Example:**

```
set interface l2 xconnect GE1/0/1 GE1/0/2
commit
```

This creates a bidirectional L2 cross-connect between `GE1/0/1` and `GE1/0/2`.

!!! note "Bidirectional Configuration"
    L2 xconnect is bidirectional. You only need to configure it once - traffic flows both ways automatically.

### Remove L2 xconnect

To remove an interface from L2 xconnect and return it to Layer 3 mode:

**Syntax:**

```
set interface l3 <interface>
```

**Example:**

```
set interface l3 GE1/0/1
set interface l3 GE1/0/2
commit
```

---

## Show L2 xconnect Status

### Show All Cross-Connects

```
show l2patch
```

Displays all active L2 cross-connect pairs.

### Show Interface Mode

```
show mode [<interface>]
```

Displays the L2 mode (l3, xconnect, bridge) for each interface.

---

## L2 xconnect with VLAN Tag Rewrite

You can combine L2 xconnect with VLAN tag manipulation for advanced scenarios.

### Tag Rewrite Operations

| Operation | Description |
|-----------|-------------|
| `push` | Add VLAN tag |
| `pop` | Remove VLAN tag |
| `translate` | Replace VLAN tag |

### Configure Tag Rewrite

**Syntax:**

```
set interface l2 tag-rewrite <interface> {push|pop|translate} [dot1q <vlan-id>] [dot1ad <vlan-id>]
```

**Example - Pop outer tag:**

```
set interface l2 tag-rewrite GE1/0/1 pop 1
commit
```

**Example - Push VLAN tag:**

```
set interface l2 tag-rewrite GE1/0/1 push dot1q 100
commit
```

**Example - Translate VLAN:**

```
set interface l2 tag-rewrite GE1/0/1 translate 1-1 dot1q 200
commit
```

---

## Configuration Examples

### Example 1: Simple Two-Port Cross-Connect

Connect two customer ports transparently.

```
                Customer A                    Customer A
                  Site 1                       Site 2
                    │                            │
                    ▼                            ▼
              ┌──────────┐                ┌──────────┐
              │ GE1/0/1  │◄── xconnect ──►│ GE1/0/2  │
              └──────────┘                └──────────┘
                         FloofOS Router
```

**Configuration:**

```
# Create L2 xconnect between customer ports
set interface state GE1/0/1 up
set interface state GE1/0/2 up
set interface l2 xconnect GE1/0/1 GE1/0/2
commit

# Verify
show mode GE1/0/1 GE1/0/2
```

### Example 2: VLAN-to-VLAN Cross-Connect

Connect two different VLANs with tag translation.

```
            VLAN 100                         VLAN 200
                │                                │
                ▼                                ▼
          ┌──────────┐                    ┌──────────┐
          │GE1/0/1.100│◄── xconnect ────►│GE1/0/2.200│
          └──────────┘   (translate)      └──────────┘
```

**Configuration:**

```
# Create sub-interfaces
create sub-interfaces GE1/0/1 100
create sub-interfaces GE1/0/2 200

# Enable interfaces
set interface state GE1/0/1 up
set interface state GE1/0/2 up
set interface state GE1/0/1.100 up
set interface state GE1/0/2.200 up

# Create L2 xconnect
set interface l2 xconnect GE1/0/1.100 GE1/0/2.200

commit
```

### Example 3: Cross-Connect to VXLAN Tunnel

Extend L2 connectivity over VXLAN tunnel (pseudowire over IP).

```
         Local Port                      VXLAN Tunnel
             │                                │
             ▼                                ▼
       ┌──────────┐                    ┌─────────────┐
       │ GE1/0/1  │◄── xconnect ────►│vxlan_tunnel0│
       └──────────┘                    └─────────────┘
                                              │
                                              ▼
                                        Remote Site
```

**Configuration:**

```
# Create VXLAN tunnel
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 10001 instance 0

# Enable interfaces
set interface state GE1/0/1 up
set interface state vxlan_tunnel0 up

# Create L2 xconnect between local port and VXLAN tunnel
set interface l2 xconnect GE1/0/1 vxlan_tunnel0

commit

# Verify
show l2patch
```

### Example 4: Multi-Tenant Pseudowire Service

Multiple customers with isolated cross-connects.

```
# Customer A - VLAN 100 to VXLAN VNI 10100
create sub-interfaces GE1/0/1 100
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 10100 instance 0
set interface state GE1/0/1.100 up
set interface state vxlan_tunnel0 up
set interface l2 xconnect GE1/0/1.100 vxlan_tunnel0

# Customer B - VLAN 200 to VXLAN VNI 10200
create sub-interfaces GE1/0/1 200
create vxlan tunnel src 10.0.0.1 dst 10.0.0.3 vni 10200 instance 1
set interface state GE1/0/1.200 up
set interface state vxlan_tunnel1 up
set interface l2 xconnect GE1/0/1.200 vxlan_tunnel1

# Customer C - VLAN 300 to VXLAN VNI 10300
create sub-interfaces GE1/0/1 300
create vxlan tunnel src 10.0.0.1 dst 10.0.0.4 vni 10300 instance 2
set interface state GE1/0/1.300 up
set interface state vxlan_tunnel2 up
set interface l2 xconnect GE1/0/1.300 vxlan_tunnel2

commit

# Verify all cross-connects
show l2patch
```

### Example 5: Cross-Connect with QinQ

Double-tagged customer traffic.

```
# Create QinQ sub-interface (outer 100, inner 200)
create sub-interfaces GE1/0/1 100 inner-dot1q 200

# Create xconnect to another port
set interface state GE1/0/1.100.200 up
set interface state GE1/0/2 up
set interface l2 xconnect GE1/0/1.100.200 GE1/0/2

commit
```

---

## Layer 3 Cross-Connect (L3XC)

L3XC forces traffic matching specific criteria to a predetermined next-hop, bypassing the normal FIB lookup. This is useful for traffic steering and policy-based routing.

### Use Cases

- **Traffic steering** - Force specific traffic to alternate path
- **Service chaining** - Direct traffic through middleboxes
- **Policy-based routing** - Bypass normal routing for specific flows
- **Load balancing** - Distribute traffic across multiple paths

### Configure L3XC

**Syntax:**

```
l3xc add <interface> via <next-hop-ip> <outgoing-interface>
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `<interface>` | Input interface for L3XC |
| `<next-hop-ip>` | Forced next-hop IP address |
| `<outgoing-interface>` | Outgoing interface |

**Example:**

```
l3xc add GE1/0/1 via 10.0.0.2 GE1/0/2
commit
```

### Delete L3XC

**Syntax:**

```
l3xc del <interface>
```

**Example:**

```
l3xc del GE1/0/1
commit
```

### Show L3XC

```
show l3xc
```

---

## L3XC Configuration Examples

### Example 1: Force Traffic to Specific Gateway

Force all traffic from a specific interface to use alternate gateway.

```
# All traffic from GE1/0/1 goes to gateway 10.0.0.254 via GE1/0/2
l3xc add GE1/0/1 via 10.0.0.254 GE1/0/2
commit
```

### Example 2: Service Chaining Through Firewall

Direct traffic through a firewall appliance.

```
                    Firewall
                   10.1.1.1
                      │
              ┌───────┴───────┐
              │    GE2/0/0    │
              │               │
     Traffic  │   FloofOS     │
    ─────────►│               │
      GE1/0/1 │    L3XC      │
              └───────────────┘
```

**Configuration:**

```
# Force all traffic from GE1/0/1 to firewall at 10.1.1.1
l3xc add GE1/0/1 via 10.1.1.1 GE2/0/0
commit
```

---

## Best Practices

### L2 xconnect

| Recommendation | Description |
|----------------|-------------|
| **Interface State** | Ensure both interfaces are `up` before creating xconnect |
| **MTU Matching** | Use same MTU on both xconnect interfaces |
| **Monitoring** | Monitor interface counters for traffic verification |
| **Documentation** | Document all xconnect pairs for operations |

### L3XC

| Recommendation | Description |
|----------------|-------------|
| **Use Sparingly** | L3XC bypasses routing - use only when necessary |
| **Next-hop Reachable** | Ensure next-hop is directly connected or ARP-resolvable |
| **Monitor Traffic** | Verify traffic is flowing as expected |

---

## Troubleshooting

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| No traffic flow | Interface down | Check `show interface` status |
| One-way traffic | Asymmetric xconnect | Verify xconnect on both interfaces |
| MTU issues | MTU mismatch | Match MTU on both interfaces |
| Tagged traffic issues | Missing tag rewrite | Configure appropriate tag-rewrite |

### Verification Commands

```
# Check xconnect configuration
show l2patch
show mode

# Check interface status
show interface GE1/0/1
show interface GE1/0/2

# Check interface counters
show interface

# Check L3XC
show l3xc
```

---

## Related Documentation

- [Bridge Domains](bridge.md)
- [VLAN Configuration](vlan.md)
- [VXLAN Configuration](vxlan.md)
- [EVPN-VXLAN](evpn-vxlan.md)

## External References

- [FD.io VPP L2 CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_vnet_l2.html){:target="_blank"}
- [FD.io VPP L3XC CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_plugins_l3xc.html){:target="_blank"}
