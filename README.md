# Rust Programming Jargons, Idioms, and Mental Models

A beginner-friendly reference for the ideas, vocabulary, and patterns that make Rust feel unfamiliar at first — and then suddenly click.

This repository is built as a practical catalog: not just a glossary, but a guide to how Rust thinks.

Rust has a lot of jargon because it is solving several difficult problems at once:

- memory safety without garbage collection
- predictable performance
- strong concurrency guarantees
- explicit error handling
- expressive but safe abstractions

The language’s terms are not random. They correspond to real rules and mental models.

---

## Quick start: what you need to understand first

If you only learn five things, learn these:

1. Ownership — every value has one owner.
2. Borrowing — access data without taking ownership.
3. Lifetimes — references must stay valid.
4. Traits — shared behavior contracts.
5. `Option` and `Result` — Rust’s way of modeling absence and failure.

Everything else in Rust usually builds on top of these.

---

## Why Rust feels jargon-heavy

Rust vocabulary is dense because the compiler is enforcing correctness at compile time.

That means the language often names concepts directly:

- `Copy` means a value can be duplicated cheaply.
- `Drop` means a value does cleanup when it goes out of scope.
- `Send` means it is safe to move across threads.
- `Sync` means safe shared access across threads.
- `unsafe` means the compiler cannot verify the safety invariant.

These are not just keyword names. They describe meaningful guarantees.

---

## Fast mental model

Think of Rust as a language organized around these pillars:

- ownership: who owns the value?
- borrowing: who can access it and for how long?
- lifetimes: how long is a reference valid?
- traits: what behavior does this type support?
- safety: what invariants must be preserved?

If a Rust error feels confusing, it is often because one of these pillars is being violated.

---

## Recommended reading order for beginners

1. Ownership and borrowing
2. `String` vs `&str`
3. `Option` and `Result`
4. Structs and enums
5. `match` and pattern matching
6. Traits and generics
7. Iterators and closures
8. Concurrency basics
9. Async fundamentals
10. `unsafe` only when needed

---

## Core concepts

### Ownership

Ownership is the heart of Rust.

Every value has exactly one owner. When ownership moves, the previous binding can no longer be used.

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}", s2);
// println!("{}", s1); // compile error
```

Why it matters:

- no double-free bugs
- no use-after-free bugs
- no hidden aliasing bugs

### Move semantics

By default, Rust moves values rather than copying them.

```rust
let x = String::from("rust");
let y = x;
```

Here, `x` is moved into `y`.

### Borrowing

Borrowing lets you access a value without taking ownership.

```rust
let s = String::from("hello");
let r = &s;
println!("{}", r);
```

There are two kinds of borrows:

- `&T` — immutable borrow
- `&mut T` — mutable borrow

Rules:

- many immutable borrows are allowed
- only one mutable borrow is allowed at a time
- mutable and immutable borrows cannot overlap

### Mutability

Rust variables are immutable by default.

```rust
let mut x = 5;
x += 1;
```

This is intentional. It makes state transitions explicit and predictable.

### Scope and drop

Values are dropped when they go out of scope.

```rust
{
    let s = String::from("hello");
    println!("{}", s);
} // s is dropped here
```

`Drop` is the trait that runs cleanup logic when a value is dropped.

### Copy and Clone

Some types are `Copy` because they are cheap to duplicate bit-for-bit.

Examples:

- integers
- booleans
- characters
- floating point values
- tuples of `Copy` values

```rust
let a = 42;
let b = a; // copied, not moved
println!("{} {}", a, b);
```

`Clone` is explicit duplication for types that are not `Copy`.

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("{} {}", s1, s2);
```

### Lifetime

Lifetimes describe how long references are valid.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

The `'a` means the returned reference must live at least as long as the input references.

This is one of the biggest conceptual jumps for beginners, but it is what makes Rust references safe.

### Borrow checker

The borrow checker is the compiler component that enforces borrowing rules.

It prevents:

- use-after-free
- dangling references
- hidden aliasing
- data races

