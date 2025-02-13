# Crystal libraries

- [Crystal libraries](#crystal-libraries)
  - [Built-in data types](#built-in-data-types)
    - [Range](#range)
    - [Collection-related](#collection-related)
      - [Array](#array)
      - [Hash](#hash)
    - [String](#string)
  - [Kernel methods](#kernel-methods)

## Built-in data types

### Range

APIs

```cr
range.(includes|covers|in)? value   # inclusion
range.each { ... }
range.sample
range.sum
```

### Collection-related

#### Array

Common methods:

```rb
[@i]
[@i]=
each(@block)
delete(@entry)
includes(@entry)
shift(@entry)
pop()
```

#### Hash

```rb
[@key]               # Infallibe
[@key]?              # Fallible
has_key?(@key)
```

### String

```rb
"str".to_i            # Doesn't support nil
```

## Kernel methods

```rb
rand($val)            # can use a Range
```
