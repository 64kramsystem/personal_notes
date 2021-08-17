# RACC/Parsing

- [RACC/Parsing](#raccparsing)
  - [References](#references)
  - [Lexer](#lexer)
    - [Test](#test)
  - [Parser](#parser)
    - [Test](#test-1)

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
cat > test_language.rex <<'REX'
class TestLanguage
macro
  BLANK     [\ \t]+
  DIGIT     \d+
  ADD       \+
  SUBTRACT  \-
  MULTIPLY  \*
  DIVIDE    \/

rule
  {BLANK}      # no action
  {DIGIT}      { [:DIGIT, text.to_i] }
  {ADD}        { [:ADD, text] }
  {SUBTRACT}   { [:SUBTRACT, text] }
  {MULTIPLY}   { [:MULTIPLY, text] }
  {DIVIDE}     { [:DIVIDE, text] }

inner
  def tokenize(code)
    scan_setup(code)
    tokens = []
    while token = next_token
      tokens << token
    end
    tokens
  end
end
REX

rex test_language.rex -o lexer.rb
```

### Test

```sh
cat > language_lexer_spec.rb <<'RUBY'
require './lexer'

class TestLanguageTester
  describe 'Testing the Lexer' do
    before do
      @evaluator = TestLanguage.new
    end

    it "tests for a digit" do
      result = @evaluator.tokenize("2")
      expect(result[0][0]).to eql(:DIGIT)
      expect(result[0][1]).to eql(2)
    end

    it "tests for a symbol" do
      result = @evaluator.tokenize("+")
      expect(result[0][0]).to eql(:ADD)
      expect(result[0][1]).to eql("+")
    end

    it "tests for a calculation" do
      result = @evaluator.tokenize("2+2")
      expect(result[0][0]).to eql(:DIGIT)
      expect(result[0][1]).to eql(2)
      expect(result[1][0]).to eql(:ADD)
      expect(result[1][1]).to eql("+")
      expect(result[2][0]).to eql(:DIGIT)
      expect(result[2][1]).to eql(2)
    end
  end
end
RUBY

rspec language_lexer_spec.rb
```

## Parser

```sh
cat > test_language.y <<'RACC'
class TestLanguage
rule
  expression : DIGIT
  | DIGIT ADD DIGIT       { return val[0] + val[2] }
  | DIGIT SUBTRACT DIGIT  { return val[0] - val[2] }
  | DIGIT MULTIPLY DIGIT  { return val[0] * val[2] }
  | DIGIT DIVIDE DIGIT    { return val[0] / val[2] }
end

---- header
  require_relative 'lexer'

---- inner
  def parse(input)
    scan_str(input)
  end
RACC

racc test_language.y -o parser.rb
```

### Test

```sh
cat > language_parser_spec.rb <<'RUBY'
require './parser.rb'

class TestLanguageParser
  describe 'Testing the Parser' do
    before do
      @evaluator = TestLanguage.new
    end

    it 'tests for a digit' do
      @result = @evaluator.parse("2")
      @result.should == 2
    end

    it 'tests for addition' do
      @result = @evaluator.parse("2+2")
      @result.should == 4
    end
  end
end
RUBY
```