This is the central safety mechanism in Rust.

### Aliasing

Aliasing means two names point to the same data.

Rust allows safe aliasing only under strict rules:

- many immutable references are okay
- one mutable reference at a time

This gives Rust a strong safety story without a garbage collector.

### Lifetime elision

Many functions do not require explicit lifetime annotations because Rust applies a shorthand rule set called lifetime elision.

```rust
fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}
```

The compiler can often infer common lifetime patterns without you writing `'a` everywhere.

### Static lifetime

`'static` means the reference is valid for the entire lifetime of the program.

```rust
let s: &'static str = "hello";
```

This is common for string literals and global data.

### Non-lexical lifetimes (NLL)

Rust’s borrow checker uses non-lexical lifetimes: it does not require a variable to remain alive until the end of its scope if the compiler can prove it is no longer used earlier.

This is why some Rust code becomes more ergonomic over time. It reduces unnecessary borrow restrictions.

### Reborrow

A reborrow happens when you borrow a reference that itself is a reference.

```rust
let mut v = vec![1, 2, 3];
let r = &mut v;
let r2 = &mut *r;
```

This is common in nested borrowing and method-call patterns.

---

## Types and data modeling

### Primitive types

Rust has primitive types like:

- integers: `i8`, `i16`, `i32`, `i64`, `isize`
- unsigned integers: `u8`, `u16`, `u32`, `u64`, `usize`
- floats: `f32`, `f64`
- booleans: `bool`
- characters: `char`

### Strings

Rust has two important string forms:

- `String` — owned, heap-allocated, growable string
- `&str` — borrowed string slice

```rust
let owned: String = String::from("hello");
let borrowed: &str = &owned;
```

This distinction is central in Rust APIs.

### Arrays and slices

Arrays have fixed length.

```rust
let arr = [1, 2, 3, 4];
```

Slices are views into a contiguous memory region.

```rust
let slice = &arr[1..3];
```

### Vec<T>

`Vec<T>` is the standard dynamic array type.

```rust
let mut v = vec![1, 2, 3];
v.push(4);
```

### Box<T>

`Box<T>` is a heap-allocated value with a single owner.

```rust
let x = Box::new(42);
```

It is often used when you want ownership on the heap or when working with recursive types.

### Structs

Structs model data grouped into named fields.

```rust
struct User {
    id: u64,
    name: String,
    active: bool,
}
```

### Enums

Enums let you model a value that may be one of several variants.

```rust
enum Color {
    Red,
    Green,
    Blue,
}
```

Enums become especially powerful with pattern matching.

### Tuples

Tuples can combine values of different types.

```rust
let pair = (10, "hello");
let (x, y) = pair;
```

### Unit type

`()` is the unit type. It represents “nothing” when there is no meaningful value.

```rust
fn do_nothing() {}
```

### Never type

`!` is the never type, meaning the function never returns.

```rust
fn exit() -> ! {
    panic!("fatal");
}
```

### Option<T>

`Option<T>` represents “maybe a value.”

```rust
let x: Option<i32> = Some(42);
let y: Option<i32> = None;
```

This is Rust’s replacement for null-like values.

### Result<T, E>

`Result<T, E>` represents success or failure.

```rust
fn parse_num(s: &str) -> Result<i32, std::num::ParseIntError> {
    s.parse::<i32>()
}
```

This is the standard error-handling model in Rust.

### Generics

Generics let one function or type work for many values.

```rust
fn identity<T>(value: T) -> T {
    value
}
```

### Traits

A trait defines a shared behavior contract.

```rust
trait Speak {
    fn speak(&self);
}
```

Then a type can implement it.

```rust
struct Dog;

impl Speak for Dog {
    fn speak(&self) {
        println!("woof");
    }
}
```

### Trait bounds

Trait bounds constrain generic types.

```rust
fn print_debug<T: std::fmt::Debug>(value: T) {
    println!("{:?}", value);
}
```

### Associated types

Associated types let traits define one type the implementer must provide.

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

### `dyn Trait` and trait objects

