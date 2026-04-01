# VLAN Configuration

This section covers 802.1Q VLAN sub-interface and Q-in-Q (802.1ad) configuration in FloofOS.

## Overview

FloofOS supports VLAN tagging through sub-interfaces. Each sub-interface can be assigned a VLAN ID and configured independently with IP addresses, MTU, and LCP mappings.

---

## Create Sub-Interfaces

### Syntax

```
create sub-interfaces <interface> <subid> [dot1q <vlan>] [dot1ad <vlan>] [inner-vlan <vlan>] [exact-match] [default]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `interface` | Parent interface (e.g., GE12/0/0) |
| `subid` | Sub-interface ID (appears as interface.subid) |
| `dot1q vlan` | 802.1Q VLAN tag (1-4094) |
| `dot1ad vlan` | 802.1ad (Q-in-Q) outer VLAN tag |
| `inner-vlan vlan` | Q-in-Q inner VLAN tag |
| `exact-match` | Match only specified VLAN tags |
| `default` | Match untagged and unmatched tagged frames |

---

## Basic VLAN Sub-Interface

### Create VLAN Sub-Interface

```
create sub-interfaces GE12/0/0 100 dot1q 100
```

The sub-interface name format is `parent.subid`.

### Configure Sub-Interface

After creation, configure the sub-interface like any other interface:

```
set interface state GE12/0/0.100 up
set interface ip address GE12/0/0.100 192.168.100.1/24
commit
```

### Complete Example

```
create sub-interfaces GE12/0/0 100 dot1q 100
set interface state GE12/0/0.100 up
set interface ip address GE12/0/0.100 192.168.100.1/24
commit
```

!!! note "Automatic LCP Creation"
    When `lcp lcp-auto-subint on` is enabled (default), FloofOS automatically creates the corresponding Linux interface (e.g., `ge-12-0-0.100`) when you commit. No manual `lcp create` is required for sub-interfaces.

---

## Sub-Interface Matching Modes

### Exact Match

With `exact-match`, the sub-interface only matches packets with exactly the specified VLAN tags.

```
create sub-interfaces GE12/0/0 100 dot1q 100 exact-match
```

This matches packets with exactly one VLAN tag (100). Packets with VLAN 100 and an inner tag would NOT match.

### Default Sub-Interface

A `default` sub-interface matches any VLAN ID that does not match another configured sub-interface.

```
create sub-interfaces GE12/0/0 999 default
```

### Untagged Traffic

Create a sub-interface to handle untagged packets specifically:

```
create sub-interfaces GE12/0/0 1 dot1q 0
```

---

## Q-in-Q (Double VLAN Tagging)

Q-in-Q (802.1ad) allows stacking of two VLAN tags for service provider environments.

### Create Q-in-Q Sub-Interface

#### Syntax

```
create sub-interfaces <interface> <subid> dot1ad <outer-vlan> inner-vlan <inner-vlan>
```

#### Example

```
create sub-interfaces GE12/0/0 100200 dot1ad 100 inner-vlan 200
set interface state GE12/0/0.100200 up
set interface ip address GE12/0/0.100200 10.100.200.1/24
commit
```

### Q-in-Q with 802.1Q Outer Tag

For environments using 802.1Q (not 802.1ad) as the outer tag:

```
create sub-interfaces GE12/0/0 100200 dot1q 100 inner-vlan 200
```

---

## View Sub-Interfaces

### Show Interface

```
show interface
```

### Show Interface Addresses

```
show interface addr
```

---

## Delete Sub-Interface

```
delete sub-interface GE12/0/0.100
commit
```

---

## LCP with VLANs

With the default configuration (`lcp lcp-auto-subint on`), Linux interfaces for VLAN sub-interfaces are automatically created when you commit. This enables BIRD routing daemon and standard Linux tools to use the VLAN interface.

After commit, verify the LCP interface was created:

```
show lcp
```

---

## Multi-VLAN Configuration Example

### Configuration

```
# Ensure parent interface is up
set interface state GE12/0/0 up

# VLAN 100 - Users
create sub-interfaces GE12/0/0 100 dot1q 100
set interface state GE12/0/0.100 up
set interface ip address GE12/0/0.100 192.168.100.1/24

# VLAN 200 - Servers
create sub-interfaces GE12/0/0 200 dot1q 200
set interface state GE12/0/0.200 up
set interface ip address GE12/0/0.200 192.168.200.1/24

# VLAN 300 - Management
create sub-interfaces GE12/0/0 300 dot1q 300
set interface state GE12/0/0.300 up
set interface ip address GE12/0/0.300 192.168.0.1/24

commit
```

LCP interfaces (`ge-12-0-0.100`, `ge-12-0-0.200`, `ge-12-0-0.300`) are automatically created after commit.

---

## Q-in-Q Service Provider Example

Service provider using outer VLAN per customer, inner VLAN for customer services.

```
# Customer A - Data (S-VLAN 100, C-VLAN 10)
create sub-interfaces GE12/0/0 10010 dot1ad 100 inner-vlan 10
set interface state GE12/0/0.10010 up
set interface ip address GE12/0/0.10010 10.100.10.1/24

# Customer A - Voice (S-VLAN 100, C-VLAN 20)
create sub-interfaces GE12/0/0 10020 dot1ad 100 inner-vlan 20
set interface state GE12/0/0.10020 up
set interface ip address GE12/0/0.10020 10.100.20.1/24

# Customer B - Data (S-VLAN 200, C-VLAN 10)
create sub-interfaces GE12/0/0 20010 dot1ad 200 inner-vlan 10
set interface state GE12/0/0.20010 up
set interface ip address GE12/0/0.20010 10.200.10.1/24

commit
```

---

## Troubleshooting

### Sub-Interface Not Receiving Traffic

1. Check parent interface is up:
   ```
   show interface GE12/0/0
   ```

2. Verify VLAN tag matches switch configuration

3. Check sub-interface state:
   ```
   show interface GE12/0/0.100
   ```

### No IP Connectivity

1. Check IP assignment:
   ```
   show interface addr
   ```

2. Check routing:
   ```
   show ip fib
   ```

3. Check ARP:
   ```
   show ip neighbor
   ```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `create sub-interfaces <if> <id> dot1q <vlan>` | Create 802.1Q sub-interface |
| `create sub-interfaces <if> <id> dot1q <vlan> exact-match` | Create exact-match sub-interface |
| `create sub-interfaces <if> <id> dot1ad <vlan> inner-vlan <vlan>` | Create Q-in-Q sub-interface |
| `create sub-interfaces <if> <id> default` | Create default sub-interface |
| `delete sub-interface <if>.<id>` | Delete sub-interface |
| `set interface state <if>.<id> up` | Enable sub-interface |
| `set interface ip address <if>.<id> <ip>` | Assign IP to sub-interface |
| `show lcp` | Verify auto-created LCP interfaces |
