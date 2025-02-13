# Crystal

- [Crystal](#crystal)
  - [Compiler basics](#compiler-basics)
  - [Language basics](#language-basics)
    - [Debugging-related](#debugging-related)
    - [Typing](#typing)
      - [Collections](#collections)
      - [Enums](#enums)
      - [Range](#range)
    - [General syntax](#general-syntax)
      - [Control flow](#control-flow)
    - [Methods](#methods)

## Compiler basics

```sh
crystal [run] $file                # Run
crystal build [--release] $file    # Build
crustal i[nteractive]              # REPL (!)
```

## Language basics

### Debugging-related

```rb
puts(); pp(); print(); printf(); sprintf()  # Basic debug (print) commands
p "foo"                                     # Shorthand for `puts`
export CRYSTAL_OPTS="--error-trace"         # Print the full stack trace
```

### Typing

Data types:

```rb
"str"                   # String
-42                     # Int32 (8..64), UInt
2.2                     # Float64 (32, 64)
'c'                     # Char
false                   # Bool
[]                      # Array(type)
{}                      # Hash(type => type)
:sym                    # Symbol
(Int32 | Nil)           # Union
@start..@end            # Range
Pointer(@type)          # Pointer(type)

false, nil, null ptr    # Falsey; anything else is truthy
```

Syntax/methods:

```rb
typeof(@val)            # (Parentheses required)
@val.is_a?(String)      # Type check
foo : Int32 [ = @val]   # Manually declare a type
foo : Int32?            # Nilable type
Pointer(Void).null      # Null pointer
2_u16                   # Specify a data type
```

Crystal can detect codepaths where the Nil type is removed from a union, like:

```rb
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
#
puts str[i] if i
```

#### Collections

```rb
foo = [] of String        # Empty collections require the type
bar = {} of String => Int32
Array(String).new         # Alt. syntaxes
Hash(String, Int32).new

foo = ['f', "oo"]         # Valid - array of union type

bar = { "foo" => "bar" }  # Standard hash
```

#### Enums

```rb
enum Foo
  Bar
  Baz
end

var = Foo::Bar
```

#### Range

0.0...1.0     # Right-open interval, also supporting floats (!!)
..1           # Can be limitless also on the right and both sides
'a'..'z'      # Chars supported
"aa".."zz"    # ^^ strings too (!!)

### General syntax

```rb
$invalid            # Globals are not allowed
MYCONST = 42        # Constant; can't be changed
"_#{@expr}_"        # String interpolation
foo, bar = 0, 1     # Multiple assignment (can also exchange variables)
foo ? bar : baz     # Ternary operator is supported
```

#### Control flow

```rb
case $val
when 1, 2, 3
when Int32
when 1..3
else
end
```

### Methods

```rb
def foo(bar); end                 # Types are not required in the signature
def foo(bar: Int32 | Int64); end  # Method with type (unions allowed!)
def foo(bar = 15)                 # Default value
foo(bar: 33)                      # Arguments are both named and positional (!!)
```
