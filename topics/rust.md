# Rust

- [Rust](#rust)
  - [Basic structure/Printing/Input](#basic-structureprintinginput)
    - [Printing/formatting/write!](#printingformattingwrite)
  - [Data types](#data-types)
    - [Casting/conversions](#castingconversions)
    - [`.` operator](#-operator)
  - [Functions/Closures](#functionsclosures)
  - [Collections](#collections)
    - [Iterators](#iterators)
      - [Ranges and iterable APIs](#ranges-and-iterable-apis)
      - [Method chaining](#method-chaining)
    - [Arrays/Vectors/Slices](#arraysvectorsslices)
      - [Shared Vec/array methods](#shared-vecarray-methods)
    - [Hash maps](#hash-maps)
      - [HashSet](#hashset)
      - [Manual hashing](#manual-hashing)
    - [Other data structures](#other-data-structures)
  - [Arithmetic/math APIs](#arithmeticmath-apis)
  - [Strings](#strings)
    - [String/Slice/Char-related APIs/conversions](#stringslicechar-related-apisconversions)
    - [Chars iterator](#chars-iterator)
    - [Internal representation (bytes/chars/graphemes)](#internal-representation-bytescharsgraphemes)
  - [Control flow](#control-flow)
  - [Enums](#enums)
    - [Option](#option)
      - [Convenient Option patterns](#convenient-option-patterns)
    - [Result, convient APIs, and interop with Option](#result-convient-apis-and-interop-with-option)
    - [Convert to/from numeric](#convert-tofrom-numeric)
  - [Error handling](#error-handling)
    - [Basic patterns](#basic-patterns)
    - [Complex error handling](#complex-error-handling)
  - [Pattern matching](#pattern-matching)
  - [Unsafe](#unsafe)
    - [Interoperability with other languages (C)](#interoperability-with-other-languages-c)
  - [Structs](#structs)
    - [Associated functions/methods](#associated-functionsmethods)
    - [`self` in methods](#self-in-methods)
    - [Operator overloading](#operator-overloading)
    - [Method overloading (workaround)](#method-overloading-workaround)
    - [Unions](#unions)
  - [Generics](#generics)
    - [Const Generics](#const-generics)
  - [Traits](#traits)
    - [Basics (and Generics #2)](#basics-and-generics-2)
    - [Approaches to collections and static/dynamic dispatching](#approaches-to-collections-and-staticdynamic-dispatching)
    - [Supertraits and inheritance (object-orientation)](#supertraits-and-inheritance-object-orientation)
      - [Inheritance: making overridden methods private](#inheritance-making-overridden-methods-private)
    - [Trait limitations/workarounds](#trait-limitationsworkarounds)
    - [Downcasting/Upcasting](#downcastingupcasting)
    - [Iterator trait/Associated types/impl trait](#iterator-traitassociated-typesimpl-trait)
  - [Ownership](#ownership)
    - [Move](#move)
    - [Borrowing](#borrowing)
    - [Dangling pointers](#dangling-pointers)
    - [Lifetimes](#lifetimes)
      - [Elision rules](#elision-rules)
    - [Slices](#slices)
  - [Smart pointers](#smart-pointers)
    - [Box<T>](#boxt)
    - [RC<T>](#rct)
    - [Cell<T>/RefCell<T>/Mutex<T> and interior mutability](#celltrefcelltmutext-and-interior-mutability)
    - [Modifying Rc/Arc without inner mutable types (and conversion Box -> RC type)](#modifying-rcarc-without-inner-mutable-types-and-conversion-box---rc-type)
    - [Weak<T> and reference cycles](#weakt-and-reference-cycles)
      - [`Rc<RefCell>` or `RefCell<Rc>`](#rcrefcell-or-refcellrc)
      - [Real case of modeling a thread-safe tree with trait objects, with children addition](#real-case-of-modeling-a-thread-safe-tree-with-trait-objects-with-children-addition)
      - [Real complex case of iterating a recursive structure with Rc/RefCell](#real-complex-case-of-iterating-a-recursive-structure-with-rcrefcell)
  - [Multithreading](#multithreading)
    - [Channels: Multiple Producers Single Consumer](#channels-multiple-producers-single-consumer)
    - [Simple Multiple Producers Multiple Consumes](#simple-multiple-producers-multiple-consumes)
    - [Mutex<T>/Arc<T>](#mutextarct)
    - [Atomic primitive type wrappers](#atomic-primitive-type-wrappers)
    - [Barrier](#barrier)
    - [Condvar](#condvar)
    - [Busy waiting/spin loops (pause)](#busy-waitingspin-loops-pause)
  - [Project structure](#project-structure)
    - [Prelude structure](#prelude-structure)
    - [Modules (details)](#modules-details)
  - [Assorted insanity](#assorted-insanity)

## Basic structure/Printing/Input

```rust
// "attributes": metadata with different purposes.
// with the `!`, they are at crate level (must be in the root crate); without, they are at method
// level (place them immediately above the method definition).
//
#![allow(dead_code)]
#![allow(unused_variables)]
#![allow(unused_imports)]
#![allow(unused_assignments)]
#![allow(unused_must_use)]
#![allow(unused_mut)]

use std::io;
use std::io::Write; // bring flush() into scope

// Unused parameters can be named `_`.
//
fn testing(n: u32) -> String {
  if n > 10 {
    panic!("Error message!");
  }
  // In order to return a value without using `return`, omit the semicolon.
  //
  String::from("abc")
}

// The return value is optional. This specific one is convenient [for testing].
//
fn main() -> Result<(), Box<dyn Error>> {
  print!("Enter guess: ");
  io::stdout().flush().unwrap(); // makes sure that the output is flushed, since O/S generally do it per-line.

  // `mut`: mutable.
  // the `new` function is not dictated by the language, but a common practice.
  //
  let mut buffer = String::new();

  // Appends to the buffer.
  //
  let bytes_read = io::stdin().read_line(&mut buffer)?;

  // Placeholder: `{}`
  //
  println!("Guess: {}", guess);

  // See fn return value.
  //
  Ok(());
}
```

"Divergent" functions are functions that never return, marked with `!` return type; exiting from them is considered an error:

```rust
fn cycle_forever() -> ! {
  loop { /* ... */ }
}
```

### Printing/formatting/write!

WATCH OUT! Don't forget to bring `std::io::Write` into scope, when using write-related traits.

```rust
println!("{:#?}", vec);                 // generic pretty printing
println!("{:?}", vec);                  // `Debug` format (requires the `Debug` trait; if generic, requires `use std::fmt::Debug`)

eprintln!("Error!");                    // print on stderr!

// The placeholder types can be mixed (named ones must be last).
//
format!("The number is {}", 1)                                // "positional" pl.; the template *must* be a literal (!)
format!("The number is {0:.2}, again {0:.2}, not {1}!", 1, 2) // indexed placeholders!
format!("The number is {value:.2}", value=1.0)                // "named" placeholders!

writeln!(writer, "{}", 123).unwrap();   // write formatted data into a Write implementor; `write()` also available
```

Formatting (see https://doc.rust-lang.org/std/fmt):

```rust
"{:.2}"             // round float
"{:x}/{:X}"         // lower/upper hex
"{:b}"              // binary (via trait std::fmt::Binary)
"{:p}"              // pointer

"{:5}"              // padding (right align)
"{:05}"             // padding with char (zero)

"{:<5}"             // left align
"{:^5}"             // center align
"{:>5}"             // right align

"{:-^5X}"           // pad with char, align, and convert ot hex
```

In order to escape a curly brace, duplicate it.

In order to convert bytes to a hex string, use:

```rust
use core::fmt::Write;
let mut buffer = String::with_capacity(2 * array.len());
for byte in buffer {
  write!(s, "{:02X}", byte)?;
}
```

Sample Debug implementation:

```rust
impl Debug for SortableFloat {
  fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
    write!(f, "{}", self.0)
  }
}
```

## Data types

SVs differ from constants:

- constants can be duplicated in memory;
- SVs can be mutable (in this case, they're unsafe).

Integer types:

- `[iu](8|16|32|64|128|size)`
  - the `size` ones depend on the architecture
  - the default, if not specified and can't be inferred, is `i32`

Max value functions (example): `u64::max_value()`, `u64::MAX`

Floats types: `f(32|64)`.

Integer literals:

```rust
0xff      // hex
0o77      // octal
0b1100    // binary
b'A'      // byte; only `u8`; require single quotes
```

All number literals support the type as suffix (e.g. `32u8`), and the (cosmetic) underscore.

Extra string literals:

```rust
let bytestr: &[u8] = b"hello";      // byte string (use `as_bytes()` to convert from string types)
let rawstr: &str = r"hello";        // raw string (doesn't process escapes)
let byterawstr: &[u8] = br"hello";  // byte raw string

// Any number of hashes can be added to the raw string delimiters, to disambiguate the ending one:

let zzz = br#"foo"bar"#;
let zzz = br##"foo"#bar"##;         // with a single delimiting hash, the first `"#` is considered terminating
```

Char literals:

```rust
'🤯'          // 4 bytes, require single quotes
```

Escapes ([Tokens](https://doc.rust-lang.org/reference/tokens.html) subset):

```rust
"\u{0304}"    // 24-bit Unicode character code (up to 6 digits); valid also as char
```

Tuples:

```rust
let foo = ("bar", "baz");
let (mut bar, mut baz) = foo;     // multiple assignment (unpacking); foo can also be a tuple literal
let first_element = tuple.0;      // tuple indexing

// Use a tuple as function argument; borrowing is an option.
//
fn multiply(dimensions: &(u32, u32)) -> u32 {
  dimensions.0 * dimensions.1
}

multiply(&(2, 3));
```

It's not possible to iterate tuples, because order or type consistency are not guaranteed!

For strings, see the [Strings chapter](#strings).

Memory-related operations:

```rs
std::mem::swap(&mut a, &mut b); // !! swap two values (variables, array entries, etc.) !!
std::mem::size_of::(Type)       // memory occupation of a type
std::mem::size_of_val(v)        // memory occupation of a variable !!
```

### Casting/conversions

```rust
let int_as_float = (10 as f64);     // type casting

const MAX_PRIMES: u32 = 100000;     // constants; the data type is required

((1u128 << CONST_U64) - 1) as u64   // WATCH OUT the priorities! In this example, the brackets are all required!

type Kilometers = i32;                     // type aliasing (typedef)
type Result<T> = Result<T, std::io:Error>; // library example: `std::io::Result`

let bool_as_int = true as i32;      // true: 1, false: 0
let int_as_bool = 1 as bool;        // 1: true, 0: false, other: !!undefined!!
```

For static variables, see [static section](rust_libraries.md#staticglobal-variables-lazy_static-once_cell-thread_local)

Number-related casts/operations:

```rust
0xFF_u8 as u16;          // 0x00FF ("zero-extend")
  -1_i8 as u16;          // 0xFFFF ("signed-extend")

0xFF_u8 as i16;          // WATCH OUT!!: 0x00FF
(0xFF_u8 as i8) as i16;  // 0xFFFF

1.to_string();             // numeric to string

0_u32.to_be_bytes();                          // convert big endian u32 to array of bytes
u32::from_le_bytes([u8; _])                   // convert big endian array of bytes to u32
u32::from_le_bytes(&[u8].try_into().unwrap()) // same, from slice
f64.to_bits()                                 // transmute to u64 (for bit-wise ops)
f64::from_bits(u64)                           // transmute from u64 (for bit-wise ops)
f64::is_sign_positive()                       // important! we can't use `-0.0 >= 0.0` for comparison (if sign is important)

integer.to_string();                      // integer to string
char.to_digit(radix).unwrap();            // convert (ASCII) char to number
char::from_digit(u32, radix);             // convert (number, radix) to char
1_u8 as char;                             // only u8 can be casted to char
std::char::from_u32(1_u32);               // cast u32 to char (not available for other data types)

// Convenient conversion traits
FromStr#from_str(s: &str)                 // convert &str to any (implementing) type
ToString#to_string()                      // supertrait of Display; it's preferrable to implement Display, though

// UTF-8 conversions
String::from_utf8(vec)?                   // requires valid input; also `str`
String::from_utf8_lossy(&byte)            // replaces invalid chars with `�`
String::from_utf8_unchecked(&byte)        // assumes valid input; also `str`

// Parse string to numeric type; with any numeric implementing `FromString`
// f64 will parse integer strings (e.g. `1`).
// WATCH OUT! Whitespace is not accepted, so it must be trimmed.
//
let guess: u32 = string.trim().parse().unwrap();
```

Automatic casting, for types supporting the `Deref` trait:

- `&String` -> `&str`
- `&Vec<T>` -> `&[T]`
- `&Box<T>` -> `&T`

In order to implement arbitrary casting, implement the [`Into<T>` trait](rust_libraries.md#from-into).

### `.` operator

```rs
// Automatic dereferencing; any number is followed.
//
let r = &&&myobj;
println!("{}", r.field);

// Automatic "referencing" (due to `sort(&mut self)`)
//
let v = vec![1, 2];
v.sort();

// Auto dereferencing on comparison is legal as long as the depth is equivalent
//
&&myobj == &&myobj; // valid
&&myobj == &myobj;  // invalid

// WATCH OUT!! `.` is higher priority than unary operators:
//
-1.method(2) // => interpreted as `-(1.method(2))`
```

## Functions/Closures

Equivalent of Ruby blocks!

```rs
#[must_use]
fn retval() -> u32 { 1 };                       // `#[must_use]` causes a warning, if the retval is not used
retval();                                       // <-- warning!

fn unpack(In(val): In<u32>) { }                 // Unpacking can be performed also at parameter level (!!)

let multiple_of_10 = |x: i32| { x % 10 == 0 };  // yay! braces and type annotation are optional
(0..100).any(multiple_of_10);                   // double yay!

// When closures refer to variables in the outer scope, they borrow them; alternatively, they can `move`
//
let key_fn = move |city| city.print_stat(stat);

// Generic closure signature. The closure types passed don't need to be annotated.
//
struct Calculator<T: Fn(u32) -> u32> {
  calculation: T,
}

// Function accepting a closure.
//
pub fn with_player_mut<F>(&mut self, player_h: Handle<Player>, mut f: F)
where F: FnMut(&mut Player),
{
    let player = self.players.borrow_mut(player_h);
    f(player)
}
```

Functions can be assigned to variables (function pointers); their type is `fn` (don't confuse with `Fn`!!). However, they can't reference the context, and functions defined inside a function can't be assigned to a variable.

```rust
fn sum_fn(x: i32) -> i32 {
  x
}

fn main() {
  let y = 10;

  fn sum_fn(x: i32) -> i32 { x + y };     // Invalid

  let my_fn = sum_fn;                     // Valid

  fn return_value_fn(x: i32) -> i32 { x } // Valid

  return_value_fn(1);
}

// Syntax to pass methods (both instance/static):

fn cycle_decode(&self) -> (fn(&Self, u8), u8) {
  let cycle_execute = Self::cycle_execute_foo;
  (cycle_execute, 0)
}

let (cycle_execute, x) = self.cycle_decode();
cycle_execute(self, x);
```

If the error `expected fn pointer, found fn item` is raised, explictly define the fn type (seems related to git.io/JGz2L).

Because of the capturing, closures have overhead compared to functions.

Closures can use function pointers, including initializer functions!!:

```rust
[0, 1, 2, 3].iter().map(ToString::to_string);   // Associated method `to_string()` of the `ToString` trait
[0, 1, 2, 3].iter().map(Some);                  // Initializer function of the `Some` enum
```

The compiler performs the "Deref coercion", if required - essentially, a series of dereferencings (following the `Deref` trait):

```rust
fn hello_slice(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let pizza = Box::new(String::from("abc"));
    hello_slice(&pizza);
}
```

in the above case, both the box dereference, and the `String` deference that turns `&String` into `&str`.

Closure can have three traits, which are inferred:

- `FnOnce`: take ownership (which can't be taken more than "once")
- `FnMut`: borrow mutably
- `Fn`: borrow immutably

In order to return closures from a function, they must be boxed:

```rust
fn my_closure() -> Box<dyn Fn(i32) -> i32> {
  Box::new(|x| x + 1)
}
```

`const` closures [can't be defined](https://stackoverflow.com/questions/29191170/is-there-any-way-to-explicitly-write-the-type-of-a-closure).

## Collections

### Iterators

#### Ranges and iterable APIs

General form: `[start] .. [[=]end]`. Technically they're `Iterator`s of types `Range(Inclusive|From|To|Full)?`.

For `Iterator` details, see the [related section](#iterator-traitassociated-typesimpl-trait).

Iterators are lazy; !! they implement arithmetic operators !!

`std::iter::Iterator` methods (also implemented by Range).

```rust
map(|x| x * 2)               // Ruby :map
map(|(x, y)| x + y)          // Tuples unpacking: useful for example, on the result of zip()
flat_map(|x| x)              // Ruby :flat_map. WATCH OUT! flattens only one level.
fold(a, |a, x| a + x)        // Ruby :inject; there is `rfold()`
try_fold(a, |a, x|)          // Like fold(), but uses Result; on Ok, follows up, on Err, returns it immediately; there is `try_rfold()`.
fold_first(|a, x| a + x)     // Like fold(), using the first element as initial value
filter(|x| x % 2 == 0)       // Ruby :select
filter_map(|x| Some(x * 2))  // AWESOME!!! Combines filter and map; None values are discarded
[r]find(|x| x % 2 == 0)      // Ruby :find/:detect
find_map(|x| cond(x).then(y)) // Like find(); stops when the block returns Some<T>
[r]position(|x| x == 2)      // Ruby :[r]index(&block); None if not found
rev()                        // reverse. WATCH OUT, UNINTUITIVE: since it's not inclusive, it goes from 99 to 0.
nth(n); nth_back()           // nth element (0-based)
last()
take(n); skip(n)             // iterator taking/skipping the first n elements; use nth() if choosing only one element
next()                       // Ruby :first
take_while(||)               // stops iteration, by returning always false after the first false
skip_while(||)               // start iteration on the first true, returning all the others
enumerate()                  // iterator (index, &value) (Ruby :each_with_index)
zip(iter)                    // zip two arrays (iterators); stops at the shortest iterator
drain(range)                 // removes and returns the elements in the range
fuse()                       // ensures that after the first None, any other iteration returns None

// Partition (divide a collection in two (true, false)). Remember that:
// 1. must specify `let (a, b): (Vec<_>, Vec<_>) = ...`
// 2. don't forget `iter_mut()` when needed; if the types are complex, the errors are unreadable.
// 3. for complex types, intermediate steps can be confusing, and the compiler may require types; in
//    this case, temporarily use intermediate typed vectors.
partition(|x| x % 2 == 0)

// Math
// For each max() there is a min()
max().unwrap()               // Compute max, on types implementing `Ord`; with variations `_by(||)` and `_by_key(value)`
sum()                        // WATCH OUT! Returns the same type, so conversion is needed, e.g. `.map(|&x| x as u32).sum();`
product()                    // ^^ same as above

// `_by` and `by_key` variations example
// WATCH OUT!! the comparison is a<>b, not the reverse!
max_by(|v1, v2| v1.x.cmp(&v2.x)).unwrap()      // Custom max (the closure returns `Ordering`; see [Sorting Floats](rust_libraries.md#sorting-floats))
max_by_key(|v| v.field)                        // Easier version of max
col.sort_by(|a, b| a.partial_cmp(b).unwrap()); // sort with closure 😍!

// Iterator comparisons (!); supported: lt, le, eq, etc.
iterator1.lt(iterator2)

// Boolean inspection
any(|x| x == 33)             // terminates on the first true
all(|x| x % 2 == 0)          // terminates on the first false

// Manual implementation of chunks(2), where not available
coll.iter().step_by(2).zip(coll.iter().skip(1).step_by(2))

// Quasi-Ruby :flatten. WATCH OUT! flattens only one level.
// !! Since None() evaluates to empty iterator, invoking on an Iterator<Option> performs Ruby :compact !!
// Same applies to an Iterator<Result>
// In order to flatten tuples, map them to arrays first.
flatten()

// If one wants to convert an iterator from borrowed (&T) to owned (T), use copied() or cloned().
// Example, from [f64; _] to Vec<f64>:
myarr_f64.iter().flatten().copied().collect::<Vec<_>>();
myarr_f64.iter().flatten().cloned().collect::<Vec<_>>();

// Transform an iterator into a collection ("consume")
// Can use Iterator#size_hint() for optimization purposes.
collect()
collect::<Vec<i32>>()
collect::<Vec<_>>()

// Create an iterator for:
//
iter::empty()                  // empty
iter::once(x)                  // single entry
iter::repeat(x)                // repeating a value
iter::repeat(x).take(n)        // create n repeating values
iter::repeat_with(|| func())   // repeate a function (Ruby's `Array.new() {}` )

// Fancy, but impractical, way of creating a duplicated array (Ruby's `Array.new() {}`); `collect()`
// is necessary because the `RepeatWith` iterator doesn't implement `TryInto`.
// The simplest way is just to make Foo a Copy and instantiate the array directly.
//
let _arr: [Foo; 10] = iter::repeat_with(|| Foo { v: 1 })
    .take(10)
    .collect::<Vec<_>>()
    .try_into()
    .unwrap();

// Simplified version of fold(), where each item is based on the last value. Example with multiples
// of 10:
//
iter::successors(Some(1), |n| n.checked_mul(10))

// Example of creating an iterator via from_fn()
// Use take(n) if want to use size as limitation, instead of Option.
//
let mut count = 0;
let counter = std::iter::from_fn(move || {
    count += 1;
    if count < 6 { Some(count) } else { None }
});

// Other adapters:
peek()                 // Returns a peekable iterator (`peek() -> Option<T>`)
chain(other_iter)      // Appends iterators
inspect(|v|)           // Executes the function, without affecting the chain (useful for debugging)
by_ref()               // Converts the iterator to mutable references (e.g. `String.lines().by_ref()...`)
cycle()                // Endless repeating

// Ruby :times can be emulated via ranges.
// Range supports only the `into_iter()` iterator, but no iterator invocation is actually needed.
// There is `[try_]for_each()`, but they provide no advantage over a standard iteration.
//
(0..SHARKS_COUNT).map(|x| 2 * x).collect::<Vec<_>>();
(0..SHARKS_COUNT).for_each(|x| println!("{}", x)); // for_each() returns no values

// !! Option and Result are iterators !!
// This is very clever, as it explains some neat iterator APIs design
//
Some(v), Ok(v) // iterator with v
None, Err(e)   // empty iterator
```

Array/Vec "block" APIs:

```rs
// `_mut()` variations are available.

// split() typical additional variations: `r...`, `...n`.
split(|x| myfn(x))
split_at(); split_first(); split_last()

chunks(n)                    // iterate in chunks of n elements; includes last chunk, if smaller
chunks_exact(n)              // (preferred) iterate in chunks of n elements; does not include the last chunk, if smaller
windows(n)                   // like chunks, but with overlapping slices (Ruby :each_cons); doesn't have variations
```

#### Method chaining

Chain iterators, AREL-style:

```rust
pub fn compose_iterator<T>(elements: &Vec<T>, reverse: bool) {
    let mut base_iter = elements.iter();
    let mut rev_iter;

    let mut composed_iter: &mut dyn Iterator<Item = &T> = &mut base_iter;

    if reverse {
        rev_iter = base_iter.rev();
        composed_iter = &mut rev_iter;
    }

    // ...
}
```

Additional iterators can be added following the same `rev_iter` pattern.

### Arrays/Vectors/Slices

Arrays (immutable, so they're allocated on the stack):

```rust
let arr = [1, 2, 3];
let arr = [true; 4];                     // 4 elements initialized as true; won't work with variable size (use a Vec)
let arr: [u32; 3] = [1, 2, 3];           // with data type annotation; ugly!
let mut arr: [Option<u32>; 3] = [None; 3];  // with Option<T>; super-ugly!

// In order to store mixed types, use a tuple. Example of pseudo-bidimensional array:
//
let mtarr = [
  ((1.0, 2.0), -1, "cde"),
  // ...
];

// If a struct has an array, it's possible to initialize the array implicitly by using the [Default](#default) trait on the struct.

arr[512..512 + source.len()].copy_from_slice(&source) // memcpy (copy) from/to slices/vectors; source/dest size must be the same!
slice.fill(value)                       // memset; can be used with arrays or slices (eg. partial fill)
[1, 2, 3] == [1, 2, 4]                  // arrays can be compared via `==`

arr.to_vec()                            // convert array to vector

// invocation: process_list(&my_list)
//
fn process_list(list: &[i32]) {}

// For unpacking, see `if let`
```

Watch out!! Very large arrays will overflow the stack; use `Vec` in such cases.

Vectors (mutable):

```rust
let mut vec = Vec::new();               // Basic (untyped) instantiation (if the type can be inferred)
let mut vec: Vec<i32> = Vec::new();     // Basic, if type can't be inferred
let mut vec = vec![1, 2, 3];            // Macro to initialize a vector from a literal list; allocates exact capacity
let mut vec = vec![true; n];            // Same, with variable-specified length and initialization; allocates exact capacity
(0..5).collect();                       // Create vector from range
vec.fill_with(|| rand::random())        // Set vector values via closure; uses len()
vec.fill(1);                            // Set vector values via Clone; uses len()

// Size-optimization APIs
Vec::with_capacity(cap);                // Preallocating version; WATCH OUT! The length is still 0 at start; use `vec![<val>, n]` if required
vec.reserve(n)
vec.reserve_exact(n)
vec.shrink_to_fit(n)

let val = &vec[0];
vec[0] = 2;

// Safe versions; they also have a `_mut` version
vec.get(2);                             // Safe (Option<T>) version
vec.first();
vec.last();

vec.push(1);                            // Push at the end
vec.pop();                              // Pop from the end
vec.remove(0);                          // Ruby's `Array#shift()`. WATCH OUT! Panics if index is out of bounds

vec.extend([1, 2, 3].iter().copied());  // Append (concatenate) an array
vec.extend(&[1, 2, 3]);                 // Append (borrowing version) an array/vec
vec[range].copy_from_slice(&source);    // memcpy; see array example

// Convert vector to array (same len). If type annotations are required, use the second form:
// In order to convert an iterator to array, collect to Vec first.
//
vec.try_into().unwrap();
TryInto::<&[f64; 3]>::try_into(vec);

// Convert an array of a wider type (e.g. u32) to a vec/array of narrower type (e.g. u8) (two approaches)
//
source.iter().flat_map(|v| v.to_le_bytes()).collect::<Vec<_>>();
//
let mut dest = [0; 16];
for (dest_c, source_e) in dest.chunks_exact_mut(4).zip(source.iter()) {
    dest_c.copy_from_slice(&source_e.to_le_bytes())
}

// Vectors can be received as array reference type (mutable, if required).
//
fn process_list<T>(list: &[T]) {};
process_list(&vec);
```

In order to unpack a vector, convert to slice and use if let (see [pattern matching](#pattern-matching)).

Element(s) conditional removal:

```rust
// Remove one match; ignore if element not found.
//
if let Some(i) = vec.iter().position(|vec_item| *vec_item == remove_item) {
  vec.remove(i);
}

// Remove all matching elements (Ruby delete_if(), but with opposite condition)
//
vec.retain(|vec_item| *vec_item != remove_item);
```

Arrays implement the `Debug` trait.

Using an Enum to store different data types in an array!!:

```rust
enum CsvRow {
  Int(i32),
  Float(f64),
};

let mut vec = vec![];

vec.push(CsvRow::Int(1));
vec.push(CsvRow::Float(2.0));
```

Slices apply to arrays as well:

```rust
let array = [8u32; 5];          // type is [u32; _]
let slice = &array[..];         // type is `&[u32]`
let slice = &array[n..][..4]    // arguably convenient form of [n..n+4]
```

See the [ownership chapter](#ownership), for the related properties.

#### Shared Vec/array methods

```rust
col.swap(pos1, pos2)
col1.swap(&mut col2)                    // Swatp the contants of two slices (must have same len)
let old = col.swap_remove(i);           // Fast removal (fills in with the last element)
std::mem::swap(&mut new, &mut col[0])   // Replace an entry with another
col.resize(new_len, value: T);          // resize (extend/shrink) a vector; T must be Clone
col.resize_with(new_len, || expr);      // resize, via function (e.g. when T is not Clone)

// Splitting/joining
(sl1, sl2) = coll.split_at(split_point);     // immutable
(sl1, sl2) = coll.split_at_mut(split_point); // mutable: the two arrays are subarrays of the source
coll2 = coll.split_off(split_point);         // mutable: the returned (sub)array is removed from the source
join("str");                                 // join using str; for char[], see String [chapter](#stringslicechar-related-apisconversions)
concat();                                    // join without separator

col.len();
col.iter();                             // iterator

// Sorting (destructive!) and finding
col.sort()                              // stable; with variations `_by(||)` and `_by_key(value)`
col.sort_unstable()                     // unstable (typically faster than stable)
col.binary_search(&value)               // with variations `_by(||)` and `_by_key(value)`
col.contains(value)                     // linear search
col1.starts_with(col2)
col1.ends_with(col2)

// In order to deduplicate (Ruby :uniq), collect into a HashSet, or sort_unstable() + dedup()
//
dedup();                                // Deduplicate consecutive entries, with variations `_by(|a, b|)` and `_by_key(|k|)`
```

In order to pick a random element, see [rand crate](#random-with-and-without-rand).

### Hash maps

The default hashing function is cryptographically secure!!, and slow(er). See (rust_ruby_libraries.md#faster-hashing-ahash-phf).

WATCH OUT! In order to avoid BCK headaches when getting and setting in the same scope/function, use the [Entry API](https://stackoverflow.com/a/28512504/210029) (see below).

```rust
use std::collections::HashMap;

// When choosing between owned/borrowed, keeping in mind that if elements are created in a function
// and returned, ownership is necessary.
//
let mut map = HashMap::new();

map.insert("b", 10); // if the value was existing, `Some(old_value)` is returned; otherwise, `None`
map.remove("b");

// Getters, and basic mutable access to values
//
// WATCH OUT!: Getters take and return references.
//
// There's no function for getting with a default, but `[cloned()].unwrap_or()` works well.
// There's also no unchecked mutable (see https://stackoverflow.com/a/30414450).
//
map["b"];                       // Immutable, unchecked
map.get("a");                   // Immutable, checked (Option)
map.get_mut("a");               // Mutable, checked

// Entry API: `entry()` gets the value for in-place modification.
//
// `or_insert()` sets the given value if the key doesn't exist; its return value can be used to modify
// the value in-place; consumes the entry.
//
// If the value to insert is a function (or relatively expensive), use `or_insert_with(Fn)`, which executes
// only if the value is going to be inserted.
//
map.entry(key).and_modify(|v| *v += 1).or_insert(50);      // modify if existing; insert otherwise
let entry_ref = map.entry(key).or_insert(50); *entry += 1; // ^^ alternative
map.entry(key).or_insert_with(Vec::new);                   // insert (expensive) if not existing

// Modify a map in a loop.
//
for triangle in triangles {
  let group = groups.entry(current_group);
  group.and_modify(|group| Group::add_child(group, &triangle));
}

// Invalid! Once a key is inserted, it's owned by the hash map!
//
let key = String::from("abc");
map.insert(key, 20);
println!("{}", key)

// Iterate a map.
// In order to iterate and modify, use `keys().cloned().collect::<Vec<_>>()`
//
for (book, review) in &book_reviews { /* ... */ };

// Other APIs
//
map.keys();    // Iterator
map.values();  // Iterator
map.contains_key(key);

// Keys may need a bit of treatment in order to be collected. For example, if the map has String keys,
// the &String references will need to be converted to slices:
//
map
  .keys()
  .map(|k| k.as_str())
  .collect::<Vec<_>>()
  .as_slice();
```

Conveniences:

```rust
// Convert a map to owned vector
//
let vec = score_table
  .into_iter()
  .map(|(_k, v)| v)
  .collect::<Vec<_>>();

// Create a map from two arrays.
//
let scores = teams
  .iter()
  .zip(initial_scores.iter())
  .collect::<HashMap<_, _>>();

// Create a map from a vector of tuples (a, b)
//
let haxx: HashMap<_, _> = vec![("foo", 1), ("barry", 2)]
    .into_iter()
    .collect();
```

In order to use enums as keys, annotate them with `#[derive(Eq, PartialEq, Hash)]`.

Map literals are not supported. See the `maplit` crate.

#### HashSet

HashSet is based on HashMap, but it doesn't take space for the values, due to compiler's optimization.

```rs
// Convenient constructor
//
let set = HashSet::from(["Foo", "Bar"])

set.insert(value)    // returns true if the value was in the set; false if not
```

#### Manual hashing

```rs
let mut hash = DefaultHasher::new();
instance.hash(&mut hash);
// This doesn't reset the hasher; in order to replicate the same hash, must reinstantiate it.
let hash_value = hash.finish();
```

### Other data structures

`BinaryHeap`: max heap.

```rs
// Convenient construct:
//
let heap = iter.collect::<BinaryHeap>();

// In order to represent a min heap, use Reverse, or implement a custom Ord:
//
heap.push(std::cmp::Reverse(1));
```

## Arithmetic/math APIs

```rust
x / y                           // nearest int (-3 / 2 == -1) -> different from Ruby
val += 1; val -= 1;             // increment/decrement value (no postfix)
val <<= n; val >>= n;           // unsigned is zero-extending, signed sign-extending; overflows are ignored

10_f64.clamp(-100., 100.)       // clamp a number (limit value within interval); floats only
10_u64.pow(2);                  // exponentiation (power), int/int
10_f64.powi(2);                 // exponentiation, float/int
10_f64.sqrt();                  // square root
(-1_f64).rem_euclid(20);        // 19 (nonnegative remainder); standard modulo: -1 % 20 == -1; see https://doc.rust-lang.org/std/primitive.i32.html#method.rem_euclid
10_f64.sin();                   // sine (in rad)
10_f64.cos();                   // cosine (in rad)
10_f64.signum();                // sign. float: (>= +0.0 -> 1.0), (<= -0.0 -> -1.0), (NaN -> NaN); int: (0 -> 0), (< 0 -> -1), (> 0 -> 1)
10_f64.hypot(10_f64);           // hypotenuse (!!)

std::f64::consts::{PI,E,SQRT_2,FRAC1_PI,FRAC_1_SQRT_2,LN_2,LOG10_2,...};   // some constants
std::f64::INFINITY, NEG_INFINITY;

u32::min(1, 2);                 // minimum between two numbers
std::cmp::max(x, u);            // maximum between two numbers

z = x.rotate_left(y);
z = x << y;                      // Shift; errors only if y is higher than the number of machine bits.

// Checked, wrapping, saturating and overflowing arithmetic.
//
// With standard operators, on overlow, Rust panics in debug, and wraps in release.
//
x.checked_add(y);                // returns Option<T>
x.wrapping_add(y);
x.saturating_sub(y);             // saturating operation (result is within to the data type bounds, e.g. unsigned 1 - 2 = 0)
z, carry = x.overflowing_add(y); // adds and wraps around in case of overflow; <carry> is bool.
x.overflowing_shl(y);            // WATCH OUT! This is not the intuitive 1-bit left shift - Rust doesn't have it predefined; for that,
                                 // manually compute the carry, then execute `wrapping_shl()`. Alternatively, override the shift operator.
                                 // WATCH OUT! For other `overflow_` operations, `carry` may not the intuitive value, eg. for bit shift

// Handling mixed-sign operations
//
5_u64.wrapping_add(-4_i32 as u64);   // proper way to add signed to unsigned
5_u64.checked_sub(-(-4_i32) as u64); // otherwise, branch positive/negative, and do checked op for both cases

// Round to specific number of decimals (ugly!!; also see #printing)
//
(f * 100.0).round() / 100.0;
```

## Strings

There are two types of strings:

- literals (`&'static str`); hardcoded in the executable.
- `std::string::String`s; allocated on the heap: `String::from("text")`.

Then, there are the slices (don't forget the `&` operator!!!).

```rust
// The `String` data type is mutable.
//
let string: String = String::from("pizza!");
let string: String = "pizza!".to_string();
let mut string = String::new();

let slice: &str = &string[0..3];
let literal: &str = "Your pizza";             // string literals are slices!

// multi-line string, with leading spaces removed.
// for the Ruby squiggly heredoc, use the `indoc` crate.
//
let string = "\
    CatacombSDL\n\
\
    Usage: catacomb [windowed <width> <height>] [screen <num>]\n\
" // this is not included as newline, since there is a backslash before
```

Using strings with functions:

```rust
// Valid, but forces to pass a string
//
fn str_method(s: &String) -> &str { s }

// Better: accepts slices
//
fn str_method(s: &str) -> &str { s }

// All valid!
//
str_method(&string[..]);
str_method(&literal[..]);
str_method(literal);
```

### String/Slice/Char-related APIs/conversions

String (`s`)/Slice (`sl`) APIs:

```rust
s.eq(&str)                              // test equality (compare)
sl.len()
sl.is_empty()                           // true if 0 chars long
s.contains("pattern");
s.start_with("pref");
sl.is_char_boundary(i)
s1 == s2;                               // String comparison (all comparisons supported); char-based (not graph.cl.!)

s += &s2;                               // concatenate via overloaded operator (`+`/`+=`); can take &str or &String
s.push_str(&str);                       // concatenate (append) strings
s.strip_suffix(&str) -> Option<&str>    // strip a suffix; returns the stripped string; None if the suffix was not found
s.push('c');
s.pop();                                // remove last character and returns it
s.insert(pos, 'c');                     // insert; use also as Ruby unshift()
s.insert_str(pos, slice)                // ^^ same, for slice
s.to_lowercase(); s.to_uppercase();
s.clear();                              // blank a string
s.truncate(n)                           // truncate after n chars
s.drain(range)                          // remove and return iterator over removed

writeln!(str, "text")?                  // concatenate via `fmt::Write` trait
format!("{}/{}/{}"), s1, s2, s3);       // preferred format for more complex concatenations
s.repeat(8);                            // string repeat (multiplication)

s.trim(); s.trim_end(); s.trim_start(); // trim/strip
s.trim_end_matches("suffix");           // chomp suffix (but repeated)! also accepts a closure

s.as_bytes();                           // byte slice (&[u8]) of the string contents
s.into_bytes();                         // convert to Vec[u8]
sl.bytes()

// Splits; there is a `rsplit*` version for each (which iterates from the right).
// In order to convert to string and/or use slice, do `collect()`.
//
sl.split("sep")
sl.split_mut("sep")                     // returns mutable slices
sl.split(char::is_numeric);
sl.split(|c: char| c.is_numeric());
sl.splitn(max_splits, "sep");           // performs maximum `max_splits`
sl.split_at(i)
sl.split_terminator(pattern)            // doesn't produce an empty string if separator is last char; has `r` variation
sl.split_whitespace(); sl.split_ascii_whitespace()

s.lines();                              // the newline char is not included in the output!

s.find(pattern)                         // find position; accepts char, slice, String, FnMut
s.contains(pattern)
s.replace(pattern, repl)                // Ruby :gsub; with `...n` variation
s.replace_range(range, repl);
```

Slicing is tricky, as there are different to interpret a string:

- byte-based (Rust range addressing default)
- char unit-based (`char[]`)
- grapheme-cluster-based (requires create)

```rs
// Different approaches (and discussion) [here](https://users.rust-lang.org/t/how-to-get-a-substring-of-a-string/1351)

// Slice per char unit (extract a substring at given index)
//
"ab¢de".chars().skip(2).take(2).collect::<String>()

// Split a string at a given character, then convert a substring of the second to a number:
//
let mut split: Split<char> = "/foo/bar{1}".rsplitn(2, '{');

let mut depth = split.next().unwrap().to_string();
depth.pop();
depth = depth
    .parse()
    .expect("Expected number between braces in path '{}'");

let path = split.next().unwrap().to_string();
```

Char APIs:

```rust
// WATCH OUT! These method may also include unintuitive chars, like `⑧` and `ᛮ`.
// Use the `is_ascii_*()` methods in order to restrict to ascii chars.
// Note that there are other methods.
//
c.is_alphabetic()
c.is_numeric()
c.is_whitespace()
c.is_control()                            // control char
c.is_digit()                              // ASCII-only
c.is_lowercase(); c.is_uppercase()        // also non-latin

c.to_ascii_lowercase(); c.to_ascii_uppercase(); // Also implemented for u8; ignores non-ascii chars
c.to_lowercase(); c.to_uppercase();             // UTF8; returns an iterator (AAARGH!!!)
```

See [Casting/Conversions](#castingconversions) for cast-related APIs.

### Chars iterator

`String#char()` is a special iterator:

```rs
let mut str_chars = "u16".chars();

// Result: Some('u'), "16"
//
println!("{:?}, {:?}", str_chars.next(), str_chars.collect::<String>());
```

In order to get the length, must `collect()`.

There is also `char_indices()`, whose type `CharIndices` iterates over (byte position, char).

### Internal representation (bytes/chars/graphemes)

A `String` is a wrapper over a `Vec<u8>`.

Rust has three notions of string composition. Example `नस्` (the second character is accented):

- bytes: `[224, 164, 168,   224, 164, 184,   224, 165, 141]`
- Unicode scalars (codepoints): `['न', 'स', ' ्']`
- grapheme clusters: `"न", "स्"`

Rust doesn't handle graphemes natively (requires a crate).

Accessing:

```rust
// Not valid: direct array indexing
//
"ननन"[0]

// Range access, but panics if the sequence returned is not at char boundary.
&"ननन"[0..3]

// Extract chars. WATCH OUT! Does not split into graphemes, e.g. "ü" will be 2 chars (!).
string.chars();

// Extract bytes.
string.bytes();
```

## Control flow

For loops:

```rust
// For the reverse iteration, see rev() in the Ranges section, with warning(s).
//
for x in 0..10 { }

// Custom increment/decrement. See previous comment; goes from 98 to 0. It's not possible to write
// `.rev().step_by(-2)`, since `step_by()` takes a usize.
//
for x in (0..100).step_by(2).rev() {}

// Iterate an array/vector
//
for i in collection.iter() { }        // see #ranges-and-stditeriterator-methods for the other methods
for i in &collection { }
for i in &mut collection { *i *= 2 }
```

While:

```rust
// Using labels (lifetime) to break at outer positions:
'external:
while x > 0 {
    while y > 1 {
        break 'external;
    }
};
```

Loops, to use instead of `while true` (because it's subject to flow-sensitive analysis) :

```rust
let val = loop {
    if true {
        break "foo";    // loops can return a value! (labels go between break and the expr)
    }
};
```

If (let)/then/else:

```rust
if x > 5 {
  println!("{}!", x);
} else if x == 4 {
  println!("{}~", x);
} else {
  println!("{}", x);
}

// Fancy if->Some(v) else->None
//
(a == b).then(|| value)
(a == b).then_some(value)

// If with (multiple) assignment.
//
let (a, b) =
  if true {
    (1, 2)
  } else {
    (3, 4)
  };

// If let (can match the same way pattern matching does; see specific section).
//
if let Coin::Quarter(state) = coin {
  println!("State quarter from {:?}!", state);
} else {
  count += 1;
}

// In order to compose a complex expression with pattern matching, use `matches!`
//
if matches!(coin, Coin::Quarter(x) if x > 2) && random_event() {
  // Can use a guard (above), but can't use the value here (makes sense)!
}
```

While let: same. In this case, with stacked Option<T>:

```rs
while let Some(Some(value)) = optional_values_vec.pop() {
  println!("current value: {}", value);
}
```

## Enums

Definition:

```rust
// Each entry is called "variant".
//
#[derive(Debug, PartialEq, Eq)]
enum IpAddrKind {
  V4,
  V6,
}

// Enum with associated data; it can be of any kind, e.g. anonymous structs.
//
enum IpAddr {
  V4(u8, u8, u8, u8),
  V6(String),
}

// For the memory layout, see [here](rust_performance_and_low_level.md#memory-layout)

// Methods can even be defined on enums!
//
impl IpAddr {
  fn is_local(&self) -> bool { true }
}
```

Assignment and invocation:

```rust
let four = IpAddrKind::V4;
let home = IpAddr::V4(127, 0, 0, 1);

fn route(ip_kind: IpAddrKind) { }
```

Enums can't be iterated. See `strum` crate for this purpose.

### Option

Foundation of Rust. In order to use the contained value, we must extract (and test) it.

WATCH OUT! unwrap() moves the value out, so trying to do `&mut opt.unwrap()` doesn't work, because it's a reference to the result, rather than the content.  
In order to solve this case, use `if let` or `as_ref()`/`as_mut()`.

```rust
enum Option<T> {
  Some(T),
  None,
}

// For None, the type must be specified, otherwise it can't be inferred.
//
let some_number = Some(5);
let absent_number: Option<i32> = None;

// Extract a value, raising an error if None. Since `expect()` consumes the value, use `as_ref()`/
// `as_mut()` in order to borrow.
// If the message needs formatting, use `format!`, optionally via  `unwrap_or_else()`.
//
let value = opt.expect("it shouldn't be None!");
let &mut value = opt.as_mut().expect("it shouldn't be None!");

// take(): extract a value and replace with None (no errors raised if invoked on None).
// This is useful when we want to move out an instance that doesn't implement Copy.
//
let mut x = Some(2);
let y = x.take();
x == None;
y == Some(2);

// replace(): extract a value and replace it with the value passed.
//
let mut x = Some(2);    // works also on None
let old = x.replace(5);

// convert Option<&T> to the owned version (Option<T>)
opt.cloned()
opt.copied()
```

#### Convenient Option patterns

```rs
// map(): map the content, only if it's Some.
//
opt.map(|n| 2 * n);               // Option -> Option
opt.and_then(|n| func(n));        // Option -> func(v) (doesn't wrap)
opt.and(val);                     // Option -> v       (doesn't wrap)

// Boolean patterns
//
(a == b).then(|| value)           // bool -> Option
(a == b).then_some(value)         // bool -> Option; unstable
opt.is_some_and(|n| func(n)) {  } // Option -> bool; unstable, equivalent to `matches!(val, Some(x) if f(x))`;
                                  // can also split in `let res = ...` -> `if Some(val) = res {}`

// There are more compact ways, but they're not as clear
opt.is_none() || opt.unwrap() == value

// Convenient pattern. Companion APIs:
//
// - `unwrap_or`:         eagerly evaluated
// - `unwrap_or_default`: invokes the `default()` (!!)
//
opt.unwrap_or_else(|err| {
  println!("Problem parsing arguments: {}", err);
  std::process::exit(1);
});

// Return the value, or set (and return) it if not set
opt.get_or_insert_with(|| value)
```

### Result, convient APIs, and interop with Option

WATCH OUT!!! The Result error enum is `Err`, not `Error` !!!!!!

Many of the methods are common between `Option` and `Result`.

```rs
// Return the (Option<>) source that caused the error, when there has been a chain.
//
err.source();

// Convert Result to Option; if Err, the error is discarded
//
result.ok();

// Check if a result is Ok and discard the content.
if result.is_ok() { /* ... */ }

// Try multiple result values; for functions, use or_else().
//
res.or(res2);

// Control flow based on return value
//
res.or_else(|e| f(e)).or_else(|e| f2(e));

// Convert Option to Result.
//
option.ok_or("error!");
option.ok_or_else(|| slow_function() );

// Mappings borrowed <> owned.
// Result also supports as_ref()/as_mut().
//
instance.as_ref()
instance.as_mut()
```

### Convert to/from numeric

Enums can be casted to numerics, but WATCH OUT! they will truncate silently (clippy will report this, through).

Numeric to enum requires matching the variants; some crates do this, but a `TryFrom`-implementing macro will also do:

```rs
// Variation of a [StackOverflow macro](https://stackoverflow.com/a/57578431), which accepts a type:

macro_rules! impl_try_from_numeric {
    (
        $from_type:ty,
        $(#[$meta:meta])*
        $vis:vis enum $name:ident {
        $(
            $(#[$vmeta:meta])*
            $vname:ident $(= $val:expr)?,
        )*
        }
    ) => {
        $(#[$meta])*
        $vis enum $name {
        $(
            $(#[$vmeta])*
            $vname $(= $val)?,
        )*
        }

        impl std::convert::TryFrom<$from_type> for $name {
            type Error = ();

            fn try_from(v: $from_type) -> Result<Self, Self::Error> {
                match v {
                    $(x if x == $name::$vname as $from_type => Ok($name::$vname),)*
                    _ => Err(()),
                }
            }
        }
    }
}

impl_try_from_numeric! { u16,
    #[repr(u16)]
    #[derive(Clone, Copy)]
    pub enum ClassType {
        Lastclass = 23,
        Guns = 22,
        Gune = 21,
    }
}
```

It's possible to make the macro accept multiple types (which is a bit tricky; see blog article), but on can simply cast on-site to the desired type.

## Error handling

### Basic patterns

```rs
// Question mark ('?') operator: convenient syntax for returning None/Err from the function, if it's the value of an Option/Result.
// If the Error type is different and From<T> is implemented, it's invoked; it's therefore possible for the function to return a
// Box<dyn Error> or the specific Error type.
//
fn mine() -> Result<String, io::Error> {
  let value = errorable_operation()?;
  result = process(value);
  Ok(result)
}

// Pattern for ignoring an error, useful in some cases
//
let _ = writeln!(stderr(), "error: {}", err);

// Match only a certain error! Possible because `dyn Error` implements `downcast_ref()`.
//
match result_box_dyn_error {
    Ok(()) => return Ok(()),
    Err(err) => {
      if let Some(pie) = err.downcast_ref::<ParseIntError>() { /* ... */ }
      // ...
    }
}
```

Unspecific error handling (see downside in the next chapter):

```rs
// Catch-all version of Error; aliased for convenience.
//
type GenericError = Box<dyn std::error::Error + Send + Sync + 'static>;

// Manual conversion from another error type. Possible because the above `Box<dyn Error...>` implements `From<Error...>`.
//
let converted_err: Box<dyn Error...> = GenericError::from(another_error);
```

### Complex error handling

If a function returns `Box<dyn Error>`, it's not possible to infer the error type (ignoring `downcast_ref()`).

The typical solution is to create an enum, and implement the required traits, plus a convenient Result typedef:

```rs
#[derive(Debug)]
pub enum MyError {
    FileOpenError(&'str),
    WrongFileError(&'str),
}

// The `description()` method is optional.
//
impl Error for MyError {}

impl Display for MyError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{}", match_self_something)
    }
}

impl From<io::Error> for MyError {
    fn from(_: io::Error) -> Self {
        todo!()
    }
}

// Convenient, to avoid boilerplate.
//
pub type MyResult<T> = Result<T, MyError>;

pub fn myfailfn() -> MyResult<File> {
    // Because of the `From<io::Error>` implementation, the error is automatically converted.
    let result = File::open("abc")?;

    if !correct_file(result) {
      return WrongFileError(123);
    }

    Ok(result)
}
```

## Pattern matching

`let` can be used to destructure (e.g. a struct) even without being an `if let`!

(for testing a value directly, one can use `value.is_none()`/`value.is_some()`).

```rust
// Generic match. Can also match char intervals!!
//
let xxx = match val {
  1 | 2 => "1 or 2",
  3..=5 => "between 3 and 5", // chars also allowed; exclusive ranges are (as of 8/2021) not supported!!
  n     => format("{}!", n),  // catch-all; use the unnamed var (`_`) to discard the value
};

// Convenient macro.
// When implementing on PartialCmp, import `std::cmp::Ordering::{Equal, Greater, Less}`.
//
matches!(self.partial_cmp(other), Some(Less) | Some(Equal))

// Match enum; all the entries must be exhausted (or the `_` placeholder must be used).
// The second case is a variant with an associated value!
//
// See `if let` for an alternative structure.
//
match message {
  Message::Move{x, y} => { self.position = Point { x: x, y: y } }
  Message::Echo(s) => { self.echo(s) }
  Message::ChangeColor(c1, c2, c3) => { self.color = (c1, c2, c3) } // enum of tuple struct
  Message::Publish(Message(message)) => {} // nested!
  Message::None(_) => {}                   // ignore content
}

// Match tuples
//
match (2, 4, 8, 16, 32) {
  (first, .., last) => { // ignore values
    println!("Some numbers: {}, {}", first, last);
  }
};

// Match (unpack) slices/arrays

if let [r, g, b] = &raw_pixels[..] { /* ... */ } else { panic!() }

// Match Option<T>
//
let x = Some(10);
match x {
  None              => None,
  Some(i)           => Some(i + 1),
  Some(x) if x == y => Some(10),    // "Match guard"
  Some(y)           => Some(y * 2), // Shadowing! Not what it looks!
};

// Match Result<T, Err>
//
match Url::parse(&line) {
  Ok(url) => println!("ok"),
  _ => println!("ko"),
};

// Match a struct (!!)
//
let point = Point { x: 1, y: 1 };
match point {
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y }    => println!("On neither axis: ({}, {})", x, y),
    Point { x, .. }   => {} // ignore rest of the struct

    // "Binding": assign a value while testing, via `@`. This can avoid a copy!
    //
    Point { x: x_val @ 3..=7 } => println!("Found an x in range: {}", id_variable),
};

// Hardcode destucturing.
//
if let KeyboardInputEvent {
    input:
        KeyboardInput {
            virtual_keycode: Some(key_code),
            ..
        },
    ..
} = event

// Borrowing matcher. Also mut!
//
match account {
  Account { ref name, ref mut surname, .. } => {
    println!("{}", name);
    *surname = "abc"
  };
}

// Match a reference (different from the previous).
// WATCH OUT! If not borrowing a reference, the value is copied (below, `x`).
//
match center {
  &Point { x, ref y } => /* ... */
}

// Match an enum inside a struct (!!!):
//
if let Event::Window { win_event: WindowEvent::SizeChanged(new_width, new_height), .. } = event { /* ... */ }
```

## Unsafe

Raw pointers:

```rust
fn print_raw_pointers(ref_mut: *mut i32, ref_const: *const i32) {
  unsafe {
    println!("r1:{}, r2:{}", *ref_const, *ref_mut);
  }
}

let mut num = 5;

// Creating pointers is not unsafe - only dereferencing them.
//
let ref_mut = &mut num as *mut i32;
let ref_const = &num as *const i32;

print_raw_pointers(ref_mut, ref_const);

// Invalid: (currently) this can't be done, in order to work around the BC - the variable needs to be
// declared (separately).
//
mymethod(&mut num, &num as *const i32);

// (Some) APIs, specific to raw pointers (unsafe):

contents                                      // *mut u8
    .offset(1)                                // pointer arithmetic; unit: size_of::<T>
    .copy_from(source.as_ptr(), source.len()) // memcpy equiv.

std::intrinsics::write_bytes(contents, byte, count) // memset equiv.
libc::memset(contents, value, count);               // strict memset

std::ptr::null_mut();                         // null pointer, to use in cases where it's discarded
```

Functions:

```rust
// Unsafe function; unsafe calls themselves don't require the enclosing function to be marked as such.
//
unsafe fn dangerous() {}

// Unsafe funcion call
//
unsafe {
  dangerous();
}
```

Example: slicing an array. It can't be done safely, because we can't mutably borrow twice.

```rust
let slice = &mut [0, 1, 2, 3][..];
let mid = 2;

let len = slice.len();
let ptr: *mut u32 = slice.as_mut_ptr(); // convert a slice to a raw pointer

let (slice1, slice2) = unsafe {
  (
    std::slice::from_raw_parts_mut(ptr, mid),
    std::slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid),
  )
};
```

Referencing raw memory (undefined behavior):

```rust
let address = 0x012345usize;
let r = address as *mut i32;

let slice: &[i32] = unsafe {
  std::slice::from_raw_parts_mut(r, 10000)
};
```

### Interoperability with other languages (C)

Invoking C code:

```rust
// `"C"` defines the ABI, in this case, the C one.
//
extern "C" {
  fn abs(input: i32) -> i32;
}

unsafe {
  println!("Absolute value of -3 according to C: {}", abs(-3));
}
```

Allow code to be invoked from C:

```rust
// Requires disabling mangling.
//
#[no_mangle]
pub extern "C" fn call_from_c() { }
```

## Structs

```rust
// Define a struct
struct User {
  username: String,
  active: bool,     // the last field can be ?Sized
}

// Instantiate it
let mut user = User {
  username: String::from("sav"),
  active: false,
};

// Update it. Requires the whole struct to be mutable!
user.active = true

// Destructuring a struct (!!)
let User {username: mut a, active: b, ..} = user;
println!("a:{}, b:{}", a, b); // "a:sav, b:false"
a = 123;

// Hardcore destructuring
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

Printing requires `Debug` trait!

Empty ("unit-like") struts can also be created, although they have a particular role.

Tuple structs:

```rust
struct Color(i32, i32, i32);
let black = Color(0, 0, 0); // the constructor is actually a function (!)
```

tuple structs with same definition cannot share instances.

Tuple struct with a single field can be used as "newtype" pattern - a wrapper to get stricter type checking, e.g. (`Ascii(Vec<u8>)` for ascii strings, rather than a simple `Vec<u8>`).

Odd functionality:

```rust
// "Field init shorthand" syntax for reducing struct definitions in functions.
fn create_user(email: String) -> User {
  User {
    email,
    active: true,
  }
}

// "Struct update" syntax to copy part of another instance.
// The field assigned before `..` are not taken from the instance after.
let mut user2 = User {
  active: false,
  ..user
}
```

### Associated functions/methods

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

// Multiple methods can be defined in the same `impl` block (and multiple blocks can be defined).
impl Rectangle {
    // When there is `self`, it's called "method".
    // `self` can also be borrowed mutably.
    // Methods taking ownership of `self` are rare and specific.
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    // Where there isn't `self`, it's called "associated function" (static method, in other langs).
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

### `self` in methods

In methods, the `self` reference can be also a smart pointer:

```rust
struct MyStruct {
  fn my_method(self: Arc<Self>, param: u32) { /* ... */ }
}
arc_ref = Arc::new(MyStruct {});
arc_ref.my_method(123);
```

This is currently limited to `Box`/`Rc`/`Arc`.

### Operator overloading

WATCH OUT! Think carefully about ownership - whether owned values (self/rhs) or references should be used.

```rust
struct Point(i32);
struct BigPoint(i32);

impl Add for Point {
  type Output = Self;

  // `rhs` stays for `Right Hand Side`
  //
  fn add(self, rhs: Self) -> Self::Output {
    Point(self.0 + rhs.0)
  }
}

impl Add<&BigPoint> for Point {
  type Output = Self;

  fn add(self, rhs: &BigPoint) -> Self::Output {
    Point(self.0 + 1000 * rhs.0)
  }
}

Point(10) + Point(20)
Point(10) + &BigPoint(1)
```

Some operators:

- `std::ops::Add/Sub/Mul/Div/Rem`: `self.add`, etc -> `Output` (`Rem`=%)
- `PartialEq`: `&self.eq`
- `std::ops::Neg`: `self.neg` -> `Output` (unary negation)

In order to overload on a trait, use the following syntax:

```rust
impl PartialEq for dyn Shape + '_ {
    fn eq(&self, rhs: &Self) -> bool {
        self.id() == rhs.id()
    }
}
```

### Method overloading (workaround)

Silly example. For basic data types, `Into<T>` has blanket implementations:

```rust
struct Foo {
  value: Monster
}

impl Foo {
  fn set<T:Into<Monster>>(&mut self, value: T) {
    self.value += value.as_uint();
  }
}

impl Into<Monster> for Foo {
  fn into(self) -> Monster {
    self.monster
  }
}

impl Into<Monster> for Bar {
  fn into(self) -> Monster {
    self.monster
  }
}
```

### Unions

```rust
// Watch out, x86 is little endian!
//
#[derive(Copy, Clone)]
struct Register8Pair {
    l: u8,
    h: u8,
}

// `#[repr(C)]` is necessary, to ensure that all the fields start at the same location.
//
#[repr(C)]
union Register16 {
    r16: u16,
    r8: Register8Pair,
}

let mut ax = Register16 { r16: 0xCAFE };

// Access is unsafe, because it's effectively undefined behavior.
unsafe { println!("AX:{:4X} AH:{:2X} AL:{:2X}", ax.r16, ax.r8.h, ax.r8.l) };

// Unsafe is not required for writes to `Copy` union fields.
ax.r8.l = 0xFF;

// Borrowing rules still apply: the first invocation is valid, but not the second!
//
fn test(_r1: &mut u8, _r2: &mut u8) {}
fn test2(_r1: &mut u16, _r2: &mut u8) {}
unsafe { test(&mut ax.r8.h, &mut ax.r8.l) };
unsafe { test2(&mut ax.r16, &mut ax.r8.l) };
```

Pattern matching works as usual, but exactly one field must be specified.

See [reference](https://doc.rust-lang.org/reference/items/unions.html#pattern-matching-on-unions) for the so-called "tagged" unions.

A proposal for ["unnamed" fields](https://rust-lang.github.io/rfcs/2102-unnamed-fields.html) has been approved, but not yet implemented.

## Generics

"Monomorphization": turn generic types into concrete ones at compile time.

```rust
fn larger<T: PartialOrd>(a: T, b: T) -> bool {
  a > b
}

enum Option<T> {
  Some(T),
  None
}

struct Point<T, U> {
  x: T,
  y: U,
}

impl<T, U> Point<T, U> {
  fn x(&self) -> &T {
    &self.x
  }
}
```

### Const Generics

```rs
pub fn sound<const T: usize>(indexes: [u8; T])

// As of Aug/2022, `const N` can't be moved into a `where` clause
impl<T: Deserialize + Debug, const N: usize> Deserialize for [T; N] { /* ... */ }
```

With unstable, it's even possible to use enums:

```rs
#![feature(adt_const_params)]

#[derive(PartialEq, Eq)]
pub enum GameStep { AwaitingInput, MovePlayer }

pub fn next_step<const T: GameStep>() { insert_resource(T); }

// `{ }` disambiguate type/const generics.
//
app.with_system(next_step::next_step::<{ AwaitingInput }>)
```

## Traits

### Basics (and Generics #2)

```rust
pub trait Summary {
    // Trait constants are not constants in a strict sense - they're defaults. They must be accessed
    // from the implementing type (in this case, Article::MAX_LENGTH or <Article as Summary>::MAX_LENGTH).
    // WATCH OUT! This implies that they can't used with trait objects, since the type must be known!
    //
    const MAX_LENGTH: usize = 4096;

    fn summarize(&self) -> String;

    // Default method. Overriding doesn't require special syntax.
    //
    fn default_placeholder(&self) -> String {
        "Filomegna donde estas".to_string()
    }
}

pub struct Article {
  pub text: String,
}

// Traits can be implemented only if the trait or the type are local to the crate!
// This way, one doesn't find "surprises" from other crates.
//
impl Summary for Article {
    fn summarize(&self) -> String {
      // See note above on trait constants.
      //
      if self.text.len() > Self::MAX_LENGTH {
          panic!();
      }

      "Summary of an article!".to_string()
    }
}

// Specify trait as type. Can specify more (e.g. <T: Summary + Display>).
// The alternative syntax is `print_summary(text: impl T) -> impl T`.
//
fn print_summary<T: Summary>(text: T) -> T {
  println!("{}", text.summarize());
  text
}

// Returning different types is not allowed this way!
//
fn get_summary_type<T: Summary>(switch: bool) -> T {
  if switch {
    Article { text: "article".to_string() }
  } else {
    Tweet { text: "tweet".to_string() }
  }
}

// `where`: syntactic sugar.
//
fn fancy_types_function<T, U>(t: T, u: U)
where
    T: Display + Clone,
    U: Clone + Debug,
{}

// Unsafe trait and implementation
//
unsafe trait Foo {}
unsafe impl Foo for i32 {}

fn main() {
  let article = Article {
    text: "news".to_string(),
  };

  print_summary(article);
}
```

Things to keep in mind when writing generic functions:

```rust
// With this specific structure, T needs to implement Copy because:
//
// - Returning the first element is a copy operation
// - iter() also returns copies
//
// An alternative not using references is Clone, which is expensive.
//
fn first_element<T: Copy>(list: &[T]) -> T {
  for &item in list.iter() {}
  list[0]
}
```

Conditional ("blanket") implementations (uses `Point` from [Generics](#generics)):

```rust
// Implement only for `Point`s whose T implements Display and Ord.
//
impl<T: Display + Ord, U> Point<T, U> {
  fn greatest_x(&self, other: &Point<T, U>) {
    let max_x = std::cmp::max(&self.x, &other.x);
    println!("Largest x: {}", self.x);
  }
}
```

### Approaches to collections and static/dynamic dispatching

Let's say we have the following trait + concrete types:

```rust
pub trait Draw { fn draw(&self); }

pub struct Square {}
impl Draw for Square { fn draw(&self) {} }

pub struct Circle {}
impl Draw for Circle { fn draw(&self) {} }
```

If we want a generic collection of instances of the same type, we use generics; this is faster, as it's static dispatching:

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

// Syntax for defining methods with generics.
impl<T: Draw> Screen<T> { pub fn run(&self) {} }

let screen = Screen { components: vec![Square {}, Square {}] };
```

If the instances can be of different types, we use dynamic dispatching:

```rust
pub struct Screen {
  // In the past, this could be defined as `Vec<&Draw>`, which is now deprecated.
  pub components: Vec<Box<dyn Draw>>,
}
impl Screen { pub fn run(&self) {} }

let screen = Screen { components: vec![Square {}, Circle {}] };
```

`dyn <type>`s are called "trait objects". They require a pointer, like `Box<T>` or a reference (`&`); the methods in the thread must be "object safe":

- return type isn't self;
- no generics.

In order to specify a dynamic type when boxing, must explicitly cast:

```rust
Box::new(NullLogger {});                    // type = Box<NullLogger>
Box::new(NullLogger {}) as Box<dyn Logger>; // type = Box<dyn Logger>
```

It's possible to write a function signature that performs both static and dynamic dispatching, depending on the call parameters (of course, it's monomorphized):


```rs
// static:  pass &MyType
// dynamic: pass &dyn MyTrait
//
fn flexible_dispatch<T: MyTrait + ?Sized>(t: &T) { /* ... */}
```

### Supertraits and inheritance (object-orientation)

Supertraits are traits depending on other traits. `X` is a supertrait of `Y` means that `Y` must implement `X` (this "super" as higher in a tree).

```rust
// Example using default implementation.
//
trait BetterDisplay: fmt::Display {
    fn better_to_string(&self) -> String {
        format!("Better!: {}", self.to_string())
    }
}

// - Multiple supertraits;
// - Generic supertrait returning self -> requires Sized
//
trait Matrix: Sized + Mul<Self> { /* ... */ }
```

We can use supertraits in order to implemented object-oriented inheritance; the essence is that the tree is based on traits, rather than types:

```rust
trait Root {
    fn update(&self) {
        println!("M:root");
    }

    fn type_() {
        println!("AF:root");
    }
}

trait Middle: Root {
    fn update(&self) {
        Root::update(self);
        println!("M:middle");
    }
}

trait Leaf: Middle {
    fn update(&self) {
        Middle::update(self);
        println!("M:leaf");
    }
}

struct Node {}

// Note that we need to implement all the traits.
impl Root for Node {
    fn type_() {
        println!("AF:node+root");
    }
}
impl Middle for Node {}
impl Leaf for Node {}

impl Node {
    pub fn update(&self) {
        println!("M:node");
    }

    pub fn type_() {
        println!("AF:node");
    }
}

let node = Node {};

// Note the so-called disambiguations (#1 and #4/5)

Leaf::update(&node);      // M:root/M:middle/M:leaf
node.update();            // M:node                 - if `impl` Node is not provided, this is invalid

Node::type_();            // AF:node
<Node as Root>::type_();  // AF:node+root           - if `impl` Node is provided
<Node as Root>::type_();  // AF:root                - if `impl` Node is not provided
Root::type_();            // Invalid!               - must invoke on an `Impl` type; traits alone are not objects!
```

#### Inheritance: making overridden methods private

The simplest way to make overridden methods private is to use private trait methods:

```rust
pub(crate) mod private {
    pub trait SubInterface {
        fn redefined(&self);
    }
}

pub trait SuperClass: private::SubInterface {
    fn virtual_(&self) {
        println!("virtual_");
        self.redefined()
    }
}

struct SubClass {}

impl private::SubInterface for SubClass {
    fn redefined(&self) {
        println!("redefined");
    }
}

impl SuperClass for SubClass {
    fn virtual_(&self) {
        println!("ovr-virtual_");
        private::SubInterface::redefined(self);
    }
}

let instance = SubClass {};

instance.virtual_(); // virtual_/redefined     - if virtual_() not overridden
instance.virtual_(); // ovr-virtual_/redefined - if virtual_() is overridden
```

An ugly alternative is to pass a "sub interface" implementors a "base class" type:

```rust
impl BaseClass {
    fn virtual_<T: SubInterface>(&self, redefiner: T) {
        redefiner.redefined();
    }
}
```

The other strategy is embedding; in order to avoid forwarding boilerplate, there is the [`delegate` crate](#inheritance-emulation-via-delegate-crate), or the [`Deref\[Mut\]` antipattern](https://github.com/rust-unofficial/patterns/blob/master/anti_patterns/deref.md)).

### Trait limitations/workarounds

See https://stackoverflow.com/q/30938499.

```rust
trait MyTrait {
    // This can return Self, which is ?Sized (not necessarily sized), because the Sized responsibility
    // is down to the implementor.
    //
    fn new() -> Self;

    // In this case, the responsibility in the trait, therefore, Self needs to be Sized.
    // As alternative to the `where` bound, add the `Sized` bound to the trait.
    //
    fn default_new() -> Self
    where
        Self: Sized,
    {
        Self::new()
    }

    // These methods have limitations (see below).
    //
    fn mix(&self, other: &Self) -> Self;                    // #1
    fn mix(&self, other: &dyn MyTrait) -> dyn MyTrait;      // #2

    // This works, however, it can be incompatible with the presence of other trait methods/functions,
    // so generally, the solution is to move it to an adhoc trait.
    //
    fn mix(&self, other: &dyn MyTrait) -> Box<dyn MyTrait>; // #3 (Self not allowed)
}

// This doesn't work with #1, because the compiler can't verify that `right` is the same type as
// `left`.
//
fn mix_trait_objs(left: &dyn MyTrait, right: &dyn MyTrait) {
    // This doesn't work with #2, because the result is not Sized.
    //
    let mixed = left.mix(right);
}
```

### Downcasting/Upcasting

Downcasting from trait object to the concrete class (reference); useful to enable it only in testing:

```rust
use std::any::Any;

trait Trait {
  // In some (non useful) cases, it's possible to get away without implementing this.
  fn as_any(&self) -> &dyn Any;
}

struct Concr {}

impl Trait for Concr {
  fn as_any(&self) -> &dyn Any {
    self
  }
}

fn my_test() {
  let as_trait: Box<dyn Trait> = Box::new(Concr {});

  // Note the implicit Box dereferencing.
  //
  if let Some(concr) = as_trait.as_any().downcast_ref::<Concr>() { /* ... */ }

  if as_trait.as_any().is::<Concr>() { /* .. */ }
}
```

A reference to a trait **can't** be converted directly to a reference of a supertrait (upcasting):

```rust
// See https://stackoverflow.com/q/28632968.
//
let super_ref = &my_better_display as &fmt::Display;

// Basic workaround; for a more elaborate solution, see https://stackoverflow.com/a/28664881.
//
let super_ref = &(my_better_display as fmt::Display);
```

### Iterator trait/Associated types/impl trait

Terminology:

- `iterator`: type implementing `Iterator`
- `iterable`: type implementing `IntoIterator`
- an iterator produces `values`
- the code that receives values is a `consumer`

Iterator-related types/APIs:

```rust
trait Iterator {
    type Item;
    // Returns None when finished, however, it's UB what happens on subsequent calls.
    //
    fn next(&mut self) -> Option<Self::Item>;
    ... // many default methods
}

// IntoIterator trait represent a natural iteration; generally (not always, e.g. for slices) consuming.
// If there are other natural iterations possible, other iteration methods are provided (e.g. for String:
// chars, bytes...).
//
trait IntoIterator where Self::IntoIter: Iterator<Item=Self::Item> {
    type Item;
    type IntoIter: Iterator;
    // depends on being invoked on:
    // - &     -> iterates shared refs
    // - &mut  -> iterates mut refs
    // - owned -> iterates owned values
    //
    fn into_iter(self) -> Self::IntoIter;
}

// The below are conveniences for `&[ mut]iterable.into_iter()`; they're not bound to any trait.
//
collection.iter()            // immutable references
collection.iter_mut()        // mutable references

// Use in reversible iterations
//
trait DoubleEndedIterator: Iterator {
    fn next_back(&mut self) -> Option<Self::Item>;
}

// Use for position-based APIs
//
trait ExactSizeIterator: Iterator {
    fn len(&self) -> usize { ... }
    fn is_empty(&self) -> bool { ... }
}
```

Iterator-related general notes:

- there is not contract for iterables to provide all three implementations (e.g. `iter_mut()` for sorted collections)
- `collect()` relies on `FromIterator`, which converts an iterator to a (generally) collection
- `Iterator` provides a `size_hint()`, which help optimizing building a collection from it

Generic function receiving an iterable (`IntoIterator`):

```rs
fn iter_u32<T: IntoIterator<Item=u32>>(iterable: T) { /* ... */ }
// This signature allows using unsized types
fn iter_generic<T: IntoIterator<Item=U>, U: MyType>(iterable: T) { /* ... */ }
```

Basic `Iterator` implementation:

```rust
impl Iterator for PhonyCounter {
  // Associated type. Similar to generics, however, doesn't require the type to be specified every
  // time the related methods (in this case, `next()`) are invoked.
  // In this case, a default is specified, which is optional.
  //
  type Item = u32;

  fn next(&mut self) -> Option<Self::Item> {
    Some(0)
  }
}
```

`impl <Trait>` is a convenience that:

1. simplifies complex generics
2. avoids the overhead of using Box

```rs
fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> iter::Cycle<iter::Chain<IntoIter<u8>, IntoIter<u8>>> {
    v.into_iter().chain(u.into_iter()).cycle()
}

fn cyclical_zip(v: Vec<u8>, u: Vec<u8>) -> impl Iterator<Item=u8> { /* ... */ }

// WATCH OUT! This doesn't work for trait objects, since it's static dispatching, and the return type
// must be known!
```

If a type can be iterated, it should implement `IntoIterator`:

```rs
trait IntoIterator where Self::IntoIter: Iterator<Item=Self::Item> {
  type Item;
  type IntoIter: Iterator;
  fn into_iter(self) -> Self::IntoIter;
}
```

For loops are actually invoking `into_iter()` on the iterable (`Iterator`s return themselves).

## Ownership

### Move

Variables can be simple (eg. integers) or complex (eg. strings); by default Rust copies the "simple" part (shallow copying).

When variables go out of scope, `drop` is invoked, freeing the memory:

```rust
{
  let val = String::from("text");
  // drop invoked!
}
```

On assignments/functions, simple data types are copied:

```rust
{
  let a = 5;
  let b = a;

  println!("{}", a); // valid
}
```

but complex ones are _moved_; ownership is transferred when assigning or passing to functions:

```rust
{
  let a = String::from("text");

  let b = a;

  // Invalid, unless the `Copy` trait is implemented, which declares that moved variables are still
  // usable. If the `Drop` trait is implemented, `Copy` is not allowed.
  //
  println!("{}", a);

  let c = my_function(b);

  // Invalid as well!
  //
  println!("{}", b);

  // If the function returns the variable, ownership is transferred.
  //
  // Valid!
  //
  println!("{}", c);
}
```

### Borrowing

Passing references doesn't transfer ownership; this is called _borrowing_:

```rust
{
  let a = String::from("text");

  my_function_ref(&a);

  // Valid!
  //
  println!("{}", a);
}
```

Borrowing mutable references has restrictions:

```rust
{
  let mut s = String::from("hello");

  {
    let r1 = &mut s;

    // No more than one mutable reference to the same value in the same scope:
    //
    // Invalid!
    //
    let r2 = &mut s;
  }

  {
    let r1 = &mut s;
  }

  // Different scope.
  //
  // Valid!
  //
  let r2 = &mut s;

  // Multiple immutable references can be in the same scope, but a mutable one excludes any other.

  let mut s2 = String::from("hello");
  let r1 = &s;

  // Valid!
  //
  let r2 = &s;

  // Invalid!
  //
  let r3 = &mut s;
}
```

### Dangling pointers

In Rust, it's not possible to have dangling pointers:

```rust
// Invalid!
//
fn dangling() -> &String {
  &String::from("text");
}

// Must return the string directly.
//
// Valid!
//
fn not_dangling() -> String {
  String::from("text");
}
```

### Lifetimes

Lifetimes describe the scope where a reference lives; the compiler must be sure that the lifetime of a reference is contained within that of the value it references - if the value is dropped/moved during the lifetime of a reference, the reference will be invalid!

For example, here, the returned vector is bound to the matches; if `matches` is freed, the content of the vector may be dangling!

```rust
// Pronounce: "tick a"
//
fn extract_interval_arguments<'a>(matches: &'a clap::ArgMatches) -> Vec<&'a str> {
    matches
      .values_of("INTERVALS")
      .unwrap()
      .collect::<Vec<&str>>()
}
```

The `'static` lifetime refers to a reference that is valid across the whole program lifetime; example of valid one:

```rs
static MYARRAY: [i32, 3] = [0, 1, 2];

fn return_myarray_slice() -> &'static [i32] {
  &MYARRAY[..2]
}

// Other typical refs with static lifetime are hardcoded strings
```

Structs can have lifetimes, but they must be applied to all the members.

```rust
struct ImportantExcerpt<'a> {
  part: &'a str,
}

// Syntax for implemented methods:
//
impl<'a> ImportantExcerpt<'a> {
  fn level(&self) -> i32 { 3 }
}
```

Complex example ([source](https://dev.to/takaakifuruse/rust-lifetimes-a-high-wall-for-rust-newbies-3ap)). It seems that this example is not proper - `parse_context()` should borrow the context, not own it. However, this is interesting for the sake of understanding lifetimes.

```rust
struct Context<'c> {
  text: &'c str,
}

// The context outlives the parser.
struct Parser<'p, 'c> {
  context: &'p Context<'c>,
}

impl<'p, 'c> Parser<'p, 'c> {
  // This makes clear the lifetimes.
  fn new(context: &'p Context<'c>) -> Parser<'p, 'c> {
    Parser { context }
  }

  // The return value has the lifetime of the context.
  fn parse(&self) -> &'c str {
    &self.context.text
  }
}

// In order to make this possible, we need the two lifetimes. The `context` field of the `Parser` has
// the lifetime of the parameter, which is used in the return value of the function `parse()`, so that
// the `&str` is actually `&'c str`.
fn parse_context(context: Context) -> &str {
  Parser::new(&context).parse()
}

// The borrowed version doesn't need any special handling.

struct Context<'a> {
  text: &'a str,
}

struct Parser<'a> {
  context: &'a Context<'a>,
}

// Constructor omitted for simplicity.
impl<'a> Parser<'a> {
  fn parse(&self) -> &'a str {
    &self.context.text
  }
}

fn parse_context<'a>(context: &'a Context) -> &'a str {
  Parser { context }.parse()
}
```

#### Elision rules

In some cases, Rust can infer lifetimes, so they don't need to be specified. There are three rules:

```rs
// The lifetimes defined are added by the compiler.

// 1. Each param reference gets its own lifetime.
//
fn myfn(foo: &'a str, b:&'b str) { /* ... */ }

// 2. If there is only one param ref, the return ref gets the same lifetime
//
fn myfn(foo: &'a str) -> &'a str  { /* ... */ }

// 3. If there is ref among the param refs, the return ref gets the same lifetime
//
fn myfn(&'a self, foo: &'b str) -> &'a str  { /* ... */ }
```

Lifetime elision depends exclusively on the signature, not on the body!!

### Slices

Slices can refer to arrays and strings. They are immutable references, so the ownership needs to be considered:

```rust
{
  let mut s = String::from("margherita");

  let slice = &s[..]; // Immutable borrow
  s.clear();          // Mutable borrow: Invalid!

  println!("{}", slice);
}
```

String slices are at *byte* points!

Using string slices as arguments is preferrable to string references, as they're more generic (they can also take strings).

## Smart pointers

Smart pointers implement the traits:

- `Deref`: makes instances behave like pointers (implementing the deference operator (`*`));
- `Drop`: invoked when instances go out of scope.

In order to manually drop an instance, use `std::mem::drop`.

### Box<T>

Allow storing data on the heap rather than on the stack; useful for:

- a type whose size can’t be known at compile, but it's required, e.g. recursive types;
- data size constitutes a performance problem (if copied);
- owning a value when a trait is required, without forcing it to be of a specific type.

Generic usage:

```rust
{
  let boxed = Box::new(5);

  // Dereference, in order to use the boxed value.
  //
  assert_eq!(5, *boxed);

  // here the pointer and data are deallocated
}

// If the reference to the contained object is required, use:
//
let object_ref = Box::new(object).as_ref();
```

Use in recursive structures:

```rust
// Without Box, the compiler complains that indirection is needed.
//
enum Node {
  Parent(Box<Node>),
  Nil,
}

use self::Node::{Nil, Parent};

let parent_a = Parent(Box::new(Parent(Box::new(Nil))));
```

### RC<T>

"Reference Counting"; use when there are multiple owners, e.g. graphs, that is, when the compiler can't know who will use the reference last.

Can hold only immutable values; can be used only on single thread contexts.

```rust
enum Node {
  Parent(Rc<Node>),
  Nil,
}

use self::Node::{Nil, Parent};
use std::rc::Rc;

// Invalid; immutable-only!
//
let value = Rc::new(1);
*value = 32;

// `Rc::clone()` performs a shallow copy (as only the reference needs to be cloned); `clone()` is not
// appropriate, since it makes a deep copy.
//
let a = Rc::new(Parent(Rc::new(Parent(Rc::new(Nil)))));
{
  let _b = Parent(Rc::clone(&a));
  let _c = Parent(Rc::clone(&a));
  println!("Count: {}", Rc::strong_count(&a)); // 3
}
println!("Count: {}", Rc::strong_count(&a)); // 1
```

### Cell<T>/RefCell<T>/Mutex<T> and interior mutability

`RefCell<T>` allows mutating the contained value, even if the variable itself is immutable, therefore bypassing the compiler; generally speaking, it allows multiple owners while retaining mutability. The rules are enforced at runtime though, so this has an overhead.

`RefCell` is not `Sync`; in multithreaded context, `Mutex`/`RwLock` must be used instead.

`Cell` is like `RefCell`, but with limitations:

- it disallows borrowing immutably (so one can read only with `T: Copy`)
- it allows writing or swapping the value

it has no runtime overhead, so it's preferrable, where it's possible to use it. See [Stack Overflow](https://stackoverflow.com/a/53671414).

```rust
enum Node {
    Parent(Rc<RefCell<i32>>, Rc<Node>),
    Nil,
}

use self::Node::{Nil, Parent};
use std::cell::RefCell;
use std::rc::Rc;

let value = Rc::new(RefCell::new(1));

let a = Rc::new(Parent(Rc::clone(&value), Rc::new(Nil)));
let _b = Parent(Rc::new(RefCell::new(2)), Rc::clone(&a));
let _c = Parent(Rc::new(RefCell::new(3)), Rc::clone(&a));

*value.borrow_mut() += 10;
value2.try_borrow_mut()?.value = 10; // try version

// Dereference and cast to trait object.
//
&*value as &dyn MyTrait;

// Convenient design to access internal Rc<RefCell> values. `Deref(Mut)` can't be implemented, because
// it returns a reference, while we need to return an owned instance.
// The methods can also return RefMut; `impl` just hides it.
// WATCH OUT! importing `borrow::BorrowMut` will mess with cause troubles.
//
// Case where the field belongs to Self:
//
pub fn field_mut(&self) -> impl DerefMut<Target = u32> + '_ { (*self.field).borrow_mut() }
//
// Case where the field belongs to an enclosed type:
//
pub fn val_mut(&self) -> impl DerefMut<Target = u32> + '_ {
    RefMut::map(inner_bm, |inner| &mut inner.foo)
}
```

### Modifying Rc/Arc without inner mutable types (and conversion Box -> RC type)

If one wants to modify an RC type before sharing it, without inner mutable types, there is `get_mut()` (and unsafe/nightly `get_mut_unchecked()`). Example using the unchecked version and `Arc`:

```rust
pub fn new(mut children: Vec<Arc<Self>>) -> Arc<Self> {
  let mut parent = Arc::new(Self {
      parent: Weak::<Self>::new(),
      children: vec![],
  });

  for child in children.iter_mut() {
    let child_parent_ref = &mut unsafe { Arc::get_mut_unchecked(child) }.parent;
    *child_parent_ref = Arc::downgrade(&parent);
}

  let parent_mut = unsafe { Arc::get_mut_unchecked(&mut parent) };
  parent_mut.children = children;

  parent
}
```

Another option is to convert a `Box` to an RC type:

```rust
let mut boxed = Box::new(MyType { field: 64 });
boxed.field = 128;

let pizza = Into::<Arc<MyType>>::into(boxed);
let pizza: Arc<MyType> = boxed.into();
```

### Weak<T> and reference cycles

W.R. are used to share RC (strong) references, without causing cycles; they don't cause them because any cycle involving weak references will be broken once the associated RC count is 0.

Example with a tree data structure, with nodes pointing both to children and parents. The problem is that if we don't use weak references, there will be circular references (therefore leaks) because of parents pointing to children, and viceversa.

It's important to always think who is the owner. A parent ultimately owns the children - if the former is dropped, the children should be dropped too; therefore, the reference to the parent should be weak.

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
  value: i32,
  // RefCell is needed here, because Node will be wrapped in an Rc; without it, the contents can't
  // be mutated.
  //
  parent: RefCell<Weak<Node>>,
  children: RefCell<Vec<Rc<Node>>>,
}

// Can't instantiate `Weak` with a reference; must use `Rc::downgrade()`.
// `Weak::new()` is essentially a placeholder.
//
let leaf = Rc::new(Node {
  value: 3,
  parent: RefCell::new(Weak::new()),
  children: RefCell::new(vec![]),
});

let branch = Rc::new(Node {
  value: 5,
  parent: RefCell::new(Weak::new()),
  children: RefCell::new(vec![Rc::clone(&leaf)]),
});

// Common WATCH OUT! concepts:
//
// - don't forget to pass a reference when up/downgrading;
// - if a reference is borrowed, and must assign it, clone() it.

// When invoking `clone()`, it's important to use the for `(Weak|Rc)::clone(ref)`, otherwise, a deep
// clone is performed.
// For consistency with `Rc::downgrade()`, one can always use the type function `Weak::upgrade(ref)`,
// rather than `ref.upgrade()`.

// The `Rc`-enclosed type methods can be accessed via auto-dereferencing (`leaf.parent`).
//
// In order to change the content of a `RefCell`, use `borrow_mut()`.
//
// WATCH OUT! One may need to borrow and do the desired operation on the same statement, otherwise, BCK
// hilarity may ensue.
//
*leaf.parent.borrow_mut() = Rc::downgrade(&branch);

// In order to access a `Weak<T>` value, one must upgrade and `unwrap()` (if sure that there is a value).
//
println!("leaf parent = {:?}", Weak::upgrade(leaf.parent.borrow()).unwrap());

// When accessing an Option<Weak|Rc>, use the `if let`, or `clone().unwrap()`.
// Otherwise, an error is raised: `cannot move out of dereference of `RefMut<'_, ...>``
//
if let Some(ref mut child_wk) = opt_weak_ref { /* */ };
opt_weak_ref.clone().unwrap();

// Finally, Weak<dyn Trait> must receive a phony type on new() (on structs, one can use Self).
// See https://users.rust-lang.org/t/why-cant-weak-new-be-used-with-a-trait-object.
//
let parent: Mutex<Weak<dyn Shape>> = Mutex::new(Weak::<Plane>::new());
```

Counting functions:

```rust
Rc::strong_count(&branch);
Rc::weak_count(&branch);
```

#### `Rc<RefCell>` or `RefCell<Rc>`

This is not trivial. See:

1. https://stackoverflow.com/questions/57367092/what-is-the-difference-between-rcrefcellt-and-refcellrct
2. https://github.com/rust-lang/book/issues/1543#issuecomment-696995826

First case:

With `RefCell<Rc>`, `RefCell` can't be shared, and `Rc` can't be mutated.  
With `Rc<RefCell>`, `Rc` can be shared, and `RefCell` allows mutation.

Second case:

`children` (nodes) options:

```rust
// Vector is not shared, but it can be modified; the children are shared.
// Understanding: since the parent node is wrapped in an Rc, although the vector is not directly shared,
// it ends up being indirectly so.
//
RefCell<Vec<Rc<Node>>>

// Vector is shared; it can be modified, and the children, too.
// ??? -> When cloning, the list of nodes is shared, which is not what wanted.
//
Rc<RefCell<Vec<Node>>>

// Vector is shared, but can't be itself modified; instead, each child can be individually modified.
// This may not make much sense, since nodes can't be added/removed.
//
Rc<Vec<RefCell<Node>>>
```

A way to look at this is that what is between `Rc` and `RefCell` _can't_ be modified.

#### Real case of modeling a thread-safe tree with trait objects, with children addition

Example, with Tree implementing the Node trait. This is a **slow** implementation, since it will cause locking on the vector.

Read-only trees better do without locking at all (only Arc).

```rust
#[derive(SmartDefault)]
pub struct Tree {
  #[default(Mutex::new(Weak::<Self>::new()))]  // Note the (st00pid) necessary generic
  pub parent: Mutex<Weak<dyn Node>>,
  #[default(Mutex::new(vec![]))]
  pub children: Mutex<Vec<Arc<dyn Node>>>,
}

impl Tree {
  // This can't be an associate method, because we can't create a smart pointer to the parent (which
  // implies ownership), so we must pass a smart pointer with the parent, so that we can clone it.
  //
  pub fn add_child(parent: &Arc<dyn Node>, child: &Arc<dyn Node>) {
    parent.children().lock().unwrap().push(Arc::clone(child));
    let mut child_parent_ref = child.parent().lock().unwrap();
    *child_parent_ref = Arc::downgrade(parent);
  }
}
```

#### Real complex case of iterating a recursive structure with Rc/RefCell

See https://stackoverflow.com/questions/36597987/cyclic-reference-of-refcell-borrows-in-traversal.

```rust
struct LinkedList<T: Copy>(
  Option<(
    Rc<RefCell<Node<T>>>,
    RefCell<Node<T>>>
  )>
);

struct Node<T: Copy> {
  value: T,
  next: Option<Rc<RefCell<Node<T>>>>,
  previous: Option<Weak<RefCell<Node<T>>>>,
}

impl<T: Copy> LinkedList<T> {
  pub fn print_values(&self) -> Vec<T> {
    if let Some((ref front, _)) = self.0 {
      let mut current = Rc::clone(front);

      loop {
        println!("{}", current.borrow().value);

        // The are two crucial aspects.

        // 1. we can't borrow current outside of the `if let`, otherwise, the `current` assignment fails
        // because it's borrowed.
        //
        // let current_brw = current.borrow()
        // if let Some(_) = current_brw.next()

        let next = if let Some(ref next) = current.borrow().next {
          // 2. we need an `if let` construct, because if we assign current inside here, again, it's
          // borrowed. The solution here is to return an owned value, while releasing the borrow.

          Rc::clone(next)
        } else {
          break;
        };

        current = next;
      }
    }
  }
}
```

## Multithreading

Base usage:

```rust
use std::io::{self, Write};
use std::thread;
use std::time::Duration;

let message = "Hello";

// `move` is required to access variable in the context, with move semantics, because, the closure may
// outlive the referenced variable!
//
let handle = thread::spawn(move || {
  // Thread-safe way of printing.
  writeln!(&mut io::stdout().lock(), "{}", message).unwrap();
  thread::sleep(Duration::from_millis(1));
});

handle.join().unwrap();
```

Traits:

- `Send`: types safe to pass by value across threads
- `Sync`: safe for pass by non-mut reference

If there are serious problems with lifetimes (eg. recursively passing references), use concurrent toolkits (see concurrency tools section).

### Channels: Multiple Producers Single Consumer

For SPMC, see crate [bus](#channels-single-producer-multiple-consumers-bus), although it can be emulated with multiple channels.

Channels use optimized strategies, per-usage:

- if only one object is ever sent, the overhead is minimal
- if the Sender is cloned, it's switched to a thread-safe strategy
- regardless, any strategy is a lock-free queue implementation

The important thing is to make sure not to overload the queue (see below).

```rust
use std::sync::mpsc;

// Sender channels can be cloned ("Multiple Producer Single Consumer").
//
// For synchronous channels (bound by a buffer), use mpsc::sync_channel(bufsize). A buffer size of
// 0 blocks until a receiver reads.
//
let (tx1, rx) = mpsc::channel();
let tx2 = mpsc::Sender::clone(&tx1);

thread::spawn(move || {
  let val = "hi";
  // send() doesn't block
  tx1.send(val).unwrap();
  // `val` is now moved; can't be used anymore
});

thread::spawn(move || {
  tx2.send("hello").unwrap();
});

// Non-blocking version: `rx.try_recv()`.
//
println!("{}", rx.recv().unwrap());

for received in rx {
  println!("{}", received);
}
```

### Simple Multiple Producers Multiple Consumes

```rs
pub mod shared_channel {
    use std::sync::mpsc::{channel, Receiver, Sender};
    use std::sync::{Arc, Mutex};

    // Thread-safe wrapper around a `Receiver`.
    #[derive(Clone)]
    pub struct SharedReceiver<T>(Arc<Mutex<Receiver<T>>>);

    impl<T> Iterator for SharedReceiver<T> {
        type Item = T;
        // Get the next item from the wrapped receiver.
        fn next(&mut self) -> Option<T> {
            let guard = self.0.lock().unwrap();
            guard.recv().ok()
        }
    }

    // Create a new channel whose receiver can be shared across threads.
    // This returns a sender and a receiver, just like the stdlib's
    // `channel()`, and sometimes works as a drop-in replacement.
    //
    pub fn shared_channel<T>() -> (Sender<T>, SharedReceiver<T>) {
        let (sender, receiver) = channel();
        (sender, SharedReceiver(Arc::new(Mutex::new(receiver))))
    }
}
```

### Mutex<T>/Arc<T>

WATCH OUT!:

- Mutex is not reentrant, so multiple calls by the same thread (unless in the same function) will deadlock.
- When using as global, set it as `static`, not `const`!

Note that an Arc can be safely modified if there is only one reference; see [the interior mutability section](#refcellt-and-interior-mutability).

```rust
// We need to use an "Atomic Reference Counter" because the counter is shared between threads;
// the compiler interprets the counter usage as multiple move.
//
let counter = Arc::new(Mutex::new(0));

for _ in 0..10 {
  let counter = Arc::clone(&counter);

  thread::spawn(move || {
    let mut num = counter.lock().unwrap();
    *num += 1;
  });
}

// threads must be joined
```

Note that `Arc<Mutex<T>>` can be cloned and moved around, but must be extremely careful, because it's very easy to deadlock.

The ownership of an instance in a mutex can be released, if there are no other threads holding the lock: `mutex_instance.into_inner().unwrap()`.

Convenient design to access a mutex value from the outside the enclosing type (the same strategy of `RefMut` can't be applied):

```rs
// inner: Arc<Mutex<InnerType>>;
pub fn lock<R, F: FnMut(&mut InnerType) -> R>(&self, mut fx: F) -> R {
    let mut lock = (*self.inner).lock().unwrap();
    fx(&mut lock)
}

instance.guard(|inner| /* ... */);
```

### Atomic primitive type wrappers

Primitive types have special atomic structs that are very efficient, e.g. (simplified):

```rs
// Atomic types besides the obvious ones:
//
// - AtomicU128 (only in nightly)
// - AtomicPtr<T>: shared value of the unsafe pointer type *mut T

let current_cycle = Arc::new(AtomicU32::new(0)); // Arc is required

{
    let current_cycle = current_cycle.clone();

    thread::spawn(move || {
      let mut cycle_to_execute = 0;

      while cycle_to_execute < cycles {
        while cycle_to_execute <= current_cycle.load(Ordering::Relaxed) {
          // some work
          cycle_to_execute += 1;
        }
      }
    })
}

for cycle_i in 0..cycles {
  current_cycle.store(cycle_i, Ordering::Relaxed);
}

// Atomics can be globals!!!
//
static NEXT_ID: AtomicU32 = AtomicU32::new(1);

pub(crate) fn new_shape_id() -> u32 {
    NEXT_ID.fetch_add(1, Ordering::SeqCst)
}
```

Some APIs (return the previous value):

```rs
ab.load(ordering);                       // Load
ab.store(value, ordering);               // Store
ab.fetch_add(value, ordering);           // Add to the current value, and return the previous
let old: bool = ab.swap(new, ordering);  // Replace the old value with a new one, and return the old one

// If the existing value equals `current`, set to `new`; the two orderings are:
//
// - success: ordering for the read-modify-write operation if the comparison with `current` succeeds
// - failure: ordering for the load operation if the comparison fail
//
// Returns Ok(old)/Err(old), in case of success/failure
//
// There is a faster version, but weaker guarantees (`compare_exchange_weak()`).
//
let result: Option<bool> = ab.compare_exchange(current, new, success_ord, failure_ord);
```

**WATCH OUT**: Check out the [CPP reference](https://en.cppreference.com/w/cpp/atomic/memory_order) to understand memory orderings. The most conservative ordering, `Ordering::SeqCst` has still performance good enough to be used as default, according to Programming Rust 2nd ed.

### Barrier

Enable multiple threads to start synchronized; `wait()` will block on each thread until the number has been reached.  
A random thread is the elected leader (`is_leader() == true`).  
After the barrier is released, it can be reused.

```rust
  let threads_number = 10;
  let barrier = Arc::new(std::sync::Barrier::new(threads_number));

  for _ in 0..threads_number {
    let barrier = barrier.clone();

    thread::spawn(move || {
      let barrier_wait_result = barrier.wait();
      barrier_wait_result.is_leader();
      my_operation();
    });
  }
```

### Condvar

A [condvar](https://doc.rust-lang.org/beta/std/sync/struct.Condvar.html) allows threads to wait, and then be notified when a condition is met.

Watch out! This can't be used for as a pseudo-channel, because the mutex value could be changed between the notification, and the time the thread resumes. Example:

```rust
let pair = Arc::new((Mutex::new(0), Condvar::new()));

let handle = {
  let pair = pair.clone();

  thread::spawn(move || {
    let (lock, cvar) = &*pair;

    let mut cycles_guard = lock.lock().unwrap();

    loop {
      // ... some work ...

      if *cycles_guard == cycles - 1 {
        break;
      } else {
        cycles_guard = cvar.wait(cycles_guard).unwrap();
      }
    }
  })
};

let (lock, cvar) = &*pair;

for cycle_number in 0..cycles {
  let mut cycles_mutex = lock.lock().unwrap();
  *cycles_mutex = cycle_number;
  cvar.notify_one();
}

handle.join().unwrap();
```

### Busy waiting/spin loops (pause)

Intrinsic to use in busy waiting:

```rust
std::hint::spin_loop();
```

## Project structure

A package (=set of 1+ crates) can contain at most one library crate.

Crate "roots":

- `src/main.rs` -> binary crate (same name as package)
- `src/lib.rs` -> library crate (same name as package)

This won't require any addition; Cargo will build the binary by default, and the library when `--lib` is specified.

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

### Prelude structure

In the main file, do something like:

```rs
mod camera;
pub mod components; // if the module is referenced as part of the prelude

mod prelude {
    pub use bracket_lib::prelude::*;
    pub use legion::*;
    pub const SCREEN_WIDTH: i32 = 80;
    pub const SCREEN_HEIGHT: i32 = 50;
    pub use crate::camera::*;
    pub use crate::components::*;
}
```

then, in the modules (files) use:

```rs
use crate::prelude::*;
```

What is exported depends on the intended design; a very basic design is to export:

- global constants
- commonly used external library APIs
- crate types

so that external library APIs that are used in a single module (file) only, are not in the prelude; if they become used in multiple modules, they can be moved to the prelude.

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

- `pub`, private
- `pub(crate)`
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

## Assorted insanity

- Complex behavior of traits <> lifetimes (now obsolete; using `&Self` as argument is enough): https://stackoverflow.com/questions/54329200/mysterious-lifetime-issue-while-implementing-trait-for-dyn-object.
