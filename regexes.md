# Regexes

- [Regexes](#regexes)
  - [Capturing sequences of N characters](#capturing-sequences-of-n-characters)

## Capturing sequences of N characters

Use a null-character match, followed by lookahead!

The characters in the lookahead need to be in a group, otherwise, they're not captured; the outermost group is purely lookahead syntax.

```ruby
"01234".scan /(?=(\d{3}))/
# [["012"], ["123"], ["234"]]
```
