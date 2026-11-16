---
title: "Rust from Zero to Expert"
subtitle: "A complete course in Rust (2024 edition) — from ownership to a crash-tested, persistent network server"
lang: en
---

# Preface

Rust is a systems programming language with the performance of C and C++ and a compiler that rules out, before the program runs, the memory-safety bugs and data races that cause most security vulnerabilities and many crashes in systems software. It is used in operating-system kernels (Linux and Windows), browsers, databases, cloud infrastructure, command-line tools, embedded devices, and WebAssembly.

Rust's reputation for difficulty comes almost entirely from one idea: **ownership** — who owns each value, who may look at it, and who may change it. This book puts that idea at the centre, explains the rules precisely, and shows what the compiler does with them. After Chapter 2, the rest of the language is built on what you already know.

The book begins with the toolchain and how Rust compiles, then covers core syntax and types, ownership and borrowing, structs and enums, error handling, traits and generics, collections and iterators, crates and testing, smart pointers, threads, async Rust with Tokio, unsafe code and FFI, macros, and performance. It ends with a capstone: a networked key-value store with a write-ahead log, group commit, snapshots, and crash recovery, tested by killing it in the middle of writes.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the language's guarantees, the standard library's documented behaviour, Cargo, the compiler, the operating system, or established practice.
2. **Evidence.** Every program in this book was compiled and run with Rust 1.97, and every output shown is real — including compiler errors, which are part of how Rust teaches. When an example's first version was wrong, or an expected speed-up did not happen (several did not), the book shows that too.
3. **Order.** Nothing is used before it is taught, and later chapters refer back by unit number.

This book is the third in a series with *C from Zero to Systems* and *C++ from Zero to Expert*. You do not need them — no previous language is assumed — but where Rust's choice is easiest to understand by contrast (move semantics, undefined behaviour, data races, iterator invalidation), the book shows the C or C++ program that compiles and fails, and the Rust program that does not compile.

# How to use this book

## Who it is for

- **Beginners** who are willing to work carefully: start at Chapter 0 and read in order.
- **Programmers from garbage-collected languages** (Python, Java, Go, JavaScript): read Chapter 1 quickly; spend real time on Chapters 2, 5, and 8.
- **C and C++ programmers:** Chapter 2 will feel both familiar and strict; the *Deep & Tricky* entries compare Rust's rules with C's and C++'s throughout. Chapters 9, 11, and 13 are where your experience pays off most.
- **Working Rust programmers:** use Appendix A to test yourself, then focus on Chapters 10–14.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | The rules, with small complete programs and their real output — including compiler errors. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, each labelled with its source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions: decide the answer before running the code. Answers are at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a specification, a complete reference solution with real output, and a review checklist. Try the project yourself first.

## How to study

1. **Type the programs and run them.** Compare your output with the book's.
2. **Read every compiler error to the end.** Rust's errors usually explain the rule you broke and suggest the fix; `rustc --explain E0502` gives the full story.
3. **Predict before you run,** especially whether code compiles. When you are wrong, find out why.
4. **Break things on purpose:** remove a `move`, add a second `&mut`, drop a value too early — and read what the compiler says.
5. **Use the tools from day one:** `cargo check`, `cargo clippy`, `cargo fmt`, `cargo test` (Chapter 0).
6. **Revise with Appendix A,** which lists every DT and P entry by title.

## How to read the labels

| Label | Meaning |
| --- | --- |
| *Guarantee: Rust language* | What the language guarantees (the Rust Reference); holds for every conforming compiler and future versions within an edition. |
| *Guarantee: Rust standard library* | Documented, stable behaviour of `std`. |
| *Library behaviour* | How a library (`std`, Tokio) behaves where its documentation describes the behaviour but it is not a language rule. |
| *Compiler behaviour*, *Toolchain behaviour*, *Cargo behaviour* | What rustc, the linker, or Cargo do. Stable in practice; may change between versions. |
| *OS behaviour*, *Runtime behaviour* | Properties of the operating system or of an async runtime. |
| *Design fact*, *Practice* | Engineering consequences and professional practice — experience, not language rules. |

## Toolchain and versions

All code was compiled and run on **Linux x86-64 (Ubuntu 24.04)** with:

- **Rust 1.97** (stable, July 2026), **edition 2024**, with Cargo, rustfmt, and Clippy;
- crates from crates.io, at the versions recorded in each project's `Cargo.lock` (Tokio 1, Rayon 1, Criterion 0.5, syn 2, anyhow 1, thiserror 2, memmap2 0.9, rustc-hash 2, crc32fast 1);
- **GCC 13** for the C side of FFI, **Valgrind 3.22**, **strace**, and **perf** (software clock sampling; hardware counters were not available in the virtual machine), and the system **zlib** (zlib-ng 1.3.1 in compatibility mode).

Two tools could not be used and the text says so where they matter: **Miri**, the undefined-behaviour interpreter, and Rust's sanitizers both require a nightly toolchain, which could not be installed in the build environment. Chapter 11 uses Valgrind instead and shows how to run Miri.

Measurements (Chapters 5, 6, 7, 9, 10, 13, 14) were taken on a 2-core virtual machine. Your numbers will differ; compare ratios and shapes, and expect run-to-run variation, which the text reports where it is large.

## Setting up

1. Install Rust with **rustup** from rust-lang.org: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` on Linux and macOS, or the installer on Windows. It installs `rustc`, `cargo`, `rustfmt`, and `clippy`, and keeps them updated (`rustup update`).
2. Check: `rustc --version`, `cargo --version`. For Chapter 11's Miri: `rustup toolchain install nightly --component miri`.
3. Use an editor with **rust-analyzer** (VS Code, RustRover, Neovim, Emacs, Zed, Helix). It shows types, errors, and inlay hints as you type, and runs Clippy on save.
4. On Windows, everything except the Unix-specific parts of Chapters 11 and 13 (Valgrind, `perf`, `strace`) works natively; use WSL2 for those.

# Roadmap

| Chapter | Title | You will be able to… |
| --- | --- | --- |
| 0 | The Rust Toolchain and How Rust Compiles | build with rustc and Cargo; explain crates, monomorphisation, and profiles; use fmt, clippy, test, doc |
| 1 | Core Syntax and Types | use variables, scalar and compound types, strings, functions, control flow, formatting, and input |
| 2 | Ownership, Borrowing, and Lifetimes | predict moves, copies, and drops; borrow within the rules; write lifetime annotations |
| 3 | Structs, Enums, and Pattern Matching | model data so invalid states are unrepresentable; match exhaustively |
| 4 | Error Handling | separate bugs (panic) from failures (`Result`); design error types; use `?`, anyhow, thiserror |
| 5 | Traits and Generics | write generic code with trait bounds; use trait objects; implement standard traits |
| 6 | Collections, Closures, and Iterators | use the standard collections; write closures; build zero-cost iterator pipelines |
| 7 | Modules, Crates, Workspaces, Testing, Docs | organise code and crates; manage dependencies and features; test and document |
| 8 | Smart Pointers and Interior Mutability | use `Box`, `Rc`, `Weak`, `Cell`, `RefCell`, `Cow`; choose arenas over pointer graphs |
| 9 | Concurrency | use threads, `Send`/`Sync`, channels, locks, atomics, and Rayon; design for independence |
| 10 | Async Rust | explain futures and executors; write Tokio servers; handle timeouts and cancellation |
| 11 | Unsafe Rust, FFI, and the Memory Model | write sound unsafe code; call C and be called from C; wrap a C library safely |
| 12 | Macros | write declarative and procedural macros with good errors |
| 13 | Performance and Systems Programming | benchmark, profile, remove allocations, read assembly; do efficient I/O and process control |
| 14 | Capstone: A Persistent Key-Value Store | design, build, crash-test, and load-test a durable network server |
| A | Index of Deep & Tricky entries and Patterns | revise every entry by title |
| B | Glossary | look up any term |
| C | Editions, versions, and toolchain | see what each edition changed and which features need which Rust version |

Chapters 0–6 are the **foundation**; read them in order. Chapter 7 can be read at any point after Chapter 4. Chapters 8–13 can be read in any order after Chapter 7, except that Chapter 10 builds on Chapter 9. Chapter 14 uses the whole book.

# Chapter 0 — The Rust Toolchain and How Rust Compiles

*Goal: install and use the Rust toolchain; build programs with `rustc` and Cargo; understand what the compiler does with your code — crates, monomorphisation, LLVM, debug and release profiles — and read its error messages, which are the best teacher you will have.*

Rust is a systems programming language that gives the control and performance of C and C++ while guaranteeing, at compile time, that safe code has no memory-safety bugs and no data races. It does this with a set of rules about **ownership** and **borrowing** that the compiler checks. Learning Rust is largely learning those rules — and the compiler explains them every time you break one.

**What you will be able to do after this chapter**

- Compile and run Rust programs with `rustc` and with Cargo.
- Create a Cargo package, add dependencies, and use the dev and release profiles.
- Describe the path from source to executable: crates, MIR, borrow checking, monomorphisation, LLVM, and linking.
- Read a compiler error, and look up any error code with `rustc --explain`.
- Use `rustfmt`, Clippy, `cargo doc`, and `cargo test` from the start.

## Topic 0A — Rust and its tools

## Unit 0.1 — What Rust is

**One-line summary:** Rust is a compiled, statically typed language with no garbage collector, whose compiler enforces **ownership** (each value has one owner, freed when the owner goes out of scope) and **borrowing** (references must not outlive what they refer to, and you can have either many readers or one writer). Code that follows the rules is called **safe Rust**; code that needs to break them is marked `unsafe`.

The design goals, and how Rust meets them:

- **Performance like C and C++.** Rust compiles to machine code through LLVM (the same back end as Clang), has no runtime or garbage collector, and its abstractions — iterators, closures, generics, traits — compile to the same code a careful C programmer would write.
- **Memory safety without a garbage collector.** Use-after-free, double free, dangling references, buffer overflows, and iterator invalidation — the bug classes of the C and C++ books' Chapters 12 and 13 — are compile-time errors in safe Rust, or (for array indexing) checked at run time.
- **Fearless concurrency.** The same ownership rules make data races compile-time errors (Chapter 9).
- **Explicit errors.** No exceptions: functions that can fail return `Result`, and the type system makes you handle it (Chapter 4).
- **Modern tooling as part of the language.** One official build tool and package manager (Cargo), formatter (rustfmt), linter (Clippy), documentation generator (rustdoc), and test runner, used by the whole ecosystem.

### Editions and versions

A new stable Rust compiler is released every six weeks; this book uses **Rust 1.97**. Language changes that would break old code are grouped into **editions** — 2015, 2018, 2021, and **2024** (the edition used in this book). Each crate chooses its edition, and crates of different editions link together freely, so the ecosystem never splits.

### Deep & Tricky

**DT-1. Safe Rust guarantees no undefined behaviour — but the guarantee is only as strong as the `unsafe` code underneath it.** *(Guarantee: Rust language; soundness)* The standard library, and many crates, use `unsafe` internally (a `Vec` manages raw memory). Safe Rust's guarantee is that *if* every `unsafe` block upholds its documented contract, no safe code can cause undefined behaviour. A bug in an `unsafe` block — or, rarely, a compiler bug — can break it. That is why `unsafe` is kept small, documented, and reviewed carefully (Chapter 11). What safe Rust does *not* prevent: logic errors, panics, deadlocks, memory leaks (leaking is safe), and integer overflow (which is defined, not UB — DT-2).

## Unit 0.2 — Your first programs with `rustc`

**One-line summary:** `rustc` compiles one **crate** — a program or library — from its root source file. `fn main()` is the entry point; `println!` is a **macro** (the `!` marks it) that checks its format string at compile time.

```rust
// file: hello.rs
fn main() {
    let name = "Rust";
    let year = 2015;
    println!("Hello from {name}, stable since {year}.");
    let squares: Vec<u64> = (1..=5).map(|n| n * n).collect();
    println!("squares: {squares:?}, sum = {}", squares.iter().sum::<u64>());
}
```

```console
$ rustc --edition 2024 hello.rs && ./hello
Hello from Rust, stable since 2015.
squares: [1, 4, 9, 16, 25], sum = 55
$ ls -l hello | awk '{print $5, $9}'
4380392 hello
$ rustc --edition 2024 -O -C strip=symbols hello.rs -o hello_small && ls -l hello_small | awk '{print $5, $9}'
343536 hello_small
```

Two programs in one line each: a formatted print with variables captured by name, and an iterator pipeline that maps a range to squares and collects them into a vector. The binary is statically linked with the Rust standard library (but dynamically with the C library), and most of its size is debug information and symbols; an optimised, stripped build is much smaller.

The compiler's error messages are part of the language's design. Here is the first rule of ownership being broken:

```rust
// file: moved.rs
fn main() {
    let v = vec![1, 2, 3];
    let w = v;                     // ownership of the vector moves to w
    println!("{v:?} {w:?}");       // v can no longer be used
}
```

```console
$ rustc --edition 2024 moved.rs
error[E0382]: borrow of moved value: `v`
 --> moved.rs:5:16
  |
3 |     let v = vec![1, 2, 3];
  |         - move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait
4 |     let w = v;                     // ownership of the vector moves to w
  |             - value moved here
5 |     println!("{v:?} {w:?}");       // v can no longer be used
  |                ^ value borrowed here after move
  |
help: consider cloning the value if the performance cost is acceptable
  |
4 |     let w = v.clone();                     // ownership of the vector moves to w
  |              ++++++++

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0382`.
$ rustc --explain E0382 | head -n 3
A variable was used after its contents have been moved elsewhere.

Erroneous code example:
```

The message shows where the value was created, where it moved, and where it was used after the move; explains *why* (`Vec<i32>` does not implement `Copy`); and suggests a fix. Every error has a code, and `rustc --explain` prints a full explanation with examples. Read compiler errors fully — in Rust they are usually right about both the problem and the fix.

### Mastery check

1. **Predict.** If `let v = vec![1, 2, 3];` is changed to `let v = [1, 2, 3];` (an array of `i32`, which is `Copy`), does `moved.rs` compile? What does it print?

## Unit 0.3 — Cargo

**One-line summary:** **Cargo** creates packages, fetches dependencies from **crates.io**, builds, runs, tests, benchmarks, and documents them. A **package** is described by `Cargo.toml`; `Cargo.lock` records the exact dependency versions used. Everyday commands: `cargo new`, `cargo build`, `cargo run`, `cargo check` (type-check without generating code: the fastest feedback), `cargo test`, `cargo doc`, `cargo add`.

```console
$ rm -rf hello_cargo && cargo new -q hello_cargo && cd hello_cargo && find . -not -path './.git*' | sort && cat Cargo.toml && cat src/main.rs
.
./Cargo.toml
./src
./src/main.rs
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
fn main() {
    println!("Hello, world!");
}
$ cd hello_cargo && cargo run -q
Hello, world!
$ cd hello_cargo && cargo build 2>&1 | sed -E 's/in [0-9.]+m?s/in <time>/' && cargo build --release 2>&1 | sed -E 's/in [0-9.]+m?s/in <time>/' && ls target
    Finished `dev` profile [unoptimized + debuginfo] target(s) in <time>
   Compiling hello_cargo v0.1.0 (hello_cargo)
    Finished `release` profile [optimized] target(s) in <time>
CACHEDIR.TAG  debug  release
```

`cargo new` creates a package with a Git repository, a manifest, and `src/main.rs`. (The first `cargo build` had nothing to compile: `cargo run` had already built the debug binary.) Build outputs go to `target/debug` (fast to compile, unoptimised, with debug assertions) or `target/release` (optimised). The `dev` and `release` **profiles** control the compiler settings and can be customised in `Cargo.toml`:

```toml
[profile.release]
lto = "fat"            # link-time optimisation across all crates
codegen-units = 1      # fewer parallel units: slower build, faster code
panic = "abort"        # no unwinding: smaller binaries
```

### Deep & Tricky

**DT-2. Integer overflow panics in debug builds and wraps in release builds — and neither is undefined behaviour.** *(Guarantee: Rust language; Compiler behaviour: `overflow-checks`)* Unlike C and C++, where signed overflow is UB, Rust defines overflow as an error that is *checked* when `debug-assertions` are on (the default for the dev profile) and *wraps* in two's complement otherwise. Code that needs a particular behaviour says so: `wrapping_add`, `checked_add` (returns `Option`), `saturating_add`, `overflowing_add`, or the `Wrapping<T>` type. You can also keep checks in release builds with `overflow-checks = true`.

```rust
// file: overflow.rs
fn main() {
    let x: u8 = std::env::args().count() as u8 + 254;    // 255 when run without arguments
    println!("checked: {:?}, wrapping: {}, saturating: {}", x.checked_add(1), x.wrapping_add(1), x.saturating_add(1));
    let y = x + 1;                                          // overflow!
    println!("x + 1 = {y}");
}
```

```console
$ rustc --edition 2024 overflow.rs -o overflow_debug && ./overflow_debug
checked: None, wrapping: 0, saturating: 255

thread 'main' (5788) panicked at overflow.rs:5:13:
attempt to add with overflow
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
$ rustc --edition 2024 -O overflow.rs -o overflow_release && ./overflow_release
checked: None, wrapping: 0, saturating: 255
x + 1 = 0
```

## Unit 0.4 — How Rust compiles

**One-line summary:** The unit of compilation is the **crate**, not the file. `rustc` parses the crate, expands macros, resolves names, type-checks, lowers to **MIR** (mid-level IR) where the **borrow checker** runs, then **monomorphises** generic code — generating a separate copy for each concrete type used — and hands **LLVM IR** to LLVM, which optimises and generates machine code in parallel **codegen units**. Finally the system linker produces the executable.

| Stage | What happens | Errors you see here |
|---|---|---|
| Parsing and macro expansion | text → syntax tree; `println!`, `vec!`, derives expanded | syntax errors, macro errors |
| Name resolution, type checking, trait solving | every expression gets a type | `E0308` mismatched types, `E0277` trait not implemented |
| MIR building and **borrow checking** | ownership, borrowing, lifetimes checked on a control-flow graph | `E0382` use after move, `E0499`/`E0502` borrow conflicts, `E0597` doesn't live long enough |
| Monomorphisation | one copy of each generic function per set of type arguments | (errors after this point are rare) |
| LLVM optimisation and code generation | inlining, vectorisation, machine code | — |
| Linking | the system linker (`cc`) combines objects and libraries | undefined symbols in FFI code |

You can look at the intermediate forms. Here is monomorphisation made visible — one generic function, used with two types, becomes two functions in the binary:

```rust
// file: mono.rs
fn largest<T: PartialOrd + Copy>(items: &[T]) -> T {
    let mut best = items[0];
    for &x in items {
        if x > best {
            best = x;
        }
    }
    best
}

fn main() {
    println!("{} {}", largest(&[3, 9, 2]), largest(&[1.5, -2.0]));
}
```

```console
$ rustc --edition 2024 -C opt-level=0 mono.rs && ./mono && nm -C mono | grep -o 'mono::largest.*' | sort
9 1.5
mono::largest::<f64>
mono::largest::<i32>
```

Two instances, `largest::<i32>` and `largest::<f64>`, each compiled and optimised for its type: this is why generics in Rust (like templates in C++) cost nothing at run time — and why heavy generic code increases compile times and binary size.

### Deep & Tricky

**DT-3. The crate, not the file, is the unit of compilation — so a large crate rebuilds as a whole.** *(Compiler behaviour)* Changing one file recompiles its whole crate (incremental compilation reuses unchanged parts of the analysis, and codegen units allow parallel code generation, but the crate is still the unit). Large Rust projects are therefore split into several crates in a **workspace** (Chapter 7), so that a change rebuilds only the crates that depend on it — the Rust equivalent of keeping C++ headers small.

## Unit 0.5 — Formatting, linting, testing, and documentation

**One-line summary:** `cargo fmt` formats code to the community style (so nobody argues about it); `cargo clippy` adds hundreds of lints for bugs, performance, and idiomatic style; `cargo test` runs unit tests, integration tests, *and* the examples in documentation comments; `cargo doc` builds HTML documentation.

```rust
// file: tools_demo/src/lib.rs
/// Returns the mean of the values, or `None` for an empty slice.
///
/// ```
/// assert_eq!(tools_demo::mean(&[1.0, 2.0, 6.0]), Some(3.0));
/// assert_eq!(tools_demo::mean(&[]), None);
/// ```
pub fn mean(values: &[f64]) -> Option<f64> {
    if values.len() == 0 {
        return None;
    }
    let mut total = 0.0;
    for i in 0..values.len() {
        total = total + values[i];
    }
    Some(total / values.len() as f64)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn mean_of_one_value_is_that_value() {
        assert_eq!(mean(&[4.5]), Some(4.5));
    }
}
```

```toml
# file: tools_demo/Cargo.toml
[package]
name = "tools_demo"
version = "0.1.0"
edition = "2024"
```

```console
$ cd tools_demo && cargo test -q 2>&1 | grep -E 'test result|running'
running 1 test
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
running 1 test
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
$ cd tools_demo && cargo clippy -q 2>&1 | grep -E '^(warning|error)|^ +--> '
warning: length comparison to zero
 --> src/lib.rs:9:8
warning: the loop variable `i` is only used to index `values`
  --> src/lib.rs:13:14
warning: manual implementation of an assign operation
  --> src/lib.rs:14:9
```

Both kinds of test ran: the unit test in the `tests` module, and the example in the documentation comment (a **doctest** — documentation examples that are compiled and run can never go out of date). Clippy, meanwhile, found three things to improve in eleven lines of working code: `len() == 0` should be `is_empty()`; the index loop should iterate over the slice directly; and `total = total + x` should be `total += x`. Rewritten the way Clippy suggests, the function body becomes `if values.is_empty() { return None; } Some(values.iter().sum::<f64>() / values.len() as f64)`.

### Pattern

**P-1. Make `cargo fmt --check`, `cargo clippy -- -D warnings`, and `cargo test` the gate for every change.** *(Practice)* All three are fast and run the same way on every machine. Running them in CI (and in an editor through rust-analyzer) keeps a codebase consistently formatted, lint-clean, and tested from the first commit, at almost no cost. *Trade-off:* occasionally a Clippy lint is wrong for your case; silence it locally with `#[allow(clippy::lint_name)]` and a comment saying why, rather than turning it off globally.

## Chapter 0 Project — Inspecting a Cargo build

**Goal:** create a package with a library and a binary; add a dependency from crates.io; and inspect what Cargo and the compiler produce — the dependency tree, the lock file, debug and release performance, and binary sizes.

The library counts primes below a limit with a sieve; the binary parses its argument with the `clap`-free standard library and times the work. A dependency, `itoa` (fast integer formatting), gives Cargo something to fetch.

```toml
# file: primes/Cargo.toml
[package]
name = "primes"
version = "0.1.0"
edition = "2024"

[dependencies]
itoa = "1"
```

```rust
// file: primes/src/lib.rs
/// Counts the primes strictly below `limit` with the sieve of Eratosthenes.
pub fn count_primes(limit: usize) -> usize {
    if limit < 3 {
        return 0;
    }
    let mut composite = vec![false; limit];
    let mut count = 0;
    for n in 2..limit {
        if !composite[n] {
            count += 1;
            let mut multiple = n * n;
            while multiple < limit {
                composite[multiple] = true;
                multiple += n;
            }
        }
    }
    count
}

#[cfg(test)]
mod tests {
    use super::count_primes;

    #[test]
    fn known_counts() {
        assert_eq!(count_primes(0), 0);
        assert_eq!(count_primes(10), 4);        // 2 3 5 7
        assert_eq!(count_primes(100), 25);
        assert_eq!(count_primes(1_000_000), 78_498);
    }
}
```

```rust
// file: primes/src/main.rs
use std::time::Instant;

fn main() {
    let limit: usize = std::env::args().nth(1).and_then(|s| s.parse().ok()).unwrap_or(10_000_000);
    let start = Instant::now();
    let n = primes::count_primes(limit);
    let elapsed = start.elapsed();
    let mut buf = itoa::Buffer::new();
    println!("{} primes below {limit} in {elapsed:.1?}", buf.format(n));
}
```

```console
$ cd primes && cargo test -q 2>&1 | grep 'test result' | head -n 1
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.02s
$ cd primes && cargo tree && grep -A2 'name = "itoa"' Cargo.lock
primes v0.1.0 (primes)
└── itoa v1.0.19
name = "itoa"
version = "1.0.19"
source = "registry+https://github.com/rust-lang/crates.io-index"
$ cd primes && cargo build -q && cargo build -q --release && ./target/debug/primes 50000000 && ./target/release/primes 50000000
3001134 primes below 50000000 in 1.3s
3001134 primes below 50000000 in 336.4ms
$ cd primes && ls -l target/debug/primes target/release/primes | awk '{print $5, $9}'
4524256 target/debug/primes
459720 target/release/primes
```

What the build shows:

- **The dependency tree and lock file.** `cargo tree` shows `itoa` as the one dependency, and `Cargo.lock` pins its exact version and checksum, so every build — yours, CI's, a colleague's next year — uses the same code. Commit `Cargo.lock` for applications.
- **Debug versus release.** The same sieve ran about three and a half times faster in the release build. The debug build checks every arithmetic operation for overflow, does not inline, and keeps every value in memory for the debugger. *Never benchmark a debug build* — and expect debug Rust to be slower relative to release than debug C is, because idiomatic Rust leans on inlining of small functions and iterator adaptors.
- **Size.** The debug binary carries full debug information; the release binary is much smaller, and `strip`, `panic = "abort"`, and `opt-level = "z"` shrink it further when size matters.

### Review checklist

- Can you explain what each file in the package is for, and which ones belong in version control (`Cargo.lock` too, for a binary)?
- Run `cargo build` twice: why does the second one do almost nothing? Touch `src/lib.rs` and build again: what is rebuilt, and what is not (the `itoa` dependency)?
- Where do the tests for `count_primes` live, and why do they not end up in the release binary? (`#[cfg(test)]`.)

## Answers to the Chapter 0 prediction questions

**0.2 Q1.** Yes. An array of `i32` is `Copy`, so `let w = v;` copies it and both remain usable; the program prints `[1, 2, 3] [1, 2, 3]`. Moving only invalidates the source for types that are not `Copy` — usually because they own heap memory or another resource.

# Chapter 1 — Core Syntax and Types

*Goal: write small, correct Rust programs — variables and mutability, the scalar and compound types, strings, functions and expressions, control flow, formatted output, and reading input — and know the precise rules where Rust differs from C, C++, and other languages.*

**What you will be able to do after this chapter**

- Declare immutable and mutable variables, constants, and statics, and use shadowing.
- Choose integer and floating-point types; convert between them knowing exactly what `as` does.
- Use tuples, arrays, slices, `String`, and `&str`, and explain why strings cannot be indexed by position.
- Write functions whose bodies are expressions; use `if`, `match` (first look), `loop`, `while`, and `for`.
- Format output with `println!`, `format!`, and `dbg!`, and read and parse input.

## Topic 1A — Values and types

## Unit 1.1 — Variables, mutability, and shadowing

**One-line summary:** `let` bindings are **immutable** by default; `let mut` makes them mutable. A new `let` with the same name **shadows** the old binding (it can even change the type). `const` declares a compile-time constant (inlined at each use); `static` declares one fixed memory location for the whole program.

```rust
// file: vars.rs
const MAX_USERS: u32 = 100_000;          // type required; underscores are digit separators
static GREETING: &str = "hello";         // a single location in the binary

fn main() {
    let x = 5;
    // x = 6;                            // error[E0384]: cannot assign twice to immutable variable
    let mut count = 0;
    count += x;
    let spaces = "   ";                  // a &str ...
    let spaces = spaces.len();           // ... shadowed by a usize: a new variable, same name
    let x = x * 2;                       // shadowing again: x is still immutable
    println!("{GREETING}: count={count}, spaces={spaces}, x={x}, max={MAX_USERS}");

    let shadowed = 1;
    {
        let shadowed = "inner";          // shadows only inside this block
        println!("inner: {shadowed}");
    }
    println!("outer: {shadowed}");
}
```

```console
$ rustc --edition 2024 vars.rs && ./vars
hello: count=5, spaces=3, x=10, max=100000
inner: inner
outer: 1
```

Immutability by default is not just style: the compiler relies on it (an immutable value cannot change behind your back, which matters for borrowing in Chapter 2), and it makes `mut` a visible warning sign that a value changes. Unused `mut` gets a compiler warning.

<!-- expanded -->
### Going deeper: shadowing versus `mut`, and constants

Shadowing (`let x = ...; let x = ...;`) creates a *new* variable that happens to have the same name; the old one still exists until the end of its scope, unreachable by name. Because it is a new variable, it may have a different type — which `mut` cannot do. `const` is different again: a value computed at compile time and copied into every place it is used, with a type that must be written out. `static` is one fixed memory location for the whole program.

```rust
// file: shadow.rs
const MAX_USERS: u32 = 10 * 1000;           // evaluated at compile time, inlined where used
static GREETING: &str = "hello";            // one location, lives for the whole program

fn main() {
    let input = "  42  ";                   // &str
    let input = input.trim();               // still &str, a narrower slice
    let input: u32 = input.parse().unwrap(); // now a u32: a different type, same name
    println!("input + 1 = {}", input + 1);

    let x = 5;
    {
        let x = x * 2;                      // shadows only inside this block
        println!("inner x = {x}");
    }
    println!("outer x = {x}");

    let mut count = 0;
    count += 1;                             // mut: same variable, same type, new value
    // count = "one";                       // error: expected integer, found &str
    println!("count = {count}, MAX_USERS = {MAX_USERS}, {GREETING}");
}
```

```console
$ rustc --edition 2024 shadow.rs && ./shadow
input + 1 = 43
inner x = 10
outer x = 5
count = 1, MAX_USERS = 10000, hello
```

Shadowing is idiomatic for "the same data in a cleaner form" (raw input → trimmed → parsed). It is a trap only in long functions, where a reader may not notice that `x` now means something else.

## Unit 1.2 — Scalar types and conversions

**One-line summary:** Integers have explicit sizes: `i8`–`i128` and `u8`–`u128`, plus `isize`/`usize` (pointer-sized; `usize` is the type of lengths and indices). Floats are `f32` and `f64`; `bool` is `true`/`false`; `char` is a **Unicode scalar value** (4 bytes). There are **no implicit numeric conversions** at all: you convert with `as` (which never fails, but may lose information), or with `From`/`TryFrom` (which are lossless or checked).

```rust
// file: scalars.rs
fn main() {
    let a: i32 = -17;
    let b = 3;                                     // inferred i32 from use below
    println!("{} {} {} {}", a / b, a % b, a.div_euclid(b), a.rem_euclid(b));   // truncating vs Euclidean
    println!("sizes: i8={} i64={} i128={} usize={} char={} bool={}",
             size_of::<i8>(), size_of::<i64>(), size_of::<i128>(), size_of::<usize>(), size_of::<char>(), size_of::<bool>());

    // `as` between integers: truncates or sign-extends, never fails
    println!("300 as u8 = {}, -1i32 as u32 = {}, 200u8 as i8 = {}", 300i32 as u8, -1i32 as u32, 200u8 as i8);
    // `as` from float to integer: rounds toward zero and SATURATES; NaN becomes 0
    println!("2.9 as i32 = {}, -1.5 as u8 = {}, 1e10 as i32 = {}, NaN as i32 = {}", 2.9f64 as i32, -1.5f64 as u8, 1e10f64 as i32, f64::NAN as i32);

    // lossless conversions with From; checked ones with TryFrom
    let small: u8 = 200;
    let wide = u32::from(small);
    let back = u8::try_from(wide + 100);
    println!("u32::from(200u8) = {wide}, u8::try_from(300) = {back:?}");

    let c = 'é';
    let crab = '🦀';
    println!("'{c}' is U+{:04X}, '{crab}' is U+{:X}, as u32 = {}", c as u32, crab as u32, crab as u32);
    println!("0.1 + 0.2 = {}, == 0.3? {}", 0.1 + 0.2, 0.1 + 0.2 == 0.3);
}
```

```console
$ rustc --edition 2024 scalars.rs && ./scalars
-5 -2 -6 1
sizes: i8=1 i64=8 i128=16 usize=8 char=4 bool=1
300 as u8 = 44, -1i32 as u32 = 4294967295, 200u8 as i8 = -56
2.9 as i32 = 2, -1.5 as u8 = 0, 1e10 as i32 = 2147483647, NaN as i32 = 0
u32::from(200u8) = 200, u8::try_from(300) = Err(TryFromIntError(PosOverflow))
'é' is U+00E9, '🦀' is U+1F980, as u32 = 129408
0.1 + 0.2 = 0.30000000000000004, == 0.3? false
```

### Deep & Tricky

**DT-4. `as` from a float to an integer saturates, and NaN becomes 0 — it is never undefined behaviour.** *(Guarantee: Rust language, since Rust 1.45)* In C and C++, converting an out-of-range float to an integer is undefined behaviour. Rust defines it: values round toward zero, then clamp to the target type's range (`1e10 as i32` is `i32::MAX`, `-1.5 as u8` is 0), and NaN converts to 0. Integer-to-integer `as` truncates to the low bits (`300 as u8` is 44). Because `as` silently loses information, prefer `From` where the conversion is lossless and `TryFrom` where it might not be, and reserve `as` for the cases where truncation is what you want.

<!-- expanded -->
### Going deeper: checked, wrapping, saturating, and overflowing arithmetic

Plain `+` on integers panics on overflow in debug builds and wraps in release builds (DT-2). When overflow is *possible and meaningful*, say which behaviour you want with an explicit method — these behave the same in every build.

```rust
// file: arith.rs
fn main() {
    let a: u8 = 250;
    println!("checked_add(10):     {:?}", a.checked_add(10));      // None on overflow
    println!("checked_add(5):      {:?}", a.checked_add(5));
    println!("wrapping_add(10):    {}", a.wrapping_add(10));        // modulo 256
    println!("saturating_add(10):  {}", a.saturating_add(10));      // clamps at 255
    println!("overflowing_add(10): {:?}", a.overflowing_add(10));   // (result, did it overflow)

    let b: i32 = i32::MIN;
    println!("i32::MIN.checked_abs(): {:?}", b.checked_abs());     // |MIN| does not fit
    println!("-7 / 2 = {}, -7 % 2 = {}", -7 / 2, -7 % 2);          // truncates toward zero
    println!("(-7i32).div_euclid(2) = {}, rem_euclid = {}", (-7i32).div_euclid(2), (-7i32).rem_euclid(2));

    let big: u64 = 300;
    println!("300 as u8 = {}, u8::try_from(300) = {:?}", big as u8, u8::try_from(big));
}
```

```console
$ rustc --edition 2024 arith.rs && ./arith
checked_add(10):     None
checked_add(5):      Some(255)
wrapping_add(10):    4
saturating_add(10):  255
overflowing_add(10): (4, true)
i32::MIN.checked_abs(): None
-7 / 2 = -3, -7 % 2 = -1
(-7i32).div_euclid(2) = -4, rem_euclid = 1
300 as u8 = 44, u8::try_from(300) = Err(TryFromIntError(PosOverflow))
```

`as` between integers silently truncates (300 becomes 44); `TryFrom` returns an error instead. For array indices and money, prefer `checked_*` or `try_from`; for hash functions and checksums, `wrapping_*` says the wrap is intended. `rem_euclid` gives a non-negative remainder, which is what you want for "which bucket" or "which day of the week" calculations with negative numbers.

## Unit 1.3 — Tuples, arrays, and slices

**One-line summary:** A **tuple** groups values of different types (`(i32, &str)`), accessed by `.0`, `.1` or destructuring. An **array** `[T; N]` has a fixed length that is part of its type, and lives inline (on the stack, for a local). A **slice** `&[T]` is a view of a contiguous run of elements — a pointer and a length — and is how functions accept arrays and vectors of any length. Indexing is **bounds-checked**: out of range is a panic, never a buffer overflow.

```rust
// file: compound.rs
fn sum(values: &[i64]) -> i64 {          // accepts arrays, vectors, and sub-slices
    let mut total = 0;
    for v in values {
        total += v;
    }
    total
}

fn main() {
    let pair: (i32, &str) = (7, "seven");
    let (n, name) = pair;                                  // destructuring
    println!("{} {} / {n} {name}", pair.0, pair.1);

    let days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];   // [i64; 12] (from sum's parameter type)
    let zeros = [0u8; 5];                                  // five zeros
    println!("year: {}, first quarter: {}, zeros: {zeros:?}", sum(&days), sum(&days[0..3]));
    let v = vec![10, 20, 30];
    println!("vector: {}, len {}", sum(&v), v.len());

    let index = days.len() + std::env::args().count();   // not known at compile time
    match days.get(index) {                                // checked access returns Option
        Some(d) => println!("day count {d}"),
        None => println!("no month at index {index}"),
    }
    println!("{}", days[index]);                           // panics: index out of bounds
}
```

```console
$ rustc --edition 2024 compound.rs && ./compound
7 seven / 7 seven
year: 365, first quarter: 90, zeros: [0, 0, 0, 0, 0]
vector: 60, len 3
no month at index 13

thread 'main' (1073) panicked at compound.rs:26:20:
index out of bounds: the len is 12 but the index is 13
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: core::panicking::panic_bounds_check
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:271:5
   3: compound::main
   4: <fn() as core::ops::function::FnOnce<()>>::call_once
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

`get` returns an `Option` — `Some(value)` or `None` — that the program must handle (Chapter 3 covers `Option` and `match` fully). Plain indexing panics with the index and the length. A panic in the main thread ends the program with a message and exit code 101: a controlled stop, not memory corruption.

<!-- expanded -->
### Deep & Tricky

**DT-5. Arrays are `Copy` when their elements are, so passing `[i32; 1000]` to a function or assigning it copies 4000 bytes — and a large array declared as a local lives on the stack, where `[0u8; 16 * 1024 * 1024]` overflows the main thread's 8 MiB stack.** *(Guarantee: `[T; N]: Copy` when `T: Copy`; stack size is the platform's — 8 MiB for the main thread on Linux, 2 MiB for spawned threads by default)* Pass arrays by reference (`&[T]` or `&[T; N]`), and put large buffers in a `Vec` or `Box<[T]>`.

```rust
// file: bigarray.rs
fn sum(a: [u64; 4]) -> u64 { a.iter().sum() }      // receives a copy

#[inline(never)]
fn big_local(i: usize) -> u8 {
    let on_stack = [0u8; 16 * 1024 * 1024];        // 16 MiB on the stack
    std::hint::black_box(&on_stack)[i]
}

fn main() {
    let mut a = [1u64, 2, 3, 4];
    let total = sum(a);
    a[0] = 100;                                    // the caller's array is unaffected by the call
    println!("sum {total}, a[0] now {}", a[0]);

    let on_heap = vec![0u8; 16 * 1024 * 1024];     // 16 MiB on the heap: fine
    println!("heap buffer of {} bytes", on_heap.len());
    if std::env::args().count() > 1 {
        println!("{}", big_local(std::env::args().count()));
    }
}
```

```console
$ rustc --edition 2024 bigarray.rs && ./bigarray && ./bigarray boom; echo "exit status $?"
sum 10, a[0] now 100
heap buffer of 16777216 bytes
sum 10, a[0] now 100
heap buffer of 16777216 bytes

thread 'main' (1104) has overflowed its stack
fatal runtime error: stack overflow, aborting
bash: line 1:  1104 Aborted                 ./bigarray boom
exit status 134
```

Rust's runtime catches the stack overflow (with a guard page) and prints a clear message before aborting, rather than corrupting memory silently. The big array is in a separate function on purpose: a function's whole stack frame is reserved when it is entered, so had the array been declared inside the `if` in `main`, the program would have crashed at the start of `main` even when the branch was never taken — which is what the first version of this example did.

## Unit 1.4 — Strings: `String` and `&str`

**One-line summary:** Rust strings are always valid **UTF-8**. `String` is an owned, growable buffer (like `std::string`); `&str` is a borrowed **string slice** — a view into UTF-8 bytes owned by someone else (a `String`, or the program binary for literals). Because a character may take 1 to 4 bytes, strings cannot be indexed by an integer; you iterate over `chars()` or `bytes()`, or slice at byte offsets that fall on character boundaries.

```rust
// file: strings.rs
fn main() {
    let literal: &str = "naïve café";                  // borrowed from the binary
    let mut owned = String::from("crab");               // owned, on the heap
    owned.push_str(" 🦀");
    owned.push('!');
    let joined = format!("{literal} + {owned}");       // format! builds a new String

    println!("{joined}");
    println!("bytes: {}, chars: {}", literal.len(), literal.chars().count());
    println!("uppercase: {}, contains 'café': {}", literal.to_uppercase(), literal.contains("café"));
    println!("first 5 bytes: {:?}", &literal[0..5]);    // 'n','a','ï' (2 bytes),'v' — on boundaries
    for (i, ch) in "aï🦀".char_indices() {
        println!("  byte {i}: {ch:?} ({} bytes)", ch.len_utf8());
    }
    let words: Vec<&str> = "  split   these words ".split_whitespace().collect();
    println!("{words:?}");
    let maybe: Option<char> = literal.chars().nth(2);  // the third character, by walking
    println!("third char: {maybe:?}");
    println!("{}", &literal[0..3]);                      // byte 3 is inside 'ï': panic
}
```

```console
$ rustc --edition 2024 strings.rs && ./strings
naïve café + crab 🦀!
bytes: 12, chars: 10
uppercase: NAÏVE CAFÉ, contains 'café': true
first 5 bytes: "naïv"
  byte 0: 'a' (1 bytes)
  byte 1: 'ï' (2 bytes)
  byte 3: '🦀' (4 bytes)
["split", "these", "words"]
third char: Some('ï')

thread 'main' (1105) panicked at strings.rs:20:28:
end byte index 3 is not a char boundary; it is inside 'ï' (bytes 2..4 of string)
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: core::str::slice_error_fail_rt
   3: core::str::slice_error_fail
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/str/mod.rs:69:5
   4: <core::ops::range::Range<usize> as core::slice::index::SliceIndex<str>>::index
   5: <str as core::ops::index::Index<core::ops::range::Range<usize>>>::index
   6: strings::main
   7: <fn() as core::ops::function::FnOnce<()>>::call_once
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

### Deep & Tricky

**DT-6. `len()` on a string counts bytes, and slicing in the middle of a character panics.** *(Guarantee: Rust standard library)* `"naïve café".len()` is 12, not 10, because `ï` and `é` are two bytes each in UTF-8. `s[i]` does not compile — there is no constant-time way to find the *i*th character — and `&s[a..b]` panics unless both offsets are on character boundaries. To work with characters use `chars()`; for user-perceived characters (an emoji with a skin-tone modifier, or a letter with combining accents, is several `char`s) use the `unicode-segmentation` crate. Byte offsets from `find` or `char_indices` are always on boundaries and safe to slice with.

<!-- expanded -->
### Going deeper: iterating characters, bytes, and graphemes

A `String` is UTF-8 bytes. `.bytes()` gives the bytes, `.chars()` gives Unicode scalar values (`char`, 4 bytes each), and `.char_indices()` gives each `char` with its byte offset — the offsets you may safely slice at. What a user sees as one character (a *grapheme cluster*, like a flag or "é" written as `e` + combining accent) can be several `char`s; splitting those needs the `unicode-segmentation` crate.

```rust
// file: chars.rs
fn main() {
    let s = "naïve café 🇮🇳";
    println!("bytes: {}, chars: {}", s.len(), s.chars().count());
    for (i, c) in s.char_indices().filter(|(_, c)| !c.is_ascii()) {
        println!("  byte {i:2}: {c:?} is {} bytes (U+{:04X})", c.len_utf8(), c as u32);
    }
    let first_word_end = s.find(' ').unwrap();           // byte index of the first space
    println!("first word: {}", &s[..first_word_end]);
    println!("safe to cut at byte 3? {}", s.is_char_boundary(3));
    println!("uppercase: {}", s.to_uppercase());
    let reversed: String = s.chars().rev().collect();
    println!("chars reversed: {reversed}");
}
```

```console
$ rustc --edition 2024 chars.rs && ./chars
bytes: 21, chars: 13
  byte  2: 'ï' is 2 bytes (U+00EF)
  byte 10: 'é' is 2 bytes (U+00E9)
  byte 13: '🇮' is 4 bytes (U+1F1EE)
  byte 17: '🇳' is 4 bytes (U+1F1F3)
first word: naïve
safe to cut at byte 3? false
uppercase: NAÏVE CAFÉ 🇮🇳
chars reversed: 🇳🇮 éfac evïan
```

The flag 🇮🇳 is two `char`s (two "regional indicator" symbols); reversing by `char` swaps them, which turns it into a different flag (🇳🇮, Nicaragua's code "NI"). Reversing user-visible text correctly needs grapheme segmentation.

## Topic 1B — Code

## Unit 1.5 — Functions, statements, and expressions

**One-line summary:** Functions declare every parameter type and the return type (`-> T`); the body is a **block**, and a block's value is its last expression — written *without* a semicolon. Almost everything in Rust is an expression: `if`, `match`, `loop`, and blocks all produce values. A semicolon turns an expression into a statement whose value is `()` (the **unit** type, Rust's "nothing").

```rust
// file: functions.rs
fn area(width: f64, height: f64) -> f64 {
    width * height                       // no semicolon: this is the return value
}

fn classify(n: i32) -> &'static str {
    if n < 0 {
        return "negative";               // early return uses `return`
    }
    if n % 2 == 0 { "even" } else { "odd" }   // if is an expression
}

fn gcd(a: u64, b: u64) -> u64 {
    if b == 0 { a } else { gcd(b, a % b) }
}

fn main() {
    let a = area(3.0, 4.5);
    let label = {
        let doubled = a * 2.0;
        if doubled > 20.0 { "big" } else { "small" }   // the block's value
    };
    println!("area {a}, {label}; 7 is {}, -2 is {}; gcd(84, 36) = {}", classify(7), classify(-2), gcd(84, 36));
}
```

```console
$ rustc --edition 2024 functions.rs && ./functions
area 13.5, big; 7 is odd, -2 is negative; gcd(84, 36) = 12
```

Adding a semicolon after the last expression is a common beginner error; the compiler explains it:

```rust
// file: semicolon.rs
fn square(x: i32) -> i32 {
    x * x;
}

fn main() {
    println!("{}", square(4));
}
```

```console
$ rustc --edition 2024 semicolon.rs
error[E0308]: mismatched types
 --> semicolon.rs:2:22
  |
2 | fn square(x: i32) -> i32 {
  |    ------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
3 |     x * x;
  |          - help: remove this semicolon to return this value

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0308`.
```

<!-- expanded -->
### Going deeper: blocks are expressions

Almost everything in Rust is an expression: a block `{ ... }` evaluates to its last expression (if it has no semicolon), and so do `if`, `match`, and `loop` (through `break value`). This removes the need for many temporary `mut` variables.

```rust
// file: blocks.rs
fn classify(n: i32) -> &'static str {
    if n < 0 { "negative" } else if n == 0 { "zero" } else { "positive" }   // the if is the return value
}

fn main() {
    let area = {
        let w = 3;
        let h = 4;
        w * h                                     // no semicolon: the block's value
    };
    let first_square_over_50 = {
        let mut i = 1;
        loop {
            if i * i > 50 { break i; }            // loop returns a value through break
            i += 1;
        }
    };
    let parity = match area % 2 { 0 => "even", _ => "odd" };
    let unit = { area; };                         // a trailing semicolon makes the value ()
    println!("{area} {first_square_over_50} {parity} {} {unit:?}", classify(-5));
}
```

```console
$ rustc --edition 2024 blocks.rs 2>&1 | grep -m1 -oE "warning: .*"; ./blocks
warning: path statement with no effect
12 8 even negative ()
```

The compiler warns that `area;` is a statement with no effect, and the block's value is the unit type `()`. That trailing-semicolon mistake is also behind the common error "expected `i32`, found `()`" on functions whose last line ends with `;`.

## Unit 1.6 — Control flow

**One-line summary:** `if` needs a `bool` (no truthiness: `if count` does not compile); `loop` repeats forever and can `break` with a value; `while` loops on a condition; `for` iterates over anything iterable — ranges (`0..n`, `0..=n`), collections, iterators. **Labels** (`'outer:`) let `break` and `continue` target an enclosing loop.

```rust
// file: control.rs
fn main() {
    // loop with a value: find the first power of 3 above 1000
    let mut p = 1u32;
    let first = loop {
        p *= 3;
        if p > 1000 {
            break p;
        }
    };
    // while
    let mut n = 27u64;
    let mut steps = 0;
    while n != 1 {
        n = if n % 2 == 0 { n / 2 } else { 3 * n + 1 };
        steps += 1;
    }
    // for over ranges, reversed, stepped
    let evens: Vec<i32> = (0..=10).step_by(2).collect();
    let countdown: Vec<i32> = (1..=5).rev().collect();
    // labelled break out of nested loops
    let mut found = None;
    'search: for a in 1..20 {
        for b in a..20 {
            if a * a + b * b == 13 * 13 {
                found = Some((a, b));
                break 'search;
            }
        }
    }
    println!("first power of 3 above 1000: {first}; Collatz(27) takes {steps} steps");
    println!("evens {evens:?}, countdown {countdown:?}, 5-12-13 search: {found:?}");
}
```

```console
$ rustc --edition 2024 control.rs && ./control
first power of 3 above 1000: 2187; Collatz(27) takes 111 steps
evens [0, 2, 4, 6, 8, 10], countdown [5, 4, 3, 2, 1], 5-12-13 search: Some((5, 12))
```

<!-- expanded -->
### Going deeper: labelled loops and `for` over ranges

Nested loops can be broken out of, or continued, by label. Ranges come in two forms: `a..b` excludes `b`, `a..=b` includes it; `.rev()` and `.step_by(n)` change the direction and the step.

```rust
// file: loops.rs
fn main() {
    let grid = [[1, 2, 3], [4, 99, 6], [7, 8, 9]];
    let mut found = None;
    'rows: for (r, row) in grid.iter().enumerate() {
        for (c, &v) in row.iter().enumerate() {
            if v == 99 {
                found = Some((r, c));
                break 'rows;                       // leaves both loops
            }
        }
    }
    println!("found 99 at {found:?}");

    let evens: Vec<u32> = (0..=10).step_by(2).collect();
    let countdown: Vec<u32> = (1..4).rev().collect();
    println!("{evens:?} {countdown:?}");

    let mut n = 27u64;
    let mut steps = 0;
    while n != 1 {                                 // the Collatz sequence
        n = if n % 2 == 0 { n / 2 } else { 3 * n + 1 };
        steps += 1;
    }
    println!("27 reaches 1 in {steps} steps");
}
```

```console
$ rustc --edition 2024 loops.rs && ./loops
found 99 at Some((1, 1))
[0, 2, 4, 6, 8, 10] [3, 2, 1]
27 reaches 1 in 111 steps
```

## Unit 1.7 — Formatted output and debugging prints

**One-line summary:** `println!`, `print!`, `eprintln!` (to stderr), and `format!` (to a `String`) share one format syntax, checked at compile time: `{}` uses the type's `Display` (for users), `{:?}` its `Debug` (for programmers), `{:#?}` pretty-prints. After the colon come fill, alignment, width, precision, and base. `dbg!(expr)` prints the expression, its value, and the file and line to stderr — and returns the value.

```rust
// file: format.rs
fn main() {
    let pi = std::f64::consts::PI;
    let name = "Ada";
    let items = vec![("tea", 3.5), ("cake", 4.25)];
    println!("[{name:>8}] [{name:<8}] [{name:^8}] [{name:*^9}]");
    println!("{pi:.3} {pi:10.2} {pi:<10.4}| {:e}", 1234.5);
    println!("{0} {1} {0} / {x}", "a", "b", x = 42);
    println!("hex {:x} {:#x} {:08b} octal {:o}", 255, 255, 5u8, 8);
    println!("{:+} {:+}", 5, -5);
    println!("debug: {items:?}");
    println!("pretty: {items:#?}");
    let width = 12;
    println!("[{:>width$.2}]", pi);
    let total = dbg!(items.iter().map(|(_, price)| price).sum::<f64>()) * 2.0;
    eprintln!("this line goes to stderr; total x2 = {total}");
}
```

```console
$ rustc --edition 2024 format.rs && ./format 2>&1
[     Ada] [Ada     ] [  Ada   ] [***Ada***]
3.142       3.14 3.1416    | 1.2345e3
a b a / 42
hex ff 0xff 00000101 octal 10
+5 -5
debug: [("tea", 3.5), ("cake", 4.25)]
pretty: [
    (
        "tea",
        3.5,
    ),
    (
        "cake",
        4.25,
    ),
]
[        3.14]
[format.rs:15:17] items.iter().map(|(_, price)| price).sum::<f64>() = 7.75
this line goes to stderr; total x2 = 15.5
```

<!-- expanded -->
### Going deeper: format specifications

The text inside `{}` has a small language: `{:>8}` right-aligns in 8 columns, `{:<8}` left, `{:^8}` centres, `{:08.3}` pads a float with zeros to width 8 with 3 decimals, `{:#x}` and `{:#b}` print hex and binary with a prefix, `{:e}` uses scientific notation, `{:?}` and `{:#?}` print `Debug` forms. Width and precision can come from arguments, by name or position.

```rust
// file: fmtspec.rs
fn main() {
    let items = [("apples", 3, 0.5), ("kiwis", 12, 0.25), ("watermelon", 1, 3.0)];
    println!("{:<12}|{:>5}|{:>8}", "item", "qty", "cost");
    for (name, qty, price) in items {
        println!("{name:<12}|{qty:>5}|{:>8.2}", qty as f64 * price);
    }
    let width = 10;
    println!("[{:^width$}] [{:>1$}] [{:.*}]", "mid", 6, 3, std::f64::consts::PI);
    println!("{:#x} {:#b} {:#o} {:08.3} {:e}", 255, 5, 8, -3.14159, 1234.5);
    println!("{{literal braces}} and {0} {0} {1}", "repeat", "once");
}
```

```console
$ rustc --edition 2024 fmtspec.rs && ./fmtspec
item        |  qty|    cost
apples      |    3|    1.50
kiwis       |   12|    3.00
watermelon  |    1|    3.00
[   mid    ] [     6] [3.142]
0xff 0b101 0o10 -003.142 1.2345e3
{literal braces} and repeat repeat once
```

The format string is checked at compile time: a wrong number of arguments, or `{:x}` on a type without `LowerHex`, is a compile error.

## Unit 1.8 — Reading input and parsing

**One-line summary:** `std::io::stdin().read_line(&mut s)` appends a line (with its newline) to a `String`; `stdin().lines()` iterates over lines. `str::parse::<T>()` converts text to a number (or anything implementing `FromStr`) and returns a `Result` — `Ok(value)` or `Err(error)` — that must be handled. Handling it with `match` is shown here; Chapter 4 covers error handling in full.

```rust
// file: input.rs
use std::io;

fn main() {
    let mut total: i64 = 0;
    for (i, line) in io::stdin().lines().enumerate() {
        let line = line.expect("stdin should be readable");   // an I/O error would panic with this message
        match line.trim().parse::<i64>() {
            Ok(n) => total += n,
            Err(e) => eprintln!("line {}: {:?} is not a number ({e})", i + 1, line),
        }
    }
    println!("total = {total}");
}
```

```console
$ rustc --edition 2024 input.rs && printf '10\n 20 \nabc\n\n-5\n9999999999999999999999\n' | ./input
line 3: "abc" is not a number (invalid digit found in string)
line 4: "" is not a number (cannot parse integer from empty string)
line 6: "9999999999999999999999" is not a number (number too large to fit in target type)
total = 25
```

`parse` reports *why* each bad line failed: invalid digits, an empty string, or a number too large for `i64`.

### Deep & Tricky

**DT-7. `f64` does not implement `Ord`, so `sort()` and `max()` do not compile on floats.** *(Guarantee: Rust standard library; IEEE 754 behaviour)* Because NaN is not equal to itself and not ordered with anything, floats have only a *partial* order (`PartialOrd`). Rust refuses to pretend otherwise: `v.sort()` on a `Vec<f64>` is a compile-time error. Use `v.sort_by(f64::total_cmp)`, which uses the IEEE 754 *totalOrder* (negative NaNs first, then −∞ … +∞, then positive NaNs), or filter out NaN first. C++'s `std::sort` on data containing NaN violates its precondition and is undefined behaviour; Rust makes you choose.

<!-- expanded -->
### Going deeper: reading all of standard input quickly

`read_line` in a loop allocates and locks standard input on every call. For programs that read a lot of input — competitive programming, log processing — lock `stdin` once and either iterate `lines()` on the locked handle or read everything into one `String` and split it.

```rust
// file: fastinput.rs
use std::io::{self, BufRead, Read};

fn main() -> io::Result<()> {
    let mode = std::env::args().nth(1).unwrap_or_default();
    let mut sum: i64 = 0;
    if mode == "all" {
        let mut text = String::new();
        io::stdin().lock().read_to_string(&mut text)?;          // one read of everything
        for tok in text.split_ascii_whitespace() { sum += tok.parse::<i64>().unwrap(); }
    } else {
        for line in io::stdin().lock().lines() {                // one lock, line by line
            for tok in line?.split_ascii_whitespace() { sum += tok.parse::<i64>().unwrap(); }
        }
    }
    println!("sum = {sum}");
    Ok(())
}
```

```console
$ rustc --edition 2024 -O fastinput.rs && seq 1 1000000 > nums.txt && ./fastinput < nums.txt && ./fastinput all < nums.txt
sum = 500000500000
sum = 500000500000
```

### Mastery check

1. **Predict.** What does `println!("{}", (-7i32).rem_euclid(3) + (-7 % 3));` print?
2. **Predict.** What do `"héllo".len()`, `"héllo".chars().count()`, and `"héllo".chars().rev().collect::<String>()` produce?

## Chapter 1 Project — `scorestat`: statistics over a stream of scores

**Goal:** a command-line program that reads scores (one per line, possibly with a name: `Ada 91.5` or just `88`) from standard input, skips and reports bad lines, and prints count, mean, median, minimum, maximum, standard deviation, and a text histogram in buckets of 10.

### The program

```rust
// file: scorestat/scorestat.rs
use std::io;

fn parse_line(line: &str) -> Result<f64, String> {
    let field = line.split_whitespace().last().ok_or("empty line")?;   // the score is the last field
    let value: f64 = field.parse().map_err(|_| format!("{field:?} is not a number"))?;
    if !(0.0..=100.0).contains(&value) {
        return Err(format!("{value} is outside 0..=100"));
    }
    Ok(value)
}

fn median(sorted: &[f64]) -> f64 {
    let n = sorted.len();
    if n % 2 == 1 { sorted[n / 2] } else { (sorted[n / 2 - 1] + sorted[n / 2]) / 2.0 }
}

fn main() {
    let mut scores: Vec<f64> = Vec::new();
    let mut bad = 0;
    for (number, line) in io::stdin().lines().enumerate() {
        let line = line.expect("stdin should be readable");
        if line.trim().is_empty() {
            continue;
        }
        match parse_line(&line) {
            Ok(v) => scores.push(v),
            Err(why) => {
                bad += 1;
                eprintln!("skipping line {}: {why}", number + 1);
            }
        }
    }
    if scores.is_empty() {
        println!("no valid scores ({bad} bad lines)");
        return;
    }
    scores.sort_by(f64::total_cmp);                       // DT-7: f64 has no total order
    let n = scores.len() as f64;
    let mean = scores.iter().sum::<f64>() / n;
    let variance = scores.iter().map(|s| (s - mean) * (s - mean)).sum::<f64>() / n;
    println!("count  {:>7}   (skipped {bad})", scores.len());
    println!("mean   {mean:>7.2}");
    println!("median {:>7.2}", median(&scores));
    println!("min    {:>7.2}   max {:.2}", scores[0], scores[scores.len() - 1]);
    println!("stddev {:>7.2}", variance.sqrt());

    let mut buckets = [0usize; 10];                       // 0-9, 10-19, ..., 90-100
    for &s in &scores {
        let b = ((s / 10.0) as usize).min(9);             // 100 goes in the last bucket
        buckets[b] += 1;
    }
    let widest = *buckets.iter().max().unwrap_or(&1);
    for (i, &count) in buckets.iter().enumerate() {
        let bar = "#".repeat((count * 40).div_ceil(widest.max(1)));
        let range = if i == 9 { "90-100".to_string() } else { format!("{}-{}", i * 10, i * 10 + 9) };
        println!("{range:>6} | {bar} {count}");
    }
}
```

```text
<!-- file: scorestat/scores.txt -->
Ada 91.5
Grace 88
Linus 72.25
Margaret 100
Dennis 64
Ken 79
Barbara 85.5
Bjarne fifty
Guido 58
Anders 120
Brian 77

James 93
Niklaus 69.75
Edsger 99
Alan 40
```

```console
$ cd scorestat && rustc --edition 2024 -O scorestat.rs && ./scorestat < scores.txt
skipping line 8: "fifty" is not a number
skipping line 10: 120 is outside 0..=100
count       13   (skipped 2)
mean     78.23
median   79.00
min      40.00   max 100.00
stddev   16.71
   0-9 |  0
 10-19 |  0
 20-29 |  0
 30-39 |  0
 40-49 | ########## 1
 50-59 | ########## 1
 60-69 | #################### 2
 70-79 | ############################## 3
 80-89 | #################### 2
90-100 | ######################################## 4
$ cd scorestat && printf 'only bad\n' | ./scorestat
skipping line 1: "bad" is not a number
no valid scores (1 bad lines)
```

The program uses everything in this chapter: `String` and `&str` with `split_whitespace`; `parse` with errors turned into messages; a `Vec<f64>` sorted with `total_cmp`; slices for `median`; an array of counters; `as` conversions chosen deliberately (truncating a score to a bucket index is exactly what we want); and formatted, aligned output. It also uses two things that Chapter 4 explains properly: functions returning `Result`, and the `?` operator, which returns early with the error.

### Review checklist

- Which inputs reach each branch of `parse_line`? Try an empty name, a negative score, `NaN`, and `inf` — what happens to `NaN`, and why? (Hint: `(0.0..=100.0).contains(&NaN)` is `false`.)
- Why does the program sort with `total_cmp` instead of `sort()`?
- What would happen with `as usize` if a score of `-0.5` got through? Is that possible here?

## Answers to the Chapter 1 prediction questions

**1.8 Q1.** `1`. `(-7).rem_euclid(3)` is 2 (the Euclidean remainder is never negative), and `-7 % 3` is −1 (Rust's `%` truncates toward zero, like C), so the sum is 1.

**1.8 Q2.** `6` (the `é` is two bytes), `5`, and `"olléh"`. Reversing by `char` is correct here because each character is a single Unicode scalar value; it would not be for characters built from combining marks.

# Chapter 2 — Ownership, Borrowing, and Lifetimes

*Goal: understand the rules that make Rust memory-safe without a garbage collector — ownership, moves, borrowing, and lifetimes — well enough to predict what the borrow checker will accept, to read its errors, and to restructure code that it rejects.*

This is the most important chapter in the book. Everything else in Rust — collections, traits, smart pointers, threads, async — is built on the three rules introduced here. They feel restrictive for a few weeks, then they become the way you think about data: who owns it, who is looking at it, and who may change it.

**What you will be able to do after this chapter**

- State the ownership rules, and predict when a value is moved, copied, or dropped.
- Use shared (`&T`) and mutable (`&mut T`) references within the borrowing rules.
- Explain how the borrow checker prevents dangling references, iterator invalidation, and aliasing bugs.
- Write lifetime annotations on functions and structs, and know when elision makes them unnecessary.
- Restructure code the borrow checker rejects, using the standard techniques.

## Topic 2A — Ownership

## Unit 2.1 — The ownership rules

**One-line summary:** (1) Every value has exactly one **owner** — a variable, a field, or an element of a collection. (2) When the owner goes out of scope, the value is **dropped**: its destructor (`Drop`) runs and its memory — including any heap memory it manages — is freed. (3) Assigning or passing a value **moves** ownership; the source can no longer be used. Types whose values are plain bits (`i32`, `f64`, `bool`, `char`, `&T`, and tuples and arrays of them) are **`Copy`**: they are duplicated instead of moved.

A type with a visible `Drop` makes the rules observable:

```rust
// file: ownership.rs
struct Noisy(&'static str);

impl Drop for Noisy {
    fn drop(&mut self) {
        println!("  drop {}", self.0);
    }
}

fn take(n: Noisy) {
    println!("  take() owns {}", n.0);
}                                                   // n dropped here

fn make(name: &'static str) -> Noisy {
    Noisy(name)                                     // ownership returned to the caller
}

fn main() {
    println!("start");
    let a = Noisy("a");
    let b = a;                                      // move: `a` is now unusable
    take(b);                                        // move into the function
    println!("after take");
    let c = make("c");
    {
        let _d = Noisy("d");
        println!("inner scope ends");
    }                                               // d dropped here
    let e = Noisy("e");
    drop(e);                                        // drop early with std::mem::drop
    let x = 5;
    let y = x;                                      // i32 is Copy: both usable
    println!("x={x} y={y}, c is {}", c.0);
    println!("end of main");
}                                                   // c dropped here
```

```console
$ rustc --edition 2024 ownership.rs && ./ownership
start
  take() owns a
  drop a
after take
inner scope ends
  drop d
  drop e
x=5 y=5, c is c
end of main
  drop c
```

Every drop happens exactly once, at a point the compiler determines statically: at the end of the owner's scope, or when ownership is handed somewhere else that drops it. There is no garbage collector and no reference counting here — the compiler simply inserts the destructor calls, like C++'s RAII (C++ book, Chapter 2), but with the moved-from object *statically* unusable instead of left in a "valid but unspecified" state.

Values that own heap memory are moved, not copied, because copying the pointer would give two owners and a double free. To duplicate the data explicitly, call `.clone()`:

```rust
// file: clone.rs
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();                            // deep copy: a second heap buffer
    println!("{s1} {s2}, different buffers: {}", s1.as_ptr() != s2.as_ptr());
    let s3 = s1;                                    // move: s3 takes over s1's buffer
    println!("{s3}");
    // println!("{s1}");                            // error[E0382]: borrow of moved value
}
```

```console
$ rustc --edition 2024 clone.rs && ./clone
hello hello, different buffers: true
hello
```

### Deep & Tricky

**DT-8. A move is a bitwise copy of the value's bytes — the compiler then forbids using the source.** *(Guarantee: Rust language; Compiler behaviour)* Moving a `String` copies its three words (pointer, capacity, length); the heap buffer is not touched. No code runs (there are no move constructors in Rust), and the source is not modified — it simply may not be used again, and its destructor will not run. The optimiser usually eliminates the copy entirely. This is why Rust moves are always cheap and can never fail, and why every type is movable without any work from its author. (`Copy` types are the ones for which that bitwise copy *is* a complete, independent duplicate.)

## Unit 2.2 — Moves in control flow

**One-line summary:** The compiler tracks moves along every path. A value moved in one branch of an `if`, or inside a loop, may not be used where it *might* have been moved. Use a reference instead, clone, or restructure so ownership moves once.

```rust
// file: loopmove.rs
fn consume(s: String) -> usize {
    s.len()
}

fn main() {
    let s = String::from("data");
    let mut total = 0;
    for _ in 0..3 {
        total += consume(s);                        // moved in the first iteration
    }
    println!("{total}");
}
```

```console
$ rustc --edition 2024 loopmove.rs
error[E0382]: use of moved value: `s`
  --> loopmove.rs:10:26
   |
 7 |     let s = String::from("data");
   |         - move occurs because `s` has type `String`, which does not implement the `Copy` trait
 8 |     let mut total = 0;
 9 |     for _ in 0..3 {
   |     ------------- inside of this loop
10 |         total += consume(s);                        // moved in the first iteration
   |                          ^ value moved here, in previous iteration of loop
   |
note: consider changing this parameter type in function `consume` to borrow instead if owning the value isn't necessary
  --> loopmove.rs:2:15
   |
 2 | fn consume(s: String) -> usize {
   |    -------    ^^^^^^ this parameter takes ownership of the value
   |    |
   |    in this function
help: consider cloning the value if the performance cost is acceptable
   |
10 |         total += consume(s.clone());                        // moved in the first iteration
   |                           ++++++++

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0382`.
```

The compiler suggests two fixes — clone, or change `consume` to borrow. The second is usually right: a function that only *reads* a value should take a reference.

<!-- expanded -->
### Going deeper: moving out of a collection, and partial moves

You cannot move a value out of a `Vec` by indexing (`let s = v[0];` with `String` elements is an error), because that would leave a hole the vector does not know about. The options are to clone, to *take* it and leave something in its place (`std::mem::take`, `std::mem::replace`, `Option::take`), to remove it (`swap_remove`, `remove`), or to consume the whole vector with `into_iter()`. Moving one field out of a struct is allowed — a *partial move* — but then the struct as a whole is unusable.

```rust
// file: moveout.rs
#[derive(Debug)]
struct User { name: String, email: String }

fn main() {
    let mut names = vec![String::from("ada"), String::from("alan"), String::from("grace")];
    // let first = names[0];                           // error: cannot move out of index of Vec<String>
    let first = std::mem::take(&mut names[0]);         // leaves an empty String behind
    let last = names.pop().unwrap();                   // removes it
    let mut slot = Some(String::from("token"));
    let tok = slot.take();                             // leaves None
    println!("{first} {last} {names:?} {tok:?} {slot:?}");

    for n in names.into_iter() { print!("owned {n}; "); }   // consumes the vector
    println!();

    let u = User { name: "ada".into(), email: "ada@example.org".into() };
    let name = u.name;                                 // partial move
    println!("moved name {name}, still usable field {}", u.email);
    // println!("{u:?}");                              // error: borrow of partially moved value
}
```

```console
$ rustc --edition 2024 moveout.rs && ./moveout
ada grace ["", "alan"] Some("token") None
owned ; owned alan; 
moved name ada, still usable field ada@example.org
```

## Topic 2B — Borrowing

## Unit 2.3 — References and the borrowing rules

**One-line summary:** A **reference** borrows a value without taking ownership. `&T` is a **shared** reference: many may exist at once, and none may modify the value. `&mut T` is a **mutable** (exclusive) reference: while it exists, no other reference to the value may be used. In short: **many readers or one writer, never both** — and every reference must be valid for as long as it is used.

```rust
// file: borrow.rs
fn length(s: &String) -> usize {                    // borrows; does not own
    s.len()
}

fn shout(s: &mut String) {                          // borrows mutably
    s.make_ascii_uppercase();
    s.push('!');
}

fn main() {
    let mut s = String::from("hello");
    let r1 = &s;
    let r2 = &s;                                    // many shared borrows: fine
    println!("{r1} {r2} {}", length(&s));
    shout(&mut s);                                  // r1 and r2 are no longer used: fine
    println!("{s}");
}
```

```console
$ rustc --edition 2024 borrow.rs && ./borrow
hello hello 5
HELLO!
```

A borrow lasts from where it is created to its **last use** — not to the end of the scope (this is called *non-lexical lifetimes*). That is why `shout(&mut s)` is allowed after `r1` and `r2` are last used. Using a shared borrow *after* a mutable one is created breaks the rule:

```rust
// file: conflict.rs
fn main() {
    let mut s = String::from("hello");
    let r1 = &s;
    let w = &mut s;
    w.push('!');
    println!("{r1}");                               // r1 still in use: conflict
}
```

```console
$ rustc --edition 2024 conflict.rs
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> conflict.rs:5:13
  |
4 |     let r1 = &s;
  |              -- immutable borrow occurs here
5 |     let w = &mut s;
  |             ^^^^^^ mutable borrow occurs here
6 |     w.push('!');
7 |     println!("{r1}");                               // r1 still in use: conflict
  |                -- immutable borrow later used here

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0502`.
```

Why is this rule worth the trouble? Because "one writer, no readers" is exactly the condition under which a whole family of bugs cannot occur. Here is one of them — iterator invalidation, C++ book Unit 13.2 — which in C++ compiles cleanly and fails at run time:

```rust
// file: invalidate.rs
fn main() {
    let mut v = vec![1, 2, 3];
    let first = &v[0];                              // a reference into v's buffer
    v.push(4);                                      // may reallocate the buffer
    println!("first = {first}");
}
```

```console
$ rustc --edition 2024 invalidate.rs
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> invalidate.rs:5:5
  |
4 |     let first = &v[0];                              // a reference into v's buffer
  |                  - immutable borrow occurs here
5 |     v.push(4);                                      // may reallocate the buffer
  |     ^^^^^^^^^ mutable borrow occurs here
6 |     println!("first = {first}");
  |                        ----- immutable borrow later used here

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0502`.
```

The C++ version of this program was a heap-use-after-free that only AddressSanitizer caught. In Rust it does not compile: `push` needs `&mut v`, and `first` is a live shared borrow of `v`.

### Deep & Tricky

**DT-9. `&mut` means *exclusive*, not just *mutable* — and the compiler optimises on that guarantee.** *(Guarantee: Rust language; Compiler behaviour: `noalias`)* Because a `&mut T` is the only active path to its target, and the target of a `&T` cannot change while the reference is in use (unless it is wrapped in an `UnsafeCell`, Chapter 8), rustc tells LLVM that references do not alias — what C's `restrict` promises, but checked. Two `&mut` to the same value, created by `unsafe` code, are undefined behaviour even if neither is ever written through. Some types (`Cell`, `RefCell`, `Mutex`, atomics) allow mutation through a shared reference; they are built on `UnsafeCell` precisely to opt out of this guarantee in a controlled way.

<!-- expanded -->
### Going deeper: two-phase borrows

`v.push(v.len())` looks like it breaks the rules: `push` needs `&mut v`, and `v.len()` needs `&v` at the same time. It compiles because of **two-phase borrows**: the mutable borrow for a method's receiver is first *reserved* and only *activated* after the arguments have been evaluated. The rule covers exactly this autoref-method-call shape, not general code.

```rust
// file: twophase.rs
fn main() {
    let mut v = vec![1, 2, 3];
    v.push(v.len());                      // OK: &mut v is activated after v.len() runs
    println!("{v:?}");

    let r = &mut v;
    // r.push(v.len());                   // error: v is already mutably borrowed through r
    let n = r.len();
    r.push(n);                            // compute first, then mutate
    println!("{v:?}");
}
```

```console
$ rustc --edition 2024 twophase.rs && ./twophase
[1, 2, 3, 3]
[1, 2, 3, 3, 4]
```

## Unit 2.4 — No dangling references

**One-line summary:** A reference may not outlive the value it refers to. The borrow checker compares how long each value lives with how long each reference to it is used, and rejects any program where a reference could be used after its referent is dropped or moved.

```rust
// file: dangle_return.rs
fn longest_word(text: &str) -> &str {               // fine: the result borrows from `text`
    text.split_whitespace().max_by_key(|w| w.len()).unwrap_or("")
}

fn make_greeting() -> &String {                    // a reference to what?
    let s = String::from("hi");
    &s                                              // s is dropped when the function returns
}

fn main() {
    println!("{} {}", longest_word("borrow checker rules"), make_greeting());
}
```

```console
$ rustc --edition 2024 dangle_return.rs 2>&1 | grep -E '^error|^ *= help|^help|-->'
error[E0106]: missing lifetime specifier
 --> dangle_return.rs:6:23
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime, but this is uncommon unless you're returning a borrowed value from a `const` or a `static`
help: instead, you are more likely to want to return an owned value
error: aborting due to 1 previous error
```

```rust
// file: dangle_scope.rs
fn main() {
    let r;
    {
        let x = 5;
        r = &x;
    }                                               // x dropped here
    println!("{r}");
}
```

```console
$ rustc --edition 2024 dangle_scope.rs
error[E0597]: `x` does not live long enough
 --> dangle_scope.rs:6:13
  |
5 |         let x = 5;
  |             - binding `x` declared here
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }                                               // x dropped here
  |     - `x` dropped here while still borrowed
8 |     println!("{r}");
  |                - borrow later used here

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0597`.
```

Both bugs are caught. For `make_greeting`, the signature itself is rejected: a returned reference must borrow from an input, and there is none — the compiler suggests returning the owned `String`, which is almost always the fix. (`longest_word` needs no annotation: its result borrows from its only reference parameter.) In `dangle_scope.rs`, `r` would be used after `x` is dropped.

<!-- expanded -->
### Deep & Tricky

**DT-10. Returning a reference into a value you create inside the function is always an error — but the fix depends on *why* you wanted a reference. If the caller needs the data, return the owned value (`String`, not `&str`); if the data is constant, return `&'static str`; if it is a view of an argument, tie the lifetimes together.** *(Guarantee: the borrow checker rejects returning references to locals, E0515/E0106)* Adding `'static` or `Box::leak` to silence the error usually turns a compile error into a memory leak or a wrong design.

```rust
// file: returnref.rs
fn owned(n: u32) -> String { format!("item-{n}") }                    // the caller gets ownership
fn constant(ok: bool) -> &'static str { if ok { "yes" } else { "no" } } // literals live forever
fn view(s: &str) -> &str { s.split(',').next().unwrap_or("") }        // borrows from the argument

fn main() {
    let a = owned(7);
    let b = constant(true);
    let input = String::from("first,second");
    let c = view(&input);
    println!("{a} {b} {c}");
}
```

```console
$ rustc --edition 2024 returnref.rs && ./returnref
item-7 yes first
$ printf 'fn bad() -> &str { let s = String::from("x"); &s }\nfn main() { bad(); }\n' > badref.rs && rustc --edition 2024 badref.rs 2>&1 | grep -m1 -oE "error\[E[0-9]+\]: .*"
error[E0106]: missing lifetime specifier
```

The compiler's first complaint is the missing lifetime (E0106) — and its suggestion is to use `'static` or to return an owned `String`; only the second is right here.

## Unit 2.5 — Slices are borrows

**One-line summary:** `&str` and `&[T]` are references to a *part* of a value — and they keep the whole value borrowed. Functions that read text or sequences should take `&str` and `&[T]`, which accept `String`/`Vec` (by borrowing), literals, arrays, and sub-ranges alike.

```rust
// file: slices.rs
fn first_word(s: &str) -> &str {
    match s.find(' ') {
        Some(i) => &s[..i],
        None => s,
    }
}

fn main() {
    let mut sentence = String::from("hello world");
    let word = first_word(&sentence);               // borrows `sentence`
    sentence.clear();                               // needs &mut sentence while `word` is alive
    println!("first word: {word}");
}
```

```console
$ rustc --edition 2024 slices.rs
error[E0502]: cannot borrow `sentence` as mutable because it is also borrowed as immutable
  --> slices.rs:12:5
   |
11 |     let word = first_word(&sentence);               // borrows `sentence`
   |                           --------- immutable borrow occurs here
12 |     sentence.clear();                               // needs &mut sentence while `word` is alive
   |     ^^^^^^^^^^^^^^^^ mutable borrow occurs here
13 |     println!("first word: {word}");
   |                            ---- immutable borrow later used here

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0502`.
```

In C, `first_word` would return a pointer into the string, and `clear` would leave it pointing at garbage. The borrow tells the compiler that `word` depends on `sentence`, so `sentence` may not be mutated while `word` is used.

### Pattern

**P-2. Take parameters by reference unless the function needs to own the value.** *(Practice; Rust API Guidelines)* Accept `&str` rather than `&String` or `String`, `&[T]` rather than `&Vec<T>` or `Vec<T>`, and `&T` rather than `T`, when the function only reads. Take `&mut T` when it modifies in place. Take ownership (`String`, `Vec<T>`, `T`) only when the function stores the value or consumes it — then the caller decides whether to move or clone. *Trade-off:* an owned parameter is sometimes simpler when the function would clone internally anyway; taking it by value lets callers who no longer need their copy avoid the clone.

<!-- expanded -->
### Going deeper: `split_at_mut` and other ways to borrow two parts mutably

Taking `&mut v[0]` and `&mut v[1]` at the same time does not compile: the borrow checker sees two mutable borrows of `v` and does not reason about indices. The standard library provides safe ways to get *disjoint* mutable pieces: `split_at_mut`, `split_first_mut`, `chunks_mut`, `iter_mut`, and (Rust 1.86+) `get_disjoint_mut`.

```rust
// file: disjoint.rs
fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];
    // let (a, b) = (&mut v[0], &mut v[5]);      // error: cannot borrow v as mutable more than once
    let (left, right) = v.split_at_mut(3);
    std::mem::swap(&mut left[0], &mut right[2]);
    println!("after swap: {v:?}");

    if let Ok([a, b]) = v.get_disjoint_mut([1, 4]) {
        *a += 100;
        *b += 100;
    }
    println!("after get_disjoint_mut: {v:?}");

    for chunk in v.chunks_mut(2) { chunk.reverse(); }
    println!("pairs reversed: {v:?}");
    println!("same index twice: {:?}", v.get_disjoint_mut([2, 2]).is_err());
}
```

```console
$ rustc --edition 2024 disjoint.rs && ./disjoint
after swap: [6, 2, 3, 4, 5, 1]
after get_disjoint_mut: [6, 102, 3, 4, 105, 1]
pairs reversed: [102, 6, 4, 3, 1, 105]
same index twice: true
```

`v.swap(0, 5)` exists for the simplest case. When none of these fit — a graph whose nodes point to each other — use indices into a `Vec` instead of references (Unit 2.7).

## Topic 2C — Lifetimes

## Unit 2.6 — Lifetime annotations

**One-line summary:** A **lifetime** is the region of code during which a reference is valid. Usually the compiler infers lifetimes. When a function returns a reference derived from one of several reference parameters, you must say which, with a **lifetime parameter**: `fn longest<'a>(x: &'a str, y: &'a str) -> &'a str` says "the result is valid as long as *both* inputs are". Annotations describe relationships; they never make anything live longer.

```rust
// file: lifetimes.rs
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() >= y.len() { x } else { y }
}

fn first<'a>(x: &'a str, _y: &str) -> &'a str {    // result tied to x only
    x
}

struct Excerpt<'a> {                                // a struct that holds a borrow
    part: &'a str,
}

impl<'a> Excerpt<'a> {
    fn part(&self) -> &'a str {
        self.part
    }
}

fn main() {
    let long_lived = String::from("a fairly long string");
    let result;
    {
        let short_lived = String::from("xyz");
        result = first(&long_lived, &short_lived);   // fine: depends only on long_lived
        println!("longest inside: {}", longest(&long_lived, &short_lived));
    }
    println!("first: {result}");

    let novel = String::from("Call me Ishmael. Some years ago...");
    let sentence = novel.split('.').next().unwrap_or("");
    let e = Excerpt { part: sentence };
    println!("excerpt: {}", e.part());
    let s: &'static str = "string literals live for the whole program";
    println!("{s}");
}
```

```console
$ rustc --edition 2024 lifetimes.rs && ./lifetimes
longest inside: a fairly long string
first: a fairly long string
excerpt: Call me Ishmael
string literals live for the whole program
```

Changing `first` to `longest` in the inner block — so that `result` could refer to `short_lived` — is rejected, because the signature says the result may borrow from either argument:

```rust
// file: lifetimes_bad.rs
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() >= y.len() { x } else { y }
}

fn main() {
    let long_lived = String::from("a fairly long string");
    let result;
    {
        let short_lived = String::from("xyz");
        result = longest(&long_lived, &short_lived);
    }
    println!("{result}");
}
```

```console
$ rustc --edition 2024 lifetimes_bad.rs
error[E0597]: `short_lived` does not live long enough
  --> lifetimes_bad.rs:11:39
   |
10 |         let short_lived = String::from("xyz");
   |             ----------- binding `short_lived` declared here
11 |         result = longest(&long_lived, &short_lived);
   |                                       ^^^^^^^^^^^^ borrowed value does not live long enough
12 |     }
   |     - `short_lived` dropped here while still borrowed
13 |     println!("{result}");
   |                ------ borrow later used here

error: aborting due to 1 previous error

For more information about this error, try `rustc --explain E0597`.
```

The compiler reasons *only* from the signature, never from the body: even though `long_lived` is longer and would be returned at run time, the signature permits returning `y`, so the program is rejected. This is what makes borrow checking modular — a function's callers can be checked without looking inside it.

**Lifetime elision.** Most functions need no annotations, because three rules fill them in: each reference parameter gets its own lifetime; if there is exactly one input lifetime, it is given to all output references; and in methods, if one parameter is `&self` or `&mut self`, its lifetime is given to all outputs. `fn first_word(s: &str) -> &str` is therefore complete as written.

### Deep & Tricky

**DT-11. `T: 'static` does not mean "lives forever"; it means "contains no borrows shorter than the whole program".** *(Guarantee: Rust language)* A `&'static str` points at data that is valid until the program ends, such as a string literal. But the *bound* `T: 'static` — required, for example, by `std::thread::spawn` — is satisfied by every owned type with no references inside it: a `String`, a `Vec<u8>`, an `i32`. Such values can be dropped at any time; what the bound guarantees is that holding one can never leave a dangling reference. "Make it `'static`" usually means "pass owned data", not "leak it".

<!-- expanded -->
### Going deeper: lifetime elision rules

Most functions with references need no lifetime annotations because of three **elision rules**: (1) each reference parameter gets its own lifetime; (2) if there is exactly one input lifetime, it is given to all output references; (3) if one parameter is `&self` or `&mut self`, its lifetime is given to all output references. When the rules do not determine the output lifetime, you must write it.

```rust
// file: elision.rs
struct Parser { text: String }

impl Parser {
    fn first_word(&self, _sep: &str) -> &str {       // rule 3: the output borrows from self, not from _sep
        self.text.split(' ').next().unwrap_or("")
    }
}

fn trim_both(s: &str) -> &str { s.trim() }             // rule 2: one input, so output borrows from s

fn pick<'a>(a: &'a str, _b: &str) -> &'a str { a }     // two inputs: we must say which one

fn main() {
    let p = Parser { text: String::from("hello world") };
    let word;
    {
        let sep = String::from(" ");
        word = p.first_word(&sep);                       // fine: word does not borrow from sep
    }
    println!("{word} [{}] {}", trim_both("  x  "), pick("a", "b"));
}
```

```console
$ rustc --edition 2024 elision.rs && ./elision
hello [x] a
```

Rule 3 is occasionally wrong for what you mean — a method returning a piece of its *argument* needs an explicit lifetime — and then the error appears at the call site, where the borrow of `self` lasts longer than you expected.

## Unit 2.7 — Working with the borrow checker

**One-line summary:** When the borrow checker rejects correct-looking code, the fix is usually one of a few standard moves: end the borrow sooner, borrow disjoint parts separately, use indices or keys instead of long-lived references, clone small data, or restructure into a function that takes exactly what it needs.

```rust
// file: techniques.rs
struct Player {
    name: String,
    score: u32,
}

fn bump(score: &mut u32, by: u32) {
    *score += by;
}

fn main() {
    // 1. Disjoint fields can be borrowed separately
    let mut p = Player { name: String::from("Ada"), score: 10 };
    let name = &p.name;                             // shared borrow of one field...
    bump(&mut p.score, 5);                          // ...mutable borrow of another: fine
    println!("{name}: {}", p.score);

    // 2. Disjoint parts of a slice: split_at_mut
    let mut v = [1, 2, 3, 4, 5, 6];
    let (left, right) = v.split_at_mut(3);
    left[0] += right[0];                            // two &mut into one array, proven disjoint
    println!("{v:?}");

    // 3. Indices instead of references while mutating the collection
    let mut words = vec![String::from("a"), String::from("bb")];
    let longest = (0..words.len()).max_by_key(|&i| words[i].len()).unwrap();
    words.push(format!("{}!", words[longest]));     // no live borrow across the push
    println!("{words:?}");

    // 4. Compute first, then mutate (two-phase borrows allow v.push(v.len()))
    let mut nums = vec![1, 2];
    nums.push(nums.len());
    let total: usize = nums.iter().sum();
    nums.push(total);
    println!("{nums:?}");

    // 5. Swap or take values out without moving from a borrow
    let mut slot = Some(String::from("payload"));
    let taken = slot.take();                        // leaves None behind
    let mut a = String::from("left");
    let mut b = String::from("right");
    std::mem::swap(&mut a, &mut b);
    let old = std::mem::replace(&mut a, String::from("new"));
    println!("{taken:?} {slot:?} {a} {b} {old}");
}
```

```console
$ rustc --edition 2024 techniques.rs && ./techniques
Ada: 15
[5, 2, 3, 4, 5, 6]
["a", "bb", "bb!"]
[1, 2, 2, 5]
Some("payload") None new left right
```

The borrow checker reasons about **paths**: `p.name` and `p.score` are different paths, so they can be borrowed independently. It cannot see through a method call, though — `p.name()` borrows *all* of `p` — and it cannot prove that `v[0]` and `v[1]` are disjoint (indices are run-time values), which is why `split_at_mut` exists: an `unsafe`-implemented, safe-to-call function that proves disjointness by construction.

<!-- expanded -->
### Going deeper: indices instead of references

Data structures where items refer to each other — graphs, trees with parent links, entity systems in games — fight the borrow checker when built from references, because a reference fixes a borrow for its whole lifetime. The idiomatic answer is an **arena**: store the items in a `Vec` and refer to them by index (often wrapped in a newtype so an index for one arena cannot be used in another).

```rust
// file: arena.rs
#[derive(Clone, Copy, Debug, PartialEq)]
struct NodeId(usize);

struct Graph { names: Vec<String>, edges: Vec<Vec<NodeId>> }

impl Graph {
    fn new() -> Self { Graph { names: vec![], edges: vec![] } }
    fn add(&mut self, name: &str) -> NodeId {
        self.names.push(name.to_string());
        self.edges.push(vec![]);
        NodeId(self.names.len() - 1)
    }
    fn link(&mut self, a: NodeId, b: NodeId) { self.edges[a.0].push(b); self.edges[b.0].push(a); }
    fn neighbours(&self, n: NodeId) -> impl Iterator<Item = &str> {
        self.edges[n.0].iter().map(|id| self.names[id.0].as_str())
    }
}

fn main() {
    let mut g = Graph::new();
    let (a, b, c) = (g.add("delhi"), g.add("mumbai"), g.add("pune"));
    g.link(a, b);
    g.link(b, c);
    g.link(c, a);                                      // a cycle: no problem with indices
    for n in [a, b, c] {
        println!("{}: {:?}", g.names[n.0], g.neighbours(n).collect::<Vec<_>>());
    }
}
```

```console
$ rustc --edition 2024 arena.rs && ./arena
delhi: ["mumbai", "pune"]
mumbai: ["delhi", "pune"]
pune: ["mumbai", "delhi"]
```

The cost: an index can outlive the item it named (if you remove items), which is a logic bug rather than memory unsafety. Crates like `slotmap` add generation counters to detect that.

### Mastery check

1. **Predict.** Does this compile? `let mut v = vec![1]; let r = &v; v.push(2); println!("{}", v.len());`
2. **Predict.** Does this compile? `let mut s = String::new(); let a = &mut s; let b = &mut s; b.push('x'); a.push('y');`

## Chapter 2 Project — `concord`: a zero-copy concordance

**Goal:** read a text and build a **concordance** — every word, how often it occurs, and the line numbers where it occurs — without copying any word: the index stores `&str` slices that borrow from the text. Print the most frequent words and a keyword-in-context view for one word. Then show what the borrow checker does when the text would be freed too early.

```rust
// file: concord/concord.rs
use std::collections::BTreeMap;

/// One occurrence of a word: borrows its text from the document.
struct Occurrence<'t> {
    line_no: usize,
    line: &'t str,
}

/// The index borrows from the document text, so it cannot outlive it.
struct Concordance<'t> {
    entries: BTreeMap<String, Vec<Occurrence<'t>>>,   // key: the lowercased word
    total_words: usize,
}

impl<'t> Concordance<'t> {
    fn build(text: &'t str) -> Concordance<'t> {
        let mut entries: BTreeMap<String, Vec<Occurrence<'t>>> = BTreeMap::new();
        let mut total_words = 0;
        for (i, line) in text.lines().enumerate() {
            for raw in line.split(|c: char| !c.is_alphanumeric() && c != '\'') {
                let word = raw.trim_matches('\'');
                if word.is_empty() {
                    continue;
                }
                total_words += 1;
                entries.entry(word.to_lowercase()).or_default().push(Occurrence { line_no: i + 1, line });
            }
        }
        Concordance { entries, total_words }
    }

    fn most_frequent(&self, n: usize) -> Vec<(&str, usize)> {
        let mut counts: Vec<(&str, usize)> = self.entries.iter().map(|(w, occ)| (w.as_str(), occ.len())).collect();
        counts.sort_by(|a, b| b.1.cmp(&a.1).then(a.0.cmp(b.0)));     // by count, then alphabetically
        counts.truncate(n);
        counts
    }

    fn in_context(&self, word: &str) -> &[Occurrence<'t>] {
        self.entries.get(&word.to_lowercase()).map(Vec::as_slice).unwrap_or(&[])
    }
}

fn main() {
    let path = std::env::args().nth(1).expect("usage: concord FILE [WORD]");
    let text = std::fs::read_to_string(&path).expect("readable UTF-8 file");
    let index = Concordance::build(&text);
    println!("{} words, {} distinct", index.total_words, index.entries.len());
    for (word, count) in index.most_frequent(8) {
        println!("{count:>4}  {word}");
    }
    if let Some(word) = std::env::args().nth(2) {
        println!("--- {word} ---");
        for occ in index.in_context(&word) {
            println!("{:>4}: {}", occ.line_no, occ.line.trim());
        }
    }
}
```

The keys are lowercased copies (lowercasing must produce new text), but every `Occurrence` holds a `&str` pointing into the original document: the lines are never copied, however many times a word appears. The lifetime parameter `'t` ties the index to the text it borrows from.

```text
<!-- file: concord/ownership.txt -->
Each value in Rust has an owner.
There can only be one owner at a time.
When the owner goes out of scope, the value will be dropped.
A reference lets you use a value without taking ownership of it.
At any given time, you can have either one mutable reference
or any number of immutable references,
and references must always be valid.
The owner decides when the value is dropped; the borrower does not.
```

```console
$ cd concord && rustc --edition 2024 -O concord.rs && ./concord ownership.txt owner
75 words, 49 distinct
   5  the
   4  owner
   4  value
   3  a
   3  be
   3  of
   2  any
   2  at
--- owner ---
   1: Each value in Rust has an owner.
   2: There can only be one owner at a time.
   3: When the owner goes out of scope, the value will be dropped.
   8: The owner decides when the value is dropped; the borrower does not.
$ cd concord && ./concord ownership.txt REFERENCE | tail -n 3
--- REFERENCE ---
   4: A reference lets you use a value without taking ownership of it.
   5: At any given time, you can have either one mutable reference
```

Now the mistake the lifetime prevents. A function that builds the index from a file it reads itself would have to return an index that borrows from a local `String`:

```rust
// file: concord/broken.rs
use std::collections::BTreeMap;

struct Index<'t> {
    lines: BTreeMap<usize, &'t str>,
}

fn load(path: &str) -> Index<'_> {
    let text = std::fs::read_to_string(path).unwrap();
    let lines = text.lines().enumerate().collect();
    Index { lines }                                 // borrows `text`, which is dropped here
}

fn main() {
    let index = load("ownership.txt");
    println!("{}", index.lines.len());
}
```

```console
$ cd concord && rustc --edition 2024 broken.rs 2>&1 | grep -E '^error|returns a value referencing|`text` is borrowed|^ +--> '
error[E0515]: cannot return value referencing local variable `text`
  --> broken.rs:11:5
   |                 ---- `text` is borrowed here
   |     ^^^^^^^^^^^^^^^ returns a value referencing data owned by the current function
error: aborting due to 1 previous error
```

In C++ the equivalent compiles, and returns an index full of `string_view`s into freed memory (C++ book, DT-35). Here the compiler refuses. By the elision rules, `Index<'_>` borrows from `path` — the only reference parameter — but the lines actually borrow from `text`, a local that is dropped when `load` returns, and the compiler says exactly that. The design fixes are the ones from Unit 2.7: have the caller own the text and pass `&str` in (as `concord.rs` does), or make the index own its data (`String` instead of `&str`) and pay for the copies.

### Review checklist

- Which data does `Concordance` own, and which does it borrow? What would change if `Occurrence` stored `String` instead of `&str`?
- Why does `most_frequent` return `Vec<(&str, usize)>` without a lifetime annotation? Which elision rule applies?
- Why does `in_context` return `&[Occurrence<'t>]` and not `Vec<Occurrence<'t>>`?

## Answers to the Chapter 2 prediction questions

**2.7 Q1.** Yes. `r` is never used after it is created, so its borrow ends immediately (non-lexical lifetimes), and `v.push(2)` can take `&mut v`. It would not compile if `r` were used after the `push`.

**2.7 Q2.** No: error E0499, "cannot borrow `s` as mutable more than once at a time". `a` is used after `b` is created (`a.push('y')`), so the two exclusive borrows overlap. Moving `a.push('y')` before `let b = Swapping the last two statements (using `a` before creating `b`) makes it compile.mut s;` makes it compile.

# Chapter 3 — Structs, Enums, and Pattern Matching

*Goal: model data with structs and enums, attach behaviour with methods, derive standard traits, and take data apart with exhaustive pattern matching — so that invalid states cannot be represented and the compiler checks that every case is handled.*

**What you will be able to do after this chapter**

- Define structs (named, tuple, and unit), and write methods and associated functions.
- Derive `Debug`, `Clone`, `Copy`, `PartialEq`, `Eq`, `Hash`, `Default`, `PartialOrd`, and `Ord`, knowing what each derived implementation does.
- Define enums whose variants carry data, and use `Option` instead of null.
- Write `match` expressions with every kind of pattern, and use `if let`, `let else`, and `while let`.
- Design types that make invalid states unrepresentable.

## Topic 3A — Structs

## Unit 3.1 — Defining and using structs

**One-line summary:** A **struct** groups named fields. **Tuple structs** have unnamed fields (`struct Meters(f64);`) and **unit structs** have none. Construct with `Name { field: value, .. }`; the **field init shorthand** writes `Name { x }` for `Name { x: x }`; **struct update syntax** `Name { a: 1, ..other }` takes the remaining fields from another value.

```rust
// file: structs.rs
#[derive(Debug)]
struct User {
    name: String,
    email: String,
    active: bool,
    logins: u64,
}

#[derive(Debug)]
struct Rgb(u8, u8, u8);          // tuple struct

#[derive(Debug)]
struct Marker;                   // unit struct: no data, size 0

fn new_user(name: &str, email: &str) -> User {
    User { name: name.to_string(), email: email.to_string(), active: true, logins: 0 }
}

fn main() {
    let mut ada = new_user("Ada", "ada@example.com");
    ada.logins += 1;
    let email = String::from("grace@example.com");
    let grace = User { email, name: String::from("Grace"), ..ada };   // shorthand + update syntax
    println!("{grace:?}");
    println!("ada still has: active={} logins={}", ada.active, ada.logins);
    println!("ada is still whole: {} <{}>", ada.name, ada.email);   // name and email were not taken
    let teal = Rgb(0, 128, 128);
    println!("{teal:?} r={} g={} b={} {:?} size_of Marker = {}", teal.0, teal.1, teal.2, Marker, size_of::<Marker>());
}
```

```console
$ rustc --edition 2024 structs.rs && ./structs
User { name: "Grace", email: "grace@example.com", active: true, logins: 1 }
ada still has: active=true logins=1
ada is still whole: Ada <ada@example.com>
Rgb(0, 128, 128) r=0 g=128 b=128 Marker size_of Marker = 0
```

### Deep & Tricky

**DT-12. Struct update syntax moves the fields it copies — which can make the source partially moved.** *(Guarantee: Rust language)* `..ada` takes every field not listed explicitly from `ada`. Fields of `Copy` types are copied; fields like `String` are *moved*, leaving `ada` **partially moved**: its other fields remain usable, but `ada` as a whole cannot be used or borrowed. In the program above, `grace` lists `name` and `email` explicitly, so only `active` and `logins` (both `Copy`) come from `ada`, and `ada` is still whole. Had `name` been taken from `..ada`, `ada.name` and `ada` itself would be unusable.

<!-- expanded -->
### Going deeper: the newtype pattern

A tuple struct with one field — `struct Meters(f64);` — is a **newtype**: a distinct type with the same representation as its field and zero run-time cost. It prevents mixing up values that share a primitive type, and it lets you implement traits on a foreign type (getting around the orphan rule, DT-19).

```rust
// file: newtype.rs
#[derive(Debug, Clone, Copy, PartialEq, PartialOrd)]
struct Meters(f64);
#[derive(Debug, Clone, Copy, PartialEq, PartialOrd)]
struct Feet(f64);

impl From<Feet> for Meters { fn from(f: Feet) -> Meters { Meters(f.0 * 0.3048) } }

fn clearance_ok(height: Meters) -> bool { height >= Meters(2.5) }

#[allow(dead_code)]
struct UserId(u64);
struct OrderId(u64);
fn cancel(_user: UserId, order: OrderId) -> u64 { order.0 }

fn main() {
    let door = Feet(9.0);
    // clearance_ok(door);                         // error: expected Meters, found Feet
    println!("9 ft = {:?}, ok: {}", Meters::from(door), clearance_ok(door.into()));
    println!("cancelled order {}", cancel(UserId(7), OrderId(42)));
    // cancel(OrderId(42), UserId(7));             // error: arguments swapped
    println!("size of Meters = {}", std::mem::size_of::<Meters>());
}
```

```console
$ rustc --edition 2024 newtype.rs && ./newtype
9 ft = Meters(2.7432000000000003), ok: true
cancelled order 42
size of Meters = 8
```

## Unit 3.2 — Methods and associated functions

**One-line summary:** An `impl` block attaches functions to a type. **Methods** take `self` in one of three forms — `&self` (read), `&mut self` (modify), or `self` (consume) — and are called with dot syntax. Functions without `self` are **associated functions**, called as `Type::name()`; `new` is the conventional constructor. `Self` names the type inside its `impl`.

```rust
// file: methods.rs
#[derive(Debug, Clone, PartialEq)]
struct Rect {
    w: f64,
    h: f64,
}

impl Rect {
    fn new(w: f64, h: f64) -> Self {            // associated function (constructor)
        Self { w, h }
    }
    fn square(side: f64) -> Self {
        Self::new(side, side)
    }
    fn area(&self) -> f64 {                     // borrows immutably
        self.w * self.h
    }
    fn scale(&mut self, k: f64) {               // borrows mutably
        self.w *= k;
        self.h *= k;
    }
    fn can_hold(&self, other: &Rect) -> bool {
        self.w >= other.w && self.h >= other.h
    }
    fn into_square(self) -> Rect {              // consumes self
        let side = self.w.max(self.h);
        Rect::square(side)
    }
}

fn main() {
    let mut r = Rect::new(3.0, 4.0);
    println!("area {}", r.area());              // auto-borrow: (&r).area()
    r.scale(2.0);                               // auto-borrow: (&mut r).scale(2.0)
    println!("{r:?} holds a 5x5 square: {}", r.can_hold(&Rect::square(5.0)));
    let s = r.into_square();                    // r is moved
    println!("{s:?}");
}
```

```console
$ rustc --edition 2024 methods.rs && ./methods
area 12
Rect { w: 6.0, h: 8.0 } holds a 5x5 square: true
Rect { w: 8.0, h: 8.0 }
```

Method calls automatically add `&`, `&mut`, or `*` to match the method's receiver, so `r.area()` and `r.scale(2.0)` need no explicit borrows. A method that takes `self` by value consumes the object — a natural way to express "this operation finishes the object", such as converting a builder into the thing it builds.

<!-- expanded -->
### Going deeper: the builder pattern

Rust has no default arguments and no named arguments. A struct with many optional settings is usually built with a **builder**: a separate type with a method per setting, each taking `self` by value and returning it, so calls chain, and a final `build()` that checks the combination.

```rust
// file: builder.rs
#[allow(dead_code)]                    // the fields are only read through Debug here
#[derive(Debug)]
struct Request { url: String, method: String, timeout_ms: u64, retries: u32, headers: Vec<(String, String)> }

struct RequestBuilder { url: String, method: String, timeout_ms: u64, retries: u32, headers: Vec<(String, String)> }

impl RequestBuilder {
    fn new(url: &str) -> Self {
        RequestBuilder { url: url.into(), method: "GET".into(), timeout_ms: 30_000, retries: 0, headers: vec![] }
    }
    fn method(mut self, m: &str) -> Self { self.method = m.into(); self }
    fn timeout_ms(mut self, t: u64) -> Self { self.timeout_ms = t; self }
    fn retries(mut self, r: u32) -> Self { self.retries = r; self }
    fn header(mut self, k: &str, v: &str) -> Self { self.headers.push((k.into(), v.into())); self }
    fn build(self) -> Result<Request, String> {
        if !self.url.starts_with("http") { return Err(format!("bad url {:?}", self.url)); }
        if self.method == "POST" && self.retries > 0 { return Err("refusing to retry a POST".into()); }
        Ok(Request { url: self.url, method: self.method, timeout_ms: self.timeout_ms, retries: self.retries, headers: self.headers })
    }
}

fn main() {
    let r = RequestBuilder::new("https://example.org").timeout_ms(500).retries(2).header("Accept", "text/plain").build();
    println!("{r:#?}");
    println!("{:?}", RequestBuilder::new("https://example.org").method("POST").retries(3).build().unwrap_err());
}
```

```console
$ rustc --edition 2024 builder.rs && ./builder
Ok(
    Request {
        url: "https://example.org",
        method: "GET",
        timeout_ms: 500,
        retries: 2,
        headers: [
            (
                "Accept",
                "text/plain",
            ),
        ],
    },
)
"refusing to retry a POST"
```

For structs whose fields are all public and have sensible defaults, `#[derive(Default)]` plus struct update syntax (`Config { port: 8080, ..Default::default() }`) is the lighter alternative.

## Unit 3.3 — Deriving standard traits

**One-line summary:** `#[derive(...)]` generates implementations of standard traits from the type's fields: `Debug` (`{:?}` printing), `Clone` (`.clone()`), `Copy` (implicit copies; requires all fields `Copy`), `PartialEq`/`Eq` (`==`), `PartialOrd`/`Ord` (`<`, sorting), `Hash` (use as a `HashMap` key), and `Default` (`Type::default()`). Derived comparisons work **field by field, in declaration order**.

```rust
// file: derive.rs
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash, Default)]
struct Version {
    major: u32,
    minor: u32,
    patch: u32,
}

#[derive(Debug, PartialEq, Eq, PartialOrd, Ord)]
enum Level {
    Low,                                        // variants compare in declaration order
    Medium,
    High,
}

fn main() {
    let mut versions = vec![
        Version { major: 1, minor: 10, patch: 0 },
        Version { major: 1, minor: 2, patch: 7 },
        Version { major: 0, minor: 99, patch: 1 },
    ];
    versions.sort();                            // derived Ord: major, then minor, then patch
    println!("{versions:?}");
    let v = versions[0];                        // Copy: versions still usable
    println!("default {:?}, equal: {}, max: {:?}", Version::default(), v == versions[0], versions.iter().max());
    println!("{:?} < {:?} < {:?}: {}", Level::Low, Level::Medium, Level::High, Level::Low < Level::Medium && Level::Medium < Level::High);
}
```

```console
$ rustc --edition 2024 derive.rs && ./derive
[Version { major: 0, minor: 99, patch: 1 }, Version { major: 1, minor: 2, patch: 7 }, Version { major: 1, minor: 10, patch: 0 }]
default Version { major: 0, minor: 0, patch: 0 }, equal: true, max: Some(Version { major: 1, minor: 10, patch: 0 })
Low < Medium < High: true
```

### Deep & Tricky

**DT-13. Derived `PartialOrd`/`Ord` compare fields in declaration order, so reordering fields silently changes sorting.** *(Guarantee: Rust standard library, derive semantics)* The derived comparison is lexicographic over the fields as written. Moving `patch` above `major` in `Version` would still compile, and would sort versions by patch number first. Likewise enum variants compare by their position in the declaration. When the order is part of the meaning, either document it on the type or implement `Ord` by hand — and if you implement `Ord`, implement `PartialEq`, `Eq`, and `PartialOrd` consistently with it.

<!-- expanded -->
### Going deeper: `PartialEq` versus `Eq`, and why it matters for `HashMap`

`PartialEq` gives `==`. `Eq` is a *marker* promising that `==` is a true equivalence — in particular that every value equals itself. `f64` is only `PartialEq` because `NaN != NaN`. `HashMap` keys need `Eq + Hash`, so a struct with an `f64` field cannot derive `Eq` and cannot be a key — the compiler stops you before you build a map in which a `NaN` key could never be found again.

```rust
// file: eqhash.rs
use std::collections::HashMap;

#[derive(Debug, PartialEq, Eq, Hash)]
struct Cell { row: i32, col: i32 }

#[derive(Debug, PartialEq)]
struct Reading { sensor: u32, value: f64 }

fn main() {
    let mut grid = HashMap::new();
    grid.insert(Cell { row: 1, col: 2 }, "wall");
    println!("{:?}", grid.get(&Cell { row: 1, col: 2 }));

    let nan = f64::NAN;
    println!("NaN == NaN: {}", nan == nan);
    let r = Reading { sensor: 1, value: nan };
    println!("reading equals itself: {}", r == r);
    // let mut m = HashMap::new(); m.insert(r, 1);   // error: Reading does not implement Eq or Hash
    let mut by_bits = HashMap::new();                 // a common workaround: key on the bit pattern
    by_bits.insert(nan.to_bits(), "nan bucket");
    println!("{:?}", by_bits.get(&f64::NAN.to_bits()));
}
```

```console
$ rustc --edition 2024 eqhash.rs 2>&1 | grep -m1 -oE "warning: .*"; ./eqhash
Some("wall")
NaN == NaN: false
reading equals itself: false
Some("nan bucket")
```

(A `Reading` compared with itself is `false` — exactly the property `Eq` would forbid.) If you implement `Hash` by hand, it must agree with `Eq`: equal values must have equal hashes, or lookups fail silently.

## Topic 3B — Enums and pattern matching

## Unit 3.4 — Enums with data, and `Option`

**One-line summary:** An **enum** is a type that is exactly one of several **variants**, each of which can carry its own data — unit-like, tuple-like, or struct-like. Rust has no null: a value that may be absent has type `Option<T>`, which is `Some(T)` or `None`, and the compiler forces you to handle `None` before using the value.

```rust
// file: enums.rs
#[derive(Debug)]
enum Shape {
    Circle { r: f64 },
    Rect { w: f64, h: f64 },
    Triangle(f64, f64, f64),                    // three sides
    Empty,
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle { r } => std::f64::consts::PI * r * r,
            Shape::Rect { w, h } => w * h,
            Shape::Triangle(a, b, c) => {
                let s = (a + b + c) / 2.0;      // Heron's formula
                (s * (s - a) * (s - b) * (s - c)).sqrt()
            }
            Shape::Empty => 0.0,
        }
    }
}

fn find_big(shapes: &[Shape], min_area: f64) -> Option<&Shape> {
    for s in shapes {
        if s.area() >= min_area {
            return Some(s);
        }
    }
    None
}

fn main() {
    let shapes = vec![Shape::Empty, Shape::Triangle(3.0, 4.0, 5.0), Shape::Circle { r: 2.0 }, Shape::Rect { w: 2.0, h: 8.0 }];
    for s in &shapes {
        println!("{:<40} area {:>6.2}", format!("{s:?}"), s.area());
    }
    println!("first with area >= 10: {:?}", find_big(&shapes, 10.0));
    println!("first with area >= 100: {:?}", find_big(&shapes, 100.0));
    println!("sizes: Shape {}, Option<&Shape> {}, &Shape {}, Option<u32> {}, Option<Box<u8>> {}",
             size_of::<Shape>(), size_of::<Option<&Shape>>(), size_of::<&Shape>(), size_of::<Option<u32>>(), size_of::<Option<Box<u8>>>());
}
```

```console
$ rustc --edition 2024 enums.rs && ./enums
Empty                                    area   0.00
Triangle(3.0, 4.0, 5.0)                  area   6.00
Circle { r: 2.0 }                        area  12.57
Rect { w: 2.0, h: 8.0 }                  area  16.00
first with area >= 10: Some(Circle { r: 2.0 })
first with area >= 100: None
sizes: Shape 32, Option<&Shape> 8, &Shape 8, Option<u32> 8, Option<Box<u8>> 8
```

An enum value is as large as its largest variant plus a **tag** saying which variant it is. But `Option<&Shape>` is the same size as `&Shape`: references can never be null, so the compiler uses the null value to represent `None` — no tag needed.

### Deep & Tricky

**DT-14. `Option<&T>`, `Option<Box<T>>`, and `Option<NonZeroU32>` cost nothing extra: the compiler stores `None` in an impossible value (a "niche").** *(Guarantee: Rust language for `Option` of references, `Box`, `NonNull`, function pointers, and `NonZero*`; Compiler behaviour for other niche layouts)* A reference is never null, so `None` is the null pointer — the same bit pattern C uses, with none of the risk, because the type system forces a check. This is guaranteed (FFI relies on it: `Option<&T>` and `Option<extern "C" fn()>` are valid as nullable C pointers). `Option<u32>` has no impossible value and needs a separate tag, making it 8 bytes; `Option<NonZeroU32>` is 4.

## Unit 3.5 — `match` and patterns

**One-line summary:** `match` compares a value against patterns in order and runs the first arm that matches. It must be **exhaustive** — every possible value covered — which the compiler checks. Patterns can be literals, ranges (`1..=9`), alternatives (`a | b`), bindings (`x`, `name @ pattern`), wildcards (`_`, `..`), destructuring of tuples, structs, enums, slices, and references, plus **guards** (`if condition`).

```rust
// file: patterns.rs
#[derive(Debug)]
enum Event {
    Key { code: u32, shift: bool },
    Click { x: i32, y: i32 },
    Scroll(i32),
    Quit,
}

fn describe(e: &Event) -> String {
    match e {
        Event::Key { code: 27, .. } => "escape".to_string(),
        Event::Key { code: c @ 65..=90, shift: true } => format!("capital {}", char::from_u32(*c).unwrap()),
        Event::Key { code, .. } => format!("key {code}"),
        Event::Click { x: 0, y: 0 } => "click at origin".to_string(),
        Event::Click { x, y } if x == y => format!("click on the diagonal at {x}"),
        Event::Click { x, y } => format!("click at ({x}, {y})"),
        Event::Scroll(d) if *d < 0 => format!("scroll up {}", -d),
        Event::Scroll(d) => format!("scroll down {d}"),
        Event::Quit => "quit".to_string(),
    }
}

fn classify(values: &[i32]) -> &'static str {
    match values {
        [] => "empty",
        [_] => "one element",
        [first, .., last] if first == last => "same at both ends",
        [0, ..] => "starts with zero",
        [.., 0] => "ends with zero",
        _ => "something else",
    }
}

fn main() {
    let events = [Event::Key { code: 27, shift: false }, Event::Key { code: 72, shift: true }, Event::Key { code: 72, shift: false },
                  Event::Click { x: 0, y: 0 }, Event::Click { x: 5, y: 5 }, Event::Click { x: 1, y: 2 },
                  Event::Scroll(-3), Event::Scroll(2), Event::Quit];
    for e in &events {
        println!("{:<36} -> {}", format!("{e:?}"), describe(e));
    }
    for v in [&[][..], &[4], &[7, 1, 7], &[0, 3], &[3, 0], &[1, 2]] {
        println!("{v:?}: {}", classify(v));
    }
    let n = 42;
    let size = match n {
        0 => "zero",
        1..=9 => "small",
        10 | 20 | 30 => "round",
        _ if n % 2 == 0 => "even and big",
        _ => "odd and big",
    };
    println!("{n} is {size}");
}
```

```console
$ rustc --edition 2024 patterns.rs && ./patterns
Key { code: 27, shift: false }       -> escape
Key { code: 72, shift: true }        -> capital H
Key { code: 72, shift: false }       -> key 72
Click { x: 0, y: 0 }                 -> click at origin
Click { x: 5, y: 5 }                 -> click on the diagonal at 5
Click { x: 1, y: 2 }                 -> click at (1, 2)
Scroll(-3)                           -> scroll up 3
Scroll(2)                            -> scroll down 2
Quit                                 -> quit
[]: empty
[4]: one element
[7, 1, 7]: same at both ends
[0, 3]: starts with zero
[3, 0]: ends with zero
[1, 2]: something else
42 is even and big
```

Remove the `Event::Quit` arm, and the program stops compiling:

```console
$ sed '/Event::Quit => "quit"/d' patterns.rs > patterns_missing.rs && rustc --edition 2024 patterns_missing.rs 2>&1 | head -n 9
error[E0004]: non-exhaustive patterns: `&Event::Quit` not covered
  --> patterns_missing.rs:11:11
   |
11 |     match e {
   |           ^ pattern `&Event::Quit` not covered
   |
note: `Event` defined here
  --> patterns_missing.rs:3:6
   |
```

Exhaustiveness is what makes enums safe to evolve: add a variant, and every `match` that does not handle it becomes a compile error pointing at the exact place to update. (A `_ =>` catch-all arm gives that up; use it deliberately.)

<!-- expanded -->
### Going deeper: binding, guards, ranges, and slice patterns

Patterns can test and extract at the same time: `@` binds a name to a value while matching it against a sub-pattern, `if` guards add conditions, `..=` matches ranges, and slice patterns match on the shape of a slice — first element, last element, everything in between.

```rust
// file: patterns2.rs
fn describe(xs: &[i32]) -> String {
    match xs {
        [] => "empty".into(),
        [one] => format!("just {one}"),
        [first, .., last] if first == last => format!("starts and ends with {first}"),
        [first, rest @ ..] => format!("{first} then {} more", rest.len()),
    }
}

fn grade(score: u32) -> &'static str {
    match score {
        90..=100 => "A",
        75..90 => "B",
        n @ 50..75 if n % 2 == 0 => "C (even)",
        50..75 => "C",
        0..50 => "F",
        _ => "invalid",
    }
}

fn main() {
    for xs in [&[][..], &[7], &[3, 1, 3], &[1, 2, 3, 4]] { println!("{}", describe(xs)); }
    for s in [95, 80, 62, 63, 10, 150] { print!("{} ", grade(s)); }
    println!();
    let msg = Some(42);
    if let Some(n @ 40..=50) = msg { println!("in range: {n}"); }
}
```

```console
$ rustc --edition 2024 patterns2.rs && ./patterns2
empty
just 7
starts and ends with 3
1 then 3 more
A B C (even) C F invalid 
in range: 42
```

The compiler checks that a `match` is **exhaustive**: removing the `_` arm from `grade` is an error, because `u32` values above 100 would not be covered.

## Unit 3.6 — `if let`, `let else`, `while let`, and `matches!`

**One-line summary:** When only one pattern matters, `if let PATTERN = value { ... } else { ... }` is shorter than `match`. `let PATTERN = value else { return/break/continue/panic };` binds the pattern's variables *in the enclosing scope* or diverges — ideal for early exits. `while let` loops while a pattern matches; `matches!(value, PATTERN)` returns a `bool`.

```rust
// file: iflet.rs
fn parse_kv(line: &str) -> Option<(&str, i64)> {
    let Some((key, value)) = line.split_once('=') else {
        return None;                            // no '=': give up early
    };
    let Ok(number) = value.trim().parse::<i64>() else {
        return None;
    };
    Some((key.trim(), number))
}

fn main() {
    for line in ["width = 80", "height=24", "title: none", "depth = deep"] {
        if let Some((k, v)) = parse_kv(line) {
            println!("{k} -> {v}");
        } else {
            println!("ignored {line:?}");
        }
    }
    let mut stack = vec![1, 2, 3];
    while let Some(top) = stack.pop() {
        print!("{top} ");
    }
    println!();
    let c = 'q';
    println!("lowercase letter: {}", matches!(c, 'a'..='z'));
}
```

```console
$ rustc --edition 2024 iflet.rs && ./iflet
width -> 80
height -> 24
ignored "title: none"
ignored "depth = deep"
3 2 1 
lowercase letter: true
```

### Pattern

**P-3. Make invalid states unrepresentable.** *(Practice: type-driven design)* Instead of a struct of flags and optional fields whose combinations may be inconsistent (`connected: bool`, `session: Option<Session>`, `error: Option<String>`), use an enum whose variants carry exactly the data valid in that state (`Disconnected`, `Connected { session }`, `Failed { error }`). Wrap validated values in **newtypes** (`struct Email(String)` with a constructor that validates) so that unvalidated data cannot reach code that assumes validity. The compiler then enforces the invariants that would otherwise live in comments and tests. *Trade-off:* more types and conversions, in exchange for whole classes of bugs becoming compile errors.

<!-- expanded -->
### Deep & Tricky

**DT-15. A temporary created in the scrutinee of `if let` / `while let` / `match` lives until the end of the whole statement — so `if let Some(x) = mutex.lock().unwrap().get(0) { ... }` holds the lock for the entire `if let` body *and* its `else` branch. Edition 2024 shortened this for `if let` (the temporary is dropped before `else`), but `match` and `while let` keep the old rule.** *(Guarantee: temporary lifetime extension rules, changed for `if let` in Rust 2024 — the `if_let_rescope` change)* Bind the guard to a variable in its own block, or copy what you need out first, when holding a lock.

```rust
// file: tempscope.rs
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(vec![1, 2, 3]);
    match m.lock().unwrap().first() {
        Some(x) => println!("match arm sees {x}; lock still held: {}", m.try_lock().is_err()),
        None => {}
    }
    if let Some(x) = m.lock().unwrap().first() {
        println!("if let body sees {x}; lock still held: {}", m.try_lock().is_err());
    } else {
        unreachable!();
    }
    let first = m.lock().unwrap().first().copied();      // guard dropped at the end of this statement
    println!("copied {first:?}; lock free afterwards: {}", m.try_lock().is_ok());
}
```

```console
$ rustc --edition 2024 tempscope.rs && ./tempscope
match arm sees 1; lock still held: true
if let body sees 1; lock still held: true
copied Some(1); lock free afterwards: true
```

Inside the body, the lock is held in both cases; with edition 2021, an `else` branch that locked the mutex again would deadlock.

### Mastery check

1. **Predict.** What does `match (3, -3) { (x, y) if x + y == 0 => "opposites", (x, _) if x > 0 => "positive first", _ => "other" }` produce?
2. **Predict.** What are `size_of::<Option<bool>>()` and `size_of::<Option<Option<bool>>>()`?

## Chapter 3 Project — A connection state machine

**Goal:** model a simplified TCP-style connection as a state machine. States carry exactly the data meaningful in that state; events are an enum; one `match` over `(state, event)` defines every legal transition; illegal events are rejected without changing the state. A script of events drives it and prints each transition.

```rust
// file: conn/conn.rs
#[derive(Debug, Clone, PartialEq)]
enum State {
    Closed,
    Listening { port: u16 },
    SynReceived { port: u16, peer_seq: u32 },
    Established { port: u16, sent: u64, received: u64 },
    Closing { reason: String },
}

#[derive(Debug, Clone)]
enum Event {
    Listen(u16),
    Syn { seq: u32 },
    Ack { ack: u32 },
    Data(Vec<u8>),
    Send(usize),
    Close,
    Reset(String),
}

#[derive(Debug)]
struct Rejected {
    state: State,
    event: Event,
}

impl State {
    /// Consumes the current state and the event; returns the next state or a rejection.
    fn on(self, event: Event) -> Result<State, Rejected> {
        use Event as E;
        use State as S;
        match (self, event) {
            (S::Closed, E::Listen(port)) if port != 0 => Ok(S::Listening { port }),
            (S::Listening { port }, E::Syn { seq }) => Ok(S::SynReceived { port, peer_seq: seq }),
            (S::SynReceived { port, peer_seq }, E::Ack { ack }) if ack == peer_seq.wrapping_add(1) => {
                Ok(S::Established { port, sent: 0, received: 0 })
            }
            (S::Established { port, sent, received }, E::Data(bytes)) => {
                Ok(S::Established { port, sent, received: received + bytes.len() as u64 })
            }
            (S::Established { port, sent, received }, E::Send(n)) => {
                Ok(S::Established { port, sent: sent + n as u64, received })
            }
            (S::Established { sent, received, .. }, E::Close) => {
                Ok(S::Closing { reason: format!("normal close after {sent} bytes out, {received} in") })
            }
            (S::Closing { .. }, E::Close) => Ok(S::Closed),
            (state, E::Reset(why)) if state != S::Closed => Ok(S::Closing { reason: format!("reset: {why}") }),
            (state, event) => Err(Rejected { state, event }),
        }
    }
}

fn main() {
    let script = vec![
        Event::Syn { seq: 1 },                       // not listening yet: rejected
        Event::Listen(0),                            // port 0 is invalid: rejected by the guard
        Event::Listen(8080),
        Event::Syn { seq: 41 },
        Event::Ack { ack: 7 },                       // wrong ack number: rejected
        Event::Ack { ack: 42 },
        Event::Data(b"GET /".to_vec()),
        Event::Send(512),
        Event::Data(vec![0; 7]),
        Event::Close,
        Event::Close,
        Event::Listen(9090),
        Event::Syn { seq: u32::MAX },
        Event::Ack { ack: 0 },                       // u32::MAX + 1 wraps to 0
        Event::Reset("peer vanished".to_string()),
        Event::Close,
        Event::Reset("too late".to_string()),        // reset while closed: rejected
    ];
    let mut state = State::Closed;
    for event in script {
        let shown = format!("{event:?}");
        match state.on(event) {
            Ok(next) => {
                println!("{shown:<40} => {next:?}");
                state = next;
            }
            Err(Rejected { state: unchanged, event }) => {
                println!("{:<40} !! rejected in {unchanged:?}", format!("{event:?}"));   // the event comes back too
                state = unchanged;
            }
        }
    }
}
```

```console
$ cd conn && rustc --edition 2024 conn.rs && ./conn
Syn { seq: 1 }                           !! rejected in Closed
Listen(0)                                !! rejected in Closed
Listen(8080)                             => Listening { port: 8080 }
Syn { seq: 41 }                          => SynReceived { port: 8080, peer_seq: 41 }
Ack { ack: 7 }                           !! rejected in SynReceived { port: 8080, peer_seq: 41 }
Ack { ack: 42 }                          => Established { port: 8080, sent: 0, received: 0 }
Data([71, 69, 84, 32, 47])               => Established { port: 8080, sent: 0, received: 5 }
Send(512)                                => Established { port: 8080, sent: 512, received: 5 }
Data([0, 0, 0, 0, 0, 0, 0])              => Established { port: 8080, sent: 512, received: 12 }
Close                                    => Closing { reason: "normal close after 512 bytes out, 12 in" }
Close                                    => Closed
Listen(9090)                             => Listening { port: 9090 }
Syn { seq: 4294967295 }                  => SynReceived { port: 9090, peer_seq: 4294967295 }
Ack { ack: 0 }                           => Established { port: 9090, sent: 0, received: 0 }
Reset("peer vanished")                   => Closing { reason: "reset: peer vanished" }
Close                                    => Closed
Reset("too late")                        !! rejected in Closed
```

Points to notice:

- **Each state holds only what is valid in it.** There is no `port` in `Closed` and no byte counters before `Established`, so code cannot read a port that does not exist or count bytes on a connection that is not open.
- **`on` takes `self` by value and returns the next state.** The old state is consumed, so there is no way to keep using a stale state after a transition. A rejected event hands the unchanged state back inside `Rejected`.
- **One `match` on the pair `(state, event)` is the whole transition table**, with guards for conditions (`port != 0`, the ack number) and a final catch-all arm that turns every other combination into a rejection. Remove that last arm and the compiler lists the unhandled combinations.
- **`wrapping_add`** makes the sequence-number arithmetic explicit (DT-2): `u32::MAX + 1` wraps to 0, as TCP sequence numbers do.

### Review checklist

- Add a `TimeWait { until_ms: u64 }` state between `Closing` and `Closed`. Which arms must change, and does the compiler help you find them?
- Why does the reset arm need the guard `state != S::Closed`, and what does that require of `State` (which derive)?
- Could `Event::Send` carry a `Vec<u8>` instead of a count? What would that cost per event?

## Answers to the Chapter 3 prediction questions

**3.6 Q1.** `"opposites"`. Arms are tried in order; the first guard (`3 + -3 == 0`) is true.

**3.6 Q2.** Both are 1. `bool` uses only the values 0 and 1, so `Option<bool>` stores `None` as 2, and `Option<Option<bool>>` uses 3 for its outer `None` — niches can nest (DT-14).

# Chapter 4 — Error Handling

*Goal: handle failure the Rust way — `panic!` for bugs, `Result` for expected failures — propagate errors with `?`, design error types for libraries, use `anyhow` and `thiserror` the way the ecosystem does, and give every error enough context to act on.*

Rust has no exceptions. A function that can fail says so in its return type, `Result<T, E>`, and the caller must deal with the error — handle it, convert it, or pass it up with the `?` operator. Bugs (violated invariants, impossible states) **panic** instead, which unwinds the thread or aborts the process. Keeping the two apart is the core skill of this chapter.

**What you will be able to do after this chapter**

- Decide between panicking and returning an error, and use `unwrap` and `expect` deliberately.
- Transform and combine `Result` and `Option` values with combinators and the `?` operator.
- Define error types that implement `std::error::Error`, with `From` conversions and source chains.
- Use `Box<dyn Error>`, `anyhow`, and `thiserror`, and add context to errors.
- Return errors from `main` and choose exit codes.

## Topic 4A — Panics and results

## Unit 4.1 — `panic!`: unrecoverable errors

**One-line summary:** `panic!` signals a bug: it prints a message and, by default, **unwinds** the current thread — running destructors (`Drop`) of every live value on the way up — then ends the thread (the whole program, if it is the main thread). Indexing out of bounds, `unwrap` on `None`, integer overflow in debug builds, and division by zero all panic. With `panic = "abort"` in the profile, the process aborts immediately instead.

```rust
// file: panic.rs
struct Guard(&'static str);
impl Drop for Guard {
    fn drop(&mut self) {
        println!("drop {}", self.0);
    }
}

fn average(values: &[i32]) -> i32 {
    let _g = Guard("inner");
    values.iter().sum::<i32>() / values.len() as i32       // divide by zero if empty
}

fn main() {
    let _outer = Guard("outer");
    println!("average of [2, 4] = {}", average(&[2, 4]));
    let result = std::panic::catch_unwind(|| average(&[]));
    println!("caught a panic: {}", result.is_err());
    let config: Option<&str> = None;
    let value = config.expect("config must be loaded before use");
    println!("{value}");
}
```

```console
$ rustc --edition 2024 panic.rs && ./panic; echo "exit status $?"
drop inner
average of [2, 4] = 3

thread 'main' (1539) panicked at panic.rs:11:5:
attempt to divide by zero
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: core::panicking::panic_const::panic_const_div_by_zero
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:175:17
   3: panic::average
   4: panic::main::{closure#0}
   5: std::panicking::catch_unwind::do_call::<panic::main::{closure#0}, i32>
   6: __rust_try
   7: std::panic::catch_unwind::<panic::main::{closure#0}, i32>
   8: panic::main
   9: <fn() as core::ops::function::FnOnce<()>>::call_once
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
drop inner
caught a panic: true

thread 'main' (1539) panicked at panic.rs:20:24:
config must be loaded before use
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: core::panicking::panic_display::<&str>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:259:5
   3: core::option::expect_failed
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/option.rs:2257:5
   4: <core::option::Option<&str>>::expect
   5: panic::main
   6: <fn() as core::ops::function::FnOnce<()>>::call_once
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
drop outer
exit status 101
```

The first `drop inner` is an ordinary end of scope; the next two drops ran *during unwinding*, so locks are released and files are closed even when a thread panics. `catch_unwind` exists for boundaries — a thread pool or FFI layer that must not let a panic escape — not for ordinary error handling. A panic that escapes `main` exits with status 101.

**When to panic:** when continuing would be wrong because the program itself is broken — a violated invariant, an index your own code computed wrongly, a "can't happen" case. **When to return `Result`:** when the failure comes from outside — a missing file, bad input, a network timeout — and a caller could reasonably handle it. In tests, examples, and prototypes, `unwrap` and `expect` are fine; in production code, every `unwrap` should be a claim you can justify ("this cannot be `None` because…"), and `expect("…")` is how you write that claim down.

<!-- expanded -->
### Going deeper: catching a panic at a boundary

`std::panic::catch_unwind` stops a panic from unwinding further and turns it into an `Err`. It is not an exception mechanism for ordinary errors; it exists for *boundaries*: a thread pool that must survive one bad job, a test harness, and FFI, where unwinding into C is undefined behaviour (Unit 11.5). It works only when panics unwind — with `panic = "abort"` in `Cargo.toml`, nothing can be caught.

```rust
// file: catch.rs
use std::panic;

fn risky_job(n: u32) -> u32 {
    if n == 3 { panic!("job {n} hit a bug"); }
    n * 10
}

fn main() {
    panic::set_hook(Box::new(|info| eprintln!("[hook] {info}")));     // replaces the default message
    let mut results = vec![];
    for n in 1..=4 {
        match panic::catch_unwind(|| risky_job(n)) {
            Ok(v) => results.push(v),
            Err(payload) => {
                let msg = payload.downcast_ref::<String>().cloned().unwrap_or_default();
                println!("job {n} failed: {msg}");
            }
        }
    }
    println!("results: {results:?}");
}
```

```console
$ rustc --edition 2024 catch.rs && ./catch
[hook] panicked at catch.rs:5:17:
job 3 hit a bug
job 3 failed: job 3 hit a bug
results: [10, 20, 40]
$ rustc --edition 2024 -C panic=abort catch.rs -o catch_abort && ./catch_abort; echo "exit status $?"
[hook] panicked at catch.rs:5:17:
job 3 hit a bug
bash: line 1:  1570 Aborted                 ./catch_abort
exit status 134
```

With `-C panic=abort`, the hook still runs, but then the whole process aborts (status 134): there is nothing to unwind and nothing to catch.

## Unit 4.2 — `Result` and the `?` operator

**One-line summary:** `Result<T, E>` is `Ok(T)` or `Err(E)`. `expr?` evaluates to the `Ok` value, or *returns early* from the function with the `Err` — converting the error with `From::from` on the way. `?` also works on `Option` in functions that return `Option`. Combinators transform results without `match`: `map`, `map_err`, `and_then`, `unwrap_or`, `unwrap_or_else`, `ok_or`, `ok()`.

```rust
// file: question.rs
use std::fs;
use std::num::ParseIntError;

fn parse_pair(s: &str) -> Result<(i32, i32), ParseIntError> {
    let (a, b) = s.split_once(',').unwrap_or((s, "0"));
    let x = a.trim().parse::<i32>()?;                    // early return on error
    let y = b.trim().parse::<i32>()?;
    Ok((x, y))
}

fn first_even_square(v: &[i32]) -> Option<i32> {
    let first = v.iter().find(|&&x| x % 2 == 0)?;       // ? on Option: return None early
    first.checked_mul(*first)                            // None on overflow
}

fn read_number(path: &str) -> Result<i64, String> {
    let text = fs::read_to_string(path).map_err(|e| format!("cannot read {path}: {e}"))?;
    text.trim().parse::<i64>().map_err(|e| format!("{path}: {e}"))
}

fn main() {
    for input in ["3, 4", "10", "7,x", ""] {
        println!("{input:?} -> {:?}", parse_pair(input));
    }
    println!("{:?} {:?} {:?}", first_even_square(&[3, 5, 6]), first_even_square(&[1, 3]), first_even_square(&[65536]));
    std::fs::write("n.txt", "42\n").unwrap();
    println!("{:?}", read_number("n.txt"));
    println!("{:?}", read_number("missing.txt"));
    let port: u16 = std::env::var("PORT").ok().and_then(|s| s.parse().ok()).unwrap_or(8080);
    let doubled = parse_pair("5,6").map(|(a, b)| a * b).unwrap_or_default();
    println!("port {port}, product {doubled}");
}
```

```console
$ rustc --edition 2024 question.rs && ./question
"3, 4" -> Ok((3, 4))
"10" -> Ok((10, 0))
"7,x" -> Err(ParseIntError { kind: InvalidDigit })
"" -> Err(ParseIntError { kind: Empty })
Some(36) None None
Ok(42)
Err("cannot read missing.txt: No such file or directory (os error 2)")
port 8080, product 30
```

### Deep & Tricky

**DT-16. `?` converts the error with `From::from`, so a function can use `?` on several different error types only if its own error type has a `From` impl for each.** *(Guarantee: Rust language; desugaring of `?`)* `x?` means roughly `match x { Ok(v) => v, Err(e) => return Err(From::from(e)) }`. If the function returns `Result<T, MyError>` and `x` is a `Result<U, io::Error>`, there must be an `impl From<io::Error> for MyError`. With `Box<dyn Error>` or `anyhow::Error` as the return type, a blanket `From` impl accepts any error type, which is why they are convenient in applications. The conversion is also why `?` cannot mix `Option` and `Result` directly — use `ok_or(...)` to turn a `None` into an error.

<!-- expanded -->
### Going deeper: `Option` and `Result` combinators

`?` handles the "return early on failure" case. For other shapes, `Option` and `Result` have methods that avoid nested `match`: `map` (transform the success value), `and_then` (chain a step that can itself fail), `ok_or` / `ok_or_else` (turn `Option` into `Result`), `unwrap_or` / `unwrap_or_else` / `unwrap_or_default` (fallbacks), `map_err` (change the error), and `.ok()` (drop the error).

```rust
// file: combinators.rs
use std::collections::HashMap;

fn port_from(env: &HashMap<&str, &str>) -> Result<u16, String> {
    env.get("PORT")
        .ok_or("PORT is not set")?                                   // Option -> Result
        .parse::<u16>()
        .map_err(|e| format!("PORT is not a number: {e}"))
}

fn main() {
    let mut env = HashMap::new();
    println!("{:?}", port_from(&env));
    env.insert("PORT", "80x");
    println!("{:?}", port_from(&env));
    env.insert("PORT", "8080");
    println!("{:?}", port_from(&env));

    let words = ["10", "x", "30"];
    let parsed: Vec<i32> = words.iter().filter_map(|w| w.parse().ok()).collect();            // skip failures
    let all: Result<Vec<i32>, _> = words.iter().map(|w| w.parse::<i32>()).collect();         // stop at the first
    println!("filter_map: {parsed:?}, collect into Result: {}", all.is_err());
    let doubled = Some(4).map(|x| x * 2).filter(|x| *x > 5).unwrap_or(0);
    println!("doubled: {doubled}, missing name: {}", None::<&str>.map_or("anonymous".len(), |s| s.len()));
}
```

```console
$ rustc --edition 2024 combinators.rs && ./combinators
Err("PORT is not set")
Err("PORT is not a number: invalid digit found in string")
Ok(8080)
filter_map: [10, 30], collect into Result: true
doubled: 8, missing name: 9
```

Collecting an iterator of `Result`s into `Result<Vec<_>, _>` stops at the first error — a very common idiom for "parse all of these or fail".

## Topic 4B — Designing errors

## Unit 4.3 — Defining error types

**One-line summary:** A library's error type is usually an **enum** with one variant per failure kind, carrying the data a caller needs. It implements `Debug`, `Display` (the human message), and `std::error::Error` (with `source()` returning the underlying cause), plus `From` impls for errors it wraps, so that `?` converts automatically.

```rust
// file: errors.rs
use std::error::Error;
use std::fmt;
use std::num::ParseIntError;

#[derive(Debug)]
enum ConfigError {
    Io(std::io::Error),
    MissingKey(&'static str),
    BadNumber { key: String, source: ParseIntError },
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            ConfigError::Io(_) => write!(f, "could not read the configuration"),
            ConfigError::MissingKey(k) => write!(f, "missing required key `{k}`"),
            ConfigError::BadNumber { key, .. } => write!(f, "value of `{key}` is not a valid number"),
        }
    }
}

impl Error for ConfigError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            ConfigError::Io(e) => Some(e),
            ConfigError::BadNumber { source, .. } => Some(source),
            ConfigError::MissingKey(_) => None,
        }
    }
}

impl From<std::io::Error> for ConfigError {
    fn from(e: std::io::Error) -> Self {
        ConfigError::Io(e)
    }
}

fn workers(path: &str) -> Result<u32, ConfigError> {
    let text = std::fs::read_to_string(path)?;          // io::Error -> ConfigError via From
    let line = text.lines().find(|l| l.starts_with("workers=")).ok_or(ConfigError::MissingKey("workers"))?;
    let value = &line["workers=".len()..];
    value.parse().map_err(|source| ConfigError::BadNumber { key: "workers".into(), source })
}

fn report(e: &dyn Error) {
    eprint!("error: {e}");
    let mut cause = e.source();
    while let Some(c) = cause {
        eprint!("\n  caused by: {c}");
        cause = c.source();
    }
    eprintln!();
}

fn main() {
    std::fs::write("good.conf", "name=demo\nworkers=8\n").unwrap();
    std::fs::write("bad.conf", "workers=eight\n").unwrap();
    std::fs::write("empty.conf", "name=demo\n").unwrap();
    for path in ["good.conf", "bad.conf", "empty.conf", "nope.conf"] {
        match workers(path) {
            Ok(n) => println!("{path}: {n} workers"),
            Err(e) => report(&e),
        }
    }
}
```

```console
$ rustc --edition 2024 errors.rs && ./errors 2>&1
good.conf: 8 workers
error: value of `workers` is not a valid number
  caused by: invalid digit found in string
error: missing required key `workers`
error: could not read the configuration
  caused by: No such file or directory (os error 2)
```

The `Display` message describes *this* level's failure; the cause is reachable through `source()`, and a reporter walks the chain. Do not repeat the cause's text in your own message, or chains print every message twice.

<!-- expanded -->
### Going deeper: a complete error type by hand

A library's error type should be an `enum` with one variant per kind of failure the caller might handle differently, implement `Display` (for people) and `std::error::Error` (with `source()` pointing at the underlying cause), and offer `From` conversions so `?` works. Writing one by hand once shows exactly what `thiserror` generates.

```rust
// file: errtype.rs
use std::{error::Error, fmt, num::ParseIntError};

#[derive(Debug)]
enum ConfigError {
    Missing(&'static str),
    BadNumber { key: &'static str, source: ParseIntError },
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ConfigError::Missing(k) => write!(f, "missing key `{k}`"),
            ConfigError::BadNumber { key, .. } => write!(f, "`{key}` is not a valid number"),
        }
    }
}

impl Error for ConfigError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            ConfigError::BadNumber { source, .. } => Some(source),
            _ => None,
        }
    }
}

fn workers(text: Option<&str>) -> Result<u32, ConfigError> {
    let t = text.ok_or(ConfigError::Missing("workers"))?;
    t.parse().map_err(|e| ConfigError::BadNumber { key: "workers", source: e })
}

fn report(e: &dyn Error) {
    print!("error: {e}");
    let mut cause = e.source();
    while let Some(c) = cause { print!("\n  caused by: {c}"); cause = c.source(); }
    println!();
}

fn main() {
    for input in [Some("8"), None, Some("-3")] {
        match workers(input) { Ok(n) => println!("workers = {n}"), Err(e) => report(&e) }
    }
}
```

```console
$ rustc --edition 2024 errtype.rs && ./errtype
workers = 8
error: missing key `workers`
error: `workers` is not a valid number
  caused by: invalid digit found in string
```

The `Display` message does *not* repeat the cause's text; the chain is printed by walking `source()`. This is the convention `anyhow` follows with `{:#}` and its `Caused by:` list — repeating the cause in both places makes it appear twice.

## Unit 4.4 — `Box<dyn Error>`, `anyhow`, and `thiserror`

**One-line summary:** Applications that only report errors can return `Box<dyn Error>` — any error type, boxed. The **`anyhow`** crate improves on it: `anyhow::Result<T>`, `.context("what we were doing")` to add a layer to the chain, `bail!` and `ensure!` for quick errors, and a report format that prints the whole chain. Libraries, whose callers may want to match on the error, define their own types — and **`thiserror`** derives the `Display`, `Error`, `source`, and `From` boilerplate of Unit 4.3.

```toml
# file: anyerr/Cargo.toml
[package]
name = "anyerr"
version = "0.1.0"
edition = "2024"

[dependencies]
anyhow = "1"
thiserror = "2"
```

```rust
// file: anyerr/src/main.rs
use anyhow::{Context, Result, bail, ensure};
use thiserror::Error;

/// A library-style error: callers can match on the variants.
#[derive(Debug, Error)]
pub enum ParseError {
    #[error("line {line}: expected `key=value`")]
    Syntax { line: usize },
    #[error("line {line}: invalid number for `{key}`")]
    Number { line: usize, key: String, #[source] source: std::num::ParseIntError },
}

fn parse_limits(text: &str) -> Result<Vec<(String, u32)>, ParseError> {
    let mut out = Vec::new();
    for (i, line) in text.lines().enumerate().filter(|(_, l)| !l.trim().is_empty()) {
        let (k, v) = line.split_once('=').ok_or(ParseError::Syntax { line: i + 1 })?;
        let n = v.trim().parse().map_err(|source| ParseError::Number { line: i + 1, key: k.trim().into(), source })?;
        out.push((k.trim().to_string(), n));
    }
    Ok(out)
}

/// Application code: anyhow adds context at each level.
fn load(path: &str) -> Result<Vec<(String, u32)>> {
    let text = std::fs::read_to_string(path).with_context(|| format!("reading limits file {path}"))?;
    let limits = parse_limits(&text).with_context(|| format!("parsing {path}"))?;
    ensure!(!limits.is_empty(), "{path} defines no limits");
    if let Some((k, _)) = limits.iter().find(|(_, v)| *v == 0) {
        bail!("limit `{k}` in {path} must be positive");
    }
    Ok(limits)
}

fn main() {
    std::fs::write("ok.txt", "connections = 100\nrequests=5000\n").unwrap();
    std::fs::write("typo.txt", "connections = 100\nrequests 5000\n").unwrap();
    std::fs::write("bignum.txt", "connections = 99999999999\n").unwrap();
    std::fs::write("zero.txt", "connections = 0\n").unwrap();
    for path in ["ok.txt", "typo.txt", "bignum.txt", "zero.txt", "absent.txt"] {
        match load(path) {
            Ok(l) => println!("{path}: {l:?}"),
            Err(e) => println!("{path}: {e:#}"),            // {:#} prints the chain on one line
        }
    }
    let e = load("bignum.txt").unwrap_err();
    if let Some(ParseError::Number { line, key, .. }) = e.downcast_ref::<ParseError>() {
        println!("a caller can still match the typed error: line {line}, key {key}");
    }
}
```

```console
$ cd anyerr && cargo run -q
ok.txt: [("connections", 100), ("requests", 5000)]
typo.txt: parsing typo.txt: line 2: expected `key=value`
bignum.txt: parsing bignum.txt: line 1: invalid number for `connections`: number too large to fit in target type
zero.txt: limit `connections` in zero.txt must be positive
absent.txt: reading limits file absent.txt: No such file or directory (os error 2)
a caller can still match the typed error: line 1, key connections
```

### Pattern

**P-4. Libraries return typed errors; applications use `anyhow` with context.** *(Practice: the Rust ecosystem's convention)* A library cannot know how its caller will react, so it exposes an error enum (often derived with `thiserror`) whose variants can be matched. An application mostly reports errors to a human or a log, so it uses `anyhow::Result` and adds `.context(...)` at each layer — "reading limits file x" → "parsing x" → "line 2: expected key=value" — producing messages that say what was being attempted, not just what failed. *Trade-off:* typed errors need maintenance as the library evolves (mark them `#[non_exhaustive]` so adding variants is not a breaking change); `anyhow` errors are easy to create but hard to match on (`downcast_ref` is the escape hatch).

<!-- expanded -->
### Deep & Tricky

**DT-17. `Box<dyn Error>` is not `Send + Sync`, so it cannot be sent across threads or held across an `.await` in a spawned Tokio task; library code that returns `Box<dyn Error>` forces that limit on every caller. Use `Box<dyn Error + Send + Sync + 'static>` (what `anyhow::Error` wraps), or a concrete error enum.** *(Guarantee: auto traits are not implied for trait objects; `dyn Error` has only the bounds you write)* The error appears far from the cause — at `thread::spawn` or `tokio::spawn` — as "`dyn std::error::Error` cannot be sent between threads safely".

```rust
// file: senderr.rs
use std::error::Error;
use std::thread;

type BoxErr = Box<dyn Error + Send + Sync + 'static>;

fn parse(s: &str) -> Result<i32, BoxErr> { Ok(s.trim().parse()?) }

fn main() {
    let h = thread::spawn(|| parse(" 41 ").map(|n| n + 1));     // the Result travels back to main
    println!("{:?}", h.join().unwrap());
    let h = thread::spawn(|| parse("x"));
    println!("{}", h.join().unwrap().unwrap_err());
}
```

```console
$ rustc --edition 2024 senderr.rs && ./senderr
Ok(42)
invalid digit found in string
$ sed 's/dyn Error + Send + Sync + .static/dyn Error/' senderr.rs > senderr_bad.rs && rustc --edition 2024 senderr_bad.rs 2>&1 | grep -m1 -oE "error\[E[0-9]+\]: .*"
error[E0277]: `dyn std::error::Error` cannot be sent between threads safely
```

## Unit 4.5 — Errors from `main`, and exit codes

**One-line summary:** `main` may return `Result<(), E>` where `E: Debug`; an `Err` is printed with `{:?}` and the process exits with status 1. For full control, return `std::process::ExitCode`, or call `std::process::exit(code)` (which does *not* run destructors). With `anyhow`, the `Debug` output of an error from `main` prints the whole chain.

```rust
// file: mainresult.rs
use std::process::ExitCode;

fn run(args: &[String]) -> Result<u32, String> {
    let n: u32 = args.get(1).ok_or("usage: mainresult NUMBER")?.parse().map_err(|e| format!("bad number: {e}"))?;
    if n > 100 {
        return Err(format!("{n} is too large"));
    }
    Ok(n * 2)
}

fn main() -> ExitCode {
    let args: Vec<String> = std::env::args().collect();
    match run(&args) {
        Ok(v) => {
            println!("{v}");
            ExitCode::SUCCESS
        }
        Err(msg) => {
            eprintln!("mainresult: {msg}");
            ExitCode::from(2)                    // conventional "usage error" status
        }
    }
}
```

```console
$ rustc --edition 2024 mainresult.rs && ./mainresult 21; echo "status $?"; ./mainresult; echo "status $?"; ./mainresult 500 2>&1; echo "status $?"
42
status 0
mainresult: usage: mainresult NUMBER
status 2
mainresult: 500 is too large
status 2
```

### Deep & Tricky

**DT-18. A panic while already unwinding aborts the process — so `Drop` implementations must not panic.** *(Guarantee: Rust language, panic runtime)* If a destructor panics during unwinding from another panic, Rust cannot unwind two panics at once and aborts immediately: no further destructors run, and the error message may be lost in the abort. For the same reason, `Drop` cannot report errors (it returns `()`): types such as `BufWriter`, whose drop may fail to flush, ignore the error — call `flush()` (or a `close`-style method) explicitly where the error matters. This mirrors C++'s rule that destructors are `noexcept` (C++ book, Chapter 6).

<!-- expanded -->
### Going deeper: `ExitCode` and the `Termination` trait

`main` can return `()`, `Result<(), E>` where `E: Debug`, or `std::process::ExitCode`. Returning `ExitCode` lets you choose the status without calling `std::process::exit`, which skips destructors (buffered output in a `BufWriter` would be lost). The convention from BSD's `sysexits.h`: 0 success, 1 general failure, 2 misuse of the command line, 64–78 specific causes.

```rust
// file: exitcode.rs
use std::io::{BufWriter, Write};
use std::process::ExitCode;

fn main() -> ExitCode {
    let mut out = BufWriter::new(std::io::stdout());
    let args: Vec<String> = std::env::args().skip(1).collect();
    if args.is_empty() {
        eprintln!("usage: exitcode NUMBER...");
        return ExitCode::from(2);
    }
    let mut bad = 0;
    for a in &args {
        match a.parse::<f64>() {
            Ok(x) => writeln!(out, "{a} squared is {}", x * x).unwrap(),
            Err(_) => { eprintln!("not a number: {a}"); bad += 1; }
        }
    }
    if bad > 0 { ExitCode::FAILURE } else { ExitCode::SUCCESS }   // out is flushed when it drops
}
```

```console
$ rustc --edition 2024 exitcode.rs && ./exitcode 3 1.5; echo "status $?"; ./exitcode 2 two; echo "status $?"; ./exitcode; echo "status $?"
3 squared is 9
1.5 squared is 2.25
status 0
not a number: two
2 squared is 4
status 1
usage: exitcode NUMBER...
status 2
$ sed 's/if bad > 0 { ExitCode::FAILURE } else { ExitCode::SUCCESS }   \/\/ out is flushed when it drops/std::process::exit(if bad > 0 { 1 } else { 0 })/' exitcode.rs > exitnow.rs && rustc --edition 2024 exitnow.rs 2>/dev/null && ./exitnow 3 1.5 | wc -l | sed 's/$/ lines printed with process::exit (the BufWriter was never flushed)/'
0 lines printed with process::exit (the BufWriter was never flushed)
```

### Mastery check

1. **Predict.** A function returns `Result<u8, String>` and contains `let n: u8 = "300".parse()?;`. Does it compile?
2. **Predict.** What does `Some(4).filter(|x| x % 2 == 1).map(|x| x * 10).unwrap_or(-1)` evaluate to?

## Chapter 4 Project — A configuration loader with precise errors

**Goal:** a library module that parses an INI-style configuration (`[section]` headers, `key = value` lines, `#` comments) into a typed `ServerConfig`, reporting **every** problem it finds — not just the first — with file name, line, and column; and a binary that loads a file, prints the configuration or a numbered list of errors, and sets the exit status.

```toml
# file: confload/Cargo.toml
[package]
name = "confload"
version = "0.1.0"
edition = "2024"

[dependencies]
thiserror = "2"
anyhow = "1"
```

```rust
// file: confload/src/lib.rs
use std::collections::HashMap;
use std::fmt;
use thiserror::Error;

#[derive(Debug, Clone, PartialEq)]
pub struct ServerConfig {
    pub host: String,
    pub port: u16,
    pub workers: u32,
    pub log_level: LogLevel,
    pub timeout_ms: u64,
}

#[derive(Debug, Clone, Copy, PartialEq)]
pub enum LogLevel { Error, Warn, Info, Debug }

/// Where a problem is.
#[derive(Debug, Clone, PartialEq)]
pub struct Span { pub line: usize, pub column: usize }

impl fmt::Display for Span {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{}:{}", self.line, self.column)
    }
}

#[derive(Debug, Error, PartialEq)]
#[non_exhaustive]
pub enum ConfigError {
    #[error("{at}: expected `key = value`, `[section]`, or a comment")]
    Syntax { at: Span },
    #[error("{at}: unknown key `{key}` in section [{section}]")]
    UnknownKey { at: Span, section: String, key: String },
    #[error("{at}: `{key}` is set twice (first at line {first})")]
    Duplicate { at: Span, key: String, first: usize },
    #[error("{at}: invalid value {value:?} for `{key}`: {reason}")]
    Invalid { at: Span, key: String, value: String, reason: String },
    #[error("missing required key `{0}`")]
    Missing(&'static str),
}

const KNOWN: [(&str, &str); 5] =
    [("server", "host"), ("server", "port"), ("server", "workers"), ("log", "level"), ("server", "timeout_ms")];

/// Parses the whole text and returns the config, or every error found.
pub fn parse(text: &str) -> Result<ServerConfig, Vec<ConfigError>> {
    let mut errors = Vec::new();
    let mut values: HashMap<String, (String, Span)> = HashMap::new();
    let mut section = String::new();
    for (i, raw) in text.lines().enumerate() {
        let line_no = i + 1;
        let indent = raw.len() - raw.trim_start().len();
        let line = raw.trim();
        if line.is_empty() || line.starts_with('#') {
            continue;
        }
        if let Some(name) = line.strip_prefix('[').and_then(|l| l.strip_suffix(']')) {
            section = name.trim().to_string();
            continue;
        }
        let Some((key, value)) = line.split_once('=') else {
            errors.push(ConfigError::Syntax { at: Span { line: line_no, column: indent + 1 } });
            continue;
        };
        let (key, value) = (key.trim(), value.trim());
        let at = Span { line: line_no, column: indent + 1 };
        if !KNOWN.contains(&(section.as_str(), key)) {
            errors.push(ConfigError::UnknownKey { at, section: section.clone(), key: key.to_string() });
            continue;
        }
        let full = format!("{section}.{key}");
        if let Some((_, first)) = values.get(&full) {
            errors.push(ConfigError::Duplicate { at, key: full, first: first.line });
            continue;
        }
        values.insert(full, (value.to_string(), at));
    }

    // Typed extraction: each field reports its own error and parsing continues.
    let mut field = |key: &'static str| values.remove(key);
    let host = field("server.host");
    let port = field("server.port");
    let workers = field("server.workers");
    let level = field("log.level");
    let timeout = field("server.timeout_ms");

    let host = match host {
        Some((h, _)) => Some(h),
        None => { errors.push(ConfigError::Missing("server.host")); None }
    };
    let port = number::<u16>("server.port", port, 1..=u16::MAX as u64, &mut errors, true);
    let workers = number::<u32>("server.workers", workers, 1..=256, &mut errors, false).or(Some(4));
    let timeout_ms = number::<u64>("server.timeout_ms", timeout, 1..=600_000, &mut errors, false).or(Some(30_000));
    let log_level = match level {
        None => Some(LogLevel::Info),
        Some((v, at)) => match v.to_ascii_lowercase().as_str() {
            "error" => Some(LogLevel::Error),
            "warn" => Some(LogLevel::Warn),
            "info" => Some(LogLevel::Info),
            "debug" => Some(LogLevel::Debug),
            _ => {
                errors.push(ConfigError::Invalid { at, key: "log.level".into(), value: v, reason: "expected error, warn, info, or debug".into() });
                None
            }
        },
    };

    match (host, port, workers, log_level, timeout_ms) {
        (Some(host), Some(port), Some(workers), Some(log_level), Some(timeout_ms)) if errors.is_empty() => {
            Ok(ServerConfig { host, port, workers, log_level, timeout_ms })
        }
        _ => Err(errors),
    }
}

fn number<T: TryFrom<u64>>(
    key: &'static str,
    entry: Option<(String, Span)>,
    range: std::ops::RangeInclusive<u64>,
    errors: &mut Vec<ConfigError>,
    required: bool,
) -> Option<T> {
    let Some((value, at)) = entry else {
        if required {
            errors.push(ConfigError::Missing(key));
        }
        return None;
    };
    let invalid = |reason: String| ConfigError::Invalid { at: at.clone(), key: key.into(), value: value.clone(), reason };
    match value.parse::<u64>() {
        Err(e) => { errors.push(invalid(e.to_string())); None }
        Ok(n) if !range.contains(&n) => {
            errors.push(invalid(format!("must be in {}..={}", range.start(), range.end())));
            None
        }
        Ok(n) => T::try_from(n).ok(),
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn minimal_config_uses_defaults() {
        let c = parse("[server]\nhost = example.org\nport = 443\n").unwrap();
        assert_eq!((c.workers, c.timeout_ms, c.log_level), (4, 30_000, LogLevel::Info));
    }

    #[test]
    fn reports_every_error_with_positions() {
        let errs = parse("[server]\nport = 70000\n  colour = blue\nport = 1\nnonsense\n").unwrap_err();
        assert_eq!(errs.len(), 5, "{errs:#?}");
        assert!(matches!(&errs[0], ConfigError::UnknownKey { at: Span { line: 3, column: 3 }, .. }));
        assert!(matches!(&errs[1], ConfigError::Duplicate { first: 2, .. }));
        assert!(matches!(&errs[2], ConfigError::Syntax { at: Span { line: 5, .. } }));
        assert_eq!(errs[3], ConfigError::Missing("server.host"));
        assert!(matches!(&errs[4], ConfigError::Invalid { key, .. } if key == "server.port"));
    }
}
```

```rust
// file: confload/src/main.rs
use anyhow::{Context, Result};
use std::process::ExitCode;

fn main() -> Result<ExitCode> {
    let path = std::env::args().nth(1).context("usage: confload FILE")?;
    let text = std::fs::read_to_string(&path).with_context(|| format!("reading {path}"))?;
    match confload::parse(&text) {
        Ok(config) => {
            println!("{config:#?}");
            Ok(ExitCode::SUCCESS)
        }
        Err(errors) => {
            eprintln!("{path}: {} error(s)", errors.len());
            for (i, e) in errors.iter().enumerate() {
                eprintln!("  {}. {path}:{e}", i + 1);
            }
            Ok(ExitCode::from(65))                       // EX_DATAERR from sysexits.h
        }
    }
}
```

```text
<!-- file: confload/good.ini -->
# production server
[server]
host = api.example.org
port = 8443
workers = 16

[log]
level = Debug
```

```text
<!-- file: confload/bad.ini -->
[server]
host = api.example.org
port = 80800
workers = many
  timeout = 5000
port = 8443
this line is wrong

[log]
level = loud
```

```console
$ cd confload && cargo test -q 2>&1 | grep 'test result' | head -n 1
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
$ cd confload && cargo run -q -- good.ini; echo "status $?"
ServerConfig {
    host: "api.example.org",
    port: 8443,
    workers: 16,
    log_level: Debug,
    timeout_ms: 30000,
}
status 0
$ cd confload && cargo run -q -- bad.ini; echo "status $?"
bad.ini: 6 error(s)
  1. bad.ini:5:3: unknown key `timeout` in section [server]
  2. bad.ini:6:1: `server.port` is set twice (first at line 3)
  3. bad.ini:7:1: expected `key = value`, `[section]`, or a comment
  4. bad.ini:3:1: invalid value "80800" for `server.port`: must be in 1..=65535
  5. bad.ini:4:1: invalid value "many" for `server.workers`: invalid digit found in string
  6. bad.ini:10:1: invalid value "loud" for `log.level`: expected error, warn, info, or debug
status 65
$ cd confload && cargo run -q -- missing.ini; echo "status $?"
Error: reading missing.ini

Caused by:
    No such file or directory (os error 2)

Stack backtrace:
   0: <std::io::error::Error as anyhow::context::ext::StdError>::ext_context::<alloc::string::String>
             at /root/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/anyhow-1.0.104/src/backtrace.rs:10:14
   1: <core::result::Result<alloc::string::String, std::io::error::Error> as anyhow::Context<alloc::string::String, std::io::error::Error>>::with_context::<alloc::string::String, confload::main::{closure#0}>
             at /root/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/anyhow-1.0.104/src/context.rs:65:37
   2: confload::main
             at ./src/main.rs:7:47
   3: <fn() -> core::result::Result<std::process::ExitCode, anyhow::Error> as core::ops::function::FnOnce<()>>::call_once
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/ops/function.rs:250:5
   4: std::sys::backtrace::__rust_begin_short_backtrace::<fn() -> core::result::Result<std::process::ExitCode, anyhow::Error>, core::result::Result<std::process::ExitCode, anyhow::Error>>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/sys/backtrace.rs:166:18
   5: std::rt::lang_start::<core::result::Result<std::process::ExitCode, anyhow::Error>>::{closure#0}
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/rt.rs:206:18
   6: <&dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::RefUnwindSafe as core::ops::function::FnOnce<()>>::call_once
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/ops/function.rs:287:21
   7: std::panicking::catch_unwind::do_call::<&dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::RefUnwindSafe, i32>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:581:40
   8: std::panicking::catch_unwind::<i32, &dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::RefUnwindSafe>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:544:19
   9: std::panic::catch_unwind::<&dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::RefUnwindSafe, i32>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panic.rs:359:14
  10: std::rt::lang_start_internal::{closure#0}
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/rt.rs:175:24
  11: std::panicking::catch_unwind::do_call::<std::rt::lang_start_internal::{closure#0}, isize>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:581:40
  12: std::panicking::catch_unwind::<isize, std::rt::lang_start_internal::{closure#0}>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:544:19
  13: std::panic::catch_unwind::<std::rt::lang_start_internal::{closure#0}, isize>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panic.rs:359:14
  14: std::rt::lang_start_internal
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/rt.rs:171:5
  15: std::rt::lang_start::<core::result::Result<std::process::ExitCode, anyhow::Error>>
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/rt.rs:205:5
  16: main
  17: __libc_start_call_main
             at ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
  18: __libc_start_main_impl
             at ./csu/../csu/libc-start.c:360:3
  19: _start
status 1
```

The design choices:

- **The library returns typed errors, all of them.** `parse` collects errors in a `Vec` instead of stopping at the first, because a person fixing a config file wants the full list. Each error carries a `Span`, and the variants hold the data (key, value, reason) a caller might act on. `#[non_exhaustive]` lets the library add variants later without breaking callers.
- **`thiserror` writes the boilerplate.** Each `#[error("...")]` attribute becomes the `Display` implementation, with fields interpolated.
- **The binary uses `anyhow`.** Failing to read the file is reported with context ("reading missing.ini") and the cause ("No such file or directory"), and returning `Err` from `main` gives status 1. A config with errors is not a crash — it is an expected outcome, reported with its own status (65, `EX_DATAERR`).
- **Defaults are explicit.** Optional keys get defaults with `.or(Some(default))`; only `host` and `port` are required.

### Review checklist

- Why does `parse` return `Result<ServerConfig, Vec<ConfigError>>` instead of `Result<ServerConfig, ConfigError>`? What does a caller lose or gain?
- Which errors could a caller want to *match on* rather than just print? Is the error type's information enough for an editor to underline the problem?
- What happens to a key in an unknown section, such as `[database] url = x`? Is that the right behaviour?

## Answers to the Chapter 4 prediction questions

**4.5 Q1.** No. `"300".parse::<u8>()` returns `Result<u8, ParseIntError>`, and `?` needs `From<ParseIntError> for String`, which does not exist (error E0277: "`?` couldn't convert the error to `String`"). Write `.map_err(|e| e.to_string())?`.

**4.5 Q2.** `-1`. `filter` turns `Some(4)` into `None` because 4 is even; `map` leaves `None` alone; `unwrap_or` supplies −1.

# Chapter 5 — Traits and Generics

*Goal: write code that works for many types — generic functions and types constrained by traits — and choose between static dispatch (generics, monomorphised) and dynamic dispatch (trait objects, vtables) knowing what each costs.*

Traits are Rust's single mechanism for shared behaviour. They play the roles that interfaces, abstract base classes, type classes, operator overloading, and C++ concepts play in other languages. Generics with trait bounds give compile-time polymorphism; trait objects (`dyn Trait`) give run-time polymorphism.

**What you will be able to do after this chapter**

- Define traits with required and default methods, and implement them for your types and for others'.
- Write generic functions, structs, and `impl` blocks with trait bounds and `where` clauses.
- Implement the standard traits: `Display`, `From`/`Into`, `Default`, operators, and comparisons.
- Use trait objects, explain their layout, and know which traits can be used with `dyn`.
- Use associated types and constants, supertraits, and blanket implementations.

## Topic 5A — Traits

## Unit 5.1 — Defining and implementing traits

**One-line summary:** A **trait** declares methods a type must provide (and may supply **default** implementations). `impl Trait for Type` implements it. You can implement your trait for any type — including `i32` or `Vec<T>` — and any trait for your type; the **orphan rule** forbids implementing a foreign trait for a foreign type.

```rust
// file: traits.rs
trait Describe {
    fn name(&self) -> String;                         // required
    fn describe(&self) -> String {                    // default, may be overridden
        format!("<{}>", self.name())
    }
}

struct Dog { name: String }
struct Robot { serial: u32 }

impl Describe for Dog {
    fn name(&self) -> String { self.name.clone() }
    fn describe(&self) -> String { format!("{} the dog", self.name) }
}

impl Describe for Robot {
    fn name(&self) -> String { format!("unit-{}", self.serial) }    // uses the default describe
}

impl Describe for i32 {                               // our trait on a foreign type: allowed
    fn name(&self) -> String { format!("the number {self}") }
}

fn main() {
    let d = Dog { name: "Rex".into() };
    let r = Robot { serial: 7 };
    println!("{} | {} | {}", d.describe(), r.describe(), 42.describe());
}
```

```console
$ rustc --edition 2024 traits.rs && ./traits
Rex the dog | <unit-7> | <the number 42>
```

### Deep & Tricky

**DT-19. The orphan rule: you may implement a trait for a type only if your crate defines the trait or the type.** *(Guarantee: Rust language, coherence)* `impl Display for Vec<MyType>` is rejected — both `Display` and `Vec` are foreign, even though `MyType` is yours. The rule guarantees **coherence**: there is never more than one implementation of a trait for a type in a whole program, so adding a dependency can never make your code ambiguous. The standard workaround is a **newtype**: `struct Report(Vec<MyType>);` is your type, so you can implement `Display` for it.

<!-- expanded -->
### Going deeper: default methods and calling them through `self`

A trait method with a body is a **default method**: implementors get it for free and may override it. Default methods can call the trait's required methods, so a trait can offer a large interface while asking implementors for one or two functions — the way `Iterator` asks only for `next` and provides some seventy methods built on it.

```rust
// file: defaults.rs
trait Shape {
    fn area(&self) -> f64;                                   // required
    fn name(&self) -> String { "shape".into() }              // default
    fn describe(&self) -> String {                           // default, built on the others
        format!("{} with area {:.2}", self.name(), self.area())
    }
}

struct Circle(f64);
struct Square(f64);

impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.0 * self.0 }
    fn name(&self) -> String { "circle".into() }             // overrides the default
}
impl Shape for Square {
    fn area(&self) -> f64 { self.0 * self.0 }                // keeps the default name
}

fn main() {
    println!("{}", Circle(1.0).describe());
    println!("{}", Square(2.0).describe());
}
```

```console
$ rustc --edition 2024 defaults.rs && ./defaults
circle with area 3.14
shape with area 4.00
```

## Unit 5.2 — Generic functions and trait bounds

**One-line summary:** `fn f<T: Bound>(x: T)` is a function for every type `T` that implements `Bound`; the compiler generates a specialised copy per type used (monomorphisation, Unit 0.4). Multiple bounds combine with `+`; long bounds go in a `where` clause. `impl Trait` in argument position is shorthand for a generic parameter; in return position it means "some single concrete type that implements the trait, which the caller cannot name".

```rust
// file: generics.rs
use std::fmt::Display;

fn largest<T: PartialOrd + Copy>(items: &[T]) -> Option<T> {
    let mut iter = items.iter().copied();
    let mut best = iter.next()?;
    for x in iter {
        if x > best {
            best = x;
        }
    }
    Some(best)
}

fn show_all<T>(label: &str, items: &[T]) -> String
where
    T: Display,
{
    let parts: Vec<String> = items.iter().map(|x| x.to_string()).collect();
    format!("{label}: {}", parts.join(", "))
}

fn print_twice(x: impl Display) {                    // same as fn print_twice<T: Display>(x: T)
    println!("{x} {x}");
}

fn evens(limit: u32) -> impl Iterator<Item = u32> {  // the concrete iterator type stays hidden
    (0..limit).filter(|n| n % 2 == 0)
}

#[derive(Debug)]
struct Pair<T> {
    a: T,
    b: T,
}

impl<T: PartialOrd + Display> Pair<T> {              // methods only when T supports them
    fn larger(&self) -> &T {
        if self.a >= self.b { &self.a } else { &self.b }
    }
}

fn main() {
    println!("{:?} {:?} {:?}", largest(&[3, 8, 1]), largest(&[0.5, -1.0]), largest::<char>(&[]));
    println!("{}", show_all("primes", &[2, 3, 5, 7]));
    print_twice("hi");
    println!("{:?}", evens(10).collect::<Vec<_>>());
    let p = Pair { a: "pear", b: "apple" };
    println!("{p:?} larger: {}", p.larger());
}
```

```console
$ rustc --edition 2024 generics.rs && ./generics
Some(8) Some(0.5) None
primes: 2, 3, 5, 7
hi hi
[0, 2, 4, 6, 8]
Pair { a: "pear", b: "apple" } larger: pear
```

Calling a generic function with a type that does not meet its bounds fails at the call, with a message naming the missing trait — the same benefit C++20 concepts brought to templates (C++ book, Chapter 8), here from the start:

```rust
// file: bound_error.rs
fn largest<T: PartialOrd + Copy>(items: &[T]) -> T {
    let mut best = items[0];
    for &x in items {
        if x > best {
            best = x;
        }
    }
    best
}

fn main() {
    let names = vec![String::from("b"), String::from("a")];
    println!("{}", largest(&names));
}
```

```console
$ rustc --edition 2024 bound_error.rs 2>&1 | head -n 12 
error[E0277]: the trait bound `String: Copy` is not satisfied
  --> bound_error.rs:14:28
   |
14 |     println!("{}", largest(&names));
   |                    ------- ^^^^^^ the trait `Copy` is not implemented for `String`
   |                    |
   |                    required by a bound introduced by this call
   |
note: required by a bound in `largest`
  --> bound_error.rs:2:28
   |
 2 | fn largest<T: PartialOrd + Copy>(items: &[T]) -> T {
```

The body of a generic function is checked *once*, against its bounds, not per instantiation: inside `largest`, only operations that `PartialOrd + Copy` provide are allowed. A generic Rust function that compiles works for every type satisfying its bounds.

<!-- expanded -->
### Going deeper: `impl Trait` in argument and return position

`fn f(x: impl Display)` is shorthand for a generic `fn f<T: Display>(x: T)` (except that callers cannot name `T` with turbofish). `fn g() -> impl Iterator<Item = u32>` is different: the *function* picks one concrete type and hides it. That is how you return closures and iterator chains, whose types cannot be written. All return paths must produce the same type; when they cannot, return `Box<dyn Trait>`.

```rust
// file: impltrait.rs
use std::fmt::Display;

fn show_all(items: &[impl Display]) -> String {
    items.iter().map(|i| i.to_string()).collect::<Vec<_>>().join(", ")
}

fn evens(limit: u32) -> impl Iterator<Item = u32> { (0..limit).filter(|n| n % 2 == 0) }

fn make_adder(k: i32) -> impl Fn(i32) -> i32 { move |x| x + k }

fn numbers(desc: bool) -> Box<dyn Iterator<Item = u32>> {
    if desc { Box::new((1..=3).rev()) } else { Box::new(1..=3) }   // two different types: needs Box<dyn>
}

fn main() {
    println!("{}", show_all(&[1.5, 2.0]));
    println!("{:?}", evens(10).collect::<Vec<_>>());
    println!("{}", make_adder(10)(5));
    println!("{:?} {:?}", numbers(true).collect::<Vec<_>>(), numbers(false).collect::<Vec<_>>());
}
```

```console
$ rustc --edition 2024 impltrait.rs && ./impltrait
1.5, 2
[0, 2, 4, 6, 8]
15
[3, 2, 1] [1, 2, 3]
```

Writing `numbers` with `-> impl Iterator` fails with "`if` and `else` have incompatible types": `Rev<RangeInclusive<u32>>` is not `RangeInclusive<u32>`.

## Unit 5.3 — Implementing standard traits

**One-line summary:** Much of Rust's convenience comes from implementing a handful of standard traits: `Display` (for `{}` and `.to_string()`), `From<T>` (and the `Into<T>` it provides for free), `Default`, the operator traits in `std::ops` (`Add`, `Sub`, `Mul`, `Neg`, `Index`, `AddAssign`, …), and `PartialEq`/`PartialOrd` when derived behaviour is not right.

```rust
// file: stdtraits.rs
use std::fmt;
use std::ops::{Add, Mul, Neg};

#[derive(Debug, Clone, Copy, PartialEq, Default)]
struct V2 {
    x: f64,
    y: f64,
}

impl fmt::Display for V2 {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match f.precision() {                                 // honour {:.N} from the caller
            Some(p) => write!(f, "({:.*}, {:.*})", p, self.x, p, self.y),
            None => write!(f, "({}, {})", self.x, self.y),
        }
    }
}

impl Add for V2 {
    type Output = V2;
    fn add(self, o: V2) -> V2 { V2 { x: self.x + o.x, y: self.y + o.y } }
}

impl Mul<f64> for V2 {                                        // V2 * f64
    type Output = V2;
    fn mul(self, k: f64) -> V2 { V2 { x: self.x * k, y: self.y * k } }
}

impl Mul<V2> for f64 {                                        // f64 * V2: our trait impl on a foreign type
    type Output = V2;
    fn mul(self, v: V2) -> V2 { v * self }
}

impl Neg for V2 {
    type Output = V2;
    fn neg(self) -> V2 { V2 { x: -self.x, y: -self.y } }
}

impl From<(f64, f64)> for V2 {
    fn from((x, y): (f64, f64)) -> V2 { V2 { x, y } }
}

fn main() {
    let a = V2 { x: 1.0, y: 2.0 };
    let b: V2 = (0.5, -1.0).into();                          // Into comes free with From
    let c = a + b * 2.0 + 0.5 * -a;
    println!("{a} + {b}*2 + 0.5*-a = {c:.2}; default {}; c as text has {} chars", V2::default(), c.to_string().len());
}
```

```console
$ rustc --edition 2024 stdtraits.rs && ./stdtraits
(1, 2) + (0.5, -1)*2 + 0.5*-a = (1.50, -1.00); default (0, 0); c as text has 9 chars
```

`impl Mul<V2> for f64` is allowed by the orphan rule because `Mul<V2>` — the trait *with its type argument* — mentions our type. Implement `From`, never `Into`: the standard library's blanket `impl<T, U: From<T>> Into<U> for T` provides `Into` automatically.

<!-- expanded -->
### Going deeper: `From`/`Into`, `AsRef`, and `Display`

A handful of standard traits make your types fit in everywhere. Implement `From<A> for B` and you get `Into<B> for A` free, plus `?` conversions for errors. Accept `impl AsRef<str>` (or `AsRef<Path>`) to take `String`, `&str`, and others without forcing a conversion. Implement `Display` and you get `.to_string()`.

```rust
// file: convtraits.rs
use std::fmt;
use std::path::Path;

struct Rgb(u8, u8, u8);

impl From<u32> for Rgb {
    fn from(hex: u32) -> Self { Rgb((hex >> 16) as u8, (hex >> 8) as u8, hex as u8) }
}
impl fmt::Display for Rgb {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result { write!(f, "#{:02x}{:02x}{:02x}", self.0, self.1, self.2) }
}

fn shout(s: impl AsRef<str>) -> String { s.as_ref().to_uppercase() }
fn extension(p: impl AsRef<Path>) -> Option<String> {
    p.as_ref().extension().map(|e| e.to_string_lossy().into_owned())
}

fn main() {
    let c: Rgb = 0x1e90ff.into();                                // Into comes from From
    println!("{c} {}", Rgb::from(0xff8800).to_string());
    println!("{} {}", shout("hi"), shout(String::from("there")));
    println!("{:?} {:?}", extension("notes.txt"), extension(std::path::PathBuf::from("/tmp/a.tar.gz")));
}
```

```console
$ rustc --edition 2024 convtraits.rs && ./convtraits
#1e90ff #ff8800
HI THERE
Some("txt") Some("gz")
```

Implement `From`, not `Into`: the blanket implementation gives you `Into` for nothing, and the reverse is not true.

## Topic 5B — Dispatch

## Unit 5.4 — Trait objects and dynamic dispatch

**One-line summary:** `dyn Trait` is a type meaning "some type implementing `Trait`, decided at run time". It is unsized, so it is always used behind a pointer — `&dyn Trait`, `Box<dyn Trait>`, `Arc<dyn Trait>` — which is a **fat pointer**: a data pointer plus a pointer to a **vtable** of the trait's methods for the concrete type. Use trait objects for heterogeneous collections and plugin-style boundaries; use generics when the type is known at compile time.

```rust
// file: dynamic.rs
trait Shape {
    fn area(&self) -> f64;
    fn name(&self) -> &'static str;
}

struct Circle(f64);
struct Square(f64);

impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.0 * self.0 }
    fn name(&self) -> &'static str { "circle" }
}
impl Shape for Square {
    fn area(&self) -> f64 { self.0 * self.0 }
    fn name(&self) -> &'static str { "square" }
}

fn total_static<S: Shape>(shapes: &[S]) -> f64 {          // one type per call, inlined
    shapes.iter().map(|s| s.area()).sum()
}

fn total_dynamic(shapes: &[Box<dyn Shape>]) -> f64 {      // any mix of types, via the vtable
    shapes.iter().map(|s| s.area()).sum()
}

fn main() {
    let mixed: Vec<Box<dyn Shape>> = vec![Box::new(Circle(1.0)), Box::new(Square(2.0)), Box::new(Circle(0.5))];
    for s in &mixed {
        println!("{:<7} {:.3}", s.name(), s.area());
    }
    println!("dynamic total {:.3}, static total of squares {:.1}", total_dynamic(&mixed), total_static(&[Square(1.0), Square(3.0)]));
    println!("size_of: &Circle {}, &dyn Shape {}, Box<dyn Shape> {}", size_of::<&Circle>(), size_of::<&dyn Shape>(), size_of::<Box<dyn Shape>>());

    // the cost: 10 million area() calls each way
    let n = 10_000_000;
    let squares: Vec<Square> = (0..n).map(|i| Square((i % 7) as f64)).collect();
    let boxed: Vec<Box<dyn Shape>> = (0..n).map(|i| Box::new(Square((i % 7) as f64)) as Box<dyn Shape>).collect();
    let t = std::time::Instant::now();
    let s1 = total_static(&squares);
    let t_static = t.elapsed();
    let t = std::time::Instant::now();
    let s2 = total_dynamic(&boxed);
    let t_dyn = t.elapsed();
    println!("static {t_static:.1?} vs dynamic {t_dyn:.1?} (sums equal: {})", s1 == s2);
}
```

```console
$ rustc --edition 2024 -O dynamic.rs && ./dynamic
circle  3.142
square  4.000
circle  0.785
dynamic total 7.927, static total of squares 10.0
size_of: &Circle 8, &dyn Shape 16, Box<dyn Shape> 16
static 10.8ms vs dynamic 32.1ms (sums equal: true)
```

The static version is faster for two reasons that usually matter more than the indirect call itself: the compiler can inline and vectorise `area` into the loop, and the `Vec<Square>` stores the values contiguously, while each `Box<dyn Shape>` is a separate heap allocation reached through a pointer (C++ book, Unit 12.4).

### Deep & Tricky

**DT-20. Only *dyn-compatible* traits can be used as `dyn Trait`: no generic methods, no methods returning `Self`, no associated constants.** *(Guarantee: Rust language; formerly "object safety")* A vtable holds one function pointer per method, so every method must have a single, non-generic signature, and the caller must not need to know the concrete type's size (as returning `Self` by value would). `Clone` is therefore not dyn-compatible (`fn clone(&self) -> Self`). A trait can keep such methods but exclude them from the vtable with `where Self: Sized`, which makes them callable on concrete types only.

<!-- expanded -->
### Going deeper: what a trait object is in memory

A `&dyn Trait` or `Box<dyn Trait>` is a **fat pointer**: two machine words, one pointing to the data and one to a **vtable** — a table holding the type's size, alignment, destructor, and one function pointer per trait method. Calling a method loads the function pointer from the vtable and calls it, which prevents inlining across the call.

```rust
// file: fatptr.rs
use std::mem::size_of;

trait Speak { fn speak(&self) -> String; }
struct Dog;
impl Speak for Dog { fn speak(&self) -> String { "woof".into() } }

fn main() {
    println!("&Dog: {} bytes, &dyn Speak: {} bytes, Box<dyn Speak>: {} bytes",
             size_of::<&Dog>(), size_of::<&dyn Speak>(), size_of::<Box<dyn Speak>>());
    println!("&[u8]: {} bytes, &str: {} bytes (pointer + length)", size_of::<&[u8]>(), size_of::<&str>());
    let d: &dyn Speak = &Dog;
    let (data, vtable): (*const (), *const ()) = unsafe { std::mem::transmute(d) };
    println!("data pointer and vtable pointer differ: {}", data != vtable);
    println!("{}", d.speak());
}
```

```console
$ rustc --edition 2024 fatptr.rs && ./fatptr
&Dog: 8 bytes, &dyn Speak: 16 bytes, Box<dyn Speak>: 16 bytes
&[u8]: 16 bytes, &str: 16 bytes (pointer + length)
data pointer and vtable pointer differ: true
woof
```

(The `transmute` here only splits the fat pointer for display; its layout is not a stable guarantee, so real code never does this.)

## Unit 5.5 — Associated types, constants, supertraits, and blanket impls

**One-line summary:** A trait can declare **associated types** (`type Item;`) that each implementation fixes — unlike a generic parameter, there is one per implementing type, so callers need not specify it — and **associated constants**. A **supertrait** (`trait Pretty: Display`) requires another trait. A **blanket implementation** (`impl<T: Display> Pretty for T`) implements a trait for every type meeting a bound.

```rust
// file: assoc.rs
use std::fmt::Display;

trait Container {
    type Item;                                         // chosen by each implementation
    const KIND: &'static str;
    fn items(&self) -> Vec<&Self::Item>;
    fn first(&self) -> Option<&Self::Item> {
        self.items().into_iter().next()
    }
}

struct Shelf { books: Vec<String> }

impl Container for Shelf {
    type Item = String;
    const KIND: &'static str = "shelf";
    fn items(&self) -> Vec<&String> { self.books.iter().collect() }
}

trait Labelled: Display {                              // supertrait: every Labelled is Display
    fn label(&self) -> String { format!("[{self}]") }
}
impl<T: Display> Labelled for T {}                     // blanket impl: every Display type is Labelled

fn summary<C>(c: &C) -> String
where
    C: Container,
    C::Item: Display,
{
    format!("{} with {} items, first = {}", C::KIND, c.items().len(), c.first().map(|x| x.to_string()).unwrap_or_default())
}

fn main() {
    let s = Shelf { books: vec!["Dune".into(), "Emma".into()] };
    println!("{}", summary(&s));
    println!("{} {} {}", 42.label(), "text".label(), 2.5.label());
}
```

```console
$ rustc --edition 2024 assoc.rs && ./assoc
shelf with 2 items, first = Dune
[42] [text] [2.5]
```

`Iterator` is the most important trait with an associated type (`type Item`): a type iterates over one kind of item, so `Iterator<Item = u32>` names it without a generic parameter on every function that uses iterators. Generic parameters on traits (`From<T>`, `Mul<Rhs>`) are for traits a type may implement many times with different arguments.

### Pattern

**P-5. Prefer generics for performance and for single-type call sites; prefer trait objects for heterogeneous collections, plugin boundaries, and keeping compile times and binary size down.** *(Practice)* Generics are monomorphised: fastest code, but one copy per type (Unit 0.4). Trait objects compile once and add an indirect call and usually a heap allocation. A common hybrid is a generic public function that immediately calls a non-generic inner function taking `&dyn Trait` — the standard library does this for `std::fs::read(path: impl AsRef<Path>)` to avoid duplicating the body for every path type. *Trade-off:* measure before converting one into the other; for many call sites the difference is noise.

<!-- expanded -->
### Going deeper: the extension-trait pattern

You cannot add methods to a type from another crate directly, but you can define your *own* trait and implement it for that type — the orphan rule allows it because the trait is yours. This **extension trait** pattern is how crates like `itertools` add methods to every iterator.

```rust
// file: extension.rs
trait StrExt {
    fn word_count(&self) -> usize;
    fn title_case(&self) -> String;
}

impl StrExt for str {
    fn word_count(&self) -> usize { self.split_whitespace().count() }
    fn title_case(&self) -> String {
        self.split_whitespace()
            .map(|w| {
                let mut c = w.chars();
                c.next().map(|f| f.to_uppercase().chain(c.flat_map(|x| x.to_lowercase())).collect::<String>()).unwrap_or_default()
            })
            .collect::<Vec<_>>()
            .join(" ")
    }
}

trait IterSumSq: Iterator<Item = f64> + Sized {
    fn sum_of_squares(self) -> f64 { self.map(|x| x * x).sum() }
}
impl<I: Iterator<Item = f64>> IterSumSq for I {}                 // blanket impl: every f64 iterator

fn main() {
    let s = "the QUICK brown fox";
    println!("{} words, {:?}", s.word_count(), s.title_case());
    println!("{}", [1.0, 2.0, 3.0].into_iter().sum_of_squares());
}
```

```console
$ rustc --edition 2024 extension.rs && ./extension
4 words, "The Quick Brown Fox"
14
```

The methods are available only where the trait is in scope (`use my_crate::StrExt;`), which keeps them from colliding with other crates' extensions.

### Mastery check

1. **Predict.** Does `let v: Vec<Box<dyn Clone>> = Vec::new();` compile?
2. **Predict.** What is `size_of::<&[u8]>()` and `size_of::<&str>()` on a 64-bit target, and why are they the same as `size_of::<&dyn Shape>()`?

## Chapter 5 Project — Units of measure in the type system

**Goal:** a small library of physical quantities in which the **unit is part of the type**, so that adding metres to seconds is a compile-time error, dividing a length by a time produces a velocity, and conversions between compatible units (kilometres, miles) are explicit and checked. It uses generics, `PhantomData`, operator traits, associated constants, and `Display`.

```rust
// file: units/units.rs
use std::fmt;
use std::marker::PhantomData;
use std::ops::{Add, Div, Mul, Sub};

/// A unit belongs to a dimension and has a factor to the dimension's base unit.
trait Unit {
    type Dim;
    const SYMBOL: &'static str;
    const TO_BASE: f64;
}

// Dimensions are zero-sized marker types.
struct LengthDim;
struct TimeDim;
struct SpeedDim;

struct Meter;
struct Kilometer;
struct Mile;
struct Second;
struct Hour;
struct MeterPerSecond;
struct KmPerHour;

macro_rules! unit {
    ($name:ident, $dim:ident, $sym:expr, $f:expr) => {
        impl Unit for $name {
            type Dim = $dim;
            const SYMBOL: &'static str = $sym;
            const TO_BASE: f64 = $f;
        }
    };
}
unit!(Meter, LengthDim, "m", 1.0);
unit!(Kilometer, LengthDim, "km", 1000.0);
unit!(Mile, LengthDim, "mi", 1609.344);
unit!(Second, TimeDim, "s", 1.0);
unit!(Hour, TimeDim, "h", 3600.0);
unit!(MeterPerSecond, SpeedDim, "m/s", 1.0);
unit!(KmPerHour, SpeedDim, "km/h", 1000.0 / 3600.0);

/// A value tagged with its unit. PhantomData<U> uses U in the type without storing anything.
struct Q<U: Unit> {
    value: f64,
    unit: PhantomData<U>,
}

// Derives would require U: Clone etc., so implement Clone/Copy by hand for any U.
impl<U: Unit> Clone for Q<U> {
    fn clone(&self) -> Self { *self }
}
impl<U: Unit> Copy for Q<U> {}

impl<U: Unit> Q<U> {
    fn new(value: f64) -> Self { Q { value, unit: PhantomData } }
    /// Converts to another unit of the same dimension; other dimensions do not compile.
    fn to<V: Unit<Dim = U::Dim>>(self) -> Q<V> {
        Q::new(self.value * U::TO_BASE / V::TO_BASE)
    }
}

impl<U: Unit> fmt::Display for Q<U> {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let p = f.precision().unwrap_or(2);
        write!(f, "{:.*} {}", p, self.value, U::SYMBOL)
    }
}

impl<U: Unit> Add for Q<U> {                            // same unit only
    type Output = Q<U>;
    fn add(self, o: Q<U>) -> Q<U> { Q::new(self.value + o.value) }
}
impl<U: Unit> Sub for Q<U> {
    type Output = Q<U>;
    fn sub(self, o: Q<U>) -> Q<U> { Q::new(self.value - o.value) }
}
impl<U: Unit> Mul<f64> for Q<U> {                       // scaling by a plain number
    type Output = Q<U>;
    fn mul(self, k: f64) -> Q<U> { Q::new(self.value * k) }
}

/// length / time = speed, in base units, for any length and time units.
impl<L, T> Div<Q<T>> for Q<L>
where
    L: Unit<Dim = LengthDim>,
    T: Unit<Dim = TimeDim>,
{
    type Output = Q<MeterPerSecond>;
    fn div(self, t: Q<T>) -> Q<MeterPerSecond> {
        Q::new(self.value * L::TO_BASE / (t.value * T::TO_BASE))
    }
}

/// speed * time = length (in metres)
impl<S, T> Mul<Q<T>> for Q<S>
where
    S: Unit<Dim = SpeedDim>,
    T: Unit<Dim = TimeDim>,
{
    type Output = Q<Meter>;
    fn mul(self, t: Q<T>) -> Q<Meter> {
        Q::new(self.value * S::TO_BASE * t.value * T::TO_BASE)
    }
}

fn main() {
    let marathon = Q::<Kilometer>::new(42.195);
    let time = Q::<Hour>::new(2.0) + Q::<Second>::new(35.0 * 60.0 + 9.0).to::<Hour>();  // 2:35:09 elite-ish time
    let speed = marathon / time;
    println!("marathon {marathon:.3} = {:.3} = {:.1}", marathon.to::<Mile>(), marathon.to::<Meter>());
    println!("time {time:.4} = {:.0}", time.to::<Second>());
    println!("average speed {speed:.3} = {:.2}", speed.to::<KmPerHour>());
    let ten_minutes = Q::<Second>::new(600.0);
    println!("distance in 10 minutes at that pace: {:.0}", speed * ten_minutes);
    let lap = Q::<Meter>::new(400.0) * 2.5;
    println!("2.5 laps = {lap}, minus 1 km = {}", lap - Q::<Kilometer>::new(1.0).to::<Meter>());
    println!("size of Q<Mile> = {} bytes (the unit costs nothing at run time)", size_of::<Q<Mile>>());
}
```

```console
$ cd units && rustc --edition 2024 units.rs && ./units
marathon 42.195 km = 26.219 mi = 42195.0 m
time 2.5858 h = 9309 s
average speed 4.533 m/s = 16.32 km/h
distance in 10 minutes at that pace: 2720 m
2.5 laps = 1000.00 m, minus 1 km = 0.00 m
size of Q<Mile> = 8 bytes (the unit costs nothing at run time)
```

And the errors the type system now catches:

```rust
// file: units/mistakes.rs
include!("units_lib.rs");

fn main() {
    let d = Q::<Meter>::new(100.0);
    let t = Q::<Second>::new(9.58);
    let _a = d + t;                         // metres + seconds
    let _b = d + Q::<Kilometer>::new(1.0);  // different units: convert first
    let _c = t.to::<Mile>();                // a time is not a length
}
```

```console
$ cd units && sed '/^fn main/,$d' units.rs > units_lib.rs && rustc --edition 2024 mistakes.rs 2>&1 | grep -E '^error' 
error[E0308]: mismatched types
error[E0308]: mismatched types
error[E0271]: type mismatch resolving `<Mile as Unit>::Dim == TimeDim`
error: aborting due to 3 previous errors
```

All three mistakes are compile errors, and none of the checking survives to run time: `Q<Mile>` is just an `f64`. The **marker types** (`Meter`, `LengthDim`, …) are zero-sized and exist only in the type system; `PhantomData<U>` tells the compiler that `Q` is "about" `U` without storing a `U`. The conversion method `to::<V>()` accepts any unit `V` whose `Dim` equals ours — an associated-type equality bound — so converting a time to miles has no matching `impl`. The `unit!` macro is a first look at declarative macros (Chapter 12), used here to avoid writing seven nearly identical `impl` blocks.

### Review checklist

- Why are `Clone` and `Copy` implemented by hand instead of derived? (Try `#[derive(Clone, Copy)]` and read the error when you copy a `Q<Mile>`: derive adds the bound `U: Clone`.)
- `d + Q::<Kilometer>` is rejected. Would it be better to allow it and convert automatically? What would `Add`'s output unit be?
- Add an `Acceleration` dimension and implement speed / time. How many new `impl` blocks does it take?

## Answers to the Chapter 5 prediction questions

**5.5 Q1.** No. `Clone::clone` returns `Self`, so `Clone` is not dyn-compatible (DT-20); the compiler reports error E0038, "the trait `Clone` is not dyn compatible". (Crates such as `dyn-clone` work around it with a helper trait that returns `Box<dyn Trait>`.)

**5.5 Q2.** Both are 16 bytes: a pointer plus a length. `&dyn Shape` is also 16 bytes — a pointer plus a vtable pointer. All three are fat pointers to unsized types (slices, `str`, trait objects), whose extra word holds what the compiler cannot know statically: the length, or which type's methods to call.

# Chapter 6 — Collections, Closures, and Iterators

*Goal: use the standard collections fluently, write closures and understand how they capture, and process data with iterator pipelines that are as fast as hand-written loops.*

**What you will be able to do after this chapter**

- Use `Vec`, `VecDeque`, `HashMap`, `HashSet`, `BTreeMap`, `BTreeSet`, and `BinaryHeap`, and choose between them.
- Use the `entry` API to update maps without double lookups.
- Write closures, and explain `Fn`, `FnMut`, `FnOnce`, and `move`.
- Build iterator pipelines with adaptors and consumers, and implement `Iterator` for your own types.
- Collect into any collection — including `Result<Vec<_>, _>` to stop at the first error.

## Topic 6A — Collections

## Unit 6.1 — `Vec` and slices

**One-line summary:** `Vec<T>` is a growable, contiguous, heap-allocated array — Rust's default collection. It doubles its capacity when full (amortised O(1) `push`); `with_capacity` and `reserve` avoid regrowth. Most algorithms live on **slices** (`[T]`) and so work on vectors, arrays, and sub-ranges alike: `sort`, `sort_unstable`, `binary_search`, `contains`, `windows`, `chunks`, `split_at`, `iter`, `reverse`, `rotate_left`, plus `Vec`-specific `retain`, `dedup`, `drain`, `extend`, `truncate`.

```rust
// file: vecs.rs
fn main() {
    let mut v: Vec<i32> = Vec::new();
    let mut last_cap = v.capacity();
    for i in 0..100 {
        v.push(i);
        if v.capacity() != last_cap {
            print!("{} ", v.capacity());
            last_cap = v.capacity();
        }
    }
    println!("<- capacities while pushing 100 elements");

    let mut scores = vec![72, 95, 61, 88, 95, 40, 73, 88];
    scores.sort_unstable();
    scores.dedup();                                    // removes consecutive duplicates
    println!("sorted unique {scores:?}, has 88 at {:?}, 90 would go at {:?}",
             scores.binary_search(&88), scores.binary_search(&90));
    scores.retain(|&s| s >= 60);                       // keep passing scores
    let windows: Vec<i32> = scores.windows(2).map(|w| w[1] - w[0]).collect();
    println!("passing {scores:?}, gaps {windows:?}");
    for chunk in scores.chunks(2) {
        print!("{chunk:?} ");
    }
    println!();
    let drained: Vec<i32> = scores.drain(1..3).collect();
    println!("drained {drained:?}, left {scores:?}");
    let mut words = vec!["pear", "Fig", "apple", "banana"];
    words.sort_by_key(|w| w.to_lowercase());           // stable sort by a derived key
    println!("{words:?}");
}
```

```console
$ rustc --edition 2024 vecs.rs && ./vecs
4 8 16 32 64 128 <- capacities while pushing 100 elements
sorted unique [40, 61, 72, 73, 88, 95], has 88 at Ok(4), 90 would go at Err(5)
passing [61, 72, 73, 88, 95], gaps [11, 1, 15, 7]
[61, 72] [73, 88] [95] 
drained [72, 73], left [61, 88, 95]
["apple", "banana", "Fig", "pear"]
```

`VecDeque` is a ring buffer with O(1) push and pop at both ends (a queue); `BinaryHeap` is a max-heap (a priority queue; wrap items in `std::cmp::Reverse` for a min-heap); `LinkedList` exists but is almost never the right choice (C++ book, Unit 4.1, applies unchanged).

<!-- expanded -->
### Going deeper: capacity, `with_capacity`, and `retain`

A `Vec` is three words: pointer, length, capacity. Pushing beyond the capacity reallocates (doubling in the current implementation) and moves every element. `Vec::with_capacity(n)` allocates once; `retain` removes elements in place in one pass; `dedup` removes *consecutive* duplicates; `drain(range)` removes a range and yields it.

```rust
// file: vecops.rs
fn main() {
    let mut v: Vec<u32> = Vec::new();
    let mut caps = vec![];
    for i in 0..100 {
        v.push(i);
        if caps.last() != Some(&v.capacity()) { caps.push(v.capacity()); }
    }
    println!("capacities while pushing 100: {caps:?}");
    let w: Vec<u32> = Vec::with_capacity(100);
    println!("with_capacity(100): len {}, capacity {}", w.len(), w.capacity());

    v.retain(|x| x % 10 == 0);
    println!("retain multiples of 10: {v:?}");
    let removed: Vec<u32> = v.drain(1..4).collect();
    println!("drained {removed:?}, left {v:?}");
    let mut d = vec![1, 1, 2, 2, 2, 1, 3, 3];
    d.dedup();
    println!("dedup: {d:?} (the second 1 survives: not adjacent)");
    d.sort_unstable();
    d.dedup();
    println!("sort + dedup: {d:?}");
    println!("size_of::<Vec<u8>>() = {}", std::mem::size_of::<Vec<u8>>());
}
```

```console
$ rustc --edition 2024 vecops.rs && ./vecops
capacities while pushing 100: [4, 8, 16, 32, 64, 128]
with_capacity(100): len 0, capacity 100
retain multiples of 10: [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
drained [10, 20, 30], left [0, 40, 50, 60, 70, 80, 90]
dedup: [1, 2, 1, 3] (the second 1 survives: not adjacent)
sort + dedup: [1, 2, 3]
size_of::<Vec<u8>>() = 24
```

## Unit 6.2 — Maps and sets

**One-line summary:** `HashMap<K, V>` gives average O(1) lookup with keys that implement `Hash + Eq`; `BTreeMap<K, V>` keeps keys **sorted** (O(log n), keys implement `Ord`) and supports range queries. `HashSet` and `BTreeSet` are the key-only versions. The **entry API** — `map.entry(key).or_insert(default)` and friends — inserts or updates with a single lookup.

```rust
// file: maps.rs
use std::collections::{BTreeMap, HashMap, HashSet};

fn main() {
    let text = "the quick brown fox jumps over the lazy dog the end";
    let mut counts: HashMap<&str, usize> = HashMap::new();
    for word in text.split_whitespace() {
        *counts.entry(word).or_insert(0) += 1;         // one lookup per word
    }
    println!("the={} fox={} cat={:?}", counts["the"], counts["fox"], counts.get("cat"));

    let mut by_len: BTreeMap<usize, Vec<&str>> = BTreeMap::new();   // sorted by key
    for word in counts.keys() {
        by_len.entry(word.len()).or_default().push(word);
    }
    for words in by_len.values_mut() {
        words.sort();
    }
    println!("{by_len:?}");
    println!("words of length 4..=5: {:?}", by_len.range(4..=5).collect::<Vec<_>>());

    let a: HashSet<char> = "hello".chars().collect();
    let b: HashSet<char> = "world".chars().collect();
    let mut common: Vec<&char> = a.intersection(&b).collect();
    common.sort();
    println!("common letters {common:?}, a has {} distinct", a.len());

    // HashMap iteration order is unspecified (and randomised per process):
    let keys: Vec<&&str> = counts.keys().take(3).collect();
    println!("first three keys this run: {keys:?}");
}
```

```console
$ rustc --edition 2024 maps.rs && ./maps && ./maps | tail -n 1
the=3 fox=1 cat=None
{3: ["dog", "end", "fox", "the"], 4: ["lazy", "over"], 5: ["brown", "jumps", "quick"]}
words of length 4..=5: [(4, ["lazy", "over"]), (5, ["brown", "jumps", "quick"])]
common letters ['l', 'o'], a has 4 distinct
first three keys this run: ["over", "jumps", "the"]
first three keys this run: ["dog", "fox", "quick"]
```

The last line differs between the two runs: `HashMap` uses a **randomly seeded** hash function, so iteration order changes from run to run. Never depend on it; use `BTreeMap` (or sort) when order matters.

### Deep & Tricky

**DT-21. `HashMap`'s default hasher is SipHash-1-3 with a random key — chosen for resistance to hash-flooding attacks, not for speed.** *(Library behaviour: `RandomState`)* An attacker who can choose keys (HTTP headers, JSON object keys) cannot predict collisions, so they cannot make every insert O(n). The cost is speed: for small keys such as integers, faster non-cryptographic hashers (`FxHash` from the `rustc-hash` crate, `ahash`, `foldhash`) can make maps several times faster. Use them for keys an attacker cannot influence (compiler internals, numeric IDs), via `HashMap<K, V, S>`'s third type parameter. Chapter 13 measures the difference.

<!-- expanded -->
### Going deeper: the entry API

Looking a key up and then inserting it does two hash lookups, and with borrows it often does not even compile. The **entry API** does it once: `map.entry(key)` returns an `Entry` that is either occupied or vacant, with helpers for the common cases.

```rust
// file: entry.rs
use std::collections::{BTreeMap, HashMap};

fn main() {
    let text = "the cat and the dog and the bird";
    let mut counts: BTreeMap<&str, usize> = BTreeMap::new();
    for w in text.split(' ') { *counts.entry(w).or_insert(0) += 1; }
    println!("{counts:?}");

    let mut groups: HashMap<usize, Vec<&str>> = HashMap::new();
    for w in text.split(' ') { groups.entry(w.len()).or_default().push(w); }
    let mut keys: Vec<_> = groups.keys().copied().collect();
    keys.sort();
    for k in keys { println!("length {k}: {:?}", groups[&k]); }

    let mut cache: HashMap<u64, u64> = HashMap::new();
    let mut computed = 0;
    for n in [10, 20, 10, 10] {
        let v = *cache.entry(n).or_insert_with(|| { computed += 1; n * n });   // closure runs only when vacant
        print!("{v} ");
    }
    println!("\ncomputed {computed} times");
    let first_two: Vec<_> = counts.range("b".."d").collect();
    println!("BTreeMap range b..d: {first_two:?}");
}
```

```console
$ rustc --edition 2024 entry.rs && ./entry
{"and": 2, "bird": 1, "cat": 1, "dog": 1, "the": 3}
length 3: ["the", "cat", "and", "the", "dog", "and", "the"]
length 4: ["bird"]
100 400 100 100 
computed 2 times
BTreeMap range b..d: [("bird", 1), ("cat", 1)]
```

`or_insert(expensive())` evaluates `expensive()` every time, even when the key exists; `or_insert_with(|| expensive())` only when it is needed. `BTreeMap` keeps keys sorted, so it supports `range` queries and iterates in order — at the price of slower lookups than `HashMap`.

## Topic 6B — Closures

## Unit 6.3 — Closures and how they capture

**One-line summary:** A **closure** `|args| body` is an anonymous function that can use variables from its environment. The compiler captures each variable in the least demanding way the body allows — by shared reference, by mutable reference, or by value — and gives the closure one of three traits: **`Fn`** (only reads captures: callable many times, even concurrently), **`FnMut`** (mutates captures: callable many times, needs `&mut`), **`FnOnce`** (consumes a capture: callable once). `move` forces capture by value.

```rust
// file: closures.rs
fn apply_twice<F: Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(f(x))
}

fn call_n<F: FnMut()>(mut f: F, n: usize) {
    for _ in 0..n {
        f();
    }
}

fn consume<F: FnOnce() -> String>(f: F) -> String {
    f()
}

fn make_adder(k: i32) -> impl Fn(i32) -> i32 {
    move |x| x + k                                  // k is copied into the closure
}

fn make_counter() -> impl FnMut() -> u32 {
    let mut count = 0;
    move || {
        count += 1;                                 // the closure owns and mutates its own count
        count
    }
}

fn main() {
    let offset = 10;
    let add = |x| x + offset;                       // captures offset by reference: Fn
    println!("apply_twice: {}", apply_twice(add, 1));

    let mut log = Vec::new();
    call_n(|| log.push(log.len()), 3);              // captures log by &mut: FnMut
    println!("log: {log:?}");

    let name = String::from("Ferris");
    let greeting = move || format!("hello, {name}"); // name moved into the closure
    println!("{}", consume(greeting));              // FnOnce would also accept Fn closures

    let add5 = make_adder(5);
    let mut next = make_counter();
    println!("add5(1) = {}, counter: {} {} {}", add5(1), next(), next(), next());

    let ops: Vec<(&str, Box<dyn Fn(f64) -> f64>)> = vec![
        ("double", Box::new(|x| x * 2.0)),
        ("square", Box::new(|x| x * x)),
        ("plus offset", Box::new(move |x| x + offset as f64)),
    ];
    for (label, op) in &ops {
        println!("{label}(3) = {}", op(3.0));
    }
    println!("size of add5 closure: {} bytes, of a capture-free closure: {}", size_of_val(&add5), size_of_val(&|x: i32| x));
}
```

```console
$ rustc --edition 2024 closures.rs && ./closures
apply_twice: 21
log: [0, 1, 2]
hello, Ferris
add5(1) = 6, counter: 1 2 3
double(3) = 6
square(3) = 9
plus offset(3) = 13
size of add5 closure: 4 bytes, of a capture-free closure: 0
```

Every closure has its own unique, unnameable type, containing exactly its captures — `add5` is a 4-byte struct holding `k`; a closure that captures nothing is zero-sized. Generic functions taking `F: Fn(...)` are monomorphised for each closure and inline it completely; `Box<dyn Fn(...)>` erases the type for storage in collections, at the cost of an allocation and an indirect call (Unit 5.4).

### Deep & Tricky

**DT-22. Since Rust 2021, closures capture *disjoint fields*, not whole variables.** *(Guarantee: Rust language, edition 2021+)* A closure that uses `p.name` captures only `p.name`, so other fields of `p` stay usable — and borrowable mutably — while the closure exists. In the 2018 edition the closure captured all of `p`. One visible consequence: a `move` closure that uses `p.name` moves only that field, leaving `p` partially moved (DT-12), and *when* `p`'s other fields are dropped can change between editions; `cargo fix --edition` inserts `let _ = &p;` where that matters.

<!-- expanded -->
### Deep & Tricky

**DT-23. Each closure has its own unique, unnameable type — even two closures with identical code — so `vec![|x| x + 1, |x| x + 2]` does not compile, and `if c { |x| x + 1 } else { |x| x * 2 }` does not either. Store them as `Box<dyn Fn(i32) -> i32>` (or `fn(i32) -> i32` when they capture nothing: non-capturing closures coerce to function pointers).** *(Guarantee: Reference, "closure types" — each closure expression has a distinct anonymous type)*

```rust
// file: closuretypes.rs
fn main() {
    let ops: Vec<fn(i32) -> i32> = vec![|x| x + 1, |x| x * 2, |x| -x];      // no captures: fn pointers
    println!("{:?}", ops.iter().map(|f| f(10)).collect::<Vec<_>>());

    let k = 5;
    let capturing: Vec<Box<dyn Fn(i32) -> i32>> = vec![Box::new(move |x| x + k), Box::new(|x| x - 1)];
    println!("{:?}", capturing.iter().map(|f| f(10)).collect::<Vec<_>>());
    println!("sizes: fn pointer {}, Box<dyn Fn> {}", std::mem::size_of::<fn(i32) -> i32>(), std::mem::size_of::<Box<dyn Fn(i32) -> i32>>());
}
```

```console
$ rustc --edition 2024 closuretypes.rs && ./closuretypes
[11, 20, -10]
[15, 9]
sizes: fn pointer 8, Box<dyn Fn> 16
$ printf 'fn main() { let k = 1; let v = vec![move |x: i32| x + k, move |x: i32| x + k]; }\n' > twoclosures.rs && rustc --edition 2024 twoclosures.rs 2>&1 | grep -m2 -oE "(error\[E[0-9]+\]: .*|no two closures.*)"
error[E0308]: mismatched types
no two closures, even if identical, have the same type
```

## Topic 6C — Iterators

## Unit 6.4 — The `Iterator` trait

**One-line summary:** An iterator is anything implementing `trait Iterator { type Item; fn next(&mut self) -> Option<Self::Item>; }` — plus about 75 provided methods built on `next`. **Adaptors** (`map`, `filter`, `enumerate`, `zip`, `take`, `skip`, `chain`, `flat_map`, …) wrap an iterator in another, lazily; **consumers** (`sum`, `count`, `collect`, `fold`, `find`, `any`, `max_by_key`, `for_each`, …) drive it. Collections provide three iterators: `iter()` yields `&T`, `iter_mut()` yields `&mut T`, `into_iter()` yields `T` by value (consuming the collection). A `for` loop calls `into_iter()`.

```rust
// file: iterators.rs
/// An iterator over the Fibonacci numbers that stops before overflow.
struct Fib {
    a: u64,
    b: u64,
    done: bool,
}

impl Iterator for Fib {
    type Item = u64;
    fn next(&mut self) -> Option<u64> {
        if self.done {
            return None;
        }
        let out = self.a;
        match self.a.checked_add(self.b) {
            Some(next) => (self.a, self.b) = (self.b, next),
            None => self.done = true,
        }
        Some(out)
    }
}

fn fib() -> Fib {
    Fib { a: 0, b: 1, done: false }
}

fn main() {
    let lazy = (1..).map(|x| x * x).filter(|x| x % 3 == 1);   // nothing has run yet
    let first: Vec<u64> = lazy.take(5).collect();               // now it runs: just enough
    println!("squares ≡ 1 mod 3: {first:?}");
    println!("even Fibonacci below 4 million sum to {}", fib().take_while(|&f| f < 4_000_000).filter(|f| f % 2 == 0).sum::<u64>());
    println!("u64 holds {} Fibonacci numbers; the last is {}", fib().count(), fib().last().unwrap());

    let mut v = vec![1, 2, 3];
    for x in v.iter_mut() {
        *x *= 10;
    }
    let total: i32 = v.iter().sum();                             // borrows
    let strings: Vec<String> = v.into_iter().map(|x| x.to_string()).collect();   // consumes v
    println!("total {total}, strings {strings:?}");

    let names = ["ada", "grace", "linus"];
    let years = [1815, 1906, 1969];
    for (i, (n, y)) in names.iter().zip(years).enumerate() {
        print!("{i}:{n}/{y} ");
    }
    println!();
}
```

```console
$ rustc --edition 2024 iterators.rs && ./iterators
squares ≡ 1 mod 3: [1, 4, 16, 25, 49]
even Fibonacci below 4 million sum to 4613732
u64 holds 93 Fibonacci numbers; the last is 7540113804746346429
total 60, strings ["10", "20", "30"]
0:ada/1815 1:grace/1906 2:linus/1969 
```

Iterators are **zero-cost abstractions**: after inlining, a chain of adaptors compiles to the same loop you would write by hand. (Iterating with an iterator also has no index to bounds-check; here the optimiser removed the checks from the index loop too.) Here is the evidence:

```rust
// file: zerocost.rs
use std::hint::black_box;
use std::time::Instant;

fn loop_version(v: &[u32]) -> u64 {
    let mut total = 0u64;
    let mut i = 0;
    while i < v.len() {
        if v[i] % 3 == 0 {
            total += (v[i] as u64) * (v[i] as u64);
        }
        i += 1;
    }
    total
}

fn iter_version(v: &[u32]) -> u64 {
    v.iter().filter(|&&x| x % 3 == 0).map(|&x| x as u64 * x as u64).sum()
}

fn main() {
    let v: Vec<u32> = (0..50_000_000).map(|i: u32| i.wrapping_mul(2_654_435_761) % 1000).collect();
    for _ in 0..2 {
        let t = Instant::now();
        let a = loop_version(black_box(&v));
        let t1 = t.elapsed();
        let t = Instant::now();
        let b = iter_version(black_box(&v));
        let t2 = t.elapsed();
        println!("index loop {t1:.1?}, iterator chain {t2:.1?}, same result: {}", a == b);
    }
}
```

```console
$ rustc --edition 2024 -O zerocost.rs && ./zerocost
index loop 103.4ms, iterator chain 121.5ms, same result: true
index loop 123.5ms, iterator chain 124.8ms, same result: true
```

<!-- expanded -->
### Going deeper: implementing `IntoIterator` for your own type

`for x in collection` calls `IntoIterator::into_iter(collection)`. Implementing it for `&MyType` (yielding references) and for `MyType` (yielding owned values) lets your type be used in `for` loops exactly like a `Vec`.

```rust
// file: intoiter.rs
struct Playlist { songs: Vec<String> }

impl<'a> IntoIterator for &'a Playlist {
    type Item = &'a String;
    type IntoIter = std::slice::Iter<'a, String>;
    fn into_iter(self) -> Self::IntoIter { self.songs.iter() }
}
impl IntoIterator for Playlist {
    type Item = String;
    type IntoIter = std::vec::IntoIter<String>;
    fn into_iter(self) -> Self::IntoIter { self.songs.into_iter() }
}

fn main() {
    let p = Playlist { songs: vec!["one".into(), "two".into()] };
    for s in &p { print!("borrowed {s}; "); }          // p is still usable
    println!();
    for s in p { print!("owned {s}; "); }              // p is consumed
    println!();
}
```

```console
$ rustc --edition 2024 intoiter.rs && ./intoiter
borrowed one; borrowed two; 
owned one; owned two; 
```

## Unit 6.5 — The iterator toolbox

**One-line summary:** Learn the adaptors and consumers as a vocabulary: transforming (`map`, `filter_map`, `flat_map`, `flatten`, `scan`), selecting (`filter`, `take`, `take_while`, `skip`, `skip_while`, `step_by`, `peekable`), combining (`zip`, `chain`, `enumerate`, `rev`, `cycle`), and consuming (`collect`, `sum`, `product`, `fold`, `reduce`, `count`, `min_by_key`, `max_by`, `position`, `any`, `all`, `partition`, `unzip`). `collect` can build any collection implementing `FromIterator` — including `String`, `HashMap`, and `Result<Vec<T>, E>`.

```rust
// file: toolbox.rs
use std::collections::HashMap;

fn main() {
    let lines = ["alice,34,paris", "bob,x,rome", "carol,29,paris", "", "dave,41,oslo"];

    // filter_map: parse what parses, skip the rest
    let ages: Vec<u32> = lines.iter().filter_map(|l| l.split(',').nth(1)?.parse().ok()).collect();
    println!("ages {ages:?}");

    // collect into Result: stops at the first error
    let strict: Result<Vec<u32>, _> = lines.iter().filter(|l| !l.is_empty()).map(|l| l.split(',').nth(1).unwrap_or("").parse::<u32>()).collect();
    println!("strict parse: {strict:?}");

    // group into a map
    let mut by_city: HashMap<&str, Vec<&str>> = HashMap::new();
    for (name, city) in lines.iter().filter_map(|l| { let f: Vec<&str> = l.split(',').collect(); (f.len() == 3).then(|| (f[0], f[2])) }) {
        by_city.entry(city).or_default().push(name);
    }
    let mut cities: Vec<_> = by_city.into_iter().collect();
    cities.sort();
    println!("{cities:?}");

    // partition, unzip, fold, scan, flat_map, peekable
    let (even, odd): (Vec<i32>, Vec<i32>) = (1..=10).partition(|n| n % 2 == 0);
    let (xs, ys): (Vec<i32>, Vec<char>) = [(1, 'a'), (2, 'b')].into_iter().unzip();
    let running: Vec<i32> = [3, 1, 4, 1, 5].iter().scan(0, |acc, &x| { *acc += x; Some(*acc) }).collect();
    let csv = ["a b", "c", "d e f"].iter().flat_map(|s| s.split(' ')).collect::<Vec<_>>().join(",");
    let longest = ["fig", "banana", "kiwi"].iter().max_by_key(|s| s.len()).unwrap();
    let product = (1..=10u64).fold(1, |acc, n| acc * n);
    println!("even {even:?} odd {odd:?} | {xs:?} {ys:?} | running {running:?} | {csv} | {longest} | 10! = {product}");

    // peekable: group consecutive runs
    let data = [1, 1, 2, 2, 2, 3, 1];
    let mut it = data.iter().peekable();
    let mut runs = Vec::new();
    while let Some(&x) = it.next() {
        let mut n = 1;
        while it.next_if(|&&y| y == x).is_some() {
            n += 1;
        }
        runs.push((x, n));
    }
    println!("runs {runs:?}");
}
```

```console
$ rustc --edition 2024 toolbox.rs && ./toolbox
ages [34, 29, 41]
strict parse: Err(ParseIntError { kind: InvalidDigit })
[("oslo", ["dave"]), ("paris", ["alice", "carol"]), ("rome", ["bob"])]
even [2, 4, 6, 8, 10] odd [1, 3, 5, 7, 9] | [1, 2] ['a', 'b'] | running [3, 4, 8, 9, 14] | a,b,c,d,e,f | banana | 10! = 3628800
runs [(1, 2), (2, 3), (3, 1), (1, 1)]
```

### Deep & Tricky

**DT-24. Iterator adaptors are lazy: an adaptor chain that is never consumed does nothing, and side effects inside it run in an interleaved order.** *(Guarantee: Rust standard library)* `v.iter().map(|x| println!("{x}"));` prints nothing — the compiler warns "unused `Map` that must be used". When a chain is consumed, each element passes through *all* stages before the next element starts (`map` 1, `filter` 1, `map` 2, `filter` 2, …), not stage by stage as in some other languages' collection APIs. This is what makes `(1..).map(...).take(5)` terminate, and why side effects in `map` should be avoided — use `for_each` or a `for` loop when the point is the effect.

### Pattern

**P-6. Write data processing as iterator pipelines; fall back to a `for` loop when the logic has early exits across several collections or complex state.** *(Practice)* Pipelines state *what* is computed (filter, transform, aggregate), avoid off-by-one index bugs, skip bounds checks, and parallelise with almost no change (the `rayon` crate's `par_iter`, Chapter 9). A `for` loop is clearer when the body mutates several things, `break`s on conditions that span iterations, or needs `?` on errors mid-loop (though `try_for_each` and `collect::<Result<_,_>>` cover many such cases). *Trade-off:* very long chains with nested closures can be harder to debug than a loop; name intermediate iterators or extract functions.

<!-- expanded -->
### Going deeper: `fold`, `scan`, `windows`, `chunks`, `zip`, and `peekable`

```rust
// file: toolbox2.rs
fn main() {
    let prices = [100.0, 102.0, 99.0, 105.0, 110.0];
    let moving_avg: Vec<f64> = prices.windows(3).map(|w| w.iter().sum::<f64>() / 3.0).collect();
    let running_max: Vec<f64> = prices.iter().scan(f64::MIN, |m, &p| { *m = m.max(p); Some(*m) }).collect();
    let changes: Vec<f64> = prices.iter().zip(prices.iter().skip(1)).map(|(a, b)| b - a).collect();
    let product = (1..=10u64).fold(1, |acc, n| acc * n);
    println!("moving average: {moving_avg:?}");
    println!("running max: {running_max:?}");
    println!("changes: {changes:?}");
    println!("10! = {product}, chunks of 2: {:?}", [1, 2, 3, 4, 5].chunks(2).collect::<Vec<_>>());

    let mut it = "aaabccdd".chars().peekable();                    // run-length encoding with peek
    let mut rle = String::new();
    while let Some(c) = it.next() {
        let mut n = 1;
        while it.peek() == Some(&c) { it.next(); n += 1; }
        rle.push_str(&format!("{c}{n}"));
    }
    println!("rle: {rle}");
    let (small, large): (Vec<i32>, Vec<i32>) = [5, 50, 3, 30].into_iter().partition(|&x| x < 10);
    println!("partition: {small:?} {large:?}, position of 30: {:?}", large.iter().position(|&x| x == 30));
}
```

```console
$ rustc --edition 2024 toolbox2.rs && ./toolbox2
moving average: [100.33333333333333, 102.0, 104.66666666666667]
running max: [100.0, 102.0, 102.0, 105.0, 110.0]
changes: [2.0, -3.0, 6.0, 5.0]
10! = 3628800, chunks of 2: [[1, 2], [3, 4], [5]]
rle: a3b1c2d2
partition: [5, 3] [50, 30], position of 30: Some(1)
```

### Mastery check

1. **Predict.** What does `(1..=4).flat_map(|n| std::iter::repeat(n).take(n)).collect::<Vec<_>>()` produce?
2. **Predict.** What does `["3", "x", "5"].iter().map(|s| s.parse::<i32>()).collect::<Result<Vec<_>, _>>().is_err()` return, and how many times is `parse` called?

## Chapter 6 Project — `logstat`: an access-log analyser

**Goal:** a program that reads a web-server access log and reports traffic per status class, the busiest paths, latency percentiles per path, and requests per minute — written as iterator pipelines over `HashMap`/`BTreeMap`, tested on a generated million-line log, and fast enough to process it in well under a second.

Each line looks like: `2026-11-15T10:04:31 GET /api/users 200 37` (timestamp, method, path, status, milliseconds). A generator creates a reproducible log with a realistic mix, including some malformed lines.

```rust
// file: logstat/gen.rs
fn main() {
    let n: u64 = std::env::args().nth(1).and_then(|s| s.parse().ok()).unwrap_or(1_000_000);
    let paths = ["/", "/api/users", "/api/orders", "/static/app.js", "/login", "/api/search", "/health"];
    let weights = [20u64, 25, 15, 25, 5, 8, 2];
    let mut x: u64 = 0x2545F4914F6CDD1D;                         // xorshift64: reproducible, no crates
    let mut rand = move || { x ^= x << 13; x ^= x >> 7; x ^= x << 17; x };
    let total: u64 = weights.iter().sum();
    let mut out = String::with_capacity(48 * n as usize);
    for i in 0..n {
        if rand() % 5000 == 0 {
            out.push_str("garbage line\n");
            continue;
        }
        let mut pick = rand() % total;
        let p = weights.iter().position(|&w| if pick < w { true } else { pick -= w; false }).unwrap();
        let status = match rand() % 100 { 0..=1 => 500, 2..=6 => 404, 7..=9 => 302, _ => 200 };
        let base = [5, 40, 60, 2, 80, 120, 1][p];
        let ms = base + rand() % (base * 2 + 1) + if rand() % 100 == 0 { 2000 } else { 0 };   // 1% slow outliers
        let secs = i * 3600 / n;                                  // one hour of traffic
        let method = if p == 2 && rand() % 3 == 0 { "POST" } else { "GET" };
        out.push_str(&format!("2026-11-15T10:{:02}:{:02} {method} {} {status} {ms}\n", secs / 60, secs % 60, paths[p]));
    }
    print!("{out}");
}
```

```rust
// file: logstat/logstat.rs
use std::collections::{BTreeMap, HashMap};
use std::time::Instant;

struct Entry<'a> {
    minute: &'a str,            // "10:04": borrowed from the input, no allocation
    path: &'a str,
    status: u16,
    ms: u32,
}

fn parse(line: &str) -> Option<Entry<'_>> {
    let mut f = line.split(' ');
    let ts = f.next()?;
    let _method = f.next()?;
    let path = f.next()?;
    let status = f.next()?.parse().ok()?;
    let ms = f.next()?.parse().ok()?;
    let minute = ts.get(11..16)?;
    Some(Entry { minute, path, status, ms })
}

fn percentile(sorted: &[u32], p: f64) -> u32 {
    sorted[((sorted.len() - 1) as f64 * p).round() as usize]
}

fn main() {
    let path = std::env::args().nth(1).expect("usage: logstat FILE");
    let text = std::fs::read_to_string(&path).expect("readable log");
    let start = Instant::now();

    let (entries, bad): (Vec<Option<Entry>>, Vec<Option<Entry>>) = text.lines().map(parse).partition(Option::is_some);
    let entries: Vec<Entry> = entries.into_iter().flatten().collect();

    let mut by_class: BTreeMap<u16, usize> = BTreeMap::new();
    let mut by_path: HashMap<&str, Vec<u32>> = HashMap::new();
    let mut per_minute: BTreeMap<&str, usize> = BTreeMap::new();
    for e in &entries {
        *by_class.entry(e.status / 100 * 100).or_default() += 1;
        by_path.entry(e.path).or_default().push(e.ms);
        *per_minute.entry(e.minute).or_default() += 1;
    }
    let mut paths: Vec<(&str, Vec<u32>)> = by_path.into_iter().collect();
    for (_, ms) in paths.iter_mut() {
        ms.sort_unstable();
    }
    paths.sort_by(|a, b| b.1.len().cmp(&a.1.len()).then(a.0.cmp(b.0)));
    let busiest = per_minute.iter().max_by_key(|&(m, n)| (*n, std::cmp::Reverse(*m))).unwrap();
    let quietest = per_minute.iter().min_by_key(|&(m, n)| (*n, *m)).unwrap();
    let elapsed = start.elapsed();

    println!("{} requests, {} malformed lines", entries.len(), bad.len());
    for (class, n) in &by_class {
        println!("  {}xx {:>8}  {:5.2}%", class / 100, n, 100.0 * *n as f64 / entries.len() as f64);
    }
    println!("{:<16} {:>8} {:>6} {:>6} {:>6} {:>6}", "path", "count", "p50", "p95", "p99", "max");
    for (p, ms) in &paths {
        println!("{:<16} {:>8} {:>6} {:>6} {:>6} {:>6}", p, ms.len(), percentile(ms, 0.50), percentile(ms, 0.95), percentile(ms, 0.99), ms[ms.len() - 1]);
    }
    println!("busiest minute {} ({} req), quietest {} ({} req), {} minutes", busiest.0, busiest.1, quietest.0, quietest.1, per_minute.len());
    eprintln!("analysed in {elapsed:.0?}");
}
```

```console
$ cd logstat && rustc --edition 2024 -O gen.rs && ./gen 1000000 > access.log && wc -l < access.log && head -n 3 access.log
1000000
2026-11-15T10:00:00 GET / 200 8
2026-11-15T10:00:00 GET /static/app.js 200 2
2026-11-15T10:00:00 GET /api/orders 200 114
$ cd logstat && rustc --edition 2024 -O logstat.rs && ./logstat access.log
999784 requests, 216 malformed lines
  2xx   900025  90.02%
  3xx    29971   3.00%
  4xx    49968   5.00%
  5xx    19820   1.98%
path                count    p50    p95    p99    max
/api/users         250454     80    117    120   2120
/static/app.js     249681      4      6   2002   2006
/                  199790     10     15     15   2015
/api/orders        150254    121    176   2062   2180
/api/search         79893    241    351   2131   2360
/login              49517    161    234    240   2240
/health             20195      2      3   2001   2003
busiest minute 10:37 (16667 req), quietest 10:33 (16655 req), 60 minutes
analysed in 167ms
```

How the program uses the chapter:

- **Zero-copy parsing.** `Entry<'a>` borrows `minute` and `path` from the input text (Chapter 2), so a million entries cost no string allocations; only the per-path latency vectors allocate.
- **`partition` + `flatten`** split parsed lines from malformed ones in one pass.
- **The right map for each job:** `HashMap` for the per-path grouping (no order needed), `BTreeMap` for status classes and minutes (sorted output for free).
- **Sorting with composite keys:** `then` chains comparisons, and `Reverse` flips one component, so ties break deterministically.
- The generator's 1% of slow outliers (about 2 seconds) show up in the maximum of every path and in the p99 of most — exactly at the edge, since they are 1% of requests — while p50 and p95 are untouched. A mean would blend them into every number; this is why latency is reported as percentiles.

### Review checklist

- `parse` uses `?` on `Option` throughout. What happens to a line with a missing field, an extra field, or a non-numeric status?
- `Entry` borrows from `text`. Could `main` return `entries` from a helper function that reads the file itself? (Chapter 2's project answers this.)
- Which step dominates the run time? Try replacing the `HashMap` with a `BTreeMap`, or `sort_unstable` with `sort`, and measure.

## Answers to the Chapter 6 prediction questions

**6.5 Q1.** `[1, 2, 2, 3, 3, 3, 4, 4, 4, 4]`.

**6.5 Q2.** `true`. Collecting into `Result` stops at the first `Err`: `parse` runs for `"3"` and `"x"` — twice — and `"5"` is never parsed.

# Chapter 7 — Modules, Crates, Workspaces, Testing, and Documentation

*Goal: organise code that outgrows one file — modules and visibility, library and binary crates, features, dependencies and semantic versioning, multi-crate workspaces — and test and document it the way Rust projects do.*

**What you will be able to do after this chapter**

- Split a crate into modules across files, control visibility, and design a public API with re-exports.
- Structure a package with a library, several binaries, and optional **features**.
- Understand how Cargo resolves dependency versions, and what `Cargo.lock` guarantees.
- Build a **workspace** of several crates that share a lock file and build directory.
- Write unit tests, integration tests, and documentation tests, and generate documentation.

## Topic 7A — Code organisation

## Unit 7.1 — Modules and visibility

**One-line summary:** A crate is a tree of **modules**. `mod name;` declares a module whose code is in `name.rs` (or `name/mod.rs`); `mod name { ... }` declares it inline. Everything is **private** to its module (and that module's descendants) unless marked `pub`; `pub(crate)` makes an item visible within the crate only. Paths start from `crate::` (the root), `self::`, or `super::` (the parent); `use` brings names into scope, and `pub use` **re-exports** them, so the public API need not mirror the file layout.

```toml
# file: shop/Cargo.toml
[package]
name = "shop"
version = "0.1.0"
edition = "2024"
```

```rust
// file: shop/src/lib.rs
//! A tiny shop: the crate root decides what the public API looks like.

mod inventory;                              // src/inventory.rs
mod pricing;                                // src/pricing/mod.rs + src/pricing/discount.rs

pub use inventory::{Inventory, Item};       // re-exports: users write shop::Item
pub use pricing::total;

/// Version of the pricing rules, visible inside the crate only.
pub(crate) const RULES_VERSION: u32 = 3;
```

```rust
// file: shop/src/inventory.rs
#[derive(Debug, Clone)]
pub struct Item {
    pub name: String,
    pub cents: u64,
    sku: u32,                               // private field: only this module can set it
}

impl Item {
    pub fn new(name: &str, cents: u64) -> Item {
        Item { name: name.to_string(), cents, sku: next_sku() }
    }
    pub fn sku(&self) -> u32 {
        self.sku
    }
}

fn next_sku() -> u32 {                      // private helper
    use std::sync::atomic::{AtomicU32, Ordering};
    static NEXT: AtomicU32 = AtomicU32::new(1000);
    NEXT.fetch_add(1, Ordering::Relaxed)
}

#[derive(Default)]
pub struct Inventory {
    items: Vec<Item>,
}

impl Inventory {
    pub fn add(&mut self, item: Item) {
        self.items.push(item);
    }
    pub fn items(&self) -> &[Item] {
        &self.items
    }
}
```

```rust
// file: shop/src/pricing/mod.rs
mod discount;                               // src/pricing/discount.rs, private to pricing

use crate::inventory::Item;

/// Total price in cents after the bulk discount.
pub fn total(items: &[Item]) -> u64 {
    let gross: u64 = items.iter().map(|i| i.cents).sum();
    gross - discount::bulk(gross, items.len())
}
```

```rust
// file: shop/src/pricing/discount.rs
/// 10% off orders of 3 or more items, under rules version `RULES_VERSION`.
pub(super) fn bulk(gross: u64, count: usize) -> u64 {
    debug_assert!(crate::RULES_VERSION >= 3);
    if count >= 3 { gross / 10 } else { 0 }
}
```

```rust
// file: shop/src/main.rs
use shop::{Inventory, Item};

fn main() {
    let mut inv = Inventory::default();
    for (name, cents) in [("pen", 150), ("pad", 400), ("ink", 950)] {
        inv.add(Item::new(name, cents));
    }
    for item in inv.items() {
        println!("#{} {:<4} {:>5.2}", item.sku(), item.name, item.cents as f64 / 100.0);
    }
    println!("total with bulk discount: {:.2}", shop::total(inv.items()) as f64 / 100.0);
}
```

```console
$ cd shop && find src -name '*.rs' | sort && cargo run -q
src/inventory.rs
src/lib.rs
src/main.rs
src/pricing/discount.rs
src/pricing/mod.rs
#1000 pen   1.50
#1001 pad   4.00
#1002 ink   9.50
total with bulk discount: 13.50
```

The package has two crates: a **library** (`src/lib.rs`) and a **binary** (`src/main.rs`) that uses the library like any external user would — through `shop::`, seeing only what is `pub` and reachable. Trying to reach a private item is a clear error:

```rust
// file: shop/examples/peek.rs
fn main() {
    let item = shop::Item::new("x", 1);
    println!("{}", item.sku);                   // private field
    shop::pricing::total(&[]);                  // private module
}
```

```console
$ cd shop && cargo run -q --example peek 2>&1 | grep -E '^error'
error[E0603]: module `pricing` is private
error[E0616]: field `sku` of struct `Item` is private
error: could not compile `shop` (example "peek") due to 2 previous errors
```

<!-- expanded -->
### Going deeper: `pub(crate)`, re-exports, and the public API

Visibility in Rust has more steps than public and private: `pub(crate)` is visible anywhere in the same crate but not to users of the crate, `pub(super)` to the parent module, `pub(in path)` to a chosen ancestor. A library's *public API* is whatever is reachable from the crate root, so a common layout keeps modules private and **re-exports** the chosen items at the top with `pub use` — users write `mylib::Parser` instead of `mylib::parse::internal::Parser`, and you can move code between modules without breaking them.

```rust
// file: vis/src/lib.rs
mod parse {
    pub struct Parser { pub(crate) depth: usize }       // field visible in this crate only
    impl Parser {
        pub fn new() -> Self { Parser { depth: 0 } }
        pub fn parse(&mut self, s: &str) -> usize { self.depth = helpers::nesting(s); self.depth }
    }
    mod helpers {
        pub(super) fn nesting(s: &str) -> usize {        // usable by parse, not by the rest of the crate
            let (mut d, mut max) = (0i32, 0i32);
            for c in s.chars() {
                if c == '(' { d += 1; max = max.max(d); } else if c == ')' { d -= 1; }
            }
            max as usize
        }
    }
}
pub use parse::Parser;                                  // the public path is vis::Parser

#[cfg(test)]                                            // only the tests need it
pub(crate) fn debug_depth(p: &Parser) -> usize { p.depth }

#[cfg(test)]
mod tests {
    #[test]
    fn depth_is_visible_inside_the_crate() {
        let mut p = super::Parser::new();
        p.parse("(a(b)(c(d)))");
        assert_eq!(super::debug_depth(&p), 3);
    }
}
```

```toml
# file: vis/Cargo.toml
[package]
name = "vis"
version = "0.1.0"
edition = "2024"
```

```rust
// file: vis/examples/use_it.rs
use vis::Parser;                                         // the re-exported path

fn main() {
    let mut p = Parser::new();
    println!("depth = {}", p.parse("((x))"));
    // println!("{}", p.depth);                          // error: field `depth` is private (pub(crate))
}
```

```console
$ cd vis && cargo test -q 2>&1 | grep "test result" | head -1 && cargo run -q --example use_it
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
depth = 2
```

`cargo doc` shows exactly the public API; checking that page is the quickest review of what you are promising to keep stable.

## Unit 7.2 — Packages, dependencies, and features

**One-line summary:** A **package** (one `Cargo.toml`) contains at most one library crate and any number of binaries (`src/main.rs`, `src/bin/*.rs`), examples, tests, and benchmarks. Dependencies are declared with **semantic version requirements**: `serde = "1.0.200"` means `^1.0.200` — any version `>= 1.0.200` and `< 2.0.0`. **Features** are named, additive switches that enable optional code and optional dependencies, tested with `#[cfg(feature = "name")]`.

```toml
# file: feat/Cargo.toml
[package]
name = "feat"
version = "0.1.0"
edition = "2024"

[dependencies]
rustc-hash = { version = "2", optional = true }   # only compiled when the feature is on

[features]
default = []
fast-hash = ["dep:rustc-hash"]                    # the feature enables the dependency
```

```rust
// file: feat/src/main.rs
#[cfg(feature = "fast-hash")]
type Map<K, V> = rustc_hash::FxHashMap<K, V>;
#[cfg(not(feature = "fast-hash"))]
type Map<K, V> = std::collections::HashMap<K, V>;

fn main() {
    let mut m: Map<u64, u64> = Map::default();
    let t = std::time::Instant::now();
    for i in 0..5_000_000u64 {
        *m.entry(i % 100_000).or_insert(0) += i;
    }
    let hasher = if cfg!(feature = "fast-hash") { "FxHash" } else { "SipHash (std)" };
    println!("{hasher:<14} {} keys in {:.0?}", m.len(), t.elapsed());
}
```

```console
$ cd feat && cargo run -q --release && cargo run -q --release --features fast-hash
SipHash (std)  100000 keys in 119ms
FxHash         100000 keys in 40ms
$ cd feat && cargo tree --features fast-hash && cargo tree
feat v0.1.0 (feat)
└── rustc-hash v2.1.3
feat v0.1.0 (feat)
```

### Deep & Tricky

**DT-25. Features are unified across the whole dependency graph: if any crate enables a feature of a dependency, every user of that dependency gets it.** *(Cargo behaviour: feature unification)* Cargo builds each dependency once per build, with the **union** of all features requested anywhere in the graph. That is why features must be **additive** — turning one on must never remove functionality or change behaviour incompatibly — and why `#[cfg(not(feature = "x"))]` code paths can disappear from a build without your crate asking for it. (Resolver version 2, the default since the 2021 edition, avoids unifying features between build-time dependencies and normal ones, and between targets.)

**DT-26. Cargo picks the newest compatible version of each dependency, and `Cargo.lock` freezes that choice.** *(Cargo behaviour: dependency resolution)* Within a major version (or within `0.x` minor versions, which Cargo treats as incompatible with each other), the resolver prefers the newest release, and tries to use one version per major version across the graph. `Cargo.lock` records the exact versions so builds are reproducible; `cargo update` re-resolves within the version requirements. Libraries' lock files are ignored by their users, so a library must actually work with the oldest version its requirements allow — `cargo update -Z minimal-versions` (nightly) or the `cargo-minimal-versions` tool check it.

## Unit 7.3 — Workspaces

**One-line summary:** A **workspace** groups several packages under one root `Cargo.toml` with a `[workspace]` section. Members share one `Cargo.lock` and one `target/` directory, can depend on each other by `path`, and can inherit common settings (`version`, `edition`, dependency versions) from `[workspace.package]` and `[workspace.dependencies]`. `cargo build`, `test`, and `clippy` at the root act on all members (`-p name` selects one).

The chapter project is a workspace; see it for the layout.

<!-- expanded -->
### A small workspace, step by step

A workspace shares one `Cargo.lock` and one `target/` directory between its member packages, so a dependency used by three members is built once, and `cargo test --workspace` runs every member's tests. The root `Cargo.toml` can also hold shared settings: `[workspace.package]` (edition, version, licence) and `[workspace.dependencies]` (one version of each dependency for all members, which members opt into with `dep = { workspace = true }`).

```toml
# file: ws/Cargo.toml
[workspace]
members = ["geom", "app"]
resolver = "3"

[workspace.package]
edition = "2024"
version = "0.2.0"
```

```toml
# file: ws/geom/Cargo.toml
[package]
name = "geom"
version.workspace = true
edition.workspace = true
```

```rust
// file: ws/geom/src/lib.rs
/// Area of a triangle from its three side lengths (Heron's formula).
pub fn heron(a: f64, b: f64, c: f64) -> Option<f64> {
    let s = (a + b + c) / 2.0;
    let sq = s * (s - a) * (s - b) * (s - c);
    (sq >= 0.0).then(|| sq.sqrt())
}

#[cfg(test)]
mod tests {
    #[test]
    fn right_triangle() { assert_eq!(super::heron(3.0, 4.0, 5.0), Some(6.0)); }
    #[test]
    fn impossible() { assert_eq!(super::heron(1.0, 1.0, 5.0), None); }
}
```

```toml
# file: ws/app/Cargo.toml
[package]
name = "app"
version.workspace = true
edition.workspace = true

[dependencies]
geom = { path = "../geom" }
```

```rust
// file: ws/app/src/main.rs
fn main() {
    match geom::heron(5.0, 5.0, 6.0) {
        Some(a) => println!("area = {a}"),
        None => println!("not a triangle"),
    }
}
```

```console
$ cd ws && cargo test -q --workspace 2>&1 | grep "test result" | grep -v " 0 passed" && cargo run -q -p app && ls
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
area = 12
Cargo.lock  Cargo.toml	app  geom  target
```

One `target/` directory at the root serves both members; the members have none of their own.

## Topic 7B — Testing and documentation

## Unit 7.4 — Tests

**One-line summary:** `cargo test` builds and runs three kinds of tests. **Unit tests** live in a `#[cfg(test)] mod tests` inside the module they test and can call private functions. **Integration tests** live in `tests/*.rs`; each file is a separate crate that sees only the library's public API. **Doctests** are the code examples in documentation comments. Tests pass by returning normally (or `Ok(())`) and fail by panicking (`assert!`, `assert_eq!`, `assert_ne!`) or returning `Err`; `#[should_panic(expected = "...")]` tests expected panics, and `#[ignore]` marks slow tests run with `--ignored`.

Tests run **in parallel threads** by default, in the same process. This matters:

```rust
// file: partest/src/lib.rs
pub fn config_path() -> String {
    std::env::var("APP_CONFIG").unwrap_or_else(|_| "/etc/app.conf".to_string())
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn uses_default_path() {
        assert_eq!(config_path(), "/etc/app.conf");
    }

    #[test]
    fn honours_environment() {
        unsafe { std::env::set_var("APP_CONFIG", "/tmp/test.conf") };   // process-wide!
        std::thread::sleep(std::time::Duration::from_millis(50));
        assert_eq!(config_path(), "/tmp/test.conf");
        unsafe { std::env::remove_var("APP_CONFIG") };
    }

    #[test]
    #[should_panic(expected = "divide by zero")]
    fn panics_on_zero() {
        let zero = std::hint::black_box(0);
        let _ = 1 / zero;
    }

    #[test]
    fn returns_result() -> Result<(), std::num::ParseIntError> {
        assert_eq!("42".parse::<i32>()?, 42);
        Ok(())
    }
}
```

```toml
# file: partest/Cargo.toml
[package]
name = "partest"
version = "0.1.0"
edition = "2024"
```

```console
$ cd partest && for i in 1 2 3 4 5 6; do cargo test -q 2>&1 | grep -E 'test result'; done | sort | uniq -c
      6 test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
      6 test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.06s
$ cd partest && cargo test -q -- --test-threads=1 2>&1 | grep 'test result'
test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.11s
test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Run six times, the suite failed six times: `honours_environment` sets a process-wide environment variable, and `uses_default_path`, running at the same time in another thread, sees it. (The 50 ms sleep makes the overlap certain here; without it, the failure would be intermittent — worse, because it would pass on your machine and fail in CI.) With one test thread, all four tests pass. The real fix is to not share global state between tests — pass the configuration in, or inject the environment lookup. (Edition 2024 makes `set_var` an `unsafe` function precisely because modifying the environment while other threads read it is a data race in the C library.)

### Deep & Tricky

**DT-27. Integration tests in `tests/` can only test a library crate — a binary-only package has nothing for them to import.** *(Cargo behaviour)* Each file in `tests/` is compiled as its own crate that links against the package's *library*. Logic that lives only in `src/main.rs` is invisible to it. This is the main reason idiomatic Rust binaries are thin: `main.rs` parses arguments and calls into `lib.rs`, where the testable code lives. (Each file in `tests/` is also a separate test binary, so many small files slow builds; put shared helpers in `tests/common/mod.rs`, which is not compiled as a test crate itself.)

## Unit 7.5 — Documentation

**One-line summary:** `///` documents the item that follows, `//!` documents the enclosing item (the crate or module). Doc comments are Markdown, can link to other items with **intra-doc links** (`` [`Inventory::add`] ``), and by convention have `# Examples`, `# Errors`, `# Panics`, and `# Safety` sections. `cargo doc --open` builds HTML for your crate and all dependencies; examples in doc comments are compiled and run by `cargo test` (lines starting with `# ` are hidden in the output but still compiled).

### Pattern

**P-7. Keep `main.rs` thin and put logic in a library; give each public item a doc comment with a tested example.** *(Practice; Rust API Guidelines C-EXAMPLE)* A thin binary makes everything testable through the library (DT-27) and lets other programs reuse it. Doctests keep documentation honest — an example that stops compiling fails the test suite — and `#![warn(missing_docs)]` in the crate root reports undocumented public items. *Trade-off:* writing docs and examples takes time; spend it on the public API, not on private helpers.

<!-- expanded -->
### Going deeper: doc tests are real tests

Code blocks in `///` comments are compiled and run by `cargo test` as **doc tests**. This keeps examples honest: an example that stops compiling after an API change fails the test suite. Lines starting with `# ` are compiled but hidden from the rendered docs, `should_panic` and `no_run` change how a block is treated, and `ignore` skips it.

```rust
// file: doctest_demo/src/lib.rs
//! Small text helpers.

/// Returns the initials of a name, uppercased.
///
/// ```
/// use doctest_demo::initials;
/// assert_eq!(initials("ada lovelace"), "AL");
/// assert_eq!(initials(""), "");
/// ```
///
/// Non-letters are skipped:
///
/// ```
/// # use doctest_demo::initials;
/// assert_eq!(initials("  jean-luc   picard "), "JP");
/// ```
pub fn initials(name: &str) -> String {
    name.split_whitespace().filter_map(|w| w.chars().next()).flat_map(char::to_uppercase).collect()
}

/// Divides, panicking on zero.
///
/// ```should_panic
/// doctest_demo::div(1, 0);
/// ```
pub fn div(a: i32, b: i32) -> i32 { a / b }
```

```toml
# file: doctest_demo/Cargo.toml
[package]
name = "doctest_demo"
version = "0.1.0"
edition = "2024"
```

```console
$ cd doctest_demo && cargo test -q --doc 2>&1 | grep -E "test result|running"
running 3 tests
test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.06s
```

Three doc tests ran: two examples and the `should_panic` block. Doc tests compile each block as a separate small crate that depends on yours, which is why they see only your *public* API and why they are slow — a large crate's doc tests can take longer than its unit tests.

### Mastery check

1. **Predict.** In a module `a::b`, a function is declared `pub(super) fn f()`. Can code in `a` call it? Code in the crate root? Code in `a::c`?
2. **Predict.** Your crate requires `regex = "1.5"`. The newest release is 1.11.1. Which version does a fresh build without a `Cargo.lock` use? And with `regex = "=1.5.4"`?

<!-- expanded -->
## Unit 7.6 — Serialisation with `serde`

**One-line summary:** `serde` separates *what* is serialised (your types, via `#[derive(Serialize, Deserialize)]`) from *the format* (JSON, TOML, YAML, MessagePack, … — each a separate crate), and attributes such as `rename_all`, `default`, `skip_serializing_if`, and `tag` adapt your types to an existing format without hand-written code.

Almost every Rust program that reads configuration or talks to a web API uses serde. The derive macros generate code that visits each field; a format crate such as `serde_json` provides the reader and writer. The generated code is as fast as a hand-written parser and checks types while reading: a string where a number belongs is an error with a line and column, not a silently wrong value.

```toml
# file: serdedemo/Cargo.toml
[package]
name = "serdedemo"
version = "0.1.0"
edition = "2024"

[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
// file: serdedemo/src/main.rs
use serde::{Deserialize, Serialize};
use std::collections::BTreeMap;

#[derive(Debug, Serialize, Deserialize)]
#[serde(rename_all = "camelCase")]                     // Rust snake_case <-> JSON camelCase
struct Service {
    name: String,
    listen_port: u16,
    #[serde(default)]                                   // missing in the input -> Default::default()
    replicas: u32,
    #[serde(default, skip_serializing_if = "Vec::is_empty")]
    tags: Vec<String>,
    health: Health,
    env: BTreeMap<String, String>,
}

#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "kind", rename_all = "lowercase")]       // an enum written as {"kind": "...", ...}
enum Health {
    Http { path: String },
    Tcp,
    None,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let input = r#"{
        "name": "api",
        "listenPort": 8080,
        "health": { "kind": "http", "path": "/healthz" },
        "env": { "RUST_LOG": "info", "MODE": "prod" }
    }"#;
    let svc: Service = serde_json::from_str(input)?;
    println!("{svc:?}");
    println!("{}", serde_json::to_string(&svc)?);

    let bad = r#"{ "name": "api", "listenPort": "eighty", "health": {"kind": "tcp"}, "env": {} }"#;
    match serde_json::from_str::<Service>(bad) {
        Ok(_) => println!("unexpected success"),
        Err(e) => println!("error: {e}"),
    }
    let too_big = r#"{ "name": "x", "listenPort": 70000, "health": {"kind": "none"}, "env": {} }"#;
    println!("error: {}", serde_json::from_str::<Service>(too_big).unwrap_err());

    let untyped: serde_json::Value = serde_json::from_str(input)?;    // when the shape is not known
    println!("health path via Value: {}", untyped["health"]["path"]);
    Ok(())
}
```

```console
$ cd serdedemo && cargo run -q
Service { name: "api", listen_port: 8080, replicas: 0, tags: [], health: Http { path: "/healthz" }, env: {"MODE": "prod", "RUST_LOG": "info"} }
{"name":"api","listenPort":8080,"replicas":0,"health":{"kind":"http","path":"/healthz"},"env":{"MODE":"prod","RUST_LOG":"info"}}
error: invalid type: string "eighty", expected u16 at line 1 column 39
error: invalid value: integer `70000`, expected u16 at line 1 column 34
health path via Value: "/healthz"
```

Read it line by line: `rename_all = "camelCase"` maps `listen_port` to `"listenPort"`; `#[serde(default)]` let the input omit `replicas` (it became 0) and `tags`; `skip_serializing_if` left the empty `tags` out of the output; the internally tagged enum read `{"kind": "http", "path": ...}` into `Health::Http`. The two bad inputs fail with the exact field problem and position — including `70000`, which is a valid JSON number but does not fit in a `u16`. `serde_json::Value` is the escape hatch for JSON of unknown shape, at the cost of checking types by hand.

### Pattern

Derive `Serialize`/`Deserialize` on plain data types that mirror the format, and convert from those into your domain types (with validation) in one place. Keeping the serde types separate stops the wire format from leaking into the rest of the program, and lets the format change without touching the domain logic.

### Deep & Tricky

**DT-28. By default, serde *ignores* unknown fields when deserialising, so a misspelt key in a config file (`"replica": 3` for `replicas`) is silently dropped and the default is used. Add `#[serde(deny_unknown_fields)]` to configuration types, where a typo should be an error.** *(Library behaviour: serde derive documentation — unknown fields are ignored unless `deny_unknown_fields` is set; it cannot be combined with `flatten`)*

```rust
// file: serdedemo/examples/unknown.rs
#![allow(dead_code)]                       // the fields are only looked at through Debug
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Lenient { #[serde(default)] replicas: u32 }

#[derive(Debug, Deserialize)]
#[serde(deny_unknown_fields)]
struct Strict { #[serde(default)] replicas: u32 }

fn main() {
    let typo = r#"{ "replica": 3 }"#;
    println!("lenient: {:?}", serde_json::from_str::<Lenient>(typo));
    println!("strict:  {}", serde_json::from_str::<Strict>(typo).unwrap_err());
}
```

```console
$ cd serdedemo && cargo run -q --example unknown
lenient: Ok(Lenient { replicas: 0 })
strict:  unknown field `replica`, expected `replicas` at line 1 column 11
```

### Mastery check

1. **Write.** A `Config` type for this TOML-like JSON: `{"database": {"url": "...", "pool_size": 10}, "features": ["a", "b"]}`, where `pool_size` defaults to 5 and unknown keys are rejected. Serialise it back and check that the round trip gives an equal value (derive `PartialEq`).
2. **Explain.** Why does `"listenPort": 70000` fail even though the JSON is valid?
3. **Predict.** With `#[serde(tag = "kind")]`, how is `Health::Tcp` written by `serde_json::to_string`?

## Unit 7.7 — Command-line programs with `clap`

**One-line summary:** `clap`'s derive API turns a struct into a command-line parser — fields become options and arguments, doc comments become help text, enums become subcommands — and it produces errors and `--help` output that follow the usual Unix conventions.

Hand-parsing `std::env::args()` is fine for one argument. Real tools need flags, defaults, typed values, validation, subcommands, and help — `clap` generates all of it from types, so the parser and the documentation cannot drift apart.

```toml
# file: clapdemo/Cargo.toml
[package]
name = "clapdemo"
version = "0.3.1"
edition = "2024"

[dependencies]
clap = { version = "4", features = ["derive"] }
```

```rust
// file: clapdemo/src/main.rs
use clap::{Parser, Subcommand, ValueEnum};
use std::path::PathBuf;

/// A tiny file tool, to show clap's derive API.
#[derive(Parser, Debug)]
#[command(version, about)]
struct Cli {
    /// Print more detail (repeat for even more: -vv)
    #[arg(short, long, action = clap::ArgAction::Count)]
    verbose: u8,

    #[command(subcommand)]
    command: Command,
}

#[derive(Subcommand, Debug)]
enum Command {
    /// Count lines, words, or bytes in files
    Count {
        /// What to count
        #[arg(short, long, value_enum, default_value_t = Unit::Lines)]
        unit: Unit,
        /// Files to read
        #[arg(required = true)]
        files: Vec<PathBuf>,
    },
    /// Print the first N lines of a file
    Head {
        #[arg(short = 'n', long, default_value_t = 10, value_parser = clap::value_parser!(u32).range(1..=1000))]
        lines: u32,
        file: PathBuf,
    },
}

#[derive(Copy, Clone, Debug, ValueEnum)]
enum Unit { Lines, Words, Bytes }

fn main() {
    let cli = Cli::parse();                      // exits with status 2 and a message on bad input
    if cli.verbose > 0 { eprintln!("parsed: {cli:?}"); }
    match cli.command {
        Command::Count { unit, files } => {
            for f in files {
                let text = std::fs::read_to_string(&f).unwrap_or_default();
                let n = match unit {
                    Unit::Lines => text.lines().count(),
                    Unit::Words => text.split_whitespace().count(),
                    Unit::Bytes => text.len(),
                };
                println!("{n:>6} {}", f.display());
            }
        }
        Command::Head { lines, file } => {
            let text = std::fs::read_to_string(&file).unwrap_or_default();
            for l in text.lines().take(lines as usize) { println!("{l}"); }
        }
    }
}
```

```console
$ cd clapdemo && cargo build -q && printf 'one two\nthree\nfour five six\n' > t.txt && ./target/debug/clapdemo count -u words t.txt && ./target/debug/clapdemo head -n 2 t.txt && ./target/debug/clapdemo --version
     6 t.txt
one two
three
clapdemo 0.3.1
$ cd clapdemo && ./target/debug/clapdemo --help
A tiny file tool, to show clap's derive API

Usage: clapdemo [OPTIONS] <COMMAND>

Commands:
  count  Count lines, words, or bytes in files
  head   Print the first N lines of a file
  help   Print this message or the help of the given subcommand(s)

Options:
  -v, --verbose...  Print more detail (repeat for even more: -vv)
  -h, --help        Print help
  -V, --version     Print version
$ cd clapdemo && ./target/debug/clapdemo count --unit chars t.txt; echo "status $?"; ./target/debug/clapdemo head -n 0 t.txt; echo "status $?"
error: invalid value 'chars' for '--unit <UNIT>'
  [possible values: lines, words, bytes]

For more information, try '--help'.
status 2
error: invalid value '0' for '--lines <LINES>': 0 is not in 1..=1000

For more information, try '--help'.
status 2
$ cd clapdemo && ./target/debug/clapdemo -vv head t.txt 2>&1 | head -1 | cut -c1-80
parsed: Cli { verbose: 2, command: Head { lines: 10, file: "t.txt" } }
```

The doc comment on `Cli` became the `about` text, the doc comments on fields became option help, `#[command(version)]` took the version from `Cargo.toml`, and the subcommands are listed with their own descriptions. Invalid input — an unknown `--unit` value, a number outside `1..=1000` — is rejected before your code runs, with exit status 2 (the Unix convention for usage errors) and a hint listing the possible values.

### Pattern

Keep `main` thin: parse with clap, then call a function that takes the parsed struct and returns `Result`. Tests can then build the struct directly, or call `Cli::try_parse_from(["prog", "count", "x"])`, which returns an error instead of exiting.

### Mastery check

1. **Write.** Add a `Grep` subcommand with a required pattern, an `-i/--ignore-case` flag, and one or more files; test it with `Cli::try_parse_from` in a `#[test]`.
2. **Explain.** Why does `clap` exit with status 2, not 1, for a bad option?
3. **Predict.** What happens if a field is `Option<u32>` without a default — is the option required?

## Chapter 7 Project — The `textkit` workspace

**Goal:** a workspace of three crates — a core library, a command-line tool, and a crate of shared test fixtures — with workspace-inherited settings, an optional feature, unit tests, integration tests, doctests, and generated documentation.

```toml
# file: textkit/Cargo.toml
[workspace]
members = ["core", "cli", "fixtures"]
resolver = "3"

[workspace.package]
version = "0.3.0"
edition = "2024"
license = "MIT OR Apache-2.0"

[workspace.dependencies]
textkit-core = { path = "core" }
textkit-fixtures = { path = "fixtures" }
```

```toml
# file: textkit/core/Cargo.toml
[package]
name = "textkit-core"
version.workspace = true
edition.workspace = true
license.workspace = true

[features]
unicode-case = []                       # extra case-folding rules, off by default

[dev-dependencies]
textkit-fixtures.workspace = true
```

```rust
// file: textkit/core/src/lib.rs
//! Text statistics: words, sentences, and readability.
//!
//! ```
//! let stats = textkit_core::Stats::of("One fish. Two fish!");
//! assert_eq!(stats.words, 4);
//! assert_eq!(stats.sentences, 2);
//! ```
#![warn(missing_docs)]

mod readability;
mod tokens;

pub use readability::flesch_reading_ease;
pub use tokens::{normalise, words};

/// Counts computed over a text.
#[derive(Debug, Clone, PartialEq, Default)]
pub struct Stats {
    /// Number of words.
    pub words: usize,
    /// Number of sentences (runs of text ending in `.`, `!`, or `?`).
    pub sentences: usize,
    /// Number of syllables, estimated from vowel groups.
    pub syllables: usize,
}

impl Stats {
    /// Computes statistics for `text`.
    ///
    /// Text with no sentence-ending punctuation counts as one sentence if it has any words.
    pub fn of(text: &str) -> Stats {
        let words: Vec<&str> = words(text).collect();
        let ends = text.chars().filter(|c| matches!(c, '.' | '!' | '?')).count();
        Stats {
            words: words.len(),
            sentences: if ends == 0 && !words.is_empty() { 1 } else { ends },
            syllables: words.iter().map(|w| readability::syllables(w)).sum(),
        }
    }
}
```

```rust
// file: textkit/core/src/tokens.rs
/// Iterates over the words of `text`: maximal runs of letters, digits, and apostrophes.
///
/// ```
/// let w: Vec<&str> = textkit_core::words("It's 9 o'clock -- isn't it?").collect();
/// assert_eq!(w, ["It's", "9", "o'clock", "isn't", "it"]);
/// ```
pub fn words(text: &str) -> impl Iterator<Item = &str> {
    text.split(|c: char| !(c.is_alphanumeric() || c == '\'')).map(|w| w.trim_matches('\'')).filter(|w| !w.is_empty())
}

/// Lower-cases a word for counting. With the `unicode-case` feature, also folds `ß` to `ss`.
///
/// ```
/// assert_eq!(textkit_core::normalise("Straße"), if cfg!(feature = "unicode-case") { "strasse" } else { "straße" });
/// ```
pub fn normalise(word: &str) -> String {
    let lower = word.to_lowercase();
    if cfg!(feature = "unicode-case") { lower.replace('ß', "ss") } else { lower }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn apostrophes_at_edges_are_trimmed() {
        assert_eq!(words("'quoted' rock'n'roll").collect::<Vec<_>>(), ["quoted", "rock'n'roll"]);
    }
}
```

```rust
// file: textkit/core/src/readability.rs
/// Estimates syllables by counting groups of vowels, with a silent-`e` rule.
pub(crate) fn syllables(word: &str) -> usize {
    let w = word.to_lowercase();
    let mut count = 0;
    let mut prev_vowel = false;
    for c in w.chars() {
        let vowel = "aeiouy".contains(c);
        if vowel && !prev_vowel {
            count += 1;
        }
        prev_vowel = vowel;
    }
    if w.ends_with('e') && !w.ends_with("le") && count > 1 {
        count -= 1;
    }
    count.max(1)
}

/// Flesch reading-ease score: higher is easier (90+ very easy, below 30 very hard).
///
/// Returns `None` for text with no words.
pub fn flesch_reading_ease(stats: &crate::Stats) -> Option<f64> {
    if stats.words == 0 {
        return None;
    }
    let w = stats.words as f64;
    Some(206.835 - 1.015 * (w / stats.sentences as f64) - 84.6 * (stats.syllables as f64 / w))
}

#[cfg(test)]
mod tests {
    #[test]
    fn syllable_estimates() {
        for (word, n) in [("cat", 1), ("table", 2), ("make", 1), ("beautiful", 3), ("rhythm", 1)] {
            assert_eq!(super::syllables(word), n, "{word}");     // private function: unit tests can see it
        }
    }
}
```

```toml
# file: textkit/fixtures/Cargo.toml
[package]
name = "textkit-fixtures"
version.workspace = true
edition.workspace = true
publish = false
```

```rust
// file: textkit/fixtures/src/lib.rs
//! Sample texts shared by the workspace's tests.

/// A very easy text.
pub const EASY: &str = "The cat sat. The dog ran. We had fun.";
/// A hard text: long sentences and long words.
pub const HARD: &str = "Notwithstanding considerable institutional reluctance, the organisation's administrative \
    infrastructure underwent comprehensive restructuring, necessitating unprecedented interdepartmental coordination.";
```

```rust
// file: textkit/core/tests/api.rs
// An integration test: sees only the public API of textkit-core.
use textkit_core::{Stats, flesch_reading_ease};
use textkit_fixtures::{EASY, HARD};

#[test]
fn easy_text_scores_higher_than_hard_text() {
    let easy = flesch_reading_ease(&Stats::of(EASY)).unwrap();
    let hard = flesch_reading_ease(&Stats::of(HARD)).unwrap();
    assert!(easy > 90.0, "easy = {easy}");
    assert!(hard < 10.0, "hard = {hard}");
}

#[test]
fn empty_text_has_no_score() {
    assert_eq!(Stats::of(""), Stats::default());
    assert_eq!(flesch_reading_ease(&Stats::of("")), None);
}
```

```toml
# file: textkit/cli/Cargo.toml
[package]
name = "textkit-cli"
version.workspace = true
edition.workspace = true

[[bin]]
name = "textkit"
path = "src/main.rs"

[dependencies]
textkit-core.workspace = true
```

```rust
// file: textkit/cli/src/main.rs
use std::collections::HashMap;
use std::io::Read;
use textkit_core::{Stats, flesch_reading_ease, normalise, words};

fn main() {
    let mut text = String::new();
    std::io::stdin().read_to_string(&mut text).expect("UTF-8 input");
    let stats = Stats::of(&text);
    let mut freq: HashMap<String, usize> = HashMap::new();
    for w in words(&text) {
        *freq.entry(normalise(w)).or_default() += 1;
    }
    let mut top: Vec<_> = freq.into_iter().collect();
    top.sort_by(|a, b| b.1.cmp(&a.1).then_with(|| a.0.cmp(&b.0)));
    println!("{} words, {} sentences, {} syllables", stats.words, stats.sentences, stats.syllables);
    match flesch_reading_ease(&stats) {
        Some(score) => println!("reading ease {score:.1}"),
        None => println!("reading ease: n/a"),
    }
    let top: Vec<String> = top.iter().take(5).map(|(w, n)| format!("{w}×{n}")).collect();
    println!("top words: {}", top.join(" "));
}
```

```console
$ cd textkit && cargo test -q --workspace 2>&1 | grep -E '^test result|running [0-9]+ tests?'
running 0 tests
test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
running 2 tests
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
running 2 tests
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
running 0 tests
test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
running 3 tests
test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
running 0 tests
test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
$ cd textkit && cargo test -q -p textkit-core --features unicode-case --doc 2>&1 | grep 'test result'
test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
$ cd textkit && echo "Die Straße ist lang. STRASSE und Straße sind gleich?" | cargo run -q -p textkit-cli && echo "Die Straße ist lang. STRASSE und Straße sind gleich?" | cargo run -q -p textkit-cli --features textkit-core/unicode-case
9 words, 2 sentences, 9 syllables
reading ease 117.7
top words: straße×2 die×1 gleich×1 ist×1 lang×1
9 words, 2 sentences, 9 syllables
reading ease 117.7
top words: strasse×3 die×1 gleich×1 ist×1 lang×1
$ cd textkit && cargo doc -q --no-deps -p textkit-core && ls target/doc/textkit_core/ | sort | head -n 6 && ls target
all.html
fn.flesch_reading_ease.html
fn.normalise.html
fn.words.html
index.html
readability
CACHEDIR.TAG  debug  doc  tmp
```

What the workspace shows:

- **One lock file, one `target/`.** All three crates build into `textkit/target`, so the shared code is compiled once.
- **Inherited settings.** `version.workspace = true` keeps all crates at one version; `[workspace.dependencies]` declares each internal crate's path once.
- **Each kind of test in its place:** unit tests next to private code (`syllables` is `pub(crate)` and tested where it lives), integration tests in `core/tests/` using only the public API plus the fixtures crate (a **dev-dependency**, never part of a normal build), and doctests in the documentation — including the one that checks both settings of the feature.
- **Features from the command line.** `--features textkit-core/unicode-case` enables a dependency's feature for that build; the output changes from treating `straße` and `strasse` as different words to counting them together.

### Review checklist

- Why is `textkit-fixtures` marked `publish = false`, and why is it a dev-dependency rather than a normal one?
- `syllables` is `pub(crate)`. What would change, for users and for semver, if it were `pub`?
- Run `cargo clippy --workspace` and `cargo doc` with `#![warn(missing_docs)]`: is every public item documented?

## Answers to the Chapter 7 prediction questions

**7.5 Q1.** `pub(super)` in `a::b` makes `f` visible in `a` and everything inside `a` — so code in `a` and in `a::c` can call it (as `super::b::f()` from `a::c`), but the crate root cannot.

**7.5 Q2.** Without a lock file, Cargo picks the newest version compatible with `^1.5`, which is 1.11.1. With `=1.5.4` it must use exactly 1.5.4. Exact pins are rarely a good idea in libraries: two crates pinning different exact versions of one dependency cannot be built together.

- **Unit 7.6, Q3:** `{"kind":"tcp"}` — a unit variant in an internally tagged enum is written as just the tag.
- **Unit 7.7, Q3:** No. An `Option<T>` field makes the argument optional; it is `None` when the option is not given.

# Chapter 8 — Smart Pointers and Interior Mutability

*Goal: go beyond single ownership when a design needs it — heap allocation with `Box`, shared ownership with `Rc`, mutation through shared references with `Cell` and `RefCell`, and non-owning links with `Weak` — and know when an index-based design is better than all of them.*

**What you will be able to do after this chapter**

- Use `Box<T>` for recursive types, trait objects, and large values.
- Implement `Deref` and `Drop`, and predict drop order.
- Share ownership with `Rc<T>`, and break cycles with `Weak<T>`.
- Mutate through shared references with `Cell`, `RefCell`, and `OnceCell`/`LazyLock`, and explain what each checks and when.
- Use `Cow` to avoid allocation when data usually needs no change.
- Choose between pointer graphs (`Rc<RefCell<T>>`) and arenas with indices.

## Topic 8A — Owning pointers

## Unit 8.1 — `Box<T>`

**One-line summary:** `Box<T>` owns a `T` on the heap; it is one pointer wide and frees the value when dropped. Use it for **recursive types** (whose size would otherwise be infinite), for **trait objects** (`Box<dyn Trait>`), and to move large values cheaply.

```rust
// file: boxes.rs
#[derive(Debug)]
enum Expr {
    Num(f64),
    Add(Box<Expr>, Box<Expr>),        // without Box, Expr would contain itself: infinite size
    Mul(Box<Expr>, Box<Expr>),
    Neg(Box<Expr>),
}

use Expr::*;

fn eval(e: &Expr) -> f64 {
    match e {
        Num(n) => *n,
        Add(a, b) => eval(a) + eval(b),
        Mul(a, b) => eval(a) * eval(b),
        Neg(a) => -eval(a),
    }
}

fn main() {
    // (2 + 3) * -4
    let e = Mul(Box::new(Add(Box::new(Num(2.0)), Box::new(Num(3.0)))), Box::new(Neg(Box::new(Num(4.0)))));
    println!("{e:?} = {}", eval(&e));
    println!("size_of: Expr {}, Box<Expr> {}, Box<[u8; 1_000_000]> {}", size_of::<Expr>(), size_of::<Box<Expr>>(), size_of::<Box<[u8; 1_000_000]>>());
    let big = vec![0u8; 1_000_000].into_boxed_slice();     // a heap slice: pointer + length
    println!("boxed slice len {} in a {}-byte handle", big.len(), size_of_val(&big));
}
```

```console
$ rustc --edition 2024 boxes.rs && ./boxes
Mul(Add(Num(2.0), Num(3.0)), Neg(Num(4.0))) = -20
size_of: Expr 24, Box<Expr> 8, Box<[u8; 1_000_000]> 8
boxed slice len 1000000 in a 16-byte handle
```

Remove the `Box`es from `Add`, and the compiler explains the problem and suggests exactly this fix (error E0072, "recursive type `Expr` has infinite size … insert some indirection").

<!-- expanded -->
### Going deeper: recursive types need a `Box`, and `Box<[T]>`

A type that contains itself directly would have infinite size, so `enum List { Cons(i32, List), Nil }` is rejected ("recursive type has infinite size"). A `Box` is one pointer of known size, which breaks the cycle. Two other uses of `Box`: moving a large value to the heap so that moving it around copies only a pointer, and `Box<[T]>` — a fixed-length heap slice, two words instead of a `Vec`'s three, with no spare capacity.

```rust
// file: boxes2.rs
#[derive(Debug)]
enum Expr {
    Num(f64),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
    Neg(Box<Expr>),
}
use Expr::*;

fn eval(e: &Expr) -> f64 {
    match e {
        Num(n) => *n,
        Add(a, b) => eval(a) + eval(b),
        Mul(a, b) => eval(a) * eval(b),
        Neg(a) => -eval(a),
    }
}

fn main() {
    let e = Add(Box::new(Num(2.0)), Box::new(Mul(Box::new(Num(3.0)), Box::new(Neg(Box::new(Num(4.0)))))));
    println!("2 + 3 * -4 = {}", eval(&e));
    println!("size_of::<Expr>() = {} bytes", std::mem::size_of::<Expr>());
    let v: Vec<u32> = (0..5).collect();
    let fixed: Box<[u32]> = v.into_boxed_slice();
    println!("Vec<u32>: {} bytes, Box<[u32]>: {} bytes, contents {fixed:?}",
             std::mem::size_of::<Vec<u32>>(), std::mem::size_of::<Box<[u32]>>());
}
```

```console
$ rustc --edition 2024 boxes2.rs && ./boxes2
2 + 3 * -4 = -10
size_of::<Expr>() = 24 bytes
Vec<u32>: 24 bytes, Box<[u32]>: 16 bytes, contents [0, 1, 2, 3, 4]
```

`Expr` is 24 bytes: the largest variant holds two 8-byte boxes, plus a tag for which variant it is.

## Unit 8.2 — `Deref`, `Drop`, and drop order

**One-line summary:** Implementing **`Deref`** lets a type be used like a reference to its target (`*x`, method calls); **deref coercion** turns `&Box<String>` into `&String` into `&str` automatically at function calls. **`Drop`** runs cleanup when a value goes out of scope. Drop order is defined: local variables in **reverse** order of declaration; struct fields in **declaration** order; elements of a `Vec` front to back.

```rust
// file: deref_drop.rs
use std::ops::Deref;

struct Loud(&'static str);
impl Drop for Loud {
    fn drop(&mut self) {
        println!("drop {}", self.0);
    }
}

#[allow(dead_code)]                     // the fields exist only to be dropped
struct Pair {
    first: Loud,
    second: Loud,
}

/// A wrapper that behaves like the String inside it.
struct Name(String);
impl Deref for Name {
    type Target = String;
    fn deref(&self) -> &String {
        &self.0
    }
}

fn greet(s: &str) -> String {
    format!("hello, {s}")
}

fn main() {
    let n = Name("Ferris".to_string());
    println!("{} (len {})", greet(&n), n.len());       // &Name -> &String -> &str; len() found through Deref
    let boxed = Box::new(String::from("boxed"));
    println!("{}", greet(&boxed));                      // &Box<String> -> &String -> &str

    let _a = Loud("local a");
    let _b = Loud("local b");
    let _p = Pair { first: Loud("field first"), second: Loud("field second") };
    let _v = vec![Loud("vec[0]"), Loud("vec[1]")];
    println!("end of main");
}
```

```console
$ rustc --edition 2024 deref_drop.rs && ./deref_drop
hello, Ferris (len 6)
hello, boxed
end of main
drop vec[0]
drop vec[1]
drop field first
drop field second
drop local b
drop local a
```

### Deep & Tricky

**DT-29. `let _ = value;` drops immediately; `let _guard = value;` lives to the end of the scope.** *(Guarantee: Rust language)* The pattern `_` does not bind, so a temporary assigned to it is dropped at the end of the statement. For a lock guard or any RAII value, `let _ = mutex.lock();` releases the lock at once — a classic bug — while `let _guard = mutex.lock();` (any name starting with `_`) keeps it until the end of the block. The same applies to temporaries in an expression statement: `mutex.lock().unwrap().push(1);` holds the lock only for that statement.

## Topic 8B — Shared ownership and interior mutability

## Unit 8.3 — `Rc<T>`: reference counting

**One-line summary:** `Rc<T>` (reference counted) allows **several owners** of one heap value. `Rc::clone(&rc)` increments a count (it does not copy the value); dropping an `Rc` decrements it; the value is dropped when the count reaches zero. `Rc` gives only **shared** (`&T`) access, and is single-threaded (it is not `Send`; `Arc` is the thread-safe version, Chapter 9).

```rust
// file: rc.rs
use std::rc::Rc;

#[derive(Debug)]
struct Texture {
    name: String,
    bytes: usize,
}

impl Drop for Texture {
    fn drop(&mut self) {
        println!("freeing texture {}", self.name);
    }
}

struct Sprite {
    texture: Rc<Texture>,                         // many sprites share one texture
    x: i32,
}

fn main() {
    let grass = Rc::new(Texture { name: "grass.png".into(), bytes: 4096 });
    println!("count after creation: {}", Rc::strong_count(&grass));
    let mut sprites: Vec<Sprite> = (0..3).map(|i| Sprite { texture: Rc::clone(&grass), x: i * 16 }).collect();
    println!("count with 3 sprites: {}", Rc::strong_count(&grass));
    drop(grass);                                  // the original handle; sprites still hold the texture
    println!("count after dropping the original: {}", Rc::strong_count(&sprites[0].texture));
    for s in &sprites {
        print!("{}@{} ", s.texture.name, s.x);
    }
    println!("({} bytes stored once)", sprites[0].texture.bytes);
    sprites.truncate(1);
    println!("count with 1 sprite: {}", Rc::strong_count(&sprites[0].texture));
    sprites.clear();                              // last owner gone: texture freed here
    println!("end of main");
}
```

```console
$ rustc --edition 2024 rc.rs && ./rc
count after creation: 1
count with 3 sprites: 4
count after dropping the original: 3
grass.png@0 grass.png@16 grass.png@32 (4096 bytes stored once)
count with 1 sprite: 1
freeing texture grass.png
end of main
```

<!-- expanded -->
### Going deeper: `Rc::make_mut` and `Rc::try_unwrap`

`Rc<T>` gives shared, read-only access. Two functions let you mutate without `RefCell` when possible: `Rc::make_mut(&mut rc)` gives `&mut T` — directly if this is the only `Rc`, or after cloning the value if it is shared (clone-on-write); `Rc::try_unwrap(rc)` gives back the `T` if this is the last `Rc`, or the `Rc` itself as an error if not.

```rust
// file: rcmut.rs
use std::rc::Rc;

fn main() {
    let mut a = Rc::new(vec![1, 2, 3]);
    Rc::make_mut(&mut a).push(4);                       // unique: modified in place
    let b = Rc::clone(&a);
    println!("before: a {a:?}, b {b:?}, count {}", Rc::strong_count(&a));
    Rc::make_mut(&mut a).push(5);                       // shared: a gets its own copy first
    println!("after:  a {a:?}, b {b:?}, a count {}, b count {}", Rc::strong_count(&a), Rc::strong_count(&b));
    println!("same allocation: {}", Rc::ptr_eq(&a, &b));

    let c = Rc::clone(&b);
    let b = Rc::try_unwrap(b).unwrap_err();             // c still points to it: cannot unwrap
    drop(c);
    let owned: Vec<i32> = Rc::try_unwrap(b).unwrap();   // last owner now: get the Vec back
    println!("unwrapped {owned:?}");
}
```

```console
$ rustc --edition 2024 rcmut.rs && ./rcmut
before: a [1, 2, 3, 4], b [1, 2, 3, 4], count 2
after:  a [1, 2, 3, 4, 5], b [1, 2, 3, 4], a count 1, b count 1
same allocation: false
unwrapped [1, 2, 3, 4]
```

This is how persistent data structures in Rust share structure cheaply and copy only what changes.

## Unit 8.4 — Interior mutability: `Cell`, `RefCell`, `OnceCell`

**One-line summary:** Sometimes a value must be modified through a shared reference — a cache inside an immutable object, a counter shared by `Rc` owners. **Interior mutability** types allow it, safely: `Cell<T>` lets you `get`/`set`/`replace` a `Copy` (or swappable) value, never handing out references to it; `RefCell<T>` hands out references but checks the borrowing rules **at run time** — `borrow()` and `borrow_mut()` panic (or `try_borrow*` return `Err`) if the rules would be broken. `OnceCell`/`LazyLock` initialise a value once.

```rust
// file: interior.rs
use std::cell::{Cell, OnceCell, RefCell};
use std::collections::HashMap;

struct Fib {
    calls: Cell<u64>,                                   // a counter changed through &self
    memo: RefCell<HashMap<u64, u64>>,                   // a cache changed through &self
}

impl Fib {
    fn get(&self, n: u64) -> u64 {                      // note: &self, not &mut self
        self.calls.set(self.calls.get() + 1);
        if n < 2 {
            return n;
        }
        if let Some(&v) = self.memo.borrow().get(&n) {  // shared borrow ends at the end of this statement
            return v;
        }
        let v = self.get(n - 1) + self.get(n - 2);      // recursion re-borrows the RefCell: fine
        self.memo.borrow_mut().insert(n, v);
        v
    }
}

fn main() {
    let f = Fib { calls: Cell::new(0), memo: RefCell::new(HashMap::new()) };
    println!("fib(90) = {} using {} calls", f.get(90), f.calls.get());

    let config: OnceCell<String> = OnceCell::new();
    let a = config.get_or_init(|| { println!("(initialising config)"); "debug=true".to_string() });
    let b = config.get_or_init(|| unreachable!());
    println!("{a} {b}");

    let shared = RefCell::new(vec![1, 2, 3]);
    let reader = shared.borrow();
    println!("try_borrow_mut while a reader exists: {:?}", shared.try_borrow_mut().map(|_| ()));
    drop(reader);
    shared.borrow_mut().push(4);
    let r = shared.borrow();
    let w = shared.borrow_mut();                        // panics: already borrowed
    println!("{r:?} {w:?}");
}
```

```console
$ rustc --edition 2024 interior.rs && ./interior
fib(90) = 2880067194370816120 using 179 calls
(initialising config)
debug=true debug=true
try_borrow_mut while a reader exists: Err(BorrowMutError)

thread 'main' (7556) panicked at interior.rs:40:20:
RefCell already borrowed
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: core::cell::panic_already_borrowed::do_panic::runtime
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panic.rs:172:21
   3: core::cell::panic_already_borrowed::do_panic
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/intrinsics/mod.rs:2450:9
   4: core::cell::panic_already_borrowed
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panic.rs:177:9
   5: <core::cell::RefCell<alloc::vec::Vec<i32>>>::borrow_mut
   6: interior::main
   7: <fn() as core::ops::function::FnOnce<()>>::call_once
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

The borrowing rules have not changed — one writer or many readers — but `RefCell` enforces them when the program runs instead of when it compiles, with a small cost (a counter per `RefCell`) and the risk of a panic. Compile-time checking is always preferable; use `RefCell` when the access pattern is correct but the compiler cannot prove it, and keep each borrow as short as possible.

### Deep & Tricky

**DT-30. A `RefCell` borrow held across a call can panic far from the bug — keep borrows in the narrowest scope.** *(Library behaviour: `RefCell`)* A `Ref` or `RefMut` guard lasts until it is dropped. `if let Some(x) = cell.borrow().get(k) { cell.borrow_mut().insert(...) }` panics, because the temporary `Ref` in the `if let` condition lives for the whole `if let` block. (Edition 2024 shortened temporary lifetimes in `if let` conditions so that this particular form now drops the guard before the `else` branch, but not before the `then` block.) The robust habit: copy what you need out of the borrow into a local, end the borrow, then mutate — as `Fib::get` does above.

## Unit 8.5 — `Weak<T>` and reference cycles

**One-line summary:** `Rc` cycles leak: two values that hold `Rc`s to each other never reach count zero. **`Weak<T>`** is a non-owning handle (created with `Rc::downgrade`) that does not keep the value alive; `upgrade()` returns `Some(Rc<T>)` if the value still exists. Owning links go one way (parent → children as `Rc`), back-links go the other way as `Weak` (child → parent).

```rust
// file: cycles.rs
use std::cell::RefCell;
use std::rc::{Rc, Weak};

struct Node {
    name: String,
    parent: RefCell<Weak<Node>>,                  // back-link: does not own
    children: RefCell<Vec<Rc<Node>>>,             // forward links: own
}

impl Drop for Node {
    fn drop(&mut self) {
        println!("drop {}", self.name);
    }
}

fn node(name: &str) -> Rc<Node> {
    Rc::new(Node { name: name.into(), parent: RefCell::new(Weak::new()), children: RefCell::new(vec![]) })
}

struct Leaky {
    name: &'static str,
    other: RefCell<Option<Rc<Leaky>>>,
}
impl Drop for Leaky {
    fn drop(&mut self) {
        println!("drop {}", self.name);
    }
}

fn main() {
    {
        let root = node("root");
        let leaf = node("leaf");
        *leaf.parent.borrow_mut() = Rc::downgrade(&root);
        root.children.borrow_mut().push(Rc::clone(&leaf));
        let parent_name = leaf.parent.borrow().upgrade().map(|p| p.name.clone());
        println!("leaf's parent: {parent_name:?}; root strong={} weak={}", Rc::strong_count(&root), Rc::weak_count(&root));
    }
    println!("-- tree scope ended (both dropped) --");
    {
        let a = Rc::new(Leaky { name: "a", other: RefCell::new(None) });
        let b = Rc::new(Leaky { name: "b", other: RefCell::new(Some(Rc::clone(&a))) });
        *a.other.borrow_mut() = Some(Rc::clone(&b));       // a -> b -> a: a cycle of strong references
        println!("a strong={} b strong={}", Rc::strong_count(&a), Rc::strong_count(&b));
    }
    println!("-- cycle scope ended (nothing dropped: leaked) --");
}
```

```console
$ rustc --edition 2024 cycles.rs && ./cycles
leaf's parent: Some("root"); root strong=1 weak=1
drop root
drop leaf
-- tree scope ended (both dropped) --
a strong=2 b strong=2
-- cycle scope ended (nothing dropped: leaked) --
```

### Pattern

**P-8. For graphs and trees with back-links, prefer an arena with indices over `Rc<RefCell<T>>`.** *(Practice)* Store all nodes in one `Vec<Node>` and refer to them by index (`usize`, or a newtype `NodeId`). There is no reference counting, no run-time borrow checking, no cycles to leak, and nodes are contiguous in memory. Mutation needs only `&mut Vec`. Deleted nodes can be handled with a free list or a generation counter (the `slotmap` crate). *Trade-off:* an index can outlive its node (a logical dangling reference, though never memory-unsafe), and every access needs the arena; `Rc<RefCell<T>>` is simpler for small, mostly-tree-shaped data.

## Unit 8.6 — `Cow`: clone on write

**One-line summary:** `Cow<'a, B>` ("clone on write") is either `Borrowed(&'a B)` or `Owned(B::Owned)`. A function that *usually* returns its input unchanged but *sometimes* must modify it can return `Cow<str>` and allocate only in the second case.

```rust
// file: cow.rs
use std::borrow::Cow;

fn escape_html(input: &str) -> Cow<'_, str> {
    if !input.contains(['<', '>', '&']) {
        return Cow::Borrowed(input);                 // common case: no allocation
    }
    let mut out = String::with_capacity(input.len() + 16);
    for c in input.chars() {
        match c {
            '<' => out.push_str("&lt;"),
            '>' => out.push_str("&gt;"),
            '&' => out.push_str("&amp;"),
            _ => out.push(c),
        }
    }
    Cow::Owned(out)
}

fn main() {
    for s in ["plain text", "a < b && c > d"] {
        let e = escape_html(s);
        let kind = match &e { Cow::Borrowed(_) => "borrowed", Cow::Owned(_) => "owned" };
        println!("{kind:>8}: {e}");
    }
}
```

```console
$ rustc --edition 2024 cow.rs && ./cow
borrowed: plain text
   owned: a &lt; b &amp;&amp; c &gt; d
```

<!-- expanded -->
### Going deeper: `Cow` in a real API

`Cow<'a, str>` ("clone on write") is either a borrowed `&'a str` or an owned `String`. A function that *usually* returns its input unchanged, but *sometimes* has to build a new string, can return `Cow` and avoid allocating in the common case.

```rust
// file: cow2.rs
use std::borrow::Cow;

fn escape_html(s: &str) -> Cow<'_, str> {
    if !s.contains(['<', '>', '&', '"']) {
        return Cow::Borrowed(s);                       // nothing to do: no allocation
    }
    let mut out = String::with_capacity(s.len() + 16);
    for c in s.chars() {
        match c {
            '<' => out.push_str("&lt;"),
            '>' => out.push_str("&gt;"),
            '&' => out.push_str("&amp;"),
            '"' => out.push_str("&quot;"),
            _ => out.push(c),
        }
    }
    Cow::Owned(out)
}

fn main() {
    for s in ["plain text", "a < b && c", r#"say "hi""#] {
        let e = escape_html(s);
        let kind = match e { Cow::Borrowed(_) => "borrowed", Cow::Owned(_) => "owned" };
        println!("{kind:8} {e}");
    }
    let mut c = escape_html("plain");
    c.to_mut().push_str("!");                           // to_mut clones a borrowed value once
    println!("{c} is now owned: {}", matches!(c, Cow::Owned(_)));
}
```

```console
$ rustc --edition 2024 cow2.rs && ./cow2
borrowed plain text
owned    a &lt; b &amp;&amp; c
owned    say &quot;hi&quot;
plain! is now owned: true
```

`String::from_utf8_lossy` in the standard library has exactly this signature: it borrows when the bytes are valid UTF-8 and allocates only when it must replace invalid sequences.

### Mastery check

1. **Predict.** `let a = Rc::new(5); let b = a.clone(); let c = Rc::clone(&b); drop(a);` — what is `Rc::strong_count(&c)`?
2. **Predict.** Does `let c = Cell::new(String::from("x")); let s = c.get();` compile?

## Chapter 8 Project — A file-system tree, two ways

**Goal:** model a directory tree in which every node knows its parent (to compute full paths) and directories know their children (to compute sizes). Build it twice — with `Rc`, `Weak`, and `RefCell`, then with an arena of indices — run the same operations on both, check that no node leaks, and compare the code and the cost.

```rust
// file: fstree/fstree.rs
use std::cell::{Cell, RefCell};
use std::rc::{Rc, Weak};
use std::time::Instant;

thread_local! {
    static LIVE: Cell<isize> = const { Cell::new(0) };     // live Rc nodes, to detect leaks
}

// ---------- Version 1: Rc / Weak / RefCell ----------
struct RcNode {
    name: String,
    size: u64,                                            // file size; 0 for directories
    parent: RefCell<Weak<RcNode>>,
    children: RefCell<Vec<Rc<RcNode>>>,
}

impl Drop for RcNode {
    fn drop(&mut self) {
        LIVE.with(|l| l.set(l.get() - 1));
    }
}

fn rc_add(parent: &Rc<RcNode>, name: &str, size: u64) -> Rc<RcNode> {
    LIVE.with(|l| l.set(l.get() + 1));
    let n = Rc::new(RcNode { name: name.into(), size, parent: RefCell::new(Rc::downgrade(parent)), children: RefCell::new(vec![]) });
    parent.children.borrow_mut().push(Rc::clone(&n));
    n
}

fn rc_path(n: &Rc<RcNode>) -> String {
    let mut parts = vec![n.name.clone()];
    let mut cur = n.parent.borrow().upgrade();
    while let Some(p) = cur {
        parts.push(p.name.clone());
        cur = p.parent.borrow().upgrade();
    }
    parts.reverse();
    parts.join("/")
}

fn rc_total(n: &RcNode) -> u64 {
    n.size + n.children.borrow().iter().map(|c| rc_total(c)).sum::<u64>()
}

// ---------- Version 2: arena + indices ----------
#[derive(Clone, Copy, PartialEq, Debug)]
struct NodeId(usize);

struct ArenaNode {
    name: String,
    size: u64,
    parent: Option<NodeId>,
    children: Vec<NodeId>,
}

#[derive(Default)]
struct Tree {
    nodes: Vec<ArenaNode>,
}

impl Tree {
    fn root(&mut self, name: &str) -> NodeId {
        self.nodes.push(ArenaNode { name: name.into(), size: 0, parent: None, children: vec![] });
        NodeId(self.nodes.len() - 1)
    }
    fn add(&mut self, parent: NodeId, name: &str, size: u64) -> NodeId {
        let id = NodeId(self.nodes.len());
        self.nodes.push(ArenaNode { name: name.into(), size, parent: Some(parent), children: vec![] });
        self.nodes[parent.0].children.push(id);
        id
    }
    fn path(&self, mut id: NodeId) -> String {
        let mut parts = vec![self.nodes[id.0].name.as_str()];
        while let Some(p) = self.nodes[id.0].parent {
            parts.push(&self.nodes[p.0].name);
            id = p;
        }
        parts.reverse();
        parts.join("/")
    }
    fn total(&self, id: NodeId) -> u64 {
        let n = &self.nodes[id.0];
        n.size + n.children.iter().map(|&c| self.total(c)).sum::<u64>()
    }
}

/// Deterministic pseudo-random tree shape shared by both versions.
fn shape(n: usize) -> Vec<(usize, u64)> {
    let mut x: u64 = 88172645463325252;
    let mut out = Vec::with_capacity(n);
    for i in 1..n {
        x ^= x << 13;
        x ^= x >> 7;
        x ^= x << 17;
        let parent = if i < 50 { (x as usize) % i } else { (x as usize) % (i / 10) };   // first 10% are directories
        let size = if i < n / 10 { 0 } else { x % 100_000 };
        out.push((parent, size));
    }
    out
}

fn main() {
    let n = 200_000;
    let plan = shape(n);

    let t = Instant::now();
    let (rc_paths, rc_sum, rc_deepest) = {
        LIVE.with(|l| l.set(l.get() + 1));
        let root = Rc::new(RcNode { name: "root".into(), size: 0, parent: RefCell::new(Weak::new()), children: RefCell::new(vec![]) });
        let mut all = vec![Rc::clone(&root)];
        for (i, &(p, size)) in plan.iter().enumerate() {
            let parent = Rc::clone(&all[p]);
            all.push(rc_add(&parent, &format!("n{}", i + 1), size));
        }
        let deepest = all.iter().max_by_key(|n| rc_path(n).len()).map(rc_path).unwrap();
        (all.iter().map(|n| rc_path(n).len()).sum::<usize>(), rc_total(&root), deepest)
    };                                                     // root and `all` dropped here
    let t_rc = t.elapsed();
    let leaked = LIVE.with(|l| l.get());

    let t = Instant::now();
    let mut tree = Tree::default();
    let root = tree.root("root");
    let mut ids = vec![root];
    for (i, &(p, size)) in plan.iter().enumerate() {
        ids.push(tree.add(ids[p], &format!("n{}", i + 1), size));
    }
    let arena_deepest = ids.iter().map(|&id| tree.path(id)).max_by_key(|p| p.len()).unwrap();
    let arena_paths: usize = ids.iter().map(|&id| tree.path(id).len()).sum();
    let arena_sum = tree.total(root);
    let t_arena = t.elapsed();

    println!("nodes {n}: total size {rc_sum} vs {arena_sum}, path chars {rc_paths} vs {arena_paths}");
    println!("deepest path: {rc_deepest}");
    println!("same results: {}", rc_sum == arena_sum && rc_paths == arena_paths && rc_deepest == arena_deepest);
    println!("Rc nodes still alive after the Rc tree was dropped: {leaked}");
    println!("time: Rc/Weak/RefCell {t_rc:.0?}, arena {t_arena:.0?}");
    println!("per-node handle size: Rc {} bytes + {} for counts, NodeId {} bytes", size_of::<Rc<RcNode>>(), 2 * size_of::<usize>(), size_of::<NodeId>());
}
```

```console
$ cd fstree && rustc --edition 2024 -O fstree.rs && ./fstree
nodes 200000: total size 9016492446 vs 9016492446, path chars 6237104 vs 6237104
deepest path: root/n1/n5/n13/n16/n25/n40/n42/n1419/n18313/n199851
same results: true
Rc nodes still alive after the Rc tree was dropped: 0
time: Rc/Weak/RefCell 143ms, arena 126ms
per-node handle size: Rc 8 bytes + 16 for counts, NodeId 8 bytes
```

Both versions compute the same sizes and paths, and the `Rc` tree freed every node: the `Weak` parent links did not keep anything alive. The arena version was faster and is simpler to reason about — no `borrow()`, no `upgrade()`, no counts — because all nodes live in one `Vec` and links are plain indices. The `Rc` version's strength is that nodes are independent values that can be handed around and outlive the tree; when the structure is one owned graph, the arena wins.

### Review checklist

- Where would a cycle of strong references appear if `parent` were `Rc<RcNode>` instead of `Weak`? What would `leaked` print?
- How would you delete a subtree in each version? What happens to `NodeId`s that pointed into it?
- Why does `rc_path` call `upgrade()` at each step, and when could it return `None`?

## Answers to the Chapter 8 prediction questions

**8.6 Q1.** 2. `a`, `b`, and `c` all point to the same allocation (count 3); dropping `a` leaves 2.

**8.6 Q2.** No. `Cell::get` requires `T: Copy`, and `String` is not `Copy` (error E0599: the method `get` exists but its trait bounds were not satisfied). Use `c.take()`, `c.replace(..)`, or `c.into_inner()`, which move values in and out without handing out references.

# Chapter 9 — Concurrency: Threads, `Send`/`Sync`, Channels, and Atomics

*Goal: write multi-threaded programs that the compiler proves free of data races — threads and scoped threads, message passing, shared state with `Arc` and `Mutex`, atomics with explicit memory ordering, and data parallelism with Rayon — and measure what parallelism actually buys.*

In C and C++, a data race is undefined behaviour that the compiler cannot detect; ThreadSanitizer finds it at run time, if the test happens to exercise it (C++ book, Unit 13.4). In safe Rust, a data race is a **compile-time error**. The same ownership and borrowing rules from Chapter 2, plus two marker traits, `Send` and `Sync`, are enough. What Rust does *not* prevent is deadlock, starvation, and logic races (two correct operations in the wrong order) — those are still yours to design away.

**What you will be able to do after this chapter**

- Spawn threads, pass data to them, collect results, and borrow local data with scoped threads.
- Explain `Send` and `Sync`, and read the errors that enforce them.
- Communicate between threads with channels, including bounded channels for back-pressure.
- Share state with `Arc<Mutex<T>>` and `RwLock`, use `Condvar`, and understand lock poisoning.
- Use atomics with the right memory ordering.
- Parallelise data processing with Rayon, and compare designs by measurement.

## Topic 9A — Threads

## Unit 9.1 — Spawning and joining threads

**One-line summary:** `std::thread::spawn(closure)` runs the closure on a new OS thread and returns a `JoinHandle<T>`; `handle.join()` waits and returns `Ok(T)`, or `Err` if the thread panicked. The closure must be `'static` — it may not borrow the spawning function's locals — so data is moved in with `move`. **Scoped threads** (`std::thread::scope`) lift that restriction: threads spawned in a scope may borrow locals, because the scope joins them all before it returns.

```rust
// file: threads.rs
use std::thread;
use std::time::Duration;

fn main() {
    let data = vec![1u64, 2, 3, 4, 5, 6, 7, 8];

    // spawn: data must be moved (or cloned) into the thread
    let owned = data.clone();
    let handle = thread::spawn(move || owned.iter().map(|x| x * x).sum::<u64>());
    println!("sum of squares from a spawned thread: {}", handle.join().unwrap());

    // scoped threads may borrow `data`: the scope joins them before returning
    let (left, right) = data.split_at(data.len() / 2);
    let (a, b) = thread::scope(|s| {
        let h1 = s.spawn(|| left.iter().sum::<u64>());
        let h2 = s.spawn(|| right.iter().sum::<u64>());
        (h1.join().unwrap(), h2.join().unwrap())
    });
    println!("halves {a} + {b} = {}", a + b);

    // a panicking thread does not take down the process; join reports it
    let bad = thread::spawn(|| {
        thread::sleep(Duration::from_millis(10));
        let v: Vec<i32> = Vec::new();
        v[0]
    });
    match bad.join() {
        Ok(v) => println!("got {v}"),
        Err(e) => println!("thread panicked: {:?}", e.downcast_ref::<String>()),
    }
    println!("available parallelism: {}", thread::available_parallelism().unwrap());
}
```

```console
$ rustc --edition 2024 threads.rs && ./threads 2>&1
sum of squares from a spawned thread: 204
halves 10 + 26 = 36

thread '<unnamed>' (7666) panicked at threads.rs:26:10:
index out of bounds: the len is 0 but the index is 0
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: core::panicking::panic_bounds_check
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:271:5
   3: <usize as core::slice::index::SliceIndex<[i32]>>::index
   4: <alloc::vec::Vec<i32> as core::ops::index::Index<usize>>::index
   5: threads::main::{closure#2}
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
thread panicked: Some("index out of bounds: the len is 0 but the index is 0")
available parallelism: 2
```

Remove `move` from the first `spawn`, and the compiler refuses: the closure may outlive `main`'s locals, so it cannot borrow `owned` (error E0373, "closure may outlive the current function, but it borrows `owned`"), and it suggests `move`.

<!-- expanded -->
### Going deeper: scoped threads

`thread::spawn` requires its closure to be `'static`, because the new thread may outlive the function that started it — so it cannot borrow local variables. `std::thread::scope` (Rust 1.63) creates threads that are guaranteed to finish before the scope ends, which lets them borrow freely, including mutably borrowing *different* parts of the same data.

```rust
// file: scoped.rs
use std::thread;

fn main() {
    let data: Vec<u64> = (1..=1_000_000).collect();
    let mut partials = [0u64; 4];
    thread::scope(|s| {
        for (chunk, slot) in data.chunks(data.len() / 4).zip(partials.iter_mut()) {
            s.spawn(move || *slot = chunk.iter().sum());        // borrows data and one slot
        }
    });                                                         // all threads joined here
    println!("partials {partials:?}, total {}", partials.iter().sum::<u64>());

    let mut words = vec!["b", "a"];
    thread::scope(|s| {
        s.spawn(|| words.sort());                               // a mutable borrow, safely
    });
    println!("{words:?}");
}
```

```console
$ rustc --edition 2024 -O scoped.rs && ./scoped
partials [31250125000, 93750125000, 156250125000, 218750125000], total 500000500000
["a", "b"]
```

No `Arc`, no `Mutex`, no `clone`: the borrow checker proves each thread has its own slot, and the scope proves the borrows end in time.

## Unit 9.2 — `Send` and `Sync`

**One-line summary:** A type is **`Send`** if a value of it can be moved to another thread, and **`Sync`** if it can be *shared* between threads by reference (`T: Sync` exactly when `&T: Send`). Both are **auto traits**: the compiler implements them for every type whose fields are all `Send`/`Sync`. `thread::spawn` requires its closure and result to be `Send`. `Rc<T>` (non-atomic count) is neither; `Arc<T>` is both when `T` is; `Cell`/`RefCell` are `Send` but not `Sync`; `Mutex<T>` is `Sync` when `T` is `Send`.

```rust
// file: notsend.rs
use std::rc::Rc;
use std::thread;

fn main() {
    let counter = Rc::new(5);
    let c = Rc::clone(&counter);
    let h = thread::spawn(move || println!("{c}"));
    h.join().unwrap();
}
```

```console
$ rustc --edition 2024 notsend.rs 2>&1 | grep -E '^error|cannot be sent|within|required by' | head -n 5 
error[E0277]: `Rc<i32>` cannot be sent between threads safely
  |             |             `Rc<i32>` cannot be sent between threads safely
  |             |             within this `{closure@notsend.rs:8:27: 8:34}`
  |             required by a bound introduced by this call
  = help: within `{closure@notsend.rs:8:27: 8:34}`, the trait `Send` is not implemented for `Rc<i32>`
```

The error is exact: `Rc<i32>` cannot be sent between threads safely, because its reference count is updated without atomic instructions — two threads cloning the same `Rc` would race on the count. The fix is `Arc`, whose count is atomic. This is the whole mechanism by which Rust prevents data races: every way of reaching data from two threads goes through a type that is `Sync`, and only types that synchronise internally (or are immutable) are.

### Deep & Tricky

**DT-31. `Send` and `Sync` are checked only at thread boundaries — but they are inferred from every field, so one non-`Sync` field anywhere makes the whole type non-`Sync`.** *(Guarantee: Rust language, auto traits)* A struct with a `Cell<u32>` cache field cannot be shared across threads, even if no method touches the cache concurrently. The error appears at the `spawn` (or `Arc` + `spawn`) site, often several layers away, with a "within this type" chain explaining which field is to blame. Raw pointers are neither `Send` nor `Sync` by default; types built on them (like `Vec`) opt back in with `unsafe impl Send` — a promise checked by humans, not the compiler (Chapter 11).

## Topic 9B — Communication and shared state

## Unit 9.3 — Channels

**One-line summary:** `std::sync::mpsc::channel()` returns a `(Sender<T>, Receiver<T>)` pair: multiple producers (clone the sender), single consumer. `send` transfers ownership of the value to the receiving thread — no sharing, so no locks. Iterating over the receiver ends when every sender has been dropped. `sync_channel(n)` creates a **bounded** channel whose `send` blocks when `n` messages are waiting — back-pressure that keeps a fast producer from exhausting memory.

```rust
// file: channels.rs
use std::sync::mpsc;
use std::thread;
use std::time::{Duration, Instant};

fn main() {
    let (tx, rx) = mpsc::channel::<(usize, u64)>();
    for id in 0..3 {
        let tx = tx.clone();                              // one sender per producer
        thread::spawn(move || {
            for i in 0..3u64 {
                tx.send((id, i * 10 + id as u64)).unwrap();
                thread::sleep(Duration::from_millis(5));
            }
        });                                               // this producer's sender dropped here
    }
    drop(tx);                                             // drop the original, or the loop never ends
    let mut got: Vec<(usize, u64)> = rx.iter().collect(); // ends when all senders are gone
    got.sort();
    println!("received {} messages: {got:?}", got.len());

    // bounded channel: the producer can be at most 2 items ahead of a slow consumer
    let (tx, rx) = mpsc::sync_channel::<u32>(2);
    let start = Instant::now();
    let producer = thread::spawn(move || {
        for i in 0..6 {
            tx.send(i).unwrap();
            println!("  sent {i} at {:>3} ms", start.elapsed().as_millis() / 10 * 10);
        }
    });
    for v in rx {
        thread::sleep(Duration::from_millis(50));         // slow consumer
        println!("  handled {v} at {:>3} ms", start.elapsed().as_millis() / 10 * 10);
    }
    producer.join().unwrap();
}
```

```console
$ rustc --edition 2024 channels.rs && ./channels
received 9 messages: [(0, 0), (0, 10), (0, 20), (1, 1), (1, 11), (1, 21), (2, 2), (2, 12), (2, 22)]
  sent 0 at   0 ms
  sent 1 at   0 ms
  sent 2 at   0 ms
  handled 0 at  50 ms
  sent 3 at  50 ms
  handled 1 at 100 ms
  sent 4 at 100 ms
  handled 2 at 150 ms
  sent 5 at 150 ms
  handled 3 at 200 ms
  handled 4 at 250 ms
  handled 5 at 300 ms
```

With the bounded channel, the first sends complete immediately (the buffer has room); after that each `send` waits until the consumer takes an item, so the producer runs at the consumer's pace. (Timings are rounded to 10 ms.)

<!-- expanded -->
### Going deeper: bounded channels and back-pressure

`mpsc::channel()` is unbounded: a fast producer can fill memory with messages a slow consumer has not read. `mpsc::sync_channel(n)` holds at most `n` messages, and `send` blocks when it is full — **back-pressure** that slows the producer to the consumer's speed. With `n = 0` it is a *rendezvous* channel: every `send` waits for a matching `recv`.

```rust
// file: bounded.rs
use std::sync::mpsc;
use std::thread;
use std::time::{Duration, Instant};

fn main() {
    let (tx, rx) = mpsc::sync_channel::<u32>(2);
    let start = Instant::now();
    let producer = thread::spawn(move || {
        for i in 0..6 {
            tx.send(i).unwrap();                     // blocks while 2 messages are waiting
        }
        start.elapsed()
    });
    for v in rx.iter() {
        thread::sleep(Duration::from_millis(50));    // a slow consumer
        print!("{v} ");
    }
    let produced_in = producer.join().unwrap();
    println!("\nproducer was held back: {}", produced_in >= Duration::from_millis(150));

    let (tx, rx) = mpsc::channel::<String>();
    drop(rx);                                        // receiver gone
    println!("send to a closed channel: {:?}", tx.send("x".into()).is_err());
}
```

```console
$ rustc --edition 2024 bounded.rs && ./bounded
0 1 2 3 4 5 
producer was held back: true
send to a closed channel: true
```

`rx.iter()` ends when every `Sender` has been dropped — here when the producer thread finished — which is the usual way to shut a pipeline down.

## Unit 9.4 — `Arc`, `Mutex`, `RwLock`, and `Condvar`

**One-line summary:** To share mutable state, wrap it in a `Mutex<T>` and share the mutex with `Arc` (atomic reference counting). `lock()` returns a `MutexGuard` that dereferences to `T` and **unlocks when dropped** — the data is reachable *only* through the lock. `RwLock<T>` allows many readers or one writer. `Condvar` lets a thread sleep until another signals that a condition may have changed. If a thread panics while holding a lock, the lock is **poisoned**: later `lock()` calls return `Err` (with access to the data via `into_inner`), so other threads learn that an invariant may be broken.

```rust
// file: shared.rs
use std::collections::VecDeque;
use std::sync::{Arc, Condvar, Mutex, RwLock};
use std::thread;

fn main() {
    // Arc<Mutex<T>>: four threads increment one counter
    let counter = Arc::new(Mutex::new(0u64));
    let handles: Vec<_> = (0..4)
        .map(|_| {
            let c = Arc::clone(&counter);
            thread::spawn(move || {
                for _ in 0..100_000 {
                    *c.lock().unwrap() += 1;               // guard dropped at end of statement
                }
            })
        })
        .collect();
    for h in handles {
        h.join().unwrap();
    }
    println!("counter = {} (Arc strong count now {})", *counter.lock().unwrap(), Arc::strong_count(&counter));

    // RwLock: many readers at once
    let config = Arc::new(RwLock::new(String::from("v1")));
    thread::scope(|s| {
        for i in 0..3 {
            let cfg = &config;
            s.spawn(move || println!("reader {i} sees {}", cfg.read().unwrap()));
        }
    });
    *config.write().unwrap() = "v2".into();
    println!("after write: {}", config.read().unwrap());

    // Mutex + Condvar: a blocking queue
    let queue = Arc::new((Mutex::new(VecDeque::new()), Condvar::new()));
    let q2 = Arc::clone(&queue);
    let consumer = thread::spawn(move || {
        let (lock, ready) = &*q2;
        let mut taken = Vec::new();
        while taken.len() < 5 {
            let mut q = ready.wait_while(lock.lock().unwrap(), |q: &mut VecDeque<i32>| q.is_empty()).unwrap();
            taken.extend(q.drain(..));
        }
        taken
    });
    for i in 1..=5 {
        let (lock, ready) = &*queue;
        lock.lock().unwrap().push_back(i * i);
        ready.notify_one();
    }
    println!("consumer took {:?}", consumer.join().unwrap());

    // poisoning
    let m = Arc::new(Mutex::new(vec![1, 2, 3]));
    let m2 = Arc::clone(&m);
    let _ = thread::spawn(move || {
        let mut v = m2.lock().unwrap();
        v.push(4);
        panic!("invariant broken halfway through an update");
    })
    .join();
    match m.lock() {
        Ok(v) => println!("not poisoned: {v:?}"),
        Err(poisoned) => println!("poisoned! data was left as {:?}", poisoned.into_inner()),
    }
}
```

```console
$ rustc --edition 2024 shared.rs && ./shared 2>&1 | grep -v -E '^note:|^$'
counter = 400000 (Arc strong count now 1)
reader 0 sees v1
reader 1 sees v1
reader 2 sees v1
after write: v2
consumer took [1, 4, 9, 16, 25]
thread '<unnamed>' (7768) panicked at shared.rs:60:9:
invariant broken halfway through an update
stack backtrace:
   0: __rustc::rust_begin_unwind
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/std/src/panicking.rs:689:5
   1: core::panicking::panic_fmt
             at /rustc/2d8144b7880597b6e6d3dfd63a9a9efae3f533d3/library/core/src/panicking.rs:80:14
   2: shared::main::{closure#3}
poisoned! data was left as [1, 2, 3, 4]
```

`wait_while` re-checks the condition after every wake-up, so spurious wake-ups and missed notifications are handled — the hand-written `while (cond) wait();` loop of C and C++ (C book, Unit 9.3) built into the API.

### Deep & Tricky

**DT-32. Rust prevents data races, not deadlocks — and a `MutexGuard` held across an `await` or a long call is the usual way to create one.** *(Guarantee: Rust language; Library behaviour)* Two threads locking two mutexes in opposite orders deadlock in Rust exactly as in C (C book, Unit 9.4). Lock in a consistent global order, keep critical sections short, and never call unknown code (callbacks, `await`) while holding a guard. A subtle form: `let x = m.lock().unwrap().get(&k).cloned(); m.lock()` is fine (the first guard is a temporary, dropped at the `;`), but `match m.lock().unwrap().get(&k) { ... m.lock() ... }` deadlocks, because the first guard lives until the end of the `match` (DT-29). Rust's `Mutex` is not re-entrant: locking it twice on one thread deadlocks or panics.

<!-- expanded -->
### Deep & Tricky

**DT-33. A thread that panics while holding a `Mutex` *poisons* it: every later `lock()` returns `Err(PoisonError)`, so the common `lock().unwrap()` turns one thread's panic into a panic in every thread that touches the lock. The data may be half-updated; if you know it is still consistent, `into_inner()` on the error recovers the guard.** *(Guarantee: `std::sync::Mutex` documentation, "Poisoning"; `parking_lot` and Tokio mutexes do not poison)*

```rust
// file: poison.rs
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let balance = Arc::new(Mutex::new(100));
    let b = Arc::clone(&balance);
    let r = thread::spawn(move || {
        let mut g = b.lock().unwrap();
        *g -= 30;
        panic!("crashed in the middle of a transfer");      // the +30 elsewhere never happens
    })
    .join();
    println!("worker panicked: {}", r.is_err());
    match balance.lock() {
        Ok(g) => println!("balance {}", *g),
        Err(poisoned) => {
            println!("mutex is poisoned");
            let g = poisoned.into_inner();                   // choose to look anyway
            println!("half-updated balance: {}", *g);
        }
    }
    balance.clear_poison();
    println!("after clear_poison: is_poisoned = {}", balance.is_poisoned());
}
```

```console
$ rustc --edition 2024 poison.rs && ./poison 2>/dev/null
worker panicked: true
mutex is poisoned
half-updated balance: 70
after clear_poison: is_poisoned = false
```

## Unit 9.5 — Atomics and memory ordering

**One-line summary:** `std::sync::atomic` provides `AtomicBool`, `AtomicUsize`, `AtomicU64`, `AtomicPtr`, and friends, with `load`, `store`, `fetch_add`, `swap`, and `compare_exchange`. Each operation takes an `Ordering`: `Relaxed` (atomic, but no ordering with other memory), `Release` (on a store: earlier writes become visible to whoever `Acquire`-loads this value), `Acquire`, `AcqRel`, and `SeqCst` (a single global order). The model is C++20's (C++ book, Unit 10.4).

```rust
// file: atomics.rs
use std::sync::Arc;
use std::sync::atomic::{AtomicBool, AtomicU64, Ordering};
use std::thread;
use std::time::Instant;

static mut PAYLOAD: [u64; 4] = [0; 4];                 // plain memory published by a flag
static READY: AtomicBool = AtomicBool::new(false);

fn main() {
    // 1. Relaxed counter: atomicity is all we need
    let hits = Arc::new(AtomicU64::new(0));
    let t = Instant::now();
    thread::scope(|s| {
        for _ in 0..4 {
            s.spawn(|| for _ in 0..1_000_000 { hits.fetch_add(1, Ordering::Relaxed); });
        }
    });
    println!("atomic counter: {} in {:.0?}", hits.load(Ordering::Relaxed), t.elapsed());

    // 2. Release/Acquire: publish data written non-atomically
    let writer = thread::spawn(|| {
        unsafe { *(&raw mut PAYLOAD) = [1, 2, 3, 4] };           // ordinary writes...
        READY.store(true, Ordering::Release);                    // ...published by the release store
    });
    let reader = thread::spawn(|| {
        while !READY.load(Ordering::Acquire) {                   // acquire pairs with the release
            std::hint::spin_loop();
        }
        unsafe { *(&raw const PAYLOAD) }                         // guaranteed to see [1, 2, 3, 4]
    });
    writer.join().unwrap();
    println!("reader saw {:?}", reader.join().unwrap());

    // 3. compare_exchange: a lock-free maximum
    let max = AtomicU64::new(0);
    thread::scope(|s| {
        for t in 0..4u64 {
            let max = &max;
            s.spawn(move || {
                for i in 0..1000u64 {
                    let v = (i * 7919 + t * 104_729) % 100_003;
                    let mut cur = max.load(Ordering::Relaxed);
                    while v > cur {
                        match max.compare_exchange_weak(cur, v, Ordering::Relaxed, Ordering::Relaxed) {
                            Ok(_) => break,
                            Err(actual) => cur = actual,              // someone else changed it: retry
                        }
                    }
                }
            });
        }
    });
    println!("max = {} (fetch_max would do this in one call)", max.load(Ordering::Relaxed));
}
```

```console
$ rustc --edition 2024 -O atomics.rs && ./atomics
atomic counter: 4000000 in 78ms
reader saw [1, 2, 3, 4]
max = 99984 (fetch_max would do this in one call)
```

The `static mut` in part 2 needs `unsafe`, because the compiler cannot see that the flag protects it — exactly the kind of reasoning `unsafe` exists for (Chapter 11). In real code, use a type that encodes the protocol: `OnceLock` for publish-once data, a `Mutex`, or a channel.

<!-- expanded -->
### Going deeper: a spin lock, and why `Acquire`/`Release` are enough

A lock built on an `AtomicBool` shows the orderings at work. Taking the lock with `compare_exchange(false, true, Acquire, Relaxed)` makes everything the previous holder wrote visible; releasing with `store(false, Release)` publishes this holder's writes to the next. Real code should use `Mutex` — a spin lock wastes CPU while waiting and behaves badly when the holder is descheduled — but the pattern is the core of every lock.

```rust
// file: spinlock.rs
use std::cell::UnsafeCell;
use std::sync::atomic::{AtomicBool, Ordering::{Acquire, Relaxed, Release}};
use std::thread;

struct SpinLock<T> { locked: AtomicBool, value: UnsafeCell<T> }
unsafe impl<T: Send> Sync for SpinLock<T> {}               // we guarantee exclusive access ourselves

impl<T> SpinLock<T> {
    const fn new(v: T) -> Self { SpinLock { locked: AtomicBool::new(false), value: UnsafeCell::new(v) } }
    fn with<R>(&self, f: impl FnOnce(&mut T) -> R) -> R {
        while self.locked.compare_exchange_weak(false, true, Acquire, Relaxed).is_err() {
            std::hint::spin_loop();
        }
        let r = f(unsafe { &mut *self.value.get() });      // exclusive while locked is true
        self.locked.store(false, Release);
        r
    }
}

static COUNTER: SpinLock<u64> = SpinLock::new(0);

fn main() {
    thread::scope(|s| {
        for _ in 0..4 {
            s.spawn(|| for _ in 0..100_000 { COUNTER.with(|c| *c += 1) });
        }
    });
    println!("counter = {}", COUNTER.with(|c| *c));
}
```

```console
$ rustc --edition 2024 -O spinlock.rs && ./spinlock
counter = 400000
```

The `unsafe impl Sync` is the promise the compiler cannot check: that `with` never hands out two `&mut T` at once. If `f` panicked, this lock would stay locked forever — a real implementation releases in a `Drop` guard.

## Topic 9C — Data parallelism

## Unit 9.6 — Rayon

**One-line summary:** The **Rayon** crate turns iterator pipelines into parallel ones: change `.iter()` to `.par_iter()` (or `into_par_iter`, `par_chunks`, `par_sort`), and Rayon splits the work across a global thread pool using **work stealing**. Because Rayon's closures must be `Send + Sync`, the compiler rejects pipelines that would race.

```toml
# file: par/Cargo.toml
[package]
name = "par"
version = "0.1.0"
edition = "2024"

[dependencies]
rayon = "1"
```

```rust
// file: par/src/main.rs
use rayon::prelude::*;
use std::time::Instant;

fn is_prime(n: u64) -> bool {
    n >= 2 && (2..).take_while(|d| d * d <= n).all(|d| n % d != 0)
}

fn main() {
    let limit = 3_000_000u64;
    let t = Instant::now();
    let seq = (0..limit).filter(|&n| is_prime(n)).count();
    let t_seq = t.elapsed();
    let t = Instant::now();
    let par = (0..limit).into_par_iter().filter(|&n| is_prime(n)).count();
    let t_par = t.elapsed();
    println!("primes below {limit}: {seq} sequential in {t_seq:.0?}, {par} parallel in {t_par:.0?} on {} threads ({:.2}x)",
             rayon::current_num_threads(), t_seq.as_secs_f64() / t_par.as_secs_f64());

    let mut v: Vec<u64> = (0..10_000_000).map(|i: u64| i.wrapping_mul(0x9E3779B97F4A7C15) >> 20).collect();
    let mut w = v.clone();
    let t = Instant::now();
    v.sort_unstable();
    let t_sort = t.elapsed();
    let t = Instant::now();
    w.par_sort_unstable();
    println!("sort 10M: {t_sort:.0?} sequential, {:.0?} parallel, equal: {}", t.elapsed(), v == w);
}
```

```console
$ cd par && cargo run -q --release
primes below 3000000: 216816 sequential in 517ms, 216816 parallel in 310ms on 2 threads (1.67x)
sort 10M: 198ms sequential, 168ms parallel, equal: true
```

On this two-core machine, the CPU-bound prime count doubled in speed, while the parallel sort gained nothing: sorting moves a lot of memory relative to its computation, and Rayon's parallel quicksort partitions sequentially at the top level before the halves can run in parallel. The upper limit is the number of cores; the actual gain depends on how much of the work is parallel and how much is memory bandwidth (Amdahl's law, and the C book's Chapter 9 measurements).

### Pattern

**P-9. Prefer, in order: no sharing (move data to threads, collect results), message passing, then shared state behind a lock — and reach for atomics only for simple counters and flags, or with a proven algorithm.** *(Practice)* Each step down the list adds coordination cost and ways to get it wrong. Data parallelism with Rayon or scoped threads over disjoint slices needs no synchronisation at all. Channels make ownership transfer explicit. Locks are simple but serialise. Hand-written lock-free code with orderings weaker than `SeqCst` is very hard to get right; use a well-tested crate (`crossbeam`, `dashmap`, `arc-swap`) instead. *Trade-off:* the more independent the work, the less code shares state — so design for independence first.

<!-- expanded -->
### Going deeper: when Rayon does not help

Rayon splits work across threads, which costs scheduling and, for small or memory-bound work, can make a loop *slower*. Parallelism pays off when each element does real computation; summing a small array is not that.

```rust
// file: par/examples/when.rs
use rayon::prelude::*;
use std::time::Instant;

fn time<F: FnOnce() -> u64>(label: &str, f: F) -> f64 {
    let t = Instant::now();
    let r = f();
    let ms = t.elapsed().as_secs_f64() * 1000.0;
    std::hint::black_box(r);
    let _ = label;
    ms
}

fn heavy(x: u64) -> u64 { (0..2000).fold(x, |a, i| a.wrapping_mul(6364136223846793005).wrapping_add(i)) }

fn main() {
    let small: Vec<u64> = (0..1000).collect();
    let big: Vec<u64> = (0..200_000).collect();
    let s_seq = time("small seq", || small.iter().sum());
    let s_par = time("small par", || small.par_iter().sum());
    let h_seq = time("heavy seq", || big.iter().map(|&x| heavy(x)).fold(0, u64::wrapping_add));
    let h_par = time("heavy par", || big.par_iter().map(|&x| heavy(x)).reduce(|| 0, u64::wrapping_add));
    println!("threads: {}", rayon::current_num_threads());
    println!("tiny sum: parallel {} than sequential", if s_par > s_seq { "slower" } else { "not slower" });
    println!("heavy work: parallel speed-up {:.1}x", h_seq / h_par);
}
```

```console
$ cd par && cargo run -q --release --example when
threads: 2
tiny sum: parallel slower than sequential
heavy work: parallel speed-up 2.0x
```

On the book's machine the parallel sum of 1000 numbers was slower than the plain loop, while the heavy per-element work sped up roughly in proportion to the number of threads (2.0 times with 2 threads).

### Mastery check

1. **Predict.** Does this compile? `let v = vec![1, 2, 3]; std::thread::spawn(|| println!("{:?}", v)).join().unwrap();`
2. **Predict.** Is `Arc<RefCell<i32>>` `Send`? Can it be moved into a spawned thread?

## Chapter 9 Project — Five ways to count words in parallel

**Goal:** count word frequencies in a 40 MB text using all cores, five different ways — a single-threaded baseline, one shared `Mutex<HashMap>`, per-thread maps merged at the end, a pipeline with channels, and Rayon's `fold`/`reduce` — verify that all five produce identical counts, and measure them. The point is not the fastest word counter but seeing how the *design of sharing* determines performance.

```toml
# file: wc5/Cargo.toml
[package]
name = "wc5"
version = "0.1.0"
edition = "2024"

[dependencies]
rayon = "1"

[profile.release]
debug = false
```

```rust
// file: wc5/src/main.rs
use rayon::prelude::*;
use std::collections::HashMap;
use std::sync::{Arc, Mutex, mpsc};
use std::thread;
use std::time::Instant;

type Counts = HashMap<String, u64>;

fn words(text: &str) -> impl Iterator<Item = &str> {
    text.split(|c: char| !c.is_ascii_alphabetic()).filter(|w| !w.is_empty())
}

fn count_into(map: &mut Counts, text: &str) {
    for w in words(text) {
        match map.get_mut(w) {                                    // avoid allocating a key for known words
            Some(n) => *n += 1,
            None => { map.insert(w.to_ascii_lowercase(), 1); }
        }
    }
}

/// Splits text into `n` chunks at whitespace, so no word is cut in two.
fn chunks(text: &str, n: usize) -> Vec<&str> {
    let mut out = Vec::new();
    let mut start = 0;
    for i in 1..=n {
        let mut end = if i == n { text.len() } else { (text.len() * i / n).max(start) };
        while end < text.len() && !text.as_bytes()[end].is_ascii_whitespace() {
            end += 1;
        }
        out.push(&text[start..end]);
        start = end;
    }
    out
}

fn merge(mut a: Counts, b: Counts) -> Counts {
    for (w, n) in b {
        *a.entry(w).or_insert(0) += n;
    }
    a
}

fn sequential(text: &str) -> Counts {
    let mut m = Counts::new();
    count_into(&mut m, text);
    m
}

fn shared_mutex(text: &str, threads: usize) -> Counts {
    let map = Arc::new(Mutex::new(Counts::new()));
    thread::scope(|s| {
        for chunk in chunks(text, threads) {
            let map = Arc::clone(&map);
            s.spawn(move || {
                for w in words(chunk) {
                    let mut m = map.lock().unwrap();             // one lock per word
                    match m.get_mut(w) {
                        Some(n) => *n += 1,
                        None => { m.insert(w.to_ascii_lowercase(), 1); }
                    }
                }
            });
        }
    });
    Arc::into_inner(map).unwrap().into_inner().unwrap()
}

fn per_thread_merge(text: &str, threads: usize) -> Counts {
    thread::scope(|s| {
        let handles: Vec<_> = chunks(text, threads).into_iter().map(|c| s.spawn(move || sequential(c))).collect();
        handles.into_iter().map(|h| h.join().unwrap()).fold(Counts::new(), merge)
    })
}

fn channel_pipeline(text: &str, threads: usize) -> Counts {
    let (tx, rx) = mpsc::sync_channel::<Counts>(threads);
    thread::scope(|s| {
        for chunk in chunks(text, threads * 8) {              // more, smaller jobs
            let tx = tx.clone();
            s.spawn(move || tx.send(sequential(chunk)).unwrap());
        }
        drop(tx);
        rx.iter().fold(Counts::new(), merge)                   // the merging thread runs concurrently
    })
}

fn rayon_fold(text: &str) -> Counts {
    chunks(text, rayon::current_num_threads() * 8)
        .into_par_iter()
        .map(sequential)
        .reduce(Counts::new, merge)
}

fn main() {
    let path = std::env::args().nth(1).expect("usage: wc5 FILE");
    let raw = std::fs::read_to_string(&path).expect("text file");
    let text = raw.to_ascii_lowercase();                       // normalise once, up front
    let threads = thread::available_parallelism().unwrap().get();
    println!("{:.1} MB, {threads} threads", text.len() as f64 / 1e6);

    let mut results: Vec<(&str, Counts)> = Vec::new();
    let mut base = 0.0;
    let runs: Vec<(&str, Box<dyn Fn(&str) -> Counts>)> = vec![
        ("sequential", Box::new(sequential)),
        ("shared Mutex<HashMap>", Box::new(move |t| shared_mutex(t, threads))),
        ("per-thread maps + merge", Box::new(move |t| per_thread_merge(t, threads))),
        ("channel pipeline", Box::new(move |t| channel_pipeline(t, threads))),
        ("rayon map/reduce", Box::new(rayon_fold)),
    ];
    for (name, f) in runs {
        let t = Instant::now();
        let counts = f(&text);
        let ms = t.elapsed().as_secs_f64() * 1e3;
        if base == 0.0 { base = ms; }
        println!("{name:<25} {ms:>7.0} ms  {:>5.2}x", base / ms);
        results.push((name, counts));
    }
    let same = results.iter().all(|(_, c)| *c == results[0].1);
    let total: u64 = results[0].1.values().sum();
    let mut top: Vec<_> = results[0].1.iter().collect();
    top.sort_by(|a, b| b.1.cmp(a.1).then(a.0.cmp(b.0)));
    println!("all five agree: {same}; {total} words, {} distinct; top: {:?}", results[0].1.len(), &top[..3]);
}
```

The input is generated with a Zipf-distributed vocabulary, like real text (the C++ book's Chapter 12 used the same idea):

```rust
// file: wc5/gen.rs
fn main() {
    let mut x: u64 = 0x9E3779B97F4A7C15;
    let mut next = move || { x ^= x << 13; x ^= x >> 7; x ^= x << 17; x };
    let vocab: Vec<String> = (0..30_000)
        .map(|_| (0..2 + next() % 8).map(|_| (b'a' + (next() % 26) as u8) as char).collect())
        .collect();
    let harmonic: f64 = (1..=vocab.len()).map(|k| 1.0 / k as f64).sum();
    let mut out = String::new();
    while out.len() < 40_000_000 {
        let mut u = (next() % 1_000_000_000) as f64 / 1e9 * harmonic;   // inverse-CDF sampling of Zipf
        let mut k = 0;
        while k + 1 < vocab.len() && u > 1.0 / (k + 1) as f64 {
            u -= 1.0 / (k + 1) as f64;
            k += 1;
        }
        out.push_str(&vocab[k]);
        out.push(if next() % 12 == 0 { '\n' } else { ' ' });
    }
    print!("{out}");
}
```

```console
$ cd wc5 && rustc --edition 2024 -O gen.rs && ./gen > corpus.txt && cargo run -q --release -- corpus.txt
40.0 MB, 2 threads
sequential                    205 ms   1.00x
shared Mutex<HashMap>        2213 ms   0.09x
per-thread maps + merge       119 ms   1.73x
channel pipeline              175 ms   1.17x
rayon map/reduce              158 ms   1.30x
all five agree: true; 6037191 words, 26583 distinct; top: [("kkymhdm", 554774), ("srzng", 276697), ("hmnbpljks", 184517)]
$ cd wc5 && cargo run -q --release -- corpus.txt | tail -n 6
sequential                    191 ms   1.00x
shared Mutex<HashMap>        1879 ms   0.10x
per-thread maps + merge       133 ms   1.43x
channel pipeline              296 ms   0.65x
rayon map/reduce              289 ms   0.66x
all five agree: true; 6037191 words, 26583 distinct; top: [("kkymhdm", 554774), ("srzng", 276697), ("hmnbpljks", 184517)]
```

Reading the results (two cores, so 2× is the ceiling; the two runs above and four more varied by up to 30% on this virtual machine, so compare designs, not digits):

- **Shared `Mutex<HashMap>` was by far the slowest design — four to seven times slower than one thread.** Every word takes and releases the lock, so the threads spend their time contending for one cache line and waiting, and the critical section is the whole of the work. Adding threads to a design whose work is all inside one lock makes it slower, not faster.
- **Per-thread maps merged at the end** removed the sharing: each thread counts its own half with no synchronisation, and the only serial part is one merge. It was consistently the fastest, typically 1.7× the sequential time (and once, measured against a slow sequential run, above 2×).
- **The channel pipeline and Rayon were barely faster than one thread — sometimes slower.** This was not the expected result, and the reason is in the code: both split the text into `threads * 8` = 16 chunks to balance load, and with a Zipf vocabulary *each* chunk contains most of the 26,583 distinct words. So each chunk allocates its own ~25,000 `String` keys, and the final merge processes about 400,000 entries — serial work that the per-thread version, with only two chunks, does an eighth as often. Finer-grained tasks balance load better, but here the per-task overhead (a full partial map) dominated. Changing `* 8` to `* 1` brought both into the same range as the per-thread version in three trial runs (118–239 ms, noisy), as the review checklist invites you to verify.
- **All five produce identical counts.** The compiler guaranteed the absence of data races in every version (try removing the `Mutex` from `shared_mutex`: it does not compile), but performance came entirely from the design: what is shared, and how much partial work must be combined.

### Review checklist

- In `shared_mutex`, the lock is taken once per word. Lock once per 1,000 words instead (buffer locally, then flush). How close does it get to `per_thread_merge`?
- Why does `chunks` move each boundary forward to whitespace? What would happen to the counts without it?
- `channel_pipeline` uses `sync_channel(threads)`. What is the largest number of partial maps that can exist at once?
- Change the chunk multiplier in `channel_pipeline` and `rayon_fold` from 8 to 1, and to 64. Plot time against the number of chunks.

## Answers to the Chapter 9 prediction questions

**9.6 Q1.** No. The closure borrows `v`, but `thread::spawn` requires a `'static` closure, since the thread could outlive `v` — error E0373, with the suggestion to add `move`. (`thread::scope` would allow the borrow.)

**9.6 Q2.** No. `Arc<T>` is `Send` only if `T` is `Send + Sync`, and `RefCell<i32>` is not `Sync` (its borrow counter is not atomic). Moving it into `spawn` fails to compile; use `Arc<Mutex<i32>>` (or an `AtomicI32`).

# Chapter 10 — Async Rust: Futures, Executors, and Tokio

*Goal: write programs that handle thousands of concurrent I/O operations with a handful of threads — understand what a `Future` is and how an executor drives it, use `async`/`await` with the Tokio runtime, handle timeouts and cancellation, and avoid the classic async mistakes.*

Threads are the right tool for CPU-bound parallel work (Chapter 9). For **I/O-bound** concurrency — a server with ten thousand mostly idle connections, a crawler with a thousand requests in flight — one OS thread per task costs too much memory and too many context switches. **Async** Rust lets one thread interleave many tasks: each task runs until it would wait for I/O, then yields so another can run. The compiler turns each `async fn` into a state machine, so a waiting task costs only the bytes of its state, not a whole thread stack.

**What you will be able to do after this chapter**

- Explain the `Future` trait, `poll`, `Poll::Pending`, and wakers, and build a minimal executor.
- Write `async fn`s and `async` blocks, and run them on Tokio.
- Run tasks concurrently with `spawn`, `join!`, and `select!`, with timeouts.
- Write async TCP servers and clients, and communicate between tasks with async channels.
- Explain cancellation by dropping, and avoid blocking the executor.

## Topic 10A — How async works

## Unit 10.1 — Futures and polling

**One-line summary:** A **future** is a value representing a computation that may not have finished. Its one method, `poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<Output>`, makes as much progress as it can and returns `Poll::Ready(value)` or `Poll::Pending`. A future that returns `Pending` must arrange for `cx.waker().wake()` to be called when it can make progress; an **executor** calls `poll` again after a wake. `async fn` and `async { }` produce futures automatically; `.await` polls an inner future and returns `Pending` up the chain until it is ready.

To make this concrete, here is a complete executor and a timer future, using only the standard library:

```rust
// file: mini_executor.rs
use std::future::Future;
use std::pin::{Pin, pin};
use std::sync::{Arc, Mutex};
use std::task::{Context, Poll, Wake, Waker};
use std::thread;
use std::time::{Duration, Instant};

/// A future that completes after a delay, using a helper thread to call wake().
struct Delay {
    when: Instant,
    started: bool,
    name: &'static str,
}

impl Future for Delay {
    type Output = &'static str;
    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<&'static str> {
        println!("  poll({}) at {:>3} ms", self.name, ms_since_start());
        if Instant::now() >= self.when {
            return Poll::Ready(self.name);
        }
        if !self.started {
            self.started = true;
            let waker = cx.waker().clone();
            let when = self.when;
            thread::spawn(move || {
                thread::sleep(when.saturating_duration_since(Instant::now()));
                waker.wake();                                  // "poll me again"
            });
        }
        Poll::Pending
    }
}

/// The executor's waker unparks the executor thread.
struct ThreadWaker(thread::Thread);
impl Wake for ThreadWaker {
    fn wake(self: Arc<Self>) {
        self.0.unpark();
    }
}

/// Runs one future to completion on the current thread.
fn block_on<F: Future>(fut: F) -> F::Output {
    let mut fut = pin!(fut);
    let waker = Waker::from(Arc::new(ThreadWaker(thread::current())));
    let mut cx = Context::from_waker(&waker);
    loop {
        match fut.as_mut().poll(&mut cx) {
            Poll::Ready(v) => return v,
            Poll::Pending => thread::park(),                   // sleep until woken
        }
    }
}

static START: Mutex<Option<Instant>> = Mutex::new(None);
fn ms_since_start() -> u128 {
    START.lock().unwrap().unwrap().elapsed().as_millis() / 10 * 10
}

async fn two_steps() -> String {
    let a = Delay { when: Instant::now() + Duration::from_millis(100), started: false, name: "first" }.await;
    let b = Delay { when: Instant::now() + Duration::from_millis(50), started: false, name: "second" }.await;
    format!("{a} then {b}")
}

fn main() {
    *START.lock().unwrap() = Some(Instant::now());
    let fut = two_steps();                                     // nothing runs yet
    println!("future created; size of its state machine: {} bytes", size_of_val(&fut));
    let result = block_on(fut);
    println!("result: {result} at {} ms", ms_since_start());
}
```

```console
$ rustc --edition 2024 mini_executor.rs && ./mini_executor
future created; size of its state machine: 64 bytes
  poll(first) at   0 ms
  poll(first) at 100 ms
  poll(second) at 100 ms
  poll(second) at 150 ms
result: first then second at 150 ms
```

The trace shows the protocol: each `Delay` is polled once, returns `Pending`, and is polled again only after its waker fires; the `async fn` resumes exactly where it left off. The whole `two_steps` computation, including both delays and the local variables that live across `.await`s, is a state machine of a few dozen bytes. Real executors (Tokio, smol, embassy for embedded) do the same with many futures at once, using the operating system's readiness APIs (`epoll`, `kqueue`, IOCP) instead of a thread per timer.

### Deep & Tricky

**DT-34. Futures are lazy: calling an `async fn` does nothing until the future is polled — and a future that is never awaited never runs.** *(Guarantee: Rust language)* Unlike JavaScript promises or C# tasks, which start running when created, a Rust future is inert until an executor polls it. `send_email(msg);` without `.await` silently does nothing (the compiler warns: "unused implementer of `Future` that must be used"). This is also what makes Rust futures cheap and cancellable: dropping an unpolled or partly polled future simply discards its state (Unit 10.5).

**DT-35. A future may be moved before it is first polled but not after — which is what `Pin` expresses.** *(Guarantee: Rust language, `Pin`)* An `async fn`'s state machine can contain references to its own local variables (a borrow that lives across an `.await`). Moving the state machine after polling would invalidate those internal pointers, so `poll` takes `Pin<&mut Self>`: a pointer that promises the value will not move again. You rarely touch `Pin` directly — `.await`, `tokio::spawn`, `Box::pin`, and the `pin!` macro handle it — but it is why some futures must be pinned (`Box::pin(fut)`) before being stored or polled in a loop.

## Topic 10B — Tokio

## Unit 10.2 — The Tokio runtime and tasks

**One-line summary:** **Tokio** is the most widely used async runtime: a multi-threaded work-stealing executor, async TCP/UDP/Unix sockets, timers, channels, synchronisation primitives, and file and process APIs. `#[tokio::main]` starts the runtime; `tokio::spawn(future)` runs a **task** concurrently (it must be `Send + 'static`, like a thread); `join!` runs several futures concurrently on the current task and waits for all; `select!` waits for the first to finish; `tokio::time::timeout` bounds any future.

```toml
# file: tok/Cargo.toml
[package]
name = "tok"
version = "0.1.0"
edition = "2024"

[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
// file: tok/src/main.rs
use std::time::{Duration, Instant};
use tokio::time::{sleep, timeout};

async fn fetch(name: &str, ms: u64) -> String {
    sleep(Duration::from_millis(ms)).await;                // a stand-in for network I/O
    format!("{name}({ms}ms)")
}

#[tokio::main]
async fn main() {
    let t = Instant::now();
    let a = fetch("a", 100).await;                          // sequential: one after another
    let b = fetch("b", 100).await;
    println!("sequential: {a} {b} in {} ms", t.elapsed().as_millis() / 10 * 10);

    let t = Instant::now();
    let (a, b, c) = tokio::join!(fetch("a", 100), fetch("b", 100), fetch("c", 150));   // concurrent
    println!("join!: {a} {b} {c} in {} ms", t.elapsed().as_millis() / 10 * 10);

    let t = Instant::now();
    let handles: Vec<_> = (0..10_000).map(|i| tokio::spawn(async move { sleep(Duration::from_millis(200)).await; i })).collect();
    let mut sum = 0u64;
    for h in handles {
        sum += h.await.unwrap();
    }
    println!("10,000 spawned tasks each sleeping 200 ms: sum {sum} in {} ms", t.elapsed().as_millis() / 10 * 10);

    tokio::select! {
        r = fetch("fast", 50) => println!("select!: {r} won"),
        r = fetch("slow", 500) => println!("select!: {r} won"),
    }
    match timeout(Duration::from_millis(80), fetch("too slow", 300)).await {
        Ok(v) => println!("got {v}"),
        Err(_) => println!("timeout: gave up after 80 ms"),
    }
}
```

```console
$ cd tok && cargo run -q --release
sequential: a(100ms) b(100ms) in 200 ms
join!: a(100ms) b(100ms) c(150ms) in 150 ms
10,000 spawned tasks each sleeping 200 ms: sum 49995000 in 200 ms
select!: fast(50ms) won
timeout: gave up after 80 ms
```

Ten thousand concurrent tasks, each waiting 200 ms, finished in a little over 200 ms on a two-core machine. With OS threads, that would mean ten thousand stacks (8 MiB of address space each by default) and ten thousand kernel scheduler entries; as Tokio tasks, each is a small heap allocation holding its state machine.

<!-- expanded -->
### Going deeper: `join!`, `JoinSet`, and timeouts

Three tools cover most "run several things" needs. `tokio::join!` runs a fixed number of futures concurrently *in the current task* and waits for all. A `JoinSet` holds any number of spawned tasks and yields their results as they finish. `tokio::time::timeout` wraps any future and gives up after a deadline.

```rust
// file: tok/examples/concurrency.rs
use std::time::Duration;
use tokio::task::JoinSet;
use tokio::time::{sleep, timeout, Instant};

async fn fetch(id: u32, ms: u64) -> String {
    sleep(Duration::from_millis(ms)).await;          // stands in for a network call
    format!("response {id}")
}

#[tokio::main]
async fn main() {
    let t = Instant::now();
    let (a, b) = tokio::join!(fetch(1, 100), fetch(2, 100));
    println!("{a}, {b}: concurrent ({} ms, not 200)", t.elapsed().as_millis() / 100 * 100);

    let mut set = JoinSet::new();
    for (id, ms) in [(1, 300), (2, 100), (3, 200)] {
        set.spawn(fetch(id, ms));
    }
    let mut order = vec![];
    while let Some(res) = set.join_next().await {
        order.push(res.unwrap());
    }
    println!("completion order: {order:?}");

    match timeout(Duration::from_millis(50), fetch(9, 1000)).await {
        Ok(r) => println!("{r}"),
        Err(_) => println!("fetch 9 timed out after 50 ms"),
    }
}
```

```console
$ cd tok && cargo run -q --example concurrency
response 1, response 2: concurrent (100 ms, not 200)
completion order: ["response 2", "response 3", "response 1"]
fetch 9 timed out after 50 ms
```

Dropping a `JoinSet` aborts every task still in it — a convenient way to make sure no background work outlives the function that started it.

## Unit 10.3 — Async I/O: a TCP server

**One-line summary:** Tokio's `TcpListener::accept().await` and the `AsyncReadExt`/`AsyncWriteExt` methods (`read`, `read_exact`, `write_all`) look like blocking code but yield to other tasks while waiting. The standard server shape is an accept loop that `spawn`s one task per connection.

```rust
// file: echo/src/main.rs
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};
use tokio::net::{TcpListener, TcpStream};

async fn handle(stream: TcpStream, id: usize) -> std::io::Result<()> {
    let (reader, mut writer) = stream.into_split();
    let mut lines = BufReader::new(reader).lines();
    while let Some(line) = lines.next_line().await? {
        writer.write_all(format!("[{id}] {}\n", line.to_uppercase()).as_bytes()).await?;
    }
    Ok(())                                                 // client closed the connection
}

#[tokio::main]
async fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:7878").await?;
    eprintln!("listening on {}", listener.local_addr()?);
    for id in 1.. {
        let (stream, _peer) = listener.accept().await?;
        tokio::spawn(async move {
            if let Err(e) = handle(stream, id).await {
                eprintln!("connection {id}: {e}");
            }
        });
    }
    Ok(())
}
```

```toml
# file: echo/Cargo.toml
[package]
name = "echo"
version = "0.1.0"
edition = "2024"

[dependencies]
tokio = { version = "1", features = ["full"] }
```

```console
$ cd echo && cargo build -q --release && (./target/release/echo 2> server.log & echo $! > server.pid) && sleep 0.5 && printf 'hello\nasync world\n' | nc -q 1 127.0.0.1 7878 && printf 'second client\n' | nc -q 1 127.0.0.1 7878; kill $(cat server.pid); cat server.log
[1] HELLO
[1] ASYNC WORLD
[2] SECOND CLIENT
listening on 127.0.0.1:7878
```

### Deep & Tricky

**DT-36. Blocking inside an async task stalls every task scheduled on that worker thread.** *(Runtime behaviour: cooperative scheduling)* Tasks yield only at `.await` points. A `std::thread::sleep`, a synchronous file read, a CPU-heavy loop, or a `std::sync::Mutex` lock held while waiting blocks the worker thread, and every other task on it waits too. Use the async equivalents (`tokio::time::sleep`, `tokio::fs`), move blocking or CPU-heavy work to `tokio::task::spawn_blocking` (or Rayon), and hold a `std::sync::Mutex` only for short, non-`await`ing critical sections — use `tokio::sync::Mutex` only when the guard must be held across an `.await`.

```rust
// file: tok/examples/blocking.rs
use std::time::{Duration, Instant};

async fn ticker(start: Instant) {
    for _ in 0..4 {
        tokio::time::sleep(Duration::from_millis(100)).await;
        println!("  tick at {:>3} ms", start.elapsed().as_millis() / 10 * 10);
    }
}

#[tokio::main(flavor = "current_thread")]                 // one worker thread makes the effect obvious
async fn main() {
    let start = Instant::now();
    println!("with std::thread::sleep(350 ms) in another task:");
    let t = tokio::spawn(ticker(start));
    tokio::spawn(async { std::thread::sleep(Duration::from_millis(350)); });   // blocks the only thread
    t.await.unwrap();

    let start = Instant::now();
    println!("with spawn_blocking:");
    let t = tokio::spawn(ticker(start));
    tokio::task::spawn_blocking(|| std::thread::sleep(Duration::from_millis(350)));
    t.await.unwrap();
}
```

```console
$ cd tok && cargo run -q --release --example blocking
with std::thread::sleep(350 ms) in another task:
  tick at 350 ms
  tick at 450 ms
  tick at 550 ms
  tick at 650 ms
with spawn_blocking:
  tick at 100 ms
  tick at 200 ms
  tick at 300 ms
  tick at 400 ms
```

With the blocking sleep, the ticker's first tick arrives at 350 ms instead of 100 — the only worker thread was asleep, so the timer could not even be checked — and every later tick shifts with it. With `spawn_blocking`, the sleep runs on a separate thread pool and the ticks arrive every 100 ms.

## Unit 10.4 — Channels and shared state between tasks

**One-line summary:** `tokio::sync` provides async versions of the Chapter 9 tools: `mpsc` (bounded, multi-producer), `oneshot` (a single reply — ideal for request/response between tasks), `broadcast` (every receiver gets every message), `watch` (receivers see the latest value), plus `Mutex`, `RwLock`, `Semaphore`, and `Notify`. A common architecture gives each piece of state to one **actor** task that owns it and receives commands over a channel — no locks at all.

```rust
// file: tok/examples/actor.rs
use std::collections::HashMap;
use tokio::sync::{mpsc, oneshot};

enum Command {
    Incr { key: String, by: i64 },
    Get { key: String, reply: oneshot::Sender<Option<i64>> },
}

/// The actor owns the map; nobody else can touch it.
async fn store(mut rx: mpsc::Receiver<Command>) {
    let mut map: HashMap<String, i64> = HashMap::new();
    while let Some(cmd) = rx.recv().await {
        match cmd {
            Command::Incr { key, by } => *map.entry(key).or_default() += by,
            Command::Get { key, reply } => { let _ = reply.send(map.get(&key).copied()); }
        }
    }
}

#[tokio::main]
async fn main() {
    let (tx, rx) = mpsc::channel(64);
    tokio::spawn(store(rx));
    let mut workers = Vec::new();
    for w in 0..8 {
        let tx = tx.clone();
        workers.push(tokio::spawn(async move {
            for i in 0..1000 {
                tx.send(Command::Incr { key: format!("k{}", i % 3), by: w }).await.unwrap();
            }
        }));
    }
    for w in workers {
        w.await.unwrap();
    }
    for key in ["k0", "k1", "k2", "missing"] {
        let (reply, answer) = oneshot::channel();
        tx.send(Command::Get { key: key.into(), reply }).await.unwrap();
        println!("{key} = {:?}", answer.await.unwrap());
    }
}
```

```console
$ cd tok && cargo run -q --release --example actor
k0 = Some(9352)
k1 = Some(9324)
k2 = Some(9324)
missing = None
```

<!-- expanded -->
### Going deeper: `tokio::sync::Mutex` versus `std::sync::Mutex`

In async code, the default should still be `std::sync::Mutex` for short critical sections that do not `.await` while locked: it is faster and simpler. Tokio's `Mutex` exists for the case where you *must* hold the lock across an `.await` (for example, around a shared connection), because its `lock()` is itself a future that yields instead of blocking the worker thread. Holding a `std` guard across an `.await` is a compile error in spawned tasks (the guard is not `Send`) — the compiler stops the deadlock-prone version.

```rust
// file: tok/examples/locks.rs
use std::sync::{Arc, Mutex};
use std::time::Duration;

#[tokio::main]
async fn main() {
    let hits = Arc::new(Mutex::new(0u64));                 // std Mutex: never held across .await
    let mut tasks = vec![];
    for _ in 0..100 {
        let hits = Arc::clone(&hits);
        tasks.push(tokio::spawn(async move {
            tokio::time::sleep(Duration::from_millis(1)).await;
            *hits.lock().unwrap() += 1;                     // lock, update, unlock: no await inside
        }));
    }
    for t in tasks { t.await.unwrap(); }
    println!("hits = {}", *hits.lock().unwrap());

    let conn = Arc::new(tokio::sync::Mutex::new(Vec::<String>::new()));   // held across an await
    let c = Arc::clone(&conn);
    tokio::spawn(async move {
        let mut log = c.lock().await;
        tokio::time::sleep(Duration::from_millis(10)).await;              // fine with tokio's Mutex
        log.push("written while holding the lock".into());
    })
    .await
    .unwrap();
    println!("{:?}", conn.lock().await);
}
```

```console
$ cd tok && cargo run -q --example locks
hits = 100
["written while holding the lock"]
$ cd tok && sed 's|\*hits.lock().unwrap() += 1;                     // lock, update, unlock: no await inside|let mut g = hits.lock().unwrap(); tokio::time::sleep(Duration::from_millis(1)).await; *g += 1;|' examples/locks.rs > examples/locks_bad.rs && cargo build -q --example locks_bad 2>&1 | grep -m1 -oE "error: .*"; rm -f examples/locks_bad.rs
error: future cannot be sent between threads safely
```

## Unit 10.5 — Cancellation

**One-line summary:** In Rust, **cancelling a future means dropping it**. `select!` drops the branches that lose; `timeout` drops the inner future when time runs out; dropping a `JoinHandle` does *not* cancel a spawned task, but `handle.abort()` does. When a future is dropped at an `.await` point, its locals are dropped normally (destructors run), but whatever it had not yet done is simply not done.

### Deep & Tricky

**DT-37. In `select!`, a branch's future is dropped when another wins — so a future that has partly completed its work loses that work.** *(Library behaviour: `tokio::select!`; Guarantee: drop semantics)* Reading a message from a socket with `read_exact` inside a `select!` loop alongside a timer is a classic bug: if the timer wins halfway through a read, the bytes already read are lost with the dropped future, and the stream is out of sync. Tokio documents each method's **cancellation safety**: `mpsc::Receiver::recv`, `TcpListener::accept`, and `AsyncReadExt::read` are cancel-safe; `read_exact`, `read_to_end`, and `write_all` are not. In loops, create the future once outside the `select!` (and pin it) or use cancel-safe operations.

### Pattern

**P-10. Use async for many concurrent waits; use threads (or `spawn_blocking`/Rayon) for computation; and give each piece of shared state one owner task.** *(Practice)* Async pays off when tasks spend most of their time waiting on I/O or timers. It adds complexity — `Send` bounds on spawned futures, pinning, cancellation semantics, "function colouring" (async functions can only be awaited from async code) — that is not worth paying for CPU-bound programs or for a few dozen connections. Within async code, an actor that owns its state and receives messages avoids most locking questions. *Trade-off:* actors add a channel hop per operation and make request/response slightly more verbose than a shared `Mutex`.

<!-- expanded -->
### Going deeper: graceful shutdown with a cancellation signal

A server should stop accepting work on Ctrl-C (or `SIGTERM`), let in-flight work finish, and exit. The usual shape: a `watch` or broadcast channel that every task listens to in a `select!`, and the main task waiting for the signal, sending the shutdown, then joining the workers.

```rust
// file: tok/examples/shutdown.rs
use std::time::Duration;
use tokio::sync::watch;
use tokio::task::JoinSet;

async fn worker(id: u32, mut stop: watch::Receiver<bool>) -> u32 {
    let mut jobs = 0;
    loop {
        tokio::select! {
            _ = tokio::time::sleep(Duration::from_millis(20)) => jobs += 1,   // "process a job"
            _ = stop.changed() => break,                                      // shutdown requested
        }
    }
    println!("worker {id} finishing after {} jobs", if jobs > 0 { "some" } else { "no" });
    jobs
}

#[tokio::main]
async fn main() {
    let (tx, rx) = watch::channel(false);
    let mut set = JoinSet::new();
    for id in 0..3 { set.spawn(worker(id, rx.clone())); }

    tokio::select! {
        _ = tokio::signal::ctrl_c() => println!("Ctrl-C received"),
        _ = tokio::time::sleep(Duration::from_millis(200)) => println!("demo: shutting down after 200 ms"),
    }
    tx.send(true).unwrap();
    let mut done = 0;
    while let Some(r) = set.join_next().await { r.unwrap(); done += 1; }
    println!("{done} workers stopped cleanly");
}
```

```console
$ cd tok && cargo run -q --example shutdown | sort
3 workers stopped cleanly
demo: shutting down after 200 ms
worker 0 finishing after some jobs
worker 1 finishing after some jobs
worker 2 finishing after some jobs
```

(The output is sorted because the three workers finish in an unpredictable order.) For larger programs the `tokio-util` crate's `CancellationToken` packages the same idea, with child tokens for sub-tasks.

### Mastery check

1. **Predict.** In `#[tokio::main] async fn main() { let f = async { println!("hi") }; }`, is anything printed?
2. **Predict.** Two `tokio::spawn`ed tasks each call `std::thread::sleep(1 s)`. On the default multi-threaded runtime with 2 worker threads, how long until both finish? On a `current_thread` runtime?

## Chapter 10 Project — An async chat server with a load test

**Goal:** a TCP chat server — clients connect, choose a name, and every line one client sends is broadcast to all others — built with Tokio tasks and a `broadcast` channel; plus a load-test client in the same binary that opens hundreds of concurrent connections, sends messages, verifies that every message reaches every other client, and reports throughput and latency.

```toml
# file: chat/Cargo.toml
[package]
name = "chat"
version = "0.1.0"
edition = "2024"

[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
// file: chat/src/main.rs
use std::sync::Arc;
use std::sync::atomic::{AtomicUsize, Ordering};
use std::time::{Duration, Instant};
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};
use tokio::net::{TcpListener, TcpStream};
use tokio::sync::broadcast;

#[derive(Clone, Debug)]
struct Msg {
    from: usize,                                    // connection id, so senders skip their own messages
    text: Arc<str>,                                 // shared, not copied, across all receivers
}

async fn serve(addr: &str) -> std::io::Result<()> {
    let listener = TcpListener::bind(addr).await?;
    let (tx, _) = broadcast::channel::<Msg>(4096);
    let online = Arc::new(AtomicUsize::new(0));
    eprintln!("chat server on {addr}");
    for id in 0usize.. {
        let (stream, _) = listener.accept().await?;
        let (tx, mut rx, online) = (tx.clone(), tx.subscribe(), Arc::clone(&online));
        tokio::spawn(async move {
            let (r, mut w) = stream.into_split();
            let mut lines = BufReader::new(r).lines();
            let name = match lines.next_line().await {          // first line: the user's name
                Ok(Some(n)) if !n.trim().is_empty() => n.trim().to_string(),
                _ => return,
            };
            let n = online.fetch_add(1, Ordering::SeqCst) + 1;
            let _ = w.write_all(format!("* welcome {name}, {n} online\n").as_bytes()).await;
            loop {
                tokio::select! {
                    line = lines.next_line() => match line {          // next_line is cancel-safe
                        Ok(Some(text)) => { let _ = tx.send(Msg { from: id, text: format!("{name}: {text}").into() }); }
                        _ => break,                                   // disconnect or error
                    },
                    msg = rx.recv() => match msg {
                        Ok(m) if m.from != id => {
                            if w.write_all(m.text.as_bytes()).await.is_err() || w.write_all(b"\n").await.is_err() { break; }
                        }
                        Ok(_) => {}
                        Err(broadcast::error::RecvError::Lagged(k)) => {
                            let _ = w.write_all(format!("* you missed {k} messages\n").as_bytes()).await;
                        }
                        Err(broadcast::error::RecvError::Closed) => break,
                    },
                }
            }
            online.fetch_sub(1, Ordering::SeqCst);
        });
    }
    Ok(())
}

/// Opens `clients` connections; each sends `per_client` messages; checks everyone receives everyone else's.
async fn load_test(addr: &str, clients: usize, per_client: usize) {
    let start = Instant::now();
    let mut conns = Vec::new();
    for c in 0..clients {
        let stream = TcpStream::connect(addr).await.expect("connect");
        stream.set_nodelay(true).unwrap();
        let (r, mut w) = stream.into_split();
        w.write_all(format!("user{c}\n").as_bytes()).await.unwrap();
        let mut lines = BufReader::new(r).lines();
        lines.next_line().await.unwrap();                         // the welcome line
        conns.push((lines, w));
    }
    let connected = start.elapsed();
    let expected = (clients - 1) * per_client;                    // messages each client should receive
    let start = Instant::now();
    let mut tasks = Vec::new();
    for (c, (mut lines, mut w)) in conns.into_iter().enumerate() {
        tasks.push(tokio::spawn(async move {
            let reader = tokio::spawn(async move {
                let mut got = 0;
                while got < expected {
                    match tokio::time::timeout(Duration::from_secs(10), lines.next_line()).await {
                        Ok(Ok(Some(l))) if !l.starts_with('*') => got += 1,
                        Ok(Ok(Some(_))) => {}
                        _ => break,
                    }
                }
                got
            });
            for i in 0..per_client {
                w.write_all(format!("message {i} from {c}\n").as_bytes()).await.unwrap();
                tokio::time::sleep(Duration::from_millis(2)).await;    // a human-ish pace, so the
            }                                                          // 4096-message buffer is not overrun
            let got = reader.await.unwrap();
            drop(w);
            got
        }));
    }
    let mut received = 0;
    let mut complete = 0;
    for t in tasks {
        let got = t.await.unwrap();
        received += got;
        complete += usize::from(got == expected);
    }
    let elapsed = start.elapsed();
    println!("{clients} clients connected in {connected:.0?}; {} messages sent, {received} deliveries in {elapsed:.0?} ({:.0} deliveries/s)",
             clients * per_client, received as f64 / elapsed.as_secs_f64());
    println!("clients that received all {expected} messages from the others: {complete}/{clients}");
}

#[tokio::main]
async fn main() {
    let args: Vec<String> = std::env::args().collect();
    match args.get(1).map(String::as_str) {
        Some("serve") => serve("127.0.0.1:7979").await.unwrap(),
        Some("load") => {
            let clients = args.get(2).and_then(|s| s.parse().ok()).unwrap_or(100);
            let per = args.get(3).and_then(|s| s.parse().ok()).unwrap_or(10);
            load_test("127.0.0.1:7979", clients, per).await;
        }
        _ => eprintln!("usage: chat serve | chat load CLIENTS MESSAGES"),
    }
}
```

```console
$ cd chat && cargo build -q --release && (./target/release/chat serve 2> server.log & echo $! > server.pid) && sleep 0.5 && cat server.log
chat server on 127.0.0.1:7979
$ cd chat && ((printf 'ada\nhello everyone\n'; sleep 1.5) | nc -q 1 127.0.0.1 7979 > ada.out &); sleep 0.3; (printf 'grace\n'; sleep 0.5; printf 'hi ada\n'; sleep 0.5) | nc -q 1 127.0.0.1 7979; sleep 1.5; echo '--- ada saw:'; cat ada.out
* welcome grace, 2 online
--- ada saw:
* welcome ada, 1 online
grace: hi ada
$ cd chat && ./target/release/chat load 200 20 && ./target/release/chat load 500 10
200 clients connected in 95ms; 4000 messages sent, 796000 deliveries in 614ms (1295829 deliveries/s)
clients that received all 3980 messages from the others: 200/200
500 clients connected in 123ms; 5000 messages sent, 2144088 deliveries in 11s (187892 deliveries/s)
clients that received all 4990 messages from the others: 0/500
$ cd chat && kill $(cat server.pid)
```

The server runs one task per connection, each waiting on two things at once with `select!`: a line from its client, or a message from the broadcast channel. Messages are `Arc<str>`, so a line broadcast to hundreds of clients is stored once. Two small things in the manual test: Grace never saw Ada's "hello everyone", because it was sent before Grace connected — a broadcast channel delivers only to current subscribers, with no history — and Ada's own message was not echoed back to her (`m.from != id`).

The two load tests tell different stories:

- **200 clients × 20 messages:** every one of the 200 clients received all 3,980 messages from the others — 796,000 deliveries in about 0.6 s, over a million per second on two cores, with each connection costing a task and two small buffers, not a thread.
- **500 clients × 10 messages: not one client received everything.** 500 senders at one message per 2 ms put 250,000 messages per second into the channel, each to be written to 499 sockets — more than two cores can deliver. Receivers fell more than 4,096 messages behind, the channel told them so with `Lagged` (which the load test skips, as it skips all `*` notices), and their readers then waited out the 10-second timeout for messages that had been dropped. This is the bounded buffer doing its job: a server that cannot keep up sheds load for the slow receivers instead of growing its memory without limit. Whether dropping is acceptable is a product decision; the alternatives are a larger buffer (more memory, more latency), per-client queues with disconnection of laggards, or back-pressure on senders.

Design points:

- **`broadcast` with a bounded buffer** decouples fast and slow clients: a client that falls more than 4,096 messages behind gets a `Lagged` notice instead of blocking everyone else — as the 500-client test showed.
- **`next_line` and `recv` are cancel-safe** (DT-37), so using them in a `select!` loop loses nothing when the other branch wins.
- **The online counter is an atomic**, shared by `Arc`: the only state shared between connection tasks besides the channel.

### Review checklist

- What happens to a client that stops reading but keeps its connection open? Does it slow down others? (Look at what `write_all` does when the socket buffer is full, and where that task is blocked.)
- Add a `/nick NEWNAME` command and a `/who` command. Which state must now be shared, and should it be a `Mutex` or an actor?
- The load test checks counts. How would you check *order* — that each client's messages arrive in the order sent?
- Find the largest number of clients (at 10 messages each) for which every client still receives everything. What limits it: CPU, the buffer size, or the senders' pace?

## Answers to the Chapter 10 prediction questions

**10.5 Q1.** Nothing. The `async` block creates a future that is never awaited or spawned, so it never runs (DT-34); the compiler warns that the future is unused.

**10.5 Q2.** About 1 second on the multi-threaded runtime: each task blocks one of the two worker threads, in parallel. About 2 seconds on a `current_thread` runtime: the first task blocks the only thread for a second, then the second does. Both cases also block every other task on those threads for the duration (DT-36).

# Chapter 11 — Unsafe Rust, FFI, and the Memory Model

*Goal: know exactly what `unsafe` permits and what it still forbids; write small, well-documented unsafe code that upholds Rust's rules; build safe abstractions on top of it; and call C from Rust and Rust from C.*

Safe Rust's guarantees rest on code that the compiler cannot check: allocators, `Vec`, `Mutex`, system calls, and every C library you call. `unsafe` is how that code is written. It does not turn off the borrow checker or make anything legal that was undefined before; it lets you do five specific things whose correctness *you* must prove. The skill is to keep those proofs small, local, and written down, and to wrap them in safe interfaces that no caller can misuse.

**What you will be able to do after this chapter**

- List the five operations `unsafe` permits, and the undefined behaviour Rust still forbids.
- Use raw pointers correctly, and write `// SAFETY:` comments that justify each unsafe operation.
- Build safe abstractions over unsafe code: a hand-written vector, `split_at_mut`, `unsafe impl Send`.
- Call C functions from Rust (with strings, buffers, and callbacks) and expose Rust functions to C.
- Wrap a real C library — zlib — in a safe, idiomatic Rust API, and check it with Valgrind.

## Topic 11A — `unsafe`

## Unit 11.1 — What `unsafe` permits

**One-line summary:** Inside an `unsafe { }` block you may: (1) dereference a raw pointer; (2) call an `unsafe fn` (including foreign functions); (3) access or modify a `static mut`; (4) implement an `unsafe trait` (such as `Send` or `Sync`); (5) access fields of a `union`. That is all. Everything else — type checking, borrow checking of references, bounds checks on slices — still applies. What changes is that **you** promise the operations do not cause undefined behaviour.

The undefined behaviour that unsafe code must never cause includes: dereferencing a dangling, null, or unaligned pointer; **data races**; creating a reference that **aliases** a live `&mut` (or mutating through a `&` without `UnsafeCell`); producing an **invalid value** (a `bool` that is not 0 or 1, a `char` that is not a Unicode scalar value, an enum with an impossible discriminant, a null reference or `Box`); reading **uninitialised memory** as a typed value; and unwinding across an FFI boundary that does not allow it.

```rust
// file: unsafe_basics.rs
/// Returns the element at `index` without a bounds check.
///
/// # Safety
/// `index` must be less than `slice.len()`.
unsafe fn get_unchecked_demo(slice: &[u32], index: usize) -> u32 {
    // SAFETY: the caller guarantees index < len, so the pointer is in bounds and aligned.
    unsafe { *slice.as_ptr().add(index) }
}

static mut EVENTS: u32 = 0;

fn main() {
    let mut x = 10;
    let p = &raw mut x;                           // creating raw pointers is safe...
    let q = &raw const x;
    unsafe {
        *p += 1;                                  // ...dereferencing them is not
        println!("x = {}, read through q = {}", *p, *q);
    }

    let data = [3, 1, 4, 1, 5];
    // SAFETY: 2 < data.len() == 5.
    let third = unsafe { get_unchecked_demo(&data, 2) };
    println!("third = {third}");

    // SAFETY: single-threaded program; no other reference to EVENTS exists.
    unsafe {
        EVENTS += 1;
        println!("events = {}", *(&raw const EVENTS));
    }

    let n: u8 = 1;
    // SAFETY: 1 is a valid bool representation (0 or 1 only).
    let b: bool = unsafe { std::mem::transmute(n) };
    println!("transmuted bool = {b}");
}
```

```console
$ rustc --edition 2024 unsafe_basics.rs && ./unsafe_basics
x = 11, read through q = 11
third = 4
events = 1
transmuted bool = true
```

Since edition 2024, the body of an `unsafe fn` is not automatically an unsafe block: each unsafe operation inside it needs its own `unsafe { }`, so every one gets its own justification. The convention — enforced by Clippy's `undocumented_unsafe_blocks` lint — is a `// SAFETY:` comment on every `unsafe` block, and a `# Safety` section in the documentation of every `unsafe fn` saying what the caller must guarantee.

<!-- expanded -->
### Going deeper: Miri, the undefined-behaviour detector

`unsafe` code can compile and pass its tests while still having undefined behaviour. **Miri** is an interpreter for Rust's intermediate representation that checks every memory access against Rust's rules: out-of-bounds reads, use after free, invalid values (a `bool` that is 3), misaligned pointers, and aliasing violations (Stacked/Tree Borrows). It runs your tests a hundred or more times slower, so it is used on the `unsafe` parts.

```rust
// file: miridemo/src/lib.rs
/// Returns the first element, using a raw pointer.
pub fn first(v: &[u32]) -> Option<u32> {
    if v.is_empty() { return None; }
    Some(unsafe { *v.as_ptr() })
}

/// Bug: reads one past the end.
pub fn last_buggy(v: &[u32]) -> u32 {
    unsafe { *v.as_ptr().add(v.len()) }
}

#[cfg(test)]
mod tests {
    #[test]
    fn first_works() { assert_eq!(super::first(&[7, 8]), Some(7)); }
    #[test]
    fn last_buggy_reads_past_the_end() {
        let v = vec![1u32, 2, 3];
        let _ = super::last_buggy(&v);          // passes under plain cargo test
    }
}
```

```toml
# file: miridemo/Cargo.toml
[package]
name = "miridemo"
version = "0.1.0"
edition = "2024"
```

```console
$ cd miridemo && cargo test -q --lib 2>&1 | grep "test result"
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
$ cd miridemo && bin=$(cargo test -q --lib --no-run --message-format=json 2>/dev/null | grep -oE '"executable":"[^"]+"' | cut -d'"' -f4) && valgrind -q --error-exitcode=9 "$bin" --test-threads=1 last_buggy 2>&1 | grep -m1 -oE "Invalid read of size [0-9]+"
Invalid read of size 4
```

A plain test run reports both tests as passing; the read past the end returned whatever bytes were there. Miri is installed with `rustup +nightly component add miri` and run with `cargo +nightly miri test`; for this test it stops with an "Undefined Behavior: … out-of-bounds" error. (The machine used for this book could not download the nightly toolchain, so the second command above shows the next-best tool, Valgrind, running the same test binary: it catches this particular bug as an invalid 4-byte read. Valgrind sees only bad *memory accesses*; Miri also catches Rust-specific UB such as aliasing violations and invalid `bool` values, which Valgrind cannot.) Running Miri in CI on every crate with `unsafe` is standard practice in the Rust ecosystem.

## Unit 11.2 — Raw pointers and what can go wrong

**One-line summary:** `*const T` and `*mut T` are pointers with no lifetime and no aliasing rules attached: they may be null, dangling, or unaligned, and the compiler does not track what they point to. They are needed for FFI, for data structures with shared or cyclic links, and for implementing collections. The cost is that every bug from the C book's Chapter 12 is possible again inside `unsafe` code.

```rust
// file: dangling.rs
fn make_pointer() -> *const String {
    let s = String::from("I live on the heap");
    &raw const s                                  // pointer to a local that is about to die
}

fn main() {
    let p = make_pointer();
    // SAFETY: none — this is a use-after-free, shown on purpose.
    unsafe {
        println!("length of dead string: {}", (*p).len());
    }
}
```

```console
$ rustc --edition 2024 dangling.rs 2>&1 | grep -E '^(warning|error)'
warning: function returns a dangling pointer to dropped local variable `s`
error: implicit autoref creates a reference to the dereference of a raw pointer
error: aborting due to 1 previous error; 1 warning emitted
```

The compiler caught both problems even though they are inside and around `unsafe` code: a lint warns that the function returns a pointer to a local that is dropped on return, and `(*p).len()` is rejected outright, because calling a method that takes `&self` silently creates a `&String` from the raw pointer — a reference that must point to a live, valid value. These lints catch the obvious cases. Once a dangling pointer escapes through a data structure, only run-time tools can help:

```rust
// file: dangling2.rs
fn main() {
    let v = vec![1u64, 2, 3];
    let first = v.as_ptr();
    drop(v);                                      // the buffer is freed
    // SAFETY: none — use-after-free, shown on purpose.
    let value = unsafe { first.read() };
    println!("first element after free: {value}");
}
```

```console
$ rustc --edition 2024 -g dangling2.rs && valgrind -q ./dangling2 2>&1 | grep -E 'Invalid read|first element|at 0x.*main|free' | head -n 5
==16299== Invalid read of size 8
==16299==  Address 0x4aacce0 is 0 bytes inside a block of size 24 free'd
==16299==    at 0x484988F: free (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
first element after free: 1
```

The program compiled cleanly and printed a plausible-looking value — the classic face of undefined behaviour — and Valgrind reported the read of freed heap memory, with the stack traces of the read and of the `free`. In a project with a nightly toolchain, **Miri** — Rust's interpreter for detecting UB — would report it too, along with aliasing violations that neither Valgrind nor sanitizers can see (`cargo +nightly miri test`). Miri could not be installed in this book's build environment, so its output is not shown; use it on any crate containing `unsafe`.

### Deep & Tricky

**DT-38. Creating a `&mut` that aliases another live reference is undefined behaviour even if neither is ever used to write.** *(Guarantee: Rust language; Stacked/Tree Borrows aliasing model)* `let a = &mut *p; let b = &mut *p; *a = 1;` is UB in Rust, though the equivalent pointer code in C is fine. The compiler marks `&mut` arguments `noalias` (DT-9) and optimises on that, so code that creates overlapping `&mut`s from raw pointers can be miscompiled. Inside unsafe code, keep working with raw pointers as long as aliasing is possible, and create references only for the duration in which they are truly exclusive (or shared and immutable). Miri checks this; no other tool does.

## Unit 11.3 — Building safe abstractions

**One-line summary:** The point of `unsafe` is to implement **safe** functions and types whose public API cannot be used to cause UB, no matter what safe code the caller writes. The unsafe code's correctness depends on invariants that the type's private fields and methods maintain; privacy is what makes those invariants enforceable.

`split_at_mut` from Unit 2.7 is the canonical example: the borrow checker cannot prove that two `&mut` sub-slices do not overlap, but a few lines of reasoning can.

```rust
// file: rawvec.rs
use std::alloc::{Layout, alloc, dealloc, realloc};
use std::ptr::NonNull;

fn split_at_mut<T>(s: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = s.len();
    assert!(mid <= len, "mid out of range");              // the check that makes this sound
    let p = s.as_mut_ptr();
    // SAFETY: [0, mid) and [mid, len) are disjoint and within the original slice, which is
    // exclusively borrowed for the lifetime of both results.
    unsafe { (std::slice::from_raw_parts_mut(p, mid), std::slice::from_raw_parts_mut(p.add(mid), len - mid)) }
}

/// A minimal growable vector. Invariants: `ptr` points to an allocation of `cap` Ts
/// (dangling when cap == 0), and the first `len` of them are initialised.
pub struct MyVec<T> {
    ptr: NonNull<T>,
    cap: usize,
    len: usize,
}

// SAFETY: MyVec owns its Ts exactly like Vec does; sending it sends the Ts.
unsafe impl<T: Send> Send for MyVec<T> {}
// SAFETY: &MyVec only gives out &T.
unsafe impl<T: Sync> Sync for MyVec<T> {}

impl<T> MyVec<T> {
    pub fn new() -> Self {
        assert!(size_of::<T>() != 0, "zero-sized types not supported in this sketch");
        MyVec { ptr: NonNull::dangling(), cap: 0, len: 0 }
    }

    fn grow(&mut self) {
        let new_cap = if self.cap == 0 { 4 } else { self.cap * 2 };
        let new_layout = Layout::array::<T>(new_cap).expect("capacity overflow");
        let new_ptr = if self.cap == 0 {
            // SAFETY: new_layout has non-zero size (T is not zero-sized, new_cap > 0).
            unsafe { alloc(new_layout) }
        } else {
            let old_layout = Layout::array::<T>(self.cap).unwrap();
            // SAFETY: ptr was allocated with old_layout by this allocator; new size is non-zero.
            unsafe { realloc(self.ptr.as_ptr() as *mut u8, old_layout, new_layout.size()) }
        };
        self.ptr = NonNull::new(new_ptr as *mut T).unwrap_or_else(|| std::alloc::handle_alloc_error(new_layout));
        self.cap = new_cap;
    }

    pub fn push(&mut self, value: T) {
        if self.len == self.cap {
            self.grow();
        }
        // SAFETY: len < cap, so the slot is inside the allocation and uninitialised.
        unsafe { self.ptr.as_ptr().add(self.len).write(value) };
        self.len += 1;
    }

    pub fn pop(&mut self) -> Option<T> {
        if self.len == 0 {
            return None;
        }
        self.len -= 1;
        // SAFETY: the slot at the old len-1 was initialised; after decrementing len it is
        // logically uninitialised, so reading it out moves the value exactly once.
        Some(unsafe { self.ptr.as_ptr().add(self.len).read() })
    }

    pub fn as_slice(&self) -> &[T] {
        // SAFETY: the first len elements are initialised and the pointer is valid and aligned
        // (dangling-but-aligned is fine for len == 0).
        unsafe { std::slice::from_raw_parts(self.ptr.as_ptr(), self.len) }
    }
}

impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        while self.pop().is_some() {}                     // drop each element
        if self.cap > 0 {
            // SAFETY: allocated with exactly this layout in grow().
            unsafe { dealloc(self.ptr.as_ptr() as *mut u8, Layout::array::<T>(self.cap).unwrap()) };
        }
    }
}

fn main() {
    let mut v = [1, 2, 3, 4, 5];
    let (a, b) = split_at_mut(&mut v, 2);
    a[0] = 10;
    b[0] = 30;
    println!("{v:?}");

    let mut mv = MyVec::new();
    for i in 0..10 {
        mv.push(format!("item-{i}"));                     // heap-owning elements test Drop
    }
    println!("popped {:?}; len {} cap {}; first three {:?}", mv.pop(), mv.len, mv.cap, &mv.as_slice()[..3]);
    let moved = std::thread::spawn(move || mv.as_slice().len()).join().unwrap();   // Send in action
    println!("length seen by another thread: {moved}");
}
```

```console
$ rustc --edition 2024 -g rawvec.rs && valgrind -q --leak-check=full ./rawvec && echo "valgrind: no errors, no leaks"
[10, 2, 30, 4, 5]
popped Some("item-9"); len 9 cap 16; first three ["item-0", "item-1", "item-2"]
length seen by another thread: 9
valgrind: no errors, no leaks
```

The unsafe operations are each a line or two, each with the invariant that justifies it. Because `ptr`, `cap`, and `len` are private, no safe caller can break the invariants — the soundness argument covers every possible use. Valgrind confirms that every element's `String` and the buffer itself were freed exactly once.

<!-- expanded -->
### Going deeper: `MaybeUninit` for uninitialised memory

Creating an uninitialised value of most types is undefined behaviour — `let x: bool = mem::uninitialized();` was deprecated precisely because the compiler may assume a `bool` is 0 or 1. `MaybeUninit<T>` is the correct tool: it holds memory that may not yet be a valid `T`, and you promise, with `assume_init`, when it is.

```rust
// file: maybeuninit.rs
use std::mem::MaybeUninit;

fn squares<const N: usize>() -> [u64; N] {
    let mut arr: [MaybeUninit<u64>; N] = [const { MaybeUninit::uninit() }; N];
    for (i, slot) in arr.iter_mut().enumerate() {
        slot.write((i * i) as u64);                         // initialise every element
    }
    // SAFETY: every element was written in the loop above.
    unsafe { std::mem::transmute_copy::<_, [u64; N]>(&arr) }
}

fn main() {
    println!("{:?}", squares::<6>());
    let arr2: [String; 3] = std::array::from_fn(|i| format!("item {i}"));   // the safe way, when it fits
    println!("{arr2:?}");
}
```

```console
$ rustc --edition 2024 maybeuninit.rs && ./maybeuninit
[0, 1, 4, 9, 16, 25]
["item 0", "item 1", "item 2"]
```

`std::array::from_fn` covers most needs safely; `MaybeUninit` is for buffers filled by C functions or by algorithms whose initialisation order the compiler cannot follow. Every `unsafe` block that relies on an invariant should carry a `// SAFETY:` comment saying which one — Clippy's `undocumented_unsafe_blocks` lint can enforce it.

## Topic 11B — FFI

## Unit 11.4 — Calling C from Rust

**One-line summary:** Declare foreign functions in an `unsafe extern "C" { ... }` block (edition 2024 requires the `unsafe` keyword: the declarations are promises you make about the C side), link the library (`#[link(name = "z")]`, a build script, or `-l`), and call them in `unsafe` blocks. Pass data with C-compatible types: integers from `std::ffi` (`c_int`, `c_ulong`, …), `#[repr(C)]` structs, raw pointers, and `CString`/`CStr` for NUL-terminated strings. Callbacks are `extern "C" fn` pointers.

```rust
// file: callc.rs
use std::ffi::{CStr, CString, c_char, c_int, c_void};

unsafe extern "C" {
    fn strlen(s: *const c_char) -> usize;
    fn qsort(base: *mut c_void, n: usize, size: usize, cmp: extern "C" fn(*const c_void, *const c_void) -> c_int);
    fn getenv(name: *const c_char) -> *const c_char;
    safe fn abs(x: c_int) -> c_int;                // edition 2024: declare a foreign fn as safe to call
}

extern "C" fn compare_i32(a: *const c_void, b: *const c_void) -> c_int {
    // SAFETY: qsort passes pointers to elements of the i32 array we gave it.
    let (x, y) = unsafe { (*(a as *const i32), *(b as *const i32)) };
    x.cmp(&y) as c_int
}

fn main() {
    let s = CString::new("hello, C").unwrap();     // adds the NUL terminator; rejects interior NULs
    // SAFETY: s is a valid NUL-terminated string that outlives the call.
    println!("strlen = {}", unsafe { strlen(s.as_ptr()) });

    let mut v = [42, -7, 19, 0, 3];
    // SAFETY: base/len/size describe our array exactly, and compare_i32 matches the element type.
    unsafe { qsort(v.as_mut_ptr() as *mut c_void, v.len(), size_of::<i32>(), compare_i32) };
    println!("qsorted by C: {v:?}, abs(-5) = {}", abs(-5));

    let name = CString::new("HOME").unwrap();
    // SAFETY: getenv returns NULL or a pointer to a NUL-terminated string owned by the C library.
    let home = unsafe { getenv(name.as_ptr()) };
    if !home.is_null() {
        // SAFETY: non-null, NUL-terminated, and not modified while we read it.
        println!("HOME = {}", unsafe { CStr::from_ptr(home) }.to_string_lossy());
    }
    println!("CString::new with interior NUL: {:?}", CString::new("a\0b").map(|_| ()));
}
```

```console
$ rustc --edition 2024 callc.rs && HOME=/home/reader ./callc
strlen = 8
qsorted by C: [-7, 0, 3, 19, 42], abs(-5) = 5
HOME = /home/reader
CString::new with interior NUL: Err(NulError(1, [97, 0, 98]))
```

`strlen`, `qsort`, and `getenv` are in the C library, which every Rust program on Linux already links. The callback is an ordinary Rust function with the C calling convention. If it panicked, the panic would try to unwind into C's `qsort`; since Rust 1.81 that aborts the process with a clear message instead of being undefined behaviour — but a callback should catch panics (`std::panic::catch_unwind`) or never panic.

<!-- expanded -->
### Going deeper: `bindgen` and C strings

Writing `extern "C"` declarations by hand for a large C library is error-prone. `bindgen` reads the C headers and generates them, usually from a `build.rs` script at build time. Whatever the source of the declarations, strings cross the boundary as `CStr` (borrowed, NUL-terminated, from C) and `CString` (owned, for passing to C); a Rust `&str` is not NUL-terminated and may contain interior NULs.

```rust
// file: cstrings.rs
use std::ffi::{CStr, CString, c_char};

unsafe extern "C" {
    fn strlen(s: *const c_char) -> usize;
    fn getenv(name: *const c_char) -> *const c_char;
}

fn main() {
    let owned = CString::new("hello").unwrap();                       // adds the trailing NUL
    println!("strlen = {}", unsafe { strlen(owned.as_ptr()) });
    println!("interior NUL rejected: {}", CString::new("a\0b").is_err());

    let key = c"HOME";                                                 // a C string literal (Rust 1.77+)
    let p = unsafe { getenv(key.as_ptr()) };
    if !p.is_null() {
        let home: &CStr = unsafe { CStr::from_ptr(p) };               // borrowed from C's memory
        println!("HOME is {} bytes, valid UTF-8: {}", home.to_bytes().len(), home.to_str().is_ok());
    }
    // let dangling = CString::new("x").unwrap().as_ptr();            // pointer to a freed temporary!
}
```

```console
$ rustc --edition 2024 cstrings.rs && HOME=/home/reader ./cstrings
strlen = 5
interior NUL rejected: true
HOME is 12 bytes, valid UTF-8: true
```

The commented-out last line is a classic FFI bug: the `CString` is a temporary dropped at the end of the statement, leaving the pointer dangling. Recent compilers warn about it (`dangling_pointers_from_temporaries`).

## Unit 11.5 — Calling Rust from C

**One-line summary:** Mark a function `#[unsafe(no_mangle)] pub extern "C" fn` to export it under its plain name with the C ABI, build the crate as a `staticlib` (`.a`) or `cdylib` (`.so`), and write (or generate with `cbindgen`) a C header. Pass ownership across the boundary explicitly: if Rust allocates, Rust must free — export a matching `free` function.

```rust
// file: rustlib/src/lib.rs
use std::ffi::{CStr, CString, c_char};

/// Returns the number of words in a NUL-terminated UTF-8 string, or -1 if it is not valid UTF-8.
///
/// # Safety
/// `text` must be a valid pointer to a NUL-terminated string.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn rl_word_count(text: *const c_char) -> i64 {
    // SAFETY: guaranteed by the caller per the contract above.
    let s = unsafe { CStr::from_ptr(text) };
    match s.to_str() {
        Ok(t) => t.split_whitespace().count() as i64,
        Err(_) => -1,
    }
}

/// Returns a newly allocated upper-case copy; free it with `rl_free`.
///
/// # Safety
/// `text` must be a valid pointer to a NUL-terminated string.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn rl_upper(text: *const c_char) -> *mut c_char {
    // SAFETY: guaranteed by the caller.
    let s = unsafe { CStr::from_ptr(text) }.to_string_lossy().to_uppercase();
    CString::new(s).map(CString::into_raw).unwrap_or(std::ptr::null_mut())
}

/// Frees a string returned by `rl_upper`. Passing NULL is allowed.
///
/// # Safety
/// `p` must be NULL or a pointer returned by `rl_upper` that has not been freed.
#[unsafe(no_mangle)]
pub unsafe extern "C" fn rl_free(p: *mut c_char) {
    if !p.is_null() {
        // SAFETY: p came from CString::into_raw in rl_upper and is freed exactly once.
        drop(unsafe { CString::from_raw(p) });
    }
}
```

```toml
# file: rustlib/Cargo.toml
[package]
name = "rustlib"
version = "0.1.0"
edition = "2024"

[lib]
crate-type = ["staticlib"]

[profile.release]
panic = "abort"
```

```c
// file: rustlib/main.c
#include <stdint.h>
#include <stdio.h>

int64_t rl_word_count(const char *text);
char *rl_upper(const char *text);
void rl_free(char *p);

int main(void) {
    const char *msg = "Rust functions, called from C";
    char *up = rl_upper(msg);
    printf("%lld words; upper: %s\n", (long long)rl_word_count(msg), up);
    rl_free(up);                       /* Rust allocated it, so Rust frees it */
    printf("invalid UTF-8 -> %lld\n", (long long)rl_word_count("\xff\xfe"));
    return 0;
}
```

```console
$ cd rustlib && cargo build -q --release && cc -Wall -O2 main.c target/release/librustlib.a -o from_c && ./from_c && valgrind -q --leak-check=full ./from_c > /dev/null && echo "valgrind: clean"
5 words; upper: RUST FUNCTIONS, CALLED FROM C
invalid UTF-8 -> -1
valgrind: clean
$ cd rustlib && nm target/release/librustlib.a 2>/dev/null | grep -E ' T rl_'
0000000000000000 T rl_free
0000000000000000 T rl_upper
0000000000000000 T rl_word_count
```

The functions appear in the archive under their plain C names. `panic = "abort"` ensures that a panic can never unwind into the C caller. The allocation rule is the important design point: memory allocated by Rust's allocator must be freed by Rust (C's `free` on a `CString` pointer is undefined behaviour, even though both often use `malloc` underneath).

### Deep & Tricky

**DT-39. C macros and inline functions are not symbols, so FFI cannot call them — find the function the macro expands to.** *(Toolchain behaviour)* Many C APIs are partly macros. zlib's `deflateInit(strm, level)` is a macro for `deflateInit_(strm, level, ZLIB_VERSION, sizeof(z_stream))`, which passes the header's version string and struct size so the library can detect a mismatched header. Rust bindings must call `deflateInit_` with the same arguments. Tools like `bindgen` generate declarations from headers automatically, but they cannot translate macros that are function-like; the `-sys` crates of the ecosystem (`libz-sys`, `openssl-sys`) do this work once for everyone.

### Pattern

**P-11. Wrap every C library in two layers: a raw `-sys` layer of exact declarations, and a safe layer that owns resources with RAII and turns error codes into `Result`.** *(Practice: the `-sys` crate convention)* The raw layer is mechanical and unsafe; the safe layer contains all the reasoning: each C handle becomes a Rust type with a `Drop` impl calling the C cleanup function, borrowed buffers become slices, NUL-terminated strings become `&CStr`/`String`, and return codes become `Result<_, Error>`. Users of the safe layer never write `unsafe`. *Trade-off:* the safe layer must encode the C library's real rules (which functions are thread-safe, who owns which pointer) — read the C documentation carefully, because mistakes here are UB that safe callers cannot detect.

### Mastery check

1. **Predict.** `let s = CString::new("hi").unwrap(); let p = s.as_ptr(); drop(s); unsafe { strlen(p) }` — is this sound?
2. **Predict.** Is `unsafe { std::mem::transmute::<u8, bool>(2) }` undefined behaviour even if the result is never used?

## Chapter 11 Project — A safe Rust wrapper for zlib

**Goal:** wrap the system zlib library — written in C and installed on nearly every Unix system — in a safe Rust API: one-shot `compress`/`decompress`, a `crc32` function, and a streaming `Compressor` that owns a `z_stream` and frees it in `Drop`. Verify round-trips and the CRC against known values, compare the output with the `gzip`-compatible format, and check the whole program with Valgrind.

```rust
// file: zwrap/src/sys.rs
//! Raw declarations matching zlib.h (zlib 1.2/1.3, 64-bit Linux). Every item here is unsafe to use.
#![allow(non_camel_case_types)]
use std::ffi::{c_char, c_int, c_uint, c_ulong, c_void};

#[repr(C)]
pub struct z_stream {
    pub next_in: *const u8,
    pub avail_in: c_uint,
    pub total_in: c_ulong,
    pub next_out: *mut u8,
    pub avail_out: c_uint,
    pub total_out: c_ulong,
    pub msg: *const c_char,
    pub state: *mut c_void,
    pub zalloc: Option<unsafe extern "C" fn(*mut c_void, c_uint, c_uint) -> *mut c_void>,
    pub zfree: Option<unsafe extern "C" fn(*mut c_void, *mut c_void)>,
    pub opaque: *mut c_void,
    pub data_type: c_int,
    pub adler: c_ulong,
    pub reserved: c_ulong,
}

pub const Z_OK: c_int = 0;
pub const Z_STREAM_END: c_int = 1;
pub const Z_BUF_ERROR: c_int = -5;
pub const Z_NO_FLUSH: c_int = 0;
pub const Z_FINISH: c_int = 4;

#[link(name = "z")]
unsafe extern "C" {
    pub fn zlibVersion() -> *const c_char;
    pub fn compressBound(source_len: c_ulong) -> c_ulong;
    pub fn compress2(dest: *mut u8, dest_len: *mut c_ulong, source: *const u8, source_len: c_ulong, level: c_int) -> c_int;
    pub fn uncompress(dest: *mut u8, dest_len: *mut c_ulong, source: *const u8, source_len: c_ulong) -> c_int;
    pub fn crc32(crc: c_ulong, buf: *const u8, len: c_uint) -> c_ulong;
    pub fn deflateInit2_(strm: *mut z_stream, level: c_int, method: c_int, window_bits: c_int, mem_level: c_int,
                         strategy: c_int, version: *const c_char, stream_size: c_int) -> c_int;
    pub fn deflate(strm: *mut z_stream, flush: c_int) -> c_int;
    pub fn deflateEnd(strm: *mut z_stream) -> c_int;
}
```

```rust
// file: zwrap/src/lib.rs
//! A safe wrapper over the system zlib. No `unsafe` is needed to use it.
mod sys;

use std::ffi::{CStr, c_int, c_uint, c_ulong};
use std::mem::MaybeUninit;

/// An error code returned by zlib.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct ZError(pub i32);

impl std::fmt::Display for ZError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        let what = match self.0 { -2 => "stream error", -3 => "data error", -4 => "out of memory", -5 => "buffer error", _ => "zlib error" };
        write!(f, "{what} ({})", self.0)
    }
}
impl std::error::Error for ZError {}

/// The version string of the zlib library actually loaded.
pub fn version() -> String {
    // SAFETY: zlibVersion returns a pointer to a static NUL-terminated string.
    unsafe { CStr::from_ptr(sys::zlibVersion()) }.to_string_lossy().into_owned()
}

/// CRC-32 (as used by gzip, PNG, and ZIP) of `data`.
pub fn crc32(data: &[u8]) -> u32 {
    let mut crc: c_ulong = 0;
    for chunk in data.chunks(c_uint::MAX as usize) {       // zlib takes a 32-bit length
        // SAFETY: chunk is a valid readable buffer of chunk.len() bytes.
        crc = unsafe { sys::crc32(crc, chunk.as_ptr(), chunk.len() as c_uint) };
    }
    crc as u32
}

/// Compresses `data` into the zlib format at `level` (0-9).
pub fn compress(data: &[u8], level: i32) -> Result<Vec<u8>, ZError> {
    // SAFETY: compressBound is a pure function of its argument.
    let bound = unsafe { sys::compressBound(data.len() as c_ulong) } as usize;
    let mut out = Vec::with_capacity(bound);
    let mut out_len = bound as c_ulong;
    // SAFETY: out has capacity `bound` writable bytes and out_len says so; data is readable.
    let rc = unsafe { sys::compress2(out.as_mut_ptr(), &mut out_len, data.as_ptr(), data.len() as c_ulong, level) };
    if rc != sys::Z_OK {
        return Err(ZError(rc));
    }
    // SAFETY: zlib wrote exactly out_len (<= bound) initialised bytes.
    unsafe { out.set_len(out_len as usize) };
    Ok(out)
}

/// Decompresses zlib data whose decompressed size is at most `max_len`.
pub fn decompress(data: &[u8], max_len: usize) -> Result<Vec<u8>, ZError> {
    let mut out = Vec::with_capacity(max_len);
    let mut out_len = max_len as c_ulong;
    // SAFETY: as in compress: out has max_len writable bytes.
    let rc = unsafe { sys::uncompress(out.as_mut_ptr(), &mut out_len, data.as_ptr(), data.len() as c_ulong) };
    if rc != sys::Z_OK {
        return Err(ZError(rc));
    }
    // SAFETY: zlib reports how many bytes it wrote.
    unsafe { out.set_len(out_len as usize) };
    Ok(out)
}

/// A streaming compressor producing gzip format. Owns a z_stream; freed on drop.
pub struct GzipEncoder {
    strm: Box<sys::z_stream>,          // boxed: zlib keeps a pointer back to the stream, so it must not move
    out: Vec<u8>,
}

impl GzipEncoder {
    pub fn new(level: i32) -> Result<Self, ZError> {
        // SAFETY: an all-zero z_stream is the documented initial state (NULL zalloc/zfree/opaque = defaults).
        let mut strm: Box<sys::z_stream> = Box::new(unsafe { MaybeUninit::zeroed().assume_init() });
        // windowBits 15 + 16 selects the gzip wrapper; memLevel 8 and strategy 0 are zlib's defaults.
        // SAFETY: strm is valid and zeroed; version and size match the header we were written against (DT-39).
        let rc = unsafe {
            sys::deflateInit2_(&mut *strm, level, 8, 15 + 16, 8, 0, sys::zlibVersion(), size_of::<sys::z_stream>() as c_int)
        };
        if rc != sys::Z_OK {
            return Err(ZError(rc));                        // nothing allocated: no deflateEnd needed
        }
        Ok(GzipEncoder { strm, out: Vec::new() })
    }

    fn run(&mut self, input: &[u8], flush: c_int) -> Result<(), ZError> {
        self.strm.next_in = input.as_ptr();
        self.strm.avail_in = input.len() as c_uint;
        loop {
            self.out.reserve(64 * 1024);
            let spare = self.out.capacity() - self.out.len();
            // SAFETY: the pointer is just past the initialised part of `out`, with `spare` bytes of capacity.
            self.strm.next_out = unsafe { self.out.as_mut_ptr().add(self.out.len()) };
            self.strm.avail_out = spare as c_uint;
            // SAFETY: strm was initialised by deflateInit2_ and its in/out pointers are valid right now.
            let rc = unsafe { sys::deflate(&mut *self.strm, flush) };
            let written = spare - self.strm.avail_out as usize;
            // SAFETY: deflate initialised `written` bytes after the old length.
            unsafe { self.out.set_len(self.out.len() + written) };
            match rc {
                sys::Z_STREAM_END => return Ok(()),
                sys::Z_OK | sys::Z_BUF_ERROR if self.strm.avail_in == 0 && flush == sys::Z_NO_FLUSH => return Ok(()),
                sys::Z_OK | sys::Z_BUF_ERROR => continue,           // more output to produce
                e => return Err(ZError(e)),
            }
        }
    }

    /// Feeds more input.
    pub fn write(&mut self, input: &[u8]) -> Result<(), ZError> {
        self.run(input, sys::Z_NO_FLUSH)
    }

    /// Finishes the stream and returns the gzip bytes.
    pub fn finish(mut self) -> Result<Vec<u8>, ZError> {
        self.run(&[], sys::Z_FINISH)?;
        Ok(std::mem::take(&mut self.out))                  // Drop still runs and frees the stream
    }
}

impl Drop for GzipEncoder {
    fn drop(&mut self) {
        // SAFETY: strm was successfully initialised in new() and is ended exactly once, here.
        unsafe { sys::deflateEnd(&mut *self.strm) };
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn crc32_known_value() {
        assert_eq!(crc32(b"123456789"), 0xCBF4_3926);       // the standard CRC-32 check value
    }

    #[test]
    fn round_trip_and_errors() {
        let data = b"hello hello hello hello zlib".repeat(100);
        let z = compress(&data, 9).unwrap();
        assert!(z.len() < data.len() / 10);
        assert_eq!(decompress(&z, data.len()).unwrap(), data);
        assert_eq!(decompress(&z, 10), Err(ZError(-5)));         // output buffer too small
        assert_eq!(decompress(b"not zlib", 100), Err(ZError(-3)));  // corrupt input
    }
}
```

```toml
# file: zwrap/Cargo.toml
[package]
name = "zwrap"
version = "0.1.0"
edition = "2024"
```

```rust
// file: zwrap/src/main.rs
use std::io::Write;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("zlib version {}", zwrap::version());
    let text: Vec<u8> = (0..20_000).flat_map(|i| format!("line {i}: the quick brown fox\n").into_bytes()).collect();
    for level in [1, 6, 9] {
        let z = zwrap::compress(&text, level)?;
        println!("level {level}: {} -> {} bytes ({:.1}%)", text.len(), z.len(), 100.0 * z.len() as f64 / text.len() as f64);
    }
    let z = zwrap::compress(&text, 6)?;
    let back = zwrap::decompress(&z, text.len())?;
    println!("round trip identical: {}, crc32 {:08x} == {:08x}", back == text, zwrap::crc32(&text), zwrap::crc32(&back));

    let mut enc = zwrap::GzipEncoder::new(6)?;
    for chunk in text.chunks(1000) {                       // stream it in pieces
        enc.write(chunk)?;
    }
    let gz = enc.finish()?;
    std::fs::File::create("out.txt.gz")?.write_all(&gz)?;
    std::fs::write("out.txt", &text)?;
    println!("wrote out.txt.gz ({} bytes)", gz.len());
    let dropped_early = zwrap::GzipEncoder::new(9)?;      // never finished: Drop must still free it
    drop(dropped_early);
    Ok(())
}
```

```console
$ cd zwrap && cargo test -q 2>&1 | grep 'test result' | head -n 1
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
$ cd zwrap && cargo build -q && valgrind -q --leak-check=full --errors-for-leak-kinds=all ./target/debug/zwrap && echo "valgrind: no errors, no leaks"
zlib version 1.3.1.zlib-ng
level 1: 628890 -> 53541 bytes (8.5%)
level 6: 628890 -> 53097 bytes (8.4%)
level 9: 628890 -> 52784 bytes (8.4%)
round trip identical: true, crc32 6194fa02 == 6194fa02
wrote out.txt.gz (53109 bytes)
valgrind: no errors, no leaks
$ cd zwrap && gzip -t out.txt.gz && gunzip -c out.txt.gz | cmp - out.txt && echo "gzip accepts our stream; contents identical" && gzip -l out.txt.gz | tail -n 1
gzip accepts our stream; contents identical
              53109              628890  91.6% out.txt
```

The wrapper follows P-11. `sys.rs` is the raw layer, a transcription of `zlib.h`. `lib.rs` contains every `unsafe` block — thirteen of them, each with its justification — and exposes an API in which nothing is unsafe: slices in, `Vec<u8>` out, `Result` for errors, and a `GzipEncoder` whose `Drop` guarantees `deflateEnd`, even for an encoder that is never finished. The library loaded here reports itself as `1.3.1.zlib-ng`: Ubuntu 24.04 ships **zlib-ng** in zlib-compatible mode, a faster reimplementation with the same API and `z_stream` layout — the declarations work unchanged, which is the point of a stable C ABI. The standard `gzip` tool accepts the stream, and Valgrind reports no invalid accesses and no leaks — including from zlib's own allocations, which `Drop` released.

Two details carry the soundness argument:

- **`Box<z_stream>`.** zlib stores a pointer back to the `z_stream` inside its private state, so the struct must never move after `deflateInit2_`. A `GzipEncoder` itself can be moved freely, because only the `Box` (a pointer) moves; the stream stays put on the heap.
- **`Vec::set_len` only after the C code reports what it wrote.** The vector's spare capacity is uninitialised; the length is advanced by exactly the byte count zlib returns, so the vector never claims to contain bytes no one wrote.

### Review checklist

- `decompress` needs `max_len`. How would you write a version that grows its buffer when it gets `Z_BUF_ERROR`? (Use `inflateInit2_` and a loop like `GzipEncoder::run`.)
- What would go wrong if `strm` were a plain `z_stream` field instead of `Box<z_stream>`, and the encoder were moved after `new`?
- Is `GzipEncoder` `Send`? Should it be? (zlib streams may be used from any thread, but not concurrently.) What does the compiler say, and why? (Raw pointers in `z_stream`, DT-31.)

## Answers to the Chapter 11 prediction questions

**11.5 Q1.** No. `as_ptr` borrows the `CString`'s buffer without keeping it alive; after `drop(s)` the pointer dangles, and `strlen` reads freed memory. This exact pattern — `CString::new(x).unwrap().as_ptr()` in one expression, which drops the temporary at the end of the statement — is so common that rustc has a dedicated lint, `dangling_pointers_from_temporaries`, that warns about it. Keep the `CString` in a variable for as long as C uses the pointer.

**11.5 Q2.** Yes. Producing an invalid value — a `bool` that is neither 0 nor 1 — is undefined behaviour at the moment it is produced, whether or not it is used. The compiler is entitled to assume every `bool` is 0 or 1 (and uses the other 254 values as niches, DT-14).

# Chapter 12 — Macros: Declarative and Procedural

*Goal: write macros that remove real repetition — declarative macros with `macro_rules!` and procedural macros (derive, attribute, function-like) with `syn` and `quote` — and know where macros beat functions and generics, and where they do not.*

Macros in Rust operate on **tokens**, not text, and run inside the compiler before type checking. They are hygienic (they cannot accidentally capture the caller's variables) and their invocations are always marked — `name!(...)`, `#[derive(Name)]`, `#[name]` — so a reader can see where code is generated. `println!`, `vec!`, `format!`, `assert_eq!`, `#[derive(Debug)]`, `#[tokio::main]`, and `#[test]` are all macros.

**What you will be able to do after this chapter**

- Write `macro_rules!` macros with fragment specifiers, repetitions, and multiple rules.
- Write recursive macros, and export macros from a crate correctly.
- Explain macro hygiene, and avoid evaluating arguments more than once.
- Write a derive macro, an attribute macro, and a function-like procedural macro with `syn` and `quote`, with good error messages.

## Topic 12A — Declarative macros

## Unit 12.1 — `macro_rules!`

**One-line summary:** A declarative macro is a list of **rules** `(matcher) => { expansion }`. The matcher captures input fragments with **fragment specifiers** — `$e:expr`, `$i:ident`, `$t:ty`, `$p:pat`, `$l:literal`, `$b:block`, `$tt:tt` (any single token tree) — and **repetitions** `$( ... ),*` (zero or more, comma-separated) or `$( ... );+` (one or more). The first rule that matches is expanded.

```rust
// file: decl.rs
use std::collections::HashMap;

/// hashmap!{ "a" => 1, "b" => 2 } — like vec! for maps
macro_rules! hashmap {
    () => { HashMap::new() };
    ( $( $k:expr => $v:expr ),+ $(,)? ) => {{                // optional trailing comma
        let mut m = HashMap::with_capacity([$( stringify!($k) ),+].len());
        $( m.insert($k, $v); )+
        m
    }};
}

/// Generates a newtype with Display and From, for each name given.
macro_rules! newtypes {
    ( $( $name:ident($inner:ty) ),* ) => {
        $(
            #[derive(Debug, Clone, Copy, PartialEq)]
            struct $name($inner);
            impl std::fmt::Display for $name {
                fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
                    write!(f, "{}({})", stringify!($name), self.0)
                }
            }
            impl From<$inner> for $name {
                fn from(v: $inner) -> Self { $name(v) }
            }
        )*
    };
}

newtypes!(UserId(u64), Celsius(f64));

/// A tiny assertion macro that reports the expression text.
macro_rules! check {
    ($cond:expr) => {
        if !$cond {
            println!("check failed: {}  (at {}:{})", stringify!($cond), file!(), line!());
        }
    };
    ($cond:expr, $($fmt:tt)+) => {
        if !$cond {
            println!("check failed: {}: {}", stringify!($cond), format!($($fmt)+));
        }
    };
}

fn main() {
    let ages: HashMap<&str, u32> = hashmap! { "ada" => 36, "alan" => 41, };
    let empty: HashMap<u8, u8> = hashmap!();
    let mut names: Vec<_> = ages.keys().collect();
    names.sort();
    println!("{names:?}, empty has {}", empty.len());
    let id: UserId = 42.into();
    println!("{id} {}", Celsius(21.5));
    check!(ages["ada"] > 40);
    check!(ages.len() == 3, "expected 3 people, got {}", ages.len());
}
```

```console
$ rustc --edition 2024 decl.rs && ./decl
["ada", "alan"], empty has 0
UserId(42) Celsius(21.5)
check failed: ages["ada"] > 40  (at decl.rs:56)
check failed: ages.len() == 3: expected 3 people, got 2
```

The double braces `{{ ... }}` in `hashmap!` make the expansion a block *expression*, so its local `m` is scoped and the block's value is the map. `stringify!` turns tokens into a string literal at compile time, which is how `check!` (like `assert!`) can print the source text of the condition.

<!-- expanded -->
### Going deeper: fragment specifiers, and a macro that writes code you would otherwise repeat

A `macro_rules!` pattern matches *token trees* with **fragment specifiers**: `$x:expr` (an expression), `$t:ty` (a type), `$i:ident` (an identifier), `$l:literal`, `$p:pat`, `$b:block`, and `$tt:tt` (any single token tree). Repetition is `$(...),*` (zero or more, comma-separated) or `$(...);+` (one or more). A typical use is generating repetitive items that functions cannot produce — trait impls for many types, or struct definitions.

```rust
// file: macrogen.rs
trait Describe { fn describe() -> String; }

macro_rules! impl_describe {
    ($($t:ty => $desc:literal),* $(,)?) => {
        $( impl Describe for $t { fn describe() -> String { format!("{} ({} bytes)", $desc, std::mem::size_of::<$t>()) } } )*
    };
}
impl_describe!(u8 => "byte", u32 => "word", f64 => "double", (u8, u32) => "pair",);

macro_rules! newtype {
    ($name:ident($inner:ty)) => {
        #[derive(Debug, Clone, Copy, PartialEq)]
        struct $name($inner);
        impl From<$inner> for $name { fn from(v: $inner) -> Self { $name(v) } }
    };
}
newtype!(Celsius(f64));
newtype!(Port(u16));

fn main() {
    println!("{}\n{}\n{}\n{}", u8::describe(), u32::describe(), f64::describe(), <(u8, u32)>::describe());
    let c: Celsius = 21.5.into();
    let p = Port::from(8080);
    println!("{c:?} {p:?}");
}
```

```console
$ rustc --edition 2024 macrogen.rs && ./macrogen
byte (1 bytes)
word (4 bytes)
double (8 bytes)
pair (8 bytes)
Celsius(21.5) Port(8080)
```

The `$(,)?` at the end of the pattern accepts an optional trailing comma — a small courtesy that makes the macro behave like a normal list. To see what a macro expands to, use `cargo expand` (from the `cargo-expand` tool) or `rustc -Zunpretty=expanded` on nightly.

## Unit 12.2 — Recursive macros and exporting

**One-line summary:** A macro may invoke itself, consuming its input a piece at a time (a "tt muncher"), which allows counting, parsing small mini-languages, and generating code from structured input. Macros are textually scoped within a crate; `#[macro_export]` makes one available to other crates at the crate root, and `$crate::path` in the expansion refers to items in the defining crate no matter where the macro is used.

```rust
// file: recursive.rs
/// Counts its arguments at compile time.
macro_rules! count {
    () => { 0usize };
    ($head:tt $($tail:tt)*) => { 1usize + count!($($tail)*) };
}

/// Builds a nested max: max!(a, b, c) => a.max(b.max(c))
macro_rules! max {
    ($x:expr) => { $x };
    ($x:expr, $($rest:expr),+) => { std::cmp::max($x, max!($($rest),+)) };
}

/// A mini-language: `bits! { flag_a = 0, flag_b = 3 }` makes constants and a describe() fn.
macro_rules! bits {
    ( $( $name:ident = $bit:literal ),* $(,)? ) => {
        $( #[allow(non_upper_case_globals)] const $name: u32 = 1 << $bit; )*
        fn describe(v: u32) -> Vec<&'static str> {
            let mut out = Vec::new();
            $( if v & $name != 0 { out.push(stringify!($name)); } )*
            out
        }
    };
}

bits! { readable = 0, writable = 1, executable = 2, hidden = 7 }

fn main() {
    const N: usize = count!(a b c d e);                 // evaluated at compile time
    println!("count = {N}, max = {}", max!(3, 9, 4, 7));
    let perms = readable | executable | hidden;
    println!("{perms:#010b} = {:?}", describe(perms));
}
```

```console
$ rustc --edition 2024 recursive.rs && ./recursive
count = 5, max = 9
0b10000101 = ["readable", "executable", "hidden"]
```

### Deep & Tricky

**DT-40. Macro hygiene: identifiers created inside a macro cannot see or shadow the caller's — but items (functions, types, statics) are not hygienic.** *(Guarantee: Rust language, `macro_rules!` hygiene)* A `let x` inside an expansion is a different `x` from the caller's `x`, even with the same spelling, so a macro cannot accidentally capture or clobber local variables (the classic C preprocessor bug, C book Unit 4.3). To *use* a caller's variable, the caller must pass its name in as an `$i:ident`. Item names, however, are resolved normally: `bits!` above defines `describe` and the constants in the caller's scope, which is exactly what it intends — and a name clash with an existing item is an ordinary compile error.

```rust
// file: hygiene.rs
macro_rules! double_it {
    ($e:expr) => {{
        let x = $e;                                     // this x is the macro's own...
        x * 2
    }};
}

macro_rules! square_twice_bad {
    ($e:expr) => { $e * $e };                           // evaluates $e twice!
}

macro_rules! square_once {
    ($e:expr) => {{ let v = $e; v * v }};               // evaluates $e once
}

fn next(counter: &mut i32) -> i32 {
    *counter += 1;
    *counter
}

fn main() {
    let x = 10;
    println!("double_it!(x + 1) = {}, caller's x still {x}", double_it!(x + 1));   // ...not the caller's x
    let mut c = 0;
    let bad = square_twice_bad!(next(&mut c));          // next() called twice: 1 * 2
    let mut d = 0;
    let good = square_once!(next(&mut d));              // next() called once: 1 * 1
    println!("square_twice_bad = {bad} (counter {c}), square_once = {good} (counter {d})");
}
```

```console
$ rustc --edition 2024 hygiene.rs && ./hygiene
double_it!(x + 1) = 22, caller's x still 10
square_twice_bad = 2 (counter 2), square_once = 1 (counter 1)
```

Unlike C macros, `$e:expr` is a complete expression, so `square_twice_bad!(1 + 2)` correctly gives 9, not `1 + 2 * 1 + 2`. But the expression is still *pasted* wherever `$e` appears, so side effects run once per occurrence — bind macro arguments to a local first.

## Topic 12B — Procedural macros

## Unit 12.3 — How procedural macros work

**One-line summary:** A **procedural macro** is a Rust function, compiled into a special `proc-macro` crate, that the compiler calls with a `TokenStream` and that returns a `TokenStream`. There are three kinds: **derive** macros (`#[derive(Name)]` on a struct or enum; they *add* items), **attribute** macros (`#[name(args)]` on an item; they *replace* it), and **function-like** macros (`name!(...)`). The `syn` crate parses tokens into a Rust syntax tree, and `quote!` turns Rust-like templates back into tokens.

The chapter project builds a derive macro step by step. Here is a smaller attribute macro first, to show the shape: `#[timed]` wraps a function so that it reports how long each call took.

```toml
# file: pm/timed/Cargo.toml
[package]
name = "timed"
version = "0.1.0"
edition = "2024"

[lib]
proc-macro = true

[dependencies]
syn = { version = "2", features = ["full"] }
quote = "1"
proc-macro2 = "1"
```

```rust
// file: pm/timed/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{ItemFn, parse_macro_input};

/// Prints the wall-clock time of every call to the annotated function.
#[proc_macro_attribute]
pub fn timed(_args: TokenStream, item: TokenStream) -> TokenStream {
    let ItemFn { attrs, vis, sig, block } = parse_macro_input!(item as ItemFn);
    let name = sig.ident.to_string();
    quote! {
        #(#attrs)* #vis #sig {
            let __start = ::std::time::Instant::now();
            let __result = (move || #block)();
            eprintln!("[timed] {} took {:.1?}", #name, __start.elapsed());
            __result
        }
    }
    .into()
}
```

```toml
# file: pm/app/Cargo.toml
[package]
name = "app"
version = "0.1.0"
edition = "2024"

[dependencies]
timed = { path = "../timed" }
```

```rust
// file: pm/app/src/main.rs
use timed::timed;

#[timed]
fn slow_sum(n: u64) -> u64 {
    (0..n).map(|i| i % 7).sum()
}

#[timed]
fn greet(name: &str) -> String {
    format!("hello, {name}")
}

fn main() {
    println!("{}", slow_sum(std::hint::black_box(30_000_000)));
    println!("{}", greet("macros"));
}
```

```console
$ cd pm/app && cargo run -q --release 2>&1
[timed] slow_sum took 25.5ms
89999995
[timed] greet took 796.0ns
hello, macros
```

The macro received the whole function as tokens, parsed it with `syn` into an `ItemFn`, and emitted a new function with the same attributes, visibility, and signature whose body times the original body. `#name` interpolates a Rust value into the output tokens; `#(#attrs)*` interpolates a repetition. Paths in generated code are written fully qualified (`::std::time::Instant`) so they work wherever the macro is used, whatever the user has imported.

### Deep & Tricky

**DT-41. Procedural macros are compiled for and run on the build machine, inside the compiler — and they can run arbitrary code.** *(Toolchain behaviour)* A proc-macro crate is a compiler plugin: it is built for the host, loaded by `rustc`, and executed during compilation of every crate that uses it. It can read files, access the network, or take a long time; builds of crates using heavy macros (`serde`, `syn` itself) spend a noticeable fraction of their time compiling and running them. This is also a supply-chain consideration: depending on a proc-macro crate means running its code on your machine at build time, just as `build.rs` scripts do.

## Unit 12.4 — Errors in macros

**One-line summary:** A macro that receives input it cannot handle should not `panic!` (which produces an unhelpful "proc macro panicked" message) but return a compile error pointing at the offending tokens: `syn::Error::new_spanned(tokens, "message").to_compile_error()`. Declarative macros use `compile_error!("...")` in a dedicated rule.

### Pattern

**P-12. Use a function first, generics second, a declarative macro third, and a procedural macro last.** *(Practice)* Functions and generics are type-checked, debuggable, and understood by every tool. Reach for `macro_rules!` when you need something functions cannot do: variable numbers of arguments, generating items (structs, impls, tests) from a list, or capturing source text. Reach for a procedural macro when the generated code depends on the *structure* of a type — its fields and their attributes — as `serde`'s `Serialize` does. *Trade-off:* macros remove boilerplate at the cost of compile time, harder error messages for users, and code that IDEs and readers see only after expansion.

<!-- expanded -->
### Going deeper: `compile_error!` and helpful macro errors

A declarative macro can reject bad input with its own message by matching it in an arm that expands to `compile_error!`. Since the error is reported at the macro call, users see a sentence written for them instead of an error deep inside generated code.

```rust
// file: macroerr.rs
macro_rules! percent {
    ($v:literal %) => { ($v as f64) / 100.0 };
    ($v:literal) => { compile_error!("percent! needs a % sign, like percent!(15 %)") };
}

fn main() {
    let tax = percent!(18 %);
    println!("18% of 250 = {}", 250.0 * tax);
    #[cfg(feature_missing_sign)]
    let _ = percent!(18);
}
```

```console
$ rustc --edition 2024 macroerr.rs && ./macroerr
18% of 250 = 45
$ rustc --edition 2024 --cfg feature_missing_sign macroerr.rs -o macroerr2 2>&1 | grep -m1 -oE "error: .*"
error: percent! needs a % sign, like percent!(15 %)
```

Procedural macros do the same with `syn::Error::new_spanned(tokens, "message").to_compile_error()`, which also points the error at the exact tokens that caused it.

### Mastery check

1. **Predict.** What does `count!(a, b, c)` evaluate to with the `count!` macro above (note the commas)?
2. **Predict.** A `macro_rules!` macro's expansion contains `let result = 5;`, and the caller's code after the invocation uses `result`. Does it compile?

## Chapter 12 Project — `#[derive(Builder)]`

**Goal:** a derive macro that generates the **builder pattern** for any struct with named fields: a `FooBuilder` with one setter per field, optional fields (`Option<T>`) that may be omitted, `#[builder(default = expr)]` for defaults, and a `build()` that returns `Result<Foo, String>` naming the first missing required field. Invalid uses — on an enum, on a tuple struct, with an unknown attribute — produce compile errors pointing at the right place.

```toml
# file: builder/builder_derive/Cargo.toml
[package]
name = "builder_derive"
version = "0.1.0"
edition = "2024"

[lib]
proc-macro = true

[dependencies]
syn = { version = "2", features = ["full"] }
quote = "1"
proc-macro2 = "1"
```

```rust
// file: builder/builder_derive/src/lib.rs
use proc_macro::TokenStream;
use proc_macro2::TokenStream as TokenStream2;
use quote::{format_ident, quote};
use syn::{Data, DeriveInput, Expr, Fields, GenericArgument, PathArguments, Type, parse_macro_input};

#[proc_macro_derive(Builder, attributes(builder))]
pub fn derive_builder(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    match expand(&input) {
        Ok(tokens) => tokens.into(),
        Err(e) => e.to_compile_error().into(),          // a real compile error, at the right span
    }
}

/// If `ty` is `Option<T>`, returns `T`.
fn option_inner(ty: &Type) -> Option<&Type> {
    let Type::Path(p) = ty else { return None };
    let seg = p.path.segments.last()?;
    if seg.ident != "Option" {
        return None;
    }
    let PathArguments::AngleBracketed(args) = &seg.arguments else { return None };
    match args.args.first()? {
        GenericArgument::Type(t) => Some(t),
        _ => None,
    }
}

/// Reads `#[builder(default = <expr>)]`, if present.
fn default_expr(field: &syn::Field) -> syn::Result<Option<Expr>> {
    let mut found = None;
    for attr in field.attrs.iter().filter(|a| a.path().is_ident("builder")) {
        attr.parse_nested_meta(|meta| {
            if meta.path.is_ident("default") {
                found = Some(meta.value()?.parse::<Expr>()?);
                Ok(())
            } else {
                Err(meta.error("unknown builder attribute; expected `default = ...`"))
            }
        })?;
    }
    Ok(found)
}

fn expand(input: &DeriveInput) -> syn::Result<TokenStream2> {
    let name = &input.ident;
    let builder = format_ident!("{name}Builder");
    let Data::Struct(data) = &input.data else {
        return Err(syn::Error::new_spanned(name, "Builder can only be derived for structs"));
    };
    let Fields::Named(fields) = &data.fields else {
        return Err(syn::Error::new_spanned(&data.fields, "Builder needs a struct with named fields"));
    };

    let mut storage = Vec::new();        // fields of the builder struct
    let mut setters = Vec::new();        // one method per field
    let mut build_fields = Vec::new();   // how build() produces each field
    for f in &fields.named {
        let ident = f.ident.as_ref().unwrap();
        let ty = &f.ty;
        let default = default_expr(f)?;
        if let Some(inner) = option_inner(ty) {
            storage.push(quote! { #ident: #ty });
            setters.push(quote! {
                pub fn #ident(mut self, value: impl Into<#inner>) -> Self { self.#ident = Some(value.into()); self }
            });
            build_fields.push(quote! { #ident: self.#ident });
        } else {
            storage.push(quote! { #ident: ::core::option::Option<#ty> });
            setters.push(quote! {
                pub fn #ident(mut self, value: impl Into<#ty>) -> Self { self.#ident = Some(value.into()); self }
            });
            let missing = format!("missing required field `{ident}`");
            build_fields.push(match default {
                Some(expr) => quote! { #ident: self.#ident.unwrap_or_else(|| #expr) },
                None => quote! { #ident: self.#ident.ok_or_else(|| #missing.to_string())? },
            });
        }
    }
    let vis = &input.vis;
    Ok(quote! {
        #[derive(Default)]
        #vis struct #builder { #(#storage,)* }

        impl #builder {
            #(#setters)*
            pub fn build(self) -> ::core::result::Result<#name, ::std::string::String> {
                Ok(#name { #(#build_fields,)* })
            }
        }

        impl #name {
            pub fn builder() -> #builder { #builder::default() }
        }
    })
}
```

```toml
# file: builder/demo/Cargo.toml
[package]
name = "demo"
version = "0.1.0"
edition = "2024"

[dependencies]
builder_derive = { path = "../builder_derive" }
```

```rust
// file: builder/demo/src/main.rs
use builder_derive::Builder;

#[derive(Builder, Debug)]
pub struct Request {
    url: String,
    method: String,
    #[builder(default = 30)]
    timeout_secs: u64,
    #[builder(default = vec![("user-agent".to_string(), "demo/1.0".to_string())])]
    headers: Vec<(String, String)>,
    body: Option<String>,
}

fn main() {
    let get = Request::builder().url("https://example.org/").method("GET").build();
    println!("{get:#?}");
    let post = Request::builder().url("https://example.org/api").method("POST").timeout_secs(5u64).body("{\"x\":1}").build().unwrap();
    println!("{} {}: timeout {}s, body {:?}, headers {}", post.method, post.url, post.timeout_secs, post.body, post.headers.len());
    println!("{:?}", Request::builder().method("GET").build());
}
```

```console
$ cd builder/demo && cargo run -q
Ok(
    Request {
        url: "https://example.org/",
        method: "GET",
        timeout_secs: 30,
        headers: [
            (
                "user-agent",
                "demo/1.0",
            ),
        ],
        body: None,
    },
)
POST https://example.org/api: timeout 5s, body Some("{\"x\":1}"), headers 1
Err("missing required field `url`")
```

The generated code is ordinary Rust, type-checked like any other: setters take `impl Into<T>` (so `.url("...")` accepts a `&str` for a `String` field), `Option` fields are optional and their setters take the inner type, defaults are arbitrary expressions evaluated only when the field was not set, and `build` reports the first missing required field.

Now the misuse. Each mistake becomes a compile error at the exact tokens responsible:

```rust
// file: builder/demo/examples/misuse.rs
use builder_derive::Builder;

#[derive(Builder)]
enum NotAStruct { A, B }

#[derive(Builder)]
struct Tuple(u32, String);

#[derive(Builder)]
struct Typo {
    #[builder(defualt = 3)]
    n: u32,
}

fn main() {}
```

```console
$ cd builder/demo && cargo build -q --example misuse 2>&1 | grep -E '^error|^ +--> |^ *[0-9]+ \|.*(enum|struct Tuple|defualt)' 
error: Builder can only be derived for structs
 --> examples/misuse.rs:5:6
5 | enum NotAStruct { A, B }
error: Builder needs a struct with named fields
 --> examples/misuse.rs:8:13
8 | struct Tuple(u32, String);
error: unknown builder attribute; expected `default = ...`
  --> examples/misuse.rs:12:15
12 |     #[builder(defualt = 3)]
error: could not compile `demo` (example "misuse") due to 3 previous errors
```

All three errors point at the source the user wrote — the enum's name, the tuple fields, the misspelt attribute — with messages written for the user, not for the macro's author. That is the difference between a macro that is pleasant to use and one that produces "proc macro panicked" and a wall of generated code.

### Review checklist

- How does the macro decide that `body` is optional? What happens with a field of type `std::option::Option<String>`, or a type alias `type Maybe<T> = Option<T>;`? (Macros see tokens, not types — they cannot resolve aliases.)
- Why does the generated code write `::core::option::Option` and `::std::string::String` instead of `Option` and `String`?
- Extend the macro with `#[builder(each = "header")]` on a `Vec` field, generating a method that pushes one element at a time.

## Answers to the Chapter 12 prediction questions

**12.4 Q1.** 5. The commas are tokens too, so `count!(a, b, c)` counts five token trees: `a`, `,`, `b`, `,`, `c`. A comma-aware version uses the matcher `$($x:tt),*`.

**12.4 Q2.** No. Because of hygiene, the `result` introduced by the expansion is invisible to the caller's code, which gets error E0425, "cannot find value `result` in this scope". To define a variable for the caller, the caller must supply its name: `macro_rules! def { ($name:ident) => { let $name = 5; } }` then `def!(result);`.

# Chapter 13 — Performance and Systems Programming

*Goal: make Rust programs fast on purpose — measure with benchmarks and profilers, remove allocations, choose hashers and data layouts, read the generated code — and do systems-level work: buffered and memory-mapped I/O, processes, and system calls.*

Rust's defaults are already fast: values are unboxed, generics are monomorphised, iterators compile to tight loops, and there is no garbage collector. Most remaining performance problems are the same as in C and C++ — allocation, cache misses, hashing, I/O patterns — plus a few that are specific to Rust: debug builds, unbuffered I/O, bounds checks, and SipHash. The method is the one from the C and C++ books: measure, change one thing, measure again.

**What you will be able to do after this chapter**

- Benchmark with Criterion, profile with `perf`, and read assembly from `rustc`.
- Count and remove allocations; swap the global allocator.
- Choose a hasher, a data layout, and compiler settings (LTO, codegen units, target CPU) by measurement.
- Recognise when bounds checks cost something and how to remove them safely.
- Do I/O efficiently: buffering, reading whole files, memory mapping; run processes; make system calls.
- Take a realistic data-processing program through a measured, step-by-step optimisation.

## Topic 13A — Measuring

## Unit 13.1 — Benchmarking with Criterion

**One-line summary:** **Criterion** is the standard benchmarking crate: it warms up, runs each benchmark many times, reports a confidence interval, and compares with the previous run to detect regressions. `std::hint::black_box` stops the optimiser from computing results at compile time or deleting unused work (C++ book, DT-32).

```toml
# file: bench/Cargo.toml
[package]
name = "bench"
version = "0.1.0"
edition = "2024"

[dev-dependencies]
criterion = { version = "0.5", default-features = false }
rustc-hash = "2"

[[bench]]
name = "maps"
harness = false
```

```rust
// file: bench/src/lib.rs
use std::collections::HashMap;
use std::hash::BuildHasher;

/// Counts occurrences of each key using a map with hasher `S`.
pub fn count<S: BuildHasher + Default>(keys: &[u64]) -> HashMap<u64, u32, S> {
    let mut m: HashMap<u64, u32, S> = HashMap::default();
    for &k in keys {
        *m.entry(k).or_insert(0) += 1;
    }
    m
}
```

```rust
// file: bench/benches/maps.rs
use bench::count;
use criterion::{Criterion, criterion_group, criterion_main};
use std::collections::hash_map::RandomState;
use std::hint::black_box;

fn maps(c: &mut Criterion) {
    let keys: Vec<u64> = (0..100_000u64).map(|i| i.wrapping_mul(0x9E37_79B9_7F4A_7C15) % 10_000).collect();
    let mut g = c.benchmark_group("count 100k u64 keys");
    g.bench_function("std SipHash", |b| b.iter(|| count::<RandomState>(black_box(&keys))));
    g.bench_function("FxHash", |b| b.iter(|| count::<rustc_hash::FxBuildHasher>(black_box(&keys))));
    g.finish();
}

criterion_group!(benches, maps);
criterion_main!(benches);
```

```console
$ cd bench && cargo bench -q --bench maps -- --warm-up-time 1 --measurement-time 3 2>/dev/null | grep -E '^count|time:'
count 100k u64 keys/std SipHash
                        time:   [1.2535 ms 1.2751 ms 1.3022 ms]
count 100k u64 keys/FxHash
                        time:   [286.65 µs 287.10 µs 287.62 µs]
```

This is the measurement that DT-21 promised: for small integer keys, FxHash makes the same map operations several times faster than the default SipHash. The default stays SipHash because it resists hash-flooding; switch only where keys are not attacker-controlled.

## Unit 13.2 — Profiling

**One-line summary:** On Linux, `perf record` samples where a program spends its time and `perf report` attributes the samples to functions. Build with optimisations *and* debug information (`[profile.release] debug = true`, or `CARGO_PROFILE_RELEASE_DEBUG=true`) so that inlined Rust functions can be named. `cargo flamegraph` turns the same data into a flame graph.

The chapter project profiles a real program; the command is:

```text
perf record -e cpu-clock --call-graph dwarf ./target/release/program args
perf report --stdio --no-children
```

(Hardware performance counters are not available in this book's virtual machine, so profiles here use software clock sampling, `-e cpu-clock`, as in the C and C++ books.)

<!-- expanded -->
### Going deeper: a quick profile with `perf`

On Linux, `perf record` samples the program's call stack thousands of times a second, and `perf report` lists the functions where the samples landed. Build with optimisations *and* debug symbols — for Cargo, `[profile.release] debug = true` — so the report shows function names and lines.

```rust
// file: hot.rs
#[inline(never)]
fn slow_part(n: u64) -> u64 { (0..n).map(|i| i.wrapping_mul(i) % 7).sum() }
#[inline(never)]
fn fast_part(n: u64) -> u64 { (0..n).sum() }

fn main() {
    let mut total = 0;
    for _ in 0..500 {
        total += std::hint::black_box(slow_part(2_000_000));
        total += std::hint::black_box(fast_part(2_000_000));
    }
    println!("{total}");
}
```

```console
$ rustc --edition 2024 -O -g -C force-frame-pointers=yes hot.rs && (command -v perf >/dev/null && perf record -q -g -o perf.data ./hot >/dev/null 2>&1 && perf report -i perf.data --stdio --no-children 2>/dev/null | grep -m2 -E "^ +[0-9.]+%" | sed -E "s/_RNvCs[A-Za-z0-9_]+_3hot[0-9]+/hot::/" || echo "perf is not available in this environment (install linux-tools for your kernel)")
    69.23%  hot      hot                   [.] hot::slow_part
     7.69%  hot      [kernel.kallsyms]     [k] exc_page_fault
```

When `perf` is available, `slow_part` dominates the report (the modulo by 7 on every element costs far more than the plain sum, which the compiler turns into a formula). On macOS, Instruments' Time Profiler does the same job; the `cargo flamegraph` tool wraps `perf` (or DTrace) and draws a flame graph SVG.

## Topic 13B — What makes Rust code slow or fast

## Unit 13.3 — Allocation

**One-line summary:** Allocation is the most common hidden cost in Rust as in C++: every `String`, `Vec`, `Box`, `format!`, `to_string`, `collect`, and `clone` of an owning type may allocate. Count allocations by wrapping the **global allocator**; remove them with `with_capacity`, reused buffers (`clear()` keeps capacity), `&str` instead of `String`, `Cow`, and `write!` into an existing `String`. Swapping the global allocator (`mimalloc`, `jemalloc`) helps allocation-heavy programs.

```rust
// file: allocs.rs
use std::alloc::{GlobalAlloc, Layout, System};
use std::fmt::Write;
use std::sync::atomic::{AtomicUsize, Ordering};

struct Counting;
static ALLOCS: AtomicUsize = AtomicUsize::new(0);

// SAFETY: forwards every call to the system allocator unchanged.
unsafe impl GlobalAlloc for Counting {
    unsafe fn alloc(&self, l: Layout) -> *mut u8 {
        ALLOCS.fetch_add(1, Ordering::Relaxed);
        unsafe { System.alloc(l) }
    }
    unsafe fn dealloc(&self, p: *mut u8, l: Layout) {
        unsafe { System.dealloc(p, l) }
    }
    unsafe fn realloc(&self, p: *mut u8, l: Layout, n: usize) -> *mut u8 {
        ALLOCS.fetch_add(1, Ordering::Relaxed);
        unsafe { System.realloc(p, l, n) }
    }
}

#[global_allocator]
static GLOBAL: Counting = Counting;

fn measure(label: &str, f: impl FnOnce() -> usize) {
    let before = ALLOCS.load(Ordering::Relaxed);
    let result = f();
    println!("{label:<50} {:>6} allocations (result {result})", ALLOCS.load(Ordering::Relaxed) - before);
}

fn main() {
    let words: Vec<&str> = "the quick brown fox jumps over the lazy dog".split(' ').collect();
    measure("format! per item, collect, join", || {
        let parts: Vec<String> = (0..10_000).map(|i| format!("{}-{}", words[i % 9], i)).collect();
        parts.join(",").len()
    });
    measure("one String, write! into it", || {
        let mut s = String::new();
        for i in 0..10_000 {
            if i > 0 { s.push(','); }
            write!(s, "{}-{}", words[i % 9], i).unwrap();
        }
        s.len()
    });
    measure("same, with_capacity first", || {
        let mut s = String::with_capacity(10_000 * 10);
        for i in 0..10_000 {
            if i > 0 { s.push(','); }
            write!(s, "{}-{}", words[i % 9], i).unwrap();
        }
        s.len()
    });
    measure("to_uppercase per word, new Vec each round", || {
        (0..1000).map(|_| words.iter().map(|w| w.to_uppercase()).collect::<Vec<_>>().len()).sum()
    });
    measure("reuse one buffer per word (clear keeps capacity)", || {
        let mut buf = String::new();
        let mut total = 0;
        for _ in 0..1000 {
            for w in &words {
                buf.clear();
                buf.extend(w.chars().flat_map(char::to_uppercase));
                total += buf.len();
            }
        }
        total
    });
}
```

```console
$ rustc --edition 2024 -O allocs.rs && ./allocs
format! per item, collect, join                     15302 allocations (result 97777)
one String, write! into it                             15 allocations (result 97777)
same, with_capacity first                               1 allocations (result 97777)
to_uppercase per word, new Vec each round           10000 allocations (result 9000)
reuse one buffer per word (clear keeps capacity)        1 allocations (result 35000)
```

<!-- expanded -->
### Going deeper: counting allocations with a global allocator

Rust lets a program replace its memory allocator with `#[global_allocator]`. Wrapping the system allocator with a counter is a two-minute way to find out how many allocations a piece of code makes — and to write tests that fail if a hot path starts allocating.

```rust
// file: countalloc.rs
use std::alloc::{GlobalAlloc, Layout, System};
use std::sync::atomic::{AtomicUsize, Ordering::Relaxed};

struct Counting;
static ALLOCS: AtomicUsize = AtomicUsize::new(0);

unsafe impl GlobalAlloc for Counting {
    unsafe fn alloc(&self, l: Layout) -> *mut u8 { ALLOCS.fetch_add(1, Relaxed); unsafe { System.alloc(l) } }
    unsafe fn dealloc(&self, p: *mut u8, l: Layout) { unsafe { System.dealloc(p, l) } }
}
#[global_allocator]
static GLOBAL: Counting = Counting;

fn count<R>(label: &str, f: impl FnOnce() -> R) -> R {
    let before = ALLOCS.load(Relaxed);
    let r = f();
    println!("{label:36} {} allocations", ALLOCS.load(Relaxed) - before);
    r
}

fn main() {
    let words: Vec<String> = (0..1000).map(|i| format!("w{i}")).collect();
    count("join via format! in a loop", || { let mut s = String::new(); for w in &words { s = format!("{s}{w},"); } s.len() });
    count("push_str into one String", || { let mut s = String::new(); for w in &words { s.push_str(w); s.push(','); } s.len() });
    count("push_str with_capacity", || { let mut s = String::with_capacity(6000); for w in &words { s.push_str(w); s.push(','); } s.len() });
    count("collect into Vec without size hint", || (0..1000).filter(|x| x % 3 == 0).collect::<Vec<u32>>().len());
    count("collect from exact-size iterator", || (0..1000u32).map(|x| x * 3).collect::<Vec<u32>>().len());
}
```

```console
$ rustc --edition 2024 -O countalloc.rs && ./countalloc
join via format! in a loop           1998 allocations
push_str into one String             11 allocations
push_str with_capacity               1 allocations
collect into Vec without size hint   8 allocations
collect from exact-size iterator     1 allocations
```

`format!` in a loop allocates a new string every time; `push_str` grows one buffer by doubling; with the capacity reserved up front it allocates once. `collect` uses the iterator's size hint — a `filter` cannot know its length in advance, so the vector grows by doubling, while a `map` over a range allocates exactly once.

## Unit 13.4 — Bounds checks and reading the generated code

**One-line summary:** Indexing a slice checks the index on every access, and the check can stop the compiler from vectorising a loop. Usually the optimiser removes checks it can prove redundant; when it cannot, restructure so that it can — iterate instead of indexing, `zip` two slices, slice to a known length up front (`let a = &a[..n];`) — before reaching for `get_unchecked` (which is `unsafe`). To check what happened, read the assembly: `rustc --emit asm` or `cargo rustc --release -- --emit asm`, or Compiler Explorer.

```rust
// file: bounds.rs
use std::hint::black_box;
use std::time::Instant;

#[inline(never)]
pub fn dot_indexed(a: &[f32], b: &[f32], idx: &[usize]) -> f32 {
    let mut s = 0.0;
    for &i in idx {
        s += a[i] * b[i];                          // data-dependent index: checks cannot be removed
    }
    s
}

#[inline(never)]
pub fn sum_pairs_indexed(a: &[u32], b: &[u32]) -> u32 {
    let mut s = 0u32;
    for i in 0..a.len() {
        s = s.wrapping_add(a[i].wrapping_mul(b[i]));   // b[i] may be out of range: checked every time
    }
    s
}

#[inline(never)]
pub fn sum_pairs_zip(a: &[u32], b: &[u32]) -> u32 {
    a.iter().zip(b).fold(0u32, |s, (x, y)| s.wrapping_add(x.wrapping_mul(*y)))   // no checks: vectorised
}

#[inline(never)]
pub fn sum_pairs_presliced(a: &[u32], b: &[u32]) -> u32 {
    let b = &b[..a.len()];                         // one check up front; the loop is then provably in bounds
    let mut s = 0u32;
    for i in 0..a.len() {
        s = s.wrapping_add(a[i].wrapping_mul(b[i]));
    }
    s
}

fn time(label: &str, f: impl Fn() -> u32) {
    let t = Instant::now();
    let mut r = 0;
    for _ in 0..20 {
        r = f();
    }
    println!("{label:<22} {:>6.1?}  (result {r})", t.elapsed() / 20);
}

fn main() {
    let n = 10_000_000;
    let a: Vec<u32> = (0..n).map(|i| i as u32).collect();
    let b: Vec<u32> = (0..n).map(|i| (i * 7) as u32).collect();
    time("indexed loop", || sum_pairs_indexed(black_box(&a), black_box(&b)));
    time("zip", || sum_pairs_zip(black_box(&a), black_box(&b)));
    time("pre-sliced index loop", || sum_pairs_presliced(black_box(&a), black_box(&b)));
    let idx: Vec<usize> = (0..1000).collect();
    let f: Vec<f32> = vec![1.0; 1000];
    println!("dot_indexed = {}", dot_indexed(&f, &f, &idx));
}
```

```console
$ rustc --edition 2024 -O bounds.rs && ./bounds
indexed loop            4.6ms  (result 1316715072)
zip                     4.2ms  (result 1316715072)
pre-sliced index loop   4.8ms  (result 1316715072)
dot_indexed = 1000
$ rustc --edition 2024 -O --emit asm bounds.rs -o bounds.s && for f in sum_pairs_indexed sum_pairs_zip sum_pairs_presliced dot_indexed; do body=$(awk "/^_R.*6bounds[0-9]+${f}:/,/\.cfi_endproc/" bounds.s); echo "$f: $(echo "$body" | grep -c 'panic_bounds_check') bounds-check calls, $(echo "$body" | grep -cE 'pmuludq|mulps') SIMD multiplies"; done
sum_pairs_indexed: 1 bounds-check calls, 4 SIMD multiplies
sum_pairs_zip: 0 bounds-check calls, 4 SIMD multiplies
sum_pairs_presliced: 0 bounds-check calls, 4 SIMD multiplies
dot_indexed: 2 bounds-check calls, 0 SIMD multiplies
```

The assembly tells a more interesting story than the folklore. All three `sum_pairs` versions were vectorised — the SIMD multiplies are there in each (`pmuludq`: baseline x86-64 has no single 32-bit vector multiply, so LLVM builds one from two 64-bit ones and shuffles) — and the plain indexed loop contains exactly *one* call to `panic_bounds_check`, not one per element. LLVM checked once, before the loop, whether `b` is at least as long as `a`, and then ran the vectorised loop with no checks — it pre-sliced the loop for us. The three timings differed by up to a third, and their order changed from run to run: with the same vectorised inner loop in all three, the differences are measurement noise and memory effects, not bounds checks.

`dot_indexed` is different: its indices come from data (`idx`), so nothing can be proven in advance; it keeps its bounds checks (two, one for each slice) inside a scalar loop. That is the realistic case for bounds-check cost — indirect indexing, as in sparse matrices, graphs, and hash tables. There, if profiling shows the checks matter, validate the indices once (e.g. `assert!(idx.iter().all(|&i| i < a.len().min(b.len())))`) and then use `get_unchecked` with a `// SAFETY:` comment referring to that check.

### Deep & Tricky

**DT-42. Rust reorders struct fields to minimise padding, unless you ask it not to.** *(Guarantee: Rust language — `repr(Rust)` layout is unspecified)* Unlike C, the default representation lets the compiler place fields in any order; it sorts them to reduce padding, so `struct { a: u8, b: u64, c: u8 }` is 16 bytes in Rust but 24 in C. `#[repr(C)]` gives C's in-order layout (required for FFI, Chapter 11); `#[repr(packed)]` removes padding (and makes references to misaligned fields an error); `#[repr(u8)]` fixes an enum's tag type. Never depend on `repr(Rust)` field order or offsets — they may change between compiler versions.

```rust
// file: layout.rs
#[allow(dead_code)]
struct RustOrder { a: u8, b: u64, c: u8 }
#[allow(dead_code)]
#[repr(C)]
struct COrder { a: u8, b: u64, c: u8 }
#[allow(dead_code)]
enum Small { A(u8), B(u16), C }

fn main() {
    println!("repr(Rust) {} bytes, repr(C) {} bytes, enum Small {} bytes, Option<Box<u8>> {} bytes",
             size_of::<RustOrder>(), size_of::<COrder>(), size_of::<Small>(), size_of::<Option<Box<u8>>>());
    println!("offsets (Rust): a={} b={} c={}", std::mem::offset_of!(RustOrder, a), std::mem::offset_of!(RustOrder, b), std::mem::offset_of!(RustOrder, c));
}
```

```console
$ rustc --edition 2024 layout.rs && ./layout
repr(Rust) 16 bytes, repr(C) 24 bytes, enum Small 4 bytes, Option<Box<u8>> 8 bytes
offsets (Rust): a=8 b=0 c=9
```

## Unit 13.5 — Compiler settings

**One-line summary:** Beyond `--release`, four settings matter: **LTO** (`lto = "fat"` or `"thin"`: optimise across crates, often 5–20% on code that crosses crate boundaries in hot paths), **`codegen-units = 1`** (fewer, larger units: better optimisation, slower builds), **`target-cpu`** (`RUSTFLAGS="-C target-cpu=native"` enables the build machine's full instruction set, such as AVX2 — the binary may not run on older CPUs), and **`panic = "abort"`** (smaller code, no unwinding). **PGO** (profile-guided optimisation, via `cargo pgo`) and **BOLT** go further for large programs. Measure each; none is free.

### Pattern

**P-13. Before optimising Rust code, check the four usual suspects: a debug build, unbuffered I/O, per-item allocation, and SipHash on hot maps.** *(Practice)* These account for most "Rust is slower than expected" reports, and each has a one-line fix: `--release`; wrap readers and writers in `BufReader`/`BufWriter` (and lock `stdout` once); reuse buffers and borrow instead of cloning; use a fast hasher for trusted keys. Only after that, profile and consider layout, bounds checks, and compiler flags. *Trade-off:* none of these fixes cost readability — which is why they come first.

### Mastery check

1. **Predict.** What are `size_of` of `struct S { a: u8, b: u32, c: u16 }` and of the same struct with `#[repr(C)]`?
2. **Predict.** A program writes 1,000 lines with `println!` in a loop. Roughly how many `write` system calls does it make, and how do you reduce that to a handful?

## Topic 13C — Systems programming

## Unit 13.6 — I/O: buffering, whole-file reads, memory mapping

**One-line summary:** `File` and `Stdout` are **unbuffered** at the Rust level — every `write` is a system call (stdout is line-buffered, which is just as bad for many short lines). Wrap them in `BufWriter`/`BufReader`, or lock stdout once (`io::stdout().lock()`). For whole files, `fs::read`/`fs::read_to_string` is one allocation and a few large reads. For very large files read randomly or in parallel, **memory mapping** (`memmap2` crate) maps the file into the address space and lets the OS page it in on demand.

```rust
// file: io_speed.rs
use std::fs::File;
use std::io::{BufWriter, Write};
use std::time::Instant;

fn main() -> std::io::Result<()> {
    let n = 200_000;
    let mode = std::env::args().nth(1).unwrap_or_default();     // "unbuffered", "buffered", or both
    let t = Instant::now();
    if mode != "buffered" {
        let mut f = File::create("unbuffered.txt")?;
        for i in 0..n {
            writeln!(f, "line {i}")?;                   // write(2) system calls for every line
        }
    }
    let unbuffered = t.elapsed();
    if mode == "unbuffered" {
        return Ok(());
    }

    let t = Instant::now();
    let mut f = BufWriter::new(File::create("buffered.txt")?);   // 8 KiB buffer
    for i in 0..n {
        writeln!(f, "line {i}")?;
    }
    f.flush()?;                                         // flush explicitly: errors in drop are ignored (DT-18)
    let buffered = t.elapsed();
    println!("{n} lines: unbuffered {unbuffered:.0?}, BufWriter {buffered:.1?} ({:.0}x)", unbuffered.as_secs_f64() / buffered.as_secs_f64());
    Ok(())
}
```

```console
$ rustc --edition 2024 -O io_speed.rs && ./io_speed
200000 lines: unbuffered 139ms, BufWriter 5.0ms (28x)
$ for mode in unbuffered buffered; do echo "$mode: $(strace -c -e trace=write ./io_speed $mode 2>&1 >/dev/null | awk '$NF == "write" {print $4}') write calls"; done
unbuffered: 600000 write calls
buffered: 281 write calls
```

`strace -c` counts the system calls. Without the buffer it was not one `write` per line but *three*: `writeln!` formats in pieces — the literal `line `, the number, the newline — and an unbuffered `File` passes each piece straight to the kernel. With `BufWriter`, 200,000 lines (about 2.3 MB) took a few hundred system calls, one per 8 KiB buffer.

## Unit 13.7 — Processes and system calls

**One-line summary:** `std::process::Command` runs programs with arguments, environment, working directory, and piped stdin/stdout/stderr, returning an `ExitStatus` or captured `Output` — with no shell involved, so no quoting bugs or injection. For system calls the standard library does not wrap, the `libc` crate provides raw declarations and the `nix` and `rustix` crates provide safe wrappers.

```rust
// file: procs.rs
use std::io::Write;
use std::process::{Command, Stdio};

fn main() -> std::io::Result<()> {
    let out = Command::new("uname").arg("-sr").output()?;
    print!("uname: {}", String::from_utf8_lossy(&out.stdout));

    // pipe data through `sort -n` and read the result
    let mut child = Command::new("sort").arg("-n").stdin(Stdio::piped()).stdout(Stdio::piped()).spawn()?;
    child.stdin.take().unwrap().write_all(b"10\n9\n100\n1\n")?;   // dropping stdin closes the pipe
    let sorted = child.wait_with_output()?;
    println!("sorted: {:?}", String::from_utf8_lossy(&sorted.stdout).lines().collect::<Vec<_>>());

    // arguments are passed as-is, never through a shell: this file name is harmless
    let status = Command::new("ls").arg("no such file; rm -rf /").stderr(Stdio::null()).status()?;
    println!("ls exited with {status} (code {:?})", status.code());

    println!("pid {}, parent pid {}", std::process::id(), std::os::unix::process::parent_id());
    Ok(())
}
```

```console
$ rustc --edition 2024 procs.rs && ./procs
uname: Linux 6.18.44-fc-v470
sorted: ["1", "9", "10", "100"]
ls exited with exit status: 2 (code Some(2))
pid 15867, parent pid 14816
```

<!-- expanded -->
### Going deeper: running a child process and talking to it

`std::process::Command` builds a child process: arguments, environment, working directory, and what to do with its standard input, output, and error. `output()` runs it to completion and collects everything; `spawn()` gives a `Child` you can write to and read from while it runs.

```rust
// file: pipes.rs
use std::io::Write;
use std::process::{Command, Stdio};

fn main() -> std::io::Result<()> {
    let out = Command::new("ls").arg("-1").arg("/").output()?;
    println!("ls succeeded: {}, listed bin/: {}", out.status.success(), String::from_utf8_lossy(&out.stdout).lines().any(|l| l == "bin"));

    let mut child = Command::new("sort")
        .arg("-r")
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()?;
    child.stdin.take().unwrap().write_all(b"pear\napple\nmango\n")?;   // dropping stdin sends EOF
    let sorted = child.wait_with_output()?;
    print!("sorted -r:\n{}", String::from_utf8_lossy(&sorted.stdout));

    let missing = Command::new("definitely-not-a-command").output();
    println!("missing program: {}", missing.unwrap_err().kind());
    let failing = Command::new("sh").args(["-c", "echo oops >&2; exit 3"]).output()?;
    println!("exit code {:?}, stderr {:?}", failing.status.code(), String::from_utf8_lossy(&failing.stderr).trim());
    Ok(())
}
```

```console
$ rustc --edition 2024 pipes.rs && ./pipes
ls succeeded: true, listed bin/: true
sorted -r:
pear
mango
apple
missing program: entity not found
exit code Some(3), stderr "oops"
```

`Command` never goes through a shell unless you run one yourself (`sh -c`), so arguments are passed exactly as given — no quoting problems and no shell injection. Taking `stdin` out of the child and dropping it is what tells `sort` the input has ended; forgetting that makes `wait_with_output` wait forever.

## Chapter 13 Project — Aggregating ten million measurements

**Goal:** a program reads a file of weather-station measurements — `Station name;23.4` per line — and prints, for each station in alphabetical order, the minimum, mean, and maximum temperature. (This is a scaled-down version of the "One Billion Row Challenge" that circulated among Java, Rust, and C++ programmers in 2024.) The program is written the obvious way, then optimised in measured steps to a parallel, allocation-free version, with every version producing byte-identical output.

```rust
// file: brc/gen.rs
use std::io::{BufWriter, Write};

fn main() -> std::io::Result<()> {
    let n: usize = std::env::args().nth(1).and_then(|s| s.parse().ok()).unwrap_or(10_000_000);
    let stations: [(&str, f64); 24] = [
        ("Abha", 18.0), ("Accra", 26.4), ("Bergen", 7.7), ("Bulawayo", 18.9), ("Cairo", 21.4), ("Dakar", 24.0),
        ("Edmonton", 4.2), ("Fukuoka", 17.0), ("Hanoi", 23.6), ("Ho Chi Minh City", 27.4), ("Istanbul", 13.9),
        ("Jakarta", 26.7), ("Kraków", 8.3), ("Lagos", 26.8), ("Lima", 19.0), ("Montréal", 6.8), ("Nouméa", 23.0),
        ("Oslo", 5.7), ("Reykjavík", 4.3), ("São Paulo", 19.6), ("Tromsø", 2.9), ("Ürümqi", 7.4), ("Zürich", 9.3), ("İzmir", 17.9),
    ];
    let mut x: u64 = 0x853C49E6748FEA9B;
    let mut out = BufWriter::with_capacity(1 << 20, std::io::stdout().lock());
    for _ in 0..n {
        x ^= x << 13; x ^= x >> 7; x ^= x << 17;
        let (name, mean) = stations[(x % 24) as usize];
        let noise = ((x >> 20) % 2001) as f64 / 100.0 - 10.0;          // -10.00 ..= +10.00
        writeln!(out, "{name};{:.1}", mean + noise)?;
    }
    Ok(())
}
```

```rust
// file: brc/src/main.rs
use rayon::prelude::*;
use rustc_hash::FxHashMap;
use std::collections::{BTreeMap, HashMap};
use std::io::{BufRead, BufReader, Write};
use std::time::Instant;

#[derive(Clone, Copy)]
struct Stats { min: i32, max: i32, sum: i64, count: u64 }        // temperatures in tenths of a degree

impl Stats {
    fn new(t: i32) -> Self { Stats { min: t, max: t, sum: t as i64, count: 1 } }
    fn add(&mut self, t: i32) { self.min = self.min.min(t); self.max = self.max.max(t); self.sum += t as i64; self.count += 1; }
    fn merge(&mut self, o: &Stats) { self.min = self.min.min(o.min); self.max = self.max.max(o.max); self.sum += o.sum; self.count += o.count; }
}

/// Formats the result identically for every version: {name=min/mean/max, ...}
fn report(stats: BTreeMap<String, Stats>) -> String {
    let parts: Vec<String> = stats.iter().map(|(name, s)| {
        let mean = (s.sum as f64 / s.count as f64 / 10.0 * 10.0).round() / 10.0;
        format!("{name}={:.1}/{:.1}/{:.1}", s.min as f64 / 10.0, mean, s.max as f64 / 10.0)
    }).collect();
    format!("{{{}}}", parts.join(", "))
}

/// Parses "-12.3" or "4.5" (always one decimal) into tenths, without floating point.
fn parse_tenths(b: &[u8]) -> i32 {
    let (neg, digits) = if b[0] == b'-' { (true, &b[1..]) } else { (false, b) };
    let mut v = 0i32;
    for &c in digits {
        if c != b'.' { v = v * 10 + (c - b'0') as i32; }
    }
    if neg { -v } else { v }
}

// v1: the obvious version — read_to_string, lines, split, f64, String keys, std HashMap
fn v1(path: &str) -> String {
    let text = std::fs::read_to_string(path).unwrap();
    let mut map: HashMap<String, Stats> = HashMap::new();
    for line in text.lines() {
        let (name, temp) = line.split_once(';').unwrap();
        let t = (temp.parse::<f64>().unwrap() * 10.0).round() as i32;
        map.entry(name.to_string()).and_modify(|s| s.add(t)).or_insert(Stats::new(t));
    }
    report(map.into_iter().collect())
}

// v2: streaming with BufRead::read_line into one reused String — less memory, same per-line work
fn v2(path: &str) -> String {
    let mut reader = BufReader::with_capacity(1 << 20, std::fs::File::open(path).unwrap());
    let mut line = String::new();
    let mut map: HashMap<String, Stats> = HashMap::new();
    while reader.read_line(&mut line).unwrap() > 0 {
        let (name, temp) = line.trim_end().split_once(';').unwrap();
        let t = (temp.parse::<f64>().unwrap() * 10.0).round() as i32;
        match map.get_mut(name) {                                    // allocate a key only for new stations
            Some(s) => s.add(t),
            None => { map.insert(name.to_string(), Stats::new(t)); }
        }
        line.clear();
    }
    report(map.into_iter().collect())
}

// v3: bytes instead of UTF-8 text, integer parsing, FxHash with &[u8] keys borrowed from the buffer
fn v3_chunk(data: &[u8]) -> FxHashMap<&[u8], Stats> {
    let mut map: FxHashMap<&[u8], Stats> = FxHashMap::default();
    for line in data.split(|&b| b == b'\n').filter(|l| !l.is_empty()) {
        let semi = line.iter().rposition(|&b| b == b';').unwrap();   // temperatures are short: search from the end
        let (name, temp) = (&line[..semi], &line[semi + 1..]);
        let t = parse_tenths(temp);
        match map.get_mut(name) {
            Some(s) => s.add(t),
            None => { map.insert(name, Stats::new(t)); }
        }
    }
    map
}

fn to_report(map: FxHashMap<&[u8], Stats>) -> String {
    report(map.into_iter().map(|(k, v)| (String::from_utf8(k.to_vec()).unwrap(), v)).collect())
}

fn v3(path: &str) -> String {
    let data = std::fs::read(path).unwrap();
    to_report(v3_chunk(&data))
}

// v4: memory-map the file and process chunks in parallel, splitting at line boundaries, then merge
fn v4(path: &str) -> String {
    let file = std::fs::File::open(path).unwrap();
    // SAFETY: the file is not modified while mapped (we created it and nothing else writes it).
    let data = unsafe { memmap2::Mmap::map(&file).unwrap() };
    let pieces = rayon::current_num_threads() * 4;
    let mut bounds = vec![0];
    for i in 1..pieces {
        let mut p = data.len() * i / pieces;
        while p < data.len() && data[p - 1] != b'\n' { p += 1; }    // move to the start of a line
        bounds.push(p);
    }
    bounds.push(data.len());
    let merged = bounds.windows(2).collect::<Vec<_>>().par_iter()
        .map(|w| v3_chunk(&data[w[0]..w[1]]))
        .reduce(FxHashMap::default, |mut a, b| {
            for (k, s) in b { a.entry(k).and_modify(|x| x.merge(&s)).or_insert(s); }
            a
        });
    to_report(merged)
}

fn main() {
    let path = std::env::args().nth(1).expect("usage: brc FILE");
    let versions: [(&str, fn(&str) -> String); 4] = [
        ("v1 read_to_string, lines, f64, String keys", v1),
        ("v2 BufRead, reused line buffer", v2),
        ("v3 bytes, integer parse, FxHash, &[u8] keys", v3),
        ("v4 mmap + rayon on 2 cores", v4),
    ];
    let mut outputs = Vec::new();
    let mut base = 0.0;
    for (name, f) in versions {
        let t = Instant::now();
        let out = f(&path);
        let secs = t.elapsed().as_secs_f64();
        if base == 0.0 { base = secs; }
        println!("{name:<46} {:>6.0} ms  {:>5.1}x", secs * 1e3, base / secs);
        outputs.push(out);
    }
    println!("all versions identical: {}", outputs.iter().all(|o| *o == outputs[0]));
    let mut stdout = std::io::stdout().lock();
    writeln!(stdout, "{}…", &outputs[0][..outputs[0].char_indices().nth(150).unwrap().0]).unwrap();
}
```

```toml
# file: brc/Cargo.toml
[package]
name = "brc"
version = "0.1.0"
edition = "2024"

[dependencies]
rayon = "1"
rustc-hash = "2"
memmap2 = "0.9"

[profile.release]
debug = true          # symbols for perf
```

```console
$ cd brc && rustc --edition 2024 -O gen.rs && ./gen 10000000 > measurements.txt && ls -l measurements.txt | awk '{print $5}' && head -n 3 measurements.txt
127812839
Edmonton;5.0
Accra;35.8
Lagos;29.0
$ cd brc && cargo build -q --release && ./target/release/brc measurements.txt
v1 read_to_string, lines, f64, String keys        839 ms    1.0x
v2 BufRead, reused line buffer                    731 ms    1.1x
v3 bytes, integer parse, FxHash, &[u8] keys       327 ms    2.6x
v4 mmap + rayon on 2 cores                        190 ms    4.4x
all versions identical: true
{Abha=8.0/18.0/28.0, Accra=16.4/26.4/36.4, Bergen=-2.3/7.7/17.7, Bulawayo=8.9/18.9/28.9, Cairo=11.4/21.4/31.4, Dakar=14.0/24.0/34.0, Edmonton=-5.8/4.2…
$ cd brc && perf record -q -e cpu-clock -o perf.data ./target/release/brc measurements.txt > /dev/null 2>&1; perf report -i perf.data --stdio --no-children 2>/dev/null | grep -E '^ +[0-9]' | head -n 8 | c++filt | sed -E 's/\[\.\] //; s/\[[0-9a-f]{16}\]//g; s/<[^>]{20,}>/<…>/g' | cut -c1-100
    18.12%  brc      brc                   brc::v3_chunk
    13.32%  brc      libc.so.6             __memcmp_evex_movbe
     9.83%  brc      brc                   core::str::converts::from_utf8
     8.35%  brc      brc                   brc::main
     6.51%  brc      brc                   <…>::next_match
     5.38%  brc      brc                   <…>::from_str
     4.60%  brc      brc                   core::slice::memchr::memchr_aligned
     4.31%  brc      brc                   <…> as core::hash::Hasher>::write
```

What each step removed:

- **v1 → v2:** streaming with a reused line buffer avoids holding the whole file as a `String`, and `get_mut` before `insert` stops allocating a `String` key for every line — `entry(name.to_string())` in v1 allocated ten million keys only to drop nearly all of them.
- **v2 → v3:** working on bytes skips UTF-8 validation of 140 MB; parsing temperatures as integer tenths replaces general `f64` parsing (the format always has one decimal); FxHash replaces SipHash (Unit 13.1); and keys are `&[u8]` slices borrowed from the buffer, so no key is ever allocated.
- **v3 → v4:** memory-mapping avoids copying the file into a buffer, and splitting it at line boundaries into independent chunks lets Rayon use both cores; partial maps (24 stations each) are tiny, so merging costs nothing — unlike Chapter 9's word count, where each partial map held most of the vocabulary.
- **Correctness throughout:** all versions produce the identical report. Integer tenths also make min/max/sum exact, avoiding floating-point accumulation error over ten million values.

The profile covers a full run of all four versions. The top entry is `v3_chunk` — the inner loop of both v3 and v4, so it runs twice — followed by `memcmp` (comparing station-name keys on every hash lookup), UTF-8 validation (`from_utf8`, from v1 and v2), `main` (into which v1 and v2 were inlined), the character search behind `split_once`, `f64` parsing, and the standard `HashMap`'s `entry`. Everything below `v3_chunk` is a cost that the later versions removed or reduced; the next step for a faster v4 would be visible here too — a cheaper key comparison, for example by hashing names while scanning for the `;`, as the C++ book's word counter did.

### Review checklist

- `parse_tenths` trusts its input. What happens with a malformed line such as `Oslo;abc` or `Oslo;` in each version? Which behaviour would you want, and what does it cost?
- In v4, why must chunk boundaries move forward to the start of a line? What if a single line were longer than a chunk?
- The station names include `Kraków`, `Ürümqi`, and `İzmir`. Does sorting the `String` keys in a `BTreeMap` give the order a human expects? (It sorts by UTF-8 bytes, so `Ürümqi` and `İzmir` sort after `Zürich`.)

## Answers to the Chapter 13 prediction questions

**13.5 Q1.** 8 and 12. Rust places `b` (4 bytes) first, then `c` (2) and `a` (1), and rounds up to the 4-byte alignment: 8. `repr(C)` keeps the declared order: `a` at 0, three bytes of padding, `b` at 4, `c` at 8, and two bytes of tail padding: 12 (DT-42).

**13.5 Q2.** About 1,000. Rust's `stdout` is *always* line-buffered — even when redirected to a file or a pipe, unlike C's `stdout` — so each line is flushed with its own `write`. Lock stdout once and wrap it in a `BufWriter` — `let mut out = BufWriter::new(io::stdout().lock());` then `writeln!(out, ...)` — and the 1,000 lines go out in a few writes of 8 KiB.

# Chapter 14 — Capstone: A Networked, Persistent Key-Value Store

*Goal: design, build, test, crash, and measure a real Rust system — an async TCP key-value server with a write-ahead log, group commit, snapshots, and crash recovery — using everything in this book.*

This chapter introduces no new language features. The system is **kvr**, a cousin of the C book's `kvd`: a key-value database served over TCP, made durable by a **write-ahead log** (WAL) of checksummed records, compacted into **snapshots** written atomically, and able to recover from a crash in the middle of a write. The Rust version is shaped by the language: Tokio tasks handle connections (Chapter 10), a single storage thread owns the data and needs no locks (Chapters 9 and 10), errors are values (Chapter 4), and the on-disk format is parsed with slices that cannot read out of bounds (Chapters 1 and 13).

The reference implementation below was built and tested while writing this book. It is deliberately small — about 450 lines of library and tools, plus tests — so that you can read all of it. Your own capstone should go further; Unit 14.7 lists directions.

**Capstone deliverables**

1. A design note: requirements, decisions, and failure modes (Unit 14.1).
2. The storage engine: log, recovery, snapshots (Units 14.2–14.3).
3. The server: protocol, connection tasks, and the storage thread with group commit (Unit 14.4).
4. Tests: unit tests of the file format, an integration test of the protocol, and a crash test that kills the server mid-write (Unit 14.5).
5. A load test comparing durability settings (Unit 14.6), and a design review (Unit 14.7).

## Topic 14A — Design

## Unit 14.1 — Requirements and design

### Requirements

- **R1 Protocol:** line-based text over TCP: `SET key value`, `GET key`, `DEL key`, `KEYS [prefix]`, `STATS`, `COMPACT`, `QUIT`. One response line per request: `OK`, `VALUE v`, `NIL`, `OK 0|1`, `KEYS n k1 k2 …`, `ERR message`.
- **R2 Durability:** a write is **acknowledged only after it is in the log**; with `--sync always`, only after the log is on disk (`fdatasync`). After any crash, every acknowledged write is present.
- **R3 Recovery:** at start-up, load the snapshot, replay the log, and **detect and discard a torn or corrupt tail** (a record cut off by a crash, or garbage), without losing valid records before it.
- **R4 Compaction:** `COMPACT` writes the whole map to a new snapshot **atomically** and empties the log; a crash at any point during compaction loses nothing.
- **R5 Concurrency:** hundreds of concurrent connections; no data races (guaranteed by the compiler) and no locks on the data (by design).

### Key decisions

| Decision | Choice | Why, and the alternative |
|---|---|---|
| Who owns the data | one **storage thread** owns the `Store`; connection tasks send it commands over a channel and await a `oneshot` reply | an `Arc<Mutex<Store>>` would block async worker threads during `fsync` (DT-36) |
| Making writes durable | **group commit**: the storage thread drains up to 256 queued commands, applies them, then commits the log **once** and replies to all | one `fsync` per write limits throughput to the disk's sync rate |
| Record format | `len (u32) · crc32 (u32) · payload`; payload = `tag · key len (u32) · key · value` | the length finds record boundaries; the CRC detects torn and corrupt records |
| Snapshot | same record format; written to `snapshot.tmp`, `fsync`ed, renamed over `snapshot.db`, directory `fsync`ed | renaming is atomic on POSIX file systems, so readers see the old or the new snapshot, never half of one |
| Map | `HashMap<Vec<u8>, Vec<u8>>` | `BTreeMap` would make `KEYS prefix` faster; prefix scans are rare here |

### Failure modes designed for

- **Crash while appending a record** → the tail of the log is a partial record → CRC or length check fails → recovery truncates it. Acknowledged writes are all before it, because acknowledgement follows the commit.
- **Crash after the snapshot rename, before the log is emptied** → recovery loads the new snapshot and replays the old log on top of it. This is harmless because replaying a log onto the state it produced gives the same state (DT-44).
- **Crash during the snapshot write** → `snapshot.tmp` is incomplete but `snapshot.db` is the old, complete one, and the log is untouched.

## Topic 14B — Build

## Unit 14.2 — The log and snapshot format

```rust
// file: kvr/src/wal.rs
//! The write-ahead log and snapshot format.
//!
//! Record: `len: u32 LE | crc32(payload): u32 LE | payload`
//! Payload: `tag: u8 (1 = set, 2 = delete) | key_len: u32 LE | key | value`
use std::fs::{File, OpenOptions};
use std::io::{self, BufWriter, Write};
use std::path::Path;

#[derive(Debug, Clone, PartialEq)]
pub enum Op {
    Set(Vec<u8>, Vec<u8>),
    Del(Vec<u8>),
}

fn encode_parts(tag: u8, key: &[u8], value: &[u8], out: &mut Vec<u8>) {
    let start = out.len();
    out.extend_from_slice(&((1 + 4 + key.len() + value.len()) as u32).to_le_bytes());
    out.extend_from_slice(&[0; 4]);                                // CRC placeholder
    out.push(tag);
    out.extend_from_slice(&(key.len() as u32).to_le_bytes());
    out.extend_from_slice(key);
    out.extend_from_slice(value);
    let crc = crc32fast::hash(&out[start + 8..]);
    out[start + 4..start + 8].copy_from_slice(&crc.to_le_bytes());
}

/// Appends the encoded record for `op` to `out`.
pub fn encode(op: &Op, out: &mut Vec<u8>) {
    match op {
        Op::Set(k, v) => encode_parts(1, k, v, out),
        Op::Del(k) => encode_parts(2, k, &[], out),
    }
}

fn decode_one(d: &[u8]) -> Option<(Op, usize)> {
    let len = u32::from_le_bytes(d.get(0..4)?.try_into().ok()?) as usize;
    let crc = u32::from_le_bytes(d.get(4..8)?.try_into().ok()?);
    let payload = d.get(8..8usize.checked_add(len)?)?;             // a truncated record ends here
    if payload.len() < 5 || crc32fast::hash(payload) != crc {      // a corrupt one here
        return None;
    }
    let key_len = u32::from_le_bytes(payload[1..5].try_into().ok()?) as usize;
    let key = payload.get(5..5usize.checked_add(key_len)?)?.to_vec();
    let value = payload[5 + key_len..].to_vec();
    let op = match payload[0] {
        1 => Op::Set(key, value),
        2 if value.is_empty() => Op::Del(key),
        _ => return None,
    };
    Some((op, 8 + len))
}

/// Decodes records until the end of `data` or the first invalid record.
/// Returns the operations and the number of bytes that were valid.
pub fn decode_all(data: &[u8]) -> (Vec<Op>, usize) {
    let mut ops = Vec::new();
    let mut pos = 0;
    while let Some((op, used)) = decode_one(&data[pos..]) {
        ops.push(op);
        pos += used;
    }
    (ops, pos)
}

/// What recovery found in the log.
pub struct Recovered {
    pub ops: Vec<Op>,
    pub truncated: u64,
}

/// Reads the log, and truncates an invalid tail (a torn or corrupt last write) so appends continue after valid data.
pub fn recover(path: &Path) -> io::Result<Recovered> {
    let data = match std::fs::read(path) {
        Ok(d) => d,
        Err(e) if e.kind() == io::ErrorKind::NotFound => Vec::new(),
        Err(e) => return Err(e),
    };
    let (ops, valid) = decode_all(&data);
    let truncated = (data.len() - valid) as u64;
    if truncated > 0 {
        let f = OpenOptions::new().write(true).open(path)?;
        f.set_len(valid as u64)?;
        f.sync_all()?;
    }
    Ok(Recovered { ops, truncated })
}

/// An open log, appending through a buffer.
pub struct Wal {
    out: BufWriter<File>,
    pub bytes: u64,
    scratch: Vec<u8>,
}

impl Wal {
    pub fn open_append(path: &Path) -> io::Result<Wal> {
        let f = OpenOptions::new().create(true).append(true).open(path)?;
        let bytes = f.metadata()?.len();
        Ok(Wal { out: BufWriter::with_capacity(1 << 16, f), bytes, scratch: Vec::new() })
    }

    pub fn append(&mut self, op: &Op) -> io::Result<()> {
        self.scratch.clear();
        encode(op, &mut self.scratch);
        self.out.write_all(&self.scratch)?;
        self.bytes += self.scratch.len() as u64;
        Ok(())
    }

    /// Hands buffered records to the OS; with `sync`, waits until they are on disk.
    pub fn commit(&mut self, sync: bool) -> io::Result<()> {
        self.out.flush()?;
        if sync {
            self.out.get_ref().sync_data()?;
        }
        Ok(())
    }

    /// Empties the log (after a snapshot has made its contents redundant).
    pub fn reset(&mut self) -> io::Result<()> {
        self.out.flush()?;
        self.out.get_ref().set_len(0)?;
        self.out.get_ref().sync_all()?;
        self.bytes = 0;
        Ok(())
    }
}

/// Writes all entries as a snapshot, atomically: temporary file, fsync, rename, fsync the directory.
pub fn write_snapshot<'a>(dir: &Path, entries: impl Iterator<Item = (&'a Vec<u8>, &'a Vec<u8>)>) -> io::Result<u64> {
    let tmp = dir.join("snapshot.tmp");
    let mut out = BufWriter::new(File::create(&tmp)?);
    let mut buf = Vec::new();
    let mut written = 0;
    for (k, v) in entries {
        buf.clear();
        encode_parts(1, k, v, &mut buf);
        out.write_all(&buf)?;
        written += buf.len() as u64;
    }
    let file = out.into_inner().map_err(|e| e.into_error())?;
    file.sync_all()?;                                              // the data is on disk...
    std::fs::rename(&tmp, dir.join("snapshot.db"))?;               // ...before the name points to it
    File::open(dir)?.sync_all()?;                                  // and the rename itself is durable
    Ok(written)
}

#[cfg(test)]
mod tests {
    use super::*;

    fn sample() -> Vec<Op> {
        vec![Op::Set(b"a".to_vec(), b"1".to_vec()), Op::Del(b"a".to_vec()), Op::Set(b"key".to_vec(), vec![0, 255, 10])]
    }

    #[test]
    fn round_trip() {
        let mut buf = Vec::new();
        for op in sample() {
            encode(&op, &mut buf);
        }
        assert_eq!(decode_all(&buf), (sample(), buf.len()));
    }

    #[test]
    fn every_torn_tail_is_detected() {
        let mut buf = Vec::new();
        for op in sample() {
            encode(&op, &mut buf);
        }
        let mut first_two = Vec::new();
        encode(&sample()[0], &mut first_two);
        encode(&sample()[1], &mut first_two);
        for cut in first_two.len()..buf.len() {                     // cut the last record at every byte
            let (ops, valid) = decode_all(&buf[..cut]);
            assert_eq!((ops.len(), valid), (2, first_two.len()), "cut at {cut}");
        }
    }

    #[test]
    fn corruption_stops_replay_at_the_bad_record() {
        let mut buf = Vec::new();
        for op in sample() {
            encode(&op, &mut buf);
        }
        let mut first = Vec::new();
        encode(&sample()[0], &mut first);
        buf[first.len() + 9] ^= 0x40;                               // flip a bit inside record 2
        assert_eq!(decode_all(&buf).0, vec![sample()[0].clone()]);
    }
}
```

The format is deliberately simple, and every read of it goes through `get(range)?`, so a corrupt length can never index out of bounds: it makes `decode_one` return `None`, which ends replay. The `every_torn_tail_is_detected` test simulates a crash at **every** possible byte position of the last record.

### Deep & Tricky

**DT-43. `write` and `flush` do not make data durable, and dropping a `File` does not `fsync` it.** *(OS behaviour; Library behaviour: `std::fs::File`)* `BufWriter::flush` moves bytes from your buffer into the kernel's page cache; they survive a crash of *your process* but not a power failure or kernel crash. Only `File::sync_data` (`fdatasync`) or `File::sync_all` (`fsync`) waits until the device has them. A rename is a change to the *directory*, so making it durable needs an `fsync` of the directory itself — `File::open(dir)?.sync_all()` — which is why `write_snapshot` ends that way. `File`'s `Drop` closes the descriptor without syncing and cannot report errors (DT-18).

## Unit 14.3 — The store and recovery

```rust
// file: kvr/src/store.rs
use crate::wal::{self, Op, Wal};
use std::collections::HashMap;
use std::io;
use std::path::{Path, PathBuf};

/// When a commit waits for the disk.
#[derive(Clone, Copy, Debug, PartialEq)]
pub enum SyncPolicy {
    /// fdatasync before acknowledging: survives power loss.
    Always,
    /// hand to the OS before acknowledging: survives a process crash, not power loss.
    Never,
}

#[derive(Default, Debug)]
pub struct Stats {
    pub sets: u64,
    pub dels: u64,
    pub gets: u64,
    pub commits: u64,
    pub compactions: u64,
}

pub struct Store {
    map: HashMap<Vec<u8>, Vec<u8>>,
    wal: Wal,
    dir: PathBuf,
    sync: SyncPolicy,
    pending: bool,
    pub stats: Stats,
}

fn apply(map: &mut HashMap<Vec<u8>, Vec<u8>>, op: Op) {
    match op {
        Op::Set(k, v) => {
            map.insert(k, v);
        }
        Op::Del(k) => {
            map.remove(&k);
        }
    }
}

impl Store {
    /// Opens (or creates) a store in `dir`: snapshot, then log replay. Returns a recovery report.
    pub fn open(dir: &Path, sync: SyncPolicy) -> io::Result<(Store, String)> {
        std::fs::create_dir_all(dir)?;
        let mut map = HashMap::new();
        let mut from_snapshot = 0;
        let snap = dir.join("snapshot.db");
        if snap.exists() {
            let data = std::fs::read(&snap)?;
            let (ops, valid) = wal::decode_all(&data);
            if valid != data.len() {
                return Err(io::Error::new(io::ErrorKind::InvalidData, "snapshot.db is corrupt"));
            }
            from_snapshot = ops.len();
            ops.into_iter().for_each(|op| apply(&mut map, op));
        }
        let log = dir.join("wal.log");
        let rec = wal::recover(&log)?;
        let replayed = rec.ops.len();
        rec.ops.into_iter().for_each(|op| apply(&mut map, op));
        let report = format!(
            "recovered {} keys: {from_snapshot} from snapshot, {replayed} log records replayed, {} bytes of torn/corrupt tail discarded",
            map.len(), rec.truncated
        );
        let wal = Wal::open_append(&log)?;
        Ok((Store { map, wal, dir: dir.to_path_buf(), sync, pending: false, stats: Stats::default() }, report))
    }

    pub fn get(&mut self, key: &[u8]) -> Option<&Vec<u8>> {
        self.stats.gets += 1;
        self.map.get(key)
    }

    /// Logs and applies a set. Not durable until `commit`.
    pub fn set(&mut self, key: Vec<u8>, value: Vec<u8>) -> io::Result<()> {
        let op = Op::Set(key, value);
        self.wal.append(&op)?;                                 // log first...
        apply(&mut self.map, op);                              // ...then apply
        self.pending = true;
        self.stats.sets += 1;
        Ok(())
    }

    pub fn del(&mut self, key: &[u8]) -> io::Result<bool> {
        if !self.map.contains_key(key) {
            return Ok(false);
        }
        self.wal.append(&Op::Del(key.to_vec()))?;
        self.map.remove(key);
        self.pending = true;
        self.stats.dels += 1;
        Ok(true)
    }

    /// Makes all writes so far durable according to the policy. Called once per batch.
    pub fn commit(&mut self) -> io::Result<()> {
        if self.pending {
            self.wal.commit(self.sync == SyncPolicy::Always)?;
            self.pending = false;
            self.stats.commits += 1;
        }
        Ok(())
    }

    pub fn keys(&self, prefix: &[u8]) -> Vec<&Vec<u8>> {
        let mut keys: Vec<&Vec<u8>> = self.map.keys().filter(|k| k.starts_with(prefix)).collect();
        keys.sort();
        keys
    }

    /// Writes a snapshot and empties the log. Returns (log bytes before, snapshot bytes).
    pub fn compact(&mut self) -> io::Result<(u64, u64)> {
        self.commit()?;
        let before = self.wal.bytes;
        let snapshot = wal::write_snapshot(&self.dir, self.map.iter())?;
        self.wal.reset()?;                                     // a crash before this line is harmless (DT-44)
        self.stats.compactions += 1;
        Ok((before, snapshot))
    }

    pub fn len(&self) -> usize {
        self.map.len()
    }

    pub fn is_empty(&self) -> bool {
        self.map.is_empty()
    }

    pub fn log_bytes(&self) -> u64 {
        self.wal.bytes
    }
}
```

### Deep & Tricky

**DT-44. Recovery must be idempotent: replaying a log onto the state it already produced must change nothing.** *(Design fact)* If the process crashes after the snapshot rename but before the log is emptied, recovery applies the old log on top of the new snapshot. Because every record is "set key to this value" or "delete key" — never "add 1 to key" — each key's final value is decided by the *last* record for it, which the snapshot already reflects; replaying is a no-op. A log of non-idempotent operations (increments, appends) would need sequence numbers, with the snapshot recording the last sequence number it includes. The general rule: either make operations idempotent, or make replay remember exactly where it stopped.

## Unit 14.4 — The server

```rust
// file: kvr/src/server.rs
use crate::store::Store;
use std::io;
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};
use tokio::net::{TcpListener, TcpStream};
use tokio::sync::{mpsc, oneshot};

#[derive(Debug, PartialEq)]
pub enum Cmd {
    Get(Vec<u8>),
    Set(Vec<u8>, Vec<u8>),
    Del(Vec<u8>),
    Keys(Vec<u8>),
    Stats,
    Compact,
}

type Job = (Cmd, oneshot::Sender<String>);
const MAX_BATCH: usize = 256;

/// Parses one request line.
pub fn parse(line: &str) -> Result<Cmd, String> {
    let mut parts = line.splitn(3, ' ');
    let verb = parts.next().unwrap_or("").to_ascii_uppercase();
    match (verb.as_str(), parts.next(), parts.next()) {
        ("GET", Some(k), None) => Ok(Cmd::Get(k.into())),
        ("SET", Some(k), Some(v)) => Ok(Cmd::Set(k.into(), v.into())),
        ("DEL", Some(k), None) => Ok(Cmd::Del(k.into())),
        ("KEYS", prefix, None) => Ok(Cmd::Keys(prefix.unwrap_or("").into())),
        ("STATS", None, None) => Ok(Cmd::Stats),
        ("COMPACT", None, None) => Ok(Cmd::Compact),
        _ => Err(format!("ERR cannot parse {line:?}")),
    }
}

fn execute(store: &mut Store, cmd: Cmd) -> io::Result<String> {
    Ok(match cmd {
        Cmd::Get(k) => match store.get(&k) {
            Some(v) => format!("VALUE {}", String::from_utf8_lossy(v)),
            None => "NIL".to_string(),
        },
        Cmd::Set(k, v) => {
            store.set(k, v)?;
            "OK".to_string()
        }
        Cmd::Del(k) => format!("OK {}", u8::from(store.del(&k)?)),
        Cmd::Keys(prefix) => {
            let keys = store.keys(&prefix);
            let shown: Vec<String> = keys.iter().take(20).map(|k| String::from_utf8_lossy(k).into_owned()).collect();
            format!("KEYS {} {}", keys.len(), shown.join(" "))
        }
        Cmd::Stats => format!("STATS keys={} log_bytes={} {:?}", store.len(), store.log_bytes(), store.stats),
        Cmd::Compact => {
            let (log, snap) = store.compact()?;
            format!("OK compacted {log} log bytes into a {snap}-byte snapshot")
        }
    })
}

/// The storage thread owns the Store. It executes commands in batches and commits once per batch
/// (group commit), replying only after the commit — so an acknowledged write is durable.
fn storage_thread(mut store: Store, mut jobs: mpsc::Receiver<Job>) {
    let mut batch: Vec<Job> = Vec::with_capacity(MAX_BATCH);
    let mut replies: Vec<(oneshot::Sender<String>, String)> = Vec::with_capacity(MAX_BATCH);
    while let Some(first) = jobs.blocking_recv() {
        batch.push(first);
        while batch.len() < MAX_BATCH {
            match jobs.try_recv() {
                Ok(job) => batch.push(job),
                Err(_) => break,
            }
        }
        for (cmd, reply) in batch.drain(..) {
            let answer = execute(&mut store, cmd).unwrap_or_else(|e| format!("ERR {e}"));
            replies.push((reply, answer));
        }
        let committed = store.commit();
        for (reply, answer) in replies.drain(..) {
            let _ = reply.send(match &committed {
                Ok(()) => answer,
                Err(e) => format!("ERR commit failed: {e}"),
            });
        }
    }
}

async fn connection(stream: TcpStream, jobs: mpsc::Sender<Job>) -> io::Result<()> {
    stream.set_nodelay(true)?;
    let (reader, mut writer) = stream.into_split();
    let mut lines = BufReader::new(reader).lines();
    while let Some(line) = lines.next_line().await? {
        let line = line.trim();
        if line.is_empty() {
            continue;
        }
        if line.eq_ignore_ascii_case("QUIT") {
            break;
        }
        let answer = match parse(line) {
            Ok(cmd) => {
                let (tx, rx) = oneshot::channel();
                if jobs.send((cmd, tx)).await.is_err() {
                    break;
                }
                rx.await.unwrap_or_else(|_| "ERR storage thread stopped".to_string())
            }
            Err(e) => e,
        };
        writer.write_all(answer.as_bytes()).await?;
        writer.write_all(b"\n").await?;
    }
    Ok(())
}

/// Serves connections from `listener` until the process ends.
pub async fn run(listener: TcpListener, store: Store) -> io::Result<()> {
    let (tx, rx) = mpsc::channel::<Job>(4096);
    std::thread::Builder::new().name("storage".into()).spawn(move || storage_thread(store, rx))?;
    loop {
        let (stream, _) = listener.accept().await?;
        let jobs = tx.clone();
        tokio::spawn(async move {
            let _ = connection(stream, jobs).await;
        });
    }
}
```

```rust
// file: kvr/src/lib.rs
//! kvr: a small persistent key-value server. See Chapter 14.
pub mod server;
pub mod store;
pub mod wal;
```

The storage thread is a plain OS thread, not a Tokio task, because it does blocking work (`fdatasync`) by design; `blocking_recv` lets it wait on a Tokio channel. While it syncs, more requests queue up in the channel — and the next batch commits them all with one sync. Group commit needs no tuning: under light load batches are size 1 and latency is one sync; under heavy load batches grow and the cost of each sync is shared.

### Pattern

**P-14. Give mutable state a single owner thread, and let everything else send it messages.** *(Practice: the actor model, Unit 10.4)* The storage thread can use `&mut Store` freely, with no locks, no `RefCell`, and no reasoning about interleavings — the compiler sees one owner. Batching falls out naturally (drain the queue), and so does ordering (commands from one connection are processed in order). *Trade-off:* the single thread is a throughput ceiling for the work it does; when it becomes the bottleneck, shard — several stores, each with its own thread and log, chosen by hashing the key.

The command-line front end runs the server and three client tools used in the tests: `writer` (numbered `SET`s, printing each acknowledged number), `check` (verifies a range of keys), and `load` (a concurrent benchmark).

```rust
// file: kvr/src/main.rs
use kvr::store::{Store, SyncPolicy};
use std::io;
use std::time::{Duration, Instant};
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader, Lines};
use tokio::net::TcpStream;
use tokio::net::tcp::{OwnedReadHalf, OwnedWriteHalf};

struct Client {
    lines: Lines<BufReader<OwnedReadHalf>>,
    writer: OwnedWriteHalf,
}

impl Client {
    async fn connect(port: u16) -> io::Result<Client> {
        let stream = TcpStream::connect(("127.0.0.1", port)).await?;
        stream.set_nodelay(true)?;
        let (r, writer) = stream.into_split();
        Ok(Client { lines: BufReader::new(r).lines(), writer })
    }
    async fn call(&mut self, request: &str) -> io::Result<String> {
        self.writer.write_all(format!("{request}\n").as_bytes()).await?;
        self.lines.next_line().await?.ok_or_else(|| io::Error::new(io::ErrorKind::UnexpectedEof, "server closed"))
    }
}

fn opt(args: &[String], name: &str, default: &str) -> String {
    args.windows(2).find(|w| w[0] == name).map(|w| w[1].clone()).unwrap_or_else(|| default.to_string())
}

async fn load(port: u16, conns: usize, ops: usize, keyspace: u64) -> io::Result<()> {
    let start = Instant::now();
    let mut tasks = Vec::new();
    for c in 0..conns {
        tasks.push(tokio::spawn(async move {
            let mut client = Client::connect(port).await?;
            let mut lat = Vec::with_capacity(ops);
            let mut x = 0x9E37_79B9_7F4A_7C15u64 ^ c as u64;
            for _ in 0..ops {
                x ^= x << 13; x ^= x >> 7; x ^= x << 17;
                let key = x % keyspace;
                let req = if x % 2 == 0 { format!("SET k{key} v{x}") } else { format!("GET k{key}") };
                let t = Instant::now();
                client.call(&req).await?;
                lat.push(t.elapsed());
            }
            Ok::<_, io::Error>(lat)
        }));
    }
    let mut all = Vec::new();
    for t in tasks {
        all.extend(t.await.unwrap()?);
    }
    let elapsed = start.elapsed();
    all.sort();
    let pct = |p: f64| all[((all.len() - 1) as f64 * p) as usize];
    println!("{conns} conns x {ops} ops: {:.0} ops/s, latency p50 {:.2?} p99 {:.2?} max {:.2?}",
             all.len() as f64 / elapsed.as_secs_f64(), pct(0.5), pct(0.99), all[all.len() - 1]);
    Ok(())
}

#[tokio::main]
async fn main() -> io::Result<()> {
    let args: Vec<String> = std::env::args().collect();
    let port: u16 = opt(&args, "--port", "7400").parse().expect("port");
    match args.get(1).map(String::as_str) {
        Some("serve") => {
            let dir = opt(&args, "--dir", "data");
            let sync = if opt(&args, "--sync", "always") == "never" { SyncPolicy::Never } else { SyncPolicy::Always };
            let (store, report) = Store::open(dir.as_ref(), sync)?;
            eprintln!("{report}");
            let listener = tokio::net::TcpListener::bind(("127.0.0.1", port)).await?;
            eprintln!("kvr listening on 127.0.0.1:{port}, sync {sync:?}");
            kvr::server::run(listener, store).await
        }
        Some("writer") => {
            let mut client = Client::connect(port).await?;
            for i in 0u64.. {
                match client.call(&format!("SET key{i} value-{i}")).await {
                    Ok(r) if r == "OK" => println!("{i}"),           // printed only after the ack
                    _ => break,
                }
            }
            Ok(())
        }
        Some("check") => {
            let upto: u64 = opt(&args, "--upto", "0").parse().expect("number");
            let mut client = Client::connect(port).await?;
            let mut missing = 0;
            for i in 0..=upto {
                if client.call(&format!("GET key{i}")).await? != format!("VALUE value-{i}") {
                    missing += 1;
                }
            }
            println!("checked key0..=key{upto}: {missing} missing");
            Ok(())
        }
        Some("load") => {
            let conns = opt(&args, "--conns", "50").parse().unwrap();
            let ops = opt(&args, "--ops", "1000").parse().unwrap();
            load(port, conns, ops, 10_000).await
        }
        _ => {
            eprintln!("usage: kvr serve|writer|check|load [--port P] [--dir D] [--sync always|never] [--upto N] [--conns C --ops N]");
            tokio::time::sleep(Duration::from_millis(1)).await;
            Ok(())
        }
    }
}
```

```toml
# file: kvr/Cargo.toml
[package]
name = "kvr"
version = "0.1.0"
edition = "2024"

[dependencies]
tokio = { version = "1", features = ["full"] }
crc32fast = "1"
```

## Topic 14C — Verify

## Unit 14.5 — Tests: format, protocol, and a real crash

The integration test runs the server in-process on a port chosen by the OS (`127.0.0.1:0`), talks to it over TCP, and then opens the same data directory with a second `Store` — exactly what recovery after a crash would see.

```rust
// file: kvr/tests/protocol.rs
use kvr::store::{Store, SyncPolicy};
use tokio::io::{AsyncBufReadExt, AsyncWriteExt, BufReader};
use tokio::net::{TcpListener, TcpStream};

#[tokio::test]
async fn protocol_and_persistence() {
    let dir = std::env::temp_dir().join(format!("kvr-test-{}", std::process::id()));
    let _ = std::fs::remove_dir_all(&dir);
    let (store, report) = Store::open(&dir, SyncPolicy::Always).unwrap();
    assert!(report.starts_with("recovered 0 keys"), "{report}");
    let listener = TcpListener::bind("127.0.0.1:0").await.unwrap();
    let port = listener.local_addr().unwrap().port();
    tokio::spawn(kvr::server::run(listener, store));

    let stream = TcpStream::connect(("127.0.0.1", port)).await.unwrap();
    let (r, mut w) = stream.into_split();
    let mut lines = BufReader::new(r).lines();
    let script = [
        ("SET user:1 Ada Lovelace", "OK"),
        ("SET user:2 Alan Turing", "OK"),
        ("GET user:1", "VALUE Ada Lovelace"),
        ("GET nobody", "NIL"),
        ("DEL user:2", "OK 1"),
        ("DEL user:2", "OK 0"),
        ("SET counter 1", "OK"),
        ("KEYS user:", "KEYS 1 user:1"),
        ("FROB x", "ERR cannot parse \"FROB x\""),
        ("set lower case", "OK"),
    ];
    for (request, expected) in script {
        w.write_all(format!("{request}\n").as_bytes()).await.unwrap();
        assert_eq!(lines.next_line().await.unwrap().unwrap(), expected, "request {request:?}");
    }

    // Everything acknowledged is in the log: a second Store sees it (as recovery would).
    let (mut reopened, report) = Store::open(&dir, SyncPolicy::Always).unwrap();
    assert!(report.starts_with("recovered 3 keys"), "{report}");
    assert_eq!(reopened.get(b"user:1").map(|v| v.as_slice()), Some(&b"Ada Lovelace"[..]));
    assert_eq!(reopened.get(b"user:2"), None);
    let _ = std::fs::remove_dir_all(&dir);
}

#[test]
fn compaction_preserves_state_across_restarts() {
    let dir = std::env::temp_dir().join(format!("kvr-compact-{}", std::process::id()));
    let _ = std::fs::remove_dir_all(&dir);
    {
        let (mut s, _) = Store::open(&dir, SyncPolicy::Never).unwrap();
        for i in 0..1000 {
            s.set(format!("k{}", i % 100).into_bytes(), format!("v{i}").into_bytes()).unwrap();
        }
        s.commit().unwrap();
        let (log, snap) = s.compact().unwrap();
        assert!(snap < log / 5, "snapshot {snap} should be much smaller than log {log}");
        s.set(b"after".to_vec(), b"compaction".to_vec()).unwrap();
        s.commit().unwrap();
    }
    let (mut s, report) = Store::open(&dir, SyncPolicy::Never).unwrap();
    assert_eq!(report, "recovered 101 keys: 100 from snapshot, 1 log records replayed, 0 bytes of torn/corrupt tail discarded");
    assert_eq!(s.get(b"k7").unwrap(), b"v907");
    let _ = std::fs::remove_dir_all(&dir);
}
```

```console
$ cd kvr && cargo build -q --release && cargo test -q --release 2>&1 | grep -E 'test result'
test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.04s
test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The four result lines are the library's unit tests (the three format tests), the binary (no tests), the integration tests in `tests/protocol.rs`, and the doctests.

Now a manual session, then the real test of R2 and R3: **kill the server with `SIGKILL` while a client is writing**, append garbage to the log to simulate a torn record, restart, and check that every write the client saw acknowledged is present.

```console
$ cd kvr && rm -rf demo && (./target/release/kvr serve --dir demo --port 7400 2> demo.log & echo $! > server.pid) && sleep 0.5 && printf 'SET lang Rust\nSET year 2015\nGET lang\nKEYS\nDEL year\nSTATS\n' | nc -q 1 127.0.0.1 7400; kill $(cat server.pid); sleep 0.3; cat demo.log
OK
OK
VALUE Rust
KEYS 2 lang year
OK 1
STATS keys=1 log_bytes=59 Stats { sets: 2, dels: 1, gets: 1, commits: 3, compactions: 0 }
recovered 0 keys: 0 from snapshot, 0 log records replayed, 0 bytes of torn/corrupt tail discarded
kvr listening on 127.0.0.1:7400, sync Always
$ cd kvr && (./target/release/kvr serve --dir demo --port 7400 2> demo.log & echo $! > server.pid) && sleep 0.5 && printf 'GET lang\nGET year\n' | nc -q 1 127.0.0.1 7400; kill $(cat server.pid); sleep 0.3; head -n 1 demo.log
VALUE Rust
NIL
recovered 1 keys: 0 from snapshot, 3 log records replayed, 0 bytes of torn/corrupt tail discarded
$ cd kvr && rm -rf crash && (./target/release/kvr serve --dir crash --port 7401 2> crash.log & echo $! > server.pid) && sleep 0.5 && (./target/release/kvr writer --port 7401 > acked.txt 2>/dev/null &) && sleep 2 && kill -9 $(cat server.pid) && sleep 0.5 && echo "acknowledged writes: $(wc -l < acked.txt), last acked key$(tail -n 1 acked.txt)" && printf '\x40\x00\x00\x00garbage' >> crash/wal.log && ls -l crash | awk 'NR>1 {print $5, $9}'
acknowledged writes: 7765, last acked key7764
230741 wal.log
$ cd kvr && (./target/release/kvr serve --dir crash --port 7401 2> crash.log & echo $! > server.pid) && sleep 1 && cat crash.log && ./target/release/kvr check --port 7401 --upto $(tail -n 1 acked.txt) && printf 'STATS\nCOMPACT\nSTATS\n' | nc -q 1 127.0.0.1 7401; kill $(cat server.pid)
recovered 7765 keys: 0 from snapshot, 7765 log records replayed, 11 bytes of torn/corrupt tail discarded
kvr listening on 127.0.0.1:7401, sync Always
checked key0..=key7764: 0 missing
STATS keys=7765 log_bytes=230730 Stats { sets: 0, dels: 0, gets: 7765, commits: 0, compactions: 0 }
OK compacted 230730 log bytes into a 230730-byte snapshot
STATS keys=7765 log_bytes=0 Stats { sets: 0, dels: 0, gets: 7765, commits: 0, compactions: 1 }
$ cd kvr && (./target/release/kvr serve --dir crash --port 7401 2> crash.log & echo $! > server.pid) && sleep 1 && head -n 1 crash.log && ./target/release/kvr check --port 7401 --upto $(tail -n 1 acked.txt); kill $(cat server.pid)
recovered 7765 keys: 7765 from snapshot, 0 log records replayed, 0 bytes of torn/corrupt tail discarded
checked key0..=key7764: 0 missing
```

What the crash test established:

- **Every acknowledged write survived `kill -9`** — 7,765 of them in two seconds — and recovery discarded exactly the 11 bytes of simulated torn record (a length field promising a 64-byte payload, then the 7 bytes `garbage`, read as a CRC and the first 3 bytes of a payload that never arrived) without losing a single valid record before them.
- **Compaction** rewrote the log into a snapshot holding only the live keys, and the next restart loaded the snapshot, replayed an empty log, and still found every acknowledged key. (Here the snapshot is exactly as large as the log, because the writer set each key once; with overwrites and deletes the snapshot is much smaller — five times or more in the `compaction_preserves_state_across_restarts` test.)
- What the test **cannot** show: power loss. `kill -9` destroys the process but not the kernel's page cache, so even `--sync never` would pass this test — the data was handed to the OS before each acknowledgement. Only the `fdatasync` in `--sync always` protects against power failure or a kernel crash, and testing that needs a VM whose virtual disk can be cut off, or a tool like `dm-flakey`/`LazyFS`; it was not done here (the same limitation as the C book's capstone).

## Topic 14D — Measure

## Unit 14.6 — Load test: the price of durability

```console
$ cd kvr && rm -rf la ln && (./target/release/kvr serve --dir la --port 7402 --sync always 2>/dev/null & echo $! > a.pid) && (./target/release/kvr serve --dir ln --port 7403 --sync never 2>/dev/null & echo $! > n.pid) && sleep 0.5 && for c in 1 50; do echo "sync always: $(./target/release/kvr load --port 7402 --conns $c --ops 2000)"; echo "sync never:  $(./target/release/kvr load --port 7403 --conns $c --ops 2000)"; done; printf 'STATS\n' | nc -q 1 127.0.0.1 7402; kill $(cat a.pid) $(cat n.pid)
sync always: 1 conns x 2000 ops: 6469 ops/s, latency p50 155.79µs p99 449.52µs max 3.90ms
sync never:  1 conns x 2000 ops: 14346 ops/s, latency p50 52.26µs p99 305.82µs max 5.05ms
sync always: 50 conns x 2000 ops: 61913 ops/s, latency p50 724.98µs p99 2.50ms max 24.60ms
sync never:  50 conns x 2000 ops: 109059 ops/s, latency p50 422.09µs p99 992.57µs max 4.56ms
STATS keys=5000 log_bytes=1951414 Stats { sets: 50967, dels: 0, gets: 51033, commits: 5323, compactions: 0 }
```

Reading the numbers:

- **With one connection, `sync always` costs one `fdatasync` per write** — the client waits for each — and `sync never` was a little over twice as fast. This virtual machine's disk completes a sync in roughly 100 µs, which is unusually quick (a laptop NVMe drive takes from tens of microseconds to a millisecond depending on its cache, a hard disk around 10 ms, and many cloud volumes several milliseconds); on slower storage the gap is far larger. Measure on the hardware you will deploy on.
- **With 50 connections, group commit raised `sync always` throughput almost tenfold** and narrowed the gap to `sync never`. While one sync runs, other writes queue up; the next batch commits them all with one sync. `STATS` shows the evidence: about 51,000 `SET`s needed only 5,323 commits — roughly ten writes per `fdatasync`.
- **Tail latency** (p99 and max) is dominated by sync time and by batches that wait behind a slow sync — the reason latency-sensitive databases keep their logs on fast, dedicated devices.

## Unit 14.7 — Design review and where to go next

| Requirement | Status | Evidence |
|---|---|---|
| R1 protocol | met | integration test script, including errors and case-insensitive verbs |
| R2 acknowledged writes are durable | met for process crashes; designed but not tested for power loss | crash test with `kill -9`; `fdatasync` before ack in `--sync always` |
| R3 torn-tail recovery | met | unit test cutting the last record at every byte; crash test with an injected torn record |
| R4 atomic compaction | met | rename-based snapshot, directory `fsync`, idempotent replay (DT-44); compaction test and restart |
| R5 concurrency without data races or locks on data | met | single storage thread (P-14); hundreds of concurrent connections in the load test |

**Known limitations and next steps** (pick at least two):

1. **Compaction blocks all requests** while the snapshot is written. Write the snapshot from a copy-on-write view or in a background thread while new writes go to a fresh log segment.
2. **The whole data set must fit in memory.** An LSM tree (sorted runs on disk, merged in the background) or a B-tree file removes that limit.
3. **Binary-safe protocol.** Values cannot contain newlines; adopt RESP (the Redis protocol) so that existing Redis clients work.
4. **Expiry (TTL)** and atomic operations (`INCR`, compare-and-set) — with care for DT-44: they are not idempotent, so the log needs sequence numbers.
5. **Replication:** stream the WAL to a follower that applies it; then failover.
6. **Fuzz the decoder** (`cargo fuzz`, as in the C++ book's Chapter 13) and property-test the store against a `HashMap` model under random operations and random crash points (the C++ book's differential-testing pattern, P-9 there).
7. **Metrics and tracing** with the `tracing` crate, and graceful shutdown on `SIGTERM` (Tokio's `signal` module).

### Mastery check

1. **Predict.** If the storage thread replied to each command immediately after `execute` and committed afterwards, which requirement would break, and what would the crash test show?
2. **Predict.** A record header is written completely, but the crash happens after only half of its payload. What does `decode_one` return, and what does recovery do with the file?

## Answers to the Chapter 14 prediction questions

**14.7 Q1.** R2. A client could receive `OK` for a write that is still only in the `BufWriter` (not even in the page cache); a `kill -9` at that moment would lose it, and `check` would report missing keys up to the batch size. With `--sync always` the window would include the time until the next `fdatasync` as well.

**14.7 Q2.** `None`: `d.get(8..8 + len)` fails because fewer than `len` bytes remain. Replay stops after the previous record, and `recover` truncates the file to the end of the last valid record, so new appends continue from a clean boundary.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | Safe Rust guarantees no undefined behaviour — but the guarantee is only as strong as the `unsafe` code underneath it. | Unit 0.1 |
| DT-2 | Integer overflow panics in debug builds and wraps in release builds — and neither is undefined behaviour. | Unit 0.3 |
| DT-3 | The crate, not the file, is the unit of compilation — so a large crate rebuilds as a whole. | Unit 0.4 |
| DT-4 | `as` from a float to an integer saturates, and NaN becomes 0 — it is never undefined behaviour. | Unit 1.2 |
| DT-5 | Arrays are `Copy` when their elements are, so passing `[i32; 1000]` to a function or assigning it copies 4000 bytes — and a large array declared as a local lives on the stack, where `[0u8; 16 * 1024 * 1024]` overflows the main thread's 8 MiB stack. | Unit 1.3 |
| DT-6 | `len()` on a string counts bytes, and slicing in the middle of a character panics. | Unit 1.4 |
| DT-7 | `f64` does not implement `Ord`, so `sort()` and `max()` do not compile on floats. | Unit 1.8 |
| DT-8 | A move is a bitwise copy of the value's bytes — the compiler then forbids using the source. | Unit 2.1 |
| DT-9 | `&mut` means *exclusive*, not just *mutable* — and the compiler optimises on that guarantee. | Unit 2.3 |
| DT-10 | Returning a reference into a value you create inside the function is always an error — but the fix depends on *why* you wanted a reference. If the caller needs the data, return the owned value (`String`, not `&str`); if the data is constant, return `&'static str`; if it is a view of an argument, tie the lifetimes together. | Unit 2.4 |
| DT-11 | `T: 'static` does not mean "lives forever"; it means "contains no borrows shorter than the whole program". | Unit 2.6 |
| DT-12 | Struct update syntax moves the fields it copies — which can make the source partially moved. | Unit 3.1 |
| DT-13 | Derived `PartialOrd`/`Ord` compare fields in declaration order, so reordering fields silently changes sorting. | Unit 3.3 |
| DT-14 | `Option<&T>`, `Option<Box<T>>`, and `Option<NonZeroU32>` cost nothing extra: the compiler stores `None` in an impossible value (a "niche"). | Unit 3.4 |
| DT-15 | A temporary created in the scrutinee of `if let` / `while let` / `match` lives until the end of the whole statement — so `if let Some(x) = mutex.lock().unwrap().get(0) { ... }` holds the lock for the entire `if let` body *and* its `else` branch. Edition 2024 shortened this for `if let` (the temporary is dropped before `else`), but `match` and `while let` keep the old rule. | Unit 3.6 |
| DT-16 | `?` converts the error with `From::from`, so a function can use `?` on several different error types only if its own error type has a `From` impl for each. | Unit 4.2 |
| DT-17 | `Box<dyn Error>` is not `Send + Sync`, so it cannot be sent across threads or held across an `.await` in a spawned Tokio task; library code that returns `Box<dyn Error>` forces that limit on every caller. Use `Box<dyn Error + Send + Sync + 'static>` (what `anyhow::Error` wraps), or a concrete error enum. | Unit 4.4 |
| DT-18 | A panic while already unwinding aborts the process — so `Drop` implementations must not panic. | Unit 4.5 |
| DT-19 | The orphan rule: you may implement a trait for a type only if your crate defines the trait or the type. | Unit 5.1 |
| DT-20 | Only *dyn-compatible* traits can be used as `dyn Trait`: no generic methods, no methods returning `Self`, no associated constants. | Unit 5.4 |
| DT-21 | `HashMap`'s default hasher is SipHash-1-3 with a random key — chosen for resistance to hash-flooding attacks, not for speed. | Unit 6.2 |
| DT-22 | Since Rust 2021, closures capture *disjoint fields*, not whole variables. | Unit 6.3 |
| DT-23 | Each closure has its own unique, unnameable type — even two closures with identical code — so `vec![¦x¦ x + 1, ¦x¦ x + 2]` does not compile, and `if c { ¦x¦ x + 1 } else { ¦x¦ x * 2 }` does not either. Store them as `Box<dyn Fn(i32) -> i32>` (or `fn(i32) -> i32` when they capture nothing: non-capturing closures coerce to function pointers). | Unit 6.3 |
| DT-24 | Iterator adaptors are lazy: an adaptor chain that is never consumed does nothing, and side effects inside it run in an interleaved order. | Unit 6.5 |
| DT-25 | Features are unified across the whole dependency graph: if any crate enables a feature of a dependency, every user of that dependency gets it. | Unit 7.2 |
| DT-26 | Cargo picks the newest compatible version of each dependency, and `Cargo.lock` freezes that choice. | Unit 7.2 |
| DT-27 | Integration tests in `tests/` can only test a library crate — a binary-only package has nothing for them to import. | Unit 7.4 |
| DT-28 | By default, serde *ignores* unknown fields when deserialising, so a misspelt key in a config file (`"replica": 3` for `replicas`) is silently dropped and the default is used. Add `#[serde(deny_unknown_fields)]` to configuration types, where a typo should be an error. | Unit 7.6 |
| DT-29 | `let _ = value;` drops immediately; `let _guard = value;` lives to the end of the scope. | Unit 8.2 |
| DT-30 | A `RefCell` borrow held across a call can panic far from the bug — keep borrows in the narrowest scope. | Unit 8.4 |
| DT-31 | `Send` and `Sync` are checked only at thread boundaries — but they are inferred from every field, so one non-`Sync` field anywhere makes the whole type non-`Sync`. | Unit 9.2 |
| DT-32 | Rust prevents data races, not deadlocks — and a `MutexGuard` held across an `await` or a long call is the usual way to create one. | Unit 9.4 |
| DT-33 | A thread that panics while holding a `Mutex` *poisons* it: every later `lock()` returns `Err(PoisonError)`, so the common `lock().unwrap()` turns one thread's panic into a panic in every thread that touches the lock. The data may be half-updated; if you know it is still consistent, `into_inner()` on the error recovers the guard. | Unit 9.4 |
| DT-34 | Futures are lazy: calling an `async fn` does nothing until the future is polled — and a future that is never awaited never runs. | Unit 10.1 |
| DT-35 | A future may be moved before it is first polled but not after — which is what `Pin` expresses. | Unit 10.1 |
| DT-36 | Blocking inside an async task stalls every task scheduled on that worker thread. | Unit 10.3 |
| DT-37 | In `select!`, a branch's future is dropped when another wins — so a future that has partly completed its work loses that work. | Unit 10.5 |
| DT-38 | Creating a `&mut` that aliases another live reference is undefined behaviour even if neither is ever used to write. | Unit 11.2 |
| DT-39 | C macros and inline functions are not symbols, so FFI cannot call them — find the function the macro expands to. | Unit 11.5 |
| DT-40 | Macro hygiene: identifiers created inside a macro cannot see or shadow the caller's — but items (functions, types, statics) are not hygienic. | Unit 12.2 |
| DT-41 | Procedural macros are compiled for and run on the build machine, inside the compiler — and they can run arbitrary code. | Unit 12.3 |
| DT-42 | Rust reorders struct fields to minimise padding, unless you ask it not to. | Unit 13.4 |
| DT-43 | `write` and `flush` do not make data durable, and dropping a `File` does not `fsync` it. | Unit 14.2 |
| DT-44 | Recovery must be idempotent: replaying a log onto the state it already produced must change nothing. | Unit 14.3 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Make `cargo fmt --check`, `cargo clippy -- -D warnings`, and `cargo test` the gate for every change. | Unit 0.5 |
| P-2 | Take parameters by reference unless the function needs to own the value. | Unit 2.5 |
| P-3 | Make invalid states unrepresentable. | Unit 3.6 |
| P-4 | Libraries return typed errors; applications use `anyhow` with context. | Unit 4.4 |
| P-5 | Prefer generics for performance and for single-type call sites; prefer trait objects for heterogeneous collections, plugin boundaries, and keeping compile times and binary size down. | Unit 5.5 |
| P-6 | Write data processing as iterator pipelines; fall back to a `for` loop when the logic has early exits across several collections or complex state. | Unit 6.5 |
| P-7 | Keep `main.rs` thin and put logic in a library; give each public item a doc comment with a tested example. | Unit 7.5 |
| P-8 | For graphs and trees with back-links, prefer an arena with indices over `Rc<RefCell<T>>`. | Unit 8.5 |
| P-9 | Prefer, in order: no sharing (move data to threads, collect results), message passing, then shared state behind a lock — and reach for atomics only for simple counters and flags, or with a proven algorithm. | Unit 9.6 |
| P-10 | Use async for many concurrent waits; use threads (or `spawn_blocking`/Rayon) for computation; and give each piece of shared state one owner task. | Unit 10.5 |
| P-11 | Wrap every C library in two layers: a raw `-sys` layer of exact declarations, and a safe layer that owns resources with RAII and turns error codes into `Result`. | Unit 11.5 |
| P-12 | Use a function first, generics second, a declarative macro third, and a procedural macro last. | Unit 12.4 |
| P-13 | Before optimising Rust code, check the four usual suspects: a debug build, unbuffered I/O, per-item allocation, and SipHash on hot maps. | Unit 13.5 |
| P-14 | Give mutable state a single owner thread, and let everything else send it messages. | Unit 14.4 |

# Appendix B — Glossary

Short definitions of the terms used in this book. The unit given (where there is one) explains the term fully.

| Term | Meaning |
| --- | --- |
| Actor | A task or thread that exclusively owns some state and receives commands as messages (Units 10.4, 14.4) |
| Arc | Atomically reference-counted shared pointer; the thread-safe `Rc` (Unit 9.4) |
| Arena | A collection that owns many objects, referred to by index rather than by pointer (P-8) |
| Associated type | A type named inside a trait and fixed by each implementation, e.g. `Iterator::Item` (Unit 5.5) |
| `async` / `.await` | Syntax that turns a function or block into a future, and waits for a future without blocking the thread (Unit 10.1) |
| Auto trait | A trait the compiler implements automatically from a type's fields: `Send`, `Sync`, `Unpin` (Unit 9.2) |
| Blanket implementation | `impl<T: Bound> Trait for T`: implements a trait for every type meeting a bound (Unit 5.5) |
| Borrow checker | The compiler pass that enforces the borrowing and lifetime rules on MIR (Units 0.4, 2.3) |
| Borrowing | Accessing a value through a reference without taking ownership (Unit 2.3) |
| `Box<T>` | An owning pointer to a heap value (Unit 8.1) |
| Cargo | Rust's build tool and package manager (Unit 0.3) |
| Clippy | The official linter: hundreds of checks for bugs, performance, and style (Unit 0.5) |
| Closure | An anonymous function that can capture variables from its environment (Unit 6.3) |
| Coherence | The guarantee that at most one implementation of a trait exists for a type, enforced by the orphan rule (DT-19) |
| `Copy` | A marker trait for types duplicated by a plain bit copy instead of being moved (Unit 2.1) |
| `Cow` | Clone-on-write: either borrowed or owned data (Unit 8.6) |
| Crate | The unit of compilation: a library or a binary (Units 0.4, 7.2) |
| Data race | Unsynchronised concurrent access to memory, at least one a write; impossible in safe Rust (Chapter 9) |
| Deref coercion | Automatic conversion of `&Wrapper` to `&Target` through `Deref`, e.g. `&String` to `&str` (Unit 8.2) |
| Derive | `#[derive(...)]`: generates trait implementations from a type's definition (Unit 3.3) |
| Doctest | A code example in a documentation comment, compiled and run by `cargo test` (Unit 7.4) |
| `Drop` | The trait whose method runs when a value goes out of scope (Units 2.1, 8.2) |
| `dyn Trait` | A trait object: a value of some type implementing the trait, dispatched through a vtable (Unit 5.4) |
| Edition | A versioned set of language rules chosen per crate (2015, 2018, 2021, 2024) (Unit 0.1, Appendix C) |
| Enum | A type whose value is one of several variants, each of which may carry data (Unit 3.4) |
| Executor | The part of an async runtime that polls futures when they can make progress (Unit 10.1) |
| Fat pointer | A pointer plus extra data: a length for slices and `str`, a vtable for trait objects (Unit 5.4) |
| Feature (Cargo) | A named, additive switch that enables optional code and dependencies (Unit 7.2) |
| FFI | Foreign function interface: calling C from Rust and Rust from C (Units 11.4, 11.5) |
| `Fn` / `FnMut` / `FnOnce` | The traits of closures that read, modify, or consume their captures (Unit 6.3) |
| Future | A value representing a computation that may not have finished; polled to completion (Unit 10.1) |
| Generic | Code parameterised over types, specialised by monomorphisation (Unit 5.2) |
| Group commit | Making a batch of writes durable with one disk sync (Unit 14.4) |
| Hygiene | The property that identifiers introduced by a macro cannot clash with the caller's (DT-40) |
| Interior mutability | Mutation through a shared reference, made safe by `Cell`, `RefCell`, `Mutex`, atomics (Unit 8.4) |
| Iterator | A type with a `next` method returning `Option<Item>`; the basis of `for` loops and adaptor chains (Unit 6.4) |
| Lifetime | The region of code in which a reference is valid; written `'a` (Unit 2.6) |
| Lifetime elision | Rules that let the compiler infer lifetime annotations in common function signatures (Unit 2.6) |
| MIR | Mid-level intermediate representation, where borrow checking runs (Unit 0.4) |
| Miri | An interpreter for Rust's MIR that detects undefined behaviour in unsafe code (Unit 11.2) |
| Monomorphisation | Generating a separate copy of generic code for each set of concrete type arguments (Unit 0.4) |
| Move | Transferring ownership; the source can no longer be used (Unit 2.1) |
| Niche | An impossible bit pattern of a type that the compiler uses to store an enum's tag (DT-14) |
| `Option<T>` | `Some(T)` or `None`; Rust's replacement for null (Unit 3.4) |
| Orphan rule | You may implement a trait for a type only if your crate defines one of them (DT-19) |
| Ownership | Each value has one owner, which drops it when it goes out of scope (Unit 2.1) |
| Panic | An unrecoverable error that unwinds (or aborts) the current thread (Unit 4.1) |
| `PhantomData` | A zero-sized marker that makes a type "use" a type parameter it does not store (Chapter 5 project) |
| `Pin` | A pointer wrapper guaranteeing the pointee will not move; needed for self-referential futures (DT-35) |
| Poisoning | A `Mutex` marking its data as possibly inconsistent after a panic while locked (Unit 9.4) |
| Procedural macro | A Rust function, run by the compiler, that transforms token streams (Unit 12.3) |
| `Rc` / `Weak` | A reference-counted shared pointer, and its non-owning counterpart (Units 8.3, 8.5) |
| `RefCell` | Interior mutability with borrowing rules checked at run time (Unit 8.4) |
| `Result<T, E>` | `Ok(T)` or `Err(E)`: the type of fallible operations (Unit 4.2) |
| Scoped thread | A thread that may borrow from its parent, joined before the scope ends (Unit 9.1) |
| `Send` / `Sync` | Marker traits: safe to move to / share with another thread (Unit 9.2) |
| Shadowing | Declaring a new variable with the name of an existing one (Unit 1.1) |
| Slice | A view `&[T]` or `&str` of a contiguous sequence: pointer plus length (Units 1.3, 2.5) |
| Soundness | The property that no safe code can cause undefined behaviour through an API (DT-1, Unit 11.3) |
| `'static` | The lifetime of the whole program; as a bound, "contains no shorter borrows" (DT-11) |
| Tokio | The most widely used async runtime (Unit 10.2) |
| Trait | A named set of methods a type can implement; Rust's mechanism for shared behaviour (Unit 5.1) |
| Undefined behaviour | Behaviour for which the language imposes no requirements; possible only through `unsafe` (Unit 11.1) |
| `unsafe` | A block, function, or trait whose correctness the programmer, not the compiler, guarantees (Unit 11.1) |
| Unwinding | Running destructors up the stack after a panic (Unit 4.1) |
| Vtable | A table of function pointers for one type's implementation of a trait (Unit 5.4) |
| WAL | Write-ahead log: changes are recorded durably before they are acknowledged (Unit 14.2) |
| Workspace | A set of packages sharing one `Cargo.lock` and `target/` directory (Unit 7.3) |
| Zero-cost abstraction | An abstraction that compiles to code as fast as the hand-written equivalent (Unit 6.4) |

# Appendix C — Editions, Versions, and Toolchain

## Release model

A new stable Rust compiler is released **every six weeks**, and each release is backwards compatible with code written for earlier stable releases of the same edition. A crate declares a **minimum supported Rust version** with `rust-version = "1.85"` in `Cargo.toml`; Cargo refuses to build it with an older compiler, and its resolver can prefer dependency versions compatible with it.

Changes that would break existing code are grouped into **editions**. Each crate chooses one in `Cargo.toml`, crates of different editions link together freely, and `cargo fix --edition` migrates code mechanically.

| Edition | Stable in | Main changes |
| --- | --- | --- |
| 2015 | Rust 1.0 (May 2015) | the original language |
| 2018 | Rust 1.31 | the module system as it is today (`crate::`, no `extern crate`); `dyn Trait` syntax; non-lexical lifetimes; `async`/`await` became reserved keywords |
| 2021 | Rust 1.56 | disjoint closure captures (DT-22); `IntoIterator` for arrays by value; Cargo feature resolver 2; panic macro consistency; reserved prefixes (`k#`, `f"..."`) |
| 2024 | Rust 1.85 (February 2025) | **the edition of this book.** `unsafe extern` blocks and `safe` items (Unit 11.4); unsafe operations inside `unsafe fn` need their own `unsafe` blocks (Unit 11.1); `unsafe(no_mangle)` and other unsafe attributes (Unit 11.5); `std::env::set_var` is `unsafe` (Unit 7.4); `static mut` references denied (use `&raw const`/`&raw mut`, Unit 9.5); temporary lifetimes shortened in `if let` and block tails (DT-30); RPIT lifetime capture rules; `gen` reserved; Cargo resolver 3 (MSRV-aware); `Future` and `IntoFuture` in the prelude |

## Language and library features by version

Features used in this book and the Rust version that stabilised them:

| Version | Features |
| --- | --- |
| 1.39 (2019) | `async`/`.await` |
| 1.45 | `as` from float to int saturates (DT-4) |
| 1.51 | const generics (`[T; N]` with generic `N`) |
| 1.53 | arrays implement `IntoIterator` |
| 1.58 | captured identifiers in format strings (`println!("{x}")`) |
| 1.63 | scoped threads (`std::thread::scope`) |
| 1.65 | `let`-`else`; generic associated types |
| 1.70 | `OnceCell`, `OnceLock`; `is_some_and` |
| 1.75 | `async fn` and `impl Trait` return types in traits |
| 1.77 | `offset_of!`; C-string literals `c"..."` |
| 1.80 | `LazyLock`, `LazyCell`; exclusive range patterns (`a..b` in `match`) |
| 1.81 | `#[expect(lint)]`; unwinding out of `extern "C"` functions aborts |
| 1.82 | `&raw const` / `&raw mut`; `unsafe extern` blocks; `Option::is_none_or` |
| 1.85 | edition 2024; async closures |
| 1.88 | `let` chains in `if` and `while` (edition 2024) |

Before using a newer feature in a library, check its `rust-version` and your users' compilers.

## Toolchain components

| Tool | Purpose | Chapter |
| --- | --- | --- |
| `rustup` | installs and updates toolchains (stable, beta, nightly) and components | 0 |
| `rustc` | the compiler | 0 |
| `cargo` | build, dependencies, test, bench, doc, publish | 0, 7 |
| `rustfmt` / `cargo fmt` | formatter | 0 |
| `clippy` / `cargo clippy` | linter | 0 |
| `rustdoc` / `cargo doc` | documentation generator | 7 |
| `rust-analyzer` | language server for editors | 0 |
| `miri` (nightly) | undefined-behaviour interpreter | 11 |
| `cargo bench` + Criterion | benchmarking | 13 |
| `cargo-expand`, `cargo-flamegraph`, `cargo-fuzz`, `cargo-udeps`, `cargo-deny`, `cargo-audit` | popular third-party Cargo subcommands: macro expansion, profiling, fuzzing, unused dependencies, licence and security policy | 12, 13 |
