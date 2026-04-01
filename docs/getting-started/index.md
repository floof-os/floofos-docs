# Getting Started

Welcome to FloofOS! This section guides you through initial setup and installation.

## Overview

FloofOS is a high-performance network operating system built on VPP (Vector Packet Processing) with BIRD2 for BGP routing.

---

## Quick Start Path

1. **[First Boot](first-boot.md)** - Initial login and interface configuration
2. **[Installation](installation.md)** - Install FloofOS to permanent storage
3. **[Quick Start](quick-start.md)** - Basic configuration guide

---

## Default Credentials

| Username | Password | Access |
|----------|----------|--------|
| `root` | `floofos` | Console only |
| `floofos` | `floofos` | Console and SSH |

!!! warning
    Change default passwords immediately after first login.

---

## System Requirements

FloofOS supports a wide range of hardware with network interfaces from 1G to 400G. The system auto-detects available NICs during first boot.

### Supported Interface Speeds

| Speed | VPP Naming | LCP Naming |
|-------|------------|------------|
| 1 Gbps | `GEX/Y/Z` | `ge-x-y-z` |
| 10 Gbps | `10GEX/Y/Z` | `10ge-x-y-z` |
| 25 Gbps | `25GEX/Y/Z` | `25ge-x-y-z` |
| 40 Gbps | `40GEX/Y/Z` | `40ge-x-y-z` |
| 100 Gbps | `100GEX/Y/Z` | `100ge-x-y-z` |
| 400 Gbps | `400GEX/Y/Z` | `400ge-x-y-z` |

---

## Next Steps

After completing installation:

- [Configure System Settings](../configuration/system.md)
- [Set Up User Accounts](../security/users.md)
- [Configure BGP](../configuration/bgp.md)
- [Enable Security Features](../security/index.md)
