# Golang

- [Golang](#golang)
  - [Syntax](#syntax)
    - [Variables/Constants](#variablesconstants)
    - [Data types](#data-types)
    - [Arrays, slices](#arrays-slices)
    - [Maps](#maps)
    - [Loops](#loops)
      - [For](#for)
    - [Builtin  functions](#builtin-functions)
  - [APIs](#apis)
    - [Strings](#strings)
      - [General concepts](#general-concepts)
      - [Formatting](#formatting)
    - [Math](#math)
    - [I/O](#io)
    - [O/S](#os)
  - [Management](#management)
    - [Modules](#modules)
    - [Shared libraries (invoke from other languages)](#shared-libraries-invoke-from-other-languages)
  - [Mastering Go (2nd ed.)](#mastering-go-2nd-ed)
    - [01. Go and the Operating system](#01-go-and-the-operating-system)
      - [Command line arguments](#command-line-arguments)
      - [Standard file descriptors/IO, and useful I/O operations](#standard-file-descriptorsio-and-useful-io-operations)
      - [String handling: printing and conversion](#string-handling-printing-and-conversion)
      - [Logging](#logging)
      - [Custom errors](#custom-errors)

## Syntax

### Variables/Constants

```golang
var v <type>
a, b := 3, 4              // Multiple assignment
const c = 1               // Type is optional
```

Constants, with `iota` and (optional) custom type:

```golang
type Position int

const (
  First Position = iota // 0, automatic
  Second                // 1
)
```

### Data types

Data types:

- `float64`

Casting:

```golang
toType(fromType)
```

Typedef:

```golang
type Kind int
```

### Arrays, slices

Arrays:

```golang
var arr[<size>]<data_type>        // instantiate an array
arr := [...]string{"a", "b"}      // inline array declaration
```

Slices:

```golang
slice := []string{"a", "b"}
slice := make([]string, <size>[, <capacity>])
slice := array[[<start>]:[<end>]] // make from an array; **end element is not included**
```

Accessing:

- Golang doesn't have negative indexing! Use `len() - 1`.

### Maps

Map literal:

```golang
OrbitalPeriods := map[Planet]float64{
  "Mercury": 0.2408467,
}
```

### Loops

#### For

```golang
for <boolean_expr> {}
for a, b := range <collection> {}                   // index/entry for Arrays, key/value for Maps
for <start_expr>; <condition>; <increment_expr> {}
```

### Builtin  functions

```golang
len(collection)                // elements in a collection; String -> number of bytes
```

## APIs

### Strings

#### General concepts

Strings are seen both as a collection of bytes, and a collection of "Runes" (individual characters, stored as `int32`).

Accessing a string via index, or using `len`, will access the individual bytes.

```golang
str_runes := []rune(str);
fmt.Printf("String chars len: %v\n", len(str_runes))
```

Iterate runes:

```golang
for index, runeValue := range str {
  fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
}
```

#### Formatting

String formatting:

```golang
str := fmt.Sprintf("%s, %s", date, time)    // printf (to string) in Golang
```

Formatters:

- Generic
  - `%v`: the value in a default format; when printing structs, the plus flag (%+v) adds field names
  - `%T`: Go-syntax representation of the type of the value
- Integer
  - `%b`: base 2
  - `%c`: the character represented by the corresponding Unicode code point
  - `%X`: base 16, with upper-case letters
  - `%U`: Unicode format: U+1234; same as "U+%04X"

Using `%#`  increases the verbosity for some: at least `%v`, `%U`.

### Math

```golang
math.Inf(sign int)              // positive/negative sign = positive/negative infinity
math.IsInf(sign int)            // positive/negative, or 0 for indifferent

math.NaN
math.IsNaN
```

Note that infinity _is_ a number.

### I/O

```golang
import "os"

f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)    // create a file

import "path/filepath"

filepath.Base(string)        // file basename
```

### O/S

```golang
os.Exit(<exit_status>)              /// exit to the O/S
```

## Management

### Modules

[Modules](https://github.com/golang/go/wiki/Modules) basic usage:

```sh
go mod init github.com/my/repo
```

There are many other aspects, including:

- allow the project to be stored anywhere
- dependencies are automatically handled

Configuration is stored in a `go.mod` file.

### Shared libraries (invoke from other languages)

See https://github.com/vladimirvivien/go-cshared-examples.

## Mastering Go (2nd ed.)

### 01. Go and the Operating system

#### Command line arguments

```golang
import "os"

arguments := os.Args
```

#### Standard file descriptors/IO, and useful I/O operations

Standard FDs are represented by `os.Stdin`, `os.stdOut`, `os.Stderr`.

```golang
import "io"

io.WriteString(os.Stdout, s)     // allows to write to any IO stream (`w Writer`, ...)
```

Read line by line:

```golang
import "bufio"

scanner := bufio.NewScanner(f)
for scanner.Scan() {
  fmt.Println(">", scanner.Text())
}
```

#### String handling: printing and conversion

Printing:

```golang
import "fmt"

fmt.Println(v1, v2)            // a space is inserted between the variables
fmt.Print(v1, v2, "\n")
fmt.Printf("%s%d\n", v1, v2)
```

Conversion:

```golang
import "strconv"

b, err := strconv.ParseBool("true")
f, err := strconv.ParseFloat("3.1415", 64)           // (..., bitSize int (32/64))
i, err := strconv.ParseInt("-42", 10, 64)            // (..., base int, ...)
u, err := strconv.ParseUint("42", 10, 64)
```

#### Logging

```golang
import "log/syslog"

sysLog, err := syslog.New(syslog.LOG_INFO | syslog.LOG_LOCAL7, programName)        // (priority|facility)

log.Fatal(v ...interface{})        // terminates the Go program, after logging
log.Panic(v ...interface{})        // also terminates, but prints more information

// Log levels: composition of: (Print|Fatal|Panic)(|f|ln)
// Standard log facilities: auth, authpriv, cron, daemon, kern, lpr, mail, mark, news, syslog, user, UUCP, local(0..7)

myLog := log.New(f, ogLinePrefix, log.LstdFlags | log.Lshortfile)    // Custom logger; LstdFlags prints the timestamp, Lshortfile filename+line num
```

#### Custom errors

```golang
err := errors.New("Error in returnError() function!")
err.Error()                        // get the message
panic(err)                        // panic throwing the given error
```
