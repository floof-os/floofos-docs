# QoS Configuration

This section covers Quality of Service (QoS) configuration in FloofOS for traffic prioritization and policing.

## Overview

FloofOS QoS provides:

- **Marking** — Set DSCP/CoS values on packets
- **Recording** — Read and store QoS values from packets
- **Policing** — Rate limiting with token bucket algorithm
- **Scheduling** — Priority queuing (future)

---

## QoS Maps

QoS maps define how to translate between different QoS encodings (DSCP, MPLS EXP, VLAN CoS).

### Show QoS Maps

```
show qos egress map
show qos mark source
```

---

## QoS Marking

### Record QoS from Packets

Record incoming QoS values for use in marking:

```
qos record interface GE12/0/0 input source ip
commit
```

### Mark Packets

Apply QoS marking based on recorded values:

```
qos mark interface GE12/0/0 output source ip
commit
```

### View QoS Interface Configuration

```
show qos record interface
show qos mark interface
show qos store interface
```

---

## Policer Configuration

Policers implement token bucket rate limiting.

### Create Policer

#### Syntax

```
configure policer name <name> cir <rate> cb <burst> [eir <rate>] [eb <burst>] [rate-type <kbps|pps>] [round <closest|up|down>] [type <1r2c|1r3c|2r3c-2698|2r3c-4115|2r3c-mef5cf1>] [conform-action <action>] [exceed-action <action>] [violate-action <action>] [color-aware]
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `cir` | Committed Information Rate |
| `cb` | Committed Burst Size |
| `eir` | Excess Information Rate |
| `eb` | Excess Burst Size |
| `rate-type` | Rate unit (kbps or pps) |
| `type` | Policer algorithm |

### Actions

| Action | Description |
|--------|-------------|
| `drop` | Drop packets |
| `transmit` | Forward packets |
| `mark-and-transmit` | Mark and forward |

### Policer Types

| Type | Description |
|------|-------------|
| `1r2c` | Single Rate Two Color |
| `1r3c` | Single Rate Three Color (RFC 2697) |
| `2r3c-2698` | Two Rate Three Color (RFC 2698) |
| `2r3c-4115` | Two Rate Three Color (RFC 4115) |
| `2r3c-mef5cf1` | MEF 5 CF1 |

### Example: Simple Rate Limiter

```
configure policer name rate-100m cir 100000 cb 10000 rate-type kbps type 1r2c conform-action transmit exceed-action drop
commit
```

### Example: Three Color Policer

```
configure policer name gold-service cir 100000 cb 10000 eir 150000 eb 15000 rate-type kbps type 2r3c-2698 conform-action transmit exceed-action mark-and-transmit violate-action drop
commit
```

### View Policers

```
show policer
```

---

## Apply Policer to Interface

### Input Policer

```
policer input rate-100m GE12/0/0
commit
```

### Output Policer

```
policer output rate-100m GE12/0/0
commit
```

### Remove Policer

```
policer input del rate-100m GE12/0/0
commit
```

---

## DSCP Values Reference

| DSCP | PHB | Description |
|------|-----|-------------|
| 0 | BE | Best Effort |
| 8 | CS1 | Scavenger |
| 10 | AF11 | Assured Forwarding 1-1 |
| 12 | AF12 | Assured Forwarding 1-2 |
| 14 | AF13 | Assured Forwarding 1-3 |
| 18 | AF21 | Assured Forwarding 2-1 |
| 20 | AF22 | Assured Forwarding 2-2 |
| 22 | AF23 | Assured Forwarding 2-3 |
| 26 | AF31 | Assured Forwarding 3-1 |
| 28 | AF32 | Assured Forwarding 3-2 |
| 30 | AF33 | Assured Forwarding 3-3 |
| 34 | AF41 | Assured Forwarding 4-1 |
| 36 | AF42 | Assured Forwarding 4-2 |
| 38 | AF43 | Assured Forwarding 4-3 |
| 46 | EF | Expedited Forwarding |
| 48 | CS6 | Network Control |
| 56 | CS7 | Network Control |

---

## Bandwidth Limiting Example

### Per-Customer Rate Limiting

```
# Create policers for different service tiers
configure policer name bronze-10m cir 10000 cb 1000 rate-type kbps type 1r2c conform-action transmit exceed-action drop

configure policer name silver-50m cir 50000 cb 5000 rate-type kbps type 1r2c conform-action transmit exceed-action drop

configure policer name gold-100m cir 100000 cb 10000 rate-type kbps type 1r2c conform-action transmit exceed-action drop

# Apply to customer interfaces
policer input bronze-10m GE12/0/0.100
policer input silver-50m GE12/0/0.200
policer input gold-100m GE12/0/0.300

commit
```

---

## Troubleshooting

### Check Policer Statistics

```
show policer
```

### Verify Interface Binding

```
show qos record interface
show qos mark interface
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show policer` | Display policers |
| `configure policer name <name> ...` | Create policer |
| `policer input <name> <if>` | Apply input policer |
| `policer output <name> <if>` | Apply output policer |
| `policer input del <name> <if>` | Remove input policer |
| `qos record interface <if> input source ip` | Record QoS values |
| `qos mark interface <if> output source ip` | Mark packets |
| `show qos record interface` | Display QoS recording |
| `show qos mark interface` | Display QoS marking |
