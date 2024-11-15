# Crystal

- [Crystal](#crystal)
  - [Compiler basics](#compiler-basics)
  - [Language basics](#language-basics)
    - [Debugging-related](#debugging-related)
    - [Typing](#typing)
    - [Syntax](#syntax)

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
```

### Typing

Data types:

```rb
"str"                   # String
-42                     # Int32
'c'                     # Char
false                   # Bool
[]                      # Array(type)
{}                      # Hash(type => type)
:sym                    # Symbol
(Int32 | Nil)           # Union
@start..@end            # Range
Pointer(Void).null      # (Null) Pointer

false, nil, null_ptr    # Falsey; anything else is truthy
```

Syntax/methods:

```rb
typeof(@val)            # Parentheses required
foo : Int32 [ = @val]   # Manually declare a type
```

Crystal can detect codepaths where the Nil type is removed from a union, like:

```rb
str, i = "foo", "foo".index("b")
# Raises `expected argument #1 to 'String#[]' to be Int, not (Int32 | Nil)`
puts str[i]
# Runs
puts str[i] if i
```

### Syntax

```rb
$invalid            # Globals are not allowed
MYCONST = 42        # Constant; can't be changed
"_#{@expr}_"        # String interpolation
```

Collections:

```rb
foo = [] of String        # Empty collections require the type
bar = {} of String => Int32
Array(String).new         # Alt. syntaxes
Hash(String, Int32).new

foo = ['f', "oo"]         # Valid - array of union type

bar = { "foo" => "bar" }  # Standard hash
```

Methods:

```rb
def foo(bar); end         # Types are not required in the signature
def foo(bar: Int32); end  # Method with type
```
