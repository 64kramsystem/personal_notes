# Rust libraries

- [Rust libraries](#rust-libraries)
  - [Standard library](#standard-library)
    - [Traits](#traits)
      - [Default](#default)
      - [Copy, Clone, Drop and their relationships](#copy-clone-drop-and-their-relationships)
      - [Display](#display)
      - [[Try]From/Into](#tryfrominto)
      - [Index[Mut]](#indexmut)
      - [Deref[Mut]](#derefmut)
      - [AsRef[Mut]/Borrow[Mut]](#asrefmutborrowmut)
    - [Sorting](#sorting)
      - [Sorting floats](#sorting-floats)
    - [Base I/O (reading/writing)](#base-io-readingwriting)
      - [Spawning a process (and piping to it) (executing commands)](#spawning-a-process-and-piping-to-it-executing-commands)
    - [File operations](#file-operations)
    - [Directories/filenames/paths handling](#directoriesfilenamespaths-handling)
    - [File/directory operations/informations](#filedirectory-operationsinformations)
    - [Testing](#testing)
      - [Integration tests](#integration-tests)
    - [Collections](#collections)
    - [COW (clone-on-write)](#cow-clone-on-write)
    - [TCP client/server](#tcp-clientserver)
    - [Commandline arguments (basic)](#commandline-arguments-basic)
    - [O/S, Processes](#os-processes)
    - [Blackbox (nightly)](#blackbox-nightly)
  - [Rust utilities (`cargo install`)](#rust-utilities-cargo-install)
    - [Cargo crates handling (`cargo-edit`)](#cargo-crates-handling-cargo-edit)
  - [Crates](#crates)
    - [Partial/more flexible defaults (`smart-default`)](#partialmore-flexible-defaults-smart-default)
    - [Random](#random)
      - [`rand`](#rand)
      - [fastrand/Other crates](#fastrandother-crates)
    - [Regular expressions (`regex`)](#regular-expressions-regex)
    - [Date/times (standard)](#datetimes-standard)
    - [Date/times (`chrono`)](#datetimes-chrono)
    - [Commandline parsing (`clap`)](#commandline-parsing-clap)
    - [Terminal interaction (`termion`)](#terminal-interaction-termion)
    - [HashMap literals (`maplit`)](#hashmap-literals-maplit)
    - [Faster/Perfect hashing (`ahash`, `phf`)](#fasterperfect-hashing-ahash-phf)
    - [Channels: Single Producer Multiple Consumers (`bus`)](#channels-single-producer-multiple-consumers-bus)
    - [Unit testing](#unit-testing)
      - [RSpec-style testing (`rspec`)](#rspec-style-testing-rspec)
      - [Macro-based RSpec-style testing (`demonstrate`)](#macro-based-rspec-style-testing-demonstrate)
      - [Cucumber](#cucumber)
      - [Utilities](#utilities)
    - [Static/global variables (`lazy_static`, `once_cell`, `thread_local!`)](#staticglobal-variables-lazy_static-once_cell-thread_local)
    - [Single-time initialization (`std::sync::Once`)](#single-time-initialization-stdsynconce)
    - [Concurrency (multithreading) tools (`rayon`/`crossbeam`)](#concurrency-multithreading-tools-rayoncrossbeam)
    - [Enum utils, e.g. iterate (`strum`)/convert(`num`)](#enum-utils-eg-iterate-strumconvertnum)
    - [Convenience macros for operator overloading (`auto_ops`)](#convenience-macros-for-operator-overloading-auto_ops)
    - [Indented Heredoc-like strings (`indoc`)](#indented-heredoc-like-strings-indoc)
    - [User directories (`dirs`)](#user-directories-dirs)
    - [Traverse filesystem (`walkdir`)](#traverse-filesystem-walkdir)
    - [Error conveniences (`failure`, `thiserror`)](#error-conveniences-failure-thiserror)
    - [Clipboard management (`cli-clipboard`, `copypasta`)](#clipboard-management-cli-clipboard-copypasta)
    - [De/serialization](#deserialization)
      - [`serde`/`bincode`](#serdebincode)
      - [TOML, simplest parsing (`toml`)](#toml-simplest-parsing-toml)
      - [Guaranteed endianness storage (`byteorder`)](#guaranteed-endianness-storage-byteorder)
    - [Resource-light image size detection (`imagesize`)](#resource-light-image-size-detection-imagesize)
  - [C2Rust: Port C to Rust](#c2rust-port-c-to-rust)

## Standard library

### Traits

#### Default

```rust
// Generates a `::default()` method that fills the fields with the default values (0 for numeric, false for bool).
//
#[derive(Default)]
struct SomeOptions {
  foo: i32,
  bar: bool,
}

// Override the defaults.
//
SomeOptions { bar: true, ..Default::default() };
```

See [smart-default crate](#partialmore-flexible-defaults-smart-default) for partial/more flexible defaults.

#### Copy, Clone, Drop and their relationships

See:

- https://stackoverflow.com/questions/51704063/why-does-rust-not-allow-the-copy-and-drop-traits-on-one-type
  - copy of `Copy` data is done via trivial `memcpy`; if drop was performed on a `Copy`+`Drop` copy, the original instance could include reference to invalid (not cleaned up) data
- https://www.reddit.com/r/rust/comments/8laxam/why_does_copy_require_clone
  - `Clone` is a supertrait of `Copy`

#### Display

Simple implementation of Display:

```rust
impl std::fmt::Display for MyType {
  fn fmt(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
    write!(formatter, "value: {}", self.value)
  }
}
```

#### [Try]From/Into

Implement when representing conversions from a type; the `Into` (`into()`) trait will be automatically handled.

```rust
struct Number {
  value: i32,
}

impl From<i32> for Number {
  fn from(item: i32) -> Self {
    Number { value: item }
  }
}

let num = Number::from(30);
let num: Number = int.into();
```

Important: `From/To` don't guarantee that the trait implementation is cheap.

The respective `Try` versions are fallible (return `Option`):

```rs
// See [num](#num) for convenient enum casting
//
impl TryFrom<u32> for dirtype {
    type Error = ();

    fn try_from(value: u32) -> Result<Self, Self::Error> {
        FromPrimitive::from_u32(value).ok_or(())
    }
}
```

#### Index[Mut]

Allow convenient map-like access to a struct.

```rust
struct Registers {
    SP: u16,
    PC: u16,
}

impl std::ops::Index<Register> for Registers {
    type Output = u16;

    fn index(&self, register: Register) -> &Self::Output {
        match register {
            Register::SP => &self.SP,
            Register::PC => &self.PC,
        }
    }
}

// IndexMut requires Index to be implemented.
impl IndexMut<Register> for Registers {
    fn index_mut(&mut self, register: Register) -> &mut Self::Output {
        match register {
            Register::SP => &mut self.SP,
            Register::PC => &mut self.PC,
        }
    }
}

// Access
//
let addr = self.registers[src_register] as usize;
self.registers[dst_register] = 0x21;
```

#### Deref[Mut]

```rs
struct Selector<T> {
    elements: Vec<T>,
    current: usize,
}

impl<T> Deref for Selector<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.elements[self.current]
    }
}

let s = Selector {
    elements: vec!['x', 'y', 'z'],
    current: 2,
};

// Example of Deref
assert_eq!(*s, 'z');

// Deref coercions are not applied to satisfy type bounds (generics)!
//
fn show_it_generic<T: Display>(thing: T) { println!("{}", thing) }
//
show_it_generic(&s); // error
show_it_generic(&*s); // works - manually deref
```

#### AsRef[Mut]/Borrow[Mut]

`AsRef` represents a type where `&T` can be borrowed efficiently. Typically used to make functions flexible:

```rs
// Example implementors: String, str...
fn open<P: AsRef<Path>>(path: P) -> Result<File>

// Accepts String, &str...
fn sound<S: AsRef<str>>(&self, base: S)
```

Use `AsRef` when possible, rather than creating a new `AsFoo` trait!

`Borrow` is like `AsRef`, but it requires that the two types must hash and compares the same! It's designed specifically for hash maps etc. Example:

```rs
// not good! requires owned value
fn get(&self, key: K) -> Option<&V> { ... }

// not good! converting &str -> &String requires `&"str".to_string()`
fn get(&self, key: &K) -> Option<&V> { ... }

// good!
fn get<Q: ?Sized + Eq + Hash, K: Borrow<Q>>(&self, key: &Q) -> Option<&V>
```

`String` for example, implements `Borrow<str>`, since they guarantee the requirements.

### Sorting

Definitions:

- `PartialEq`: the type is a partial equality relation (floats isn't, because NaN != NaN); allows comparison with asserts.
- `Eq`: the type is an equality relation; good practice to implement if applies.

Watch out! The f64 doesn't support Ord (which is often required); only PartialOrd.

#### Sorting floats

Convenient solutions:

```rs
// Super-simple, and includes NaN, but be aware of the ordering (https://doc.rust-lang.org/std/primitive.f32.html#method.total_cmp)
floats.sort_by(f32::total_cmp);

// Simple, if there is no NaN
floats.sort_by(|a, b| a.partial_cmp(b).unwrap());

// If NaN can be included; no exact location for NaN
floats.sort_by(|a, b| a.partial_cmp(b).unwrap_or(std::cmp::Ordering::Less));
```

Generic solution, supporting NaN; requires a wrapper class (and a truckload of boilerplate):

```rust
// Implementation that considers NaN as greater than the other floats, and equal to itself.
// This doesn't conform to the standard.
//
pub struct SortableFloat(pub f64);

// This only informs the compiler that the type supports (full) equivalence.
//
impl Eq for SortableFloat {}

impl PartialEq for SortableFloat {
  fn eq(&self, other: &Self) -> bool {
    if self.0.is_nan() {
      other.0.is_nan()
    } else {
      self.0 == other.0
    }
  }
}

impl Ord for SortableFloat {
  fn cmp(&self, other: &Self) -> Ordering {
    let (lhs, rhs) = (self.0, other.0);

    if let Some(result) = lhs.partial_cmp(&rhs) {
      result
    } else {
      if lhs.is_nan() {
        if rhs.is_nan() {
          Ordering::Equal
        } else {
          Ordering::Greater
        }
      } else {
        Ordering::Less
      }
    }
  }
}

impl PartialOrd for SortableFloat {
  fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
    Some(self.cmp(other))
  }
}
```

In order to hash, if there are no NaN, one can hash the bits!:

```rs
impl Hash for SortableFloat {
    fn hash<H: std::hash::Hasher>(&self, state: &mut H) {
        if self.0.is_nan() {
            panic!("NaN is not accepted!");
        }
        self.0.to_bits().hash(state);
    }
}
```

### Base I/O (reading/writing)

(there APIs other than the described)

```rust
// std::io::Read/Write
//
use std::io::prelude::*;                        // Convenience, when using io module methods
r.read_to_end(&mut vec_buffer) -> Result<usize> // Read until the EOF; binary; content is appended
r.read_exact(&mut buf)?                         // Read exactly buf.len() bytes
r.take(n)                                       // Create an adapter reading at most n bytes
w.write_all(&mut buf)?                          // Write the whole buffer to a writer (e.g. File)
w.write(&mut buf)?                              // Prefer `write_all()` to `write()`, since the latter doesn't guarantee that the whole buffer is written!
w.flush()?

// There's no fast write; just open File and write to it.
fs::read(path)?                        // binary content; filename can be relative
fs::read_to_string(path)?              // valid UTF-8

// Copy
//
let bytes_copied = std::io::copy(&reader, &mut writer)?  // copy a reader into a writer until EOF is reached

// Use arrays/vecs as Read.
//
fn <R: Read> myread(mut r: R) {
  r.read_exact(&mut my_buffer).unwrap(); // requires a mutable instance
};
myread(vec![0, 1, 2].as_slice()); // it's fine to pass an immutable slice
myread(&[0, 1, 2][..]);           // don't forget that slice is a reference, not a range of an array!
```

Buffered operations:

```rs
// Read (e.g. BufReader)
let len = reader.read_line(&mut line)?
let lines = reader.lines().collect::<Result<Vec<_>, _>>()?
for line in reader.lines() { println!("{}", line?); }

// Write. Dont' forget about write() vs. write_all()
// BufWriter flushes when the buffer is full, not at the end of each line.
//
let mut stream = BufWriter::new(TcpStream::connect("127.0.0.1:34254").unwrap());
stream.write(&[666]).unwrap();
BufWriter::with_capacity(size, writer)     // custom buffer size

// Per-line buffered write; convenient if each line must be immediately available.
let mut writer = LineWriter::new(file);
writer.write_all(b"I like pizza!\n")?;

// `Vec` can be trivially used as `StringIO` equivalent:

BufReader::new(&str.as_bytes());
BufReader::new(vec.as_slice());
BufWriter::new(vec);
```

Other types:

- `io::Cursor`                  : for more complex operations (ie. involving seek)
- `io::stdin()`, `io::stdout()` : they have mutexes; require invoking `lock()`
- `io::sink`                    : discard all the data sent
- `io::repeat(byte)`
- `io::empty()`

Section of APIs that handle size/errors:

```rs
let len = match reader.read(&mut buf) {
  Ok(0) => return Ok(written),
  Ok(len) => len,                                               // size handling
  Err(ref e) if e.kind() == ErrorKind::Interrupted => continue, // COOL! match guard, to handle a specific error type (and continue)
  Err(e) => return Err(e),
};
```

#### Spawning a process (and piping to it) (executing commands)

Base spawning/executing:

```rs
Command::new("echo").args(["-n", "abc"]).output()?;

// Replace the current process with a new one (Linux-only; see https://stackoverflow.com/a/53479765):
//
Command::new("xdg-open").args([filename]).exec();
```

Piping:

```rs
let mut child = Command::new("grep").arg("foo").stdin(Stdio::piped()).spawn()?;

// The #stdin type is `process::ChildStdin`
let mut child_stdin = child.stdin.take().unwrap();

for word in my_words {
  writeln!(child_stdin, "{}", word)?;
}

drop(child_stdin); // close grep's stdin, so it will exit
child.wait()?;
```

### File operations

```rust
let f = File::open("log.txt")?             // read mode (no write/create)
let mut f = File::create("log.txt")        // write mode; if existing, truncate

let f: File = OpenOptions::new()           // Open file R/W + create
                  .create(true)            // use create_new(true) to fail if the file exists
                  .write(true)             // append() is also available
                  .read(true)
                  .open("log.txt");

// Don't forget that reads/writes move the current position.

file.set_len(len)?;               // truncate/extend; the cursor position stays the same (if was end, and the file was shrunk, it will be past the end!)
file.seek(SeekFrom::Start(i64))?; // also: SeekFrom::{End,Current}
file.seek(SeekFrom::Current(0))?; // get current position
```

### Directories/filenames/paths handling

(for user paths/directories, see the crates section)

WATCH OUT!! Path separators in simple strings are not translated across platforms; the correct way to build paths is `PathBuf::new(...).join(...)`).

OsStr and Path are strings type that can handle invalid UTF-8 filenames (first is owned):

- `PathBuf`/`Path`: full pathnames; it has utility methods;
- `OsString`/`OsStr`: single part of a filename.

```rust
let p: PathBuf = PathBuf::from("/path/to/file");
let p: PathBuf = Path::new(ASSETS_PATH).join("triangles.obj");
let p: PathBuf = Path::new('/path').to_owned();

// Methods common to Path and PathBuf.
//
let p: Option<&OsStr> = path.file_name(); // Ruby basename(); for the poor man's version, use String#split
let p: Option<&OsStr> = path.file_stem(); // Filename without extension/path. Must do the (very)

// Convenient APIs
//
pathbuf.pop()                                  // Remove the last child (but doesn't return it)
path.strip_prefix("parent")?                   // Can be used to find out a path relative to another
path.parent()?                                 // Find the parent path
pathbuf.canonicalize()?                        // Ruby File.expand_path

// Conversions (methods available to both Path/PathBuf):
//
// - to borrowed types is easy;
// - to owned (String) is easy (but lossy) via to_string_lossy() (which returns Cow<str>).
//
let p: &OsStr         = path.as_os_str();
let p: Option<&str>   = path.to_str();
let p: String         = path.to_string_lossy().into_owned();

// The conversions are quite ugly (without the "quite"). The simplest way is to go through &str
//
any
  .unwrap()         // where necessary
  .to_str()
  .unwrap()
  .to_string()

// WATCH OUT! Splitting chains can be tricky in some contexts, due to temporary values.
// One can go through OsString, which is even uglier, and it doesn't implement fmt::Display (but it
// implements fmt::Debug).

Path::new(os_str.unwrap());  // Convert OsStr to Path
```

### File/directory operations/informations

There are some other APIs, e.g. for traversing directories.

```rust
path.exists()                   // test if file/dir exists (Ruby File.exists?)
Path::exists(path)              // ^^ (same)
path.is_dir()                   // test if is directory; also: is_file(), is_symlink()

fs::remove_file(path)
fs::remove_dir_all(path)        // (rm -r)

fs::create_dir(path)            // create a directory (mkdir)
fs::create_dir_all(path)        // create a directory (mkdir -p)

fs::copy(src, dest)             // copy a file (cp -p)
fs::rename(src, dest)           // copy a file (cp -p)
fs::rename(src, dest)           // copy a file (cp -p)

std::os::unix::fs::symlink(target, dest) // OS-specific (use `std::os::unix::prelude::*` for others)
fs::canonicalize(path)          // Ruby File.expand_path
file.metadata()?                // metadata (file stats), including permissions(), len(), is_dir(); follows symlinks
fs::metadata(path)              // ^^ (same)
fs::set_permissions(path)

// Read a directory's file basenames non-recursively.
// The stdlib doesn't have glob APIs; use `walkdir` crate for a full dir traversal solution.
// WATCH OUT! The result is not sorted! `.` and `..` are not included.
// `path()` is the basename with the path sent to read_dir().
//
// Other `DirEntry` APIs: `metadata()`, `file_type()`
//
let entries: ReadDir = std::fs::read_dir("/dev").unwrap();
entries
    .map(|entry| {
      entry.unwrap()             // DirEntry
        .file_name()             // base filename, OsString
        .into_string().unwrap()  // String
    })
    .collect::<Vec<String>>()

// Rigorous way of filtering + mapping a directory's filenames
// This also shows how to work with full path names.
//
entries
    .filter_map(|entry| {
        let path = entry.unwrap().path();
        let file_name = path.file_name().unwrap().to_str().unwrap();

        if file_name.starts_with(JOYSTICK_BLOCK_DEVICE_FILENAME_PREFIX) {
            Some(path.into_os_string().into_string().unwrap())
        } else {
            None
        }
    })
    .collect::<Vec<String>>()
```

### Testing

WATCH OUT! UTs are run in parallel by default!! In order to run specific tests serially, must use the [serial_test crate](https://crates.io/crates/serial_test).

```rust
fn my_method() -> u32 {
  1
}

#[cfg(test)]
mod tests {
  // If really required, since this is a child module, this allows private functions to be tested.
  //
  use super::*;

  #[test]
  fn test_my_method() {
    assert_eq!(my_method(), 1);
    assert_ne!(my_method(), 0);
    assert!(my_method() == 1, "Value not matching!");
  }

  #[test]
  #[should_panic(expected = "yikes!")]
  fn test_my_method_panic() {
    panic!("yikes!")
  }

  #[test]
  fn test_with_result() -> Result<(), &'static str> {
    if true {
      Ok(())
    } else {
      Err("Error!")
    }
  }

  #[test]
  #[ignore] // Not included unless specified
  fn slow_test() {
    thread::sleep(Duration::from_secs(1));
  }
}
```

Cargo options:

```sh
cargo test $test_name_substring      # no patterns/regexes supported
cargo test -- --test=test-threads=1  # run serially
cargo test -- --nocapture            # don't steal stdout output
cargo test -- --ignored              # include ignored tests
```

#### Integration tests

Integration tests are placed in the standard location `$project_root/tests`; run using `cargo test`.

```rust
// `#[cfg(test)]` is not required (assumed because of the directory), but the import is, because
// integration tests are not in scope
//
use mylibrary;

#[test] // blahblah
```

If one wants to write a shared module, put it in a submodule (eg. file `tests/common/mod.rs`), otherwise, it's interpreted as integration test suite.

When testing binary crates, don't forget that binary crates can't expose functions to be used by other crates (one of the reasons why `main.rs` is canonically thin an imports `lib.rs`).

### Collections

`HashSet`/`BTreeSet` (un/sorted sets):

```rust
let set = BTreeSet::new()

set.insert(10);
set.insert(4);
set.insert(1981);

set.contains(88)     // false
set.remove(123)      // false
set.replace(&value)  // replace a value with another
set.clear();

set.first();      // 4
set.last();       // 1981

// Whole-set operations; there are other APIs
set.is_subset(&BTreeSet);
set.is_superset(&BTreeSet);
set.is_disjoint(&BTreeSet);  // disjoint: no elements in common
set.intersection(&BTreeSet).clone().collect();
set.union(&BTreeSet).clone().collect();
```

For sorting floats, see [sorting section](#sorting-floats).

`VecDeque` (double-ended queue, implemented via ring buffer):

```rust
let mut mailbox = VecDeque::new();
mailbox.push_back(item); mailbox.push_front(item)
mailbox.pop_front(); mailbox.pop_back()
mailbox.front(); mailbox.back()                     // With `_mut()` variation

// There are other in-depth APIs
```

`BinaryHeap` (priority queue):

```rs
push(); pop()
peek()
peek_mut()
```

There is a `LinkedList`, however, according to Programming Rust, there are better alternatives.

### COW (clone-on-write)

Avoids allocations, when possible. Example, for strings:

```rs
// If the var is None, we use a &str, avoiding the allocation of a String.
//
// For strings, the stdlib has special support for Cow<'a, str>, so the into() version can be used.
//
fn get_name() -> Cow<'static, str> {
    std::env::var("USER")
        .map(|v| Cow::Owned(v))                       // v.into()
        .unwrap_or(Cow::Borrowed("whoever you are"))  // "...".into()
}

// Based on the condition, we get an owned instance or a borrowed one, which we unify via Cow; the
// alternative is to clone `name.0`, which causes an allocation.
//
let text: Cow<String> = if condition {
    Cow::Owned(format!("{} : {} hp", &name.0, health.current))
} else {
    Cow::Borrowed(&name.0)
};
draw_batch.print(screen_pos, text);
```

### TCP client/server

Client:

```rust
let mut read_buffer = String::new();
let mut connection = TcpStream::connect("127.0.0.1:8080")?;

connection.write_all("Hello".as_bytes())?; // requires `std::io::Read`

connection.shutdown(Shutdown::Write)?; // Read/Write/Both

let bytes_read = connection.read_to_string(&mut read_buffer)?; // requires `std::io::Write`
```

Server:

```rust
let listener = std::net::TcpListener::bind("127.0.0.1:8080")?;
for stream in listener.incoming() { handle_client(stream?) } // Watch out (Result!)
```

### Commandline arguments (basic)

Don't forget that the first is the binary filename.

```rust
std::env::args();                                       // only valid Unicode; can collect to Vec<String>
std::env::args_os();                                    // returns `OsString`s, which are not restricted to Unicode
let exe: io::Result<PathBuf> = std::env::current_exe(); // current binary/executable path
let exe: io::Result<PathBuf> = std::env::current_dir(); // working directory
```

### O/S, Processes

```rust
std::env::consts::OS;               // values: https://doc.rust-lang.org/std/env/consts/constant.OS.html
std::process::exit(exit_status);    // terminate program (exit)
std::env::var("CASE_INSENSITIVE");  // access environment variables
```

In order to spawn a process and pipe I/O, see the [specific section](#spawning-a-process-and-piping-to-it).

### Blackbox (nightly)

Be pessimistic about the side effects of this function (typically used not to discard values/functions in the compiled code); doesn't make any absolute guarantee.

```rust
#![feature(bench_black_box)]

let abc = std::hint::black_box(-1);
```

## Rust utilities (`cargo install`)

### Cargo crates handling (`cargo-edit`)

```sh
# The crate location (name/repo/path) is detected automatically.
#
cargo add [--features=a,b] $crate_ref            # by default, the version specifier is the latest `^M.m.p` (with minor upgrade strategy)
--vers=$version --branch=$branch                 # version modifiers
```

## Crates

### Partial/more flexible defaults (`smart-default`)

```rust
#[derive(SmartDefault)]
pub struct Sphere {
  #[default(_code = "Self::new_id()")] // the method can be private
  pub id: u32,
  #[default(Matrix::identity(4))]
  pub transformation: Matrix,
  #[default(Material::default())]
  pub material: Material,

  // All the fields need to have a default; the non-meaningful defaults can be overridden on an instantiation
  // method, e.g. new().
}

let sphere1 = Sphere::default();

let mut sphere2 = Sphere {
  material: Material { /* .. */ },
  ..Sphere::default()
};
```

### Random

Poor man's random:

```rust
extern "C" {
  fn srand() -> u32;
  fn rand() -> u32;
}

unsafe {
  srand();
  println!("{}", rand());
}
```

#### `rand`

`rand` is the most common crate, and it's a relatively large one.

```rust
// Shortcut API; invokes `thread_rng().gen()`.
//
let rand_byte: u8 = rand::random();

// Ends: [low, high); requires `rand::Rng`.
//
let randval: u8 = rand::thread_rng().gen_range(0..2);

// Fetching a random element from a vector
//
use rand::seq::SliceRandom;
let entry: Option<&MyType> = vec.choose(&mut rand::thread_rng());

// Use thread_rng
// `thread_rng()`: seeded by the O/S; local to the current thread.
//
use rand::prelude::*;
let mut rng = rand::thread_rng();
let randval: f64 = rng.gen();

let mut data = [0u8; 32];
rand::thread_rng().fill_bytes(&mut data);

// Alternative way of creating a series of random values; this allows to set the range.
let range = Uniform::new(0, 20);
let values: Vec<u64> = rand::thread_rng().sample_iter(&range).take(100).collect();
```

Use a deterministic generator, for testing purposes:

```rust
// Bools are converted from (little endian) i32 (4 bytes); the highest bit determines the value.

use rand::{rngs::adapter::ReadRng, Rng, RngCore};

let data = [true, false, true, true, false, false, true]
    .iter()
    .flat_map(|f| vec![0, 0, 0, (*f as u8) << 7])
    .collect::<Vec<_>>();

let mut rng: Box<dyn RngCore> = Box::new(ReadRng::new(data.as_slice()));
let rand_bool = rng.gen::<bool>();

// Sample type with user-definable RNG.
//
struct MyRng<'a> {
  rng: Box<dyn RngCore + 'a>,
}
impl<'a> MyRng<'a> {
  pub fn new(rng: Option<Box<dyn RngCore + 'a>>) -> Self {
    let rng = rng.unwrap_or_else(|| Box::new(thread_rng()));
    MyRng { rng }
  }
}
```

#### fastrand/Other crates

For other crates, see https://sr.ht/~icefox/oorandom.

A convenient one for small projects is [`fastrand`](https://github.com/smol-rs/fastrand):

```rust
fastrand::bool();
fastrand::u32(..); // range

// Random array element (Ruby sample())
let vec_i = fastrand::shuffle(..vec.len());
vec[vec_i];

// Shuffle an array
fastrand::shuffle(&mut vec);

// Manual seed
fastrand::seed(7);
```

### Regular expressions (`regex`)

Remember that capture 0 is the whole string (as standard).

```rust
let re = Regex::new(r"(\d{4})(\d{2})").unwrap();
let text = "201203, 201301, 201407";

// The match is by substring, unless specified.
//
re.is_match(text); // true

// Flags are specified inside the expression.
// Other flag: `m`, `s`, `x` (and other minor ones)
//
Regex::new(r"(?i)\w+").unwrap();

// If the regex doesn't match, None is returned.
//
if let Some(captures) = re.captures("pizza{2}") {
    // use `c.map(|m| m.as_str())` if the captures are NOT guaranteed
    println!("{:?}", captures.get(0).unwrap().as_str());
}

// COOL!!!! Convert to Vec<_>, and pattern match.
//
let values = re
        .captures(text)
        .unwrap()
        .iter()
        .skip(1)                         // skip global capture
        .map(|c| c.unwrap().as_str())
        .collect::<Vec<_>>();

// If the captures are not guaranteed, must handle the `Option<_>`
//
if let [v1, n1, v2, n2, v3, n3] = values.as_slice() {
    FaceWithNormal((*v1, *n1), (*v2, *n2), (*v3, *n3))
} else { unreachable!() }

// The is also a captures_iter(), although it's a bit odd
for cap in re.captures_iter(text) {
    println!("Month: {} Day: {} Year: {}", &cap[2], &cap[3], &cap[1]);
}
```

Match multiple regexes:

```rust
lazy_static::lazy_static! {
  static ref VERTEX_REGEX: Regex = Regex::new(r"^v (-?1(?:\.\d+)) (-?1(?:\.\d+)) (-?1(?:\.\d+))$").unwrap();
  static ref FACE_REGEX: Regex = Regex::new(r"^f (\d+) (\d+) (\d+)$").unwrap();
}

if let Some(captures) = VERTEX_REGEX.captures(&line) {
  let x: f64 = captures[1].parse().unwrap();
  // ...
} else if let Some(captures) = FACE_REGEX.captures(&line) {
  // ...
} else {
  // ...
}
```

### Date/times (standard)

The Instant counter doesn't stop when a sleep is issued.

```rust
// Compute time elapsed.
//
let current_time = SystemTime::now();
current_time.elapsed();

// Get current time in seconds; also supports `as_nanos()` etc.
//
let current_time_secs = SystemTime::now()
    .duration_since(UNIX_EPOCH)
    .unwrap()
    .as_secs();
```
`SystemTime` is not monotonic; `Instant` is, which is important for gameloops, benchmarks etc. See [Instance manpage](https://doc.rust-lang.org/std/time/struct.Instant.html) for the details.

Simplified game loop:

```rust
let cycle_start_time = Instant::now();
some_work();
let next_cycle_time = cycle_start_time + Duration::new(0, 1_000_000_000 / 500);

// Remember the sleep time is the *minimum* quantity. For very small sleeps (e.g. 0.1ms), a sleep
// of 2.5 times as specified has been observed; this is expected in O/S implementations.
//
thread::sleep(next_cycle_time - Instant::now());
```

WATCH OUT! While `Instant`s can be compared, it's not possible to perform an operation whose result is negative.

### Date/times (`chrono`)

WATCH OUT! Check accurately how the chrono crate is more convenient than the stdlib version.

WATCH OUT! `std::time::Duration` doesn't support negative values.

```rust
let start:DateTime<Utc> = Utc::now();                   // Current time

DateTime::parse_from_str(string, "%d.%m.%Y %H:%M %P %z")?;   // Parse a datetime, with timezone
NaiveTime::parse_from_str(string, "%H:%M:%S")?;              // Parse a time
NaiveDateTime::parse_from_str(string, "%Y-%m-%d %H:%M:%S")?; // Parse a datetime, without timezone

start + Duration::days(3);                              // Arithmetic
start.checked_add(Duration::days(3));                   // Safe arithmetic
end.checked_sub(start);                                 // Returns Option<Duration> (none if negative)
naive_time_1 + naive_time_2                             // Returns duration
nt1 + nt2 + Duration::seconds(nt3.second() as i64);     // Sample arithmetic with 3+ (odd) NaiveTime

// Conversions/tests

naive_time.second();                                    // requires `Timelike`
duration.num_seconds();

duration < Duration::zero();
```

### Commandline parsing (`clap`)

Notes:

- In order to check boolean flags, use `ArgMatches#is_present(<str>)`;
- `Arg#takes_value(<bool>)` defaults to false;
- `Arg#index(<int>)` indicates the position of the argument (for positional arguments).

Example of varargs, partially encapsulated:

```rust
fn parse_commandline_arguments<'a>(args: &'a Vec<String>) -> clap::ArgMatches {
    App::new("test")
        .setting(AppSettings::TrailingVarArg)
        .arg(
            Arg::with_name("INTERVALS")
                .required(true)
                .index(1)
                .multiple(true),
        )
        .get_matches_from(args)
}

fn extract_interval_arguments<'a>(matches: &'a clap::ArgMatches) -> Vec<&'a str> {
    matches
        .values_of("INTERVALS")
        .unwrap()
        .collect::<Vec<&str>>()
}

fn main() {
    let commandline_args = std::env::args().collect::<Vec<String>>();
    let matches = parse_commandline_arguments(&commandline_args);
    let intervals = extract_interval_arguments(&matches);
}
```

Example, fully encapsulated, but owned:

```rust
fn decode_commandline_args() -> Vec<String> {
  let commandline_args = std::env::args().collect::<Vec<String>>();

  let matches = App::new("test")
    .setting(AppSettings::TrailingVarArg)
    .arg(
      Arg::with_name("INTERVALS")
        .required(true)
        .index(1)
        .multiple(true),
    )
    .get_matches_from(commandline_args);

  matches
    .values_of("INTERVALS")
    .unwrap()
    .map(|arg| arg.to_string())
    .collect::<Vec<String>>()
}
```

Examples of extended form:

```rust
let matches = App::new("My Super Program")
    .version("1.0")
    .author("Kevin K. <kbknapp@gmail.com>")
    .about("Does awesome things")
    .arg(
        Arg::with_name("config")
            .short("c")
            .long("config")
            .value_name("FILE")
            .help("Sets a custom config file")
            .takes_value(true),
    )
    .arg(
        Arg::with_name("INPUT")
            .help("Sets the input file to use")
            .required(true)
            .index(1),
    )
    .arg(
        Arg::with_name("v")
            .short("v")
            .multiple(true)
            .help("Sets the level of verbosity"),
    )
    .subcommand(
        SubCommand::with_name("test")
            .about("controls testing features")
            .version("1.3")
            .author("Someone E. <someone_else@other.com>")
            .arg(
                Arg::with_name("debug")
                    .short("d")
                    .help("print debug information verbosely"),
            ),
    )
    .get_matches();

// Gets a value for config if supplied by user, or defaults to "default.conf"
let config = matches.value_of("config").unwrap_or("default.conf");
println!("Value for config: {}", config);

// Calling .unwrap() is safe here because "INPUT" is required (if "INPUT" wasn't
// required we could have used an 'if let' to conditionally get the value)
println!("Using input file: {}", matches.value_of("INPUT").unwrap());

// Vary the output based on how many times the user used the "verbose" flag
// (i.e. 'myprog -v -v -v' or 'myprog -vvv' vs 'myprog -v'
match matches.occurrences_of("v") {
    0 => println!("No verbose info"),
    1 => println!("Some verbose info"),
    2 => println!("Tons of verbose info"),
    3 | _ => println!("Don't be crazy"),
}

// You can handle information about subcommands by requesting their matches by name
// (as below), requesting just the name used, or both at the same time
if let Some(matches) = matches.subcommand_matches("test") {
    if matches.is_present("debug") {
        println!("Printing debug info...");
    } else {
        println!("Printing normally...");
    }
}
```

Examples of compact form:

```rust
let matches = App::new("myapp")
    .version("1.0")
    .author("Kevin K. <kbknapp@gmail.com>")
    .about("Does awesome things")
    .args_from_usage(
        "-c, --config=[FILE] 'Sets a custom config file'
  <INPUT>              'Sets the input file to use'
  -v...                'Sets the level of verbosity'",
    )
    .subcommand(
        SubCommand::with_name("test")
            .about("controls testing features")
            .version("1.3")
            .author("Someone E. <someone_else@other.com>")
            .arg_from_usage("-d, --debug 'Print debug information'"),
    )
    .get_matches();
```

### Terminal interaction (`termion`)

Operations are performed by writing to the buffer.

```rust
use std::io::Write;

let (width, height) = terminal_size()?;

let mut term: RawTerminal<Stdout> = std::io::stdout().into_raw_mode()?;

// Clear screen
write!(term, "{}", clear::All)?;

// Print text at specific coordinates
writeln!(term, "{}foo", Goto(x_u16, y_u16))?;

// Change (current) color.
//
// Working with different colors is awkard. Color is a trait, and since Fg(col)/Bg(col) don't accept
// a Boxed instance, and there are no From<> implementations, the easiest thing is to convert to
// string.
//
let fmt_color = if true {
    format!("{}", Fg(Black))
} else {
    format!("{}", Fg(White))
};
writeln!(term, "{}bar", fmt_color)?;
```

### HashMap literals (`maplit`)

(Remember that a HashMap can be created from a vector of tuples)

```rust
#[macro_use] extern crate maplit;

let map = hashmap!{
    "a" => 1,
    "b" => 2,
};
```

### Faster/Perfect hashing (`ahash`, `phf`)

Faster hashing (`ahash`):

```rs
// Convenience; the full instantiation is more verbose
use ahash::AHashMap;
let mut map: AHashMap<i32, i32> = AHashMap::new();
```

Perfect hashing (`phf`), `const`-compatible:

```rs
// Requires `macros` feature
// Must be a const; variable (`let`) is not supported.
// Enums are not supported as keys; Structs are supported as values.
//
const HAZZ: phf::Map<&str, i32> = phf_map! {
    "loop" => 4,
    "continue" => 2,
};
```

### Channels: Single Producer Multiple Consumers (`bus`)

```rust
// Buffer size. zero causes undefined behavior.
// Messages are put into the buffer, and the buffer is emptied when all the readers have read.
// There is a sweet spot in the buffer size for the performance.
// Performance need to be verified (in release mode) against the MPSC analog.
//
let mut tx = bus::Bus::new(1);

let mut rx1 = tx.add_rx();
let mut rx2 = tx.add_rx();

tx.broadcast("Hello");

thread::spawn(move || {
  println!("{}", rx1.recv().unwrap());
});

thread::spawn(move || {
  println!("{}", rx2.recv().unwrap());
});
```

### Unit testing

In order to display `println!` output, pass `-- --show-output`.

Standard testing:

```rust
// The attribute can't be applied to a block, so the alternative it apply it once per each test module.

#[cfg(test)]
mod tests {
  // If code under test refers to types using the name of current create, it must be imported with
  // alias, e.g. `use crate as serdine`.

  use super::*;

  #[test]
  fn test_sort() {
    // If files needs to be opened, the current path is the root of the current project (not workspace
    // root!); can also use `CARGO_MANIFEST_DIR`.
    //
    let test_file_path = Path::new(&std::env::var("CARGO_MANIFEST_DIR").unwrap()).join("test_data/deserialize.dat");

    let mut $collection = &mut vec![3, 2, 1];
    $stat
    let expected_collection = &vec![1, 2, 3];
    assert_eq!($collection, expected_collection);
  }
}
```

#### RSpec-style testing (`rspec`)

Neat!! See [repository](https://github.com/rust-rspec/rspec).

#### Macro-based RSpec-style testing (`demonstrate`)

Repository [here](https://github.com/austinsheep/demonstrate).

It's cool, but also annoying, because relatively large suites have usability problems due to macros being used.

```rust
use demonstrate::demonstrate;

// If a single instance needs to be shared between UTs ("before all"), then trickery is needed.
// The Mutex is required for mutability.
//
// A simpler approach to this is to use an empty mutex (`Mutex<()>`) and initialize on each UT.
//
use std::sync::Mutex;
unsafe impl Send for MyInterface {}
static INTERFACE: Mutex<MyInterface> = Mutex::new(MyInterface::init("test"));

demonstrate! {
  // Not needed in this case; here for reference.
  use super::*;

  describe "test module 2" {
    before { let context = 5; }
    subject { context + 5 }

    it "used a shared object" {
      INTERFACE().lock().unwrap().read_pixel(0, 0)
    }

    it "test subject" {
      assert_eq!(subject, 10);
    }

    #[should_panic]
    it "should fail" {
        None::<()>.unwrap();
    }

    context "context is 5" {
      it "should equal 5" {
        assert_eq!(context, 5);
      }
    }
  }
}
```

There is a [Cucumber crate](https://github.com/bbqsrc/cucumber-rust). As of Nov/2020, the README tutorial [is broken](https://github.com/bbqsrc/cucumber-rust/issues/90), so see [another tutorial](https://www.florianreinhard.de/2020-10-05/cucumber-07-in-rust-beginners-tutorial).

#### Cucumber

There are two options - the stable version, and the master version, which has macros that make the test structure arguably more readable; this section uses the latter.

Update the configuration:

```sh
cat >> Cargo.toml <<TOML
async-trait = "0.1.42" # This is currently required to properly initialize the world in cucumber-rust
futures = "0.3.8" # You can use a different executor if you wish

[[test]]
name = "cucumber"
harness = false # Allows Cucumber to print output instead of libtest

[dev-dependencies]
cucumber_rust = { git = "https://github.com/bbqsrc/cucumber-rust", branch = "main", features = ["macros"] }
TOML
```

Add the feature(s):

```sh
mkdir features

cat > features/examples.feature <<CUCUMBER
Feature: Example feature

  Scenario: An example scenario
    Given I am trying out Cucumber
    When I consider what I am doing
    Then I am interested in ATDD
    And we can implement rules with regex
CUCUMBER
```

and the test(s):

```sh
mkdir tests

cat > tests/cucumber.rs <<'RUST'
use std::{cell::RefCell, convert::Infallible};

use async_trait::async_trait;
use cucumber_rust::{given, then, when, World, WorldInit};

#[derive(WorldInit)]
pub struct MyWorld {
    // You can use this struct for mutable context in scenarios.
    foo: String,
    bar: usize,
    some_value: RefCell<u8>,
}

impl MyWorld {
    async fn test_async_fn(&mut self) {
        *self.some_value.borrow_mut() = 123u8;
        self.bar = 123;
    }
}

#[async_trait(?Send)]
impl World for MyWorld {
    type Error = Infallible;

    async fn new() -> Result<Self, Infallible> {
        Ok(Self {
            foo: "wat".into(),
            bar: 0,
            some_value: RefCell::new(0),
        })
    }
}

#[given("a thing")]
async fn a_thing(world: &mut MyWorld) {
    world.foo = "elho".into();
    world.test_async_fn().await;
}

#[when(regex = "something goes (.*)")]
async fn something_goes(_: &mut MyWorld, _wrong: String) {}

#[given("I am trying out Cucumber")]
fn i_am_trying_out(world: &mut MyWorld) {
    world.foo = "Some string".to_string();
}

#[when("I consider what I am doing")]
fn i_consider(world: &mut MyWorld) {
    let new_string = format!("{}.", &world.foo);
    world.foo = new_string;
}

#[then("I am interested in ATDD")]
fn i_am_interested(world: &mut MyWorld) {
    assert_eq!(world.foo, "Some string.");
}

#[then(regex = r"^we can (.*) rules with regex$")]
fn we_can_regex(_: &mut MyWorld, action: String) {
    // `action` can be anything implementing `FromStr`.
    assert_eq!(action, "implement");
}

fn main() {
    let runner = MyWorld::init(&["./features"]);
    futures::executor::block_on(runner.run_and_exit());
}
RUST
```

Fire!:

```sh
cargo test --test cucumber
```

#### Utilities

```rust
// `serial_test` allows marked tests to be run serially; works with `demonstrate`!
//
#[serial]
it "mytest" { /* ... */ }

#[serial]
it "mytest2" { /* ... */ }

// assert_float_eq: nice, but doesn't support messages
//
assert_float_absolute_eq!(3.0, 3.0);      # default epsilon = 1e-6
assert_float_absolute_eq!(3.0, 3.9, 1.0);

// The [Spectral](https://github.com/cfrancia/spectral) create has fluent assertions
//
asserting(&"test condition").that(&1).is_equal_to(&2);
```

### Static/global variables (`lazy_static`, `once_cell`, `thread_local!`)

Static variables can be defined inside a function (c-style) or at the root level; they need a const expression. They are unsafe when mutable. The difference from const is that they're associated with only one memory location.

```rust
static HELLO_WORLD: u32 = 1000;
```

Vanilla mutable static vars are unsafe; the safe alternatives are using [atomics](rust.md#atomic-primitive-type-wrappers) (if the architecture supports it) or wrapping with a `Mutex`/`RwLock` (both Rust 1.63+).

Thread-local variables:

```rust
// In stdlib.
//
thread_local!(static TL_VAR: RefCell<u32> = RefCell::new(123));

TL_VAR.with(|tl_var_cell| {
  let mut tl_var_ref = tl_var_cell.borrow_mut();
  *tl_var_ref = 32;
});

TL_VAR.with(|tl_var_cell| {
  let tl_value = tl_var_cell.borrow();
  println!("TLV: {}", tl_value); // 32
});
```

### Single-time initialization (`std::sync::Once`)

```rs
// Makes sure that this initialization is invoked only once, even if called multiple times.
//
fn ensure_initialized() {
    static ONCE: std::sync::Once = std::sync::Once::new();
    ONCE.call_once(|| unsafe {
        check(raw::git_libgit2_init()).expect("initializing libgit2 failed");
        assert_eq!(libc::atexit(shutdown), 0);
    });
}
```


### Concurrency (multithreading) tools (`rayon`/`crossbeam`)

Easy parallel iteration!!!:

```rust
use rayon::prelude::*;

// Use this in order to limit the used threads (or use the `RAYON_NUM_THREADS` env var); by default,
// all the hw threads are used.
//
rayon::ThreadPoolBuilder::new()
    .num_threads(8)
    .build_global()
    .unwrap();

array.par_iter()
     .map(|&i| i * i)
     .sum()

range.into_par_iter().for_each(|v| { /* parallel operation */ });
```

Both Rayon and [Crossbeam](https://github.com/crossbeam-rs/crossbeam) provide tools for concurrent programming:

```rust
// Threads pooling. Queue size is chosen automatically based on cores.
//
rayon::join(|| quicksort(a), || quicksort(&mut b[1..]))
```

### Enum utils, e.g. iterate (`strum`)/convert(`num`)

```rust
// include both `strum` and `strum_macros`.

use strum_macros::EnumIter;

#[derive(Copy, Clone, Debug, EnumIter)]
pub enum Flag {
  z,
  n,
  h,
  c,
}

use strum::IntoEnumIterator;

for flag in Flag::iter() { /* ... */ }
```

Convert enums from numeric types (requires `num`, `num-traits`, `num-derive`):

```rs
#[derive(FromPrimitive, ToPrimitive)]
pub enum dirtype { east = 1, north = 0 }

impl TryFrom<u32> for dirtype {
    type Error = ();
    fn try_from(value: u32) -> Result<Self, Self::Error> {
        FromPrimitive::from_u32(value).ok_or(())
    }
}

impl From<grtype> for u16 {
    fn from(value: grtype) -> Self {
        value.to_u16().unwrap()
    }
}

let mydt: dirtype = 1_u32.try_into().unwrap();
let myu16: u16 = dirtype.into();
```

### Convenience macros for operator overloading (`auto_ops`)

See [repository](https://github.com/carbotaniuman).

### Indented Heredoc-like strings (`indoc`)

```rust
use indoc::indoc;

// Ruby squiggly heredoc-alike syntax.
//
let expected_string = indoc! {"
    P3
    5 3
    255
    0 0 0 0 0 0 0 0 0 0 0 0 0 0 255 0 0 0 0 0 0 0 128 0 0 0 0 0 0 0 255 0
    0 0 0 0 0 0 0 0 0 0 0 0 0
"};
```

### User directories (`dirs`)

Aside the temporary directory, there's not builtin way in Rust to find the home (and related) user directories.

Generic create search: https://crates.io/search?q=home

```rust
// stdlib temp dir
//
let tmpdir: PathBuf = std::env::temp_dir();

// `dirs` crate
//
dirs::home_dir();                     // Returns Option<PathBuf>
dirs::desktop_dir();

dirs::home_dir().unwrap().join(path); // home
```

### Traverse filesystem (`walkdir`)

The walker is recursive by default.

```rs
let walker = WalkDir::new(search_path) // raises an error if search_path doesn't exist
    .min_depth(1)                      // if not specified, the base dir is included
    .max_depth(max)
    .into_iter()
    .filter_entry(|e| {                // only one filter_entry() call is allowed
        if self.stop_search {          // no measurable performance hit
            return false;
        };
        // filter out entries; WATCH OUT! don't exclude directories whose content need to be filtered,
        // otherwise, they'll be skipped entirely.
        !self.skip_entry(e)
    });

// Perform inclusion filtering here.
//
walker
    .into_iter()
    .filter_map(|e| Self::include_entry(&e.unwrap(), pattern))
```

See [pm-spotlight](https://github.com/64kramsystem/pm-spotlight/blob/d99f6798/src/search/file_searcher.rs#L95) for example of skipping/including entries.

### Error conveniences (`failure`, `thiserror`)

```rust
// Convient Error types handling:

use failure_derive::*;

#[derive(Fail, Debug)]
pub enum BlobError {
    #[fail(display = "No Room")]
    NoRoom,
    #[fail(display = "Too Big {}", 0)]
    TooBig(u64),
    #[fail(display = "Bincode {}", 0)]
    Bincode(bincode::Error),
}

// Easy error messages definition:

use thiserror::Error;

#[derive(Error, Debug)]
#[error("{message:} ({line:}, {column})")]
pub struct JsonError {
    message: String,
    line: usize,
    column: usize,
}
```

### Clipboard management (`cli-clipboard`, `copypasta`)

There are (at least) two crates:

- `cli-clipboard`: for terminal programs
- `copypasta`: for GUI programs; did *not* work correctly

### De/serialization

#### `serde`/`bincode`

Serde basics:

```rs
// serde = {version = "1.0.136", features = ["derive"]}
// serde_json = "1.0"

#[derive(Serialize, Deserialize, Debug)]
struct MyStruct {}

let serialized = serde_json::to_string(&MyStruct {})?;
let deserialized: MyStruct = serde_json::from_str(&serialized)?; // must specify the type
```

Bincode is a convenient binary format+routines (uses `serde` as backend):

```rust
bincode::deserialize_from(read)?;      // deserialize from Read
bincode::serialize_into(write, &val)?; // serialize to Write

// Example of a blob, where the first 8 (usize) bytes are the length of the array, and the other
// the array.
//
let val_len: usize = bincode::deserialize_from(r)?;
let mut val_buffer = vec![0_u8; k_len];
r.read_exact(&mut val_buffer)?;

// Deserialization functions, showcasing the types involved (owned/borrowed)

pub fn deserialize<V: DeserializeOwned>(bindata: &[u8]) -> Result<V, bincode::Error> {
    bincode::deserialize(bindata)
}

pub fn deserialize_b<'d, V: Deserialize<'d>>(bindata: &'d [u8]) -> Result<V, bincode::Error> {
    bincode::deserialize(bindata)
}
```

#### TOML, simplest parsing (`toml`)

For simple parsing (no comments, etc.), use `toml`.

Prelude:

```rs
// Convenient copy/paste
let config_filename = dirs::home_dir().unwrap().join(CONFIG_BASENAME); // `dirs` crate
let config_str = fs::read_to_string(config_filename).unwrap();

// For the examples
let doc_str = r#"
    mystr = "abc"
    myarr = ["1", "2"]
"#;
```

Most general approach, using `serde`:

```rs
#[derive(Deserialize)]
struct Doc {
    mystr: String,      // Use Option when an entry may not be present
    myarr: Vec<String>, // If a field, the error is not bad, but better to raise a manual one
}

let doc: Doc = toml::from_str(doc_str).unwrap();

println!("{:?}", doc.mystr);
println!("{:?}", doc.myarr);
```

Parse using `Value`:

```rs
let doc = doc_str.parse::<Value>().unwrap();

let mystr: &str = doc["mystr"].as_str().unwrap();
let myarr: Vec<&str> = doc["myarr"]
    .as_array()                // Parses as Vec<Value>
    .unwrap()
    .iter()
    .map(|v| v.as_str().unwrap())
    .collect();
```

If a structure entries are only of one type, one can parse into a single data type:

```rs
// Assumes a document made only of `strings = array` entries.
//
// Using HashMap<&str, Value> is possible to add flexibility, but it defeats the simplicity of this approach.
//
let doc: HashMap<&str, Vec<&str>> = toml::from_str(&doc_str).unwrap();

let myarr = &doc["myarr"];
```

Finally, the full-blown deserialized approach:

```rs
#[derive(Deserialize)]
struct Config {
    mystr: String,
    myarr: Vec<String>,
}

fn main() {
    let str = r#"
        mystr = "abc"
        myarr = ["1", "2"]
    "#;

    let config: Config = toml::from_str(str).unwrap();

    println!("{:?}", config.mystr);
    println!("{:?}", config.myarr);
}
```

#### Guaranteed endianness storage (`byteorder`)

```rs
f.read_u32::<LittleEndian>()?           // read an u32 stored in little endian format
```

### Resource-light image size detection (`imagesize`)

Small crate, and reads the smallest possible amount of data.

```rs
let ImageSize { width, height } = imagesize::size(path).unwrap();
```

## C2Rust: Port C to Rust

Requires the compile commands; see [C notebook strategies](c.md#find-compilation-commands).

```sh
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 $dir
# The binary parameter is the name of the main() Rust file, without extension.
# `--reorganize-definitions` should merge the header values (see https://immunant.com/blog/2019/12/header_merging),
# however, in the test project, it seems it didn't merge anything (or almost), while adding considerable
# noise (`c2rust::src_loc` attribute).
c2rust transpile --binary catacomb compile_commands.json
```

The [blog](https://immunant.com/blog) contains handling of complex real-world cases.
