# IPIP Tunnel Configuration

This section covers IP-in-IP (IPIP) tunnel configuration in FloofOS. IPIP provides simple IP encapsulation for creating point-to-point tunnels.

---

## Overview

IPIP tunneling encapsulates IPv4 or IPv6 packets inside another IP packet:

| Feature | Description |
|---------|-------------|
| **Simple Encapsulation** | Minimal overhead (20-byte IPv4 header) |
| **No Encryption** | Raw IP-in-IP (use with IPsec for security) |
| **Point-to-Point** | Connects two endpoints |
| **IPv4/IPv6 Support** | IPv4-in-IPv4, IPv6-in-IPv6, IPv4-in-IPv6, IPv6-in-IPv4 |

### Tunnel Modes

| Mode | Inner | Outer | Protocol |
|------|-------|-------|----------|
| **IPIP (4in4)** | IPv4 | IPv4 | Protocol 4 |
| **IP6IP6 (6in6)** | IPv6 | IPv6 | Protocol 41 |
| **6in4** | IPv6 | IPv4 | Protocol 41 |
| **4in6** | IPv4 | IPv6 | Protocol 4 |

---

## Create IPIP Tunnel

### IPv4-in-IPv4 Tunnel

**Syntax:**

```
create ipip tunnel src <local-ip> dst <remote-ip> [outer-table-id <vrf-id>]
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `src <local-ip>` | Local tunnel endpoint IP address |
| `dst <remote-ip>` | Remote tunnel endpoint IP address |
| `outer-table-id <vrf-id>` | VRF for outer IP lookup (default: 0) |

**Example:**

```
create ipip tunnel src 203.0.113.1 dst 198.51.100.1
commit
```

The command returns the created tunnel interface name (`ipip0`).

### IPv6-in-IPv6 Tunnel

**Syntax:**

```
create ipip tunnel src <local-ipv6> dst <remote-ipv6>
```

**Example:**

```
create ipip tunnel src 2001:db8:a::1 dst 2001:db8:b::1
commit
```

### 6in4 Tunnel (IPv6 over IPv4)

Used for IPv6 transition mechanisms.

```
create ipip tunnel src 203.0.113.1 dst 198.51.100.1

# Assign IPv6 address to tunnel
set interface ip address ipip0 2001:db8:tunnel::1/64
commit
```

### 4in6 Tunnel (IPv4 over IPv6)

```
create ipip tunnel src 2001:db8:a::1 dst 2001:db8:b::1

# Assign IPv4 address to tunnel
set interface ip address ipip0 10.0.0.1/30
commit
```

---

## Delete IPIP Tunnel

**Syntax:**

```
delete ipip tunnel <interface>
```

**Example:**

```
delete ipip tunnel ipip0
commit
```

---

## Configure IPIP Tunnel Interface

### Enable Interface

```
set interface state ipip0 up
commit
```

### Assign IP Address

```
set interface ip address ipip0 10.255.0.1/30
commit
```

### Set MTU

!!! note "MTU Consideration"
    IPIP adds 20 bytes (IPv4) or 40 bytes (IPv6) overhead. Adjust MTU accordingly.

```
# For IPv4-in-IPv4 over 1500 MTU network
set interface mtu 1480 ipip0
commit
```

---

## Show IPIP Tunnel Status

### Show All IPIP Tunnels

```
show ipip tunnel
```

Displays mode, source/destination endpoints, outer table ID, MTU, and admin/link state for each tunnel.

### Show Specific Tunnel

```
show ipip tunnel ipip0
```

### Show Interface Statistics

```
show interface ipip0
```

---

## Configuration Examples

### Example 1: Basic Site-to-Site IPIP Tunnel

Connect two sites with a simple IPIP tunnel.

**Site A (203.0.113.1):**

```
# Create IPIP tunnel
create ipip tunnel src 203.0.113.1 dst 198.51.100.1

# Configure tunnel interface
set interface state ipip0 up
set interface ip address ipip0 10.255.0.1/30
set interface mtu 1480 ipip0

# Add route to Site B LAN via tunnel
ip route add 192.168.2.0/24 via 10.255.0.2

commit
```

**Site B (198.51.100.1):**

```
# Create IPIP tunnel
create ipip tunnel src 198.51.100.1 dst 203.0.113.1