`dyn Trait` is used for runtime-dispatched behavior.

```rust
fn print_value(v: &dyn std::fmt::Display) {
    println!("{}", v);
}
```

This is a form of dynamic polymorphism.

### `Default`, `From`, `Into`

These are important Rust conventions.

- `Default` means a value can be created with defaults.
- `From` creates one type from another.
- `Into` is the inverse conversion direction.

```rust
let s = String::from("hello");
let v: Vec<_> = s.into_bytes();
```

### `AsRef` and `AsMut`

These traits let you borrow a type as another type without consuming it.

```rust
fn print_path<P: AsRef<std::path::Path>>(p: P) {
    println!("{:?}", p.as_ref());
}
```

### `PhantomData`

`PhantomData<T>` tells the compiler a type logically contains `T` even though it does not store it at runtime.

It is often used in marker types and safe abstractions around raw pointers.

### `Cow`

`Cow` stands for “clone on write.” It is a type that can hold either an owned or borrowed value, and may clone lazily when mutation is needed.

This is a real-world Rust idiom for performance-sensitive APIs.

### `Sized`

`Sized` is a trait implemented for all types whose size is known at compile time.

Most generic types automatically satisfy `Sized`, but `?Sized` is used for dynamically sized types like `str` and `[T]`.

### `?Sized`

`?Sized` means the type may not have a fixed compile-time size.

```rust
fn foo<T: ?Sized>(x: &T) {}
```

This is common when working with references to dynamically sized types.

### `impl Trait`

`impl Trait` is a convenient syntax for anonymous trait types in function arguments and return types.

```rust
fn iter() -> impl Iterator<Item = i32> {
    vec![1, 2, 3].into_iter()
}
```

This is often used to hide concrete types while preserving trait behavior.

### Opaque types

An opaque type is a type whose concrete form is hidden behind a trait contract.

`impl Trait` is the Rust syntax for many opaque result types.

---

## Control flow and patterns

### `match`

`match` is one of Rust’s core control flow tools. It is exhaustive and expressive.

```rust
match value {
    0 => println!("zero"),
    1 => println!("one"),
    _ => println!("other"),
}
```

### `if let` and `while let`

Useful when you care only about one pattern.

```rust
if let Some(x) = maybe_value {
    println!("{}", x);
}
```

### `let else`

`let else` is a pattern that handles the “otherwise” branch directly.

```rust
let Some(value) = maybe_value else {
    return;
};
```

This is a common modern Rust idiom for early exit.

### `matches!`

`matches!` checks whether a value matches a pattern.

```rust
if matches!(value, Some(42)) {
    println!("found 42");
}
```

### `loop`

`loop` repeats forever until a `break` occurs.

```rust
loop {
    println!("repeat");
    break;
}
```

### Iterators

Rust encourages iteration over explicit data processing pipelines.

```rust
let doubled: Vec<_> = vec![1, 2, 3]
    .into_iter()
    .map(|x| x * 2)
    .collect();
```

### Closures

Closures are anonymous functions that can capture variables from the outer scope.

```rust
let add_one = |x| x + 1;
println!("{}", add_one(5));
```

Closures are heavily used in iterator chains and functional-style Rust.

### Destructuring

Rust supports destructuring in variables, matches, and patterns.

```rust
let point = (10, 20);
let (x, y) = point;
```

### Pattern guards

A match arm can have a guard condition after the pattern.

```rust
match value {
    x if x > 10 => println!("large"),
    _ => println!("small"),
}
```

This is a common idiom for refining matches.

### `for` loops and `into_iter`

Rust’s `for` loop is built around iterators, often using `into_iter`, `iter`, or `iter_mut`.

```rust
for item in &values {
    println!("{}", item);
}
```

This is often more idiomatic than manually indexing.

---

## Error handling and safety

### `panic!`

`panic!` is for unrecoverable failures.

```rust
panic!("bad state");
```

### `unwrap`, `expect`, `unwrap_or`

These are common convenience methods on `Option` and `Result`.

