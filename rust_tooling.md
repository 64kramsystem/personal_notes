# Rust Tooling

- [Rust Tooling](#rust-tooling)
  - [Cargo/Rustup](#cargorustup)
    - [Features](#features)
    - [Conditional build (ifdef-like)](#conditional-build-ifdef-like)
    - [Toolchain](#toolchain)
    - [Fast builds](#fast-builds)
    - [Cross-compilation](#cross-compilation)
    - [Cargo doc](#cargo-doc)
  - [Rustfmt](#rustfmt)
  - [Packaging](#packaging)
    - [Project structure (with modules)](#project-structure-with-modules)
    - [Modules (details)](#modules-details)

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
edition = "2018"

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

# Another way to declare a dependency
[dependencies.amethyst]
features = ["vulkan"]
version = "0.15"

# There are 4 profiles: `dev`, `release`, `test`, and `bench`
[profile.release]
strip = "symbols"
```

Workspace: manage multiple projects.
Using cargo from root requires the member name; otherwise, each member can be treated as an individual project.

```toml
# Some settings must be in the workspace configuration when using a workspace, eg. nightly features.
# Add the member before creating the crate.

[workspace]
members = ["playground", "rust_programming_by_example"]
```

Versioning is pessimistic by default.

See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

At the root, `Cargo.lock`, managed by Cargo, manages the dependency versions.

The cargo configuration file (see custom configs below) `toml` extension is optional; it's search in many locations: https://doc.rust-lang.org/cargo/reference/config.html.

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

### Fast builds

Bevy hello world: 8.75" -> 1.25" (!!).

```sh
# If using rust stable, remove the "-Zshare-generics=y" below.
# Requires lld. Source: git.io/JsfhD (includes other O/Ss).

mkdir .cargo
cat > .cargo/config.toml << 'TOML'
[target.x86_64-unknown-linux-gnu]
linker = "/usr/bin/clang"
rustflags = ["-Clink-arg=-fuse-ld=lld", "-Zshare-generics=y"]
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

Add a `rustfmt.toml` to the project root. See: https://rust-lang.github.io/rustfmt (don't forget to consider the channel).

```toml
# Unstable features work only on nightly!
#
unstable_features = true

blank_lines_upper_bound = 10             # unstable
preserve_block_start_blank_lines = true  # unstable
```

## Packaging

### Project structure (with modules)

A package (=set of 1+ crates) can contain at most one library crate.

Crate "roots":

- `src/main.rs` -> binary crate (same name as package)
- `src/lib.rs` -> library crate (same name as package)

An option to put small binary crates inside a crate is to puth them in `src/bin`:

- `src/bin/mycrate.rs` -> binary crate (named `mycrate`)

this doesn't require addition to `Cargo.toml`; it's included in the `build`; in order to run, add `--bin mycrate`.

Alternative configuration for multiple crates, via `Cargo.toml`:

```toml
# Array of tables -> there can be multiple.
#
[[bin]]
name = "daemon"
path = "src/daemon/bin/main.rs"
```

The relative (to the project root) source file path can be found via `file!()`.

Modules structure (comments are content):

```
crate_name
├── Cargo.toml
└── src
    ├── main.rs              // pub mod mod1; pub mod mod2
    ├── main_a.rs
    ├── mod1
    │   ├── mod.rs           // pub mod mod1_a; pub mod mod1_b
    │   ├── mod1_a.rs
    │   └── mod1_b.rs
    ├── mod2
    │   ├── mod2_a.rs
    └── mod2.rs              // pub mod mod2_a
```

`mod2` has an alternative structure (modules defined in a corresponding `.rs` file).

### Modules (details)

All items (including modules) and their children are private by default, but:

- children can access their parents' items;
- public enums' items are public.

```rust
mod front_of_house {
  // Public module; doesn't make the *contents* public.
  //
  pub mod hosting {
    // Can see stuff in `front_of_house`.
    //
    pub fn add_to_waitlist() {}
  }
}

pub fn eat_at_restaurant() {
  // Relative path.
  // With this tree, the function invoked must be public.
  //
  front_of_house::hosting::add_to_waitlist();

  hosting::add_to_waitlist();
}
```

Modifiers (prefixes):

- `crate`: absolute path
- `super`: parent module (like current directory)
- `self`

Security levels:

- `pub`, private, `pub(crate)`
- `pub(super)`: visible to parents only
- `pub in <crate>`: visible to to a specific parent module and descendants

Multiple files/directories structure:

```rust
// This loads either `module.rs` or `module/mod.rs`.
//
mod <module>;
```

Importing:

```rust
// The item imported must be public. Using doesn't make the item public!
// In order to import relatively, must (currently) use `self`.
//
// It's unidiomatic to import functions, while it's idiomatic to import enums, structs, etc.
//
use crate::front_of_house::hosting;

// Import sibling modules in sibling files (shape.rs). The first assumes no reexport; the second assumes
// a flattening reexport.
//
use super::shape::Shape;
use super::Shape;

// Solutions to clashing; both iditiomatic.
//
use std::io::Result as Ioresult;
use std::io; // and reference `io::Result`

// "Re-exporting"; allows referencing `hosting::add_to_waitlist`. Useful when the whole path is not
// meaningful for the clients.
//
pub use crate::front_of_the_house::hosting;

// Disambiguate; this refers to a `image` crate
//
use ::image::Pixels;

// Other use syntaxes
//
use std::io::{self, Write}
use std::collections::*; // useful for testing; unidiomatic for the rest
```
