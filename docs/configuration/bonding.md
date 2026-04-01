# Bonding Configuration

This section covers link aggregation (bonding) configuration in FloofOS for high availability and increased bandwidth.

## Overview

FloofOS supports link aggregation using various bonding modes for combining multiple physical interfaces into a single logical interface.

---

## Bonding Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `round-robin` | Transmit packets sequentially | Load balancing |
| `active-backup` | Only one active interface | High availability |
| `xor` | Hash-based load balancing | Static LAG |
| `broadcast` | Transmit on all interfaces | Special cases |
| `lacp` | 802.3ad Dynamic LAG | Standard LAG with switch |

---

## Create Bond Interface

### Syntax

```
create bond mode <mode> [load-balance <l2|l23|l34>]
```

### Example: LACP Bond (Recommended)

```
create bond mode lacp
```

### Example: Active-Backup Bond

```
create bond mode active-backup
```

### Example: LACP with Load Balance Mode

```
create bond mode lacp load-balance l34
```

---

## Add Members to Bond

### Syntax

```
bond add <bond-interface> <member-interface>
```

### Example

```
bond add BondEthernet0 GE12/0/0
bond add BondEthernet0 GE12/0/1
commit
```

---

## View Bond Status

### Show Bond Summary

```
show bond
```

Displays bond mode, load balance setting, active member count, and list of member interfaces.

### Show Bond Details

```
show bond details
```

Displays detailed per-member information including state, link speed, and LACP actor/partner state.

---

## Configure Bond Interface

### Bring Up

```
set interface state BondEthernet0 up
```

### Assign IP Address

```
set interface ip address BondEthernet0 192.168.1.1/24
```

### Create LCP Interface

```
lcp create BondEthernet0 host-if bond0
```

---

## Remove Member from Bond

### Syntax

```
bond del <bond-interface> <member-interface>
```

### Example

```
bond del BondEthernet0 GE12/0/1
commit
```

---

## Delete Bond Interface

```
delete bond BondEthernet0
commit
```

---

## Load Balancing Options

### Hash Modes

| Mode | Description | Recommendation |
|------|-------------|----------------|
| `l2` | MAC address hash | Layer 2 traffic |
| `l23` | MAC + VLAN hash | Mixed traffic |
| `l34` | IP + port hash | IP traffic (default) |

For most IP traffic scenarios, use `l34` for optimal distribution.

---

## Complete LACP Example

### FloofOS Configuration

```
# Create LACP bond
create bond mode lacp load-balance l34

# Add member interfaces
bond add BondEthernet0 GE12/0/0
bond add BondEthernet0 GE12/0/1

# Configure bond interface
set interface state BondEthernet0 up
set interface ip address BondEthernet0 192.168.1.1/24
lcp create BondEthernet0 host-if bond0

commit
```

### Switch Configuration (Cisco Example)

```
interface Port-channel1
 switchport mode trunk

interface GigabitEthernet0/1
 channel-group 1 mode active

interface GigabitEthernet0/2
 channel-group 1 mode active
```

### Switch Configuration (Juniper Example)

```
set interfaces ae0 aggregated-ether-options lacp active
set interfaces ge-0/0/0 ether-options 802.3ad ae0
set interfaces ge-0/0/1 ether-options 802.3ad ae0
```

---

## Active-Backup Example

For simple failover without switch configuration:

```
create bond mode active-backup
bond add BondEthernet0 GE12/0/0
bond add BondEthernet0 GE12/0/1
set interface state BondEthernet0 up
set interface ip address BondEthernet0 192.168.1.1/24
commit
```

---

## VLAN over Bond

Create VLAN sub-interfaces on bond:

```
create sub-interfaces BondEthernet0 100 dot1q 100
set interface state BondEthernet0.100 up
set interface ip address BondEthernet0.100 192.168.100.1/24
lcp create BondEthernet0.100 host-if bond0.100
commit
```

---

## Troubleshooting

### Bond Not Forming

1. Check member interfaces are up:
   ```
   show interface GE12/0/0
   show interface GE12/0/1
   ```

2. Check LACP status:
   ```
   show bond details
   ```

3. Verify switch configuration matches (LACP mode active/active or active/passive)

### Only One Active Member

1. Check LACP state:
   ```
   show bond details
   ```

2. Verify:
   - Both ports on switch are in same channel-group
   - LACP mode matches (active/active or active/passive)
   - Physical link is up on both ports

### Performance Issues

1. Verify load balance mode is appropriate:
   ```
   show bond
   ```

2. For IP traffic, use `l34` mode for best distribution

---

## Command Reference

| Command | Description |
|---------|-------------|
| `create bond mode <mode>` | Create bond interface |
| `create bond mode <mode> load-balance <lb>` | Create bond with load balance |
| `bond add <bond> <if>` | Add member to bond |
| `bond del <bond> <if>` | Remove member from bond |
| `delete bond <bond>` | Delete bond interface |
| `show bond` | Display bond summary |
| `show bond details` | Display bond details |
| `set interface state <bond> up` | Enable bond interface |
| `set interface ip address <bond> <ip>` | Assign IP to bond |
