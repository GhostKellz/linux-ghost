# Maintainer: GhostKellz <ghost@ghostkellz.sh>
# linux-ghost: Custom kernel for AMD Zen5 X3D + NVIDIA RTX 5090 + AI + Gaming workloads
# Based on: CachyOS linux-cachyos & linux-tkg

### ============================================================
### BUILD OPTIONS
### ============================================================

### Kernel version selection
# 'stable' - Use stable release (6.18.x)
# 'rc' - Use release candidate (6.19-rcX)
: "${_kernel_type:=stable}"

### Selecting the CPU scheduler
# 'ghost' - GHOST Scheduler on EEVDF (Recommended for desktop/gaming, Zen4/Zen5 optimized)
# 'eevdf' - Vanilla EEVDF (upstream default)
: "${_cpusched:=ghost}"

### CPU compiler optimizations
# 'native' - Auto-detect CPU (recommended when building on target machine)
# 'zen5' - AMD Zen5 (9950X3D, 9900X3D, etc.)
# 'zen4' - AMD Zen4 (7950X3D, 7900X3D, etc.)
# 'generic' - Generic x86-64
: "${_processor_opt:=native}"

### Compiler selection
# 'llvm' - Clang/LLVM toolchain (recommended, better LTO)
# 'gcc' - GCC toolchain
: "${_compiler:=llvm}"

### LTO (Link Time Optimization)
# For LLVM: 'full', 'thin', or 'none'
#   full - slower build, potentially best performance
#   thin - faster build, good performance
# For GCC: 'full', 'thin', or 'none'
#   full - GCC LTO (slow, high memory usage)
#   thin - Not supported for GCC, falls back to full
: "${_lto_mode:=full}"

### Enable Zenify gaming optimizations (from Zen/Liquorix)
# Tunes EEVDF scheduler for lower latency gaming
: "${_zenify:=yes}"

### Enable Clear Linux patches (Intel optimizations)
: "${_clear_patches:=no}"

### Enable OpenRGB i2c controller support
: "${_openrgb:=yes}"

### Tweak kernel options prior to a build via nconfig
: "${_makenconfig:=no}"

### Tweak kernel options prior to a build via xconfig
: "${_makexconfig:=no}"

### Compile ONLY used modules to reduce build time
# Requires modprobed-db: https://wiki.archlinux.org/index.php/Modprobed-db
: "${_localmodcfg:=no}"

### Path to the list of used modules
: "${_localmodcfg_path:="$HOME/.config/modprobed.db"}"

### Use the current kernel's .config file
: "${_use_current:=no}"

### Enable KBUILD_CFLAGS -O3
: "${_cc_harder:=yes}"

### Set performance governor as default
: "${_per_gov:=yes}"

### Enable TCP_CONG_BBR3
: "${_tcp_bbr3:=yes}"

### Running tick rate (1000Hz recommended for gaming)
: "${_HZ_ticks:=1000}"

### Choose between perodic, idle or full
# Full tickless recommended for gaming/low-latency
: "${_tickrate:=full}"

### Choose between full(low-latency), lazy, voluntary or none
: "${_preempt:=full}"

### Transparent Hugepages
# 'always' - better for gaming/large memory workloads
# 'madvise' - more conservative
: "${_hugepage:=always}"

### Bundle NVIDIA open kernel modules with kernel
# Set to 'yes' to bundle nvidia-open modules (like CachyOS does)
# Set to 'no' to use nvidia-open-dkms instead (default, recommended)
# DKMS rebuilds automatically for each kernel - better for multi-kernel setups
# WARNING: Bundled modules can conflict with other kernels using DKMS!
: "${_nvidia_bundle:=no}"

### Build a debug package with non-stripped vmlinux
: "${_build_debug:=no}"

### Enable sched-ext support (for scx_lavd, scx_bpfland, etc.)
: "${_sched_ext:=yes}"

### Zen5 3D V-Cache optimizations
# Enable extra optimizations for X3D processors (9950X3D, 7950X3D)
: "${_zen5_x3d:=yes}"

### ACS Override (for VFIO/GPU passthrough)
# Note: Already included in CachyOS base patch
: "${_acs_override:=no}"

