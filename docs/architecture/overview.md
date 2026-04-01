# System Overview

High-level architecture of FloofOS.

---

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              FloofOS                                      │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                         User Space                                   │ │
│  │                                                                      │ │
│  │  ┌─────────────┐    ┌─────────────────────────────────────────────┐ │ │
│  │  │  floofctl   │    │           Dataplane Namespace               │ │ │
│  │  │    CLI      │    │                                             │ │ │
│  │  │             │    │  ┌───────────┐  ┌────────────────────────┐ │ │ │
│  │  │  - show     │    │  │    VPP    │  │      BIRD2             │ │ │ │
│  │  │  - set      │────┼──│  (DPDK)   │──│  - BGP (Pathvector)    │ │ │ │
│  │  │  - commit   │    │  │           │  │  - OSPF                │ │ │ │
│  │  │  - rollback │    │  └───────────┘  │  - Static              │ │ │ │
│  │  │             │    │        │        └────────────────────────┘ │ │ │
│  │  └─────────────┘    │        │                                   │ │ │
│  │        │            │  ┌─────┴─────┐  ┌──────────┐  ┌─────────┐ │ │ │
│  │        │            │  │    LCP    │  │   SSH    │  │  SNMP   │ │ │ │
│  │        │            │  │ (Linux CP)│  │  Server  │  │  Agent  │ │ │ │
│  │        │            │  └───────────┘  └──────────┘  └─────────┘ │ │ │
│  │        │            └─────────────────────────────────────────────┘ │ │
│  │        │                                                             │ │
│  │  ┌─────┴───────────────────────────────────────────────────────────┐│ │
│  │  │                    Control Plane Services                        ││ │
│  │  │  - nftables Firewall                                            ││ │
│  │  │  - fail2ban                                                      ││ │
│  │  │  - Audit Logging                                                 ││ │
│  │  │  - Configuration Management                                      ││ │
│  │  └─────────────────────────────────────────────────────────────────┘│ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                         Kernel Space                                 │ │
│  │                                                                      │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │                    Linux Kernel                               │  │ │
│  │  │  - Network Stack (for control plane)                          │  │ │
│  │  │  - vfio-pci / uio_pci_generic (DPDK drivers)                 │  │ │
│  │  │  - Hugepages memory                                           │  │ │
│  │  │  - Network Namespaces                                         │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │                         Hardware                                     │ │
│  │                                                                      │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐│ │
│  │  │   CPU    │  │  Memory  │  │  Storage │  │   Network Cards      ││ │
│  │  │ (Multi-  │  │ (NUMA +  │  │  (NVMe)  │  │  (DPDK-compatible)   ││ │
│  │  │  core)   │  │Hugepages)│  │          │  │  Intel, Mellanox     ││ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────────────────┘│ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Component Overview

### floofctl CLI

- Written in Go
- Juniper-style hierarchical commands
- Two modes: Operational and Configuration
- Interacts with VPP via vppctl
- Interacts with BIRD via birdc
- Manages configuration files

### VPP (Vector Packet Processing)

- High-performance packet processing
- DPDK for kernel bypass
- Graph node architecture
- Multi-core scaling with worker threads
- Supports 10G, 25G, 40G, 100G NICs

### BIRD2

- Production-grade routing daemon
- BGP, OSPF, Static routing
- Runs in dataplane namespace
- Configured via Pathvector

### Pathvector

- BGP automation tool
- Generates BIRD configuration
- IRR filtering
- RPKI validation
- Community management

---

## Network Namespaces

FloofOS uses network namespaces to isolate the dataplane:

```
┌─────────────────────────────────────┐
│         Default Namespace           │
│                                     │
│  - floofctl CLI                    │
│  - Management interface            │
│  - nftables firewall               │
│  - Audit logging                   │
│  - System services                 │
└───────────────┬─────────────────────┘
                │
           veth pair
                │
┌───────────────┴─────────────────────┐
│        Dataplane Namespace          │
│                                     │
│  - VPP (with DPDK interfaces)      │
│  - BIRD routing daemon             │
│  - SSH server                      │
│  - SNMP agent                      │
│  - LCP interfaces                  │
└─────────────────────────────────────┘
```

### Why Namespaces?

1. **Isolation** - Dataplane traffic separate from control
2. **Security** - Management interface protected
3. **Flexibility** - Linux tools work in dataplane
4. **BIRD Integration** - BIRD sees VPP interfaces via LCP

---

## Data Flow

### Packet Processing

```
Physical NIC → DPDK PMD → VPP Input Node → Processing Graph → VPP Output Node → DPDK PMD → Physical NIC
```

1. **DPDK PMD** polls NIC for packets (no interrupts)
2. **VPP Input** batches packets into vectors
3. **Processing Graph** routes/switches packets
4. **VPP Output** transmits packets
5. **DPDK PMD** sends to NIC

### Control Plane

```
CLI Command → floofctl → vppctl/birdc → VPP/BIRD → Response → floofctl → CLI Output
```

---

## Configuration Flow

```
User Input → floofctl → Config Files → Services → Running Config
                            │
                            ▼
                    Commit History
                    (50 rollbacks)
```

### Commit Process

1. User makes changes via CLI
2. Changes written to config files
3. `commit` triggers:
   - Save current config to rollback history
   - Reload affected services
   - Validate configuration
4. Success/failure reported to user

---

## Security Model

```
┌─────────────────────────────────────────┐
│              External Network            │
└─────────────────┬───────────────────────┘
                  │
           ┌──────┴──────┐
           │  nftables   │ ← Control plane firewall
           │  Firewall   │
           └──────┬──────┘
                  │
           ┌──────┴──────┐
           │  fail2ban   │ ← Brute-force protection
           └──────┬──────┘
                  │
           ┌──────┴──────┐
           │    SSH      │ ← Key-based auth
           │   Server    │
           └──────┬──────┘
                  │
           ┌──────┴──────┐
           │  floofctl   │ ← User authentication
           │    CLI      │ ← Privilege levels
           └──────┬──────┘
                  │
           ┌──────┴──────┐
           │   Audit     │ ← All actions logged
           │    Log      │
           └─────────────┘
```

---

## Performance Characteristics

### Theoretical Limits

| NIC Speed | Max PPS (64B) | Max Throughput |
|-----------|---------------|----------------|
| 10G | ~14.88 Mpps | 10 Gbps |
| 25G | ~37.2 Mpps | 25 Gbps |
| 40G | ~59.5 Mpps | 40 Gbps |
| 100G | ~148.8 Mpps | 100 Gbps |

### VPP Scaling

- One main thread (core 0)
- Multiple worker threads (cores 1+)
- Each worker handles subset of queues
- NUMA-aware for multi-socket systems

---

## High Availability

### Current Features

- Graceful restart for BGP
- Configuration rollback
- Service auto-restart

### Future Roadmap

- VRRP support
- Active-standby clustering
- Configuration synchronization

