# Ruby

- [Ruby](#ruby)
  - [General syntax/concepts](#general-syntaxconcepts)
    - [Variables](#variables)
    - [Array literals](#array-literals)
    - [Equality/inclusion testing](#equalityinclusion-testing)
    - [Block-oriented processing methods](#block-oriented-processing-methods)
    - [Ampersand prefix operator `&<object>`](#ampersand-prefix-operator-object)
    - [Heredoc](#heredoc)
    - [Data type conversions](#data-type-conversions)
    - [Collections destructuring in blocks](#collections-destructuring-in-blocks)
    - [Modules](#modules)
    - [Refinements](#refinements)
  - [Metadata (constants etc.) modifiers](#metadata-constants-etc-modifiers)
    - [Regular expression APIs/notes](#regular-expression-apisnotes)
    - [Weak references](#weak-references)
  - [Special variables/Built-in constants/Other metadata](#special-variablesbuilt-in-constantsother-metadata)
    - [Verbose mode (enable warnings)](#verbose-mode-enable-warnings)
  - [Strings](#strings)
  - [Classes/Metaprogramming](#classesmetaprogramming)
    - [Dynamic class instantiation](#dynamic-class-instantiation)
    - [Reflection](#reflection)
      - [Pass a method as map block parameter](#pass-a-method-as-map-block-parameter)
    - [Simulate Java annotations!!!](#simulate-java-annotations)
  - [Concurrency](#concurrency)
    - [Mutex](#mutex)
    - [Thread-safe data structures](#thread-safe-data-structures)
    - [IO.pipe for IPC](#iopipe-for-ipc)
  - [Collections](#collections)
    - [Array](#array)
    - [Iterable APIs/Useful operations](#iterable-apisuseful-operations)
    - [Hash](#hash)
    - [Enumerable](#enumerable)
  - [Basic I/O](#basic-io)
  - [Handling processes](#handling-processes)
    - [Basic handling, via `IO.popen`](#basic-handling-via-iopopen)
    - [Using `IO.popen3`](#using-iopopen3)
    - [Live execution, via `Kernel#system`](#live-execution-via-kernelsystem)
    - [Backticks](#backticks)
    - [Definitive way of capturing Ctrl+C (trap signals)](#definitive-way-of-capturing-ctrlc-trap-signals)
  - [YARD documentation (+Solargraph)](#yard-documentation-solargraph)
  - [Misc](#misc)
    - [Terminal](#terminal)
    - [Poor man's deep_dup (deep duplicate objects)](#poor-mans-deep_dup-deep-duplicate-objects)
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

### Ampersand prefix operator `&<object>`

`&<object>` is turned to  `object.to_proc`.

```ruby
myblock = ->(i) { 2 * i }
def mymethod(i); 2 * i; end

# &<block> => intuitive invocation
#
[1, 2, 3].map(&myblock)
[1, 2, 3].map { |i| 2 * i }

# &method(:<method>) => { |arg| <method>(arg) }
#
# method(:<method>) doesn't take any arguments, so it can't be turned into a `<method>(arg1, arg2...)` invocation.
#
[1, 2, 3].map(&method(:mymethod))
[1, 2, 3].map { |i| mymethod(i) }

# &:<method> => { |obj, *args| obj.send(<method>, *args) }
#
# Fits with :inject, but it's not clear what else could it apply to.
#
[1, 2, 3].inject(:+)
[1, 2, 3].inject { |i, j| i.send(:+), j }
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

Common conversions. Everything is a string; "int" represents a decimal.

Generic methods:

```ruby
# int -> a numerical string in the given base
# IMPORTANT! There's no left padding.
#
int.to_s(base)

# numerical string in the given base -> int
#
numerical_str.to_i(base)

# Array of bytes to binary string; the directive is the base of the string; some directives:
#
# - `B*`: binary
# - `H*`: hex
#
# Reference (also for unpack): https://apidock.com/ruby/Array/pack
#
array.pack(directive)

# Binary string to value
#
# - `Q`/`q`: native endian qword, unsigned/signed
# - `L`    : native endian dword, unsigned
#
# `unpack1` avoids invoking `:first`.
#
binstr.unpack1(directive)
```

Specific conversions:

```ruby
bytes.each_byte.map { |b| "%02X" % b }.join   # array of bytes → hex string (see printf link below)
bytes.pack('c*').unpack1("h*")                # ^^
bytes.each_byte.each_slice(2) { |i, j| puts "%016b" % (256 * i + j) } # bytes → binary, padded 16 bits big endian
bytes.each_byte.map(&:to_i)                   # array of bytes → array of ints
arr.map(&:chr).join                           # array of ints → array of bytes

"0x%016x" % bytes[loc, 8].pack("c*").unpack1("Q")  # native endian unsigned qword (array of bytes) → int -> (padded lowcase) hex string

"c".ord                                       # char  → int
int.chr                                       # int   → byte/char (ASCII-8)

hex.hex                                       # hex   → int (same as `to_i(16)`)

str.codepoints                                # like str.bytes, but each entry is a full codepoint
[codepoints].pack('U*')                       # codepoints to string
[bytes].map(&:chr).join                       # print string with printable chars and non-printable as escape sequence, from bytes;
                                              # add :inspect at the end to have the string not automatically converted.
```

See https://idiosyncratic-ruby.com/49-what-the-format.html for `printf`-style formatting.

Printf `%#X`prefixes with `0X`, which is ugly.

There is a gem, `bitarray`, however, it's not so much better than just using a string (bits = `"0" * 8`) for generic work, as it doesn't even support bit operations like `OR`.

Another gem, `bitset`, is very functional, but it's old, and needs to be checked.

### Collections destructuring in blocks

Destructure an array in blocks ("unpacking" refers to arrays):

```ruby
# "key:a, b:1, c:2" !!!
#
{a: {b: 1, c: 2}}.each do |key, b:, c: nil| # default provided for :c
  puts "key:#{key}, b:#{b}, c:#{c}"
end
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
    puts per_class
  end

  # Invocation: MyClass.new.my_instance_method()
  #
  # Each MyClass instance will have their own @per_instance.
  #
  def my_instance_method
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

## Metadata (constants etc.) modifiers

```ruby
class Klazz
  CONST = 123; CONST2 = 234
  private_constant :CONST, :CONST2 # can't be inlined or put before the const definition
end
```

### Regular expression APIs/notes

Flags:

- `/x` : ignore whitespace and comments
- `/s` : don't use it - not needed, and it has a different meaning

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

#### Pass a method as map block parameter

```ruby
# Invokes :mymethod on each of the `enumerable` items.
#
enumerable.map(&method(:mymethod))
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

## Concurrency

### Mutex

```ruby
mutex = Mutex.new
mutex.synchronize { }
```

### Thread-safe data structures

`Queue`s are thread-safe FIFO queues, although very primitive.

It has a `empty?` method, which is useless, since the class doesn't provide an atomic way to atomically check and pop.

```ruby
queue = Queue.new
(0..9).each { |item| queue << item }

threads = 3.times.map do |i|
  Thread.new do
    loop do
      begin
        sleep(rand * 0.1)

        # param `non_block` (false): if false, blocks until an element is available; if true, raises
        # a ThreadError if no elements are available.
        #
        value = queue.pop(true)

        puts "Thread #{i}: #{value}"
      rescue ThreadError
        break
      end
    end
  end
end

threads.each(&:join)
```

### IO.pipe for IPC

IO.pipe(s) can be used for IPC (but also inter-thread), like Golang channels. The string encoding is converted in the write/read process.

IMPORTANT NOTES:

- it's possible to use the write pipe in multiple forks, but must use the pseudo-non-blocking version
  - closing a forked write pipe won't affect another forked write pipe

- it's important to close the endpoints in forked processes, since even if not referenced, they will be in scope!!!

- :read will receive data only if:
  - the writer on the other process has been closed
  - the writer on the same process has been closed
- the pseudo-non-blocking version (:read_nonblock) will work without the two conditions above

```ruby
# PROCESSES + BLOCKING READ

rd, wr = IO.pipe

# fork() returns PID in the parent, and nil in the child, so the first block is the parent, the second, the child
if fork
  wr.close
  puts "Read: #{rd.read.inspect}"
  rd.close
  Process.wait	# waits on the child/writer
else
  rd.close
  wr.write "Message"
  wr.close
end

# THREADS + PSEUDO-NON-BLOCKING READ
# The pattern for pseudo-non-blocking (which blocks) is quite strange.

rd, wr = IO.pipe

reader_thread = Thread.new do
  data_received = \
    begin
      rd.read_nonblock(100)
    rescue IO::WaitReadable, IO::EAGAINWaitReadable
      IO.select([rd])	# blocks until data is availabel
      retry
    end

  puts "Read: #{data_received.inspect}"
end

write_thread = Thread.new do
  wr.write "Message"
  wr.flush	# this is probably a good practice
end

reader_thread.join
write_thread.join
```

## Collections

### Array

```ruby
arr = [0, 1, 2]
Array.new(size) { |i| fx(i) }           # creates an array of the given size, where each element is the result of the block

arr.fill(nil, arr.size...5)             # resize/extend (destructive) in arguably expressive form; returns the array; !!! watch out the `...` syntax !!!
arr[5] ||= nil                          # other resize/extend, in arguably less expressive form

arr.delete(obj)                         # delete an object matching via `==`; returns self
arr.delete_at(i)                        # delete an object at index `i`; returns the value deleted, or nil if no deletion
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

### Iterable APIs/Useful operations

APIs:

```rb
each_slice(size)                        # divide into slices
each_cons(size)                         # sliding windows of size
```

Perform COUNT/GROUP BY on an array; Ruby 2.7 implements #tally:

```ruby
[a:, a:, :b, :c, :c: :c]
  .group_by(&:itself)        # => {a: [:a, :a], b: [:b], c: [:c, :c, :c]}
  .transform_values(&:count) # => {a: 2, b: 1, c: 3}
```

### Hash

```ruby
Hash.new(default_value)                   # sets a default value when a key is not found; uses always the same instance, though
Hash.new {|h, k| h[k] = default_value}    # same as previous, but generates new instances
Hash.new { |h, k| h[k] = Hash.new(&h.default_proc) } # same as previous, but supports default nested hashes!

[[:a, 1], [:b, 2]].to_h                   # => {a: 1, b: 2} # !!! convert an array to hash !!!
Hash[:a, 1, :b, 2]                        # => {a: 1, b: 2} # !!! convert an array to hash !!!
Hash[*flat_array]                         # => same as previous

hash.symbolize_keys                       # non-destructive
{a: 1, b: 2, c: 3}.slice(:a, :b)          # => {a: 1, b: 2} # doesn't modify the source hash
hash.values_at(:key1, :key2)
hash.key?(:key)                           # check inclusion
hash.clone                                # !!! shallow copy !!!
hash.delete_if { |k, v| fx() }            # destructive; returns hash (not the deleted values)
hash.merge!(hash) {|_, v| fx(v)}          # !!! transform the values of a hash !!! non-destructive (without `!`) version also works
hash = hash.map {|k, v| [fx(k), v] }.to_h # !!! transform the keys of a hash !!!
```

ActiveSupport additions:

```ruby
{a: 1, b: 2}.extract!(:a)                 # => {a: 1} # destructive :slice
{a: 1, b: 2, c: 3}.except(:a, :b)         # => {c: 3}
```

### Enumerable

```rb
enu.each_cons(n)                        # each overlapping subarray of `n` items; last non-exact subarrays are not included
enu.each_slice(n)                       # each non overlapping subarray of `n` items; last non-exact subarray is included
```

## Basic I/O

Files:

```rb
io_obj.readline                         # includes the newline!
io_obj.readchar                         # reads one char; raises EOFError at the EOF
```

## Handling processes

The sections assume `require 'English'`, which aliases `$?` (`Process::Status`) to `$CHILD_STATUS`.

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

### Using `IO.popen3`

The output is not printed as well, so the same considerations as `IO.popen` apply.

```ruby
Open3.popen3([env,] command) do |stdin, stdout, stderr, wait_thread|
  stdout_content, stderr_content = stdout.read, stderr.read

  $stdout.puts(stdout_content) if stdout_content != ''
  $stderr.puts(stderr_content) if stderr_content != ''

  # `wait_thread.value` waits until completion
  #
  raise "Error (exit status: #{wait_thread.value.exitstatus})" if !wait_thread.value.success?

  stdout_content
end
```

### Live execution, via `Kernel#system`

This way, stdout/stderr content is printed straight away.

The exit status can be obtained directly via return value, or via `$CHILD_STATUS`.

```ruby
success = system(">&2 echo abc; sleep 2; false")

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