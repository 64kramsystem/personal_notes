# Assembly MASM/UASM

- [Assembly MASM/UASM](#assembly-masmuasm)
  - [Requirements](#requirements)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Data types](#data-types)
  - [Public interfacing](#public-interfacing)
  - [Memory layout](#memory-layout)

## Requirements

The reference is [assembly_nasm.md](assembly_nasm.md).

## Basic structure

```asm
myConst1 equ 2               ; constant; can be located _anywhere_
myconst  =   2               ; constant, other format

; sections may appear: in any order, and multiple times

         .const              ; constants that occupy memory

pi       real4 3.14159       ; must be initialized

         .data?              ; uninitialized objects
maybeVar db  ?

         .data

myVar    dw  0xCAFE
myVar2   dq  ?               ; uninitialized value
myStr    db  "a string", 0
myArr    db  10 dup (?)      ; can use a: constant, initialized value(s)

         align 4             ; align the following variable (1 to 16)
myByte   db  0

union1   label dword         ; labels don't occupy space themselves
union2   label dword         ; tee-hee unions
         dd  0

         .code

main     proc

         ret

main     endp

         end
```

### Makefile for compiling

(see [assembly_nasm.md](assembly_nasm.md#makefile-for-compiling))

```sh
# UASM
#
# -Zi3: debug info; there is no DWARF support; there are extra values for CodeView (Windows) info
# -Fl: generate list file (!! the option must precede `hello.asm` !!)
#
uasm -elf64 -Zi3 -Fl=hello.lst hello.asm
```

## Data types

|   Name   | Description                  |
| :------: | ---------------------------- |
| byte/db  | Unsigned 8-bit               |
| word/dw  | Unsigned 16-bit              |
| dword/dd | Unsigned 32-bit              |
| qword/dq | Unsigned 64-bit              |
|  real4   | Single-precision (32-bit) FP |
|  real8   | Double-precision (64-bit) FP |

Non-FP types have a signed version (e.g. `sbyte`), although they are purely for descriptive purposes.

Note: there are a few others types.

## Public interfacing

```asm
; The default is to uppercase function names; C/++ is case-sensitive, so in order to reference the name
; as is, we disable it.
; For currently unclear reasons, this was not necessary when testing (with UASM).
;
      option  casemap:none

      .code

; Access to C function
;
      externdef printf:proc

; Don't call exported functions `main`, since it would conflict with the C/++ entry function.
;
      public asmFunc
asmFunc  PROC

      ret

asmFunc  ENDP
```

## Memory layout

Low to high:

- O/S reserved
- stack
- heap
- code
- readonly
- variables
