# lldb

- [lldb](#lldb)
  - [To review](#to-review)
  - [Misc](#misc)
  - [Common commands](#common-commands)
  - [Core dump/load](#core-dumpload)

## To review

- `bt`: backtrace
- `f`: frame
- `p`: print
- `rp`: Ruby print (from ``misc/lldb_cruby.py``)

## Misc

```sh
# Run on start; multiple `-o` can be specified.
#
lldb -o run $filename
```

WATCH OUT!! When debugging a Ruby sessions, must NOT load bundler. See git.io/JirNE for an implementation; when running via shell script, the principle is the same (don't use bundle exec neither outside the script nor inside).

## Common commands

General:

```sh
h $command            # help $command

di -s $RIP-5          # expression!! RIP - 5 bytes
```

Disassemble/Inspect/Modify:

```sh
di -b -s 0x1eb8 -c 20 # disassemble --bytes --start-address 0x1eb8 --count 20
p [$expr]             # evaluate and print the expression

m rea $rdi-8          # memory read
m rea -s 8 -f x $rsp  # read with a size of 8 bytes, format hex (ie. display qwords)

re re [$reg]          # register(s) read
re wr $reg $val       # register write

bt                    # print backtrace
f [$idx]              # print the current/indexed frame (0 = current)
```

Operational:

```
Ctrl+D                # exit without (annoying) confirmation
```

## Core dump/load

- save the core: `pr sa $path` (`process save-core`)
- load via: `lldb --core $path $binary` (binary running at dump time)
