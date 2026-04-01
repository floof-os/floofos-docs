# User Management

This section covers user account administration in FloofOS, including creating, modifying, and deleting user accounts.

## Overview

FloofOS provides a unified command interface for managing user accounts. All user management commands require configuration mode and must be committed to take effect.

---

## Default Users

FloofOS ships with two default accounts:

| Username | Default Password | Access | Notes |
|----------|------------------|--------|-------|
| `root` | `floofos` | Console only | SSH disabled by default |
| `floofos` | `floofos` | Console and SSH | Primary administrative user |

!!! warning "Security Best Practice"
    Change default passwords immediately after deployment.

---

## Creating Users

### Syntax

```
create user <username> password <password> [ssh-key <public-key>]
commit
```

### Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| `username` | Account username | Yes |
| `password` | Account password | Yes |
| `ssh-key` | SSH public key for key-based authentication | No |

### Example: Create New User

```
create user NOC password SecretPass123
commit
```

### Example: Create User with SSH Key

```
create user NOC password SecretPass123 ssh-key ssh-rsa AAAA...
commit
```

!!! tip "SSH Key Generation"
    SSH public keys can be generated using PuTTYgen or `ssh-keygen`. Paste the complete public key string when configuring.

---

## Changing Passwords

### Changing Default User Passwords

#### Root Password

```
create user root password YourPass123
commit
```

#### FloofOS User Password

```
create user floofos password YourPass1234
commit
```

### Updating Existing User Password

To change an existing user's password, re-issue the `create user` command with the new password:

```
create user NOC password NewPass123
commit
```

!!! note "Password Updates"
    The `create user` command automatically detects existing users and updates their credentials instead of creating duplicates. This same method applies to updating SSH keys.

---

## Deleting Users

### Syntax

```
delete user <username>
commit
```

### Example

```
delete user NOC
commit
```

### Restrictions

- The `root` user cannot be deleted
- All other users, including the default `floofos` user, can be deleted

!!! warning "Caution"
    Ensure at least one administrative user with SSH access remains before deleting the `floofos` user.

---

## Viewing Users

### Syntax

```
show users
```

Displays all FloofCTL user accounts with their username, privilege level, and creation timestamp. Built-in system accounts are marked as `(system)`.

### Output Fields

| Field | Description |
|-------|-------------|
| Username | Account name |
| Privilege | User privilege level (admin) |
| Created | Account creation timestamp or `(system)` for built-in accounts |

---

## SSH Key Management

For managing SSH keys on existing users without changing passwords, see [SSH Service Configuration](ssh.md).

### Adding SSH Key to Existing User

```
set service ssh key <username> <public-key>
```

### Removing SSH Key from User

```
delete service ssh key <username>
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `create user <name> password <pass>` | Create user or update password |
| `create user <name> password <pass> ssh-key <key>` | Create user with SSH key |
| `delete user <name>` | Delete user account |
| `show users` | Display all user accounts |
