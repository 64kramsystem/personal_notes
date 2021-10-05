# Ruby Core development

- [Ruby Core development](#ruby-core-development)
  - [Compile with custom options](#compile-with-custom-options)

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
