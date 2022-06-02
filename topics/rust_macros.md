# Rust Macros (WIP)

- [Rust Macros (WIP)](#rust-macros-wip)
  - [Function-like](#function-like)
    - [Rules and details](#rules-and-details)
    - [Importing](#importing)
  - [Custom derive macros](#custom-derive-macros)
  - [Attribute macros](#attribute-macros)

## Function-like

Simple, fixed expressions:

```rust
// Capture the `expr`ession as `$name`.
//
macro_rules! simple_print {
    ($name:expr) => {
        println!("Hey {}", $name)
    };
}

// `envar!(setValue: "/home/me" forKey: "HOME")`
//
macro_rules! envar {
    (setValue: $value:tt forKey: $key:tt) => {
        println!("setValue:{} forKey:{}", $value, $key);
    };
}

// Multiple expressions for one macro:
//
// - `add!(one to 5)`
// - `add!(two to 5)`
//
macro_rules! add {
    (one to $input:expr) => {
        $input + 1
    };
    (two to $input:expr) => {
        $input + 2
    };
}
```

Macros can also be invoked using both round and square brackets.

Multiple occurrences. Metachars are:

- `*`: 0+
- `+`: 1+
- `?`: 0 or 1 (doesn't take a separator)

```rust
// The character preceding `*` is the separator.
//
macro_rules! repeated_print {
    ($( $name:expr ),*) => {
      $( println!("Hey {}", $name); )*
    };
}

// This way repeats only the expressions.
//
macro_rules! build_and_print_array {
  ($( $item:expr ),*) => {
    let myvec = vec![$( $item ),*];
    println!("{:?}", myvec);
  };
}

// Accepts expressions with the Ruby hash (rocket) syntax.
// Since we return a value, we need to define a scope (extra curly braces).
//
macro_rules! ruby_hash {
  ($( $key:expr => $value:expr ),*) => {
    {
      let mut hm = HashMap::new();
      $( hm.insert($key, $value); )*
      hm
    }
  };
}

// Simulate an optional parameter.
// Different use of the comma here; it's consumed as part of the expression, but the expanded one is
// the one defined by the repetition.
//
macro_rules! optional_param {
  ($mandatory:expr $(, $optional:expr)*) => {
    println!("M:{} O:{}", $mandatory, $($optional),*)
  };
}

// Simulate optional, named, parameters.
// For simplicity, the comma after each expression is mandatory.
//
//     test_cpu_execute!(
//         zf:80=>81,
//         hf:82=>83,
//     );
//
macro_rules! test_cpu_execute {
  (
    $( zf: $zf_pre_value:literal => $zf_post_value:literal, )?
    $( nf: $nf_pre_value:literal => $nf_post_value:literal, )?
    $( hf: $hf_pre_value:literal => $hf_post_value:literal, )?
    $( cf: $cf_pre_value:literal => $cf_post_value:literal, )?
  ) => {
    $( println!("zf: {} -> {}", $zf_pre_value, $zf_post_value); )?
    $( println!("nf: {} -> {}", $nf_pre_value, $nf_post_value); )?
    $( println!("hf: {} -> {}", $hf_pre_value, $hf_post_value); )?
    $( println!("cf: {} -> {}", $cf_pre_value, $cf_post_value); )?
  };
}
```

Other usages:

```rust
// Define a function (!!).
// If we don't define `$name` as `ident`, but for example as `expr`, the compiler will complain that
// an identifier is expected.
//
macro_rules! generate_func {
  ($name:ident) => {
    fn $name(param: i32) {
      println!("Value: {}", param);
    }
  };
}

generate_func!(foo);
foo(123); // `Value: 123`

// Define a struct.
//
macro_rules! matrix {
  ($name:ident, $order: literal) => {
    pub struct $name {
      pub values: [[u32; $order]; $order],
    }
  };
}

// Define variables (!)
//
macro_rules! vars {
  ($data:expr, $stride:expr, $var1:ident, $var2:ident, $var3:ident) => {
    let $var1 = $data[0];
    let $var2 = $data[1 * $stride];
  };
}

// Receive a statement; use a variable defined inside the macro
//
macro_rules! test_sort {
  ($collection:ident, $stat:stmt) => {
    #[cfg(test)]
    mod tests {
      use super::*;

      #[test]
      fn test_sort() {
        let mut $collection = &mut vec![3, 2, 1];
        $stat
        let expected_collection = &vec![1, 2, 3];
        assert_eq!($collection, expected_collection);
      }
    }
  };
}

test_sort!(collection, bubble_sort(collection));
```

### Rules and details

There are some rules:

- Statements and expressions can only be followed by `=>`, a comma, or a semicolon
- (other)

Macro variable types:

`expr`: Expressions that you can write after an `=` sign, such as `76+4` or `if a==1 {"something"} else {"other thing"}`.
`ident`: An identifier or binding name, such as `foo` or `bar`.
`path`: A qualified path. This will be a path that you could write in a use sentence, such as `foo::bar::MyStruct` or `foo::bar::my_func`.
`ty`: A type, such as `u64` or `MyStruct`. It can also be a path to the type.
`pat`: A pattern that you can write at the left side of an `=` sign or in a match expression, such as `Some(t)` or `(a, b, _)`.
`stmt`: A full statement, such as a `let` binding like `let a = 43;`.
`block`: A block element that can have multiple statements and a possible expression between braces, such as `{vec.push(33); vec.len()}`.
`item`: What Rust calls items. For example, function or type declarations, complete modules, or trait definitions.
`meta`: A meta element, which you can write inside of an attribute (`#[]`). For example, `cfg(feature = "foo")`.
`tt`: Any token tree that will eventually get parsed by a macro pattern, which means almost anything. This is useful for creating recursive macros, for example.
`literal`

Interesting articles:

- Tutorial: https://hub.packtpub.com/creating-macros-in-rust-tutorial
- Case study: https://notes.iveselov.info/programming/time_it-a-case-study-in-rust-macros

### Importing

Sample of importing macros across files:

```rust
// helpers.rs
//
[macro_export]
macro_rules! test_sort {}

// main.rs
//
mod helpers;

// tests.rs
//
use crate::test_sort;
test_sort!(collection, bubble_sort(collection));
```

Macros are pulled in the main scope, so the `use` must not be followed by the macro enclosing module (`helpers`).

## Custom derive macros

Procedural macros must be defined in a crate with the crate type of proc-macro.

```sh
cargo new macros --lib

# Convenience if in a workspace
perl -i -pe 's/members = \[\K/"macros", /' Cargo.toml

 Append straight below `[dependencies]`.

cat >> macros/Cargo.toml <<TOML
quote = "1.0"
syn = "1.0"

[lib]
proc-macro = true
TOML
```

`lib.rs` content; macro definitions *must* reside in the root crate:

```rust
// stdlib compiler API that allows manipulating Rust code.
extern crate proc_macro;

// syn converts a string into a data structure to work on; quote does the reverse.
use quote::quote;
use syn;

use proc_macro::TokenStream;

[proc_macro_derive(MyMacro)]
pub fn my_macro_derive(input: TokenStream) -> TokenStream {
    // Replace unwrap() with better error handling on production code.
    let ast: syn::DeriveInput = syn::parse(input).unwrap();

    let name = &ast.ident;

    // stringify() converts an expression to string literal (&str), not String!
    let gen = quote! {
        // in order to import entities, rebind them
        use std::sync::Mutex as HelloMacroMutex;

        impl HelloMacro for #name {
            fn my_macro() {
                println!("{} my_macro() implementation via custom derive attribute!", stringify!(#name));
            }
        }
    };

    gen.into()
}
```

Now, from the client crate, just add the dependency (assumed to be in the same workspace):

```toml
macros = {path = "../macros"}
```

include in the project:

```rust
use macros::MyMacro;

// or (must be at the crate root)

[macro_use]
extern crate macros; // name of the crate
```

then use!

## Attribute macros

Macros must reside in a separate library crate, marked with `proc-macro`.

```sh
cargo new macro_derive --lib # macro
cargo new main               # example binary, with trait

# For some types (eg. ItemStruct), must enable syn's "full" feature.
#
cat >> macro_derive/Cargo.toml << 'TOML'

[lib]
proc-macro = true

[dependencies]
quote = "1.0"
syn = "1.0"
TOML

cat >> main/Cargo.toml << 'TOML'

[dependencies]
macro_derive = {path = "../macro_derive"}
TOML
```

Main:

```rs
use hello_macro_derive::*;

pub trait MyTrait { fn print_my_field(&self); }

#[whole_my_trait]
struct MyStruct {}

fn main() {
    let instance = MyStruct { myfield: "foo".to_string() };
    instance.print_my_field();
}
```

Macro!

Two types of macros are relevant to this context/purpose:

- derive macros (`proc_macro_derive(MyTrait)`)
- attribute macros (`proc_macro_attribute`)

Derive macros can't change the passed struct, they can only add other items (e.g. a trait impl), so trying to add a field will result in the struct defined twice.  
In order to add new fields (but can also add other items), one must use an attribute macro, which returns a new struct definition.

```rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{
    self,
    parse::{self, Parser},
    parse_macro_input, Data, DataStruct, DeriveInput, Fields,
};

#[proc_macro_attribute]
pub fn base_actor(args: TokenStream, input: TokenStream) -> TokenStream {
    let mut ast: DeriveInput = syn::parse(input).unwrap();
    let _ = parse_macro_input!(args as parse::Nothing); // Make sure no arguments are passed

    if let Data::Struct(DataStruct {
        fields: Fields::Named(fields),
        ..
    }) = &mut ast.data
    {
        let fields_tokens = vec![
            quote! { img_base: &'static str },
            quote! { img_indexes: Vec<u8> },
        ];

        for field_tokens in fields_tokens {
            fields
                .named
                .push(syn::Field::parse_named.parse2(field_tokens).unwrap());
        }
    } else {
        panic!("Unexpected input (missing curly braces?)");
    }

    let name = &ast.ident;

    let gen = quote! {
        #ast

        impl MyTrait for #name {
            fn print_my_field(&self) {
                println!("myfield: {}", self.myfield);
            }
        }
    };

    gen.into()
}
```
