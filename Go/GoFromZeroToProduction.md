---
title: "Go from Zero to Production"
subtitle: "A complete apprenticeship in the Go programming language — from bits and bytes to a production routing backend"
lang: en
---

# Preface

This book teaches Go the way an apprenticeship teaches a craft: one small, precise step at a time, each step checked before the next, until you can do the work of a senior engineer on your own.

It begins below the language — with what a running program physically is, how numbers live in memory, and why integers overflow silently — because almost every confusing Go behaviour you will meet later is explained by the machine. It then covers the language itself rule by rule, the standard library, testing, data structures and algorithms, concurrency, networking, databases, performance engineering, production operations, and distributed systems. It ends with two chapters in which you build, prove, and operate a real system: a road-routing service that reads OpenStreetMap data and answers fastest-route queries under load.

Three promises shape every page:

1. **Precision.** Each claim about Go is labelled with how strong it is (see *How to read the labels* below). Where the language specification guarantees something, the book says so; where a behaviour is only how today's compiler or runtime happens to work, the book says that instead.
2. **Evidence.** Code in this book was compiled and run. Test output, benchmark numbers, profiles, race-detector reports, and load-test results are real, from the machines described in the text. When a measurement showed *no* improvement, the book reports that too, because learning to accept evidence you did not want is part of the craft.
3. **Order.** Nothing is used before it is taught. Each unit lists what it builds on; each chapter ends with a project that combines its units; later chapters refer back by unit number so you can revisit exactly what you need.

# How to use this book

## Who it is for

- **Complete beginners** to programming who are willing to work carefully: start at Chapter 0 and read in order.
- **Programmers coming from other languages**: read Chapter 0 quickly (it still contains things most programmers never learned), then work through every *Deep & Tricky* entry and *Mastery check* — that is where Go differs from what you know.
- **Working Go developers** aiming for senior level: use Appendix A to test yourself on every *Deep & Tricky* entry, then focus on Chapters 6–14.

## The shape of a unit

Every unit has the same structure, so you always know where you are:

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. If you remember nothing else, remember this. |
| **Mental model** | An analogy or picture to hang the details on (where useful). |
| **Lesson** | The rules, with small runnable programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise even experienced developers, each labelled with its source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Common mistakes** | What learners and professionals actually get wrong. |
| **Mastery check** | *Explain* (say it in your own words), *Predict* (decide the output before running), *Write* (code it). Answers to the prediction questions are at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a specification, requirements, a complete reference solution with real output, and a review checklist. Try the project yourself before reading the solution. Your version will differ; compare the decisions, not the lines.

## How to study

1. **Type the code; do not paste it.** Typing forces you to read every character, and the mistakes you make while typing are exactly the ones you need to learn from.
2. **Predict before you run.** Write down what you expect a program to print. When you are wrong, find out why before moving on — that is where learning happens.
3. **Do the mastery checks.** If you cannot explain a unit's one-line summary to someone else, re-read the unit.
4. **Revise with Appendix A.** It lists every DT and P entry by title. Cover the right-hand column, read a title, and explain the entry from memory.
5. **Expect to take months, not weeks.** The book covers what professional Go developers learn over several years. A unit a day is a good pace.

## How to read the labels

*Deep & Tricky* entries carry a label in italics saying how far you can rely on them:

| Label | Meaning |
| --- | --- |
| *Guarantee* / *Guarantee: Go specification* | Required by the Go language specification or the documented API contract. True on every conforming implementation and every future Go 1.x version. |
| *Library behaviour*, *Standard-library behaviour*, *fmt behaviour* | Documented behaviour of a standard-library package. Stable, but defined by the package documentation rather than the language. |
| *Toolchain behaviour*, *Compiler behaviour* | What the `go` command or the gc compiler does. Stable in practice; occasionally changed between releases. |
| *Implementation detail*, *Runtime behaviour* | How the current runtime or compiler works internally (map layout, GC pacing, escape analysis). Useful for understanding performance; never write code that depends on it for correctness. |
| *Version-dependent* | Changed in a specific Go release; the entry names the version. |
| *Practice*, *Go convention*, *Design rule* | Community and professional practice, not a rule of the language. |
| *Hardware fact*, *OS behaviour*, *IEEE 754 behaviour*, *Algorithmic fact* | True because of the machine, operating system, floating-point standard, or mathematics — not because of Go. |

## Toolchain and versions

All code in this book was compiled and tested with **Go 1.24** on Linux (amd64). Programs that use databases were run against **PostgreSQL 16** and **Redis 7**.

Go is backwards compatible: everything here works on Go 1.24 and every later Go 1.x release. Features introduced in **Go 1.25, 1.26, and 1.27** are taught where they belong and clearly marked with their version — for example generic methods (Go 1.27), `errors.AsType` and `new(expr)` (Go 1.26), and `sync.WaitGroup.Go` (Go 1.25). Those examples follow the official release notes but could not be compiled with the Go 1.24 toolchain used to verify this book; check them against your own installation. Appendix C lists which version introduced which feature.

A few artefacts were written to production standard but **not executed** while the book was verified, and the text says so where they appear: PostGIS SQL (Unit 9.6), the Dockerfile, CI workflow, and Kubernetes manifests (Chapter 11), and gRPC code (Unit 8.8). Everything else — every test, benchmark, profile, race report, and load test — was run.

## Setting up

1. Install Go from **go.dev/dl** (any version from 1.24 on; the newest is best). Check with `go version`.
2. Install an editor with the official Go language server, **gopls** (VS Code with the Go extension, GoLand, Neovim, Zed, or Emacs all work).
3. Create a folder for your work and, inside it, one module per project: `go mod init example.com/numinfo` (Unit 3.13 explains modules properly; until then, follow the steps in each project).
4. For Chapters 9 onwards, install **PostgreSQL** and **Redis** locally (or run them in containers).
5. Keep a learning journal: the predictions you got wrong and why. It will become your most valuable revision tool.

# Roadmap

| Chapter | Title | You will be able to… |
| --- | --- | --- |
| 0 | Computing Foundations & Toolchain | explain what a running program physically is: compilation, binary numbers, two's complement, floats, memory, the stack and heap |
| 1 | Core Language Fundamentals | use Go's types, constants, operators, control flow, functions, and closures with exact knowledge of the rules |
| 2 | Composite Data & Memory | master arrays, slices, strings, maps, structs, and pointers — where most real Go bugs live |
| 3 | Types, Methods, Interfaces, Errors, Packages, Generics | organise programs with clean abstractions, handle errors properly, and write generic code |
| 4 | Testing, Tooling & Debugging | test with tables, fakes, golden files, fuzzing, and benchmarks; debug with Delve and the race detector |
| 5 | Standard Library & I/O | stream data with `io`, process text, JSON, and binary formats, and work with files, time, and the command line |
| 6 | Data Structures, Algorithms & Complexity | analyse complexity and implement the structures and graph algorithms behind fast software |
| 7 | Concurrency | write correct, leak-free concurrent programs with goroutines, channels, `context`, and `sync` |
| 8 | Networking, HTTP & RPC | build production-shaped HTTP services and clients, WebSockets, and RPC |
| 9 | Databases, Persistence & Caching | use SQL and PostgreSQL correctly — transactions, migrations, indexes — and cache safely with Redis |
| 10 | Performance Engineering & Runtime Internals | benchmark and profile rigorously, understand the scheduler and GC, and prove optimisations |
| 11 | Production Engineering | configure, secure, observe, containerise, and deploy services that can be operated |
| 12 | Distributed Systems & System Design | design systems that survive failures: consistency, queues, idempotency, sagas, and consensus |
| 13 | Geospatial Computing & Routing Algorithms | work with coordinates, spatial indexes, OpenStreetMap data, and fast routing algorithms |
| 14 | Capstone: Production Routing Backend | design, build, prove, operate, and defend a production routing service |
| A | Index of Deep & Tricky entries and Patterns | revise every entry by title |
| B | Glossary | look up any term used in the book |
| C | Go versions and features | see which Go release introduced each modern feature |

Chapters 0–5 are the **foundation**: read them in order. Chapters 6–9 can be read in order or, if you need networking sooner, as 6, 7, 8, 9. Chapters 10–12 assume everything before them. Chapters 13–14 use the whole book.

# Chapter 0 — Computing Foundations & Toolchain

*Goal: understand what a running program physically is before writing real Go.*

Most Go books start with "Hello, World" and move straight to syntax. This one starts one level lower, because almost every confusing Go behaviour you will meet later (silent overflow, `NaN`, compile errors that look like run-time errors, stack vs heap, why `go run` prints `exit status 2`) is explained by what the machine is actually doing. Six short units, one small project. Nothing here is wasted: every later chapter refers back to it.

**What you will be able to do after this chapter**

- Explain the path from `main.go` to a process running on a CPU, and say for any error whether it happens at compile time or at run time.
- Read and write numbers in binary, hex, and octal, and predict exactly what a fixed-size integer does when it overflows.
- Explain two's complement well enough to predict `int8(200)` without running the code.
- Describe registers, caches, the stack, and the heap, and why a Go programmer cares about each.

**One question to carry through the whole book:** *"Who calculates this, and when?"* The compiler (at compile time, with exact constants) or the CPU (at run time, with fixed-size values)? You will be surprised how many puzzles this one question solves.

## Topic 0A — From source code to a running process

## Unit 0.1 — From source code to running process

**One-line summary:** You write text (source code). The compiler translates it once into machine code (a binary file). When you run that file, the operating system creates a process, and the CPU executes the instructions.

**Mental model (kitchen analogy):**

- Source code = a recipe written in English. Humans can read it. The CPU cannot.
- Compiler = a translator who converts the full recipe into the CPU's own language before cooking starts.
- Binary / executable = the translated recipe, saved as a file. You can run it many times without translating again.
- Process = the actual cooking happening right now, with its own kitchen space (memory).
- CPU = the cook. It follows very small, simple instructions extremely fast.

**The pipeline:**

```
main.go (text)  --go build-->  hello (binary file)  --./hello-->  process running on CPU
  [you write]      [compile time]      [stored on disk]    [run time]
```

**Key words:**

- **Compile time** = when the compiler is checking and translating your code. The program is not running yet.
- **Run time** = when the program is actually executing as a process.
- **Compile error** = the compiler refuses to make a binary. No program exists to run.
- **Runtime error (panic in Go)** = the binary was made and started, but something went wrong while running.
- **Exit code** = a small number a process gives back to the operating system when it ends. 0 means success. Non-zero means some failure.

**Go is a compiled language.** The whole program is translated to machine code before it runs. Python (normally) is different: an interpreter program reads and executes your code line by line at run time.

**`go run` vs `go build`:**

| Command | What it does | Binary left behind? |
| --- | --- | --- |
| `go build` | Compiles and saves the binary in the current folder | Yes |
| `go run main.go` | Compiles into a temporary folder, runs it, then removes the temporary binary | No |

`go run` is NOT an interpreter. It still compiles fully. It just hides the binary from you.

**Example program:**

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go")
}
```

- `package main` + `func main()` = this package becomes an executable program, and `main` is where execution starts.
- When `main` returns normally, the process exits with code 0.

**Compile time vs run time — the division example:**

```go
fmt.Println(10 / 0)
```

Compile error: `invalid operation: division by zero`. The compiler can see both numbers, so it refuses to build. No binary, no process.

```go
x := 0
fmt.Println(10 / x)
```

This compiles. At run time it panics: `panic: runtime error: integer divide by zero`, prints a stack trace, and the process exits with code 2. (`go run` shows this as `exit status 2`.)

Lesson: the earlier an error is caught, the cheaper it is. Compile errors are caught before any user ever runs your code.

**Common beginner mistakes:**

- Thinking `go run` interprets code like Python. It compiles.
- Thinking the `.go` file itself is what the CPU runs. The CPU only runs machine code.
- Thinking "it compiled" means "it is correct". The compiler catches only some errors. Logic mistakes and many runtime failures are still possible.
- Thinking a binary built on Windows runs on Linux. Machine code is specific to an OS and CPU type (see Deep & Tricky: cross-compilation).

### Going deeper 1 — Syntax error is only ONE kind of compile error

Syntax = grammar. Think of English:

- "Dog the rice ate the." Grammar is broken. That is like a **syntax error**.
- "The rice ate the dog." Grammar is perfect, but the meaning breaks a rule. That is like a **type error or rule error**. The compiler still refuses.

The compiler works roughly in steps: first it checks grammar (parsing), then it checks meaning and rules (type checking). Failing ANY step = compile error.

| Code | Kind of compile error | Go's message starts like |
| --- | --- | --- |
| `fmt.Println("hi"` (missing `)`) | Syntax error | `syntax error: ...` |
| `name := "Ravi"; age := 30; fmt.Println(name + age)` | Type error (string + int) | `invalid operation: name + age (mismatched types string and int)` |
| `fmt.Println(100 / 0)` | Rule: constant divisor must not be zero | `invalid operation: division by zero` |
| `import "os"` never used | Rule: no unused imports | `"os" imported and not used` |
| `x := 10` never used | Rule: no unused local variables | `declared and not used: x` |

Tip: the Go compiler itself prints the words `syntax error` only for real grammar problems. Read the message; it tells you the kind.

### Going deeper 2 — Integer vs float division by zero

| Code (variables) | Result |
| --- | --- |
| `a, b := 10, 0` then `a / b` | Run-time panic: integer divide by zero |
| `a, b := 5.0, 0.0` then `a / b` | `+Inf` |
| `a, b := -5.0, 0.0` then `a / b` | `-Inf` |
| `a, b := 0.0, 0.0` then `a / b` | `NaN` ("Not a Number") |

Why the difference: every bit pattern of an int is an ordinary number, so there is no value that means "infinity". Float numbers follow the IEEE 754 standard, which reserves special values: `+Inf`, `-Inf`, and `NaN`. So a float division has a legal answer to give, and an integer division does not.

Danger: `NaN` and `Inf` spread silently. `Inf + 1` is still `Inf`. A price calculation can quietly become `NaN` and get saved to a database without any crash. A panic is loud; a `NaN` is silent.

### Going deeper 3 — A binary depends on BOTH the OS and the CPU

- CPU type decides which machine instructions are valid (for example amd64 vs arm64).
- OS decides the executable file format (Windows uses PE files, `.exe`; Linux uses ELF files) and how the program asks the OS for services like printing or opening files. These requests are called **system calls**.
- So Go has two settings: `GOOS=linux` (which OS) and `GOARCH=amd64` (which CPU). Example: `GOOS=linux GOARCH=amd64 go build` on Windows gives a binary for a normal Linux server.

### Going deeper 4 — Read the WHOLE compiler message

`invalid operation:` starts both type errors and rule errors. The words in brackets decide:

- `invalid operation: "total" + 5 (mismatched types untyped string and untyped int)` → type error
- `invalid operation: division by zero` → rule error
- Anything starting `syntax error:` → syntax error (for example an extra `}` after `main`)

### Deep & Tricky

**DT-1. `go run` still compiles, and Go caches builds.** *(Toolchain behaviour)* `go run` builds a real binary in a temporary folder, runs it, and deletes it. Compiled packages are kept in the build cache (see `go env GOCACHE`). That is why the second build of the same code is much faster than the first.

**DT-2. Your binary carries the Go runtime inside it.** *(Implementation detail)* Even "Hello, Go" includes the garbage collector, the goroutine scheduler, and memory-management code. So a Go hello-world binary is usually a few MB, while an equivalent C program can be a few KB. Exact size depends on Go version, OS, and CPU type. Benefit: you copy one file to a server and it runs, with no separate runtime to install.

**DT-3. Cross-compilation is built in.** *(Toolchain behaviour)* `GOOS=linux GOARCH=arm64 go build` on a Windows or Mac laptop produces a Linux ARM binary. `go tool dist list` shows every supported OS/CPU pair. Machine code is specific to OS + CPU, which is why this setting exists.

**DT-4. Pure Go programs are usually statically linked.** *(Toolchain behaviour)* Statically linked means all needed code is inside the one file. If a program uses cgo (calling C code), the binary may depend on system C libraries instead. Some standard-library packages can use cgo on some platforms. `CGO_ENABLED=0` forces pure Go. We will study this properly in Chapter 11 (containers).

**DT-5. Unused imports vs unused variables — two different rules.** *(Guarantee vs implementation restriction)* The Go specification itself makes an unused import illegal. For an unused local variable, the spec only *allows* compilers to reject it, and the standard Go compiler does. Unused package-level (global) variables are allowed. Most people think both are the same rule.

**DT-6. Division by zero: compile time, run time, and floats all differ.** *(Guarantee, with one implementation-specific part)*

- `10 / 0` with constants: compile error. The spec requires constant division to have a non-zero divisor.
- `10 / x` where `x` is an int variable holding 0: run-time panic. The spec requires this.
- `a / b` with float64 variables where `b` is 0: no panic in the standard Go compiler; result is `+Inf` (or `NaN` for `0/0`). The spec leaves floating-point division by zero to IEEE 754 rules, and says whether a panic happens is implementation-specific.
- An unrecovered panic makes the process exit with code 2 in the current runtime (normal success is 0).

**DT-7. `NaN` is not equal to itself.** *(Guarantee: Go follows IEEE 754 for float comparison)* `x := 0.0; y := x / x` makes `y` a `NaN`. Then `y == y` is `false`, and `y != y` is `true`. So you can never check for NaN with `==`. Use `math.IsNaN(y)`. This also causes surprises later with maps and sorting.

**DT-8. Same zero, different verdict: int variable vs float variable.** *(Compiler behaviour, consistent with the spec)*

- `x := 7; fmt.Println(x / 0)` is a **compile error**. The divisor is a constant zero and the operands are integers, so the compiler knows it must panic, and rejects it early.
- `f := 7.0; fmt.Println(f / 0)` **compiles** and prints `+Inf`. For floats, a zero divisor has a legal answer, so there is nothing to reject.
- `d := 0; fmt.Println(7 / d)` compiles and **panics** at run time. The compiler does not track the value inside a variable here.

Lesson: the compiler rejects what it can prove is wrong from constants and types. It does not run your program in its head.

**DT-9. `go run` hides the real exit code.** *(Toolchain behaviour; `go help run` documents it)* The documentation says the exit status of `go run` is not the exit status of your compiled program. When your program fails, `go run` prints a line like `exit status 2` (your program's real code) and then exits with its own status (historically 1). In scripts and CI, build the binary and run it directly if the exit code matters. `go help run` says it directly: "The exit status of Run is not the exit status of the compiled binary." Try it with a program that panics: `go run . ; echo $?` gives 1, while `go build && ./prog ; echo $?` gives 2. Notice also that the line `exit status 2` is printed by `go run`, not by your program; running the binary directly never shows it. Why: `go run` is itself a process (the go tool). It builds your program, starts it as a separate child process, waits, and prints the child's status as text (`exit status 2`). Then it exits with its OWN code, 1, meaning "the thing I ran failed".

**DT-10. Go constants are exact numbers, not IEEE floats.** *(Guarantee: Go specification)* The spec says numeric constants are exact values of arbitrary precision and never overflow. Because of this, no constant can be `+Inf`, `-Inf`, or `NaN`. So:

- `fmt.Println(4.0 / 0.0)` is a compile error. Both are constants, the compiler must calculate an exact answer, and none exists.
- `a := 4.0; fmt.Println(a / 0.0)` compiles and prints `+Inf`. Now `a` is a float64 variable, and the division happens at run time using IEEE 754 rules.

First question to ask for any expression: "Will the compiler calculate this, or will it happen at run time?" Only then ask "int or float?". Constants get a full lesson in Chapter 1.

**DT-11. `$?` remembers only the last command.** *(Shell behaviour, not Go)* Every command, including `echo`, sets `$?`. To see a program's exit code, check it immediately: `./app; echo $?`. In PowerShell the equivalent is `$LASTEXITCODE`.

**DT-12. A Go binary carries debug information too.** *(Toolchain behaviour)* Run `file` on a Go binary built on Linux and it typically says `statically linked` (confirms DT-4) and `with debug_info, not stripped`. The binary contains a symbol table and debugging data, which help stack traces and debuggers. `go build -ldflags="-s -w"` removes most of it: `-s` drops the symbol table, `-w` drops the DWARF debug data. The binary gets smaller, but debugging tools get less information. Panic stack traces still show function names, because the Go runtime keeps its own separate table for that.

### Pattern

**P-0. Fail early (engineering principle, not yet a code pattern).** Prefer errors that are caught at compile time over errors found at run time, and run-time errors caught in tests over errors found by users. Go's strict compiler (unused imports, static types) is designed around this idea.

## Topic 0B — How numbers are stored

## Unit 0.2 — Bits, bytes, and how numbers live in memory

**One-line summary:** Everything in a computer is stored as bits (0 or 1). Eight bits make a byte. Memory is a long row of numbered boxes, each holding one byte. With n bits you get exactly 2^n different patterns.

**Mental model:**

- A **bit** = one light switch: off (0) or on (1).
- A **byte** = a row of 8 switches.
- **Memory (RAM)** = a very long street of houses. Each house holds exactly one byte. Each house has a number: its **address**.
- A bigger value (like a 64-bit number) lives in several neighbouring houses (8 bytes in a row).

**Why binary?** Electronic circuits can reliably tell two states apart (voltage high or low). Telling 10 levels apart reliably would be much harder and error-prone.

**Binary place values** (right to left, each place doubles):

| Place | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 13 = | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 1 |

13 = 8 + 4 + 1, so 13 in binary is `1101` (or `00001101` as a full byte).

**The 2^n rule:** n bits give 2^n different patterns.

| Bits | Patterns | If used for 0 upwards |
| --- | --- | --- |
| 1 | 2 | 0 to 1 |
| 8 (1 byte) | 256 | 0 to 255 |
| 16 | 65,536 | 0 to 65,535 |
| 32 | about 4.29 billion | 0 to 4,294,967,295 |
| 64 | about 1.8 × 10^19 | 0 to 18,446,744,073,709,551,615 |

Largest value = 2^n − 1, because one pattern is used for zero.

**Hexadecimal (hex):** base 16, digits 0–9 then A–F (A=10 … F=15). One hex digit = exactly 4 bits, so one byte = exactly 2 hex digits. `0xFF` = `1111 1111` = 255. Hex is just a shorter way to write bits. Used for memory addresses, colours (`#FF8800`), hashes, and IDs.

**Connection to Unit 0.1:** an integer type uses ALL of its 2^n patterns for normal numbers. No pattern is left over to mean "infinity". The float format deliberately reserves some patterns for `Inf` and `NaN`.

**Go number literals** (all these are the same number, 13):

```go
a := 13      // decimal
b := 0b1101  // binary (prefix 0b)
c := 0xD     // hex (prefix 0x)
d := 0o15    // octal, base 8 (prefix 0o)
e := 1_3     // underscores are allowed between digits, for readability
```

**Printing in different bases with `fmt.Printf`:**

| Verb | Meaning | `fmt.Printf(verb, 13)` prints |
| --- | --- | --- |
| `%d` | decimal | `13` |
| `%b` | binary | `1101` |
| `%08b` | binary, padded with zeros to width 8 | `00001101` |
| `%x` / `%X` | hex, lower / upper case | `d` / `D` |
| `%o` | octal | `15` |

`Printf` does not add a newline by itself. Add `\n` at the end of the format string.

**Width with spaces vs zeros:** `%3d` means "at least 3 characters wide, fill with spaces on the left". `%03d` means the same but fill with zeros. So `fmt.Printf("%3d|%03d\n", 7, 7)` prints `  7|007` (two spaces before 7). Useful for lining up columns in tables.

**Units of size:**

- b = bit, B = byte. Internet speed is in megabits per second (Mbps); file sizes are in megabytes (MB). 100 Mbps ≈ 12.5 MB/s (divide by 8).
- KB = 1000 bytes (standard), KiB = 1024 bytes. Many tools mix these; the difference grows with size.

**Common mistakes:**

- Saying a byte holds values up to 256. It holds 256 *different* values, 0 to 255.
- Thinking `0xFF` and `255` are different numbers. Same number, different way of writing.
- Writing a number with a leading zero, like `010`, expecting ten. In Go this is octal (8). See DT-13.
- Mixing up bits and bytes (Mbps vs MB/s).

**Two ways to convert decimal → binary (both correct):**

- *Subtract place values:* 42 → 32 fits (10 left), 8 fits (2 left), 2 fits → `101010`.
- *Repeated division by 2* : divide, write the remainder, repeat; read the remainders bottom to top. Same method with 16 gives hex (200 ÷ 16 = 12 rem 8 → `C8`).

**Binary → hex shortcut:** group bits in 4s from the right, convert each group. `0010 1010` → `2` `A` → `0x2A` = 42. No counting tables needed.

**Octal = groups of 3 bits.** One octal digit = exactly 3 bits (0–7). This is why octal is still used for Unix file permissions: each permission group (owner, group, others) is 3 bits: r = 4, w = 2, x = 1.

| `ls -l` shows | Bits | Octal |
| --- | --- | --- |
| `rwx` | 111 | 7 |
| `r-x` | 101 | 5 |
| `rw-` | 110 | 6 |
| `r--` | 100 | 4 |

So `rwxr-xr-x` (the permissions of a typical program binary) = `0o755`. In Go you pass this to functions like `os.WriteFile(name, data, 0o644)`.

### Deep & Tricky

**DT-13. A leading zero makes a number octal.** *(Guarantee: Go specification)* `fmt.Println(010)` prints `8`, not `10`. Any integer literal starting with `0` (and more digits) is read as base 8. This comes from C and has caused real bugs, for example file permissions like `0755`. Since Go 1.13 you can write `0o755`, which makes the base obvious. Prefer `0o`.

**DT-14. The size of `int` is not fixed by the language.** *(Guarantee that it is 32 OR 64 bits; which one depends on the platform)* The spec says `int` and `uint` are either 32 or 64 bits. On amd64 and arm64 machines (laptops and normal servers) they are 64 bits. On some 32-bit systems they are 32 bits. When the exact size matters (file formats, network protocols), use sized types like `int32` or `int64`.

**DT-15. `byte` is just another name for `uint8`.** *(Guarantee)* `byte` and `uint8` are identical types, not two similar types. Same for `rune` and `int32`. Code can use either name interchangeably.

**DT-16. Byte order (endianness) — the `LSB` in `file` output.** *(Hardware/platform property)* A 4-byte number must be stored in 4 neighbouring memory boxes. Which end goes first? **Little-endian** stores the least significant byte first (the `LSB` in `ELF 64-bit LSB`). x86-64 and arm64 as used in Macs, PCs, and normal Linux servers are little-endian. Network protocols traditionally use **big-endian** ("network byte order"). Go's `encoding/binary` package makes you choose explicitly (`binary.LittleEndian` or `binary.BigEndian`) when turning numbers into bytes. Getting this wrong makes numbers read back as garbage. Full lesson in Chapter 5.

**DT-17. Why `{` must be on the same line: automatic semicolons.** *(Guarantee: Go specification)* You almost never type `;` in Go, but the grammar still uses them. The lexer (the first stage of the compiler, which splits text into tokens) automatically inserts a `;` at the end of a line if the line's last token is an identifier, a literal, one of `break continue fallthrough return`, or one of `++ -- ) ] }`. `func main()` ends with `)`, so it becomes `func main();`. The `{` on the next line now starts a new statement, which is illegal: `syntax error: unexpected semicolon or newline before {`. This is not a style preference like in C or Java; it is grammar. The same rule means a multi-line function call or composite literal needs a trailing comma on its last line, which you will see often.

**DT-18. `%x` vs `%X` bugs hide when there are no letters.** *(fmt behaviour)* For 34 the hex is `22`, identical in both verbs. The difference only appears with digits A–F. General lesson for testing: choose test values that can actually reveal the bug.

## Unit 0.3 — Unsigned integers and overflow

**One-line summary:** A fixed-size integer has a fixed number of patterns. When a run-time calculation goes past the end, Go does NOT panic; the value wraps around, like a car odometer. When a constant goes past the end, the compiler refuses.

**Mental model — the odometer:** A 3-digit odometer shows 000 to 999. At 999, one more km shows 000. There is no 4th digit to hold the 1. A `uint8` is an 8-bit odometer: 255 + 1 → 0, and 0 − 1 → 255.

**Unsigned integer types** (unsigned = no minus sign, only 0 and above):

| Type | Bits | Range |
| --- | --- | --- |
| `uint8` (= `byte`) | 8 | 0 to 255 |
| `uint16` | 16 | 0 to 65,535 |
| `uint32` | 32 | 0 to 4,294,967,295 |
| `uint64` | 64 | 0 to 18,446,744,073,709,551,615 |
| `uint` | 32 or 64 (platform) | 64 on all common 64-bit machines |

Go gives the limits as constants: `math.MaxUint8`, `math.MaxUint16`, `math.MaxUint32`, `math.MaxUint64`.

**The wraparound rule (guarantee):** for unsigned integers, `+`, `−`, `*` are calculated modulo 2^n. "Modulo" = keep only the remainder after dividing by 2^n. In bit terms: the true answer may need a 9th bit; that bit is simply thrown away.

| Run-time calculation (uint8) | True answer | Stored result | Why |
| --- | --- | --- | --- |
| 255 + 1 | 256 | 0 | 256 mod 256 |
| 200 + 200 | 400 | 144 | 400 − 256 |
| 0 − 1 | −1 | 255 | wraps backwards |
| 16 \* 16 | 256 | 0 | 256 mod 256 |

Binary view of 255 + 1: `1111 1111` + `1` = `1 0000 0000` (9 bits). Keep the lowest 8 bits: `0000 0000` = 0.

**Constants vs run time — "who calculates?" again:**

| Code | What happens |
| --- | --- |
| `var b uint8 = 255` then `b = b + 1` | Runs, `b` becomes 0 (wraps silently) |
| `var c uint8 = 256` | Compile error: constant 256 overflows uint8 |
| `const x uint8 = 255` then `x + 1` | Compile error: a typed constant expression is checked too |

The compiler can check constants because it knows their exact values. It does not know what a variable will hold at run time, so it cannot check `b + 1`.

**`%T` prints the type:** `fmt.Printf("%T\n", g)` prints `uint8`. When you write `g := f + f` and `f` is `uint8`, the result is also `uint8`. The result does not get bigger automatically to fit.

**Why no panic?** CPUs wrap naturally in hardware; checking every `+` would slow down every program. Go chose defined, predictable wrapping instead. The cost: overflow bugs are silent, like NaN.

**Real-world shape of the bug:** a warehouse stores `stock` as an unsigned number. `stock = 5`, a customer orders 7, code does `stock - ordered`. Instead of an error, stock becomes a huge positive number (for uint8, 254), and the system happily sells items that do not exist.

**Common mistakes:**

- Expecting a panic on overflow. Go never panics for integer overflow.
- Expecting the result type to grow (`uint8 + uint8` → still `uint8`).
- Using unsigned types just because a value "should never be negative". Subtraction can still go below zero, and then it wraps to a huge number instead of showing a clear negative value. See P-1.
- Thinking the compiler catches every overflow. It only catches constants.

### Deep & Tricky

**DT-19. Integer overflow never panics in Go, and the result is defined.** *(Guarantee)* The spec says unsigned `+ - *` are computed modulo 2^n, and overflow does not cause a run-time panic. Compare with C: unsigned wraps there too, but signed overflow in C is "undefined behaviour", meaning the compiler may assume it never happens and produce strange code. Go defines signed overflow as well (Unit 0.4). Predictable, but silent.

**DT-20. `fmt.Println(math.MaxUint64)` does not compile.** *(Guarantee, from constant rules)* `math.MaxUint64` is an untyped constant. When you pass an untyped integer constant to `Println`, Go gives it its default type, `int`. The value 18,446,744,073,709,551,615 does not fit in a 64-bit `int` (signed, max about 9.2 × 10^18), so it is a compile error saying it overflows int. Fix: give it a type first, `fmt.Println(uint64(math.MaxUint64))`. Many experienced developers are surprised by this one.

**DT-21. "Underflow" is used loosely.** *(Terminology)* People often say "underflow" for `0 − 1` wrapping to the maximum. Strictly, integer wrapping in either direction is overflow (wraparound). "Underflow" properly means a float result too close to zero to represent. Know both uses so you understand colleagues and documentation.

**DT-22. A countdown with an unsigned counter never ends.** *(Consequence of DT-19; loops come in Chapter 1)* `for i := uint(3); i >= 0; i-- { ... }` loops forever: an unsigned value is always `>= 0`, and after 0 it wraps to the maximum. Linters such as staticcheck warn that the condition is always true. One more reason for P-1.

### Patterns

**P-1. Use `int` for counts and sizes; keep unsigned types for bits and bytes.** *(Go convention)*

- *Problem it solves:* unsigned subtraction below zero silently wraps to a huge number (warehouse stock bug, endless countdown loop).
- *Why Go developers do it:* the standard library itself returns `int` from `len()` and `cap()`, even though lengths are never negative. A negative `int` is an obvious wrong value that is easy to notice and check; a wrapped unsigned value looks like a real, large number.
- *When to use unsigned:* raw bytes (`byte`), bit manipulation, hashes, binary file and network formats where the size is fixed by a specification.
- *Trade-off:* you lose the positive half of the range compared with the same-size unsigned type. With 64 bits this almost never matters.

**P-2. Check before you calculate, when overflow matters.** *(Defensive pattern; full code after `if` in Chapter 1)* Go will not warn you at run time. When values come from users or outside systems, check that the operation fits BEFORE doing it (for example, "is `a` greater than `math.MaxUint8 - b`?"). The `math/bits` package also has functions like `bits.Add64` that return a carry flag telling you whether overflow happened.

## Unit 0.4 — Signed integers and two's complement

**One-line summary:** Signed integers store negative numbers using "two's complement": the top bit has a NEGATIVE place value. The same bits mean different numbers depending on the type. Signed overflow also wraps silently in Go.

**The mental model — the top place value is negative.** For `int8`:

| Place | **−128** | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `0111 1111` | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| `1000 0000` | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `1111 1111` | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |

- `0111 1111` = 64+32+16+8+4+2+1 = **127** (largest int8)
- `1000 0000` = −128 (smallest int8)
- `1111 1111` = −128 + 127 = **−1**

If the top bit is 1, the number is negative. That is why the top bit is often called the "sign bit", but it is not just a sign: it has the weight −128.

**Range:** n bits → −2^(n−1) to 2^(n−1) − 1. The 256 patterns of 8 bits = 128 negatives + zero + 127 positives. One extra negative, because zero uses one of the "non-negative" patterns.

| Type | Range |
| --- | --- |
| `int8` | −128 to 127 |
| `int16` | −32,768 to 32,767 |
| `int32` (= `rune`) | about −2.1 billion to 2.1 billion (2,147,483,647) |
| `int64` | about ±9.2 × 10^18 (9,223,372,036,854,775,807) |
| `int` | 64 bits on all common 64-bit machines |

Constants: `math.MaxInt8`, `math.MinInt8`, … `math.MaxInt64`, `math.MinInt64`.

**Making a negative number: flip all bits, then add 1.** 5 = `0000 0101` → flip → `1111 1010` → add 1 → `1111 1011` = −5. Check: −128+64+32+16+8+2+1 = −5. The same steps turn −5 back into 5.

**Why this design won (and the simple "sign bit + size" idea lost):**

- A simple sign bit gives two zeros (`+0` = `0000 0000` and `−0` = `1000 0000`). Wasteful and confusing.
- With two's complement, ONE addition circuit works for both signed and unsigned numbers. `1111 1111 + 0000 0001 = 0000 0000`: as uint8 that is 255 + 1 = 0 (wrap); as int8 it is −1 + 1 = 0 (correct!). Same bits, same circuit, different meaning.

**Bits have no meaning without a type.** `1111 1011` is 251 as `uint8` and −5 as `int8`. The type tells Go how to read the bits.

**Signed overflow wraps too (guarantee):**

| Run-time (int8) | True answer | Stored |
| --- | --- | --- |
| 127 + 1 | 128 | −128 |
| −128 − 1 | −129 | 127 |
| 100 + 100 | 200 | −56 |

**Converting between integer types: `T(x)`.** `uint8(x)` means "give me x as a uint8". Between same-size integer types the bits stay the same; only the meaning changes:

```go
var x int8 = -5
fmt.Printf("%08b %d\n", uint8(x), uint8(x)) // 11111011 251
```

Careful: `uint8(-5)` with a constant is a compile error (the compiler knows −5 cannot be a uint8). With a variable it reinterprets the bits.

**`%b` on a negative number does NOT show the stored bits.** `fmt.Printf("%b", int8(-5))` prints `-101` (a minus sign + the size). To see the real two's-complement bits, convert to the unsigned type of the same size first.

**Real-world signed overflow:**

- **Year 2038 problem:** many systems store time as a signed 32-bit count of seconds since 1 Jan 1970. The max, 2,147,483,647 seconds, is reached on 19 January 2038; the next second wraps to a negative number (a date in 1901). Go's `time` package uses 64-bit values, but databases, file formats, and older C code may not.
- **YouTube, 2014:** the view counter of a very popular video passed 2,147,483,647 (max int32), and YouTube moved the counter to 64 bits.
- **Boeing 787, 2015:** a US aviation safety directive required power cycling because generator control units could fail after 248 days of continuous power. This matches a signed 32-bit counter of hundredths of a second: 2,147,483,647 / 100 seconds ≈ 248.5 days.

**Common mistakes:**

- Thinking the top bit is "just a minus sign" (`1000 0101` is not −5).
- Expecting `int8` to go up to 128, or expecting a symmetric range.
- Expecting `%b` to show two's-complement bits for negative numbers.
- Thinking `-x` is always positive when `x` is negative (see DT-24).

### Deep & Tricky

**DT-23. Same bits, different number — the type decides.** *(Fundamental)* Memory stores only bits. `1111 1111` is 255 as `uint8` and −1 as `int8`. The CPU's addition gives identical bits for both. This is why conversions like `uint8(x)` between same-size integer types cost nothing at run time: no bits change.

**DT-24. Negating the minimum value gives itself.** *(Guarantee, consequence of two's complement)* `var x int8 = -128; x = -x` leaves `x` = −128, because +128 does not exist in int8 (flip `1000 0000` → `0111 1111`, add 1 → `1000 0000`). So an "absolute value" function can return a NEGATIVE number for `math.MinInt64`. A classic bug in code that does `if x < 0 { x = -x }` and then assumes `x >= 0`.

**DT-25. `math.MinInt64 / -1` does not crash in Go.** *(Guarantee: Go specification)* The true answer (+9.2 × 10^18 + 1) does not fit. The Go spec says this quotient equals the dividend (MinInt64) and the remainder is 0. On x86 CPUs this division normally triggers a hardware fault (in C it is undefined behaviour and often crashes); Go's compiler handles this case so the result is defined.

**DT-26. The compiler may NOT assume `x < x + 1`.** *(Guarantee: Go specification)* Because Go defines signed overflow as wrapping, the spec explicitly says a compiler may not optimise code assuming overflow never happens, and gives `x < x + 1` as the example: it is false when `x` is the max value. C compilers may assume the opposite, which has produced famous security bugs where overflow checks were optimised away.

**DT-27. `uint8(-5)` vs `uint8(x)` — "who calculates?" once more.** *(Guarantee)* Constant conversion: compile error, the value −5 cannot be represented as uint8. Variable conversion: allowed, keeps the bits, gives 251. Same text, different rules, decided by whether the compiler or the run time does the work.

**DT-28. Who really drops the 9th bit? The compiler plans it, the CPU does it.** *(Implementation detail; varies by CPU and Go version)* Modern 64-bit CPUs (amd64, arm64) usually do small-integer arithmetic in full 64-bit registers. arm64 has no 8-bit arithmetic registers at all. So for `b = b + 1` with a `uint8`, the compiler typically generates a normal wide addition, plus an instruction that keeps only the lowest 8 bits (a "zero-extend"), or it stores just one byte to memory. Then, at run time, the CPU executes those instructions. So the correct answer to "who discards the bit, and when?" is: the CPU, at run time, by executing instructions the compiler chose in advance. The compiler cannot discard anything itself, because it never knows the run-time value. (A plain arm64 ADD does not even set a carry flag.) You will be able to see the generated instructions yourself in Chapter 10 with `go build -gcflags=-S`.

**DT-29. Same size, different type: `int` vs `int64`.** *(Guarantee)* Even where `int` is 64 bits, `int` and `int64` are distinct types. `a + b` with `a int` and `b int64` is a type error: `mismatched types int and int64`. Go never converts between numeric types automatically; you must write `int64(a) + b`. This explicitness prevents silent overflow and precision bugs when moving values between types.

## Topic 0C — The machine and its memory

## Unit 0.5 — How the CPU runs your program

**One-line summary:** The CPU repeats one simple loop billions of times per second: fetch the next instruction from memory, decode it, execute it, move on. It works on tiny, very fast storage slots called registers, and waits a long time whenever it needs data from RAM. Caches exist to hide that wait.

**The fetch → decode → execute loop:**

1. **Fetch:** read the next instruction from memory. A special register, the **program counter (PC)**, holds the address of that instruction.
2. **Decode:** work out what the instruction means ("add these two registers").
3. **Execute:** do it.
4. Move the PC to the next instruction, or somewhere else if the instruction was a **jump**.

`if`, loops, and function calls are all built from jumps: "if this comparison is true, set the PC to that address".

**Registers:** a few very fast storage slots inside the CPU. arm64 (Apple Silicon, many cloud servers) has 31 general-purpose 64-bit registers; amd64 has 16. The CPU can only do arithmetic on values in registers, so data must be loaded from memory into registers first, and stored back afterwards.

**What `c := a + b` roughly becomes** (simplified, not real assembly):

```
LOAD  R1, [address of a]    // copy a from memory into register R1
LOAD  R2, [address of b]
ADD   R3, R1, R2            // R3 = R1 + R2
STORE R3, [address of c]    // copy result back to memory
```

The compiler is clever: it often keeps values in registers and skips loads and stores, and if `a` and `b` are constants it just writes the answer (see DT-30).

**The memory hierarchy (kitchen analogy, rough orders of magnitude):**

| Level | Kitchen analogy | Rough time to get data | Typical size |
| --- | --- | --- | --- |
| Registers | In your hand | under 1 ns | a few hundred bytes total |
| L1 cache | On the kitchen counter | about 1 ns | tens to a few hundred KB |
| L2 / L3 cache | In the fridge / store room | about 3–40 ns | MBs |
| RAM | Kirana shop down the road | about 100 ns | GBs |
| SSD | Wholesale market across the city | tens to hundreds of µs | hundreds of GB |

(1 ns = one billionth of a second; 1 µs = 1000 ns.) Exact numbers vary by machine; the important point is the RATIO: RAM is roughly 100 times slower than L1, and an SSD is roughly 1000 times slower again.

**Caches and cache lines:** the CPU keeps recently used data in caches automatically. Data moves from RAM to cache in fixed-size chunks called **cache lines** (64 bytes on most x86 machines; check yours with `sysctl hw.cachelinesize`). When you read one byte, its neighbours come along for free.

**Why a Go programmer cares:**

- Reading memory **in order** (like walking through an array or slice) is fast: each cache line brings the next values with it. This is called **spatial locality**.
- Jumping around memory randomly (like following many pointers in a linked list) is slow: almost every step can be a trip to the kirana shop.
- The same Big-O algorithm can be several times faster or slower depending on memory layout. We will measure this ourselves in Chapter 6 and Chapter 10.

**Common mistakes:**

- Thinking the CPU works directly on RAM. It works on registers; RAM is far away.
- Thinking "fewer instructions" always means faster. Waiting for memory often costs far more than the arithmetic.
- Thinking the CPU knows about Go types. It only sees instructions and bits; the compiler chose the instructions based on the types (DT-31).

### Deep & Tricky

**DT-30. Constant folding: `c := 2 + 3` has no ADD at run time.** *(Constant expressions: guarantee; folding of other known values: optimisation)* The spec says constant expressions are evaluated exactly at compile time, so the binary simply contains 5. Optimising compilers also fold some non-constant cases when they can prove the values. This is "who calculates?" seen from the machine-code side.

**DT-31. Same ADD, different compare and divide.** *(Hardware fact)* Two's complement lets signed and unsigned addition share one instruction (Unit 0.4). But comparison and division do NOT share: `1000 0000 < 0000 0001` is false as uint8 (128 < 1) and true as int8 (−128 < 1). So CPUs have separate signed and unsigned versions (for example arm64 has SDIV and UDIV, and different condition codes for signed and unsigned comparisons). The CPU never sees your types; the compiler reads your types and picks the right instruction.

**DT-32. Apple Silicon uses bigger cache lines.** *(Hardware detail; check your machine)* Most x86 CPUs use 64-byte cache lines. Apple's M-series chips report 128 bytes. This will matter in Chapter 7 for "false sharing", a concurrency performance problem where two goroutines slow each other down by writing to different variables that happen to sit in the same cache line.

**DT-33. Cache lines are aligned blocks.** *(Hardware detail)* Memory is divided into fixed blocks of the cache-line size (128 bytes on Apple Silicon): addresses 0–127, 128–255, and so on. Reading any byte loads the whole block it belongs to, not "the next 128 bytes from here". A value that crosses a block boundary needs two cache lines. This is one reason the compiler aligns values in memory (struct layout and padding come in Chapter 3 and Chapter 10).

## Unit 0.6 — Stack and heap (first look)

**One-line summary:** A running process divides its memory into areas. The **stack** holds short-lived data for function calls and is extremely cheap. The **heap** holds data that must live longer or whose size is not known in advance; it costs more and must be cleaned up (in Go, by the garbage collector). In Go, the compiler decides which one each value uses.

**Process memory layout (simplified):**

| Area | What lives there | Lifetime |
| --- | --- | --- |
| Code (text) | Machine-code instructions of your program | Whole run, read-only |
| Static data | Package-level (global) variables, constant strings | Whole run |
| Stack | Local variables and bookkeeping of each active function call | Until that function returns |
| Heap | Data that must outlive the function that made it, or is too big / of unknown size | Until nothing uses it any more (then GC frees it) |

**The stack — a pile of plates (tiffin carrier):**

- Each function call gets a **stack frame**: a block holding its local variables, its arguments, and the address to jump back to when it returns.
- Calling a function puts a new frame on top. Returning removes the top frame. Last in, first out (LIFO).
- Allocating a frame = moving one register (the **stack pointer**) by a fixed amount. One instruction. Freeing = moving it back. No search, no cleanup work.
- The compiler must know the frame size ahead of time.
- When a function returns, its frame is gone. Nothing in it may be used afterwards.

**The heap — a rented godown (warehouse):**

- For data that must survive after its function returns, or that is very large, or whose size is decided at run time.
- Allocating means finding free space (more work than moving one register).
- Someone must free it. In C the programmer calls `free` (forgetting = memory leak; freeing too early = crash or security bug). In Go the **garbage collector (GC)** finds data nothing refers to any more and frees it automatically. Safe, but the GC itself uses CPU time.

**Who decides in Go? The compiler, using escape analysis.** There is no keyword to choose stack or heap. The compiler studies each value: if it can prove the value is not used after the function returns, it can stay on the stack. If the value "escapes" the function, it goes to the heap. You can see these decisions with `go build -gcflags=-m` (full lesson in Chapter 10).

**Why a Go programmer cares (later stages):**

- Heap allocations cost time to create AND create work for the GC. Reducing unnecessary allocations is one of the main Go performance techniques (Chapter 10).
- Correctness never depends on stack vs heap in Go. Only performance does (DT-34).

**Each goroutine has its own stack.** Goroutines (Chapter 7) are Go's lightweight threads. Their stacks start small and grow when needed (DT-36). The heap is shared by all goroutines.

**Common mistakes:**

- "Small values go on the stack, big values on the heap." Size matters sometimes, but the main question is lifetime: does the value escape?
- "I can choose stack or heap in Go." You cannot directly; the compiler chooses.
- "Heap is bad." The heap is necessary; unnecessary heap allocations are what we avoid.
- Thinking the stack trace is unrelated to the stack. It is literally a printout of the stack frames at the moment of failure.

### A closer look at the stack and the heap

**"Last in, first out" (LIFO) — "in" and "out" are about TIME, not position.** Put three tiffin boxes on a pile: A first, then B, then C. C went in last. You can only take from the top, so C comes out first. The last one IN is the first one OUT. You can also say "first in, last out" (A went in first and comes out last); both describe the same thing. Compare with a queue at a railway ticket counter: first person in line is served first: first in, first out (FIFO).

For function calls: `main` is called first, then `sumOfSquares`, then `square`. `square` (last called) finishes first; `main` (first called) finishes last.

**What "area" means: regions of the process's own address space.** Each process gets its own private range of addresses, called its **virtual address space**. Think: every process thinks it owns a whole street numbered from 0 upwards. The OS and CPU keep a map (the **page table**) from each process's house numbers to real houses in physical RAM. Memory is mapped in fixed blocks called **pages** (16 KB on Apple Silicon Macs, 4 KB on most x86 Linux machines).

Consequences:

- Two processes can both use address `0x1000`; each maps to a different real RAM location. They cannot see or overwrite each other's memory. This is a basic security protection.
- Pages are only given real RAM when actually used.
- The "areas" (code, static data, stack, heap) are just ranges of addresses inside one process's address space. When used, they are backed by real RAM. So yes, they end up in RAM, but organised per process through this map.

**Classic layout (C programs), from low addresses to high:**

```
high addresses
+------------------+
|      stack       |  grows DOWN (towards lower addresses) as calls are made
|        |         |
|        v         |
|                  |  (unused gap)
|        ^         |
|        |         |
|       heap       |  grows UP as more is allocated
+------------------+
|   static data    |  globals
+------------------+
|       code       |  machine instructions (read-only)
+------------------+
low addresses
```

Go twist: goroutine stacks are not placed in that classic stack area. The Go runtime allocates them from memory it manages itself (see DT-37).

**Step-by-step stack snapshots for the lesson program:**

| Step | What happens | Stack (top first) |
| --- | --- | --- |
| 1 | `main` starts | main |
| 2 | main calls `sumOfSquares(3, 4)` | sumOfSquares(a=3,b=4), main |
| 3 | it calls `square(3)` | square(n=3), sumOfSquares, main |
| 4 | `square(3)` returns 9 | sumOfSquares, main |
| 5 | it calls `square(4)` — new frame in the SAME place | square(n=4), sumOfSquares, main |
| 6 | `square(4)` returns 16 | sumOfSquares, main |
| 7 | `sumOfSquares` returns 25 | main (total=25) |
| 8 | `main` returns | empty: process exits with code 0 |

What a frame holds: the function's local variables, its arguments, and the **return address** (where the program counter must jump back to when this function finishes). Removing a frame does NOT erase it; the stack pointer just moves back, and the old bytes stay until the next call overwrites them. That is why reading a frame after its function returned is dangerous (a classic C bug), and why Go never allows it to happen.

**"Data dies when the function returns" — but `square` returns 9 to its caller. How?** Two different things are involved:

- The **variables** of `square` (`n` and `result`) live in its frame. They die when the frame is removed.
- The **value** being returned (9) is **copied out before the frame is removed**: the return instruction puts a copy of 9 where the caller can pick it up (in current Go on amd64 and arm64, a CPU register — see DT-38). Then the frame is removed. The caller (`sumOfSquares`) now holds its own copy.

So `result` (the box) is gone, but 9 (the contents) survived as a photocopy. Like a shopkeeper who closes his shop after handing you your parcel: the shop closes, your parcel is in your hand. What CANNOT survive is anything that still points INTO the closed shop; that is the case escape analysis handles.

**Heap and the garbage collector, in more detail:**

- Some data must outlive its function, for example a function that reads a file of unknown size and returns all its lines to the caller. The frame is thrown away on return, so the lines must live somewhere else: the heap.
- The GC decides what is still in use by **reachability**: it starts from the "roots" (variables in every goroutine's stack frames, and global variables) and follows every reference it can find, marking what it reaches. Anything never reached cannot be used by the program again, so its memory is freed ("sweep"). This is called **mark and sweep**.
- Go's GC mostly runs at the same time as your program (concurrently), using some CPU time; it can briefly pause goroutines. Details in Chapter 10.

**Escape analysis — the photocopy analogy:**

- Returning a plain value (like the number 9 from `square`) = handing the caller a **photocopy**. The copy goes into the caller's frame; the original can be thrown away with the frame. Nothing escapes.
- Returning a **reference** to data (its address, Chapter 2) = handing the caller a **location slip**. If the data were in the frame that is about to be thrown away, the slip would point to nothing. So the compiler places that data in the heap from the start. The data "escapes".

**Stack vs heap summary:**

|  | Stack | Heap |
| --- | --- | --- |
| Allocation cost | Move one register: tiny | Find free space: more work |
| Freeing | Automatic on function return, free of cost | Garbage collector (costs CPU time) |
| Lifetime | Until the function returns | Until nothing references it |
| Size known at | Compile time | Can be decided at run time |
| Shared between goroutines? | No, each goroutine has its own | Yes, one shared heap |
| Who chooses (in Go)? | Compiler (escape analysis) | Compiler (escape analysis) |

### Deep & Tricky

**DT-34. The Go specification never mentions "stack" or "heap".** *(Guarantee, by absence)* The language only defines how long a variable can be used, not where it lives. Stack vs heap is entirely the compiler's choice, can differ between Go versions, and never changes what your program computes, only how fast. Example of the consequence: in Go it is completely safe for a function to return the address of its own local variable (pointers come in Chapter 2). The compiler simply moves that variable to the heap. In C, the same code is a famous bug.

**DT-35. Goroutine stacks grow by copying.** *(Implementation detail; changes between versions)* An OS thread usually gets a large fixed stack (often several MB). A goroutine starts with a small stack (a few KB) and, when it needs more, the runtime allocates a bigger stack and copies the old one into it. This is why a Go program can run hundreds of thousands of goroutines. Since Go 1.19, the starting size also adapts to the average stack usage the runtime has observed.

**DT-36. Apple Silicon pages are 16 KB.** *(Platform detail)* The OS maps memory in pages. Most x86 Linux systems use 4 KB pages; Apple Silicon macOS uses 16 KB. Check with the `pagesize` command. Page size affects how much memory small programs appear to use and matters for memory-mapped files (Chapter 5).

**DT-37. Goroutine stacks live in runtime-managed memory.** *(Implementation detail)* The classic "stack at the top of the address space" picture describes C programs and OS threads. Goroutine stacks are allocated by the Go runtime from memory it manages, which is what makes growing them by copying possible (DT-35). The garbage collector still treats every goroutine stack as a set of roots for reachability.

**DT-38. Go passes arguments and return values in registers.** *(Implementation detail; since Go 1.17 on amd64 and Go 1.18 on arm64)* Older Go versions passed every argument and return value through the stack, which meant extra memory loads and stores. The "register-based calling convention" (ABI) now uses CPU registers for most of them, which made typical programs a few percent faster. So `square`'s 9 travels back to the caller in a register, then the caller decides where to keep it.

**DT-39. Stack overflow is a fatal error, not a panic.** *(Runtime behaviour)* Infinite recursion keeps adding frames until the goroutine's stack hits the runtime's maximum (1 GB on 64-bit systems, shown as `goroutine stack exceeds 1000000000-byte limit`). The runtime then reports `fatal error: stack overflow` and the process exits with code 2. Unlike a panic (such as divide by zero), a fatal error cannot be caught and recovered from with `recover` (Chapter 3). The program always dies.

**DT-40. Inlined functions show `(...)` in stack traces.** *(Compiler behaviour; visible in any stack trace)* Inlining = the compiler copies a small function's body into its caller instead of doing a real call. No real call means no real frame. But the compiler keeps a record of the inlining, so the stack trace still shows every function, in the correct order. Look for the evidence in a trace: `main.level3(...)` with `(...)` instead of argument values, and NO `+0x..` offset, while `main.main()` (a real frame) has `+0x28`. Inlining removes call overhead and often lets the compiler optimise further. Chapter 10 covers it with `-gcflags=-m`.

**DT-41. Converting to a smaller integer type keeps only the lowest bits.** *(Guarantee)* `uint8(n)` for an `int` `n` keeps the lowest 8 bits and throws the rest away. 300 = `1 0010 1100` → lowest 8 bits `0010 1100` = 44. With `int8(n)`, the same lowest 8 bits are read as two's complement. No panic, no warning; it is silent like all integer overflow. (With a constant, `uint8(300)` is a compile error, "who calculates?".)

**DT-42. Even a local that never escapes may go to the heap.** *(Implementation detail)* The Go compiler puts very large local variables on the heap even if they never escape, because goroutine stacks are meant to be small (the current limit for explicitly declared variables is around 10 MB). Another proof that the spec makes no stack promise (DT-34).

**DT-43. Every variable starts at its zero value — a spec guarantee, implemented cleverly.** *(Guarantee + implementation detail)* The spec says a variable declared without a value starts at its type's **zero value**: `0` for numbers, `false` for bool, `""` for strings (more in Chapter 1). So a Go program can never read leftover garbage from an old frame, unlike C. How the compiler achieves it is an implementation detail: it does NOT always wipe whole frames. It writes zeros only where needed, and skips it when the variable is assigned before any use. Memory that will hold pointers must be zeroed so the garbage collector never sees fake addresses. Why it matters: no random values, no leaking old data (a real security issue in C), and programs behave the same every run.

**DT-44. Hex formatting of negative numbers.** *(fmt behaviour)* `%X` of `-1` prints `-1`, and `%x` of `-129` prints `-81`: fmt prints a minus sign plus the size, like `%b`. `%#x` adds a `0x` prefix (`-0x81` for negatives); `%#X` gives an uppercase `0X` prefix. To show the real two's-complement bytes in hex, convert to an unsigned type first, for example `uint8(n)` or `uint64(n)`.

## Chapter 0 Project — `numinfo`

**Goal:** a command-line tool that takes one integer and prints everything this chapter taught about it.

**New tools you need (full lessons come later):**

- `os.Args` — the command-line words. `os.Args[0]` is the program's name; `os.Args[1]` is the first argument; `len(os.Args)` counts them.
- `strconv.ParseInt(text, 10, 64)` — turns text like `"200"` into an `int64`, and returns an error if the text is not a valid 64-bit integer. (`strconv.Atoi` is the shortcut for `int`.)
- `if cond { ... }` and `switch` — run code only when a condition holds (Unit 1.6).
- `fmt.Fprintln(os.Stderr, ...)` prints to stderr; `os.Exit(1)` stops with exit code 1.
- `%-10s` prints a string left-aligned in a 10-character field, so labels line up.

**Requirements**

1. Exactly one argument; otherwise print a usage line to stderr and exit with code 1.
2. A non-integer (or out-of-range) argument: a clear message on stderr naming the bad input, exit code 1.
3. Print, one per line with aligned labels: sign (positive/negative/zero), decimal, binary, octal (`0o` prefix), hex (`0x` prefix, uppercase digits; negatives as `-0x…`).
4. Print the lowest 8 bits as a padded 8-digit binary, and read as `uint8` and as `int8`.
5. Print every type from `int8 uint8 int16 uint16 int32 uint32 int64 uint64` that can hold the number.
6. Must work for `math.MinInt64` (the hex line is the trap: DT-24, DT-70).
7. Formatted with `gofmt`, passes `go vet`.

**Test plan** — run each and record output AND exit code from a *real run*:

| Input | Must show |
| --- | --- |
| `200` | hex `0xC8`, as int8 `-56`, fits `uint8` but not `int8` |
| `-1` | hex `-0x1`, low 8 bits `11111111`, as uint8 `255` |
| `0` | sign zero, fits everything |
| `300` | low 8 bits `00101100`, both readings 44 |
| `-9223372036854775808` | hex `-0x8000000000000000` (no double minus) |
| `abc`, no argument, two arguments | stderr message, exit 1 |

**Reference solution**

```go
// Command numinfo prints facts about one integer: its bases, its lowest
// 8 bits read as uint8 and int8, and which integer types can hold it.
package main

import (
	"fmt"
	"math"
	"os"
	"strconv"
)

const usage = "usage: numinfo <integer>"

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintln(os.Stderr, usage)
		os.Exit(1)
	}
	n, err := strconv.ParseInt(os.Args[1], 10, 64)
	if err != nil {
		fmt.Fprintf(os.Stderr, "numinfo: %q is not a 64-bit integer\n", os.Args[1])
		os.Exit(1)
	}

	switch {
	case n > 0:
		fmt.Printf("%-10s %s\n", "sign:", "positive")
	case n < 0:
		fmt.Printf("%-10s %s\n", "sign:", "negative")
	default:
		fmt.Printf("%-10s %s\n", "sign:", "zero")
	}

	fmt.Printf("%-10s %d\n", "decimal:", n)
	fmt.Printf("%-10s %b\n", "binary:", n)
	fmt.Printf("%-10s %O\n", "octal:", n)
	if n < 0 {
		// uint64(-n) is the correct size even for MinInt64 (DT-70).
		fmt.Printf("%-10s -0x%X\n", "hex:", uint64(-n))
	} else {
		fmt.Printf("%-10s 0x%X\n", "hex:", n)
	}

	low := uint8(n) // keeps only the lowest 8 bits (DT-41)
	fmt.Printf("%-10s %08b\n", "low 8:", low)
	fmt.Printf("%-10s %d\n", "as uint8:", low)
	fmt.Printf("%-10s %d\n", "as int8:", int8(n))

	fmt.Printf("%-10s", "fits:")
	if n >= math.MinInt8 && n <= math.MaxInt8 {
		fmt.Print(" int8")
	}
	if n >= 0 && n <= math.MaxUint8 {
		fmt.Print(" uint8")
	}
	if n >= math.MinInt16 && n <= math.MaxInt16 {
		fmt.Print(" int16")
	}
	if n >= 0 && n <= math.MaxUint16 {
		fmt.Print(" uint16")
	}
	if n >= math.MinInt32 && n <= math.MaxInt32 {
		fmt.Print(" int32")
	}
	if n >= 0 && n <= math.MaxUint32 {
		fmt.Print(" uint32")
	}
	fmt.Print(" int64")
	if n >= 0 {
		fmt.Print(" uint64")
	}
	fmt.Println()
}
```

Real output for `200`:

```
sign:      positive
decimal:   200
binary:    11001000
octal:     0o310
hex:       0xC8
low 8:     11001000
as uint8:  200
as int8:   -56
fits:      uint8 int16 uint16 int32 uint32 int64 uint64
```

**Review checklist** (the issues reviewers find most often in this project)

- The `0x` prefix is missing, or `%#x` was used (lowercase digits), or `%#X` (uppercase `0X` prefix). `"0x%X"` is what the requirement asks for.
- Negative hex printed with `%X` directly gives `-1` style output; for `MinInt64`, printing `-0x%X` of `-n` gives a *double* minus because `-n` is still negative. Use `uint64(-n)`.
- `n&0xFF` and `uint8(n)` both give the low 8 bits; choose one and make the code match the choice.
- Labels aligned with `\t` depend on the reader's tab width; `%-10s` gives the same layout everywhere.
- `fmt.Scan` reads stdin, not arguments. An "upgrade" must keep everything that already works (arguments, validation, exit codes).
- Remove scaffold comments ("TODO: step 3"); comments should explain the code that exists.
- Every requirement line checked against the final output before submitting.

### Patterns introduced by this project

**P-3. CLI convention: results to stdout, errors to stderr, non-zero exit code on failure.** *(Unix and Go convention)*

- *Problem it solves:* scripts and other programs need to tell success from failure, and separate real output from error messages.
- *How:* every process has two output streams: **stdout** (normal output, `fmt.Println`) and **stderr** (errors, `fmt.Fprintln(os.Stderr, ...)`). A shell can redirect them separately: `./numinfo 42 > out.txt` saves only stdout, and errors still appear on screen. On failure, exit with a non-zero code: `os.Exit(1)`.
- *Why Go developers do it:* the `go` tool and almost every Go CLI behave this way; CI systems and shell scripts depend on it.
- *When not to:* inside library code, never call `os.Exit`. Libraries return errors (Chapter 3) and let `main` decide. `os.Exit` also skips deferred cleanup (Chapter 1).

**P-4. Validate input at the edge.** *(Engineering practice)* Check everything that comes from outside (command-line arguments, files, network) as soon as it enters the program, and stop with a clear message if it is wrong. The rest of the program can then trust its data. In numinfo: wrong number of arguments and non-numeric input are rejected in the first few lines of `main`.

**P-5. Always format with `gofmt`.** *(Go convention, universal)*

- *Problem it solves:* teams waste time arguing about spaces, braces, and alignment, and diffs fill with formatting noise.
- *How:* `gofmt -d file.go` shows what it would change; `gofmt -w file.go` (or `go fmt ./...`) rewrites the files. Most editors run it on save.
- *Why Go developers do it:* Go has exactly one official format, and almost all public Go code uses it. Code that is not gofmt'd looks unprofessional in review.
- *When not to:* never skip it. It only changes layout, never behaviour.

## Answers to the Chapter 0 prediction questions

**0.1 Q2.** (a) Compile error, rule: `invalid operation: division by zero`. (b) Panic: `integer divide by zero`, exit code 2. (c) `+Inf`. (d) Compile error, type: mismatched types untyped string and untyped int. (e) `false` — NaN equals nothing.

**0.2 Q2.** (a) `00101010` (b) `FF` (c) `8` — leading zero means octal (d) `  42|42  |` (e) `30` (10 + 10 + 10).

**0.3 Q2.** (a) 4 (260 − 256) (b) 244 (500 − 256) (c) 255 (d) compile error: constant 256 overflows uint8 (e) `uint8`.

**0.4 Q2.** (a) −56 (b) −128 (c) −128 (DT-24) (d) `-101` (e) `11111011`.

**0.4 Q3.** 37 = `0010 0101` → flip `1101 1010` → add 1 → `1101 1011`. Check: −128 + 64 + 16 + 8 + 2 + 1 = −37.

# Chapter 1 — Core Language Fundamentals

*Goal: know the exact rules of Go's building blocks.*

This chapter covers the core language: variables, constants, operators, floating point, control flow, functions, scope, closures, `defer`, reading input, and random numbers. It is longer than Chapter 0 because precision matters here. Professionals are rarely bitten by *what* `for` does; they are bitten by the edge cases: integer division of negative numbers, `0.1 + 0.2`, shadowed `err`, a deferred argument evaluated too early. Every edge case in this chapter has caused real bugs in real systems.

**What you will be able to do after this chapter**

- Predict the type and value of any expression built from constants, variables, and operators.
- Write idiomatic control flow with guard clauses, and loops without off-by-one errors.
- Design functions with `(value, error)` and `(value, ok)` results; use closures and `defer` correctly.
- Build a complete, testable command-line program that reads input safely.

## Topic 1A — Values and types

## Unit 1.1 — Variables and zero values

**One-line summary:** A variable is a named box with a fixed type. The type is decided when the variable is declared and can never change. A variable declared without a value holds its type's zero value.

**Mental model:** a labelled steel tiffin box. The label (name) and the shape (type) are fixed when it is made. You can change what is inside (assign a new value of the same type), but you can never turn an `int` box into a `string` box. This is **static typing**: types are checked by the compiler, before the program runs.

**Declaration forms:**

| Form | Example | Type comes from | Where allowed |
| --- | --- | --- | --- |
| var with type, no value | `var count int` | written type; value = zero value | anywhere |
| var with type and value | `var ratio float32 = 1.5` | written type | anywhere |
| var with value only | `var name = "Ravi"` | the value (default type) | anywhere |
| short declaration | `name := "Ravi"` | the value (default type) | inside functions only |
| grouped | `var ( a int; b string )` | as above | anywhere |
| multiple | `x, y := 1, "hi"` | each value | inside functions only |

**Zero values (the starting value when none is given):**

| Type | Zero value |
| --- | --- |
| all integer types | `0` |
| float32, float64 | `0` (prints as `0`) |
| bool | `false` |
| string | `""` (empty string) |
| pointers, slices, maps, channels, functions, interfaces | `nil` (later stages) |

**Default types when the type is inferred from a constant:**

| Value written | Type of `x := value` |
| --- | --- |
| `5` | `int` |
| `5.0` or `1e3` | `float64` |
| `'a'` | `rune` (= `int32`) |
| `"a"` | `string` |
| `true` | `bool` |

**`=` vs `:=`:** `:=` declares (creates) AND assigns. `=` only assigns to a variable that already exists.

**Rules of `:=`:**

- Only inside functions. At package level you must use `var`.
- At least ONE variable on the left must be new in the current scope. The others are simply assigned.
- So `x := 5` then `x := 6` is an error ("no new variables on left side of :="), but `x, y := 6, 7` is allowed: `y` is new, `x` just gets 6.

**Idiomatic choice (what Go reviewers expect):**

- Use `:=` inside functions when you have a starting value: `total := 0`, `name := "Ravi"`.
- Use `var x T` when you deliberately want the zero value, especially when the zero value is meaningful: `var count int`, `var found bool`.
- Use `var x T = value` only when the type must differ from the default type: `var ratio float32 = 1.5`, `var b byte = 'A'`.
- Avoid `var count int = 0`: the `= 0` repeats what the zero value already gives. Linters point this out.

**The blank identifier `_`:** a write-only placeholder that throws a value away: `_, err := strconv.Atoi(s)` when you only need the error. `_ = x` silences "declared and not used" during development, but should not stay in finished code.

**Common mistakes:**

- Using `:=` when the variable already exists (error) or when you meant `=` in an inner block (creates a NEW variable: shadowing, Unit 1.9).
- Trying `:=` at package level.
- Expecting a variable's type to change after assigning a different kind of value.
- Writing `= 0`, `= ""`, `= false` explicitly when the zero value already does it.

### Deep & Tricky

**DT-45. `:=` with existing variables: the `err` idiom.** *(Guarantee)* Because `:=` only needs one new variable on the left, Go code commonly writes:

```go
a, err := strconv.Atoi(s1)
b, err := strconv.Atoi(s2) // b is new; err is REUSED (assigned), not redeclared
```

This only reuses `err` if it was declared in the SAME scope. Inside an inner block (like an `if`), `:=` creates a brand-new `err` that hides the outer one: the famous shadowing bug (Unit 1.9).

**DT-46. Multiple assignment evaluates the right side first.** *(Guarantee: the spec defines two phases)* `a, b = b, a` swaps two values with no temporary variable. All expressions on the right are evaluated using the OLD values, then the assignments happen. So `x := 1` followed by `x, y := x+1, x+10` gives `x = 2`, `y = 11` (not 12).

**DT-47. `1e3` is a float, even though 1000 is a whole number.** *(Guarantee)* Scientific notation always makes a floating-point constant, so `n := 1e3` gives `n` the type `float64`. `'a'` is a rune constant, so `c := 'a'` makes `c` an `int32` holding 97, not a string. Check with `%T` whenever unsure.

**DT-48. "Make the zero value useful."** *(Go proverb, design principle)* Go designers aim for types whose zero value is ready to use without setup. Examples you will meet: `var sb strings.Builder` and `var mu sync.Mutex` work immediately with no constructor. You will design your own types this way in Chapter 3.

**DT-49. Your editor and the compiler can word the same error differently.** *(Tooling behaviour; a common surprise)* For `count := 0` at package level, VS Code showed "expected declaration, found count". That message comes from gopls (the Go language server behind the VS Code extension), which uses the `go/parser` package. `go build` uses the compiler's own parser and says `syntax error: non-declaration statement outside function body`. Same problem, same category (syntax error), different words. When asked for "the compiler message", use `go build`.

## Unit 1.2 — Constants and untyped constants

**One-line summary:** A constant is a value fixed at compile time. Constants written without a type are **untyped**: exact numbers that have not chosen a type yet. They adapt to wherever they are used, and fall back to a default type only when nothing decides for them.

**Declaring constants:**

```go
const pi = 3.14159          // untyped floating-point constant
const maxRetries int = 3    // typed constant: type int, strict
const (
	greeting = "namaste"
	debug    = false
)
```

Rules:

- The value must be computable by the compiler: `x := 5; const y = x` is an error, because `x` is a variable.
- Only booleans, numbers (including runes), and strings can be constants. No constant slices, maps, or structs.
- `:=` cannot make a constant; only `const` can.

**Mental model — untyped constant = water, typed variable = a container.** An untyped constant is like water: it takes the shape of whatever container it is poured into. `2` poured into a `float64` becomes float64 2.0; poured into a `uint8` it becomes uint8 2. If there is no container (`x := 2`, or passing to `fmt.Println`), it uses its default shape (the default type, Unit 1.1). A typed constant is water already frozen into ice of one shape: it only fits that shape.

**Why `f * 2` works but `f * n` does not:**

```go
var f float64 = 2.5
fmt.Println(f * 2)   // 5: the untyped 2 adapts to float64

n := 2               // n is a VARIABLE of type int
fmt.Println(f * n)   // type error: mismatched types float64 and int
```

**Representable or not — the compiler checks the exact value:**

| Code | Result | Why |
| --- | --- | --- |
| `var i int = 2.0` | OK, i = 2 | 2.0 is exactly a whole number, so it fits int |
| `var i int = 2.5` | compile error (truncated) | 2.5 cannot be an int without losing .5 |
| `var b byte = 300` | compile error (overflows) | 300 does not fit in 8 bits |
| `const t int = 5; var f float64 = t` | type error | `t` is typed int; typed constants do not adapt |

**Kinds of untyped constants:** boolean, rune, integer, floating-point, complex, string. When kinds mix in one expression, the result takes the kind that comes later in the list rune → integer → floating-point → complex. Example: `1 + 2.5` is untyped float 3.5; `'a' + 1` is an untyped RUNE constant 98, so `fmt.Println('a' + 1)` prints `98` (not `b`).

**Exact, huge precision:** untyped constants are exact and can be much bigger than any Go type, as long as the final value fits where it is used:

```go
const big = 1 << 100
fmt.Println(big >> 98)   // 4: the huge intermediate is fine
fmt.Println(big)         // compile error: overflows int
```

**iota — a counter inside a const block:**

```go
const (
	Small  = iota // 0
	Medium        // 1  (expression repeated automatically)
	Large         // 2
)
```

- `iota` starts at 0 in each `const` block and goes up by 1 on each line (each constant specification).
- A line with no expression repeats the previous line's expression, with the new iota.
- `_` skips a value: it uses up one iota number without making a name.
- Expressions work: `1 << iota` gives 1, 2, 4, 8 (bit flags); `1 << (10 * (iota + 1))` gives KB, MB, GB.

In Chapter 3 you will attach iota constants to your own named type, which is Go's way of writing enums.

**Common mistakes:**

- Thinking a constant has a type just because it has a value (`const x = 5` is untyped).
- Expecting typed constants to adapt like untyped ones.
- Trying to make a constant from a variable or a function call result.
- Expecting `'a' + 1` to be a letter. Runes are numbers.
- Writing constant names in ALL\_CAPS (see P-6).

### Deep & Tricky

**DT-50. `var i int = 2.0` compiles.** *(Guarantee)* An untyped constant can be converted to a type if its exact value is representable in that type. 2.0 is exactly the whole number 2, so it fits `int`. 2.5 is not, so `var i int = 2.5` fails with "truncated". The rule is about the VALUE, not about how it was written.

**DT-51. Compilers must handle very large constants.** *(Guarantee: implementation restriction in the spec)* The spec requires compilers to represent integer constants with at least 256 bits, and floating-point constants with at least a 256-bit mantissa. That is why `1 << 100` is fine as an intermediate value. Overflow is only checked when a constant is finally given a real type.

**DT-52. iota counts lines, not uses.** *(Guarantee)* Two constants on the same line share the same iota:

```go
const (
	a, b = iota, iota * 10 // a=0, b=0
	c, d                   // c=1, d=10 (line repeated with iota = 1)
)
```

**DT-53. `'a' + 1` prints 98.** *(Guarantee)* Mixing a rune constant with an integer constant gives an untyped rune constant, whose default type is `rune` (int32). `fmt.Println` prints an int32 as a number. To print the character, use `%c`: `fmt.Printf("%c\n", 'a'+1)` prints `b`.

### Pattern

**P-6. Name constants in MixedCaps, never ALL\_CAPS.** *(Go convention)*

- *How:* `maxRetries` (only usable inside its package) or `MaxRetries` (usable from other packages; the first letter decides visibility, Chapter 3). Not `MAX_RETRIES`.
- *Why:* in Go, capitalisation already has a meaning (exported or not), so ALL\_CAPS would make every constant exported and look foreign to Go readers. The standard library follows this: `math.MaxInt64`, `http.StatusOK`.
- *Also:* replace "magic numbers" with named constants: `const maxUploadSize = 10 << 20` is clearer than a bare `10485760` in the middle of code.

## Topic 1B — Operators and numbers

## Unit 1.3 — Arithmetic, comparison, and logical operators

**One-line summary:** Operators combine values into new values. Go's rules are strict and explicit: both sides must have the same type, integer division drops the fraction, comparisons give `bool`, and `&&` / `||` stop early when the answer is already known.

**Arithmetic operators:** `+  -  *  /  %`

- Both operands must have the same type (untyped constants adapt, Unit 1.2).
- `+` also joins strings: `"Go" + "lang"` = `"Golang"`.
- `%` (remainder) works only on integers.

**Integer division drops the fraction, towards zero:**

| Expression | Result | Note |
| --- | --- | --- |
| `7 / 2` | `3` | .5 dropped |
| `-7 / 2` | `-3` | towards zero (Python gives −4) |
| `7 % 3` | `1` |  |
| `-7 % 3` | `-1` | the remainder takes the sign of the LEFT number |
| `7 % -3` | `1` |  |
| `7.0 / 2` | `3.5` | float division, because 7.0 is a float constant |

The rule that ties them together: `(a / b) * b + a % b == a` always holds.

**Comparison operators:** `==  !=  <  <=  >  >=`. The result is always a `bool`. Strings compare byte by byte, like a dictionary that uses character codes: uppercase letters (65–90) come before lowercase (97–122), so `"Zebra" < "apple"` is `true`, and `"10" < "9"` is `true` (it compares `'1'` with `'9'` first).

**Logical operators:** `&&` (and), `||` (or), `!` (not). They work only on `bool`.

**Short-circuit evaluation (guarantee):** the right side is evaluated only if it is needed.

- `a && b`: if `a` is false, the answer is false; `b` is never evaluated.
- `a || b`: if `a` is true, the answer is true; `b` is never evaluated.

So order matters: `d != 0 && 10/d > 1` is safe even when `d` is 0, but `10/d > 1 && d != 0` panics first.

**Increment and compound assignment:**

- `x++` and `x--` are STATEMENTS, not expressions. They produce no value, so `y := x++` is a syntax error. There is no `++x` form.
- Compound assignment: `x += 5`, `x -= 2`, `x *= 3`, `x /= 2`, `x %= 4` mean `x = x + 5` and so on.

**Precedence (who goes first), simplified:** `*  /  %` before `+  -`, then comparisons, then `&&`, then `||`. Use parentheses whenever a reader might hesitate. The full table, including bit operators, comes in Unit 1.4.

**Common mistakes:**

- Expecting `7 / 2` to be 3.5 (both are integers).
- Checking "odd" with `n%2 == 1`, which is false for negative odd numbers (`-3 % 2` is `-1`). Use `n%2 != 0`.
- Putting the dangerous check on the wrong side of `&&`.
- Comparing strings expecting human alphabetical order regardless of case.
- Using `x++` inside an expression.

### Deep & Tricky

**DT-54. `var f float64 = 7 / 2` gives 3, not 3.5.** *(Guarantee)* Both `7` and `2` are untyped INTEGER constants, so the compiler first calculates `7 / 2` as an integer constant expression: 3. Only then is the result converted to float64: 3.0. The float type on the left does not reach into the expression. Write `7.0 / 2` (or `7 / 2.0`) to get 3.5. The same trap appears with variables: `float64(a / b)` divides as integers first; `float64(a) / float64(b)` gives the fraction.

**DT-55. Go truncates towards zero; Python floors.** *(Guarantee)* Go: `-7 / 2` = −3 and `-7 % 2` = −1. Python: `-7 // 2` = −4 and `-7 % 2` = 1. Code translated from Python (or from maths formulas that assume a non-negative remainder) can silently break for negative inputs. Example: wrapping an index around a circular buffer with `i % n` gives a negative index when `i` is negative.

**DT-56. `x++` is a statement on purpose.** *(Language design)* In C, `y = x++ + ++x` style code is legal but confusing, and some forms are undefined. Go removed the problem: `++` and `--` cannot be used inside expressions at all, and there is no prefix form. One clear way to do it.

**DT-57. `fmt.Println` prints `4.0` as `4`, and `1000000.0` as `1e+06`.** *(fmt behaviour; corrected)* For floats, `%v` (what `Println` uses) prints the shortest text that still represents the exact value, so `4.0` prints as `4` and `4.5` as `4.5`. It switches to exponent form when the exponent is 6 or more, or less than −4: `fmt.Println(1000000.0)` prints `1e+06`, `fmt.Println(1234567.0)` prints `1.234567e+06`, and `fmt.Println(0.00001)` prints `1e-05`. (Correction: an earlier version of this note said the switch happens at `1e+21`. That threshold belongs to `encoding/json`, not `fmt`.) To control output, use `%.2f` (fixed decimals), `%e` (always exponent), or `%g` with a precision.

## Unit 1.4 — Bitwise operators, shifts, and precedence

**One-line summary:** Bitwise operators work on each bit position separately, like eight (or 64) tiny independent calculations side by side. Shifts slide all bits left or right. Go's precedence rules put bit operators at the same level as multiplication and addition, which avoids C's famous traps.

**The bitwise operators (integers only):**

| Operator | Name | Result bit is 1 when… | Everyday use |
| --- | --- | --- | --- |
| `a & b` | AND | both bits are 1 | check or keep certain bits (a "mask") |
| `a \| b` | OR | at least one bit is 1 | turn bits on |
| `a ^ b` | XOR | the bits are different | flip selected bits |
| `a &^ b` | AND NOT (bit clear) | bit is 1 in `a` and 0 in `b` | turn bits off (Go-specific operator) |
| `^a` | NOT (unary) | the bit of `a` is 0 | flip every bit |

Go uses `^a` for NOT. There is no `~` operator for this (in Go, `~` only appears in generic constraints, Chapter 3).

Worked example with `uint8`:

```
a      = 1100 1010
b      = 1010 0110
a & b  = 1000 0010
a | b  = 1110 1110
a ^ b  = 0110 1100
a &^ b = 0100 1000   (start from a, switch off every bit that is 1 in b)
^a     = 0011 0101
```

**Shifts:** `x << n` moves every bit n places left (filling with 0s); `x >> n` moves them right.

- `x << n` multiplies by 2^n, as long as nothing falls off the left end (then it wraps, like overflow).
- `x >> n` for unsigned types divides by 2^n, dropping the remainder. New bits on the left are 0.
- `x >> n` for signed types is an **arithmetic shift**: new bits on the left copy the sign bit, so negative numbers stay negative. `-8 >> 1` = −4.
- The shift count must be an integer and not negative. A negative variable count panics at run time; a negative constant count is a compile error.
- Shifting by the type's width or more is defined (unlike C): `<<` gives 0; `>>` gives 0, or −1 for a negative signed number.

**Bit flags — the classic use:**

```go
const (
	Exec  = 1 << iota // 001 = 1
	Write             // 010 = 2
	Read              // 100 = 4
)

perm := Read | Write          // 110: turn on two flags
hasWrite := perm&Write != 0   // check one flag: true
perm |= Exec                  // turn on Exec: 111
perm &^= Write                // turn off Write: 101
perm ^= Read                  // toggle Read: 001
```

These values match Unix permission bits from Unit 0.2 (r = 4, w = 2, x = 1).

**Full precedence table (higher number = done first):**

| Level | Operators |
| --- | --- |
| 5 | `*  /  %  <<  >>  &  &^` |
| 4 | `+  -  \|  ^` |
| 3 | `==  !=  <  <=  >  >=` |
| 2 | `&&` |
| 1 | `\|\|` |

Operators on the same level go left to right. Unary operators (`-x`, `!x`, `^x`) bind tightest of all.

Consequences worth remembering:

- `x & 1 == 0` means `(x & 1) == 0`, which is what you meant. (In C it means `x & (1 == 0)`, a classic bug.)
- `1 << 2 + 1` means `(1 << 2) + 1` = 5. (In C it is `1 << 3` = 8.)
- `a | b & c` means `a | (b & c)`.
- Even though Go's rules are friendlier, add parentheses whenever mixing bit and arithmetic operators.

**Common mistakes:**

- Using `^` for powers (`2 ^ 3` is XOR = 1, not 8). Go has no power operator; use `1 << n` for powers of two, or `math.Pow` for floats.
- Using `&&` / `||` (logical, for bool) where `&` / `|` (bitwise, for integers) is meant, or the reverse.
- Forgetting `!= 0` when checking a flag: `if perm&Write` does not compile, because the result is an int, not a bool.
- Expecting `>>` on a negative number to behave exactly like `/ 2` (see DT-58).

### Deep & Tricky

**DT-58. `-7 >> 1` is −4, but `-7 / 2` is −3.** *(Guarantee)* An arithmetic right shift rounds DOWN (towards minus infinity): `1111 1001` (−7) shifted right with sign copying gives `1111 1100` (−4). Integer division rounds towards ZERO (DT-55): −3. For non-negative numbers both give the same result, which is why compilers can replace `x / 2` with a shift only when they know `x` is not negative (or add a correction step).

**DT-59. Shifts are fully defined in Go.** *(Guarantee)* In C, shifting by a negative amount or by at least the type's width is undefined behaviour. In Go: a negative variable shift count panics (`negative shift amount`), and a too-large count gives 0 (or −1 for a negative signed value shifted right). Since Go 1.13 the shift count may also be a signed integer type; before that it had to be unsigned.

**DT-60. Untyped constants in non-constant shifts.** *(Guarantee; advanced)* In `1 << s` where `s` is a variable, the untyped `1` gets the type it would have if the shift were not there. So `var i int64 = 1 << s` makes the `1` an int64, but `var f float64 = 1 << s` is a compile error: the `1` would become a float64, and floats cannot be shifted. This example comes straight from the spec.

**DT-61. Go has no power operator.** *(Language design)* `^` is XOR. `2 ^ 3` is `0010 ^ 0011` = `0001` = 1. Powers of two: `1 << n`. Other powers: `math.Pow(x, y)`, which works on float64.

### Pattern

**P-7. Bit flags for small sets of on/off options.** *(Common pattern, especially at system boundaries)*

- *Problem it solves:* storing several yes/no options compactly and combining them in one value: file permissions, protocol header flags, OS options. The standard library does this, for example `os.O_RDONLY | os.O_CREATE` when opening files.
- *How:* `const ( A = 1 << iota; B; C )`, then `|` to combine, `&` with `!= 0` to check, `&^` to clear.
- *Why Go developers use it:* cheap to store, copy, and compare; matches how operating systems and network formats already represent options.
- *When NOT to use it:* for a few options inside ordinary application code, separate `bool` fields (Chapter 3 structs) are easier to read and debug. Bit flags also stop at 64 options in one `uint64`.
- *Trade-off:* compact and fast versus less readable. Printing a flags value shows a number, not names, unless you write extra code.

## Unit 1.5 — Floating-point numbers and their traps

**One-line summary:** A float stores a number like scientific notation in binary: a sign, some significant digits, and a power of 2. This gives a huge range and fractions, but only a fixed number of significant digits, so most decimal fractions (like 0.1) are stored approximately.

**Mental model — scientific notation in base 2.** In school: 6.02 × 10^23 (digits × power of 10). A float64 stores: sign × 1.xxxx (in binary) × 2^exponent.

| Type | Sign | Exponent bits | Fraction bits | About this many correct decimal digits |
| --- | --- | --- | --- | --- |
| `float32` | 1 | 8 | 23 | 6–9 |
| `float64` | 1 | 11 | 52 | 15–17 |

`float64` is the default type for float constants and the normal choice. Use `float32` only when memory matters a lot (graphics, machine learning, huge arrays).

**Why 0.1 is not exact.** In decimal, 1/3 = 0.3333… never ends. In binary, 1/10 never ends: 0.0001100110011… With 52 fraction bits, the computer must cut it off and round. Only fractions whose denominator is a power of 2 are exact: 0.5 (1/2), 0.25 (1/4), 0.75 (3/4), 0.375 (3/8).

```go
a, b := 0.1, 0.2
fmt.Println(a + b)        // 0.30000000000000004
fmt.Println(a+b == 0.3)   // false
fmt.Println(0.1+0.2 == 0.3) // true! (constants are exact; see DT-62)
```

**Comparing floats: use a tolerance, not `==`.**

```go
const eps = 1e-9
if math.Abs(x-y) < eps { /* treat as equal */ }
```

The right tolerance depends on the size of the numbers and the domain. For values that can be very large or very small, compare relative to their size: `math.Abs(x-y) <= eps*math.Max(math.Abs(x), math.Abs(y))`.

**Whole numbers in float64 are exact only up to 2^53 = 9,007,199,254,740,992.** Beyond that, the gaps between representable floats are bigger than 1, so some integers cannot be stored and get rounded to a neighbour.

**Float → int conversion drops the fraction, towards zero:**

| Code | Result |
| --- | --- |
| `int(3.9)` (with a float variable) | 3 |
| `int(-3.9)` | −3 |
| `math.Round(2.5)` | 3 (halves round away from zero) |
| `math.Round(-2.5)` | −3 |
| `math.Floor(-2.5)` | −3 (always down) |
| `math.Ceil(-2.5)` | −2 (always up) |
| `math.Trunc(-2.5)` | −2 (towards zero) |

`math.Round` and friends return a `float64`; convert with `int(...)` afterwards if you need an int.

**Printing floats:** `Println` uses the shortest exact form (DT-57). For reports use `%.2f`. For scientific values use `%e`.

**Special values (from Unit 0.1):** `+Inf`, `-Inf`, `NaN`, and also negative zero `-0` (DT-65).

**Real-world float bugs:**

- **Patriot missile, 1991:** the system counted time in tenths of a second using a number format that could not store 0.1 exactly. After about 100 hours of running, the clock had drifted by about a third of a second, enough to miss an incoming missile; 28 soldiers died. (US GAO report, 1992.)
- **Vancouver Stock Exchange, 1982:** an index was recalculated thousands of times a day and each result was cut (truncated) to 3 decimals instead of rounded. After about two years the index showed roughly half its true value.

**Common mistakes:**

- Using `==` on computed floats.
- Using floats for money (see P-8).
- Expecting `int(2.9)` to round to 3.
- Using `float32` "to save memory" in normal code and losing precision.
- Assuming every large whole number fits exactly in a float64.

### Seven simple ideas that make floats click

**Idea 1 — The 4-digit calculator.** Imagine a calculator that can show only 4 digits after the decimal point. 1 ÷ 3 = 0.3333 (the rest is cut). Now 0.3333 × 3 = 0.9999, not 1. The small error came from cutting. A float64 is the same kind of calculator: it keeps about 16 significant digits and cuts the rest.

**Idea 2 — Why 0.1 must be cut.** Whether a fraction ends depends on the number system:

- Decimal (base 10): 1/2 = 0.5 ends, 1/4 = 0.25 ends, but 1/3 = 0.333… never ends.
- Binary (base 2): 1/2 = 0.1 ends, 1/4 = 0.01 ends, 3/4 = 0.11 ends, but 1/10 = 0.000110011001100… never ends.

So 0.1 in binary is like 1/3 in decimal: it must be cut, and a tiny error stays. Fractions that are halves, quarters, eighths (and sums of them) are exact; 0.1, 0.2, 0.3 are not.

**Idea 3 — See the hidden digits.** Ask for 20 decimals:

| Code | Prints |
| --- | --- |
| `fmt.Printf("%.20f\n", 0.1)` | `0.10000000000000000555` |
| `fmt.Printf("%.20f\n", 0.2)` | `0.20000000000000001110` |
| `fmt.Printf("%.20f\n", 0.3)` | `0.29999999999999998890` |
| `fmt.Printf("%.20f\n", 0.5)` | `0.50000000000000000000` (exact) |

The stored 0.1 and 0.2 are both a little TOO BIG; the stored 0.3 is a little TOO SMALL. So 0.1 + 0.2 (too big + too big) cannot land on the stored 0.3 (too small). Result: `0.30000000000000004`.

**Idea 4 — Paper vs calculator (constants vs variables).** The compiler working on constants is like doing maths on paper with exact fractions: 1/10 + 2/10 = 3/10 exactly, so `0.1+0.2 == 0.3` is true. Variables are like the cutting calculator: each value is cut first, then added, so `a+b == 0.3` is false.

**Idea 5 — "Close enough" instead of "exactly equal".** A jeweller weighing gold reads 10.0001 g, then 10.0002 g. He does not say "different gold"; the difference is smaller than his scale's accuracy. Same for floats: `math.Abs(x-y) < 0.000001` means "close enough". How small "close enough" should be depends on the job: 1 mm is nothing for distances between cities, but huge for parts inside a phone chip.

**Idea 6 — `int()` tears off the decimal part.** `int(7.8)` = 7 and `int(-7.8)` = −7: the digits after the point are simply thrown away, never rounded. For rounding, use `math.Round(7.5)` = 8.

**Idea 7 — Money in paise.** A shopkeeper who writes ₹2.50 as 250 paise never has decimal problems: 250 + 175 = 425 paise = ₹4.25, exactly. With floats, `x := 0.1; x + x + x` gives `0.30000000000000004`. With paise, `p := 10; p + p + p` gives exactly 30.

### Deep & Tricky

**DT-62. `0.1 + 0.2 == 0.3` is TRUE in Go when written as constants.** *(Guarantee)* With constants, the compiler does exact arithmetic (Unit 1.2): 0.1 + 0.2 is exactly 0.3, so the comparison is true. With float64 VARIABLES, each value is first rounded to binary, and the sum is 0.30000000000000004, so `a+b == 0.3` is false. Same text, different results: "who calculates?" once more. Many developers who know the classic 0.30000000000000004 fact are surprised by the constant case.

**DT-63. float32 → float64 shows the hidden error.** *(IEEE 754 behaviour)* `f := float32(0.1)` prints as `0.1`, because `Println` prints the shortest text that identifies that float32. But `float64(f)` prints `0.10000000149011612`: converting does not add precision back; it reveals how far the float32 value really was from 0.1.

**DT-64. Out-of-range float → int conversion does not panic.** *(Guarantee that it does not panic; the result is implementation-dependent)* `f := 1e30; i := int64(f)` compiles and runs, but the spec says the result is implementation-dependent when the value does not fit. It may differ between CPU types (amd64 vs arm64). Always check the range before converting floats that come from outside. (A constant like `int64(1e30)` is a compile error instead.)

**DT-65. Negative zero.** *(IEEE 754 behaviour)* Floats have both `+0` and `−0` (sign bit set, everything else zero). `-0.0 == 0.0` is true, so it usually does not matter, but `1 / negZero` gives `-Inf` instead of `+Inf`, and `fmt.Println` prints `-0`. Get it with `math.Copysign(0, -1)`; a constant `-0.0` is just exact 0 (no sign), so it does not produce negative zero.

**DT-66. Integers above 2^53 lose exactness in float64.** *(IEEE 754 behaviour)* `n := 9007199254740993` (2^53 + 1) as an int is exact, but `float64(n)` becomes 9007199254740992. So `int(float64(n)) == n` is false. This matters when JSON numbers pass through JavaScript (which uses float64 for all numbers): large IDs get silently changed. That is why APIs often send big IDs as strings.

### Pattern

**P-8. Never use floats for money; use integer minor units.** *(Universal engineering practice)*

- *Problem it solves:* 0.1 and 0.01 are not exact in binary, so sums of prices drift (0.30000000000000004), and rounding differences across millions of transactions cause accounting mismatches.
- *How:* store money as an integer count of the smallest unit: paise for rupees (₹10.50 = 1050), cents for dollars. Use `int64`. Add and subtract exactly. Format only at the edge: `fmt.Printf("₹%d.%02d\n", p/100, p%100)` (for non-negative amounts).
- *When you need more:* interest rates, currency conversion, or tax with many decimal places need a decimal type (for example the `shopspring/decimal` library, or `math/big`), with explicit rounding rules.
- *Trade-off:* you must think about rounding explicitly (for example, when splitting ₹100 three ways) instead of hiding it inside float errors. That is a feature: the rounding rule becomes a visible business decision.

## Topic 1C — Control flow

## Unit 1.6 — if, else, and switch

**One-line summary:** `if` runs code only when a `bool` condition is true; `else if` / `else` choose between options; `switch` picks the FIRST matching case from top to bottom and then stops. In machine code, all of these are conditional jumps (Unit 0.5).

**`if` rules:**

```go
if marks >= 35 {
	fmt.Println("pass")
} else if marks >= 25 {
	fmt.Println("supplementary exam")
} else {
	fmt.Println("fail")
}
```

- No brackets around the condition; braces `{ }` are ALWAYS required, even for one line.
- The condition must be a `bool`. `if n { }` with an int `n` is a type error; Go has no "truthy" values (0, "" and nil are not automatically false).
- `else` must be on the same line as the closing `}` (automatic semicolons, DT-17).
- Go has no ternary operator (`a > b ? a : b`). Write a short if/else instead.

**`if` with a short statement:**

```go
if n, err := strconv.Atoi(s); err != nil {
	fmt.Println("bad number:", err)
} else {
	fmt.Println("double:", n*2)
}
// n and err do NOT exist here
```

The variables made before the `;` exist only inside this if / else if / else chain (including the else branches). This keeps short-lived variables from leaking into the rest of the function.

**`switch` with a value:**

```go
switch day {
case "Sat", "Sun":          // several values in one case
	fmt.Println("weekend")
case "Mon":
	fmt.Println("start of week")
default:                    // when nothing else matches (optional)
	fmt.Println("weekday")
}
```

- Cases are checked top to bottom; the FIRST match runs, then the switch ends. No `break` needed (unlike C and Java).
- `default` runs when no case matches; it can be written anywhere but is usually last.
- Two identical constant cases are a compile error (`duplicate case`).

**`switch` with no value (tagless switch)** replaces long if-else chains:

```go
switch {
case marks >= 90:
	grade = "A"
case marks >= 75:
	grade = "B"
default:
	grade = "C"
}
```

Order matters: put the most specific (strictest) condition first, because the first true case wins.

**`fallthrough`:** at the end of a case, it jumps into the NEXT case's body WITHOUT checking that case's condition. Rarely needed; many Go programmers never use it. It cannot be used in the last case.

**Common mistakes:**

- Using a non-bool as a condition (`if count`).
- Expecting C-style fall-through between cases (Go stops after one case).
- Thinking `fallthrough` checks the next case's condition.
- Putting a broad condition first in a tagless switch, so later cases never run.
- Trying to use variables from an `if` short statement after the if.

### Deep & Tricky

**DT-67. `fallthrough` ignores the next case's condition.** *(Guarantee)*

```go
x := 2
switch x {
case 2:
	fmt.Println("two")
	fallthrough
case 100:
	fmt.Println("hundred") // printed, even though x is not 100
}
```

The word sounds like "continue checking", but it means "run the next body no matter what". It must be the last statement in its case, cannot appear in the last case, and is not allowed in type switches (Chapter 3).

**DT-68. `break` inside a switch only leaves the switch.** *(Guarantee; matters in Unit 1.7)* A `break` written in a case ends the switch, not a surrounding `for` loop. To leave the loop from inside a switch you need a labelled break (`break outer`), covered with loops. This catches many people writing a loop with a switch inside.

**DT-69. Switch cases can be any expressions, evaluated in order.** *(Guarantee)* Case values do not have to be constants: `case x * 2:` or `case getLimit():` are legal. They are evaluated top to bottom, left to right, only until a match is found, so a function call in a later case may never run (like short-circuit evaluation). Only duplicate CONSTANT cases are rejected by the compiler.

**DT-70. Safe absolute value for printing: `uint64(-n)`.** *(Guarantee, uses DT-24 and DT-41)* For every negative int64 `n`, `uint64(-n)` gives the correct size, even for MinInt64. Why: `-n` of MinInt64 wraps back to MinInt64 (bits `1000…0000`), and reading those same bits as unsigned gives 2^63 = 9,223,372,036,854,775,808, which is exactly the right size. Small example with int8: n = −128 → −n = −128 (`1000 0000`) → `uint8` reads 128. So `fmt.Printf("-0x%X", uint64(-n))` prints `-0x8000000000000000` correctly.

### Pattern

**P-9. Guard clauses: handle problems first, keep the happy path un-indented.** *(Strong Go convention)*

- *Problem it solves:* deeply nested if/else ("arrow code") where the main logic sits four levels deep and the error handling is far from the check that caused it.
- *How:* check each bad case at the top and leave immediately (`return`, `os.Exit`, later `continue`), so the normal path continues down the left edge. Your numinfo `main` already does this: wrong argument count → exit; bad number → exit; then the real work.
- *Why Go developers do it:* Go code is read top to bottom as "check, check, check, do the work". With errors as values (Chapter 3), `if err != nil { return err }` after each step is the standard shape.
- *Related rule:* if an `if` block ends with `return` (or exit), do not write `else`: just continue after the block. Linters flag the unnecessary `else`.
- *When not to:* when both branches are equally normal (for example, choosing a label for even vs odd), a plain if/else is clearer.

## Unit 1.7 — for loops

**One-line summary:** Go has only ONE loop keyword, `for`, used in four shapes. `break` leaves a loop, `continue` skips to the next round, and labels let you break or continue an outer loop from inside an inner one.

**Mental model:** a loop is a conditional jump backwards (Unit 0.5): "if the condition is still true, jump back to the top of the body".

**The four shapes:**

```go
// 1. Three-part loop: start; condition; after-each-round
for i := 0; i < 5; i++ {
	fmt.Println(i) // 0 1 2 3 4
}

// 2. Condition only (Go's "while")
n := 1
for n < 100 {
	n *= 2
}

// 3. Forever (stop with break or return)
for {
	// ...
	if done {
		break
	}
}

// 4. range over an integer (Go 1.22+): i = 0, 1, 2, 3, 4
for i := range 5 {
	fmt.Println(i)
}
for range 3 { // no variable needed
	fmt.Println("hi")
}
```

`range` also loops over strings, slices, maps, and channels (Chapter 2 and Chapter 7).

**Order of the three-part loop:** run `init` once → check `condition` → run body → run `post` → check condition → body → … until the condition is false.

**`break` and `continue`:**

- `break` leaves the loop immediately.
- `continue` skips the rest of this round and goes to the next round (in a three-part loop, the `post` statement still runs).

**Labels for nested loops:**

```go
outer:
for i := 0; i < 3; i++ {
	for j := 0; j < 3; j++ {
		if i*j == 2 {
			break outer // leaves BOTH loops
		}
	}
}
```

Without a label, `break` only leaves the innermost loop (or switch, DT-68).

**Common loop recipes:**

- Sum / count: start a variable at 0 before the loop, add inside.
- Largest value: start with the first value, compare each next value.
- Search: loop, and `break` as soon as you find it.

**Common mistakes:**

- Off-by-one: `i <= n` vs `i < n`. Ask: "how many times does this run?" `for i := 0; i < n; i++` runs exactly n times.
- Forgetting to change the condition variable in a condition-only loop (infinite loop).
- Unsigned countdown `for i := uint(3); i >= 0; i--` never ends (DT-22).
- Expecting `break` inside a `switch` to leave the loop.
- Adding floats in a loop and expecting an exact result (DT-74).

### Deep & Tricky

**DT-71. Changing the loop variable: three-part loop vs range.** *(Guarantee)*

```go
for i := 0; i < 5; i++ { fmt.Print(i, " "); i += 2 } // 0 3
for i := range 5        { fmt.Print(i, " "); i += 2 } // 0 1 2 3 4
```

In a three-part loop, the change carries into the `i++` and the next check. In `range 5`, the next value always comes from the range itself (0, 1, 2, 3, 4), so changing `i` inside the body has no effect on later rounds.

**DT-72. Since Go 1.22, each loop round gets a NEW variable.** *(Guarantee; depends on the `go` line in go.mod)* Before Go 1.22, a loop had ONE variable `i` reused for all rounds. Code that captured `i` in a closure or goroutine (Unit 1.10, Chapter 7) saw only the last value, a very famous bug. Since 1.22, every round has its own `i`. The rule is chosen per module by the `go` version line in `go.mod`: a module saying `go 1.27` gets the new behaviour. Old modules that say `go 1.21` keep the old behaviour, even with a new toolchain.

**DT-73. Labels are only for break, continue, and goto.** *(Guarantee)* A label (`outer:`) must be used, or it is a compile error (`label outer defined and not used`). `continue outer` jumps to the next round of the outer loop (running its post statement). Go also has `goto`, but it is rarely used and cannot jump over variable declarations or into blocks.

**DT-74. Adding 0.1 ten times gives 0.9999999999999999.** *(IEEE 754 behaviour)* Each addition is rounded, and the small errors build up in a loop: `total := 0.0; for range 10 { total += 0.1 }` prints `0.9999999999999999`. With paise: `p := 0; for range 10 { p += 10 }` gives exactly 100. This is P-8 in action.

## Topic 1D — Functions and their behaviour

## Unit 1.8 — Functions

**One-line summary:** A function is a named, reusable piece of work with inputs (parameters) and outputs (results). Go copies every argument into the function's own frame, can return several results at once, and treats functions as ordinary values.

**Mental model — a tea stall order.** You give the chaiwala an order slip (arguments). He works in his own corner of the stall (his own stack frame, Unit 0.6), and hands back the cups (results). He works on a COPY of your slip: if he scribbles on it, your original slip is unchanged.

**Syntax:**

```go
func add(a int, b int) int {   // parameters, then result type
	return a + b
}

func add2(a, b int) int {       // same type: write it once
	return a + b
}
```

The order of functions in a file does not matter; `main` can call a function written below it.

**Multiple results:**

```go
func divmod(a, b int) (int, int) {
	return a / b, a % b
}

q, r := divmod(17, 5) // q = 3, r = 2
_, r2 := divmod(9, 4) // ignore the first result
```

The most common use in Go: `(value, error)`, as in `strconv.Atoi`. The error is always the LAST result.

**Named results:** `func divmod(a, b int) (q, r int)` gives names to the results. They start at their zero values, and a bare `return` returns their current values. Names help as documentation, but bare `return` in a long function hurts readability; prefer `return q, r`.

**Pass by value (copies):** Go always copies arguments.

```go
func double(x int) { x = x * 2 } // changes only the copy

n := 5
double(n)
fmt.Println(n) // 5
```

To get a new value back, return it: `func double(x int) int { return x * 2 }`, then `n = double(n)`. (Pointers, Chapter 2, are the other way.)

**Variadic functions (any number of arguments):**

```go
func total(nums ...int) int { // inside, nums is a slice of int
	sum := 0
	for _, n := range nums {
		sum += n
	}
	return sum
}

total()          // 0
total(1, 2, 3)   // 6
```

Only the LAST parameter can be variadic. `fmt.Println` is variadic.

**Functions are values:**

```go
square := func(x int) int { return x * x } // function literal stored in a variable

func apply(f func(int) int, v int) int {     // f has the function TYPE func(int) int
	return f(v)
}

apply(square, 4) // 16
```

The zero value of a function variable is `nil`; calling a nil function panics.

**Recursion:** a function calling itself. Always needs a base case that stops it.

```go
func factorial(n int) int {
	if n <= 1 { // base case
		return 1
	}
	return n * factorial(n-1)
}
```

Every call adds a stack frame (Unit 0.6). Without a base case: infinite recursion → `fatal error: stack overflow` (DT-39).

**What Go deliberately does NOT have:** default parameter values, function overloading (two functions with the same name), named arguments at the call site. Go prefers one clear way: different names, variadic parameters, or (Chapter 3) an options struct.

**Built-in `min` and `max` (Go 1.21+):** `max(3, 7, 5)` = 7. If you name your own function `max`, it hides (shadows) the built-in one in your package (Unit 1.9).

**Common mistakes:**

- Expecting a function to change the caller's variable by assigning to its parameter.
- Forgetting the base case in recursion.
- Putting the error anywhere but the last result.
- Long functions with bare `return` and named results.
- Naming functions `max`, `min`, `len`, `new` and hiding built-ins.

### Deep & Tricky

**DT-75. Go is ALWAYS pass-by-value.** *(Guarantee)* Every argument is copied into the called function's frame: ints, floats, strings, arrays, structs, pointers, slices, maps. Some of those values (pointers, slices, maps) CONTAIN an address, so the copy still leads to the same underlying data. That is why a function can change the elements of a slice you pass in, but not the slice's length as seen by the caller (Chapter 2). There is no "pass by reference" in Go. Copying a big array (for example `[1_000_000]int`) copies all of it on every call.

**DT-76. Go does not optimise tail calls.** *(Implementation behaviour)* Some languages turn `return f(n-1)` into a jump, so deep recursion uses no extra stack. Go does not: every recursive call adds a frame. Very deep recursion (millions of levels) can hit the 1 GB stack limit and die with `fatal error: stack overflow`. For very deep work, use a loop.

**DT-77. Calling a nil function value panics.** *(Guarantee)* `var f func(int) int` is `nil`. `f(3)` panics: `invalid memory address or nil pointer dereference`. Check `if f != nil` before calling a function value that may be unset.

**DT-78. Function values cannot be compared, except with nil.** *(Guarantee)* `f == g` for two function values is a compile error. Only `f == nil` is allowed. So function values cannot be map keys either.

**DT-79. Multiple results are not tuples.** *(Guarantee)* Go has no tuple type. `x := divmod(7, 2)` is a compile error (`assignment mismatch: 1 variable but divmod returns 2 values`). You also cannot put multiple results inside another expression like `divmod(7, 2) + 1`. One special case IS allowed: if a function's results exactly match another function's parameters, you can pass them straight through: `f(g())`. That is why `fmt.Println(divmod(7, 2))` works: Println accepts any number of values.

### Pattern

**P-10. `(result, error)` with the error last.** *(Universal Go convention)*

- *Problem it solves:* a function that can fail needs to report failure clearly, without exceptions hidden in the flow.
- *How:* return the normal result(s) first and an `error` last. The caller checks `if err != nil` immediately (P-9 guard clause) and uses the result only when `err` is nil. On error, the other results are usually zero values and should not be used.
- *Why Go developers do it:* the whole standard library follows it (`strconv.Atoi`, `os.Open`, `io.Reader.Read`), so every Go reader instantly understands the shape.
- *When not to:* when failure is truly impossible, do not add an error result "just in case". For a simple yes/no lookup, Go often uses `(value, ok bool)` instead (the comma-ok idiom, Chapter 2 maps).
- Full error handling (wrapping, `errors.Is`) comes in Chapter 3.

## Unit 1.9 — Scope and shadowing

**One-line summary:** The scope of a name is the part of the code where that name can be used. Scopes nest like boxes inside boxes. Go looks up a name from the innermost box outwards, and a name declared in an inner box hides (shadows) the same name from an outer box.

**Mental model — nicknames at home.** At home, "Bhaiya" means your elder brother. At the office, "Bhaiya" might mean the canteen manager. The word is the same; WHERE you say it decides who answers. Inside the home, the home meaning wins, even though the office one also exists.

**The levels of scope (outermost to innermost):**

| Level | What lives there | Example |
| --- | --- | --- |
| Universe | Predeclared names: types, constants, built-in functions | `int`, `string`, `true`, `nil`, `len`, `max` |
| Package | Top-level names in ANY file of the package | `func largest`, package-level `var`, `const` |
| File | Imports (each file imports for itself) | `import "fmt"` |
| Function | Parameters and results | `nums` in `largest(nums ...int)` |
| Block | Anything declared inside `{ }`, including if / for / switch / case blocks and their short statements | `x := 2` inside an if |

**Lookup rule:** start in the current block, then go outwards one level at a time. The first match wins.

**Shadowing:** declaring a name that already exists in an outer scope creates a NEW variable that hides the old one until the inner block ends.

```go
x := 1
if true {
	x := 2    // NEW x (note :=), hides the outer x
	x++
	fmt.Println("inner", x) // inner 3
}
fmt.Println("outer", x)     // outer 1 (never touched)
```

With `=` instead of `:=` inside the block, there is only one `x`, and the outer one becomes 3.

**The famous `err` shadowing bug:**

```go
var err error
if s != "" {
	n, err := strconv.Atoi(s) // := makes a NEW err inside the if
	_ = n
}
fmt.Println(err)              // always <nil>: the outer err was never set
```

Fix: declare `n` outside too and use `=`, or move the check inside the block.

**Shadowing names from outer levels:**

- Built-ins can be shadowed: `len := 3` makes `len("hi")` fail to compile (`cannot call non-function len`).
- Package names can be shadowed: `fmt := "hi"` makes `fmt.Println` fail (the name `fmt` now means a string).
- Your own package-level names too: a local variable named `largest` inside `func largest` hides the function itself inside its body.

**Order rules:** package-level names can be used before they appear in the file (order does not matter). Inside a function, a variable must be declared before it is used.

**Common mistakes:**

- Using `:=` inside an if / for when you meant to update an outer variable with `=`.
- Naming variables after built-ins (`len`, `max`, `new`, `copy`) or imported packages (`url`, `fmt`, `time`).
- Expecting a variable from an if or for short statement to exist after the block.
- Thinking the compiler always catches shadowing: it only complains if the inner variable is unused.

### Worked examples

**The nil-function program.** Expected output when run as a binary:

```
16
true
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation ...]   (details vary by machine)

goroutine 1 [running]:
main.main()
        .../main.go:14 +0x...
```

Exit code 2. `last line` never prints: the panic stops `main` at `g(3)`, so nothing after it runs. It compiled fine, because `g` exists and has the right type; it only fails when the program actually tries to call through a nil value (see DT-84). Compare: `undefined: apply` stops the build itself; no program is ever produced.

**`total := total + i` in a loop:**

| Round | i | right side uses OUTER total (10) | inner total | printed |
| --- | --- | --- | --- | --- |
| 1 | 0 | 10 + 0 | 10 | in loop: 10 |
| 2 | 1 | 10 + 1 | 11 | in loop: 11 |
| 3 | 2 | 10 + 2 | 12 | in loop: 12 |

Then `after: 10`. Each round makes a fresh inner `total` from the untouched outer one, so the values never build up.

### Deep & Tricky

**DT-80. Go has only 25 keywords; `true`, `nil`, `int`, `len` are NOT keywords.** *(Guarantee)* Keywords (like `func`, `if`, `for`, `return`, `var`) can never be used as names. But `true`, `false`, `nil`, `int`, `string`, `len`, `max`, `error`, and friends are predeclared identifiers in the universe scope, so they CAN be shadowed. `true := false` compiles, and after it `fmt.Println(true)` prints `false`. Legal, confusing, and a reason linters exist.

**DT-81. Imports belong to the file, not the package.** *(Guarantee)* Two files in the same package each need their own `import "fmt"`. But package-level functions, variables, and constants are shared by all files of the package. An unused import is checked per file.

**DT-82. The compiler only catches shadowing when the inner variable is unused.** *(Compiler behaviour)* If the shadowing variable is never used, you get `declared and not used`, which sometimes reveals the bug by accident. If it IS used (for example, printed or checked), the code compiles silently. The standard `go vet` does not check shadowing by default; a separate `shadow` analyzer (also available through linters such as golangci-lint) can report it.

**DT-83. A new variable's scope starts AFTER its declaration.** *(Guarantee)* In `total := total + i` inside a block, the right side is evaluated while only the OUTER `total` is visible; the new inner `total` comes into existence after the statement. So it reads the outer value, and then shadows it. The same rule makes `x := x` legal in an inner block (it copies the outer x into a new inner x).

**DT-84. Why calling nil crashes: page zero is never mapped.** *(OS and runtime behaviour)* A Go function value is (in the current implementation) a pointer to a small block that holds the code address. A nil function value points to address 0. Operating systems deliberately leave the lowest page of every process's address space unmapped (Unit 0.6, pages), so any read there makes the CPU raise a fault (`SIGSEGV`). The Go runtime turns that fault into the panic `invalid memory address or nil pointer dereference`. The same mechanism catches nil pointers, nil maps used wrongly, and similar bugs.

### Pattern

**P-11. Keep scopes small, but watch your `:=`.** *(Go convention)*

- *Problem it solves:* variables that live longer than needed get reused by mistake; short-lived values clutter the function.
- *How:* declare variables as close as possible to where they are used; use if / switch short statements (`if v, err := f(); err != nil {...}`) so helper variables disappear after the check.
- *The catch:* in an inner block, `:=` creates a NEW variable. When you mean to update an outer variable, use `=`.
- *Naming:* never name variables after built-ins (`len`, `max`, `new`, `copy`, `error`) or after imported packages. Use `n`, `maxVal`, `u` (for a URL), and so on.
- *When not to:* do not squeeze long logic into an if short statement just to limit scope; readability comes first.

## Unit 1.10 — Closures

**One-line summary:** A closure is a function literal that uses variables from the scope around it. It does not take a copy of those variables; it keeps a live link to the variables themselves, and they stay alive as long as the closure does.

**Mental model — a dabba (lunchbox) with a window.** Normally a function's local variables are thrown away when it returns (its frame disappears). A closure is like a small dabba that carries a window looking into the variables it uses. It does not carry a photo of them (a copy) but a real window: if the variable changes, the closure sees the change, and if the closure changes it, everyone else sees the change too.

**Example 1 — the closure sees later changes:**

```go
msg := "hello"
say := func() { fmt.Println(msg) }
msg = "bye"
say() // bye (it reads msg NOW, not when say was created)
```

**Example 2 — the closure can change the outer variable:**

```go
x := 10
add := func(n int) { x += n }
add(5)
add(5)
fmt.Println(x) // 20
```

**Example 3 — a function that returns a function (a generator):**

```go
func makeCounter() func() int {
	count := 0
	return func() int {
		count++
		return count
	}
}

c1 := makeCounter()
fmt.Println(c1(), c1()) // 1 2
c2 := makeCounter()
fmt.Println(c2())       // 1  (its own, separate count)
fmt.Println(c1())       // 3
```

Each call to `makeCounter` creates a NEW `count`. `c1` and `c2` each hold a window onto their own `count`.

**Where does `count` live?** `makeCounter` has returned, so its frame is gone, yet `count` is still used by `c1`. So the compiler's escape analysis moves `count` to the HEAP (Unit 0.6). `go build -gcflags=-m` reports `moved to heap: count`.

**The classic loop-closure bug (and the Go 1.22 fix):**

```go
var fs [3]func()
for i := 0; i < 3; i++ {
	fs[i] = func() { fmt.Println(i) }
}
for j := 0; j < 3; j++ {
	fs[j]()
}
```

- Go 1.22 and later (any module whose go.mod says `go 1.22` or newer): each round has its own `i`, so it prints 0 1 2.
- Go 1.21 and earlier: there was ONE `i` for the whole loop; all three closures had a window onto that same `i`, which was 3 after the loop, so it printed 3 3 3. This bug hit real production code for years, especially with goroutines (Chapter 7).

**Where closures are used:** counters and generators, sorting with a custom comparison, callbacks, HTTP handlers and middleware (Chapter 8), deferred cleanup (Unit 1.11).

**Common mistakes:**

- Thinking a closure copies the value at creation time (it links to the variable).
- Forgetting that two closures created in the same scope share the same variable.
- Assuming old code behaves like Go 1.22+ (check the module's go version).
- Using closures to hold lots of state that would be clearer as a struct with methods (P-12).

### Where to take the "photo"

```go
name := "Asha"
photo := name                              // photo taken NOW, outside: "Asha"
greet := func() { fmt.Println("Hi", photo) }
name = "Ravi"
greet()                                    // Hi Asha
```

The body of a closure runs only when the closure is called. A copy written inside the body is taken at CALL time; a copy written outside, before creating the closure, is taken at CREATION time.

### Deep & Tricky

**DT-85. Closures capture variables, not values.** *(Guarantee)* A closure refers to the variable itself. If you want a snapshot of the current value instead, copy it into a new variable first (`v := x`) and use `v` inside the closure, or pass it as an argument.

**DT-86. Captured variables that outlive their function move to the heap.** *(Implementation behaviour, via escape analysis)* In `makeCounter`, `count` must survive after `makeCounter` returns, so the compiler allocates it on the heap. You can see this with `go build -gcflags=-m`: `moved to heap: count`. If a closure is only used inside the function that created it (and does not escape), the captured variables can often stay on the stack. Each heap allocation costs a little time and adds GC work, which matters in very hot code (Chapter 10).

**DT-87. The loop-variable change is per module, controlled by go.mod.** *(Guarantee, Go 1.22+)* The new per-iteration loop variables apply only to code in modules whose `go.mod` says `go 1.22` or later. The same source file prints 3 3 3 under `go 1.21` and 0 1 2 under `go 1.22`, even with the same Go toolchain installed. This is how Go changed a language rule without breaking old code.

### Pattern

**P-12. Closures for small, local behaviour; structs for real state.** *(Go practice)*

- *Problem it solves:* passing a little custom behaviour (a comparison, a callback, a counter) without creating a whole new type.
- *How:* write a function literal right where it is needed, capturing only what it needs.
- *When NOT to:* when the hidden state grows (several variables, several operations, needs testing or inspection), use a struct with methods (Chapter 3). Hidden state inside closures is hard to see, debug, and test.
- *Trade-off:* closures are short and local, but their captured state is invisible from outside; and captured variables may cost a heap allocation (DT-86).

## Unit 1.11 — defer

**One-line summary:** `defer` schedules a function call to run just before the surrounding function finishes, whether it finishes normally or because of a panic. Deferred calls run in reverse order (LIFO), and their arguments are worked out immediately, at the `defer` line.

**Mental model — the "before you leave" checklist.** When you enter an office, you note on a sticky pad: "switch off the AC", then "lock the cupboard", then "return the visitor badge". The notes pile up; when you leave, you go through the pile from the TOP (the last note first): badge, cupboard, AC. `defer` adds a note to the pile; leaving the function goes through the pile.

**Basic use:**

```go
func main() {
	fmt.Println("start")
	defer fmt.Println("cleanup") // runs when main is finishing
	fmt.Println("working")
}
// start
// working
// cleanup
```

**Rule 1 — LIFO order:** several defers run last-in, first-out, like stack frames (Unit 0.6).

```go
defer fmt.Println("1")
defer fmt.Println("2")
defer fmt.Println("3")
// prints 3, 2, 1
```

**Rule 2 — arguments are evaluated immediately (a photo), but the call happens later:**

```go
x := 1
defer fmt.Println("value:", x) // x is read NOW: 1 is stored
x = 2
// prints: value: 1
```

If you defer a closure instead, it looks through its window at the END (Unit 1.10):

```go
x := 1
defer func() { fmt.Println("value:", x) }() // reads x when it runs
x = 2
// prints: value: 2
```

Note the `()` at the end: `defer` needs a function CALL, so the function literal is called.

**Rule 3 — deferred calls run after `return` sets the results, and can change NAMED results:**

```go
func double() (n int) {
	defer func() { n *= 2 }()
	return 5 // step 1: n = 5; step 2: defers run (n = 10); step 3: function returns 10
}
```

With an unnamed result, the deferred closure can only change a local variable, not the value already chosen for return.

**Rule 3 — the courier counter.** Think of a courier counter. Every function that returns a value has a **result box** at the counter. `return` does NOT hand the box to the caller immediately. It happens in 3 steps:

1. **Fill the box:** put the return value in the result box.
2. **Do the sticky notes:** run all deferred calls.
3. **Hand over the box:** give the caller whatever is in the box NOW.

Named result `(n int)` = the box has a label, `n`. A deferred closure can reach the box by its name and change what is inside during step 2. Unnamed result `int` = the box has no label. `return n` COPIES the value of the local variable `n` into the unlabelled box in step 1; in step 2 the closure changes the local `n`, but the box already holds the copy.

|  | `func f() (n int)` named | `func g() int` unnamed, local n |
| --- | --- | --- |
| Step 1 | `return 5` puts 5 in box `n` | `return n` copies 5 into the hidden box |
| Step 2 | defer: `n *= 2` changes the BOX to 10 | defer: `n *= 2` changes the LOCAL n to 10; box still 5 |
| Step 3 | caller gets 10 | caller gets 5 |

Why it exists: in Chapter 3 a deferred closure can add context to an `err` result after the function decided to return it.


**Rule 4 — defers run during a panic** (before the panic message is printed and the program dies). This is how cleanup still happens when something goes wrong, and where `recover` lives (Chapter 3).

**Rule 5 — `os.Exit` skips ALL defers.** The process ends immediately. This is one reason library code should return errors instead of calling `os.Exit` (P-3).

**Defer in a loop:** each round adds a note, and NONE run until the whole function ends. With resources like open files, they all stay open until then. Fix: move the loop body into its own function so each round's defer runs at the end of that small function.

**Where defer is used:** closing files and network connections, unlocking mutexes (Chapter 7), finishing timers and traces, recovering from panics.

**Common mistakes:**

- Expecting a deferred call to see later changes to its arguments (it took a photo).
- Forgetting the `()` when deferring a function literal.
- Deferring inside a long loop and holding resources too long.
- Expecting defers to run after `os.Exit`.
- Deferring a cleanup BEFORE checking whether the resource was actually obtained (P-13).

### Deep & Tricky

**DT-88. `defer` takes a photo of the arguments, and of the function value.** *(Guarantee)* At the `defer` line, Go evaluates the function value and all arguments and stores them; only the call is postponed. So `defer fmt.Println(x)` prints the old `x`. The same rule applies to method receivers: `defer t.Stop()` uses the `t` that existed at the defer line.

**DT-89. Deferring a nil function panics LATER, not at the defer line.** *(Guarantee)* `var f func(); defer f()` does not fail when the `defer` statement runs. It panics when the function is finishing and Go tries to call `f`. The spec says this explicitly. The panic appears far from the line that caused it, which makes it confusing to debug.

**DT-90. Deferred calls run before the panic message is printed.** *(Runtime behaviour)* When a panic is not recovered, Go first runs every deferred call in every function on the way up the stack, and only then prints `panic: ...` and the stack trace. So output from deferred calls appears ABOVE the panic message.

**DT-91. Defer is almost free now (usually).** *(Implementation detail, since Go 1.14)* In most functions, the compiler "open-codes" defers: it inserts the deferred calls directly at each exit, costing only a few nanoseconds. Defers inside loops (and functions with very many defers) cannot be open-coded and use a slower path. So use defer freely for clarity, but avoid it in very hot loops (measure first, Chapter 10).

### Pattern

**P-13. Acquire, check, then defer the cleanup.** *(Universal Go idiom)*

```go
f, err := os.Open(path)
if err != nil {
	return err      // nothing was opened, so nothing to close
}
defer f.Close()     // right after success: impossible to forget later
```

- *Problem it solves:* forgetting to release resources (files, locks, connections) on one of many return paths, including panics.
- *Why Go developers do it:* the cleanup sits right next to the acquisition, so a reader sees both together, and every exit path is covered automatically.
- *Order matters:* check the error FIRST. Deferring `f.Close()` before the check could call Close on a nil or invalid value.
- *Caveat:* for files you WRITE, the error from `Close` can mean data was not saved; ignoring it with a plain `defer f.Close()` can hide data loss. Chapter 5 shows how to handle it.
- *When not to:* inside a long loop (see defer in loops), and in extremely hot code paths after measuring.

## Topic 1E — Talking to the outside world

## Unit 1.12 — Reading input

**One-line summary:** A program can get input from command-line arguments (`os.Args`), from standard input (stdin: the keyboard or a pipe), or from files (Chapter 5). For stdin, the reliable Go way is: read one line at a time with `bufio.Scanner`, clean it, convert it with `strconv`, and validate it.

**Mental model — a conveyor belt.** stdin is a conveyor belt of bytes coming into your program. When you type, the terminal puts each finished line on the belt when you press Enter. When you pipe (`printf "5\n7\n" | ./prog`), another program fills the belt. When the belt is empty and the sender says "no more" (Ctrl+D on Mac/Linux, or the end of a pipe or file), that is **EOF** (end of file).

**The three input sources:**

| Source | How to send | How to read | Good for |
| --- | --- | --- | --- |
| Arguments | `./prog 42 abc` | `os.Args` | a few values known at start |
| stdin | typing, `cmd \| ./prog`, `./prog < file` | `bufio.Scanner`, `fmt.Scan` | interactive input, streams of data |
| Files | file path | `os.Open` (Chapter 5) | stored data |

**The standard pattern — line by line with bufio.Scanner:**

```go
scanner := bufio.NewScanner(os.Stdin)
for scanner.Scan() {                    // false at EOF or on error
	line := strings.TrimSpace(scanner.Text())
	n, err := strconv.Atoi(line)
	if err != nil {
		fmt.Fprintf(os.Stderr, "not a number: %q\n", line)
		continue
	}
	// use n
}
if err := scanner.Err(); err != nil {   // was it EOF, or a real error?
	fmt.Fprintln(os.Stderr, "read error:", err)
	os.Exit(1)
}
```

- `scanner.Scan()` reads the next line and returns `true`; it returns `false` at EOF or on an error.
- `scanner.Text()` gives the line WITHOUT the newline at the end.
- After the loop, `scanner.Err()` is `nil` for a normal EOF, or the real error otherwise. Always check it.
- `strings.TrimSpace` removes spaces, tabs, and newlines at both ends, so " 5 " becomes "5".

**`fmt.Scan` (quick but tricky):** `fmt.Scan(&n)` reads the next space-separated value into `n`. The `&` means "the address of n" so Scan knows where to store the value (pointers: Chapter 2). It returns `(count, error)`. Problems: invalid input leaves `n` as 0 and returns an error that is easy to ignore; the bad text can stay on the belt; and it treats newlines like spaces, so it does not work line by line. Fine for quick experiments; use `bufio.Scanner` for real programs.

**Prompting:** use `fmt.Print("Enter age: ")` (no newline) so the cursor stays on the same line.

**Testing input programs without typing:** pipe fixed input in: `printf "10\nabc\n5\n" | ./prog`. Same input every time = repeatable tests (Chapter 4).

**Common mistakes:**

- Ignoring the error from `fmt.Scan` or `strconv.Atoi`.
- Forgetting `scanner.Err()` after the loop.
- Not trimming spaces before converting.
- Not handling EOF (the user presses Ctrl+D, or the pipe ends) in an "ask until valid" loop, which then loops forever or crashes.
- Using `fmt.Scan` for line-based input.

### Deep & Tricky

**DT-92. bufio.Scanner has a 64 KB line limit by default.** *(Standard-library behaviour)* By default, a single line (token) longer than 64 KB (`bufio.MaxScanTokenSize`) makes `Scan()` return false, and `scanner.Err()` returns `bufio.ErrTooLong`. If you forget to check `Err()`, the program silently stops reading in the middle of the input. Raise the limit with `scanner.Buffer(make([]byte, 0, 1024*1024), 10*1024*1024)` when lines can be long (log files, JSON lines).

**DT-93. Scanner removes Windows line endings for you.** *(Standard-library behaviour)* Windows text files end lines with `\r\n`. The default line splitter (`bufio.ScanLines`) drops a trailing `\r`, so `scanner.Text()` is the same on all systems. Code that splits text by hand on `"\n"` keeps the `\r`, and then `strconv.Atoi("42\r")` fails mysteriously.

**DT-94. fmt.Scan treats newlines as spaces; fmt.Scanln does not.** *(fmt behaviour)* `fmt.Scan(&a, &b)` happily reads `a` from one line and `b` from the next line. `fmt.Scanln` stops at the end of the line and returns an error if there are not enough values on it. Mixing these with each other, or with `bufio.Scanner` on the same stdin, gives confusing results because each may have read ahead and buffered input the other never sees.

### Pattern

**P-14. Keep `main` tiny: `func run() error` + one `os.Exit`.** *(Common Go pattern for programs)*

```go
func main() {
	if err := run(); err != nil {
		fmt.Fprintln(os.Stderr, "error:", err)
		os.Exit(1)
	}
}

func run() error {
	defer fmt.Println("cleanup") // runs when run() returns, BEFORE os.Exit
	// ... all the real work, returning errors instead of exiting
	return nil
}
```

- *Problem it solves:* `os.Exit` skips deferred calls (Rule 5), so cleanup in `main` is lost on error exits.
- *How it works:* all work and all defers live in `run()`. `run()` returns normally (so its defers run), and only then does `main` call `os.Exit`.
- *Extra benefit:* `run()` can be called from tests (Chapter 4), while `os.Exit` cannot be tested easily. Arguments and I/O can be passed in as parameters later to make it fully testable.
- *When not to:* tiny throwaway scripts.

## Unit 1.13 — Random numbers

**One-line summary:** Computers follow exact instructions, so normal "random" numbers are pseudo-random: a formula produces a long sequence that looks random, starting from a seed. Use `math/rand/v2` for games, simulations, and shuffling; use `crypto/rand` for anything an attacker could benefit from guessing.

**Mental model — a giant pre-shuffled deck.** Imagine a deck of billions of cards, shuffled once in a fixed way. A pseudo-random generator deals cards from this deck. The **seed** decides where in the deck you start. Same seed → same cards in the same order, every time. A different seed → a different starting point, so the numbers look different.

**math/rand/v2 (Go 1.22+):**

```go
import "math/rand/v2"

rand.IntN(6)       // 0, 1, 2, 3, 4, or 5   (never 6!)
rand.IntN(6) + 1   // dice: 1 to 6
rand.Float64()     // 0.0 up to (not including) 1.0
```

- The global functions are seeded randomly by Go at start, so each run gives different numbers. No `Seed` call needed.
- A number from `min` to `max` (both included): `min + rand.IntN(max-min+1)`.
- `rand.IntN(n)` panics if `n <= 0`.

**Reproducible randomness (for tests and debugging):** make your own generator with a fixed seed.

```go
r := rand.New(rand.NewPCG(42, 1024)) // fixed seed
fmt.Println(r.IntN(100), r.IntN(100), r.IntN(100)) // same three numbers on every run
```

A test can then expect exact results, and a bug seen once can be reproduced.

**crypto/rand — for secrets:** reads unpredictable randomness from the operating system. Use it for passwords, OTPs, session IDs, password-reset links, API keys.

```go
import "crypto/rand"

token := rand.Text() // Go 1.24+: a random string, suitable for secret tokens
```

**Which one?**

| Need | Use | Why |
| --- | --- | --- |
| Game dice, shuffling cards, simulations | `math/rand/v2` | fast; predictability does not hurt anyone |
| Retry "jitter" (spreading out retries, Chapter 12) | `math/rand/v2` | fast; not a secret |
| Repeatable test data | `math/rand/v2` with a fixed seed | same sequence every run |
| OTP, password, session ID, reset link, API key | `crypto/rand` | an attacker must not be able to predict it |

If both packages are needed in one file, import one under another name: `import crand "crypto/rand"`.

**Common mistakes:**

- Off-by-one: `rand.IntN(6)` never returns 6.
- Using `math/rand` for tokens or OTPs.
- Expecting a fixed-seed generator to give different numbers on each run (it will not).
- Calling `rand.IntN(0)` (panic).

### Deep & Tricky

**DT-95. Old math/rand gave the SAME numbers every run (before Go 1.20).** *(History; version-dependent)* In the original `math/rand`, the global generator started from seed 1 unless you called `rand.Seed`. Every run of a program produced exactly the same "random" sequence, which surprised countless beginners (and created predictable values in real software). Since Go 1.20 the global generator is seeded randomly, `rand.Seed` is deprecated, and `math/rand/v2` (Go 1.22) is the modern package. Old blog posts and Stack Overflow answers still show `rand.Seed(time.Now().UnixNano())`; you do not need it.

**DT-96. Modulo bias: why `rand.IntN` exists.** *(Maths + library design)* Turning a random 0–255 value into a dice roll with `% 6` is slightly unfair: 256 is not a multiple of 6, so the remainders 0–3 appear 43 times each but 4–5 only 42 times. With huge ranges the bias is tiny, but it is real. `rand.IntN(n)` uses a method that gives every value exactly the same chance. Prefer it over `% n` on raw random numbers.

**DT-97. crypto/rand.Read no longer returns an error (Go 1.24+).** *(Version-dependent)* Since Go 1.24, `crypto/rand.Read` is guaranteed to fill the buffer; if the operating system cannot provide randomness, the program crashes instead of returning an error (continuing with weak randomness would be worse). Older code checks its error; that check is now unnecessary but harmless.

### Pattern

**P-15. Secrets come from crypto/rand, always.** *(Security practice)*

- *Problem it solves:* `math/rand` output can be predicted by someone who sees enough of it or can guess the seed. Predictable OTPs, session IDs, or reset links let attackers take over accounts.
- *How:* anything that grants access or must stay secret: `crypto/rand`. Everything else (games, simulations, jitter, sampling): `math/rand/v2`.
- *Trade-off:* `crypto/rand` is slower, which almost never matters for secrets because you generate few of them.

## Chapter 1 Project — `playtool`

**Goal:** one command-line program with two modes that uses everything in this chapter: constants, operators, floats, `if`/`switch`, loops, functions, closures or small helpers, `defer`, input, and randomness — plus the Chapter 0 conventions (stderr, exit codes, validation at the edge).

**Modes**

- `playtool guess [seed]` — guess a secret number from 1 to 100 in at most 7 attempts. An optional seed makes the secret repeatable for testing.
- `playtool convert` — read lines like `10 km mi`, `100 c f`, `5 kg lb` (and reverse directions) until `quit` or EOF.

**New tools**

- `strings.Fields(line)` splits a line on whitespace; `strings.TrimSpace` removes surrounding whitespace.
- `strconv.ParseFloat(text, 64)` and `strconv.ParseUint(text, 10, 64)` convert text, returning an error on bad input.
- `errors.New("...")` and `fmt.Errorf("... %w", err)` create error values that `run()` returns (full lesson in Chapter 3).

**Exact constants:** 1 mile = 1.609344 km; 1 pound = 0.45359237 kg; °F = °C × 9 / 5 + 32.

**Requirements**

- *G1* No mode or unknown mode → usage on stderr, exit 1. *G2* Secret in [1, 100], from `math/rand/v2`; with a seed, from a fixed-seed generator. *G3* Invalid seed → error that keeps the original reason, exit 1. *G4* After each guess print "too low", "too high", or "correct". *G5* A rejected line (not a number, out of range) prints the reason on stderr and does **not** use up an attempt. *G6* On a win, print the attempts used and a rating. *G7* Out of attempts → reveal the number, exit 1. *G8* EOF during the game → "game aborted", exit 1; a real read error is reported differently.
- *C1* Six conversions, all exact constants; `−40 c f` must give −40 (watch `9/5`). *C2* Wrong word count, bad number, or unknown pair → reason on stderr, keep going. *C3* `quit` or EOF ends with "bye", exit 0. *C4* Check `scanner.Err()`.
- *P1* `main` only calls `run()` and exits (P-14). *P2* No magic numbers: `minNumber`, `maxNumber`, `maxAttempts`. *P3* Each problem reported once. *P4* `gofmt`, `go vet` clean.

**Test plan** (run every line; record real output and `echo $?`):

```
./playtool; echo $?                                  # usage, 1
./playtool fly; echo $?                              # unknown mode, 1
./playtool guess x; echo $?                          # invalid seed with reason, 1
printf "50\n75\n62\n" | ./playtool guess 42; echo $? # win in 3 (with this solution's seeding), 0
printf "1\n2\nabc\n3\n4\n5\n6\n7\n" | ./playtool guess 42; echo $?   # abc not counted; out of attempts, 1
printf "50\n" | ./playtool guess 42; echo $?         # EOF: aborted, 1
printf "10 km mi\n-40 c f\nabc\nquit\n" | ./playtool convert; echo $?  # 0
```

**Reflection questions**

1. Why are 7 attempts always enough for 1–100 with the best strategy? (Hint: 2^7.)
2. What does a fixed seed give you in testing that a random seed cannot?
3. Which package would you use for the secret in a game, and which for a password-reset link? Why?

**Reference solution**

```go
// Command playtool is a number-guessing game and a unit converter.
//
//	playtool guess [seed]   guess a number from 1 to 100 in at most 7 attempts
//	playtool convert        convert lines like "10 km mi" until "quit" or EOF
package main

import (
	"bufio"
	"errors"
	"fmt"
	"math/rand/v2"
	"os"
	"strconv"
	"strings"
)

const usage = "usage: playtool guess [seed] | playtool convert"

const (
	minNumber   = 1
	maxNumber   = 100
	maxAttempts = 7
)

const (
	kmPerMi = 1.609344
	kgPerLb = 0.45359237
)

func main() {
	if err := run(os.Args[1:]); err != nil {
		fmt.Fprintln(os.Stderr, "error:", err)
		os.Exit(1)
	}
}

// run checks the arguments and dispatches to a mode. It never calls os.Exit,
// so every deferred call inside it runs (P-14).
func run(args []string) error {
	if len(args) == 0 {
		return errors.New(usage)
	}
	switch args[0] {
	case "guess":
		if len(args) > 2 {
			return errors.New(usage)
		}
		seedArg := ""
		if len(args) == 2 {
			seedArg = args[1]
		}
		secret, err := newSecret(seedArg)
		if err != nil {
			return err
		}
		return playGuess(bufio.NewScanner(os.Stdin), secret)
	case "convert":
		if len(args) != 1 {
			return errors.New(usage)
		}
		return runConvert(bufio.NewScanner(os.Stdin))
	default:
		return fmt.Errorf("unknown mode %q\n%s", args[0], usage)
	}
}

// newSecret returns a random number in [minNumber, maxNumber]. With a seed,
// the number is the same on every run, which makes the game testable.
func newSecret(seedArg string) (int, error) {
	span := maxNumber - minNumber + 1
	if seedArg == "" {
		return minNumber + rand.IntN(span), nil
	}
	seed, err := strconv.ParseUint(seedArg, 10, 64)
	if err != nil {
		return 0, fmt.Errorf("invalid seed %q: %w", seedArg, err)
	}
	r := rand.New(rand.NewPCG(seed, seed))
	return minNumber + r.IntN(span), nil
}

func playGuess(sc *bufio.Scanner, secret int) error {
	fmt.Printf("I picked a number from %d to %d. You have %d attempts.\n",
		minNumber, maxNumber, maxAttempts)
	attempt := 1
	for attempt <= maxAttempts {
		fmt.Printf("attempt %d/%d: ", attempt, maxAttempts)
		if !sc.Scan() {
			if err := sc.Err(); err != nil {
				return fmt.Errorf("reading input: %w", err)
			}
			return errors.New("game aborted: input ended")
		}
		text := strings.TrimSpace(sc.Text())
		guess, err := strconv.Atoi(text)
		if err != nil || guess < minNumber || guess > maxNumber {
			// A rejected line does not use up an attempt.
			fmt.Fprintf(os.Stderr, "please enter a whole number from %d to %d (got %q)\n",
				minNumber, maxNumber, text)
			continue
		}
		switch {
		case guess < secret:
			fmt.Println("too low")
		case guess > secret:
			fmt.Println("too high")
		default:
			fmt.Printf("correct! %d attempt(s): %s\n", attempt, rating(attempt))
			return nil
		}
		attempt++
	}
	return fmt.Errorf("out of attempts, the number was %d", secret)
}

func rating(attempts int) string {
	switch {
	case attempts <= 3:
		return "well played"
	case attempts <= 5:
		return "good"
	default:
		return "just made it"
	}
}

func runConvert(sc *bufio.Scanner) error {
	fmt.Println(`enter "<value> <from> <to>" (km mi kg lb c f), or "quit"`)
	for sc.Scan() {
		line := strings.TrimSpace(sc.Text())
		if line == "" {
			continue
		}
		if line == "quit" {
			break
		}
		fields := strings.Fields(line)
		if len(fields) != 3 {
			fmt.Fprintf(os.Stderr, "need 3 words, got %d: %q\n", len(fields), line)
			continue
		}
		v, err := strconv.ParseFloat(fields[0], 64)
		if err != nil {
			fmt.Fprintf(os.Stderr, "not a number: %q\n", fields[0])
			continue
		}
		out, ok := convert(v, fields[1], fields[2])
		if !ok {
			fmt.Fprintf(os.Stderr, "unknown conversion %s -> %s\n", fields[1], fields[2])
			continue
		}
		fmt.Printf("%g %s = %.4f %s\n", v, fields[1], out, fields[2])
	}
	if err := sc.Err(); err != nil {
		return fmt.Errorf("reading input: %w", err)
	}
	fmt.Println("bye")
	return nil
}

// convert reports ok == false for an unknown pair of units.
func convert(v float64, from, to string) (float64, bool) {
	switch from + ">" + to {
	case "km>mi":
		return v / kmPerMi, true
	case "mi>km":
		return v * kmPerMi, true
	case "kg>lb":
		return v / kgPerLb, true
	case "lb>kg":
		return v * kgPerLb, true
	case "c>f":
		return v*9/5 + 32, true
	case "f>c":
		return (v - 32) * 5 / 9, true
	}
	return 0, false
}
```

Note `v*9/5 + 32`: operators of equal precedence go left to right, so this is `(v*9)/5`. `v*9` is a float64 multiplication and there is no integer trap. Writing `9/5*v + 32` instead computes `9/5` first, and both are untyped *integer* constants, so `9/5` is exactly 1 (DT-54): −40 °C would become −8 °F. That is why the −40 test is in the requirements: −40 is the one temperature that is equal in both scales, so any wrong formula shows up immediately.

**Review checklist** (issues reviewers find most often)

1. `scanner.Err()` never checked: a read error or a line over 64 KB (DT-92) is reported as "bye" with exit 0.
2. The same problem reported twice: a message printed on stderr *and* an error returned and printed by `main`. One problem, one report.
3. Magic numbers: `100` and `7` repeated in the welcome text, the range check, the loop, and the secret.
4. `run()` doing everything: move seed parsing into `newSecret(seedArg string) (int, error)`.
5. Original errors thrown away: `errors.New("bad seed")` hides *why*; `fmt.Errorf("invalid seed %q: %w", s, err)` keeps it.
6. Exported names in `package main` (`KmPerMi`): nothing imports `main`, so use `kmPerMi`.
7. `rand.IntN(maxNumber) + minNumber` is only right while `minNumber` is 1. Once values are named constants, every formula must be correct for *any* value of them: `minNumber + rand.IntN(maxNumber-minNumber+1)`.
8. The test plan and reflection answers are part of the deliverable. The path you did not run (often the *winning* path) is where the bug hides.

**Reflection answers:** (1) Each guess halves the remaining range; 2^7 = 128 ≥ 100, so 7 halvings always isolate one number. (2) Determinism: the same input gives the same output every run, so tests can assert exact results and bugs can be reproduced. (3) `math/rand/v2` for the game (predictability harms no one); `crypto/rand` for the reset link (an attacker must not be able to predict it, P-15).

## Answers to the Chapter 1 prediction questions

**1.1 Q2.** (a) `2 11` (right side evaluated with old values, DT-46) (b) `float64` (c) `int32 97` (d) `""`.

**1.2 Q2.** (a) `98` (b) compiles, i = 2 (c) compile error: truncated (d) `4` (e) A = 0, B = 10, D = 30.

**1.3 Q2.** (a) −3 (b) −1 (c) 1 (d) `true` (e) `true` (f) 3 (prints `3`).

**1.4 Q2.** (a) `10000010` (b) `11101110` (c) `01101100` (d) `01001000` (e) `00110101` (f) 5 (g) 1 (XOR).

**1.5 Q2.** (a) −3 (b) −3 (c) −3 (d) `0.9999999999999999` (e) `1e+06` (f) `0.10000000149011612`.

**1.6 Q2.** Grade for "pass" (the first true case wins). Put the strictest condition first: 90, then 75, then 35. **Q3.** `two` then `hundred`.

**1.7 Q2.** (a) `0 3 6 9` (b) no — `break` leaves only the switch (DT-68) (c) (0,0) (0,1) (0,2) (1,0) (1,1) (1,2) — at (1,2) `i*j == 2`, `break outer` ends both loops.

**1.8 Q2.** (a) 5 (b) `3 2` (c) compile error: assignment mismatch (d) panic: nil pointer dereference.

**1.8 Q4.** The iterative version. Each recursive call adds a frame; ten million frames exceed the 1 GB stack limit: `fatal error: stack overflow` (DT-39, DT-76).

**1.9 Q2.** `in loop: 10`, `in loop: 11`, `in loop: 12`, `after: 10`. **Q3.** Compiles and prints `false` (DT-80).

**1.10 Q2.** `3` then `100`. **Q3.** `go 1.21`: 3 3 3; `go 1.22`+: 0 1 2.

**1.11 Q2.** a() = 11, b() = 10, c() = 30. **Q3.** `B: 2` then `A: 1` (LIFO; A took a photo, B looks through a window).

**1.12 Q2.** `n` stays 0, an error is returned, and `abc` may remain unread on stdin, so the next `Scan` can see it again.

**1.13 Q3.** No; it returns 0–5. `rand.IntN(0)` panics. `10 + rand.IntN(91)` gives 10 to 100 inclusive.

**2.1 Q2.** (a) 1 (b) 6 (c) `true` (d) run-time panic: index out of range [3] with length 3. **Q3.** `3` — range iterates over a copy made before the loop started.

# Chapter 2 — Composite Data & Memory

*Goal: master arrays, slices, strings, maps, structs, and pointers — the places where most real Go bugs live.*

Chapter 1 worked with single values: one int, one float, one string. Real programs work with *collections* and *records*: a list of orders, a map from user ID to session, a struct describing a shipment. Go gives you a small set of composite types, and each one has a precise memory story. Once you can draw that story (what is copied, what is shared, where the bytes are), the "mysterious" bugs — an `append` that changes another slice, a map that panics, a string that `len` says is 18 characters long — stop being mysterious.

**What you will be able to do after this chapter**

- Draw the slice header and backing array for any slicing or `append` operation and predict aliasing.
- Process text correctly in any language: bytes vs runes vs what a human calls a character.
- Use maps idiomatically (comma-ok, sets, deterministic output) and know their limits.
- Design structs with sensible memory layout, and choose between value and pointer semantics with a reason.

**The central question of this chapter:** *"Is this a copy, or is it shared?"* Every composite type answers it differently:

| Type | Assignment / passing copies… | So the original data is… |
| --- | --- | --- |
| Array `[N]T` | all N elements | never shared |
| Struct | all fields (shallowly) | never shared, except through fields that are themselves pointers, slices, or maps |
| Slice `[]T` | a 3-word header (pointer, len, cap) | **shared** (the backing array) |
| String | a 2-word header (pointer, len) | shared, but immutable, so sharing is safe |
| Map | one pointer to the runtime's map | **shared** |
| Pointer `*T` | the address | **shared** |

Keep this table in mind; the rest of the chapter explains each row.

## Topic 2A — Arrays and slices

## Unit 2.1 — Arrays

**One-line summary:** An array is a fixed number of values of one type, stored side by side in memory. The length is part of the type, and arrays are VALUES: assigning or passing an array copies every element.

**Mental model — a steel tiffin carrier with a fixed number of boxes.** A 3-tier tiffin always has exactly 3 boxes, stacked together, numbered from 0. You cannot add a 4th tier; a 4-tier tiffin is a different product. Giving someone your tiffin (assignment, function call) means giving them a full COPY of all boxes and their food.

**Declaring arrays:**

```go
var a [3]int                 // [0 0 0]  (zero values)
b := [3]int{10, 20, 30}      // [10 20 30]
c := [...]int{1, 2, 3, 4}    // compiler counts: type is [4]int
d := [5]int{2: 7, 4: 9}      // [0 0 7 0 9]  (index: value)
```

**Indexing:** `a[0]` is the first element, `a[len(a)-1]` the last. `len(a)` is the length, known at compile time.

**Out-of-range index — "who calculates?" again:**

| Code | Result |
| --- | --- |
| `a[3] = 1` (constant index, length 3) | compile error: index 3 out of bounds |
| `i := 3; a[i] = 1` | run-time panic: `index out of range [3] with length 3` |

Go checks every index (bounds checking), so it can never read or write memory outside the array (unlike C, where this causes crashes and security holes).

**The length is part of the type:** `[3]int` and `[4]int` are different types. `var p [3]int; var q [4]int; p = q` is a type error.

**Arrays are values (copied):**

```go
a := [3]int{1, 2, 3}
b := a        // b is a full COPY
b[0] = 99
fmt.Println(a, b) // [1 2 3] [99 2 3]
```

Passing an array to a function also copies it (DT-75). Arrays of comparable elements can be compared with `==` (element by element) and used as map keys.

**Memory layout:** elements sit next to each other with no gaps between same-size elements. `[4]int64` is exactly 32 bytes. Walking through an array in order is very cache-friendly (Unit 0.5).

**Iterating:**

```go
for i := range a { ... }     // index only
for i, v := range a { ... }  // index and a COPY of the element
for _, v := range a { ... }  // value only
```

Changing `v` does not change the array; write `a[i] = ...` instead.

**Multi-dimensional arrays:** `var board [3][3]int` is an array of 3 arrays of 3 ints (a tic-tac-toe board). `board[1][2]` is row 1, column 2.

**Where arrays are used in real Go:** fixed-size values where copying is fine or wanted: hashes (`sha256.Sum256` returns `[32]byte`), IPv4 addresses (4 bytes), small matrices, game boards, map keys made of several numbers. For lists whose size changes, Go uses SLICES (Unit 2.2), which are built on top of arrays.

**Common mistakes:**

- Expecting `b := a` to share data (it copies).
- Expecting a function to change the caller's array.
- Changing `v` in a range loop and expecting the array to change.
- Mixing arrays of different lengths.
- Using huge arrays as function parameters (every call copies all of it).

### Deep & Tricky

**DT-98. `range` over an array iterates over a COPY of the array.** *(Guarantee)* The range expression is evaluated once before the loop starts. For an array, that means the whole array is copied when you use the value variable. So changing `arr[2]` inside `for i, v := range arr` does not change the `v` values you see later in the same loop. Ranging over `&arr` (a pointer, Chapter 2) or a slice avoids the copy and sees the changes.

**DT-99. Constant indexes are checked at compile time.** *(Guarantee)* `a[5]` on a `[3]int` with a constant 5 does not compile. With a variable index, the compiler inserts a bounds check that panics at run time. When the compiler can prove an index is always valid (for example `for i := range a { a[i] }`), it removes the check: "bounds-check elimination" (Chapter 10).

**DT-100. Arrays are comparable and can be map keys; slices cannot.** *(Guarantee)* `[3]int{1, 2, 3} == [3]int{1, 2, 3}` is true. This makes arrays useful as composite map keys, for example `map[[2]int]bool` for visited grid cells (useful in Chapter 6 and in routing). Slices cannot be compared with `==` (only with nil) and cannot be map keys.

### Pattern

**P-16. Arrays for fixed-size values, slices for lists.** *(Go convention)*

- *Use arrays when:* the size is part of the meaning and never changes (a 32-byte hash, a 4-byte IPv4 address, RGB colour `[3]uint8`, a 3×3 board), you want value semantics (a copy is a separate thing), or you need a comparable map key.
- *Use slices when:* the number of elements can change or is known only at run time (almost every "list" in Go code).
- *Trade-off:* arrays are copied completely on assignment and function calls; that is cheap for small arrays and expensive for big ones.

## Unit 2.2 — Slices I: the slice header

**One-line summary:** A slice is a small descriptor — a pointer to an element of an array, a length, and a capacity — that gives you a *window* onto that array. Copying a slice copies the descriptor, not the elements, so two slices can look at the same memory.

**Mental model — a window onto a long shelf.** Picture a long shelf of numbered jars (the **backing array**). A slice is a cardboard frame you hold up in front of the shelf. The frame records three things: which jar it starts at (the **pointer**), how many jars you can see through it (the **length**), and how many jars there are from its start to the end of the shelf (the **capacity**). Two people can hold two frames over overlapping parts of the same shelf: if one of them changes a jar, the other sees it.

```
backing array:  [ 10 | 20 | 30 | 40 | 50 ]
index:             0    1    2    3    4

s := arr[1:3]   ptr ──────┐
                len = 2   ▼
                cap = 4  [ 20 | 30 ] 40   50     (40 and 50 are "beyond len, within cap")
```

### The slice header

In the current implementation a slice value is exactly three machine words:

| Field | Meaning | On 64-bit |
| --- | --- | --- |
| `ptr` | address of the first element the slice can see | 8 bytes |
| `len` | how many elements you may index: `s[0]` to `s[len-1]` | 8 bytes |
| `cap` | how many elements exist from `ptr` to the end of the backing array | 8 bytes |

`unsafe.Sizeof(s)` is 24 on every 64-bit platform, whether the slice holds 3 elements or 3 million. This is why passing a slice to a function is cheap: only 24 bytes are copied.

### Making slices

```go
a := []int{10, 20, 30}        // slice literal: Go creates a [3]int and a slice over it
b := make([]int, 3)           // len 3, cap 3, all zero: [0 0 0]
c := make([]int, 0, 100)      // len 0, cap 100: room for 100 appends without moving
var d []int                   // nil slice: ptr = nil, len 0, cap 0
arr := [5]int{10, 20, 30, 40, 50}
e := arr[1:3]                 // a slice over an existing array: [20 30]
```

Note the difference from arrays: `[3]int{...}` (length inside the brackets) is an array; `[]int{...}` (empty brackets) is a slice.

### Slicing: `s[lo:hi]`

`s[lo:hi]` makes a new header that starts at element `lo` and stops *before* `hi`:

- new `len` = `hi − lo`
- new `cap` = old `cap − lo`
- new `ptr` = address of element `lo`

Defaults: `s[:hi]` means `s[0:hi]`; `s[lo:]` means `s[lo:len(s)]`; `s[:]` is the whole slice.

| Starting from `arr := [5]int{10, 20, 30, 40, 50}` | Elements | len | cap |
| --- | --- | --- | --- |
| `arr[:]` | 10 20 30 40 50 | 5 | 5 |
| `arr[1:3]` | 20 30 | 2 | 4 |
| `arr[2:]` | 30 40 50 | 3 | 3 |
| `arr[:0]` | (none) | 0 | 5 |
| `arr[4:5]` | 50 | 1 | 1 |

**The bounds rule:** `0 <= lo <= hi <= cap(s)`. Note: `hi` may go up to the *capacity*, not just the length. So you can **re-slice upwards** into the hidden part:

```go
s := make([]int, 2, 5) // [0 0], cap 5
s = s[:4]              // allowed: [0 0 0 0] — reveals elements that were beyond len
// s[:6] would panic: slice bounds out of range [:6] with capacity 5
```

Indexing (`s[i]`) is checked against `len`; slicing (`s[lo:hi]`) is checked against `cap`. Both panic at run time when out of range.

### Aliasing: two slices, one array

Because slicing never copies elements, slices made from the same array share them:

```go
arr := [5]int{10, 20, 30, 40, 50}
x := arr[0:3] // [10 20 30]
y := arr[2:5] // [30 40 50]
y[0] = 99
fmt.Println(arr, x, y) // [10 20 99 40 50] [10 20 99] [99 40 50]
```

One write through `y` changed what `arr` and `x` see, because `x[2]`, `y[0]`, and `arr[2]` are the same memory. This is called **aliasing**. It is a feature (cheap views, no copying) and a classic source of bugs (an unexpected change "from nowhere").

### Slices are not comparable

`s1 == s2` is a compile error for slices; the only allowed comparison is with `nil`. (Should `==` compare headers or elements? Either answer surprises someone, so Go allows neither.) Compare elements with `slices.Equal(s1, s2)` (Unit 2.4).

### Deep & Tricky

**DT-101. A slice expression on an array makes the array escape — or not.** *(Implementation detail)* `arr[:]` creates a pointer into `arr`. If that slice is returned or stored somewhere long-lived, `arr` must outlive the function, so escape analysis moves it to the heap (Unit 0.6). A slice used only locally lets the array stay on the stack. `-gcflags=-m` shows the decision.

**DT-102. `len` and `cap` of a nil slice are 0, and you can range over it, append to it, and slice it with `[:0]`.** *(Guarantee)* A nil slice behaves exactly like an empty slice for reading. You almost never need to check `s == nil` before using a slice; check `len(s) == 0` instead, which covers both.

**DT-103. Re-slicing up to capacity can expose "deleted" data.** *(Guarantee + security consequence)* If a function shortens a slice with `s = s[:n]` to "remove" secret bytes, the bytes are still in the backing array, and anyone holding the slice can bring them back with `s[:cap(s)]`. To really erase data, overwrite it (`clear(s[n:])`, Unit 2.4) before shortening, or copy the part you keep into a fresh slice.

**DT-104. Indexing is checked against `len`, slicing against `cap`.** *(Guarantee)* `s[3]` with `len(s) == 3` panics even if `cap(s)` is 10; `s[:10]` with the same slice is fine. The error messages say which limit was hit: `index out of range [3] with length 3` vs `slice bounds out of range [:11] with capacity 10`.

### Common mistakes

- Thinking `b := a` for slices copies the elements. It copies the header; both share the array.
- Forgetting that a sub-slice keeps the *whole* backing array alive (see Unit 2.4, memory leaks).
- Confusing `[3]int{}` (array) with `[]int{}` (slice).
- Comparing slices with `==`.

### Mastery check

1. **Explain.** What three things are in a slice header? Why does passing a 1,000,000-element slice to a function cost the same as passing a 3-element slice?
2. **Predict.** With `arr := [6]int{0, 1, 2, 3, 4, 5}` and `s := arr[2:4]`: (a) `len(s), cap(s)` (b) `s[:4]` (c) `s[2]` (d) `s[1:5]` (e) after `s[0] = 9`, what is `arr`?
3. **Write.** Draw (on paper) the shelf-and-frames picture for `x := arr[1:4]; y := x[1:2]; y = y[:3]`, then write the code and print `len`, `cap`, and contents of `x` and `y` to check your drawing.

## Unit 2.3 — Slices II: append, copy, and growth

**One-line summary:** `append` adds elements to a slice. If the backing array has room (len < cap), it writes into that array and the new slice *shares* it with the old one; if not, it allocates a bigger array, copies everything, and the new slice no longer shares anything with the old one. Always use the result of `append`.

**Mental model — the shelf again.** Appending to a slice means "put a jar just after the last visible one, and widen my frame by one". If there is an empty spot on the shelf right there (capacity), the jar goes on the *same shelf* — and anyone else whose frame covers that spot sees it. If the shelf is full, Go fetches a *new, longer shelf*, moves all the jars across, puts the new jar at the end, and gives you a frame on the new shelf. Your old frame (and everyone else's) still looks at the old shelf.

### The rule of `append`

```go
s = append(s, x)            // one element
s = append(s, x, y, z)      // several
s = append(s, other...)     // all elements of another slice
b = append(b, "text"...)    // special case: append a string's bytes to a []byte
```

`append` returns a new slice header. You must store it, usually back into the same variable. `append(s, x)` on its own line is a compile error (`append(s, x) (value of type []int) is not used`), because forgetting the result is always a bug.

What happens inside:

1. If `len(s) + number of new elements <= cap(s)`: write the new elements into the existing backing array right after `len(s)`, and return a header with the larger `len` and the **same** `ptr` and `cap`.
2. Otherwise: allocate a new, larger array; copy the old elements; write the new ones; return a header pointing to the **new** array.

### Growth: how big is the new array?

The exact policy is an implementation detail, but you should know its shape. Appending one element at a time to an empty slice on Go 1.24 gives these capacities:

```
1 2 4 8 16 32 64 128 256 512 848 1280 1792 2560 ...
```

Small slices double; from 256 elements on, growth smoothly decreases towards about 1.25× per step; and the final size is rounded up to one of the memory allocator's **size classes**, which is why you see 848 rather than 640. The doubling is what makes `append` fast *on average*: if each growth doubles the capacity, the total copying over n appends is at most about 2n element copies, so each append costs O(1) **amortized** (Unit 6.1 proves it).

**Pre-size when you know the size:** `make([]T, 0, n)` gives capacity n up front, so n appends never reallocate. This is one of the simplest and most effective Go performance habits.

```go
names := make([]string, 0, len(users))
for _, u := range users {
	names = append(names, u.Name)
}
```

### The aliasing trap

Case 1 — capacity available, so the append writes into *shared* memory:

```go
a := []int{1, 2, 3, 4, 5}
b := a[1:3]          // [2 3], len 2, cap 4
b = append(b, 99)    // room available: writes into a[3]!
fmt.Println(a)       // [1 2 3 99 5]   <- a changed
fmt.Println(b)       // [2 3 99]
```

Case 2 — no capacity, so the append moves to a new array and the link breaks:

```go
a := []int{1, 2, 3}  // len 3, cap 3
b := append(a, 4)    // must grow: new array
b[0] = 100
fmt.Println(a, b)    // [1 2 3] [100 2 3 4]  <- a unchanged
```

The same line of code (`append`) shares memory in one case and not in the other, depending on a number you usually cannot see. That is the trap. The rule that keeps you safe: **after `b := append(a, ...)`, treat `a` as possibly sharing memory with `b`, and do not use both for writing.** If you need independent slices, copy explicitly.

### The full slice expression: `s[lo:hi:max]`

A third index limits the capacity of the new slice: `cap = max − lo`. With capacity equal to length, the next `append` *must* allocate, so it cannot overwrite the original array:

```go
a := []int{1, 2, 3, 4, 5}
c := a[1:3:3]        // [2 3], len 2, cap 2
c = append(c, 77)    // no room: new array
fmt.Println(a, c)    // [1 2 3 4 5] [2 3 77]   <- a is safe
```

Use it when you hand out a sub-slice and do not want the receiver's appends to scribble over your data.

### `copy`

```go
n := copy(dst, src) // copies min(len(dst), len(src)) elements, returns that number
```

- `copy` never grows `dst`; it copies only as many elements as fit in `dst`'s **length** (not capacity). `copy(make([]int, 0, 10), src)` copies nothing.
- It works correctly when `dst` and `src` overlap (like `memmove` in C).
- `copy(b, "string")` copies a string's bytes into a `[]byte`.

The idiomatic ways to make an independent copy:

```go
c1 := make([]int, len(s))
copy(c1, s)

c2 := append([]int(nil), s...) // older idiom
c3 := slices.Clone(s)           // clearest (Go 1.21+)
```

### nil slice vs empty slice

```go
var a []int     // nil:   a == nil is true,  len 0, cap 0
b := []int{}    // empty: b == nil is false, len 0, cap 0
```

For almost everything — `len`, `range`, `append`, printing (`[]` both) — they behave the same. They differ when the difference is *observed*: `encoding/json` encodes a nil slice as `null` and an empty slice as `[]` (Chapter 5), and `reflect.DeepEqual(a, b)` is false. Idiomatic Go: declare with `var s []T` (nil) and let `append` allocate; check emptiness with `len(s) == 0`.

### Passing slices to functions

Go is always pass-by-value (DT-75): the function receives a *copy of the header*. So:

```go
func setFirst(s []int)  { s[0] = 100 }       // visible to the caller: same backing array
func addOne(s []int)    { s = append(s, 1) } // NOT visible: the caller's header still has the old len
func addOneOK(s []int) []int { return append(s, 1) }

nums := make([]int, 3, 10)
setFirst(nums)            // nums[0] == 100
addOne(nums)              // nums is still len 3 (though nums[:4][3] is now 1 — the write happened!)
nums = addOneOK(nums)     // len 4
```

The `addOne` case is subtle: the append *did* write into the shared array (cap was 10), but the caller's header still says `len 3`, so the caller cannot see it. Return the new slice, exactly as `append` itself does. This is why the standard library's functions that grow slices (`append`, `strconv.AppendInt`, `slices.Insert`) all *return* the slice.

### Deep & Tricky

**DT-105. Growth rounds up to size classes.** *(Implementation detail; changes between versions)* After computing the target capacity, the runtime rounds the allocation up to the nearest malloc size class so no space is wasted. For `[]int` that turns 640 elements (5120 bytes) into 848 (6784 bytes) on Go 1.24. So `cap` after a growth is not a number you can rely on — only `cap >= len` is guaranteed.

**DT-106. `append` to a nil slice works; there is no need to `make` first.** *(Guarantee)* `var s []int; s = append(s, 1)` allocates the first array. Pre-sizing with `make` is an optimisation, not a requirement.

**DT-107. A function that appends to a shared slice can corrupt the caller's other data.** *(Guarantee; classic production bug)* A function receives `s[:2]` of a 5-element slice and appends. Because cap is 5, it overwrites elements 2, 3, … of the caller's slice — data the function was never meant to see. Defensive code that hands out sub-slices uses the three-index form `s[:2:2]`.

**DT-108. `copy` copies by length, not capacity.** *(Guarantee)* `dst := make([]int, 0, len(src)); copy(dst, src)` copies zero elements and silently leaves `dst` empty. Use `make([]int, len(src))`.

### Pattern

**P-17. Pre-size slices with `make([]T, 0, n)` when n is known or estimable.** *(Performance idiom)*

- *Problem it solves:* repeated growth in a loop reallocates and copies many times and creates garbage for the GC.
- *How:* `out := make([]T, 0, len(in))`, then `append`. Use `make([]T, n)` plus `out[i] = ...` when you fill every index.
- *When not to:* when n is unknown and could be huge (pre-allocating the worst case wastes memory), or when the slice is tiny and short-lived. Measure first in hot paths (Chapter 10).
- *Trade-off:* a little more code for fewer allocations; over-estimating wastes memory.

**P-18. Functions that may grow a slice return it.** *(Go convention)* `func appendDigits(dst []byte, n int) []byte` — mirror `append`'s shape: accept the slice, return the possibly-new slice, let the caller reassign. Never rely on the caller "seeing" an append.

### Common mistakes

- `append(s, x)` without assigning (compile error) or assigning to a different variable and then using both for writes (aliasing bug).
- Expecting `copy` to grow the destination.
- Appending to a function parameter and expecting the caller to see the new length.
- Assuming a specific capacity after growth.

### Mastery check

1. **Explain.** Why can the same `append` line sometimes change another slice and sometimes not? What decides it?
2. **Predict.**
   ```go
   a := make([]int, 3, 4)
   b := append(a, 1)
   c := append(a, 2)
   fmt.Println(a, b, c)
   ```
3. **Predict.** `s := []int{1, 2, 3}; t := s[:1]; t = append(t, 9); fmt.Println(s)` — then the same with `t := s[:1:1]`.
4. **Write.** A function `appendUnique(s []string, v string) []string` that appends `v` only if it is not already present. Show with `len`/`cap` printing that your caller code reassigns correctly.
5. **Write.** Append 10,000 elements to a nil slice and print every new capacity. Then do it again with `make([]int, 0, 10_000)` and count the allocations with `testing.AllocsPerRun` (preview of Chapter 4).

## Unit 2.4 — Slices III: working with slices

**One-line summary:** Real code needs 2D slices, insert and delete, filtering, sorting, and searching. The `slices` package (Go 1.21+) and the `clear` built-in cover most of it; knowing what they do to the backing array keeps you out of memory leaks.

### 2D slices

A slice of slices: each row is its own slice header, possibly with its own length.

```go
grid := make([][]int, rows)
for i := range grid {
	grid[i] = make([]int, cols)
}
grid[1][2] = 5
```

Each row is a separate allocation and rows can be scattered in memory. For a big, fixed-shape grid, one flat slice with index arithmetic is faster (one allocation, sequential memory — Unit 0.5):

```go
cells := make([]int, rows*cols)
at := func(r, c int) *int { return &cells[r*cols+c] }
*at(1, 2) = 5
```

Compare with arrays: `[3][3]int` is one solid block of 9 ints, copied as a whole; `[][]int` is a header pointing to headers pointing to rows.

### The `slices` package

| Task | Function |
| --- | --- |
| Sort in place | `slices.Sort(s)`; custom order: `slices.SortFunc(s, func(a, b T) int {...})`; stable: `slices.SortStableFunc` |
| Search a sorted slice | `i, found := slices.BinarySearch(s, x)` |
| Linear search | `slices.Contains(s, x)`, `slices.Index(s, x)`, `slices.IndexFunc(s, pred)` |
| Compare | `slices.Equal(a, b)`, `slices.Compare(a, b)` |
| Min / max | `slices.Min(s)`, `slices.Max(s)` (panic on an empty slice) |
| Insert / delete | `s = slices.Insert(s, i, v...)`, `s = slices.Delete(s, i, j)`, `s = slices.DeleteFunc(s, pred)` |
| Copy / grow / combine | `slices.Clone(s)`, `s = slices.Grow(s, n)`, `slices.Concat(a, b, c)` |
| Other | `slices.Reverse(s)`, `s = slices.Compact(s)` (remove adjacent duplicates), `slices.Chunk(s, n)` |

Comparison functions return an `int`: negative if `a` sorts before `b`, zero if equal, positive if after. `cmp.Compare(a, b)` produces exactly that for ordered types:

```go
slices.SortFunc(people, func(a, b Person) int {
	if c := cmp.Compare(a.Age, b.Age); c != 0 {
		return c
	}
	return strings.Compare(a.Name, b.Name) // tie-break: name
})
```

### `clear`

`clear(s)` (Go 1.21+) sets every element of `s[:len(s)]` to its zero value; length and capacity do not change. (On a map, `clear` removes all entries — Unit 2.8.)

### Insert and delete by hand (to understand what the package does)

```go
// delete element i, keep order
s = append(s[:i], s[i+1:]...)

// delete element i, order does not matter (O(1))
s[i] = s[len(s)-1]
s = s[:len(s)-1]

// insert v at i
s = append(s, zero)        // grow by one
copy(s[i+1:], s[i:])       // shift right
s[i] = v
```

The first form moves elements left inside the *same* backing array, so the last slot of the old slice still holds a (now duplicated) value beyond the new length. `slices.Delete` (since Go 1.22) clears those vacated slots to the zero value:

```go
x := []int{1, 2, 3, 4, 5}
x = slices.Delete(x, 1, 3)
fmt.Println(x, x[:5]) // [1 4 5] [1 4 5 0 0]
```

### Filtering in place

```go
// keep only even numbers, reusing the same backing array: no allocation
out := s[:0]
for _, v := range s {
	if v%2 == 0 {
		out = append(out, v)
	}
}
clear(s[len(out):]) // optional: drop references in the leftover tail (important for pointers)
s = out
```

Or simply `s = slices.DeleteFunc(s, func(v int) bool { return v%2 != 0 })`.

### Memory leaks through sub-slices

A slice keeps its **entire** backing array alive, even if it only shows one element:

```go
func firstLine(data []byte) []byte {   // data is a 50 MB file
	i := bytes.IndexByte(data, '\n')
	return data[:i]                     // 20 bytes visible, 50 MB kept alive
}
```

As long as the returned slice is used, the garbage collector cannot free the 50 MB, because the GC sees a live pointer into that array (Unit 0.6). Fix: copy what you keep: `return bytes.Clone(data[:i])` (or `slices.Clone`).

The same problem in a subtler form: a slice of *pointers* shortened with `s = s[:n]` still holds the pointers in `s[n:cap]`, so the objects they point to cannot be collected. `clear(s[n:])` before shortening fixes it — which is exactly what `slices.Delete` does for you.

### `range` over slices

```go
for i, v := range s { ... } // v is a COPY of s[i]
```

- The range expression is evaluated **once**; the number of iterations is fixed by `len(s)` at the start. Appending inside the loop does not add iterations.
- `v` is a copy. Modifying `v` changes nothing; write `s[i] = ...`. For a slice of large structs, `for i := range s { p := &s[i]; ... }` avoids copying each element and lets you modify in place.
- Unlike an array (DT-98), ranging over a slice does *not* copy the elements first: a change to `s[2]` made during iteration *is* seen when the loop reaches index 2 (the header was copied, not the array).

### Deep & Tricky

**DT-109. `slices.Delete` and friends zero the tail (Go 1.22+).** *(Library behaviour, version-dependent)* `Delete`, `DeleteFunc`, `Compact`, `CompactFunc`, and `Replace` clear the elements between the new and old length. Before Go 1.22 they did not, and the old tail kept pointers alive. If you rely on the old values still being there (you should not), your code changed behaviour.

**DT-110. `range` over a slice sees writes; over an array it does not.** *(Guarantee)* For `sl := []int{1, 2, 3}`, setting `sl[2] = 100` during the first iteration makes the third iteration print 100. For `arr := [3]int{1, 2, 3}` it prints 3, because ranging an array copies the array (DT-98). Same code, different collection type, different output.

**DT-111. `slices.Sort` is not stable; `slices.SortStableFunc` is.** *(Guarantee)* "Stable" means equal elements keep their original order. `slices.Sort` uses pattern-defeating quicksort (pdqsort), which may reorder equals. If output order of ties matters (and in user-visible output it usually does), either use a stable sort or add a tie-breaker to the comparison so no two elements compare equal.

### Pattern

**P-19. Copy out small pieces of large buffers.** *(Memory practice)* When you keep a small part of a large slice or string (a header line of a file, one field of a big JSON message) beyond the current function, clone it (`bytes.Clone`, `slices.Clone`, `strings.Clone`). *Trade-off:* one small allocation instead of pinning a large one. *When not to:* when the large buffer lives as long as the piece anyway.

### Common mistakes

- Holding a tiny sub-slice of a huge buffer in a long-lived cache.
- Hand-written delete that leaves stale pointers in the tail.
- Calling `slices.Max` on an empty slice (panics) without a length check.
- Sorting user-visible output with an unstable sort and no tie-breaker, so the order changes between runs.

### Mastery check

1. **Explain.** Why can returning `data[:20]` from a function keep 50 MB of memory alive? How do you fix it?
2. **Predict.** `s := []int{1, 2, 3, 4}; for i, v := range s { if i == 0 { s = append(s, 5); s[3] = 40 }; fmt.Print(v, " ") }` — and what is `s` after the loop? (Careful: does the append allocate?)
3. **Write.** `removeAt(s []string, i int) []string` that preserves order, plus a version that does not; benchmark mentally — which is O(1)?
4. **Write.** Sort a slice of `struct{ Name string; Score int }` by score descending, then by name ascending, and print it. Use `slices.SortFunc` and `cmp.Compare`.

## Topic 2B — Strings and text

## Unit 2.5 — Strings, bytes, and runes

**One-line summary:** A Go string is an immutable sequence of **bytes**, conventionally holding UTF-8 text. `len` counts bytes, indexing gives a byte, and `range` decodes **runes** (Unicode code points). What a human calls "one character" can be one byte, several bytes, or even several runes.

**Mental model — beads on a thread.** The string is a thread of bytes. In English text each letter is one bead. In Hindi, Chinese, or emoji, one letter may be 2, 3, or 4 beads tied together. `len` counts beads. `s[i]` picks one bead (possibly half of a letter). `range` walks letter-sized groups of beads (runes).

### Strings are immutable byte sequences

```go
s := "hello"
fmt.Println(len(s), s[0]) // 5 104   (s[0] is the byte 'h' = 104)
// s[0] = 'H'             // compile error: cannot assign to s[0] (neither addressable nor a map index expression)
```

- A string value is a 2-word header: a pointer to the bytes and a length (16 bytes on 64-bit). Assigning a string copies the header, never the bytes.
- Because the bytes can never change, sharing them is always safe. `s[2:4]` creates a new header pointing into the same bytes — no copy.
- "Changing" a string always means building a new one.

### UTF-8 in five minutes

**Unicode** gives every character a number called a **code point**, written `U+` and hex: `A` is U+0041, `é` is U+00E9, `न` is U+0928, `€` is U+20AC, `😀` is U+1F600. **UTF-8** is the encoding that turns code points into bytes:

| Code point range | Bytes | Examples |
| --- | --- | --- |
| U+0000 – U+007F | 1 | ASCII: `A`, `7`, `{` |
| U+0080 – U+07FF | 2 | `é`, `ñ`, Greek, Cyrillic, Arabic |
| U+0800 – U+FFFF | 3 | `न` (Devanagari), Chinese, `€` |
| U+10000 – U+10FFFF | 4 | emoji, rare scripts |

Design points worth knowing: ASCII text is already valid UTF-8 (backwards compatible); the first byte tells you how many bytes follow; continuation bytes always start with `10` in binary, so a decoder can resynchronise in the middle of a stream. Go source files are UTF-8, and string literals are UTF-8.

### `len`, indexing, and `range`

```go
s := "héllo"
fmt.Println(len(s))        // 6, not 5: 'é' takes 2 bytes
fmt.Println(s[1])          // 195: the FIRST byte of 'é' (0xC3)
for i, r := range s {
	fmt.Print(i, ":", string(r), " ")
}
// 0:h 1:é 3:l 4:l 5:o    <- index jumps from 1 to 3
```

- `range` over a string decodes one rune per iteration; `i` is the **byte** offset where the rune starts.
- A **rune** is an `int32` holding a code point. `'é'` is a rune literal (value 233).
- Invalid UTF-8 bytes decode as U+FFFD (the replacement character, `�`), one byte at a time, and the loop continues.

Counting characters:

```go
utf8.RuneCountInString("नमस्ते") // 6 runes
len("नमस्ते")                    // 18 bytes
```

But a person reading नमस्ते sees about 3 or 4 "letters". Devanagari combines a consonant with vowel signs and the *virama* (्), each a separate rune. Likewise `é` can be one rune (U+00E9) or two (`e` + U+0301 combining acute accent) — they look identical and compare **unequal**. What a person perceives as one character is a **grapheme cluster**, and Go's standard library does not segment graphemes (the `golang.org/x/text` module and third-party packages do). For most programs, runes are the right unit; for cursor movement or "truncate to 10 characters" in a UI, graphemes are.

### Converting between strings, bytes, and runes

| Conversion | Result | Copies? |
| --- | --- | --- |
| `[]byte(s)` | the bytes, mutable | yes |
| `string(b)` | a string from bytes | yes |
| `[]rune(s)` | one element per code point | yes |
| `string(r)` for a rune `r` | the UTF-8 encoding of that one rune | yes |
| `string(65)` with an int | `"A"` — **not** `"65"`; `go vet` warns | — |
| `strconv.Itoa(65)` | `"65"` | — |

The copies are necessary: a string is immutable and a `[]byte` is mutable, so they cannot share memory safely. (The compiler skips the copy in a few safe cases — DT-114.)

### Printing text: `%c`, `%q`, `%U`, `% x`

```go
r := 'न'
fmt.Printf("%c %q %U %d\n", r, r, r, r) // न 'न' U+0928 2344
s := "héllo"
fmt.Printf("%q\n", s)       // "héllo"
fmt.Printf("%+q\n", s)      // "h\u00e9llo"   (ASCII-only)
fmt.Printf("% x\n", s)      // 68 c3 a9 6c 6c 6f
```

`%q` is the best way to print a string while debugging: invisible characters, trailing spaces, `\r`, and `\t` become visible.

### Raw string literals

Backquotes make a **raw string**: no escape sequences, may span lines.

```go
path := `C:\new\table`      // backslashes are literal
query := `SELECT id
FROM users`
```

Useful for regular expressions, file paths, SQL, and JSON snippets in tests. A raw string cannot contain a backquote.

### Deep & Tricky

**DT-112. Strings may contain any bytes, not only valid UTF-8.** *(Guarantee)* `"\xff"` is a legal one-byte string. Only *literals* in source code are UTF-8 by convention; data from files and networks may not be. Check with `utf8.ValidString` at the edge (P-4) when it matters.

**DT-113. `string(i)` for an integer `i` is a rune conversion.** *(Guarantee + vet check)* `string(rune(65))` is `"A"`. Writing `string(65)` with an int is allowed by the spec but almost always a bug (the author wanted `"65"`), so `go vet` reports "conversion from int to string yields a string of one rune". Use `strconv.Itoa` for decimal text.

**DT-114. The compiler avoids some `[]byte` ↔ `string` copies.** *(Implementation detail)* `m[string(b)]` for a map lookup, `string(b) == "x"` comparisons, `for i, c := range []byte(s)`, and `s + string(b)` concatenations do not allocate a separate copy of `b` in current compilers, because the temporary string cannot escape. Do not contort code to avoid conversions until a profile says so.

**DT-115. Comparing strings is byte-wise, so "é" may not equal "é".** *(Guarantee + Unicode fact)* Precomposed U+00E9 and decomposed `e` + U+0301 render identically, but their bytes differ, so `==` is false and they sort differently. Normalise to a standard form (NFC) with `golang.org/x/text/unicode/norm` before comparing user-entered text.

### Common mistakes

- Using `len(s)` as "number of characters" for non-ASCII text.
- Indexing `s[i]` and treating the result as a character.
- Slicing a string at a byte offset that falls in the middle of a rune (`"héllo"[:2]` gives `"h\xc3"`, broken text).
- `string(n)` to format a number.
- Assuming one rune is one visible character (combining marks, emoji sequences).

### Mastery check

1. **Explain.** Why is `len("€")` 3? Why does `range` jump byte offsets?
2. **Predict.** (a) `len("Go🙂")` (b) `utf8.RuneCountInString("Go🙂")` (c) `fmt.Println("Go🙂"[2])` (d) `fmt.Printf("%q\n", "Go🙂"[:3])` (e) `string(rune(0x1F642))`
3. **Write.** `reverse(s string) string` that reverses by runes, then test it with `"नमस्ते"` and observe why the result looks broken. Explain what grapheme clusters have to do with it.

## Unit 2.6 — Building and converting strings

**One-line summary:** The `strings` package does searching, splitting, and trimming; `strings.Builder` builds strings efficiently; `strconv` converts between strings and numbers precisely; `unicode` and `unicode/utf8` classify and encode runes. The `bytes` package mirrors `strings` for `[]byte`.

### The `strings` toolbox

| Task | Functions |
| --- | --- |
| Test | `Contains`, `ContainsRune`, `ContainsAny`, `HasPrefix`, `HasSuffix`, `EqualFold` (case-insensitive equality) |
| Find | `Index`, `LastIndex`, `IndexByte`, `IndexRune`, `IndexFunc`, `Count` |
| Split | `Split(s, sep)`, `SplitN`, `Fields` (on any whitespace), `FieldsFunc`, `Cut(s, sep)`, `CutPrefix`, `CutSuffix`, `CutLast` (Go 1.27) |
| Trim | `TrimSpace`, `Trim(s, cutset)`, `TrimPrefix`, `TrimSuffix`, `TrimFunc`, `TrimLeft`, `TrimRight` |
| Change | `ToUpper`, `ToLower`, `Title`-style casing via `golang.org/x/text/cases`, `Replace`, `ReplaceAll`, `Repeat`, `Map` |
| Join | `Join(parts, sep)` |
| Iterate (Go 1.24+) | `Lines`, `SplitSeq`, `FieldsSeq` — iterators that avoid building a slice |

`strings.Cut` deserves special mention; it replaced many awkward `Index` + slicing combinations:

```go
key, value, ok := strings.Cut("name=Ravi", "=")
// key "name", value "Ravi", ok true
```

**Two traps:** `strings.Split("", ",")` returns `[""]` (one empty element), not an empty slice; and `TrimLeft`/`TrimRight` take a *set of characters*, not a prefix: `strings.TrimLeft("aabcd", "ab")` is `"cd"`. For prefixes use `TrimPrefix`.

### Building strings: `+` vs `strings.Builder`

Each `+` makes a new string and copies both sides. In a loop, that is O(n²) bytes copied:

```go
s := ""
for i := range 10_000 {
	s += strconv.Itoa(i) // copies everything built so far, every time
}
```

`strings.Builder` keeps a growing `[]byte` internally and turns it into a string at the end without another copy:

```go
var b strings.Builder           // zero value is ready to use (DT-48)
for i := range 10_000 {
	b.WriteString(strconv.Itoa(i))
	b.WriteByte(',')
}
s := b.String()
```

`b.Grow(n)` pre-sizes it (P-17). A Builder must not be copied after first use (the copy and the original would share the buffer; the type detects it and panics). `fmt.Fprintf(&b, ...)` works too, because `*strings.Builder` is an `io.Writer` (Chapter 5). For a handful of concatenations, `+` or `fmt.Sprintf` is perfectly fine and clearer.

### `strconv`: precise conversions

```go
n, err := strconv.Atoi("42")                 // int
i, err := strconv.ParseInt("-ff", 16, 64)    // base 16, must fit int64: -255
u, err := strconv.ParseUint("300", 10, 8)    // bitSize 8: error "value out of range"
f, err := strconv.ParseFloat("3.25", 64)
ok, err := strconv.ParseBool("true")         // also accepts 1, t, T, TRUE, True, 0, f, ...

s1 := strconv.Itoa(-42)                      // "-42"
s2 := strconv.FormatInt(255, 2)              // "11111111"
s3 := strconv.FormatFloat(1.5, 'f', 2, 64)   // "1.50"
s4 := strconv.Quote("tab\there")             // the text "tab\there", with quotes and a visible \t escape
buf := strconv.AppendInt(nil, 42, 10)        // append form: no intermediate string
```

The errors are `*strconv.NumError` values that say what failed and why (`strconv.ParseUint: parsing "300": value out of range`). Prefer `strconv` over `fmt.Sscanf` for single values: it is faster and stricter.

`ParseInt`'s `base` 0 means "detect from the prefix", accepting `0x`, `0o`, `0b`, and underscores like Go literals: `strconv.ParseInt("0x_FF", 0, 64)` is 255.

### `unicode` and `unicode/utf8`

```go
unicode.IsLetter('न')   // true
unicode.IsMark('े')     // true: a vowel sign is a mark, not a letter
unicode.IsDigit('٣')    // true: Arabic-Indic three
unicode.IsSpace('\u00A0') // true: no-break space
unicode.ToUpper('ß')    // 'ß' — unchanged; see DT-117

r, size := utf8.DecodeRuneInString("€uro") // '€', 3
utf8.RuneLen('€')                          // 3
utf8.ValidString("\xff")                   // false
b := utf8.AppendRune(nil, '€')             // [226 130 172]
```

### `bytes`: the same toolbox for `[]byte`

`bytes.Contains`, `bytes.Split`, `bytes.TrimSpace`, `bytes.Buffer` (a read/write byte buffer), and so on mirror `strings`. Use them when your data is already `[]byte` (file contents, network data) to avoid converting back and forth.

### Deep & Tricky

**DT-116. `strings.ToUpper` can change the length of a string.** *(Unicode fact)* Case mapping is not one-to-one in all languages: some lowercase characters have uppercase forms that are encoded with a different number of bytes (for example Turkish dotless `ı` → `I`, or `ſ` → `S`). Never assume `len(strings.ToUpper(s)) == len(s)`, and never use the byte offsets of one to slice the other.

**DT-117. Simple case mapping is not full case mapping.** *(Library behaviour)* `unicode.ToUpper` works on single runes and uses simple one-to-one mappings, so `ß` stays `ß` (its full uppercase is "SS", two characters). `strings.ToUpper("ß")` also returns `"ß"` for the same reason. Language-aware casing (`İ` in Turkish, "SS" in German) needs `golang.org/x/text/cases`. Case-insensitive comparison should use `strings.EqualFold`, which uses Unicode case folding, not `ToLower(a) == ToLower(b)`.

**DT-118. A `strings.Builder` panics if copied after use.** *(Library behaviour)* The Builder stores a pointer to itself on first write and checks it on every call. `b2 := b1` after writing, then `b2.WriteString(...)`, panics with `strings: illegal use of non-zero Builder copied by value`. Pass `*strings.Builder` to functions.

### Pattern

**P-20. Parse with `strconv`, not `fmt.Sscan`, for single values; always keep the error.** *(Go practice)* `strconv` reports exactly what was wrong (syntax vs range), is several times faster, and does not silently accept trailing garbage the way some `Sscanf` formats do. Wrap the error with context (`fmt.Errorf("parsing port %q: %w", s, err)`, Chapter 3) instead of replacing it.

### Common mistakes

- Building large strings with `+=` in a loop.
- `TrimLeft(s, "prefix")` when `TrimPrefix` was meant.
- Comparing case-insensitively with `ToLower` instead of `EqualFold`.
- `ParseInt(s, 10, 64)` and then converting to `int8` without a range check (silent truncation, DT-41) — pass `bitSize` 8 instead.

### Mastery check

1. **Explain.** Why is `s += x` in a loop O(n²), and how does `strings.Builder` avoid it?
2. **Predict.** (a) `strings.Split("a,b,,c", ",")` (b) `len(strings.Split("", ","))` (c) `strings.TrimLeft("xxhixx", "x")` (d) `strings.Fields("  a \t b\n")` (e) `strconv.ParseUint("256", 10, 8)` (f) `strings.EqualFold("Straße", "STRASSE")`
3. **Write.** `parseKV(line string) (map[string]string, error)` for input like `name=Ravi; city = Pune ;age=30`: split on `;`, then `Cut` on `=`, trim spaces, reject empty keys with a clear error.

## Topic 2C — Maps

## Unit 2.7 — Maps I

**One-line summary:** A map stores key → value pairs and finds a value by its key in constant time on average, using a hash table. Reading a missing key gives the zero value; the comma-ok form tells you whether the key was really there. Writing to a nil map panics, and iteration order is deliberately random.

**Mental model — the post office sorting rack.** A post office has a rack of pigeonholes. To store a letter, the clerk computes a pigeonhole number from the address (the **hash**) and puts the letter there. To find it later, the same computation sends him straight to the right pigeonhole — no need to search all letters. Sometimes two addresses land in the same pigeonhole (a **collision**); he then checks the few letters in that hole one by one. When the rack gets too full, he builds a bigger rack and re-sorts everything.

### Creating and using maps

```go
ages := map[string]int{       // map literal
	"Asha": 31,
	"Ravi": 28,               // trailing comma required on multi-line literals (DT-17)
}
scores := make(map[string]int)      // empty, ready to use
big := make(map[int]string, 10_000) // size hint: room for ~10,000 entries without growing
var m map[string]int                // nil map: reading works, WRITING PANICS

ages["Meera"] = 35            // insert or overwrite
fmt.Println(ages["Asha"])     // 31
fmt.Println(ages["Nobody"])   // 0 — missing key gives the zero value, no error
delete(ages, "Ravi")          // remove; deleting a missing key is a no-op
fmt.Println(len(ages))        // 2
```

### The comma-ok idiom

Because a missing key returns the zero value, `ages["Nobody"] == 0` cannot tell "missing" from "present with value 0". The two-result form can:

```go
age, ok := ages["Nobody"] // ok == false: the key is not in the map
if v, ok := ages["Asha"]; ok {
	fmt.Println("found", v)
}
```

This `(value, ok)` shape appears throughout Go: maps, type assertions (Unit 3.6), channel receives (Unit 7.3). Use it whenever the zero value is a meaningful value.

**Counting with the zero value.** The missing-key rule makes counting elegant:

```go
counts := make(map[string]int)
for _, w := range words {
	counts[w]++ // missing key reads as 0, then 1 is stored
}
```

The same trick works for grouping: `groups[k] = append(groups[k], v)` — a missing key gives a nil slice, and appending to nil works (DT-106).

### Nil maps

```go
var m map[string]int
fmt.Println(m["x"], len(m)) // 0 0 — reading is fine
for k, v := range m {}      // zero iterations — fine
delete(m, "x")              // fine (no-op)
m["x"] = 1                  // panic: assignment to entry in nil map
```

A nil map has no hash table to write into. Always `make` (or use a literal) before writing. This is the most common map panic in real code, typically from a struct field `map[...]...` that was never initialised (Unit 3.8 shows constructors that prevent it).

### Iteration order is random

```go
for name, age := range ages {
	fmt.Println(name, age) // order differs from run to run
}
```

The Go runtime deliberately randomises the starting point of every map iteration. Code that accidentally depended on insertion order would break mysteriously on another Go version; making the order random from day one makes such bugs show up immediately in tests. When you need a stable order — reports, tests, anything a human reads — collect the keys and sort them (Unit 2.8).

### Which types can be keys?

Keys must be **comparable** with `==`: booleans, numbers, strings, pointers, channels, interfaces, arrays of comparable types, and structs whose fields are all comparable. Slices, maps, and functions cannot be keys (compile error: `invalid map key type`). Values can be any type, including slices and other maps.

Composite keys with arrays or structs are often cleaner than building a string:

```go
visited := map[[2]int]bool{}        // grid cell (row, col) — DT-100
type edge struct{ from, to int }
weight := map[edge]float64{{1, 2}: 3.5}
```

### Hash tables, briefly

A Go map hashes the key with a fast hash function seeded randomly per map (so attackers cannot predict collisions), uses part of the hash to pick a group of slots, and compares keys within the group. Average cost of lookup, insert, and delete: O(1). When the table gets too full it grows and moves entries into a larger table (incrementally, so no single insert stalls for long). Unit 6.4 builds a hash table from scratch; Unit 10.8 looks at Go's own implementation.

### Deep & Tricky

**DT-119. Go 1.24 rewrote maps as "Swiss tables".** *(Implementation detail)* Since Go 1.24 the built-in map uses a Swiss-table design: slots in groups of 8 with a control word of one-byte hash fragments, checked several at a time. Lookups and inserts got noticeably faster for large maps. Nothing about map *semantics* changed — which is the point of keeping the representation an implementation detail.

**DT-120. A map with NaN keys can hold "the same" key many times.** *(Guarantee, consequence of DT-7)* `m[math.NaN()] = 1; m[math.NaN()] = 2` gives `len(m) == 2`, and `m[math.NaN()]` always reads 0, because NaN is not equal to itself, so a lookup never matches. You can only get such entries back by ranging over the map or remove them with `clear(m)`. Avoid float keys; if you must use them, reject NaN at the edge.

**DT-121. Iteration order is randomised per iteration, not just per run.** *(Implementation behaviour)* Two `range` loops over the same unchanged map in the same run usually visit keys in different orders. Never assume "the same order as last time".

**DT-122. Adding to a map during `range` may or may not show up in the same loop.** *(Guarantee)* The spec says an entry added during iteration may be produced or skipped; an entry deleted before it is reached will not be produced. Deleting the *current* key during range is safe and common.

### Common mistakes

- Writing to a nil map (often a struct field never initialised).
- Using `m[k] == 0` to test presence when 0 is a valid value.
- Printing or testing output that depends on map iteration order.
- Using slices as keys (compile error) — use an array or a string.

### Mastery check

1. **Explain.** Why does the runtime randomise iteration order? What kind of bug does that prevent?
2. **Predict.** (a) `var m map[string]int; fmt.Println(m["a"])` (b) `m["a"] = 1` on that map (c) `m := map[string]int{"a": 0}; v, ok := m["a"]` (d) `delete(m, "zzz")` (e) `len(map[float64]int{math.NaN(): 1})` — careful: is that a constant? Can it compile?
3. **Write.** `groupByLength(words []string) map[int][]string`, then print the groups in increasing length order (so the output is the same on every run).

## Unit 2.8 — Maps II

**One-line summary:** Use `map[K]struct{}` for sets, the `maps` and `slices` packages for keys and sorting, and remember three hard rules: map elements are not addressable, maps are shared when copied, and maps are not safe for concurrent writes.

### Sets

Go has no set type; a map with an empty value is the idiom:

```go
seen := make(map[string]struct{})
seen["go"] = struct{}{}
if _, ok := seen["go"]; ok { /* member */ }
```

`struct{}` takes zero bytes (DT-125). `map[string]bool` is also common and reads more naturally (`if seen[x]`); the memory difference is small, so choose for clarity unless the set is huge.

### Map elements are not addressable

```go
type point struct{ X, Y int }
pts := map[string]point{"a": {1, 2}}
pts["a"].X = 10       // compile error: cannot assign to struct field pts["a"].X in map
```

The map may move its entries when it grows, so Go does not let you hold the address of an entry (`&pts["a"]` is also an error). Two ways to update:

```go
p := pts["a"] // copy out
p.X = 10
pts["a"] = p  // write back

ptrs := map[string]*point{"a": {1, 2}} // or store pointers
ptrs["a"].X = 10                         // fine: modifies the pointed-to struct
```

`m[k]++`, `m[k] += 5`, and `m[k] = append(m[k], v)` are fine: they read the value, compute, and store it back as a whole.

### Maps are shared when copied

A map value is a pointer to the runtime's table. Assigning or passing a map copies that pointer:

```go
a := map[string]int{"x": 1}
b := a
b["x"] = 99
fmt.Println(a["x"]) // 99
```

A function that receives a map can modify the caller's map. To get an independent copy: `c := maps.Clone(a)` (a *shallow* copy: values that are slices or pointers are still shared).

### The `maps` package and sorted output

| Task | Code |
| --- | --- |
| Keys / values (iterators, Go 1.23+) | `maps.Keys(m)`, `maps.Values(m)` |
| Sorted keys | `slices.Sorted(maps.Keys(m))` |
| Copy | `maps.Clone(m)`, `maps.Copy(dst, src)` |
| Compare | `maps.Equal(a, b)` |
| Delete matching | `maps.DeleteFunc(m, func(k K, v V) bool {...})` |
| Remove everything | `clear(m)` |

Deterministic output, the idiom you will write most often:

```go
for _, name := range slices.Sorted(maps.Keys(ages)) {
	fmt.Printf("%-6s %d\n", name, ages[name])
}
```

`maps.Keys` returns an *iterator* (Unit 3.16), not a slice; `slices.Sorted` collects it into a sorted slice.

### Not safe for concurrent writes

If one goroutine writes to a map while another reads or writes it, the runtime may detect it and **kill the program**:

```
fatal error: concurrent map writes
```

This is a *fatal error*, not a panic: `recover` cannot catch it (DT-39 explains the difference). Protect shared maps with a `sync.Mutex` or `sync.RWMutex`, or use `sync.Map` for a few special cases (Chapter 7). Concurrent *reads* with no writer are safe.

### Memory: maps do not shrink

Deleting entries frees the *values'* references but the table itself keeps its size: a map that once held 10 million entries keeps the memory for 10 million slots even after you delete all but ten. If a long-lived map shrinks a lot, copy the survivors into a new map and drop the old one.

### Deep & Tricky

**DT-123. `clear(m)` is not the same as `m = make(...)`.** *(Guarantee)* `clear(m)` empties the map that *every holder* of `m` sees (they share it). `m = make(map[K]V)` gives only *this variable* a new empty map; other holders still see the old contents. Choose deliberately.

**DT-124. `fatal error: concurrent map writes` is best-effort detection.** *(Implementation behaviour)* The runtime checks a flag on each map operation and catches many races, but not all. A program can corrupt a map silently before it is detected. The race detector (`go test -race`, Chapter 7) is the reliable tool.

**DT-125. Zero-size types share an address.** *(Implementation detail; the spec allows it)* `struct{}{}` and `[0]int{}` occupy no memory, and pointers to different zero-size variables may compare equal. Never use the address of a zero-size value as an identity.

### Pattern

**P-21. Sort keys for any output a human or a test reads.** *(Go practice)* Map order is random by design; reports, logs compared in tests, and golden files (Unit 4.2) must not depend on it. `slices.Sorted(maps.Keys(m))` costs O(n log n), which is negligible for output-sized maps.

### Common mistakes

- Trying `m[k].field = v` on a map of structs.
- Assuming a function cannot modify a map it received (it can).
- Using a plain map from several goroutines.
- Thinking `maps.Clone` is a deep copy.

### Mastery check

1. **Explain.** Why are map elements not addressable? Give two ways to update a struct stored in a map.
2. **Predict.** `func add(m map[string]int) { m["x"] = 1; m = nil }` — after `m := map[string]int{}; add(m)`, what is `len(m)`? Why did `m = nil` inside `add` not matter?
3. **Write.** Set operations on `map[string]struct{}`: `union`, `intersection`, `difference`, each returning a new set, and a `sortedMembers` helper. Test with small examples.

## Topic 2D — Structs and pointers

## Unit 2.9 — Structs

**One-line summary:** A struct groups named fields of different types into one value. Structs are values: assigning one copies all its fields. A struct can be compared with `==` when all its fields are comparable.

**Mental model — a form with labelled boxes.** A struct type is a printed form: "Name: ___, Age: ___, City: ___". A struct value is one filled-in copy of that form. Photocopying the form (assignment) gives an independent copy, except where a box says "see attached file" (a pointer, slice, or map field) — both copies point to the same attachment.

### Declaring and creating

```go
type Employee struct {
	Name   string
	Age    int
	Skills []string
	active bool // lower-case: unexported (visible only inside this package, Unit 3.12)
}

var e1 Employee                        // zero value: {"" 0 nil false}
e2 := Employee{Name: "Asha", Age: 31}  // keyed literal: missing fields get zero values
e3 := Employee{"Ravi", 28, nil, true}  // positional: ALL fields, in order — avoid
e2.Age++                               // field access with .
```

Prefer keyed literals: they keep working when fields are added or reordered, and they say what each value means. `go vet` warns about unkeyed literals of struct types from *other* packages. Since Go 1.27, keys in a struct literal may also name **promoted** fields of embedded structs (Unit 3.7).

### Zero values compose

A struct's zero value is every field at its own zero value. A well-designed struct is useful at zero (DT-48): `var b bytes.Buffer` or `var mu sync.Mutex` work immediately. Note what that means for fields of map type: their zero value is nil, so writing to them panics — a struct with a map field usually needs a constructor (Unit 3.8).

### Structs are values

```go
a := Employee{Name: "Asha", Skills: []string{"Go"}}
b := a                       // copies every field
b.Name = "Meera"             // a.Name is still "Asha"
b.Skills[0] = "Rust"         // a.Skills[0] is now "Rust" too: the slice header was copied, the array is shared
```

The copy is **shallow**: fields that are slices, maps, or pointers copy only their headers or addresses. Passing a struct to a function copies it the same way.

### Comparison

Two struct values are equal with `==` if all corresponding fields are equal. That requires every field type to be comparable; a struct with a slice, map, or function field cannot be compared (compile error) and cannot be a map key. Comparable structs make excellent map keys (Unit 2.7).

### Anonymous structs

A struct type can be written inline without a name, which is handy for one-off groupings, especially table-driven tests (Unit 4.1):

```go
tests := []struct {
	in   string
	want int
}{
	{"a", 1},
	{"bb", 2},
}
point := struct{ X, Y int }{1, 2}
```

### Struct tags (preview)

A field may carry a string **tag**, metadata that packages read through reflection:

```go
type User struct {
	ID    int    `json:"id"`
	Email string `json:"email,omitempty"`
	Pass  string `json:"-"` // never encoded
}
```

The compiler ignores tags; `encoding/json` (Chapter 5), database libraries (Chapter 9), and validators read them. Format: space-separated `key:"value"` pairs; `go vet` checks the syntax.

### The empty struct

`struct{}` has no fields and takes zero bytes. It is used for set values (Unit 2.8) and for channel signals where only the event matters (Chapter 7).

### Deep & Tricky

**DT-126. Struct literals of pointer type: `&T{...}` is fine, and Go allows `{...}` shorthand inside composite literals.** *(Guarantee)* `ps := []*Point{{1, 2}, {3, 4}}` means `[]*Point{&Point{1, 2}, &Point{3, 4}}`. The elided `&T` in nested literals is legal and idiomatic.

**DT-127. Field order and names are part of the type.** *(Guarantee)* `struct{ X, Y int }` and `struct{ Y, X int }` are different types, and so are structs with the same fields but different tags... except that *conversion* between struct types ignores tags (since Go 1.8). Two named struct types with identical underlying fields can be converted to each other with `T2(v)` (Unit 3.1).

**DT-128. Comparing structs with `==` compares padding-free field values, not memory.** *(Guarantee)* Equality is defined field by field; padding bytes (Unit 2.10) never affect it. A float field holding NaN makes the struct unequal to itself (DT-7).

### Common mistakes

- Positional literals that silently break when a field is added.
- Expecting a struct copy to deep-copy its slices and maps.
- A struct with a nil map field written to without initialisation.
- Exporting every field by habit; start unexported and export what callers need (Unit 3.12).

### Mastery check

1. **Explain.** What does "shallow copy" mean for a struct containing a slice? Draw it.
2. **Predict.** (a) `type P struct{ X, Y int }; P{1, 2} == P{1, 2}` (b) a struct with a `[]int` field compared with `==` (c) `type T struct{ M map[string]int }; var t T; t.M["a"] = 1`
3. **Write.** An `Order` struct (ID, customer, lines of {product, quantity, price in paise}), a function `total(o Order) int64`, and a table-driven check using an anonymous struct slice.

## Unit 2.10 — Struct memory layout

**One-line summary:** Each type has an *alignment*: its address must be a multiple of some power of two. To satisfy alignment the compiler inserts invisible **padding** between fields, so field order can change a struct's size. Go does not reorder fields for you.

**Mental model — packing a tiffin bag with fixed slots.** Some containers only fit in slots that start at a multiple of 8 cm; small ones fit anywhere. Put a tiny container first, then a big one, and the big one must skip to the next 8 cm mark, leaving a gap. Put the big ones first and the small ones together at the end, and the gaps disappear.

### Size and alignment of basic types (64-bit platforms)

| Type | Size | Alignment |
| --- | --- | --- |
| `bool`, `int8`, `uint8` | 1 | 1 |
| `int16`, `uint16` | 2 | 2 |
| `int32`, `uint32`, `float32`, `rune` | 4 | 4 |
| `int64`, `uint64`, `float64`, `int`, `uint`, pointers, `uintptr` | 8 | 8 |
| `string` | 16 (ptr + len) | 8 |
| slice | 24 (ptr + len + cap) | 8 |
| map, channel, function | 8 (one pointer) | 8 |
| interface | 16 (two words, Unit 3.5) | 8 |
| `[N]T` | N × size of T | alignment of T |
| struct | sum of fields + padding | largest field alignment |

Rules: each field starts at an offset that is a multiple of its alignment; the whole struct's size is rounded up to a multiple of its own alignment (so that arrays of the struct keep every element aligned).

### Padding in action

```go
type Bad struct {
	a bool  // offset 0, size 1
	        // 7 bytes padding so b starts at 8
	b int64 // offset 8, size 8
	c bool  // offset 16, size 1
	        // 7 bytes padding to round the size to a multiple of 8
}           // unsafe.Sizeof(Bad{}) == 24

type Good struct {
	b int64 // offset 0
	a bool  // offset 8
	c bool  // offset 9
	        // 6 bytes padding
}           // unsafe.Sizeof(Good{}) == 16
```

Same data, a third less memory. For one value it does not matter; for a slice of 10 million it is 80 MB, and more values per cache line (Unit 0.5) also means faster scans.

### Measuring

```go
import "unsafe"

unsafe.Sizeof(Bad{})     // 24
unsafe.Alignof(Bad{}.b)  // 8
unsafe.Offsetof(Bad{}.c) // 16
```

`unsafe.Sizeof` is computed at compile time and is safe to call; the danger in package `unsafe` lies elsewhere (Unit 10.10). The `fieldalignment` analyzer (in `golang.org/x/tools`) reports structs that could be smaller and suggests an order.

### When to care

- Large slices or arrays of a struct; caches holding millions of entries; data structures in the routing chapters (Unit 13.8).
- **Not** for ordinary structs that exist a few at a time: there, order fields for *readability* (related fields together).

### Deep & Tricky

**DT-129. A zero-size field at the end of a struct adds padding.** *(Implementation detail)* `struct{ x int32; y struct{} }` is 8 bytes, not 4: if a zero-size final field had the struct's end address, `&s.y` would point past the struct, possibly into the next object, which would confuse the garbage collector. The compiler pads it. Put zero-size fields first if it matters.

**DT-130. 64-bit atomic operations need 8-byte alignment, which 32-bit platforms do not give automatically.** *(Platform rule)* On 32-bit ARM and x86, an `int64` field may be only 4-byte aligned, and `sync/atomic` 64-bit functions on it crash. The old rule was "put 64-bit atomic fields first". The typed `atomic.Int64` (Unit 7.6) guarantees alignment and is the modern answer.

**DT-131. Go does not reorder struct fields.** *(Current compiler behaviour; the spec does not promise layout)* The gc compiler lays fields out in declaration order. Code that talks to C, to hardware, or to binary formats relies on this, so it is unlikely to change, but the language spec itself does not guarantee a memory layout.

### Mastery check

1. **Explain.** Why must a struct's size be a multiple of its alignment?
2. **Predict.** `unsafe.Sizeof` of (a) `struct{ a int8; b int32; c int8 }` (b) `struct{ b int32; a, c int8 }` (c) `struct{ s string; b bool }` (d) `struct{}`
3. **Write.** Define a struct with at least six fields of mixed sizes, measure it, reorder for minimum size, and verify with `unsafe.Offsetof` for every field.

## Unit 2.11 — Pointers

**One-line summary:** A pointer holds the address of a variable. `&x` takes the address; `*p` reads or writes the variable it points to. Pointers let functions modify the caller's data and let many parts of a program share one value. Go pointers are safe: no arithmetic, garbage-collected targets, and a nil dereference panics instead of corrupting memory.

**Mental model — a house address on a slip of paper.** A variable is a house. A pointer is a slip with the house's address written on it. Copying the slip gives two slips with the same address: whoever follows either slip reaches the same house. A nil pointer is a blank slip: following it is an error.

### `&`, `*`, and pointer types

```go
x := 10
p := &x          // p has type *int: "pointer to int"
fmt.Println(*p)  // 10: dereference — read through the pointer
*p = 20          // write through the pointer
fmt.Println(x)   // 20
```

- `*int` (in a type) means "pointer to int". `*p` (in an expression) means "the variable p points to". Same symbol, two roles.
- The zero value of any pointer type is `nil`.

### Pointers let functions modify the caller's variables

Go copies arguments (DT-75); copying a pointer gives the function the caller's address:

```go
func double(n *int) { *n *= 2 }

v := 5
double(&v)
fmt.Println(v) // 10
```

This is also how `fmt.Scan(&n)` (Unit 1.12) stores into your variable.

### Pointers to structs

```go
type Account struct{ Balance int64 }

func deposit(a *Account, amt int64) { a.Balance += amt } // a.Balance means (*a).Balance

acc := &Account{Balance: 100}  // & on a composite literal: allocate and take the address
deposit(acc, 50)
fmt.Println(acc.Balance)       // 150
```

Go automatically dereferences pointers to structs for field access, so you write `a.Balance`, never `(*a).Balance`. The same applies to pointers to arrays: `pa[2]` works for `pa *[5]int`.

### `new`

`new(T)` allocates a zero `T` and returns its address: `p := new(int)` gives a `*int` pointing to 0. It is rarely needed (`&T{}` is more common for structs). Since Go 1.26, `new` also accepts an expression, giving the new variable an initial value: `p := new(42)` or `age := new(yearsSince(born))`. This is especially handy for optional fields represented as pointers (`Name: new("Asha")` in a struct literal where `Name *string`).

### nil pointers

```go
var p *Account
fmt.Println(p == nil) // true
fmt.Println(p.Balance) // panic: runtime error: invalid memory address or nil pointer dereference
```

The panic comes from the same mechanism as calling a nil function (DT-84): address 0 is never mapped, the CPU faults, the runtime turns it into a panic. Check for nil where a pointer may legitimately be absent, and design APIs so that it usually cannot be.

### What Go pointers cannot do

- **No pointer arithmetic.** `p + 1` is a compile error. You cannot walk memory; you use slices instead. (Package `unsafe` can, at your own risk, Unit 10.10.)
- **No dangling pointers.** A pointer keeps its target alive; the GC never frees memory something still points to.
- **No pointers to map entries or to string bytes** (they could move or must stay immutable).

### Pointers and loops (Go 1.22+)

Since Go 1.22, each iteration has its own loop variable (DT-72), so this collects three distinct addresses:

```go
var ptrs []*int
for i := range 3 {
	ptrs = append(ptrs, &i)
}
fmt.Println(*ptrs[0], *ptrs[1], *ptrs[2]) // 0 1 2   (before 1.22: 3 3 3)
```

But `for _, v := range items { ptrs = append(ptrs, &v) }` gives pointers to *copies*, not to the elements of `items`. To point into the slice, use the index: `&items[i]`.

### Deep & Tricky

**DT-132. A pointer into a slice's backing array goes stale after a growing append.** *(Guarantee + classic bug)* `p := &s[0]; s = append(s, x)` — if the append reallocated, `p` still points into the *old* array. Writes through `p` are no longer visible in `s`. No crash, just silently diverging data. Take element pointers after you finish growing, or store indexes instead.

**DT-133. Pointers are comparable; equal pointers point to the same variable.** *(Guarantee, with the zero-size exception of DT-125)* `p == q` is true if they point to the same variable or are both nil. Pointers can be map keys, which is useful for identity-based sets (for example "nodes already visited").

**DT-134. Taking the address of a local is always safe in Go.** *(Guarantee)* `func newCounter() *int { n := 0; return &n }` is correct: escape analysis moves `n` to the heap (DT-34). The same code in C returns a pointer into a dead stack frame.

### Common mistakes

- Dereferencing a pointer that may be nil.
- Taking `&v` of a range copy and expecting it to point into the collection.
- Holding element pointers across `append`.
- Using a pointer just to "avoid copying" a small struct (Unit 2.12 explains why that is often slower).

### Mastery check

1. **Explain.** What exactly is copied when you pass a `*Account` to a function, and why does the function's change become visible to the caller?
2. **Predict.** (a) `a := 1; p := &a; q := p; *q = 5; fmt.Println(a)` (b) `var p *int; fmt.Println(p == nil, *p)` (c) `s := []int{1}; p := &s[0]; s = append(s, 2); *p = 9; fmt.Println(s)`
3. **Write.** `swap(a, b *int)`; a function `maxPtr(nums []int) *int` that returns a pointer to the largest element (nil for an empty slice) and lets the caller modify it in place.

## Unit 2.12 — Value vs pointer semantics

**One-line summary:** Choosing between passing `T` and `*T` is choosing between *copying* and *sharing*. Decide by meaning first (should the callee change it? does identity matter?), and by cost second (size, allocations, cache locality). Returning a pointer to a local is safe; whether it costs a heap allocation is decided by escape analysis.

### Two kinds of types

Most types naturally have one kind of semantics:

| Value semantics: copies are independent and fine | Pointer semantics: there is one "real" thing, shared |
| --- | --- |
| `time.Time`, a `Point`, a `Money` amount, a small config | a database connection pool, a bank account being updated, a `bytes.Buffer` being written, anything containing a mutex |
| Pass `T`, return `T` | Pass `*T`, return `*T` |

Rule of thumb from experienced Go teams: **decide once per type, then be consistent.** If any method needs to modify the value or the type contains a `sync.Mutex`, use pointers everywhere for that type (Unit 3.3 makes this precise with method sets).

### Questions to decide

1. **Must the callee modify the caller's value?** Yes → pointer (or return the new value).
2. **Does identity matter** — is there exactly one of this thing (a connection, a cache)? Yes → pointer.
3. **Must copies never happen** — contains a `sync.Mutex`, `sync.WaitGroup`, or `strings.Builder`? Yes → pointer (`go vet`'s copylocks check catches copies).
4. **Is it big?** A struct of many hundreds of bytes copied in a hot loop costs time. Pointer may help — but measure.
5. **Is "absent" a meaningful state?** `*T` can be nil; `T` cannot. Sometimes this is useful (optional fields), sometimes it is a source of nil panics.

### Why pointers are not automatically faster

Passing a 24-byte struct by value copies 24 bytes, usually in registers (DT-38): extremely cheap. Passing a pointer is 8 bytes, but:

- if the pointed-to value **escapes** to the heap, it costs an allocation plus GC work (Unit 0.6);
- every access goes through an extra indirection, possibly a cache miss;
- the compiler is more conservative about optimising code with pointers (it must consider aliasing).

So `[]Point` (values side by side in memory) is usually much faster to scan than `[]*Point` (pointers to objects scattered across the heap).

### Seeing escape analysis

```go
package main

import "fmt"

type Point struct{ X, Y int }

//go:noinline
func byValue() Point { p := Point{1, 2}; return p }

//go:noinline
func byPointer() *Point { p := Point{1, 2}; return &p }

func main() {
	a := byValue()
	b := byPointer()
	fmt.Println(a, *b)
}
```

```
$ go build -gcflags=-m .
./main.go:14:27: moved to heap: p
./main.go:19:13: ... argument does not escape
./main.go:19:14: a escapes to heap
./main.go:19:17: *b escapes to heap
```

(Some lines about inlining are omitted.) The `//go:noinline` directives stop the compiler from inlining the two functions into `main`, which would let it prove that `p` does not really escape. Three things to read here:

- `moved to heap: p` on line 14: `byPointer` returns `&p`, so `p` must outlive the call — a heap allocation on every call. `byValue` has no such line: its `p` is copied out (Unit 0.6).
- `a escapes to heap` and `*b escapes to heap`: passing values to `fmt.Println` converts them to interfaces (Unit 3.5), which, for values that are not tiny, means a copy on the heap. Printing is not free.
- The messages depend on the Go version and on inlining decisions. Treat `-gcflags=-m` as a microscope, not as a specification.

### Constructors return pointers or values?

Look at the standard library: `bytes.NewBuffer` returns `*Buffer` (shared, mutable), `time.Now` returns `Time` (a value), `errors.New` returns `error` (an interface holding a pointer). Follow the semantics of the type, not a blanket rule.

### Deep & Tricky

**DT-135. "Pass a pointer to avoid copying" can make code slower.** *(Performance fact; measure)* The copy saved is often a few words in registers; the cost added can be a heap allocation per call and GC pressure. Profiles of real Go services frequently show allocations from needless pointers as a top cost. Default to values for small types (rule of thumb: a few words), and change only with benchmark evidence (Chapter 10).

**DT-136. `go vet` reports copied locks.** *(Tool behaviour)* Passing or assigning a struct containing a `sync.Mutex` by value creates a second, independent lock — the protection silently disappears. `go vet`'s `copylocks` analyzer flags `passes lock by value` and `assignment copies lock value`. Types with locks must use pointer semantics.

### Pattern

**P-22. Pick value or pointer semantics per type, and stay consistent.** *(Go design practice)*

- *Problem it solves:* mixing `T` and `*T` for the same type causes lost updates (a method modified a copy) and confusing APIs.
- *How:* small, immutable-feeling data (coordinates, money, timestamps, IDs) → values. Things with identity, internal state, or locks → pointers. All methods and functions for that type follow the choice.
- *When not to:* the occasional function that legitimately needs a private copy of a pointer-semantics type should make the copy explicit (a `Clone` method).
- *Trade-off:* values give safety (no unexpected sharing) and locality; pointers give shared mutation and avoid copying big objects, at the cost of possible nil, aliasing, and heap allocations.

### Mastery check

1. **Explain.** Give two reasons why `[]Point` is usually faster to iterate than `[]*Point`.
2. **Explain.** Why is it safe in Go to return `&local`, and what does it cost?
3. **Write.** Two versions of `func (move) ...` for a `Point` — one taking and returning `Point`, one taking `*Point` — and a benchmark plan (written now, run in Chapter 4) comparing them in a loop of 10 million calls. Predict which wins and why.

## Chapter 2 Project — `wordfreq`, a Unicode-correct word counter

**Goal:** read text from files or standard input, count words correctly in any language, and print the most frequent words in a stable order. This project exercises slices, strings and runes, maps, structs, sorting, and careful input handling.

**Specification**

```
wordfreq [N] [file ...]
```

- N (optional, default 10) is how many words to print; it must be positive.
- With no files, read standard input; otherwise read each file in turn.
- A **word** is a run of letters, digits, and combining marks, optionally joined by an apostrophe (`'` or `’`) or hyphen *between* two word characters: `don't`, `well-known`. Leading or trailing apostrophes and hyphens are not part of words.
- Counting is case-insensitive (output in lower case).
- Output: a header line `T words, D distinct`, then the top N words with counts, highest count first; ties in alphabetical (byte) order, so output is identical on every run (P-21). Counts aligned in a column.
- Errors (unreadable file, bad N, read error, extremely long line) go to stderr with exit code 1, and keep the original error text (it already names the file).

**Requirements checklist**

1. `run(args []string, stdin io.Reader, stdout io.Writer) error` holds all logic, so tests can call it with fake input (P-14).
2. Splitting is done by a separate, tested function `splitWords(line string) []string`.
3. Handles Hindi (`नमस्ते दुनिया` is two words), accented Latin, and digits.
4. The `bufio.Scanner` buffer is raised so lines up to 16 MiB work (DT-92), and `sc.Err()` is checked.
5. Sorting uses `slices.SortFunc` with a tie-breaker; the map is never ranged for output directly.
6. Includes a table-driven test of `splitWords` with at least six cases (empty input, punctuation, apostrophes, hyphens, Hindi, mixed case).

**Hints**

- `unicode.IsLetter` returns false for Devanagari vowel signs such as `े` and the virama `्`: they are *marks*. Without `unicode.IsMark` your Hindi words fall apart into pieces. Find this out by testing first, then read about Unicode categories.
- To check "the next character is a word character", you need to look one rune ahead; converting the line to `[]rune` makes that simple.
- The `io.Reader` and `io.Writer` types are explained fully in Chapter 5; for now: `os.Stdin`, files, and `strings.NewReader` are readers; `os.Stdout` and `*strings.Builder` are writers.

**Reference solution**

```go
// Command wordfreq counts words in its input and prints the most frequent.
//
//	wordfreq [N] [file ...]
//
// With no files it reads standard input. N (default 10) is how many
// words to print. Words are runs of letters, digits, and combining marks,
// optionally joined by an apostrophe or hyphen ("don't", "well-known").
// Counting is case-insensitive.
package main

import (
	"bufio"
	"cmp"
	"errors"
	"fmt"
	"io"
	"os"
	"slices"
	"strconv"
	"strings"
	"unicode"
	"unicode/utf8"
)

func main() {
	if err := run(os.Args[1:], os.Stdin, os.Stdout); err != nil {
		fmt.Fprintln(os.Stderr, "wordfreq:", err)
		os.Exit(1)
	}
}

type entry struct {
	word  string
	count int
}

func run(args []string, stdin io.Reader, stdout io.Writer) error {
	top := 10
	if len(args) > 0 {
		if n, err := strconv.Atoi(args[0]); err == nil {
			if n <= 0 {
				return fmt.Errorf("N must be positive, got %d", n)
			}
			top = n
			args = args[1:]
		}
	}

	counts := make(map[string]int)
	total := 0
	if len(args) == 0 {
		n, err := countWords(stdin, counts)
		if err != nil {
			return fmt.Errorf("stdin: %w", err)
		}
		total += n
	}
	for _, name := range args {
		n, err := countFile(name, counts)
		if err != nil {
			return err
		}
		total += n
	}

	entries := make([]entry, 0, len(counts))
	for w, c := range counts {
		entries = append(entries, entry{w, c})
	}
	// Highest count first; ties in alphabetical order, so output is stable.
	slices.SortFunc(entries, func(a, b entry) int {
		if c := cmp.Compare(b.count, a.count); c != 0 {
			return c
		}
		return strings.Compare(a.word, b.word)
	})
	if len(entries) > top {
		entries = entries[:top]
	}

	width := 0
	for _, e := range entries {
		width = max(width, utf8.RuneCountInString(e.word))
	}
	fmt.Fprintf(stdout, "%d words, %d distinct\n", total, len(counts))
	for _, e := range entries {
		pad := strings.Repeat(" ", width-utf8.RuneCountInString(e.word))
		fmt.Fprintf(stdout, "%s%s  %d\n", e.word, pad, e.count)
	}
	return nil
}

func countFile(name string, counts map[string]int) (int, error) {
	f, err := os.Open(name)
	if err != nil {
		return 0, err // the error already contains the file name
	}
	defer f.Close()
	n, err := countWords(f, counts)
	if err != nil {
		return n, fmt.Errorf("%s: %w", name, err)
	}
	return n, nil
}

// countWords adds every word read from r to counts and returns how many
// words it saw.
func countWords(r io.Reader, counts map[string]int) (int, error) {
	sc := bufio.NewScanner(r)
	sc.Buffer(make([]byte, 0, 64*1024), 16*1024*1024)
	total := 0
	for sc.Scan() {
		for _, w := range splitWords(sc.Text()) {
			counts[w]++
			total++
		}
	}
	if err := sc.Err(); err != nil {
		if errors.Is(err, bufio.ErrTooLong) {
			return total, fmt.Errorf("a line is longer than 16 MiB: %w", err)
		}
		return total, err
	}
	return total, nil
}

func isWordRune(r rune) bool {
	// unicode.IsMark matters: Devanagari vowel signs (like the "े" in "नमस्ते")
	// are marks, not letters. Without it, Hindi words fall apart.
	return unicode.IsLetter(r) || unicode.IsDigit(r) || unicode.IsMark(r)
}

func isJoiner(r rune) bool {
	return r == '\'' || r == '’' || r == '-'
}

// splitWords returns the lower-cased words in line.
func splitWords(line string) []string {
	var words []string
	runes := []rune(line)
	var b strings.Builder
	flush := func() {
		if b.Len() > 0 {
			words = append(words, b.String())
			b.Reset()
		}
	}
	for i, r := range runes {
		switch {
		case isWordRune(r):
			b.WriteRune(unicode.ToLower(r))
		case isJoiner(r) && b.Len() > 0 && i+1 < len(runes) && isWordRune(runes[i+1]):
			// keep an apostrophe or hyphen only between two word characters
			if r == '’' {
				r = '\''
			}
			b.WriteRune(r)
		default:
			flush()
		}
	}
	flush()
	return words
}
```

**Tests (`main_test.go`)**

```go
package main

import (
	"slices"
	"strings"
	"testing"
)

func TestSplitWords(t *testing.T) {
	tests := []struct {
		in   string
		want []string
	}{
		{"Hello, hello WORLD!", []string{"hello", "hello", "world"}},
		{"don't stop - well-known", []string{"don't", "stop", "well-known"}},
		{"नमस्ते दुनिया", []string{"नमस्ते", "दुनिया"}},
		{"Ünïcödé café", []string{"ünïcödé", "café"}},
		{"'quoted' -dash-", []string{"quoted", "dash"}},
		{"", nil},
	}
	for _, tt := range tests {
		if got := splitWords(tt.in); !slices.Equal(got, tt.want) {
			t.Errorf("splitWords(%q) = %q, want %q", tt.in, got, tt.want)
		}
	}
}

func TestRun(t *testing.T) {
	in := strings.NewReader("b a b c b a\n")
	var out strings.Builder
	if err := run([]string{"2"}, in, &out); err != nil {
		t.Fatal(err)
	}
	want := "6 words, 3 distinct\nb  3\na  2\n"
	if out.String() != want {
		t.Errorf("got %q, want %q", out.String(), want)
	}
}
```

A real run:

```
$ printf "The cat and the hat.\nThe CAT sat; the hat didn't.\nनमस्ते नमस्ते दुनिया\n" | ./wordfreq 4
14 words, 8 distinct
the     4
cat     2
hat     2
नमस्ते  2
```

**Review checklist**

- Is output stable across runs (no direct map iteration for output)? Run the program five times and `diff` the outputs.
- Do Hindi and accented words survive intact? Does `'quoted'` lose its outer apostrophes?
- Is the error from `os.Open` returned as-is (it already contains the path), rather than wrapped with the path a second time?
- Is `sc.Err()` checked, and does a 1 MB single-line file work?
- **Known limitation, worth discussing:** the column alignment counts runes, but the *display width* of text is not its rune count: Devanagari combining marks take no column, and Chinese characters and many emoji take two. Correct terminal alignment needs a width library (for example `golang.org/x/text/width` plus grapheme segmentation). Knowing where your solution is approximate is part of engineering.

**Extensions (optional)**

1. Add a `-stop` file of stop words to ignore (a set, Unit 2.8).
2. Print bigrams (pairs of consecutive words) as well; what key type will you use?
3. Process a 1 GB file and measure memory with `/usr/bin/time -v` (Linux) or `/usr/bin/time -l` (macOS). Where does the memory go?

## Answers to the Chapter 2 prediction questions

**2.2 Q2.** (a) `2 4` (b) `[2 3 4 5]` (c) panic: index out of range [2] with length 2 (d) panic: slice bounds out of range [:5] with capacity 4 (e) `[0 1 9 3 4 5]`.

**2.3 Q2.** `[0 0 0] [0 0 0 2] [0 0 0 2]` — both appends had room (cap 4), so both wrote into the *same* slot `a[3]`; the second write (2) overwrote the first (1), and `b` and `c` share it. **Q3.** First version: `[1 9 3]` (the append wrote into `s[1]`). With `s[:1:1]`: `[1 2 3]` (the append had to allocate).

**2.4 Q2.** Prints `1 2 3 4`. The `append` had no capacity left (cap 4), so it allocated a new array; `s[3] = 40` wrote into the new array, but `range` keeps using the original header and array. After the loop `s` is `[1 2 3 40 5]`.

**2.5 Q2.** (a) 6 (b) 3 (c) 240 (the first byte of the emoji, 0xF0) (d) `"Go\xf0"` (e) `"🙂"`.

**2.6 Q2.** (a) `["a" "b" "" "c"]` (b) 1 (c) `"hixx"` (d) `[a b]` (e) error: value out of range (f) `false` — simple case folding does not map `ß` to `ss` (DT-117).

**2.7 Q2.** (a) 0 (b) panic: assignment to entry in nil map (c) `0 true` (d) no-op (e) `math.NaN()` is a function call, not a constant, so the literal compiles; the result is 1.

**2.8 Q2.** 1. `add` received a copy of the map *pointer*; it wrote through it (visible), then set *its own copy* to nil (invisible to the caller).

**2.9 Q2.** (a) `true` (b) compile error: struct containing `[]int` cannot be compared (c) panic: assignment to entry in nil map.

**2.10 Q2.** (a) 12 (b) 8 (c) 24 (d) 0.

**2.11 Q2.** (a) 5 (b) `true`, then panic: nil pointer dereference (c) `[1 2]` — the append reallocated, so `p` points into the old array (DT-132).

# Chapter 3 — Types, Methods, Interfaces, Errors, Packages, Generics

*Goal: organise programs with clean abstractions and clear boundaries.*

Chapters 1 and 2 taught the pieces. This chapter teaches how to *organise* them: your own types with behaviour (methods), contracts between parts of a program (interfaces), composition instead of inheritance (embedding), failure as ordinary values (errors), code in packages and modules, and code that works for many types (generics and iterators). These are the tools senior Go engineers use to keep a 200,000-line codebase understandable.

Go's approach is deliberately small. There are no classes, no inheritance, no exceptions, no annotations, no constructors as a language feature. Each of those absences is a design decision, and you will learn what replaces each one and why.

**What you will be able to do after this chapter**

- Define named types and enums with methods, and choose value or pointer receivers correctly.
- Design small interfaces, explain interface values (and the nil-interface trap), and use type switches.
- Handle errors like a professional: wrap with context, inspect with `errors.Is` / `errors.As`, and know when to panic.
- Split a program into packages with a clean public API, and manage dependencies with modules.
- Write generic functions and types, and your own iterators — and know when *not* to.

## Topic 3A — Your own types

## Unit 3.1 — Named types

**One-line summary:** `type Celsius float64` creates a brand-new type with the same underlying representation as `float64` but a different identity: the compiler will not mix it with `float64` or with `Fahrenheit` unless you convert explicitly. Only named types defined in your package can have methods.

**Mental model — the same steel, different moulds.** Two containers made from the same steel (the **underlying type**) are still different products if they come from different moulds (different **named types**): a milk can and a petrol can. You can melt one and recast it as the other (a **conversion**), but you cannot pour petrol into the milk can by accident.

### Type definitions

```go
type Celsius float64
type Fahrenheit float64
type UserID int64
type Path []string

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

var t Celsius = 36.6
var f Fahrenheit = CToF(t) // OK
// f = t                   // compile error: cannot use t (variable of type Celsius) as Fahrenheit value
f = Fahrenheit(t)          // compiles — but is semantically wrong! Conversion only changes the label.
```

- The **underlying type** of `Celsius` is `float64`. Operators that work on the underlying type work on the named type: `t + 1`, `t * 2`, `t < 40`.
- Arithmetic between two `Celsius` values gives a `Celsius`. Between `Celsius` and `float64` *variables*: type error. Untyped constants adapt as usual (Unit 1.2).
- **Conversion** `T(x)` is allowed between types with the same underlying type. It never changes bits, so it cannot change meaning — `Fahrenheit(t)` of 36.6 °C is 36.6 °F, which is wrong. Use functions like `CToF` for real unit changes.

Named types catch whole classes of bugs at compile time: passing a `UserID` where an `OrderID` is expected, mixing metres and feet, or seconds and milliseconds (the standard library's `time.Duration` is `type Duration int64`).

### Type aliases

```go
type MyByte = byte // an ALIAS: MyByte and byte are the SAME type
```

An alias (note the `=`) is just a second name; it creates no new type and adds no type safety. Aliases exist mainly for gradual refactoring (moving a type between packages without breaking callers). `byte` = `uint8` and `rune` = `int32` are aliases (DT-15), and `any` = `interface{}`. Since Go 1.24, aliases may have type parameters (`type Set[T comparable] = map[T]struct{}`).

### Defined types vs type literals

`[]string`, `map[string]int`, `func(int) bool` are **type literals** (unnamed types). A value of an unnamed type can be assigned to a variable of a named type with the same underlying type without conversion:

```go
type Path []string
var p Path = []string{"usr", "local"} // OK: []string is unnamed
var q []string = p                    // OK too: one side is unnamed
```

This is why `sort.Sort(sort.StringSlice(names))` and `http.HandlerFunc(f)` work smoothly.

### Methods need a defined type in your package

You can declare methods only on types defined in the same package (Unit 3.3). You cannot add a method to `int` or to `time.Time`; you can define `type Celsius float64` and add methods to that.

### Deep & Tricky

**DT-137. A named type does not inherit the methods of its underlying named type.** *(Guarantee)* `type MyDuration time.Duration` has the *representation* of `time.Duration` but **none** of its methods: `MyDuration(5).String()` does not compile. (The method set of an *embedded* type is promoted — Unit 3.7 — but a type definition is not embedding.) An alias, by contrast, keeps everything, because it *is* the same type.

**DT-138. Typed constants of a named type give you compile-time checked "units".** *(Guarantee)* `time.Second` is a constant of type `time.Duration`. `time.Sleep(2)` compiles (untyped 2 becomes 2 nanoseconds!) but `time.Sleep(n)` with an `int` variable does not; you must write `time.Duration(n) * time.Second`. That bit of friction prevents seconds-vs-nanoseconds bugs everywhere except with bare constants, which is why `time.Sleep(2)` is a classic mistake.

### Common mistakes

- Using conversion to change units (`Fahrenheit(c)`).
- `time.Sleep(5)` meaning 5 seconds (it is 5 ns).
- Expecting a defined type to keep the methods of the type it was defined from.

### Mastery check

1. **Explain.** What is the difference between `type A = int` and `type A int`? Give a reason to use each.
2. **Predict.** (a) `type Meters float64; var m Meters = 3; fmt.Println(m * 2)` (b) `var f float64 = 1; m = m + f` (c) `type D time.Duration; fmt.Println(D(time.Second))` (d) `time.Sleep(2)` — how long?
3. **Write.** Types `Rupees` and `Paise` (int64) with conversion functions, and show at least one bug that the compiler now prevents.

## Unit 3.2 — Enums in Go

**One-line summary:** Go has no `enum` keyword. The idiom is a named integer type plus a `const` block with `iota`, plus a `String()` method so values print as names. You must handle invalid values yourself.

### The idiom

```go
type Weekday int

const (
	Sunday Weekday = iota // 0
	Monday                // 1, type Weekday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)

func (d Weekday) String() string {
	names := [...]string{"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}
	if d < 0 || int(d) >= len(names) {
		return fmt.Sprintf("Weekday(%d)", int(d))
	}
	return names[d]
}

fmt.Println(Tuesday)          // Tuesday   — fmt calls String() automatically (Unit 3.4)
fmt.Printf("%d\n", Tuesday)   // 2
```

Repeating an expression in a `const` block also repeats its *type*, so every constant is a `Weekday`. That gives type safety: a function `func schedule(d Weekday)` will not accept a stray `int` variable.

### Make the zero value mean "unknown"

Every variable starts at its zero value (DT-43). If 0 is a real choice (`Sunday`), you cannot tell "deliberately Sunday" from "forgot to set it". Common fix:

```go
type Status int

const (
	StatusUnknown Status = iota // zero value: not set
	StatusActive
	StatusSuspended
	StatusClosed
)
```

### Invalid values are possible

Nothing stops `Status(42)` or a value decoded from a database or JSON. Defend at the edges:

```go
func (s Status) Valid() bool { return s > StatusUnknown && s <= StatusClosed }

func ParseStatus(text string) (Status, error) { ... } // the only way in from outside
```

A `switch` over an enum should have a `default` that handles (or reports) unexpected values. The `exhaustive` linter checks that every named constant appears in a switch.

### Generating `String()` with `stringer`

Writing name tables by hand is error-prone when constants change. The `stringer` tool generates them:

```go
//go:generate go run golang.org/x/tools/cmd/stringer -type=Status
```

`go generate ./...` then writes `status_string.go`. The generated code even includes a compile-time check that breaks the build if the constants change without regenerating.

### Bit-flag enums

When values can combine, use `1 << iota` (Unit 1.4, P-7) on a named type:

```go
type Perm uint8

const (
	PermRead Perm = 1 << iota
	PermWrite
	PermExec
)

func (p Perm) Has(q Perm) bool { return p&q == q }
```

### Deep & Tricky

**DT-139. `%v` on a type with a `String()` method that formats itself with `%v` recurses forever.** *(fmt behaviour)* `func (s Status) String() string { return fmt.Sprintf("%v", s) }` calls itself through fmt until the stack overflows (`fatal error: stack overflow`). Inside `String`, convert to the underlying type first: `fmt.Sprintf("Status(%d)", int(s))`. `go vet` catches the most direct form of this.

**DT-140. Untyped constants still sneak through.** *(Guarantee)* With `func schedule(d Weekday)`, the call `schedule(42)` compiles, because 42 is an untyped constant representable as `Weekday`. Only *variables* of other types are rejected. Validation at run time is still required.

### Mastery check

1. **Explain.** Why should the zero value of an enum usually mean "unknown"? What bug does it prevent?
2. **Predict.** `type Size int; const ( Small Size = iota + 1; Medium; Large )` — values? `fmt.Println(Size(0))` without a `String` method?
3. **Write.** A `Priority` enum (Low, Medium, High, Critical) with `String`, `ParsePriority` (case-insensitive), and `Valid`; a test that round-trips every value through `String` and `ParsePriority`.

## Topic 3B — Methods and interfaces

## Unit 3.3 — Methods

**One-line summary:** A method is a function with a **receiver**: `func (a Account) Balance() int64`. A value receiver works on a copy; a pointer receiver works on the original. Which methods a type "has" (its **method set**) depends on whether you hold a value or a pointer, and that decides which interfaces it satisfies.

**Mental model — a form vs a pointer to the form.** A value-receiver method is handed a photocopy of the form: it can read everything, but scribbles on the copy are lost. A pointer-receiver method is handed the address of the original form: its changes stick.

### Declaring methods

```go
type Account struct {
	owner   string
	balance int64 // paise
}

func (a Account) Balance() int64 { return a.balance }   // value receiver: reads a copy

func (a *Account) Deposit(amt int64) {                  // pointer receiver: changes the original
	a.balance += amt
}

acc := Account{owner: "Asha"}
acc.Deposit(500)            // Go automatically takes &acc because acc is addressable
fmt.Println(acc.Balance())  // 500
```

A method is just a function whose first parameter is written before the name. In fact `acc.Deposit(500)` is the same call as `(*Account).Deposit(&acc, 500)`.

### Value or pointer receiver?

Use a **pointer receiver** when:

- the method modifies the receiver;
- the type contains something that must not be copied (`sync.Mutex`, DT-136);
- the struct is large and copying it on every call is wasteful;
- for consistency: **if any method needs a pointer receiver, give all methods pointer receivers** (P-22).

Use a **value receiver** for small, immutable-feeling types (`time.Time`, `Celsius`, `Point`), where copies are cheap and safe, and for maps, functions, and channels (already reference-like).

### Automatic `&` and `*` — and where it fails

Go inserts `&` or `*` for you when calling methods:

- `v.PtrMethod()` becomes `(&v).PtrMethod()` — **only if `v` is addressable** (a variable, a field of an addressable struct, a slice element, `*p`).
- `p.ValueMethod()` becomes `(*p).ValueMethod()` — always allowed for a pointer `p`.

Non-addressable values cannot call pointer methods:

```go
m := map[string]Account{"a": {}}
m["a"].Deposit(5)      // compile error: cannot call pointer method Deposit on Account
Account{}.Deposit(5)   // compile error: same reason (a composite literal value is not addressable)
```

### Method sets: the rule that decides interfaces

| Type | Its method set contains |
| --- | --- |
| `T` | methods with receiver `T` |
| `*T` | methods with receiver `T` **and** `*T` |

So if `Deposit` has a pointer receiver, the type `Account` does *not* have `Deposit` in its method set; only `*Account` does. This matters as soon as interfaces appear (Unit 3.4): a `*Account` can satisfy an interface requiring `Deposit`; an `Account` cannot. The reason: an interface may hold a copy of an `Account` value that is not addressable, so the compiler could not take its address safely.

### Methods on any defined type

Receivers do not have to be structs:

```go
type Celsius float64
func (c Celsius) String() string { return fmt.Sprintf("%.1f°C", float64(c)) }

type IntSet map[int]struct{}
func (s IntSet) Add(n int) { s[n] = struct{}{} } // value receiver is fine: the map is shared

type HandlerFunc func(string)
func (f HandlerFunc) Handle(s string) { f(s) }   // methods on a function type (net/http does this)
```

### Method values and method expressions

```go
acc := &Account{}
dep := acc.Deposit      // method VALUE: bound to acc — a func(int64)
dep(100)                // acc.balance == 100

f := (*Account).Deposit // method EXPRESSION: receiver becomes the first argument — func(*Account, int64)
f(acc, 50)
```

Method values are handy for callbacks: `defer mu.Unlock()`, `time.AfterFunc(d, srv.Shutdown)`, `slices.SortFunc(s, cmpByName)`.

### Nil receivers are legal

A pointer-receiver method can be called on a nil pointer; it only panics if it dereferences it. Some types use this deliberately:

```go
type Tree struct{ Left, Right *Tree; Val int }

func (t *Tree) Sum() int {
	if t == nil {
		return 0
	}
	return t.Val + t.Left.Sum() + t.Right.Sum()
}
```

### Deep & Tricky

**DT-141. A method value evaluates (copies) its receiver immediately.** *(Guarantee)* With a value receiver, `g := c.Get` copies `c` at that moment; later changes to `c` are invisible to `g`. With a pointer receiver, `inc := c.Inc` stores `&c`, so `inc()` changes `c`. This is the same "photo vs window" distinction as `defer` (DT-88).

**DT-142. Map elements and return values are not addressable, so they cannot call pointer methods.** *(Guarantee)* `m[k].Deposit(1)` and `getAccount().Deposit(1)` are compile errors. Store pointers in the map, or copy out, modify, and store back.

**DT-143. Mixing value and pointer receivers creates "lost update" bugs.** *(Classic bug)* If `Deposit` mistakenly has a value receiver, `acc.Deposit(500)` compiles, runs, modifies a copy, and the balance stays 0. Nothing warns you. Consistent pointer receivers for mutable types prevent it.

### Mastery check

1. **Explain.** State the method-set rule. Why can a `*T` call value-receiver methods but a non-addressable `T` cannot call pointer-receiver methods?
2. **Predict.** `type C struct{ n int }; func (c C) Get() int { return c.n }; func (c *C) Inc() { c.n++ }` then `c := C{1}; g := c.Get; c.n = 50; fmt.Println(g()); inc := c.Inc; inc(); fmt.Println(c.n)`.
3. **Write.** A `Counter` type with `Inc`, `Add(n)`, `Value`, and `Reset`. Write one version with a value receiver on `Inc` by mistake, show the bug in a test, then fix it.

## Unit 3.4 — Interfaces I

**One-line summary:** An interface is a set of method signatures. Any type with those methods satisfies the interface automatically — there is no `implements` keyword. Small interfaces defined where they are *used* are the backbone of decoupled, testable Go code.

**Mental model — a job description, not a family tree.** A job ad for "driver" says: *must be able to `Drive()` and `Park()`*. Anyone with those skills qualifies, whatever their background. The ad (interface) and the people (types) never need to know about each other in advance.

### Declaring and satisfying

```go
type Shape interface {
	Area() float64
	Perimeter() float64
}

type Rect struct{ W, H float64 }
func (r Rect) Area() float64      { return r.W * r.H }
func (r Rect) Perimeter() float64 { return 2 * (r.W + r.H) }

type Circle struct{ R float64 }
func (c Circle) Area() float64      { return math.Pi * c.R * c.R }
func (c Circle) Perimeter() float64 { return 2 * math.Pi * c.R }

func totalArea(shapes []Shape) float64 {
	sum := 0.0
	for _, s := range shapes {
		sum += s.Area() // dynamic dispatch: calls Rect.Area or Circle.Area
	}
	return sum
}

totalArea([]Shape{Rect{2, 3}, Circle{1}})
```

`Rect` never mentions `Shape`. It satisfies `Shape` because it has the methods. This is **implicit** (or **structural**) satisfaction. Consequence: you can define an interface *after the fact* that existing types — even types from other packages you cannot change — already satisfy.

### The most important interfaces in the standard library

```go
type Stringer interface { String() string }                   // fmt uses it for %v and Println
type error interface { Error() string }                        // Unit 3.9
type Reader interface { Read(p []byte) (n int, err error) }    // io: anything you can read bytes from
type Writer interface { Write(p []byte) (n int, err error) }   // io: anything you can write bytes to
```

`io.Reader` and `io.Writer` (Chapter 5) are satisfied by files, network connections, buffers, HTTP bodies, compressors, hashers, and more. A function that accepts an `io.Reader` works with all of them — which is how `wordfreq` (Chapter 2 project) could be tested with `strings.NewReader`.

### Small interfaces

"The bigger the interface, the weaker the abstraction." — Go proverb. One- or two-method interfaces (`Reader`, `Writer`, `Stringer`, `error`, `sort.Interface` has three) are easy to satisfy, easy to fake in tests, and compose well. An interface with fifteen methods has one implementation and one mock and abstracts nothing.

### Define interfaces where they are used

In Java, the type author declares the interface. In Go, the **consumer** usually does:

```go
// package report — needs only to fetch orders
type OrderSource interface {
	OrdersSince(t time.Time) ([]Order, error)
}

func Weekly(src OrderSource) (Summary, error) { ... }
```

The database package exports a concrete `*Store` with twenty methods; `report` declares the one method it needs. Tests pass a tiny fake. The database package does not even know `report` exists. The library project in this chapter does exactly this with `catalog.Store`.

### Accept interfaces, return concrete types

A common guideline: functions should *accept* the narrowest interface they need (flexible for callers), and *return* concrete types (callers get all methods and fields, and you can add methods later without breaking anyone). Exceptions exist — `error` is always returned as an interface, and some constructors return interfaces to hide implementations — but it is a strong default.

### Compile-time satisfaction check

Because satisfaction is implicit, a typo in a method name silently means "does not implement". Add an assertion that fails the build:

```go
var _ Shape = Rect{}            // value type must satisfy Shape
var _ io.Writer = (*MyWriter)(nil) // pointer type must satisfy io.Writer
```

### Deep & Tricky

**DT-144. Value vs pointer receivers decide which types satisfy an interface.** *(Guarantee, from method sets)* If `Area` had a pointer receiver, `Shape(Rect{2, 3})` would not compile (`Rect does not implement Shape (method Area has pointer receiver)`), while `Shape(&Rect{2, 3})` would. This is the method-set rule of Unit 3.3 in action.

**DT-145. `fmt` checks for `error` before `Stringer`.** *(fmt behaviour)* If a type implements both `Error()` and `String()`, `%v` prints `Error()`. Also, `fmt` recovers panics inside `String` and `Error` methods and prints them as `%!v(PANIC=String method: ...)` — which is why a nil-pointer panic in a `String` method can appear as odd text in logs instead of a crash.

**DT-146. Interfaces are satisfied by method *names and signatures*, not meaning.** *(Language design)* Any type with `Read([]byte) (int, error)` is an `io.Reader`, whether or not it follows `io.Reader`'s documented contract (for example: return `n > 0` bytes before an error, never return `0, nil` repeatedly). The compiler checks shape; tests and documentation must check behaviour.

### Pattern

**P-23. Consumer-defined, minimal interfaces.** *(Core Go design practice)*

- *Problem it solves:* tight coupling between packages and hard-to-test code that depends on concrete databases, clocks, or network clients.
- *How:* in the package that *uses* a dependency, declare an interface with only the methods it calls. Accept that interface. Provide the real implementation in `main` (wiring) and a fake in tests.
- *When not to:* do not create an interface for every type "just in case". If there is one implementation and no test need, a concrete type is simpler. Interfaces are discovered, not designed up front.
- *Trade-off:* a level of indirection (and a tiny call cost) in exchange for decoupling and testability.

### Mastery check

1. **Explain.** What does "implicit satisfaction" let you do that Java-style `implements` does not?
2. **Explain.** Why should the *consumer* define the interface? Use the `report`/database example.
3. **Write.** An interface `Notifier { Notify(to, msg string) error }`, two implementations (`EmailNotifier` that prints, `RecordingNotifier` that stores messages in a slice for tests), and a function `remindOverdue(n Notifier, ...)` tested with the recording one. Add compile-time checks.

## Unit 3.5 — Interfaces II: interface values

**One-line summary:** An interface value is a pair: **(dynamic type, dynamic value)**. It is nil only when *both* are absent. Storing a nil pointer in an interface gives a non-nil interface — the most famous trap in Go.

**Mental model — a labelled box.** An interface variable is a box with a label slot and a content slot. An empty box (no label, no content) is `nil`. A box with the label "`*MyErr`" and empty content is *not* empty — it has a label. `box == nil` asks "is the box completely empty?", and a labelled box answers no.

### The two words

```go
var s fmt.Stringer        // (nil, nil) — a nil interface
s = Celsius(36.6)         // (Celsius, 36.6)
s = &Rect{}               // (*Rect, pointer to a Rect)
```

In memory (current implementation) an interface value is two words: a pointer to type information (including the method table for this type/interface pair) and a pointer to the data (or, for some small values, the data itself). That is why `unsafe.Sizeof` of an interface is 16 on 64-bit platforms (Unit 2.10). Calling `s.String()` looks up the method in the table and calls it: **dynamic dispatch**.

### The nil-interface trap

```go
type MyErr struct{}
func (*MyErr) Error() string { return "my" }

func find() error {
	var p *MyErr // nil pointer
	// ... nothing went wrong ...
	return p     // returns (type *MyErr, value nil) — NOT a nil error!
}

err := find()
fmt.Println(err == nil) // false
fmt.Println(err)        // my   — the method still runs on a nil receiver
```

The interface has a dynamic type (`*MyErr`), so it is not nil, even though the pointer inside is nil. Callers check `if err != nil`, see an error, and report a failure that never happened.

**The rule:** never return a typed nil pointer as an interface. Return a literal `nil` in the success path:

```go
func find() error {
	if somethingWrong {
		return &MyErr{}
	}
	return nil // untyped nil → (nil, nil)
}
```

Watch especially for functions whose *result variable* is a concrete pointer type (`var err *MyErr`) that is later returned as `error`.

### `any` (the empty interface)

`interface{}` has no methods, so every type satisfies it. Since Go 1.18 it has the alias `any`. `fmt.Println(a ...any)` accepts anything because of it. Using `any` throws away static type information: to get the value back you need a type assertion (Unit 3.6). Prefer concrete types or generics (Unit 3.14) where possible; use `any` for genuinely heterogeneous data (decoded JSON, printing, containers before generics existed).

### Comparing interface values

Two interface values are equal if their dynamic types are identical and their dynamic values are equal. If the dynamic type is not comparable (a slice, map, or function), the comparison **panics at run time**:

```go
var a, b any = []int{1}, []int{1}
fmt.Println(a == b) // panic: runtime error: comparing uncomparable type []int
```

The same panic happens when such a value is used as a map key of interface type. It compiles because the compiler cannot know what the interfaces will hold.

### Storing a value in an interface may allocate

Converting a non-pointer value (a struct, a large number, a string) to an interface often copies it to the heap so the interface can point to it — the `escapes to heap` lines you saw in Unit 2.12 when passing values to `fmt.Println`. Small integers (0–255) and zero-size values are optimised. In hot code this "boxing" can matter (Chapter 10).

### Deep & Tricky

**DT-147. A nil pointer inside an interface is not a nil interface.** *(Guarantee)* The interface `(*MyErr, nil)` compares unequal to `nil`. This is the source of countless "error is not nil but prints `<nil>`" bugs. `fmt.Println` of such an error may print `<nil>` (if the method handles nil) or the method's output — which makes the bug even more confusing.

**DT-148. Interface comparison can panic.** *(Guarantee)* `==` on interface values compiles even when it may hold slices or maps, and panics at run time with `comparing uncomparable type`. Map keys of type `any` have the same risk.

**DT-149. Method calls through interfaces cost an indirect call and block some optimisations.** *(Implementation detail)* The compiler usually cannot inline a call through an interface (it does not know which method will run), unless it can *devirtualize* it — prove the dynamic type — which PGO helps with (Unit 10.9). For almost all code this cost is irrelevant; in the hottest inner loops it can matter.

### Common mistakes

- Returning a typed nil pointer as `error`.
- Comparing `any` values that might hold slices.
- Using `any` where a generic type parameter or a concrete type would keep type safety.

### Mastery check

1. **Explain.** Draw the two-word picture for `var s fmt.Stringer = (*Rect)(nil)` and explain why `s != nil`.
2. **Predict.** (a) the `find()` program (b) `var e error; fmt.Println(e == nil)` (c) `var p *MyErr; var e error = p; fmt.Println(e == nil, p == nil)` (d) `any(1) == any(1.0)`
3. **Write.** A function `validate(s string) error` that *reproduces* the typed-nil bug, a test that catches it, and the fix.

## Unit 3.6 — Type assertions and type switches

**One-line summary:** A type assertion `x.(T)` extracts the concrete value (or a narrower interface) from an interface value. Use the comma-ok form to avoid a panic. A type switch tests several types at once.

### Type assertions

```go
var s fmt.Stringer = Celsius(36.6)

c := s.(Celsius)          // c is a Celsius. If s held something else: PANIC
c, ok := s.(Celsius)      // comma-ok: ok == false instead of a panic
r, ok := s.(Rect)         // ok == false, r is the zero Rect
```

The asserted type can also be an **interface**, which asks "does the dynamic value *also* have these methods?":

```go
if wt, ok := w.(io.WriterTo); ok {
	// fast path: the value knows how to write itself
}
```

This "upgrade" check is how the standard library offers optional fast paths: `io.Copy` checks whether its source implements `io.WriterTo` or its destination implements `io.ReaderFrom`.

### Type switches

```go
func describe(v any) string {
	switch x := v.(type) {
	case nil:
		return "nil"
	case int, int64:
		return fmt.Sprintf("integer %v", x) // in a multi-type case, x has the interface type (any)
	case string:
		return "string of length " + strconv.Itoa(len(x)) // here x is a string
	case error:
		return "error: " + x.Error()
	case fmt.Stringer:
		return "stringer: " + x.String()
	default:
		return fmt.Sprintf("other %T", x)
	}
}
```

- Cases are checked in order; put specific cases before general interfaces.
- In a single-type case, `x` has that type; in a multi-type case or `default`, `x` keeps the original interface type.
- `fallthrough` is not allowed in type switches.

### Interface embedding

Interfaces can include other interfaces:

```go
type ReadWriter interface {
	io.Reader
	io.Writer
}
```

`ReadWriter`'s method set is the union. `io.ReadCloser`, `io.ReadWriteCloser`, and friends are built this way.

### When type switches are a smell

If you find yourself writing `switch v := s.(type) { case Rect: ...; case Circle: ... }` to compute area, the behaviour belongs in a method on the interface. Type switches are right for *external* variety you do not control (decoded JSON values, AST nodes, events from another system) and for optional capabilities; they are wrong as a replacement for polymorphism.

### Deep & Tricky

**DT-150. Errors should be inspected with `errors.As`, not type assertions.** *(Practice + guarantee)* `err.(*fs.PathError)` fails if the error was wrapped (`fmt.Errorf("load: %w", err)`, Unit 3.10). `errors.As` and `errors.AsType` walk the whole chain. Direct assertions on errors are a latent bug as soon as anyone adds context.

**DT-151. A failed assertion to an interface type and to a concrete type panic differently.** *(Runtime behaviour)* `s.(Rect)` panics with `interface conversion: fmt.Stringer is main.Celsius, not main.Rect`; `s.(io.Writer)` panics with `interface conversion: main.Celsius is not io.Writer: missing method Write`. The message tells you the dynamic type, which is often the fastest clue when debugging.

### Mastery check

1. **Explain.** When is a type switch the right tool, and when should the behaviour be a method instead?
2. **Predict.** `describe(nil)`, `describe(42)`, `describe(errors.New("x"))`, `describe(Celsius(1))` (given `Celsius` has `String()`), `describe([]int{})`.
3. **Write.** `func sizeOf(v any) (int, bool)` that returns a length for strings, slices of int, maps of string to int, and anything with a `Len() int` method; `false` otherwise.

## Topic 3C — Composition and type design

## Unit 3.7 — Embedding and composition

**One-line summary:** Embedding a type in a struct (writing the type without a field name) promotes its fields and methods to the outer struct. It is composition with convenient syntax — not inheritance: there is no "super", no overriding of calls made *inside* the embedded type, and the outer type is not a subtype of the inner one.

### Struct embedding

```go
type Logger struct{ prefix string }
func (l *Logger) Logf(format string, args ...any) {
	fmt.Printf(l.prefix+format+"\n", args...)
}

type Server struct {
	*Logger         // embedded: field name is "Logger"
	addr    string
}

s := Server{Logger: &Logger{prefix: "[srv] "}, addr: ":8080"}
s.Logf("listening on %s", s.addr) // promoted: same as s.Logger.Logf(...)
```

- The embedded field's name is the type name (`Logger`); you can always use it explicitly: `s.Logger.Logf`.
- Promoted methods become part of the outer type's method set, so `Server` can satisfy interfaces through them.
- You can embed a value type (`Logger`) or a pointer (`*Logger`). With a pointer, a zero `Server` has a nil `Logger`, and calling `s.Logf` panics.

Since Go 1.27, a struct literal may also set promoted fields directly by their names: with `type Point struct{ X, Y int }` and `type Pixel struct{ Point; Color string }`, `Pixel{X: 1, Y: 2, Color: "red"}` is allowed (before 1.27 you had to write `Pixel{Point: Point{1, 2}, Color: "red"}`).

### Shadowing and ambiguity

If the outer struct declares a field or method with the same name, it wins (the shallower depth wins):

```go
func (s *Server) Logf(format string, args ...any) {
	s.Logger.Logf("(server) "+format, args...) // wrap the embedded method explicitly
}
```

If two embedded types at the same depth have the same name, using that name is a compile error (`ambiguous selector`) — but only if you use it.

### Embedding is not inheritance

```go
type Base struct{}
func (Base) Name() string   { return "base" }
func (b Base) Hello() string { return "hello from " + b.Name() }

type Derived struct{ Base }
func (Derived) Name() string { return "derived" }

fmt.Println(Derived{}.Hello()) // "hello from base" — NOT "derived"
```

`Hello` is a method of `Base`. When it runs, its receiver is the inner `Base` value; it knows nothing about `Derived`. There are no virtual methods. If you want behaviour to be pluggable, pass an interface (or a function) explicitly. The library project uses this: `store.File` embeds `*store.Memory` and *overrides* the write methods, but if any `Memory` method called `m.PutBook` internally, it would call `Memory.PutBook`, not `File.PutBook`.

Also, a `Server` is not a `Logger`: you cannot pass a `Server` where a `*Logger` is expected. Only interfaces give substitutability.

### Embedding interfaces in structs

```go
type countingWriter struct {
	io.Writer // embed the interface: all Writer methods are promoted
	n int
}

func (c *countingWriter) Write(p []byte) (int, error) {
	n, err := c.Writer.Write(p) // delegate
	c.n += n
	return n, err
}
```

This is the decorator pattern in a few lines. Embedding an interface in a test fake is also useful: implement only the methods a test needs; calling any other panics (the embedded interface value is nil) — a clear signal that the test touched something unexpected.

### Composition over inheritance

Instead of deep class hierarchies, Go programs build types from small parts: a `Server` *has* a logger, a store, and a clock. Behaviour is shared through functions and interfaces. Advantages: no fragile base classes, explicit dependencies, and easy substitution in tests.

### Deep & Tricky

**DT-152. Embedding a type with a `sync.Mutex` promotes `Lock` and `Unlock` — publicly.** *(Guarantee + design consequence)* `type Cache struct { sync.Mutex; m map[string]string }` lets *any caller* do `c.Lock()`, making the lock part of your API forever. Use a named, unexported field: `mu sync.Mutex`.

**DT-153. Embedding a type promotes its `String()` or `Error()`, which can hijack printing.** *(Guarantee + surprise)* Embed `time.Time` in a struct, and `fmt.Println(event)` prints only the time, because `Event` now has a promoted `String()` method and fmt uses it (Unit 3.4). The same happens with an embedded error. Use a named field if you do not want the outer type to *be* a Stringer.

**DT-154. Promoted fields in composite literals are a Go 1.27 feature.** *(Version-dependent)* Code using `Pixel{X: 1, Color: "red"}` compiles only when the module's `go` line is 1.27 or later. Libraries supporting older Go versions must keep the nested form.

### Pattern

**P-24. Decorate with embedded interfaces.** *(Go idiom)* Wrap an interface value in a struct that embeds the interface and overrides one or two methods to add behaviour (counting, logging, metrics, limiting). Used throughout `net/http` middleware (Chapter 8) and `io` (Chapter 5). *Trade-off:* methods you do not override pass through unchanged — including optional interfaces like `io.WriterTo`, which the wrapper silently *hides* because the wrapper type does not have them.

### Mastery check

1. **Explain.** Why does `Derived{}.Hello()` print "hello from base"? What would you do to make the name pluggable?
2. **Predict.** A struct embedding both `A` and `B`, each with a method `Close()`: does it compile? Does `x.Close()` compile?
3. **Write.** A `countingWriter` as above; use it with `fmt.Fprintf` to count bytes written to `os.Stdout`.

## Unit 3.8 — Designing types

**One-line summary:** Good Go types are simple to create, hard to misuse, and useful at their zero value where possible. When a zero value cannot work, provide a `NewX` constructor that validates and initialises. For many optional settings, use an options struct.

### Make the zero value useful (DT-48)

```go
var b bytes.Buffer  // ready to use
b.WriteString("hi")

var mu sync.Mutex   // unlocked, ready to use
var wg sync.WaitGroup
```

Design trick: lazily initialise internal state on first use, so the zero value works:

```go
type Counter struct {
	m map[string]int
}

func (c *Counter) Add(key string) {
	if c.m == nil {
		c.m = make(map[string]int) // lazy init: zero Counter is usable
	}
	c.m[key]++
}
```

### Constructors: `NewX`

When there are required values or invariants, provide a constructor and keep fields unexported so callers cannot build invalid values:

```go
type Temperature struct{ kelvin float64 }

func NewTemperature(celsius float64) (Temperature, error) {
	if celsius < -273.15 {
		return Temperature{}, fmt.Errorf("%.2f°C is below absolute zero", celsius)
	}
	return Temperature{kelvin: celsius + 273.15}, nil
}

func (t Temperature) Celsius() float64 { return t.kelvin - 273.15 }
```

Naming conventions: `New` if the package has one main type (`list.New`), `NewX` otherwise (`bytes.NewBuffer`); getters are named after the field without a `Get` prefix (`t.Celsius()`, not `t.GetCelsius()`); setters are `SetX`.

### Options structs

For constructors with many optional parameters (Go has no default arguments), pass a struct whose zero values mean "use the default":

```go
type ServerOptions struct {
	Addr         string        // default ":8080"
	ReadTimeout  time.Duration // default 5s
	MaxBodyBytes int64         // default 1 MiB
}

func NewServer(opts ServerOptions) *Server {
	if opts.Addr == "" {
		opts.Addr = ":8080"
	}
	if opts.ReadTimeout == 0 {
		opts.ReadTimeout = 5 * time.Second
	}
	...
}

srv := NewServer(ServerOptions{Addr: ":9000"})
```

The alternative "functional options" pattern (`NewServer(WithAddr(":9000"), WithTimeout(time.Second))`) is popular in libraries that need to evolve without breaking callers; it is more code and more indirection. Prefer the options struct unless you need what functional options add.

### Immutability by convention

Go has no `const` for struct values. To make a value effectively immutable, keep fields unexported, expose getters, and return *copies* of internal slices and maps (`slices.Clone`), or callers can modify your internal state through the returned slice.

### Deep & Tricky

**DT-155. Returning an internal slice leaks write access.** *(Guarantee + design consequence)* `func (r *Registry) Names() []string { return r.names }` lets callers do `reg.Names()[0] = "hacked"` and change the registry. Return `slices.Clone(r.names)` — or, since Go 1.23, an iterator (`slices.Values(r.names)`, Unit 3.16) that gives read access without copying.

**DT-156. Options structs make "zero" and "unset" the same.** *(Design trade-off)* With `MaxRetries int` in an options struct, you cannot express "zero retries" if zero means "default". Fixes: choose a default that is also the natural zero, use a pointer (`*int`, nil = unset — and Go 1.26's `new(0)` makes setting it easy), or use a negative sentinel with documentation.

### Pattern

**P-25. Constructors that validate; unexported fields that protect invariants.** *(Go design practice)*

- *Problem it solves:* values in invalid states (negative balance, empty ID) created by struct literals anywhere in the code.
- *How:* unexported fields + `NewX(...) (X, error)` that validates; methods maintain the invariant.
- *When not to:* plain data carriers (DTOs decoded from JSON, configuration, test tables) are better as exported fields with a separate `Validate() error` method.
- *Trade-off:* safety and a single place for rules versus more boilerplate and less convenient literals.

### Mastery check

1. **Explain.** Give two standard-library types whose zero values are useful, and one whose zero value is not (and how you get a usable one).
2. **Explain.** Options struct vs functional options: what does each cost and buy?
3. **Write.** A `RateLimiter` type with an options struct (`PerSecond int`, default 10; `Burst int`, default equal to `PerSecond`) and a constructor that rejects negative values. Tests for defaults and validation.

## Topic 3D — Errors and panics

## Unit 3.9 — Errors I

**One-line summary:** In Go, an error is an ordinary value of the interface type `error`. Functions that can fail return it as their last result; callers check it immediately with `if err != nil`. Good errors carry context so that the final message tells a human exactly what failed.

**Mental model — a delivery slip with a remarks column.** Every delivery comes back with a slip. Usually the remarks column is empty (`nil`). When something went wrong, the courier writes a remark: "address not found". Each depot that passes the slip back adds its own short note in front: "Pune hub: van 7: address not found". By the time it reaches you, the slip tells the whole story.

### The `error` interface

```go
type error interface {
	Error() string
}
```

Any type with an `Error() string` method is an error. The simplest ones:

```go
err := errors.New("disk full")
err := fmt.Errorf("user %d not found", id)
```

### Errors are values: check them right away

```go
f, err := os.Open(path)
if err != nil {
	return fmt.Errorf("open config: %w", err) // add context, return (Unit 3.10 explains %w)
}
defer f.Close()
```

This is P-9 (guard clauses) and P-10 (`(result, error)`) together. The happy path runs down the left margin; each failure is handled next to the line that caused it.

Why values instead of exceptions? Every place an operation can fail is *visible* in the code. Nothing can jump out of the middle of a function invisibly. The cost is verbosity, which Go accepts in exchange for clarity.

### Handle each error exactly once

For each error, do **one** of:

1. **Return it** (usually with added context) — the most common choice.
2. **Handle it** — retry, use a default, show it to the user, choose another path — and then do *not* also return it.
3. **Log it and continue** — only at the top level or for genuinely non-fatal problems.

The anti-pattern is *log and return*: the same failure appears in the logs five times, once per layer, with no added information. (Same lesson as "one problem, one report" in the Chapter 1 project.)

### Writing good error messages

- Lower-case, no trailing punctuation: they get embedded in longer messages (`load config: open /etc/app.yaml: permission denied`).
- Add *what you were doing* and the key identifiers: `fmt.Errorf("charge order %s: %w", orderID, err)`.
- Do not repeat what the wrapped error already says (`os.Open` errors already include the path).
- Never put secrets (passwords, tokens) in errors; they end up in logs.

### Errors from deferred calls

`defer f.Close()` ignores `Close`'s error. For files opened for *reading* that is acceptable. For *writing*, a failed `Close` can mean data was not saved (P-13's caveat). Use a named result and a deferred closure (the Unit 1.11 Rule 3 mechanism):

```go
func writeReport(path string, data []byte) (err error) {
	f, err := os.Create(path)
	if err != nil {
		return err
	}
	defer func() {
		if cerr := f.Close(); cerr != nil && err == nil {
			err = cerr // report Close's error only if nothing else failed
		}
	}()
	_, err = f.Write(data)
	return err
}
```

### Deep & Tricky

**DT-157. A function returning `(T, error)` usually leaves T unspecified on error.** *(Convention, not a guarantee)* Callers must not use the value when `err != nil`. A few functions document a useful partial result (`io.Reader.Read` returns `n > 0` bytes *and* an error; `strconv.ParseInt` returns the clamped maximum on range errors). Read the documentation; when in doubt, ignore the value on error.

**DT-158. `errors.New` creates a distinct value each call.** *(Guarantee)* `errors.New("x") == errors.New("x")` is false: each call allocates a new error, and errors compare by identity. That is why sentinel errors (Unit 3.10) are created once in a package-level `var`.

### Mastery check

1. **Explain.** What are the three things you can do with an error, and why is "log and return" an anti-pattern?
2. **Predict.** `errors.New("a") == errors.New("a")`; `err := fmt.Errorf("read %s: %v", "x.txt", io.EOF); err == io.EOF`.
3. **Write.** `loadPort(env map[string]string) (int, error)`: missing key, non-number, and out-of-range port (1–65535) each produce a distinct, well-worded error. Print the three messages.

## Unit 3.10 — Errors II: wrapping and inspecting

**One-line summary:** `fmt.Errorf("...: %w", err)` wraps an error, adding context while keeping the original inside. `errors.Is` checks whether any error in the chain equals a known value; `errors.As` (or `errors.AsType`, Go 1.26) finds an error of a specific type in the chain. Choose sentinel errors or typed errors depending on what callers need to know.

### Wrapping with `%w`

```go
func loadConfig(path string) error {
	data, err := os.ReadFile(path)
	if err != nil {
		return fmt.Errorf("load config: %w", err) // %w: wrap
	}
	...
}
```

The resulting error's message is `load config: open app.yaml: no such file or directory`, and it *contains* the original. With `%v` instead of `%w`, you get the same text but the chain is cut: callers can no longer inspect the cause.

**When to use `%w` vs `%v`:** `%w` makes the wrapped error part of your API (callers may start depending on it). Wrap when callers should be able to react to the cause; use `%v` when the cause is an implementation detail you may change (for example, an error from a database driver you might replace).

### `errors.Is`: "is it this specific error?"

```go
if errors.Is(err, fs.ErrNotExist) {
	// create a default config
}
```

`errors.Is` compares `err` and then each wrapped error (following `Unwrap`) with the target. Compare with `==` only when you know nothing wraps — in practice, always use `errors.Is`.

### `errors.As` and `errors.AsType`: "is there an error of this type?"

```go
var pe *fs.PathError
if errors.As(err, &pe) {
	fmt.Println("failed path:", pe.Path)
}

// Go 1.26+: generic, type-safe, no pointer-to-pointer
if pe, ok := errors.AsType[*fs.PathError](err); ok {
	fmt.Println("failed path:", pe.Path)
}
```

`As` gives you the typed error with all its fields. `AsType` is the recommended form in new code targeting Go 1.26+: it cannot be called with a wrong target type (a common `As` mistake, caught at run time by a panic or by `go vet`'s `errorsas` check).

### Sentinel errors vs typed errors

| | Sentinel error | Typed error |
| --- | --- | --- |
| What | A package-level variable: `var ErrNotFound = errors.New("not found")` | A type implementing `error`: `type ValidationError struct{ Field, Problem string }` |
| Caller checks with | `errors.Is(err, pkg.ErrNotFound)` | `errors.As` / `errors.AsType` |
| Carries data? | No (only identity) | Yes (fields) |
| Examples | `io.EOF`, `sql.ErrNoRows`, `fs.ErrNotExist`, `context.Canceled` | `*fs.PathError`, `*strconv.NumError`, `*net.OpError`, `*json.SyntaxError` |

Rules of thumb: use a sentinel when the caller only needs to know *which* condition happened; a typed error when they need *details*; and plain `fmt.Errorf` text when callers should not branch on the error at all (most errors!). Every exported sentinel or error type is API you must maintain.

### Writing a custom error type that wraps

```go
type QueryError struct {
	Query string
	Err   error
}

func (e *QueryError) Error() string { return "query " + e.Query + ": " + e.Err.Error() }
func (e *QueryError) Unwrap() error { return e.Err } // makes Is/As look inside
```

Use a pointer receiver and return `&QueryError{...}` (and see DT-147: never return a nil `*QueryError` as `error`).

### Several errors at once

```go
err := errors.Join(err1, err2, err3) // nil if all are nil; message is the messages joined by newlines
errors.Is(err, err2)                  // true

err = fmt.Errorf("sync: %w; cleanup: %w", errSync, errCleanup) // several %w (Go 1.20+)
```

Useful when validating many fields, or when a cleanup step fails after the main step already failed. Types can report several wrapped errors with `Unwrap() []error`.

### Deep & Tricky

**DT-159. `errors.As` panics if the target is not a non-nil pointer to an error type or interface.** *(Guarantee)* `errors.As(err, pe)` (missing `&`) or `errors.As(err, &notAnError)` panics at run time. `go vet` reports many of these; `errors.AsType` makes the mistake impossible.

**DT-160. A type can customise matching with `Is(target error) bool` and `As(target any) bool` methods.** *(Guarantee)* For example, a `*TimeoutError` can say it `Is` `context.DeadlineExceeded`. Use sparingly: it makes `errors.Is` results non-obvious to readers.

**DT-161. `go vet` (Go 1.27) flags `%w` with a pointer to a value-receiver error type.** *(Tool behaviour, version-dependent)* If `E` implements `error` with a value receiver and you write `fmt.Errorf("...: %w", p)` where `p` is `*E`, vet now reports it: `errors.As(err, &e)` with a value `e` would not match the `*E` in the chain. Be consistent about whether an error type is used as a value or a pointer.

**DT-162. Wrapping `io.EOF` breaks callers that check `err == io.EOF`.** *(Practice)* Some code — including older standard-library-style loops — compares `err == io.EOF` directly. If your `Read` method returns `fmt.Errorf("x: %w", io.EOF)`, those callers treat the normal end of data as a failure. Return `io.EOF` itself, unwrapped, from readers; the `io.Reader` contract expects exactly that.

### Pattern

**P-26. Wrap with context at every layer boundary; inspect only with `errors.Is`/`As`.** *(Universal Go practice)*

- *Problem it solves:* bare errors like `connection refused` with no clue *which* operation or ID failed; brittle string matching on messages.
- *How:* `return fmt.Errorf("verb object %s: %w", id, err)` when an error crosses a function or package boundary; check causes with `errors.Is` / `errors.As` / `errors.AsType`; never parse `err.Error()` text.
- *When not to:* do not wrap when the message already says everything (repeating "open file: open file.txt: ..."), and do not wrap with `%w` internals you do not want to expose — use `%v`.
- *Trade-off:* a little more typing; massively better debugging.

### Mastery check

1. **Explain.** What changes for callers between `fmt.Errorf("x: %w", err)` and `fmt.Errorf("x: %v", err)`?
2. **Predict.** (a) `errors.Is(fmt.Errorf("a: %w", fs.ErrNotExist), fs.ErrNotExist)` (b) the same with `%v` (c) `errors.Join(nil, nil) == nil` (d) `errors.Is(fmt.Errorf("x: %w, y: %w", fs.ErrNotExist, fs.ErrPermission), fs.ErrPermission)`
3. **Write.** A `ValidationErrors` type that collects several `*FieldError` values, implements `Error()` and `Unwrap() []error`, and lets callers find one field's error with `errors.As`.

## Unit 3.11 — panic and recover

**One-line summary:** `panic` stops normal execution and unwinds the stack, running deferred calls. `recover`, called directly inside a deferred function, stops the unwinding and returns the panic value. Panics are for programmer errors and impossible states, not for expected failures — those are errors.

### What panics are for

Panics signal bugs: an index out of range, a nil map write, a nil pointer dereference, a broken invariant ("this switch has a case for every enum value; reaching default is a bug"). Expected failures — file not found, invalid input, network down, insufficient balance — are **errors** and are returned.

```go
func mustPositive(n int) int {
	if n <= 0 {
		panic(fmt.Sprintf("mustPositive: got %d", n)) // a caller bug
	}
	return n
}
```

**`Must` functions** convert an error into a panic for values that are fixed at program start, where failure means the program itself is wrong:

```go
var validID = regexp.MustCompile(`^[a-z]+\[[0-9]+\]$`) // a bad pattern is a bug, found at startup
var tmpl = template.Must(template.New("x").Parse(text))
```

### What happens during a panic

1. Normal execution of the current function stops.
2. Its deferred calls run (LIFO), then the caller's deferred calls, and so on up the goroutine's stack.
3. If nothing recovers, the program prints `panic: <value>` and the stack trace of the goroutine and exits with code 2 (DT-6). Deferred output appears *before* the panic message (DT-90).

### `recover`

```go
func safeDivide(a, b int) (q int, err error) {
	defer func() {
		if r := recover(); r != nil {
			err = fmt.Errorf("safeDivide: %v", r) // turn the panic into an error
		}
	}()
	return a / b, nil
}

safeDivide(1, 0) // 0, "safeDivide: runtime error: integer divide by zero"
```

Rules:

- `recover` only works when called **directly** by a deferred function. Called anywhere else (or in a helper called *by* the deferred function), it returns nil and does nothing.
- After a successful recover, the function returns normally to its caller, with whatever its named results hold.
- A panic in one goroutine cannot be recovered in another. An unrecovered panic in **any** goroutine kills the **whole program** (Chapter 7).

### Where recovering is right

- At the boundary of an independent unit of work, so one bad request does not kill a server: `net/http` recovers panics in each handler and logs them (Chapter 8).
- In libraries that use panics internally as a control-flow shortcut (deep recursive parsers), recovering at the public API and returning an error — never letting the panic escape.

Recovering is **wrong** as a general error-handling mechanism ("try/catch"). After a panic the program's state may be inconsistent: a mutex may be locked, a half-updated structure left behind.

### Panics vs fatal errors

Some failures are not panics and cannot be recovered: `fatal error: stack overflow` (DT-39), `fatal error: concurrent map writes` (DT-124), `fatal error: all goroutines are asleep - deadlock!` (Chapter 7), out of memory. The runtime stops the program because its own state can no longer be trusted.

### Deep & Tricky

**DT-163. `panic(nil)` became a real panic in Go 1.21.** *(Version-dependent guarantee)* Before 1.21, `panic(nil)` made `recover()` return nil, indistinguishable from "no panic". Since 1.21 (for modules with `go 1.21` or later), `recover()` returns a `*runtime.PanicNilError` with the message `panic called with nil argument`.

**DT-164. Re-panicking keeps the original stack in the message.** *(Runtime behaviour)* If a deferred function recovers and calls `panic(r)` again (for example, after logging), the final output shows both panics, marked `[recovered]`. Use this to add cleanup without hiding the original crash.

**DT-165. `runtime.Goexit` runs defers but is not a panic.** *(Guarantee)* `Goexit` terminates the calling goroutine after running its deferred calls; `recover` returns nil during it. `t.FailNow` and `t.Fatal` in tests use `Goexit`, which is why they must be called from the test's own goroutine (Unit 4.1).

### Mastery check

1. **Explain.** Give two situations where panicking is right and two where returning an error is right.
2. **Predict.** A function defers `fmt.Println("A")`, then defers a closure that calls `recover()` and prints it, then defers `fmt.Println("C")`, then panics with `"boom"`. What is printed, in which order, and does the program exit?
3. **Write.** `func protect(f func()) (err error)` that runs `f` and converts any panic into an error that includes the panic value. Test with a nil map write and with a normal function.

## Topic 3E — Packages and modules

## Unit 3.12 — Packages

**One-line summary:** A package is a directory of `.go` files compiled together. Names starting with an upper-case letter are **exported** (visible to importers); everything else is private to the package. Good packages have short names, a focused purpose, and a small exported API. `internal/` directories restrict who may import, import cycles are forbidden, and initialisation happens in a defined order.

### Packages and directories

- All `.go` files in one directory (excluding `_test.go` files of an external test package) belong to **one package**, named in each file's `package` clause.
- The **import path** is the module path plus the directory: in module `example.com/library`, the directory `internal/catalog` is imported as `example.com/library/internal/catalog`.
- By convention the package name equals the last element of the import path (`catalog`).
- `package main` with `func main()` builds an executable; any other name builds a library.

### Exported vs unexported

```go
package catalog

type Book struct {   // exported type
	ID    string     // exported field
	title string     // unexported field: only code in package catalog can use it
}

func New() *Library  // exported
func countLoans()    // unexported
```

Capitalisation is the *only* visibility mechanism. There is no `public`/`private`/`protected`. Export as little as possible: everything exported is a promise to callers.

### Naming packages

Package names are short, lower-case, single words, no underscores or mixedCaps: `strconv`, `http`, `catalog`. Avoid meaningless names like `util`, `common`, `helpers`, `models` — they become dumping grounds. Since callers write `pkg.Name`, avoid stutter: `catalog.Book`, not `catalog.CatalogBook`; `http.Server`, not `http.HTTPServer`.

### Package documentation

A comment directly before the `package` clause documents the package; a comment directly before an exported name documents it, starting with that name:

```go
// Package catalog holds the library's domain rules: books, members, and loans.
package catalog

// Borrow lends one copy of a book to a member and returns the due date.
func (l *Library) Borrow(bookID, memberID string) (time.Time, error)
```

`go doc catalog.Library.Borrow` prints it; pkg.go.dev renders it. Since Go 1.19, doc comments support headings, lists, and links (`[Library.Borrow]`).

### `internal/`

A package in a directory named `internal` can be imported only by code rooted at the parent of `internal`. `example.com/library/internal/catalog` is importable from anywhere inside `example.com/library/...`, but not from other modules. Use it for everything you do not want to support as a public API — in applications, that is usually almost everything.

### Import cycles are forbidden

If package `a` imports `b`, then `b` (directly or indirectly) may not import `a`: compile error `import cycle not allowed`. This forces a layered design. Common fixes: move the shared type into a third, lower package; or have the lower package define an interface that the higher one implements (consumer-defined interfaces, P-23 — `catalog` defines `Store`, `store` implements it, and `catalog` never imports `store`).

### Initialisation order

1. Imported packages are initialised first (each only once, even if imported many times).
2. Within a package, package-level variables are initialised in **dependency order** (a variable whose initialiser uses another is initialised after it), otherwise in declaration order across files (files presented to the compiler in name order).
3. Then each `func init()` runs, in the order they appear (several per file allowed).
4. Finally, `main.main`.

```go
var a = b + 1   // initialised second
var b = 10      // initialised first
func init() { fmt.Println("init:", a, b) } // init: 11 10
```

Keep `init` minimal: it runs before `main`, cannot return errors, makes tests harder, and hides work. Prefer explicit initialisation called from `main`. Legitimate uses: registering drivers or codecs (`image/png` registers itself; `import _ "image/png"` imports a package only for its `init` side effect).

### Deep & Tricky

**DT-166. A test file can be in the same package or in `pkg_test`.** *(Toolchain behaviour)* `catalog_test.go` with `package catalog` sees unexported names (white-box tests); with `package catalog_test` it sees only the exported API (black-box tests), exactly like a real user — and it can import other packages that import `catalog` without a cycle. The library project's tests use the external form.

**DT-167. Unexported fields are invisible to reflection-based encoders.** *(Guarantee)* `encoding/json` cannot read or write unexported fields: a struct with only lower-case fields encodes as `{}`. Data that must be serialised needs exported fields (plus struct tags for the wire names), or custom marshalling (Unit 5.4).

**DT-168. Dot imports and package-name shadowing hurt readability.** *(Practice)* `import . "fmt"` lets you write `Println` unqualified — and makes every reader guess where names come from. A local variable named like an imported package (`url := ...` with `net/url` imported) shadows the package (Unit 1.9). Rename the import (`neturl "net/url"`) or the variable.

### Mastery check

1. **Explain.** How does `internal/` differ from simply not documenting a package?
2. **Predict.** Package `a` imports `b`, and `b` needs a type from `a`. What error do you get, and name two ways to restructure.
3. **Write.** A module with packages `money` (a `Paise` type, `Format`, `Parse`) and `cart` (uses `money`), plus `cmd/cart` main. Add doc comments and run `go doc ./money`.

## Unit 3.13 — Modules

**One-line summary:** A module is a versioned collection of packages, defined by a `go.mod` file at its root. `go.mod` lists the module path, the Go version, and required dependencies; `go.sum` records cryptographic hashes so builds are reproducible and tamper-evident. Go picks dependency versions with **minimal version selection**.

### `go.mod`

```
module example.com/library

go 1.27

toolchain go1.27.1

require (
	github.com/google/go-cmp v0.7.0
	golang.org/x/text v0.24.0
)

require golang.org/x/sys v0.33.0 // indirect

tool golang.org/x/tools/cmd/stringer
```

| Line | Meaning |
| --- | --- |
| `module` | the module path: the prefix of every package import path in it |
| `go` | the language version the code is written for (enables features such as per-iteration loop variables, DT-72); since 1.21 also the *minimum* Go version required |
| `toolchain` | optional: preferred toolchain to use when this module is the main module |
| `require` | dependencies and their minimum versions; `// indirect` marks ones not imported directly |
| `replace` | use a different module or a local directory instead (for local development or forks) |
| `exclude`, `retract` | skip a bad version / declare that your own published version should not be used |
| `tool` | (Go 1.24+) a tool dependency, run with `go tool stringer` |
| `godebug` | default GODEBUG settings for this program |

### Commands

```
go mod init example.com/library   # create go.mod (Go 1.26+ writes a go line one minor version below the toolchain)
go get github.com/google/go-cmp@v0.7.0   # add or change a dependency
go get -u ./...                   # upgrade dependencies (carefully)
go mod tidy                       # add missing and remove unused requirements; update go.sum
go list -m all                    # every module in the build, with versions
go mod why -m golang.org/x/sys    # why is this module needed?
go mod download / go mod verify   # fetch / check hashes of the module cache
```

Run `go mod tidy` before committing; CI should fail if it would change anything.

### Semantic versioning and major versions

Module versions are `vMAJOR.MINOR.PATCH`. Within a major version, newer versions must be backwards compatible. A breaking change requires a new major version, and **from v2 on, the major version is part of the module path**: `github.com/user/lib/v2`. So `lib` v1 and `lib/v2` are different modules and can even be used together in one build. Versions `v0.x.y` promise nothing.

### Minimal version selection (MVS)

If your module requires `A v1.2.0` and another dependency requires `A v1.4.0`, Go selects **v1.4.0** — the *minimum* version that satisfies everyone, never "the latest available". Builds are therefore reproducible without a lock file: the same `go.mod` files always select the same versions. Upgrades happen only when someone changes a `go.mod` (`go get -u`).

### `go.sum`, the proxy, and the checksum database

`go.sum` contains a hash of every module version used. On download, the `go` command verifies hashes against `go.sum` and, for new modules, against the public **checksum database** (`sum.golang.org`), so a compromised server cannot hand you different code for the same version. Modules are fetched through `GOPROXY` (default `https://proxy.golang.org,direct`). For private code set `GOPRIVATE=example.com/private/*` so it bypasses the public proxy and checksum database.

### Workspaces

`go work init ./app ./lib` creates `go.work`, letting you develop several modules together with local changes and no `replace` lines. Usually not committed.

### Deep & Tricky

**DT-169. The `go` line can trigger a toolchain download.** *(Toolchain behaviour, Go 1.21+)* If `go.mod` says `go 1.27` and your installed Go is 1.26, the `go` command downloads and runs Go 1.27 automatically (controlled by `GOTOOLCHAIN`). In locked-down environments this fails with a download error — the reason the code in this book was checked with whatever toolchain was available, with newer features marked.

**DT-170. Language changes are gated by each module's `go` line.** *(Guarantee)* A dependency with `go 1.21` keeps the old loop-variable semantics even when your module says `go 1.27` (DT-87). The same mechanism gates generic methods (1.27) and promoted-field literals (1.27): using them in a module whose `go` line is older is a compile error telling you which version is needed.

**DT-171. Never edit `go.sum` by hand, and commit it.** *(Practice)* It is the integrity record of your build. Merge conflicts in `go.sum` are resolved by `go mod tidy`.

### Mastery check

1. **Explain.** Why does minimal version selection make builds reproducible without a lock file?
2. **Explain.** Why must a v2 module change its import path?
3. **Write.** Create a module, add `golang.org/x/text`, use it to title-case a string with `cases.Title(language.English)`, then run `go mod tidy`, `go list -m all`, and `go mod why golang.org/x/text`. Explain each output line.

## Topic 3F — Generics and iterators

## Unit 3.14 — Generics I

**One-line summary:** Type parameters let one function or type work for many types while staying statically type-checked: `func Max[T cmp.Ordered](a, b T) T`. A **constraint** (an interface) says which types are allowed and which operations the code may use. The compiler usually infers the type arguments.

**Mental model — a mould with a slot for the material.** A generic function is a mould with a slot: "fill with any material that can be compared". You can cast it in steel or in brass, and the compiler checks each casting separately. Before generics, Go programmers either wrote the mould once per material (copy-paste) or used `any` and checked at run time.

### Generic functions

```go
func Map[T, U any](s []T, f func(T) U) []U {
	out := make([]U, 0, len(s))
	for _, v := range s {
		out = append(out, f(v))
	}
	return out
}

lengths := Map([]string{"go", "rust"}, func(s string) int { return len(s) }) // [2 4]
```

- Type parameters go in square brackets before the regular parameters: `[T, U any]`.
- **Type inference** works out `T = string, U = int` from the arguments. You can also write them: `Map[string, int](...)`. Since Go 1.21 inference also works from the assigned type in more places, and Go 1.27 extended it to every context where a generic function is assigned or converted to a matching function type.

### Constraints

A constraint is an interface that limits the type arguments:

```go
func Max[T cmp.Ordered](a, b T) T { // cmp.Ordered: types supporting < <= > >=
	if a > b {
		return a
	}
	return b
}

func Index[T comparable](s []T, v T) int { // comparable: types supporting == and !=
	for i, x := range s {
		if x == v {
			return i
		}
	}
	return -1
}
```

| Constraint | Allows | Operations you may use |
| --- | --- | --- |
| `any` | every type | assignment, passing, `var zero T` — no `==`, no `<`, no `+` |
| `comparable` | types with `==` | `==`, `!=`, map keys |
| `cmp.Ordered` | integers, floats, strings (and named types based on them) | `< <= > >=` plus `==` |
| your own interface with methods | types with those methods | calling the methods |

The compiler checks the generic body *once*, against the constraint: you may only use operations that every allowed type supports.

### Generic types

```go
type Stack[T any] struct {
	items []T
}

func (s *Stack[T]) Push(v T) { s.items = append(s.items, v) }

func (s *Stack[T]) Pop() (T, bool) {
	var zero T // the zero value of whatever T is
	if len(s.items) == 0 {
		return zero, false
	}
	v := s.items[len(s.items)-1]
	s.items = s.items[:len(s.items)-1]
	return v, true
}

var s Stack[int] // instantiate with int; the zero value is ready to use
s.Push(1)
```

Inside methods, the receiver repeats the type parameter names (`Stack[T]`). `var zero T` is the standard way to get a zero value of a type parameter.

### Deep & Tricky

**DT-172. `comparable` admits interface types, whose `==` can panic.** *(Guarantee, since Go 1.20)* `Index[any]([]any{[]int{1}}, []int{1})` compiles, and the comparison panics at run time (DT-148). "Strictly comparable" types (no interfaces inside) never panic.

**DT-173. Type inference never uses the result type of a call alone.** *(Guarantee)* `func Zero[T any]() T` cannot be called as `var n int = Zero()`. With no arguments to infer from, write `Zero[int]()`. Inference works from argument types (and, since 1.21, from the *function type* a generic function is assigned to).

### Mastery check

1. **Explain.** What does a constraint do *for the caller* and *for the function body*?
2. **Predict.** (a) `Max(3, 7.5)` (b) `Max("go", "Go")` (c) `Index([]float64{math.NaN()}, math.NaN())` (d) a generic `Sum[T any]` that writes `s += v` — does it compile?
3. **Write.** `Filter[T any](s []T, keep func(T) bool) []T`, `Reduce[T, A any](s []T, init A, f func(A, T) A) A`, and use them to compute the total length of all words longer than 3 letters.

## Unit 3.15 — Generics II

**One-line summary:** Constraints can list types with `|` and `~`, generic data structures replace copy-paste, and since Go 1.27 methods may have their own type parameters. Use generics for algorithms and containers that are truly type-independent — not as a reflex.

### Union and `~` constraints

```go
type Number interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64 | ~float32 | ~float64
}

func Sum[T Number](xs []T) T {
	var s T
	for _, x := range xs {
		s += x // allowed: every type in the union supports +
	}
	return s
}

type Paise int64
Sum([]Paise{100, 250}) // 350 — works because of ~int64
```

- `A | B` means "A or B".
- `~int64` means "any type whose **underlying** type is `int64`", including your named types. Without `~`, `Sum([]Paise{...})` would be rejected.
- Interfaces containing type unions can be used **only as constraints**, not as ordinary variable types.
- The standard `cmp.Ordered` is defined exactly this way.

### Generic data structures

```go
type Set[T comparable] struct{ m map[T]struct{} }

func NewSet[T comparable](items ...T) Set[T] {
	s := Set[T]{m: make(map[T]struct{}, len(items))}
	for _, it := range items {
		s.m[it] = struct{}{}
	}
	return s
}

func (s Set[T]) Has(v T) bool { _, ok := s.m[v]; return ok }
func (s Set[T]) Add(v T)      { s.m[v] = struct{}{} }
func (s Set[T]) Len() int     { return len(s.m) }
```

Chapter 6 builds a full generic data-structure library (heap, queue, LRU cache, graph).

### Generic methods (Go 1.27)

Before Go 1.27, a method could use its type's parameters but could not declare new ones, so helper "methods" like `Map` had to be top-level functions. Go 1.27 allows methods to have their own type parameters:

```go
type List[T any] struct{ items []T }

// Go 1.27+: Map declares its own type parameter U.
func (l List[T]) Map[U any](f func(T) U) List[U] {
	out := List[U]{items: make([]U, 0, len(l.items))}
	for _, v := range l.items {
		out.items = append(out.items, f(v))
	}
	return out
}
```

Two restrictions remain: **interface methods cannot have type parameters**, and a generic method **cannot be used to satisfy an interface**. (Satisfying an interface requires a fixed method signature; a generic method is a family of signatures.)

### When NOT to use generics

- When an **interface with methods** expresses the need: `func Render(w io.Writer)` is better than `func Render[W io.Writer](w W)`.
- When the code is written for **one** type today. Wait until you have two or three real uses.
- When different types need different *implementations* — that is what methods and interfaces are for.

Good uses: container types, algorithms over slices and maps (`slices`, `maps`), functional helpers, type-safe wrappers (`atomic.Pointer[T]`, `sync.OnceValue`).

### How generics are compiled (briefly)

The Go compiler uses **GC-shape stenciling with dictionaries**: it generates one copy of the code per *memory shape* (all pointer types share a shape, `int` and `int64` share a shape on 64-bit, and so on) and passes a hidden dictionary describing the actual type. This keeps binaries small; the cost is that some generic code is a little slower than hand-written code for one type, particularly when calling methods on type parameters (Chapter 10 measures it).

### Deep & Tricky

**DT-174. A type parameter's methods are only those in its constraint.** *(Guarantee)* Inside `func F[T fmt.Stringer](v T)`, you may call `v.String()`, but not any other method, even if every type you plan to pass has it. And you cannot type-switch on `T` directly; you can only switch on `any(v)`.

**DT-175. Generic code that calls methods through a type parameter may be slower than an interface call.** *(Implementation detail)* Because all pointer types share one shape, a call like `v.String()` inside a generic function goes through the dictionary — an extra indirection on top of the method lookup. For performance-critical code, measure; sometimes a plain interface parameter is faster *and* simpler.

### Pattern

**P-27. Generics for algorithms and containers; interfaces for behaviour.** *(Go design practice)* If you need "the same code for many types", use type parameters. If you need "different code behind the same call", use an interface. Start concrete, generalise when the second or third use appears. *Trade-off:* generics remove duplication and `any`-casting, but add type-parameter noise and can obscure simple code.

### Mastery check

1. **Explain.** What does `~` add to a constraint? Why does `cmp.Ordered` use it?
2. **Explain.** Why can a generic method not satisfy an interface method?
3. **Write.** `Set[T]` with `Union`, `Intersect`, and `Sorted` (for `T cmp.Ordered` — think about where that constraint must go); tests with `Set[string]` and `Set[int]`.

## Unit 3.16 — Iterators

**One-line summary:** Since Go 1.23, `for range` accepts functions of the shapes `func(yield func(V) bool)` and `func(yield func(K, V) bool)`. The `iter` package names them `iter.Seq[V]` and `iter.Seq2[K, V]`. Iterators let collections expose their elements lazily and uniformly, and the standard library's `slices` and `maps` packages produce and consume them.

### Range over a function

```go
// Countdown yields n, n-1, ..., 1.
func Countdown(n int) iter.Seq[int] {
	return func(yield func(int) bool) {
		for i := n; i > 0; i-- {
			if !yield(i) { // the loop body ran; false means the loop said "stop" (break, return)
				return
			}
		}
	}
}

for v := range Countdown(3) {
	fmt.Println(v) // 3 2 1
}
```

How it works: the compiler turns the loop *body* into the `yield` function. Each `yield(v)` runs the body once with `v`. If the body executes `break` (or `return`, or a labelled jump out), `yield` returns false, and the iterator **must** stop calling it — calling `yield` again after it returned false panics.

### Iterators in the standard library

```go
for i, v := range slices.All(s) {}    // iter.Seq2[int, E]
for v := range slices.Values(s) {}    // iter.Seq[E]
for k := range maps.Keys(m) {}        // iter.Seq[K]
for k, v := range maps.All(m) {}      // iter.Seq2[K, V]

keys := slices.Sorted(maps.Keys(m))   // consume an iterator into a sorted slice
vals := slices.Collect(maps.Values(m))

for line := range strings.Lines(text) {}       // Go 1.24
for part := range strings.SplitSeq(s, ",") {}  // Go 1.24, no intermediate slice
```

### Exposing your collection as an iterator

```go
type Library struct{ books []Book }

func (l *Library) Books() iter.Seq[Book] {
	return func(yield func(Book) bool) {
		for _, b := range l.books {
			if !yield(b) {
				return
			}
		}
	}
}
```

Callers get read-only, lazy access (no copied slice; compare DT-155) and can stop early. The iterator can also do work per element — filter, decode, page through an API — that would otherwise require building a slice first.

### Pull iterators

Sometimes you need to pull values on demand, for example to merge two sorted sequences. `iter.Pull` converts a push iterator into a `next` function:

```go
next, stop := iter.Pull(seq)
defer stop()            // always call stop, or the iterator's resources leak
v, ok := next()
```

### Deep & Tricky

**DT-176. An iterator that ignores `yield`'s false result panics.** *(Guarantee)* If the loop breaks and the iterator calls `yield` again, the runtime panics with `range function continued iteration after function for loop body returned false`. Always `if !yield(v) { return }`.

**DT-177. `defer` inside an iterator runs when the iterator function returns — after the loop ends or breaks.** *(Guarantee)* This makes iterators good at managing resources: open a file, `defer f.Close()`, yield lines. Breaking out of the caller's loop makes `yield` return false, the iterator returns, and the file closes. (But see DT-176 — the iterator must actually return.)

**DT-178. Single-use vs reusable iterators.** *(Convention)* `slices.Values(s)` can be ranged over many times; an iterator reading from a network stream can be used once. Document which kind you return. Iterators returned by `iter.Pull` are single-use by nature.

### Mastery check

1. **Explain.** What does the compiler turn the loop body into? What happens on `break`?
2. **Predict.** `for v := range Countdown(5) { if v == 3 { break }; fmt.Print(v) }`. And what happens if `Countdown` ignored the result of `yield`?
3. **Write.** `Fibonacci() iter.Seq[int]` (infinite), `Take[T any](seq iter.Seq[T], n int) iter.Seq[T]`, and `Filter[T any](seq iter.Seq[T], keep func(T) bool) iter.Seq[T]`. Print the first 10 even Fibonacci numbers.

## Chapter 3 Project — `library`, a lending-library CLI

**Goal:** a command-line program for a small lending library, organised into packages with a clean domain layer, a storage layer behind a consumer-defined interface, typed and sentinel errors, an enum, and tests. This is the first project that looks like a real Go codebase.

### Specification

```
library add-book <id> <copies> <genre> <title...>
library add-member <id> <name...>
library borrow <book-id> <member-id>
library return <book-id> <member-id>
library list
library overdue
```

- Data is kept in a JSON file: `$LIBRARY_DATA`, or `library.json` in the current directory.
- A book has an ID, title, genre (fiction, science, history, children), and number of copies. A loan lasts 14 days.
- Borrowing fails if the book or member does not exist, or if all copies are out. Returning fails if that member has not borrowed that book.
- Exit codes: 0 success, 1 operation failed (message on stderr), 2 usage error (message and usage text on stderr).

### Required structure

```
library/
├── go.mod                       module example.com/library
├── cmd/library/main.go          argument parsing and output only
└── internal/
    ├── catalog/                 domain: types, rules, errors, the Store interface
    │   ├── catalog.go
    │   └── catalog_test.go      tests through the public API (package catalog_test)
    └── store/                   Memory and File implementations of catalog.Store
        └── store.go
```

**Requirements**

1. `catalog` does not import `store` (the dependency points *towards* the domain, P-23).
2. `catalog.Store` has only the methods `catalog` actually calls.
3. Sentinel errors for conditions callers branch on (`ErrNotFound`, `ErrAlreadyExists`, `ErrNoCopies`, `ErrNotBorrowed`); a typed `*ValidationError` with the field name; every other error wrapped with context.
4. `Genre` is an enum with `String()` and `ParseGenre`; the zero value means unknown.
5. Time is injectable so tests are deterministic (`WithClock`).
6. `store.File` saves atomically (write a temporary file, then rename).
7. `go vet ./...` clean; tests cover borrow/return success, no copies, not borrowed, not found, duplicates, validation, and genre parsing.

### Reference solution — `internal/catalog/catalog.go`

```go
// Package catalog holds the library's domain rules: books, members, and
// loans. It knows nothing about files or the command line; storage is
// reached through the small Store interface, which this package defines
// because this package is the one that uses it.
package catalog

import (
	"cmp"
	"errors"
	"fmt"
	"slices"
	"strings"
	"time"
)

// Genre is an enum: a named integer type with a fixed set of constants.
type Genre int

const (
	GenreUnknown Genre = iota // zero value means "not set"
	Fiction
	Science
	History
	Children
)

var genreNames = [...]string{"unknown", "fiction", "science", "history", "children"}

func (g Genre) String() string {
	if g < 0 || int(g) >= len(genreNames) {
		return fmt.Sprintf("Genre(%d)", int(g))
	}
	return genreNames[g]
}

// ParseGenre converts a name such as "science" to a Genre.
func ParseGenre(s string) (Genre, error) {
	for i, name := range genreNames {
		if i > 0 && strings.EqualFold(s, name) {
			return Genre(i), nil
		}
	}
	return GenreUnknown, &ValidationError{Field: "genre", Problem: fmt.Sprintf("unknown genre %q", s)}
}

type Book struct {
	ID     string
	Title  string
	Author string
	Genre  Genre
	Copies int // copies owned by the library
}

type Member struct {
	ID   string
	Name string
}

type Loan struct {
	BookID   string
	MemberID string
	Due      time.Time
}

// Sentinel errors: callers compare with errors.Is.
var (
	ErrNotFound      = errors.New("not found")
	ErrAlreadyExists = errors.New("already exists")
	ErrNoCopies      = errors.New("no copies available")
	ErrNotBorrowed   = errors.New("book is not borrowed by this member")
)

// ValidationError is a typed error: callers extract it with errors.As
// (or errors.AsType in Go 1.26+) to learn which field was wrong.
type ValidationError struct {
	Field   string
	Problem string
}

func (e *ValidationError) Error() string { return e.Field + ": " + e.Problem }

// Store is what the catalog needs from storage. Small on purpose.
type Store interface {
	Book(id string) (Book, error) // returns an error wrapping ErrNotFound if missing
	PutBook(b Book) error
	Books() ([]Book, error)
	Member(id string) (Member, error)
	PutMember(m Member) error
	Loans() ([]Loan, error)
	PutLoans(loans []Loan) error
}

// LoanPeriod is how long a member may keep a book.
const LoanPeriod = 14 * 24 * time.Hour

// Library applies the rules. Create it with New.
type Library struct {
	store Store
	now   func() time.Time // replaceable in tests
}

func New(s Store) *Library {
	return &Library{store: s, now: time.Now}
}

// WithClock returns a copy of l that uses now instead of time.Now.
func (l *Library) WithClock(now func() time.Time) *Library {
	c := *l
	c.now = now
	return &c
}

func (l *Library) AddBook(b Book) error {
	b.ID = strings.TrimSpace(b.ID)
	b.Title = strings.TrimSpace(b.Title)
	switch {
	case b.ID == "":
		return &ValidationError{Field: "id", Problem: "must not be empty"}
	case b.Title == "":
		return &ValidationError{Field: "title", Problem: "must not be empty"}
	case b.Copies < 1:
		return &ValidationError{Field: "copies", Problem: fmt.Sprintf("must be at least 1, got %d", b.Copies)}
	}
	if _, err := l.store.Book(b.ID); err == nil {
		return fmt.Errorf("book %q: %w", b.ID, ErrAlreadyExists)
	} else if !errors.Is(err, ErrNotFound) {
		return err
	}
	return l.store.PutBook(b)
}

func (l *Library) AddMember(m Member) error {
	if strings.TrimSpace(m.ID) == "" {
		return &ValidationError{Field: "id", Problem: "must not be empty"}
	}
	if _, err := l.store.Member(m.ID); err == nil {
		return fmt.Errorf("member %q: %w", m.ID, ErrAlreadyExists)
	} else if !errors.Is(err, ErrNotFound) {
		return err
	}
	return l.store.PutMember(m)
}

// Borrow lends one copy of a book to a member and returns the due date.
func (l *Library) Borrow(bookID, memberID string) (time.Time, error) {
	book, err := l.store.Book(bookID)
	if err != nil {
		return time.Time{}, fmt.Errorf("borrow: %w", err)
	}
	if _, err := l.store.Member(memberID); err != nil {
		return time.Time{}, fmt.Errorf("borrow: %w", err)
	}
	loans, err := l.store.Loans()
	if err != nil {
		return time.Time{}, fmt.Errorf("borrow: %w", err)
	}
	if out := countLoans(loans, bookID); out >= book.Copies {
		return time.Time{}, fmt.Errorf("borrow %q: all %d copies are out: %w", bookID, book.Copies, ErrNoCopies)
	}
	due := l.now().Add(LoanPeriod)
	loans = append(loans, Loan{BookID: bookID, MemberID: memberID, Due: due})
	if err := l.store.PutLoans(loans); err != nil {
		return time.Time{}, fmt.Errorf("borrow: %w", err)
	}
	return due, nil
}

// Return ends a member's loan of a book.
func (l *Library) Return(bookID, memberID string) error {
	loans, err := l.store.Loans()
	if err != nil {
		return fmt.Errorf("return: %w", err)
	}
	i := slices.IndexFunc(loans, func(ln Loan) bool {
		return ln.BookID == bookID && ln.MemberID == memberID
	})
	if i < 0 {
		return fmt.Errorf("return %q by %q: %w", bookID, memberID, ErrNotBorrowed)
	}
	return l.store.PutLoans(slices.Delete(loans, i, i+1))
}

// Availability is one line of the catalogue listing.
type Availability struct {
	Book      Book
	Available int
}

// List returns every book with its number of available copies, by title.
func (l *Library) List() ([]Availability, error) {
	books, err := l.store.Books()
	if err != nil {
		return nil, err
	}
	loans, err := l.store.Loans()
	if err != nil {
		return nil, err
	}
	out := make([]Availability, 0, len(books))
	for _, b := range books {
		out = append(out, Availability{Book: b, Available: b.Copies - countLoans(loans, b.ID)})
	}
	slices.SortFunc(out, func(a, b Availability) int {
		return cmp.Or(
			strings.Compare(a.Book.Title, b.Book.Title),
			strings.Compare(a.Book.ID, b.Book.ID),
		)
	})
	return out, nil
}

// Overdue returns the loans whose due date is before now.
func (l *Library) Overdue() ([]Loan, error) {
	loans, err := l.store.Loans()
	if err != nil {
		return nil, err
	}
	now := l.now()
	return slices.DeleteFunc(loans, func(ln Loan) bool { return !ln.Due.Before(now) }), nil
}

func countLoans(loans []Loan, bookID string) int {
	n := 0
	for _, ln := range loans {
		if ln.BookID == bookID {
			n++
		}
	}
	return n
}
```

`cmp.Or` (Go 1.22) returns its first non-zero argument — a neat way to chain comparison tie-breakers.

### Reference solution — `internal/store/store.go`

```go
// Package store implements catalog.Store in memory and in a JSON file.
package store

import (
	"encoding/json"
	"errors"
	"fmt"
	"io/fs"
	"maps"
	"os"
	"slices"

	"example.com/library/internal/catalog"
)

// Memory keeps everything in maps. Its zero value is NOT ready to use
// (nil maps panic on write), so callers use NewMemory.
type Memory struct {
	books   map[string]catalog.Book
	members map[string]catalog.Member
	loans   []catalog.Loan
}

// Compile-time check that *Memory satisfies catalog.Store.
var _ catalog.Store = (*Memory)(nil)

func NewMemory() *Memory {
	return &Memory{
		books:   make(map[string]catalog.Book),
		members: make(map[string]catalog.Member),
	}
}

func (m *Memory) Book(id string) (catalog.Book, error) {
	b, ok := m.books[id]
	if !ok {
		return catalog.Book{}, fmt.Errorf("book %q: %w", id, catalog.ErrNotFound)
	}
	return b, nil
}

func (m *Memory) PutBook(b catalog.Book) error { m.books[b.ID] = b; return nil }

func (m *Memory) Books() ([]catalog.Book, error) {
	return slices.Collect(maps.Values(m.books)), nil
}

func (m *Memory) Member(id string) (catalog.Member, error) {
	mem, ok := m.members[id]
	if !ok {
		return catalog.Member{}, fmt.Errorf("member %q: %w", id, catalog.ErrNotFound)
	}
	return mem, nil
}

func (m *Memory) PutMember(mem catalog.Member) error { m.members[mem.ID] = mem; return nil }

func (m *Memory) Loans() ([]catalog.Loan, error) { return slices.Clone(m.loans), nil }

func (m *Memory) PutLoans(loans []catalog.Loan) error { m.loans = slices.Clone(loans); return nil }

// File is a Memory that is loaded from and saved to a JSON file.
// Embedding *Memory promotes all of its methods; File overrides the
// three that change data so it can save after each change.
type File struct {
	*Memory
	path string
}

var _ catalog.Store = (*File)(nil)

type fileData struct {
	Books   []catalog.Book   `json:"books"`
	Members []catalog.Member `json:"members"`
	Loans   []catalog.Loan   `json:"loans"`
}

// OpenFile loads path, or starts empty if the file does not exist yet.
func OpenFile(path string) (*File, error) {
	f := &File{Memory: NewMemory(), path: path}
	data, err := os.ReadFile(path)
	if errors.Is(err, fs.ErrNotExist) {
		return f, nil
	}
	if err != nil {
		return nil, err
	}
	var fd fileData
	if err := json.Unmarshal(data, &fd); err != nil {
		return nil, fmt.Errorf("%s: %w", path, err)
	}
	for _, b := range fd.Books {
		f.books[b.ID] = b
	}
	for _, m := range fd.Members {
		f.members[m.ID] = m
	}
	f.loans = fd.Loans
	return f, nil
}

func (f *File) PutBook(b catalog.Book) error     { f.Memory.PutBook(b); return f.save() }
func (f *File) PutMember(m catalog.Member) error { f.Memory.PutMember(m); return f.save() }
func (f *File) PutLoans(l []catalog.Loan) error  { f.Memory.PutLoans(l); return f.save() }

func (f *File) save() error {
	fd := fileData{Loans: f.loans}
	for _, id := range slices.Sorted(maps.Keys(f.books)) {
		fd.Books = append(fd.Books, f.books[id])
	}
	for _, id := range slices.Sorted(maps.Keys(f.members)) {
		fd.Members = append(fd.Members, f.members[id])
	}
	data, err := json.MarshalIndent(fd, "", "  ")
	if err != nil {
		return err
	}
	// Write to a temporary file, then rename: a crash never leaves half a file.
	tmp := f.path + ".tmp"
	if err := os.WriteFile(tmp, data, 0o644); err != nil {
		return err
	}
	return os.Rename(tmp, f.path)
}
```

`Memory` returns *clones* of its loans slice and stores clones of what it is given: callers can never change the store's data through a slice they hold (DT-155). JSON encoding (`json.MarshalIndent`, struct tags) is taught properly in Unit 5.4; the `Genre` field is stored as its number.

### Reference solution — `cmd/library/main.go`

```go
// Command library manages a small lending library stored in a JSON file.
//
//	library add-book <id> <copies> <genre> <title...>
//	library add-member <id> <name...>
//	library borrow <book-id> <member-id>
//	library return <book-id> <member-id>
//	library list
//	library overdue
//
// The data file is $LIBRARY_DATA, or library.json in the current directory.
package main

import (
	"errors"
	"fmt"
	"io"
	"os"
	"strconv"
	"strings"

	"example.com/library/internal/catalog"
	"example.com/library/internal/store"
)

const usage = `usage:
  library add-book <id> <copies> <genre> <title...>
  library add-member <id> <name...>
  library borrow <book-id> <member-id>
  library return <book-id> <member-id>
  library list
  library overdue`

// errUsage marks errors for which the usage text should be printed.
var errUsage = errors.New("bad usage")

func main() {
	err := run(os.Args[1:], os.Stdout)
	if err == nil {
		return
	}
	fmt.Fprintln(os.Stderr, "library:", err)
	if errors.Is(err, errUsage) {
		fmt.Fprintln(os.Stderr, usage)
		os.Exit(2)
	}
	os.Exit(1)
}

func run(args []string, out io.Writer) error {
	if len(args) == 0 {
		return fmt.Errorf("no command: %w", errUsage)
	}
	path := os.Getenv("LIBRARY_DATA")
	if path == "" {
		path = "library.json"
	}
	st, err := store.OpenFile(path)
	if err != nil {
		return err
	}
	lib := catalog.New(st)

	cmd, rest := args[0], args[1:]
	switch cmd {
	case "add-book":
		if len(rest) < 4 {
			return fmt.Errorf("add-book needs 4 or more arguments: %w", errUsage)
		}
		copies, err := strconv.Atoi(rest[1])
		if err != nil {
			return fmt.Errorf("copies %q: %w", rest[1], err)
		}
		genre, err := catalog.ParseGenre(rest[2])
		if err != nil {
			return err
		}
		b := catalog.Book{ID: rest[0], Copies: copies, Genre: genre, Title: strings.Join(rest[3:], " ")}
		if err := lib.AddBook(b); err != nil {
			return err
		}
		fmt.Fprintf(out, "added %s %q\n", b.ID, b.Title)
	case "add-member":
		if len(rest) < 2 {
			return fmt.Errorf("add-member needs 2 or more arguments: %w", errUsage)
		}
		if err := lib.AddMember(catalog.Member{ID: rest[0], Name: strings.Join(rest[1:], " ")}); err != nil {
			return err
		}
		fmt.Fprintf(out, "added member %s\n", rest[0])
	case "borrow":
		if len(rest) != 2 {
			return fmt.Errorf("borrow needs 2 arguments: %w", errUsage)
		}
		due, err := lib.Borrow(rest[0], rest[1])
		if err != nil {
			return err
		}
		fmt.Fprintf(out, "due %s\n", due.Format("2006-01-02"))
	case "return":
		if len(rest) != 2 {
			return fmt.Errorf("return needs 2 arguments: %w", errUsage)
		}
		if err := lib.Return(rest[0], rest[1]); err != nil {
			return err
		}
		fmt.Fprintln(out, "returned")
	case "list":
		items, err := lib.List()
		if err != nil {
			return err
		}
		for _, it := range items {
			fmt.Fprintf(out, "%-6s %-30s %-9s %d/%d available\n",
				it.Book.ID, it.Book.Title, it.Book.Genre, it.Available, it.Book.Copies)
		}
	case "overdue":
		loans, err := lib.Overdue()
		if err != nil {
			return err
		}
		for _, ln := range loans {
			fmt.Fprintf(out, "%s borrowed by %s, due %s\n", ln.BookID, ln.MemberID, ln.Due.Format("2006-01-02"))
		}
	default:
		return fmt.Errorf("unknown command %q: %w", cmd, errUsage)
	}
	return nil
}
```

### Reference solution — `internal/catalog/catalog_test.go`

```go
package catalog_test

import (
	"errors"
	"testing"
	"time"

	"example.com/library/internal/catalog"
	"example.com/library/internal/store"
)

func newLib(t *testing.T) *catalog.Library {
	t.Helper()
	fixed := time.Date(2026, 1, 1, 10, 0, 0, 0, time.UTC)
	lib := catalog.New(store.NewMemory()).WithClock(func() time.Time { return fixed })
	if err := lib.AddBook(catalog.Book{ID: "b1", Title: "Go", Copies: 1}); err != nil {
		t.Fatal(err)
	}
	if err := lib.AddMember(catalog.Member{ID: "m1", Name: "Asha"}); err != nil {
		t.Fatal(err)
	}
	return lib
}

func TestBorrowAndReturn(t *testing.T) {
	lib := newLib(t)
	due, err := lib.Borrow("b1", "m1")
	if err != nil {
		t.Fatal(err)
	}
	if want := time.Date(2026, 1, 15, 10, 0, 0, 0, time.UTC); !due.Equal(want) {
		t.Errorf("due = %v, want %v", due, want)
	}
	if _, err := lib.Borrow("b1", "m1"); !errors.Is(err, catalog.ErrNoCopies) {
		t.Errorf("second borrow: got %v, want ErrNoCopies", err)
	}
	if err := lib.Return("b1", "m1"); err != nil {
		t.Fatal(err)
	}
	if err := lib.Return("b1", "m1"); !errors.Is(err, catalog.ErrNotBorrowed) {
		t.Errorf("second return: got %v, want ErrNotBorrowed", err)
	}
}

func TestErrors(t *testing.T) {
	lib := newLib(t)
	if _, err := lib.Borrow("nope", "m1"); !errors.Is(err, catalog.ErrNotFound) {
		t.Errorf("unknown book: got %v", err)
	}
	if err := lib.AddBook(catalog.Book{ID: "b1", Title: "Again", Copies: 1}); !errors.Is(err, catalog.ErrAlreadyExists) {
		t.Errorf("duplicate: got %v", err)
	}
	err := lib.AddBook(catalog.Book{ID: "b2", Title: "X", Copies: 0})
	var ve *catalog.ValidationError
	if !errors.As(err, &ve) || ve.Field != "copies" {
		t.Errorf("zero copies: got %v", err)
	}
}

func TestGenre(t *testing.T) {
	if g, err := catalog.ParseGenre("Science"); err != nil || g != catalog.Science {
		t.Errorf("ParseGenre = %v, %v", g, err)
	}
	if s := catalog.Genre(42).String(); s != "Genre(42)" {
		t.Errorf("String() = %q", s)
	}
}
```

A real session:

```
$ library add-book b1 2 science The Go Programming Language
added b1 "The Go Programming Language"
$ library add-book b2 1 fiction Malgudi Days
added b2 "Malgudi Days"
$ library add-member m1 Asha Rao
added member m1
$ library borrow b2 m1
due 2026-11-29
$ library borrow b2 m1; echo $?
library: borrow "b2": all 1 copies are out: no copies available
1
$ library list
b2     Malgudi Days                   fiction   0/1 available
b1     The Go Programming Language    science   2/2 available
$ library fly; echo $?
library: unknown command "fly": bad usage
usage:
  ...
2
```

### Review checklist

- Does `catalog` import `store`? It must not. Draw the import graph.
- Is any error compared with `==` or matched by its text? Use `errors.Is` / `errors.As`.
- Can a caller corrupt store data through a returned slice? (Try it in a test.)
- Is the `Store` interface larger than what `catalog` calls?
- Are there typed-nil error returns (DT-147)?
- Is the JSON write atomic? What happens if the program is killed in the middle of `os.WriteFile` of the real file, without the rename trick?
- **Known limitation to discuss:** two `library` processes running at the same moment can each read the file, change it, and write it, losing one update. Fixing this needs file locking or a database — exactly the problem Chapter 9 solves with transactions.

### Extensions

1. Add an `iter.Seq[Loan]` method `LoansOf(memberID)` and use it in a new `member <id>` command.
2. Add a `history` command using a new `Event` enum and an append-only event list.
3. Replace `Store` with a generic `Repo[K comparable, V any]` for books and members. Is the code simpler or not? Decide and justify (P-27).

## Answers to the Chapter 3 prediction questions

**3.1 Q2.** (a) `6` (b) compile error: mismatched types Meters and float64 (c) `1000000000` — `D` has none of `Duration`'s methods (DT-137) (d) 2 nanoseconds.

**3.2 Q2.** Small = 1, Medium = 2, Large = 3; `Size(0)` prints `0`.

**3.3 Q2.** `1` (the method value copied `c` when it was created), then `51` (`inc` holds `&c`).

**3.5 Q2.** (a) `false`, and printing `err` shows `my` (b) `true` (c) `false true` (d) `false` — different dynamic types (`int` and `float64`).

**3.6 Q2.** `nil`, `integer 42`, `error: x`, `stringer: 1.0°C`, `other []int`.

**3.7 Q2.** It compiles; `x.Close()` is a compile error (`ambiguous selector x.Close`) unless the outer type declares its own `Close`.

**3.9 Q2.** `false` (different values, DT-158); `false` — `%v` formats the error into text, so the result is a brand-new error, not `io.EOF` (and even with `%w`, `==` would be false; only `errors.Is` sees through wrapping).

**3.10 Q2.** (a) `true` (b) `false` (c) `true` (d) `true`.

**3.11 Q2.** `C`, then the closure prints `boom`, then `A`. The program does not crash: the panic was recovered, and the function returns normally (with its result values as they were).

**3.12 Q2.** `import cycle not allowed`. Move the shared type to a new lower package both can import, or have `b` define an interface that `a`'s type satisfies.

**3.14 Q2.** (a) `7.5` — both untyped constants, so `T` is inferred as float64 (b) `"go"` — lower-case letters sort after upper-case (c) `-1`, because NaN never equals NaN (d) no: `+` is not allowed on `any`.

**3.16 Q2.** Prints `54`. If `Countdown` ignored `yield`'s false result and called it again, the program would panic: `range function continued iteration after function for loop body returned false` (DT-176).

# Chapter 4 — Testing, Tooling & Debugging

*Goal: make verifying code a habit, not an afterthought.*

Every chapter so far has said "run it, don't guess". This chapter turns that into engineering: automated tests that run in seconds, benchmarks that measure instead of guessing, fuzzers that find inputs you never imagined, static analysis that catches whole classes of bugs before the code runs, and a systematic way to debug when something still goes wrong.

Go's testing tools are built into the `go` command and the standard library — no framework to choose, no runner to configure. That is a big reason Go codebases tend to be well tested: the path of least resistance is the right one.

**What you will be able to do after this chapter**

- Write table-driven tests with subtests, helpers, cleanup, temporary directories, and golden files.
- Read coverage reports critically and use example functions as executable documentation.
- Design code with seams (small interfaces, injected clocks) so it can be tested without real databases or networks.
- Write honest benchmarks, compare them with `benchstat`, and fuzz parsers.
- Run `go vet`, `staticcheck`, `govulncheck`, and debug with Delve and a disciplined method.

## Topic 4A — Writing tests

## Unit 4.1 — Testing basics

**One-line summary:** Tests live in files ending in `_test.go`, in functions named `TestXxx(t *testing.T)`. `go test` compiles them with the package and runs them. Table-driven tests with subtests are the standard Go style.

**Mental model — the quality-control bench.** A factory has a bench next to the line where every batch is checked against a list: input, expected output, actual output. The list (the table) grows every time a defect is found, so the same defect is never shipped twice.

### A first test

```go
// file: mathx/mathx.go
package mathx

func Abs(n int) int {
	if n < 0 {
		return -n
	}
	return n
}
```

```go
// file: mathx/mathx_test.go
package mathx

import "testing"

func TestAbs(t *testing.T) {
	if got := Abs(-3); got != 3 {
		t.Errorf("Abs(-3) = %d, want 3", got)
	}
}
```

```
$ go test ./mathx
ok      example.com/mathx    0.002s
```

- `t.Errorf` records a failure and **continues** the test; `t.Fatalf` records a failure and **stops** this test immediately (use it when continuing makes no sense, for example after a setup error).
- The failure message convention is `Func(input) = got, want expected`. A reader of CI output should understand the failure without opening the code.
- Test files are excluded from normal builds; they can use the same package (white-box) or `package mathx_test` (black-box, DT-166).

### Table-driven tests and subtests

```go
func TestAbs(t *testing.T) {
	tests := []struct {
		name string
		in   int
		want int
	}{
		{"positive", 3, 3},
		{"negative", -3, 3},
		{"zero", 0, 0},
		{"min int", math.MinInt, math.MinInt}, // DT-24: documents a real edge case
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := Abs(tt.in); got != tt.want {
				t.Errorf("Abs(%d) = %d, want %d", tt.in, got, tt.want)
			}
		})
	}
}
```

- Adding a case is one line, so people actually add them.
- `t.Run` creates a **subtest** with its own name, its own pass/fail, and its own `t`. `t.Fatal` inside a subtest stops only that subtest.
- Run one case: `go test -run 'TestAbs/negative' ./mathx` (the `-run` pattern is a regular expression, split on `/` for subtests; spaces in names become underscores).

### Useful `go test` flags

| Flag | Effect |
| --- | --- |
| `-v` | print every test name and its logs, not only failures |
| `-run REGEX` | run only matching tests (and subtests) |
| `-count=1` | ignore the test cache (results of unchanged packages are cached) |
| `-race` | enable the race detector (Chapter 7) — run it in CI |
| `-shuffle=on` | run tests in random order to expose hidden dependencies between them |
| `-failfast` | stop after the first failure |
| `-short` | tests may call `testing.Short()` and skip slow work |
| `-timeout 30s` | fail (with all goroutine stacks) if tests take longer (default 10 minutes) |
| `-cover` | report coverage (Unit 4.3) |
| `-json` | machine-readable output for CI tools |

`t.Log` / `t.Logf` print only for failing tests (or with `-v`). `t.Skip("reason")` skips a test (for example when an external tool is missing).

### What makes a good test

- **Tests behaviour, not implementation.** Assert on outputs and observable effects, not on private fields or the order of internal calls. Such tests survive refactoring.
- **Deterministic.** No dependence on the current time, random numbers without a seed, map iteration order, network, or test order.
- **Fast.** Unit tests in milliseconds, so people run them on every save.
- **Readable failures.** The message says what was called, with what, what came back, and what was expected.
- **One reason to fail.** A test named `TestBorrowWhenNoCopies` should fail only if that behaviour breaks.

### Deep & Tricky

**DT-179. `t.Fatal` must be called from the test's own goroutine.** *(Guarantee)* It calls `runtime.Goexit` (DT-165), which stops only the *current* goroutine. Called from a goroutine the test started, it stops that goroutine but not the test, which may then hang or pass wrongly. `go vet`'s `testinggoroutine` check reports it. From other goroutines, use `t.Error` and return.

**DT-180. Test results are cached.** *(Toolchain behaviour)* `go test ./...` reuses a cached `ok` for packages whose code, dependencies, flags, and *read files and environment variables* have not changed (it tracks which ones the test used). `(cached)` in the output means it did not run. Use `-count=1` when the test depends on something Go cannot see (a database, the network).

**DT-181. Since Go 1.27, `go test` runs the `stdversion` vet check by default.** *(Toolchain behaviour, version-dependent)* It reports uses of standard-library symbols newer than the module's `go` line — for example calling `errors.AsType` in a module that says `go 1.25`. `go test` already ran a small, high-confidence subset of vet checks (atomic, bool, buildtags, directive, errorsas, ifaceassert, nilfunc, printf, stringintconv, tests); this adds one more.

### Mastery check

1. **Explain.** When do you use `t.Errorf` and when `t.Fatalf`? Give an example of each.
2. **Predict.** A table test uses `t.Fatalf` inside `t.Run` and the second of five cases fails. How many cases run? What if `t.Fatalf` is used *without* `t.Run`?
3. **Write.** A table-driven test for `classify(n int) string` from Unit 1.6, with names, boundary cases, and one deliberately wrong expected value; read the failure message, then fix the table.

## Unit 4.2 — Test helpers and fixtures

**One-line summary:** `t.Helper()` makes failures point at the caller; `t.Cleanup` registers teardown; `t.TempDir` and `t.Setenv` give isolated files and environment; `t.Parallel` runs tests concurrently; the `testdata` directory and **golden files** hold expected outputs.

### Helpers

```go
func mustAddBook(t *testing.T, lib *catalog.Library, b catalog.Book) {
	t.Helper() // without this, failures report THIS line, not the test's line
	if err := lib.AddBook(b); err != nil {
		t.Fatalf("AddBook(%q): %v", b.ID, err)
	}
}
```

Helpers take `t` as the first parameter and call `t.Helper()` first. They keep tests short without hiding *what* is being tested — setup goes in helpers, the assertion that matters stays visible in the test.

### Cleanup, temporary directories, environment

```go
func TestExport(t *testing.T) {
	dir := t.TempDir()                 // unique directory, deleted after the test
	t.Setenv("APP_MODE", "test")       // restored after the test (not allowed in parallel tests)
	srv := startFakeServer()
	t.Cleanup(func() { srv.Close() })  // runs after the test and its subtests, LIFO like defer
	ctx := t.Context()                 // Go 1.24: canceled just before Cleanup functions run
	...
}
```

`t.Cleanup` beats `defer` in helpers: a helper's `defer` would run when the *helper* returns, but `t.Cleanup` runs when the *test* ends.

### Parallel tests

```go
func TestSlowThings(t *testing.T) {
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			t.Parallel() // pauses here; resumes in parallel after the loop finishes
			...
		})
	}
}
```

Parallel subtests share nothing unless you share it. They cannot use `t.Setenv` or `t.Chdir` (process-wide state). Since Go 1.22's per-iteration loop variables (DT-72), capturing `tt` in a parallel subtest is safe; in older code you will see `tt := tt` before `t.Run`.

### `testdata` and golden files

The `go` tool ignores directories named `testdata`, so they are the place for input files and expected outputs. Tests run with the **package directory** as the working directory, so `os.ReadFile("testdata/input.txt")` works.

A **golden file** stores the expected output of a complex operation (a report, generated code, a rendered page). The test compares actual output with the file; a flag rewrites the file when the change is intended:

```go
var update = flag.Bool("update", false, "rewrite golden files")

func TestListGolden(t *testing.T) {
	got := renderReport(...)
	golden := filepath.Join("testdata", "report.golden")
	if *update {
		os.WriteFile(golden, []byte(got), 0o644)
	}
	want, err := os.ReadFile(golden)
	if err != nil {
		t.Fatal(err)
	}
	if got != string(want) {
		t.Errorf("output differs from %s:\n got: %q\nwant: %q", golden, got, want)
	}
}
```

`go test -run TestListGolden -update` regenerates it; the diff in code review shows exactly what changed. Golden files are only as good as the review of their diffs.

### Comparing complex values

`==` works for comparable values; `slices.Equal` and `maps.Equal` for slices and maps; `reflect.DeepEqual` for arbitrary nesting (but with poor failure messages). Many teams use `github.com/google/go-cmp/cmp`, whose `cmp.Diff(want, got)` prints a readable diff.

### Deep & Tricky

**DT-182. `TestMain` controls the whole package's test run.** *(Guarantee)* `func TestMain(m *testing.M) { setup(); code := m.Run(); teardown(); os.Exit(code) }` runs once per package. Use it for expensive shared fixtures (a database container, Unit 9.8). Since Go 1.15 you may simply return instead of calling `os.Exit`; the testing framework exits with the right code.

**DT-183. Parallel subtests run after the parent function returns.** *(Guarantee)* A parent test's deferred calls run *before* its parallel subtests execute, so a `defer db.Close()` in the parent closes the database the subtests still need. Use `t.Cleanup` in the parent, which waits for subtests.

### Mastery check

1. **Explain.** Why is `t.Cleanup` better than `defer` inside a helper function?
2. **Explain.** What is a golden file, and what discipline makes golden tests trustworthy?
3. **Write.** A helper `writeFile(t, dir, name, content string) string` that creates a file in `t.TempDir()` and returns its path; use it in a table test of a function that counts lines in a file.

## Unit 4.3 — Coverage and examples

**One-line summary:** Coverage shows which statements tests executed — a map of what is *untested*, not a proof of correctness. Example functions are tests that double as documentation: their output is checked, and they appear in `go doc` and on pkg.go.dev.

### Coverage

```
$ go test -cover ./...
ok   example.com/library/internal/catalog   coverage: 84.2% of statements
$ go test -coverprofile=cover.out ./...
$ go tool cover -func=cover.out      # per function
$ go tool cover -html=cover.out      # open a colour-coded view in the browser
```

How to use it well:

- Look at the **red lines** in the HTML view. Are they error paths? Boundary conditions? Those are where bugs hide (the Chapter 1 lesson: the path you never ran is the one that breaks).
- Do not chase 100%. Some code (a `main` that only wires things, a "should never happen" panic) is not worth testing. 70–90% is typical for well-tested Go packages; the *right* number depends on risk.
- 100% coverage does not mean correct: every line can run while the assertions check nothing. The Chapter 4 project shows a bug that a high-coverage suite misses.
- `-coverpkg=./...` counts coverage of *all* packages exercised, not just the package under test (useful for integration tests). `go build -cover` builds a binary that writes coverage when run, for coverage from end-to-end tests.

### Example functions

```go
func ExampleAbs() {
	fmt.Println(Abs(-5))
	fmt.Println(Abs(7))
	// Output:
	// 5
	// 7
}
```

- Named `Example`, `ExampleFunc`, `ExampleType`, `ExampleType_Method`, optionally with a `_suffix` (`ExampleAbs_negative`).
- `go test` runs it and compares stdout with the `// Output:` comment. `// Unordered output:` ignores line order (handy for map output). Without an output comment, the example is compiled but not run.
- `go doc` and pkg.go.dev show it next to the function. Users copy examples; making them compiled and checked guarantees they work.

### Deep & Tricky

**DT-184. Coverage counts statements, not branches or conditions.** *(Tool behaviour)* `if a && b {` is one statement; a test where `a` is false covers the line but never evaluates `b`. Coverage of 100% says nothing about whether each condition was tried both ways. Think in terms of cases, not lines.

### Mastery check

1. **Explain.** Give an example of a function with 100% coverage and a serious bug the tests did not catch.
2. **Write.** Two example functions for a package of your own, one with `// Output:` and one for a map-based function with `// Unordered output:`. Break one deliberately and read the failure.

## Unit 4.4 — Designing for testability

**One-line summary:** Code is easy to test when its dependencies (time, randomness, storage, network, the file system, stdin/stdout) come in through parameters or small interfaces. Then tests can substitute **fakes**, **stubs**, or **recorders** — no mocking framework needed.

### The seams

A **seam** is a place where a test can swap in a different implementation without changing the code under test.

| Dependency | Seam | Test substitute |
| --- | --- | --- |
| Current time | `now func() time.Time` field or parameter | fixed or advancing fake clock |
| Randomness | `*rand.Rand` parameter | `rand.New(rand.NewPCG(1, 2))` (Unit 1.13) |
| Storage | small interface (`catalog.Store`) | in-memory implementation |
| HTTP services | interface, or an `*http.Client` / base URL | `httptest.Server` (Unit 8.7) |
| Output | `io.Writer` parameter | `*strings.Builder` / `bytes.Buffer` |
| Input | `io.Reader` parameter | `strings.NewReader("...")` |
| Environment, args | `run(args []string, env func(string) string)` | literal values |

The `run(args, stdin, stdout)` pattern (P-14) is the same idea at program level: `main` passes the real `os.Stdin`/`os.Stdout`; tests pass buffers.

### Kinds of test doubles

- **Fake:** a working, simplified implementation (an in-memory store). The most useful kind: tests read naturally and check *behaviour*.
- **Stub:** returns canned answers ("always return `ErrNotFound`") — good for forcing error paths.
- **Recorder (spy):** remembers the calls it received so the test can check them (the `RecordingNotifier` in Unit 3.4).
- **Mock:** a stub plus expectations about exactly which calls happen in which order. Mocks generated by tools couple tests to implementation details; prefer fakes and assert on outcomes. Use strict mocks only when the *interaction itself* is the behaviour (for example "charge the card exactly once").

### A stub that forces an error path

Embed a real fake and override one method (the outer method shadows the promoted one, Unit 3.7):

```go
// failingLoans is a stub: a real in-memory store whose Loans method fails.
type failingLoans struct {
	*store.Memory
}

func (failingLoans) Loans() ([]catalog.Loan, error) {
	return nil, errors.New("disk on fire")
}

func TestBorrowStorageError(t *testing.T) {
	mem := store.NewMemory()
	if err := mem.PutBook(catalog.Book{ID: "b1", Title: "T", Copies: 1}); err != nil {
		t.Fatal(err)
	}
	if err := mem.PutMember(catalog.Member{ID: "m1"}); err != nil {
		t.Fatal(err)
	}
	lib := catalog.New(failingLoans{mem})
	_, err := lib.Borrow("b1", "m1")
	if err == nil || !strings.Contains(err.Error(), "disk on fire") {
		t.Fatalf("Borrow error = %v, want the storage error", err)
	}
}
```

This is the decorator pattern of P-24 used for testing. Note what the test checks: that the storage error *reaches the caller* (with context added), not which internal calls happened.

### Don't over-abstract

Not every dependency needs an interface. Pure functions need no seams at all. A package that only formats strings needs none. Add a seam when a test needs it, at the boundary to something slow, nondeterministic, or external.

### Pattern

**P-28. Inject time, randomness, and I/O; keep the core pure.** *(Testability practice)*

- *Problem it solves:* tests that are slow, flaky (depend on the clock or the network), or impossible (how do you test "after 14 days"?).
- *How:* pass `now func() time.Time`, `*rand.Rand`, `io.Reader`/`io.Writer`, and consumer-defined interfaces (P-23). Production wiring happens in `main`. Keep business rules in functions that take values and return values.
- *When not to:* trivial code, or when the real dependency is fast and deterministic (an in-process `map` does not need a seam).
- *Trade-off:* a few extra parameters or fields in exchange for fast, deterministic tests.

### Mastery check

1. **Explain.** Fake vs stub vs mock: which would you use to test "a member cannot borrow when the store fails", and why?
2. **Write.** A function `greeting(now func() time.Time) string` that says "Good morning/afternoon/evening"; test all three plus the boundaries (11:59:59, 12:00:00) without waiting for the clock.

## Topic 4B — Measuring and fuzzing

## Unit 4.5 — Benchmarks

**One-line summary:** Benchmarks are functions `BenchmarkXxx(b *testing.B)` that run code many times and report time and allocations per operation. Measure before optimising, compare runs statistically with `benchstat`, and beware of benchmarks that measure the wrong thing.

### Writing a benchmark

```go
func BenchmarkJoin(b *testing.B) {
	parts := strings.Fields(strings.Repeat("go ", 1000)) // setup: not timed (Go 1.24 b.Loop)
	b.ReportAllocs()
	for b.Loop() { // Go 1.24+: runs the body as many times as needed
		_ = strings.Join(parts, ",")
	}
}
```

```
$ go test -run XXX -bench Join -benchmem
BenchmarkJoin-8   	  200000	      5321 ns/op	    3072 B/op	       1 allocs/op
```

- `-run XXX` (any pattern that matches no test) skips the normal tests. `-bench .` runs all benchmarks.
- Columns: iterations, time per operation, bytes allocated per operation, allocations per operation. The `-8` suffix is GOMAXPROCS.
- `b.Loop()` (Go 1.24) is the modern form. It times only the loop, keeps the body's results alive so the compiler cannot delete the work, and runs setup code before the loop exactly once. Older code uses `for i := 0; i < b.N; i++ { ... }` with `b.ResetTimer()` after setup; both are still valid.

### Comparing with `benchstat`

A single benchmark run is noisy (CPU frequency, other processes, thermal state). Run each version several times and compare statistically:

```
$ go test -run XXX -bench List -count 10 > old.txt
  ... change the code ...
$ go test -run XXX -bench List -count 10 > new.txt
$ go run golang.org/x/perf/cmd/benchstat@latest old.txt new.txt
name    old time/op    new time/op    delta
List-8    982µs ± 2%     41µs ± 3%   -95.83%  (p=0.000 n=10+10)
```

`benchstat` reports the median, the variation, and whether the difference is statistically significant (`p` < 0.05). "± 15%" variation means your machine is too noisy to detect small changes; quiet it down (close other programs, plug in a laptop) before trusting results.

### Misleading benchmarks

- **Dead-code elimination:** with the old `b.N` loop, `for i := 0; i < b.N; i++ { square(5) }` may be compiled away when the result is unused — a "0.3 ns/op" miracle. `b.Loop` prevents this; with `b.N`, assign results to a package-level sink variable.
- **Unrealistic inputs:** benchmarking with 10 elements says little about 10 million (caches, Unit 0.5). Benchmark with realistic sizes, often several (`b.Run("n=1000", ...)` sub-benchmarks).
- **Measuring setup:** setup inside the timed loop dominates the result.
- **Warm caches:** repeating the same input keeps everything in L1 cache; real workloads may not.
- **Comparing across machines:** only compare runs from the same machine and conditions.

### Allocations

`testing.AllocsPerRun(100, func() {...})` returns the average allocation count; tests can assert "this hot function does not allocate" to stop regressions.

### Deep & Tricky

**DT-185. A benchmark is a hypothesis test, not a speed reading.** *(Practice)* The question is "is B faster than A on this workload, beyond the noise?" — which needs multiple runs and a statistical comparison. A single run that shows a 3% improvement is usually noise.

**DT-186. The library project's `List` benchmark reveals an O(books × loans) loop.** *(Seen in this book's own benchmark)* `countLoans` scans every loan for every book: about 1 ms for 1,000 books with 334 loans. Counting loans per book once into a map makes it O(books + loans). Chapter 6 gives the vocabulary (Big-O) and Chapter 10 the tooling (profiles) to find such loops systematically.

### Mastery check

1. **Explain.** Why can a benchmark report 0.3 ns/op for a function that computes something, and how does `b.Loop` prevent it?
2. **Write.** Benchmarks for three ways of building a 10,000-element comma-separated string (`+=`, `strings.Builder`, `strings.Join`), with `-benchmem`; run them 10 times each and compare with `benchstat`. Explain the allocation counts.

## Unit 4.6 — Fuzzing

**One-line summary:** A fuzz test (`FuzzXxx(f *testing.F)`) feeds automatically mutated inputs to your code, looking for crashes and violated properties. It is especially powerful for parsers and anything that handles untrusted input. Failing inputs are saved and become permanent regression tests.

### Writing a fuzz test

```go
func FuzzParseGenre(f *testing.F) {
	for _, s := range []string{"fiction", "SCIENCE", "", "unknown", "hist ory"} {
		f.Add(s) // the seed corpus
	}
	f.Fuzz(func(t *testing.T, s string) {
		g, err := catalog.ParseGenre(s)
		if err != nil {
			if g != catalog.GenreUnknown {
				t.Errorf("error with non-zero genre %v", g)
			}
			return
		}
		// Property: the name of a parsed genre parses back to the same genre.
		g2, err := catalog.ParseGenre(g.String())
		if err != nil || g2 != g {
			t.Errorf("round trip %q -> %v -> %v, %v", s, g, g2, err)
		}
	})
}
```

- `go test` (without `-fuzz`) runs the fuzz function on the seed corpus only — a normal, fast test.
- `go test -fuzz FuzzParseGenre -fuzztime 30s` mutates inputs (coverage-guided: inputs that reach new code are kept and mutated further).
- A failing input is written to `testdata/fuzz/FuzzParseGenre/<hash>`. Commit it: from then on, plain `go test` re-runs it.
- Arguments may be strings, `[]byte`, integers, floats, bools, and runes.

### What to assert: properties

You cannot write the expected output for random input, so fuzz tests check **properties** that must always hold:

- **No panic** (the minimum — any panic fails the test).
- **Round trip:** `Parse(Format(x)) == x`, `Decode(Encode(b)) == b`.
- **Equivalence:** a fast implementation gives the same answer as a slow, obviously-correct one.
- **Invariants:** output is valid UTF-8; length never exceeds input length; a sorted result is sorted.

### Deep & Tricky

**DT-187. Fuzzing uses all CPU cores and grows the cache.** *(Toolchain behaviour)* `-fuzz` runs workers on every core (`-parallel` limits it) and stores generated corpus entries in the build cache (`$GOCACHE/fuzz`), which can grow large; `go clean -fuzzcache` empties it. In CI, fuzz for a fixed `-fuzztime` on a schedule, not on every commit.

### Mastery check

1. **Explain.** Why does a fuzz test check properties rather than exact outputs? Give three properties for a URL parser.
2. **Write.** A fuzz test for `splitWords` from the Chapter 2 project with the properties: never panics, every word is non-empty, contains no spaces, and is lower case (`strings.ToLower(w) == w`). Run it for a minute. Did it find anything? (Think about DT-116 before you answer.)

## Topic 4C — Tools and debugging

## Unit 4.7 — Static analysis

**One-line summary:** Static analysers read your code without running it and report likely bugs. Run `gofmt`, `go vet`, and `staticcheck` on every change, `govulncheck` regularly, and consider `golangci-lint` to run many linters together.

### The toolbox

| Tool | What it catches | How |
| --- | --- | --- |
| `gofmt` / `goimports` | formatting; `goimports` also adds and removes imports | `gofmt -l .` lists unformatted files (CI fails if non-empty) |
| `go vet` | suspicious constructs: printf mismatches, copied locks (DT-136), unreachable code, bad struct tags, `errors.As` misuse, self-assignment, loop-closure capture in old code, recursive `String` (DT-139) | `go vet ./...` |
| `staticcheck` | hundreds of checks: deprecated APIs, impossible conditions (`uint >= 0`, DT-22), unused code, simplifications, common bugs | `go run honnef.co/go/tools/cmd/staticcheck@latest ./...` |
| `govulncheck` | known vulnerabilities in dependencies **that your code actually calls** | `go run golang.org/x/vuln/cmd/govulncheck@latest ./...` |
| `golangci-lint` | runs many linters (including the above, `errcheck`, `gosec`, `revive`, `exhaustive`, shadow checks) with one config | `golangci-lint run` |
| `go fix` | (Go 1.26 rewrite) applies "modernizers": replaces old idioms with new ones (`interface{}` → `any`, loops → `slices.Contains`, `errors.As` → `errors.AsType`, …) | `go fix ./...` |

`govulncheck` is notable: it does not just list vulnerable *modules* in `go.mod`; it analyses the call graph and reports only vulnerabilities in functions your program can reach, so the results are actionable.

### Making it automatic

- Editor: gopls runs vet-like checks and staticcheck on save.
- Pre-commit or CI: `gofmt -l`, `go vet ./...`, `staticcheck ./...`, `go test -race ./...`, `govulncheck ./...` (Unit 11.7 builds the full pipeline).
- Treat warnings as errors in CI; a linter whose output everyone ignores is worse than none.

### Deep & Tricky

**DT-188. Linters can be silenced line by line, and that should be rare and explained.** *(Tool behaviour)* `//lint:ignore SA1019 reason` (staticcheck) or `//nolint:errcheck // reason` (golangci-lint) suppress a finding. Require a reason in the comment so reviewers can judge it.

### Mastery check

1. **Explain.** Why is `govulncheck`'s call-graph analysis more useful than a list of vulnerable module versions?
2. **Write.** Take any program from Chapters 1–3, deliberately add: a printf with the wrong number of arguments, a copied `sync.Mutex`, an unsigned `>= 0` loop, and an ignored error. Run `go vet` and `staticcheck`; which tool finds which?

## Unit 4.8 — Debugging

**One-line summary:** Debugging is a method, not luck: reproduce, read the evidence (the panic message and stack trace), form a hypothesis, test it with the smallest experiment, fix, and add a test so it never returns. Delve is the Go debugger for stepping through code and inspecting goroutines.

### Reading a panic

```
panic: runtime error: index out of range [5] with length 3

goroutine 1 [running]:
main.lastScore(...)
        /home/asha/proj/main.go:12
main.report({0xc000012345, 0x3, 0x3})
        /home/asha/proj/main.go:20 +0x1d
main.main()
        /home/asha/proj/main.go:27 +0x45
exit status 2
```

Read it top down: the **message** says what (index 5, length 3); the **first frame** in your code says where (`main.go:12`); the frames below say how you got there. `(...)` marks an inlined frame (DT-40). The hex values in `report({0xc000012345, 0x3, 0x3})` are the raw words of the arguments: a slice is shown as its three-word header (pointer, len 3, cap 3, Unit 2.2). `GOTRACEBACK=all` prints every goroutine; `GOTRACEBACK=crash` also produces a core dump on supported systems. Since Go 1.27, goroutine headers also show `runtime/pprof` labels, if the program sets them (Unit 10.2).

### A systematic method

1. **Reproduce** reliably — ideally as a failing test. A bug you cannot reproduce you cannot verify as fixed.
2. **Read the evidence:** the full error message, the stack, the logs around the failure, recent changes (`git log`, `git bisect` to find the commit that introduced it).
3. **Hypothesise:** "the slice is empty when the input file has a trailing blank line".
4. **Test the hypothesis** with the smallest experiment: a print, an assertion, a debugger breakpoint, a narrowed test case. Change one thing at a time.
5. **Fix the cause, not the symptom.** A `recover` that hides a panic is not a fix.
6. **Add the regression test** (the failing test from step 1 now passes) and look for the same bug elsewhere.

### Delve

```
$ go install github.com/go-delve/delve/cmd/dlv@latest
$ dlv debug ./cmd/library -- borrow b1 m1     # build with debug info and start
(dlv) break catalog.(*Library).Borrow         # or: break catalog.go:97
(dlv) continue
(dlv) next          # next line (step over calls)
(dlv) step          # step into a call
(dlv) print book    # inspect a variable; also: locals, args
(dlv) stack         # current goroutine's stack
(dlv) goroutines    # all goroutines (essential for Chapter 7)
(dlv) quit
$ dlv test ./internal/catalog -- -test.run TestBorrowAndReturn   # debug a test
```

Delve understands Go: it prints slices, maps, strings, and interfaces properly and knows about goroutines. Editors (VS Code, GoLand) use it underneath. For optimised builds, variables may be "optimized away"; `dlv debug` builds with `-gcflags='all=-N -l'` (no optimisation, no inlining) for you.

### Print debugging is fine — with care

A quick `fmt.Fprintf(os.Stderr, "DEBUG n=%d s=%q\n", n, s)` is often the fastest experiment. Use `%q` and `%#v` to see exact values (hidden spaces, nil vs empty). Remove them before committing; for permanent diagnostics use structured logging (Unit 5.8).

### Deep & Tricky

**DT-189. Stack traces show argument words, not argument values.** *(Runtime behaviour)* The hex numbers after a function name are the raw machine words of its arguments (strings as pointer + length, slices as pointer + len + cap, interfaces as two words), truncated after a few words and possibly inaccurate for register-passed arguments (marked `?` in newer versions). Use them for hints (a length of 0, a nil pointer `0x0`), not as reliable values.

### Mastery check

1. **Explain.** Walk through the six steps of the debugging method with a real bug from your own work in this book.
2. **Write.** Introduce an off-by-one into `wordfreq`'s top-N cut, reproduce it with a test, then use Delve to break in `run`, inspect `entries`, and confirm your hypothesis before fixing it.

## Chapter 4 Project — A real test suite, and a bug hunt

**Goal:** turn the Chapter 3 `library` into a properly tested codebase, measure it, and then prove your tests work by hunting planted bugs.

### Part 1 — Build the test suite

Add, to the Chapter 3 project:

1. **Table-driven validation tests** for `AddBook`: empty ID, blank ID, empty title, zero copies, negative copies — each asserting the *field* of the `*ValidationError`.
2. **`List` ordering test** with two books of the same title (ordering must fall back to ID) and availability counts after a loan.
3. **`Overdue` boundary tests** with a controllable fake clock: before the due date, *exactly at* the due date (not overdue), one second after, and after two loans.
4. **Store tests** (`internal/store/store_test.go`, white-box): a `File` round trip through `t.TempDir()`, a corrupt-file error, no `.tmp` file left behind, and a test proving callers cannot modify store data through returned or passed slices.
5. **Golden test** for the `list` command output via `run`, with an `-update` flag.
6. **Usage-error tests** for `run`.
7. **A fuzz test** for `ParseGenre` (round-trip property).
8. **A benchmark** for `List` with 1,000 books.

Target: `go test -cover ./...` above 80% for `catalog` and `store`, and the whole suite under one second (excluding fuzzing).

### Reference test suite

`internal/catalog/list_test.go`:

```go
package catalog_test

import (
	"errors"
	"testing"
	"time"

	"example.com/library/internal/catalog"
	"example.com/library/internal/store"
)

// fakeClock is a controllable clock for tests.
type fakeClock struct{ t time.Time }

func (c *fakeClock) Now() time.Time          { return c.t }
func (c *fakeClock) Advance(d time.Duration) { c.t = c.t.Add(d) }

func TestList(t *testing.T) {
	lib := catalog.New(store.NewMemory())
	books := []catalog.Book{
		{ID: "b2", Title: "Zebra", Copies: 2},
		{ID: "b1", Title: "Apple", Copies: 1},
		{ID: "b3", Title: "Apple", Copies: 3},
	}
	for _, b := range books {
		if err := lib.AddBook(b); err != nil {
			t.Fatal(err)
		}
	}
	if err := lib.AddMember(catalog.Member{ID: "m1"}); err != nil {
		t.Fatal(err)
	}
	if _, err := lib.Borrow("b3", "m1"); err != nil {
		t.Fatal(err)
	}

	got, err := lib.List()
	if err != nil {
		t.Fatal(err)
	}
	want := []struct {
		id    string
		avail int
	}{{"b1", 1}, {"b3", 2}, {"b2", 2}} // by title, then ID
	if len(got) != len(want) {
		t.Fatalf("len = %d, want %d", len(got), len(want))
	}
	for i, w := range want {
		if got[i].Book.ID != w.id || got[i].Available != w.avail {
			t.Errorf("row %d = %s/%d, want %s/%d", i, got[i].Book.ID, got[i].Available, w.id, w.avail)
		}
	}
}

func TestOverdue(t *testing.T) {
	clock := &fakeClock{t: time.Date(2026, 3, 1, 9, 0, 0, 0, time.UTC)}
	lib := catalog.New(store.NewMemory()).WithClock(clock.Now)
	for _, err := range []error{
		lib.AddBook(catalog.Book{ID: "b1", Title: "T", Copies: 2}),
		lib.AddMember(catalog.Member{ID: "m1"}),
		lib.AddMember(catalog.Member{ID: "m2"}),
	} {
		if err != nil {
			t.Fatal(err)
		}
	}
	if _, err := lib.Borrow("b1", "m1"); err != nil {
		t.Fatal(err)
	}
	clock.Advance(24 * time.Hour)
	if _, err := lib.Borrow("b1", "m2"); err != nil {
		t.Fatal(err)
	}

	tests := []struct {
		name  string
		after time.Duration // from the first loan
		want  int
	}{
		{"before any due date", 13 * 24 * time.Hour, 0},
		{"exactly at first due date", 14 * 24 * time.Hour, 0}, // due, not yet overdue
		{"just after first due date", 14*24*time.Hour + time.Second, 1},
		{"after both", 16 * 24 * time.Hour, 2},
	}
	start := time.Date(2026, 3, 1, 9, 0, 0, 0, time.UTC)
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			clock.t = start.Add(tt.after)
			got, err := lib.Overdue()
			if err != nil {
				t.Fatal(err)
			}
			if len(got) != tt.want {
				t.Errorf("Overdue() returned %d loans, want %d", len(got), tt.want)
			}
		})
	}
}

func TestAddBookValidation(t *testing.T) {
	tests := []struct {
		name      string
		book      catalog.Book
		wantField string
	}{
		{"empty id", catalog.Book{Title: "T", Copies: 1}, "id"},
		{"blank id", catalog.Book{ID: "  ", Title: "T", Copies: 1}, "id"},
		{"empty title", catalog.Book{ID: "x", Copies: 1}, "title"},
		{"zero copies", catalog.Book{ID: "x", Title: "T"}, "copies"},
		{"negative copies", catalog.Book{ID: "x", Title: "T", Copies: -2}, "copies"},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			lib := catalog.New(store.NewMemory())
			err := lib.AddBook(tt.book)
			var ve *catalog.ValidationError
			if !errors.As(err, &ve) {
				t.Fatalf("got %v, want a *ValidationError", err)
			}
			if ve.Field != tt.wantField {
				t.Errorf("field = %q, want %q", ve.Field, tt.wantField)
			}
		})
	}
}

func FuzzParseGenre(f *testing.F) {
	for _, s := range []string{"fiction", "SCIENCE", "", "unknown", "hist ory"} {
		f.Add(s)
	}
	f.Fuzz(func(t *testing.T, s string) {
		g, err := catalog.ParseGenre(s)
		if err != nil {
			if g != catalog.GenreUnknown {
				t.Errorf("error with non-zero genre %v", g)
			}
			return
		}
		// Round trip: the name of a parsed genre parses to the same genre.
		g2, err := catalog.ParseGenre(g.String())
		if err != nil || g2 != g {
			t.Errorf("round trip %q -> %v -> %v, %v", s, g, g2, err)
		}
	})
}

func BenchmarkList(b *testing.B) {
	lib := catalog.New(store.NewMemory())
	_ = lib.AddMember(catalog.Member{ID: "m"})
	for i := range 1000 {
		id := "b" + string(rune('a'+i%26)) + time.Duration(i).String()
		_ = lib.AddBook(catalog.Book{ID: id, Title: id, Copies: 2})
		if i%3 == 0 {
			_, _ = lib.Borrow(id, "m")
		}
	}
	b.ReportAllocs()
	for b.Loop() {
		if _, err := lib.List(); err != nil {
			b.Fatal(err)
		}
	}
}
```

`internal/store/store_test.go`:

```go
package store

import (
	"os"
	"path/filepath"
	"testing"
	"time"

	"example.com/library/internal/catalog"
)

func TestFileRoundTrip(t *testing.T) {
	path := filepath.Join(t.TempDir(), "lib.json") // removed automatically after the test
	f, err := OpenFile(path)
	if err != nil {
		t.Fatal(err)
	}
	due := time.Date(2026, 5, 1, 0, 0, 0, 0, time.UTC)
	must(t, f.PutBook(catalog.Book{ID: "b1", Title: "Go", Genre: catalog.Science, Copies: 2}))
	must(t, f.PutMember(catalog.Member{ID: "m1", Name: "Asha"}))
	must(t, f.PutLoans([]catalog.Loan{{BookID: "b1", MemberID: "m1", Due: due}}))

	g, err := OpenFile(path) // a fresh load from disk
	if err != nil {
		t.Fatal(err)
	}
	b, err := g.Book("b1")
	if err != nil || b.Title != "Go" || b.Genre != catalog.Science || b.Copies != 2 {
		t.Errorf("book after reload = %+v, %v", b, err)
	}
	loans, _ := g.Loans()
	if len(loans) != 1 || !loans[0].Due.Equal(due) {
		t.Errorf("loans after reload = %+v", loans)
	}
	if _, err := os.Stat(path + ".tmp"); !os.IsNotExist(err) {
		t.Errorf("temporary file left behind: %v", err)
	}
}

func TestOpenFileCorrupt(t *testing.T) {
	path := filepath.Join(t.TempDir(), "bad.json")
	if err := os.WriteFile(path, []byte("{not json"), 0o644); err != nil {
		t.Fatal(err)
	}
	if _, err := OpenFile(path); err == nil {
		t.Fatal("expected an error for a corrupt file")
	}
}

func TestLoansAreCopies(t *testing.T) {
	m := NewMemory()
	in := []catalog.Loan{{BookID: "b1"}}
	must(t, m.PutLoans(in))
	in[0].BookID = "changed-by-caller"
	out, _ := m.Loans()
	out[0].MemberID = "changed-by-reader"
	again, _ := m.Loans()
	if again[0].BookID != "b1" || again[0].MemberID != "" {
		t.Errorf("store data was modified through a slice: %+v", again[0])
	}
}

func must(t *testing.T, err error) {
	t.Helper() // failures are reported at the caller's line
	if err != nil {
		t.Fatal(err)
	}
}
```

`cmd/library/main_test.go`:

```go
package main

import (
	"flag"
	"os"
	"path/filepath"
	"strings"
	"testing"
)

var update = flag.Bool("update", false, "rewrite golden files")

func TestListGolden(t *testing.T) {
	t.Setenv("LIBRARY_DATA", filepath.Join(t.TempDir(), "lib.json"))
	steps := [][]string{
		{"add-book", "b1", "2", "science", "The Go Programming Language"},
		{"add-book", "b2", "1", "fiction", "Malgudi Days"},
		{"add-member", "m1", "Asha"},
		{"borrow", "b2", "m1"},
	}
	for _, args := range steps {
		if err := run(args, new(strings.Builder)); err != nil {
			t.Fatalf("%v: %v", args, err)
		}
	}
	var out strings.Builder
	if err := run([]string{"list"}, &out); err != nil {
		t.Fatal(err)
	}

	golden := filepath.Join("testdata", "list.golden")
	if *update {
		if err := os.WriteFile(golden, []byte(out.String()), 0o644); err != nil {
			t.Fatal(err)
		}
	}
	want, err := os.ReadFile(golden)
	if err != nil {
		t.Fatal(err)
	}
	if out.String() != string(want) {
		t.Errorf("list output differs from %s:\n got: %q\nwant: %q", golden, out.String(), want)
	}
}

func TestUsageErrors(t *testing.T) {
	t.Setenv("LIBRARY_DATA", filepath.Join(t.TempDir(), "lib.json"))
	for _, args := range [][]string{nil, {"fly"}, {"borrow", "only-one"}} {
		err := run(args, new(strings.Builder))
		if err == nil || !strings.Contains(err.Error(), "bad usage") {
			t.Errorf("run(%q) = %v, want a usage error", args, err)
		}
	}
}
```

Real results with this suite:

```
$ go test -cover ./...
ok   example.com/library/cmd/library        coverage: 52.5% of statements
ok   example.com/library/internal/catalog   coverage: 84.2% of statements
ok   example.com/library/internal/store     coverage: 82.0% of statements
$ go test -run XXX -bench List ./internal/catalog
BenchmarkList-2   	     200	    982272 ns/op	  238784 B/op	      13 allocs/op
```

### Part 2 — The bug hunt

Copy the project and apply each of the following **planted bugs** one at a time. Before running the tests, *predict* which test (if any) will fail. Then run `go test ./...` and compare. Record prediction and result in a table.

| # | File | Change |
| --- | --- | --- |
| 1 | `catalog.go`, `Borrow` | `out >= book.Copies` → `out > book.Copies` |
| 2 | `catalog.go`, `Overdue` | `return !ln.Due.Before(now)` → `return ln.Due.After(now)` |
| 3 | `catalog.go`, `List` | tie-break `strings.Compare(a.Book.ID, b.Book.ID)` → `strings.Compare(b.Book.ID, a.Book.ID)` |
| 4 | `store.go`, `Memory.Loans` | `return slices.Clone(m.loans), nil` → `return m.loans, nil` |
| 5 | `catalog.go`, `AddBook` | delete the line `b.ID = strings.TrimSpace(b.ID)` |
| 6 | `store.go`, `save` | `return os.Rename(tmp, f.path)` → `return nil` |
| 7 | `catalog.go`, `Return` | `slices.Delete(loans, i, i+1)` → `slices.Delete(loans, 0, 1)` |

### Part 3 — Optimise with evidence

Rewrite `List` so it counts loans per book once (a `map[string]int`), keep the behaviour identical (the tests prove it), and compare old and new with `-count 10` and `benchstat`.

### Answer key for the bug hunt (verified by running each bug)

| # | Tests that fail | Why |
| --- | --- | --- |
| 1 | `TestBorrowAndReturn` | The second borrow of a 1-copy book succeeds instead of `ErrNoCopies`. |
| 2 | `TestOverdue/exactly_at_first_due_date` | A loan *due* right now is wrongly reported as overdue: a boundary bug, caught only because the table has the boundary row. |
| 3 | `TestList` | Same-title books come out in reverse ID order. |
| 4 | `TestLoansAreCopies` **and** `TestOverdue` (two subtests) | Surprise: `Overdue` calls `slices.DeleteFunc` on the slice returned by `Loans`. Without the clone, that *modifies the store's own loans* (Unit 2.4 aliasing), so later subtests see loans that have vanished. One missing `Clone` broke an unrelated feature. |
| 5 | `TestAddBookValidation/blank_id` | `"  "` is no longer rejected. |
| 6 | `TestFileRoundTrip` and `TestListGolden` | Data is written to `.tmp` and never renamed, so a reload sees nothing. |
| 7 | **None.** | Every test returns the *only* loan, so removing index 0 happens to be correct. The suite has high coverage and still misses it. |

Bug 7 is the most important lesson of the chapter: **coverage showed the line ran, but no test checked the case that distinguishes right from wrong.** The fix is a new test where the loan to return is not the first one:

```go
func TestReturnRemovesTheRightLoan(t *testing.T) {
	lib := catalog.New(store.NewMemory())
	for _, err := range []error{
		lib.AddBook(catalog.Book{ID: "b1", Title: "A", Copies: 1}),
		lib.AddBook(catalog.Book{ID: "b2", Title: "B", Copies: 1}),
		lib.AddMember(catalog.Member{ID: "m1"}),
	} {
		if err != nil {
			t.Fatal(err)
		}
	}
	for _, id := range []string{"b1", "b2"} {
		if _, err := lib.Borrow(id, "m1"); err != nil {
			t.Fatal(err)
		}
	}
	if err := lib.Return("b2", "m1"); err != nil { // return the SECOND loan
		t.Fatal(err)
	}
	items, err := lib.List()
	if err != nil {
		t.Fatal(err)
	}
	for _, it := range items {
		want := map[string]int{"b1": 0, "b2": 1}[it.Book.ID]
		if it.Available != want {
			t.Errorf("%s available = %d, want %d", it.Book.ID, it.Available, want)
		}
	}
}
```

With bug 7 applied it fails with `b1 available = 1, want 0` and `b2 available = 0, want 1`. This technique — deliberately breaking code to check that tests notice — is called **mutation testing**, and tools exist to automate it.

### Review checklist

- Does every test name say what behaviour it checks?
- Would each assertion's failure message be understandable in CI without opening the code?
- Is any test dependent on time, map order, or another test (run with `-shuffle=on -count=3`)?
- For each `if` in the domain code, is there a test for both outcomes — and for the boundary?
- Is the benchmark comparison backed by `benchstat` output, not one run?

## Answers to the Chapter 4 prediction questions

**4.1 Q2.** With `t.Run`, all five cases run: `t.Fatalf` stops only the failing subtest. Without `t.Run`, `t.Fatalf` stops the whole test function at the second case, so cases 3–5 never run (and you do not learn whether they pass).

**4.6 Q2.** Most likely nothing. When this exercise was run for 40 seconds (2.2 million inputs), the fuzzer found no violation: lower-casing each rune with `unicode.ToLower` and lower-casing the whole word with `strings.ToLower` agree, and the word rules never admit spaces. Two lessons. First, a passing fuzz run is *evidence*, not proof: random byte mutations rarely produce specific multi-byte characters, so add seeds from every script and edge case you care about (Turkish `İ`, German `ß`, combining marks, emoji). Second, the act of writing the properties forced you to decide what a "word" must satisfy — often the most valuable part of fuzzing.

# Chapter 5 — Standard Library & I/O

*Goal: use Go's strong standard library instead of reaching for dependencies.*

Go ships with a standard library that covers files, streams, compression, JSON, CSV, binary encodings, time, command-line flags, signals, subprocesses, structured logging, templates, regular expressions, big numbers, and cryptographic randomness — all maintained by the Go team and covered by the compatibility promise. Experienced Go engineers reach for the standard library first and add dependencies only for a clear reason.

The single most important idea in this chapter is **streaming composition**: `io.Reader` and `io.Writer` are tiny interfaces, and almost everything that moves bytes implements them. Snap a file, a gzip decompressor, a hash, and a network connection together like pipes, and data flows through all of them in constant memory.

**What you will be able to do after this chapter**

- Compose readers and writers, and process files of any size in constant memory.
- Work safely with files, paths, permissions, and embedded assets.
- Encode and decode JSON (including its traps), CSV, and binary formats.
- Handle time correctly: durations, time zones, monotonic clocks, timers.
- Build professional command-line tools: flags, signals, subprocesses, exit codes, structured logs.
- Read the standard library's own source to learn how experts write Go.

## Topic 5A — I/O foundations

## Unit 5.1 — Readers and writers

**One-line summary:** `io.Reader` has one method, `Read(p []byte) (n int, err error)`; `io.Writer` has one method, `Write(p []byte) (n int, err error)`. Because they are so small, files, network connections, buffers, compressors, hashes, and HTTP bodies all implement them, and functions written against them work with all of these.

**Mental model — garden hoses and connectors.** A reader is a tap that delivers water in cupfuls when asked. A writer is a drain that accepts cupfuls. Adapters (gzip, buffering, hashing, limiting) are hose connectors: water goes in one side and comes out the other, changed. You can connect any number of them, and at no point does the whole tank of water need to be in one place.

### The contracts

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

**Read** fills up to `len(p)` bytes into `p` and returns how many (`n`). Key rules:

- It may return **fewer** bytes than requested, even if more are coming (a network packet, a pipe). Never assume a full buffer.
- At the end of the data it returns `io.EOF` — possibly *together with* the last bytes (`n > 0, err == io.EOF`). **Always process the `n` bytes before looking at `err`.**
- `io.EOF` is a sentinel that means "normal end", not a failure (and should not be wrapped, DT-162).

**Write** writes all of `p` or returns an error explaining why it wrote less (`n < len(p)` must come with `err != nil`).

The correct hand-written read loop:

```go
buf := make([]byte, 32*1024)
for {
	n, err := r.Read(buf)
	if n > 0 {
		process(buf[:n]) // first use the data
	}
	if err == io.EOF {
		break // normal end
	}
	if err != nil {
		return err // real failure
	}
}
```

You will rarely write this loop, because helpers do it for you.

### The helpers

| Need | Use |
| --- | --- |
| Copy everything from a reader to a writer | `io.Copy(dst, src)` (constant memory, 32 KB buffer) |
| Read everything into memory | `io.ReadAll(r)` (only for data you *know* is small) |
| Read exactly n bytes | `io.ReadFull(r, buf)` |
| Read at most n bytes | `io.LimitReader(r, n)` — wrap untrusted input! |
| Concatenate readers | `io.MultiReader(r1, r2, ...)` |
| Write to several writers at once | `io.MultiWriter(w1, w2, ...)` |
| Copy what is read into a writer | `io.TeeReader(r, w)` |
| Connect a writer to a reader in memory | `io.Pipe()` (with goroutines, Chapter 7) |
| Discard output | `io.Discard` |
| Read from a string / byte slice | `strings.NewReader(s)`, `bytes.NewReader(b)` |
| Collect writes in memory | `bytes.Buffer`, `strings.Builder` |

### Composition in action

Compute the SHA-256 of a gzip file's *decompressed* contents while also counting bytes, without holding the file in memory:

```go
f, err := os.Open("data.csv.gz")
if err != nil {
	return err
}
defer f.Close()

gz, err := gzip.NewReader(f) // file -> decompressor
if err != nil {
	return err
}
defer gz.Close()

h := sha256.New()               // a hash is an io.Writer
n, err := io.Copy(h, gz)        // decompressed bytes flow into the hash
if err != nil {
	return err
}
fmt.Printf("%d bytes, sha256 %x\n", n, h.Sum(nil))
```

Change `os.Open` to an HTTP response body, or `sha256.New()` to an `os.Stdout`, and nothing else changes. That is the power of small interfaces (Unit 3.4).

### Optional fast paths

`io.Copy` checks for `io.WriterTo` on the source and `io.ReaderFrom` on the destination (Unit 3.6). For example, copying from an `*os.File` to a network connection can use the operating system's `sendfile`/`splice` calls, moving data without ever copying it into your process. Wrapping a reader in your own type hides these interfaces (P-24's trade-off).

### Deep & Tricky

**DT-190. `Read` can return data and `io.EOF` at the same time.** *(Contract)* Code that checks `err` first and breaks on EOF loses the last chunk. Process `n` bytes, then inspect `err`. This bug survives testing with `strings.NewReader` (which returns EOF on a separate call) and appears with other readers.

**DT-191. `io.ReadAll` on untrusted input is a denial-of-service bug.** *(Practice)* A client that sends 20 GB makes your server allocate 20 GB. Wrap with `io.LimitReader(r, maxBytes)` (or `http.MaxBytesReader` for HTTP bodies, Unit 8.4) and treat "limit reached" as an error.

### Pattern

**P-29. Write functions against `io.Reader`/`io.Writer`, not file names.** *(Core Go practice)*

- *Problem it solves:* functions that open files themselves cannot be reused for network data, compressed input, or in-memory tests.
- *How:* `func Parse(r io.Reader) (...)`, `func Render(w io.Writer, ...) error`. A thin wrapper opens the file and calls them.
- *When not to:* when you need random access (`io.ReaderAt`, `io.Seeker`) or file metadata — then take the narrowest interface that provides it.
- *Trade-off:* almost none; this is one of the few near-universal rules.

### Mastery check

1. **Explain.** Why must you process `n` bytes before checking `err` in a read loop?
2. **Predict.** `r := io.LimitReader(strings.NewReader("hello world"), 5); b, _ := io.ReadAll(r); fmt.Println(string(b))`; and `io.Copy(io.Discard, strings.NewReader("abc"))` returns what?
3. **Write.** An `upperWriter` that wraps an `io.Writer` and upper-cases ASCII letters on the way through; use it with `io.Copy(upperWriter{os.Stdout}, os.Stdin)`. Then a `countingReader` that counts bytes read.

## Unit 5.2 — Files

**One-line summary:** `os.Open`, `os.Create`, and `os.OpenFile` give you an `*os.File` (a reader and writer). Use `bufio` for efficient line-by-line reading and buffered writing, handle `Close` errors on writes, set permissions deliberately, and write files atomically when a crash must not leave half a file.

### Opening files

```go
f, err := os.Open(path)        // read only
f, err := os.Create(path)      // write; create or TRUNCATE; mode 0o666 before umask
f, err := os.OpenFile(path, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0o644) // full control
```

The flags are bit flags (P-7): `O_RDONLY`, `O_WRONLY`, `O_RDWR`, combined with `O_APPEND`, `O_CREATE`, `O_EXCL` (fail if it exists), `O_TRUNC`. The permission argument is used only when the file is created, and the process's **umask** (usually `022`) removes bits: `0o666` becomes `0o644` (Unit 0.2's octal permissions).

For small files, the whole-file helpers are simplest: `os.ReadFile(path)` and `os.WriteFile(path, data, 0o644)`.

### Errors you will meet

```go
_, err := os.Open("missing.txt")
fmt.Println(err)                         // open missing.txt: no such file or directory
errors.Is(err, fs.ErrNotExist)           // true
var pe *fs.PathError
errors.As(err, &pe)                      // pe.Op "open", pe.Path "missing.txt", pe.Err syscall.ENOENT
```

Check conditions with `errors.Is(err, fs.ErrNotExist)`, `fs.ErrPermission`, `fs.ErrExist` — portable across operating systems.

### Reading large files in constant memory

```go
f, err := os.Open(path)
if err != nil {
	return err
}
defer f.Close()

sc := bufio.NewScanner(f)
sc.Buffer(make([]byte, 0, 64*1024), 16*1024*1024) // allow long lines (DT-92)
for sc.Scan() {
	handle(sc.Text())
}
return sc.Err()
```

Memory use is the size of the buffer, whether the file is 1 KB or 100 GB. `sc.Split(bufio.ScanWords)` splits on words instead; `bufio.ScanRunes` on runes; you can write a custom `SplitFunc` for other formats. For records that are not line-based, `bufio.NewReader(f)` offers `ReadString('\n')`, `ReadByte`, `ReadRune`, and `Peek`.

`sc.Bytes()` returns a slice of the scanner's internal buffer, overwritten by the next `Scan` — copy it if you keep it (the sub-slice idea of Unit 2.4). `sc.Text()` allocates a new string each time, which is safe.

### Writing efficiently and safely

Every `Write` on an `*os.File` is a system call. Many small writes are slow; buffer them:

```go
func writeReport(path string, rows []Row) (err error) {
	f, err := os.Create(path)
	if err != nil {
		return err
	}
	defer func() {
		if cerr := f.Close(); cerr != nil && err == nil {
			err = cerr
		}
	}()
	w := bufio.NewWriter(f)
	for _, r := range rows {
		fmt.Fprintf(w, "%s,%d\n", r.Name, r.Count)
	}
	return w.Flush() // without Flush, the last buffered bytes are LOST
}
```

Forgetting `Flush` is one of the most common file bugs in Go: the program "works" but the last few kilobytes of the output are missing.

### Atomic writes

If the program crashes (or the machine loses power) during a write, the file is left half-written. Write to a temporary file in the **same directory**, sync it, then rename over the target — `rename` within one file system is atomic on POSIX systems, so readers see either the old file or the new one, never a mix:

```go
tmp, err := os.CreateTemp(filepath.Dir(path), ".tmp-*")
... write to tmp ...
tmp.Sync()                    // force data to the disk (durability)
tmp.Close()
os.Rename(tmp.Name(), path)   // atomic replace
```

The Chapter 3 project did a simpler version of this.

### Other operations

`os.Stat` (size, mode, modification time; `errors.Is(err, fs.ErrNotExist)` to test existence), `os.Remove`, `os.RemoveAll`, `os.Rename`, `os.MkdirAll(dir, 0o755)`, `os.ReadDir` (sorted directory entries), `os.Chmod`, `os.TempDir`, `os.CreateTemp`, `os.MkdirTemp`. Since Go 1.24, `os.Root` (`os.OpenRoot(dir)`) opens files *only inside* a directory, rejecting `../` escapes and symbolic links that lead outside — essential when file names come from users (Unit 11.2).

### Deep & Tricky

**DT-192. `defer f.Close()` on a file you wrote can lose errors — and data.** *(OS behaviour)* Many file systems (especially network file systems) report write failures only at `Close` or `Sync`. Ignoring `Close`'s error means a full disk can silently produce a truncated file. Check it, as in `writeReport` above (P-13's caveat).

**DT-193. Checking whether a file exists before opening it is a race.** *(Concurrency fact)* Between `os.Stat` saying "does not exist" and `os.Create`, another process may create it. Use `os.OpenFile` with `O_CREATE|O_EXCL` to "create only if absent" atomically, and simply handle the error from `Open` instead of checking first.

### Mastery check

1. **Explain.** Why does writing through `bufio.Writer` need an explicit `Flush`, and what happens if you forget?
2. **Explain.** Why must the temporary file for an atomic write be in the same directory as the target?
3. **Write.** `tail(path string, n int) ([]string, error)` that returns the last n lines of a file of any size using constant memory (hint: a ring buffer of n strings, Unit 6.3 preview).

## Unit 5.3 — Paths and file systems

**One-line summary:** Use `path/filepath` for operating-system file paths, `path` for slash-separated paths (URLs, embedded files), `io/fs` for read-only file-system abstractions, and `embed` to compile files into your binary.

### `path/filepath`

```go
filepath.Join("data", "2026", "log.txt") // data/2026/log.txt (data\2026\log.txt on Windows)
filepath.Base("/a/b/c.txt")              // c.txt
filepath.Dir("/a/b/c.txt")               // /a/b
filepath.Ext("archive.tar.gz")           // .gz
filepath.Clean("a//b/../c/./d")          // a/c/d
filepath.Abs("x.txt")                    // absolute path based on the working directory
filepath.Rel("/a/b", "/a/b/c/d")         // c/d
filepath.Match("*.go", "main.go")        // true, nil
filepath.Glob("testdata/*.json")
```

Never build paths with `+ "/" +`: it breaks on Windows and creates `//` or missing separators. And never trust a path component from a user: `filepath.Join("uploads", "../../etc/passwd")` cleans to `../etc/passwd`, outside your directory (use `os.Root` or `filepath.IsLocal`, Go 1.20+).

### Walking a directory tree

```go
err := filepath.WalkDir("src", func(path string, d fs.DirEntry, err error) error {
	if err != nil {
		return err // or log and return nil to keep going
	}
	if d.IsDir() && d.Name() == ".git" {
		return filepath.SkipDir
	}
	if !d.IsDir() && filepath.Ext(path) == ".go" {
		fmt.Println(path)
	}
	return nil
})
```

`WalkDir` (Go 1.16) is faster than the older `Walk` because it does not `Stat` every file.

### `io/fs`: file systems as interfaces

`fs.FS` is a read-only file-system interface with one method, `Open(name string) (fs.File, error)`, using slash-separated paths. Implementations include a real directory (`os.DirFS("static")`), embedded files (`embed.FS`), zip archives (`zip.Reader`), and test fakes (`testing/fstest.MapFS`). Functions like `fs.ReadFile`, `fs.WalkDir`, `fs.Glob`, and `template.ParseFS` and `http.FileServerFS` work with any of them. Same idea as `io.Reader`, one level up.

### `embed`: files inside your binary

```go
import "embed"

//go:embed templates/*.html static
var assets embed.FS

//go:embed version.txt
var version string
```

At build time the files are copied into the binary; `assets` behaves as an `fs.FS`. Deploying becomes copying one file (DT-2's advantage, extended to templates, migrations, and web assets). Patterns cannot contain `..`, and files beginning with `.` or `_` are excluded unless you use the `all:` prefix.

### Deep & Tricky

**DT-194. `path` and `path/filepath` are different on Windows.** *(Guarantee)* `path.Join` always uses `/`; `filepath.Join` uses the OS separator. Use `path` for URLs and `fs.FS` paths (always slash-separated, even on Windows), and `filepath` for real OS paths. Mixing them works on Linux and macOS and breaks on Windows.

### Mastery check

1. **Explain.** Why is `fs.FS` useful for testing code that reads templates?
2. **Predict.** `filepath.Clean("/a/../../b")`, `filepath.Ext("noext")`, `filepath.Join("a", "/b")`.
3. **Write.** `du(dir string) (map[string]int64, error)` that sums file sizes per file extension under a directory, skipping hidden directories; test it with `fstest.MapFS` by writing it against `fs.FS`.

## Topic 5B — Data formats

## Unit 5.4 — encoding/json

**One-line summary:** `json.Marshal` and `json.Unmarshal` convert between Go values and JSON using struct fields and tags; `json.Encoder`/`Decoder` do the same over streams. Know the traps: only exported fields, `omitempty` vs `omitzero`, numbers becoming `float64`, unknown fields silently ignored, and nil vs empty slices. Since Go 1.27, `encoding/json` runs on the new v2 implementation, and `encoding/json/v2` is available with stricter defaults.

### Struct tags control the mapping

```go
type Order struct {
	ID        string    `json:"id"`
	Customer  string    `json:"customer"`
	Items     []Item    `json:"items"`
	Total     int64     `json:"total_paise"`
	Note      string    `json:"note,omitempty"` // omitted when ""
	CreatedAt time.Time `json:"created_at"`     // RFC 3339 text
	Paid      *bool     `json:"paid,omitempty"` // nil = unknown, distinct from false
	internal  int       // unexported: never encoded (DT-167)
	Secret    string    `json:"-"`              // never encoded
}

data, err := json.Marshal(order)             // []byte
data, err := json.MarshalIndent(order, "", "  ")

var o Order
err = json.Unmarshal(data, &o)               // pass a POINTER
```

Matching on decode is **case-insensitive** on field names (`"ID"`, `"id"`, `"Id"` all fill `ID`) in v1 semantics. Unknown fields in the JSON are **silently ignored** by default.

### Type mapping

| Go | JSON |
| --- | --- |
| `bool` | `true` / `false` |
| integers, floats | number |
| `string` | string (invalid UTF-8 replaced with U+FFFD in v1; rejected in v2) |
| struct | object |
| `map[string]T` (also integer keys, as strings) | object, keys sorted on encode |
| slice, array | array; **nil slice → `null`**, empty slice → `[]` |
| `[]byte` | base64 string |
| pointer | the pointed-to value, or `null` |
| `time.Time` | RFC 3339 string |
| `any` on decode | `map[string]any`, `[]any`, `float64`, `string`, `bool`, `nil` |

### The traps

**Numbers into `any` become `float64`** — and integers above 2^53 lose precision (DT-66):

```go
var v any
json.Unmarshal([]byte(`{"id": 9007199254740993}`), &v)
// v["id"] is float64 9.007199254740992e+15 — the ID changed!
```

Decode into a typed struct (`ID int64`), or use `dec.UseNumber()` to get `json.Number` (a string you convert yourself).

**`omitempty` does not omit zero structs** (including `time.Time`):

```go
type A struct {
	T  time.Time `json:"t,omitempty"`  // still encoded: "0001-01-01T00:00:00Z"
	T2 time.Time `json:"t2,omitzero"`  // Go 1.24+: omitted when zero (or when IsZero() reports true)
}
```

`omitempty` omits false, 0, "", nil pointers and interfaces, and empty slices and maps — but never structs. Use `omitzero` (Go 1.24) or a pointer.

**Unknown fields and typos.** A client sends `"custmer"`; your `Customer` stays empty and no error is reported. For strict APIs:

```go
dec := json.NewDecoder(r)
dec.DisallowUnknownFields()
err := dec.Decode(&o) // json: unknown field "custmer"
```

### Streaming

```go
enc := json.NewEncoder(w)   // writes to any io.Writer, one value per Encode, newline-terminated
enc.SetIndent("", "  ")
enc.Encode(report)

dec := json.NewDecoder(r)   // reads from any io.Reader
for {
	var e Event
	if err := dec.Decode(&e); err == io.EOF {
		break
	} else if err != nil {
		return err
	}
	handle(e)
}
```

This handles **JSON Lines** (one object per line) in constant memory — the standard format for logs and data pipelines.

### Custom marshalling

Implement `json.Marshaler` / `json.Unmarshaler` (or `encoding.TextMarshaler`/`TextUnmarshaler`, used also for map keys and other encodings) to control a type's form:

```go
func (g Genre) MarshalText() ([]byte, error) { return []byte(g.String()), nil }

func (g *Genre) UnmarshalText(b []byte) error {
	v, err := ParseGenre(string(b))
	if err != nil {
		return err
	}
	*g = v
	return nil
}
```

Now `Genre` appears in JSON as `"science"` instead of `2` — readable and stable even if the constant order changes.

### JSON v2 (Go 1.27)

Go 1.27 added `encoding/json/v2` and the low-level `encoding/json/jsontext`, and re-implemented the classic `encoding/json` on top of the new engine (with options that keep v1 behaviour; unmarshalling became significantly faster). The v2 API (`json.Marshal(v, opts...)`, `json.UnmarshalRead(r, &v)`, `MarshalWrite`) has **stricter defaults**: it rejects invalid UTF-8 and duplicate object names, matches field names case-sensitively, and encodes nil slices and maps as `[]` and `{}`. For new code that handles untrusted input on Go 1.27+, v2's strictness removes several of the traps above. Existing code using `encoding/json` keeps working; error message *texts* may differ, so never match on them.

### Deep & Tricky

**DT-195. Unmarshalling into a non-empty value merges.** *(Guarantee)* `json.Unmarshal` into a struct that already has values only overwrites fields present in the JSON; others keep their old values. Into a map, it adds keys. Reusing one variable across decodes in a loop can leak values from the previous record. Declare a fresh variable per record.

**DT-196. Duplicate keys: v1 takes the last, v2 rejects.** *(Version-dependent behaviour)* `{"role":"user","role":"admin"}` decodes as `admin` with v1 semantics. If one system validates with the first value and another uses the last, attackers can exploit the difference. v2's default rejection closes the hole.

**DT-197. HTML characters are escaped by default.** *(v1 behaviour)* `json.Marshal` escapes `<`, `>`, and `&` as `<` and so on, so JSON can be embedded in HTML safely. Use an `Encoder` with `SetEscapeHTML(false)` if the output is not going into HTML and must be human-readable.

### Pattern

**P-30. Separate wire types from domain types.** *(Design practice)* Define small structs that mirror the JSON exactly (tags, pointers for optional fields), decode into them, validate, and convert to domain types. Changing the API format then never ripples through business logic, and validation has one obvious place. *Trade-off:* some duplicated fields and conversion code.

### Mastery check

1. **Explain.** Why does `{"id": 9007199254740993}` decoded into `any` come back as a different number? Name two fixes.
2. **Predict.** Encoding `struct{ S []int \`json:"s"\` }{}`; `struct{ S []int \`json:"s,omitempty"\` }{S: []int{}}`; a struct with an unexported field only.
3. **Write.** Give the `Genre` enum of Chapter 3 `MarshalText`/`UnmarshalText`, and a test that a `Book` round-trips through JSON with `"genre":"science"` and that `"genre":"poetry"` produces an error.

## Unit 5.5 — CSV and binary formats

**One-line summary:** `encoding/csv` reads and writes comma-separated records correctly (quotes, embedded commas, newlines inside fields). `encoding/binary` converts numbers to and from bytes with an explicit byte order, which is how file formats and network protocols are built.

### CSV

```go
r := csv.NewReader(f)
r.FieldsPerRecord = 3   // error if a row has a different number of fields (0 = same as first row)
r.TrimLeadingSpace = true
for {
	rec, err := r.Read() // []string
	if err == io.EOF {
		break
	}
	if err != nil {
		return err // *csv.ParseError includes line and column
	}
	...
}

w := csv.NewWriter(os.Stdout)
w.Write([]string{"name", "city"})
w.Write([]string{"Rao, Asha", "Pune"}) // written as "Rao, Asha",Pune
w.Flush()
if err := w.Error(); err != nil { ... }
```

Never split CSV with `strings.Split(line, ",")`: quoted fields with commas or newlines break it. `r.Comma = '\t'` handles tab-separated files; `r.ReuseRecord = true` avoids allocating a new slice per row in hot loops.

### Binary encoding and byte order

```go
buf := make([]byte, 8)
binary.BigEndian.PutUint32(buf, 0xCAFEBABE)          // CA FE BA BE
binary.LittleEndian.PutUint32(buf[4:], 0xCAFEBABE)   // BE BA FE CA
x := binary.BigEndian.Uint32(buf)                    // 0xCAFEBABE

b := binary.BigEndian.AppendUint16(nil, 443)         // append form (Go 1.19+)

// Variable-length integers: small numbers take fewer bytes
b = binary.AppendUvarint(b, 300)                     // 2 bytes: AC 02
v, n := binary.Uvarint(b[2:])

// Fixed-size structs (all fields fixed-size numbers)
type Header struct {
	Magic   uint32
	Version uint16
	Flags   uint16
}
err := binary.Read(r, binary.LittleEndian, &h)
```

Remember DT-16: the format's specification decides the byte order, never the machine's. Network protocols are usually big-endian ("network byte order"); many file formats (and most CPUs) are little-endian. Mixing them up produces numbers that look like garbage: 1 becomes 16,777,216.

### Other formats in the standard library

`encoding/xml`, `encoding/base64`, `encoding/hex`, `encoding/pem`, `encoding/gob` (Go-to-Go only), `archive/zip`, `archive/tar`, `compress/gzip`, `compress/zlib`, `hash/crc32`, `crypto/sha256`. The `uuid` package (Go 1.27) generates and parses UUIDs. Protocol Buffers (Unit 8.8) and YAML/TOML need external modules.

### Deep & Tricky

**DT-198. `binary.Read` with a struct is reflection-based and slow.** *(Implementation detail)* It is convenient for headers, but for millions of records decode fields manually with `binary.LittleEndian.Uint32(buf[0:4])` and so on — often more than ten times faster (measure, Chapter 10).

**DT-199. A UTF-8 byte-order mark can hide in CSV files from Excel.** *(Real-world format quirk)* Files saved as "CSV UTF-8" by spreadsheet programs often start with the bytes `EF BB BF`. The first header becomes `"﻿name"` and `rec[0] == "name"` is false. Strip a leading BOM before parsing.

### Mastery check

1. **Explain.** Why is `strings.Split` wrong for CSV? Show a row that breaks it.
2. **Predict.** The bytes of `binary.BigEndian.PutUint16` and `LittleEndian.PutUint16` for 258; `binary.AppendUvarint(nil, 127)` and `(nil, 128)` lengths.
3. **Write.** A tiny binary file format: a header (magic `"GOAP"`, version uint16) followed by records (uint32 ID, uvarint name length, name bytes). Write an encoder and decoder with tests, including a truncated file.

## Topic 5C — Time, command-line tools, and logging

## Unit 5.6 — time

**One-line summary:** `time.Time` is an instant (with a location for display), `time.Duration` is an `int64` count of nanoseconds. Formatting uses a reference time (`Mon Jan 2 15:04:05 MST 2006`) instead of `%Y`-style codes. Compare instants with `Before`/`After`/`Equal`, measure elapsed time with the monotonic clock, store and transmit times in UTC.

### Instants and durations

```go
now := time.Now()
later := now.Add(90 * time.Minute)
d := later.Sub(now)            // time.Duration: 1h30m0s
d.Minutes()                    // 90 (float64)
time.Since(start)              // shorthand for time.Now().Sub(start)
time.Until(deadline)

t := time.Date(2026, time.November, 15, 14, 5, 9, 0, time.UTC)
t.Year(), t.Month(), t.Day(), t.Weekday(), t.YearDay()
t.AddDate(0, 1, 0)             // calendar arithmetic: one month later
t.Truncate(time.Hour)          // 14:00:00
```

`AddDate` normalises: `Jan 31 + 1 month` is `Feb 31`, which normalises to **March 3** (in 2026). Calendar arithmetic needs explicit business rules ("end of month" billing must handle this).

### Comparing

Use `t1.Before(t2)`, `t1.After(t2)`, `t1.Equal(t2)`, and `t1.Compare(t2)` (Go 1.20). Do not use `==`: two `Time` values for the same instant can differ in location or monotonic reading, making `==` false (DT-200).

### Formatting with the reference time

Go formats by example. The reference instant is `Mon Jan 2 15:04:05 MST 2006` — memorable as **1 2 3 4 5 6 7**: month 1, day 2, hour 3 (15), minute 4, second 5, year 6, zone -7.

```go
t.Format("2006-01-02")               // 2026-11-15
t.Format("02/01/2006 3:04PM")        // 15/11/2026 2:05PM
t.Format(time.RFC3339)               // 2026-11-15T14:05:09Z — use for machines
t.Format(time.DateTime)              // 2026-11-15 14:05:09 (Go 1.20 constants: DateTime, DateOnly, TimeOnly)
t.Format("YYYY-MM-DD")               // "YYYY-MM-DD" — literal text! No error.
time.Parse("2006-01-02", "2026-11-15")
time.ParseInLocation("2006-01-02 15:04", "2026-11-15 09:30", ist)
```

### Time zones

```go
ist, err := time.LoadLocation("Asia/Kolkata") // needs the tz database (embed with import _ "time/tzdata")
fmt.Println(t.In(ist))                        // 2026-11-15 19:35:09 +0530 IST
```

A `Time` stores the instant once; the location only affects display and calendar calculations. Store and transmit in UTC (`t.UTC()`), convert to a user's zone at the edge. A day is not always 24 hours: across daylight-saving changes, `t.AddDate(0, 0, 1)` and `t.Add(24 * time.Hour)` give different results in zones that observe DST.

### The monotonic clock

`time.Now()` reads two clocks: the **wall clock** (can jump when NTP corrects it or an admin changes it) and the **monotonic clock** (only moves forward). `t2.Sub(t1)` uses the monotonic readings when both values have them, so elapsed-time measurements are correct even if the wall clock jumps. `fmt.Println(time.Now())` shows the monotonic part as `m=+0.000123`. Operations that produce a "calendar" time (`t.Round(0)`, `t.In`, `t.UTC`, parsing, decoding) strip it.

### Timers and tickers

```go
time.Sleep(200 * time.Millisecond)
<-time.After(time.Second)          // a channel that receives once (Chapter 7)
timer := time.NewTimer(5 * time.Second)
defer timer.Stop()
ticker := time.NewTicker(time.Minute)
defer ticker.Stop()
for t := range ticker.C { ... }
time.AfterFunc(d, f)               // run f in its own goroutine after d
```

Since Go 1.23, unreferenced timers and tickers are garbage-collected even if not stopped, and their channels are unbuffered, so `Reset` and `Stop` no longer leave a stale value behind (Go 1.27 removed the old behaviour completely). Chapter 7 uses timers with `select` for timeouts.

### Deep & Tricky

**DT-200. Never compare `time.Time` with `==`, and never use it as a map key without normalising.** *(Guarantee + trap)* `==` compares the location pointer and monotonic reading too. `t.Equal(u)` compares instants. As map keys, use `t.UTC().Round(0)` or a Unix timestamp.

**DT-201. `time.Duration(n) * time.Second` vs `time.Duration(n * time.Second)`.** *(Type rules)* With `n int`, `n * time.Second` does not compile (mismatched types, Unit 3.1). Write `time.Duration(n) * time.Second`. But `time.Duration(1.5) * time.Second` with a *constant* 1.5 is a compile error (truncated), and with a float variable converts 1.5 to 1 first. Use `time.Duration(f * float64(time.Second))` for fractional seconds.

**DT-202. Layout mistakes compile and fail silently.** *(fmt-like behaviour)* `t.Format("2006-01-02 15:04:05")` is right; `"2006-02-01"` swaps month and day; `"2006-01-02 03:04:05"` gives a 12-hour clock without AM/PM; `"YYYY"` is printed literally. Test formatting with a date whose day is greater than 12 (like the 15th) so swapped fields are visible (the DT-18 lesson again).

### Mastery check

1. **Explain.** What is the monotonic clock for, and why does `time.Since(start)` stay correct when NTP adjusts the system time?
2. **Predict.** (a) `time.Date(2026, 1, 31, 0, 0, 0, 0, time.UTC).AddDate(0, 1, 0)` (b) `t.Format("YYYY")` (c) `time.Duration(90) * time.Minute` (d) `time.Sleep(2)`.
3. **Write.** `nextBusinessDay(t time.Time) time.Time` (skip Saturday and Sunday), and `billingDate(start time.Time, month int) time.Time` that clamps to the last day of the month instead of overflowing; tests including January 31 and leap years.

## Unit 5.7 — Command-line tools

**One-line summary:** `flag` parses command-line flags; `os/signal` lets a program react to Ctrl+C and termination signals; `os/exec` runs other programs safely; exit codes and stdout/stderr follow the conventions of P-3.

### `flag`

```go
var (
	addr    = flag.String("addr", ":8080", "listen address")
	verbose = flag.Bool("v", false, "verbose output")
	timeout = flag.Duration("timeout", 5*time.Second, "request timeout")
)

func main() {
	flag.Parse()
	files := flag.Args() // remaining non-flag arguments
	...
}
```

- Accepts `-flag value`, `-flag=value`, `--flag`; booleans as `-v` or `-v=false`. Parsing stops at the first non-flag argument or `--`.
- `-h` / `-help` prints generated usage. `flag.Usage = func() {...}` customises it.
- For testable programs, use a `flag.NewFlagSet(name, flag.ContinueOnError)` inside `run(args)` instead of the global set (the Chapter 5 project does this).
- `flag.Var` and `flag.TextVar` accept custom types (any `encoding.TextUnmarshaler`, like the `Genre` of Unit 5.4).
- Subcommands (`git commit -m`) are done with one `FlagSet` per subcommand. Larger CLIs often use the `cobra` module; the standard library is enough for most tools.

### Signals

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()

for {
	select {
	case <-ctx.Done():
		fmt.Fprintln(os.Stderr, "interrupted, cleaning up")
		return saveProgress()
	case job := <-jobs:
		process(job)
	}
}
```

Ctrl+C sends `SIGINT` (`os.Interrupt`); `kill` and container orchestrators send `SIGTERM` and, after a grace period, the un-catchable `SIGKILL`. A good tool or server catches SIGINT/SIGTERM, stops taking new work, finishes or saves current work, and exits — **graceful shutdown** (Unit 8.7, Unit 11.8). `context` is fully explained in Unit 7.9; here, `ctx.Done()` simply means "a signal arrived". After the first signal, `stop()` restores the default behaviour, so a second Ctrl+C kills the program immediately.

### Running other programs: `os/exec`

```go
cmd := exec.CommandContext(ctx, "git", "log", "--oneline", "-n", "5")
cmd.Dir = repoDir
out, err := cmd.Output() // stdout; on failure err is *exec.ExitError with Stderr captured
if err != nil {
	var ee *exec.ExitError
	if errors.As(err, &ee) {
		return fmt.Errorf("git failed (exit %d): %s", ee.ExitCode(), ee.Stderr)
	}
	return err // e.g. git not installed: exec.ErrNotFound
}
```

- Arguments are passed directly to the program, **not through a shell**, so user input cannot inject shell commands (`; rm -rf /`). Never write `exec.Command("sh", "-c", "git log " + userInput)`.
- `cmd.Stdin`, `cmd.Stdout`, `cmd.Stderr` accept readers and writers (Unit 5.1) — stream large outputs instead of buffering them.
- `CommandContext` kills the process when the context is cancelled (timeouts).
- `exec.LookPath("git")` finds a program; since Go 1.19 it refuses to run programs from the current directory implicitly (a Windows security issue).

### Exit codes

`os.Exit(n)` with 0 for success, 1 for general failure, 2 for usage errors is the common convention (the `flag` package exits with 2 on bad flags when using `ExitOnError`). Remember that `os.Exit` skips deferred calls (Unit 1.11) — the `run()` pattern (P-14) keeps them working.

### Deep & Tricky

**DT-203. Child processes do not die automatically when your program is killed.** *(OS behaviour)* If your program is killed with SIGKILL, programs it started with `os/exec` keep running (orphans). `CommandContext` helps only when your process is alive to cancel the context. For robust supervision, use process groups (`SysProcAttr` on Unix) or a supervisor.

**DT-204. `cmd.Output()` with a huge output buffers it all in memory.** *(Library behaviour)* Like `io.ReadAll` (DT-191). For large or unbounded output, set `cmd.Stdout` to a writer or use `cmd.StdoutPipe()` and stream it.

### Mastery check

1. **Explain.** Why must a program that starts subprocesses never build a shell command string from user input?
2. **Predict.** With `fs := flag.NewFlagSet("x", flag.ContinueOnError)`, a `-n int` flag, and args `["-n", "3", "file.txt", "-v"]`: what are `n`, `fs.Args()`, and is `-v` parsed?
3. **Write.** `countdown -from 10 -interval 1s` that prints a countdown; on Ctrl+C it prints "aborted at N" and exits with code 130 (the shell convention: 128 + signal number 2).

## Unit 5.8 — Logging with log/slog

**One-line summary:** `log/slog` (Go 1.21) writes **structured** logs: a message plus typed key–value attributes, rendered as text or JSON, with levels. Structured logs can be searched and aggregated by machines, which plain `fmt.Println` lines cannot.

### Basics

```go
logger := slog.New(slog.NewJSONHandler(os.Stderr, &slog.HandlerOptions{Level: slog.LevelInfo}))
slog.SetDefault(logger) // also routes the old log package through it

slog.Info("order placed", "order_id", id, "amount_paise", amt, "items", len(items))
slog.Warn("slow query", "duration", d, "query", name)
slog.Error("payment failed", "order_id", id, "err", err)
```

```json
{"time":"2026-11-15T14:05:09.123Z","level":"INFO","msg":"order placed","order_id":"o-17","amount_paise":125000,"items":3}
```

- Levels: `Debug`, `Info`, `Warn`, `Error` (and any integer in between). The handler's `Level` filters.
- `slog.NewTextHandler` writes `key=value` lines — readable in development; JSON in production, where a log system indexes the keys.
- `logger.With("request_id", rid)` returns a logger that adds attributes to every record — pass it down a request's call chain (or put it in a `context`, Unit 7.9).
- For hot paths, use typed attributes to avoid allocations: `slog.Int("items", n)`, `slog.String("user", u)`, and `logger.LogAttrs(ctx, slog.LevelInfo, "msg", attrs...)`.
- `slog.Group("http", "method", m, "path", p)` nests attributes.

### What to log

- **Events with context**, not narration: "payment failed" with the order ID, amount, provider, and error — not "entering function X".
- **Errors once**, where they are handled (Unit 3.9's handle-once rule).
- **Never secrets or personal data**: passwords, tokens, full card numbers, OTPs. Implement `slog.LogValuer` on sensitive types to redact them automatically.
- Consistent key names across the codebase (`user_id`, not `userId` here and `uid` there), so queries work.

### Deep & Tricky

**DT-205. A mismatched key–value list produces `!BADKEY`.** *(Library behaviour)* `slog.Info("x", "user")` (missing value) logs `!BADKEY=user`. `go vet`'s `slog` check catches many of these; typed `slog.Attr` constructors avoid them entirely.

**DT-206. Arguments are evaluated even when the level is disabled.** *(Language rule)* `slog.Debug("state", "dump", expensiveDump())` calls `expensiveDump()` before `slog` checks the level. Guard expensive work with `if logger.Enabled(ctx, slog.LevelDebug)`, or pass a `slog.LogValuer` whose `LogValue` runs only if the record is emitted.

### Mastery check

1. **Explain.** What can you do with structured logs that you cannot do with `fmt.Printf` lines?
2. **Write.** A `User` type with `LogValue()` that logs only the ID and a masked email (`a***@example.com`); show the JSON output.

## Topic 5D — Text and numbers

## Unit 5.9 — Text processing

**One-line summary:** Beyond `strings` and `bytes` (Unit 2.6), the standard library offers `regexp` (guaranteed linear-time regular expressions), `text/template` for generating text, `html/template` for generating HTML safely, and `text/tabwriter` for aligned columns.

### `regexp`

```go
var datePat = regexp.MustCompile(`^(\d{4})-(\d{2})-(\d{2})$`) // compile once (Unit 3.11 Must)

datePat.MatchString("2026-11-15")              // true
m := datePat.FindStringSubmatch("2026-11-15")  // ["2026-11-15" "2026" "11" "15"]
re := regexp.MustCompile(`(?P<user>\w+)@(?P<host>[\w.]+)`)
re.ReplaceAllString(text, "$user at $host")
re.FindAllString(text, -1)
```

Go's `regexp` uses the RE2 algorithm: matching time is **linear** in the input size, guaranteed. It cannot be made to take exponential time by a malicious pattern or input (the "ReDoS" attacks that affect backtracking engines in other languages). The price: no backreferences (`\1`) and no lookaround. Use raw strings for patterns (`\d` without double escaping), compile once at package level, and prefer `strings` functions when no pattern is needed — they are much faster.

### `text/template`

```go
const report = `Order {{.ID}} for {{.Customer}}
{{range .Items}}  {{printf "%-20s" .Name}} x{{.Qty}}
{{end}}{{if .Paid}}PAID{{else}}DUE{{end}}
`
tmpl := template.Must(template.New("report").Parse(report))
err := tmpl.Execute(os.Stdout, order)
```

`{{.Field}}` accesses fields and methods, `{{range}}`, `{{if}}`, `{{with}}`, `{{template "name" .}}`, pipelines (`{{.Name | printf "%q"}}`), and custom functions with `Funcs`. Templates are good for emails, reports, generated code, and configuration files.

### `html/template`: safe HTML

`html/template` has the same API but **escapes data according to context**: inside HTML text, inside an attribute, inside JavaScript, inside a URL. A user name of `<script>alert(1)</script>` is rendered harmlessly. Always use `html/template` for HTML; using `text/template` for HTML is a cross-site scripting (XSS) vulnerability (Unit 11.2).

### `text/tabwriter`

Aligns columns by padding cells — used by the Chapter 5 project's text report:

```go
tw := tabwriter.NewWriter(os.Stdout, 0, 0, 2, ' ', 0)
fmt.Fprintln(tw, "NAME\tCOUNT")
fmt.Fprintln(tw, "go\t42")
tw.Flush()
```

### Deep & Tricky

**DT-207. `regexp` matches are leftmost-first, like Perl — not longest.** *(Library behaviour)* `regexp.MustCompile("a|ab").FindString("ab")` is `"a"`. Use `CompilePOSIX` (or `re.Longest()`) for POSIX leftmost-longest semantics.

**DT-208. Templates fail at execution, not only at parse.** *(Library behaviour)* `{{.Nmae}}` (a typo) parses fine and fails only when executed with data that has no `Nmae` field — possibly halfway through writing output. Test every template with realistic data, and render into a buffer first if partial output would be harmful.

### Mastery check

1. **Explain.** What is ReDoS, and why is Go's `regexp` immune?
2. **Write.** A template that renders the `wordfreq` top-N as a Markdown table; then render the same data as an HTML table with `html/template` and show that a word containing `<b>` is escaped.

## Unit 5.10 — Exact numbers and secure randomness

**One-line summary:** `math/big` gives arbitrary-precision integers (`big.Int`), rationals (`big.Rat`), and floats (`big.Float`). Money uses integer minor units (P-8) or a decimal library with explicit rounding. `crypto/rand` provides secrets (P-15).

### `math/big`

```go
a := new(big.Int)
a.SetString("123456789012345678901234567890", 10)
b := big.NewInt(1_000_000_007)
c := new(big.Int).Mul(a, a)       // result goes into the receiver
c.Mod(c, b)
fmt.Println(c.String())

f := new(big.Int).MulRange(1, 50) // 50!
r := big.NewRat(1, 3)             // exact 1/3
r.Add(r, big.NewRat(1, 6))        // exactly 1/2
```

The API style is unusual: methods store the result in the receiver and also return it (`z.Add(x, y)` sets `z = x + y`), so you can reuse values and avoid allocations. Values must not be copied (use pointers). `big.Int` is used for cryptography, exact combinatorics, and IDs that overflow `uint64`. Go 1.27 added `Int.Divide` with explicit rounding modes (`Trunc`, `Floor`, `Round`, `Ceil`).

### Money, revisited

- Store amounts as `int64` minor units (paise, cents) — P-8. `int64` holds ±92 quadrillion rupees in paise; enough.
- For rates, taxes, conversions, and interest, compute with `big.Rat` or a decimal library (`github.com/shopspring/decimal`, `github.com/cockroachdb/apd`) and **round once, explicitly**, at a business-defined point ("round half to even at the invoice line").
- Splitting ₹100 three ways is 3333 + 3333 + 3334 paise — someone gets the extra paisa, and the rule for who is a business decision written in code.

### `crypto/rand`

```go
token := rand.Text()            // Go 1.24+: 26 base32 characters, 128+ bits of randomness
b := make([]byte, 32)
rand.Read(b)                    // never fails since Go 1.24 (DT-97)
n, err := rand.Int(rand.Reader, big.NewInt(1_000_000)) // uniform in [0, 1e6): a 6-digit OTP
otp := fmt.Sprintf("%06d", n)
```

Use it for every secret (P-15). Go 1.26 also made the cryptographic packages ignore caller-provided randomness sources in key generation, always using the secure source (tests use `testing/cryptotest.SetGlobalRandom`).

### Mastery check

1. **Explain.** Why does `big.Int`'s API put the result in the receiver?
2. **Write.** `split(total int64, n int) []int64` that splits paise as evenly as possible (differences of at most 1 paisa, sum exactly equal to total), plus tests for 10000 ÷ 3 and negative totals.

## Topic 5E — Learning from the source

## Unit 5.11 — Reading standard-library source code

**One-line summary:** The standard library is the largest body of idiomatic, reviewed, production Go you can read for free, and it is on your machine. Reading it teaches patterns, edge cases, and performance techniques no book covers fully.

### How to find and read it

```
$ go env GOROOT                  # where your Go installation lives
$ go doc -src strings.Cut        # print a function's source
$ go doc -src io.Copy
$ go doc bytes.ExampleBuffer     # Go 1.27: print an example's source
```

Or jump to definition in your editor, or browse `cs.opensource.google/go/go`. Start with small, self-contained packages:

| Package | What you learn |
| --- | --- |
| `strings` (`Cut`, `Fields`, `Builder`) | fast paths for ASCII, careful allocation, the copy check of DT-118 |
| `io` (`Copy`, `LimitReader`, `MultiReader`, `Pipe`) | interface composition, optional fast paths (`WriterTo`) |
| `bufio` (`Scanner`) | buffer management, `SplitFunc` design, error semantics |
| `errors` (`Is`, `As`, `Join`) | how wrapping is traversed |
| `sort` / `slices` (`pdqsort`) | a modern sorting algorithm with fallbacks |
| `sync` (`Once`, `WaitGroup`) | memory-model-correct synchronisation (Chapter 7) |
| `container/heap`, `container/list` | classic data structures in Go style (Chapter 6) |
| `net/http` (`Server`, `Transport`) | large-scale API design (Chapter 8) |

### How to read

1. Read the **package documentation** and the exported API first — the contract.
2. Read the **tests** and **examples**: they show intended use and edge cases.
3. Read one function at a time and ask: why this check first? why this buffer size? what does this comment protect against? Many comments cite issues (`go.dev/issue/NNNN`) — read them for the history of a decision.
4. Notice the conventions you now know: guard clauses, `(value, error)`, small interfaces, zero values that work, careful documentation of every exported name.

### Mastery check

1. **Write.** Read the source of `strings.Fields`. Explain its ASCII fast path and why it counts fields before allocating the result.
2. **Write.** Read `io.Copy` (`copyBuffer`). List the optional interfaces it checks and in what order, and why `LimitReader` hides one of them.

## Chapter 5 Project — `logscan`, a streaming log analyzer

**Goal:** a professional command-line tool that summarises HTTP access logs of any size in constant memory, reads plain or gzip-compressed files or stdin, filters by time, prints a text or JSON report, logs problems in structured form, and stops gracefully on Ctrl+C with a partial report.

### Input format

```
2026-11-01T10:00:00Z GET /api/users 200 123ms 512
time (RFC 3339)      method path status duration bytes
```

### Specification

```
logscan [-from T] [-to T] [-top N] [-slow D] [-json] [file ...]
```

- `-from` (inclusive) and `-to` (exclusive) filter by time; `-top` (default 5) paths; `-slow` (default 500ms) threshold; `-json` switches the output format.
- Report: requests, invalid lines, bytes, time range, counts per status class, slow requests, latency p50/p95/p99, top paths (count descending, then path).
- Invalid lines are counted; the first three are logged with `slog` (file, line number, reason) on stderr.
- Memory must not grow with the number of lines (it may grow with the number of *distinct paths*).
- Ctrl+C stops reading and prints the report marked partial.
- Bad flags → error and exit 1.

### Design notes

- **Constant-memory percentiles.** Storing every duration to sort it is O(lines) memory. Instead, count requests in 1 ms buckets up to 60 s (a fixed array of 60,001 counters, about 480 KB). The p95 is the first bucket where the cumulative count reaches 95% of the total — exact to 1 ms. This is a **histogram**, the same idea monitoring systems such as Prometheus use (Unit 11.5).
- **Composition.** `os.File` → (optional) `gzip.Reader` → `bufio.Scanner`. Nothing else in the program knows whether the input was compressed.
- **Testability.** `run(ctx, args, stdin, stdout, stderr)` with its own `FlagSet`; tests pass strings and buffers.

### Reference solution — `main.go`

```go
// Command logscan summarises HTTP access logs in constant memory.
//
//	logscan [flags] [file ...]
//
// Each input line has six space-separated fields:
//
//	2026-11-01T10:00:00Z GET /api/users 200 123ms 512
//	time (RFC 3339)      method path status duration bytes
//
// Files ending in .gz are decompressed on the fly. With no files, logscan
// reads standard input. Ctrl+C stops reading and prints a partial report.
package main

import (
	"bufio"
	"compress/gzip"
	"context"
	"encoding/json"
	"errors"
	"flag"
	"fmt"
	"io"
	"log/slog"
	"os"
	"os/signal"
	"strings"
	"time"
)

type config struct {
	from, to time.Time
	top      int
	slow     time.Duration
	asJSON   bool
}

func main() {
	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
	defer stop()
	if err := run(ctx, os.Args[1:], os.Stdin, os.Stdout, os.Stderr); err != nil {
		fmt.Fprintln(os.Stderr, "logscan:", err)
		os.Exit(1)
	}
}

func run(ctx context.Context, args []string, stdin io.Reader, stdout, stderr io.Writer) error {
	fs := flag.NewFlagSet("logscan", flag.ContinueOnError)
	fs.SetOutput(stderr)
	var cfg config
	var from, to string
	fs.StringVar(&from, "from", "", "only lines at or after this RFC 3339 time")
	fs.StringVar(&to, "to", "", "only lines before this RFC 3339 time")
	fs.IntVar(&cfg.top, "top", 5, "number of paths to list")
	fs.DurationVar(&cfg.slow, "slow", 500*time.Millisecond, "requests at least this long count as slow")
	fs.BoolVar(&cfg.asJSON, "json", false, "print the report as JSON")
	if err := fs.Parse(args); err != nil {
		return err // flag already printed the message and usage
	}
	var err error
	if cfg.from, err = parseTimeFlag("from", from); err != nil {
		return err
	}
	if cfg.to, err = parseTimeFlag("to", to); err != nil {
		return err
	}
	if cfg.top < 1 {
		return fmt.Errorf("-top must be at least 1, got %d", cfg.top)
	}

	log := slog.New(slog.NewTextHandler(stderr, &slog.HandlerOptions{Level: slog.LevelWarn}))
	st := newStats()

	if fs.NArg() == 0 {
		if err := scan(ctx, stdin, "stdin", cfg, st, log); err != nil {
			return err
		}
	}
	for _, name := range fs.Args() {
		if err := scanFile(ctx, name, cfg, st, log); err != nil {
			return err
		}
	}

	rep := st.report(cfg)
	if ctx.Err() != nil {
		rep.Partial = true
	}
	if cfg.asJSON {
		enc := json.NewEncoder(stdout)
		enc.SetIndent("", "  ")
		return enc.Encode(rep)
	}
	return rep.writeText(stdout)
}

func parseTimeFlag(name, v string) (time.Time, error) {
	if v == "" {
		return time.Time{}, nil
	}
	t, err := time.Parse(time.RFC3339, v)
	if err != nil {
		return time.Time{}, fmt.Errorf("-%s: %w", name, err)
	}
	return t, nil
}

func scanFile(ctx context.Context, name string, cfg config, st *stats, log *slog.Logger) error {
	f, err := os.Open(name)
	if err != nil {
		return err
	}
	defer f.Close()

	var r io.Reader = f
	if strings.HasSuffix(name, ".gz") {
		gz, err := gzip.NewReader(f)
		if err != nil {
			return fmt.Errorf("%s: %w", name, err)
		}
		defer gz.Close()
		r = gz // readers compose: file -> gzip -> scanner
	}
	return scan(ctx, r, name, cfg, st, log)
}

// scan reads r line by line. Memory use does not depend on the input size.
func scan(ctx context.Context, r io.Reader, name string, cfg config, st *stats, log *slog.Logger) error {
	sc := bufio.NewScanner(r)
	lineNo := 0
	for sc.Scan() {
		lineNo++
		if lineNo%4096 == 0 && ctx.Err() != nil {
			return nil // interrupted: keep what we have
		}
		e, err := parseLine(sc.Text())
		if err != nil {
			st.invalid++
			if st.invalid <= 3 {
				log.Warn("skipping invalid line", "file", name, "line", lineNo, "err", err)
			}
			continue
		}
		if !cfg.from.IsZero() && e.time.Before(cfg.from) {
			continue
		}
		if !cfg.to.IsZero() && !e.time.Before(cfg.to) {
			continue
		}
		st.add(e, cfg)
	}
	if err := sc.Err(); err != nil {
		return fmt.Errorf("%s: %w", name, err)
	}
	return nil
}

var errFields = errors.New("want 6 fields")

type entry struct {
	time     time.Time
	method   string
	path     string
	status   int
	duration time.Duration
	bytes    int64
}

func parseLine(line string) (entry, error) {
	f := strings.Fields(line)
	if len(f) != 6 {
		return entry{}, fmt.Errorf("%w, got %d", errFields, len(f))
	}
	var e entry
	var err error
	if e.time, err = time.Parse(time.RFC3339, f[0]); err != nil {
		return entry{}, err
	}
	e.method, e.path = f[1], f[2]
	if _, err := fmt.Sscanf(f[3], "%d", &e.status); err != nil || e.status < 100 || e.status > 599 {
		return entry{}, fmt.Errorf("bad status %q", f[3])
	}
	if e.duration, err = time.ParseDuration(f[4]); err != nil {
		return entry{}, err
	}
	if _, err := fmt.Sscanf(f[5], "%d", &e.bytes); err != nil || e.bytes < 0 {
		return entry{}, fmt.Errorf("bad byte count %q", f[5])
	}
	return e, nil
}
```

### Reference solution — `stats.go`

```go
package main

import (
	"cmp"
	"fmt"
	"io"
	"maps"
	"slices"
	"text/tabwriter"
	"time"
)

// Latencies are counted in 1 ms buckets up to maxBucket; longer requests
// share the last bucket. Fixed memory, and percentiles exact to 1 ms.
const maxBucket = 60_000 // 60 s

type stats struct {
	total, invalid, slow int
	bytes                int64
	byClass              [6]int // index 1..5 = 1xx..5xx
	byPath               map[string]int
	latency              [maxBucket + 1]int
	first, last          time.Time
}

func newStats() *stats { return &stats{byPath: make(map[string]int)} }

func (s *stats) add(e entry, cfg config) {
	s.total++
	s.bytes += e.bytes
	s.byClass[e.status/100]++
	s.byPath[e.path]++ // distinct paths: memory grows with paths, not lines
	ms := min(int(e.duration/time.Millisecond), maxBucket)
	s.latency[ms]++
	if e.duration >= cfg.slow {
		s.slow++
	}
	if s.first.IsZero() || e.time.Before(s.first) {
		s.first = e.time
	}
	if e.time.After(s.last) {
		s.last = e.time
	}
}

// percentile returns the smallest bucket value with at least p of requests at or below it.
func (s *stats) percentile(p float64) time.Duration {
	if s.total == 0 {
		return 0
	}
	need := int(p*float64(s.total) + 0.999999) // ceil
	seen := 0
	for ms, n := range s.latency {
		seen += n
		if seen >= need {
			return time.Duration(ms) * time.Millisecond
		}
	}
	return maxBucket * time.Millisecond
}

type pathCount struct {
	Path  string `json:"path"`
	Count int    `json:"count"`
}

// Report is what logscan prints; the struct tags define the JSON form.
type Report struct {
	Requests     int            `json:"requests"`
	InvalidLines int            `json:"invalid_lines"`
	Bytes        int64          `json:"bytes"`
	First        *time.Time     `json:"first,omitempty"`
	Last         *time.Time     `json:"last,omitempty"`
	StatusClass  map[string]int `json:"status_classes"`
	Slow         int            `json:"slow_requests"`
	P50          string         `json:"p50"`
	P95          string         `json:"p95"`
	P99          string         `json:"p99"`
	TopPaths     []pathCount    `json:"top_paths"`
	Partial      bool           `json:"partial,omitempty"`
}

func (s *stats) report(cfg config) Report {
	r := Report{
		Requests:     s.total,
		InvalidLines: s.invalid,
		Bytes:        s.bytes,
		StatusClass:  map[string]int{},
		Slow:         s.slow,
		P50:          s.percentile(0.50).String(),
		P95:          s.percentile(0.95).String(),
		P99:          s.percentile(0.99).String(),
	}
	if s.total > 0 {
		r.First, r.Last = &s.first, &s.last
	}
	for c := 1; c <= 5; c++ {
		if s.byClass[c] > 0 {
			r.StatusClass[fmt.Sprintf("%dxx", c)] = s.byClass[c]
		}
	}
	for _, p := range slices.Collect(maps.Keys(s.byPath)) {
		r.TopPaths = append(r.TopPaths, pathCount{p, s.byPath[p]})
	}
	slices.SortFunc(r.TopPaths, func(a, b pathCount) int {
		return cmp.Or(cmp.Compare(b.Count, a.Count), cmp.Compare(a.Path, b.Path))
	})
	r.TopPaths = r.TopPaths[:min(cfg.top, len(r.TopPaths))]
	return r
}

func (r Report) writeText(w io.Writer) error {
	tw := tabwriter.NewWriter(w, 0, 0, 2, ' ', 0)
	fmt.Fprintf(tw, "requests\t%d\n", r.Requests)
	fmt.Fprintf(tw, "invalid lines\t%d\n", r.InvalidLines)
	fmt.Fprintf(tw, "bytes sent\t%d\n", r.Bytes)
	if r.First != nil {
		fmt.Fprintf(tw, "time range\t%s – %s\n", r.First.Format(time.RFC3339), r.Last.Format(time.RFC3339))
	}
	for _, c := range slices.Sorted(maps.Keys(r.StatusClass)) {
		fmt.Fprintf(tw, "status %s\t%d\n", c, r.StatusClass[c])
	}
	fmt.Fprintf(tw, "slow requests\t%d\n", r.Slow)
	fmt.Fprintf(tw, "latency p50/p95/p99\t%s / %s / %s\n", r.P50, r.P95, r.P99)
	fmt.Fprintln(tw, "top paths\t")
	for _, p := range r.TopPaths {
		fmt.Fprintf(tw, "  %s\t%d\n", p.Path, p.Count)
	}
	if r.Partial {
		fmt.Fprintln(tw, "(partial: interrupted)\t")
	}
	return tw.Flush()
}
```

Note the pointers `First` and `Last`: with plain `time.Time` fields, `omitempty` would still print `0001-01-01T00:00:00Z` for an empty report (Unit 5.4). On Go 1.24+ `time.Time` with `omitzero` is the cleaner alternative.

### Reference solution — `main_test.go`

```go
package main

import (
	"bytes"
	"compress/gzip"
	"context"
	"encoding/json"
	"fmt"
	"os"
	"path/filepath"
	"strings"
	"testing"
	"time"
)

const sample = `2026-11-01T10:00:00Z GET /api/users 200 120ms 512
2026-11-01T10:00:01Z GET /api/users 200 80ms 512
2026-11-01T10:00:02Z POST /api/orders 201 640ms 64
2026-11-01T10:00:03Z GET /health 200 1ms 2
this line is broken
2026-11-01T10:00:04Z GET /api/users 500 2s 0
`

func TestParseLine(t *testing.T) {
	e, err := parseLine("2026-11-01T10:00:00Z GET /x 404 1.5s 10")
	if err != nil {
		t.Fatal(err)
	}
	if e.status != 404 || e.duration != 1500*time.Millisecond || e.path != "/x" {
		t.Errorf("got %+v", e)
	}
	for _, bad := range []string{"", "a b c", "2026-11-01T10:00:00Z GET /x 999 1s 1", "2026-11-01T10:00:00Z GET /x 200 fast 1"} {
		if _, err := parseLine(bad); err == nil {
			t.Errorf("parseLine(%q) succeeded", bad)
		}
	}
}

func TestReportJSON(t *testing.T) {
	var out, errOut bytes.Buffer
	err := run(context.Background(), []string{"-json", "-top", "2"}, strings.NewReader(sample), &out, &errOut)
	if err != nil {
		t.Fatal(err)
	}
	var r Report
	if err := json.Unmarshal(out.Bytes(), &r); err != nil {
		t.Fatal(err)
	}
	if r.Requests != 5 || r.InvalidLines != 1 || r.Slow != 2 {
		t.Errorf("requests=%d invalid=%d slow=%d", r.Requests, r.InvalidLines, r.Slow)
	}
	if r.StatusClass["2xx"] != 4 || r.StatusClass["5xx"] != 1 {
		t.Errorf("status classes = %v", r.StatusClass)
	}
	if len(r.TopPaths) != 2 || r.TopPaths[0].Path != "/api/users" || r.TopPaths[0].Count != 3 {
		t.Errorf("top paths = %+v", r.TopPaths)
	}
	if r.P50 != "120ms" || r.P99 != "2s" {
		t.Errorf("p50=%s p99=%s", r.P50, r.P99)
	}
	if !strings.Contains(errOut.String(), "skipping invalid line") {
		t.Errorf("no warning logged: %q", errOut.String())
	}
}

func TestTimeFilterAndGzip(t *testing.T) {
	path := filepath.Join(t.TempDir(), "access.log.gz")
	var buf bytes.Buffer
	zw := gzip.NewWriter(&buf)
	zw.Write([]byte(sample))
	zw.Close()
	if err := os.WriteFile(path, buf.Bytes(), 0o644); err != nil {
		t.Fatal(err)
	}
	var out bytes.Buffer
	args := []string{"-json", "-from", "2026-11-01T10:00:01Z", "-to", "2026-11-01T10:00:04Z", path}
	if err := run(context.Background(), args, nil, &out, new(bytes.Buffer)); err != nil {
		t.Fatal(err)
	}
	var r Report
	json.Unmarshal(out.Bytes(), &r)
	if r.Requests != 3 { // 10:00:01, :02, :03
		t.Errorf("requests = %d, want 3", r.Requests)
	}
}

func TestBadFlags(t *testing.T) {
	for _, args := range [][]string{{"-top", "0"}, {"-from", "yesterday"}, {"-nosuchflag"}} {
		if err := run(context.Background(), args, strings.NewReader(""), new(bytes.Buffer), new(bytes.Buffer)); err == nil {
			t.Errorf("run(%q) succeeded", args)
		}
	}
}

func BenchmarkScan(b *testing.B) {
	var sb strings.Builder
	for i := range 10_000 {
		fmt.Fprintf(&sb, "2026-11-01T10:%02d:%02dZ GET /p/%d 200 %dms 100\n", i/60%60, i%60, i%50, i%900)
	}
	input := sb.String()
	b.SetBytes(int64(len(input)))
	for b.Loop() {
		if err := run(context.Background(), []string{"-json"}, strings.NewReader(input), new(bytes.Buffer), new(bytes.Buffer)); err != nil {
			b.Fatal(err)
		}
	}
}
```

A real run:

```
$ printf '...four valid lines and one "garbage" line...' | ./logscan
time=2026-11-15T01:15:20.651Z level=WARN msg="skipping invalid line" file=stdin line=4 err="want 6 fields, got 1"
requests             4
invalid lines        1
bytes sent           1088
time range           2026-11-01T10:00:00Z – 2026-11-01T10:00:04Z
status 2xx           3
status 5xx           1
slow requests        2
latency p50/p95/p99  120ms / 2s / 2s
top paths
  /api/users         3
  /api/orders        1
```

The benchmark reports about **51 MB/s** on the machine used for this book. That is slow for line parsing — Chapter 10's project profiles this program and finds why (spoiler: `fmt.Sscanf` and `time.Parse` dominate).

### Review checklist

- Run it on a generated 1 GB file and watch memory (`/usr/bin/time -v`): does it stay flat?
- Does a `.gz` file give exactly the same report as the uncompressed file?
- Are the first, middle, and last percentile buckets correct? (Test with 100 requests of 1..100 ms: p50 must be 50ms, p95 95ms.)
- Is any invalid line silently dropped without being counted?
- Does Ctrl+C on a large file print a partial report within a second?
- Is the JSON schema documented (field names are an API, Unit 8.6)?

### Extensions

1. Accept JSON Lines input (`{"time":...,"path":...}`) with `-format jsonl`, using `json.Decoder` in streaming mode.
2. Add per-minute request counts (a map keyed by `t.Truncate(time.Minute)`): what is the memory bound now?
3. Read several files concurrently (after Chapter 7) and merge their `stats`.

## Answers to the Chapter 5 prediction questions

**5.1 Q2.** `hello`; `io.Copy` returns `3, nil` (EOF is not an error for `Copy`).

**5.3 Q2.** `/b` (cannot go above the root); `""`; `a/b` (`Join` does not treat a leading `/` in later elements as absolute).

**5.4 Q2.** `{"s":null}`; `{}` — `omitempty` omits an empty slice; `{}`.

**5.5 Q2.** 258 = 0x0102: big-endian `01 02`, little-endian `02 01`. `AppendUvarint(nil, 127)` is 1 byte; 128 needs 2 bytes (7 bits per byte).

**5.6 Q2.** (a) 2026-03-03 (b) the literal text `YYYY` (c) 1h30m0s (d) 2 nanoseconds (DT-138).

**5.7 Q2.** `n = 3`; `fs.Args()` is `["file.txt", "-v"]`; `-v` is not parsed because flag parsing stops at the first non-flag argument.

# Chapter 6 — Data Structures, Algorithms & Complexity

*Goal: the computer science behind fast software — and behind the routing capstone.*

You have been using data structures since Chapter 2: slices are dynamic arrays, maps are hash tables, and `slices.Sort` is a sophisticated sorting algorithm. This chapter opens the box. You will learn to describe the cost of code precisely (Big-O), implement the classic structures and algorithms in idiomatic Go, and — just as important — learn when the theory's predictions are overturned by the machine (Unit 0.5's caches).

Everything here leads to Chapters 13 and 14: a routing engine is a graph, a priority queue, a few clever preprocessing tricks, and a lot of attention to memory layout.

**What you will be able to do after this chapter**

- Analyse time and space complexity, including amortised cost.
- Implement and choose between queues, stacks, heaps, hash tables, trees, tries, and union-find.
- Write correct binary searches (a famously error-prone task) and explain sorting trade-offs.
- Model problems as graphs and solve them with BFS, DFS, topological sort, and Dijkstra.
- Recognise dynamic programming, two-pointer, and sliding-window problems.
- Prove with benchmarks that memory layout can matter more than Big-O.

## Topic 6A — Tools for analysis

## Unit 6.1 — Big-O

**One-line summary:** Big-O describes how an algorithm's cost grows as the input grows, ignoring constant factors. O(1) does not grow, O(log n) grows very slowly, O(n) grows in proportion, O(n log n) is the cost of good sorting, O(n²) becomes painful quickly. Amortised analysis explains why `append` is O(1) on average even though it sometimes copies everything.

**Mental model — searching for a name.** Looking up page 512 of a book by its page number: O(1). Finding a name in a phone book by opening the middle and halving: O(log n). Reading every page to find all mentions of "Pune": O(n). Comparing every page with every other page to find duplicates: O(n²).

### The common classes

| Class | Name | n = 1,000 | n = 1,000,000 | Typical example |
| --- | --- | --- | --- | --- |
| O(1) | constant | 1 | 1 | slice index, map lookup (average) |
| O(log n) | logarithmic | ~10 | ~20 | binary search, balanced tree lookup, heap push |
| O(n) | linear | 1,000 | 1,000,000 | scanning a slice, `strings.Contains` |
| O(n log n) | linearithmic | ~10,000 | ~20,000,000 | `slices.Sort`, merge sort |
| O(n²) | quadratic | 1,000,000 | 10¹² — hours | nested loops over the same data, insertion sort |
| O(2ⁿ) | exponential | — | — | trying every subset |

At a billion simple operations per second, O(n²) for n = 1,000,000 takes about 17 minutes; O(n log n) takes 20 milliseconds. Choosing the right class beats any micro-optimisation.

### Rules for working it out

- **Count the work in terms of the input size.** One loop over n items: O(n). A loop inside a loop over the same items: O(n²). A loop that halves the range each time: O(log n).
- **Drop constants and smaller terms.** 3n + 50 is O(n); n² + n is O(n²).
- **Different inputs get different letters.** `countLoans` called for each book in Chapter 3's `List` is O(books × loans) — written O(b·l), not O(n²) (DT-186).
- **Hidden costs count.** `strings.Contains` inside a loop is a loop. `s += x` in a loop is O(n²) bytes copied (Unit 2.6). `slices.Contains` on a slice in a loop over the same slice is O(n²); a map makes it O(n).
- **Space complexity** is analysed the same way: how much extra memory as n grows. `logscan` was designed to be O(1) in lines and O(p) in distinct paths.

### Best, worst, average, amortised

- **Worst case:** the guarantee. Map lookups are O(n) in the worst case (every key collides) but O(1) on average with a good hash.
- **Amortised:** the average over a *sequence* of operations, guaranteed. `append` copies all n elements when it grows — O(n) for that one call — but because capacity doubles (Unit 2.3), the total copying for n appends is at most n + n/2 + n/4 + … < 2n. So the cost *per append*, averaged over the sequence, is O(1).

### Big-O is not the whole story

Big-O ignores constants, and constants matter: an O(n) scan of a contiguous slice can beat an O(log n) walk of a pointer-based tree for thousands of elements, because the slice scan streams through cache lines while the tree misses cache on every node (Unit 6.13 measures it). Use Big-O to avoid disasters; use benchmarks to choose between reasonable options.

### Deep & Tricky

**DT-209. Amortised O(1) still has O(n) spikes.** *(Consequence of amortisation)* An `append` that triggers growth of a 100-million-element slice copies 800 MB at once — a latency spike in a server even though the average is tiny. Pre-sizing (P-17) or chunked structures avoid spikes when latency matters.

**DT-210. Hash maps are O(1) only on average.** *(Algorithmic fact)* A hash function that maps many keys to the same bucket makes lookups linear. Go seeds its map hash randomly per map so attackers cannot construct colliding keys (hash-flooding attacks, which took down web servers in other languages in 2011).

### Mastery check

1. **Explain.** Why is `append` O(1) amortised? Use the doubling argument.
2. **Predict.** The Big-O of: (a) checking whether a slice has duplicates with two nested loops (b) the same with a map (c) sorting then comparing neighbours (d) `strings.Join` of n parts (e) `List` in Chapter 3.
3. **Write.** Rewrite Chapter 3's `List` to be O(b + l) and confirm with the Chapter 4 benchmark.

## Unit 6.2 — Recursion

**One-line summary:** A recursive function solves a problem by calling itself on smaller instances until a base case. It is natural for trees, divide-and-conquer, and backtracking. In Go each call costs a stack frame and there is no tail-call optimisation, so very deep recursion should become a loop with an explicit stack.

### Thinking recursively

Three questions: What is the **base case** (smallest problem with a direct answer)? How do I make the problem **smaller**? How do I **combine** the smaller answer into mine?

```go
// sum of a slice: base case empty; smaller = the rest; combine = first + sum(rest)
func sum(s []int) int {
	if len(s) == 0 {
		return 0
	}
	return s[0] + sum(s[1:])
}
```

### Memoisation: remembering answers

Naive recursive Fibonacci recomputes the same values exponentially many times (fib(40) makes about 300 million calls). Store answers:

```go
func fibMemo(n int, memo map[int]int) int {
	if n < 2 {
		return n
	}
	if v, ok := memo[n]; ok {
		return v
	}
	v := fibMemo(n-1, memo) + fibMemo(n-2, memo)
	memo[n] = v
	return v
}
```

Now each value is computed once: O(n). This is the doorway to dynamic programming (Unit 6.12).

### Recursion vs iteration in Go

- Each call adds a frame (Unit 0.6). Go's goroutine stacks grow by copying (DT-35) up to 1 GB, so recursion depth of tens of thousands is fine; millions are not (DT-39, DT-76).
- Recursion over a *balanced* tree is depth O(log n) — always safe. Recursion over a linked list, a degenerate tree, or a path in a large graph can be depth O(n) — convert to iteration with an explicit stack (a slice):

```go
// depth-first traversal without recursion
stack := []*node{root}
for len(stack) > 0 {
	n := stack[len(stack)-1]
	stack = stack[:len(stack)-1]
	visit(n)
	if n.right != nil {
		stack = append(stack, n.right)
	}
	if n.left != nil {
		stack = append(stack, n.left)
	}
}
```

### Deep & Tricky

**DT-211. Recursive closures must be declared before they are assigned.** *(Scope rule, Unit 1.9)* `walk := func(n *node) { walk(n.left) }` does not compile: `walk` is not in scope inside its own initialiser (DT-83). Declare first: `var walk func(*node)`, then `walk = func(n *node) { ... walk(n.left) ... }`. You will see this pattern in the trie and BST code of this chapter.

### Mastery check

1. **Explain.** Why is naive recursive Fibonacci exponential, and how does memoisation make it linear?
2. **Write.** `permutations(s []int) [][]int` by recursion (backtracking); then count how many results for n = 8 (should be 40,320).
3. **Write.** Convert a recursive in-order traversal of a binary tree to an iterative one with an explicit stack.

## Topic 6B — Linear structures and hashing

## Unit 6.3 — Lists, stacks, queues

**One-line summary:** A **stack** (LIFO) and a **queue** (FIFO) are access disciplines; in Go both are usually built on slices. A **ring buffer** makes a queue O(1) at both ends with no wasted memory. **Linked lists** have O(1) insertion in the middle but poor cache behaviour, so they are rarely the right choice in Go.

### Stack on a slice

```go
var stack []int
stack = append(stack, 1, 2, 3) // push
top := stack[len(stack)-1]     // peek: 3
stack = stack[:len(stack)-1]   // pop
```

Uses: undo history, expression evaluation, DFS (Unit 6.10), matching brackets. Unit 0.6's call stack is the same idea in hardware.

### Queue: why `s = s[1:]` is not enough

Popping with `q = q[1:]` is O(1), but the front of the backing array is never reused: in a long-running queue, `append` keeps allocating new arrays while old space is wasted. A **ring buffer** reuses slots by wrapping indexes around with `%`:

```go
// Queue is a FIFO queue backed by a growable ring buffer.
// The zero value is an empty queue ready to use.
type Queue[T any] struct {
	buf        []T
	head, size int
}

func (q *Queue[T]) Push(v T) {
	if q.size == len(q.buf) {
		q.grow()
	}
	q.buf[(q.head+q.size)%len(q.buf)] = v
	q.size++
}

func (q *Queue[T]) Pop() (v T, ok bool) {
	if q.size == 0 {
		return v, false
	}
	v = q.buf[q.head]
	var zero T
	q.buf[q.head] = zero // do not keep a reference for the GC
	q.head = (q.head + 1) % len(q.buf)
	q.size--
	return v, true
}

func (q *Queue[T]) grow() {
	n := max(4, 2*len(q.buf))
	nb := make([]T, n)
	for i := range q.size {
		nb[i] = q.buf[(q.head+i)%len(q.buf)]
	}
	q.buf, q.head = nb, 0
}
```

A fixed-capacity ring buffer that *overwrites* the oldest element is the classic structure for "last n log lines" (the `tail` exercise in Unit 5.2) and for buffers between producers and consumers. A **deque** (double-ended queue) adds push and pop at the front with the same technique.

### Linked lists

A linked list stores each element in its own node with a pointer to the next (and previous) node. Insert and delete at a known node are O(1); finding the k-th element is O(k). `container/list` provides a doubly linked list.

In practice, Go programmers rarely use linked lists, because every node is a separate heap allocation scattered in memory: walking a list of a million nodes allocated in random order was **114 times slower** than summing a slice in this book's benchmark (Unit 6.13). They earn their place inside other structures where O(1) *unlinking of a known node* is needed — the LRU cache of Unit 6.13 is the classic case.

### Deep & Tricky

**DT-212. A slice-based queue that pops with `q = q[1:]` can leak memory.** *(Consequence of Unit 2.4)* Popped elements stay in the backing array until it is reallocated; for pointer elements the GC cannot free their targets. The ring buffer above zeroes each slot on pop for the same reason.

### Mastery check

1. **Explain.** Why does a ring buffer need `%` (or a mask with power-of-two sizes)? Trace head and size through five pushes and three pops on capacity 4.
2. **Write.** A fixed-size `Ring[T]` that overwrites the oldest element when full, with `Push`, `Len`, and `All() iter.Seq[T]` (oldest first). Use it to implement `tail -n`.

## Unit 6.4 — Hash tables from scratch

**One-line summary:** A hash table turns a key into an array index with a hash function, handles collisions (two keys, one slot) by chaining or open addressing, and grows when the **load factor** gets too high, keeping operations O(1) on average.

### The pieces

1. **Hash function:** turns a key into a number, spreading similar keys far apart. Go provides `hash/maphash` (seeded, fast) for your own tables.
2. **Index:** `hash % len(buckets)` (or `hash & (len-1)` when the length is a power of two).
3. **Collision handling:**
   - *Chaining:* each bucket holds a small list of entries.
   - *Open addressing:* on collision, probe other slots in a fixed order (linear probing: the next slot). Better cache behaviour; deletions need care ("tombstones").
4. **Load factor** = entries / slots. Above a threshold (often 0.75–0.875), allocate a table twice as big and re-insert everything. Doubling makes growth amortised O(1), exactly like `append`.

### A minimal chained hash map

```go
type entry[V any] struct {
	key string
	val V
}

type HashMap[V any] struct {
	seed    maphash.Seed
	buckets [][]entry[V]
	n       int
}

func NewHashMap[V any]() *HashMap[V] {
	return &HashMap[V]{seed: maphash.MakeSeed(), buckets: make([][]entry[V], 8)}
}

func (m *HashMap[V]) index(k string) int {
	return int(maphash.String(m.seed, k) % uint64(len(m.buckets)))
}

func (m *HashMap[V]) Put(k string, v V) {
	i := m.index(k)
	for j := range m.buckets[i] {
		if m.buckets[i][j].key == k {
			m.buckets[i][j].val = v
			return
		}
	}
	m.buckets[i] = append(m.buckets[i], entry[V]{k, v})
	m.n++
	if m.n > len(m.buckets)*3/4 { // load factor 0.75
		m.resize(2 * len(m.buckets))
	}
}

func (m *HashMap[V]) Get(k string) (V, bool) {
	for _, e := range m.buckets[m.index(k)] {
		if e.key == k {
			return e.val, true
		}
	}
	var zero V
	return zero, false
}

func (m *HashMap[V]) resize(n int) {
	old := m.buckets
	m.buckets = make([][]entry[V], n)
	for _, b := range old {
		for _, e := range b {
			i := m.index(e.key)
			m.buckets[i] = append(m.buckets[i], e)
		}
	}
}
```

Go's built-in map is far more sophisticated (Swiss tables with groups of 8 slots and SIMD-style control bytes, DT-119; incremental growth), but the ideas are these. Building your own is an exercise in understanding; in production, use the built-in map unless measurements show a specialised table is needed.

### Deep & Tricky

**DT-213. A good hash for a composite key combines all its parts.** *(Algorithmic practice)* Hashing only the first field of a struct key puts all keys with equal first fields into one bucket. `maphash.Comparable(seed, key)` (Go 1.24) hashes any comparable value correctly; Go 1.27's `maphash.Hasher` interface and `ComparableHasher` support custom hash-based containers.

### Mastery check

1. **Explain.** What is the load factor and why must the table grow when it gets high? Why does doubling keep `Put` amortised O(1)?
2. **Write.** Add `Delete` to the chained map, plus a test that inserts 100,000 random keys, deletes half, and checks the rest. Count the longest bucket chain; how does it compare with the average?

## Topic 6C — Searching and sorting

## Unit 6.5 — Binary search and its variants

**One-line summary:** Binary search finds an element in a sorted slice in O(log n) by halving the search range. Write it with a clear **invariant** to avoid the classic off-by-one bugs, and generalise it to "find the first position where a condition becomes true" — which solves many problems that do not look like searching.

### The invariant-based version

```go
// lowerBound returns the first index i with s[i] >= x (len(s) if none).
func lowerBound(s []int, x int) int {
	lo, hi := 0, len(s) // invariant: s[:lo] < x, s[hi:] >= x
	for lo < hi {
		mid := int(uint(lo+hi) >> 1) // avoids overflow
		if s[mid] < x {
			lo = mid + 1
		} else {
			hi = mid
		}
	}
	return lo
}
```

With `s = [1 3 3 3 7 9]`: `lowerBound(s, 3)` is 1 (first 3), `lowerBound(s, 4)` is 4 (where 4 would be inserted), `lowerBound(s, 10)` is 6 (`len(s)`). "Is x present?" is `i < len(s) && s[i] == x`. "Last element ≤ x" and "count of x" follow from lower bounds of x and x+1.

The standard library does this for you: `slices.BinarySearch(s, x)` returns `(index, found)`, `slices.BinarySearchFunc` takes a comparison, and `sort.Search(n, func(i int) bool)` finds the first i where a predicate is true.

### Binary search on the answer

The real power: any monotone condition (false…false, true…true) can be searched.

```go
// firstTrue finds the smallest n in [lo, hi) for which ok(n) is true,
// given ok is false...false true...true. Returns hi if never true.
func firstTrue(lo, hi int, ok func(int) bool) int {
	for lo < hi {
		mid := int(uint(lo+hi) >> 1)
		if ok(mid) {
			hi = mid
		} else {
			lo = mid + 1
		}
	}
	return lo
}

firstTrue(0, 1000, func(n int) bool { return n*n >= 700 }) // 27: smallest n with n² ≥ 700
```

Examples: the smallest capacity that ships all parcels within D days; the first commit where a test fails (`git bisect` is binary search); the smallest timeout that makes 99% of requests succeed.

### Deep & Tricky

**DT-214. `(lo + hi) / 2` can overflow.** *(Classic bug)* For huge indexes, `lo + hi` exceeds the maximum int and becomes negative (Unit 0.4). This bug existed in Java's standard library for nine years. `int(uint(lo+hi) >> 1)` (what Go's `sort.Search` uses) or `lo + (hi-lo)/2` is safe. On 64-bit Go with slice indexes it is mostly theoretical, but in binary search over a *numeric range* of int64 values it is real.

### Mastery check

1. **Explain.** State the invariant of `lowerBound`. Why does the loop terminate, and why is `lo` the answer?
2. **Predict.** `slices.BinarySearch([]int{1, 3, 3, 3, 7}, 3)` and `(..., 4)`.
3. **Write.** `shipCapacity(weights []int, days int) int` — the smallest truck capacity that ships parcels in order within `days` (each day loads parcels in order until the next would exceed capacity). Use `firstTrue`.

## Unit 6.6 — Sorting

**One-line summary:** Insertion sort is O(n²) but fast on tiny or nearly-sorted inputs; merge sort is O(n log n) and stable; quicksort is O(n log n) on average and in place; heap sort is O(n log n) worst case. Go's `slices.Sort` uses **pdqsort**, a hybrid that combines their strengths. Know stability and comparison-function rules.

### Insertion sort

```go
func insertionSort[T cmp.Ordered](s []T) {
	for i := 1; i < len(s); i++ {
		for j := i; j > 0 && s[j] < s[j-1]; j-- {
			s[j], s[j-1] = s[j-1], s[j]
		}
	}
}
```

O(n²) in general, O(n) on sorted input, and the fastest choice below about a dozen elements — which is why hybrid sorts finish small partitions with it.

### Merge sort

```go
func mergeSort[T cmp.Ordered](s []T) []T {
	if len(s) <= 1 {
		return s
	}
	mid := len(s) / 2
	left, right := mergeSort(s[:mid]), mergeSort(s[mid:])
	out := make([]T, 0, len(s))
	i, j := 0, 0
	for i < len(left) && j < len(right) {
		if right[j] < left[i] { // strict: equal elements keep left first -> stable
			out = append(out, right[j])
			j++
		} else {
			out = append(out, left[i])
			i++
		}
	}
	out = append(out, left[i:]...)
	return append(out, right[j:]...)
}
```

Always O(n log n), **stable**, but needs O(n) extra memory. Its merge step is also how you combine sorted files too large for memory (external sorting).

### Quicksort

```go
func quickSort[T cmp.Ordered](s []T) {
	if len(s) <= 1 {
		return
	}
	p := s[len(s)/2] // middle element as pivot
	i, j := 0, len(s)-1
	for i <= j { // Hoare-style partition
		for s[i] < p {
			i++
		}
		for s[j] > p {
			j--
		}
		if i <= j {
			s[i], s[j] = s[j], s[i]
			i++
			j--
		}
	}
	quickSort(s[:j+1])
	quickSort(s[i:])
}
```

In place and very fast on average, but O(n²) with consistently bad pivots, and not stable.

### Heap sort

Build a binary heap (Unit 6.8), then repeatedly move the maximum to the end. O(n log n) worst case, in place, not stable, and cache-unfriendly in practice.

### What Go actually uses

`slices.Sort` and `sort.Sort` use **pattern-defeating quicksort** (pdqsort): quicksort with smart pivot choice, insertion sort for small partitions, detection of already-sorted runs, and a switch to heap sort if partitioning goes badly — O(n log n) worst case and very fast on real data. `slices.SortStableFunc` uses a stable insertion/merge strategy.

### Comparison functions must be consistent

`slices.SortFunc(s, cmp)` requires a **strict weak ordering**: if `cmp(a, b) < 0` then `cmp(b, a) > 0`; transitive; equal elements compare 0. A comparison that is inconsistent (comparing floats including NaN with `<`, or a random tie-break) can produce an unsorted result. `cmp.Compare` handles NaN consistently (NaN sorts before all other floats).

### Deep & Tricky

**DT-215. Comparison-based sorting cannot beat O(n log n).** *(Theorem)* Any sort that only compares elements needs about log₂(n!) ≈ n log₂ n comparisons in the worst case. Faster sorts (counting sort, radix sort: O(n + k)) exploit the *structure* of keys — small integers, fixed-length strings — and are worth knowing for large numeric datasets.

**DT-216. Sorting floats with NaN using `<` breaks.** *(IEEE 754 + algorithm contract)* `NaN < x` and `x < NaN` are both false, so NaN "equals" everything, violating transitivity; elements around NaN may end up out of order. `slices.Sort` on `[]float64` handles NaN by placing them first; a hand-written `func(a, b float64) int` using `<` does not.

### Mastery check

1. **Explain.** What does "stable" mean, and give a real case where it matters (hint: sort orders by date, then stably by customer).
2. **Explain.** Why do hybrid sorts switch to insertion sort for small partitions?
3. **Write.** Counting sort for a slice of exam marks (0–100), stable, and benchmark it against `slices.Sort` for 10 million marks.

## Topic 6D — Trees and heaps

## Unit 6.7 — Trees

**One-line summary:** A binary search tree (BST) keeps keys ordered so lookup, insert, and delete take O(height). Unbalanced, the height can become n; **balanced** trees (AVL, red-black) keep it O(log n). **B-trees** keep many keys per node to match disk and cache blocks, which is why every database index is one.

### A generic BST

```go
type BST[K cmp.Ordered, V any] struct {
	root *bstNode[K, V]
	n    int
}

type bstNode[K cmp.Ordered, V any] struct {
	key         K
	val         V
	left, right *bstNode[K, V]
}

func (t *BST[K, V]) Put(k K, v V) {
	p := &t.root // pointer to the link we might replace
	for *p != nil {
		switch c := cmp.Compare(k, (*p).key); {
		case c < 0:
			p = &(*p).left
		case c > 0:
			p = &(*p).right
		default:
			(*p).val = v
			return
		}
	}
	*p = &bstNode[K, V]{key: k, val: v}
	t.n++
}

func (t *BST[K, V]) Get(k K) (V, bool) {
	n := t.root
	for n != nil {
		switch c := cmp.Compare(k, n.key); {
		case c < 0:
			n = n.left
		case c > 0:
			n = n.right
		default:
			return n.val, true
		}
	}
	var zero V
	return zero, false
}

// InOrder visits keys in sorted order.
func (t *BST[K, V]) InOrder(visit func(K, V)) {
	var walk func(*bstNode[K, V])
	walk = func(n *bstNode[K, V]) {
		if n == nil {
			return
		}
		walk(n.left)
		visit(n.key, n.val)
		walk(n.right)
	}
	walk(t.root)
}
```

The pointer-to-pointer trick in `Put` (`p := &t.root`) avoids special-casing the root: `p` always points at the link to replace.

### Balance

Insert 1, 2, 3, …, n into a plain BST and every node becomes a right child: a linked list of height n, so every operation is O(n). Balanced trees restructure on insert and delete:

- **AVL trees** keep the heights of every node's subtrees within 1, using rotations. Very strictly balanced: fast lookups.
- **Red-black trees** colour nodes and enforce rules that keep the longest path at most twice the shortest. Fewer rotations on updates. Used in many standard libraries (C++ `std::map`, Java `TreeMap`, the Linux kernel's scheduler).

You rarely implement these by hand; know what they guarantee, and that Go's standard library deliberately has no ordered map (use a sorted slice for small, read-mostly data, or a third-party B-tree such as `github.com/google/btree`).

### B-trees and databases

A B-tree node holds many keys (hundreds) and many children. The tree is very shallow — four levels can index billions of rows — and each node is one disk page or several cache lines, so a lookup touches only a handful of pages. PostgreSQL indexes, SQLite, MySQL InnoDB, and file systems use B-trees or B+ trees (Unit 9.2). It is the same lesson as Unit 0.5: design for the memory hierarchy.

### Deep & Tricky

**DT-217. Sorted slices beat trees for read-mostly data.** *(Performance practice)* A sorted slice with binary search is compact, cache-friendly, and has zero per-element allocation; inserts are O(n) because of shifting. For data built once and queried often (configuration, routing tables, dictionaries), it is usually faster than any tree. Trees win when inserts and deletes are frequent and interleaved with lookups.

### Mastery check

1. **Explain.** Why does inserting sorted keys into a plain BST destroy its performance?
2. **Write.** Add `Min`, `Max`, and `Floor(k)` (largest key ≤ k) to the BST, and a `Height()` method. Insert 1..1000 in order and in random order; compare heights.

## Unit 6.8 — Heaps and priority queues

**One-line summary:** A binary heap is a complete binary tree stored in a slice where every parent is smaller (min-heap) than its children. It gives O(1) access to the minimum and O(log n) push and pop — the data structure behind priority queues, Dijkstra's algorithm, schedulers, timers, and top-k problems.

### The array layout

A heap needs no pointers: for the element at index i, its children are at 2i+1 and 2i+2, and its parent at (i−1)/2. **Push** appends at the end and "sifts up" while smaller than its parent; **Pop** takes the root, moves the last element to the root, and "sifts down" while larger than its smallest child. Both walk one path of a tree of height log₂ n.

The generic priority queue from this chapter's project:

```go
// PQ is a priority queue: Pop returns the element for which less
// reports true against all others. Create it with NewPQ.
type PQ[T any] struct {
	items []T
	less  func(a, b T) bool
}

func NewPQ[T any](less func(a, b T) bool) *PQ[T] { return &PQ[T]{less: less} }

func (p *PQ[T]) Len() int { return len(p.items) }

func (p *PQ[T]) Push(v T) {
	p.items = append(p.items, v)
	p.up(len(p.items) - 1)
}

func (p *PQ[T]) Pop() (v T, ok bool) {
	n := len(p.items)
	if n == 0 {
		return v, false
	}
	v = p.items[0]
	p.items[0] = p.items[n-1]
	var zero T
	p.items[n-1] = zero
	p.items = p.items[:n-1]
	if len(p.items) > 0 {
		p.down(0)
	}
	return v, true
}

func (p *PQ[T]) up(i int) {
	for i > 0 {
		parent := (i - 1) / 2
		if !p.less(p.items[i], p.items[parent]) {
			return
		}
		p.items[i], p.items[parent] = p.items[parent], p.items[i]
		i = parent
	}
}

func (p *PQ[T]) down(i int) {
	n := len(p.items)
	for {
		smallest := i
		l, r := 2*i+1, 2*i+2
		if l < n && p.less(p.items[l], p.items[smallest]) {
			smallest = l
		}
		if r < n && p.less(p.items[r], p.items[smallest]) {
			smallest = r
		}
		if smallest == i {
			return
		}
		p.items[i], p.items[smallest] = p.items[smallest], p.items[i]
		i = smallest
	}
}
```

### `container/heap`

The standard library's heap works on any type implementing `heap.Interface` (`sort.Interface` plus `Push` and `Pop`):

```go
type Task struct {
	Name     string
	Priority int
}

type taskHeap []Task

func (h taskHeap) Len() int           { return len(h) }
func (h taskHeap) Less(i, j int) bool { return h[i].Priority > h[j].Priority } // max-heap
func (h taskHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *taskHeap) Push(x any)        { *h = append(*h, x.(Task)) }
func (h *taskHeap) Pop() any {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[:n-1]
	return x
}

func topTasks(tasks []Task) []string {
	h := &taskHeap{}
	for _, t := range tasks {
		heap.Push(h, t) // call heap.Push, NOT h.Push
	}
	var out []string
	for h.Len() > 0 {
		out = append(out, heap.Pop(h).(Task).Name)
	}
	return out
}
```

Note the confusing part: your `Push`/`Pop` methods only append and remove the *last* element; the package functions `heap.Push`/`heap.Pop` do the sifting. `heap.Fix(h, i)` restores order after changing element i's priority — needed for "decrease-key" in some graph algorithms. The generic `PQ` above is easier to use and avoids `any` boxing; `container/heap` predates generics.

### Top-k in O(n log k)

To find the 10 largest of a billion numbers, keep a **min**-heap of size 10: push each number; if the heap exceeds 10, pop the minimum. Memory O(k), time O(n log k).

### Deep & Tricky

**DT-218. Building a heap from n elements is O(n), not O(n log n).** *(Algorithmic fact)* `heap.Init` sifts down from the middle of the array to the start; most elements are near the bottom and move very little, and the total work sums to O(n). Pushing n elements one by one is O(n log n).

### Mastery check

1. **Explain.** For index 9 in a heap slice, which indexes are its parent and children?
2. **Write.** `topK(nums []int, k int) []int` using a min-heap of size k, and a test comparing it with sorting the whole slice.
3. **Write.** Merge k sorted slices into one sorted slice in O(n log k) with a heap of (value, slice index, position).

## Unit 6.9 — Tries and string algorithms basics

**One-line summary:** A trie stores strings by their characters along tree paths, so all words with a given prefix sit under one node — ideal for autocomplete, prefix routing, and dictionaries. Simple string-search ideas (naive search, the idea behind KMP and rolling hashes) round out the basics.

### A trie

```go
type Trie struct {
	children map[rune]*Trie
	end      bool // a word ends here
}

func (t *Trie) Insert(word string) {
	n := t
	for _, r := range word { // runes, so Hindi and emoji work (Unit 2.5)
		if n.children == nil {
			n.children = make(map[rune]*Trie)
		}
		next, ok := n.children[r]
		if !ok {
			next = &Trie{}
			n.children[r] = next
		}
		n = next
	}
	n.end = true
}

// WithPrefix returns all stored words that start with prefix, sorted.
func (t *Trie) WithPrefix(prefix string) []string {
	n := t
	for _, r := range prefix {
		if n = n.children[r]; n == nil {
			return nil
		}
	}
	var out []string
	var walk func(*Trie, []rune)
	walk = func(n *Trie, cur []rune) {
		if n.end {
			out = append(out, string(cur))
		}
		for r, c := range n.children {
			walk(c, append(cur, r))
		}
	}
	walk(n, []rune(prefix))
	slices.Sort(out) // map order is random (Unit 2.7)
	return out
}
```

Lookup is O(length of the word), independent of how many words are stored. The cost is memory: one map per node. Production tries compress chains of single-child nodes (radix trees), which is how HTTP routers (`net/http`'s `ServeMux` patterns, Unit 8.4) and IP routing tables match prefixes quickly.

### String search in brief

- **Naive search** compares the pattern at every position: O(n·m) worst case. Go's `strings.Index` is much smarter: it uses fast byte search (often SIMD instructions) for the first byte and a Rabin–Karp rolling hash or other techniques for longer patterns.
- **Rabin–Karp** computes a hash of each window of the text in O(1) from the previous window ("rolling hash"), comparing strings only when hashes match. Also the basis of content-defined chunking in backup tools.
- **KMP** precomputes, for each prefix of the pattern, how far to fall back after a mismatch, giving O(n + m) worst case.

Use `strings.Index`, `strings.Contains`, `bytes.Index` — they are highly optimised. Know the ideas so you recognise them in specialised problems (searching many patterns at once uses **Aho–Corasick**, a trie with failure links).

### Mastery check

1. **Explain.** Why is trie lookup independent of the number of stored words? What does it cost instead?
2. **Write.** Add `Delete(word)` that also removes now-empty nodes, and a `Count()` of stored words; test with words sharing prefixes ("go", "gopher", "golang").

## Topic 6E — Graphs

## Unit 6.10 — Graph basics

**One-line summary:** A graph is a set of nodes (vertices) connected by edges, directed or undirected, weighted or not. Represent it as an **adjacency list** (usually) or an **adjacency matrix** (dense graphs). **BFS** explores level by level (shortest paths in unweighted graphs); **DFS** goes deep first (reachability, cycles); **topological sort** orders tasks by their dependencies.

**Mental model — the city map.** Intersections are nodes, roads are edges, one-way streets are directed edges, travel times are weights. Almost every "connections" problem — road networks, social networks, package dependencies, build systems, network routing — is a graph problem.

### Representations

| | Adjacency list | Adjacency matrix |
| --- | --- | --- |
| Go form | `map[Node][]Node` or `[][]int` (nodes numbered 0..n−1) | `[][]bool` or flat `[]float64` of size n² |
| Memory | O(V + E) | O(V²) |
| "Is there an edge u→v?" | O(degree of u) | O(1) |
| "All neighbours of u" | O(degree of u) | O(V) |
| Use for | sparse graphs (roads, the web) | small or dense graphs |

Road networks are very sparse (an intersection has about 3 neighbours), so adjacency lists — eventually in compact flat-array form (Unit 13.8) — are the right choice for routing.

### Breadth-first search

BFS uses a **queue**: visit the start, then all nodes 1 edge away, then 2 edges away, … The first time BFS reaches a node is along a path with the fewest edges. The Chapter 6 project's `Grid.BFS` is BFS on a grid graph (Unit 6.3's `Queue` does the work). O(V + E).

### Depth-first search

DFS uses recursion (or an explicit stack): go as deep as possible, then backtrack.

```go
type Graph map[string][]string // adjacency list

// DFS visits every node reachable from start, in depth-first order.
func DFS(g Graph, start string, visit func(string)) {
	seen := map[string]bool{}
	var walk func(string)
	walk = func(n string) {
		if seen[n] {
			return
		}
		seen[n] = true
		visit(n)
		for _, m := range g[n] {
			walk(m)
		}
	}
	walk(start)
}
```

Uses: reachability ("can this package import that one?"), connected components, cycle detection (a node seen again *while still on the current path*), maze generation, and topological sort.

### Topological sort

Given dependencies ("pay before ship"), find an order where every task comes after the tasks it depends on. Kahn's algorithm repeatedly takes a node with no remaining incoming edges:

```go
var ErrCycle = errors.New("graph has a cycle")

// TopoSort returns an order in which every node comes after its
// dependencies (Kahn's algorithm). Edges point from a node to the nodes
// that depend on it.
func TopoSort(g Graph) ([]string, error) {
	indeg := map[string]int{}
	for n, outs := range g {
		if _, ok := indeg[n]; !ok {
			indeg[n] = 0
		}
		for _, m := range outs {
			indeg[m]++
		}
	}
	var ready []string
	for n, d := range indeg {
		if d == 0 {
			ready = append(ready, n)
		}
	}
	slices.Sort(ready) // deterministic output
	var order []string
	for len(ready) > 0 {
		n := ready[0]
		ready = ready[1:]
		order = append(order, n)
		next := slices.Clone(g[n])
		slices.Sort(next)
		for _, m := range next {
			indeg[m]--
			if indeg[m] == 0 {
				ready = append(ready, m)
			}
		}
	}
	if len(order) != len(indeg) {
		return nil, ErrCycle // some nodes never became ready: they are in a cycle
	}
	return order, nil
}

TopoSort(Graph{"shop": {"pay", "ship"}, "pay": {"ship"}, "ship": {"done"}})
// [shop pay ship done]
```

This is how `go build` orders package compilation, how `make` orders targets, and how database migrations and package initialisation (Unit 3.12) are ordered. An import cycle (forbidden in Go) is exactly a graph cycle.

### Deep & Tricky

**DT-219. Recursive DFS on large graphs can overflow the stack — or just be slow.** *(Go-specific, DT-39, DT-76)* A road graph with millions of nodes can produce DFS paths millions deep. Use an explicit stack for production graph code. Note also that `map[string]bool` for "seen" is convenient but slow at scale; numbering nodes 0..n−1 and using a `[]bool` (or a bitset) is often 10× faster (Unit 13.8).

### Mastery check

1. **Explain.** Why does BFS find the fewest-edges path, and why does it fail for weighted roads?
2. **Write.** Detect a cycle in a directed graph with DFS (three colours: unvisited, on the current path, finished) and return the cycle's nodes.
3. **Write.** Given a module's `go list -m -json all`-style dependency pairs, print a valid build order or report the cycle.

## Unit 6.11 — Shortest paths

**One-line summary:** **Dijkstra's algorithm** finds cheapest paths from a source in graphs with non-negative edge weights, using a priority queue: O((V + E) log V). **Union-find** answers "are these connected?" in nearly O(1) and powers minimum spanning trees. These are the foundation of the routing chapters.

### Dijkstra

Idea: always expand the cheapest unexplored node; with non-negative weights, its distance is then final.

1. `dist[start] = 0`; push (start, 0) into a min-priority queue.
2. Pop the node with the smallest distance. If this entry is stale (a cheaper distance was already found), skip it.
3. If it is the target, stop. Otherwise, for each neighbour, if `dist[node] + weight` improves the neighbour's distance, record it and its predecessor, and push it.
4. Rebuild the path by following predecessors back from the target.

The project's implementation on a weighted grid:

```go
func (g *Grid) Dijkstra() ([]Point, int, error) {
	type item struct {
		p    Point
		dist int
	}
	dist := map[Point]int{g.Start: 0}
	prev := make(map[Point]Point)
	pq := ds.NewPQ(func(a, b item) bool { return a.dist < b.dist })
	pq.Push(item{g.Start, 0})
	for pq.Len() > 0 {
		it, _ := pq.Pop()
		if it.dist > dist[it.p] {
			continue // stale entry: a shorter route was found after it was pushed
		}
		if it.p == g.End {
			return path(prev, g.Start, g.End), it.dist, nil
		}
		g.neighbours(it.p, func(n Point) {
			nd := it.dist + g.cost[n.R*g.cols+n.C]
			if old, ok := dist[n]; !ok || nd < old {
				dist[n] = nd
				prev[n] = it.p
				pq.Push(item{n, nd})
			}
		})
	}
	return nil, 0, ErrNoPath
}
```

"Lazy deletion" (pushing a node again instead of decreasing its key, and skipping stale entries) is simpler than `heap.Fix` and usually as fast. Negative weights break Dijkstra; Bellman–Ford handles them in O(V·E). For routing on real maps, Chapter 13 adds A* (Dijkstra guided by a distance estimate), bidirectional search, and preprocessing (contraction hierarchies) that answer queries thousands of times faster.

### Union-find (disjoint sets)

```go
// UnionFind tracks a partition of the integers 0..n-1 into sets.
type UnionFind struct {
	parent, size []int
}

func NewUnionFind(n int) *UnionFind {
	u := &UnionFind{parent: make([]int, n), size: make([]int, n)}
	for i := range n {
		u.parent[i], u.size[i] = i, 1
	}
	return u
}

// Find returns the representative of x's set, compressing the path.
func (u *UnionFind) Find(x int) int {
	for u.parent[x] != x {
		u.parent[x] = u.parent[u.parent[x]] // path halving
		x = u.parent[x]
	}
	return x
}

// Union merges the sets of a and b and reports whether they were separate.
func (u *UnionFind) Union(a, b int) bool {
	ra, rb := u.Find(a), u.Find(b)
	if ra == rb {
		return false
	}
	if u.size[ra] < u.size[rb] {
		ra, rb = rb, ra // attach the smaller tree under the larger
	}
	u.parent[rb] = ra
	u.size[ra] += u.size[rb]
	return true
}
```

With union by size and path compression, operations take amortised O(α(n)) — the inverse Ackermann function, below 5 for any input that fits in the universe. Uses: connected components of a road network (which parts of the map are reachable at all — useful before routing), Kruskal's minimum spanning tree, and clustering.

### Deep & Tricky

**DT-220. Dijkstra's correctness depends on non-negative weights — and so does your early exit.** *(Algorithmic fact)* With a negative edge, a node popped "final" can later be improved, and stopping when the target is popped returns a wrong answer. Road travel times are never negative, but "rewards" or "discounts" modelled as negative costs break the algorithm silently.

### Mastery check

1. **Explain.** Why can Dijkstra stop as soon as it pops the target? What about a target pushed but not yet popped?
2. **Write.** Dijkstra on an adjacency-list graph (`[][]Edge` with `To int; W int`) using `ds.PQ`, returning distances to all nodes; test against a brute-force Bellman–Ford on random small graphs.
3. **Write.** Count the connected components of a grid map with union-find (cells are nodes; join open neighbours).

## Topic 6F — Techniques and real-world speed

## Unit 6.12 — Problem-solving techniques

**One-line summary:** **Dynamic programming** solves problems with overlapping subproblems by building a table of answers; **two pointers** walk a sorted sequence from both ends; a **sliding window** maintains a running answer over a moving range. Recognising the pattern is most of the work.

### Dynamic programming

When a problem's answer depends on answers to smaller versions of the same problem, and those smaller problems repeat, store them. Two styles: top-down with memoisation (Unit 6.2) or bottom-up with a table.

**Fewest coins** for an amount — greedy (always take the largest coin) fails for coins {1, 3, 4} and amount 6 (greedy: 4+1+1 = 3 coins; best: 3+3 = 2 coins):

```go
// minCoins returns the fewest coins that sum to amount, or -1.
func minCoins(coins []int, amount int) int {
	const inf = 1 << 62
	best := make([]int, amount+1) // best[a] = fewest coins for a
	for a := 1; a <= amount; a++ {
		best[a] = inf
		for _, c := range coins {
			if c <= a && best[a-c] != inf {
				best[a] = min(best[a], best[a-c]+1)
			}
		}
	}
	if best[amount] == inf {
		return -1
	}
	return best[amount]
}
```

**Longest common subsequence** (the core of `diff` tools) with two rolling rows to use O(m) memory:

```go
func lcs(a, b string) int {
	prev := make([]int, len(b)+1)
	cur := make([]int, len(b)+1)
	for i := 1; i <= len(a); i++ {
		for j := 1; j <= len(b); j++ {
			if a[i-1] == b[j-1] {
				cur[j] = prev[j-1] + 1
			} else {
				cur[j] = max(prev[j], cur[j-1])
			}
		}
		prev, cur = cur, prev
	}
	return prev[len(b)]
}
```

`lcs("AGGTAB", "GXTXAYB")` is 4 ("GTAB"). The recipe: define the state (`best[a]`, `L[i][j]`), write the recurrence, choose an order so dependencies are computed first, identify the base cases.

### Two pointers

On a sorted slice, find a pair with a given sum in O(n) instead of O(n²):

```go
func pairWithSum(s []int, target int) (int, int, bool) {
	i, j := 0, len(s)-1
	for i < j {
		switch sum := s[i] + s[j]; {
		case sum == target:
			return i, j, true
		case sum < target:
			i++ // need a bigger sum
		default:
			j-- // need a smaller sum
		}
	}
	return 0, 0, false
}
```

Also: merging sorted lists, removing duplicates in place, reversing, and the in-place filter of Unit 2.4.

### Sliding window

Maintain a window `[start, i]` that satisfies a condition, extending on the right and shrinking on the left:

```go
// longestUniqueRun returns the length of the longest substring of s
// without repeated bytes.
func longestUniqueRun(s string) int {
	last := [256]int{} // last index+1 where each byte was seen
	best, start := 0, 0
	for i := 0; i < len(s); i++ {
		if last[s[i]] > start {
			start = last[s[i]] // jump past the previous occurrence
		}
		last[s[i]] = i + 1
		best = max(best, i+1-start)
	}
	return best
}
```

Real uses: rate limiters ("requests in the last 60 seconds", Unit 12.8), moving averages in monitoring, and detecting bursts in logs.

### Mastery check

1. **Explain.** Why does greedy coin change fail for {1, 3, 4}, and why does the DP succeed?
2. **Write.** `maxSumWindow(s []int, k int) int` (maximum sum of any k consecutive elements) in O(n).
3. **Write.** Edit distance (insert, delete, replace) between two strings with DP; then make it work on runes, not bytes, and test with Hindi words.

## Unit 6.13 — Caches and memory-friendly design

**One-line summary:** An LRU cache combines a hash map and a doubly linked list for O(1) lookups and evictions. And memory layout — contiguous slices, row-major traversal, values instead of pointers — can change real speed by an order of magnitude or more without changing Big-O. Measure it.

### LRU cache

"Least recently used" eviction keeps the most useful items when space runs out (Unit 9.7 uses it in front of a database):

```go
type LRU[K comparable, V any] struct {
	cap        int
	items      map[K]*lruNode[K, V]
	head, tail *lruNode[K, V] // head = most recent
}

type lruNode[K comparable, V any] struct {
	key        K
	val        V
	prev, next *lruNode[K, V]
}

func (c *LRU[K, V]) Get(k K) (v V, ok bool) {
	n, ok := c.items[k]
	if !ok {
		return v, false
	}
	c.moveToFront(n)
	return n.val, true
}

func (c *LRU[K, V]) Put(k K, v V) {
	if n, ok := c.items[k]; ok {
		n.val = v
		c.moveToFront(n)
		return
	}
	if len(c.items) == c.cap {
		old := c.tail // evict the least recently used
		c.unlink(old)
		delete(c.items, old.key)
	}
	n := &lruNode[K, V]{key: k, val: v}
	c.items[k] = n
	c.pushFront(n)
}
```

The map finds a node in O(1); the list moves it to the front or removes the tail in O(1) because each node knows its neighbours. The full code (with `unlink`, `pushFront`, `moveToFront`) is in the project. This is the textbook case where a linked list earns its place. (This version is not safe for concurrent use; Chapter 7 adds a mutex.)

### The measurements

Benchmarks run for this book (Go 1.24, x86-64 server, 2²⁰ = 1,048,576 integers):

| Operation | Time | Relative |
| --- | --- | --- |
| Sum a `[]int` | 0.43 ms | 1× |
| Sum a `container/list` built in order | 3.55 ms | 8× |
| Sum a hand-made linked list whose nodes were shuffled in memory | 48.6 ms | **114×** |
| Sum a 2048×2048 `[]int32` grid row by row | 2.07 ms | 1× |
| Same grid, column by column | 27.7 ms | **13×** |

All of these are O(n). The differences come entirely from the memory hierarchy (Unit 0.5): the slice and the row-major walk read memory sequentially, so every cache line brings 8 or 16 useful values and the hardware prefetcher streams ahead; the shuffled list and the column walk jump around, so almost every access waits for RAM.

### Design rules that follow

- Prefer slices of values (`[]Point`) over slices of pointers (`[]*Point`) for data you scan (Unit 2.12).
- Traverse multi-dimensional data in the order it is laid out (row-major for `r*cols + c`).
- Number your nodes 0..n−1 and use slices instead of maps keyed by structs or strings in hot graph code (Unit 13.8).
- Keep hot fields together and cold fields elsewhere ("struct of arrays" for very large datasets: `xs []float64; ys []float64` instead of `[]struct{ x, y float64; name string; ... }` when you only scan x and y).
- And always: **benchmark** (Unit 4.5) before and after.

### Deep & Tricky

**DT-221. The same algorithm can be 100× apart depending on allocation order.** *(Measured above)* The in-order `container/list` was 14× faster than the shuffled list: its nodes happened to be allocated next to each other. Benchmarks of pointer-based structures built in a tidy order can wildly overstate their real-world performance, where nodes are allocated at different times and scattered across the heap.

### Pattern

**P-31. Pick data structures by access pattern, then verify by measurement.** *(Engineering practice)*

- *Problem it solves:* choosing structures by Big-O alone, then discovering they are slow in practice, or by habit ("a tree for sorted data") when a sorted slice is faster.
- *How:* write down the operations and their frequencies (reads vs writes, point lookups vs scans, sizes); choose the simplest structure that serves them; benchmark with realistic sizes and allocation patterns.
- *When not to:* for small data, choose the clearest code; performance work belongs where profiles point (Chapter 10).
- *Trade-off:* measurement takes time, but guessing wrong in a hot path costs far more.

### Mastery check

1. **Explain.** Why does an LRU cache need both a map and a doubly linked list? What would each alone cost?
2. **Explain.** Using cache lines, explain the 13× difference between row-major and column-major traversal.
3. **Write.** A benchmark comparing `[]struct{ X, Y float64; Name string; Tags []string }` with two separate `[]float64` slices for computing the average X. Predict the ratio first.

## Chapter 6 Project — A generic data-structures library and a city-grid route finder

**Goal:** build `ds`, a small generic library (ring-buffer queue, priority queue, LRU cache, union-find) with tests and benchmarks, then use it in `grid`, a route finder for a city map with walls and slow roads, comparing BFS (fewest steps) with Dijkstra (lowest cost).

### Map format

```
S.....#...
.####.#.#.
.#..9.9.#.
.#.##.#.#.
...#....#E
```

`#` wall, `.` cost 1 to enter, `1`–`9` slow road costing that much to enter, `S` start, `E` end (cost 1). Moves: up, down, left, right.

### Requirements

1. `ds.Queue[T]`: zero value usable; ring buffer; Pop on empty reports `ok == false`; popped slots zeroed.
2. `ds.PQ[T]` with a `less` function; tested against `slices.Sort` on random input with duplicates (a *differential* test).
3. `ds.LRU[K, V]` with O(1) `Get`/`Put`, eviction of the least recently used, update without eviction.
4. `ds.UnionFind` with path halving and union by size.
5. `grid.Parse` validates the map (rectangular, known characters, exactly one S and E) and stores costs in a flat slice (Unit 2.4).
6. `Grid.BFS` and `Grid.Dijkstra` return paths; `ErrNoPath` when unreachable.
7. `cmd/gridpath` reads a map from stdin and draws both paths.

### Reference solution — `ds/ds.go`

```go
// Package ds provides small generic data structures: a ring-buffer
// queue, a binary-heap priority queue, an LRU cache, and union-find.
package ds

// Queue is a FIFO queue backed by a growable ring buffer.
// The zero value is an empty queue ready to use.
type Queue[T any] struct {
	buf        []T
	head, size int
}

func (q *Queue[T]) Len() int { return q.size }

func (q *Queue[T]) Push(v T) {
	if q.size == len(q.buf) {
		q.grow()
	}
	q.buf[(q.head+q.size)%len(q.buf)] = v
	q.size++
}

// Pop removes and returns the oldest element. ok is false if q is empty.
func (q *Queue[T]) Pop() (v T, ok bool) {
	if q.size == 0 {
		return v, false
	}
	v = q.buf[q.head]
	var zero T
	q.buf[q.head] = zero // do not keep a reference for the GC
	q.head = (q.head + 1) % len(q.buf)
	q.size--
	return v, true
}

func (q *Queue[T]) grow() {
	n := max(4, 2*len(q.buf))
	nb := make([]T, n)
	for i := range q.size {
		nb[i] = q.buf[(q.head+i)%len(q.buf)]
	}
	q.buf, q.head = nb, 0
}

// PQ is a priority queue: Pop returns the element for which less
// reports true against all others. Create it with NewPQ.
type PQ[T any] struct {
	items []T
	less  func(a, b T) bool
}

func NewPQ[T any](less func(a, b T) bool) *PQ[T] { return &PQ[T]{less: less} }

func (p *PQ[T]) Len() int { return len(p.items) }

func (p *PQ[T]) Push(v T) {
	p.items = append(p.items, v)
	p.up(len(p.items) - 1)
}

func (p *PQ[T]) Pop() (v T, ok bool) {
	n := len(p.items)
	if n == 0 {
		return v, false
	}
	v = p.items[0]
	p.items[0] = p.items[n-1]
	var zero T
	p.items[n-1] = zero
	p.items = p.items[:n-1]
	if len(p.items) > 0 {
		p.down(0)
	}
	return v, true
}

func (p *PQ[T]) up(i int) {
	for i > 0 {
		parent := (i - 1) / 2
		if !p.less(p.items[i], p.items[parent]) {
			return
		}
		p.items[i], p.items[parent] = p.items[parent], p.items[i]
		i = parent
	}
}

func (p *PQ[T]) down(i int) {
	n := len(p.items)
	for {
		smallest := i
		l, r := 2*i+1, 2*i+2
		if l < n && p.less(p.items[l], p.items[smallest]) {
			smallest = l
		}
		if r < n && p.less(p.items[r], p.items[smallest]) {
			smallest = r
		}
		if smallest == i {
			return
		}
		p.items[i], p.items[smallest] = p.items[smallest], p.items[i]
		i = smallest
	}
}

// LRU is a fixed-capacity cache that evicts the least recently used key.
// Get and Put are O(1): a map finds entries, a doubly linked list keeps
// them in recency order.
type LRU[K comparable, V any] struct {
	cap        int
	items      map[K]*lruNode[K, V]
	head, tail *lruNode[K, V] // head = most recent
}

type lruNode[K comparable, V any] struct {
	key        K
	val        V
	prev, next *lruNode[K, V]
}

func NewLRU[K comparable, V any](capacity int) *LRU[K, V] {
	if capacity < 1 {
		panic("ds: LRU capacity must be at least 1")
	}
	return &LRU[K, V]{cap: capacity, items: make(map[K]*lruNode[K, V], capacity)}
}

func (c *LRU[K, V]) Len() int { return len(c.items) }

func (c *LRU[K, V]) Get(k K) (v V, ok bool) {
	n, ok := c.items[k]
	if !ok {
		return v, false
	}
	c.moveToFront(n)
	return n.val, true
}

func (c *LRU[K, V]) Put(k K, v V) {
	if n, ok := c.items[k]; ok {
		n.val = v
		c.moveToFront(n)
		return
	}
	if len(c.items) == c.cap {
		old := c.tail
		c.unlink(old)
		delete(c.items, old.key)
	}
	n := &lruNode[K, V]{key: k, val: v}
	c.items[k] = n
	c.pushFront(n)
}

func (c *LRU[K, V]) unlink(n *lruNode[K, V]) {
	if n.prev != nil {
		n.prev.next = n.next
	} else {
		c.head = n.next
	}
	if n.next != nil {
		n.next.prev = n.prev
	} else {
		c.tail = n.prev
	}
	n.prev, n.next = nil, nil
}

func (c *LRU[K, V]) pushFront(n *lruNode[K, V]) {
	n.next = c.head
	if c.head != nil {
		c.head.prev = n
	}
	c.head = n
	if c.tail == nil {
		c.tail = n
	}
}

func (c *LRU[K, V]) moveToFront(n *lruNode[K, V]) {
	if c.head == n {
		return
	}
	c.unlink(n)
	c.pushFront(n)
}

// UnionFind tracks a partition of the integers 0..n-1 into sets.
type UnionFind struct {
	parent, size []int
}

func NewUnionFind(n int) *UnionFind {
	u := &UnionFind{parent: make([]int, n), size: make([]int, n)}
	for i := range n {
		u.parent[i], u.size[i] = i, 1
	}
	return u
}

// Find returns the representative of x's set, compressing the path.
func (u *UnionFind) Find(x int) int {
	for u.parent[x] != x {
		u.parent[x] = u.parent[u.parent[x]] // path halving
		x = u.parent[x]
	}
	return x
}

// Union merges the sets of a and b and reports whether they were separate.
func (u *UnionFind) Union(a, b int) bool {
	ra, rb := u.Find(a), u.Find(b)
	if ra == rb {
		return false
	}
	if u.size[ra] < u.size[rb] {
		ra, rb = rb, ra
	}
	u.parent[rb] = ra
	u.size[ra] += u.size[rb]
	return true
}
```

### Reference solution — `ds/ds_test.go`

```go
package ds

import (
	"math/rand/v2"
	"slices"
	"testing"
)

func TestQueue(t *testing.T) {
	var q Queue[int]
	for i := range 10 {
		q.Push(i)
		if i%3 == 0 {
			q.Pop() // wrap around inside the ring
		}
	}
	var got []int
	for q.Len() > 0 {
		v, _ := q.Pop()
		got = append(got, v)
	}
	if want := []int{4, 5, 6, 7, 8, 9}; !slices.Equal(got, want) {
		t.Errorf("got %v, want %v", got, want)
	}
	if _, ok := q.Pop(); ok {
		t.Error("Pop on empty queue reported ok")
	}
}

func TestPQSortsLikeSlicesSort(t *testing.T) {
	r := rand.New(rand.NewPCG(1, 2))
	for trial := range 50 {
		n := r.IntN(200)
		in := make([]int, n)
		pq := NewPQ(func(a, b int) bool { return a < b })
		for i := range in {
			in[i] = r.IntN(50) // duplicates on purpose
			pq.Push(in[i])
		}
		var out []int
		for pq.Len() > 0 {
			v, _ := pq.Pop()
			out = append(out, v)
		}
		slices.Sort(in)
		if !slices.Equal(in, out) {
			t.Fatalf("trial %d: heap order %v, want %v", trial, out, in)
		}
	}
}

func TestLRU(t *testing.T) {
	c := NewLRU[string, int](2)
	c.Put("a", 1)
	c.Put("b", 2)
	c.Get("a")    // a is now most recent
	c.Put("c", 3) // evicts b
	if _, ok := c.Get("b"); ok {
		t.Error("b should have been evicted")
	}
	if v, ok := c.Get("a"); !ok || v != 1 {
		t.Errorf("Get(a) = %v, %v", v, ok)
	}
	c.Put("a", 10) // update, no eviction
	if c.Len() != 2 {
		t.Errorf("Len = %d", c.Len())
	}
	if v, _ := c.Get("a"); v != 10 {
		t.Errorf("updated a = %d", v)
	}
}

func TestUnionFind(t *testing.T) {
	u := NewUnionFind(6)
	u.Union(0, 1)
	u.Union(1, 2)
	u.Union(3, 4)
	if u.Find(0) != u.Find(2) || u.Find(0) == u.Find(3) || u.Union(2, 0) {
		t.Error("wrong partition")
	}
}

func BenchmarkPQ(b *testing.B) {
	r := rand.New(rand.NewPCG(1, 2))
	for b.Loop() {
		pq := NewPQ(func(a, b int) bool { return a < b })
		for range 10_000 {
			pq.Push(r.IntN(1_000_000))
		}
		for pq.Len() > 0 {
			pq.Pop()
		}
	}
}
```

### Reference solution — `grid/grid.go`

```go
// Package grid finds shortest paths on a rectangular city map.
//
// The map is text: '#' is a wall, '.' costs 1 to enter, digits '1'..'9'
// cost that much to enter (slow roads), 'S' is the start and 'E' the end
// (both cost 1). Moves go up, down, left, and right.
package grid

import (
	"errors"
	"fmt"
	"strings"

	"example.com/dsgrid/ds"
)

type Point struct{ R, C int }

type Grid struct {
	rows, cols int
	cost       []int // flat row-major: cost[r*cols+c]; 0 means wall
	Start, End Point
}

var ErrNoPath = errors.New("no path")

// Parse reads a map. Every row must have the same length.
func Parse(text string) (*Grid, error) {
	lines := strings.Fields(text)
	if len(lines) == 0 {
		return nil, errors.New("empty map")
	}
	g := &Grid{rows: len(lines), cols: len(lines[0])}
	g.cost = make([]int, g.rows*g.cols)
	foundS, foundE := false, false
	for r, line := range lines {
		if len(line) != g.cols {
			return nil, fmt.Errorf("row %d has %d cells, want %d", r, len(line), g.cols)
		}
		for c := range len(line) {
			ch := line[c]
			i := r*g.cols + c
			switch {
			case ch == '#':
				g.cost[i] = 0
			case ch == '.':
				g.cost[i] = 1
			case ch >= '1' && ch <= '9':
				g.cost[i] = int(ch - '0')
			case ch == 'S':
				g.cost[i], g.Start, foundS = 1, Point{r, c}, true
			case ch == 'E':
				g.cost[i], g.End, foundE = 1, Point{r, c}, true
			default:
				return nil, fmt.Errorf("row %d col %d: unknown cell %q", r, c, ch)
			}
		}
	}
	if !foundS || !foundE {
		return nil, errors.New("map needs one S and one E")
	}
	return g, nil
}

func (g *Grid) neighbours(p Point, yield func(Point)) {
	for _, d := range [4]Point{{-1, 0}, {1, 0}, {0, -1}, {0, 1}} {
		q := Point{p.R + d.R, p.C + d.C}
		if q.R >= 0 && q.R < g.rows && q.C >= 0 && q.C < g.cols && g.cost[q.R*g.cols+q.C] > 0 {
			yield(q)
		}
	}
}

// BFS returns a path with the fewest steps, ignoring costs.
func (g *Grid) BFS() ([]Point, error) {
	prev := make(map[Point]Point)
	seen := map[Point]bool{g.Start: true}
	var q ds.Queue[Point]
	q.Push(g.Start)
	for q.Len() > 0 {
		p, _ := q.Pop()
		if p == g.End {
			return path(prev, g.Start, g.End), nil
		}
		g.neighbours(p, func(n Point) {
			if !seen[n] {
				seen[n] = true
				prev[n] = p
				q.Push(n)
			}
		})
	}
	return nil, ErrNoPath
}

// Dijkstra returns a cheapest path and its total cost (the cost of every
// cell entered after the start).
func (g *Grid) Dijkstra() ([]Point, int, error) {
	type item struct {
		p    Point
		dist int
	}
	dist := map[Point]int{g.Start: 0}
	prev := make(map[Point]Point)
	pq := ds.NewPQ(func(a, b item) bool { return a.dist < b.dist })
	pq.Push(item{g.Start, 0})
	for pq.Len() > 0 {
		it, _ := pq.Pop()
		if it.dist > dist[it.p] {
			continue // stale entry: a shorter route was found after it was pushed
		}
		if it.p == g.End {
			return path(prev, g.Start, g.End), it.dist, nil
		}
		g.neighbours(it.p, func(n Point) {
			nd := it.dist + g.cost[n.R*g.cols+n.C]
			if old, ok := dist[n]; !ok || nd < old {
				dist[n] = nd
				prev[n] = it.p
				pq.Push(item{n, nd})
			}
		})
	}
	return nil, 0, ErrNoPath
}

func path(prev map[Point]Point, start, end Point) []Point {
	var out []Point
	for p := end; p != start; p = prev[p] {
		out = append(out, p)
	}
	out = append(out, start)
	for i, j := 0, len(out)-1; i < j; i, j = i+1, j-1 {
		out[i], out[j] = out[j], out[i]
	}
	return out
}

// Draw returns the map with the path marked by '*'.
func (g *Grid) Draw(text string, p []Point) string {
	rows := strings.Fields(text)
	b := make([][]byte, len(rows))
	for i, r := range rows {
		b[i] = []byte(r)
	}
	for _, pt := range p {
		if pt != g.Start && pt != g.End {
			b[pt.R][pt.C] = '*'
		}
	}
	var sb strings.Builder
	for _, r := range b {
		sb.Write(r)
		sb.WriteByte('\n')
	}
	return sb.String()
}
```

### Reference solution — `grid/grid_test.go`

```go
package grid

import (
	"errors"
	"testing"
)

const city = `
S.....#...
.####.#.#.
.#..9.9.#.
.#.##.#.#.
...#....#E
`

func TestBFSAndDijkstraDiffer(t *testing.T) {
	g, err := Parse(city)
	if err != nil {
		t.Fatal(err)
	}
	bp, err := g.BFS()
	if err != nil {
		t.Fatal(err)
	}
	dp, cost, err := g.Dijkstra()
	if err != nil {
		t.Fatal(err)
	}
	if steps := len(bp) - 1; steps != 17 {
		t.Errorf("BFS steps = %d, want 17", steps)
	}
	if steps := len(dp) - 1; steps != 21 || cost != 21 {
		t.Errorf("Dijkstra steps = %d cost = %d, want 21 and 21", steps, cost)
	}
	// The fewest-steps path goes through a '9' cell, so it costs more.
	if bc := pathCost(g, bp); bc != 25 {
		t.Errorf("BFS path cost = %d, want 25", bc)
	}
}

func pathCost(g *Grid, p []Point) int {
	sum := 0
	for _, pt := range p[1:] {
		sum += g.cost[pt.R*g.cols+pt.C]
	}
	return sum
}

func TestNoPath(t *testing.T) {
	g, err := Parse("S#.\n##E")
	if err != nil {
		t.Fatal(err)
	}
	if _, err := g.BFS(); !errors.Is(err, ErrNoPath) {
		t.Errorf("BFS err = %v", err)
	}
	if _, _, err := g.Dijkstra(); !errors.Is(err, ErrNoPath) {
		t.Errorf("Dijkstra err = %v", err)
	}
}

func TestParseErrors(t *testing.T) {
	for _, bad := range []string{"", "S..\n..", "S.x\n..E", "...\n..E"} {
		if _, err := Parse(bad); err == nil {
			t.Errorf("Parse(%q) succeeded", bad)
		}
	}
}
```

### Reference solution — `cmd/gridpath/main.go`

```go
// Command gridpath reads a city map on stdin and prints the fewest-steps
// path (BFS) and the cheapest path (Dijkstra).
package main

import (
	"fmt"
	"io"
	"os"

	"example.com/dsgrid/grid"
)

func main() {
	data, err := io.ReadAll(io.LimitReader(os.Stdin, 10<<20))
	if err != nil {
		fmt.Fprintln(os.Stderr, "gridpath:", err)
		os.Exit(1)
	}
	g, err := grid.Parse(string(data))
	if err != nil {
		fmt.Fprintln(os.Stderr, "gridpath:", err)
		os.Exit(1)
	}
	bp, err := g.BFS()
	if err != nil {
		fmt.Fprintln(os.Stderr, "gridpath:", err)
		os.Exit(1)
	}
	dp, cost, _ := g.Dijkstra()
	fmt.Printf("fewest steps (BFS): %d steps\n%s\n", len(bp)-1, g.Draw(string(data), bp))
	fmt.Printf("cheapest (Dijkstra): %d steps, cost %d\n%s", len(dp)-1, cost, g.Draw(string(data), dp))
}
```

A real run on the sample map:

```
fewest steps (BFS): 17 steps
S*****#***
.####*#*#*
.#..9***#*
.#.##.#.#*
...#....#E

cheapest (Dijkstra): 21 steps, cost 21
S*****#***
.####*#*#*
.#..9*9*#*
.#.##*#*#*
...#.***#E
```

BFS takes the short route straight through a `9` (17 steps, but cost 25); Dijkstra walks 4 more steps to avoid both 9s and pays only 21. This is the difference between "shortest" and "fastest" route that every navigation app must get right. (A note on honesty: when the test for this project was first written, its expected numbers were *guessed* — and wrong. The real run corrected them. Never fill expected values from your head.)

### Review checklist

- Does every structure handle the empty case (Pop on empty, LRU of capacity 1, union of an element with itself)?
- Is the PQ tested against a reference implementation (differential testing) rather than a few hand-picked cases?
- Are popped slots zeroed in `Queue` and `PQ` (DT-212)?
- In `Dijkstra`, is the stale-entry check present? (Remove it and measure how many extra nodes are expanded on a 1000×1000 map.)
- Would the grid code survive a 10,000×10,000 map? The `map[Point]...` structures are the bottleneck: replace them with flat slices indexed by `r*cols+c` and benchmark (Unit 6.13).

### Extensions

1. Replace the maps in `BFS` and `Dijkstra` with flat `[]int32` slices; benchmark on a generated 2000×2000 map.
2. Add diagonal moves with cost × √2 (use float costs) — and think about how to keep costs exact (P-8).
3. Implement A* with the Manhattan distance as the heuristic (preview of Unit 13.6) and count expanded nodes vs Dijkstra.

## Answers to the Chapter 6 prediction questions

**6.1 Q2.** (a) O(n²) (b) O(n) expected (c) O(n log n) (d) O(total length) (e) O(b·l) for b books and l loans (DT-186).

**6.5 Q2.** `(1, true)` — the first 3 (`BinarySearch` returns the lowest index of equal elements); `(4, false)` — where 4 would be inserted.

**6.8 Q1.** Parent (9−1)/2 = 4; children 2·9+1 = 19 and 20.

# Chapter 7 — Concurrency

*Goal: write correct, leak-free concurrent programs.*

Concurrency is why many teams choose Go. Goroutines are cheap enough to start one per request or per task, channels make communication between them a first-class language feature, and the runtime spreads the work across all CPU cores. But concurrency is also where the hardest bugs live: races that appear once a week in production, deadlocks, goroutines that leak until the server runs out of memory. This chapter teaches both the power and the discipline.

The guiding proverb: **"Do not communicate by sharing memory; instead, share memory by communicating."** Prefer passing ownership of data through channels; when you must share, protect it with a mutex — and always know which goroutine owns what.

**What you will be able to do after this chapter**

- Explain processes, threads, goroutines, scheduling, concurrency vs parallelism, and what a data race is.
- Use channels, `select`, timeouts, `sync` primitives, and atomics correctly.
- Explain the Go memory model's happens-before rule well enough to judge whether code is race-free.
- Find races with the race detector, and avoid deadlocks and goroutine leaks.
- Propagate cancellation and deadlines with `context`.
- Build worker pools, pipelines, fan-out/fan-in, semaphores, rate limiters, and error groups — and test them.

## Topic 7A — Foundations

## Unit 7.1 — The operating-system view

**One-line summary:** A process is a running program with its own memory; a thread is a sequence of execution inside a process, sharing that memory. The OS scheduler switches threads on and off CPU cores. Concurrency is *dealing with* many things at once; parallelism is *doing* many things at once. When two threads access the same memory and at least one writes, without synchronisation, the result is a **data race**.

**Mental model — a restaurant kitchen.** The restaurant is a process: one building, one store room (memory). Cooks are threads: they work independently but share the store room. The head chef (OS scheduler) decides who works at which stove (CPU core); with four stoves, at most four cooks cook at the same instant (parallelism), but a dozen orders can be "in progress" (concurrency). If two cooks reach for the same jar of salt without coordinating, one adds salt the other did not expect — a race.

### Processes and threads

| | Process | OS thread |
| --- | --- | --- |
| Memory | own private address space (Unit 0.6) | shares its process's memory |
| Creation cost | high (milliseconds) | moderate (tens of µs, ~1–8 MB of stack reserved) |
| Switching cost | high | about a microsecond (save registers, enter kernel) |
| Communication | pipes, sockets, files (copying) | shared memory (fast, dangerous) |

### Scheduling

The OS gives each runnable thread a time slice on a core, then **preempts** it (interrupts it) and runs another. A thread also leaves the CPU when it blocks — waiting for disk, network, a lock, or a sleep. From your program's point of view, a thread can be paused **between any two instructions**, for an unpredictable time. All concurrency reasoning follows from that.

### Concurrency vs parallelism

- **Concurrency:** structuring a program as independently executing tasks (a web server handling 10,000 connections, most of them waiting for the network).
- **Parallelism:** executing several computations at the same instant on several cores (resizing 1,000 images on 8 cores).

A concurrent program runs correctly on one core; on many cores it may also run in parallel. Go lets you write concurrent code and the runtime provides parallelism up to `GOMAXPROCS` (by default the number of usable cores — and since Go 1.25, the container's CPU limit, Unit 11.8).

### What a race is

```go
counter := 0
var wg sync.WaitGroup
for range 1000 {
	wg.Add(1)
	go func() {
		defer wg.Done()
		counter++ // read, add, write — three steps, not one
	}()
}
wg.Wait()
fmt.Println("counter:", counter)
```

Three real runs of this program printed **983, 979, and 982**. `counter++` is a read, an addition, and a write. Two goroutines both read 41, both add 1, both write 42: one increment is lost. This is a **data race**: concurrent access to the same memory, at least one a write, with no synchronisation. In Go, a program with a data race has **no guaranteed behaviour** beyond "something reasonable for that word of memory" — and with multi-word values (strings, slices, interfaces) a race can even produce a torn value that crashes the program (DT-225).

### Mastery check

1. **Explain.** Use the kitchen analogy to explain the difference between concurrency and parallelism.
2. **Explain.** Why is `counter++` not safe from multiple goroutines? Describe the interleaving that loses an update.

## Unit 7.2 — Goroutines

**One-line summary:** `go f(x)` starts `f` running concurrently in a new goroutine — a function execution managed by the Go runtime, not by the OS. Goroutines start with tiny stacks and are multiplexed onto a few OS threads, so a program can run hundreds of thousands of them. The program exits when `main` returns, without waiting for other goroutines.

### Starting goroutines

```go
go sendEmail(user)                 // runs concurrently; the call returns immediately
go func(id int) { process(id) }(7) // arguments are evaluated NOW, in the caller (like defer, DT-88)
```

There is no handle, no return value, and no way to kill a goroutine from outside. A goroutine ends when its function returns. To get results back, to wait for it, or to stop it, you build that in: channels (Unit 7.3), `sync.WaitGroup` (Unit 7.5), `context` (Unit 7.9).

### `main` does not wait

```go
func main() {
	go fmt.Println("hello from a goroutine")
} // main returns: the program exits, probably before the goroutine runs
```

When `main` returns, the process exits and all goroutines vanish mid-flight — deferred calls in them do not run. Every goroutine you start needs an answer to: **who waits for it, and how does it end?**

### A first look at the scheduler

The runtime runs goroutines on OS threads using the **G-M-P model** (details in Unit 10.7):

- **G** — a goroutine: its stack, its program counter, its state.
- **M** — an OS thread ("machine").
- **P** — a processor context, `GOMAXPROCS` of them; an M must hold a P to run Go code. Each P has a queue of runnable Gs.

When a goroutine blocks on a channel, a mutex, a sleep, or network I/O, the runtime parks it and runs another G on the same thread — a switch costing tens of nanoseconds, with no kernel involvement. Network I/O uses the OS's event notification (epoll, kqueue), so 10,000 goroutines waiting on sockets need only a handful of threads. Blocking *system calls* (disk I/O, cgo) do occupy a thread; the runtime hands the P to another thread meanwhile. Since Go 1.14, long-running goroutines are **preempted asynchronously**, so a tight loop cannot starve the others.

### Goroutine stacks

A goroutine starts with a small stack (a few KB; since Go 1.19 sized from observed averages) and grows by copying (DT-35) up to 1 GB. Thousands of mostly-idle goroutines cost only megabytes. This is what makes "one goroutine per connection" a sound design in Go, where it would be wasteful with OS threads.

### Deep & Tricky

**DT-222. An unrecovered panic in any goroutine kills the whole program.** *(Guarantee)* `recover` only works in the panicking goroutine's own deferred functions (Unit 3.11). A library that starts goroutines must recover inside them (and report the error) if a panic there should not crash its host. `net/http` does this for handler goroutines.

**DT-223. Goroutines have no identity you can use.** *(Language design)* There is no public goroutine ID, no thread-local storage, no `goroutine.current()`. This is deliberate: code that depends on "which goroutine am I" breaks when work moves between goroutines. Pass what you need (a `context`, a logger, a request ID) explicitly.

### Mastery check

1. **Explain.** Why can Go run 100,000 goroutines when 100,000 OS threads would be a problem?
2. **Predict.** `for i := range 3 { go fmt.Print(i) }` followed by `time.Sleep(10 * time.Millisecond)` — what can be printed? What if the sleep is removed?
3. **Write.** Start 10,000 goroutines that each sleep 1 second, wait for all with a `WaitGroup`, and measure total time and memory (`runtime.ReadMemStats` before and after). Explain the numbers.

## Topic 7B — Channels

## Unit 7.3 — Channels

**One-line summary:** A channel is a typed, synchronised pipe between goroutines. Sending and receiving on an **unbuffered** channel happen together (a handshake); a **buffered** channel holds up to N values. Closing a channel says "no more values"; receivers see the remaining values, then zero values with `ok == false`. Nil channels block forever.

**Mental model — handing over a parcel.** An unbuffered channel is a hand-to-hand handover: the sender waits until a receiver takes the parcel, so both know the handover happened. A buffered channel is a pigeonhole rack with N slots: the sender drops the parcel and leaves, unless the rack is full. Closing the channel is putting up a sign: "no more parcels".

### Basics

```go
ch := make(chan int)      // unbuffered
buf := make(chan int, 10) // buffered, capacity 10

go func() { ch <- 42 }()  // send: blocks until someone receives
v := <-ch                 // receive: blocks until someone sends
```

| Operation | Unbuffered | Buffered (capacity N) |
| --- | --- | --- |
| send | blocks until a receiver takes it | blocks only if N values are already waiting |
| receive | blocks until a sender sends | blocks only if empty |
| `len(ch)`, `cap(ch)` | 0, 0 | values waiting, N |

The unbuffered handshake is also a **synchronisation**: everything the sender did before the send is visible to the receiver after the receive (Unit 7.7).

### Directional channel types

```go
func produce(out chan<- int) { out <- 1 } // can only send
func consume(in <-chan int)  { <-in }     // can only receive
```

A `chan int` converts implicitly to either direction. Use directions in function signatures: they document ownership and the compiler enforces them.

### Close and range

```go
func numbers(n int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out) // the SENDER closes, when done
		for i := range n {
			out <- i
		}
	}()
	return out
}

for v := range numbers(3) { // receives until the channel is closed and drained
	fmt.Println(v)            // 0 1 2
}
```

- After `close(ch)`, receivers get the buffered values, then the zero value immediately, forever: `v, ok := <-ch` gives `ok == false`.
- **Sending on a closed channel panics** (`send on closed channel`). **Closing a closed channel panics** (`close of closed channel`).
- Rule: **only the sender closes, and only when no more sends can happen.** With several senders, a separate coordinator closes after all senders finish (a `WaitGroup`, Unit 7.10).
- Closing is not required for garbage collection; close only to *signal* "done" to receivers.

### Nil channels

Sending to or receiving from a nil channel blocks **forever**. This sounds useless, but in `select` it is a tool: setting a channel variable to nil disables its case (Unit 7.4).

### Deep & Tricky

**DT-224. A buffered channel is not a queue you should inspect.** *(Practice)* `len(ch)` is outdated the moment you read it — another goroutine may send or receive right after. Never write `if len(ch) > 0 { v := <-ch }`; use `select` with `default` for non-blocking operations.

**DT-225. Races on multi-word values can crash, not just give wrong numbers.** *(Implementation consequence)* A string or slice header is written as two or three words. A reader racing with a writer can see the new pointer with the old length, then read past the end of memory. Interfaces (type + data words) can be torn so that a method is called on data of the wrong type. "It is only a counter, a small race is fine" is never true in Go.

### Mastery check

1. **Explain.** Why must only the sender close a channel? What goes wrong with several senders?
2. **Predict.** (a) `ch := make(chan int, 1); ch <- 1; ch <- 2` (in `main`, nothing else running) (b) `ch := make(chan int); close(ch); v, ok := <-ch` (c) `var ch chan int; <-ch` (d) `close(ch)` twice.
3. **Write.** A generator `primes(ctx)` returning `<-chan int` that produces primes until the context is cancelled, closing the channel when it stops.

## Unit 7.4 — select

**One-line summary:** `select` waits on several channel operations at once and runs the first one that can proceed (choosing randomly among several ready ones). With `default` it never blocks. Combined with timers and `ctx.Done()`, it expresses timeouts, cancellation, and multiplexing.

### Basics

```go
select {
case msg := <-messages:
	handle(msg)
case results <- next: // a send can be a case too
	next = compute()
case <-time.After(2 * time.Second):
	fmt.Println("timed out")
case <-ctx.Done():
	return ctx.Err()
}
```

- If several cases are ready, one is chosen **uniformly at random** (so no case starves).
- If none is ready, `select` blocks until one is.
- With a `default` case, `select` does not block: `default` runs if nothing is ready.

### Non-blocking send and receive

```go
select {
case events <- e:
default:
	dropped++ // buffer full: drop rather than block the caller (e.g. telemetry)
}
```

### Timeouts

```go
select {
case r := <-resultCh:
	return r, nil
case <-time.After(500 * time.Millisecond):
	return Result{}, errors.New("timeout")
}
```

Since Go 1.23, the timer behind an abandoned `time.After` is garbage-collected promptly, so this idiom no longer leaks timers in loops (DT-226 covers the old caveat). For a deadline that applies to a whole operation and everything it calls, use `context.WithTimeout` (Unit 7.9).

### Disabling cases with nil channels

Merging two channels until both are closed:

```go
func merge(a, b <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for a != nil || b != nil {
			select {
			case v, ok := <-a:
				if !ok {
					a = nil // closed: disable this case
					continue
				}
				out <- v
			case v, ok := <-b:
				if !ok {
					b = nil
					continue
				}
				out <- v
			}
		}
	}()
	return out
}
```

### Deep & Tricky

**DT-226. Before Go 1.23, `time.After` in a loop leaked timers until they fired.** *(Version-dependent)* A `select` with `case <-time.After(time.Minute)` inside a busy loop created a new timer each iteration that stayed alive for a minute. Older code uses `time.NewTimer` with `Stop`/`Reset` for this reason. Modules with `go 1.23`+ do not need to.

**DT-227. An empty `select {}` blocks forever.** *(Guarantee)* It is occasionally used at the end of `main` in programs whose work happens in other goroutines. If those goroutines all exit, the runtime reports `fatal error: all goroutines are asleep - deadlock!`.

### Mastery check

1. **Explain.** Why does `select` choose randomly among ready cases?
2. **Write.** `firstOf(ctx, fetchers ...func(context.Context) (string, error)) (string, error)` that runs all fetchers concurrently and returns the first success, cancelling the rest — with no goroutine leaks (check with `runtime.NumGoroutine` in a test).

## Topic 7C — Shared-memory synchronization

## Unit 7.5 — The sync package

**One-line summary:** `sync.WaitGroup` waits for a group of goroutines; `sync.Mutex` and `sync.RWMutex` protect shared data so only one goroutine (or many readers) touches it at a time; `sync.Once`, `OnceFunc`, and `OnceValue` run initialisation exactly once; `sync.Cond` is a rarely needed wait/signal primitive.

### WaitGroup

```go
var wg sync.WaitGroup
for _, u := range urls {
	wg.Add(1) // BEFORE starting the goroutine
	go func() {
		defer wg.Done()
		fetch(u) // Go 1.22+: u is per-iteration (DT-72)
	}()
}
wg.Wait()
```

Since Go 1.25 you can write `wg.Go(func() { fetch(u) })`, which does the `Add(1)` and `defer Done()` for you and removes the most common WaitGroup bug (calling `Add` inside the goroutine, after `Wait` may already have returned). Go 1.27's `go fix` includes a modernizer (`waitgroupgo`) that rewrites the old pattern.

### Mutex

```go
type Counter struct {
	mu sync.Mutex     // protects the fields below it
	m  map[string]int
}

func (c *Counter) Inc(key string) {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.m[key]++
}

func (c *Counter) Value(key string) int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.m[key]
}
```

Rules:

- Keep the critical section (the code between `Lock` and `Unlock`) **short**. Never do network or disk I/O while holding a lock.
- Every access — reads too — must hold the lock. A read without the lock while another goroutine writes is still a race.
- `defer mu.Unlock()` right after `Lock` (P-13) keeps unlock correct on every path, including panics.
- A mutex must not be copied after first use (DT-136): use pointer receivers, and never embed it publicly (DT-152).
- Go's mutexes are **not reentrant**: a goroutine that calls `Lock` twice deadlocks itself. Structure code so that exported methods lock and unexported helpers assume the lock is held (and say so in a comment).

### RWMutex

`sync.RWMutex` allows many simultaneous readers (`RLock`/`RUnlock`) or one writer (`Lock`/`Unlock`). Use it when reads vastly outnumber writes *and* critical sections are not tiny; otherwise a plain `Mutex` is often faster (RWMutex has more overhead). Measure.

### Once, OnceFunc, OnceValue

```go
var loadConfig = sync.OnceValues(func() (*Config, error) { // Go 1.21
	return readConfig("app.yaml")
})

cfg, err := loadConfig() // the first caller runs readConfig; everyone else waits and gets the same result
```

`sync.Once{}.Do(f)` is the older form. `OnceFunc`, `OnceValue`, and `OnceValues` (Go 1.21) are simpler and also re-panic consistently if `f` panicked. Use for lazy initialisation of expensive shared resources.

### Cond

`sync.Cond` lets goroutines wait until a condition on shared state becomes true. It is easy to misuse, and channels usually express the same thing more clearly. You will rarely need it; when you see it in libraries (bounded queues), recognise the pattern: `for !condition { c.Wait() }` while holding the lock.

### Deep & Tricky

**DT-228. `wg.Add` inside the goroutine is a race with `Wait`.** *(Classic bug)* If the goroutine has not run yet when `Wait` is called, the counter may be 0, and `Wait` returns immediately. Always `Add` before `go`, or use `wg.Go` (Go 1.25+).

**DT-229. A mutex protects data, not code.** *(Design principle)* Document which fields a mutex guards (put them right below it with a comment). Two methods that lock the same mutex but touch different data, or one mutex for unrelated data, are design smells: contention without protection or protection without clarity.

**DT-230. Go's `Mutex` can be unlocked by a different goroutine than the one that locked it.** *(Guarantee, unlike many languages)* It is legal but almost always a sign of confused ownership. Keep Lock and Unlock in the same function.

### Pattern

**P-32. Confine state to one goroutine, or guard it with a documented mutex.** *(Concurrency practice)*

- *Problem it solves:* data races and unclear ownership.
- *How:* either (a) **confinement** — one goroutine owns the data and others send it requests through channels — or (b) a struct that holds the data and a `sync.Mutex`, with every method locking. Document which fields the mutex protects.
- *When not to use channels:* for simple shared counters, caches, and registries a mutex is clearer and faster. Channels shine for passing ownership, signalling, and pipelines.
- *Trade-off:* confinement makes races impossible by construction but adds message passing; mutexes are direct but rely on discipline.

### Mastery check

1. **Explain.** Why is a read without a lock still a race if another goroutine writes under the lock?
2. **Predict.** A method `func (c *Cache) Get(k string) string { c.mu.Lock(); defer c.mu.Unlock(); return c.load(k) }` where `load` also calls `c.mu.Lock()`. What happens?
3. **Write.** Make the `ds.LRU` from Chapter 6 safe for concurrent use with a `sync.Mutex`, and a test that hammers it from 50 goroutines under `-race`. Why can `Get` not use `RLock`?

## Unit 7.6 — sync/atomic and typed atomics

**One-line summary:** Atomic operations read, write, add, swap, or compare-and-swap a single variable indivisibly, without a lock. Use the typed atomics (`atomic.Int64`, `atomic.Bool`, `atomic.Pointer[T]`) for counters, flags, and lock-free publication of immutable snapshots.

```go
type Stats struct {
	requests atomic.Int64
	healthy  atomic.Bool
	config   atomic.Pointer[Config] // swap in a whole new immutable config
}

s.requests.Add(1)
n := s.requests.Load()
s.healthy.Store(true)
s.config.Store(&Config{...})
cfg := s.config.Load() // readers get a consistent snapshot, never a half-updated one
```

- **Compare-and-swap** (`CompareAndSwap(old, new)`) succeeds only if the value is still `old` — the building block of lock-free algorithms. The Chapter 7 project's test uses a CAS loop to track a maximum.
- The fixed counter from Unit 7.1: `var counter atomic.Int64` with `counter.Add(1)` prints 1000 every time.
- Atomics make **one variable** safe. Two related variables updated with two atomic operations are *not* updated together; use a mutex, or put both in one immutable struct behind an `atomic.Pointer`.
- Typed atomics (Go 1.19) are preferred over the older functions (`atomic.AddInt64(&x, 1)`), because a typed atomic cannot be accidentally read non-atomically and is correctly aligned on 32-bit platforms (DT-130). Go 1.27's `go fix` has an `atomictypes` modernizer for this.

### Deep & Tricky

**DT-231. Atomics are not free under contention.** *(Hardware fact)* An atomic add is a few nanoseconds uncontended, but when many cores update the same variable, its cache line bounces between cores (Unit 7.11), and each operation can cost a hundred nanoseconds. High-traffic counters are often **sharded** (one counter per P or per CPU, summed when read).

### Mastery check

1. **Explain.** Why is "check then act" with two atomic calls (`if x.Load() == 0 { x.Store(1) }`) racy, and how does `CompareAndSwap` fix it?
2. **Write.** A config hot-reload: a goroutine that every second reads a file and, if changed and valid, `Store`s a new `*Config`; handlers `Load` it. Test with `-race`.

## Unit 7.7 — The Go memory model

**One-line summary:** Without synchronisation, one goroutine is **not guaranteed** to see another's writes — or to see them in order — because compilers and CPUs reorder memory operations. The memory model defines **happens-before**: if a write happens before a read through a chain of synchronisation (channel operations, mutex unlock/lock, `WaitGroup`, `Once`, atomics), the read sees the write. A program without data races behaves as if executed sequentially.

### Why intuition fails

```go
var data int
var ready bool

go func() {
	data = 42
	ready = true
}()

for !ready { } // RACE: may spin forever (the compiler can hoist the read out of the loop)
fmt.Println(data) // and even if ready is seen, data may still read 0
```

Both the compiler (keeping `ready` in a register) and the CPU (store buffers, out-of-order execution) may make this fail. Nothing in this code establishes that `data = 42` *happens before* the print.

### Happens-before edges you can rely on

- Within one goroutine, statements happen in program order.
- `go f()` happens before `f` starts.
- A **send** on a channel happens before the corresponding **receive** completes. (For an unbuffered channel, the receive also happens before the send completes.)
- **Closing** a channel happens before a receive that returns because the channel is closed.
- The k-th receive on a channel with capacity C happens before the (k+C)-th send completes — which is why a buffered channel works as a semaphore.
- `mu.Unlock()` happens before any later `mu.Lock()` returns.
- `wg.Done()` happens before the `wg.Wait()` it unblocks.
- `once.Do(f)`: the completion of `f` happens before any `Do` returns.
- Atomic operations: if an atomic read observes an atomic write, the write happens before the read (Go's atomics are sequentially consistent).

Happens-before is **transitive**. The fixed version of the example:

```go
done := make(chan struct{})
go func() {
	data = 42
	close(done) // data = 42 happens before close
}()
<-done           // close happens before this receive returns
fmt.Println(data) // so this is guaranteed to print 42
```

### The practical rule

**If two goroutines access the same variable and at least one writes, there must be a happens-before chain between the accesses — through a channel, a lock, a WaitGroup, a Once, or an atomic.** If you cannot point to that chain, there is a race, and the race detector (Unit 7.8) will usually find it.

### Deep & Tricky

**DT-232. "Benign" races do not exist in Go.** *(Memory model)* The 2022 revision of the Go memory model gives racy programs limited, but not zero, guarantees: a racy read of a single machine word returns some value actually written, but multi-word values can be torn (DT-225), and the compiler may assume no races when optimising. Treat every reported race as a bug.

**DT-233. Double-checked locking without atomics is broken.** *(Classic bug)* `if instance == nil { mu.Lock(); if instance == nil { instance = new(T) }; mu.Unlock() }` — the first unlocked read races with the write, and a reader may see a non-nil pointer to a not-yet-initialised object. Use `sync.Once` / `OnceValue`.

### Mastery check

1. **Explain.** Name the happens-before chain that makes `fmt.Println(data)` correct in the fixed example.
2. **Explain.** Why can `for !ready {}` loop forever even after another goroutine sets `ready = true`?

## Topic 7D — Correctness

## Unit 7.8 — Races, deadlocks, and leaks

**One-line summary:** Three ways concurrent programs fail: **races** (unsynchronised access — find them with `-race`), **deadlocks** (goroutines waiting on each other forever — the runtime detects only the total kind), and **leaks** (goroutines blocked forever, silently consuming memory — prevent with ownership and cancellation, detect with goroutine counts and profiles).

### The race detector

```
$ go test -race ./...
$ go build -race -o app && ./app
```

The detector instruments every memory access and reports conflicting accesses not ordered by happens-before. Real output from the counter of Unit 7.1:

```
==================
WARNING: DATA RACE
Read at 0x00c000114078 by goroutine 12:
  main.main.func1()
      /tmp/s/main.go:27 +0x84

Previous write at 0x00c000114078 by goroutine 7:
  main.main.func1()
      /tmp/s/main.go:27 +0x96

Goroutine 12 (running) created at:
  main.main()
      /tmp/s/main.go:25 +0x7cd
...
==================
```

It shows both accesses with stacks and where each goroutine was created. Important limits: it finds races **that actually happen during the run** (so tests must exercise concurrent paths), it slows programs 2–20× and uses 5–10× memory, and a program with detected races exits with status 66 by default. Run all tests with `-race` in CI (Unit 11.7), and consider a canary deployment of a race-enabled build.

### Deadlocks

A deadlock is a cycle of waiting: A holds lock 1 and waits for lock 2; B holds lock 2 and waits for lock 1. Or a goroutine sends on an unbuffered channel no one will ever receive from.

If **every** goroutine is blocked, the runtime notices:

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
```

But if any goroutine is still running (a timer, an HTTP server's listener), a partial deadlock is **not** detected: the stuck goroutines simply wait forever. Prevention:

- Acquire multiple locks in a **consistent global order**.
- Do not hold a lock while sending on a channel or calling unknown code (callbacks).
- Every blocking operation should be able to give up: `select` with `ctx.Done()`.

### Goroutine leaks

A leaked goroutine is blocked forever: waiting to send to a channel nobody reads, or to receive from a channel nobody closes. It holds its stack and everything it references. Classic example:

```go
func firstResult(urls []string) string {
	ch := make(chan string) // unbuffered
	for _, u := range urls {
		go func() { ch <- fetch(u) }()
	}
	return <-ch // the other len(urls)-1 goroutines block forever on send: LEAK
}
```

Fixes: a buffered channel with room for every sender (`make(chan string, len(urls))`), or a `select` on `ctx.Done()` in the senders with cancellation when the first result arrives.

Detecting leaks:

- In tests, compare `runtime.NumGoroutine()` before and after (allowing a moment for goroutines to exit), or use `go.uber.org/goleak`.
- In production, the goroutine profile (`/debug/pprof/goroutine`, Unit 10.2) shows thousands of goroutines stuck at the same line.
- Go 1.27 made the **goroutine leak profile** generally available (`goroutineleak` in `runtime/pprof` and `/debug/pprof/goroutineleak`): it uses the garbage collector's reachability to find goroutines blocked on channels or locks that no running code can ever reach again.

### Deep & Tricky

**DT-234. The race detector cannot prove absence of races.** *(Tool limitation)* It reports races in executions it observes. A race in a code path your tests never run concurrently stays hidden. Combine `-race` with tests that deliberately run things in parallel (`t.Parallel`, many goroutines, `-count=50`).

**DT-235. Every goroutine needs an owner and an exit path.** *(Design rule)* Before writing `go`, answer: what makes this goroutine return? Who waits for it? What happens to it on error, on timeout, on shutdown? Most leaks come from an exit path that only exists for the happy case.

### Mastery check

1. **Explain.** Why did the runtime detect the deadlock in a program with one goroutine, but would not detect it in an HTTP server?
2. **Predict.** Does the `firstResult` example leak if `urls` has one element? Five? What is the cheapest fix?
3. **Write.** Deliberately create a lock-order deadlock between two goroutines, observe it (with a timeout in a test and a goroutine dump via `SIGQUIT` / `Ctrl+\`), then fix it with a consistent order.

## Topic 7E — Cancellation and patterns

## Unit 7.9 — context

**One-line summary:** A `context.Context` carries a **cancellation signal**, a **deadline**, and a few **request-scoped values** across API boundaries and goroutines. Every function that does I/O or may take long should accept `ctx context.Context` as its first parameter, pass it down, and stop when `ctx.Done()` is closed.

**Mental model — the order slip with a "cancel" checkbox.** When a customer cancels an order at the counter, the slip carrying the order number is marked cancelled, and everyone working on that order — kitchen, packing, delivery — checks the slip and stops. A context is that slip, passed to everyone working on one request.

### Creating contexts

```go
ctx := context.Background()                          // the root: in main, tests, top of a request
ctx, cancel := context.WithCancel(parent)            // cancel manually
ctx, cancel := context.WithTimeout(parent, 2*time.Second)
ctx, cancel := context.WithDeadline(parent, t)
ctx, cancel := context.WithCancelCause(parent)       // cancel(err) records why (Go 1.20)
defer cancel()                                        // ALWAYS: releases resources (go vet checks this)
```

Contexts form a tree: cancelling a parent cancels all its children; a child's deadline can only be *earlier* than its parent's. HTTP servers give each request a context that is cancelled when the client disconnects (Chapter 8); `signal.NotifyContext` gives one cancelled on Ctrl+C (Unit 5.7).

### Respecting cancellation

```go
func process(ctx context.Context, jobs <-chan Job) error {
	for {
		select {
		case <-ctx.Done():
			return ctx.Err() // context.Canceled or context.DeadlineExceeded
		case j, ok := <-jobs:
			if !ok {
				return nil
			}
			if err := handle(ctx, j); err != nil { // pass ctx down
				return err
			}
		}
	}
}
```

Standard-library I/O takes contexts: `http.NewRequestWithContext`, `db.QueryContext` (Chapter 9), `exec.CommandContext`, `net.Dialer.DialContext`. A CPU-bound loop should check `ctx.Err()` every so often (the `logscan` project checked every 4096 lines).

### Conventions

- `ctx` is the **first parameter**, named `ctx`. Do not store it in structs (a struct outlives the request); pass it on every call.
- Never pass a nil context; use `context.TODO()` while refactoring.
- Check errors with `errors.Is(err, context.Canceled)` / `context.DeadlineExceeded` — they are usually wrapped.
- `context.WithoutCancel(ctx)` (Go 1.21) keeps values but drops cancellation — for work that must finish even if the request ends (writing an audit log). `context.AfterFunc(ctx, f)` (Go 1.21) runs `f` when ctx is done.

### Values — and when not to use them

```go
type ctxKey struct{} // unexported key type: no collisions with other packages

func WithRequestID(ctx context.Context, id string) context.Context {
	return context.WithValue(ctx, ctxKey{}, id)
}

func RequestID(ctx context.Context) string {
	id, _ := ctx.Value(ctxKey{}).(string)
	return id
}
```

Use values only for **request-scoped data that crosses API boundaries and is not a function parameter**: request IDs, trace spans, authenticated user identity. Never for optional parameters, database handles, or configuration — that hides dependencies and defeats the type system.

### Deep & Tricky

**DT-236. Forgetting `cancel()` leaks until the parent is cancelled.** *(Library behaviour)* Each `WithCancel`/`WithTimeout` registers the child with its parent. Without `cancel()`, the child (and a timer, for timeouts) stays alive until the parent is done — with `context.Background()` as parent, forever. `go vet`'s `lostcancel` check reports it.

**DT-237. Cancellation is cooperative.** *(Design)* Cancelling a context does not stop any goroutine by force; it only closes a channel. Code that never checks `ctx.Done()` (or calls functions that do) keeps running. A goroutine stuck in a non-context-aware call (a blocking read on a raw file, a C library) cannot be cancelled at all.

### Pattern

**P-33. Context first, everywhere work can block.** *(Universal Go convention)* Every exported function that performs I/O, waits, or loops for a long time takes `ctx context.Context` first and honours it. *Trade-off:* a parameter on many functions, in exchange for timeouts and cancellation that work end to end — the basis of robust services (Unit 8.7, Unit 12.7).

### Mastery check

1. **Explain.** What happens to a child context when its parent is cancelled? When the child's own timeout is longer than the parent's?
2. **Explain.** Why should a database handle not be stored in a context value?
3. **Write.** `retry(ctx, attempts int, f func(context.Context) error) error` with exponential backoff (100 ms, 200 ms, 400 ms…) that stops immediately when the context is cancelled, returning a wrapped error that still satisfies `errors.Is(err, context.Canceled)`.

## Unit 7.10 — Concurrency patterns

**One-line summary:** A handful of patterns cover most real concurrent code: **worker pools** (bounded parallel processing), **pipelines** (stages connected by channels), **fan-out/fan-in**, **semaphores** (bounding concurrency), **error groups** (first error cancels the rest), **backpressure**, and **rate limiting**.

### Worker pool

```go
func processAll(ctx context.Context, jobs []Job, workers int) []Result {
	in := make(chan Job)
	out := make(chan Result)

	var wg sync.WaitGroup
	for range workers {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for j := range in {
				select {
				case out <- process(ctx, j):
				case <-ctx.Done():
					return
				}
			}
		}()
	}

	go func() { // feeder: owns `in`, closes it
		defer close(in)
		for _, j := range jobs {
			select {
			case in <- j:
			case <-ctx.Done():
				return
			}
		}
	}()

	go func() { // closer: closes `out` after all workers have finished
		wg.Wait()
		close(out)
	}()

	var results []Result
	for r := range out {
		results = append(results, r)
	}
	return results
}
```

Note the ownership: the feeder owns `in` and closes it; a separate goroutine closes `out` after the `WaitGroup` says every sender is done. Results arrive in completion order; carry an index in `Result` if order matters.

### Pipelines and fan-out / fan-in

A pipeline is stages connected by channels: `generate → parse → enrich → store`, each stage a goroutine (or a pool) reading from one channel and writing to the next. **Fan-out** runs several goroutines reading the same input channel; **fan-in** merges several channels into one (the `merge` of Unit 7.4). Each stage must stop on `ctx.Done()` and close its output when its input is exhausted, so shutdown propagates cleanly down the pipeline.

### Semaphores: bounding concurrency

A buffered channel of capacity N is a counting semaphore:

```go
sem := make(chan struct{}, 10) // at most 10 at once
for _, f := range files {
	sem <- struct{}{} // acquire (blocks when 10 are running)
	wg.Add(1)
	go func() {
		defer wg.Done()
		defer func() { <-sem }() // release
		upload(f)
	}()
}
wg.Wait()
```

The crawler in this chapter's project uses exactly this to cap in-flight requests. `golang.org/x/sync/semaphore` offers a weighted version with context support.

### errgroup: the first error cancels the rest

```go
import "golang.org/x/sync/errgroup"

g, ctx := errgroup.WithContext(ctx)
g.SetLimit(8) // at most 8 goroutines at once
for _, id := range ids {
	g.Go(func() error {
		return syncUser(ctx, id) // ctx is cancelled as soon as any call returns an error
	})
}
if err := g.Wait(); err != nil { // the first error
	return err
}
```

`errgroup` (in the `golang.org/x/sync` module maintained by the Go team) is the standard tool for "run these in parallel, fail fast". Prefer it over hand-rolled WaitGroup-plus-error-channel code.

### Backpressure

When producers are faster than consumers, something must give. A bounded channel makes the producer wait (backpressure propagates upstream — good for pipelines); a `select` with `default` drops work (good for telemetry); an unbounded queue just converts the problem into an out-of-memory crash later. Choose deliberately.

### Rate limiting

Bounding concurrency ("at most 10 at a time") is different from bounding rate ("at most 5 per second"). The simplest rate limiter is a ticker:

```go
tick := time.NewTicker(200 * time.Millisecond) // 5 per second
defer tick.Stop()
for _, req := range requests {
	<-tick.C
	go send(req)
}
```

For bursts and per-client limits, a **token bucket** (`golang.org/x/time/rate`): a bucket refills at r tokens per second up to b tokens; each request takes one token or waits (`limiter.Wait(ctx)`). Unit 12.8 compares rate-limiting algorithms.

### Deep & Tricky

**DT-238. Unbounded goroutine creation is a bug waiting for a big input.** *(Practice)* `for _, x := range items { go work(x) }` with a million items starts a million goroutines — memory, file descriptors, and downstream services all suffer. Bound with a pool, a semaphore, or `errgroup.SetLimit`.

**DT-239. Results from a worker pool arrive in completion order.** *(Consequence of concurrency)* Tests that expect input order will pass sometimes and fail sometimes. Sort, index, or design the output to be order-independent.

### Mastery check

1. **Explain.** Who closes each channel in the worker pool, and why could a worker not close `out`?
2. **Explain.** Bounding concurrency vs bounding rate: give an example where you need each.
3. **Write.** A three-stage pipeline (read lines from a file → parse to numbers → sum) with cancellation; test that cancelling midway stops all stages with no leaked goroutines.

## Topic 7F — Advanced tools and testing

## Unit 7.11 — Advanced tools

**One-line summary:** `sync.Pool` reuses temporary objects to reduce allocations in hot paths; `sync.Map` is a concurrent map optimised for two specific access patterns; **false sharing** makes independent variables slow each other down when they share a cache line.

### sync.Pool

```go
var bufPool = sync.Pool{
	New: func() any { return new(bytes.Buffer) },
}

func render(w io.Writer, data Page) error {
	buf := bufPool.Get().(*bytes.Buffer)
	buf.Reset()
	defer bufPool.Put(buf)
	if err := tmpl.Execute(buf, data); err != nil {
		return err
	}
	_, err := w.Write(buf.Bytes())
	return err
}
```

A pool is a cache of reusable objects that the garbage collector may empty at any time. Use it for **temporary** objects allocated at high rates (buffers in encoders, formatters, HTTP servers). Rules: always reset objects before reuse; never put objects that are still referenced elsewhere; avoid pooling huge buffers (they pin memory — drop buffers above a size limit instead of putting them back). Measure the gain (Chapter 10); `fmt` and `encoding/json` use pools internally.

### sync.Map

`sync.Map` is safe for concurrent use without extra locking and is optimised for (1) keys written once and read many times (caches that only grow), and (2) goroutines working on disjoint sets of keys. For everything else, a plain `map` with a `sync.Mutex` or `RWMutex` is typically faster and clearer, and type-safe. Since Go 1.24 its implementation is a concurrent hash-trie with better performance, but the advice stands.

### False sharing

Two goroutines on two cores each update their *own* counter — no data race — but the counters sit in the same cache line (DT-32, DT-33). Every write invalidates the line in the other core's cache, so they bounce it back and forth, and the "independent" updates run many times slower. Fix by padding hot per-goroutine data to separate cache lines:

```go
type paddedCounter struct {
	n atomic.Int64
	_ [120]byte // 8 + 120 = 128: one line on Apple Silicon, two on x86
}

counters := make([]paddedCounter, runtime.GOMAXPROCS(0))
```

`golang.org/x/sys/cpu.CacheLinePad` provides the right size per architecture. Only pad after a benchmark shows contention: padding wastes memory.

### Mastery check

1. **Explain.** Why may `sync.Pool` drop objects, and why is that acceptable?
2. **Write.** A benchmark with 8 goroutines each incrementing its own `atomic.Int64` 10 million times, first in a plain `[8]atomic.Int64`, then padded. Report the ratio on your machine.

## Unit 7.12 — Testing concurrent code

**One-line summary:** Concurrent tests must be deterministic: no sleeps that "usually" suffice, no dependence on scheduling order. Use synchronisation instead of time, run with `-race` and high `-count`, inject clocks, and use `testing/synctest` (Go 1.25+) to test time-dependent concurrent code in a fake-time "bubble".

### Rules

- **Never use `time.Sleep` to wait for a goroutine.** Wait for an event: a channel receive, `wg.Wait()`, a result. Sleeps make tests slow on fast machines and flaky on slow CI machines.
- When you must wait for a condition, poll with a deadline and a short interval, failing with a clear message — and prefer making the code expose an event to wait on.
- Run concurrency tests with `-race -count=50` (or more) locally to shake out rare interleavings.
- Check for leaks: goroutine count before and after, or `goleak`.
- Inject time (P-28): code that uses `time.Now` and `time.After` directly is hard to test.

### `testing/synctest`

Go 1.25 made `testing/synctest` stable (it was an experiment in 1.24). Inside `synctest.Test`, goroutines run in a **bubble** with a fake clock: time advances only when every goroutine in the bubble is blocked, instantly jumping to the next timer. A test of a 30-second timeout runs in microseconds, deterministically:

```go
func TestTimeout(t *testing.T) {
	synctest.Test(t, func(t *testing.T) {
		ctx, cancel := context.WithTimeout(t.Context(), 30*time.Second)
		defer cancel()
		start := time.Now()     // fake time inside the bubble
		<-ctx.Done()            // all goroutines blocked: the clock jumps 30s
		if d := time.Since(start); d != 30*time.Second {
			t.Errorf("waited %v", d)
		}
	})
}
```

`synctest.Wait()` blocks until every other goroutine in the bubble is durably blocked — a precise "let everything settle" without sleeps. Go 1.27 added `synctest.Sleep` (a sleep followed by `Wait`) and `httptest.NewTestServer`, an HTTP test server on an in-memory network that works inside bubbles.

### Deep & Tricky

**DT-240. A test that passes with `-count=1` can fail with `-count=1000`.** *(Probability)* A race window hit once in 500 runs passes casually and fails in production at scale. Repetition, `-race`, and `GOMAXPROCS` variation (`-cpu=1,2,8`) are how concurrency bugs are flushed out before users find them.

### Mastery check

1. **Explain.** Why is `time.Sleep(100 * time.Millisecond)` in a test both too long and too short?
2. **Write.** Using `synctest` (Go 1.25+) or an injected clock, test a "debouncer" that calls a function only after 500 ms without new events.

## Chapter 7 Project — `crawl`, a concurrent web crawler

**Goal:** a crawler that visits every page of one web site reachable from a start URL, with bounded parallel requests, a request-rate limit, a depth limit, cancellation (timeout and Ctrl+C), and no races or leaks — and a test suite that proves each property using a local test server.

### Specification

```
crawl [-depth 2] [-workers 4] [-rps 5] [-timeout 30s] URL
```

- Visit each URL at most once; follow only `http`/`https` links on the start URL's host.
- At most `-workers` requests in flight; at most `-rps` request starts per second.
- Report each page (depth, URL, number of links, or the error) as soon as it is fetched.
- Stop when everything reachable within `-depth` is visited, the timeout expires, or Ctrl+C is pressed; exit 1 if stopped early.
- Read at most 2 MB per page; time out each request after 10 s.

### Design

- `crawl.Crawl(ctx, fetcher, start, opts, report)` is a library function; the command is a thin wrapper.
- `Fetcher` is a consumer-defined interface (P-23) with an HTTP implementation.
- A **mutex-protected `seen` map** deduplicates URLs; a **buffered-channel semaphore** bounds in-flight requests; a **ticker** limits the rate; a **WaitGroup** knows when all work is done; the **context** stops everything.
- Each discovered page gets its own goroutine, which waits for a rate tick and a semaphore slot. Goroutines are cheap; requests are bounded.
- Tests use `httptest.NewServer` with a fake site: no network needed, fully deterministic.

### Reference solution — `crawl/crawl.go`

```go
// Package crawl visits the pages of one web site concurrently, with a
// bound on parallel requests, a request-rate limit, a depth limit, and
// cancellation through a context.
package crawl

import (
	"context"
	"errors"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"regexp"
	"sync"
	"time"
)

// Fetcher downloads a page. It is an interface so tests can use a fake.
type Fetcher interface {
	Fetch(ctx context.Context, url string) (body string, err error)
}

// Result describes one visited page.
type Result struct {
	URL   string
	Depth int
	Links int
	Err   error
}

type Options struct {
	Workers  int           // maximum requests in flight (default 4)
	MaxDepth int           // 0 = only the start page
	Interval time.Duration // minimum time between request starts (0 = no limit)
}

// Crawl visits start and pages linked from it on the same host, up to
// MaxDepth links away. It calls report for every page from several
// goroutines at once, so report must be safe for concurrent use.
// It returns when every reachable page was visited or ctx is done.
func Crawl(ctx context.Context, f Fetcher, start string, opt Options, report func(Result)) error {
	base, err := url.Parse(start)
	if err != nil {
		return fmt.Errorf("start URL: %w", err)
	}
	if opt.Workers < 1 {
		opt.Workers = 4
	}

	var (
		mu   sync.Mutex
		seen = map[string]bool{start: true}
		wg   sync.WaitGroup
		sem  = make(chan struct{}, opt.Workers) // counting semaphore
	)
	var tick <-chan time.Time
	if opt.Interval > 0 {
		t := time.NewTicker(opt.Interval)
		defer t.Stop()
		tick = t.C
	}

	var visit func(u string, depth int)
	visit = func(u string, depth int) {
		defer wg.Done()

		// Wait for a rate-limit slot and a worker slot, or give up on cancel.
		if tick != nil {
			select {
			case <-tick:
			case <-ctx.Done():
				return
			}
		}
		select {
		case sem <- struct{}{}:
		case <-ctx.Done():
			return
		}
		body, err := f.Fetch(ctx, u)
		<-sem // release the worker slot before spawning children

		if err != nil {
			report(Result{URL: u, Depth: depth, Err: err})
			return
		}
		links := sameHostLinks(base, u, body)
		report(Result{URL: u, Depth: depth, Links: len(links)})
		if depth == opt.MaxDepth {
			return
		}
		for _, l := range links {
			mu.Lock()
			isNew := !seen[l]
			seen[l] = true
			mu.Unlock()
			if isNew {
				wg.Add(1)
				go visit(l, depth+1)
			}
		}
	}

	wg.Add(1)
	go visit(start, 0)
	wg.Wait()
	return ctx.Err()
}

// hrefRE is a deliberately simple link finder. A real crawler should use
// an HTML parser (golang.org/x/net/html); regular expressions cannot
// parse HTML in general.
var hrefRE = regexp.MustCompile(`href="([^"#]+)`)

func sameHostLinks(base *url.URL, page, body string) []string {
	pu, err := url.Parse(page)
	if err != nil {
		return nil
	}
	var out []string
	for _, m := range hrefRE.FindAllStringSubmatch(body, -1) {
		ref, err := url.Parse(m[1])
		if err != nil {
			continue
		}
		abs := pu.ResolveReference(ref)
		if abs.Host == base.Host && (abs.Scheme == "http" || abs.Scheme == "https") {
			abs.Fragment = ""
			out = append(out, abs.String())
		}
	}
	return out
}

// HTTPFetcher fetches pages over HTTP with a size limit.
type HTTPFetcher struct {
	Client   *http.Client
	MaxBytes int64
}

var ErrStatus = errors.New("unexpected HTTP status")

func (h HTTPFetcher) Fetch(ctx context.Context, u string) (string, error) {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, u, nil)
	if err != nil {
		return "", err
	}
	resp, err := h.Client.Do(req)
	if err != nil {
		return "", err
	}
	defer resp.Body.Close()
	if resp.StatusCode != http.StatusOK {
		return "", fmt.Errorf("%w: %s", ErrStatus, resp.Status)
	}
	b, err := io.ReadAll(io.LimitReader(resp.Body, h.MaxBytes))
	return string(b), err
}
```

Why release the semaphore **before** spawning children? If a goroutine held its slot while waiting for its children to get slots, a deep enough site would deadlock with every slot held by a parent waiting for children. Holding resources only for the operation that needs them is the general lesson.

### Reference solution — `crawl/crawl_test.go`

```go
package crawl

import (
	"context"
	"errors"
	"fmt"
	"net/http"
	"net/http/httptest"
	"slices"
	"sync"
	"sync/atomic"
	"testing"
	"time"
)

// A small site with a cycle, a broken link, and an external link.
var site = map[string]string{
	"/":         `<a href="/a">a</a> <a href="/b">b</a> <a href="https://other.example/x">ext</a>`,
	"/a":        `<a href="/">home</a> <a href="/a/deep">deep</a>`,
	"/b":        `<a href="/missing">broken</a> <a href="/a">a</a>`,
	"/a/deep":   `<a href="/a/deeper">deeper</a>`,
	"/a/deeper": `the end`,
}

func newSite(t *testing.T, delay time.Duration, inFlight, maxInFlight *atomic.Int64) *httptest.Server {
	t.Helper()
	srv := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		n := inFlight.Add(1)
		defer inFlight.Add(-1)
		for {
			m := maxInFlight.Load()
			if n <= m || maxInFlight.CompareAndSwap(m, n) {
				break
			}
		}
		select {
		case <-time.After(delay):
		case <-r.Context().Done():
			return
		}
		body, ok := site[r.URL.Path]
		if !ok {
			http.NotFound(w, r)
			return
		}
		fmt.Fprint(w, body)
	}))
	t.Cleanup(srv.Close)
	return srv
}

func collect() (func(Result), func() []Result) {
	var mu sync.Mutex
	var rs []Result
	return func(r Result) {
			mu.Lock()
			rs = append(rs, r)
			mu.Unlock()
		}, func() []Result {
			mu.Lock()
			defer mu.Unlock()
			slices.SortFunc(rs, func(a, b Result) int {
				if a.URL < b.URL {
					return -1
				}
				if a.URL > b.URL {
					return 1
				}
				return 0
			})
			return rs
		}
}

func TestCrawlVisitsEachPageOnce(t *testing.T) {
	var inFlight, maxInFlight atomic.Int64
	srv := newSite(t, 20*time.Millisecond, &inFlight, &maxInFlight)
	report, results := collect()
	f := HTTPFetcher{Client: srv.Client(), MaxBytes: 1 << 20}
	err := Crawl(context.Background(), f, srv.URL+"/", Options{Workers: 2, MaxDepth: 5}, report)
	if err != nil {
		t.Fatal(err)
	}
	rs := results()
	var got []string
	errCount := 0
	for _, r := range rs {
		got = append(got, r.URL[len(srv.URL):])
		if r.Err != nil {
			errCount++
			if !errors.Is(r.Err, ErrStatus) {
				t.Errorf("%s: unexpected error %v", r.URL, r.Err)
			}
		}
	}
	want := []string{"/", "/a", "/a/deep", "/a/deeper", "/b", "/missing"}
	if !slices.Equal(got, want) {
		t.Errorf("visited %v, want %v", got, want)
	}
	if errCount != 1 {
		t.Errorf("%d errors, want 1 (the 404)", errCount)
	}
	if m := maxInFlight.Load(); m > 2 {
		t.Errorf("%d requests in flight, limit was 2", m)
	}
}

func TestCrawlDepthLimit(t *testing.T) {
	var a, b atomic.Int64
	srv := newSite(t, 0, &a, &b)
	report, results := collect()
	f := HTTPFetcher{Client: srv.Client(), MaxBytes: 1 << 20}
	if err := Crawl(context.Background(), f, srv.URL+"/", Options{MaxDepth: 1}, report); err != nil {
		t.Fatal(err)
	}
	for _, r := range results() {
		if r.Depth > 1 {
			t.Errorf("%s visited at depth %d", r.URL, r.Depth)
		}
	}
	if n := len(results()); n != 3 { // /, /a, /b
		t.Errorf("visited %d pages, want 3", n)
	}
}

func TestCrawlCancel(t *testing.T) {
	var a, b atomic.Int64
	srv := newSite(t, 2*time.Second, &a, &b) // every page is very slow
	ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
	defer cancel()
	f := HTTPFetcher{Client: srv.Client(), MaxBytes: 1 << 20}
	start := time.Now()
	err := Crawl(ctx, f, srv.URL+"/", Options{MaxDepth: 5}, func(Result) {})
	if !errors.Is(err, context.DeadlineExceeded) {
		t.Errorf("err = %v, want DeadlineExceeded", err)
	}
	if d := time.Since(start); d > time.Second {
		t.Errorf("Crawl took %v after cancellation", d)
	}
}

func TestRateLimit(t *testing.T) {
	var a, b atomic.Int64
	srv := newSite(t, 0, &a, &b)
	f := HTTPFetcher{Client: srv.Client(), MaxBytes: 1 << 20}
	start := time.Now()
	err := Crawl(context.Background(), f, srv.URL+"/", Options{Workers: 8, MaxDepth: 5, Interval: 50 * time.Millisecond}, func(Result) {})
	if err != nil {
		t.Fatal(err)
	}
	// 6 requests, at most one every 50ms: the first waits one tick too.
	if d := time.Since(start); d < 250*time.Millisecond {
		t.Errorf("6 requests finished in %v; rate limit not applied", d)
	}
}
```

The test server records the **maximum number of concurrent requests** with an atomic compare-and-swap loop: the bound is *proved*, not assumed. The whole suite passes under `go test -race -count=1`.

### Reference solution — `cmd/crawl/main.go`

```go
// Command crawl prints the pages of a web site reachable from a start URL.
//
//	crawl [-depth 2] [-workers 4] [-rps 5] [-timeout 30s] URL
package main

import (
	"context"
	"flag"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"sync"
	"time"

	"example.com/crawler/crawl"
)

func main() {
	depth := flag.Int("depth", 2, "maximum link depth")
	workers := flag.Int("workers", 4, "maximum parallel requests")
	rps := flag.Float64("rps", 5, "maximum requests per second (0 = unlimited)")
	timeout := flag.Duration("timeout", 30*time.Second, "overall time limit")
	flag.Parse()
	if flag.NArg() != 1 {
		fmt.Fprintln(os.Stderr, "usage: crawl [flags] URL")
		os.Exit(2)
	}

	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
	defer stop()
	ctx, cancel := context.WithTimeout(ctx, *timeout)
	defer cancel()

	var interval time.Duration
	if *rps > 0 {
		interval = time.Duration(float64(time.Second) / *rps)
	}
	f := crawl.HTTPFetcher{
		Client:   &http.Client{Timeout: 10 * time.Second},
		MaxBytes: 2 << 20,
	}
	var mu sync.Mutex // report is called concurrently
	n := 0
	err := crawl.Crawl(ctx, f, flag.Arg(0), crawl.Options{Workers: *workers, MaxDepth: *depth, Interval: interval},
		func(r crawl.Result) {
			mu.Lock()
			defer mu.Unlock()
			n++
			if r.Err != nil {
				fmt.Printf("%d\t%s\tERROR %v\n", r.Depth, r.URL, r.Err)
				return
			}
			fmt.Printf("%d\t%s\t%d links\n", r.Depth, r.URL, r.Links)
		})
	fmt.Fprintf(os.Stderr, "%d pages\n", n)
	if err != nil {
		fmt.Fprintln(os.Stderr, "crawl stopped:", err)
		os.Exit(1)
	}
}
```

Note `time.Duration(float64(time.Second) / *rps)` for a fractional rate (DT-201).

### Review checklist

- Run `go test -race -count=100 ./...`. Any failure, even once, is a bug.
- For every goroutine: what makes it return on success, on error, on cancel?
- Is `report` documented as called concurrently? Does the command's callback lock?
- Does cancellation stop *waiting* goroutines (rate tick, semaphore) as well as in-flight requests?
- **Known limitation to discuss:** goroutines are bounded per *request*, not per *discovered URL*: a site with a million links spawns a million waiting goroutines (DT-238). For huge sites, replace "goroutine per URL" with a fixed pool of workers reading from a queue — and then think hard about how the pool knows when the crawl is finished (the queue can be empty while workers are still about to discover more URLs).
- **Politeness:** real crawlers honour `robots.txt` and identify themselves with a `User-Agent`.

### Extensions

1. Replace the goroutine-per-URL design with a fixed worker pool and a "pending work" counter; keep all tests passing.
2. Use `golang.org/x/net/html` to extract links properly (anchors, `<base href>`, relative URLs).
3. Add per-host rate limits for crawling several hosts, with a `map[string]*rate.Limiter` guarded by a mutex.

## Answers to the Chapter 7 prediction questions

**7.2 Q2.** Any order of 0, 1, 2 (or fewer, if a goroutine has not run before the sleep ends — unlikely but possible). Without the sleep, usually nothing: `main` returns and the program exits before the goroutines run.

**7.3 Q2.** (a) `fatal error: all goroutines are asleep - deadlock!` on the second send (buffer full, no receiver) (b) `0 false` (c) blocks forever; as the only goroutine, the runtime reports a deadlock (d) `panic: close of closed channel`.

**7.5 Q2.** Deadlock: Go mutexes are not reentrant, so `load` waits forever for the lock `Get` already holds. (In a server the runtime will not notice: only a total deadlock is detected, Unit 7.8.)

**7.8 Q2.** With one URL, no leak (the only sender is received from). With five, four goroutines leak. Cheapest fix: `make(chan string, len(urls))`, so every sender can complete.

# Chapter 8 — Networking, HTTP & RPC

*Goal: build production-shaped network services.*

Go was designed at Google for writing network servers, and it shows: a TCP server is twenty lines, a production-quality HTTP server is in the standard library, and the goroutine-per-connection model (Chapter 7) makes concurrent servers straightforward. This chapter goes from packets and sockets up to HTTP APIs and RPC, always with the robustness habits that separate a demo from a service: timeouts everywhere, bounded inputs, graceful shutdown, and clear error contracts.

**What you will be able to do after this chapter**

- Explain IP, ports, TCP, UDP, DNS, and TLS well enough to debug real network problems.
- Write TCP servers and clients with `net`, including deadlines and graceful shutdown.
- Explain HTTP semantics (methods, status codes, headers, connection reuse, HTTP/2).
- Build HTTP servers with Go 1.22+ routing and middleware, and HTTP clients that do not leak.
- Design REST APIs (resources, versioning, pagination, idempotency, error formats).
- Test services with `httptest`, and know when to use gRPC.

## Topic 8A — Network fundamentals

## Unit 8.1 — How networks work

**One-line summary:** Data crosses networks in packets. **IP** addresses machines and routes packets (best effort, no guarantees); **ports** address programs on a machine; **TCP** adds a reliable, ordered byte stream on top of IP; **UDP** sends independent datagrams with no guarantees; **DNS** turns names into addresses; **TLS** encrypts and authenticates a TCP connection.

**Mental model — the postal system.** IP is the postal network: it moves envelopes between addresses but may lose, delay, or reorder them. A port is the flat number in a building. TCP is a registered-mail conversation: numbered letters, receipts, and resending of anything lost, so the reader gets every page in order. UDP is a postcard: cheap and fast, no receipt. DNS is the directory that turns "go.dev" into an address. TLS is a sealed, tamper-proof envelope with an ID check of the recipient.

### The layers you work with

| Layer | Examples | What it gives you |
| --- | --- | --- |
| Application | HTTP, gRPC, DNS, your chat protocol | meaning: requests, responses, messages |
| Security | TLS | confidentiality, integrity, server (and optionally client) identity |
| Transport | TCP, UDP (QUIC is built on UDP) | process-to-process delivery; TCP: reliable ordered stream |
| Network | IPv4, IPv6 | machine-to-machine routing |
| Link | Ethernet, Wi-Fi | one hop |

### TCP essentials

- **Connection setup:** the three-way handshake (SYN, SYN-ACK, ACK) costs one round trip before any data. Over a 100 ms link that is 100 ms per new connection — which is why HTTP reuses connections (Unit 8.3).
- **A byte stream, not messages.** If the client writes "hello" and "world" in two calls, the server may read "hellowor" and "ld". Your protocol must define message boundaries (newlines in the chat server, length prefixes in binary protocols, HTTP's headers and `Content-Length`).
- **Reliability:** lost segments are retransmitted; a connection can stall for seconds before an error appears. Without timeouts, a program can wait forever on a dead peer (Unit 8.2).
- **Flow and congestion control:** TCP slows down when the receiver or the network cannot keep up — backpressure at the network level.
- **Closing:** each side closes its direction (FIN). A reader sees EOF when the other side has closed for writing. A connection reset (RST) appears as `connection reset by peer`.

### UDP

No connection, no ordering, no retransmission; each datagram arrives whole or not at all. Used where speed matters more than completeness (DNS queries, video, games, metrics) and as the base for QUIC (HTTP/3).

### DNS

A name lookup asks a resolver, which asks the DNS hierarchy and caches answers for their TTL. Practical consequences: a lookup can take tens of milliseconds or fail; one name can have many addresses (load balancing, IPv4 and IPv6); results change over time (do not cache them forever in a long-running process). Go's resolver (`net.LookupHost`, used automatically by `net.Dial`) has a pure-Go implementation and can use the system's C resolver (DT-4 territory).

### TLS in one paragraph

TLS performs a handshake in which the server proves its identity with a **certificate** signed by a certificate authority the client trusts, both sides agree on keys, and from then on everything is encrypted and integrity-protected. TLS 1.3 needs one round trip for the handshake. In Go, `crypto/tls` implements it entirely in Go (no OpenSSL), and HTTPS clients verify certificates by default. Since Go 1.24, TLS key exchange includes post-quantum hybrid algorithms by default, and Go 1.27 adds ML-DSA signatures. Never set `InsecureSkipVerify: true` outside a throwaway test.

### Deep & Tricky

**DT-241. "It works on localhost" hides latency and loss.** *(Network reality)* Localhost has ~0.05 ms round trips and no packet loss. Production paths have 1–200 ms, occasional loss, and middleboxes that drop idle connections. Code that makes 50 sequential requests per page feels instant locally and takes seconds in production. Test with realistic latency (Linux `tc netem` can add delay and loss).

### Mastery check

1. **Explain.** Why must a TCP-based protocol define message boundaries? Give two ways to do it.
2. **Explain.** What does one extra round trip cost for a user 150 ms away, and how does connection reuse help?

## Unit 8.2 — Sockets with the net package

**One-line summary:** `net.Listen` and `Accept` give a server one `net.Conn` per client; `net.Dial` gives a client a `net.Conn`. A `net.Conn` is an `io.Reader` and `io.Writer` (Chapter 5) with deadlines. The standard server pattern is one goroutine per connection, with read and write deadlines so dead clients cannot hold resources forever.

### A TCP echo server

```go
ln, err := net.Listen("tcp", ":9000")
if err != nil {
	return err
}
defer ln.Close()
for {
	conn, err := ln.Accept()
	if err != nil {
		return err
	}
	go func() {
		defer conn.Close()
		io.Copy(conn, conn) // echo everything back until the client closes
	}()
}
```

And a client:

```go
conn, err := net.DialTimeout("tcp", "localhost:9000", 3*time.Second)
if err != nil {
	return err
}
defer conn.Close()
fmt.Fprintln(conn, "hello")
reply, err := bufio.NewReader(conn).ReadString('\n')
```

Use `":0"` as the address in tests to get any free port, then read it from `ln.Addr()`.

### Deadlines

```go
conn.SetReadDeadline(time.Now().Add(5 * time.Minute))  // an absolute time, not a duration
conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
```

A blocked `Read` or `Write` returns an error once the deadline passes (`errors.Is(err, os.ErrDeadlineExceeded)`). Deadlines are absolute, so reset them before each operation that should have its own budget (an "idle timeout" resets on every read). Without deadlines, a client that disappears without closing (network cable pulled, NAT entry expired) keeps its goroutine and file descriptor forever — a slow leak that eventually exhausts the server.

### Protocol framing

The chat server in this chapter's project uses **newline-delimited text** with a `bufio.Scanner` capped at 1 KB per line (an unbounded line is an attack, DT-191). Binary protocols usually send a **length prefix** (`binary.BigEndian.Uint32`, Unit 5.5) and then exactly that many bytes (`io.ReadFull`), with a maximum length check before allocating.

### Graceful shutdown of a TCP server

`Accept` blocks; to stop the loop, close the listener from another goroutine (that makes `Accept` return an error), then wait for connection goroutines to finish:

```go
go func() {
	<-ctx.Done()
	ln.Close() // unblocks Accept
}()
for {
	conn, err := ln.Accept()
	if err != nil {
		if ctx.Err() != nil {
			wg.Wait() // let active connections finish
			return nil
		}
		return err
	}
	wg.Add(1)
	go func() { defer wg.Done(); handle(ctx, conn) }()
}
```

### Deep & Tricky

**DT-242. A write that succeeded does not mean the peer received it.** *(TCP semantics)* `Write` returns when the data is in the kernel's send buffer. If the peer crashes, the error appears on a *later* write or read. Protocols that need confirmation must have application-level acknowledgements (HTTP responses, database commits).

**DT-243. Each connection uses a file descriptor, and the limit is finite.** *(OS resource)* Linux defaults to 1024 open files per process in many shells (`ulimit -n`). A server holding 10,000 connections must raise it (systemd `LimitNOFILE`, container settings). The error is `accept: too many open files`; well-written servers back off and retry instead of spinning (`net/http` sleeps briefly after such errors).

### Mastery check

1. **Explain.** Why are deadlines absolute times, and how do you implement an idle timeout with them?
2. **Write.** A length-prefixed message protocol: `WriteMsg(w io.Writer, b []byte) error` and `ReadMsg(r io.Reader, max int) ([]byte, error)`, with tests for truncated input and oversized messages.

## Topic 8B — HTTP

## Unit 8.3 — HTTP semantics

**One-line summary:** HTTP is a request–response protocol: a method on a resource path, headers, an optional body; a status code, headers, and a body back. Methods have defined meanings (safe, idempotent), status codes have defined classes, and connections are reused (keep-alive in HTTP/1.1, multiplexing in HTTP/2).

### A request and a response

```
POST /api/links HTTP/1.1
Host: sho.rt
Content-Type: application/json
Content-Length: 46

{"url": "https://go.dev/doc/", "code": "godocs"}
```

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/links/godocs
X-Request-Id: J2PWT5LTNERQ

{"code":"godocs","url":"https://go.dev/doc/","created_at":"2026-11-15T01:34:13.391124175Z","hits":0}
```

(A real exchange with this chapter's project.)

### Methods

| Method | Meaning | Safe (no side effects)? | Idempotent (repeat = same effect)? |
| --- | --- | --- | --- |
| GET | read a resource | yes | yes |
| HEAD | GET without the body | yes | yes |
| POST | create / perform an action | no | **no** |
| PUT | replace a resource entirely | no | yes |
| PATCH | modify part of a resource | no | not necessarily |
| DELETE | remove a resource | no | yes |
| OPTIONS | ask what is allowed (CORS preflight) | yes | yes |

**Idempotency** matters for retries (Unit 12.7): a client that times out can safely retry a PUT or DELETE, but retrying a POST may create a duplicate order — unless the API supports idempotency keys (Unit 8.6). Never change state in a GET: crawlers, link previewers, and prefetchers will call it.

### Status codes

| Class | Meaning | Common codes |
| --- | --- | --- |
| 2xx | success | 200 OK, 201 Created, 202 Accepted, 204 No Content |
| 3xx | redirection | 301 Moved Permanently, 302 Found, 304 Not Modified, 307/308 (keep the method) |
| 4xx | **client** error — do not retry unchanged | 400 Bad Request, 401 Unauthorized (not authenticated), 403 Forbidden (not allowed), 404 Not Found, 405 Method Not Allowed, 409 Conflict, 413 Content Too Large, 422 Unprocessable Content, 429 Too Many Requests |
| 5xx | **server** error — may succeed on retry | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout |

The 4xx/5xx split drives client behaviour (retry or not), alerting (5xx pages the on-call engineer; 4xx usually does not), and dashboards. Get it right.

### Headers you must know

`Content-Type`, `Content-Length`, `Location` (with 201 and 3xx), `Authorization`, `Cache-Control`, `ETag` / `If-None-Match` (conditional requests → 304), `Accept`, `User-Agent`, `Retry-After` (with 429 and 503), `X-Request-ID` (correlation, a de facto standard). Header names are case-insensitive; Go canonicalises them (`x-request-id` → `X-Request-Id`).

### Connections: keep-alive, HTTP/2, HTTP/3

- **HTTP/1.1** reuses a TCP (and TLS) connection for many requests one after another ("keep-alive"), saving the handshakes. But one connection serves one request at a time; browsers open several.
- **HTTP/2** multiplexes many concurrent requests over one connection as interleaved streams, compresses headers, and is binary. Go's `net/http` uses it automatically over TLS for both servers and clients.
- **HTTP/3** runs over QUIC (UDP), avoiding TCP's head-of-line blocking. Not in Go's standard library (third-party `quic-go` exists).

### Mastery check

1. **Explain.** Why is it dangerous for a GET endpoint to delete something?
2. **Predict.** Which status for: (a) malformed JSON (b) valid JSON but `quantity: -3` (c) the user is logged in but not allowed (d) the database is down (e) creating a link whose custom code exists (f) too many requests from one client?

## Unit 8.4 — HTTP servers

**One-line summary:** A handler is anything with `ServeHTTP(http.ResponseWriter, *http.Request)`. Since Go 1.22, `http.ServeMux` routes by method and path patterns with wildcards (`GET /api/links/{code}`). Middleware is a function that wraps a handler. Every request runs in its own goroutine with a context cancelled when the client goes away.

### Handlers

```go
type Handler interface {
	ServeHTTP(w http.ResponseWriter, r *http.Request)
}

// http.HandlerFunc turns any function with the right signature into a Handler
// (a method on a function type, Unit 3.3).
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
	name := r.URL.Query().Get("name")
	fmt.Fprintf(w, "hello, %s\n", name)
})
```

Writing to the `ResponseWriter`:

1. Set headers: `w.Header().Set("Content-Type", "application/json")`.
2. Write the status: `w.WriteHeader(201)` — optional, the first `Write` sends 200.
3. Write the body: `w.Write(...)`, `fmt.Fprintf(w, ...)`, `json.NewEncoder(w).Encode(v)`.

Headers set after the first `Write` (or `WriteHeader`) are silently ignored; a second `WriteHeader` logs `superfluous response.WriteHeader call`. Order matters.

### Routing with `ServeMux` (Go 1.22+)

```go
mux := http.NewServeMux()
mux.HandleFunc("POST /api/links", a.create)
mux.HandleFunc("GET /api/links/{code}", a.stats)       // r.PathValue("code")
mux.HandleFunc("GET /files/{path...}", serveFile)        // {name...} matches the rest of the path
mux.HandleFunc("GET /{$}", home)                         // {$}: exactly "/", not every path
mux.HandleFunc("GET /{code}", a.redirect)
```

- A method in the pattern restricts it (GET also matches HEAD). A request to a known path with the wrong method gets an automatic **405 Method Not Allowed** with an `Allow` header — the project's test checks this.
- When several patterns match, the **most specific** wins (`/api/links/{code}` beats `/{code}` for `/api/links/x`); patterns that conflict ambiguously cause a panic at registration, not a surprise at run time.
- Before Go 1.22, `ServeMux` had no methods or wildcards, which is why routers like `chi` and `gorilla/mux` became popular. For most services the standard mux is now enough.

### Middleware

Middleware wraps a handler to add behaviour before and/or after it — logging, request IDs, panic recovery, authentication, rate limiting, CORS, compression, metrics:

```go
func logRequests(log *slog.Logger, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		rec := &statusRecorder{ResponseWriter: w, status: http.StatusOK}
		next.ServeHTTP(rec, r)
		log.InfoContext(r.Context(), "request",
			"method", r.Method, "path", r.URL.Path, "status", rec.status,
			"duration", time.Since(start), "request_id", RequestID(r.Context()))
	})
}
```

`statusRecorder` embeds the `ResponseWriter` and overrides `WriteHeader` to remember the code (the decorator pattern, P-24). Chain middleware by nesting: `requestID(logRequests(log, recoverPanics(log, mux)))` — the outermost runs first.

### Request handling essentials

- **Body size:** `r.Body = http.MaxBytesReader(w, r.Body, 16<<10)` — reading more fails with an error (and the server closes the connection). Unbounded bodies are a denial-of-service hole (DT-191).
- **JSON input:** `json.NewDecoder(r.Body)` with `DisallowUnknownFields` for strict APIs (Unit 5.4); validate after decoding (P-30).
- **Context:** `r.Context()` is cancelled when the client disconnects or the server shuts down; pass it to every database and HTTP call the handler makes (P-33).
- **Concurrency:** handlers run concurrently. Shared state needs synchronisation (the project's store uses a `sync.RWMutex`).
- **Panics:** `net/http` recovers a panic in a handler, logs it, and closes the connection; recovery middleware lets you log it with context and return a proper 500. `panic(http.ErrAbortHandler)` deliberately aborts a response.
- **Errors:** map domain errors to status codes in **one** place (the project's `fail` function); never send internal error text to clients (it leaks implementation details and sometimes secrets).

### Deep & Tricky

**DT-244. `http.ListenAndServe` with no timeouts is vulnerable to slow clients.** *(Library defaults)* The zero `http.Server` has no read or write timeouts; a client can open connections and send headers one byte per minute ("Slowloris"), holding goroutines and descriptors indefinitely. Always construct an `http.Server` with at least `ReadHeaderTimeout`, and usually `ReadTimeout`, `WriteTimeout`, and `IdleTimeout` (the project's `main` shows a reasonable set). `go vet` does not catch this; reviewers must.

**DT-245. `WriteTimeout` covers the whole handler.** *(Library behaviour)* It is measured from the end of reading the request headers until the response is written, so a slow handler (or a long streaming response) gets cut off silently: the client sees a closed connection, the handler keeps running. For per-handler budgets, use `http.TimeoutHandler` or `http.NewResponseController(w).SetWriteDeadline` (Go 1.20) and context deadlines.

### Pattern

**P-34. One error-to-status mapping per service.** *(API practice)* Handlers return or pass domain errors to a single function that maps sentinel and typed errors to HTTP status codes and a uniform JSON error body, logs unexpected errors with the request ID, and returns a generic message for 500s. *Trade-off:* a little indirection; in exchange, consistent APIs and no leaked internals.

### Mastery check

1. **Explain.** In which order do `requestID(logRequests(log, recoverPanics(log, mux)))` run, and why must `requestID` be outermost?
2. **Predict.** A handler calls `w.Write([]byte("ok"))` and then `w.WriteHeader(500)`. What status does the client get?
3. **Write.** Middleware `requireJSON` that rejects POST requests without `Content-Type: application/json` with 415, and a test using `httptest.NewRecorder`.

## Unit 8.5 — HTTP clients

**One-line summary:** Use an `http.Client` with a **timeout**, create requests with a context, **always close response bodies**, check status codes yourself, and reuse one client so connections are pooled. The default client has no timeout.

### The correct shape

```go
var client = &http.Client{Timeout: 10 * time.Second} // one per program (or per downstream), reused

func getUser(ctx context.Context, id string) (User, error) {
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, baseURL+"/users/"+url.PathEscape(id), nil)
	if err != nil {
		return User{}, err
	}
	req.Header.Set("Accept", "application/json")
	resp, err := client.Do(req)
	if err != nil {
		return User{}, fmt.Errorf("get user %s: %w", id, err) // network error, timeout, cancellation
	}
	defer resp.Body.Close()                                      // ALWAYS, even on error statuses
	if resp.StatusCode != http.StatusOK {
		b, _ := io.ReadAll(io.LimitReader(resp.Body, 1024))
		return User{}, fmt.Errorf("get user %s: %s: %s", id, resp.Status, b)
	}
	var u User
	if err := json.NewDecoder(io.LimitReader(resp.Body, 1<<20)).Decode(&u); err != nil {
		return User{}, fmt.Errorf("decode user %s: %w", id, err)
	}
	return u, nil
}
```

The checklist this code satisfies:

- **Timeout.** `http.DefaultClient` and `http.Get` have **no** timeout; a hung server hangs your goroutine forever. Set `Client.Timeout` and pass a context with the caller's deadline.
- **Close the body.** An unclosed body leaks the connection (it cannot be reused or released). `defer resp.Body.Close()` right after the error check (P-13).
- **Status codes are not errors.** `client.Do` returns `err == nil` for a 404 or 500; check `resp.StatusCode`.
- **Bounded reads.** A misbehaving server can send gigabytes (DT-191).
- **Escaping.** `url.PathEscape` and `url.Values{...}.Encode()` for query strings; never concatenate raw user input into URLs.

### Connection reuse

`http.Client` uses an `http.Transport` that keeps idle connections per host and reuses them. Two conditions for reuse: the body was read to the end and closed. (Go 1.27's HTTP/1 transport drains unread content on `Close` automatically up to a limit, which fixes many accidental non-reuse cases; older code drains with `io.Copy(io.Discard, resp.Body)`.) Tune `Transport.MaxIdleConnsPerHost` (default 2!) for services that call one backend heavily — otherwise most requests open new connections, each paying TCP and TLS handshakes.

### Deep & Tricky

**DT-246. Creating a new `http.Client` (or `Transport`) per request defeats pooling.** *(Library behaviour)* Each new `Transport` has its own empty pool, so every request does a full TCP+TLS handshake, and old idle connections accumulate until the Transport is garbage-collected. Create clients once and share them; they are safe for concurrent use.

**DT-247. `Client.Timeout` includes reading the body.** *(Library behaviour)* The timeout covers the whole exchange, including your `io.ReadAll` of a large body. A large download that legitimately takes minutes needs a longer timeout, or a context deadline per phase, or streaming with progress-based deadlines.

### Mastery check

1. **Explain.** List the five mistakes the `getUser` example avoids.
2. **Write.** A client for the shortener API (`Shorten`, `Stats`) with typed errors (`ErrNotFound`, `ErrConflict`) mapped from status codes, tested against the real handler via `httptest.NewServer`.

## Topic 8C — API design and robustness

## Unit 8.6 — REST API design

**One-line summary:** A good HTTP API models **resources** with nouns and uses methods for actions, returns accurate status codes, has a consistent error format, paginates collections, versions breaking changes, and makes non-idempotent operations safe to retry with idempotency keys.

### Resources and URLs

```
GET    /api/v1/links              list (paginated)
POST   /api/v1/links              create
GET    /api/v1/links/{code}       read one
PATCH  /api/v1/links/{code}       change some fields
DELETE /api/v1/links/{code}       delete
GET    /api/v1/links/{code}/hits  a sub-resource
```

Plural nouns, lower case, hyphens for multiple words, no verbs in paths (`POST /links`, not `/createLink`). Actions that are not CRUD can be modelled as sub-resources (`POST /orders/{id}/cancellation`).

### Consistent errors

Pick one format and use it for every error. The project uses `{"error": "...", "request_id": "..."}`. A standard alternative is RFC 9457 *Problem Details* (`application/problem+json` with `type`, `title`, `status`, `detail`). Include a request ID so support can find the logs; include field-level detail for validation errors; never include stack traces or SQL.

### Pagination

Never return an unbounded collection. Two styles:

- **Offset:** `?limit=50&offset=100`. Simple, allows jumping to page N, but slow for deep pages (the database still scans the skipped rows) and inconsistent when rows are inserted while paging.
- **Cursor (keyset):** `?limit=50&after=<opaque cursor>`, where the cursor encodes the last item's sort key (`created_at`, `id`). Fast at any depth and stable under inserts. Return `next_cursor` in the response. Preferred for large or changing collections (Unit 9.2 explains why it is fast).

Always enforce a maximum `limit`.

### Versioning and compatibility

Adding a field to a response or an optional field to a request is **compatible** (clients must ignore unknown fields — and your JSON decoding should be strict only where you control the clients). Removing or renaming fields, changing types or meanings, or tightening validation are **breaking**. Breaking changes need a new version (`/api/v2/...` or a header), a migration period, and communication. The JSON field names are your API (P-30).

### Idempotency keys

A mobile app sends `POST /payments`, the connection drops before the response, and the app retries. Did the payment happen once or twice? With an `Idempotency-Key: <uuid>` header, the server stores the key with the result of the first request; a retry with the same key returns the stored result instead of acting again. Unit 12.7 builds this; Stripe's API is the well-known example.

### Mastery check

1. **Explain.** Why is cursor pagination stable when new rows are inserted, and offset pagination is not?
2. **Explain.** Which of these are breaking: adding `expires_at` to responses; making `code` required; changing `hits` from a number to a string; adding a new optional query parameter?
3. **Write.** Design (on paper) the endpoints, request and response bodies, status codes, and errors for a "team todo list" API with users, lists, and items, including pagination and one idempotent operation.

## Unit 8.7 — Robust services

**One-line summary:** A robust service bounds everything (time, body size, concurrency), shuts down gracefully, propagates request contexts, and is tested end to end with `httptest` — including failure paths.

### Timeouts everywhere

| Where | How |
| --- | --- |
| Incoming connections | `http.Server` `ReadHeaderTimeout`, `ReadTimeout`, `WriteTimeout`, `IdleTimeout` (DT-244) |
| Each request's work | the request context, plus `context.WithTimeout` for budgets inside the handler |
| Outgoing HTTP | `Client.Timeout` and per-request contexts (Unit 8.5) |
| Database calls | context on every query (Unit 9.3) |
| Shutdown | a deadline for `Server.Shutdown` |

A request that waits forever on any dependency ties up a goroutine, a connection, and memory; under load those accumulate until the service falls over. Timeouts turn slow failures into fast, visible errors.

### Graceful shutdown

When Kubernetes (Unit 11.8) or a deploy script stops your service, it sends SIGTERM and waits a grace period before SIGKILL. In that window: stop accepting new connections, let in-flight requests finish, close idle connections, flush logs and metrics, close the database pool. `http.Server.Shutdown(ctx)` does the HTTP part:

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()

errCh := make(chan error, 1)
go func() { errCh <- srv.ListenAndServe() }()

select {
case err := <-errCh:
	return err // could not start (port in use, ...)
case <-ctx.Done():
}
shutdownCtx, cancel := context.WithTimeout(context.Background(), 15*time.Second)
defer cancel()
if err := srv.Shutdown(shutdownCtx); err != nil { // stop accepting; wait for in-flight requests
	return err
}
if err := <-errCh; !errors.Is(err, http.ErrServerClosed) {
	return err
}
```

Note the fresh `context.Background()` for the shutdown deadline: the signal context is already cancelled. `ListenAndServe` returns `http.ErrServerClosed` after `Shutdown` — that is the normal case, not an error. Long-lived connections (WebSockets, streaming) are not tracked by `Shutdown`; register `srv.RegisterOnShutdown` callbacks to tell them to finish.

### Testing with `httptest`

Two tools:

- `httptest.NewRecorder()` — a fake `ResponseWriter` for calling one handler directly: fast unit tests of handlers and middleware.
- `httptest.NewServer(handler)` — a real HTTP server on a random local port: end-to-end tests through the real client stack, middleware, routing, and JSON. The project's tests use it.

```go
rec := httptest.NewRecorder()
req := httptest.NewRequest(http.MethodGet, "/api/links/nope", nil)
handler.ServeHTTP(rec, req)
if rec.Code != http.StatusNotFound { ... }
```

Test failure paths as thoroughly as success: invalid JSON, unknown fields, too-large bodies, wrong methods, conflicts, timeouts of dependencies (with stubbed stores that block until the context is cancelled). Go 1.27's `httptest.NewTestServer` runs on an in-memory network that works with `testing/synctest` for time-dependent tests (Unit 7.12).

### Deep & Tricky

**DT-248. Shutdown and readiness must be coordinated with the load balancer.** *(Operations)* If the service stops accepting connections the instant it receives SIGTERM, but the load balancer still sends it traffic for a few seconds (its health check has not noticed yet), users get errors. Production services first fail their readiness check, wait a few seconds, then call `Shutdown` (Unit 11.8).

### Mastery check

1. **Explain.** Why does the shutdown code use `context.Background()` rather than the signal context for the shutdown deadline?
2. **Write.** A test proving that a handler whose store blocks until the context is cancelled returns promptly when the client gives up (use a client with a 50 ms timeout and a stub store).

## Topic 8D — RPC

## Unit 8.8 — Protocol Buffers and gRPC

**One-line summary:** gRPC is an RPC framework: you define services and messages in a `.proto` file, generate Go code, and call remote methods like local functions over HTTP/2, with binary Protocol Buffers encoding, deadlines, streaming, and interceptors. It shines for internal service-to-service communication; REST/JSON remains better for public and browser-facing APIs.

### Defining a service

```proto
syntax = "proto3";
package shortener.v1;
option go_package = "example.com/shortener/gen/shortenerv1";

service Shortener {
  rpc Shorten(ShortenRequest) returns (Link);
  rpc GetStats(GetStatsRequest) returns (Link);
  rpc WatchHits(WatchHitsRequest) returns (stream HitEvent); // server streaming
}

message ShortenRequest {
  string url = 1;   // field numbers identify fields on the wire: never reuse or change them
  string code = 2;
}

message Link {
  string code = 1;
  string url = 2;
  int64 hits = 3;
  google.protobuf.Timestamp created_at = 4;
}
```

`protoc` (or `buf`) with the `protoc-gen-go` and `protoc-gen-go-grpc` plugins generates message structs and client/server interfaces.

### Server and client (requires `google.golang.org/grpc`; shown, not compiled for this book)

```go
type server struct {
	shortenerv1.UnimplementedShortenerServer // forward compatibility: new methods return Unimplemented
	svc *links.Service
}

func (s *server) Shorten(ctx context.Context, req *shortenerv1.ShortenRequest) (*shortenerv1.Link, error) {
	l, err := s.svc.Shorten(ctx, req.GetUrl(), req.GetCode())
	switch {
	case errors.Is(err, links.ErrCodeTaken):
		return nil, status.Error(codes.AlreadyExists, "code already in use")
	case err != nil:
		return nil, toStatus(err) // map domain errors to gRPC codes, like P-34
	}
	return &shortenerv1.Link{Code: l.Code, Url: l.URL, Hits: l.Hits}, nil
}

// client
conn, err := grpc.NewClient("shortener:9090", grpc.WithTransportCredentials(creds))
c := shortenerv1.NewShortenerClient(conn)
ctx, cancel := context.WithTimeout(ctx, 2*time.Second) // the deadline travels to the server
defer cancel()
link, err := c.Shorten(ctx, &shortenerv1.ShortenRequest{Url: "https://go.dev"})
```

### What gRPC gives you

- **Deadlines propagate:** the client's context deadline is sent to the server, whose handler context expires at the same moment, and onward to calls it makes. End-to-end timeouts for free (P-33).
- **Streaming:** client streaming, server streaming, and bidirectional streams over one HTTP/2 connection.
- **Interceptors:** gRPC's middleware (unary and stream), for logging, auth, metrics, retries.
- **Status codes:** `codes.NotFound`, `InvalidArgument`, `AlreadyExists`, `DeadlineExceeded`, `Unavailable` (retryable), and so on — analogous to HTTP status classes.
- **Schema evolution:** add fields with new numbers; never reuse numbers; old clients ignore new fields.

### When to choose what

| | REST + JSON | gRPC |
| --- | --- | --- |
| Clients | browsers, third parties, curl | your own services |
| Schema | OpenAPI (optional) | `.proto` (required, generated code) |
| Encoding | text, human-readable | binary, compact, fast |
| Streaming | limited (SSE, WebSockets) | built in |
| Tooling to debug | universal | needs `grpcurl` and similar |

### Deep & Tricky

**DT-249. Protobuf zero values are indistinguishable from "unset" for scalars in proto3.** *(Encoding rule)* `hits = 0` and "hits not sent" decode identically unless the field is declared `optional` (which generates a pointer field and `Has` methods). Same problem as JSON's `omitempty` (DT-156) — design messages so zero is a valid default or use `optional`.

### Mastery check

1. **Explain.** How does a gRPC deadline propagate across three services, and why is that valuable?
2. **Explain.** Why must a protobuf field number never be reused?

## Chapter 8 Projects — A TCP chat server, then a URL-shortener API

This chapter has two projects: a TCP chat server (sockets, protocol framing, deadlines, goroutine ownership, graceful shutdown) and the first version of the **URL shortener** service, which Chapters 9 and 11 extend into a database-backed, production-ready system.

### Project A — `chat`, a TCP chat server

**Specification**

- Listen on `-addr` (default `localhost:9000`); clients connect with `nc localhost 9000`.
- The first line a client sends is its name. Every later line is broadcast to all other clients as `name: text`. `/who` lists users; `/quit` leaves. Joins and leaves are announced.
- Lines are capped at 1 KB; a client idle for 5 minutes is disconnected.
- A slow client (whose outgoing queue of 32 messages is full) is disconnected instead of slowing everyone else down.
- Ctrl+C shuts down gracefully: stop accepting, close all clients, return.

**Design: one owner per piece of state.** The **hub** goroutine owns the set of clients (no mutex: confinement, P-32), and is the *only* goroutine that sends on or closes a client's outgoing channel ("only the sender closes", Unit 7.3). Each connection has a **reader** (the connection's goroutine) and a **writer** goroutine (the only one that writes to the socket). A note from development: the first version of this design let the reader goroutine send the welcome message and `/who` reply directly on the client's channel. The tests passed — but during shutdown the hub could close that channel at the same moment, which panics with `send on closed channel`. Moving *all* sends to the hub fixed it. Rare interleavings are exactly what tests miss and ownership rules prevent.

```go
// Command chat is a line-based TCP chat server. Connect with
// "nc localhost 9000"; the first line you send is your name.
// Commands: /who lists users, /quit leaves.
package main

import (
	"bufio"
	"context"
	"errors"
	"flag"
	"fmt"
	"log/slog"
	"net"
	"os"
	"os/signal"
	"slices"
	"strings"
	"sync"
	"time"
)

const (
	idleTimeout = 5 * time.Minute
	maxLine     = 1024
	outBuffer   = 32 // messages queued per client before it is considered too slow
)

type client struct {
	name string
	out  chan string // the hub sends; the client's writer goroutine receives
}

type message struct {
	from *client
	text string
}

// Hub owns the set of clients. Only the hub's goroutine touches the map,
// and only the hub sends on or closes a client's out channel: state
// confinement (P-32) and "only the sender closes" (Unit 7.3).
type Hub struct {
	join  chan *client
	leave chan *client
	msgs  chan message
	who   chan *client
}

func NewHub() *Hub {
	return &Hub{
		join:  make(chan *client),
		leave: make(chan *client),
		msgs:  make(chan message),
		who:   make(chan *client),
	}
}

func (h *Hub) Run(ctx context.Context) {
	clients := map[*client]bool{}
	send := func(c *client, text string) {
		select {
		case c.out <- text:
		default: // slow client: drop it rather than block everyone (backpressure)
			delete(clients, c)
			close(c.out)
		}
	}
	broadcast := func(text string, except *client) {
		for c := range clients {
			if c != except {
				send(c, text)
			}
		}
	}
	for {
		select {
		case c := <-h.join:
			clients[c] = true
			send(c, "welcome, "+c.name+" (/who, /quit)")
			broadcast(c.name+" joined", c)
		case c := <-h.leave:
			if clients[c] {
				delete(clients, c)
				close(c.out)
				broadcast(c.name+" left", nil)
			}
		case m := <-h.msgs:
			broadcast(m.from.name+": "+m.text, m.from)
		case c := <-h.who:
			if !clients[c] {
				continue
			}
			var names []string
			for other := range clients {
				names = append(names, other.name)
			}
			slices.Sort(names)
			send(c, "online: "+strings.Join(names, ", "))
		case <-ctx.Done():
			for c := range clients {
				close(c.out)
			}
			return
		}
	}
}

// Serve accepts connections until ctx is done, then waits for them to end.
func Serve(ctx context.Context, ln net.Listener, h *Hub, log *slog.Logger) error {
	var wg sync.WaitGroup
	go func() {
		<-ctx.Done()
		ln.Close() // unblocks Accept
	}()
	for {
		conn, err := ln.Accept()
		if err != nil {
			if ctx.Err() != nil {
				wg.Wait()
				return nil
			}
			return err
		}
		wg.Add(1)
		go func() {
			defer wg.Done()
			handle(ctx, conn, h, log)
		}()
	}
}

func handle(ctx context.Context, conn net.Conn, h *Hub, log *slog.Logger) {
	defer conn.Close()
	log.Info("connected", "remote", conn.RemoteAddr())
	sc := bufio.NewScanner(conn)
	sc.Buffer(make([]byte, 0, maxLine), maxLine)

	readLine := func() (string, bool) {
		conn.SetReadDeadline(time.Now().Add(idleTimeout))
		if !sc.Scan() {
			return "", false
		}
		return strings.TrimSpace(sc.Text()), true
	}

	fmt.Fprint(conn, "name: ")
	name, ok := readLine()
	if !ok || name == "" {
		return
	}
	c := &client{name: name, out: make(chan string, outBuffer)}

	select {
	case h.join <- c: // from now on the hub owns c.out and will close it
	case <-ctx.Done():
		return
	}

	// Writer goroutine: the only goroutine that writes to conn from now on.
	// It ends when the hub closes c.out (after /quit, on shutdown, or
	// because this client was too slow).
	writerDone := make(chan struct{})
	go func() {
		defer close(writerDone)
		for line := range c.out {
			conn.SetWriteDeadline(time.Now().Add(10 * time.Second))
			if _, err := fmt.Fprintln(conn, line); err != nil {
				break
			}
		}
		conn.Close() // also unblocks the reader below
		for range c.out {
			// drain until the hub closes the channel, so the hub never blocks
		}
	}()

	for {
		line, ok := readLine()
		if !ok || line == "/quit" {
			break
		}
		switch {
		case line == "":
		case line == "/who":
			select {
			case h.who <- c:
			case <-ctx.Done():
			}
		default:
			select {
			case h.msgs <- message{c, line}:
			case <-ctx.Done():
			}
		}
		if ctx.Err() != nil {
			break
		}
	}
	select {
	case h.leave <- c:
	case <-ctx.Done():
	}
	<-writerDone
	if err := sc.Err(); err != nil && !errors.Is(err, net.ErrClosed) {
		log.Info("read error", "name", name, "err", err)
	}
	log.Info("disconnected", "name", name)
}

func main() {
	addr := flag.String("addr", "localhost:9000", "listen address")
	flag.Parse()
	log := slog.New(slog.NewTextHandler(os.Stderr, nil))

	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt)
	defer stop()
	ln, err := net.Listen("tcp", *addr)
	if err != nil {
		log.Error("listen", "err", err)
		os.Exit(1)
	}
	log.Info("listening", "addr", ln.Addr())
	h := NewHub()
	go h.Run(ctx)
	if err := Serve(ctx, ln, h, log); err != nil {
		log.Error("serve", "err", err)
		os.Exit(1)
	}
}
```

**Test (`chat_test.go`)** — real TCP connections on a random port, then a graceful-shutdown check:

```go
package main

import (
	"bufio"
	"context"
	"io"
	"log/slog"
	"net"
	"strings"
	"testing"
	"time"
)

type testClient struct {
	conn net.Conn
	r    *bufio.Reader
}

func dial(t *testing.T, addr, name string) *testClient {
	t.Helper()
	conn, err := net.Dial("tcp", addr)
	if err != nil {
		t.Fatal(err)
	}
	t.Cleanup(func() { conn.Close() })
	c := &testClient{conn: conn, r: bufio.NewReader(conn)}
	c.expect(t, "name: ")
	c.send(t, name)
	c.expectLine(t, "welcome, "+name)
	return c
}

func (c *testClient) send(t *testing.T, s string) {
	t.Helper()
	if _, err := io.WriteString(c.conn, s+"\n"); err != nil {
		t.Fatal(err)
	}
}

func (c *testClient) expect(t *testing.T, prefix string) {
	t.Helper()
	c.conn.SetReadDeadline(time.Now().Add(2 * time.Second))
	buf := make([]byte, len(prefix))
	if _, err := io.ReadFull(c.r, buf); err != nil || string(buf) != prefix {
		t.Fatalf("got %q, %v; want %q", buf, err, prefix)
	}
}

func (c *testClient) expectLine(t *testing.T, prefix string) {
	t.Helper()
	c.conn.SetReadDeadline(time.Now().Add(2 * time.Second))
	line, err := c.r.ReadString('\n')
	if err != nil || !strings.HasPrefix(line, prefix) {
		t.Fatalf("got %q, %v; want prefix %q", line, err, prefix)
	}
}

func start(t *testing.T) (string, context.CancelFunc, chan error) {
	t.Helper()
	ln, err := net.Listen("tcp", "127.0.0.1:0") // port 0: any free port
	if err != nil {
		t.Fatal(err)
	}
	ctx, cancel := context.WithCancel(context.Background())
	h := NewHub()
	go h.Run(ctx)
	done := make(chan error, 1)
	log := slog.New(slog.NewTextHandler(io.Discard, nil))
	go func() { done <- Serve(ctx, ln, h, log) }()
	return ln.Addr().String(), cancel, done
}

func TestChat(t *testing.T) {
	addr, cancel, done := start(t)
	defer cancel()

	asha := dial(t, addr, "asha")
	ravi := dial(t, addr, "ravi")
	asha.expectLine(t, "ravi joined")

	asha.send(t, "namaste")
	ravi.expectLine(t, "asha: namaste")

	ravi.send(t, "/who")
	ravi.expectLine(t, "online: asha, ravi")

	ravi.send(t, "/quit")
	asha.expectLine(t, "ravi left")

	cancel() // graceful shutdown: Serve must return
	select {
	case err := <-done:
		if err != nil {
			t.Fatal(err)
		}
	case <-time.After(2 * time.Second):
		t.Fatal("Serve did not stop after cancel")
	}
}
```

It passes `go test -race -count=20`.

### Project B — `shortener`, a URL-shortener REST API

**Specification**

```
POST /api/links          {"url": "...", "code": "optional"}  -> 201 + link, Location header
GET  /api/links/{code}   -> 200 + link with hit count
GET  /{code}             -> 302 redirect to the target URL (counts a hit)
GET  /healthz            -> 200 "ok"
```

- URLs must be `http`/`https` with a host, at most 2048 bytes. Custom codes: 3–32 letters, digits, `-`, `_`. Generated codes: 7 characters from an alphabet without look-alike characters, from `crypto/rand` (guessable codes would let anyone enumerate links), with retry on collision.
- Errors: 400 invalid input, 404 unknown code, 409 code taken, 405 wrong method, 500 internal (generic message, details logged). Every error body is `{"error": ..., "request_id": ...}`.
- Request bodies limited to 16 KB; unknown JSON fields rejected.
- Middleware: request ID (accepting a client-supplied `X-Request-ID`), structured access logs, panic recovery.
- Server timeouts and graceful shutdown on SIGINT/SIGTERM.
- Storage behind a `links.Store` interface with an in-memory implementation (Chapter 9 adds PostgreSQL).

**Layout**

```
shortener/
├── cmd/shortener/main.go        wiring, server, shutdown
└── internal/
    ├── links/                   domain: Link, Store, Service, errors
    │   ├── links.go
    │   ├── memory.go
    │   └── links_test.go
    └── httpapi/                 HTTP: routes, handlers, middleware, error mapping
        ├── httpapi.go
        └── httpapi_test.go
```

**`internal/links/links.go`**

```go
// Package links holds the URL shortener's rules: validating URLs,
// generating short codes, and resolving them. Storage is behind Store.
package links

import (
	"context"
	"crypto/rand"
	"errors"
	"fmt"
	"net/url"
	"regexp"
	"time"
)

type Link struct {
	Code      string    `json:"code"`
	URL       string    `json:"url"`
	CreatedAt time.Time `json:"created_at"`
	Hits      int64     `json:"hits"`
}

var (
	ErrNotFound  = errors.New("link not found")
	ErrCodeTaken = errors.New("code already in use")
)

// InvalidError reports bad input; the HTTP layer maps it to 400.
type InvalidError struct{ Reason string }

func (e *InvalidError) Error() string { return "invalid input: " + e.Reason }

// Store is what the service needs from storage.
type Store interface {
	// Create saves l, or returns ErrCodeTaken if l.Code exists.
	Create(ctx context.Context, l Link) error
	// Get returns the link, or ErrNotFound.
	Get(ctx context.Context, code string) (Link, error)
	// Hit increments the hit counter and returns the target URL, or ErrNotFound.
	Hit(ctx context.Context, code string) (string, error)
}

type Service struct {
	store Store
	now   func() time.Time
}

func NewService(s Store) *Service { return &Service{store: s, now: time.Now} }

var codeRE = regexp.MustCompile(`^[A-Za-z0-9_-]{3,32}$`)

const (
	alphabet  = "abcdefghijkmnpqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ23456789" // no 0/O, 1/l/I
	genLength = 7
	maxURLLen = 2048
)

// Shorten stores rawURL under code, or under a generated code if code is "".
func (s *Service) Shorten(ctx context.Context, rawURL, code string) (Link, error) {
	if err := validateURL(rawURL); err != nil {
		return Link{}, err
	}
	if code != "" {
		if !codeRE.MatchString(code) {
			return Link{}, &InvalidError{"code must be 3-32 letters, digits, '-' or '_'"}
		}
		l := Link{Code: code, URL: rawURL, CreatedAt: s.now().UTC()}
		return l, s.store.Create(ctx, l)
	}
	// Random codes can collide; with 56^7 ≈ 1.7e12 codes it is rare, but retry.
	for range 5 {
		l := Link{Code: randomCode(), URL: rawURL, CreatedAt: s.now().UTC()}
		err := s.store.Create(ctx, l)
		if errors.Is(err, ErrCodeTaken) {
			continue
		}
		return l, err
	}
	return Link{}, fmt.Errorf("could not find a free code: %w", ErrCodeTaken)
}

// Resolve returns the target URL for code and counts the hit.
func (s *Service) Resolve(ctx context.Context, code string) (string, error) {
	return s.store.Hit(ctx, code)
}

// Stats returns the link with its hit count.
func (s *Service) Stats(ctx context.Context, code string) (Link, error) {
	return s.store.Get(ctx, code)
}

func validateURL(raw string) error {
	if len(raw) > maxURLLen {
		return &InvalidError{fmt.Sprintf("url longer than %d bytes", maxURLLen)}
	}
	u, err := url.Parse(raw)
	if err != nil {
		return &InvalidError{"url does not parse"}
	}
	if u.Scheme != "http" && u.Scheme != "https" {
		return &InvalidError{"url must start with http:// or https://"}
	}
	if u.Host == "" {
		return &InvalidError{"url has no host"}
	}
	return nil
}

func randomCode() string {
	b := make([]byte, genLength)
	rand.Read(b) // crypto/rand: codes must not be guessable (P-15)
	for i := range b {
		b[i] = alphabet[int(b[i])%len(alphabet)] // tiny modulo bias (DT-96) is acceptable here
	}
	return string(b)
}
```

**`internal/links/memory.go`**

```go
package links

import (
	"context"
	"sync"
)

// Memory is an in-memory Store, safe for concurrent use.
type Memory struct {
	mu    sync.RWMutex
	links map[string]Link
}

var _ Store = (*Memory)(nil)

func NewMemory() *Memory { return &Memory{links: make(map[string]Link)} }

func (m *Memory) Create(_ context.Context, l Link) error {
	m.mu.Lock()
	defer m.mu.Unlock()
	if _, ok := m.links[l.Code]; ok {
		return ErrCodeTaken
	}
	m.links[l.Code] = l
	return nil
}

func (m *Memory) Get(_ context.Context, code string) (Link, error) {
	m.mu.RLock()
	defer m.mu.RUnlock()
	l, ok := m.links[code]
	if !ok {
		return Link{}, ErrNotFound
	}
	return l, nil
}

func (m *Memory) Hit(_ context.Context, code string) (string, error) {
	m.mu.Lock() // a write: the hit counter changes
	defer m.mu.Unlock()
	l, ok := m.links[code]
	if !ok {
		return "", ErrNotFound
	}
	l.Hits++
	m.links[code] = l
	return l.URL, nil
}
```

**`internal/httpapi/httpapi.go`**

```go
// Package httpapi exposes the shortener over HTTP.
//
//	POST /api/links          {"url": "...", "code": "optional"} -> 201 + link
//	GET  /api/links/{code}   -> 200 + link with hit count
//	GET  /{code}             -> 302 redirect to the target URL
//	GET  /healthz            -> 200 "ok"
package httpapi

import (
	"context"
	"crypto/rand"
	"encoding/json"
	"errors"
	"fmt"
	"log/slog"
	"net/http"
	"time"

	"example.com/shortener/internal/links"
)

const maxBodyBytes = 16 << 10

type API struct {
	svc *links.Service
	log *slog.Logger
}

// New returns the complete handler, with middleware applied.
func New(svc *links.Service, log *slog.Logger) http.Handler {
	a := &API{svc: svc, log: log}
	mux := http.NewServeMux()
	mux.HandleFunc("POST /api/links", a.create)
	mux.HandleFunc("GET /api/links/{code}", a.stats)
	mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, _ *http.Request) {
		w.Write([]byte("ok\n"))
	})
	mux.HandleFunc("GET /{code}", a.redirect)
	// Middleware is applied outside-in: requestID runs first.
	return requestID(logRequests(log, recoverPanics(log, mux)))
}

type createRequest struct {
	URL  string `json:"url"`
	Code string `json:"code"`
}

func (a *API) create(w http.ResponseWriter, r *http.Request) {
	r.Body = http.MaxBytesReader(w, r.Body, maxBodyBytes)
	dec := json.NewDecoder(r.Body)
	dec.DisallowUnknownFields()
	var req createRequest
	if err := dec.Decode(&req); err != nil {
		a.writeError(w, r, http.StatusBadRequest, "body must be JSON like {\"url\": \"https://...\"}: "+err.Error())
		return
	}
	l, err := a.svc.Shorten(r.Context(), req.URL, req.Code)
	if err != nil {
		a.fail(w, r, err)
		return
	}
	w.Header().Set("Location", "/api/links/"+l.Code)
	writeJSON(w, http.StatusCreated, l)
}

func (a *API) stats(w http.ResponseWriter, r *http.Request) {
	l, err := a.svc.Stats(r.Context(), r.PathValue("code"))
	if err != nil {
		a.fail(w, r, err)
		return
	}
	writeJSON(w, http.StatusOK, l)
}

func (a *API) redirect(w http.ResponseWriter, r *http.Request) {
	target, err := a.svc.Resolve(r.Context(), r.PathValue("code"))
	if err != nil {
		a.fail(w, r, err)
		return
	}
	http.Redirect(w, r, target, http.StatusFound)
}

// fail maps domain errors to HTTP status codes in one place.
func (a *API) fail(w http.ResponseWriter, r *http.Request, err error) {
	var inv *links.InvalidError
	switch {
	case errors.As(err, &inv):
		a.writeError(w, r, http.StatusBadRequest, inv.Reason)
	case errors.Is(err, links.ErrNotFound):
		a.writeError(w, r, http.StatusNotFound, "no such link")
	case errors.Is(err, links.ErrCodeTaken):
		a.writeError(w, r, http.StatusConflict, "code already in use")
	case errors.Is(err, context.Canceled):
		// the client went away; nobody will read a response
	default:
		a.log.ErrorContext(r.Context(), "internal error", "err", err, "request_id", RequestID(r.Context()))
		a.writeError(w, r, http.StatusInternalServerError, "internal error") // never leak details
	}
}

type errorBody struct {
	Error     string `json:"error"`
	RequestID string `json:"request_id"`
}

func (a *API) writeError(w http.ResponseWriter, r *http.Request, status int, msg string) {
	writeJSON(w, status, errorBody{Error: msg, RequestID: RequestID(r.Context())})
}

func writeJSON(w http.ResponseWriter, status int, v any) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	json.NewEncoder(w).Encode(v)
}

// ---- middleware ----

type ctxKey struct{}

// RequestID returns the request's ID, or "" outside a request.
func RequestID(ctx context.Context) string {
	id, _ := ctx.Value(ctxKey{}).(string)
	return id
}

func requestID(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		id := r.Header.Get("X-Request-ID")
		if id == "" || len(id) > 64 {
			id = rand.Text()[:12]
		}
		w.Header().Set("X-Request-ID", id)
		next.ServeHTTP(w, r.WithContext(context.WithValue(r.Context(), ctxKey{}, id)))
	})
}

// statusRecorder remembers the status code a handler wrote.
type statusRecorder struct {
	http.ResponseWriter
	status int
}

func (s *statusRecorder) WriteHeader(code int) {
	s.status = code
	s.ResponseWriter.WriteHeader(code)
}

func (s *statusRecorder) Unwrap() http.ResponseWriter { return s.ResponseWriter } // for http.ResponseController

func logRequests(log *slog.Logger, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		rec := &statusRecorder{ResponseWriter: w, status: http.StatusOK}
		next.ServeHTTP(rec, r)
		log.InfoContext(r.Context(), "request",
			"method", r.Method, "path", r.URL.Path, "status", rec.status,
			"duration", time.Since(start), "request_id", RequestID(r.Context()))
	})
}

func recoverPanics(log *slog.Logger, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		defer func() {
			if v := recover(); v != nil {
				if v == http.ErrAbortHandler { // deliberate abort: let net/http handle it
					panic(v)
				}
				log.ErrorContext(r.Context(), "panic", "value", fmt.Sprint(v), "request_id", RequestID(r.Context()))
				http.Error(w, "internal error", http.StatusInternalServerError)
			}
		}()
		next.ServeHTTP(w, r)
	})
}
```

**`cmd/shortener/main.go`**

```go
// Command shortener runs the URL shortener HTTP service.
package main

import (
	"context"
	"errors"
	"flag"
	"log/slog"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"example.com/shortener/internal/httpapi"
	"example.com/shortener/internal/links"
)

func main() {
	if err := run(); err != nil {
		slog.Error("shortener stopped", "err", err)
		os.Exit(1)
	}
}

func run() error {
	addr := flag.String("addr", ":8080", "listen address")
	flag.Parse()
	log := slog.New(slog.NewJSONHandler(os.Stderr, nil))

	srv := &http.Server{
		Addr:              *addr,
		Handler:           httpapi.New(links.NewService(links.NewMemory()), log),
		ReadHeaderTimeout: 5 * time.Second, // slow clients cannot hold connections forever
		ReadTimeout:       10 * time.Second,
		WriteTimeout:      10 * time.Second,
		IdleTimeout:       60 * time.Second,
		MaxHeaderBytes:    16 << 10,
	}

	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	errCh := make(chan error, 1)
	go func() {
		log.Info("listening", "addr", *addr)
		errCh <- srv.ListenAndServe()
	}()

	select {
	case err := <-errCh:
		return err // could not start (port in use, ...)
	case <-ctx.Done():
	}
	log.Info("shutting down")
	shutdownCtx, cancel := context.WithTimeout(context.Background(), 15*time.Second)
	defer cancel()
	if err := srv.Shutdown(shutdownCtx); err != nil { // stop accepting; wait for in-flight requests
		return err
	}
	if err := <-errCh; !errors.Is(err, http.ErrServerClosed) {
		return err
	}
	log.Info("stopped cleanly")
	return nil
}
```

**Tests — `internal/httpapi/httpapi_test.go`**

```go
package httpapi

import (
	"encoding/json"
	"io"
	"log/slog"
	"net/http"
	"net/http/httptest"
	"strings"
	"testing"

	"example.com/shortener/internal/links"
)

func newServer(t *testing.T) *httptest.Server {
	t.Helper()
	log := slog.New(slog.NewTextHandler(io.Discard, nil))
	srv := httptest.NewServer(New(links.NewService(links.NewMemory()), log))
	t.Cleanup(srv.Close)
	return srv
}

// noRedirect makes the client return 3xx responses instead of following them.
var noRedirect = &http.Client{CheckRedirect: func(*http.Request, []*http.Request) error {
	return http.ErrUseLastResponse
}}

func post(t *testing.T, srv *httptest.Server, body string) (*http.Response, map[string]any) {
	t.Helper()
	resp, err := http.Post(srv.URL+"/api/links", "application/json", strings.NewReader(body))
	if err != nil {
		t.Fatal(err)
	}
	defer resp.Body.Close()
	var m map[string]any
	json.NewDecoder(resp.Body).Decode(&m)
	return resp, m
}

func TestCreateRedirectStats(t *testing.T) {
	srv := newServer(t)
	resp, body := post(t, srv, `{"url": "https://go.dev/doc/", "code": "godocs"}`)
	if resp.StatusCode != http.StatusCreated || body["code"] != "godocs" {
		t.Fatalf("create: %d %v", resp.StatusCode, body)
	}
	if resp.Header.Get("X-Request-ID") == "" {
		t.Error("no request ID header")
	}

	for range 3 {
		r, err := noRedirect.Get(srv.URL + "/godocs")
		if err != nil {
			t.Fatal(err)
		}
		r.Body.Close()
		if r.StatusCode != http.StatusFound || r.Header.Get("Location") != "https://go.dev/doc/" {
			t.Fatalf("redirect: %d %q", r.StatusCode, r.Header.Get("Location"))
		}
	}

	r, err := http.Get(srv.URL + "/api/links/godocs")
	if err != nil {
		t.Fatal(err)
	}
	defer r.Body.Close()
	var l links.Link
	json.NewDecoder(r.Body).Decode(&l)
	if l.Hits != 3 {
		t.Errorf("hits = %d, want 3", l.Hits)
	}
}

func TestErrors(t *testing.T) {
	srv := newServer(t)
	post(t, srv, `{"url": "https://example.com", "code": "taken"}`)
	tests := []struct {
		name, body string
		want       int
	}{
		{"not json", `hello`, 400},
		{"unknown field", `{"url": "https://x.com", "colour": "red"}`, 400},
		{"bad scheme", `{"url": "ftp://x.com"}`, 400},
		{"no host", `{"url": "https://"}`, 400},
		{"bad code", `{"url": "https://x.com", "code": "a b"}`, 400},
		{"duplicate code", `{"url": "https://x.com", "code": "taken"}`, 409},
		{"too big", `{"url": "https://x.com/` + strings.Repeat("a", 20000) + `"}`, 400},
		{"generated code", `{"url": "https://x.com"}`, 201},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			resp, body := post(t, srv, tt.body)
			if resp.StatusCode != tt.want {
				t.Errorf("status %d, want %d (%v)", resp.StatusCode, tt.want, body)
			}
			if tt.want >= 400 && body["request_id"] == "" {
				t.Error("error body has no request_id")
			}
		})
	}
	r, _ := http.Get(srv.URL + "/nope")
	if r.StatusCode != 404 {
		t.Errorf("unknown code: %d", r.StatusCode)
	}
	req, _ := http.NewRequest(http.MethodDelete, srv.URL+"/api/links/taken", nil)
	r, _ = http.DefaultClient.Do(req)
	if r.StatusCode != http.StatusMethodNotAllowed {
		t.Errorf("DELETE: %d, want 405", r.StatusCode)
	}
}
```

**Tests — `internal/links/links_test.go`** (a stub store that collides, to test the retry loop)

```go
package links

import (
	"context"
	"errors"
	"testing"
)

// collidingStore reports ErrCodeTaken for the first n creates.
type collidingStore struct {
	*Memory
	n int
}

func (c *collidingStore) Create(ctx context.Context, l Link) error {
	if c.n > 0 {
		c.n--
		return ErrCodeTaken
	}
	return c.Memory.Create(ctx, l)
}

func TestShortenGeneratesCodes(t *testing.T) {
	svc := NewService(NewMemory())
	seen := map[string]bool{}
	for range 1000 {
		l, err := svc.Shorten(context.Background(), "https://example.com/x", "")
		if err != nil {
			t.Fatal(err)
		}
		if !codeRE.MatchString(l.Code) || len(l.Code) != genLength || seen[l.Code] {
			t.Fatalf("bad or repeated code %q", l.Code)
		}
		seen[l.Code] = true
	}
}

func TestShortenRetriesCollisions(t *testing.T) {
	svc := NewService(&collidingStore{Memory: NewMemory(), n: 3})
	if _, err := svc.Shorten(context.Background(), "https://a.com", ""); err != nil {
		t.Fatalf("3 collisions should be retried: %v", err)
	}
	svc = NewService(&collidingStore{Memory: NewMemory(), n: 100})
	if _, err := svc.Shorten(context.Background(), "https://a.com", ""); !errors.Is(err, ErrCodeTaken) {
		t.Fatalf("want ErrCodeTaken after 5 attempts, got %v", err)
	}
}
```

**A real session**

```
$ shortener -addr 127.0.0.1:18080 &
$ curl -i -X POST -d '{"url":"https://go.dev/doc/","code":"godocs"}' http://127.0.0.1:18080/api/links
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/links/godocs
X-Request-Id: J2PWT5LTNERQ

{"code":"godocs","url":"https://go.dev/doc/","created_at":"2026-11-15T01:34:13.391124175Z","hits":0}
$ curl -s -o /dev/null -w "%{http_code} -> %{redirect_url}\n" http://127.0.0.1:18080/godocs
302 -> https://go.dev/doc/
$ curl -s http://127.0.0.1:18080/api/links/godocs
{"code":"godocs","url":"https://go.dev/doc/","created_at":"2026-11-15T01:34:13.391124175Z","hits":1}
$ kill -INT %1
{"level":"INFO","msg":"shutting down"}
{"level":"INFO","msg":"stopped cleanly"}
```

### Review checklist (both projects)

- Every network read and write has a deadline or a context.
- Every goroutine has an owner and an exit path; `go test -race -count=20` passes.
- Inputs are bounded: line length, body size, URL length, header size.
- 4xx vs 5xx is correct for every error path; 500s never leak internal text.
- `Shutdown` is tested (the chat test does it; add one for the shortener).
- **Known limitations to discuss:** the shortener's data vanishes on restart (Chapter 9 fixes it); the redirect counts a hit with a write lock on every request — fine for thousands of requests per second, a contention point for much more (Chapter 9 moves counting to the database, Chapter 12 discusses batching); the chat server has no authentication, so anyone can pretend to be anyone.

## Answers to the Chapter 8 prediction questions

**8.3 Q2.** (a) 400 (b) 422 (or 400; be consistent) (c) 403 (d) 503 (or 500) (e) 409 (f) 429 with `Retry-After`.

**8.4 Q2.** 200: the first `Write` sends the status line with 200; the later `WriteHeader(500)` is ignored (and logged as superfluous).

**8.6 Q2.** Breaking: making `code` required; changing `hits` to a string. Compatible: adding `expires_at`; adding an optional query parameter.

# Chapter 9 — Databases, Persistence & Caching

*Goal: store data correctly and fast.*

Almost every backend system is, at its core, a careful conversation with a database. Most production incidents in such systems are not exotic: a missing index, a lost update between two concurrent requests, a migration that locked a table, a connection pool exhausted by a slow query, a cache that served stale data or collapsed under a stampede. This chapter teaches you to prevent each of them.

Everything in this chapter was run against a real PostgreSQL 16 server and a real Redis server while writing it; the query plans, timings, and anomalies you will see are real output.

**What you will be able to do after this chapter**

- Design relational schemas and write the SQL you need (joins, grouping, constraints).
- Read query plans and choose indexes; know why deep OFFSET pagination is slow.
- Use `database/sql` and `pgx` correctly: pools, contexts, scanning, NULLs, errors.
- Use transactions and isolation levels to prevent lost updates and other anomalies.
- Evolve schemas safely with migrations, and stop SQL injection.
- Understand spatial data in PostgreSQL/PostGIS (for the routing chapters).
- Cache with cache-aside, TTLs, invalidation, and stampede protection.
- Test against real databases.

## Topic 9A — SQL and schema design

## Unit 9.1 — Relational model and SQL

**One-line summary:** A relational database stores data in **tables** of rows and typed columns, connects tables through **keys**, and enforces rules with **constraints**. SQL describes *what* data you want (`SELECT … FROM … JOIN … WHERE … GROUP BY …`); the database decides *how* to get it.

**Mental model — a set of well-kept ledgers.** Each table is a ledger with fixed columns. Every row has a unique ID (the **primary key**). Ledgers refer to each other by those IDs (**foreign keys**), so a fact is written once — a customer's address lives in the customers ledger, not copied into every order — and the clerk (the database) refuses entries that break the rules.

### A schema

```sql
CREATE TABLE customers (
    id         bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email      text NOT NULL UNIQUE,
    name       text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE orders (
    id           bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id  bigint NOT NULL REFERENCES customers (id),
    status       text   NOT NULL CHECK (status IN ('new', 'paid', 'shipped', 'cancelled')),
    amount_paise bigint NOT NULL CHECK (amount_paise >= 0),   -- P-8: money as integers
    created_at   timestamptz NOT NULL DEFAULT now()
);
```

Constraints are your last line of defence: `NOT NULL`, `UNIQUE`, `CHECK`, `REFERENCES` (foreign keys), and `PRIMARY KEY` are enforced for **every** writer — your service, a migration, a support engineer with `psql`. Application validation (P-4) gives good error messages; constraints guarantee integrity. Use both.

Types worth knowing in PostgreSQL: `bigint` (Go `int64`), `text` (Go `string`; no length limit, add `CHECK` if needed), `boolean`, `numeric` (exact decimals), `timestamptz` (an instant; always prefer it over `timestamp`, which has no time zone), `date`, `jsonb` (indexed JSON), `uuid`, arrays (`text[]`).

### Querying

```sql
-- the 5 customers who spent the most on paid orders this year
SELECT c.name, sum(o.amount_paise) AS total_paise, count(*) AS orders
FROM orders o
JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'paid'
  AND o.created_at >= date_trunc('year', now())
GROUP BY c.id, c.name
HAVING count(*) >= 2
ORDER BY total_paise DESC
LIMIT 5;
```

- `JOIN … ON` combines rows from two tables where the condition holds (an **inner** join). `LEFT JOIN` keeps every row from the left table, with NULLs where the right has no match (the project's daily-hits query uses one to show days with zero hits).
- `WHERE` filters rows *before* grouping; `HAVING` filters groups *after*.
- `GROUP BY` collapses rows into groups for aggregates (`count`, `sum`, `avg`, `min`, `max`).
- **Logical order of evaluation:** FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT. This explains why you cannot use a SELECT alias in WHERE.

`INSERT … RETURNING`, `UPDATE … RETURNING`, and `INSERT … ON CONFLICT … DO UPDATE` (an **upsert**) are PostgreSQL features the project uses heavily — each saves a round trip and avoids race conditions between "check" and "write".

### Normalisation

Store each fact once. If a customer's email is copied into every order row, changing it requires updating many rows, and they can disagree. **Normal forms** formalise this: every non-key column should depend on the key, the whole key, and nothing but the key. In practice: one table per entity, foreign keys for relationships, a join table for many-to-many relationships (`order_items(order_id, product_id, quantity)`).

**Denormalisation** (deliberately storing derived or duplicated data) is a performance tool, used with care: the project stores `links.hits` (a running total) *and* `daily_hits` (per-day counts), because reading one number is much cheaper than counting millions of click rows on every stats request. The cost is keeping them consistent — which is why the project updates both in one transaction.

### NULL

`NULL` means "unknown", and SQL uses three-valued logic: `NULL = NULL` is NULL (not true), `WHERE x <> 'a'` skips rows where x is NULL, `count(column)` skips NULLs. Use `IS NULL` / `IS NOT NULL`, and declare columns `NOT NULL` unless "unknown" is a real state. In Go, NULLs need special types (Unit 9.3).

### Deep & Tricky

**DT-250. PostgreSQL stores timestamps with microsecond precision; Go has nanoseconds.** *(Seen in this chapter's project)* A `time.Time` created with `time.Now()` and stored in `timestamptz` comes back with its last three digits gone: `…876921805Z` became `…876921Z`. Tests that compare a written value with the read value using `==` or `Equal` fail mysteriously. Truncate before storing (`t.Truncate(time.Microsecond)`) or compare with tolerance.

**DT-251. `NOT IN` with a NULL in the list returns nothing.** *(SQL three-valued logic)* `WHERE id NOT IN (SELECT customer_id FROM blocked)` returns no rows at all if any `customer_id` is NULL, because `x <> NULL` is unknown. Use `NOT EXISTS`, which behaves as expected.

### Mastery check

1. **Explain.** Why should integrity rules live in the database as constraints even if the application validates input?
2. **Predict.** How many rows does `SELECT count(*), count(email) FROM customers` return, and how do the two counts differ?
3. **Write.** A schema for a library (books, copies, members, loans) with all constraints you can justify, and a query listing members with overdue loans and how many days overdue.

## Unit 9.2 — Indexes and query plans

**One-line summary:** An index is a separate, sorted data structure (usually a B-tree, Unit 6.7) that lets the database find rows without scanning the whole table. `EXPLAIN ANALYZE` shows the plan the database chose and what it actually cost. Indexes speed up reads and slow down writes; choose them from your queries.

### Seeing the difference

A table of 1,000,000 orders, searching for one customer's orders — real output:

```
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 4242;

-- without an index on customer_id
 Gather  (cost=1000.00..14813.43 rows=11 width=38) (actual time=2.739..26.596 rows=14 loops=1)
   Workers Planned: 2
   ->  Parallel Seq Scan on orders  (cost=0.00..13812.33 rows=5 width=38) (actual time=1.499..20.550 rows=5 loops=3)
         Filter: (customer_id = 4242)
         Rows Removed by Filter: 333329
 Execution Time: 26.618 ms

-- after CREATE INDEX orders_customer_idx ON orders (customer_id);
 Bitmap Heap Scan on orders  (cost=4.51..47.47 rows=11 width=38) (actual time=0.033..0.050 rows=14 loops=1)
   Recheck Cond: (customer_id = 4242)
   ->  Bitmap Index Scan on orders_customer_idx  (cost=0.00..4.51 rows=11 width=0) (actual time=0.028..0.028 rows=14 loops=1)
         Index Cond: (customer_id = 4242)
 Execution Time: 0.070 ms
```

From 26.6 ms (three processes reading a million rows and throwing away 999,986 of them) to 0.07 ms — **380 times faster**. At 1,000 requests per second, the first version needs about 27 CPU-seconds per second; the second needs 0.07.

### Reading a plan

- Read from the **innermost** (most indented) node outwards.
- `cost=start..total` is the planner's *estimate* in abstract units; `actual time=start..total` and `rows=` are *measurements* (with `ANALYZE`). A big difference between estimated and actual rows means stale statistics (`ANALYZE table`) or a hard-to-estimate condition.
- Node types: `Seq Scan` (read everything), `Index Scan` (follow the index, fetch each row), `Index Only Scan` (the index contains all needed columns), `Bitmap Heap Scan` (collect matching row locations from the index, then read the table pages in order), `Nested Loop` / `Hash Join` / `Merge Join` (join strategies), `Sort`, `Limit`.
- `EXPLAIN (ANALYZE, BUFFERS)` adds how many pages came from cache vs disk.

A sequential scan is not always bad: for a query returning a large fraction of a table, reading everything sequentially beats jumping around through an index (Unit 0.5's locality again). The planner chooses using statistics.

### Designing indexes

- Index the columns used in `WHERE`, `JOIN`, and `ORDER BY` of frequent queries.
- **Composite indexes** serve queries on a *prefix* of their columns: an index on `(customer_id, created_at)` serves `WHERE customer_id = ?` and `WHERE customer_id = ? AND created_at > ?` and `… ORDER BY created_at`, but not `WHERE created_at > ?` alone. Put equality columns first, range or sort columns last.
- **Unique indexes** enforce `UNIQUE` constraints (the project's primary key on `code` is one).
- **Partial indexes** index only some rows: `CREATE INDEX ON orders (created_at) WHERE status = 'new'`.
- **Expression indexes**: a function applied to a column defeats a plain index:

```
EXPLAIN SELECT * FROM orders WHERE date_trunc('day', created_at) = '2026-06-01';
 Parallel Seq Scan on orders
   Filter: (date_trunc('day'::text, created_at) = ...)
```

Rewrite as a range (`created_at >= '2026-06-01' AND created_at < '2026-06-02'`), or create an index on the expression.

- Every index slows `INSERT`/`UPDATE`/`DELETE` and uses disk and memory. Do not index "just in case"; check `pg_stat_user_indexes` for indexes never used.
- On a live table, `CREATE INDEX CONCURRENTLY` builds without blocking writes (Unit 9.5).

### Why deep OFFSET pagination is slow

Real output, same table, page 45,001 of 20 rows:

```
EXPLAIN ANALYZE SELECT id, created_at FROM orders ORDER BY created_at, id LIMIT 20 OFFSET 900000;
 Limit (actual time=511.139..511.151 rows=20 loops=1)
   ->  Index Only Scan using orders_created_idx on orders (actual time=0.047..486.967 rows=900020 loops=1)
 Execution Time: 511.166 ms

EXPLAIN ANALYZE SELECT id, created_at FROM orders
WHERE (created_at, id) > ($last_created_at, $last_id) ORDER BY created_at, id LIMIT 20;
 Limit (actual time=0.035..0.048 rows=20 loops=1)
   ->  Index Only Scan using orders_created_idx on orders (actual time=0.034..0.045 rows=20 loops=1)
         Index Cond: (ROW(created_at, id) > ROW(...))
 Execution Time: 0.068 ms
```

OFFSET must walk past 900,000 index entries to throw them away (511 ms); **keyset** (cursor) pagination seeks directly to the last-seen key and reads 20 (0.07 ms). This is the reason behind Unit 8.6's advice. The `id` in the key breaks ties, so no row is skipped or repeated when timestamps are equal.

### Deep & Tricky

**DT-252. A query on a nullable or low-selectivity column may ignore your index — correctly.** *(Planner behaviour)* An index on `status` (4 values over a million rows) rarely helps: each value matches 25% of the table, and a sequential scan is cheaper. Indexes pay off for *selective* conditions. Check the plan instead of assuming.

**DT-253. `LIKE 'abc%'` can use a B-tree index; `LIKE '%abc'` cannot.** *(Index structure)* A sorted index can find a prefix range, not a suffix or substring. Substring search needs a trigram index (`pg_trgm`) or full-text search.

### Mastery check

1. **Explain.** Why does an index on `(customer_id, created_at)` help `WHERE customer_id = 7 ORDER BY created_at DESC LIMIT 10` but not `WHERE created_at > now() - interval '1 day'`?
2. **Explain.** In the OFFSET plan, why does the `Index Only Scan` show `rows=900020`?
3. **Write.** Load a million rows of your own, write three queries your application would run, and design indexes for them. Record `EXPLAIN ANALYZE` before and after for each.

## Topic 9B — Go and databases

## Unit 9.3 — database/sql and pgx

**One-line summary:** Go talks to databases through a **connection pool**. `database/sql` is the standard, driver-agnostic API; `pgx` is a PostgreSQL-specific driver and toolkit with its own faster API (`pgxpool`). Always pass a context, always close rows, scan into typed variables, and handle `NULL` and "no rows" explicitly.

### The pool

`sql.Open` / `pgxpool.New` do not open a single connection; they create a **pool** that opens connections on demand, reuses them, and limits their number. Create **one pool per database per process** at startup and share it (it is safe for concurrent use).

```go
cfg, err := pgxpool.ParseConfig(dsn)
if err != nil {
	return nil, err
}
cfg.MaxConns = 20                    // more is not better: see DT-254
cfg.MaxConnIdleTime = 5 * time.Minute
db, err := pgxpool.NewWithConfig(ctx, cfg)
if err != nil {
	return nil, err
}
if err := db.Ping(ctx); err != nil { // fail fast at startup if the database is unreachable
	return nil, err
}
```

The equivalent with `database/sql`: `db, err := sql.Open("pgx", dsn)` (using pgx's `stdlib` adapter), then `db.SetMaxOpenConns(20)`, `db.SetMaxIdleConns(20)`, `db.SetConnMaxIdleTime(5*time.Minute)`.

### Exec, Query, QueryRow

```go
// Exec: statements that return no rows
tag, err := db.Exec(ctx, `UPDATE links SET hits = hits + 1 WHERE code = $1`, code)
if tag.RowsAffected() == 0 { /* no such code */ }

// QueryRow: exactly one row expected
var l links.Link
err := db.QueryRow(ctx,
	`SELECT code, url, created_at, hits FROM links WHERE code = $1`, code,
).Scan(&l.Code, &l.URL, &l.CreatedAt, &l.Hits)
if errors.Is(err, pgx.ErrNoRows) { // database/sql: sql.ErrNoRows
	return links.Link{}, links.ErrNotFound
}

// Query: many rows
rows, err := db.Query(ctx, `SELECT code, hits FROM links ORDER BY hits DESC LIMIT $1`, n)
if err != nil {
	return nil, err
}
defer rows.Close() // returns the connection to the pool
var out []TopLink
for rows.Next() {
	var t TopLink
	if err := rows.Scan(&t.Code, &t.Hits); err != nil {
		return nil, err
	}
	out = append(out, t)
}
return out, rows.Err() // an error that ended the iteration early
```

pgx also offers `pgx.CollectRows(rows, pgx.RowToStructByName[T])` and similar helpers that remove the loop boilerplate; the project's `Daily` uses `CollectRows`.

Rules:

- **Placeholders, always** (`$1`, `$2` in PostgreSQL; `?` in MySQL and SQLite). Never build SQL with `fmt.Sprintf` and user data (SQL injection, Unit 9.5).
- **Context on every call** (P-33): a cancelled request stops waiting for its query, and the driver cancels the query on the server.
- **Close rows** (`defer rows.Close()`), or the connection never returns to the pool, and after `MaxConns` leaks every query waits forever.
- **Check `rows.Err()`** after the loop — the same lesson as `scanner.Err()` (Unit 1.12).

### NULLs in Go

A column that can be NULL cannot be scanned into a plain `string` (error: "cannot scan NULL into *string"). Options: a pointer (`*string`, nil for NULL), `sql.Null[string]` (Go 1.22's generic version of `sql.NullString`), or pgx's `pgtype.Text`. Or avoid the problem: `NOT NULL` columns, `coalesce(col, '')` in the query when "empty" is acceptable.

### Errors from the database

PostgreSQL reports errors with a five-character SQLSTATE code. pgx exposes them as `*pgconn.PgError`:

```go
var pgErr *pgconn.PgError
if errors.As(err, &pgErr) && pgErr.Code == "23505" { // unique_violation
	return links.ErrCodeTaken
}
```

Common codes: `23505` unique violation, `23503` foreign key violation, `23514` check violation, `40001` serialization failure (retry, Unit 9.4), `40P01` deadlock detected (retry), `57014` query cancelled. Translating them into domain errors (P-26) keeps SQL details out of the rest of the program.

### `database/sql` vs pgx

| | `database/sql` (+ a driver) | pgx native API |
| --- | --- | --- |
| Databases | any with a driver | PostgreSQL only |
| Performance | good | better (binary protocol, fewer allocations) |
| PostgreSQL features | basic | arrays, `COPY`, `LISTEN/NOTIFY`, batch queries, rich types |
| Ecosystem | `sqlc`, `sqlx`, many ORMs | `sqlc` supports it too |

For a PostgreSQL-only service, pgx's native API is the common choice; libraries that must support several databases use `database/sql`. Many teams generate type-safe Go code from SQL with `sqlc` instead of writing scanning by hand — a good middle ground between hand-written SQL and an ORM.

### Deep & Tricky

**DT-254. A bigger connection pool can make a database slower.** *(Database behaviour)* Each PostgreSQL connection is a process with its own memory; beyond a few times the number of CPU cores, more concurrent queries just contend for CPU, locks, and disk, and throughput drops. With 10 service instances × 50 connections you have 500 backends — often worse than 10 × 10. Size pools from measurement; put PgBouncer in front of PostgreSQL when many instances share one database.

**DT-255. A query that is waiting for a pool connection is not yet running.** *(Pool behaviour)* When all connections are busy, `Query` blocks inside the pool until one frees up (or the context ends). Latency graphs show "slow queries" that are actually fast queries queued behind a few slow ones. `db.Stat()` (`pgxpool.Stat`: `EmptyAcquireCount`, `AcquireDuration`) reveals pool waits; export it as a metric (Unit 11.5).

**DT-256. Holding a transaction or an unclosed `rows` while calling another service ties up a connection.** *(Practice)* A handler that opens a transaction, then calls a slow payment API, then commits, holds a database connection (and row locks) for the whole call. Under load the pool empties. Do external calls outside transactions.

### Mastery check

1. **Explain.** What happens to the pool if a function returns early without `rows.Close()`, and how would the bug show up in production?
2. **Explain.** Why should `UNIQUE` violations be detected from the database error rather than by checking with a `SELECT` first?
3. **Write.** `TopLinks(ctx, n int) ([]links.Link, error)` with `pgx.CollectRows` and `pgx.RowToStructByName`, plus an integration test (Unit 9.8).

## Unit 9.4 — Transactions

**One-line summary:** A transaction groups statements so they succeed or fail together (**atomicity**) and controls how concurrent transactions see each other (**isolation**). PostgreSQL's default, READ COMMITTED, still allows **lost updates** with read-modify-write code. Fix them with atomic statements, row locks (`SELECT … FOR UPDATE`), or SERIALIZABLE isolation with retries.

### ACID

- **Atomicity:** all statements commit, or none do (`ROLLBACK`, or a crash before `COMMIT`).
- **Consistency:** constraints hold after every transaction.
- **Isolation:** concurrent transactions do not see each other's partial work; the *level* decides how much they can see.
- **Durability:** once `COMMIT` returns, the data survives a crash (it is in the write-ahead log on disk).

### Transactions in Go

```go
err := pgx.BeginFunc(ctx, db, func(tx pgx.Tx) error {
	// every statement uses tx, not db
	if _, err := tx.Exec(ctx, `UPDATE accounts SET balance = balance - $1 WHERE id = $2`, amt, from); err != nil {
		return err // BeginFunc rolls back
	}
	_, err := tx.Exec(ctx, `UPDATE accounts SET balance = balance + $1 WHERE id = $2`, amt, to)
	return err // nil: BeginFunc commits
})
```

`pgx.BeginFunc` commits if the function returns nil and rolls back otherwise (including on panic). With `database/sql`, the manual form is `tx, err := db.BeginTx(ctx, nil)`, then `defer tx.Rollback()` (a no-op after a successful commit), then `tx.Commit()`. The classic bug: using `db.Exec` instead of `tx.Exec` inside the transaction — that statement runs on *another* connection, outside the transaction.

### The lost update — measured

Fifty goroutines each add 1 to a balance, concurrently, against real PostgreSQL. Five strategies, real results:

```
read-modify-write (lost updates)   balance =  2  (failed: 0)
atomic update                      balance = 50  (failed: 0)
select ... for update              balance = 50  (failed: 0)
serializable, no retry             balance =  6  (failed: 44)
serializable, with retry           balance = 50  (failed: 0)
```

1. **Read-modify-write** (`SELECT balance`, compute `b+1` in Go, `UPDATE … SET balance = $1`): **48 of 50 increments were lost**, with no error anywhere. Every goroutine read nearly the same old balance and overwrote the others — Unit 7.1's `counter++` race, at database scale. This is the default behaviour under READ COMMITTED.
2. **Atomic update** (`UPDATE accounts SET balance = balance + 1`): correct. The database reads and writes the row under its own row lock. Prefer this whenever the change can be expressed in one statement.
3. **`SELECT … FOR UPDATE`** inside a transaction: correct. The `SELECT` locks the row; other transactions wait at their own `SELECT … FOR UPDATE` until the first commits. Use it when the new value needs logic in Go between read and write.
4. **SERIALIZABLE without retry:** the database detects that concurrent transactions could not have run in some serial order and **aborts** them with SQLSTATE `40001` — 44 failed. Correct (nothing was lost silently), but useless without the next step.
5. **SERIALIZABLE with retry:** correct. Retry the whole transaction on `40001`.

The code of the read-modify-write and retry versions:

```go
// WRONG under concurrency: lost updates
var b int64
db.QueryRow(ctx, `SELECT balance FROM accounts WHERE id = 1`).Scan(&b)
db.Exec(ctx, `UPDATE accounts SET balance = $1 WHERE id = 1`, b+1)

// SERIALIZABLE with retry
for {
	err := pgx.BeginTxFunc(ctx, db, pgx.TxOptions{IsoLevel: pgx.Serializable}, func(tx pgx.Tx) error {
		var b int64
		if err := tx.QueryRow(ctx, `SELECT balance FROM accounts WHERE id = 1`).Scan(&b); err != nil {
			return err
		}
		_, err := tx.Exec(ctx, `UPDATE accounts SET balance = $1 WHERE id = 1`, b+1)
		return err
	})
	var pgErr *pgconn.PgError
	if errors.As(err, &pgErr) && pgErr.Code == "40001" { // serialization_failure: try again
		continue // production code: bounded retries with backoff and jitter (Unit 12.7)
	}
	return err
}
```

### Isolation levels and anomalies

| Level (PostgreSQL) | Dirty read | Non-repeatable read | Phantom read | Lost update / write skew |
| --- | --- | --- | --- | --- |
| READ COMMITTED (default) | no | possible | possible | possible |
| REPEATABLE READ (snapshot) | no | no | no (in PostgreSQL) | lost update detected (error); **write skew possible** |
| SERIALIZABLE | no | no | no | no (aborts with `40001`) |

- **Non-repeatable read:** reading the same row twice in one transaction gives different values because another transaction committed in between.
- **Phantom:** re-running a query returns new rows inserted by another transaction.
- **Write skew:** two transactions read overlapping data, make decisions, and write *different* rows, together breaking a rule neither broke alone. Classic example: "at least one doctor must be on call" — two doctors each check that the other is on call and both go off call. Only SERIALIZABLE (or explicit locking) prevents it.

### Locking and deadlocks

Writes lock rows until commit. Two transactions that lock rows in opposite orders deadlock; PostgreSQL detects it after `deadlock_timeout` (1 s by default) and aborts one with `40P01`. Prevent with a consistent lock order (sort IDs before updating several rows — the same rule as Unit 7.8), keep transactions short, and treat `40P01` as retryable.

### Deep & Tricky

**DT-257. Isolation anomalies produce no errors at READ COMMITTED.** *(Measured above)* The lost-update run returned `failed: 0` and a balance of 2. Nothing in logs, no exception: just wrong data. That is why "it works in testing" means little for concurrent database code; test with real concurrency against a real database.

**DT-258. Retrying requires the whole transaction to be re-run, including the reads.** *(Practice)* Retrying only the failed `UPDATE` reuses a stale read and recreates the bug. Put the entire transaction in a function and retry the function, and make sure it has no side effects outside the database (no emails sent inside the transaction function).

### Pattern

**P-35. Prefer one atomic statement; otherwise lock or serialize — and retry.** *(Data-correctness practice)*

- *Problem it solves:* lost updates and check-then-act races in concurrent requests.
- *How:* express changes as single statements (`SET x = x + 1`, `INSERT … ON CONFLICT`, `UPDATE … WHERE version = $n RETURNING`); when logic must sit between read and write, use `SELECT … FOR UPDATE` or SERIALIZABLE with a bounded retry loop on `40001`/`40P01`.
- *When not to:* read-only reporting queries need none of this.
- *Trade-off:* locks reduce concurrency; SERIALIZABLE costs retries under contention; atomic statements are the cheapest when they fit.

### Mastery check

1. **Explain.** Why did the read-modify-write version lose 48 updates without any error?
2. **Explain.** Write skew: why does REPEATABLE READ not prevent the on-call doctors example, and what does?
3. **Write.** A `Transfer(ctx, from, to, amount)` that never lets a balance go negative, is safe under concurrency, and cannot deadlock when two transfers between the same accounts run in opposite directions. Test it with 100 concurrent transfers and check that the total money is unchanged.

## Unit 9.5 — Migrations and safety

**One-line summary:** Schema changes are code: versioned **migration** files applied in order, recorded in the database, reviewed like code, and designed to run against a live database without downtime. Queries are always parameterised, which makes SQL injection impossible.

### Migrations

A migration is a numbered SQL file. A migration tool records which ones have run (the project uses a `schema_migrations` table) and applies the rest in order at deploy time. The project's minimal migrator embeds the files into the binary (`//go:embed`, Unit 5.3), takes a PostgreSQL **advisory lock** so two instances starting at once do not both migrate, and runs each file in its own transaction (PostgreSQL can roll back DDL):

```go
//go:embed migrations/*.sql
var migrations embed.FS

func Migrate(ctx context.Context, db *pgxpool.Pool) error {
	conn, err := db.Acquire(ctx)
	if err != nil {
		return err
	}
	defer conn.Release()

	const lockID = 727274 // any constant shared by all instances
	if _, err := conn.Exec(ctx, `SELECT pg_advisory_lock($1)`, lockID); err != nil {
		return err
	}
	defer conn.Exec(context.Background(), `SELECT pg_advisory_unlock($1)`, lockID)
	...
}
```

Mature tools (`golang-migrate`, `goose`, `atlas`) add down-migrations, dirty-state detection, and CLI support; the principles are the same.

### Changing a live schema safely

During a rolling deploy, **old and new versions of your code run at the same time against the same schema**. So every migration must be compatible with the code before and after it. The **expand–contract** pattern:

1. **Expand:** add the new column (nullable, or with a default), new table, or new index — old code ignores it.
2. **Migrate code:** deploy code that writes both old and new, then reads new.
3. **Backfill** existing rows in small batches (not one giant `UPDATE` that locks the table and bloats the log).
4. **Contract:** once no code uses the old column, drop it in a later release.

Dangerous operations on big tables: adding a column with a volatile default (rewrites the table in old PostgreSQL versions), adding `NOT NULL` without a default, changing a column's type, creating an index without `CONCURRENTLY` (blocks writes for the whole build). Set `lock_timeout` in migrations so a migration that cannot get its lock fails fast instead of queueing behind a long query and blocking all traffic behind *itself*.

### SQL injection

```go
// VULNERABLE: user input becomes SQL
q := fmt.Sprintf("SELECT * FROM links WHERE code = '%s'", code)
// code = "x' OR '1'='1"  → returns every row
// code = "x'; DROP TABLE links; --"  → worse

// SAFE: the value is sent separately from the SQL text
db.QueryRow(ctx, `SELECT * FROM links WHERE code = $1`, code)
```

With placeholders, the database receives the SQL and the values separately; no value can change the query's structure. Identifiers (table and column names) cannot be parameterised: if they must vary, choose from a fixed allow-list in code (`switch sortColumn { case "hits": … }`) or quote with `pgx.Identifier{name}.Sanitize()`. Prepared statements (pgx prepares and caches statements automatically) also save parsing time for repeated queries.

### Deep & Tricky

**DT-259. A migration waiting for a lock blocks everything behind it.** *(PostgreSQL locking)* `ALTER TABLE` needs a strong lock. If a long-running query holds a weaker lock, the `ALTER` waits — and every new query on that table queues behind the `ALTER`. A two-second schema change becomes a full outage. Always `SET lock_timeout = '3s'` (and retry later) in migrations on busy tables.

### Mastery check

1. **Explain.** Why must a migration work with both the old and the new version of the code?
2. **Write.** Plan (as a sequence of migrations and deploys) renaming `links.url` to `links.target_url` with zero downtime.

## Topic 9C — Spatial data

## Unit 9.6 — PostgreSQL + PostGIS introduction

**One-line summary:** PostGIS adds geometry and geography types, spatial functions, and spatial indexes (GiST, an R-tree-like structure) to PostgreSQL, so "find the 5 nearest charging stations" or "which delivery zone contains this point" become indexed SQL queries. Chapter 13 builds on these ideas.

*(PostGIS was not installed on the machine used to write this chapter, so the SQL below is shown but was not executed here. It uses standard, long-stable PostGIS functions.)*

### Types

- `geometry(Point, 4326)`: coordinates in a planar system; with SRID 4326 (WGS84 longitude/latitude, Unit 13.1) distances come out in **degrees**, which is wrong for most purposes.
- `geography(Point, 4326)`: coordinates on the Earth's ellipsoid; distances in **metres**. Simpler to use correctly, a bit slower.
- `LineString` (a road), `Polygon` (a zone), `MultiPolygon` (a city with islands).

```sql
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE stations (
    id   bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name text NOT NULL,
    loc  geography(Point, 4326) NOT NULL
);
CREATE INDEX stations_loc_idx ON stations USING gist (loc);

INSERT INTO stations (name, loc)
VALUES ('Shivajinagar', ST_SetSRID(ST_MakePoint(73.8478, 18.5308), 4326)::geography); -- (lon, lat)!
```

### Queries

```sql
-- stations within 2 km of a point (uses the GiST index)
SELECT name, ST_Distance(loc, ST_MakePoint(73.8567, 18.5204)::geography) AS metres
FROM stations
WHERE ST_DWithin(loc, ST_MakePoint(73.8567, 18.5204)::geography, 2000)
ORDER BY metres;

-- the 5 nearest stations: index-assisted nearest neighbour with the <-> operator
SELECT name
FROM stations
ORDER BY loc <-> ST_MakePoint(73.8567, 18.5204)::geography
LIMIT 5;

-- which zone contains a point
SELECT zone_id FROM zones WHERE ST_Contains(boundary, ST_SetSRID(ST_MakePoint($1, $2), 4326));
```

Two classic bugs: **longitude comes first** in `ST_MakePoint(lon, lat)` (the opposite of how people say coordinates), and `ST_Distance(a, b) < 2000` in `WHERE` cannot use the index while `ST_DWithin` can.

### From Go

Pass coordinates as plain numbers and build points in SQL (`ST_MakePoint($1, $2)`), or exchange well-known text/binary (`ST_AsText`, `ST_GeomFromText`). Libraries such as `github.com/twpayne/go-geom` handle encoding when you need full geometries.

### Mastery check

1. **Explain.** Why is `geometry` with SRID 4326 a trap for distance queries?
2. **Write.** (If you can install PostGIS.) Load 100,000 random points in a city's bounding box, and compare `EXPLAIN ANALYZE` of a 1 km `ST_DWithin` query with and without the GiST index.

## Topic 9D — Caching

## Unit 9.7 — Caching strategies

**One-line summary:** A cache keeps copies of expensive-to-get data in faster storage. **Cache-aside** (read the cache, on a miss read the source and fill the cache) with a **TTL** is the default pattern. The hard parts are **invalidation** (stale data), **stampedes** (many simultaneous misses), and deciding what consistency your users actually need.

**Mental model — the kitchen counter again.** Unit 0.5's counter (L1 cache) and kirana shop (RAM) are the same idea one level up: Redis or an in-process map is the counter; the database is the shop. A good cache serves most requests from the counter; a bad one serves stale milk.

### Where to cache

| Layer | Example | Speed | Shared across instances? |
| --- | --- | --- | --- |
| In-process | a map + mutex, the LRU of Unit 6.13 | ~100 ns | no |
| Distributed | Redis, Memcached | ~0.2–1 ms (network) | yes |
| HTTP | `Cache-Control`, CDN | avoids reaching you at all | yes |
| Database | its own buffer cache | automatic | — |

### Cache-aside

```go
func (c *Links) lookup(ctx context.Context, code string) (string, error) {
	v, found, err := c.kv.Get(ctx, key(code))
	if err != nil {
		c.log.WarnContext(ctx, "cache get failed; using database", "err", err)
	} else if found {
		if v == notFound {
			return "", links.ErrNotFound
		}
		return v, nil
	}
	res, err, _ := c.group.Do(code, func() (any, error) {
		l, err := c.backend.Get(ctx, code)
		switch {
		case errors.Is(err, links.ErrNotFound):
			c.kv.Set(ctx, key(code), notFound, c.negTTL)
			return "", err
		case err != nil:
			return "", err
		}
		if err := c.kv.Set(ctx, key(code), l.URL, c.ttl); err != nil {
			c.log.WarnContext(ctx, "cache set failed", "err", err)
		}
		return l.URL, nil
	})
	if err != nil {
		return "", err
	}
	return res.(string), nil
}
```

(From this chapter's project.) Four production details are in those lines:

1. **A cache failure is not a request failure.** If Redis is down, log and fall through to the database. The cache is an optimisation.
2. **TTL.** Every entry expires (`c.ttl`), bounding how stale data can be and how long a bug's effects last.
3. **Negative caching.** "No such code" is cached briefly too, so a bot hammering random codes cannot turn every request into a database query.
4. **Stampede protection.** When a popular entry expires, hundreds of concurrent requests miss at once and all hit the database. `singleflight.Group` (from `golang.org/x/sync`) makes concurrent calls with the same key share one execution. The project's test proves it: **100 concurrent cold requests cause exactly 1 database read**.

### Invalidation

"There are only two hard things in computer science: cache invalidation and naming things." Strategies:

- **TTL only:** simplest; data is stale for up to the TTL. Fine for data that rarely changes or where slight staleness is harmless.
- **Delete on write:** after updating the database, delete the cache key; the next read refills it. Small race: a reader that loaded the old value just before the update can write it back to the cache after the delete. TTL limits the damage.
- **Write-through:** write the new value to the cache as part of the write (the project's `Create` does this, which also clears a cached "not found").
- **Versioned keys:** include a version or updated-at in the key; old entries simply age out.

Never cache without thinking: what is the worst thing a stale value can cause here? A stale redirect target for a minute is harmless; a stale account balance or permission is not.

### Write-behind batching

The project caches the redirect lookup, but each redirect also *counts a hit*. Writing a row per redirect would put the database right back on the hot path. Instead, hits are counted in memory and flushed every second in **one** statement for the whole batch (`UPDATE … FROM unnest($1::text[], $2::bigint[])`). The trade-offs, written down in the package documentation: counts reach the database up to a second late (the stats endpoint adds the pending counts so users do not notice), and a crash loses at most one second of counts. For hit counters that is a good trade; for payments it would be unacceptable.

### Redis basics

Redis is an in-memory data-structure server: strings with TTL (`SET key value EX 600`, `GET`), counters (`INCR`), hashes, lists, sets, sorted sets (leaderboards, rate limiters), pub/sub, streams. With `github.com/redis/go-redis/v9`:

```go
rdb := redis.NewClient(&redis.Options{Addr: "localhost:6379",
	DialTimeout: 2 * time.Second, ReadTimeout: 200 * time.Millisecond, WriteTimeout: 200 * time.Millisecond})
err := rdb.Set(ctx, "link:godocs", "https://go.dev/doc/", 10*time.Minute).Err()
v, err := rdb.Get(ctx, "link:godocs").Result()
if errors.Is(err, redis.Nil) { /* not in cache */ }
```

Short timeouts matter: a cache that takes 5 seconds to fail is worse than no cache.

### Deep & Tricky

**DT-260. Caches hide capacity problems until they fail.** *(Operations)* A database sized for 10% of traffic (because 90% hits the cache) falls over the moment the cache is flushed, restarted, or a deploy changes key formats — a "cold start" outage. Know your database's capacity without the cache, warm caches gradually, and use singleflight and request coalescing.

**DT-261. A decorator hides optional interfaces unless it forwards them.** *(Seen in this chapter's project, P-24)* The cache layer wraps the store. The store has an optional `Daily` method (checked with a type assertion, Unit 3.6), but the cache type does not have it unless it explicitly implements and forwards it. Without the forwarding method, the analytics endpoint would silently return 501 "not available" after adding the cache.

### Pattern

**P-36. Cache-aside with TTL, negative caching, and singleflight; the cache is never required.** *(Caching practice)* Read through the cache, fill on miss, expire everything, cache "not found" briefly, collapse concurrent misses, and treat every cache error as a miss. *Trade-off:* bounded staleness and extra moving parts in exchange for lower latency and database load.

### Mastery check

1. **Explain.** Describe the race in "delete on write" invalidation step by step. How does a TTL limit it?
2. **Explain.** What would happen to the database if the cache's singleflight were removed and a link with 5,000 requests per second expired?
3. **Write.** Add a `Delete` endpoint to the shortener: delete from the database, then invalidate the cache key. Write a test that would catch the stale-redirect bug if the invalidation were missing.

## Topic 9E — Testing with databases

## Unit 9.8 — Integration testing with real databases in containers

**One-line summary:** Database code must be tested against a real database of the same kind and version — mocks cannot reproduce SQL semantics, constraints, isolation, or query plans. Give each test an isolated database (or schema, or transaction), start the server in a container in CI, and skip cleanly when it is unavailable.

### Why not mock the database?

A mock of `Exec` and `QueryRow` would have "passed" every test of the lost-update code in Unit 9.4, never seen a `23505` unique violation, never noticed that PostgreSQL truncates nanoseconds (DT-250), and never tested a migration. The database's behaviour *is* what you are testing. Use fakes (the in-memory store) for testing *other* layers; test the store against the real thing.

### Isolation strategies

| Strategy | How | Speed | Notes |
| --- | --- | --- | --- |
| Database per test | `CREATE DATABASE test_<random>`, migrate, drop at cleanup | ~50–200 ms per test | complete isolation; the project uses this |
| Template database | migrate once into a template, `CREATE DATABASE … TEMPLATE` per test | fast | PostgreSQL feature |
| Schema per test | `CREATE SCHEMA`, set `search_path` | fast | isolation within one database |
| Transaction per test | begin, run the test, roll back | fastest | cannot test code that commits or uses several connections |

The project's helper:

```go
// newDB creates a fresh, migrated database for one test and drops it after.
// It needs TEST_DATABASE_URL pointing at a server where the user may create
// databases, e.g. postgres://postgres@localhost/postgres.
func newDB(t *testing.T) *pgxpool.Pool {
	t.Helper()
	admin := os.Getenv("TEST_DATABASE_URL")
	if admin == "" {
		t.Skip("TEST_DATABASE_URL not set; skipping PostgreSQL integration test")
	}
	ctx := context.Background()
	conn, err := pgx.Connect(ctx, admin)
	if err != nil {
		t.Fatal(err)
	}
	name := "test_" + strings.ToLower(rand.Text()[:10])
	if _, err := conn.Exec(ctx, "CREATE DATABASE "+name); err != nil {
		t.Fatal(err)
	}
	cfg, _ := pgx.ParseConfig(admin)
	cfg.Database = name
	db, err := Open(ctx, cfg.ConnString())
	if err != nil {
		t.Fatal(err)
	}
	t.Cleanup(func() {
		db.Close()
		conn.Exec(ctx, "DROP DATABASE "+name)
		conn.Close(ctx)
	})
	return db
}
```

(`CREATE DATABASE` cannot take a placeholder; the name is generated by the test, never from input, so concatenation is safe here — and only here.)

### Containers

`docker run -d -p 5432:5432 -e POSTGRES_HOST_AUTH_METHOD=trust postgres:16` gives every developer and CI job the same database version. CI systems (GitHub Actions `services:`, GitLab services) start one next to the test job. The `testcontainers-go` library starts containers from inside `TestMain` (DT-182), waits until they accept connections, and removes them afterwards — so `go test ./...` works with no manual setup when Docker is available.

### Make it fast enough to run always

- Run integration tests in parallel (`t.Parallel`) — each has its own database.
- Migrate once into a template database, then clone it per test.
- Keep a separate build tag or the skip-without-environment approach for developers without Docker, but **always** run them in CI.

### Mastery check

1. **Explain.** Name three bugs from this chapter that a mocked database would never reveal.
2. **Write.** Convert the project's `newDB` to the template-database strategy and measure the time saved per test.

## Chapter 9 Project — The URL shortener on PostgreSQL, with a cache and analytics

**Goal:** turn the Chapter 8 shortener into a persistent, multi-instance-safe service: PostgreSQL storage with embedded migrations, per-day analytics, a Redis cache-aside layer with stampede protection and negative caching, batched hit counting, and integration tests against a real database.

### Requirements

1. `pgstore.Store` implements `links.Store` with `pgx`: unique codes enforced by the database (`23505` → `links.ErrCodeTaken`), `ErrNotFound` for missing codes, every call with a context.
2. Migrations embedded in the binary, applied at startup under an advisory lock, recorded in `schema_migrations`, safe to run twice.
3. `daily_hits(code, day, hits)` maintained in the same transaction as `links.hits`; a new endpoint `GET /api/links/{code}/daily?days=7` returns one entry per day including zeros (1–366 days).
4. `cache.Links` decorates the store: cache-aside for redirects, TTL 10 minutes, negative caching 30 s, `singleflight` for misses, Redis optional (in-process cache when `REDIS_ADDR` is empty), cache errors never fail requests.
5. Hits are counted in memory and flushed every second with one batched statement; a failed flush keeps the counts for the next attempt; shutdown flushes after the HTTP server has stopped.
6. Configuration from environment variables (`DATABASE_URL`, `REDIS_ADDR`, `ADDR`) — the twelve-factor style expanded in Unit 11.1.
7. Integration tests with a fresh database per test (skipped without `TEST_DATABASE_URL`), cache tests that prove 100 concurrent cold hits cause one backend read, and `go vet` clean.

### What changed from Chapter 8

`links` gains an optional capability, and the service exposes it:

```go
// DayCount is the number of hits on one day (YYYY-MM-DD, UTC).
type DayCount struct {
	Day  string `json:"day"`
	Hits int64  `json:"hits"`
}

// DailyCounter is an optional Store capability (Unit 3.6).
type DailyCounter interface {
	Daily(ctx context.Context, code string, days int) ([]DayCount, error)
}

var ErrUnsupported = errors.New("not supported by this store")

// Daily returns per-day hits for the last days (1-366).
func (s *Service) Daily(ctx context.Context, code string, days int) ([]DayCount, error) {
	if days < 1 || days > 366 {
		return nil, &InvalidError{"days must be between 1 and 366"}
	}
	dc, ok := s.store.(DailyCounter)
	if !ok {
		return nil, ErrUnsupported
	}
	return dc.Daily(ctx, code, days)
}
```

`httpapi` gains one route and one error mapping:

```go
	mux.HandleFunc("GET /api/links/{code}/daily", a.daily)
...
func (a *API) daily(w http.ResponseWriter, r *http.Request) {
	days := 7
	if v := r.URL.Query().Get("days"); v != "" {
		n, err := strconv.Atoi(v)
		if err != nil {
			a.writeError(w, r, http.StatusBadRequest, "days must be a number")
			return
		}
		days = n
	}
	counts, err := a.svc.Daily(r.Context(), r.PathValue("code"), days)
	if err != nil {
		a.fail(w, r, err)
		return
	}
	writeJSON(w, http.StatusOK, counts)
}
...
	case errors.Is(err, links.ErrUnsupported):
		a.writeError(w, r, http.StatusNotImplemented, "not available")
```

### `internal/pgstore/migrations`

`001_links.sql`:

```sql
CREATE TABLE links (
    code       text        PRIMARY KEY,
    url        text        NOT NULL CHECK (length(url) <= 2048),
    created_at timestamptz NOT NULL DEFAULT now(),
    hits       bigint      NOT NULL DEFAULT 0 CHECK (hits >= 0)
);
```

`002_daily_hits.sql`:

```sql
-- One row per link per day: small, and fast to aggregate.
CREATE TABLE daily_hits (
    code text   NOT NULL REFERENCES links (code) ON DELETE CASCADE,
    day  date   NOT NULL,
    hits bigint NOT NULL CHECK (hits > 0),
    PRIMARY KEY (code, day)
);
```

### `internal/pgstore/migrate.go`

```go
package pgstore

import (
	"context"
	"embed"
	"fmt"
	"io/fs"
	"slices"

	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgxpool"
)

//go:embed migrations/*.sql
var migrations embed.FS

// Migrate applies every migration not yet recorded in schema_migrations,
// in file-name order, each in its own transaction. An advisory lock makes
// it safe when several instances start at once.
func Migrate(ctx context.Context, db *pgxpool.Pool) error {
	conn, err := db.Acquire(ctx)
	if err != nil {
		return err
	}
	defer conn.Release()

	const lockID = 727274 // any constant shared by all instances
	if _, err := conn.Exec(ctx, `SELECT pg_advisory_lock($1)`, lockID); err != nil {
		return err
	}
	defer conn.Exec(context.Background(), `SELECT pg_advisory_unlock($1)`, lockID)

	if _, err := conn.Exec(ctx, `CREATE TABLE IF NOT EXISTS schema_migrations (
		name text PRIMARY KEY, applied_at timestamptz NOT NULL DEFAULT now())`); err != nil {
		return err
	}
	names, err := fs.Glob(migrations, "migrations/*.sql")
	if err != nil {
		return err
	}
	slices.Sort(names)
	for _, name := range names {
		var done bool
		if err := conn.QueryRow(ctx, `SELECT EXISTS (SELECT 1 FROM schema_migrations WHERE name = $1)`, name).Scan(&done); err != nil {
			return err
		}
		if done {
			continue
		}
		sql, err := migrations.ReadFile(name)
		if err != nil {
			return err
		}
		err = pgx.BeginFunc(ctx, conn, func(tx pgx.Tx) error {
			if _, err := tx.Exec(ctx, string(sql)); err != nil {
				return err
			}
			_, err := tx.Exec(ctx, `INSERT INTO schema_migrations (name) VALUES ($1)`, name)
			return err
		})
		if err != nil {
			return fmt.Errorf("migration %s: %w", name, err)
		}
	}
	return nil
}
```

The advisory lock is taken on a dedicated connection (`Acquire`), because PostgreSQL session locks belong to a connection: taking it through the pool and unlocking through a different pooled connection would not release it.

### `internal/pgstore/pgstore.go`

```go
// Package pgstore stores links in PostgreSQL.
package pgstore

import (
	"context"
	"errors"
	"fmt"
	"time"

	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgconn"
	"github.com/jackc/pgx/v5/pgxpool"

	"example.com/shortener/internal/links"
)

type Store struct {
	db  *pgxpool.Pool
	now func() time.Time
}

var (
	_ links.Store        = (*Store)(nil)
	_ links.DailyCounter = (*Store)(nil)
)

func New(db *pgxpool.Pool) *Store { return &Store{db: db, now: time.Now} }

// Open connects, checks the connection, and applies migrations.
func Open(ctx context.Context, dsn string) (*pgxpool.Pool, error) {
	cfg, err := pgxpool.ParseConfig(dsn)
	if err != nil {
		return nil, err
	}
	cfg.MaxConns = 20
	cfg.MaxConnIdleTime = 5 * time.Minute
	db, err := pgxpool.NewWithConfig(ctx, cfg)
	if err != nil {
		return nil, err
	}
	if err := db.Ping(ctx); err != nil {
		db.Close()
		return nil, fmt.Errorf("ping database: %w", err)
	}
	if err := Migrate(ctx, db); err != nil {
		db.Close()
		return nil, err
	}
	return db, nil
}

const uniqueViolation = "23505"

func (s *Store) Create(ctx context.Context, l links.Link) error {
	_, err := s.db.Exec(ctx,
		`INSERT INTO links (code, url, created_at) VALUES ($1, $2, $3)`,
		l.Code, l.URL, l.CreatedAt)
	var pgErr *pgconn.PgError
	if errors.As(err, &pgErr) && pgErr.Code == uniqueViolation {
		return links.ErrCodeTaken // the database enforces uniqueness, even across instances
	}
	return err
}

func (s *Store) Get(ctx context.Context, code string) (links.Link, error) {
	var l links.Link
	err := s.db.QueryRow(ctx,
		`SELECT code, url, created_at, hits FROM links WHERE code = $1`, code,
	).Scan(&l.Code, &l.URL, &l.CreatedAt, &l.Hits)
	if errors.Is(err, pgx.ErrNoRows) {
		return links.Link{}, links.ErrNotFound
	}
	return l, err
}

// Hit counts one visit and returns the URL: two writes in one transaction.
func (s *Store) Hit(ctx context.Context, code string) (string, error) {
	var url string
	err := pgx.BeginFunc(ctx, s.db, func(tx pgx.Tx) error {
		err := tx.QueryRow(ctx,
			`UPDATE links SET hits = hits + 1 WHERE code = $1 RETURNING url`, code).Scan(&url)
		if errors.Is(err, pgx.ErrNoRows) {
			return links.ErrNotFound
		}
		if err != nil {
			return err
		}
		_, err = tx.Exec(ctx, `
			INSERT INTO daily_hits (code, day, hits) VALUES ($1, $2, 1)
			ON CONFLICT (code, day) DO UPDATE SET hits = daily_hits.hits + 1`,
			code, s.now().UTC().Format(time.DateOnly))
		return err
	})
	return url, err
}

// AddHits applies many counted hits at once: one round trip and one
// transaction for a whole batch (used by the cache's flusher).
func (s *Store) AddHits(ctx context.Context, hits map[string]int64) error {
	if len(hits) == 0 {
		return nil
	}
	codes := make([]string, 0, len(hits))
	counts := make([]int64, 0, len(hits))
	for c, n := range hits {
		codes = append(codes, c)
		counts = append(counts, n)
	}
	day := s.now().UTC().Format(time.DateOnly)
	return pgx.BeginFunc(ctx, s.db, func(tx pgx.Tx) error {
		_, err := tx.Exec(ctx, `
			UPDATE links SET hits = links.hits + b.n
			FROM unnest($1::text[], $2::bigint[]) AS b(code, n)
			WHERE links.code = b.code`, codes, counts)
		if err != nil {
			return err
		}
		_, err = tx.Exec(ctx, `
			INSERT INTO daily_hits (code, day, hits)
			SELECT b.code, $3::date, b.n
			FROM unnest($1::text[], $2::bigint[]) AS b(code, n)
			WHERE EXISTS (SELECT 1 FROM links WHERE links.code = b.code)
			ON CONFLICT (code, day) DO UPDATE SET hits = daily_hits.hits + EXCLUDED.hits`,
			codes, counts, day)
		return err
	})
}

// Daily returns hits per day for the last `days` days, oldest first,
// including days with zero hits.
func (s *Store) Daily(ctx context.Context, code string, days int) ([]links.DayCount, error) {
	if _, err := s.Get(ctx, code); err != nil {
		return nil, err
	}
	today := s.now().UTC().Truncate(24 * time.Hour)
	rows, err := s.db.Query(ctx, `
		SELECT d::date, coalesce(h.hits, 0)
		FROM generate_series($2::date - ($3::int - 1), $2::date, interval '1 day') AS d
		LEFT JOIN daily_hits h ON h.code = $1 AND h.day = d::date
		ORDER BY d`, code, today, days)
	if err != nil {
		return nil, err
	}
	return pgx.CollectRows(rows, func(r pgx.CollectableRow) (links.DayCount, error) {
		var dc links.DayCount
		var day time.Time
		err := r.Scan(&day, &dc.Hits)
		dc.Day = day.Format(time.DateOnly)
		return dc, err
	})
}
```

Notes: `AddHits` sends whole arrays as two parameters and lets `unnest` turn them into rows — one round trip for any batch size, and still fully parameterised. The `WHERE EXISTS` guard stops a hit for a deleted link from violating the foreign key and failing the whole batch.

### `internal/cache/cache.go`

```go
// Package cache puts a cache-aside layer in front of a links.Store.
//
// Redirects are the hot path. Cached links skip the database read, and
// hits are counted in memory and written to the store in batches, so a
// redirect usually costs one cache lookup and no database work at all.
// The price: hit counts reach the database up to one flush interval late,
// and a crash loses at most that interval's counts.
package cache

import (
	"context"
	"errors"
	"log/slog"
	"sync"
	"time"

	"golang.org/x/sync/singleflight"

	"example.com/shortener/internal/links"
)

// KV is the small part of a key-value cache (such as Redis) that we use.
type KV interface {
	Get(ctx context.Context, key string) (value string, found bool, err error)
	Set(ctx context.Context, key, value string, ttl time.Duration) error
}

// Backend is a store that can also apply hits in batches.
type Backend interface {
	links.Store
	AddHits(ctx context.Context, hits map[string]int64) error
}

const notFound = "\x00" // cached marker for "no such code" (negative caching)

type Links struct {
	backend Backend
	kv      KV
	ttl     time.Duration
	negTTL  time.Duration
	log     *slog.Logger
	group   singleflight.Group
	mu      sync.Mutex
	pending map[string]int64 // hits not yet written to the backend
}

func New(backend Backend, kv KV, ttl time.Duration, log *slog.Logger) *Links {
	return &Links{backend: backend, kv: kv, ttl: ttl, negTTL: 30 * time.Second,
		log: log, pending: make(map[string]int64)}
}

var _ links.Store = (*Links)(nil)

func key(code string) string { return "link:" + code }

func (c *Links) Create(ctx context.Context, l links.Link) error {
	if err := c.backend.Create(ctx, l); err != nil {
		return err
	}
	// Overwrite any cached "not found" marker for this code.
	if err := c.kv.Set(ctx, key(l.Code), l.URL, c.ttl); err != nil {
		c.log.WarnContext(ctx, "cache set failed", "err", err)
	}
	return nil
}

// Get is not cached: stats should be as fresh as the database, plus the
// hits still waiting in memory.
func (c *Links) Get(ctx context.Context, code string) (links.Link, error) {
	l, err := c.backend.Get(ctx, code)
	if err != nil {
		return l, err
	}
	c.mu.Lock()
	l.Hits += c.pending[code]
	c.mu.Unlock()
	return l, nil
}

// Daily passes the optional capability through. Without this method the
// decorator would hide it (P-24's trade-off) and the endpoint would 501.
func (c *Links) Daily(ctx context.Context, code string, days int) ([]links.DayCount, error) {
	dc, ok := c.backend.(links.DailyCounter)
	if !ok {
		return nil, links.ErrUnsupported
	}
	if err := c.Flush(ctx); err != nil { // include recent hits
		return nil, err
	}
	return dc.Daily(ctx, code, days)
}

func (c *Links) Hit(ctx context.Context, code string) (string, error) {
	url, err := c.lookup(ctx, code)
	if err != nil {
		return "", err
	}
	c.mu.Lock()
	c.pending[code]++
	c.mu.Unlock()
	return url, nil
}

// lookup is cache-aside: try the cache, else load from the backend and fill
// the cache. singleflight makes concurrent misses for one code share a single
// backend call, so a popular link expiring cannot stampede the database.
func (c *Links) lookup(ctx context.Context, code string) (string, error) {
	v, found, err := c.kv.Get(ctx, key(code))
	if err != nil {
		c.log.WarnContext(ctx, "cache get failed; using database", "err", err)
	} else if found {
		if v == notFound {
			return "", links.ErrNotFound
		}
		return v, nil
	}
	res, err, _ := c.group.Do(code, func() (any, error) {
		l, err := c.backend.Get(ctx, code)
		switch {
		case errors.Is(err, links.ErrNotFound):
			c.kv.Set(ctx, key(code), notFound, c.negTTL)
			return "", err
		case err != nil:
			return "", err
		}
		if err := c.kv.Set(ctx, key(code), l.URL, c.ttl); err != nil {
			c.log.WarnContext(ctx, "cache set failed", "err", err)
		}
		return l.URL, nil
	})
	if err != nil {
		return "", err
	}
	return res.(string), nil
}

// Flush writes pending hits to the backend. On failure the hits are put
// back, so they are retried at the next flush.
func (c *Links) Flush(ctx context.Context) error {
	c.mu.Lock()
	batch := c.pending
	c.pending = make(map[string]int64)
	c.mu.Unlock()
	if len(batch) == 0 {
		return nil
	}
	if err := c.backend.AddHits(ctx, batch); err != nil {
		c.mu.Lock()
		for k, n := range batch {
			c.pending[k] += n
		}
		c.mu.Unlock()
		return err
	}
	return nil
}

// Run flushes every interval until ctx is done, then flushes once more.
func (c *Links) Run(ctx context.Context, interval time.Duration) {
	t := time.NewTicker(interval)
	defer t.Stop()
	for {
		select {
		case <-t.C:
			if err := c.Flush(ctx); err != nil {
				c.log.ErrorContext(ctx, "flushing hits", "err", err)
			}
		case <-ctx.Done():
			final, cancel := context.WithTimeout(context.WithoutCancel(ctx), 5*time.Second)
			if err := c.Flush(final); err != nil {
				c.log.Error("final flush", "err", err)
			}
			cancel()
			return
		}
	}
}
```

The final flush uses `context.WithoutCancel(ctx)` (Unit 7.9): `ctx` is already cancelled at that point, but the last hits must still be written, within their own 5-second budget.

### `internal/cache/kv.go`

```go
package cache

import (
	"context"
	"errors"
	"sync"
	"time"

	"github.com/redis/go-redis/v9"
)

// RedisKV adapts a go-redis client to KV.
type RedisKV struct{ C *redis.Client }

func (r RedisKV) Get(ctx context.Context, key string) (string, bool, error) {
	v, err := r.C.Get(ctx, key).Result()
	if errors.Is(err, redis.Nil) {
		return "", false, nil
	}
	if err != nil {
		return "", false, err
	}
	return v, true, nil
}

func (r RedisKV) Set(ctx context.Context, key, value string, ttl time.Duration) error {
	return r.C.Set(ctx, key, value, ttl).Err()
}

// MemoryKV is an in-process KV with expiry, for tests and single-instance use.
type MemoryKV struct {
	mu  sync.Mutex
	m   map[string]memEntry
	now func() time.Time
}

type memEntry struct {
	v   string
	exp time.Time
}

func NewMemoryKV() *MemoryKV { return &MemoryKV{m: map[string]memEntry{}, now: time.Now} }

func (k *MemoryKV) Get(_ context.Context, key string) (string, bool, error) {
	k.mu.Lock()
	defer k.mu.Unlock()
	e, ok := k.m[key]
	if !ok || k.now().After(e.exp) {
		return "", false, nil
	}
	return e.v, true, nil
}

func (k *MemoryKV) Set(_ context.Context, key, value string, ttl time.Duration) error {
	k.mu.Lock()
	defer k.mu.Unlock()
	k.m[key] = memEntry{value, k.now().Add(ttl)}
	return nil
}
```

(`MemoryKV` never deletes expired entries: fine for tests; for production single-instance use, combine it with the LRU of Unit 6.13 to bound memory.)

### `cmd/shortener/main.go`

```go
// Command shortener runs the URL shortener HTTP service.
//
// Configuration comes from the environment:
//
//	DATABASE_URL  PostgreSQL connection string (required)
//	REDIS_ADDR    Redis address, e.g. localhost:6379 (optional; in-process cache if empty)
//	ADDR          listen address (default :8080)
package main

import (
	"cmp"
	"context"
	"errors"
	"log/slog"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/redis/go-redis/v9"

	"example.com/shortener/internal/cache"
	"example.com/shortener/internal/httpapi"
	"example.com/shortener/internal/links"
	"example.com/shortener/internal/pgstore"
)

func main() {
	log := slog.New(slog.NewJSONHandler(os.Stderr, nil))
	if err := run(log); err != nil {
		log.Error("shortener stopped", "err", err)
		os.Exit(1)
	}
}

func run(log *slog.Logger) error {
	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	dsn := os.Getenv("DATABASE_URL")
	if dsn == "" {
		return errors.New("DATABASE_URL is required")
	}
	startCtx, cancel := context.WithTimeout(ctx, 30*time.Second)
	db, err := pgstore.Open(startCtx, dsn)
	cancel()
	if err != nil {
		return err
	}
	defer db.Close()

	var kv cache.KV = cache.NewMemoryKV()
	if addr := os.Getenv("REDIS_ADDR"); addr != "" {
		rdb := redis.NewClient(&redis.Options{Addr: addr, DialTimeout: 2 * time.Second,
			ReadTimeout: 200 * time.Millisecond, WriteTimeout: 200 * time.Millisecond})
		defer rdb.Close()
		kv = cache.RedisKV{C: rdb}
	}
	cached := cache.New(pgstore.New(db), kv, 10*time.Minute, log)
	flushCtx, stopFlush := context.WithCancel(context.Background())
	flushed := make(chan struct{})
	go func() {
		defer close(flushed)
		cached.Run(flushCtx, time.Second)
	}()
	// Runs last (defers are LIFO): after the HTTP server has stopped, stop the
	// flusher, which writes the remaining hits, and wait for it.
	defer func() {
		stopFlush()
		<-flushed
	}()

	srv := &http.Server{
		Addr:              cmp.Or(os.Getenv("ADDR"), ":8080"),
		Handler:           httpapi.New(links.NewService(cached), log),
		ReadHeaderTimeout: 5 * time.Second,
		ReadTimeout:       10 * time.Second,
		WriteTimeout:      10 * time.Second,
		IdleTimeout:       60 * time.Second,
	}
	errCh := make(chan error, 1)
	go func() {
		log.Info("listening", "addr", srv.Addr)
		errCh <- srv.ListenAndServe()
	}()
	select {
	case err := <-errCh:
		return err
	case <-ctx.Done():
	}
	log.Info("shutting down")
	shutdownCtx, cancel2 := context.WithTimeout(context.Background(), 15*time.Second)
	defer cancel2()
	if err := srv.Shutdown(shutdownCtx); err != nil {
		return err
	}
	log.Info("http server stopped; flushing hits")
	return nil
}
```

A note from development: the first version called `stopFlush()` only at the end of the happy path, and `go vet` reported `the stopFlush function is not used on all paths (possible context leak)` — if `ListenAndServe` failed at startup, the flusher goroutine would never be stopped. Moving the stop into a `defer` fixed it. Run `go vet` on every change; it catches exactly the bugs that are easy to miss in review.

### Tests

`internal/pgstore/pgstore_test.go` (the `newDB` helper is shown in Unit 9.8):

```go
func TestStore(t *testing.T) {
	db := newDB(t)
	s := New(db)
	s.now = func() time.Time { return time.Date(2026, 11, 15, 10, 0, 0, 0, time.UTC) }
	ctx := context.Background()

	l := links.Link{Code: "godocs", URL: "https://go.dev/doc/", CreatedAt: s.now()}
	if err := s.Create(ctx, l); err != nil {
		t.Fatal(err)
	}
	if err := s.Create(ctx, l); !errors.Is(err, links.ErrCodeTaken) {
		t.Fatalf("duplicate: %v", err)
	}
	for range 3 {
		if url, err := s.Hit(ctx, "godocs"); err != nil || url != l.URL {
			t.Fatalf("Hit = %q, %v", url, err)
		}
	}
	if _, err := s.Hit(ctx, "nope"); !errors.Is(err, links.ErrNotFound) {
		t.Fatalf("unknown: %v", err)
	}
	if err := s.AddHits(ctx, map[string]int64{"godocs": 10, "ghost": 5}); err != nil {
		t.Fatal(err)
	}
	got, err := s.Get(ctx, "godocs")
	if err != nil || got.Hits != 13 {
		t.Fatalf("Get = %+v, %v; want 13 hits", got, err)
	}
	days, err := s.Daily(ctx, "godocs", 3)
	if err != nil {
		t.Fatal(err)
	}
	want := []links.DayCount{{Day: "2026-11-13"}, {Day: "2026-11-14"}, {Day: "2026-11-15", Hits: 13}}
	if len(days) != 3 || days[0] != want[0] || days[2] != want[2] {
		t.Errorf("Daily = %+v, want %+v", days, want)
	}
}

func TestMigrateIsIdempotent(t *testing.T) {
	db := newDB(t) // already migrated once by Open
	if err := Migrate(context.Background(), db); err != nil {
		t.Fatalf("second migration run: %v", err)
	}
}
```

`internal/cache/cache_test.go`:

```go
package cache

import (
	"context"
	"errors"
	"io"
	"log/slog"
	"sync"
	"sync/atomic"
	"testing"
	"time"

	"example.com/shortener/internal/links"
)

// slowBackend is an in-memory backend whose Get is slow and counted.
type slowBackend struct {
	*links.Memory
	gets  atomic.Int64
	mu    sync.Mutex
	added map[string]int64
	fail  bool
}

func (b *slowBackend) Get(ctx context.Context, code string) (links.Link, error) {
	b.gets.Add(1)
	time.Sleep(20 * time.Millisecond)
	return b.Memory.Get(ctx, code)
}

func (b *slowBackend) AddHits(_ context.Context, hits map[string]int64) error {
	b.mu.Lock()
	defer b.mu.Unlock()
	if b.fail {
		return errors.New("database down")
	}
	for k, n := range hits {
		b.added[k] += n
	}
	return nil
}

func setup(t *testing.T) (*Links, *slowBackend) {
	t.Helper()
	b := &slowBackend{Memory: links.NewMemory(), added: map[string]int64{}}
	if err := b.Memory.Create(context.Background(), links.Link{Code: "go", URL: "https://go.dev"}); err != nil {
		t.Fatal(err)
	}
	log := slog.New(slog.NewTextHandler(io.Discard, nil))
	return New(b, NewMemoryKV(), time.Minute, log), b
}

func TestStampedeProtectionAndBatching(t *testing.T) {
	c, b := setup(t)
	ctx := context.Background()
	var wg sync.WaitGroup
	for range 100 {
		wg.Add(1)
		go func() {
			defer wg.Done()
			if url, err := c.Hit(ctx, "go"); err != nil || url != "https://go.dev" {
				t.Errorf("Hit = %q, %v", url, err)
			}
		}()
	}
	wg.Wait()
	if n := b.gets.Load(); n != 1 {
		t.Errorf("100 concurrent cold hits caused %d backend reads, want 1", n)
	}
	if err := c.Flush(ctx); err != nil {
		t.Fatal(err)
	}
	if b.added["go"] != 100 {
		t.Errorf("flushed %d hits, want 100", b.added["go"])
	}
}

func TestNegativeCaching(t *testing.T) {
	c, b := setup(t)
	for range 5 {
		if _, err := c.Hit(context.Background(), "nope"); !errors.Is(err, links.ErrNotFound) {
			t.Fatalf("err = %v", err)
		}
	}
	if n := b.gets.Load(); n != 1 {
		t.Errorf("5 misses for an unknown code caused %d backend reads, want 1", n)
	}
}

func TestFailedFlushKeepsHits(t *testing.T) {
	c, b := setup(t)
	ctx := context.Background()
	c.Hit(ctx, "go")
	c.Hit(ctx, "go")
	b.fail = true
	if err := c.Flush(ctx); err == nil {
		t.Fatal("expected flush error")
	}
	b.fail = false
	c.Hit(ctx, "go")
	if err := c.Flush(ctx); err != nil {
		t.Fatal(err)
	}
	if b.added["go"] != 3 {
		t.Errorf("hits after retry = %d, want 3", b.added["go"])
	}
}
```

Real results: `TEST_DATABASE_URL=postgres://postgres@/postgres?host=/tmp go test -race ./...` — all pass, including the integration tests against PostgreSQL 16.

### A real session with PostgreSQL and Redis

```
$ DATABASE_URL=postgres://postgres@/shortener?host=/tmp REDIS_ADDR=localhost:6379 ADDR=127.0.0.1:18081 shortener &
$ curl -s -X POST -d '{"url":"https://go.dev/doc/","code":"godocs"}' http://127.0.0.1:18081/api/links
{"code":"godocs","url":"https://go.dev/doc/","created_at":"2026-11-15T01:41:29.876921805Z","hits":0}
$ for i in $(seq 1 200); do curl -s -o /dev/null http://127.0.0.1:18081/godocs; done
$ curl -s http://127.0.0.1:18081/api/links/godocs
{"code":"godocs","url":"https://go.dev/doc/","created_at":"2026-11-15T01:41:29.876921Z","hits":200}
$ psql -tAc "select hits from links where code='godocs'"
1
$ curl -s "http://127.0.0.1:18081/api/links/godocs/daily?days=3"
[{"day":"2026-11-13","hits":0},{"day":"2026-11-14","hits":0},{"day":"2026-11-15","hits":200}]
$ redis-cli get link:godocs
https://go.dev/doc/
$ kill -INT %1
$ psql -tAc "select hits from links where code='godocs'"
200
```

Read it carefully, because it shows every design decision at work:

- The API reports **200** hits while the database row says **1**: 199 hits are still in memory, waiting for the next flush — and the stats endpoint adds them, so users see the right number (eventual consistency, hidden well).
- The `daily` call flushed first, so it shows all 200.
- The redirect target is in Redis.
- After `SIGINT`, the database has all **200**: the final flush ran after the HTTP server stopped.
- `created_at` lost its last three digits on the way through PostgreSQL (DT-250).

### Review checklist

- Run two instances against one database: are codes still unique? Are hit counts still exact? (They should be: increments are additive.)
- Stop Redis while the service runs: do redirects keep working (slower)?
- Kill the database for 10 seconds: do pending hits survive and arrive later? What do users see?
- Is every query parameterised? Is every `rows` closed? Is every call given a context?
- **Known limitations to discuss:** if an instance crashes, up to one second of hits is lost; cache entries can serve a deleted link for up to 10 minutes unless deletion invalidates the key; a single `links` row for a viral link becomes a write hotspot (batching helps; Chapter 12 discusses sharded counters).

## Answers to the Chapter 9 prediction questions

**9.1 Q2.** One row. `count(*)` counts all rows; `count(email)` counts rows where `email` is not NULL — equal here, because `email` is `NOT NULL`.

**9.2 Q2.** OFFSET must produce and discard the first 900,000 rows in order, so the index scan really walks 900,020 entries; only the last 20 are returned.

# Chapter 10 — Performance Engineering & Runtime Internals

*Goal: explain and prove performance instead of guessing.*

Performance work has one rule that matters more than all the techniques: **measure first**. Engineers are reliably wrong about where time goes; profiles are not. This chapter teaches the measuring tools (benchmarks done properly, pprof, execution traces), the compiler and runtime behaviour that explains what you measure (escape analysis, inlining, bounds checks, the garbage collector, the scheduler, how maps and slices and interfaces are built), and the optimisation techniques that follow from them.

The chapter's worked example is real: the Chapter 5 log analyzer, profiled and made **4.2× faster with 89% fewer allocations**, with every number below taken from actual runs.

**What you will be able to do after this chapter**

- Run benchmarks that produce trustworthy comparisons, and read `benchstat`.
- Profile CPU, heap, allocations, blocking, mutexes, and goroutines with pprof, and trace execution.
- Read escape-analysis, inlining, and bounds-check output, and a little assembly.
- Explain and tune the garbage collector (GOGC, GOMEMLIMIT) and the scheduler (G-M-P).
- Explain the internals of slices, strings, maps, interfaces, and channels.
- Use profile-guided optimisation, `runtime/metrics`, and GODEBUG — and know why to avoid `unsafe`.

## Topic 10A — Measurement

## Unit 10.1 — Measure first

**One-line summary:** Optimise only what a measurement shows to be slow, using realistic workloads, repeated runs, and statistical comparison. Decide in advance what "faster" means (latency? throughput? memory? at which percentile?) and stop when the goal is met.

### The loop

1. **Define the goal.** "p99 redirect latency under 5 ms at 2,000 requests/second" or "process a 1 GB log in under 10 seconds with less than 50 MB of memory". Without a target, optimisation never ends.
2. **Build a realistic benchmark** — realistic input sizes and distributions (Unit 4.5's warnings), ideally captured from production.
3. **Measure the baseline**, several runs (`-count 10`).
4. **Profile** to find where the time or memory goes (Unit 10.2). Not where you *think* it goes.
5. **Change one thing.**
6. **Measure again** and compare with `benchstat`. Keep the change only if the improvement is significant *and* worth its complexity.
7. Repeat until the goal is met.

### Benchmark rigour

- Use `b.Loop()` (Go 1.24+) so setup is excluded and results cannot be optimised away (Unit 4.5).
- Report allocations (`-benchmem` or `b.ReportAllocs()`): allocation counts are stable across runs and machines, unlike nanoseconds, and are often the root cause.
- Report throughput with `b.SetBytes(n)` for data-processing code (the log analyzer reports MB/s).
- Run on a quiet machine. Laptops change frequency with temperature and power; CI machines are shared. For decisions, use dedicated hardware or many runs.
- Use sub-benchmarks for several input sizes, because behaviour changes with size (caches, Unit 6.13).

### Reading `benchstat`

The real comparison from this chapter's project, 6 runs each:

```
       │   old.txt   │              new.txt               │
       │   sec/op    │   sec/op     vs base               │
Scan-2   9.768m ± 7%   2.306m ± 5%  -76.39% (p=0.002 n=6)

       │    old.txt    │               new.txt               │
       │     B/op      │     B/op      vs base               │
Scan-2   3991.9Ki ± 0%   958.7Ki ± 0%  -75.99% (p=0.002 n=6)

       │   old.txt   │              new.txt               │
       │  allocs/op  │  allocs/op   vs base               │
Scan-2   90.09k ± 0%   10.07k ± 0%  -88.82% (p=0.002 n=6)
```

- The first number in each column is the **median**; `± 7%` is the spread (a confidence interval).
- `p=0.002` is the probability of seeing a difference this large by chance if there were no real difference. Below 0.05 is conventionally "significant".
- A result shown as `~` means **no significant difference** — as in the PGO experiment of Unit 10.9 (`~ (p=0.132 n=6)`). Do not claim a speed-up that benchstat calls `~`.

### Deep & Tricky

**DT-262. Microbenchmarks can mislead about whole programs.** *(Measurement fact)* A function 3× faster in isolation may make the service 0.1% faster if it was 0.03% of the time; it can even make it slower if the "optimisation" increased memory use and GC work elsewhere. Validate important changes with whole-program measurements (load tests, Unit 14.6) as well as microbenchmarks.

### Mastery check

1. **Explain.** What do `± 5%` and `p=0.002` mean in benchstat output? When should you not trust a result?
2. **Write.** A performance goal statement for the Chapter 9 shortener's redirect endpoint, including the metric, percentile, load, and hardware.

## Unit 10.2 — Profiling with pprof

**One-line summary:** pprof samples where a program spends CPU time, allocates memory, blocks, or contends on locks, and attributes it to call stacks. Collect profiles from benchmarks (`-cpuprofile`, `-memprofile`) or from live services (`net/http/pprof`), then explore with `go tool pprof` (`top`, `list`, `web`, flame graphs).

### Collecting

From a benchmark:

```
go test -run XXX -bench Scan -cpuprofile cpu.out -memprofile mem.out .
go tool pprof logscan.test cpu.out
```

From a running service — import the handler package for its side effect:

```go
import _ "net/http/pprof" // registers /debug/pprof/* on http.DefaultServeMux
```

…and serve `http.DefaultServeMux` **on a separate, internal-only port** (never on the public listener: profiles reveal internals and profiling costs CPU):

```go
go func() { log.Println(http.ListenAndServe("localhost:6060", nil)) }()
```

```
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30   # CPU, 30 s
go tool pprof http://localhost:6060/debug/pprof/heap                  # live heap
go tool pprof http://localhost:6060/debug/pprof/allocs                # all allocations since start
go tool pprof http://localhost:6060/debug/pprof/goroutine             # where every goroutine is
go tool pprof http://localhost:6060/debug/pprof/block                 # where goroutines block (enable with runtime.SetBlockProfileRate)
go tool pprof http://localhost:6060/debug/pprof/mutex                 # lock contention (runtime.SetMutexProfileFraction)
go tool pprof http://localhost:6060/debug/pprof/goroutineleak         # Go 1.27: leaked goroutines (Unit 7.8)
```

### The profile types

| Profile | Question it answers |
| --- | --- |
| CPU | Where is CPU time spent? (Samples the running stack ~100 times per second.) |
| heap (`inuse_space`) | What is holding memory *right now*? (Leaks, big caches.) |
| allocs (`alloc_space`, `alloc_objects`) | What allocates the most over time? (GC pressure.) |
| block | Where do goroutines wait on channels, mutexes, selects? |
| mutex | Which locks are contended, and who holds them? |
| goroutine | What is every goroutine doing? (Leaks, deadlocks, pile-ups.) |

### Reading `top`

The real CPU profile of the original log analyzer (`go tool pprof -top`):

```
      flat  flat%   sum%        cum   cum%
     570ms  7.71%  7.71%      870ms 11.77%  strings.Fields
     320ms  4.33% 12.04%      980ms 13.26%  fmt.(*ss).ReadRune
     290ms  3.92% 15.97%      290ms  3.92%  runtime.memclrNoHeapPointers
     200ms  2.71% 21.52%      970ms 13.13%  runtime.mallocgcSmallScanNoHeader
     ...
     150ms  2.03% 39.92%     2300ms 31.12%  fmt.(*ss).scanOne
     140ms  1.89% 47.63%     2710ms 36.67%  fmt.(*ss).doScanf
```

- **flat:** time spent in the function itself.
- **cum:** time in the function *and everything it called*.
- Sort by cumulative (`-top -cum`) to find the expensive *subtrees* in your own code:

```
         0     0%     0%      6.44s 87.14%  example.com/logscan.run
     0.02s  0.27%  0.27%      6.01s 81.33%  example.com/logscan.scan
     0.02s  0.27%  0.54%      5.46s 73.88%  example.com/logscan.parseLine
         0     0%  2.03%      3.64s 49.26%  fmt.Sscanf (inline)
     0.57s  7.71% 27.88%      0.87s 11.77%  strings.Fields
         0     0% 39.78%      0.25s  3.38%  time.Parse
```

The diagnosis jumps out: **half of all CPU time** is `fmt.Sscanf` parsing two integers. Most developers would have guessed `time.Parse` (3.4%).

Inside pprof's interactive mode: `top`, `top -cum`, `list parseLine` (annotated source with time per line), `peek`, `web` (call graph in a browser), or run `go tool pprof -http=:8081 cpu.out` for a web UI with **flame graphs** — the fastest way to see where time goes.

### The allocation profile

```
go tool pprof -sample_index=alloc_space -top logscan.test mem.out
      flat  flat%   sum%        cum   cum%
      53MB 27.11% 27.11%   149.01MB 76.21%  example.com/logscan.parseLine
   47.50MB 24.30% 51.40%    47.50MB 24.30%  strings.Fields
   43.50MB 22.25% 73.65%    43.50MB 22.25%  fmt.newScanState
   22.50MB 11.51% 85.16%    22.50MB 11.51%  bufio.(*Scanner).Text (inline)
   21.43MB 10.96% 96.12%    21.43MB 10.96%  example.com/logscan.newStats (inline)
```

`strings.Fields` allocates a new slice per line; `fmt.Sscanf` allocates scanner state per call; and `parseLine` itself allocates because passing `&e.status` to `Sscanf` makes the whole `entry` escape to the heap (Unit 10.4).

### Deep & Tricky

**DT-263. CPU profiles miss time spent waiting.** *(Profiler behaviour)* The CPU profiler samples goroutines that are *running*. A request that takes 2 seconds because it waits 1.9 seconds for a database shows almost nothing in a CPU profile. For latency, look at block and mutex profiles, execution traces (Unit 10.3), and distributed traces (Unit 11.5).

**DT-264. Profile labels attribute samples to requests or tenants.** *(Library feature)* `pprof.Do(ctx, pprof.Labels("endpoint", "redirect"), func(ctx context.Context) {...})` tags samples taken while the function runs; `pprof -tagfocus` filters by label. Since Go 1.27 these labels also appear in goroutine tracebacks.

### Mastery check

1. **Explain.** What is the difference between flat and cumulative time? Why sort by cumulative first?
2. **Write.** Add a pprof endpoint on `localhost:6060` to the shortener, generate load with a loop of requests, capture a 10-second CPU profile, and list the top five functions by cumulative time.

## Unit 10.3 — Execution tracing with go tool trace

**One-line summary:** The execution tracer records every scheduling event — goroutines starting, blocking, unblocking, running on which processor, GC phases, system calls — with nanosecond timestamps. `go tool trace` shows them on a timeline. Use it for latency problems that profiles cannot explain: goroutines waiting to be scheduled, GC pauses at bad moments, poor parallelism.

### Collecting and viewing

```
go test -run TestX -trace trace.out
curl -o trace.out http://localhost:6060/debug/pprof/trace?seconds=5
go tool trace trace.out             # opens a browser UI (Go 1.27: listens on localhost only by default)
```

Or in code: `trace.Start(f)` … `trace.Stop()` from `runtime/trace`. Since Go 1.25, a **flight recorder** (`trace.FlightRecorder`) keeps the last few seconds of trace in memory, so a service can dump the trace *after* noticing a slow request — catching rare events without tracing all the time.

### What to look for

- **Goroutine analysis:** for each goroutine type, how long it spent executing, waiting on the network, blocked on sync, waiting for the scheduler ("scheduler latency"). High scheduler latency means too much runnable work for too few Ps.
- **Timeline:** are all Ps busy (good parallelism) or mostly idle while work waits on a lock?
- **GC:** when do GC cycles run, how long do they take, and which goroutines were drafted into "mark assist" (doing GC work instead of their own because they allocate fast)?
- **User annotations:** `trace.WithRegion(ctx, "parse", func() {...})` and `trace.NewTask` mark your own operations on the timeline.

### Deep & Tricky

**DT-265. Tracing is more expensive than profiling, but cheaper than it used to be.** *(Implementation detail)* Go 1.21 cut tracing overhead to a few percent of CPU (from 10–20%), and Go 1.22 redesigned the trace format to be streamable. Short traces in production are now reasonable; the flight recorder makes it routine.

### Mastery check

1. **Explain.** Give an example of a latency problem a CPU profile would not reveal but a trace would.
2. **Write.** Trace the Chapter 7 worker pool with 2 and with 8 workers on a CPU-heavy job; compare processor utilisation on the timeline.

## Topic 10B — Compiler and allocations

## Unit 10.4 — Escape analysis

**One-line summary:** The compiler places a value on the stack when it can prove the value does not outlive its function; otherwise it **escapes** to the heap. Heap allocations cost allocation time and GC work. `-gcflags=-m` shows the decisions; reducing needless escapes is one of the most effective Go optimisations.

### Common reasons values escape

| Cause | Example |
| --- | --- |
| Returning a pointer to a local | `return &p` (Unit 2.12) |
| Storing a pointer in something long-lived | appending `&x` to a package-level slice or a struct field that escapes |
| Converting to an interface | `fmt.Println(x)`, `any(x)`, storing in `[]any` (Unit 3.5) |
| Closures that capture variables and outlive the function | goroutines, returned functions (DT-86) |
| Passing a pointer to a function the compiler cannot see through | `fmt.Sscanf(s, "%d", &e.status)` — `Sscanf` takes `...any` |
| Sizes unknown at compile time | `make([]byte, n)` with a variable `n` (usually) |
| Very large values | locals over ~64 KB (DT-42) |

The log analyzer's `parseLine` allocated 53 MB in its own frame for exactly the `Sscanf` reason: `&e.status` was passed as an `any`, so `e` escaped — every line allocated an `entry` on the heap. Replacing `Sscanf` with `strconv.Atoi` let `e` stay on the stack.

### Reading `-gcflags=-m`

```
go build -gcflags=-m ./...          # one level of detail
go build -gcflags='-m -m' ./...     # explains WHY something escapes
```

Messages: `moved to heap: x` (a variable became a heap allocation), `x escapes to heap` (a value is copied to the heap, usually through an interface), `does not escape` (good), `can inline f`, `inlining call to f`. Use them to *confirm* what a memory profile shows; do not try to make every line "not escape".

### Techniques to reduce allocations

- Avoid interface conversions in hot paths (`strconv` instead of `fmt` for single values).
- Reuse buffers: `buf = buf[:0]`, `strings.Builder.Reset`, `sync.Pool` (Unit 7.11), `strconv.AppendInt(buf, n, 10)` instead of `Itoa`.
- Pre-size slices and maps (P-17).
- Use fixed-size arrays for small, bounded collections (the optimised `parseLine` uses `var f [6]string`).
- Use iterators instead of building intermediate slices (`strings.FieldsSeq` instead of `strings.Fields`).
- Pass small structs by value (Unit 2.12) rather than pointers that escape.
- Avoid `[]byte` ↔ `string` conversions in loops where possible (DT-114 lists the free cases).

### Deep & Tricky

**DT-266. Inlining changes escape decisions.** *(Compiler behaviour)* A function that returns `&p` makes `p` escape — unless the call is inlined into a caller that does not let the pointer escape, in which case the allocation disappears. That is why `-m` output for the same function can differ between call sites, and why Unit 2.12 needed `//go:noinline` to show the escape reliably.

**DT-267. Go 1.26 can put more slice backing arrays on the stack.** *(Version-dependent optimisation)* Recent compilers allocate small `make([]T, n)` buffers on the stack in more situations, even with variable `n` (using a fixed-size stack buffer when `n` is small). Always re-measure after upgrading Go: some of your manual optimisations may become unnecessary.

### Mastery check

1. **Explain.** Why does passing `&e.status` to `fmt.Sscanf` make the whole `e` escape?
2. **Write.** Find the top allocation site in one of your projects with `-memprofile`, confirm with `-gcflags='-m -m'`, remove it, and show the `benchstat` result for allocs/op.

## Unit 10.5 — The compiler

**One-line summary:** The gc compiler parses and type-checks your code, converts it to **SSA** (static single assignment) form, optimises it (inlining, constant folding, dead-code elimination, bounds-check elimination, escape analysis), and generates machine code for the target architecture. Knowing its main optimisations helps you write code it can make fast — and read the evidence.

### Inlining

Small functions are copied into their callers, removing call overhead and enabling further optimisation (escape analysis, constant folding across the boundary). The compiler uses a cost budget (roughly 80 "nodes"); functions with loops, `defer`, `recover`, or `select` were historically harder to inline. PGO (Unit 10.9) raises the budget for hot call sites. See decisions with `-gcflags=-m`:

```
./stats.go:47:6: can inline (*stats).percentile
./stats.go:90:29: inlining call to (*stats).percentile
```

### Bounds-check elimination (BCE)

Every slice index is checked (DT-99), but the compiler removes checks it can prove unnecessary. Real output of `-gcflags=-d=ssa/check_bce/debug=1` for this file:

```go
func sumFirst4(s []int) int {
	return s[0] + s[1] + s[2] + s[3] // four bounds checks
}

func sumFirst4Hint(s []int) int {
	_ = s[3] // one check up front proves the other indexes are in range
	return s[0] + s[1] + s[2] + s[3]
}

func sumAll(s []int) int {
	t := 0
	for i := range s {
		t += s[i] // the compiler proves i < len(s): no check
	}
	return t
}
```

```
./b.go:4:10: Found IsInBounds
./b.go:4:17: Found IsInBounds
./b.go:4:24: Found IsInBounds
./b.go:4:31: Found IsInBounds
./b.go:8:7: Found IsInBounds
```

Four checks in `sumFirst4`, one in the hinted version (line 8, the `_ = s[3]`), none in `sumAll`. The standard library's `encoding/binary` uses exactly this `_ = b[7]` hint before reading 8 bytes. A bounds check costs very little when predicted correctly; this matters only in the tightest loops — measure.

### Reading assembly

`go build -gcflags=-S` prints the generated assembly (Go's own assembler syntax). For `func add(a, b int) int { return a + b }` on amd64:

```
TEXT	bce.add(SB), NOSPLIT|NOFRAME|ABIInternal, $0-16
ADDQ	BX, AX
RET
```

One instruction and a return: the arguments arrived in registers AX and BX, and the result is returned in AX — the register-based calling convention of DT-38, visible. `NOSPLIT` means the function needs no stack-growth check (it uses no stack). You will rarely write assembly, but reading it settles arguments about what the compiler does. `go tool objdump` disassembles binaries; the Compiler Explorer (godbolt.org) shows Go assembly side by side with source.

### SSA

`GOSSAFUNC=parseLine go build` writes `ssa.html`, showing the function after each of dozens of optimisation passes — the deepest view of what the compiler did. Useful for understanding why an optimisation did or did not happen.

### Mastery check

1. **Explain.** Why does `for i := range s { s[i] }` need no bounds check, while `for i := 0; i <= n; i++ { s[i] }` does?
2. **Write.** Write a function that reads a big-endian `uint32` from `b []byte` at offset `i`, check its bounds checks, and reduce them to one with a hint.

## Topic 10C — Runtime internals

## Unit 10.6 — Garbage collector

**One-line summary:** Go's GC is a concurrent, tri-colour, mark-and-sweep collector: it finds live objects by tracing from roots while your program keeps running, with only very short stop-the-world pauses. Its cost is CPU time (proportional to live heap and pointer density) and memory headroom. `GOGC` trades memory for CPU; `GOMEMLIMIT` sets a soft memory ceiling.

### How it works (enough to reason about it)

1. **Mark:** starting from roots (goroutine stacks, globals, registers), the GC marks every reachable object. Tri-colour: white (not yet seen), grey (seen, children not yet scanned), black (scanned). A **write barrier** keeps the invariant correct while your goroutines modify pointers during marking.
2. **Mark assist:** a goroutine that allocates quickly during marking is made to do some marking work itself, so allocation cannot outrun the collector. This shows up as latency in allocation-heavy code.
3. **Sweep:** memory of unmarked objects is reclaimed, lazily, as new allocations need it.
4. **Pacing:** a new cycle starts when the heap grows to a **target**: by default `live heap × (1 + GOGC/100)`, i.e. with `GOGC=100`, when the heap doubles since the last cycle's live size.

Pauses are typically well under a millisecond. Go 1.26 made the **Green Tea** collector the default: it scans memory in page-sized spans rather than object by object, improving locality and cutting GC CPU cost by 10–40% on many workloads.

### Reading a GC trace

`GODEBUG=gctrace=1` prints one line per cycle. Real output from the optimised log analyzer:

```
gc 1 @0.012s 1%: 0.070+0.82+0.049 ms clock, 0.14+0.079/0.097/0.54+0.098 ms cpu, 3->3->1 MB, 4 MB goal, 0 MB stacks, 0 MB globals, 2 P
```

- `gc 1 @0.012s 1%`: cycle 1, 12 ms after start, GC has used 1% of CPU so far.
- `0.070+0.82+0.049 ms clock`: stop-the-world sweep termination + concurrent mark + stop-the-world mark termination. The pauses are the first and last numbers: 70 µs and 49 µs.
- `3->3->1 MB`: heap at cycle start → at mark end → live after marking.
- `4 MB goal`: the heap size that triggers the next cycle.

### GOGC and GOMEMLIMIT — measured

The log analyzer processing a 51 MB, 1,000,000-line file, real measurements:

| Setting | Time | Peak memory (RSS) | GC cycles |
| --- | --- | --- | --- |
| `GOGC=25` | 0.32 s | 5.9 MB | 61 |
| `GOGC=100` (default) | 0.31 s | 7.8 MB | 22 |
| `GOGC=400` | 0.30 s | 21.8 MB | 4 |
| `GOGC=off` | 0.31 s | 64.9 MB | 0 |

Two lessons. First, the trade-off is real: more GC cycles ↔ less memory. Second, **the run time barely changed** — because the optimisation of Unit 10.2 had already removed 89% of the allocations, the GC had little to do. Reducing allocations beats tuning the collector.

- **`GOGC`** (default 100): higher = fewer GC cycles and more memory; `off` disables collection (only for short batch jobs that fit in memory).
- **`GOMEMLIMIT`** (Go 1.19): a soft limit on total Go memory, such as `GOMEMLIMIT=900MiB` in a container with 1 GiB. The GC runs more often as the limit approaches, avoiding out-of-memory kills. Combined with `GOGC=off`, the GC runs only when needed to stay under the limit. If live data genuinely exceeds the limit, the GC can run continuously ("death spiral" — the runtime caps GC CPU to about 50% to keep the program making progress). Set limits with headroom.
- Both can be set at run time with `debug.SetGCPercent` and `debug.SetMemoryLimit`.

### Reducing GC cost

GC work is proportional to the **live heap** and the number of **pointers** to scan; allocation *rate* sets how often it runs. So: allocate less (Unit 10.4), keep fewer long-lived pointers (a `[]int32` of node IDs instead of `[]*Node`; the GC does not scan memory without pointers — Unit 13.8 uses this), and size `GOMEMLIMIT` to the container.

### Deep & Tricky

**DT-268. Finalizers and `runtime.AddCleanup` are not destructors.** *(Runtime behaviour)* `runtime.SetFinalizer` (and its safer Go 1.24 replacement `runtime.AddCleanup`) run some time after an object becomes unreachable — or never, if the program exits first. Never rely on them to close files or release locks; use `defer` and explicit `Close`. They are a safety net for leaked resources, nothing more.

**DT-269. A large live heap of pointer-free data is nearly free for the GC.** *(Implementation behaviour)* Memory allocated for types without pointers (`[]byte`, `[]float64`, structs of numbers) is marked "no scan": the GC marks the block live without reading it. A 2 GB `[]int32` graph costs almost nothing per GC cycle; 2 GB of `*Node` objects costs a full scan every cycle.

### Mastery check

1. **Explain.** Why did changing `GOGC` hardly change the run time of the optimised analyzer? Predict what it would have done to the *original* version.
2. **Explain.** What does `GOMEMLIMIT` protect against in a container, and what is the risk of setting it too close to the live heap size?
3. **Write.** Run the *original* analyzer with `GODEBUG=gctrace=1` on the same file and compare cycle counts and GC CPU percentage with the optimised version.

## Unit 10.7 — Scheduler

**One-line summary:** The Go scheduler multiplexes many goroutines (G) onto a few OS threads (M), each running Go code only while holding one of `GOMAXPROCS` processors (P). Each P has a local run queue; idle Ps **steal** work from others; long-running goroutines are **preempted**; blocking system calls hand off the P so other goroutines keep running.

### G, M, P in more detail

- **P** owns a local run queue of up to 256 runnable goroutines, plus a "next" slot (a goroutine just woken by the running one runs next, which is good for ping-pong communication). A global run queue holds overflow.
- **Scheduling points:** a goroutine yields when it blocks (channel, mutex, network, sleep), calls `runtime.Gosched`, or is preempted. Since Go 1.14, preemption is **asynchronous**: the runtime signals the thread, so even a tight loop without function calls yields after about 10 ms.
- **Work stealing:** an idle P checks its local queue, then the global queue, then the network poller, then tries to steal half the run queue of a random other P. This keeps all cores busy without a central lock.
- **System calls:** before a blocking syscall (file I/O), the M releases its P; if the syscall takes long, another M picks up the P and continues running goroutines. When the syscall returns, the M tries to get a P back.
- **Network poller:** sockets are non-blocking; a goroutine waiting on network I/O is parked, and the poller (epoll/kqueue/IOCP) wakes it when data arrives. Thousands of idle connections cost no threads.

### Stack growth

Every function prologue checks whether the stack has room for its frame (unless it is `NOSPLIT`, as in Unit 10.5's assembly). If not, the runtime allocates a stack twice as large, copies the old one, and adjusts pointers into it (DT-35). Copying is O(stack size), so deep recursion that repeatedly grows and shrinks can cost noticeable time; it is rarely an issue.

### GOMAXPROCS

Defaults to the number of CPUs the process can use. Since Go 1.25, on Linux the default also respects the container's **CPU limit** (cgroup quota) and updates if it changes. Before 1.25, a container limited to 2 CPUs on a 64-core host got `GOMAXPROCS=64`, causing heavy throttling and latency spikes — the reason the `uber-go/automaxprocs` library existed. Setting `GOMAXPROCS` explicitly overrides the default.

### Deep & Tricky

**DT-270. `runtime.LockOSThread` pins a goroutine to its thread.** *(Runtime API)* Needed when calling C libraries or OS APIs that use thread-local state (some graphics libraries, Linux namespaces). The thread is not used for other goroutines until unlocked. Rarely needed elsewhere, and a source of thread explosion if misused.

**DT-271. Too many runnable goroutines raise latency, not throughput.** *(Scheduling fact)* With 8 Ps and 10,000 runnable CPU-bound goroutines, each waits its turn; the work finishes in the same total time, but individual requests see long scheduler delays. Bound CPU-bound concurrency to roughly GOMAXPROCS (worker pools, Unit 7.10); unbounded concurrency only helps I/O-bound work.

### Mastery check

1. **Explain.** What happens, step by step, when a goroutine reads from a network socket that has no data?
2. **Explain.** Why did containers need `automaxprocs` before Go 1.25?

## Unit 10.8 — Data-structure internals

**One-line summary:** The built-in types have simple, knowable representations: slices are (ptr, len, cap), strings are (ptr, len), interfaces are (type/itab, data), maps are Swiss tables, channels are a locked ring buffer with queues of waiting goroutines. Knowing them explains their costs.

| Type | Representation (64-bit) | Cost implications |
| --- | --- | --- |
| slice | 24 bytes: pointer, length, capacity (Unit 2.2) | cheap to pass; growth copies (Unit 2.3); sub-slices pin memory (Unit 2.4) |
| string | 16 bytes: pointer, length | immutable, so substrings share bytes; conversions to `[]byte` copy |
| array | the elements themselves | copied on assignment; no indirection |
| interface | 16 bytes: `itab` pointer (type + method table) or type pointer for `any`, plus a data pointer | method calls are indirect (DT-149); boxing non-pointer values often allocates |
| map | pointer to a runtime structure: since Go 1.24 a Swiss table (groups of 8 slots with a control word of 7-bit hash fragments, matched several at a time), split into tables that grow independently | O(1) average; iteration order randomised; memory not returned on delete (Unit 2.8) |
| channel | pointer to `hchan`: a mutex, a circular buffer of `cap` elements, send and receive wait queues | every operation takes a lock; unbuffered sends hand the value directly to a waiting receiver |
| func value | pointer to a closure object: code pointer + captured variables | calling through it is indirect; captured variables may live on the heap |

Practical consequences you have already met: `[]*T` vs `[]T` (Unit 2.12), why channels are slower than mutexes for simple shared counters (lock plus queue management), why a map of a million entries is many separate memory regions while a sorted slice is one (DT-217), why `any(x)` in a hot loop allocates.

### Deep & Tricky

**DT-272. A channel operation costs tens of nanoseconds; a mutex lock/unlock costs about ten when uncontended.** *(Implementation behaviour; measure on your hardware)* Channels are the right tool for communication and coordination; for protecting a counter or a map in a hot path, a mutex or an atomic is several times cheaper. Choose by design first (P-32), and let measurements settle hot spots.

### Mastery check

1. **Explain.** Using the representations above, explain why `for _, v := range []any{...}` with a type switch is slower than ranging over a concrete `[]int`.
2. **Write.** Benchmark sending 1,000,000 integers through an unbuffered channel, a buffered channel (capacity 1024), and a mutex-protected slice. Explain the results with the `hchan` design.

## Topic 10D — Advanced tools

## Unit 10.9 — Profile-guided optimisation (PGO)

**One-line summary:** Give the compiler a CPU profile from a representative production workload (`default.pgo` in the main package directory), and it inlines more aggressively at hot call sites and devirtualises hot interface calls. Typical gains are a few percent, with no code changes — but measure, because some workloads gain nothing.

### Using PGO

1. Collect a CPU profile from production or a realistic load test (`/debug/pprof/profile?seconds=30`).
2. Save it as `default.pgo` in the main package's directory and commit it.
3. Build normally: `go build` uses `default.pgo` automatically since Go 1.21 (`-pgo=auto`). `-pgo=off` disables it.
4. Refresh the profile periodically; the compiler tolerates source changes, but a stale profile helps less.

### Measured on this chapter's project

Using the optimised analyzer's own CPU profile:

```
       │   new.txt   │           pgo.txt            │
       │   sec/op    │   sec/op     vs base         │
Scan-2   2.306m ± 5%   2.253m ± 5%  ~ (p=0.132 n=6)
```

**No significant difference.** The hot path (`strings.FieldsSeq`, `time.Parse`, map updates) was already inlined or dominated by work PGO cannot change. This is a useful, honest result: PGO helps most in large programs with deep call chains and hot interface calls (servers, compilers — the Go team reports 2–14% on such workloads), and little in small, already-tight loops. Do not claim a PGO speed-up without benchstat showing one.

### Mastery check

1. **Explain.** Why does PGO help with interface-heavy code (hint: devirtualisation, DT-149)?
2. **Write.** Apply PGO to the Chapter 9 shortener with a profile captured under load, and report the benchstat comparison of a redirect benchmark — whatever the result.

## Unit 10.10 — Low-level tools

**One-line summary:** `unsafe` breaks Go's memory-safety guarantees and should be a last resort behind a safe API and thorough tests. `runtime/metrics` exposes the runtime's internal measurements for monitoring. `GODEBUG` settings switch runtime and library behaviours for debugging and compatibility.

### `unsafe`

`unsafe.Pointer` can convert between any pointer types; `unsafe.Add`, `unsafe.Slice`, `unsafe.String`, `unsafe.StringData`, `unsafe.SliceData` (Go 1.17–1.20) build slices and strings over raw memory without copying. Legitimate uses: zero-copy conversions in hot paths of serialization libraries, interoperating with C or memory-mapped files (Unit 13.8), and inspecting layouts. The costs:

- No compiler or runtime protection: a wrong length or alignment silently corrupts memory or crashes later, far from the cause.
- Code may break with new Go versions (the rules for valid `unsafe.Pointer` patterns are narrow; `go vet`'s `unsafeptr` check enforces some of them).
- `unsafe.String(ptr, n)` over a `[]byte` that is later modified violates string immutability — invisible bugs in maps and comparisons.

Rule: isolate `unsafe` in a tiny package, document why, test heavily, and benchmark to prove it is worth it. Go 1.27's `go fix` has an `unsafefuncs` modernizer that replaces old `reflect.SliceHeader` tricks with the safer functions.

### `runtime/metrics`

```go
samples := []metrics.Sample{
	{Name: "/gc/heap/live:bytes"},
	{Name: "/sched/goroutines:goroutines"},
	{Name: "/gc/cycles/total:gc-cycles"},
	{Name: "/sched/latencies:seconds"}, // a histogram of scheduler latency
}
metrics.Read(samples)
for _, s := range samples {
	switch s.Value.Kind() {
	case metrics.KindUint64:
		fmt.Println(s.Name, s.Value.Uint64())
	case metrics.KindFloat64Histogram:
		fmt.Println(s.Name, "histogram with", len(s.Value.Float64Histogram().Counts), "buckets")
	}
}
```

`metrics.All()` lists every available metric with its description. Prometheus's Go client exports them automatically (Unit 11.5). Prefer `runtime/metrics` over the older `runtime.ReadMemStats`, which stops the world.

### `GODEBUG`

`GODEBUG=name=value,...` changes runtime and library behaviours: `gctrace=1` (Unit 10.6), `schedtrace=1000` (scheduler state every second), `inittrace=1` (time spent in each package's `init`), `madvdontneed=1`, and many **compatibility settings** that let a program keep old behaviour after a Go upgrade (for example, HTTP or TLS defaults). A module can set defaults with `godebug` lines in `go.mod` or `//go:debug` directives in the main package. Go 1.27 accepts removed settings in those places only if they are set to their final default value.

### Deep & Tricky

**DT-273. `unsafe` conversions can outlive their safety.** *(Classic bug)* `s := unsafe.String(&b[0], len(b))` creates a string sharing `b`'s memory. If `b` is later reused (a pooled buffer!), `s` changes — while sitting as a key in a map that hashed the old contents. The resulting bugs look like map corruption and appear far from the cause.

### Mastery check

1. **Explain.** Why is `runtime.ReadMemStats` in a frequently called metrics handler a bad idea, and what replaces it?
2. **Write.** A `/debug/runtime` endpoint (internal port only) that reports live heap, goroutine count, GC cycles, and the p99 of `/sched/latencies:seconds` computed from the histogram buckets.

## Chapter 10 Project — Profile and optimise with evidence

**Goal:** apply the full measure–profile–change–verify loop to two earlier projects, and write up the results like a professional performance report.

### Part 1 — The log analyzer (worked example)

This part is fully worked; repeat it yourself and compare your numbers.

**Baseline.** `go test -run XXX -bench Scan -benchmem -count 6 -cpuprofile cpu.out . > old.txt` on 10,000 generated lines: about **9.8 ms per run, 44 MB/s, 90,089 allocations**.

**Profile.** The cumulative CPU view (Unit 10.2) shows `fmt.Sscanf` at **49%** of CPU and `strings.Fields` at 12%; the allocation profile shows `strings.Fields`, `fmt.newScanState`, and `parseLine`'s own frame (because `&e.status` passed to `Sscanf` makes `e` escape) as the top allocators.

**Change.** Replace `strings.Fields` with a fixed array filled from `strings.FieldsSeq`, and `fmt.Sscanf` with `strconv.Atoi` / `strconv.ParseInt`:

```go
func parseLine(line string) (entry, error) {
	var f [6]string // fixed-size array: no allocation (Unit 2.1)
	n := 0
	for field := range strings.FieldsSeq(line) { // Go 1.24 iterator: no slice built
		if n == len(f) {
			n++
			break
		}
		f[n] = field
		n++
	}
	if n != len(f) {
		return entry{}, errFields
	}
	var e entry
	var err error
	if e.time, err = time.Parse(time.RFC3339, f[0]); err != nil {
		return entry{}, err
	}
	e.method, e.path = f[1], f[2]
	if e.status, err = strconv.Atoi(f[3]); err != nil || e.status < 100 || e.status > 599 {
		return entry{}, fmt.Errorf("bad status %q", f[3])
	}
	if e.duration, err = time.ParseDuration(f[4]); err != nil {
		return entry{}, err
	}
	if e.bytes, err = strconv.ParseInt(f[5], 10, 64); err != nil || e.bytes < 0 {
		return entry{}, fmt.Errorf("bad byte count %q", f[5])
	}
	return e, nil
}
```

(The error for a wrong field count no longer reports how many fields were found — a deliberate small trade-off; the tests still pass because they check that an error occurs, not its text.)

**Verify.** All tests pass, and `benchstat old.txt new.txt`:

```
Scan-2   9.768m ± 7%   2.306m ± 5%  -76.39% (p=0.002 n=6)     sec/op
Scan-2   43.61Mi ± 7%  184.70Mi ± 5% +323.55% (p=0.002 n=6)    B/s
Scan-2   3991.9Ki ± 0% 958.7Ki ± 0%  -75.99% (p=0.002 n=6)     B/op
Scan-2   90.09k ± 0%   10.07k ± 0%   -88.82% (p=0.002 n=6)     allocs/op
```

**4.2× faster, 89% fewer allocations** — from two small, local changes found by a profile, not by guessing.

**Re-profile.** The new top is `strings.FieldsSeq`'s inlined loop (26%), then `time.ParseDuration` (10%), map assignment, and `time.Parse`. Each is now a reasonable cost; the next steps would be a hand-written field splitter for the fixed format, and `sc.Bytes()` instead of `sc.Text()` to avoid one string allocation per line (keeping the map key conversion for new paths only). Stop when the goal is met.

**Whole-program check.** The optimised binary processes a 51 MB, 1,000,000-line file in **0.31 s** with a peak of **7.8 MB** of memory (`GOGC=100`), confirming the constant-memory design under real I/O.

**PGO.** No significant change (Unit 10.9). Reported anyway.

### Part 2 — The shortener's redirect path (your turn)

1. Add `net/http/pprof` on `localhost:6060` to the Chapter 9 shortener.
2. Write a load generator in Go (a worker pool, Unit 7.10, of 50 goroutines sending `GET /{code}` for a set of codes for 30 seconds, recording latencies into the histogram idea of Chapter 5) — or use a tool such as `hey`, `oha`, or `vegeta`.
3. Measure throughput and p50/p99 latency with the in-process cache, with Redis, and with caching disabled (the plain PostgreSQL store).
4. Capture CPU, allocation, block, and mutex profiles under load. Find the top three costs. (Expect to see JSON logging of every request, the `slog` handler, and the `pending` mutex near the top.)
5. Make at most two changes, and prove each with before/after numbers.

### The report

Write one page per part: goal, environment (CPU, Go version, data), baseline with variance, profile evidence (the relevant lines of `top -cum`), each change and its benchstat result, the final whole-program measurement, what you decided *not* to optimise and why, and remaining risks. This is the format senior engineers use to justify performance work in design reviews.

### Review checklist

- Every claim has a number, every number has a variance, every comparison has a p-value.
- Profiles were captured on realistic data and sizes.
- Optimisations kept the code clear, and the tests still pass.
- Allocation counts were reduced before GC tuning was considered.
- Nothing `unsafe` without overwhelming evidence.

## Answers to the Chapter 10 prediction questions

**10.6 Q1.** The optimised version allocates little, so the GC had little work whatever its setting. The original allocated about nine times more objects per line; with it, low `GOGC` would have caused many more cycles and visible extra CPU time, and `GOGC=off` would have used several times more memory before finishing.

**10.5 Q1.** In `for i := range s`, the compiler knows `0 <= i < len(s)` by construction. In `for i := 0; i <= n; i++`, nothing relates `n` to `len(s)` (and `i <= n` even allows `i == n`), so every access is checked — and may panic.

# Chapter 11 — Production Engineering

*Goal: turn working code into operable software.*

A service that works on your laptop is perhaps a third of the job. Production software must be configured without code changes, keep secrets secret, resist attackers, know who is calling and what they may do, tell its operators what it is doing (logs, metrics, traces, health), ship as a small and safe artifact, be tested and checked automatically on every change, and start, scale, and stop cleanly under an orchestrator. This chapter takes the Chapter 9 shortener through all of it.

Everything Go-related in this chapter's project was built and run end to end; the container, CI, and Kubernetes files are standard configurations that could not be executed in the environment used to write this book, and are marked as such.

**What you will be able to do after this chapter**

- Load and validate configuration, and handle secrets safely.
- Apply the most important security practices for Go services: validation, injection, XSS/CSRF, TLS, password hashing, cryptography, dependency scanning.
- Implement authentication (sessions, JWTs, OAuth2/OIDC concepts, API keys) and authorisation.
- Make a service observable with structured logs, Prometheus metrics, OpenTelemetry traces, and health checks.
- Build minimal, secure container images; write a CI pipeline; deploy to Kubernetes with correct probes, resources, and graceful shutdown.

## Topic 11A — Configuration and security

## Unit 11.1 — Configuration and secrets

**One-line summary:** Configuration comes from the environment (and optionally flags or a file), is validated completely at startup, and fails fast with every problem listed at once. Secrets never live in code, images, or logs; read them from files mounted by the platform or from a secret manager.

### The twelve-factor rule

Store configuration that varies between deployments (database URLs, feature flags, limits) in the **environment**, not in code. The same binary and image then run in development, staging, and production. Flags are fine for CLIs; long-running services usually read environment variables because every platform (Docker, Kubernetes, systemd, CI) can set them.

### Load everything, validate everything, report everything

```go
// Load reads configuration using getenv (os.Getenv in production, a map in
// tests). Every problem is reported at once, not one per restart.
func Load(getenv func(string) string) (Config, error) {
	c := Config{
		Addr:          or(getenv("ADDR"), ":8080"),
		AdminAddr:     or(getenv("ADMIN_ADDR"), "127.0.0.1:9090"),
		...
	}
	var errs []error
	...
	if v := getenv("RATE_PER_SECOND"); v != "" {
		if c.RatePerSecond, err = strconv.ParseFloat(v, 64); err != nil || c.RatePerSecond <= 0 {
			errs = append(errs, fmt.Errorf("RATE_PER_SECOND: must be a positive number, got %q", v))
		}
	}
	...
	return c, errors.Join(errs...)
}
```

(The full package is in the project.) Design points:

- `getenv` is a **parameter**, so tests pass a map (P-28) — no global environment mutation, tests can run in parallel.
- Defaults are explicit and safe (the admin listener defaults to `127.0.0.1`).
- `errors.Join` reports **all** problems in one startup. Nothing is more frustrating than fixing one variable, redeploying, and learning about the next.
- A `String()` method prints the configuration for the startup log **without secrets** — and a test checks that it does not leak them.

### Secrets

- **Never** in source code, Git history, container images, or command-line arguments (visible in `ps`).
- Read from **files** mounted by the platform: the project supports `DATABASE_URL_FILE` and `API_KEY_HASHES_FILE` as alternatives to the plain variables, the convention used by Docker secrets and Kubernetes `Secret` volumes.
- Or fetch from a secret manager (Vault, AWS Secrets Manager, GCP Secret Manager) at startup, with the service's own identity — no secret needed to get secrets.
- Never log them; redact with `slog.LogValuer` (Unit 5.8); never return them in errors (Unit 3.9).
- Store hashes instead of secrets where possible: the project stores **SHA-256 hashes** of API keys, so a leaked configuration does not leak usable keys.
- Plan for **rotation**: accept several valid keys at once so a new key can be rolled out before the old one is removed.

### Mastery check

1. **Explain.** Why read secrets from files rather than environment variables, given both are set by the platform?
2. **Write.** Add a `FEATURE_CUSTOM_CODES` boolean (default true) to the config, validated with `strconv.ParseBool`, and make the API reject custom codes when it is false. Test both settings.

## Unit 11.2 — Security

**One-line summary:** Most security failures in services are ordinary bugs with serious consequences: trusting input, building queries or HTML from strings, weak password storage, misused cryptography, outdated dependencies, and leaking internal details. Go's standard library makes the safe path easy; know where it ends.

### Input validation

Validate at the edge (P-4): types, ranges, lengths, formats, allow-lists. Bound sizes everywhere (bodies, headers, lines, uploads — DT-191). Treat every byte from outside as hostile until checked: request bodies, headers, query strings, cookies, file names, data from other services and from your own database (it may contain old or injected data).

### Injection

- **SQL:** placeholders always (Unit 9.5).
- **Shell commands:** `exec.Command` with separate arguments, never `sh -c` with user data (Unit 5.7).
- **Paths:** never join user input into file paths without `filepath.IsLocal` or, better, `os.Root` (Go 1.24), which confines all access to one directory tree (Unit 5.3).
- **Templates:** `html/template` for HTML, which escapes by context (Unit 5.9).
- **Headers and logs:** reject or escape newline characters in values you put into headers or log lines; structured logging (slog JSON) escapes them for you.

### XSS and CSRF (for services that serve browsers)

- **Cross-site scripting (XSS):** attacker-controlled text runs as script in a victim's browser. Defences: `html/template` escaping; a `Content-Security-Policy` header that forbids inline scripts; `HttpOnly` cookies so scripts cannot read session tokens.
- **Cross-site request forgery (CSRF):** a malicious page makes the victim's browser send a state-changing request to your site with the victim's cookies. Defences: `SameSite=Lax` or `Strict` cookies; CSRF tokens; checking the `Origin`/`Sec-Fetch-Site` headers. Go 1.25 added `http.CrossOriginProtection`, a middleware that rejects cross-origin non-safe requests using those headers. APIs authenticated with bearer tokens in headers (like the project's API keys) are not exposed to CSRF, because browsers do not attach such headers automatically.

### TLS

Serve HTTPS (`srv.ListenAndServeTLS(certFile, keyFile)`, or `golang.org/x/crypto/acme/autocert` for automatic Let's Encrypt certificates), or terminate TLS at a load balancer and keep internal traffic on a private network (or mutual TLS between services). Go's defaults (TLS 1.2 minimum, modern cipher suites, post-quantum hybrid key exchange since Go 1.24) are good; do not lower them. Set `Strict-Transport-Security` so browsers refuse plain HTTP afterwards.

### Passwords

Never store passwords, and never store a plain fast hash (SHA-256) of them: GPUs try billions of guesses per second. Use a slow, salted password hash: **bcrypt** (`golang.org/x/crypto/bcrypt`) or **argon2id** (`golang.org/x/crypto/argon2`).

```go
hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
err = bcrypt.CompareHashAndPassword(hash, []byte(attempt)) // nil = match
```

Why may the project use plain SHA-256 for **API keys**? Because they are 128+ bits of `crypto/rand` randomness (P-15) — not guessable, so slowing guesses down adds nothing. Human-chosen passwords are short and predictable; that is what slow hashes are for.

### Cryptography: use the high-level packages

- Randomness for secrets: `crypto/rand` (P-15).
- Comparing secrets: `crypto/subtle.ConstantTimeCompare`, so response time does not reveal how many bytes matched (the project uses it).
- Encryption: an AEAD such as AES-GCM (`crypto/cipher`) or `golang.org/x/crypto/chacha20poly1305`, with unique nonces; or `crypto/hpke` (Go 1.26) for public-key encryption.
- Signatures: Ed25519 (`crypto/ed25519`); post-quantum ML-DSA arrived in Go 1.27 (`crypto/mldsa`).
- Message authentication: HMAC-SHA-256 (`crypto/hmac`).
- Never design your own scheme, reuse nonces, use ECB mode, or "encrypt" with base64.

Go's crypto is implemented in Go, audited, constant-time where it matters, and offers a FIPS 140-3 mode (Go 1.24+) for regulated environments.

### The supply chain

- `govulncheck` in CI reports known vulnerabilities in code paths you actually call (Unit 4.7).
- `go.sum` and the checksum database make dependency tampering detectable (Unit 3.13).
- Fewer dependencies = smaller attack surface (one reason for Chapter 5's "standard library first").
- Pin versions, review upgrades, read changelogs of security-relevant dependencies; build images from pinned base images and rebuild them regularly for OS patches.

### Behind a proxy: whose IP is it?

`r.RemoteAddr` is the address of whoever connected — behind a load balancer, that is the load balancer. The client's address is then in `X-Forwarded-For`, but **a client can send that header itself**. Trust it only when your own proxy overwrites it, and then take the address that your proxy appended (the right-most untrusted entry), not the left-most one the client may have forged. The project's rate limiter uses `RemoteAddr`, and its comment says why.

### Deep & Tricky

**DT-274. Error messages are an information leak.** *(Security practice)* `pq: duplicate key value violates unique constraint "users_email_key"` tells an attacker your database, schema, and that an email is registered. Return generic messages for 5xx and carefully chosen ones for 4xx (P-34); log the details server-side with the request ID.

**DT-275. Timing can leak secrets.** *(Security fact)* Comparing a token with `==` stops at the first different byte; measuring response times over many requests can reveal the token byte by byte. Use `subtle.ConstantTimeCompare` for secrets — and note it still leaks the *length*, which is why the project compares fixed-length SHA-256 digests.

### Mastery check

1. **Explain.** Why is bcrypt right for passwords but unnecessary for 128-bit random API keys?
2. **Explain.** Why are bearer-token APIs not vulnerable to CSRF, while cookie-based sessions are?
3. **Write.** A file download handler that serves files from `./public` by name, using `os.Root` so `../` and symlink escapes are rejected; tests with `../../etc/passwd`.

## Unit 11.3 — Authentication

**One-line summary:** Authentication proves *who* is calling. Browser applications usually use **server-side sessions** in secure cookies; APIs use **tokens** — opaque API keys, or signed **JWTs**; "Log in with Google" and enterprise single sign-on use **OAuth 2.0 / OpenID Connect**. Each has trade-offs in revocation, scalability, and complexity.

### Sessions

The server creates a random session ID (`crypto/rand.Text()`), stores session data (user ID, expiry) in a database or Redis, and sends the ID in a cookie:

```go
http.SetCookie(w, &http.Cookie{
	Name: "session", Value: id, Path: "/",
	HttpOnly: true,                 // scripts cannot read it (XSS)
	Secure:   true,                 // HTTPS only
	SameSite: http.SameSiteLaxMode, // CSRF protection
	MaxAge:   int((12 * time.Hour).Seconds()),
})
```

Logout or revocation is trivial (delete the session row). Each request costs a session lookup — cache it. Rotate the session ID on login (prevents session fixation).

### API keys

Long random tokens issued to a client (another service, a customer's script), sent as `Authorization: Bearer <key>`. Store only hashes (the project), allow several per client for rotation, record a key ID and last-used time, scope them (read-only vs read-write), and support revocation.

```go
// From the project: compare against every configured hash in constant time.
key, ok := strings.CutPrefix(r.Header.Get("Authorization"), "Bearer ")
sum := sha256.Sum256([]byte(key))
valid := false
for _, h := range want {
	if subtle.ConstantTimeCompare(sum[:], h) == 1 {
		valid = true
	}
}
```

### JWT

A JSON Web Token is three base64url parts — header, claims (`sub`, `exp`, `iat`, `aud`, custom data), signature — so any service with the key can verify it **without a database lookup**. Rules if you use them:

- Verify the signature with an algorithm **you** choose (reject `alg: none` and unexpected algorithms), and verify `exp`, `nbf`, `iss`, `aud`.
- Keep them short-lived (minutes), because a signed token cannot be revoked before it expires without extra machinery (a deny-list, which brings back the database lookup); pair with refresh tokens.
- Never put secrets in claims — they are only encoded, not encrypted.
- Use a maintained library (`github.com/golang-jwt/jwt/v5`, or a JOSE library) rather than writing verification yourself.

JWTs shine between services (stateless verification at scale) and are often overused for browser sessions, where a plain session cookie is simpler and revocable.

### OAuth 2.0 and OpenID Connect

OAuth 2.0 lets a user grant an application limited access to their account at another service without sharing their password; **OpenID Connect** (OIDC) adds a standard *identity* layer on top ("who is this user?") with an ID token (a JWT). The browser flow for web apps is the **authorization code flow with PKCE**: redirect the user to the identity provider, receive a one-time code at your callback, exchange it server-side for tokens, verify the ID token. `golang.org/x/oauth2` handles the protocol; `github.com/coreos/go-oidc` verifies ID tokens. For service-to-service calls, the **client credentials** flow gives a service its own token.

### Deep & Tricky

**DT-276. "Logged in" must be re-checked on every request.** *(Design rule)* Authentication state lives in the token or session, not in the connection or the client's claims. Every request is authenticated independently; HTTP/2 multiplexes many users' requests over one connection through proxies, so nothing about the connection identifies the user.

### Mastery check

1. **Explain.** Compare sessions and JWTs for (a) revocation (b) scaling across services (c) complexity.
2. **Write.** Add API key IDs to the project: keys of the form `<id>.<secret>`, stored as `id → hash`, so lookups are O(1) and logs can record which key made each request without logging the secret.

## Unit 11.4 — Authorisation

**One-line summary:** Authorisation decides *what* an authenticated caller may do. Model it explicitly — roles, permissions, and ownership checks — enforce it in one place per resource, deny by default, and test it as thoroughly as any feature.

### Models

- **Role-based (RBAC):** users have roles (`viewer`, `editor`, `admin`), roles have permissions (`links:read`, `links:create`, `links:delete`). Simple and common.
- **Ownership / attribute-based:** "a user may delete *their own* links; an admin may delete any". Decisions depend on attributes of the user and the resource.
- **Relationship-based (ReBAC):** permissions follow relationships ("members of the team that owns the folder"), as in Google's Zanzibar; tools like OpenFGA implement it.

### Enforcement

```go
type Principal struct {
	ID    string
	Roles []string
}

var permissions = map[string][]string{
	"viewer": {"links:read"},
	"editor": {"links:read", "links:create"},
	"admin":  {"links:read", "links:create", "links:delete"},
}

func (p Principal) Can(perm string) bool {
	for _, r := range p.Roles {
		if slices.Contains(permissions[r], perm) {
			return true
		}
	}
	return false // deny by default
}

// in the service layer, not only in HTTP middleware:
func (s *Service) Delete(ctx context.Context, p Principal, code string) error {
	l, err := s.store.Get(ctx, code)
	if err != nil {
		return err
	}
	if !p.Can("links:delete") && l.Owner != p.ID {
		return ErrForbidden // the HTTP layer maps it to 403 (or 404, to hide existence)
	}
	return s.store.Delete(ctx, code)
}
```

Rules: **deny by default**; check in the **service layer** where the resource is known (middleware cannot know who owns link `abc` — a missing ownership check is the "insecure direct object reference", one of the most common real vulnerabilities); return 404 instead of 403 when even the existence of a resource is secret; log authorisation failures; and write table-driven tests of the permission matrix (every role × every action).

### Mastery check

1. **Explain.** Why can authorisation for "delete *my* link" not be done entirely in HTTP middleware?
2. **Write.** Add an `owner` column to links, record the API key ID as owner, and implement `DELETE /api/links/{code}` allowed for the owner or an admin key; a table test of the matrix.

## Topic 11B — Observability

## Unit 11.5 — Logs, metrics, and traces

**One-line summary:** Observability is being able to answer new questions about a running system from the outside. **Logs** record events with context; **metrics** are numbers aggregated over time, cheap to keep and alert on; **traces** follow one request across services. Health and readiness checks tell the platform whether to send traffic and whether to restart.

### Logs

Structured JSON logs with `log/slog` (Unit 5.8): one line per request (the project's access log), one per significant event, errors once with context and request ID, never secrets. Logs are for *investigating*; do not alert on log patterns when a metric would do.

### Metrics: the RED and USE methods

For every service, measure **RED**: **R**ate (requests per second), **E**rrors (failed requests per second), **D**uration (latency distribution). For every resource (CPU, memory, pool, queue), measure **USE**: **U**tilisation, **S**aturation (queueing), **E**rrors.

Prometheus metric types:

- **Counter:** only goes up (`http_requests_total`); rates come from `rate(...)` in queries.
- **Gauge:** goes up and down (`go_goroutines`, queue length).
- **Histogram:** counts observations into buckets (`http_request_duration_seconds_bucket{le="0.1"}`); percentiles are computed at query time across all instances (`histogram_quantile(0.99, ...)`).

The project exposes these in the Prometheus **text format** with a small hand-written package, to show exactly what they are. Real output after a few requests:

```
# HELP http_requests_total HTTP requests by method, route, and status code.
# TYPE http_requests_total counter
http_requests_total{method="GET",route="GET /{code}",code="302"} 2
http_requests_total{method="GET",route="unmatched",code="429"} 4
http_requests_total{method="POST",route="POST /api/links",code="201"} 1
http_requests_total{method="POST",route="unmatched",code="401"} 1
http_request_duration_seconds_count{method="GET",route="GET /{code}"} 2
...
# TYPE go_goroutines gauge
go_goroutines 10
```

In a real service, use `github.com/prometheus/client_golang` (which also exports `runtime/metrics` and process metrics) or OpenTelemetry metrics.

**Label cardinality:** every distinct combination of label values is a separate time series. Label by **route pattern** (`GET /{code}`, bounded), never by raw path (`/abc123`, unbounded — millions of series will take down your metrics system). The project uses `r.Pattern` (Go 1.23), which the `ServeMux` sets after routing.

### Traces

A **trace** is a tree of **spans** (an operation with start, end, attributes) for one request across services; context propagation passes the trace ID in headers (`traceparent`, W3C Trace Context). With OpenTelemetry (`go.opentelemetry.io/otel`), you instrument HTTP servers and clients (`otelhttp`), database drivers, and your own code:

```go
ctx, span := tracer.Start(ctx, "links.Shorten")
defer span.End()
span.SetAttributes(attribute.String("code", code))
if err != nil {
	span.RecordError(err)
	span.SetStatus(codes.Error, "shorten failed")
}
```

Traces answer "where did these 2 seconds go across five services?" — the question CPU profiles cannot (DT-263). Put the trace ID in your logs so you can jump from a log line to its trace.

### Health checks

- **Liveness** (`/livez`): "is the process healthy enough to keep running?" If it fails, the platform **restarts** the container. It must check only the process itself — **never dependencies**: if liveness checked the database, a database outage would make the platform restart every instance in a loop, turning a partial outage into a total one.
- **Readiness** (`/readyz`): "should this instance receive traffic right now?" It may check critical dependencies (the project pings the database) and is deliberately failed during startup and shutdown.

Serve them, metrics, and pprof on a separate **admin listener** that is not exposed publicly (the project uses `ADMIN_ADDR`, defaulting to `127.0.0.1:9090`).

### Deep & Tricky

**DT-277. Requests rejected by outer middleware have no route.** *(Seen in this chapter's project)* `r.Pattern` is set by the `ServeMux` when it routes. The project's 401s (auth) and 429s (rate limit) are produced by middleware *before* the mux runs, so they appear as `route="unmatched"`. That is acceptable (and bounded), but dashboards must know it: "429s on `GET /{code}`" will show zero while clients are being throttled. Either place such middleware after routing (per route) or label rejections by a coarse path group.

**DT-278. Averages hide the users who suffer.** *(Measurement fact)* A mean latency of 50 ms is compatible with 1% of requests taking 5 seconds. Alert on high percentiles (p99, p99.9) from histograms, and on error *rates* — never on averages alone.

### Pattern

**P-37. Instrument RED for every endpoint and USE for every resource, from day one.** *(Operations practice)* Rate, errors, and duration per route (bounded labels), plus pool, queue, and runtime saturation, give you the first answer to almost every incident question. *Trade-off:* a little code and some metrics storage; without them, incidents are investigated by guessing.

### Mastery check

1. **Explain.** Why must a liveness probe not check the database?
2. **Explain.** Why is labelling metrics by raw URL path dangerous?
3. **Write.** Add a gauge for the cache's pending-hits count and a counter for flush failures to the project's metrics, and a test that reads them from the text output.

## Topic 11C — Delivery and operations

## Unit 11.6 — Containers

**One-line summary:** Build Go services as static binaries (`CGO_ENABLED=0`) in a multi-stage Docker build and copy them into a minimal, non-root runtime image (distroless or scratch). Stamp version information with `-ldflags -X`. The result is a small image with almost nothing for an attacker to use.

### The project's Dockerfile

*(Standard configuration; Docker image builds could not be run in the environment used to write this book.)*

```dockerfile
# syntax=docker/dockerfile:1

# ---- build stage: full Go toolchain ----
FROM golang:1.27 AS build
WORKDIR /src
# Download dependencies first: this layer is cached until go.mod/go.sum change.
COPY go.mod go.sum ./
RUN go mod download
COPY . .
ARG VERSION=dev
ARG COMMIT=unknown
# CGO_ENABLED=0: a static binary that needs no C libraries (DT-4).
# -trimpath: no local paths in the binary. -s -w: no symbol table/DWARF (DT-12).
RUN CGO_ENABLED=0 go build -trimpath \
    -ldflags="-s -w -X main.version=${VERSION} -X main.commit=${COMMIT}" \
    -o /out/shortener ./cmd/shortener

# ---- runtime stage: no shell, no package manager, non-root ----
FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=build /out/shortener /shortener
USER nonroot:nonroot
EXPOSE 8080 9090
ENTRYPOINT ["/shortener"]
```

Why each choice:

- **Multi-stage:** the final image contains only the binary — not the compiler, source code, or module cache. Hundreds of MB → a few MB plus the base.
- **`CGO_ENABLED=0`:** a fully static binary (DT-4) that runs on `distroless/static` or even `scratch`. With cgo you need a base with the right C libraries.
- **`distroless/static:nonroot`:** CA certificates (for outgoing TLS), time-zone data, `/etc/passwd` with a non-root user — and no shell or package manager for an attacker to use. (`scratch` is emptier still; then embed `time/tzdata` and copy CA certificates yourself.)
- **Layer order:** `go.mod`/`go.sum` and `go mod download` before `COPY . .`, so code changes do not re-download dependencies.
- **`.dockerignore`:** keeps `.git`, build outputs, and local secrets out of the build context.

### Version information

`-ldflags "-X main.version=1.4.2 -X main.commit=abc1234"` sets string variables at link time. The project prints them at startup and serves them at `/version`. Real output:

```
$ curl -s localhost:19092/version
1.4.2 abc1234
```

Since Go 1.18, `runtime/debug.ReadBuildInfo()` also reports the module version and VCS revision automatically (`go version -m ./shortener` shows them for any binary) — useful even without `-X`.

### Deep & Tricky

**DT-279. A static Go binary in `scratch` has no CA certificates or time zones.** *(Container fact)* Outgoing HTTPS fails with `x509: certificate signed by unknown authority`, and `time.LoadLocation("Asia/Kolkata")` fails. Use `distroless/static` (which has both), or copy `/etc/ssl/certs/ca-certificates.crt` into the image and `import _ "time/tzdata"` to embed the time-zone database (about 450 KB).

### Mastery check

1. **Explain.** What does each of `-trimpath`, `-s`, `-w`, and `CGO_ENABLED=0` change about the binary?
2. **Write.** (If you have Docker.) Build the image, compare its size with a single-stage `golang:1.27` image, and run it with the database URL passed as a mounted secret file.

## Unit 11.7 — CI/CD pipelines

**One-line summary:** Every change runs the same automated gate: formatting, `go mod tidy` check, vet, static analysis, tests with the race detector against real dependencies, and vulnerability scanning; then an immutable, versioned artifact is built once and promoted through environments.

### The project's pipeline (GitHub Actions)

*(Standard configuration, not executed here.)*

```yaml
name: ci
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_HOST_AUTH_METHOD: trust
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready --health-interval 5s --health-timeout 5s --health-retries 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
      - name: Formatting
        run: test -z "$(gofmt -l .)"
      - name: go mod tidy is clean
        run: go mod tidy -diff
      - name: Vet
        run: go vet ./...
      - name: Staticcheck
        run: go run honnef.co/go/tools/cmd/staticcheck@latest ./...
      - name: Tests with the race detector, including PostgreSQL integration tests
        env:
          TEST_DATABASE_URL: postgres://postgres@localhost:5432/postgres
        run: go test -race -count=1 -shuffle=on ./...
      - name: Known vulnerabilities in reachable code
        run: go run golang.org/x/vuln/cmd/govulncheck@latest ./...

  image:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: |
          docker build \
            --build-arg VERSION=${GITHUB_SHA::7} --build-arg COMMIT=${GITHUB_SHA::7} \
            -t ghcr.io/${{ github.repository }}:${GITHUB_SHA::7} .
      # then: scan the image, push it, and deploy (environment-specific)
```

Principles:

- **Fast feedback:** formatting and vet fail in seconds; slow integration tests run after.
- **Real dependencies:** the PostgreSQL service container makes the integration tests of Unit 9.8 run on every change, not just on the developer's machine.
- **`-race -shuffle=on -count=1`:** races, order dependence, and cached results are all excluded.
- **Build once, deploy many:** the image is built once from the tested commit, tagged immutably (commit SHA), and the *same* image goes to staging and production. Never rebuild for production.
- **Deploy progressively:** canary (a small share of traffic first) or blue-green, with automatic rollback on error-rate or latency regressions (the RED metrics of Unit 11.5).
- Pin third-party actions to commit SHAs in high-security settings (they run with your repository's permissions).

### Mastery check

1. **Explain.** Why should the production image be the same artifact that was tested in staging rather than a fresh build?
2. **Write.** Add a CI step that fails if test coverage of `internal/` drops below 80%.

## Unit 11.8 — Kubernetes basics

**One-line summary:** Kubernetes runs containers as **Pods** managed by a **Deployment** (desired replica count, rolling updates), exposed through a **Service**, configured with environment variables, ConfigMaps, and Secrets, and kept healthy with **probes**. A Go service must cooperate: correct probes, resource limits that match the runtime (GOMAXPROCS, GOMEMLIMIT), and graceful shutdown on SIGTERM.

### The project's manifest

*(Standard configuration, not executed here.)*

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shortener
spec:
  replicas: 3
  selector:
    matchLabels: {app: shortener}
  strategy:
    rollingUpdate: {maxUnavailable: 0, maxSurge: 1}   # never drop below 3 ready pods
  template:
    metadata:
      labels: {app: shortener}
    spec:
      terminationGracePeriodSeconds: 30               # > drain wait (5s) + shutdown grace (15s)
      securityContext: {runAsNonRoot: true}
      containers:
        - name: shortener
          image: ghcr.io/example/shortener:1.4.2      # an immutable tag, never :latest
          ports:
            - {name: http, containerPort: 8080}
            - {name: admin, containerPort: 9090}
          env:
            - {name: ADDR, value: ":8080"}
            - {name: ADMIN_ADDR, value: ":9090"}      # reachable inside the cluster only
            - {name: REDIS_ADDR, value: "redis:6379"}
            - {name: DATABASE_URL_FILE, value: /secrets/db/url}
            - {name: API_KEY_HASHES_FILE, value: /secrets/api/hashes}
            - {name: GOMEMLIMIT, value: "230MiB"}      # ~90% of the memory limit (Unit 10.6)
          volumeMounts:
            - {name: db, mountPath: /secrets/db, readOnly: true}
            - {name: api, mountPath: /secrets/api, readOnly: true}
          resources:
            requests: {cpu: 250m, memory: 128Mi}
            limits: {cpu: "1", memory: 256Mi}          # Go 1.25+ sets GOMAXPROCS from the CPU limit
          readinessProbe:
            httpGet: {path: /readyz, port: admin}
            periodSeconds: 2
            failureThreshold: 1
          livenessProbe:
            httpGet: {path: /livez, port: admin}       # liveness must NOT check the database
            periodSeconds: 10
            failureThreshold: 3
          securityContext:
            readOnlyRootFilesystem: true
            allowPrivilegeEscalation: false
            capabilities: {drop: [ALL]}
      volumes:
        - name: db
          secret: {secretName: shortener-db}
        - name: api
          secret: {secretName: shortener-api-keys}
---
apiVersion: v1
kind: Service
metadata:
  name: shortener
spec:
  selector: {app: shortener}
  ports:
    - {name: http, port: 80, targetPort: http}
```

### Graceful shutdown, end to end

When a pod is deleted (rolling update, scale-down, node drain), Kubernetes sends SIGTERM and, at the same time, starts removing the pod from Service endpoints — but endpoint removal propagates to load balancers and other nodes over a few seconds. If the service stops accepting connections immediately, those seconds produce errors (DT-248). The project's sequence:

1. SIGTERM arrives; `ctx` is cancelled.
2. **Readiness fails immediately** (`ready.Store(false)`), so the endpoint is removed even faster.
3. **Wait** (`drainWait`, 5 s) while traffic stops arriving — requests that still arrive are served normally.
4. `public.Shutdown(ctx)` stops accepting and waits for in-flight requests, up to the grace (15 s).
5. Deferred cleanup: stop the hit flusher (which writes the remaining hits), close Redis and the database pool.
6. Exit before `terminationGracePeriodSeconds` (30 s), after which Kubernetes would SIGKILL.

The real log of that sequence from the project:

```
"msg":"shutdown: readiness failing, draining"
readyz during drain: 503
"msg":"http servers stopped; flushing hits"
```

### Resources and the Go runtime

- **CPU limit → GOMAXPROCS:** Go 1.25+ reads the container's CPU limit (Unit 10.7). With older Go, set `GOMAXPROCS` explicitly or use `automaxprocs`, or the service is throttled.
- **Memory limit → GOMEMLIMIT:** set it to ~90% of the container limit so the GC works harder before the kernel's OOM killer acts (Unit 10.6).
- **Requests** decide scheduling and guaranteed capacity; **limits** cap usage. Measure real usage under load (Unit 14.6) before choosing them.

### Other essentials

- **ConfigMaps** for non-secret configuration, **Secrets** mounted as files (Unit 11.1).
- **Horizontal Pod Autoscaler** scales replicas on CPU or custom metrics (requests per second).
- **PodDisruptionBudget** keeps a minimum number of replicas during voluntary disruptions.
- `readOnlyRootFilesystem`, non-root, dropped capabilities: defence in depth that costs a Go binary nothing.

### Deep & Tricky

**DT-280. A failing liveness probe during load turns slowness into an outage.** *(Operations)* If `/livez` shares the request path's resources (same goroutines, same locks, a database check), it times out under heavy load, Kubernetes restarts the pod, the remaining pods get more load, their probes fail too — a cascading restart loop. Keep liveness trivially cheap and on the admin listener.

### Mastery check

1. **Explain.** Walk through what happens to an in-flight request and to a new request during each step of the shutdown sequence.
2. **Explain.** Why `maxUnavailable: 0`? What does it require from readiness?
3. **Write.** A PodDisruptionBudget and a HorizontalPodAutoscaler for the shortener, with your reasoning for each number.

## Chapter 11 Project — Productionise the URL shortener

**Goal:** take the Chapter 9 shortener to production quality: validated configuration with file-based secrets, API-key authentication for writes, per-client rate limiting, security headers, Prometheus metrics, liveness and readiness on an admin listener, pprof, version stamping, graceful drain, a minimal container image, a CI pipeline, and Kubernetes manifests.

### Requirements

1. `internal/config`: environment-based, every error reported at once (`errors.Join`), `*_FILE` secrets, a secret-free `String()`, unit tests.
2. `internal/guard`: `RequireAPIKey` (Bearer keys, SHA-256 hashes, constant-time comparison, writes only), a token-bucket `Limiter` per client IP with `Retry-After`, a `Sweep` to bound memory, `SecurityHeaders`; tests with an injected clock.
3. `internal/metrics`: request counter and latency histogram labelled by method, route pattern, and status; runtime gauges; Prometheus text format; a test.
4. `httpapi.New` accepts extra middleware applied around the router (so `r.Pattern` is visible).
5. `main`: two listeners (public, admin), `/livez`, `/readyz` (fails during startup and shutdown, pings the database), `/metrics`, `/version`, `/debug/pprof/*` on admin only; readiness-drain-shutdown sequence; version via `-ldflags -X`.
6. `Dockerfile` (multi-stage, static, distroless, non-root), `.github/workflows/ci.yml`, `deploy/kubernetes.yaml`.

### What changed in `httpapi`

```go
// New returns the complete handler, with middleware applied. Extra
// middleware in wrap is applied directly around the router, inside request
// IDs, logging, and panic recovery, so it can see r.Pattern (Go 1.23) and
// the request ID.
func New(svc *links.Service, log *slog.Logger, wrap ...func(http.Handler) http.Handler) http.Handler {
	a := &API{svc: svc, log: log}
	mux := http.NewServeMux()
	...
	var h http.Handler = mux
	for i := len(wrap) - 1; i >= 0; i-- { // wrap[0] ends up outermost
		h = wrap[i](h)
	}
	// Middleware is applied outside-in: requestID runs first.
	return requestID(logRequests(log, recoverPanics(log, h)))
}
```

Existing callers (`New(svc, log)`) keep compiling: a variadic parameter is a compatible API change.

### `internal/config/config.go`

```go
// Package config loads the service configuration from the environment,
// validates all of it at startup, and reads secrets from files.
package config

import (
	"errors"
	"fmt"
	"os"
	"strconv"
	"strings"
	"time"
)

type Config struct {
	Addr          string        // public listener
	AdminAddr     string        // metrics, pprof, health: never exposed publicly
	DatabaseURL   string        // secret
	RedisAddr     string        // optional
	APIKeyHashes  []string      // hex SHA-256 of valid API keys (secret)
	RatePerSecond float64       // per client IP
	RateBurst     int
	CacheTTL      time.Duration
	ShutdownGrace time.Duration
}

// Load reads configuration using getenv (os.Getenv in production, a map in
// tests). Every problem is reported at once, not one per restart.
func Load(getenv func(string) string) (Config, error) {
	c := Config{
		Addr:          or(getenv("ADDR"), ":8080"),
		AdminAddr:     or(getenv("ADMIN_ADDR"), "127.0.0.1:9090"),
		RedisAddr:     getenv("REDIS_ADDR"),
		RatePerSecond: 20,
		RateBurst:     40,
		CacheTTL:      10 * time.Minute,
		ShutdownGrace: 15 * time.Second,
	}
	var errs []error
	var err error
	if c.DatabaseURL, err = secret(getenv, "DATABASE_URL"); err != nil {
		errs = append(errs, err)
	} else if c.DatabaseURL == "" {
		errs = append(errs, errors.New("DATABASE_URL (or DATABASE_URL_FILE) is required"))
	}
	keys, err := secret(getenv, "API_KEY_HASHES")
	if err != nil {
		errs = append(errs, err)
	}
	for _, h := range strings.Split(keys, ",") {
		if h = strings.TrimSpace(h); h == "" {
			continue
		}
		if len(h) != 64 || strings.Trim(h, "0123456789abcdef") != "" {
			errs = append(errs, fmt.Errorf("API_KEY_HASHES: %q is not a hex SHA-256", h))
			continue
		}
		c.APIKeyHashes = append(c.APIKeyHashes, h)
	}
	if v := getenv("RATE_PER_SECOND"); v != "" {
		if c.RatePerSecond, err = strconv.ParseFloat(v, 64); err != nil || c.RatePerSecond <= 0 {
			errs = append(errs, fmt.Errorf("RATE_PER_SECOND: must be a positive number, got %q", v))
		}
	}
	if v := getenv("RATE_BURST"); v != "" {
		if c.RateBurst, err = strconv.Atoi(v); err != nil || c.RateBurst < 1 {
			errs = append(errs, fmt.Errorf("RATE_BURST: must be a positive integer, got %q", v))
		}
	}
	if v := getenv("CACHE_TTL"); v != "" {
		if c.CacheTTL, err = time.ParseDuration(v); err != nil || c.CacheTTL <= 0 {
			errs = append(errs, fmt.Errorf("CACHE_TTL: must be a positive duration, got %q", v))
		}
	}
	return c, errors.Join(errs...)
}

// secret returns NAME, or the trimmed contents of the file named by
// NAME_FILE. Files work with Docker and Kubernetes secrets and keep
// secrets out of `ps` output and crash dumps of the environment.
func secret(getenv func(string) string, name string) (string, error) {
	v, file := getenv(name), getenv(name+"_FILE")
	if v != "" && file != "" {
		return "", fmt.Errorf("set only one of %s and %s_FILE", name, name)
	}
	if file == "" {
		return v, nil
	}
	b, err := os.ReadFile(file)
	if err != nil {
		return "", fmt.Errorf("%s_FILE: %w", name, err)
	}
	return strings.TrimSpace(string(b)), nil
}

func or(v, def string) string {
	if v == "" {
		return def
	}
	return v
}

// String describes the configuration without secrets, for startup logs.
func (c Config) String() string {
	return fmt.Sprintf("addr=%s admin=%s redis=%t api_keys=%d rate=%.1f/s burst=%d cache_ttl=%s",
		c.Addr, c.AdminAddr, c.RedisAddr != "", len(c.APIKeyHashes), c.RatePerSecond, c.RateBurst, c.CacheTTL)
}
```

### `internal/guard/guard.go`

```go
// Package guard holds HTTP middleware that protects the service:
// API-key authentication, per-client rate limiting, and security headers.
package guard

import (
	"crypto/sha256"
	"crypto/subtle"
	"encoding/hex"
	"math"
	"net"
	"net/http"
	"strconv"
	"strings"
	"sync"
	"time"
)

// RequireAPIKey rejects state-changing API requests (POST, PUT, PATCH,
// DELETE under /api/) unless they carry "Authorization: Bearer <key>"
// whose SHA-256 is in hashes. Only hashes are stored, so a leaked config
// does not leak usable keys; comparisons take constant time.
func RequireAPIKey(hashes []string) func(http.Handler) http.Handler {
	want := make([][]byte, 0, len(hashes))
	for _, h := range hashes {
		b, err := hex.DecodeString(h)
		if err == nil && len(b) == sha256.Size {
			want = append(want, b)
		}
	}
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			if r.Method == http.MethodGet || r.Method == http.MethodHead || !strings.HasPrefix(r.URL.Path, "/api/") {
				next.ServeHTTP(w, r)
				return
			}
			key, ok := strings.CutPrefix(r.Header.Get("Authorization"), "Bearer ")
			sum := sha256.Sum256([]byte(key))
			valid := false
			for _, h := range want {
				// ConstantTimeCompare: no timing difference reveals how many bytes matched.
				if subtle.ConstantTimeCompare(sum[:], h) == 1 {
					valid = true
				}
			}
			if !ok || !valid {
				w.Header().Set("WWW-Authenticate", `Bearer realm="shortener"`)
				http.Error(w, `{"error":"missing or invalid API key"}`, http.StatusUnauthorized)
				return
			}
			next.ServeHTTP(w, r)
		})
	}
}

// HashKey returns the hex SHA-256 of an API key, for the config.
func HashKey(key string) string {
	s := sha256.Sum256([]byte(key))
	return hex.EncodeToString(s[:])
}

// Limiter is a token bucket per client: each client may make burst
// requests at once and rate requests per second on average.
type Limiter struct {
	rate    float64
	burst   float64
	now     func() time.Time
	mu      sync.Mutex
	buckets map[string]*bucket
}

type bucket struct {
	tokens float64
	last   time.Time
}

func NewLimiter(rate float64, burst int) *Limiter {
	return &Limiter{rate: rate, burst: float64(burst), now: time.Now, buckets: map[string]*bucket{}}
}

// Allow reports whether the client may make a request now, and if not,
// how long until it may.
func (l *Limiter) Allow(client string) (bool, time.Duration) {
	l.mu.Lock()
	defer l.mu.Unlock()
	now := l.now()
	b, ok := l.buckets[client]
	if !ok {
		b = &bucket{tokens: l.burst, last: now}
		l.buckets[client] = b
	}
	b.tokens = math.Min(l.burst, b.tokens+now.Sub(b.last).Seconds()*l.rate) // refill
	b.last = now
	if b.tokens >= 1 {
		b.tokens--
		return true, 0
	}
	wait := time.Duration((1 - b.tokens) / l.rate * float64(time.Second))
	return false, wait
}

// Sweep removes buckets that are full again (idle clients), bounding memory.
func (l *Limiter) Sweep() {
	l.mu.Lock()
	defer l.mu.Unlock()
	now := l.now()
	for k, b := range l.buckets {
		if b.tokens+now.Sub(b.last).Seconds()*l.rate >= l.burst {
			delete(l.buckets, k)
		}
	}
}

// Middleware rate-limits by client IP. It uses the connection's address:
// trusting X-Forwarded-For is only safe behind a proxy you control that
// overwrites it (see Unit 11.2).
func (l *Limiter) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ip, _, err := net.SplitHostPort(r.RemoteAddr)
		if err != nil {
			ip = r.RemoteAddr
		}
		if ok, wait := l.Allow(ip); !ok {
			w.Header().Set("Retry-After", strconv.Itoa(int(math.Ceil(wait.Seconds()))))
			http.Error(w, `{"error":"too many requests"}`, http.StatusTooManyRequests)
			return
		}
		next.ServeHTTP(w, r)
	})
}

// SecurityHeaders sets conservative headers for an API.
func SecurityHeaders(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		h := w.Header()
		h.Set("X-Content-Type-Options", "nosniff")
		h.Set("X-Frame-Options", "DENY")
		h.Set("Referrer-Policy", "no-referrer")
		h.Set("Content-Security-Policy", "default-src 'none'; frame-ancestors 'none'")
		h.Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
		next.ServeHTTP(w, r)
	})
}
```

The token bucket in one sentence: each client has a bucket of up to `burst` tokens that refills at `rate` per second; a request spends one token or is rejected with how long until the next token. Note the loop that compares against **every** hash without stopping early, so the number of configured keys — not which one matched — determines the time taken.

### `internal/metrics/metrics.go`

```go
// Package metrics records HTTP request metrics and writes them in the
// Prometheus text exposition format. A real service would usually use
// github.com/prometheus/client_golang; this small version shows exactly
// what a counter, a histogram, and the format are.
package metrics

import (
	"fmt"
	"io"
	"net/http"
	"runtime/metrics"
	"slices"
	"strconv"
	"sync"
	"time"
)

// Buckets are upper bounds in seconds for request durations.
var Buckets = []float64{0.001, 0.0025, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5}

type key struct{ method, route, code string }

type hist struct {
	counts []uint64 // one per bucket, cumulative at write time
	sum    float64
	count  uint64
}

type HTTP struct {
	mu        sync.Mutex
	requests  map[key]uint64
	durations map[[2]string]*hist // method, route
}

func NewHTTP() *HTTP {
	return &HTTP{requests: map[key]uint64{}, durations: map[[2]string]*hist{}}
}

type recorder struct {
	http.ResponseWriter
	code int
}

func (r *recorder) WriteHeader(c int)            { r.code = c; r.ResponseWriter.WriteHeader(c) }
func (r *recorder) Unwrap() http.ResponseWriter { return r.ResponseWriter }

// Middleware must wrap the router directly so r.Pattern is set; it uses the
// route pattern (bounded set) rather than the raw path (unbounded) as a label.
func (m *HTTP) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		rec := &recorder{ResponseWriter: w, code: http.StatusOK}
		next.ServeHTTP(rec, r)
		route := r.Pattern // e.g. "GET /{code}"; set by ServeMux (Go 1.23+)
		if route == "" {
			route = "unmatched"
		}
		m.observe(r.Method, route, rec.code, time.Since(start))
	})
}

func (m *HTTP) observe(method, route string, code int, d time.Duration) {
	m.mu.Lock()
	defer m.mu.Unlock()
	m.requests[key{method, route, strconv.Itoa(code)}]++
	h := m.durations[[2]string{method, route}]
	if h == nil {
		h = &hist{counts: make([]uint64, len(Buckets))}
		m.durations[[2]string{method, route}] = h
	}
	s := d.Seconds()
	for i, b := range Buckets {
		if s <= b {
			h.counts[i]++
		}
	}
	h.sum += s
	h.count++
}

// Write writes all metrics in Prometheus text format, sorted for stable output.
func (m *HTTP) Write(w io.Writer) {
	m.mu.Lock()
	defer m.mu.Unlock()
	fmt.Fprintln(w, "# HELP http_requests_total HTTP requests by method, route, and status code.")
	fmt.Fprintln(w, "# TYPE http_requests_total counter")
	keys := make([]key, 0, len(m.requests))
	for k := range m.requests {
		keys = append(keys, k)
	}
	slices.SortFunc(keys, func(a, b key) int {
		return cmpStr(a.method+a.route+a.code, b.method+b.route+b.code)
	})
	for _, k := range keys {
		fmt.Fprintf(w, "http_requests_total{method=%q,route=%q,code=%q} %d\n", k.method, k.route, k.code, m.requests[k])
	}
	fmt.Fprintln(w, "# HELP http_request_duration_seconds HTTP request latency.")
	fmt.Fprintln(w, "# TYPE http_request_duration_seconds histogram")
	hk := make([][2]string, 0, len(m.durations))
	for k := range m.durations {
		hk = append(hk, k)
	}
	slices.SortFunc(hk, func(a, b [2]string) int { return cmpStr(a[0]+a[1], b[0]+b[1]) })
	for _, k := range hk {
		h := m.durations[k]
		for i, b := range Buckets {
			fmt.Fprintf(w, "http_request_duration_seconds_bucket{method=%q,route=%q,le=%q} %d\n",
				k[0], k[1], strconv.FormatFloat(b, 'g', -1, 64), h.counts[i])
		}
		fmt.Fprintf(w, "http_request_duration_seconds_bucket{method=%q,route=%q,le=\"+Inf\"} %d\n", k[0], k[1], h.count)
		fmt.Fprintf(w, "http_request_duration_seconds_sum{method=%q,route=%q} %g\n", k[0], k[1], h.sum)
		fmt.Fprintf(w, "http_request_duration_seconds_count{method=%q,route=%q} %d\n", k[0], k[1], h.count)
	}
	writeRuntime(w)
}

func cmpStr(a, b string) int {
	switch {
	case a < b:
		return -1
	case a > b:
		return 1
	}
	return 0
}

func writeRuntime(w io.Writer) {
	s := []metrics.Sample{
		{Name: "/sched/goroutines:goroutines"},
		{Name: "/gc/heap/live:bytes"},
		{Name: "/gc/cycles/total:gc-cycles"},
	}
	metrics.Read(s)
	fmt.Fprintf(w, "# TYPE go_goroutines gauge\ngo_goroutines %d\n", s[0].Value.Uint64())
	fmt.Fprintf(w, "# TYPE go_heap_live_bytes gauge\ngo_heap_live_bytes %d\n", s[1].Value.Uint64())
	fmt.Fprintf(w, "# TYPE go_gc_cycles_total counter\ngo_gc_cycles_total %d\n", s[2].Value.Uint64())
}

// Handler serves the metrics for a Prometheus scraper.
func (m *HTTP) Handler() http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, _ *http.Request) {
		w.Header().Set("Content-Type", "text/plain; version=0.0.4")
		m.Write(w)
	})
}
```

A note from development: this method was first called `WriteTo(w io.Writer)`, and `go vet` reported `method WriteTo(w io.Writer) should have signature WriteTo(io.Writer) (int64, error)`. Methods named like the standard interfaces (`WriteTo`, `ReadFrom`, `String`, `Error`, `Format`) must match their signatures, or code that checks for those interfaces (Unit 3.6, `io.Copy`'s fast paths) will misbehave. Renaming to `Write` fixed it.

### `cmd/shortener/main.go`

```go
// Command shortener runs the URL shortener service.
//
// Configuration: see package config. Two listeners: the public API on
// ADDR, and an admin listener on ADMIN_ADDR (metrics, health, pprof) that
// must never be exposed to the internet.
package main

import (
	"context"
	"errors"
	"log/slog"
	"net/http"
	"net/http/pprof"
	"os"
	"os/signal"
	"sync/atomic"
	"syscall"
	"time"

	"github.com/redis/go-redis/v9"

	"example.com/shortener/internal/cache"
	"example.com/shortener/internal/config"
	"example.com/shortener/internal/guard"
	"example.com/shortener/internal/httpapi"
	"example.com/shortener/internal/links"
	"example.com/shortener/internal/metrics"
	"example.com/shortener/internal/pgstore"
)

// Set at build time: go build -ldflags "-X main.version=1.4.2 -X main.commit=$(git rev-parse --short HEAD)"
var (
	version = "dev"
	commit  = "unknown"
)

func main() {
	log := slog.New(slog.NewJSONHandler(os.Stderr, nil)).With("version", version)
	if err := run(log); err != nil {
		log.Error("shortener stopped", "err", err)
		os.Exit(1)
	}
}

func run(log *slog.Logger) error {
	cfg, err := config.Load(os.Getenv)
	if err != nil {
		return err // every configuration problem, at once
	}
	log.Info("starting", "commit", commit, "config", cfg.String())

	ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
	defer stop()

	startCtx, cancel := context.WithTimeout(ctx, 30*time.Second)
	db, err := pgstore.Open(startCtx, cfg.DatabaseURL)
	cancel()
	if err != nil {
		return err
	}
	defer db.Close()

	var kv cache.KV = cache.NewMemoryKV()
	if cfg.RedisAddr != "" {
		rdb := redis.NewClient(&redis.Options{Addr: cfg.RedisAddr, DialTimeout: 2 * time.Second,
			ReadTimeout: 200 * time.Millisecond, WriteTimeout: 200 * time.Millisecond})
		defer rdb.Close()
		kv = cache.RedisKV{C: rdb}
	}
	cached := cache.New(pgstore.New(db), kv, cfg.CacheTTL, log)
	bg, stopBG := context.WithCancel(context.Background())
	flushed := make(chan struct{})
	go func() {
		defer close(flushed)
		cached.Run(bg, time.Second)
	}()
	defer func() { stopBG(); <-flushed }()

	limiter := guard.NewLimiter(cfg.RatePerSecond, cfg.RateBurst)
	go func() {
		t := time.NewTicker(time.Minute)
		defer t.Stop()
		for {
			select {
			case <-t.C:
				limiter.Sweep()
			case <-bg.Done():
				return
			}
		}
	}()

	m := metrics.NewHTTP()
	api := httpapi.New(links.NewService(cached), log,
		m.Middleware, // outermost of the extras: counts 401s and 429s too
		guard.SecurityHeaders,
		limiter.Middleware,
		guard.RequireAPIKey(cfg.APIKeyHashes),
	)
	public := &http.Server{Addr: cfg.Addr, Handler: api,
		ReadHeaderTimeout: 5 * time.Second, ReadTimeout: 10 * time.Second,
		WriteTimeout: 10 * time.Second, IdleTimeout: 60 * time.Second}

	// Readiness: false until started, and false again as soon as shutdown
	// begins, so the load balancer stops sending traffic before we stop.
	var ready atomic.Bool
	admin := http.NewServeMux()
	admin.Handle("GET /metrics", m.Handler())
	admin.HandleFunc("GET /livez", func(w http.ResponseWriter, _ *http.Request) { w.Write([]byte("ok\n")) })
	admin.HandleFunc("GET /readyz", func(w http.ResponseWriter, r *http.Request) {
		ctx, cancel := context.WithTimeout(r.Context(), time.Second)
		defer cancel()
		if !ready.Load() {
			http.Error(w, "shutting down or starting", http.StatusServiceUnavailable)
			return
		}
		if err := db.Ping(ctx); err != nil {
			http.Error(w, "database unavailable", http.StatusServiceUnavailable)
			return
		}
		w.Write([]byte("ok\n"))
	})
	admin.HandleFunc("GET /version", func(w http.ResponseWriter, _ *http.Request) {
		w.Write([]byte(version + " " + commit + "\n"))
	})
	admin.HandleFunc("/debug/pprof/", pprof.Index)
	admin.HandleFunc("/debug/pprof/profile", pprof.Profile)
	admin.HandleFunc("/debug/pprof/trace", pprof.Trace)
	adminSrv := &http.Server{Addr: cfg.AdminAddr, Handler: admin, ReadHeaderTimeout: 5 * time.Second}

	errCh := make(chan error, 2)
	go func() { errCh <- public.ListenAndServe() }()
	go func() { errCh <- adminSrv.ListenAndServe() }()
	ready.Store(true)
	log.Info("listening", "public", cfg.Addr, "admin", cfg.AdminAddr)

	select {
	case err := <-errCh:
		return err
	case <-ctx.Done():
	}

	// Graceful shutdown (Unit 11.8): fail readiness, wait for the load
	// balancer to notice, then stop accepting and drain in-flight requests.
	ready.Store(false)
	log.Info("shutdown: readiness failing, draining", "wait", drainWait)
	time.Sleep(drainWait)
	shutCtx, cancel := context.WithTimeout(context.Background(), cfg.ShutdownGrace)
	defer cancel()
	if err := public.Shutdown(shutCtx); err != nil {
		return err
	}
	adminSrv.Shutdown(shutCtx)
	for range 2 {
		if err := <-errCh; !errors.Is(err, http.ErrServerClosed) {
			return err
		}
	}
	log.Info("http servers stopped; flushing hits")
	return nil
}

// drainWait should exceed the load balancer's readiness probe period.
var drainWait = 5 * time.Second
```

The pprof handlers are registered explicitly on the admin mux instead of importing `net/http/pprof` for its side effect on `http.DefaultServeMux` — so nothing can accidentally expose them on the public listener.

### A real end-to-end run

With `API_KEY_HASHES` set to the SHA-256 of `demo-key-123`, `RATE_PER_SECOND=2`, `RATE_BURST=4`:

```
$ curl -s localhost:19092/version
1.4.2 abc1234
$ curl -s localhost:19092/readyz
ok
POST without a key:      401
POST with the key:       201
5 redirects in a burst:  302 302 429 429 429
$ curl -sD - -o /dev/null localhost:18082/godev
HTTP/1.1 429 Too Many Requests
Retry-After: 1
X-Content-Type-Options: nosniff
$ kill -TERM <pid>
readyz during drain: 503
"msg":"shutdown: readiness failing, draining"
"msg":"http servers stopped; flushing hits"
```

Why only two redirects succeeded: the burst of 4 tokens is per client IP, and the two POSTs from the same IP already spent two. The metrics output from this run is shown in Unit 11.5, including the `route="unmatched"` labels of DT-277.

### Review checklist

- Start with a bad configuration: are *all* errors reported in one go? Does any log line contain a secret?
- Is pprof reachable on the public port? (It must not be.)
- Does `readyz` fail before the server stops accepting connections during shutdown?
- Are metrics labels bounded? Try 10,000 random paths and count the series.
- Does the image run as non-root with a read-only root filesystem?
- **Known limitations to discuss:** the rate limiter is per instance (with 3 replicas a client gets 3× the rate — a Redis-based limiter shares state, Unit 12.8); API keys have no IDs or scopes yet (Unit 11.3's exercise); the configuration test checks that `String()` leaks no secret, but not that every field it prints is the right one — extend it.

## Answers to the Chapter 11 prediction questions

**11.8 Q2.** With `maxUnavailable: 0`, Kubernetes starts a new pod and waits until it is **ready** before stopping an old one, so capacity never drops below the desired replica count during a rollout. It depends entirely on an honest readiness probe: if `/readyz` reported ready before the database pool and migrations were done, traffic would reach a pod that cannot serve it.

# Chapter 12 — Distributed Systems & System Design

*Goal: design systems that keep working when parts fail.*

Once a system has more than one process — two instances behind a load balancer, a service and its database, a producer and a consumer — it is a distributed system, and it inherits a set of problems that no amount of careful single-machine code can remove: messages are lost or duplicated, clocks disagree, any call can time out with no way to know whether it happened, and parts fail independently. This chapter gives you the concepts and patterns professionals use to build reliable systems from unreliable parts, and a method for system design.

The chapter's project is real and was run against PostgreSQL under injected failures: an event-driven order pipeline in which **162 calls to a flaky payment gateway** — with transient errors, retries, and simulated crashes — produced **exactly 90 charges, 90 shipments, and 10 compensating cancellations**.

**What you will be able to do after this chapter**

- Explain partial failure, latency, and partitions, and why "exactly once" delivery is impossible while exactly-once *effects* are achievable.
- Reason about time and ordering with logical clocks.
- Explain replication, partitioning, consistent hashing, CAP/PACELC, consistency models, and Raft at a working level.
- Build messaging with the outbox pattern, idempotent consumers, retries with backoff and jitter, dead-letter queues, and sagas.
- Apply reliability patterns (timeouts, circuit breakers, bulkheads, rate limiting, load balancing) and architecture patterns (CQRS, event sourcing).
- Run a structured system-design process with estimates and failure analysis.

## Topic 12A — Fundamentals

## Unit 12.1 — Why distributed systems are hard

**One-line summary:** In a distributed system, a request can succeed, fail, or **neither as far as you can tell** (a timeout). Components fail independently and partially, latency is variable and sometimes huge, and the network can split the system into groups that cannot talk to each other. Every design must say what happens in each case.

### The fallacies

Engineers new to distributed systems tend to assume (the "fallacies of distributed computing"): the network is reliable, latency is zero, bandwidth is infinite, the network is secure, topology does not change, there is one administrator, transport cost is zero, the network is homogeneous. Each false assumption is a class of outage.

### The timeout problem

The shortener calls a database; the payment service calls a bank. The call times out after 2 seconds. Did the bank charge the card? **You cannot know.** The request may have been lost on the way, processed with the reply lost, or still be in progress. Three responses:

1. **Retry** — risks doing it twice unless the operation is **idempotent** (Unit 8.3, Unit 12.7).
2. **Give up** — risks never doing it.
3. **Ask** — query the other side's state later ("did order 42 get charged?"), if it supports that.

Every call that crosses the network must be designed for all three outcomes. This is the single most important idea in the chapter.

### Partial failure, latency, partitions

- **Partial failure:** one of five replicas is slow, one disk is full, one region is down. The system is neither up nor down — it is degraded, and code must handle it.
- **Latency is a distribution:** p50 1 ms, p99 50 ms, p99.9 2 s. A request that calls 100 services in parallel almost certainly hits someone's p99 ("tail at scale"). Timeouts, hedged requests, and fewer dependencies help.
- **Partitions:** a network fault splits nodes into groups. Each group may believe the others are dead. If both groups keep accepting writes, they diverge ("split brain"). Unit 12.4 is about choosing what to do.

### Deep & Tricky

**DT-281. Exactly-once *delivery* is impossible; exactly-once *effects* are not.** *(Theorem + practice)* With message loss and crashes, a sender must retry, so a receiver may see duplicates; or it must not retry, and may lose messages. Systems that advertise "exactly once" achieve exactly-once **processing**: at-least-once delivery plus idempotent handling (deduplication or naturally idempotent operations). The project does exactly this.

### Mastery check

1. **Explain.** A client sends `POST /payments` and gets a timeout. List what may have happened and what the client should do in each case.
2. **Explain.** Why does calling 100 backends in parallel make the p99 of the whole request much worse than any one backend's p99?

## Unit 12.2 — Time and ordering

**One-line summary:** Clocks on different machines disagree and can jump, so wall-clock timestamps cannot reliably order events across machines. **Logical clocks** (Lamport clocks, vector clocks) capture the "happened-before" order that matters, and Unit 7.7's happens-before idea reappears at the scale of networks.

### Physical clocks

Each machine's clock drifts (parts per million), NTP corrects it periodically (possibly jumping backwards), and leap seconds and virtualisation add surprises. Typical skew between well-synchronised servers is milliseconds; badly configured ones can be off by minutes. Consequences:

- "Last write wins" by timestamp can lose the *actual* last write if its machine's clock is behind.
- A lease ("I own this lock until 12:00:05") is unsafe if clocks disagree about when 12:00:05 is.
- Within one process, measure durations with the monotonic clock (Unit 5.6); across processes, do not compare raw timestamps for correctness.

(Google's Spanner uses GPS and atomic clocks to bound uncertainty and *waits out* the uncertainty interval before committing — an expensive, hardware-assisted solution that shows how hard the problem is.)

### Lamport clocks

Each process keeps a counter. Increment it for each local event; send it with each message; on receiving, set it to `max(local, received) + 1`. If event A happened before event B (same process earlier, or A is a send and B the matching receive, transitively), then `L(A) < L(B)`. The converse does not hold: `L(A) < L(B)` does not prove causality.

```go
type Lamport struct {
	mu sync.Mutex
	t  uint64
}

func (l *Lamport) Tick() uint64 { l.mu.Lock(); defer l.mu.Unlock(); l.t++; return l.t }

func (l *Lamport) Observe(remote uint64) uint64 {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.t = max(l.t, remote) + 1
	return l.t
}
```

Tie-breaking by process ID gives a total order consistent with causality — enough for many replication and locking algorithms.

### Vector clocks

Each process keeps a vector of counters, one per process. They can tell "A before B", "B before A", or "**concurrent**" — which is exactly what you need to detect conflicting concurrent updates (two people edited the same document offline). Dynamo-style databases and collaborative editors use them or their relatives (version vectors, CRDTs).

### Deep & Tricky

**DT-282. Database-generated IDs and timestamps do not order events across services.** *(Practice)* `created_at` from two services' clocks or two databases' sequences cannot be merged into a reliable global order. When order matters, give events a sequence from a single authority (one partition of a log, Unit 12.6), or use logical clocks, or design so that order does not matter (commutative operations like counters).

### Mastery check

1. **Explain.** Why can last-write-wins by wall-clock time lose the newest update?
2. **Write.** Simulate three processes exchanging messages with Lamport clocks; print each event's clock and verify that every receive has a larger clock than its send.

## Topic 12B — Distributing data

## Unit 12.3 — Replication and partitioning

**One-line summary:** **Replication** keeps copies of the same data on several machines for availability and read scaling; **partitioning (sharding)** splits data across machines for capacity and write scaling. **Consistent hashing** assigns keys to partitions so that adding or removing a node moves only a small fraction of keys.

### Replication

- **Leader–follower (primary–replica):** one leader accepts writes and streams changes to followers, which serve reads. PostgreSQL streaming replication, MySQL, most systems. **Synchronous** replication waits for followers before acknowledging (no data loss on leader failure, slower writes); **asynchronous** does not (fast, but the last writes can be lost on failover, and followers lag — **replication lag**: a user writes, then reads from a follower and does not see their own write).
- **Multi-leader:** several nodes accept writes (multi-region setups); concurrent writes conflict and need resolution.
- **Leaderless (Dynamo-style):** clients write to and read from several replicas; with N replicas, writing to W and reading from R where R + W > N guarantees overlap (quorums). Cassandra, DynamoDB's ancestry.

Common read-after-write fix: read from the leader for a short time after a user's own write, or carry a "minimum log position" with the session.

### Partitioning

Split data by key: **range partitioning** (keys a–f on node 1, g–m on node 2: good for range scans, prone to hot spots) or **hash partitioning** (hash(key) mod N: even spread, no range scans). Choose the partition key so that the common queries hit one partition and load spreads evenly. A viral short link or a celebrity's account is a **hot key** that no partitioning scheme spreads by itself — it needs caching (Unit 9.7), replication of that key, or splitting the key (sharded counters).

### Consistent hashing

With `hash(key) % N`, changing N from 4 to 5 moves about 80% of keys — a disaster for caches. **Consistent hashing** places nodes and keys on a ring of hash values; each key belongs to the next node clockwise. Adding a node moves only the keys between it and its predecessor (about 1/N of them). **Virtual nodes** (each server placed at many points) even out the load.

```go
type Ring struct {
	points []uint64          // sorted hash points
	owner  map[uint64]string // point -> node
}

func NewRing(nodes []string, vnodes int) *Ring {
	r := &Ring{owner: map[uint64]string{}}
	for _, n := range nodes {
		for i := range vnodes {
			h := hash64(fmt.Sprintf("%s#%d", n, i))
			r.points = append(r.points, h)
			r.owner[h] = n
		}
	}
	slices.Sort(r.points)
	return r
}

func (r *Ring) Node(key string) string {
	h := hash64(key)
	i, _ := slices.BinarySearch(r.points, h) // first point >= h (Unit 6.5)
	if i == len(r.points) {
		i = 0 // wrap around the ring
	}
	return r.owner[r.points[i]]
}

func hash64(s string) uint64 {
	h := fnv.New64a()
	h.Write([]byte(s))
	return h.Sum64()
}
```

Used by distributed caches (Memcached clients, Redis Cluster uses a related fixed-slot scheme), Dynamo-style databases, and load balancers that need session affinity.

### Mastery check

1. **Explain.** Why does `hash % N` move most keys when N changes, and how does consistent hashing avoid it?
2. **Write.** Test the ring above: with 4 nodes and 100 virtual nodes each, assign 100,000 keys, add a fifth node, and measure the fraction of keys that moved and the load spread. Then try 1 virtual node per server.

## Unit 12.4 — CAP / PACELC and consistency models

**One-line summary:** During a network **p**artition, a replicated system must choose between **c**onsistency (refuse some requests) and **a**vailability (answer with possibly stale or divergent data) — CAP. **E**lse, in normal operation, it trades **l**atency against **c**onsistency — PACELC. Consistency models (linearizable, sequential, causal, eventual, read-your-writes) describe precisely what clients can observe.

### CAP, properly stated

CAP is often misquoted as "pick two of three". Partitions are not optional — networks fail — so the real choice is what to do *during* a partition:

- **CP:** keep consistency; the minority side refuses writes (and maybe reads). Example: a Raft-based store (etcd, Consul), a single-leader database whose leader is unreachable.
- **AP:** stay available; both sides accept requests and reconcile later. Example: Dynamo-style stores, DNS, caches.

### PACELC

Most of the time there is no partition, and the trade-off is **latency vs consistency**: synchronously waiting for replicas (consistent, slower) or answering from the nearest copy (fast, possibly stale). Spanner is PC/EC; Cassandra with low consistency levels is PA/EL.

### Consistency models (strongest to weakest, simplified)

| Model | Guarantee | Cost |
| --- | --- | --- |
| **Linearizable** | every operation appears to take effect at one instant between its start and end; everyone sees the same order, respecting real time | coordination on every operation |
| **Sequential** | everyone sees the same order, consistent with each client's program order (not necessarily real time) | coordination |
| **Causal** | operations related by happens-before are seen in that order by everyone; concurrent ones may be seen in different orders | track dependencies (vector clocks) |
| **Read-your-writes / monotonic reads** | a client always sees its own writes, and never goes back in time | session tracking |
| **Eventual** | if writes stop, replicas converge | none during operation |

Choose per use case, not per system: account balances and inventory reservations need strong guarantees; like counts, view counters, and recommendations are fine eventually consistent. The Chapter 9 shortener's hit counts are eventually consistent (up to a second behind), deliberately.

### Mastery check

1. **Explain.** Why is "CA" (consistent and available, no partition tolerance) not a real option for a distributed system?
2. **Explain.** For a shopping cart, a seat reservation, and a "people also bought" list, which consistency model does each need, and why?

## Unit 12.5 — Consensus concepts: Raft

**One-line summary:** Consensus lets a group of machines agree on a sequence of values (a replicated log) despite failures, as long as a **majority** is alive and connected. **Raft** does it with an elected leader that replicates log entries to followers and commits an entry once a majority has stored it. etcd, Consul, CockroachDB, and many others build on it.

### The pieces of Raft

- **Terms:** time is divided into numbered terms; each has at most one leader.
- **Leader election:** followers expect regular heartbeats from the leader. A follower that hears nothing for a randomised **election timeout** (e.g. 150–300 ms) becomes a candidate, increments the term, votes for itself, and asks others for votes. A node grants at most one vote per term, and only to a candidate whose log is at least as up to date as its own. A candidate with votes from a **majority** becomes leader. Randomised timeouts make split votes rare.
- **Log replication:** clients send commands to the leader; it appends them to its log and sends them to followers. Once a **majority** has stored an entry, it is **committed**; the leader applies it to its state machine and tells followers, who apply it too. Every node applies the same commands in the same order, so they reach the same state.
- **Safety:** a committed entry is present in the log of every future leader (because any majority that elects a leader overlaps with the majority that stored the entry, and voters reject candidates with older logs).

### Majorities and fault tolerance

With 2f + 1 nodes, Raft tolerates f failures: 3 nodes tolerate 1, 5 tolerate 2. A cluster of 4 tolerates only 1 (a majority of 4 is 3), so odd sizes are used. During a partition, only the side with a majority can elect a leader and commit — Raft is CP (Unit 12.4).

### When you need it

Rarely directly. You use consensus *through* systems: etcd or ZooKeeper for configuration, leader election, and distributed locks; Kubernetes stores its state in etcd. Use `go.etcd.io/etcd/client/v3`'s `concurrency` package for leader election among your service instances rather than inventing a protocol. If you ever implement Raft, use the `etcd-io/raft` or `hashicorp/raft` libraries — correct consensus implementations take years to harden.

### Deep & Tricky

**DT-283. A distributed lock needs a fencing token.** *(Classic bug)* A client acquires a lock with a 10-second lease, pauses for 15 seconds (a GC pause, a VM migration), the lease expires, another client acquires the lock, and the first client wakes up and writes, believing it still holds the lock. The fix: the lock service issues a monotonically increasing **fencing token** with each grant, and the storage rejects writes carrying a token older than the newest it has seen.

### Mastery check

1. **Explain.** Why must a Raft candidate have an up-to-date log to win an election? What could go wrong otherwise?
2. **Explain.** Why do clusters have 3 or 5 nodes, not 4?

## Topic 12C — Communication and reliability

## Unit 12.6 — Messaging

**One-line summary:** Message queues and logs (RabbitMQ, Kafka, NATS, SQS, or a database table) decouple producers from consumers in time and load. Their delivery guarantee is usually **at least once**, so consumers must be **idempotent**. The **transactional outbox** pattern publishes events reliably together with database changes.

### Queues vs logs

- **Queue** (RabbitMQ, SQS): messages are delivered to one of several competing consumers and deleted when acknowledged. Good for distributing work.
- **Log** (Kafka, Redpanda, NATS JetStream): an append-only, partitioned, retained sequence; each consumer group tracks its own position (offset) and can replay. Order is guaranteed **within a partition**; choose the partition key (an order ID) so related events stay in order. Good for event streams many services read.

### Delivery guarantees

- **At most once:** acknowledge before processing; a crash loses the message.
- **At least once:** acknowledge after processing; a crash causes redelivery — duplicates (DT-281).
- **Exactly-once effects:** at least once + idempotent consumers (deduplicate by message ID, or make operations naturally idempotent: `SET status = 'paid' WHERE status = 'placed'`, upserts, `ON CONFLICT DO NOTHING`).

### The dual-write problem and the outbox

A service must update its database **and** publish an event. Two separate operations can half-fail: the database commit succeeds and the publish fails (event lost: downstream never ships the order), or the publish succeeds and the commit fails (an event for an order that does not exist). The **transactional outbox**: write the event into an `outbox` table **in the same database transaction** as the business change; a separate relay reads the outbox and publishes to the broker (at least once), marking rows as sent. The project goes one step further and uses the outbox table *directly as the queue*, which is a sound design for moderate volumes on PostgreSQL:

```go
// Place stores the order and its event in ONE transaction: either both
// exist or neither does. No "saved the order but lost the event".
func Place(ctx context.Context, db *pgxpool.Pool, id string, amount int64) error {
	return pgx.BeginFunc(ctx, db, func(tx pgx.Tx) error {
		if _, err := tx.Exec(ctx, `INSERT INTO orders (id, amount_paise) VALUES ($1, $2)`, id, amount); err != nil {
			return err
		}
		return queue.Publish(ctx, tx, "order.placed", OrderEvent{id, amount})
	})
}
```

Change data capture (Debezium reading the database's write-ahead log) is the other common way to turn database changes into events without dual writes.

### Competing consumers with `SKIP LOCKED`

```sql
SELECT id, topic, payload, attempts FROM messages
WHERE topic = $1 AND available_at <= now()
ORDER BY id
LIMIT 1
FOR UPDATE SKIP LOCKED
```

`FOR UPDATE` locks the claimed row for this transaction; `SKIP LOCKED` makes other consumers skip rows already locked instead of waiting, so many workers process different messages in parallel. If a worker crashes, its transaction rolls back, the lock is released, and another worker picks the message up — automatic redelivery.

### Deep & Tricky

**DT-284. Ordering and parallelism pull in opposite directions.** *(Design trade-off)* Strict order means one consumer at a time per ordering key; parallelism means messages can complete out of order (DT-239). Kafka resolves it with partitions (order per key, parallelism across keys). In the project's table queue, messages for one order travel on different topics in sequence, so per-order order is ensured by the workflow itself; within a topic, consumers may finish out of order — and nothing depends on it.

### Mastery check

1. **Explain.** Describe both half-failures of a dual write and how the outbox removes them.
2. **Explain.** Why does `SKIP LOCKED` give automatic redelivery when a worker crashes?

## Unit 12.7 — Reliability patterns

**One-line summary:** Every remote call gets a **timeout**; retries are only for **idempotent** operations and use **exponential backoff with jitter** and a cap; **idempotency keys** make non-idempotent operations safe to retry; **circuit breakers** stop calling a failing dependency; **bulkheads** keep one failing dependency from exhausting shared resources; **dead-letter queues** take poison messages out of the way.

### Timeouts and deadlines

Covered in Units 7.9 and 8.7: a context deadline on every call, propagated downstream. A timeout should be shorter than the caller's own deadline, leaving time to handle the failure.

### Retries with exponential backoff and jitter

Immediate retries by thousands of clients hammer a recovering service ("retry storm"). Back off exponentially and add **jitter** so clients do not retry in lockstep. "Full jitter" (a random delay between 0 and the exponential backoff) spreads load best. From the project:

```go
backoff := c.BaseBackoff << (attempts - 1)               // 1x, 2x, 4x, ...
delay := time.Duration(rand.Int64N(int64(backoff) + 1)) // full jitter: uniform in [0, backoff]
```

Rules: retry only transient failures (timeouts, 503, connection reset, serialization failures) — never 400s; cap attempts and total time; retry at **one** layer (if every layer of a five-service chain retries 3 times, one failure becomes 243 calls); and respect `Retry-After`.

### Idempotency keys

For operations that are not naturally idempotent (charging money, sending an email), the caller generates a unique key per *logical* operation and sends it with every attempt; the server stores the key with the result and returns the stored result for repeats. In the project, the payment handler uses the order ID as the gateway's idempotency key, so a redelivered message cannot charge twice — **even though the gateway call is outside the database transaction**. The test's gateway counts real charges per key: 162 calls, 90 charges, never two for one order.

### Circuit breakers

When a dependency is failing, continuing to call it wastes resources and slows every request to the timeout. A circuit breaker tracks recent failures:

- **Closed** (normal): calls go through; if failures exceed a threshold, it **opens**.
- **Open:** calls fail immediately (fast failure, perhaps a fallback), giving the dependency time to recover.
- **Half-open:** after a cool-down, a few trial calls go through; success closes the circuit, failure reopens it.

Libraries: `github.com/sony/gobreaker`. Combine with fallbacks where possible (serve cached data, degrade a feature).

### Bulkheads

Separate resource pools per dependency (separate HTTP clients with their own connection limits, separate worker pools, separate database pools for different workloads), so that one slow dependency exhausting its pool does not take down unrelated requests — like watertight compartments in a ship. The semaphore of Unit 7.10 per dependency is the simplest bulkhead.

### Dead-letter queues

A **poison message** (a payload that can never be processed) retried forever blocks progress and burns resources. After a maximum number of attempts, or immediately for permanent errors, move it to a dead-letter table or queue, alert on it, and let a human inspect and replay. The project does this; its test includes a poison message, which lands in `dead_letters` and does not disturb the 100 real orders.

### Deep & Tricky

**DT-285. A retry is a new request to a system that may already be overloaded.** *(Operations)* Retries multiply load exactly when the dependency is weakest. Retry budgets (e.g. retries may add at most 10% to normal traffic), circuit breakers, and backoff with jitter keep retries from turning a brief slowdown into a prolonged outage.

### Pattern

**P-38. At-least-once delivery + idempotent handlers + bounded retries with jittered backoff + a dead-letter queue.** *(Distributed-systems practice)*

- *Problem it solves:* lost messages, duplicated effects, retry storms, and stuck poison messages.
- *How:* publish via an outbox; consume with acknowledgement after processing; deduplicate by message ID (or make effects idempotent) in the same transaction as the effects; pass idempotency keys to external systems; retry transient errors with capped exponential backoff and full jitter; dead-letter permanent failures and exhausted retries.
- *When not to:* fire-and-forget telemetry may accept at-most-once and skip all of this.
- *Trade-off:* more tables and code, and duplicates must be designed for everywhere — in exchange for no lost work and no double charges.

### Mastery check

1. **Explain.** Why must retries happen at only one layer of a call chain?
2. **Write.** A generic `Retry(ctx, policy, f)` with capped exponential backoff, full jitter, a maximum elapsed time, and a predicate for retryable errors; test it with an injected random source and clock.

## Unit 12.8 — Traffic control

**One-line summary:** **Rate limiting** protects services and enforces fairness; the main algorithms are fixed window, sliding window, token bucket, and leaky bucket. **Load balancing** spreads requests across instances; algorithms range from round robin to least-connections and "power of two choices", and must account for health.

### Rate-limiting algorithms

| Algorithm | How | Trade-offs |
| --- | --- | --- |
| Fixed window | count requests per calendar window (per minute) | simple; allows 2× bursts at window boundaries |
| Sliding window log | keep timestamps of recent requests | exact; memory per request |
| Sliding window counter | weight the previous window's count by overlap | close to exact, cheap |
| Token bucket | tokens refill at rate r up to burst b; each request spends one | allows controlled bursts; the Chapter 11 project's limiter |
| Leaky bucket | requests drain from a queue at a fixed rate | smooths output; adds queueing delay |

Distributed limits (the same client hitting any of several instances) need shared state: Redis with an atomic script (a token bucket in Lua, or `INCR` with `EXPIRE` for fixed windows), or approximate per-instance limits of r/N. Respond with 429 and `Retry-After`; limit per API key or user, not only per IP (many users share IPs behind NAT).

### Load balancing

- **Round robin:** simple; ignores differences in load and request cost.
- **Least connections / least outstanding requests:** sends to the least busy instance; good for variable request costs.
- **Power of two choices:** pick two instances at random, send to the less loaded one. Nearly as good as global least-loaded, with no global state — used widely in modern proxies.
- **Consistent hashing:** for affinity (cache locality, sticky sessions), Unit 12.3.
- **Health:** remove instances failing readiness (Unit 11.5); slow-start new instances; eject outliers with high error rates.

L4 balancers forward TCP connections (cheap; one long-lived HTTP/2 or gRPC connection pins all its requests to one backend — DT-286); L7 balancers understand HTTP and balance individual requests, retry, and route by path.

### Deep & Tricky

**DT-286. Long-lived connections defeat connection-level load balancing.** *(Operations)* gRPC and HTTP/2 clients multiplex many requests over one connection; behind an L4 load balancer, all of a client's traffic goes to one backend, and newly added backends receive nothing. Use L7 balancing, client-side balancing (gRPC's `round_robin` policy with DNS or xDS), or periodic connection recycling (`MaxConnectionAge` on gRPC servers).

### Mastery check

1. **Explain.** Show how a fixed-window limiter of 100/minute can admit 200 requests within two seconds.
2. **Write.** A simulation comparing random assignment, round robin, and power-of-two-choices for 10 servers with random request durations; report the maximum queue length for each.

## Topic 12D — Architecture

## Unit 12.9 — Architecture patterns

**One-line summary:** Caching at scale, **CQRS** (separate models for writes and reads), **event sourcing** (store events, derive state), and **sagas** (multi-step workflows with compensating actions instead of distributed transactions) are tools for specific problems. Each adds complexity; use them when the problem demands it.

### Caching at scale

Layers (Unit 9.7) multiply: CDN → reverse-proxy cache → service cache → database cache. At scale, the risks grow: cache stampedes (singleflight, early probabilistic refresh), hot keys (local in-process caches in front of Redis), cold starts (warm caches gradually, DT-260), and consistency (invalidate via events from the outbox).

### CQRS

Command Query Responsibility Segregation separates the **write model** (normalised, enforcing invariants) from one or more **read models** (denormalised for specific queries — a search index, a reporting table), updated from events. The Chapter 9 `daily_hits` table is a tiny read model maintained alongside the write. Benefits: each side is optimised for its job and scales independently. Costs: read models are eventually consistent and there is more code and infrastructure.

### Event sourcing

Instead of storing current state, store the sequence of **events** ("OrderPlaced", "PaymentCaptured", "OrderShipped") and derive state by replaying them (with snapshots for speed). Benefits: a complete audit history, the ability to rebuild read models or fix bugs by replaying, temporal queries ("what was the balance on 1 March?"). Costs: schema evolution of events is hard (events are forever), queries need projections, and the mental model is unfamiliar. Natural for ledgers and domains where history *is* the data; overkill for most CRUD.

### Sagas

A business process spanning several services (order → payment → shipping) cannot use one database transaction, and distributed transactions (two-phase commit) are slow, fragile, and unavailable across most modern systems. A **saga** is a sequence of local transactions, each publishing an event that triggers the next; if a step fails, **compensating transactions** undo the earlier steps (refund the payment, cancel the order, release the reserved stock).

- **Choreography:** each service reacts to events (the project: `order.placed` → payment → `payment.succeeded` → shipping; `payment.failed` → cancel). Simple for short flows; hard to follow as flows grow.
- **Orchestration:** a central coordinator tells each service what to do and tracks the saga's state. Easier to understand, monitor, and change for long flows. (Workflow engines such as Temporal implement durable orchestration.)

Compensations must be idempotent and must handle "the step you are compensating may not have happened yet" (a cancellation that arrives before the payment event).

### Mastery check

1. **Explain.** Why do sagas need compensating actions instead of rollbacks?
2. **Explain.** When would you choose event sourcing, and what is the hardest long-term cost?

## Unit 12.10 — System design method

**One-line summary:** Design systems in a fixed order: clarify **requirements** (functional and non-functional), **estimate** load and storage, define **APIs** and **data models**, sketch the **architecture**, then walk through **failure modes**, trade-offs, and evolution. Write it down as a design document.

### The steps

1. **Requirements.** Functional: what users can do (shorten URLs, redirect, see stats). Non-functional, with numbers: availability (99.9% = 43 minutes of downtime a month; 99.99% = 4.3 minutes), latency (p99 redirect < 50 ms), durability, consistency needs (Unit 12.4), security and compliance. Out of scope, explicitly.
2. **Estimates (back-of-the-envelope).** Requests per second (average and peak), read/write ratio, data size per record × records × years, bandwidth, cache size for the hot set. Example for a shortener: 100 M new links/month ≈ 40 writes/s average, perhaps 400/s peak; 100:1 reads ≈ 4,000 redirects/s average, 40,000 peak; 500 bytes per link × 1.2 B links/year ≈ 600 GB/year. Round numbers to orders of magnitude; the point is to find which component is the bottleneck.
3. **APIs.** Endpoints or RPCs with request/response shapes, errors, idempotency, pagination (Unit 8.6).
4. **Data model.** Entities, keys, indexes, partition keys (Unit 9.1–9.2, 12.3), retention.
5. **High-level architecture.** Clients, load balancers, stateless services, caches, databases, queues, background workers — and why each exists. Start simple: one service and one database handle more than most people expect (thousands of requests per second on modest hardware).
6. **Deep dives** on the hardest parts (code generation without collisions, the hot-link problem, analytics at scale).
7. **Failure modes.** For every component: what if it is slow, down, or wrong? What if a whole zone fails? What do users see? How is it detected (metrics, alerts) and recovered (failover, replay, runbook)?
8. **Trade-offs and evolution.** What you chose not to do, what would change at 10× scale, what you would build next.

### Useful numbers (orders of magnitude)

| Operation | Time |
| --- | --- |
| L1 cache reference | ~1 ns |
| Main memory reference | ~100 ns |
| Read 1 MB sequentially from memory | ~10–50 µs |
| Round trip within a datacentre | ~0.5 ms |
| SSD random read | ~100 µs |
| Read 1 MB sequentially from SSD | ~0.2–1 ms |
| Round trip between continents | ~100–150 ms |

(Unit 0.5's table, extended. Exact values vary; the ratios drive designs.)

### Mastery check

1. **Explain.** Why estimate before designing? Give an example where an estimate changes the design.
2. **Write.** A two-page design document for a "notifications service" (email, SMS, push; 10 M users; per-user preferences; at most one SMS per user per minute), following the steps above, including failure modes.

## Chapter 12 Project — An event-driven order pipeline with idempotent consumers

**Goal:** build an order workflow — place an order, charge payment, ship or cancel — as a choreographed saga over a durable PostgreSQL-backed queue, with at-least-once delivery, idempotent consumers, retries with exponential backoff and full jitter, dead-lettering, and an external payment gateway called with idempotency keys. Prove its guarantees under injected failures.

### Design

```
Place ──order.placed──▶ payments ──payment.succeeded──▶ shipping
                                 └─payment.failed────▶ cancel (compensation)
```

- **`queue` package:** `messages` table as transactional outbox *and* queue; `Publish(ctx, tx, topic, v)` writes inside the caller's transaction; `Consumer` claims messages with `FOR UPDATE SKIP LOCKED`, records `(consumer, message_id)` in `processed` for idempotency, runs the handler inside a **savepoint**, deletes the message on success, and on failure records the error and reschedules with backoff and jitter, or moves it to `dead_letters`.
- **`shop` package:** `Place` writes the order and its event atomically; `PaymentHandler` charges through a `Gateway` interface with the order ID as idempotency key; `ShippingHandler` and `CancelHandler` are idempotent by construction (`ON CONFLICT DO NOTHING`, guarded status transitions).
- **The test** places 100 orders (10 to be declined) plus one poison message, runs three competing workers per topic, uses a gateway that fails transiently 30% of the time, and injects a crash *after* a handler's effects but *before* commit for every fifth message.

### Reference solution — `queue/queue.go`

```go
// Package queue is a durable message queue stored in PostgreSQL.
//
// Publishing happens inside the caller's transaction, so a message is
// stored if and only if the business change commits: the transactional
// outbox pattern, with the outbox table serving directly as the queue.
// Delivery is at least once; consumers must be idempotent.
package queue

import (
	"context"
	"encoding/json"
	"errors"
	"fmt"
	"math/rand/v2"
	"time"

	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgxpool"
)

const Schema = `
CREATE TABLE IF NOT EXISTS messages (
    id           bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    topic        text        NOT NULL,
    payload      jsonb       NOT NULL,
    attempts     int         NOT NULL DEFAULT 0,
    available_at timestamptz NOT NULL DEFAULT now(),
    last_error   text
);
CREATE INDEX IF NOT EXISTS messages_ready ON messages (topic, available_at);
CREATE TABLE IF NOT EXISTS dead_letters (
    id       bigint PRIMARY KEY,
    topic    text  NOT NULL,
    payload  jsonb NOT NULL,
    attempts int   NOT NULL,
    error    text  NOT NULL,
    died_at  timestamptz NOT NULL DEFAULT now()
);
CREATE TABLE IF NOT EXISTS processed (
    consumer   text   NOT NULL,
    message_id bigint NOT NULL,
    PRIMARY KEY (consumer, message_id)
);`

// Publish stores a message in the caller's transaction.
func Publish(ctx context.Context, tx pgx.Tx, topic string, v any) error {
	b, err := json.Marshal(v)
	if err != nil {
		return err
	}
	_, err = tx.Exec(ctx, `INSERT INTO messages (topic, payload) VALUES ($1, $2)`, topic, b)
	return err
}

// Message is one delivery. ID is stable across redeliveries.
type Message struct {
	ID       int64
	Topic    string
	Payload  []byte
	Attempts int
}

// Handler processes one message inside tx. Returning nil commits the
// message's removal together with everything the handler wrote in tx.
type Handler func(ctx context.Context, tx pgx.Tx, m Message) error

// ErrPermanent marks failures that retrying cannot fix (bad payload):
// the message goes straight to the dead-letter table.
var ErrPermanent = errors.New("permanent failure")

type Consumer struct {
	DB          *pgxpool.Pool
	Name        string // used for idempotency records
	Topic       string
	Handle      Handler
	MaxAttempts int
	BaseBackoff time.Duration
}

// Run polls for messages until ctx is done.
func (c *Consumer) Run(ctx context.Context) error {
	for {
		n, err := c.ProcessBatch(ctx, 10)
		if err != nil && ctx.Err() == nil {
			return err
		}
		if n == 0 {
			select {
			case <-ctx.Done():
				return nil
			case <-time.After(50 * time.Millisecond):
			}
		}
	}
}

// ProcessBatch claims up to n ready messages and handles each in its own
// transaction. FOR UPDATE SKIP LOCKED lets many consumers work in parallel
// without taking the same message.
func (c *Consumer) ProcessBatch(ctx context.Context, n int) (int, error) {
	done := 0
	for range n {
		ok, err := c.processOne(ctx)
		if err != nil || !ok {
			return done, err
		}
		done++
	}
	return done, nil
}

func (c *Consumer) processOne(ctx context.Context) (bool, error) {
	tx, err := c.DB.Begin(ctx)
	if err != nil {
		return false, err
	}
	defer tx.Rollback(context.WithoutCancel(ctx))

	var m Message
	err = tx.QueryRow(ctx, `
		SELECT id, topic, payload, attempts FROM messages
		WHERE topic = $1 AND available_at <= now()
		ORDER BY id
		LIMIT 1
		FOR UPDATE SKIP LOCKED`, c.Topic).Scan(&m.ID, &m.Topic, &m.Payload, &m.Attempts)
	if errors.Is(err, pgx.ErrNoRows) {
		return false, nil
	}
	if err != nil {
		return false, err
	}

	// Idempotency: if this consumer already processed this message (a
	// previous attempt committed its effects but crashed before the
	// message was deleted), skip the work.
	tag, err := tx.Exec(ctx, `INSERT INTO processed (consumer, message_id) VALUES ($1, $2)
		ON CONFLICT DO NOTHING`, c.Name, m.ID)
	if err != nil {
		return false, err
	}
	if tag.RowsAffected() == 1 {
		// Run the handler in a savepoint so a failure can be recorded
		// without losing the claim on the message.
		herr := pgx.BeginFunc(ctx, tx, func(sp pgx.Tx) error { return c.Handle(ctx, sp, m) })
		if herr != nil {
			if _, err := tx.Exec(ctx, `DELETE FROM processed WHERE consumer = $1 AND message_id = $2`, c.Name, m.ID); err != nil {
				return false, err
			}
			if err := c.fail(ctx, tx, m, herr); err != nil {
				return false, err
			}
			return true, tx.Commit(ctx)
		}
	}
	if _, err := tx.Exec(ctx, `DELETE FROM messages WHERE id = $1`, m.ID); err != nil {
		return false, err
	}
	return true, tx.Commit(ctx)
}

// fail schedules a retry with exponential backoff and full jitter, or
// moves the message to dead_letters.
func (c *Consumer) fail(ctx context.Context, tx pgx.Tx, m Message, herr error) error {
	attempts := m.Attempts + 1
	if attempts >= c.MaxAttempts || errors.Is(herr, ErrPermanent) {
		_, err := tx.Exec(ctx, `
			WITH moved AS (DELETE FROM messages WHERE id = $1 RETURNING id, topic, payload)
			INSERT INTO dead_letters (id, topic, payload, attempts, error)
			SELECT id, topic, payload, $2, $3 FROM moved`, m.ID, attempts, herr.Error())
		return err
	}
	backoff := c.BaseBackoff << (attempts - 1)              // 1x, 2x, 4x, ...
	delay := time.Duration(rand.Int64N(int64(backoff) + 1)) // full jitter: uniform in [0, backoff]
	_, err := tx.Exec(ctx, `UPDATE messages SET attempts = $2, last_error = $3,
		available_at = now() + $4 * interval '1 microsecond' WHERE id = $1`,
		m.ID, attempts, herr.Error(), delay.Microseconds())
	if err != nil {
		return fmt.Errorf("scheduling retry: %w", err)
	}
	return nil
}
```

Three details carry the correctness:

1. **Savepoint.** `pgx.BeginFunc(ctx, tx, ...)` on an existing transaction creates a *savepoint*. If the handler fails, only its writes are rolled back; the outer transaction keeps its lock on the message and records the failure (`attempts`, `available_at`) — so failures are counted even though the handler's work is undone.
2. **Idempotency record in the same transaction as the effects.** `processed(consumer, message_id)` commits together with the handler's database writes and the message deletion — all or nothing. If the process dies at any point, either everything committed (and a redelivery is impossible because the message is gone) or nothing did (and the redelivery redoes everything).
3. **External effects use idempotency keys.** The gateway call cannot join the transaction, so it can happen several times — the key makes the repeats harmless.

(Why keep the `processed` table at all, if the message is deleted in the same transaction? Because real systems often split consumption from deletion — a relay to Kafka, or handlers that call `queue.Publish` into another database — and the table makes the dedup rule explicit and testable. In this exact design it is defence in depth.)

### Reference solution — `shop/shop.go`

```go
// Package shop is a small event-driven order pipeline:
//
//	Place ──order.placed──▶ payment consumer ──payment.succeeded──▶ shipping consumer
//	                                          └─payment.failed────▶ order cancelled (compensation)
package shop

import (
	"context"
	"encoding/json"
	"errors"
	"fmt"

	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgxpool"

	"example.com/orders/queue"
)

const Schema = `
CREATE TABLE IF NOT EXISTS orders (
    id           text PRIMARY KEY,
    amount_paise bigint NOT NULL CHECK (amount_paise > 0),
    status       text   NOT NULL DEFAULT 'placed'
                 CHECK (status IN ('placed', 'paid', 'cancelled', 'shipped'))
);
CREATE TABLE IF NOT EXISTS shipments (
    order_id text PRIMARY KEY REFERENCES orders (id)
);`

type OrderEvent struct {
	OrderID string `json:"order_id"`
	Amount  int64  `json:"amount_paise"`
}

// Place stores the order and its event in ONE transaction: either both
// exist or neither does. No "saved the order but lost the event".
func Place(ctx context.Context, db *pgxpool.Pool, id string, amount int64) error {
	return pgx.BeginFunc(ctx, db, func(tx pgx.Tx) error {
		if _, err := tx.Exec(ctx, `INSERT INTO orders (id, amount_paise) VALUES ($1, $2)`, id, amount); err != nil {
			return err
		}
		return queue.Publish(ctx, tx, "order.placed", OrderEvent{id, amount})
	})
}

// Gateway is an external payment provider. Charge must be idempotent per
// key: calling it twice with the same key charges once (real providers
// support idempotency keys for exactly this reason).
type Gateway interface {
	Charge(ctx context.Context, idempotencyKey string, amount int64) (approved bool, err error)
}

var ErrDeclined = errors.New("payment declined")

// PaymentHandler charges for an order and publishes the outcome.
func PaymentHandler(gw Gateway) queue.Handler {
	return func(ctx context.Context, tx pgx.Tx, m queue.Message) error {
		var ev OrderEvent
		if err := json.Unmarshal(m.Payload, &ev); err != nil || ev.OrderID == "" {
			return fmt.Errorf("%w: bad payload: %v", queue.ErrPermanent, err)
		}
		// The order ID is the idempotency key: a redelivered message cannot
		// charge the customer twice, even though the gateway is outside our
		// database transaction.
		approved, err := gw.Charge(ctx, "order-"+ev.OrderID, ev.Amount)
		if err != nil {
			return err // transient: retried with backoff
		}
		if !approved {
			return queue.Publish(ctx, tx, "payment.failed", ev)
		}
		if _, err := tx.Exec(ctx, `UPDATE orders SET status = 'paid' WHERE id = $1 AND status = 'placed'`, ev.OrderID); err != nil {
			return err
		}
		return queue.Publish(ctx, tx, "payment.succeeded", ev)
	}
}

// ShippingHandler creates exactly one shipment per paid order.
func ShippingHandler(ctx context.Context, tx pgx.Tx, m queue.Message) error {
	var ev OrderEvent
	if err := json.Unmarshal(m.Payload, &ev); err != nil {
		return fmt.Errorf("%w: %v", queue.ErrPermanent, err)
	}
	if _, err := tx.Exec(ctx, `INSERT INTO shipments (order_id) VALUES ($1) ON CONFLICT DO NOTHING`, ev.OrderID); err != nil {
		return err
	}
	_, err := tx.Exec(ctx, `UPDATE orders SET status = 'shipped' WHERE id = $1 AND status = 'paid'`, ev.OrderID)
	return err
}

// CancelHandler is the saga's compensating action for a failed payment.
func CancelHandler(ctx context.Context, tx pgx.Tx, m queue.Message) error {
	var ev OrderEvent
	if err := json.Unmarshal(m.Payload, &ev); err != nil {
		return fmt.Errorf("%w: %v", queue.ErrPermanent, err)
	}
	_, err := tx.Exec(ctx, `UPDATE orders SET status = 'cancelled' WHERE id = $1 AND status = 'placed'`, ev.OrderID)
	return err
}
```

Status transitions are guarded (`WHERE status = 'placed'`), so applying any handler twice changes nothing the second time — naturally idempotent effects.

### Reference solution — `shop/shop_test.go` (the failure-injection test)

```go
// flakyGateway fails transiently 30% of the time, declines orders whose
// amount ends in 13, and remembers keys so a repeated key charges once.
type flakyGateway struct {
	mu      sync.Mutex
	charged map[string]int // key -> number of real charges
	calls   int
	r       *mrand.Rand
}

func (g *flakyGateway) Charge(_ context.Context, key string, amount int64) (bool, error) {
	g.mu.Lock()
	defer g.mu.Unlock()
	g.calls++
	if g.r.IntN(10) < 3 {
		return false, errors.New("gateway timeout")
	}
	if amount%100 == 13 {
		return false, nil
	}
	if g.charged[key] == 0 {
		g.charged[key]++ // the provider deduplicates by idempotency key
	}
	return true, nil
}

// crashAfterEffects simulates a consumer that dies after its handler's
// work but before commit, on the first attempt of every 5th message.
func crashAfterEffects(h queue.Handler) queue.Handler {
	var mu sync.Mutex
	crashed := map[int64]bool{}
	return func(ctx context.Context, tx pgx.Tx, m queue.Message) error {
		if err := h(ctx, tx, m); err != nil {
			return err
		}
		mu.Lock()
		defer mu.Unlock()
		if m.ID%5 == 0 && !crashed[m.ID] {
			crashed[m.ID] = true
			return errors.New("simulated crash before commit")
		}
		return nil
	}
}

func TestPipeline(t *testing.T) {
	db := newDB(t) // fresh database per test, as in Unit 9.8
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	const n = 100
	for i := range n {
		amount := int64(1000 + i*100)
		if i%10 == 0 {
			amount += 13 // declined
		}
		if err := Place(ctx, db, fmt.Sprintf("o%03d", i), amount); err != nil {
			t.Fatal(err)
		}
	}
	// A poison message: retrying can never fix it.
	pgx.BeginFunc(ctx, db, func(tx pgx.Tx) error {
		return queue.Publish(ctx, tx, "order.placed", map[string]string{"nonsense": "x"})
	})

	gw := &flakyGateway{charged: map[string]int{}, r: mrand.New(mrand.NewPCG(1, 2))}
	consumers := []*queue.Consumer{
		{Name: "payments", Topic: "order.placed", Handle: crashAfterEffects(PaymentHandler(gw))},
		{Name: "shipping", Topic: "payment.succeeded", Handle: crashAfterEffects(ShippingHandler)},
		{Name: "cancel", Topic: "payment.failed", Handle: CancelHandler},
	}
	var wg sync.WaitGroup
	for _, c := range consumers {
		c.DB, c.MaxAttempts, c.BaseBackoff = db, 8, 5*time.Millisecond
		for range 3 { // three competing workers per topic
			wg.Add(1)
			go func() {
				defer wg.Done()
				if err := c.Run(ctx); err != nil {
					t.Error(err)
				}
			}()
		}
	}

	deadline := time.Now().Add(20 * time.Second)
	for {
		var left int
		db.QueryRow(ctx, `SELECT count(*) FROM messages`).Scan(&left)
		if left == 0 {
			break
		}
		if time.Now().After(deadline) {
			t.Fatalf("%d messages still queued", left)
		}
		time.Sleep(50 * time.Millisecond)
	}
	cancel()
	wg.Wait()

	// ... count orders by status, shipments, dead letters (see the full file) ...
	if counts["shipped"] != 90 || counts["cancelled"] != 10 || shipments != 90 {
		t.Errorf("want 90 shipped and 10 cancelled, got %v and %d shipments", counts, shipments)
	}
	for k, c := range gw.charged {
		if c != 1 {
			t.Errorf("%s charged %d times", k, c)
		}
	}
	if len(gw.charged) != 90 || dead != 1 {
		t.Errorf("real charges = %d (want 90), dead letters = %d (want 1)", len(gw.charged), dead)
	}
}
```

(The waiting loop polls a database count — acceptable here because the condition is external state, and it has a deadline; Unit 7.12's "no sleeps" rule is about waiting for goroutines you control.)

### Real result

Four consecutive runs with the race detector, all identical:

```
orders by status: map[cancelled:10 shipped:90], shipments: 90, dead letters: 1, gateway calls: 162, real charges: 90
--- PASS: TestPipeline (0.55s)
```

**162 gateway calls** — first attempts, transient-failure retries, and redeliveries after simulated crashes — produced **exactly 90 real charges**, one per approved order; **90 shipments**, one per paid order; **10 cancellations** via the compensating action; and the poison message parked in `dead_letters` after one attempt, without blocking anything.

### Review checklist

- Kill a worker (cancel its context) in the middle of processing: is the message redelivered and processed exactly once in effect?
- Raise the gateway's failure rate to 90%: do messages end up in `dead_letters` after `MaxAttempts`, with the last error recorded?
- Run with one worker and with ten: same final state?
- What happens if the shipping consumer is down for an hour? (Messages wait; nothing is lost; the backlog drains when it returns.)
- **Known limitations to discuss:** the handler runs while the message row is locked, so a slow handler holds a database transaction open (DT-256) — production systems often use a **lease** (`locked_until` timestamp) instead of a long transaction; polling every 50 ms adds latency and load (PostgreSQL `LISTEN/NOTIFY` can wake consumers); the `processed` table grows forever (delete old rows on a schedule); per-order ordering within one topic is not guaranteed.

### Extensions

1. Replace the long transaction with leases: claim with `UPDATE … SET locked_until = now() + interval '30 seconds' … RETURNING`, process outside the transaction, then delete.
2. Add `LISTEN/NOTIFY` so consumers wake immediately on new messages.
3. Add an orchestrated version of the same saga (a coordinator with a `sagas` table) and compare the code and observability.

## Answers to the Chapter 12 prediction questions

**12.1 Q2.** If each backend has a 1% chance of a slow (p99) response, a request that waits for all 100 has a 1 − 0.99¹⁰⁰ ≈ 63% chance of hitting at least one slow response. The slow path becomes the common path.

**12.5 Q2.** A majority of 4 is 3, so 4 nodes tolerate only 1 failure — the same as 3 nodes — while adding one more machine that can fail and one more vote to collect.

# Chapter 13 — Geospatial Computing & Routing Algorithms

*Goal: the domain knowledge for the routing capstone.*

Navigation is one of the most impressive everyday applications of computer science: a phone finds the fastest route across a country with millions of road segments in milliseconds. This chapter builds that capability from the ground up in Go: coordinates and distances, spatial indexes, OpenStreetMap data, a compact road graph, snapping points to roads, and the routing algorithms — Dijkstra, A*, bidirectional search, and the preprocessing techniques (landmarks, contraction hierarchies) that make continental routing fast.

All code in this chapter was built and tested. Because a real OpenStreetMap extract could not be downloaded in the environment used to write this book, the tests and measurements use a **synthetic city** generated as a genuine OSM XML file (a 300 × 300 grid of streets near Pune with arterial roads, one-way streets, varied speeds, and missing segments). The parser and every algorithm treat it exactly like a real extract; run the same code on a real city's extract from Geofabrik or BBBike to see real-world numbers.

**What you will be able to do after this chapter**

- Work correctly with latitude/longitude, WGS84, great-circle distances, projections, geohashes, and polylines.
- Choose and implement spatial indexes (grids, geohash, quadtrees, R-trees; S2 and H3 concepts).
- Read OpenStreetMap data and build a directed, weighted road graph in compact CSR form.
- Snap GPS points to the network.
- Implement and compare Dijkstra, A*, and bidirectional Dijkstra, and explain ALT and contraction hierarchies.
- Return routes users can use: geometry, encoded polylines, distances, and travel times.

## Topic 13A — Geography in code

## Unit 13.1 — Coordinates

**One-line summary:** A position on Earth is a latitude (−90 to 90, north–south) and longitude (−180 to 180, east–west) in the **WGS84** datum used by GPS. Distances on a sphere use the **haversine** formula; maps flatten the Earth with **projections** that always distort something. Order matters: humans say "lat, lon", but many formats (GeoJSON, PostGIS's `ST_MakePoint`) use "lon, lat".

### Latitude, longitude, and what a degree means

- One degree of **latitude** is about 111 km everywhere.
- One degree of **longitude** is about 111 km × cos(latitude): 105 km in Pune (18.5° N), 72 km in London (51.5° N), 0 at the poles.
- Five decimal places (0.00001°) ≈ 1.1 m; seven ≈ 1.1 cm. Store coordinates as `float64` (Unit 1.5) or as fixed-point `int32` in units of 10⁻⁷ degrees (what OSM does internally — 4 bytes instead of 8, exact, and enough precision).

### Haversine distance

```go
// Point is a WGS84 coordinate in degrees.
type Point struct{ Lat, Lon float64 }

const EarthRadiusM = 6_371_008.8 // mean Earth radius in metres

func rad(d float64) float64 { return d * math.Pi / 180 }

// Haversine returns the great-circle distance in metres. It treats the Earth
// as a sphere: error up to about 0.5% versus the WGS84 ellipsoid, which is
// fine for routing heuristics and nearest-neighbour search.
func Haversine(a, b Point) float64 {
	dLat := rad(b.Lat - a.Lat)
	dLon := rad(b.Lon - a.Lon)
	h := math.Sin(dLat/2)*math.Sin(dLat/2) +
		math.Cos(rad(a.Lat))*math.Cos(rad(b.Lat))*math.Sin(dLon/2)*math.Sin(dLon/2)
	return 2 * EarthRadiusM * math.Asin(math.Min(1, math.Sqrt(h)))
}
```

Pune to Mumbai comes out at about 120 km as the crow flies (the test checks it). The `math.Min(1, …)` guards against floating-point rounding pushing the argument of `Asin` just above 1 (which would return NaN — DT-7 in a navigation app). For sub-metre accuracy over long distances, use an ellipsoidal formula (Vincenty, or Karney's algorithm in `github.com/golang/geo` or GeographicLib ports).

For short distances (a few kilometres), the **equirectangular approximation** is several times faster and accurate to centimetres — the snapping index in this chapter uses it:

```go
func Equirectangular(a, b Point) float64 {
	x := rad(b.Lon-a.Lon) * math.Cos(rad((a.Lat+b.Lat)/2))
	y := rad(b.Lat - a.Lat)
	return EarthRadiusM * math.Sqrt(x*x+y*y)
}
```

### Projections

A projection maps the sphere to a plane. **Web Mercator** (EPSG:3857), used by almost every web map, preserves angles and local shapes but inflates areas towards the poles (Greenland looks as big as Africa). **UTM** zones give nearly true metres within a 6°-wide zone — good for local engineering calculations. Rule: store and exchange WGS84 lat/lon (EPSG:4326); project only for display or for local planar computations.

### Deep & Tricky

**DT-287. The antimeridian and the poles break naive code.** *(Geometry fact)* A bounding box from longitude 179° to −179° is 2° wide, not 358°; "is this point inside the box?" with `minLon <= lon <= maxLon` fails across the 180° line (Fiji, the Aleutians, the Pacific routes of every airline). Near the poles, longitude differences mean almost nothing. Most city routing never meets these cases — but global services must.

### Mastery check

1. **Explain.** Why is one degree of longitude shorter in London than in Pune?
2. **Predict.** Roughly how far apart are two points that differ by 0.001° in latitude? In longitude, at 18.5° N?
3. **Write.** `Bearing(a, b Point) float64` returning the initial compass bearing in degrees; test with known city pairs.

## Unit 13.2 — Spatial indexing

**One-line summary:** "Which roads are near this point?" over millions of objects needs a **spatial index**. A uniform **grid** is simplest and fast for evenly spread data; **geohash** encodes cells as sortable strings; **quadtrees** adapt to density; **R-trees** index rectangles and power spatial databases; **S2** and **H3** are global hierarchical cell systems used at Google and Uber.

### Grid index

Divide the bounding box into cells of fixed size; store each object in its cell; to find the nearest object to a point, search the point's cell, then rings of cells outwards until no unvisited cell can contain anything closer than the best found. This chapter's `graph.Index` does exactly that, stored in CSR form (one flat slice of node IDs plus cell offsets, Unit 6.13):

```go
// Nearest returns the closest node and its distance in metres, searching
// rings of cells outwards until no closer node can exist.
func (ix *Index) Nearest(p geo.Point) (int32, float64) {
	r0 := min(max(int((p.Lat-ix.minLat)/ix.cellDeg), 0), ix.rows-1)
	c0 := min(max(int((p.Lon-ix.minLon)/ix.cellDeg), 0), ix.cols-1)
	best, bestD := int32(-1), math.Inf(1)
	cellM := ix.cellDeg * 111_000 * math.Cos(p.Lat*math.Pi/180) // narrowest cell side in metres
	for ring := 0; ring <= max(ix.rows, ix.cols); ring++ {
		if best >= 0 && float64(ring-1)*cellM > bestD {
			break // every unvisited cell is farther than the best found
		}
		for r := r0 - ring; r <= r0+ring; r++ {
			for c := c0 - ring; c <= c0+ring; c++ {
				if r < 0 || c < 0 || r >= ix.rows || c >= ix.cols {
					continue
				}
				if max(abs(r-r0), abs(c-c0)) != ring {
					continue // only the ring's border
				}
				cell := r*ix.cols + c
				for _, v := range ix.nodes[ix.cellStart[cell]:ix.cellStart[cell+1]] {
					if d := geo.Equirectangular(p, ix.g.Point(v)); d < bestD {
						best, bestD = v, d
					}
				}
			}
		}
	}
	return best, bestD
}
```

The stopping rule is the subtle part: a ring at distance k cells can contain points as close as (k − 1) cell widths, so the search stops only when even that lower bound exceeds the best distance. The project's test compares `Nearest` with a brute-force scan on 200 random points — a differential test (as in Unit 6's PQ tests).

Weakness: uneven data. City centres have thousands of nodes per cell; rural cells are empty and the ring search walks far.

### Geohash

A geohash interleaves the bits of longitude and latitude (alternately halving the world) and writes them in base 32. Precision 7 is a cell of about 153 m × 153 m; nearby points usually share a prefix, so a geohash string can be a database key with prefix queries (`LIKE 'te7u%'`). Edge cases: two points a metre apart across a cell boundary can have completely different hashes, so proximity queries always check the neighbouring cells too. The chapter's implementation reproduces the reference value from geohash.org's original example (`u4pruydqqvj` for 57.64911, 10.40744):

```go
const base32 = "0123456789bcdefghjkmnpqrstuvwxyz"

// Geohash encodes p with the given number of characters (precision 7 is a
// cell of about 153 m × 153 m). Nearby points usually share a prefix.
func Geohash(p Point, precision int) string {
	latLo, latHi, lonLo, lonHi := -90.0, 90.0, -180.0, 180.0
	var sb strings.Builder
	bit, ch, even := 0, 0, true
	for sb.Len() < precision {
		if even { // longitude bit
			mid := (lonLo + lonHi) / 2
			if p.Lon >= mid {
				ch |= 1 << (4 - bit)
				lonLo = mid
			} else {
				lonHi = mid
			}
		} else { // latitude bit
			mid := (latLo + latHi) / 2
			if p.Lat >= mid {
				ch |= 1 << (4 - bit)
				latLo = mid
			} else {
				latHi = mid
			}
		}
		even = !even
		if bit++; bit == 5 {
			sb.WriteByte(base32[ch])
			bit, ch = 0, 0
		}
	}
	return sb.String()
}
```

This is binary search (Unit 6.5) on two dimensions at once, and bit manipulation (Unit 1.4) to pack the result.

### Quadtrees, R-trees, S2, H3

- **Quadtree:** recursively split a square into four children until each holds at most k points. Adapts to density: deep in city centres, shallow in the countryside.
- **R-tree:** a balanced tree (a cousin of the B-tree, Unit 6.7) of nested bounding rectangles; indexes shapes (road segments, polygons) as well as points. PostGIS's GiST index is R-tree-like (Unit 9.6). Go packages: `github.com/tidwall/rtree`.
- **S2 (Google):** projects the sphere onto a cube and orders cells along a Hilbert curve, so cell IDs are 64-bit integers whose numeric ranges correspond to regions — spatial queries become integer range scans. `github.com/golang/geo/s2`.
- **H3 (Uber):** a hierarchy of hexagonal cells; hexagons have uniform neighbour distances, which suits aggregation (demand per cell) and movement analysis.

### Mastery check

1. **Explain.** Why must a nearest-neighbour search in a grid continue past the first non-empty cell?
2. **Explain.** Why do two points one metre apart sometimes have geohashes with no common prefix?
3. **Write.** A quadtree with `Insert(p)` and `Nearest(p)`, and a differential test against brute force.

## Topic 13B — Map data and the road graph

## Unit 13.3 — OpenStreetMap data

**One-line summary:** OpenStreetMap (OSM) models the world with **nodes** (points with an ID and coordinates), **ways** (ordered lists of node IDs: roads, rivers, building outlines), and **relations** (groups such as turn restrictions or bus routes), each with free-form **tags** (`highway=primary`, `oneway=yes`, `maxspeed=50`). Data comes as XML (`.osm`) or the compact binary PBF format.

### The data model in practice

```xml
<node id="123" lat="18.5204" lon="73.8567"/>
<way id="456">
  <nd ref="123"/><nd ref="124"/><nd ref="125"/>
  <tag k="highway" v="primary"/>
  <tag k="name" v="FC Road"/>
  <tag k="oneway" v="yes"/>
  <tag k="maxspeed" v="40"/>
</way>
<relation id="789">
  <member type="way" ref="456" role="from"/>
  <member type="node" ref="125" role="via"/>
  <member type="way" ref="457" role="to"/>
  <tag k="type" v="restriction"/>
  <tag k="restriction" v="no_left_turn"/>
</relation>
```

Tags are conventions, not a schema, so code must be defensive: `maxspeed` may be `50`, `50 km/h`, `30 mph`, `IN:urban`, or missing; `oneway` may be `yes`, `true`, `1`, `-1` (one way *against* the drawing direction), or `reversible`; `highway` values decide whether cars may use a way at all (`footway`, `cycleway`, and `path` are not for cars).

### Reading OSM XML in constant memory

A city extract is hundreds of megabytes of XML, a country many gigabytes — never build a DOM. Stream tokens with `encoding/xml`'s `Decoder.Token` (Unit 5.4's streaming idea for XML) and keep only what routing needs:

```go
// Read streams OSM XML with an xml.Decoder token by token, so memory holds
// only the extracted data, not the whole document tree.
func Read(r io.Reader) (*Data, error) {
	d := &Data{Nodes: map[int64]Node{}}
	dec := xml.NewDecoder(r)
	var cur *Way
	for {
		tok, err := dec.Token()
		if err == io.EOF {
			break
		}
		if err != nil {
			return nil, err
		}
		switch t := tok.(type) {
		case xml.StartElement:
			switch t.Name.Local {
			case "node":
				n, err := parseNode(t.Attr)
				if err != nil {
					return nil, err
				}
				d.Nodes[n.ID] = n
			case "way":
				id, err := strconv.ParseInt(attr(t.Attr, "id"), 10, 64)
				if err != nil {
					return nil, fmt.Errorf("way id: %w", err)
				}
				cur = &Way{ID: id}
			case "nd":
				if cur != nil {
					ref, err := strconv.ParseInt(attr(t.Attr, "ref"), 10, 64)
					if err != nil {
						return nil, fmt.Errorf("way %d: nd ref: %w", cur.ID, err)
					}
					cur.Nodes = append(cur.Nodes, ref)
				}
			case "tag":
				if cur != nil {
					applyTag(cur, attr(t.Attr, "k"), attr(t.Attr, "v"))
				}
			}
		case xml.EndElement:
			if t.Name.Local == "way" && cur != nil {
				if carRoads[cur.Highway] && len(cur.Nodes) >= 2 {
					d.Ways = append(d.Ways, *cur)
				}
				cur = nil
			}
		}
	}
	return d, nil
}
```

Real numbers for the synthetic city: a **28.9 MB** OSM file with 90,000 nodes and 171,430 ways parses in **865 ms**. `encoding/xml` is not fast; production importers read **PBF** (protocol-buffer blocks, zlib-compressed, decodable in parallel — `github.com/paulmach/osm` and `github.com/qedus/osmpbf`) and keep all nodes in compact arrays, because a country has hundreds of millions of nodes (a `map[int64]Node` would need tens of gigabytes). A real pipeline does two passes: first collect the node IDs used by roads, then read only those nodes' coordinates.

### Deep & Tricky

**DT-288. Most OSM nodes are not intersections.** *(Data fact)* Ways contain many shape points to follow curves; only nodes shared by two or more ways (or way ends) are intersections where a route can turn. Keeping every shape point as a graph node makes the graph several times larger than necessary. Production builders contract degree-2 chains into single edges and keep the shape points separately for drawing the route (Unit 13.9).

### Mastery check

1. **Explain.** Why stream OSM XML token by token instead of decoding it into structs with `xml.Unmarshal`?
2. **Write.** Extend `applyTag` to handle `maxspeed` values like `IN:urban` with a lookup table, and `oneway=reversible` (treat as not routable), with a table test of tag combinations.

## Unit 13.4 — Building a road graph

**One-line summary:** A road network becomes a **directed, weighted graph**: nodes are intersections (and shape points), edges are road segments in each permitted direction, and weights are **travel times** from length and speed. Stored in **CSR** form — flat arrays of numbers — the graph is compact, cache-friendly, and nearly invisible to the garbage collector.

### From ways to edges

For each consecutive pair of nodes in a road way: compute the segment's length (haversine), its speed (`maxspeed`, or a default by road type), and its travel time; add an edge forwards unless the road is one-way backwards, and backwards unless it is one-way forwards. Then renumber the used OSM nodes 0…n−1, so every later structure is a slice indexed by node number, not a map (DT-219, Unit 6.13).

### Compressed sparse row (CSR)

Sort edges by source node; then the outgoing edges of node v are the positions `First[v]` to `First[v+1]` of the parallel arrays `To` and `Cost`:

```go
// Graph: the outgoing edges of node v are To[First[v]:First[v+1]], with
// travel times in milliseconds in Cost at the same positions.
type Graph struct {
	Lat, Lon []float64 // per node
	First    []int32   // len = nodes+1
	To       []int32   // per edge
	Cost     []uint32  // per edge, milliseconds
	MaxSpeed float64   // fastest speed in the graph, m/s (for A*)
	OSMID    []int64   // per node, for debugging
}
```

Iterating a node's neighbours is a sequential walk through two arrays. The synthetic city's 90,000 nodes and 189,370 edges take **3.3 MB** as CSR arrays (the whole live heap after loading was 4.6 MB). The same graph as `map[int64]*Node` with `[]*Edge` slices would take ten or more times as much memory and be scanned by the GC on every cycle — while CSR's pointer-free slices are not scanned at all (DT-269).

Design choices worth noticing:

- `int32` node IDs and `uint32` millisecond costs: half the memory of `int`/`uint64`, more values per cache line, and enough range (2 billion nodes; 49 days per edge).
- **Integer** costs (milliseconds, rounded **up**): exact arithmetic, no floating-point comparison trouble (Unit 1.5) in the priority queue.
- A **reverse graph** (every edge flipped) for searching backwards from the target (bidirectional search), built by counting incoming edges, prefix-summing, and filling — the same CSR construction again.

### Real-world complications

- **Turn restrictions** (OSM relations: no left turn from way A to way B via node N) and **turn costs** (U-turns, left turns across traffic) cannot be expressed on a node-based graph. Production routers use an **edge-based graph**, whose nodes are the original graph's edges, so a turn becomes an edge with its own cost — at the price of a graph about three times larger.
- **Speeds vary by time** (traffic): time-dependent edge costs, or live traffic overlays applied to a profile, with algorithms adapted accordingly.
- **Profiles:** car, bicycle, and pedestrian graphs differ in accessible roads, directions, and speeds — usually separate graphs.

### Deep & Tricky

**DT-289. Round edge times up, never down.** *(Algorithmic subtlety)* A* needs a heuristic that never overestimates the true remaining cost. If edge costs were rounded *down*, the real cost of a path could be lower than its straight-line time at maximum speed rounded normally — breaking admissibility by a millisecond here and there and occasionally returning non-optimal routes that differential tests catch only rarely. Rounding edges up and the heuristic down keeps the inequality safe.

### Mastery check

1. **Explain.** How do you find the outgoing edges of node 7 in CSR? Its incoming edges?
2. **Write.** `Stats(g)` that reports nodes, edges, average and maximum out-degree, and the number of nodes with no outgoing edges (dead ends) — then explain where the dead ends in the synthetic city come from.

## Unit 13.5 — Snapping a point to the nearest road

**One-line summary:** Users and GPS devices give coordinates that are not on the road graph. **Snapping** finds the nearest road position — at its simplest, the nearest graph node (with a spatial index, Unit 13.2); properly, the nearest point on the nearest *edge*, inserting a virtual node there. Reject points that are too far from any road.

### Node snapping

The project's `Index.Nearest(p)` returns the nearest node and its distance. The routing service rejects requests where either point is more than 500 m from any node (`422 Unprocessable Content`: the request is well-formed but cannot be served) — a user in the middle of a lake should get an error, not a route from the nearest shore 20 km away.

### Edge snapping (production)

On long roads, the nearest *node* can be far from the nearest *road*: a point next to the middle of a 1 km segment snaps to one of its ends, 500 m away. Proper snapping projects the point onto each nearby segment (vector projection clamped to the segment, computed in a local planar approximation), picks the closest, and inserts a temporary node splitting that edge, with partial costs for both halves. Spatial indexes over segments (R-trees, or a grid storing each segment in every cell it touches) find candidates.

### Snapping GPS traces (map matching)

A sequence of noisy GPS points (a recorded trip) is matched to the most likely road path with a **hidden Markov model**: candidate roads per point are the hidden states, distance from point to road gives emission probabilities, and the plausibility of driving between candidates (route distance vs straight-line distance) gives transition probabilities; the **Viterbi** algorithm (dynamic programming, Unit 6.12) finds the best sequence. This is how ride-sharing apps compute fares and how traffic speeds are inferred.

### Mastery check

1. **Explain.** Why is snapping to the nearest node inadequate on long segments? Draw an example.
2. **Write.** `NearestOnSegment(p, a, b geo.Point) (geo.Point, float64)` using a local equirectangular projection; test with points before, beside, and after a segment.

## Topic 13C — Routing algorithms

## Unit 13.6 — Faster searches

**One-line summary:** Dijkstra (Unit 6.11) explores outward in all directions. **A\*** adds an admissible estimate of the remaining cost (straight-line distance at the maximum speed), steering the search towards the target. **Bidirectional Dijkstra** searches from both ends and stops when the frontiers meet. Measured on the synthetic city: A* settles 3.3× fewer nodes than Dijkstra on random routes.

### The router

Per-query arrays (`dist`, `prev`, `settled`) of size n would cost O(n) to reset before every query — 90,000 writes for a route that might settle 1,000 nodes. The router instead **stamps** each entry with a query number and treats entries with an old stamp as unset: an O(1) reset (DT-290). A `Router` therefore holds mutable scratch space and is not safe for concurrent use; the routing service keeps a `sync.Pool` of them (Unit 7.11).

```go
// Router owns per-query scratch space. Searches reuse it without clearing
// it, by stamping entries with a query number (O(1) reset instead of O(n)).
// A Router is not safe for concurrent use; create one per goroutine.
type Router struct {
	g, rev  *graph.Graph
	dist    [2][]uint64
	prev    [2][]int32
	stamp   [2][]uint32
	settled [2][]uint32
	query   uint32
	heaps   [2]heap
}

func (r *Router) d(side int, v int32) uint64 {
	if r.stamp[side][v] != r.query {
		return inf
	}
	return r.dist[side][v]
}
```

### Dijkstra and A* in one function

A* is Dijkstra with the priority `dist[v] + h(v)` instead of `dist[v]`, where `h` is a **lower bound** on the remaining cost. With `h ≡ 0` it *is* Dijkstra:

```go
// Dijkstra is the baseline: expand nodes in order of travel time.
func (r *Router) Dijkstra(src, dst int32) (Result, error) {
	return r.astar(src, dst, func(int32) uint64 { return 0 })
}

// AStar adds a lower bound on the remaining time: straight-line distance
// at the fastest speed in the graph. It never overestimates (admissible),
// so the result is still optimal, but far fewer nodes are settled.
func (r *Router) AStar(src, dst int32) (Result, error) {
	target := r.g.Point(dst)
	maxSpeed := r.g.MaxSpeed
	return r.astar(src, dst, func(v int32) uint64 {
		return uint64(geo.Haversine(r.g.Point(v), target) / maxSpeed * 1000) // ms, rounded down
	})
}

func (r *Router) astar(src, dst int32, h func(int32) uint64) (Result, error) {
	r.begin()
	g := r.g
	r.set(0, src, 0, -1)
	r.heaps[0].push(h(src), src)
	settled := 0
	for r.heaps[0].len() > 0 {
		_, v := r.heaps[0].pop()
		if r.settled[0][v] == r.query {
			continue // stale heap entry
		}
		r.settled[0][v] = r.query
		settled++
		if v == dst {
			return Result{Nodes: r.path(src, dst), DurationMs: r.d(0, dst), Settled: settled}, nil
		}
		dv := r.d(0, v)
		for e := g.First[v]; e < g.First[v+1]; e++ {
			w := g.To[e]
			nd := dv + uint64(g.Cost[e])
			if nd < r.d(0, w) {
				r.set(0, w, nd, v)
				r.heaps[0].push(nd+h(w), w)
			}
		}
	}
	return Result{Settled: settled}, ErrNoRoute
}
```

Why A* stays optimal: if `h` never overestimates (no road is faster than the fastest road in a straight line), the target cannot be popped before every cheaper path has been considered. The cost: one haversine (trigonometry) per pushed node.

### Bidirectional Dijkstra

```go
// Bidirectional runs Dijkstra forwards from src and backwards (on the
// reversed graph) from dst, alternating, and stops when the two searches
// can no longer improve the best meeting point.
func (r *Router) Bidirectional(src, dst int32) (Result, error) {
	r.begin()
	gs := [2]*graph.Graph{r.g, r.rev}
	r.set(0, src, 0, -1)
	r.set(1, dst, 0, -1)
	r.heaps[0].push(0, src)
	r.heaps[1].push(0, dst)
	best, meet := uint64(inf), int32(-1)
	settled := 0
	for r.heaps[0].len() > 0 || r.heaps[1].len() > 0 {
		// Stop when the smallest keys on both sides sum to at least best.
		if best != inf && r.heaps[0].minKey()+r.heaps[1].minKey() >= best {
			break
		}
		side := 0
		if r.heaps[1].len() > 0 && (r.heaps[0].len() == 0 || r.heaps[1].minKey() < r.heaps[0].minKey()) {
			side = 1
		}
		_, v := r.heaps[side].pop()
		if r.settled[side][v] == r.query {
			continue
		}
		r.settled[side][v] = r.query
		settled++
		dv := r.d(side, v)
		g := gs[side]
		for e := g.First[v]; e < g.First[v+1]; e++ {
			w := g.To[e]
			nd := dv + uint64(g.Cost[e])
			if nd < r.d(side, w) {
				r.set(side, w, nd, v)
				r.heaps[side].push(nd, w)
			}
			if other := r.d(1-side, w); other != inf && nd+other < best {
				best, meet = nd+other, w
			}
		}
	}
	...
}
```

Two searches of radius r/2 explore about half the area of one search of radius r (area grows with the square of the radius in a planar network). The **stopping rule** is the classic trap: stopping at the first node settled by both sides can return a suboptimal route; the correct rule is to stop when `minKey(forward) + minKey(backward) ≥ best` (DT-291).

### Measured

On the synthetic city (90,000 nodes, 189,370 edges), 200 random routable pairs, Go 1.24 on a 2-vCPU x86 server:

| Algorithm | Time per query | Nodes settled per query |
| --- | --- | --- |
| Dijkstra | 4.56 ms | 44,834 |
| A* (haversine heuristic) | 2.75 ms | 13,772 |
| Bidirectional Dijkstra | 3.34 ms | 30,713 |

A* settles **3.3×** fewer nodes but is only **1.7×** faster: each pushed node pays for a haversine with five trigonometric functions. (An equirectangular heuristic, or precomputing each node's projected x/y once, makes it cheaper — Extension 1.) Bidirectional Dijkstra helps less here than on real road networks, because a dense grid's one-way streets make the backward search's frontier irregular. Every algorithm returned **exactly the same travel time** on 300 random pairs in the correctness test — the property that matters most.

### Deep & Tricky

**DT-290. Resetting O(n) arrays per query can cost more than the query.** *(Performance)* A short route in a large graph settles a few hundred nodes; clearing `dist` arrays of 10 million entries costs milliseconds per query. Generation stamps (store the query number with each entry) or a list of touched entries to reset afterwards make the reset proportional to the work actually done.

**DT-291. Bidirectional search must not stop at the first meeting node.** *(Algorithmic subtlety)* The first node settled by both searches is not necessarily on the shortest path. Keep the best `dist_f(u) + cost(u,w) + dist_b(w)` seen while relaxing edges, and stop only when the two queues' minimum keys together cannot beat it. The project's differential test against plain Dijkstra on 300 pairs is what gives confidence in this code.

### Mastery check

1. **Explain.** Why does A* remain optimal with an admissible heuristic, and what goes wrong with an inadmissible one?
2. **Explain.** Why did A* settle 3.3× fewer nodes but run only 1.7× faster?
3. **Write.** Bidirectional A* is subtle (the two heuristics must be made consistent); research the "average potential" method and implement it, with the differential test as your safety net.

## Unit 13.7 — Preprocessing for speed

**One-line summary:** For continental graphs (tens of millions of nodes), even A* settles too many nodes per query. **Preprocessing** spends minutes or hours once to make each query take microseconds to milliseconds: **ALT** precomputes distances to a few landmarks for a much better A* heuristic; **contraction hierarchies (CH)** add shortcut edges so a bidirectional search only ever goes "upwards" through a node ordering and settles a few hundred nodes.

### ALT: A*, Landmarks, Triangle inequality

Pick a few (8–16) **landmarks**, usually on the edge of the map. Precompute the travel time from every node to every landmark and from every landmark to every node (two Dijkstra runs per landmark on the full graph). By the triangle inequality, for any node v, target t, and landmark L:

```
dist(v, t) ≥ dist(L, t) − dist(L, v)
dist(v, t) ≥ dist(v, L) − dist(t, L)
```

The maximum of these bounds over all landmarks is an admissible heuristic that is far tighter than straight-line distance, because it knows about the actual road network (rivers, missing bridges, one-way systems). Memory: 2 × landmarks × nodes distances (for 10 million nodes and 16 landmarks, about 1.3 GB as `uint32`). Typical speed-up over Dijkstra: 10–50×. ALT also adapts easily to changing weights (only landmark distances need recomputing).

### Contraction hierarchies

1. **Order** the nodes by importance (heuristics: edge difference, how many shortcuts contracting it would add, its depth; small residential nodes first, motorway junctions last).
2. **Contract** nodes in that order: remove node v, and for every pair of neighbours u → v → w whose shortest path goes through v, add a **shortcut** edge u → w with the combined cost (a small local "witness search" checks whether another path is as short; if so, no shortcut is needed).
3. **Query:** a bidirectional Dijkstra in which each side only relaxes edges leading to *higher-ranked* nodes. Both searches climb towards the important roads and meet near the top of the hierarchy. On continental road networks, a query settles **a few hundred nodes** and takes well under a millisecond — thousands of times faster than Dijkstra.
4. **Unpack** shortcuts recursively (each remembers the node it bypassed) to recover the full path.

CH is the basis of OSRM, GraphHopper, and many production routers. Its weakness is changing weights: live traffic requires re-contraction, which led to **customisable** variants (CCH, CRP/multi-level Dijkstra) that separate a slow metric-independent preprocessing from a fast per-metric customisation.

### Choosing

| Approach | Preprocessing | Memory | Query (country-scale) | Weight changes |
| --- | --- | --- | --- | --- |
| Dijkstra / A* | none | graph only | 100s of ms – seconds | instant |
| ALT | minutes | + landmark tables | 10s of ms | recompute landmarks |
| Contraction hierarchies | minutes – hours | + shortcuts (~graph size) | < 1 ms | re-contract (or CCH) |

For a city (the capstone's scale), A* with a good heuristic is often enough; CH is the natural next step and the capstone's stretch goal.

### Mastery check

1. **Explain.** Use the triangle inequality to explain why landmark bounds never overestimate.
2. **Explain.** Why does a CH query only need to go "upwards"? What guarantees that the shortest path is found?
3. **Write.** Implement ALT with 8 landmarks chosen by the "farthest" heuristic (each new landmark is the node farthest from those already chosen). Report settled nodes vs A* on the synthetic city, with the differential test.

## Topic 13D — Engineering the graph

## Unit 13.8 — Compact graph layouts

**One-line summary:** At country scale, memory layout decides whether routing fits in RAM and how fast it runs. Use flat arrays of small integers (CSR), number nodes so that nearby nodes have nearby IDs (locality), avoid pointers, and store the preprocessed graph in a file that can be **memory-mapped** for instant startup.

### Techniques

- **Small types:** `int32` IDs, `uint32` costs, coordinates as `int32` fixed-point (10⁻⁷ degrees) instead of `float64` — the 90,000-node graph's coordinates would shrink from 1.44 MB to 0.72 MB.
- **Struct of arrays** (Unit 6.13): the search loop touches only `First`, `To`, and `Cost`; coordinates are in separate arrays, used only by A* and for output.
- **Node ordering for locality:** renumber nodes along a space-filling curve (Hilbert or Z-order, the idea behind geohash and S2) or in BFS/DFS order, so a node's neighbours tend to be stored nearby. The search then reads memory in mostly sequential runs — the 13× row/column-major lesson of Unit 6.13, applied to graphs. Real routers report 20–50% faster queries from reordering alone.
- **No pointers:** the GC never scans the graph (DT-269); a 2 GB graph costs the GC nothing per cycle.

### Memory-mapped graph files

Building a large graph takes minutes; loading it should not. Write the arrays to a file in a fixed binary layout (a header with counts and offsets, then each array, aligned — Unit 5.5's `encoding/binary` ideas), then at startup **memory-map** the file (`syscall.Mmap` on Unix, `golang.org/x/exp/mmap` for read-only access) and view the mapped bytes as typed slices with `unsafe.Slice` (Unit 10.10). Startup becomes instant (pages are loaded lazily as queries touch them), several processes share one copy of the graph in the OS page cache, and a new graph is deployed by writing a new file and switching to it.

The risks are exactly those of Unit 10.10: endianness and alignment must match the file format, and a truncated or corrupted file must be detected (checksums, a magic number and version in the header) *before* any slice is created over it.

### Mastery check

1. **Explain.** Why does renumbering nodes along a space-filling curve speed up Dijkstra without changing a single algorithmic step?
2. **Write.** `WriteGraph(w io.Writer, g *Graph)` and `ReadGraph(r io.Reader) (*Graph, error)` with a header (magic, version, counts, CRC32), little-endian arrays, and tests including a truncated file. (Use plain reading first; memory-mapping is the extension.)

## Unit 13.9 — Routes for users

**One-line summary:** A route is useful only when presented well: the sequence of nodes becomes **geometry** (coordinates, including shape points), encoded compactly as a **polyline**; plus total **distance** and **travel time**, turn-by-turn **instructions**, and sensible handling of the start and end (snapping distances).

### Geometry and polylines

The node path converts to coordinates (`route.Geometry`). Sending thousands of coordinate pairs as JSON numbers is wasteful; **Google's encoded polyline format** stores each coordinate as the difference from the previous one, scaled to integers (1e-5 degrees), zig-zag encoded, in 5-bit chunks mapped to printable characters:

```go
// EncodePolyline encodes points in Google's polyline format (precision 1e5),
// the compact string format most map clients accept.
func EncodePolyline(pts []Point) string {
	var sb strings.Builder
	var pLat, pLon int64
	for _, p := range pts {
		lat := int64(math.Round(p.Lat * 1e5))
		lon := int64(math.Round(p.Lon * 1e5))
		encodeSigned(&sb, lat-pLat)
		encodeSigned(&sb, lon-pLon)
		pLat, pLon = lat, lon
	}
	return sb.String()
}

func encodeSigned(sb *strings.Builder, v int64) {
	u := uint64(v << 1)
	if v < 0 {
		u = ^u // zig-zag: small negative numbers become small positive ones
	}
	for u >= 0x20 {
		sb.WriteByte(byte((0x20 | (u & 0x1f)) + 63))
		u >>= 5
	}
	sb.WriteByte(byte(u + 63))
}
```

The test reproduces Google's documented example exactly (`_p~iF~ps|U_ulLnnqC_mqNvxq\`@` for three points). Notice the techniques from earlier chapters: delta encoding, two's complement and bit manipulation (Units 0.4, 1.4), variable-length integers (like `binary.AppendUvarint`, Unit 5.5).

### Distance, time, instructions

- **Travel time** is the route's cost (already computed); **distance** sums segment lengths (`route.LengthM`).
- **Turn-by-turn instructions** come from the angles between consecutive segments (bearing changes, Unit 13.1's exercise), road names from the OSM ways (keep a way index per edge), and road-class changes ("merge onto the motorway"). Most of the work is presentation logic and localisation.
- **ETAs** should add realistic penalties the graph lacks (turns, traffic lights), or be calibrated against historical trip data.

### Deep & Tricky

**DT-292. Rounding in polyline encoding accumulates if you encode absolute values carelessly.** *(Format subtlety)* The format encodes *differences* of rounded integer coordinates. Computing differences of the unrounded floats and rounding each difference instead lets rounding errors accumulate along the line, drifting long routes off the road. Round each absolute coordinate first, then take integer differences — as the code above does.

### Mastery check

1. **Explain.** Why does the polyline format use differences and zig-zag encoding?
2. **Write.** `DecodePolyline(s string) ([]Point, error)` and a property-based round-trip test (encode random points rounded to 1e-5, decode, compare exactly).

## Chapter 13 Project — Point-to-point routing on a city's OSM data

**Goal:** build a working router from an OSM extract: parse the data, build a CSR road graph with one-way streets and speeds, snap coordinates to the network, and answer fastest-route queries with Dijkstra, A*, and bidirectional Dijkstra — proving they agree, and measuring how much work each does.

### Requirements

1. `geo`: haversine, equirectangular distance, geohash, polyline encoding — each tested against published reference values.
2. `osm`: streaming XML reader keeping car-accessible roads, `oneway` (including `-1`), `maxspeed` (including mph), names.
3. `graph`: CSR graph with `int32` node IDs and `uint32` millisecond costs rounded up; a reverse graph; a grid spatial index with an exact nearest-node search (tested against brute force).
4. `route`: Dijkstra, A*, and bidirectional Dijkstra sharing one `Router` with O(1) per-query reset; a specialised heap; results with path, duration, and settled-node count.
5. Tests: all three algorithms return identical durations on at least 300 random pairs, and the returned path's edge costs sum to the reported duration.
6. A benchmark comparing time and settled nodes per algorithm on a large city.
7. `cmd/gencity` to generate test cities (or use a real extract: convert PBF to XML with `osmium cat city.osm.pbf -o city.osm`).

### Reference solution

All the code for `geo`, `osm`, `graph`, and `route` appears in the units above: `Haversine`, `Equirectangular`, `Geohash`, and `EncodePolyline` (Units 13.1, 13.2, 13.9); `osm.Read` (Unit 13.3) with this tag handling:

```go
func applyTag(w *Way, k, v string) {
	switch k {
	case "highway":
		w.Highway = v
	case "name":
		w.Name = v
	case "oneway":
		switch v {
		case "yes", "true", "1":
			w.OneWay = 1
		case "-1", "reverse":
			w.OneWay = -1
		}
	case "maxspeed":
		// "50", "50 km/h", or "30 mph"
		f := strings.Fields(v)
		if len(f) > 0 {
			if n, err := strconv.Atoi(f[0]); err == nil {
				if len(f) > 1 && f[1] == "mph" {
					n = n * 1609 / 1000
				}
				w.MaxSpeed = n
			}
		}
	}
	if w.Highway == "motorway" && w.OneWay == 0 {
		w.OneWay = 1 // motorways are one-way by convention unless tagged otherwise
	}
}
```

…the graph builder:

```go
// Build turns OSM ways into a directed graph. Only nodes used by roads are
// kept, renumbered 0..n-1 (slices instead of maps from here on, Unit 6.13).
func Build(d *osm.Data) (*Graph, error) {
	idx := map[int64]int32{}
	g := &Graph{}
	node := func(id int64) (int32, error) {
		if i, ok := idx[id]; ok {
			return i, nil
		}
		n, ok := d.Nodes[id]
		if !ok {
			return 0, fmt.Errorf("way refers to missing node %d", id)
		}
		i := int32(len(g.Lat))
		idx[id] = i
		g.Lat = append(g.Lat, n.Lat)
		g.Lon = append(g.Lon, n.Lon)
		g.OSMID = append(g.OSMID, id)
		return i, nil
	}
	var edges []edge
	for _, w := range d.Ways {
		kmh := w.MaxSpeed
		if kmh == 0 {
			kmh = defaultSpeed[w.Highway]
		}
		if kmh == 0 {
			kmh = 30
		}
		mps := float64(kmh) / 3.6
		g.MaxSpeed = max(g.MaxSpeed, mps)
		for i := 0; i+1 < len(w.Nodes); i++ {
			a, err := node(w.Nodes[i])
			if err != nil {
				return nil, err
			}
			b, err := node(w.Nodes[i+1])
			if err != nil {
				return nil, err
			}
			ms := uint32(math.Ceil(geo.Haversine(g.Point(a), g.Point(b)) / mps * 1000))
			if w.OneWay >= 0 {
				edges = append(edges, edge{a, b, ms})
			}
			if w.OneWay <= 0 {
				edges = append(edges, edge{b, a, ms})
			}
		}
	}
	slices.SortFunc(edges, func(x, y edge) int { return int(x.from) - int(y.from) })
	n := len(g.Lat)
	g.First = make([]int32, n+1)
	g.To = make([]int32, len(edges))
	g.Cost = make([]uint32, len(edges))
	for i, e := range edges {
		g.First[e.from+1]++
		g.To[i], g.Cost[i] = e.to, e.cost
	}
	for v := range n {
		g.First[v+1] += g.First[v]
	}
	return g, nil
}
```

…the reverse graph:

```go
// Reverse returns the graph with every edge flipped (for searching backwards
// from the target, as bidirectional Dijkstra does).
func (g *Graph) Reverse() *Graph {
	n := g.NumNodes()
	r := &Graph{Lat: g.Lat, Lon: g.Lon, OSMID: g.OSMID, MaxSpeed: g.MaxSpeed,
		First: make([]int32, n+1), To: make([]int32, len(g.To)), Cost: make([]uint32, len(g.Cost))}
	for v := range n {
		for e := g.First[v]; e < g.First[v+1]; e++ {
			r.First[g.To[e]+1]++
		}
	}
	for v := range n {
		r.First[v+1] += r.First[v]
	}
	next := slices.Clone(r.First[:n])
	for v := range n {
		for e := g.First[v]; e < g.First[v+1]; e++ {
			w := g.To[e]
			r.To[next[w]], r.Cost[next[w]] = int32(v), g.Cost[e]
			next[w]++
		}
	}
	return r
}
```

…and the specialised heap used by the router:

```go
// heap is a binary min-heap of (key, node) specialised for speed: no
// interfaces, no closures, no generics in the hot loop (Unit 10.8).
type heap struct {
	keys  []uint64
	nodes []int32
}

func (h *heap) push(k uint64, v int32) {
	h.keys = append(h.keys, k)
	h.nodes = append(h.nodes, v)
	i := len(h.keys) - 1
	for i > 0 {
		p := (i - 1) / 2
		if h.keys[p] <= h.keys[i] {
			break
		}
		h.keys[p], h.keys[i] = h.keys[i], h.keys[p]
		h.nodes[p], h.nodes[i] = h.nodes[i], h.nodes[p]
		i = p
	}
}

func (h *heap) pop() (uint64, int32) {
	k, v := h.keys[0], h.nodes[0]
	n := len(h.keys) - 1
	h.keys[0], h.nodes[0] = h.keys[n], h.nodes[n]
	h.keys, h.nodes = h.keys[:n], h.nodes[:n]
	i := 0
	for {
		l, s := 2*i+1, i
		if l < n && h.keys[l] < h.keys[s] {
			s = l
		}
		if l+1 < n && h.keys[l+1] < h.keys[s] {
			s = l + 1
		}
		if s == i {
			return k, v
		}
		h.keys[s], h.keys[i] = h.keys[i], h.keys[s]
		h.nodes[s], h.nodes[i] = h.nodes[i], h.nodes[s]
		i = s
	}
}
```

(Keys and nodes in two parallel slices rather than a slice of structs: a struct-of-arrays layout, Unit 6.13 — the comparisons in `pop` touch only the keys.)

### The decisive test

```go
// All three algorithms must agree on the optimal travel time.
func TestAlgorithmsAgree(t *testing.T) {
	g, rev := loadCity(t, 60)
	r := New(g, rev)
	rnd := rand.New(rand.NewPCG(1, 2))
	n := int32(g.NumNodes())
	agreed, noRoute := 0, 0
	for range 300 {
		s, d := rnd.Int32N(n), rnd.Int32N(n)
		a, errA := r.Dijkstra(s, d)
		b, errB := r.AStar(s, d)
		c, errC := r.Bidirectional(s, d)
		if (errA == nil) != (errB == nil) || (errA == nil) != (errC == nil) {
			t.Fatalf("%d->%d: errors differ: %v %v %v", s, d, errA, errB, errC)
		}
		if errA != nil {
			noRoute++
			continue
		}
		if a.DurationMs != b.DurationMs || a.DurationMs != c.DurationMs {
			t.Fatalf("%d->%d: durations differ: dijkstra %d, A* %d, bidirectional %d", s, d, a.DurationMs, b.DurationMs, c.DurationMs)
		}
		// the returned path must really have that duration
		if got := pathCost(g, c.Nodes); got != c.DurationMs {
			t.Fatalf("%d->%d: bidirectional path costs %d, reported %d", s, d, got, c.DurationMs)
		}
		agreed++
	}
	t.Logf("%d routes agreed, %d pairs unreachable", agreed, noRoute)
}
```

Result: `300 routes agreed, 0 pairs unreachable`. A plain Dijkstra is easy to get right; the faster algorithms are where subtle bugs hide (DT-289, DT-291) — so the simple one becomes the **oracle** for the clever ones. This differential-testing technique is how production routing engines are validated.

### Real measurements (summary)

- 28.9 MB OSM XML → 90,000 nodes, 189,370 edges, parsed in 865 ms, built in 68 ms, **3.3 MB** of graph arrays.
- Per query on random pairs: Dijkstra 4.56 ms / 44,834 settled; A* 2.75 ms / 13,772; bidirectional 3.34 ms / 30,713.

### Review checklist

- Do all algorithms agree on hundreds of random pairs, including unreachable ones?
- Are edge costs rounded up and the heuristic rounded down (DT-289)?
- Is per-query reset O(work), not O(n) (DT-290)?
- Does the nearest-node search match brute force?
- Does a `oneway=-1` way produce edges only in the reverse direction? (Add a unit test with a hand-written three-node OSM snippet.)

### Extensions

1. Make A*'s heuristic cheaper: precompute each node's local planar x, y in metres once, and use Euclidean distance on them. Measure the speed-up.
2. Implement ALT (Unit 13.7's exercise) and contraction hierarchies, validating both with the differential test.
3. Download a real city extract (Geofabrik provides country and region files; BBBike provides city cuts), convert it, and report the same measurements. Compare graph size, settled nodes, and query times with the synthetic city.

## Answers to the Chapter 13 prediction questions

**13.1 Q2.** 0.001° of latitude ≈ 111 m. 0.001° of longitude at 18.5° N ≈ 111 m × cos(18.5°) ≈ 105 m.

# Chapter 14 — Capstone: Production Routing Backend

*Goal: independently design, build, and operate a serious routing service.*

This chapter is different. Earlier chapters taught; this one asks you to **lead**. You will write a design document, build a routing service from OpenStreetMap data to a production deployment, prove its performance with load tests and profiles, make it observable and safe to operate, and defend it in a senior-level architecture review. Each unit gives you the requirements, the decisions you must make (with the trade-offs you now know how to weigh), a working starting point where it helps, and the acceptance criteria.

A complete, tested starting point exists: the Chapter 13 packages plus the `server` package and `routed` command in this chapter, which were built, tested (including hot reload under concurrent load), and load-tested while writing this book. Your capstone should go well beyond it.

**Capstone deliverables**

1. A design document (Unit 14.1) reviewed before you write most of the code.
2. A repository with: an ingestion command, a graph file format, the routing service, tests (unit, differential, integration, load), Dockerfile, CI, deployment manifests, dashboards and alerts as code.
3. A performance report (Unit 14.6) in the Chapter 10 format.
4. An operations runbook (Unit 14.7).
5. A recorded or written architecture review (Unit 14.8).

## Topic 14A — Design

## Unit 14.1 — Requirements and design document with trade-off decisions

**One-line summary:** Before building, write down what the system must do, how well, under what constraints, and how it will fail — and get it reviewed. A design document is a thinking tool first and a communication tool second.

### A design document template

1. **Context and goals** — the problem, who has it, what success looks like.
2. **Non-goals** — what you deliberately will not do (turn-by-turn voice guidance, public transport, live traffic in version 1).
3. **Requirements** — functional (route between two coordinates for cars; alternative routes later; a distance/time matrix later) and non-functional with numbers (Unit 12.10):
   - Coverage: one metropolitan region (e.g. Pune: ~1–2 million graph nodes after contraction of shape points).
   - Latency: p99 < 50 ms per route at the target load, measured at the server.
   - Throughput: 500 routes/s sustained per instance; horizontal scaling to 5,000/s.
   - Availability: 99.9%; data updates weekly without downtime.
   - Correctness: routes optimal for the cost model (verified by differential tests); snapping within 500 m or a clear error.
4. **Estimates** — graph size in memory (bytes per node and edge × counts, from Unit 13.4's 3.3 MB for 90,000 nodes and 189,370 edges, about 37 bytes per node-with-edges), preprocessing time, request bandwidth (polyline sizes), instance counts.
5. **API** — endpoints, parameters, responses, errors, versioning (Unit 8.6).
6. **Architecture** — components and data flow (ingestion → graph file → service instances behind a load balancer); diagrams.
7. **Key decisions with alternatives considered** — the heart of the document. For each: options, trade-offs, decision, and what would make you revisit it.
8. **Failure modes** — for every component and dependency.
9. **Observability, security, rollout, and testing plans.**
10. **Open questions and risks.**

### Decisions you must make and justify

| Decision | Options | Trade-offs to discuss |
| --- | --- | --- |
| Routing algorithm | A* with planar heuristic; ALT; contraction hierarchies | query speed vs preprocessing time and memory vs ease of updating weights (Unit 13.7) |
| Graph model | node-based; edge-based (turn restrictions) | correctness for real roads vs ~3× size |
| Data format | build at startup from PBF; custom binary file; memory-mapped file | startup time, deploy simplicity, `unsafe` risk (Unit 13.8) |
| Protocol | REST/JSON; gRPC | clients, tooling, streaming (Unit 8.8) |
| Scaling | replicate the full graph per instance; shard by region | simplicity vs memory; cross-region routes |
| Caching | none; cache frequent origin–destination pairs | hit rate on random queries is low — measure before adding; if added, follow P-36 |
| Updates | restart with new data; hot reload; blue-green instances | availability vs memory during the swap |

A good design document records *why* — six months later, the team needs to know whether the reason for a decision still holds.

### Acceptance criteria

The document fits in about 6–10 pages, states every requirement as a measurable number, lists at least two alternatives for each major decision, and contains a failure-mode table covering at least: bad input, oversized input, corrupt data file, out-of-memory, overload, slow clients, a bad deploy, and a dependency outage.

### Mastery check

1. **Write.** Your capstone design document, then ask someone (a colleague, a study partner) to review it with the Unit 14.8 questions before you build.

## Topic 14B — Build

## Unit 14.2 — Ingestion pipeline and storage

**One-line summary:** Turn a raw OSM extract into a validated, versioned, binary graph file in a reproducible batch job, separate from the service. The service only ever loads finished, checked files.

### The pipeline

```
city.osm.pbf ──read (2 passes, parallel PBF blocks)──▶ roads + used nodes
             ──build──▶ CSR graph (+ reverse, + spatial index data)
             ──contract shape points (DT-288), reorder nodes (Unit 13.8)
             ──preprocess (ALT tables or CH shortcuts, Unit 13.7)
             ──validate──▶ graph-2026-11-15.bin (header: magic, version, counts, source hash, CRC)
```

Requirements:

- **Reproducible:** same input file and code version → byte-identical output (sort everything; no map iteration order in outputs — P-21).
- **Validated before publishing:** connectivity (size of the largest strongly connected component; flag islands), sanity of speeds and coordinates, and a **golden set** of routes whose durations must stay within a tolerance of the previous version (a regression test for data, not just code).
- **Versioned and immutable:** files are never modified in place; the service is told which version to load.
- **Streamed and bounded:** memory proportional to the road network, not to the raw file (Units 5.1, 13.3).

### Starting point

`gencity` produces test inputs; `osm.Read` + `graph.Build` + `Reverse` build the graph. Your work: PBF input, shape-point contraction, node reordering, a binary format (Unit 13.8's exercise), validation, and a CLI (`buildgraph -in city.osm.pbf -out graph.bin`).

### Acceptance criteria

The pipeline runs in CI on a small extract, produces identical files on repeated runs (`sha256sum` equal), rejects a deliberately corrupted input with a clear error, and its golden-route check fails when you change a speed default.

## Unit 14.3 — Graph preprocessing and fast routing

**One-line summary:** Choose and implement the routing speed-up your requirements need, validated against plain Dijkstra as an oracle on thousands of random pairs.

### Requirements

- At least A* with a cheap admissible heuristic (planar coordinates precomputed, Unit 13.6 Extension 1).
- Stretch: ALT or contraction hierarchies (Unit 13.7).
- The differential test of Chapter 13 extended to 10,000 random pairs, including unreachable pairs and pairs on one-way streets, run in CI.
- Benchmarks reporting time and settled nodes per algorithm on the target region (Unit 10.1 rigour, `benchstat` comparisons for every optimisation).

### Starting point

`route.Router` with Dijkstra, A*, bidirectional Dijkstra, the stamped O(1) reset, and the specialised heap; measured on the 90,000-node synthetic city at 4.56 ms / 2.75 ms / 3.34 ms per query.

### Acceptance criteria

Zero disagreements with the oracle; the chosen algorithm meets the latency requirement for single queries with headroom (aim for p99 at least 3× below the target, because load adds queueing); every performance claim backed by benchstat output.

## Unit 14.4 — Service API

**One-line summary:** Expose routing over HTTP (and optionally gRPC) with strict input validation, snapping, clear errors, compact responses, and safe concurrency.

### The starting point: `server` package

```go
// Package server answers routing queries over HTTP and can swap in a new
// road graph without downtime.
package server

import (
	"bufio"
	"encoding/json"
	"errors"
	"fmt"
	"log/slog"
	"net/http"
	"os"
	"strconv"
	"strings"
	"sync"
	"sync/atomic"
	"time"

	"example.com/routing/geo"
	"example.com/routing/graph"
	"example.com/routing/osm"
	"example.com/routing/route"
)

// Engine is an immutable, fully built routing dataset.
type Engine struct {
	G, Rev  *graph.Graph
	Index   *graph.Index
	Loaded  time.Time
	Source  string
	routers sync.Pool // *route.Router: not concurrency-safe, so one per request
}

// Load builds an Engine from an OSM XML file.
func Load(path string) (*Engine, error) {
	f, err := os.Open(path)
	if err != nil {
		return nil, err
	}
	defer f.Close()
	d, err := osm.Read(bufio.NewReaderSize(f, 1<<20))
	if err != nil {
		return nil, fmt.Errorf("%s: %w", path, err)
	}
	g, err := graph.Build(d)
	if err != nil {
		return nil, fmt.Errorf("%s: %w", path, err)
	}
	if g.NumNodes() == 0 {
		return nil, fmt.Errorf("%s: no roads", path)
	}
	e := &Engine{G: g, Rev: g.Reverse(), Index: graph.NewIndex(g, 0.002), Loaded: time.Now(), Source: path}
	e.routers.New = func() any { return route.New(e.G, e.Rev) }
	return e, nil
}

type Server struct {
	engine  atomic.Pointer[Engine] // swapped atomically on reload (Unit 7.6)
	log     *slog.Logger
	maxSnap float64 // metres: farther from any road is an error
}

func New(e *Engine, log *slog.Logger) *Server {
	s := &Server{log: log, maxSnap: 500}
	s.engine.Store(e)
	return s
}

// Swap installs a new engine. In-flight requests finish on the old one,
// which the garbage collector frees when they are done.
func (s *Server) Swap(e *Engine) { s.engine.Store(e) }

func (s *Server) Handler() http.Handler {
	mux := http.NewServeMux()
	mux.HandleFunc("GET /route", s.route)
	mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, _ *http.Request) {
		e := s.engine.Load()
		fmt.Fprintf(w, "ok nodes=%d edges=%d loaded=%s\n", e.G.NumNodes(), e.G.NumEdges(), e.Loaded.Format(time.RFC3339))
	})
	return mux
}

type routeResponse struct {
	DurationS    float64 `json:"duration_s"`
	DistanceM    float64 `json:"distance_m"`
	Polyline     string  `json:"polyline"`
	SnapFromM    float64 `json:"snap_from_m"`
	SnapToM      float64 `json:"snap_to_m"`
	SettledNodes int     `json:"settled_nodes"`
	Algorithm    string  `json:"algorithm"`
}

func (s *Server) route(w http.ResponseWriter, r *http.Request) {
	q := r.URL.Query()
	from, err1 := parsePoint(q.Get("from"))
	to, err2 := parsePoint(q.Get("to"))
	if err := errors.Join(err1, err2); err != nil {
		httpError(w, http.StatusBadRequest, err.Error())
		return
	}
	algo := q.Get("algo")
	if algo == "" {
		algo = "astar"
	}
	e := s.engine.Load() // one consistent dataset for the whole request
	src, ds := e.Index.Nearest(from)
	dst, dd := e.Index.Nearest(to)
	if ds > s.maxSnap || dd > s.maxSnap {
		httpError(w, http.StatusUnprocessableEntity, fmt.Sprintf("point is more than %.0f m from any road", s.maxSnap))
		return
	}
	rt := e.routers.Get().(*route.Router)
	defer e.routers.Put(rt)
	var res route.Result
	var err error
	switch algo {
	case "dijkstra":
		res, err = rt.Dijkstra(src, dst)
	case "astar":
		res, err = rt.AStar(src, dst)
	case "bidirectional":
		res, err = rt.Bidirectional(src, dst)
	default:
		httpError(w, http.StatusBadRequest, "algo must be dijkstra, astar, or bidirectional")
		return
	}
	if errors.Is(err, route.ErrNoRoute) {
		httpError(w, http.StatusNotFound, "no route between these points")
		return
	}
	pts := route.Geometry(e.G, res.Nodes)
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(routeResponse{
		DurationS: float64(res.DurationMs) / 1000, DistanceM: route.LengthM(e.G, res.Nodes),
		Polyline: geo.EncodePolyline(pts), SnapFromM: ds, SnapToM: dd,
		SettledNodes: res.Settled, Algorithm: algo,
	})
}

func parsePoint(s string) (geo.Point, error) {
	a, b, ok := strings.Cut(s, ",")
	if !ok {
		return geo.Point{}, fmt.Errorf("point %q must be lat,lon", s)
	}
	lat, err1 := strconv.ParseFloat(strings.TrimSpace(a), 64)
	lon, err2 := strconv.ParseFloat(strings.TrimSpace(b), 64)
	if err1 != nil || err2 != nil || lat < -90 || lat > 90 || lon < -180 || lon > 180 {
		return geo.Point{}, fmt.Errorf("point %q is not a valid lat,lon", s)
	}
	return geo.Point{Lat: lat, Lon: lon}, nil
}

func httpError(w http.ResponseWriter, code int, msg string) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(code)
	json.NewEncoder(w).Encode(map[string]string{"error": msg})
}
```

Design points to recognise:

- **Immutable engine + `atomic.Pointer`:** a request loads the pointer **once** and uses that engine for the whole request, so a concurrent reload can never mix two datasets in one answer.
- **A `sync.Pool` of routers per engine:** routers hold large scratch arrays and are not concurrency-safe; pooling avoids allocating megabytes per request (Unit 7.11). The pool belongs to the engine, so routers sized for an old graph are never used with a new one.
- **Validation** of every parameter (`errors.Join` reports both bad points at once), **422** for "well-formed but cannot be served" (too far from roads), **404** for "no route".

A real response for a corner-to-corner route across the synthetic city (`from=18.4512,73.8021&to=18.7051,74.0752`), with the 1,738-character polyline shortened here:

```
{"duration_s":3462.631,"distance_m":57356.882265613545,"polyline":"euroBwlmaMClDxD?KeDJ_EK{DTqEQmD…",
 "snap_from_m":39.90819003015627,"snap_to_m":42.7891761858107,"settled_nodes":80274,"algorithm":"astar"}
```

Two things to notice. A route across the whole map settles 80,274 of 90,000 nodes even with A*: the heuristic helps most for short and medium routes, and long routes are where preprocessing (Unit 13.7) pays off. And `distance_m` has fourteen meaningless decimal places — rounding it to whole metres is a small, real improvement to the API.

Your work: API versioning (`/v1/route`), a request size and rate limit (Unit 11.6 `guard`), response field documentation, a `POST /v1/matrix` endpoint (many-to-many durations, a common product need; think about its cost: n×m routes, or one-to-many Dijkstra runs), optional gRPC, and caching only if measurements justify it.

### Acceptance criteria

Every parameter validated with tests for each error; the API documented (OpenAPI or a Markdown reference); no shared mutable state without synchronisation (`go test -race` with concurrent requests).

## Unit 14.5 — Hot reload of graph data and failure handling

**One-line summary:** Replace the graph without downtime and without risk: build the new engine in the background, validate it, swap atomically, keep serving the old one if anything fails — and plan for every other failure the design document lists.

### The starting point

`routed` reloads on `SIGHUP`:

```go
// SIGHUP: rebuild from the file in the background, then swap atomically.
// A bad file is logged and ignored: the old graph keeps serving.
hup := make(chan os.Signal, 1)
signal.Notify(hup, syscall.SIGHUP)
go func() {
	for range hup {
		e, err := server.Load(*path)
		if err != nil {
			log.Error("reload failed; keeping current graph", "err", err)
			continue
		}
		s.Swap(e)
		log.Info("graph reloaded", "nodes", e.G.NumNodes())
	}
}()
```

The test proves the property that matters: **400 requests from 8 concurrent clients all succeed while the graph is swapped** from a 1,600-node to a 3,600-node city mid-flight:

```go
// Hot reload under load: requests keep succeeding while the graph is swapped.
bigger, err := Load(city(t, "60"))
if err != nil {
	t.Fatal(err)
}
var wg sync.WaitGroup
for range 8 {
	wg.Add(1)
	go func() {
		defer wg.Done()
		for range 50 {
			if code, _ := get(q); code != 200 {
				t.Errorf("status %d during reload", code)
				return
			}
		}
	}()
}
s.Swap(bigger)
wg.Wait()
```

### Failure handling to implement

| Failure | Required behaviour |
| --- | --- |
| New data file corrupt or fails validation | keep serving the old graph; alert; expose the loaded version in `/healthz` and metrics |
| Not enough memory for two graphs during the swap | plan capacity (old + new must fit; `GOMEMLIMIT`, Unit 10.6), or do blue-green instance replacement instead of in-process reload |
| Overload | bounded concurrency per instance (semaphore, Unit 7.10), 503 with `Retry-After` beyond it, autoscaling on CPU or in-flight requests |
| Pathological queries (very long routes) | a per-request deadline enforced inside the search loop (check `ctx.Err()` every N settled nodes) |
| Slow clients | server timeouts (Unit 8.4) |
| Bad deploy | readiness gates, canary, automatic rollback on error-rate or latency regression (Unit 11.7) |
| Instance crash | stateless instances; the load balancer routes around it; restarts load the graph quickly (memory-mapped files make this seconds instead of minutes) |

### Acceptance criteria

A chaos test suite: corrupt file reload (old graph keeps serving), reload under load (no errors), overload (503s, no crash, latency of admitted requests within target), a request deadline that stops a cross-region search. All automated.

## Topic 14C — Prove and operate

## Unit 14.6 — Load testing, profiling, and optimisation

**One-line summary:** Find the service's real capacity and its bottleneck with realistic load, then optimise with the Chapter 10 loop until the requirements are met with headroom. Report it like a professional.

### A load generator

```go
// Command loadgen sends random route requests with a fixed number of
// concurrent workers for a fixed time and reports throughput and latency
// percentiles.
//
//	loadgen -url http://localhost:8090 -c 8 -d 20s
package main

import (
	"context"
	"flag"
	"fmt"
	"io"
	"math/rand/v2"
	"net/http"
	"slices"
	"sync"
	"time"
)

func main() {
	base := flag.String("url", "http://localhost:8090", "server")
	conc := flag.Int("c", 8, "concurrent workers")
	dur := flag.Duration("d", 10*time.Second, "test duration")
	algo := flag.String("algo", "astar", "algorithm")
	flag.Parse()

	ctx, cancel := context.WithTimeout(context.Background(), *dur)
	defer cancel()
	client := &http.Client{Timeout: 5 * time.Second,
		Transport: &http.Transport{MaxIdleConnsPerHost: *conc}} // DT-246: reuse connections
	var mu sync.Mutex
	var lat []time.Duration
	errors := 0
	var wg sync.WaitGroup
	for w := range *conc {
		wg.Add(1)
		go func() {
			defer wg.Done()
			r := rand.New(rand.NewPCG(uint64(w), 99))
			for ctx.Err() == nil {
				// random points inside the synthetic city's bounding box
				q := fmt.Sprintf("%s/route?from=%.5f,%.5f&to=%.5f,%.5f&algo=%s", *base,
					18.45+r.Float64()*0.26, 73.80+r.Float64()*0.28,
					18.45+r.Float64()*0.26, 73.80+r.Float64()*0.28, *algo)
				start := time.Now()
				resp, err := client.Get(q)
				if err == nil {
					io.Copy(io.Discard, resp.Body)
					resp.Body.Close()
				}
				d := time.Since(start)
				mu.Lock()
				if err != nil || resp.StatusCode >= 500 {
					errors++
				} else {
					lat = append(lat, d)
				}
				mu.Unlock()
			}
		}()
	}
	wg.Wait()
	slices.Sort(lat)
	p := func(q float64) time.Duration { return lat[int(q*float64(len(lat)-1))].Round(10 * time.Microsecond) }
	fmt.Printf("%s, %d workers, %v: %d requests (%.0f/s), errors %d, p50 %v, p95 %v, p99 %v, max %v\n",
		*algo, *conc, *dur, len(lat), float64(len(lat))/dur.Seconds(), errors, p(0.50), p(0.95), p(0.99), lat[len(lat)-1].Round(10*time.Microsecond))
}
```

This is a **closed-loop** generator: each worker waits for a response before sending the next request, so it measures capacity at a given concurrency. Real users form an **open loop** (requests arrive regardless of how fast you answer), which closed-loop tests can under-represent at overload ("coordinated omission"); tools such as `vegeta` (constant request rate) or `k6` model open loops. Use both.

### Real results for the starting point

The synthetic 90,000-node city, `routed` and `loadgen` on the same 2-vCPU machine (so the load generator steals CPU from the server — a real test uses separate machines), 4 workers, 10 seconds:

```
dijkstra, 4 workers, 10s: 4266 requests (427/s), errors 0, p50 6.98ms, p95 27.4ms, p99 38.47ms, max 98.9ms
astar,    4 workers, 10s: 7236 requests (724/s), errors 0, p50 2.85ms, p95 20.84ms, p99 31.33ms, max 79.62ms
```

How to read it: A* serves 70% more requests per second at a lower median; but p95 and p99 are an order of magnitude above the median — random origin–destination pairs vary from a few blocks to across the city, and queueing on 2 cores amplifies the long ones. The single-query benchmark (2.75 ms average for A*) does not predict the p99 under load: **only a load test does**.

### What to do next (your capstone work)

1. Profile under load (`/debug/pprof/profile`, Unit 10.2). Expect the haversine heuristic, heap operations, JSON encoding of long polylines, and `Nearest` to be visible.
2. Apply the Chapter 13 extensions (cheap heuristic, node reordering) and, if needed, ALT or CH; measure each with benchstat and with the load test.
3. Test with realistic query distributions (most real routes are short; use a distribution from logs or a gravity model), at the target rate, for long enough to see GC and tail behaviour (minutes, not seconds).
4. Find the knee: the request rate at which p99 starts to rise sharply. Set autoscaling and load-shedding thresholds below it.

### Acceptance criteria

A performance report (the Chapter 10 project's format) showing the requirement met — p99 under target at target load — with profiles before and after each optimisation, the open-loop results, and the capacity knee per instance.

## Unit 14.7 — Observability, deployment, and scalability

**One-line summary:** Operate the service like the Chapter 11 shortener — structured logs, RED metrics, traces, health checks, a secure container, CI/CD, Kubernetes — plus routing-specific signals and a scaling plan.

### Routing-specific observability

- Metrics: route latency histogram per algorithm; **settled nodes** histogram (work per query — a regression in data or code shows here before latency does); snapping distance histogram; error counts by type (`no_route`, `too_far`); loaded graph version, node and edge counts, load time; reload success/failure counter.
- Logs: one line per request with origin and destination **rounded** (precise coordinates are personal data — DT-293), duration, settled nodes, request ID.
- Traces: spans for snapping, search, and response encoding.
- Dashboards and alerts as code (Prometheus rules, Grafana JSON) in the repository.

### Deployment

The Chapter 11 Dockerfile, CI, and Kubernetes manifests apply directly, with two changes: the graph file is mounted from a volume or downloaded at startup (not baked into the image, so data and code deploy independently), and readiness stays false until the graph is loaded (startup can take seconds or minutes; use a Kubernetes `startupProbe` so liveness does not kill a slowly loading pod).

### Scalability

- **Vertical:** the graph is read-only and shared by all goroutines; throughput scales with cores until memory bandwidth limits it.
- **Horizontal:** stateless instances behind a load balancer; each holds a full copy of the regional graph.
- **Beyond one region's memory:** shard by region (a route between regions needs either an overlay graph of boundary nodes or a coarser long-distance graph) — say what you would do, even if you do not build it.

### The runbook

For each alert: what it means, how to confirm it, how to mitigate (roll back, scale out, shed load, revert data), how to escalate. Include: "graph reload failed", "p99 latency above SLO", "error rate above 1%", "memory near limit", "settled nodes per query doubled". Practise each once.

### Acceptance criteria

Dashboards that answer "is it healthy, is it fast, is it correct, what data version is it running" at a glance; alerts tied to the SLO; a runbook entry for every alert; a deployment that survives a pod being killed under load with no user-visible errors.

### Deep & Tricky

**DT-293. Location data is personal data.** *(Privacy and law)* Origins and destinations reveal homes, workplaces, hospitals, and places of worship. Logging exact coordinates with user identifiers creates a sensitive dataset subject to privacy law (India's DPDP Act, the EU's GDPR). Round or truncate coordinates in logs, avoid storing request histories by default, set retention limits, and restrict access.

## Unit 14.8 — Final senior-level architecture review

**One-line summary:** Present the system and answer the questions a principal engineer would ask. Being able to defend decisions with evidence — and to say clearly what you would change — is the final skill this book teaches.

### Prepare

A 20-minute walkthrough: the problem and requirements, the architecture, three key decisions with alternatives and evidence, the performance report, the failure-mode table, the operational story, and what you would do with another month.

### Questions to be ready for

**Correctness**
- How do you know your routes are optimal? What is your oracle, how many cases does it check, and what classes of bug would it miss?
- What happens with one-way streets, turn restrictions, ferries, and closed roads? Which do you ignore, and what is the user impact?
- How do you detect a bad data import before users see it?

**Performance**
- What is the p99 at target load, measured how, on what hardware, with what query distribution? Where is the knee?
- What does the CPU profile look like at peak? What is the next optimisation and its expected gain?
- How much memory does an instance need, and what happens during a reload?

**Reliability and operations**
- Walk me through a node failure, a bad deploy, a corrupt data file, and a 10× traffic spike.
- How long does a new instance take to become ready? What would make it faster?
- Which alerts would wake someone up, and are they all actionable?

**Design judgement**
- Why this algorithm and not contraction hierarchies (or why CH and not A*)? What would make you change your mind?
- Where did you choose simplicity over performance, and how will you know when that choice stops being right?
- What would you do differently if you started again?

**Security and privacy**
- What input can make the service misbehave or consume excessive resources? How is it bounded?
- What data do you log and keep, and why?

### Acceptance criteria

You can answer every question above with evidence (a test, a measurement, a design-document section) or with a clear "we have not done this; here is the risk and the plan". The second kind of answer, given honestly, is a mark of seniority — not a failure.

## Where to go from here

You started this book with the difference between compile time and run time and ended designing a production routing system. Along the way you built a habit that matters more than any single fact: **do not guess — measure, test, and read the evidence.** Every chapter's "notes from development" — the guessed test values that were wrong, the race that the tests did not catch, the `go vet` warnings that found real bugs, the PGO run that changed nothing — are the normal texture of professional work. Expect them, and build the checks that catch them.

Ways to keep growing:

- **Read Go's source and proposals** (Unit 5.11): `go.dev/issue`, the release notes of each new version, and the design documents behind generics, iterators, and the memory model.
- **Contribute** to an open-source Go project: a bug fix with a good test teaches more than many tutorials.
- **Go deeper in a direction** this book opened: databases (*Designing Data-Intensive Applications*), distributed systems (the Raft paper, Jepsen analyses), performance (the Go runtime source, *Systems Performance*), routing (the contraction-hierarchies and customisable-route-planning papers).
- **Teach**: explaining a concept to someone else — as this book's original notes were written, one unit at a time — is the fastest way to find what you have not yet understood.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and try to explain the entry before looking it up.

## Deep & Tricky Go Knowledge

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | `go run` still compiles, and Go caches builds. | Unit 0.1 |
| DT-2 | Your binary carries the Go runtime inside it. | Unit 0.1 |
| DT-3 | Cross-compilation is built in. | Unit 0.1 |
| DT-4 | Pure Go programs are usually statically linked. | Unit 0.1 |
| DT-5 | Unused imports vs unused variables — two different rules. | Unit 0.1 |
| DT-6 | Division by zero: compile time, run time, and floats all differ. | Unit 0.1 |
| DT-7 | `NaN` is not equal to itself. | Unit 0.1 |
| DT-8 | Same zero, different verdict: int variable vs float variable. | Unit 0.1 |
| DT-9 | `go run` hides the real exit code. | Unit 0.1 |
| DT-10 | Go constants are exact numbers, not IEEE floats. | Unit 0.1 |
| DT-11 | `$?` remembers only the last command. | Unit 0.1 |
| DT-12 | A Go binary carries debug information too. | Unit 0.1 |
| DT-13 | A leading zero makes a number octal. | Unit 0.2 |
| DT-14 | The size of `int` is not fixed by the language. | Unit 0.2 |
| DT-15 | `byte` is just another name for `uint8`. | Unit 0.2 |
| DT-16 | Byte order (endianness) — the `LSB` in `file` output. | Unit 0.2 |
| DT-17 | Why `{` must be on the same line: automatic semicolons. | Unit 0.2 |
| DT-18 | `%x` vs `%X` bugs hide when there are no letters. | Unit 0.2 |
| DT-19 | Integer overflow never panics in Go, and the result is defined. | Unit 0.3 |
| DT-20 | `fmt.Println(math.MaxUint64)` does not compile. | Unit 0.3 |
| DT-21 | "Underflow" is used loosely. | Unit 0.3 |
| DT-22 | A countdown with an unsigned counter never ends. | Unit 0.3 |
| DT-23 | Same bits, different number — the type decides. | Unit 0.4 |
| DT-24 | Negating the minimum value gives itself. | Unit 0.4 |
| DT-25 | `math.MinInt64 / -1` does not crash in Go. | Unit 0.4 |
| DT-26 | The compiler may NOT assume `x < x + 1`. | Unit 0.4 |
| DT-27 | `uint8(-5)` vs `uint8(x)` — "who calculates?" once more. | Unit 0.4 |
| DT-28 | Who really drops the 9th bit? The compiler plans it, the CPU does it. | Unit 0.4 |
| DT-29 | Same size, different type: `int` vs `int64`. | Unit 0.4 |
| DT-30 | Constant folding: `c := 2 + 3` has no ADD at run time. | Unit 0.5 |
| DT-31 | Same ADD, different compare and divide. | Unit 0.5 |
| DT-32 | Apple Silicon uses bigger cache lines. | Unit 0.5 |
| DT-33 | Cache lines are aligned blocks. | Unit 0.5 |
| DT-34 | The Go specification never mentions "stack" or "heap". | Unit 0.6 |
| DT-35 | Goroutine stacks grow by copying. | Unit 0.6 |
| DT-36 | Apple Silicon pages are 16 KB. | Unit 0.6 |
| DT-37 | Goroutine stacks live in runtime-managed memory. | Unit 0.6 |
| DT-38 | Go passes arguments and return values in registers. | Unit 0.6 |
| DT-39 | Stack overflow is a fatal error, not a panic. | Unit 0.6 |
| DT-40 | Inlined functions show `(...)` in stack traces. | Unit 0.6 |
| DT-41 | Converting to a smaller integer type keeps only the lowest bits. | Unit 0.6 |
| DT-42 | Even a local that never escapes may go to the heap. | Unit 0.6 |
| DT-43 | Every variable starts at its zero value — a spec guarantee, implemented cleverly. | Unit 0.6 |
| DT-44 | Hex formatting of negative numbers. | Unit 0.6 |
| DT-45 | `:=` with existing variables: the `err` idiom. | Unit 1.1 |
| DT-46 | Multiple assignment evaluates the right side first. | Unit 1.1 |
| DT-47 | `1e3` is a float, even though 1000 is a whole number. | Unit 1.1 |
| DT-48 | "Make the zero value useful." | Unit 1.1 |
| DT-49 | Your editor and the compiler can word the same error differently. | Unit 1.1 |
| DT-50 | `var i int = 2.0` compiles. | Unit 1.2 |
| DT-51 | Compilers must handle very large constants. | Unit 1.2 |
| DT-52 | iota counts lines, not uses. | Unit 1.2 |
| DT-53 | `'a' + 1` prints 98. | Unit 1.2 |
| DT-54 | `var f float64 = 7 / 2` gives 3, not 3.5. | Unit 1.3 |
| DT-55 | Go truncates towards zero; Python floors. | Unit 1.3 |
| DT-56 | `x++` is a statement on purpose. | Unit 1.3 |
| DT-57 | `fmt.Println` prints `4.0` as `4`, and `1000000.0` as `1e+06`. | Unit 1.3 |
| DT-58 | `-7 >> 1` is −4, but `-7 / 2` is −3. | Unit 1.4 |
| DT-59 | Shifts are fully defined in Go. | Unit 1.4 |
| DT-60 | Untyped constants in non-constant shifts. | Unit 1.4 |
| DT-61 | Go has no power operator. | Unit 1.4 |
| DT-62 | `0.1 + 0.2 == 0.3` is TRUE in Go when written as constants. | Unit 1.5 |
| DT-63 | float32 → float64 shows the hidden error. | Unit 1.5 |
| DT-64 | Out-of-range float → int conversion does not panic. | Unit 1.5 |
| DT-65 | Negative zero. | Unit 1.5 |
| DT-66 | Integers above 2^53 lose exactness in float64. | Unit 1.5 |
| DT-67 | `fallthrough` ignores the next case's condition. | Unit 1.6 |
| DT-68 | `break` inside a switch only leaves the switch. | Unit 1.6 |
| DT-69 | Switch cases can be any expressions, evaluated in order. | Unit 1.6 |
| DT-70 | Safe absolute value for printing: `uint64(-n)`. | Unit 1.6 |
| DT-71 | Changing the loop variable: three-part loop vs range. | Unit 1.7 |
| DT-72 | Since Go 1.22, each loop round gets a NEW variable. | Unit 1.7 |
| DT-73 | Labels are only for break, continue, and goto. | Unit 1.7 |
| DT-74 | Adding 0.1 ten times gives 0.9999999999999999. | Unit 1.7 |
| DT-75 | Go is ALWAYS pass-by-value. | Unit 1.8 |
| DT-76 | Go does not optimise tail calls. | Unit 1.8 |
| DT-77 | Calling a nil function value panics. | Unit 1.8 |
| DT-78 | Function values cannot be compared, except with nil. | Unit 1.8 |
| DT-79 | Multiple results are not tuples. | Unit 1.8 |
| DT-80 | Go has only 25 keywords; `true`, `nil`, `int`, `len` are NOT keywords. | Unit 1.9 |
| DT-81 | Imports belong to the file, not the package. | Unit 1.9 |
| DT-82 | The compiler only catches shadowing when the inner variable is unused. | Unit 1.9 |
| DT-83 | A new variable's scope starts AFTER its declaration. | Unit 1.9 |
| DT-84 | Why calling nil crashes: page zero is never mapped. | Unit 1.9 |
| DT-85 | Closures capture variables, not values. | Unit 1.10 |
| DT-86 | Captured variables that outlive their function move to the heap. | Unit 1.10 |
| DT-87 | The loop-variable change is per module, controlled by go.mod. | Unit 1.10 |
| DT-88 | `defer` takes a photo of the arguments, and of the function value. | Unit 1.11 |
| DT-89 | Deferring a nil function panics LATER, not at the defer line. | Unit 1.11 |
| DT-90 | Deferred calls run before the panic message is printed. | Unit 1.11 |
| DT-91 | Defer is almost free now (usually). | Unit 1.11 |
| DT-92 | bufio.Scanner has a 64 KB line limit by default. | Unit 1.12 |
| DT-93 | Scanner removes Windows line endings for you. | Unit 1.12 |
| DT-94 | fmt.Scan treats newlines as spaces; fmt.Scanln does not. | Unit 1.12 |
| DT-95 | Old math/rand gave the SAME numbers every run (before Go 1.20). | Unit 1.13 |
| DT-96 | Modulo bias: why `rand.IntN` exists. | Unit 1.13 |
| DT-97 | crypto/rand.Read no longer returns an error (Go 1.24+). | Unit 1.13 |
| DT-98 | `range` over an array iterates over a COPY of the array. | Unit 2.1 |
| DT-99 | Constant indexes are checked at compile time. | Unit 2.1 |
| DT-100 | Arrays are comparable and can be map keys; slices cannot. | Unit 2.1 |
| DT-101 | A slice expression on an array makes the array escape — or not. | Unit 2.2 |
| DT-102 | `len` and `cap` of a nil slice are 0, and you can range over it, append to it, and slice it with `[:0]`. | Unit 2.2 |
| DT-103 | Re-slicing up to capacity can expose "deleted" data. | Unit 2.2 |
| DT-104 | Indexing is checked against `len`, slicing against `cap`. | Unit 2.2 |
| DT-105 | Growth rounds up to size classes. | Unit 2.3 |
| DT-106 | `append` to a nil slice works; there is no need to `make` first. | Unit 2.3 |
| DT-107 | A function that appends to a shared slice can corrupt the caller's other data. | Unit 2.3 |
| DT-108 | `copy` copies by length, not capacity. | Unit 2.3 |
| DT-109 | `slices.Delete` and friends zero the tail (Go 1.22+). | Unit 2.4 |
| DT-110 | `range` over a slice sees writes; over an array it does not. | Unit 2.4 |
| DT-111 | `slices.Sort` is not stable; `slices.SortStableFunc` is. | Unit 2.4 |
| DT-112 | Strings may contain any bytes, not only valid UTF-8. | Unit 2.5 |
| DT-113 | `string(i)` for an integer `i` is a rune conversion. | Unit 2.5 |
| DT-114 | The compiler avoids some `[]byte` ↔ `string` copies. | Unit 2.5 |
| DT-115 | Comparing strings is byte-wise, so "é" may not equal "é". | Unit 2.5 |
| DT-116 | `strings.ToUpper` can change the length of a string. | Unit 2.6 |
| DT-117 | Simple case mapping is not full case mapping. | Unit 2.6 |
| DT-118 | A `strings.Builder` panics if copied after use. | Unit 2.6 |
| DT-119 | Go 1.24 rewrote maps as "Swiss tables". | Unit 2.7 |
| DT-120 | A map with NaN keys can hold "the same" key many times. | Unit 2.7 |
| DT-121 | Iteration order is randomised per iteration, not just per run. | Unit 2.7 |
| DT-122 | Adding to a map during `range` may or may not show up in the same loop. | Unit 2.7 |
| DT-123 | `clear(m)` is not the same as `m = make(...)`. | Unit 2.8 |
| DT-124 | `fatal error: concurrent map writes` is best-effort detection. | Unit 2.8 |
| DT-125 | Zero-size types share an address. | Unit 2.8 |
| DT-126 | Struct literals of pointer type: `&T{...}` is fine, and Go allows `{...}` shorthand inside composite literals. | Unit 2.9 |
| DT-127 | Field order and names are part of the type. | Unit 2.9 |
| DT-128 | Comparing structs with `==` compares padding-free field values, not memory. | Unit 2.9 |
| DT-129 | A zero-size field at the end of a struct adds padding. | Unit 2.10 |
| DT-130 | 64-bit atomic operations need 8-byte alignment, which 32-bit platforms do not give automatically. | Unit 2.10 |
| DT-131 | Go does not reorder struct fields. | Unit 2.10 |
| DT-132 | A pointer into a slice's backing array goes stale after a growing append. | Unit 2.11 |
| DT-133 | Pointers are comparable; equal pointers point to the same variable. | Unit 2.11 |
| DT-134 | Taking the address of a local is always safe in Go. | Unit 2.11 |
| DT-135 | "Pass a pointer to avoid copying" can make code slower. | Unit 2.12 |
| DT-136 | `go vet` reports copied locks. | Unit 2.12 |
| DT-137 | A named type does not inherit the methods of its underlying named type. | Unit 3.1 |
| DT-138 | Typed constants of a named type give you compile-time checked "units". | Unit 3.1 |
| DT-139 | `%v` on a type with a `String()` method that formats itself with `%v` recurses forever. | Unit 3.2 |
| DT-140 | Untyped constants still sneak through. | Unit 3.2 |
| DT-141 | A method value evaluates (copies) its receiver immediately. | Unit 3.3 |
| DT-142 | Map elements and return values are not addressable, so they cannot call pointer methods. | Unit 3.3 |
| DT-143 | Mixing value and pointer receivers creates "lost update" bugs. | Unit 3.3 |
| DT-144 | Value vs pointer receivers decide which types satisfy an interface. | Unit 3.4 |
| DT-145 | `fmt` checks for `error` before `Stringer`. | Unit 3.4 |
| DT-146 | Interfaces are satisfied by method *names and signatures*, not meaning. | Unit 3.4 |
| DT-147 | A nil pointer inside an interface is not a nil interface. | Unit 3.5 |
| DT-148 | Interface comparison can panic. | Unit 3.5 |
| DT-149 | Method calls through interfaces cost an indirect call and block some optimisations. | Unit 3.5 |
| DT-150 | Errors should be inspected with `errors.As`, not type assertions. | Unit 3.6 |
| DT-151 | A failed assertion to an interface type and to a concrete type panic differently. | Unit 3.6 |
| DT-152 | Embedding a type with a `sync.Mutex` promotes `Lock` and `Unlock` — publicly. | Unit 3.7 |
| DT-153 | Embedding a type promotes its `String()` or `Error()`, which can hijack printing. | Unit 3.7 |
| DT-154 | Promoted fields in composite literals are a Go 1.27 feature. | Unit 3.7 |
| DT-155 | Returning an internal slice leaks write access. | Unit 3.8 |
| DT-156 | Options structs make "zero" and "unset" the same. | Unit 3.8 |
| DT-157 | A function returning `(T, error)` usually leaves T unspecified on error. | Unit 3.9 |
| DT-158 | `errors.New` creates a distinct value each call. | Unit 3.9 |
| DT-159 | `errors.As` panics if the target is not a non-nil pointer to an error type or interface. | Unit 3.10 |
| DT-160 | A type can customise matching with `Is(target error) bool` and `As(target any) bool` methods. | Unit 3.10 |
| DT-161 | `go vet` (Go 1.27) flags `%w` with a pointer to a value-receiver error type. | Unit 3.10 |
| DT-162 | Wrapping `io.EOF` breaks callers that check `err == io.EOF`. | Unit 3.10 |
| DT-163 | `panic(nil)` became a real panic in Go 1.21. | Unit 3.11 |
| DT-164 | Re-panicking keeps the original stack in the message. | Unit 3.11 |
| DT-165 | `runtime.Goexit` runs defers but is not a panic. | Unit 3.11 |
| DT-166 | A test file can be in the same package or in `pkg_test`. | Unit 3.12 |
| DT-167 | Unexported fields are invisible to reflection-based encoders. | Unit 3.12 |
| DT-168 | Dot imports and package-name shadowing hurt readability. | Unit 3.12 |
| DT-169 | The `go` line can trigger a toolchain download. | Unit 3.13 |
| DT-170 | Language changes are gated by each module's `go` line. | Unit 3.13 |
| DT-171 | Never edit `go.sum` by hand, and commit it. | Unit 3.13 |
| DT-172 | `comparable` admits interface types, whose `==` can panic. | Unit 3.14 |
| DT-173 | Type inference never uses the result type of a call alone. | Unit 3.14 |
| DT-174 | A type parameter's methods are only those in its constraint. | Unit 3.15 |
| DT-175 | Generic code that calls methods through a type parameter may be slower than an interface call. | Unit 3.15 |
| DT-176 | An iterator that ignores `yield`'s false result panics. | Unit 3.16 |
| DT-177 | `defer` inside an iterator runs when the iterator function returns — after the loop ends or breaks. | Unit 3.16 |
| DT-178 | Single-use vs reusable iterators. | Unit 3.16 |
| DT-179 | `t.Fatal` must be called from the test's own goroutine. | Unit 4.1 |
| DT-180 | Test results are cached. | Unit 4.1 |
| DT-181 | Since Go 1.27, `go test` runs the `stdversion` vet check by default. | Unit 4.1 |
| DT-182 | `TestMain` controls the whole package's test run. | Unit 4.2 |
| DT-183 | Parallel subtests run after the parent function returns. | Unit 4.2 |
| DT-184 | Coverage counts statements, not branches or conditions. | Unit 4.3 |
| DT-185 | A benchmark is a hypothesis test, not a speed reading. | Unit 4.5 |
| DT-186 | The library project's `List` benchmark reveals an O(books × loans) loop. | Unit 4.5 |
| DT-187 | Fuzzing uses all CPU cores and grows the cache. | Unit 4.6 |
| DT-188 | Linters can be silenced line by line, and that should be rare and explained. | Unit 4.7 |
| DT-189 | Stack traces show argument words, not argument values. | Unit 4.8 |
| DT-190 | `Read` can return data and `io.EOF` at the same time. | Unit 5.1 |
| DT-191 | `io.ReadAll` on untrusted input is a denial-of-service bug. | Unit 5.1 |
| DT-192 | `defer f.Close()` on a file you wrote can lose errors — and data. | Unit 5.2 |
| DT-193 | Checking whether a file exists before opening it is a race. | Unit 5.2 |
| DT-194 | `path` and `path/filepath` are different on Windows. | Unit 5.3 |
| DT-195 | Unmarshalling into a non-empty value merges. | Unit 5.4 |
| DT-196 | Duplicate keys: v1 takes the last, v2 rejects. | Unit 5.4 |
| DT-197 | HTML characters are escaped by default. | Unit 5.4 |
| DT-198 | `binary.Read` with a struct is reflection-based and slow. | Unit 5.5 |
| DT-199 | A UTF-8 byte-order mark can hide in CSV files from Excel. | Unit 5.5 |
| DT-200 | Never compare `time.Time` with `==`, and never use it as a map key without normalising. | Unit 5.6 |
| DT-201 | `time.Duration(n) * time.Second` vs `time.Duration(n * time.Second)`. | Unit 5.6 |
| DT-202 | Layout mistakes compile and fail silently. | Unit 5.6 |
| DT-203 | Child processes do not die automatically when your program is killed. | Unit 5.7 |
| DT-204 | `cmd.Output()` with a huge output buffers it all in memory. | Unit 5.7 |
| DT-205 | A mismatched key–value list produces `!BADKEY`. | Unit 5.8 |
| DT-206 | Arguments are evaluated even when the level is disabled. | Unit 5.8 |
| DT-207 | `regexp` matches are leftmost-first, like Perl — not longest. | Unit 5.9 |
| DT-208 | Templates fail at execution, not only at parse. | Unit 5.9 |
| DT-209 | Amortised O(1) still has O(n) spikes. | Unit 6.1 |
| DT-210 | Hash maps are O(1) only on average. | Unit 6.1 |
| DT-211 | Recursive closures must be declared before they are assigned. | Unit 6.2 |
| DT-212 | A slice-based queue that pops with `q = q[1:]` can leak memory. | Unit 6.3 |
| DT-213 | A good hash for a composite key combines all its parts. | Unit 6.4 |
| DT-214 | `(lo + hi) / 2` can overflow. | Unit 6.5 |
| DT-215 | Comparison-based sorting cannot beat O(n log n). | Unit 6.6 |
| DT-216 | Sorting floats with NaN using `<` breaks. | Unit 6.6 |
| DT-217 | Sorted slices beat trees for read-mostly data. | Unit 6.7 |
| DT-218 | Building a heap from n elements is O(n), not O(n log n). | Unit 6.8 |
| DT-219 | Recursive DFS on large graphs can overflow the stack — or just be slow. | Unit 6.10 |
| DT-220 | Dijkstra's correctness depends on non-negative weights — and so does your early exit. | Unit 6.11 |
| DT-221 | The same algorithm can be 100× apart depending on allocation order. | Unit 6.13 |
| DT-222 | An unrecovered panic in any goroutine kills the whole program. | Unit 7.2 |
| DT-223 | Goroutines have no identity you can use. | Unit 7.2 |
| DT-224 | A buffered channel is not a queue you should inspect. | Unit 7.3 |
| DT-225 | Races on multi-word values can crash, not just give wrong numbers. | Unit 7.3 |
| DT-226 | Before Go 1.23, `time.After` in a loop leaked timers until they fired. | Unit 7.4 |
| DT-227 | An empty `select {}` blocks forever. | Unit 7.4 |
| DT-228 | `wg.Add` inside the goroutine is a race with `Wait`. | Unit 7.5 |
| DT-229 | A mutex protects data, not code. | Unit 7.5 |
| DT-230 | Go's `Mutex` can be unlocked by a different goroutine than the one that locked it. | Unit 7.5 |
| DT-231 | Atomics are not free under contention. | Unit 7.6 |
| DT-232 | "Benign" races do not exist in Go. | Unit 7.7 |
| DT-233 | Double-checked locking without atomics is broken. | Unit 7.7 |
| DT-234 | The race detector cannot prove absence of races. | Unit 7.8 |
| DT-235 | Every goroutine needs an owner and an exit path. | Unit 7.8 |
| DT-236 | Forgetting `cancel()` leaks until the parent is cancelled. | Unit 7.9 |
| DT-237 | Cancellation is cooperative. | Unit 7.9 |
| DT-238 | Unbounded goroutine creation is a bug waiting for a big input. | Unit 7.10 |
| DT-239 | Results from a worker pool arrive in completion order. | Unit 7.10 |
| DT-240 | A test that passes with `-count=1` can fail with `-count=1000`. | Unit 7.12 |
| DT-241 | "It works on localhost" hides latency and loss. | Unit 8.1 |
| DT-242 | A write that succeeded does not mean the peer received it. | Unit 8.2 |
| DT-243 | Each connection uses a file descriptor, and the limit is finite. | Unit 8.2 |
| DT-244 | `http.ListenAndServe` with no timeouts is vulnerable to slow clients. | Unit 8.4 |
| DT-245 | `WriteTimeout` covers the whole handler. | Unit 8.4 |
| DT-246 | Creating a new `http.Client` (or `Transport`) per request defeats pooling. | Unit 8.5 |
| DT-247 | `Client.Timeout` includes reading the body. | Unit 8.5 |
| DT-248 | Shutdown and readiness must be coordinated with the load balancer. | Unit 8.7 |
| DT-249 | Protobuf zero values are indistinguishable from "unset" for scalars in proto3. | Unit 8.8 |
| DT-250 | PostgreSQL stores timestamps with microsecond precision; Go has nanoseconds. | Unit 9.1 |
| DT-251 | `NOT IN` with a NULL in the list returns nothing. | Unit 9.1 |
| DT-252 | A query on a nullable or low-selectivity column may ignore your index — correctly. | Unit 9.2 |
| DT-253 | `LIKE 'abc%'` can use a B-tree index; `LIKE '%abc'` cannot. | Unit 9.2 |
| DT-254 | A bigger connection pool can make a database slower. | Unit 9.3 |
| DT-255 | A query that is waiting for a pool connection is not yet running. | Unit 9.3 |
| DT-256 | Holding a transaction or an unclosed `rows` while calling another service ties up a connection. | Unit 9.3 |
| DT-257 | Isolation anomalies produce no errors at READ COMMITTED. | Unit 9.4 |
| DT-258 | Retrying requires the whole transaction to be re-run, including the reads. | Unit 9.4 |
| DT-259 | A migration waiting for a lock blocks everything behind it. | Unit 9.5 |
| DT-260 | Caches hide capacity problems until they fail. | Unit 9.7 |
| DT-261 | A decorator hides optional interfaces unless it forwards them. | Unit 9.7 |
| DT-262 | Microbenchmarks can mislead about whole programs. | Unit 10.1 |
| DT-263 | CPU profiles miss time spent waiting. | Unit 10.2 |
| DT-264 | Profile labels attribute samples to requests or tenants. | Unit 10.2 |
| DT-265 | Tracing is more expensive than profiling, but cheaper than it used to be. | Unit 10.3 |
| DT-266 | Inlining changes escape decisions. | Unit 10.4 |
| DT-267 | Go 1.26 can put more slice backing arrays on the stack. | Unit 10.4 |
| DT-268 | Finalizers and `runtime.AddCleanup` are not destructors. | Unit 10.6 |
| DT-269 | A large live heap of pointer-free data is nearly free for the GC. | Unit 10.6 |
| DT-270 | `runtime.LockOSThread` pins a goroutine to its thread. | Unit 10.7 |
| DT-271 | Too many runnable goroutines raise latency, not throughput. | Unit 10.7 |
| DT-272 | A channel operation costs tens of nanoseconds; a mutex lock/unlock costs about ten when uncontended. | Unit 10.8 |
| DT-273 | `unsafe` conversions can outlive their safety. | Unit 10.10 |
| DT-274 | Error messages are an information leak. | Unit 11.2 |
| DT-275 | Timing can leak secrets. | Unit 11.2 |
| DT-276 | "Logged in" must be re-checked on every request. | Unit 11.3 |
| DT-277 | Requests rejected by outer middleware have no route. | Unit 11.5 |
| DT-278 | Averages hide the users who suffer. | Unit 11.5 |
| DT-279 | A static Go binary in `scratch` has no CA certificates or time zones. | Unit 11.6 |
| DT-280 | A failing liveness probe during load turns slowness into an outage. | Unit 11.8 |
| DT-281 | Exactly-once *delivery* is impossible; exactly-once *effects* are not. | Unit 12.1 |
| DT-282 | Database-generated IDs and timestamps do not order events across services. | Unit 12.2 |
| DT-283 | A distributed lock needs a fencing token. | Unit 12.5 |
| DT-284 | Ordering and parallelism pull in opposite directions. | Unit 12.6 |
| DT-285 | A retry is a new request to a system that may already be overloaded. | Unit 12.7 |
| DT-286 | Long-lived connections defeat connection-level load balancing. | Unit 12.8 |
| DT-287 | The antimeridian and the poles break naive code. | Unit 13.1 |
| DT-288 | Most OSM nodes are not intersections. | Unit 13.3 |
| DT-289 | Round edge times up, never down. | Unit 13.4 |
| DT-290 | Resetting O(n) arrays per query can cost more than the query. | Unit 13.6 |
| DT-291 | Bidirectional search must not stop at the first meeting node. | Unit 13.6 |
| DT-292 | Rounding in polyline encoding accumulates if you encode absolute values carelessly. | Unit 13.9 |
| DT-293 | Location data is personal data. | Unit 14.7 |

## Go Patterns & Engineering Practices

| # | Pattern | Where |
| --- | --- | --- |
| P-0 | Fail early (engineering principle, not yet a code pattern). | Unit 0.1 |
| P-1 | Use `int` for counts and sizes; keep unsigned types for bits and bytes. | Unit 0.3 |
| P-2 | Check before you calculate, when overflow matters. | Unit 0.3 |
| P-3 | CLI convention: results to stdout, errors to stderr, non-zero exit code on failure. | Chapter 0 Project |
| P-4 | Validate input at the edge. | Chapter 0 Project |
| P-5 | Always format with `gofmt`. | Chapter 0 Project |
| P-6 | Name constants in MixedCaps, never ALL\_CAPS. | Unit 1.2 |
| P-7 | Bit flags for small sets of on/off options. | Unit 1.4 |
| P-8 | Never use floats for money; use integer minor units. | Unit 1.5 |
| P-9 | Guard clauses: handle problems first, keep the happy path un-indented. | Unit 1.6 |
| P-10 | `(result, error)` with the error last. | Unit 1.8 |
| P-11 | Keep scopes small, but watch your `:=`. | Unit 1.9 |
| P-12 | Closures for small, local behaviour; structs for real state. | Unit 1.10 |
| P-13 | Acquire, check, then defer the cleanup. | Unit 1.11 |
| P-14 | Keep `main` tiny: `func run() error` + one `os.Exit`. | Unit 1.12 |
| P-15 | Secrets come from crypto/rand, always. | Unit 1.13 |
| P-16 | Arrays for fixed-size values, slices for lists. | Unit 2.1 |
| P-17 | Pre-size slices with `make([]T, 0, n)` when n is known or estimable. | Unit 2.3 |
| P-18 | Functions that may grow a slice return it. | Unit 2.3 |
| P-19 | Copy out small pieces of large buffers. | Unit 2.4 |
| P-20 | Parse with `strconv`, not `fmt.Sscan`, for single values; always keep the error. | Unit 2.6 |
| P-21 | Sort keys for any output a human or a test reads. | Unit 2.8 |
| P-22 | Pick value or pointer semantics per type, and stay consistent. | Unit 2.12 |
| P-23 | Consumer-defined, minimal interfaces. | Unit 3.4 |
| P-24 | Decorate with embedded interfaces. | Unit 3.7 |
| P-25 | Constructors that validate; unexported fields that protect invariants. | Unit 3.8 |
| P-26 | Wrap with context at every layer boundary; inspect only with `errors.Is`/`As`. | Unit 3.10 |
| P-27 | Generics for algorithms and containers; interfaces for behaviour. | Unit 3.15 |
| P-28 | Inject time, randomness, and I/O; keep the core pure. | Unit 4.4 |
| P-29 | Write functions against `io.Reader`/`io.Writer`, not file names. | Unit 5.1 |
| P-30 | Separate wire types from domain types. | Unit 5.4 |
| P-31 | Pick data structures by access pattern, then verify by measurement. | Unit 6.13 |
| P-32 | Confine state to one goroutine, or guard it with a documented mutex. | Unit 7.5 |
| P-33 | Context first, everywhere work can block. | Unit 7.9 |
| P-34 | One error-to-status mapping per service. | Unit 8.4 |
| P-35 | Prefer one atomic statement; otherwise lock or serialize — and retry. | Unit 9.4 |
| P-36 | Cache-aside with TTL, negative caching, and singleflight; the cache is never required. | Unit 9.7 |
| P-37 | Instrument RED for every endpoint and USE for every resource, from day one. | Unit 11.5 |
| P-38 | At-least-once delivery + idempotent handlers + bounded retries with jittered backoff + a dead-letter queue. | Unit 12.7 |

# Appendix B — Glossary

Short, plain definitions of the terms used in this book. The unit where a term is taught explains it fully.

| Term | Meaning |
| --- | --- |
| A* (A-star) | Shortest-path search that adds a lower-bound estimate of the remaining cost to Dijkstra's priority, steering the search towards the target (Unit 13.6) |
| Address | The number of one byte-sized box in memory |
| Admissible heuristic | An estimate that never overestimates the true remaining cost; keeps A* optimal |
| Allocation | Reserving memory for a value, usually on the heap; measured by benchmarks with -benchmem |
| ALT | A*, Landmarks, Triangle inequality: A* with precomputed landmark distances as a tighter heuristic (Unit 13.7) |
| Amortised cost | Average cost per operation over a long sequence, e.g. append is O(1) amortised although it sometimes copies |
| API | Application Programming Interface: the operations a package or service offers to its users |
| Atomic operation | A memory operation that happens indivisibly, with no data race (sync/atomic) |
| Backpressure | Slowing producers down when consumers cannot keep up, instead of buffering without limit |
| Benchmark | A function BenchmarkXxx(b *testing.B) that measures the time and allocations of code |
| Big-O notation | Describes how cost grows with input size, ignoring constants: O(1), O(log n), O(n), O(n log n), O(n²) |
| Binary | Base-2 number system; only digits 0 and 1 |
| Binary / executable | A file containing machine code that the computer can run directly |
| Binary search | Finding a value in sorted data by repeatedly halving the range; O(log n) |
| Bit | One 0-or-1 value; the smallest unit of data |
| Blank identifier | The name _, used to ignore a value |
| Buffered channel | A channel with capacity; sends block only when the buffer is full |
| Build tag (build constraint) | A //go:build line that includes a file only for certain OSes, architectures, or tags |
| Byte | 8 bits; can hold 256 different values (0 to 255) |
| Byte slice ([]byte) | A mutable sequence of bytes; the usual type for I/O buffers |
| Cache (CPU) | Small, fast memory near the CPU core holding recently used data in 64-byte lines |
| Cache-aside | Read from the cache, on a miss read the database and fill the cache (P-36) |
| CAP theorem | During a network partition a distributed system must choose between consistency and availability |
| Capacity (cap) | How many elements a slice's backing array can hold from the slice's start |
| Channel | A typed conduit for sending values between goroutines, with optional buffering |
| Circuit breaker | Stops calling a failing dependency for a while so it can recover and callers fail fast |
| Closure | A function value that refers to variables from the scope where it was created |
| Compile error | ANY reason the compiler refuses to build; no program is produced |
| Compile time | The moment the compiler checks and translates code, before running |
| Compiler | A program that translates your whole source code into machine code before it runs |
| Composite literal | A literal that builds a struct, array, slice, or map value: Point{X: 1} |
| Concurrency | Structuring a program as independently executing tasks; not necessarily simultaneous |
| Connection pool | A set of reusable open connections (database/sql and http.Transport keep one) |
| Consensus | Agreement among distributed nodes on a value despite failures (Raft, Paxos) |
| Context (context.Context) | Carries cancellation, deadlines, and request-scoped values across API boundaries |
| Contraction hierarchies (CH) | Routing preprocessing that adds shortcut edges so queries search only upwards in a node ordering |
| CPU | The chip that executes instructions, one tiny step at a time, very fast |
| CSR (compressed sparse row) | Graph storage as flat arrays: First[v]..First[v+1] index the edges of node v |
| Data race | Two goroutines access the same memory concurrently, at least one writes, without synchronisation; undefined behaviour |
| Deadlock | Goroutines waiting for each other forever; Go reports it when all goroutines are blocked |
| Defer | Schedules a function call to run when the surrounding function returns |
| Dependency injection | Passing a component its dependencies (often as interfaces) instead of creating them inside |
| Differential testing | Comparing a fast implementation's results with a simple, trusted one on many inputs |
| Dijkstra's algorithm | Shortest paths with non-negative edge weights by settling nodes in order of distance |
| ELF / PE / Mach-O | Executable file formats: ELF for Linux, PE (.exe) for Windows, Mach-O for macOS |
| Embedding | Including a type in a struct without a field name, promoting its fields and methods |
| Endianness | The order in which the bytes of a multi-byte number are stored |
| Error wrapping | Adding context to an error while keeping the original inspectable with errors.Is/As (%w) |
| Escape analysis | The compiler's decision whether a value can live on the stack or must move to the heap |
| Eventual consistency | Replicas may differ temporarily but converge when updates stop |
| Exactly-once (effectively) | At-least-once delivery combined with idempotent processing |
| Exit code | A number a finished process gives to the OS; 0 = success |
| Fan-in / fan-out | Merging several channels into one / distributing work from one source to many workers |
| Fuzzing | Automatically generating inputs to find crashes or broken properties (go test -fuzz) |
| Garbage collector (GC) | Runtime component that frees heap memory no longer reachable; Go's is concurrent mark-sweep |
| Generics | Functions and types with type parameters, constrained by interfaces (Go 1.18+) |
| Geohash | A string encoding of a lat/lon cell; nearby points usually share a prefix |
| GOARCH | Go setting that picks the target CPU type, e.g. amd64, arm64 |
| GOGC | Percentage of heap growth that triggers the next GC cycle (default 100) |
| GOMAXPROCS | Number of OS threads that can execute Go code simultaneously |
| GOMEMLIMIT | Soft memory limit for the Go runtime; the GC works harder as it is approached |
| GOOS | Go setting that picks the target operating system, e.g. linux, windows, darwin |
| Goroutine | A lightweight function execution managed by the Go scheduler; starts with a small stack |
| Goroutine leak | A goroutine blocked forever, holding its memory; usually a channel nobody will send to or receive from |
| gRPC | An RPC framework using HTTP/2 and Protocol Buffers |
| Haversine formula | Great-circle distance between two lat/lon points on a sphere |
| Heap (data structure) | A tree in an array where each parent is ≤ its children; gives a priority queue |
| Heap (memory) | Memory for values whose lifetime is not tied to one function call; managed by the GC |
| Hexadecimal (hex) | Base-16 system, digits 0-9 and A-F; one hex digit = 4 bits |
| Hot reload | Replacing data or configuration in a running service without downtime |
| Idempotency | Doing an operation twice has the same effect as doing it once |
| Idempotency key | A client-supplied ID letting a server recognise and deduplicate retried requests |
| IEEE 754 | The worldwide standard for how float numbers are stored and calculated |
| Inf | Special float value meaning infinity (+Inf or -Inf) |
| Interface | A type defined by a method set; any type with those methods satisfies it implicitly |
| Interpreter | A program that reads and runs source code directly, step by step, while running |
| Iterator (range-over-func) | A function of type iter.Seq or iter.Seq2 that a for-range loop can consume (Go 1.23+) |
| Latency percentile (p50, p99) | The latency below which 50% or 99% of requests complete |
| Literal | A value written directly in code, like `42`, `0xFF`, or `"hi"` |
| Load shedding | Rejecting some requests early under overload to keep serving the rest well |
| Machine code | Instructions in the CPU's own language (just bytes) |
| Map (Go) | A built-in hash table from keys to values; iteration order is unspecified |
| Memory model | The rules for when a write in one goroutine is guaranteed visible to a read in another |
| Method set | The methods callable on a type; decides which interfaces it satisfies |
| Migration (database) | A versioned, ordered change to a database schema |
| Module | A versioned collection of packages defined by a go.mod file |
| Mutex | A lock giving one goroutine at a time exclusive access (sync.Mutex) |
| NaN | "Not a Number": special float value for undefined results like 0.0/0.0 |
| Nil | The zero value for pointers, slices, maps, channels, functions, and interfaces |
| Nil interface vs interface holding nil | An interface is nil only if both its type and value are nil |
| Observability | Understanding a system's internal state from logs, metrics, and traces |
| Octal | Base-8 system; Go literals start with `0o` (or an old-style leading `0`) |
| OpenStreetMap (OSM) | A free, editable world map; data as nodes, ways, relations, and tags |
| Outbox pattern | Writing an event to a database table in the same transaction as the change, then publishing it |
| Package | A directory of Go files compiled together; the unit of encapsulation |
| Panic | Go's name for a serious run-time error that stops the program unless handled |
| Parallelism | Actually executing multiple tasks at the same instant on multiple cores |
| PGO (profile-guided optimisation) | Compiling with a CPU profile so the compiler optimises hot paths |
| Pointer | A value holding the address of another value |
| Polyline (encoded) | Google's compact string encoding of a sequence of coordinates |
| pprof | Go's profiler and its tools for CPU, memory, goroutine, block, and mutex profiles |
| Priority queue | A collection that always yields its smallest (or largest) element next |
| Process | A program that is currently running, with its own memory given by the OS |
| Projection (map) | A mapping from the sphere to a plane, e.g. Web Mercator; always distorts something |
| R-tree | A balanced tree of bounding rectangles for indexing spatial objects |
| Race detector | Tool enabled by -race that reports data races observed at run time |
| RAM | The computer's main working memory; fast, and cleared when power goes off |
| Read replica | A database copy that serves reads; may lag behind the primary |
| Receiver | The value a method is called on; value or pointer receiver |
| Reflection | Inspecting and manipulating types and values at run time (package reflect) |
| Run time | The period when the program is actually executing |
| Rune | An int32 holding a Unicode code point |
| Saga | A sequence of local transactions with compensating actions instead of one distributed transaction |
| Scheduler (Go) | Runtime component that multiplexes goroutines onto OS threads (the G-M-P model) |
| Select | Statement waiting on several channel operations, proceeding with one that is ready |
| Semaphore | Limits how many goroutines may do something at once; often a buffered channel |
| Sharding | Splitting data across several machines by key |
| singleflight | Collapsing concurrent identical calls into one execution (golang.org/x/sync/singleflight) |
| SKIP LOCKED | PostgreSQL row-locking option that lets many workers claim different queue rows |
| Slice | A view (pointer, length, capacity) onto a backing array |
| SLO / SLI | Service level objective (target) and indicator (the measured quantity), e.g. p99 < 50 ms |
| Snapping (map matching) | Moving a coordinate to the nearest point on the road network |
| Source code | The human-readable text you write, e.g. main.go |
| SQL injection | An attack where input is executed as SQL; prevented by parameterised queries |
| Stack (memory) | Per-goroutine memory for function calls' local values; grows automatically |
| Stack trace | The list of function calls printed when a panic happens, showing where it failed |
| Struct | A composite type grouping named fields |
| Struct tag | A string attached to a struct field read by packages like encoding/json |
| Syntax error | Grammar of the code is broken (missing bracket, brace, etc.); one kind of compile error |
| System call | A request from a program to the OS, e.g. "write this text to the screen" |
| Table-driven test | A test that loops over a slice of cases with inputs and expected outputs |
| Transaction | A group of database operations that commit or roll back together (ACID) |
| Transaction isolation level | How much concurrent transactions can see of each other: read committed, repeatable read, serializable |
| Type assertion | x.(T): extracts the concrete type T from an interface value |
| Type error | Code has correct grammar but mixes types wrongly, e.g. string + int |
| Type parameter | A placeholder type in a generic function or type, e.g. [T any] |
| Type switch | A switch on an interface value's dynamic type |
| UTF-8 | Variable-length encoding of Unicode in 1–4 bytes; Go source and strings use it |
| Variadic function | A function whose last parameter accepts any number of arguments (...T) |
| WaitGroup | Waits for a collection of goroutines to finish (sync.WaitGroup) |
| WGS84 | The world geodetic system used by GPS; standard lat/lon coordinates |
| Worker pool | A fixed number of goroutines processing jobs from a channel |
| Zero value | The value a variable has when declared without initialisation: 0, "", false, nil, … |

# Appendix C — Go versions and features

Go releases a new version every six months (February and August) and supports the two most recent. Go 1 code keeps compiling on every later 1.x release. The `go` line in `go.mod` sets the minimum language version a module needs, and since Go 1.21 it also controls version-dependent behaviour such as loop-variable semantics (Unit 3.13).

This table lists the features this book relies on, and the version that introduced each one. Features from **Go 1.25 onwards** are marked in the text, and the book's code examples for them were not compiled (see *Toolchain and versions* in the front matter).

| Version | Released | Features used or discussed in this book |
| --- | --- | --- |
| 1.13 | 2019 | `0o` octal and `0b` binary literals, digit separators `1_000`; error wrapping with `%w`, `errors.Is`, `errors.As` |
| 1.16 | 2021 | `embed` package; `io/fs`; `os.ReadFile` / `os.WriteFile` |
| 1.18 | 2022 | **Generics** (type parameters, constraints); fuzzing in `go test`; workspaces (`go.work`) |
| 1.20 | 2023 | `errors.Join` and multiple `%w`; profile-guided optimisation (preview); `context.WithCancelCause` |
| 1.21 | 2023 | `min`, `max`, `clear` built-ins; `slices`, `maps`, `cmp` packages; `log/slog`; PGO generally available; toolchain management (`toolchain` line) |
| 1.22 | 2024 | **Per-iteration loop variables**; `range` over integers; `net/http.ServeMux` method and wildcard patterns; `math/rand/v2` |
| 1.23 | 2024 | **Range-over-func iterators** (`iter.Seq`, `iter.Seq2`); iterator functions in `slices` and `maps`; `unique` package; timer and ticker changes (unbuffered, collectable) |
| 1.24 | 2025 | Generic type aliases; `testing.B.Loop`; `os.Root`; `weak` package; Swiss-table map implementation; `omitzero` JSON tag; `tool` directives in `go.mod`; `crypto/mlkem` — **the version used to verify this book** |
| 1.25 | 2025 | Container-aware `GOMAXPROCS`; `testing/synctest`; `sync.WaitGroup.Go`; `encoding/json/v2` experiment; trace flight recorder; Green Tea GC experiment |
| 1.26 | 2026 | `new(expr)`; `errors.AsType`; Green Tea garbage collector by default; `crypto/hpke`; `go fix` modernisers |
| 1.27 | 2026 | Generic methods (not allowed in interfaces, and cannot satisfy interfaces); promoted-field keys in struct literals; `encoding/json` backed by the v2 engine, plus `encoding/json/v2`; `uuid` package; `crypto/mldsa`; goroutine-leak profile; `synctest.Sleep`; `httptest.NewTestServer`; `strings.CutLast` / `bytes.CutLast`; `Rand.N` |

To learn what a new release changes, read its release notes at **go.dev/doc/devel/release** and run `go fix ./...` — since Go 1.26 it applies the modernisers that rewrite old idioms into new ones.
