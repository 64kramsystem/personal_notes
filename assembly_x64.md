# Assembly x64

- [Assembly x64](#assembly-x64)
  - [Basic structure](#basic-structure)
  - [Makefile for compiling](#makefile-for-compiling)
  - [Data types](#data-types)
  - [Registers/Flags](#registersflags)
  - [References](#references)

## Basic structure

```asm
; Data; this section goes into the executable
;
section .data
    msg    db      "hello, world", 0  ; any defined bytes sequence is called a "string"
    pi     equ     3.1415

; Reserved space; initialized at runtime with 0s.
; "Block Started by Symbol"
;
section .bss
    myres  resw    5    ; 5 words

; Program code
;
section .text
    global main
main:
    mov    rax, 60      ; exit
    mov     rdi, 0      ; exit code
    syscall             ; Watch out! Different between 32 and 64 bits
```

## Makefile for compiling

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

## Data types

- `db`, `dw`, `dd`, `dq`
- `resb`, `resw`, `resres`, `resq`

## Registers/Flags

General purpose registers:

| 64-bit | 32-bit | 16-bit | low  8-bit | high 8-bit | note      |
| :----: | :----: | :----: | :--------: | :--------: | --------- |
|  rax   |  eax   |   ax   |     al     |     ah     |           |
|  rbx   |  ebx   |   bx   |     bl     |     bh     |           |
|  rcx   |  ecx   |   cx   |     cl     |     ch     |           |
|  rdx   |  edx   |   dx   |     dl     |     dh     |           |
|  rsi   |  esi   |   si   |    sil     |     -      |           |
|  rdi   |  edi   |   di   |    dil     |     -      |           |
|  rbp   |  ebp   |   bp   |    bpl     |     -      |           |
|  rsp   |  esp   |   sp   |    spl     |     -      |           |
|   r8   |  r8d   |  r8w   |    r8b     |     -      | Up to r15 |

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

## References

- Linux x86-64 syscalls: http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64
