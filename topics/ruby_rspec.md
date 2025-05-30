# RSpec

- [RSpec](#rspec)
  - [Structure examples](#structure-examples)
    - [Hooks](#hooks)
  - [Setup/configuration](#setupconfiguration)
    - [Add helpers to example groups (`describe`)](#add-helpers-to-example-groups-describe)
    - [Hooks and execution params](#hooks-and-execution-params)
    - [Sharing](#sharing)
  - [Available variables](#available-variables)
  - [Mocks](#mocks)
    - [Matchers](#matchers)
      - [Stdout handling](#stdout-handling)
    - [Matching arguments](#matching-arguments)
    - [Receive counts](#receive-counts)
    - [Responses](#responses)
      - [Conditional stubbing](#conditional-stubbing)
  - [Testing modules](#testing-modules)
  - [Transactions](#transactions)
  - [Customizations](#customizations)
    - [Custom matcher](#custom-matcher)
    - [Custom expectation](#custom-expectation)
  - [Useful stuff](#useful-stuff)

## Structure examples

- `describe`: example group (/suite).

```ruby
describe Game
  describe "#start"
    it "sends a welcome message"
    it "prompts for the first guess"
  describe "#guess"
    context "with no matches"
      it "sends a mark with ''"
    context "with 1 number match"
      it "sends a mark with '-'"
    context "with 1 exact match"
      it "sends a mark with '+'"

describe Stack
  describe "when full"
    describe "when it receives push"
      it "should raise an error"
  describe "when almost full (one less than capacity)"
    describe "when it receives push"
      it "should be full"

describe MessagesController
  describe "POST create" do
    it "creates a new message"
    it "saves the message"
    context "when the message saves successfully"
      it "sets a flash[:notice] message"
      it "redirects to the Messages index"
    context "when the message fails to save"
      it "assigns @message"
      it "renders the new template"
```

### Hooks

```ruby
# Appends the block at the beginning of the `before` list in the same scope, rather than appending.
#
prepend_before { }

# Surround examples with the given code.
#
around :each do |example|
  Timecop.freeze(REFERENCE_DATE) do
    example.run
  end
end
```

## Setup/configuration

On top of each test file, add `require 'rspec'`, which will load `spec/spec_helper.rb`.

### Add helpers to example groups (`describe`)

```ruby
RSpec.configure do |config|
  # Make `:my_key` a shorthand for `my_key: true`
  #
  config.treat_symbols_as_metadata_keys_with_true_values = true

  # Static inclusion.
  #
  config.include MyHelper, :include_my_helper
  config.extend  MyHelper, :extend_my_helper

  # Conditional inclusion, based on the described class.
  #
  config.include MailerSpecHelper, example_group: do
    describes: -> { |described| described < ActionMailer::Base }
  end
end

describe MyClass, :include_my_helper do
  # the helper is NOT available here!

  it "should do something" do
    # the helper is available here!
  end
end

describe MyClass, :extend_my_helper do
  # the helper is available here!

  it "should do something" do
    # the helper is NOT available here!
  end
end
```

### Hooks and execution params

```rb
RSpec.configure do |config|
  # Run this hook if the example group :myconfig parameter is passed.
  # Additionally, use the :myparam example group parameter.
  #
  config.before(:each, :myconfig) do
    preset_data(myparam: self.class.metadata[:myparam])
  end
end

describe MyClass, :myconfig, myparam: 10 do ...
```

### Sharing

Share `let`s:

```rb
# Put this in a file, and require it in `rspec_helper.rb`.
# Shared context has more complex functionality; this is a basic one.
#
RSpec.shared_context "my context" do
  let(:my_double) { double("my_double") }
end

describe LaunchStagingService do
  # Make :my_double accessible to the examples.
  include_examples "my context"
end
```

## Available variables

- `subject`
- `described_class` (not available for helpers)

## Mocks

It's possible to specify multiple ("compound") expectations, but negatives are not allowed:

```rb
not_to raise_error.and output(test_content).to_stdout

# For negatives, define and use the opposite:
#
RSpec::Matchers.define_negated_matcher :succeed, :raise_error
```

### Matchers

See [reference](https://relishapp.com/rspec/rspec-expectations/v/3-10/docs/built-in-matchers).

 | Target       | Methods                                              | Notes                                             |
 | ------------ | ---------------------------------------------------- | ------------------------------------------------- |
 | Class (type) | `be_a`, `be_an`, `be_a_kind_of`, `be_an_instance_of` |                                                   |
 |              | `be` `< / <= / == / => / >` `expected`               |                                                   |
 |              | `be_within(span).of(reference)`                      | Time/Floats                                       |
 |              | `be === expected`                                    |                                                   |
 |              | `eq(expected)`                                       |                                                   |
 |              | `change { dynamic_value }.from(start).to(end)`       | from/to are optional                              |
 |              | `change { dynamic_value }.by(amount)`                |                                                   |
 |              | `match(pattern)`                                     | String also accepted                              |
 |              | `match_array(array)`                                 | Order is ignored!                                 |
 |              | `contain_exactly(*values)`                           | Like `match_array`, but splatted                  |
 |              | `be_false`, `be_true`, `be_falsey`, `be_truthy`      |                                                   |
 |              | `be_empty`                                           |                                                   |
 |              | `start_with`                                         |                                                   |
 | Include      | `include`                                            |                                                   |
 | Error        | `raise_error(ErrorClass, "message")`                 | Defaults to Exception                             |
 | Stdout/err   | `output("message").to_stdout`                        | Captures `$stdout`/`$stderr`; `to_*` is necessary |

- `dynamic_value` can be anything, like `instance.attribute` or `Class.count`

#### Stdout handling

As of Apr/2025, it seems that there is no way to capture stdout without assertions; therefore, replace `$stdout` if needed:

```rb
original_stdout, $stdout = $stdout, StringIO.new
expect(foo).to bar
puts $stdout.string
$stdout = original_stdout
```

If one wants to optionally test stdout, use a pattern like this:

```rb
expected_output ||= //

expect {
  result = subject.execute
  expect(result).to include("bar")
}.to output(expected_stdout).to_stdout
```

### Matching arguments

See [reference](https://relishapp.com/rspec/rspec-mocks/docs/setting-constraints/matching-arguments).

| Argument                   | Methods (`with`)     | Notes                                     |
| -------------------------- | -------------------- | ----------------------------------------- |
| Regexp/===                 | `/expr/`             | Anything supporting case equality (`===`) |
| Instance of specific class | `instance_of(klazz)` |                                           |
| Anything                   | `anything()`         | Any positional argument                   |

### Receive counts

See [reference](https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/setting-constraints/receive-counts).

- `once`, `twice`, `exactly(n).times`
- `at_least(:once)`, `at_least(:twice)`, `at_least(n).times`
- `at_most(:once)`, `at_most(:twice)`, `at_most(n).times`

### Responses

See [reference](https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/configuring-responses).

- `and_return`
- `and_raise`
- `and_throw`
- `and_yield`
- `and_call_original` : call the original method after the expectation (proxy)
- `and_wrap_original { |meth, *args, &block| }`

#### Conditional stubbing

```ruby
allow(repository).to receive(:remote).with('origin').and_return('git@github.com:saveriomiroddi/goby-dev')
allow(repository).to receive(:remote).with('upstream').and_return('git@github.com:goby-lang/goby')

# Better for modifying the way the method is called, rather than selective stubbing
#
allow(API).to receive(:solve_for).and_wrap_original { |m, *args| m.call(*args).first(5) }
```

In order to disable a method, just `allow` it:

```ruby
allow_any_instance_of(MyModule::MyClass).to receive(:my_method)
```

Return different values for multiple invocations:

```ruby
# Different return values: 1, then 2, then 3.
#
expect_any_instance_of(AWSInfo).to receive(:burst_balance).and_return(1, 2, 3)

# Different params (see :ordered).
# allow_any_instance_of does not support/need this.
#
expect(collaborator_1).to receive(:step_1).ordered
expect(collaborator_2).to receive(:step_2).ordered
```

Arbitrary handling (references: [README](https://github.com/rspec/rspec-mocks#arbitrary-handling) and [Rubydoc](https://rubydoc.info/gems/rspec-mocks#arbitrary-handling)):

```ruby
# WATCH OUT! When using :expect_any_instance_of, the first arg is the instance.
# If a `with()` expectation is added, it's still verified.
# Use double() to stub/mock instances.
#
mock_instance = double(:label)
expect(mock_instance).to receive(:msg) do |instance, *args|
  expect(args.size).to eq(7)
end
```

## Testing modules

```ruby
# :described_class is not available for modules
let(:helper) { Class.new { extend ModuleToTest } }
```

## Transactions

```rb
describe Klazz do
  # Make all the examples non-transactional; required for examples requiring a COMMIT
  self.use_transactional_tests = false
end

describe Klazz do
  # Enable per-example
  uses_transaction "foo"
  it "foo" do
    # data changed here will no be rolled back, so this must be handled.
  end
end
```

Data changes is a PITA (see above), and the same hold for example groups that create/resets data.

Manual rollback (ie. manually updating the data to what it was before) is another PITA, so the simplest (dev-wise) solution is to reset the data before each UT:

```rb
describe Klazz, "non-transactional examples" do
  self.use_transactional_tests = false

  before do
    reset_data

    # logic that would regularly be located in `before :all` must be moved in the `before (:each)`.
  end

  # `let` directives are compatible, even if in nested example groups
  let! :my_instance { ... }

  it "foo" { ... }

  describe NestedExampleGroup do
    let! :my_instance do
      # `let` invocations are compatible with this approach, even if in nested example groups.
    end

    before do
      # nested `before`s are fine; see note above for `before :all`.
    end
  end
end
```

## Customizations

### Custom matcher

Implementation via subclass:

```rb
class LooksLike < String
  def failure_message
    "#{@me} different from #{@another}"
  end

  def matches?(another)
    @me      = some_operation(self)
    @another = some_operation(another)

    @me == @another
  end
end

def look_like(string)
  LooksLike.new(string)
end

expect("abcd").to look_like("abcd")
```

Implementation via module:

```rb
module ListMatcher
  extend RSpec::Matchers::DSL

  matcher :match_list do |expected_list|
    match do |actual_list|
      expected_list = ListParser.parse(expected_list).sort
      actual_list = ListParser.parse(actual_list).sort

      actual_list == expected_list
    end
  end
end

# Put in the example group
include ListMatcher

expect(actual_list).to match_list(expected_list)
```

### Custom expectation

```rb
module NewExpectation
  def expect_instance_list(instance, source_name)
    list = instance.reload.attributes["#{source_name}_list"]

    expect(list)
  end
end

# Put in the example group
include NewExpectation

expect_instance_list(instance, "product").to eql("foo")
```

## Useful stuff

Stub `Kernel#sleep`:

```rb
# Stub the enclosing class, in this case, assume the subject.
expect(subject).to receive(sleep).with(time)
```