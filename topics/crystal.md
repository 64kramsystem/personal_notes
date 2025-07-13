 Crystal

- [Compiler basics](#compiler-basics)
- [Managing a project](#managing-a-project)
- [Debugging-related](#debugging-related)
- [Types](#types)
  - [Built-in types](#built-in-types)
  - [Numeric](#numeric)
  - [Collections](#collections)
    - [Enumerable](#enumerable)
    - [Array](#array)
    - [Hash](#hash)
    - [(Named)Tuple](#namedtuple)
    - [Other ones](#other-ones)
  - [Enum](#enum)
  - [Range](#range)
  - [String](#string)
  - [Regex](#regex)
  - [Floats](#floats)
  - [Time](#time)
  - [File/IO](#fileio)
- [General syntax](#general-syntax)
  - [Control flow](#control-flow)
  - [Blocks](#blocks)
- [Top-level methods](#top-level-methods)
- [Special variables/constants](#special-variablesconstants)
- [Object orientation](#object-orientation)
  - [Project organization](#project-organization)
  - [Methods](#methods)
  - [Classes](#classes)

## Compiler basics

```sh
crystal [run] $file                              # Run
crystal b[uild] [--release] [--no-codegen] $file # Build; `--no-codegen` doesn't generate a file
crustal i[nteractive]                            # REPL
```

## Managing a project

```sh
crystal init (app|lib) $dir
# … edit shard.yml (see below) …
shards install
shards build
shards run [-- $binary_params…]
```

`shard.yml` example:

```yml
name: openscripts
version: 0.1.0
license: GPL3

authors:
  - Saverio Miroddi <saverio.pub2@gmail.com>

targets:
  ../download_ubuntu_packages:          # binary path (relative to `bin/`!)
    main: src/download_ubuntu_packages.cr

crystal: '>= 1.16.3'

dependencies:
  progress_bar:
    github: mgomes/progress_bar
    # version: ~> 0.4.0
    branch: master
```

## Debugging-related

```cr
puts(); pp(); print(); printf(); sprintf()  # Basic debug (print) commands
p "foo"                                     # Shorthand for `puts`
export CRYSTAL_OPTS="--error-trace"         # Print the full stack trace
```

## Types

### Built-in types

```cr
# Primitive

-42                     # Int32 (8..64), UInt
2.2                     # Float64 (32, 64)
false                   # Bool
'c'                     # Char
nil

 Core Sdtlib

"str"                   # String
:sym                    # Symbol
[:foo, :bar]            # Array
{"foo" => 2}            # Hash
{:foo, 1, "bar"}        # Tuple(Symbol, Int32, String)
{foo: 2}                # NamedTuple
Set{1, 2}
Deque(Int32).new
@start..@end            # Range

# Implicitly Available

Pointer(@type)          # Pointer(type)
```

```cr
(Int32 | Nil)           # Union
false, nil, null ptr    # Falsey; anything else is truthy
```

Syntax/methods:

```cr
typeof(@val)             # Parentheses required; accepts any expression (see below)
@val.is_a?(String)       # Type check
@val.as(OtherType)       # Checked runtime typecast; raises error if invalid
@val.as?(OtherType)      # Safe runtime typecast; returns nil if invalid

foo : Int32 [= @val]    # Manually declare a type
foo : Int32? [= nil]    # Nilable type declaration
[1, 2] of Int32 | String # Literal and type can be defined together

Pointer(Void).null       # Null pointer

2_u16                    # Specify a data type
@val.not_nil!            # Force treat the value as not nil
```

Crystal can detect codepaths where the Nil type is removed from a union, like:

```cr
str, i = "foo", "foo".index("b")
# Raises `expected argument #1 to 'String#[]' to be Int, not (Int32 | Nil)`
puts str[i]
# Type restriction (automatic conversion to non-nilable)
#
# WATCH OUT! Under some circumstances, it doesn't work:
# - instance variables (https://crystal-lang.org/reference/1.15/syntax_and_semantics/if_var_nil.html#instance-variables)
# - blocks changing the variable (https://crystal-lang.org/reference/1.15/syntax_and_semantics/closures.html#type-of-closured-variables)
#
# In such cases:
# - assign to a local variable before testing
# - or (TBC) use `var.not_nil!`

puts str[i] if i
```

Variables can be typed using a `typeof` call that infers the type from an expression, e.g.:

```cr
def newlist(list, &)
  new_list = [] of typeof(yield list[0])
end
```

### Numeric

```cr
Math.max(@a, @b)            # Find min/max between two numbers
```

### Collections

#### Enumerable

```cr
reduce { |accumulator, entry| }   # inject() doesn't exist
min; max
min_by { |e| }; max_by { |e| }
min_of { |e| }; max_of { |e| }    # like max_by, but returns the block retval
```

#### Array

Syntax:

```cr
['f', "oo"]                 # Valid - array of union type
[] of String                # Empty collections require the type
Array(String).new           # Alt. syntax
```

APIs:

```cr
[@i]
[@i]=
[@i, @l]                    # slice() doesn't exist
each(@block)
delete(@entry)
includes(@entry)
shift(@entry)
pop()

map {}.to_a.reverse         # WATCH OUT! reverse() doesn't exist for map iterators
```

#### Hash

Syntax:

```cr
{:foo => 2}                 # WATCH OUT!!! Hashes use `=>`.
{} of String => Int32
Hash(String, Int32).new     # Can pass a default (value or block)
```

APIs:

```cr
[@key]               # Infallibe
[@key]?              # Fallible
has_key?(@key)
to_a.sort_by { }     # sort_by() doesn't exist; must convert to Array
```

#### (Named)Tuple

Immutable; each element has its own type, not union (like `Array`).

Syntax:

```cr
t = {:foo, 1, "bar"}
nt = {foo: 2, bar: 3}
NamedTuple.new                           # empty NT declaration
NamedTuple(foo: Int32).from({:foo => 1}) # convert Hash to NT; tuples must match

# Named tuples can be unpacked using values(), but it doesn't automatically match by variable name
foo, bar = nt.values
```

APIs:

```cr
nt[:foo]           # infallible; fetch() doesn't exist
nt[:bar]?          # fallible
nt.merge(foo: 1)   # "build" named tuples
```

#### Other ones

```cr
# Deque is optimized for fast insertion/deletion on both ends

Deque(Int32).new
```

### Enum

```cr
enum Foo
  Bar
  Baz
end

var = Foo::Bar
```

### Range

Syntax:

```cr
0.0...1.0     # Right-open interval, also supporting floats (!!)
..1           # Can be limitless also on the right and both sides
'a'..'z'      # Chars supported
"aa".."zz"    # ^^ strings too (!!)
```

APIs:

```cr
range.(includes|covers|in)? value   # inclusion
range.each { ... }
range.sample
range.sum
```

### String

```cr
"str".to_i              # Doesn't support nil

# Heredoc format.
# Leading whitespace is removed until the `-` (can't be earlier!).
str =
  <<-STRING # => "Hello\n  world"
    Hello
      world
    STRING
```

### Regex

```cr
val = match[0] if match = /bar/.match("foo")          # Workaround lack of $LAST_MATCH_INFO
```

### Floats

```cr
-Float64::INFINITY      # Float::INFINITY doesn't exit
```

### Time

```cr
Time.local              # Time.now() doesn't exist
Time.local - @n.minutes # Operations
@time.to_s("%i")        # strftime() doesn't exist
```

### File/IO

```cr
# IO doesn't exist; lines terminators are stripped.
# Expects a block, so it does not return an enumerable.
#
File.each_line(@file) { |line| ... }
```

## General syntax

```cr
$invalid            # Globals are not allowed
MYCONST = 42        # Constant; can't be changed
"_#{@expr}_"        # String interpolation
foo, bar = 0, 1     # Multiple assignment (can also exchange variables)

foo ? bar : baz     # Ternary operator is supported
(a) .. (b)          # Flip-flop operator is NOT supported

var.try { |v| … }   # Safe navigation operator
```

Simulate a flip-flop operator:

```cr
# Range-style

File.each_line(@file).reduce(false) do |inside_range, line|
  inside_range = !inside_range if line =~ /start_regex/ || (inside_range && line =~ /end_regex/)
  puts line if inside_range
  inside_range
end

# Activation only

File.each_line(@file).reduce(false) do |activated, line|
  activated ||= line =~ /start_regex/
  puts line if activated
  activated
end
```

### Control flow

```cr
case $val
when 1, 2, 3
when Int32
when 1..3
else
end
```

### Blocks

```cr
@arr.{ |(a, b)| }             # Unpack tuples
```

Complex block control flow (`next`/`break`/`return`):

```cr
def generate(&)
  first = yield 1
  second = yield 2
  third = yield 3

  first + second + third
end

result = generate do |x|
  # result = 16: (1+1) + 10 + (3+1)
  #
  next 10 if x == 2

  # result = 10: generate() returns immediately
  #
  break 10 if x == 2

  # in this case, error - can't call from top level
  # in general, it exits entirely from the entire surrounding method
  #
  return 10 if x == 2

  x + 1
end
```

## Top-level methods

```cr
rand($val)                    # Also accepts Range
```

## Special variables/constants

```cr
PROGRAM_NAME                  # Invoked program name (binary)
__FILE__                      # WATCH OUT! Source file, not binary!
$LAST_MATCH_INFO              # Doesn't exist; see Regex
```

## Object orientation

### Project organization

```cr
require "./file"              # File relative to current path
require "file"                # Standard library
```

### Methods

Crystal support overloading (by arguments/type).

```cr
def foo(bar)                  # Types are not required in the signature
def foo(bar: Int32 | Int64)   # Method with (union) type
def foo(bar = 15)             # Default value
foo(bar: 33)                  # Arguments are both named and positional (!!)
def foo(bar, *, baz = 33)     # `*` forces argument to the left to be positional
def increment(by value = 1)   # Arguments can have different external and internal names
def foo(bar, &)               # `&` is optional: denotes a block
```

WATCH OUT!! When returning multiple values, don't return `Array`, return `Tuple`:

```cr
def parse; {1, "a"}; end  # num=Int32, str=String
def parse; [1, "a"]; end  # both are (Int32 | String)
num, str = parse
```

### Classes

Base structure, with generics:

```cr
class MinHeap(T)
  def initialize
    @items = [] of T
  end

  def pop : T?
    @items.pop
  end
end
```
