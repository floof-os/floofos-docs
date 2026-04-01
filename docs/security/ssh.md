# SSH Service Configuration

This section covers SSH server configuration in FloofOS, including port settings, access control, and key management.

## Overview

SSH is enabled by default on FloofOS for remote management. The service runs in the dataplane namespace and supports both password and public key authentication.

---

## Viewing SSH Status

### Syntax

```
show service ssh
```

Displays the current SSH configuration including port, listen address, root login permission, password authentication state, public key authentication state, and service state.

### Status Fields

| Field | Description | Default |
|-------|-------------|---------|
| Port | SSH listening port | 22 |
| Listen Address | Bound IP addresses | 0.0.0.0, :: (all interfaces) |
| Root Login | Root SSH access permission | disabled |
| Password Auth | Password authentication | enabled |
| Pubkey Auth | Public key authentication | enabled |
| Service | Service state | active |

---

## Port Configuration

### Changing SSH Port

#### Syntax

```
set service ssh port <1-65535>
```

#### Example

```
set service ssh port 2234
commit
```

### Resetting to Default Port

#### Syntax

```
delete service ssh port
```

---

## Root Login Access

By default, the `root` user cannot authenticate via SSH. This can be modified if required.

### Enabling Root SSH Access

#### Syntax

```
set service ssh root-login enable
commit
```

### Disabling Root SSH Access

```
set service ssh root-login disable
commit
```

!!! warning "Security Consideration"
    Enabling root SSH login increases security risk. Consider using a regular administrative user with key-based authentication instead.

---

## Listen Address Binding

Restrict SSH to specific IP addresses for enhanced security.

### Binding to Specific Address

#### Syntax

```
set service ssh listen-address <ipv4/ipv6>
```

#### Example

```
set service ssh listen-address 172.16.32.1
commit
```

### Resetting to Default (All Interfaces)

#### Syntax

```
delete service ssh listen-address <ipv4/ipv6>
```

---

## SSH Key Management

### Adding SSH Key to User

#### Syntax

```
set service ssh key <username> <public-key>
```

Adds an SSH public key for key-based authentication. Use this when adding keys to existing users without changing their password.

### Removing SSH Key from User

#### Syntax

```
delete service ssh key <username>
```

Removes all SSH public keys for the specified user.

!!! tip "SSH Key Generation"
    Generate SSH key pairs using PuTTYgen (Windows) or `ssh-keygen` (Linux/macOS). Copy the complete public key string when configuring.

---

## Services Overview

View all services including SSH:

```
show service
```

Displays status and configuration for all active services (SSH and SNMP).

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show service ssh` | Display SSH configuration |
| `set service ssh port <port>` | Set SSH port |
| `delete service ssh port` | Reset SSH port to default |
| `set service ssh root-login enable` | Allow root SSH login |
| `set service ssh root-login disable` | Disallow root SSH login |
| `set service ssh listen-address <ip>` | Bind SSH to specific IP |
| `delete service ssh listen-address <ip>` | Reset to all interfaces |
| `set service ssh key <user> <key>` | Add SSH key for user |
| `delete service ssh key <user>` | Remove SSH keys for user |

---

## Best Practices

1. **Use non-standard port** - Consider changing from port 22 to reduce automated attacks
2. **Disable root login** - Use regular administrative accounts
3. **Use SSH keys** - Prefer key-based authentication over passwords
4. **Bind to management interface** - Restrict SSH to specific management IP when possible
5. **Enable Fail2ban** - Keep Fail2ban enabled for brute-force protection
