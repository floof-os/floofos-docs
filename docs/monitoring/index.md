# Monitoring

This section covers monitoring capabilities in FloofOS.

## Overview

FloofOS provides tools for monitoring system performance and network traffic:

- **SNMP** - Integration with external monitoring systems
- **Traffic Monitoring** - Real-time interface traffic visualization
- **Logging** - System and audit logging

---
## Documentation

| Topic | Description |
|-------|-------------|
| [SNMP](snmp.md) | SNMP agent configuration |
| [Traffic Monitoring](traffic.md) | Real-time traffic display |
| [Logging](logging.md) | System logging |
---

## Quick Reference

### SNMP

```
# Enable SNMP
set service snmp enable

# Configure SNMP
set service snmp community <string>
set service snmp location <location>
set service snmp contact <email>

# View status
show service snmp
```

### Traffic Monitoring

```
# Real-time traffic
show traffic interface <interface>
```

### System Logging

```
# View logs
show system logging
show system logging last <n>
show system logging user <username>
show system logging today
```