```rust
let maybe = Some(42);
println!("{}", maybe.unwrap());

let empty: Option<i32> = None;
println!("{}", empty.unwrap_or(0));
```

They are useful, but idiomatic Rust favors explicit handling when errors matter.

### The `?` operator

The `?` operator propagates errors concisely.

```rust
fn read_file() -> Result<String, std::io::Error> {
    let data = std::fs::read_to_string("file.txt")?;
    Ok(data)
}
```

This is one of the most common idioms in Rust.

### `Result` and `Option` as design language

Rust uses them not just to represent values, but to shape APIs.

This is a huge difference from languages that often use exceptions or nulls as hidden control flow.

### `thiserror` and `anyhow`

These are crates commonly used to simplify error modeling in Rust applications.

- `thiserror` — derive error types cleanly
- `anyhow` — ergonomic error propagation for application code

This is a practical ecosystem concept many Rust developers know.

### `unsafe`

`unsafe` tells the compiler to trust that the invariants are valid.

Typical uses:

- raw pointers
- FFI calls
- unsafe trait implementations
- low-level memory manipulation

```rust
unsafe {
    let ptr = 0x1234 as *const i32;
    let value = *ptr;
    println!("{}", value);
}
```

Important: `unsafe` is not the default style. It is intentionally isolated and explicit.

### Raw pointers

Rust has raw pointers:

- `*const T`
- `*mut T`

These bypass normal borrow rules and are only allowed in `unsafe` code.

### FFI

FFI means Foreign Function Interface. Rust can call C code and expose Rust functions to other languages through ABI boundaries.

This is one of the major use cases for `unsafe`.

### `extern "C"`

This indicates a foreign ABI interface for FFI or interop.

```rust
extern "C" {
    fn puts(s: *const i8);
}
```

### `repr(C)`

`#[repr(C)]` tells Rust to lay out a struct in the same way as C does.

This is important for FFI and binary compatibility.

### `MaybeUninit`

`MaybeUninit<T>` allows data to be uninitialized temporarily before being initialized properly.

This is an advanced but common low-level Rust term.

### `transmute`

`std::mem::transmute` reinterprets bits from one type to another.

It is highly unsafe and should only be used with strong justification.

---

## Concurrency and async

### Threads

Rust has standard threading support.

```rust
use std::thread;

let handle = thread::spawn(|| {
    println!("hello from another thread");
});

handle.join().unwrap();
```

### `Send` and `Sync`

These are marker traits for thread safety.

- `Send`: safe to move to another thread
- `Sync`: safe to share by reference across threads

### `Rc` and `Arc`

- `Rc<T>` — single-threaded reference counting
- `Arc<T>` — thread-safe reference counting

```rust
use std::sync::Arc;
let value = Arc::new(String::from("shared"));
```

### `Mutex` and `RwLock`

These are synchronization primitives.

```rust
use std::sync::Mutex;

let m = Mutex::new(5);
```

They protect shared mutable state.

### `RefCell`

`RefCell<T>` gives interior mutability for single-threaded code.

It enforces borrow rules dynamically rather than statically.

This is a common Rust concept but it is not the normal path for thread-safe design.

### `Cell`

`Cell<T>` allows interior mutability for Copy types by replacing values through shared references.

It is simple but limited, and often used for optimization or internal state patterns.

### `Future` and `await`

A `Future` represents an asynchronous computation.

```rust
async fn hello() -> &'static str {
    "hi"
}
```

`await` waits for the future to complete.

```rust
async fn main() {
    let value = hello().await;
    println!("{}", value);
}
```

### Async runtimes

Async Rust needs a runtime such as:

- Tokio
- async-std
- smol

These schedule and run futures.

### Pinning

Pinning ensures a value cannot be moved while it is being polled.

This matters for async state machines and self-referential data.

### Polling model

Async Rust is based on a polling model. A future is polled repeatedly until it is ready.

This is a key conceptual phrase in async Rust.

### Waker

A `Waker` is the object used to notify a task or future that it may be ready to continue polling.

