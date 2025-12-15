# Linux Ghost Scheduler Guide

## Overview

Linux Ghost supports multiple CPU scheduling strategies for different workloads.

## Built-in Schedulers

### BORE (Burst-Oriented Response Enhancer)

**Default scheduler for linux-ghost.**

BORE is built on top of EEVDF and adds burst-oriented response enhancement for better desktop interactivity.

```bash
# Build with BORE (default)
_cpusched=bore makepkg -sf
```

**Best for:**
- Desktop usage
- Gaming
- Interactive workloads
- Mixed workloads

### EEVDF (Earliest Eligible Virtual Deadline First)

Vanilla upstream scheduler. Clean, predictable behavior without additional heuristics.

```bash
# Build with vanilla EEVDF
_cpusched=eevdf makepkg -sf
```

**Best for:**
- Server workloads
- Predictable latency requirements
- Debugging scheduler issues

---

## sched-ext (Runtime Schedulers)

Linux Ghost includes `CONFIG_SCHED_CLASS_EXT=y` for runtime scheduler switching via BPF.

### Installation

```bash
sudo pacman -S scx-scheds
```

### Available Schedulers

| Scheduler | Use Case | Command |
|-----------|----------|---------|
| **scx_lavd** | Gaming, low-latency | `sudo scx_lavd` |
| **scx_bpfland** | Desktop, interactive | `sudo scx_bpfland` |
| **scx_rusty** | General purpose | `sudo scx_rusty` |
| **scx_flash** | Fairness focused | `sudo scx_flash` |

### scx_lavd (Recommended for Gaming)

Latency-criticality Aware Virtual Deadline scheduler. Designed specifically for gaming workloads.

```bash
# Start manually
sudo scx_lavd

# Or use linux-ghost service
sudo systemctl start scx-ghost-gaming
sudo systemctl enable scx-ghost-gaming
```

**Features:**
- Latency-critical task detection
- Core compaction (power efficiency when idle)
- Per-LLC scheduling domains
- Optimized frame pacing

### scx_bpfland (Recommended for Desktop)

vruntime-based scheduler prioritizing interactive workloads.

```bash
sudo scx_bpfland

# Or use linux-ghost service
sudo systemctl start scx-ghost-desktop
```

**Features:**
- Interactive task prioritization via voluntary context switch rate
- Cache-aware core selection
- Lower overhead than scx_lavd

---

## Linux Ghost Services

Pre-configured systemd services in `sched-ext/`:

```bash
# Copy services
sudo cp sched-ext/*.service /etc/systemd/system/
sudo systemctl daemon-reload

# Gaming mode
sudo systemctl enable --now scx-ghost-gaming

# Desktop mode
sudo systemctl enable --now scx-ghost-desktop
```

---

## Checking Active Scheduler

```bash
# Check if sched-ext is active
cat /sys/kernel/sched_ext/state

# Check which scheduler is loaded
cat /sys/kernel/sched_ext/root/ops

# Check BORE status
dmesg | grep -i bore
```

---

## Performance Tuning

### Disable ananicy-cpp

When using sched-ext, disable ananicy-cpp to avoid conflicts:

```bash
sudo systemctl disable --now ananicy-cpp
```

### CPU Governor

For maximum gaming performance:

```bash
# Set performance governor
sudo cpupower frequency-set -g performance

# Or use linux-ghost default (already set)
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

---

## Troubleshooting

### sched-ext not loading

```bash
# Verify kernel support
zcat /proc/config.gz | grep SCHED_CLASS_EXT
# Should show: CONFIG_SCHED_CLASS_EXT=y

# Check for errors
journalctl -u scx-ghost-gaming -f
```

### High latency with scx_lavd

Try adjusting performance mode:

```bash
sudo scx_lavd --performance
```

### Return to BORE

Simply stop the sched-ext scheduler:

```bash
sudo systemctl stop scx-ghost-gaming
# Or Ctrl+C if running manually
```

---

## References

- [sched-ext GitHub](https://github.com/sched-ext/scx)
- [scx_lavd Documentation](https://sched-ext.com/docs/scheds/rust/scx_lavd)
- [BORE Scheduler](https://github.com/firelzrd/bore-scheduler)
