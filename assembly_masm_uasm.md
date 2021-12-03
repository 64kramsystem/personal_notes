# Assembly MASM/UASM

- [Assembly MASM/UASM](#assembly-masmuasm)
  - [Requirements](#requirements)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Public interfacing](#public-interfacing)

## Requirements

The reference is [assembly_nasm.md](assembly_nasm.md).

## Basic structure

```asm
      .code

main  PROC

      ret

main  ENDP

      END
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

## Public interfacing

```asm
; The default is to uppercase function names; C/++ is case-sensitive, so in order to reference the name
; as is, we disable it.
; For currently unclear reasons, this was not necessary when testing (with UASM).
;
      option  casemap:none

; Don't call exported functions `main`, since it would conflict with the C/++ entry function.
;
      public asmFunc
asmFunc  PROC

      ret

asmFunc  ENDP
```
