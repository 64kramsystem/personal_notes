# Rust Performance

- [Rust Performance](#rust-performance)
  - [User-facing](#user-facing)
    - [std::mem APIs](#stdmem-apis)
    - [Unsafe](#unsafe)
    - [FFI](#ffi)
      - [C to Rust types mapping](#c-to-rust-types-mapping)
      - [Extern functions/Build scripts](#extern-functionsbuild-scripts)
      - [C library patterns](#c-library-patterns)
    - [Uninitialized memory (`std::mem::MaybeUninit`)/Partially initialized structs](#uninitialized-memory-stdmemmaybeuninitpartially-initialized-structs)
      - [Bidimensional array struct, uninitialized and macro'ed](#bidimensional-array-struct-uninitialized-and-macroed)
    - [Manually allocating memory](#manually-allocating-memory)
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

### FFI

#### C to Rust types mapping

(types are mostly `std::os::raw`)

|            C            |            Rust             |
| :---------------------: | :-------------------------: |
|          short          |           c_short           |
|           int           |            c_int            |
|          long           |           c_long            |
|        long long        |         c_longlong          |
|     unsigned short      |          c_ushort           |
| unsigned , unsigned int |           c_uint            |
|      unsigned long      |           c_ulong           |
|   unsigned long long    |         c_ulonglong         |
|          char           |           c_char            |
|       signed char       |           c_schar           |
|      unsigned char      |           c_uchar           |
|          float          |           c_float           |
|         double          |          c_double           |
|  void * , const void *  | *mut c_void , *const c_void |
|    size_t, ptrdiff_t    |        usize, isize         |

Other types (Rust -> C):

- `char`: not `wchar_t` (no fixed implementation); `char32_t` is closer, but not guaranteed to be Unicode
- `*mut T`, `*const T`: C, C++ pointers, C++
- (`std::ffi`) `CString`, `CStr`: Strings (limited)
- `ptr::null_mut()`: null pointer

```rs
// Mapping to an incomplete struct type, like `typedef struct mytype mytype`
#[repr(C)] pub struct mytype { _private: [u8; 0] }
```

For functions accepting uninitialized memory, see [Manually allocating memory](#manually-allocating-memory).

#### Extern functions/Build scripts

Extern functions are declared in linked libraries.

```rs
extern "C" {
    static environ: *mut *mut c_char;
    fn strlen(s: *const c_char) -> usize;
}

unsafe {
    let len = strlen(CString::new("I'll be back")?.as_ptr());

    if !environ.is_null() && !(*environ).is_null() {
        let str = CStr::from_ptr(*environ).to_string_lossy();
    }
}
```

In order to link to a specific library (linker's option `-lgit2`), use `#[link(name = "my_lib_name")]`.

The location of external libraries needs to be specified; for this, use a so-called build script:

```sh
# Put in the same directory as `Cargo.toml`. In real-world cases, the path should not be absolute.
#
$ cat > build.rs << 'RUST'
fn main() {
    println!("cargo:rustc-link-search=native=/path/to/libgit2-0.25.1/build");
}
RUST

# Must specify the library location. The altenative is to statically link.
#
$ export LD_LIBRARY_PATH=/path/to/libgit2-0.25.1/build:"$LD_LIBRARY_PATH"
$ cargo run
```

The convention for a wrapper library project is:

- crates providing access to a C library are named `LIB-sys`, where `LIB` is the name of the C library
- a `-sys` crate should contain only the statically linked library and Rust modules containing extern blocks and type definitions
- higher-level interfaces then belong in crates that depend on the `-sys` crate

#### C library patterns

```rs
// Register this on initialization.
// Note that closures can't be called from C, but standard functions can, as long as they're marked
// `extern` (so that calling conventions are adapted).
//
assert_eq!(libc::atexit(shutdown), 0);

// Must exit via abort(), since we're in a C context.
//
extern "C" fn shutdown() {
    // if any error...
    std::process::abort();
}

// Manage C<>Rust strings.
// The `as_bytes` method exists only on Unix-like systems.
//
#[cfg(unix)]
fn path_to_cstring(path: &Path) -> Result<CString> {
    use std::os::unix::ffi::OsStrExt;
    Ok(CString::new(path.as_os_str().as_bytes())?)
}

// Define lifetimes when the type has no (Rust) references to another type, via PhantomData
//
pub struct Commit<'r> {
    raw: *mut raw::git_commit,
    _marker: PhantomData<&'r Repository>, // as it contained a &Repository, with a 'r lifetime
}

// Use drop in order to free C objects.
//
impl Drop for Repository {
    fn drop(&mut self) {
        raw::git_repository_free(self.raw);
    }
}
```

### Uninitialized memory (`std::mem::MaybeUninit`)/Partially initialized structs

Safe(r) version of initialization with uninitialized values:

```rs
// Simple data type
//
let mut result = MaybeUninit::<u32>::uninit();
unsafe { result.as_mut_ptr().write(0); }
let result = unsafe { um_u32.assume_init() };

// Array
// For a more sophisticated application to bidimensional arrays, see the [subsection below](#bidimensional-array-struct-uninitialized-and-macroed).
//
let mut result: [MaybeUninit<u32>; array_size] = unsafe { MaybeUninit::uninit().assume_init() };
result[0] = MaybeUninit::new(0);
// ...other entries...
let result = unsafe { mem::transmute::<_, [u32; array_size]>(result) };

// Field-by-field initialization of a struct (extract)
// See https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#initializing-a-struct-field-by-field.
//
unsafe {
  // We use a macro in order to find the address of a field from a mut point.
  //
  addr_of_mut!((*ptr).id).write(0);
}
```

Simpler but unsafe(r) versions, via different semantics/APIs:

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

### Manually allocating memory

```rs
// For simplicity, set buffer_size as multiple of PAGE_SIZE (see https://linux.die.net/man/3/posix_memalign).
// Requires `extern crate libc` in order to import `libc::_SC_PAGESIZE`.

let buffer: *mut u8 = unsafe {
    let page_size = libc::sysconf(_SC_PAGESIZE) as usize;

    // Since posix_memalign discards the existing associate memory, we can use init as null pointer.
    // We can use MaybeUninit, but in this case it's more verbose and no more useful.
    //
    let mut buffer_addr: *mut c_void = std::ptr::null_mut();

    // Allocate the memory, aligned to the page.
    //
    libc::posix_memalign(&mut buffer_addr, page_size, buffer_size);

    libc::mprotect(
        buffer_addr, buffer_size, libc::PROT_EXEC | libc::PROT_READ | libc::PROT_WRITE,
    );

    // The API accepts a 32 bits value (c_int), however it converts it to an unsigned char.
    //
    libc::memset(buffer_addr, fill_value, buffer_size);

    buffer_addr as *mut u8
};

// Deallocate, on drop
libc::munmap(buffer as *mut _, buffer_size);
```

The alternative to `null_mut()` is:

```rs
// Extract; see https://users.rust-lang.org/t/removing-the-libc-calls/62298/13

let mut buffer_addr: MaybeUninit<*mut c_void> = MaybeUninit::uninit();

libc::posix_memalign(buffer_addr.as_mut_ptr(), G.page_size, size);

// Shadow here, in order to avoid multiple assume_init().
//
let buffer_addr: *mut c_void = buffer_addr.assume_init();

// etc.etc.
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
# The name of the `.s` file is the name of the binary/library, as defined in Cargo.toml.
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

- VSC: set `disasexpl.associations`
- Cargo: set `[profile.release] debug = true`
- emit the asm for the binary and/or library (e.g. `cargo rustc --release --lib -- --emit asm`), and `c++filt` it
- follow up the extension instructions (copy/rename the `.s` file(s)) :)

Other (not applicable/convenient) frontends:

- VS Code, in the July 2021 insiders version, has a Disassembly view (right click on a line while debugging), but it seems to be available only on C/++;
- with the CodeLLDB extension installed, choose "LLDB: Show Disassembly"->"Always" (on breakpoint, the disassembly will be opened), but it has next to no symbols.

Note that (in release mode?), breakpoints are not reliable, and it's not clear, when the debugger stops, the correspondence with the source code.

#### Prevent function inlining

In order to prevent inlining, one can move the functions in a library, and declare them public.

Cargo automatically supports both library and binary management if the files are called `src/lib.rs` and `src/main.rs`, so just create the library file, and generate the asm for it.

WATCH OUT! This doesn't prevent the compiler to inline the functions in the binary!
