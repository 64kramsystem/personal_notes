# DOS and debugging

See virtualization DOSBox [section](virtualization.md#dosbox).

For general resources, see [Awesome DOS](https://github.com/balintkissdev/awesome-dos).

For compiling C programs, see C [section](c.md#compilers-for-16-bit-dos-target).

## DEBUG.COM commands

- `U [address]`          disassemble
- `T [=address] [count]` trace (execute instructions and stop); with address, it stops **past** it
- `A [address]`          assemble
- `D`                    dump
- `R [register [value]]` print/change registers
