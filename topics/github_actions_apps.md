# GitHub Actions/Apps

- [GitHub Actions/Apps](#github-actionsapps)
  - [Actions (CI)](#actions-ci)
  - [Concepts](#concepts)
    - [Expressions](#expressions)
    - [Variables](#variables)
    - [Caching](#caching)
  - [Examples](#examples)
    - [Cancel existing actions for the same PR (branch)](#cancel-existing-actions-for-the-same-pr-branch)
    - [tmate debuggin'!](#tmate-debuggin)
    - [Minimal conditional execution](#minimal-conditional-execution)
    - [Run command (e.g. install package)](#run-command-eg-install-package)
    - [Ruby generic tasks](#ruby-generic-tasks)
  - [Preset CIs](#preset-cis)
    - [Ruby](#ruby)
    - [Rust](#rust)
  - [Apps](#apps)
    - [Bors](#bors)

## Actions (CI)

In order to receive notifications on workflow completion, see the `Actions` in the [account settings](https://github.com/settings/notifications).

Usage limits are described [here](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration#usage-limits).

## Concepts

For the main example, see [Ruby generic tasks](#ruby-generic-tasks).

WATCH OUT! The configuration files are not legal YAML (!).

### Expressions

Expressions are wrapped in double curly braces: `{{ expression }}`.

### Variables

Contexts:

- `github.*`: information about the workflow/event ([reference](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context))
  - `github.workspace`: where the repo is checked out
  - `github.event.number`: PR number ([reference](https://github.com/actions/checkout/issues/58#issuecomment-663103947))
  - `github.job`: job id (yaml key value)
- `secrets.<SECRET_NAME>`: secrets

There are environment variables, but must check the conditions, e.g. if they persist across jobs ([reference](https://docs.github.com/en/actions/learn-github-actions/environment-variables#default-environment-variables)):

- `env.GITHUB_WORKSPACE`
- `env.GITHUB_ENV`

In order to access variables/secrets from scripts, pass them via env (but predefined env vars are already available):

```yml
env:
  ENV_VAR_NAME: ${{ secrets.SECRET_NAME }}
```

`env` can also be defined in the top scope of the YAML file, which will set it for all the jobs.

In order to set and read variables across jobs, can do the following:

```yml
# Don't do this with process substitution, since failures are not detected!
#
# Reference: https://docs.github.com/en/actions/learn-github-actions/environment-variables#passing-values-between-steps-and-jobs-in-a-workflow
#
- run: echo "MYVAR=1" >> $GITHUB_ENV

# In case of process substitution, append to the github env vars inside the script:
#
# Step 1
# Have the script execute `echo RUN_CURRENT_SUITE=1 >> "$GITHUB_ENV"`
#
- run:  echo "script/ci/test_if_run_suite.sh ${{ github.job }}
# Step 2
- if: env.RUN_CURRENT_SUITE == 1
```

### Caching

WATCH OUT!!! Caching is not shared across branches (PRs), but it is for the same branch, and from master to branches. In order to share caches across all the branches, use a job that builds them on the main branch.

Examples: https://github.com/actions/cache/blob/main/examples.md.

See also [Preset CIs](#preset-cis) in this document.

## Examples

### Cancel existing actions for the same PR (branch)

Use [this](https://github.com/marketplace/actions/cancel-workflow-action):

```yml
jobs:
  cancel-previous-runs:
    runs-on: ubuntu-latest
    steps:
    - name: Cancel Previous Runs
      uses: styfle/cancel-workflow-action@0.9.1
      with:
        access_token: ${{ github.token }}
  other-jobs:
#   ...
```

### tmate debuggin'!

```yml
- name: Setup tmate session
  uses: mxschmitt/action-tmate@v3
```

### Minimal conditional execution

```yml
name: CI

on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Test if current suite is run
      run:  echo "RUN_CURRENT_SUITE=$(script/ci/test_if_run_suite.sh "shellcheck")" >> $GITHUB_ENV
    - name: Run Shellcheck
      if: env.RUN_CURRENT_SUITE == 1
      run: script/ci/run_shellcheck.sh
```

### Run command (e.g. install package)

```yml
- name: Install Alsa dev library
  run: sudo apt install libasound2-dev
```

### Ruby generic tasks

Reference: https://docs.github.com/en/actions/guides/building-and-testing-ruby.

Add a file named `.github/workflows/ci.yml`:

```yml
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
    # Conditionals can also be at step level.
    # When expressions are in an `if` key, curly braces are optional.
    #
    if: matrix.suite == 'rspec
    # See caching section.
    #
    steps:
    - uses: actions/checkout@v3
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

For caching, use the specific action:

```yml
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: ...
    bundler-cache: true
```

## Preset CIs

### Ruby

```yml
# Default branch workflow

on:
  push:
    branches: [ $default-branch ]
jobs:
  build_ruby_cache:
    name: Build Ruby cache
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: ruby/setup-ruby@v1
      with:
        bundler-cache: true

# PR Workflow

# Will automatically run also after the job (to cache the data).
- uses: ruby/setup-ruby@v1
  if: env.RUN_CURRENT_SUITE == 1
  with:
    bundler-cache: true
```

Manual Ruby Bundler caching is discouraged (as it's not trivial); see the [Ruby example](#ruby-generic-tasks).

### Rust

Format/Clippy checks (with caching). WATCH OUT! For unclear reasons, this setup causes Clippy to check also dependencies (!).

```yml
# Default branch workflow

name: Build caches

on:
  push:
    branches: [ $default-branch ]
jobs:
  # Watch out! Clippy (cache) data is shared with build data, but it's not the same.
  # In order to cache build, add a separate job (or extend this).
  build_clippy_cache:
    name: Build Clippy cache
    runs-on: ubuntu-latest
    steps:
    - name: Install dev packages
      run: sudo apt install libasound2-dev libudev-dev
    - uses: actions/checkout@v3
    - uses: actions/cache@v3
      with:
        path: |
          ~/.cargo/bin/
          ~/.cargo/registry/index/
          ~/.cargo/registry/cache/
          ~/.cargo/git/db/
          target/
        key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
    - uses: actions-rs/cargo@v1
      with:
        command: clippy

# PR Workflow

name: CI

on:
  pull_request:

jobs:
  check_formatting:
    runs-on: ubuntu-latest
    name: Check Rust formatting
    steps:
      - uses: actions/checkout@v2
      - uses: actions-rs/cargo@v1
        with:
          command: fmt
          args: --all -- --check
  clippy_correctness_checks:
    runs-on: ubuntu-latest
    name: Clippy correctness checks
    steps:
      - name: Install Alsa dev library
        run: sudo apt install libasound2-dev libudev-dev
      - uses: actions/checkout@v3
      - uses: actions/cache@v3
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      - uses: actions-rs/cargo@v1
        with:
          command: clippy
          args: -- -W clippy::correctness -D warnings
```

## Apps

### Bors

Add a `.github/bors.toml` file, with the content:

```toml
# Workflow file names, without (`yml`) extension; see comment below.
status = ["ci"]
```

It's not clear where the file should be located; possibly, anywhere; typically under `.github`.

It's also not clear what the entries can be exactly; they're described as workflow names, but the examples (also in the wild) use workflow filenames without extensions.
