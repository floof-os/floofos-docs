# EVPN-VXLAN Configuration

This section covers Ethernet VPN (EVPN) with VXLAN configuration in FloofOS for building modern data center fabric overlays and Layer 2/Layer 3 extension across sites.

---

## Overview

EVPN-VXLAN combines two powerful technologies:

| Technology | Function |
|------------|----------|
| **EVPN** | Control plane - MAC/IP learning and distribution via BGP |
| **VXLAN** | Data plane - Layer 2 overlay encapsulation |

### Benefits

- **Scalable MAC Learning** - BGP-based, eliminates flood-and-learn
- **Optimal Traffic Paths** - Distributed anycast gateway
- **Multi-tenancy** - VNI-based isolation
- **Active-Active Multihoming** - No spanning tree limitations
- **Integrated L2/L3** - Bridging and routing in one fabric

---

## Architecture Components

### EVPN Terminology

| Term | Description |
|------|-------------|
| **VTEP** | VXLAN Tunnel Endpoint - encap/decap point |
| **VNI** | VXLAN Network Identifier (24-bit) |
| **EVI** | EVPN Instance |
| **Route Type 2** | MAC/IP Advertisement |
| **Route Type 3** | Inclusive Multicast Ethernet Tag |
| **Route Type 5** | IP Prefix Route |
| **Route Distinguisher (RD)** | Makes routes unique per VRF |
| **Route Target (RT)** | Controls route import/export |

### Topology Example

```
                    ┌─────────────────────┐
                    │    IP Underlay      │
                    │   (Spine/Leaf)      │
                    └─────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────┴────┐       ┌─────┴────┐      ┌─────┴────┐
   │  VTEP1  │       │  VTEP2   │      │  VTEP3   │
   │ Leaf-1  │       │  Leaf-2  │      │  Leaf-3  │
   └────┬────┘       └─────┬────┘      └─────┬────┘
        │                  │                  │
   ┌────┴────┐       ┌─────┴────┐      ┌─────┴────┐
   │ Hosts   │       │  Hosts   │      │  Hosts   │
   │ VNI 100 │       │ VNI 100  │      │ VNI 200  │
   └─────────┘       └──────────┘      └──────────┘
```

---

## VXLAN Configuration

### Create VXLAN Tunnel Interface

First, create the VXLAN interface for the overlay.

**Syntax:**

```
create vxlan tunnel src <local-vtep-ip> dst <remote-vtep-ip> vni <vni-id> [encap-vrf-id <vrf>] [decap-next {l2|node <name>}] [instance <n>]
```

**Example - Unicast VXLAN:**

```
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 100 instance 0
commit
```

### Create Multiple VXLAN Tunnels (Multicast BUM)

For BUM (Broadcast, Unknown unicast, Multicast) traffic with multicast underlay:

```
create vxlan tunnel src 10.0.0.1 group 239.1.1.100 vni 100 instance 0
commit
```

---

## Bridge Domain Configuration

### Create Bridge Domain for L2VNI

```
# Create bridge domain for VLAN 100
create bridge-domain 100

# Add VXLAN tunnel to bridge domain
set interface l2 bridge vxlan_tunnel0 100

# Add local interface to bridge domain
set interface l2 bridge GE1/0/1 100

commit
```

### Create BVI for IRB (Integrated Routing and Bridging)

```
# Create BVI interface
create loopback interface bvi100

# Add BVI to bridge domain
set interface l2 bridge bvi100 100 bvi

# Assign IP address (Anycast Gateway)
set interface ip address bvi100 192.168.100.1/24

commit
```

---

## EVPN Control Plane with BGP

EVPN uses BGP to distribute MAC and IP reachability information. Configure BGP with EVPN address family.

### BGP EVPN Configuration (BIRD)

FloofOS uses BIRD2 for BGP. Configure EVPN in `/etc/bird/bird.conf`:

```
# EVPN BGP configuration
protocol bgp evpn_spine1 {
    local 10.0.0.1 as 65001;
    neighbor 10.255.0.1 as 65000;

    ipv4 {
        import none;
        export none;
    };

    l2vpn evpn {
        import all;
        export all;
    };
}
```

### Route Distinguisher and Route Target

Configure RD and RT for EVPN instances:

```
# In BIRD configuration
template bgp evpn_template {
    l2vpn evpn {
        import filter {
            # Import routes with matching RT
            if (65001, 100) ~ bgp_ext_community then accept;
            reject;
        };
        export filter {
            # Export with RT
            bgp_ext_community.add((rt, 65001, 100));
            accept;
        };
    };
}
```

---

## Configuration Examples

### Example 1: Basic L2 EVPN-VXLAN

Two leaf switches extending VLAN 100 over VXLAN.

**Leaf-1 (VTEP IP: 10.0.0.1):**

```
# Create VXLAN tunnel to Leaf-2
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 100 instance 0

# Create bridge domain
create bridge-domain 100

# Add VXLAN tunnel to bridge domain
set interface l2 bridge vxlan_tunnel0 100

# Add local access port
set interface l2 bridge GE1/0/1 100

# Enable interfaces
set interface state vxlan_tunnel0 up
set interface state GE1/0/1 up

commit
```

**Leaf-2 (VTEP IP: 10.0.0.2):**

```
# Create VXLAN tunnel to Leaf-1
create vxlan tunnel src 10.0.0.2 dst 10.0.0.1 vni 100 instance 0

# Create bridge domain
create bridge-domain 100

# Add VXLAN tunnel to bridge domain
set interface l2 bridge vxlan_tunnel0 100

# Add local access port
set interface l2 bridge GE1/0/2 100

# Enable interfaces
set interface state vxlan_tunnel0 up
set interface state GE1/0/2 up

commit
```

