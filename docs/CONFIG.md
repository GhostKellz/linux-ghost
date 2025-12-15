# Linux Ghost Configuration Guide

## Overview

Linux Ghost uses a config fragment system (`ghost.fragment`) to override the base CachyOS kernel config with Ghost-specific optimizations.

## ghost.fragment

Located at `config/ghost.fragment`, this file contains kernel config options merged on top of the base config during build.

### How It Works

```bash
# During prepare(), the PKGBUILD runs:
scripts/kconfig/merge_config.sh -m .config ghost.fragment
```

Options in ghost.fragment override the base config values.

---

## Configuration Categories

### Scheduler & Performance

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_SCHED_CLASS_EXT` | y | Enable sched-ext BPF scheduler framework |
| `CONFIG_BPF_JIT` | y | BPF just-in-time compilation |
| `CONFIG_BPF_JIT_ALWAYS_ON` | y | Always-on JIT for sched-ext performance |
| `CONFIG_HIGH_RES_TIMERS` | y | Microsecond precision timers |

### AMD Zen5/X3D Optimizations

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_X86_AMD_PLATFORM_DEVICE` | y | AMD platform driver support |
| `CONFIG_AMD_NB` | y | AMD northbridge support |
| `CONFIG_AMD_IOMMU` | y | AMD IOMMU (for VFIO passthrough) |
| `CONFIG_X86_AMD_PSTATE` | y | AMD P-State frequency driver |
| `CONFIG_X86_AMD_PSTATE_DEFAULT_MODE` | 3 | Guided autonomous mode |
| `CONFIG_AMD_3D_VCACHE` | y | **3D V-Cache optimizer for X3D CPUs** |
| `CONFIG_AMD_PSTATE_PREFCORE` | y | Preferred core ranking |
| `CONFIG_NUMA` | y | NUMA support for chiplet design |
| `CONFIG_NUMA_BALANCING` | y | Automatic NUMA page migration |
| `CONFIG_NR_CPUS` | 64 | Support up to 32 cores / 64 threads |

#### AMD P-State Modes

| Mode | Value | Description |
|------|-------|-------------|
| Disabled | 0 | P-State driver disabled |
| Passive | 1 | OS controls frequency |
| Active | 2 | Firmware controls frequency |
| **Guided** | **3** | Firmware-guided with OS hints (default) |

### 3D V-Cache Optimizer

The `CONFIG_AMD_3D_VCACHE=y` option enables runtime switching between cache-optimized and frequency-optimized CCDs on X3D processors:

```bash
# Check current mode
cat /sys/bus/platform/drivers/amd_x3d_vcache/mode

# Gaming (prefer V-Cache CCD)
echo cache | sudo tee /sys/bus/platform/drivers/amd_x3d_vcache/mode

# Productivity (prefer frequency CCD)
echo frequency | sudo tee /sys/bus/platform/drivers/amd_x3d_vcache/mode
```

**Supported CPUs:** 9950X3D, 9900X3D, 7950X3D, 7900X3D, 7800X3D

**BIOS Requirement:** CBS → CPU Common → CPPC → **Driver**

### Memory Management

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_TRANSPARENT_HUGEPAGE` | y | Enable THP |
| `CONFIG_TRANSPARENT_HUGEPAGE_ALWAYS` | y | THP always on (better for gaming) |
| `CONFIG_COMPACTION` | y | Memory compaction for hugepages |
| `CONFIG_KERNEL_ZSTD` | y | Zstd kernel/module compression |

### NVIDIA Support

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_DRM` | y | Direct Rendering Manager |
| `CONFIG_DRM_NOUVEAU` | m | Nouveau (required before nvidia-open loads) |
| `CONFIG_FB` | y | Framebuffer support |
| `CONFIG_DRM_FBDEV_EMULATION` | y | fbdev for nvidia-drm (6.11+ requirement) |
| `CONFIG_SYSFB_SIMPLEFB` | y | Simple framebuffer |

