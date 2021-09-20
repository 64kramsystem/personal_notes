# RACC/Parsing

- [RACC/Parsing](#raccparsing)
  - [References](#references)
  - [Lexer](#lexer)
    - [Gotchas](#gotchas)
  - [Parser](#parser)

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

Comments can be `//` or `#`; blank lines are mandatory!

```sh
cat > lexer.rex <<'REX'
class TestLanguage
# The `macro` section values are regex patterns; they don't need quoting
macro
  BLANK     [\ \t]+    # Can also use `?` metachar
  DIGIT     \d+
  ADD       \+
  SUBTRACT  \-

# The symbols produced (e.g. `DIGIT`) are called "terminals" or "tokens".
#
# WATCH OUT! If whitespace is meaningful to the parser (eg. to distinguish separated from joined tokens),
# then the rule must return a value.
#
rule
  {BLANK}      # no action; it's discarded and the parse doesn't receive it
  {DIGIT}      { [:DIGIT, text.to_i] }  # return values must be Array(2) (or nothing at all)
  {ADD}        { [:ADD, text] }
  {SUBTRACT}   { [:SUBTRACT, text] }

inner
  # This is just for testing the lexer; it's not needed for parsing.
  #
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

### Gotchas

1. There is no semantic safety in the lexer/parser definitions (only syntax checks of the parser file format); if there is a mistake, e.g misnamed token, it will cause a parsing error while parsing, without any mention of the actual cause!

2. Tokens ordering matters; in the following example, the `SUN` rule (the macro doesn't matter) must precede the `S` and `U`, otherwise, the `sun` string will be interpreted as `S`->`U`->error:

```
macro
  SUN        sun
  S          s
  U          u

rule
  {SUN}        { [:SUN, text] }
  {S}          { [:S, text] }
  {U}          { [:U, text] }
```

## Parser

The rule code blocks won't be executed if there is a parsing error, so they can't be used for debug purposes.

Blank lines are not mandatory here.

```sh
cat > parser.y <<'RACC'
# This is example performs logic inside the parser, so parsing the expression returns the result. Non
# trivial parsers separate domains (as possible).
# `val` is an array; fetch() is just used to avoid bugs.
#
class TestLanguage
rule
  expression
    : DIGIT
    | DIGIT ADD DIGIT       { return val.fetch(0) + val.fetch(2) } # MUST include the `return`!!
    | DIGIT SUBTRACT DIGIT  { return val.fetch(0) - val.fetch(2) }
    ;

  # Repetition is encoded as recursion
  options
    : options option
    | option                # define option is in a separate rule
    ;

  # In order to define optional productions, must use three cases:

  rule_b
    : TOK_B
    | TOK_B rule_c     # If `c` is the last token of the string, use the token (ie. TOK_C) instead of the rule.
    | rule_c
    ;

  # The empty string allows a simpler encoding of optional productions.
  # In order for this to work, optional tokens (in this case B and C) must not cause conflicts (ambiguity):
  # - the optional tokens should not start with the same letter;
  # - if they're productions, the first token must not be ambiguous with the one following the optional
  #   production, e.g. `rule_C: | D C`.

  expression: A rule_B rule_C D
  rule_B: | B                 # syntax for empty string
  rule_C: | C
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