### ============================================================
### INTERNAL - Version Configuration
### ============================================================

# Stable kernel
_major=6.18
_minor=4

# RC kernel (when _kernel_type=rc)
_rc_major=6.19
_rc_ver=rc1

if [[ "$_kernel_type" == "rc" ]]; then
    pkgver="${_rc_major}.${_rc_ver/rc/}"
    _stable="${_rc_major}-${_rc_ver}"
    _srcname="linux-${_rc_major}-${_rc_ver}"
    _kernel_src="https://git.kernel.org/torvalds/t/linux-${_rc_major}-${_rc_ver}.tar.gz"
else
    pkgver="${_major}.${_minor}"
    _stable="${_major}.${_minor}"
    _srcname="linux-${_stable}"
    _kernel_src="https://cdn.kernel.org/pub/linux/kernel/v${_major%%.*}.x/${_srcname}.tar.xz"
fi

# Package naming based on compiler
if [[ "$_compiler" == "llvm" && "$_lto_mode" != "none" ]]; then
    pkgbase=linux-ghost
    _compiler_suffix="-lto"
elif [[ "$_compiler" == "gcc" ]]; then
    pkgbase=linux-ghost-gcc
    _compiler_suffix="-gcc"
else
    pkgbase=linux-ghost
    _compiler_suffix=""
fi

pkgdesc="Linux Ghost - GHOST Scheduler + Zenify + Zen4/Zen5 X3D + RTX 5090 + sched-ext (${_compiler^^}${_compiler_suffix:+ }${_lto_mode^^} LTO)"
pkgrel=1
_kernver="$pkgver-$pkgrel"
_kernuname="${pkgver}-${pkgbase#linux-}"
arch=('x86_64')
url="https://github.com/ghostkellz/linux-ghost"
license=('GPL-2.0-only')
options=('!strip' '!debug' '!lto')
install=linux-ghost.install

makedepends=(
    bc
    cpio
    gettext
    libelf
    pahole
    perl
    python
    tar
    xz
    zstd
    # Rust (for kernel rust support)
    rust
    rust-bindgen
    rust-src
)

# Add compiler-specific makedepends
if [[ "$_compiler" == "llvm" ]]; then
    makedepends+=(clang llvm lld)
else
    makedepends+=(gcc)
fi

# Patch sources
_patchsource="https://raw.githubusercontent.com/cachyos/kernel-patches/master/${_major}"
_tkgpatch="https://raw.githubusercontent.com/Frogging-Family/linux-tkg/master/linux-tkg-patches/${_major}"

# NVIDIA versions - RTX 5090 (Blackwell) requires 570+
_nv_ver=580.105.08
_nv_open_pkg="NVIDIA-kernel-module-source-${_nv_ver}"

source=(
    "${_kernel_src}"
    "kernel.config::https://raw.githubusercontent.com/GhostKellz/linux-ghost/refs/heads/main/config/config"
    "kernel.fragment::https://raw.githubusercontent.com/GhostKellz/linux-ghost/refs/heads/main/config/ghost.fragment"
    # CachyOS base patches (amd-pstate, bbr3, sched-ext, block opts, etc.)
    "${_patchsource}/all/0001-cachyos-base-all.patch"
)
sha256sums=('SKIP' 'SKIP' 'SKIP' 'SKIP')

# GHOST scheduler patch (our enhanced scheduler based on BORE)
if [[ "$_cpusched" == "ghost" ]]; then
    source+=("0001-ghost-sched.patch::https://raw.githubusercontent.com/GhostKellz/linux-ghost/main/patches/0001-ghost-sched.patch")
    sha256sums+=('SKIP')
fi

# Zenify gaming patches (from linux-tkg)
if [[ "$_zenify" == "yes" ]]; then
    source+=("${_tkgpatch}/0003-glitched-base.patch")
    sha256sums+=('SKIP')
    # EEVDF-specific zenify tuning
    if [[ -n "$(curl -sI "${_tkgpatch}/0003-glitched-eevdf-additions.patch" 2>/dev/null | grep '200 OK')" ]]; then
        source+=("${_tkgpatch}/0003-glitched-eevdf-additions.patch")
        sha256sums+=('SKIP')
    fi
