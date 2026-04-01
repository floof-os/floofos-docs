# Installing FloofOS to Disk

This section describes how to install FloofOS from installation media to permanent storage.

## Prerequisites

- FloofOS booted from installation media (USB, ISO, etc.)
- Target disk for installation
- Console or SSH access to the system

---

## Installation Procedure

### Initiating Installation

From configuration mode, execute the installation command:

```
floofos(config)# system install 

FloofOS System Installation

This command will install FloofOS to permanent storage.
All existing data on the target disk will be erased.

Would you like to continue? [y/N]: y
```

### Disk Selection

The installer probes available disks and presents a selection menu:

```
Probing disks...
1 disk(s) found

The following disks were found:
  1: Drive: /dev/sda (32G) HARDDISK

Which disk should be used for installation? (Default: /dev/sda): 
```

!!! warning "Data Loss Warning"
    Installation erases all data on the selected disk. Ensure you have backups of any important data.

### Installation Options

The installer prompts for configuration options:

```
Installation will delete all data on the drive. Continue? [y/N]: y

Would you like to use all the free space on the drive? [Y/n]: y

What console should be used by default? (K: KVM, S: Serial) [K]: 

What is the hostname of this system? [floofos]: core-router

Please enter a password for the 'floofos' user: 
Please confirm the password: 
```

#### Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| Disk space | Use all available space | Yes |
| Console | Primary console type (KVM or Serial) | KVM |
| Hostname | System hostname | `floofos` |
| Password | Password for `floofos` user | Required |

### Configuration Preservation

If an existing configuration exists, the installer offers to preserve it:

```
Configuration found in floofos-config:

  1: commit 0

Would you like to preserve configuration? [Y/n]: y
Using configuration: commit 0
```

### Installation Summary

Before proceeding, the installer displays a summary:

```
Installation summary:

  Target disk     : /dev/sda
  Hostname        : core-router
  Console         : kvm
  User            : floofos
  Config restore  : commit 0

Proceed with installation? [y/N]: y
```

---

## Installation Progress

The installation proceeds through nine phases:

```
[1/9] Unmounting existing partitions...
[2/9] Creating partition table...
[3/9] Formatting partitions...
[4/9] Mounting target filesystem...
[5/9] Copying system files...
[6/9] Configuring system...
[7/9] Preserving configuration...
[8/9] Installing bootloader...
[9/9] Cleaning up...
```

!!! info "Installation Time"
    Installation typically takes approximately 15 minutes depending on hardware performance.

---

## Post-Installation

### Completion Message

Upon successful installation:

```
FloofOS installation on disk completed successfully

Remove the installation media and reboot the system
To reboot now, enter configuration mode 'configure' and type 'system reboot'
```

### Rebooting the System

1. Remove the installation media (USB drive, etc.)
2. Reboot the system:

```
floofos(config)# system reboot 
Reboot system? (Y/N): y
```

!!! note "Manual Restart"
    If the system does not respond to the reboot command, you may perform a physical power cycle by disconnecting and reconnecting power to the hardware.

---

## Verifying Installation

After reboot, verify the installation:

1. Log in with configured credentials
2. Verify configuration was preserved:

```
floofos(config)# show configuration
```

The previously committed interface configuration should appear in the output.

---

## Next Steps

After successful installation:

1. [Configure System Settings](../configuration/system.md)
2. [Set Up User Accounts](../security/users.md)
3. [Configure Network Interfaces](../configuration/interfaces.md)
4. [Enable Security Features](../security/index.md)
