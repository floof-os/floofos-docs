# Operations

This section covers operational procedures for FloofOS.

## Overview

FloofOS provides comprehensive tools for system operations:

- **Commit and Rollback** - Configuration management and versioning
- **Backup and Restore** - Configuration preservation and recovery
- **Diagnostics** - Network troubleshooting tools

---

## Documentation

| Topic | Description |
|-------|-------------|
| [Commit and Rollback](commit-rollback.md) | Configuration versioning |
| [Backup and Restore](backup-restore.md) | Configuration backup |
| [Network Diagnostics](diagnostics.md) | Ping and traceroute |

---

## Quick Reference

### Configuration Management

```
# Commit changes
commit
commit comment <description>

# View commit history
show system commit

# Rollback
rollback <0-49>
system reboot
```

### Backup Operations

```
# Export backup
backup export <name>

# View backups
show backups

# Import backup
backup import <name>
```

### Diagnostics

```
# Connectivity test
ping <ip>

# Route tracing
traceroute <ip>
traceroute <ip> asn
```
