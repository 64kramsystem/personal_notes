# Assembly MASM/UASM

- [Assembly MASM/UASM](#assembly-masmuasm)
  - [Requirements](#requirements)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Data types](#data-types)
    - [Unions](#unions)
    - [Structs](#structs)
  - [Addressing](#addressing)
  - [Constant operators](#constant-operators)
  - [Public interfacing](#public-interfacing)
  - [Memory layout](#memory-layout)

## Requirements

The reference is [assembly_nasm.md](assembly_nasm.md).

## Basic structure

```asm
; constants

myConst1 equ 2                ; preferrably numeric; can be located _anywhere_
myConst2 =   2                ; necessarily numeric
myConst3 textequ <"my_str">   ; textual; this is like a `#define` C macro (WATCH OUT!!)
myConst4 = 1FFFh shl myConst1 ; expressions can be used!

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
union2   label dword         ; poor man's union (see section for MASM support)
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

### Unions

```asm
; Use `endu textequ <ends>` to make the end consistent with MASM general end syntax.
;
myUnion union
a       byte
b       word
myUnion ends
```

### Structs

```asm
; `4` is the alignment - each field will be aligned as `min(alignment, field_size)`
; `align` can also be used inside the struct, but it's inconvenient.
student  struct 4
Id       dword
FullName byte 64 dup (?)

Grades   grades {}        ; nested struct!

         union            ; anonymous union!
a        byte
b        word
         ends
student  ends

studentsArray student 4 dup ({}) ; array of students

; Declaration ((trailing) fields can be omitted)
; WATCH OUT!!! `Parent` can't be used as instance name!!
John student {1, "abc", {1}}

; Allocated and access an array
mov rcx, sizeof student
call malloc
mov [rax].student.Final, 100

; Access a nested struct
mov bx, John.Grades.Homework

; Access an element in an array of structs (no LARGEADDRESS)
imul ebx, i, sizeOf student
mov eax, studentsArray.Id[rbx]
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
