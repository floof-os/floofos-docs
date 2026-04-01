# SNMP Configuration

This section covers Simple Network Management Protocol (SNMP) configuration in FloofOS for network monitoring integration.

## Overview

FloofOS includes an SNMP agent that exposes VPP interface statistics directly to monitoring systems. The SNMP service is **disabled by default** and must be explicitly enabled.

!!! info "VPP Interface Monitoring"
    The SNMP agent reads interface statistics directly from VPP (not Linux LCP interfaces). When configuring monitoring systems (Zabbix, LibreNMS, PRTG, Grafana, Cacti), interface names appear as VPP interface names.

---

## SNMP State

### Viewing SNMP Status

#### Syntax

```
show service snmp
```

Displays the state of the `snmpd` service and `vpp-snmp-agent`, and when enabled, the listening address and port (UDP 161).

### Enabling SNMP

#### Syntax

```
set service snmp enable
commit
```

### Disabling SNMP

```
set service snmp disable
commit
```

---

## SNMP Configuration

### Community String

#### Syntax

```
set service snmp community <string>
```

Sets the SNMPv2c community string for read-only access.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `string` | Community string | public |

#### Example

```
set service snmp community floof123
commit
```

### System Location

#### Syntax

```
set service snmp location <string>
```

Sets the sysLocation MIB value for device identification.

#### Example

```
set service snmp location DC_1
commit
```

### System Contact

#### Syntax

```
set service snmp contact <string>
```

Sets the sysContact MIB value for administrative contact information.

#### Example

```
set service snmp contact noc@company.com
commit
```

---

## Viewing Configuration

### SNMP Configuration Display

#### Syntax

```
show service snmp config
```

Displays the active SNMP configuration including community strings and system contact.

---

## SNMP Statistics

### Viewing SNMP Counters

#### Syntax

```
show service snmp statistics
```

Displays SNMP protocol counters including incoming/outgoing packet counts, bad community name attempts, GetNext requests, and GetResponse packets sent.

### Key Statistics

| Counter | Description |
|---------|-------------|
| snmpInPkts | Total incoming SNMP packets |
| snmpOutPkts | Total outgoing SNMP packets |
| snmpInBadCommunityNames | Requests with incorrect community |
| snmpInGetNexts | SNMP GetNext requests |
| snmpOutGetResponses | SNMP GetResponse packets sent |

---

## Services Overview

View all services including SNMP:

```
show service
```

Displays status and configuration for all active services (SSH and SNMP).

---

## Command Reference

| Command | Description |
|---------|-------------|
| `set service snmp enable` | Enable SNMP agent |
| `set service snmp disable` | Disable SNMP agent |
| `set service snmp community <str>` | Set community string |
| `set service snmp location <str>` | Set system location |
| `set service snmp contact <str>` | Set system contact |
| `show service snmp` | Display SNMP status |
| `show service snmp config` | Display SNMP configuration |
| `show service snmp statistics` | Display SNMP counters |
| `show service` | Display all services |

---

## Integration with Monitoring Systems

When configuring monitoring platforms:

1. **Protocol**: SNMPv2c
2. **Port**: UDP 161
3. **Community**: As configured (default: `public`)
4. **Interface Names**: VPP format (e.g., `GE12/0/0`)

### Supported Platforms

- Zabbix
- LibreNMS
- PRTG
- Grafana
- Cacti
- Other SNMP-compatible monitoring solutions

---

## Best Practices

1. **Change default community** - Never use `public` in production
2. **Use firewall rules** - Restrict SNMP access to monitoring servers
3. **Set location and contact** - Helpful for inventory and alerting
4. **Monitor statistics** - Check `snmpInBadCommunityNames` for unauthorized access attempts
