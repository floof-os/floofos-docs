# DHCP Configuration

This section covers Dynamic Host Configuration Protocol (DHCP) configuration in FloofOS, including DHCP client, DHCPv6, and DHCP proxy/relay functionality.

---

## Overview

FloofOS supports comprehensive DHCP functionality for both IPv4 and IPv6:

| Feature | Description |
|---------|-------------|
| **DHCP Client** | Obtain IP address dynamically on an interface |
| **DHCPv6 Client** | Obtain IPv6 address via DHCPv6 |
| **DHCPv6 PD** | Prefix Delegation for IPv6 |
| **DHCP Proxy** | Relay DHCP requests to a remote server |
| **DHCPv6 Proxy** | Relay DHCPv6 requests to a remote server |

---

## DHCP Client

### Enable DHCP Client

Configure an interface to obtain its IP address via DHCP.

**Syntax:**

```
set dhcp client intfc <interface> [hostname <name>]
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `intfc <interface>` | Interface to enable DHCP client |
| `hostname <name>` | Optional hostname to send in DHCP request |

**Example:**

```
set dhcp client intfc GE1/0/0 hostname floofos-router
commit
```

### Disable DHCP Client

```
set dhcp client del intfc <interface>
```

**Example:**

```
set dhcp client del intfc GE1/0/0
commit
```

### Show DHCP Client Status

Display DHCP client information and lease details.

**Syntax:**

```
show dhcp client [intfc <interface>] [verbose]
```

---

## DHCPv6 Client

### Enable DHCPv6 Client

Configure an interface to obtain IPv6 address via DHCPv6.

**Syntax:**

```
dhcp6 client <interface> [disable]
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `<interface>` | Interface to enable DHCPv6 client |
| `disable` | Disable DHCPv6 client on interface |

**Example - Enable:**

```
dhcp6 client GE1/0/0
commit
```

**Example - Disable:**

```
dhcp6 client GE1/0/0 disable
commit
```

### Show DHCPv6 Client Status

```
show dhcp6 clients
```

### Show DHCPv6 Addresses

```
show dhcp6 addresses
```

---

## DHCPv6 Prefix Delegation (PD)

DHCPv6 Prefix Delegation allows obtaining an IPv6 prefix from an upstream provider for distribution to downstream networks.

### Enable DHCPv6 PD Client

**Syntax:**

```
dhcp6 pd client <interface> prefix group <group-name>
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `<interface>` | Upstream interface to request prefix |
| `prefix group <group-name>` | Name of the prefix group |

**Example:**

```
dhcp6 pd client GE1/0/0 prefix group wan-prefix
commit
```

### Disable DHCPv6 PD Client

```
dhcp6 pd client GE1/0/0 disable
commit
```

### Assign Address from Prefix Group

Assign an IPv6 address to an interface using an obtained prefix.

**Syntax:**

```
set ip6 address <interface> prefix group <group-name> <suffix>
```

**Example:**

```
set ip6 address GE2/0/0 prefix group wan-prefix ::1/64
commit
```

This assigns the address `<delegated-prefix>::1/64` to `GE2/0/0`.

### Show IPv6 Prefix Delegation Status

```
show ip6 pd clients
show ip6 prefixes
```

---

## DHCP Proxy/Relay

DHCP Proxy (Relay) forwards DHCP requests from clients to a remote DHCP server, enabling centralized IP address management.

### Configure DHCP Proxy

**Syntax:**

```
set dhcp proxy server <server-ip> src-address <source-ip> [server-fib-id <n>] [rx-fib-id <n>]
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `server <server-ip>` | DHCP server IP address |
| `src-address <source-ip>` | Source IP for relayed packets |
| `server-fib-id <n>` | VRF/FIB ID where server resides (default: 0) |
| `rx-fib-id <n>` | VRF/FIB ID where clients reside (default: 0) |

**Example:**

```
set dhcp proxy server 10.0.0.100 src-address 192.168.1.1
commit
```

### Remove DHCP Proxy

```
set dhcp proxy del server 10.0.0.100 src-address 192.168.1.1
commit
```

### Show DHCP Proxy Configuration

