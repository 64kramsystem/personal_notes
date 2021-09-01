# RACC/Parsing

- [RACC/Parsing](#raccparsing)
  - [References](#references)
  - [Lexer](#lexer)
  - [Parser](#parser)
    - [More complex example](#more-complex-example)

## References

Sources:
 - https://github.com/tenderlove/rexical/wiki/Overview-of-rexical
 - https://github.com/tenderlove/rexical/wiki/Overview-of-rexical-part-2
 - https://github.com/ruby/racc/blob/master/doc/en/Overview-of-racc.md

Small tutorial on a more complex parser: https://practicingruby.com/articles/parsing-json-the-hard-way

```sh
if [[ -z $(gem list -q '^rexical$') ]]; then
  gem install rexical
fi
```

## Lexer

```sh
cat > lexer.rex <<'REX'
class TestLanguage
// The `macro` section values are regex patterns, so a character doesn't need quotes.
macro
  BLANK     [\ \t]+
  DIGIT     \d+
  ADD       \+
  SUBTRACT  \-

// The symbols produced (e.g. DIGIT) are called "terminals" or "tokens".
// The return values MUST be Array[2]!
rule
  {BLANK}      # no action
  {DIGIT}      { [:DIGIT, text.to_i] }
  {ADD}        { [:ADD, text] }
  {SUBTRACT}   { [:SUBTRACT, text] }

inner
  def tokenize(code)
    scan_setup(code)
    tokens = []
    while token = next_token; tokens << token; end
    tokens
  end
end
REX

rex lexer.rex -o lexer.rb

ruby -r ./lexer <<'RUBY'
  puts TestLanguage.new.tokenize("2")   # [[:DIGIT, 2]]
  puts TestLanguage.new.tokenize("+")   # [[:ADD, "+"]]
  puts TestLanguage.new.tokenize("2+2") # [[:DIGIT, 2], [:ADD, "+"], [:DIGIT, 2]]
RUBY
```

## Parser

```sh
cat > parser.y <<'RACC'
class TestLanguage
rule
  expression
    : DIGIT
    | DIGIT ADD DIGIT       { return val[0] + val[2] } # MUST include the `return`!!
    | DIGIT SUBTRACT DIGIT  { return val[0] - val[2] }
    ;
end

---- header
  require_relative 'lexer'

---- inner
  def parse(input)
    scan_str(input)
  end
RACC

racc parser.y -o parser.rb

ruby -r ./parser <<'RUBY'
  puts TestLanguage.new.parse("2")   # 2
  puts TestLanguage.new.parse("2+2") # 4
RUBY
```

### More complex example

```racc
  // Ordering matters!
  options              // Repetition is encoded as recursion
    : options option
    | option
    ;
  option
    : FIXED       { checked_assign(:fixed, val[0]) }
    | FIXED TIME  { checked_assign(:fixed_time, val[1]) }
    | SKIP        { checked_assign(:skip, val[0]) }
    | UPDATE      { checked_assign(:update, val[0]) }
    ;
```
