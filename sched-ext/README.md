# Linux Ghost Scheduler Profiles

Pre-configured sched-ext profiles and AMD V-Cache management for linux-ghost.

## Quick Start

```bash
# Install sched-ext schedulers
sudo pacman -S scx-scheds

# Gaming mode (scx_lavd)
sudo systemctl start scx-ghost-gaming

# Desktop mode (scx_bpfland)
sudo systemctl start scx-ghost-desktop
```

## Installation

```bash
# Copy services
sudo cp sched-ext/*.service /etc/systemd/system/
sudo cp sched-ext/ghost-vcache /usr/bin/
sudo chmod +x /usr/bin/ghost-vcache

# Reload systemd
sudo systemctl daemon-reload
```

## sched-ext Profiles

### Gaming (scx_lavd)

Optimized for low-latency gaming and interactive workloads:

```bash
# Start
sudo systemctl start scx-ghost-gaming

# Enable on boot
sudo systemctl enable scx-ghost-gaming

# Check status
systemctl status scx-ghost-gaming
```

**Features:**
- Latency-critical task detection
- Core compaction (power efficiency when idle)
- Per-LLC scheduling domains
- Optimized for frame pacing

### Desktop (scx_bpfland)

Balanced scheduler for desktop workloads:

```bash
sudo systemctl start scx-ghost-desktop
sudo systemctl enable scx-ghost-desktop
```

**Features:**
- vruntime-based fairness
- Interactive task prioritization
- Cache-aware core selection
- Lower overhead than scx_lavd

## AMD V-Cache Management (X3D)

For Ryzen 9950X3D, 7950X3D, 7900X3D processors.

### Prerequisites

1. BIOS Setting: CBS > CPU Common > CPPC > **Driver**
2. Kernel: CONFIG_AMD_3D_VCACHE=y (included in linux-ghost)

### Usage

```bash
# Check current mode
ghost-vcache status

# Gaming mode (prefer V-Cache CCD)
sudo ghost-vcache cache

# Productivity mode (prefer high-frequency CCD)
sudo ghost-vcache frequency

# Auto-detect based on running processes
sudo ghost-vcache auto
```

### Service (Auto V-Cache on Boot)

```bash
# Enable cache mode on boot (gaming default)
sudo systemctl enable ghost-vcache

# Start now
sudo systemctl start ghost-vcache
```

## Recommended Combinations

### Gaming Setup
```bash
sudo systemctl enable --now scx-ghost-gaming
sudo systemctl enable --now ghost-vcache
```

### Productivity Setup
```bash
sudo systemctl enable --now scx-ghost-desktop
# V-Cache defaults to frequency mode when service not running
```

### Hybrid (Manual Switching)
```bash
# Before gaming
sudo ghost-vcache cache
sudo systemctl start scx-ghost-gaming

# After gaming
sudo ghost-vcache frequency
sudo systemctl start scx-ghost-desktop
```

## Troubleshooting

### sched-ext not working
```bash
# Check if sched-ext is enabled
cat /sys/kernel/sched_ext/state

# Check for errors
journalctl -u scx-ghost-gaming -f
```

### V-Cache mode not changing
```bash
# Check driver loaded
ls /sys/bus/platform/drivers/amd_x3d_vcache/

# Check BIOS setting (must be "Driver" not "Auto")
dmesg | grep -i vcache
```

### Conflict with ananicy-cpp
Disable ananicy-cpp when using sched-ext:
```bash
sudo systemctl disable --now ananicy-cpp
```

## References

- [sched-ext GitHub](https://github.com/sched-ext/scx)
- [scx_lavd Documentation](https://sched-ext.com/docs/scheds/rust/scx_lavd)
- [AMD V-Cache Optimizer](https://www.phoronix.com/review/amd-3d-vcache-optimizer-9950x3d)
