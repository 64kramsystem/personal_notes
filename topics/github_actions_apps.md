# GitHub Actions/Apps

- [GitHub Actions/Apps](#github-actionsapps)
  - [Actions (CI)](#actions-ci)
  - [Concepts](#concepts)
    - [Expressions](#expressions)
    - [Conditionals/Functions](#conditionalsfunctions)
    - [Variables](#variables)
      - [Contexts](#contexts)
    - [Branch filtering](#branch-filtering)
    - [Caching](#caching)
    - [Labels/conditional jobs run](#labelsconditional-jobs-run)
  - [Examples](#examples)
    - [Cancel existing actions for the same PR (branch)](#cancel-existing-actions-for-the-same-pr-branch)
    - [Block autosquash commits](#block-autosquash-commits)
    - [tmate debugging!](#tmate-debugging)
    - [Conditional jobs run](#conditional-jobs-run)
    - [Run command (e.g. install package)](#run-command-eg-install-package)
    - [Dynamically generate a matrix](#dynamically-generate-a-matrix)
      - [Perform jobs on for directories that have been changed](#perform-jobs-on-for-directories-that-have-been-changed)
    - [Ruby generic tasks](#ruby-generic-tasks)
    - [Run a job conditional to changes to certain files](#run-a-job-conditional-to-changes-to-certain-files)
    - [Rust Cargo Clippy on multiple targets](#rust-cargo-clippy-on-multiple-targets)
  - [Preset CIs](#preset-cis)
    - [Ruby](#ruby)
    - [Rust](#rust)
  - [Apps](#apps)
    - [Kodiak](#kodiak)
    - [Bors](#bors)

## Actions (CI)

In order to receive notifications on workflow completion, see the `Actions` in the [account settings](https://github.com/settings/notifications).

Usage limits are described [here](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration#usage-limits).

WATCH OUT!!

- Jobs need a `:name` field, otherwise, Github won't list them in the status checks.
- The "Branch protection rules" settings don't save automatically!!
- The `uses` field can also use a reference (e.g. commit), not only the version

## Concepts

For the main example, see [Ruby generic tasks](#ruby-generic-tasks).

WATCH OUT! The configuration files are not legal YAML (!).

### Expressions

Reference: https://docs.github.com/en/actions/learn-github-actions/expressions.

Expressions are wrapped in double curly braces: `{{ expression }}`.

### Conditionals/Functions

```yaml
# For variables, expressions braces are not required.
#
- name: Use WASM target for Clippy
  if: matrix.config.target == 'wasm32-unknown-unknown'

- name: Test label
  if: "contains(github.event.pull_request.labels.*.name, 'Tests: Legacy')"
  run: echo "RUN_CURRENT_SUITE=1" >> $GITHUB_ENV
```

Available functions:

- `contains($collection, value)`

### Variables

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

#### Contexts

- `github.*`: information about the workflow/event ([reference](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context))
  - `github.workspace`                       : where the repo is checked out
  - `github.event.number`                    : PR number ([reference](https://github.com/actions/checkout/issues/58#issuecomment-663103947))
  - `github.event.pull_request.labels.*.name`: Use this to get all the label names of a PR
  - `github.job`                             : job id (yaml key value)
  - `github.workspace`                       : project (workspace) directory (full path)
  - `github.ref`                             : current branch, in format `refs/heads/$name`
  - `github.token`                           : same as `secrets.GITHUB_TOKEN`
- `secrets.<SECRET_NAME>`: secrets

There are environment variables, but must check the conditions, e.g. if they persist across jobs ([reference](https://docs.github.com/en/actions/learn-github-actions/environment-variables#default-environment-variables)):

- `env.GITHUB_WORKSPACE`
- `env.GITHUB_ENV`

### Branch filtering

```yml
# https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#on
#
on:
  push:
    # WATCH OUT!! Don't put `$default-branch` on concrete workflows! It's valid only in templates,
    # and it's converted to the branch name on instantiation (see https://github.com/orgs/community/discussions/26597).
    #
    branches: [ master ]
  # Leave without values in order to to disable filtering.
  #
  pull_request:

  # Other examples:
  #
  page_build:
  release:
    types: # This configuration does not affect the page_build event above
      - created

# Simplified form (also single string accepted)
#
on: [push, pull_request]
```

### Caching

WATCH OUT!!! Caching is not shared across branches (PRs), but it is for the same branch, and from master to branches. In order to share caches across all the branches, use a job that builds them on the main branch.

Examples: https://github.com/actions/cache/blob/main/examples.md.

See also [Preset CIs](#preset-cis) in this document.

### Labels/conditional jobs run

WATCH OUT!!! In principle, one could conditionally run jobs like this: `if: "contains(github.event.pull_request.labels.*.name, 'MyTestLabel')"`.

However, this is a problem for devs who create PRs via API; the PR creation API doesn't support labels, so they need to be assigned separately, and even if done immediately, it will cause a race condition, and labels may not be found by the action.

An alternative is use make the workflow trigger on PR labeling (`types: [labeled]`), however, this will trigger one workflow run for each label, and worse, `styfle/cancel-workflow-action` did not cancel previous workflows with such setup.

There are a couple of solutions:

1. use a script to test labels; while this is still subject, in theory, to race conditions, it's slow enough that the labels are found (if assigned immediately); see [example](#conditional-jobs-run).
2. make the workflow run only on non-draft PRs, and have devs create PR as draft, add labels, then unmark as draft.

## Examples

### Cancel existing actions for the same PR (branch)

Use [this](https://github.com/marketplace/actions/cancel-workflow-action):

```yml
jobs:
  cancel-previous-runs:
    runs-on: ubuntu-latest
    if: github.ref != 'refs/heads/master'
    steps:
    - name: Cancel Previous Runs
      uses: styfle/cancel-workflow-action@0.12.1
      with:
        access_token: ${{ github.token }}
  other-jobs:
#   ...
```

### Block autosquash commits

```yml
block-autosquash-commits:
  runs-on: ubuntu-latest
  if: github.ref != 'refs/heads/master'
  steps:
    - name: Block Autosquash Commits
      uses: xt0rted/block-autosquash-commits-action@v2
      with:
        repo-token: ${{ github.token }}
```

### tmate debugging!

```yml
- name: Setup tmate session
  uses: mxschmitt/action-tmate@v3
```

### Conditional jobs run

Add something like this to the related jobs:

```yml
    - name: Test if current suite is run
      run:  script/ci/set_suite_run_flag.sh ${{ github.job }}

# Test via:

      if: env.RUN_CURRENT_SUITE == 1
```

Script, without boilerplate:

```sh
# Format:
#
#   [Label] => "suites"
#
# The suite matching strings (hash values) are regexes; suites are separated by `|`.
#
declare -Ax c_conditional_suites=(
  [Tests: Foo]='pattern1|pattern2'
  [Tests: Bar]='pattern3.+'
)

c_github_api_token=$OWNER:$GITHUB_READ_PR_LABELS_TOKEN                                   # the latter is a secret
c_labels_api_url=https://api.github.com/repos/$ORG/$REPO/issues/$GITHUB_PR_NUMBER/labels # the latter is a built-in

# Find the PR labels, and print one per line. For simplicity, we assume that labels don't have
# newlines (which anyway is not a realistic case).
#
function find_pr_labels {
  # For an example response, see https://docs.github.com/en/rest/issues/labels.
  #
  # jq logic:
  #
  # .[]    : iterate the array
  # {name} : extract the corresponding values of the "name" keys
  # []     : map to an array
  # -r     : print raw (unquoted) strings
  #
  curl -u "$c_github_api_token" "$c_labels_api_url" | jq -r '.[] | {name}[]'
}

function main {
  local current_suite=$1

  while IFS= read -r label; do
    if [[ -v c_conditional_suites[$label] ]]; then
      echo "- Checking label: $label"

      local label_suites=${c_conditional_suites[$label]}

      if echo "$current_suite" | grep -qP "^($label_suites)$"; then
        echo "  - Running suite"
        echo "RUN_CURRENT_SUITE=1" >> "$GITHUB_ENV"
        exit
      else
        echo "  - Skipping suite"
      fi
    else
      echo "- Ignoring label: $label"
    fi
  done <<< "$(find_pr_labels)"
}

main "$1"
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
  - uses: actions/checkout@v4
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
  # Important!! Otherwise, an error is raised on empty matrices.
  if: ${{ needs.generate_projects_matrix.outputs.matrix != '[]' }}
  steps:
    - uses: actions/checkout@v4
    - uses: actions-rs/cargo@v2
      with:
        command: fmt
        args: --all --manifest-path=${{ matrix.cfg.port_manifest }} -- --check
```

#### Perform jobs on for directories that have been changed

This can be performed via a convenient action.

```yml
# WATCH OUT! Must include fetch-depth, otherwise the commit is not usable for diffing.
# Possibly, a depth of 2 (smaller size) works for squash merges.
#
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- name: Get changed files
  id: changed-files
  uses: tj-actions/changed-files@v31
  with:
    # Newlines are better, but are currently unsupported (https://github.com/tj-actions/changed-files#known-limitation);
    # commas are equally safe for this project, though.
    separator: ","
    dir_names: true
- id: generate-matrix
  run: echo ::set-output name=matrix::$(.github/workflows/generate-projects-matrix.sh ${{ steps.changed-files.outputs.all_changed_files }})
```

Script (simplified):

```sh
# Filters in the project directories, located under the workspace root, which include a `Cargo.toml`
# file; prints a JSON5 document with an array of `{ "port_manifest": "<relative_port_dir>/Cargo.toml" }`
# entries.
#
# - we assume that we don't need quoting;
# - we include only the root directories, if they include a `Cargo.toml` file;
# - the JSON is actually JSON5, since a trailing comma is present inside the array; screw JSON.

# Input dirs, relative to workspace root, separated by ','.
mapfile -td, changed_dirs <<< "$1"

echo -n "["

for changed_dir in "${changed_dirs[@]}"; do
  if [[ $changed_dir != . && $(dirname "$changed_dir") == . ]]; then
    local cargo_file=$changed_dir/Cargo.toml

    if [[ -f $cargo_file ]]; then
      echo "  { \"port_manifest\": \"$cargo_file\" },"
    fi
  fi
done

echo -n "]"
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
    branches: [ master ]
  pull_request:

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
    # Optional (allowed to fail) job.
    # For simple jobs, just set to true.
    # `debug` is only to show the functionality - it's not included above.
    #
    continue-on-error: ${{ endsWith(matrix['ruby-version'], 'head') || matrix['ruby-version'] == 'debug' }}
    # Conditionals can also be at step level.
    # When expressions are in an `if` key, curly braces are optional.
    #
    if: matrix.suite == 'rspec'
    # See caching section.
    #
    steps:
    - uses: actions/checkout@v4
    - run: echo "💡 The ${{ github.repository }} repository has been cloned to the runner."
    # WATCH OUT!!! Creates a `vendor` directory, with gems.
    #
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
        GITHUB_TOKEN: ${{ github.token }}
      run: bundle exec rspec
```

For caching, use the specific action:

```yml
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: ...
    bundler-cache: true
```

### Run a job conditional to changes to certain files

```yml
check-default-gems:
  runs-on: ubuntu-22.04
  permissions:
    contents: read            # may be necessary only on private repos
    pull-requests: read
  steps:
  - uses: actions/checkout@v4
  - uses: dorny/paths-filter@v3
    id: changes
    with:
      filters: |
        gem_changes:
          - '.ruby-version'
          - 'Gemfile.lock'
  - name: Run Default Gems Check
    run: script/ci/run_${{ github.job }}.rb
    if: steps.changes.outputs.gem_changes == 'true' # format: "steps.<id>.outputs.<filter>"
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
      - uses: actions/checkout@v4
      - name: Use WASM target for Clippy
        if: matrix.config.target == 'wasm32-unknown-unknown'
        uses: actions-rs/toolchain@v1
        with:
            toolchain: stable
            target: ${{ matrix.config.target }}
            components: clippy
      - uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
            target/
            web-target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      - uses: actions-rs/cargo@v2
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
    branches: [ master ]
jobs:
  build_ruby_cache:
    name: Build Ruby cache
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
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

The cache is indexed by `Cargo.lock`; it may not be appropriate for all type of projects.

```yml
# Default branch workflow

name: Build caches

on:
  push:
    branches: [ master ]
jobs:
  # Watch out! Clippy (cache) data is shared with build data, but it's not the same.
  # In order to cache build, add a separate job (or extend this).
  build_clippy_cache:
    name: Build Clippy cache
    runs-on: ubuntu-latest
    steps:
    - name: Install dev packages
      run: sudo apt install libasound2-dev libudev-dev
    - uses: actions/checkout@v4
    - uses: actions/cache@v4
      with:
        path: |
          ~/.cargo/bin/
          ~/.cargo/registry/index/
          ~/.cargo/registry/cache/
          ~/.cargo/git/db/
          target/
        key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
    - uses: actions-rs/cargo@v2
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
      - uses: actions/checkout@v4
      - uses: actions-rs/cargo@v2
        with:
          command: fmt
          args: --all -- --check
  clippy_correctness_checks:
    runs-on: ubuntu-latest
    name: Clippy correctness checks
    steps:
      - name: Install dev libraries
        run: sudo apt install libasound2-dev libudev-dev
      - uses: actions/checkout@v4
      - uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      - uses: actions-rs/cargo@v2
        with:
          command: clippy
          args: -- -W clippy::correctness -D warnings
```

## Apps

### Kodiak

- Merge by labelling with `automerge`
- In order to update branches (e.g. semi-linear history), must enable `Require branches to be up to date before merging`
- At least one status check must be enabled

```sh
cat > .github/.kodiak.toml << 'TOML'

version = 1
merge.method = "rebase_fast_forward" # semi-linear history
TOML
```

See:

- https://kodiakhq.com/docs/config-reference
- https://kodiakhq.com/docs/features

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