fi

# Clear Linux patches (Intel optimizations)
if [[ "$_clear_patches" == "yes" ]]; then
    source+=("${_tkgpatch}/0002-clear-patches.patch")
    sha256sums+=('SKIP')
fi

# Zen5 MZEN5 Kconfig patch (adds -march=znver5 support)
source+=("ghostzen5.patch::https://raw.githubusercontent.com/GhostKellz/linux-ghost/main/patches/ghostzen5.patch")
sha256sums+=('SKIP')

# OpenRGB i2c support
if [[ "$_openrgb" == "yes" ]]; then
    source+=("${_tkgpatch}/0014-OpenRGB.patch")
    sha256sums+=('SKIP')
fi

# ACS Override for VFIO
if [[ "$_acs_override" == "yes" ]]; then
    source+=("${_tkgpatch}/0006-add-acs-overrides_iommu.patch")
    sha256sums+=('SKIP')
fi

# NVIDIA open modules (RTX 5090 Blackwell support)
if [[ "$_nvidia_bundle" == "yes" ]]; then
    source+=(
        "https://download.nvidia.com/XFree86/NVIDIA-kernel-module-source/${_nv_open_pkg}.tar.xz"
        # linux-ghost exclusive NVIDIA patches
        "nvidia-ghost.conf::nvidia/nvidia-ghost.conf"
        "nv-atomic-modesetting.patch::nvidia/Enable-atomic-kernel-modesetting-by-default.diff"
        "nv-ibt-support.patch::nvidia/Add-IBT-support.diff"
        "nv-fbdev-fix.patch::nvidia/6.11-fbdev.diff"
        "nv-gcc15.patch::nvidia/gcc-15.diff"
        # Exclusive: Thunderbolt eGPU hot-plug support (PR #985)
        "nv-thunderbolt-egpu.patch::nvidia/0001-thunderbolt-egpu-hotplug.patch"
        # Exclusive: NULL pointer deref fix (PR #978)
        "nv-uvm-null-deref.patch::nvidia/0002-uvm-null-ptr-deref-fix.patch"
    )
    sha256sums+=('SKIP' 'SKIP' 'SKIP' 'SKIP' 'SKIP' 'SKIP' 'SKIP' 'SKIP')
fi

# Build flags based on compiler choice
if [[ "$_compiler" == "llvm" ]]; then
    BUILD_FLAGS=(
        CC=clang
        LD=ld.lld
        LLVM=1
        LLVM_IAS=1
    )
    # Add DKMS clang patch for LLVM builds
    source+=("${_patchsource}/misc/dkms-clang.patch")
    sha256sums+=('SKIP')
else
    BUILD_FLAGS=()
fi

export KBUILD_BUILD_HOST=ghostkellz
export KBUILD_BUILD_USER="$pkgbase"
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

_die() { error "$@" ; exit 1; }

