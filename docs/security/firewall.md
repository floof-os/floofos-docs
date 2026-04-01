# Firewall Configuration

FloofOS includes a stateful firewall based on nftables for control plane protection. This section covers firewall configuration and rule management.

## Overview

The firewall protects the control plane by filtering traffic destined to the router itself. By default, the firewall is **disabled** and must be explicitly enabled.

---

## Firewall State

### Enabling the Firewall

#### Syntax

```
set security firewall enable
commit
```

### Disabling the Firewall

```
set security firewall disable
commit
```

---

## Viewing Firewall Status

### Syntax

```
show security firewall status
```

Displays firewall state, default chain policies, active rule count, last modified timestamp, and control plane protection status.

### Status Fields

| Field | Description |
|-------|-------------|
| Firewall | Current state (enabled/disabled) |
| Policy | Default chain policies |
| Rules | Active rule count and breakdown |
| Last modified | Timestamp of last rule change |
| Control plane | Protection status |

---

## Default Rules

When enabled, the firewall includes default rules for essential network services:

### Viewing Rules

#### Syntax

```
show security firewall rules
```

Displays all active firewall rules with their name, protocol, source, destination port, and action.

### Default Rule Descriptions

| Rule Name | Protocol | Port | Purpose |
|-----------|----------|------|---------|
| `system-ssh` | TCP | 22 | Secure Shell access |
| `system-bgp` | TCP | 179 | BGP inbound sessions |
| `system-bgp-out` | TCP | 179 (source) | BGP outbound sessions |
| `system-bfd` | UDP | 3784-3785 | BFD inbound |
| `system-bfd-out` | UDP | 3784-3785 (source) | BFD outbound |
| `system-snmp` | UDP | 161 | SNMP queries |

!!! note
    Rules prefixed with `system-` are default FloofOS rules. These can be deleted and recreated as needed.

---

## Managing Firewall Rules

### Creating Rules

#### Syntax

```
set security firewall rule <name> protocol <tcp|udp> port <port> [src-address <ip/cidr>] action <accept|drop>
```

#### Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| `name` | Unique rule identifier | Yes |
| `protocol` | Transport protocol (tcp/udp) | Yes |
| `port` | Destination port or port range | Yes |
| `src-address` | Source IP address with CIDR notation | No |
| `action` | Rule action (accept/drop) | Yes |

#### Example

```
set security firewall rule api-lookingglass protocol tcp port 5000 action accept
commit
```

### Deleting Rules

#### Syntax

```
delete security firewall rule <name>
```

#### Example

```
delete security firewall rule system-snmp
commit
```

---

## Viewing Security Overview

### Syntax

```
show security
```

Displays a combined status summary of both the firewall (state, policy, rule count) and Fail2ban (state, thresholds, ban counts).

---

## Command Reference

| Command | Description |
|---------|-------------|
| `set security firewall enable` | Enable firewall |
| `set security firewall disable` | Disable firewall |
| `show security firewall status` | Display firewall status |
| `show security firewall rules` | List all firewall rules |
| `set security firewall rule <name> ...` | Create firewall rule |
| `delete security firewall rule <name>` | Delete firewall rule |
| `show security` | Display all security status |

---

## Best Practices

1. **Enable firewall in production** - Always enable the firewall on production systems
2. **Review default rules** - Assess whether all default rules are necessary for your deployment
3. **Use specific source addresses** - When possible, restrict rules by source IP/CIDR
4. **Document custom rules** - Use descriptive names for custom rules
5. **Commit changes** - Remember to commit after making firewall changes
