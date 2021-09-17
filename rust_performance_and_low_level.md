# Rust Performance

- [Rust Performance](#rust-performance)
  - [User-facing](#user-facing)
    - [std::mem APIs](#stdmem-apis)
    - [Unsafe](#unsafe)
    - [Uninitialized memory (`std::mem::MaybeUninit`)](#uninitialized-memory-stdmemmaybeuninit)
      - [Bidimensional array struct, uninitialized and macro'ed](#bidimensional-array-struct-uninitialized-and-macroed)
    - [std::num::NonZero*](#stdnumnonzero)
    - [(LLVM) Intrinsics/ASM APIs](#llvm-intrinsicsasm-apis)
    - [Embedded/OS development](#embeddedos-development)
    - [Count allocations](#count-allocations)
  - [Benchmarking (`criterion`)](#benchmarking-criterion)
  - [Internal](#internal)
    - [Memory layout](#memory-layout)
    - [References](#references)
    - [Functions/closures](#functionsclosures)
    - [Null pointer optimization](#null-pointer-optimization)
    - [Examine ASM (Assembler) output](#examine-asm-assembler-output)
      - [Prevent function inlining](#prevent-function-inlining)

## User-facing

### std::mem APIs

```rs
size_of::<Type>()         // memory occupation of a type
size_of_val(&v)           // memory occupation of a variable
unsafe { zeroed::<T>() }  // return zeroed memory for type; it's unsafe because zero(s) may not be a valid value
```

### Unsafe

Raw pointers:

```rust
fn print_raw_pointers(ref_mut: *mut i32, ref_const: *const i32) {
  unsafe {
    println!("r1:{}, r2:{}", *ref_const, *ref_mut);
  }
}

let mut num = 5;

// Creating pointers is not unsafe - only dereferencing them.
// They don't own the data, and the multiple raw pointers to the same data are valid.
//
let ref_mut = &mut num as *mut i32;
let ref_const = &num as *const i32;
let ref_const_2 = &num as *const i32;

print_raw_pointers(ref_mut, ref_const);

// Invalid, due to BC; as workaround, pass the raw pointer as separate variable.
//
mymethod(&mut num, &num as *const i32);
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
let ptr = slice.as_mut_ptr();

let (slice1, slice2) = unsafe {
  (
    std::slice::from_raw_parts_mut(ptr, mid),
    std::slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid),
  )
};
```

Referencing raw memory (undefined behavior):

```rust
let address = 0x012345_usize;
let r = address as *mut i32;

let slice: &[i32] = unsafe {
  std::slice::from_raw_parts_mut(r, 10000)
};
```

### Uninitialized memory (`std::mem::MaybeUninit`)

Safe(r) version of initialization with uninitialized values of Array and i32; the simpler but unsafe(r) versions are below.

```rs
let mut result: [MaybeUninit<crate::Color>; pixels_count] = unsafe { MaybeUninit::uninit().assume_init() };
result[0] = MaybeUninit::new(crate::Color { r: 0.0, g: 0.0, b: 0.0 });
let result = unsafe { mem::transmute::<_, [crate::Color; pixels_count]>(result) };

let mut x = MaybeUninit::<&i32>::uninit();
unsafe { x.as_mut_ptr().write(&0); }
let x = unsafe { x.assume_init() };
```

For a more sophisticated application to bidimensional arrays, see the [subsection below](#bidimensional-array-struct-uninitialized-and-macroed).

Other uninitialized forms:

```rs
let mut result: [[f64; $order]; $order] = usafe { mem::zeroed() };
let mut result: [[f64; $order]; $order] = usafe { MaybeUninit::zeroed().assume_init( }) // same as previous

let mut result: [[f64; $order]; $order] = usafe { mem::uninitialized() };
let mut result: [[f64; $order]; $order] = usafe { MaybeUninit::uninit().assume_init( }) // same as previous
```

There are also more sophisticated uses, e.g. partially initialized arrays.

#### Bidimensional array struct, uninitialized and macro'ed

For the lulz; in such cases, avoiding the MaybeUninit intermediate value is simpler.

```rust
macro_rules! matrix {
  ($name:ident, $order: literal) => {
    #[derive(Debug)]
    pub struct $name {
      pub values: [[f64; $order]; $order],
    }

    impl $name {
      pub fn new(source_values: &[f64]) -> Self {
        let mut dest_values: [MaybeUninit<[f64; $order]>; $order] = unsafe { MaybeUninit::uninit().assume_init() };

        for (dest_row, source_row) in dest_values.iter_mut().zip(source_values.chunks($order)) {
          *dest_row = MaybeUninit::new(source_row.try_into().unwrap());
        }

        let dest_values = unsafe { mem::transmute::<_, [[f64; $order]; $order]>(dest_values) };

        $name { values: dest_values }
      }
    }
  };
}
```

Simpler/unsafer logic:

```rust
let mut values: [[f64; $order]; $order] = unsafe { MaybeUninit::uninit().assume_init() };

for (row, source_row) in values.iter_mut().zip(source_values.chunks($order)) {
  row.copy_from_slice(source_row);
}
```

### std::num::NonZero*

If an integer variable is guaranteed never to be 0, one can use `NonZero*` data types, which allow some optimizations.

### (LLVM) Intrinsics/ASM APIs

Intrinsic/ASM APIs are unstable.

```rs
#![feature(core_intrinsics)]  // Enable the (unstable) intrinsics APIs
#![feature(link_llvm_intrinsics)]
#![feature(asm)]              // Enable the (unstable) ASM APIs

use core::intrinsics;

unsafe { intrinsics::abort(); }

extern "C" {
    // Provides specific instructions to the linker about where it should look to find the function
    // definitions
    #[link_name = "llvm.eh.sjlj.setjmp"]
    pub fn setjmp(_: *mut i8) -> i32;
}

loop {
    unsafe { asm!("hlt"); }
}
```

### Embedded/OS development

Basic structure for an O/S program:

```rs
#![no_std]                    // Don't include the stdlib.
#![no_main]                   // Program doesn't have main()
#![feature(core_intrinsics)]
#![feature(asm)]

use core::intrinsics;
use core::panic::PanicInfo;

#[panic_handler]
#[no_mangle]
pub fn panic(_info: &PanicInfo) -> ! {
    unsafe { intrinsics::abort(); }
}

// [lang]uage items are elements of Rust implemented as libraries outside of the compiler itself; in
// this context, we don't include the stdlib.
// [E]xception [H]andler. "Personality" routines are executed at each step of the call stack unwinding.
//
#[lang = "eh_personality"]
#[no_mangle]
pub extern "C" fn eh_personality() {}

#[no_mangle]
pub extern "C" fn _start() -> ! {
    let mut framebuffer = 0xb8000 as *mut u8;

    unsafe {
        // Using write_volatile() prevents the compiler from performing certain optimizations, e.g.
        // presetting the memory before execution rather than writing during execution.
        framebuffer.offset(1).write_volatile(0x30);
        // Alternate write form.
        *(framebuffer + 1) = 0x30;
    }

    // Never return, as there's nowhere to return to.
    loop {
        unsafe { asm!("hlt"); }
    }
}
```

### Count allocations

Use the [`alloc-counter`](https://gitlab.com/sio4/code/alloc-counter) crate:

```rs
use alloc_counter::{count_alloc, AllocCounterSystem};
#[global_allocator]
static A: AllocCounterSystem = AllocCounterSystem;

let (counts, v) = count_alloc(|| {
    let mut v = Vec::new(); // no alloc
    v.push(0);              // alloc
    v.push(1);              // realloc
    v                       // return the vector without deallocating
});

assert_eq!(counts, (1, 1, 0));
```

## Benchmarking (`criterion`)

Rust has an built-in bencharking tool (`cargo bench`), but it's unstable and too minimal.

Add to `Cargo.toml`:

```toml
[[bench]]
harness = false
name = "my_benchmark"
```

and install the `cargo-criterion` crate.

Create `benches/my_benchmark.rs`

```rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use my_crate::my_function; // or, for simplicity, declare the function in this file

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| my_function(black_box(20))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

Run with `cargo criterion`.

## Internal

### Memory layout

Structs without fields are "Zero-sized types" (ZST), and don't take any space in the resulting program.

There are no guarantees about the struct underlying memory layout (unless `#[repr(C)]` is used), except that the fields are stored in the struct memory block.

Enums also don't have a guaranteed layout, except that the first byte is the tag.

### References

Reference:

- are pointers; for dynamically sized types, they also include an integer (with extra information)
- are aligned to `usize`

### Functions/closures

Space taken by variables in closures:

- when borrowing: reference
- when moving: copied variable
- nothing else; if there are no vars, closures are identical to functions

### Null pointer optimization

With an enum like this:

```rs
enum MyEnum {
  EmptyVariant
  Variant<NeverZeroType>
}
```

internally, `EmptyVariant` will be stored as `Variant` with zero bits, making the enum data structure take no space.

### Examine ASM (Assembler) output

Notes:

- optimized ASM is obviously hard to map to the source code, so annotations in general are just indicative;
- if one is testing a function whose result is not used, one must use `test::bench::black_box`, however, it's simpler to create a library and make that function public.

The rust compiler can output the asm:

```
# Release is automatically selected.
# Setting `[profile.release] debug = true` doesn't help, as the symbols are not interpreted from an
# ASM perspective; `c++filt` helps by demangling the function names. 
# The name of the `.s` file is the name of the binary, or the library (if building with `--lib`).
#
cargo rustc --release -- --emit asm
cat target/**/playground-*.s | c++filt
```

but the `cargo-asm` crate is more convenient, and it has more intuitive output:

```
# Defaults to release mode.
# `--rust`: annotate with source code
#
$ cargo asm --rust playground::c_3
 pub fn c_3() -> f32 {
 push    rax
 unsafe { rand() << 1 }
 call    qword, ptr, [rip, +, rand@GOTPCREL]
 f32::from_bits((gen_random_c_full() & F32_SIGN_BITMASK) | F32_EXP_BITMASK)
 and     eax, -1073741824
 add     eax, eax
 add     eax, 1065353216
     movd    xmm0, eax
 }
 pop     rax
 ret
```

Another way is to use the `Disassembly Explorer` VSC extension:

- extract the functions to examin (use the trick in the section below)
- emit the asm for the library (e.g. `cargo rustc --release --lib -- --emit asm`), and `c++filt` it
- follow up the extension instructions (copy/rename the `.s` file), and use it on the library functions :)

Other (not applicable/convenient) frontends:

- VS Code, in the July 2021 insiders version, has a Disassembly view (right click on a line while debugging), but it seems to be available only on C/++;
- with the CodeLLDB extension installed, choose "LLDB: Show Disassembly"->"Always" (on breakpoint, the disassembly will be opened), but it has next to no symbols.

Note that (in release mode?), breakpoints are not reliable, and it's not clear, when the debugger stops, the correspondence with the source code.

#### Prevent function inlining

In order to prevent inlining, one can move the functions in a library, and declare them public.

A convenient structure to be able to run the binary and disassemble the library is the following.

Cargo.toml (extract):

```toml
[lib]
name = "mylib"
path = "src/lib.rs"

[[bin]]
name = "mybin"
path = "src/main.rs"
```

main.rs:

```
mod lib
use crate::lib::*;
```

Use cargo-asm to disassemble:

```sh
cargo asm --rust --lib mylib::mymethod
```

WATCH OUT! The binary is still inlined; this strategy is purely for convenience.
