# Network Diagnostics

This section covers network troubleshooting tools available in FloofOS, including ping and traceroute.

## Overview

FloofOS provides built-in network diagnostic tools that operate directly through VPP. These commands can be executed from both operational mode and configuration mode.

---

## Ping

### Description

The `ping` command sends ICMP echo requests to test connectivity. Ping is handled directly by VPP for accurate dataplane testing.

### Syntax

```
ping {<ip-addr> | ipv4 <ip4-addr> | ipv6 <ip6-addr>} [ipv4 <ip4-addr> | ipv6 <ip6-addr>] [source <interface>] [size <pktsize:60>] [interval <sec:1>] [repeat <cnt:5>] [table-id <id:0>] [burst <count:1>] [verbose]
```

### Common Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `source <interface>` | Source interface | Auto |
| `size <pktsize>` | Packet size in bytes | 60 |
| `interval <sec>` | Interval between pings | 1 |
| `repeat <cnt>` | Number of pings | 5 |
| `table-id <id>` | VRF/routing table ID | 0 |
| `burst <count>` | Burst count | 1 |
| `verbose` | Verbose output | Off |

### Examples

```
# Basic ping
ping 172.16.32.1

# Ping from specific interface
ping 192.168.1.1 source GE1/0/0

# Custom count and size
ping 10.0.0.1 repeat 10 size 1400
```

### Error Messages

| Message | Meaning |
|---------|---------|
| `Failed: no egress interface` | No route to destination (missing gateway) |
| `Request timeout` | ICMP request timed out |

!!! tip "No Egress Interface"
    If you receive "no egress interface" errors, verify that a default gateway is configured. This can be set via BGP or static routes in the global-config section of BGP configuration.

---

## Traceroute

### Description

The `traceroute` command traces the path packets take to reach a destination.

### Syntax

```
traceroute <address> [asn | source <ip> | interface <interface>]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `address` | Destination IP address or hostname |
| `asn` | Display ASN information for each hop |
| `source <ip>` | Source IP address |
| `interface <interface>` | Source interface |

### Examples

```
# Basic traceroute
traceroute 172.16.32.1

# Traceroute with ASN display
traceroute 1.1.1.1 asn

# Traceroute from specific source
traceroute 8.8.8.8 source 10.0.0.1
```

!!! note "ASN Lookup Requirement"
    The `asn` option requires a working default route. Configure a gateway via BGP or static routes in the `global-config` section of BGP configuration.

---

## Troubleshooting Guide

### Connectivity Issues

| Symptom | Possible Cause | Resolution |
|---------|----------------|------------|
| "no egress interface" | No default route | Configure BGP or static gateway |
| "Network is unreachable" | No route to network | Verify routing configuration |
| Request timeout | Firewall/ACL blocking | Check firewall rules |
| High latency | Network congestion | Check interface traffic |

### Verifying Routes

Before running diagnostics, verify routing:

```
show configuration
```

Look for:
- Interface IP addresses
- BGP peer configuration
- Static routes in global-config

---

## Command Reference

| Command | Description |
|---------|-------------|
| `ping <ip>` | Basic connectivity test |
| `ping <ip> source <if>` | Ping from specific interface |
| `ping <ip> repeat <n>` | Ping with custom count |
| `traceroute <ip>` | Trace route to destination |
| `traceroute <ip> asn` | Trace with ASN display |
| `traceroute <ip> source <ip>` | Trace from specific source |

---

## Related Documentation

- [BGP Configuration](../configuration/bgp.md) - Configure routing and gateways
- [Traffic Monitoring](../monitoring/traffic.md) - Real-time traffic analysis
- [Interface Configuration](../configuration/interfaces.md) - Interface setup
