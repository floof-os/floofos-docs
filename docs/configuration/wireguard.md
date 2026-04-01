# WireGuard VPN Configuration

This section covers WireGuard VPN tunnel configuration in FloofOS. WireGuard is a modern, high-performance VPN protocol that uses state-of-the-art cryptography.

---

## Overview

WireGuard provides:

| Feature | Description |
|---------|-------------|
| **Modern Cryptography** | Curve25519, ChaCha20, Poly1305, BLAKE2s |
| **High Performance** | Minimal overhead, kernel-level processing |
| **Simple Configuration** | Fewer parameters than IPsec |
| **Roaming Support** | Handles IP address changes seamlessly |
| **Cryptokey Routing** | Simple peer-to-IP mapping |

---

## Key Concepts

### WireGuard Components

| Component | Description |
|-----------|-------------|
| **Interface** | Virtual tunnel interface |
| **Private Key** | Local device's secret key |
| **Public Key** | Derived from private key, shared with peers |
| **Peer** | Remote endpoint configuration |
| **Allowed IPs** | IP ranges routed through the peer |
| **Endpoint** | Remote peer's IP:port (optional for responder) |

---

## Create WireGuard Interface

### Generate Keys

First, generate a key pair for the local device.

**Syntax:**

```
wireguard genkey
```

!!! warning "Key Security"
    Store the private key securely. Never share it. Only share the public key with peers.

### Create WireGuard Tunnel Interface

**Syntax:**

```
create wireguard tunnel src <local-ip> port <listen-port> private-key <base64-key>
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `src <local-ip>` | Local IP address for tunnel endpoint |
| `port <listen-port>` | UDP port to listen on (default: 51820) |
| `private-key <base64-key>` | Base64-encoded private key |

**Example:**

```
create wireguard tunnel src 203.0.113.1 port 51820 private-key <base64-private-key>
commit
```

The command returns the created interface name (`wg0`).

### Delete WireGuard Interface

**Syntax:**

```
delete wireguard tunnel <interface>
```

**Example:**

```
delete wireguard tunnel wg0
commit
```

---

## Configure WireGuard Peers

### Add Peer

**Syntax:**

```
wireguard peer add <interface> public-key <peer-public-key> endpoint <ip:port> allowed-ip <prefix> [persistent-keepalive <seconds>]
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `<interface>` | WireGuard interface (e.g., `wg0`) |
| `public-key <key>` | Peer's public key (base64) |
| `endpoint <ip:port>` | Peer's public IP and port |
| `allowed-ip <prefix>` | IP prefix(es) allowed from this peer |
| `persistent-keepalive <n>` | Keepalive interval in seconds |

**Example:**

```
wireguard peer add wg0 public-key <peer-public-key> endpoint 198.51.100.1:51820 allowed-ip 10.0.0.0/24 persistent-keepalive 25
commit
```

### Add Multiple Allowed IPs

You can specify multiple allowed IP ranges for a peer:

```
wireguard peer add wg0 public-key <peer-key> endpoint 198.51.100.1:51820 allowed-ip 10.0.0.0/24 allowed-ip 10.1.0.0/24 allowed-ip 192.168.0.0/16
commit
```

### Remove Peer

**Syntax:**

```
wireguard peer remove <interface> public-key <peer-public-key>
```

**Example:**

```
wireguard peer remove wg0 public-key <peer-public-key>
commit
```

---

## Assign IP Address to WireGuard Interface

After creating the WireGuard interface, assign an IP address:

```
set interface state wg0 up
set interface ip address wg0 10.0.0.1/24
commit
```

---

## Show WireGuard Status

### Show All WireGuard Interfaces

```
show wireguard interface
```

Displays the interface public key, listen port, local address, and connected peers with their transfer statistics.

### Show Specific Interface

```
show wireguard interface wg0
```

### Show WireGuard Peers

```
show wireguard peer
```

Displays peer public keys, endpoints, allowed IPs, handshake time, and transfer statistics.

---

## Configuration Examples

### Example 1: Site-to-Site VPN

Connect two sites via WireGuard VPN.

**Site A Configuration (203.0.113.1):**

