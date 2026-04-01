# Commit and Rollback

This section covers configuration commit and rollback operations in FloofOS.

## Overview

FloofOS uses a commit-based configuration model. Changes made in configuration mode are staged until committed. The system maintains up to 50 commit points for rollback purposes.

---

## Committing Configuration

### Basic Commit

#### Syntax

```
commit
```

### Commit with Comment

#### Syntax

```
commit comment <comment>
```

#### Parameters

| Parameter | Description |
|-----------|-------------|
| `comment` | Descriptive label for the commit (no spaces, use dashes) |

#### Example

```
commit comment set-bgp-to-IXP
```

!!! tip "Comment Format"
    Comments cannot contain spaces. Use dashes (`-`) to separate words (e.g., `add-upstream-peer`).

---

## Viewing Commit History

### Syntax

```
show system commit
```

Displays a numbered list of the last 50 commit timestamps and optional comments, with `0` being the current (most recent) commit.

### History Fields

| Field | Description |
|-------|-------------|
| Index | Commit point number (0 = current) |
| Timestamp | Date and time of commit |
| `(current)` | Indicates active configuration |
| Comment | Optional commit comment in quotes |

!!! info "Commit Retention"
    FloofOS retains up to 50 commit points. Older commits are automatically purged as new commits are made.

---

## Rollback Operations

### Understanding Rollback

Rollback restores configuration to a previous commit point. After rollback, you must commit and reboot for changes to take effect.

### Rollback to Previous Commit

#### Syntax

```
rollback
```

or

```
rollback 0
```

Both commands are equivalent and rollback to the current configuration state.

### Rollback to Specific Commit

#### Syntax

```
rollback <0-49>
```

#### Example

```
rollback 5
```

### Post-Rollback Procedure

After rollback:

1. Review the loaded configuration: `show configuration`
2. Commit the rollback: `commit`
3. Reboot the system: `system reboot`

```
commit
system reboot
```

!!! warning "Reboot Required"
    VPP configuration changes from rollback require a system reboot to take effect. Hot reload is not currently supported for rollback operations.

---

## Alternative to Rollback

For minor changes, consider manually reverting specific settings using `delete` commands instead of full rollback:

| Original Command | Revert Command |
|-----------------|----------------|
| `set hostname core-router` | `set hostname floofos` |
| `set service ssh port 2234` | `delete service ssh port` |
| `set security firewall enable` | `set security firewall disable` |

For BGP configuration changes, edit the configuration file directly using `set protocol bgp`.

This approach avoids the reboot requirement of full rollback.

---

## Viewing Backup and Commit History

The `show backups` command displays both exported backups and commit history:

```
show backups
```

Displays exported backup files (name, timestamp, size) and the full rollback commit history.

---

## Command Reference

| Command | Description |
|---------|-------------|
| `commit` | Commit staged configuration |
| `commit comment <text>` | Commit with descriptive comment |
| `show system commit` | Display commit history |
| `rollback` | Rollback to previous state |
| `rollback <n>` | Rollback to specific commit point |
| `show backups` | Display backups and commit history |
| `system reboot` | Reboot system (required after rollback) |

---

## Best Practices

1. **Use commit comments** - Document significant changes for easier identification
2. **Review before rollback** - Check `show system commit` to identify correct rollback point
3. **Test changes incrementally** - Commit after each major change for granular rollback options
4. **Plan maintenance windows** - Rollback requires reboot, plan accordingly
5. **Consider manual revert** - For single changes, manual revert avoids reboot requirement
