# Technical Rust Jargons

A beginner-friendly reference for the ideas, vocabulary, and patterns that make Rust feel unfamiliar at first — and then suddenly click.

Rust is often described as hard because it asks you to think differently before the compiler will accept your code. The language is not trying to be mysterious. It is trying to make ownership, borrowing, lifetime, and concurrency explicit so the compiler can protect you from the most expensive bugs.

This repo is a glossary of the mental model behind Rust: the terms people say when they are talking about real code, not just the syntax they happen to type.

---

## Why Rust feels unfamiliar

Rust combines a few ideas that are uncommon in beginner-friendly languages:

- ownership is explicit
- references are tracked by the compiler
- mutation is controlled by borrowing rules
- errors are expressed as values instead of exceptions
- concurrency is safe by default
- the compiler is part of the learning process

That means beginners often feel like they are learning a language plus a type system plus a safety model at the same time.

The good news is that once the vocabulary clicks, the language becomes much more coherent.

---

## The core mental model

Rust is built around a few guiding ideas:

- every value has exactly one owner
- you can borrow a value without taking ownership
- references must stay valid for as long as they are used
- mutation is controlled by rules, not by convention
- the compiler enforces the model before runtime

This is why Rust often feels strict: the compiler is not being difficult, it is checking the rules that keep memory safe and concurrency predictable.

---

## Ownership

Ownership is the center of Rust.

Every value has one owner. When that owner goes out of scope, the value is dropped.

```rust
let s = String::from("hello");
println!("{}", s);
```

The value is freed automatically when `s` goes out of scope.

### Move semantics

When a value is assigned to another variable, ownership moves.

```rust
let s1 = String::from("hello");
let s2 = s1;

// s1 is no longer valid here
println!("{}", s2);
```

This is a move. The compiler prevents use-after-move bugs.

### Copy types

Some types are cheap to duplicate. They implement `Copy`.

```rust
let x = 42;
let y = x;
println!("{} {}", x, y);
```

Common `Copy` types include integers, booleans, characters, and tuples of `Copy` types.

### Clone

If you want to keep both versions, call `clone()`.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("{} {}", s1, s2);
```

`clone()` is explicit because it is a choice to pay the cost of duplication.

---

## Borrowing

Borrowing means using a value without taking ownership.

```rust
fn print_len(s: &String) {
    println!("length = {}", s.len());
}

let s = String::from("hello");
print_len(&s);
println!("{}", s);
```

Here `&s` creates an immutable reference. The original value still belongs to `s`.

### Immutable borrowing

You can have many immutable references at once.

```rust
let s = String::from("hello");
let a = &s;
let b = &s;
println!("{} {}", a, b);
```

### Mutable borrowing

If you want to mutate a value through a reference, use `&mut`.

```rust
let mut s = String::from("hello");
let r = &mut s;
r.push('!');
println!("{}", s);
```

Rust enforces a core rule:

- either one mutable borrow
- or many immutable borrows
- but not both at the same time

This is what prevents data races and invalid shared mutation.

---

## References and lifetimes

A reference is only valid for as long as the compiler can prove it is safe.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

The `'a` is a lifetime annotation. It says the returned reference is valid as long as both inputs are valid.

### Lifetime elision

Most common cases do not need explicit annotations.

```rust
fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}
```

The compiler often infers lifetimes automatically in simple, common patterns.

### Dangling references

Rust prevents references that outlive the value they point to.

This is one of the biggest conceptual differences from C and C++: the compiler rejects invalid references before runtime.

---

## `String` vs `&str`

This is one of the most common beginner confusions.

- `String` is an owned, growable string
- `&str` is a borrowed string slice

```rust
let owned: String = String::from("hello");
let borrowed: &str = &owned;
```

A `String` owns the data. A `&str` borrows it.

The slice form is often used when you want to read a string without taking ownership.

---

## Slices and collections

A slice is a view into a sequence.

```rust
let arr = [10, 20, 30, 40];
let middle = &arr[1..3];
println!("{:?}", middle);
```

Slices appear frequently in Rust APIs because they are a cheap way to borrow parts of arrays, vectors, and strings.

### `Vec<T>`

