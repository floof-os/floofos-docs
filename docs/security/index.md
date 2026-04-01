# Security

This section covers security configuration in FloofOS.

## Overview

FloofOS provides multiple layers of security for protecting the control plane and managing access:

- **User Management** - Account administration and authentication
- **SSH** - Secure remote access configuration
- **Firewall** - Control plane protection using nftables
- **Fail2ban** - Brute-force attack prevention

---

## Default Security Posture

| Feature | Default State | Recommendation |
|---------|---------------|----------------|
| Firewall | Disabled | Enable in production |
| Fail2ban | Enabled | Keep enabled |
| Root SSH | Disabled | Keep disabled |
| Default Passwords | Set | Change immediately |

---

## Documentation

| Topic | Description |
|-------|-------------|
| [User Management](users.md) | Create, modify, and delete users |
| [SSH Configuration](ssh.md) | Secure shell settings |
| [Firewall](firewall.md) | nftables firewall rules |
| [Fail2ban](fail2ban.md) | Brute-force protection |

---

## Quick Reference

### Essential Security Commands

```
# View security status
show security

# User management
show users
create user <name> password <pass>
delete user <name>

# Firewall
set security firewall enable
show security firewall rules

# Fail2ban
show security fail2ban status
show security fail2ban banned
```
