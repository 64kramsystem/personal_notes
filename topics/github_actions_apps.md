# GitHub Actions/Apps

- [GitHub Actions/Apps](#github-actionsapps)
  - [Actions (CI)](#actions-ci)
  - [Concepts](#concepts)
    - [Expressions](#expressions)
    - [Conditionals](#conditionals)
    - [Variables](#variables)
    - [Caching](#caching)
  - [Examples](#examples)
    - [Cancel existing actions for the same PR (branch)](#cancel-existing-actions-for-the-same-pr-branch)
    - [tmate debuggin'!](#tmate-debuggin)
    - [Minimal conditional execution](#minimal-conditional-execution)
    - [Run command (e.g. install package)](#run-command-eg-install-package)
    - [Dynamically generate a matrix](#dynamically-generate-a-matrix)
    - [Ruby generic tasks](#ruby-generic-tasks)
    - [Rust Cargo Clippy on multiple targets](#rust-cargo-clippy-on-multiple-targets)
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

### Conditionals

```yaml
# For variables, expressions braces are not required.
#
- name: Use WASM target for Clippy
  if: matrix.config.target == 'wasm32-unknown-unknown'
```

### Variables

Contexts:

- `github.*`: information about the workflow/event ([reference](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context))
  - `github.workspace`: where the repo is checked out
  - `github.event.number`: PR number ([reference](https://github.com/actions/checkout/issues/58#issuecomment-663103947))
  - `github.job`: job id (yaml key value)
  - `github.workspace`: project (workspace) directory
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

### Dynamically generate a matrix

```yml
# The names/references must match:
#
# - `steps.generate-matrix.outputs.XXX`          -> `set-output name=XXX`
# - `needs.generate_projects_matrix.outputs.YYY` -> `outputs: YYY:`
#
generate_projects_matrix:
  runs-on: ubuntu-latest
  outputs:
    matrix: ${{ steps.generate-matrix.outputs.matrix }}
  steps:
  - uses: actions/checkout@v3
  - id: generate-matrix
    # The script must print a JSON document to stdout. It's not specified if it's JSON5, but trailing
    # commas are allowed.
    # The command can be alternatively split into assignment to variable (`VAR=$(...)`), then output
    # setting (`echo ...::$VAR`)
    run: echo ::set-output name=matrix::$(.github/workflows/generate-projects-matrix.sh ${{ github.workspace }})
check_code_formatting:
  runs-on: ubuntu-latest
  needs: generate_projects_matrix
  strategy:
    fail-fast: false
    matrix:
      cfg: ${{ fromJson(needs.generate_projects_matrix.outputs.matrix) }}
  steps:
    - uses: actions/checkout@v3
    - uses: actions-rs/cargo@v1
      with:
        command: fmt
        args: --all --manifest-path=${{ matrix.cfg.port_manifest }} -- --check
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
    branches: [ master ] # Can't use `$default-branch` here!!!
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

### Rust Cargo Clippy on multiple targets

```yml
clippy_correctness_checks:
    runs-on: ubuntu-latest
    name: Clippy correctness checks
    strategy:
      fail-fast: false
      matrix:
        config:
          - { target: 'x86_64-unknown-linux-gnu', target_dir: 'target' }
          - { target: 'wasm32-unknown-unknown', target_dir: 'web-target' }
    steps:
      - uses: actions/checkout@v3
      - name: Use WASM target for Clippy
        if: matrix.config.target == 'wasm32-unknown-unknown'
        uses: actions-rs/toolchain@v1
        with:
            toolchain: stable
            target: ${{ matrix.config.target }}
            components: clippy
      - uses: actions/cache@v3
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
            target/
            web-target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      - uses: actions-rs/cargo@v1
        env:
          CARGO_TARGET_DIR: ${{ matrix.config.target_dir }}
        with:
          command: clippy
          args: --target ${{ matrix.config.target }} -- -W clippy::correctness -D warnings
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
  check_code_formatting:
    runs-on: ubuntu-latest
    name: Check code formatting
    steps:
      - uses: actions/checkout@v3
      - uses: actions-rs/cargo@v1
        with:
          command: fmt
          args: --all -- --check
  clippy_correctness_checks:
    runs-on: ubuntu-latest
    name: Clippy correctness checks
    steps:
      - name: Install dev libraries
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
status = [
  "Build (ubuntu-latest, x86_64-unknown-linux-gnu)", # Names of the jobs, not workflows!
  "Check Rust formatting",
]
```

The CI workflows should also be triggered for `staging` and `trying` branches:

```yaml
push:
  branches:
    - master
    - staging
    - trying
# etc.etc.
```
