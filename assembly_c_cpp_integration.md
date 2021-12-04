# Assembly C/C++ integration

- [Assembly C/C++ integration](#assembly-cc-integration)
  - [Requirements](#requirements)
  - [C Calling convention (System V ABI)](#c-calling-convention-system-v-abi)
  - [Access ASM functions from C](#access-asm-functions-from-c)
  - [C Inline assembly](#c-inline-assembly)

## Requirements

Specialized code is in the specific assembler notes.

## C Calling convention (System V ABI)

Pushes are required only if the regs are used in that scope.

- caller:
  - push `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`, `xmm0-7` (parameter regs; "caller saved")
  - set parameter regs
  - push additional params (in reverse order)
  - callee:
    - allocate local vars on the stack
    - push `rbx`, `rbp`, `r12-15` ("callee saved")
    - execute; retval: `rax`, `xmm0`, `xmm1`
    - pop callee saved regs
    - deallocate local vars
  - pop additional params
  - pop caller saved regs

Notes:

- don't forget that `call`/`ret` push/pop `rip`
- `r10`, `r11`, `xmm8-15` are not required to be saved

## Access ASM functions from C

See specific assembler notes for public interfacing.

```cpp
extern "C" {
    void asmFunc(void); // example signature
};
```

## C Inline assembly

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
