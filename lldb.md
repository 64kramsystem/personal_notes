# lldb

- [lldb](#lldb)
  - [Misc](#misc)
  - [Common commands](#common-commands)

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

Disassemble/Inspect:

```sh
di -b -s 0x1eb8 -c 20 # disassemble --bytes --start-address 0x1eb8 --count 20
m rea $rdi-8          # memory read
re re [$reg]          # register(s) read
```

Operational:

```
Ctrl+D                # exit without (annoying) confirmation
```
