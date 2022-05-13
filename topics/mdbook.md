# mdBook

- [mdBook](#mdbook)
  - [Basics](#basics)
  - [Book structure](#book-structure)
    - [`src/SUMMARY.md`](#srcsummarymd)
  - [Continuous integration](#continuous-integration)

## Basics

```sh
mdbook init hello-book # asks some questions
cd hello-book
mdbook serve --open

mdbook build           # build
mdbook watch           # watch and rebuild on change

# Generate command autocompletions (!)
mdbook completions bash > ~/.local/share/bash-completion/completions/mdbook
```

## Book structure

- `book.toml`: Some metadata; pregenerated.
- `src/SUMMARY.md`: TOC; manually updated.

`src`'s directory tree structure is replacted in the generated HTML.

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

## Continuous integration

Example build:

```sh
mkdir bin

curl -sSL "$(curl -sSL https://api.github.com/repos/rust-lang/mdBook/releases/latest | jq --raw-output '.assets[] | .browser_download_url' | grep 'linux-gnu.tar.gz$')" | tar -xz --directory=bin

bin/mdbook build
```