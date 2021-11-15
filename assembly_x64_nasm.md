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
  - [Utilities](#utilities)

## Basic structure

```asm
; Data (optional): this section goes into the executable
;
section .data
    msg     db      "Hello world!", 0x0A, 0 ; any defined bytes sequence is called a "string"
    msgLen  equ     $ - msg - 1             ; constant (only for ints); `$` is the current address
                                            ; WATCH OUT! Don't forget `-1` (terminator) when printing a string
    %define         pi 3.14                 ; macro; can use to define non-int pseudo-constants (but they
                                            ; have slightly different semantics)

; Reserved space (optional): not in the executable; initialized at runtime with 0s
; "Block Started by Symbol"
;
section .bss
    myres  resw    5    ; 5 words

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

- `MOV`       : WATCH OUT! `mov eax, $val` clears the `rax` upper bits (mov `al`/`ax` doesn't)
- `MOVSD`     : DP-float to/from `xmm`

Arithmetic:

- `ADD`/`SUB`, `INC`/`DEC`
- `SAL`/`SHL` : Move the high bit into CF
- `SAR`       : Keeps the high bit as before shifting
- `SHR`       : Sets the high bit to 0
- `IMUL`      : Multiply; result in `rdx`:`rax`
- `IDIV`      : Divide `rdx`:`rax`; result in `rax`, modulo in `rdx`

Floating-point:

- `ADDSD`, `SUBSD`: Add/sub DP in `xmm`
- `MULSD`, `DIVSD`: Multiply/divide DP in `xmm`
- `SQRTSD`        : Square root DP in `xmm`

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
  - push `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9` (parameter regs; "caller saved")
  - set parameter regs
  - push additional params (in reverse order)
  - callee:
    - allocate local vars on the stack
    - push `rbx`, `rbp`, `r12` .. `r15` ("callee saved")
    - execute; retval: `rax`
    - pop callee saved regs
    - deallocate local vars
  - pop additional params
  - pop caller saved regs

Notes:

- don't forget that `call`/`ret` push/pop `rip`
- `r10`/`r11` are not required to be saved

## Syscalls

Linux x86-64 syscalls: http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64

WATCH OUT! Syscalls are different between 32 and 64 bits.

## Utilities

```sh
# Printf useful file info, e.g. the header includes the entry point.
#
readelf --file-header $file

# Notice `main`, `__data_start`, `__bss_start`
readelf --symbols $file | tail +10 | sort -k 2 -r
```
