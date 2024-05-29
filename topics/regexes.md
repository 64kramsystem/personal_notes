# Regexes

- [Regexes](#regexes)
  - [Character classes](#character-classes)
  - [Quantifiers (including `*`/`+`)](#quantifiers-including-)
  - [Anchors](#anchors)
  - [Backreferences](#backreferences)
  - [PCRE flags (Perl)](#pcre-flags-perl)
  - [Complex cases](#complex-cases)
    - [Capturing sequences of N characters](#capturing-sequences-of-n-characters)
    - [Negative matches](#negative-matches)
    - [Matching a pattern when it's not at the beginning](#matching-a-pattern-when-its-not-at-the-beginning)
  - [Language incompatibilities](#language-incompatibilities)
    - [Javascript](#javascript)

## Character classes

|   Class   | Extended form |
| :-------: | :-----------: |
| `:alnum:` |  `a-zA-Z0-9`  |
| `:blank:` |     ` \t`     |

## Quantifiers (including `*`/`+`)

Form: `{M}`, `{M, N}`, `{M,}`.

They're greedy by default! The non greedy form is `{M, N}}`.

Don't forget that when used with a capturing group, the group will capture only the last occurrence:

```ruby
" 2 3 4".scan /( \d){2,}/ # => [[" 4"]]
```

WATCH OUT!! When there are group quantifiers, only the last occurrence is captured. Such cases must be transformed to single captures, and the language-specific finder must be used:

```ruby
" a b".match(/( \w)+/)[1..] # => [" b"]
" a b".scan(/( \w)/)        # => [" a", " b"]
```

## Anchors

Reference: https://www.regular-expressions.info/refanchors.html.

WATCH OUT!!! In languages where `^` matches the end of a line (e.g. Ruby), it can also match a newline character:

```ruby
"\nfoo\nbar".scan(/^[^f].+/) # => ["\nfoo", "bar"]
```

## Backreferences

Ruby:

```rb
# In search pattern: `\1`.
#
# WATCH OUT! The referenced group *must* be capturing.
#
'aaabcc'.scan(/((\w)\2*)/) # [["aaa", "a"], ["b", "b"], ["cc", "c"]]
```

Perl:

```sh
# In search pattern: `\g1`.
# Example: Find the `end` of a method, by matching the `def` indentation.
#
perl -i -pe 's/^(\s+)def mymethod$.*?^\g1end$\n\n//sm'

# In replacement pattern: `$1` (or ${1} to disambiguate).
#
perl -i -pe 's/^(ident_file = .*)/#${1}/'
```

## PCRE flags (Perl)

These are Perl; PCRE may have slightly different details:

- `/m`: multiline mode: on 0777, match `^`/`$` on every line
- `/s`: single-line mode: match newlines with `.`
- `/x`: ignore whitespace/comments (in the search expression, not replace!)

WATCH OUT! There are exceptions on some languages; see the specific language.

## Complex cases

### Capturing sequences of N characters

Use a null-character match, followed by lookahead!

The characters in the lookahead need to be in a group, otherwise, they're not captured; the outermost group is purely lookahead syntax.

```ruby
"01234".scan /(?=(\d{3}))/
# [["012"], ["123"], ["234"]]
```

### Negative matches

Prefix: `^(?!prefix)`

Generic pattern: see https://stackoverflow.com/a/406408.

### Matching a pattern when it's not at the beginning

Use a negative lookahead with the beginning-of-input metacharacter, as it's supported in capturing groups:

```ruby
"Re: Re: Re: Pizza".gsub /(?!^)Re: /, '' # "Re: Pizza"
```

The lookbehind version (`/(?<!^)Re: /`) is semantically more precise, however, it's not supported in all JS versions.

## Language incompatibilities

### Javascript

The negative lookbehind (`?<!`) may not be supported in some Javascript versions. Where possible, use the negative lookahead (`?!`).
