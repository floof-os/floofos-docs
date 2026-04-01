# VXLAN Configuration

This section covers VXLAN tunnel configuration for Layer 2 overlay networks in FloofOS.

## Overview

VXLAN (Virtual Extensible LAN) encapsulates Layer 2 Ethernet frames in UDP packets, enabling Layer 2 networks to span across Layer 3 infrastructure.

---

## Create VXLAN Tunnel

### Syntax

```
create vxlan tunnel src <source-ip> dst <destination-ip> vni <vxlan-id> [instance <n>] [decap-next <l2|l3>]
```

### Example

```
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 10000
commit
```

---

## View VXLAN Tunnels

```
show vxlan tunnel
```

---

## Configure VXLAN Interface

### Bring Interface Up

```
set interface state vxlan_tunnel0 up
```

### Assign to Bridge Domain

```
set interface l2 bridge vxlan_tunnel0 100
```

See [Bridge Configuration](bridge.md) for complete bridge domain documentation.

---

## Delete VXLAN Tunnel

```
delete vxlan tunnel vxlan_tunnel0
commit
```

---

## VXLAN with Bridge Domain

### Create Bridge Domain

```
create bridge-domain 100
```

### Add Interfaces to Bridge

```
set interface l2 bridge GE12/0/0 100
set interface l2 bridge vxlan_tunnel0 100
```

### View Bridge Domain

```
show bridge-domain
```

---

## Multicast VXLAN

For BUM traffic handling:

```
create vxlan tunnel src 10.0.0.1 dst 239.1.1.1 vni 10000
```

---

## Network Example

### Site A Configuration

```
# Underlay
set interface ip address GE12/0/0 10.0.0.1/24
set interface state GE12/0/0 up

# VXLAN tunnel
create vxlan tunnel src 10.0.0.1 dst 10.0.0.2 vni 10000
set interface state vxlan_tunnel0 up

# Bridge domain
create bridge-domain 100
set interface l2 bridge GE12/0/1 100
set interface l2 bridge vxlan_tunnel0 100

commit
```

### Site B Configuration

```
# Underlay
set interface ip address GE12/0/0 10.0.0.2/24
set interface state GE12/0/0 up

# VXLAN tunnel
create vxlan tunnel src 10.0.0.2 dst 10.0.0.1 vni 10000
set interface state vxlan_tunnel0 up

# Bridge domain
create bridge-domain 100
set interface l2 bridge GE12/0/1 100
set interface l2 bridge vxlan_tunnel0 100

commit
```

---

## L2 FIB

View MAC address table:

```
show l2fib all
```

---

## Troubleshooting

### Tunnel Not Working

1. Check underlay connectivity:
   ```
   ping 10.0.0.2
   ```

2. Verify tunnel status:
   ```
   show vxlan tunnel
   show interface vxlan_tunnel0
   ```

### MTU Issues

VXLAN adds 50 bytes overhead. Ensure underlay MTU is sufficient:

```
set interface mtu 9000 GE12/0/0
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `create vxlan tunnel src <ip> dst <ip> vni <id>` | Create VXLAN tunnel |
| `delete vxlan tunnel <tunnel>` | Delete VXLAN tunnel |
| `show vxlan tunnel` | Display VXLAN tunnels |
| `set interface state <tunnel> up` | Enable tunnel |
| `set interface l2 bridge <tunnel> <bd>` | Add tunnel to bridge |
