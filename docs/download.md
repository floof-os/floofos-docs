# Download

Download the latest FloofOS release for your platform.

---

## FloofOS 1.0 — Stable

!!! success "Current Stable Release — December 2025"
    The first stable release of FloofOS featuring VPP 25.10 dataplane, BIRD2 routing, and floofctl management CLI.

[Download FloofOS 1.0 ISO (~1.0 GB)](https://cdn.floofos.io/FloofOS-1.0.iso){ .md-button .md-button--primary }
[Release Notes](changelog/1.0.0.md){ .md-button }

??? note "Checksums"
    **SHA256**
    ```
    <checksum will be added after build>
    ```
    **MD5**
    ```
    <checksum will be added after build>
    ```

??? note "Verify Download"
    === "Linux/macOS"

        ```bash
        sha256sum FloofOS-1.0.iso
        md5sum FloofOS-1.0.iso
        ```

    === "Windows (PowerShell)"

        ```powershell
        Get-FileHash FloofOS-1.0.iso -Algorithm SHA256
        Get-FileHash FloofOS-1.0.iso -Algorithm MD5
        ```

---

## System Requirements

### Minimum

- **CPU:** 2 cores (64-bit x86_64)
- **RAM:** 2 GB
- **Storage:** 8 GB
- **NIC:** Intel/Mellanox DPDK-compatible

### Recommended for Production

- **CPU:** 8+ cores with AES-NI
- **RAM:** 16 GB+
- **Storage:** 32 GB SSD
- **NIC:** Intel X710/XXV710, Mellanox ConnectX-4+

---

## Quick Install

1. Write ISO to USB or mount in VM
2. Boot from ISO
3. Follow the installer prompts
4. Reboot and configure via `floofctl`

For detailed instructions, see the [Installation Guide](getting-started/installation.md).

---

## Source Code

FloofOS is open source:

- [floofos](https://github.com/floof-os/floofos) — Main repository
- [floofctl](https://github.com/floof-os/floofctl) — CLI tool
- [floofos-build](https://github.com/floof-os/floofos-build) — ISO build system