```
show dhcp proxy
```

---

## DHCPv6 Proxy/Relay

### Configure DHCPv6 Proxy

**Syntax:**

```
set dhcpv6 proxy server <server-ipv6> src-address <source-ipv6> [server-fib-id <n>] [rx-fib-id <n>]
```

**Example:**

```
set dhcpv6 proxy server 2001:db8::100 src-address 2001:db8:1::1
commit
```

### Remove DHCPv6 Proxy

```
set dhcpv6 proxy del server 2001:db8::100 src-address 2001:db8:1::1
commit
```

### Show DHCPv6 Proxy Configuration

```
show dhcpv6 proxy
```

---

## DHCP Option 82 (Relay Agent Information)

Option 82 allows the relay agent to insert additional information about the client's location.

### Configure Option 82 VSS

**Syntax:**

```
set dhcp option-82 vss table <table-id> [oui <n> vpn-id <n> | vpn-ascii-id <text>]
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `table <table-id>` | VRF/Table ID |
| `oui <n>` | Organizationally Unique Identifier |
| `vpn-id <n>` | VPN Identifier |
| `vpn-ascii-id <text>` | ASCII VPN Identifier |

**Example:**

```
set dhcp option-82 vss table 1 vpn-ascii-id "CUSTOMER-A"
commit
```

### Remove Option 82 VSS

```
set dhcp option-82 vss del table 1
commit
```

### Show Option 82 Configuration

```
show dhcp vss
show dhcp option-82-address interface GE1/0/0
```

---

## DHCPv6 VSS Configuration

### Configure DHCPv6 VSS

**Syntax:**

```
set dhcpv6 vss table <table-id> [oui <n> vpn-id <n> | vpn-ascii-id <text>]
```

**Example:**

```
set dhcpv6 vss table 1 vpn-ascii-id "CUSTOMER-A-V6"
commit
```

### Show DHCPv6 VSS

```
show dhcpv6 vss
```

---

## Configuration Examples

### Example 1: WAN Interface with DHCP Client

```
# Configure WAN interface to obtain IP via DHCP
set dhcp client intfc GE1/0/0 hostname edge-router-01
commit

# Verify DHCP lease
show dhcp client
```

### Example 2: Dual-Stack with DHCPv6

```
# Enable both DHCP and DHCPv6 on WAN interface
set dhcp client intfc GE1/0/0 hostname edge-router-01
dhcp6 client GE1/0/0
commit

# Verify
show dhcp client
show dhcp6 clients
```

### Example 3: DHCP Relay for Multiple VLANs

```
# Configure DHCP relay for branch office
# DHCP server at 10.0.0.100 in data center

# VLAN 100 - Employee Network
set dhcp proxy server 10.0.0.100 src-address 192.168.100.1 rx-fib-id 0
commit

# Verify relay is working
show dhcp proxy
```

### Example 4: IPv6 Prefix Delegation for Home Gateway

```
# Request prefix from ISP
dhcp6 pd client GE1/0/0 prefix group isp-prefix
commit

# Assign addresses to LAN interfaces using delegated prefix
set ip6 address GE2/0/0 prefix group isp-prefix ::1/64
set ip6 address GE3/0/0 prefix group isp-prefix :1:0:0:0:1/64
commit

# Verify prefix delegation
show ip6 pd clients
show ip6 prefixes
```

---

## Troubleshooting

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| No DHCP lease obtained | Server unreachable | Check network connectivity to DHCP server |
| Relay not forwarding | Incorrect source address | Verify source address is reachable from server |
| DHCPv6 PD not working | ISP doesn't support PD | Contact ISP to enable prefix delegation |

### Debug Commands

```
# Check interface has DHCP enabled
show dhcp client intfc GE1/0/0 verbose

# Verify relay configuration
show dhcp proxy

# Check DHCPv6 state
show dhcp6 clients
```

---

## Related Documentation

- [Interfaces Configuration](interfaces.md)
- [Routing Configuration](routing.md)
- [VRF Configuration](routing.md#vrf-configuration)

## External References

- [FD.io VPP DHCP CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_plugins_dhcp.html){:target="_blank"}
