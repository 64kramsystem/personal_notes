# lldb

- [lldb](#lldb)
  - [Misc](#misc)

## Misc

```sh
# Run on start; multiple `-o` can be specified.
#
lldb -o run $filename
```

WATCH OUT!! When debugging a Ruby sessions, must NOT load bundler. See git.io/JirNE for an implementation; when running via shell script, the principle is the same (don't use bundle exec neither outside the script nor inside).
