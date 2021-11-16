# Assembly x64 (NASM)

- [Assembly x64 (NASM)](#assembly-x64-nasm)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Data types](#data-types)
  - [Memory layout](#memory-layout)
  - [Stack alignment/frame](#stack-alignmentframe)
  - [Addressing](#addressing)
  - [Registers/Flags](#registersflags)
  - [Instructions](#instructions)
  - [Optimizations](#optimizations)
  - [Functions/Access levels](#functionsaccess-levels)
  - [C Calling convention (System V ABI)](#c-calling-convention-system-v-abi)
  - [Syscalls](#syscalls)
  - [NASM Preprocessor directives/Macros](#nasm-preprocessor-directivesmacros)
  - [C Integration (inline assembly)](#c-integration-inline-assembly)
  - [Utilities](#utilities)

## Basic structure

```asm
; Data (optional): this section goes into the executable
;
section .data
    msg             db "Hello world!", 0x0A, 0 ; any defined bytes sequence is called a "string"
    msgLen          equ $ - msg - 1            ; constant (only for ints); `$` is the current address
                                               ; WATCH OUT! Don't forget `-1` (terminator) when printing
                                               ; a string
    myArr           times 5 dw 0               ; array of 5 words with value=0
    %define pi      3.14                       ; define non-int pseudo-constants via macro (but they
                                               ; have slightly slightly different semantics)

; Reserved space (optional): not in the executable; initialized at runtime with 0s
; "Block Started by Symbol"
;
section .bss
    myres          resw 5                      ; 5 words
    msgFullLen     msgLen + 1                  ; constants can be used in declarations, anywhere a number
                                               ; is used

; Program code
;
section .text
    global main

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

; Exit
    mov            rax, 60     ; exit
    mov            rdi, 0      ; exit code
    syscall
```

### Makefile for compiling

```makefile
# - no-pie: makes debugging easier ("Position-Independent Executable"), and allows external functions.
#
hello: hello.o
      gcc -o hello hello.o -no-pie

# -f: output format
# -g: add debug info
# -F: debug info type
# -l: generate list file
#
hello.o: hello.asm
      nasm -f elf64 -g -F dwarf hello.asm -l hello.lst
```

Can add for convenience:

```
debug: hello
      lldb -o r ./hello

run: hello
      ./hello
```

## Data types

- `db`, `dw`, `dd`, `dq`
- `resb`, `resw`, `resres`, `resq`

## Memory layout

Low to high:

- text (main)
- data
- bss
- heap
- stack
- env vars, cmdline args

## Stack alignment/frame

Stack alignment is required by some instructions; the stack frame is useful primarily for debuggers.

When there is complete control over a workflow (e.g. leaf routine), the two can be omitted.

If one doesn't know the alignment state, he can use `AND rsp, 0xfffffffffffffff0`.

## Addressing

```asm
mov rsi, radius      ; WATCH OUT!! Stores *the address*
mov rsi, [radius]    ; Dereferences an address (stores *the value*)
lea rsi, [radius]    ; Stores the address
```

## Registers/Flags

General purpose registers:

| 64-bit  | 32-bit | 16-bit | low  8-bit | high 8-bit |
| :-----: | :----: | :----: | :--------: | :--------: |
|   rax   |  eax   |   ax   |     al     |     ah     |
|   rbx   |  ebx   |   bx   |     bl     |     bh     |
|   rcx   |  ecx   |   cx   |     cl     |     ch     |
|   rdx   |  edx   |   dx   |     dl     |     dh     |
|   rsi   |  esi   |   si   |    sil     |     -      |
|   rdi   |  edi   |   di   |    dil     |     -      |
|   rbp   |  ebp   |   bp   |    bpl     |     -      |
|   rsp   |  esp   |   sp   |    spl     |     -      |
| r8..r15 |  rXd   |  rXw   |    rXb     |     -      |

(r/e)ip is not considered general purpose.

Flags:

|   Name    | Symbol |  Bit  | Content                                                          |
| :-------: | :----: | :---: | ---------------------------------------------------------------- |
|   Carry   |   CF   |   0   | Previous instruction had a carry                                 |
|  Parity   |   PF   |   2   | Last byte has even number of 1s                                  |
|  Adjust   |   AF   |   4   | BCD operations                                                   |
|   Zero    |   ZF   |   6   | Previous instruction resulted a zero                             |
|   Sign    |   SF   |   8   | Previous instruction resulted in most significant bit equal to 1 |
| Direction |   DF   |  10   | Direction of string operations (increment or decrement)          |
| Overflow  |   OF   |  11   | Previous instruction resulted in overflow                        |

## Instructions

`DP` = Double Precision

Control flow:

- `loop $label` : Decreases `rcx`
- `enter 0,0`   : Prologue (same as `push rbp` + `mov rbp, rsp`) (see [performance](#optimizations))
- `leave`       : Epilogue (same/performance as `mov rsp, rbp` + `pop rbp`)

Storage:

- `mov`       : WATCH OUT! `mov eax, $val` clears the `rax` upper bits (mov `al`/`ax` doesn't)
- `movsd`     : DP-float to/from `xmm`

Arithmetic:

- `sal`/`shl` : Move the high bit into CF
- `sar`       : Keeps the high bit as before shifting
- `shr`       : Sets the high bit to 0
- `imul`      : Multiply; result in `rdx`:`rax`
- `idiv`      : Divide `rdx`:`rax`; result in `rax`, modulo in `rdx`

Flags:

- `clc`, `stc`    : Clear/Set Carry flag

Bit manipulation:

- `setCC`         : Set operator to 1 if `CC` flag is set (omit the `F`, e.g. `setc`)
- `bts/btr op, n` : Bit `n` set/reset
- `bt op, reg`    : Test `reg`ᵗʰ bit (doesn't support immediate); stores byte 0/1 into operand
  ```asm
  mov   rax, 61             ; test bit 61
  xor   rdi, rdi            ; optional if only dil is used
  bt    [bitflags], rax
  setc  dil
  ```

Floating-point:

- `addsd`, `subsd`: Add/sub DP in `xmm`
- `mulsd`, `divsd`: Multiply/divide DP in `xmm`
- `sqrtsd`        : Square root DP in `xmm`

Trivial:

- `add`/`sub`
- `inc`/`dec`
- `rol`/`ror`
- `xor`, `or`, `and`, `not`

## Optimizations

- `enter 0,0`           : Slower than `push rbp` + `mov rbp, rsp`!
- `xor $reg, $reg`      : Fastest way to reset a register
- `dec rcx; jnz $label` : Faster than `loop $label`!!!

## Functions/Access levels

```asm
; External function.
; WATCH OUT! Require PIE to be disabled.
;
extern printf

; By making a function global, it can be also accessed from C, via `extern`.
;
global myvar         ; global variable
global myfunc        ; global function

; Function-private section; place it as any other function.
;
area:
section .data
    .pi  dq  3.141   ; local to area
section .text
    ; ... use .pi ...
    ret
```

## C Calling convention (System V ABI)

Pushes are required only if the regs are used in that scope.

- caller:
  - push `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`, `xmm0-7` (parameter regs; "caller saved")
  - set parameter regs
  - push additional params (in reverse order)
  - callee:
    - allocate local vars on the stack
    - push `rbx`, `rbp`, `r12` .. `r15` ("callee saved")
    - execute; retval: `rax`/`xmm0-1`
    - pop callee saved regs
    - deallocate local vars
  - pop additional params
  - pop caller saved regs

Notes:

- don't forget that `call`/`ret` push/pop `rip`
- `r10`/`r11`, `xmm8-15` are not required to be saved

## Syscalls

Linux x86-64 syscalls: http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64

- Syscall x64 symbols: `/usr/include/asm/unistd_64.h`.
- I/O symbols: `/usr/include/asm-generic/fcntl.h` (WATCH OUT: `0nn` is octal)

WATCH OUT! Syscalls are different between 32 and 64 bits.

## NASM Preprocessor directives/Macros

Conditional assembly:

```asm
; if O_READ != 0
;
%IF O_READ
; conditional block
%ENDIF
```

Macros (define before the data section):

```asm
; single line macro
;
%define double_it(r)    sal r, 1

; multiline macro with 2 arguments
;
%macro prntf 2
section .data
    %%arg1    db    %1, 0           ; 1ˢᵗ macro argument
    %%fmtint  db    "%s %ld", 10, 0 ; 2ⁿᵈ macro argument
section .text
    mov   rdi, %%fmtint
    mov   rsi, %%arg1
    mov   rdx, [%2]       ; second argument
    mov   rax,0
    call  printf
%endmacro
```

## C Integration (inline assembly)

"Extended" inline:

```c
// inline.c extract

int x = 11, y = 12, product; // can be globals or locals

// The metadata is optional, including the colons, but in some cases are required, e.g. `:::"rbx"`.
//
__asm__(
  ".intel_syntax noprefix;"
  "mov rbx, rdx;"
  "imul rbx, rcx;"
  "mov rax, rbx;"
  :"=a"(product)   # output operands (rax)
  :"d"(x), "c"(y)  # input operands (rdx, rcx)
  :"rbx"           # clobbered regs; they're saved/restored by the compiler
);

printf("Product: %d\n", product);
```

Operand register constraints:

- `a` => rax, eax, ax, al
- `b` => rbx, ebx, bx, bl
- `c` => rcx, ecx, cx, cl
- `d` => rdx, edx, dx, dl
- `S` => rsi, esi, si
- `D` => rdi, edi, di
- `r` => any register

There is a basic inline, but it's more restricted.

Required Makefile:

```makefile
# c files can be added to the gcc command, along the object files.
# `-masm=intel` is required to use inline assembly with intel dialect.
#
hello: hello.o
      gcc -o hello inline.c -masm=intel -no-pie
```

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