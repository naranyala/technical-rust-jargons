# Technical Rust Jargons

A beginner-friendly reference for the ideas, vocabulary, and patterns that make Rust feel unfamiliar at first—and then suddenly click.

Rust combines low-level control with compile-time checks for memory safety and thread safety. The compiler asks you to be explicit about who owns a value, how it is shared, and how long references remain valid. This guide introduces those ideas and shows how they appear in everyday Rust. It is a companion to hands-on practice, not a substitute for [The Rust Book](https://doc.rust-lang.org/book/) or the [standard library documentation](https://doc.rust-lang.org/std/).

## Table of contents

- [Why Rust feels different](#why-rust-feels-different)
- [The core mental model](#the-core-mental-model)
- [Ownership, moves, and copying](#ownership-moves-and-copying)
- [Choosing function parameters](#choosing-function-parameters)
- [Borrowing and references](#borrowing-and-references)
- [Lifetimes](#lifetimes)
- [Strings, slices, and collections](#strings-slices-and-collections)
- [Structs, enums, and pattern matching](#structs-enums-and-pattern-matching)
- [Traits and generics](#traits-and-generics)
- [Errors: `Option` and `Result`](#errors-option-and-result)
- [Iterators and closures](#iterators-and-closures)
- [Modules, crates, and packages](#modules-crates-and-packages)
- [Collections cheat sheet](#collections-cheat-sheet)
- [Smart pointers and interior mutability](#smart-pointers-and-interior-mutability)
- [Concurrency and thread safety](#concurrency-and-thread-safety)
- [Async Rust](#async-rust)
- [Macros and derives](#macros-and-derives)
- [Cargo, tests, and compiler errors](#cargo-tests-and-compiler-errors)
- [Common beginner questions](#common-beginner-questions)
- [Quick glossary](#quick-glossary)
- [Suggested learning order](#suggested-learning-order)
- [Further learning](#further-learning)

---

## Why Rust feels different

Rust puts ownership, borrowing, and many thread-safety requirements into the type system. The compiler checks these rules before the program runs, helping prevent many memory-safety bugs and data races in safe code. It does not prove a program is bug-free: logic errors, deadlocks, and incorrect results are still possible.

## The core mental model

- A value has an owner responsible for its lifetime and cleanup.
- Moving a value transfers ownership; borrowing lets code use it without taking ownership.
- References must not be used beyond the validity of the data they refer to.
- A value can be accessed through multiple shared references or one exclusive mutable reference at a time, subject to borrowing rules.
- The compiler checks these rules before runtime.

In short: **ownership governs responsibility for a value; borrowing governs temporary access to it.**

---

## Ownership, moves, and copying

When an owned value's owner goes out of scope, Rust drops it and runs its cleanup behavior.

```rust
let name = String::from("Ada");
println!("{name}");
// `name` is dropped at the end of its scope.
```

### Moves

For many types, assigning a value to another binding moves it. The new binding becomes the owner; the old binding can no longer be used.

```rust
let first = String::from("hello");
let second = first;
println!("{second}");
// `first` was moved and cannot be used here.
```

Passing an owned value to a function can move it too. Functions can also return ownership.

### `Copy` and `Clone`

Types such as integers, `bool`, and `char` implement `Copy`; assignment copies their value, leaving the original usable. Tuples are `Copy` when all their elements are `Copy`.

```rust
let x = 42;
let y = x;
println!("{x} {y}");
```

`Clone` explicitly produces another value. Its cost depends on the type; cloning a `String`, for example, copies its contents into a new allocation.

```rust
let first = String::from("hello");
let second = first.clone();
println!("{first} {second}");
```

Types can customize cleanup with `Drop`. Rust normally invokes this automatically when a value leaves scope.

## Choosing function parameters

A useful first question is: **Does this function need to own the value, just read it, or modify it?**

- Take `T` when the function needs ownership, will store the value, or should consume it.
- Take `&T` when it only needs to read a value temporarily.
- Take `&mut T` when it needs to modify the caller's value.

```rust
fn byte_len(text: &str) -> usize {
    text.len()
}

fn add_exclamation(text: &mut String) {
    text.push('!');
}

fn take_and_return(text: String) -> String {
    text
}

let mut message = String::from("hello");
let length = byte_len(&message);
add_exclamation(&mut message);
let message = take_and_return(message);
println!("{message} has {length} bytes before the change");
```

For text inputs, `&str` is usually more flexible than `&String`: it accepts string literals and borrowed views of owned strings. Choose ownership based on what the function does, not simply to silence a compiler error.

---

## Borrowing and references

A reference lets you access a value without taking ownership. Shared references (`&T`) allow reading; mutable references (`&mut T`) allow mutation.

```rust
let mut text = String::from("hello");
let shared = &text;
println!("{shared}");

let mutable = &mut text;
mutable.push('!');
println!("{mutable}");
```

A shared borrow may coexist with other shared borrows. A mutable borrow must be exclusive while it is used. Rust's non-lexical lifetimes mean a borrow can end after its last use, even before its variable leaves scope. The important rule is that incompatible borrows cannot be used at the same time.

This prevents unsafe aliasing and data races in safe Rust; it does not prevent every kind of concurrency bug.

### Borrow-checker examples

Valid: the shared borrows are used together.

```rust
let text = String::from("hello");
let first = &text;
let second = &text;
println!("{first} {second}");
```

Invalid: the shared borrow is used after a mutable borrow is created.

```rust,compile_fail
let mut text = String::from("hello");
let shared = &text;
let mutable = &mut text;
println!("{shared} {mutable}");
```

A reference to a local value cannot escape the function that owns it. Return an owned `String`, or borrow data supplied by the caller instead.

```rust,compile_fail
fn dangling() -> &str {
    let text = String::from("hello");
    &text
}
```

---

## Lifetimes

A lifetime describes how long a reference is valid and expresses relationships between references. An annotation does not make data live longer.

```rust
fn longer<'a>(left: &'a str, right: &'a str) -> &'a str {
    if left.len() > right.len() { left } else { right }
}
```

The returned reference is valid only for a scope supported by both inputs; `'a` does not say they have exactly the same actual lifetime.

Rust infers lifetimes in many common signatures:

```rust
fn first_word(text: &str) -> &str {
    text.split_whitespace().next().unwrap_or("")
}
```

Because there is one input reference, Rust can infer that the returned reference is tied to it. Most code uses references without explicit lifetime annotations; write them when a relationship needs to be made clear.

---

## Strings, slices, and collections

- `String` is owned, growable UTF-8 text.
- `&str` is a borrowed view of UTF-8 text.
- A slice such as `&[T]` is a borrowed view into a sequence.
- `Vec<T>` is an owned, growable sequence.

```rust
let owned = String::from("hello");
let borrowed: &str = &owned;

let numbers = [10, 20, 30, 40];
let middle = &numbers[1..3];
println!("{borrowed}; {middle:?}");
```

Rust strings are UTF-8, so integer indexing does not return a character: characters can take multiple bytes. Use `.chars()`, `.char_indices()`, or byte operations as appropriate. Slice indexing can panic if a range is invalid; `.get(range)` returns an `Option` instead.

---

## Structs, enums, and pattern matching

A struct groups fields into a type. An enum describes a value that can be one of several variants, each of which can carry different data.

```rust
struct User {
    id: u64,
    name: String,
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
}
```

`match` handles alternatives and must cover every possible case, explicitly or with a catch-all pattern.

```rust
let maybe_value: Option<i32> = Some(7);
match maybe_value {
    Some(value) => println!("value: {value}"),
    None => println!("nothing here"),
}
```

`if let` is concise when only one pattern matters:

```rust
if let Some(value) = maybe_value {
    println!("{value}");
}
```

Enums and exhaustive matching make states and alternatives explicit.

---

## Traits and generics

A trait defines behavior that types can implement. Generics let code work with multiple concrete types; trait bounds specify required behavior.

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

fn print_debug<T: std::fmt::Debug>(value: T) {
    println!("{value:?}");
}
```

In argument position, `impl Trait` accepts a concrete type implementing the trait. In return position, it hides the name of one concrete return type.

```rust
fn show(value: impl std::fmt::Display) {
    println!("{value}");
}

fn numbers() -> impl Iterator<Item = i32> {
    0..3
}
```

`dyn Trait` is used for dynamic dispatch through a trait object, behind a pointer such as `&dyn Trait` or `Box<dyn Trait>`.

```rust
let values: Vec<&dyn std::fmt::Debug> = vec![&42, &"hello"];
```

---

## Errors: `Option` and `Result`

`Option<T>` represents a value that may be absent (`Some(value)` or `None`). `Result<T, E>` represents success (`Ok(value)`) or failure (`Err(error)`). These types make expected outcomes explicit.

```rust
use std::{fs, io};

fn read_name() -> Result<String, io::Error> {
    let content = fs::read_to_string("name.txt")?;
    Ok(content)
}
```

The `?` operator returns an error to the caller if the expression is an `Err`; otherwise it unwraps the success value. The function's return type must support the error being propagated.

Handle a result with `match` when both cases need explicit behavior:

```rust
match "42".parse::<i32>() {
    Ok(number) => println!("parsed {number}"),
    Err(error) => eprintln!("not a number: {error}"),
}
```

`unwrap()` and `expect()` panic if the value is an error or `None`. They can be reasonable in tests or when a failure truly means a broken invariant, but for expected failures prefer handling or propagating the result. Libraries such as [`thiserror`](https://docs.rs/thiserror/) help define domain errors; [`anyhow`](https://docs.rs/anyhow/) is often used for application-level error propagation.

A panic signals that normal execution cannot continue from that point. It is generally not the right way to report expected conditions like invalid user input or a missing file.

---

## Iterators and closures

A closure is an anonymous function that can capture values from its environment. Closures may capture by shared borrow, mutable borrow, or by value; `Fn`, `FnMut`, and `FnOnce` describe how they can be called.

Iterator adapters such as `map` and `filter` are often lazy: they do work when a consumer such as `collect`, `sum`, or a loop asks for values.

```rust
let total: i32 = [1, 2, 3].iter().map(|number| number * 2).sum();
let evens: Vec<_> = (0..10).filter(|number| number % 2 == 0).collect();
println!("{total}; {evens:?}");
```

### `iter()`, `iter_mut()`, and `into_iter()`

These methods express different ownership choices:

- `iter()` yields shared references to items (`&T`); the collection remains usable.
- `iter_mut()` yields mutable references (`&mut T`); the collection remains owned by the caller.
- `into_iter()` consumes the collection and yields its items by value (`T`).

```rust
let mut names = vec![String::from("Ada"), String::from("Lin")];
for name in names.iter_mut() {
    name.push('!');
}
let lengths: Vec<usize> = names.iter().map(String::len).collect();
let owned_names: Vec<String> = names.into_iter().collect();
```

For collections, `into_iter()` consumes the collection. Array iteration has edition-related details, so when teaching or targeting a specific edition, check the relevant method resolution and compiler edition.

---

## Modules, crates, and packages

These terms are related but not interchangeable:

- A **module** organizes names and controls visibility within a crate.
- A **crate** is a compilation unit, such as a library or binary crate.
- A **package** is a Cargo unit described by `Cargo.toml`; a package can contain one or more crates.

A common layout includes `Cargo.toml`, `src/main.rs` for a binary crate, `src/lib.rs` for a library crate, and `tests/` for integration tests. `pub` exposes items subject to Rust's module visibility rules; items are otherwise private to their module and descendants.

---

## Collections cheat sheet

| Type | Useful for |
| --- | --- |
| `Vec<T>` | Growable sequence that preserves insertion order |
| `VecDeque<T>` | Double-ended queue |
| `HashMap<K, V>` | Key-value lookup; iteration order is not guaranteed |
| `BTreeMap<K, V>` | Key-value map with sorted keys |
| `HashSet<T>` | Unique values; iteration order is not guaranteed |
| `BTreeSet<T>` | Unique values in sorted order |
| `String` | Owned, growable UTF-8 text |
| `&str` | Borrowed view of UTF-8 text |

Choose a collection based on the operations and ordering your program needs.

---

## Smart pointers and interior mutability

- `Box<T>` owns a value on the heap and provides indirection.
- `Rc<T>` enables shared ownership in a single-threaded program.
- `Arc<T>` enables shared ownership across threads when its contents satisfy the necessary thread-safety requirements.

Reference counting does not itself make contents mutable; reference cycles can also keep allocations alive indefinitely.

Interior mutability permits mutation through a shared reference under additional rules:

- `Cell<T>` and `RefCell<T>` are single-threaded tools. `RefCell<T>` checks borrowing rules at runtime and can panic if they are violated.
- `Mutex<T>` and `RwLock<T>` synchronize access across threads.

Use these when ordinary mutable borrowing does not fit, and account for their runtime checks or synchronization costs.

---

## Concurrency and thread safety

- `Send` means a type can be transferred to another thread safely.
- `Sync` means shared references to a type can be shared across threads safely; formally, `T` is `Sync` when `&T` is `Send`.

These marker traits are often implemented automatically when a type's fields meet the requirements. `Arc` provides atomic reference counting, but does not make arbitrary interior mutation safe.

A mutex provides synchronized mutable access. Locking it returns a guard that grants access while the lock is held:

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
{
    let mut value = counter.lock().expect("mutex was poisoned");
    *value += 1;
}
```

`unsafe` permits certain operations the compiler cannot verify, such as dereferencing raw pointers or calling unsafe functions. It does not switch off all compiler checks: the programmer must uphold each operation's safety requirements. Keep unsafe code limited and encapsulate it behind a safe API when possible.

---

## Async Rust

An `async fn` returns a future. Creating a future does not by itself run it to completion; an executor polls futures and schedules tasks. `.await` lets an async function yield while a future is pending, rather than necessarily blocking the operating-system thread.

```rust
async fn fetch_label() -> String {
    "done".to_string()
}
```

Async tasks are often multiplexed over a smaller number of threads, which is useful for many I/O-bound operations. Async and threads can also be used together. Runtimes such as Tokio provide executors and facilities such as network I/O and timers.

---

## Macros and derives

Macros generate or transform code during compilation. Declarative macros are commonly written with `macro_rules!`; procedural macros include derive and attribute macros.

`derive` asks the compiler to generate implementations for supported traits, provided the fields meet the trait's requirements:

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

---

## Cargo, tests, and compiler errors

Cargo is Rust's package manager and build tool. It creates projects, resolves dependencies, builds code, runs tests, and generates documentation.

```sh
cargo new my_project
cd my_project
cargo check
cargo test
cargo run
cargo fmt
cargo clippy
cargo doc --open
```

- `cargo check` checks whether code compiles without producing a final executable.
- `cargo test` builds and runs tests.
- `cargo fmt` formats Rust code; `cargo clippy` runs additional lints.
- Cargo also manages dependencies and workspaces, which group related packages.

A unit test can live alongside the code it tests:

```rust
fn add(left: i32, right: i32) -> i32 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn adds_two_numbers() {
        assert_eq!(add(2, 3), 5);
    }
}
```

Integration tests live in the package's `tests/` directory and test the public API from outside the library crate. Run both kinds with `cargo test`.

### Reading a compiler error

When code does not compile:

1. Start with the first error; later errors can be consequences of it.
2. Read the primary message and the highlighted source span.
3. Check the related notes and help text, including where a borrow began and where it is later used.
4. Make the smallest change that reflects your intent: transfer ownership, borrow immutably, or borrow mutably.
5. Re-run `cargo check` or `cargo test` to verify the change.

For example, if a function only reads a `String`, taking `String` by value may move it and prevent the caller from using it afterward. Change the parameter to `&str` (or `&String`, when appropriate) if the function only needs a temporary read. Do not add `.clone()` automatically; first decide whether the function really needs an independent owned value.

---

## Common beginner questions

### Why do I need `clone()`?

You may be trying to create another owned value when borrowing or transferring ownership would suffice. Clone only when you need another value; the operation can be expensive.

### Why can’t I use a value after moving it?

Ownership was transferred to another binding or function. Borrow it instead if the recipient only needs temporary access.

### Why does `&str` feel different from `String`?

`String` owns growable text; `&str` borrows a view of text. Use `&str` for read-only text parameters in many APIs.

### Why can’t I borrow mutably while a value is borrowed immutably?

A mutable reference requires exclusive access. An overlapping shared reference could otherwise observe mutation unexpectedly. A borrow can end after its last use.

### Why do lifetimes exist?

They let the compiler check that references are not used after their referenced data becomes invalid.

### Why does `unsafe` exist?

Some low-level operations cannot be verified by the compiler. `unsafe` makes the programmer responsible for upholding their safety requirements.

### Why do Rust examples use `Result`?

`Result` represents recoverable failure as data, allowing callers to handle or propagate errors explicitly.

---

## Quick glossary

- **Borrowing:** access through a reference without taking ownership.
- **Cargo:** Rust's package manager and build tool.
- **Closure:** an anonymous function that can capture values from its environment.
- **Clone:** explicitly produce another value; its cost depends on the type.
- **Copy:** a trait for values copied rather than moved in ordinary assignments and calls.
- **Crate:** a Rust compilation unit, such as a library or binary.
- **Drop:** cleanup performed when a value is dropped.
- **Enum:** a type with a defined set of variants.
- **Future:** a value representing asynchronous work that can be polled.
- **Generic:** a type or function parameterized over other types.
- **Interior mutability:** mutation through shared ownership using runtime checks or synchronization.
- **Lifetime:** a description of reference validity and relationships between references.
- **Macro:** compile-time code generation or transformation.
- **Move:** transfer of ownership from one place to another.
- **Module:** a unit for organizing names and controlling visibility within a crate.
- **Option:** a type representing a value that may be absent.
- **Package:** a Cargo unit described by `Cargo.toml` that may contain one or more crates.
- **Result:** a type representing success or failure.
- **Slice:** a borrowed view into a sequence.
- **Trait:** a definition of behavior that types can implement.
- **`unsafe`:** operations that require the programmer to uphold safety requirements.
- **`Vec<T>`:** an owned, growable sequence of values.

---

## Suggested learning order

1. Variables, functions, and basic types
2. Structs and enums
3. `Option` and `Result`
4. Ownership and moves
5. Borrowing and references
6. `String`, `&str`, and slices
7. Pattern matching
8. Collections, iterators, and closures
9. Traits and generics
10. Modules, packages, crates, and Cargo
11. Lifetimes and more complex borrowing
12. Smart pointers and interior mutability
13. Concurrency and thread safety
14. Async Rust and macros, as needed

You do not need to master every topic before building something. Let small experiments and compiler messages guide what to learn next.

## Further learning

- [The Rust Programming Language (The Book)](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [The Rust standard library](https://doc.rust-lang.org/std/)
- [The Cargo Book](https://doc.rust-lang.org/cargo/)

