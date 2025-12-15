# Linux Ghost NVIDIA Guide

## Overview

Linux Ghost is optimized for NVIDIA GPUs using the open kernel modules (nvidia-open).

**Supported GPUs:** RTX 20+ (Turing, Ampere, Ada Lovelace, Blackwell)

---

## Installation Methods

### Method 1: DKMS (Recommended)

Use nvidia-open-dkms for automatic module rebuilding across kernel updates.

```bash
# Install DKMS modules
sudo pacman -S nvidia-open-dkms nvidia-utils lib32-nvidia-utils

# Copy linux-ghost optimized config
sudo cp nvidia/nvidia-ghost.conf /etc/modprobe.d/

# Regenerate initramfs
sudo mkinitcpio -P

# Reboot
sudo reboot
```

**Pros:**
- Works with multiple kernels
- Auto-rebuilds on kernel updates
- No version conflicts

### Method 2: Bundled Modules

Build NVIDIA modules directly into the kernel package.

```bash
_nvidia_bundle=yes makepkg -sf
sudo pacman -U linux-ghost-nvidia-open-*.pkg.tar.zst
```

**Pros:**
- Single package
- Includes linux-ghost exclusive patches

**Cons:**
- May conflict with other kernels using DKMS

---

## Linux Ghost Exclusive Patches

When using `_nvidia_bundle=yes`, these exclusive patches are applied:

### 1. Thunderbolt eGPU Hot-Plug (PR #985)

Prevents kernel crashes when Thunderbolt eGPUs are unplugged.

- Device tracking across nvidia, nvidia-modeset, nvidia-drm
- Safe hardware disconnect handling
- Tested on RTX 3060 via Thunderbolt 3

### 2. UVM NULL Pointer Fix (PR #978)

Fixes crash in GPU initialization error path.

- Affects NVIDIA 570/580/590 drivers
- NVIDIA tracking ID: 5610224

### 3. GSP Stutter Mitigation

Modprobe config to reduce GSP firmware stutter on Wayland/KDE.

---

## nvidia-ghost.conf

Optimized modprobe configuration installed to `/etc/modprobe.d/`:

```bash
# DRM/Wayland
options nvidia-drm modeset=1
options nvidia-drm fbdev=1

# Performance
options nvidia NVreg_UsePageAttributeTable=1
options nvidia NVreg_InitializeSystemMemoryAllocations=0
options nvidia NVreg_RegistryDwords="RMIntrLockingMode=1"
options nvidia NVreg_EnableResizableBar=1

# Power/Suspend
options nvidia NVreg_PreserveVideoMemoryAllocations=1

# GSP (nvidia-open)
options nvidia NVreg_EnableGpuFirmwareTaskTimeSlice=1

# Compatibility
options nvidia NVreg_OpenRmEnableUnsupportedGpus=1
```

---

## Kernel Configuration

ghost.fragment includes required NVIDIA configs:

```
CONFIG_DRM=y
CONFIG_DRM_FBDEV_EMULATION=y
CONFIG_SYSFB_SIMPLEFB=y
CONFIG_FB=y
CONFIG_FRAMEBUFFER_CONSOLE=y
```

---

## Wayland Setup

### KDE Plasma

```bash
# Ensure DRM modeset is enabled
cat /sys/module/nvidia_drm/parameters/modeset
# Should show: Y

# Check explicit sync
cat /sys/module/nvidia_drm/parameters/fbdev
# Should show: Y
```

### GNOME

Works out of the box with nvidia-open 555+.

---

## Troubleshooting

### Module Version Mismatch

```bash
# Check versions
pacman -Q nvidia-utils nvidia-open-dkms

# Must be same version (e.g., 580.105.08)
# If mismatched:
sudo pacman -S nvidia-utils=580.105.08
```

### GSP Stutter (KDE/Wayland)

```bash
# Verify config is loaded
cat /sys/module/nvidia/parameters/NVreg_EnableGpuFirmwareTaskTimeSlice

# If not, ensure nvidia-ghost.conf is installed
sudo cp nvidia/nvidia-ghost.conf /etc/modprobe.d/
sudo mkinitcpio -P
sudo reboot
```

### Black Screen on Boot

```bash
# Boot with nomodeset, then:
sudo nano /etc/modprobe.d/nvidia-ghost.conf

# Ensure these are set:
options nvidia-drm modeset=1
options nvidia-drm fbdev=1

sudo mkinitcpio -P
sudo reboot
```

### Thunderbolt eGPU Not Detected

```bash
# Check Thunderbolt status
cat /sys/bus/thunderbolt/devices/*/device_name

# Check NVIDIA detection
nvidia-smi

# Check dmesg
dmesg | grep -i thunderbolt
dmesg | grep -i nvidia
```

---

## RTX 5090 (Blackwell) Notes

- Requires nvidia-open 570+
- GSP firmware required (cannot disable)
- Full support in linux-ghost with CONFIG_DRM_FBDEV_EMULATION

---

## References

- [NVIDIA open-gpu-kernel-modules](https://github.com/NVIDIA/open-gpu-kernel-modules)
- [Arch Wiki NVIDIA](https://wiki.archlinux.org/title/NVIDIA)
- [nvidia-ghost.conf](../nvidia/nvidia-ghost.conf)
