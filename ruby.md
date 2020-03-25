## Table of contents

- [General syntax](#general-syntax)
  - [Heredoc](#heredoc)
- [Misc](#misc)
  - [System](#system)
  - [Convert curl request to Ruby](#convert-curl-request-to-ruby)

## General syntax

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

## Misc

### System

Current user run dir, on Linux:

```ruby
ENV.fetch('XDG_RUNTIME_DIR')
```

### Convert curl request to Ruby

See https://jhawthorn.github.io/curl-to-ruby.
