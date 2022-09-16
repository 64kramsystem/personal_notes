# Rust Mad Stuff

- [Rust Mad Stuff](#rust-mad-stuff)
  - [Implicit temporaries](#implicit-temporaries)

## Implicit temporaries

In some cases, e.g. dereference expressions, Rust creates a sort of temporary let binding:

```rs
&mut *mutex.lock().unwrap();
//    ^ here

fn create_foo() -> Foo { /* ... */ }
my_fn_call(&create_foo());
//          ^ here
```

The intermediate result is stored into a temporary memory location, whose lifetime is like it was a `let` binding.

However, in the first case (`DerefMut`), the lifetime is for the whole enclosing block, while in the second case, only for the method call.

Since in the following:

```rs
mutex.lock().unwrap().deref_mut();
```

`deref_mut()` takes `&self`, it's like the second case, and the temporary doesn't live long enough.
