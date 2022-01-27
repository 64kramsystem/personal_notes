# Kick Assembler (Assembly 6502)

- [Kick Assembler (Assembly 6502)](#kick-assembler-assembly-6502)
  - [Basic structure](#basic-structure)
  - [Data definition](#data-definition)
  - [Imports](#imports)
  - [Macros](#macros)

## Basic structure

```asm
// Predefined macro: generate a BASIC program with SYS $main
//
BasicUpstart2(main)

main:   inc $d021
	      jmp main

* = $1000 "Music"                                  // Set the current location; the label is optional
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
```

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
