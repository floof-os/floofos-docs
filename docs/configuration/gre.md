# GRE Tunnel Configuration

This section covers Generic Routing Encapsulation (GRE) tunnel configuration in FloofOS.

## Overview

GRE is a tunneling protocol that encapsulates a wide variety of network layer protocols inside point-to-point links. Common use cases include:

- Site-to-site connectivity over public networks
- Connecting non-contiguous networks
- Encapsulating multicast traffic
- Transparent Ethernet Bridging (TEB)

---

## Create GRE Tunnel

### Syntax

```
create gre tunnel src <local-ip> dst <remote-ip> [instance <n>] [outer-table-id <vrf>] [teb] [erspan <session-id>]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `src` | Local tunnel endpoint IP address |
| `dst` | Remote tunnel endpoint IP address |
| `instance` | Tunnel instance number |
| `outer-table-id` | VRF for outer IP header |
| `teb` | Transparent Ethernet Bridging mode |
| `erspan` | ERSPAN session ID for mirroring |

### Example: Basic GRE Tunnel

```
create gre tunnel src 10.0.0.1 dst 10.0.0.2
set interface state gre0 up
set interface ip address gre0 192.168.100.1/30
commit
```

---

## View GRE Tunnels

```
show gre tunnel
```

---

## Delete GRE Tunnel

```
delete gre tunnel src 10.0.0.1 dst 10.0.0.2
commit
```

---

## GRE Tunnel Types

### Standard GRE (Layer 3)

For routing IP traffic between sites:

```
create gre tunnel src 10.0.0.1 dst 10.0.0.2
set interface state gre0 up
set interface ip address gre0 192.168.100.1/30
commit
```

### GRE TEB (Transparent Ethernet Bridging)

For bridging Layer 2 traffic:

```
create gre tunnel src 10.0.0.1 dst 10.0.0.2 teb
set interface state gre0 up
set interface l2 bridge gre0 100
commit
```

### ERSPAN (Encapsulated Remote SPAN)

For remote traffic mirroring:

```
create gre tunnel src 10.0.0.1 dst 10.0.0.2 erspan 1
```

---

## Site-to-Site GRE Example

### Topology

```
    Site A                                    Site B
    10.0.0.1                                  10.0.0.2
       |                                         |
   +---+---+          GRE Tunnel             +---+---+
   |FloofOS|=============================|FloofOS|
   +---+---+     192.168.100.0/30        +---+---+
       |                                         |
  LAN: 10.1.0.0/24                        LAN: 10.2.0.0/24
```

### Site A Configuration

```
# Create GRE tunnel
create gre tunnel src 10.0.0.1 dst 10.0.0.2
set interface state gre0 up
set interface ip address gre0 192.168.100.1/30

# Route to remote LAN via tunnel
ip route add 10.2.0.0/24 via 192.168.100.2

commit
```

### Site B Configuration

```
# Create GRE tunnel
create gre tunnel src 10.0.0.2 dst 10.0.0.1
set interface state gre0 up
set interface ip address gre0 192.168.100.2/30

# Route to remote LAN via tunnel
ip route add 10.1.0.0/24 via 192.168.100.1

commit
```

---

## GRE over VRF

Create GRE tunnel with outer header in specific VRF:

```
create gre tunnel src 10.0.0.1 dst 10.0.0.2 outer-table-id 100
```

---

## Troubleshooting

### Tunnel Not Working

1. Check tunnel status:
   ```
   show gre tunnel
   show interface gre0
   ```

2. Verify underlay connectivity:
   ```
   ping 10.0.0.2
   ```

3. Check routing:
   ```
   show ip fib
   ```

### MTU Issues

GRE adds 24 bytes overhead. Adjust MTU if needed:

```
set interface mtu 1476 gre0
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `create gre tunnel src <ip> dst <ip>` | Create GRE tunnel |
| `create gre tunnel src <ip> dst <ip> teb` | Create GRE TEB tunnel |
| `create gre tunnel src <ip> dst <ip> erspan <id>` | Create ERSPAN tunnel |
| `delete gre tunnel src <ip> dst <ip>` | Delete GRE tunnel |
| `show gre tunnel` | Display GRE tunnels |
| `set interface state gre<n> up` | Enable tunnel |
| `set interface ip address gre<n> <ip>` | Assign IP to tunnel |