prepare() {
    cd "$_srcname"

    echo "Setting version..."
    echo "-$pkgrel" > localversion.10-pkgrel
    echo "${pkgbase#linux}" > localversion.20-pkgname

    local src
    for src in "${source[@]}"; do
        src="${src%%::*}"
        # Skip nvidia patches (applied separately to nvidia source)
        [[ "$src" == nv-*.patch ]] && continue
        [[ "$src" == nvidia-ghost.conf ]] && continue
        # Skip config fragments
        [[ "$src" == *.fragment ]] && continue
        src="${src##*/}"
        src="${src%.zst}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        patch -Np1 < "../$src" || echo "Warning: patch $src failed, continuing..."
    done

    echo "Setting config..."
    cp "${startdir}/kernel.config" .config

    # Apply ghost.fragment config overrides
    if [[ -f "${startdir}/kernel.fragment" ]]; then
        echo "Applying kernel.fragment config..."
        scripts/kconfig/merge_config.sh -m .config "${startdir}/kernel.fragment"
    fi

    ### ============================================================
    ### CPU OPTIMIZATIONS (Zen5 X3D / Zen4 X3D)
    ### ============================================================

    if [ -n "$_processor_opt" ]; then
        MARCH="${_processor_opt^^}"
        case "$MARCH" in
            ZEN5)
                echo "Enabling Zen5 (9950X3D) optimizations (-march=znver5)..."
                scripts/config -d GENERIC_CPU -d MZEN4 -e MZEN5 -d X86_NATIVE_CPU
                ;;
            ZEN4)
                echo "Enabling Zen4 (7950X3D) optimizations (-march=znver4)..."
                scripts/config -d GENERIC_CPU -e MZEN4 -d MZEN5 -d X86_NATIVE_CPU
                ;;
            NATIVE)
                echo "Enabling native CPU optimizations (auto-detect)..."
                scripts/config -d GENERIC_CPU -d MZEN4 -d MZEN5 -e X86_NATIVE_CPU
                ;;
            GENERIC*)
                scripts/config -e GENERIC_CPU -d MZEN4 -d MZEN5 -d X86_NATIVE_CPU
                ;;
        esac
    else
        scripts/config -d GENERIC_CPU -d MZEN4 -d MZEN5 -e X86_NATIVE_CPU
    fi

    ### Zen5 X3D specific optimizations
    if [[ "$_zen5_x3d" == "yes" ]]; then
        echo "Enabling Zen5 3D V-Cache optimizations..."
        # NUMA for chiplet design
        scripts/config -e NUMA -e NUMA_BALANCING -e NUMA_BALANCING_DEFAULT_ENABLED
        # AMD platform
        scripts/config -e X86_AMD_PLATFORM_DEVICE -e AMD_NB
        scripts/config -e AMD_IOMMU -m AMD_IOMMU_V2
        # AMD P-State (preferred for Zen4/5)
        scripts/config -e X86_AMD_PSTATE
        # High core count
        scripts/config --set-val NR_CPUS 32
    fi

    ### PCIe optimizations for high-end GPU (RTX 5090 / high-bandwidth)
    echo "Enabling PCIe performance optimizations..."
    scripts/config -d PCIEASPM_DEFAULT \
        -d PCIEASPM_POWERSAVE \
        -d PCIEASPM_POWER_SUPERSAVE \
        -e PCIEASPM_PERFORMANCE \
        -e PCI_REALLOC_ENABLE_AUTO

    ### ============================================================
    ### SCHEDULER CONFIGURATION
    ### ============================================================

    echo "Enabling Ghost kernel marker..."
    scripts/config -e CACHY

    case "$_cpusched" in
        ghost)
            echo "Enabling GHOST scheduler (Zen4/Zen5 X3D optimized)..."
            scripts/config -e SCHED_GHOST
            ;;
        eevdf)
            echo "Using vanilla EEVDF scheduler..."
            ;;
        *)
            _die "Invalid scheduler: $_cpusched"
            ;;
    esac

    ### Enable sched-ext (for scx_lavd, scx_bpfland)
    if [[ "$_sched_ext" == "yes" ]]; then
        echo "Enabling sched-ext support..."
        scripts/config -e SCHED_CLASS_EXT
        scripts/config -e BPF_JIT -e BPF_JIT_ALWAYS_ON
    fi

    ### Zenify tuning (lower latency for gaming)
    if [[ "$_zenify" == "yes" ]]; then
        echo "Enabling Zenify gaming optimizations..."
        scripts/config -e ZENIFY 2>/dev/null || true
    fi

    ### ============================================================
    ### PERFORMANCE SETTINGS
    ### ============================================================

    ### LTO Configuration
    if [[ "$_compiler" == "llvm" ]]; then
        case "$_lto_mode" in
            thin)
                echo "Enabling Clang ThinLTO..."
                scripts/config -e LTO_CLANG_THIN -d LTO_CLANG_FULL -d LTO_NONE
                ;;
            full)
                echo "Enabling Clang Full LTO..."
                scripts/config -e LTO_CLANG_FULL -d LTO_CLANG_THIN -d LTO_NONE
                ;;
            none)
                echo "Disabling LTO..."
                scripts/config -e LTO_NONE -d LTO_CLANG_THIN -d LTO_CLANG_FULL
                ;;
        esac
    else
        # GCC LTO
        case "$_lto_mode" in
            full|thin)
                echo "Enabling GCC LTO..."
                scripts/config -e LTO_GCC -d LTO_NONE 2>/dev/null || \
                echo "Note: GCC LTO may not be available in this kernel config"
                ;;
            none)
                echo "Disabling LTO..."
                scripts/config -e LTO_NONE -d LTO_GCC 2>/dev/null || true
                ;;
        esac
    fi
    echo "Compiler: $_compiler, LTO: $_lto_mode"

    ### Tick rate
    case "$_HZ_ticks" in
        100|250|500|600|750|1000)
            scripts/config -d HZ_300 -e "HZ_${_HZ_ticks}" --set-val HZ "${_HZ_ticks}";;
        300)
            scripts/config -e HZ_300 --set-val HZ 300;;
    esac
    echo "Setting tick rate to ${_HZ_ticks}Hz..."

    ### Performance governor
    if [[ "$_per_gov" == "yes" ]]; then
        echo "Setting performance governor as default..."
        scripts/config -d CPU_FREQ_DEFAULT_GOV_SCHEDUTIL -e CPU_FREQ_DEFAULT_GOV_PERFORMANCE
    fi

    ### Tick type
    case "$_tickrate" in
        perodic) scripts/config -d NO_HZ_IDLE -d NO_HZ_FULL -d NO_HZ -d NO_HZ_COMMON -e HZ_PERIODIC;;
        idle) scripts/config -d HZ_PERIODIC -d NO_HZ_FULL -e NO_HZ_IDLE -e NO_HZ -e NO_HZ_COMMON;;
        full) scripts/config -d HZ_PERIODIC -d NO_HZ_IDLE -d CONTEXT_TRACKING_FORCE -e NO_HZ_FULL_NODEF -e NO_HZ_FULL -e NO_HZ -e NO_HZ_COMMON -e CONTEXT_TRACKING;;
    esac
    echo "Selecting '$_tickrate' tick type..."

    ### Preemption
    case "$_preempt" in
        full) scripts/config -e PREEMPT_DYNAMIC -e PREEMPT -d PREEMPT_VOLUNTARY -d PREEMPT_LAZY -d PREEMPT_NONE;;
        lazy) scripts/config -e PREEMPT_DYNAMIC -d PREEMPT -d PREEMPT_VOLUNTARY -e PREEMPT_LAZY -d PREEMPT_NONE;;
        voluntary) scripts/config -d PREEMPT_DYNAMIC -d PREEMPT -e PREEMPT_VOLUNTARY -d PREEMPT_LAZY -d PREEMPT_NONE;;
        none) scripts/config -d PREEMPT_DYNAMIC -d PREEMPT -d PREEMPT_VOLUNTARY -d PREEMPT_LAZY -e PREEMPT_NONE;;
    esac
    echo "Selecting '$_preempt' preempt type..."

    ### O3 optimization
    if [[ "$_cc_harder" == "yes" ]]; then
        echo "Enabling KBUILD_CFLAGS -O3..."
        scripts/config -d CC_OPTIMIZE_FOR_PERFORMANCE -e CC_OPTIMIZE_FOR_PERFORMANCE_O3
    fi

    ### BBR3
    if [[ "$_tcp_bbr3" == "yes" ]]; then
        echo "Enabling TCP BBR3..."
        scripts/config -m TCP_CONG_CUBIC -d DEFAULT_CUBIC \
            -e TCP_CONG_BBR -e DEFAULT_BBR --set-str DEFAULT_TCP_CONG bbr \
            -m NET_SCH_FQ_CODEL -e NET_SCH_FQ -d CONFIG_DEFAULT_FQ_CODEL -e CONFIG_DEFAULT_FQ
    fi

    ### THP
    case "$_hugepage" in
        always) scripts/config -d TRANSPARENT_HUGEPAGE_MADVISE -e TRANSPARENT_HUGEPAGE_ALWAYS;;
        madvise) scripts/config -d TRANSPARENT_HUGEPAGE_ALWAYS -e TRANSPARENT_HUGEPAGE_MADVISE;;
    esac
    echo "Selecting '$_hugepage' TRANSPARENT_HUGEPAGE..."

    ### ============================================================
    ### NVIDIA RTX 5090 (Blackwell) SUPPORT
    ### ============================================================

    if [[ "$_nvidia_bundle" == "yes" ]]; then
        echo "Configuring for NVIDIA RTX 5090 (Blackwell)..."
        # DRM/fbdev required for 6.11+ with nvidia
        scripts/config -e DRM -m DRM_NOUVEAU
        scripts/config -e FB -e FRAMEBUFFER_CONSOLE
        scripts/config -e DRM_FBDEV_EMULATION -e SYSFB_SIMPLEFB
    fi

    ### ============================================================
    ### GHOSTKELLZ.MYFRAG FEATURES
    ### ============================================================

    echo "Enabling ghostkellz features..."

    # Containers (Docker/Podman)
    scripts/config -e NAMESPACES -e UTS_NS -e PID_NS -e NET_NS -e USER_NS
    scripts/config -e CGROUPS -e CGROUP_SCHED -e CGROUP_PIDS -e CGROUP_FREEZER
    scripts/config -e CGROUP_DEVICE -e CGROUP_CPUACCT -e CGROUP_PERF -e CGROUP_BPF
    scripts/config -e MEMCG -e BLK_CGROUP
    scripts/config -m OVERLAY_FS
    scripts/config -e SECCOMP -e SECCOMP_FILTER
    scripts/config -e SECURITY_APPARMOR

    # Networking (nftables, wireguard, tailscale)
    scripts/config -m TUN -m VETH -m BRIDGE -m BRIDGE_NETFILTER
    scripts/config -m WIREGUARD -m VXLAN -m GENEVE
    scripts/config -e NETFILTER -e NETFILTER_ADVANCED -e NF_CONNTRACK
    scripts/config -e NF_TABLES -e NF_TABLES_INET
    scripts/config -m NFT_NAT -m NFT_MASQ -m NFT_COMPAT
    scripts/config -e IP_ADVANCED_ROUTER -e IP_MULTIPLE_TABLES

    # KVM/VFIO (virtualization & GPU passthrough)
    scripts/config -e KVM -m KVM_AMD -m KVM_INTEL
    scripts/config -e VIRTIO -m VIRTIO_PCI -e VIRTIO_NET -m VIRTIO_BLK
    scripts/config -m VIRTIO_GPU -m VIRTIO_FS
    scripts/config -e VFIO -e VFIO_PCI -m VFIO_MDEV
    scripts/config -e VHOST -e VHOST_NET -e VHOST_VSOCK

    # Storage (NVMe)
    scripts/config -e BLK_DEV_NVME -e NVME_CORE -e NVME_MULTIPATH

    # USB
    scripts/config -e USB_XHCI_HCD -e USB_EHCI_HCD

    # Elgato webcam support
    scripts/config -e MEDIA_SUPPORT -e MEDIA_USB_SUPPORT -e MEDIA_CAMERA_SUPPORT
    scripts/config -m USB_VIDEO_CLASS -m SND_USB_AUDIO

    # CIFS/SMB
    scripts/config -m CIFS -e CIFS_XATTR -e CIFS_DFS_UPCALL

    # Crypto (Wine/Proton TLS)
    scripts/config -e TLS -m CRYPTO_AES_NI_INTEL

    # Futex (Proton esync/fsync - built-in on 6.18)
    scripts/config -e FUTEX

    ### ============================================================
    ### FINALIZE
    ### ============================================================

    echo "Enabling USER_NS_UNPRIVILEGED..."
    scripts/config -e USER_NS

    ### Optionally load needed modules for localmodconfig
    if [[ "$_localmodcfg" == "yes" ]]; then
        if [ -e "$_localmodcfg_path" ]; then
            echo "Running make localmodconfig..."
            make "${BUILD_FLAGS[@]}" LSMOD="${_localmodcfg_path}" localmodconfig
        else
            _die "No modprobed.db found at $_localmodcfg_path"
        fi
    fi

    ### Rewrite configuration
    echo "Rewrite configuration..."
    make "${BUILD_FLAGS[@]}" prepare
    yes "" | make "${BUILD_FLAGS[@]}" config >/dev/null
    diff -u ../config .config || :

    ### Prepared version
    make -s kernelrelease > version
    echo "Prepared $pkgbase version $(<version)"

    ### Running make nconfig
    [[ "$_makenconfig" == "yes" ]] && make "${BUILD_FLAGS[@]}" nconfig

    ### Running make xconfig
    [[ "$_makexconfig" == "yes" ]] && make "${BUILD_FLAGS[@]}" xconfig

    ### Save configuration for later reuse
    echo "Saving config for reuse..."
    cat .config > "${startdir}/config-${pkgver}-${pkgrel}${pkgbase#linux}"

    ### Prepare NVIDIA open modules with linux-ghost exclusive patches
    if [[ "$_nvidia_bundle" == "yes" ]]; then
        cd "${srcdir}"
        echo "Preparing NVIDIA open modules with linux-ghost patches..."

        # Standard compatibility patches
        echo "Applying atomic modesetting patch..."
        patch -Np1 -i "${srcdir}/nv-atomic-modesetting.patch" -d "${srcdir}/${_nv_open_pkg}/kernel-open" || true

        echo "Applying IBT support patch..."
        patch -Np1 -i "${srcdir}/nv-ibt-support.patch" -d "${srcdir}/${_nv_open_pkg}/" || true

        echo "Applying fbdev fix patch..."
        patch -Np1 -i "${srcdir}/nv-fbdev-fix.patch" -d "${srcdir}/${_nv_open_pkg}/" || true

        echo "Applying GCC 15 compatibility patch..."
        patch -Np1 -i "${srcdir}/nv-gcc15.patch" -d "${srcdir}/${_nv_open_pkg}/" || true

        # linux-ghost EXCLUSIVE patches
        echo "Applying Thunderbolt eGPU hot-plug support (PR #985)..."
        patch -Np1 -i "${srcdir}/nv-thunderbolt-egpu.patch" -d "${srcdir}/${_nv_open_pkg}/" || true

        echo "Applying UVM NULL pointer deref fix (PR #978)..."
        patch -Np1 -i "${srcdir}/nv-uvm-null-deref.patch" -d "${srcdir}/${_nv_open_pkg}/" || true
    fi
}

