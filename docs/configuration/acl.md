# ACL Configuration

This section covers Access Control List (ACL) configuration in FloofOS for traffic filtering.

## Overview

ACLs in FloofOS provide stateful packet filtering based on:

- Source/destination IP addresses
- Source/destination ports
- Protocol (TCP, UDP, ICMP, etc.)
- TCP flags

ACLs can be applied to interfaces for inbound or outbound traffic filtering.

---

## View ACLs

### Show All ACLs

```
show acl-plugin acl
```

Displays all configured ACLs with their index, name, and rules.

### Show Interface ACL Bindings

```
show acl-plugin interface
```

Displays which ACLs are bound to each interface for input and output.

### Show ACL Sessions

```
show acl-plugin sessions
```

---

## Create ACL

### Syntax

```
set acl-plugin acl <acl-index> <rules>
```

### Rule Format

```
[tag <tag>] <ipv4|ipv6> <permit|deny> src <prefix> dst <prefix> proto <n> sport <range> dport <range> [tcpflags <flags> mask <mask>]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `tag` | Optional rule name/tag |
| `ipv4/ipv6` | IP version |
| `permit/deny` | Action |
| `src` | Source prefix (x.x.x.x/n or any) |
| `dst` | Destination prefix |
| `proto` | Protocol number (6=TCP, 17=UDP, 1=ICMP) |
| `sport` | Source port or range (e.g., 0-65535, 80, 1024-65535) |
| `dport` | Destination port or range |
| `tcpflags` | TCP flags to match |
| `mask` | TCP flags mask |

### Example: Allow Web Traffic

```
set acl-plugin acl 1 ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 80, ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 443
commit
```

### Example: Allow SSH from Specific Network

```
set acl-plugin acl 2 ipv4 permit src 10.0.0.0/8 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 22
commit
```

### Example: Block All Except Established

```
set acl-plugin acl 3 ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 0-65535 tcpflags 0x12 mask 0x12, ipv4 deny src 0.0.0.0/0 dst 0.0.0.0/0 proto 0 sport 0-65535 dport 0-65535
commit
```

---

## Apply ACL to Interface

### Input ACL (Inbound Traffic)

```
set acl-plugin interface GE12/0/0 input acl 1
commit
```

### Output ACL (Outbound Traffic)

```
set acl-plugin interface GE12/0/0 output acl 2
commit
```

### Multiple ACLs

```
set acl-plugin interface GE12/0/0 input acl 1 acl 2
commit
```

---

## Remove ACL from Interface

```
set acl-plugin interface GE12/0/0 input acl del 1
commit
```

---

## Delete ACL

```
set acl-plugin acl del 1
commit
```

---

## Common ACL Examples

### Allow Ping (ICMP)

```
set acl-plugin acl 10 ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 1 sport 0-65535 dport 0-65535
```

### Allow DNS

```
set acl-plugin acl 11 ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 17 sport 0-65535 dport 53, ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 53
```

### Allow BGP

```
set acl-plugin acl 12 ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 179, ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 179 dport 0-65535
```

### Block RFC1918 from Internet Interface

```
set acl-plugin acl 20 ipv4 deny src 10.0.0.0/8 dst 0.0.0.0/0 proto 0 sport 0-65535 dport 0-65535, ipv4 deny src 172.16.0.0/12 dst 0.0.0.0/0 proto 0 sport 0-65535 dport 0-65535, ipv4 deny src 192.168.0.0/16 dst 0.0.0.0/0 proto 0 sport 0-65535 dport 0-65535, ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 0 sport 0-65535 dport 0-65535
```

---

## Firewall Use Case

### Topology

```
                    Internet
                        |
                   GE12/0/0 (WAN)
                   +----+----+
                   | FloofOS |
                   +----+----+
                   GE12/0/1 (LAN)
                        |
                   Internal Network
                   192.168.1.0/24
```

### Configuration

```
# ACL for WAN interface - allow established, block inbound
set acl-plugin acl 100 tag allow-established ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 6 sport 0-65535 dport 0-65535 tcpflags 0x10 mask 0x10, tag allow-related ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 17 sport 0-65535 dport 0-65535, tag allow-icmp ipv4 permit src 0.0.0.0/0 dst 0.0.0.0/0 proto 1 sport 0-65535 dport 0-65535, tag deny-all ipv4 deny src 0.0.0.0/0 dst 0.0.0.0/0 proto 0 sport 0-65535 dport 0-65535

# Apply to WAN interface
set acl-plugin interface GE12/0/0 input acl 100

commit
```

---

## Troubleshooting

### Check ACL Hit Counters

```
show acl-plugin acl
```

### Verify Interface Binding

```
show acl-plugin interface
```

### Check Session Table

```
show acl-plugin sessions
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show acl-plugin acl` | Display all ACLs |
| `show acl-plugin interface` | Display interface ACL bindings |
| `show acl-plugin sessions` | Display ACL sessions |
| `set acl-plugin acl <n> <rules>` | Create/update ACL |
| `set acl-plugin acl del <n>` | Delete ACL |
| `set acl-plugin interface <if> input acl <n>` | Apply input ACL |
| `set acl-plugin interface <if> output acl <n>` | Apply output ACL |
| `set acl-plugin interface <if> input acl del <n>` | Remove input ACL |
