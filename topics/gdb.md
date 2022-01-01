# GDB

- [GDB](#gdb)
  - [Setup and run](#setup-and-run)
  - [Debugging](#debugging)
  - [Inspect](#inspect)

## Setup and run

```
cat > ~/.gdbinit << CFG
set disassembly-flavor intel
CFG

gdb $filename
```

## Debugging

Breakpoint (`break`):

```
(gdb) b main
Breakpoint 1 at 0x401110: file hello.asm, line 18.
```

Run (`run`):

```
(gdb) r
Starting program: /path/to/hello

Breakpoint 1, main () at hello.asm:18
18	    mov rbp, rsp; for correct debugging
```

Step (`s`tep):

```
(gdb) s
20	    push           rbp
```

## Inspect

List source (`list`):

```
(gdb) l
1	; Data: this section goes into the executable
2	;
3	section .data
4	    msg     db      "Hello world!", 0x0A, 0 ; any defined bytes sequence is called a "string"
# ...
```

Disassemble (`disassemble`); `main` = frame

```
(gdb) `disas` main
Dump of assembler code for function main:
   0x0000000000401110 <+0>:	push   rbp
   0x0000000000401111 <+1>:	mov    rbp,rsp
   0x0000000000401114 <+4>:	mov    eax,0x0
# ...
```

Print values (e`x`amine).  
Format = `x/[<number>]<type>`; Types: `s`tring; `c`har; he`x`; `d`ecimal.

```
(gdb) x/s 0x404028
0x404028 <msg>:	"Hello world!\n"
(gdb) x/13c 0x404028
0x404028 <msg>: 72 'H'  101 'e' 108 'l' 108 'l' 111 'o' 32 ' ' 119 'w' 111 'o'
0x404030:       114 'r' 108 'l' 100 'd' 33 '!'  10 '\n'
```

Print a variable:

```
(gdb) x/s &msg
0x404028 <msg>:	"Hello world!\n"
```

Print registers (`i`nfo `r`egisters):

```
(gdb) i r
rax            0x401110            4198672
rbx            0x401150            4198736
rcx            0x401150            4198736
rdx            0x7fffffffce88      140737488342664
# ...
```
