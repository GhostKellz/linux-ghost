# NVIDIA Support

Linux Ghost is optimized for NVIDIA GPUs (RTX 20+ / Blackwell) with **exclusive patches** not found in CachyOS or linux-tkg.

## Default: nvidia-open-dkms (Recommended)

The kernel includes all necessary configs for NVIDIA but does **not** bundle modules by default.
Use nvidia-open-dkms which rebuilds automatically for each kernel:

```bash
# Install NVIDIA open modules via DKMS
sudo pacman -S nvidia-open-dkms nvidia-utils lib32-nvidia-utils

# Copy our optimized config
sudo cp nvidia/nvidia-ghost.conf /etc/modprobe.d/
```

Benefits:
- Works with multiple kernels (linux-ghost + linux-cachyos + linux-lts)
- Automatic rebuilds on kernel updates
- No version conflicts

## Optional: Bundled Modules (with Exclusive Patches)

Bundle nvidia-open directly with the kernel including linux-ghost exclusive patches:

```bash
_nvidia_bundle=yes makepkg -sf
```

This creates `linux-ghost-nvidia-open` package with:
- All standard compatibility patches
- **Exclusive** Thunderbolt eGPU hot-plug support
- **Exclusive** UVM NULL pointer deref fix
- **Exclusive** GSP stutter mitigation config

## Supported GPUs

NVIDIA open modules require **Turing or newer** (GSP firmware):
- RTX 20xx series (Turing)
- RTX 30xx series (Ampere)
- RTX 40xx series (Ada Lovelace)
- RTX 50xx series (Blackwell)

Older GPUs (GTX 10xx and below) need proprietary nvidia-dkms.

---

## linux-ghost Exclusive Patches

### 1. Thunderbolt eGPU Hot-Plug Support (PR #985)

**File:** `0001-thunderbolt-egpu-hotplug.patch`

Prevents kernel crashes when Thunderbolt external GPUs are unexpectedly unplugged:
- Device tracking across nvidia, nvidia-modeset, nvidia-drm modules
- Safety checks before hardware operations
- ISR protection when GPU is inaccessible
- Proper DRM cleanup during surprise removal

**Status:** Community patch, tested on RTX 3060 via Thunderbolt 3

### 2. UVM NULL Pointer Deref Fix (PR #978)

**File:** `0002-uvm-null-ptr-deref-fix.patch`

Fixes crash in `uvm_parent_gpu_t` error path:
- Bug persists in NVIDIA 570/580/590
- Affects systems with GPU initialization failures
- Fixes null pointer dereference of uninitialized `pci_dev`

**NVIDIA ID:** 5610224

### 3. GSP Stutter Mitigation Config

**File:** `nvidia-ghost.conf`

Modprobe configuration to reduce GSP firmware stutter issues:
- `NVreg_EnableGpuFirmwareTaskTimeSlice=1` - Reduces idle stutter
- DRM modesetting and fbdev enabled
- Video memory preservation for suspend/resume
- G-Sync on non-validated displays

---

## Standard Compatibility Patches

| Patch | Purpose |
|-------|---------|
| `Enable-atomic-kernel-modesetting-by-default.diff` | Better Wayland/KMS support |
| `Add-IBT-support.diff` | Intel Branch Tracking compatibility |
| `6.11-fbdev.diff` | fbdev fixes for 6.11+ kernels |
| `gcc-15.diff` | GCC 15 compatibility |

---

## Kernel Configs

The ghost.fragment enables required NVIDIA support:

```
CONFIG_DRM=y
CONFIG_DRM_FBDEV_EMULATION=y
CONFIG_SYSFB_SIMPLEFB=y
CONFIG_FB=y
CONFIG_FRAMEBUFFER_CONSOLE=y
```

---

## Comparison: linux-ghost vs Others

| Feature | CachyOS | TKG | linux-ghost |
|---------|---------|-----|-------------|
| Atomic modesetting | Yes | No | Yes |
| IBT support | Yes | No | Yes |
| Thunderbolt eGPU hotplug | **No** | **No** | **Yes** |
| UVM null deref fix | **No** | **No** | **Yes** |
| GSP stutter config | **No** | **No** | **Yes** |
| fbdev 6.11+ fix | Yes | No | Yes |

---

## Troubleshooting

### GSP Stutter (KDE Plasma / Wayland)
```bash
# Install our config
sudo cp nvidia/nvidia-ghost.conf /etc/modprobe.d/
sudo mkinitcpio -P
reboot
```

### Thunderbolt eGPU not detected
```bash
# Check if GPU is recognized
lspci | grep -i nvidia

# Check dmesg for hotplug events
dmesg | grep -i thunderbolt
```

### Module version mismatch
```bash
# nvidia-utils version must match nvidia-open-dkms
pacman -Q nvidia-utils nvidia-open-dkms
```

### Blackwell (RTX 5090) not detected
Requires nvidia-open 570+ with GSP firmware support.

---

## References

- [NVIDIA open-gpu-kernel-modules](https://github.com/NVIDIA/open-gpu-kernel-modules)
- [Thunderbolt eGPU PR #985](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/985)
- [UVM NULL deref PR #978](https://github.com/NVIDIA/open-gpu-kernel-modules/pull/978)
- [GSP Stutter Issue #777](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/777)
