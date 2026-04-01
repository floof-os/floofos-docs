# IKEv2 Configuration

This section covers Internet Key Exchange version 2 (IKEv2) configuration in FloofOS for establishing IPsec VPN tunnels with automatic key management.

---

## Overview

IKEv2 provides automated key exchange and security association (SA) management for IPsec VPNs:

| Feature | Description |
|---------|-------------|
| **Automatic Key Exchange** | Negotiates encryption keys automatically |
| **Perfect Forward Secrecy** | Optional PFS with Diffie-Hellman |
| **NAT Traversal** | Built-in NAT-T support |
| **Dead Peer Detection** | Automatic peer liveness detection |
| **Multiple Auth Methods** | PSK, certificates, EAP |
| **MOBIKE** | Supports IP address changes |

---

## IKEv2 Components

### Terminology

| Term | Description |
|------|-------------|
| **IKE SA** | Security Association for IKE protocol itself |
| **Child SA** | IPsec SA negotiated through IKE |
| **Profile** | Configuration template for IKE negotiations |
| **Transform Set** | Encryption and integrity algorithms |
| **Traffic Selector** | Defines protected traffic |

---

## IKEv2 Profile Configuration

### Create IKEv2 Profile

**Syntax:**

```
ikev2 profile add <profile-name>
```

**Example:**

```
ikev2 profile add site-to-site
commit
```

### Delete IKEv2 Profile

```
ikev2 profile del <profile-name>
```

---

## Authentication Configuration

### Pre-Shared Key (PSK) Authentication

**Syntax:**

```
ikev2 profile set <profile-name> auth shared-key-mic string <psk>
```

**Example:**

```
ikev2 profile set site-to-site auth shared-key-mic string MySecretKey123!
commit
```

### Certificate Authentication

**Syntax:**

```
ikev2 profile set <profile-name> auth rsa-sig cert-file <path>
```

**Example:**

```
ikev2 profile set site-to-site auth rsa-sig cert-file /etc/floofos/certs/router.crt
commit
```

---

## Identity Configuration

### Set Local Identity

**Syntax:**

```
ikev2 profile set <profile-name> id local <type> <value>
```

**Identity Types:**

| Type | Description | Example |
|------|-------------|---------|
| `ip4-addr` | IPv4 address | 203.0.113.1 |
| `ip6-addr` | IPv6 address | 2001:db8::1 |
| `fqdn` | Fully qualified domain name | router.example.com |
| `email` | Email address | admin@example.com |

**Example:**

```
ikev2 profile set site-to-site id local fqdn site-a.example.com
commit
```

### Set Remote Identity

**Syntax:**

```
ikev2 profile set <profile-name> id remote <type> <value>
```

**Example:**

```
ikev2 profile set site-to-site id remote fqdn site-b.example.com
commit
```

---

## Transform Set Configuration

### Configure IKE (Phase 1) Transforms

**Syntax:**

```
ikev2 profile set <profile-name> ike-crypto-alg <algorithm> ike-integ-alg <algorithm> ike-dh <group>
```

**Supported Algorithms:**

| Type | Algorithms |
|------|------------|
| **Encryption** | aes-cbc-128, aes-cbc-192, aes-cbc-256, aes-gcm-128, aes-gcm-256 |
| **Integrity** | sha1-96, sha-256-128, sha-384-192, sha-512-256 |
| **DH Group** | modp-768, modp-1024, modp-1536, modp-2048, modp-3072, modp-4096, ecp-256, ecp-384 |

**Example:**

```
ikev2 profile set site-to-site ike-crypto-alg aes-cbc-256 ike-integ-alg sha-256-128 ike-dh modp-2048
commit
```

### Configure ESP (Phase 2) Transforms

**Syntax:**

```
ikev2 profile set <profile-name> esp-crypto-alg <algorithm> esp-integ-alg <algorithm> [esp-dh <group>]
```

**Example:**

