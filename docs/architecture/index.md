# Architecture

Technical architecture documentation for FloofOS.

---
## Overview

FloofOS is a high-performance network operating system built on:

| Component | Technology |
|-----------|------------|
| Base OS | Debian Linux |
| Dataplane | VPP (Vector Packet Processing) |
| Routing | BIRD2 + Pathvector |
| CLI | floofctl (Go) |
---

## Architecture Sections

- **[System Overview](overview.md)** - High-level architecture
- **[Dataplane Architecture](dataplane.md)** - VPP and DPDK details

---

## Design Principles

1. **Performance First**
   - DPDK for kernel bypass
   - VPP graph architecture for batched processing
   - NUMA-aware memory allocation

2. **Operational Simplicity**
   - Juniper-style CLI
   - Transactional configuration
   - Comprehensive logging

3. **Production Ready**
   - Security hardened
   - High availability features
   - Enterprise monitoring (SNMP)

4. **Standards Compliant**
   - BGP (RFC 4271, 6793, 7606)
   - OSPF (RFC 2328, 5340)
   - SNMP (RFC 3411-3418)

