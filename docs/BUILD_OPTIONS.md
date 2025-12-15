# Linux Ghost Build Options

## Overview

All build options are configured via environment variables before running `makepkg`.

```bash
# Example: Build with GCC, thin LTO, EEVDF scheduler
_compiler=gcc _lto_mode=thin _cpusched=eevdf makepkg -sf
```

---

## Kernel Version

### `_kernel_type`

| Value | Description |
|-------|-------------|
| `stable` | Stable release (6.18.x) - **default** |
| `rc` | Release candidate (6.19-rcX) |

```bash
# Build RC kernel
_kernel_type=rc makepkg -sf
```

---

## Compiler

### `_compiler`

| Value | Description |
|-------|-------------|
| `llvm` | Clang/LLVM toolchain - **default** |
| `gcc` | GCC toolchain |

```bash
# Build with GCC
_compiler=gcc makepkg -sf
```

### `_lto_mode`

| Value | LLVM | GCC | Description |
|-------|------|-----|-------------|
| `full` | Full LTO | GCC LTO | Best performance, slow build - **default** |
| `thin` | ThinLTO | Falls back to full | Faster build, good performance |
| `none` | Disabled | Disabled | Fastest build, no LTO |

```bash
# Faster build with ThinLTO
_lto_mode=thin makepkg -sf
```

---

## CPU Scheduler

### `_cpusched`

| Value | Description |
|-------|-------------|
| `bore` | BORE on EEVDF - **default** |
| `eevdf` | Vanilla EEVDF |

```bash
# Vanilla EEVDF
_cpusched=eevdf makepkg -sf
```

---

## CPU Optimizations

### `_processor_opt`

| Value | Description |
|-------|-------------|
| `native` | Auto-detect CPU (build machine) - **default** |
| `zen5` | AMD Zen5 (9950X3D, 9900X3D) |
| `zen4` | AMD Zen4 (7950X3D, 7900X3D) |
| `generic` | Generic x86-64 |

```bash
# Force Zen5 optimizations
_processor_opt=zen5 makepkg -sf
```

### `_zen5_x3d`

| Value | Description |
|-------|-------------|
| `yes` | Enable X3D optimizations - **default** |
| `no` | Disable |

Enables NUMA balancing, AMD platform device, P-State, high core count support.

---

## Performance Options

### `_zenify`

| Value | Description |
|-------|-------------|
| `yes` | Enable Zenify gaming patches - **default** |
| `no` | Disable |

Gaming/low-latency optimizations from Zen/Liquorix.

### `_cc_harder`

| Value | Description |
|-------|-------------|
| `yes` | Enable -O3 optimization - **default** |
| `no` | Use default -O2 |

### `_per_gov`

| Value | Description |
|-------|-------------|
| `yes` | Performance governor default - **default** |
| `no` | Schedutil governor default |

### `_tcp_bbr3`

| Value | Description |
|-------|-------------|
| `yes` | Enable BBR3 TCP congestion - **default** |
| `no` | Use CUBIC |

---

## Timing

### `_HZ_ticks`

| Value | Description |
|-------|-------------|
| `1000` | 1000Hz - **default** (gaming) |
| `750` | 750Hz |
| `500` | 500Hz |
| `300` | 300Hz |
| `250` | 250Hz |
| `100` | 100Hz (server) |

### `_tickrate`

| Value | Description |
|-------|-------------|
| `full` | Full tickless - **default** |
| `idle` | Idle tickless |
| `perodic` | Periodic ticks |

### `_preempt`

| Value | Description |
|-------|-------------|
| `full` | Full preemption - **default** |
| `lazy` | Lazy preemption (PREEMPT_LAZY) |
| `voluntary` | Voluntary preemption |
| `none` | No preemption |

---

## Memory

### `_hugepage`

| Value | Description |
|-------|-------------|
| `always` | THP always - **default** |
| `madvise` | THP on madvise only |

---

## Features

### `_sched_ext`

| Value | Description |
|-------|-------------|
| `yes` | Enable sched-ext support - **default** |
| `no` | Disable |

### `_openrgb`

| Value | Description |
|-------|-------------|
| `yes` | Enable OpenRGB i2c support - **default** |
| `no` | Disable |

### `_acs_override`

| Value | Description |
|-------|-------------|
| `yes` | Enable ACS override for VFIO - **default** |
| `no` | Disable |

### `_clear_patches`

| Value | Description |
|-------|-------------|
| `no` | Disable Clear Linux patches - **default** |
| `yes` | Enable (Intel optimizations) |

---

## NVIDIA

### `_nvidia_bundle`

| Value | Description |
|-------|-------------|
| `no` | Use nvidia-open-dkms - **default** |
| `yes` | Bundle nvidia-open modules |

```bash
# Bundle NVIDIA modules
_nvidia_bundle=yes makepkg -sf
```

---

## Interactive Config

### `_makenconfig`

| Value | Description |
|-------|-------------|
| `no` | Skip nconfig - **default** |
| `yes` | Run `make nconfig` before build |

### `_makexconfig`

| Value | Description |
|-------|-------------|
| `no` | Skip xconfig - **default** |
| `yes` | Run `make xconfig` before build |

### `_localmodcfg`

| Value | Description |
|-------|-------------|
| `no` | Build all modules - **default** |
| `yes` | Build only loaded modules |

Requires modprobed-db. Set path with `_localmodcfg_path`.

### `_use_current`

| Value | Description |
|-------|-------------|
| `no` | Use provided config - **default** |
| `yes` | Use running kernel's config |

---

## Debug

### `_build_debug`

| Value | Description |
|-------|-------------|
| `no` | No debug package - **default** |
| `yes` | Build debug package with unstripped vmlinux |

---

## Example Builds

```bash
# Gaming build (defaults)
makepkg -sf

# Server build
_cpusched=eevdf _HZ_ticks=100 _tickrate=idle _per_gov=no makepkg -sf

# Fast test build
_lto_mode=none _build_debug=no makepkg -sf

# Full GCC build
_compiler=gcc _lto_mode=full makepkg -sf

# Zen5 with bundled NVIDIA
_processor_opt=zen5 _nvidia_bundle=yes makepkg -sf
```