This is part of the low-level futures machinery.

### Executor

An executor runs asynchronous tasks by polling futures until they complete.

### Task

A task is a scheduled unit of asynchronous work, often managed by an async runtime.

### Channels

Rust channels let threads or async tasks send messages between each other.

Typical ones include:

- `std::sync::mpsc` — multiple producer, single consumer
- `tokio::sync::mpsc` — async channel variants

---

## Rust idioms and conventions

These are the patterns you see all the time in real Rust code.

### Prefer explicit ownership and borrowing

Rust code usually makes ownership rules clear rather than hiding them behind magic.

### Prefer `match` over nested `if`

When dealing with enums and multiple states, `match` is idiomatic.

### Prefer iterators over manual indexing

Rust often favors iterator pipelines for readability and safety.

### Prefer `Result` and `Option` over exceptions and nulls

Rust treats error handling as part of the type system.

### Keep `unsafe` isolated and minimal

Unsafe code should be small, explicit, and well-documented.

### Use `Vec` for dynamic sequences

Use slices for views; use vectors for owning dynamic collections.

### Use trait bounds and generics to express behavior

This is core Rust design philosophy.

### Prefer small focused functions

Rust idioms often value simple, explicit functions over large complex blocks.

### Use `Debug` and `Display` deliberately

- `Debug` for debugging output
- `Display` for user-facing formatting

These are common trait-related jargon terms.

### Embrace expression-oriented code

Rust functions often return values from expressions rather than imperative “procedure-like” blocks.

```rust
fn bigger(a: i32, b: i32) -> i32 {
    if a > b { a } else { b }
}
```

This is a frequent Rust aphorism: “Rust is expression-based.”

### Borrowing and method ergonomics

Rust methods often use receiver types like `&self`, `&mut self`, and `self` to communicate ownership intent clearly.

This is core to Rust’s idiomatic API design.

---

## Common beginner gotchas

### “Why is my value moved?”

Because the type is not `Copy`, and ownership was transferred.

### “Why can’t I use a value after moving it?”

Because ownership is single-owner by design.

### “Why is `String` different from `&str`?”

Because one owns data; the other is a borrowed view.

### “Why do lifetimes appear?”

Because references must be valid for the right period of time.

### “Why do I keep seeing `Option` and `Result`?”

Because Rust encodes absence and failure in the type system.

### “Why is `unsafe` so prominent?”

Because low-level correctness often requires escaping compiler checks intentionally.

### “Why does the compiler complain about borrowing?”

Because multiple references or mutation patterns are invalid under safe aliasing rules.

### “Why do people say Rust is explicit?”

Because the type system is making intent visible: ownership, mutability, error handling, and value semantics are all explicit.

---

## Missing-but-common Rust jargon

These are phrases you often hear in Rust discussions and reviews.

### RAII

Resource Acquisition Is Initialization.

Rust’s ownership + drop system is a practical implementation of RAII: values acquire resources when they are created and release them when dropped.

### Monomorphization

Generics are compiled into specialized code for each concrete type used.

This is part of why idiomatic Rust can be fast while still staying generic.

### Zero-cost abstraction

Rust aims to provide high-level abstractions without runtime overhead.

This is a major cultural value in the language.

### Borrowing is not ownership

A borrow is temporary access. It does not transfer responsibility.

This distinction is fundamental.

### `mut` is not just “variable is mutable”

It communicates a safety contract. Mutation is explicit and often restricted through borrowing rules.

### `where` clauses

A readability and expressiveness tool for constraints on generics.

```rust
fn foo<T>(value: T)
where
    T: Clone,
{
    let _ = value.clone();
}
```

### `Self`

`Self` refers to the type implementing a trait or method.

### `&self` vs `self` vs `&mut self`

These are core method syntax terms:

- `&self` — borrow immutably
- `&mut self` — borrow mutably
- `self` — take ownership of the receiver

### method syntax and receiver types

Rust method calls are not just sugar; they encode ownership and borrowing semantics.

### `#[must_use]`

