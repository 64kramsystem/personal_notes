# Ruby

- [Ruby](#ruby)
  - [General syntax/concepts](#general-syntaxconcepts)
    - [Variables](#variables)
    - [Array literals](#array-literals)
    - [Equality/inclusion testing](#equalityinclusion-testing)
    - [Block-oriented processing methods](#block-oriented-processing-methods)
    - [Operators](#operators)
      - [Ampersand prefix operator `&<object>`](#ampersand-prefix-operator-object)
    - [Heredoc](#heredoc)
    - [Data type conversions](#data-type-conversions)
    - [Collections destructuring in blocks](#collections-destructuring-in-blocks)
    - [Modules](#modules)
    - [Refinements](#refinements)
    - [Call stack metadata](#call-stack-metadata)
    - [Access modifiers](#access-modifiers)
    - [Regular expression APIs/notes](#regular-expression-apisnotes)
    - [Weak references](#weak-references)
  - [Special variables/Built-in constants/Other metadata](#special-variablesbuilt-in-constantsother-metadata)
    - [Verbose mode (enable warnings)](#verbose-mode-enable-warnings)
  - [Strings](#strings)
  - [Classes/Metaprogramming](#classesmetaprogramming)
    - [Dynamic class instantiation](#dynamic-class-instantiation)
    - [Reflection](#reflection)
    - [Simulate Java annotations!!!](#simulate-java-annotations)
  - [Collections](#collections)
    - [Array](#array)
      - [Binary search operations](#binary-search-operations)
    - [Enumerable](#enumerable)
    - [Hash](#hash)
    - [Useful Collection Operations](#useful-collection-operations)
    - [Not present: Linked list, Heap](#not-present-linked-list-heap)
    - [SortedSet](#sortedset)
  - [Basic I/O](#basic-io)
  - [Handling (execute) processes](#handling-execute-processes)
    - [Basic handling, via `IO.popen`](#basic-handling-via-iopopen)
    - [Using `Open3`](#using-open3)
    - [Live execution, via `Kernel#system`](#live-execution-via-kernelsystem)
    - [Backticks](#backticks)
    - [Definitive way of capturing Ctrl+C (trap signals)](#definitive-way-of-capturing-ctrlc-trap-signals)
    - [Process tree handling](#process-tree-handling)
  - [YARD documentation (+Solargraph)](#yard-documentation-solargraph)
  - [Misc](#misc)
    - [Terminal](#terminal)
    - [Poor man's deep\_dup (deep duplicate objects)](#poor-mans-deep_dup-deep-duplicate-objects)
    - [Debug a live process (print current stacktrace)](#debug-a-live-process-print-current-stacktrace)

## General syntax/concepts

### Variables

Instance variable metaprogramming:

```ruby
instance_variable_set(:my_var, 2)
instance_variable_get(:my_var)    # 2

# Test and undefine an instance variable; undefine raises an error if the variable is not defined.
#
remove_instance_variable :@my_var if instance_variable_defined?(:@my_var)
```

### Array literals

```ruby
%i(foo bar)           # array of symbols
```

### Equality/inclusion testing

Equality methods:

- `equal?` : never touch it
- `==`     : for unspecific comparison; when required, performs type conversion (eg. Numeric: 1.0 == 1). Array#include? uses this.
- `eql?`   : for specific comparison, without type conversion. Hash uses this (and #hash).
- `===`    : for case/when

Inclusion testing:

```ruby
# Override in order for an object to be found by Array#include:
#
Object.==()

# Override in order for an object to be used in Hash:
#
Object.eql?()
Object.hash()
```

### Block-oriented processing methods

```ruby
# `#tap` allows modifying the object while returning the object
#
Customer.new.tap { |c| c.attribute = value; c.save! }

# `#then` (or `#yield_self`) returns instead the result of the block.
# Arguably, it's preferrable for a  pipeline - see https://zverok.github.io/blog/2018-01-24-yield_self.html:
#
url
  .then { |url| Faraday.get(url) }.body
  .then { |response| JSON.parse(response) }
  .dig('object', 'id')
  .then { |id| id || '<undefined>' }
  .then { |id| "server:#{id}" }

# opposed to:

"server:" + (JSON.parse(Faraday.get(construct_url)).dig('object', 'id') || '<undefined>')
```

### Operators

The closed range operator is inconsistent:

- `(...7).last`         -> `7`   (includes the last)
- `(6...7).map(&:to_i)` -> `[6]` (doesn't include the last)

- `(1..)` and `(1...)` have the same elements (both include Float::INFINITY), but they're not equal

#### Ampersand prefix operator `&<object>`

`&<object>` is turned to  `object.to_proc`.

```ruby
myblock = ->(i) { 2 * i }
def mymethod(i); 2 * i; end

# &<block> => intuitive invocation
#
[1, 2, 3].map(&myblock)
[1, 2, 3].map { |i| 2 * i }

# Pass a method as map() block parameter
#
# &method(:<method>) => { |arg| <method>(arg) }
#
# method(:<method>) doesn't take any arguments, so it can't be turned into a `<method>(arg1, arg2...)` invocation.
#
[1, 2, 3].map(&method(:mymethod))
[1, 2, 3].map { |i| mymethod(i) }

# :<method> => { |arg, *args| arg.<method>(*args) }
#
# Fits with :inject, but it's not clear what else could it apply to.
#
[1, 2, 3].inject(:+)
[1, 2, 3].inject { |i, j| i.+(j) }
```

### Heredoc

The closing token must be the only one on a line; preceding spaces are ignored (and a newline is appended).

With minus, allows delimiter to be anywhere:

```ruby
<<-EOF
  text
    EOF
```

With tilde, also allows delimiter to be anywhere:

```ruby
<<~EOF
  test
  EOF
```

Special syntaxes:

```ruby
# Pass to a method
#
StringIO.new(<<~HEADER, "a")
  myheader
HEADER

# Apply a condition (for cases where parentheses are required)
#
{
  expectations: (<<~RUST if condition_matching)
    cond
  RUST
}
```

### Data type conversions

- `pack(directive)`   : string/bytes array -> binary string
- `unpack1(directive)` : binary string -> int ("1" avoids invoking `:first`)

The star suffix (`*`) is for multiple chars. Reference (also for unpack): https://apidock.com/ruby/Array/pack

Directives:

|  type  | directive |          definition           |                      input                      |       output       |
| :----: | :-------: | :---------------------------: | :---------------------------------------------: | :----------------: |
| string |    `B`    |       bits, big endian        |         ['1111000011110000' ].pack 'B*'         |     "\xF0\xF0      |
| string |    `H`    |        hex, big endian        |         ['A0FF'             ].pack 'H*'         |     "\xA0\xFF"     |
|  int   |    `C`    |      8-bit unsigned char      |         [0xFF               ].pack 'C*'         |       "\xFF"       |
|  int   |    `Q`    | native endian qword, unsigned | "\x01\x02\x03\x04\x05\x06\x07\x08".unpack1 'Q*' | 0x0807060504030201 |
|  int   |    `L`    | native endian dword, unsigned | "\x01\x02\x03\x04"                .unpack1 'L*' |     0x04030201     |

Note that:

- "\x07" is represented as "\a"
- unpack1 output is an int, so `output` is processed

Generic conversion examples (consider constraints, where specified, e.g. "codepoint"):

|       input       |          output          | code                                                   | notes                                    |
| :---------------: | :----------------------: | ------------------------------------------------------ | ---------------------------------------- |
|       "0Aa"       |         "304161"         | `each_byte.map { [b] "%02X" % b }.join`                |                                          |
|       "0Aa"       |         "304161"         | `bytes_arr.pack('c*').unpack1("H*")`                   |                                          |
|    [0x77,0x0F]    |    "0111011100001111"    | `pack("C*").unpack1("B*")`                             | padded big endian 8-bits bit string      |
|   [0,0xF,0,0xF]   | ["0111011100001111"] * 2 | `each_slice(2).map { [a] a.pack("C*").unpack1("B*") }` | (same as previous, but in 16-bit chunks) |
|       "0Aa"       |        [48,65,97]        | `codepoints`                                           | valid codepoints                         |
|   [48,65,0xFF]    |         "0A\xFF"         | `map(&:chr).join`                                      | non-ASCII chars are escaped              |
|  [48,0x80,0x81]   |     "0\u0080\u0081"      | `pack('U*')`                                           | invalid codepoints are escaped           |
| [1,2,3,4,5,6,7,8] |    "0807060504030201"    | `"%016x" % @input.pack("c*").unpack1("Q")`             | native endian unsigned qword bytes       |
|        "A"        |            65            | `ord`                                                  | valid codepoint                          |
|        65         |           chr            | `A`                                                    | non-ASCII chars are escaped              |
|      "FFFF"       |          65535           | `hex` / `to_i(16)`                                     |                                          |
|       4095        |          "fff"           | `to_s(16)`                                             | no padding!!                             |
|       "123"       |           123            | `Integer(123)`                                         | (1) raise an error if invalid            |
|       123.2       |           123            | `Integer(123.2)`                                       | (2) truncates                            |

(brackets are used as block vars due to an issue with the VSC plugin)

See https://idiosyncratic-ruby.com/49-what-the-format.html for `printf`-style formatting.

Printf `%#X`prefixes with `0X`, which is ugly.

There is a gem, `bitarray`, however, it's not so much better than just using a string (bits = `"0" * 8`) for generic work, as it doesn't even support bit operations like `OR`.

Another gem, `bitset`, is very functional, but it's old, and needs to be checked.

### Collections destructuring in blocks

```ruby
# WATCH OUT!! Destructure a hash in blocks, doesn't work anymore in Ruby 3.
# See https://bugs.ruby-lang.org/issues/14183#note-114.
#
[{a: 1}].each { |a:, **| }
#
# It does work in functions, but the hash needs to be destructured by the caller.
#
def myfn(a:); end
myfn(**{a: 1})
```

### Modules

```ruby
module MyModule
  # This is defined at module level; it won't be available to instance methods, only to class methods.
  # The attr_accessor is a convenience.
  #
  @per_class = Mutex.new
  class << self; attr_accessor :per_class; end

  # Invocation: MyModule.my_class_method.
  #
  def self.my_class_method
    # inside here, `self` is the MyModule class
    puts per_class
  end

  # Invocation: MyClass.new.my_instance_method()
  #
  # Each MyClass instance will have their own @per_instance.
  #
  def my_instance_method
    # inside here, `self` is the includee
    @per_instance = Mutex.new unless instance_variable_defined?(:@per_instance)
    puts @per_instance.object_id
  end
end

module MyModule
  # Add invocation: MyModule.my_method()
  #
  extend self

  def my_method; end
end

class MyClass; include MyModule; end

# This is the solid way of including other modules in the included class.
# putting the inclusion(s) directly inside the outer module works, however, the modules won't be formally
# included in the class, which can be expected in a workflow.
#
module ModuleIncludingModules
  def self.included(klass)
    klass.class_eval do
      include Bar
    end
  end
end
```

If one wants to avoid method clashing between the module and including class, just define a class method:

```rb
module MyModule
  def meth; MyModule.module_meth; end
  def self.module_meth; puts "module_meth"; end
end

include MyModule
meth
```

### Refinements

```ruby
module NilEmptyHelper
  refine NilClass do
    def empty?
      true
    end
  end
end

describe NilEmptyHelper do
  using NilEmptyHelper

  it "should make nil#empty? return true" do
    expect(nil.empty?).to be(true)
  end
end
```

### Call stack metadata

```rb
__method__ # enclosing method (static; just looks at the definition)
__caller__ # enclosing method (dynamic; considers aliasing)
```

### Access modifiers

```ruby
class Klazz
  CONST = 123; CONST2 = 234
  private_constant :CONST, :CONST2 # can't be inlined or put before the const definition
end
```

### Regular expression APIs/notes

Flags:

- `/m` : match newlines with `.`; WATCH OUT! This is like both Perl `sm`
- `/x` : ignore whitespace and comments
- `/u`, `/e`, `/s`, `/n` : force encodings; don't mistake `/s` for the Perl one!

```ruby
Regexp.union(*strings_or_regexes)         # Generates a `/a|b|.../` Regexp instance; String expressions are escaped
```

### Weak references

```ruby
def cached_load(url)
  @cache ||= {}

  # It's crucial to call __getobj__; if we don't, we have a couple of problems if the value has
  # been collected:
  # - the reference returned is ambivalent: it's not nil (being WeakRef), so it can't be tested
  #   using ||, but will raise error when using .nil?
  # - if we rescue while testing the value, we still can't guarantee that the object hasn't been
  #   garbage collected between the test and the actual usage (same goes for using the API
  #   :weakref_alive?)
  # - we could theoretically test and use the value and rescue at the same time (maybe something
  #   like "value.to_s rescue", but that would be very convoluted
  # - on top of all this awkwardness, we need to call something on the instance, in order to
  #   check if it's alive or not, so the cache, in a way, can't be oblivious to the interface of
  #   the object.
  #
  if @cache.has_key?(url)
    page_content = @cache[url].__getobj__ rescue nil
  end

  if page_content.nil?
    page_content = GenericHelper.do_retry { open(url).read }
    @cache[url] = WeakRef.new(page_content)
  end

  page_content
end
```

## Special variables/Built-in constants/Other metadata

Updated up to: https://ruby-doc.org/stdlib-2.3.0/libdoc/English/rdoc/English.html

Entries marked with `ENG` need to `require 'English'` in order for the english version to be usable.

```ruby
$:   $LOAD_PATH                     # load path
$0   $PROGRAM_NAME                  # the name of the ruby script file
$*   $ARGV                          # [ENG] the command line arguments
$?   $CHILD_STATUS                  # [ENG] exit status of last executed child process
$$   $PID, $PROCESS_ID              # [ENG] interpreter's process ID

$~   $LAST_MATCH_INFO               # [ENG] MatchData instance for the last regexp match
$<n>                                # nth subexpression in the last match (same as $~[n])
$&   $MATCH                         # [ENG] string last matched by regexp
$+   $LAST_PAREN_MATCH              # [ENG] last match from the previous successful pattern match
$`   $PREMATCH                      # [ENG] string before the actual matched string of the previous successful pattern match.
$'   $POSTMATCH                     # [ENG] string after the actual matched string of the previous successful pattern match.

$!   $ERROR_INFO                    # [ENG] last error message
$@   $ERROR_POSITION                # [ENG] last error backtrace

$_   $LAST_READ_LINE                # [ENG] string last read by gets
$.   $NR, $INPUT_LINE_NUMBER        # [ENG] line number last read by interpreter

$,  $OFS, $OUTPUT_FIELD_SEPARATOR   # [ENG]
$;  $FS, $FIELD_SEPARATOR           # [ENG]
$/  $RS, $INPUT_RECORD_SEPARATOR    # [ENG] input record separator
$\  $ORS, $OUTPUT_RECORD_SEPARATOR  # [ENG] output record separator

$>  $DEFAULT_OUTPUT                 # [ENG]
$<  $DEFAULT_INPUT                  # [ENG]

$=  $IGNORECASE                     # [ENG] obsolete: case-insensitivity flag
```

```ruby
RUBY_PATCHLEVEL                     # patch level (Integer)
RUBY_VERSION                        # e.g. "2.7.0"
__dir__                             # directory of current file
```

```rb
Gem.ruby                            # binary/executable path
```

```rb
require 'rbconfig'
RbConfig::CONFIG["CFLAGS"]
RbConfig::CONFIG["CC"]              # compiler (+ some other info)
```

### Verbose mode (enable warnings)

Set `$VERBOSE = true`.

## Strings

```rb
str.b                               # ASCII-8 (binary) copy of the string
str.slice!(interval)                # removes and return the sliced substring, so it can be used in expressions
str.prepend(prefix)
```

## Classes/Metaprogramming

Autoload:

```ruby
MyModule.autoload :MyClass, '/path/to/file.rb' # MyModule is optional
```

### Dynamic class instantiation

Three ways to dynamically define a class:

```ruby
# `superclass` is optional.
#
new_class = Class.new(superclass) do
  attr_accessor my_attr
end

# The parameters passed enable source debugging.
# If the defined class needs to be in the root scope, append `::` to the class name be sure!
#
ActiveTrigger.class_eval <<-'RUBY', __FILE__, __LINE__ + 1
  class Application < Rails::Application; end
'RUBY'

ActiveTrigger.const_set :Application, Class.new(Rails::Application) do; end
```

### Reflection

Module methods are defined as instance methods, so they are accessible via `instance_*` APIs.

```rb
module MyModule
  def mymethod(foo, bar=true, baz: false, qux:); end
end

# Get the method parameters:
#
# - :req:    required positional
# - :bar:    optional positional
# - :keyreq: required keyword
# - :key:    optional keyword
#
# Note that the ordering of the keyword args doesn't necessarily match the signature's.
#
MyModule.instance_method(:mymethod).parameters
# => [[:req, :foo], [:opt, :bar], [:keyreq, :qux], [:key, :baz]]
```

Source location:

```rb
instance.method(:meth).source_location
# Use Object as parent of root objects
mod.const_source_location(:Klass)      # String accepted
Klass.const_source_location(:CONST)    # String accepted
```

### Simulate Java annotations!!!

Makes available a `roles(*roles)` method, which puts the following method in the `@access_list` instance variable of the class including `Authorization`:

```ruby
module Authorization
  def self.included(base)
    base.extend(Authorization::ClassMethods)
  end

  module ClassMethods
    def self.extended(base)
      # @access_list is not shared between classes in the hierarchy!
      # it's NOT a class/instance variable, but a "class level" instance variable
      # it will be included in base.instance_variables
      base.class_eval <<-END
        @access_list = {}
      END
    end

    def roles(*roles)
      @roles = roles
    end

    def method_added(method)
      if @roles
        @access_list[method] = @roles
        @roles = nil
      end
    end
  end
end
```

## Collections

```ruby
collection.none?(pattern)               # supports String, Regex and Proc. DON'T USE FOR HASHES, it's confusing
collection.sort_by { |v| }              # Hash returns a [K, V] array
```

### Array

```ruby
arr = [0, 1, 2]
Array.new(size) { |i| fx(i) }           # creates an array of the given size, where each element is the result of the block

arr.fill(nil, arr.size...@newlen)          # resize/extend (destructive) in arguably expressive form; returns the array; !!! watch out the `...` syntax !!!
arr.fill(arr.size, @add_len) { |i| fx(i) } # other form, for non-copy types
arr[5] ||= nil                             # other resize/extend, in arguably less expressive form

arr.delete(obj)                         # delete an object matching via `==`; returns self
arr.delete_at(i)                        # delete an object at index `i`; returns the value deleted, or nil if no deletion

arr.delete_if { |e| fx(e) }                    # WATCH OUT! The deleted items are lost (self is returned)
deleted, arr = arr.partition { |e| del_fx(e) } # useful way to perform a "splitting" delete_if

arr.slice!(interval)                    # removes and returns the slice; e.g. `array.slice(3..)`

[ [[1, 2], [3, 4]], [[5]] ].flatten(1)  # flatten only N levels => [[1, 2], [3, 4], [5]]
```

Transpose a matrix (array of arrays) (intuitively, converts from array of the rows to array of the columns):

```rb
# Requires all the arrays to be the same size:
#
matrix.transpose

# Simplest solution when the arrays are not the same size.
# Yields nils, so it needs to be compacted, if required.
#
Array.new(matrix.map(&:size).max) { |i| matrix.map { |e| e[i] } }
```

#### Binary search operations

```rb
# Binary search. WATCH OUT!! `n` must be the right operand in the comparison!
#
[2, 3, 4, 5].bsearch_index { |n| 2 <=> n } # 0
[2.0, 3.0, 4.0].bsearch { |n| 2 <=> n }    # 2.0

# Binary insertion
#
(0..10).to_a.bsearch_index { |n| n > 5 } # 6 => use this index to insert
```

### Enumerable

```rb
# WATCH OUT!! Don't confuse `.max { }` with `.max_by {}` (or `.map {}.max`) !!!
#
[[1, 2], [3, 4]].max { |a, b| a <=> b }       # the block is for comparing elements
[[1, 2], [3, 4]].max_by { |a, b| b / a.to_f } # the block is for providing a mapped value

enu.count { |a| a.odd? }                # `.count {}` works as intuitive
enu.filter_map { |n| n**2 if n.even? }  # yay! (falsey values are discarded)
enu.each_cons(n)                        # each overlapping subarray of `n` items; WATCH OUT!! if an array has size < window, it's *not* included
enu.each_slice(n)                       # each non overlapping subarray of `n` items; last windows, if smaller, *is* included

enum.each_with_index.drop(1).each {}    # skip the first N values of an iterator; WATCH OUT! not lazy (use lazy() if needed)
```

### Hash

```ruby
Hash.new(default_value)                   # sets a default value when a key is not found; uses always the same instance, though
Hash.new {|h, k| h[k] = default_value}    # same as previous, but generates new instances
Hash.new { |h, k| h[k] = Hash.new(&h.default_proc) } # same as previous, but supports default nested hashes!

[[:a, 1], [:b, 2]].to_h                   # => {a: 1, b: 2} # !!! convert an array to hash !!!
Hash[:a, 1, :b, 2]                        # => {a: 1, b: 2} # !!! convert an array to hash !!!
Hash[*flat_array]                         # => same as previous

hash.symbolize_keys
hash.transform_keys(&block)
{a: 1, b: 2, c: 3}.slice(:a, :b)          # => {a: 1, b: 2} # doesn't modify the source hash
hash.values_at(:key1, :key2)
hash.key?(:key)                           # check inclusion
hash&.store(:key, val)                    # Hash#[]= alias, convenient for safe navigation
hash.clone                                # !!! shallow copy !!!
hash.delete_if { |k, v| fx() }            # destructive; returns hash (not the deleted values)
hash.merge!(hash) {|_, v| fx(v)}          # !!! transform the values of a hash !!! non-destructive (without `!`) version also works
hash = hash.map {|k, v| [fx(k), v] }.to_h # !!! transform the keys of a hash !!!
```

Keys are compared using `#hash` and `#eql?`.

ActiveSupport additions:

```ruby
{a: 1, b: 2}.extract!(:a)                 # => {a: 1} # destructive :slice
{a: 1, b: 2, c: 3}.except(:a, :b)         # => {c: 3}
```

### Useful Collection Operations

Perform COUNT/GROUP BY on an array (Ruby 2.7 implements `#tally`):

```ruby
[a:, a:, :b, :c, :c: :c]
  .group_by(&:itself)        # => {a: [:a, :a], b: [:b], c: [:c, :c, :c]}
  .transform_values(&:count) # => {a: 2, b: 1, c: 3}
```

### Not present: Linked list, Heap

Ruby has no linked lists!

- For linked lists with unique entries, Set/Hash can be used, although they don't have an efficient tail access
- SortedSet can be used where possible, and it has head/tail efficient access
- Barring a manual implementation/gem, only array can be used (`Queue` is a simple stack)

Heap is also not present.

### SortedSet

If the elements of a collection are unique, this can be used to implement an efficient sorted linked list.

```rb
require 'set'
set = SortedSet.new([2, 1, 5, 6, 10, 24, 3])

set.include?(33)       # false
set.first              # 1; there is not last! reverse_each().first is not efficient
set.min, set.max       # can use these as first/last

set << 33
set.delete(33)
```

If the elements are not unique, one can create a struct which includes the value and its count (must implement `#hash` and `#eql?`)

## Basic I/O

Files:

```rb
io_obj.readline                         # includes the newline!
io_obj.readchar                         # reads one char; raises EOFError at the EOF
```

## Handling (execute) processes

The sections assume `require 'English'`, which aliases `$?` (`Process::Status`) to `$CHILD_STATUS`.

In order to execute under a certain directory, use `Dir.chdir { ... }`; alternatively, `open3`-like methods can set the `PWD` env var.

### Basic handling, via `IO.popen`

The stdout content is captured, so this is not appropriate for long operations.
Stderr content is instead printed immediately, unless `err: [:child, :out]` is passed, which causes it to be mixed with the stdout.

Invalid commands cause an error (`Errno::ENOENT`), while valid ones with failure exit status need to be detected (see below).

```ruby
stdout_content = IO.popen('sed "s/foo/bar/"', 'r+') do |io|
  io.puts "foo"
  io.close_write
  io.read # return value of the block
end

if !$CHILD_STATUS.success?
  exit $CHILD_STATUS.exitstatus
else
  print stdout_content # "bar\n"
end
```

A non-block form exists, which returns the `IO` object.

### Using `Open3`

The output is not printed as well, so the same considerations as `IO.popen` apply.

`env_vars` must be strings, not symbols.

Options (partial):

- `:stdin_data`: send data to stdin

```ruby
Open3.popen3([env_vars,] command [, options]) do |stdin, stdout, stderr, wait_thread|
  stdout_content, stderr_content = stdout.read, stderr.read

  $stdout.puts(stdout_content) if stdout_content != ''
  $stderr.puts(stderr_content) if stderr_content != ''

  # `wait_thread.value` waits until completion
  #
  raise "Error (exit status: #{wait_thread.value.exitstatus})" if !wait_thread.value.success?

  stdout_content
end
```

A convenient way to execute and capture the output is `capture3`:

```ruby
stdout, stderr, status = Open3.capture3([env_vars,] command [, options])

$stdout.puts stdout
$stderr.puts stderr

exit status.exitstatus if !status.success?
```

### Live execution, via `Kernel#system`

This way, stdout/stderr content is printed straight away; interaction (input) is supported.

The exit status can be obtained directly via return value, or via `$CHILD_STATUS`.

```ruby
success = system(">&2 echo abc; sleep 2; false", exception: true) # default: false

# Use `expection: true` in order to raise an exception instead.
#
exit $CHILD_STATUS.exitstatus if !success
```

In order to pass env variables:

```ruby
# The hash key/values must be all strings.
#
system({'FOO' => 'BAR'}, 'echo $FOO') # => BAR
```

### Backticks

Backticks behave like `IO.popen`: The stdout content is captured, stderr content is instead printed immediately.

Use `$CHILD_STATUS` (ENG) to capture errors.

### Definitive way of capturing Ctrl+C (trap signals)

- The existing stack trace will be the included in the exception raised;
- Rescue(Interrupt) worked as well, in a local test;
- For any given signal, only one trap statement is possible.

See https://stackoverflow.com/questions/18662273/rescue-capture-ctrl-break.

```ruby
trap("INT") { cleanup(); raise Interrupt }

class MyClass
  def initialize
    @myhash = {a: 1}

    trap('SIGHUP') do
      @myhash[:b] = 2
      puts "Updated configuration: #{@myhash}"
    end
  end
end
```

Interrupt can still be regularly caught in some contexts, e.g. on `STDIN.gets`.

### Process tree handling

Kill a process tree:

```rb
_, _, wait_thr = Open3.popen2e("smplayer #{smplayer_actions_option}", pgroup: true)
# ...
pgid = Process.getpgid(wait_thr.pid)
Process.kill '-SIGINT', pgid
```

## YARD documentation (+Solargraph)

Standard tags:

```rb
# @return [String]
#
# Other types: `nil`, `self`, `void`, `Boolean`.
#
def meth; 'str'; end

# @return [Integer]
attr_reader :number

# @param val [Integer, String] Optional param description, with a
#   second line.
#
def meth(val); val * 2; end

# @yieldparam [String]
def meth; yield 'str'; end
```

Solargraph extensions:

```rb
# @type [String]
str = method_returning_string
```

[YARD](https://www.rubydoc.info/gems/yard/file/docs/Tags.md) and [Solargraph](https://solargraph.org/guides/yard) documentations.

## Misc

### Terminal

Check if I/O is terminal:

```rb
STDIN.istty
```

### Poor man's deep_dup (deep duplicate objects)

```ruby
def deep_dup(source)
  case source
  when Hash
    source.each_with_object({}) do |(key, value), destination|
      destination[key] = deep_dup(value)
    end
  when Array
    source.map do |entry|
      deep_dup(entry)
    end
  else
    source.dup
  end
end
```

### Debug a live process (print current stacktrace)

Redirect stdout/err to the current terminal (the program debugged has its own).  
The $stdout reassignment to file strategy caused SIGABRT when tested.

When the gdb command `p` is invoked (or anything that returns a value), the output is assigned to the `$1` (and so on.) convenience variable.  
This is essentially a short form for invoking `--eval-command="set \$tty= \"$(tty)\""` and using `$tty` instead of `$1`.

The open() and close() calls can also be, respectively, sequential. They way it works is that when calling `open()`, "The file descriptor returned by a successful call will be the lowest-numbered file descriptor not currently open for the process.".  
1 = O_WRONLY (write only) - read/write is not really needed); setting O_RDONLY, as expected, causes `Bad file descriptor @ io_writev - <STDOUT>...`

```sh
sudo gdb --eval-command="p \"$(tty)\"" -p $(pgrep -n ruby)

call close(1)
call open($1, 1, 0)
call close(2)
call open($1, 1, 0)

call rb_backtrace()

# Not working (changes fd 8, which is $stdout)
```
