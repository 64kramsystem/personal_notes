# Assembly x64 (NASM)

- [Assembly x64 (NASM)](#assembly-x64-nasm)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Data types](#data-types)
  - [Functions](#functions)
  - [Registers/Flags](#registersflags)
  - [C Calling convention (System V ABI)](#c-calling-convention-system-v-abi)
  - [Instructions](#instructions)
  - [Syscalls](#syscalls)

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
; Function prologue; can use the `enter 0,0` instruction, but it's slower.
    push           rbp
    mov            rbp, rsp

; System call
    mov            rax, 1         ; 1 = write
    mov            rdi, 1         ; 1 = to stdout
    mov            rsi, msg
    mov            rdx, msgLen
    syscall

; Function epilogue; can use the `leave` instruction.
    mov            rsp, rbp
    pop            rbp

; Exit
    mov            rax, 60     ; exit
    mov            rdi, 0      ; exit code
    syscall
```

### Makefile for compiling

```makefile
# - no-pie: makes debugging easier ("Position-Independent Executable")
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
run: hello
      ./hello
```

## Data types

- `db`, `dw`, `dd`, `dq`
- `resb`, `resw`, `resres`, `resq`

## Functions

Private function; put it inside the `text` section, before `main`.

```asm
area:
section .data
    .pi  dq  3.141   ; local to area
section .text
    enter 0,0
    leave
    ret
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

## Instructions

Arithmetic:

- `SAL`/`SHL` : Move the high bit into CF
- `SAR`       : Keeps the high bit as before shifting
- `SHR`       : Sets the high bit to 0

## Syscalls

Linux x86-64 syscalls: http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64

WATCH OUT! Syscalls are different between 32 and 64 bits.
