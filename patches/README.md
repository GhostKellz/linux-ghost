# Linux Ghost Patches

This directory contains custom/modified kernel patches maintained by linux-ghost.
Upstream patches from CachyOS and linux-tkg are fetched at build time from their respective repos.

## Local Patches

| Patch | Source | Description |
|-------|--------|-------------|
| `ghost-sched-v2-linux7.patch` | linux-ghost | GHOST Scheduler v2 on EEVDF (gaming-optimized, Zen4/Zen5 tuned), rebased for 7.1 |
| `0003-glitched-base.patch` | linux-tkg (modified) | Zenify gaming/low-latency tuning, fixed for the current CachyOS pre-patched source |
| `0003-glitched-eevdf-additions.patch` | linux-ghost (rebased from linux-tkg) | Zenify EEVDF/VM tunables (cfs_bandwidth_slice, NR_MIGRATE_BREAK, energy_aware, dirty ratios); scheduler-independent |
| `0003-glitched-eevdf-migration-cost.patch` | linux-ghost | Zenify `sched_migration_cost` 250us override for the CACHY layout (`_cpusched=ghost`/`eevdf`) |
| `0003-glitched-eevdf-migration-cost-bore.patch` | linux-ghost | Zenify `sched_migration_cost` 250us override for the BORE layout (`_cpusched=bore`) |
| `ghostzen5.patch` | linux-ghost | CONFIG_MZEN5 support (-march=znver5) for Ryzen 9000 series |
| `0012-misc-additions.patch` | linux-tkg (modified) | Magic Trackpad 2 fix, ondemand governor fix, max ASLR bits |
| `bbr3-default.patch` | linux-ghost | Adds the `DEFAULT_BBR3` Kconfig choice so BBR3 can be the compile-time `DEFAULT_TCP_CONG` |

## Remote Patches (fetched at build time)

| Patch | Source | Condition |
|-------|--------|-----------|
| `0001-bore-cachy.patch` | CachyOS | `_cpusched=bore` |
| `0014-OpenRGB.patch` | linux-tkg | `_openrgb=yes` |
| `0006-add-acs-overrides_iommu.patch` | linux-tkg | `_acs_override=yes` |
| `dkms-clang.patch` | CachyOS | `_compiler=llvm` |

## Patch Application Order

PKGBUILD applies patches in source order:
1. BORE or GHOST scheduler (depending on `_cpusched`)
2. Zenify base + EEVDF additions + scheduler-specific migration_cost variant (if `_zenify=yes`)
3. Zen5 MZEN5 Kconfig patch
4. Misc TKG additions
5. BBR3 default choice (if `_tcp_bbr3=yes`)
6. OpenRGB (if `_openrgb=yes`)
7. ACS Override (if `_acs_override=yes`)
8. DKMS clang fix (if `_compiler=llvm`)

## References

- [CachyOS kernel-patches](https://github.com/CachyOS/kernel-patches)
- [linux-tkg patches](https://github.com/Frogging-Family/linux-tkg/tree/master/linux-tkg-patches)
- [BORE Scheduler](https://github.com/firelzrd/bore-scheduler)
