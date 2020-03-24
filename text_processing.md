## Table of contents

- [Sed](#sed)
  - [Useful examples](#useful-examples)
  - [Special characters](#special-characters)

## Sed

### Useful examples

Delete numbered line: `1d`.

### Special characters

In order to handle tabs (`\t`), either use `$` quoting or parameter substitution:

```sh
sed $'s/\t/ /'
sed "s/$(printf '\t')/ /"
```
