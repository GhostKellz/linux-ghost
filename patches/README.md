# Linux Ghost Patches

This directory contains custom/modified kernel patches maintained by linux-ghost.
Upstream patches from CachyOS and linux-tkg are fetched at build time from their respective repos.

## Local Patches

| Patch | Source | Description |
|-------|--------|-------------|
| `ghost-sched-v2-linux7.patch` | linux-ghost | GHOST Scheduler v2 on EEVDF (gaming-optimized, Zen4/Zen5 tuned) |
| `0003-glitched-base.patch` | linux-tkg (modified) | Zenify gaming/low-latency tuning, fixed for CachyOS 7.0 pre-patched source |
| `ghostzen5.patch` | linux-ghost | CONFIG_MZEN5 support (-march=znver5) for Ryzen 9000 series |
| `0012-misc-additions.patch` | linux-tkg (modified) | Magic Trackpad 2 fix, ondemand governor fix, max ASLR bits |

## Remote Patches (fetched at build time)

| Patch | Source | Condition |
|-------|--------|-----------|
| `0001-bore-cachy.patch` | CachyOS | `_cpusched=bore` |
| `0003-glitched-eevdf-additions.patch` | linux-tkg | `_zenify=yes` (if available upstream) |
| `0014-OpenRGB.patch` | linux-tkg | `_openrgb=yes` |
| `0006-add-acs-overrides_iommu.patch` | linux-tkg | `_acs_override=yes` |
| `dkms-clang.patch` | CachyOS | `_compiler=llvm` |

## Patch Application Order

PKGBUILD applies patches in source order:
1. BORE or GHOST scheduler (depending on `_cpusched`)
2. Zenify gaming patches (if `_zenify=yes`)
3. Zen5 MZEN5 Kconfig patch
4. Misc TKG additions
5. OpenRGB (if `_openrgb=yes`)
6. ACS Override (if `_acs_override=yes`)
7. DKMS clang fix (if `_compiler=llvm`)

## References

- [CachyOS kernel-patches](https://github.com/CachyOS/kernel-patches)
- [linux-tkg patches](https://github.com/Frogging-Family/linux-tkg/tree/master/linux-tkg-patches)
- [BORE Scheduler](https://github.com/firelzrd/bore-scheduler)
