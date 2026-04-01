# System Logging

This section covers system logging and audit trail functionality in FloofOS.

## Overview

FloofOS maintains an audit log that records system events, commands executed, and configuration changes. Logging is **enabled by default**.

---

## Viewing Logs

### Basic Log Display

#### Syntax

```
show system logging
```

Displays the last 50 log entries. Each entry includes a timestamp, entry type (`[CMD]`, `[INFO]`, or `[COMMIT]`), username, and the command or event description.

---

## Log Entry Types

| Type | Description |
|------|-------------|
| `[CMD]` | Command executed by user |
| `[INFO]` | Informational event (login, service change) |
| `[COMMIT]` | Configuration commit |

---

## Filtering Logs

### Last N Entries

```
show system logging last <n>
```

Display the last N log entries.

### By User

```
show system logging user <username>
```

Filter logs to show only entries from a specific user.

### Configuration Changes Only

```
show system logging config
```

Display only configuration-related entries.

### Commit Logs Only

```
show system logging commit
```

Display only commit entries.

### Today's Entries

```
show system logging today
```

Display only log entries from today.

---

## Enabling/Disabling Logging

### Syntax

```
set system logging enable
set system logging disable
```

!!! warning
    Disabling logging removes audit trail functionality. Not recommended for production environments.

---

## Configuration Verification

Logging status appears in the running configuration:

```
show configuration
```

Look for `set system logging enable` or `set system logging disable` in the output.

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show system logging` | Display last 50 entries |
| `show system logging last <n>` | Display last N entries |
| `show system logging user <name>` | Filter by user |
| `show system logging config` | Configuration changes only |
| `show system logging commit` | Commit logs only |
| `show system logging today` | Today's entries |
| `set system logging enable` | Enable logging |
| `set system logging disable` | Disable logging |
