# System Configuration

This section covers system-level configuration including hostname, time settings, logging, and system information.

---

## Hostname Configuration

### Setting Hostname

#### Syntax

```
set hostname <hostname>
```

#### Example

```
set hostname core-router
commit
```

!!! note
    The hostname change takes effect immediately and is reflected in the CLI prompt.

---

## System Information

### Viewing System Status

#### Syntax

```
show system
```

Displays FloofOS version, hostname, uptime, kernel version, load average, and network stack versions (VPP, BIRD, Pathvector).

### Output Fields

| Field | Description |
|-------|-------------|
| FloofOS Version | Installed FloofOS version |
| Hostname | Current system hostname |
| Uptime | System uptime since last boot |
| Kernel | Linux kernel version |
| Load Average | System load (1, 5, 15 minute averages) |
| VPP | Vector Packet Processing version |
| BIRD | BIRD routing daemon version |
| Pathvector | Pathvector BGP automation version |

---

## System Resources

### Viewing Resource Usage

#### Syntax

```
show resource
```

Displays CPU model, core count, and usage percentage; memory totals (total, used, free, available, HugePages); VPP memory heap and worker threads; total interface count; and disk usage for the root filesystem.

---

## Time Configuration

### Viewing Current Time

#### Syntax

```
show system time
```

Displays the current system time, NTP sync status, and NTP synchronization state.

### Setting Time Zone

#### Syntax

```
set system time-zone <timezone>
```

#### Parameters

Timezone values follow the IANA Time Zone Database format. Reference: [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

#### Example

```
set system time-zone Europe/Berlin
commit
```

### Configuring NTP Server

#### Syntax

```
set system ntp server <hostname>
```

!!! tip "NTP Pool Servers"
    Find appropriate NTP servers for your region at [NTP Pool Project](https://www.ntppool.org/en/).

### Setting Manual Time

#### Syntax

```
set system clock <YYYY-MM-DD HH:MM:SS>
```

#### Example

```
set system clock 2026-01-16 12:00:00
```

!!! warning "Not Recommended"
    Manual time configuration is not recommended for production environments. Use NTP with a proper time zone instead.

---

## System Logging

### Viewing Logs

#### Syntax Options

| Command | Description |
|---------|-------------|
| `show system logging` | Display last 50 log entries |
| `show system logging last <n>` | Display last N entries |
| `show system logging user <username>` | Filter by user |
| `show system logging config` | Configuration changes only |
| `show system logging commit` | Commit logs only |
| `show system logging today` | Today's entries only |

Each log entry includes a timestamp, entry type, username, and the command or event description.

### Log Entry Types

| Type | Description |
|------|-------------|
| `[CMD]` | Command executed |
| `[INFO]` | Informational event |
| `[COMMIT]` | Configuration commit |

### Enabling/Disabling Logging

```
set system logging enable
set system logging disable
```

---

## System Reboot

### Syntax

```
system reboot
```

Prompts for confirmation before rebooting the system.

---

## Commit History

### Viewing Commit History

#### Syntax

```
show system commit
```

Displays a numbered list of the last 50 commit timestamps, with `0` being the current (most recent) commit.

!!! info "Commit Retention"
    FloofOS retains up to 50 commit points for rollback purposes. See [Commit and Rollback](../operations/commit-rollback.md) for details.

---

## Configuration Display

### Viewing Running Configuration

#### Syntax

```
show configuration
```

Displays the full running configuration in CLI format, including hostname, time zone, logging settings, interface configuration, protocol configuration, and security settings.

---

## Command Reference

| Command | Description |
|---------|-------------|
| `set hostname <name>` | Set system hostname |
| `show system` | Display system information |
| `show resource` | Display resource usage |
| `show system time` | Display current time and NTP status |
| `set system time-zone <tz>` | Set timezone |
| `set system ntp server <host>` | Configure NTP server |
| `set system clock <time>` | Set manual time (not recommended) |
| `show system logging` | Display audit logs |
| `set system logging enable/disable` | Enable/disable logging |
| `show system commit` | Display commit history |
| `show configuration` | Display running configuration |
| `system reboot` | Reboot the system |