```
ikev2 profile set site-to-site esp-crypto-alg aes-cbc-256 esp-integ-alg sha-256-128
commit
```

---

## Traffic Selectors

### Configure Traffic Selectors

**Syntax:**

```
ikev2 profile set <profile-name> traffic-selector local ip-range <start-ip> - <end-ip> port-range <start-port> - <end-port> protocol <proto>
ikev2 profile set <profile-name> traffic-selector remote ip-range <start-ip> - <end-ip> port-range <start-port> - <end-port> protocol <proto>
```

**Example - All Traffic:**

```
ikev2 profile set site-to-site traffic-selector local ip-range 0.0.0.0 - 255.255.255.255 port-range 0 - 65535 protocol 0
ikev2 profile set site-to-site traffic-selector remote ip-range 0.0.0.0 - 255.255.255.255 port-range 0 - 65535 protocol 0
commit
```

**Example - Specific Networks:**

```
ikev2 profile set site-to-site traffic-selector local ip-range 192.168.1.0 - 192.168.1.255 port-range 0 - 65535 protocol 0
ikev2 profile set site-to-site traffic-selector remote ip-range 192.168.2.0 - 192.168.2.255 port-range 0 - 65535 protocol 0
commit
```

---

## Lifetime Configuration

### Configure SA Lifetimes

**Syntax:**

```
ikev2 profile set <profile-name> lifetime <seconds>
```

**Example:**

```
# Set SA lifetime to 8 hours (28800 seconds)
ikev2 profile set site-to-site lifetime 28800
commit
```

---

## Initiate IKEv2 Session

### Configure Responder (Passive)

**Syntax:**

```
ikev2 profile set <profile-name> responder
```

### Configure Initiator (Active)

**Syntax:**

```
ikev2 initiate sa-init <profile-name>
```

**Example:**

```
ikev2 initiate sa-init site-to-site
```

---

## Show IKEv2 Status

### Show IKEv2 Profiles

```
show ikev2 profile [<name>]
```

Displays authentication method, identities, IKE/ESP transforms, lifetime, and traffic selectors for each profile.

### Show IKEv2 SAs

```
show ikev2 sa
```

Displays active IKE and Child Security Associations with their encryption parameters and traffic selectors.

### Show IKEv2 Traffic

```
show ikev2 traffic
```

---

## Configuration Examples

### Example 1: Basic Site-to-Site VPN with PSK

**Site A (203.0.113.1):**

```
# Create IKEv2 profile
ikev2 profile add corporate-vpn

# Configure authentication
ikev2 profile set corporate-vpn auth shared-key-mic string SuperSecretKey2025!

# Configure identities
ikev2 profile set corporate-vpn id local fqdn site-a.company.com
ikev2 profile set corporate-vpn id remote fqdn site-b.company.com

# Configure transforms
ikev2 profile set corporate-vpn ike-crypto-alg aes-cbc-256 ike-integ-alg sha-256-128 ike-dh modp-2048
ikev2 profile set corporate-vpn esp-crypto-alg aes-cbc-256 esp-integ-alg sha-256-128

# Configure traffic selectors
ikev2 profile set corporate-vpn traffic-selector local ip-range 192.168.1.0 - 192.168.1.255 port-range 0 - 65535 protocol 0
ikev2 profile set corporate-vpn traffic-selector remote ip-range 192.168.2.0 - 192.168.2.255 port-range 0 - 65535 protocol 0

# Set lifetime
ikev2 profile set corporate-vpn lifetime 28800

commit

# Initiate connection
ikev2 initiate sa-init corporate-vpn
```

**Site B (198.51.100.1):**

