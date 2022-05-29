# Rust Tooling

- [Rust Tooling](#rust-tooling)
  - [Cargo/Rustup](#cargorustup)
    - [Workspaces](#workspaces)
    - [Visual Studio Code/Rust Analyzer](#visual-studio-coderust-analyzer)
    - [Debugging information](#debugging-information)
    - [Features](#features)
    - [Conditional build (ifdef-like)](#conditional-build-ifdef-like)
    - [Toolchain](#toolchain)
    - [Build times (profile/improve)](#build-times-profileimprove)
    - [Cross-compilation](#cross-compilation)
    - [Cargo doc](#cargo-doc)
  - [Rustfmt](#rustfmt)

## Cargo/Rustup

Base operations:

```sh
cargo new "$project_name" [--lib]
cargo check                          # check for errors
cargo run
cargo build [--release]              # builds (default: debug); if necessary, updates the crates index, and installs the dependencies
cargo update                         # updates the crates index, and the installed dependencies
cargo test [-- --nocapture]          # pass nocapture in order to output print statements
cargo fmt
cargo clippy                         # linter
cargo doc [--open]                   # builds and optionally opens docs for the installed crates

# Build/run options

--features draw_hitboxes
```

When a project is run via Cargo, the env variable `CARGO_MANIFEST_DIR` is passed to the binary.

Configuration file example:

```toml
# Enable nightly features; `strip` is on 1.46
cargo-features = ["strip"]

[package]
name = "rust"
version = "0.1.0"
authors = ["Saverio Miroddi <saverio.etc@etc.com>"]
edition = "2021"                                           # WATCH OUT! Inappropriate edition may confusingly break projects

# Customize the binary target.
[[bin]]
name = "play"
path = "src/play.rs"

[[test]]
harness = false # Allow Cucumber to print output instead of libtest
name = "cucumber"

[dev-dependencies]
cucumber = {package = "cucumber_rust", version = "^0.7.0"}

[dependencies]
rand = "0.7.3"
redisish = { path = "../redisish" }                         # Relative dependency
amethyst = { git = "https://github.com/amethyst/amethyst" } # Repository; options: `branch`/`tag`/`rev` (master branch is the default)
image = { path = "vendor/image", version = "0.13.0" }       # When both `path` and `version` are specified, `path` is used locally, and `version` publicly

# Override a (indirect) dependency.
# See https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html.
#
[patch.crates-io]
+winit = {git = "https://github.com/rust-windowing/winit.git", rev = "5d85c10a2"}

# Another way to declare a dependency
[dependencies.amethyst]
features = ["vulkan"]
version = "0.15"

# There are 4 profiles: `dev`, `release`, `test`, and `bench`
[profile.release]
strip = true         # highest stripping (equivalent to "symbols")
```

Versioning:

- "~X.Y.Z": Pessimistic versioning
- "X.Y.Z", "^X.Y.Z": WATCH OUT!! 1. They're the same 2. They're not pessimistic, e.g. "^1.2.0" can upgrade to 1.3.0 (but not to 2.0.0)

See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

The cargo configuration file (see custom configs below) `toml` extension is optional; it's searched in many locations (https://doc.rust-lang.org/cargo/reference/config.html), also recursively (also if outside the workspace).

### Workspaces

Workspace: manage multiple projects.

Using cargo from root requires the member name; otherwise, each member can be treated as an individual project.

```toml
# Some settings must be in the workspace configuration when using a workspace, eg. nightly features.
# Add the member before creating the crate.

[workspace]
# This can be globs (?/*).
#
# It can be set to `*`, but will include also `.git` and other files; although they could be excluded
# via `exclude = [".git", ...]`, but it gets awkward (it also doesn't support wildcards).
# In this case, the best layout is to create a directory `crates` and use `members = ["crates/*"]`.
#
members = ["playground", "rust_programming_by_example"]
```

Members must have compatible dependencies, so projects can't be unrelated; without workspaces, it's possible to share artifacts by sharing the target dir (`.cargo/config` -> `[build] target-dir = "/path/to/target`; if relative, it refers to the `.cargo` parent directory).

### Visual Studio Code/Rust Analyzer

As of Apr/2022, the alternative to creating a formal Cargo workspace is to:

- don't add the workspace root directory to the VSC workspace (WATCH OUT!!)
- add each project (sub) directory to VSC
- create one `.vscode/launch.json` for each project
- if there are required files in the root, create a temp directory with symlinks, and add it to VSC and the .gitignore

Trying to add the root to VSC causes problems, firstly, it seems that it's not possible to specify the directory from where Cargo is executed (in `launch.json`), so it runs from the root and fails (since there's no formal workspace).

A trick to avoid contention between Rust Analyzer and terminal builds is to specify a separate target dir:

```json
"rust-analyzer.server.extraEnv": {
    "CARGO_TARGET_DIR": "target-ra"
}
```

### Debugging information

If one needs to debug in release mode, the debug information needs to be retained:

- via rustc, specify `-C debuginfo=2 -C opt-level=3`
- or via Cargo, add `[profile.release] debug = true`

### Features

Define the features of a project.

```toml
[features]
default = ["draw_hitboxes"] // features enabled by default
draw_hitboxes = []          // `[]` is the dependencies
```

### Conditional build (ifdef-like)

The `cfg` attribute performs a conditional build:

```rust
#[cfg(test)]                 // Compile only in test builds
#[cfg(not(test))]            // ... in non-test builds
#[cfg(target_os = "linux")]  // ... on linux
#[cfg(feature = "draw_hitboxes")] // ... with feature enabled
```

The attribute can be applied to methods, statements, etc.

### Toolchain

Set the specified toolchain for the current project:

```sh
# Using rustup; this is stored in rustup's config.
#
rustup override set nightly
rustup override unset # raise error if none is set

# Explicit alternative
#
echo nightly > rust-toolchain
```

The `rust-toolchain` is not read outside a workspace (ie. directory including it). It can't be specified in the Cargo configuration.

### Build times (profile/improve)

Profile build; Graph with breakdown by crate; very simple and builtin:

```sh
cargo build --timings
```

More sophisticated profiling (see: https://blog.rust-lang.org/inside-rust/2020/02/25/intro-rustc-self-profile.html):

```sh
cargo install --git https://github.com/rust-lang/measureme crox flamegraph summarize

rm -f *.mm_profdata

RUSTFLAGS=-Zself-profile cargo build

for f in *.mm_profdata; do
  echo $f

  # Ouput a flamegraph SVG
  #
  # flamegraph $f

  # Print a summary table; filter out below 5%; skip size data
  #
  summarize summarize $f -p 5 | perl -ne 'print if //../^Total/ '

  echo
done
```

Improve build times:

Bevy hello world: 8.75" -> 1.25" (!!).

```sh
# The option "-Zshare-generics=y" is available only in nightly.
#
# The lld linker is considerably faster than the Rust one; the "mold" linker is even faster.
#
# Source: git.io/JsfhD (includes other O/Ss).
# Perf tests [here](https://docs.near.org/docs/community/contribute/contribute-nearcore).
# Rust perf book chapter [here](https://nnethercote.github.io/perf-book/compile-times.html).
#
# Can be verified via `readelf -p .comment target/debug/$binary` (will show `mold...`).

mkdir .cargo
cat > .cargo/config.toml << 'TOML'
[target.x86_64-unknown-linux-gnu]
linker = "/usr/bin/clang"
rustflags = ["-Clink-arg=-fuse-ld=/path/to/mold", "-Zshare-generics=y"]
TOML
```

### Cross-compilation

```sh
# Preparation. Requires Ubuntu package `gcc-mingw-w64-x86-64`
#
rustup target add x86_64-pc-windows-gnu
rustup toolchain install stable-x86_64-pc-windows-gnu

# Cross-compile
#
cargo build --release --target x86_64-pc-windows-gnu

# Alternative: Configure Cargo for cross-compilation:
#
mkdir .cargo
cat > .cargo/config.toml <<TOML
[build]

target = "riscv64gc-unknown-linux-gnu"

[target.riscv64gc-unknown-linux-gnu]

linker = "riscv64-unknown-linux-gnu-gcc"
TOML
```

### Cargo doc

See Programming Rust, chapter 8.

## Rustfmt

Use the attribute macro `#[rustfmt::skip]` in order to disable automatic formatting.

Add a `rustfmt.toml` to the project/workspace root. See: https://rust-lang.github.io/rustfmt (don't forget to consider the channel).

```toml
# Unstable features work only on nightly!
#
unstable_features = true

blank_lines_upper_bound = 10             # unstable
preserve_block_start_blank_lines = true  # unstable; seems to have disappeared (put a comment at the beginning to workaround)
```
