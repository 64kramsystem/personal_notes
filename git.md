## Table of contents

- [Bisect](#bisect)

## Bisect

Run using the command passed exit code in order to discern failures:

```sh
git bisect run <command>'
```

If the command has aliases/etc, use `bash -c '<commands>'` as command.

## Export (`archive`)

Export in tar format:

```sh
# Example export->import into another directory.
git archive master Gemfile Gemfile.lock gems_dir | tar x -f - -C /export
```
