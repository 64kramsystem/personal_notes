# Commodore 64

- [Commodore 64](#commodore-64)
  - [Hello world (C64)](#hello-world-c64)
  - [VICE Debugging](#vice-debugging)

## Hello world (C64)

Changes screen colors rapidly (Kick Assembler):

```asm
// Generate a BASIC program with SYS $main
//
BasicUpstart2(main)

main:
	inc $d021
	jmp main
```

Manual (non-macro) version:

```asm
// BASIC listings start.
//
* = $0801

// Encoded "10 SYS 2064"
//
.byte $0E, $08, $0A, $00, $9E, $20, $28, $32
.byte $30, $36, $34, $29, $00, $00, $00

* = $0810

main:
	inc $d021
	jmp main
```

## VICE Debugging

In order for VICE to load and autostart ASM programs, the setting `AutostartPrgMode` must be enabled (`=1`); otherwise, not only it won't autostart, but `RUN` will also not do anything.