build() {
    cd "$_srcname"
    make "${BUILD_FLAGS[@]}" -j"$(nproc)" all
    make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1

    if [[ "$_nvidia_bundle" == "yes" ]]; then
        local MODULE_FLAGS=(
            KERNEL_UNAME="${_kernuname}"
            IGNORE_PREEMPT_RT_PRESENCE=1
            SYSSRC="${srcdir}/${_srcname}"
            SYSOUT="${srcdir}/${_srcname}"
            IGNORE_CC_MISMATCH=yes
        )
        cd "${srcdir}/${_nv_open_pkg}"
        echo "Building NVIDIA open modules..."
        CFLAGS= CXXFLAGS= LDFLAGS= make "${BUILD_FLAGS[@]}" "${MODULE_FLAGS[@]}" -j"$(nproc)" modules
    fi
}

_package() {
    pkgdesc="The $pkgdesc kernel and modules"
    depends=('coreutils' 'kmod' 'initramfs')
    optdepends=(
        'wireless-regdb: to set the correct wireless channels of your country'
        'linux-firmware: firmware images needed for some devices'
        'modprobed-db: Keeps track of EVERY kernel module probed'
        'scx-scheds: sched-ext schedulers (scx_lavd, scx_bpfland)'
        'nvidia-open-dkms: NVIDIA open kernel modules (RTX 20+/Blackwell)'
        'nvidia-utils: NVIDIA userspace utilities'
        'lib32-nvidia-utils: 32-bit NVIDIA libs (Steam/Wine)'
    )
    provides=(VIRTUALBOX-GUEST-MODULES WIREGUARD-MODULE KSMBD-MODULE NTSYNC-MODULE VHBA-MODULE)

    cd "$_srcname"
    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    echo "Installing boot image..."
    install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

    echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

    echo "Installing modules..."
    ZSTD_CLEVEL=19 make "${BUILD_FLAGS[@]}" INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
        DEPMOD=/doesnt/exist modules_install

    rm "$modulesdir"/build
}

