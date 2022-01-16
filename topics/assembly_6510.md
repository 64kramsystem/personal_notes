# Assembly 6510

- [Assembly 6510](#assembly-6510)
  - [Architecture](#architecture)
  - [Addressing modes](#addressing-modes)
  - [Instructions](#instructions)
    - [Illegal opcodes](#illegal-opcodes)
    - [Multiplication](#multiplication)

## Architecture

The CPU is little endian. 1 page = 256 bytes.

Page 0 has special treatment (addressing, etc.).

The stack is on page 1, and grows downwards. WATCH OUT!! The push/pull sequence is inverted compared to x86:

- push: write -> move
- pull: move -> read

so the stack top is the first available slot ($1FF).

## Addressing modes

- Absolute, X, Y   : 16 bits absolute, plus optional X or Y offset   ; `op (, X|Y)`
- Indirect         : 16 bits absolute -> 16 bits absolute            ; `[op]`

- Zero page, X, Y  : Zero-page, plus optional X or Y offset          ; `op (, X|Y)`
- Indexed indirect : Zero-page plus X offset -> 16 bits absolute     ; `[op + X]`
- Indirect indexed : Zero-page -> 16 bits absolute -> plus Y offset  : `[op], Y`

- Relative         : 8 bits relative, used by branches

- Immediate        : 1 byte operand                                  : `#op`
- Accumulator      : single-byte function using A                    : `instrA` (e.g. `ASLA`)
- Implied          : single-byte function

## Instructions

"transfer" = copy

- `LDA`, `STA`     : LoaD/STore Accumulator
- `TAX`, `TXA`     : Transfer X <> Accumulator

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

- `INC`, `INX`, `INY` : Increment address/X/Y
- `ADC`, `SBC`        : ADd/SuBtraCt accumulator with Carry

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
