# Ruby Core development

- [Ruby Core development](#ruby-core-development)
  - [Disassemble code](#disassemble-code)
  - [Compile with custom options](#compile-with-custom-options)

## Disassemble code

```rb
# Printout form
RubyVM::InstructionSequence.of(method(:empty_hash)).disasm
RubyVM::InstructionSequence.compile(%q[ {a: 2, b: 'something'} ]).disasm

# Structured format (ignore the rest of the data)
#
RubyVM::InstructionSequence.compile(%q[ {a: 1, b: 2} ]).to_a.last[2..] # => [[:duphash, {:a=>1, :b=>2}], [:leave]]
```

## Compile with custom options

```sh
# RVM: Compile with debug symbols
#
# -ggdb3 produces gdb debug info, at maximum level (see https://stackoverflow.com/a/10475077).
# -O0 disables optimizations; use at discretion (debug info is still present)
#
# Disabling rvm's binary download is crucial!
#
# If compiling from source, just export `optflags`.
#
optflags="-O0 -ggdb3" rvm install 3.0.2-dbg --disable-binary

# RVM: Compile with Clang (includes debug info by default)
#
rvm install 3.0.2-cla --with-gcc=clang
```
