# linux-ghost Documentation

Documentation for linux-ghost, an Arch-oriented custom kernel package for AMD Zen/X3D desktops, NVIDIA RTX systems, sched-ext experimentation, and low-latency gaming workloads.

## Quick Start

New users should start here:

1. [Project README](../README.md) - overview, install flow, and common commands
2. [Build options](getting-started/build-options.md) - `makepkg` environment flags
3. [Kernel configuration](kernel/configuration.md) - `ghost.fragment` and config policy
4. [Scheduler guide](kernel/schedulers.md) - GHOST, BORE, EEVDF, and sched-ext
5. [NVIDIA guide](hardware/nvidia.md) - DKMS, bundled modules, and Blackwell notes

Current package baseline: Linux 7.1.x (CachyOS 7.1.0-1 stable).

## Documentation Index

### Getting Started

Build and packaging workflow.

| Document | Description |
|----------|-------------|
| [getting-started/build-options.md](getting-started/build-options.md) | Build-time environment options for `makepkg` |

### Kernel

Kernel config, scheduler behavior, and maintained patch details.

| Document | Description |
|----------|-------------|
| [kernel/configuration.md](kernel/configuration.md) | `config/ghost.fragment`, kernel option groups, and verification |
| [kernel/schedulers.md](kernel/schedulers.md) | Built-in schedulers, sched-ext profiles, and tuning |
| [../patches/README.md](../patches/README.md) | Local and remote patch inventory |

### Hardware

Hardware-specific setup and compatibility notes.

| Document | Description |
|----------|-------------|
| [hardware/nvidia.md](hardware/nvidia.md) | NVIDIA open modules, bundled module option, and troubleshooting |
| [../nvidia/README.md](../nvidia/README.md) | NVIDIA patch inventory and modprobe config details |
| [../sched-ext/README.md](../sched-ext/README.md) | Installed sched-ext services and `ghost-vcache` helper |

### Development

Project maintenance and contributor workflow.

| Document | Description |
|----------|-------------|
| [development/README.md](development/README.md) | Maintainer workflow and release prep checklist |
| [../CONTRIBUTING.md](../CONTRIBUTING.md) | Contribution guidelines |
| [../SECURITY.md](../SECURITY.md) | Security policy |
| [../CHANGELOG.md](../CHANGELOG.md) | Release history |

## Repository Map

| Path | Purpose |
|------|---------|
| `PKGBUILD` | Arch package build script and build option defaults |
| `config/` | Base kernel config plus linux-ghost overlay fragment |
| `patches/` | linux-ghost maintained kernel patches |
| `nvidia/` | Optional NVIDIA open module patches and modprobe config |
| `sched-ext/` | systemd profiles and AMD V-Cache helper |
| `docs/` | User, maintainer, and hardware documentation |

## Maintenance Notes

When preparing a kernel 7.1 update, keep these documents in sync:

- `PKGBUILD`: `_major`, `_minor`, `_cachyos_tagrel`, source URLs, NVIDIA reference version
- `CHANGELOG.md`: new release section and migration notes
- `SECURITY.md`: supported version table
- `docs/kernel/configuration.md`: config deltas and removed/renamed Kconfig symbols
- `docs/kernel/schedulers.md`: scheduler patch status for the target kernel
- `docs/getting-started/build-options.md`: supported build flags and defaults