This attribute warns when the result of a function should not be ignored.

### `#[non_exhaustive]`

This means external code cannot match exhaustively against a type or enum outside the crate.

### `#[derive(...)]`

A common macro pattern used to auto-implement traits like `Debug`, `Clone`, and `PartialEq`.

### `std::mem::take`

A common utility that replaces a value with a default, often used in place mutation.

### `std::mem::replace`

Like take, but explicit replacement of a value with another value.

### `as_ref` / `as_mut`

Used for converting owned or borrowed values into borrowed forms without consuming them.

### `match` ergonomics and patterns

Rust’s strength comes from pattern matching being first-class: `if let`, `match`, `while let`, destructuring, and `let else` all work together.

### `higher-ranked trait bounds` (HRTB)

HRTBs are more advanced trait-bound syntax used when a generic type needs to work with references that may live for different lifetimes.

This is a term you encounter in advanced Rust trait work.

### `opaque type`

A type whose concrete implementation is hidden behind a trait or `impl Trait`.

### `coercion`

Rust can coerce values in a few cases, such as unsizing or trait object conversion. This is a subtle but important part of the type system.

### `shadowing`

A variable can be redefined with the same name in a new scope.

```rust
let x = 5;
let x = x + 1;
```

This is common and idiomatic in Rust.

---

## Advanced Rust jargon and concepts

This section is for the next layer beyond beginner Rust.

### `Fn`, `FnMut`, and `FnOnce`

These are the closure traits:

- `FnOnce` — callable once
- `FnMut` — callable repeatedly with mutable capture
- `Fn` — callable repeatedly with immutable capture

These are fundamental when using closures as arguments.

```rust
fn apply<F: FnOnce()>(f: F) {
    f();
}
```

### `Iterator`, `IntoIterator`, and `FromIterator`

These traits form the ecosystem of collection iteration and collection creation.

- `Iterator` produces values one at a time
- `IntoIterator` means a type can be iterated by value
- `FromIterator` creates a collection from an iterator

### `Deref` and `DerefMut`

These enable smart pointer-like behavior by allowing implicit dereferencing.

```rust
use std::ops::Deref;
```

Very common in pointer wrappers and string-like types.

### `Borrow`, `ToOwned`

These traits support generic conversion between borrowed and owned forms.

### `TryFrom` and `TryInto`

Like `From` and `Into`, but fallible.

```rust
let x: Result<i32, _> = "42".parse();
```

### `trait object safety`

A trait is object-safe if it can be used as a `dyn Trait` trait object. This requires methods not returning `Self` and no generic methods.

### `blanket impl`

A generic implementation that covers many possible types.

This is a standard pattern in trait-based APIs.

### `autoderef` and autoref

Rust automatically dereferences and references values in method calls and overloaded operator resolution.

This is what makes method syntax feel ergonomic.

### `const fn`

A `const fn` can be called at compile time.