_package-headers() {
    pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
    if [[ "$_compiler" == "llvm" ]]; then
        depends=('pahole' "${pkgbase}" 'clang' 'llvm' 'lld')
    else
        depends=('pahole' "${pkgbase}")
    fi

    cd "${_srcname}"
    local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

    echo "Installing build files..."
    install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
        localversion.* version vmlinux tools/bpf/bpftool/vmlinux.h
    install -Dt "$builddir/kernel" -m644 kernel/Makefile
    install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
    cp -t "$builddir" -a scripts
    ln -srt "$builddir" "$builddir/scripts/gdb/vmlinux-gdb.py"

    install -Dt "$builddir/tools/objtool" tools/objtool/objtool

    if [ -f tools/bpf/resolve_btfids/resolve_btfids ]; then
        install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids
    fi

    echo "Installing headers..."
    cp -t "$builddir" -a include
    cp -t "$builddir/arch/x86" -a arch/x86/include
    install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

    install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
    install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h
    install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h
    install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
    install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
    install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h
    install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

    echo "Installing KConfig files..."
    find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

    echo "Installing unstripped VDSO..."
    make INSTALL_MOD_PATH="$pkgdir/usr" vdso_install link=

    echo "Removing unneeded architectures..."
    local arch
    for arch in "$builddir"/arch/*/; do
        [[ $arch = */x86/ ]] && continue
        rm -r "$arch"
    done

    echo "Removing documentation..."
    rm -r "$builddir/Documentation"

    echo "Removing broken symlinks..."
    find -L "$builddir" -type l -printf 'Removing %P\n' -delete

    echo "Removing loose objects..."
    find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

    echo "Stripping build tools..."
    local file
    while read -rd '' file; do
        case "$(file -Sib "$file")" in
            application/x-sharedlib\;*)      strip -v $STRIP_SHARED "$file" ;;
            application/x-archive\;*)        strip -v $STRIP_STATIC "$file" ;;
            application/x-executable\;*)     strip -v $STRIP_BINARIES "$file" ;;
            application/x-pie-executable\;*) strip -v $STRIP_SHARED "$file" ;;
        esac
    done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

    echo "Stripping vmlinux..."
    strip -v $STRIP_STATIC "$builddir/vmlinux"

    echo "Adding symlink..."
    mkdir -p "$pkgdir/usr/src"
    ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