### Example 2: L2/L3 EVPN-VXLAN with Anycast Gateway

Distributed routing with anycast gateway IP.

**Leaf-1:**

```
# Create VXLAN tunnel
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 100 instance 0

# Create bridge domain
create bridge-domain 100

# Create BVI for IRB
create loopback interface bvi100

# Configure bridge domain
set interface l2 bridge vxlan_tunnel0 100
set interface l2 bridge GE1/0/1 100
set interface l2 bridge bvi100 100 bvi

# Configure anycast gateway (same IP on all leaves)
set interface ip address bvi100 192.168.100.1/24

# Set anycast gateway MAC (same on all leaves)
set interface mac address bvi100 00:00:5e:00:01:01

# Enable interfaces
set interface state vxlan_tunnel0 up
set interface state GE1/0/1 up
set interface state bvi100 up

commit
```

!!! note "Anycast Gateway"
    Configure the **same IP and MAC address** on all leaf switches for the anycast gateway. This enables hosts to route locally without hair-pinning traffic.

### Example 3: Multi-VNI EVPN Fabric

Multiple VNIs for different tenants or VLANs.

```
# === VNI 100 (Tenant A - Web) ===
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 100 instance 0
create vxlan tunnel src 10.0.0.1 dst 10.0.0.3 vni 100 instance 1
create bridge-domain 100
set interface l2 bridge vxlan_tunnel0 100
set interface l2 bridge vxlan_tunnel1 100
set interface l2 bridge GE1/0/1.100 100

# === VNI 200 (Tenant A - DB) ===
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 200 instance 2
create vxlan tunnel src 10.0.0.1 dst 10.0.0.3 vni 200 instance 3
create bridge-domain 200
set interface l2 bridge vxlan_tunnel2 200
set interface l2 bridge vxlan_tunnel3 200
set interface l2 bridge GE1/0/1.200 200

# === VNI 300 (Tenant B) ===
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 300 instance 4
create bridge-domain 300
set interface l2 bridge vxlan_tunnel4 300
set interface l2 bridge GE1/0/2 300

# Create BVIs for routing
create loopback interface bvi100
create loopback interface bvi200
create loopback interface bvi300

set interface l2 bridge bvi100 100 bvi
set interface l2 bridge bvi200 200 bvi
set interface l2 bridge bvi300 300 bvi

set interface ip address bvi100 192.168.100.1/24
set interface ip address bvi200 192.168.200.1/24
set interface ip address bvi300 10.0.30.1/24

commit
```

### Example 4: EVPN Type-5 (IP Prefix Routes)

Advertise IP prefixes for inter-subnet routing.

```
# Configure L3VNI for IP routing
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 9999 instance 10
set interface state vxlan_tunnel10 up

# Create VRF for tenant
ip table add 1

# Associate L3VNI with VRF (in BIRD BGP config)
# Configure BGP to advertise Type-5 routes

# Add static route to be advertised
ip route add 172.16.0.0/16 via 192.168.100.100 table 1

commit
```

---

## Show Commands

### Show VXLAN Tunnels

```
show vxlan tunnel
```

### Show Bridge Domain

```
show bridge-domain 100 detail
```

### Show L2 FIB (MAC Table)

```
show l2fib all
```

### Show BGP EVPN Routes

```
birdc show route protocol evpn_spine1
```

---

## Best Practices

### Design Recommendations

| Aspect | Recommendation |
|--------|----------------|
| **VTEP Loopback** | Use dedicated loopback for VTEP source IP |
| **VNI Assignment** | Use consistent VNI scheme (e.g., VNI = VLAN ID + 10000) |
| **Anycast Gateway** | Same IP and MAC across all leaves |
| **Underlay** | eBGP or OSPF for VTEP reachability |
| **MTU** | Minimum 1550 bytes on underlay (VXLAN adds 50 bytes) |

### VNI Planning

| VNI Range | Purpose |
|-----------|---------|
| 10001-19999 | L2VNI (VLAN extension) |
| 90001-99999 | L3VNI (VRF routing) |

### MTU Considerations

```
# Set underlay interface MTU to accommodate VXLAN overhead
set interface mtu 9000 GE1/0/0
commit
```

---

## Troubleshooting

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| No MAC learning | VXLAN tunnel down | Check VTEP connectivity |
| BUM flooding not working | Multicast misconfigured | Verify multicast routing or use ingress replication |
| Asymmetric traffic | Anycast gateway mismatch | Verify same IP/MAC on all VTEPs |
| MTU issues | Underlay MTU too small | Increase MTU to 1550+ |

### Verification Commands

```
# Verify VXLAN tunnels
show vxlan tunnel

# Check bridge domain membership
show bridge-domain 100 detail

# Verify MAC learning
show l2fib all

# Test underlay connectivity
ping 10.0.0.2 source 10.0.0.1

# Check BGP EVPN session
birdc show protocols
```

---

## Related Documentation

- [VXLAN Configuration](vxlan.md)
- [Bridge Domains](bridge.md)
- [BGP Configuration](bgp.md)
- [Routing Configuration](routing.md)

## External References

- [VPP and EVPN (ipng.ch)](https://ipng.ch/s/articles/2025/07/12/vpp-and-evpn/vxlan-part-1/){:target="_blank"}
- [FD.io VPP VXLAN CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_plugins_vxlan.html){:target="_blank"}
- [RFC 8365 - A Network Virtualization Overlay Solution Using EVPN](https://datatracker.ietf.org/doc/html/rfc8365){:target="_blank"}
- [RFC 7432 - BGP MPLS-Based Ethernet VPN](https://datatracker.ietf.org/doc/html/rfc7432){:target="_blank"}
