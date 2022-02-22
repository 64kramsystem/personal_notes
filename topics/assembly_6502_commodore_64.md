# Commodore 64

- [Commodore 64](#commodore-64)
  - [Notes](#notes)
  - [Memory map](#memory-map)
  - [Input handling {input}](#input-handling-input)
  - [Interrupt hooking](#interrupt-hooking)
  - [PETSCII {input}](#petscii-input)
  - [Sprites {sprites}](#sprites-sprites)
  - [Debugging {debug}](#debugging-debug)
  - [Utility routines](#utility-routines)
    - [Random number](#random-number)
  - [VICE](#vice)
    - [Monitor](#monitor)
    - [Compilation](#compilation)

## Notes

Some chapters have tags (in the form "{tag_name}"), which are references to the memory map chapter.

## Memory map

- Simpler: https://sta.c64.org/cbm64mem.html
- Detailed: http://www.zimmers.net/anonftp/pub/cbm/c64/manuals/mapping-c64.txt

| hex         | dec         | description                           | tag     |
| :---------- | :---------- | ------------------------------------- | ------- |
| $00c5       | 197         | Key currently pressed                 | input   |
| $00c6       | 198         | Length of pressed key buffer          | input   |
| $0277-$0280 | 631-640     | Buffer last keys pressed              | input   |
| $0314-$0315 | 788-789     | Service interrupt vector              |         |
| $0316-$0317 | 790-791     | `BRK` interrupt vector                | debug   |
| $0334-$033b | 820-827     | empty                                 |         |
| $033c-03fb  | 828-1019    | datasette buffer (usable)             |         |
| $03fc-03ff  | 1020-1023   | empty                                 |         |
| $0400-$07e7 | 1024-2023   | Screen memory                         |         |
| $07f8-$07ff | 2040-2047   | Sprite shape data pointers (loc=n*64) | sprites |
| $0801       | 2049        | BASIC listings start                  |         |
| $C000-$CFFF | 49152-53247 | Upper RAM                             |
| $d000-$d001 | 53248-53249 | Sprite 0 X/Y                          | sprites |
| $d015       | 53269       | Sprites enabled bitmap                | sprites |
| $d01e       | 53278       | Sprite-sprite log bitmap              | sprites |
| $d01f       | 53279       | Sprite-background log bitmap          | sprites |
| $d021       | 53281       | Background color                      |         |
| $d027-$d02e | 53287-53294 | Sprite colors                         | sprites |
| $dc00-$dc01 | 56320-56321 | Joystick 2/1 ports                    | input   |

## Input handling {input}

The last keys pressed buffer can be cleared by just resetting the length: `POKE 198, 0`.

Table for decoding the joystick (255 is -1):

| Value obtained |  Direction  | X Vector | Y Vector |
| :------------: | :---------: | :------: | :------: |
|      0-4       |      -      |    -     |    -     |
|       5        |  Left+Down  |    1     |    1     |
|       6        |  Right+Up   |    1     |   255    |
|       7        |    Right    |    1     |    0     |
|       8        |      -      |    -     |    -     |
|       9        | Left + Down |   255    |    1     |
|       10       |  Left + Up  |   255    |   255    |
|       11       |    Left     |   255    |    0     |
|       12       |      -      |    -     |    -     |
|       13       |    Down     |    0     |    1     |
|       14       |     Up      |    0     |   255    |
|       15       |      -      |    -     |    -     |

ASM routine:

```asm
joystick_mapping: .byte 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
                        1, 1, 1, 255, 1, 0, 0, 0, 255, 1, 255, 255, 255, 0, 0, 0, 0, 1, 0, 255,
                        0, 0

        lda $dc01                   // get joystick position
        and #15                     // isolate meaningful bytes
        asl                         // multiply by 2 (each position is associated to two values)
        tax
        lda joystick_mapping, x     // lookup x displacement
        clc
        adc $d000                   // add displacement to sprite #0 x
        sta $d000
        lda joystick_mapping + 1, x // lookup y displacement
        clc
        adc $d001                   // add displacement to sprite #0 y
        sta $d001
        rts
```

## Interrupt hooking

The [service interrupt vector](#memory-map) runs by default 60 times per second (the frequency can be changed), and handles keyboard, cursor and clock. It can be hooked, but in order to maintain the original functionalities, jmp to the default location (`$ea31`) at the end of the hook.

A typical structure is:

```asm
        dec int_counter # execute the action only #INT_COUNT
        bne exit
        # ... action ...
        lda #INT_COUNT
        sta int_counter
exit:   jmp $ea31       # or jump to next hook, if there are multiple
```

Use `RTI` when not calling the interrupt service.

## PETSCII {input}

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

## Sprites {sprites}

A sprite is 3x21 (WxH) bytes.

Basic example (not smooth):

- Point the sprite 0 shape to (15 * 64 = 960);
- Set all the bits to 1
- Activate sprite 0
- Cycle: increase the x and y coordinates by 3 (making sure they don't exceed 255)

```bas
10 POKE 2040,15
20 FOR A=960 TO 1022 : POKE A,255 : NEXT
22 POKE 53269,1
30 X=PEEK(53248) : POKE 53248, (X+3) AND 255
32 Y=PEEK(53249) : POKE 53249, (X+3) AND 255
34 GOTO 30
```

## Debugging {debug}

Modify the `BRK` [interrupt vector](#memory-map), and place `BRK`.

## Utility routines

### Random number

Source: https://www.atarimagazines.com/compute/issue72/random_numbers.php.

```asm
preset_rnd:
        lda #$ff  // maximum frequency value
        sta $d40e // voice 3 frequency low byte
        sta $d40f // voice 3 frequency high byte
        lda #$80  // noise waveform, gate bit off
        sta $d412 // voice 3 control register
        rts

// Write a random number to A, without using any other register.
// Need to waste some cycles, since a new value is generated every 26 cycles (see https://www.lemon64.com/forum/viewtopic.php?t=70039)
get_rnd:
        lda #10
!:      sec
        sbc #1
        bcs !-
        lda $d41b
        rts
```

## VICE

In order to copy/paste, source listings must be lower case.

In order for VICE to load and autostart ASM programs, the setting `AutostartPrgMode` must be enabled (`=1`); otherwise, not only it won't autostart, but `RUN` will also not do anything

### Monitor

Just telnet to port 6510!

Addresses must be in hex, and 4 digits long.

- `d[isass] [<start> [<end>]]`      : disassemble
- `ret[urn]`                        : continue machine execution
- `s[ave] "<path>" 0 <start> <end>` : dump memory; first two bytes are `<start>`
- `l[oad] "<path>" 0 <start>`       : load memory dump
- `n[ext] <count>`                  : step over
- `z`                               : step in

### Compilation

Convenient configure: `./configure --config-cache --enable-cpuhistory --enable-external-ffmpeg --disable-rs232 --disable-ipv6 --disable-realdevice`; see the output of `./configure` for explanations about the options

Packages required for compiling: see installation script.

For recent versions (e.g. 3.6), must copy some GLSL files (`$repo/vice/vice/data/GLSL/`) to the data directory: `bicubic.frag`, `bicubic-interlaced.frag`, `builtin.frag`, `builtin-interlaced.frag`, `viewport.vert`
