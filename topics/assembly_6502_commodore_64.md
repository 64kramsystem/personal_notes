# Commodore 64

- [Commodore 64](#commodore-64)
  - [Memory map](#memory-map)
  - [PETSCII](#petscii)
  - [Debugging](#debugging)
  - [VICE](#vice)

## Memory map

(`w` = word)

- $0316 (w)   : `BRK` interrupt vector
- $0400-$07e7 : Screen memory
- $0801       : BASIC listings start
- $d021       : Background color

## PETSCII

Subset:

|  H->  |  $0   |      $1      |   $2    |  $3   |
| :---: | :---: | :----------: | :-----: | :---: |
|  $0   |   @   |      p       | {space} |   0   |
|  $1   |   a   |      q       |    !    |   1   |
|  $2   |   b   |      r       |    "    |   2   |
|  $3   |   c   |      s       |    #    |   3   |
|  $4   |   d   |      t       |    $    |   4   |
|  $5   |   e   |      u       |    %    |   5   |
|  $6   |   f   |      v       |    &    |   6   |
|  $7   |   g   |      w       |    '    |   7   |
|  $8   |   h   |      x       |    (    |   8   |
|  $9   |   i   |      y       |    )    |   9   |
|  $A   |   j   |      z       |    *    |   :   |
|  $B   |   k   |      [       |    +    |   ;   |
|  $C   |   l   |   {pound}    |    ,    |   <   |
|  $D   |   m   |      ]       |    -    |   =   |
|  $E   |   n   |  {up_arrow}  |    .    |   >   |
|  $F   |   o   | {back_arrow} |    /    |   ?   |


## Debugging

Modify the `BRK` [interrupt vector](#memory-map), and place `BRK`.

## VICE

The tested v3.6.1 hung after a few breakpoint triggers; v3.4 didn't.

Convenient configure: `./configure --config-cache --enable-cpuhistory --enable-external-ffmpeg --disable-rs232 --disable-ipv6 --disable-realdevice`; see the output of `./configure` for explanations about the options

Packages required for compiling: see installation script.

For recent versions (e.g. 3.6), must copy some GLSL files (`$repo/vice/vice/data/GLSL/`) to the data directory: `bicubic.frag`, `bicubic-interlaced.frag`, `builtin.frag`, `builtin-interlaced.frag`, `viewport.vert`

In order for VICE to load and autostart ASM programs, the setting `AutostartPrgMode` must be enabled (`=1`); otherwise, not only it won't autostart, but `RUN` will also not do anything
