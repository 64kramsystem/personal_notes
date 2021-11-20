# TenderJIT

- [TenderJIT](#tenderjit)
  - [General notes](#general-notes)

## General notes

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
