# Ruby libraries

- [Ruby libraries](#ruby-libraries)
  - [APIs/Stdlib](#apisstdlib)
    - [URL/HTML encoding](#urlhtml-encoding)
    - [Strings](#strings)
      - [Printf/interpolation](#printfinterpolation)
      - [Encoding](#encoding)
      - [Activesupport/Inflector](#activesupportinflector)
    - [Date/time](#datetime)
      - [Date/time templated parsing](#datetime-templated-parsing)
    - [Arithmetic/Maths/Rational](#arithmeticmathsrational)
    - [Algorithms](#algorithms)
    - [CSV](#csv)
    - [JSON/5](#json5)
    - [XML](#xml)
      - [REXML (`rexml/document`)](#rexml-rexmldocument)
      - [XPath](#xpath)
      - [Builder](#builder)
    - [YAML/Psych](#yamlpsych)
    - [OpenStruct (ostruct)/Struct](#openstruct-ostructstruct)
      - [Convert Hash/Array to recursive openstruct](#convert-hasharray-to-recursive-openstruct)
    - [Optparse](#optparse)
    - [open-uri](#open-uri)
    - [File/Dir/FileUtils/Pathname](#filedirfileutilspathname)
    - [Tempfile, Tmpdir](#tempfile-tmpdir)
    - [Concurrency](#concurrency)
      - [Threads exception handling](#threads-exception-handling)
      - [Variables vs threads](#variables-vs-threads)
      - [Mutex](#mutex)
      - [Thread-safe data structures (Queue)](#thread-safe-data-structures-queue)
      - [IO.pipe for IPC](#iopipe-for-ipc)
    - [I/O and terminal](#io-and-terminal)
    - [Compression](#compression)
    - [Hashing](#hashing)
    - [Encryption (openssl)](#encryption-openssl)
    - [SecureRandom](#securerandom)
    - [Flock](#flock)
    - [Etc](#etc)
    - [StringIO](#stringio)
    - [Built-in Unit Testing](#built-in-unit-testing)
    - [Gem](#gem)
      - [Version comparisons](#version-comparisons)
    - [ERB templates](#erb-templates)
    - [Rubocop](#rubocop)
    - [GC (garbage collection)](#gc-garbage-collection)
    - [Networking](#networking)
      - [HTTP](#http)
      - [Telnet](#telnet)
      - [SMTP](#smtp)
      - [Ping (ICMP)](#ping-icmp)
      - [TCP Server](#tcp-server)
      - [Other](#other)
  - [DnsMadeEasy (`dnsmadeeasy-rest-api`)](#dnsmadeeasy-dnsmadeeasy-rest-api)
  - [Databases](#databases)
    - [SQLite 3](#sqlite-3)
    - [Mysql2](#mysql2)
  - [Sorbet/Gradual Typing](#sorbetgradual-typing)

## APIs/Stdlib

### URL/HTML encoding

Rigorously encoding is not fully addressed by the Ruby STL; see https://stackoverflow.com/a/13059657.

Conclusions:

- Use `CGI::escape` if you only need form escape;
- If you need to work with URIs, use [`Addressable`](https://rubygems.org/gems/addressable), it offers URL encoding, form encoding and normalizes URLs.

Ruby examples:

```ruby
CGI::escape("&&&")                            # URL-encode a string: "%26%26%26"
URI.encode_www_form(p1: "&&&", "p2" => "!!!") # URL-encode params: "p1=%26%26%26&p2=%21%21%21"

CGI::escapeHTML('"html"')       # escape HTML

CGI.unescapeHTML("html")        # decode HTML entities; use only for basic cases, as it' not 100% complete (gem: https://github.com/threedaymonk/htmlentities)
HTMLEntities.new.decode("html") # htmlentities gem
```

Addressable examples:

```ruby
Addressable::URI.form_encode(p1: "&&&", "p2" => "!!!") # "p1=%26%26%26&p2=%21%21%21"

# Other APIs are provided for unencoding, parsing etc. (pubmeds `Addressable::URI`)
```

### Strings

Useful methods:

- `casecmp(str)`                      : case insensitive comparison
- `center`, `ljust(int[, str])`, `rjust(int[, str])`: WATCH OUT! Padding == justifying to the opposite direction!!
- `strip`, `lstrip`, `rstrip`
- `index(substr)`, `rindex(substr)`
- `slice(start[, end])`, `slice!`
- `start_with?`
- `lcomp`                             : doesn't exist; use `str.gsub /^(Regexp.escape(expr))+/`
- `each_slice`                        : doesn't exist; use `str.scan /.{1,n}/`

Splitting:

```ruby
# WATCH OUT!!!:
# - the default separator is /\s+/, which is convenient, but not intuitive
# - by default (`limit: nil`), empty fields are not included in the result; in order to include them,
#   specify `limit: -1`
split(separator[, limit])

split(/(separator)/)                  # retain the separator as separate token
split(/(?<=\+)\n(?=\+)/)              # retain the separator in the splitted fields via LB/LA (complex case: `\n` is discarded)

"ä\x00ß\x00\x00".split("\x00")        # ["ä", "ß"] - empty tokens are not included by default!!
"ä\x00ß\x00\x00".split("\x00", -1)    # ["ä", "ß", "", ""] - pass `-1` as limit in order to keep them
```

Substituting/matching:

```ruby
'foobar'.gsub(/(ba.)/, '\1\1')                        # 'foobarbar'; WATCH OUT!: the replacement string can't be manipulated, e.g. `upcase()`
                                                      # also, sub applies to the whole pattern, not just the capturing group
'foobar'.gsub(/(b)(a)/) { |match| $2.upcase + $1 }    # 'fooaBr'; `match` is a String
"Saverio <a@b.c>".match(/<(.*)>/)                     # returns MatchData object; [0] = entire match; [1..] = matching groups
str[/regex/, idx]                                     # same as `str.match(/regex/)[idx]`
"saverio".scan(/(ver(io))/)                           # [["verio", "io"]]
"saverio".scan(/cusumano/)                            # []
File.fnmatch(@pattern, @string)                       # use this API in order to perform glob matching!

# The non-capturing group is: included in the match;     replaced; not captured.
# The lookahead is            included in the match; not replaced; not captured.
#
# This is inconsistent with :scan, which will only extract the capturing groups.
#
"y:3:a".gsub(/(?:y):(\d):(?=\w)/) do |match|
  puts match.inspect            # "y:3:"
  puts $1.inspect               # "3"
  puts $2.inspect               # nil
  puts $3.inspect               # nil

  "z:#{$1.to_i + 1}:"
end # => "z:4:a"

# Need to use this strategy in order to detect if (g)sub matched.
# If this pattern doesn't fit, e.g. not in a function, use an outer boolean.
# WATCH OUT! Make sure that the use case logic can't be simplified instead of using sub() this way.
#
def find_match
  host.sub(HOST_MATCHER) { |stack| return "#{stack}#{$1}" }
  raise "Not found"
end

# Linux tool `tr`.
#
"abcZHNC".tr("ZHNC", "*")                               # return "abc****"
```

#### Printf/interpolation

`String% *values` equals to `sprintf(str, *values)`

Named parameters:

```rb
"%{val}" % {val: 28}                    # unformatted param
"%<num>.d" % {num: 28}                  # formatted param
```

#### Encoding

Also see [Data type Conversions](#data-type-conversions).

```ruby
String.new(str, encoding: enc)    # !! Defaults to ASCII-8 encoding !!
str.encode(encoding)
str.force_encoding(encoding)
str.valid_encoding?

# Clean non UTF-8 chars.
#
str.encode('UTF-8', invalid: :replace, undef: :replace, replace: '')

# scrub 4-bytes characters from an utf8 string (essentially, mysql's `utf8mb4` to `utf8mb3`)
#
string.each_char.select { |char| char.bytesize < 4 }.join

# Scripting encoding magic comment; must be in first or second line.
#
# [en]coding: UTF-8

# Scripting encoding.
#
__ENCODING__

# The external encoding is the encoding used when creating the IO object.
# If the internal enc. is specified, IO objects (valid for read and write) are transcoded
# to it before being read/write.
#
Encoding.default_(internal|external)

# File opening options
#
open(file, "r:<external_enc>:<internal_enc>")
open(file, "w:<external_enc>"

# ignore Byte Order Mark
#
IO.read(filename, 'bom|utf-8')
```

#### Activesupport/Inflector

Don't forget that the gem is `active_support` (with underscore!!)

- `underscore`                  : `ActiveModel::Errors` => `active_model/errors`
- `underscore`                  : `ABC CD/E`            => `abc cd/e`            # !! WATCH OUT !!
- `pluralize`                   : `flying_jellyfish`    => `flying_jellyfishes`
- `tableize`                    : `Order`               => `orders`
- `humanize`                    : `ActiveModel::Errors` => `activemodel_errors`
- `parameterize(separator: "_")`: `My Exercise`         => `my_exercise`         # opposite of humanize()

### Date/time

General APIs/operations:

```ruby
@time + seconds

# These are not available to Time
#
@date >> @months; @date << @months                        # Adds/subtracts whole months (not days) to a date
@date >> 12 * @months                                     # Use to add/subtract whole months/years (not days) to a date
@date.next_month(@months=1), @date.prev_month(@months=1)  # More readable; also supports `day`, `year`

@date_time.to_time.to_i                                   # DateTime to Unix time (epoch)
@time.subset                                              # Fractional part of a time (rational)
Time.at(time_i)                                           # Unix time to Time

@date.wday                                                # Sun=0 .. Sat=6
```

In order to substract/add an year/month, while keeping the smaller units (ie. day/month for year), use `(prev|next)_(year|month)`. The date 29/Feb is converted to 28/Feb.

`DateTime` (require `time` or `date`) supports calendar operations, but not timezones (only offsets).
If one needs to do calendar operations but print the timezone, use `DateTime`, but print the zone using something like `Time.now.zone`.

In order to parse a freeform time, use `DateTime`, but mind the offset:

```ruby
# If the offset is not specified, it's assumed to be UTC.
#
DateTime.parse("10:40 #{Time.now.strftime("%z")}")     # => #<DateTime: 2020-10-27T10:40:00+01:00 ...>

# WATCH OUT! When parsing weekdays, the date in the current week is always returned, which for Ruby
# starts on Sunday.
#
DateTime.parse("thu 10:33 #{Time.now.strftime("%z")}") # => #<DateTime: 2020-10-29T10:33:00+01:00 ...>
```

#### Date/time templated parsing

Padding is also available for minutes/hour, and it's insane.

- `%a`: The abbreviated weekday name ('Sun')
- `%A`: The full weekday name ('Sunday')
- `%b`: The abbreviated month name ('Jan')
- `%B`: The full month name ('January')
- `%c`: The preferred local date and time representation
- `%d`: Day of the month (01..31)
  - `%-d`: non-padded (1..31)
- `%H`: Hour of the day, 24-hour clock (00..23)
- `%I`: Hour of the day, 12-hour clock (01..12)
- `%j`: Day of the year (001..366)
- `%L`: Millisecond of the second (000..999)
- `%N`: Fractional seconds digits, default is 9 digits (nanosecond)
  - `%3N`: millisecond (3 digits)
  - `%6N`: microsecond (6 digits)
  - `%9N`: nanosecond (9 digits)
  - `%12N`: picosecond (12 digits)
- `%m`: Month of the year (01..12)
  - `%_m`: blank-padded (' 1'..12)
  - `%-m`: non-padded (1..12)
- `%M`: Minute of the hour (00..59)
- `%p`: Meridian indicator ('AM' or 'PM')
  - `%P`: lower case
- `%S`: Second of the minute (00..60)
- `%U`: Week number of the current year,starting with the first Sunday as the first day of the first week (00..53)
- `%W`: Week number of the current year, starting with the first Monday as the first day of the first week (00..53)
- `%w`: Day of the week (Sunday is 0, 0..6)
- `%x`: Preferred representation for the date alone, no time
- `%X`: Preferred representation for the time alone, no date
- `%y`: Year without a century (00..99)
- `%Y`: Year with century
- `%z`: Time zone as hour and minute offset from UTC (+0900)
  - `%:z`: hour and minute offset from UTC with a colon (+09:00)
  - `%::z`: hour, minute and second offset from UTC (+09:00:00)
- `%Z`: Time zone name (CEST)
- `%%`: Literal ``%'' character

Combinations:

- `%F`: The ISO 8601 date format (%Y-%m-%d)
- `%R`: 24-hour time (%H:%M)
- `%T`: 24-hour full time (%H:%M:%S)

Examples:

```ruby
'%F %T'             # 2008-12-31 01:23:45
'%a %b %d %T %Z %Y' # Wed May 02 01:23:45 GMT 2014
'%m/%d/%y %I:%M %p' # 10/31/09 01:23 PM
'%YT%T%:z'          # 2014T14:20:54+00:00 (json format)
```

### Arithmetic/Maths/Rational

```rb
-1 / 2                  # WATCH OUT!!! Result is -1 !!!
100.divmod(3)           # [3, 1] div + mod!! WATCH OUT, it's (carry, sum = ...) when computing additions!

r = Rational(8, 9)      # no new()!!
(r * 9).to_i == 8       # arithmetic yiels other Rational instances

x.gcd(y)                # Greatest Common Divisor (Massimo Comun Divisore, MCD)
arr[1..].inject(arr[0]) { |result, n| result.gcd(n) } # GCD of multiple numbers
Prime.prime_division(x) # Primes factorization; returns an array of [factor, exp] (requires 'prime')
                        # returns `[]` if x == 1

arr.permutation(len)    # find all the permutations of length len of an array; ordering is not guaranteed

x = Math.log2(y)
x = Math.sqrt(y)        # square root; DO NOT USE `y ** 0.5`!!!
x = Math.sin(y_rad)     # remember that rad = deg / 180 * Pi
x = Math.cos(y_rad)
x = Math.pow(y[, m])    # if m is specified, it's performed `x^y % m`, without huge intermediate numbers
Math::PI

x.clamp(a, b)           # not really math, but convenient
```

### Algorithms

```rb
# DON'T FORGET!!
#
include Containers

# MinHeap works the same.
# WATCH OUT!! This gem's heap implementation is *very* slow: adding 10k elements took 2.6", against 0.06" of the standard sort.
#
heap = MaxHeap.new([4, 5])
heap << 10                 # Can't concatenate appends
heap.max                   # => 10; doesn't pop
heap.pop                   # => 10; returns nil if there are no elements
```

Other data structures:

- `Queue` (implemented via doubly linked list)
- `RubyDeque` (implemented via doubly linked list)
- `Trie`, `RBTreeMap`, `PriorityQueue`, `Stack`, `SplayTreeMap`, `SuffixArray`

There are also some algorithms, and C versions of the above; see [docs](https://kanwei.github.io/algorithms).

### CSV

Options:

- `headers: :first_row`
- `force_quotes: true`

```ruby
CSV.read(filename)                                        # Read csv; the output is a CSV::Table, which can be iterated.
CSV.read(filename).to_a                                   # Doesn't work as expected: first entry is the header; each entry is a flat array, not a hash.
CSV.parse(data_string[,options])                          # Parse a string
CSV.foreach(filename, options) { |row| ... }              # Stream (for large files)

csv_content = CSV.generate { |csv| csv << [values] }      # Write to string
CSV.open("/path/file.csv", "w", options) { |csv| ... }    # Write to file
CSV.open('links.csv', 'ab', options) { |csv| ... }        # Append to file

[values].to_csv                                           # Encode a single row
```

Row can be indexed as a hash; empty fields are parsed as nil.

Utils:

```ruby
CSV.read(csv_file)[0]                                     # Read headers. There is no obvious way from the API (what if the file has only headers?)
row.to_hash.merge(row) { |_, value| value.to_s }          # Simple way to convert each row nil values to blank strings
```

### JSON/5

```ruby
JSON.parse(string)                                        # Returns the tree (as Ruby objects); the root object class depends on the input (eg. Array, Hash, ...)
JSON.parse(string, object_class: OpenStruct)              # Parse into OpenStruct; very convenient.
string = JSON.generate(input)                             # `input` can be a hash
string = JSON.pretty_generate(input)
```

JSON5 gems (only parse; use `json` gem to convert to string):

- `json5`: fast to parse, but has no `load_file`
- `rb_json5`: slow to parse, but has `load_file`

### XML

#### REXML (`rexml/document`)

```ruby
xml_str = "\
<myroot>
  <internal_1>
    <leaf_1/>
    <leaf_2>33</leaf_2>
  </internal_1>
  <internal_2>1</internal_2>
  <internal_2>2</internal_2>
</myroot>"

root = REXML::Document.new(xml_str)

# Watch out! REXML formatters insert newlines, without any option, in the text values,
# so if newlines are meaningful, REXML must not be used – use nokogiri.
#
xml_formatter = REXML::Formatters::Pretty.new  # pretty print
xml_formatter.compact = true
xml_formatter.write(root, $stdout)

myroot.elements.[reverse_]each { |child| }      # iterate the child elements
myroot.elements[1..-1]                          # !!! NOT supported !!!
myroot.elements['internal_2']                   # pick the first child named element
myroot.elements.to_a('//myroot/internal_2')     # access via XPath

internal_2.parent   # (Element myroot)
internal_2.name     # 'internal_2'
internal_2.text     # '2'

internal_3 = myroot.add_element("internal_3", 'attr' => 'value')  # add element, optionally with attributes
myroot.delete(internal_3)                                         # remove element
internal_3.parent = nil                                           # remove element (other way)

internal_1.attributes['type'] = 'array'         # add/remove attributes
internal_1.attributes.delete('type')

internal_1.insert_before( 'leaf_2', REXML::Element.new('leaf_between'))
internal_1.insert_after( 'leaf_2', REXML::Element.new('leaf_end'))

internal_4 = myroot.add_element("internal_4", 'type' => "mytype")
internal_4.text = 'mytext'
```

#### XPath

```ruby
# Search by multiple attributes:
#
index_document.elements.to_a( %Q|//RDF:RDF/RDF:Description[@NS1:type="folder" and @title="title"]| )

# Search relative to element:
#
folder_content_element.elements.to_a( 'RDF:li' )
```

#### Builder

- There is no need of a target; each action on it returns the string. but NEVER use `to_s`!!!
- Every block has the xml element as argument, but the 'xml' variable can be used as well!

```ruby
# Base example

xml = Builder::XmlMarkup.new [:target => some_string, :indent => spaces]
xml.instruct!
xml.pizzas do # here we aren't using a named element pointer
   xml.4_stagioni :type => 'buona' do                                          # !!! REQUIRES do/end, NOT {} !!!
     xml.ingrediente :type => 'scaduto', 'catrame'                             # :type = attribute
     xml.photo do |photo_xml|                                                  # here we are using a named element pointer
       photo_xml.cdata! 'my cdata!'
     end
   end
end
def generate_xml_children(xml, level, &block) # recursion works!
  node_name = [nil, :shows, :events][level]
  xml.__send__( node_name, :type => 'array', &block)                         # use __send__, because send method is interpreted as xml element
end
```

### YAML/Psych

The `yaml` library is actually an alias for `psych`. Comments are always ignored when parsing!

```ruby
# WATCH OUT! This doesn't interpret ERB templates. See #erb-templates for that.

YAML.load_file(filename)  # Parse from file
YAML.load(string)         # ... from string

YAML.dump(object[, io])   # Convert to string, optionally into IO
object.to_yaml            # ... convenience
```

### OpenStruct (ostruct)/Struct

Struct is in the stdlib; it doesn't accept new arguments. OpenStruct doesn't allow methods definition.

```ruby
os = OpenStruct.new(field: 'val')  # the hash is optional
os.name = "John"
os['surname'] = "Smith"       # alternate assignment form
puts os.name, os.surname
os.delete_field(:surname)     # remove a field

# Enclose whe whole block in a module in order to namespace.
#
Klazz = Struct.new(:a, :b) do
  def mymethod; end
end

Klazz.new('aval', 'bval')

Klazz.members # defined members ([:a, :b])
Klazz.to_a    # values array    (['aval', 'bval'])
Klazz.to_h    # values hash     ({a: 'aval', b: 'bval'})

# Alternative form, yields `Struct::Klass`.
#
Struct.new('Klass', :a, :b)
```

#### Convert Hash/Array to recursive openstruct

Convenient solution for converting a Hash/Array to a recursive openstruct, builder pattern-style:

```ruby
def to_recursive_ostruct(node)
  case node
  when Hash
    node.each_with_object(OpenStruct.new) { |(key, value), ostruct| ostruct[key] = to_recursive_ostruct(value) }
  when Array
    node.map { |item| to_recursive_ostruct(item) }
  else
    node
  end
end

strk = to_recursive_ostruct({key: [1,2,3], key2: {subkey: 'a'}})
strk.key           # [1,2,3]
strk.key2.subkey   # 'a'
strk.key2.notfound # nil
```

Original source: https://stackoverflow.com/a/42520668.

### Optparse

```ruby
# Set default options
#
options = { garbage: false, logfile: nil, volume: 1.0, is_moron: nil, crappy: true }

optparse = OptionParser.new do |op|
  op.banner = "Usage: #{File.basename(__FILE__)} [-b|--base] [-p PARAM] [-c PARAM] [-e PARAM] [-[no-]n|--[no-]negated]"

  # Base form; if there is more than one param (e.g. the long version), the last array entry is the
  # description.
  #
  op.on('-b', '--base', 'Base form') do
    options[:base] = true
  end

  # With optional parameter (not enforced, due to default); upper case is not necessary.
  #
  op.on('-p PARAM' 'Optional param') do |param|
    options[:param] = param || 'default_value'
  end

  # Class conversion. An Array can be passed, it takes the form 'a,b,c'.
  # Use `TrueClass` for booleans.
  #
  op.on('-c PARAM', Float, 'Converted param') do |param|
    options[:converted] = param
  end

  # Enum params. Can be specified as symbol or string, and are passed the same
  #
  op.on('-e PARAM', ['yes', 'no', 'maybe']) do |param|
    options[:enum] = param
  end

  # Negation; passes false when negated, true otherwise.
  # When specified in long form, it's automatically activated for the short one (in this example,
  # '-no-n').
  #
  op.on('-n', '--[no-]negated') do |param|
    options[:negated] = param
  end
end

# Removes options/params from ARGV
#
optparse.parse!

puts "Options: [#{ options.inspect }]"
```

### open-uri

Open files/download http content; automatically detects the appropriate class for loading the stream content:

```ruby
open('http://cippa-lippa').read
```

### File/Dir/FileUtils/Pathname

WATCH OUT! File stats, in case of symlinks, apply to the target file; use `File.lstat` instead.

```ruby
File.dirname(path)                              # Parent dir
File.extname(fname)                             # Extract the (last) extension, including leading dot
File.size(fname)
File.mtime(fname)
File.lstat(fname).mtime                         # For symlinks; see note above.
File.symlink(target, link)
File.utime(atime, mtime, fname)                 # Update a/mtime
`system("touch -h -m -t #{atime.strftime('%Y%m%d%H%M.%S')} #{fname.shellescape}")` # For symlinks (!) (mtime)

Dir.pwd                                         # working path
Dir.chdir(path) { }                             # Change current dir; WATCH OUT! not thread safe! see https://bugs.ruby-lang.org/issues/9785
Dir.home(user)                                  # Find home of the given user (optional)

Dir.foreach(dosPath){|fname|}                   # not recursive; includes '.[.]'
Dir.glob(antPath){|fname|}                      # glob format; 'path/**/pattern' => '**' recurses under path
Dir["**/*"]                                     # recursive dir list; can prepend dir; dirs before files

Dir.mkdir                                       # not recursive
FileUtils.mkdir_p(path)                         # recursive

Dir.delete(dosPath)                             # not recursive
FileUtils.remove_dir(path, true)                # recursive
FileUtils.rm_rf(path)                           # rm -rf!!

FileUtils.touch(filename)
FileUtils.chmod(permissions, filename)          # change permissions; can be symbolic, e.g. `+x`
FileUtils.chown(user[, group[, filename]])      # change owner - File.chown needs the group/user id!!

FileUtils.cp(src, dest, **options)              # copy file
```

Paths handling:

```ruby
File.realpath("/path/to/symlink")               # get the real filename from symlinks:

require 'pathname'; Pathname.new(@path2).relative_path_from(Pathname.new(@path1)).to_s # find the directory relative to another directory
```

File opening/closing (flags):

```ruby
# WATCH OUT!! The flags are not the same as the standard ruby; 'w' != WRONLY, 'a' != APPEND
#
# - 'w': File::CREAT | File::WRONLY | File::TRUNC
# - 'a': File::CREAT | File::WRONLY | File::APPEND

# Append to file, creating if non existent
#
File.open(@file, 'a') { ... }

# If a file needs to be closed deterministically, then do it manually - GC collection doesn't guarantee
# that it will be collected/closed.
#
f = File.open(@file, File::CREAT | File::WRONLY | File::APPEND) {... }
f.close

# Use file modes, e.g. to append. WATCH OUT! They're letters (see https://ruby-doc.org/core-3.1.2/IO.html#method-c-write).
#
IO.write @file, @content, mode: @mode
```

File locking, via flock:

```ruby
# WATCH OUT!! This strategy doesn't play well with forked processes, if the file handle is acquired
# before forking; in this case, the lock will be acquired on both processes (!).

lock_file = File.open('/tmp/mylock', File::RDWR | File::CREAT)

# Exclusive lock, non-blocking
if lock_file.flock(File::LOCK_EX | File::LOCK_NB)
  # ... critical area ...
  # Make sure the handle is not GC'ed!
  $lock_file = lock_file
else
  # See note in "File opening/closing"
  lock_file.close
end
```

### Tempfile, Tmpdir

Do not use tempfile for references that stick around!! Tempfile instances are always deleted on garbage collection.

```ruby
# @naming can be either "prefix" or ["prefix", ".extension"]

file = Tempfile.create(@naming)                # Creates, closes, doesn't delete
file.print content
puts file.path
file.close                                     # WATCH OUT!! Don't forget!!
File.unlink(file)  # optional

file = Tempfile.create(@naming).path           # Easiest: path + IO.write
IO.write(file, content )
File.unlink(file)                              # optional

Tempfile.create(@naming) { |f| operation(f) }  # Creates, operates, deletes
Tempfile.new(@naming)                          # Same as `.create`; don't use
Tempfile.open(@naming) { |f| operation(f) }    # Deprecated for most usages; don't use

# Generates a temporary filename with `a`/`.png` prefix/extension, in the system temporary directory.
# !!This is the best API for tempfiles that need to stick around!! Note that no file is created.
# The block is ugly, but required.
#
require 'tmpdir'
Dir::Tmpname.create(['a', '.png']) { }

# Creates a temporary dir, for the duration of the block.
# @prefix_suffix is the same format as Tmpname.create.
# If the block is not provided, the directory is not deleted.
Dir.mktmpdir(@naming) { |dir| ... }

# Find system temporary directory
require 'tmpdir'
Dir.tmpdir
```

### Concurrency

#### Threads exception handling

If one wants threads error to be raised in the main thread:

- either set `Thread.abort_on_exception = true`
- or it will be raised on `Thread#join` (or `#value`)

`abort_on_exception` can be set on a per-instance basis; from the thread block: `Thread.current.abort_on_exception = true`

#### Variables vs threads

!! WATCH OUT !! Don't let thread blocks access variables in the outer scope; if they start with a delay, hilarity will ensue:

```rb
# When threads start late, they will use the `socket` variable of the following cycle.
# `loop` works, because the`socket` variable is not shared, as it's in an inner scope (!!!).

while true
  socket = @server.accept
  Thread.new { puts socket.gets }
end

# Instead, pass them to the Thread:

while true
  socket = @server.accept
  Thread.new(socket) { |socket| puts socket.gets }
end
```


```rb
mythread.value                    # Joins the thread, and returns the return value
Thread.current[:pending_inputs]   # Thread-local variables
```

#### Mutex

```ruby
mutex = Mutex.new
mutex.synchronize { }
```

#### Thread-safe data structures (Queue)

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

Methods:

- `push(element)`
- `pop`
- there is no peek; must `pop` and `push`

#### IO.pipe for IPC

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
## PROCESSES + BLOCKING READ

rd, wr = IO.pipe

## fork() returns PID in the parent, and nil in the child, so the first block is the parent, the second, the child
if parent_pid = fork
  wr.close
  puts "Read: #{rd.read.inspect}"
  rd.close
  Process.wait	# waits on the child/writer
else
  rd.close
  wr.write "Message"
  wr.close
end

## THREADS + PSEUDO-NON-BLOCKING READ
## The pattern for pseudo-non-blocking (which blocks) is quite strange.

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

### I/O and terminal

```ruby
# Read a single char
#
require 'io/console' # stdlib
$stdin.getch

# Get the terminal size (this gem is the most reliable metho)
#
require 'tty-screen'
TTY::Screen.width    # => 280; aliases: columns/col
TTY::Screen.height   # => 51;  aliases: rows/lines
```

### Compression

Zip (rubyzip):

```rb
# Create a zip archive
Zip::File.open(archive_filename, create: true) do |zipfile|
  # Create an archive entry
  zipfile.get_output_stream(zip_entry_name) do |zip_entry_io|
    # Stream writes to the archive entry
    rows.each { |row| zip_entry_io.write(row.to_csv) }
  end
end
```

Gzip:

```rb
# Long form
gz = Zlib::GzipWriter.new(io)
gz.write 'content...'
gz.close                       # This *must* be invoked

# Short forms!
Zlib::GzipWriter.wrap(io, ...) { |gz| ... }
Zlib::GzipReader.wrap(StringIO.new(source), &:read)
```

### Hashing

```rb
Digest::MD5.hexdigest 'abc'      #=> "90015098..."
```

### Encryption (openssl)

```ruby
cipher = OpenSSL::Cipher::Cipher.new(algo)
cipher.encrypt

iv = cipher.random_iv

cipher.key = key
cipher.iv  = iv

ciphertext = iv + cipher.update(plaintext) + cipher.final

ciphertext = Base64.encode64(ciphertext) if base_64_encoding

# decoding

cipher = OpenSSL::Cipher::Cipher.new(algo)
cipher.decrypt

cipher.key = key

cipher.iv = ciphertext.slice!(0, cipher.iv_len)
plaintext = cipher.update(ciphertext) + cipher.final

# useful

cipher.random_key          # generate a key
```

### SecureRandom

Interesting :crypt API; generate a linux password hash

```sh
ruby -rsecurerandom -e 'print "Password: "; puts gets.chomp.crypt("$6$" + SecureRandom.hex(8))'
```

### Flock

Flags:

- `LOCK_EX` - Exclusive
- `LOCK_SH` - Shared lock
- `LOCK_UN` - Unlock
- `LOCK_NB` - Non-blocking acquisition; combine using logical or. Returns `0` (success) or `false` (failure).

The lock is released on GC (/exit), so if one wants it to hang around, it must be assigned to global variable.

```ruby
# WATCH OUT: don't use `w` file mode. Default permissions: 0664.
#
$lock_file = File.open("/tmp/coverband-flock", File::RDWR | File::CREAT)
$lock_file.flock(File::LOCK_EX | File::LOCK_NB)
```

### Etc

```ruby
Etc.getpwuid.name             # Get current user name
Etc.getlogin                  # Current LOGGED IN user; WATCH OUT!: not necessarily the current
Etc.getpwuid.dir              # Get current user home dir. For generic users, use `Dir.home`
```

### StringIO

```ruby
# In order to append and allow reading at the same time, use the combination below.
#
# WATCH OUT! Must consider the cursor (on read/write) when initializing with a string:
#
# - if append mode is not specified, the cursor is at the beginning (writes will overwrite)
# - if append mode is specified, the cursor is at the end (reads will be blank)
#
# can use #rewind to reset the cursor
#
StringIO.new("start_string", File::RDWR | File::APPEND)

# Check size (there's no :empty? method in StringIO).
#
StringIO.new.size

# WATCH OUT! :puts adds a newline, but chomps the parameter's last trailing newline
#
buffer.print "line\n"    # one trailing newline
buffer.puts  "line\n"    # one trailing newline
buffer.puts  "line\n\n"  # two trailing newlines
buffer.puts  "line", ""  # two trailing newlines
```

### Built-in Unit Testing

There are many assertions; see [stdlib doc](https://ruby-doc.org/stdlib-3.1.1/libdoc/test-unit/rdoc/Test/Unit/Assertions.html#method-i-assert_equal).

```rb
# The gem is built-in, but when running via Bundler, must include it (`test-unit`).

require 'test/unit'

include Test::Unit::Assertions

assert       true
assert_equal 1, 1
assert_same  [], []   # fails; objects must be the same (nil: same as nil)
assert_nil   1

error = assert_raises(MyError) { stack.pop }  # Error class defaults to RuntimeError
assert_equal "Stacks Empty!", error.message
assert_raises(MyError, "printed message") { } # WATCH OUT! The message is _not_ the one raised by the block
assert_raises("printed message") { }          # (alternate form)
```

### Gem

#### Version comparisons

Rigorous way to compare versions:

```ruby
'3.0' > '10.0'                                     # true (wrong!)
Gem::Version.new('3.0') > Gem::Version.new('10.0') # correct
Gem.ruby_version > Gem::Version.new('10.0')        # ready API to retrieve the ruby version
```

### ERB templates

Simple rendering logic example; keep in mind the binding considerations.

```ruby
template_content = IO.read(template_file)

# WATCH OUT! Instance variables of the binding object are different from instance variables in the
# binding - ie. using :instance_variable_set will not work as expected.
# There are different approaches. This is st00pid simple; for a more sophisticated one, see
# https://stackoverflow.com/a/41590728.
#
variables.each { |name, value| TOPLEVEL_BINDING.eval "@#{name} = #{value.inspect}" }

# It's possible to render with a local context, and evaluate in the global one, however, it doesn't
# have any particular advantage (at least, for the use case).
#
rendered_template = ERB.new(template_content).result(TOPLEVEL_BINDING)

TOPLEVEL_BINDING.eval(rendered_template, template_file)

# Since we're working on the top level binding (so that classes will be in the global namespace),
# it's better to clean up :-)
#
variables.keys.each { |name| TOPLEVEL_BINDING.eval "remove_instance_variable '@#{name}'" }
```

If one doesn't use variables, but just `ENV` references (e.g. yaml configfile), then just:

```rb
erb_template = IO.read(template_file)
yaml_string = ERB.new(erb_template).result
YAML.load(yaml_string)
```

### Rubocop

```rb
# To disable on one line, comment on the side
#
puts 123 # rubocop:disable Foo/Bar

# Disable multiple cops
#
# rubocop:disable Foo/Bar, Baz/Qux
puts 123

# "Todo" disabling; works like disable, but convenient
#
# rubocop:todo Foo/Bar
puts 123
```

### GC (garbage collection)

APIs:

- `enable`/`disable`
- `start`

WATCH OUT! When profiling, the GC should be disabled, because profilers may not be able to recognize GC pauses.


### Networking

#### HTTP

```ruby
# An URI instance ("URI(address)") can be used, whose :host, :port, :request_uri can be used
#
Net::HTTP.start(uri.host, use_ssl: true) do |http|
  request = Net::HTTP::Get.new(uri)   # for post: Post.new(uri) + body=data
  request.basic_auth @user, @pwd
  request['Accept'] = 'application/vnd.github.v3+json'
  response = http.request(request)
  status_header = response['Status']
  headers = response.to_hash
end
```

#### Telnet

```ruby
# Set `telnetmode` to false in order to connect eg.to pop3.
#
Net::Telnet.new('host' => addr, 'port' => port, 'telnetmode' => false)

# Connecting to memcached (or others).
# The 'Prompt' is essentially what is returned after a command is executed.
#
connection = Net::Telnet.new( 'Host' => host, 'Port' => 11211, 'Prompt' => /^OK\n/ )
connection.cmd( 'flush_all' )
```

#### SMTP

https://stackoverflow.com/a/3559598

```ruby
# WATCH OUT! This can't be any arbitrary string.
#
BOUNDARY_MARKER = 'C64PIZZAOHYEAH!!!'

# cc/recipients:  standard format (emails separated by `;`)
# attachments: (array or single string) filenames
#
def prepare_email_text(sender, recipients, subject, body, cc: nil, attachments: [])
  raise "The body includes the boundary_marker!" if BODY.include?(BOUNDARY_MARKER)

  # Spacing is crucial!!! Pay much attention to where newlines are (and aren't).

  body += "\n" if body !~ /\n\Z/

  mail_headers = <<~TEXT
    To: #{recipients}
    From: #{sender}
    Subject: #{subject}
  TEXT

  mail_headers <<~TEXT if cc
    Cc: #{cc}
  TEXT

  if attachments.size > 0
    mail_headers << <<~TEXT
      MIME-Version: 1.0
      Content-Type: multipart/mixed; boundary="#{BOUNDARY_MARKER}"
      --#{BOUNDARY_MARKER}
      Content-Type: text/plain
      Content-Transfer-Encoding: 7bit
    TEXT

    attachment_section = Array(attachments).inject("") do |attachments_buffer, full_filename|
      base_filename = File.basename(full_filename)
      raw_content = IO.read(full_filename)

      encoded_content = [raw_content].pack('m')  # base 64

      attachments_buffer += <<~TEXT.chomp

        --#{BOUNDARY_MARKER}
        Content-Type: application/octet-stream; name=\"#{base_filename}\"
        Content-Transfer-Encoding: base64
        Content-Disposition: attachment; filename="#{base_filename}"

        #{encoded_content}
      TEXT
    end

    attachment_section << "--#{BOUNDARY_MARKER}--"
  end

  "#{mail_headers}\n#{body}#{attachment_section}"
end

# sender:            assumed to be the same as login user
# cc/bcc/recipients: standard format (emails separated by `;`)
#
def send_email(sender, recipients, email_text, password, cc: '', bcc: '')
  domain = sender[/@(.+)/, 1]

  # BCCs are simply emails that are SMTP recipients, but are not listed in the email headers.
  #
  all_recipients = recipients.split(';') + cc.split(';') + bcc.split(';')

  # To send via local mail transport agent, simply use:
  #
  #     Net::SMTP.start('localhost')
  #
  smtp = Net::SMTP.new('smtp.gmail.com', 587) # TLS send (see https://stackoverflow.com/a/3559598)
  smtp.enable_starttls
  smtp.start(domain, sender, password, :login) do
    smtp.send_message(email_text, sender, all_recipients)
  end
end

email_text = prepare_email_text(SENDER_EMAIL, RECIPIENT_EMAILS, SUBJECT, BODY, cc: CC_EMAILS attachments: attachment)
send_email(SENDER_EMAIL, RECIPIENT_EMAILS, email_text, password, cc: CC_EMAILS, bcc: BCC_EMAILS)
```

#### Ping (ICMP)

`net/ping` has been removed from the stdlib anymore. Must use a gem or the `ping` unix tool, since the solution proposed on [StackOverflow](https://stackoverflow.com/a/7520485) has inconsistent results.

#### TCP Server

```rb
require 'socket'

server = TCPServer.new(2000) # port

loop do
  client = server.accept    # Wait for a client to connect
  message = client.gets     # WATCH OUT! gets() expects a newline terminator
  client.puts "Response!"
  client.close
end
```

#### Other

- Convert curl request to Ruby: see https://jhawthorn.github.io/curl-to-ruby.
- Get hostname: `Socket.gethostname`.

## DnsMadeEasy (`dnsmadeeasy-rest-api`)

The underlying DME API is horrific, both in the implementation and documentation, and the gem seems not to be better either.

```rb
def update_records(record_names_ids, limit: nil)
  record_names_ids = record_names_ids.to_a[0, limit || -1]

  # Both the gem and the APIs are badly documented.
  # The API is badly designed; in principle, only the id and the fields to update should be required;
  # additionally, the ideal structure should be a map with the ids as keys, and the fields to update
  # as value.
  # The example in the gem "documentation" misses the `ttl` field, which causes the example to fail
  # (with "400 Bad Request").
  #
  # Reference: https://api-docs.dnsmadeeasy.com/#c9a001b1-b550-4d28-a7ff-605d4ad89c69.
  #
  conditions = record_names_ids.map do |name, record_id|
    { "id" => record_id, "name" => name, "type" => RECORD_TYPE, "ttl" => TTL }
  end

  @api.update_records(@domain, conditions, { "value" => LB_ADDRESS })

  # The following is kept for the records.
  #
=begin
  record_names_ids.each do |name, record_id|
    # This is completely insane.
    #
    # The input values are a confusing mess:
    #
    # - the last argument (value) is ignored.
    # - the subdomain and record type are partially ignored (!); some values cause errors, some others
    #   are ignored.
    # - non-nil values are still required.
    # - at least, the domain (and the record id) required 🙄.
    # - if the combination (domain, record_id) is not found, a "404 Not Found" is raised.
    #
    # However, without the whole and correct data (except value, it seems), the record is DELETED (!!!!!!).
    #
    @api.update_record(@domain, record_id, name, RECORD_TYPE, nil, {"value" => LB_ADDRESS})
  end
=end
end

def find_record_ids(subdomains)
  # The gem doesn't consider paging (!!!).

  # If the domain is not valid, an error is raised.
  # If there is not record, or the record type is invalid, an empty array is returned.
  #
  # There's no API for finding multiple record ids; internally, the gem uses :record_for.
  #
  # record_id = @api.find_record_id(@domain, subdomain, RECORD_TYPE)

  # CNAMEs are unique, so we can put the entries in a hash without checking.
  #
  all_records_data = @api.records_for(@domain)

  raise "The result is paged!" if all_records_data.fetch("totalPages") > 1

  all_record_names_ids = all_records_data
    .fetch("data")
    .each_with_object({}) { |record, records_hash| records_hash[record.fetch("name")] = record.fetch("id") }

  subdomains
    .each_with_object({}) { |name, result| result[name] = all_record_names_ids[name] }
end
```

## Databases

### SQLite 3

```ruby
require 'arrayfields' # makes rows accessible also as hash in addition to be arrays

db = SQLite3::Database.new("test.db")

rows           = db.execute(query[, params])
columns, *rows = db.execute2(query)

db.execute(query) { |row| ... }

row   = db.get_first_row("select * from table")
count = db.get_first_value("select count(*) from table")

db.execute("insert into foo (?)", SQLite3::Blob.new("\0\1\2\3\4\5"))  # insert blob
db.execute_batch(queries)  # accepts multiple statements (normal #execute doesn't); no blocks accepted.

db.results_as_hash  = true
db.type_translation = true  # translate the data to the declared data type

# In order to rollback from the block form transaction, one needs to raise/rescue a custom excpetion,
# since the #rollback method is not accepted there.
database.transaction { |db| db.execute(...); raise MyRollback}
db.transaction; db.execute(...); db.commit # or db.rollback

db.execute "SELECT name FROM mysql_master WHERE type = 'table'"  # get table names

db.extend(SQLite::Pragmas); db.table_info(table_name)  # table metadata
```

### Mysql2

```rb
@client = Mysql2::Client.new(host: HOST, username: USER, password: PWD)

# Queries return hashes by default.
# The #query returned class is a proxy.
result = @client.query(sql, as: :array)

# Iterate.map
result.each { || ... }
result.map { |c1, c2| ... }

# Get the column names
result.fields

# Get another row
result.first
```

In order to stream:

```rb
# The :cache_rows option avoids a warning.
#
result = @client.query(sql, stream: true, cache_rows: false)

# WATCH OUT! If terminating early (e.g. error recovery), must invoke this, otherwise, new queries will
# raise the error "Commands out of sync; you can't run this command now"
#
result.free
```

## Sorbet/Gradual Typing

See Ticketsolve's notes.
