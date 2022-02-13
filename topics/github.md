# GitHub

- [GitHub](#github)
  - [Searches](#searches)
  - [Ruby CI action](#ruby-ci-action)
    - [Variables](#variables)
    - [Caching](#caching)
    - [Minimal example](#minimal-example)
  - [Rust CI action](#rust-ci-action)
  - [Coauthorship (multiple authors)](#coauthorship-multiple-authors)
  - [README.md](#readmemd)
    - [Images](#images)
    - [Refer to project root](#refer-to-project-root)

## Searches

Sample search using multiple topics and a language:

- GUI: `language:rust topic:hacktoberfest topic:virtualization`
- HTTP: https://github.com/search?l=Rust&q=topic%3Ahacktoberfest+topic%3Avirtualization&type=Repositories

## Ruby CI action

Reference: https://docs.github.com/en/actions/guides/building-and-testing-ruby.

Add a file named `.github/workflows/ci.yml`:

```yml
# Note that this is not legal YAML (!)
#
name: Ruby CI

# https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#on
#
on:
  push:
    branches: [ $default-branch ]
  # Leave without values in order to to disable filtering.
  #
  pull_request:
  #
  # Other examples:
  #
  # page_build:
  # release:
  #   types: # This configuration does not affect the page_build event above
  #     - created

# Simplified form (also single string accepted)
#
# on: [push, pull_request]
#

jobs:
  test:
    # This doesn't guarantee a specific version, may run on the beforelast version.
    #
    runs-on: ubuntu-latest
    # https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstrategy
    #
    strategy:
      # Important! If true (default), failure of one job will stop all the other jobs.
      #
      fail-fast: false
      # In order to base the matrix on env vars, add `env: [K1=V1, ...]`, and add it to the step (see
      # below).
      #
      matrix: # optional
        os: [ubuntu, macos]
        ruby-version: [head, 3.0, 2.9, 2.8, 2.7, 2.6, 2.5]
    # `debug` is only to show the functionality - it's not included above.
    #
    continue-on-error: ${{ endsWith(matrix.ruby, 'head') || matrix.ruby == 'debug' }}
    # See caching section.
    #
    steps:
    - uses: actions/checkout@v2
    - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
    - name: Setup Ruby ${{ matrix.ruby-version }}
      uses: ruby/setup-ruby@v1
      if: ${{ matrix.suite == 'rspec' }}
      with:
        ruby-version: ${{ matrix.ruby-version }} # not needed if .ruby-version is present and used
        bundler-cache: true
    - name: Install dependencies
      run: bundle install
    - name: Run tests
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: bundle exec rspec
```

### Variables

- `github.workspace`: where the repo is checked out
- `env.GITHUB_WORKSPACE`: don't use; for unclear reasons, it's blank, even after `checkout@v2` has run

### Caching

Base caching:

```yml
- uses: actions/cache@v2
  with:
    path: |
      ~/.cargo/registry
      ~/.cargo/git
      target
    key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
```

Ruby bundler caching is not trivial, so there's a specific action:

```yml
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: ...
    bundler-cache: true
```

Other examples: https://github.com/actions/cache/blob/main/examples.md.

### Minimal example

```yml
name: CI

on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run shellcheck
      run: ci/run_shellcheck.sh
```

## Rust CI action

Format/Clippy checks:

```yml
jobs:
  fmt:
    runs-on: ubuntu-latest
    name: Formatting
    steps:
      - uses: actions/checkout@v2
      - name: Check Rust formatting
        uses: actions-rs/cargo@v1
        with:
          command: fmt
          args: --all -- --check
  # To improve: cache installation
  clippy:
    runs-on: ubuntu-latest
    name: Linting
    steps:
      - uses: actions/checkout@v2
      - name: Install Clippy
        run: rustup component add clippy
      - name: Run Clippy
        run: cargo clippy
```

## Coauthorship (multiple authors)

Github's specification for coauthorship is to add each additional author in the commit message, in the format `Co-authored-by: Name <email>`. This is supported also by Gitlab.

## README.md

### Images

Display an image (WATCH OUT! This won't be rendered from external sites):

```md
![Highlighted block rendering](/readme_images/hightlighted_block_rendering.png?raw=true)
```

Image (raw) path: `https://github.com/64kramsystem/vscode-markdown-code-blocks-asm-syntax-highlighting/blob/master/pizza/test.png?raw=true`

### Refer to project root

Prefix the path with `/../../` (see [Stack Overflow](https://stackoverflow.com/a/40440270/210029))
