# Linux Ghost Patches

This directory contains kernel patches for linux-ghost.

## Included Patches

| Patch | Size | Source | Description |
|-------|------|--------|-------------|
| `0001-cachyos-base-all.patch` | 1.2M | CachyOS | Core patches: sched-ext, AMD P-State, BBR3, performance opts |
| `0001-bore-cachy.patch` | 36K | CachyOS | BORE (Burst-Oriented Response Enhancer) on EEVDF |
| `0003-glitched-base.patch` | 21K | linux-tkg | Zenify gaming/low-latency tuning |
| `0006-add-acs-overrides_iommu.patch` | 6.6K | linux-tkg | ACS override for VFIO/GPU passthrough |
| `0014-OpenRGB.patch` | 18K | linux-tkg | OpenRGB i2c controller support |
| `dkms-clang.patch` | 2K | CachyOS | DKMS compatibility for clang-built kernels |

## What Each Patch Does

### 0001-cachyos-base-all.patch (CachyOS Base)

Comprehensive patch bundle containing:
- **sched-ext support** - BPF extensible scheduler framework
- **AMD P-State improvements** - Better frequency scaling for Zen processors
- **BBR3** - Google's TCP congestion control algorithm
- **Block layer optimizations** - I/O scheduler improvements
- **Memory management** - THP and MGLRU enhancements
- **Hardware enablement** - Latest AMD/Intel platform support

### 0001-bore-cachy.patch (BORE Scheduler)

Burst-Oriented Response Enhancer built on EEVDF:
- Detects and prioritizes bursty interactive tasks
- Improves desktop responsiveness and gaming frame pacing
- Maintains fairness for background workloads

### 0003-glitched-base.patch (Zenify)

Gaming-focused tunings from linux-tkg/Liquorix:
- Reduced scheduling latency
- Optimized timer handling
- Better context switch behavior for games

### 0006-add-acs-overrides_iommu.patch (ACS Override)

Enables IOMMU group separation for VFIO:
- Required for single-GPU passthrough
- Allows splitting PCIe devices into separate IOMMU groups
- Essential for VM gaming setups

### 0014-OpenRGB.patch

I2C/SMBus controller access for RGB control:
- Exposes motherboard SMBus to userspace
- Required for OpenRGB to control RGB lighting

### dkms-clang.patch

Removes strict Werror flags for DKMS compatibility:
- Allows building out-of-tree modules with clang kernels
- Fixes nvidia-dkms and other DKMS modules on LLVM builds

## Patch Application Order

PKGBUILD applies patches in this order:
1. `0001-cachyos-base-all.patch` (always)
2. `0001-bore-cachy.patch` (if `_cpusched=bore`)
3. `0003-glitched-base.patch` (if `_zenify=yes`)
4. `0006-add-acs-overrides_iommu.patch` (if `_acs_override=yes`)
5. `0014-OpenRGB.patch` (if `_openrgb=yes`)
6. `dkms-clang.patch` (if `_compiler=llvm`)

## Using Local Patches

By default, PKGBUILD downloads patches from upstream sources. To use local patches instead:

```bash
_use_local_patches=yes makepkg -sf
```

## Adding Custom Patches

1. Place `.patch` file in this directory
2. Add to `source=()` in PKGBUILD
3. Run `updpkgsums` to update checksums
4. Build with `makepkg -sf`

### Naming Convention

```
XXXX-category-description.patch
```

| Range | Category |
|-------|----------|
| 0001-0099 | Core/CachyOS patches |
| 0100-0199 | CPU/Scheduler |
| 0200-0299 | GPU/Display |
| 0300-0399 | Storage/FS |
| 0400-0499 | Network |
| 0500+ | Experimental |

## Updating Patches

To refresh patches from upstream:

```bash
# CachyOS patches
curl -LO "https://raw.githubusercontent.com/cachyos/kernel-patches/master/6.18/all/0001-cachyos-base-all.patch"
curl -LO "https://raw.githubusercontent.com/cachyos/kernel-patches/master/6.18/sched/0001-bore-cachy.patch"

# linux-tkg patches
curl -LO "https://raw.githubusercontent.com/Frogging-Family/linux-tkg/master/linux-tkg-patches/6.18/0003-glitched-base.patch"
```

## References

- [CachyOS kernel-patches](https://github.com/CachyOS/kernel-patches)
- [linux-tkg patches](https://github.com/Frogging-Family/linux-tkg/tree/master/linux-tkg-patches)
- [BORE Scheduler](https://github.com/firelzrd/bore-scheduler)
