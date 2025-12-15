<p align="center">
  <img src="assets/logo/linux-ghost.png" alt="Linux Ghost" width="400">
</p>

<p align="center">
  <a href="#amd-zenx3d"><img src="https://img.shields.io/badge/AMD%20Ryzen-X3D-ED1C24?style=for-the-badge&logo=amd&logoColor=white" alt="AMD Ryzen X3D"></a>
  <a href="#nvidia-rtx-20"><img src="https://img.shields.io/badge/NVIDIA-RTX-76B900?style=for-the-badge&logo=nvidia&logoColor=white" alt="NVIDIA RTX"></a>
  <a href="#nvidia-rtx-20"><img src="https://img.shields.io/badge/nvidia--open-580+-00D4AA?style=for-the-badge&logo=nvidia&logoColor=white" alt="nvidia-open"></a>
</p>

<p align="center">
  <a href="#cpu--scheduling"><img src="https://img.shields.io/badge/BORE-EEVDF-8B5CF6?style=for-the-badge" alt="BORE EEVDF"></a>
  <a href="#sched-ext-gaming"><img src="https://img.shields.io/badge/sched--ext-scx__lavd-3B82F6?style=for-the-badge" alt="sched-ext"></a>
  <a href="#performance"><img src="https://img.shields.io/badge/LTO-Full-F97316?style=for-the-badge" alt="LTO"></a>
</p>

<p align="center">
  <a href="#build-options"><img src="https://img.shields.io/badge/Clang-19-5C6BC0?style=for-the-badge&logo=llvm&logoColor=white" alt="Clang"></a>
  <a href="#build-options"><img src="https://img.shields.io/badge/GCC-14%2F15-D32F2F?style=for-the-badge&logo=gnu&logoColor=white" alt="GCC"></a>
  <a href="#quick-start"><img src="https://img.shields.io/badge/Arch-Linux-1793D1?style=for-the-badge&logo=archlinux&logoColor=white" alt="Arch Linux"></a>
  <a href="#"><img src="https://img.shields.io/badge/Linux-6.18+-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Kernel 6.18+"></a>
</p>

---

> [!WARNING]
> **Experimental Project** — This kernel is currently in active development and should be considered a concept/preview. It is **not production ready** and may cause system instability. Use at your own risk. Contributions and testing feedback welcome.

---

## What is Linux Ghost?

A custom Linux kernel built for enthusiasts running AMD Ryzen X3D processors (9950X3D, 7950X3D) with NVIDIA GPUs. Combines the best of CachyOS and linux-tkg with exclusive optimizations for gaming, AI workloads, and desktop performance.

### Key Differentiators

| | Linux Ghost | CachyOS | linux-tkg |
|:--|:--:|:--:|:--:|
| **AMD 3D V-Cache Optimizer** | **Yes** | No | No |
| **Thunderbolt eGPU Hotplug** | **Yes** | No | No |
| **NVIDIA GSP Stutter Fix** | **Yes** | No | No |
| **X3D-First Tuning** | **Yes** | Generic | Generic |
| **Pre-configured sched-ext** | **Yes** | Manual | No |

---

## Features

### CPU & Scheduling
- **BORE on EEVDF** — Burst-oriented response enhancer for snappy desktop feel
- **sched-ext Ready** — Runtime scheduler switching with scx_lavd (gaming) or scx_bpfland (desktop)
- **AMD 3D V-Cache Optimizer** — Automatic cache/frequency CCD preference for X3D chips
- **1000Hz Full Tickless** — Ultra-low latency configuration

### AMD Zen5/X3D
- **CONFIG_AMD_3D_VCACHE** — V-Cache mode switching via sysfs
- **AMD P-State EPP** — Energy performance preference with preferred core ranking
- **NUMA Balancing** — Optimized for chiplet architecture
- **Native CPU Detection** — Auto-tuned for your specific processor

### NVIDIA (RTX 20+)
- **nvidia-open Ready** — Kernel configured for open modules (Blackwell/RTX 5090 supported)
- **Exclusive Patches** — Thunderbolt eGPU hotplug, UVM null deref fix, GSP stutter mitigation
- **DRM/fbdev** — Proper Wayland and 6.11+ kernel support

### Performance
- **Full Clang LTO** — Link-time optimization for better codegen
- **BBR3** — Modern TCP congestion control
- **Transparent Hugepages** — Always-on for gaming/large workloads
- **O3 Optimization** — Aggressive compiler optimizations

---

## Quick Start

### Prerequisites

```bash
# Arch Linux build dependencies
sudo pacman -S base-devel bc cpio gettext libelf pahole perl python \
    tar xz zstd clang llvm lld rust rust-bindgen rust-src
```

### Build & Install

```bash
git clone https://github.com/ghostkellz/linux-ghost.git
cd linux-ghost

# Build with defaults (LLVM, Full LTO, BORE, native CPU)
makepkg -sf

# Install
sudo pacman -U linux-ghost-*.pkg.tar.zst

# Regenerate initramfs
sudo mkinitcpio -P

# Reboot and select linux-ghost in bootloader
```

### NVIDIA Setup