`Vec<T>` is the standard dynamic array.

```rust
let mut v = vec![1, 2, 3];
v.push(4);
println!("{:?}", v);
```

### `Option<T>`

`Option<T>` models the idea of “maybe there is a value.”

```rust
let maybe_number: Option<i32> = Some(42);
let maybe_none: Option<i32> = None;
```

This is Rust's alternative to `null`.

### `Result<T, E>`

`Result<T, E>` models success or failure.

```rust
let ok: Result<i32, &str> = Ok(10);
let err: Result<i32, &str> = Err("bad input");
```

Rust's error handling is designed around this type instead of exceptions.

---

## Traits and generics

### Trait

A trait describes behavior shared by types.

```rust
trait Speak {
    fn speak(&self);
}

struct Dog;

impl Speak for Dog {
    fn speak(&self) {
        println!("woof");
    }
}
```

Traits are central to Rust's design. They are how generic behavior is expressed.

### Generic types

Generics let a type or function work with many types.

```rust
struct Pair<T> {
    first: T,
    second: T,
}
```

This is used heavily in collections and library APIs.

### Trait bounds

A trait bound says “this generic type must implement this trait.”

```rust
fn print_debug<T: std::fmt::Debug>(value: T) {
    println!("{:?}", value);
}
```

### `impl Trait`

`impl Trait` is a concise way to say “some type that implements this trait.”

```rust
fn double(x: impl Into<i32>) -> i32 {
    x.into() * 2
}
```

### `dyn Trait`

`dyn Trait` is used for trait objects and dynamic dispatch.

```rust
let values: Vec<&dyn std::fmt::Debug> = vec![&42, &"hello"];
```

This is the runtime-polymorphism form of trait usage.

---

## Structs, enums, and pattern matching

### Structs

Structs define data.

```rust
struct User {
    id: u64,
    name: String,
    active: bool,
}
```

### Enums

Enums define a type with a finite set of variants.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
}
```

Enums are one of Rust's most powerful tools because they model state directly in the type system.

### Pattern matching

`match` is exhaustive and type-safe.

```rust
match some_value {
    Some(x) => println!("value: {}", x),
    None => println!("no value"),
}
```

Rust encourages matching all cases explicitly. This is part of why code tends to be safer and clearer.

### `if let` and `while let`

Rust also has concise patterns for common cases.

```rust
if let Some(value) = maybe_value {
    println!("{}", value);
}
```

---

## Error handling

Rust does not use exceptions as the primary error mechanism. It uses `Result` and `Option`.

```rust
use std::fs::read_to_string;

fn read_name() -> Result<String, std::io::Error> {
    let content = read_to_string("name.txt")?;
    Ok(content)
}
```

The `?` operator propagates errors cleanly.

### `panic!`

A panic means the program is stopping because it hit an unrecoverable condition.

```rust
panic!("something went very wrong");
```

Panics are not the main way to signal normal application error handling.

---

## Ownership in practice: smart pointers

### `Box<T>`

`Box<T>` allocates on the heap and gives the value a stable, owned location.

```rust
let value = Box::new(42);
println!("{}", value);
```

Useful when you need heap allocation or recursive types.

### `Rc<T>`

`Rc<T>` allows multiple owners in single-threaded code via reference counting.

```rust
use std::rc::Rc;

let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a);
```

### `Arc<T>`

`Arc<T>` is the thread-safe version of `Rc<T>`.

```rust
use std::sync::Arc;

let a = Arc::new(String::from("hello"));
let b = Arc::clone(&a);
```

### `Mutex<T>`

`Mutex<T>` provides mutual exclusion for shared state across threads.

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
```

This is the standard way to protect shared mutable state in threaded code.

---

## Concurrency and thread safety

### `Send`

A type is `Send` if it can be moved to another thread safely.

### `Sync`

A type is `Sync` if it can be shared between threads via reference safely.

These markers are part of Rust's concurrency model. The compiler uses them to prevent data races.

### `unsafe`

`unsafe` is Rust's escape hatch. It tells the compiler, “I know this is dangerous; verify it manually.”

This is used for:

- raw pointers
- FFI
- unsafe trait implementations
- low-level memory operations

