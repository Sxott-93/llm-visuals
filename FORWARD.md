# Forward (this fork)

Upstream: https://github.com/DingoOz/llm-visuals

This fork expands llm-visuals to **handle multiple GPUs from different vendors simultaneously** and adds sysfs-based monitoring for Intel Arc B70 hardware.

## Multi-vendor GPU support

The original code could detect both NVIDIA and Intel cards, but would only show one card type at a time. This fork fixes the collection pipeline so all detected GPUs render together regardless of whether they're NVIDIA (NVML/nvidia-smi), Intel (xpu-smi), or Apple Silicon (macmon/powermetrics).

## Intel Arc B70: sysfs patching

The upstream xpu-smi tool returns `"N/A"` for utilization, power, temperature, and name on Arc B70 hardware. This fork supplements `xpu-smi` with:

- **Name** — `xpu-smi discovery` table parsed from header lines.
- **Temperature** — `hwmon/temp2_input` (package sensor).
- **Fan speed** — `hwmon/fan1_input` (already had partial support).
- **GPU utilization** — delta of `tile0/gt{0,1}/gtidle/idle_residency_ms`.
- **Power draw** — delta of `hwmon/energy1_input` (μJ / Δt → W).

These fields are patched into the existing `GpuStats` struct when xpu-smi reports N/A, so the display is identical — no UI changes needed.
