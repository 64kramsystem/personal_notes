# Golang

- [Golang](#golang)
  - [General program structure](#general-program-structure)
  - [Syntax](#syntax)
    - [Variables/Constants/Literals](#variablesconstantsliterals)
    - [Functions](#functions)
    - [Data types](#data-types)
      - [Big numbers](#big-numbers)
    - [Pointers](#pointers)
    - [Operators](#operators)
    - [Arrays, slices](#arrays-slices)
      - [Slice behind-the-scenes concepts](#slice-behind-the-scenes-concepts)
    - [Maps](#maps)
    - [Structs](#structs)
      - [Composition](#composition)
    - [Type casting/assertion/switch](#type-castingassertionswitch)
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
    - [Streams read/write](#streams-readwrite)
    - [Files/descriptors](#filesdescriptors)
    - [O/S, Command line arguments](#os-command-line-arguments)
    - [Logging](#logging)
    - [Memory allocation](#memory-allocation)
  - [Management](#management)
    - [Modules](#modules)
    - [Shared libraries (invoke from other languages)](#shared-libraries-invoke-from-other-languages)

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

- `float(32|64)`
- `[u]int(8|16|32|64)`
- `[u]int`: depends on the architecture
- `byte`: alias of `uint8`
- `rune`: alias of `int32`

Casting:

```golang
toType(fromType)
```

Typedef:

```golang
type Kind int
```

#### Big numbers

```golang
bigA := big.NewInt(123)
bigA.Add(bigA, big.NewInt(1))
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

Arrays can be compared (via `==`), if they have the same type and length. Slices can't be compared (via `==`).

Arrays:

```golang
var arr [10]int                // instantiate with a size and default values
arr := [...]string{"a", "b"}   // literal form
```

Slices:

```golang
var sl []int                               // instantiate without size
sl := make([]string, <size>[, <capacity>]) // with a size
sl := []string{"a", "b"}                   // literal form
sl := array[[<start>]:[<end>]]             // make from an array; **end element is not included**
                                           // omitting start/end indexes indicate first/last indexes

sl = append(sl, 64)                        // append to a slice (can't append to/from an array)
sl2 := append(sl, sl...)                   // append a slice to another slice (variadic notation)

copy(dest[z:a], source[x:y])               // copy slices (indexing is optional); !! the destination capacity is equal to the source length !!
cap(sl)                                    // capacity of a slice
```

Accessing:

- Golang doesn't have negative indexing! Use `len() - 1`.

#### Slice behind-the-scenes concepts

Slices are **views on arrays**!! If the underlying array data changes, the slice contents will change as well.

One needs to be aware of **which/when slice operations cause an underlying array to be copied**, since different slices will then point to different arrays!

**Underlying arrays are copied when the capacity is not enough**

### Maps

```golang
// Map literal

var OrbitalPeriods = map[Planet]float64{
  "Mercury": 0.2408467,
}


value, found := myMap["key"]              // Check if map contains an element
delete(myMap, "key")                      // Deletion; returns nothing!
```

### Structs

```golang
type mytype struct {
  field1  []int
  field2  byte
}

// Instantiation

// 1: fields are optional.
mt := mytype{
  field1: []int{1, 2, 3},
}

// 2: all fields must be specified.
mt := mytype{
  []int{1, 2, 3},
  1,
}

// 3: all fields are initialized with zero value
var mt mytype

// Anonymous structs

as := struct {
  field1 []int
  field2 byte
}{}
as.field1 = []int{1, 2, 3}
as.field2 = 3
```

Structs can be compared, as long as the types are the same, and they're comparable (so, no slices!).

#### Composition

```golang
type myints struct {
	i  int
	i2 int
}

type mycomposed struct {
	myints

	i2 int
	f  float64
}

mt := mycomposed{
  myints: myints{ // member/struct names must be the same!
    i:  1,
    i2: 2,
  },
  i2: 3,
  f:  4.4,
}

mt.i = 10         // "promoted" -> can be accessed as member of the parent struct
mt.myints.i = 10  // other way of accessing the member
mt.myints.i2 = 20 // not "promoted", as there is overlapping
mt.i2 = 30
mt.f = 44.0
```

### Type casting/assertion/switch

Simple typecast:

```golang
i := 333
f := float64(i)
```

Type assertion. Types must have the expected interface; it's not a casting:

```golang
func test(v interface{}) {
	f, ok := v.(string)

	if !ok {
		fmt.Println("casted!:", f)
	} else {
		fmt.Println("not casted!")
	}
}

test(123) // not casted!
```

Type switch:

```golang
func test(v interface{}) {
	switch t := v.(type) {
	case bool:
		fmt.Println(t, "is a bool!")
	case float32, float64:
		fmt.Println(t, "is a float!")
	default:
		fmt.Println(t, "is something else!")
	}
}
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
for [[<start_expr>]; [<condition>]; [<increment_expr>]] {
  if true {
    break
  } else {
    continue
  }
}

// For strings, `i` is the index of the byte in the string.
//
for i := range <array> {}                           // Array: index (!!), Map: key
for i, e := range <collection> {}                   // Array: index/entry, Map: key/value. !! the iteration order is randomized !!

// There is no easy way to iterate a range in reverse; the index-based for loop needs to be used.
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

```golang
// The `\n` is not interpreted. there's no way to include a single quote.
//
raw_string := 'foo
bar\nbaz'                             

// Can't be on separate lines.
//
interpreted_string := "foo\nbar\nbaz"
```

#### General concepts

Strings are seen both as a collection of bytes, and a collection of `rune`.

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

Striong formatting:

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
// `strings` package

strings.HasSuffix(str, suffix)
strings.ToUpper(str)
strings.Join(str, separator)
strings.ReplaceAll(str, search, replace)
strings.Split(str, separator)
strings.TrimSpace(str)
strings.Repeat("*", n)o

// `unicode` package

unicode.IsUpper(char)
unicode.IsLower(char)
unicode.IsNumber(char)
unicode.IsPunct(char)
unicode.IsSymbol(char)
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
import "math/rand"

rand.Seed(time.Now().UnixNano())
index := rand.Intn(int)
```

### Streams read/write

Reading line by line:

```golang
scanner := bufio.NewScanner(f)

for scanner.Scan() {
  fmt.Println(">", scanner.Text())
}
```

Writing to a stream:

```golang
io.WriteString(os.Stdout, s)     // allows to write to any IO stream (`w Writer`, ...)
```

### Files/descriptors

Create a file:

```golang
f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
```

Standard FDs:

- `os.Stdin`
- `os.stdOut`
- `os.Stderr`

File(name) operations:

```golang
import "path/filepath"

filepath.Base(string)        // file basename
```

### O/S, Command line arguments

```golang
os.Exit(<exit_status>)              // exit to the O/S
os.Args                             // commandline arguments
```

### Logging

```golang
import "log/syslog"

sysLog, err := syslog.New(syslog.LOG_INFO | syslog.LOG_LOCAL7, programName)        // (priority|facility)

log.Fatal(v ...interface{})        // terminates the Go program, after logging
log.Panic(v ...interface{})        // also terminates, but prints more information

// Log levels: composition of: (Print|Fatal|Panic)(|f|ln)
// Standard log facilities: auth, authpriv, cron, daemon, kern, lpr, mail, mark, news, syslog, user, UUCP, local(0..7)

myLog := log.New(f, ogLinePrefix, log.LstdFlags | log.Lshortfile)    // Custom logger; LstdFlags prints the timestamp, Lshortfile filename+line num
```

### Memory allocation

```golang
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("TotalAlloc (Heap) = %v MiB\n", m.TotalAlloc/1048576)
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
