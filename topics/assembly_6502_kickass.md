# Kick Assembler (Assembly 6502)

- [Kick Assembler (Assembly 6502)](#kick-assembler-assembly-6502)
  - [Basic structure](#basic-structure)
  - [Data definition](#data-definition)
  - [Labels](#labels)
    - [Self-modifying code](#self-modifying-code)
  - [Block (locations)](#block-locations)
  - [Expressions](#expressions)
  - [Imports](#imports)
  - [Macros](#macros)

## Basic structure

```asm
// Predefined macro: generate a BASIC program with SYS $main
//
BasicUpstart2(main)

main:   inc $d021
	      jmp main
```

## Data definition

Binary data:

```asm
.byte $12, $34             // Other: word, dword
.text "Hello World"
.import binary "music.bin" // Import a file in memory!
```

Filling:

```
.fill 5, 0                                         // 0,0,0,0,0
.fill 256, 127.5 + 127.5*sin(toRadians(i*360/256)) // Sine curve!!

.fill 2, [$10, 'D']                                // $10, 'D', $10, 'D'
.fill 3, [i, i * $10]                              // 0, 0, 1, $10, 2, $20

.fillword 2, i * $80                               // .word $0000, $0080

// There are other functionalities
```

Vars/consts:

```asm
.var foo = 123                                     // the value is mandatory
.const bar = 456

// it may be more convenient to use labels, since they can be detected by some debuggers
foo: .byte 123
```

Hi/lo byte operators (work only with mnemonics):

```asm
lda #<VAL16     // Lower 8 bits of a 16-bits value
lda #>VAL16     // Higher
```

## Labels

There is support for multiple ("multi") labels, which can share the same name; they're convenient for inner loops:

```asm
!loop:
       // ...blah...
       bcc !loop-    // `-` references the previous loop, `+` the following one
```

Unnamed labels are also valid (`!`, `bCC !-`).

### Self-modifying code

```asm
sta bccA
clc
bcc bccA:#$00
```

## Block (locations)

Blocks define the location in memory. Labels are optional.

```asm
// Standard: block stored in the PRG.
//
* = $1000 "Music"

// Virtual: block not stored in memory.
//
// WATCH OUT! If this is in the zero page, place it before anything after the ZP, otherwise, the assembler
// will generate absolute references.
//
* = 2 virtual
```

## Expressions

WATCH OUT! Non-integer results of integer expressions are floats; they can cause mistakes if not rounded.

Math functions (subset):

- `floor(x)`
- `mod(a, b)` - there is no `%` operator!

## Imports

```asm
import "MyLibrary.asm"
importif STAND_ALONE "UpstartCode.asm"            // Conditional import
```

## Macros

```asm
main:   SetBorderColor(2)

// User-defined macro
//
.macro SetBorderColor(color) {
        lda #color
        sta $d020
}
```
