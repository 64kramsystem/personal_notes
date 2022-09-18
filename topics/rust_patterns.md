# Rust Patterns

- [Rust Patterns](#rust-patterns)
  - [Builder (emulate named parameters)](#builder-emulate-named-parameters)

## Builder (emulate named parameters)

Use the builder pattern in order to emulate named parameters:

```rs
// Ugly version: `MyTarget.read(filename, true, true)` -> options are unclear!
//
let mut target = MyTarget::new(filename)?;
let val = ReadOptions::builder().option1().read(&mut target)?;
```

With the `derive_builder` crate:

```rs
#[derive(Builder)]
struct Channel {
    token: i32,
    special_info: i32,
}

let ch = ChannelBuilder::default()
    .token(19124)
    .special_info(42)
    .build()
    .unwrap();
```
