# Assembly NASM

- [Assembly NASM](#assembly-nasm)
  - [Basic structure](#basic-structure)
    - [Makefile for compiling](#makefile-for-compiling)
  - [Addressing syntax](#addressing-syntax)
  - [Functions/Public interfacing](#functionspublic-interfacing)
  - [Preprocessor directives/Macros](#preprocessor-directivesmacros)
  - [Memory layout](#memory-layout)

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
align 16                                       ; align section!
    %define pi      3.14                       ; define non-int pseudo-constants via macro (but they
                                               ; have slightly slightly different semantics)

; Reserved space (optional): not in the executable; initialized at runtime with 0s
; "Block Started by Symbol"
;
section .bss
alignb 16                                      ; BSS align uses a different keyword
    myres          resw 5                      ; 5 words
    msgFullLen     msgLen + 1                  ; constants can be used in declarations, anywhere a number
                                               ; is used

; Program code
;
section .text
    global main

main:
    ; blah
```

### Makefile for compiling

```makefile
# - no-pie: makes debugging easier ("Position-Independent Executable"), and allows external functions.
#
hello: hello.o
      gcc -o hello hello.o -no-pie

hello.o: hello.asm
      # NASM
      #
      # -f: output format
      # -g: add debug info
      # -F: debug info type
      # -l: generate list file
      #
      nasm -f elf64 -g -F dwarf hello.asm -l hello.lst

      # UASM
      #
      # -Zi3: debug info; there is no DWARF support; there are extra values for CodeView (Windows) info
      # -Fl: generate list file (!! the option must precede `hello.asm` !!)
      #
	    uasm -elf64 -Zi3 -Fl=hello.lst hello.asm
```

Can add for convenience:

```
debug: hello
      lldb -o r ./hello

run: hello
      ./hello

clean:
      rm -f hello hello.o hello.lst
```

## Addressing syntax

```asm
push qword [radius]
```

## Functions/Public interfacing

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

## Preprocessor directives/Macros

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

## Memory layout

Low to high:

- text (main)
- data
- bss
- heap..stack
- env vars, cmdline args
