# Contributing to linux-ghost

## Getting Started

linux-ghost is a custom Arch Linux kernel package targeting AMD Zen/X3D and NVIDIA RTX workloads. Contributions that improve gaming performance, hardware support, scheduler behavior, or build reliability are welcome.

### Prerequisites

- Arch Linux (or Arch-based) system
- `base-devel` package group
- LLVM/Clang toolchain (`clang`, `llvm`, `lld`)
- Enough disk space for a kernel build, usually 30GB or more
- Familiarity with Arch `makepkg`

## Building

```bash
git clone https://github.com/ghostkellz/linux-ghost.git
cd linux-ghost
makepkg -sf
```

Override build options via environment. See [docs/getting-started/build-options.md](docs/getting-started/build-options.md) for the full list.

```bash
_cpusched=bore _lto_mode=thin makepkg -sf
```

## Project Structure

```
PKGBUILD                  # Arch package build script
config/
  config                  # Base kernel .config
  ghost.fragment          # Config overlay (applied on top of base config)
patches/
  ghost-sched-v2-*.patch  # GHOST scheduler (maintained here)
  ghostzen5.patch         # Zen5 MZEN5 Kconfig support
  0003-glitched-base.patch # Zenify (modified for CachyOS pre-patched source)
  0012-misc-additions.patch # TKG misc fixes
nvidia/                   # Optional NVIDIA module patches
docs/                     # Structured user and maintainer docs
.github/workflows/        # CI: build test, nightly check, release
```

## Submitting Changes

1. Fork the repo and create a branch from `main`
2. Make your changes
3. Test build with `makepkg -sf` (at minimum `makepkg -o` to verify prepare/patch step)
4. Open a pull request against `main`

### Commit Messages

Follow the existing convention:

```
feat: description        # new feature or capability
fix: description         # bug fix
chore: description       # maintenance, version bumps
```

## Patch Contributions

### Modifying existing patches

If a patch needs updating for a new kernel version, edit it in `patches/` and test that it applies cleanly on the CachyOS pre-patched source.

### Adding new patches

1. Place the `.patch` file in `patches/`
2. Add an `_apply_local_patch "<file>.patch"` call in `prepare()`, in the correct
   order relative to the other patches
3. Update `patches/README.md`
4. Test a full build

Linux-ghost's own patches are applied locally from `${startdir}/patches/` during
`prepare()` (not fetched from a URL), so release builds use the checked-out tag
content.

### Upstream patches

Patches from CachyOS or linux-tkg (OpenRGB, ACS override, dkms-clang, the BORE
patch) are currently fetched at build time and added to the `source=()` array
with a `sha256sums+=('SKIP')` entry.

## Config Changes

- `config/config` — full base kernel config. Regenerate with `make olddefconfig` against new kernel source when bumping versions.
- `config/ghost.fragment` — overlay config for linux-ghost specific options. Prefer adding options here over modifying the base config directly.

## Kernel Version Bumps

When a new CachyOS release is available:

1. Update `_major`, `_minor`, `_cachyos_tagrel` in PKGBUILD
2. Verify the CachyOS tag exists: `https://github.com/CachyOS/linux/releases`
3. Update `config/config` by running `make olddefconfig` against new source
4. Test-build patches apply cleanly
5. Update [docs/README.md](docs/README.md), [CHANGELOG.md](CHANGELOG.md), and [SECURITY.md](SECURITY.md)
6. Tag as `v{major}.{minor}-{pkgrel}`

## Reporting Issues

Open an issue on GitHub with:
- Kernel version and build options used
- Relevant `dmesg` or build log output
- Hardware details if it's a hardware-specific issue

## Documentation

Keep documentation changes close to the behavior they describe:

- Build flags: [docs/getting-started/build-options.md](docs/getting-started/build-options.md)
- Kernel config: [docs/kernel/configuration.md](docs/kernel/configuration.md)
- Schedulers: [docs/kernel/schedulers.md](docs/kernel/schedulers.md)
- NVIDIA support: [docs/hardware/nvidia.md](docs/hardware/nvidia.md)
- Release workflow: [docs/development/README.md](docs/development/README.md)
