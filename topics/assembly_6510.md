# Assembly 6510

- [Assembly 6510](#assembly-6510)
  - [Architecture](#architecture)
  - [Addressing modes](#addressing-modes)
  - [Instructions](#instructions)
    - [Illegal opcodes](#illegal-opcodes)
    - [Multiplication](#multiplication)
  - [Programming patterns](#programming-patterns)
    - [Table-based switch/case](#table-based-switchcase)

## Architecture

The CPU is little endian. 1 page = 256 bytes.

Page 0 has special treatment (addressing, etc.).

The stack is on page 1, and grows downwards. WATCH OUT!! The push/pull sequence is inverted compared to x86:

- push: write -> move
- pull: move -> read

so the stack top is the first available slot ($1FF).

## Addressing modes

(curly braces = optional)

- Absolute, X, Y   : 16 bits absolute, plus optional X or Y offset   ; `op16 {, X/Y}`
- Indirect         : 16 bits absolute -> 16 bits absolute            ; `[op16]`

- Zero page, X, Y  : Zero-page, plus optional X or Y offset          ; `op8 {, X/Y}`
- Indexed indirect : Zero-page plus X offset -> 16 bits absolute     ; `[op8 + X]`
- Indirect indexed : Zero-page -> 16 bits absolute -> plus Y offset  : `[op8], Y`

- Relative         : 8 bits relative, used by branches

- Immediate        : 1 byte operand                                  : `#op8`
- Accumulator      : single-byte function using A                    : `instrA` (e.g. `ASLA`)
- Implied          : single-byte function

## Instructions

"transfer" = copy

- `LDA`, `STA`             : LoaD/STore Accumulator
- `TAX`/`TAY`, `TXA`/`TYA` : Transfer X/Y <> Accumulator

- `CLC`, `SEC`     : CLear/SEt carry

- `JMP`
- `JSR`, `RTS`     : Jump to SubRoutine, ReTurn from Subroutine

- `CMP`, `CPX`, `CPY` : CoMPare accumulator/X/Y
- `BEQ`, `BNE`        : Branch if (Not) EQual
- `BCC`, `BCS`        : Branch if Carry Clear/Set

- `PHA`, `PLA`     : PusH/PuLl Accumulator
- `PHP`, `PLP`     : PusH/PuLl Processor status register
- `TXS`, `TSX`     : Transfer X <> Stack pointer

- `AND`, `ORA`, `EOR`    : Bitwise operations with accumulator; EOR = XOR
- `ASL`(`A`), `LSR`(`A`) : Arithmetic Shift Left/Logic Shift Right (address or Accumulator); spill to carry
- `ROL`, `ROR`           : ROtate Left/Right through carry

- `INC`, `INX`, `INY` : Increment address/X/Y; doesn't interact with the Carry
- `ADC`, `SBC`        : ADd/SuBtraCt accumulator with Carry

- `BRK`               : Trigger a non-maskable interrupt; see machine memory map for the IV

There are no unconditional jumps, so one must simulate via conditional: `CLC + BCC`.

### Illegal opcodes

- `ANC`: AND + set negative flag

### Multiplication

There is no multiplication. Multiplication needs to be decomposed into shifts and additions; pseudocode:

```
while A != 0
  if A & 1 = 1
    T = T + B

  A = A / 2
  B = B * 2
```

## Programming patterns

### Table-based switch/case

```asm
// Variable in `A`

ASLA          // A *= 2*2*2
ASLA
ASLA

STA bccA      // self-modifying code (kickass syntax)
CLC
BCC bccA:#$00

JSR case1
JMP end
NOP           // padding
NOP

// ...other cases...
```