```rust
const fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

### `const generics`

Const generics allow generic parameters that are integer or const values rather than types.

```rust
fn repeat<const N: usize>(value: i32) -> [i32; N] {
    [value; N]
}
```

### `specialization`

An advanced trait concept whose stable support is limited in Rust. In practice, it is often avoided in stable Rust due to complexity.

### `type alias`

A type alias gives another name to an existing type.

```rust
type UserId = u64;
```

### `associated const`

A constant associated with a trait or type.

```rust
trait Foo {
    const BAR: i32;
}
```

### `dyn` and object safety

`dyn Trait` is not just for dynamic dispatch; it also interacts with object safety rules.

### `lifetime subtyping`

This is a deeper lifetime concept that describes one lifetime outliving another in a structured way.

It is advanced and often shows up in trait bounds and complex APIs.

### `high-ranked trait bounds` (HRTB)

When a function needs to work with values that borrow for arbitrary lifetime parameters.

This is a common phrase in advanced generic language design.

---

## Advanced unsafe / systems jargon

### `core` and `alloc`

- `core` — foundational Rust library with no OS dependency
- `alloc` — heap allocation support
- `std` — standard library, includes OS interaction and more features

These are important when discussing embedded or no-std contexts.

### `no_std`

A Rust project may avoid the standard library entirely for embedded or microcontroller use.

### `allocator`

A runtime component that allocates memory for a program. In Rust this is often abstracted and configurable.

### `strict provenance`

A modern memory safety concept around pointer provenance and aliasing. This is a deeper systems language topic.

### `ptr::read`, `ptr::write`, and `ptr::copy_nonoverlapping`

Low-level pointer functions used for memory manipulation and optimized unsafe code.

### `slice::from_raw_parts`

Creates a slice from raw memory without safety checks. This is an advanced unsafe pattern used in FFI and system code.

### `repr(transparent)`

A representation attribute ensuring a struct has the same memory layout as its single field.

Useful in FFI and wrappers around primitive types.

### `repr(C)`

Already covered, but this is one of the most important low-level Rust concepts.

### `MaybeUninit`

A type designed to manage uninitialized memory safely.

---

## Async and runtime jargon

### `poll`

The operation used to advance a future toward completion.

### `waker`

Provides a way to notify a future that it should be polled again.

### `executor`

Runs tasks and polls futures.

### `spawn`

Schedules a future to run in a runtime-managed task.

### `runtime`

Defines the environment in which async tasks execute.

### `blocking`

When a task blocks the runtime thread instead of yielding.

This matters for performance and runtime correctness in async systems.

### `mpsc`

Multiple-producer, single-consumer channels.

### `oneshot`

A channel used for one-off message passing.

### `broadcast`

A channel where many consumers receive the same value.

---

## Macro and compiler jargon

### `macro_rules!`

The declarative macro system in Rust.

```rust
macro_rules! say_hello {
    () => {
        println!("hello");
    };
}
```

### Procedural macro

A macro implemented in Rust code that manipulates the abstract syntax tree.

Common forms:

- derive macros
- attribute macros
- function-like macros

### `quote!`

A macro helper library for generating Rust syntax in proc macros.

### `span`

Source code span metadata used in diagnostic and macro systems.

### `hygiene`

Macro hygiene is about preventing name conflicts and preserving semantics across macro expansion boundaries.

This is a very real and important macro concept in Rust.

---

## Ecosystem and project jargon

### crate

A crate is Rust’s unit of compilation. A binary is a crate; a library is also a crate.

### module

A module organizes Rust code into namespaces.

### `Cargo.toml`

The manifest file for Rust packages, containing metadata and dependencies.

### workspace

A Cargo workspace allows multiple crates to be built and managed together.

### edition

Rust edition defines language-version compatibility conventions.

Examples:

- 2018
- 2021
- 2024

### dependency resolver

Cargo resolves versions and feature compatibility across the crate graph.

### feature flag

A crate can expose optional capability sets via feature flags.

```toml
[features]
default = ["std"]
std = []
```

### semver

Semantic versioning controls compatibility rules for crates.

---

## Common Rust phrases in code reviews

These are phrases you will hear a lot when people talk about Rust code.

- “This is a lifetime problem.”
- “This value is moved here; you need a borrow or clone.”
- “This should be `&str`, not `String`.”
- “This could be simplified with `match`.”
- “This should probably be `Result`, not `unwrap`.”
- “This is a `Send`/`Sync` concern.”
- “This should be behind a small `unsafe` boundary.”
- “This API is too eager to clone; consider borrowing.”
- “This function should probably return `Option`.”
- “This type should implement `Default` or `From`.”
- “This is a borrow-checker complaint, not a logic bug.”
- “This is a trait bound issue.”
- “This API is not object-safe.”
- “This is a higher-ranked trait bound problem.”

These phrases are part of the practical vocabulary of Rust discourse.

---

## Quick cheat-sheet table

| Term | Meaning |
| --- | --- |
| Ownership | Single owner for a value |
| Borrowing | Temporary access without transfer |
| Lifetime | How long a reference is valid |
| Move | Ownership transfer |
| Copy | Cheap duplication |
| Clone | Explicit duplication |
| Drop | Cleanup on scope end |
| Trait | Shared behavior contract |
| Generic | Type or function parameterized by type |
| `Option<T>` | Maybe a value |
| `Result<T, E>` | Success or failure |
| `unsafe` | Opt out of compiler safety checks |
| `Send` | Safe to move across threads |
| `Sync` | Safe to share across threads |
| `Future` | Async computation |
| `await` | Wait for async completion |
| `trait object` | Runtime-dispatched type |
| `macro` | Generates code |
| `RAII` | Resource release on scope end |
| `monomorphization` | Generic specialization at compile time |
| `zero-cost abstraction` | High-level code without overhead |

---

## Glossary cheat sheet

- `crate` — a Rust package or compilation unit
- `module` — a namespace for code organization
- `trait` — shared behavior contract
- `impl` — implementation block
- `derive` — auto-generate trait implementations
- `macro` — code-generating code
- `borrow checker` — compiler logic enforcing borrowing rules
- `lifetime` — validity period of a reference
- `drop` — cleanup at end of scope
- `Copy` — duplicated bit-for-bit
- `Clone` — explicit duplication
- `Send` — safe to move to another thread
- `Sync` — safe to share across threads
- `Future` — asynchronous computation
- `async` — syntax for futures
- `await` — wait for future completion
- `unreachable!` — assert that a path cannot happen
- `panic!` — unrecoverable failure
- `unsafe` — bypass compiler safety checks with responsibility
- `RAII` — resource acquisition is initialization
- `zerocost abstraction` — no runtime overhead from abstraction
- `trait object` — runtime dynamic dispatch via `dyn Trait`
- `monomorphization` — generic specialization at compile time

---

## Beginner roadmap

### Stage 1: Learn the core rules

Focus on:

- ownership
- borrowing
- moves and copies
- `String` vs `&str`
- `Option` and `Result`

### Stage 2: Learn the syntax of real Rust

Focus on:

- structs and enums
- `match`
- `if let`
- `loop` and `for`
- methods and `impl`

### Stage 3: Learn idiomatic patterns

Focus on:

- iterators
- closures
- trait implementations
- generic functions
- error propagation via `?`

### Stage 4: Learn the ecosystem

Explore:

- `serde`
- `tokio`
- `reqwest`
- `clap`
- `thiserror`
- `anyhow`

### Stage 5: Learn advanced Rust

Then move into:

- lifetimes in APIs
- async and runtime management
- macros
- unsafe and FFI
- custom trait design

---

## Learning resources

Official:

- Rust Book: https://doc.rust-lang.org/book/
- Rust By Example: https://doc.rust-lang.org/stable/rust-by-example/
- Rust Reference: https://doc.rust-lang.org/reference/
- Rustonomicon: https://doc.rust-lang.org/nomicon/

Practice:

- Rustlings: https://github.com/rust-lang/rustlings
- Exercism Rust track: https://exercism.org/tracks/rust
- Too Many Linked Lists: https://rust-unofficial.github.io/too-many-lists/

Community:

- This Week in Rust: https://this-week-in-rust.org/
- Rust subreddit: https://www.reddit.com/r/rust/
- Users forum: https://users.rust-lang.org/

---

## Final note

Rust is often described as difficult, but the challenge is usually not syntax. The real challenge is learning the language’s mental model.

The jargon exists because each term is a shortcut to a deep idea:

- ownership for safety
- borrowing for access control
- lifetimes for validity
- traits for behavior
- `Option` and `Result` for explicit data flow
- `unsafe` for deliberate low-level power

Once these concepts click, the language becomes much more approachable.

This repository aims to make that shift easier by organizing the vocabulary in a beginner-friendly way.

---

## Contributing

This catalog can be expanded with:

- more jargon entries
- deeper examples
- beginner-friendly diagrams or “mental models” notes
- “common confusion” sections
- links to high-quality learning resources
- real-world Rust code snippets

This is meant to be a living reference for learners.