```bash
# Recommended: Use DKMS (works with multiple kernels)
sudo pacman -S nvidia-open-dkms nvidia-utils lib32-nvidia-utils

# Copy optimized config
sudo cp nvidia/nvidia-ghost.conf /etc/modprobe.d/
sudo mkinitcpio -P
```

---

## Build Options

```bash
# Compiler: llvm (default) or gcc
_compiler=gcc makepkg -sf

# LTO: full (default), thin, or none
_lto_mode=thin makepkg -sf

# Scheduler: bore (default) or eevdf
_cpusched=eevdf makepkg -sf

# CPU target: native (default), zen5, zen4, generic
_processor_opt=zen5 makepkg -sf

# Bundle NVIDIA modules (default: no, use DKMS)
_nvidia_bundle=yes makepkg -sf

# RC kernel (6.19-rc)
_kernel_type=rc makepkg -sf
```

---

## sched-ext Gaming

Linux Ghost includes pre-configured systemd services for sched-ext schedulers.

```bash
# Install schedulers
sudo pacman -S scx-scheds

# Gaming mode (scx_lavd - latency optimized)
sudo systemctl start scx-ghost-gaming

# Desktop mode (scx_bpfland - balanced)
sudo systemctl start scx-ghost-desktop

# Enable on boot
sudo systemctl enable scx-ghost-gaming
```

### AMD V-Cache Switching (X3D Only)

```bash
# Check current mode
ghost-vcache status

# Gaming (prefer V-Cache CCD)
sudo ghost-vcache cache

# Productivity (prefer frequency CCD)
sudo ghost-vcache frequency
```

> **Note:** Requires BIOS setting: CBS → CPU Common → CPPC → **Driver**

---

## Package Output

| Package | Description |
|---------|-------------|
| `linux-ghost` | Kernel and modules |
| `linux-ghost-headers` | Headers for DKMS/module building |
| `linux-ghost-nvidia-open` | Bundled NVIDIA modules (optional) |

---

## Configuration

### Kernel Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| Tick Rate | 1000Hz | Low latency |
| Tick Type | Full Tickless | Gaming/interactive |
| Preemption | Full (PREEMPT_LAZY) | Responsiveness |
| THP | Always | Memory performance |
| Governor | Performance | Maximum clocks |
| TCP | BBR3 | Network performance |
| LTO | Full Clang | Code optimization |

### ghost.fragment Highlights

```
CONFIG_AMD_3D_VCACHE=y          # V-Cache optimizer
CONFIG_SCHED_CLASS_EXT=y        # sched-ext support
CONFIG_PREEMPT_LAZY=y           # New lazy preemption
CONFIG_AMD_PSTATE_PREFCORE=y    # Preferred core ranking
CONFIG_NTSYNC=m                 # Wine/Proton sync
```

---

## Project Structure

```
linux-ghost/
├── PKGBUILD                 # Main build script
├── linux-ghost.install      # Pacman hooks
├── config/
│   └── ghost.fragment       # Kernel config overrides
├── nvidia/
│   ├── *.patch              # NVIDIA patches (inc. exclusives)
│   ├── nvidia-ghost.conf    # Modprobe config
│   └── README.md
├── sched-ext/
│   ├── scx-ghost-gaming.service
│   ├── scx-ghost-desktop.service
│   ├── ghost-vcache          # V-Cache CLI tool
│   └── README.md
├── patches/
│   └── README.md
└── assets/
    └── logo/
```

---

## Troubleshooting

<details>
<summary><b>Build fails with missing dependencies</b></summary>

```bash
sudo pacman -S base-devel bc cpio gettext libelf pahole perl python \
    tar xz zstd clang llvm lld rust rust-bindgen rust-src
```
</details>

<details>
<summary><b>NVIDIA module version mismatch</b></summary>

```bash
# Check versions match
pacman -Q nvidia-utils nvidia-open-dkms
# Both should be same version (e.g., 580.105.08)
```
</details>

<details>
<summary><b>sched-ext not loading</b></summary>

```bash
# Verify kernel support
zcat /proc/config.gz | grep SCHED_CLASS_EXT
# Should show: CONFIG_SCHED_CLASS_EXT=y

# Check scx status
cat /sys/kernel/sched_ext/state
```
</details>

<details>
<summary><b>V-Cache mode not changing</b></summary>

1. Verify BIOS: CBS → CPU Common → CPPC → **Driver**
2. Check driver: `ls /sys/bus/platform/drivers/amd_x3d_vcache/`
3. Must be X3D processor (9950X3D, 7950X3D, etc.)
</details>

---

## Credits

- [CachyOS](https://github.com/CachyOS/linux-cachyos) — Base patches and configuration
- [linux-tkg](https://github.com/Frogging-Family/linux-tkg) — Zenify patches and inspiration
- [BORE Scheduler](https://github.com/firelzrd/bore-scheduler) — BORE patches
- [sched-ext](https://github.com/sched-ext/scx) — Extensible scheduler framework
- [NVIDIA open-gpu-kernel-modules](https://github.com/NVIDIA/open-gpu-kernel-modules) — Community patches

---

<p align="center">
  <i>Built for enthusiasts who demand more from their hardware.</i>
</p>
