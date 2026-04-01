# Traffic Monitoring

This section covers real-time traffic monitoring for VPP interfaces in FloofOS.

## Overview

FloofOS provides real-time traffic visualization directly from VPP interfaces. The traffic monitor displays a text-based graphical interface showing throughput statistics.

!!! info "VPP Direct Monitoring"
    Traffic statistics are read directly from VPP interfaces, not Linux LCP interfaces. Use VPP interface names (from `show interface`) when monitoring traffic.

---

## Real-Time Traffic Display

### Syntax

```
show traffic interface <interface>
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `interface` | VPP interface name (e.g., `GE12/0/0`) |

### Example

```
show traffic interface GE12/0/0
```

### Output

The command launches a text-based user interface (TUI) displaying real-time traffic graphs, similar to bandwidth monitoring tools. The display shows:

- Incoming traffic (RX)
- Outgoing traffic (TX)
- Current throughput rates
- Historical traffic graph

### Exiting

Press `Ctrl+C` or `q` to exit the traffic monitoring interface.

---

## Finding Interface Names

Before monitoring traffic, identify available interfaces:

```
show interface
```

Use the VPP interface name (left column) with the traffic monitoring command.

---

## Use Cases

| Scenario | Command |
|----------|---------|
| Monitor uplink traffic | `show traffic interface GE12/0/0` |
| Verify BGP session activity | Monitor peer-facing interface |
| Troubleshoot throughput | Real-time visualization during testing |

---

## Related Commands

| Command | Description |
|---------|-------------|
| `show interface` | List all interfaces |
| `show resource` | Display system resource usage |
| `ping` | Test connectivity |
| `traceroute` | Trace packet path |
