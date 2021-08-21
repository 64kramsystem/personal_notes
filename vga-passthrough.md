# VGA Passthrough

- [VGA Passthrough](#vga-passthrough)
  - [nvidia-smi](#nvidia-smi)
  - [Xorg](#xorg)
  - [AMD power profiles](#amd-power-profiles)

## nvidia-smi

`persistenced` reference: https://docs.nvidia.com/deploy/driver-persistence/index.html#persistence-daemon

```sh
nvidia-smi --help-query-supported-clocks
nvidia-smi -i 1 --format=csv --query-supported-clocks=mem
nvidia-smi -i 1 --format=csv --query-supported-clocks=gr

nvidia-smi --help-query-gpu
nvidia-smi -i 1 --format=csv,noheader -l 1 --query-gpu=pstate,power.draw,clocks.mem,clocks.gr # P8, 13.54 W, 405 MHz, 300 MHz

# Power state can't be set.
# Power limits can be set, but the minimum is too high!
#
nvidia-smi -i 1 -ac 405,300 # mem/gr; not supported!
```

## Xorg

With `AutoAddGPU = false`, there is no process running on the card 1 (there is, without it), but still, unbinding hangs.

## AMD power profiles

See: https://wiki.archlinux.org/title/AMDGPU#Power_profiles.
