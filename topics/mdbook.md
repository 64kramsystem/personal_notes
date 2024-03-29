# mdBook

- [mdBook](#mdbook)
  - [Basics](#basics)
  - [Configuration](#configuration)
    - [Google analytics](#google-analytics)
  - [Book structure](#book-structure)
    - [`src/SUMMARY.md`](#srcsummarymd)
  - [Code snippets](#code-snippets)
    - [Testing](#testing)
      - [Continuous integration](#continuous-integration)

## Basics

```sh
mdbook init hello-book # asks some questions
cd hello-book

mdbook build           # build
mdbook watch           # watch and rebuild on change

# This will also automatically update the browser pages on change!
#
# WATCH OUT!! If a .gitignore file is not present, changes will not be detected!! (see
# https://github.com/rust-lang/mdBook/issues/1186).
#
mdbook serve --open

# Generate command autocompletions (!)
mdbook completions bash > ~/.local/share/bash-completion/completions/mdbook
```

## Configuration

```toml
[build]
build-dir = "book"
```

### Google analytics

In order to add analytics, add code to `theme/head.hbs`. WATCH OUT!! See Google analytics for the latest snippet code.

```sh
$ mkdir theme                # `src` dir sibling
$ SITE_TAG=G-0000000000      # set here
$ cat > theme/head.hbs <<JS
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=$SITE_TAG"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){window.dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '$SITE_TAG');
</script>
JS
```

## Book structure

- `book.toml`: Some metadata; pregenerated.
- `src/SUMMARY.md`: TOC; manually updated.

`src`'s directory tree structure is replacted in the generated HTML.

The first (topmost) chapter is displayed by default.

### `src/SUMMARY.md`

The top H1 header should always be `# Summary`, although it's optional.

Each numbered chapter is a bullet list; the can be nested:

```md
- [First Chapter](relative/path/to/markdown.md)
  - [Subchapter](relative/path/to/markdown.md)
```

Before the numbered chapters, "prefix" chapters can be added. They're not preceded by the dash, and can't be nested. Example:

```md
[A Prefix Chapter](relative/path/to/markdown.md)
```

Suffix chapters are like prefix ones, but after the numbered chapters.

Draft chapters are essentially placeholders; they don't have a file path:

```md
- [Draft Chapter]()
```

Separators render as lines between the TOC entries; they're defined as 3+ dashes:

```md
- [First Chapter](relative/path/to/markdown.md)
---
- [Second Chapter](relative/path/to/markdown.md)
```

A book can be divided in more sections, which are like individual "sub-books"; they're delimited by an H1 header:

```md
# Part IV
```

## Code snippets

Example:

````
```rust,noplayground
# extern crate mycrate
# fn main() {
println!("Hello!");
# }
```
````

The `main()` function is not required. WATCH OUT! It must be `rust`, not `rs`.

The `#`-prefixed lines are not displayed to the user, but they're included in the compilation.

A playground button is automatically added.

There are many attributes, some are:

- `no_run`       : compiled but not run (implies `noplayground`)
- `noplayground`
- `compile_fail` : compilation should fail

### Testing

Test code snippets via:

```sh
mdbook test -L ~/code/$crate/target/debug/deps`
```

However, this doesn't play well with developer builds (which, for example, include multiple versions); probably, the best strategy is to perform a clean compile, and copy the dependencies to a local subdirectory.

#### Continuous integration

Example build:

```sh
mkdir bin

curl -sSL "$(curl -sSL https://api.github.com/repos/rust-lang/mdBook/releases/latest | jq --raw-output '.assets[] | .browser_download_url' | grep 'linux-gnu.tar.gz$')" | tar -xz --directory=bin

bin/mdbook build
```

Github action (with everything compiled):

```yml
name: Test Code

env:
  CARGO_TERM_COLOR: always

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Update
        run: sudo apt update

      - name: Update Rust
        run: rustup update

      - name: Install Dependencies
        run: sudo apt-get install libasound2-dev libudev-dev pkg-config xorg-dev libxcb-shape0-dev libxcb-xfixes0-dev libxkbcommon-dev

      - name: Install fyrox
        run: git clone https://github.com/FyroxEngine/Fyrox

      - name: Build fyrox
        run: cd Fyrox && cargo build && cd ..

      - name: Setup mdBook
        uses: peaceiris/actions-mdbook@v1
        with:
          mdbook-version: "latest"

      - name: Test Code
        run: mdbook test -L ./Fyrox/target/debug/deps
```
