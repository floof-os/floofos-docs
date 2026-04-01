# Backup and Restore

This section covers configuration backup and restore operations in FloofOS.

## Overview

FloofOS supports exporting and importing configuration backups as compressed archives (`.tar.gz`). Backups include all configuration files and can be transferred off-device for safekeeping.

---

## Backup Storage Location

By default, backups are stored in:

```
/etc/floofos-config/backups/
```

Backup files can be downloaded using SCP or SFTP for off-device storage.

---

## Exporting Backups

### Standard Export (Recommended)

#### Syntax

```
backup export <name>
```

#### Parameters

| Parameter | Description |
|-----------|-------------|
| `name` | Backup filename (without extension) |

#### Example

```
backup export backup1
```

The backup is saved to the default location as `backup1.tar.gz`.

### Export to Custom Path

#### Syntax

```
backup export <name> path <path>
```

#### Parameters

| Parameter | Description |
|-----------|-------------|
| `name` | Backup filename (without extension) |
| `path` | Target directory path |

#### Example

```
backup export backup2 path /etc
```

---

## Viewing Backups

### Syntax

```
show backups
```

Displays exported backup files (name, timestamp, and size) from the default backup directory, followed by the rollback commit history.

### Output Description

| Section | Description |
|---------|-------------|
| Location | Default backup directory |
| Backup list | Name, timestamp, and size of each backup |
| Rollback History | Commit history for rollback operations |

!!! note "Backup Display"
    Backup files are displayed without the `.tar.gz` extension, but the actual files include this extension. Only backups in the default location (`/etc/floofos-config/backups`) are shown.

---

## Importing Backups

### Standard Import (Recommended)

#### Syntax

```
backup import <name>
```

#### Parameters

| Parameter | Description |
|-----------|-------------|
| `name` | Backup filename (without extension) |

#### Example

```
backup import backup1
```

### Import from Custom Path

#### Syntax

```
backup import <name> path <path>
```

#### Example

```
backup import backup2 path /etc
```

### Post-Import Procedure

After importing a backup:

1. Commit the imported configuration:
   ```
   commit
   ```

2. Reboot the system:
   ```
   system reboot
   ```

!!! warning "Reboot Required"
    Configuration from imported backups requires a system reboot to take full effect.

---

## Transferring Backups

### Downloading Backups (SCP/SFTP)

Use SCP or SFTP to download backups from the router:

```bash
# From remote machine
scp floofos@router:/etc/floofos-config/backups/backup1.tar.gz ./
```

### Uploading Backups

Upload previously exported backups:

```bash
# To default location
scp backup1.tar.gz floofos@router:/etc/floofos-config/backups/

# To custom location
scp backup1.tar.gz floofos@router:/tmp/
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `backup export <name>` | Export backup to default location |
| `backup export <name> path <path>` | Export backup to custom path |
| `backup import <name>` | Import backup from default location |
| `backup import <name> path <path>` | Import backup from custom path |
| `show backups` | Display available backups |
| `commit` | Commit imported configuration |
| `system reboot` | Reboot after import |

---

## Best Practices

1. **Regular backups** - Create backups before and after major changes
2. **Off-device storage** - Transfer backups to external storage
3. **Descriptive names** - Use meaningful backup names (e.g., `pre-ixp-migration`)
4. **Verify backups** - Check `show backups` after export
5. **Document custom paths** - Record path if using non-default location
6. **Test restore procedure** - Periodically verify backup integrity
