# Commodore 64: Programming (advanced)

- [Commodore 64: Programming (advanced)](#commodore-64-programming-advanced)
  - [PETSCII](#petscii)
  - [Input handling](#input-handling)
  - [Sprites](#sprites)
  - [Sound](#sound)
    - [Voice params](#voice-params)
    - [Voice control register](#voice-control-register)
  - [VICE](#vice)
    - [Monitor](#monitor)
    - [Work with D1541 files](#work-with-d1541-files)

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
