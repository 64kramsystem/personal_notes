# Assembly x64

- [Assembly x64](#assembly-x64)
  - [Basic structure](#basic-structure)
  - [Data types](#data-types)
  - [Stack alignment/frame](#stack-alignmentframe)
  - [Addressing](#addressing)
    - [Pages (MMU)](#pages-mmu)
  - [Registers/Flags](#registersflags)
  - [Instructions](#instructions)
  - [Optimizations](#optimizations)
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

## Stack alignment/frame

Stack alignment is required by some instructions; the stack frame is useful primarily for debuggers.

When there is complete control over a workflow (e.g. leaf routine), the two can be omitted.

If one doesn't know the alignment state, he can use:

```asm
; irreversible
;
and rsp, 0xfffffffffffffff0

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

## Registers/Flags

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

Flags:

|   Name    | Symbol |  Bit  | Content                                                                             |
| :-------: | :----: | :---: | ----------------------------------------------------------------------------------- |
|   Carry   |   CF   |   0   | Previous instruction had a carry                                                    |
|  Parity   |   PF   |   2   | Last byte has even number of 1s                                                     |
|  Adjust   |   AF   |   4   | BCD operations                                                                      |
|   Zero    |   ZF   |   6   | Result = 0                                                                          |
|   Sign    |   SF   |   8   | Result < 0 (HO bit = 1)                                                             |
| Direction |   DF   |  10   | Direction of string operations (increment or decrement)                             |
| Overflow  |   OF   |  11   | MSB changed because added nums with same sign or subtracted nums with opposite sign |

`OF`, `CF`, `SF` and `ZF` are called "condition flags".

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

## Instructions

`SP`/`DP` = Single/Double Precision

There is only one instruction that supports a 64-bit constant (`mov reg64, imm64`).

Control flow:

- `loop $label` : Decreases `rcx`
- `enter 0,0`   : Prologue (same as `push rbp` + `mov rbp, rsp`) (see [performance](#optimizations))
- `leave`       : Epilogue (same/performance as `mov rsp, rbp` + `pop rbp`)

Conditional jump flags:

- `jb`  / `jae` : `CF`             / `!CF`
- `jbe` / `ja`  : `CF || ZF`       / `!CF && !ZF`
- `jl`  / `jge` : `SF != OF`       / `SF == OF`
- `jle` / `jg`  : `SF != OF || ZF` / `SF == OF && !ZF`

Storage:

- `mov`            : WATCH OUT! `reg64/32, src32` zero extend; `reg16`/`reg8` don't.
- `movzx reg, op`  : move with zero extend
- `movsd`          : DP-float to/from `xmm`
- `push`/`pop`     : reg₁₆/₆₄, mem₁₆/₆₄; pop: const₆₄ (use `pushw` for const₁₆)
- `pushfq`/`popfq` : rflags (eflags without `q`)

Arithmetic:

- `inc`/`dec`                : WATCH OUT! They don't affect `CF`
- `sal`/`shl`                : Move the high bit into CF
- `sar`                      : Keeps the high bit as before shifting
- `shr`                      : Sets the high bit to 0
- `imul destreg, op`         : Multiply; `destreg *= op`; no 8 bit; 32bit consts are sign-extended; on overflow, `CF`+`OF` are set
- `imul destreg, src, const` : ^^;       `destreg = src * const`
- `idiv`                     : Divide `rdx`:`rax`; result in `rax`, modulo in `rdx`
- `neg`                      : Two's complement negation

Flags:

- `clc`, `stc`, `cmc` : Clear/Set/Complement the Carry flag
- `sahf`, `lahf`      : (Store AH into/Load AH from) low 8 bit flags

Bit testing/manipulation:

- `test`          : Performs `and`, discards the result, and sets `SF`/`PF`/`ZF` accordingly; can't be used to test
                    if multiple bits are set (any one set will unset ZF)
- `setCC`         : Set the (8-bit) operand to 1 if `CC` flag is set (omit the `F`, e.g. `setc`); DOES NOT reset the higher bits
- `bts/btr op, n` : Bit `n` set/reset
- `bt op, reg`    : Test `reg`ᵗʰ bit (doesn't support immediate); stores bit value into CF
  ```asm
  mov   rax, 61             ; test bit 61
  bt    $op, rax
  setc  al
  ```

Floating-point:

- `movss`              : Move scalar SP
- `cvtss2sd`           : Convert scalar SP to scalar DP
- `addsd`, `subsd`     : Add/sub DP in `xmm`
- `mulsd`, `divsd`     : Multiply/divide DP in `xmm`
- `sqrtsd`             : Square root DP in `xmm`
- `ldmxcsr`, `stmxcsr` : Load/store `mxcsr` (32-bit)

SSE:

- `mov[ua]p[sd]`         : Move Un/Aligned packed S/DP
- `movdqa`               : Move Double int Aligned (4 sizes)
- `addp[sd]`             : Add Packed S/DP
- `paddd`                : Packed Add Double int (4 sizes)
- `pextrd reg, xmm, pos` : Packed Extract Double int at (xmm) Pos into reg (4 sizes)
- `pinsrd xmm, reg, pos` : Packed Insert reg into (xmm) Pos Double int (4 sizes)

Other:

- `cpuid`         : Read CPU characteristics; requires mode in `eax` -> sets the result in `rcx`/`rdx`
                    Infos here: https://exceptionshub.com/how-to-check-if-a-cpu-supports-the-sse3-instruction-set.html
- `rdtsc`         : Start timing; store timestamp into `edx`:`eax`; must precede with a serializing instruction
                    (e.g. `cpuid`), in order to prevent out-of-order execution (which would hamper the timing precision)
- `rdtscp`        : Stop timing; store timestamp into `edx`:`eax`; must be followed by a serializing instruction
- `bswap`         : Swaps the byte order of a reg

Trivial:

- `add`/`sub`
- `inc`/`dec`
- `rol`/`ror`
- `xor`, `or`, `and`, `not`

## Optimizations

- `enter 0, 0`          : Slower than `push rbp` + `mov rbp, rsp`!
- `xor $reg, $reg`      : Fastest way to reset a register
- `dec rcx; jnz $label` : Faster than `loop $label`!!!
- `mov $reg, $imm64`    : Preferrable to `lea reg, $addr`, but only for this case (see https://stackoverflow.com/a/35475959)

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
