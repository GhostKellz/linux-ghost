# AMD Zen5 / X3D Setup

linux-ghost targets AMD Zen4/Zen5 and X3D processors (9950X3D, 9900X3D,
7950X3D). This page covers CPU microcode, the `znver5` build target, 3D
V-Cache control, and post-boot validation.

## CPU Microcode

linux-ghost does **not** embed AMD microcode in the kernel image. Microcode is
loaded early from the initramfs, the same approach used by Arch's stock kernels
and linux-tkg. Install the firmware package and let the bootloader prepend it:

```bash
sudo pacman -S amd-ucode
```

`amd-ucode` is listed as an `optdepends` of `linux-ghost`. The kernel is built
with `CONFIG_MICROCODE=y`; on Linux 7.1 the old per-vendor `MICROCODE_AMD`
symbol is folded into `MICROCODE` (gated by `CPU_SUP_AMD`), so this single
option covers AMD early loading. Late loading (`MICROCODE_LATE_LOADING`) is left
disabled on purpose — early initramfs loading is the safe path.

### Bootloader ordering

The microcode image must be loaded **before** the kernel initramfs.

**systemd-boot** (`/boot/loader/entries/linux-ghost.conf`):

```
title    Linux Ghost
linux    /vmlinuz-linux-ghost
initrd   /amd-ucode.img
initrd   /initramfs-linux-ghost.img
options  root=... rw
```

The `amd-ucode.img` line must come first.

**GRUB / mkinitcpio:** microcode is picked up automatically once `amd-ucode` is
installed and the config is regenerated:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**Unified Kernel Image (UKI):** add `/boot/amd-ucode.img` ahead of the initramfs
in the `.initrd` section (e.g. the `mkinitcpio` UKI `microcode=` setting or the
`ukify` `--microcode` argument).

## Zen5 (`znver5`)

Mainline 7.1 and the CachyOS base only ship CPU targets up to `MZEN4`.
linux-ghost carries `ghostzen5.patch`, which adds `CONFIG_MZEN5`
(`-march=znver5`). The patch is **still required** on the 7.1 base — it has not
been upstreamed.

Build with the Zen5 target:

```bash
_processor_opt=zen5 makepkg -sf
```

`CONFIG_MZEN5` depends on Clang `>= 19.1` (`CLANG_VERSION >= 190100`). With an
older compiler `olddefconfig` silently drops `MZEN5` and falls back to a generic
target, so build with a recent Clang/LLVM.

## 3D V-Cache (X3D)

For dual-CCD X3D parts, set the BIOS CPPC option to **Driver** (CBS > CPU Common
> CPPC) so the `amd_x3d_vcache` driver can expose the cache/frequency preference:

```bash
# Prefer the V-Cache CCD (latency-sensitive / gaming)
echo cache     | sudo tee /sys/bus/platform/drivers/amd_x3d_vcache/AMDI0101:00/amd_x3d_mode
# Prefer the high-frequency CCD (all-core throughput)
echo frequency | sudo tee /sys/bus/platform/drivers/amd_x3d_vcache/AMDI0101:00/amd_x3d_mode
```

## Post-boot validation

After installing and rebooting into linux-ghost:

```bash
# Running the ghost kernel
uname -r

# Early microcode loaded
dmesg | grep -i microcode

# Zen5 target compiled in
zcat /proc/config.gz | grep -E 'CONFIG_MZEN5|CONFIG_MICROCODE='

# AMD P-State with preferred-core ranking
cat /sys/devices/system/cpu/amd_pstate/status
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver

# X3D V-Cache mode (when firmware/BIOS exposes it)
cat /sys/bus/platform/drivers/amd_x3d_vcache/AMDI0101:00/amd_x3d_mode
```
