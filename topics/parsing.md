# Parsing

- [Parsing](#parsing)
  - [Ruby/RACC](#rubyracc)
    - [References](#references)
    - [Lexer](#lexer)
      - [Gotchas](#gotchas)
    - [Parser](#parser)
  - [Rust/rust-peg](#rustrust-peg)

## Ruby/RACC

### References

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

### Lexer

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

#### Gotchas

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

### Parser

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

## Rust/rust-peg

```rs
// This allows differentiating expressions via type system; in theory, strings could be used (via
// `$(...)`), but then the consumer should parse again.
//
#[derive(Debug)]
pub enum Expression {
    Identifier(String),
    CountStar,
    String(String),
}

peg::parser! {
    // Fit for the purpose as minimally as possible.
    //
    pub grammar sql_parser() for str {
        // Case-insensitive match (see https://github.com/kevinmehall/rust-peg/issues/216).
        //
        rule i(literal: &'static str) =
            input:$([_]*<{literal.len()}>)
            {
                ? if input.eq_ignore_ascii_case(literal) { Ok(()) } else { Err(literal) }
            }

        rule word_chars() = ['a'..='z' | 'A'..='Z' | '0'..='9' | '_']
        rule whitespace() = [' ' | '\n' | '\t']+;
        rule non_single_quote_chars() = [^ '\'']
        rule non_double_quote_chars() = [^ '"']

        // The $ is a convenience to return the &str; here it needs to be returned because letters()
        // doesn't return a value. See below for another example.
        //
        rule identifier() -> Expression =
            word_chars:$(word_chars()+)
            {
                Expression::Identifier(word_chars.to_string())
            }

        rule count_star() -> Expression =
            i("count") "(*)"
            {
                Expression::CountStar
            }

        // The `/` symbol is the "or" condition.
        //
        // The count rule must be the first, otherwise, the parser starts matching the identifier,
        // and stumbles at the open parenthesis.
        //
        rule expression() -> Expression =
            count:count_star()
            {
                count
            }
            / identifier:identifier()
            {
                identifier
            }

        // Simplified; assumes a string made of word_chars.
        //
        rule string() -> Expression =
            "'" chars:$(non_single_quote_chars()*) "'"
            {
                Expression::String(chars.to_string())
            }
            /
            "\"" chars:$(non_double_quote_chars()*) "\""
            {
                Expression::String(chars.to_string())
            }

        // Here the `$` si convenient, otherwise we need to match the Expression.
        //
        rule column_definition() -> String =
            name:$(identifier())
            defines:$(whitespace() identifier())*
            {
                name.to_string()
            }

        // `++` is a repetition operator (one or more); another one is `**` (zero or more, which
        // can be quantified, like `**<1,>`).
        //
        pub rule create_table() -> (String, Vec<String>) =
            i("create") whitespace() i("table") whitespace() table_name:$(identifier() / string()) whitespace()*
            "(" whitespace()*
            column_defs:(column_definition() ++ (whitespace()* "," whitespace()*))
            whitespace()* ")"
            {
                (
                    table_name.to_string(),
                    column_defs
                )
            }

        // Very simplified; simple equality between an identifier and a string.
        // Additionally, the preceding whitespace is embedded in this rule.
        //
        pub rule where_condition() -> (Expression, Expression) =
            whitespace()+
            i("where")
            whitespace()*
            identifier:identifier()
            whitespace()*
            "="
            whitespace()*
            string:string()
            {
                (identifier, string)
            }


        pub rule select() -> (Vec<Expression>, String, Option<((Expression, Expression))>) =
            i("select")
            whitespace()+
            expressions:(expression() ++ (whitespace()* "," whitespace()*))
            whitespace()+
            i("from")
            whitespace()+
            table_name:$(identifier())
            where_condition:where_condition()?
        {
            (
                expressions,
                table_name.to_string(),
                where_condition,
            )
        }
    }
}

// Ok(("mytable", ["id", "mystr"]))
println!("> {:?}", create_table_parser::create_table("CREATE TABLE mytable (id INT, mystr VARCHAR)"));
// Ok(([CountStar, Identifier("mycol")], "mytable"))
println!("> {:?}", create_table_parser::select("SELECT COUNT(*), mycol FROM mytable"));
```
