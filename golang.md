# Golang

- [Golang](#golang)
  - [General program structure](#general-program-structure)
  - [Syntax](#syntax)
    - [Variables/Constants/Literals](#variablesconstantsliterals)
    - [Functions](#functions)
    - [Data types](#data-types)
    - [Pointers](#pointers)
    - [Operators](#operators)
    - [Arrays, slices](#arrays-slices)
    - [Maps](#maps)
    - [If/then/else, switch/case](#ifthenelse-switchcase)
    - [Loops](#loops)
      - [For](#for)
    - [Builtin functions](#builtin-functions)
    - [Errors](#errors)
  - [APIs](#apis)
    - [Strings](#strings)
      - [General concepts](#general-concepts)
      - [Formatting](#formatting)
      - [Parsing/Numerical conversion](#parsingnumerical-conversion)
      - [Utils](#utils)
    - [Regular expressions](#regular-expressions)
    - [Time](#time)
    - [Math](#math)
    - [Random](#random)
    - [I/O](#io)
    - [O/S](#os)
  - [Management](#management)
    - [Modules](#modules)
    - [Shared libraries (invoke from other languages)](#shared-libraries-invoke-from-other-languages)
  - [Mastering Go (2nd ed.)](#mastering-go-2nd-ed)
    - [01. Go and the Operating system](#01-go-and-the-operating-system)
      - [Command line arguments](#command-line-arguments)
      - [Standard file descriptors/IO, and useful I/O operations](#standard-file-descriptorsio-and-useful-io-operations)
      - [Logging](#logging)

## General program structure

```golang
package main

// Can also be one import per line, without round brackets.
import (
	"fmt"
)

func main() {
	fmt.Println("Hello world!")
}
```

then execute `go run main.go` (compile and run).

## Syntax

### Variables/Constants/Literals

```golang
var v <type>
var v [<type>] = <value>

const c [<type>] = 1

a, b := 3, 4              // Multiple assignment; useful also for swapping.

largeNum := 1_000_000_000 // Cosmetic underscores
```

Constants, with `iota` and (optional) custom type:

```golang
type Position int

const (
  First Position = iota // 0, automatic
  Second                // 1
)
```

### Functions

```golang
func myFunc(myVar int) bool { return true }

// Implicit return variables declaration!
func myFunc(myVar int) (myRetVal bool) { return myRetVal }

func myFunc(myVar int) (int, bool) { return 1, true }

// Function variable
myFunc := func(phrase string) bool { return phrase == "" }

// Anonymous function
result := func(phrase string) bool { return phrase == "" }("abc")
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

### Pointers

Passing by value:

- can be less efficient;
- uses the stack, which is simple to manage.

Passign by reference:

- use the heap, which is managed by the garbage collector.

Basic usage:

```golang
// Alternative instantiations
var ptr *int
ptr := new(int)

val := 123
ptr = &val
*ptr = 456

fmt.Printf("%v\n", val)  // 456
fmt.Printf("%v\n", *ptr) // 456
```

### Operators

```golang
var++
var--
```

### Arrays, slices

Arrays:

```golang
var arr [10]int                // instantiate an array
arr := []string{"a", "b"}      // array literal

var sl []int                   // instantiate slice
make([]rune, len(str))         // (other way)

sl = append(sl, 64)            // append to a slice (can't append to an array)
copy(dest[z:a], source[x:y])   // copy slices (indexing is optional)
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

```golang
// Map literal

var OrbitalPeriods = map[Planet]float64{
  "Mercury": 0.2408467,
}

// Check if map contains an element

value, found := myMap["key"]
```

### If/then/else, switch/case

The `<initialization_expr>` can be an assignment (e.g. `err`).

```golang
if [<init_expr>]; <boolean_expr> {
} else if {
} else {
}
```

If `value_expr` is present, the `case` expressions are matching values; if not, then they're expressions.

```golang
switch [<init_expr>;] [<value_expr>] {
case <expr>[, <expr>]:
  <statements>
  [fallthrough]
default:
  <statements>
}
```

Note how in order to only have the `init_expr`, it must be terminated with a semicolon.

### Loops

#### For

```golang
for <boolean_expr> {}

// Multiple variables can be initialized (and also incremented) at once, via multiple assignment.
// Without any expression, it's a while true.
//
for [[<start_expr>]; [<condition>]; [<increment_expr>]] {}

// For strings, `i` is the index of the byte in the string.
//
for i, e := range <collection> {}                   // Array: index/entry, Map: key/value
for i := range <array> {}                           // Array: index (!!), Map: key
```

### Builtin functions

```golang
len(collection)                // elements in a collection; String -> number of bytes
```

### Errors

```golang
err := fmt.Errorf("Unexpected string: %v", str)        // preferred
err := errors.New("Error in returnError() function!")  // alternative

err.Error()                        // get the message
panic(err)                         // panic throwing the given error
```

## APIs

### Strings

#### General concepts

Strings are seen both as a collection of bytes, and a collection of "Runes" (individual characters, stored as `int32`).

Accessing a string via index, or using `len`, will access the individual bytes.

```golang
runes := []rune(str);
fmt.Printf("String chars len: %v\n", len(runes))

fmt.Printf("String chars len: %v\n", utf8.RuneCountInString(str)
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
fmt.Println(v1, v2)                         // a space is inserted between the variables
fmt.Print(v1, v2, "\n")
fmt.Printf("%s%d\n", v1, v2)

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

#### Parsing/Numerical conversion

```golang
b, err := strconv.ParseBool("true")
f, err := strconv.ParseFloat(str, bitsize)           // excessive numbers cause an Inf log, but not an error
i, err := strconv.ParseInt(str, base, bitsize)
u, err := strconv.ParseUint(str, base, bitsize)

str := strconv.Itoa(97)                              // integer to string
string(intVar); string(byteVar)                      // alternative
```

#### Utils

```golang
strings.HasSuffix(str, suffix)
strings.ToUpper(str)
strings.Join(str, separator)
strings.ReplaceAll(str, search, replace)
strings.Split(str, separator)
strings.TrimSpace(str)
strings.Repeat("*", n)o
```

### Regular expressions

```golang
matches, err := regexp.MatchString("<pattern>", "<string>")

// Panics in case of compilation error.
//
r := regexp.MustCompile(`^(-?\d+(.\d+)?), (-?\d+(.\d+)?)$`)

// Same as the basic regexp.MatchString, but doesn't return err.
//
m := r.MatchString

// Finds all matching substrings. Non-capturing groups are included!
// -1: find all matches
//
s := r.FindAllString(coordinatesStr, -1)

// Finds capturing groups, as arrays of []string.
// -1: find all matches
//
g := r.FindAllStringSubmatch(coordinatesStr, -1)

// Search/replace
s := r.ReplaceAllString(s, replace)
```

Modifiers:

- `(?i)`: case insensitive

### Time

Instantiation:

```golang
now = time.now()
t = time.Date(2009, 11, 17, 20, 34, 58, 651387237, time.UTC))
monday = time.Monday
```

Operations/Comparisons:

```golang
t2 := t.Add(25 * time.Second)
t3 := t.Sub(1 * time.Minute)

t2.Before(t)
t2.After(t)
t2.Equal(t)
```

Conversions:

```golang
t.Weekday()
t.Nanoseconds()
```

### Math

```golang
math.Abs(f float)
math.Inf(sign int)              // positive/negative sign = positive/negative infinity
math.IsInf(sign int)            // positive/negative, or 0 for indifferent

math.NaN
math.IsNaN
```

Note that infinity _is_ a number.

### Random

```golang
// imports: "math/rand", "time"
rand.Seed(time.Now().UnixNano())
index := rand.Intn(int)
```

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