```
# Generate keys
wireguard genkey

# Create WireGuard interface
create wireguard tunnel src 203.0.113.1 port 51820 private-key <site-a-private-key>

# Configure interface
set interface state wg0 up
set interface ip address wg0 10.255.255.1/30

# Add Site B as peer
wireguard peer add wg0 public-key <site-b-public-key> endpoint 198.51.100.1:51820 allowed-ip 10.255.255.2/32 allowed-ip 192.168.2.0/24 persistent-keepalive 25

# Add route to Site B network
ip route add 192.168.2.0/24 via 10.255.255.2

commit
```

**Site B Configuration (198.51.100.1):**

```
# Generate keys
wireguard genkey

# Create WireGuard interface
create wireguard tunnel src 198.51.100.1 port 51820 private-key <site-b-private-key>

# Configure interface
set interface state wg0 up
set interface ip address wg0 10.255.255.2/30

# Add Site A as peer
wireguard peer add wg0 public-key <site-a-public-key> endpoint 203.0.113.1:51820 allowed-ip 10.255.255.1/32 allowed-ip 192.168.1.0/24 persistent-keepalive 25

# Add route to Site A network
ip route add 192.168.1.0/24 via 10.255.255.1

commit
```

### Example 2: Hub-and-Spoke Topology

Central hub with multiple branch offices.

**Hub Configuration (203.0.113.1):**

```
# Create WireGuard interface on hub
create wireguard tunnel src 203.0.113.1 port 51820 private-key <hub-private-key>
set interface state wg0 up
set interface ip address wg0 10.255.0.1/24

# Add Branch 1
wireguard peer add wg0 public-key <branch1-public-key> endpoint 198.51.100.1:51820 allowed-ip 10.255.0.2/32 allowed-ip 192.168.10.0/24 persistent-keepalive 25

# Add Branch 2
wireguard peer add wg0 public-key <branch2-public-key> endpoint 198.51.100.2:51820 allowed-ip 10.255.0.3/32 allowed-ip 192.168.20.0/24 persistent-keepalive 25

# Add Branch 3
wireguard peer add wg0 public-key <branch3-public-key> endpoint 198.51.100.3:51820 allowed-ip 10.255.0.4/32 allowed-ip 192.168.30.0/24 persistent-keepalive 25

# Add routes
ip route add 192.168.10.0/24 via 10.255.0.2
ip route add 192.168.20.0/24 via 10.255.0.3
ip route add 192.168.30.0/24 via 10.255.0.4

commit
```

### Example 3: Road Warrior (Mobile Client)

Configure a WireGuard peer for mobile/roaming clients without fixed endpoint.

```
# Hub configuration for road warrior
# Note: No endpoint specified since client IP changes
wireguard peer add wg0 public-key <mobile-client-public-key> allowed-ip 10.255.0.100/32 persistent-keepalive 25

commit
```

!!! note "Road Warrior Notes"
    - No endpoint is specified for roaming clients
    - The peer will connect and establish the endpoint dynamically
    - Persistent keepalive keeps NAT mappings alive

---

## Best Practices

### Security Recommendations

1. **Key Management**
   - Generate unique key pairs for each device
   - Store private keys securely
   - Rotate keys periodically

2. **Firewall Configuration**
   - Allow UDP traffic on WireGuard port (default: 51820)
   - Restrict allowed IPs to necessary networks only

3. **Keepalive Settings**
   - Use persistent-keepalive for NAT traversal
   - 25 seconds is recommended for most NAT devices

### Performance Tuning

```
# Set MTU for optimal performance (typically 1420 for WireGuard)
set interface mtu 1420 wg0
commit
```

---

## Troubleshooting

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| No handshake | Firewall blocking UDP | Open UDP port 51820 |
| Asymmetric traffic | Incorrect allowed-ip | Verify allowed-ip on both peers |
| High latency | MTU issues | Reduce MTU to 1420 |
| Connection drops | NAT timeout | Enable persistent-keepalive |

### Verification Commands

```
# Check WireGuard interface status
show wireguard interface

# Verify peer connections
show wireguard peer

# Check interface statistics
show interface wg0
```

---

## Related Documentation

- [IPsec VPN Configuration](ipsec.md)
- [GRE Tunnels](gre.md)
- [Routing Configuration](routing.md)

## External References

- [FD.io VPP WireGuard CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_plugins_wireguard.html){:target="_blank"}
- [WireGuard Protocol](https://www.wireguard.com/){:target="_blank"}
