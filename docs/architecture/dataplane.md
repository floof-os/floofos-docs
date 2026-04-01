# Dataplane Architecture

VPP and DPDK architecture details.

---

## VPP Overview

VPP (Vector Packet Processing) is a high-performance packet processing stack originally developed by Cisco and now part of the Linux Foundation's FD.io project.

---

## Graph Node Architecture

VPP processes packets through a directed graph of nodes:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        VPP Processing Graph                              │
│                                                                          │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐             │
│   │  dpdk-input  │───▶│ethernet-input│───▶│  ip4-input   │             │
│   └──────────────┘    └──────────────┘    └──────────────┘             │
│                                                  │                       │
│                           ┌──────────────────────┼──────────────────┐   │
│                           │                      │                  │   │
│                           ▼                      ▼                  ▼   │
│                    ┌──────────────┐       ┌──────────────┐   ┌─────────┐│
│                    │  ip4-lookup  │       │ ip4-icmp-input│   │ip4-drop ││
│                    └──────────────┘       └──────────────┘   └─────────┘│
│                           │                      │                       │
│                           ▼                      ▼                       │
│                    ┌──────────────┐       ┌──────────────┐              │
│                    │ip4-rewrite  │       │ ip4-icmp-echo│              │
│                    └──────────────┘       └──────────────┘              │
│                           │                                              │
│                           ▼                                              │
│                    ┌──────────────┐                                     │
│                    │  interface-  │                                     │
│                    │    output    │                                     │
│                    └──────────────┘                                     │
│                           │                                              │
│                           ▼                                              │
│                    ┌──────────────┐                                     │
│                    │ dpdk-output  │                                     │
│                    └──────────────┘                                     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Vector Processing

Traditional: Process one packet through all nodes

VPP: Process vector of packets at each node

Benefits:

- Better cache utilization
- Reduced instruction cache misses
- Higher throughput

---

## DPDK Integration

DPDK (Data Plane Development Kit) provides:

### Poll Mode Drivers (PMD)

- Direct NIC access from userspace
- No kernel involvement
- Zero-copy packet handling

### Supported NICs

| Vendor | NICs | PMD |
|--------|------|-----|
| Intel | X710, XXV710, E810 | i40e, ice |
| Intel | X520, X540 | ixgbe |
| Intel | I350, I210 | igb |
| Mellanox | ConnectX-4/5/6 | mlx5 |
| Virtio | Virtual | virtio |

### DPDK Binding

NICs are bound to DPDK drivers at boot:

```bash
# Before (kernel driver)
0000:00:08.0 'Ethernet Controller' drv=i40e

# After (DPDK driver)
0000:00:08.0 'Ethernet Controller' drv=vfio-pci
```

---

## Memory Architecture

### Hugepages

VPP uses hugepages for:

- Packet buffers
- Memory pools
- Data structures

Configuration in `/etc/vpp/startup.conf`:

```
memory {
    main-heap-size 2048M
    main-heap-page-size default-hugepage
}

buffers {
    buffers-per-numa 32768
    page-size default-hugepage
}
```

### NUMA Awareness

For multi-socket systems:

- Memory allocated on local NUMA node
- Worker threads pinned to local CPUs
- Minimizes cross-socket traffic

```
dpdk {
    socket-mem 2048,2048    # 2GB per socket
}
```

---

## CPU Pinning

### VPP Thread Model

