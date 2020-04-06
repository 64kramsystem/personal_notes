# Rust

- [Rust](#rust)
  - [General program structure](#general-program-structure)
  - [Cargo](#cargo)

## General program structure

```golang
fn main() {
    println!("Hello, world!");
}
```

then execute `cargo run` (compile and run).

## Cargo

Base operations:

```sh
cargo (new <project_name>|run|check)
cargo build [--release]              # defaults to debug version
cargo test
cargo fmt
cargo clippy                         # linter
```

Configuration file example:

```
[package]
name = "rust"
version = "0.1.0"
authors = ["Saverio Miroddi <saverio.etc@etc.com>"]
edition = "2018"

[dependencies]
```

At the root, `Cargo.lock`, managed by Cargo, manages the dependency versions.
