# Ruby

- [Ruby](#ruby)
  - [General syntax](#general-syntax)
    - [Array literals](#array-literals)
    - [Heredoc](#heredoc)
  - [APIs/Stdlib](#apisstdlib)
    - [Array](#array)
    - [CGI/URI (encoding)](#cgiuri-encoding)
  - [Handling processes](#handling-processes)
    - [Basic handling, via `IO.popen`](#basic-handling-via-iopopen)
    - [Using `IO.popen3`](#using-iopopen3)
    - [Live execution, via `Kernel#system`](#live-execution-via-kernelsystem)
    - [Backticks](#backticks)
  - [Misc](#misc)
    - [System](#system)
    - [Convert curl request to Ruby](#convert-curl-request-to-ruby)

## General syntax

### Array literals

```ruby
%i(foo bar)           # array of symbols
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

## APIs/Stdlib

### Array

```ruby
arr = [0, 1, 2]

arr.fill(nil, arr.size...5)             # resize/extend (destructive) in arguably expressive form; returns the array; !!! watch out the `...` syntax !!!
arr[5] ||= nil                          # other resize/extend, in arguably less expressive form
```

### CGI/URI (encoding)

URI is already in scope.

```ruby
CGI::escape("&&&")                            # URL-encode a string: "%26%26%26"
URI.encode_www_form(p1: "&&&", "p2" => "!!!") # URL-encode params: "p1=%26%26%26&p2=%21%21%21"

CGI.unescapeHTML("html")	      # decode HTML entities; use only for basic cases, as it' not 100% complete (gem: https://github.com/threedaymonk/htmlentities)
HTMLEntities.new.decode("html")	# htmlentities gem
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

### Backticks

Backticks behave like `IO.popen`: The stdout content is captured, stderr content is instead printed immediately.

## Misc

### System

Current user run dir, on Linux:

```ruby
ENV.fetch('XDG_RUNTIME_DIR')
```

### Convert curl request to Ruby

See https://jhawthorn.github.io/curl-to-ruby.
