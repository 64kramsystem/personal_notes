# Commodore 64: System

- [Commodore 64: System](#commodore-64-system)
  - [Notes](#notes)
  - [Memory layout](#memory-layout)
  - [Memory map](#memory-map)
  - [Kernal routines](#kernal-routines)
    - [$E544: CLR\_SCREEN](#e544-clr_screen)
    - [$EA31: DEFAULT\_ISR](#ea31-default_isr)
    - [$EA81: ISR\_RETURN](#ea81-isr_return)
    - [$FF84: IOINIT](#ff84-ioinit)
    - [$FF90: SETMSG](#ff90-setmsg)
    - [$FF9F: SCNKEY](#ff9f-scnkey)
    - [$FFBD: SETNAM](#ffbd-setnam)
    - [$FFE4: GETIN](#ffe4-getin)
  - [Interrupts](#interrupts)
    - [$0314-$0315: IRQ vector](#0314-0315-irq-vector)
    - [$D01A: VIC interrupt mask](#d01a-vic-interrupt-mask)
  - [Screen](#screen)
    - [$D011: VIC control register 1](#d011-vic-control-register-1)
    - [$D012: Raster line](#d012-raster-line)
    - [$D016: VIC control register 2](#d016-vic-control-register-2)
    - [$D018: VIC memory control](#d018-vic-memory-control)
  - [CIA](#cia)
    - [$DC0E: CIA1 Timer A control](#dc0e-cia1-timer-a-control)
    - [$DD00: CIA2 Port A](#dd00-cia2-port-a)
  - [PETSCII](#petscii)
  - [Input handling](#input-handling)
  - [Sprites](#sprites)
  - [Sound](#sound)
    - [Voice params](#voice-params)
    - [Voice control register](#voice-control-register)
  - [Debugging](#debugging)
  - [VICE](#vice)
    - [Monitor](#monitor)
    - [Work with D1541 files](#work-with-d1541-files)

## Notes

Some chapters have tags (in the form "{tag_name}"), which are references to the memory map chapter.

## Memory layout

- BASIC ROM  | 8 KB ($A000–$BFFF)
- I/O area   | 4 KB ($D000–$DFFF) | rarely banked out, since one can't talk to HW
- KERNAL ROM | 8 KB ($E000–$FFFF)

## Memory map

- Simpler: https://sta.c64.org/cbm64mem.html
  - Index of all docs on the website: https://sta.c64.org/cbmdocs.html
- Detailed: http://www.zimmers.net/anonftp/pub/cbm/c64/manuals/mapping-c64.txt

| hex         | dec         | tag        | description                                                                            |
| :---------- | :---------- | ---------- | -------------------------------------------------------------------------------------- |
| $0001       | 1           |            | MEMCFG: CPU port memory map configuration                                              |
| $0024       |             | screen     | CURSOR_COL: Current horizontal cursor position (0-39)                                  |
| $00C5       | 197         | input      | KEYPRESS: Current key pressed                                                          |
| $00C6       | 198         | input      | KEYBUF_LEN: Keyboard buffer length                                                     |
| $00D6       |             | screen     | CURSOR_ROW: Current vertical cursor position (0-24)                                    |
| $0277-$0280 | 631-640     | input      | KEYBUF: Keyboard buffer (10 bytes)                                                     |
| $0314-$0315 | 788-789     | irq        | IRQ_VEC: Hardware IRQ vector                                                           |
| $0316-$0317 | 790-791     | debug      | BRK_VEC: BRK interrupt vector                                                          |
| $0334-$033B | 820-827     |            | empty                                                                                  |
| $033C-$03FB | 828-1019    | datasette  | DATASETTE_BUF: Datasette buffer (reusable)                                             |
| $03FC-$03FF | 1020-1023   |            | empty                                                                                  |
| $0400-$07E7 | 1024-2023   | screen     | SCREEN_RAM: Default screen memory                                                      |
| $07F8-$07FF | 2040-2047   | sprites    | SPRITE_PTRS: Sprite shape data pointers (addr=n*64)                                    |
| $0801       | 2049        | basic      | BASIC_START: BASIC program start                                                       |
| $C000-$CFFF | 49152-53247 |            | UPPER_RAM: Free RAM (4 KB)                                                             |
| $D000-$D001 | 53248-53249 | sprites    | SP0_XY: Sprite 0 X/Y coordinates                                                       |
| $D011       | 53265       | screen     | VIC_CR1: VIC control register 1 (screen height, bitmap mode, display enable, Y scroll) |
| $D012       | 53266       | irq        | VIC_RASTER: Raster line register (low 8 bits; bit 8 in $D011 bit 7)                    |
| $D015       | 53269       | sprites    | SPRITE_EN: Sprite enable bitmap                                                        |
| $D016       | 53270       | screen     | VIC_CR2: VIC control register 2 (screen width, multicolor mode, X scroll)              |
| $D018       | 53272       | screen     | VIC_MEMCTL: Screen RAM and charset base address selector                               |
| $D019       | 53273       | irq        | VIC_ISTAT: VIC interrupt status register (write to acknowledge)                        |
| $D01A       | 53274       | irq        | VIC_IMASK: VIC interrupt mask register (bit 0=raster IRQ)                              |
| $D01E       | 53278       | sprites    | SP_SP_COLL: Sprite-sprite collision register                                           |
| $D01F       | 53279       | sprites    | SP_BG_COLL: Sprite-background collision register                                       |
| $D020       | 53280       | screen     | BORDCOL: Border color                                                                  |
| $D021       | 53281       | screen     | BGCOL: Background color                                                                |
| $D027-$D02E | 53287-53294 | sprites    | SP_COLORS: Sprite colors                                                               |
| $D400-$D414 | 54272-54292 | sound      | SID_VOICES: SID voice 1/2/3 params (7 bytes each)                                      |
| $D418       | 54296       | sound      | SID_VOL: SID volume and filter modes                                                   |
| $D41B       | 54299       | sound      | SID_V3OUT: Voice 3 waveform output (random number source)                              |
| $DC00-$DC01 | 56320-56321 | input      | JOY_PORTS: Joystick 2/1 ports (active low)                                             |
| $DC0E       |             | cia        | CIA1_CRA: CIA1 Timer A control register                                                |
| $DD00       |             | cia        | CIA2_PRA: VIC bank select (bits 0-1 inverted; %00=bank 3 $C000-$FFFF)                  |
| $E544       | 58692       | kernal_rom | CLR_SCREEN: Clear screen                                                               |
| $EA31       | 59953       | kernal_rom | DEFAULT_ISR: Default interrupt service routine                                         |
| $EA81       | 60033       | kernal_rom | ISR_RETURN: Interrupt return (restore regs + RTI, skips keyboard/clock)                |
| $FF84       |             | kernal_rom | IOINIT: Initialize CIA, SID, VIC to defaults                                           |
| $FF90       |             | kernal_rom | SETMSG: Enable/disable KERNAL control and error messages                               |
| $FF9F       |             | kernal_rom | SCNKEY: Scan keyboard matrix                                                           |
| $FFBD       |             | kernal_rom | SETNAM: Set filename for LOAD/SAVE/OPEN                                                |
| $FFE4       |             | kernal_rom | GETIN: Read one character from input device (non-blocking)                             |

## Kernal routines

### $E544: CLR_SCREEN

- Clears the screen and homes the cursor
- Internal KERNAL routine (not in the jump table)

### $EA31: DEFAULT_ISR

- Default interrupt service routine
- Handles keyboard scanning (calls SCNKEY), cursor blinking, and system clock
- Runs 60 times/second via the IRQ vector at $0314-$0315
- Games that hook the IRQ typically JMP here at the end to preserve keyboard/clock

### $EA81: ISR_RETURN

- Tail end of the default ISR: restores registers from stack and executes RTI
- Skips keyboard scanning, cursor blinking, and clock update
- Used by custom IRQ handlers that only need a clean interrupt exit (JMP $EA81)
- Used by NE: both raster IRQ handlers ($A7E0 and $A880) jump here

### $FF84: IOINIT

- Initializes CIA, SID, and VIC-II to their default states
- Resets CIA timers, clears SID registers, sets VIC defaults
- Used by NE: called during INIT_GAME ($06A0)

### $FF90: SETMSG

- Controls KERNAL control/error message output
- A = bitmask: bit 7 = error messages, bit 6 = control messages
- $00 = suppress all, $C0 = enable both (BASIC default)
- Used by NE: set to $00 during INIT_GAME ($06A5) to suppress messages

### $FF9F: SCNKEY

- Scans the keyboard matrix via CIA1
- Updates the internal key buffer at $0277-$0280 (10-byte circular buffer)
- Sets $C6 (number of keys in buffer)
- Normally called by the IRQ handler 60 times/second - if a game disables IRQs, it calls this manually to keep the keyboard working

### $FFBD: SETNAM

- Sets the filename for subsequent LOAD/SAVE/OPEN calls
- A = filename length, X/Y = pointer to filename (lo/hi)
- Used by NE: called at $972B (likely for save/load game)

### $FFE4: GETIN

- Reads one character from the current input device (default: keyboard buffer)
- Returns the PETSCII code in A, or $00 if no key is pending
- Doesn't block; returns 0 if nothing is there

## Interrupts

### $0314-$0315: IRQ vector

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

### $D01A: VIC interrupt mask

Controls which VIC-II interrupts are enabled. Bit 0 enables the raster interrupt, which fires when the raster beam reaches the line set in $D012. This is the most common way games synchronize with the display.

## Screen

### $D011: VIC control register 1

Bits:
- 0-2: Y scroll (default 3)
- 3: row select (0 = 24 rows, 1 = 25 rows)
- 4: display enable (0 = blanked, 1 = visible)
- 5: bitmap mode (0 = character mode, 1 = bitmap mode)
- 6: extended color mode
- 7: raster compare bit 8 (high bit of $D012)

Common values:
- `$1B` = %00011011: standard text, 25 rows, display on, Y scroll 3
- `$3B` = %00111011: bitmap mode, 25 rows, display on
- `$0B` = %00001011: display OFF (bit 4 = 0)

### $D012: Raster line

Low 8 bits of the raster compare line. When the VIC beam reaches this line (combined with bit 7 of $D011), a raster interrupt fires if enabled in $D01A.

### $D016: VIC control register 2

Bits:
- 0-2: X scroll (default 0)
- 3: column select (0 = 38 columns, 1 = 40 columns)
- 4: multicolor mode (0 = hires, 1 = multicolor)
- 5-7: not connected (ignored on write)

Common values:
- `$C8` = %11001000: 40 columns, hires, X scroll 0
- `$D8` = %11011000: 40 columns, multicolor, X scroll 0

### $D018: VIC memory control

- Bits 4-7: screen RAM offset within VIC bank (x $0400)
- Bits 1-3: character set / bitmap base offset within VIC bank (x $0800)
- Bit 0: unused

Example: `$3E` in VIC bank 3 ($C000): screen at $CC00, charset at $F800 (character ROM).

## CIA

### $DC0E: CIA1 Timer A control

CIA1 Timer A drives the IRQ that handles keyboard scanning, cursor blinking, and the system clock. Stopping it (clearing bit 0) disables those IRQs — typically done during time-critical or interrupt-sensitive operations to prevent the IRQ from firing mid-routine and corrupting state.

### $DD00: CIA2 Port A

Bits 0-1 (inverted) select the VIC-II memory bank out of four $4000-sized regions. %00 = bank 3 ($C000-$FFFF). The remaining bits control the serial bus and RS-232 lines.

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


## Input handling

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

## Sprites

A sprite is 3x21 (WxH) bytes (=24x21 pixels). The top left corner is x=24, y=50; bottom right (addressable) corner is x=255+24, y=229+21.

In order to use x>255, complex manipulation is required (see https://www.lemon64.com/forum/viewtopic.php?t=15318&sid=9c1ea88879666603c6dfc064c09f5ac5).

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

## Sound

### Voice params

- V  , V+1: Frequency
- V+2, V+3: Pulse waveform width
- V+4     : Main control register
- V+5     : Attack/decay
- V+6     : Sustain/release

### Voice control register

Bits:

- 0: 0 = Voice off, Release cycle; 1 = Voice on, Attack-Decay-Sustain cycle
- 1: 1 = Synchronization enabled
- 2: 1 = Ring modulation enabled
- 3: 1 = Disable voice, reset noise generator
- 4: 1 = Triangle waveform enabled
- 5: 1 = Saw waveform enabled
- 6: 1 = Rectangle waveform enabled
- 7: 1 = Noise enabled

## Debugging

Modify the `BRK` [interrupt vector](#memory-map), and place `BRK`.

## VICE

In order to copy/paste, source listings must be lower case.

In order for VICE to load and autostart ASM programs, the setting `AutostartPrgMode` must be enabled (`=1`); otherwise, not only it won't autostart, but `RUN` will also not do anything.


### Monitor

Listens on port 6510.

Addresses must be in hex, and 4 digits long.

Commands:

- `d[isass] [<start> [<end>]]`      : disassemble
- `ret[urn]`                        : continue machine execution
- `s[ave] "<path>" 0 <start> <end>` : dump memory; first two bytes are `<start>`
- `l[oad] "<path>" 0 <start>`       : load memory dump
- `n[ext] <count>`                  : step over
- `z`                               : step in

### Work with D1541 files

Files extractor; notes:

- doesn't support all file types, eg. REL
- files with non-ASCII chars will cause errors and won't be copied
- errors don't make c1541 exit with error

```sh
# Skip header and footer (optional, since the perl expression would filter them out anyway).
# WATCH OUT to the space before the end of each line!
#
files_list=$(c1541 "$d64_file" -dir | head -n -1 | tail -n +2 | perl -lne 'print "$1 $2" if /"(.+)" +(\w+) $/')

while IFS= read -r file_entry; do
  filename=${file_entry% *}
  extension=${file_entry//* /}

  output_file=$out_dir/$filename.$extension

  if [[ $extension == seq ]]; then
    filename=$filename,s
  fi

  c1541 "$d64_file" -read "$filename" "$output_file"
done <<< "$files_list"
```
