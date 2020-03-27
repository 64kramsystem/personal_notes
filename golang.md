## Table of contents

- [General notes](#general-notes)
  - [Builtin  functions](#builtin--functions)
  - [Arrays, slices](#arrays-slices)
  - [Data types](#data-types)
  - [I/O](#io)
  - [Loops](#loops)
    - [For](#for)
  - [Modules](#modules)
  - [O/S](#os)
  - [Shared libraries (invoke from other languages)](#shared-libraries-invoke-from-other-languages)
  - [String operations](#string-operations)
  - [Variables](#variables)
- [Mastering Go (2nd ed.)](#mastering-go-2nd-ed)
  - [01. Go and the Operating system](#01-go-and-the-operating-system)
    - [Command line arguments](#command-line-arguments)
    - [Standard file descriptors/IO, and useful I/O operations](#standard-file-descriptorsio-and-useful-io-operations)
    - [String handling: printing and conversion](#string-handling-printing-and-conversion)
    - [Logging](#logging)
    - [Custom errors](#custom-errors)

## General notes

### Builtin  functions

```golang
len(collection)                // elements in a collection; String -> number of bytes
```

### Arrays, slices

```golang
slice := array[[<start>]:[<end>]]
```

### Data types

Data types:

- `float64`

Casting:

```golang
toType(fromType)
```

### I/O

```golang
import "os"

f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)    // create a file

import "path/filepath"

filepath.Base(string)        // file basename
```

### Loops

#### For

```golang
for <boolean> {}
for a, b := range <collection> {}         // index/entry for Arrays, key/value for Maps
for <start_assignment>; <condition>; <increment_statement> {}
```

### Modules

[Modules](https://github.com/golang/go/wiki/Modules) basic usage:

```sh
go mod init github.com/my/repo
```

There are many other aspects, including:

- allow the project to be stored anywhere
- dependencies are automatically handled

Configuration is stored in a `go.mod` file.

### O/S

```golang
os.Exit(<exit_status>)              /// exit to the O/S
```

### Shared libraries (invoke from other languages)

See https://github.com/vladimirvivien/go-cshared-examples.

### String operations

String formatting:

```golang
str := fmt.Sprintf("%s, %s", date, time)    // printf (to string) in Golang
```

### Variables

```golang
a, b := 3, 4
```

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