```
# Create IKEv2 profile
ikev2 profile add corporate-vpn

# Configure authentication (same PSK)
ikev2 profile set corporate-vpn auth shared-key-mic string SuperSecretKey2025!

# Configure identities (swapped from Site A)
ikev2 profile set corporate-vpn id local fqdn site-b.company.com
ikev2 profile set corporate-vpn id remote fqdn site-a.company.com

# Configure transforms (must match)
ikev2 profile set corporate-vpn ike-crypto-alg aes-cbc-256 ike-integ-alg sha-256-128 ike-dh modp-2048
ikev2 profile set corporate-vpn esp-crypto-alg aes-cbc-256 esp-integ-alg sha-256-128

# Configure traffic selectors (swapped from Site A)
ikev2 profile set corporate-vpn traffic-selector local ip-range 192.168.2.0 - 192.168.2.255 port-range 0 - 65535 protocol 0
ikev2 profile set corporate-vpn traffic-selector remote ip-range 192.168.1.0 - 192.168.1.255 port-range 0 - 65535 protocol 0

# Set as responder
ikev2 profile set corporate-vpn responder

# Set lifetime
ikev2 profile set corporate-vpn lifetime 28800

commit
```

### Example 2: High-Security VPN with AES-GCM

```
# Create profile with strong encryption
ikev2 profile add high-security

# Configure authentication
ikev2 profile set high-security auth shared-key-mic string VeryLongAndComplexPSK!@#$%^&*()

# Configure strong transforms
ikev2 profile set high-security ike-crypto-alg aes-gcm-256 ike-integ-alg sha-512-256 ike-dh modp-4096
ikev2 profile set high-security esp-crypto-alg aes-gcm-256 esp-integ-alg sha-512-256

# Configure identities
ikev2 profile set high-security id local ip4-addr 203.0.113.1
ikev2 profile set high-security id remote ip4-addr 198.51.100.1

# Configure traffic selectors
ikev2 profile set high-security traffic-selector local ip-range 0.0.0.0 - 255.255.255.255 port-range 0 - 65535 protocol 0
ikev2 profile set high-security traffic-selector remote ip-range 0.0.0.0 - 255.255.255.255 port-range 0 - 65535 protocol 0

# Shorter lifetime for higher security
ikev2 profile set high-security lifetime 3600

commit
```

---

## Best Practices

### Security Recommendations

| Recommendation | Description |
|----------------|-------------|
| **Use AES-256** | Minimum 256-bit encryption for production |
| **Use SHA-256+** | Avoid SHA-1 for new deployments |
| **Use DH 2048+** | Minimum 2048-bit Diffie-Hellman |
| **Strong PSK** | Use complex, long pre-shared keys |
| **Rotate Keys** | Change PSK periodically |

### Recommended Algorithm Sets

**Standard Security:**
```
IKE: aes-cbc-256, sha-256-128, modp-2048
ESP: aes-cbc-256, sha-256-128
```

**High Security:**
```
IKE: aes-gcm-256, sha-512-256, modp-4096
ESP: aes-gcm-256, sha-512-256
```

---

## Troubleshooting

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| SA not establishing | PSK mismatch | Verify PSK on both ends |
| Transform mismatch | Algorithm mismatch | Ensure same algorithms on both sides |
| Traffic selector mismatch | TS configuration error | Check local/remote TS are swapped correctly |
| NAT traversal issues | Firewall blocking | Open UDP 500 and 4500 |

### Debug Commands

```
# Show IKEv2 SA status
show ikev2 sa

# Show profile configuration
show ikev2 profile corporate-vpn

# Check IPsec SA
show ipsec sa
```

---

## Related Documentation

- [IPsec VPN Configuration](ipsec.md)
- [WireGuard VPN](wireguard.md)
- [Firewall Configuration](../security/firewall.md)

## External References

- [FD.io VPP IKEv2 CLI Reference](https://docs.fd.io/vpp/25.10/cli-reference/clis/clicmd_src_plugins_ikev2.html){:target="_blank"}
- [RFC 7296 - IKEv2 Protocol](https://datatracker.ietf.org/doc/html/rfc7296){:target="_blank"}
