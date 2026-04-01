# Fail2ban Configuration

FloofOS includes Fail2ban for SSH brute-force protection. This section covers Fail2ban configuration and monitoring.

## Overview

Fail2ban monitors authentication logs and temporarily bans IP addresses that show malicious signs, such as repeated failed login attempts. By default, Fail2ban is **enabled** to protect SSH access.

!!! warning "Security Recommendation"
    Keep Fail2ban enabled on all production systems. Additionally, change all default passwords immediately after deployment.

---

## Fail2ban State

### Enabling Fail2ban

#### Syntax

```
set security fail2ban enable
```

### Disabling Fail2ban

```
set security fail2ban disable
```

---

## Viewing Fail2ban Status

### Status Overview

#### Syntax

```
show security fail2ban status
```

Displays Fail2ban state, max retry threshold, ban time, find time window, and current/total banned IP counts.

### Status Fields

| Field | Description | Default |
|-------|-------------|---------|
| Fail2ban | Service state | enabled |
| Max retry | Failed attempts before ban | 3 |
| Ban time | Ban duration in seconds | 600 (10 minutes) |
| Find time | Time window for counting failures | 600 (10 minutes) |
| Currently banned | Active bans | - |
| Total banned | Cumulative bans since boot | - |

---

## Viewing Banned IPs

### Syntax

```
show security fail2ban banned
```

Displays the SSH jail status including current/total failed attempts, monitored log files, and the list of currently banned IP addresses.

### Output Description

| Section | Description |
|---------|-------------|
| Filter | Log parsing statistics |
| Currently failed | Active failed attempts in find window |
| Total failed | Cumulative failed attempts |
| File list | Monitored log files |
| Actions | Ban enforcement statistics |
| Banned IP list | Currently banned IP addresses |

!!! note
    When brute-force attempts occur, banned IP addresses appear in the "Banned IP list" field.

---

## Configuration Parameters

### Maximum Retry Attempts

#### Syntax

```
set security fail2ban maxretry <number>
```

Sets the number of failed authentication attempts allowed before an IP is banned.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `number` | Maximum attempts before ban | 3 |

### Ban Duration

#### Syntax

```
set security fail2ban bantime <seconds>
```

Sets how long an IP address remains banned.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `seconds` | Ban duration in seconds | 600 |

---

## Command Reference

| Command | Description |
|---------|-------------|
| `set security fail2ban enable` | Enable Fail2ban |
| `set security fail2ban disable` | Disable Fail2ban |
| `set security fail2ban maxretry <n>` | Set max failed attempts |
| `set security fail2ban bantime <sec>` | Set ban duration |
| `show security fail2ban status` | Display Fail2ban status |
| `show security fail2ban banned` | Display banned IPs |
| `show security` | Display all security status |

---

## Best Practices

1. **Keep Fail2ban enabled** - Essential protection against brute-force attacks
2. **Change default passwords** - Fail2ban is a safety net, not a replacement for strong credentials
3. **Use SSH keys** - Configure SSH key authentication for additional security
4. **Monitor banned IPs** - Regularly check `show security fail2ban banned` for attack patterns
5. **Adjust thresholds** - Tune `maxretry` and `bantime` based on your security requirements