### Containers (Docker/Podman)

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_NAMESPACES` | y | Enable namespaces |
| `CONFIG_USER_NS` | y | User namespaces (rootless containers) |
| `CONFIG_CGROUPS` | y | Control groups |
| `CONFIG_OVERLAY_FS` | m | OverlayFS for container layers |
| `CONFIG_SECCOMP_FILTER` | y | Syscall filtering |

### Networking (Tailscale/WireGuard/Docker)

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_WIREGUARD` | m | WireGuard VPN |
| `CONFIG_TUN` | m | TUN/TAP devices |
| `CONFIG_VETH` | m | Virtual ethernet pairs |
| `CONFIG_BRIDGE` | m | Network bridging |
| `CONFIG_NF_TABLES` | y | nftables firewall |
| `CONFIG_IP_MULTIPLE_TABLES` | y | Policy routing (Tailscale) |

### Virtualization (KVM/QEMU)

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_KVM` | y | Kernel Virtual Machine |
| `CONFIG_KVM_AMD` | m | AMD KVM support (SVM) |
| `CONFIG_VFIO` | y | VFIO framework |
| `CONFIG_VFIO_PCI` | y | VFIO PCI driver (GPU passthrough) |
| `CONFIG_VIRTIO_*` | m | VirtIO drivers |

### Gaming (Wine/Proton)

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_FUTEX` | y | Fast userspace mutexes |
| `CONFIG_FUTEX2` | y | Futex2 extensions |
| `CONFIG_NTSYNC` | m | NT synchronization primitives (Wine) |

### Preemption

| Option | Value | Purpose |
|--------|-------|---------|
| `CONFIG_PREEMPT_LAZY` | y | Lazy preemption (new in 6.13) |
| `CONFIG_PREEMPT_DYNAMIC` | y | Runtime preemption switching |

**PREEMPT_LAZY:** Full preemption for RT/latency-sensitive tasks, lazy for normal tasks. Best balance of responsiveness and throughput.

---

## Customizing ghost.fragment

### Adding Options

Edit `config/ghost.fragment`:

```bash
# My custom option
CONFIG_MY_OPTION=y
```

### Disabling Options

To disable an option set in base config:

```bash
# Disable option
# CONFIG_UNWANTED_OPTION is not set
```

Or use `=n`:

```bash
CONFIG_UNWANTED_OPTION=n
```

### Module vs Built-in

| Suffix | Meaning |
|--------|---------|
| `=y` | Built into kernel |
| `=m` | Build as module |
| `=n` | Disabled |

**Guideline:** Use modules (`=m`) for hardware-specific drivers. Use built-in (`=y`) for core functionality needed at boot.

---

## Verifying Configuration

After building, check applied options:

```bash
# Check running kernel config
zcat /proc/config.gz | grep CONFIG_AMD_3D_VCACHE

# Check module parameters
modinfo nvidia | grep parm

# Check sched-ext
cat /sys/kernel/sched_ext/state
```

---

## Common Customizations

### Server Build (disable desktop features)

```bash
# In ghost.fragment
CONFIG_TRANSPARENT_HUGEPAGE_MADVISE=y
# CONFIG_TRANSPARENT_HUGEPAGE_ALWAYS is not set
CONFIG_HZ_100=y
# CONFIG_HZ_1000 is not set
```

### Minimal Build (reduce modules)

Use `_localmodcfg=yes` in PKGBUILD instead of editing ghost.fragment.

### Debug Build

```bash
CONFIG_DEBUG_INFO=y
CONFIG_DEBUG_INFO_DWARF5=y
CONFIG_KALLSYMS=y
CONFIG_KALLSYMS_ALL=y
```

---

## References

- [Kernel Kconfig Documentation](https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html)
- [AMD P-State Documentation](https://www.kernel.org/doc/html/latest/admin-guide/pm/amd-pstate.html)
- [sched-ext Documentation](https://github.com/sched-ext/scx)
