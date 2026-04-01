# Interface Configuration

This section covers network interface configuration in FloofOS. The documentation is organized into the following categories:

- [Basic Interface Commands](#basic-interface-commands) - Show and clear interface statistics
- [Hardware Interface Commands](#hardware-interface-commands) - Hardware details and diagnostics
- [Set Interface Commands](#set-interface-commands) - Configure interface properties
- [Create Interface Commands](#create-interface-commands) - Create virtual interfaces

For Layer 2 configuration, see:

- [VLAN Configuration](vlan.md) - 802.1Q VLAN sub-interfaces and Q-in-Q
- [Bridge Configuration](bridge.md) - Layer 2 bridge domains
- [Bonding Configuration](bonding.md) - Link aggregation (LAG/LACP)
- [VXLAN Configuration](vxlan.md) - VXLAN overlay tunnels

---

## Interface Naming Convention

FloofOS uses a structured naming scheme for network interfaces based on speed, slot position, and port index.

### Physical Interface Naming

| Speed | VPP Interface | LCP Interface (Linux) |
|-------|---------------|----------------------|
| 1 Gbps | `GEX/Y/Z` | `ge-x-y-z` |
| 2.5 Gbps | `2.5GEX/Y/Z` | `2.5ge-x-y-z` |
| 10 Gbps | `10GEX/Y/Z` | `10ge-x-y-z` |
| 25 Gbps | `25GEX/Y/Z` | `25ge-x-y-z` |
| 40 Gbps | `40GEX/Y/Z` | `40ge-x-y-z` |
| 50 Gbps | `50GEX/Y/Z` | `50ge-x-y-z` |
| 100 Gbps | `100GEX/Y/Z` | `100ge-x-y-z` |
| 400 Gbps | `400GEX/Y/Z` | `400ge-x-y-z` |

### Naming Components

| Component | Description | Example |
|-----------|-------------|---------|
| **X** | Card/Slot ID - Identifies physical NIC or riser slot position | `12` |
| **Y** | NUMA Node - Indicates NUMA affinity (0 or 1 for dual-socket systems) | `0` |
| **Z** | Port Index - Sequential port number on the NIC | `0` |

**Example:** `GE12/0/0` = 1 Gbps interface on slot 12, NUMA node 0, port 0

### Virtual Interface Types

| Interface Type | Naming Pattern | Example |
|---------------|----------------|---------|
| Loopback | `loop<N>` | `loop0` |
| Bond | `BondEthernet<N>` | `BondEthernet0` |
| VXLAN Tunnel | `vxlan_tunnel<N>` | `vxlan_tunnel0` |
| VLAN Sub-interface | `<parent>.<vlan-id>` | `GE12/0/0.100` |
| TAP | `tap<N>` | `tap0` |
| LCP Host | `host-<name>` | `host-ge-12-0-0` |

---

## Basic Interface Commands

Basic commands for displaying interface information and clearing statistics.

### Show Interface

Displays software interface information including counters and features.

#### Syntax

```
show interface [address|addr|features|feat] [<interface> [<interface> [..]]]
```

#### Show All Interfaces

```
show interface
```

#### Show Interface Addresses

```
show interface addr
```

#### Show RX Placement

```
show interface rx-placement
```

### Clear Interfaces

Clears the statistics for all interfaces (statistics associated with `show interface`).

#### Syntax

```
clear interfaces
```

---

## Hardware Interface Commands

Commands for displaying hardware details and driver information.

### Show Hardware-Interfaces

Displays detailed information about hardware interfaces including driver, speed, and MAC address.

#### Syntax

```
show hardware-interfaces [brief|verbose|detail] [<interface> [<interface> [..]]]
```

#### Parameters

| Parameter | Description |
|-----------|-------------|
| `brief` | Only show name, index, and state |
| `verbose` | Display additional attributes (default) |
| `detail` | Display all attributes and extended statistics |

### Clear Hardware-Interfaces

Clears extended statistics for all or specified interfaces.

#### Syntax

```
clear hardware-interfaces [<interface> [<interface> [..]]]
```

---

## Set Interface Commands

Commands for configuring interface properties.

### Set Interface State

Enables or disables an interface administratively.

#### Syntax

```
set interface state <interface> <up|down>
```

#### Example: Bring Interface Up

```
set interface state GE12/0/0 up
commit
```

#### Example: Bring Interface Down

```
set interface state GE12/0/0 down
commit
```

### Set Interface IP Address

Assigns IPv4 or IPv6 addresses to an interface.

#### Syntax

```
set interface ip address <interface> <address>/<prefix>
set interface ip address del <interface> <address>/<prefix>
```

#### Example: Add IPv4 Address

```
set interface ip address GE12/0/0 172.16.32.32/24
commit
```

#### Example: Add IPv6 Address

```
set interface ip address GE12/0/0 2001:db8::1/64
commit
```

#### Example: Multiple Addresses

```
set interface ip address GE12/0/0 192.168.1.1/24
set interface ip address GE12/0/0 192.168.2.1/24
set interface ip address GE12/0/0 2001:db8:1::1/64
set interface ip address GE12/0/0 2001:db8:2::1/64
commit
```

#### Example: Remove IP Address

```
set interface ip address del GE12/0/0 192.168.2.1/24
commit
```

### Set Interface MTU

Configures the Maximum Transmission Unit for an interface.

#### Syntax

```
set interface mtu <size> <interface>
```

#### Common MTU Values

| MTU | Use Case |
|-----|----------|
| 1500 | Standard Ethernet |
| 9000 | Jumbo frames (data center) |
| 9216 | Large jumbo frames |

#### Example

```
set interface mtu 9000 GE12/0/0
commit
```

!!! tip "MTU Recommendation"
    For data center deployments, use MTU 9000 for optimal throughput. Ensure all devices in the path support the same MTU.

### Set Interface Promiscuous

Enables or disables promiscuous mode for packet capture or monitoring.

#### Syntax

```
set interface promiscuous <on|off> <interface>
```

#### Example: Enable

```
set interface promiscuous on GE12/0/0
```

#### Example: Disable

```
set interface promiscuous off GE12/0/0
```

### Set Interface RX-Mode

Configures the receive mode for performance tuning.

#### Syntax

```
set interface rx-mode <interface> <polling|interrupt|adaptive>
```

#### Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `polling` | Continuous polling (default) | High-throughput, low-latency |
| `interrupt` | Interrupt-driven | Low-traffic, CPU conservation |
| `adaptive` | Automatic switching | Variable traffic patterns |

#### Example

```
set interface rx-mode GE12/0/0 polling
```

### Set Interface RX-Placement

Assigns receive queues to specific worker threads for performance optimization.

#### Syntax

```
set interface rx-placement <interface> queue <n> worker <n>
```

#### Example

```
set interface rx-placement GE12/0/0 queue 0 worker 1
set interface rx-placement GE12/0/0 queue 1 worker 2
commit
```

### Set Interface L2 Bridge

Assigns an interface to a Layer 2 bridge domain.

#### Syntax

```
set interface l2 bridge <interface> <bridge-domain-id> [bvi]
```

#### Example

```
set interface l2 bridge GE12/0/0 100
commit
```

See [Bridge Configuration](bridge.md) for complete bridge domain documentation.

---

## Create Interface Commands

Commands for creating virtual interfaces.

### Create Loopback Interface

Creates a loopback interface for router-id, management, or internal communication.

#### Syntax

```
create loopback interface
delete loopback interface <name>
```

#### Example

```
create loopback interface
set interface state loop0 up
set interface ip address loop0 10.255.255.1/32
commit
```

!!! tip "Router ID"
    Use a loopback interface address as your BGP router-id for stable peering.

### Create Sub-Interface

Creates VLAN sub-interfaces. See [VLAN Configuration](vlan.md) for detailed documentation.

#### Syntax

```
create sub-interfaces <interface> <subid> [dot1q <vlan>] [exact-match]
```

#### Example: Simple VLAN

```
create sub-interfaces GE12/0/0 100 dot1q 100
set interface state GE12/0/0.100 up
set interface ip address GE12/0/0.100 192.168.100.1/24
commit
```

!!! note "Automatic LCP"
    With `lcp lcp-auto-subint on` (default), the corresponding Linux interface is automatically created after commit. No manual `lcp create` required.

### Create TAP Interface

Creates TAP interfaces for Linux kernel connectivity.

#### Syntax

```
create tap host-if-name <linux-name>
```

#### Example

```
create tap host-if-name mgmt0
set interface state tap0 up
set interface ip address tap0 10.0.0.1/24
commit
```

---

## Linux Control Plane (LCP)

LCP creates Linux network interfaces that mirror VPP interfaces, enabling:

- Standard Linux tools (`ip`, `ping`, `traceroute`)
- BIRD routing daemon integration
- Management access through Linux networking

### Default LCP Settings

FloofOS uses these default LCP settings for automatic Linux interface creation:

```
lcp default netns dataplane
lcp lcp-sync on
lcp lcp-auto-subint on
```

| Setting | Description |
|---------|-------------|
| `lcp default netns dataplane` | LCP interfaces created in dataplane namespace |
| `lcp lcp-sync on` | Synchronize interface state between VPP and Linux |
| `lcp lcp-auto-subint on` | Automatically create LCP for sub-interfaces |

### Show LCP

```
show lcp
```

Displays the mapping between VPP interfaces and their corresponding Linux host interfaces.

### Create LCP Interface

For physical interfaces, LCP is typically created during first-boot wizard. For manual creation:

#### Syntax

```
lcp create <vpp-interface> host-if <linux-name>
```

#### Example

```
lcp create GE12/0/0 host-if ge-12-0-0
commit
```

!!! note "Automatic LCP Creation"
    - **First boot:** LCP interfaces are automatically created when you accept the interface wizard
    - **Sub-interfaces:** With `lcp lcp-auto-subint on` (default), LCP is automatically created for sub-interfaces after commit

### Delete LCP Interface

#### Syntax

```
lcp delete <vpp-interface>
```

#### Example

```
lcp delete GE12/0/0
commit
```

### LCP Configuration Commands

| Command | Description |
|---------|-------------|
| `lcp default netns <name>` | Set default network namespace |
| `lcp lcp-sync <on/off>` | Enable/disable LCP sync |
| `lcp lcp-auto-subint <on/off>` | Auto-create sub-interface LCP |

---

## Troubleshooting

### Interface Not Appearing

1. Verify interface detection during boot:
   ```
   show hardware-interfaces
   ```

2. Check VPP logs for errors:
   ```
   start shell
   ```
   Then run `journalctl -u vpp -n 50` in the shell.

### Interface Down

1. Check physical link status:
   ```
   show hardware-interfaces GE12/0/0
   ```

2. Check administrative state:
   ```
   show interface GE12/0/0
   ```

3. Verify cable connection and remote port status

### No Traffic

1. Check interface counters:
   ```
   show interface
   ```

2. Verify IP configuration:
   ```
   show interface addr
   ```

3. Check routing table:
   ```
   show ip fib
   ```

4. Verify LCP interface (for BIRD):
   ```
   show lcp
   ```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show interface` | Display interface status and counters |
| `show interface addr` | Display interface IP addresses |
| `show interface rx-placement` | Display RX queue placement |
| `show hardware-interfaces` | Display hardware details |
| `clear interfaces` | Clear interface counters |
| `clear hardware-interfaces` | Clear hardware counters |
| `set interface state <if> up/down` | Set administrative state |
| `set interface ip address <if> <ip>` | Assign IP address |
| `set interface mtu <size> <if>` | Set MTU |
| `set interface promiscuous on/off <if>` | Set promiscuous mode |
| `set interface rx-mode <if> <mode>` | Set receive mode |
| `set interface rx-placement <if> queue <n> worker <n>` | Set RX placement |
| `set interface l2 bridge <if> <bd>` | Add to bridge domain |
| `create loopback interface` | Create loopback |
| `create tap host-if-name <name>` | Create TAP interface |
| `lcp create <if> host-if <name>` | Create LCP interface |
| `show lcp` | Display LCP interfaces |