# Configure tunnel interface
set interface state ipip0 up
set interface ip address ipip0 10.255.0.2/30
set interface mtu 1480 ipip0

# Add route to Site A LAN via tunnel
ip route add 192.168.1.0/24 via 10.255.0.1

commit
```

### Example 2: IPv6 Transition with 6in4 Tunnel

Provide IPv6 connectivity over IPv4-only infrastructure.

**Site with IPv6 (203.0.113.1):**

```
# Create 6in4 tunnel
create ipip tunnel src 203.0.113.1 dst 198.51.100.1

# Configure IPv6 on tunnel
set interface state ipip0 up
set interface ip address ipip0 2001:db8:tunnel::1/64
set interface mtu 1480 ipip0

# Route IPv6 traffic through tunnel
ip route add 2001:db8:remote::/48 via 2001:db8:tunnel::2

commit
```

### Example 3: IPIP with VRF

Use IPIP tunnel in a specific VRF.

```
# Create VRF
ip table add 100

# Create IPIP tunnel with outer lookup in VRF 100
create ipip tunnel src 203.0.113.1 dst 198.51.100.1 outer-table-id 100

# Configure tunnel
set interface state ipip0 up
set interface ip address ipip0 10.255.0.1/30

# Put tunnel interface in VRF
set interface ip table ipip0 100

commit
```

### Example 4: IPIP with IPsec Protection

Secure IPIP tunnel with IPsec encryption.

```
# Create IPIP tunnel
create ipip tunnel src 203.0.113.1 dst 198.51.100.1

set interface state ipip0 up
set interface ip address ipip0 10.255.0.1/30

# Configure IPsec to protect tunnel traffic
# (See IPsec documentation for full configuration)
ipsec sa add 10 spi 1001 crypto-alg aes-cbc-256 crypto-key <key> integ-alg sha-256-128 integ-key <key> tunnel-src 203.0.113.1 tunnel-dst 198.51.100.1

commit
```

---

## IPIP vs Other Tunneling Protocols

| Feature | IPIP | GRE | VXLAN | WireGuard |
|---------|------|-----|-------|-----------|
| Overhead | 20B | 24B | 50B | ~60B |
| Encryption | No | No | No | Yes |
| Multicast | No | Yes | Yes | No |
| Multipoint | No | Yes | Yes | Yes |
| Simplicity | High | Medium | Medium | Medium |

---

## Best Practices

### When to Use IPIP

- **Simple point-to-point connections**
- **IPv6 transition (6in4, 4in6)**
- **Low overhead requirements**
- **Combined with IPsec for security**

### When NOT to Use IPIP

- **Multicast traffic needed** → Use GRE
- **Built-in encryption required** → Use WireGuard or IPsec
- **Layer 2 extension needed** → Use VXLAN

### MTU Guidelines

| Tunnel Type | Overhead | Recommended MTU |
|-------------|----------|-----------------|
| IPv4-in-IPv4 | 20 bytes | 1480 |
| IPv6-in-IPv6 | 40 bytes | 1460 |
| 6in4 | 20 bytes | 1480 |
| 4in6 | 40 bytes | 1460 |

---

## Troubleshooting

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Tunnel not working | Protocol 4/41 blocked | Check firewall allows IP protocol 4 or 41 |
| Packet drops | MTU too large | Reduce MTU to account for encapsulation |
| Asymmetric routing | Return path misconfigured | Verify routes on both endpoints |
| No connectivity | Tunnel endpoint unreachable | Verify underlay connectivity |

### Verification Commands

```
# Check tunnel status
show ipip tunnel

# Verify interface is up
show interface ipip0

# Check routes through tunnel
show ip fib 192.168.2.0/24
```

---

## Related Documentation

- [GRE Tunnels](gre.md)
- [IPsec VPN](ipsec.md)
- [Routing Configuration](routing.md)
- [VRF Configuration](routing.md#vrf-configuration)

## External References

- [FD.io VPP IPIP CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_vnet_ipip.html){:target="_blank"}
- [RFC 2003 - IP Encapsulation within IP](https://datatracker.ietf.org/doc/html/rfc2003){:target="_blank"}
- [RFC 2473 - Generic Packet Tunneling in IPv6](https://datatracker.ietf.org/doc/html/rfc2473){:target="_blank"}
