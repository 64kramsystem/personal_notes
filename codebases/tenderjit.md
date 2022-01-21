# TenderJIT

- [TenderJIT](#tenderjit)
  - [Stream 21/Jan](#stream-21jan)
  - [TJ design](#tj-design)
  - [Useful patterns](#useful-patterns)
  - [Debugging](#debugging)
  - [Ruby concepts](#ruby-concepts)

## Stream 21/Jan

The `EC` (Execution Context) points ("Control Frame Pointer", `cfp`) to currently executed Stack Frame (top of the Stack Frames stack)

Stack Frame: also called "Control Frame" (`rb_control_frame_t`)

The Call Frame Stack includes `main` on start.

The `PC` always points to the instruction (in the stack) that what we're going to execute next.

"Value Stack":

- shared across the stack frames
- where instructions put the values
- the EP points to it
- the SP refers to the next position

On method call (see `getlocal` instruction):

- the method is pushed on top of the Stack Frames, and the `cfp` is updated
- the callee is pushed (self)
- the arguments are pushed
- 3 "magic values" are pushed on the value stack
- the EP points at magic 3
- the SP points to after magic 3

On exit method call:

- magic values are removed
- pointers are rewound


## TJ design

The are two compilers:

- instant (`handle_*`)
- deferred: executes when a method is invoked, since it needs to figure out the receiving object.

When a request is made to compile an instruction sequence (iseq), the compiler checks to see if there is already an `ISEQCompiler` object associated with the iseq. If not, it allocates one, then calls compile on the object. The compiler will compile as many instructions in a row as it can, then will quit compilation. Depending on the instructions that were compiled, it may resume later on.

Unimplemented instructions don't have a `handle` method; the compiler generates an "exit" and the machine code will pass control back to YARV

## Useful patterns

```rb
klass_addr = rb.rb_class_of(recv) # returns the pointer
Fiddle.dlunwrap(klass_addr)       # find the class; in general, `dlunwrap` returns the object at the given address
Fiddle.dlwrap(ruby_value)         # convert to temp stack value
```

## Debugging

Command: `ruby -d`

- The address on the right  is of the method to executed instruction belongs to
- `NEW ISEQ Compiler` -> means recursively compiled new method

The Ruby lldb helper adds `rp` (ruby print):

```
$ rp ((VALUE*)$r15)[0]
bits: [    ]
T_ARRAY: len=4 (shared) shared=0
(const VALUE*) $2=0x600000244240 {
  (const VALUE) [0] = 0x3
  (const VALUE) [1] = 0x5
}
```

`p` can also be used:

```
$ p ((struct RBasic *)$r9)->flags
(VALUE) $8 = 0x4007
```

## Ruby concepts

Execution:

- Control Frame Pointer (CFP) has pointers to:
  - current iseq
  - PC -> next instruction
  - SP -> top of the stack (next empty slot)
- Each new function creates a CFP
  - new CFP
    - PC: first instruction
    - SP: pointing to empty slot
  - on instructon execution
    - PC is advanced
    - fetch value, process, push in the stack
  - TenderJIT doesn't advance PC and SP!
    - so they may not represent reality
    - they need to be adjusted before returning control

Types:

- `RB_SPECIAL_CONST_P`: any of the 3 low bits set (not allocated)
- `RB_BUILTIN_TYPE`: any other type (allocated)

Pointers:

```c
// Pointer to current VM (in theory, there can be more VMs)
*ruby_current_vm_ptr

// Object space
*ruby_current_vm_ptr->objspace
```

YJIT:

- `/yjit_codegen.c` -> code generator (equivalent of `ISEQCompiler`)
