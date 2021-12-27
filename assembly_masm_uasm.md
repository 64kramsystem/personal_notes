# Assembly MASM/UASM

- [Assembly MASM/UASM](#assembly-masmuasm)
  - [Requirements](#requirements)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Data types](#data-types)
  - [Addressing](#addressing)
  - [Constant operators](#constant-operators)
  - [Public interfacing](#public-interfacing)
  - [Memory layout](#memory-layout)

## Requirements

The reference is [assembly_nasm.md](assembly_nasm.md).

## Basic structure

```asm
; constants

myConst1 equ 2               ; preferrably numeric; can be located _anywhere_
myconst  =   2               ; necessarily numeric
myConst3 textequ <"my_str">  ; textual; this is like a `#define` C macro (WATCH OUT!!)

; pseudo-enum

variant1  = 0
variant2  = variant1 + 1

; sections may appear: in any order, and multiple times

         .const              ; constants that occupy memory

pi       real4 3.14159       ; must be initialized; can also be in the code section!

         .data?              ; uninitialized objects

maybeVar db  ?               ; Win initializes it 0, but it's good practice not to rely on it

         .data

myVar    dw  0xCAFE
myVar2   dq  ?               ; uninitialized value
myStr    db  "a string", 0
myArr    db  10 dup (?)      ; array, implicit form (duplicates the value); can use a: constant, initialized value(s); 
                             ; ^^ can be nested! (`10 dup (10 dup ?)`)
myArr2   db  1, 2, 3         ; array, explicit form

         align 4             ; align the following variable (1 to 16)
myByte   db  0

union1   label dword         ; labels don't occupy space themselves
union2   label dword         ; tee-hee unions
         dd  0
eou      equ this byte       ; like $, but can specify a type

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

|    Name     | Description                  |
| :---------: | ---------------------------- |
|   byte/db   | Unsigned 8-bit               |
|   word/dw   | Unsigned 16-bit              |
|  dword/dd   | Unsigned 32-bit              |
|  qword/dq   | Unsigned 64-bit              |
|    real4    | Single-precision (32-bit) FP |
|    real8    | Double-precision (64-bit) FP |
| x/y/zmmword | SSE regs size!!              |

Non-FP types have a signed version (e.g. `sbyte`), although they are purely for descriptive purposes.

Note: there are a few others types.

Typedef: `$newtype typedef $existing_type`.

Type coercion (pointed type): `$type ptr $expr`, e.g. `xmmword ptr [rbx]`.

Structs:

```asm
student  struct
Id       dword
FullName byte 64 dup (?)
Final    byte
Grades   grades {}        ; nested struct!
student  ends

John student {} ; empty declaration

; Allocated and access an array
mov rcx, sizeof student
call malloc
mov [rax].student.Final, 100

; Access a nested struct
mov bx, John.Grades.Homework
```

## Addressing

`x[y]` is the same as `[x][y]`, which is the same as `x + y`.

## Constant operators

There are many, among which:

- `mod`
- `EQ`, `LT`, `GE`...
- `AND`...
- `SIZEOF`

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

- O/S reserved (also traps the NULL ptr!)
- stack
- heap
- code
- readonly
- variables

Complex programs may add new and/or combine sections.

WATCH OUT!! It's not guaranteed that the data in sections that are defined as contiguous, will be contiguous in memory!
