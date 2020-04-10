# Rust

- [Rust](#rust)
  - [Cargo](#cargo)
  - [Syntax/basics](#syntaxbasics)
    - [Basic structure/Printing/Input](#basic-structureprintinginput)
    - [Variables/Data types](#variablesdata-types)
    - [For loop](#for-loop)
    - [If/then/else](#ifthenelse)
  - [APIs](#apis)
    - [Strings conversions/formatting](#strings-conversionsformatting)
    - [Random](#random)

## Cargo

Base operations:

```sh
cargo new "$project_name"
cargo check                          # check for errors
cargo run
cargo build [--release]              # builds (default: debug); if necessary, updates the crates index, and installs the dependencies
cargo update                         # updates the crates index, and the installed dependencies
cargo test
cargo fmt
cargo clippy                         # linter
cargo doc [--open]                   # builds and optionally opens docs for the installed crates
```

Configuration file example:

```toml
[package]
name = "rust"
version = "0.1.0"
authors = ["Saverio Miroddi <saverio.etc@etc.com>"]
edition = "2018"

# Customize the binary target.
[[bin]]
name = "play"
path = "src/play.rs"

[dependencies]
rand = "0.7.3"
```

Versioning is pessimistic by default.

See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

At the root, `Cargo.lock`, managed by Cargo, manages the dependency versions.

## Syntax/basics

### Basic structure/Printing/Input

```rust
use std::io;
use std::io::Write; // bring flush() into scope

// "attributes": metadata with different purposes.
//
#[allow(dead_code)]
fn testing() { }

fn main() {
  println!("Enter guess:");
  io::stdout().flush().unwrap(); // makes sure that the output is flushed, since O/S generally do it per-line.

  // `mut`: mutable.
  // the `new` function is not dictated by the language, but a common practice.
  // static methods are called "associated functions".
  //
  let mut guess = String::new();

  // `&`: reference
  // `&mut` is necessary, if expected, even if the variable is mutable.
  // `read_line()` returns the enum `io::Result`; the "variants" are `Ok` and `Err`.
  //
  io::stdin().read_line(&mut guess).expect("Failed to read guess!");

  // Placeholder: `{}`
  //
  print!("Guess: {}", guess);
}
```

### Variables/Data types

```rust
// Convert data type
//
let guess: u32 = guess.parse().E;
```

### For loop

```rust
for x in 0..10 { }              // [0, 10)
for x in (0..100).rev() {}      // reverse iteration
```

### If/then/else

```rust
if x > 5 {
  println!("{}!", x);
} else if x == 4 {
  println!("{}~", x);
} else {
  println!("{}", x);
}

// If with (multiple) assignment.
//
let (a, b) =
  if true {
    (1, 2)
  } else {
    (3, 4)
  };
```

## APIs

### Strings conversions/formatting

```rust
integer.to_string();                // integer to string
let guess: u32 = string.parse().E;  // string to numeric type

format!("The number is {}", 1);     // the template *must* be a literal (!)
```

### Random

```rust
use rand::Rng;

// `thread_rng()`: seeded by the O/S; local to the current thread.
// `gen_range()`: close,open ends.
//
let secret_number = rand::thread_rng().gen_range(0, 2);
```
