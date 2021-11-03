# Ruby Core development

- [Ruby Core development](#ruby-core-development)
  - [Disassemble code](#disassemble-code)
  - [Compile with custom options](#compile-with-custom-options)
  - [JIT](#jit)
    - [Create an executable buffer, via Fiddle](#create-an-executable-buffer-via-fiddle)

## Disassemble code

```rb
# Printout form
RubyVM::InstructionSequence.of(method(:empty_hash)).disasm
RubyVM::InstructionSequence.compile(%q[ {a: 2, b: 'something'} ]).disasm

# Structured format (ignore the rest of the data)
#
RubyVM::InstructionSequence.compile(%q[ {a: 1, b: 2} ]).to_a.last[2..] # => [[:duphash, {:a=>1, :b=>2}], [:leave]]
```

```sh
ruby --dump=insns -e "puts nil"
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

## JIT

### Create an executable buffer, via Fiddle

Compact version of how Fisk generates the buffer:

```rb
include Fiddle

PROT_READ   = 0x01
PROT_WRITE  = 0x02
PROT_EXEC   = 0x04
MAP_PRIVATE = 0x0002
MAP_ANON    = RUBY_PLATFORM =~ /darwin/ ? 0x1000 : 0x20

def allocate_buffer size
  mmap = Function.new(
    Handle::DEFAULT["mmap"],
    [TYPE_VOIDP, TYPE_SIZE_T, TYPE_INT, TYPE_INT, TYPE_INT, TYPE_INT],
    TYPE_VOIDP,
    name: "mmap"
  )

  ptr = mmap.call 0, size, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANON, -1, 0
  ptr.size = size
  ptr
end

buffer = allocate_buffer 4096
# ... fill the buffer ...
buffer.to_function([], TYPE_VOID).call
```