```
┌─────────────────────────────────────────────────────────────┐
│                         CPU Cores                            │
│                                                              │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐            │
│  │ Core 0 │  │ Core 1 │  │ Core 2 │  │ Core 3 │  ...       │
│  │        │  │        │  │        │  │        │            │
│  │  Main  │  │Worker 1│  │Worker 2│  │Worker 3│            │
│  │ Thread │  │        │  │        │  │        │            │
│  └────────┘  └────────┘  └────────┘  └────────┘            │
│      │           │           │           │                  │
│      │      ┌────┴───────────┴───────────┴────┐            │
│      │      │     Packet Processing           │            │
│      │      │     (RX/TX queues)              │            │
│      │      └─────────────────────────────────┘            │
│      │                                                      │
│      ▼                                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Control Plane (CLI, API, timers, stats)            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

Configuration:

```
cpu {
    main-core 0
    corelist-workers 1-7
}
```

### Core Assignment

| Core | Function |
|------|----------|
| 0 | VPP main thread, control plane |
| 1-N | VPP worker threads, packet processing |

---

## RX/TX Queues

Each interface can have multiple queues:

```
┌─────────────────────────────────────────────┐
│              Physical NIC                    │
│                                              │
│  ┌────────┐ ┌────────┐ ┌────────┐          │
│  │ RX Q0  │ │ RX Q1  │ │ RX Q2  │  ...     │
│  └────┬───┘ └────┬───┘ └────┬───┘          │
│       │          │          │               │
│       ▼          ▼          ▼               │
│  ┌────────┐ ┌────────┐ ┌────────┐          │
│  │Worker 1│ │Worker 2│ │Worker 3│  ...     │
│  └────┬───┘ └────┬───┘ └────┬───┘          │
│       │          │          │               │
│       ▼          ▼          ▼               │
│  ┌────────┐ ┌────────┐ ┌────────┐          │
│  │ TX Q0  │ │ TX Q1  │ │ TX Q2  │  ...     │
│  └────────┘ └────────┘ └────────┘          │
└─────────────────────────────────────────────┘
```

Configuration:

```
dpdk {
    dev 0000:00:08.0 {
        num-rx-queues 4
        num-tx-queues 4
    }
}
```

---

## Linux Control Plane (LCP)

LCP creates Linux netdevices that mirror VPP interfaces:

```
┌─────────────────────────────────────────────┐
│                   VPP                        │
│                                              │
│  ┌──────────────────────────────────────┐  │
│  │      GigabitEthernet0/8/0            │  │
│  │      (DPDK interface)                │  │
│  └──────────────────┬───────────────────┘  │
│                     │ LCP                   │
│                     ▼                       │
│  ┌──────────────────────────────────────┐  │
│  │         host-eth0                    │  │
│  │    (Linux TAP interface)             │  │
│  └──────────────────────────────────────┘  │
│                     │                       │
└─────────────────────┼───────────────────────┘
                      │
                      ▼
              Linux Network Stack
              (BIRD sees this interface)
```

Benefits:

- BIRD can manage VPP interfaces
- Standard Linux tools work
- Routing table sync via kernel

---

## Buffer Management

### Buffer Pool

```
┌─────────────────────────────────────────────┐
│              VPP Buffer Pool                 │
│                                              │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐              │
│  │Buf1│ │Buf2│ │Buf3│ │Buf4│  ...         │
│  └────┘ └────┘ └────┘ └────┘              │
│                                              │
│  Each buffer:                               │
│  - 2KB default size                         │
│  - Pre-allocated from hugepages             │
│  - Reference counted                        │
│  - Cache-line aligned                       │
└─────────────────────────────────────────────┘
```

Configuration:

```
buffers {
    buffers-per-numa 32768
    default data-size 2048
}
```

---

## Performance Tuning

### Key Parameters

| Parameter | Impact | Recommendation |
|-----------|--------|----------------|
| Worker cores | Throughput | 1 per 10G |
| RX queues | Parallelism | Match workers |
| Buffers | Burst handling | 32K+ per NUMA |
| Hugepages | Memory access | 1GB pages |

### Auto-Tuning

FloofOS auto-configures VPP based on:

- Detected CPU cores
- Available memory
- NUMA topology
- NIC capabilities

This happens at boot via `vpp-auto-config.service`.

---

## Troubleshooting

### Low Throughput

1. Check worker utilization:

```
show runtime
```

2. Verify queue distribution:

```
show interface rx-placement
```

3. Check for drops:

```
show buffers
show errors
```

### High Latency

1. Check RX mode:

```
show interface rx-mode
```

2. Verify CPU isolation (check for scheduler interference)

### Memory Issues

1. Check hugepage allocation:

```bash
cat /proc/meminfo | grep Huge
```

2. Check VPP memory:

```
show memory
show buffers
```

