# TenderJIT

- [TenderJIT](#tenderjit)
  - [New notes (to review)](#new-notes-to-review)
  - [Old notes (to review)](#old-notes-to-review)
  - [Debug mode](#debug-mode)
  - [Generic design](#generic-design)
  - [Useful patterns](#useful-patterns)

## New notes (to review)

- bt backtrace
- f frame
- p print
- rp ruby print

```
# Kill self; get an extensive stack trace
# Ruby will create a core dump
irb> Process.kill 'SEGV', $$

# Required  to create a core dump (check why)
$ ulimit -c unlimited

# Debug using a core dump
$ lldb --core /path/to/core.dump ruby

# Now can print the backtrace
(lldb) bt
```

```
# In theory, there can be more VMs
p *ruby_current_vm_ptr

# Get the object space
p *ruby_current_vm_ptr->objspace

# In upcoming Ruby versions, there are more object pools
p ruby_current_vm_ptr->objspace->size_pools[0].eden_heap
```

## Old notes (to review)

- `yjit/yjit_codegen.c` -> code generator (equivalent of `ISEQCompiler`)

- When a request is made to compile an instruction sequence (**iseq**), the compiler checks to see if there is already an `ISEQCompiler` object associated with the iseq. If not, it allocates one, then calls compile on the object.
  - The compiler will compile as many instructions in a row as it can, then will quit compilation. Depending on the instructions that were compiled, it may resume later on.
- Unimplemented instructions don't have the `handle` method
  - When no corresponding handler function is found, the compiler will generate an "exit" and the machine code will pass control back to YARV
- **Control Frame Pointer** has pointers to:
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

## Debug mode

Command: `ruby -d`

- The address on the right is of the method to executed instruction belongs to
- `NEW ISEQ Compiler` -> means recursively compiled new method

## Generic design

The are two compilers:

- an instant one (`handle_*`)
- a deferred one

the deferred one executes when a method is invoked, since it needs to figure out the receiving object.

## Useful patterns

```rb
klass_addr = rb.rb_class_of(recv) # returns the pointer
puts Fiddle.dlunwrap(klass_addr)  # find the class; in general, `dlunwrap` returns the object at the given address
```
