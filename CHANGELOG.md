# Changelog

All notable changes to linux-ghost are documented here.

## [7.0.5-1] - 2026-05-11

### Added
- GHOST Scheduler v2 for Linux 7.0 (`ghost-sched-v2-linux7.patch`)
- BORE scheduler as an optional alternative (`_cpusched=bore`)
- `kernel.fragment` config overlay system (`config/ghost.fragment`)
- Zenify gaming patch fixed for CachyOS 7.0 pre-patched source
- `0012-misc-additions.patch` (Magic Trackpad 2, ondemand governor fix, max ASLR bits)
- `ghostzen5.patch` for CONFIG_MZEN5 / `-march=znver5` on CachyOS 7.0
- Git init in `prepare()` for git-format patch compatibility

### Changed
- **Kernel 7.0.5** (Linux 7.0 follows 6.19 - no 6.20)
- Base source switched to CachyOS pre-patched tarball (`cachyos-7.0.5-1`)
- NVIDIA reference version bumped to 595.71.05
- CI workflows updated for `src/cachyos-*/` source directory naming
- Release notes updated for GHOST v2, Zenify, PREEMPT_LAZY, -O3
- Patches reorganized into `patches/` directory with GitHub raw URL references
- Config updated for kernel 7.0

### Removed
- `_kernel_type` stable/rc toggle (7.0 is now stable)
- `_clear_patches` option (Intel Clear Linux patches)
- Stale local copies of upstream patches (BORE, dkms-clang, ACS override, OpenRGB, cachyos-base-all) — now fetched from CachyOS/TKG at build time
- Old GHOST scheduler v1 patch (`0001-ghost-sched.patch`)

## [6.19.11-1] - 2025-04-09

### Changed
- Upgraded to kernel 6.19.11
- Switched to CachyOS pre-patched source tarball

## [6.19.10-1] - 2025-03-28

### Changed
- Upgraded to kernel 6.19.10
- NVIDIA bumped to 595.58.03

## [6.19.6-1] - 2025-03-15

### Changed
- Upgraded to kernel 6.19.6
- NVIDIA bumped to 595.45.04

## [6.19.2-1] - 2025-02-28

### Changed
- Upgraded to kernel 6.19.2

## [6.19.0-1] - 2025-02-15

### Added
- Initial kernel 6.19 release
- GHOST Scheduler v1 on EEVDF
- AMD Zen5/X3D optimizations
- sched-ext support
- Full LTO with LLVM/Clang
- NVIDIA RTX 5090 (Blackwell) support via optional bundled modules
- Zenify gaming patches from linux-tkg

## [6.18.4-1] - 2025-01-30

### Changed
- Bumped to kernel 6.18.4

## [6.18.3-1] - 2025-01-25

### Added
- Nightly CI: auto-detect new kernel releases and build
- Zen 5 (MZEN5) support and PCIe optimizations

### Fixed
- Disabled duplicate ACS override patch (already in CachyOS base)
