# Kick Assembler (Assembly 6502)

- [Kick Assembler (Assembly 6502)](#kick-assembler-assembly-6502)
  - [Basic structure](#basic-structure)
  - [Data definition](#data-definition)
  - [Labels](#labels)
    - [Self-modifying code](#self-modifying-code)
  - [Memory handling: Block/Segments](#memory-handling-blocksegments)
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

// WATCH OUT!! This can't be used with the pattern lo/hi of a label (address):

lda #>my_label  // Doesn't decode to (my_label + 1)!!, instead, to the hi byte of my_label
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

## Memory handling: Block/Segments

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

Zero-page labels; allow ZP forward-resolution of labels:

```asm
* = $10 virtual
.zp {
zpReg1: .byte 0
zpReg2: .byte 0
}
```

A segment is a list of memory blocks. If none is defined, a `Default` segment is created.

```asm
      .segmentdef MySegment1                // Start if defined where used
      .segmentdef MySegment2 [start=$1000]

      // The second parameter (optional) implicitly defines a named memory block.
      .segment MySegment1 "First Block"
      *=$4000
      ldx #30
l1:   inc $d021
      dex
      bne l1

      // Start at the location defined in the declaration.
      // This will place code in the default memory block (not `Default` segment!), since no block is specified.
      .segment MySegment2
      inc $d021
      jmp *-3

      // Append code
      .segment MySegment1 "Second Block"
      inc $d020
      jmp *-3

      // Shorthand for declaration+definition.
      .segment MySegment3 [start=$2000]
```

More advanced segment concepts:

```asm
      // Define boundaries; if the code exceeds them, an error is raised.
      .segment Data [start=$c000, min=$c000, max=$cfff]

      // Two segments can overlap; a use case for this is to reuse some memory that is no longer needed
      // after use (e.g. init code).
      .segmentdef Code      [start=$1000]
      .segmentdef InitCode  [startAfter="Code"]
      .segmentdef Buffer    [startAfter="Code"]
```

Example multi-segment structure ([source](https://www.lemon64.com/forum/viewtopic.php?p=960735#960735)):

```asm
.segment ZeroPage[start = $2, virtual]
.segment Code[start = $801, max = $1fff]
.segment BSS[start = $c000, virtual, min = $c000, max = $cfff]

Main:
{
   .const START_VAR_1     = 0
   .const SCREEN_LOCATION = $0400

   .segment ZeroPage
   .zp
   {
      param_1:    .byte 0
      param_2_lo: .byte 0
      param_2_hi: .byte 0
   }

   .segment Code
   BasicUpstart2(Entry)

   #import "another_file.asm"

   .segment BSS
   var_2_lo: .byte 0
   var_2_hi: .byte 0

   .segment Code
   initialised_var_1:    .byte 123
   Entry:
   {
      lda START_VAR_1
      sta zp.param_1
      // etc
   }

   .segment BSS
   .file[name = "my.prg", segments = "Default,Code"]
}
```

## Expressions

WATCH OUT! Non-integer results of integer expressions are floats; they can cause mistakes if not rounded.

Math functions (subset):

- `floor(x)`
- `mod(a, b)` - there is no `%` operator!

## Imports

```asm
#import "MyLibrary.asm"
#importif STAND_ALONE "UpstartCode.asm"            // Conditional import
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
