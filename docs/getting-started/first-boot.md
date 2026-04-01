# First Boot Configuration

This section describes the initial configuration process when booting FloofOS for the first time.

## Login Banner

Upon first boot, the system displays the following login prompt:

```
FloofOS Network Operating Sytem

floofos (tty1)
floofos login:
```

## Default Credentials

FloofOS ships with two default user accounts:

| Username | Password | Access Method |
|----------|----------|---------------|
| `root` | `floofos` | Console only (SSH disabled by default) |
| `floofos` | `floofos` | Console and SSH |

!!! warning "Security Notice"
    Change default passwords immediately after first login using the `create user` command. See [User Management](../security/users.md) for details.

---

## Interface Detection Wizard

When no VPP interface configuration exists, FloofOS automatically detects available network interfaces and presents a configuration wizard.

### Welcome Banner

After successful login, the system displays:

```
   ______          ___ ____  ____
  / __/ /__  ___  / _// __ \/ __/
 / _// / _ \/ _ \/ _// /_/ /\ \  
/_/ /_/\___/\___/_/  \____/___/  

Network Operating System
Copyright (c) 2026 | https://floofos.io

Welcome floofos (SSH from 172.16.32.1)
Sessions: 3 active | Last login: Thu Jan 15 18:34:06

Type '?' for available commands
```

### Interface Discovery

The wizard probes for available network interfaces:

```
Waiting for VPP...
Detected 1 network interface(s):

  VPP Name       Linux Name       Speed   NUMA   PCI Address
  ────────────   ──────────────   ─────   ────   ────────────────
  GE12/0/0       ge-12-0-0        1G      0      0000:00:12.0

Configure interfaces? [Y/n]: 
```

---

## Interface Naming Convention

FloofOS uses a structured naming scheme for network interfaces based on speed, slot position, and port index.

### VPP and Linux Control Plane (LCP) Names

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

| Component | Description |
|-----------|-------------|
| **X** | Card/Slot ID - Identifies the physical NIC or riser slot position |
| **Y** | NUMA Node - Indicates NUMA affinity (0 or 1 for dual-socket systems) |
| **Z** | Port Index - Sequential port number on the NIC |

---

## Configuring Interfaces

### Automatic Configuration (Recommended)

When prompted, enter `Y` to automatically create Linux Control Plane (LCP) interfaces:

```
Configure interfaces? [Y/n]: y

Configuring GE12/0/0 -> ge-12-0-0... done

Configured 1 interface(s)

Enter 'configure' mode and type 'commit' to save configuration
```

### Saving Configuration

After automatic configuration, enter configuration mode and commit changes:

```
floofos> conf
Entering configuration mode
floofos(config)# commit
Validating configuration...
Configuration committed successfully
floofos(config)# 
```

### Manual Configuration

Enter `N` at the wizard prompt to skip automatic LCP creation. You must then manually create LCP mappings using VPP commands.

!!! tip "Recommendation"
    Automatic configuration is recommended for most deployments. Manual configuration provides granular control for advanced use cases.

---

## Verifying Configuration

After committing, verify the interface configuration:

```
floofos(config)# show configuration 
```

The output displays the current running configuration including the newly created LCP interfaces.

---

## Next Steps

After completing first boot configuration:

1. **Install to Disk** - [Install FloofOS to permanent storage](installation.md)
2. **Change Default Passwords** - [User Management](../security/users.md)
3. **Configure Hostname** - [System Configuration](../configuration/system.md)
4. **Set Up Network** - [Interface Configuration](../configuration/interfaces.md)
