# IPsec Configuration

This section covers IPsec (Internet Protocol Security) configuration in FloofOS for secure network communications.

## Overview

IPsec provides cryptographic security services for IP traffic:

- **Authentication** — Verify packet origin
- **Integrity** — Detect packet tampering
- **Confidentiality** — Encrypt packet contents
- **Anti-replay** — Prevent packet replay attacks

FloofOS supports both tunnel mode and transport mode IPsec.

---

## IPsec Components

| Component | Description |
|-----------|-------------|
| **SA (Security Association)** | Defines encryption/authentication parameters |
| **SPD (Security Policy Database)** | Defines which traffic to protect |
| **SAD (Security Association Database)** | Stores active SAs |

---

## View IPsec Configuration

### Show All IPsec

```
show ipsec all
```

### Show Security Associations

```
show ipsec sa
```

### Show Security Policies

```
show ipsec spd
```

---

## Create Security Association

### Syntax

```
ipsec sa add <sa-id> spi <spi> crypto-alg <alg> crypto-key <key> [integ-alg <alg> integ-key <key>] [tunnel src <ip> dst <ip>] [esn] [use-anti-replay]
```

### Encryption Algorithms

| Algorithm | Description |
|-----------|-------------|
| `aes-cbc-128` | AES-CBC 128-bit |
| `aes-cbc-192` | AES-CBC 192-bit |
| `aes-cbc-256` | AES-CBC 256-bit |
| `aes-gcm-128` | AES-GCM 128-bit (AEAD) |
| `aes-gcm-192` | AES-GCM 192-bit (AEAD) |
| `aes-gcm-256` | AES-GCM 256-bit (AEAD) |

### Integrity Algorithms

| Algorithm | Description |
|-----------|-------------|
| `sha1-96` | HMAC-SHA1-96 |
| `sha-256-96` | HMAC-SHA256-96 |
| `sha-256-128` | HMAC-SHA256-128 |
| `sha-384-192` | HMAC-SHA384-192 |
| `sha-512-256` | HMAC-SHA512-256 |

### Example: Create SA with AES-GCM

```
ipsec sa add 10 spi 0x0a000001 crypto-alg aes-gcm-256 crypto-key 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef tunnel src 10.0.0.1 dst 10.0.0.2 esn use-anti-replay
commit
```

### Example: Create SA with AES-CBC + SHA256

```
ipsec sa add 20 spi 0x14000001 crypto-alg aes-cbc-256 crypto-key 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef integ-alg sha-256-128 integ-key 0123456789abcdef0123456789abcdef tunnel src 10.0.0.1 dst 10.0.0.2
commit
```

---

## Create Security Policy Database

### Syntax

```
ipsec spd add <spd-id>
```

### Example

```
ipsec spd add 1
commit
```

---

## Create Security Policy

### Syntax

```
ipsec policy add spd <spd-id> priority <n> <inbound|outbound> action <bypass|discard|protect> [sa <sa-id>] [local-ip-range <start>-<end>] [remote-ip-range <start>-<end>] [protocol <n>]
```

### Actions

| Action | Description |
|--------|-------------|
| `bypass` | Allow traffic without IPsec |
| `discard` | Drop traffic |
| `protect` | Apply IPsec protection |

### Example: Protect Policy

```
ipsec policy add spd 1 priority 100 outbound action protect sa 10 local-ip-range 10.1.0.0-10.1.255.255 remote-ip-range 10.2.0.0-10.2.255.255
ipsec policy add spd 1 priority 100 inbound action protect sa 20 local-ip-range 10.1.0.0-10.1.255.255 remote-ip-range 10.2.0.0-10.2.255.255
commit
```

---

## Bind SPD to Interface

### Syntax

```
set interface ipsec spd <interface> <spd-id>
```

### Example

```
set interface ipsec spd GE12/0/0 1
commit
```

---

## Site-to-Site IPsec Example

### Topology

```
    Site A                                    Site B
    10.0.0.1                                  10.0.0.2
       |                                         |
   +---+---+        IPsec Tunnel             +---+---+
   |FloofOS|=============================|FloofOS|
   +---+---+                                 +---+---+
       |                                         |
  LAN: 10.1.0.0/24                        LAN: 10.2.0.0/24
```

### Site A Configuration

```
# Create outbound SA (encrypt)
ipsec sa add 10 spi 0x0a000001 crypto-alg aes-gcm-256 crypto-key 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef tunnel src 10.0.0.1 dst 10.0.0.2 esn use-anti-replay

# Create inbound SA (decrypt)
ipsec sa add 20 spi 0x0a000002 crypto-alg aes-gcm-256 crypto-key fedcba9876543210fedcba9876543210fedcba9876543210fedcba9876543210 tunnel src 10.0.0.2 dst 10.0.0.1 esn use-anti-replay

# Create SPD
ipsec spd add 1

# Create policies
ipsec policy add spd 1 priority 100 outbound action protect sa 10 local-ip-range 10.1.0.0-10.1.255.255 remote-ip-range 10.2.0.0-10.2.255.255

ipsec policy add spd 1 priority 100 inbound action protect sa 20 local-ip-range 10.1.0.0-10.1.255.255 remote-ip-range 10.2.0.0-10.2.255.255

# Bind to interface
set interface ipsec spd GE12/0/0 1

commit
```

### Site B Configuration

```
# Create outbound SA (encrypt) - mirrors Site A's inbound
ipsec sa add 10 spi 0x0a000002 crypto-alg aes-gcm-256 crypto-key fedcba9876543210fedcba9876543210fedcba9876543210fedcba9876543210 tunnel src 10.0.0.2 dst 10.0.0.1 esn use-anti-replay

# Create inbound SA (decrypt) - mirrors Site A's outbound
ipsec sa add 20 spi 0x0a000001 crypto-alg aes-gcm-256 crypto-key 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef tunnel src 10.0.0.1 dst 10.0.0.2 esn use-anti-replay

# Create SPD
ipsec spd add 1

# Create policies
ipsec policy add spd 1 priority 100 outbound action protect sa 10 local-ip-range 10.2.0.0-10.2.255.255 remote-ip-range 10.1.0.0-10.1.255.255

ipsec policy add spd 1 priority 100 inbound action protect sa 20 local-ip-range 10.2.0.0-10.2.255.255 remote-ip-range 10.1.0.0-10.1.255.255

# Bind to interface
set interface ipsec spd GE12/0/0 1

commit
```

---

## Troubleshooting

### Check SA Status

```
show ipsec sa
show ipsec sa detail
```

### Check Policy Match

```
show ipsec spd
```

### Check Counters

```
show errors
```

---

## Command Reference

| Command | Description |
|---------|-------------|
| `show ipsec all` | Display all IPsec configuration |
| `show ipsec sa` | Display Security Associations |
| `show ipsec spd` | Display Security Policy Database |
| `ipsec sa add <id> spi <spi> ...` | Create Security Association |
| `ipsec sa del <id>` | Delete Security Association |
| `ipsec spd add <id>` | Create SPD |
| `ipsec spd del <id>` | Delete SPD |
| `ipsec policy add spd <id> ...` | Add security policy |
| `set interface ipsec spd <if> <spd>` | Bind SPD to interface |