_package-nvidia-open() {
    pkgdesc="NVIDIA open kernel modules ${_nv_ver} for ${pkgbase} (RTX 5090 + Thunderbolt eGPU)"
    depends=("$pkgbase=$_kernver" "nvidia-utils=${_nv_ver}" "libglvnd")
    provides=('NVIDIA-MODULE')
    conflicts=('nvidia-open' 'nvidia-open-dkms')
    license=('MIT AND GPL-2.0-only')

    cd "$_srcname"
    local modulesdir="$pkgdir/usr/lib/modules/$(<version)/extramodules"

    cd "${srcdir}/${_nv_open_pkg}"
    install -dm755 "${modulesdir}"
    install -m644 kernel-open/*.ko "${modulesdir}"
    install -Dt "$pkgdir/usr/share/licenses/${pkgname}" -m644 COPYING

    # Install linux-ghost NVIDIA config (GSP stutter mitigation, etc.)
    install -Dm644 "${srcdir}/nvidia-ghost.conf" \
        "$pkgdir/etc/modprobe.d/nvidia-ghost.conf"

    find "$pkgdir" -name '*.ko' -exec zstd --rm -19 -T0 {} +

    echo "==> linux-ghost NVIDIA exclusive patches applied:"
    echo "    - Thunderbolt eGPU hot-plug support (PR #985)"
    echo "    - UVM NULL pointer deref fix (PR #978)"
    echo "    - GSP stutter mitigation config"
}

pkgname=("$pkgbase" "$pkgbase-headers")
[[ "$_nvidia_bundle" == "yes" ]] && pkgname+=("$pkgbase-nvidia-open")

for _p in "${pkgname[@]}"; do
    eval "package_$_p() {
        $(declare -f "_package${_p#$pkgbase}")
        _package${_p#$pkgbase}
    }"
done

