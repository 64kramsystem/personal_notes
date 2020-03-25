## Table of contents

- [Perl](#perl)
  - [General concepts](#general-concepts)
    - [Priority](#priority)
  - [Useful examples](#useful-examples)
- [Sed](#sed)
  - [Useful examples](#useful-examples)
  - [Special characters](#special-characters)

## Perl

### General concepts

#### Priority

Watch out the priority!!!

```sh
# Prints only "Line 1" and an empty line
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline && print $line'

# Prints "Line 1" and "Line 2"
printf 'Line 1\nLine 2' | perl -lne 'print $_; $line = readline and print $line'
printf 'Line 1\nLine 2' | perl -lne 'print $_; ($line = readline) && print $line'
```

### Useful examples

Strip trailing file spaces: `chomp if eof`

## Sed

### Useful examples

Delete numbered line: `1d`.

### Special characters

In order to handle tabs (`\t`), either use `$` quoting or parameter substitution:

```sh
sed $'s/\t/ /'
sed "s/$(printf '\t')/ /"
```
