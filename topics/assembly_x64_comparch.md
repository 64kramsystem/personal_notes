# Assembly x64

- [Assembly x64](#assembly-x64)
  - [Basic structure](#basic-structure)
  - [Data types](#data-types)
    - [IEEE754](#ieee754)
    - [Boolean representation](#boolean-representation)
  - [Stack alignment/frame](#stack-alignmentframe)
  - [Addressing](#addressing)
    - [Pages (MMU)](#pages-mmu)
  - [Registers](#registers)
  - [Flags](#flags)
  - [Control flow patterns](#control-flow-patterns)
  - [Instructions](#instructions)
    - [Control flow](#control-flow)
      - [Conditional jump flags/SetCC/CmovCC](#conditional-jump-flagssetcccmovcc)
    - [Storage](#storage)
    - [String operations](#string-operations)
    - [Sign-extension](#sign-extension)
    - [Arithmetic](#arithmetic)
      - [Multiplication](#multiplication)
      - [Division](#division)
    - [Flags](#flags-1)
    - [Comparison/Bit testing/manipulation:](#comparisonbit-testingmanipulation)
    - [Floating-point/SSE](#floating-pointsse)
    - [Other](#other)
  - [Optimizations](#optimizations)
    - [Low-level](#low-level)
    - [Higher level](#higher-level)
      - [Control flow](#control-flow-1)
  - [Syscalls](#syscalls)
  - [Utilities](#utilities)
  - [General concepts](#general-concepts)
    - [JIT and calls](#jit-and-calls)

## Basic structure

```asm
; The other sections edited out; see notes for the specific assembler.

main:
; Function prologue ("stack frame" setup)
    push           rbp
    mov            rbp, rsp

; System call
    mov            rax, 1         ; 1 = write
    mov            rdi, 1         ; 1 = to stdout
    mov            rsi, msg
    mov            rdx, msgLen
    syscall

; Function epilogue
    mov            rsp, rbp
    pop            rbp

; Exit. At the top-level, can also exit with `mov al, 0; ret`
    mov            rax, 60     ; exit
    mov            rdi, 0      ; exit code
    syscall
```

## Data types

- `db`, `dw`, `dd`, `dq`
- `resb`, `resw`, `resres`, `resq`

The `word` size depends on the architecture!!

Assuming a 16-bit arch, the 128-bit size is `long word`.

### IEEE754

Precision bits (sgn, exp, mant)

- Single precision bits: 1 + 8  + 23
- Double precision bits: 1 + 11 + 52

Exponent: signed
Mantissa: 1.0 + 0.5^(left position 1-based); there are special cases

Max interval of integers that can be represented exactly (see https://blog.demofox.org/2017/11/21/floating-point-precision etc.):

- f32: ±2^24 = ±16_777_216
- f64: ±2^53 = ±9_007_199_254_740_992

```rb
9007199254740993.0 # => 9.007199254740992e+15 😬
```

### Boolean representation

"Logical systems", and advantages:

- "boolean" (0, 1): the values are consistent with the ASM bit operations (except not, but can use `xor op, 1` (!)) and setCC
- "arithmetic" (0, anything else): non-1 false values don't need conversion

## Stack alignment/frame

Stack alignment is required by some instructions; the stack frame is useful primarily for debuggers.

When there is complete control over a workflow (e.g. leaf routine), the two can be omitted.

If one doesn't know the alignment state, they can use:

```asm
; irreversible; can also only mask RSP, if possible in that context:
;
lea rax, [rsp + 0xf]
and rax, -0x10

; reversible
;
xor r15, r15
test rsp, 0xf        ; not aligned?
setnz r15b           ; set if not aligned
shl r15, 3           ; multiply by 8
sub rsp, r15         ; substract 0 or 8
call $myfunc
add rsp, r15         ; add 0 or 8 to restore rsp
```

## Addressing

Syntax:

```asm
mov rsi, radius      ; WATCH OUT!! Stores *the address*
mov rsi, [radius]    ; Dereferences an address (stores *the value*)
lea rsi, [radius]    ; Stores the address
```

Addressing modes (effective = end value):

- Register          | `reg, reg`
- PC-relative       | `op, symbol`                                 | WATCH OUT! The displacement is encoded, not the effective address!
- Register-indirect | `[reg64], op`                                | Effective addr.
- Indirect+offset   | `[reg64 ± displ]`                            | Effective addr.
- Scaled-indexed    | `[base_reg64 + index_reg64 * scale ± displ]` | Effective addr.; scale = 1,2,4,8

If `LARGEADDRESSAWARE` is disabled, other PC-relative addressing modes are available (e.g. `var[reg64]`), however, they can address only ±2 GB.

### Pages (MMU)

In x64, pages are 4k; they're managed by the MMU, and can be Read/Write/Executable.

## Registers

General purpose registers:

| 64-bit  | 32-bit | 16-bit | high 8-bit | low  8-bit |
| :-----: | :----: | :----: | :--------: | :--------: |
|   rax   |  eax   |   ax   |     ah     |     al     |
|   rbx   |  ebx   |   bx   |     bh     |     bl     |
|   rcx   |  ecx   |   cx   |     ch     |     cl     |
|   rdx   |  edx   |   dx   |     dh     |     dl     |
|   rsi   |  esi   |   si   |     -      |    sil     |
|   rdi   |  edi   |   di   |     -      |    dil     |
|   rbp   |  ebp   |   bp   |     -      |    bpl     |
|   rsp   |  esp   |   sp   |     -      |    spl     |
| r8..r15 |  rXd   |  rXw   |     -      |    rXb     |

(r/e)ip is not considered general purpose.

FP/SSE registers:

- `xmm` registers: can contain a scalar (e.g. a single float) or packed data:
  - 4 * single precision
  - 2 * double precision
  - 16 * 8-bit
  - 8 * 16-bit
  - 4 * 32-bit
  - 2 * 64-bit
- `mxcsr`: control flags (lower 16 bits used)
  - 0..5, 7..12 : errors (e.g. overflow) and corresponding masks
  - 13, 14      : rounding behavior (nearest, down, up, truncate)
  - 6, 15       : other flags

AVX registers: 16 `ymm`, 256-bit

AVX-512 registers: 32 `zmm`, 512-bit

## Flags

|   Name    | Symbol |  Bit  | Content                                                                             |
| :-------: | :----: | :---: | ----------------------------------------------------------------------------------- |
|   Carry   |   CF   |   0   | Previous instruction had a carry                                                    |
|  Parity   |   PF   |   2   | LO byte (BYTE!!) has even number of 1s                                              |
|  Adjust   |   AF   |   4   | BCD operations                                                                      |
|   Zero    |   ZF   |   6   | Result = 0                                                                          |
|   Sign    |   SF   |   8   | Result < 0 (MSB = 1)                                                                |
| Direction |   DF   |  10   | Direction of string operations (increment or decrement)                             |
| Overflow  |   OF   |  11   | MSB changed because added nums with same sign or subtracted nums with opposite sign |

`OF`, `CF`, `SF` and `ZF` are called "condition flags".

## Control flow patterns

When reading if/then (and loops), generally, the conditional is the opposite:

```asm
; if (foo == bar); then baz; end
;
        cmp foo, bar
        jnz end
        baz
end:
```

Loops can be optimized by applying the conditional jump at the end (see [optimizations section](#control-flow-1))

If the loop variable has a last value of zero, use `jns` instead of `jnz`.

When a boolean expressions has multiple conditionals, each individual comparison can be accumulated in halves of a reg8, via setCC:

```asm
; bl = eax < y && ebx > z
        cmp eax, y
        setl bl
        cmp ebx, z
        setg bh
        and bl, bh
```

Switch/case implementation depends on the cases:

- contiguous ints: jump table (linear array of addresses)
  - can subtract indexes via scaled indexed addressing (e.g `rax * 8 - 5 * 8` if the first case is 5)
- noncontiguous ints (few cases): jump table, with default value entries
- noncontiguous ints (many cases): mix if/then and jump tables
- really hard cases: use binary search via if/then

Trampolines (64-bit jump):

```asm
; jmp [rip] (!)

        jmp destPtr
destPtr dq  ?

; store into stack -> ret

        push [destPtr]
        ret
```

## Instructions

`SP`/`DP` = Single/Double Precision

"A-ext" = (my term) AL->AX ... EAX->RAX
"A+ext" = (my term) AL->AX, AX->DX:AX ... RAX->RDX:RAX

`W`     = Width (`b`/`w`/`d`/`q`)

There is only one instruction that supports a 64-bit constant (`mov reg64, imm64`).

### Control flow

- `loop $label` : Decreases `rcx`
- `enter 0,0`   : Prologue (same as `push rbp` + `mov rbp, rsp`) (see [performance](#optimizations))
- `leave`       : Epilogue (same/performance as `mov rsp, rbp` + `pop rbp`)

#### Conditional jump flags/SetCC/CmovCC

For signed comparisons:

- `SF != OF` : left < right
- `SF == OF` : left >= right

for unsigned, use `CF`. The `ZF` is added to the boolean condition, in order to handle the equality.

Signed: `a`/`b`, unsigned: `g`/`l`.

- `jb`  / `jae` : `CF`             / `!CF`
- `jbe` / `ja`  : `CF || ZF`       / `!CF && !ZF`
- `jl`  / `jge` : `SF != OF`       / `SF == OF`
- `jle` / `jg`  : `SF != OF || ZF` / `SF == OF && !ZF`

- `setCC op8` : Set the (8-bit) operand to 1 if `CC` flag is set (omit the `F`, e.g. `setc`); DOES NOT reset the higher bits
              : There is one for each jCC

- `cmovCC`    : Conditional mov (see [optimization](#optimizations))

### Storage

- `mov`            : WATCH OUT! `reg64/32, src32` zero extend (even in the same reg); `reg16`/`reg8` don't.
- `push`/`pop`     : reg₁₆/₆₄, mem₁₆/₆₄; pop: const₆₄ (use `pushw` for const₁₆)
- `pushfq`/`popfq` : rflags (eflags without `q`)
- `xlat`           : executes `mov al, [rbx + al]`
- `rep`            : does not execute if `ecx` = 0

### String operations

WATCH OUT!! String operations respect the endianness.

- `repz`           : repeat while (`ZF` || `ecx` > 0)
- `repnz`          : repeat while (!`ZF` || `ecx` > 0)
- `scasW`          : compare (`rdi`); see below

WATCH OUT!! `repnz scasW` (find a string) exits the cycle _after_ updating `rdi` (so `rdi` is **after** the location of the string found).

### Sign-extension

- `cwde`/`cdqe`              : Sign-extend A-ext (16/32 bits)
- `cbw`/`cwd`/`cdq`/`cqo`    : Sign-extend A+ext (8..64 bits)
- `movsxd dst64, src32`      : Move with sign-extension, specific 32->64
- `movsx dst, src`           : ^^, all the rest
- `movzx dst, src`           : Move with zero-extension (WATCH OUT! See `mov`)

### Arithmetic

- `add`/`sub`, `adc`/`sbb`
- `inc`/`dec`                : WATCH OUT! They don't affect `CF`
- `sal`/`shl`                : Move the high bit into CF
- `sar`                      : Keeps the high bit as before shifting; on negative values, it's not equivalent to a division by 2!
- `shr`                      : Sets the high bit to 0
- `shrd/shld dest, fill, cnt` : Shifts dest bits of $cnt positions, and shifts `fill` bits (in the same direction) into the opened positions
- `rol`/`ror`/`rcl`/`rcr`  : All those instructions set the CF accordingly!
- `idiv`                     : Divide `rdx`:`rax`; result in `rax`, modulo in `rdx`
- `neg`                      : Two's complement negation

#### Multiplication

- `mul op`                  : unsigned; A+ext; no const
- `imul op`                 : ^^, signed
- `imul dstreg, op`         : `destreg *= op`; no 8 bit; 32bit consts are sign-extended; on overflow, `CF`+`OF` are set
- `imul dstreg, src, const` : `destreg = src * const`

WATCH OUT!! In the 2+ operand imul operations, the dstreg width is the same as the other operands!!!

#### Division

- `[i]div op`               : A+ext (low=quotient; high=modulo)

WATCH OUT!! The op is half the size of the source (A+), so must sign/zero-extend the source, if needed! Also, the result may not fit the dest!!

WATCH OUT!! Errors can be raised, e.g. division by zero

### Flags

- `clc`, `stc`, `cmc` : Clear/Set/Complement the Carry flag
- `sahf`, `lahf`      : (Store AH into/Load AH from) low 8 bit flags

### Comparison/Bit testing/manipulation:

In order to apply operations searching for set bits (e.g. bsf) to a clear bit, precede with NOT.

- `xor`, `or`, `and`, `not`: Always clear CF/OF (don't forget!!) and set the SF/ZF/PF accordingly.
- `cmp`                    : Performa `sub`, discards the result, and sets `SF`/`PF`/`ZF` accordingly
                           : WATCH OUT!! Don't forget that consts are max 32 bits!
- `test`                   : Performs `and`, discards the result, and sets `SF`/`PF`/`ZF` accordingly; can't be used to test
                             if multiple bits are set (any one set will unset ZF)
- `bts/btr/btc op, n`      : Bit `n` set/reset/complement
- `bt op, reg`             : Test `reg`ᵗʰ bit (doesn't support immediate); stores bit value into CF
  ```asm
  mov   rax, 61             ; test bit 61
  bt    $op, rax
  setc  al
  ```
- `bsf/bsr reg, op`       : Search the first set bit [f]orward (from LO)/[r]everse
- `bswap reg`             : Swaps the byte order

BM1 set instructions:

- `bextr reg, op, ctrl` : Extract bits from op into reg, regulated by ctrl (bits 0-7: starting pos; bits 8-15: length)
- `blsi reg, op`        : Extract lowest set bit; set zero if not found
- `blsr reg, op`        : Clear the lowest set bit, and copy result to destination
- `blsmsk reg, op`      : Sets all bits from LO to the lowest set bit (inclusive), and resets the others
- `blsi + dec`          : Like blsmsk, but not including the lowest set bit
- `tzcnt reg, op`       ; Count LO zeros up to first set bit (this equals the position of the first set bit)

BM2 set has instructions to insert/extract bit sets.

SSE 4.1:

- `popcnt reg, op`      : Count the set bits

### Floating-point/SSE

SSE doesn't support 80-bits FP.

SSE instructions are granular because the underlying implementation may be optimized for the semantics of the operation required (e.g. storing floats, as opposed to a simple `mov`, may trigger preparations for FP operations).

- `P` : `s`/`d` precision
- `N` : `P` + `i` (integer)
- `A` : `u`/`a` alignment

- `movss`, `movsd`     : Move SP/DP scalar to/from `xmm`
- `movd`, `movq`       : Move 32/64-bit to/from regular registers
- `ldmxcsr`, `stmxcsr` : Load/store `mxcsr` (32-bit)
- `cvtsN2sN`           : Convert between floats and integers

- `addsP`, `subsP`
- `mulsP`, `divsP`
- `minsP`, `maPsP`     : Min/max, store into dest
- `sqrtsP`             : Square root
- `rcpss`              : Reciprocal
- `rsqrtss`            : Reciprocal of square root

- `movApP`               : Move Un/Aligned packed S/DP
- `movdqa`               : Move Double int Aligned (4 sizes)
- `addpP`                : Add Packed S/DP
- `paddd`                : Packed Add Double int (4 sizes)
- `pextrd reg, xmm, pos` : Packed Extract Double int at (xmm) Pos into reg (4 sizes)
- `pinsrd xmm, reg, pos` : Packed Insert reg into (xmm) Pos Double int (4 sizes)

- `cmpsP xmm, op, imm8`  : imm8 is the comparison
- `cmpeqsP xmm, op`, ... : Special MASM/UASM format

- `andpd`, `orpd`, ...   : boolean operations (two ops; there are `V` versions, with three ops)
- `ptest`                : like `test`

### Other

- `cpuid`         : Read CPU characteristics; requires mode in `eax` -> sets the result in `rcx`/`rdx`
                    Infos here: https://exceptionshub.com/how-to-check-if-a-cpu-supports-the-sse3-instruction-set.html
- `rdtsc`         : Start timing; store timestamp into `edx`:`eax`; must precede with a serializing instruction
                    (e.g. `cpuid`), in order to prevent out-of-order execution (which would hamper the timing precision)
- `rdtscp`        : Stop timing; store timestamp into `edx`:`eax`; must be followed by a serializing instruction

## Optimizations

### Low-level

- `enter 0, 0`          : Slower than `push rbp` + `mov rbp, rsp`!
- `xor $reg, $reg`      : Fastest way to reset a register
- `dec rcx; jnz $label` : Faster than `loop $label`!!!
- `mov $reg, $imm64`    : Preferrable to `lea reg, $addr`, but only for this case (see https://stackoverflow.com/a/35475959)
- `cmovCC`              : CmovCC is not necessarily better than jCC - see [L. Torvald's considerations](https://yarchive.net/comp/linux/cmov.html)

Multiplications without imul:

- use `lea` + indexed scaled addressing (e.g. `ebx + 2 * ebx`)
- use shifts + additions

Divisions with idiv:

- use shifts
- multiply by the reciprocal
  ```asm
        ; DX = AX / n (!!!); works because of the bits shift - the bits "migrate" from AX into DX!!
        mov dx, (65536 / n)
        mul dx
  ```

Abs(x) without conditional (!):

```asm
        cdq             ; set EDX to -1 if EAX < 0, and to 0 otherwise
        xor eax, edx    ; NOT if EAX was < 0
        and edx, 1      ; 1 if EAX was < 0
        add eax, edx    ; +1 if EAX was < 0 (two's complement)

        mov edx, eax    ; even smaller!!!
        neg edx
        cmovns eax, edx
```

### Higher level

Cycle a 2^n counter via bitmask: `inc $op; and $op, 0x11..1`.

Lookup tables: must consider that memory access may be slow, and CPU computations fast! LT are optimal when small, so that they fit in the CPU cache.

#### Control flow

Loops can be optimized by applying the conditional jump at the end:

```asm
        jmp loop_test ; this is not needed if the loop runs at least once
loop_start:
        inc eax
loop_test:
        cmp eax, val
        jne loop_start
```

When implementing short-circuit boolean expressions:

- encode the shortest codepaths based on the distribution of the cases:
  ```c
  // if (a == b) often, and (c < d) rarely, first encode the (jnl) test for (c < d).
  (a == b) && (c < d)
  ```
- if some comparisons are expensive, evaluate them later than inexpensive ones.

If some code is rarely executed, deinline it, so that the hot path is denser.

## Syscalls

Linux x86-64 syscalls: http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64

- Syscall x64 symbols: `/usr/include/asm/unistd_64.h`.
- I/O symbols: `/usr/include/asm-generic/fcntl.h` (WATCH OUT: `0nn` is octal)

WATCH OUT! Syscalls are different between 32 and 64 bits.

In Windows, "syscalls" are internal; one uses the Windows API.

## Utilities

```sh
# Printf useful file info, e.g. the header includes the entry point.
#
readelf --file-header $file

# Notice `main`, `__data_start`, `__bss_start`
#
readelf --symbols $file | tail +10 | sort -k 2 -r

# Disassemble a file.
#
# - `-d`: disassemble
# - `-M`: syntax
#
objdump -M intel -d $file
```

## General concepts

### JIT and calls

See epic posts:

- https://stackoverflow.com/questions/54947302/handling-calls-to-potentially-far-away-ahead-of-time-compiled-functions-from-j
- https://stackoverflow.com/questions/19552158/call-an-absolute-pointer-in-x86-machine-code