Unsafe is powerful, but it should be isolated and well documented.

---

## Async, futures, and executors

Rust's async model is based on futures.

```rust
async fn fetch() -> String {
    "done".to_string()
}
```

A future represents work that may not be complete yet. An executor drives it to completion.

This is often used with runtimes like Tokio.

Key ideas:

- `async fn` creates a future
- `.await` waits for a future to complete
- executors schedule tasks
- `Send` and `Sync` still matter in async systems

---

## Macros, derives, and crates

### Macro

Macros generate code at compile time.

Rust has two commonly discussed categories:

- declarative macros (`macro_rules!`)
- procedural macros (derive and attribute macros)

### `derive`

`derive` automatically implements common traits.

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

### Crate

A crate is a Rust package or library unit.

### Module

Modules organize code and control visibility.

```rust
mod math {
    pub fn add(a: i32, b: i32) -> i32 {
        a + b
    }
}
```

### Workspace

A Cargo workspace groups multiple crates in one project.

---

## Cargo workflow

Cargo is Rust's package manager and build system.

```bash
cargo new my_project
cargo build
cargo run
cargo test
cargo check
cargo fmt
cargo clippy
cargo doc
```

Useful commands:

- `cargo check` for fast validation
- `cargo test` for test execution
- `cargo fmt` for formatting
- `cargo clippy` for linting
- `cargo doc` for documentation generation

---

## Common beginner confusions

These terms show up again and again in Rust learning:

### “Why does my value not live after I move it?”

Because ownership moved. You can no longer use the old binding.

### “Why do I need `clone()` everywhere?”

Because ownership is strict. If you want two owners, you need to intentionally duplicate or share.

### “Why does `&str` feel different from `String`?”

Because one is owned and the other is borrowed. They represent different kinds of data semantics.

### “Why do I need lifetimes?”

To prove that references do not outlive the data they point to.

### “Why does the compiler complain about borrowing?”

Because it is trying to prevent invalid aliasing, mutation through stale references, or data races.

### “Why are enums and pattern matching so important?”

Because Rust often models state as a small algebraic data type, which makes code safer and easier to reason about.

---

## Beginner-friendly learning path

A practical order for learning Rust:

1. variables, functions, and basic types
2. ownership and moves
3. borrowing and references
4. `String` vs `&str`
5. `Option` and `Result`
6. structs and enums
7. pattern matching
8. traits and generics
9. iterators and closures
10. lifetimes and async basics
11. concurrency and thread safety

This order mirrors how actual Rust understanding develops: from value ownership to shared behavior to safe concurrency.

---

## Quick glossary

- Borrowing: accessing a value without taking ownership
- Clone: explicitly duplicate an owned value
- Copy: a type duplicated by value rather than moved
- Drop: logic executed when a value goes out of scope
- Enum: a custom type with named variants
- Future: async work that may not be complete yet
- Generic: code parameterized by type
- Lifetime: how long a reference is valid
- Macro: code that generates more code
- Move: transferring ownership
- Option: a value that may be absent
- Ownership: the rule that each value has a single owner
- Pattern matching: checking values against a set of forms
- Result: a value representing success or failure
- Slice: a view into part of a collection
- Trait: shared behavior for types
- `unsafe`: opt-in low-level access that bypasses compiler guarantees
- `Vec`: dynamically-sized, heap-backed array

---

## Final takeaway

Rust is not “hard” because it is arbitrary. It is hard because it asks for a stronger mental model than many other languages.

Once you learn the vocabulary, the pieces fall into place:

- ownership explains memory safety
- borrowing explains how access is shared
- lifetimes explain how references remain valid
- traits explain reusable behavior
- enums and pattern matching explain state modeling
- `Result` and `Option` explain explicit error and absence handling

The real Rust jargon is not just words like `Send`, `Sync`, or `Clone`. It is the whole way Rust thinks about values, ownership, and responsibility.

That is the conceptual core behind the language.

---

## One-sentence summary

Rust feels unfamiliar at first because it turns memory safety, ownership, and concurrency into explicit rules the compiler enforces, but once you learn its vocabulary, the language becomes a precise and powerful way to build reliable software.
