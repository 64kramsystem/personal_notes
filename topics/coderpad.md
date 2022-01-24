# Coderpad

## Key bindings

- Ctrl+Enter : Runs code
- Ctrl+D     : Add next match to selection
- Ctrl+Space : Autocompletion

## Settings

- autocompletion parentheses/brackets (WATCH OUT! doesn't complete quotes/do-end)

## Available gems

Included by default where specified.

### rspec (not included)

```rb
require "rspec/autorun"

describe { ... }
```

### pry

```rb
binding.pry # no require required
```

- `ls`       : print vars in current scope
- `whereami` : show surrounding source code
- `p $expr`  : print expression
- `wtf?`     : backtrace of last exception
- `exit`     : return to program

### Others

- active_support
- [algorithms](https://github.com/kanwei/algorithms)
