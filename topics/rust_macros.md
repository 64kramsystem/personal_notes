# Rust Macros

- [Rust Macros](#rust-macros)
  - [General informations](#general-informations)
  - [Declarative macros](#declarative-macros)
    - [Rules and details](#rules-and-details)
    - [Nested iterations](#nested-iterations)
    - [Recursive macros](#recursive-macros)
    - [Importing](#importing)
  - [Procedural/derive macro general informations](#proceduralderive-macro-general-informations)
    - [Quoting](#quoting)
    - [Attributes](#attributes)
  - [Procedural (derive) macros](#procedural-derive-macros)
    - [Project structure](#project-structure)
    - [Macro](#macro)
  - [Attribute macros](#attribute-macros)
    - [Project structure](#project-structure-1)
      - [Macro crate](#macro-crate)
      - [Main crate](#main-crate)
    - [Example (basics)](#example-basics)
    - [Example with field attributes](#example-with-field-attributes)

## General informations

## Declarative macros

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

- `expr`: Expressions that you can write after an `=` sign, such as `76+4` or `if a==1 {"something"} else {"other thing"}`.
- `ident`: An identifier or binding name, such as `foo` or `bar`.
  - WATCH OUT! This can also be used for simple types, like `u64`
- `path`: A qualified path. This will be a path that you could write in a use sentence, such as `foo::bar::MyStruct` or `foo::bar::my_func`.
- `ty`: A type, such as `u64` or `MyStruct`. It can also be a path to the type.
  - WATCH OUT! Has more constraints, when compared to `ident`.
- `pat`: A pattern that you can write at the left side of an `=` sign or in a match expression, such as `Some(t)` or `(a, b, _)`.
- `stmt`: A full statement, such as a `let` binding like `let a = 43;`.
- `block`: A block element that can have multiple statements and a possible expression between braces, such as `{vec.push(33); vec.len()}`, but not a struct or so.
- `item`: What Rust calls items. For example, function or type declarations, complete modules, or trait definitions.
- `meta`: A meta element, which you can write inside of an attribute (`#[]`). For example, `cfg(feature = "foo")`.
- `tt`: Any token tree that will eventually get parsed by a macro pattern, which means almost anything. This is useful for creating recursive macros, for example.
  - Can use `$($all:tt)*` to match anything, but a `tt` alone will _not_ match anything.
- `literal`

Interesting articles:

- Tutorial: https://hub.packtpub.com/creating-macros-in-rust-tutorial
- Case study: https://notes.iveselov.info/programming/time_it-a-case-study-in-rust-macros

### Nested iterations

Declarative macros don't directly support nested iterations.

- there are [some workarounds](https://stackoverflow.com/q/37752133), but they have limits (e.g. a single `tt` can't match a `item`)
- it's possible to use recursion in order to achieve this (see [SO answer](https://stackoverflow.com/a/54552848)).

### Recursive macros

Example:

```rs
macro_rules! rec_macro {
    (
        $($my_types:ty),+; $val:literal
    ) => {
        // If the `ident` type is used instead of `ty`, it's possible _not_ to use separators in the
        // internal calls, which is slightly simpler.
        //
        rec_macro!{@inner $($my_types),+; $val}
    };
    (@inner
        ; $val:literal
    ) => { println!("end") };
    (@inner
        // WATCH OUT!! Notice where the comma is! This is crucial for the recursion.
        //
        $my_type:ty $(, $my_types:ty)*; $val:literal
    ) => {
        let val: $my_type = $val;
        println!("{}", val);
        rec_macro! {@inner $($my_types),* ; $val}
    };
}

rec_macro!(u16, i16; -5); // (correctly) causes an error, due to `let val: u16 = -5`
```

### Importing

Import macro from a module within a crate:

```rs
macro_rules! bail { /* ... */ }

// Trick!
pub(crate) use bail;
```

Export macros across crates (as of Aug/2022, it doesn't work if a crate is `proc-macro`):

```rs
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

## Procedural/derive macro general informations

Two types of macros are relevant to this context/purpose:

- derive macros (`proc_macro_derive(MyTrait)`)
- attribute macros (`proc_macro_attribute`)

Derive macros can't change the passed struct, they can only add other items (e.g. a trait impl), so trying to add a field will result in the struct defined twice; in order to add new fields (but can also add other items), one must use an attribute macro, which returns a new struct definition.

In order to print debug information, set the `extra-traits` feature of the `syn` crate.

WATCH OUT! When generating macros, make sure that the code generated is correct; if not, strange things will happen, e.g. `()` being produced, and/or output being incomplete.

### Quoting

When a variable is quoted, the result depends on the type; if one needs to quote a identifier, `Ident` needs to be used, otherwise, for example `String` is passed, a string (with quotes) will be generated.

In order to quote an iterable (anything `IntoIterator` is supported, e.g. `Vec`), use:

```rs
// Without separator
#(#field_deserialization)*

// With separator.
// WATCH OUT! If quoting multiple statements, embed the separator in the statements; don't use this!
#(#field_deserialization),*
```

### Attributes

Reference: https://docs.rs/syn/latest/syn/struct.Attribute.html.

Field attributes can be accesses via Field#attrs:

```rs
// For easy comparisong
let no_load_attr: syn::Attribute = parse_quote! { #[has_load_progress(none)] };

for attr in &field.attrs {
  if attr == &no_load_attr { /* ... */ }
}

In order to match attributes, one can quote them, then compare:

```rs
```

## Procedural (derive) macros

### Project structure

Procedural macros must be defined in a crate with the crate type of proc-macro.

```sh
cargo new macros --lib

# Convenience if in a workspace
perl -i -pe 's/members = \[\K/"macros", /' Cargo.toml

Append straight below `[dependencies]`:

cat >> macros/Cargo.toml <<TOML
quote = "1.0"
syn = "1.0"

[lib]
proc-macro = true
TOML
```

Macro definitions *must* reside in the root crate.

### Macro

`lib.rs` content:

```rust
// stdlib compiler API that allows manipulating Rust code.
extern crate proc_macro;

// syn converts a string into a data structure to work on; quote does the reverse.
use quote::quote;
use syn;

use proc_macro::TokenStream;

#[proc_macro_derive(MyMacro)]
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

### Project structure

#### Macro crate

Attribute macros must reside in a separate library crate, marked with `proc-macro`.

```sh
cargo new macro_derive --lib # macro

# For some types (eg. ItemStruct), must enable syn's "full" feature.
#
cat >> macro_derive/Cargo.toml << 'TOML'

[lib]
proc-macro = true

[dependencies]
quote = "1.0"
syn = "1.0"
proc-macro2 = "1.0.40"
TOML
```

`lib.rs` contains the module imports/exports.

A bail macro is convenient:

```rs
// macro_derive/src/bail.rs

// The appropriate way to report macro errors is using syn::Error, and converting it into proc_macro2::TokenStream
// via into_compile_error(), then into TokenStream via into() (see below).
//
macro_rules! bail {
    ( $msg:expr $(,)? ) => {
        return ::syn::Result::<_>::Err(::syn::Error::new(::proc_macro2::Span::call_site(), &$msg))
    };

    // Handy rule to get nicely spanned error messages. For example, to reject the struct name being Foo:
    //
    //     if input.ident == "Foo" {
    //         bail! {
    //             "Use a more creative name!" => &input.ident,
    //         }
    //     }
    //
    ( $msg:expr => $spanned:expr $(,)? ) => {
        return ::syn::Result::<_>::Err(::syn::Error::new_spanned(&$spanned, &$msg))
    };
}

pub(crate) use bail;
```

#### Main crate

```sh
cargo new main               # example binary, with trait

cat >> main/Cargo.toml << 'TOML'

[dependencies]
macro_derive = {path = "../macro_derive"}
TOML
```

`main.rs`:

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

### Example (basics)

`lib.rs`:

```rs
mod bail;
use my_actor_based::impl_my_actor_based;
use proc_macro::TokenStream;

// The proc macro public function must be declared inside lib.rs

#[proc_macro_attribute]
pub fn my_actor_based(args: TokenStream, input: TokenStream) -> TokenStream {
    let my_actor_based_impl = impl_my_actor_based(args, input);

    // We don't want to panic! Instead, we convert to TokenStream via `into_compile_error().into()`.
    //
    my_actor_based_impl
        .unwrap_or_else(::syn::Error::into_compile_error)
        .into()
}
```

`my_actor_based.rs`:

```rs
// Reference: https://discord.com/channels/273534239310479360/981857974089814086

// It's good practice to fully qualify (even with leading `::`) all the types.

use proc_macro::TokenStream;
use quote::quote;
use syn::{
    self,
    parse::{self, Parser},
    Data, DataStruct, DeriveInput, Fields,
};

type TokenStream2 = proc_macro2::TokenStream;

fn impl_my_actor_based(
    args: impl Into<TokenStream2>,
    input: impl Into<TokenStream2>,
) -> ::syn::Result<TokenStream2> {
    let mut ast: DeriveInput = ::syn::parse2(input.into())?;
    let _: parse::Nothing = ::syn::parse2(args.into())?;

    add_fields(&mut ast)?;

    let trait_impl = impl_trait(&ast)?;

    Ok(quote!(
        #ast

        #trait_impl
    ))
}

fn add_fields(ast: &'_ mut DeriveInput) -> ::syn::Result<()> {
    if let Data::Struct(DataStruct {
        fields: Fields::Named(fields),
        ..
    }) = &mut ast.data
    {
        let fields_tokens = vec![
            quote! { pub img_base: &'static str },
            quote! { pub img_indexes: Vec<u8> },
            quote! { pub vpos: Vector2<f32> },
            quote! { anchor: Anchor },
            quote! { rectangle_h: Handle<Node> },
        ];

        for field_tokens in fields_tokens {
            // It should be possible to use parse_quote! as a shorthand for parse2(quote!(…)).unwrap(),
            // but it doesn't work; the indication on the chat seems to have a couple of issues.
            //
            let field = syn::Field::parse_named.parse2(field_tokens).unwrap();
            fields.named.push(field);
        }

        Ok(())
    } else {
        bail!("Unexpected input (missing curly braces?)")
    }
}

// Use `Result<ItemImpl>` if you prefer strongly typed (replace `quote!` with `parse_quote!`)
//
fn impl_trait(ast: &'_ DeriveInput) -> ::syn::Result<TokenStream2> {
    #[allow(non_snake_case)]
    let TyName = &ast.ident;
    let (intro_generics, forward_generics, maybe_where_clause) = ast.generics.split_for_impl();

    Ok(quote!(
        impl #intro_generics
            crate::my_actor::MyActor
        for
            #TyName #forward_generics
        #maybe_where_clause
        {
            fn vpos(&self) -> Vector2<f32> {
                self.vpos
            }

            fn vpos_mut(&mut self) -> &mut Vector2<f32> {
                &mut self.vpos
            }

            fn img_base(&self) -> &'static str {
                self.img_base
            }

            fn img_indexes(&self) -> &[u8] {
                &self.img_indexes
            }

            fn anchor(&self) -> Anchor {
                self.anchor
            }

            fn rectangle_h(&self) -> Handle<Node> {
                self.rectangle_h
            }
        }
    ))
}
```

With this division, the code can be unit tested!:

```rs
#[test]
fn basic()
{
  let attr_args = quote!();
  let input = quote!(
      struct Foo {}
  );
  let expected_output = quote!(
      struct Foo { myfield: ::std::string::String }
      impl …
  );

  assert_eq!(
      impl_my_actor_based(attr_args, input).unwrap().to_string(),
      expected_output.to_string(),
  );
}
```

### Example with field attributes

This example implements the `Deserialize` trait, based on a struct fields and enums; it allows specifying a `#[deserialize = "my_fn"]`, which replaces the default `deserialize` implementation.

`deserialize.rs`:

```rs
use crate::collection::{collect_variants_data, find_type_numeric_repr};
use crate::fields_data::{NamedFieldData, VariantData};
use crate::target::Target::ForDeserialization;
use crate::{bail::bail, collection::collect_named_fields_data};

use proc_macro2::Ident;
use quote::quote;
use syn::{self, parse2, Data, DataStruct, DeriveInput, Fields};

type TokenStream2 = proc_macro2::TokenStream;

pub(crate) fn impl_deserialize(input: impl Into<TokenStream2>) -> syn::Result<TokenStream2> {
    let ast: DeriveInput = parse2(input.into())?;
    let type_name = &ast.ident;

    let deserialize_impl = match &ast.data {
        Data::Struct(DataStruct { fields, .. }) => match fields {
            Fields::Named(fields) => {
                let named_fields_data = collect_named_fields_data(fields, ForDeserialization)?;
                impl_trait_with_named_fields(type_name, named_fields_data)?
            }
            Fields::Unnamed(_) => bail!("Unnamed fields not supported!"),
            Fields::Unit => bail!("Unit fields not supported!"),
        },
        Data::Enum(data_enum) => {
            let enum_repr = find_type_numeric_repr(&ast)?;
            let variants_data = collect_variants_data(data_enum)?;
            impl_trait_with_enum_variants(type_name, enum_repr, variants_data)?
        }
        Data::Union(_) => bail!("Unions not supported!"),
    };

    Ok(quote!(
        #deserialize_impl
    ))
}

fn impl_trait_with_named_fields(
    type_name: &Ident,
    fields_data: Vec<NamedFieldData>,
) -> syn::Result<TokenStream2> {
    let fields_deserialization = fields_data.iter().map(
        |NamedFieldData {
             field,
             deserialization_fn,
             ..
         }| {
            let quoted_deserialization_fn = if let Some(deserialization_fn) = deserialization_fn {
                let deserialization_fn =
                    Ident::new(&deserialization_fn.value(), deserialization_fn.span());
                quote! { #deserialization_fn(&mut r) }
            } else {
                quote! { serdine::Deserialize::deserialize(&mut r) }
            };

            quote! { let #field = #quoted_deserialization_fn; }
        },
    );

    let self_fields = fields_data
        .iter()
        .map(|NamedFieldData { field, .. }| quote! { #field, });

    Ok(quote!(
        impl serdine::Deserialize for #type_name {
            fn deserialize<R: std::io::Read>(mut r: R) -> Self {
                #(#fields_deserialization)*

                Self {
                    #(#self_fields)*
                }
            }
        }
    ))
}

fn impl_trait_with_enum_variants(
    type_name: &Ident,
    enum_repr: Ident,
    variants_data: Vec<VariantData>,
) -> syn::Result<TokenStream2> {
    let field_matches = variants_data.iter().map(
        |VariantData {
             variant,
             discriminant,
         }| {
            quote! { #discriminant => Self::#variant, }
        },
    );

    Ok(quote!(
        impl serdine::Deserialize for #type_name {
            fn deserialize<R: std::io::Read>(mut r: R) -> Self {
                let mut buffer = [0; std::mem::size_of::<Self>()];

                r.read_exact(&mut buffer).unwrap();

                match #enum_repr::from_le_bytes(buffer) {
                    #(#field_matches)*
                    value => panic!("Unrecognized value for 'MyEnum' variant: {}", value),
                }
            }
        }
    ))
}
```

`collection.rs`:

```rs
use syn::{DeriveInput, Expr, ExprLit, FieldsNamed, Ident, Lit, Meta, MetaNameValue};

use crate::{
    bail::bail,
    fields_data::{NamedFieldData, VariantData},
    target::Target,
};

const REPR_PATH: &str = "repr";

// ////////////////////////////////////////////////////////////////////////////////
// STRUCT WITH NAMED FIELDS
// ////////////////////////////////////////////////////////////////////////////////

pub fn collect_named_fields_data(
    fields: &FieldsNamed,
    target: Target,
) -> syn::Result<Vec<NamedFieldData>> {
    let mut fields_data = vec![];

    for field in &fields.named {
        // Fields are named, so an ident is necessarily found.
        //
        let mut field_data = NamedFieldData::new(field.ident.clone().unwrap());

        for attr in &field.attrs {
            let attr_meta = match attr.parse_meta() {
                Ok(meta) => meta,
                Err(error) => bail!(error),
            };

            if let Meta::NameValue(MetaNameValue {
                ref path, ref lit, ..
            }) = attr_meta
            {
                // There are different approaches; all a bit odd, but avoid duplicating the rest.
                //
                if path.is_ident(target.attribute_name()) {
                    if let Lit::Str(lit_val) = lit {
                        target.set_serialization_fn(&mut field_data, lit_val.to_owned());
                    } else {
                        bail!(format!(
                            "The `{}` attribute requires a string literal",
                            target.attribute_name()
                        ));
                    }
                }
            }
        }

        fields_data.push(field_data);
    }

    Ok(fields_data)
}

// ////////////////////////////////////////////////////////////////////////////////
// ENUMS
// ////////////////////////////////////////////////////////////////////////////////

pub fn find_type_numeric_repr(ast: &'_ DeriveInput) -> syn::Result<Ident> {
    for attr in &ast.attrs {
        if attr.path.is_ident(REPR_PATH) {
            if let Ok(ident) = attr.parse_args::<Ident>() {
                // It seems that there is no way of natively identifying primitive types, so we must
                // verify manually (see https://stackoverflow.com/q/66906261).

                let ident_str = ident.to_string();
                let mut ident_chars = ident_str.chars();

                let numeric_type = ident_chars.next();

                if matches!(numeric_type, Some('i') | Some('u')) {
                    let type_width = ident_chars.collect::<String>();

                    if type_width.parse::<u8>().is_ok() {
                        return Ok(ident);
                    }
                }
            }
        };
    }

    bail!("Enum repr() not found!")
}

pub fn collect_variants_data(data_enum: &syn::DataEnum) -> syn::Result<Vec<VariantData>> {
    let mut variants_data = vec![];

    for variant in &data_enum.variants {
        let ident = variant.ident.clone();
        let discriminant = if let Some((
            _,
            Expr::Lit(ExprLit {
                lit: Lit::Int(lit_int),
                ..
            }),
        )) = &variant.discriminant
        {
            lit_int.clone()
        } else {
            bail!(format!("'{}' variant discriminant not found!", ident))
        };

        variants_data.push(VariantData::new(ident, discriminant));
    }

    Ok(variants_data)
}
```
