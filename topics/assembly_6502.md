# Assembly 6510

- [Assembly 6510](#assembly-6510)
  - [Architecture](#architecture)
  - [Addressing modes](#addressing-modes)
  - [Flags](#flags)
  - [Instructions](#instructions)
    - [Illegal opcodes](#illegal-opcodes)
    - [Multiplication](#multiplication)
  - [Programming patterns](#programming-patterns)
    - [16-bit operations](#16-bit-operations)
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

- Absolute, X, Y   : 16 bit absolute, plus optional X/Y offset     : `addr16 {, X/Y}`
- Indirect         : 16 bit absolute -> 16 bits absolute           : `(addr16)`       # `JMP` only!

- Zero page, X, Y  : 8 bit absolute (ZP), plus optional X/Y offset : `addr8 {, X/Y}`  # Wraps around at $100!!
- Indexed indirect : Zero-page plus X offset -> 16 bit addr        : `(addr8, X)`     # Also called "preindexed indirect"; typically used with X=0 for
                                                                                      # 16-bits ZP indirect addressing
- Indirect indexed : Zero-page -> 16 bit addr -> plus Y offset     : `(addr8), Y`     # Also called "postindexed indirect"

- Relative         : 8 bit relative, used by branches

- Immediate        : 1 byte operand                                : `#op8`
- Accumulator      : single-byte function using A                  : `instrA` (e.g. `ASLA`)
- Implied          : single-byte function

## Flags

- `N`egative
- O`V`erflow
- `B`reak
- `D`ecimal
- `I`nterrupt
- `Z`ero
- `C`array (see `SBC` for peculiarities!)

## Instructions

Convenient reference: https://www.pagetable.com/c64ref/6502/?tab=2#

"transfer" = copy

- `LDA`                    : LoaD Accumulator; WATCH OUT!! Affects flags ZN
- `STA`                    : STore Accumulator
- `TAX`/`TAY`, `TXA`/`TYA` : Transfer X/Y <> Accumulator

- `CLC`, `SEC`     : CLear/SEt carry

- `JMP`            : WATCH OUT !!! With Indirect addressing, the pointer MUST NOT be stored across pages ($..FF and $..00) !!!
- `JSR`, `RTS`     : Jump to SubRoutine, ReTurn from Subroutine

- `CMP`, `CPX`, `CPY` : CoMPare accumulator/X/Y
- `BEQ`, `BNE`        : Branch if (Not) EQual
- `BCC`, `BCS`        : Branch if Carry Clear/Set

- `PHA`, `PLA`     : PusH/PuLl Accumulator (no push X/Y!)
- `PHP`, `PLP`     : PusH/PuLl Processor status register
- `TXS`, `TSX`     : Transfer X <> Stack pointer

- `AND`, `ORA`, `EOR`    : Bitwise operations with accumulator; EOR = XOR
- `ASL`(`A`), `LSR`(`A`) : Arithmetic Shift Left/Logic Shift Right (address or Accumulator); spill to carry
- `ROL`, `ROR`           : ROtate Left/Right through carry

- `INC`, `INX`, `INY` : Increment address/X/Y; doesn't support A; flags: ZN
- `ADC`               : ADd accumulator with Carry
- `SBC`               : SuBtraCt accumulator with Carry; WATCH OUT !!!! If the C is set, it counts as 0, if it's clear, it counts as 1 !!!!

- `BRK`               : Trigger a non-maskable interrupt; see machine memory map for the IV

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

### 16-bit operations

Increment:

```asm
    INC lo_byte
    BNE !+
    INC hi_byte
!:
```

Decrement:

```asm
    LDA lo_byte
    BNE !+
    DEC hi_byte
!:  DEC lo_byte
```

Comparison:

```asm
    LDA v1_lo
    CMP v2_lo
    BNE not_equal
    LDA v1_hi
    CMP v2_hi
    BNE not_equal
equal:
```

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
