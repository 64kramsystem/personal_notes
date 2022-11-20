# DOS and debugging

See virtualization DOSBox [section](virtualization.md#dosbox).

For general resources, see [Awesome DOS](https://github.com/balintkissdev/awesome-dos).

For compiling C programs, see C [section](c.md#compilers-for-16-bit-dos-target).

For a ready windows, see [this post](http://www.abandonia.com/vbullet/showthread.php?t=27770).

## DEBUG.COM commands

- `U [address]`          disassemble
- `T [=address] [count]` trace (execute instructions and stop); with address, it stops **past** it
- `A [address]`          assemble
- `D`                    dump
- `R [register [value]]` print/change registers

## DOSBox(-X)

### Key bindings

- `F12 + C`: for the configuration GUI
- `F12 + O`: swap floppy (unclear if there is a GUI option)

### Floppy

IMGs can be mounted via `IMGMOUNT` command; multiple images can be mounted (only via command), and swapped:

```sh
# See bindings for swapping.
#
IMGMOUNT A ms_c-cpp_compiler_19920818/*.IMG
```

### DPMI

The preinstalled DPMI server (CWSDPMI) crashes, and the common workaround (`-s-`) doesn't have any effect.

A working DPMI server is [HX](https://github.com/Baron-von-Riedesel/HX); run `C:\HX_PATH\BIN\HDPMI32 /r` to enable it.

### Basic configuration

Full config reference: https://github.com/joncampbell123/dosbox-x/blob/master/dosbox-x.reference.full.conf.

```conf
[sdl]
maximize = true

[dosbox]
fastbioslogo = true
startbanner = false
quit warning = false

[render]
aspect = true

[cpu]
# 486DX at 33MHz (https://dosbox-x.com/wiki/Guide%3ACPU-settings-in-DOSBox%E2%80%90X#_cycles)
# PII/300 = 200000
cycles = 12019
# Enable for fastest CPU speed. WATCH OUT! This is disabled on the first keypress.
# See https://dosbox-x.com/wiki/Guide%3AInstalling-Windows-3.x.
turbo = false

[fdc, primary]
# Enable for fastest floppy speed
instant mode = false

[autoexec]
# By default, the free size is whole disk; it's not clear if GBs of free size may cause programs to
# crash, so better restrict it. Size is in MB.
MOUNT C . -freesize 128
C:
IMGMOUNT A ms_c-cpp_compiler_19920818/*.IMG
```
