# Commodore 64

- [Commodore 64](#commodore-64)
  - [Memory map](#memory-map)
  - [Debugging](#debugging)
  - [VICE](#vice)

## Memory map

(`w` = word)

- $0316/w : `BRK` interrupt vector
- $0801   : BASIC listings start
- $d021   : Background color

## Debugging

Modify the `BRK` [interrupt vector](#memory-map), and place `BRK`.

## VICE

The tested v3.6.1 hung after a few breakpoint triggers; v3.4 didn't.

Convenient configure: `./configure --config-cache --enable-cpuhistory --enable-external-ffmpeg --disable-rs232 --disable-ipv6 --disable-realdevice`; see the output of `./configure` for explanations about the options

Packages required for compiling: see installation script.

For recent versions (e.g. 3.6), must copy some GLSL files (`$repo/vice/vice/data/GLSL/`) to the data directory: `bicubic.frag`, `bicubic-interlaced.frag`, `builtin.frag`, `builtin-interlaced.frag`, `viewport.vert`

In order for VICE to load and autostart ASM programs, the setting `AutostartPrgMode` must be enabled (`=1`); otherwise, not only it won't autostart, but `RUN` will also not do anything
