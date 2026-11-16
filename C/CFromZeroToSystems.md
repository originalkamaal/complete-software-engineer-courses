---
title: "C from Zero to Systems"
subtitle: "A complete course in the C programming language — from how a program runs to a crash-safe database server"
lang: en
---

# Preface

C is fifty years old and still runs the world: operating-system kernels, databases, language runtimes, network stacks, embedded firmware, and the libraries underneath almost every other language. Learning it well changes how you understand every other kind of programming, because C hides almost nothing about the machine.

C also forgives almost nothing. A program can compile without warnings, pass its tests, and still contain undefined behaviour that corrupts memory in production or becomes a security hole. So this book teaches C the way professionals actually write it: precise knowledge of the rules, plus the habit of checking everything with tools — compiler warnings, sanitizers, Valgrind, debuggers, static analysers, fuzzers, profilers, and measurements.

The book starts below the language, with what a compiler, linker, and running process are. It covers the core language rule by rule, then pointers, memory, program structure, undefined behaviour and debugging, the standard library, data structures and algorithms, POSIX systems programming, concurrency, networking, performance, and security. It ends with a capstone: a networked key-value database with a write-ahead log, snapshots, and crash recovery, tested by killing it mid-write.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the C standard, a particular implementation, the operating system, or the hardware.
2. **Evidence.** Every program in this book was compiled and run, and every output shown is real — including warnings, sanitizer reports, crashes, and benchmark numbers. When the first version of an example was wrong (and several were), the book shows what went wrong and how the tools revealed it, because that is how real C development goes.
3. **Order.** Nothing is used before it is taught, and later chapters refer back by unit number.

# How to use this book

## Who it is for

- **Beginners** who are willing to work carefully: start at Chapter 0 and read in order. No previous programming is assumed, though it helps.
- **Programmers from other languages:** read Chapters 0–2 quickly, but do every *Deep & Tricky* entry and *Mastery check*; C's rules differ from what you know in exactly those places.
- **Working C programmers:** use Appendix A to test yourself, then focus on Chapters 5, 9, 11, and 12.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | The rules, with small complete programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, each labelled with its source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Common mistakes** | What people actually get wrong (in some units). |
| **Mastery check** | *Explain*, *Predict* (decide the output before running), *Write* (code it). Answers to prediction questions are at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a specification, a complete reference solution with real output, and a review checklist. Try the project yourself first.

## How to study

1. **Type the programs and run them.** Compile every one with the warning flags shown. Compare your output with the book's.
2. **Predict before you run.** When you are wrong, find out why before moving on.
3. **Break things on purpose.** Remove a bounds check, free something twice, forget a `close` — and watch which tool catches it.
4. **Use the tools from day one:** `-Wall -Wextra`, `-fsanitize=address,undefined`, and GDB. Chapter 0 shows how; Chapter 5 explains them fully.
5. **Revise with Appendix A,** which lists every DT and P entry by title.

## How to read the labels

| Label | Meaning |
| --- | --- |
| *Guarantee* / *Guarantee: C standard* | Required by the ISO C standard. True on every conforming implementation. |
| *Guarantee: UB* | The standard says the behaviour is undefined: no requirements at all. |
| *Library behaviour* | Specified by the C library documentation (glibc here) or POSIX rather than by the language. |
| *POSIX*, *OS behaviour* | Specified by POSIX or by the operating system (Linux here), not by ISO C. |
| *Toolchain behaviour*, *Compiler behaviour*, *Optimiser behaviour* | What GCC, Clang, or the linker do. Stable in practice; may change between versions. |
| *Implementation detail* | How today's implementation works internally. Useful for performance; never rely on it for correctness. |
| *Hardware fact*, *TCP behaviour*, *IEEE 754 behaviour* | True because of the machine, the protocol, or the floating-point standard. |
| *Practice*, *Design fact*, *Security fact* | Professional practice and experience, not rules of the language. |

## Toolchain and versions

All code was compiled and run on **Linux x86-64 (Ubuntu 24.04)** with:

- **GCC 13.3** (default compiler) and **GCC 14.2** (for C23 examples),
- **Clang 18** (sanitizers, MemorySanitizer, libFuzzer, `clang-tidy`),
- **glibc 2.39**, **GNU Make 4.3**, **CMake 3.28**, **NASM 2.16**, **GDB 15**, **Valgrind 3.22**, **cppcheck 2.13**, **perf** (Linux 6.8 tools).

The language baseline is **C17** (`-std=c17`); **C23** features are marked and compiled with `gcc-14 -std=c23`. POSIX programs (Chapters 8–10, 13) need Linux or another Unix; most also work on macOS and the BSDs, except Linux-specific APIs such as `epoll` (macOS and BSDs use `kqueue`). On Windows, use WSL2 to follow the book.

Measurements (Chapters 7, 9, 10, 11, 13) were taken on a 2-core virtual machine (Intel Xeon at 2.1 GHz). Your numbers will differ; compare the ratios and shapes.

A few things could not be demonstrated directly and the text says so where they arise: hardware performance counters were unavailable in the virtual machine (Cachegrind is used instead), and durability against **power loss** (as opposed to process crashes) was not tested in Chapter 13.

## Setting up

1. Install a compiler and tools. On Debian/Ubuntu: `sudo apt install build-essential clang gdb valgrind cmake`. On Fedora: `sudo dnf install gcc clang gdb valgrind cmake make`. On macOS: `xcode-select --install` (Clang; use a Linux VM or container for Valgrind, `epoll`, and `perf`).
2. Check: `gcc --version`, `clang --version`, `gdb --version`.
3. Use any editor with C support through **clangd** (VS Code, CLion, Neovim, Emacs, Zed).
4. Create a working folder with one subfolder per chapter, and keep a journal of the predictions you got wrong.

# Roadmap

| Chapter | Title | You will be able to… |
| --- | --- | --- |
| 0 | How C Becomes a Running Program | follow source code through preprocessing, compiling, assembling, and linking into a process; read binary and hex; set up warnings and sanitizers |
| 1 | Core Language | apply C's exact rules for types, conversions, operators, evaluation order, control flow, and functions |
| 2 | Pointers, Arrays, and Strings | use pointers, pointer arithmetic, arrays, strings, `const`, and function pointers safely |
| 3 | Memory and Data Layout | manage dynamic memory with clear ownership; design structs, unions, and opaque types; predict layout |
| 4 | Program Structure, the Preprocessor, and Builds | organise multi-file programs, use the preprocessor safely, and build with Make, CMake, and libraries |
| 5 | Undefined Behaviour, Debugging, and Testing | recognise UB; use warnings, static analysis, sanitizers, Valgrind, GDB; test and measure coverage |
| 6 | The Standard Library and I/O | use streams, formatted and binary I/O, files, time, and modern C11–C23 features |
| 7 | Data Structures and Algorithms in C | implement lists, hash tables, trees, heaps, sorting, and graphs, and write generic code |
| 8 | POSIX Systems Programming | use file descriptors, processes, pipes, signals, `mmap`, and `dlopen`; write a shell |
| 9 | Concurrency | write correct threaded code with mutexes, condition variables, and atomics; measure speed-up |
| 10 | Network Programming | write TCP/UDP clients and servers and an `epoll` event-driven HTTP server |
| 11 | Performance and the Machine | read assembly, use the ABI, exploit caches, branch prediction, and SIMD, and profile |
| 12 | Security and Robustness | understand memory-safety vulnerabilities and mitigations, write secure code, and fuzz |
| 13 | Capstone: A Persistent Key-Value Database Server | design, build, crash-test, load-test, and defend a durable database server |
| A | Index of Deep & Tricky entries and Patterns | revise every entry by title |
| B | Glossary | look up any term |
| C | C standards and toolchain support | see which standard introduced each feature, and compiler support |

Chapters 0–6 are the **foundation**; read them in order. Chapters 7–12 can be read in any order after Chapter 6 (Chapter 10 uses Chapter 8, and Chapter 9's project uses Chapter 7's hash table). Chapter 13 uses the whole book.

# Chapter 0 — How C Becomes a Running Program

*Goal: understand exactly what happens between typing C code and a process running on a CPU, so that nothing later feels like magic.*

C is the language closest to the machine that most programmers will ever use for whole programs. That closeness is why operating systems, databases, language runtimes, embedded firmware, and network stacks are still written in it — and why C punishes vague understanding. A Python or Go programmer can be productive without knowing what a linker does; a C programmer meets the linker on the first day of a multi-file project and meets undefined behaviour on the first day of pointer arithmetic.

This chapter builds the foundation: what C is, how the toolchain turns text into an executable in four stages, what an object file and a process look like, how numbers are stored in bits, and how to ask the compiler for every warning it can give. Everything here was run on Linux x86-64 with GCC 13.3 and Clang 18; outputs shown are real.

**What you will be able to do after this chapter**

- Compile, run, and inspect C programs with GCC or Clang, and read their exit status.
- Stop the compiler after each of its four stages and explain what each produced.
- Tell a compile error from a link error from a run-time failure — instantly, from the message.
- Read binary and hexadecimal, and explain two's complement and endianness.
- Describe the memory layout of a running process and point to where each kind of variable lives.
- Explain what *undefined behaviour* is and why it is the most important idea in C.

## Topic 0A — C and its toolchain

## Unit 0.1 — What C is

**One-line summary:** C is a small, compiled, statically typed language, standardised by ISO, that describes programs in terms of an *abstract machine*; compilers translate it into native machine code with almost no hidden runtime, which is why it is fast, portable, and unforgiving.

### A short history that explains the language

- **1972:** Dennis Ritchie creates C at Bell Labs to rewrite the Unix operating system, which had been written in assembly. C's design goal: be close enough to the hardware to write an OS, but portable across machines.
- **1978:** Kernighan and Ritchie publish *The C Programming Language* ("K&R C").
- **1989/1990:** ANSI and then ISO standardise C — **C89/C90**. From here on, "C" means "the ISO standard", not "what my compiler does".
- **1999 — C99:** `//` comments, `long long`, `stdint.h` fixed-width integers, `bool`, declarations anywhere in a block, variable-length arrays, designated initialisers.
- **2011 — C11:** threads and atomics, `_Generic`, `_Static_assert`, anonymous structs and unions, `aligned_alloc`.
- **2018 — C17:** a bug-fix release of C11; no new features. **This book's baseline.**
- **2024 — C23:** `bool`, `true`, `false`, `nullptr`, `constexpr` objects, `typeof`, `#embed`, binary literals `0b1010`, digit separators `1'000'000`, `[[attributes]]`, `auto` type inference for objects, `<stdbit.h>`. GCC 13+ and Clang 18 support most of it with `-std=c23` (older spelling `-std=c2x`). C23 features in this book are marked **(C23)**.

### The abstract machine

The C standard does not describe your CPU. It describes an **abstract machine**: objects with values, stored in bytes; expressions evaluated with certain rules; side effects (writing a variable, writing a file) that happen in a certain order. A compiler's only obligation is that the **observable behaviour** of your program — what it writes to files and the terminal, its accesses to `volatile` objects — is the same as the abstract machine's would be. Everything else may be changed: variables kept in registers or removed completely, loops rewritten, function calls inlined.

That freedom is why C is fast. It is also why the next idea matters so much.

### Undefined behaviour

For some operations the standard says the behaviour is **undefined**: it places *no requirements at all* on what happens. Examples you will meet in this book: reading an array element past its end, using memory after `free`, overflowing a signed integer, dereferencing a null pointer, reading an uninitialised local variable in many cases, and data races between threads.

Undefined behaviour (UB) does not mean "crashes" or "gives a garbage value". It means the compiler may **assume it never happens** and optimise on that assumption. A program with UB can work in testing, fail in production, behave differently at `-O0` and `-O2`, or have entire `if` statements removed. Chapter 5 is devoted to UB; every chapter points it out where it lives.

The standard also has two milder categories:

| Category | Meaning | Example |
| --- | --- | --- |
| **Implementation-defined** | The compiler chooses and must document the choice | the size of `int`; whether `char` is signed |
| **Unspecified** | The compiler chooses among options and need not document or be consistent | the order in which `f() + g()` calls `f` and `g` |
| **Undefined** | No requirements at all | `INT_MAX + 1`, `a[10]` for `int a[10]` |

### Where C runs today

Linux, Windows, and macOS kernels; most embedded systems; SQLite, PostgreSQL, Redis; the CPython interpreter and most language runtimes; Git, curl, OpenSSL, nginx; game engines' low layers. Learning C also means you can read the code underneath almost everything else.

### Deep & Tricky

**DT-1. "It worked when I ran it" proves nothing in C.** *(Guarantee: C standard, UB definition)* A program with undefined behaviour is not required to fail. Its behaviour can change with the compiler version, the optimisation level, the surrounding code, or the phase of the moon. The only defences are knowing the rules, enabling warnings, and running tools that detect UB at run time (sanitizers, Unit 0.7 and Chapter 5).

**DT-2. The standard describes C; your compiler implements C plus extensions.** *(Toolchain behaviour)* GCC and Clang accept many non-standard extensions by default (statement expressions, `__attribute__`, zero-length arrays, nested functions in GCC). Code that relies on them is not portable. `-std=c17 -pedantic` makes the compiler warn about them; `-std=gnu17` (GCC's default dialect) allows them silently.

### Mastery check

1. **Explain.** What is the abstract machine, and why does the compiler only need to preserve "observable behaviour"?
2. **Explain.** What is the difference between implementation-defined, unspecified, and undefined behaviour? Give an example of each.

## Unit 0.2 — Your first program

**One-line summary:** A C program is one or more source files with exactly one `main` function; you compile it into an executable, run the executable, and the value `main` returns becomes the process's **exit status**.

### Hello, C

```c
// file: hello.c
#include <stdio.h>

int main(void) {
    printf("Hello, C!\n");
    return 0;
}
```

Line by line:

- `#include <stdio.h>` is a **preprocessor directive**: before compilation, it pastes in the contents of the header `stdio.h`, which *declares* `printf` (tells the compiler its name and type). The *definition* — the machine code of `printf` — lives in the C standard library, which the linker adds later.
- `int main(void)` defines the function where execution starts. `void` in the parentheses means "takes no arguments". (`int main()` without `void` means "unspecified arguments" before C23 — a legacy form; always write `void`.)
- `printf("Hello, C!\n")` writes formatted text to standard output. `\n` is a newline character; without it, the shell prompt appears on the same line.
- `return 0;` ends `main` with exit status 0, which by convention means success.

Compile and run:

```console
$ gcc hello.c -o hello
$ ./hello
Hello, C!
```

`gcc hello.c -o hello` runs the whole toolchain and writes the executable `hello`. Without `-o hello`, GCC names the output `a.out` (a name dating from the 1970s: "assembler output"). `./hello` runs it; the `./` is needed because the current directory is (deliberately) not on the shell's search path.

### Exit status

```c
// file: status.c
#include <stdlib.h>

int main(void) {
    return 3;  /* any value 0-255 is visible to the shell */
}
```

```console
$ gcc status.c -o status
$ ./status; echo "exit status: $?"
exit status: 3
```

The shell variable `$?` holds the exit status of the last command. Scripts, `make`, and CI systems all use it: 0 is success, anything else is failure. `<stdlib.h>` defines the portable names `EXIT_SUCCESS` and `EXIT_FAILURE`. Two details:

- Reaching the closing `}` of `main` without a `return` is the same as `return 0;` (C99 and later; this rule applies **only** to `main`).
- On POSIX systems only the low 8 bits reach the parent: `return 256;` shows up as 0.

### GCC and Clang

Both are excellent, free, and accept almost the same options. GCC is the default on most Linux distributions; Clang is the default on macOS (where `gcc` is actually Clang in disguise) and powers many analysis tools. Seeing both compilers' warnings for the same code is a cheap way to catch more bugs; this book uses GCC by default and Clang where its tools (sanitizers, `clang-tidy`) or messages are better.

### Mastery check

1. **Predict.** What exit status does the shell report for a `main` that executes `return 300;`?
2. **Write.** A program that returns `EXIT_FAILURE`; check its status with `echo $?`.

## Unit 0.3 — The four stages of a build

**One-line summary:** `gcc file.c -o prog` runs four programs in sequence: the **preprocessor** (text substitution: `#include`, `#define`), the **compiler proper** (C to assembly), the **assembler** (assembly to machine code in an object file), and the **linker** (object files plus libraries to an executable). You can stop after any stage and look.

```text
 hello.c ──preprocess (cpp, gcc -E)──▶ hello.i
         ──compile    (cc1, gcc -S)──▶ hello.s    (assembly text)
         ──assemble   (as,  gcc -c)──▶ hello.o    (object file: machine code + symbols)
         ──link       (ld)─────────▶ hello      (executable)
```

### Stage 1: preprocessing

```c
// file: stages.c
#include <stdio.h>
#define GREETING "Hello"
#define SQUARE(x) ((x) * (x))

int main(void) {
    int n = 7;
    printf("%s, square of %d is %d\n", GREETING, n, SQUARE(n));
    return 0;
}
```

```console
$ gcc -E stages.c -o stages.i
$ wc -l stages.i
823 stages.i
$ tail -n 5 stages.i
int main(void) {
    int n = 7;
    printf("%s, square of %d is %d\n", "Hello", n, ((n) * (n)));
    return 0;
}
```

The preprocessor pasted in `stdio.h` and everything it includes — hundreds of lines of declarations — and replaced `GREETING` and `SQUARE(n)` by plain text. The compiler proper never sees `#define`; it sees the expanded text. That is why macros have no types, no scope, and surprising behaviour (Chapter 4).

### Stage 2: compiling to assembly

```console
$ gcc -S -O1 stages.c -o stages.s
$ grep -A12 '^main:' stages.s
main:
.LFB23:
	.cfi_startproc
	endbr64
	subq	$8, %rsp
	.cfi_def_cfa_offset 16
	movl	$49, %r8d
	movl	$7, %ecx
	leaq	.LC0(%rip), %rdx
	leaq	.LC1(%rip), %rsi
	movl	$2, %edi
	movl	$0, %eax
	call	__printf_chk@PLT
```

Even if you cannot read x86-64 assembly yet (Chapter 11 teaches it), two things stand out. The multiplication is gone: the compiler computed `SQUARE(7)` = 49 at compile time (`$49`). And the output is a `call` to a function this file does not define — something the linker must find. It is `__printf_chk`, not `printf`: Ubuntu's GCC enables `_FORTIFY_SOURCE` when optimising, which swaps some library calls for checked versions that detect certain buffer overflows. Your distribution may show plain `printf`.

### Stage 3: assembling an object file

```console
$ gcc -c stages.c -o stages.o
$ file stages.o
stages.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
$ nm stages.o
0000000000000000 T main
                 U printf
```

An **object file** contains machine code and a **symbol table**. `nm` lists symbols: `T main` means "main is defined here, in the Text (code) section"; `U printf` means "printf is Undefined here — used but not defined". The object file is not runnable: it has unresolved references.

### Stage 4: linking

```console
$ gcc stages.o -o stages
$ file stages
stages: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=21061f652e9695b4c1764385c0f4045f8e1b29b4, for GNU/Linux 3.2.0, not stripped
$ ./stages
Hello, square of 7 is 49
```

The linker combines object files and libraries, resolves every `U` symbol to a definition (here `printf` from the C library, `libc.so.6`), assigns final addresses, and writes an executable. "Dynamically linked" means `printf`'s code is not copied in; the dynamic loader maps `libc.so.6` into memory when the program starts (Chapter 4 covers static and shared libraries).

### Seeing it all

`gcc -v` prints every command it runs, and `-save-temps` keeps the intermediate files: try `gcc -save-temps stages.c -o stages` and list the directory.

### Deep & Tricky

**DT-3. The compiler sees one file at a time.** *(Guarantee: C translation model)* Each `.c` file, after preprocessing, is a **translation unit** compiled independently, with no knowledge of other `.c` files. Everything a file uses from elsewhere must be *declared* in it (usually via a header). Only the linker sees the whole program. Consequences: mismatched declarations across files are not caught by the compiler (DT-5), and most whole-program optimisations require link-time optimisation (`-flto`, Chapter 11).

**DT-4. `gcc` is a driver, not a compiler.** *(Toolchain behaviour)* The `gcc` command decides which programs to run (`cc1`, `as`, `collect2`/`ld`) based on file extensions and flags. `gcc x.o` only links; `gcc x.c` does everything; `gcc -c` stops before linking. This is why the same command compiles C, links C++ objects, or assembles `.s` files.

### Mastery check

1. **Explain.** Which stage would report each of these: a missing semicolon; `#include "nothere.h"`; calling a function declared in a header but defined nowhere?
2. **Write.** Use `-E`, `-S`, and `-c` on your own program with a macro, and find the macro's expansion, the assembly for `main`, and the `U` symbols in the object file.

## Unit 0.4 — Errors at each stage

**One-line summary:** Learn to recognise *where* an error comes from by its wording: preprocessor errors mention files and directives, compiler errors point to a line of your code, linker errors say "undefined reference" or "multiple definition" and mention no line of your code, and run-time failures happen only when the program runs.

### A compile error

```c
// file: err_compile.c
#include <stdio.h>

int main(void) {
    int x = "hello";
    printf("%d\n", x)
    return 0;
}
```

```console
$ gcc err_compile.c -o err_compile
err_compile.c: In function ‘main’:
err_compile.c:5:13: warning: initialization of ‘int’ from ‘char *’ makes integer from pointer without a cast [-Wint-conversion]
    5 |     int x = "hello";
      |             ^~~~~~~
err_compile.c:6:22: error: expected ‘;’ before ‘return’
    6 |     printf("%d\n", x)
      |                      ^
      |                      ;
    7 |     return 0;
      |     ~~~~~~            
```

Two things to learn from this output. The compiler reported a **warning** for assigning a string to an `int` (in GCC 14 and later, this became an error by default) and an **error** for the missing semicolon, pointing at the *next* token, because that is where it noticed the problem. Compile errors always come with `file:line:column`. Read the first error first: later errors are often consequences of it.

### A link error

```c
// file: err_link.c
#include <stdio.h>

int helper(int);  /* declared, never defined */

int main(void) {
    printf("%d\n", helper(1));
    return 0;
}
```

```console
$ gcc -c err_link.c -o err_link.o
$ gcc err_link.o -o err_link
/usr/bin/ld: err_link.o: in function `main':
err_link.c:(.text+0xe): undefined reference to `helper'
collect2: error: ld returned 1 exit status
```

The compile step succeeded: the declaration promised that `helper` exists somewhere. Only the linker discovered that no object file or library defines it. Link errors mention symbols, not lines of your code — and often appear in a large project as a missing library (`-lm` for math functions on some systems) or a missing `.c` file in the build.

### A run-time failure

```c
// file: err_run.c
#include <stdio.h>

int main(void) {
    int *p = NULL;
    printf("about to crash\n");
    fflush(stdout);
    return *p;  /* undefined behaviour: null pointer dereference */
}
```

```console
$ gcc err_run.c -o err_run
$ ./err_run; echo "exit status: $?"
about to crash
bash: line 1:  1755 Segmentation fault      ./err_run
exit status: 139
```

The program compiled and linked cleanly and failed only when run. Linux killed it with signal 11 (`SIGSEGV`, segmentation fault); the shell reports 128 + 11 = 139. Strictly, dereferencing a null pointer is undefined behaviour, and a crash is just the most common outcome on Linux (address 0 is never mapped). Chapter 5 shows how sanitizers and GDB pinpoint the faulting line.

### Deep & Tricky

**DT-5. Mismatched declarations across files are invisible to both compiler and linker.** *(Guarantee: UB; toolchain behaviour)* If `a.c` declares `double f(double);` and `b.c` defines `int f(int x)`, each file compiles cleanly, and the linker matches the names (C symbols carry no type information). Calling `f` is undefined behaviour and in practice reads garbage. The cure is a **single header** that declares `f`, included both by callers *and* by the file that defines it, so the compiler checks the definition against the declaration (Chapter 4).

### Common mistakes

- Fixing the last error message first. Fix the first one, recompile, repeat.
- Assuming "undefined reference" means a missing `#include`. Headers declare; the linker needs a *definition*: a missing source file or library.

### Mastery check

1. **Predict.** A program compiles with `gcc -c` but `gcc prog.o -o prog` fails with "multiple definition of `count'". What happened? (Hint: two files.)
2. **Explain.** Why is a null-pointer dereference not reported at compile time?

## Topic 0B — Data in the machine

## Unit 0.5 — Bits, bytes, and how C stores numbers

**One-line summary:** Memory is a sequence of **bytes** (8 bits each on every machine you will use); each C type occupies a fixed number of bytes reported by `sizeof`; integers are stored in binary, negative integers in **two's complement**, and multi-byte values in an order called **endianness**.

### Binary and hexadecimal

A bit is 0 or 1. With n bits there are 2ⁿ patterns: a byte has 256 (0–255). Binary numbers are long, so programmers write them in **hexadecimal** (base 16, digits 0–9 and a–f), where each hex digit is exactly four bits:

| Decimal | Binary | Hex |
| --- | --- | --- |
| 10 | 0000 1010 | 0x0A |
| 255 | 1111 1111 | 0xFF |
| 4096 | 0001 0000 0000 0000 | 0x1000 |

C writes hex literals with `0x`, octal with a leading `0` (a trap: `010` is 8 — DT-6), and binary with `0b` (**C23**; a GCC/Clang extension before that).

### Sizes

```c
// file: sizes.c
#include <limits.h>
#include <stdint.h>
#include <stdio.h>

int main(void) {
    printf("char      %zu byte(s), CHAR_BIT = %d\n", sizeof(char), CHAR_BIT);
    printf("short     %zu\n", sizeof(short));
    printf("int       %zu, range %d .. %d\n", sizeof(int), INT_MIN, INT_MAX);
    printf("long      %zu\n", sizeof(long));
    printf("long long %zu\n", sizeof(long long));
    printf("pointer   %zu\n", sizeof(void *));
    printf("int32_t   %zu, uint64_t %zu\n", sizeof(int32_t), sizeof(uint64_t));
    printf("float %zu, double %zu, long double %zu\n",
           sizeof(float), sizeof(double), sizeof(long double));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall sizes.c -o sizes
$ ./sizes
char      1 byte(s), CHAR_BIT = 8
short     2
int       4, range -2147483648 .. 2147483647
long      8
long long 8
pointer   8
int32_t   4, uint64_t 8
float 4, double 8, long double 16
```

These are the sizes on 64-bit Linux (the **LP64** model: `long` and pointers are 64 bits). On 64-bit Windows (**LLP64**), `long` is 4 bytes. The standard only guarantees minimum ranges: `char` at least 8 bits, `int` at least 16, `long` at least 32, `long long` at least 64. When an exact size matters — file formats, network protocols, hardware registers — use the fixed-width types from `<stdint.h>`: `int8_t` … `int64_t`, `uint8_t` … `uint64_t`. `%zu` is the `printf` format for `size_t`, the unsigned type that `sizeof` produces.

### Unsigned integers wrap; two's complement for signed

An n-bit unsigned integer holds 0 to 2ⁿ−1 and arithmetic on it is defined **modulo 2ⁿ**: `UINT_MAX + 1u` is 0. That is a guarantee.

Signed integers on every modern machine use **two's complement**, and since C23 the standard requires it: the top bit has weight −2ⁿ⁻¹ instead of +2ⁿ⁻¹. For 8 bits, `1000 0000` is −128 and `1111 1111` is −1. To negate: invert all bits and add 1.

```c
// file: twos.c
#include <limits.h>
#include <stdint.h>
#include <stdio.h>

int main(void) {
    int8_t a = -1, b = -128;
    printf("-1   as bits: 0x%02X\n", (uint8_t)a);
    printf("-128 as bits: 0x%02X\n", (uint8_t)b);

    unsigned u = UINT_MAX;
    u = u + 1u;                          /* defined: wraps to 0 */
    printf("UINT_MAX + 1u = %u\n", u);

    uint8_t big = 200;
    int8_t conv = (int8_t)big;          /* implementation-defined before C23; -56 on GCC */
    printf("(int8_t)200 = %d\n", conv);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall twos.c -o twos
$ ./twos
-1   as bits: 0xFF
-128 as bits: 0x80
UINT_MAX + 1u = 0
(int8_t)200 = -56
```

But **signed overflow is undefined behaviour** (DT-7). The machine would wrap; the compiler is allowed to assume it never happens.

### Endianness

A 4-byte `int` occupies four consecutive addresses. **Little-endian** machines (x86, almost all ARM configurations) store the least significant byte first; **big-endian** machines (network byte order, some older and embedded CPUs) store the most significant byte first:

```c
// file: endian.c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint32_t x = 0x11223344;
    unsigned char *bytes = (unsigned char *)&x;  /* view the object as raw bytes: allowed */
    for (int i = 0; i < 4; i++)
        printf("byte %d at %p: 0x%02x\n", i, (void *)&bytes[i], bytes[i]);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall endian.c -o endian
$ ./endian
byte 0 at 0x7ffecaec04d8: 0x44
byte 1 at 0x7ffecaec04d9: 0x33
byte 2 at 0x7ffecaec04da: 0x22
byte 3 at 0x7ffecaec04db: 0x11
```

The lowest address holds `0x44`: this machine is little-endian. (The addresses change from run to run: Unit 0.6.) Viewing any object through an `unsigned char *` is explicitly allowed by the standard — it is how you inspect representations, copy memory, and write binary files. Endianness matters when bytes leave the machine: files and network protocols must define a byte order and convert explicitly (Chapters 6 and 10).

### Deep & Tricky

**DT-6. A leading zero makes an integer literal octal.** *(Guarantee: C standard)* `int perms = 0755;` is octal (493 decimal), which is what `chmod` wants — but `int code = 010;` is 8. Zero-padding numbers for alignment in a table silently changes their values. Unlike Go or Python, C (up to and including C23) has no `0o` prefix to make octal explicit; it is being added in the next revision of the standard, so for now the leading zero is the only spelling.

**DT-7. Signed integer overflow is undefined; unsigned overflow wraps.** *(Guarantee: C standard)* `INT_MAX + 1` is undefined behaviour even though the CPU would wrap to `INT_MIN`. Compilers use this: `x + 1 > x` may be compiled to `true` for a signed `x`, and loops with signed counters are optimised assuming they cannot wrap. Unsigned arithmetic is defined modulo 2ⁿ. C23 mandated two's complement *representation* but kept overflow undefined. Use `-fsanitize=undefined` to catch overflow at run time, and C23's `ckd_add` (`<stdckdint.h>`) or GCC/Clang's `__builtin_add_overflow` to check it.

**DT-8. Converting an out-of-range value to a signed type was implementation-defined until C23.** *(Guarantee, version-dependent)* `(int8_t)200` gives −56 on every two's-complement compiler (GCC and Clang document this), and C23 now requires wrapping. Converting an out-of-range *floating-point* value to an integer type, though, is **undefined**: `(int)1e20` can produce anything.

### Mastery check

1. **Predict.** What does `printf("%d\n", 010 + 0x10);` print?
2. **Predict.** In 8-bit two's complement, what bit pattern represents −2? What value does `1000 0001` represent?
3. **Write.** A function `void dump(const void *p, size_t n)` that prints the bytes of any object in hex; use it on an `int`, a `double`, and a string.

## Unit 0.6 — The memory of a running process

**One-line summary:** A running program (a **process**) has its own virtual address space divided into regions: **text** (machine code), **data** and **bss** (global and `static` variables), the **heap** (memory from `malloc`), and the **stack** (local variables and function calls). Where a variable lives decides how long it lives.

```c
// file: layout.c
#include <stdio.h>
#include <stdlib.h>

int initialised_global = 42;     /* .data: has a non-zero initial value in the file */
int zero_global;                 /* .bss: zero-initialised, takes no space in the file */
static const char message[] = "constant";  /* .rodata: read-only data */

int main(void) {
    int local = 1;                            /* stack */
    static int counter = 5;                   /* .data, despite being inside main */
    int *heap = malloc(sizeof *heap);         /* heap */
    if (heap == NULL)
        return 1;

    printf("code    main               %p\n", (void *)main);
    printf("rodata  message            %p\n", (void *)message);
    printf("data    initialised_global %p\n", (void *)&initialised_global);
    printf("data    counter (static)   %p\n", (void *)&counter);
    printf("bss     zero_global        %p\n", (void *)&zero_global);
    printf("heap    *heap              %p\n", (void *)heap);
    printf("stack   local              %p\n", (void *)&local);
    free(heap);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall layout.c -o layout
$ ./layout
code    main               0x55c27a5df1a9
rodata  message            0x55c27a5e0008
data    initialised_global 0x55c27a5e2010
data    counter (static)   0x55c27a5e2014
bss     zero_global        0x55c27a5e201c
heap    *heap              0x55c2a05572a0
stack   local              0x7ffe3e4d532c
$ ./layout | tail -n 2
heap    *heap              0x563b60a6a2a0
stack   local              0x7fffbf4086bc
$ size layout
   text	   data	    bss	    dec	    hex	filename
   2186	    632	      8	   2826	    b0a	layout
```

(Converting a function pointer like `main` to `void *` for printing is not strictly portable C, but POSIX requires it to work; `-pedantic` warns about it.)

What the output shows:

- Code, read-only data, `.data`, and `.bss` sit together at addresses near the start of the executable's mapping; the heap is above them; the stack is far away at the top of the user address space (on x86-64 Linux, near `0x7ff…`).
- Running twice gives **different addresses**. Linux uses **address space layout randomisation (ASLR)** so attackers cannot predict where code and data live (Chapter 12).
- `size` reports the sizes of the text, data, and bss sections. `zero_global` takes no space in the file; the loader allocates and zeroes it.

### Storage duration in one table

| Where declared | Storage duration | Lives in | Initial value if not initialised |
| --- | --- | --- | --- |
| Outside any function, or with `static` | **static**: the whole program run | `.data` / `.bss` | zero |
| Inside a function, no `static` | **automatic**: until the block ends | stack (or registers) | **indeterminate** — reading it is a bug |
| `malloc`/`calloc`/`realloc` | **allocated**: until `free` | heap | indeterminate (`malloc`), zero (`calloc`) |
| `_Thread_local` / `thread_local` | **thread**: the thread's lifetime | thread-local storage | zero |

Chapter 3 is built on this table.

### The stack in action

Each function call gets a **stack frame** holding its local variables, saved registers, and the return address. Returning pops the frame; its memory is reused by the next call. This is why returning a pointer to a local variable is a bug (DT-9), and why deep recursion can overflow the stack (Linux's default main-thread stack is 8 MB; check with `ulimit -s`).

### Deep & Tricky

**DT-9. Returning the address of a local variable creates a dangling pointer.** *(Guarantee: UB)* After a function returns, its automatic variables no longer exist; a pointer to one is **dangling**, and using it is undefined behaviour. It often *appears* to work, because the bytes are still there until the next call overwrites them. GCC warns (`-Wreturn-local-addr`, on by default) only in the simplest cases.

**DT-10. Uninitialised automatic variables do not contain zero.** *(Guarantee: indeterminate value; UB in many cases)* `int x; printf("%d", x);` prints whatever was in that stack slot or register — or, because reading an uninitialised variable whose address is never taken is undefined behaviour, the optimiser may do something stranger. Globals and `static` variables are zero-initialised; locals are not. Initialise every variable at its declaration.

### Mastery check

1. **Explain.** Why does `zero_global` not make the executable file larger, while a large initialised global array does?
2. **Predict.** A function declares `static int calls = 0;` and increments it each time it is called. After three calls, what is its value? Where is it stored?
3. **Write.** A recursive function that prints the address of a local variable at each depth; observe the stack growing towards lower addresses.

## Topic 0C — A professional setup

## Unit 0.7 — Warnings, standards, and sanitizers

**One-line summary:** Always compile with a chosen standard, every useful warning, and debug information; during development also enable the **sanitizers**, which detect memory errors and undefined behaviour at run time. These flags catch more bugs than any amount of careful reading.

### The default command is too quiet

```c
// file: quiet.c
#include <stdio.h>

int sum(const int *a, int n) {
    int total;                       /* bug 1: not initialised */
    for (int i = 0; i <= n; i++)     /* bug 2: off by one */
        total += a[i];
    return total;
}

int main(void) {
    int v[4] = {1, 2, 3, 4};
    unsigned big = 3000000000u;
    if (big > -1)                    /* bug 3: -1 becomes UINT_MAX */
        printf("big is greater than -1\n");
    printf("sum = %d\n", sum(v, 4));
    printf("%d\n", "oops");          /* bug 4: wrong format */
    return 0;
}
```

```console
$ gcc quiet.c -o quiet
quiet.c: In function ‘main’:
quiet.c:17:14: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘char *’ [-Wformat=]
   17 |     printf("%d\n", "oops");          /* bug 4: wrong format */
      |             ~^     ~~~~~~
      |              |     |
      |              int   char *
      |             %s
$ ./quiet
sum = 1655124826
1887043598
```

Plain `gcc` reported only bug 4 (format warnings are on by default) and the program printed a garbage sum — and silently skipped the "greater than −1" line. Now the professional command:

```console
$ gcc -std=c17 -Wall -Wextra -pedantic -O2 -g quiet.c -o quiet
quiet.c: In function ‘main’:
quiet.c:14:13: warning: comparison of integer expressions of different signedness: ‘unsigned int’ and ‘int’ [-Wsign-compare]
   14 |     if (big > -1)                    /* bug 3: -1 becomes UINT_MAX */
      |             ^
quiet.c:17:14: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘char *’ [-Wformat=]
   17 |     printf("%d\n", "oops");          /* bug 4: wrong format */
      |             ~^     ~~~~~~
      |              |     |
      |              int   char *
      |             %s
quiet.c: In function ‘sum’:
quiet.c:7:15: warning: ‘total’ may be used uninitialized [-Wmaybe-uninitialized]
    7 |         total += a[i];
      |         ~~~~~~^~~~~~~
quiet.c:5:9: note: ‘total’ was declared here
    5 |     int total;                       /* bug 1: not initialised */
      |         ^~~~~
In function ‘sum’,
    inlined from ‘main’ at quiet.c:16:5:
quiet.c:7:15: warning: ‘total’ is used uninitialized [-Wuninitialized]
    7 |         total += a[i];
      |         ~~~~~~^~~~~~~
quiet.c: In function ‘main’:
quiet.c:5:9: note: ‘total’ was declared here
    5 |     int total;                       /* bug 1: not initialised */
      |         ^~~~~
```

- `-std=c17` — compile the standard language, not a dialect.
- `-Wall -Wextra` — the two main warning groups (despite the name, `-Wall` is not all warnings). Here `-Wextra` found the signed/unsigned comparison (bug 3), and analysing with optimisation (`-O2`) let GCC notice that `total` is used uninitialised (bug 1). Some warnings only appear when optimisation is on, because they rely on the optimiser's data-flow analysis.
- `-pedantic` — warn about non-standard extensions.
- `-g` — include debug information for GDB and readable sanitizer reports.

Add `-Werror` in CI so warnings cannot be ignored, and consider `-Wconversion -Wshadow -Wformat=2 -Wundef` for stricter projects.

### Sanitizers

The off-by-one read (bug 2) produced no warning. **AddressSanitizer** finds it at run time:

```console
$ gcc -std=c17 -g -fsanitize=address,undefined -fno-omit-frame-pointer quiet.c -o quiet_asan 2>/dev/null
$ ./quiet_asan 2>&1 | grep -E 'ERROR|READ|#[0-3] |located' | head -n 8
==1802==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fdbbc300030 at pc 0x55b677c7136d bp 0x7ffd804e42b0 sp 0x7ffd804e42a0
READ of size 4 at 0x7fdbbc300030 thread T0
    #0 0x55b677c7136c in sum quiet.c:7
    #1 0x55b677c7156b in main quiet.c:16
    #2 0x7fdbbe82a1c9 in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #3 0x7fdbbe82a28a in __libc_start_main_impl ../csu/libc-start.c:360
Address 0x7fdbbc300030 is located in stack of thread T0 at offset 48 in frame
    #0 0x55b677c713e2 in main quiet.c:11
```

AddressSanitizer reports a **stack-buffer-overflow**: a 4-byte read just past the array `v`, in `sum` at `quiet.c:7` (line numbers in this book count the `// file:` line, so they match the listings), called from `main`. The sanitized program runs about 2× slower and uses more memory — a cheap price during development and testing. Chapter 5 covers ASan, UBSan, MSan, TSan, and Valgrind in depth.

### A Makefile for the rest of the chapter

From Chapter 4 onwards, builds use `make`. For now, a shell alias is enough:

```sh
alias cc17='gcc -std=c17 -Wall -Wextra -pedantic -g -fsanitize=address,undefined'
```

### Pattern

**P-1. Compile every change with warnings and sanitizers; ship with warnings as errors.** *(Practice)* Development builds: `-std=c17 -Wall -Wextra -pedantic -g -fsanitize=address,undefined`. CI: the same plus `-Werror`, plus a run under each sanitizer, plus a build with the other compiler. Release builds: `-O2` without sanitizers. *Trade-off:* slower development builds and occasional false-positive warnings to silence, in exchange for catching most memory bugs and UB on the first run instead of in production.

### Mastery check

1. **Explain.** Why did the uninitialised-variable warning appear only with `-O2`?
2. **Write.** Take any program from this chapter, add a deliberate out-of-bounds write, and read the AddressSanitizer report.

## Chapter 0 Project — Inspect every stage of a build

**Goal:** prove you understand the toolchain by building a two-file program step by step, inspecting each artefact, and causing — then fixing — one error at each stage.

### Requirements

1. Write `mathx.h` (declarations), `mathx.c` (definitions of `int gcd(int, int)` and `long long ipow(int base, unsigned exp)`), and `main.c` (uses both and prints results).
2. Build without the one-step `gcc a.c b.c` command: run `-E`, `-S`, and `-c` for each file, then link the two objects.
3. Show with `nm` which symbols each object defines and which it needs.
4. Cause and capture: a preprocessor error, a compile error, a link error, and a run-time failure — and fix each.
5. Show that the header protects you from DT-5: change the definition's type in `mathx.c` and see the compiler error that the include produces.

### Reference solution

```c
// file: project/mathx.h
#ifndef MATHX_H
#define MATHX_H

/* gcd returns the greatest common divisor of a and b (both >= 0). */
int gcd(int a, int b);

/* ipow returns base raised to exp, by repeated squaring. */
long long ipow(int base, unsigned exp);

#endif
```

The `#ifndef MATHX_H` / `#define` / `#endif` lines are an **include guard**: if the header is included twice into one translation unit, the second copy expands to nothing (Chapter 4).

```c
// file: project/mathx.c
#include "mathx.h"

int gcd(int a, int b) {
    while (b != 0) {
        int t = a % b;
        a = b;
        b = t;
    }
    return a;
}

long long ipow(int base, unsigned exp) {
    long long result = 1, b = base;
    while (exp > 0) {
        if (exp & 1u)
            result *= b;
        b *= b;
        exp >>= 1;
    }
    return result;
}
```

`ipow` uses **exponentiation by squaring**: it looks at the bits of `exp` (Unit 0.5) and needs only about log₂(exp) multiplications instead of exp.

```c
// file: project/main.c
#include <stdio.h>
#include "mathx.h"

int main(void) {
    printf("gcd(84, 36) = %d\n", gcd(84, 36));
    printf("gcd(17, 5)  = %d\n", gcd(17, 5));
    printf("3^13 = %lld\n", ipow(3, 13));
    printf("2^62 = %lld\n", ipow(2, 62));
    return 0;
}
```

Step by step:

```console
$ cd project && gcc -std=c17 -Wall -Wextra -E main.c | tail -n 12


long long ipow(int base, unsigned exp);
# 4 "main.c" 2

int main(void) {
    printf("gcd(84, 36) = %d\n", gcd(84, 36));
    printf("gcd(17, 5)  = %d\n", gcd(17, 5));
    printf("3^13 = %lld\n", ipow(3, 13));
    printf("2^62 = %lld\n", ipow(2, 62));
    return 0;
}
$ cd project && gcc -std=c17 -Wall -Wextra -c mathx.c -o mathx.o && gcc -std=c17 -Wall -Wextra -c main.c -o main.o
$ cd project && nm mathx.o && echo --- && nm main.o
0000000000000000 T gcd
0000000000000031 T ipow
---
                 U gcd
                 U ipow
0000000000000000 T main
                 U printf
$ cd project && gcc mathx.o main.o -o mathdemo && ./mathdemo
gcd(84, 36) = 12
gcd(17, 5)  = 1
3^13 = 1594323
2^62 = 4611686018427387904
```

`main.o` needs `gcd`, `ipow`, and `printf` (`U`); `mathx.o` defines `gcd` and `ipow` (`T`). The linker matched them and found `printf` in the C library.

The four kinds of error:

```console
$ cd project && printf '#include "mathz.h"\nint main(void){return 0;}\n' > e1.c && gcc -c e1.c
e1.c:1:10: fatal error: mathz.h: No such file or directory
    1 | #include "mathz.h"
      |          ^~~~~~~~~
compilation terminated.
$ cd project && gcc -c main.c -o main.o && gcc main.o -o broken
/usr/bin/ld: main.o: in function `main':
main.c:(.text+0x13): undefined reference to `gcd'
/usr/bin/ld: main.c:(.text+0x38): undefined reference to `gcd'
/usr/bin/ld: main.c:(.text+0x5d): undefined reference to `ipow'
/usr/bin/ld: main.c:(.text+0x83): undefined reference to `ipow'
collect2: error: ld returned 1 exit status
```

The first is a preprocessor error (a missing header: no line of C was compiled). The second is a link error: `main.o` was linked without `mathx.o`, so `gcd` and `ipow` are undefined references.

And the header protecting the definition — `mathx.c` changed to define `double gcd(double a, double b)`:

```console
$ cd project && sed 's/^int gcd(int a, int b)/double gcd(double a, double b)/' mathx.c > mathx_bad.c && gcc -std=c17 -c mathx_bad.c
mathx_bad.c:4:8: error: conflicting types for ‘gcd’; have ‘double(double,  double)’
    4 | double gcd(double a, double b) {
      |        ^~~
In file included from mathx_bad.c:2:
mathx.h:6:5: note: previous declaration of ‘gcd’ with type ‘int(int,  int)’
    6 | int gcd(int a, int b);
      |     ^~~
mathx_bad.c: In function ‘gcd’:
mathx_bad.c:6:19: error: invalid operands to binary % (have ‘double’ and ‘double’)
    6 |         int t = a % b;
      |                   ^
```

Because `mathx_bad.c` includes its own header, the compiler sees both the declaration and the conflicting definition and refuses. Without the include, this mistake would have compiled and linked, and produced nonsense at run time.

### Review checklist

- Does every `.c` file include its own header first? (It catches DT-5 and proves the header is self-contained.)
- Does the header have an include guard?
- Can you say, for each error message you produced, which stage produced it and how you knew?
- Did you compile with `-Wall -Wextra` and get zero warnings?

## Answers to the Chapter 0 prediction questions

**0.2 Q1.** 300 mod 256 = 44: only the low 8 bits of the exit status reach the shell on POSIX systems.

**0.4 Q1.** Two source files each *define* a global `int count;` (or a function named `count`). Each compiled separately is fine; the linker sees two definitions. Since GCC 10 (`-fno-common` default), even two uninitialised `int count;` definitions are an error. Fix: define it in one file and declare it `extern int count;` in a header (Chapter 4).

**0.5 Q1.** `010` is octal 8 and `0x10` is 16, so it prints `24`.

**0.5 Q2.** −2 is `1111 1110` (invert `0000 0010` to `1111 1101`, add 1). `1000 0001` is −128 + 1 = −127.

**0.6 Q2.** 3. A `static` local has static storage duration: it is initialised once, before the program starts, not on each call, and keeps its value between calls. Because its initial value is zero, it lives in `.bss` (a non-zero initial value would put it in `.data`).

# Chapter 1 — Core Language

*Goal: know the exact rules of C's types, operators, control flow, and functions — including the conversion rules that cause most subtle C bugs.*

C's core language is small: a few dozen keywords, a handful of types, and operators most programmers recognise. The difficulty is not the size but the precision. C converts between types silently, evaluates some expressions in an unspecified order, and leaves some operations undefined. This chapter goes through the core language rule by rule, with each rule demonstrated by a program whose output you should predict before reading it.

**What you will be able to do after this chapter**

- Declare and initialise variables of every basic type, and choose the right integer type for a job.
- Apply the integer promotion and usual arithmetic conversion rules to predict the type and value of any arithmetic expression.
- Use floating-point numbers knowing exactly where they are inexact.
- Use every operator correctly, including shifts, bitwise operators, and the ones with surprising precedence.
- Recognise expressions with undefined or unspecified evaluation order.
- Write functions with prototypes, handle command-line arguments, and read input safely.

## Topic 1A — Values and types

## Unit 1.1 — Variables and declarations

**One-line summary:** A **declaration** introduces a name with a type; a **definition** also creates the object (or function body); an **initialiser** gives it a first value. Declare variables in the smallest scope where they are needed, initialise them immediately, and mark them `const` if they never change.

### Declarations

```c
// file: decl.c
#include <stdio.h>

int main(void) {
    int count = 0;              /* definition with initialiser */
    double ratio = 0.75;
    char grade = 'A';           /* a character constant is an int value: 65 */
    const int limit = 10;       /* cannot be assigned after initialisation */
    int a = 1, b = 2, c;        /* several declarators; c is NOT initialised */

    c = a + b;
    count = count + limit;
    printf("count=%d ratio=%.2f grade=%c (%d) c=%d\n", count, ratio, grade, grade, c);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra decl.c -o decl && ./decl
count=10 ratio=0.75 grade=A (65) c=3
```

Rules worth knowing exactly:

- **Identifiers** start with a letter or underscore and continue with letters, digits, and underscores. They are case-sensitive. Names beginning with an underscore followed by a capital letter or another underscore (`_Foo`, `__bar`) are **reserved** for the implementation; never declare them.
- Since C99, declarations may appear anywhere in a block, including in a `for` statement's first clause. Declare variables where you first need them.
- `char grade = 'A'` stores the number 65 (in ASCII and UTF-8). In C, a character constant like `'A'` has type **`int`**, not `char` — `sizeof('A')` is 4 (DT-11).
- `const` means "this object must not be modified through this name". It is *not* a compile-time constant in C before C23: `const int n = 10; int arr[n];` makes a variable-length array, not a fixed-size one. Use `enum { N = 10 };` or `#define N 10` for compile-time constants (C23 adds `constexpr` objects).

### Scope

A name declared inside a block `{ … }` is visible from its declaration to the end of the block. An inner declaration can **shadow** an outer one with the same name — legal, and a frequent source of confusion; `-Wshadow` warns about it.

```c
// file: scope.c
#include <stdio.h>

int x = 1;                       /* file scope */

int main(void) {
    printf("%d ", x);            /* 1: the global */
    int x = 2;                   /* shadows the global from here on */
    {
        printf("%d ", x);        /* 2 */
        int x = 3;               /* shadows again inside this block */
        printf("%d ", x);        /* 3 */
    }
    printf("%d\n", x);           /* 2 again */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -Wshadow scope.c -o scope && ./scope
scope.c: In function ‘main’:
scope.c:8:9: warning: declaration of ‘x’ shadows a global declaration [-Wshadow]
    8 |     int x = 2;                   /* shadows the global from here on */
      |         ^
scope.c:4:5: note: shadowed declaration is here
    4 | int x = 1;                       /* file scope */
      |     ^
scope.c:11:13: warning: declaration of ‘x’ shadows a previous local [-Wshadow]
   11 |         int x = 3;               /* shadows again inside this block */
      |             ^
scope.c:8:9: note: shadowed declaration is here
    8 |     int x = 2;                   /* shadows the global from here on */
      |         ^
1 2 3 2
```

### Deep & Tricky

**DT-11. A character constant has type `int` in C (but `char` in C++).** *(Guarantee: C standard)* `sizeof('A')` is `sizeof(int)` in C and 1 in C++. Code that compiles as both languages can behave differently. The value of `'A'` is its code in the execution character set — 65 on every ASCII-compatible system.

**DT-12. `const` objects are not constant expressions (before C23).** *(Guarantee: C standard)* A `const int` cannot be used where the language requires a constant expression: a `case` label, a bit-field width, a file-scope array size, or a static initialiser of another object. Use `enum` constants (true constants of type `int`), `#define`, or C23's `constexpr`.

### Mastery check

1. **Predict.** What does `printf("%zu %zu\n", sizeof(char), sizeof('a'));` print on 64-bit Linux?
2. **Explain.** Why does `-Wshadow` matter in a long function?

## Unit 1.2 — Integer types

**One-line summary:** C has five standard signed integer types (`signed char`, `short`, `int`, `long`, `long long`), their unsigned counterparts, plain `char` (whose signedness is implementation-defined), `_Bool`, and fixed-width aliases in `<stdint.h>`. Choose by meaning: `int` for ordinary counts, `size_t` for sizes and indexes, fixed-width types for external formats.

### The types and their guaranteed minimum ranges

| Type | Minimum width | Typical on 64-bit Linux | `printf` |
| --- | --- | --- | --- |
| `signed char` / `unsigned char` | 8 | 8 | `%hhd` / `%hhu` |
| `short` / `unsigned short` | 16 | 16 | `%hd` / `%hu` |
| `int` / `unsigned` | 16 | 32 | `%d` / `%u` |
| `long` / `unsigned long` | 32 | 64 (32 on Windows) | `%ld` / `%lu` |
| `long long` / `unsigned long long` | 64 | 64 | `%lld` / `%llu` |
| `size_t` (unsigned) | 16 | 64 | `%zu` |
| `ptrdiff_t` (signed) | 16 | 64 | `%td` |
| `int32_t`, `uint64_t`, … | exact | exact | `PRId32`, `PRIu64` from `<inttypes.h>` |
| `intmax_t` / `uintmax_t` | 64 | 64 | `%jd` / `%ju` |

`int_least16_t` and `int_fast16_t` guarantee "at least 16 bits", the smallest or the fastest such type; exact-width types like `int16_t` exist on every common platform but are technically optional.

### Literals and their types

An integer literal's type is the **first type in a list that can represent its value**. For a decimal literal without a suffix the list is `int`, `long`, `long long`; for hex and octal literals it also includes the unsigned types: `int`, `unsigned`, `long`, `unsigned long`, … Suffixes choose a starting point: `u` (unsigned), `l` (long), `ll` (long long), and combinations like `ull`.

```c
// file: literals.c
#include <stdio.h>

#define TYPE_NAME(x) _Generic((x),                    \
    int: "int", unsigned: "unsigned",                  \
    long: "long", unsigned long: "unsigned long",      \
    long long: "long long", unsigned long long: "unsigned long long", \
    default: "other")

int main(void) {
    printf("%-14s %s\n", "2147483647", TYPE_NAME(2147483647));
    printf("%-14s %s\n", "2147483648", TYPE_NAME(2147483648));
    printf("%-14s %s\n", "0x7FFFFFFF", TYPE_NAME(0x7FFFFFFF));
    printf("%-14s %s\n", "0xFFFFFFFF", TYPE_NAME(0xFFFFFFFF));
    printf("%-14s %s\n", "10u", TYPE_NAME(10u));
    printf("%-14s %s\n", "10ll", TYPE_NAME(10ll));
    printf("%-14s %s\n", "'x'", TYPE_NAME('x'));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra literals.c -o literals && ./literals
2147483647     int
2147483648     long
0x7FFFFFFF     int
0xFFFFFFFF     unsigned
10u            unsigned
10ll           long long
'x'            int
```

`_Generic` (C11) chooses an expression by the *type* of its controlling expression — here, to print type names. It is how `<tgmath.h>` makes `sqrt` work for `float` and `double`, and it is the tool for type-generic macros (Chapter 7).

Note `0xFFFFFFFF` is `unsigned int` but `4294967295` (the same value in decimal) would be `long`: hex literals move to unsigned types first. This is the source of real bugs when a hex constant is compared with a negative number (Unit 1.3).

### `char` is three types

`char`, `signed char`, and `unsigned char` are three distinct types. Whether plain `char` is signed is implementation-defined: signed on x86 Linux and Windows, **unsigned on ARM Linux** (including Raspberry Pi and AWS Graviton). Code like `char c = getchar(); if (c == EOF)` or `char c = 200; if (c > 100)` behaves differently on the two. Rules: use plain `char` only for text; use `unsigned char` (or `uint8_t`) for bytes; never store `getchar()`'s result in a `char` (Unit 1.9).

### Booleans

`_Bool` (C99) holds 0 or 1; any non-zero value converted to `_Bool` becomes 1. `<stdbool.h>` defines `bool`, `true`, and `false`; in **C23** they are keywords and the header is unnecessary. Conditions in `if` and loops accept any scalar value: zero is false, everything else is true.

### Mastery check

1. **Predict.** What is the type of `0x80000000`? Of `2147483648`? (Assume 32-bit `int`, 64-bit `long`.)
2. **Explain.** Why should a program that processes binary data use `unsigned char` rather than `char`?

## Unit 1.3 — Integer promotions and the usual arithmetic conversions

**One-line summary:** Before arithmetic, C converts operands: types narrower than `int` are **promoted** to `int`, and then two operands of different types are brought to a **common type** — and when a signed and an unsigned operand of the same rank meet, the signed one becomes **unsigned**. Most surprising C arithmetic follows from these two rules.

### Rule 1: integer promotions

Values of type `char`, `short`, `_Bool`, and bit-fields are converted to `int` (or to `unsigned int` if `int` cannot hold all their values) whenever they are used in arithmetic. So there is no such thing as `char` arithmetic in C:

```c
// file: promo.c
#include <stdio.h>

int main(void) {
    unsigned char a = 200, b = 100;
    int sum = a + b;                       /* promoted to int: 300, no wrap */
    unsigned char wrapped = a + b;         /* converted back on assignment: 300 % 256 = 44 */
    printf("a + b = %d, stored in unsigned char = %d\n", sum, wrapped);
    printf("sizeof(a + b) = %zu\n", sizeof(a + b));

    unsigned char x = 0x0F;
    printf("~x = %d (not 240!)\n", ~x);    /* ~ applies to the promoted int 0x0000000F */
    printf("(unsigned char)~x = %d\n", (unsigned char)~x);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra promo.c -o promo && ./promo
a + b = 300, stored in unsigned char = 44
sizeof(a + b) = 4
~x = -16 (not 240!)
(unsigned char)~x = 240
```

`~x` operated on a 32-bit `int` with value 15, flipping all 32 bits to give −16. This trips up anyone writing bit-manipulation code for byte values: mask or cast the result back.

### Rule 2: the usual arithmetic conversions

After promotion, if the two operands of a binary operator (`+ - * / % < > == & | ^` and others) still have different types:

1. If either is a floating type, the other is converted to the wider floating type (`long double` > `double` > `float`).
2. Otherwise both are integers. If they have the same signedness, the one with lower **rank** (`int` < `long` < `long long`) is converted to the higher.
3. If the **unsigned** operand's rank is greater than or equal to the signed one's, the signed operand is converted to the unsigned type.
4. Otherwise, if the signed type can represent all values of the unsigned type, the unsigned operand is converted to the signed type.
5. Otherwise both are converted to the unsigned version of the signed type.

Rule 3 is the dangerous one:

```c
// file: signcmp.c
#include <stdio.h>
#include <string.h>

int main(void) {
    int n = -1;
    unsigned u = 1;
    if (n < u)
        printf("-1 < 1u\n");
    else
        printf("-1 >= 1u  (because -1 became %u)\n", (unsigned)n);

    long ln = -1;                       /* long has higher rank than unsigned int ... */
    if (ln < u)                         /* ... and can hold every unsigned value: rule 4 */
        printf("-1L < 1u (on LP64, u became long)\n");

    const char *s = "hi";
    if (strlen(s) - 3 > 0)              /* size_t arithmetic: 2 - 3 wraps to a huge number */
        printf("strlen(s) - 3 > 0 ?! value = %zu\n", strlen(s) - 3);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra signcmp.c -o signcmp && ./signcmp
signcmp.c: In function ‘main’:
signcmp.c:8:11: warning: comparison of integer expressions of different signedness: ‘int’ and ‘unsigned int’ [-Wsign-compare]
    8 |     if (n < u)
      |           ^
-1 >= 1u  (because -1 became 4294967295)
-1L < 1u (on LP64, u became long)
strlen(s) - 3 > 0 ?! value = 18446744073709551615
```

`-Wextra` warned about the first comparison (`-Wsign-compare`), but not the `strlen` one, because both operands of the subtraction are already unsigned — there is nothing "mixed" to warn about. Rule of thumb: **never subtract unsigned values unless you know the result is non-negative**; compare instead (`if (strlen(s) > 3)`).

### Conversions on assignment

Assigning a value to a different type converts it:

- to an **unsigned** type: the value modulo 2ⁿ (always defined);
- to a **signed** type that cannot hold it: implementation-defined before C23 (wraps on GCC/Clang), required to wrap in C23;
- from floating to integer: truncated towards zero; **undefined** if the result does not fit (DT-8);
- from integer to floating: the nearest representable value, possibly losing precision (a 64-bit integer does not fit exactly in a `double`'s 53-bit mantissa).

`-Wconversion` warns about every implicit conversion that may change a value. It is noisy in old code but valuable in new code.

### Deep & Tricky

**DT-13. `unsigned short` × `unsigned short` can be signed overflow.** *(Guarantee: integer promotions)* Two `uint16_t` values are promoted to (signed) `int` before multiplying. `uint16_t a = 65535, b = 65535; uint32_t p = a * b;` computes 65535 × 65535 = 4,294,836,225 in `int`, which overflows a 32-bit `int`: undefined behaviour, even though every variable is unsigned. Write `(uint32_t)a * b`.

**DT-14. The type of an expression decides overflow, not the variable it is stored in.** *(Guarantee)* `long long big = 1000000 * 1000000;` multiplies two `int`s, overflows (UB), and only then converts to `long long`. Write `1000000LL * 1000000`. The same applies to `int seconds_per_year = 365 * 24 * 60 * 60;` fine, but `* 1000` for milliseconds overflows.

### Common mistakes

- Using `unsigned` for quantities that "can't be negative", then computing differences: `for (unsigned i = n - 1; i >= 0; i--)` is an infinite loop, since `i >= 0` is always true.
- Comparing `size_t` results (`strlen`, `sizeof`) with `int` variables that may be negative.

### Mastery check

1. **Predict.** For `unsigned char c = 255;`, what do `c + 1`, `(unsigned char)(c + 1)`, and `++c` evaluate to?
2. **Predict.** Does `if (-1 < sizeof(int))` take the branch? Why?
3. **Write.** A function `int safe_mul(int a, int b, int *out)` that returns 0 and stores the product if it fits, or returns −1 on overflow, without ever overflowing (use `INT_MAX / b` comparisons, then compare with `__builtin_mul_overflow`).

## Unit 1.4 — Floating-point numbers

**One-line summary:** `float` and `double` are **IEEE 754** binary floating-point numbers: a sign, an exponent, and a mantissa of 24 or 53 bits. Most decimal fractions (0.1, 0.2) cannot be represented exactly, so floating-point results are approximations; compare with a tolerance, never accumulate money in them, and know the special values `inf` and `NaN`.

```c
// file: floats.c
#include <float.h>
#include <math.h>
#include <stdio.h>

int main(void) {
    double a = 0.1 + 0.2;
    printf("0.1 + 0.2 = %.17g, == 0.3? %s\n", a, a == 0.3 ? "yes" : "no");
    printf("fabs(a - 0.3) < 1e-9? %s\n", fabs(a - 0.3) < 1e-9 ? "yes" : "no");

    float f = 16777216.0f;                 /* 2^24 */
    printf("16777216.0f + 1 = %.1f\n", f + 1.0f);   /* not representable in float */
    printf("FLT_DIG=%d DBL_DIG=%d DBL_EPSILON=%g\n", FLT_DIG, DBL_DIG, DBL_EPSILON);

    double zero = 0.0;
    double inf = 1.0 / zero, nan = zero / zero;
    printf("1/0 = %f, 0/0 = %f, nan == nan? %d, isnan: %d\n", inf, nan, nan == nan, isnan(nan));
    printf("7 / 2 = %d, 7.0 / 2 = %.1f, (double)7 / 2 = %.1f\n", 7 / 2, 7.0 / 2, (double)7 / 2);

    double sum = 0;
    for (int i = 0; i < 10; i++)
        sum += 0.1;
    printf("ten times 0.1 = %.17g\n", sum);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra floats.c -o floats -lm && ./floats
0.1 + 0.2 = 0.30000000000000004, == 0.3? no
fabs(a - 0.3) < 1e-9? yes
16777216.0f + 1 = 16777216.0
FLT_DIG=6 DBL_DIG=15 DBL_EPSILON=2.22045e-16
1/0 = inf, 0/0 = -nan, nan == nan? 0, isnan: 1
7 / 2 = 3, 7.0 / 2 = 3.5, (double)7 / 2 = 3.5
ten times 0.1 = 0.99999999999999989
```

What to take from it:

- `%.17g` shows enough digits to distinguish any two `double`s; `%f`'s default of 6 decimals hides errors.
- `float` has only 24 bits of mantissa (about 7 decimal digits). Above 2²⁴ it cannot represent every integer. Use `double` by default; use `float` only for large arrays where memory or SIMD throughput matters.
- glibc prints a NaN with its sign bit set as `-nan`; the x86 instruction that computes 0/0 produces such a "negative" NaN. The sign of a NaN means nothing.
- Floating-point division by zero is **not** undefined on IEEE 754 systems (GCC and Clang define `__STDC_IEC_559__`): it gives ±infinity or NaN. Integer division by zero **is** undefined.
- `NaN` compares unequal to everything, including itself. Use `isnan`.
- `7 / 2` is integer division: 3. Convert one operand to `double` *before* dividing.
- `-lm` links the math library. On glibc 2.34+ it is merged into libc, but other systems still need it; always write it.

### Money and exact decimals

Never represent money as `double`: use integer cents (`int64_t`), or a decimal library. Ten additions of 0.1 do not give 1.0.

### Deep & Tricky

**DT-15. `x == x` is false for NaN, so NaN breaks sorting and searching.** *(IEEE 754 behaviour)* A comparison function that uses `<` gives inconsistent answers when a NaN is present; `qsort` may then produce any order or, in some implementations, misbehave. Filter NaNs before sorting, or define a total order (`isnan` first).

**DT-16. `-ffast-math` makes the compiler assume no NaNs or infinities.** *(Toolchain behaviour)* It enables reassociation and other non-IEEE optimisations; `isnan(x)` may then be compiled to `false`. Use it only on code you have tested under it, never globally on a codebase that checks for NaN.

<!-- expanded -->
### Going deeper: what a `float` can and cannot hold

A `double` has 53 bits of precision — about 15–16 significant decimal digits; a `float` has 24 bits — about 7 digits. Two consequences surprise almost everyone the first time.

First, large integers are not all representable. Above 2^24 a `float` cannot hold every integer, and above 2^53 neither can a `double`: the gap between neighbouring values becomes 2, then 4, and so on. Adding 1 to such a value can do nothing at all.

Second, the decimal fractions we write are almost never exact. `0.1` is stored as the nearest binary fraction, which is slightly more than 0.1; adding ten of them gives a value slightly *less* than 1.0 because the rounding errors of the additions do not cancel. Printing with `%.17g` (enough digits to identify a `double` exactly) shows what is really stored.

```c
// file: fdepth.c
#include <float.h>
#include <math.h>
#include <stdio.h>

int main(void) {
    float f = 16777216.0f;                         /* 2^24 */
    printf("float 2^24 + 1 = %.1f (FLT_DIG = %d)\n", f + 1.0f, FLT_DIG);
    double d = 9007199254740992.0;                 /* 2^53 */
    printf("double 2^53 + 1 = %.1f (DBL_DIG = %d)\n", d + 1.0, DBL_DIG);

    double sum = 0.0;
    for (int i = 0; i < 10; i++) sum += 0.1;
    printf("0.1 is stored as %.17g\n", 0.1);
    printf("ten times 0.1    = %.17g, == 1.0? %s\n", sum, sum == 1.0 ? "yes" : "no");

    /* compare with a tolerance relative to the size of the numbers */
    double a = sum, b = 1.0;
    int close = fabs(a - b) <= 1e-12 * fmax(fabs(a), fabs(b));
    printf("relative difference %.3g: close enough? %s\n", fabs(a - b) / b, close ? "yes" : "no");

    printf("nextafter(1.0, 2.0) - 1.0 = %.17g (DBL_EPSILON = %.17g)\n", nextafter(1.0, 2.0) - 1.0, DBL_EPSILON);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 fdepth.c -o fdepth -lm && ./fdepth
float 2^24 + 1 = 16777216.0 (FLT_DIG = 6)
double 2^53 + 1 = 9007199254740992.0 (DBL_DIG = 15)
0.1 is stored as 0.10000000000000001
ten times 0.1    = 0.99999999999999989, == 1.0? no
relative difference 1.11e-16: close enough? yes
nextafter(1.0, 2.0) - 1.0 = 2.2204460492503131e-16 (DBL_EPSILON = 2.2204460492503131e-16)
```

`DBL_EPSILON` is the distance from 1.0 to the next larger `double`. A tolerance must scale with the numbers compared: `1e-12` is a sensible *relative* tolerance for `double` results of a few operations, but an absolute `fabs(a - b) < 1e-9` is far too strict for values near 10^12 and far too loose for values near 10^-12. There is no single right epsilon; choose it from the precision of the inputs and the number of operations.

### Deep & Tricky

**DT-17. Converting a floating-point value to an integer type that cannot hold it is undefined behaviour — it does not saturate and it does not wrap.** *(Guarantee: C17 §6.3.1.4 ¶1: "If the value of the integral part cannot be represented by the integer type, the behavior is undefined.")* `(int)3e9` and `(unsigned)-1.0` are both undefined. On x86-64, GCC usually produces the CPU's "integer indefinite" value `INT_MIN` for an out-of-range `cvttsd2si`, while ARM saturates — so the same program prints different numbers on a laptop and a phone, and UBSan's `-fsanitize=float-cast-overflow` reports it. Check the range before converting: `if (x >= INT_MIN && x < INT_MAX + 1.0) i = (int)x;`.

```c
// file: fcast.c
#include <stdio.h>

int main(int argc, char **argv) {
    (void)argv;
    volatile double big = 3e9 + argc;         /* volatile: computed at run time */
    int i = (int)big;
    printf("(int)%.0f = %d\n", big, i);
    return 0;
}
```

```console
$ gcc -std=c17 -O2 fcast.c -o fcast && ./fcast
(int)3000000001 = -2147483648
$ clang -std=c17 -g -fsanitize=float-cast-overflow fcast.c -o fcast-ub && ./fcast-ub 2>&1 | sed 's/^.*runtime error/runtime error/'
runtime error: 3e+09 is outside the range of representable values of type 'int'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior fcast.c:7:13 
(int)3000000001 = -2147483648
```

### Mastery check

1. **Predict.** What does `printf("%d\n", (int)2.99);` print? And `(int)-2.99`?
2. **Explain.** Why does `float f = 0.1f; double d = 0.1; f == d` evaluate to false?

## Topic 1B — Operators and expressions

## Unit 1.5 — Operators

**One-line summary:** C's operators are familiar, but several have exact rules worth memorising: integer `/` truncates towards zero and `%` takes the sign of the dividend; shifts by negative amounts or by the type's width are undefined, as is left-shifting a negative value; `&&` and `||` short-circuit; and bitwise operators bind more loosely than comparisons.

### Arithmetic

```c
// file: ops.c
#include <stdio.h>

int main(void) {
    printf(" 7 /  2 = %d,  7 %%  2 = %d\n", 7 / 2, 7 % 2);
    printf("-7 /  2 = %d, -7 %%  2 = %d\n", -7 / 2, -7 % 2);
    printf(" 7 / -2 = %d,  7 %% -2 = %d\n", 7 / -2, 7 % -2);

    unsigned flags = 0;
    flags |= 1u << 3;                     /* set bit 3 */
    flags |= 1u << 0;                     /* set bit 0 */
    flags &= ~(1u << 0);                  /* clear bit 0 */
    printf("flags = 0x%X, bit 3 set? %d\n", flags, (flags >> 3) & 1u);
    printf("0xF0 ^ 0xFF = 0x%X\n", 0xF0 ^ 0xFF);

    int x = 5;
    if ((x & 1) == 1)                     /* parentheses required: == binds tighter than & */
        printf("x is odd\n");
    if (x & 1 == 1)                       /* parsed as x & (1 == 1) -- happens to work here */
        printf("(precedence trap compiled, warning shown above)\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra ops.c -o ops && ./ops
ops.c: In function ‘main’:
ops.c:19:11: warning: suggest parentheses around comparison in operand of ‘&’ [-Wparentheses]
   19 |     if (x & 1 == 1)                       /* parsed as x & (1 == 1) -- happens to work here */
      |           ^
 7 /  2 = 3,  7 %  2 = 1
-7 /  2 = -3, -7 %  2 = -1
 7 / -2 = -3,  7 % -2 = 1
flags = 0x8, bit 3 set? 1
0xF0 ^ 0xFF = 0xF
x is odd
(precedence trap compiled, warning shown above)
```

Division truncates towards zero (since C99), so `-7 / 2` is −3, and `%` satisfies `(a/b)*b + a%b == a`, giving `-7 % 2 == -1`. To test whether a number is odd, use `x % 2 != 0` (not `== 1`, which fails for negative odd numbers) or `x & 1`.

### Shifts

`x << n` and `x >> n` shift bits. The rules:

- If `n` is negative or ≥ the width of the (promoted) left operand: **undefined**. `1 << 32` with a 32-bit `int` is UB — not 0, not 1. On x86 the CPU masks the count to 5 bits, so it often gives 1; the optimiser may give anything.
- Left-shifting a negative value, or shifting a 1 into or past the sign bit of a signed type (`1 << 31` for 32-bit `int`), is undefined. Use unsigned types for bit manipulation: `1u << 31` is fine.
- Right-shifting a negative value is implementation-defined (arithmetic, sign-extending, on GCC and Clang).

### Logical operators and short-circuiting

`a && b` evaluates `b` only if `a` is true; `a || b` evaluates `b` only if `a` is false. This is a guarantee you rely on constantly: `if (p != NULL && p->count > 0)` never dereferences a null pointer. The results are `int` values 0 or 1.

### Precedence you must know

Most of C's 15 precedence levels are intuitive. These are not:

| Expression | Means | Not |
| --- | --- | --- |
| `x & 1 == 1` | `x & (1 == 1)` | `(x & 1) == 1` |
| `a << 1 + b` | `a << (1 + b)` | `(a << 1) + b` |
| `*p++` | `*(p++)`: use, then advance the pointer | `(*p)++` |
| `a = b == c` | `a = (b == c)` | |
| `x > 0 ? a : b = 1` | invalid in C (the result of `?:` is not an lvalue) | |

When in doubt, add parentheses; `-Wall` warns (`-Wparentheses`) about the worst cases.

### Assignment, compound assignment, increment

Assignment is an expression whose value is the assigned value, allowing `a = b = 0;` and the idiom `while ((c = getchar()) != EOF)`. `x += y` is `x = x + y` with `x` evaluated once. `i++` yields the old value and increments; `++i` increments and yields the new value.

### The comma and conditional operators

`a, b` evaluates `a`, discards it, then evaluates `b` — mostly seen in `for (i = 0, j = n - 1; i < j; i++, j--)`. `c ? x : y` evaluates exactly one of `x` and `y`; its type follows the usual arithmetic conversions (so `cond ? 1 : 2.0` has type `double`).

### Deep & Tricky

**DT-18. `1 << 31` is undefined for a 32-bit `int`; `1u << 31` is fine.** *(Guarantee: C standard; C++20 differs)* Shifting a 1 into the sign bit of a signed integer is undefined in C (C++20 made it well-defined). Bit masks should be built from unsigned constants: `1u << n`, `UINT64_C(1) << n` for 64-bit masks — and `n` must be less than the width.

**DT-19. `%` of a negative number is negative.** *(Guarantee since C99)* `-7 % 3 == -1`. Code that uses `%` to wrap an index into a ring buffer (`(i - 1) % n`) breaks when `i - 1` is negative; write `((i - 1) % n + n) % n`, or keep indexes unsigned and in range.

### Mastery check

1. **Predict.** `printf("%d %d %d\n", -9 / 4, -9 % 4, 9 % -4);`
2. **Predict.** For `unsigned x = 0xF0;`, what is `x >> 4 & 3`? (Mind the precedence.)
3. **Write.** Functions `set_bit`, `clear_bit`, `toggle_bit`, `test_bit`, and `popcount` (count set bits) for `uint32_t`, with asserts in `main` testing them.

## Unit 1.6 — Order of evaluation and sequence points

**One-line summary:** C does not specify the order in which most operands are evaluated — in `f() + g()` either function may run first — and modifying an object twice, or modifying and reading it, without a **sequencing** rule between them is undefined behaviour. Write one side effect per expression.

```c
// file: order.c
#include <stdio.h>

static int trace(int v) {
    printf("[%d]", v);
    return v;
}

int main(void) {
    int sum = trace(1) + trace(2) * trace(3);   /* order of the three calls: unspecified */
    printf(" = %d\n", sum);
    printf("args: %d %d\n", trace(10), trace(20)); /* argument evaluation order: unspecified */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra order.c -o order && ./order
[1][2][3] = 7
[20][10]args: 10 20
$ clang -std=c17 -Wall -Wextra order.c -o order_clang && ./order_clang
[1][2][3] = 7
[10][20]args: 10 20
```

Precedence decides how the expression is *grouped* (`2 * 3` happens before the addition), not the order in which operands are *evaluated*. GCC evaluated the two `printf` arguments right to left; Clang left to right. Both are correct.

### Undefined: unsequenced modifications

```c
// file: unseq.c
#include <stdio.h>

int main(void) {
    int i = 1;
    int a = i++ + i++;      /* UB: i modified twice, unsequenced */
    int arr[4] = {0};
    int j = 1;
    arr[j] = j++;           /* UB: j read and modified, unsequenced */
    printf("a=%d i=%d arr={%d,%d,%d} j=%d\n", a, i, arr[0], arr[1], arr[2], j);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra unseq.c -o unseq && ./unseq
unseq.c: In function ‘main’:
unseq.c:6:20: warning: operation on ‘i’ may be undefined [-Wsequence-point]
    6 |     int a = i++ + i++;      /* UB: i modified twice, unsequenced */
      |                   ~^~
unseq.c:9:15: warning: operation on ‘j’ may be undefined [-Wsequence-point]
    9 |     arr[j] = j++;           /* UB: j read and modified, unsequenced */
      |              ~^~
a=3 i=3 arr={0,0,1} j=2
$ clang -std=c17 -Wall -Wextra unseq.c -o unseq_clang 2>&1 | grep -c warning; ./unseq_clang
3
a=3 i=3 arr={0,0,1} j=2
```

Both compilers warn (`-Wsequence-point` in GCC, `-Wunsequenced` in Clang). Here they happened to produce the same numbers, but nothing guarantees it: another version, another optimisation level, or a small change to the surrounding code may give a different result. The program is meaningless; the fix is to split it into separate statements.

What *is* sequenced: the left operand of `&&`, `||`, `?:`, and the comma operator is fully evaluated before the right; a function's arguments are all evaluated before the call; the end of each full expression (a statement's expression, an `if` condition) is a sequence point.

### Deep & Tricky

**DT-20. Precedence is not evaluation order.** *(Guarantee: C standard)* In `a() + b() * c()`, the multiplication groups first, but `a()` may be called first, last, or in between. If the functions have side effects that interact (printing, modifying globals, reading input), the program's output depends on the compiler. Put calls with side effects in separate statements.

<!-- expanded -->
### Going deeper: the same program, two compilers, two answers

"Unspecified order" is not a theoretical warning. GCC and Clang make different choices for the order in which function arguments are evaluated, and a program that depends on it prints different output with each:

```c
// file: argorder.c
#include <stdio.h>

static int counter;
static int next(const char *who) {
    counter++;
    printf("  %s runs, counter is now %d\n", who, counter);
    return counter;
}

int main(void) {
    printf("result: %d %d %d\n", next("first"), next("second"), next("third"));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -O2 argorder.c -o argorder-gcc && ./argorder-gcc
  third runs, counter is now 1
  second runs, counter is now 2
  first runs, counter is now 3
result: 3 2 1
$ clang -std=c17 -Wall -O2 argorder.c -o argorder-clang && ./argorder-clang
  first runs, counter is now 1
  second runs, counter is now 2
  third runs, counter is now 3
result: 1 2 3
```

GCC on x86-64 evaluated the arguments from right to left, Clang from left to right — and neither compiler warned, because the program is not undefined: each call is a separate function call, so the side effects are *indeterminately sequenced*, not unsequenced. The result is *unspecified*: one of the possible orders is chosen, but you cannot know which. The fix is to give the order a name:

```c
int a = next("first");
int b = next("second");
int c = next("third");
printf("result: %d %d %d\n", a, b, c);
```

Three statements, three sequence points, one possible output on every compiler.

### Deep & Tricky

**DT-21. "Unspecified" and "undefined" are different promises: an unspecified order (function arguments, operands of `+`) makes the program choose one of several valid behaviours, possibly a different one each time; unsequenced side effects on the same object (`i = i++ + 1`) make the whole program undefined.** *(Guarantee: C17 §3.4.4 "unspecified behavior", §6.5 ¶2 "If a side effect on a scalar object is unsequenced relative to either a different side effect on the same scalar object or a value computation using the value of the same scalar object, the behavior is undefined.")* The compiler warning `-Wsequence-point` (in `-Wall`) catches the simplest undefined cases; no warning exists for unspecified argument order, because almost every program has some — it only matters when the arguments have side effects that interact.

### Mastery check

1. **Explain.** Which of these are undefined: `x = x + 1;`, `x = x++;`, `a[i] = i;`, `a[i++] = i;`, `f(i++, i);`?

## Topic 1C — Control flow and functions

## Unit 1.7 — Control flow

**One-line summary:** C has `if`/`else`, `switch`, three loops (`while`, `do`/`while`, `for`), `break`, `continue`, `return`, and `goto`. `switch` cases **fall through** unless you `break`; `do`/`while` runs at least once; and `goto` has one respectable use: jumping to cleanup code.

### `switch` and fall-through

```c
// file: sw.c
#include <stdio.h>

static const char *kind(int c) {
    switch (c) {
    case 'a': case 'e': case 'i': case 'o': case 'u':
        return "vowel";
    case ' ':
    case '\t':
        return "space";
    default:
        if (c >= '0' && c <= '9')
            return "digit";
        return "other";
    }
}

int main(void) {
    for (const char *p = "hi 42"; *p != '\0'; p++)
        printf("'%c' %s\n", *p, kind(*p));

    int level = 2;
    switch (level) {
    case 3:
        printf("level 3 setup\n");
        /* fall through */
    case 2:
        printf("level 2 setup\n");
        __attribute__((fallthrough));      /* or [[fallthrough]]; in C23 */
    case 1:
        printf("level 1 setup\n");
        break;
    default:
        printf("unknown level\n");
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -Wimplicit-fallthrough sw.c -o sw && ./sw
'h' other
'i' vowel
' ' space
'4' digit
'2' digit
level 2 setup
level 1 setup
```

`case` labels must be integer constant expressions. Execution jumps to the matching label and continues — through later labels — until a `break` or the end of the `switch`. Intentional fall-through should be marked (a `/* fall through */` comment, `__attribute__((fallthrough))`, or C23's `[[fallthrough]]`) so `-Wimplicit-fallthrough` can catch the accidental ones.

### Loops

```c
// file: loops.c
#include <stdio.h>

int main(void) {
    for (int i = 0; i < 3; i++)          /* i exists only inside the loop */
        printf("for %d\n", i);

    int n = 0;
    while (n < 3) {
        n++;
        if (n == 2)
            continue;                     /* skip the rest of this iteration */
        printf("while %d\n", n);
    }

    int tries = 10;
    do {                                  /* body runs before the test */
        printf("do-while tries=%d\n", tries);
    } while (tries < 3);

    for (int i = 0; ; i++) {             /* omitted condition = forever */
        if (i * i > 20) {
            printf("first square > 20: %d\n", i * i);
            break;
        }
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra loops.c -o loops && ./loops
for 0
for 1
for 2
while 1
while 3
do-while tries=10
first square > 20: 25
```

### `goto` for cleanup

`goto` jumps to a label in the same function. Unrestricted, it makes code unreadable; but C has no destructors or `defer`, and the clearest way to release several resources on every error path is a ladder of labels at the end of the function. The Linux kernel uses this pattern thousands of times:

```c
int process(const char *path) {
    int rc = -1;
    FILE *f = fopen(path, "r");
    if (f == NULL)
        goto out;
    char *buf = malloc(4096);
    if (buf == NULL)
        goto close_file;
    if (fgets(buf, 4096, f) == NULL)
        goto free_buf;
    rc = 0;                     /* success: fall through the cleanup */
free_buf:
    free(buf);
close_file:
    fclose(f);
out:
    return rc;
}
```

Chapter 6 uses this pattern with real files. Rule: only jump forwards, only to cleanup.

<!-- expanded -->
### Going deeper: loops that never end, and loops that end too soon

Three loop bugs account for most of the infinite or skipped loops in real C code.

**Counting down with an unsigned index.** `size_t` is unsigned, so `i >= 0` is always true; when `i` reaches 0, `i--` wraps to `SIZE_MAX` and the loop continues with an index far outside the array.

**A semicolon after the loop header.** `for (i = 0; i < n; i++);` has an empty body; the block below runs once, after the loop. GCC warns about this pattern with `-Wempty-body` (in `-Wextra`) for `if` and about misleading indentation with `-Wmisleading-indentation` (in `-Wall`).

**`break` inside a `switch` inside a loop** leaves only the `switch`. To leave the loop, use a flag, a `return` from a helper function, or `goto`.

```c
// file: loopbugs.c
#include <stdio.h>
#include <stddef.h>

int main(void) {
    int v[] = {10, 20, 30};
    size_t n = sizeof v / sizeof v[0];

    /* correct countdown with an unsigned index: test before decrementing */
    for (size_t i = n; i-- > 0; )
        printf("v[%zu] = %d\n", i, v[i]);

    /* break in a switch does not leave the loop */
    const char *cmds[] = {"add", "add", "quit", "add"};
    int adds = 0;
    for (size_t i = 0; i < 4; i++) {
        switch (cmds[i][0]) {
        case 'a': adds++; break;
        case 'q': goto done;          /* leaves the loop as well */
        }
    }
done:
    printf("adds before quit: %d\n", adds);

    /* do-while runs its body at least once: good for "read, then test" */
    int tries = 0;
    do { tries++; } while (tries < 0);
    printf("do-while ran %d time(s) although the condition was false\n", tries);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra loopbugs.c -o loopbugs && ./loopbugs
v[2] = 30
v[1] = 20
v[0] = 10
adds before quit: 2
do-while ran 1 time(s) although the condition was false
```

The idiom `for (size_t i = n; i-- > 0; )` tests `i` *before* decrementing it, so the body sees `n-1` down to `0` and the loop stops without wrapping.

```c
// file: countdown.c
#include <stdio.h>

int main(void) {
    int v[] = {1, 2, 3};
    for (size_t i = 2; i >= 0; i--)          /* bug: always true */
        if (i < 3) printf("%d\n", v[i]); else { printf("i wrapped to %zu\n", i); break; }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra countdown.c -o countdown && ./countdown
countdown.c: In function ‘main’:
countdown.c:6:26: warning: comparison of unsigned expression in ‘>= 0’ is always true [-Wtype-limits]
    6 |     for (size_t i = 2; i >= 0; i--)          /* bug: always true */
      |                          ^~
3
2
1
i wrapped to 18446744073709551615
```

`-Wextra` says it outright: "comparison of unsigned expression in ‘>= 0’ is always true". The `if` and `break` are there only to stop the demonstration; without them the loop would read memory far outside the array.

### Deep & Tricky

**DT-22. A loop with no observable side effects that does not terminate is undefined behaviour in C11 and later, unless its controlling expression is a constant expression — so the compiler may assume that such a loop ends, and delete it.** *(Guarantee: C17 §6.8.5 ¶6: "An iteration statement whose controlling expression is not a constant expression, that performs no input/output operations, does not access volatile objects, and performs no synchronization or atomic operations in its body … may be assumed by the implementation to terminate.")* This is why `while (1) { }` (a constant condition) is a valid infinite loop, but a busy-wait such as `while (!flag) { }` on a non-`volatile`, non-atomic `flag` set by another thread may be compiled into an empty loop that never re-reads `flag`, or into nothing at all. Chapter 9 shows the right tools: atomics and condition variables.

### Mastery check

1. **Predict.** In `sw.c`, what is printed if `level` is 3? If it is 7?
2. **Write.** FizzBuzz for 1 to 30 using `switch` on `(i % 3 == 0) * 2 + (i % 5 == 0)`.

## Unit 1.8 — Functions

**One-line summary:** A function has a **prototype** (its declaration with parameter types) and a definition. Arguments are always **passed by value** — the function receives copies — so to let a function modify a caller's variable, pass a pointer to it. Functions not used outside their file should be `static`.

```c
// file: funcs.c
#include <stdio.h>

static void swap_wrong(int a, int b) {     /* receives copies */
    int t = a; a = b; b = t;
}

static void swap(int *a, int *b) {         /* receives addresses */
    int t = *a; *a = *b; *b = t;
}

static unsigned long long factorial(unsigned n) {
    return n <= 1 ? 1 : n * factorial(n - 1);   /* recursion */
}

static int divide(int a, int b, int *quotient, int *remainder) {
    if (b == 0)
        return -1;                          /* error code; outputs untouched */
    *quotient = a / b;
    *remainder = a % b;
    return 0;
}

int main(void) {
    int x = 1, y = 2;
    swap_wrong(x, y);
    printf("after swap_wrong: x=%d y=%d\n", x, y);
    swap(&x, &y);
    printf("after swap:       x=%d y=%d\n", x, y);
    printf("20! = %llu\n", factorial(20));

    int q, r;
    if (divide(17, 5, &q, &r) == 0)
        printf("17 / 5 = %d remainder %d\n", q, r);
    if (divide(1, 0, &q, &r) != 0)
        printf("division by zero reported\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra funcs.c -o funcs && ./funcs
after swap_wrong: x=1 y=2
after swap:       x=2 y=1
20! = 2432902008176640000
17 / 5 = 3 remainder 2
division by zero reported
```

Points to master:

- `&x` takes the address of `x`; `*a` in the callee reads or writes the object at that address. Chapter 2 is about pointers; here they are just the way to get results out.
- C returns one value. Common conventions for more: return an error code and write results through pointer parameters (`divide`); return a struct (Chapter 3); or return a sentinel (`NULL`, −1) for failure.
- A function must be declared before it is called. Calling an undeclared function was allowed in C89 (an "implicit declaration" assumed it returned `int`); it is an error since C99 and GCC 14 rejects it by default.
- `static` on a function gives it **internal linkage**: invisible outside its translation unit, so it cannot clash with other files' names and the compiler can inline it freely (Chapter 4).
- Recursion works and is natural for recursive data, but each call uses stack space; very deep recursion overflows the stack (Unit 0.6).

### `main` with arguments

```c
// file: args.c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("argc = %d\n", argc);
    for (int i = 0; i < argc; i++)
        printf("argv[%d] = \"%s\"\n", i, argv[i]);
    return argv[argc] == NULL ? 0 : 1;      /* guaranteed: argv[argc] is a null pointer */
}
```

```console
$ gcc -std=c17 -Wall -Wextra args.c -o args && ./args hello "two words" 42
argc = 4
argv[0] = "./args"
argv[1] = "hello"
argv[2] = "two words"
argv[3] = "42"
```

`argv[0]` is the program name (as invoked). Arguments are strings; numbers must be converted (Unit 1.9).

### Deep & Tricky

**DT-23. `int f();` and `int f(void);` are different declarations before C23.** *(Guarantee, version-dependent)* Before C23, empty parentheses in a *declaration* mean "parameters unspecified": the compiler does not check calls, so `f(1, 2, 3)` compiles. `(void)` declares "no parameters" and calls with arguments are errors. C23 changed `()` to mean `(void)`. Always write `(void)` for portability.

**DT-24. Arrays are not copied when passed — they become pointers.** *(Guarantee: array-to-pointer conversion)* "Pass by value" applies to the pointer: a parameter declared `int a[10]` is really `int *a`, `sizeof a` inside the function is the size of a pointer, and the callee modifies the caller's array. Chapter 2 explains why.

<!-- expanded -->
### Going deeper: recursion and the call stack

A function may call itself. Each call gets its own **stack frame** with its own parameters and local variables, so recursion is natural for problems defined in terms of smaller versions of themselves: factorial, tree traversal, parsing nested expressions. Two rules make a recursive function correct: a **base case** that returns without recursing, and a guarantee that every recursive call moves closer to it.

The cost is stack space. The main thread's stack on Linux is usually 8 MiB (`ulimit -s`), and each frame takes at least a few dozen bytes; a recursion that is too deep runs off the end of the stack and the process is killed with `SIGSEGV`. How deep is "too deep" depends on the frame size — and on the optimiser.

```c
// file: depth.c
#include <stdio.h>
#include <stdlib.h>

static long count_down(long n) {                 /* recursive, but the call is the last thing it does */
    if (n == 0) return 0;
    return count_down(n - 1);
}

static long sum_to(long n) {                     /* recursive, and work remains after the call */
    if (n == 0) return 0;
    return n + sum_to(n - 1);
}

int main(int argc, char **argv) {
    long n = atol(argv[2]);
    if (argv[1][0] == 't') printf("count_down(%ld) = %ld\n", n, count_down(n));
    else printf("sum_to(%ld) = %ld\n", n, sum_to(n));
    return 0;
}
```

```console
$ gcc -std=c17 -O0 depth.c -o depth0 && ./depth0 sum 100000 && ./depth0 sum 10000000; echo "exit status $?"
sum_to(100000) = 5000050000
bash: line 1:  1383 Segmentation fault      ./depth0 sum 10000000
exit status 139
$ gcc -std=c17 -O2 depth.c -o depth2 && ./depth2 tail 10000000 && ./depth2 sum 10000000; echo "exit status $?"
count_down(10000000) = 0
sum_to(10000000) = 50000005000000
exit status 0
$ ulimit -s
8192
```

At `-O0`, 100,000 levels fit in 8 MiB but 10 million do not: exit status 139 is 128 + 11, killed by `SIGSEGV`. At `-O2` both functions survive 10 million levels — GCC turned the **tail call** in `count_down` into a jump, and even rewrote `sum_to` (where an addition follows the call) into a loop with an accumulator. The language does not promise either transformation; a program that works only because the optimiser removed its recursion crashes in the debug build.

### Deep & Tricky

**DT-25. C does not guarantee tail-call elimination, so the maximum recursion depth of a program can change with the optimisation level, the compiler, and the stack limit of the thread it runs on.** *(Toolchain behaviour; C17 says nothing about stack size or tail calls)* Threads created with `pthread_create` often get smaller stacks than the main thread, and signal handlers may run on an alternate stack of only a few kilobytes. For unbounded or attacker-controlled depth (parsing nested input, walking a degenerate tree), convert the recursion to a loop with an explicit stack on the heap. Clang offers `__attribute__((musttail))` to *require* a tail call — a compile error if it cannot be done — which is the only portable-in-practice way to rely on it.

### Mastery check

1. **Explain.** Why can't `swap_wrong` swap the caller's variables?
2. **Write.** `int minmax(const int *a, int n, int *min, int *max)` returning −1 for an empty array.

## Unit 1.9 — Reading input safely

**One-line summary:** `scanf` is hard to use correctly: it leaves bad input in the stream, has no overflow protection for numbers, and needs a width for strings. The robust pattern is to read a **whole line** with `fgets` and convert it with `strtol`/`strtod`, checking `errno` and the end pointer.

### Why `getchar` returns `int`

`getchar()` returns either a byte value as `unsigned char` converted to `int` (0–255), or `EOF` (a negative `int`, usually −1). Storing the result in a `char` loses the distinction: on signed-`char` platforms byte 255 becomes −1 and looks like `EOF`; on unsigned-`char` platforms `EOF` becomes 255 and the loop never ends.

```c
int c;                       /* int, not char */
while ((c = getchar()) != EOF)
    putchar(c);
```

### Converting text to numbers

```c
// file: readnum.c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* parse_int converts a whole string to an int. Returns 0 on success, -1 on error. */
static int parse_int(const char *s, int *out) {
    char *end;
    errno = 0;
    long v = strtol(s, &end, 10);
    if (end == s)
        return -1;                          /* no digits at all */
    while (*end == ' ' || *end == '\n')
        end++;
    if (*end != '\0')
        return -1;                          /* trailing garbage, e.g. "12abc" */
    if (errno == ERANGE || v < INT_MIN || v > INT_MAX)
        return -1;                          /* out of range */
    *out = (int)v;
    return 0;
}

int main(void) {
    const char *tests[] = {"42", "  -17\n", "12abc", "", "99999999999", "0x1F"};
    for (size_t i = 0; i < sizeof tests / sizeof tests[0]; i++) {
        int v;
        if (parse_int(tests[i], &v) == 0)
            printf("%-16s -> %d\n", tests[i][0] ? tests[i] : "(empty)", v);
        else
            printf("%-16s -> error\n", tests[i][0] ? tests[i] : "(empty)");
    }

    char line[64];
    int total = 0;
    while (fgets(line, sizeof line, stdin) != NULL) {
        int v;
        if (parse_int(line, &v) == 0)
            total += v;
        else
            fprintf(stderr, "skipping bad line: %s", line);
    }
    printf("total = %d\n", total);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra readnum.c -o readnum
$ printf '10\n20\nabc\n30\n' | ./readnum
42               -> 42
  -17
           -> -17
12abc            -> error
(empty)          -> error
99999999999      -> error
0x1F             -> error
skipping bad line: abc
total = 60
```

(`"  -17\n"` printed across two lines because the test string itself contains the newline.) Every check in `parse_int` exists because of a real failure mode: `atoi("abc")` returns 0 with no way to tell it from a real zero, and `atoi("99999999999")` is undefined. Never use `atoi` for untrusted input. `strtol` also accepts leading whitespace and a sign. `"0x1F"` was rejected because `parse_int` asks for base 10; with base 0, `strtol` accepts `0x1F` (hex) and `017` (octal) too.

`fgets(buf, n, stream)` reads at most `n - 1` characters, stops after a newline (which it keeps), and always terminates the string. If a line is longer than the buffer, the rest stays in the stream for the next call — check whether the buffer ends in `'\n'` to detect that.

### Pattern

**P-2. Read lines, then parse; validate every conversion completely.** *(Practice)* Read input with `fgets` (or POSIX `getline` for unlimited lengths), convert with `strtol`/`strtoul`/`strtod`, and check four things: digits were found, the whole field was consumed, `errno` is not `ERANGE`, and the value is in your required range. *Trade-off:* more code than `scanf("%d", &x)`, in exchange for programs that never misread input or get stuck in a loop on bad data.

### Mastery check

1. **Explain.** What happens with `scanf("%d", &n)` in a loop when the user types `abc`?
2. **Write.** `parse_double` with the same four checks, rejecting `nan` and `inf` as well.

<!-- expanded -->
## Unit 1.10 — Variadic functions

**One-line summary:** A function declared with `...` (like `printf`) accepts a variable number of arguments of any types; it reads them with the macros of `<stdarg.h>` — `va_start`, `va_arg`, `va_end`, and `va_copy` — but nothing tells it how many arguments there are or what their types are: the function must find out from a fixed argument (a count, a format string, a terminating sentinel). The extra arguments undergo the **default argument promotions**: `char` and `short` become `int`, `float` becomes `double`.

```c
// file: variadic.c
#include <stdarg.h>
#include <stdio.h>
#include <time.h>

/* Sum 'count' ints. The caller must pass exactly 'count' arguments of type int. */
static long sum_ints(int count, ...) {
    va_list ap;
    va_start(ap, count);                  /* start after the last named parameter */
    long total = 0;
    for (int i = 0; i < count; i++)
        total += va_arg(ap, int);         /* the type must match what was passed (after promotion) */
    va_end(ap);
    return total;
}

/* A logging function that forwards its arguments to vfprintf. The format attribute lets
   GCC and Clang check every call's arguments against the format string, as for printf. */
__attribute__((format(printf, 2, 3)))
static void log_msg(const char *level, const char *fmt, ...) {
    char stamp[16];
    time_t now = 0;                       /* fixed time: reproducible output */
    strftime(stamp, sizeof stamp, "%H:%M:%S", gmtime(&now));
    fprintf(stderr, "%s [%s] ", stamp, level);
    va_list ap;
    va_start(ap, fmt);
    vfprintf(stderr, fmt, ap);            /* the v-functions take a va_list */
    va_end(ap);
    fputc('\n', stderr);
}

int main(void) {
    printf("sum_ints(3, 10, 20, 30) = %ld\n", sum_ints(3, 10, 20, 30));
    char c = 'A';
    float f = 1.5f;
    printf("char %c and float %f arrive as int and double\n", c, f);   /* why %f works for float */
    log_msg("info", "user %s logged in from %d.%d.%d.%d", "ada", 10, 0, 2, 15);
    log_msg("warn", "%zu bytes left", (size_t)512);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra variadic.c -o variadic && ./variadic
sum_ints(3, 10, 20, 30) = 60
char A and float 1.500000 arrive as int and double
00:00:00 [info] user ada logged in from 10.0.2.15
00:00:00 [warn] 512 bytes left
```

There is no `%lf` needed for `float` in `printf`: the `float` was promoted to `double` before `printf` ever saw it, which is also why `printf` has no separate conversion for `float`. (`scanf` is different — it receives *pointers*, which are not promoted, so `%f` means `float *` and `%lf` means `double *`.)

The format attribute is the most valuable line in the file. A typo in a call to `log_msg` is now a compile-time warning instead of a crash:

```c
// file: badlog.c
#include <stdio.h>
__attribute__((format(printf, 1, 2))) void log_msg(const char *fmt, ...);
void f(const char *name, long n) {
    log_msg("%s has %d items", n, name);          /* arguments swapped */
}
```

```console
$ gcc -std=c17 -Wall -c badlog.c 2>&1 | grep -E "warning" | sed 's/^badlog.c:[0-9:]* //'
warning: format ‘%s’ expects argument of type ‘char *’, but argument 2 has type ‘long int’ [-Wformat=]
warning: format ‘%d’ expects argument of type ‘int’, but argument 3 has type ‘const char *’ [-Wformat=]
```

### Deep & Tricky

**DT-26. `va_arg(ap, T)` with a type that does not match the promoted type of the actual argument is undefined behaviour — and `va_arg(ap, char)`, `va_arg(ap, short)` and `va_arg(ap, float)` are always wrong, because those types never arrive in a variadic call.** *(Guarantee: C17 §7.16.1.1 ¶2: the behaviour is undefined if "type is not compatible with the type of the actual next argument (as promoted according to the default argument promotions)")* GCC warns: "'float' is promoted to 'double' when passed through '...'" and adds "if this code is reached, the program will abort". A related trap: passing `0` or `NULL` as a pointer sentinel to a variadic function (as `execl` requires). `0` is an `int`, which on x86-64 is 4 bytes where a pointer is 8; write `(char *)NULL`.

```c
// file: vafloat.c
#include <stdarg.h>
#include <stdio.h>

static double first(int n, ...) {
    va_list ap;
    va_start(ap, n);
    double x = va_arg(ap, float);           /* wrong: floats arrive as doubles */
    va_end(ap);
    return x;
}

int main(void) { printf("%f\n", first(1, 2.5f)); return 0; }
```

```console
$ gcc -std=c17 -Wall vafloat.c -o vafloat 2>&1 | grep -E "warning|note" | sed 's/^vafloat.c:[0-9:]* //'; ./vafloat; echo "exit status $?"
warning: ‘float’ is promoted to ‘double’ when passed through ‘...’
note: (so you should pass ‘double’ not ‘float’ to ‘va_arg’)
note: if this code is reached, the program will abort
bash: line 1:  1421 Illegal instruction     ./vafloat
exit status 132
```

GCC kept its word: it compiled the `va_arg(ap, float)` into a trap instruction (`ud2`), and the program died with `SIGILL` — exit status 132 is 128 + 4. Clang compiles the same code without a trap and reads garbage instead; undefined behaviour has no single symptom.

### Pattern

**P-3. Every printf-like function you write gets `__attribute__((format(printf, N, M)))`, and forwards to the `v` version of the library function (`vfprintf`, `vsnprintf`) instead of re-parsing the format.** *(Practice; GCC manual "Common Function Attributes", `format`)* The attribute turns every call site into a checked call, with exactly the warnings `printf` gets. *Trade-off:* the attribute is a GCC/Clang extension; portable code hides it behind a macro that expands to nothing on other compilers (MSVC uses SAL annotations instead).

### Mastery check

1. **Predict.** `sum_ints(3, 10, 20)` is called with one argument missing. What does the standard say happens, and what is the likely result on x86-64?
2. **Write.** `char *join(const char *sep, ...)`: concatenate string arguments until a `NULL` sentinel, separated by `sep`, into a newly allocated string. Walk the list twice with `va_copy` (once to measure, once to copy).

## Chapter 1 Project — Number-format converter

**Goal:** a command-line tool that converts integers between bases and shows their binary representation and type properties — using every rule of this chapter: integer types and limits, conversions, bitwise operators, robust parsing, functions, and exit codes.

### Specification

```
numconv [-b BITS] [-s] VALUE...
```

- Each `VALUE` may be decimal (`255`), hex (`0xff`), octal (`0377`), or binary (`0b11111111`), optionally negative.
- `-b BITS` (8, 16, 32, or 64; default 32) chooses the width; `-s` interprets values as signed.
- For each value, print decimal, hex, octal, and binary (grouped in fours), or a clear error if the value does not fit in the chosen width and signedness.
- Exit status: 0 if every value converted, 1 if any failed, 2 for usage errors.

### Reference solution

```c
// file: numconv/numconv.c
/* numconv: convert integers between bases and show their bit patterns. */
#include <errno.h>
#include <inttypes.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* parse_value parses an optionally negative decimal, 0x, 0, or 0b number.
   The magnitude is returned in *mag and the sign in *neg. Returns false on
   syntax errors or if the magnitude does not fit in 64 bits. */
static bool parse_value(const char *s, uint64_t *mag, bool *neg) {
    *neg = false;
    if (*s == '-' || *s == '+') {
        *neg = (*s == '-');
        s++;
    }
    int base = 10;
    if (s[0] == '0' && (s[1] == 'b' || s[1] == 'B')) {
        base = 2;
        s += 2;
    } else if (s[0] == '0' && (s[1] == 'x' || s[1] == 'X')) {
        base = 16;
        s += 2;
    } else if (s[0] == '0' && s[1] != '\0') {
        base = 8;
        s += 1;
    }
    if (*s == '\0' || *s == '-' || *s == '+' || *s == ' ')
        return false;                       /* strtoull would accept these */
    char *end;
    errno = 0;
    unsigned long long v = strtoull(s, &end, base);
    if (*end != '\0' || errno == ERANGE)
        return false;
    *mag = v;
    return true;
}

/* fits reports whether -mag (if neg) or mag fits in a bits-wide integer,
   and stores its two's-complement bit pattern (masked to bits) in *pattern. */
static bool fits(uint64_t mag, bool neg, int bits, bool is_signed, uint64_t *pattern) {
    uint64_t mask = bits == 64 ? UINT64_MAX : (UINT64_C(1) << bits) - 1;
    if (is_signed) {
        uint64_t max_pos = mask >> 1;          /* 2^(bits-1) - 1 */
        if (neg ? mag > max_pos + 1 : mag > max_pos)
            return false;
    } else {
        if (neg && mag != 0)
            return false;
        if (mag > mask)
            return false;
    }
    /* unsigned negation is defined modulo 2^64: this is two's complement */
    *pattern = (neg ? -mag : mag) & mask;
    return true;
}

static void print_binary(uint64_t v, int bits) {
    for (int i = bits - 1; i >= 0; i--) {
        putchar((v >> i) & 1u ? '1' : '0');
        if (i % 4 == 0 && i != 0)
            putchar('_');
    }
    putchar('\n');
}

/* signed_value interprets the low bits of pattern as a two's-complement number. */
static int64_t signed_value(uint64_t pattern, int bits) {
    if (bits < 64 && (pattern >> (bits - 1)) & 1u)
        pattern |= ~((UINT64_C(1) << bits) - 1);   /* sign-extend */
    return (int64_t)pattern;   /* implementation-defined before C23; two's complement on GCC/Clang */
}

static int usage(void) {
    fprintf(stderr, "usage: numconv [-b 8|16|32|64] [-s] VALUE...\n");
    return 2;
}

int main(int argc, char *argv[]) {
    int bits = 32;
    bool is_signed = false;
    int i = 1;
    for (; i < argc && argv[i][0] == '-' && argv[i][1] != '\0' &&
           (argv[i][1] < '0' || argv[i][1] > '9'); i++) {
        if (strcmp(argv[i], "-s") == 0) {
            is_signed = true;
        } else if (strcmp(argv[i], "-b") == 0 && i + 1 < argc) {
            bits = atoi(argv[++i]);   /* validated just below against a fixed set */
            if (bits != 8 && bits != 16 && bits != 32 && bits != 64)
                return usage();
        } else {
            return usage();
        }
    }
    if (i == argc)
        return usage();

    int status = 0;
    for (; i < argc; i++) {
        uint64_t mag, pattern;
        bool neg;
        if (!parse_value(argv[i], &mag, &neg)) {
            fprintf(stderr, "%s: not a valid number\n", argv[i]);
            status = 1;
            continue;
        }
        if (!fits(mag, neg, bits, is_signed, &pattern)) {
            fprintf(stderr, "%s: does not fit in %s %d-bit integer\n",
                    argv[i], is_signed ? "a signed" : "an unsigned", bits);
            status = 1;
            continue;
        }
        printf("%s\n", argv[i]);
        if (is_signed)
            printf("  dec  %" PRId64 "\n", signed_value(pattern, bits));
        else
            printf("  dec  %" PRIu64 "\n", pattern);
        printf("  hex  0x%0*" PRIX64 "\n", bits / 4, pattern);
        printf("  oct  0%" PRIo64 "\n", pattern);
        printf("  bin  ");
        print_binary(pattern, bits);
    }
    return status;
}
```

Design notes:

- The magnitude and sign are parsed separately and range-checked *before* any conversion, so no step can overflow. `-mag` on a `uint64_t` is defined (modulo 2⁶⁴) and produces exactly the two's-complement pattern — unsigned arithmetic is the safe way to manipulate bit patterns (DT-7).
- `PRId64`, `PRIX64` from `<inttypes.h>` are the portable `printf` formats for fixed-width types. `%0*` takes the field width from an argument.
- The mask for 64 bits is a special case because `UINT64_C(1) << 64` would be undefined (DT-18).

```console
$ cd numconv && gcc -std=c17 -Wall -Wextra -pedantic -fsanitize=address,undefined -g numconv.c -o numconv
$ cd numconv && ./numconv -b 8 255 0b1010 017
255
  dec  255
  hex  0xFF
  oct  0377
  bin  1111_1111
0b1010
  dec  10
  hex  0x0A
  oct  012
  bin  0000_1010
017
  dec  15
  hex  0x0F
  oct  017
  bin  0000_1111
$ cd numconv && ./numconv -b 8 -s -1 -128 127
-1
  dec  -1
  hex  0xFF
  oct  0377
  bin  1111_1111
-128
  dec  -128
  hex  0x80
  oct  0200
  bin  1000_0000
127
  dec  127
  hex  0x7F
  oct  0177
  bin  0111_1111
$ cd numconv && ./numconv -b 16 -s 0x7fff 40000 -32769 12abc; echo "exit status: $?"
0x7fff
  dec  32767
  hex  0x7FFF
  oct  077777
  bin  0111_1111_1111_1111
40000: does not fit in a signed 16-bit integer
-32769: does not fit in a signed 16-bit integer
12abc: not a valid number
exit status: 1
$ cd numconv && ./numconv -b 64 -s -9223372036854775808 && ./numconv -b 64 18446744073709551615
-9223372036854775808
  dec  -9223372036854775808
  hex  0x8000000000000000
  oct  01000000000000000000000
  bin  1000_0000_0000_0000_0000_0000_0000_0000_0000_0000_0000_0000_0000_0000_0000_0000
18446744073709551615
  dec  18446744073709551615
  hex  0xFFFFFFFFFFFFFFFF
  oct  01777777777777777777777
  bin  1111_1111_1111_1111_1111_1111_1111_1111_1111_1111_1111_1111_1111_1111_1111_1111
$ cd numconv && ./numconv -b 64 18446744073709551616; echo "exit status: $?"
18446744073709551616: not a valid number
exit status: 1
```

The sanitizer build ran every case — including the 64-bit extremes — without reporting undefined behaviour.

### Review checklist

- Is every input checked before it is converted, so no arithmetic can overflow?
- Are bit manipulations done on unsigned types?
- Do error messages go to `stderr`, results to `stdout`, and does the exit status reflect failures?
- Did you test the boundaries: 0, the maximum, the minimum, one past each, and malformed input?

### Extensions

1. Accept C23-style digit separators (`1'000'000`) and underscores (`0b1111_0000`).
2. Add `-f` to show a `double`'s sign, exponent, and mantissa fields (copy its bytes into a `uint64_t` with `memcpy`, Chapter 3).

## Answers to the Chapter 1 prediction questions

**1.1 Q1.** `1 4`: `sizeof(char)` is 1 by definition, and `'a'` has type `int` (DT-11).

**1.2 Q1.** `0x80000000` is `unsigned int` (it does not fit in `int`, and hex literals try `unsigned int` next). `2147483648` is `long` (decimal literals skip the unsigned types).

**1.3 Q1.** `c + 1` is the `int` 256 (promotion). `(unsigned char)(c + 1)` is 0. `++c` stores 0 into `c` and evaluates to 0 (the value is converted back to `unsigned char`).

**1.3 Q2.** No. `sizeof(int)` is `size_t` (unsigned long); −1 converts to `SIZE_MAX`, which is not less than 4.

**1.4 Q1.** `2` and `-2`: conversion truncates towards zero.

**1.5 Q1.** `-2 -1 1`: division truncates towards zero; `%` takes the dividend's sign.

**1.5 Q2.** `>>` binds tighter than `&`, so it is `(x >> 4) & 3` = `0xF & 3` = `3`.

**1.7 Q1.** Level 3 prints "level 3 setup", "level 2 setup", "level 1 setup" (falling through until `break`). Level 7 prints "unknown level".

**1.10 Q1.** Undefined behaviour: `va_arg` reads a third `int` that was never passed. On x86-64 the first six integer arguments travel in registers, so the function reads whatever value happens to be left in the register for the fourth integer argument (RCX) — a plausible-looking garbage number, not a crash. On other ABIs it may read stack memory. The function cannot detect the mistake; only the caller's discipline (or a sentinel/format design that the compiler can check) prevents it.

# Chapter 2 — Pointers, Arrays, and Strings

*Goal: master the three ideas that make C both powerful and dangerous — addresses, arrays that decay into pointers, and null-terminated strings.*

Pointers are where most people decide C is hard. They are not hard; they are *precise*. A pointer is a value that holds an address, with a type that says what lives there. Everything else — arrays, strings, passing large data to functions, dynamic memory, data structures, callbacks — is built from that one idea plus a few exact rules. This chapter teaches those rules with programs you can run, and shows the mistakes the rules exist to prevent.

**What you will be able to do after this chapter**

- Declare, initialise, dereference, and compare pointers, and explain what `NULL` is.
- Use pointer arithmetic and explain why `a[i]` and `*(a + i)` are the same thing.
- Explain array-to-pointer decay, and pass arrays to functions correctly with their lengths.
- Work with C strings safely: know where the terminator is, how big every buffer is, and which library functions can overflow.
- Read and write `const`-qualified pointer declarations, pointers to pointers, and function pointers.
- Use `void *` and callbacks, as `qsort` does.

## Topic 2A — Addresses and pointers

## Unit 2.1 — Pointers

**One-line summary:** A pointer is an object whose value is the address of another object (or function), and whose type says what type of object it points to. `&x` gives the address of `x`; `*p` gives the object `p` points to; a pointer that points nowhere should hold the **null pointer** value, `NULL`.

```c
// file: ptr.c
#include <stdio.h>

int main(void) {
    int x = 10;
    int *p = &x;            /* p holds the address of x */

    printf("x = %d, &x = %p\n", x, (void *)&x);
    printf("p = %p, *p = %d\n", (void *)p, *p);

    *p = 20;                /* writes x through p */
    printf("after *p = 20: x = %d\n", x);

    int y = 5;
    p = &y;                 /* p can be re-pointed */
    *p += 1;
    printf("y = %d\n", y);

    int *q = NULL;          /* points to nothing */
    if (q == NULL)
        printf("q is null; dereferencing it would be undefined\n");
    printf("sizeof(p) = %zu, sizeof(*p) = %zu\n", sizeof p, sizeof *p);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra ptr.c -o ptr && ./ptr
x = 10, &x = 0x7ffed52f1da0
p = 0x7ffed52f1da0, *p = 10
after *p = 20: x = 20
y = 6
q is null; dereferencing it would be undefined
sizeof(p) = 8, sizeof(*p) = 4
```

### Reading declarations

`int *p` declares `p` as "pointer to `int`". The `*` belongs to the declarator, not the type, which matters when declaring several names: `int *a, b;` declares a pointer `a` and an **int** `b`. Declare one pointer per line.

In an expression, `*` is the **indirection** (dereference) operator: `*p` *is* the object `p` points to — you can read it, assign to it, take its address again. `&` is the address-of operator; `&*p` is just `p`.

### What a pointer's type does

The address is just a number, but the type tells the compiler how to interpret what is there: `*p` for an `int *` reads `sizeof(int)` bytes as an `int`; for a `double *`, 8 bytes as a `double`. The type also scales pointer arithmetic (Unit 2.2). Converting between pointer types is allowed with a cast, but *accessing* an object through a pointer of the wrong type is usually undefined (the strict aliasing rule, Chapter 3).

### The null pointer

`NULL` (from `<stddef.h>`, `<stdio.h>`, and others) is a null pointer constant: `0` or `((void *)0)`. A null pointer compares unequal to any pointer to an object or function. Dereferencing one is undefined — on Linux it crashes because page 0 is never mapped, but on embedded systems address 0 may be real memory. In C23, `nullptr` is a keyword of type `nullptr_t`, the preferred spelling.

`if (p)` and `if (p != NULL)` are equivalent. A pointer variable with automatic storage that is not initialised is *not* null; it holds an indeterminate value (a **wild pointer**). Initialise every pointer, to `NULL` if nothing else.

### Deep & Tricky

**DT-27. `int* a, b;` declares one pointer and one `int`.** *(Guarantee: C declaration syntax)* The `*` binds to each declarator separately. Writing `int* a` suggests otherwise; that is why most C style guides write `int *a`. `-Wall` will not help you here, but the type mismatch usually surfaces in the next line that uses `b`.

**DT-28. A null pointer is not necessarily all-zero bits, but `0` in source code is always a null pointer constant.** *(Guarantee: C standard)* The standard allows a machine where the null pointer's representation is not zero; the compiler converts the constant `0` to it. Consequence: `memset(&p, 0, sizeof p)` or `calloc` is not *guaranteed* to produce null pointers, though it does on every mainstream platform (POSIX requires it).

### Mastery check

1. **Predict.** After `int a = 1, b = 2; int *p = &a; int *q = &b; *p = *q; p = q; *p = 7;`, what are `a` and `b`?
2. **Explain.** Why is `int *p; *p = 5;` dangerous, and what would happen on Linux most of the time?

## Unit 2.2 — Pointer arithmetic and arrays

**One-line summary:** Adding an integer `n` to a pointer moves it by `n` *elements*, not bytes; `a[i]` is defined as `*(a + i)`; and pointer arithmetic is only valid within one array (plus one position past its end). Subtracting two pointers into the same array gives the number of elements between them, as a `ptrdiff_t`.

```c
// file: arith.c
#include <stddef.h>
#include <stdio.h>

int main(void) {
    int a[5] = {10, 20, 30, 40, 50};
    int *p = a;                       /* same as &a[0] */

    printf("p     = %p -> %d\n", (void *)p, *p);
    printf("p + 1 = %p -> %d  (moved %zu bytes)\n", (void *)(p + 1), *(p + 1), sizeof *p);
    printf("a[3] = %d, *(a + 3) = %d, 3[a] = %d\n", a[3], *(a + 3), 3[a]);

    int *end = a + 5;                 /* one past the end: valid to form and compare */
    ptrdiff_t n = end - a;
    printf("end - a = %td elements\n", n);

    int sum = 0;
    for (const int *it = a; it != end; it++)   /* the idiomatic pointer loop */
        sum += *it;
    printf("sum = %d\n", sum);

    double d[3];
    printf("double pointer step: %td bytes\n", (char *)(d + 1) - (char *)d);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra arith.c -o arith && ./arith
p     = 0x7ffd61bf6260 -> 10
p + 1 = 0x7ffd61bf6264 -> 20  (moved 4 bytes)
a[3] = 40, *(a + 3) = 40, 3[a] = 40
end - a = 5 elements
sum = 150
double pointer step: 8 bytes
```

Since `a[i]` means `*(a + i)`, and addition is commutative, `3[a]` is legal and equal to `a[3]` — a curiosity that proves the definition, never something to write.

### The rules of pointer arithmetic

For a pointer `p` into an array of `N` elements:

- `p + k` is valid if the result points to an element of the same array **or one past the last element**. Forming any other address is undefined — even without dereferencing it. (`a - 1` is undefined; loops that count down with a pointer must not step before the first element.)
- The one-past-the-end pointer may be compared and subtracted, but not dereferenced.
- Subtracting two pointers is defined only if both point into (or one past) the same array; the result is `ptrdiff_t` (`%td`).
- Relational comparisons (`<`, `>`) between pointers are defined only within the same array or struct; equality (`==`) is defined for any two pointers.
- A single non-array object behaves like an array of one element: `&x + 1` is a valid one-past pointer.

### Arrays are not pointers

An array is a sequence of objects stored contiguously. A pointer is one object holding one address. They are different:

```c
// file: notptr.c
#include <stdio.h>

int main(void) {
    int a[5] = {1, 2, 3, 4, 5};
    int *p = a;
    printf("sizeof a = %zu, sizeof p = %zu\n", sizeof a, sizeof p);
    printf("&a = %p, &a + 1 = %p (whole-array step: %td bytes)\n",
           (void *)&a, (void *)(&a + 1), (char *)(&a + 1) - (char *)&a);
    printf("number of elements: %zu\n", sizeof a / sizeof a[0]);
    /* a = p;  would not compile: an array cannot be assigned */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra notptr.c -o notptr && ./notptr
sizeof a = 20, sizeof p = 8
&a = 0x7ffdee3f8a80, &a + 1 = 0x7ffdee3f8a94 (whole-array step: 20 bytes)
number of elements: 5
```

`sizeof a` is the size of the whole array (20 bytes); `&a` has type "pointer to array of 5 `int`", so `&a + 1` steps over the entire array. In most other expressions, though, the array is converted to a pointer to its first element — the next unit.

<!-- expanded -->
### Going deeper: the one-past-the-end rule

Pointer arithmetic is defined only inside one array object — and at exactly one position after its last element. For `int a[5]`, the pointers `a + 0` to `a + 5` may be formed, compared, and subtracted; `a + 5` may not be dereferenced. Forming `a - 1` or `a + 6` is undefined behaviour *even if the pointer is never used to read memory*. The rule exists because C runs on machines where addresses are not simple numbers (segmented memory, tagged pointers, hardware bounds checks such as CHERI), and it lets the optimiser assume that `p + i` stays within the same object.

The rule has a practical consequence for loops that walk backwards:

```c
// file: backwards.c
#include <stdio.h>

static void print_reversed(const int *a, size_t n) {
    /* correct: compare before stepping, so the pointer never goes below a */
    for (const int *p = a + n; p != a; ) {
        --p;
        printf("%d ", *p);
    }
    printf("\n");
}

int main(void) {
    int v[] = {1, 2, 3, 4, 5};
    print_reversed(v, 5);

    int *first = &v[0], *end = v + 5;            /* end: one past the last element - fine */
    printf("elements between first and end: %td\n", end - first);   /* ptrdiff_t, printed with %td */
    printf("end == &v[5]? %s (but *end must never be read)\n", end == &v[5] ? "yes" : "no");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra backwards.c -o backwards && ./backwards
5 4 3 2 1 
elements between first and end: 5
end == &v[5]? yes (but *end must never be read)
```

The tempting version, `for (const int *p = a + n - 1; p >= a; p--)`, performs `p--` when `p == a` and so computes `a - 1`: undefined, even though the loop then stops. On ordinary hardware it "works", which is exactly why it survives review. The subtraction of two pointers has type `ptrdiff_t` (a signed type from `<stddef.h>`, printed with `%td`) and is defined only for pointers into the same array.

### Deep & Tricky

**DT-29. Comparing or subtracting pointers into *different* objects is undefined (for `<`, `>`, `-`) even when both pointers are valid — so `if (p >= buf && p < buf + size)` is not a portable way to test whether an arbitrary pointer points into `buf`.** *(Guarantee: C17 §6.5.6 ¶9 for subtraction and §6.5.8 ¶5 for relational operators: both pointers must point into, or one past, the same array object; `==` and `!=` are defined for any two valid pointers)* Allocators and debuggers do this anyway, by converting to `uintptr_t` first (implementation-defined, and correct on flat-memory platforms). Ordinary code should not need it: keep track of which buffer a pointer came from instead of asking afterwards.

### Mastery check

1. **Predict.** For `short s[4];` at address `0x1000`, what are `s + 2`, `&s[3]`, and `(char *)s + 2`?
2. **Explain.** Why is it legal to form `a + 5` for a 5-element array but not `a + 6`?

## Unit 2.3 — Array decay and arrays as function parameters

**One-line summary:** In almost every expression, an array is converted ("**decays**") to a pointer to its first element; the exceptions are `sizeof`, unary `&`, and a string literal initialising an array. A function parameter declared as an array *is* a pointer, so functions never know an array's length unless you pass it.

```c
// file: decay.c
#include <stdio.h>

static void show(int arr[10]) {          /* the 10 is ignored: arr is an int * */
    printf("inside show: sizeof arr = %zu\n", sizeof arr);
}

static int sum(const int *a, size_t n) {  /* the honest signature */
    int total = 0;
    for (size_t i = 0; i < n; i++)
        total += a[i];
    return total;
}

static void fill(int n, int a[n], int value) {   /* C99 VLA parameter documents the length */
    for (int i = 0; i < n; i++)
        a[i] = value;
}

#define COUNT(a) (sizeof(a) / sizeof((a)[0]))    /* only for real arrays! */

int main(void) {
    int v[3] = {1, 2, 3};
    printf("in main: sizeof v = %zu, COUNT(v) = %zu\n", sizeof v, COUNT(v));
    show(v);
    printf("sum = %d\n", sum(v, COUNT(v)));
    fill(3, v, 7);
    printf("after fill: %d %d %d\n", v[0], v[1], v[2]);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra decay.c -o decay && ./decay
decay.c: In function ‘show’:
decay.c:5:54: warning: ‘sizeof’ on array function parameter ‘arr’ will return size of ‘int *’ [-Wsizeof-array-argument]
    5 |     printf("inside show: sizeof arr = %zu\n", sizeof arr);
      |                                                      ^~~
decay.c:4:22: note: declared here
    4 | static void show(int arr[10]) {          /* the 10 is ignored: arr is an int * */
      |                  ~~~~^~~~~~~
decay.c: In function ‘main’:
decay.c:25:5: warning: ‘show’ accessing 40 bytes in a region of size 12 [-Wstringop-overflow=]
   25 |     show(v);
      |     ^~~~~~~
decay.c:25:5: note: referencing argument 1 of type ‘int[10]’
decay.c:4:13: note: in a call to function ‘show’
    4 | static void show(int arr[10]) {          /* the 10 is ignored: arr is an int * */
      |             ^~~~
in main: sizeof v = 12, COUNT(v) = 3
inside show: sizeof arr = 8
sum = 6
after fill: 7 7 7
```

GCC warns that `sizeof arr` in `show` returns the size of a pointer (`-Wsizeof-array-argument`), and — because the parameter was written `int arr[10]` — that `show` may access 40 bytes of a 12-byte array (`-Wstringop-overflow`). The size in the parameter is ignored by the language but used by GCC's warnings, which is a reason to write sizes in parameters when they are fixed. The `COUNT` macro is a classic and safe *only* when applied to a real array; applied to a pointer it silently computes 8 / 4 = 2. (GCC and Clang offer `__builtin_types_compatible_p` tricks to reject pointers at compile time; C23 has no standard `countof` yet, though it is coming in the next standard.)

### Why decay exists

C was designed for machines where copying whole arrays was expensive, and to keep the language small. Making arrays decay to pointers means passing an array is cheap (one address), and indexing through a pointer and through an array is the same operation. The cost is that **lengths are lost** — the root cause of most buffer overflows in C's history.

### Multidimensional arrays

```c
// file: matrix.c
#include <stdio.h>

#define ROWS 2
#define COLS 3

static void print_fixed(int m[][COLS], int rows) {   /* m is int (*)[COLS] */
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < COLS; c++)
            printf("%3d", m[r][c]);
        printf("\n");
    }
}

static void print_vla(int rows, int cols, int m[rows][cols]) {  /* any size, C99 */
    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            printf("%d%c", m[r][c], c == cols - 1 ? '\n' : ' ');
}

int main(void) {
    int m[ROWS][COLS] = {{1, 2, 3}, {4, 5, 6}};
    print_fixed(m, ROWS);
    print_vla(ROWS, COLS, m);
    printf("row-major: m[1][0] is element %td of the flat block\n", &m[1][0] - &m[0][0]);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra matrix.c -o matrix && ./matrix
  1  2  3
  4  5  6
1 2 3
4 5 6
row-major: m[1][0] is element 3 of the flat block
```

A 2-D array is an array of arrays, stored **row-major**: all of row 0, then all of row 1. It decays to a pointer to its first *row* (`int (*)[3]`), so the callee must know every dimension except the first. Variable-length array parameters (`int m[rows][cols]`) pass the dimensions at run time. (Local VLAs — `int buf[n];` with `n` from input — are another matter: they allocate an unbounded amount of stack and became optional in C11. Avoid them for anything with untrusted size.)

### Deep & Tricky

**DT-30. `sizeof` on an array parameter gives the pointer size.** *(Guarantee: parameter adjustment)* A parameter written `int a[]` or `int a[100]` is adjusted to `int *a`. `sizeof a` is 8, and loops written as `for (i = 0; i < sizeof a / sizeof a[0]; i++)` process two elements. Always pass the length explicitly (`size_t n`), and compute `COUNT` only where the array is declared.

**DT-31. `int a[3][4]` decays to `int (*)[4]`, not `int **`.** *(Guarantee)* A 2-D array is one contiguous block; a pointer-to-pointer refers to an array of separate row pointers. Passing `m` to a function taking `int **` is a type error (and casting it away crashes, because the function would interpret the first numbers as addresses). Use `int (*)[COLS]`, a VLA parameter, or a flat array with `m[r * cols + c]`.

### Mastery check

1. **Predict.** What does `COUNT` give for `int *p = v;` in `decay.c`, and why?
2. **Write.** `void transpose(int r, int c, const int in[r][c], int out[c][r])` and test it on a 2×3 matrix.

## Topic 2B — Strings

## Unit 2.4 — C strings

**One-line summary:** A C string is an array of `char` ending with a **null character** `'\0'`. Its length is the number of characters before the terminator, found by scanning (`strlen` is O(n)), and every buffer that holds a string needs room for one more byte than its length. String literals are stored in read-only memory: never modify them.

```c
// file: strings.c
#include <stdio.h>
#include <string.h>

int main(void) {
    char a[] = "cat";              /* array of 4 chars: 'c','a','t','\0' (modifiable copy) */
    const char *p = "cat";         /* pointer to a literal in read-only memory */
    char b[10] = "dog";            /* rest of the array is zero-filled */
    char c[3] = "cat";             /* legal in C, but NO terminator fits: not a string! */

    printf("sizeof a = %zu, strlen(a) = %zu\n", sizeof a, strlen(a));
    printf("sizeof p = %zu, strlen(p) = %zu\n", sizeof p, strlen(p));
    printf("sizeof b = %zu, strlen(b) = %zu\n", sizeof b, strlen(b));
    printf("sizeof c = %zu (c is not null-terminated)\n", sizeof c);

    a[0] = 'b';                    /* fine: a is our own array */
    printf("a = %s\n", a);

    printf("bytes of a:");
    for (size_t i = 0; i < sizeof a; i++)
        printf(" %02x", (unsigned char)a[i]);
    printf("\n");

    if (strcmp(a, "bat") == 0)     /* == would compare addresses, not contents */
        printf("strcmp: a equals \"bat\"\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra strings.c -o strings && ./strings
sizeof a = 4, strlen(a) = 3
sizeof p = 8, strlen(p) = 3
sizeof b = 10, strlen(b) = 3
sizeof c = 3 (c is not null-terminated)
a = bat
bytes of a: 62 61 74 00
strcmp: a equals "bat"
```

`char c[3] = "cat";` is the one case where C silently drops the terminator (C++ rejects it). Passing `c` to `strlen` or `printf("%s")` reads past the end. Let the compiler size string arrays (`char a[] = "…"`) or make them bigger than needed.

### String literals are read-only

```c
// file: literal.c
int main(void) {
    char *s = "hello";   /* legal C (for historical reasons), but s points to read-only data */
    s[0] = 'H';          /* undefined behaviour: crashes on Linux */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra literal.c -o literal && ./literal; echo "exit status: $?"
bash: line 1:  1090 Segmentation fault      ./literal
exit status: 139
```

The literal lives in `.rodata`, mapped read-only, so the write faults. Always declare pointers to literals as `const char *`; then `s[0] = 'H'` becomes a compile error. (GCC's `-Wwrite-strings` makes literals `const char[]` to enforce this.)

### The standard string functions and their traps

| Function | Does | Trap |
| --- | --- | --- |
| `strlen(s)` | counts characters before `'\0'` | O(n) every call; UB if no terminator |
| `strcpy(d, s)` | copies including `'\0'` | no size: overflows `d` if `s` is too long |
| `strncpy(d, s, n)` | copies at most `n` bytes | does **not** terminate if `s` has ≥ `n` characters; pads with zeros (slow) |
| `strcat(d, s)` | appends | overflows; O(len(d)) to find the end |
| `strcmp(a, b)` | <0, 0, >0 by byte values | compare result with 0, not with −1/1 |
| `strchr`, `strrchr`, `strstr` | find char / last char / substring | return `char *` even for `const` input |
| `snprintf(d, n, …)` | formatted write, at most `n` bytes including `'\0'` | returns the length it *wanted*: check for truncation |
| `memcpy`, `memmove`, `memset`, `memcmp` | work on bytes with explicit lengths | `memcpy` with overlapping regions is UB: use `memmove` |

The safe, standard way to copy or build strings with a known buffer size is `snprintf`:

```c
// file: safecopy.c
#include <stdio.h>
#include <string.h>

/* copy_str copies src into dst (size dstsz), always terminating.
   Returns 0 if everything fit, -1 if the result was truncated. */
static int copy_str(char *dst, size_t dstsz, const char *src) {
    int n = snprintf(dst, dstsz, "%s", src);
    return (n >= 0 && (size_t)n < dstsz) ? 0 : -1;
}

int main(void) {
    char small[6];
    printf("%d '%s'\n", copy_str(small, sizeof small, "hello"), small);
    printf("%d '%s'\n", copy_str(small, sizeof small, "hello world"), small);

    char path[32];
    int n = snprintf(path, sizeof path, "%s/%s.txt", "/home/user", "notes");
    printf("path='%s' (needed %d chars)\n", path, n);

    char overlap[] = "abcdef";
    memmove(overlap + 2, overlap, 4);  /* overlapping: memmove, never memcpy */
    printf("memmove result: %s\n", overlap);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra safecopy.c -o safecopy && ./safecopy
0 'hello'
-1 'hello'
path='/home/user/notes.txt' (needed 20 chars)
memmove result: ababcd
```

(BSD's `strlcpy` and `strlcat`, now in glibc 2.38 and POSIX 2024, do the same job directly.)

### Characters and `<ctype.h>`

`isalpha`, `isdigit`, `isspace`, `toupper`, and friends take an `int` that must be either `EOF` or a value representable as `unsigned char`. Passing a plain `char` with a negative value (any byte ≥ 128 on signed-`char` platforms, e.g. UTF-8 text) is undefined. Always cast: `isalpha((unsigned char)c)`.

### Deep & Tricky

**DT-32. `strncpy` is not a safe `strcpy`.** *(Library behaviour)* It was designed for fixed-width fields in old Unix directory entries: it copies at most `n` bytes and pads with zeros, and if the source is `n` characters or longer, the destination is **not null-terminated**. Use `snprintf`, `strlcpy`, or `memcpy` with explicit length handling instead.

**DT-33. Comparing strings with `==` compares addresses.** *(Guarantee)* `if (s == "yes")` compares the pointer `s` with the address of a literal; it is almost always false (and the result for two identical literals is unspecified — the compiler may or may not merge them). Use `strcmp(s, "yes") == 0`.

**DT-34. `<ctype.h>` functions need `unsigned char` values.** *(Guarantee: UB for negative non-EOF values)* `toupper(c)` with `char c = 'é'` in UTF-8 (byte 0xC3, which is −61 as a signed `char`) is undefined; glibc happens to tolerate it, other C libraries index a table with a negative number. Cast to `unsigned char` first.

<!-- expanded -->
### Going deeper: the hidden cost of `strlen`

A C string does not know its length: `strlen` walks the bytes until it finds `'\0'`, so it costs time proportional to the length. Calling it in a loop condition makes an innocent-looking loop quadratic:

```c
// file: quadratic.c
#define _POSIX_C_SOURCE 200809L        /* clock_gettime is POSIX, hidden by -std=c17 otherwise */
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

static void upper_slow(char *s) {
    for (size_t i = 0; i < strlen(s); i++)          /* strlen runs on every iteration */
        s[i] = (char)toupper((unsigned char)s[i]);
}

static void upper_fast(char *s) {
    size_t n = strlen(s);                            /* once */
    for (size_t i = 0; i < n; i++)
        s[i] = (char)toupper((unsigned char)s[i]);
}

static double ms(void (*f)(char *), char *s) {
    struct timespec a, b;
    clock_gettime(CLOCK_MONOTONIC, &a);
    f(s);
    clock_gettime(CLOCK_MONOTONIC, &b);
    return (b.tv_sec - a.tv_sec) * 1e3 + (b.tv_nsec - a.tv_nsec) / 1e6;
}

int main(void) {
    for (size_t n = 25000; n <= 100000; n *= 2) {
        char *s = malloc(n + 1);
        memset(s, 'a', n);
        s[n] = '\0';
        double slow = ms(upper_slow, s);
        memset(s, 'a', n);
        double fast = ms(upper_fast, s);
        printf("n = %6zu: strlen in the condition %8.2f ms, strlen once %6.3f ms\n", n, slow, fast);
        free(s);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 quadratic.c -o quadratic && ./quadratic
n =  25000: strlen in the condition     3.27 ms, strlen once  0.009 ms
n =  50000: strlen in the condition    15.53 ms, strlen once  0.016 ms
n = 100000: strlen in the condition    71.66 ms, strlen once  0.029 ms
```

Each doubling of `n` roughly quadruples the slow version's time and doubles the fast one's. The optimiser cannot hoist `strlen` out of the loop by itself here, because the loop writes into `s` — as far as the compiler knows, a write could change where the terminator is. Real programs that build strings with repeated `strcat` have the same problem (each call must find the end again); keep the length in a variable, or use a buffer type that stores it (the string library of this chapter's project does).

### Memory view of three ways to hold "hi"

```text
char a[] = "hi";        a: [ 'h' ][ 'i' ][ '\0' ]          3 bytes, writable copy on the stack
char *p = "hi";         p: [ address ] ----> "hi\0"         8-byte pointer to a literal in read-only memory
char b[8] = "hi";       b: [ 'h' ][ 'i' ][ '\0' ][ 0 ][ 0 ][ 0 ][ 0 ][ 0 ]   the rest is zero-filled
```

`sizeof a` is 3, `sizeof p` is 8, `sizeof b` is 8, and `strlen` is 2 for all three. Writing `a[0] = 'H'` and `b[0] = 'H'` is fine; writing `p[0] = 'H'` modifies a string literal — undefined behaviour, and on Linux a segmentation fault, because literals are placed in a read-only segment.

### Mastery check

1. **Predict.** `char s[] = "hi\0there";` What are `sizeof s` and `strlen(s)`?
2. **Explain.** Why is `strlen` called inside a loop condition (`for (i = 0; i < strlen(s); i++)`) a performance bug?
3. **Write.** `size_t count_char(const char *s, char c)` using pointer iteration and no indexes.

## Topic 2C — Pointer types in depth

## Unit 2.5 — `const` and pointers

**One-line summary:** `const` can qualify the pointed-to object (`const int *p`: you cannot modify `*p` through `p`) or the pointer itself (`int *const p`: you cannot re-point `p`), or both. Read declarations **right to left**, and make every pointer parameter that does not modify its target `const`.

```c
// file: constptr.c
#include <stdio.h>

int main(void) {
    int x = 1, y = 2;

    const int *p1 = &x;     /* pointer to const int: *p1 is read-only */
    p1 = &y;                /* OK: p1 itself may change */
    /* *p1 = 5;   error: assignment of read-only location */

    int *const p2 = &x;     /* const pointer to int */
    *p2 = 5;                /* OK: the int may change */
    /* p2 = &y;   error: assignment of read-only variable */

    const int *const p3 = &x;   /* neither may change */
    printf("x=%d *p1=%d *p3=%d\n", x, *p1, *p3);

    /* const protects the view, not the object: x is still modifiable through x */
    x = 9;
    printf("through p3 now: %d\n", *p3);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra constptr.c -o constptr && ./constptr
x=5 *p1=2 *p3=5
through p3 now: 9
```

| Declaration | Read right to left | Can change `*p`? | Can change `p`? |
| --- | --- | --- | --- |
| `const int *p` (= `int const *p`) | p is a pointer to int that is const | no | yes |
| `int *const p` | p is a const pointer to int | yes | no |
| `const int *const p` | p is a const pointer to const int | no | no |

### `const`-correct interfaces

`size_t strlen(const char *s)` promises not to modify the string, so it accepts both modifiable and read-only strings. A function that takes `char *` but does not modify its argument forces every caller with a `const char *` to cast — and casts hide real mistakes. Rule: **add `const` to every pointer parameter whose target the function does not modify.**

Converting `T *` to `const T *` is implicit and safe. The other direction requires a cast, and writing through the result is undefined if the object was originally defined `const`.

### Deep & Tricky

**DT-35. `char **` does not convert to `const char **`.** *(Guarantee: type rules, for a soundness reason)* It looks like adding `const`, but allowing it would let you store a `const char *` into a `char *` slot and then modify a read-only string through it. C requires a cast (C++ allows `const char *const *`). This is why `main`'s `argv` is `char *argv[]` and functions taking lists of strings often declare `const char *const *`.

### Mastery check

1. **Explain.** Read aloud: `const char *const *names`.
2. **Write.** Fix the signatures of `int find(char *haystack, char *needle)` and `void upcase(char *s)` for const-correctness.

## Unit 2.6 — Pointers to pointers and arrays of pointers

**One-line summary:** A pointer to a pointer (`int **`) lets a function change a caller's *pointer*; an array of pointers (`char *names[]`) holds independent objects of different lengths. `argv` is both: an array of `char *`, received as a `char **`.

```c
// file: pp.c
#include <stdio.h>
#include <string.h>

/* find_first sets *out to point at the first word starting with c; returns 0 or -1. */
static int find_first(const char *words[], size_t n, char c, const char **out) {
    for (size_t i = 0; i < n; i++) {
        if (words[i][0] == c) {
            *out = words[i];          /* modify the caller's pointer */
            return 0;
        }
    }
    return -1;
}

int main(void) {
    const char *words[] = {"apple", "banana", "cherry", "blueberry"};   /* jagged: different lengths */
    size_t n = sizeof words / sizeof words[0];

    const char *found = NULL;
    if (find_first(words, n, 'b', &found) == 0)
        printf("first b-word: %s\n", found);

    for (size_t i = 0; i < n; i++)
        printf("words[%zu] = %-10s at %p, length %zu\n", i, words[i], (void *)words[i], strlen(words[i]));

    char grid[3][7] = {"apple", "banana", "kiwi"};  /* contrast: fixed 7-byte rows */
    printf("sizeof words = %zu (4 pointers), sizeof grid = %zu (3 rows x 7)\n", sizeof words, sizeof grid);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra pp.c -o pp && ./pp
first b-word: banana
words[0] = apple      at 0x563b58b38008, length 5
words[1] = banana     at 0x563b58b3800e, length 6
words[2] = cherry     at 0x563b58b38015, length 6
words[3] = blueberry  at 0x563b58b3801c, length 9
sizeof words = 32 (4 pointers), sizeof grid = 21 (3 rows x 7)
```

`words` stores four addresses of literals scattered in read-only memory; `grid` stores 21 bytes of characters in one block. Choose the array of pointers when strings vary in length or already exist elsewhere; choose the 2-D array for small fixed-width records.

<!-- expanded -->
### Going deeper: two-dimensional arrays on the heap

A matrix whose size is known only at run time can be built two ways, with different memory layouts and costs.

```c
// file: heapmatrix.c
#include <stdio.h>
#include <stdlib.h>

/* Way 1: an array of row pointers. rows + 1 allocations, rows may differ in length. */
static double **alloc_rows(size_t rows, size_t cols) {
    double **m = malloc(rows * sizeof *m);
    if (!m) return NULL;
    for (size_t r = 0; r < rows; r++) {
        m[r] = calloc(cols, sizeof **m);
        if (!m[r]) {                                  /* free what was allocated so far */
            while (r-- > 0) free(m[r]);
            free(m);
            return NULL;
        }
    }
    return m;
}

static void free_rows(double **m, size_t rows) {
    for (size_t r = 0; r < rows; r++) free(m[r]);    /* rows first, then the pointer array */
    free(m);
}

int main(void) {
    size_t rows = 3, cols = 4;

    double **a = alloc_rows(rows, cols);
    a[2][3] = 7.5;
    printf("way 1: a[2][3] = %.1f; row 0 at %p, row 1 at %p (separate blocks)\n",
           a[2][3], (void *)a[0], (void *)a[1]);
    free_rows(a, rows);

    /* Way 2: one block, used through a pointer to a variably modified array type (C99, optional in C11+). */
    double (*b)[cols] = calloc(rows, sizeof *b);      /* sizeof *b == cols * sizeof(double) */
    b[2][3] = 7.5;
    printf("way 2: b[2][3] = %.1f; sizeof *b = %zu; row 1 starts %td bytes after row 0\n",
           b[2][3], sizeof *b, (char *)b[1] - (char *)b[0]);
    free(b);                                          /* one allocation, one free */

    /* Way 3 (most portable): one block indexed by hand */
    double *c = calloc(rows * cols, sizeof *c);
    c[2 * cols + 3] = 7.5;
    printf("way 3: c[2*cols+3] = %.1f\n", c[2 * cols + 3]);
    free(c);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address -g heapmatrix.c -o heapmatrix && ./heapmatrix
way 1: a[2][3] = 7.5; row 0 at 0x503000000070, row 1 at 0x5030000000a0 (separate blocks)
way 2: b[2][3] = 7.5; sizeof *b = 32; row 1 starts 32 bytes after row 0
way 3: c[2*cols+3] = 7.5
```

```text
way 1 (array of row pointers)          way 2/3 (one contiguous block)
m --> [ * ][ * ][ * ]                  b --> [ r0c0 r0c1 r0c2 r0c3 | r1c0 ... | r2c0 ... r2c3 ]
        |    |    \
        v    v     v
      [row0][row1][row2]   (anywhere on the heap)
```

The contiguous block is faster to allocate and free, friendlier to the CPU cache when a loop walks the whole matrix (Chapter 11), and can be handed to libraries that expect a single buffer. The array of pointers is the right choice when rows have different lengths (a "jagged" array, like `argv`) or are swapped around by exchanging pointers. Way 2 needs variable-length array *types*, which C11 made optional (`__STDC_NO_VLA__`); GCC and Clang support them, MSVC does not — which is why way 3 is common in portable code.

### Mastery check

1. **Predict.** In `pp.c`, what is the type of `words[1]`? Of `words[1][0]`? Of `&words[1]`?
2. **Write.** `void sort_words(const char *words[], size_t n)` using insertion sort and `strcmp` (it moves pointers, not characters).

## Unit 2.7 — `void *` and function pointers

**One-line summary:** `void *` is a generic object pointer: any object pointer converts to and from it without a cast, but it cannot be dereferenced or used in arithmetic. A **function pointer** holds the address of a function and can be called; together they give C generic algorithms and callbacks, such as `qsort`.

```c
// file: callbacks.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int cmp_int(const void *a, const void *b) {
    int x = *(const int *)a, y = *(const int *)b;
    return (x > y) - (x < y);        /* not x - y: that can overflow (DT-36) */
}

static int cmp_str(const void *a, const void *b) {
    /* the array holds const char *, so each element pointer is a const char *const * */
    return strcmp(*(const char *const *)a, *(const char *const *)b);
}

static int twice(int x) { return 2 * x; }
static int square(int x) { return x * x; }

static void apply(int *a, size_t n, int (*f)(int)) {
    for (size_t i = 0; i < n; i++)
        a[i] = f(a[i]);
}

int main(void) {
    int nums[] = {42, -7, 13, 0, 2147483647, -2147483647 - 1};
    size_t n = sizeof nums / sizeof nums[0];
    qsort(nums, n, sizeof nums[0], cmp_int);
    for (size_t i = 0; i < n; i++)
        printf("%d ", nums[i]);
    printf("\n");

    const char *names[] = {"pear", "apple", "fig"};
    qsort(names, 3, sizeof names[0], cmp_str);
    printf("%s %s %s\n", names[0], names[1], names[2]);

    int (*ops[])(int) = {twice, square};      /* array of function pointers */
    int v[] = {1, 2, 3};
    for (size_t k = 0; k < 2; k++) {
        apply(v, 3, ops[k]);
        printf("after op %zu: %d %d %d\n", k, v[0], v[1], v[2]);
    }

    typedef int (*unary_fn)(int);             /* typedef makes the syntax bearable */
    unary_fn f = square;
    printf("f(9) = %d\n", f(9));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra callbacks.c -o callbacks && ./callbacks
-2147483648 -7 0 13 42 2147483647 
apple fig pear
after op 0: 2 4 6
after op 1: 4 16 36
f(9) = 81
```

How `qsort` can sort anything: it receives the array as `void *`, the element size, and a comparison function. It moves elements by copying `size` bytes, and calls your comparator with pointers to two elements. The comparator converts the `void *` back to the real type.

Reading function-pointer declarations: `int (*f)(int)` — "f is a pointer to a function taking `int` and returning `int`". The parentheses around `*f` matter: `int *f(int)` declares a *function* returning `int *`. A `typedef` for the pointer type is the cure for unreadable declarations such as `void (*signal(int, void (*)(int)))(int)` (the real prototype of `signal`, Chapter 8).

### Deep & Tricky

**DT-36. `return a - b;` is a broken comparator.** *(Guarantee: signed overflow is UB)* For `a = INT_MAX` and `b = -1`, `a - b` overflows. The program above sorts `INT_MAX` and `INT_MIN` correctly only because it uses `(x > y) - (x < y)`. The same bug appears in Java and Go code, but in C it is undefined behaviour, not just a wrong answer.

**DT-37. A function pointer cannot portably be stored in a `void *`.** *(Guarantee: ISO C; POSIX extends it)* The C standard keeps object and function pointers separate (on some architectures they differ in size). POSIX requires the conversion to work because `dlsym` returns `void *`. Store function pointers in function-pointer types; any function pointer type can be converted to another and back.

<!-- expanded -->
### Going deeper: callbacks with context, and dispatch tables

A plain function pointer cannot carry data with it. The C convention is to pass a `void *` "context" next to the callback; the library passes it back unchanged, and the callback converts it to the type it knows. This is how C gets the effect of closures.

```c
// file: context.c
#include <stdio.h>
#include <string.h>

typedef int (*visit_fn)(const char *word, void *ctx);      /* a typedef makes the type readable */

static int for_each_word(const char *text, visit_fn visit, void *ctx) {
    char buf[64];
    while (*text) {
        while (*text == ' ') text++;
        size_t n = strcspn(text, " ");
        if (n == 0) break;
        if (n >= sizeof buf) n = sizeof buf - 1;
        memcpy(buf, text, n);
        buf[n] = '\0';
        if (visit(buf, ctx) != 0) return 1;                  /* callback may stop the walk */
        text += strcspn(text, " ");
    }
    return 0;
}

struct stats { int words; size_t longest; char longest_word[64]; };

static int count(const char *w, void *ctx) {
    struct stats *s = ctx;                                   /* void * converts back without a cast */
    s->words++;
    if (strlen(w) > s->longest) { s->longest = strlen(w); strcpy(s->longest_word, w); }
    return 0;
}

static int find(const char *w, void *ctx) { return strcmp(w, (const char *)ctx) == 0; }

/* a dispatch table: command name -> handler */
static int cmd_hello(const char *arg) { printf("hello, %s\n", arg); return 0; }
static int cmd_shout(const char *arg) { printf("%s!!!\n", arg); return 0; }
static const struct { const char *name; int (*run)(const char *); } commands[] = {
    {"hello", cmd_hello},
    {"shout", cmd_shout},
};

int main(void) {
    const char *text = "the quick brown fox jumps over the lazy dog";
    struct stats s = {0};
    for_each_word(text, count, &s);
    printf("%d words, longest \"%s\" (%zu letters)\n", s.words, s.longest_word, s.longest);
    printf("contains \"fox\"? %s\n", for_each_word(text, find, "fox") ? "yes" : "no");

    const char *input[][2] = {{"shout", "watch out"}, {"hello", "world"}, {"dance", "x"}};
    for (size_t i = 0; i < 3; i++) {
        size_t c;
        for (c = 0; c < sizeof commands / sizeof commands[0]; c++)
            if (strcmp(commands[c].name, input[i][0]) == 0) { commands[c].run(input[i][1]); break; }
        if (c == sizeof commands / sizeof commands[0]) printf("unknown command %s\n", input[i][0]);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra context.c -o context && ./context
9 words, longest "quick" (5 letters)
contains "fox"? yes
watch out!!!
hello, world
unknown command dance
```

The same `for_each_word` counted words with one callback and searched with another; each brought its own data through `ctx`. The dispatch table replaces a long `if`/`else` chain and is how interpreters, protocol handlers, and the Linux kernel's `file_operations` (Part II of the Linux book) are organised.

### Deep & Tricky

**DT-38. Calling a function through a pointer of a different function type is undefined behaviour, even when the parameter types "look compatible" — so passing `int cmp(const int *, const int *)` to `qsort` with a cast is wrong.** *(Guarantee: C17 §6.3.2.3 ¶8: "If a converted pointer is used to call a function whose type is not compatible with the referenced type, the behavior is undefined.")* It usually works on x86-64, because all pointers are passed the same way. It fails under control-flow integrity: Clang's `-fsanitize=cfi-icall` and the Linux kernel's kCFI check the function *type* at every indirect call and abort on a mismatch. Write the comparator with `const void *` parameters and convert inside, as Unit 2.7's `cmp_int` does.

```c
// file: badcast.c
#include <stdio.h>

typedef int (*cmp_fn)(const void *, const void *);

static int cmp_ints(const int *a, const int *b) { return (*a > *b) - (*a < *b); }

int main(void) {
    cmp_fn f = (cmp_fn)cmp_ints;               /* the cast hides the mismatch from the compiler */
    int x = 3, y = 1;
    printf("f(&x, &y) = %d\n", f(&x, &y));      /* a call through the wrong type */
    return 0;
}
```

```console
$ clang -std=c17 -O1 badcast.c -o badcast && ./badcast
f(&x, &y) = 1
$ clang -std=c17 -O1 -fsanitize=function badcast.c -o badcast-check && ./badcast-check 2>&1 | sed 's/^.*runtime error/runtime error/' | head -2
runtime error: call to function cmp_ints through pointer to incorrect function type 'int (*)(const void *, const void *)'
badcast.c: note: cmp_ints defined here
```

The ordinary build prints the right answer. With `-fsanitize=function`, Clang checks the type of the function at every indirect call made by instrumented code and reports the mismatch. (It cannot see the calls that an uninstrumented library such as the C library's `qsort` makes — one more reason not to rely on the cast "working".)

### Mastery check

1. **Explain.** Why does `cmp_str` cast to `const char *const *` rather than `const char *`?
2. **Write.** A generic `void *find(const void *base, size_t n, size_t size, const void *key, int (*cmp)(const void *, const void *))` linear search, tested on `int`s and strings.

## Chapter 2 Project — A string utility library with tests

**Goal:** write a small, safe string library — `strutil.h` and `strutil.c` — and a test program that checks every function, including edge cases, under AddressSanitizer and UndefinedBehaviorSanitizer.

### Requirements

```c
size_t su_trim(char *s);                     /* trim whitespace in place; return new length */
void   su_reverse(char *s);                  /* reverse in place */
int    su_copy(char *dst, size_t dstsz, const char *src);   /* 0 ok, -1 truncated */
size_t su_split(char *s, char sep, char *out[], size_t max); /* split in place */
int    su_replace(char *dst, size_t dstsz, const char *s,
                  const char *from, const char *to);        /* 0 ok, -1 truncated */
int    su_starts_with(const char *s, const char *prefix);
```

Rules: no function may write outside its buffers for any input; every function handles the empty string; truncation is reported, never silent; the library allocates no memory.

### Reference solution

```c
// file: strutil/strutil.h
#ifndef STRUTIL_H
#define STRUTIL_H

#include <stddef.h>

/* su_trim removes leading and trailing whitespace from s in place and
   returns the new length. */
size_t su_trim(char *s);

/* su_reverse reverses the bytes of s in place. */
void su_reverse(char *s);

/* su_copy copies src into dst (capacity dstsz bytes, including the
   terminator). Returns 0 if src fit, -1 if it was truncated. dst is
   always terminated when dstsz > 0. */
int su_copy(char *dst, size_t dstsz, const char *src);

/* su_split replaces each sep in s with '\0' and stores pointers to at most
   max fields in out. Returns the number of fields found (which may exceed
   max: only the first max are stored). */
size_t su_split(char *s, char sep, char *out[], size_t max);

/* su_replace writes s with every occurrence of from replaced by to into
   dst (capacity dstsz). Returns 0 on success, -1 if truncated or if from
   is empty. */
int su_replace(char *dst, size_t dstsz, const char *s, const char *from, const char *to);

/* su_starts_with returns 1 if s begins with prefix, else 0. */
int su_starts_with(const char *s, const char *prefix);

#endif
```

```c
// file: strutil/strutil.c
#include "strutil.h"

#include <ctype.h>
#include <string.h>

size_t su_trim(char *s) {
    char *start = s;
    while (*start != '\0' && isspace((unsigned char)*start))   /* DT-34 */
        start++;
    size_t len = strlen(start);
    while (len > 0 && isspace((unsigned char)start[len - 1]))  /* len > 0 first: no s[-1] */
        len--;
    memmove(s, start, len);          /* regions may overlap */
    s[len] = '\0';
    return len;
}

void su_reverse(char *s) {
    size_t n = strlen(s);
    if (n < 2)
        return;
    for (char *a = s, *b = s + n - 1; a < b; a++, b--) {
        char t = *a;
        *a = *b;
        *b = t;
    }
}

int su_copy(char *dst, size_t dstsz, const char *src) {
    if (dstsz == 0)
        return -1;
    size_t len = strlen(src);
    if (len >= dstsz) {
        memcpy(dst, src, dstsz - 1);
        dst[dstsz - 1] = '\0';
        return -1;
    }
    memcpy(dst, src, len + 1);
    return 0;
}

size_t su_split(char *s, char sep, char *out[], size_t max) {
    size_t count = 0;
    char *field = s;
    for (char *p = s;; p++) {
        if (*p == sep || *p == '\0') {
            int at_end = (*p == '\0');
            if (count < max)
                out[count] = field;
            count++;
            if (at_end)
                break;
            *p = '\0';
            field = p + 1;
        }
    }
    return count;
}

int su_replace(char *dst, size_t dstsz, const char *s, const char *from, const char *to) {
    size_t from_len = strlen(from), to_len = strlen(to), used = 0;
    if (from_len == 0 || dstsz == 0)
        return -1;
    while (*s != '\0') {
        const char *piece = s;
        size_t piece_len = 1;
        if (strncmp(s, from, from_len) == 0) {
            piece = to;
            piece_len = to_len;
            s += from_len;
        } else {
            s++;
        }
        if (used + piece_len >= dstsz) {        /* keep one byte for '\0' */
            dst[used] = '\0';
            return -1;
        }
        memcpy(dst + used, piece, piece_len);
        used += piece_len;
    }
    dst[used] = '\0';
    return 0;
}

int su_starts_with(const char *s, const char *prefix) {
    return strncmp(s, prefix, strlen(prefix)) == 0;
}
```

Notes on the implementation:

- `su_trim` checks `len > 0` *before* reading `start[len - 1]`, so an all-space string never reads before the buffer (and `len - 1` never wraps — Unit 1.3).
- `su_reverse` returns early for short strings: for an empty string, `s + n - 1` would be `s - 1`, an invalid pointer even if never dereferenced (Unit 2.2).
- `su_split` counts all fields but stores only `max`, so callers can detect "too many fields" without overflowing `out`.
- `su_replace` checks space *before* each copy, so truncation leaves a valid, terminated prefix.

```c
// file: strutil/test_strutil.c
#include <stdio.h>
#include <string.h>

#include "strutil.h"

static int failures = 0;

#define CHECK(cond)                                                       \
    do {                                                                  \
        if (!(cond)) {                                                    \
            fprintf(stderr, "%s:%d: CHECK failed: %s\n", __FILE__,        \
                    __LINE__, #cond);                                     \
            failures++;                                                   \
        }                                                                 \
    } while (0)

#define CHECK_STR(got, want) CHECK(strcmp((got), (want)) == 0)

static void test_trim(void) {
    struct { const char *in, *want; } cases[] = {
        {"  hello  ", "hello"}, {"hello", "hello"}, {"", ""}, {"   ", ""},
        {"\t a b \n", "a b"}, {"x", "x"},
    };
    for (size_t i = 0; i < sizeof cases / sizeof cases[0]; i++) {
        char buf[32];
        su_copy(buf, sizeof buf, cases[i].in);
        size_t n = su_trim(buf);
        CHECK_STR(buf, cases[i].want);
        CHECK(n == strlen(cases[i].want));
    }
}

static void test_reverse(void) {
    char a[] = "abc", b[] = "", c[] = "x", d[] = "abcd";
    su_reverse(a); su_reverse(b); su_reverse(c); su_reverse(d);
    CHECK_STR(a, "cba"); CHECK_STR(b, ""); CHECK_STR(c, "x"); CHECK_STR(d, "dcba");
}

static void test_copy(void) {
    char buf[4];
    CHECK(su_copy(buf, sizeof buf, "abc") == 0);
    CHECK_STR(buf, "abc");
    CHECK(su_copy(buf, sizeof buf, "abcd") == -1);
    CHECK_STR(buf, "abc");
    CHECK(su_copy(buf, 1, "abc") == -1);
    CHECK_STR(buf, "");
    CHECK(su_copy(buf, 0, "abc") == -1);
}

static void test_split(void) {
    char s[] = "a,bb,,ccc";
    char *f[8];
    size_t n = su_split(s, ',', f, 8);
    CHECK(n == 4);
    CHECK_STR(f[0], "a"); CHECK_STR(f[1], "bb"); CHECK_STR(f[2], ""); CHECK_STR(f[3], "ccc");

    char t[] = "x,y,z";
    char *g[2];
    CHECK(su_split(t, ',', g, 2) == 3);      /* reports 3, stores 2 */
    CHECK_STR(g[0], "x"); CHECK_STR(g[1], "y");

    char e[] = "";
    CHECK(su_split(e, ',', f, 8) == 1 && f[0][0] == '\0');
}

static void test_replace(void) {
    char out[32];
    CHECK(su_replace(out, sizeof out, "a cat and a cat", "cat", "dog") == 0);
    CHECK_STR(out, "a dog and a dog");
    CHECK(su_replace(out, sizeof out, "aaa", "a", "bb") == 0);
    CHECK_STR(out, "bbbbbb");
    CHECK(su_replace(out, sizeof out, "abc", "", "x") == -1);
    char tiny[5];
    CHECK(su_replace(tiny, sizeof tiny, "hello", "l", "L") == -1);
    CHECK_STR(tiny, "heLL");
    CHECK(su_replace(out, sizeof out, "", "a", "b") == 0);
    CHECK_STR(out, "");
}

static void test_starts_with(void) {
    CHECK(su_starts_with("hello", "he"));
    CHECK(su_starts_with("hello", ""));
    CHECK(!su_starts_with("he", "hello"));
    CHECK(!su_starts_with("", "a"));
}

int main(void) {
    test_trim();
    test_reverse();
    test_copy();
    test_split();
    test_replace();
    test_starts_with();
    if (failures == 0)
        printf("all strutil tests passed\n");
    return failures == 0 ? 0 : 1;
}
```

`CHECK` is wrapped in `do { … } while (0)` so that it behaves like a single statement after an `if` without braces — a macro idiom explained in Chapter 4. `#cond` turns the condition's source text into a string for the error message.

```console
$ cd strutil && gcc -std=c17 -Wall -Wextra -pedantic -g -fsanitize=address,undefined strutil.c test_strutil.c -o test_strutil && ./test_strutil
all strutil tests passed
```

To see the tests and sanitizers working, plant a bug — make `su_trim` drop its `len > 0` check — and run again:

```console
$ cd strutil && sed 's/while (len > 0 \&\& /while (/' strutil.c > strutil_bug.c && gcc -std=c17 -g -fsanitize=address,undefined strutil_bug.c test_strutil.c -o test_bug && ./test_bug 2>&1 | grep -E 'ERROR|#0|#1' | head -n 3
==1164==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffafc10009f at pc 0x562865ae87c8 bp 0x7ffe027536b0 sp 0x7ffe027536a0
    #0 0x562865ae87c7 in su_trim strutil/strutil_bug.c:12
    #1 0x562865ae9837 in test_trim strutil/test_strutil.c:28
```

AddressSanitizer stops the program at the first out-of-bounds read: trimming `"   "` read `start[-1]`, one byte before the test's buffer. Without the sanitizer, this bug would usually pass the tests silently, reading a random stack byte.

### Review checklist

- Does every function handle `""`, a one-character string, and a buffer exactly one byte too small?
- Is every `isspace`/`toupper` argument cast to `unsigned char`?
- Does any pointer arithmetic step before the start of a buffer?
- Do the tests pass under ASan and UBSan, and does each planted bug make at least one test or sanitizer fail?

## Answers to the Chapter 2 prediction questions

**2.1 Q1.** `*p = *q` copies 2 into `a`; `p = q` re-points `p` to `b`; `*p = 7` sets `b`. Result: `a = 2`, `b = 7`.

**2.2 Q1.** `s + 2` is `0x1004` (two `short`s of 2 bytes), `&s[3]` is `0x1006`, `(char *)s + 2` is `0x1002`.

**2.3 Q1.** `sizeof p / sizeof p[0]` = 8 / 4 = 2 on 64-bit Linux, regardless of how many elements `p` points to: `sizeof` of a pointer is the pointer's size.

**2.4 Q1.** `sizeof s` is 9 (8 written characters plus the terminator added by the literal); `strlen(s)` is 2 — it stops at the embedded `'\0'`.

**2.6 Q1.** `words[1]` is `const char *`; `words[1][0]` is `const char` (the `'b'` of "banana"); `&words[1]` is `const char **` (pointer to the second element of the array).

# Chapter 3 — Memory and Data Layout

*Goal: manage memory deliberately — know where every object lives, how long it lives, who frees it, and exactly how its bytes are laid out.*

C has no garbage collector and no destructors. Every byte of heap memory you allocate, you must free exactly once, after its last use, and never touch again. Every struct you define has a byte layout the compiler chooses within rules you can predict. This chapter teaches both: object lifetimes and dynamic allocation with the errors that go with them, then structs, alignment, unions, enums, bit-fields, and the idioms (flexible array members, opaque types, arenas) that professional C uses to keep memory under control.

**What you will be able to do after this chapter**

- State the storage duration and lifetime of any object, and initialise objects of every kind correctly.
- Use `malloc`, `calloc`, `realloc`, and `free` correctly, including size-overflow checks and `realloc` failure handling.
- Find leaks, double frees, and use-after-free bugs with AddressSanitizer and Valgrind.
- Define and use structs, and predict their size and padding with `offsetof` and `_Alignof`.
- Use unions, tagged unions, enums, and bit-fields — and know the strict aliasing rule and how `memcpy` avoids it.
- Design APIs with clear **ownership** rules, opaque types, and flexible array members.

## Topic 3A — Lifetimes and allocation

## Unit 3.1 — Storage duration and initialisation

**One-line summary:** Every object has one of four storage durations — static, automatic, allocated, or thread — that fixes when it is created and destroyed. Static objects are zero-initialised if you give no initialiser; automatic and `malloc`'d objects are not. Designated initialisers and compound literals let you initialise structs and arrays clearly.

The table from Unit 0.6 is the foundation; this unit adds the initialisation rules.

```c
// file: init.c
#include <stdio.h>

struct point { int x, y; };

static int counter;                         /* static: zero */
static struct point origin;                 /* static: every member zero */

static int next_id(void) {
    static int id = 100;                    /* initialised once, before main */
    return id++;
}

int main(void) {
    int arr[5] = {1, 2};                    /* remaining elements are zero */
    int zeros[100] = {0};                   /* idiom: all zero */
    int sparse[8] = {[2] = 5, [6] = 9};     /* designated array initialisers (C99) */
    struct point p = {.y = 3};              /* designated member initialiser; x = 0 */
    struct point *q = &(struct point){7, 8};/* compound literal: an unnamed object */

    printf("counter=%d origin=(%d,%d)\n", counter, origin.x, origin.y);
    printf("arr: %d %d %d %d %d\n", arr[0], arr[1], arr[2], arr[3], arr[4]);
    printf("zeros[99]=%d sparse: ", zeros[99]);
    for (int i = 0; i < 8; i++)
        printf("%d ", sparse[i]);
    printf("\np=(%d,%d) *q=(%d,%d)\n", p.x, p.y, q->x, q->y);
    printf("ids: %d %d %d\n", next_id(), next_id(), next_id());
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra init.c -o init && ./init
counter=0 origin=(0,0)
arr: 1 2 0 0 0
zeros[99]=0 sparse: 0 0 5 0 0 0 9 0 
p=(0,3) *q=(7,8)
ids: 102 101 100
```

The rule behind `{1, 2}` and `{.y = 3}`: **if an initialiser is given for any part of an aggregate, every member without an explicit initialiser is initialised as if it had static storage — to zero**. So `= {0}` zeroes any array or struct. (C23 also allows `= {}`.)

Notice the `ids` line: it printed `102 101 100`. The three `next_id()` calls are arguments of one `printf`, evaluated in an unspecified order (Unit 1.6); GCC evaluated them right to left. Another reason to keep side effects out of argument lists.

A **compound literal** `(struct point){7, 8}` creates an unnamed object; inside a function it has automatic storage and lives until the end of the enclosing block, so `q` must not be used after the block ends.

### Lifetime ends are absolute

When an object's lifetime ends — the block of an automatic variable exits, `free` is called on allocated memory — every pointer to it becomes **indeterminate**. Not just dereferencing it: even *reading* such a pointer's value (comparing or printing it) is technically undefined. In practice, the rule to keep is simpler: after the lifetime ends, do not use the pointer at all, and set it to `NULL` if it stays in scope.

<!-- expanded -->
### Going deeper: initialisers in detail

Initialisation rules decide what a variable holds before your code touches it, and they differ by storage duration:

- **Static storage** (globals, `static` locals): initialised once, before `main` starts, to the value given — or to zero if none is given. The initialiser must be a constant expression.
- **Automatic storage** (ordinary locals): initialised each time execution reaches the declaration — but only if an initialiser is written. Without one, the value is indeterminate (DT-10).
- **Aggregates** (arrays and structs) follow an all-or-nothing rule: if *any* member is initialised, every member not mentioned is set to zero. So `int a[100] = {0};` zeroes the whole array, and `= {1}` sets the first element to 1 and the other 99 to 0.

**Designated initialisers** (C99) name the members or indexes they set, in any order:

```c
// file: inits.c
#include <stdio.h>

struct config { int port; int backlog; const char *host; double timeout; };

static int calls(void) {
    static int n;              /* static local: zero before main, keeps its value between calls */
    return ++n;
}

int main(void) {
    struct config c = { .host = "localhost", .port = 8080 };   /* backlog and timeout become 0 */
    printf("port %d backlog %d host %s timeout %.1f\n", c.port, c.backlog, c.host, c.timeout);

    int days[12] = { [1] = 28, [0] = 31, [11] = 31 };         /* the others are 0 */
    printf("days[0..2] = %d %d %d, days[11] = %d\n", days[0], days[1], days[2], days[11]);

    int squares[] = { [4] = 16, [9] = 81 };                    /* size taken from the largest index */
    printf("sizeof squares / sizeof squares[0] = %zu\n", sizeof squares / sizeof squares[0]);

    char name[8] = "ab";                                       /* "ab", then 6 zero bytes */
    printf("name[5] = %d\n", name[5]);

    calls(); calls();
    printf("calls() returned %d on the third call\n", calls());
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra inits.c -o inits && ./inits
port 8080 backlog 0 host localhost timeout 0.0
days[0..2] = 31 28 0, days[11] = 31
sizeof squares / sizeof squares[0] = 10
name[5] = 0
calls() returned 3 on the third call
```

### Deep & Tricky

**DT-39. `= {0}` zeroes every member, but in C17 the empty initialiser `= {}` is not valid C (it is a GCC extension and standard only from C23); and neither form promises that *padding bytes* are zero for an automatic struct.** *(Guarantee: C17 §6.7.9 ¶21 for the zero-fill of unmentioned members; C23 §6.7.10 makes `{}` standard and specifies zero padding for it; padding of automatic objects initialised with `{0}` is unspecified in C17)* When the bytes of a struct leave the program — written to a file, sent over a socket, copied to user space by a kernel — uninitialised padding leaks whatever was on the stack. Use `memset(&s, 0, sizeof s)` before filling such a struct (the Linux kernel's rule for anything copied to user space), and compile with `-pedantic` to catch `{}` in code that must be C17.

### Mastery check

1. **Predict.** What is the value of each element of `int a[4] = {[1] = 4, 7};`? (After a designator, initialisation continues with the next element.)
2. **Explain.** Why is `int *f(void) { return (int[]){1, 2, 3}; }` a bug?

## Unit 3.2 — Dynamic memory

**One-line summary:** `malloc(n)` returns a pointer to `n` uninitialised bytes (or `NULL`), `calloc(count, size)` returns zeroed memory with an overflow-checked size, `realloc(p, n)` resizes (possibly moving the block), and `free(p)` releases it. Always check for `NULL`, compute sizes from the pointer (`sizeof *p`), guard multiplications against overflow, and never lose the original pointer when `realloc` fails.

```c
// file: dyn.c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void) {
    size_t n = 5;
    int *a = malloc(n * sizeof *a);           /* sizeof *a: stays right if a's type changes */
    if (a == NULL) {
        perror("malloc");
        return 1;
    }
    for (size_t i = 0; i < n; i++)
        a[i] = (int)(i * i);

    /* grow: assign to a temporary so the old block is not leaked on failure */
    int *bigger = realloc(a, 2 * n * sizeof *a);
    if (bigger == NULL) {
        free(a);
        return 1;
    }
    a = bigger;
    memset(a + n, 0, n * sizeof *a);          /* new part is uninitialised: clear it */
    n *= 2;
    for (size_t i = 0; i < n; i++)
        printf("%d ", a[i]);
    printf("\n");
    free(a);
    a = NULL;                                 /* defensive: a can no longer dangle */

    int *z = calloc(4, sizeof *z);            /* zeroed */
    printf("calloc: %d %d %d %d\n", z[0], z[1], z[2], z[3]);
    free(z);

    /* size overflow: calloc checks it, a naive malloc(count * size) does not */
    size_t huge = SIZE_MAX / 4 + 2;
    void *bad = calloc(huge, 4);
    printf("calloc(SIZE_MAX/4 + 2, 4) = %p\n", bad);
    printf("naive count*size wraps to %zu bytes\n", huge * 4);

    free(NULL);                               /* always allowed: does nothing */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra dyn.c -o dyn && ./dyn
0 1 4 9 16 0 0 0 0 0 
calloc: 0 0 0 0
calloc(SIZE_MAX/4 + 2, 4) = (nil)
naive count*size wraps to 4 bytes
```

The naive multiplication wrapped to **4 bytes**: `malloc(huge * 4)` would have *succeeded* and returned a 4-byte block that the program then believes holds billions of elements — a classic heap-overflow vulnerability (Chapter 12). `calloc` checks the multiplication and fails. For `malloc` and `realloc`, check explicitly: `if (count > SIZE_MAX / size) fail;`, or use `__builtin_mul_overflow` / C23 `ckd_mul`. glibc 2.26+ provides `reallocarray(p, count, size)` for exactly this.

### Rules for the four functions

| Function | Returns | Contents | Notes |
| --- | --- | --- | --- |
| `malloc(n)` | `NULL` or a block suitably aligned for any type | indeterminate | `malloc(0)` may return `NULL` or a unique pointer: implementation-defined |
| `calloc(c, s)` | same | all bytes zero | fails if `c * s` overflows |
| `realloc(p, n)` | new pointer (maybe moved) or `NULL` | old contents preserved up to the smaller size | on failure the old block is **untouched and still yours**; `realloc(NULL, n)` = `malloc(n)` |
| `free(p)` | — | — | `p` must come from these functions and not already be freed; `free(NULL)` does nothing |

`aligned_alloc(alignment, size)` (C11) returns memory with a stricter alignment, for SIMD or page-aligned buffers.

Don't cast `malloc`'s result in C (`int *a = (int *)malloc(…)`): `void *` converts implicitly, and the cast can hide a missing `#include <stdlib.h>` in old compilers. (C++ requires the cast — one of the places the languages differ.)

### Where does the memory come from?

`malloc` is a library function, not a system call. glibc's allocator carves small blocks out of large regions it gets from the kernel (`brk` for the main heap, `mmap` for allocations above 128 KiB by default) and keeps freed blocks in bins for reuse. Each block has a small header (the chunk size) before the pointer you get — which is why writing even one byte before or past a block can corrupt the allocator and crash much later, in an unrelated `malloc` or `free`.

### Deep & Tricky

**DT-40. `p = realloc(p, n)` leaks on failure.** *(Library behaviour)* If `realloc` fails, it returns `NULL` and leaves the old block allocated. Assigning the result straight to `p` overwrites your only pointer to that block. Always use a temporary.

**DT-41. On Linux, `malloc` rarely returns `NULL` — overcommit.** *(OS behaviour)* Linux by default grants virtual memory optimistically; physical pages are allocated when first touched. A huge `malloc` can succeed, and the process is killed later by the **OOM killer** when it touches the pages. Still check for `NULL` (it does happen: address-space limits, `ulimit -v`, containers with `vm.overcommit_memory=2`), but don't assume a non-null result guarantees the memory is usable.

### Mastery check

1. **Explain.** Why is `sizeof *p` better than `sizeof(int)` in `p = malloc(n * sizeof *p)`?
2. **Write.** `char *dup_str(const char *s)` (like POSIX `strdup`), and `char *concat(const char *a, const char *b)` returning a new heap string; the caller frees both.

## Unit 3.3 — Memory errors and ownership

**One-line summary:** The five classic memory errors are **leaks** (never freed), **double free**, **use after free**, **out-of-bounds access**, and **use of uninitialised memory**. Prevent them with explicit **ownership** rules — for every pointer, exactly one owner is responsible for freeing — and find them with AddressSanitizer and Valgrind.

```c
// file: membugs.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void leak(void) {
    char *p = malloc(100);
    strcpy(p, "never freed");
}                                              /* p goes out of scope; block is lost */

static void use_after_free(void) {
    int *p = malloc(sizeof *p);
    *p = 1;
    free(p);
    printf("use after free reads %d\n", *p);   /* UB */
}

static void double_free(void) {
    char *p = malloc(8);
    free(p);
    free(p);                                   /* UB: corrupts the allocator */
}

int main(int argc, char *argv[]) {
    if (argc < 2)
        return 2;
    if (strcmp(argv[1], "leak") == 0)
        leak();
    else if (strcmp(argv[1], "uaf") == 0)
        use_after_free();
    else if (strcmp(argv[1], "double") == 0)
        double_free();
    return 0;
}
```

Without tools, the results are unpredictable:

```console
$ gcc -std=c17 -g membugs.c -o membugs
$ ./membugs leak; echo "exit $?"
exit 0
$ ./membugs uaf; echo "exit $?"
use after free reads 1662320940
exit 0
$ ./membugs double; echo "exit $?"
free(): double free detected in tcache 2
bash: line 1:  1644 Aborted                 ./membugs double
exit 134
```

The leak is silent, the use-after-free printed a plausible-looking garbage value, and only the double free was caught — by glibc's own consistency check (`free(): double free detected in tcache 2`), which detects some but not all cases. Now with the tools:

```console
$ gcc -std=c17 -g -fsanitize=address membugs.c -o membugs_asan
$ ./membugs_asan uaf 2>&1 | grep -E '^==.*ERROR|^READ|#0' | head -n 3  
==1651==ERROR: AddressSanitizer: heap-use-after-free on address 0x502000000010 at pc 0x562d6cae837f bp 0x7ffd2bf32f40 sp 0x7ffd2bf32f30
READ of size 4 at 0x502000000010 thread T0
    #0 0x562d6cae837e in use_after_free membugs.c:15
$ ./membugs_asan double 2>&1 | grep -E 'ERROR|#[01] ' | head -n 3
==1655==ERROR: AddressSanitizer: attempting double-free on 0x502000000010 in thread T0:
    #0 0x7fb621cfc4d8 in free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:52
    #1 0x55739a4a63cf in double_free membugs.c:21
$ ./membugs_asan leak 2>&1 | grep -E 'ERROR|SUMMARY|#[01] ' | head -n 4
==1659==ERROR: LeakSanitizer: detected memory leaks
    #0 0x7f89012fd9c7 in malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:69
    #1 0x55f0ec2ab2be in leak membugs.c:7
SUMMARY: AddressSanitizer: 100 byte(s) leaked in 1 allocation(s).
```

AddressSanitizer names each bug, the line, and — for use-after-free — where the memory was freed and where it was allocated (the full report shows all three stacks). Its **LeakSanitizer** runs at exit and lists every leaked block with the stack of its allocation.

Valgrind finds the same errors **without recompiling**, by running the program on a simulated CPU (about 20–50× slower). It is the tool for code you cannot rebuild, and it detects reads of uninitialised memory, which ASan does not (Clang's MemorySanitizer does):

```console
$ valgrind --leak-check=full -q ./membugs leak 2>&1 | grep -E 'definitely|at 0x|by 0x' | head -n 3
==1664== 100 bytes in 1 blocks are definitely lost in loss record 1 of 1
==1664==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==1664==    by 0x1091BE: leak (membugs.c:7)
```

### Ownership

C has no type-system support for ownership, so it lives in naming, documentation, and discipline:

- **The creator owns it** unless documented otherwise. A function that returns allocated memory says so ("the caller must free the result"), and the API provides a matching destroy function for complex objects: `obj_new` / `obj_free`, `fopen` / `fclose`.
- **Borrowing:** a function that receives a pointer and does not take ownership must not free it or keep it after returning (unless documented).
- **Transferring:** a function that takes ownership (e.g. adds a heap string to a list that will free it) says so explicitly.
- **One exit path frees:** structure functions so cleanup happens in one place (the `goto` ladder of Unit 1.7).
- **Null after free** where the pointer remains in scope, so double frees become harmless `free(NULL)` and use-after-free becomes a clean null-pointer crash.

### Pattern

**P-4. Pair every allocation with a documented owner and a single release point.** *(Practice)* Give every heap object one owner; write `_new`/`_free` pairs for compound objects; document ownership transfer in every function comment that returns or accepts heap memory; run the test suite under ASan (with LeakSanitizer) on every change and under Valgrind before releases. *Trade-off:* discipline and documentation overhead in exchange for memory safety without a garbage collector.

### Deep & Tricky

**DT-42. A use-after-free often "works" — until the block is reused.** *(Implementation detail)* After `free`, the bytes usually remain in place until the allocator hands the block out again, so reading them returns the old value (or, as above, allocator metadata). The bug appears later, when another allocation reuses the block and two parts of the program share memory. Attackers exploit exactly this to overwrite data structures (Chapter 12).

### Mastery check

1. **Explain.** What is the difference between a "definitely lost" and a "still reachable" block in Valgrind's report?
2. **Write.** A function `char **split_words(const char *s, size_t *count)` that returns a heap array of heap strings, and its matching `void free_words(char **w, size_t count)`. Prove there are no leaks with ASan.

## Topic 3B — Structs and layout

## Unit 3.4 — Structs

**One-line summary:** A struct groups named members of different types into one object. Structs can be initialised, assigned (copying every byte), passed to and returned from functions **by value**; for large structs or when the callee must modify the original, pass a pointer and use `->`.

```c
// file: structs.c
#include <stdio.h>
#include <string.h>

typedef struct {
    char name[32];
    int age;
    double balance;
} Account;

static Account make_account(const char *name, int age) {    /* returned by value */
    Account a = {.age = age, .balance = 0.0};
    snprintf(a.name, sizeof a.name, "%s", name);
    return a;
}

static void deposit(Account *a, double amount) {            /* modifies the original */
    a->balance += amount;                                   /* a->x means (*a).x */
}

static void print_account(const Account *a) {              /* const pointer: read-only, no copy */
    printf("%-6s age %2d balance %8.2f\n", a->name, a->age, a->balance);
}

int main(void) {
    Account alice = make_account("Alice", 30);
    Account copy = alice;                   /* struct assignment copies every member, even arrays */
    deposit(&alice, 100.50);
    print_account(&alice);
    print_account(&copy);                   /* unaffected */
    printf("sizeof(Account) = %zu\n", sizeof(Account));

    /* comparing structs: no == operator; compare members (memcmp is unsafe: padding, DT-43) */
    printf("same name? %s\n", strcmp(alice.name, copy.name) == 0 ? "yes" : "no");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra structs.c -o structs && ./structs
Alice  age 30 balance   100.50
Alice  age 30 balance     0.00
sizeof(Account) = 48
same name? yes
```

### Struct tags and `typedef`

`struct account { … };` declares a type named `struct account` (the **tag** is in a separate name space from ordinary identifiers). `typedef struct { … } Account;` gives an anonymous struct a type name. A common combined form is `typedef struct account { … } Account;`, which is needed for self-referential types:

```c
typedef struct node {
    int value;
    struct node *next;     /* the typedef name Node isn't declared yet here */
} Node;
```

Some style guides (including the Linux kernel's) avoid typedefs for structs so that `struct` in the code signals a compound type; others prefer the brevity. Pick one per project.

### Passing by value vs pointer

Passing a struct by value copies it — fine for small structs (a 2-D point, a pair of indices; the ABI passes up to 16 bytes in registers on x86-64), wasteful for large ones. Pass `const T *` for read-only access to larger structs, `T *` for modification.

<!-- expanded -->
### Going deeper: struct assignment is a shallow copy

Structs can be assigned, passed, and returned by value: `b = a;` copies every member, byte by byte. For members that are pointers, only the pointer is copied — both structs then point at the same heap block. If each struct's "owner" frees it, the block is freed twice.

```c
// file: shallow.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct person { char *name; int age; };

static struct person person_new(const char *name, int age) {
    struct person p = { malloc(strlen(name) + 1), age };
    strcpy(p.name, name);
    return p;                                /* returning a struct by value is fine */
}

static struct person person_clone(struct person p) {     /* a deep copy: new buffer for name */
    return person_new(p.name, p.age);
}

int main(void) {
    struct person a = person_new("Ada", 36);
    struct person b = a;                     /* shallow: b.name == a.name */
    struct person c = person_clone(a);       /* deep: c.name is its own copy */
    b.name[0] = 'E';                         /* changes a's name too */
    printf("a.name = %s, b.name = %s, c.name = %s\n", a.name, b.name, c.name);
    printf("a.name == b.name? %s; a.name == c.name? %s\n", a.name == b.name ? "yes" : "no",
           a.name == c.name ? "yes" : "no");
    fflush(stdout);                          /* the crash below would lose buffered output */
    free(a.name);
    free(c.name);
    free(b.name);                            /* bug: the same block again */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -g -fsanitize=address shallow.c -o shallow && ./shallow 2>&1 | grep -E "^a\.|SUMMARY|double-free|#[01] " | sed -E 's/0x[0-9a-f]+/ADDR/g' | cut -c1-110
a.name = Eda, b.name = Eda, c.name = Ada
a.name == b.name? yes; a.name == c.name? no
==1679==ERROR: AddressSanitizer: attempting double-free on ADDR in thread T0:
    #0 ADDR in free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:52
    #1 ADDR in main shallow.c:29
    #0 ADDR in free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:52
    #1 ADDR in main shallow.c:27
    #0 ADDR in malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:69
    #1 ADDR in person_new shallow.c:9
SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:52 in free
```

AddressSanitizer names the bug ("attempting double-free") and both places involved. A struct that owns memory needs a documented owner and explicit `clone` and `free` functions — the "Ownership" rules of Unit 3.3 apply to every pointer member.

**Compound literals** (C99) create an unnamed object of a struct or array type in an expression: `draw((struct point){ .x = 1, .y = 2 });`. Inside a function the object has automatic storage, like a local variable — returning a pointer to one is the same bug as returning the address of a local.

**Passing large structs by value** copies them on every call. A 4 KiB struct passed by value costs a 4 KiB copy; pass `const struct big *` instead. Small structs (a point, a pair of numbers, a string slice of pointer + length) are cheap to pass and return by value, and on x86-64 structs of up to 16 bytes travel in registers.

### Mastery check

1. **Predict.** If `Account` contained `char *name` instead of `char name[32]`, what would `Account copy = alice;` copy, and what problem appears?
2. **Write.** `typedef struct { double x, y; } Vec2;` with functions `vec2_add`, `vec2_scale`, `vec2_len` taking and returning values.

## Unit 3.5 — Alignment and padding

**One-line summary:** Each type has an **alignment** requirement (an `int` must start at an address divisible by 4, a `double` by 8 on x86-64). To satisfy it, the compiler inserts **padding** bytes between struct members and at the end, so a struct's size depends on member order. `offsetof` and `_Alignof` let you see the layout; ordering members from largest to smallest alignment minimises padding.

```c
// file: layout.c
#include <stdalign.h>
#include <stddef.h>
#include <stdint.h>
#include <stdio.h>

struct bad  { char a; double b; char c; int d; };
struct good { double b; int d; char a; char c; };

#define SHOW(T, m) printf("  %-2s offset %2zu size %zu\n", #m, offsetof(T, m), sizeof(((T *)0)->m))

int main(void) {
    printf("alignof: char %zu, int %zu, double %zu, long double %zu, max_align_t %zu\n",
           alignof(char), alignof(int), alignof(double), alignof(long double), alignof(max_align_t));

    printf("struct bad: size %zu\n", sizeof(struct bad));
    SHOW(struct bad, a); SHOW(struct bad, b); SHOW(struct bad, c); SHOW(struct bad, d);
    printf("struct good: size %zu\n", sizeof(struct good));
    SHOW(struct good, b); SHOW(struct good, d); SHOW(struct good, a); SHOW(struct good, c);

    struct bad arr[2];
    printf("array of 2 bad: %zu bytes; &arr[1] - &arr[0] = %td bytes\n",
           sizeof arr, (char *)&arr[1] - (char *)&arr[0]);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra layout.c -o layout && ./layout
alignof: char 1, int 4, double 8, long double 16, max_align_t 16
struct bad: size 24
  a  offset  0 size 1
  b  offset  8 size 8
  c  offset 16 size 1
  d  offset 20 size 4
struct good: size 16
  b  offset  0 size 8
  d  offset  8 size 4
  a  offset 12 size 1
  c  offset 13 size 1
array of 2 bad: 48 bytes; &arr[1] - &arr[0] = 24 bytes
```

`struct bad` wastes 10 of its 24 bytes on padding: 7 bytes after `a` so that `b` is 8-aligned, 3 after `c` so `d` is 4-aligned (16 + 4 = 20, then padding to 24 at the end, because the struct's size must be a multiple of its alignment so that every element of an **array** of structs stays aligned). Reordered, the same members take 16 bytes. In a million-element array that is 8 MB saved — and fewer cache lines touched (Chapter 11).

`alignof` and `alignas` come from `<stdalign.h>` (keywords in C23; `_Alignof` and `_Alignas` are the C11 spellings). `alignas(64) char buf[256];` aligns a buffer to a cache line.

### Packed structs and misaligned access

GCC's `__attribute__((packed))` removes padding, producing exactly the layout of a file format or network header. The cost: misaligned members. x86 tolerates misaligned loads (slightly slower); some ARM and RISC-V configurations trap; and taking the address of a packed member and dereferencing it through a normal pointer is undefined (GCC warns: `-Waddress-of-packed-member`). The portable way to read a binary format is to read bytes and assemble values (Chapter 6), not to overlay a packed struct.

### Deep & Tricky

**DT-43. Padding bytes have unspecified values, so `memcmp` on structs is unreliable.** *(Guarantee: C standard)* Two structs with equal members can differ in their padding bytes (left over from previous stack contents), so `memcmp(&a, &b, sizeof a)` may say "different". Writing structs to files with `fwrite` also leaks whatever was in the padding — potentially secrets. Compare member by member; serialise member by member; or zero the whole struct with `memset` (or `= {0}`) before filling it.

### Mastery check

1. **Predict.** `sizeof(struct { char a; short b; char c; })` and `sizeof(struct { short b; char a; char c; })` on x86-64?
2. **Write.** Use `offsetof` to print the layout of a struct with an `int`, a `char[3]`, a `long long`, and a `bool`; then reorder it to minimise its size.

## Unit 3.6 — Unions, type punning, and the strict aliasing rule

**One-line summary:** A **union** overlays its members in the same storage; its size is that of the largest member. Unions are used for **tagged unions** (a value that can be one of several types, with a tag saying which) and for reinterpreting bytes. Accessing an object through a pointer of an unrelated type violates the **strict aliasing rule** (undefined behaviour); `memcpy` is the portable way to reinterpret bytes.

### Tagged unions

```c
// file: tagged.c
#include <stdio.h>

typedef enum { VAL_INT, VAL_DOUBLE, VAL_STR } ValKind;

typedef struct {
    ValKind kind;               /* the tag: which member is valid */
    union {                     /* anonymous union (C11): members accessed as v.i, v.d, v.s */
        long long i;
        double d;
        const char *s;
    };
} Value;

static void print_value(Value v) {
    switch (v.kind) {
    case VAL_INT:    printf("int %lld\n", v.i); break;
    case VAL_DOUBLE: printf("double %g\n", v.d); break;
    case VAL_STR:    printf("string \"%s\"\n", v.s); break;
    }
}

int main(void) {
    Value vals[] = {
        {.kind = VAL_INT, .i = 42},
        {.kind = VAL_DOUBLE, .d = 2.5},
        {.kind = VAL_STR, .s = "hello"},
    };
    for (size_t i = 0; i < 3; i++)
        print_value(vals[i]);
    printf("sizeof(Value) = %zu (tag 4 + padding 4 + largest member 8)\n", sizeof(Value));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra tagged.c -o tagged && ./tagged
int 42
double 2.5
string "hello"
sizeof(Value) = 16 (tag 4 + padding 4 + largest member 8)
```

Tagged unions are how C represents "one of several kinds": JSON values, interpreter values, AST nodes, network messages. With `-Wswitch` (in `-Wall`), forgetting a case of the enum in a `switch` produces a warning — keep `default:` out of such switches so the warning can work.

### Reinterpreting bytes: three ways, one correct

```c
// file: punning.c
#include <inttypes.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>

static uint64_t bits_memcpy(double d) {
    uint64_t u;
    memcpy(&u, &d, sizeof u);       /* always correct; compiled to a single register move */
    return u;
}

static uint64_t bits_union(double d) {
    union { double d; uint64_t u; } x = {.d = d};
    return x.u;                     /* allowed in C (not in C++): reads the bytes as the other type */
}

static uint64_t bits_cast(double d) {
    return *(uint64_t *)&d;         /* violates strict aliasing: UB */
}

int main(void) {
    double d = -1.5;
    printf("memcpy: 0x%016" PRIX64 "\n", bits_memcpy(d));
    printf("union : 0x%016" PRIX64 "\n", bits_union(d));
    printf("cast  : 0x%016" PRIX64 "\n", bits_cast(d));
    uint64_t u = bits_memcpy(d);
    printf("sign %d, exponent %d, mantissa 0x%013" PRIX64 "\n",
           (int)(u >> 63), (int)((u >> 52) & 0x7FF) - 1023, u & ((UINT64_C(1) << 52) - 1));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 punning.c -o punning && ./punning
punning.c: In function ‘bits_cast’:
punning.c:19:13: warning: dereferencing type-punned pointer will break strict-aliasing rules [-Wstrict-aliasing]
   19 |     return *(uint64_t *)&d;         /* violates strict aliasing: UB */
      |             ^~~~~~~~~~~~~~
memcpy: 0xBFF8000000000000
union : 0xBFF8000000000000
cast  : 0xBFF8000000000000
sign 1, exponent 0, mantissa 0x8000000000000
```

All three printed the same bits here — the cast version happens to work in this tiny function, although GCC warned about it at `-O2`. That is what makes aliasing bugs dangerous: they appear when the optimiser reorders loads and stores in larger code (DT-44). The decoded fields show IEEE 754 at work: −1.5 is sign 1, exponent 0, mantissa 0.5 (`0x8000000000000` is the binary fraction .1).

### The strict aliasing rule

An object may be accessed only through an lvalue of (roughly) its own type, a signed/unsigned variant of it, a qualified version, an aggregate or union containing it, or a **character type**. Everything else is undefined. The rule lets the compiler assume that an `int *` and a `float *` never point to the same memory, so it can keep values in registers across writes through the other type.

Practical consequences:

- Use `memcpy` to reinterpret bytes (compilers optimise it to a plain move).
- `unsigned char *` may view any object (as `dump` functions do).
- Reading a struct header from a byte buffer by casting `(struct hdr *)buf` violates aliasing (and alignment). Copy it out with `memcpy`.
- GCC's `-fno-strict-aliasing` turns the assumption off; the Linux kernel builds with it. Portable code should not need it.

### Deep & Tricky

**DT-44. Strict-aliasing violations show up only under optimisation.** *(Guarantee: UB; optimiser behaviour)* A function like `int f(int *i, float *x) { *i = 1; *x = 0; return *i; }` may be compiled to `return 1;` at `-O2`, because writing a `float` cannot change an `int`. If a caller passes the same address for both, the "obvious" answer (0's bit pattern) is not returned. GCC warns about some cases with `-Wstrict-aliasing`; most are silent.

### Mastery check

1. **Explain.** Why is `unsigned char *` allowed to alias anything, and why is that essential for `memcpy` and `fwrite`?
2. **Write.** A tagged union `Shape` (circle, rectangle, triangle) with a function `double area(const Shape *)`, and check that removing a `case` triggers a `-Wswitch` warning.

## Unit 3.7 — Enums and bit-fields

**One-line summary:** An **enum** defines named integer constants (of type `int` before C23); it documents intent and helps `switch` warnings but provides no type safety. **Bit-fields** pack small integer members into fewer bits; their layout is implementation-defined, so use them for internal compactness, never for external formats.

```c
// file: enums.c
#include <stdio.h>

enum color { RED, GREEN = 5, BLUE };          /* 0, 5, 6 */

enum perm {                                   /* flag values: one bit each */
    PERM_READ  = 1u << 0,
    PERM_WRITE = 1u << 1,
    PERM_EXEC  = 1u << 2,
};

struct flags {
    unsigned int visible : 1;
    unsigned int dirty   : 1;
    unsigned int level   : 4;                  /* 0..15 */
    unsigned int         : 0;                  /* unnamed zero-width: start a new unit */
    unsigned int id      : 10;
};

int main(void) {
    printf("RED=%d GREEN=%d BLUE=%d sizeof(enum color)=%zu\n", RED, GREEN, BLUE, sizeof(enum color));

    enum color c = 42;                         /* no error: enums are integers */
    printf("c = %d (no range checking)\n", c);

    unsigned p = PERM_READ | PERM_EXEC;
    printf("perm 0x%X: read=%d write=%d exec=%d\n", p,
           !!(p & PERM_READ), !!(p & PERM_WRITE), !!(p & PERM_EXEC));

    struct flags f = {.visible = 1, .level = 9, .id = 1000};
    f.level = 17;                              /* stored modulo 16: 1 */
    printf("visible=%u dirty=%u level=%u id=%u sizeof=%zu\n",
           f.visible, f.dirty, f.level, f.id, sizeof f);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -Wconversion enums.c -o enums && ./enums
enums.c: In function ‘main’:
enums.c:31:15: warning: unsigned conversion from ‘int’ to ‘unsigned char:4’ changes value from ‘17’ to ‘1’ [-Woverflow]
   31 |     f.level = 17;                              /* stored modulo 16: 1 */
      |               ^~
RED=0 GREEN=5 BLUE=6 sizeof(enum color)=4
c = 42 (no range checking)
perm 0x5: read=1 write=0 exec=1
visible=1 dirty=0 level=1 id=1000 sizeof=8
```

GCC caught the bit-field truncation at compile time (`-Woverflow`, on by default) because the value was a constant; for a run-time value, `-Wconversion` warns instead. `!!x` converts any non-zero value to 1 — a common idiom for printing flags.

Bit-field facts: only `_Bool`, `signed int`, `unsigned int` (and implementation-defined other types; GCC allows any integer type) are portable; whether plain `int` bit-fields are signed is implementation-defined; you cannot take a bit-field's address; and the order of bits within a storage unit, and whether a field may straddle units, are implementation-defined. **C23** lets you choose an enum's underlying type: `enum small : unsigned char { A, B };`.

<!-- expanded -->
### Going deeper: what an `enum` really is

An enumeration constant has type `int`, and the enum type itself is "compatible with `char`, a signed integer type, or an unsigned integer type" — the compiler chooses. So any integer can be stored in an enum variable, and a `switch` over an enum may receive a value that matches no `case`. GCC's `-Wswitch` (in `-Wall`) warns when a `switch` on an enum misses a constant and has no `default`, which makes adding a new constant safe: every switch that must handle it is reported.

```c
// file: enumbits.c
#include <stdio.h>

enum color { RED, GREEN, BLUE, COLOR_COUNT };             /* COLOR_COUNT: a common idiom for the number of values */

static const char *const names[COLOR_COUNT] = { [RED] = "red", [GREEN] = "green", [BLUE] = "blue" };

static const char *name(enum color c) {
    if ((unsigned)c >= COLOR_COUNT) return "invalid";       /* an enum can hold any int */
    return names[c];
}

struct flags {
    unsigned int visible : 1;
    unsigned int level   : 3;                              /* 0..7 */
    int          signed3 : 3;                              /* -4..3 with an explicitly signed type */
};

int main(void) {
    enum color c = (enum color)7;
    printf("%s %s %s; sizeof(enum color) = %zu\n", name(GREEN), name(BLUE), name(c), sizeof(enum color));

    struct flags f = {0};
    f.level = 9;                                           /* does not fit in 3 bits: keeps the low 3 bits */
    f.signed3 = 3;
    f.signed3 += 1;                                        /* 4 does not fit in -4..3 */
    printf("level = %u (9 truncated to 3 bits), signed3 = %d, sizeof(struct flags) = %zu\n",
           (unsigned)f.level, (int)f.signed3, sizeof f);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -Wconversion enumbits.c -o enumbits 2>&1 | grep -E "warning" | sed 's/^enumbits.c:[0-9:]* //'; ./enumbits
warning: unsigned conversion from ‘int’ to ‘unsigned char:3’ changes value from ‘9’ to ‘1’ [-Woverflow]
green blue invalid; sizeof(enum color) = 4
level = 1 (9 truncated to 3 bits), signed3 = -4, sizeof(struct flags) = 4
```

GCC reported the constant 9 that cannot fit at compile time; the `+= 1` it cannot judge without knowing the run-time value. At run time, the unsigned field kept the low three bits of 9 (`1001` → `001`), and the signed field went from 3 to -4: storing an out-of-range value into a signed bit-field is implementation-defined (GCC wraps).

### Deep & Tricky

**DT-45. Whether a plain `int` bit-field is signed or unsigned is implementation-defined, and almost everything else about bit-field layout is too: the order of fields within a unit, whether a field may straddle a boundary, and the size of the storage unit.** *(Guarantee: C17 §6.7.2 ¶5 for the signedness of plain `int` bit-fields, §6.7.2.1 ¶11 for allocation order and straddling)* GCC and Clang make plain `int` fields signed, so `int flag : 1;` can hold only 0 and -1 — `if (f.flag == 1)` is never true. Always write `unsigned` (or `_Bool`) for flags, and never use bit-fields to match an external format (a network header, a hardware register) in portable code; use shifts and masks on a fixed-width integer instead, as Unit 6.4 does.

### Mastery check

1. **Predict.** Given `enum { A, B = 10, C, D = B };`, what are `A`, `C`, and `D`?
2. **Explain.** Why should a network packet header not be described with bit-fields?

## Topic 3C — Memory design idioms

## Unit 3.8 — Flexible array members and opaque types

**One-line summary:** A **flexible array member** (`char data[];` as the last member) lets one allocation hold a header plus a variable-length array. An **opaque type** — a struct declared in the header but defined only in the `.c` file — hides an object's layout from users, so they can only use it through your functions. Together they are the standard way to build C data types with real encapsulation.

### Flexible array members

```c
// file: flex.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    size_t len;
    char data[];                     /* flexible array member: must be last */
} Str;

static Str *str_new(const char *s) {
    size_t len = strlen(s);
    Str *p = malloc(sizeof *p + len + 1);    /* header + characters + terminator, one block */
    if (p == NULL)
        return NULL;
    p->len = len;
    memcpy(p->data, s, len + 1);
    return p;
}

int main(void) {
    Str *s = str_new("flexible");
    if (s == NULL)
        return 1;
    printf("len=%zu data=\"%s\" sizeof(Str)=%zu\n", s->len, s->data, sizeof(Str));
    free(s);                         /* one free releases header and data */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address flex.c -o flex && ./flex
len=8 data="flexible" sizeof(Str)=8
```

One allocation instead of two means one `free`, better locality, and no possibility of freeing the header but not the buffer. `sizeof(Str)` does not include the flexible member. (The older idiom `char data[1];` or `char data[0];` relies on undefined behaviour or a GCC extension; use `[]`.)

### Opaque types

```c
// file: counter/counter.h
#ifndef COUNTER_H
#define COUNTER_H

typedef struct counter Counter;      /* incomplete type: layout unknown to users */

Counter *counter_new(const char *name);   /* caller owns the result; free with counter_free */
void counter_free(Counter *c);
void counter_add(Counter *c, long n);
long counter_value(const Counter *c);
const char *counter_name(const Counter *c);

#endif
```

```c
// file: counter/counter.c
#include "counter.h"

#include <stdlib.h>
#include <string.h>

struct counter {                     /* the definition lives only here */
    long value;
    char name[];
};

Counter *counter_new(const char *name) {
    size_t n = strlen(name) + 1;
    Counter *c = malloc(sizeof *c + n);
    if (c == NULL)
        return NULL;
    c->value = 0;
    memcpy(c->name, name, n);
    return c;
}

void counter_free(Counter *c) { free(c); }
void counter_add(Counter *c, long n) { c->value += n; }
long counter_value(const Counter *c) { return c->value; }
const char *counter_name(const Counter *c) { return c->name; }
```

```c
// file: counter/main.c
#include <stdio.h>
#include "counter.h"

int main(void) {
    Counter *c = counter_new("requests");
    if (c == NULL)
        return 1;
    counter_add(c, 3);
    counter_add(c, 4);
    printf("%s = %ld\n", counter_name(c), counter_value(c));
    /* c->value = 0;   error: invalid use of incomplete typedef 'Counter' */
    counter_free(c);
    return 0;
}
```

```console
$ cd counter && gcc -std=c17 -Wall -Wextra counter.c main.c -o counter && ./counter
requests = 7
$ cd counter && printf '#include "counter.h"\nint f(Counter *c) { return c->value; }\n' > peek.c && gcc -c peek.c
peek.c: In function ‘f’:
peek.c:2:29: error: invalid use of incomplete typedef ‘Counter’ {aka ‘struct counter’}
    2 | int f(Counter *c) { return c->value; }
      |                             ^~
```

Users of `counter.h` can hold `Counter *` pointers but cannot see inside, allocate one on the stack, or depend on its layout — so you can change the representation without breaking or even recompiling their code (only relinking). `FILE` in `<stdio.h>` is used the same way. The cost: every object is heap-allocated and every access is a function call (which link-time optimisation can inline).

### Pattern

**P-5. Opaque handle + constructor/destructor pair for every non-trivial type.** *(Practice)* Declare `typedef struct thing Thing;` in the header; define the struct in one `.c` file; provide `thing_new`/`thing_free` and accessor functions; document ownership. *Trade-off:* heap allocation and call overhead in exchange for encapsulation, ABI stability, and invariants that only your code can break.

<!-- expanded -->
### Going deeper: what assignment does to a flexible array member

A struct with a flexible array member is "incomplete" in a useful way: `sizeof` counts only the fixed part, and the array's storage exists only if you allocated it. Anything that copies the struct by value — assignment, passing by value, returning — copies only the fixed part.

```c
// file: fam.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct msg {
    size_t len;
    char data[];                           /* flexible array member: must be last */
};

static struct msg *msg_new(const char *s) {
    size_t n = strlen(s);
    struct msg *m = malloc(sizeof *m + n + 1);   /* fixed part + the array's bytes */
    if (!m) return NULL;
    m->len = n;
    memcpy(m->data, s, n + 1);
    return m;
}

int main(void) {
    struct msg *m = msg_new("flexible");
    printf("sizeof(struct msg) = %zu, allocated %zu bytes, data = \"%s\"\n", sizeof(struct msg),
           sizeof *m + m->len + 1, m->data);

    struct msg copy = *m;                  /* copies len only: copy.data has no storage of its own */
    printf("copy.len = %zu; copy.data is at %p, %td bytes after &copy: just past the end of 'copy'\n",
           copy.len, (void *)copy.data, (char *)copy.data - (char *)&copy);
    free(m);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra fam.c -o fam && ./fam
sizeof(struct msg) = 8, allocated 17 bytes, data = "flexible"
copy.len = 8; copy.data is at 0x7fff7f5182e8, 8 bytes after &copy: just past the end of 'copy'
```

`copy.data` refers to memory just past the end of the 8-byte object `copy` — reading it would read whatever lies there on the stack. Structs with flexible array members are always handled through pointers, created and copied by functions that know the real size (`msg_new`, and a `msg_clone` that allocates `sizeof *m + m->len + 1` bytes and copies them all).

### Mastery check

1. **Explain.** Why can't users of `counter.h` write `Counter c;`?
2. **Write.** An opaque `Stack` of `int` with `stack_new(capacity)`, `stack_push`, `stack_pop` (returning an error code when empty), `stack_free`, using a flexible array member.

## Chapter 3 Project — A dynamic array and an arena allocator

**Goal:** implement two of the most-used building blocks in C programs: a growable array (`Vec`) and an **arena** (region) allocator that frees many objects at once, then use both to load and index a word list — proving correctness under ASan and Valgrind and measuring the allocation savings.

### Requirements

1. `vec.h`/`vec.c`: a vector of `int` with `vec_push` (amortised O(1) growth by doubling), `vec_get` (bounds-checked), `vec_len`, `vec_free`; overflow-checked growth; `realloc` failure leaves the vector unchanged.
2. `arena.h`/`arena.c`: `arena_alloc(a, size)` returns memory aligned for any type from large chunks; `arena_strdup`; `arena_free` releases everything at once; chunks grow as needed; oversized requests get their own chunk.
3. A demo that copies many strings into an arena and their lengths into a `Vec`, and reports the number of `malloc` calls versus allocating each string separately.
4. Tests run under ASan+UBSan with no errors and no leaks, and under Valgrind with "All heap blocks were freed".

### Reference solution

```c
// file: vecarena/vec.h
#ifndef VEC_H
#define VEC_H

#include <stddef.h>

typedef struct {
    int *data;
    size_t len, cap;
} Vec;                              /* small value type: zero-initialise with Vec v = {0}; */

/* vec_push appends x. Returns 0, or -1 if memory could not be allocated
   (the vector is then unchanged). */
int vec_push(Vec *v, int x);

/* vec_get stores element i in *out; returns -1 if i is out of range. */
int vec_get(const Vec *v, size_t i, int *out);

/* vec_free releases the storage and resets v to empty. */
void vec_free(Vec *v);

#endif
```

`Vec` is deliberately *not* opaque: it is a small value type that users embed in their own structs, like `Vec v = {0};`. Choosing between an open struct and an opaque handle is a design decision (P-5 lists the trade-off).

```c
// file: vecarena/vec.c
#include "vec.h"

#include <stdint.h>
#include <stdlib.h>

int vec_push(Vec *v, int x) {
    if (v->len == v->cap) {
        size_t new_cap = v->cap ? v->cap * 2 : 8;
        if (new_cap < v->cap || new_cap > SIZE_MAX / sizeof *v->data)
            return -1;                                  /* size overflow */
        int *p = realloc(v->data, new_cap * sizeof *v->data);
        if (p == NULL)
            return -1;                                  /* v unchanged (DT-40) */
        v->data = p;
        v->cap = new_cap;
    }
    v->data[v->len++] = x;
    return 0;
}

int vec_get(const Vec *v, size_t i, int *out) {
    if (i >= v->len)
        return -1;
    *out = v->data[i];
    return 0;
}

void vec_free(Vec *v) {
    free(v->data);
    *v = (Vec){0};                                      /* reset to a valid empty vector */
}
```

Doubling the capacity makes `vec_push` **amortised O(1)**: n pushes cause about log₂ n reallocations and copy fewer than 2n elements in total (Chapter 7 analyses this).

```c
// file: vecarena/arena.h
#ifndef ARENA_H
#define ARENA_H

#include <stddef.h>

typedef struct arena Arena;          /* opaque */

/* arena_new creates an arena that allocates in chunks of chunk_size bytes. */
Arena *arena_new(size_t chunk_size);

/* arena_alloc returns size bytes aligned for any type, or NULL. The memory
   lives until arena_free; it cannot be freed individually. */
void *arena_alloc(Arena *a, size_t size);

/* arena_strdup copies s into the arena. */
char *arena_strdup(Arena *a, const char *s);

/* arena_free releases every allocation made from a, and a itself. */
void arena_free(Arena *a);

/* arena_chunks reports how many chunks (malloc calls) the arena has made. */
size_t arena_chunks(const Arena *a);

#endif
```

```c
// file: vecarena/arena.c
#include "arena.h"

#include <stdalign.h>
#include <stddef.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>

typedef struct chunk {
    struct chunk *next;
    size_t cap, used;
    alignas(max_align_t) unsigned char mem[];   /* flexible array, maximally aligned */
} Chunk;

struct arena {
    Chunk *head;                     /* most recent chunk first */
    size_t chunk_size, nchunks;
};

Arena *arena_new(size_t chunk_size) {
    Arena *a = malloc(sizeof *a);
    if (a == NULL)
        return NULL;
    *a = (Arena){.chunk_size = chunk_size ? chunk_size : 64 * 1024};
    return a;
}

static Chunk *chunk_new(size_t cap) {
    if (cap > SIZE_MAX - sizeof(Chunk))
        return NULL;
    Chunk *c = malloc(sizeof *c + cap);
    if (c != NULL)
        *c = (Chunk){.cap = cap};
    return c;
}

void *arena_alloc(Arena *a, size_t size) {
    const size_t align = alignof(max_align_t);
    if (size > SIZE_MAX - align)
        return NULL;
    size = (size + align - 1) & ~(align - 1);   /* round up: keeps every block aligned */
    Chunk *c = a->head;
    if (c == NULL || c->cap - c->used < size) {
        size_t cap = size > a->chunk_size ? size : a->chunk_size;
        Chunk *n = chunk_new(cap);
        if (n == NULL)
            return NULL;
        if (size > a->chunk_size && c != NULL) {
            n->next = c->next;               /* oversized: keep the current chunk on top */
            c->next = n;
        } else {
            n->next = c;
            a->head = n;
        }
        a->nchunks++;
        c = n;
    }
    void *p = c->mem + c->used;
    c->used += size;
    return p;
}

char *arena_strdup(Arena *a, const char *s) {
    size_t n = strlen(s) + 1;
    char *p = arena_alloc(a, n);
    if (p != NULL)
        memcpy(p, s, n);
    return p;
}

size_t arena_chunks(const Arena *a) { return a->nchunks; }

void arena_free(Arena *a) {
    if (a == NULL)
        return;
    for (Chunk *c = a->head; c != NULL;) {
        Chunk *next = c->next;
        free(c);
        c = next;
    }
    free(a);
}
```

Design notes:

- `(size + align - 1) & ~(align - 1)` rounds up to a multiple of a power of two — a bit trick worth memorising (`align` must be a power of two).
- `alignas(max_align_t)` on the flexible member makes the chunk's memory start suitably aligned for any type, so every rounded block is too.
- An oversized request gets its own exactly-sized chunk, inserted *behind* the current chunk so the current chunk's remaining space is not wasted.
- Freeing the arena walks the chunk list once: thousands of objects, a handful of `free` calls, and no possibility of a leak or double free for individual objects.

```c
// file: vecarena/demo.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "arena.h"
#include "vec.h"

int main(void) {
    enum { N = 10000 };
    Arena *a = arena_new(16 * 1024);
    Vec lengths = {0};
    char **words = malloc(N * sizeof *words);
    if (a == NULL || words == NULL)
        return 1;

    char buf[32];
    for (int i = 0; i < N; i++) {
        int n = snprintf(buf, sizeof buf, "word-%d", i * 7919 % 100003);
        words[i] = arena_strdup(a, buf);
        if (words[i] == NULL || vec_push(&lengths, n) != 0)
            return 1;
    }

    char *big = arena_alloc(a, 100000);         /* oversized: gets its own chunk */
    if (big == NULL)
        return 1;
    memset(big, 'x', 100000);

    long total = 0;
    for (size_t i = 0; i < lengths.len; i++) {
        int len;
        vec_get(&lengths, i, &len);
        total += len;
    }
    int out;
    printf("words: %zu, first \"%s\", last \"%s\"\n", lengths.len, words[0], words[N - 1]);
    printf("total characters: %ld, vec capacity: %zu\n", total, lengths.cap);
    printf("vec_get(%d) -> %d (out of range)\n", N, vec_get(&lengths, N, &out));
    printf("arena chunks (malloc calls): %zu, versus %d with one malloc per word\n",
           arena_chunks(a), N);

    vec_free(&lengths);
    free(words);
    arena_free(a);                               /* 10,000 strings released at once */
    return 0;
}
```

```console
$ cd vecarena && gcc -std=c17 -Wall -Wextra -pedantic -g -fsanitize=address,undefined vec.c arena.c demo.c -o demo && ./demo
words: 10000, first "word-0", last "word-79708"
total characters: 98888, vec capacity: 16384
vec_get(10000) -> -1 (out of range)
arena chunks (malloc calls): 11, versus 10000 with one malloc per word
$ cd vecarena && gcc -std=c17 -Wall -Wextra -g vec.c arena.c demo.c -o demo_plain && valgrind -q --leak-check=full ./demo_plain && valgrind ./demo_plain 2>&1 | grep -E 'total heap usage|All heap'
words: 10000, first "word-0", last "word-79708"
total characters: 98888, vec capacity: 16384
vec_get(10000) -> -1 (out of range)
arena chunks (malloc calls): 11, versus 10000 with one malloc per word
==1760==   total heap usage: 26 allocs, 26 frees, 479,352 bytes allocated
==1760== All heap blocks were freed -- no leaks are possible
```

The arena made 11 chunk allocations for 10,000 strings plus one 100 KB block — instead of 10,001 `malloc` calls — and Valgrind confirms every block was freed. Valgrind's total of 26 allocations adds up exactly: 11 chunks, the arena header, the `words` array, 12 for the `Vec` (capacities 8, 16, …, 16,384 — `realloc` counts as an allocation each time), and stdio's output buffer.

### Review checklist

- Can any size computation overflow before it is checked?
- Does a failed `realloc` or `malloc` leave every data structure valid and leak nothing?
- Is every pointer returned by `arena_alloc` aligned for `max_align_t`? (Add an `assert((uintptr_t)p % alignof(max_align_t) == 0)` test.)
- Do ASan, UBSan, and Valgrind all report zero errors and zero leaks?

### Extensions

1. Add `arena_reset` (keep the chunks, mark them empty) for per-request allocation in a server loop.
2. Make `Vec` generic with a macro (`VEC_DEFINE(name, type)` generating a struct and functions) — Chapter 7 does this properly.

## Answers to the Chapter 3 prediction questions

**3.1 Q1.** `{0, 4, 7, 0}`: `[1] = 4`, then the next initialiser goes to element 2; element 0 and 3 are zero.

**3.4 Q1.** It would copy the pointer, not the characters: both structs would point to the same string. Modifying one changes the other, and freeing it through one leaves the other dangling — a *shallow copy*. Structs with owned pointers need explicit copy and free functions.

**3.5 Q1.** `{char; short; char}`: `a` at 0, padding 1, `b` at 2, `c` at 4, padding 1 → **6**. `{short; char; char}` → **4**.

**3.7 Q1.** `A = 0`, `C = 11`, `D = 10`.

# Chapter 4 — Program Structure, the Preprocessor, and Builds

*Goal: organise C programs into files, headers, and libraries that build reliably — and understand every step between `make` and a running program that uses a shared library.*

Single-file programs end quickly. Real C projects have dozens or thousands of files, headers shared between them, libraries built once and linked into many programs, and build systems that rebuild exactly what changed. This chapter covers how C connects files together (linkage), how to design headers, the preprocessor in full — its power and its traps — and the tools that build projects: Make, CMake, static libraries, and shared libraries.

**What you will be able to do after this chapter**

- Explain external, internal, and no linkage, and use `extern` and `static` deliberately.
- Write self-contained, guarded headers that declare exactly a module's public interface.
- Use the preprocessor safely: function-like macros without double-evaluation bugs, `#` and `##`, variadic macros, X-macros, and conditional compilation.
- Write a Makefile with pattern rules and automatic dependency tracking.
- Build, link, and inspect static and shared libraries, and control where the dynamic loader finds them.
- Build the same project with CMake.

## Topic 4A — Linking files together

## Unit 4.1 — Translation units and linkage

**One-line summary:** Each `.c` file is compiled separately (Unit 0.3). Names at file scope have **external linkage** by default — visible to the linker and shared across files — or **internal linkage** when declared `static`. A program must contain exactly one definition of every external object or function it uses; other files refer to it through `extern` declarations, normally in a header.

### Declarations and definitions

| Code at file scope | What it is |
| --- | --- |
| `int count = 0;` | definition (with external linkage) |
| `extern int count;` | declaration only: "defined somewhere else" |
| `int count;` | a **tentative definition**: becomes `int count = 0;` if no other definition appears in this file |
| `static int count;` | definition with internal linkage: private to this file |
| `int f(void);` | function declaration (`extern` is implied) |
| `int f(void) { … }` | function definition |
| `static int f(void) { … }` | function private to this file |

```c
// file: link/config.h
#ifndef CONFIG_H
#define CONFIG_H

extern int verbosity;           /* declared here, defined in config.c */
void config_load(int level);
int config_calls(void);

#endif
```

```c
// file: link/config.c
#include "config.h"

int verbosity = 1;              /* the one definition */
static int calls;               /* internal: no other file can see or clash with it */

static void log_call(void) {    /* internal helper */
    calls++;
}

void config_load(int level) {
    log_call();
    verbosity = level;
}

int config_calls(void) { return calls; }
```

```c
// file: link/main.c
#include <stdio.h>
#include "config.h"

static int calls = 100;         /* a different object from config.c's calls: no clash */

int main(void) {
    printf("verbosity starts at %d\n", verbosity);
    config_load(3);
    config_load(4);
    printf("verbosity now %d, config_calls %d, main's calls %d\n", verbosity, config_calls(), calls);
    return 0;
}
```

```console
$ cd link && gcc -std=c17 -Wall -Wextra config.c main.c -o app && ./app
verbosity starts at 1
verbosity now 4, config_calls 2, main's calls 100
$ cd link && gcc -c config.c && gcc -c main.c && nm config.o | grep -v ' U ' && echo --- && nm main.o | grep -v ' U '
0000000000000000 b calls
000000000000003a T config_calls
000000000000001a T config_load
0000000000000000 t log_call
0000000000000000 D verbosity
---
0000000000000000 d calls
0000000000000000 T main
```

`nm` shows linkage directly: upper-case letters (`T` text, `D` initialised data, `B` bss) are global symbols the linker can match; lower-case (`t`, `b`, `d`) are local to the object file. The two `calls` variables are both lower-case `b`/`d` symbols — separate objects that never meet.

### Multiple definitions

If two files both define `int count = 0;`, the link fails ("multiple definition"). Before GCC 10, two *tentative* definitions (`int count;` in two files) were silently merged as "common" symbols; since GCC 10 the default is `-fno-common` and they are an error too — which is what the standard says. The fix is always the same: one definition in one `.c` file, `extern` declarations everywhere else (via the header).

### Pattern

**P-6. `static` by default; external linkage only for the public interface.** *(Practice)* Every function and file-scope variable that is not part of the module's documented interface is `static`. The header declares the rest. *Trade-off:* none worth mentioning — fewer name clashes, better inlining, and a clear public surface. (The only cost is that unit tests cannot call static functions directly; test through the public interface or `#include` the `.c` file into the test.)

### Deep & Tricky

**DT-46. Global variables are shared mutable state across the whole program.** *(Design consequence)* An external `int verbosity` can be changed by any file, at any time, from any thread. Prefer `static` file-scope state behind functions (`config_load`, `config_level`), which gives one place to validate, log, or lock. When globals are unavoidable, declare them in exactly one header.

### Mastery check

1. **Explain.** What does `nm` print for a `static` function, and why can't another file call it?
2. **Predict.** Two files each contain `int total;` and nothing else about `total`. What happens when they are linked with GCC 13? With `-fcommon`?

## Unit 4.2 — Designing headers

**One-line summary:** A header is the **interface** of a module: the declarations of everything other files may use, plus the types and macros they need — and nothing else. Every header must be guarded against double inclusion, must compile on its own, and must be included by its own `.c` file first.

### What belongs in a header

| In the header | In the `.c` file |
| --- | --- |
| function prototypes of the public API, with comments | function definitions |
| type definitions users need (or opaque `typedef struct x X;`) | private types, the full definition of opaque structs |
| `extern` declarations of public globals (rare) | the definitions of those globals |
| macros and `enum` constants of the API | private macros and helpers (`static`) |
| `static inline` functions that must be inlined across files | everything else |

Never put **definitions** of non-`static` objects or functions in a header: every file that includes it would define them, and linking fails.

### Include guards

```c
#ifndef PROJECT_MODULE_H      /* a name unique in the whole program */
#define PROJECT_MODULE_H
/* … declarations … */
#endif
```

`#pragma once` does the same in one line; it is not in the C standard but every major compiler supports it. Either is fine; be consistent.

### Self-contained headers and include order

A header should `#include` everything it needs (if it uses `size_t`, it includes `<stddef.h>`), so it compiles regardless of what was included before it. The simplest test: in `module.c`, include `"module.h"` **first**. If the header is missing an include, `module.c` fails to compile immediately.

### `inline` in C is not `inline` in C++

C99's `inline` has unusual rules: a plain `inline` function definition in a header is *not* an external definition, and if the compiler decides not to inline a call, the linker looks for an external definition that may not exist. The portable, simple rule: **use `static inline` for small functions defined in headers.** Each file gets its own private copy (if one is needed at all).

```c
// file: inline/vec2.h
#ifndef VEC2_H
#define VEC2_H

typedef struct { double x, y; } Vec2;

static inline Vec2 vec2_add(Vec2 a, Vec2 b) { return (Vec2){a.x + b.x, a.y + b.y}; }
static inline double vec2_dot(Vec2 a, Vec2 b) { return a.x * b.x + a.y * b.y; }

#endif
```

```c
// file: inline/use.c
#include <stdio.h>
#include "vec2.h"

inline int square(int x) { return x * x; }    /* plain inline: no external definition */

int main(void) {
    Vec2 v = vec2_add((Vec2){1, 2}, (Vec2){3, 4});
    printf("v = (%g, %g), dot = %g, square = %d\n", v.x, v.y, vec2_dot(v, v), square(7));
    return 0;
}
```

```console
$ cd inline && gcc -std=c17 -O2 -Wall use.c -o use && ./use
v = (4, 6), dot = 52, square = 49
$ cd inline && gcc -std=c17 -O0 -Wall use.c -o use0
/usr/bin/ld: /tmp/ccqtu0UV.o: in function `main':
use.c:(.text+0x161): undefined reference to `square'
collect2: error: ld returned 1 exit status
```

At `-O2` the compiler inlined `square` and everything worked; at `-O0` it emitted a call — and there is no external definition, so linking failed. (The fix is `static inline`, or providing exactly one `extern int square(int);` declaration in one `.c` file.)

### Deep & Tricky

**DT-47. A plain `inline` function can fail to link at `-O0`.** *(Guarantee: C99 inline semantics)* In C, a definition marked `inline` without `static` or `extern` is an *inline definition* only; it does not provide the external symbol. Whether the program links depends on whether the compiler inlined every call — a property of the optimisation level. `static inline` avoids the problem completely.

### Mastery check

1. **Explain.** Why should `module.c` include `module.h` before any other header?
2. **Write.** Split Chapter 2's `strutil` into a proper module with a header containing only the public API, and verify the header compiles on its own with `gcc -fsyntax-only -x c strutil.h`.

## Topic 4B — The preprocessor

## Unit 4.3 — Macros and conditional compilation

**One-line summary:** The preprocessor rewrites text before compilation: `#define` creates object-like and function-like macros, `#if`/`#ifdef` include or exclude code, `#` turns an argument into a string, and `##` pastes tokens together. Macros have no types and no scope, and their arguments are substituted textually — so parenthesise everything and never pass arguments with side effects.

### The function-like macro traps

```c
// file: macros.c
#include <stdio.h>

#define SQUARE_BAD(x) x * x
#define SQUARE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))

/* a statement-like macro must be one statement: the do { } while (0) idiom */
#define SWAP(T, a, b) do { T tmp_ = (a); (a) = (b); (b) = tmp_; } while (0)

/* GNU statement expression: evaluates each argument once (GCC/Clang extension) */
#define MAX_ONCE(a, b) ({ __typeof__(a) a_ = (a); __typeof__(b) b_ = (b); a_ > b_ ? a_ : b_; })

static int calls;
static int next(void) { return ++calls; }

int main(void) {
    printf("SQUARE_BAD(1 + 2) = %d\n", SQUARE_BAD(1 + 2));   /* 1 + 2 * 1 + 2 */
    printf("SQUARE(1 + 2)     = %d\n", SQUARE(1 + 2));
    printf("10 / SQUARE(2)    = %d\n", 10 / SQUARE(2));

    calls = 0;
    int m = MAX(next(), 0);                  /* next() is called twice */
    printf("MAX(next(), 0) = %d after %d calls\n", m, calls);
    calls = 0;
    m = MAX_ONCE(next(), 0);
    printf("MAX_ONCE(next(), 0) = %d after %d call\n", m, calls);

    int a = 1, b = 2;
    if (a < b)
        SWAP(int, a, b);                     /* works as one statement thanks to do/while(0) */
    else
        printf("never\n");
    printf("a=%d b=%d\n", a, b);
    return 0;
}
```

```console
$ gcc -std=gnu17 -Wall -Wextra macros.c -o macros && ./macros
SQUARE_BAD(1 + 2) = 5
SQUARE(1 + 2)     = 9
10 / SQUARE(2)    = 2
MAX(next(), 0) = 2 after 2 calls
MAX_ONCE(next(), 0) = 1 after 1 call
a=2 b=1
```

The rules that follow:

1. **Parenthesise every parameter** in the replacement and **the whole replacement**.
2. **Never pass arguments with side effects** (`i++`, function calls) to a macro that may evaluate them more than once — or write the macro so each argument is evaluated once.
3. **Wrap multi-statement macros in `do { … } while (0)`**, so `if (c) MACRO(); else …` works.
4. **Prefer functions.** A `static inline` function has types, evaluates arguments once, can be debugged, and is just as fast. Use macros only for what functions cannot do: stringising, token pasting, `__FILE__`/`__LINE__` capture, type-generic code, and conditional compilation.

### Stringising, pasting, and variadic macros

```c
// file: macros2.c
#include <stdio.h>

#define STR(x) #x                      /* argument text as a string literal */
#define XSTR(x) STR(x)                 /* expand x first, then stringise */
#define VERSION 3
#define CONCAT(a, b) a##b              /* paste tokens into one */

#define LOG(fmt, ...) fprintf(stderr, "[%s:%d] " fmt "\n", __FILE__, __LINE__, __VA_ARGS__)
#define CHECK(expr) ((expr) ? (void)0 : (void)fprintf(stderr, "check failed: %s\n", #expr))

int main(void) {
    int CONCAT(my, var) = 42;          /* declares myvar */
    printf("%s\n", STR(VERSION));      /* "VERSION": # does not expand its argument */
    printf("%s\n", XSTR(VERSION));     /* "3" */
    printf("myvar = %d\n", myvar);
    LOG("value is %d", myvar);
    CHECK(myvar == 41);
    printf("compiled with %s on %s, C version %ld\n", __VERSION__, __DATE__, __STDC_VERSION__);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra macros2.c -o macros2 && ./macros2 2>&1 | sed 's/on [A-Z][a-z]* [ 0-9]* [0-9]*/on <date>/'
[macros2.c:17] value is 42
check failed: myvar == 41
VERSION
3
myvar = 42
compiled with 13.3.0 on <date>, C version 201710
```

`__VA_ARGS__` stands for the arguments matched by `...`. Calling `LOG("hello")` with no extra arguments leaves a trailing comma; C23 and GCC accept `__VA_OPT__(,)` to handle that portably. `__FILE__`, `__LINE__`, `__func__` (a predefined identifier, not a macro), `__DATE__`, and `__STDC_VERSION__` are always available — the reason assertion and logging macros must be macros.

### Conditional compilation

```c
#if defined(__linux__)
#  include <sys/epoll.h>
#elif defined(__APPLE__)
#  include <sys/event.h>
#else
#  error "unsupported platform"
#endif

#ifndef NDEBUG
#  define DEBUG_PRINT(...) fprintf(stderr, __VA_ARGS__)
#else
#  define DEBUG_PRINT(...) ((void)0)
#endif
```

`-DNAME` or `-DNAME=value` on the command line defines a macro; `gcc -dM -E - < /dev/null` lists every predefined macro. `assert` from `<assert.h>` is disabled when `NDEBUG` is defined — so never put necessary work inside an `assert`.

### X-macros: one list, many uses

```c
// file: xmacro.c
#include <stdio.h>

#define COLORS(X) \
    X(RED,   0xFF0000) \
    X(GREEN, 0x00FF00) \
    X(BLUE,  0x0000FF)

#define AS_ENUM(name, rgb) COLOR_##name,
#define AS_NAME(name, rgb) #name,
#define AS_RGB(name, rgb) rgb,

enum color { COLORS(AS_ENUM) COLOR_COUNT };
static const char *const color_names[] = { COLORS(AS_NAME) };
static const unsigned color_rgb[] = { COLORS(AS_RGB) };

int main(void) {
    for (int c = 0; c < COLOR_COUNT; c++)
        printf("%d %-5s #%06X\n", c, color_names[c], color_rgb[c]);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra xmacro.c -o xmacro && ./xmacro
0 RED   #FF0000
1 GREEN #00FF00
2 BLUE  #0000FF
```

The list is written once; the enum, the names table, and the values table can never get out of sync. X-macros are common in compilers, protocol definitions, and error-code tables.

### Deep & Tricky

**DT-48. `#` does not expand its argument; you need two levels to stringise a macro's value.** *(Guarantee: preprocessor rules)* Arguments next to `#` or `##` are not macro-expanded first. `STR(VERSION)` gives `"VERSION"`; `XSTR(VERSION)` expands `VERSION` to `3` while passing it to `STR`, giving `"3"`.

**DT-49. Code inside `assert` disappears in release builds.** *(Library behaviour)* `assert(fclose(f) == 0);` closes the file in debug builds and does nothing at all with `-DNDEBUG`. Asserts check invariants; they must not have side effects.

### Mastery check

1. **Predict.** With `#define DOUBLE(x) x + x`, what is `3 * DOUBLE(2)`?
2. **Write.** An X-macro list of HTTP status codes (200 OK, 404 Not Found, …) that generates an enum and a `const char *status_text(int code)` function.

## Topic 4C — Building

## Unit 4.4 — Make

**One-line summary:** Make rebuilds files whose **prerequisites** changed, following **rules** of the form `target: prerequisites` + recipe. With pattern rules, automatic variables, and compiler-generated dependency files (`-MMD -MP`), a short Makefile builds any C project incrementally and correctly.

```make
# file: makedemo/Makefile
CC      := gcc
CFLAGS  := -std=c17 -Wall -Wextra -pedantic -g -O2 -MMD -MP
LDFLAGS :=
LDLIBS  := -lm

SRCS := main.c geometry.c
OBJS := $(SRCS:.c=.o)
DEPS := $(OBJS:.o=.d)

.PHONY: all clean

all: shapes

shapes: $(OBJS)
	$(CC) $(LDFLAGS) $^ $(LDLIBS) -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f shapes $(OBJS) $(DEPS)

-include $(DEPS)
```

```c
// file: makedemo/geometry.h
#ifndef GEOMETRY_H
#define GEOMETRY_H

double circle_area(double r);
double hypotenuse(double a, double b);

#endif
```

```c
// file: makedemo/geometry.c
#include "geometry.h"
#include <math.h>

/* M_PI is POSIX, not ISO C: with -std=c17, glibc does not define it */
static const double PI = 3.14159265358979323846;

double circle_area(double r) { return PI * r * r; }
double hypotenuse(double a, double b) { return sqrt(a * a + b * b); }
```

```c
// file: makedemo/main.c
#include <stdio.h>
#include "geometry.h"

int main(void) {
    printf("area(2) = %.4f, hypotenuse(3, 4) = %.1f\n", circle_area(2), hypotenuse(3, 4));
    return 0;
}
```

```console
$ cd makedemo && make clean >/dev/null; make
gcc -std=c17 -Wall -Wextra -pedantic -g -O2 -MMD -MP -c main.c -o main.o
gcc -std=c17 -Wall -Wextra -pedantic -g -O2 -MMD -MP -c geometry.c -o geometry.o
gcc  main.o geometry.o -lm -o shapes
$ cd makedemo && make
make: Nothing to be done for 'all'.
$ cd makedemo && touch geometry.h && make
gcc -std=c17 -Wall -Wextra -pedantic -g -O2 -MMD -MP -c main.c -o main.o
gcc -std=c17 -Wall -Wextra -pedantic -g -O2 -MMD -MP -c geometry.c -o geometry.o
gcc  main.o geometry.o -lm -o shapes
$ cd makedemo && cat main.d
main.o: main.c geometry.h
geometry.h:
```

How to read it:

- `$@` is the target, `$<` the first prerequisite, `$^` all prerequisites. `%.o: %.c` is a **pattern rule**: how to make any `.o` from the matching `.c`.
- Recipes must be indented with a **tab character**, not spaces.
- `.PHONY` marks targets that are not files, so a file named `clean` cannot confuse Make.
- `-MMD -MP` makes GCC write a `.d` file per object listing the headers it included. `-include $(DEPS)` loads them, so touching `geometry.h` rebuilt both objects that include it — and nothing else. Without this, Make does not know about headers, and changing a struct in a header leaves stale objects linked together: a source of baffling bugs.
- The second `make` did nothing: everything was up to date.

A note on `geometry.c`: the first version of this example used `M_PI`, and the build failed with "`M_PI` undeclared". `M_PI` is a POSIX extension, not ISO C; with `-std=c17` (strict mode), glibc's `<math.h>` hides every non-standard name. Either request the extensions explicitly — `#define _DEFAULT_SOURCE` (or `_POSIX_C_SOURCE`, `_GNU_SOURCE`) before the first `#include` — or, as here, define the constant yourself. This is exactly the kind of portability problem strict mode exists to reveal.

`make -j$(nproc)` builds in parallel; correct dependencies make that safe.

### Deep & Tricky

**DT-50. Without header dependencies, Make silently links stale objects.** *(Tool behaviour)* If `main.o` depends only on `main.c` in the Makefile, changing a struct definition in a header rebuilds nothing, and old and new code disagree about the struct's layout — memory corruption with no compiler error. Generate dependencies with `-MMD -MP` (or use CMake, which does it automatically).

### Mastery check

1. **Explain.** What does `-MP` add to the `.d` file, and what error does it prevent when you delete a header?
2. **Write.** Add `test` and `asan` targets to the Makefile: `make asan` builds with sanitizers into a separate directory.

## Unit 4.5 — Static and shared libraries

**One-line summary:** A **static library** (`libx.a`) is an archive of object files copied into each executable at link time. A **shared library** (`libx.so`) is linked at run time by the dynamic loader: one copy on disk and in memory, upgradable without relinking, but the program needs to find it. Build shared libraries with `-fPIC -shared`, and give them an **soname** for versioning.

```c
// file: libdemo/greet.h
#ifndef GREET_H
#define GREET_H

/* greet writes a greeting for name into buf (size n); returns snprintf's result. */
int greet(char *buf, unsigned long n, const char *name);
const char *greet_version(void);

#endif
```

```c
// file: libdemo/greet.c
#include "greet.h"
#include <stdio.h>

static const char *salutation(void) { return "Hello"; }   /* not exported */

int greet(char *buf, unsigned long n, const char *name) {
    return snprintf(buf, n, "%s, %s!", salutation(), name);
}

const char *greet_version(void) { return "1.0"; }
```

```c
// file: libdemo/app.c
#include <stdio.h>
#include "greet.h"

int main(void) {
    char buf[64];
    greet(buf, sizeof buf, "library");
    printf("%s (libgreet %s)\n", buf, greet_version());
    return 0;
}
```

### Static library

```console
$ cd libdemo && rm -f libgreet.so* && gcc -std=c17 -Wall -O2 -c greet.c -o greet.o && ar rcs libgreet.a greet.o && ar t libgreet.a
greet.o
$ cd libdemo && gcc -std=c17 -Wall app.c -L. -lgreet -o app_static && ./app_static && (ldd app_static | grep greet || echo "no libgreet dependency")
Hello, library! (libgreet 1.0)
no libgreet dependency
```

`-L.` adds the current directory to the library search path and `-lgreet` means "search for `libgreet.so` or `libgreet.a`". `ldd` lists the shared libraries an executable needs; it shows no `libgreet`, because the library's code was copied into the executable. If both `libgreet.so` and `libgreet.a` exist in a search directory, the linker prefers the shared one; to force the static one, name the file (`./libgreet.a`) or use `-Wl,-Bstatic -lgreet -Wl,-Bdynamic`. (The first time this example was run, a `libgreet.so` left over from the next section was picked up and the "static" program failed to start — a mistake worth recognising.) Order matters with static libraries: the linker processes files left to right and only pulls in archive members that satisfy *currently* undefined symbols, so libraries must come **after** the objects that use them (`gcc -lgreet app.c` fails).

### Shared library

```console
$ cd libdemo && gcc -std=c17 -Wall -O2 -fPIC -shared -Wl,-soname,libgreet.so.1 greet.c -o libgreet.so.1.0 && ln -sf libgreet.so.1.0 libgreet.so.1 && ln -sf libgreet.so.1 libgreet.so
$ cd libdemo && gcc -std=c17 -Wall app.c -L. -lgreet -o app_shared && ./app_shared
./app_shared: error while loading shared libraries: libgreet.so.1: cannot open shared object file: No such file or directory
$ cd libdemo && LD_LIBRARY_PATH=. ./app_shared
Hello, library! (libgreet 1.0)
$ cd libdemo && gcc -std=c17 -Wall app.c -L. -lgreet -Wl,-rpath,'$ORIGIN' -o app_rpath && ./app_rpath && ldd app_rpath | grep greet
Hello, library! (libgreet 1.0)
	libgreet.so.1 => libdemo/./libgreet.so.1 (0x00007fbb93650000)
$ cd libdemo && readelf -d libgreet.so.1.0 | grep SONAME && nm -D --defined-only libgreet.so.1.0 | grep greet
 0x000000000000000e (SONAME)             Library soname: [libgreet.so.1]
0000000000001120 T greet
0000000000001150 T greet_version
```

What happened:

- `-fPIC` generates **position-independent code**, which works at whatever address the loader maps the library. `-shared` produces a `.so`.
- The first run failed: the linker found `libgreet.so` in `.`, but at run time the **dynamic loader** (`ld-linux-x86-64.so.2`) searches only the system paths (`/lib`, `/usr/lib`, those in `/etc/ld.so.conf`), not the current directory.
- Fixes: `LD_LIBRARY_PATH` (for testing), an **rpath** baked into the executable (`$ORIGIN` means "the directory containing the executable"), or installing the library into a system directory and running `ldconfig`.
- The **soname** `libgreet.so.1` is what the executable records as its dependency. Installing `libgreet.so.1.1` (a compatible bug fix) and pointing the `libgreet.so.1` symlink at it upgrades every program without relinking; an incompatible version gets soname `libgreet.so.2` and can be installed alongside.
- `nm -D` lists the dynamic (exported) symbols: `greet` and `greet_version`, but not the `static` function `salutation`.

### Controlling exports

By default every non-`static` function in a shared library is exported, including internal helpers shared between the library's own files. Build with `-fvisibility=hidden` and mark the public API with `__attribute__((visibility("default")))` (usually via a macro like `GREET_API`). Fewer exported symbols means faster loading, no accidental API, and the freedom to change internals.

### Loading at run time: `dlopen`

Plugins are shared libraries loaded by name while the program runs: `void *h = dlopen("./plugin.so", RTLD_NOW); int (*f)(void) = (int (*)(void))dlsym(h, "plugin_init");`. Chapter 8 builds a plugin loader.

### Deep & Tricky

**DT-51. Library order on the command line matters for static libraries.** *(Linker behaviour)* GNU `ld` scans archives once, in order, extracting only members that resolve symbols undefined *so far*. `gcc -lfoo main.o` leaves `main.o`'s references to `foo` unresolved. Put libraries after the objects that use them, and dependent libraries before the libraries they depend on (`-lhigh -llow`). `-Wl,--start-group … --end-group` handles circular dependencies.

**DT-52. Changing a struct in a shared library's public header breaks every program compiled against the old one.** *(ABI)* Programs compiled with the old header allocate and access the struct with the old layout. The library's **ABI** (binary interface) changed even if its source API looks the same. Opaque types (P-5) avoid this: only the library knows the layout.

### Mastery check

1. **Explain.** Why must code in a shared library be position-independent, and what does `$ORIGIN` solve?
2. **Predict.** You rebuild `libgreet.so.1.0` with a changed `greet` function and copy it over the old file. Do `app_static` and `app_rpath` see the change?

## Unit 4.6 — CMake

**One-line summary:** CMake is a build-system *generator*: a `CMakeLists.txt` describes targets (executables and libraries) and their properties, and CMake generates Makefiles or Ninja files for the actual build. It handles header dependencies, shared/static libraries, compiler differences, installation, and integration with tests and IDEs — the de facto standard for C and C++ projects.

```cmake
# file: libdemo/CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(greet VERSION 1.0 LANGUAGES C)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

add_library(greet SHARED greet.c)
target_include_directories(greet PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(greet PROPERTIES VERSION 1.0 SOVERSION 1)
target_compile_options(greet PRIVATE -Wall -Wextra -pedantic)

add_executable(app app.c)
target_link_libraries(app PRIVATE greet)
```

```console
$ cd libdemo && rm -rf build && cmake -S . -B build -DCMAKE_BUILD_TYPE=Release > /dev/null && cmake --build build 2>&1 | grep -E 'Built target|Linking'
[ 50%] Linking C shared library libgreet.so
[ 50%] Built target greet
[100%] Linking C executable app
[100%] Built target app
$ cd libdemo && ./build/app && ls build | grep libgreet
Hello, library! (libgreet 1.0)
libgreet.so
libgreet.so.1
libgreet.so.1.0
```

The concepts that matter:

- **Targets** (`add_library`, `add_executable`) and **properties** on them, set with `target_*` commands. Modern CMake avoids global variables like `CMAKE_C_FLAGS` for anything target-specific.
- **`PUBLIC` / `PRIVATE` / `INTERFACE`** say whether a property applies to the target itself, to its users, or both: `target_include_directories(greet PUBLIC …)` means "greet and anything linking greet get this include path". That propagation is CMake's main value in large projects.
- **Out-of-source builds** (`-B build`) keep generated files out of the source tree.
- CMake set the soname symlinks and an rpath for the build tree automatically.
- `CMAKE_BUILD_TYPE` chooses `Debug` (`-g`), `Release` (`-O3 -DNDEBUG`), `RelWithDebInfo`, or `MinSizeRel`.

<!-- expanded -->
### Going deeper: tests, sanitizer builds, and the cache

A CMake project grows three more habits quickly: a test suite run by **CTest**, a separate build directory per configuration (one with sanitizers, one optimised), and an understanding of the **cache** — the file `build/CMakeCache.txt` in which CMake stores every option and detected setting from the first configure.

```cmake
# file: calc/CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(calc VERSION 0.1 LANGUAGES C)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

option(CALC_SANITIZE "Build with AddressSanitizer and UBSan" OFF)

add_library(calc STATIC calc.c)
target_include_directories(calc PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_options(calc PRIVATE -Wall -Wextra -pedantic)

if(CALC_SANITIZE)
  # applied to every target created after this point, for compiling and linking
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
  add_link_options(-fsanitize=address,undefined)
  target_compile_options(calc PRIVATE -fsanitize=address,undefined -fno-omit-frame-pointer)
endif()

add_executable(calc_test test_calc.c)
target_link_libraries(calc_test PRIVATE calc)

enable_testing()
add_test(NAME arithmetic COMMAND calc_test arithmetic)
add_test(NAME overflow   COMMAND calc_test overflow)
```

```c
// file: calc/include/calc.h
#ifndef CALC_H
#define CALC_H
#include <stdbool.h>
/* Add two ints; returns false (and leaves *out unchanged) if the result would overflow. */
bool calc_add(int a, int b, int *out);
#endif
```

```c
// file: calc/calc.c
#include "calc.h"

bool calc_add(int a, int b, int *out) {
    int r;
    if (__builtin_add_overflow(a, b, &r)) return false;     /* GCC/Clang builtin: no UB */
    *out = r;
    return true;
}
```

```c
// file: calc/test_calc.c
#include <limits.h>
#include <stdio.h>
#include <string.h>
#include "calc.h"

static int failures;
#define CHECK(c) do { if (!(c)) { fprintf(stderr, "FAIL %s:%d: %s\n", __FILE__, __LINE__, #c); failures++; } } while (0)

int main(int argc, char **argv) {
    int r = 0;
    if (argc > 1 && strcmp(argv[1], "arithmetic") == 0) {
        CHECK(calc_add(2, 3, &r) && r == 5);
        CHECK(calc_add(-7, 7, &r) && r == 0);
    } else if (argc > 1 && strcmp(argv[1], "overflow") == 0) {
        r = 42;
        CHECK(!calc_add(INT_MAX, 1, &r) && r == 42);
        CHECK(!calc_add(INT_MIN, -1, &r));
        CHECK(calc_add(INT_MAX, 0, &r) && r == INT_MAX);
    } else {
        fprintf(stderr, "usage: calc_test arithmetic|overflow\n");
        return 2;
    }
    return failures ? 1 : 0;
}
```

```console
$ cd calc && rm -rf build build-asan && cmake -S . -B build -DCMAKE_BUILD_TYPE=Release > /dev/null && cmake --build build > /dev/null && ctest --test-dir build 2>&1 | grep -E "Test +#|tests passed"
1/2 Test #1: arithmetic .......................   Passed    0.00 sec
2/2 Test #2: overflow .........................   Passed    0.00 sec
100% tests passed, 0 tests failed out of 2
$ cd calc && cmake -S . -B build-asan -DCMAKE_BUILD_TYPE=Debug -DCALC_SANITIZE=ON > /dev/null && cmake --build build-asan > /dev/null && ctest --test-dir build-asan 2>&1 | grep -E "tests passed" && grep -c fsanitize build-asan/CMakeFiles/calc_test.dir/flags.make
100% tests passed, 0 tests failed out of 2
1
```

Two build directories from one source tree: an optimised one and a sanitizer one, each with its own cache. CTest ran each test as a separate process and reported pass/fail by exit status — the same contract as the `make test` targets of Unit 4.4 and the CI pipelines of Chapter 5.

### Deep & Tricky

**DT-53. A cache variable keeps the value from the first configure: changing the `option()` default or a `set(... CACHE ...)` default in `CMakeLists.txt` has no effect on an existing build directory, and a `-D` given once stays until it is changed or the cache is deleted.** *(Tool behaviour: CMake documentation, `option()` — "If the variable is already set as a normal or cache variable, then the command does nothing"; `set(CACHE)` does not overwrite an existing entry unless `FORCE` is given)* This is why "I changed the default and nothing happened" is a classic CMake question, and why CI should configure into a fresh directory. Inspect the cache with `cmake -LA -N build`, change one value with `cmake -D NAME=value build`, or start over with `rm -rf build`.

```console
$ cd calc && sed -i 's/"Build with AddressSanitizer and UBSan" OFF/"Build with AddressSanitizer and UBSan" ON/' CMakeLists.txt && cmake -S . -B build > /dev/null && cmake -LA -N build | grep CALC_SANITIZE; rm -rf build-fresh && cmake -S . -B build-fresh > /dev/null && cmake -LA -N build-fresh | grep CALC_SANITIZE; sed -i 's/"Build with AddressSanitizer and UBSan" ON/"Build with AddressSanitizer and UBSan" OFF/' CMakeLists.txt
CALC_SANITIZE:BOOL=OFF
CALC_SANITIZE:BOOL=ON
```

The existing `build` directory still says `OFF` after the default was changed to `ON`; only the fresh directory picked up the new default.

### Mastery check

1. **Explain.** What is the difference between `target_link_libraries(app PRIVATE greet)` and `PUBLIC greet` when `app` is itself a library?
2. **Write.** Add an option `GREET_STATIC` that builds `greet` as a static library instead, and an `enable_testing()` + `add_test` for a small test program.

## Chapter 4 Project — A reusable library and CLI, built three ways

**Goal:** write a small but real library — an INI configuration file parser — with a clean public header, an opaque type, and full error reporting; a command-line tool that uses it; and build both with a plain compiler command, a Makefile with dependency tracking, and CMake, as both static and shared libraries.

### Specification

INI format:

```
; comment
[server]
host = example.com
port = 8080

[paths]
root = /var/www
```

Library API (`ini.h`):

```c
typedef struct ini Ini;
Ini *ini_parse_file(const char *path, char *err, size_t errsz);   /* NULL on error, message in err */
const char *ini_get(const Ini *ini, const char *section, const char *key);  /* NULL if absent */
size_t ini_count(const Ini *ini);
void ini_free(Ini *ini);
```

Rules: whitespace around keys and values is trimmed; keys outside any section belong to section `""`; lines longer than 1023 characters, missing `]`, and lines without `=` are errors reported with the line number; a later duplicate key overrides an earlier one.

CLI: `iniget FILE SECTION KEY` prints the value and exits 0, exits 1 if the key is absent, 2 on parse or usage errors.

### Reference solution

```c
// file: inilib/ini.h
#ifndef INI_H
#define INI_H

#include <stddef.h>

typedef struct ini Ini;

/* ini_parse_file reads and parses an INI file. On failure it returns NULL
   and writes a message (with the line number, if relevant) into err.
   The caller owns the result and must release it with ini_free. */
Ini *ini_parse_file(const char *path, char *err, size_t errsz);

/* ini_get returns the value of key in section ("" for keys before any
   section), or NULL. The string is owned by ini and valid until ini_free. */
const char *ini_get(const Ini *ini, const char *section, const char *key);

/* ini_count returns the number of distinct (section, key) entries. */
size_t ini_count(const Ini *ini);

/* ini_free releases ini and every string returned by ini_get. NULL is allowed. */
void ini_free(Ini *ini);

#endif
```

```c
// file: inilib/ini.c
#include "ini.h"

#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *section, *key, *value;     /* each separately allocated, owned by the Ini */
} Entry;

struct ini {
    Entry *entries;
    size_t len, cap;
};

static char *dup_str(const char *s) {
    size_t n = strlen(s) + 1;
    char *p = malloc(n);
    if (p != NULL)
        memcpy(p, s, n);
    return p;
}

/* trim returns a pointer to the first non-space character of s and
   terminates s after the last non-space character. */
static char *trim(char *s) {
    while (isspace((unsigned char)*s))
        s++;
    size_t n = strlen(s);
    while (n > 0 && isspace((unsigned char)s[n - 1]))
        s[--n] = '\0';
    return s;
}

static Entry *find(const Ini *ini, const char *section, const char *key) {
    for (size_t i = 0; i < ini->len; i++)
        if (strcmp(ini->entries[i].section, section) == 0 && strcmp(ini->entries[i].key, key) == 0)
            return &ini->entries[i];
    return NULL;
}

/* set adds or replaces an entry. Returns 0, or -1 when out of memory. */
static int set(Ini *ini, const char *section, const char *key, const char *value) {
    char *v = dup_str(value);
    if (v == NULL)
        return -1;
    Entry *e = find(ini, section, key);
    if (e != NULL) {                       /* duplicate key: later value wins */
        free(e->value);
        e->value = v;
        return 0;
    }
    if (ini->len == ini->cap) {
        size_t cap = ini->cap ? ini->cap * 2 : 16;
        Entry *p = realloc(ini->entries, cap * sizeof *p);
        if (p == NULL) {
            free(v);
            return -1;
        }
        ini->entries = p;
        ini->cap = cap;
    }
    Entry n = {dup_str(section), dup_str(key), v};
    if (n.section == NULL || n.key == NULL) {
        free(n.section);
        free(n.key);
        free(v);
        return -1;
    }
    ini->entries[ini->len++] = n;
    return 0;
}

Ini *ini_parse_file(const char *path, char *err, size_t errsz) {
    FILE *f = fopen(path, "r");
    if (f == NULL) {
        snprintf(err, errsz, "%s: %s", path, strerror(errno));
        return NULL;
    }
    Ini *ini = calloc(1, sizeof *ini);
    char line[1024], section[256] = "";
    int lineno = 0;
    if (ini == NULL) {
        snprintf(err, errsz, "out of memory");
        goto fail;
    }
    while (fgets(line, sizeof line, f) != NULL) {
        lineno++;
        size_t n = strlen(line);
        if (n == sizeof line - 1 && line[n - 1] != '\n' && !feof(f)) {
            snprintf(err, errsz, "%s:%d: line too long", path, lineno);
            goto fail;
        }
        char *s = trim(line);
        if (*s == '\0' || *s == ';' || *s == '#')
            continue;
        if (*s == '[') {
            char *close = strchr(s, ']');
            if (close == NULL) {
                snprintf(err, errsz, "%s:%d: missing ']'", path, lineno);
                goto fail;
            }
            *close = '\0';
            snprintf(section, sizeof section, "%s", trim(s + 1));
            continue;
        }
        char *eq = strchr(s, '=');
        if (eq == NULL) {
            snprintf(err, errsz, "%s:%d: expected key = value", path, lineno);
            goto fail;
        }
        *eq = '\0';
        char *key = trim(s), *value = trim(eq + 1);
        if (*key == '\0') {
            snprintf(err, errsz, "%s:%d: empty key", path, lineno);
            goto fail;
        }
        if (set(ini, section, key, value) != 0) {
            snprintf(err, errsz, "out of memory");
            goto fail;
        }
    }
    if (ferror(f)) {
        snprintf(err, errsz, "%s: read error", path);
        goto fail;
    }
    fclose(f);
    return ini;
fail:
    fclose(f);
    ini_free(ini);
    return NULL;
}

const char *ini_get(const Ini *ini, const char *section, const char *key) {
    const Entry *e = find(ini, section, key);
    return e ? e->value : NULL;
}

size_t ini_count(const Ini *ini) { return ini->len; }

void ini_free(Ini *ini) {
    if (ini == NULL)
        return;
    for (size_t i = 0; i < ini->len; i++) {
        free(ini->entries[i].section);
        free(ini->entries[i].key);
        free(ini->entries[i].value);
    }
    free(ini->entries);
    free(ini);
}
```

Design notes: lookups are linear (configuration files are small; Chapter 7's hash map would replace `find` without changing the header — the benefit of the opaque type). Every failure path goes through one `fail:` label that frees everything. Error messages follow the `file:line: message` convention that editors and CI tools recognise.

```c
// file: inilib/iniget.c
#include <stdio.h>
#include "ini.h"

int main(int argc, char *argv[]) {
    if (argc != 4) {
        fprintf(stderr, "usage: iniget FILE SECTION KEY\n");
        return 2;
    }
    char err[256];
    Ini *ini = ini_parse_file(argv[1], err, sizeof err);
    if (ini == NULL) {
        fprintf(stderr, "iniget: %s\n", err);
        return 2;
    }
    const char *v = ini_get(ini, argv[2], argv[3]);
    int status = 1;
    if (v != NULL) {
        printf("%s\n", v);
        status = 0;
    }
    ini_free(ini);
    return status;
}
```

```ini
# file: inilib/test.ini
; sample configuration
name = demo

[server]
host = example.com
  port   =   8080
host = override.example.com

[ paths ]
root = /var/www
empty =
```

```ini
# file: inilib/bad.ini
[server]
host = example.com
this line has no equals sign
```

The `# file:` line is just a comment to the INI parser (it skips lines starting with `#` or `;`).

**Build 1 — the compiler directly:**

```console
$ cd inilib && gcc -std=c17 -Wall -Wextra -pedantic -g -fsanitize=address,undefined ini.c iniget.c -o iniget
$ cd inilib && ./iniget test.ini server host && ./iniget test.ini server port && ./iniget test.ini paths root && ./iniget test.ini "" name
override.example.com
8080
/var/www
demo
$ cd inilib && ./iniget test.ini paths empty | od -c | head -n 1; ./iniget test.ini server missing; echo "exit $?"
0000000  \n
exit 1
$ cd inilib && ./iniget bad.ini server host; echo "exit $?"; ./iniget nothere.ini a b; echo "exit $?"
iniget: bad.ini:4: expected key = value
exit 2
iniget: nothere.ini: No such file or directory
exit 2
```

**Build 2 — Make, with a static library:**

```make
# file: inilib/Makefile
CC      := gcc
CFLAGS  := -std=c17 -Wall -Wextra -pedantic -O2 -g -MMD -MP
ARFLAGS := rcs

.PHONY: all clean

all: iniget

libini.a: ini.o
	$(AR) $(ARFLAGS) $@ $^

iniget: iniget.o libini.a
	$(CC) $(LDFLAGS) iniget.o -L. -lini -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f iniget *.o *.d libini.a

-include $(wildcard *.d)
```

```console
$ cd inilib && make clean > /dev/null && make && ./iniget test.ini server port
gcc -std=c17 -Wall -Wextra -pedantic -O2 -g -MMD -MP -c iniget.c -o iniget.o
gcc -std=c17 -Wall -Wextra -pedantic -O2 -g -MMD -MP -c ini.c -o ini.o
ar rcs libini.a ini.o
gcc  iniget.o -L. -lini -o iniget
8080
```

**Build 3 — CMake, with a shared library:**

```cmake
# file: inilib/CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(ini VERSION 1.0 LANGUAGES C)
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_EXTENSIONS OFF)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)

add_library(ini ini.c)
target_include_directories(ini PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(ini PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION 1)
target_compile_options(ini PRIVATE -Wall -Wextra -pedantic)

add_executable(iniget iniget.c)
target_link_libraries(iniget PRIVATE ini)

enable_testing()
add_test(NAME get_port COMMAND iniget ${CMAKE_CURRENT_SOURCE_DIR}/test.ini server port)
set_tests_properties(get_port PROPERTIES PASS_REGULAR_EXPRESSION "^8080")
add_test(NAME bad_file COMMAND iniget ${CMAKE_CURRENT_SOURCE_DIR}/bad.ini server host)
set_tests_properties(bad_file PROPERTIES WILL_FAIL TRUE)
```

```console
$ cd inilib && rm -rf build && cmake -S . -B build > /dev/null && cmake --build build > /dev/null && ldd build/iniget | grep libini
	libini.so.1 => inilib/build/libini.so.1 (0x00007f2a5ae36000)
$ cd inilib && ctest --test-dir build 2>&1 | grep -E 'tests passed|Test #'
1/2 Test #1: get_port .........................   Passed    0.00 sec
2/2 Test #2: bad_file .........................   Passed    0.00 sec
100% tests passed, 0 tests failed out of 2
```

`BUILD_SHARED_LIBS` is a standard CMake option: `cmake -DBUILD_SHARED_LIBS=OFF` builds the same project with a static library, no other change needed.

### Review checklist

- Does `ini.h` compile on its own, and does it expose nothing but the API?
- Does every error path free everything allocated so far (check with ASan's leak checker on `bad.ini`)?
- Do all three builds produce the same behaviour, and does touching `ini.h` rebuild `iniget.o` in the Make build?
- Is every non-API function `static`? (`nm -D` on the shared library should show only the four API functions plus libc imports.)

## Answers to the Chapter 4 prediction questions

**4.1 Q2.** With GCC 13 (default `-fno-common`): a "multiple definition of `total'" link error. With `-fcommon`, the two tentative definitions are merged into one common symbol and the program links.

**4.3 Q1.** `3 * 2 + 2` = `8`, not 12: the replacement is not parenthesised.

**4.5 Q2.** `app_static` does not — it contains its own copy of the old code. `app_rpath` does, the next time it starts, because it loads `libgreet.so.1` (a symlink to the replaced file) at run time.

# Chapter 5 — Undefined Behaviour, Debugging, and Testing

*Goal: write C that is correct, not just C that seems to work — by understanding undefined behaviour and by using every tool that finds bugs before users do.*

C gives the compiler the right to assume your program never does certain things. When a program does them anyway, the result is not an error message but **undefined behaviour**: code that may work today and fail after a compiler upgrade, behave differently at different optimisation levels, or become a security hole. The defence is a combination of knowledge (what is undefined), tooling (warnings, static analysis, sanitizers, Valgrind, a debugger), and discipline (assertions, error-handling conventions, and tests). This chapter covers all three, and its project is a bug hunt: ten planted bugs, each found by a specific tool.

**What you will be able to do after this chapter**

- Recognise the common kinds of undefined behaviour and explain how optimisers exploit them.
- Use compiler warnings and static analysers (`-fanalyzer`, `clang-tidy`, `cppcheck`) effectively.
- Choose and read the reports of AddressSanitizer, UndefinedBehaviorSanitizer, LeakSanitizer, MemorySanitizer, and Valgrind.
- Debug with GDB: breakpoints, stepping, backtraces, inspecting memory, watchpoints, and core dumps.
- Design error handling in C consistently, and use assertions correctly.
- Write unit tests in plain C and measure test coverage with `gcov`.

## Topic 5A — Undefined behaviour

## Unit 5.1 — A catalogue of undefined behaviour

**One-line summary:** The C standard lists about two hundred undefined behaviours; a few dozen account for almost all real bugs. Learn them as a checklist: memory (out of bounds, use after free, null), arithmetic (signed overflow, division by zero, bad shifts), types (strict aliasing, misalignment), sequencing (unsequenced modifications), and lifetime (uninitialised values, dangling pointers).

| Category | Undefined behaviour | Caught by |
| --- | --- | --- |
| Memory | array index or pointer arithmetic out of bounds | ASan, Valgrind (heap) |
| | dereferencing a null or invalid pointer | crash, ASan |
| | use after `free`, double `free`, freeing a non-heap pointer | ASan, Valgrind, glibc checks |
| | modifying a string literal or a `const` object | crash (read-only page) |
| | `memcpy` with overlapping regions | ASan (`memcpy-param-overlap`), Valgrind |
| Arithmetic | signed integer overflow | UBSan |
| | integer division or `%` by zero; `INT_MIN / -1` | UBSan, SIGFPE |
| | shift by negative amount or ≥ width; left shift of negative values | UBSan |
| | converting an out-of-range floating value to an integer | UBSan (`-fsanitize=float-cast-overflow`) |
| Types | accessing an object through an incompatible type (strict aliasing) | rarely detected; `-Wstrict-aliasing` |
| | misaligned pointer dereference | UBSan (`alignment`) |
| | calling a function through a pointer of the wrong type | UBSan (`function`, Clang) |
| Sequencing | modifying an object twice without sequencing (`i = i++`) | `-Wsequence-point`, `-Wunsequenced` |
| Lifetime | reading an uninitialised automatic variable (in many cases) | `-Wuninitialized`, MSan, Valgrind |
| | using a pointer to an object whose lifetime ended | ASan (`stack-use-after-return` option) |
| Concurrency | data race | TSan (Chapter 9) |
| Library | passing a negative `char` to `<ctype.h>` functions, wrong `printf` format | `-Wformat`; partly |
| Control | reaching the end of a non-`void` function whose value is used | `-Wreturn-type` |
| | infinite loop without side effects (C11: may be assumed to terminate) | — |

### How compilers exploit UB

Optimisers assume UB never happens and simplify code accordingly. A real demonstration:

```c
// file: ubopt.c
#include <limits.h>
#include <stdbool.h>
#include <stdio.h>

int table[4] = {1, 2, 3, 4};

__attribute__((noinline)) bool exists(int v) {
    for (int i = 0; i <= 4; i++)        /* bug: reads table[4] */
        if (table[i] == v)
            return true;
    return false;
}

__attribute__((noinline)) int bigger(int x) {
    return x + 1 > x;                   /* "always true", unless x + 1 overflows */
}

__attribute__((noinline)) int doublings(void) {
    int n = 0;
    for (int i = 1; i > 0; i *= 2) {    /* stops when i overflows to negative ... or does it? */
        n++;
        if (n > 100)
            break;                      /* safety net for this demonstration */
    }
    return n;
}

int main(void) {
    printf("exists(42)=%d bigger(INT_MAX)=%d doublings=%d\n", exists(42), bigger(INT_MAX), doublings());
    return 0;
}
```

```console
$ gcc -O0 ubopt.c -o ub_gcc0 && ./ub_gcc0
exists(42)=0 bigger(INT_MAX)=1 doublings=31
$ gcc -O2 ubopt.c -o ub_gcc2 && ./ub_gcc2
exists(42)=1 bigger(INT_MAX)=1 doublings=101
$ clang -O0 ubopt.c -o ub_clang0 && ./ub_clang0
exists(42)=0 bigger(INT_MAX)=0 doublings=31
$ clang -O2 ubopt.c -o ub_clang2 && ./ub_clang2
exists(42)=0 bigger(INT_MAX)=1 doublings=101
```

Four builds of the same program, four different outputs:

- **`exists(42)` returns true at GCC `-O2`**, although 42 is not in the table. GCC reasoned: the loop reads `table[i]` for `i` up to 4; reading `table[4]` is UB; so the loop must exit through `return true` before `i` reaches 4 — and it compiled the whole function to `return true`. (This exact pattern was found in SPEC benchmarks when GCC introduced the optimisation.)
- **`bigger(INT_MAX)`**: Clang at `-O0` computed the wrapped value and said 0; GCC folded `x + 1 > x` to 1 even at `-O0`; Clang at `-O2` also folded it to 1. Each answer is "correct" — there is no correct answer.
- **`doublings()`**: unoptimised, `i` wraps to negative after 31 doublings. Optimised, both compilers decided a positive `i` doubled stays positive, removed the test `i > 0`, and only the safety net stopped the loop at 101.

None of these builds produced a warning with default flags. Run `ubopt.c` under `-fsanitize=undefined` and each overflow is reported; `exists` needs `-fsanitize=address` (global-buffer-overflow).

### Deep & Tricky

**DT-54. Undefined behaviour can affect code that runs *before* it.** *(Guarantee: the standard places no requirements on the whole execution)* Because the optimiser reasons about all paths, UB later in a function can change how earlier code is compiled — as `exists` shows: the out-of-bounds read at `i == 4` changed the result for every input. "But the bad line never runs for this input" is not a defence if the compiler can prove it *would* run.

**DT-55. A null check after a dereference can be deleted.** *(Optimiser behaviour, a famous Linux kernel bug)* In `int v = p->x; if (p == NULL) return -1;`, the dereference implies `p` is not null, so the check is dead code and is removed. The Linux kernel's TUN driver had this bug in 2009 (CVE-2009-1897); the kernel now builds with `-fno-delete-null-pointer-checks`. Check before you dereference.

### Mastery check

1. **Explain.** Why does GCC compile `exists` to `return true` at `-O2`? Why is that a legal translation?
2. **Predict.** Is `unsigned u = UINT_MAX; u + 1 > u` subject to the same optimisation as `bigger`?

## Unit 5.2 — Warnings and static analysis

**One-line summary:** Compiler warnings are free static analysis: enable `-Wall -Wextra` plus a few extras, treat them as errors in CI, and add deeper analysers — GCC's `-fanalyzer`, `clang-tidy`, and `cppcheck` — which follow paths through your code to find leaks, null dereferences, and use-after-free without running it.

### The warning set worth enabling

| Flag | Catches |
| --- | --- |
| `-Wall -Wextra` | the core set: uninitialised use, format mismatches, sign comparison, unused values, missing returns |
| `-Wpedantic` | non-standard extensions |
| `-Wshadow` | variables shadowing others |
| `-Wconversion -Wsign-conversion` | implicit conversions that may change values (noisy, valuable in new code) |
| `-Wformat=2` | non-literal format strings (format-string attacks) |
| `-Wnull-dereference` | paths that dereference null (needs `-O2`) |
| `-Wdouble-promotion` | `float` silently promoted to `double` (embedded code) |
| `-Wvla` | variable-length arrays |
| `-Werror` | turn warnings into errors (CI) |

### Static analysers

```c
// file: analyze.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char *load(const char *path) {
    FILE *f = fopen(path, "r");
    char *buf = malloc(64);
    if (fgets(buf, 64, f) == NULL)       /* f and buf may be NULL */
        return NULL;                     /* leaks buf and f */
    fclose(f);
    return buf;
}

int main(void) {
    char *s = load("/etc/hostname");
    printf("%zu\n", s ? strlen(s) : 0);
    free(s);
    free(s);                             /* double free */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -c analyze.c && echo "gcc -Wall: no warnings"
analyze.c: In function ‘main’:
analyze.c:19:5: warning: pointer ‘s’ used after ‘free’ [-Wuse-after-free]
   19 |     free(s);                             /* double free */
      |     ^~~~~~~
analyze.c:18:5: note: call to ‘free’ here
   18 |     free(s);
      |     ^~~~~~~
gcc -Wall: no warnings
$ gcc -std=c17 -fanalyzer -c analyze.c 2>&1 | grep -E 'warning:' | sed 's/^analyze.c://'
9:9: warning: use of possibly-NULL ‘f’ where non-null expected [CWE-690] [-Wanalyzer-possible-null-argument]
19:5: warning: double-‘free’ of ‘s’ [CWE-415] [-Wanalyzer-double-free]
$ cppcheck --quiet analyze.c 2>&1 | grep -E 'error|warning' | sed 's/^analyze.c://'
10:9: error: Resource leak: f [resourceLeak]
10:9: error: Memory leak: buf [memleak]
19:5: error: Memory pointed to by 's' is freed twice. [doubleFree]
$ clang-tidy -quiet analyze.c -- -std=c17 2>&1 | grep -E 'warning:' | sed 's/^.*analyze.c://'
10:16: warning: Potential leak of memory pointed to by 'buf' [clang-analyzer-unix.Malloc]
19:5: warning: Attempt to free released memory [clang-analyzer-unix.Malloc]
```

Ordinary warnings found only the double free (GCC 12 added `-Wuse-after-free` to `-Wall`). The analysers found the rest by exploring paths, not by running code: GCC's `-fanalyzer` saw that `f` may be `NULL` when passed to `fgets`; `cppcheck` found both leaks on the early return; `clang-tidy` found the leak of `buf`; and all three found the double free. Notice that no single tool found everything — and none reported that `buf` itself may be `NULL`. Each tool finds things the others miss; analysers also produce false positives, so treat their reports as leads to investigate. `-fanalyzer` is GCC's (since GCC 10, good for C); `clang-tidy` runs the Clang Static Analyzer plus hundreds of style and bug-pattern checks (`clang-analyzer-*`, `bugprone-*`, `cert-*`); `cppcheck` is independent of both compilers.

### Mastery check

1. **Write.** Run all three analysers on your Chapter 4 INI library and investigate every report.

## Topic 5B — Run-time tools

## Unit 5.3 — Sanitizers in depth

**One-line summary:** Sanitizers are compiler instrumentation that checks for bugs as the program runs: **ASan** (memory errors), **UBSan** (arithmetic and type UB), **LSan** (leaks, part of ASan), **MSan** (uninitialised reads, Clang only), and **TSan** (data races). They find bugs only on paths your tests execute, so they are only as good as your test inputs.

| Sanitizer | Flag | Finds | Slowdown | Combine with |
| --- | --- | --- | --- | --- |
| AddressSanitizer | `-fsanitize=address` | out-of-bounds (heap, stack, globals), use-after-free, double free, use-after-return (option), leaks | ~2× | UBSan |
| UndefinedBehaviorSanitizer | `-fsanitize=undefined` | signed overflow, shifts, division by zero, null, misalignment, bad casts, `bool`/enum values | small | ASan or alone |
| LeakSanitizer | included in ASan, or `-fsanitize=leak` | leaks at exit | none until exit | — |
| MemorySanitizer | `-fsanitize=memory` (Clang) | reads of uninitialised memory | ~3× | nothing else; whole program incl. libraries must be instrumented |
| ThreadSanitizer | `-fsanitize=thread` | data races, some deadlocks | 5–15× | nothing else |

```c
// file: sanitize.c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    int which = argc > 1 ? atoi(argv[1]) : 0;
    int x = 0x7FFFFFFF, shift = 33 - which;
    int a[4] = {0};
    int idx = 3 + which;
    switch (which) {
    case 1: printf("%d\n", x + which); break;          /* signed overflow */
    case 2: printf("%d\n", 1 << shift); break;          /* shift into the sign bit */
    case 3: printf("%d\n", a[idx] / (which - 3)); break;/* out of bounds; then division by zero */
    default: printf("ok\n");
    }
    return 0;
}
```

```console
$ clang -g -fsanitize=undefined sanitize.c -o san_ub && ./san_ub 1 && ./san_ub 2
sanitize.c:11:30: runtime error: signed integer overflow: 2147483647 + 1 cannot be represented in type 'int'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior sanitize.c:11:30 
-2147483648
sanitize.c:12:30: runtime error: left shift of 1 by 31 places cannot be represented in type 'int'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior sanitize.c:12:30 
-2147483648
$ clang -g -fsanitize=address,undefined sanitize.c -o san_both && ./san_both 3 2>&1 | grep -E 'ERROR|SUMMARY' | head -n 2
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior sanitize.c:13:28 
==2427==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7f98deb00038 at pc 0x5613b100dbe7 bp 0x7ffff288aed0 sp 0x7ffff288aec8
```

UBSan reports and *continues* by default; `-fno-sanitize-recover=all` makes it stop at the first error (what you want in tests). ASan always stops at the first memory error. Useful run-time options:

- `ASAN_OPTIONS=detect_stack_use_after_return=1` (catch use of returned stack frames), `detect_leaks=0` (turn off LSan), `abort_on_error=1` (dump core for GDB).
- `UBSAN_OPTIONS=print_stacktrace=1` (full stacks), `halt_on_error=1`.
- Suppression files for known issues in third-party code.

### MemorySanitizer

```c
// file: msan.c
#include <stdio.h>

int main(int argc, char *argv[]) {
    (void)argv;
    int value;                      /* uninitialised unless argc > 1 */
    if (argc > 1)
        value = 1;
    if (value > 0)                  /* branch on an uninitialised value */
        printf("positive\n");
    return 0;
}
```

```console
$ clang -g -fsanitize=memory -fsanitize-memory-track-origins msan.c -o msan && ./msan 2>&1 | grep -E 'WARNING|#0|Uninit|created' | head -n 4
==2435==WARNING: MemorySanitizer: use-of-uninitialized-value
    #0 0x555b20c3e527 in main msan.c:9:9
  Uninitialized value was created by an allocation of 'value' in the stack frame
    #0 0x555b20c3e469 in main msan.c:6:5
```

MSan reports the branch (`msan.c:9`) and, with origin tracking, where the uninitialised value came from (the stack variable in `main`). ASan cannot find this class of bug; Valgrind can (Unit 5.4).

### Deep & Tricky

**DT-56. Sanitizers only see the paths you run.** *(Tool behaviour)* A sanitizer report is proof of a bug; its absence is not proof of correctness. Combine sanitizers with tests that cover edge cases and with fuzzing (Chapter 12), which generates inputs specifically to reach new paths.

### Mastery check

1. **Explain.** Why can't ASan and MSan be used in the same build, and why must MSan builds instrument every library?

## Unit 5.4 — Valgrind

**One-line summary:** Valgrind runs an unmodified binary on a synthetic CPU and checks every memory access. Its default tool, **Memcheck**, finds invalid reads and writes on the heap, use of uninitialised values, leaks, and bad frees — without recompiling — at a 20–50× slowdown. Its other tools profile caches (Cachegrind), calls (Callgrind), and heap usage (Massif).

```console
$ gcc -g -O0 msan.c -o msan_plain && valgrind -q ./msan_plain 2>&1 | grep -E 'Conditional|at 0x' | head -n 2
==2445== Conditional jump or move depends on uninitialised value(s)
==2445==    at 0x10916D: main (msan.c:9)
$ gcc -g sanitize.c -o san_plain && valgrind -q ./san_plain 0 2>&1 | head -n 3
ok
```

Memcheck found the uninitialised branch in the plain binary. It does **not** find the stack out-of-bounds read in `sanitize.c` (case 3): Valgrind tracks heap blocks precisely but treats the stack as one valid region. Use ASan for stack and global overflows, Valgrind for code you cannot recompile and for uninitialised values, and both in CI if you can afford it.

Reading a Memcheck report: every error has a type (`Invalid read of size 4`, `Conditional jump or move depends on uninitialised value(s)`, `Invalid free()`), a stack where it happened, and for heap errors a description of the nearest block ("Address … is 0 bytes after a block of size 16 alloc'd at …"). Leak summaries classify blocks as *definitely lost* (no pointer to them remains — a real leak), *indirectly lost* (reachable only from definitely-lost blocks), *possibly lost* (only interior pointers remain), and *still reachable* (pointers remain at exit; usually harmless global buffers).

<!-- expanded -->
### Going deeper: leak kinds and where an uninitialised value came from

Valgrind's leak report sorts lost memory into categories that point at different bugs:

- **definitely lost** — no pointer to the block exists anywhere: a classic leak.
- **indirectly lost** — the block is reachable only through a block that is itself lost (the nodes of a leaked list). Fix the definitely-lost block and these disappear with it.
- **possibly lost** — only a pointer into the *middle* of the block exists; often a real leak, sometimes a clever data structure.
- **still reachable** — a pointer still exists at exit (a global cache never freed). Usually harmless, but it hides real leaks in long-running programs.

For uninitialised values, the first report says where the value was *used*; `--track-origins=yes` adds where it was *created* — often far away.

```c
// file: leakkinds.c
#include <stdio.h>
#include <stdlib.h>

struct node { int value; struct node *next; };

static struct node *cache;                   /* still reachable at exit */

static struct node *make_list(int n) {
    struct node *head = NULL;
    for (int i = 0; i < n; i++) {
        struct node *nd = malloc(sizeof *nd);
        nd->value = i;
        nd->next = head;
        head = nd;
    }
    return head;
}

static int average(const int *v, int n) {
    int sum;                                  /* never initialised */
    for (int i = 0; i < n; i++) sum += v[i];
    return sum / n;
}

int main(void) {
    struct node *list = make_list(3);
    list = NULL;                              /* head lost: definitely; the other 2 nodes: indirectly */
    cache = make_list(1);
    int v[3] = {1, 2, 3};
    if (average(v, 3) > 1) puts("above one");
    return 0;
}
```

```console
$ gcc -std=c17 -g -O0 leakkinds.c -o leakkinds && valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./leakkinds 2>&1 | grep -E "Conditional|Uninitialised value was created|at 0x.*(main|average)|definitely lost:|indirectly lost:|still reachable:" | sed -E 's/==[0-9]+==//; s/0x[0-9A-F]+/ADDR/g'
 Conditional jump or move depends on uninitialised value(s)
    at ADDR: main (leakkinds.c:31)
  Uninitialised value was created by a stack allocation
    at ADDR: average (leakkinds.c:20)
    definitely lost: 16 bytes in 1 blocks
    indirectly lost: 32 bytes in 2 blocks
    still reachable: 16 bytes in 1 blocks
```

The origin line points at the stack allocation in `average` — the forgotten initialiser — while the use was reported in `main`, where the bad sum finally reached an `if`. Valgrind reports uninitialised memory only when it affects control flow or a system call, not on every copy, which is why the two places differ.

### Deep & Tricky

**DT-57. AddressSanitizer does not detect reads of uninitialised memory at all; Valgrind's Memcheck does (at the point where the value influences behaviour), and Clang's MemorySanitizer does at compile-instrumented speed — so "clean under ASan" says nothing about uninitialised data.** *(Tool behaviour: AddressSanitizer documentation, "AddressSanitizer does not detect … uses of uninitialized memory (use MemorySanitizer for that)")* ASan and Valgrind cannot be combined in one run (both replace `malloc`), so a thorough CI runs the tests twice: once with ASan+UBSan for bounds and lifetime errors, once under Valgrind (or MSan, which requires every library to be instrumented too) for uninitialised reads.

### Mastery check

1. **Predict.** A program allocates a linked list of 10 nodes and frees only the head pointer's variable (not the nodes) before exiting. How does Memcheck classify the head node and the other nine?

## Unit 5.5 — Debugging with GDB

**One-line summary:** GDB runs your program under control: stop at **breakpoints**, step line by line, print variables and memory, walk the call stack, stop when a variable changes (**watchpoints**), and examine a crashed program's **core dump** after the fact. Compile with `-g` and, for the clearest view, `-O0` or `-Og`.

```c
// file: crash.c
#include <stdio.h>

struct node {
    int value;
    struct node *next;
};

static int sum(const struct node *n) {
    int total = 0;
    while (n != NULL) {
        total += n->value;
        n = n->next;
    }
    return total;
}

int main(void) {
    struct node c = {3, NULL}, b = {2, &c}, a = {1, &b};
    c.next = (struct node *)0x8;        /* corrupt the list */
    printf("sum = %d\n", sum(&a));
    return 0;
}
```

```console
$ gcc -g -Og crash.c -o crash && gdb -q -batch -ex run -ex bt -ex 'print n' -ex 'print total' -ex 'frame 1' -ex 'print c' ./crash 2>&1 | grep -v -E '^\[|libthread_db|^$'
Program received signal SIGSEGV, Segmentation fault.
sum (n=0x8, n@entry=0x7fffffff9d60) at crash.c:12
12	        total += n->value;
#0  sum (n=0x8, n@entry=0x7fffffff9d60) at crash.c:12
#1  0x00005555555551d9 in main () at crash.c:21
$1 = (const struct node *) 0x8
$2 = 6
#1  0x00005555555551d9 in main () at crash.c:21
21	    printf("sum = %d\n", sum(&a));
$3 = {value = 3, next = 0x8}
```

The batch run shows the whole crash investigation in five commands: the program received `SIGSEGV` in `sum` at line 12; `bt` (backtrace) shows `sum` was called from `main` at line 21; `n` is `0x8` — not a valid pointer — and `total` is 6, so three nodes were summed before following the corrupt `next`; `frame 1` moves to `main`, where printing `c` reveals the corrupt `next` field.

### The commands that matter

| Command | Does |
| --- | --- |
| `run [args]`, `start` | run the program (start stops at `main`) |
| `break file.c:42`, `break func`, `break 42 if i == 7` | breakpoints, optionally conditional |
| `next` (`n`), `step` (`s`), `finish`, `continue` (`c`) | step over a line, into a call, out of a function, resume |
| `print expr` (`p`), `p *ptr`, `p arr[0]@10`, `p/x val` | evaluate C expressions; `@` prints an array slice; `/x` in hex |
| `x/16xb ptr` | examine 16 bytes of memory in hex |
| `bt`, `frame N`, `up`, `down` | the call stack |
| `info locals`, `info args` | all variables in the current frame |
| `watch var`, `watch -l *ptr` | stop when the value changes (hardware watchpoint) |
| `display expr` | print after every step |
| `tui enable` | source view in the terminal |

### Core dumps

When a process crashes, Linux can write its memory to a **core file**; `gdb ./prog core` then shows exactly the state at the crash — invaluable for crashes in production or in CI. Enable with `ulimit -c unlimited`; where the file goes is controlled by `/proc/sys/kernel/core_pattern` (on systemd systems, `coredumpctl gdb` retrieves it).

### Watchpoints find corruptors

When a variable has a wrong value and you don't know who wrote it, `watch -l variable` stops execution at the exact instruction that changes it — often in a completely different part of the program (a buffer overflow next door). This is the fastest way to find memory corruption that ASan cannot see, for instance writes within the same struct.

### Mastery check

1. **Write.** Use GDB on `crash.c` with a breakpoint in `sum` and `display n->value` to watch the traversal step by step.
2. **Explain.** Why is `-O2` code harder to debug? (What does "value optimized out" mean?)

## Topic 5C — Correctness by design

## Unit 5.6 — Error handling and assertions

**One-line summary:** C has no exceptions. Choose one error convention per API — return codes with outputs through pointers, `NULL`/sentinel returns with `errno`, or an error struct — and apply it consistently; check every call that can fail; and use `assert` only for programmer errors (violated invariants), never for run-time conditions like missing files.

### Conventions

| Convention | Example | When |
| --- | --- | --- |
| return 0 / negative error code, results via pointer | `int parse(const char *s, int *out)` | most library functions; Linux kernel style (`-EINVAL`) |
| return pointer or `NULL`, details in `errno` | `FILE *fopen(...)` | constructors and lookups |
| return count or −1, details in `errno` | `ssize_t read(...)` | POSIX I/O |
| error struct / message buffer | `Ini *ini_parse_file(path, err, errsz)` (Chapter 4) | when users need a descriptive message |

`errno` (from `<errno.h>`) is a thread-local integer set by library functions *on failure only* — they never clear it on success, so check the return value first, then `errno`. `strerror(errno)` and `perror("context")` turn it into text.

```c
// file: errors.c
#include <assert.h>
#include <errno.h>
#include <stdio.h>
#include <string.h>

enum { ERR_OK = 0, ERR_RANGE = -1, ERR_IO = -2 };

static const char *err_text(int e) {
    switch (e) {
    case ERR_OK: return "ok";
    case ERR_RANGE: return "value out of range";
    case ERR_IO: return "I/O error";
    }
    return "unknown error";
}

/* percent computes part/whole as a percentage. whole must be positive: a caller bug otherwise. */
static int percent(int part, int whole, int *out) {
    assert(whole > 0 && "percent: whole must be positive");     /* programmer error */
    if (part < 0 || part > whole)
        return ERR_RANGE;                                       /* run-time condition */
    *out = (int)((long long)part * 100 / whole);
    return ERR_OK;
}

int main(void) {
    int p;
    int rc = percent(30, 120, &p);
    printf("percent(30,120): %s, %d%%\n", err_text(rc), rc == ERR_OK ? p : -1);
    rc = percent(130, 120, &p);
    printf("percent(130,120): %s\n", err_text(rc));

    FILE *f = fopen("/nonexistent/file", "r");
    if (f == NULL)
        printf("fopen failed: errno=%d (%s)\n", errno, strerror(errno));

    percent(0, 0, &p);                                          /* violates the precondition */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -g errors.c -o errors && ./errors; echo "exit $?"
percent(30,120): ok, 25%
percent(130,120): value out of range
fopen failed: errno=2 (No such file or directory)
errors: errors.c:20: percent: Assertion `whole > 0 && "percent: whole must be positive"' failed.
bash: line 1:  2485 Aborted                 ./errors
exit 134
$ gcc -std=c17 -Wall -Wextra -DNDEBUG errors.c -o errors_ndebug && ./errors_ndebug > /dev/null; echo "exit $?"
bash: line 1:  2492 Floating point exception./errors_ndebug > /dev/null
exit 136
```

With assertions enabled, the precondition violation aborted the program with the file, line, function, and condition (the `&& "message"` trick puts a message in the condition text). With `-DNDEBUG` the assert vanished, `percent(0, 0, …)` divided by zero, and the process died from `SIGFPE` — showing why assertions are for *bugs* you want found during development, and why defensive code must not rely on them.

C23 and GCC also offer `static_assert(sizeof(int) == 4, "…")` (C11: `_Static_assert`) for conditions checkable at compile time — sizes, layouts, configuration constants.

### Pattern

**P-7. Check every return value; propagate errors with context; assert only invariants.** *(Practice)* Every call that can fail is checked; errors are passed up unchanged or wrapped with context (`"loading %s: %s"`); cleanup happens on one path (`goto` ladder); `assert` documents and checks conditions that indicate bugs, and production builds decide consciously whether to keep them. Mark functions whose result must be checked with `__attribute__((warn_unused_result))` (C23: `[[nodiscard]]`). *Trade-off:* verbose code in exchange for failures that are reported where they happen instead of corrupting state silently.

<!-- expanded -->
### Going deeper: `setjmp` and `longjmp`

`<setjmp.h>` provides a non-local jump: `setjmp(env)` saves the current execution context and returns 0; a later `longjmp(env, v)` — from any function called after it, however deep — returns from that same `setjmp` a second time, now with value `v`. It is C's closest thing to exceptions, and it is used for exactly that in some libraries (libpng and Lua report errors this way) and in interpreters.

```c
// file: jump.c
#include <setjmp.h>
#include <stdio.h>
#include <stdlib.h>

static jmp_buf on_error;

static double parse_number(const char *s) {
    char *end;
    double d = strtod(s, &end);
    if (end == s || *end != '\0') longjmp(on_error, 1);    /* "throw" */
    return d;
}

static double sum(const char *const *items, int n) {
    double total = 0;
    for (int i = 0; i < n; i++) total += parse_number(items[i]);
    return total;
}

int main(void) {
    const char *good[] = {"1.5", "2", "3.25"};
    const char *bad[] = {"1", "two", "3"};
    volatile int attempts = 0;                /* modified after setjmp and read after longjmp: must be volatile */

    if (setjmp(on_error) == 0) {              /* "try" */
        attempts++;
        printf("sum of good = %.2f\n", sum(good, 3));
        attempts++;
        printf("sum of bad = %.2f\n", sum(bad, 3));
    } else {                                  /* "catch": setjmp returned a second time */
        printf("parse error after %d attempt(s)\n", attempts);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 jump.c -o jump && ./jump
sum of good = 6.75
parse error after 2 attempt(s)
```

`longjmp` unwinds nothing: memory allocated, files opened, and locks taken in the functions it jumps over are simply abandoned. That is the main reason it is rare in application code — every resource between the `setjmp` and the `longjmp` needs a separate cleanup plan.

### Deep & Tricky

**DT-58. After `longjmp`, the values of local variables of the function that called `setjmp` are indeterminate if they are not `volatile` and were changed between the `setjmp` and the `longjmp`.** *(Guarantee: C17 §7.13.2.1 ¶3)* The optimiser may keep such a variable in a register, and `longjmp` restores the registers saved by `setjmp` — so a counter incremented after `setjmp` can "lose" its updates. GCC warns with `-Wclobbered` (in `-Wextra`) when it sees the danger. Calling `setjmp` itself is also restricted: it may appear only as the whole controlling expression of an `if`/`switch`/loop, possibly compared with a constant, or as an expression statement — not, for example, as `int r = setjmp(env);`.

```c
// file: clobber.c
#include <setjmp.h>
#include <stdio.h>

static jmp_buf env;
static void fail(void) { longjmp(env, 1); }

int main(void) {
    int steps = 0;                            /* not volatile */
    if (setjmp(env) == 0) {
        for (int i = 0; i < 5; i++) steps++;
        fail();
    }
    printf("steps = %d\n", steps);
    return 0;
}
```

```console
$ gcc -std=c17 -O0 clobber.c -o clobber0 && ./clobber0
steps = 5
$ gcc -std=c17 -O2 clobber.c -o clobber2 && ./clobber2
steps = 0
```

The unoptimised build kept `steps` in memory and printed 5; at `-O2` GCC kept it in a register, and `longjmp` brought back the value saved at `setjmp` time: 0. Same source, two answers — the definition of indeterminate.

### Mastery check

1. **Explain.** Why must you not check `errno` without first checking the function's return value?
2. **Write.** Add `[[nodiscard]]`-style warnings (`__attribute__((warn_unused_result))`) to your INI library's functions, and see which call sites the compiler flags.

## Unit 5.7 — Testing C code

**One-line summary:** C has no built-in test framework, but tests are just programs: functions that call your code with known inputs, check outputs, and report failures through the exit status. A few macros give you a usable framework; `gcov` shows which lines your tests execute; and running the test suite under sanitizers multiplies its power.

### A minimal test framework

```c
// file: testing/minitest.h
#ifndef MINITEST_H
#define MINITEST_H

#include <stdio.h>
#include <string.h>

static int mt_failures, mt_checks;

#define CHECK(cond) do {                                                   \
        mt_checks++;                                                       \
        if (!(cond)) {                                                     \
            mt_failures++;                                                 \
            fprintf(stderr, "%s:%d: %s: CHECK(%s) failed\n",               \
                    __FILE__, __LINE__, __func__, #cond);                  \
        }                                                                  \
    } while (0)

#define CHECK_EQ_INT(a, b) do {                                            \
        long long a_ = (a), b_ = (b);                                      \
        mt_checks++;                                                       \
        if (a_ != b_) {                                                    \
            mt_failures++;                                                 \
            fprintf(stderr, "%s:%d: %s: %s == %lld, expected %lld\n",      \
                    __FILE__, __LINE__, __func__, #a, a_, b_);             \
        }                                                                  \
    } while (0)

#define CHECK_EQ_STR(a, b) do {                                            \
        const char *a_ = (a), *b_ = (b);                                   \
        mt_checks++;                                                       \
        if (strcmp(a_, b_) != 0) {                                         \
            mt_failures++;                                                 \
            fprintf(stderr, "%s:%d: %s: %s == \"%s\", expected \"%s\"\n",   \
                    __FILE__, __LINE__, __func__, #a, a_, b_);             \
        }                                                                  \
    } while (0)

#define RUN(test) do { test(); } while (0)

#define REPORT() (fprintf(stderr, "%d checks, %d failures\n", mt_checks, mt_failures), \
                  mt_failures == 0 ? 0 : 1)

#endif
```

Each check reports the file, line, test function, expression, and the actual and expected values — enough to understand a failure without a debugger. (Established frameworks — Unity, cmocka, Check, Criterion, and GoogleTest for C++ — add fixtures, test discovery, and XML output for CI; the principles are the same.)

```c
// file: testing/roman.c
#include "roman.h"

#include <string.h>

static const struct { int value; const char *sym; } table[] = {
    {1000, "M"}, {900, "CM"}, {500, "D"}, {400, "CD"}, {100, "C"}, {90, "XC"},
    {50, "L"}, {40, "XL"}, {10, "X"}, {9, "IX"}, {5, "V"}, {4, "IV"}, {1, "I"},
};

int to_roman(int n, char *buf, size_t size) {
    if (n <= 0 || n >= 4000 || size == 0)
        return -1;
    size_t used = 0;
    for (size_t i = 0; i < sizeof table / sizeof table[0]; i++) {
        while (n >= table[i].value) {
            size_t len = strlen(table[i].sym);
            if (used + len >= size)
                return -1;
            memcpy(buf + used, table[i].sym, len);
            used += len;
            n -= table[i].value;
        }
    }
    buf[used] = '\0';
    return 0;
}
```

```c
// file: testing/roman.h
#ifndef ROMAN_H
#define ROMAN_H
#include <stddef.h>

/* to_roman writes n (1..3999) as Roman numerals into buf. Returns 0, or -1 if
   n is out of range or buf is too small. */
int to_roman(int n, char *buf, size_t size);

#endif
```

```c
// file: testing/test_roman.c
#include "minitest.h"
#include "roman.h"

static void test_known_values(void) {
    static const struct { int n; const char *want; } cases[] = {
        {1, "I"}, {4, "IV"}, {9, "IX"}, {14, "XIV"}, {40, "XL"}, {90, "XC"},
        {400, "CD"}, {1994, "MCMXCIV"}, {2024, "MMXXIV"}, {3999, "MMMCMXCIX"},
    };
    for (size_t i = 0; i < sizeof cases / sizeof cases[0]; i++) {
        char buf[32];
        CHECK_EQ_INT(to_roman(cases[i].n, buf, sizeof buf), 0);
        CHECK_EQ_STR(buf, cases[i].want);
    }
}

static void test_out_of_range(void) {
    char buf[32];
    CHECK_EQ_INT(to_roman(0, buf, sizeof buf), -1);
    CHECK_EQ_INT(to_roman(-5, buf, sizeof buf), -1);
    CHECK_EQ_INT(to_roman(4000, buf, sizeof buf), -1);
}

static void test_small_buffer(void) {
    char buf[4];
    CHECK_EQ_INT(to_roman(3, buf, sizeof buf), 0);        /* "III" + '\0' fits exactly */
    CHECK_EQ_INT(to_roman(8, buf, sizeof buf), -1);       /* "VIII" does not */
}

int main(void) {
    RUN(test_known_values);
    RUN(test_out_of_range);
    RUN(test_small_buffer);
    return REPORT();
}
```

```console
$ cd testing && gcc -std=c17 -Wall -Wextra -g -fsanitize=address,undefined roman.c test_roman.c -o test_roman && ./test_roman; echo "exit $?"
25 checks, 0 failures
exit 0
```

### Coverage with gcov

```console
$ cd testing && rm -f *.gcda *.gcno && gcc -std=c17 --coverage -O0 -c roman.c test_roman.c && gcc --coverage roman.o test_roman.o -o test_cov && ./test_cov 2>/dev/null && gcov roman.c | head -n 2
File 'roman.c'
Lines executed:100.00% of 14
$ cd testing && grep -E '#####' roman.c.gcov || echo "every line of roman.c executed"
every line of roman.c executed
```

`--coverage` instruments the program to count how often each line runs; `gcov` reports percentages and writes annotated sources (`roman.c.gcov`) where never-executed lines are marked `#####`. The `size == 0` check is on the same line as the range check, so line coverage calls it covered although no test passes `size == 0` — a reminder that 100% line coverage is a floor, not a proof. Branch coverage (`gcov -b`) is stricter.

### What to test in C specifically

- Boundaries: 0, 1, maximum, one past maximum, empty strings, buffers exactly one byte too small.
- Every error path: allocation failures (inject them with a wrapper around `malloc`), I/O errors, malformed input.
- Ownership: run every test under ASan so leaks and double frees fail the suite.

<!-- expanded -->
### Going deeper: table-driven and differential tests

Two techniques make C test suites both shorter and stronger.

A **table-driven test** puts the cases in an array of structs and checks them in one loop. Adding a case is one line, and the failure message prints the case, so the report says exactly which input failed.

A **differential test** compares the function under test with a second, simpler (usually slower) implementation of the same specification, on thousands of random inputs. It finds the cases nobody thought to write down.

```c
// file: difftest.c
#include <limits.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

/* function under test: clamp a + b to [INT_MIN, INT_MAX] without overflowing */
static int sat_add(int a, int b) {
    if (b > 0 && a > INT_MAX - b) return INT_MAX;
    if (b < 0 && a < INT_MIN - b) return INT_MIN;
    return a + b;
}

/* reference: obviously correct, using a wider type */
static int sat_add_ref(int a, int b) {
    int64_t r = (int64_t)a + b;
    return r > INT_MAX ? INT_MAX : r < INT_MIN ? INT_MIN : (int)r;
}

/* an "optimised" version with a subtle bug */
static int sat_add_fast(int a, int b) {
    int r = (int)((unsigned)a + (unsigned)b);       /* wraps, no UB */
    if (a > 0 && b > 0 && r < 0) return INT_MAX;
    if (a < 0 && b < 0 && r > 0) return INT_MIN;    /* bug: misses r == 0, i.e. INT_MIN + INT_MIN */
    return r;
}

static int random_int(void) {                       /* bias towards the edges, where bugs live */
    static const int edges[] = {0, 1, -1, INT_MAX, INT_MIN, INT_MAX - 1, INT_MIN + 1};
    if (rand() % 4 == 0) return edges[rand() % 7];
    return (int)((unsigned)rand() << 16 ^ (unsigned)rand());
}

int main(void) {
    static const struct { int a, b, want; } table[] = {
        {1, 2, 3}, {INT_MAX, 1, INT_MAX}, {INT_MIN, -1, INT_MIN}, {INT_MAX, INT_MIN, -1}, {-5, 5, 0},
    };
    int failures = 0;
    for (size_t i = 0; i < sizeof table / sizeof table[0]; i++) {
        int got = sat_add(table[i].a, table[i].b);
        if (got != table[i].want) { printf("table: sat_add(%d, %d) = %d, want %d\n", table[i].a, table[i].b, got, table[i].want); failures++; }
    }
    printf("table: %zu cases, %d failures\n", sizeof table / sizeof table[0], failures);

    srand(1);
    long checked = 0, diff_ok = 0, diff_fast = 0;
    for (; checked < 1000000; checked++) {
        int a = random_int(), b = random_int(), want = sat_add_ref(a, b);
        if (sat_add(a, b) != want) diff_ok++;
        if (sat_add_fast(a, b) != want && diff_fast++ == 0)
            printf("differential: sat_add_fast(%d, %d) = %d, reference says %d\n", a, b, sat_add_fast(a, b), want);
    }
    printf("differential: %ld random pairs; sat_add disagreed %ld times, sat_add_fast %ld times\n", checked, diff_ok, diff_fast);
    return failures || diff_ok;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 -fsanitize=undefined difftest.c -o difftest && ./difftest; echo "exit status $?"
table: 5 cases, 0 failures
differential: sat_add_fast(-2147483648, -2147483648) = 0, reference says -2147483648
differential: 1000000 random pairs; sat_add disagreed 0 times, sat_add_fast 1270 times
exit status 0
```

The table passed, and so did a million random pairs for `sat_add`. The "fast" version looks right and handles the usual overflow cases — but the differential test found the input that breaks it on its first failure: `INT_MIN + INT_MIN`, whose wrapped sum is exactly 0, so neither overflow test fires. (Because the random generator favours edge values, it hit that pair often.) Running under UBSan at the same time proves that neither implementation relies on signed overflow. Chapter 12 takes the idea further with fuzzers, which generate inputs guided by code coverage.

### Mastery check

1. **Write.** Add `from_roman(const char *s, int *out)` with tests, including invalid numerals (`"IIII"`, `"IC"`, `""`), and reach 100% branch coverage.

## Chapter 5 Project — Find and fix ten planted bugs

**Goal:** a small program, `scores`, reads student names and scores from a file and prints a report. The version below contains **ten planted bugs** of different kinds. Find all ten using the tools from this chapter — each bug is detectable by at least one of: compiler warnings, a static analyser, ASan, UBSan, LSan, Valgrind, or a unit test — then fix them, and prove the fixed program clean under every tool.

### The buggy program

```c
// file: scores/scores_buggy.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *name;
    double score;
} Record;

static int cmp_score(const void *a, const void *b) {
    const Record *x = a, *y = b;
    return (int)(y->score - x->score);                  /* sort descending */
}

static double median(const Record *r, int n) {
    if (n % 2 == 1)
        return r[n / 2].score;
    return (r[n / 2].score + r[n / 2 + 1].score) / 2;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: scores FILE\n");
        return 2;
    }
    FILE *f = fopen(argv[1], "r");
    if (f == NULL) {
        perror(argv[1]);
        return 1;
    }
    Record *recs = NULL;
    int n = 0, cap = 0;
    char line[128];
    while (fgets(line, sizeof line, f) != NULL) {
        char name[64];
        double score;
        sscanf(line, "%63s %lf", name, &score);
        if (n == cap) {
            cap = cap ? cap * 2 : 4;
            recs = realloc(recs, cap * sizeof *recs);
        }
        recs[n].name = malloc(strlen(name));
        strcpy(recs[n].name, name);
        recs[n].score = score;
        n++;
    }
    fclose(f);

    double sum;
    for (int i = 0; i < n; i++)
        sum += recs[i].score;
    qsort(recs, n, sizeof *recs, cmp_score);
    int pass = 0;
    for (int i = 0; i < n; i++)
        if (recs[i].score >= 50)
            pass++;

    printf("students: %d\n", n);
    printf("average:  %.2f\n", sum / n);
    printf("median:   %.2f\n", median(recs, n));
    printf("passed:   %d%%\n", pass * 100 / n);
    printf("top:      %s (%d)\n", recs[0].name, recs[0].score);
    free(recs);
    printf("lowest:   %s\n", recs[n - 1].name);
    return 0;
}
```

```text
# file: scores/class.txt
alice 91.5
bob 47
carol 91
dave 63.25
erin
frank 78
```

(The `# file:` line is part of the sample data here; the program reads it as a student called `#`. That is itself a test of how the program handles malformed lines.)

### Hunting

**Step 1 — warnings.** Before running anything:

```console
$ cd scores && gcc -std=c17 -Wall -Wextra -O2 -g scores_buggy.c -o scores_buggy 2>&1 | grep -E 'warning' | sed 's/^scores_buggy.c://'
63:28: warning: format ‘%d’ expects argument of type ‘int’, but argument 3 has type ‘double’ [-Wformat=]
65:34: warning: pointer ‘recs’ used after ‘free’ [-Wuse-after-free]
44:9: warning: ‘__builtin___strcpy_chk’ writing one too many bytes into a region of a size that depends on ‘strlen’ [-Wstringop-overflow=]
60:5: warning: ‘sum’ may be used uninitialized [-Wmaybe-uninitialized]
```

The compiler alone found four bugs. **Bug 1**: `%d` used for a `double` in the "top" line. **Bug 5**: `recs` used after `free` in the "lowest" line (`-Wuse-after-free`, GCC 12+). **Bug 6**: `malloc(strlen(name))` leaves no room for the terminator, so `strcpy` writes one byte too many — GCC proved it because `_FORTIFY_SOURCE` turned `strcpy` into a checked call. **Bug 2**: `sum` is used uninitialised. Four of ten bugs, for the price of reading compiler output.

**Step 2 — static analysis.**

```console
$ cd scores && gcc -std=c17 -fanalyzer -c scores_buggy.c 2>&1 | grep -E 'warning:' | sed 's/^scores_buggy.c://' | sort -u
41:20: warning: leak of ‘<unknown>’ [CWE-401] [-Wanalyzer-malloc-leak]
43:22: warning: dereference of NULL ‘recs’ [CWE-476] [-Wanalyzer-null-dereference]
44:9: warning: use of possibly-NULL ‘*<unknown>.name’ where non-null expected [CWE-690] [-Wanalyzer-possible-null-argument]
45:13: warning: leak of ‘recs’ [CWE-401] [-Wanalyzer-malloc-leak]
52:13: warning: use of uninitialized value ‘sum’ [CWE-457] [-Wanalyzer-use-of-uninitialized-value]
52:23: warning: dereference of NULL ‘recs’ [CWE-476] [-Wanalyzer-null-dereference]
53:5: warning: use of NULL ‘recs’ where non-null expected [CWE-476] [-Wanalyzer-null-argument]
63:28: warning: format ‘%d’ expects argument of type ‘int’, but argument 3 has type ‘double’ [-Wformat=]
```

`-fanalyzer` adds **bug 4**: `realloc`'s result is assigned straight to `recs`, so on failure the old block leaks (line 45) and `recs` becomes `NULL` and is dereferenced (lines 43, 52). It also flags the unchecked `malloc` result (line 44), which the fix handles.

**Step 3 — AddressSanitizer and LeakSanitizer.**

```console
$ cd scores && gcc -std=c17 -g -fsanitize=address scores_buggy.c -o scores_asan 2>/dev/null && ./scores_asan class.txt 2>&1 | grep -E 'ERROR|#[01] ' | head -n 3
==2564==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x502000000011 at pc 0x7faf906a7923 bp 0x7ffc21dc9e10 sp 0x7ffc21dc95b8
    #0 0x7faf906a7922 in strcpy ../../../../src/libsanitizer/asan/asan_interceptors.cpp:563
    #1 0x560082bb487b in main scores/scores_buggy.c:44
```

ASan confirms bug 6 at run time (`heap-buffer-overflow` in `strcpy`, called from line 44). To see what lies behind it, apply the fixes for bugs 5 and 6 and run again:

```console
$ cd scores && sed -e 's/malloc(strlen(name))/malloc(strlen(name) + 1)/' -e 's/^    free(recs);$//' -e 's/recs\[n - 1\].name);/recs[n - 1].name); free(recs);/' scores_buggy.c > partial.c && gcc -std=c17 -g -fsanitize=address partial.c -o partial 2>/dev/null && ./partial class.txt 2>&1 | grep -E 'ERROR|SUMMARY'
==2574==ERROR: LeakSanitizer: detected memory leaks
SUMMARY: AddressSanitizer: 34 byte(s) leaked in 7 allocation(s).
```

LeakSanitizer reports **bug 7**: the seven name strings are never freed (34 bytes in 7 allocations — seven, because the malformed lines are stored as students too).

**Step 4 — Valgrind** on the same partially fixed program, without sanitizers:

```console
$ cd scores && gcc -std=c17 -g -O0 partial.c -o partial_plain 2>/dev/null && valgrind -q ./partial_plain class.txt 2>&1 | grep -E 'Conditional|at 0x' | head -n 2
==2583== Conditional jump or move depends on uninitialised value(s)
==2583==    at 0x48B27C9: msort_with_tmp.part.0 (qsort.c:275)
```

Memcheck finds **bug 3**: a conditional jump inside `qsort` depends on an uninitialised value. The line `erin` has no score, `sscanf` returns 1, and the unchecked, uninitialised `score` is stored and compared. Neither the compiler nor ASan reported this one.

**Step 5 — UndefinedBehaviorSanitizer** on an empty file:

```console
$ cd scores && : > empty.txt && gcc -std=c17 -g -fsanitize=undefined -fno-sanitize-recover=all scores_buggy.c -o scores_ubsan 2>/dev/null && ./scores_ubsan empty.txt 2>&1 | grep -E 'runtime error' | sed 's/^scores_buggy.c://'
53:5: runtime error: null pointer passed as argument 1, which is declared to never be null
```

**Bug 8**: the program does not handle zero students. UBSan stopped at the first consequence — `qsort` called with a null pointer — and behind it wait a division by zero in `pass * 100 / n`, a `median` that reads `r[-1]`, and `recs[0]` dereferenced while `NULL`. An empty input is the first edge case to test.

**Step 6 — tests for the logic.** Two bugs produce wrong numbers without any memory error or UB. Only a test with known answers finds them:

- **Bug 9**: `cmp_score` converts the *difference* to `int`, so 91.5 and 91 compare equal (`(int)0.5 == 0`) and their relative order is arbitrary — the "top" student may be wrong.
- **Bug 10**: `median` for even `n` averages elements `n/2` and `n/2 + 1`; it should use `n/2 - 1` and `n/2`. For `n == 2` it even reads `r[2]`, past the end.

### The fixed program

```c
// file: scores/scores.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *name;
    double score;
} Record;

/* cmp_score orders records by descending score, then by name for stable output. */
static int cmp_score(const void *a, const void *b) {
    const Record *x = a, *y = b;
    if (x->score != y->score)
        return x->score < y->score ? 1 : -1;              /* bug 9: compare, don't subtract */
    return strcmp(x->name, y->name);
}

/* median of r[0..n-1], which must be sorted; n > 0. */
static double median(const Record *r, int n) {
    if (n % 2 == 1)
        return r[n / 2].score;
    return (r[n / 2 - 1].score + r[n / 2].score) / 2;    /* bug 10 */
}

static void free_records(Record *recs, int n) {
    for (int i = 0; i < n; i++)
        free(recs[i].name);                               /* bug 7 */
    free(recs);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: scores FILE\n");
        return 2;
    }
    FILE *f = fopen(argv[1], "r");
    if (f == NULL) {
        perror(argv[1]);
        return 1;
    }
    Record *recs = NULL;
    int n = 0, cap = 0, lineno = 0, status = 0;
    char line[128];
    while (fgets(line, sizeof line, f) != NULL) {
        lineno++;
        char name[64];
        double score;
        if (sscanf(line, "%63s %lf", name, &score) != 2 || name[0] == '#') {   /* bug 3 */
            fprintf(stderr, "%s:%d: skipping malformed line\n", argv[1], lineno);
            continue;
        }
        if (n == cap) {
            int new_cap = cap ? cap * 2 : 4;
            Record *p = realloc(recs, (size_t)new_cap * sizeof *p);         /* bug 4 */
            if (p == NULL) {
                status = 1;
                break;
            }
            recs = p;
            cap = new_cap;
        }
        recs[n].name = malloc(strlen(name) + 1);                             /* bug 6 */
        if (recs[n].name == NULL) {
            status = 1;
            break;
        }
        strcpy(recs[n].name, name);
        recs[n].score = score;
        n++;
    }
    fclose(f);
    if (status != 0) {
        fprintf(stderr, "scores: out of memory\n");
        free_records(recs, n);
        return 1;
    }
    if (n == 0) {                                                            /* bug 8 */
        printf("students: 0\n");
        free_records(recs, n);
        return 0;
    }

    double sum = 0;                                                          /* bug 2 */
    for (int i = 0; i < n; i++)
        sum += recs[i].score;
    qsort(recs, (size_t)n, sizeof *recs, cmp_score);
    int pass = 0;
    for (int i = 0; i < n; i++)
        if (recs[i].score >= 50)
            pass++;

    printf("students: %d\n", n);
    printf("average:  %.2f\n", sum / n);
    printf("median:   %.2f\n", median(recs, n));
    printf("passed:   %d%%\n", pass * 100 / n);
    printf("top:      %s (%.2f)\n", recs[0].name, recs[0].score);           /* bug 1 */
    printf("lowest:   %s (%.2f)\n", recs[n - 1].name, recs[n - 1].score);
    free_records(recs, n);                                                   /* bug 5: free last */
    return 0;
}
```

### Proving it

```console
$ cd scores && gcc -std=c17 -Wall -Wextra -Wpedantic -O2 -g scores.c -o scores && echo "no warnings"
no warnings
$ cd scores && gcc -std=c17 -fanalyzer -c scores.c -o /dev/null 2>&1 | grep -c warning || true
0
$ cd scores && gcc -std=c17 -g -fsanitize=address,undefined -fno-sanitize-recover=all scores.c -o scores_san && ./scores_san class.txt && ./scores_san empty.txt
class.txt:1: skipping malformed line
class.txt:6: skipping malformed line
students: 5
average:  74.15
median:   78.00
passed:   80%
top:      alice (91.50)
lowest:   bob (47.00)
students: 0
$ cd scores && valgrind -q --leak-check=full --error-exitcode=9 ./scores class.txt > /dev/null 2>&1; echo "valgrind exit $?"
valgrind exit 0
```

The expected values are easy to verify by hand: the four valid students' scores sum to 91.5 + 47 + 91 + 63.25 + 78 = 370.75 over 5 students = 74.15; sorted descending 91.5, 91, 78, 63.25, 47, so the median is 78; four of five passed (80%); alice is top, bob lowest.

### Review checklist

- For each of the ten bugs, which tool found it first, and which tools would have missed it?
- Does the fixed version handle an empty file, a file of only malformed lines, a name of 63 and 64 characters, and identical scores?
- Is there a test (shell script or C test) that checks the report's numbers, so bugs 9 and 10 cannot return?

## Answers to the Chapter 5 prediction questions

**5.1 Q2.** No. Unsigned arithmetic wraps by definition, so `u + 1 > u` is genuinely false for `UINT_MAX` and the compiler must compute it.

**5.4 Q1.** The head node is *definitely lost* (nothing points to it); the other nine are *indirectly lost* (only the lost head chain points to them).

# Chapter 6 — The Standard Library and I/O

*Goal: use C's standard library confidently — streams, formatted and binary I/O, files, error handling, time, and the rest of `<stdlib.h>` — and know the modern additions of C11 through C23.*

The C standard library is small compared with those of newer languages, but it is everywhere: every C program on every platform has it. This chapter covers what professional C code uses daily — stdio streams and buffering, files in text and binary, portable binary formats, error handling around I/O, time, and utility functions — and finishes with a tour of the newer standards. The project builds clones of two Unix classics, `wc` and `grep`, and checks them against the real tools.

**What you will be able to do after this chapter**

- Explain stream buffering, when output appears, and how to control it.
- Use the `printf` and `scanf` families precisely, including widths, precisions, and length modifiers.
- Read and write text and binary files with complete error handling, and write files atomically.
- Serialise data portably regardless of the machine's endianness and padding.
- Measure time and format dates correctly.
- Use `qsort`, `bsearch`, `getenv`, `exit`/`atexit`, and know which library functions to avoid.
- Use the main features added in C11, C17, and C23.

## Topic 6A — Streams and formatted I/O

## Unit 6.1 — Streams and buffering

**One-line summary:** stdio functions operate on **streams** (`FILE *`): `stdin`, `stdout`, `stderr`, and files you open. Streams are **buffered** — output collects in memory and is written in blocks — with three modes: unbuffered (`stderr`), line-buffered (`stdout` on a terminal), and fully buffered (`stdout` to a pipe or file, and files). Buffering explains most "my output appeared in the wrong order" mysteries.

```c
// file: buffering.c
#include <stdio.h>
#include <unistd.h>

int main(void) {
    printf("1. to stdout ");          /* no newline: stays in the buffer */
    fprintf(stderr, "2. to stderr\n");
    printf("3. stdout again\n");
    fflush(stdout);                    /* force everything out now */

    printf("4. before fork-like surprise\n");
    write(STDOUT_FILENO, "5. raw write(2)\n", 16);   /* bypasses the stdio buffer */
    return 0;                          /* exit flushes remaining buffers */
}
```

```console
$ gcc -std=c17 -Wall buffering.c -o buffering && ./buffering
2. to stderr
1. to stdout 3. stdout again
4. before fork-like surprise
5. raw write(2)
$ ./buffering | cat
2. to stderr
1. to stdout 3. stdout again
5. raw write(2)
4. before fork-like surprise
$ ./buffering 2>/dev/null | cat
1. to stdout 3. stdout again
5. raw write(2)
4. before fork-like surprise
```

On a terminal, `stdout` is line-buffered: line 1 waits for a newline, so stderr's line 2 appears first. Piped into `cat`, `stdout` is fully buffered, so line 4 waits in the buffer until `exit` flushes it — after the unbuffered `write` of line 5. Same program, different order, determined only by where output goes.

Rules:

- `stderr` is unbuffered: use it for diagnostics so they appear immediately.
- `fflush(stream)` writes buffered output now. `fflush(stdin)` is undefined in standard C.
- `setvbuf(stream, buf, _IOLBF, size)` sets the mode (call it before any I/O on the stream); `setvbuf(stdout, NULL, _IONBF, 0)` disables buffering.
- `exit` (and returning from `main`) flushes all streams; `_exit` and a crash do not — output still in the buffer is lost. When a program crashes, its last `printf` lines may never appear, which misleads debugging. Print diagnostics to `stderr`.
- Mixing stdio and raw `write(2)` on the same file descriptor reorders output, as line 5 shows.

### Deep & Tricky

**DT-59. After `fork`, unflushed stdio buffers are duplicated.** *(POSIX + stdio behaviour)* If `stdout` holds unflushed data when a process forks, both parent and child have a copy and both print it at exit — output appears twice, but only when redirected to a file or pipe (fully buffered). Flush before `fork` (Chapter 8).

### Mastery check

1. **Predict.** What does `./buffering > out.txt; cat out.txt` show, and in what order?

## Unit 6.2 — The `printf` and `scanf` families

**One-line summary:** A conversion specification is `%[flags][width][.precision][length]conversion`. Each must match its argument's type exactly — `%d` for `int`, `%ld` for `long`, `%zu` for `size_t`, `%f` for `double` (also for `float`, which is promoted) — because a mismatch is undefined behaviour. `snprintf` is the safe way to build strings; never pass user input as a format string.

```c
// file: formats.c
#include <inttypes.h>
#include <stdint.h>
#include <stdio.h>

int main(void) {
    int n = 42;
    double pi = 3.14159265;
    printf("[%5d] [%-5d] [%05d] [%+d] [% d]\n", n, n, n, n, n);   /* width, left, zero, sign, space */
    printf("[%x] [%X] [%#x] [%o] [%#o]\n", 255, 255, 255, 8, 8);
    printf("[%.2f] [%8.3f] [%-8.1f] [%e] [%g] [%g]\n", pi, pi, pi, pi, pi, 1e-10);
    printf("[%s] [%.3s] [%10s] [%-10s]\n", "hello", "hello", "hi", "hi");
    printf("[%*d] [%.*f]\n", 6, n, 1, pi);                         /* width/precision from args */
    printf("[%c] [%%] [%p]\n", 'A', (void *)0);

    long big = 1234567890123L;
    size_t sz = sizeof big;
    int64_t i64 = -5;
    uint8_t byte = 200;
    printf("long %ld, size_t %zu, int64 %" PRId64 ", uint8 %" PRIu8 "\n", big, sz, i64, byte);

    int written = printf("abc\n");
    printf("printf returned %d\n", written);

    char buf[8];
    int needed = snprintf(buf, sizeof buf, "%d-%d-%d", 2025, 12, 31);
    printf("buf='%s', needed=%d, truncated=%s\n", buf, needed, needed >= (int)sizeof buf ? "yes" : "no");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra formats.c -o formats && ./formats
formats.c: In function ‘main’:
formats.c:26:51: warning: ‘%d’ directive output truncated writing 2 bytes into a region of size 0 [-Wformat-truncation=]
   26 |     int needed = snprintf(buf, sizeof buf, "%d-%d-%d", 2025, 12, 31);
      |                                                   ^~
formats.c:26:18: note: ‘snprintf’ output 11 bytes into a destination of size 8
   26 |     int needed = snprintf(buf, sizeof buf, "%d-%d-%d", 2025, 12, 31);
      |                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[   42] [42   ] [00042] [+42] [ 42]
[ff] [FF] [0xff] [10] [010]
[3.14] [   3.142] [3.1     ] [3.141593e+00] [3.14159] [1e-10]
[hello] [hel] [        hi] [hi        ]
[    42] [3.1]
[A] [%] [(nil)]
long 1234567890123, size_t 8, int64 -5, uint8 200
abc
printf returned 4
buf='2025-12', needed=10, truncated=yes
```

GCC noticed at compile time that the `snprintf` output (11 bytes) cannot fit the 8-byte buffer (`-Wformat-truncation`); at run time `snprintf` truncated safely and returned the length it needed. `%p` of a null pointer prints `(nil)` in glibc — the format of `%p` is implementation-defined.

### Length modifiers

| Modifier | `d`/`i` | `u`/`x`/`o` | Other |
| --- | --- | --- | --- |
| `hh` | `signed char` | `unsigned char` | |
| `h` | `short` | `unsigned short` | |
| (none) | `int` | `unsigned` | `%f`: `double` |
| `l` | `long` | `unsigned long` | `%lf`: `double` (only matters in `scanf`) |
| `ll` | `long long` | `unsigned long long` | |
| `z` | `ssize_t` (POSIX) | `size_t` | |
| `t` | `ptrdiff_t` | | |
| `j` | `intmax_t` | `uintmax_t` | |
| `L` | | | `%Lf`: `long double` |

`<inttypes.h>` provides macros like `PRId64` and `SCNd64` for the fixed-width types, because their underlying type (and thus the right modifier) differs between platforms.

### Format-string vulnerabilities

`printf(user_input)` lets the user write conversion specifications: `%x%x%x` reads stack contents, and `%n` *writes* to memory. Always write `printf("%s", user_input)` or `fputs(user_input, stdout)`. GCC warns with `-Wformat-security` (enabled by `-Wformat=2`), and glibc refuses `%n` in writable format strings when fortified. Chapter 12 exploits this bug on purpose.

### `scanf` in brief

`scanf` returns the number of items successfully assigned, or `EOF`. Its `%s` and `%[` need a maximum width (`%63s` for a 64-byte buffer) or they overflow; `%d` overflow is undefined; bad input stays in the stream. Unit 1.9's rule stands: read lines with `fgets`, then parse with `sscanf` (checking its return value) or `strtol`.

### Deep & Tricky

**DT-60. `%f` in `printf` is `double`, but in `scanf` it is `float`.** *(Guarantee: default argument promotions)* Variadic arguments undergo promotion — `float` becomes `double`, `char` and `short` become `int` — so `printf` never receives a `float`, and `%f` and `%lf` mean the same thing there. `scanf` receives *pointers*, which are not promoted: `%f` writes a `float`, `%lf` a `double`. Passing `&d` for a `double d` with `%f` corrupts it.

### Mastery check

1. **Predict.** `printf("[%-6.2f|%06.1f|%.0f]\n", 3.14159, 2.5, 2.5);` (Note: `%.0f` rounds half to even under the default rounding mode.)
2. **Explain.** Why does `printf("%d\n", 3.0);` print garbage rather than 3?

## Topic 6B — Files

## Unit 6.3 — Reading and writing files

**One-line summary:** `fopen` opens a file in a mode (`"r"`, `"w"`, `"a"`, with `"+"` for update and `"b"` for binary); `fgets`/`fputs`/`fprintf` handle text, `fread`/`fwrite` handle blocks of bytes; `fseek`/`ftell` move around. Every I/O call can fail, end-of-file and error are different conditions, `fclose` can report a write error — and a file is safely replaced by writing a temporary file and `rename`-ing it.

```c
// file: files.c
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* write_lines writes n lines to path atomically: readers see either the old
   file or the complete new one, never a partial write. Returns 0 or -1. */
static int write_lines(const char *path, const char *const *lines, size_t n) {
    char tmp[256];
    if (snprintf(tmp, sizeof tmp, "%s.tmp", path) >= (int)sizeof tmp)
        return -1;
    FILE *f = fopen(tmp, "w");
    if (f == NULL)
        return -1;
    for (size_t i = 0; i < n; i++) {
        if (fprintf(f, "%s\n", lines[i]) < 0) {
            fclose(f);
            remove(tmp);
            return -1;
        }
    }
    if (fclose(f) != 0) {                  /* buffered data is written here: check it */
        remove(tmp);
        return -1;
    }
    return rename(tmp, path);              /* atomic replacement on POSIX file systems */
}

/* count_lines counts '\n' characters; returns -1 on a read error. */
static long count_lines(const char *path) {
    FILE *f = fopen(path, "r");
    if (f == NULL)
        return -1;
    long lines = 0;
    int c;
    while ((c = getc(f)) != EOF)
        if (c == '\n')
            lines++;
    long result = ferror(f) ? -1 : lines;  /* EOF can mean end or error: ask */
    fclose(f);
    return result;
}

int main(void) {
    const char *lines[] = {"alpha", "beta", "gamma"};
    if (write_lines("demo.txt", lines, 3) != 0) {
        perror("write_lines");
        return 1;
    }
    printf("lines: %ld\n", count_lines("demo.txt"));

    FILE *f = fopen("demo.txt", "a");      /* append */
    if (f == NULL)
        return 1;
    fputs("delta\n", f);
    fclose(f);

    f = fopen("demo.txt", "r");
    if (f == NULL)
        return 1;
    char line[64];
    while (fgets(line, sizeof line, f) != NULL) {
        line[strcspn(line, "\n")] = '\0';  /* strip the newline */
        printf("[%s]", line);
    }
    printf("\nfeof=%d ferror=%d\n", feof(f), ferror(f));

    fseek(f, 0, SEEK_END);
    printf("size via ftell: %ld bytes\n", ftell(f));
    fclose(f);

    if (fopen("/root/forbidden/x", "w") == NULL)
        printf("fopen error: %s\n", strerror(errno));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra files.c -o files && ./files
lines: 3
[alpha][beta][gamma][delta]
feof=1 ferror=0
size via ftell: 23 bytes
fopen error: No such file or directory
```

(The forbidden path failed with "No such file or directory" because the directory does not exist; for an existing directory without permission it would be "Permission denied". Always print `strerror(errno)` rather than guessing.)

### Modes

| Mode | Opens for | If the file exists | If not |
| --- | --- | --- | --- |
| `"r"` | reading | — | fail |
| `"w"` | writing | **truncated to zero** | created |
| `"a"` | appending (every write goes to the end) | kept | created |
| `"r+"` | reading and writing | kept | fail |
| `"w+"` | reading and writing | truncated | created |
| `"wx"` (C11) | writing, exclusive | **fail** | created |

`"b"` (binary) makes no difference on POSIX systems; on Windows, text mode translates `\r\n` to `\n` and treats byte 0x1A as end of file, so binary data must be opened with `"rb"`/`"wb"`. Write `"b"` for binary files in portable code.

### End of file versus error

`fgets` returning `NULL` and `getc` returning `EOF` mean "no more data — either end of file *or* an error". `feof` and `ferror` distinguish them after the fact. Loops of the form `while (!feof(f)) { fgets(...); ... }` are a classic bug: `feof` becomes true only *after* a read has failed, so the last line is processed twice. Loop on the read function's return value.

### Durability

`fclose` (or `fflush`) moves data from the stdio buffer into the kernel; the kernel writes it to disk later. A power failure can still lose it. For durability call `fsync(fileno(f))` before `fclose`, and for the `rename` trick to survive a crash, also `fsync` the directory. Databases (Chapter 13's capstone) depend on this.

### Deep & Tricky

**DT-61. `fclose` can fail, and that is where write errors appear.** *(Library behaviour)* Because output is buffered, a full disk or a network file-system error is often reported only when the buffer is flushed — by `fclose`. Ignoring `fclose`'s return value silently loses data. Check it for every file you write.

### Mastery check

1. **Explain.** Why is writing to a temporary file and renaming better than overwriting a file in place?
2. **Write.** `int copy_file(const char *src, const char *dst)` using `fread`/`fwrite` with a 64 KiB buffer and complete error handling, writing `dst` atomically.

## Unit 6.4 — Portable binary formats

**One-line summary:** Writing a struct with `fwrite(&s, sizeof s, 1, f)` produces a file that depends on the compiler's padding, the machine's endianness, and type sizes — and leaks padding bytes. Portable binary formats are defined byte by byte; code reads and writes them with explicit shifts, using fixed-width types.

```c
// file: binfmt.c
#include <stdint.h>
#include <stdio.h>
#include <string.h>

typedef struct {
    uint32_t id;
    int16_t temperature;     /* tenths of a degree */
    uint8_t flags;
} Reading;

/* Encoding: id as 4 bytes little-endian, temperature as 2 bytes little-endian
   two's complement, flags as 1 byte: 7 bytes total, no padding, any machine. */
enum { READING_SIZE = 7 };

static void put_u32le(unsigned char *p, uint32_t v) {
    p[0] = (unsigned char)v; p[1] = (unsigned char)(v >> 8);
    p[2] = (unsigned char)(v >> 16); p[3] = (unsigned char)(v >> 24);
}
static uint32_t get_u32le(const unsigned char *p) {
    return (uint32_t)p[0] | (uint32_t)p[1] << 8 | (uint32_t)p[2] << 16 | (uint32_t)p[3] << 24;
}
static void put_u16le(unsigned char *p, uint16_t v) {
    p[0] = (unsigned char)v; p[1] = (unsigned char)(v >> 8);
}
static uint16_t get_u16le(const unsigned char *p) {
    return (uint16_t)(p[0] | p[1] << 8);
}

static void encode(unsigned char out[READING_SIZE], const Reading *r) {
    put_u32le(out, r->id);
    put_u16le(out + 4, (uint16_t)r->temperature);   /* two's complement bits */
    out[6] = r->flags;
}

static Reading decode(const unsigned char in[READING_SIZE]) {
    uint16_t t = get_u16le(in + 4);
    return (Reading){
        .id = get_u32le(in),
        .temperature = (int16_t)t,                  /* well-defined in C23; two's complement on GCC */
        .flags = in[6],
    };
}

int main(void) {
    Reading r = {.id = 0x01020304, .temperature = -125, .flags = 0x81};
    unsigned char buf[READING_SIZE];
    encode(buf, &r);
    printf("sizeof(Reading) = %zu, encoded = %d bytes:", sizeof(Reading), READING_SIZE);
    for (int i = 0; i < READING_SIZE; i++)
        printf(" %02x", buf[i]);
    printf("\n");

    FILE *f = fopen("readings.bin", "wb");
    if (f == NULL || fwrite(buf, 1, sizeof buf, f) != sizeof buf || fclose(f) != 0)
        return 1;

    unsigned char in[READING_SIZE];
    f = fopen("readings.bin", "rb");
    if (f == NULL || fread(in, 1, sizeof in, f) != sizeof in)
        return 1;
    fclose(f);
    Reading back = decode(in);
    printf("decoded: id=0x%08x temperature=%d flags=0x%02x\n",
           (unsigned)back.id, back.temperature, back.flags);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra binfmt.c -o binfmt && ./binfmt && od -An -tx1 readings.bin
sizeof(Reading) = 8, encoded = 7 bytes: 04 03 02 01 83 ff 81
decoded: id=0x01020304 temperature=-125 flags=0x81
 04 03 02 01 83 ff 81
```

The struct occupies 8 bytes in memory (one byte of padding), but the format is 7 bytes, fully specified, and identical on big-endian machines, 32-bit machines, and any compiler. The shift-and-or code compiles to a single load on little-endian machines with optimisation, so it costs nothing. Network protocols use **big-endian** ("network byte order"): POSIX provides `htonl`/`ntohl` and glibc `htobe32`/`be32toh` (Chapter 10).

<!-- expanded -->
### Going deeper: records that can be checked

A file format that will be read back by another program — or by the same program after a crash — needs two more things than fixed byte order: a way to know where each record ends, and a way to know the record arrived intact. The common answer is a **length prefix** and a **checksum**: each record is written as `length (4 bytes, little-endian) | checksum (4 bytes) | payload`. The reader checks the length against a sane maximum before allocating anything, reads the payload, recomputes the checksum, and rejects the record if they differ. Chapter 13's write-ahead log uses exactly this layout.

```c
// file: records.c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_RECORD (1u << 20)

static uint32_t fnv1a(const unsigned char *p, size_t n) {     /* a simple 32-bit hash used as a checksum */
    uint32_t h = 2166136261u;
    while (n--) { h ^= *p++; h *= 16777619u; }
    return h;
}

static void put32(unsigned char *b, uint32_t v) { for (int i = 0; i < 4; i++) b[i] = (unsigned char)(v >> (8 * i)); }
static uint32_t get32(const unsigned char *b) { return b[0] | (uint32_t)b[1] << 8 | (uint32_t)b[2] << 16 | (uint32_t)b[3] << 24; }

static int write_record(FILE *f, const void *data, uint32_t len) {
    unsigned char hdr[8];
    put32(hdr, len);
    put32(hdr + 4, fnv1a(data, len));
    return fwrite(hdr, 1, 8, f) == 8 && fwrite(data, 1, len, f) == len ? 0 : -1;
}

/* returns 1 = record read, 0 = clean end of file, -1 = damaged or truncated */
static int read_record(FILE *f, unsigned char **out, uint32_t *len) {
    unsigned char hdr[8];
    size_t got = fread(hdr, 1, 8, f);
    if (got == 0) return 0;
    if (got != 8) return -1;
    *len = get32(hdr);
    if (*len > MAX_RECORD) return -1;                 /* never trust a length read from a file */
    *out = malloc(*len ? *len : 1);
    if (!*out || fread(*out, 1, *len, f) != *len) { free(*out); return -1; }
    if (fnv1a(*out, *len) != get32(hdr + 4)) { free(*out); return -1; }
    return 1;
}

static void dump(const char *path) {
    FILE *f = fopen(path, "rb");
    unsigned char *p;
    uint32_t n;
    int r;
    while ((r = read_record(f, &p, &n)) == 1) { printf("  record (%u bytes): %.*s\n", n, (int)n, p); free(p); }
    printf("  %s\n", r == 0 ? "clean end of file" : "stopped: damaged or truncated record");
    fclose(f);
}

int main(void) {
    const char *msgs[] = {"first record", "second, a little longer", "third"};
    FILE *f = fopen("log.bin", "wb");
    for (int i = 0; i < 3; i++) write_record(f, msgs[i], (uint32_t)strlen(msgs[i]));
    fclose(f);
    printf("intact file:\n");
    dump("log.bin");

    f = fopen("log.bin", "r+b");                      /* flip one byte inside the second payload */
    fseek(f, 8 + 12 + 8 + 3, SEEK_SET);
    fputc('X', f);
    fclose(f);
    printf("after corrupting one byte:\n");
    dump("log.bin");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address,undefined -g records.c -o records && ./records && od -A d -t x1 log.bin | head -2
intact file:
  record (12 bytes): first record
  record (23 bytes): second, a little longer
  record (5 bytes): third
  clean end of file
after corrupting one byte:
  record (12 bytes): first record
  stopped: damaged or truncated record
0000000 0c 00 00 00 02 65 f3 fb 66 69 72 73 74 20 72 65
0000016 63 6f 72 64 17 00 00 00 0d 3e 96 82 73 65 63 58
```

Reading stopped at the first damaged record instead of returning garbage. The length check matters as much as the checksum: without `MAX_RECORD`, a corrupted length of `0xFFFFFFFF` makes the reader try to allocate 4 GiB — the input decides how much memory the program uses, a classic denial-of-service bug (Chapter 12). A real format adds a magic number and a version field at the start of the file, and uses CRC-32 (or a stronger hash) rather than FNV-1a for error detection.

### Mastery check

1. **Explain.** Why is `(uint32_t)p[3] << 24` written with a cast, and what goes wrong without it if `p[3]` is 0x80? (Hint: promotion to `int`, DT-18.)
2. **Write.** Add a length-prefixed string field (2-byte length + bytes) to the format, with a decoder that rejects lengths longer than the remaining input.

## Topic 6C — The rest of the library

## Unit 6.5 — Time

**One-line summary:** `time()` gives calendar time as a `time_t` (seconds since 1970 UTC); `localtime_r`/`gmtime_r` break it into a `struct tm`; `strftime` formats it. For measuring durations use a **monotonic** clock — POSIX `clock_gettime(CLOCK_MONOTONIC)` or C11's `timespec_get` — never the wall clock, which can jump.

```c
// file: timing.c
#define _DEFAULT_SOURCE                    /* POSIX functions plus common extensions such as timegm */
#include <stdio.h>
#include <time.h>

static double elapsed(struct timespec a, struct timespec b) {
    return (double)(b.tv_sec - a.tv_sec) + (double)(b.tv_nsec - a.tv_nsec) / 1e9;
}

int main(void) {
    time_t now = 1767225600;                       /* fixed instant so the output is reproducible */
    struct tm utc;
    gmtime_r(&now, &utc);                          /* thread-safe; gmtime() uses a static buffer */
    char buf[64];
    strftime(buf, sizeof buf, "%Y-%m-%d %H:%M:%S UTC (%A, day %j)", &utc);
    printf("%lld -> %s\n", (long long)now, buf);
    strftime(buf, sizeof buf, "%Y-%m-%dT%H:%M:%SZ", &utc);   /* ISO 8601 */
    printf("ISO 8601: %s\n", buf);

    struct tm t = {.tm_year = 2024 - 1900, .tm_mon = 1, .tm_mday = 30, .tm_hour = 12};
    time_t normalised = timegm(&t);                /* Feb 30 normalises to Mar 1 */
    gmtime_r(&normalised, &t);
    strftime(buf, sizeof buf, "%Y-%m-%d", &t);
    printf("2024-02-30 normalises to %s\n", buf);

    struct timespec a, b;
    clock_gettime(CLOCK_MONOTONIC, &a);
    volatile double x = 0;
    for (int i = 0; i < 10000000; i++)
        x += i;
    clock_gettime(CLOCK_MONOTONIC, &b);
    printf("loop took %.1f ms (monotonic clock)\n", elapsed(a, b) * 1000);
    printf("CPU time used: %.1f ms (clock())\n", (double)clock() / CLOCKS_PER_SEC * 1000);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O1 timing.c -o timing && ./timing
1767225600 -> 2026-01-01 00:00:00 UTC (Thursday, day 001)
ISO 8601: 2026-01-01T00:00:00Z
2024-02-30 normalises to 2024-03-01
loop took 21.9 ms (monotonic clock)
CPU time used: 23.6 ms (clock())
```

Notes:

- A **feature-test macro** defined before any `#include` requests declarations beyond ISO C in strict `-std=c17` mode: `_POSIX_C_SOURCE 200809L` for POSIX (`gmtime_r`, `clock_gettime`), `_DEFAULT_SOURCE` for POSIX plus common BSD/GNU extensions such as `timegm`. (The first draft of this program used only `_POSIX_C_SOURCE`, and GCC reported an implicit declaration of `timegm` — an error in GCC 14. Reading such warnings is how you discover which standard a function belongs to.)
- `localtime` and `gmtime` return a pointer to shared static storage — not thread-safe, and the next call overwrites the result. Use the `_r` variants (C23 adds them to the standard as well).
- `mktime` interprets a `struct tm` as **local** time and normalises out-of-range fields — useful for date arithmetic ("add 30 days" by adding to `tm_mday`).
- `volatile` stops the compiler from deleting the loop as useless — benchmarking needs care (Chapter 11).
- 32-bit `time_t` overflows in January 2038; 64-bit Linux uses 64-bit `time_t`. Store timestamps in files as 64-bit integers.

<!-- expanded -->
### Going deeper: `mktime` normalises, and calendars are hard

`struct tm` stores the year minus 1900 (`tm_year`) and the month from 0 to 11 (`tm_mon`), while the day of the month runs from 1 to 31 — the source of countless off-by-one bugs. `mktime` converts a local `struct tm` to `time_t`, and as a side effect **normalises** out-of-range fields: day 32 of January becomes February 1, minute 90 becomes the next hour plus 30. That makes date arithmetic easy ("add 30 days" = add 30 to `tm_mday`, call `mktime`), and it makes invalid input silently valid.

```c
// file: dates.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

static void show(const char *label, struct tm tm) {
    char buf[64];
    time_t t = mktime(&tm);                          /* normalises tm in place */
    strftime(buf, sizeof buf, "%Y-%m-%d %H:%M %Z (%A)", &tm);
    printf("%-26s -> %s, time_t %lld\n", label, buf, (long long)t);
}

int main(void) {
    setenv("TZ", "Europe/Berlin", 1);                /* fixed zone: reproducible output */
    tzset();
    struct tm tm = { .tm_year = 2024 - 1900, .tm_mon = 1, .tm_mday = 30, .tm_hour = 12, .tm_isdst = -1 };
    show("2024-02-30 12:00", tm);                    /* there is no February 30 */

    struct tm base = { .tm_year = 2024 - 1900, .tm_mon = 0, .tm_mday = 31, .tm_hour = 9, .tm_isdst = -1 };
    base.tm_mon += 1;                                /* "one month after January 31" */
    show("2024-01-31 plus one month", base);

    struct tm dst = { .tm_year = 2024 - 1900, .tm_mon = 2, .tm_mday = 31, .tm_hour = 2, .tm_min = 30, .tm_isdst = -1 };
    show("2024-03-31 02:30 (DST gap)", dst);         /* clocks jumped from 02:00 to 03:00 */

    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra dates.c -o dates && ./dates
2024-02-30 12:00           -> 2024-03-01 12:00 CET (Friday), time_t 1709290800
2024-01-31 plus one month  -> 2024-03-02 09:00 CET (Saturday), time_t 1709366400
2024-03-31 02:30 (DST gap) -> 2024-03-31 03:30 CEST (Sunday), time_t 1711848600
```

"February 30" became March 1 without an error; "January 31 plus one month" became March 2 (there is no February 31, and 2024's February has 29 days); and 02:30 on the night the clocks went forward does not exist in Berlin, so `mktime` moved it to 03:30 CEST. Validate user input *before* `mktime` by checking that the normalised fields are unchanged.

### Deep & Tricky

**DT-62. Measure elapsed time with `CLOCK_MONOTONIC`, never with `time()`, `gettimeofday()` or `CLOCK_REALTIME`: the wall clock can jump backwards or forwards at any moment — NTP corrections, a manual change, a leap-second smear — so durations computed from it can be negative or wildly wrong.** *(POSIX `clock_gettime`: `CLOCK_MONOTONIC` "cannot be set" and represents time since an unspecified point; `CLOCK_REALTIME` "can be set")* Timeouts, rate limiters, and benchmarks all need the monotonic clock; only timestamps meant for humans or other machines need the real-time clock. On Linux, `CLOCK_MONOTONIC` stops while the system is suspended; `CLOCK_BOOTTIME` keeps counting.

### Mastery check

1. **Explain.** Why should a timeout or benchmark never use `time()` or `gettimeofday`?
2. **Write.** `int days_between(int y1, int m1, int d1, int y2, int m2, int d2)` using `timegm`.

## Unit 6.6 — `<stdlib.h>` and friends

**One-line summary:** `<stdlib.h>` provides memory allocation, number conversion (`strtol` family), sorting and searching (`qsort`, `bsearch`), environment access (`getenv`), process control (`exit`, `atexit`, `abort`), and a weak random generator (`rand`). Know which parts are safe to use and which to avoid.

```c
// file: stdlibdemo.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int cmp_int(const void *a, const void *b) {
    int x = *(const int *)a, y = *(const int *)b;
    return (x > y) - (x < y);
}

static void goodbye(void) { printf("atexit handler ran\n"); }

int main(void) {
    int v[] = {42, 7, 19, 3, 88, 21};
    size_t n = sizeof v / sizeof v[0];
    qsort(v, n, sizeof v[0], cmp_int);
    int key = 19;
    int *found = bsearch(&key, v, n, sizeof v[0], cmp_int);
    printf("sorted:");
    for (size_t i = 0; i < n; i++)
        printf(" %d", v[i]);
    printf("; 19 found at index %td\n", found ? found - v : -1);

    const char *home = getenv("HOME");
    printf("HOME is %s\n", home ? "set" : "not set");

    srand(1);
    printf("rand(): %d %d (RAND_MAX = %d)\n", rand(), rand(), RAND_MAX);

    div_t d = div(17, 5);
    printf("div(17, 5) = %d rem %d, abs(-3) = %d, llabs = %lld\n", d.quot, d.rem, abs(-3), llabs(-9LL));

    atexit(goodbye);
    printf("calling exit(3)\n");
    exit(3);
}
```

```console
$ gcc -std=c17 -Wall -Wextra stdlibdemo.c -o stdlibdemo && ./stdlibdemo; echo "exit $?"
sorted: 3 7 19 21 42 88; 19 found at index 2
HOME is set
rand(): 846930886 1804289383 (RAND_MAX = 2147483647)
div(17, 5) = 3 rem 2, abs(-3) = 3, llabs = 9
calling exit(3)
atexit handler ran
exit 3
```

What to use and what to avoid:

| Use | Avoid | Why |
| --- | --- | --- |
| `strtol`, `strtoul`, `strtod` | `atoi`, `atof` | no error detection; UB on overflow |
| `qsort` + safe comparator | hand-written sorts in production | `qsort` is well tested (Chapter 7 explains its algorithms) |
| `getenv` (read-only result) | modifying the returned string; `setenv` in threaded code | the environment is shared global state |
| `snprintf` | `sprintf`, `strcat`, `gets` (removed in C11) | no bounds |
| OS randomness (`getrandom`, `/dev/urandom`) for security; a good PRNG (PCG, xoshiro) for simulations | `rand` | small `RAND_MAX` on some platforms, poor quality, global state, not cryptographic |
| `exit` in `main`-like code | `exit` deep inside libraries | libraries must return errors, not end the caller's process |
| `system` only with constant strings, never with user data | `system(user_input)` | shell injection (Chapter 12) |

`abort` ends the process immediately with `SIGABRT` (no `atexit` handlers, no flushing) — what `assert` uses. `quick_exit`/`at_quick_exit` (C11) offer a middle path.

<!-- expanded -->
### Going deeper: `qsort` is not stable, and `rand` is not random enough

`qsort` promises a sorted result, not an order among equal elements. If records are sorted by one key, elements with the same key may come out in any order — so "sort by name, then sort by department" does not give "by department, then by name" as it would with a stable sort.

```c
// file: stability.c
#include <stdio.h>
#include <stdlib.h>

struct emp { int dept; int seq; };          /* seq: position in the input */

static int by_dept(const void *a, const void *b) {
    const struct emp *x = a, *y = b;
    return (x->dept > y->dept) - (x->dept < y->dept);
}

static int by_dept_then_seq(const void *a, const void *b) {   /* stable by construction: tie-break on input order */
    const struct emp *x = a, *y = b;
    int c = by_dept(a, b);
    return c ? c : (x->seq > y->seq) - (x->seq < y->seq);
}

/* how many neighbours with the same dept are out of input order? 0 means "stable here" */
static int disorder(const struct emp *e, int n) {
    int bad = 0;
    for (int i = 1; i < n; i++)
        if (e[i].dept == e[i - 1].dept && e[i].seq < e[i - 1].seq) bad++;
    return bad;
}

int main(void) {
    for (int n = 12; n <= 1200; n *= 10) {
        struct emp *e = malloc(n * sizeof *e), *f = malloc(n * sizeof *f);
        srand(7);
        for (int i = 0; i < n; i++) e[i] = f[i] = (struct emp){ rand() % 3, i };
        qsort(e, n, sizeof e[0], by_dept);
        qsort(f, n, sizeof f[0], by_dept_then_seq);
        printf("n = %4d: plain comparator %3d neighbours out of input order; with tie-break %d\n",
               n, disorder(e, n), disorder(f, n));
        free(e);
        free(f);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra stability.c -o stability && ./stability
n =   12: plain comparator   0 neighbours out of input order; with tie-break 0
n =  120: plain comparator   0 neighbours out of input order; with tie-break 0
n = 1200: plain comparator   0 neighbours out of input order; with tie-break 0
```

On this system the plain comparator kept equal elements in input order at every size: glibc 2.39's `qsort` uses a merge sort whenever it can allocate a temporary buffer, and merge sort is stable. That is a property of this library version, not of `qsort` — the C standard promises nothing, other C libraries (musl, the BSDs, Microsoft's) use algorithms that are not stable, and a 2023 attempt to switch glibc to a non-stable algorithm was reverted before release because existing programs had come to depend on the old behaviour. Code that needs stability must ask for it explicitly with a tie-breaking key, as the second comparator does; then the result is the same on every C library.

`rand()` is specified to return values from 0 to `RAND_MAX` (at least 32767 — glibc gives 2^31 − 1), with no quality requirements at all; `rand() % n` is also slightly biased when `RAND_MAX + 1` is not a multiple of `n`. It is fine for tests and games. For anything security-related — tokens, keys, nonces — use the operating system's generator: `getrandom()` on Linux, `arc4random()` on the BSDs and macOS (and glibc 2.36+).

### Mastery check

1. **Explain.** `bsearch` returns a pointer, not an index. How do you get the index, and what must be true about the array?

## Unit 6.7 — Modern C: C11, C17, and C23

**One-line summary:** C11 added threads, atomics, `_Generic`, `_Static_assert`, alignment control, anonymous structs and unions, and `_Noreturn`; C17 fixed defects; C23 modernised the language — `bool`/`true`/`false`/`nullptr` as keywords, `constexpr`, `typeof`, `auto`, `[[attributes]]`, binary literals and digit separators, `<stdckdint.h>` checked arithmetic, `<stdbit.h>` bit utilities, and `= {}` initialisers. GCC 14 supports almost all of C23 with `-std=c23`.

```c
// file: c23.c
#include <stdbit.h>
#include <stdckdint.h>
#include <stdio.h>

constexpr int N = 4;                       /* a true compile-time constant object */
enum small : unsigned char { A, B, C };    /* enum with a chosen underlying type */

[[nodiscard]] static int must_check(void) { return 7; }

int main(void) {
    int *p = nullptr;                      /* nullptr has its own type, nullptr_t */
    bool ok = true;                        /* keywords now: no <stdbool.h> */
    typeof(N) arr[N] = {};                 /* typeof; empty initialiser zeroes everything */
    auto mask = 0b1010'1010;               /* binary literal, digit separator, type inference */
    static_assert(sizeof(int) == 4);       /* message now optional */

    int sum;
    if (ckd_add(&sum, 2'000'000'000, 2'000'000'000))   /* checked arithmetic: no UB */
        printf("ckd_add overflowed; wrapped result %d\n", sum);
    unsigned long long big;
    bool ovf = ckd_mul(&big, 4'000'000'000ULL, 4ULL);
    printf("ckd_mul ok=%d result=%llu\n", !ovf, big);

    printf("nullptr=%d ok=%d sizeof arr=%zu mask=%d sizeof(enum small)=%zu\n",
           p == nullptr, ok, sizeof arr, mask, sizeof(enum small));
    printf("stdc_count_ones(0xF0F0u)=%u stdc_bit_width(1000u)=%u stdc_leading_zeros(1u)=%u\n",
           stdc_count_ones(0xF0F0u), stdc_bit_width(1000u), stdc_leading_zeros(1u));
    printf("%d\n", must_check());
    must_check();                          /* warning: result ignored */
    return 0;
}
```

```console
$ gcc-14 -std=c23 -Wall -Wextra c23.c -o c23 && ./c23
c23.c: In function ‘main’:
c23.c:30:5: warning: ignoring return value of ‘must_check’, declared with attribute ‘nodiscard’ [-Wunused-result]
   30 |     must_check();                          /* warning: result ignored */
      |     ^~~~~~~~~~~~
c23.c:9:26: note: declared here
    9 | [[nodiscard]] static int must_check(void) { return 7; }
      |                          ^~~~~~~~~~
ckd_add overflowed; wrapped result -294967296
ckd_mul ok=1 result=16000000000
nullptr=1 ok=1 sizeof arr=16 mask=170 sizeof(enum small)=1
stdc_count_ones(0xF0F0u)=8 stdc_bit_width(1000u)=10 stdc_leading_zeros(1u)=31
7
$ gcc -std=c17 c23.c -o c23_old 2>&1 | grep -m1 error
c23.c:3:10: fatal error: stdckdint.h: No such file or directory
```

`ckd_add`, `ckd_sub`, and `ckd_mul` compute the mathematically exact result, store it wrapped into the result type, and return whether it overflowed — the standard replacement for the overflow checks of Units 1.3 and 3.2. `<stdbit.h>` gives portable access to bit-counting instructions. GCC 13 (the book's default compiler) does not know C23's `constexpr`, which is why this unit uses `gcc-14`; check your compiler's support before relying on C23 in a project.

### What else C11 brought

- `_Generic` (Unit 1.2, Chapter 7), `_Static_assert`, `_Alignas`/`_Alignof` (Unit 3.5), anonymous structs/unions (Unit 3.6).
- `_Noreturn` (`[[noreturn]]` in C23) for functions such as `die()` that never return — better warnings and code generation.
- `<threads.h>` and `<stdatomic.h>` (Chapter 9).
- Bounds-checking "Annex K" functions (`strcpy_s`, …): optional, not implemented by glibc or musl; do not rely on them.

### Mastery check

1. **Write.** Rewrite Chapter 1's `safe_mul` with `ckd_mul`, compiled with `gcc-14 -std=c23`.

<!-- expanded -->
## Unit 6.8 — Text: bytes, UTF-8, and wide characters

**One-line summary:** A C string is a sequence of *bytes*; it knows nothing about characters. Modern text is almost always **UTF-8**, which encodes each Unicode code point in 1 to 4 bytes, ASCII unchanged. So `strlen` counts bytes, indexing `s[i]` reaches a byte that may be the middle of a character, and `toupper` works only on ASCII. To count, cut, or classify characters, decode UTF-8 (by hand or with the locale functions `mbrtowc`), and remember that `wchar_t` is 4 bytes on Linux and 2 on Windows.

UTF-8 has a simple, self-synchronising structure: the first byte says how long the sequence is, and every continuation byte starts with the bits `10`.

| Code points | Bytes | Bit pattern |
| --- | --- | --- |
| U+0000–U+007F | 1 | `0xxxxxxx` |
| U+0080–U+07FF | 2 | `110xxxxx 10xxxxxx` |
| U+0800–U+FFFF | 3 | `1110xxxx 10xxxxxx 10xxxxxx` |
| U+10000–U+10FFFF | 4 | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` |

```c
// file: utf8.c
#include <locale.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wchar.h>

/* Decode one UTF-8 sequence. Returns its length (1-4) and stores the code point,
   or returns 0 for an invalid sequence (overlong forms, surrogates, bad continuations). */
static size_t utf8_decode(const unsigned char *s, uint32_t *cp) {
    if (s[0] < 0x80) { *cp = s[0]; return 1; }
    size_t n = (s[0] & 0xE0) == 0xC0 ? 2 : (s[0] & 0xF0) == 0xE0 ? 3 : (s[0] & 0xF8) == 0xF0 ? 4 : 0;
    if (!n) return 0;
    uint32_t c = s[0] & (0x7F >> n);
    for (size_t i = 1; i < n; i++) {
        if ((s[i] & 0xC0) != 0x80) return 0;                     /* also stops at the '\0' terminator */
        c = c << 6 | (s[i] & 0x3F);
    }
    static const uint32_t min[] = {0, 0, 0x80, 0x800, 0x10000};
    if (c < min[n] || c > 0x10FFFF || (c >= 0xD800 && c <= 0xDFFF)) return 0;
    *cp = c;
    return n;
}

int main(void) {
    const char *text = "naïve café — 日本 🎉";
    printf("text: %s\n", text);
    printf("strlen (bytes): %zu\n", strlen(text));

    size_t chars = 0;
    for (const unsigned char *p = (const unsigned char *)text; *p; ) {
        uint32_t cp;
        size_t n = utf8_decode(p, &cp);
        if (!n) { printf("invalid UTF-8 at byte %td\n", p - (const unsigned char *)text); return 1; }
        if (cp > 0x7F) printf("  U+%04X uses %zu bytes\n", (unsigned)cp, n);
        chars++;
        p += n;
    }
    printf("code points: %zu\n", chars);

    printf("first 3 bytes as a C string: \"%.3s\" (cuts 'ï' in half)\n", text);

    setlocale(LC_ALL, "C.UTF-8");                   /* the locale tells mbstowcs which encoding to decode */
    wchar_t wide[64];
    size_t w = mbstowcs(wide, text, 64);
    printf("mbstowcs: %zu wide characters, sizeof(wchar_t) = %zu\n", w, sizeof(wchar_t));

    const unsigned char bad[] = {0xC0, 0xAF, 0};    /* an "overlong" encoding of '/' */
    uint32_t cp;
    printf("C0 AF decodes? %s\n", utf8_decode(bad, &cp) ? "yes" : "no (overlong form rejected)");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address,undefined -g utf8.c -o utf8 && ./utf8
text: naïve café — 日本 🎉
strlen (bytes): 28
  U+00EF uses 2 bytes
  U+00E9 uses 2 bytes
  U+2014 uses 3 bytes
  U+65E5 uses 3 bytes
  U+672C uses 3 bytes
  U+1F389 uses 4 bytes
code points: 17
first 3 bytes as a C string: "na�" (cuts 'ï' in half)
mbstowcs: 17 wide characters, sizeof(wchar_t) = 4
C0 AF decodes? no (overlong form rejected)
```

The string has 17 code points in 28 bytes: ASCII letters take one byte, `ï` and `é` two, `—` and the two Japanese characters three, and the emoji four. Printing with `%.3s` cut `ï` in half: the output ends with a lone lead byte, an invalid sequence that a terminal shows as a replacement character (or nothing). The decoder rejects the "overlong" two-byte form of `/`: accepting such forms once let attackers sneak `../` past path filters that checked only the one-byte form (the IIS directory traversal bugs of 2000–2001).

### Deep & Tricky

**DT-63. `wchar_t` is not "a Unicode character": its size and encoding are implementation-defined — 32-bit UTF-32 on Linux and macOS, 16-bit UTF-16 code units on Windows, where characters outside the Basic Multilingual Plane (such as most emoji) take two `wchar_t` values.** *(Guarantee: C17 §7.19 — `wchar_t` is "an integer type whose range of values can represent distinct codes for all members of the largest extended character set specified among the supported locales"; Microsoft documents `wchar_t` as UTF-16 on Windows)* Portable programs therefore keep text as UTF-8 in `char` arrays, convert only at the boundaries, and use C11's `char32_t` (`<uchar.h>`) when they need one fixed-size unit per code point. And even a code point is not a "character" as a user sees it: `é` can also be written as `e` followed by a combining accent (two code points), and many emoji are sequences of several code points — counting what a user perceives as characters needs a Unicode library (ICU, utf8proc).

### Mastery check

1. **Predict.** What does `toupper` do to each byte of the UTF-8 string `"é"` (bytes `C3 A9`) in the `"C"` locale?
2. **Write.** `size_t utf8_prefix(const char *s, size_t max_bytes)`: the largest number of bytes ≤ `max_bytes` that ends on a character boundary, so that truncating there never splits a character.

## Chapter 6 Project — `wc` and `grep` clones

**Goal:** implement two classic Unix tools with the standard library alone — handling files and standard input, options, errors, and exit statuses exactly like the originals — and verify the output against the real tools on the same inputs.

### Specification

- `mywc [-lwc] [FILE...]`: count lines (`\n` characters), words (maximal runs of non-whitespace), and bytes. With no options, print all three. With several files, print a `total` line. With no files (or `-`), read standard input. Output format: right-aligned columns as GNU `wc` prints them for files (width = digits of the largest count), followed by the file name.
- `mygrep [-inv] [-c] PATTERN [FILE...]`: print lines containing the fixed string `PATTERN` (`-i` case-insensitive, `-v` invert, `-n` line numbers, `-c` count only). With several files, prefix each line with `file:`. Exit status 0 if any line matched, 1 if none, 2 on errors. Lines of any length must work.

### Reference solution: `mywc`

```c
// file: tools/mywc.c
/* mywc: count lines, words, and bytes, like wc(1). */
#include <ctype.h>
#include <errno.h>
#include <stdbool.h>
#include <stdio.h>
#include <string.h>

typedef struct {
    unsigned long long lines, words, bytes;
} Counts;

static int count_stream(FILE *f, Counts *c) {
    unsigned char buf[65536];
    bool in_word = false;
    size_t n;
    while ((n = fread(buf, 1, sizeof buf, f)) > 0) {
        c->bytes += n;
        for (size_t i = 0; i < n; i++) {
            if (buf[i] == '\n')
                c->lines++;
            if (isspace(buf[i])) {                  /* buf is unsigned char: DT-34 safe */
                in_word = false;
            } else if (!in_word) {
                in_word = true;
                c->words++;
            }
        }
    }
    return ferror(f) ? -1 : 0;
}

static int digits(unsigned long long v) {
    int d = 1;
    while (v >= 10) {
        v /= 10;
        d++;
    }
    return d;
}

static void print_counts(const Counts *c, bool l, bool w, bool b, int width, const char *name) {
    const char *sep = "";
    if (l) { printf("%*llu", width, c->lines); sep = " "; }
    if (w) { printf("%s%*llu", sep, width, c->words); sep = " "; }
    if (b) { printf("%s%*llu", sep, width, c->bytes); }
    if (name != NULL)
        printf(" %s", name);
    printf("\n");
}

int main(int argc, char *argv[]) {
    bool l = false, w = false, b = false;
    int i = 1;
    for (; i < argc && argv[i][0] == '-' && argv[i][1] != '\0'; i++) {
        for (const char *o = argv[i] + 1; *o; o++) {
            switch (*o) {
            case 'l': l = true; break;
            case 'w': w = true; break;
            case 'c': b = true; break;
            default:
                fprintf(stderr, "mywc: invalid option -- '%c'\n", *o);
                return 2;
            }
        }
    }
    if (!l && !w && !b)
        l = w = b = true;

    int nfiles = argc - i, status = 0;
    if (nfiles == 0) {
        Counts c = {0};
        if (count_stream(stdin, &c) != 0) {
            perror("mywc: stdin");
            return 1;
        }
        unsigned long long m = c.lines > c.words ? c.lines : c.words;
        m = c.bytes > m ? c.bytes : m;
        int width = (l + w + b == 1) ? 1 : digits(m);
        print_counts(&c, l, w, b, width, NULL);
        return 0;
    }

    Counts all[nfiles > 0 ? nfiles : 1], total = {0};
    bool ok[nfiles > 0 ? nfiles : 1];
    for (int k = 0; k < nfiles; k++) {
        const char *name = argv[i + k];
        all[k] = (Counts){0};
        FILE *f = strcmp(name, "-") == 0 ? stdin : fopen(name, "rb");
        ok[k] = f != NULL && count_stream(f, &all[k]) == 0;
        if (!ok[k]) {
            fprintf(stderr, "mywc: %s: %s\n", name, strerror(errno));
            status = 1;
        }
        if (f != NULL && f != stdin)
            fclose(f);
        total.lines += all[k].lines;
        total.words += all[k].words;
        total.bytes += all[k].bytes;
    }
    unsigned long long biggest = (l ? total.lines : 0);
    if (w && total.words > biggest) biggest = total.words;
    if (b && total.bytes > biggest) biggest = total.bytes;
    int width = (nfiles == 1 && l + w + b == 1) ? 1 : digits(biggest);
    for (int k = 0; k < nfiles; k++)
        if (ok[k])
            print_counts(&all[k], l, w, b, width, argv[i + k]);
    if (nfiles > 1)
        print_counts(&total, l, w, b, width, "total");
    return status;
}
```

Two details mirror GNU `wc`: column width is the number of digits of the largest count (so columns line up), and a single count for a single file is printed without padding. The counts arrays are small VLAs sized by the number of command-line arguments — a bounded use of a VLA (Unit 2.3).

### Reference solution: `mygrep`

```c
// file: tools/mygrep.c
/* mygrep: print lines containing a fixed string, like grep -F. */
#define _POSIX_C_SOURCE 200809L          /* getline */
#include <ctype.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    bool ignore_case, invert, number, count;
} Options;

/* contains reports whether line contains pat, optionally ignoring ASCII case. */
static bool contains(const char *line, const char *pat, bool ignore_case) {
    if (!ignore_case)
        return strstr(line, pat) != NULL;
    size_t m = strlen(pat);
    for (const char *s = line; *s != '\0'; s++) {
        size_t k = 0;
        while (k < m && s[k] != '\0' &&
               tolower((unsigned char)s[k]) == tolower((unsigned char)pat[k]))
            k++;
        if (k == m)
            return true;
    }
    return m == 0;
}

/* grep_stream prints matching lines; returns the number of matches or -1. */
static long grep_stream(FILE *f, const char *name, const char *pat, const Options *o) {
    char *line = NULL;
    size_t cap = 0;
    ssize_t len;
    long lineno = 0, matches = 0;
    while ((len = getline(&line, &cap, f)) != -1) {   /* any line length */
        lineno++;
        if (len > 0 && line[len - 1] == '\n')
            line[len - 1] = '\0';
        if (contains(line, pat, o->ignore_case) == o->invert)
            continue;
        matches++;
        if (o->count)
            continue;
        if (name != NULL)
            printf("%s:", name);
        if (o->number)
            printf("%ld:", lineno);
        printf("%s\n", line);
    }
    bool err = ferror(f);
    free(line);
    return err ? -1 : matches;
}

int main(int argc, char *argv[]) {
    Options o = {0};
    int i = 1;
    for (; i < argc && argv[i][0] == '-' && argv[i][1] != '\0'; i++) {
        for (const char *p = argv[i] + 1; *p; p++) {
            switch (*p) {
            case 'i': o.ignore_case = true; break;
            case 'v': o.invert = true; break;
            case 'n': o.number = true; break;
            case 'c': o.count = true; break;
            default:
                fprintf(stderr, "usage: mygrep [-ivnc] PATTERN [FILE...]\n");
                return 2;
            }
        }
    }
    if (i >= argc) {
        fprintf(stderr, "usage: mygrep [-ivnc] PATTERN [FILE...]\n");
        return 2;
    }
    const char *pat = argv[i++];
    int nfiles = argc - i;
    bool any = false, error = false;
    if (nfiles == 0) {
        long m = grep_stream(stdin, NULL, pat, &o);
        if (m < 0)
            error = true;
        if (o.count && m >= 0)
            printf("%ld\n", m);
        any = m > 0;
    }
    for (int k = 0; k < nfiles; k++) {
        const char *name = argv[i + k];
        FILE *f = fopen(name, "r");
        if (f == NULL) {
            fprintf(stderr, "mygrep: ");
            perror(name);
            error = true;
            continue;
        }
        long m = grep_stream(f, nfiles > 1 ? name : NULL, pat, &o);
        fclose(f);
        if (m < 0) {
            error = true;
            continue;
        }
        if (o.count) {
            if (nfiles > 1)
                printf("%s:", name);
            printf("%ld\n", m);
        }
        if (m > 0)
            any = true;
    }
    if (error)
        return 2;
    return any ? 0 : 1;
}
```

`getline` (POSIX) allocates and grows the line buffer as needed, so lines of any length work — unlike `fgets` with a fixed buffer. Exit statuses follow `grep`: scripts write `if grep -q …` and depend on them.

### Testing against the real tools

```text
# file: tools/a.txt
The quick brown fox
jumps over the lazy dog.

  Leading spaces and	tabs
THE END
```

```text
# file: tools/b.txt
one two three
four
```

```console
$ cd tools && gcc -std=c17 -Wall -Wextra -pedantic -O2 mywc.c -o mywc && gcc -std=c17 -Wall -Wextra -pedantic -O2 mygrep.c -o mygrep
$ cd tools && ./mywc a.txt b.txt && wc a.txt b.txt
  6  18 100 a.txt
  3   7  39 b.txt
  9  25 139 total
  6  18 100 a.txt
  3   7  39 b.txt
  9  25 139 total
$ cd tools && ./mywc -l a.txt && wc -l a.txt && ./mywc < b.txt && wc < b.txt
6 a.txt
6 a.txt
 3  7 39
 3  7 39
$ cd tools && for args in "the a.txt" "-i the a.txt" "-in the a.txt b.txt" "-v -n o a.txt" "-c -i THE a.txt b.txt"; do diff <(./mygrep $args) <(grep -F $args) > /dev/null && echo "same output: $args" || echo "DIFFERENT: $args"; done
same output: the a.txt
same output: -i the a.txt
same output: -in the a.txt b.txt
same output: -v -n o a.txt
same output: -c -i THE a.txt b.txt
$ cd tools && ./mygrep -n -i the a.txt; ./mygrep zebra a.txt; echo "exit $?"; ./mygrep x nothere.txt; echo "exit $?"
2:The quick brown fox
3:jumps over the lazy dog.
6:THE END
exit 1
mygrep: nothere.txt: No such file or directory
exit 2
$ cd tools && head -c 3000000 /dev/zero | tr '\0' 'x' > long.txt && echo needle >> long.txt && ./mygrep -c x long.txt && ./mywc -c long.txt && wc -c long.txt
1
3000007 long.txt
3000007 long.txt
```

Every case matches the real tools, including a 3-megabyte line that a fixed `fgets` buffer would have split.

### Review checklist

- Do both tools read standard input when given no files, and handle unreadable files with a message on `stderr` and the right exit status?
- Are `isspace` and `tolower` always given `unsigned char` values?
- Does `mywc` count correctly when a word spans the boundary between two 64 KiB reads? (Test with a file whose word crosses byte 65536.)
- Are results compared automatically against the system tools, not by eye?

## Answers to the Chapter 6 prediction questions

**6.1 Q1.** The file contains (stderr's line 2 went to the terminal, not the file) "1. to stdout 3. stdout again" then "5. raw write(2)" then "4. before fork-like surprise": lines 1–3 were flushed by `fflush` before the `write`, but line 4 stayed in the fully buffered stdout until exit, after the raw `write` of line 5.

**6.2 Q1.** `[3.14  |0002.5|2]`: `%-6.2f` is left-aligned in 6 columns; `%06.1f` zero-pads to 6 characters; `%.0f` of 2.5 rounds half to even, giving 2.

**6.8 Q1.** In the `"C"` locale, `toupper` changes only the ASCII letters `a`–`z`; the bytes `0xC3` and `0xA9` are not lowercase letters there, so they are returned unchanged and the string stays `"é"`. In a single-byte Latin-1 locale, by contrast, `toupper(0xE9)` could return `0xC9` and corrupt a UTF-8 continuation byte — `toupper` is the wrong tool for UTF-8 text in every locale.

# Chapter 7 — Data Structures and Algorithms in C

*Goal: implement the core data structures and algorithms yourself, in C, with attention to memory layout, ownership, and measured performance — and learn the techniques C uses for generic code.*

Other languages ship hash maps, growable arrays, and priority queues in their standard libraries. C ships `qsort` and `bsearch`, and leaves the rest to you. That is why every serious C codebase contains its own containers — and why implementing them is part of learning C. This chapter covers complexity analysis, linked lists (including the intrusive lists used in the Linux kernel), stacks and queues, hash tables, trees, heaps, sorting, graphs, and the three ways to write generic code in C. Each structure is implemented, tested, and where it matters, measured.

**What you will be able to do after this chapter**

- Analyse time and space complexity, including amortised cost, and measure it.
- Implement singly and doubly linked lists, and use intrusive lists with `container_of`.
- Implement stacks, queues, and ring buffers.
- Implement a hash table with open addressing, resizing, and deletion.
- Implement binary search trees and binary heaps, and explain balanced trees.
- Implement and compare insertion sort, merge sort, and quicksort.
- Represent graphs and implement BFS, DFS, and Dijkstra's algorithm.
- Write generic containers using `void *`, macros, and `_Generic`.

## Topic 7A — Foundations

## Unit 7.1 — Complexity and measurement

**One-line summary:** Big-O notation describes how an algorithm's cost grows with input size n, ignoring constant factors: O(1) constant, O(log n), O(n), O(n log n), O(n²). Amortised analysis averages occasional expensive operations over many cheap ones. Constant factors and memory layout still matter in practice, so analysis is checked by measurement.

| Complexity | n = 1,000 | n = 1,000,000 | Typical operation |
| --- | --- | --- | --- |
| O(1) | 1 | 1 | array index, hash lookup (average) |
| O(log n) | 10 | 20 | binary search, balanced tree, heap push |
| O(n) | 1,000 | 1,000,000 | linear scan |
| O(n log n) | 10,000 | 20,000,000 | good sorting |
| O(n²) | 1,000,000 | 10¹² | nested loops over the input, insertion sort |

A measurement harness used throughout this chapter:

```c
// file: bench.h
/* Include this header FIRST: the feature-test macro below only works if it is
   defined before any system header is included (see the note below). */
#ifndef BENCH_H
#define BENCH_H
#define _POSIX_C_SOURCE 200809L
#include <time.h>

/* now_sec returns a monotonic timestamp in seconds. */
static inline double now_sec(void) {
    struct timespec ts;
    clock_gettime(CLOCK_MONOTONIC, &ts);
    return (double)ts.tv_sec + (double)ts.tv_nsec / 1e9;
}

#endif
```

```c
// file: growth.c
#include "bench.h"
#include <stdio.h>
#include <stdlib.h>

static long linear_search(const int *a, long n, int key) {
    for (long i = 0; i < n; i++)
        if (a[i] == key)
            return i;
    return -1;
}

static long binary_search(const int *a, long n, int key) {
    long lo = 0, hi = n;                     /* search in [lo, hi) */
    while (lo < hi) {
        long mid = lo + (hi - lo) / 2;       /* not (lo + hi) / 2: avoids overflow */
        if (a[mid] < key)
            lo = mid + 1;
        else
            hi = mid;
    }
    return (lo < n && a[lo] == key) ? lo : -1;
}

int main(void) {
    printf("%10s %16s %16s\n", "n", "linear (us)", "binary (us)");
    for (long n = 1000; n <= 10000000; n *= 10) {
        int *a = malloc((size_t)n * sizeof *a);
        if (a == NULL)
            return 1;
        for (long i = 0; i < n; i++)
            a[i] = (int)(2 * i);             /* sorted, even numbers */
        const int lookups = 200;
        volatile long sink = 0;
        double t0 = now_sec();
        for (int k = 0; k < lookups; k++)
            sink += linear_search(a, n, (int)(2 * (long)((k + 0.5) / lookups * n)));
        double t1 = now_sec();
        for (int k = 0; k < lookups; k++)
            sink += binary_search(a, n, (int)(2 * (long)((k + 0.5) / lookups * n)));
        double t2 = now_sec();
        printf("%10ld %16.3f %16.3f\n", n, (t1 - t0) / lookups * 1e6, (t2 - t1) / lookups * 1e6);
        free(a);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -O2 -Wall growth.c -o growth && ./growth
         n      linear (us)      binary (us)
      1000            0.141            0.046
     10000            1.265            0.064
    100000           14.125            0.078
   1000000          148.476            0.132
  10000000         1690.435            0.241
```

A note on `bench.h`: the first version of this program included it *after* `<stdio.h>`, and the build failed with "`CLOCK_MONOTONIC` undeclared". Feature-test macros such as `_POSIX_C_SOURCE` are read by glibc's internal `<features.h>` the first time *any* system header is included; defining one later has no effect. Define them at the very top of the `.c` file, on the compiler command line (`-D_POSIX_C_SOURCE=200809L`), or in a header that is always included first.

Each tenfold increase in n makes linear search about ten times slower; binary search barely changes — O(n) versus O(log n), visible directly. The `lo + (hi - lo) / 2` midpoint avoids a famous overflow bug: `(lo + hi) / 2` overflows when `lo + hi` exceeds the type's maximum (Java's `Arrays.binarySearch` had this bug for nine years).

### Amortised cost

A dynamic array that doubles its capacity when full (Chapter 3's `Vec`) occasionally copies all n elements, but over n pushes the total copying is at most 1 + 2 + 4 + … + n < 2n, so each push costs O(1) **amortised**. Growing by a constant amount instead (say +16) costs O(n) per push amortised — O(n²) for n pushes.

### Deep & Tricky

**DT-64. Big-O hides constants that can dominate.** *(Measurement fact)* A linear scan of a contiguous array of 50 integers is often faster than a hash lookup or a tree search: it touches one or two cache lines, the CPU prefetches them, and there are no pointer dereferences or hash computations. Choose by complexity for large n, and measure for small n.

### Mastery check

1. **Explain.** Why is growing a dynamic array by 1.5× still amortised O(1)? Why do some implementations prefer it to 2×?
2. **Predict.** If n doubles, roughly how much longer does an O(n log n) sort take?

## Topic 7B — Sequences

## Unit 7.2 — Linked lists

**One-line summary:** A linked list stores elements in separately allocated nodes connected by pointers: O(1) insertion and removal at a known position, O(n) access by index, and poor cache behaviour. A **pointer to a pointer** removes special cases in list manipulation, and **intrusive lists** — with the link embedded in the element and `container_of` to get back — are how the Linux kernel and many systems avoid separate node allocations.

### A singly linked list with the pointer-to-pointer technique

```c
// file: slist.c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int value;
    struct Node *next;
} Node;

/* push_front inserts at the head; returns 0 or -1. */
static int push_front(Node **head, int value) {
    Node *n = malloc(sizeof *n);
    if (n == NULL)
        return -1;
    n->value = value;
    n->next = *head;
    *head = n;
    return 0;
}

/* remove_value removes the first node with value, with no special case for the head:
   'link' points at whichever pointer currently refers to the node being examined. */
static int remove_value(Node **head, int value) {
    for (Node **link = head; *link != NULL; link = &(*link)->next) {
        if ((*link)->value == value) {
            Node *dead = *link;
            *link = dead->next;
            free(dead);
            return 0;
        }
    }
    return -1;
}

static void reverse(Node **head) {
    Node *prev = NULL, *cur = *head;
    while (cur != NULL) {
        Node *next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }
    *head = prev;
}

static void print_list(const Node *n) {
    for (; n != NULL; n = n->next)
        printf("%d -> ", n->value);
    printf("NULL\n");
}

static void free_list(Node *n) {
    while (n != NULL) {
        Node *next = n->next;          /* read next before freeing */
        free(n);
        n = next;
    }
}

int main(void) {
    Node *head = NULL;
    for (int i = 1; i <= 5; i++)
        if (push_front(&head, i) != 0)
            return 1;
    print_list(head);
    remove_value(&head, 5);            /* the head */
    remove_value(&head, 3);            /* the middle */
    remove_value(&head, 1);            /* the tail */
    print_list(head);
    reverse(&head);
    print_list(head);
    free_list(head);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -g -fsanitize=address slist.c -o slist && ./slist
5 -> 4 -> 3 -> 2 -> 1 -> NULL
4 -> 2 -> NULL
2 -> 4 -> NULL
```

`remove_value` needs no "is it the head?" branch because `link` always points at the pointer to update — either `head` itself or some node's `next` field. Linus Torvalds calls this having "good taste" in code; it is worth studying until it is obvious.

### Intrusive lists and `container_of`

```c
// file: intrusive.c
#include <stddef.h>
#include <stdio.h>

/* A doubly linked, circular list link, embedded in the objects themselves. */
struct list_head {
    struct list_head *next, *prev;
};

#define LIST_INIT(name) {&(name), &(name)}
#define container_of(ptr, type, member) ((type *)((char *)(ptr) - offsetof(type, member)))

static void list_add_tail(struct list_head *item, struct list_head *head) {
    item->prev = head->prev;
    item->next = head;
    head->prev->next = item;
    head->prev = item;
}

static void list_del(struct list_head *item) {
    item->prev->next = item->next;
    item->next->prev = item->prev;
    item->next = item->prev = NULL;
}

struct task {
    int id;
    const char *name;
    struct list_head run_link;      /* membership in the run queue */
    struct list_head all_link;      /* membership in the list of all tasks */
};

int main(void) {
    struct list_head run_queue = LIST_INIT(run_queue);
    struct list_head all_tasks = LIST_INIT(all_tasks);
    struct task t[3] = {{1, "init", {0}, {0}}, {2, "shell", {0}, {0}}, {3, "editor", {0}, {0}}};

    for (int i = 0; i < 3; i++) {
        list_add_tail(&t[i].all_link, &all_tasks);
        list_add_tail(&t[i].run_link, &run_queue);
    }
    list_del(&t[1].run_link);       /* shell sleeps: off the run queue, still in all_tasks */

    printf("runnable:");
    for (struct list_head *p = run_queue.next; p != &run_queue; p = p->next) {
        struct task *task = container_of(p, struct task, run_link);
        printf(" %d:%s", task->id, task->name);
    }
    printf("\nall:");
    for (struct list_head *p = all_tasks.next; p != &all_tasks; p = p->next)
        printf(" %s", container_of(p, struct task, all_link)->name);
    printf("\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra intrusive.c -o intrusive && ./intrusive
runnable: 1:init 3:editor
all: init shell editor
```

`container_of` subtracts the member's offset from the member's address to get the address of the enclosing struct. The benefits: no separate node allocations (the tasks here are not on the heap at all), an object can be in several lists at once, and removal is O(1) given just the object. The Linux kernel's `include/linux/list.h` is this design, used for tens of thousands of lists.

### When not to use linked lists

Each node is a separate allocation, so traversal jumps around memory and misses the cache on almost every step. For "a sequence I iterate over", a dynamic array is usually several times faster even when it must shift elements on insertion. Use lists when you need O(1) removal of arbitrary known elements, stable addresses for elements, or membership in multiple sequences.

<!-- expanded -->
### Going deeper: what a pointer chase costs

Big-O treats one step of a list traversal like one step of an array traversal. The hardware does not. An array is walked through consecutive addresses, which the CPU's prefetcher recognises and loads ahead of time; a list is walked by loading `next`, and the address of node *k+1* is unknown until node *k* has arrived from memory. If the nodes are scattered, almost every step waits for main memory — around 100 ns, a few hundred instructions' worth.

A fair benchmark must take care with one detail: nodes allocated one after another by `malloc` are often *adjacent in memory*, so a freshly built list behaves almost like an array. Real lists, built and modified over time, are not. The program below measures three cases: an array, a list whose nodes lie in allocation order, and the same list after its links have been shuffled.

```c
// file: chase.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

struct node { long value; struct node *next; };

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec / 1e9; }

int main(void) {
    enum { N = 4000000 };
    long *arr = malloc(N * sizeof *arr);
    struct node *nodes = malloc(N * sizeof *nodes);         /* one block: node i is at nodes + i */
    size_t *order = malloc(N * sizeof *order);
    for (size_t i = 0; i < N; i++) { arr[i] = (long)i; nodes[i].value = (long)i; order[i] = i; }

    for (size_t i = 0; i + 1 < N; i++) nodes[i].next = &nodes[i + 1];   /* links in memory order */
    nodes[N - 1].next = NULL;

    double t = now(); long s = 0;
    for (size_t i = 0; i < N; i++) s += arr[i];
    double t_arr = now() - t;

    t = now(); long s2 = 0;
    for (struct node *p = nodes; p; p = p->next) s2 += p->value;
    double t_seq = now() - t;

    srand(1);                                                /* shuffle the visiting order */
    for (size_t i = N - 1; i > 0; i--) { size_t j = (size_t)rand() % (i + 1); size_t x = order[i]; order[i] = order[j]; order[j] = x; }
    for (size_t i = 0; i + 1 < N; i++) nodes[order[i]].next = &nodes[order[i + 1]];
    nodes[order[N - 1]].next = NULL;

    t = now(); long s3 = 0;
    for (struct node *p = &nodes[order[0]]; p; p = p->next) s3 += p->value;
    double t_rand = now() - t;

    printf("sums agree: %s\n", s == s2 && s2 == s3 ? "yes" : "no");
    printf("array                 %6.1f ms  (%.2f ns per element)\n", t_arr * 1e3, t_arr * 1e9 / N);
    printf("list, nodes in order  %6.1f ms  (%.2f ns per node)\n", t_seq * 1e3, t_seq * 1e9 / N);
    printf("list, nodes shuffled  %6.1f ms  (%.2f ns per node)\n", t_rand * 1e3, t_rand * 1e9 / N);
    free(arr); free(nodes); free(order);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 chase.c -o chase && ./chase
sums agree: yes
array                    2.5 ms  (0.64 ns per element)
list, nodes in order    26.9 ms  (6.72 ns per node)
list, nodes shuffled   643.4 ms  (160.86 ns per node)
```

The same O(n) loop over the same 4 million values: the shuffled list was more than 200 times slower than the array, and the in-order list sat in between. (Exact numbers depend on the machine; the ratio is the lesson.)

### Deep & Tricky

**DT-65. A linked-list benchmark that builds its list with consecutive `malloc` calls and walks it right away measures a best case that real programs rarely see: the allocator places the nodes next to each other, so the "pointer chase" streams through memory like an array.** *(Implementation detail: glibc's allocator serves consecutive small requests from contiguous memory; hardware behaviour: prefetching works only for predictable access patterns)* After insertions, deletions and frees in between, nodes end up scattered, and each step becomes a cache miss. Benchmark data structures with access patterns and allocation histories like the real workload — or shuffle, as above — before concluding that a list "is fast enough".

### Mastery check

1. **Explain.** Why does `free_list` read `n->next` before `free(n)`?
2. **Write.** `void insert_sorted(Node **head, Node *n)` using the pointer-to-pointer technique, without special-casing an empty list or a new head.

## Unit 7.3 — Stacks, queues, and ring buffers

**One-line summary:** A **stack** (LIFO) is a dynamic array with push and pop at the end. A **queue** (FIFO) is best implemented as a **ring buffer**: a fixed array with head and tail indices that wrap around. With a power-of-two capacity, wrapping is a bit mask, and with free-running unsigned counters, full and empty are never ambiguous.

```c
// file: ring.c
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>

#define RING_CAP 8u                  /* must be a power of two */

typedef struct {
    int buf[RING_CAP];
    uint32_t head, tail;             /* free-running: never reset, wrap naturally (unsigned) */
} Ring;

static bool ring_push(Ring *r, int v) {
    if (r->tail - r->head == RING_CAP)
        return false;                /* full */
    r->buf[r->tail & (RING_CAP - 1)] = v;
    r->tail++;
    return true;
}

static bool ring_pop(Ring *r, int *out) {
    if (r->tail == r->head)
        return false;                /* empty */
    *out = r->buf[r->head & (RING_CAP - 1)];
    r->head++;
    return true;
}

int main(void) {
    Ring r = {.head = UINT32_MAX - 3, .tail = UINT32_MAX - 3};   /* start near wraparound on purpose */
    int pushed = 0, v;
    while (ring_push(&r, pushed))
        pushed++;
    printf("pushed %d before full\n", pushed);
    for (int i = 0; i < 3; i++) {
        ring_pop(&r, &v);
        printf("popped %d\n", v);
    }
    ring_push(&r, 100);
    ring_push(&r, 101);
    printf("size now %u (head=%u tail=%u)\n", r.tail - r.head, r.head, r.tail);
    while (ring_pop(&r, &v))
        printf("%d ", v);
    printf("\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=undefined ring.c -o ring && ./ring
pushed 8 before full
popped 0
popped 1
popped 2
size now 7 (head=4294967295 tail=6)
3 4 5 6 7 100 101 
```

The counters crossed `UINT32_MAX` and wrapped to small numbers during the test, yet `tail - head` stayed correct, because unsigned subtraction is modulo 2³² (Unit 0.5) — and UBSan found nothing, because unsigned wraparound is defined. This design (power-of-two size, free-running counters) is how the Linux kernel's `kfifo`, network card descriptor rings, and lock-free single-producer/single-consumer queues work (Chapter 9).

<!-- expanded -->
### Deep & Tricky

**DT-66. Free-running unsigned indices — `head` and `tail` that only ever increase and wrap at 2^32 — work only when the capacity divides 2^32, that is, when it is a power of two. With any other capacity, `index % cap` jumps when the counter wraps, and the ring silently reorders or overwrites elements.** *(Guarantee: unsigned arithmetic is modulo 2^N, C17 §6.2.5 ¶9; the arithmetic consequence follows from it)* The program below runs the same sequence of pushes and pops through a ring of capacity 3 and a ring of capacity 4, both starting just before the counters wrap.

```c
// file: ringwrap.c
#include <stdint.h>
#include <stdio.h>

static void run(uint32_t cap) {
    int buf[4] = {0};
    uint32_t head = UINT32_MAX - 2, tail = UINT32_MAX - 2;     /* close to wraparound */
    int next = 1;
    printf("capacity %u:", cap);
    buf[tail % cap] = next++; tail++;                          /* keep two elements in the ring */
    buf[tail % cap] = next++; tail++;
    for (int i = 0; i < 6; i++) {
        printf(" %d", buf[head % cap]); head++;                /* pop one ... */
        buf[tail % cap] = next++; tail++;                      /* ... push one */
    }
    printf("   (expected 1 2 3 4 5 6)\n");
}

int main(void) { run(3); run(4); return 0; }
```

```console
$ gcc -std=c17 -Wall -Wextra ringwrap.c -o ringwrap && ./ringwrap
capacity 3: 1 2 4 4 5 6   (expected 1 2 3 4 5 6)
capacity 4: 1 2 3 4 5 6   (expected 1 2 3 4 5 6)
```

With capacity 4 every value comes out in order. With capacity 3, `UINT32_MAX % 3` is 0 and, one step later, `0 % 3` is 0 too: when `tail` wraps, two consecutive pushes land in the same slot, value 3 is overwritten by value 4, and 4 comes out twice. Unit 7.3's ring requires a power-of-two capacity for this reason; a ring with an arbitrary capacity must wrap its indices explicitly (`if (++tail == cap) tail = 0;`) and keep a separate count to tell full from empty.

### Mastery check

1. **Explain.** With head/tail indices kept in `0..CAP-1` instead, why can't `head == tail` distinguish full from empty, and what are the two usual fixes?

## Topic 7C — Lookup structures

## Unit 7.4 — Hash tables

**One-line summary:** A hash table maps keys to values in O(1) average time by computing a **hash** of the key to choose a bucket. Collisions are resolved by **chaining** (a list per bucket) or **open addressing** (probe other slots); the table **resizes** when its load factor grows too high. A good hash function, a resize policy, and correct deletion are the three things to get right.

### Hash functions

A hash function must be fast and spread similar keys across buckets. For strings, **FNV-1a** is simple and decent:

```c
static uint64_t fnv1a(const char *s) {
    uint64_t h = 14695981039346656037u;         /* FNV offset basis */
    for (; *s; s++) {
        h ^= (unsigned char)*s;
        h *= 1099511628211u;                    /* FNV prime */
    }
    return h;
}
```

Production code uses faster or stronger hashes (xxHash, wyhash; SipHash when attackers choose the keys, to prevent **hash flooding** — inputs crafted so every key collides, turning O(1) into O(n), a known denial-of-service attack).

### Open addressing with linear probing

Store entries directly in an array. To insert, hash to a slot; if it is occupied by another key, try the next slot, and the next (linear probing), wrapping around. Lookups probe the same sequence until they find the key or an empty slot. Deletion cannot simply empty a slot — that would break the probe sequence for keys inserted after it — so a deleted slot becomes a **tombstone**: skipped by lookups, reusable by inserts. Keep the load factor (used slots ÷ capacity, counting tombstones) below about 0.7, resizing (rehashing everything into a table twice as large) when it is exceeded.

Linear probing is cache-friendly — consecutive slots share cache lines — which usually makes it faster than chaining in practice. The full implementation is this chapter's project.

### Deep & Tricky

**DT-67. Never iterate a hash table and rely on the order.** *(Design fact)* Iteration order depends on the hash function, capacity, and insertion history, and changes when the table resizes. Code that accidentally depends on it (tests comparing printed output, for instance) breaks when any of those change. Sort keys when order matters.

### Mastery check

1. **Explain.** Why must a lookup in an open-addressing table continue past a tombstone but stop at an empty slot?

## Unit 7.5 — Binary search trees and balanced trees

**One-line summary:** A **binary search tree** keeps keys ordered: everything in a node's left subtree is smaller, everything in the right is larger. Search, insertion, and deletion take O(height) — O(log n) if the tree is balanced, but O(n) if keys arrive in sorted order and the tree degenerates into a list. **Balanced trees** (AVL, red-black, B-trees) restructure themselves to keep the height logarithmic. Trees give ordered iteration and range queries, which hash tables cannot.

```c
// file: bst.c
#include <stdio.h>
#include <stdlib.h>

typedef struct Tree {
    int key;
    struct Tree *left, *right;
} Tree;

/* insert adds key (ignoring duplicates); returns 0 or -1. Uses a pointer to the
   link to update, as in the linked list (Unit 7.2). */
static int insert(Tree **t, int key) {
    while (*t != NULL) {
        if (key == (*t)->key)
            return 0;
        t = key < (*t)->key ? &(*t)->left : &(*t)->right;
    }
    *t = calloc(1, sizeof **t);
    if (*t == NULL)
        return -1;
    (*t)->key = key;
    return 0;
}

static const Tree *find(const Tree *t, int key) {
    while (t != NULL && t->key != key)
        t = key < t->key ? t->left : t->right;
    return t;
}

static int height(const Tree *t) {
    if (t == NULL)
        return 0;
    int l = height(t->left), r = height(t->right);
    return 1 + (l > r ? l : r);
}

static void in_order(const Tree *t, int lo, int hi) {   /* prints keys in [lo, hi], sorted */
    if (t == NULL)
        return;
    if (lo < t->key)
        in_order(t->left, lo, hi);
    if (lo <= t->key && t->key <= hi)
        printf("%d ", t->key);
    if (t->key < hi)
        in_order(t->right, lo, hi);
}

static void free_tree(Tree *t) {
    if (t == NULL)
        return;
    free_tree(t->left);
    free_tree(t->right);
    free(t);
}

int main(void) {
    Tree *random_tree = NULL, *sorted_tree = NULL;
    srand(42);
    for (int i = 0; i < 1023; i++) {
        insert(&random_tree, rand() % 100000);
        insert(&sorted_tree, i);                     /* worst case: sorted input */
    }
    printf("height with random keys: %d\n", height(random_tree));
    printf("height with sorted keys: %d (a linked list)\n", height(sorted_tree));
    printf("find(500) in sorted tree: %s\n", find(sorted_tree, 500) ? "found" : "missing");
    printf("range [10, 20]: ");
    in_order(sorted_tree, 10, 20);
    printf("\n");
    free_tree(random_tree);
    free_tree(sorted_tree);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 bst.c -o bst && ./bst
height with random keys: 20
height with sorted keys: 1023 (a linked list)
find(500) in sorted tree: found
range [10, 20]: 10 11 12 13 14 15 16 17 18 19 20 
```

1,023 random keys produce a height near 2·log₂ n (about 20); sorted keys produce height 1,023 — every operation becomes a linear scan, and the recursive `height` and `free_tree` would overflow the stack for a few hundred thousand nodes.

### Balanced trees in brief

- **AVL trees** keep the heights of every node's two subtrees within 1 of each other, restoring the property with **rotations** after inserts and deletes. Very balanced; slightly more work per update.
- **Red-black trees** colour nodes and enforce rules that keep the longest path at most twice the shortest. Fewer rotations than AVL; used in the Linux kernel (`lib/rbtree.c` — the CFS scheduler and memory-area tracking), C++'s `std::map`, and Java's `TreeMap`.
- **B-trees** store many keys per node (hundreds), matching disk pages or cache lines; the index structure of almost every database and file system (Chapter 13's capstone uses an append-only log and a hash index instead, with the trade-off discussed there).

Implementing a red-black tree is a classic exercise (Extension 2 of the project); in production C, use a well-tested implementation such as the kernel's or BSD's `<sys/tree.h>` macros.

<!-- expanded -->
### Going deeper: when a tree becomes a list

An unbalanced binary search tree is only as good as the order of insertion. Random keys give a height of about 2–3 times log₂ n; keys inserted in sorted order give a "tree" in which every node has only a right child — a linked list of height n, where every operation is O(n). Sorted input is not rare: IDs, timestamps, and the output of another sort all arrive in order.

```c
// file: degenerate.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

struct node { int key; struct node *left, *right; };

static struct node *insert(struct node *root, int key) {          /* iterative: no recursion depth problem */
    struct node **link = &root;
    while (*link) link = key < (*link)->key ? &(*link)->left : &(*link)->right;
    *link = calloc(1, sizeof **link);
    (*link)->key = key;
    return root;
}

static int height(const struct node *n) {                         /* iterative, with an explicit stack */
    int best = 0, top = 0;
    struct item { const struct node *n; int depth; } *stack = malloc(200000 * sizeof *stack);
    if (n) stack[top++] = (struct item){ n, 1 };
    while (top) {
        struct item it = stack[--top];
        if (it.depth > best) best = it.depth;
        if (it.n->left) stack[top++] = (struct item){ it.n->left, it.depth + 1 };
        if (it.n->right) stack[top++] = (struct item){ it.n->right, it.depth + 1 };
    }
    free(stack);
    return best;
}

static void destroy(struct node *n) {                              /* walk down the right spine iteratively */
    while (n) {
        if (n->left) { struct node *l = n->left; n->left = l->right; l->right = n; n = l; continue; }
        struct node *r = n->right;
        free(n);
        n = r;
    }
}

int main(void) {
    enum { N = 20000 };
    int *keys = malloc(N * sizeof *keys);
    for (int pass = 0; pass < 2; pass++) {
        for (int i = 0; i < N; i++) keys[i] = i;
        if (pass == 0) { srand(3); for (int i = N - 1; i > 0; i--) { int j = rand() % (i + 1), t = keys[i]; keys[i] = keys[j]; keys[j] = t; } }
        struct timespec a, b;
        clock_gettime(CLOCK_MONOTONIC, &a);
        struct node *root = NULL;
        for (int i = 0; i < N; i++) root = insert(root, keys[i]);
        clock_gettime(CLOCK_MONOTONIC, &b);
        printf("%-12s insertion: height %5d, built in %7.1f ms\n", pass ? "sorted" : "random",
               height(root), (b.tv_sec - a.tv_sec) * 1e3 + (b.tv_nsec - a.tv_nsec) / 1e6);
        destroy(root);
    }
    free(keys);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 degenerate.c -o degenerate && ./degenerate
random       insertion: height    36, built in     2.5 ms
sorted       insertion: height 20000, built in  3438.7 ms
```

The same 20,000 keys: a random order gives a tree a few dozen levels deep, sorted order a chain 20,000 levels deep that takes about a thousand times longer to build — and the gap grows quadratically with n (at 50,000 keys it took over 30 seconds). Note how the program avoids recursion everywhere — a recursive `insert` or `height` on the degenerate tree would recurse 20,000 levels deep, close to the stack limits of DT-25 for larger inputs. Self-balancing trees (red-black, AVL; Linux's `lib/rbtree.c`) or randomised structures (treaps, skip lists) keep the height logarithmic for every input order.

### Mastery check

1. **Explain.** Which operations does a balanced tree support efficiently that a hash table does not?

## Unit 7.6 — Heaps and priority queues

**One-line summary:** A **binary heap** is a complete binary tree stored in an array (children of index i at 2i+1 and 2i+2) in which every parent is ≤ its children. It gives O(1) access to the minimum and O(log n) push and pop — the standard **priority queue**, used for scheduling, event simulation, and Dijkstra's algorithm.

```c
// file: heap.c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int *a;
    size_t len, cap;
} Heap;

static void swap(int *x, int *y) { int t = *x; *x = *y; *y = t; }

static int heap_push(Heap *h, int v) {
    if (h->len == h->cap) {
        size_t cap = h->cap ? h->cap * 2 : 16;
        int *p = realloc(h->a, cap * sizeof *p);
        if (p == NULL)
            return -1;
        h->a = p;
        h->cap = cap;
    }
    size_t i = h->len++;
    h->a[i] = v;
    while (i > 0 && h->a[(i - 1) / 2] > h->a[i]) {       /* sift up */
        swap(&h->a[(i - 1) / 2], &h->a[i]);
        i = (i - 1) / 2;
    }
    return 0;
}

static int heap_pop(Heap *h, int *out) {
    if (h->len == 0)
        return -1;
    *out = h->a[0];
    h->a[0] = h->a[--h->len];
    size_t i = 0;
    for (;;) {                                            /* sift down */
        size_t l = 2 * i + 1, r = l + 1, m = i;
        if (l < h->len && h->a[l] < h->a[m]) m = l;
        if (r < h->len && h->a[r] < h->a[m]) m = r;
        if (m == i)
            break;
        swap(&h->a[i], &h->a[m]);
        i = m;
    }
    return 0;
}

int main(void) {
    Heap h = {0};
    int input[] = {42, 7, 19, 3, 88, 21, 3, 50};
    for (size_t i = 0; i < sizeof input / sizeof input[0]; i++)
        heap_push(&h, input[i]);
    printf("array layout:");
    for (size_t i = 0; i < h.len; i++)
        printf(" %d", h.a[i]);
    printf("\npopped in order:");
    int v;
    while (heap_pop(&h, &v) == 0)
        printf(" %d", v);
    printf("\n");
    free(h.a);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address,undefined heap.c -o heap && ./heap
array layout: 3 7 3 42 88 21 19 50
popped in order: 3 3 7 19 21 42 50 88
```

The array layout is not sorted — only the heap property holds — but popping repeatedly yields sorted order: that is **heapsort**, O(n log n) and in-place.

<!-- expanded -->
### Going deeper: building a heap in O(n)

Inserting n elements into a heap one at a time costs O(n log n). Turning an existing array into a heap costs only O(n), by **sifting down** from the last parent towards the root ("bottom-up heapify", due to Floyd): most nodes are near the bottom and sift down only a level or two. Counting comparisons makes the difference visible:

```c
// file: heapify.c
#include <stdio.h>
#include <stdlib.h>

static long comparisons;

static int less(int a, int b) { comparisons++; return a < b; }

static void sift_down(int *h, size_t n, size_t i) {                 /* min-heap */
    for (;;) {
        size_t l = 2 * i + 1, r = l + 1, m = i;
        if (l < n && less(h[l], h[m])) m = l;
        if (r < n && less(h[r], h[m])) m = r;
        if (m == i) return;
        int t = h[i]; h[i] = h[m]; h[m] = t;
        i = m;
    }
}

static void sift_up(int *h, size_t i) {
    while (i > 0 && less(h[i], h[(i - 1) / 2])) {
        int t = h[i]; h[i] = h[(i - 1) / 2]; h[(i - 1) / 2] = t;
        i = (i - 1) / 2;
    }
}

int main(void) {
    for (size_t n = 1000; n <= 1000000; n *= 10) {
        int *a = malloc(n * sizeof *a), *b = malloc(n * sizeof *b);
        srand(5);
        for (size_t i = 0; i < n; i++) a[i] = b[i] = rand();

        comparisons = 0;
        for (size_t i = 1; i < n; i++) sift_up(a, i);               /* n inserts */
        long by_insert = comparisons;

        comparisons = 0;
        for (size_t i = n / 2; i-- > 0; ) sift_down(b, n, i);       /* Floyd: bottom-up */
        long by_floyd = comparisons;

        printf("n = %7zu: inserts %9ld comparisons (%.2f per element), heapify %8ld (%.2f per element)\n",
               n, by_insert, (double)by_insert / n, by_floyd, (double)by_floyd / n);
        free(a); free(b);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 heapify.c -o heapify && ./heapify
n =    1000: inserts      2167 comparisons (2.17 per element), heapify     1830 (1.83 per element)
n =   10000: inserts     22691 comparisons (2.27 per element), heapify    18779 (1.88 per element)
n =  100000: inserts    227343 comparisons (2.27 per element), heapify   188010 (1.88 per element)
n = 1000000: inserts   2277477 comparisons (2.28 per element), heapify  1880232 (1.88 per element)
```

Bottom-up heapify needs a roughly constant number of comparisons per element — under two — whatever the size, which is what O(n) means; it is how `heapsort` and priority-queue constructors such as C++'s `std::make_heap` begin. For random input, repeated insertion is also cheap on average (a new random element rarely rises far); its O(n log n) worst case appears with adversarial input such as keys inserted in decreasing order into a min-heap.

### Mastery check

1. **Write.** Turn `Heap` into a priority queue of `(priority, value)` pairs, and use it to merge k sorted arrays in O(n log k).

## Topic 7D — Algorithms

## Unit 7.7 — Sorting

**One-line summary:** Insertion sort is O(n²) but fastest for small or nearly sorted arrays; merge sort is O(n log n) always and **stable** (equal keys keep their order) but needs O(n) extra memory; quicksort is O(n log n) on average, in place, and usually fastest, but O(n²) in its worst case unless pivots are chosen carefully. Production sorts combine them (introsort, Timsort, pdqsort).

```c
// file: sorts.c
#include "bench.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void insertion_sort(int *a, size_t n) {
    for (size_t i = 1; i < n; i++) {
        int x = a[i];
        size_t j = i;
        while (j > 0 && a[j - 1] > x) {
            a[j] = a[j - 1];
            j--;
        }
        a[j] = x;
    }
}

static void merge_sort_rec(int *a, int *tmp, size_t n) {
    if (n <= 16) {                         /* small subarrays: insertion sort is faster */
        insertion_sort(a, n);
        return;
    }
    size_t mid = n / 2;
    merge_sort_rec(a, tmp, mid);
    merge_sort_rec(a + mid, tmp, n - mid);
    size_t i = 0, j = mid, k = 0;
    while (i < mid && j < n)
        tmp[k++] = a[j] < a[i] ? a[j++] : a[i++];   /* '<' keeps equal elements in order: stable */
    while (i < mid)
        tmp[k++] = a[i++];
    while (j < n)
        tmp[k++] = a[j++];
    memcpy(a, tmp, n * sizeof *a);
}

static int merge_sort(int *a, size_t n) {
    int *tmp = malloc(n * sizeof *tmp);
    if (tmp == NULL)
        return -1;
    merge_sort_rec(a, tmp, n);
    free(tmp);
    return 0;
}

static void quick_sort(int *a, size_t n) {
    while (n > 16) {
        /* median of three as pivot: avoids O(n^2) on sorted input */
        size_t m = n / 2;
        int x = a[0], y = a[m], z = a[n - 1];
        int pivot = (x < y) ? (y < z ? y : (x < z ? z : x)) : (x < z ? x : (y < z ? z : y));
        size_t i = 0, j = n - 1;
        for (;;) {                           /* Hoare partition */
            while (a[i] < pivot) i++;
            while (a[j] > pivot) j--;
            if (i >= j)
                break;
            int t = a[i]; a[i] = a[j]; a[j] = t;
            i++; j--;
        }
        /* recurse into the smaller part, loop on the larger: O(log n) stack depth */
        if (j + 1 < n - j - 1) {
            quick_sort(a, j + 1);
            a += j + 1; n -= j + 1;
        } else {
            quick_sort(a + j + 1, n - j - 1);
            n = j + 1;
        }
    }
    insertion_sort(a, n);
}

static int cmp_int(const void *p, const void *q) {
    int x = *(const int *)p, y = *(const int *)q;
    return (x > y) - (x < y);
}

static int is_sorted(const int *a, size_t n) {
    for (size_t i = 1; i < n; i++)
        if (a[i - 1] > a[i])
            return 0;
    return 1;
}

int main(void) {
    enum { N = 2000000 };
    int *orig = malloc(N * sizeof *orig), *a = malloc(N * sizeof *a);
    if (orig == NULL || a == NULL)
        return 1;
    srand(7);
    for (size_t i = 0; i < N; i++)
        orig[i] = rand();

    const char *names[] = {"merge sort", "quicksort", "qsort (libc)"};
    for (int s = 0; s < 3; s++) {
        memcpy(a, orig, N * sizeof *a);
        double t0 = now_sec();
        if (s == 0) merge_sort(a, N);
        else if (s == 1) quick_sort(a, N);
        else qsort(a, N, sizeof *a, cmp_int);
        double t1 = now_sec();
        printf("%-13s %7.1f ms  sorted=%d\n", names[s], (t1 - t0) * 1000, is_sorted(a, N));
    }
    /* insertion sort: only on a small prefix, it is O(n^2) */
    memcpy(a, orig, 20000 * sizeof *a);
    double t0 = now_sec();
    insertion_sort(a, 20000);
    printf("%-13s %7.1f ms  (only n = 20,000)\n", "insertion", (now_sec() - t0) * 1000);

    for (size_t i = 0; i < N; i++)
        a[i] = (int)i;                       /* already sorted: quicksort's classic worst case */
    t0 = now_sec();
    quick_sort(a, N);
    printf("quicksort on sorted input: %.1f ms (median-of-three keeps it O(n log n))\n", (now_sec() - t0) * 1000);
    free(orig);
    free(a);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 sorts.c -o sorts && ./sorts
merge sort      149.5 ms  sorted=1
quicksort       126.1 ms  sorted=1
qsort (libc)    228.6 ms  sorted=1
insertion        37.0 ms  (only n = 20,000)
quicksort on sorted input: 13.7 ms (median-of-three keeps it O(n log n))
```

The hand-written sorts beat glibc's `qsort` here because `qsort` calls the comparison function through a pointer for every comparison and moves elements byte-by-byte-generically, while ours compare `int`s directly — the price of generic code through `void *` (Unit 7.9). Insertion sort needed a fifth of merge sort's whole time for just 1% of the data; because it is O(n²), sorting all 2,000,000 elements would take about 100² = 10,000 times longer — several minutes. (The algorithm behind `qsort` is not specified by the standard, and glibc has changed its implementation between versions; never rely on `qsort` being stable.)

### Mastery check

1. **Explain.** Why does recursing into the smaller partition and looping on the larger bound the stack depth to O(log n)?
2. **Write.** A stable sort of records by one field, and a test that shows `qsort` is not guaranteed stable.

## Unit 7.8 — Graphs

**One-line summary:** A graph is a set of vertices connected by edges. Store sparse graphs as **adjacency lists** (in C: arrays of edges per vertex, ideally in one flat array — the compressed sparse row format), and dense graphs as an adjacency matrix. **BFS** finds shortest paths by edge count; **DFS** explores depth-first (cycle detection, topological sort); **Dijkstra's algorithm** with a heap finds shortest weighted paths.

```c
// file: graph.c
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

/* Compressed sparse row: edges of vertex v are to[first[v] .. first[v+1]-1]. */
typedef struct {
    int n;
    int *first, *to, *w;
} Graph;

static Graph build(int n, int m, const int (*edges)[3]) {
    Graph g = {n, calloc((size_t)n + 1, sizeof(int)), malloc((size_t)m * sizeof(int)),
               malloc((size_t)m * sizeof(int))};
    for (int e = 0; e < m; e++)
        g.first[edges[e][0] + 1]++;                   /* count out-degrees */
    for (int v = 0; v < n; v++)
        g.first[v + 1] += g.first[v];                 /* prefix sums: start offsets */
    int *pos = malloc((size_t)n * sizeof(int));
    for (int v = 0; v < n; v++)
        pos[v] = g.first[v];
    for (int e = 0; e < m; e++) {
        int u = edges[e][0];
        g.to[pos[u]] = edges[e][1];
        g.w[pos[u]] = edges[e][2];
        pos[u]++;
    }
    free(pos);
    return g;
}

static void bfs(const Graph *g, int src, int *hops) {
    int *queue = malloc((size_t)g->n * sizeof(int)), head = 0, tail = 0;
    for (int v = 0; v < g->n; v++)
        hops[v] = -1;
    hops[src] = 0;
    queue[tail++] = src;
    while (head < tail) {
        int u = queue[head++];
        for (int e = g->first[u]; e < g->first[u + 1]; e++)
            if (hops[g->to[e]] < 0) {
                hops[g->to[e]] = hops[u] + 1;
                queue[tail++] = g->to[e];
            }
    }
    free(queue);
}

/* Dijkstra with a binary heap of (dist, vertex); stale entries are skipped. */
typedef struct { long d; int v; } Item;
static void push(Item *h, int *len, Item it) {
    int i = (*len)++;
    h[i] = it;
    while (i > 0 && h[(i - 1) / 2].d > h[i].d) {
        Item t = h[i]; h[i] = h[(i - 1) / 2]; h[(i - 1) / 2] = t;
        i = (i - 1) / 2;
    }
}
static Item pop(Item *h, int *len) {
    Item top = h[0];
    h[0] = h[--(*len)];
    for (int i = 0;;) {
        int l = 2 * i + 1, r = l + 1, m = i;
        if (l < *len && h[l].d < h[m].d) m = l;
        if (r < *len && h[r].d < h[m].d) m = r;
        if (m == i) break;
        Item t = h[i]; h[i] = h[m]; h[m] = t;
        i = m;
    }
    return top;
}

static void dijkstra(const Graph *g, int src, long *dist, int *prev) {
    int m = g->first[g->n];
    Item *heap = malloc(((size_t)m + 1) * sizeof *heap);
    int len = 0;
    for (int v = 0; v < g->n; v++) {
        dist[v] = LONG_MAX;
        prev[v] = -1;
    }
    dist[src] = 0;
    push(heap, &len, (Item){0, src});
    while (len > 0) {
        Item it = pop(heap, &len);
        if (it.d > dist[it.v])
            continue;                                  /* stale entry */
        for (int e = g->first[it.v]; e < g->first[it.v + 1]; e++) {
            long nd = it.d + g->w[e];
            if (nd < dist[g->to[e]]) {
                dist[g->to[e]] = nd;
                prev[g->to[e]] = it.v;
                push(heap, &len, (Item){nd, g->to[e]});
            }
        }
    }
    free(heap);
}

int main(void) {
    /* A small road network: from, to, minutes */
    static const int edges[][3] = {
        {0, 1, 7}, {0, 2, 9}, {0, 5, 14}, {1, 2, 10}, {1, 3, 15}, {2, 3, 11},
        {2, 5, 2}, {3, 4, 6}, {5, 4, 9}, {4, 6, 3},
    };
    const char *name[] = {"A", "B", "C", "D", "E", "F", "G"};
    int m = sizeof edges / sizeof edges[0];
    Graph g = build(7, m, edges);

    int hops[7];
    bfs(&g, 0, hops);
    long dist[7];
    int prev[7];
    dijkstra(&g, 0, dist, prev);
    for (int v = 0; v < 7; v++) {
        printf("%s: hops %d, minutes %2ld, path ", name[v], hops[v], dist[v]);
        int path[7], k = 0;
        for (int x = v; x != -1; x = prev[x])
            path[k++] = x;
        while (k > 0) {
            k--;                                       /* not inside the printf: Unit 1.6 */
            printf("%s%s", name[path[k]], k ? "->" : "\n");
        }
    }
    free(g.first); free(g.to); free(g.w);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address,undefined graph.c -o graph && ./graph
A: hops 0, minutes  0, path A
B: hops 1, minutes  7, path A->B
C: hops 1, minutes  9, path A->C
D: hops 2, minutes 20, path A->C->D
E: hops 2, minutes 20, path A->C->F->E
F: hops 1, minutes 11, path A->C->F
G: hops 3, minutes 23, path A->C->F->E->G
```

BFS and Dijkstra disagree, as they should: G is 3 hops from A by A→F→E→G, but the fastest route (23 minutes) takes 4 hops through C. The **compressed sparse row** layout (`first`, `to`, `w` as flat arrays) stores the whole graph in three allocations; iterating a vertex's edges is a contiguous scan — the same layout used by high-performance graph libraries and routing engines. (The heap allows duplicates and skips stale entries, a simpler alternative to a "decrease-key" operation, with the same O((V + E) log V) bound.)

### Mastery check

1. **Write.** A DFS-based topological sort that reports a cycle if one exists.

## Topic 7E — Generic code in C

## Unit 7.9 — Three ways to write generic containers

**One-line summary:** C has no templates or generics. Generic code is written by (1) **`void *` plus element size** — one compiled implementation, runtime overhead and no type checking (`qsort`); (2) **macros that generate type-specific code** — full speed and type safety, harder to debug; or (3) **`_Generic`** — compile-time dispatch to per-type functions behind one name. Choose by how hot the code is and how many types use it.

```c
// file: generic.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* (1) void * + size: one implementation for every type */
typedef struct {
    unsigned char *data;
    size_t len, cap, elem;
} AnyVec;

static int anyvec_push(AnyVec *v, const void *elem) {
    if (v->len == v->cap) {
        size_t cap = v->cap ? v->cap * 2 : 8;
        unsigned char *p = realloc(v->data, cap * v->elem);
        if (p == NULL)
            return -1;
        v->data = p;
        v->cap = cap;
    }
    memcpy(v->data + v->len * v->elem, elem, v->elem);
    v->len++;
    return 0;
}
static void *anyvec_at(const AnyVec *v, size_t i) { return v->data + i * v->elem; }

/* (2) macro code generation: a real type and real functions per element type */
#define VEC_DEFINE(Name, T)                                                   \
    typedef struct { T *data; size_t len, cap; } Name;                        \
    static inline int Name##_push(Name *v, T x) {                             \
        if (v->len == v->cap) {                                               \
            size_t cap = v->cap ? v->cap * 2 : 8;                             \
            T *p = realloc(v->data, cap * sizeof *p);                         \
            if (p == NULL) return -1;                                         \
            v->data = p; v->cap = cap;                                        \
        }                                                                     \
        v->data[v->len++] = x;                                                \
        return 0;                                                             \
    }                                                                         \
    static inline void Name##_free(Name *v) { free(v->data); *v = (Name){0}; }

VEC_DEFINE(IntVec, int)
VEC_DEFINE(DoubleVec, double)

typedef struct { double x, y; } Point;
VEC_DEFINE(PointVec, Point)

/* (3) _Generic: one name, dispatched on the argument type at compile time */
static void print_int(int x) { printf("int %d\n", x); }
static void print_double(double x) { printf("double %g\n", x); }
static void print_str(const char *s) { printf("string \"%s\"\n", s); }
#define print_any(x) _Generic((x), int: print_int, double: print_double, \
                              char *: print_str, const char *: print_str)(x)

int main(void) {
    AnyVec av = {.elem = sizeof(Point)};
    Point p = {1.5, 2.5};
    anyvec_push(&av, &p);
    Point *q = anyvec_at(&av, 0);              /* caller must know the real type */
    printf("AnyVec: (%g, %g)\n", q->x, q->y);
    free(av.data);

    IntVec iv = {0};
    PointVec pv = {0};
    for (int i = 0; i < 5; i++)
        IntVec_push(&iv, i * i);
    PointVec_push(&pv, (Point){3, 4});
    /* IntVec_push(&iv, "oops");  would not compile: wrong type */
    printf("IntVec: %d %d %d %d %d; PointVec: (%g, %g)\n",
           iv.data[0], iv.data[1], iv.data[2], iv.data[3], iv.data[4], pv.data[0].x, pv.data[0].y);
    IntVec_free(&iv);
    PointVec_free(&pv);

    print_any(42);
    print_any(2.5);
    print_any("hello");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address generic.c -o generic && ./generic
AnyVec: (1.5, 2.5)
IntVec: 0 1 4 9 16; PointVec: (3, 4)
int 42
double 2.5
string "hello"
```

| Technique | Type safety | Speed | Code size | Debuggability | Used by |
| --- | --- | --- | --- | --- | --- |
| `void *` + size | none (casts) | indirect calls, `memcpy` per element | one copy | good | `qsort`, `bsearch`, most C libraries |
| Macro generation | full | as fast as hand-written | one copy per type | harder (macro expansion) | `khash`, `stb_ds`, BSD `<sys/tree.h>` |
| `_Generic` | full | direct calls | per-type functions | good | `<tgmath.h>`, C23 `<stdbit.h>` |

### Pattern

**P-8. Generic containers: `void *` for cold code, generated code for hot code.** *(Practice)* Write containers once with `void *` and an element size when they are not performance-critical and serve many types; generate type-specific versions with macros (or use an established header-only library) for hot paths; reserve `_Generic` for presenting one convenient name over per-type functions. *Trade-off:* simplicity and one binary copy versus speed and type safety.

### Mastery check

1. **Explain.** Why does `qsort` take a comparison function instead of using `<`?

## Chapter 7 Project — A generic hash map and benchmark

**Goal:** implement a production-quality hash map from strings to `void *` values — open addressing with linear probing, tombstone deletion, automatic resizing, iteration — test it thoroughly against a simple reference, and benchmark it against linear search and a sorted array with binary search.

### Requirements

```c
typedef struct hmap HMap;
HMap *hmap_new(void);
void  hmap_free(HMap *m);                                  /* frees keys (copies), not values */
int   hmap_put(HMap *m, const char *key, void *value);     /* 0 ok, -1 out of memory */
void *hmap_get(const HMap *m, const char *key, int *found);
int   hmap_del(HMap *m, const char *key);                  /* 0 deleted, -1 absent */
size_t hmap_len(const HMap *m);
int   hmap_next(const HMap *m, size_t *iter, const char **key, void **value);  /* iteration */
```

Rules: the map copies keys (owns them) and stores value pointers without owning them; `NULL` is a valid value (hence the `found` flag); the load factor including tombstones stays at or below 0.7; deleting and reinserting keys repeatedly must not degrade the table.

### Reference solution

```c
// file: hmap/hmap.h
#ifndef HMAP_H
#define HMAP_H

#include <stddef.h>

typedef struct hmap HMap;

/* hmap_new creates an empty map, or returns NULL when out of memory. */
HMap *hmap_new(void);

/* hmap_free releases the map and its key copies. Values are not freed. */
void hmap_free(HMap *m);

/* hmap_put inserts or replaces key's value. The key is copied.
   Returns 0, or -1 when out of memory (the map is unchanged). */
int hmap_put(HMap *m, const char *key, void *value);

/* hmap_get returns key's value; *found (if not NULL) is set to 1 or 0. */
void *hmap_get(const HMap *m, const char *key, int *found);

/* hmap_del removes key. Returns 0, or -1 if it was absent. */
int hmap_del(HMap *m, const char *key);

/* hmap_len returns the number of keys. */
size_t hmap_len(const HMap *m);

/* hmap_next iterates: start with *iter = 0; returns 1 and sets *key and *value
   for each entry, then 0. The map must not be modified during iteration. */
int hmap_next(const HMap *m, size_t *iter, const char **key, void **value);

#endif
```

```c
// file: hmap/hmap.c
#include "hmap.h"

#include <stdint.h>
#include <stdlib.h>
#include <string.h>

static char tombstone_marker;
#define TOMBSTONE (&tombstone_marker)       /* a unique address that is never a real key */

typedef struct {
    char *key;                              /* NULL: empty; TOMBSTONE: deleted */
    void *value;
    uint64_t hash;                          /* cached: cheap rehash and fast mismatch checks */
} Slot;

struct hmap {
    Slot *slots;
    size_t cap;                             /* always a power of two */
    size_t len;                             /* live keys */
    size_t used;                            /* live keys + tombstones */
};

static uint64_t hash_str(const char *s) {   /* FNV-1a */
    uint64_t h = 14695981039346656037u;
    for (; *s; s++) {
        h ^= (unsigned char)*s;
        h *= 1099511628211u;
    }
    return h;
}

HMap *hmap_new(void) {
    HMap *m = malloc(sizeof *m);
    if (m == NULL)
        return NULL;
    m->cap = 16;
    m->len = m->used = 0;
    m->slots = calloc(m->cap, sizeof *m->slots);
    if (m->slots == NULL) {
        free(m);
        return NULL;
    }
    return m;
}

void hmap_free(HMap *m) {
    if (m == NULL)
        return;
    for (size_t i = 0; i < m->cap; i++)
        if (m->slots[i].key != NULL && m->slots[i].key != TOMBSTONE)
            free(m->slots[i].key);
    free(m->slots);
    free(m);
}

/* find_slot returns the index of key's slot, or (if absent) the index where it
   should be inserted: the first tombstone on its probe path, else the empty slot. */
static size_t find_slot(const HMap *m, const char *key, uint64_t h, int *present) {
    size_t mask = m->cap - 1, i = (size_t)h & mask, first_tomb = SIZE_MAX;
    for (;;) {
        Slot *s = &m->slots[i];
        if (s->key == NULL) {
            *present = 0;
            return first_tomb != SIZE_MAX ? first_tomb : i;
        }
        if (s->key == TOMBSTONE) {
            if (first_tomb == SIZE_MAX)
                first_tomb = i;
        } else if (s->hash == h && strcmp(s->key, key) == 0) {
            *present = 1;
            return i;
        }
        i = (i + 1) & mask;                 /* linear probing; terminates: load factor < 1 */
    }
}

/* resize rehashes every live key into a table of new_cap slots (dropping tombstones). */
static int resize(HMap *m, size_t new_cap) {
    Slot *old = m->slots;
    size_t old_cap = m->cap;
    Slot *slots = calloc(new_cap, sizeof *slots);
    if (slots == NULL)
        return -1;
    m->slots = slots;
    m->cap = new_cap;
    m->used = m->len;
    for (size_t i = 0; i < old_cap; i++) {
        if (old[i].key == NULL || old[i].key == TOMBSTONE)
            continue;
        size_t j = (size_t)old[i].hash & (new_cap - 1);
        while (slots[j].key != NULL)
            j = (j + 1) & (new_cap - 1);
        slots[j] = old[i];
    }
    free(old);
    return 0;
}

int hmap_put(HMap *m, const char *key, void *value) {
    uint64_t h = hash_str(key);
    int present;
    size_t i = find_slot(m, key, h, &present);
    if (present) {
        m->slots[i].value = value;
        return 0;
    }
    /* grow (or just clean out tombstones) before the load factor exceeds 0.7 */
    if ((m->used + 1) * 10 > m->cap * 7) {
        size_t new_cap = (m->len + 1) * 10 > m->cap * 5 ? m->cap * 2 : m->cap;
        if (resize(m, new_cap) != 0)
            return -1;
        i = find_slot(m, key, h, &present);
    }
    size_t n = strlen(key) + 1;
    char *copy = malloc(n);
    if (copy == NULL)
        return -1;
    memcpy(copy, key, n);
    if (m->slots[i].key == NULL)
        m->used++;                           /* reusing a tombstone does not raise 'used' */
    m->slots[i] = (Slot){copy, value, h};
    m->len++;
    return 0;
}

void *hmap_get(const HMap *m, const char *key, int *found) {
    int present;
    size_t i = find_slot(m, key, hash_str(key), &present);
    if (found != NULL)
        *found = present;
    return present ? m->slots[i].value : NULL;
}

int hmap_del(HMap *m, const char *key) {
    int present;
    size_t i = find_slot(m, key, hash_str(key), &present);
    if (!present)
        return -1;
    free(m->slots[i].key);
    m->slots[i].key = TOMBSTONE;
    m->slots[i].value = NULL;
    m->len--;
    return 0;
}

size_t hmap_len(const HMap *m) { return m->len; }

int hmap_next(const HMap *m, size_t *iter, const char **key, void **value) {
    for (; *iter < m->cap; (*iter)++) {
        Slot *s = &m->slots[*iter];
        if (s->key != NULL && s->key != TOMBSTONE) {
            *key = s->key;
            *value = s->value;
            (*iter)++;
            return 1;
        }
    }
    return 0;
}
```

Design notes:

- **Tombstone as a unique address.** `&tombstone_marker` can never equal a `malloc`'d key, so one pointer field encodes empty, deleted, and live.
- **Cached hashes** make resizing cheap (no rehashing of strings) and let probes skip most `strcmp` calls.
- **`used` counts tombstones**, so a workload of endless deletes and inserts triggers a same-size rehash that clears them, instead of filling the table with tombstones until probes become linear scans. (The resize policy doubles only if live keys exceed half the capacity.)
- **Insertion reuses the first tombstone** on the probe path, keeping probe sequences short.

### Tests: a differential test against a trivial reference

```c
// file: hmap/test_hmap.c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "hmap.h"

/* Reference model: a plain array indexed by key number. Slow, obviously correct. */
enum { KEYS = 2000 };
static int ref_present[KEYS];
static intptr_t ref_value[KEYS];

static void key_name(char *buf, size_t n, int k) { snprintf(buf, n, "key-%d", k); }

int main(void) {
    HMap *m = hmap_new();
    if (m == NULL)
        return 1;
    srand(12345);
    char key[32];
    long ops = 0;
    for (int round = 0; round < 200000; round++) {
        int k = rand() % KEYS, op = rand() % 3;
        key_name(key, sizeof key, k);
        if (op == 0) {                                   /* put */
            intptr_t v = rand();
            if (hmap_put(m, key, (void *)v) != 0) return 1;
            ref_present[k] = 1;
            ref_value[k] = v;
        } else if (op == 1) {                            /* del */
            int rc = hmap_del(m, key);
            if ((rc == 0) != ref_present[k]) { printf("del mismatch at %d\n", round); return 1; }
            ref_present[k] = 0;
        } else {                                         /* get */
            int found;
            void *v = hmap_get(m, key, &found);
            if (found != ref_present[k] || (found && (intptr_t)v != ref_value[k])) {
                printf("get mismatch at %d\n", round);
                return 1;
            }
        }
        ops++;
    }
    size_t expected = 0;
    for (int k = 0; k < KEYS; k++)
        expected += (size_t)ref_present[k];
    size_t iter = 0, seen = 0;
    const char *kk;
    void *vv;
    while (hmap_next(m, &iter, &kk, &vv)) {
        int k = atoi(kk + 4);
        if (!ref_present[k] || (intptr_t)vv != ref_value[k]) { printf("iteration mismatch\n"); return 1; }
        seen++;
    }
    int found;
    hmap_put(m, "null-value", NULL);
    hmap_get(m, "null-value", &found);
    printf("%ld random operations matched the reference; len=%zu expected=%zu iterated=%zu; NULL value found=%d\n",
           ops, hmap_len(m) - 1, expected, seen, found);
    hmap_free(m);
    return 0;
}
```

```console
$ cd hmap && gcc -std=c17 -Wall -Wextra -pedantic -g -O1 -fsanitize=address,undefined hmap.c test_hmap.c -o test_hmap && ./test_hmap
200000 random operations matched the reference; len=1004 expected=1004 iterated=1004; NULL value found=1
```

Two hundred thousand random puts, gets, and deletes on 2,000 keys — exercising collisions, tombstones, tombstone reuse, and resizes — all agreed with the reference model, under ASan and UBSan, with no leaks. A **differential test** against an obviously correct (if slow) model is the most effective way to test a data structure.

### Benchmark

```c
// file: hmap/bench_hmap.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "../bench.h"
#include "hmap.h"

static int cmp_str(const void *a, const void *b) {
    return strcmp(*(char *const *)a, *(char *const *)b);
}

int main(void) {
    enum { LOOKUPS = 100000 };
    printf("%9s %14s %14s %14s\n", "keys", "linear ns/op", "bsearch ns/op", "hmap ns/op");
    for (int n = 100; n <= 1000000; n *= 10) {
        char **keys = malloc((size_t)n * sizeof *keys), **sorted = malloc((size_t)n * sizeof *sorted);
        HMap *m = hmap_new();
        for (int i = 0; i < n; i++) {
            char buf[32];
            snprintf(buf, sizeof buf, "user:%d", (int)((i * 2654435761u) % 100000007u));
            keys[i] = strdup(buf);
            sorted[i] = keys[i];
            hmap_put(m, keys[i], keys[i]);
        }
        qsort(sorted, (size_t)n, sizeof *sorted, cmp_str);
        volatile size_t sink = 0;

        int linear_lookups = n <= 10000 ? LOOKUPS : 1000;     /* linear search is too slow at 1M */
        double t0 = now_sec();
        for (int q = 0; q < linear_lookups; q++) {
            const char *k = keys[(q * 7919) % n];
            for (int i = 0; i < n; i++)
                if (strcmp(keys[i], k) == 0) { sink += (size_t)i; break; }
        }
        double t1 = now_sec();
        for (int q = 0; q < LOOKUPS; q++) {
            const char *k = keys[(q * 7919) % n];
            sink += (size_t)bsearch(&k, sorted, (size_t)n, sizeof *sorted, cmp_str);
        }
        double t2 = now_sec();
        for (int q = 0; q < LOOKUPS; q++)
            sink += (size_t)hmap_get(m, keys[(q * 7919) % n], NULL);
        double t3 = now_sec();
        printf("%9d %14.1f %14.1f %14.1f\n", n, (t1 - t0) / linear_lookups * 1e9,
               (t2 - t1) / LOOKUPS * 1e9, (t3 - t2) / LOOKUPS * 1e9);
        for (int i = 0; i < n; i++)
            free(keys[i]);
        free(keys);
        free(sorted);
        hmap_free(m);
    }
    return 0;
}
```

```console
$ cd hmap && gcc -std=gnu17 -O2 -Wall hmap.c bench_hmap.c -o bench_hmap && ./bench_hmap
     keys   linear ns/op  bsearch ns/op     hmap ns/op
      100          114.3           15.8           11.6
     1000         1160.1           76.2           16.6
    10000        13217.0          230.0          102.0
   100000       187780.0          462.4           92.7
  1000000      2084605.2          751.0          133.4
```

(`strdup` is POSIX, hence `-std=gnu17` for the benchmark; C23 adds `strdup` to the standard.) The numbers tell the chapter's story: linear search grows in proportion to n and loses even at 100 keys, because every step is a `strcmp` call; binary search grows like log n; the hash map stays nearly flat until the table and the key strings outgrow the CPU caches (from about 100,000 keys, where each lookup starts to cost cache misses — Chapter 11 explains why each miss costs tens of nanoseconds).

### Review checklist

- Does the differential test cover put-after-delete of the same key (tombstone reuse) and enough operations to force several resizes?
- Can a sequence of operations make `find_slot` loop forever? (Why not: `used < cap` always, so an empty slot exists.)
- Does `hmap_put` leave the map unchanged when `malloc` or `resize` fails? (Inject a failing allocator to test it.)
- Are hash values cached and compared before calling `strcmp`?

### Extensions

1. Add `hmap_put` variants that take ownership of values with a destructor callback.
2. Replace FNV-1a with a seeded hash and show the effect of a hash-flooding input (keys chosen to collide) on lookup time.
3. Implement a red-black tree map with the same interface plus ordered iteration, and compare.

## Answers to the Chapter 7 prediction questions

**7.1 Q2.** A little more than twice as long: (2n log 2n) / (n log n) = 2 · (log n + 1) / log n, about 2.1× for n around a million.

# Chapter 8 — POSIX Systems Programming

*Goal: talk to the operating system directly — files through descriptors, processes, pipes, signals, memory mappings, and dynamic loading — the interface on which every Unix tool, server, and language runtime is built.*

The C standard library is portable but thin. Real systems programs on Linux, macOS, and the BSDs use **POSIX**: the standard interface to Unix-like operating systems, with system calls for files, processes, signals, and memory. This chapter teaches that interface with programs you can run and trace with `strace`, and builds a working Unix shell with pipelines and redirection.

**What you will be able to do after this chapter**

- Explain the difference between a library call and a system call, and trace system calls with `strace`.
- Use file descriptors with `open`, `read`, `write`, and `close`, handling short reads, short writes, and `EINTR`.
- Inspect files and directories with `stat` and `readdir`.
- Create processes with `fork`, run programs with `exec`, and collect exit statuses with `waitpid`.
- Connect processes with pipes and redirect their input and output with `dup2`.
- Handle signals safely with `sigaction`.
- Map files and shared memory with `mmap`, and load plugins with `dlopen`.

All programs in this chapter define `_POSIX_C_SOURCE 200809L` (or `_DEFAULT_SOURCE`/`_GNU_SOURCE` where noted) before any include, because strict `-std=c17` hides POSIX declarations (Unit 6.5).

## Topic 8A — Files the Unix way

## Unit 8.1 — System calls and file descriptors

**One-line summary:** A **system call** is a request to the kernel — the only way a process can do I/O, create processes, or get memory. Files, pipes, sockets, and terminals are all accessed through **file descriptors**: small integers indexing the process's table of open files (0 = stdin, 1 = stdout, 2 = stderr). `read` and `write` may transfer fewer bytes than asked, and may fail with `EINTR`; robust code loops.

```c
// file: fdio.c
#define _POSIX_C_SOURCE 200809L
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

/* write_all writes all n bytes, retrying after short writes and EINTR. */
static int write_all(int fd, const void *buf, size_t n) {
    const char *p = buf;
    while (n > 0) {
        ssize_t w = write(fd, p, n);
        if (w < 0) {
            if (errno == EINTR)
                continue;               /* interrupted by a signal before writing anything */
            return -1;
        }
        p += w;
        n -= (size_t)w;
    }
    return 0;
}

int main(void) {
    int fd = open("fd_demo.txt", O_WRONLY | O_CREAT | O_TRUNC | O_CLOEXEC, 0644);
    if (fd < 0) {
        perror("open");
        return 1;
    }
    const char *msg = "written with write(2)\n";
    if (write_all(fd, msg, strlen(msg)) != 0 || close(fd) != 0) {
        perror("write/close");
        return 1;
    }

    fd = open("fd_demo.txt", O_RDONLY | O_CLOEXEC);
    char buf[8];
    ssize_t r;
    printf("fd = %d; reading in chunks of %zu:\n", fd, sizeof buf);
    while ((r = read(fd, buf, sizeof buf)) > 0)        /* 0 means end of file */
        printf("  got %zd bytes: \"%.*s\"\n", r, (int)r, buf);
    if (r < 0)
        perror("read");
    close(fd);

    if (open("/no/such/file", O_RDONLY) < 0)
        printf("open failed: errno %d (%s)\n", errno, strerror(errno));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra fdio.c -o fdio && ./fdio
fd = 3; reading in chunks of 8:
  got 8 bytes: "written "
  got 8 bytes: "with wri"
  got 6 bytes: "te(2)
"
open failed: errno 2 (No such file or directory)
$ strace -e trace=openat,read,write,close ./fdio 2>&1 >/dev/null | grep -E 'fd_demo|no/such|read\(3' | grep -v ELF | head -n 6
openat(AT_FDCWD, "fd_demo.txt", O_WRONLY|O_CREAT|O_TRUNC|O_CLOEXEC, 0644) = 3
openat(AT_FDCWD, "fd_demo.txt", O_RDONLY|O_CLOEXEC) = 3
read(3, "written ", 8)                  = 8
read(3, "with wri", 8)                  = 8
read(3, "te(2)\n", 8)                   = 6
read(3, "", 8)                          = 0
```

`strace` shows the real conversation with the kernel: `open` is implemented by the `openat` system call, each `read(3, …)` returns at most 8 bytes, and the final read returns 0 (end of file). The first descriptor returned was 3, because 0, 1, and 2 were already open — POSIX guarantees `open` returns the **lowest** unused descriptor, which the shell's redirection trick relies on (Unit 8.4).

### `FILE *` versus file descriptors

`fopen`/`fread` (stdio) are a buffered library layer on top of `open`/`read`. stdio reduces system calls for small reads and writes; descriptors give full control (non-blocking I/O, `poll`, sockets, pipes). `fileno(f)` gets the descriptor of a stream; `fdopen(fd, mode)` wraps a descriptor in a stream.

### Open flags worth knowing

| Flag | Meaning |
| --- | --- |
| `O_RDONLY`, `O_WRONLY`, `O_RDWR` | access mode |
| `O_CREAT` (+ mode argument), `O_EXCL` | create; fail if it exists (lock files, safe temp files) |
| `O_TRUNC`, `O_APPEND` | truncate; every write goes to the end atomically (log files) |
| `O_CLOEXEC` | close automatically on `exec` (Unit 8.3) — set it by default |
| `O_NONBLOCK` | non-blocking I/O (Chapter 10) |
| `O_DIRECTORY`, `O_NOFOLLOW` | must be a directory; don't follow a final symlink |

The mode `0644` is filtered by the process's **umask** (typically 022), so files are created `rw-r--r--`.

### Deep & Tricky

**DT-68. `read` and `write` may transfer less than requested — and it is not an error.** *(POSIX)* On pipes, sockets, and terminals, `read` returns whatever is available; `write` to a pipe or socket may accept part of a large buffer; and any blocking call can be interrupted by a signal (`EINTR`). Code that assumes one `read` returns a whole message works with small files and fails with networks. Always loop (`write_all`, `read_exact`).

**DT-69. Descriptors leak into child programs unless marked close-on-exec.** *(POSIX)* By default, open descriptors survive `exec`, so a program you start inherits your open files, sockets, and pipe ends — keeping pipes from closing (a common cause of hung pipelines) and leaking privileges. Open everything with `O_CLOEXEC` (and `SOCK_CLOEXEC`, `pipe2(…, O_CLOEXEC)`) unless you intend the child to have it.

### Mastery check

1. **Explain.** Why does `strace` show `openat` instead of `open`?
2. **Write.** `ssize_t read_exact(int fd, void *buf, size_t n)` that returns n, or fewer only at end of file, or −1 on error.

## Unit 8.2 — Files and directories

**One-line summary:** `stat` returns a file's metadata (type, size, permissions, owner, timestamps, inode number); `opendir`/`readdir` list directories; links, permissions, and the distinction between a **name** (directory entry) and a **file** (inode) explain most file-system behaviour.

```c
// file: lsdemo.c
#define _DEFAULT_SOURCE
#include <dirent.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>

static void mode_string(mode_t m, char out[11]) {
    out[0] = S_ISDIR(m) ? 'd' : S_ISLNK(m) ? 'l' : S_ISFIFO(m) ? 'p' : '-';
    const char *rwx = "rwxrwxrwx";
    for (int i = 0; i < 9; i++)
        out[i + 1] = (m & (1u << (8 - i))) ? rwx[i] : '-';
    out[10] = '\0';
}

static int cmp_names(const void *a, const void *b) {
    return strcmp(*(char *const *)a, *(char *const *)b);
}

int main(int argc, char *argv[]) {
    const char *dir = argc > 1 ? argv[1] : ".";
    DIR *d = opendir(dir);
    if (d == NULL) {
        perror(dir);
        return 1;
    }
    char *names[256];
    size_t n = 0;
    struct dirent *e;
    while ((e = readdir(d)) != NULL && n < 256)
        if (e->d_name[0] != '.')
            names[n++] = strdup(e->d_name);
    closedir(d);
    qsort(names, n, sizeof names[0], cmp_names);   /* readdir order is arbitrary */

    for (size_t i = 0; i < n; i++) {
        char path[4096];
        snprintf(path, sizeof path, "%s/%s", dir, names[i]);
        struct stat st;
        if (lstat(path, &st) != 0) {                /* lstat: don't follow symlinks */
            perror(path);
            continue;
        }
        char mode[11];
        mode_string(st.st_mode, mode);
        printf("%s %2ju %8jd %s\n", mode, (uintmax_t)st.st_nlink, (intmax_t)st.st_size, names[i]);
        free(names[i]);
    }
    return 0;
}
```

```console
$ mkdir -p demo && cd demo && rm -f * && printf 'hello\n' > a.txt && ln -f a.txt hard.txt && ln -sf a.txt soft.txt && mkfifo pipe && chmod 600 a.txt && cd ..
$ gcc -std=c17 -Wall -Wextra lsdemo.c -o lsdemo && ./lsdemo demo
-rw-------  2        6 a.txt
-rw-------  2        6 hard.txt
prw-r--r--  1        0 pipe
lrwxrwxrwx  1        5 soft.txt
```

What the listing shows:

- `a.txt` and `hard.txt` are two **names for the same file** (same inode): a link count of 2, and the `chmod` on one changed "both". A file is deleted when its last name is removed *and* no process has it open — which is why a program can keep writing to a log file after it is deleted.
- `soft.txt` is a **symbolic link**: a small file containing the path `a.txt` (5 bytes). `stat` follows it; `lstat` does not.
- `pipe` is a FIFO, a named pipe in the file system.
- Directory order from `readdir` is arbitrary — sort if you print it.

### Deep & Tricky

**DT-70. Check-then-use on file names is a race (TOCTOU).** *(Security)* `if (access(path, W_OK) == 0) fd = open(path, …)` leaves a window in which another process can replace `path` with a symlink to a sensitive file. Do the operation and handle its failure (`open` with `O_CREAT | O_EXCL`, `O_NOFOLLOW`), or work relative to an open directory descriptor with `openat`. Chapter 12 returns to this.

### Mastery check

1. **Predict.** After `rm a.txt` in `demo`, what do `cat hard.txt` and `cat soft.txt` print?

## Topic 8B — Processes

## Unit 8.3 — `fork`, `exec`, and `wait`

**One-line summary:** `fork` creates a copy of the calling process: it returns twice — 0 in the child, the child's PID in the parent. `exec` replaces the current program with another (the PID stays). `waitpid` waits for a child and collects its **exit status**; children that have exited but not been waited for remain as **zombies**. Every shell command you type runs through these three calls.

```c
// file: spawn.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

/* run executes argv[0] with arguments, waits, and returns a shell-style status. */
static int run(char *const argv[]) {
    fflush(stdout);                          /* DT-59: don't duplicate buffered output */
    pid_t pid = fork();
    if (pid < 0) {
        perror("fork");
        return -1;
    }
    if (pid == 0) {                          /* child */
        execvp(argv[0], argv);               /* searches PATH; returns only on failure */
        perror(argv[0]);
        _exit(127);                          /* _exit: don't run the parent's atexit/flush */
    }
    int status;                              /* parent */
    if (waitpid(pid, &status, 0) < 0) {
        perror("waitpid");
        return -1;
    }
    if (WIFEXITED(status))
        return WEXITSTATUS(status);
    if (WIFSIGNALED(status))
        return 128 + WTERMSIG(status);
    return -1;
}

int main(void) {
    char *echo[] = {"echo", "hello from a child process", NULL};
    char *falsecmd[] = {"false", NULL};
    char *missing[] = {"no-such-command", NULL};
    char *killed[] = {"sh", "-c", "kill -TERM $$", NULL};
    printf("parent pid %d\n", (int)getpid());
    printf("echo  -> status %d\n", run(echo));
    printf("false -> status %d\n", run(falsecmd));
    printf("missing -> status %d\n", run(missing));
    printf("killed  -> status %d\n", run(killed));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra spawn.c -o spawn && ./spawn
parent pid 3589
hello from a child process
echo  -> status 0
false -> status 1
no-such-command: No such file or directory
missing -> status 127
killed  -> status 143
```

The details that matter:

- **`execvp`** takes a `NULL`-terminated argument vector and searches `PATH`; the `exec` family (`execl`, `execv`, `execve`, `execvp`, …) differ in how arguments and environment are passed. On success it never returns, so anything after it is the failure path.
- **The child calls `_exit`, not `exit`,** after a failed `exec`: `exit` would flush stdio buffers copied from the parent (printing them twice) and run the parent's `atexit` handlers.
- **Exit status encoding:** `WIFEXITED`/`WEXITSTATUS` for normal exit; `WIFSIGNALED`/`WTERMSIG` for death by signal. Shells report signal deaths as 128 + signal number (143 = 128 + `SIGTERM`), and use 127 for "command not found".
- **Zombies:** an exited child keeps a process-table entry until the parent waits. A long-running parent that never waits accumulates zombies; if the parent exits, `init` (PID 1) adopts and reaps them.
- `fork` copies the address space **copy-on-write**: pages are shared until one process writes, so forking a large process is fast. `posix_spawn` combines fork and exec more efficiently and is preferable when you don't need to run code in between.

<!-- expanded -->
### Going deeper: zombies, orphans, and how a child ended

When a process exits, the kernel keeps a small record of it — its PID and exit status — until the parent collects it with `wait` or `waitpid`. Until then the dead process is a **zombie** (`Z` in `ps`, shown as `<defunct>`). A parent that never waits accumulates zombies, and each one holds a PID; a server that forks per connection without reaping eventually cannot create processes at all. If the *parent* dies first, the child becomes an **orphan** and is adopted by PID 1 (or by a "subreaper" process), which reaps it.

The status that `waitpid` returns packs several facts into one `int`; the `W` macros take it apart:

```c
// file: reap.c
#define _DEFAULT_SOURCE                     /* POSIX plus WCOREDUMP, which is an XSI extension */
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/wait.h>
#include <unistd.h>

static void describe(pid_t pid, int status) {
    if (WIFEXITED(status)) printf("  child %d exited with status %d\n", (int)pid, WEXITSTATUS(status));
    else if (WIFSIGNALED(status)) printf("  child %d was killed by signal %d (%s)%s\n", (int)pid, WTERMSIG(status),
                                         strsignal(WTERMSIG(status)), WCOREDUMP(status) ? ", core dumped" : "");
}

int main(void) {
    pid_t a = fork();
    if (a == 0) _exit(7);                                     /* normal exit */
    pid_t b = fork();
    if (b == 0) { int *p = NULL; return *p; }                 /* crash: SIGSEGV */
    pid_t c = fork();
    if (c == 0) { pause(); _exit(0); }                        /* waits for a signal */

    sleep(1);
    char cmd[64];
    snprintf(cmd, sizeof cmd, "ps -o pid,stat,comm --ppid %d", (int)getpid());
    printf("before waiting (Z = zombie):\n");
    fflush(stdout);
    if (system(cmd) != 0) return 1;

    kill(c, SIGTERM);
    int status;
    pid_t pid;
    while ((pid = waitpid(-1, &status, 0)) > 0) describe(pid, status);
    printf("after waiting:\n");
    fflush(stdout);
    if (system(cmd) != 0) return 1;
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra reap.c -o reap && ./reap 2>&1 | sed -E 's/^ *[0-9]+ /  PID /; s/child [0-9]+/child PID/'
before waiting (Z = zombie):
  PID STAT COMMAND
  PID Z    reap
  PID Z    reap
  PID S    reap
  PID S    sh
  child PID exited with status 7
  child PID was killed by signal 11 (Segmentation fault)
  child PID was killed by signal 15 (Terminated)
after waiting:
  PID STAT COMMAND
  PID S    sh
```

Two children were already zombies while the third still slept; one `waitpid` loop collected all three and decoded each status: an exit code, a segmentation fault, and the `SIGTERM` the parent sent. After the loop, the only child left is the shell that `system` starts to run `ps` — and it, too, is reaped by `system` before it returns.

### Deep & Tricky

**DT-71. A child that has called `fork` must leave with `_exit`, not `exit` or `return from main`, when it does not `exec` — and after a failed `exec` too. `exit` runs the parent's `atexit` handlers and flushes the stdio buffers the child inherited, so output written by the parent before the `fork` appears twice, and cleanup meant to run once (deleting a temporary file, sending a "goodbye" message) runs in both processes.** *(POSIX: `_exit` "shall not call functions registered with atexit()" and does not flush stdio streams; `exit` does both)* DT-59 showed the duplicated output; the same applies to every library that registers `atexit` handlers. The idiom is `execvp(...); perror("exec"); _exit(127);` — 127 being the shell's convention for "command not found".

### Mastery check

1. **Predict.** How many times is "hi" printed by `fork(); fork(); printf("hi\n");`?
2. **Explain.** Why does the parent call `fflush(stdout)` before `fork`?

## Unit 8.4 — Pipes and redirection

**One-line summary:** `pipe` creates a one-way channel: two descriptors, one for reading and one for writing. `dup2(old, new)` makes descriptor `new` refer to the same open file as `old`. A shell implements `a | b` by creating a pipe, forking twice, `dup2`-ing the pipe ends onto stdout and stdin, closing every unused end, and `exec`-ing — and `> file` by opening the file and `dup2`-ing it onto descriptor 1.

```c
// file: pipeline.c
#define _POSIX_C_SOURCE 200809L
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

/* Runs: printf 'c\na\nb\n' | sort > sorted.txt */
int main(void) {
    int fds[2];
    if (pipe(fds) != 0) {
        perror("pipe");
        return 1;
    }
    pid_t writer = fork();
    if (writer == 0) {
        dup2(fds[1], STDOUT_FILENO);       /* stdout -> pipe's write end */
        close(fds[0]);
        close(fds[1]);                     /* the duplicate on fd 1 keeps the pipe open */
        execlp("printf", "printf", "c\\na\\nb\\n", (char *)NULL);
        _exit(127);
    }
    pid_t reader = fork();
    if (reader == 0) {
        dup2(fds[0], STDIN_FILENO);        /* stdin <- pipe's read end */
        close(fds[0]);
        close(fds[1]);                     /* essential: otherwise sort never sees end of file */
        int out = open("sorted.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
        if (out < 0)
            _exit(1);
        dup2(out, STDOUT_FILENO);          /* > sorted.txt */
        close(out);
        execlp("sort", "sort", (char *)NULL);
        _exit(127);
    }
    close(fds[0]);                         /* the parent uses neither end */
    close(fds[1]);
    int s1, s2;
    waitpid(writer, &s1, 0);
    waitpid(reader, &s2, 0);
    printf("writer exit %d, reader exit %d\n", WEXITSTATUS(s1), WEXITSTATUS(s2));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra pipeline.c -o pipeline && ./pipeline && cat sorted.txt
writer exit 0, reader exit 0
a
b
c
```

The rule that makes pipelines work: **a reader sees end of file only when every write end of the pipe is closed** — in every process. Forget one `close(fds[1])` (in the parent or in the reader itself) and `sort` waits forever for more input. That is DT-69's hang, and the reason to close unused ends immediately after `fork`.

Writing to a pipe whose read end is closed raises `SIGPIPE`, which by default kills the writer — that is how `yes | head -n 1` terminates. Servers ignore `SIGPIPE` and handle the `EPIPE` error instead (Chapter 10).

<!-- expanded -->
### Deep & Tricky

**DT-72. A reader on a pipe sees end-of-file only when *every* write descriptor for that pipe is closed — including the copies inherited through `fork` by processes that never write. Forget to close one, and the reader blocks forever.** *(POSIX `read`: "If no process has the pipe open for writing, read() shall return 0 to indicate end-of-file")* The classic case: the parent creates a pipe, forks a child that writes and exits, then reads until EOF — but the parent itself still holds the write end. The demonstration below uses `alarm` to end the hang after two seconds.

```c
// file: pipehang.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <string.h>
#include <sys/wait.h>
#include <unistd.h>

static void run(int close_write_end) {
    int fd[2];
    if (pipe(fd) < 0) return;
    if (fork() == 0) {                         /* child: write a line and exit */
        close(fd[0]);
        const char *msg = "hello through the pipe\n";
        if (write(fd[1], msg, strlen(msg)) < 0) _exit(1);
        _exit(0);
    }
    if (close_write_end) close(fd[1]);         /* the parent never writes: close its copy */
    char buf[64];
    ssize_t n, total = 0;
    alarm(2);                                  /* SIGALRM ends the process if read blocks too long */
    while ((n = read(fd[0], buf, sizeof buf)) > 0) total += n;
    alarm(0);
    printf("  read %zd bytes, then end of file\n", total);
    close(fd[0]);
    wait(NULL);
}

int main(void) {
    printf("parent closes its write end:\n");
    fflush(stdout);
    run(1);
    printf("parent forgets to close it:\n");
    fflush(stdout);
    run(0);
    printf("never reached\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra pipehang.c -o pipehang && ./pipehang; echo "exit status $?"
parent closes its write end:
  read 23 bytes, then end of file
parent forgets to close it:
bash: line 1:  3629 Alarm clock             ./pipehang
exit status 142
```

The second run read the child's 23 bytes and then waited for more, because a writer — the parent itself — still existed; `SIGALRM` killed it after two seconds (status 142 = 128 + 14). In a pipeline of several processes, every process must close every pipe end it does not use, which is why the shell in this chapter's project closes descriptors so carefully after each `dup2`.

### Mastery check

1. **Explain.** What happens if the parent forgets `close(fds[1])`?
2. **Write.** `2>&1` redirection: run a command with both stdout and stderr going into the same file.

## Topic 8C — Signals, memory mapping, and loading code

## Unit 8.5 — Signals

**One-line summary:** A **signal** is an asynchronous notification to a process — `SIGINT` (Ctrl-C), `SIGTERM` (polite kill), `SIGCHLD` (child exited), `SIGSEGV` (invalid memory access), `SIGPIPE`, `SIGALRM`. Install handlers with `sigaction`, and do almost nothing inside them: a handler may call only **async-signal-safe** functions, so the standard pattern is to set a `volatile sig_atomic_t` flag (or write to a pipe) and handle the event in the main loop.

```c
// file: signals.c
#define _POSIX_C_SOURCE 200809L
#include <errno.h>
#include <signal.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

static volatile sig_atomic_t got_term = 0, alarms = 0;

static void on_term(int sig) {
    (void)sig;
    got_term = 1;                     /* only set a flag: async-signal-safe */
}

static void on_alarm(int sig) {
    (void)sig;
    alarms++;
    const char msg[] = "  [handler] tick\n";
    write(STDOUT_FILENO, msg, sizeof msg - 1);   /* write() is safe; printf is NOT */
}

int main(void) {
    struct sigaction sa;
    memset(&sa, 0, sizeof sa);
    sigemptyset(&sa.sa_mask);
    sa.sa_handler = on_term;
    sigaction(SIGTERM, &sa, NULL);
    sa.sa_handler = on_alarm;
    sa.sa_flags = SA_RESTART;          /* restart interrupted system calls where possible */
    sigaction(SIGALRM, &sa, NULL);
    signal(SIGPIPE, SIG_IGN);          /* ignore: get EPIPE from write instead */

    setvbuf(stdout, NULL, _IONBF, 0);  /* unbuffered, so output order is obvious */
    printf("pid %d: working; SIGTERM will arrive after 3 alarms\n", (int)getpid());
    while (!got_term) {
        alarm(1);
        pause();                       /* sleep until any signal arrives */
        if (alarms == 3) {
            printf("main loop: sending myself SIGTERM\n");
            kill(getpid(), SIGTERM);
        }
    }
    printf("main loop: got SIGTERM after %d alarms; cleaning up and exiting\n", (int)alarms);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra signals.c -o signals && ./signals | sed 's/pid [0-9]*/pid N/'
pid N: working; SIGTERM will arrive after 3 alarms
  [handler] tick
  [handler] tick
  [handler] tick
main loop: sending myself SIGTERM
main loop: got SIGTERM after 3 alarms; cleaning up and exiting
```

### Why handlers must be minimal

A signal can arrive between any two instructions — including in the middle of `malloc` or `printf`, while their internal locks are held or their data structures are half-updated. If the handler calls `printf` or `malloc` too, the program can deadlock or corrupt memory. POSIX lists the async-signal-safe functions (`write`, `_exit`, `kill`, `sigaction`, …); `printf`, `malloc`, and `exit` are not among them. A `volatile sig_atomic_t` flag is the only kind of variable the C standard guarantees a handler may safely write.

Modern Linux alternatives move signal handling out of the handler entirely: `signalfd` turns signals into readable descriptors for an event loop (Chapter 10), and the **self-pipe trick** has the handler write one byte into a pipe that the main loop polls.

### Deep & Tricky

**DT-73. `signal()` semantics vary; use `sigaction`.** *(Portability)* Historically, `signal` reset the handler to the default after one delivery on some systems, and whether interrupted system calls are restarted differs. `sigaction` specifies everything explicitly (`sa_mask` blocks other signals during the handler, `SA_RESTART` restarts interrupted calls, `SA_SIGINFO` gives sender details). `signal(sig, SIG_IGN)` is fine for ignoring.

### Mastery check

1. **Explain.** Why is `volatile sig_atomic_t` needed, and why is `volatile int` not enough in theory?
2. **Write.** A program that reaps child processes with a `SIGCHLD` handler that calls `waitpid(-1, &s, WNOHANG)` in a loop, saving `errno` on entry and restoring it on exit.

## Unit 8.6 — Memory mapping

**One-line summary:** `mmap` maps a file (or anonymous memory) into the address space, so file contents can be accessed like an array without `read` calls; pages are loaded on demand by the kernel. `MAP_SHARED` mappings are shared between processes and written back to the file; `MAP_PRIVATE` mappings are copy-on-write. `malloc` itself uses anonymous `mmap` for large blocks.

```c
// file: mmapdemo.c
#define _DEFAULT_SOURCE
#include <fcntl.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <sys/wait.h>
#include <unistd.h>

int main(void) {
    /* 1. read a file through a mapping: count newlines without read() */
    int fd = open("/etc/services", O_RDONLY);
    struct stat st;
    if (fd < 0 || fstat(fd, &st) != 0)
        return 1;
    const char *data = mmap(NULL, (size_t)st.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (data == MAP_FAILED) {
        perror("mmap");
        return 1;
    }
    close(fd);                                   /* the mapping stays valid */
    long lines = 0;
    for (off_t i = 0; i < st.st_size; i++)
        lines += data[i] == '\n';
    printf("/etc/services: %jd bytes, %ld lines (via mmap)\n", (intmax_t)st.st_size, lines);
    munmap((void *)data, (size_t)st.st_size);

    /* 2. shared anonymous memory between parent and child */
    int *counter = mmap(NULL, sizeof *counter, PROT_READ | PROT_WRITE,
                        MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (counter == MAP_FAILED)
        return 1;
    *counter = 0;
    pid_t pid = fork();
    if (pid == 0) {
        *counter = 42;                           /* visible to the parent */
        _exit(0);
    }
    waitpid(pid, NULL, 0);
    printf("child wrote %d into shared memory\n", *counter);

    int *private_copy = mmap(NULL, sizeof *private_copy, PROT_READ | PROT_WRITE,
                             MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    *private_copy = 1;
    pid = fork();
    if (pid == 0) {
        *private_copy = 99;                      /* copy-on-write: parent's page unchanged */
        _exit(0);
    }
    waitpid(pid, NULL, 0);
    printf("after child wrote 99 to a MAP_PRIVATE page, parent still sees %d\n", *private_copy);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra mmapdemo.c -o mmapdemo && ./mmapdemo && wc -l < /etc/services
/etc/services: 12813 bytes, 361 lines (via mmap)
child wrote 42 into shared memory
after child wrote 99 to a MAP_PRIVATE page, parent still sees 1
361
```

Mapping is ideal for large read-mostly files (databases, indexes, loading graphs instantly — the OS shares the pages between processes and caches them), and for shared memory between processes (with synchronisation from Chapter 9). Pitfalls: accessing a mapping past the end of a file that was truncated raises `SIGBUS`; and changes to a `MAP_SHARED` file mapping reach the disk only after `msync` or eventually.

<!-- expanded -->
### Going deeper: private and shared mappings, and the page faults behind them

`mmap` creates the mapping, not the memory: nothing is read from the file until a page is touched, and each first touch is a page fault that the kernel serves from the page cache (Chapter 11 of the Linux book measures this). Two flags decide what writes do:

- `MAP_SHARED`: writes go to the page cache and eventually to the file; other processes mapping the same file see them.
- `MAP_PRIVATE`: the first write to a page makes a private copy (copy-on-write); the file never changes.

### Deep & Tricky

**DT-74. Accessing a page of a file mapping that lies beyond the current end of the file raises `SIGBUS`, not `SIGSEGV` — so if another process truncates a file while you have it mapped, your next read of the vanished part kills you.** *(POSIX `mmap`: "References within the address range starting at pa and continuing for len bytes to whole pages following the end of an object shall result in delivery of a SIGBUS signal")* Programs that map files they do not control (log viewers, databases reading shared files) must either handle `SIGBUS` or use `read` instead. The demonstration maps a two-page file, truncates it to one page, and reads both pages.

```c
// file: sigbus.c
#define _POSIX_C_SOURCE 200809L
#include <fcntl.h>
#include <stdio.h>
#include <sys/mman.h>
#include <unistd.h>

int main(void) {
    long page = sysconf(_SC_PAGESIZE);
    int fd = open("mapped.dat", O_RDWR | O_CREAT | O_TRUNC, 0644);
    if (fd < 0 || ftruncate(fd, 2 * page) < 0) return 1;
    char *p = mmap(NULL, 2 * page, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (p == MAP_FAILED) return 1;
    p[0] = 'A';
    p[page] = 'B';
    printf("before truncation: p[0] = %c, p[page] = %c\n", p[0], p[page]);
    fflush(stdout);
    if (ftruncate(fd, page) < 0) return 1;       /* "another process" shrinks the file */
    printf("after truncation:  p[0] = %c\n", p[0]);
    fflush(stdout);
    printf("p[page] = %c\n", p[page]);           /* beyond the end of the file now */
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra sigbus.c -o sigbus && ./sigbus; echo "exit status $?"
before truncation: p[0] = A, p[page] = B
after truncation:  p[0] = A
bash: line 1:  3656 Bus error               ./sigbus
exit status 135
```

The first page is still backed by the file and readable; the second no longer is, and the read raised `SIGBUS` (exit status 135 = 128 + 7).

### Mastery check

1. **Explain.** What does copy-on-write mean for `fork` and for `MAP_PRIVATE` mappings?

## Unit 8.7 — Dynamic loading with `dlopen`

**One-line summary:** `dlopen` loads a shared library at run time, `dlsym` looks up a symbol in it, and `dlclose` unloads it. This is how programs support **plugins**: a stable function-pointer interface defined by the host, implemented by separately compiled `.so` files discovered at run time.

```c
// file: plugins/plugin.h
#ifndef PLUGIN_H
#define PLUGIN_H

#define PLUGIN_API_VERSION 1

typedef struct {
    int api_version;
    const char *name;
    int (*transform)(int);
} Plugin;

/* Every plugin exports: const Plugin *plugin_get(void); */
typedef const Plugin *(*plugin_get_fn)(void);

#endif
```

```c
// file: plugins/double.c
#include "plugin.h"

static int twice(int x) { return 2 * x; }
static const Plugin p = {PLUGIN_API_VERSION, "double", twice};
const Plugin *plugin_get(void) { return &p; }
```

```c
// file: plugins/square.c
#include "plugin.h"

static int sq(int x) { return x * x; }
static const Plugin p = {PLUGIN_API_VERSION, "square", sq};
const Plugin *plugin_get(void) { return &p; }
```

```c
// file: plugins/host.c
#define _POSIX_C_SOURCE 200809L
#include <dlfcn.h>
#include <stdio.h>
#include "plugin.h"

int main(int argc, char *argv[]) {
    for (int i = 1; i < argc; i++) {
        void *h = dlopen(argv[i], RTLD_NOW | RTLD_LOCAL);
        if (h == NULL) {
            fprintf(stderr, "dlopen: %s\n", dlerror());
            continue;
        }
        plugin_get_fn get;
        *(void **)&get = dlsym(h, "plugin_get");   /* POSIX-blessed conversion (DT-37) */
        const Plugin *p = get ? get() : NULL;
        if (p == NULL || p->api_version != PLUGIN_API_VERSION) {
            fprintf(stderr, "%s: not a compatible plugin\n", argv[i]);
            dlclose(h);
            continue;
        }
        printf("plugin %-6s: 7 -> %d\n", p->name, p->transform(7));
        dlclose(h);                                 /* p is invalid after this */
    }
    return 0;
}
```

```console
$ cd plugins && for p in double square; do gcc -std=c17 -Wall -fPIC -shared $p.c -o $p.so; done && gcc -std=c17 -Wall host.c -o host && ./host ./double.so ./square.so ./missing.so
plugin double: 7 -> 14
plugin square: 7 -> 49
dlopen: ./missing.so: cannot open shared object file: No such file or directory
```

The **API version** field protects against loading a plugin built for an older interface — the ABI problem of DT-52, now at run time. (On glibc before 2.34, `-ldl` was needed to link `dlopen`; it is part of libc now.)

### Mastery check

1. **Explain.** Why is it a bug to keep using `p` after `dlclose(h)`?

## Chapter 8 Project — A Unix shell

**Goal:** write `msh`, a small but real shell: it reads commands, runs programs found on `PATH`, supports pipelines of any length, input and output redirection, the built-ins `cd`, `exit`, and `status`, and reports exit statuses like `sh`.

### Specification

- Prompt `msh$ ` when standard input is a terminal; no prompt otherwise (so scripts can be piped in).
- Tokens are separated by whitespace; single and double quotes group words (no escapes or variables needed).
- `cmd1 | cmd2 | … | cmdN` — any number of stages.
- `< file`, `> file`, `>> file` on any stage (input on the first stage, output on the last, in practice).
- Built-ins: `cd [dir]` (default `$HOME`), `exit [n]`, `status` (prints the last exit status).
- The exit status of a pipeline is that of its last command; 127 for command not found; 128+N for death by signal N.
- Ctrl-C kills the running command, not the shell.

### Reference solution

```c
// file: msh/msh.c
/* msh: a small Unix shell with pipelines, redirection, and built-ins. */
#define _POSIX_C_SOURCE 200809L
#include <errno.h>
#include <fcntl.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/wait.h>
#include <unistd.h>

enum { MAX_TOKENS = 128, MAX_STAGES = 16 };

typedef struct {
    char *argv[MAX_TOKENS];
    int argc;
    const char *in, *out;
    int append;
} Stage;

static int last_status = 0;

/* tokenize splits line into tokens, honouring single and double quotes.
   Words are copied into store (at least 2 * strlen(line) + 2 bytes), because a
   word and the operator after it ("a|b") cannot both be terminated in place.
   Operators | < > >> become their own tokens. Returns the count or -1 on error. */
static int tokenize(const char *line, char *tok[], int max, char *store) {
    static char *ops[] = {"|", "<", ">", ">>"};
    int n = 0;
    const char *p = line;
    for (;;) {
        while (*p == ' ' || *p == '\t' || *p == '\n')
            p++;
        if (*p == '\0')
            break;
        if (n == max - 1)
            return -1;
        if (*p == '|' || *p == '<' || *p == '>') {
            if (p[0] == '>' && p[1] == '>') { tok[n++] = ops[3]; p += 2; }
            else { tok[n++] = *p == '|' ? ops[0] : *p == '<' ? ops[1] : ops[2]; p++; }
            continue;
        }
        tok[n++] = store;
        while (*p && *p != ' ' && *p != '\t' && *p != '\n' && *p != '|' && *p != '<' && *p != '>') {
            if (*p == '\'' || *p == '"') {
                char q = *p++;
                while (*p && *p != q)
                    *store++ = *p++;
                if (*p != q)
                    return -1;               /* unterminated quote */
                p++;
            } else {
                *store++ = *p++;
            }
        }
        *store++ = '\0';
    }
    tok[n] = NULL;
    return n;
}

/* parse groups tokens into pipeline stages. Returns the number of stages or -1. */
static int parse(char *tok[], int ntok, Stage st[], int max) {
    int ns = 0;
    memset(&st[0], 0, sizeof st[0]);
    for (int i = 0; i < ntok; i++) {
        const char *t = tok[i];
        Stage *s = &st[ns];
        if (strcmp(t, "|") == 0) {
            if (s->argc == 0 || ns + 1 == max)
                return -1;
            s->argv[s->argc] = NULL;
            memset(&st[++ns], 0, sizeof st[0]);
        } else if (strcmp(t, "<") == 0 || strcmp(t, ">") == 0 || strcmp(t, ">>") == 0) {
            if (i + 1 >= ntok)
                return -1;
            if (t[0] == '<') s->in = tok[++i];
            else { s->out = tok[++i]; s->append = t[1] == '>'; }
        } else {
            if (s->argc == MAX_TOKENS - 1)
                return -1;
            s->argv[s->argc++] = tok[i];
        }
    }
    if (st[ns].argc == 0)
        return ntok == 0 ? 0 : -1;
    st[ns].argv[st[ns].argc] = NULL;
    return ns + 1;
}

static int builtin(Stage *s) {
    const char *cmd = s->argv[0];
    if (strcmp(cmd, "exit") == 0)
        exit(s->argc > 1 ? atoi(s->argv[1]) : last_status);
    if (strcmp(cmd, "cd") == 0) {
        const char *dir = s->argc > 1 ? s->argv[1] : getenv("HOME");
        if (dir == NULL || chdir(dir) != 0) {
            fprintf(stderr, "msh: cd: %s: %s\n", dir ? dir : "", strerror(errno));
            last_status = 1;
        } else {
            last_status = 0;
        }
        return 1;
    }
    if (strcmp(cmd, "status") == 0) {
        printf("%d\n", last_status);
        last_status = 0;
        return 1;
    }
    return 0;
}

/* redirect opens path and installs it on target_fd in the child. */
static void redirect(const char *path, int flags, int target_fd) {
    int fd = open(path, flags, 0644);
    if (fd < 0) {
        fprintf(stderr, "msh: %s: %s\n", path, strerror(errno));
        _exit(1);
    }
    dup2(fd, target_fd);
    close(fd);
}

static void run_pipeline(Stage st[], int ns) {
    if (ns == 1 && builtin(&st[0]))
        return;
    pid_t pids[MAX_STAGES];
    int prev_read = -1;                       /* read end of the previous stage's pipe */
    fflush(stdout);
    for (int i = 0; i < ns; i++) {
        int fds[2] = {-1, -1};
        if (i < ns - 1 && pipe(fds) != 0) {
            perror("msh: pipe");
            break;
        }
        pid_t pid = fork();
        if (pid < 0) {
            perror("msh: fork");
            break;
        }
        if (pid == 0) {
            signal(SIGINT, SIG_DFL);          /* children die on Ctrl-C; the shell doesn't */
            if (prev_read != -1) {
                dup2(prev_read, STDIN_FILENO);
                close(prev_read);
            }
            if (fds[1] != -1) {
                dup2(fds[1], STDOUT_FILENO);
                close(fds[0]);
                close(fds[1]);
            }
            if (st[i].in)
                redirect(st[i].in, O_RDONLY, STDIN_FILENO);
            if (st[i].out)
                redirect(st[i].out, O_WRONLY | O_CREAT | (st[i].append ? O_APPEND : O_TRUNC), STDOUT_FILENO);
            execvp(st[i].argv[0], st[i].argv);
            fprintf(stderr, "msh: %s: command not found\n", st[i].argv[0]);
            _exit(127);
        }
        pids[i] = pid;
        if (prev_read != -1)
            close(prev_read);                 /* the parent keeps no pipe ends open */
        if (fds[1] != -1)
            close(fds[1]);
        prev_read = fds[0];
    }
    for (int i = 0; i < ns; i++) {
        int status;
        if (waitpid(pids[i], &status, 0) < 0)
            continue;
        if (i == ns - 1)
            last_status = WIFEXITED(status) ? WEXITSTATUS(status) : 128 + WTERMSIG(status);
    }
}

int main(void) {
    signal(SIGINT, SIG_IGN);                  /* the shell itself ignores Ctrl-C */
    int interactive = isatty(STDIN_FILENO);
    char *line = NULL;
    size_t cap = 0;
    for (;;) {
        if (interactive) {
            fputs("msh$ ", stdout);
            fflush(stdout);
        }
        if (getline(&line, &cap, stdin) < 0)
            break;
        char *tok[MAX_TOKENS];
        Stage st[MAX_STAGES];
        char *store = malloc(2 * strlen(line) + 2);
        if (store == NULL)
            break;
        int ntok = tokenize(line, tok, MAX_TOKENS, store);
        int ns = ntok < 0 ? -1 : parse(tok, ntok, st, MAX_STAGES);
        if (ns < 0) {
            fprintf(stderr, "msh: syntax error\n");
            last_status = 2;
        } else if (ns > 0) {
            run_pipeline(st, ns);
        }
        free(store);
    }
    free(line);
    return last_status;
}
```

The structure to notice in `run_pipeline`: each stage gets the previous pipe's read end as stdin and a new pipe's write end as stdout; the parent closes each pipe end as soon as the children that need it have been forked, so at the end the parent holds no pipe descriptors at all — the condition for every reader to see end of file.

### Testing with a script

```text
# file: msh/test.msh
echo hello world
echo 'single quoted   spaces' "and double"
printf 'banana\napple\ncherry\n' | sort | head -n 2
seq 1 100000 | grep 7 | wc -l
echo first > out.txt
echo second >> out.txt
cat < out.txt | tr a-z A-Z
false
status
no-such-command
status
sh -c 'kill -TERM $$'
status
cd /tmp
pwd
cd /no/such/dir
status
echo unterminated 'quote
exit 3
```

```console
$ cd msh && gcc -std=c17 -Wall -Wextra -pedantic -g -fsanitize=address,undefined msh.c -o msh && ./msh < test.msh; echo "msh exit status: $?"
msh: #: command not found
hello world
single quoted   spaces and double
apple
banana
40951
FIRST
SECOND
1
msh: no-such-command: command not found
127
143
/tmp
msh: cd: /no/such/dir: No such file or directory
1
msh: syntax error
msh exit status: 3
$ cd msh && printf 'echo a|tr a b\necho x>y.txt\ncat<y.txt\n' | ./msh && sh < test.msh 2>/dev/null | head -n 7
b
x
hello world
single quoted   spaces and double
apple
banana
40951
FIRST
SECOND
```

The first seven results are identical to `sh`'s on the same script (from line 10 the script uses msh's own `status` built-in, where `sh` would use `echo $?`): quotes group words, three-stage and four-stage pipelines work — `grep 7` over 1 to 100,000 counts 40,951 lines — and `>`, `>>`, and `<` redirect. The status lines show `false` giving 1, a missing command 127, death by `SIGTERM` 143, and a failed `cd` 1; the unterminated quote is a syntax error; and `exit 3` sets the shell's own exit status. Operators work without spaces too (`echo a|tr a b`). The first line of `test.msh` is the `# file:` marker, which msh runs as a command named `#` — real shells treat `#` as a comment, an easy extension. AddressSanitizer reported nothing across all of it.

A note from writing this project: the first version of `tokenize` terminated words in place, writing `'\0'` into the line. For `a|b` that overwrote the `|` — the word and the operator after it share no spare byte. GCC's warning about a suspicious empty `if` body led to the rewrite that copies words into a separate buffer.

### Review checklist

- After a pipeline, does the shell hold any pipe descriptors? (Check `/proc/<pid>/fd` from another terminal while a long pipeline runs.)
- Does `cat | cat | cat` with Ctrl-D end cleanly, and does Ctrl-C stop it without killing the shell?
- Does a failed redirection (`cat < missing`) report the error and set status 1 without affecting the next command?
- Are all descriptors the shell opens for itself marked close-on-exec?

### Extensions

1. Comments (`#` to end of line), `$?` and `$VAR` expansion, and `\` escapes.
2. Background jobs (`cmd &`) with a `SIGCHLD` handler that reaps finished jobs and reports them before the next prompt.
3. `&&`, `||`, and `;` command lists.

## Answers to the Chapter 8 prediction questions

**8.2 Q1.** `cat hard.txt` prints `hello`: the file still has one name. `cat soft.txt` fails with "No such file or directory": the symlink points to the name `a.txt`, which no longer exists (a *dangling* symlink).

**8.3 Q1.** Four times: the first `fork` makes 2 processes, the second fork in each makes 4, and each prints once.

# Chapter 9 — Concurrency: Threads, Locks, and Atomics

*Goal: write correct multithreaded C — threads that share data safely, wait for each other without wasting CPU, never deadlock, and actually run faster than one thread.*

A modern CPU has many cores, and a single-threaded program uses one of them. Threads let one process use them all, sharing memory directly — which is exactly what makes them dangerous. In C, a **data race** is undefined behaviour, not just a wrong answer. This chapter teaches POSIX threads, mutexes and condition variables, the classic concurrency bugs and the tools that find them (ThreadSanitizer), C11 atomics and the memory model, lock-free communication, and thread pools — then builds a parallel file indexer and measures its speed-up.

**What you will be able to do after this chapter**

- Create, join, and pass data to threads with pthreads.
- Explain and detect data races, and protect shared data with mutexes.
- Coordinate threads with condition variables, avoiding lost and spurious wake-ups.
- Prevent deadlocks with lock ordering, and choose between mutexes, read-write locks, and atomics.
- Use C11 atomics with the right memory order, and explain acquire/release.
- Build a thread pool and measure parallel speed-up, including the effect of false sharing.

Programs here are compiled with `-pthread` (which sets the right compiler and linker flags for threads) and, during development, with `-fsanitize=thread`.

## Topic 9A — Threads and races

## Unit 9.1 — POSIX threads

**One-line summary:** `pthread_create` starts a new thread running a function `void *f(void *arg)` in the same address space; `pthread_join` waits for it and collects its return value. Every thread has its own stack and registers but shares globals, the heap, and open files with all the others.

```c
// file: threads.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int id;
    long from, to;     /* sum the integers in [from, to) */
    long long result;
} Job;

static void *worker(void *arg) {
    Job *job = arg;                          /* each thread gets its own Job */
    long long s = 0;
    for (long i = job->from; i < job->to; i++)
        s += i;
    job->result = s;                         /* written by one thread only */
    return job;                              /* returned to pthread_join */
}

int main(void) {
    enum { T = 4 };
    const long N = 100000000;
    pthread_t tid[T];
    Job jobs[T];
    for (int t = 0; t < T; t++) {
        jobs[t] = (Job){.id = t, .from = N / T * t, .to = t == T - 1 ? N : N / T * (t + 1)};
        int rc = pthread_create(&tid[t], NULL, worker, &jobs[t]);
        if (rc != 0) {                       /* pthreads return error codes, not errno */
            fprintf(stderr, "pthread_create: error %d\n", rc);
            return 1;
        }
    }
    long long total = 0;
    for (int t = 0; t < T; t++) {
        void *ret;
        pthread_join(tid[t], &ret);
        Job *j = ret;
        printf("thread %d summed [%ld, %ld): %lld\n", j->id, j->from, j->to, j->result);
        total += j->result;
    }
    printf("total = %lld (expected %lld)\n", total, (long long)N * (N - 1) / 2);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 -pthread threads.c -o threads && ./threads
thread 0 summed [0, 25000000): 312499987500000
thread 1 summed [25000000, 50000000): 937499987500000
thread 2 summed [50000000, 75000000): 1562499987500000
thread 3 summed [75000000, 100000000): 2187499987500000
total = 4999999950000000 (expected 4999999950000000)
```

This program has no shared mutable data: each thread reads its own `Job` and writes only its own `result`, and the main thread reads results only after `pthread_join` — which guarantees that everything the thread wrote is visible. **Partitioning the data so threads don't share it** is the simplest and fastest way to write concurrent code. The rest of the chapter is about what to do when they must share.

Points to remember:

- pthread functions **return** an error number (0 on success); they do not set `errno`.
- Passing `&i` of a loop variable to several threads is a classic bug: all threads see the same (changing) variable. Give each thread its own argument object, as `jobs[t]` does.
- A thread that will never be joined should be detached (`pthread_detach`) so its resources are released when it exits; joining or detaching every thread is required.
- `exit` from any thread ends the whole process; returning from `main` does too.

<!-- expanded -->
### Deep & Tricky

**DT-75. Passing `&i` — the address of a loop variable — as the argument of `pthread_create` is a data race: the thread reads `i` whenever it gets around to it, while the loop keeps changing it, so several threads may see the same value and some values may be skipped.** *(Guarantee: C17 §5.1.2.4 ¶35, data race between the loop's write and the thread's read; the start time of a new thread is unspecified)* Give each thread its own argument: an element of an array that outlives the threads, a heap-allocated struct the thread frees, or (for small integers) the value itself converted through `intptr_t`. ThreadSanitizer finds the race even on runs where the output happens to look right.

```c
// file: argrace.c
#include <pthread.h>
#include <stdint.h>
#include <stdio.h>

static void *show_bad(void *arg) { return (void *)(intptr_t)*(int *)arg; }      /* reads the shared i */
static void *show_good(void *arg) { return arg; }                              /* the value travelled with it */

int main(void) {
    enum { N = 8 };
    pthread_t t[N];
    void *got;

    printf("&i passed: ");
    for (int i = 0; i < N; i++) pthread_create(&t[i], NULL, show_bad, &i);     /* every thread gets the same address */
    for (int i = 0; i < N; i++) { pthread_join(t[i], &got); printf("%d ", (int)(intptr_t)got); }

    printf("\nvalue passed: ");
    for (int i = 0; i < N; i++) pthread_create(&t[i], NULL, show_good, (void *)(intptr_t)i);
    for (int i = 0; i < N; i++) { pthread_join(t[i], &got); printf("%d ", (int)(intptr_t)got); }
    printf("\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 -pthread argrace.c -o argrace && ./argrace
&i passed: 8 7 7 8 8 7 7 7 
value passed: 0 1 2 3 4 5 6 7 
$ gcc -std=c17 -g -O1 -pthread -fsanitize=thread argrace.c -o argrace-tsan && ./argrace-tsan 2>&1 | grep -E "WARNING|Previous write|Read of|#0 (show_bad|main)" | head -5 | sed -E 's/0x[0-9a-f]+/ADDR/g; s/ \(BuildId.*//; s/ \(pid=[0-9]+\)//'
WARNING: ThreadSanitizer: data race
  Read of size 4 at ADDR by thread T1:
    #0 show_bad argrace.c:6 (argrace-tsan+ADDR)
  Previous write of size 4 at ADDR by main thread:
    #0 main argrace.c:15 (argrace-tsan+ADDR)
```

The `&i` line usually shows repeated and missing numbers, and different ones on every run (and, worse, the variable `i` of the first loop no longer even exists when a slow thread reads it — the loop's scope has ended). The value-passing line always shows 0 to 7. ThreadSanitizer reports the read in `show_bad` racing with the loop's write in `main`.

### Mastery check

1. **Explain.** Why is it safe for `main` to read `jobs[t].result` after `pthread_join` without any lock?

## Unit 9.2 — Data races and mutexes

**One-line summary:** A **data race** is two threads accessing the same memory location concurrently, at least one writing, without synchronisation — undefined behaviour in C11. Even `count++` is three steps (load, add, store) that can interleave. A **mutex** ensures only one thread at a time executes the code that touches the shared data (the **critical section**). ThreadSanitizer finds races at run time.

```c
// file: race.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdio.h>

enum { THREADS = 4, ITERS = 1000000 };

static long unsafe_count = 0;
static long safe_count = 0;
static pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

static void *racy(void *arg) {
    (void)arg;
    for (int i = 0; i < ITERS; i++)
        unsafe_count++;                    /* data race: undefined behaviour */
    return NULL;
}

static void *locked(void *arg) {
    (void)arg;
    for (int i = 0; i < ITERS; i++) {
        pthread_mutex_lock(&lock);
        safe_count++;                      /* critical section */
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}

int main(void) {
    pthread_t t[THREADS];
    for (int i = 0; i < THREADS; i++) pthread_create(&t[i], NULL, racy, NULL);
    for (int i = 0; i < THREADS; i++) pthread_join(t[i], NULL);
    for (int i = 0; i < THREADS; i++) pthread_create(&t[i], NULL, locked, NULL);
    for (int i = 0; i < THREADS; i++) pthread_join(t[i], NULL);
    printf("expected %d, unsafe %ld, locked %ld\n", THREADS * ITERS, unsafe_count, safe_count);
    return 0;
}
```

```console
$ gcc -std=c17 -O0 -pthread race.c -o race && ./race && ./race
expected 4000000, unsafe 2189869, locked 4000000
expected 4000000, unsafe 2339593, locked 4000000
$ gcc -std=c17 -O2 -pthread race.c -o race2 && ./race2
expected 4000000, unsafe 4000000, locked 4000000
$ gcc -std=c17 -g -O1 -pthread -fsanitize=thread race.c -o race_tsan && ./race_tsan 2>&1 | grep -E 'WARNING|Write of|Previous write|#0' | head -n 5
WARNING: ThreadSanitizer: data race (pid=4157)
  Write of size 8 at 0x5614d07f0070 by thread T1:
    #0 racy race.c:14 (race_tsan+0x12cd) (BuildId: caa31fb096388c75e8f75161ac0dc7a990cd1313)
    #0 racy race.c:14 (race_tsan+0x12a3) (BuildId: caa31fb096388c75e8f75161ac0dc7a990cd1313)
    #0 pthread_create ../../../../src/libsanitizer/tsan/tsan_interceptors_posix.cpp:1022 (libtsan.so.2+0x5ac1a) (BuildId: 2a13a7710e361d06f7babbea53065ca2be93f738)
```

Without optimisation, increments were lost — different numbers on each run. With `-O2` the racy loop happened to give the right answer: the compiler, assuming no other thread could touch `unsafe_count` (a race would be UB), turned each thread's loop into a single `+= 1000000`. The program is still wrong; the race moved rather than disappeared. **ThreadSanitizer** reports the race precisely: two writes from different threads at `race.c:14`, with stacks for both. Run your concurrent tests under TSan routinely.

### Mutex rules

- Every access to shared mutable data — reads too — must hold the same mutex.
- Keep critical sections short; never call slow or blocking operations (I/O, `sleep`, user callbacks) while holding a lock if you can avoid it.
- A mutex initialised with `PTHREAD_MUTEX_INITIALIZER` needs no init call; dynamically created ones use `pthread_mutex_init` and `pthread_mutex_destroy`.
- Unlock on every path out of the critical section — including error returns.

### Deep & Tricky

**DT-76. `volatile` does not make code thread-safe.** *(Guarantee: C11 memory model)* `volatile` prevents the compiler from caching a variable in a register, which was used (wrongly) for thread communication before C11. It does not make `count++` atomic, does not order memory accesses between threads, and does not prevent the CPU from reordering. A `volatile` variable accessed by two threads without synchronisation is still a data race. Use atomics or locks.

### Mastery check

1. **Explain.** Why did the racy loop produce the correct answer at `-O2`, and why is that not reassuring?

## Unit 9.3 — Condition variables

**One-line summary:** A **condition variable** lets a thread sleep until another thread changes some shared state, without busy-waiting. Always wait **in a loop that re-checks the condition**, while holding the mutex that protects that state: `pthread_cond_wait` atomically unlocks the mutex and sleeps, and relocks it before returning — and wake-ups can be spurious.

```c
// file: bqueue.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdbool.h>
#include <stdio.h>

/* A bounded blocking queue: producers wait when full, consumers when empty. */
enum { CAP = 4 };
typedef struct {
    int items[CAP];
    int head, count;
    bool closed;
    pthread_mutex_t mu;
    pthread_cond_t not_empty, not_full;
} BQueue;

static void bq_push(BQueue *q, int v) {
    pthread_mutex_lock(&q->mu);
    while (q->count == CAP)                          /* while, not if: spurious wake-ups */
        pthread_cond_wait(&q->not_full, &q->mu);
    q->items[(q->head + q->count) % CAP] = v;
    q->count++;
    pthread_cond_signal(&q->not_empty);
    pthread_mutex_unlock(&q->mu);
}

/* bq_pop returns false when the queue is closed and empty. */
static bool bq_pop(BQueue *q, int *out) {
    pthread_mutex_lock(&q->mu);
    while (q->count == 0 && !q->closed)
        pthread_cond_wait(&q->not_empty, &q->mu);
    if (q->count == 0) {                             /* closed and drained */
        pthread_mutex_unlock(&q->mu);
        return false;
    }
    *out = q->items[q->head];
    q->head = (q->head + 1) % CAP;
    q->count--;
    pthread_cond_signal(&q->not_full);
    pthread_mutex_unlock(&q->mu);
    return true;
}

static void bq_close(BQueue *q) {
    pthread_mutex_lock(&q->mu);
    q->closed = true;
    pthread_cond_broadcast(&q->not_empty);           /* wake every waiting consumer */
    pthread_mutex_unlock(&q->mu);
}

static BQueue q = {.mu = PTHREAD_MUTEX_INITIALIZER,
                   .not_empty = PTHREAD_COND_INITIALIZER, .not_full = PTHREAD_COND_INITIALIZER};

static void *consumer(void *arg) {
    long id = (long)arg, sum = 0, n = 0;
    int v;
    while (bq_pop(&q, &v)) {
        sum += v;
        n++;
    }
    printf("consumer %ld: %ld items, sum %ld\n", id, n, sum);
    return (void *)sum;
}

int main(void) {
    pthread_t c[3];
    for (long i = 0; i < 3; i++)
        pthread_create(&c[i], NULL, consumer, (void *)i);
    for (int v = 1; v <= 1000; v++)
        bq_push(&q, v);                               /* blocks whenever 4 items are waiting */
    bq_close(&q);
    long total = 0;
    for (int i = 0; i < 3; i++) {
        void *s;
        pthread_join(c[i], &s);
        total += (long)s;
    }
    printf("total %ld (expected %d)\n", total, 1000 * 1001 / 2);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -g -pthread -fsanitize=thread bqueue.c -o bqueue && ./bqueue | sort
consumer 0: 342 items, sum 167540
consumer 1: 333 items, sum 172365
consumer 2: 325 items, sum 160595
total 500500 (expected 500500)
```

How each consumer's share varies from run to run, but the total is always exact and TSan reports nothing. The three rules visible in this code:

1. The condition (`count == CAP`, `count == 0 && !closed`) is checked in a `while` loop under the mutex, because `pthread_cond_wait` can return when no one signalled (a spurious wake-up), and because another thread may have consumed the item between the signal and this thread reacquiring the lock.
2. State changes that others may wait for are made under the mutex, followed by `signal` (wake one waiter) or `broadcast` (wake all — needed for `closed`, which every consumer must see).
3. Shutdown is part of the protocol: a `closed` flag plus `broadcast` lets consumers drain the queue and exit instead of waiting forever.

<!-- expanded -->
### Going deeper: why the wait is always in a `while` loop

`pthread_cond_wait` may return when the condition is not true: the standard allows **spurious wakeups**, and — far more common in practice — another thread may have woken first and consumed what you were waiting for. So the waiting code must re-check its condition after every wake-up, which means a `while`, never an `if`.

```c
// file: ifwait.c
#define _DEFAULT_SOURCE                       /* usleep is no longer in POSIX; glibc still offers it */
#include <pthread.h>
#include <stdint.h>
#include <stdio.h>
#include <unistd.h>

static pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
static pthread_cond_t c = PTHREAD_COND_INITIALIZER;
static int items;                     /* how many items are in the "queue" */
static int use_while;
static int waiting;

static void *consumer(void *arg) {
    pthread_mutex_lock(&m);
    waiting++;
    if (use_while) { while (items == 0) pthread_cond_wait(&c, &m); }
    else           { if (items == 0) pthread_cond_wait(&c, &m); }      /* the bug */
    items--;                                                          /* take one item */
    printf("  consumer %ld took an item, items left = %d\n", (long)(intptr_t)arg, items);
    pthread_mutex_unlock(&m);
    return NULL;
}

static void run(int with_while) {
    pthread_t a, b;
    use_while = with_while;
    items = waiting = 0;
    pthread_create(&a, NULL, consumer, (void *)1);
    pthread_create(&b, NULL, consumer, (void *)2);
    for (;;) {                                    /* wait until both consumers are asleep */
        pthread_mutex_lock(&m);
        int w = waiting;
        pthread_mutex_unlock(&m);
        if (w == 2) break;
        usleep(1000);
    }
    usleep(10000);
    pthread_mutex_lock(&m);
    items = 1;                                    /* produce ONE item ... */
    pthread_cond_broadcast(&c);                   /* ... but wake everybody */
    pthread_mutex_unlock(&m);
    usleep(100000);
    pthread_mutex_lock(&m);
    if (with_while) { items = 1; pthread_cond_broadcast(&c); }   /* second item for the waiting consumer */
    pthread_mutex_unlock(&m);
    pthread_join(a, NULL);
    pthread_join(b, NULL);
}

int main(void) {
    printf("with if:\n");
    run(0);
    printf("with while:\n");
    run(1);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -pthread ifwait.c -o ifwait && ./ifwait
with if:
  consumer 2 took an item, items left = 0
  consumer 1 took an item, items left = -1
with while:
  consumer 2 took an item, items left = 0
  consumer 1 took an item, items left = 0
```

With `if`, both consumers woke from the broadcast and both took "the" item: the count went to -1, which in a real queue means popping from an empty queue — reading a stale slot or crashing. With `while`, the second consumer found the queue empty again, went back to sleep, and took the second item when it came.

### Deep & Tricky

**DT-77. Changing the shared state and signalling must happen with the mutex held — or at least the state change must; signalling a condition variable without first changing the predicate under the mutex can wake nobody, because a thread that checked the predicate just before may not yet be waiting: the "lost wake-up".** *(POSIX `pthread_cond_wait`: the mutex is released and the thread blocked "atomically", which protects only code that changes the predicate while holding the same mutex)* The rule that makes condition variables correct is short: one mutex protects the predicate; every change to the predicate happens with that mutex held; every wait re-checks the predicate in a loop. Signalling after unlocking is allowed and sometimes faster — the predicate change is what must be locked.

### Mastery check

1. **Explain.** What goes wrong if `bq_pop` uses `if` instead of `while`?

## Unit 9.4 — Deadlocks and other locking problems

**One-line summary:** A **deadlock** occurs when threads wait for each other in a cycle — typically thread A holds lock 1 and wants lock 2 while thread B holds lock 2 and wants lock 1. Prevent it by always acquiring locks in one **global order**; detect it with TSan's lock-order checking; and know the other locking tools: read-write locks, `pthread_once`, and recursive mutexes (usually a design smell).

```c
// file: deadlock.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdio.h>

typedef struct {
    int id;
    long balance;
    pthread_mutex_t mu;
} Account;

/* transfer locks the two accounts in a global order (by id), whatever the direction. */
static void transfer(Account *from, Account *to, long amount) {
    Account *first = from->id < to->id ? from : to;
    Account *second = from->id < to->id ? to : from;
    pthread_mutex_lock(&first->mu);
    pthread_mutex_lock(&second->mu);
    from->balance -= amount;
    to->balance += amount;
    pthread_mutex_unlock(&second->mu);
    pthread_mutex_unlock(&first->mu);
}

/* transfer_bad locks 'from' then 'to': opposite transfers can deadlock. */
static void transfer_bad(Account *from, Account *to, long amount) {
    pthread_mutex_lock(&from->mu);
    pthread_mutex_lock(&to->mu);
    from->balance -= amount;
    to->balance += amount;
    pthread_mutex_unlock(&to->mu);
    pthread_mutex_unlock(&from->mu);
}

static Account a = {1, 1000, PTHREAD_MUTEX_INITIALIZER}, b = {2, 1000, PTHREAD_MUTEX_INITIALIZER};
static int use_bad;

static void *a_to_b(void *arg) {
    (void)arg;
    for (int i = 0; i < 100000; i++)
        (use_bad ? transfer_bad : transfer)(&a, &b, 1);
    return NULL;
}
static void *b_to_a(void *arg) {
    (void)arg;
    for (int i = 0; i < 100000; i++)
        (use_bad ? transfer_bad : transfer)(&b, &a, 1);
    return NULL;
}

int main(int argc, char *argv[]) {
    use_bad = argc > 1;
    pthread_t t1, t2;
    pthread_create(&t1, NULL, a_to_b, NULL);
    pthread_create(&t2, NULL, b_to_a, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("a=%ld b=%ld total=%ld\n", a.balance, b.balance, a.balance + b.balance);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -O2 -pthread deadlock.c -o deadlock && ./deadlock
a=1000 b=1000 total=2000
$ timeout 5 ./deadlock bad; echo "exit status $? (124 = killed by timeout: deadlocked)"
exit status 124 (124 = killed by timeout: deadlocked)
$ gcc -std=c17 -g -pthread -fsanitize=thread deadlock.c -o deadlock_tsan && ./deadlock_tsan 2>&1 | grep -E 'WARNING' ; timeout 20 ./deadlock_tsan bad 2>&1 | grep -E 'WARNING' | head -n 1
WARNING: ThreadSanitizer: lock-order-inversion (potential deadlock) (pid=4219)
```

The ordered version always finishes with the total preserved; the unordered version deadlocked within moments — both threads stuck forever, using no CPU. TSan found the **potential** deadlock (a lock-order inversion) from a single execution's lock acquisition pattern, even on runs where the timing happened not to deadlock. That is what makes it valuable: deadlocks are timing-dependent and rare in tests, but the inversion is visible every time.

### Other tools

- **Read-write locks** (`pthread_rwlock_t`) allow many concurrent readers or one writer. Useful for read-mostly data, but slower than a mutex when critical sections are short; measure before switching.
- **`pthread_once`** runs initialisation exactly once, however many threads call it simultaneously.
- **Recursive mutexes** may be relocked by the thread holding them. Needing one usually means the locking structure is unclear; prefer splitting functions into locked and `_locked` (caller holds the lock) variants.
- **`pthread_mutex_trylock`** and timed locks let code back off instead of waiting — one way to recover from potential deadlocks.

### Pattern

**P-9. One lock per piece of shared state, acquired in a documented global order, held briefly.** *(Practice)* Associate each mutex with the data it protects (a comment, or a struct containing both); define a lock order for code that needs several; never call unknown code (callbacks, I/O) while holding a lock; run tests under TSan. *Trade-off:* some loss of parallelism and design effort in exchange for code that cannot deadlock and is checkable.

### Mastery check

1. **Write.** Five dining philosophers with five fork mutexes, first deadlocking, then fixed by lock ordering.

## Topic 9B — Atomics and the memory model

## Unit 9.5 — C11 atomics and memory ordering

**One-line summary:** `<stdatomic.h>` provides atomic types and operations that are free of data races by definition. Each operation takes a **memory order**: `relaxed` (atomicity only), `acquire`/`release` (one-way ordering that publishes data from one thread to another), or `seq_cst` (the default: a single global order). The acquire–release pair is the foundation of all lock-free programming.

```c
// file: atomics.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdatomic.h>
#include <stdbool.h>
#include <stdio.h>
#include <time.h>

enum { THREADS = 4, ITERS = 2000000 };

static atomic_long counter = 0;
static long mutex_counter = 0;
static pthread_mutex_t mu = PTHREAD_MUTEX_INITIALIZER;

static void *atomic_worker(void *arg) {
    (void)arg;
    for (int i = 0; i < ITERS; i++)
        atomic_fetch_add_explicit(&counter, 1, memory_order_relaxed);   /* only atomicity needed */
    return NULL;
}

static void *mutex_worker(void *arg) {
    (void)arg;
    for (int i = 0; i < ITERS; i++) {
        pthread_mutex_lock(&mu);
        mutex_counter++;
        pthread_mutex_unlock(&mu);
    }
    return NULL;
}

/* Message passing: the classic acquire/release pattern. */
static int payload;                     /* plain, non-atomic data */
static atomic_bool ready = false;

static void *producer(void *arg) {
    (void)arg;
    payload = 42;                                                  /* 1. write the data */
    atomic_store_explicit(&ready, true, memory_order_release);     /* 2. publish */
    return NULL;
}

static void *consumer(void *arg) {
    (void)arg;
    while (!atomic_load_explicit(&ready, memory_order_acquire))    /* 3. observe the flag */
        ;                                                          /* (spin: fine for a demo) */
    printf("consumer sees payload = %d\n", payload);              /* 4. guaranteed to see 42 */
    return NULL;
}

static double run(void *(*fn)(void *)) {
    struct timespec a, b;
    pthread_t t[THREADS];
    clock_gettime(CLOCK_MONOTONIC, &a);
    for (int i = 0; i < THREADS; i++) pthread_create(&t[i], NULL, fn, NULL);
    for (int i = 0; i < THREADS; i++) pthread_join(t[i], NULL);
    clock_gettime(CLOCK_MONOTONIC, &b);
    return (b.tv_sec - a.tv_sec) * 1e3 + (b.tv_nsec - a.tv_nsec) / 1e6;
}

int main(void) {
    double ta = run(atomic_worker), tm = run(mutex_worker);
    printf("atomic: %ld in %.0f ms; mutex: %ld in %.0f ms\n",
           atomic_load(&counter), ta, mutex_counter, tm);

    pthread_t p, c;
    pthread_create(&c, NULL, consumer, NULL);
    pthread_create(&p, NULL, producer, NULL);
    pthread_join(p, NULL);
    pthread_join(c, NULL);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 -pthread atomics.c -o atomics && ./atomics
atomic: 8000000 in 196 ms; mutex: 8000000 in 152 ms
consumer sees payload = 42
$ gcc -std=c17 -g -O1 -pthread -fsanitize=thread atomics.c -o atomics_tsan && ./atomics_tsan 2>&1 | grep -c WARNING || true
0
```

Both counters are exact. The atomic increment is a single `lock xadd` instruction, considerably faster than lock/unlock — though with four threads hammering one variable, both are slow compared with each thread counting privately and adding once at the end (Unit 9.1's partitioning). TSan reports zero warnings, because the acquire/release pair makes the non-atomic `payload` access properly synchronised.

### The memory orders

| Order | Guarantees | Use for |
| --- | --- | --- |
| `memory_order_relaxed` | the operation itself is atomic; no ordering of other memory accesses | counters, statistics |
| `memory_order_release` (store) | all writes *before* it in this thread are visible to a thread that acquire-loads the value | publishing data, unlocking |
| `memory_order_acquire` (load) | all reads *after* it see the writes the releasing thread made before its release | consuming published data, locking |
| `memory_order_acq_rel` | both, for read-modify-write operations | lock-free updates |
| `memory_order_seq_cst` (default) | acquire/release plus one total order of all seq_cst operations | when unsure; easiest to reason about |

Why ordering matters: both the compiler and the CPU reorder memory operations that look independent in one thread. Without release/acquire, the consumer could see `ready == true` and still read the old `payload` (ARM and POWER CPUs really do this; x86's stronger model hides it at the hardware level, but the compiler may still reorder). A mutex unlock is a release and a lock is an acquire, which is why mutex-protected data is visible to the next thread that takes the lock.

`atomic_compare_exchange_weak`/`_strong` (CAS) is the building block of lock-free algorithms: "if the value is still what I read, replace it with my new value; otherwise tell me what it is now".

### Deep & Tricky

**DT-78. Plain `int` flags between threads are a data race, even on x86.** *(Guarantee: C11 memory model)* `while (!done) ;` on a non-atomic `done` written by another thread is undefined behaviour; in practice the compiler hoists the load out of the loop and spins forever at `-O2`. Use `atomic_bool` with acquire/release, or a mutex and condition variable.

### Mastery check

1. **Explain.** Why is `memory_order_relaxed` enough for the counter but not for the `ready` flag?

## Unit 9.6 — A lock-free single-producer, single-consumer queue

**One-line summary:** The ring buffer of Unit 7.3 becomes a **lock-free queue** for exactly one producer thread and one consumer thread: the producer alone writes `tail`, the consumer alone writes `head`, and acquire/release on those indices publishes the slot contents. No locks, no system calls — the structure behind high-performance logging and audio pipelines.

```c
// file: spsc.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdatomic.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>

#define CAP 1024u                                     /* power of two */

typedef struct {
    _Alignas(64) atomic_uint_fast32_t head;          /* written by the consumer only */
    _Alignas(64) atomic_uint_fast32_t tail;          /* written by the producer only */
    _Alignas(64) uint64_t slot[CAP];
} Spsc;

static bool spsc_push(Spsc *q, uint64_t v) {
    uint_fast32_t t = atomic_load_explicit(&q->tail, memory_order_relaxed);   /* our own index */
    uint_fast32_t h = atomic_load_explicit(&q->head, memory_order_acquire);   /* consumer's progress */
    if (t - h == CAP)
        return false;                                                         /* full */
    q->slot[t & (CAP - 1)] = v;                                               /* write the data ... */
    atomic_store_explicit(&q->tail, t + 1, memory_order_release);             /* ... then publish it */
    return true;
}

static bool spsc_pop(Spsc *q, uint64_t *out) {
    uint_fast32_t h = atomic_load_explicit(&q->head, memory_order_relaxed);
    uint_fast32_t t = atomic_load_explicit(&q->tail, memory_order_acquire);   /* see published slots */
    if (t == h)
        return false;                                                         /* empty */
    *out = q->slot[h & (CAP - 1)];
    atomic_store_explicit(&q->head, h + 1, memory_order_release);             /* free the slot */
    return true;
}

static Spsc q;
enum { N = 10000000 };

static void *producer(void *arg) {
    (void)arg;
    for (uint64_t i = 1; i <= N; i++)
        while (!spsc_push(&q, i))
            ;                                         /* queue full: spin */
    return NULL;
}

int main(void) {
    pthread_t p;
    pthread_create(&p, NULL, producer, NULL);
    uint64_t sum = 0, v, expected_next = 1, out_of_order = 0;
    for (long n = 0; n < N;) {
        if (spsc_pop(&q, &v)) {
            out_of_order += v != expected_next;
            expected_next = v + 1;
            sum += v;
            n++;
        }
    }
    pthread_join(p, NULL);
    printf("received %d items, sum %llu (expected %llu), out of order: %llu\n",
           N, (unsigned long long)sum, (unsigned long long)N * (N + 1) / 2,
           (unsigned long long)out_of_order);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 -pthread spsc.c -o spsc && time ./spsc
received 10000000 items, sum 50000005000000 (expected 50000005000000), out of order: 0

real	0m0.126s
user	0m0.233s
sys	0m0.000s
$ gcc -std=c17 -g -O1 -pthread -fsanitize=thread spsc.c -o spsc_tsan && ./spsc_tsan 2>&1 | grep -c WARNING || true
0
```

Ten million items crossed between threads in order, with no lock, and TSan confirms the slot accesses are correctly synchronised by the release/acquire pairs. The `_Alignas(64)` on each index keeps `head` and `tail` on separate cache lines — otherwise the two cores would fight over one line on every operation (false sharing, Unit 9.7).

This design is correct **only** for one producer and one consumer. Multi-producer or multi-consumer lock-free queues need compare-and-swap loops and are notoriously subtle (the ABA problem, memory reclamation). Use a well-reviewed library, or a mutex and condition variable, unless measurement proves the need.

### Mastery check

1. **Explain.** Which `memory_order_release` in `spsc_push` makes the consumer see the slot's value, and what could happen if it were `relaxed`?

## Topic 9C — Concurrency in practice

## Unit 9.7 — Thread pools, thread-local storage, and false sharing

**One-line summary:** Creating a thread per task is expensive; a **thread pool** keeps a fixed set of worker threads taking tasks from a queue. **Thread-local storage** (`_Thread_local`) gives each thread its own copy of a variable. **False sharing** — independent variables written by different threads sitting in the same 64-byte cache line — can make parallel code slower than serial code.

```c
// file: falseshare.c
#define _POSIX_C_SOURCE 200809L
#include <pthread.h>
#include <stdio.h>
#include <time.h>

enum { THREADS = 2, ITERS = 200000000 };

struct packed { volatile long count; };                    /* neighbours share a cache line */
struct padded { _Alignas(64) volatile long count; };      /* one cache line each */

static struct packed packed_counters[THREADS];
static struct padded padded_counters[THREADS];
static _Thread_local long tls_calls;                       /* one per thread */

static void *bump_packed(void *arg) {
    long i = (long)arg;
    for (int k = 0; k < ITERS; k++)
        packed_counters[i].count++;
    tls_calls++;
    return NULL;
}

static void *bump_padded(void *arg) {
    long i = (long)arg;
    for (int k = 0; k < ITERS; k++)
        padded_counters[i].count++;
    tls_calls++;
    return NULL;
}

static double run(void *(*fn)(void *)) {
    struct timespec a, b;
    pthread_t t[THREADS];
    clock_gettime(CLOCK_MONOTONIC, &a);
    for (long i = 0; i < THREADS; i++) pthread_create(&t[i], NULL, fn, (void *)i);
    for (int i = 0; i < THREADS; i++) pthread_join(t[i], NULL);
    clock_gettime(CLOCK_MONOTONIC, &b);
    return (b.tv_sec - a.tv_sec) * 1e3 + (b.tv_nsec - a.tv_nsec) / 1e6;
}

int main(void) {
    printf("sizeof(struct packed)=%zu sizeof(struct padded)=%zu\n",
           sizeof(struct packed), sizeof(struct padded));
    double p = run(bump_packed), q = run(bump_padded);
    printf("same cache line: %.0f ms, separate lines: %.0f ms (%.1fx)\n", p, q, p / q);
    printf("main thread's tls_calls = %ld (each worker had its own copy)\n", tls_calls);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -O2 -pthread falseshare.c -o falseshare && ./falseshare
sizeof(struct packed)=8 sizeof(struct padded)=64
same cache line: 423 ms, separate lines: 54 ms (7.9x)
main thread's tls_calls = 0 (each worker had its own copy)
```

The two threads never touch each other's counter, yet sharing a cache line made them several times slower: each write invalidates the line in the other core's cache, and it bounces back and forth. Padding each counter to its own line removed the contention. (`volatile` here only stops the compiler from collapsing each loop into one addition; it plays no part in correctness.) Per-thread data should be padded or, better, kept in thread-local or stack variables and combined at the end.

### A thread pool

The structure every pool shares — a task queue protected by a mutex and condition variable, `N` workers looping "wait for a task, run it", and a shutdown flag with `broadcast` — is exactly Unit 9.3's blocking queue with function pointers as items. The chapter project builds one.

### Mastery check

1. **Explain.** Why does `_Thread_local` make `strtok`-style hidden state safe, and what is the price?

## Unit 9.8 — C11 `<threads.h>`

**One-line summary:** C11 standardised a small threading API — `thrd_create`, `thrd_join`, `mtx_t`, `cnd_t`, `tss_t`, `call_once` — that mirrors pthreads closely. glibc supports it since 2.28; it is portable to Windows via some C libraries but lacks pthreads' attributes (stack size, scheduling). Most Unix code still uses pthreads; both use the same C11 memory model.

```c
// file: c11threads.c
#include <stdio.h>
#include <threads.h>

static mtx_t mu;
static int total;

static int work(void *arg) {
    int n = *(int *)arg;
    mtx_lock(&mu);
    total += n;
    mtx_unlock(&mu);
    return n * 10;
}

int main(void) {
    mtx_init(&mu, mtx_plain);
    thrd_t t[3];
    int args[3] = {1, 2, 3};
    for (int i = 0; i < 3; i++)
        if (thrd_create(&t[i], work, &args[i]) != thrd_success)
            return 1;
    for (int i = 0; i < 3; i++) {
        int res;
        thrd_join(t[i], &res);
        printf("thread %d returned %d\n", i, res);
    }
    printf("total %d\n", total);
    mtx_destroy(&mu);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -pthread c11threads.c -o c11threads && ./c11threads
thread 0 returned 10
thread 1 returned 20
thread 2 returned 30
total 6
```

<!-- expanded -->
### Going deeper: C11 threads or POSIX threads?

| | POSIX threads (`<pthread.h>`) | C11 threads (`<threads.h>`) |
| --- | --- | --- |
| Standard | POSIX (Linux, macOS, BSDs; Windows via libraries) | ISO C11, optional (`__STDC_NO_THREADS__`) |
| Availability | everywhere POSIX is | glibc since 2.28 (2018), musl, MSVC since 2022; not macOS's libc |
| Read-write locks, barriers, spin locks | yes | no |
| Thread attributes (stack size, scheduling, CPU affinity) | yes | no |
| Cancellation, `pthread_atfork` | yes | no |
| Timed waits | with `CLOCK_REALTIME` or (`pthread_cond_clockwait`, `pthread_condattr_setclock`) monotonic | `TIME_UTC` only |

C11 threads cover the common core — create, join, mutex, condition variable, thread-local storage, call-once — with error codes instead of `errno`. They are a reasonable choice for small portable programs that also target Windows. Larger programs on Linux use POSIX threads for everything C11 lacks; and `_Atomic` (Unit 9.5) works with either API.

## Chapter 9 Project — A parallel file indexer

**Goal:** build `pindex`, which walks a directory tree, reads every regular file, counts word frequencies, and prints the most common words — using a thread pool whose workers each count into a **private** hash table, merged at the end. Measure the speed-up from 1 to N threads, and prove the results identical and race-free under ThreadSanitizer.

### Design

```
main thread: nftw() walks the tree ──push(path)──▶ [ bounded task queue ] ──pop──▶ worker 1..N
                                                                                  │ read file, count words
                                                                                  ▼ into private table
main thread: close queue, join workers, merge the N tables, sort, print top K
```

Decisions: workers never share a table (no locking in the hot loop — Unit 9.1's partitioning); the only shared structure is the queue (mutex + two condition variables — Unit 9.3); merging is single-threaded and cheap compared with reading.

### Reference solution

The word table is a compact open-addressing hash map specialised for this job (string → count), in the style of Chapter 7's project:

```c
// file: pindex/wordtab.h
#ifndef WORDTAB_H
#define WORDTAB_H
#include <stddef.h>

typedef struct { char *word; long count; } WordCount;
typedef struct { WordCount *slots; size_t cap, len; } WordTab;

int wt_add(WordTab *t, const char *word, size_t len, long n);   /* 0 or -1 (no memory) */
void wt_free(WordTab *t);

#endif
```

```c
// file: pindex/wordtab.c
#include "wordtab.h"
#include <stdint.h>
#include <stdlib.h>
#include <string.h>

static uint64_t hash(const char *s, size_t n) {
    uint64_t h = 14695981039346656037u;
    for (size_t i = 0; i < n; i++) {
        h ^= (unsigned char)s[i];
        h *= 1099511628211u;
    }
    return h;
}

static int grow(WordTab *t) {
    size_t cap = t->cap ? t->cap * 2 : 1024;
    WordCount *s = calloc(cap, sizeof *s);
    if (s == NULL)
        return -1;
    for (size_t i = 0; i < t->cap; i++) {
        if (t->slots[i].word == NULL)
            continue;
        size_t j = hash(t->slots[i].word, strlen(t->slots[i].word)) & (cap - 1);
        while (s[j].word != NULL)
            j = (j + 1) & (cap - 1);
        s[j] = t->slots[i];
    }
    free(t->slots);
    t->slots = s;
    t->cap = cap;
    return 0;
}

int wt_add(WordTab *t, const char *word, size_t len, long n) {
    if ((t->len + 1) * 4 > t->cap * 3 && grow(t) != 0)        /* load factor <= 0.75 */
        return -1;
    size_t i = hash(word, len) & (t->cap - 1);
    while (t->slots[i].word != NULL) {
        if (strncmp(t->slots[i].word, word, len) == 0 && t->slots[i].word[len] == '\0') {
            t->slots[i].count += n;
            return 0;
        }
        i = (i + 1) & (t->cap - 1);
    }
    char *w = malloc(len + 1);
    if (w == NULL)
        return -1;
    memcpy(w, word, len);
    w[len] = '\0';
    t->slots[i] = (WordCount){w, n};
    t->len++;
    return 0;
}

void wt_free(WordTab *t) {
    for (size_t i = 0; i < t->cap; i++)
        free(t->slots[i].word);
    free(t->slots);
    *t = (WordTab){0};
}
```

```c
// file: pindex/pindex.c
/* pindex: count word frequencies in a directory tree with a thread pool. */
#define _XOPEN_SOURCE 700                /* nftw */
#include <ctype.h>
#include <ftw.h>
#include <pthread.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include "wordtab.h"

/* ---- bounded queue of paths (Unit 9.3) ---- */
enum { QCAP = 256 };
static struct {
    char *paths[QCAP];
    int head, count;
    bool closed;
    pthread_mutex_t mu;
    pthread_cond_t not_empty, not_full;
} q = {.mu = PTHREAD_MUTEX_INITIALIZER, .not_empty = PTHREAD_COND_INITIALIZER,
       .not_full = PTHREAD_COND_INITIALIZER};

static void q_push(char *path) {
    pthread_mutex_lock(&q.mu);
    while (q.count == QCAP)
        pthread_cond_wait(&q.not_full, &q.mu);
    q.paths[(q.head + q.count++) % QCAP] = path;
    pthread_cond_signal(&q.not_empty);
    pthread_mutex_unlock(&q.mu);
}

static char *q_pop(void) {                  /* NULL when closed and empty */
    pthread_mutex_lock(&q.mu);
    while (q.count == 0 && !q.closed)
        pthread_cond_wait(&q.not_empty, &q.mu);
    char *p = NULL;
    if (q.count > 0) {
        p = q.paths[q.head];
        q.head = (q.head + 1) % QCAP;
        q.count--;
        pthread_cond_signal(&q.not_full);
    }
    pthread_mutex_unlock(&q.mu);
    return p;
}

static void q_close(void) {
    pthread_mutex_lock(&q.mu);
    q.closed = true;
    pthread_cond_broadcast(&q.not_empty);
    pthread_mutex_unlock(&q.mu);
}

/* ---- workers ---- */
typedef struct {
    WordTab tab;                             /* private to this worker: no locking */
    long files, bytes;
    _Alignas(64) char pad;                   /* keep workers' hot fields on separate lines */
} Worker;

static void count_file(Worker *w, const char *path) {
    FILE *f = fopen(path, "rb");
    if (f == NULL)
        return;
    char buf[65536], word[64];
    size_t len = 0, n;
    while ((n = fread(buf, 1, sizeof buf, f)) > 0) {
        w->bytes += (long)n;
        for (size_t i = 0; i < n; i++) {
            unsigned char c = (unsigned char)buf[i];
            if (isalpha(c)) {
                if (len < sizeof word)
                    word[len++] = (char)tolower(c);
            } else if (len > 0) {
                if (len < sizeof word)       /* ignore absurdly long "words" */
                    wt_add(&w->tab, word, len, 1);
                len = 0;
            }
        }
    }
    if (len > 0 && len < sizeof word)
        wt_add(&w->tab, word, len, 1);
    w->files++;
    fclose(f);
}

static void *worker_main(void *arg) {
    Worker *w = arg;
    char *path;
    while ((path = q_pop()) != NULL) {
        count_file(w, path);
        free(path);
    }
    return NULL;
}

static int visit(const char *path, const struct stat *st, int type, struct FTW *ftw) {
    (void)st; (void)ftw;
    if (type == FTW_F) {
        char *copy = strdup(path);
        if (copy != NULL)
            q_push(copy);
    }
    return 0;                                /* keep walking */
}

static int by_count(const void *a, const void *b) {
    const WordCount *x = a, *y = b;
    if (x->count != y->count)
        return x->count < y->count ? 1 : -1;
    return strcmp(x->word, y->word);         /* deterministic order for ties */
}

int main(int argc, char *argv[]) {
    if (argc < 3) {
        fprintf(stderr, "usage: pindex THREADS DIR [TOP]\n");
        return 2;
    }
    int nthreads = atoi(argv[1]);
    int top = argc > 3 ? atoi(argv[3]) : 10;
    if (nthreads < 1 || nthreads > 64)
        return 2;

    struct timespec t0, t1;
    clock_gettime(CLOCK_MONOTONIC, &t0);
    Worker *workers = calloc((size_t)nthreads, sizeof *workers);
    pthread_t *tids = calloc((size_t)nthreads, sizeof *tids);
    if (workers == NULL || tids == NULL)
        return 1;
    for (int i = 0; i < nthreads; i++)
        pthread_create(&tids[i], NULL, worker_main, &workers[i]);
    if (nftw(argv[2], visit, 32, FTW_PHYS) != 0)
        perror(argv[2]);
    q_close();

    WordTab merged = {0};
    long files = 0, bytes = 0;
    for (int i = 0; i < nthreads; i++) {
        pthread_join(tids[i], NULL);         /* after join, the worker's table is safe to read */
        files += workers[i].files;
        bytes += workers[i].bytes;
        for (size_t s = 0; s < workers[i].tab.cap; s++) {
            WordCount *wc = &workers[i].tab.slots[s];
            if (wc->word != NULL)
                wt_add(&merged, wc->word, strlen(wc->word), wc->count);
        }
        wt_free(&workers[i].tab);
    }
    WordCount *all = malloc(merged.len * sizeof *all);
    size_t n = 0;
    for (size_t s = 0; s < merged.cap; s++)
        if (merged.slots[s].word != NULL)
            all[n++] = merged.slots[s];
    qsort(all, n, sizeof *all, by_count);
    clock_gettime(CLOCK_MONOTONIC, &t1);

    printf("%ld files, %.1f MB, %zu distinct words, %d threads, %.0f ms\n", files, bytes / 1e6, n,
           nthreads, (t1.tv_sec - t0.tv_sec) * 1e3 + (t1.tv_nsec - t0.tv_nsec) / 1e6);
    for (int i = 0; i < top && (size_t)i < n; i++)
        printf("%8ld %s\n", all[i].count, all[i].word);
    free(all);
    wt_free(&merged);
    free(workers);
    free(tids);
    return 0;
}
```

A generator makes a reproducible corpus — 400 files of pseudo-random text drawn from a fixed vocabulary with a skewed (Zipf-like) distribution, as natural language has:

```c
// file: pindex/gencorpus.c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>

static const char *vocab[] = {
    "the", "of", "and", "to", "in", "is", "that", "for", "it", "as", "with", "was", "on",
    "kernel", "thread", "memory", "pointer", "lock", "queue", "cache", "file", "process",
    "signal", "socket", "buffer", "atomic", "mutex", "compiler", "linker", "stack", "heap",
};

int main(int argc, char *argv[]) {
    if (argc != 4) return 2;
    const char *dir = argv[1];
    int files = atoi(argv[2]), words = atoi(argv[3]);
    size_t nv = sizeof vocab / sizeof vocab[0];
    srand(2024);
    for (int f = 0; f < files; f++) {
        char sub[512], path[600];
        snprintf(sub, sizeof sub, "%s/d%02d", dir, f % 10);
        mkdir(dir, 0755);
        mkdir(sub, 0755);
        snprintf(path, sizeof path, "%s/file%04d.txt", sub, f);
        FILE *out = fopen(path, "w");
        if (out == NULL) return 1;
        for (int w = 0; w < words; w++) {
            size_t r = (size_t)rand() % nv;
            size_t k = (size_t)rand() % (r + 1);          /* skew towards early words */
            fprintf(out, "%s%c", vocab[k], w % 12 == 11 ? '\n' : ' ');
        }
        fclose(out);
    }
    return 0;
}
```

```console
$ cd pindex && gcc -std=c17 -O2 -Wall gencorpus.c -o gencorpus && rm -rf corpus && ./gencorpus corpus 400 50000 && du -sh corpus | cut -f1
83M
$ cd pindex && gcc -std=c17 -Wall -Wextra -O2 -pthread pindex.c wordtab.c -o pindex && for t in 1 2 4; do ./pindex $t corpus 3; done
400 files, 86.0 MB, 31 distinct words, 1 threads, 542 ms
 2597606 the
 1951814 of
 1629379 and
400 files, 86.0 MB, 31 distinct words, 2 threads, 280 ms
 2597606 the
 1951814 of
 1629379 and
400 files, 86.0 MB, 31 distinct words, 4 threads, 257 ms
 2597606 the
 1951814 of
 1629379 and
$ cd pindex && cat corpus/*/*.txt | tr -cs 'A-Za-z' '\n' | tr A-Z a-z | grep -v '^$' | sort | uniq -c | sort -k1,1nr -k2 | head -n 3
2597606 the
1951814 of
1629379 and
$ cd pindex && gcc -std=c17 -g -O1 -pthread -fsanitize=thread pindex.c wordtab.c -o pindex_tsan && ./pindex_tsan 4 corpus 1 2>&1 | grep -c WARNING || true
0
```

Results:

- **Correctness:** every thread count produced identical counts, which match an independent Unix pipeline (`tr | sort | uniq -c`) exactly; ThreadSanitizer reported no races.
- **Speed-up:** on this 2-core machine, two threads ran close to twice as fast as one; four threads gained nothing more — there are only two cores to run them. Speed-up is bounded by cores, by the serial parts (directory walk, merge), and by I/O (here the files were in the page cache after the first run; on a cold disk, I/O would dominate and more threads than cores can help by overlapping reads).

### Review checklist

- Is any data touched by more than one worker without the queue's mutex? (The answer should be "only after `pthread_join`".)
- Does the program terminate if the directory is empty, or if `nftw` fails midway?
- Are results deterministic regardless of thread count and scheduling (including tie order)?
- Is the speed-up measured over several runs, with the same cache state for each configuration?

### Extensions

1. Let workers merge into a shared table under a mutex instead, and measure the slowdown caused by contention.
2. Replace the queue with per-worker queues and work stealing.
3. Memory-map the files (Unit 8.6) instead of `fread`, and measure.

## Answers to the Chapter 9 prediction questions

There are no prediction questions in this chapter; the mastery-check explanations are answered in the units' text.

# Chapter 10 — Network Programming

*Goal: write networked programs in C — clients and servers over TCP and UDP, with correct framing, robust error handling, and an event loop that serves thousands of connections from one thread.*

Every web server, database, and message broker is, at its core, a C (or C-like) program calling the **sockets API**: a set of system calls introduced by BSD Unix in 1983 and still the interface to every operating system's network stack. This chapter teaches that API from the ground up — addresses and byte order, TCP clients and servers, message framing, UDP — then the concurrency models servers use, ending with non-blocking I/O and `epoll`. The project is an event-driven HTTP server, tested with `curl` and load-tested with a client you write yourself.

**What you will be able to do after this chapter**

- Resolve names with `getaddrinfo` and handle IPv4 and IPv6 uniformly.
- Write TCP clients and servers with `socket`, `bind`, `listen`, `accept`, and `connect`.
- Frame messages over a byte stream, handling partial reads and writes.
- Use UDP for datagram protocols.
- Compare process-per-connection, thread-per-connection, and event-driven servers.
- Build an `epoll` event loop with non-blocking sockets.
- Parse HTTP/1.1 requests defensively and serve files.

## Topic 10A — Sockets

## Unit 10.1 — Addresses, ports, and byte order

**One-line summary:** A network endpoint is an **IP address** (IPv4: 32 bits, IPv6: 128 bits) plus a **port** (16 bits). Multi-byte numbers in protocol headers are **big-endian** (network byte order), converted with `htons`/`ntohs`/`htonl`/`ntohl`. Use `getaddrinfo` to turn host names and service names into socket addresses — it handles DNS, IPv4, and IPv6 for you.

```c
// file: resolve.c
#define _POSIX_C_SOURCE 200809L
#include <arpa/inet.h>
#include <netdb.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    const char *host = argc > 1 ? argv[1] : "localhost";
    struct addrinfo hints = {.ai_family = AF_UNSPEC, .ai_socktype = SOCK_STREAM};
    struct addrinfo *res;
    int rc = getaddrinfo(host, "http", &hints, &res);     /* service "http" -> port 80 */
    if (rc != 0) {
        fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rc));
        return 1;
    }
    for (struct addrinfo *ai = res; ai != NULL; ai = ai->ai_next) {
        char ip[INET6_ADDRSTRLEN];
        unsigned port;
        if (ai->ai_family == AF_INET) {
            struct sockaddr_in *s = (struct sockaddr_in *)ai->ai_addr;
            inet_ntop(AF_INET, &s->sin_addr, ip, sizeof ip);
            port = ntohs(s->sin_port);                    /* network to host byte order */
        } else {
            struct sockaddr_in6 *s = (struct sockaddr_in6 *)ai->ai_addr;
            inet_ntop(AF_INET6, &s->sin6_addr, ip, sizeof ip);
            port = ntohs(s->sin6_port);
        }
        printf("%s -> %s port %u (%s)\n", host, ip, port, ai->ai_family == AF_INET ? "IPv4" : "IPv6");
    }
    freeaddrinfo(res);

    unsigned short p = 8080;
    printf("8080 = 0x%04x in host order; htons gives 0x%04x\n", p, htons(p));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra resolve.c -o resolve && ./resolve localhost
localhost -> 127.0.0.1 port 80 (IPv4)
8080 = 0x1f90 in host order; htons gives 0x901f
```

`getaddrinfo` returns a linked list of candidate addresses — try them in order until one works (that is how clients handle hosts with both IPv6 and IPv4 addresses). `struct sockaddr` is a generic header; the concrete types are `sockaddr_in` (IPv4) and `sockaddr_in6` (IPv6), and `sockaddr_storage` is big enough for any. The older `gethostbyname` and `inet_addr` are obsolete: IPv4-only and (for `gethostbyname`) not thread-safe.

<!-- expanded -->
### Deep & Tricky

**DT-79. Forgetting `htons` on a port number does not cause an error — the program binds or connects to a different port: on a little-endian machine port 8080 (0x1F90) becomes 36895 (0x901F).** *(POSIX: `sin_port` and `sin_addr` are "in network byte order"; the conversion functions `htons`/`htonl`/`ntohs`/`ntohl` are no-ops on big-endian machines, so the bug is invisible there)* The symptom is a server that starts fine and a client that gets "connection refused". `ss -ltn` (or `netstat -ltn`) shows the port the server really listens on.

```c
// file: byteorder.c
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <sys/socket.h>
#include <unistd.h>

static int listen_on(unsigned short port_field) {
    int s = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in a = { .sin_family = AF_INET, .sin_port = port_field, .sin_addr.s_addr = htonl(INADDR_LOOPBACK) };
    if (bind(s, (struct sockaddr *)&a, sizeof a) < 0 || listen(s, 1) < 0) return -1;
    socklen_t len = sizeof a;
    getsockname(s, (struct sockaddr *)&a, &len);
    printf("  asked for field value 0x%04x -> listening on port %u\n", port_field, ntohs(a.sin_port));
    return s;
}

int main(void) {
    printf("8080 = 0x%04x; htons(8080) = 0x%04x in memory order\n", 8080, htons(8080));
    int good = listen_on(htons(8080));
    int bad = listen_on(8080);                       /* bug: host byte order */
    close(good);
    close(bad);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra byteorder.c -o byteorder && ./byteorder
8080 = 0x1f90; htons(8080) = 0x901f in memory order
  asked for field value 0x901f -> listening on port 8080
  asked for field value 0x1f90 -> listening on port 36895
```

### Mastery check

1. **Explain.** Why must port numbers be converted with `htons` before being stored in a `sockaddr_in`?

## Unit 10.2 — A TCP server and client

**One-line summary:** A TCP server calls `socket`, `bind` (to an address and port), `listen`, then loops on `accept`, which returns a **new** descriptor for each client connection. A client calls `socket` and `connect`. After that, both sides `read` and `write` a reliable, ordered **byte stream** — and both must handle partial reads and writes (DT-68).

```c
// file: echo/net.h
#ifndef NET_H
#define NET_H
#define _GNU_SOURCE              /* accept4 is a Linux extension; include this header first */
#include <stddef.h>
#include <sys/types.h>

/* listen_on returns a listening TCP socket on port (all addresses), or -1. */
int listen_on(const char *port);

/* connect_to returns a socket connected to host:port, or -1. */
int connect_to(const char *host, const char *port);

/* write_all writes n bytes, retrying partial writes. Returns 0 or -1. */
int write_all(int fd, const void *buf, size_t n);

#endif
```

```c
// file: echo/net.c
#include "net.h"

#include <errno.h>
#include <netdb.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

int listen_on(const char *port) {
    struct addrinfo hints = {.ai_family = AF_UNSPEC, .ai_socktype = SOCK_STREAM, .ai_flags = AI_PASSIVE};
    struct addrinfo *res;
    if (getaddrinfo(NULL, port, &hints, &res) != 0)
        return -1;
    int fd = -1;
    for (struct addrinfo *ai = res; ai != NULL; ai = ai->ai_next) {   /* "::" and/or "0.0.0.0" */
        fd = socket(ai->ai_family, ai->ai_socktype | SOCK_CLOEXEC, 0);
        if (fd < 0)
            continue;                                  /* e.g. IPv6 disabled on this host */
        int on = 1, off = 0;
        setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof on);       /* restart without waiting */
        if (ai->ai_family == AF_INET6)
            setsockopt(fd, IPPROTO_IPV6, IPV6_V6ONLY, &off, sizeof off); /* accept IPv4 too */
        if (bind(fd, ai->ai_addr, ai->ai_addrlen) == 0 && listen(fd, SOMAXCONN) == 0)
            break;
        close(fd);
        fd = -1;
    }
    freeaddrinfo(res);
    return fd;
}

int connect_to(const char *host, const char *port) {
    struct addrinfo hints = {.ai_family = AF_UNSPEC, .ai_socktype = SOCK_STREAM};
    struct addrinfo *res;
    if (getaddrinfo(host, port, &hints, &res) != 0)
        return -1;
    int fd = -1;
    for (struct addrinfo *ai = res; ai != NULL; ai = ai->ai_next) {  /* try each address */
        fd = socket(ai->ai_family, ai->ai_socktype | SOCK_CLOEXEC, 0);
        if (fd < 0)
            continue;
        if (connect(fd, ai->ai_addr, ai->ai_addrlen) == 0)
            break;
        close(fd);
        fd = -1;
    }
    freeaddrinfo(res);
    return fd;
}

int write_all(int fd, const void *buf, size_t n) {
    const char *p = buf;
    while (n > 0) {
        ssize_t w = write(fd, p, n);
        if (w < 0) {
            if (errno == EINTR)
                continue;
            return -1;
        }
        p += w;
        n -= (size_t)w;
    }
    return 0;
}
```

```c
// file: echo/echo_server.c
/* An iterative echo server: handles one client at a time. */
#include "net.h"

#include <signal.h>
#include <stdio.h>
#include <sys/socket.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    const char *port = argc > 1 ? argv[1] : "9000";
    signal(SIGPIPE, SIG_IGN);                    /* a vanished client must not kill us */
    int lfd = listen_on(port);
    if (lfd < 0) {
        perror("listen_on");
        return 1;
    }
    fprintf(stderr, "echo server on port %s\n", port);
    for (int served = 0; served < 3; served++) { /* serve 3 clients, then exit (for the demo) */
        int cfd = accept4(lfd, NULL, NULL, SOCK_CLOEXEC);
        if (cfd < 0) {
            perror("accept");
            continue;
        }
        char buf[4096];
        ssize_t n;
        long total = 0;
        while ((n = read(cfd, buf, sizeof buf)) > 0) {
            total += n;
            if (write_all(cfd, buf, (size_t)n) != 0)
                break;
        }
        fprintf(stderr, "client %d done: echoed %ld bytes\n", served + 1, total);
        close(cfd);
    }
    close(lfd);
    return 0;
}
```

```c
// file: echo/echo_client.c
#include "net.h"

#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    if (argc != 4) {
        fprintf(stderr, "usage: echo_client HOST PORT MESSAGE\n");
        return 2;
    }
    int fd = connect_to(argv[1], argv[2]);
    if (fd < 0) {
        perror("connect_to");
        return 1;
    }
    size_t len = strlen(argv[3]);
    if (write_all(fd, argv[3], len) != 0)
        return 1;
    shutdown(fd, SHUT_WR);                       /* tell the server we are done sending */
    char buf[4096];
    size_t got = 0;
    ssize_t n;
    while ((n = read(fd, buf + got, sizeof buf - 1 - got)) > 0)   /* read until the server closes */
        got += (size_t)n;
    buf[got] = '\0';
    printf("sent %zu bytes, received back: \"%s\"\n", len, buf);
    close(fd);
    return 0;
}
```

```console
$ cd echo && gcc -std=c17 -Wall -Wextra -g net.c echo_server.c -o echo_server && gcc -std=c17 -Wall -Wextra -g net.c echo_client.c -o echo_client
$ cd echo && (./echo_server 9123 2> server.log &) && sleep 0.3 && ./echo_client localhost 9123 "hello, network" && ./echo_client 127.0.0.1 9123 "over IPv4" && ./echo_client localhost 9123 "third client" && sleep 0.2 && cat server.log
sent 14 bytes, received back: "hello, network"
sent 9 bytes, received back: "over IPv4"
sent 12 bytes, received back: "third client"
echo server on port 9123
client 1 done: echoed 14 bytes
client 2 done: echoed 9 bytes
client 3 done: echoed 12 bytes
```

Key details:

- **`accept` returns a new descriptor** per connection; the listening socket keeps listening.
- **`SO_REUSEADDR`** lets the server restart immediately; without it, `bind` fails with "Address already in use" for about a minute after a restart, while old connections linger in the `TIME_WAIT` state.
- **`shutdown(fd, SHUT_WR)`** sends end-of-file to the peer while keeping the socket open for reading: the client says "I'm done sending" and still receives the echo. `close` would discard incoming data.
- **`SIGPIPE`** is ignored: writing to a connection the peer has closed then fails with `EPIPE` instead of killing the server (Unit 8.4).
- `listen_on` tries each passive address `getaddrinfo` returns. On a host with IPv6, one IPv6 socket with `IPV6_V6ONLY` turned off accepts both IPv6 and IPv4 clients (IPv4 clients appear as `::ffff:127.0.0.1`). The machine used for this book has IPv6 disabled, so `socket(AF_INET6, …)` fails and the loop falls back to IPv4 — the first version of `listen_on`, which tried only IPv6, could not start at all there. Robust network code never assumes which address families exist.

This server is **iterative**: while one client is connected, others wait in the kernel's listen queue. Unit 10.4 fixes that.

### Mastery check

1. **Explain.** What would the client receive if it called `close` instead of `shutdown(fd, SHUT_WR)` before reading?

## Unit 10.3 — Framing messages on a byte stream

**One-line summary:** TCP delivers a stream of bytes, not messages: one `write` of 100 bytes may arrive as several reads, and several writes may arrive in one read. Every protocol needs **framing** — a way to find message boundaries — typically a **delimiter** (a newline, as in HTTP headers, SMTP, Redis's text protocol) or a **length prefix** (most binary protocols). Parsers must accumulate bytes in a buffer until a full message is present.

```c
// file: framing.c
#include <stdint.h>
#include <stdio.h>
#include <string.h>

/* A receive buffer that accumulates bytes and extracts length-prefixed frames:
   4-byte big-endian length, then that many bytes of payload. */
typedef struct {
    unsigned char data[256];
    size_t len;
} RecvBuf;

enum { MAX_FRAME = 200 };

/* feed appends newly received bytes, as read() would deliver them. */
static int feed(RecvBuf *b, const unsigned char *p, size_t n) {
    if (n > sizeof b->data - b->len)
        return -1;                                /* would overflow: protocol violation */
    memcpy(b->data + b->len, p, n);
    b->len += n;
    return 0;
}

/* next_frame returns 1 and copies a complete frame out, 0 if more bytes are
   needed, -1 if the declared length is invalid. */
static int next_frame(RecvBuf *b, char *out, size_t outsz) {
    if (b->len < 4)
        return 0;
    uint32_t n = (uint32_t)b->data[0] << 24 | (uint32_t)b->data[1] << 16 |
                 (uint32_t)b->data[2] << 8 | b->data[3];
    if (n > MAX_FRAME || n >= outsz)
        return -1;                                /* never trust a length from the network */
    if (b->len < 4 + n)
        return 0;
    memcpy(out, b->data + 4, n);
    out[n] = '\0';
    memmove(b->data, b->data + 4 + n, b->len - 4 - n);   /* keep any following bytes */
    b->len -= 4 + n;
    return 1;
}

static size_t encode(unsigned char *dst, const char *msg) {
    uint32_t n = (uint32_t)strlen(msg);
    dst[0] = (unsigned char)(n >> 24); dst[1] = (unsigned char)(n >> 16);
    dst[2] = (unsigned char)(n >> 8);  dst[3] = (unsigned char)n;
    memcpy(dst + 4, msg, n);
    return 4 + n;
}

int main(void) {
    unsigned char wire[128];
    size_t w = 0;
    w += encode(wire + w, "first");
    w += encode(wire + w, "second message");
    w += encode(wire + w, "3");

    /* deliver the stream in awkward pieces, as TCP may: 3, 7, 1, then the rest */
    size_t cuts[] = {3, 7, 1, w - 11};
    RecvBuf rb = {0};
    size_t off = 0;
    char msg[MAX_FRAME + 1];
    for (size_t i = 0; i < 4; i++) {
        feed(&rb, wire + off, cuts[i]);
        off += cuts[i];
        printf("delivered %zu bytes (buffered %zu):", cuts[i], rb.len);
        int r;
        while ((r = next_frame(&rb, msg, sizeof msg)) == 1)
            printf(" [%s]", msg);
        printf("\n");
    }
    unsigned char evil[4] = {0x7F, 0xFF, 0xFF, 0xFF};        /* claims a 2 GB message */
    RecvBuf eb = {0};
    feed(&eb, evil, 4);
    printf("hostile length prefix -> %d\n", next_frame(&eb, msg, sizeof msg));
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra -fsanitize=address,undefined framing.c -o framing && ./framing
delivered 3 bytes (buffered 3):
delivered 7 bytes (buffered 10): [first]
delivered 1 bytes (buffered 2):
delivered 21 bytes (buffered 23): [second message] [3]
hostile length prefix -> -1
```

The first two deliveries completed no frame; the third completed "first"; the last delivered the remaining two at once. A parser that assumes "one read = one message" fails exactly like this under real network conditions, usually only in production. And the **hostile length** check matters: a server that `malloc`s whatever length a client declares can be made to exhaust memory with one packet.

### Pattern

**P-10. Per-connection receive buffer + incremental parser + hard limits.** *(Practice)* Each connection owns a buffer; every `read` appends to it; a parser extracts as many complete messages as are present and keeps the remainder; every length, count, and line is checked against a fixed maximum before any allocation. *Trade-off:* more code than "read a message", in exchange for correctness under any network timing and resistance to resource-exhaustion attacks.

### Mastery check

1. **Write.** The same `RecvBuf` for newline-delimited messages, rejecting lines over 1 KiB.

## Unit 10.4 — UDP

**One-line summary:** UDP sends independent **datagrams**: each `sendto` is delivered as one unit by one `recvfrom`, or not at all. No connection, no ordering, no retransmission, no flow control — but no head-of-line blocking and minimal latency. DNS, games, VoIP, video streaming, and QUIC (HTTP/3) are built on it.

```c
// file: udp.c
#define _POSIX_C_SOURCE 200809L
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

int main(void) {
    int srv = socket(AF_INET, SOCK_DGRAM, 0), cli = socket(AF_INET, SOCK_DGRAM, 0);
    struct sockaddr_in addr = {.sin_family = AF_INET, .sin_port = htons(0)};   /* port 0: any free port */
    inet_pton(AF_INET, "127.0.0.1", &addr.sin_addr);
    if (bind(srv, (struct sockaddr *)&addr, sizeof addr) != 0)
        return 1;
    socklen_t alen = sizeof addr;
    getsockname(srv, (struct sockaddr *)&addr, &alen);          /* which port did we get? */

    const char *msgs[] = {"ping 1", "ping 2", "a much longer third datagram"};
    for (int i = 0; i < 3; i++)
        sendto(cli, msgs[i], strlen(msgs[i]), 0, (struct sockaddr *)&addr, sizeof addr);

    for (int i = 0; i < 3; i++) {
        char buf[16];                                           /* deliberately small */
        struct sockaddr_in from;
        socklen_t flen = sizeof from;
        ssize_t n = recvfrom(srv, buf, sizeof buf - 1, MSG_TRUNC, (struct sockaddr *)&from, &flen);
        size_t kept = (size_t)n < sizeof buf - 1 ? (size_t)n : sizeof buf - 1;
        buf[kept] = '\0';
        printf("datagram of %zd bytes from port %u: \"%s\"%s\n", n, ntohs(from.sin_port), buf,
               (size_t)n > kept ? " (truncated: the rest is lost)" : "");
    }
    close(srv);
    close(cli);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra udp.c -o udp && ./udp | sed 's/port [0-9]*/port N/'
datagram of 6 bytes from port N: "ping 1"
datagram of 6 bytes from port N: "ping 2"
datagram of 28 bytes from port N: "a much longer t" (truncated: the rest is lost)
```

Each datagram arrived whole and separately — message boundaries are preserved, unlike TCP. But a datagram larger than the receive buffer is **truncated**, and the rest is discarded (`MSG_TRUNC` makes `recvfrom` report the full original length). Over a real network, datagrams can also be lost, duplicated, or reordered; applications add sequence numbers, acknowledgements, and timeouts as needed. Keep datagrams under about 1,200 bytes to avoid IP fragmentation on the internet.

<!-- expanded -->
### Deep & Tricky

**DT-80. A UDP datagram larger than the receive buffer is truncated, and the rest is discarded silently: `recvfrom` returns the buffer size, not an error, unless you ask with `MSG_TRUNC`, which makes Linux return the datagram's real length.** *(POSIX `recvfrom`: "If a message is too long to fit in the supplied buffer, excess bytes shall be discarded"; Linux `udp(7)`: with `MSG_TRUNC`, "return the real length of the packet, even when it was longer than the passed buffer")* A protocol on UDP should define a maximum datagram size and receive into a buffer one byte larger, so that "the buffer filled up" can be detected.

```c
// file: udptrunc.c
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

int main(void) {
    int r = socket(AF_INET, SOCK_DGRAM, 0), s = socket(AF_INET, SOCK_DGRAM, 0);
    struct sockaddr_in a = { .sin_family = AF_INET, .sin_addr.s_addr = htonl(INADDR_LOOPBACK) };
    socklen_t len = sizeof a;
    bind(r, (struct sockaddr *)&a, sizeof a);                  /* port 0: the kernel picks one */
    getsockname(r, (struct sockaddr *)&a, &len);

    char msg[100];
    memset(msg, 'x', sizeof msg);
    sendto(s, msg, sizeof msg, 0, (struct sockaddr *)&a, sizeof a);
    sendto(s, msg, sizeof msg, 0, (struct sockaddr *)&a, sizeof a);

    char buf[40];
    ssize_t n = recv(r, buf, sizeof buf, 0);
    printf("100-byte datagram, 40-byte buffer: recv returned %zd\n", n);
    n = recv(r, buf, sizeof buf, MSG_TRUNC);
    printf("same again with MSG_TRUNC: recv returned %zd (the real size; 40 bytes copied)\n", n);
    n = recv(r, buf, sizeof buf, MSG_DONTWAIT);
    printf("next recv: %zd - the other 60 bytes of each datagram are gone\n", n);
    close(r);
    close(s);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra udptrunc.c -o udptrunc && ./udptrunc
100-byte datagram, 40-byte buffer: recv returned 40
same again with MSG_TRUNC: recv returned 100 (the real size; 40 bytes copied)
next recv: -1 - the other 60 bytes of each datagram are gone
```

### Mastery check

1. **Explain.** Why do DNS queries use UDP while web pages use TCP (or QUIC)?

## Topic 10B — Serving many clients

## Unit 10.5 — Concurrency models for servers

**One-line summary:** Three ways to serve many clients: **process per connection** (`fork` after `accept`: strong isolation, heavyweight), **thread per connection** (simple blocking code, but each thread costs memory and scheduling, so thousands of connections hurt), and **event-driven** (one thread, non-blocking sockets, and a readiness API like `epoll` that tells you which connections can make progress — scales to tens of thousands of connections). Production servers combine them: an event loop per core.

| Model | Code style | Memory per connection | Scales to | Isolation | Examples |
| --- | --- | --- | --- | --- | --- |
| Process per connection | blocking, simple | process (MBs, shared copy-on-write) | hundreds | strong | classic Apache prefork, PostgreSQL backends, OpenSSH |
| Thread per connection | blocking, simple | stack (default 8 MB virtual, ~tens of KB used) | thousands | none | many Java servers, MySQL |
| Event loop (`epoll`) | non-blocking, state machines | a few KB of buffers | 10,000s+ | none | nginx, Redis, HAProxy, Node.js |
| Event loop per core | as above, ×N threads | as above | 100,000s | none | nginx (worker processes), Envoy |

The cost of the event-driven model is code structure: a handler cannot block waiting for the rest of a request, so each connection's progress must be kept in an explicit **state machine** (how much has been read, what is being written, how much is left). The next unit builds one.

<!-- expanded -->
### Deep & Tricky

**DT-81. Writing to a TCP socket whose peer has closed the connection raises `SIGPIPE`, and the default action of `SIGPIPE` is to terminate the process — so one client that disconnects at the wrong moment can kill a whole server that never installed a handler.** *(POSIX `write`/`send`: "A write was attempted on a socket that is shut down for writing, or is no longer connected. In the latter case, if the socket is of type SOCK_STREAM, a SIGPIPE signal shall also be sent to the thread")* Servers either ignore the signal process-wide (`signal(SIGPIPE, SIG_IGN)`, after which the write fails with `EPIPE`) or pass `MSG_NOSIGNAL` to every `send`. The first write after the peer closes often still succeeds (the data is queued, the peer answers with a reset); the second one triggers the signal.

```c
// file: sigpipe.c
#define _DEFAULT_SOURCE
#include <arpa/inet.h>
#include <errno.h>
#include <signal.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

static void try(int ignore) {
    int ls = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in a = { .sin_family = AF_INET, .sin_addr.s_addr = htonl(INADDR_LOOPBACK) };
    socklen_t len = sizeof a;
    bind(ls, (struct sockaddr *)&a, sizeof a);
    listen(ls, 1);
    getsockname(ls, (struct sockaddr *)&a, &len);
    int c = socket(AF_INET, SOCK_STREAM, 0);
    connect(c, (struct sockaddr *)&a, sizeof a);
    int s = accept(ls, NULL, NULL);
    close(c);                                          /* the client goes away */
    usleep(100000);
    if (ignore) signal(SIGPIPE, SIG_IGN);
    for (int i = 1; i <= 3; i++) {
        ssize_t n = write(s, "data", 4);
        printf("  write %d: %zd%s%s\n", i, n, n < 0 ? ", errno " : "", n < 0 ? strerror(errno) : "");
        fflush(stdout);
        usleep(100000);
    }
    close(s);
    close(ls);
}

int main(int argc, char **argv) {
    (void)argv;
    printf(argc > 1 ? "with SIGPIPE ignored:\n" : "default SIGPIPE action:\n");
    fflush(stdout);
    try(argc > 1);
    printf("server still alive\n");
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra sigpipe.c -o sigpipe && ./sigpipe; echo "exit status $?"; ./sigpipe ignore
default SIGPIPE action:
  write 1: 4
exit status 141
with SIGPIPE ignored:
  write 1: 4
  write 2: -1, errno Broken pipe
  write 3: -1, errno Broken pipe
server still alive
```

The first `write` "succeeded" — the data was accepted into the send buffer and the peer's kernel answered with a reset; the second killed the process (exit status 141 = 128 + 13, `SIGPIPE`). With the signal ignored, the same write returned -1 with `EPIPE` and the server lived on.

## Unit 10.6 — Non-blocking I/O and `epoll`

**One-line summary:** A **non-blocking** socket returns `EAGAIN` instead of waiting when no data is available or the send buffer is full. `epoll` lets one thread register thousands of descriptors and sleep until some are ready. **Level-triggered** mode reports a descriptor as long as it stays ready; **edge-triggered** mode reports only changes, so you must read or write until `EAGAIN` each time.

The core loop of every `epoll` server:

```c
int ep = epoll_create1(EPOLL_CLOEXEC);
add(ep, listen_fd, EPOLLIN);
for (;;) {
    struct epoll_event ev[64];
    int n = epoll_wait(ep, ev, 64, timeout_ms);       /* sleep until something is ready */
    for (int i = 0; i < n; i++) {
        if (ev[i].data.fd == listen_fd)
            accept_all_pending();                    /* accept until EAGAIN; set O_NONBLOCK */
        else if (ev[i].events & EPOLLIN)
            on_readable(conn_of(ev[i]));             /* read until EAGAIN, parse, maybe respond */
        else if (ev[i].events & EPOLLOUT)
            on_writable(conn_of(ev[i]));             /* continue a partially written response */
    }
}
```

Rules for correct event loops:

- Set every socket non-blocking (`SOCK_NONBLOCK` in `socket`/`accept4`, or `fcntl` with `O_NONBLOCK`), so no call can stall the whole loop.
- With edge-triggered `EPOLLET`, loop on `read`/`accept` until `EAGAIN`; otherwise data that arrived with the same edge stays unread and no further event comes. (Level-triggered is more forgiving; this chapter's project uses it.)
- Register for `EPOLLOUT` only while you have unsent data; a writable socket is almost always writable, and leaving `EPOLLOUT` on spins the loop.
- Keep per-connection state in a struct reachable from `ev.data.ptr`.
- Never do slow work (disk I/O on a cold file, DNS lookups, heavy computation) in the loop thread; hand it to a thread pool (Chapter 9) and get the result back through a pipe or `eventfd`.

`poll` and `select` are the portable predecessors (O(n) per call in the number of descriptors); BSD and macOS have `kqueue`; Linux 5.1+ also has `io_uring`, which submits the I/O operations themselves asynchronously.

### Deep & Tricky

**DT-82. One slow operation in an event loop delays every client.** *(Design fact)* An event loop is cooperative multitasking: while a handler runs, no other connection makes progress. A 50 ms `stat` on a network file system or an accidental blocking `read` adds 50 ms to every request in flight. Measure the loop's per-iteration time, and move blocking work off the loop.

**DT-83. Two small writes in a row can cost 40 milliseconds: Nagle's algorithm meets delayed ACK.** *(TCP behaviour, measured in this chapter's project)* TCP's Nagle algorithm holds back a small segment while an earlier one is unacknowledged; the receiver's **delayed ACK** waits up to about 40 ms hoping to piggyback the acknowledgement on a reply. A server that sends a response head and then, separately, a small body hits both: the body waits for an ACK that waits for a timeout. The first version of this chapter's HTTP server did exactly that and measured a mean latency of **43 ms** per request on localhost — and a single connection managed only 23 requests per second. The fixes: build the whole response in one buffer and send it with one call (or `writev`), and set `TCP_NODELAY` on sockets that carry request/response traffic.

### Mastery check

1. **Explain.** Why must an edge-triggered handler read until `EAGAIN`?

## Unit 10.7 — HTTP/1.1 in brief

**One-line summary:** An HTTP/1.1 request is a request line (`GET /path HTTP/1.1`), header lines, an empty line (`\r\n\r\n` ends the head), and an optional body sized by `Content-Length` (or chunked encoding). A response has a status line, headers, an empty line, and a body. Connections are **persistent** by default (keep-alive). A server parsing HTTP must enforce limits and reject anything ambiguous.

```
GET /index.html HTTP/1.1\r\n            HTTP/1.1 200 OK\r\n
Host: localhost:8080\r\n                Content-Type: text/html\r\n
User-Agent: curl/8.5.0\r\n              Content-Length: 128\r\n
Accept: */*\r\n                         \r\n
\r\n                                    <html>… 128 bytes …
```

Security rules for a minimal server:

- Limit the request head size (e.g. 8 KiB) and the number of headers; reject anything larger with `431` or `400`.
- Reject request lines that are not exactly three space-separated parts, unknown methods (`405` or `501`), and unsupported versions.
- **Path traversal:** never join a URL path to a directory without validation — `GET /../../etc/passwd` must not escape the document root. Reject `..` segments, or resolve with `realpath` and check the prefix.
- Set timeouts: a client that sends one byte per minute (Slowloris) must not hold a connection forever.
- Ambiguity between `Content-Length` and `Transfer-Encoding` is how request-smuggling attacks work; a server that doesn't support request bodies should reject them.

<!-- expanded -->
## Unit 10.8 — `poll`: waiting for many descriptors portably

**One-line summary:** `poll` takes an array of `struct pollfd` — a descriptor plus the events of interest (`POLLIN`, `POLLOUT`) — sleeps until at least one is ready or a timeout expires, and reports what happened in each entry's `revents`. It works on every POSIX system and has no limit on descriptor numbers, which makes it the portable middle ground between `select` (ancient, limited) and `epoll`/`kqueue` (scalable, system-specific). Its cost is O(n) per call in the number of descriptors watched.

```c
// file: polldemo.c
#define _DEFAULT_SOURCE
#include <poll.h>
#include <stdio.h>
#include <string.h>
#include <sys/wait.h>
#include <unistd.h>

int main(void) {
    enum { N = 3 };
    struct pollfd pfd[N];
    for (int i = 0; i < N; i++) {                         /* N children, each with its own pipe */
        int fd[2];
        if (pipe(fd) < 0) return 1;
        if (fork() == 0) {
            close(fd[0]);
            usleep((unsigned)(N - i) * 150000);           /* child 2 speaks first, child 0 last */
            char msg[32];
            int n = snprintf(msg, sizeof msg, "hello from child %d", i);
            if (write(fd[1], msg, (size_t)n) < 0) _exit(1);
            _exit(0);                                      /* closing the pipe: the reader will see POLLHUP */
        }
        close(fd[1]);
        pfd[i] = (struct pollfd){ .fd = fd[0], .events = POLLIN };
    }
    int open_pipes = N;
    while (open_pipes > 0) {
        int ready = poll(pfd, N, 2000);                   /* timeout in milliseconds; -1 = forever */
        if (ready == 0) { printf("timeout\n"); break; }
        for (int i = 0; i < N; i++) {
            if (pfd[i].revents & POLLIN) {
                char buf[64];
                ssize_t n = read(pfd[i].fd, buf, sizeof buf - 1);
                if (n > 0) { buf[n] = '\0'; printf("pipe %d readable: \"%s\"\n", i, buf); continue; }
            }
            if (pfd[i].revents & (POLLHUP | POLLERR)) {
                printf("pipe %d closed\n", i);
                close(pfd[i].fd);
                pfd[i].fd = -1;                           /* negative fd: poll ignores the entry */
                open_pipes--;
            }
        }
    }
    while (wait(NULL) > 0) { }
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra polldemo.c -o polldemo && ./polldemo
pipe 2 readable: "hello from child 2"
pipe 2 closed
pipe 1 readable: "hello from child 1"
pipe 1 closed
pipe 0 readable: "hello from child 0"
pipe 0 closed
```

One thread, three sources, handled in the order they became ready. Setting an entry's `fd` to -1 is the idiomatic way to stop watching it without compacting the array. Note that the data and the hang-up of the same pipe may be reported in one `poll` call or in two; the loop handles both.

### Deep & Tricky

**DT-84. `select` can only watch descriptors numbered below `FD_SETSIZE` (1024 on Linux), and `FD_SET` with a larger descriptor writes past the end of the `fd_set` — memory corruption, not an error. A long-running server that opens more than about a thousand files or sockets crosses that line without noticing.** *(POSIX `select`: "The behavior of these macros is undefined if the fd argument is less than 0 or greater than or equal to FD_SETSIZE"; glibc's `_FORTIFY_SOURCE` turns it into an abort: "buffer overflow detected")* This is the main reason new code should use `poll` (or `epoll`) instead of `select`, even for a handful of descriptors: the number of descriptors you watch may be small, but their *values* grow with everything else the process has open.

### Mastery check

1. **Predict.** A `poll` entry's `events` is `POLLIN`, and the peer closes a TCP connection after sending 10 bytes. What `revents` may the next `poll` report, and what do the next two `read` calls return?
2. **Write.** Rewrite Unit 10.2's echo server to serve several clients at once with `poll`, adding each accepted socket to the array and removing it on end of file.

## Chapter 10 Project — An event-driven HTTP server

**Goal:** build `evhttpd`, a single-threaded, `epoll`-based HTTP/1.1 static file server with keep-alive, request limits, path-traversal protection, idle timeouts, and correct handling of partial reads and writes — then test it with `curl` and load-test it with a small client of your own.

### Specification

- `evhttpd PORT DOCROOT` serves `GET` and `HEAD` for files under `DOCROOT`; `/` maps to `/index.html`.
- Responses: `200` with `Content-Type` (by extension) and `Content-Length`; `400` malformed; `403` traversal attempts; `404` missing; `405` other methods; `431` oversized head.
- Keep-alive by default; `Connection: close` honoured; idle connections closed after 10 seconds.
- Never blocks on a socket; large files are sent in pieces as the socket becomes writable.

### Reference solution

```c
// file: evhttpd/evhttpd.c
/* evhttpd: a single-threaded epoll HTTP/1.1 static file server. */
#define _GNU_SOURCE                       /* accept4, memmem */
#include <ctype.h>
#include <errno.h>
#include <fcntl.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <signal.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <strings.h>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>

enum { HEAD_MAX = 8192, OUT_MAX = 65536, IDLE_SECONDS = 10, MAX_EVENTS = 128 };

typedef struct Conn {
    int fd;
    char in[HEAD_MAX];          /* request head being received */
    size_t in_len;
    char out[OUT_MAX];          /* response bytes waiting to be sent */
    size_t out_len, out_off;
    int file;                   /* file being streamed after the head, or -1 */
    off_t file_left;
    bool close_after;
    time_t last_active;
    struct Conn *next;          /* list of all connections, for idle sweeps */
} Conn;

static int ep;
static const char *docroot;
static Conn *conns;
static long requests_served;

static const char *mime(const char *path) {
    const char *dot = strrchr(path, '.');
    if (dot == NULL) return "application/octet-stream";
    if (strcmp(dot, ".html") == 0) return "text/html; charset=utf-8";
    if (strcmp(dot, ".txt") == 0) return "text/plain; charset=utf-8";
    if (strcmp(dot, ".css") == 0) return "text/css";
    if (strcmp(dot, ".js") == 0) return "text/javascript";
    if (strcmp(dot, ".png") == 0) return "image/png";
    if (strcmp(dot, ".json") == 0) return "application/json";
    return "application/octet-stream";
}

static void set_events(Conn *c, uint32_t events) {
    struct epoll_event ev = {.events = events, .data.ptr = c};
    epoll_ctl(ep, EPOLL_CTL_MOD, c->fd, &ev);
}

static void conn_close(Conn *c) {
    epoll_ctl(ep, EPOLL_CTL_DEL, c->fd, NULL);
    close(c->fd);
    if (c->file >= 0)
        close(c->file);
    for (Conn **p = &conns; *p; p = &(*p)->next)       /* unlink: pointer-to-pointer, Unit 7.2 */
        if (*p == c) { *p = c->next; break; }
    free(c);
}

static void respond_simple(Conn *c, int code, const char *reason, bool keep) {
    char body[128];
    int blen = snprintf(body, sizeof body, "%d %s\n", code, reason);
    c->out_len = (size_t)snprintf(c->out, sizeof c->out,
        "HTTP/1.1 %d %s\r\nContent-Type: text/plain\r\nContent-Length: %d\r\n%s\r\n%s",
        code, reason, blen, keep ? "" : "Connection: close\r\n", body);
    c->out_off = 0;
    c->close_after = !keep;
}

/* safe_path rejects traversal: the URL path must start with '/' and contain no ".." segment. */
static bool safe_path(const char *p) {
    if (p[0] != '/')
        return false;
    for (const char *s = p; *s; s++)
        if (s[0] == '.' && s[1] == '.' && (s == p || s[-1] == '/') && (s[2] == '/' || s[2] == '\0'))
            return false;
    return true;
}

/* handle_request parses the complete head in c->in[0..head_len) and prepares a response. */
static void handle_request(Conn *c, size_t head_len) {
    char *line_end = memmem(c->in, head_len, "\r\n", 2);
    char method[8], path[1024], version[16];
    bool keep = true;
    *line_end = '\0';
    if (sscanf(c->in, "%7s %1023s %15s", method, path, version) != 3 ||
        strncmp(version, "HTTP/1.", 7) != 0) {
        respond_simple(c, 400, "Bad Request", false);
        return;
    }
    if (strcmp(version, "HTTP/1.0") == 0)
        keep = false;                                     /* 1.0 closes by default */
    for (char *h = line_end + 2; h < c->in + head_len;) { /* scan headers for Connection */
        char *e = memmem(h, (size_t)(c->in + head_len - h), "\r\n", 2);
        if (e == NULL) break;
        *e = '\0';
        if (strncasecmp(h, "Connection:", 11) == 0) {
            const char *v = h + 11;
            while (*v == ' ') v++;
            if (strcasecmp(v, "close") == 0) keep = false;
            if (strcasecmp(v, "keep-alive") == 0) keep = true;
        }
        h = e + 2;
    }
    bool head_only = strcmp(method, "HEAD") == 0;
    if (!head_only && strcmp(method, "GET") != 0) {
        respond_simple(c, 405, "Method Not Allowed", keep);
        return;
    }
    char *q = strchr(path, '?');
    if (q) *q = '\0';                                     /* ignore the query string */
    if (!safe_path(path)) {
        respond_simple(c, 403, "Forbidden", keep);
        return;
    }
    char full[2048];
    snprintf(full, sizeof full, "%s%s", docroot, strcmp(path, "/") == 0 ? "/index.html" : path);
    int fd = open(full, O_RDONLY | O_CLOEXEC);
    struct stat st;
    if (fd < 0 || fstat(fd, &st) != 0 || !S_ISREG(st.st_mode)) {
        if (fd >= 0) close(fd);
        respond_simple(c, 404, "Not Found", keep);
        return;
    }
    c->out_len = (size_t)snprintf(c->out, sizeof c->out,
        "HTTP/1.1 200 OK\r\nContent-Type: %s\r\nContent-Length: %jd\r\n%s\r\n",
        mime(full), (intmax_t)st.st_size, keep ? "" : "Connection: close\r\n");
    c->out_off = 0;
    c->close_after = !keep;
    c->file_left = head_only ? 0 : st.st_size;
    /* put the first part of the body in the same buffer as the head: one send for
       small files, and no Nagle/delayed-ACK stall between head and body (DT-83) */
    while (c->file_left > 0 && c->out_len < sizeof c->out) {
        ssize_t r = read(fd, c->out + c->out_len, sizeof c->out - c->out_len);
        if (r <= 0)
            break;
        c->out_len += (size_t)r;
        c->file_left -= r;
    }
    if (c->file_left > 0)
        c->file = fd;                                     /* stream the rest from pump() */
    else
        close(fd);
    requests_served++;
}

/* pump writes as much pending output as the socket accepts. Returns false if the
   connection was closed. */
static bool pump(Conn *c) {
    for (;;) {
        if (c->out_off == c->out_len && c->file >= 0 && c->file_left > 0) {
            ssize_t r = read(c->file, c->out, sizeof c->out);   /* refill from the file */
            if (r <= 0) { conn_close(c); return false; }
            c->out_len = (size_t)r;
            c->out_off = 0;
            c->file_left -= r;
            if (c->file_left == 0) { close(c->file); c->file = -1; }
        }
        if (c->out_off == c->out_len)
            break;                                          /* response complete */
        ssize_t w = send(c->fd, c->out + c->out_off, c->out_len - c->out_off, MSG_NOSIGNAL);
        if (w < 0) {
            if (errno == EAGAIN || errno == EWOULDBLOCK) {
                set_events(c, EPOLLOUT);                    /* wait until writable */
                return true;
            }
            conn_close(c);
            return false;
        }
        c->out_off += (size_t)w;
    }
    c->out_len = c->out_off = 0;
    if (c->close_after) {
        conn_close(c);
        return false;
    }
    set_events(c, EPOLLIN);                                 /* ready for the next request */
    return true;
}

static void on_readable(Conn *c) {
    for (;;) {
        ssize_t n = recv(c->fd, c->in + c->in_len, sizeof c->in - c->in_len, 0);
        if (n == 0) { conn_close(c); return; }              /* client closed */
        if (n < 0) {
            if (errno == EAGAIN || errno == EWOULDBLOCK) break;
            conn_close(c);
            return;
        }
        c->in_len += (size_t)n;
        c->last_active = time(NULL);
        if (c->in_len == sizeof c->in) break;
    }
    /* answer every complete request in the buffer (pipelining), one at a time */
    while (c->out_len == 0) {
        char *end = memmem(c->in, c->in_len, "\r\n\r\n", 4);
        if (end == NULL) {
            if (c->in_len == sizeof c->in) {                /* head too large */
                respond_simple(c, 431, "Request Header Fields Too Large", false);
                c->in_len = 0;
                pump(c);
            }
            return;
        }
        size_t head_len = (size_t)(end - c->in) + 4;
        handle_request(c, head_len);
        memmove(c->in, c->in + head_len, c->in_len - head_len);
        c->in_len -= head_len;
        if (!pump(c))
            return;
    }
}

static void on_accept(int lfd) {
    for (;;) {
        int fd = accept4(lfd, NULL, NULL, SOCK_NONBLOCK | SOCK_CLOEXEC);
        if (fd < 0)
            return;                                          /* EAGAIN: no more pending */
        Conn *c = calloc(1, sizeof *c);
        if (c == NULL) { close(fd); continue; }
        int one = 1;
        setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof one);   /* see DT-83 */
        c->fd = fd;
        c->file = -1;
        c->last_active = time(NULL);
        c->next = conns;
        conns = c;
        struct epoll_event ev = {.events = EPOLLIN, .data.ptr = c};
        epoll_ctl(ep, EPOLL_CTL_ADD, fd, &ev);
    }
}

static volatile sig_atomic_t stop;
static void on_term(int s) { (void)s; stop = 1; }

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "usage: evhttpd PORT DOCROOT\n");
        return 2;
    }
    docroot = argv[2];
    signal(SIGPIPE, SIG_IGN);
    struct sigaction sa = {.sa_handler = on_term};
    sigaction(SIGTERM, &sa, NULL);
    sigaction(SIGINT, &sa, NULL);

    int lfd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK | SOCK_CLOEXEC, 0), on = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof on);
    struct sockaddr_in addr = {.sin_family = AF_INET, .sin_port = htons((uint16_t)atoi(argv[1])),
                               .sin_addr.s_addr = htonl(INADDR_LOOPBACK)};
    if (bind(lfd, (struct sockaddr *)&addr, sizeof addr) != 0 || listen(lfd, SOMAXCONN) != 0) {
        perror("bind/listen");
        return 1;
    }
    ep = epoll_create1(EPOLL_CLOEXEC);
    struct epoll_event lev = {.events = EPOLLIN, .data.ptr = NULL};     /* NULL marks the listener */
    epoll_ctl(ep, EPOLL_CTL_ADD, lfd, &lev);
    fprintf(stderr, "evhttpd: serving %s on 127.0.0.1:%s\n", docroot, argv[1]);

    while (!stop) {
        struct epoll_event ev[MAX_EVENTS];
        int n = epoll_wait(ep, ev, MAX_EVENTS, 1000);
        for (int i = 0; i < n; i++) {
            Conn *c = ev[i].data.ptr;
            if (c == NULL)
                on_accept(lfd);
            else if (ev[i].events & (EPOLLERR | EPOLLHUP))
                conn_close(c);
            else if (ev[i].events & EPOLLOUT)
                pump(c);
            else if (ev[i].events & EPOLLIN)
                on_readable(c);
        }
        time_t now = time(NULL);                            /* idle sweep */
        for (Conn *c = conns, *next; c != NULL; c = next) {
            next = c->next;
            if (now - c->last_active > IDLE_SECONDS && c->out_len == 0)
                conn_close(c);
        }
    }
    while (conns)
        conn_close(conns);
    close(lfd);
    close(ep);
    fprintf(stderr, "evhttpd: shut down after %ld requests\n", requests_served);
    return 0;
}
```

Notable design decisions:

- **One struct per connection** holds the whole state machine: bytes received so far, bytes waiting to be sent, and an open file being streamed. Large files never sit in memory; each `EPOLLOUT` event sends the next 64 KiB.
- **Pipelined requests** in one read are answered one at a time: the loop parses the next request only after the previous response is fully sent.
- **Idle sweep** once per loop iteration (at least once a second, thanks to the `epoll_wait` timeout) closes connections silent for 10 seconds — Slowloris protection.
- **Clean shutdown** on `SIGTERM`: the handler sets a flag; the loop notices within a second, closes everything, and reports.
- `send(…, MSG_NOSIGNAL)` avoids `SIGPIPE` per call (it is also ignored globally).

### Testing with `curl`

```html
<!-- file: evhttpd/www/index.html -->
<!doctype html><title>evhttpd</title><h1>It works</h1>
```

```console
$ cd evhttpd && gcc -std=c17 -Wall -Wextra -O2 -g -fsanitize=address,undefined evhttpd.c -o evhttpd_asan && head -c 3000000 /dev/urandom > www/big.bin && echo "plain text" > www/note.txt
$ cd evhttpd && (./evhttpd_asan 8471 www 2> asan.log & echo $! > asan.pid) && sleep 0.5 && curl -s -i http://127.0.0.1:8471/ | tr -d '\r' && curl -s -o /dev/null -w '%{http_code} %{size_download} bytes\n' http://127.0.0.1:8471/big.bin && cmp <(curl -s http://127.0.0.1:8471/big.bin) www/big.bin && echo "big.bin identical"
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 55

<!doctype html><title>evhttpd</title><h1>It works</h1>
200 3000000 bytes
big.bin identical
$ cd evhttpd && for u in /note.txt /missing /../evhttpd.c /%2e%2e/x; do printf '%-14s ' "$u"; curl -s --path-as-is -o /dev/null -w '%{http_code}\n' "http://127.0.0.1:8471$u"; done; curl -s -o /dev/null -w 'POST: %{http_code}\n' -X POST http://127.0.0.1:8471/
/note.txt      200
/missing       404
/../evhttpd.c  403
/%2e%2e/x      404
POST: 405
$ cd evhttpd && curl -s -o /dev/null -w 'oversized head: %{http_code}\n' -H "X-Big: $(head -c 9000 /dev/zero | tr '\0' a)" http://127.0.0.1:8471/
oversized head: 431
$ cd evhttpd && curl -s -v http://127.0.0.1:8471/note.txt http://127.0.0.1:8471/index.html 2>&1 | grep -E 'Re-using|Connected to|^< HTTP' | sed 's/#[0-9]*/#N/'
* Connected to 127.0.0.1 (127.0.0.1) port 8471
< HTTP/1.1 200 OK
* Re-using existing connection with host 127.0.0.1
< HTTP/1.1 200 OK
$ cd evhttpd && exec 3<>/dev/tcp/127.0.0.1/8471 && printf 'GET /note.txt HTTP/1.1\r\nHost: x\r\n\r\nGET /note.txt HTTP/1.1\r\nHost: x\r\nConnection: close\r\n\r\n' >&3 && cat <&3 | grep -cE '^HTTP/1.1 200'
2
$ cd evhttpd && kill -TERM $(cat asan.pid); sleep 1.5; cat asan.log
evhttpd: serving www on 127.0.0.1:8471
evhttpd: shut down after 8 requests
```

The server returned correct status codes for every case, streamed a 3 MB binary file byte-for-byte identically, rejected the raw `..` traversal with 403 (`--path-as-is` stops curl from normalising it first), reused one TCP connection for two requests (keep-alive), answered two pipelined requests sent in one write (using bash's `/dev/tcp` to send raw bytes), and rejected an oversized header with 431. The `%2e%2e` (URL-encoded dots) case returned 404, not 403: this server does not decode `%` escapes, so it looked for a file literally named `%2e%2e` — safe, but a server that *does* decode escapes must check for traversal **after** decoding. The sanitizer log shows a clean shutdown with no errors or leaks.

### Load testing

```c
// file: evhttpd/loadgen.c
/* loadgen: C concurrent keep-alive connections, each sending requests for D seconds. */
#define _GNU_SOURCE
#include <netinet/in.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <time.h>
#include <unistd.h>

static int port;
static double duration;
static const char *path;

static double now(void) {
    struct timespec ts;
    clock_gettime(CLOCK_MONOTONIC, &ts);
    return ts.tv_sec + ts.tv_nsec / 1e9;
}

typedef struct { long requests, errors; double lat_sum, lat_max; } Stats;

static void *client(void *arg) {
    Stats *s = arg;
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in a = {.sin_family = AF_INET, .sin_port = htons((uint16_t)port),
                            .sin_addr.s_addr = htonl(INADDR_LOOPBACK)};
    if (connect(fd, (struct sockaddr *)&a, sizeof a) != 0) { s->errors++; return NULL; }
    char req[256], buf[65536];
    int rlen = snprintf(req, sizeof req, "GET %s HTTP/1.1\r\nHost: x\r\n\r\n", path);
    double end = now() + duration;
    while (now() < end) {
        double t0 = now();
        if (write(fd, req, (size_t)rlen) != rlen) { s->errors++; break; }
        /* read the head, find Content-Length, then read exactly the body */
        size_t got = 0;
        char *hend = NULL;
        while ((hend = memmem(buf, got, "\r\n\r\n", 4)) == NULL) {
            ssize_t n = read(fd, buf + got, sizeof buf - got);
            if (n <= 0) { s->errors++; goto out; }
            got += (size_t)n;
        }
        char *cl = strcasestr(buf, "Content-Length:");
        long body = cl ? atol(cl + 15) : 0, have = (long)(got - (size_t)(hend + 4 - buf));
        while (have < body) {
            ssize_t n = read(fd, buf, sizeof buf);
            if (n <= 0) { s->errors++; goto out; }
            have += n;
        }
        double lat = now() - t0;
        s->requests++;
        s->lat_sum += lat;
        if (lat > s->lat_max) s->lat_max = lat;
    }
out:
    close(fd);
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 5) { fprintf(stderr, "usage: loadgen PORT PATH CONNECTIONS SECONDS\n"); return 2; }
    port = atoi(argv[1]);
    path = argv[2];
    int conns = atoi(argv[3]);
    duration = atof(argv[4]);
    pthread_t *t = calloc((size_t)conns, sizeof *t);
    Stats *st = calloc((size_t)conns, sizeof *st);
    for (int i = 0; i < conns; i++) pthread_create(&t[i], NULL, client, &st[i]);
    Stats total = {0};
    for (int i = 0; i < conns; i++) {
        pthread_join(t[i], NULL);
        total.requests += st[i].requests;
        total.errors += st[i].errors;
        total.lat_sum += st[i].lat_sum;
        if (st[i].lat_max > total.lat_max) total.lat_max = st[i].lat_max;
    }
    printf("%d connections, %.0fs: %ld requests (%.0f/s), %ld errors, mean latency %.3f ms, max %.1f ms\n",
           conns, duration, total.requests, total.requests / duration, total.errors,
           total.requests ? total.lat_sum / total.requests * 1e3 : 0, total.lat_max * 1e3);
    return 0;
}
```

```console
$ cd evhttpd && gcc -std=c17 -Wall -O2 evhttpd.c -o evhttpd && gcc -std=c17 -Wall -O2 -pthread loadgen.c -o loadgen
$ cd evhttpd && (./evhttpd 8472 www 2> run.log & echo $! > run.pid) && sleep 0.3 && ./loadgen 8472 /note.txt 1 3 && ./loadgen 8472 /note.txt 50 3 && ./loadgen 8472 /note.txt 500 3; kill -TERM $(cat run.pid); sleep 1.5; cat run.log
1 connections, 3s: 75686 requests (25229/s), 0 errors, mean latency 0.040 ms, max 10.1 ms
50 connections, 3s: 289857 requests (96619/s), 0 errors, mean latency 0.517 ms, max 8.7 ms
500 connections, 3s: 231623 requests (77208/s), 0 errors, mean latency 6.484 ms, max 23.0 ms
evhttpd: serving www on 127.0.0.1:8472
evhttpd: shut down after 597166 requests
```

One thread served 500 simultaneous keep-alive connections without errors, on a 2-core machine that was also running the 500-thread load generator. Throughput barely changed between 50 and 500 connections while mean latency grew roughly in proportion — the server was already saturated (both cores were busy, half of them with the load generator), so more clients just wait longer in line. That is the expected shape: throughput plateaus at capacity, and latency then rises with load (Little's law: concurrency = throughput × latency).

### Review checklist

- Can any socket operation block? (Every socket is non-blocking; file reads are the remaining risk — fine for local disks with the page cache, not for network file systems.)
- Is every limit enforced before data is stored: head size, path length, method length?
- Are partial sends and partially received requests handled, including several requests arriving in one read?
- Does every path that closes a connection also release its file descriptor and memory? (Run the tests under ASan, as above.)

### Extensions

1. Use `sendfile(2)` to send file bodies without copying through user space, and measure.
2. Add a thread pool for file I/O so cold files don't block the loop, signalling completion with `eventfd`.
3. Run one event loop per core with `SO_REUSEPORT`, and measure the scaling.

## Answers to the Chapter 10 prediction questions

There are no prediction questions in this chapter; the explanations are in the units' text.

**10.8 Q1.** `POLLIN` (data is waiting), possibly together with `POLLRDHUP`/`POLLHUP` depending on the system and the timing. The first `read` returns the 10 bytes; the second returns 0 — end of file — because the peer's FIN has arrived. Readiness says "a read will not block", and end of file is one of the things a read can return without blocking.

# Chapter 11 — Performance and the Machine

*Goal: write fast C on purpose — by reading what the compiler produces, understanding the CPU's caches, branch predictor, and vector units, and proving every optimisation with measurements.*

C is chosen when performance matters, but C code is not fast automatically. The difference between a naive and a well-written loop is often 10× or more, and it comes almost entirely from the hardware: how data moves through caches, whether branches are predictable, whether the compiler can use vector instructions. This chapter teaches you to read x86-64 assembly and the calling convention, to see what the optimiser does, to measure the memory hierarchy and the branch predictor directly, to use SIMD, and to profile — then makes a matrix multiplication more than ten times faster, step by measured step.

All measurements here were taken on a 2-core virtual machine (Intel Xeon, 2.1 GHz, AVX-512 capable). Absolute numbers will differ on your machine; the ratios and shapes are what to compare. Because it is a virtual machine, hardware performance counters are not available to `perf`; Valgrind's Cachegrind simulates the caches instead.

**What you will be able to do after this chapter**

- Read the x86-64 assembly the compiler generates for a C function.
- Explain the System V AMD64 calling convention and call assembly from C.
- Use optimisation levels, `restrict`, inlining, and vectorisation reports deliberately.
- Measure cache and memory latency, and structure data for locality.
- Explain branch prediction and write branchless code when it helps.
- Use SIMD through auto-vectorisation and intrinsics.
- Profile with `perf` and Cachegrind, and use link-time and profile-guided optimisation.

## Topic 11A — Seeing the machine code

## Unit 11.1 — Reading x86-64 assembly

**One-line summary:** x86-64 has 16 general-purpose 64-bit registers (`rax`, `rbx`, `rcx`, `rdx`, `rsi`, `rdi`, `rbp`, `rsp`, `r8`–`r15`, with 32-bit halves `eax`, `edi`, …) and instructions that move data, compute, compare, and jump. You don't need to write assembly to benefit from reading it: a glance at the compiler's output tells you whether a loop was vectorised, a function inlined, or a bounds check removed.

```c
// file: asm1.c
int add(int a, int b) { return a + b; }

long sum(const int *a, long n) {
    long s = 0;
    for (long i = 0; i < n; i++)
        s += a[i];
    return s;
}

int is_even(unsigned x) { return x % 2 == 0; }

int div10(int x) { return x / 10; }
```

```console
$ gcc -O2 -S -masm=intel -fno-asynchronous-unwind-tables -fcf-protection=none asm1.c -o - | grep -vE '^\s*\.(file|text|globl|type|size|section|ident|p2align)|^\s*$'
	.intel_syntax noprefix
add:
	lea	eax, [rdi+rsi]
	ret
sum:
	test	rsi, rsi
	jle	.L6
	lea	rcx, [rdi+rsi*4]
	xor	eax, eax
.L5:
	movsx	rdx, DWORD PTR [rdi]
	add	rdi, 4
	add	rax, rdx
	cmp	rdi, rcx
	jne	.L5
	ret
.L6:
	xor	eax, eax
	ret
is_even:
	mov	eax, edi
	not	eax
	and	eax, 1
	ret
div10:
	movsx	rax, edi
	sar	edi, 31
	imul	rax, rax, 1717986919
	sar	rax, 34
	sub	eax, edi
	ret
```

How to read it (Intel syntax, destination first):

- **`add`**: `lea eax, [rdi+rsi]` — arguments arrive in `edi` and `esi`, and the result is returned in `eax`. `lea` ("load effective address") is the address-computing unit used as a cheap three-operand adder.
- **`sum`**: a loop with a pointer (`rax`) walking to an end pointer (`rdx`), sign-extending each `int` to 64 bits (`movsx rdx, DWORD PTR [rax]`) and adding. At `-O2` GCC 13 did not vectorise it (it does at `-O3`, or since GCC 12 for some loops at `-O2` with a cheap cost model).
- **`is_even`**: no division — `x % 2 == 0` became a test of the lowest bit (`not` + `and 1`).
- **`div10`**: no `idiv` either. Division by a constant became multiplication by a "magic number" (`0x66666667`), a shift, and a correction for negative numbers — because a multiply takes about 3 cycles and a 32-bit `idiv` 20 or more.

Tools: `gcc -S` writes assembly; `objdump -d -M intel prog` disassembles a binary (add `-S` with a `-g` build to interleave source); **Compiler Explorer** (godbolt.org) shows the assembly of any compiler instantly and is the fastest way to answer "what does this compile to?".

<!-- expanded -->
### Going deeper: the same loop at `-O0` and `-O2`

The best way to learn to read assembly is to compile a few lines you understand completely and compare optimisation levels. At `-O0` GCC translates each statement on its own and keeps every variable in memory on the stack, so the code is long but follows the source line by line — which is what a debugger needs. At `-O2` the variables live in registers and the loop shrinks to a handful of instructions.

```c
// file: sumloop.c
long sum(const int *a, long n) {
    long s = 0;
    for (long i = 0; i < n; i++)
        s += a[i];
    return s;
}
```

```console
$ gcc -O0 -S -masm=intel -fno-asynchronous-unwind-tables -fcf-protection=none sumloop.c -o - | grep -vE '^\s+\.|^$' | head -30
sum:
	push	rbp
	mov	rbp, rsp
	mov	QWORD PTR -24[rbp], rdi
	mov	QWORD PTR -32[rbp], rsi
	mov	QWORD PTR -16[rbp], 0
	mov	QWORD PTR -8[rbp], 0
	jmp	.L2
.L3:
	mov	rax, QWORD PTR -8[rbp]
	lea	rdx, 0[0+rax*4]
	mov	rax, QWORD PTR -24[rbp]
	add	rax, rdx
	mov	eax, DWORD PTR [rax]
	cdqe
	add	QWORD PTR -16[rbp], rax
	add	QWORD PTR -8[rbp], 1
.L2:
	mov	rax, QWORD PTR -8[rbp]
	cmp	rax, QWORD PTR -32[rbp]
	jl	.L3
	mov	rax, QWORD PTR -16[rbp]
	pop	rbp
	ret
$ gcc -O2 -fno-tree-vectorize -S -masm=intel -fno-asynchronous-unwind-tables -fcf-protection=none sumloop.c -o - | grep -vE '^\s+\.|^$'
sum:
	test	rsi, rsi
	jle	.L4
	lea	rcx, [rdi+rsi*4]
	xor	eax, eax
.L3:
	movsx	rdx, DWORD PTR [rdi]
	add	rdi, 4
	add	rax, rdx
	cmp	rdi, rcx
	jne	.L3
	ret
.L4:
	xor	eax, eax
	ret
```

Reading the `-O0` version: the arguments arrive in `rdi` (`a`) and `rsi` (`n`) and are immediately stored to the stack (`QWORD PTR [rbp-24]` and so on); every use of `s` or `i` reloads it from memory, adds, and stores it back. In the `-O2` version (vectorisation turned off here to keep it short): `rax` holds `s`, the pointer itself is advanced instead of an index, `movsx`/`movsxd` sign-extends each `int` to 64 bits before adding, and the loop is three or four instructions between the label and the backward jump. The `test rsi, rsi` / `jle` at the top handles `n <= 0` before entering the loop. With `-O2` and vectorisation on (GCC 12 and later enable it at `-O2`), the loop would also have a SIMD version — Unit 11.6.

### Mastery check

1. **Predict.** What will `unsigned mod8(unsigned x) { return x % 8; }` compile to? And `int smod8(int x) { return x % 8; }`? Check with `-S`.

## Unit 11.2 — The calling convention and the ABI

**One-line summary:** On Linux, macOS, and the BSDs (the **System V AMD64 ABI**), the first six integer or pointer arguments go in `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`, floating-point arguments in `xmm0`–`xmm7`, and results come back in `rax` (and `xmm0`). `rbx`, `rbp`, `r12`–`r15` must be preserved by the callee; the stack must be 16-byte aligned at each `call`. Code written in assembly follows these rules to be callable from C.

```nasm
; file: abi/dot3.asm
; long dot3(const long *a, const long *b) -- dot product of two 3-element vectors
; System V AMD64: a in rdi, b in rsi, result in rax. Uses only caller-saved registers.
        global  dot3
        section .text
dot3:
        mov     rax, [rdi]          ; rax = a[0]
        imul    rax, [rsi]          ; rax *= b[0]
        mov     rcx, [rdi + 8]      ; rcx = a[1]
        imul    rcx, [rsi + 8]      ; rcx *= b[1]
        add     rax, rcx
        mov     rcx, [rdi + 16]     ; a[2] * b[2]
        imul    rcx, [rsi + 16]
        add     rax, rcx
        ret

        section .note.GNU-stack noalloc noexec nowrite progbits   ; "this code needs no executable stack"
```

```c
// file: abi/main.c
#include <stdint.h>
#include <stdio.h>

long dot3(const long *a, const long *b);     /* implemented in dot3.asm */

static inline uint64_t rdtsc(void) {          /* inline assembly: read the CPU's timestamp counter */
    uint32_t lo, hi;
    __asm__ volatile("rdtsc" : "=a"(lo), "=d"(hi));
    return (uint64_t)hi << 32 | lo;
}

int main(void) {
    long a[3] = {1, 2, 3}, b[3] = {4, 5, 6};
    uint64_t t0 = rdtsc();
    long d = dot3(a, b);
    uint64_t t1 = rdtsc();
    printf("dot3 = %ld (expected 32), took about %llu reference cycles including the call\n",
           d, (unsigned long long)(t1 - t0));
    return 0;
}
```

```console
$ cd abi && nasm -f elf64 dot3.asm -o dot3.o && gcc -std=gnu17 -O2 -Wall main.c dot3.o -o dot3demo && ./dot3demo
dot3 = 32 (expected 32), took about 78 reference cycles including the call
```

(The `.note.GNU-stack` section matters: without it, the linker warns that the object file "implies executable stack" and — for compatibility with ancient code — makes the whole program's stack executable, which disables an important exploit mitigation (Chapter 12). The first build of this example produced exactly that warning.)

The assembly function follows the ABI, so C calls it with an ordinary prototype — exactly how performance-critical routines in libraries such as OpenSSL and glibc's `memcpy` are written. The **inline assembly** `rdtsc` uses GCC's extended asm syntax: the instruction, then outputs (`"=a"(lo)` means "this C variable gets `eax`"), inputs, and clobbers. Use inline assembly rarely — compiler **intrinsics** (Unit 11.6) are almost always better — but recognise it when you read kernel or runtime code.

### The ABI is a contract

Struct layout (Unit 3.5), argument passing (small structs in registers, large ones by hidden pointer), name mangling (none in C), and stack alignment are all part of the platform ABI. C is the lingua franca of foreign-function interfaces because its ABI is simple and stable: Python's `ctypes`, Rust's `extern "C"`, Go's cgo, and Java's JNI all call C functions through it. Windows x64 uses a *different* convention (`rcx`, `rdx`, `r8`, `r9`, plus 32 bytes of "shadow space") — portable assembly must handle both.

### Deep & Tricky

**DT-85. Calling a function through a wrong prototype breaks the ABI silently.** *(Guarantee: UB; platform behaviour)* If C code declares `double f(double)` but the function (in assembly, or another file) actually takes and returns `long`, the caller puts the argument in `xmm0` and reads the result from `xmm0`, while the callee reads `rdi` and writes `rax`. Nothing checks this across a language or file boundary (DT-5); the result is garbage.

### Mastery check

1. **Write.** An assembly function `long max3(long a, long b, long c)` using `cmp` and `cmovl`, called from C with tests.

## Unit 11.3 — What the optimiser does

**One-line summary:** At `-O2` the compiler inlines small functions, keeps variables in registers, removes dead code, folds constants, hoists invariant work out of loops, and (with `-O3` or a cheap cost model) vectorises. Aliasing limits all of this: when two pointers might refer to the same memory, the compiler must reload values. `restrict` promises they don't.

```c
// file: restrict.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

/* Without restrict: dst might overlap src, so every store may change *scale. */
__attribute__((noipa)) void scale_plain(float *dst, const float *src, const float *scale, long n) {
    for (long i = 0; i < n; i++)
        dst[i] = src[i] * *scale;
}

/* With restrict: the caller promises no overlap, so *scale is loaded once and the loop vectorised. */
__attribute__((noipa)) void scale_restrict(float *restrict dst, const float *restrict src,
                                              const float *restrict scale, long n) {
    for (long i = 0; i < n; i++)
        dst[i] = src[i] * *scale;
}

static double secs(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}

int main(void) {
    enum { N = 4096, REPS = 200000 };
    float *a = malloc(N * sizeof *a), *b = malloc(N * sizeof *b), s = 1.0001f;
    for (int i = 0; i < N; i++) a[i] = (float)i;
    double t0 = secs();
    for (int r = 0; r < REPS; r++) scale_plain(b, a, &s, N);
    double t1 = secs();
    for (int r = 0; r < REPS; r++) scale_restrict(b, a, &s, N);
    double t2 = secs();
    printf("plain: %.0f ms, restrict: %.0f ms (%.1fx), check %.1f\n",
           (t1 - t0) * 1e3, (t2 - t1) * 1e3, (t1 - t0) / (t2 - t1), b[N - 1]);
    free(a);
    free(b);
    return 0;
}
```

```console
$ gcc -std=c17 -O3 -Wall restrict.c -o restrict -fopt-info-vec-optimized 2>&1 | sed 's/^restrict.c://'; ./restrict
9:24: optimized: loop vectorized using 16 byte vectors
9:24: optimized:  loop versioned for vectorization because of possible aliasing
9:24: optimized: loop vectorized using 8 byte vectors
16:24: optimized: loop vectorized using 16 byte vectors
16:24: optimized: loop vectorized using 8 byte vectors
29:23: optimized: loop vectorized using 16 byte vectors
plain: 57 ms, restrict: 59 ms (1.0x), check 4095.4
```

`-fopt-info-vec-optimized` makes GCC report every loop it vectorised. Both loops were vectorised — GCC handles the possible overlap in `scale_plain` by generating a **runtime alias check** plus two versions of the loop, and `*scale` is still loaded inside the loop because a store to `dst` might change it. With `restrict`, the check and the reload disappear. Here the two versions measured within noise of each other: the runtime check is cheap, the loop is limited by memory bandwidth, and reloading `*scale` from L1 cache costs almost nothing. `restrict` pays off when aliasing prevents vectorisation or other optimisations entirely, which happens more often in complex loops — check `-fopt-info-vec-missed` output for "possible aliasing" before reaching for it.

`restrict` is a promise: if the pointers do overlap, the behaviour is undefined. `memcpy` is declared with `restrict` parameters, which is why it may not be used for overlapping regions (Unit 2.4).

### Optimisation flags

| Flag | Effect |
| --- | --- |
| `-O0` | no optimisation: best debugging, slowest code |
| `-Og` | optimise without hurting debugging |
| `-O2` | the standard release level |
| `-O3` | adds aggressive loop optimisations and vectorisation; sometimes larger and not always faster |
| `-Os`, `-Oz` | optimise for size |
| `-march=native` | use every instruction the build machine supports (AVX2, AVX-512): the binary may not run on older CPUs |
| `-flto` | link-time optimisation: inline and optimise across `.c` files (Unit 11.7) |
| `-ffast-math` | non-IEEE floating-point shortcuts (DT-16): only for code tested under it |

### Mastery check

1. **Explain.** Why can't the compiler hoist the load of `*scale` out of the loop in `scale_plain`?

## Topic 11B — The CPU's view of your data

## Unit 11.4 — The memory hierarchy

**One-line summary:** A CPU core executes several instructions per nanosecond, but main memory takes about 100 ns to answer. **Caches** — small, fast memories holding recently used 64-byte **cache lines** — hide that latency when access patterns are predictable: sequential and nearby accesses are cheap, scattered accesses are expensive. Designing data layouts for locality is the most important performance skill in C.

```c
// file: latency.c
/* Measure memory latency: chase a random cyclic chain of pointers through
   buffers of increasing size. Each load depends on the previous one, so the
   time per step is the latency of whichever cache level holds the buffer. */
#define _POSIX_C_SOURCE 200809L
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

static double secs(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}

int main(void) {
    printf("%10s %10s\n", "buffer", "ns/load");
    for (size_t kb = 16; kb <= 256 * 1024; kb *= 4) {
        size_t n = kb * 1024 / 64;                      /* one pointer per 64-byte line */
        char *mem = aligned_alloc(64, n * 64);
        size_t *order = malloc(n * sizeof *order);
        for (size_t i = 0; i < n; i++) order[i] = i;
        srand(1);
        for (size_t i = n - 1; i > 0; i--) {            /* random permutation (Fisher-Yates) */
            size_t j = (size_t)rand() % (i + 1);
            size_t t = order[i]; order[i] = order[j]; order[j] = t;
        }
        for (size_t i = 0; i < n; i++)                  /* link lines into one random cycle */
            *(void **)(mem + order[i] * 64) = mem + order[(i + 1) % n] * 64;
        void **p = (void **)(mem + order[0] * 64);
        long steps = 20000000;
        double t0 = secs();
        for (long s = 0; s < steps; s++)
            p = *p;
        double t1 = secs();
        printf("%7zu KB %10.1f%s\n", kb, (t1 - t0) / steps * 1e9, p == NULL ? "!" : "");
        free(mem);
        free(order);
    }
    return 0;
}
```

```console
$ gcc -std=c17 -O2 -Wall latency.c -o latency && ./latency
    buffer    ns/load
     16 KB        1.3
     64 KB        4.7
    256 KB        4.8
   1024 KB        6.4
   4096 KB       85.1
  16384 KB      131.9
  65536 KB      208.9
 262144 KB      227.2
```

The staircase is the cache hierarchy: under 2 ns while the buffer fits in the 48 KiB L1 data cache, around 5–6 ns in the 2 MiB per-core L2, 60–70 ns in this virtual machine's large shared L3, and 160–200 ns once the buffer exceeds what the caches can hold and loads go to DRAM. (On a typical desktop CPU, L3 is nearer 10–20 ns and DRAM 70–100 ns; cloud VMs on large server chips see longer latencies.) A dependent random access to main memory costs as much time as hundreds of arithmetic instructions.

### Locality in practice

```c
// file: locality.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

enum { N = 4096 };
static double secs(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}

typedef struct { double x, y, z, mass; char name[32]; } ParticleAoS;   /* array of structs */
typedef struct { double *x, *y, *z, *mass; } ParticlesSoA;             /* struct of arrays */

int main(void) {
    double (*m)[N] = malloc(sizeof(double[N][N]));      /* 128 MB matrix */
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++)
            m[i][j] = 1.0;
    double s = 0, t0 = secs();
    for (int i = 0; i < N; i++)                         /* row by row: sequential */
        for (int j = 0; j < N; j++)
            s += m[i][j];
    double t1 = secs();
    for (int j = 0; j < N; j++)                         /* column by column: 32 KB stride */
        for (int i = 0; i < N; i++)
            s += m[i][j];
    double t2 = secs();
    printf("row-major traversal %.0f ms, column-major %.0f ms (%.1fx slower)\n",
           (t1 - t0) * 1e3, (t2 - t1) * 1e3, (t2 - t1) / (t1 - t0));
    free(m);

    enum { P = 4000000 };
    ParticleAoS *aos = calloc(P, sizeof *aos);
    ParticlesSoA soa = {calloc(P, 8), calloc(P, 8), calloc(P, 8), calloc(P, 8)};
    double sx = 0;
    t0 = secs();
    for (int i = 0; i < P; i++) sx += aos[i].x;         /* uses 8 of every 64 bytes loaded */
    t1 = secs();
    for (int i = 0; i < P; i++) sx += soa.x[i];         /* uses every byte loaded */
    t2 = secs();
    printf("sum of x: array of structs %.1f ms, struct of arrays %.1f ms (sizeof AoS element %zu)\n",
           (t1 - t0) * 1e3, (t2 - t1) * 1e3, sizeof *aos);
    printf("%g\n", s + sx);
    return 0;
}
```

```console
$ gcc -std=c17 -O2 -Wall locality.c -o locality && ./locality
row-major traversal 18 ms, column-major 227 ms (12.7x slower)
sum of x: array of structs 33.4 ms, struct of arrays 7.0 ms (sizeof AoS element 64)
3.35544e+07
```

Same arithmetic, same data, very different speeds. Traversing a row-major matrix by columns touches a new cache line for every element and defeats the hardware prefetcher's best case. Summing one field of 64-byte structs loads eight times more memory than summing a dense array of that field. These two patterns — **iterate in storage order** and **store together what is used together** (struct of arrays for field-wise processing, hot/cold splitting of structs) — account for most "data-oriented design".

### Deep & Tricky

**DT-86. A linked list of N elements can be 10–100× slower to traverse than an array of N elements.** *(Hardware fact)* Each node is a dependent load from a possibly distant address: the CPU cannot start loading node k+1 until node k arrives, and the prefetcher cannot guess the address. An array traversal issues independent loads to sequential addresses, which the prefetcher streams in ahead of use. Big-O is identical; the latency table above is the difference.

### Mastery check

1. **Explain.** Why does the pointer-chasing benchmark use a *random* cycle rather than following lines in order?
2. **Write.** Split a 96-byte struct whose loop uses only 2 fields into hot and cold parts; measure the loop before and after.

## Unit 11.5 — Branch prediction

**One-line summary:** CPUs execute instructions speculatively along the path a **branch predictor** guesses; a wrong guess discards that work, costing about 15–20 cycles. Branches that follow patterns are nearly free; branches on random data are expensive. Sorting data, or replacing the branch with arithmetic (**branchless** code, conditional moves), can remove the cost.

```c
// file: branches.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

enum { N = 1 << 22, REPS = 100 };
static double secs(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}
static int cmp(const void *a, const void *b) { return *(const int *)a - *(const int *)b; } /* values 0..255: no overflow */

__attribute__((noipa)) static long sum_big_branchy(const int *a, int n) {
    long s = 0;
    for (int i = 0; i < n; i++)
        if (a[i] >= 128)                 /* unpredictable when data is random */
            s += a[i];
    return s;
}

__attribute__((noipa)) static long sum_big_branchless(const int *a, int n) {
    long s = 0;
    for (int i = 0; i < n; i++) {
        int mask = -(a[i] >= 128);      /* all ones or all zeros */
        s += a[i] & mask;
    }
    return s;
}

int main(void) {
    int *a = malloc(N * sizeof *a);
    srand(3);
    for (int i = 0; i < N; i++) a[i] = rand() % 256;
    long r = 0;
    double t0 = secs();
    for (int k = 0; k < REPS; k++) r += sum_big_branchy(a, N);
    double t1 = secs();
    for (int k = 0; k < REPS; k++) r += sum_big_branchless(a, N);
    double t2 = secs();
    qsort(a, N, sizeof *a, cmp);
    double t3 = secs();
    for (int k = 0; k < REPS; k++) r += sum_big_branchy(a, N);
    double t4 = secs();
    printf("random data:  branchy %4.0f ms, branchless %4.0f ms\n", (t1 - t0) * 1e3, (t2 - t1) * 1e3);
    printf("sorted data:  branchy %4.0f ms\n", (t4 - t3) * 1e3);
    printf("(checksum %ld)\n", r);
    return 0;
}
```

```console
$ gcc -std=c17 -O1 -fno-if-conversion -fno-if-conversion2 -fno-tree-loop-if-convert -Wall branches.c -o branches_O1 && ./branches_O1
random data:  branchy 1605 ms, branchless  497 ms
sorted data:  branchy  313 ms
(checksum 120561467100)
$ gcc -std=c17 -O3 -Wall branches.c -o branches_O3 && ./branches_O3
random data:  branchy  161 ms, branchless  150 ms
sorted data:  branchy  155 ms
(checksum 120561467100)
```

The first build uses `-O1` with GCC's *if-conversion* passes turned off, to keep a real conditional jump in the loop. There, the branchy loop on random data is several times slower than the same loop on sorted data — identical work, but on random data the predictor is wrong about half the time — and the branchless version is fast on random data too. (Without those flags, GCC converted the branch to a conditional move even at `-O1`, and the first measurement for this unit showed no difference at all.) At `-O3`, GCC vectorised the loop into branchless code by itself, and all three timings collapse together: a reminder that the compiler often applies such transformations for you, and that only measurement of *your* build tells you whether hand-tuning is needed. (The `cmp` function subtracts safely here only because the values are 0–255; DT-36 still applies in general.)

GCC's `__builtin_expect(cond, 1)` (C23-adjacent: `[[likely]]` exists only in C++) tells the compiler which way a branch usually goes, so it lays out the common path without a jump — useful for error checks, which are almost never taken.

### Mastery check

1. **Explain.** Why does sorting the data change the speed of code that does not depend on order?

## Unit 11.6 — SIMD: vector instructions

**One-line summary:** SIMD instructions operate on several values at once: SSE on 128-bit registers (4 `float`s), AVX2 on 256 bits (8 `float`s), AVX-512 on 512 bits (16). The compiler **auto-vectorises** simple loops (`-O3`, `-march=native`); **intrinsics** (`<immintrin.h>`) give explicit control when it cannot.

```c
// file: simd.c
#define _POSIX_C_SOURCE 200809L
#include <immintrin.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

static double secs(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}

__attribute__((noipa)) static float dot_scalar(const float *a, const float *b, int n) {
    float s = 0;
    for (int i = 0; i < n; i++)
        s += a[i] * b[i];        /* float addition is not associative: no vectorising without -ffast-math */
    return s;
}

__attribute__((noipa, target("avx2,fma"))) static float dot_avx2(const float *a, const float *b, int n) {
    __m256 acc0 = _mm256_setzero_ps(), acc1 = _mm256_setzero_ps();
    int i = 0;
    for (; i + 16 <= n; i += 16) {           /* two accumulators hide the FMA latency */
        acc0 = _mm256_fmadd_ps(_mm256_loadu_ps(a + i), _mm256_loadu_ps(b + i), acc0);
        acc1 = _mm256_fmadd_ps(_mm256_loadu_ps(a + i + 8), _mm256_loadu_ps(b + i + 8), acc1);
    }
    __m256 acc = _mm256_add_ps(acc0, acc1);
    float lanes[8];
    _mm256_storeu_ps(lanes, acc);
    float s = 0;
    for (int k = 0; k < 8; k++)
        s += lanes[k];
    for (; i < n; i++)                       /* leftover elements */
        s += a[i] * b[i];
    return s;
}

int main(void) {
    enum { N = 4096, REPS = 200000 };
    float *a = aligned_alloc(32, N * sizeof *a), *b = aligned_alloc(32, N * sizeof *b);
    for (int i = 0; i < N; i++) { a[i] = (float)(i % 7) * 0.5f; b[i] = (float)(i % 5) * 0.25f; }
    float r1 = 0, r2 = 0;
    double t0 = secs();
    for (int k = 0; k < REPS; k++) r1 += dot_scalar(a, b, N);
    double t1 = secs();
    for (int k = 0; k < REPS; k++) r2 += dot_avx2(a, b, N);
    double t2 = secs();
    printf("scalar %.0f ms, AVX2+FMA %.0f ms: %.1fx faster; results %.6g vs %.6g\n",
           (t1 - t0) * 1e3, (t2 - t1) * 1e3, (t1 - t0) / (t2 - t1), r1 / REPS, r2 / REPS);
    printf("this CPU supports AVX2: %s, AVX-512F: %s\n",
           __builtin_cpu_supports("avx2") ? "yes" : "no", __builtin_cpu_supports("avx512f") ? "yes" : "no");
    return 0;
}
```

```console
$ gcc -std=c17 -O3 -Wall simd.c -o simd && ./simd
scalar 449 ms, AVX2+FMA 49 ms: 9.2x faster; results 3071.97 vs 3071.97
this CPU supports AVX2: yes, AVX-512F: yes
```

The intrinsic version computes 16 multiply-adds per loop iteration with fused multiply-add instructions, many times faster than the scalar loop. (The benchmark functions are marked `noipa` — "no inter-procedural analysis" — because with plain `noinline`, GCC noticed that `dot_scalar` has no side effects, called it once, and reused the result for all 200,000 repetitions: the first run reported 0 ms for both versions. Benchmarks must defeat optimisations that would not happen in real use.) The results differ in the last digits because the additions happen in a different order — floating-point addition is not associative, which is exactly why the compiler did not vectorise `dot_scalar` itself: doing so would change the result. (`-ffast-math` or `-fassociative-math` gives it permission.)

`target("avx2,fma")` compiles just that function for AVX2, so the program still runs on older CPUs as long as it checks `__builtin_cpu_supports` before calling it — the **function multi-versioning** pattern used by `memcpy`, compression libraries, and codecs.

<!-- expanded -->
### Going deeper: why a float sum does not vectorise

The compiler vectorises an integer sum without being asked, but leaves a `float` sum as a scalar loop. The reason is not a missing optimisation: vectorising a reduction means adding the elements in a different order — eight partial sums, combined at the end — and floating-point addition is not associative. `(a + b) + c` can differ from `a + (b + c)` in the last bits, so the reordered result could differ from what the source code specifies. GCC reorders only when told that it may: `-ffast-math`, or more narrowly `-fassociative-math` (which also requires `-fno-signed-zeros -fno-trapping-math`).

```c
// file: fsum.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

__attribute__((noipa)) static float sum(const float *a, size_t n) {   /* noipa: no cross-call assumptions */
    float s = 0.0f;
    for (size_t i = 0; i < n; i++) s += a[i];
    return s;
}

int main(void) {
    enum { N = 1 << 20, REPS = 200 };
    float *a = malloc(N * sizeof *a);
    srand(1);
    for (size_t i = 0; i < N; i++) a[i] = (float)rand() / RAND_MAX;
    struct timespec t0, t1;
    float s = 0;
    clock_gettime(CLOCK_MONOTONIC, &t0);
    for (int r = 0; r < REPS; r++) s = sum(a, N);
    clock_gettime(CLOCK_MONOTONIC, &t1);
    double ms = ((t1.tv_sec - t0.tv_sec) * 1e3 + (t1.tv_nsec - t0.tv_nsec) / 1e6) / REPS;
    printf("sum = %.6f   %.3f ms per sum of %d floats\n", s, ms, N);
    free(a);
    return 0;
}
```

```console
$ gcc -std=c17 -O3 -march=native fsum.c -o fsum -fopt-info-vec-missed 2>&1 | grep -oE "reduction: unsafe fp math optimization|couldn't vectorize loop" | sort -u; ./fsum
couldn't vectorize loop
sum = 524281.968750   0.623 ms per sum of 1048576 floats
$ gcc -std=c17 -O3 -march=native -ffast-math fsum.c -o fsum-fast -fopt-info-vec-optimized 2>&1 | grep -oE "loop vectorized using [0-9]+ byte vectors" | sort -u; ./fsum-fast
loop vectorized using 16 byte vectors
loop vectorized using 32 byte vectors
sum = 524284.062500   0.233 ms per sum of 1048576 floats
```

The `-ffast-math` build is almost three times faster — and prints a slightly different sum. Both answers are "correct" approximations of the exact sum of a million random numbers; they are different approximations. Whether that matters depends on the program: for a graphics shader, never; for a financial report or a test that compares against stored output, always.

### Deep & Tricky

**DT-87. `-ffast-math` is not one optimisation but a bundle of promises you make to the compiler — no NaNs, no infinities, no signed zeros, no trapping, associativity allowed — and breaking a promise makes the program's results, not just its speed, change; it even affects other code, because linking with `-ffast-math` can switch the CPU into "flush denormals to zero" mode for the whole process.** *(Toolchain behaviour: GCC manual, `-ffast-math` "can result in incorrect output for programs that depend on an exact implementation of IEEE or ISO rules"; GCC links `crtfastmath.o`, which sets the FTZ/DAZ bits of MXCSR at start-up, unless `-shared` — changed in GCC 13 so that shared libraries no longer do this)* DT-16 showed `isnan` being optimised away under `-ffast-math`. Prefer the narrow flags you actually need (`-fassociative-math` for reductions, `-fno-math-errno` to let `sqrt` become one instruction), apply them per file, and test the results, not just the timing.

### Mastery check

1. **Explain.** Why does using two accumulators make the AVX2 loop faster than one?

## Topic 11C — Measuring and tuning

## Unit 11.7 — Profiling, LTO, and PGO

**One-line summary:** Measure before optimising: **`perf record`/`perf report`** shows where time goes by sampling; **Cachegrind** simulates caches and counts instructions exactly; **`-pg`/gprof** and **Callgrind** give call graphs. Then use the compiler's whole-program tools: **link-time optimisation** (`-flto`) inlines across files, and **profile-guided optimisation** (`-fprofile-generate`, run, `-fprofile-use`) optimises for the paths your real workload takes.

```c
// file: prof/work.c
#include "work.h"

int classify(int x) {                         /* in a separate file: not inlinable without LTO */
    if (x % 97 == 0) return 3;
    if (x % 13 == 0) return 2;
    if (x & 1) return 1;
    return 0;
}

double heavy(int n) {
    double s = 0;
    for (int i = 1; i <= n; i++)
        s += 1.0 / ((double)i * i);
    return s;
}
```

```c
// file: prof/work.h
#ifndef WORK_H
#define WORK_H
int classify(int x);
double heavy(int n);
#endif
```

```c
// file: prof/main.c
#include <stdio.h>
#include "work.h"

int main(void) {
    long counts[4] = {0};
    for (int i = 0; i < 300000000; i++)
        counts[classify(i)]++;
    printf("counts %ld %ld %ld %ld, heavy %.6f\n", counts[0], counts[1], counts[2], counts[3], heavy(50000000));
    return 0;
}
```

```console
$ cd prof && gcc -O2 -g work.c main.c -o plain && perf record -q -o perf.data ./plain > /dev/null 2>&1 && perf report -i perf.data --stdio 2>/dev/null | grep -E '^ +[0-9]' | head -n 4
    45.51%  plain    plain                 [.] classify
    40.68%  plain    plain                 [.] main
    13.58%  plain    plain                 [.] heavy
     0.06%  plain    [kernel.kallsyms]     [k] __alloc_frozen_pages_noprof
$ cd prof && gcc -O2 -flto work.c main.c -o lto && gcc -O2 -flto -fprofile-generate work.c main.c -o pgo && ./pgo > /dev/null && gcc -O2 -flto -fprofile-use work.c main.c -o pgo && for v in plain lto pgo; do TIMEFORMAT="$v: %R s"; time ./$v > /dev/null; done
plain: 0.411 s
lto: 0.323 s
pgo: 0.314 s
```

`perf report` attributes most samples to `classify` and `main`'s loop around it: the call overhead itself matters because `classify` is tiny and called 300 million times. **LTO** lets the compiler inline `classify` into `main` across the file boundary. **PGO** adds knowledge from a training run — which branches are taken how often — so the compiler orders and specialises code for the real distribution. On this program each step helped; on others PGO helps a lot (interpreters, compilers, databases — Chrome and Firefox ship PGO builds) or not at all. Measure.

(The profile-generating build must use the same output name as the final build: GCC names the profile files after the output, and the first attempt here — built as `pgo_gen` — produced "profile count data file not found" warnings and an unoptimised result.)

Cachegrind's exact counts complement sampling when you need to know *why* something is slow. Here it explains Unit 11.4's row-versus-column result:

```c
// file: prof/walk.c
#include <stdio.h>
#include <stdlib.h>
enum { N = 2048 };
int main(int argc, char *argv[]) {
    static double m[N][N];
    for (int a = 0; a < N; a++)              /* fill at run time: a never-written static */
        for (int b = 0; b < N; b++)          /* array is all zeros, and the compiler would */
            m[a][b] = a ^ b;                 /* fold the whole sum to a constant           */
    double s = 0;
    int by_column = argc > 1;
    for (int a = 0; a < N; a++)
        for (int b = 0; b < N; b++)
            s += by_column ? m[b][a] : m[a][b];
    printf("%g\n", s);
    return 0;
}
```

```console
$ cd prof && gcc -O2 -g walk.c -o walk && for mode in "" col; do valgrind --tool=cachegrind --cache-sim=yes --cachegrind-out-file=/dev/null ./walk $mode 2>&1 | grep -E 'D1  miss rate' | sed "s/==[0-9]*==/${mode:-row}/"; done
row D1  miss rate:       16.5% (     12.4%     +      24.9%  )
col D1  miss rate:       74.2% (     98.7%     +      24.9%  )
```

The first version of this program, without the fill loop, reported identical numbers for both modes: GCC had noticed that the static array is never written, so every element is zero, and replaced the entire walk with the constant 0 — there was nothing left to measure. With the fill loop (itself a row-wise pass, so it adds the same misses to both runs), the row-wise run misses on about one read in eight (12.4%) — one miss per 64-byte line of eight `double`s — while the column-wise run misses on 98.7% of its reads. (The write column, 24.9% in both, comes from the identical fill loop.) Cachegrind simulates a generic cache hierarchy, so its numbers are approximate, but it works everywhere — including virtual machines without hardware counters.

### Benchmarking rules

1. Measure the optimised build you ship, on realistic data.
2. Repeat runs and look at the spread; report the median or minimum, not a single run.
3. Keep the machine quiet; pin to a core (`taskset`) for micro-benchmarks.
4. Stop the compiler from deleting the work (use the result; `volatile` sinks; `noinline`).
5. Change one thing at a time, and keep the correctness check in the benchmark.

### Pattern

**P-11. Profile, form a hypothesis about the hardware, change one thing, measure again.** *(Practice)* Find the hot spot with a profiler; explain its cost in terms of the machine (cache misses, mispredictions, dependency chains, lack of vectorisation, call overhead); make one change that addresses that cause; verify both speed and results; keep the change only if the measurement justifies its complexity. *Trade-off:* slower progress than intuition-driven "optimisation", but changes that are real, understood, and kept for the right reasons.

### Mastery check

1. **Explain.** Why might PGO make a program *slower* on inputs unlike the training run?

## Chapter 11 Project — Make a hot loop 10× faster, and prove it

**Goal:** take a naive dense matrix multiplication (C = A × B for 1024 × 1024 `double` matrices) and make it at least ten times faster, one measured step at a time — explaining each improvement in terms of the hardware and verifying the results at every step.

### The steps

1. **Naive** `i-j-k` loops: the inner loop walks a column of B (stride 8 KB): a cache miss on almost every load.
2. **Loop interchange** to `i-k-j`: the inner loop walks rows of B and C sequentially, and becomes vectorisable.
3. **Compiler flags**: `-O3 -march=native` enables AVX2/AVX-512 and FMA.
4. **Cache blocking (tiling)**: multiply 64 × 64 blocks so the working set stays in L1/L2 and each loaded value is reused many times.
5. **Threads**: split rows of C across cores (Chapter 9).

### Reference solution

```c
// file: matmul/matmul.c
/* Dense matrix multiply, five versions, each verified against the naive result. */
#define _POSIX_C_SOURCE 200809L
#include <math.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

enum { N = 1024, BS = 64 };

static double secs(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}

static void naive(const double *restrict A, const double *restrict B, double *restrict C) {
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            double s = 0;
            for (int k = 0; k < N; k++)
                s += A[i * N + k] * B[k * N + j];        /* B walked down a column */
            C[i * N + j] = s;
        }
}

static void interchanged(const double *restrict A, const double *restrict B, double *restrict C) {
    memset(C, 0, sizeof(double) * N * N);
    for (int i = 0; i < N; i++)
        for (int k = 0; k < N; k++) {
            double a = A[i * N + k];
            for (int j = 0; j < N; j++)                  /* B and C walked along rows */
                C[i * N + j] += a * B[k * N + j];
        }
}

/* blocked multiplies rows [r0, r1) of C, tile by tile. */
static void blocked_rows(const double *restrict A, const double *restrict B, double *restrict C,
                         int r0, int r1) {
    for (int ii = r0; ii < r1; ii += BS)
        for (int kk = 0; kk < N; kk += BS)
            for (int jj = 0; jj < N; jj += BS)
                for (int i = ii; i < ii + BS && i < r1; i++)
                    for (int k = kk; k < kk + BS; k++) {
                        double a = A[i * N + k];
                        for (int j = jj; j < jj + BS; j++)
                            C[i * N + j] += a * B[k * N + j];
                    }
}

static void blocked(const double *A, const double *B, double *C) {
    memset(C, 0, sizeof(double) * N * N);
    blocked_rows(A, B, C, 0, N);
}

typedef struct { const double *A, *B; double *C; int r0, r1; } Part;
static void *part_worker(void *arg) {
    Part *p = arg;
    blocked_rows(p->A, p->B, p->C, p->r0, p->r1);
    return NULL;
}

static void threaded(const double *A, const double *B, double *C) {
    enum { T = 2 };
    memset(C, 0, sizeof(double) * N * N);
    pthread_t t[T];
    Part parts[T];
    for (int i = 0; i < T; i++) {
        parts[i] = (Part){A, B, C, N / T * i, N / T * (i + 1)};   /* disjoint rows: no sharing */
        pthread_create(&t[i], NULL, part_worker, &parts[i]);
    }
    for (int i = 0; i < T; i++)
        pthread_join(t[i], NULL);
}

static double max_diff(const double *X, const double *Y) {
    double m = 0;
    for (long i = 0; i < (long)N * N; i++)
        m = fmax(m, fabs(X[i] - Y[i]));
    return m;
}

int main(void) {
    double *A = malloc(sizeof(double) * N * N), *B = malloc(sizeof(double) * N * N);
    double *ref = malloc(sizeof(double) * N * N), *C = malloc(sizeof(double) * N * N);
    srand(5);
    for (long i = 0; i < (long)N * N; i++) {
        A[i] = rand() / (double)RAND_MAX - 0.5;
        B[i] = rand() / (double)RAND_MAX - 0.5;
    }
    const double flops = 2.0 * N * N * N;
    struct { const char *name; void (*fn)(const double *, const double *, double *); } v[] = {
        {"naive i-j-k", naive}, {"interchanged i-k-j", interchanged},
        {"blocked 64x64", blocked}, {"blocked + 2 threads", threaded},
    };
    double base = 0;
    for (int k = 0; k < 4; k++) {
        double best = 1e9;
        for (int rep = 0; rep < (k == 0 ? 1 : 3); rep++) {   /* best of 3 (naive: 1, it is slow) */
            double t0 = secs();
            v[k].fn(A, B, k == 0 ? ref : C);
            double t = secs() - t0;
            if (t < best) best = t;
        }
        if (k == 0) base = best;
        printf("%-22s %8.1f ms %6.2f GFLOP/s %6.1fx  max error %.1e\n", v[k].name, best * 1e3,
               flops / best / 1e9, base / best, k == 0 ? 0.0 : max_diff(C, ref));
    }
    free(A); free(B); free(C); free(ref);
    return 0;
}
```

```console
$ cd matmul && gcc -std=c17 -O2 -Wall -pthread matmul.c -o mm_O2 -lm && echo "== -O2" && ./mm_O2
== -O2
naive i-j-k              6328.5 ms   0.34 GFLOP/s    1.0x  max error 0.0e+00
interchanged i-k-j        518.0 ms   4.15 GFLOP/s   12.2x  max error 0.0e+00
blocked 64x64             382.4 ms   5.62 GFLOP/s   16.5x  max error 0.0e+00
blocked + 2 threads       190.7 ms  11.26 GFLOP/s   33.2x  max error 0.0e+00
$ cd matmul && gcc -std=c17 -O3 -march=native -Wall -pthread matmul.c -o mm_native -lm && echo "== -O3 -march=native" && ./mm_native
== -O3 -march=native
naive i-j-k              3050.3 ms   0.70 GFLOP/s    1.0x  max error 0.0e+00
interchanged i-k-j        451.3 ms   4.76 GFLOP/s    6.8x  max error 0.0e+00
blocked 64x64             157.2 ms  13.66 GFLOP/s   19.4x  max error 0.0e+00
blocked + 2 threads        80.0 ms  26.84 GFLOP/s   38.1x  max error 0.0e+00
```

Reading the results:

- **Interchange** gave the largest single jump — about 10× at `-O2`: the same arithmetic, but streaming through memory instead of striding, and a vectorisable inner loop.
- **`-O3 -march=native`** let every version use wide vector registers and FMA; even the naive loop became twice as fast.
- **Blocking** keeps a 64 × 64 tile of B (32 KB) hot in cache while it is reused for 64 rows of A. At `-O2` it gained nothing on this machine; with vectorisation it roughly doubled throughput. Once the arithmetic is fast, memory traffic becomes the bottleneck — and reducing memory traffic is what blocking does. An optimisation's value depends on what currently limits the code.
- **Two threads** on two cores added another 1.4–2×: the rows of C are disjoint, so no synchronisation is needed, but both threads share memory bandwidth and the L3 cache.
- **Max error is exactly 0** here: every version adds the products for each element of C in the same order, k = 0 … N−1, so the results are bit-identical. Optimisations that change the summation order — vectorising across k, splitting k between threads, `-ffast-math` — produce tiny differences (Unit 11.6). That is why the check computes a maximum difference to compare against a tolerance rather than demanding equality: a check that demanded bit-identical results would reject valid optimisations, and one with no limit would accept bugs.

From the `-O2` naive version to the native, blocked, threaded version is a speed-up of about 40×, well past the 10× goal. (Timings on a shared virtual machine vary by 10–20% between runs; the step-to-step ratios are what matter.)

Optimised libraries (OpenBLAS, Intel MKL, BLIS) reach close to the CPU's peak — several times faster again — through register blocking, packing B into contiguous panels, and hand-written assembly kernels. When you need fast linear algebra, call one; this project's purpose is to understand why it is fast.

### Review checklist

- Is every version verified against the reference with a stated tolerance?
- Are timings the best (or median) of several runs, with the same build flags noted?
- Can you explain each step's improvement in terms of caches, vectorisation, or cores — and did a profiler or a report (`-fopt-info-vec`) confirm the explanation?
- Did any step make the code harder to read without a measured gain? Remove it.

## Answers to the Chapter 11 prediction questions

**11.1 Q1.** `x % 8` for unsigned `x` becomes `and eax, 7`. For signed `x` the result must have the sign of `x` (Unit 1.5), so the compiler emits a few extra instructions (computing a bias for negative values) around the `and` — still no division.

# Chapter 12 — Security and Robustness

*Goal: write C that holds up against hostile input — by understanding how memory-safety bugs become security vulnerabilities, which defences the toolchain and OS provide, and how to find bugs with fuzzing before attackers do.*

Most critical security vulnerabilities in operating systems, browsers, and network services have been memory-safety bugs in C and C++: buffer overflows, use-after-free, integer overflows in size calculations, format-string bugs. Microsoft and Google have each reported that about 70% of their serious security bugs are of this kind. This chapter teaches how these bugs become exploitable — at the level needed to recognise and prevent them, not to build attacks — the mitigations compilers and kernels apply, the coding rules that avoid the bugs, and **fuzzing**, the most effective technique for finding them. The project fuzzes a binary parser, finds its bugs, and hardens it.

**What you will be able to do after this chapter**

- Explain how a buffer overflow corrupts adjacent data and control flow, and recognise vulnerable code.
- Describe and verify the standard mitigations: stack canaries, non-executable memory, ASLR/PIE, RELRO, `_FORTIFY_SOURCE`, and control-flow protection.
- Prevent integer overflows in size calculations, format-string bugs, and command injection.
- Apply secure-coding rules for input validation, secrets, randomness, and privileges.
- Write a libFuzzer harness, run it with sanitizers, and turn its findings into fixes and regression tests.

## Topic 12A — How memory bugs become vulnerabilities

## Unit 12.1 — Buffer overflows

**One-line summary:** Writing past the end of a buffer overwrites whatever lies next to it in memory: other variables, heap allocator metadata, or — on the stack — saved registers and the function's **return address**. An attacker who controls the overflowing bytes can change program decisions or redirect execution. The root cause is always the same: a write whose length is not checked against the destination's size.

```c
// file: overflow.c
#include <stdio.h>
#include <string.h>

struct session {
    char name[16];
    int is_admin;            /* lies right after name in memory */
};

/* login copies the user name without checking its length: the classic bug. */
static void login(struct session *s, const char *input) {
    strcpy(s->name, input);
}

int main(int argc, char *argv[]) {
    struct session s = {.is_admin = 0};
    login(&s, argc > 1 ? argv[1] : "alice");
    printf("name=\"%.16s\" is_admin=%d (0x%08x)\n", s.name, s.is_admin, (unsigned)s.is_admin);
    if (s.is_admin)
        printf("access granted to admin functions!\n");
    return 0;
}
```

```console
$ gcc -std=c17 -O0 -D_FORTIFY_SOURCE=0 -fno-stack-protector overflow.c -o overflow 2>/dev/null && ./overflow alice && ./overflow AAAAAAAAAAAAAAAAB
name="alice" is_admin=0 (0x00000000)
name="AAAAAAAAAAAAAAAA" is_admin=66 (0x00000042)
access granted to admin functions!
$ gcc -std=c17 -O2 -D_FORTIFY_SOURCE=2 overflow.c -o overflow_fortified && ./overflow_fortified AAAAAAAAAAAAAAAAB; echo "exit status $?"
*** buffer overflow detected ***: terminated
bash: line 1:  5504 Aborted                 ./overflow_fortified AAAAAAAAAAAAAAAAB
exit status 134
```

A 17-character name wrote 18 bytes (with the terminator) into a 16-byte field: the 17th byte `'B'` (0x42) and the terminator landed in `is_admin`, which became 66 — non-zero, so "access granted". No crash, no error: a **data-only attack** that changed a security decision. With `_FORTIFY_SOURCE`, glibc replaced `strcpy` with a checked version that knows the destination size and aborted the program instead. Fortification only catches cases where the compiler knows the buffer size; the real fix is to bound every copy (`snprintf`, or check `strlen(input) < sizeof s->name` first).

### The stack layout and the return address

On x86-64, a function's frame holds its local arrays, then (towards higher addresses) the saved frame pointer and the **return address** that `ret` will jump to. An overflow of a local array that runs far enough overwrites the return address; when the function returns, execution goes wherever the attacker's bytes say. Modern mitigations (Unit 12.2) make that much harder, but they work by *detecting* or *complicating* exploitation — the bug is still a bug.

```c
// file: smash.c
#include <stdio.h>
#include <string.h>

static void greet(const char *name) {
    char buf[16];
    strcpy(buf, name);          /* overflow when name has 16+ characters */
    printf("hello, %s\n", buf);
}

int main(int argc, char *argv[]) {
    greet(argc > 1 ? argv[1] : "world");
    return 0;
}
```

```console
$ gcc -std=c17 -O0 -D_FORTIFY_SOURCE=0 -fstack-protector-strong smash.c -o smash && ./smash world && ./smash "$(printf 'A%.0s' {1..40})"; echo "exit status $?"
hello, world
hello, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
*** stack smashing detected ***: terminated
bash: line 1:  5513 Aborted                 ./smash "$(printf 'A%.0s' {1..40})"
exit status 134
```

The **stack protector** placed a random value (a **canary**) between `buf` and the return address and checked it before returning: the overflow changed it, so the program aborted with "stack smashing detected" before the corrupted return address could be used.

### Heap overflows and use-after-free

Heap blocks sit next to other blocks and allocator metadata. Overflowing one can corrupt another object (for example, a function pointer in a struct) or the allocator's free lists; use-after-free (Unit 3.3) lets an attacker allocate a different object into the freed memory and have the program use it through the stale pointer. These are the dominant vulnerability classes in browsers today. Defences: AddressSanitizer in testing, hardened allocators, and — fundamentally — code whose ownership rules make the bugs impossible (P-4, P-5).

### Deep & Tricky

**DT-88. An off-by-one write of a single zero byte can be exploitable.** *(Security fact)* `buf[sizeof buf] = '\0'` or `strncat` miscalculations write one byte past a buffer. If that byte is the low byte of a saved frame pointer or of the size field in a neighbouring heap chunk's metadata, attackers have repeatedly turned it into full control of the program. There is no "small" out-of-bounds write.

### Mastery check

1. **Explain.** Why did `overflow` print `is_admin=66`, and why would `-fstack-protector` not have detected this overflow?

## Unit 12.2 — Mitigations: what the toolchain and kernel do

**One-line summary:** Defence in depth makes exploitation of memory bugs hard: **stack canaries** detect stack overflows, **non-executable memory (NX)** stops injected code from running, **ASLR** and **PIE** randomise addresses, **RELRO** makes function-pointer tables read-only, **`_FORTIFY_SOURCE`** checks many library calls, and **CET/shadow stacks** protect return addresses in hardware. Check that your binaries have them.

| Mitigation | Flag / mechanism | Stops or hinders |
| --- | --- | --- |
| Stack canaries | `-fstack-protector-strong` (default on Ubuntu, Fedora) | overwriting return addresses via linear stack overflow |
| NX / DEP | default; violated by executable-stack objects (Unit 11.2) | executing injected code from stack or heap |
| ASLR | kernel; `/proc/sys/kernel/randomize_va_space = 2` | predicting addresses of stack, heap, libraries |
| PIE | `-fPIE -pie` (default on modern distros) | predicting the address of the program's own code |
| RELRO | `-Wl,-z,relro,-z,now` | overwriting the GOT (dynamic-linking function pointers) |
| `_FORTIFY_SOURCE=2` or `=3` | needs `-O1`+; default on Ubuntu | overflows in `strcpy`, `memcpy`, `sprintf`, … with known sizes; `%n` in writable formats |
| CET (shadow stack, IBT) | `-fcf-protection` + CPU and kernel support | return-address overwrites; jumps to arbitrary code |
| Stack clash protection | `-fstack-clash-protection` | jumping over guard pages with huge stack allocations |

```c
// file: hardening/hello.c
#include <stdio.h>
int main(void) { puts("hello"); return 0; }
```

```console
$ cd hardening && gcc -O2 hello.c -o default_build && gcc -O2 -fno-stack-protector -no-pie -Wl,-z,norelro -z execstack hello.c -o weak_build 2>/dev/null
$ cd hardening && for b in default_build weak_build; do printf '%-14s ' $b; readelf -h $b | grep -q 'DYN' && printf 'PIE ' || printf 'no-PIE '; readelf -l $b | grep -A1 GNU_STACK | grep -q RWE && printf 'exec-stack ' || printf 'NX '; readelf -l $b | grep -q GNU_RELRO && printf 'RELRO ' || printf 'no-RELRO '; readelf -d $b | grep -q BIND_NOW && printf '(full) ' || printf ''; nm -D $b | grep -q __stack_chk_fail && echo canary || echo no-canary; done
default_build  PIE NX RELRO (full) no-canary
weak_build     no-PIE exec-stack no-RELRO no-canary
```

(`hello.c` has no arrays, so even the default build does not need `__stack_chk_fail`: `-fstack-protector-strong` only instruments functions with local arrays or address-taken locals.) Distributions build packages with all of these on; for your own projects, add them explicitly in release builds, and add `-D_FORTIFY_SOURCE=3 -fstack-clash-protection -fcf-protection` where supported.

Mitigations are not fixes: each has known bypasses (information leaks defeat ASLR; data-only attacks like Unit 12.1's `is_admin` ignore NX and canaries). They buy time and raise cost; the bugs must still be removed.

### Mastery check

1. **Explain.** Why does ASLR require PIE to randomise the program's own code addresses?

## Topic 12B — Bug classes beyond overflows

## Unit 12.3 — Integer bugs that become memory bugs

**One-line summary:** Integer overflow, signedness confusion, and truncation in **size calculations** produce buffers that are too small, followed by in-bounds-looking writes that overflow them. Every computation that feeds `malloc`, a length check, or an index must be checked for overflow and range.

```c
// file: intbugs.c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* Vulnerable: count * size can wrap around, producing a tiny allocation. */
static void *alloc_records_bad(uint32_t count, uint32_t size) {
    uint32_t total = count * size;                    /* 32-bit wrap */
    printf("  bad:  count=%u size=%u -> allocating %u bytes\n", count, size, total);
    return malloc(total);
}

/* Fixed: check before multiplying, in the type used for the allocation. */
static void *alloc_records_good(uint32_t count, uint32_t size) {
    if (size != 0 && count > SIZE_MAX / size) {
        printf("  good: count=%u size=%u -> rejected (overflow)\n", count, size);
        return NULL;
    }
    size_t total = (size_t)count * size;
    printf("  good: count=%u size=%u -> allocating %zu bytes\n", count, size, total);
    return malloc(total);
}

/* Vulnerable: a signed length passes the check when negative, then becomes huge. */
static int copy_bad(char *dst, size_t dstsz, const char *src, int len) {
    if (len > (int)dstsz)                             /* -1 passes this check */
        return -1;
    printf("  copy_bad would memcpy %zu bytes into a %zu-byte buffer\n", (size_t)len, dstsz);
    return 0;                                         /* (not actually copying) */
}

int main(void) {
    /* 0x10000001 records of 16 bytes = 2^32 + 16 bytes: wraps to 16 in 32 bits */
    free(alloc_records_bad(0x10000001u, 16));
    free(alloc_records_good(0x10000001u, 16));
    char buf[32];
    copy_bad(buf, sizeof buf, "x", -1);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra intbugs.c -o intbugs && ./intbugs
intbugs.c: In function ‘copy_bad’:
intbugs.c:26:27: warning: unused parameter ‘dst’ [-Wunused-parameter]
   26 | static int copy_bad(char *dst, size_t dstsz, const char *src, int len) {
      |                     ~~~~~~^~~
intbugs.c:26:58: warning: unused parameter ‘src’ [-Wunused-parameter]
   26 | static int copy_bad(char *dst, size_t dstsz, const char *src, int len) {
      |                                              ~~~~~~~~~~~~^~~
  bad:  count=268435457 size=16 -> allocating 16 bytes
  good: count=268435457 size=16 -> allocating 4294967312 bytes
  copy_bad would memcpy 18446744073709551615 bytes into a 32-byte buffer
```

The "bad" allocator would hand back a 16-byte buffer for what the caller believes are 268 million records; the first loop that fills them overflows the heap. The signed-length check accepted −1, which `memcpy` would read as `SIZE_MAX`. Both patterns have caused real remote-code-execution vulnerabilities in image decoders, font parsers, and network stacks.

Rules: use `size_t` for sizes and counts; validate lengths from untrusted input against the actual remaining input and the destination size; check multiplications (`count > SIZE_MAX / size`, `__builtin_mul_overflow`, C23 `ckd_mul`); compile with `-Wconversion -Wsign-conversion` to find silent truncations; run tests with `-fsanitize=undefined,integer` (Clang) to catch them at run time.

<!-- expanded -->
### Going deeper: checked arithmetic for sizes

The safe way to compute a size from untrusted numbers is to let the compiler check each operation. GCC and Clang provide `__builtin_add_overflow`, `__builtin_sub_overflow` and `__builtin_mul_overflow`, which compute the mathematically exact result, store the (possibly wrapped) value, and return `true` if it did not fit. C23 standardises the same idea as `ckd_add`, `ckd_sub` and `ckd_mul` in `<stdckdint.h>` (GCC 14 and Clang 18 ship it; this book's GCC 13 does not yet).

```c
// file: ckdsize.c
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

/* allocate count records of size bytes each, plus a header; NULL if any step overflows */
static void *alloc_records(size_t header, size_t count, size_t size) {
    size_t body, total;
    if (__builtin_mul_overflow(count, size, &body) || __builtin_add_overflow(header, body, &total))
        return NULL;
    return malloc(total);
}

int main(void) {
    size_t count = SIZE_MAX / 8 + 2, size = 8;                 /* count * size wraps around to 8 */
    printf("count * size computed naively = %zu\n", count * size);
    void *naive = malloc(count * size);                        /* asks for 8 bytes, "succeeds" */
    printf("malloc(count * size): %s\n", naive ? "returned a tiny buffer - every later write overflows it" : "NULL");
    free(naive);

    void *checked = alloc_records(16, count, size);
    printf("checked allocation: %s\n", checked ? "succeeded" : "refused (overflow detected)");
    void *z = calloc(count, size);                             /* calloc must check the product itself */
    printf("calloc(count, size): %s\n", z ? "succeeded" : "NULL (calloc detected the overflow)");
    free(z);

    void *ok = alloc_records(16, 1000, 24);
    printf("alloc_records(16, 1000, 24): %s\n", ok ? "succeeded" : "refused");
    free(ok);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra ckdsize.c -o ckdsize && ./ckdsize
ckdsize.c: In function ‘main’:
ckdsize.c:24:15: warning: product ‘2305843009213693953 * 8’ of arguments 1 and 2 exceeds ‘SIZE_MAX’ [-Walloc-size-larger-than=]
   24 |     void *z = calloc(count, size);                             /* calloc must check the product itself */
      |               ^~~~~~~~~~~~~~~~~~~
In file included from ckdsize.c:5:
/usr/include/stdlib.h:675:14: note: in a call to allocation function ‘calloc’ declared here
  675 | extern void *calloc (size_t __nmemb, size_t __size)
      |              ^~~~~~
count * size computed naively = 8
malloc(count * size): returned a tiny buffer - every later write overflows it
checked allocation: refused (overflow detected)
calloc(count, size): NULL (calloc detected the overflow)
alloc_records(16, 1000, 24): succeeded
```

GCC even warned about the `calloc` call at compile time — here the numbers are constants it can see. It said nothing about `malloc(count * size)`, where the product had already wrapped to a harmless-looking 8; and with sizes read from a file, neither warning is possible.

### Deep & Tricky

**DT-89. `calloc(n, size)` must fail if `n * size` overflows, but `malloc(n * size)` cannot know that an overflow happened — the multiplication was done, and wrapped, before `malloc` was called. So replacing `calloc` with `malloc` "for speed" can turn an allocation failure into a heap overflow, and so can `realloc(p, n * size)`.** *(Guarantee: C17 §7.22.3.2 — `calloc` allocates space "for an array of nmemb objects, each of whose size is size", which an implementation cannot do if the product is not representable; glibc and every modern C library check it. glibc 2.26+ provides `reallocarray(p, n, size)` with the same check for `realloc`)* Several real vulnerabilities came from exactly this pattern in image and font decoders: a width times a height times a pixel size, supplied by the file, overflowing to a small allocation.

### Mastery check

1. **Write.** A safe `void *array_alloc(size_t count, size_t size, size_t header)` that allocates `header + count * size` bytes or returns `NULL` on any overflow.

## Unit 12.4 — Format strings and injection

**One-line summary:** Passing untrusted data as a `printf` **format string** lets an attacker read memory with `%x`/`%s` and, with `%n`, write it. Passing untrusted data to a **shell** (`system`, `popen`) lets them run commands. The rule for both: untrusted data is always *data* — an argument, never part of the code or command that interprets it.

```c
// file: fmtbug.c
#include <stdio.h>

int main(int argc, char *argv[]) {
    int secret = 0x5EC12E7;
    const char *input = argc > 1 ? argv[1] : "hello";
    printf("safe:   ");
    printf("%s", input);            /* data as an argument */
    printf("\nunsafe: ");
    printf(input);                  /* data as the format: BUG */
    printf("\n");
    (void)secret;
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wformat-security fmtbug.c -o fmtbug 2>&1 | grep -E 'warning' | sed 's/^fmtbug.c://'
10:5: warning: format not a string literal and no format arguments [-Wformat-security]
$ gcc -std=c17 -O0 -D_FORTIFY_SOURCE=0 fmtbug.c -o fmtbug 2>/dev/null && ./fmtbug '%x.%x.%x.%x.%x.%x.%x.%x'
safe:   %x.%x.%x.%x.%x.%x.%x.%x
unsafe: 415d02a0.0.0.73.0.e3501338.70e69ae0.e35012f0
$ gcc -std=c17 -O2 -D_FORTIFY_SOURCE=2 fmtbug.c -o fmtbug_fort 2>/dev/null && ./fmtbug_fort 'abc%n'; echo "exit status $?"
safe:   abc%n
*** %n in writable segment detected ***
bash: line 1:  5580 Aborted                 ./fmtbug_fort 'abc%n'
exit status 134
```

The unsafe call printed values from registers and the stack — memory the user should never see (here it is hard to pick out the secret, but leaked pointers alone are enough to defeat ASLR). With fortification, glibc refuses `%n` in a format string located in writable memory and aborts. The compiler's `-Wformat-security` warning (enabled in Ubuntu's default GCC and by `-Wformat=2`) flags every non-literal format with no arguments. Fix: `printf("%s", input)` or `fputs(input, stdout)`.

### Command injection

```c
// file: inject.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

/* BAD: the file name is pasted into a shell command line. */
static void count_lines_bad(const char *file) {
    char cmd[256];
    snprintf(cmd, sizeof cmd, "wc -l < %s", file);
    system(cmd);
}

/* GOOD: no shell; the file name is one argument, whatever characters it contains. */
static void count_lines_good(const char *file) {
    pid_t pid = fork();
    if (pid == 0) {
        execlp("wc", "wc", "-l", "--", file, (char *)NULL);   /* "--": file can't be an option */
        _exit(127);
    }
    waitpid(pid, NULL, 0);
}

int main(int argc, char *argv[]) {
    const char *file = argc > 1 ? argv[1] : "/etc/hostname";
    printf("bad:  ");
    fflush(stdout);
    count_lines_bad(file);
    printf("good: ");
    fflush(stdout);
    count_lines_good(file);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall inject.c -o inject && ./inject '/etc/hostname; echo INJECTED COMMAND RAN'
bad:  1
INJECTED COMMAND RAN
good: wc: '/etc/hostname; echo INJECTED COMMAND RAN': No such file or directory
```

The shell parsed the `;` in the "file name" and ran a second command; the `exec` version passed the whole string as one argument, and `wc` simply reported that no such file exists. The same principle governs SQL (use parameterised queries), HTML (escape on output), and file paths (validate, and open relative to a trusted directory — DT-70).

### Mastery check

1. **Explain.** Why is quoting the argument (`"wc -l < '%s'"`) not a reliable fix?

## Unit 12.5 — Secure coding practices

**One-line summary:** Treat all external input as hostile, validate it completely at the boundary, handle secrets so they don't linger in memory, use the OS's cryptographically secure random source, run with the least privilege needed, and follow a published standard such as **SEI CERT C**. Prefer designs that make whole bug classes impossible over vigilance at every line.

```c
// file: secure.c
#define _GNU_SOURCE
#include <stdio.h>
#include <string.h>
#include <sys/random.h>
#include <unistd.h>

/* constant_time_equal compares without an early exit, so timing reveals nothing
   about where the first difference is. */
static int constant_time_equal(const unsigned char *a, const unsigned char *b, size_t n) {
    unsigned char diff = 0;
    for (size_t i = 0; i < n; i++)
        diff |= a[i] ^ b[i];
    return diff == 0;
}

int main(void) {
    unsigned char token[16];
    if (getrandom(token, sizeof token, 0) != (ssize_t)sizeof token)   /* kernel CSPRNG */
        return 1;
    printf("session token: ");
    for (size_t i = 0; i < sizeof token; i++)
        printf("%02x", token[i]);
    printf(" (from getrandom; never use rand() for this)\n");

    char password[64];
    snprintf(password, sizeof password, "hunter2");          /* pretend it came from input */
    unsigned char a[4] = {1, 2, 3, 4}, b[4] = {1, 2, 3, 5};
    printf("constant-time compare: %d %d\n", constant_time_equal(a, a, 4), constant_time_equal(a, b, 4));
    explicit_bzero(password, sizeof password);                 /* memset could be optimised away */
    printf("password buffer cleared: first byte = %d\n", password[0]);
    printf("running as uid %d (a server should drop root privileges after setup)\n", (int)getuid());
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -O2 secure.c -o secure && ./secure | sed 's/token: [0-9a-f]*/token: <32 hex digits>/'
session token: <32 hex digits> (from getrandom; never use rand() for this)
constant-time compare: 1 0
password buffer cleared: first byte = 0
running as uid 0 (a server should drop root privileges after setup)
```

The practices in that program:

- **Randomness for security** comes from the kernel (`getrandom`, `/dev/urandom`, or `arc4random` on BSDs and glibc 2.36+) — never from `rand` (predictable) or the time.
- **Secret comparison** must not exit at the first difference (`memcmp` does), or the time taken leaks how many leading bytes are right.
- **Clearing secrets:** a `memset` of a buffer that is never read again is a "dead store" the compiler may delete; `explicit_bzero` (or C23's `memset_explicit`) is guaranteed to happen.
- **Least privilege:** start as root only if required (binding port 80), then drop to an unprivileged user with `setgid`/`setuid` (in that order), or use capabilities, containers, and seccomp filters to limit what a compromised process can do.

### Rules worth memorising (from SEI CERT C and experience)

1. Validate every input at the trust boundary: length, range, format, and encoding — and reject rather than repair.
2. Never use unbounded functions: `gets`, `strcpy`, `strcat`, `sprintf`, `scanf("%s")`.
3. Check every allocation size for overflow, every return value, every index against its bound.
4. Initialise every variable; set pointers to `NULL` after `free`.
5. Keep untrusted data out of format strings, shell commands, and paths.
6. Compile with warnings as errors, hardening flags, and test under sanitizers and fuzzers.
7. Minimise the attack surface: small parsers, few privileges, isolated processes for risky work (the OpenSSH privilege-separation model).

### Pattern

**P-12. Parse untrusted input with a small, fuzzed, bounds-checked parser that produces validated structures.** *(Practice)* Concentrate all handling of raw untrusted bytes in one module whose only job is to turn them into well-formed internal data — checking every length against the remaining input — and fuzz that module continuously. The rest of the program then handles only validated data. *Trade-off:* an extra layer of code and data copying, in exchange for a small, auditable attack surface.

<!-- expanded -->
### Deep & Tricky

**DT-90. Checking a path for `..` with string operations does not make it safe to open: symbolic links, absolute paths, encodings (Unit 6.8's overlong forms, `%2e%2e` in URLs), and a file replaced between the check and the `open` all bypass it. On Linux, the robust tool is `openat2` with `RESOLVE_BENEATH`, which makes the kernel refuse any path that escapes the given directory — during the lookup itself.** *(Linux `openat2(2)`: `RESOLVE_BENEATH` "Do not permit the path resolution to succeed if any component of the resolution is not a descendant of the directory indicated by dirfd")* The HTTP server of Chapter 10 rejected `..` by string checks and tests; a server exposed to the internet should also confine itself with `openat2`, a `chroot`/mount namespace, or at least an `O_NOFOLLOW` open followed by `fstat` checks.

```c
// file: beneath.c
#define _GNU_SOURCE
#include <errno.h>
#include <fcntl.h>
#include <linux/openat2.h>
#include <stdio.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/syscall.h>
#include <unistd.h>

static void try(int root, const char *path) {
    struct open_how how = { .flags = O_RDONLY, .resolve = RESOLVE_BENEATH };
    long fd = syscall(SYS_openat2, root, path, &how, sizeof how);
    printf("  %-22s %s\n", path, fd >= 0 ? "opened" : strerror(errno));
    if (fd >= 0) close((int)fd);
}

int main(void) {
    mkdir("www", 0755);
    FILE *f = fopen("www/index.html", "w");
    if (f) { fputs("hi\n", f); fclose(f); }
    unlink("www/escape");
    if (symlink("/etc/hostname", "www/escape") < 0) perror("symlink");
    int root = open("www", O_DIRECTORY | O_RDONLY);
    printf("openat2 with RESOLVE_BENEATH, relative to www/:\n");
    try(root, "index.html");
    try(root, "../beneath.c");
    try(root, "/etc/hostname");
    try(root, "escape");                     /* a symlink pointing outside: no string check would see it */
    close(root);
    return 0;
}
```

```console
$ gcc -std=c17 -Wall -Wextra beneath.c -o beneath && ./beneath
openat2 with RESOLVE_BENEATH, relative to www/:
  index.html             opened
  ../beneath.c           Invalid cross-device link
  /etc/hostname          Invalid cross-device link
  escape                 Invalid cross-device link
```

The symbolic link `escape` contains no `..` at all — a string check on the requested name would accept it — yet the kernel refused to follow it outside `www/` (`EXDEV`, "Invalid cross-device link", is the error `openat2` uses for an escape).

## Topic 12C — Finding bugs automatically

## Unit 12.6 — Fuzzing

**One-line summary:** A **fuzzer** generates huge numbers of inputs, runs the code under test with each, and keeps the inputs that reach new code paths (**coverage-guided** fuzzing), mutating them further. Combined with sanitizers, it finds crashes, overflows, leaks, and UB that no human-written test thought of. **libFuzzer** (in Clang) needs only one function: `LLVMFuzzerTestOneInput(const uint8_t *data, size_t size)`.

```c
// file: fuzzdemo/parse_kv.c
/* parse_kv parses "key=value;key=value" and returns the number of pairs.
   It contains a bug that only a specific input shape triggers. */
#include <stddef.h>
#include <stdint.h>
#include <string.h>

int parse_kv(const uint8_t *data, size_t size) {
    char key[8];
    int pairs = 0;
    size_t i = 0;
    while (i < size) {
        size_t k = 0;
        while (i < size && data[i] != '=') {
            if (k < sizeof key)          /* bounds check on the key ... */
                key[k] = (char)data[i];
            k++;
            i++;
        }
        if (i == size)
            break;
        key[k] = '\0';                   /* ... but not here: k can be >= 8 */
        i++;                             /* skip '=' */
        while (i < size && data[i] != ';')
            i++;
        i++;                             /* skip ';' */
        pairs++;
    }
    return pairs;
}
```

```c
// file: fuzzdemo/fuzz_kv.c
#include <stddef.h>
#include <stdint.h>

int parse_kv(const uint8_t *data, size_t size);

int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    parse_kv(data, size);
    return 0;                            /* 0: input processed; crashes are reported by sanitizers */
}
```

```console
$ cd fuzzdemo && clang -g -O1 -fsanitize=fuzzer,address,undefined parse_kv.c fuzz_kv.c -o fuzz_kv && mkdir -p corpus && printf 'a=1;b=2' > corpus/seed && ./fuzz_kv corpus -max_total_time=60 2>&1 | grep -E 'ERROR|WRITE of|#[01] .*parse_kv|Test unit written' | sed 's/crash-[0-9a-f]*/crash-<hash>/' | head -n 4
==5610==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ff2e2d048a8 at pc 0x5608b571b9d3 bp 0x7fff8e955d70 sp 0x7fff8e955d68
WRITE of size 1 at 0x7ff2e2d048a8 thread T0
    #0 0x5608b571b9d2 in parse_kv fuzzdemo/parse_kv.c:22:16
    #0 0x5608b571b367 in parse_kv fuzzdemo/parse_kv.c:8
$ cd fuzzdemo && od -c crash-* | head -n 2
0000000   a 260 377   ?   ;   ?   b   b   =
0000011
```

Starting from one seed (`a=1;b=2`), libFuzzer mutated inputs until one had a key of 8 or more characters followed by `=`: the terminator write `key[k] = '\0'` landed past the array, and AddressSanitizer reported a stack-buffer-overflow. libFuzzer saved the input to a `crash-…` file, which reproduces the bug deterministically (`./fuzz_kv crash-…`) and becomes a regression test once fixed. Fuzzing found it in seconds; a human-written test suite with "normal" keys would never have.

Fuzzing in practice:

- **Harnesses** should be fast (thousands of executions per second), deterministic, and free of global state between runs.
- **Seed corpora** of real inputs (and dictionaries of keywords, `-dict=`) help the fuzzer reach deep paths quickly.
- **Run continuously:** Google's OSS-Fuzz runs fuzzers for over a thousand open-source projects and has found tens of thousands of bugs. AFL++ and honggfuzz are alternatives to libFuzzer; all use the same harness idea.
- **Structure-aware fuzzing** generates valid structured inputs (e.g. protocol messages) when random bytes rarely get past the first checksum.

### Mastery check

1. **Explain.** Why does a fuzzer find the `key[k]` bug quickly, when random testing with uniformly random strings would rarely produce a long key followed by `=`?

## Chapter 12 Project — Fuzz and harden a binary parser

**Goal:** a parser for a small binary **TLV** (type–length–value) message format, written the way such parsers often are in real code — and containing several memory-safety bugs. Write a fuzz harness, find the bugs with libFuzzer and sanitizers, fix them, and show the fixed parser surviving a sustained fuzzing run, with the crashing inputs kept as regression tests.

### The format

```
message := magic(2 bytes: 'T','L') count(1 byte) record*count
record  := type(1 byte) length(2 bytes, big-endian) value(length bytes)
types   : 1 = name (string), 2 = age (1 byte), 3 = tags (list of 1-byte-length strings)
```

### The first version (as it might be written in a hurry)

```c
// file: tlv/tlv_v1.c
#include <stdint.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[32];
    int age;
    char *tags[8];
    int ntags;
} Person;

int tlv_parse(const uint8_t *p, size_t n, Person *out) {
    memset(out, 0, sizeof *out);
    if (n < 3 || p[0] != 'T' || p[1] != 'L')
        return -1;
    int count = p[2];
    size_t off = 3;
    for (int r = 0; r < count; r++) {
        uint8_t type = p[off];                               /* bug A: off may be >= n */
        uint16_t len = (uint16_t)(p[off + 1] << 8 | p[off + 2]);
        off += 3;
        const uint8_t *v = p + off;                          /* bug B: len not checked vs n */
        switch (type) {
        case 1:
            memcpy(out->name, v, len);                       /* bug C: len not checked vs 32 */
            out->name[len] = '\0';
            break;
        case 2:
            out->age = v[0];
            break;
        case 3:
            for (size_t t = 0; t < len;) {
                uint8_t tl = v[t];
                out->tags[out->ntags] = malloc(tl + 1);      /* bug D: ntags not checked vs 8 */
                memcpy(out->tags[out->ntags], v + t + 1, tl);/* bug E: tl not checked vs len */
                out->tags[out->ntags][tl] = '\0';
                out->ntags++;
                t += 1u + tl;
            }
            break;
        }
        off += len;
    }
    return 0;
}

void tlv_free(Person *p) {
    for (int i = 0; i < p->ntags; i++)
        free(p->tags[i]);
}
```

```c
// file: tlv/fuzz_tlv.c
#include <stddef.h>
#include <stdint.h>
#include <stdlib.h>

typedef struct { char name[32]; int age; char *tags[8]; int ntags; } Person;
int tlv_parse(const uint8_t *p, size_t n, Person *out);
void tlv_free(Person *p);

int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    Person p;
    if (tlv_parse(data, size, &p) == 0)
        tlv_free(&p);
    return 0;
}
```

A seed corpus of valid messages helps the fuzzer get past the magic bytes:

```c
// file: tlv/mkseed.c
#include <stdio.h>
int main(void) {
    const unsigned char msg[] = {'T', 'L', 3,
        1, 0, 5, 'a', 'l', 'i', 'c', 'e',          /* name "alice" */
        2, 0, 1, 30,                                /* age 30 */
        3, 0, 8, 3, 'c', 'o', 'w', 3, 'd', 'o', 'g'};   /* tags "cow", "dog" */
    FILE *f = fopen("seeds/valid.tlv", "wb");
    if (f == NULL) return 1;
    fwrite(msg, 1, sizeof msg, f);
    return fclose(f) != 0;
}
```

```console
$ cd tlv && mkdir -p seeds crashes && gcc mkseed.c -o mkseed && ./mkseed && clang -g -O1 -fsanitize=fuzzer,address,undefined tlv_v1.c fuzz_tlv.c -o fuzz_v1
$ cd tlv && rm -f crashes/* && for i in 1 2 3 4 5 6 7 8; do ./fuzz_v1 -runs=3000000 -seed=$i -artifact_prefix=crashes/v1-$i- seeds > run$i.log 2>&1; grep -m1 -E '^    #[01] .*tlv_v1.c' run$i.log | grep -oE 'tlv_v1.c:[0-9]+'; done | sort | uniq -c && ls crashes | wc -l
      5 tlv_v1.c:20
      1 tlv_v1.c:26
      2 tlv_v1.c:36
8
```

Eight short fuzzing campaigns with different random seeds each stopped at the first bug they hit. In the run shown, all eight stopped at line 20: bug A, reading the next record's header past the end of the input — the shallowest bug, reachable by almost any malformed message. (In earlier runs while writing this chapter, some campaigns first hit line 21, line 26 — bug B, a name value running past the input — or line 36, bug E, a tag running past its record.) Every crash was a heap-buffer-overflow *read*: libFuzzer passes each input in a heap buffer of exactly its size, so reads past the input are caught immediately. Bugs C and D (writing past `name` and past `tags`) are still there, hidden behind the shallow ones; fixing A and B and fuzzing again exposes them — the normal rhythm of fuzzing. Each campaign saved its crashing input in `crashes/` as a minimal reproducer. (The log goes to a file before `grep` reads it: piping libFuzzer straight into `grep -m1` kills the fuzzer with `SIGPIPE` before it writes the crash file — which is how the first version of this command lost every reproducer.)

### The hardened version

```c
// file: tlv/tlv_v2.c
#include <stdint.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[32];
    int age;
    char *tags[8];
    int ntags;
} Person;

enum { TLV_OK = 0, TLV_ERR = -1 };

/* A cursor over untrusted bytes: every read is checked against what remains (P-12). */
typedef struct { const uint8_t *p; size_t left; } Cursor;

static int take(Cursor *c, size_t n, const uint8_t **out) {
    if (n > c->left)
        return TLV_ERR;
    *out = c->p;
    c->p += n;
    c->left -= n;
    return TLV_OK;
}

static int take_u8(Cursor *c, uint8_t *v) {
    const uint8_t *b;
    if (take(c, 1, &b) != TLV_OK) return TLV_ERR;
    *v = b[0];
    return TLV_OK;
}

static int take_u16be(Cursor *c, uint16_t *v) {
    const uint8_t *b;
    if (take(c, 2, &b) != TLV_OK) return TLV_ERR;
    *v = (uint16_t)(b[0] << 8 | b[1]);
    return TLV_OK;
}

void tlv_free(Person *p) {
    for (int i = 0; i < p->ntags; i++)
        free(p->tags[i]);
    p->ntags = 0;
}

static int parse_tags(Person *out, Cursor *c) {
    while (c->left > 0) {
        uint8_t tl;
        const uint8_t *s;
        if (out->ntags == (int)(sizeof out->tags / sizeof out->tags[0]))
            return TLV_ERR;                                   /* fixes D */
        if (take_u8(c, &tl) != TLV_OK || take(c, tl, &s) != TLV_OK)
            return TLV_ERR;                                   /* fixes E */
        char *tag = malloc((size_t)tl + 1);
        if (tag == NULL)
            return TLV_ERR;
        memcpy(tag, s, tl);
        tag[tl] = '\0';
        out->tags[out->ntags++] = tag;
    }
    return TLV_OK;
}

int tlv_parse(const uint8_t *p, size_t n, Person *out) {
    memset(out, 0, sizeof *out);
    Cursor c = {p, n};
    const uint8_t *magic;
    uint8_t count;
    if (take(&c, 2, &magic) != TLV_OK || magic[0] != 'T' || magic[1] != 'L' ||
        take_u8(&c, &count) != TLV_OK)
        return TLV_ERR;
    for (int r = 0; r < count; r++) {
        uint8_t type;
        uint16_t len;
        const uint8_t *v;
        if (take_u8(&c, &type) != TLV_OK || take_u16be(&c, &len) != TLV_OK ||   /* fixes A */
            take(&c, len, &v) != TLV_OK)                                        /* fixes B */
            goto fail;
        Cursor value = {v, len};
        switch (type) {
        case 1:
            if (len >= sizeof out->name)                                         /* fixes C */
                goto fail;
            memcpy(out->name, v, len);
            out->name[len] = '\0';
            break;
        case 2:
            if (len != 1)
                goto fail;
            out->age = v[0];
            break;
        case 3:
            if (parse_tags(out, &value) != TLV_OK)
                goto fail;
            break;
        default:
            goto fail;                                       /* unknown type: reject, don't guess */
        }
    }
    if (c.left != 0)
        goto fail;                                           /* trailing garbage */
    return TLV_OK;
fail:
    tlv_free(out);                                           /* no leaks on any error path */
    return TLV_ERR;
}
```

The structural change matters more than the individual fixes: all reads of untrusted bytes go through a `Cursor` whose `take` function is the *only* place that checks lengths. There is no arithmetic like `off + len` that could be forgotten or overflow; a record's value gets its own sub-cursor, so tag parsing cannot read beyond its record. On any error, everything allocated so far is freed.

```console
$ cd tlv && clang -g -O1 -fsanitize=fuzzer,address,undefined tlv_v2.c fuzz_tlv.c -o fuzz_v2 && ./fuzz_v2 crashes/* 2>&1 | grep -cE '^Executed .* in [0-9]+ ms'
8
$ cd tlv && ./fuzz_v2 -max_total_time=120 -print_final_stats=1 seeds 2>&1 | grep -E 'Done|stat::number_of_executed_units|ERROR'
Done 74207013 runs in 121 second(s)
stat::number_of_executed_units: 74207013
```

The hardened parser ran every crash file found by the earlier campaigns without an error (the count is the number of files executed cleanly), then survived two minutes of fuzzing — millions of executions — under AddressSanitizer and UndefinedBehaviorSanitizer with no findings. The crash files stay in the repository as a **regression corpus**: CI runs the fuzzer over them on every change (`./fuzz_v2 crashes/*` runs each file once and exits; passing a *directory* instead starts an open-ended fuzzing campaign — the first version of this command did that and ran until it was killed), so a reintroduced bug fails immediately.

### Review checklist

- Is there any read from the input not routed through a bounds-checked function?
- Are all integer computations on lengths impossible to overflow (no `a + b` on untrusted values without checks)?
- Do unknown types, trailing bytes, and oversize fields cause rejection rather than partial acceptance?
- Is the fuzzer part of CI, with the crash corpus as regression tests?

## Answers to the Chapter 12 prediction questions

There are no prediction questions in this chapter; the explanations are in the units' text.

# Chapter 13 — Capstone: A Persistent Key-Value Database Server

*Goal: design, build, test, measure, and defend a real systems program — a networked, persistent key-value store — using everything in this book.*

This chapter is different from the others. Instead of new language features, it asks you to combine them into one system and to make the engineering decisions yourself. The system is **kvd**, a small relative of Redis: an in-memory key-value database served over TCP by an event loop, made durable by a **write-ahead log** and **snapshots** taken in a forked child process, and able to recover from crashes — including a crash in the middle of writing a log record.

A complete reference implementation follows, built and tested while writing this book. It is deliberately small (about 900 lines) so that you can read all of it; your capstone should go further. Each unit states the requirements, the design decisions and their trade-offs, the code, and the evidence that it works.

**Capstone deliverables**

1. A design document (Unit 13.1).
2. The server, with storage engine, recovery, and protocol (Units 13.2–13.5).
3. A test suite: protocol tests, crash-recovery tests, corruption tests, and sanitizer runs (Unit 13.6).
4. A performance report from a load test, comparing durability settings (Unit 13.7).
5. An operations note and an architecture review (Unit 13.8).

## Topic 13A — Design

## Unit 13.1 — Requirements and design

**One-line summary:** Write down what the system must do, how well, and how it fails — before writing code. For a database, the hard questions are about **durability** (what survives a crash?), **consistency** (what does a client see?), and **performance** (latency and throughput under load).

### Requirements

- **Data model:** string keys (up to 255 bytes, no spaces) to string values (up to 64 KiB).
- **Commands:** `PING`, `SET key value`, `GET key`, `DEL key`, `INCR key`, `DBSIZE`, `BGSAVE`, `INFO`, `QUIT`.
- **Protocol:** one command per line; one reply line per command; pipelining allowed.
- **Durability:** every acknowledged write survives a process crash (`kill -9`) when `--fsync always` is chosen; with `--fsync everysec`, at most about one second of acknowledged writes may be lost on power failure; `--fsync no` leaves flushing to the OS.
- **Recovery:** on start, load the latest snapshot and replay the log; a torn final record (crash mid-write) is detected and discarded, never misread.
- **Performance:** tens of thousands of operations per second per core with `everysec`; the event loop must never block on clients.
- **Safety:** bounded memory per connection; malformed input gets an error, never a crash; clean shutdown on `SIGTERM`.

### Key decisions

| Decision | Options considered | Choice and reason |
| --- | --- | --- |
| Concurrency | thread per client; event loop | **single-threaded `epoll` loop** (Chapter 10): no locks on the data, simple reasoning; one core is enough for the target |
| Index | balanced tree; hash table | **open-addressing hash table** (Chapter 7): O(1) average, no ordering needed |
| Durability | write data files in place; log + snapshot | **append-only write-ahead log + periodic snapshot**: sequential writes, simple recovery, the design of Redis and many databases |
| Snapshot without stopping | lock and copy; `fork` | **`fork`** (Chapter 8): the child gets a copy-on-write image of the table and writes it while the parent keeps serving |
| Log integrity | trust the file; checksums | **CRC-32 per record** (Unit 13.3): detects torn and corrupted records |
| Protocol | binary; Redis RESP; text lines | **text lines**: testable with `nc` and shell scripts; RESP is an extension |

### Failure modes to design for

| Failure | Design response |
| --- | --- |
| Process killed mid-write | log records have CRCs; recovery truncates the log at the first bad record |
| Crash during snapshot | snapshot written to a temporary file, `fsync`ed, then `rename`d (Unit 6.3) |
| Disk full / write error | the write is not acknowledged; the client gets `-ERR` |
| Slow or malicious client | per-connection buffer limits; idle timeout; non-blocking I/O |
| Very long line | rejected with an error, connection closed |
| Memory exhaustion | `SET` fails with `-ERR` when allocation fails (and `maxmemory` is an extension) |

## Topic 13B — Build

## Unit 13.2 — The in-memory table

**One-line summary:** The table is Chapter 7's open-addressing hash map, specialised to own both keys and values, with tombstone deletion and iteration for snapshots.

```c
// file: kvd/table.h
#ifndef TABLE_H
#define TABLE_H

#include <stddef.h>
#include <stdint.h>

typedef struct {
    char *key, *val;            /* key == NULL: empty; key == TOMB: deleted */
    uint32_t klen, vlen;
    uint64_t hash;
} Entry;

typedef struct {
    Entry *slots;
    size_t cap, len, used;      /* used = live + tombstones */
} Table;

int table_init(Table *t);
void table_free(Table *t);
/* table_get returns the entry for key or NULL. */
const Entry *table_get(const Table *t, const char *key, uint32_t klen);
/* table_set stores copies of key and value. Returns 0 or -1 (out of memory). */
int table_set(Table *t, const char *key, uint32_t klen, const char *val, uint32_t vlen);
/* table_del returns 1 if the key existed, 0 if not. */
int table_del(Table *t, const char *key, uint32_t klen);
/* table_next iterates live entries: start with *it = 0. */
const Entry *table_next(const Table *t, size_t *it);

#endif
```

```c
// file: kvd/table.c
#include "table.h"

#include <stdlib.h>
#include <string.h>

static char tomb_marker;
#define TOMB (&tomb_marker)

static uint64_t hash(const char *s, uint32_t n) {
    uint64_t h = 14695981039346656037u;
    for (uint32_t i = 0; i < n; i++) {
        h ^= (unsigned char)s[i];
        h *= 1099511628211u;
    }
    return h;
}

int table_init(Table *t) {
    t->cap = 1024;
    t->len = t->used = 0;
    t->slots = calloc(t->cap, sizeof *t->slots);
    return t->slots ? 0 : -1;
}

void table_free(Table *t) {
    for (size_t i = 0; i < t->cap; i++)
        if (t->slots[i].key != NULL && t->slots[i].key != TOMB) {
            free(t->slots[i].key);
            free(t->slots[i].val);
        }
    free(t->slots);
    t->slots = NULL;
}

static size_t find(const Table *t, const char *key, uint32_t klen, uint64_t h, int *found) {
    size_t mask = t->cap - 1, i = (size_t)h & mask, tomb = SIZE_MAX;
    for (;;) {
        const Entry *e = &t->slots[i];
        if (e->key == NULL) {
            *found = 0;
            return tomb != SIZE_MAX ? tomb : i;
        }
        if (e->key == TOMB) {
            if (tomb == SIZE_MAX) tomb = i;
        } else if (e->hash == h && e->klen == klen && memcmp(e->key, key, klen) == 0) {
            *found = 1;
            return i;
        }
        i = (i + 1) & mask;
    }
}

static int rehash(Table *t, size_t cap) {
    Entry *n = calloc(cap, sizeof *n);
    if (n == NULL)
        return -1;
    for (size_t i = 0; i < t->cap; i++) {
        Entry *e = &t->slots[i];
        if (e->key == NULL || e->key == TOMB)
            continue;
        size_t j = (size_t)e->hash & (cap - 1);
        while (n[j].key != NULL)
            j = (j + 1) & (cap - 1);
        n[j] = *e;
    }
    free(t->slots);
    t->slots = n;
    t->cap = cap;
    t->used = t->len;
    return 0;
}

const Entry *table_get(const Table *t, const char *key, uint32_t klen) {
    int found;
    size_t i = find(t, key, klen, hash(key, klen), &found);
    return found ? &t->slots[i] : NULL;
}

int table_set(Table *t, const char *key, uint32_t klen, const char *val, uint32_t vlen) {
    uint64_t h = hash(key, klen);
    int found;
    size_t i = find(t, key, klen, h, &found);
    char *v = malloc((size_t)vlen + 1);
    if (v == NULL)
        return -1;
    memcpy(v, val, vlen);
    v[vlen] = '\0';
    if (found) {
        free(t->slots[i].val);
        t->slots[i].val = v;
        t->slots[i].vlen = vlen;
        return 0;
    }
    if ((t->used + 1) * 4 > t->cap * 3) {                 /* load factor 0.75 incl. tombstones */
        size_t cap = (t->len + 1) * 2 > t->cap ? t->cap * 2 : t->cap;
        if (rehash(t, cap) != 0) {
            free(v);
            return -1;
        }
        i = find(t, key, klen, h, &found);
    }
    char *k = malloc((size_t)klen + 1);
    if (k == NULL) {
        free(v);
        return -1;
    }
    memcpy(k, key, klen);
    k[klen] = '\0';
    if (t->slots[i].key == NULL)
        t->used++;
    t->slots[i] = (Entry){k, v, klen, vlen, h};
    t->len++;
    return 0;
}

int table_del(Table *t, const char *key, uint32_t klen) {
    int found;
    size_t i = find(t, key, klen, hash(key, klen), &found);
    if (!found)
        return 0;
    free(t->slots[i].key);
    free(t->slots[i].val);
    t->slots[i] = (Entry){.key = TOMB};
    t->len--;
    return 1;
}

const Entry *table_next(const Table *t, size_t *it) {
    while (*it < t->cap) {
        const Entry *e = &t->slots[(*it)++];
        if (e->key != NULL && e->key != TOMB)
            return e;
    }
    return NULL;
}
```

The table stores lengths explicitly (`klen`, `vlen`) rather than relying on `strlen`, so values could contain any bytes in a binary protocol extension, and comparisons never scan past a key.

<!-- expanded -->
### Reading the table code

The table is short, but almost every line encodes a decision. Read it with these questions in mind.

**How is a slot's state stored?** In the `key` pointer itself: `NULL` means "never used", the address of a private static byte (`TOMB`) means "deleted", and anything else is a live key. Using a unique address as a marker costs no extra field and cannot collide with a real allocation.

**Why tombstones?** With linear probing, a lookup walks from the key's home slot until it finds the key or an empty slot. If deletion simply emptied a slot, any key stored *after* it in the same probe run would become unreachable — the walk would stop early. A tombstone keeps the run unbroken: `find` skips over it while searching, but remembers the first one it saw, so an insertion can reuse that slot instead of extending the run.

**Why does the load factor count tombstones?** The growth check is `(used + 1) * 4 > cap * 3`, where `used` counts live entries *and* tombstones. A workload that keeps inserting and deleting different keys never increases `len`, but fills the table with tombstones; if they were not counted, probe runs would grow until every lookup scanned the whole array. Counting them forces a `rehash`, which drops all tombstones.

**Why might a rehash not grow the table?** `rehash` is called with double the capacity only if the *live* entries need it; otherwise it rebuilds at the same size, just to clear tombstones. Memory use then follows the number of keys, not the history of operations.

**Why store the hash in each entry?** Comparing a stored 64-bit hash first makes most non-matching comparisons a single integer test instead of a `memcmp`, and `rehash` never needs to call the hash function again.

**Why are keys and values copied in?** The table *owns* its strings (Unit 3.3): the caller's buffers belong to a connection's input buffer, which is reused for the next request. Every entry's `key` and `val` are freed by `table_del`, by overwriting in `table_set`, and by `table_free` — three places, which the sanitizer runs of Unit 13.6 check.

## Unit 13.3 — The write-ahead log

**One-line summary:** Every write is appended to a log file **before** it is acknowledged. Each record carries a **CRC-32** checksum over its contents, so recovery can tell a complete record from one torn by a crash or damaged on disk. The `fsync` policy decides how much acknowledged data a power failure can lose.

### Record format

```
record := crc32 (4 bytes LE) | op (1 byte: 'S' set, 'D' del) | klen (4 bytes LE) | vlen (4 bytes LE) | key | value
          ^ CRC covers everything after itself
```

```c
// file: kvd/wal.h
#ifndef WAL_H
#define WAL_H

#include <stddef.h>
#include <stdint.h>
#include "table.h"

enum fsync_policy { FSYNC_NO, FSYNC_EVERYSEC, FSYNC_ALWAYS };

typedef struct {
    int fd;
    enum fsync_policy policy;
    int dirty;                          /* written since the last fsync */
    uint64_t records;
} Wal;

uint32_t crc32(const void *data, size_t n);

/* wal_open opens (creating if needed) the log at path for appending. */
int wal_open(Wal *w, const char *path, enum fsync_policy policy);
/* wal_append writes one record; with FSYNC_ALWAYS it also fsyncs. Returns 0 or -1. */
int wal_append(Wal *w, char op, const char *key, uint32_t klen, const char *val, uint32_t vlen);
/* wal_sync fsyncs if anything was written since the last sync. */
int wal_sync(Wal *w);
void wal_close(Wal *w);
/* wal_replay applies every valid record in path to t. A torn or corrupt tail is
   truncated away. Returns the number of records applied, or -1 on I/O error. */
long wal_replay(const char *path, Table *t, long *truncated_bytes);

#endif
```

```c
// file: kvd/wal.c
#define _GNU_SOURCE
#include "wal.h"

#include <errno.h>
#include <fcntl.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <unistd.h>

/* CRC-32 (IEEE 802.3, as used by zip and PNG), table-driven. */
uint32_t crc32(const void *data, size_t n) {
    static uint32_t table[256];
    static int ready;
    if (!ready) {
        for (uint32_t i = 0; i < 256; i++) {
            uint32_t c = i;
            for (int k = 0; k < 8; k++)
                c = c & 1 ? 0xEDB88320u ^ (c >> 1) : c >> 1;
            table[i] = c;
        }
        ready = 1;
    }
    uint32_t c = 0xFFFFFFFFu;
    const unsigned char *p = data;
    for (size_t i = 0; i < n; i++)
        c = table[(c ^ p[i]) & 0xFF] ^ (c >> 8);
    return c ^ 0xFFFFFFFFu;
}

static void put32(unsigned char *p, uint32_t v) {
    p[0] = (unsigned char)v; p[1] = (unsigned char)(v >> 8);
    p[2] = (unsigned char)(v >> 16); p[3] = (unsigned char)(v >> 24);
}

static uint32_t get32(const unsigned char *p) {
    return (uint32_t)p[0] | (uint32_t)p[1] << 8 | (uint32_t)p[2] << 16 | (uint32_t)p[3] << 24;
}

int wal_open(Wal *w, const char *path, enum fsync_policy policy) {
    w->fd = open(path, O_WRONLY | O_CREAT | O_APPEND | O_CLOEXEC, 0644);
    w->policy = policy;
    w->dirty = 0;
    w->records = 0;
    return w->fd < 0 ? -1 : 0;
}

static int write_all(int fd, const void *buf, size_t n) {
    const char *p = buf;
    while (n > 0) {
        ssize_t r = write(fd, p, n);
        if (r < 0) {
            if (errno == EINTR) continue;
            return -1;
        }
        p += r;
        n -= (size_t)r;
    }
    return 0;
}

int wal_append(Wal *w, char op, const char *key, uint32_t klen, const char *val, uint32_t vlen) {
    size_t n = 13 + (size_t)klen + vlen;
    unsigned char stackbuf[512], *buf = n <= sizeof stackbuf ? stackbuf : malloc(n);
    if (buf == NULL)
        return -1;
    buf[4] = (unsigned char)op;
    put32(buf + 5, klen);
    put32(buf + 9, vlen);
    memcpy(buf + 13, key, klen);
    if (vlen)
        memcpy(buf + 13 + klen, val, vlen);
    put32(buf, crc32(buf + 4, n - 4));
    int rc = write_all(w->fd, buf, n);                 /* one write: one record */
    if (buf != stackbuf)
        free(buf);
    if (rc != 0)
        return -1;
    w->records++;
    w->dirty = 1;
    if (w->policy == FSYNC_ALWAYS)
        return wal_sync(w);
    return 0;
}

int wal_sync(Wal *w) {
    if (!w->dirty)
        return 0;
    if (fdatasync(w->fd) != 0)
        return -1;
    w->dirty = 0;
    return 0;
}

void wal_close(Wal *w) {
    if (w->fd >= 0) {
        wal_sync(w);
        close(w->fd);
        w->fd = -1;
    }
}

long wal_replay(const char *path, Table *t, long *truncated_bytes) {
    *truncated_bytes = 0;
    int fd = open(path, O_RDWR | O_CLOEXEC);
    if (fd < 0)
        return errno == ENOENT ? 0 : -1;
    struct stat st;
    if (fstat(fd, &st) != 0) {
        close(fd);
        return -1;
    }
    size_t size = (size_t)st.st_size;
    unsigned char *data = malloc(size ? size : 1);
    if (data == NULL || pread(fd, data, size, 0) != (ssize_t)size) {
        free(data);
        close(fd);
        return -1;
    }
    size_t off = 0;
    long applied = 0;
    while (off + 13 <= size) {
        uint32_t klen = get32(data + off + 5), vlen = get32(data + off + 9);
        if (klen > 255 || vlen > 65536 || 13 + (size_t)klen + vlen > size - off)
            break;                                     /* torn or corrupt: lengths impossible */
        size_t n = 13 + (size_t)klen + vlen;
        if (crc32(data + off + 4, n - 4) != get32(data + off))
            break;                                     /* torn or corrupt: checksum mismatch */
        const char *key = (const char *)data + off + 13, *val = key + klen;
        char op = (char)data[off + 4];
        if (op == 'S')
            table_set(t, key, klen, val, vlen);
        else if (op == 'D')
            table_del(t, key, klen);
        else
            break;
        off += n;
        applied++;
    }
    if (off < size) {                                  /* drop everything after the last good record */
        *truncated_bytes = (long)(size - off);
        if (ftruncate(fd, (off_t)off) != 0 || fsync(fd) != 0)
            applied = -1;
    }
    free(data);
    close(fd);
    return applied;
}
```

Design notes:

- **One `write` per record**, built in a buffer: a record is never interleaved with another, and with `O_APPEND` each write goes to the current end of the file.
- **`fdatasync`** flushes the data (and the size) without forcing unrelated metadata like access times — cheaper than `fsync` for logs.
- **Recovery is conservative:** it stops at the first record whose lengths are impossible or whose checksum fails, and truncates the file there. Everything before is valid; everything after was never acknowledged (if the crash tore the last record) or is corrupt (and replaying it would be worse than losing it).
- **Bounds before trust:** lengths read from the file are validated against limits and the remaining file size before they are used — the parser rules of Chapter 12 apply to your own files too, because files get corrupted.

### Deep & Tricky

**DT-91. `write` returning success does not mean the data is on disk.** *(OS behaviour)* After `write`, data sits in the kernel's page cache; it survives a process crash (even `kill -9`) but not a power failure or kernel panic. Only `fsync`/`fdatasync` returning success means the storage device has it — and even that depends on the device honouring cache flushes. The fsync policy is the explicit trade-off between latency and the amount of data a power failure can lose.

## Unit 13.4 — Snapshots with `fork`

**One-line summary:** Replaying an ever-growing log makes recovery slow, so the server periodically writes a **snapshot** of the whole table and starts a new log. Doing this in a `fork`ed child gives the child a frozen copy-on-write image of the table while the parent continues serving; the parent switches to a new log file at the moment of the fork, so the snapshot plus the newer logs always describe the full state.

### The file protocol

```
data/snapshot          header "KVSNAP1" + generation G, then records, then CRC of everything
data/wal.<G>           log of writes after snapshot generation G
```

On `BGSAVE` the parent increments the generation to G+1, starts appending to `wal.<G+1>`, and forks. The child writes `snapshot.tmp` with generation G+1 (meaning "everything before `wal.<G+1>`"), `fsync`s it, and renames it over `snapshot`. When the parent reaps a successful child, it deletes logs older than G+1. **Recovery** loads the snapshot (generation S, or 0 if none), then replays `wal.<S>`, `wal.<S+1>`, … in order. A crash at any point leaves a consistent set of files: either the old snapshot and all the logs since it, or the new snapshot and the new log.

```c
// file: kvd/snapshot.h
#ifndef SNAPSHOT_H
#define SNAPSHOT_H

#include <stdint.h>
#include "table.h"

/* snapshot_write writes t to dir/snapshot atomically with generation gen. Returns 0 or -1. */
int snapshot_write(const char *dir, const Table *t, uint64_t gen);
/* snapshot_load loads dir/snapshot into t and sets *gen (0 if no snapshot exists).
   Returns the number of keys loaded, or -1 if the snapshot is corrupt. */
long snapshot_load(const char *dir, Table *t, uint64_t *gen);

#endif
```

```c
// file: kvd/snapshot.c
#define _GNU_SOURCE
#include "snapshot.h"
#include "wal.h"

#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int snapshot_write(const char *dir, const Table *t, uint64_t gen) {
    char tmp[512], final[512];
    snprintf(tmp, sizeof tmp, "%s/snapshot.tmp", dir);
    snprintf(final, sizeof final, "%s/snapshot", dir);
    FILE *f = fopen(tmp, "wb");
    if (f == NULL)
        return -1;
    uint32_t crc = 0;
    unsigned char hdr[16] = "KVSNAP1";
    for (int i = 0; i < 8; i++)
        hdr[8 + i] = (unsigned char)(gen >> (8 * i));
    fwrite(hdr, 1, 16, f);
    crc = crc32(hdr, 16);
    size_t it = 0;
    const Entry *e;
    while ((e = table_next(t, &it)) != NULL) {
        unsigned char lens[8];
        for (int i = 0; i < 4; i++) {
            lens[i] = (unsigned char)(e->klen >> (8 * i));
            lens[4 + i] = (unsigned char)(e->vlen >> (8 * i));
        }
        fwrite(lens, 1, 8, f);
        fwrite(e->key, 1, e->klen, f);
        fwrite(e->val, 1, e->vlen, f);
        /* chain the CRC over every record: crc = crc32(prev_crc || record) */
        uint32_t parts[1] = {crc};
        unsigned char rec_crc_in[4 + 8];
        memcpy(rec_crc_in, parts, 4);
        memcpy(rec_crc_in + 4, lens, 8);
        crc = crc32(rec_crc_in, sizeof rec_crc_in) ^ crc32(e->key, e->klen) ^ crc32(e->val, e->vlen);
    }
    unsigned char end[12] = {0xFF, 0xFF, 0xFF, 0xFF};      /* klen = 0xFFFFFFFF marks the end */
    for (int i = 0; i < 4; i++)
        end[8 + i] = (unsigned char)(crc >> (8 * i));
    fwrite(end, 1, 12, f);
    if (fflush(f) != 0 || fsync(fileno(f)) != 0 || fclose(f) != 0) {
        unlink(tmp);
        return -1;
    }
    if (rename(tmp, final) != 0)                           /* atomic replace (Unit 6.3) */
        return -1;
    int dfd = open(dir, O_RDONLY | O_DIRECTORY);           /* make the rename itself durable */
    if (dfd >= 0) {
        fsync(dfd);
        close(dfd);
    }
    return 0;
}

static int read_exact(FILE *f, void *buf, size_t n) {
    return fread(buf, 1, n, f) == n ? 0 : -1;
}

long snapshot_load(const char *dir, Table *t, uint64_t *gen) {
    char path[512];
    snprintf(path, sizeof path, "%s/snapshot", dir);
    *gen = 0;
    FILE *f = fopen(path, "rb");
    if (f == NULL)
        return 0;                                          /* no snapshot yet */
    unsigned char hdr[16];
    long n = 0;
    char *key = malloc(256), *val = malloc(65536);
    if (key == NULL || val == NULL || read_exact(f, hdr, 16) != 0 || memcmp(hdr, "KVSNAP1", 8) != 0)
        goto corrupt;
    for (int i = 0; i < 8; i++)
        *gen |= (uint64_t)hdr[8 + i] << (8 * i);
    uint32_t crc = crc32(hdr, 16);
    for (;;) {
        unsigned char lens[8];
        if (read_exact(f, lens, 8) != 0)
            goto corrupt;
        uint32_t klen = (uint32_t)lens[0] | (uint32_t)lens[1] << 8 | (uint32_t)lens[2] << 16 | (uint32_t)lens[3] << 24;
        uint32_t vlen = (uint32_t)lens[4] | (uint32_t)lens[5] << 8 | (uint32_t)lens[6] << 16 | (uint32_t)lens[7] << 24;
        if (klen == 0xFFFFFFFFu) {
            unsigned char c[4];
            if (read_exact(f, c, 4) != 0)
                goto corrupt;
            uint32_t stored = (uint32_t)c[0] | (uint32_t)c[1] << 8 | (uint32_t)c[2] << 16 | (uint32_t)c[3] << 24;
            if (stored != crc)
                goto corrupt;
            break;
        }
        if (klen > 255 || vlen > 65536 || read_exact(f, key, klen) != 0 || read_exact(f, val, vlen) != 0)
            goto corrupt;
        unsigned char rec_crc_in[12];
        memcpy(rec_crc_in, &crc, 4);
        memcpy(rec_crc_in + 4, lens, 8);
        crc = crc32(rec_crc_in, sizeof rec_crc_in) ^ crc32(key, klen) ^ crc32(val, vlen);
        if (table_set(t, key, klen, val, vlen) != 0)
            goto corrupt;
        n++;
    }
    free(key);
    free(val);
    fclose(f);
    return n;
corrupt:
    free(key);
    free(val);
    fclose(f);
    return -1;
}
```

The snapshot ends with a checksum chained over every record, so a truncated or damaged snapshot is rejected rather than half-loaded. (The chaining is written straightforwardly rather than efficiently: one CRC per field, combined. A production format would use a streaming CRC; either works for detecting damage.)

## Unit 13.5 — The server

**One-line summary:** The server is Chapter 10's event loop with a line protocol: each connection accumulates input, complete lines are executed as commands against the table and log, replies are queued in an output buffer, and partial writes resume on `EPOLLOUT`. A one-second timer drives `everysec` fsyncs and reaps finished snapshot children.

```c
// file: kvd/kvd.c
/* kvd: a persistent key-value server. */
#define _GNU_SOURCE
#include <ctype.h>
#include <dirent.h>
#include <errno.h>
#include <fcntl.h>
#include <limits.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <signal.h>
#include <stdarg.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <sys/wait.h>
#include <time.h>
#include <unistd.h>

#include "snapshot.h"
#include "table.h"
#include "wal.h"

enum { IN_MAX = 70000, OUT_MAX = 1 << 20, IDLE_SECONDS = 300 };

typedef struct Conn {
    int fd;
    char in[IN_MAX];
    size_t in_len;
    char *out;
    size_t out_len, out_off;
    bool closing;
    time_t last_active;
    struct Conn *next;
} Conn;

static struct {
    Table table;
    Wal wal;
    const char *dir;
    uint64_t gen;                       /* current log generation */
    pid_t snap_child;
    uint64_t snap_gen;
    long commands, connections;
    time_t started;
    int ep;
    Conn *conns;
} S;

static volatile sig_atomic_t stop;

static void logmsg(const char *fmt, ...) {
    va_list ap;
    va_start(ap, fmt);
    fprintf(stderr, "kvd: ");
    vfprintf(stderr, fmt, ap);
    fprintf(stderr, "\n");
    va_end(ap);
}

/* ---------- persistence ---------- */

static void wal_path(char *buf, size_t n, uint64_t gen) {
    snprintf(buf, n, "%s/wal.%llu", S.dir, (unsigned long long)gen);
}

static int recover(void) {
    long keys = snapshot_load(S.dir, &S.table, &S.gen);
    if (keys < 0) {
        logmsg("snapshot is corrupt; refusing to start (move it away to start from the logs)");
        return -1;
    }
    if (S.gen == 0)
        S.gen = 1;
    logmsg("snapshot: %ld keys, generation %llu", keys, (unsigned long long)S.gen);
    for (uint64_t g = S.gen;; g++) {        /* replay wal.<gen>, wal.<gen+1>, ... */
        char path[512];
        wal_path(path, sizeof path, g);
        if (access(path, F_OK) != 0)
            break;
        long truncated;
        long n = wal_replay(path, &S.table, &truncated);
        if (n < 0)
            return -1;
        logmsg("replayed %s: %ld records%s", path, n, truncated ? " (torn tail truncated)" : "");
        if (truncated)
            logmsg("  discarded %ld bytes of incomplete or corrupt log", truncated);
        S.gen = g;
    }
    logmsg("recovered %zu keys", S.table.len);
    return 0;
}

static int start_bgsave(void) {
    if (S.snap_child > 0)
        return -1;                          /* one at a time */
    char path[512];
    Wal next;
    wal_path(path, sizeof path, S.gen + 1);
    if (wal_open(&next, path, S.wal.policy) != 0)
        return -1;
    wal_close(&S.wal);                      /* everything before this point is in the old log */
    S.wal = next;
    S.gen++;
    fflush(NULL);
    pid_t pid = fork();
    if (pid < 0)
        return -1;
    if (pid == 0) {                         /* child: copy-on-write view of the table */
        int rc = snapshot_write(S.dir, &S.table, S.gen);
        _exit(rc == 0 ? 0 : 1);
    }
    S.snap_child = pid;
    S.snap_gen = S.gen;
    return 0;
}

static void reap_bgsave(void) {
    int status;
    if (S.snap_child <= 0 || waitpid(S.snap_child, &status, WNOHANG) != S.snap_child)
        return;
    S.snap_child = 0;
    if (!WIFEXITED(status) || WEXITSTATUS(status) != 0) {
        logmsg("background save failed; logs kept");
        return;
    }
    for (uint64_t g = S.snap_gen; g-- > 1;) {   /* older logs are now covered by the snapshot */
        char path[512];
        wal_path(path, sizeof path, g);
        if (unlink(path) != 0)
            break;
    }
    logmsg("background save done: generation %llu", (unsigned long long)S.snap_gen);
}

/* ---------- protocol ---------- */

static void reply(Conn *c, const char *fmt, ...) {
    if (c->out == NULL && (c->out = malloc(OUT_MAX)) == NULL) {
        c->closing = true;
        return;
    }
    va_list ap;
    va_start(ap, fmt);
    int n = vsnprintf(c->out + c->out_len, OUT_MAX - c->out_len, fmt, ap);
    va_end(ap);
    if (n < 0 || (size_t)n >= OUT_MAX - c->out_len) {
        c->closing = true;                  /* client isn't reading its replies: drop it */
        return;
    }
    c->out_len += (size_t)n;
}

/* next_token splits off one space-separated word; *rest gets the remainder. */
static char *next_token(char **s) {
    while (**s == ' ')
        (*s)++;
    if (**s == '\0')
        return NULL;
    char *t = *s;
    while (**s != ' ' && **s != '\0')
        (*s)++;
    if (**s == ' ')
        *(*s)++ = '\0';
    return t;
}

static void execute(Conn *c, char *line) {
    S.commands++;
    char *rest = line, *cmd = next_token(&rest);
    if (cmd == NULL)
        return;
    for (char *p = cmd; *p; p++)
        *p = (char)toupper((unsigned char)*p);
    char *key = NULL;
    size_t klen = 0;
    if (strcmp(cmd, "GET") == 0 || strcmp(cmd, "SET") == 0 || strcmp(cmd, "DEL") == 0 ||
        strcmp(cmd, "INCR") == 0) {
        key = next_token(&rest);
        if (key == NULL || (klen = strlen(key)) > 255) {
            reply(c, "-ERR missing or too long key\n");
            return;
        }
    }
    if (strcmp(cmd, "PING") == 0) {
        reply(c, "+PONG\n");
    } else if (strcmp(cmd, "GET") == 0) {
        const Entry *e = table_get(&S.table, key, (uint32_t)klen);
        if (e) reply(c, "$%s\n", e->val);
        else reply(c, "(nil)\n");
    } else if (strcmp(cmd, "SET") == 0) {
        while (*rest == ' ') rest++;
        size_t vlen = strlen(rest);
        if (vlen > 65536) {
            reply(c, "-ERR value too long\n");
        } else if (wal_append(&S.wal, 'S', key, (uint32_t)klen, rest, (uint32_t)vlen) != 0) {
            reply(c, "-ERR log write failed: %s\n", strerror(errno));   /* not acknowledged */
        } else if (table_set(&S.table, key, (uint32_t)klen, rest, (uint32_t)vlen) != 0) {
            reply(c, "-ERR out of memory\n");
        } else {
            reply(c, "+OK\n");
        }
    } else if (strcmp(cmd, "DEL") == 0) {
        int existed = table_get(&S.table, key, (uint32_t)klen) != NULL;
        if (existed && wal_append(&S.wal, 'D', key, (uint32_t)klen, NULL, 0) != 0)
            reply(c, "-ERR log write failed\n");
        else {
            if (existed) table_del(&S.table, key, (uint32_t)klen);
            reply(c, ":%d\n", existed);
        }
    } else if (strcmp(cmd, "INCR") == 0) {
        const Entry *e = table_get(&S.table, key, (uint32_t)klen);
        long long v = 0;
        if (e) {
            char *end;
            errno = 0;
            v = strtoll(e->val, &end, 10);
            if (*end != '\0' || errno == ERANGE || e->vlen == 0) {
                reply(c, "-ERR value is not an integer\n");
                return;
            }
        }
        if (v == LLONG_MAX) {
            reply(c, "-ERR increment would overflow\n");
            return;
        }
        char buf[32];
        int n = snprintf(buf, sizeof buf, "%lld", v + 1);
        if (wal_append(&S.wal, 'S', key, (uint32_t)klen, buf, (uint32_t)n) != 0 ||
            table_set(&S.table, key, (uint32_t)klen, buf, (uint32_t)n) != 0)
            reply(c, "-ERR write failed\n");
        else
            reply(c, ":%s\n", buf);
    } else if (strcmp(cmd, "DBSIZE") == 0) {
        reply(c, ":%zu\n", S.table.len);
    } else if (strcmp(cmd, "BGSAVE") == 0) {
        if (start_bgsave() == 0) reply(c, "+Background save started\n");
        else reply(c, "-ERR background save already running or failed to start\n");
    } else if (strcmp(cmd, "INFO") == 0) {
        reply(c, "$keys=%zu commands=%ld connections=%ld wal_records=%llu generation=%llu uptime=%lds\n",
              S.table.len, S.commands, S.connections, (unsigned long long)S.wal.records,
              (unsigned long long)S.gen, (long)(time(NULL) - S.started));
    } else if (strcmp(cmd, "QUIT") == 0) {
        reply(c, "+BYE\n");
        c->closing = true;
    } else {
        reply(c, "-ERR unknown command '%.32s'\n", cmd);
    }
}

/* ---------- event loop (Chapter 10) ---------- */

static void conn_close(Conn *c) {
    epoll_ctl(S.ep, EPOLL_CTL_DEL, c->fd, NULL);
    close(c->fd);
    for (Conn **p = &S.conns; *p; p = &(*p)->next)
        if (*p == c) { *p = c->next; break; }
    free(c->out);
    free(c);
}

static bool flush_out(Conn *c) {           /* returns false if the connection was closed */
    while (c->out_off < c->out_len) {
        ssize_t w = send(c->fd, c->out + c->out_off, c->out_len - c->out_off, MSG_NOSIGNAL);
        if (w < 0) {
            if (errno == EAGAIN) {
                struct epoll_event ev = {.events = EPOLLOUT, .data.ptr = c};
                epoll_ctl(S.ep, EPOLL_CTL_MOD, c->fd, &ev);
                return true;
            }
            conn_close(c);
            return false;
        }
        c->out_off += (size_t)w;
    }
    c->out_len = c->out_off = 0;
    if (c->closing) {
        conn_close(c);
        return false;
    }
    struct epoll_event ev = {.events = EPOLLIN, .data.ptr = c};
    epoll_ctl(S.ep, EPOLL_CTL_MOD, c->fd, &ev);
    return true;
}

static void on_readable(Conn *c) {
    for (;;) {
        ssize_t n = recv(c->fd, c->in + c->in_len, sizeof c->in - c->in_len, 0);
        if (n == 0) { conn_close(c); return; }
        if (n < 0) {
            if (errno == EAGAIN) break;
            conn_close(c);
            return;
        }
        c->in_len += (size_t)n;
        if (c->in_len == sizeof c->in) break;
    }
    c->last_active = time(NULL);
    size_t start = 0;
    for (;;) {                              /* execute every complete line (pipelining) */
        char *nl = memchr(c->in + start, '\n', c->in_len - start);
        if (nl == NULL)
            break;
        *nl = '\0';
        if (nl > c->in + start && nl[-1] == '\r')
            nl[-1] = '\0';                  /* accept \r\n too */
        execute(c, c->in + start);
        start = (size_t)(nl - c->in) + 1;
        if (c->closing)
            break;
    }
    memmove(c->in, c->in + start, c->in_len - start);
    c->in_len -= start;
    if (c->in_len == sizeof c->in) {        /* a line longer than the buffer */
        reply(c, "-ERR line too long\n");
        c->closing = true;
    }
    flush_out(c);                           /* with --fsync always, each record was synced in SET */
}

static void on_accept(int lfd) {
    for (;;) {
        int fd = accept4(lfd, NULL, NULL, SOCK_NONBLOCK | SOCK_CLOEXEC);
        if (fd < 0)
            return;
        Conn *c = calloc(1, sizeof *c);
        if (c == NULL) { close(fd); continue; }
        int one = 1;
        setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof one);
        c->fd = fd;
        c->last_active = time(NULL);
        c->next = S.conns;
        S.conns = c;
        S.connections++;
        struct epoll_event ev = {.events = EPOLLIN, .data.ptr = c};
        epoll_ctl(S.ep, EPOLL_CTL_ADD, fd, &ev);
    }
}

static void on_term(int sig) { (void)sig; stop = 1; }

int main(int argc, char *argv[]) {
    const char *port = "7379";
    enum fsync_policy policy = FSYNC_EVERYSEC;
    S.dir = "data";
    for (int i = 1; i + 1 < argc; i += 2) {
        if (strcmp(argv[i], "--port") == 0) port = argv[i + 1];
        else if (strcmp(argv[i], "--dir") == 0) S.dir = argv[i + 1];
        else if (strcmp(argv[i], "--fsync") == 0) {
            if (strcmp(argv[i + 1], "always") == 0) policy = FSYNC_ALWAYS;
            else if (strcmp(argv[i + 1], "no") == 0) policy = FSYNC_NO;
            else policy = FSYNC_EVERYSEC;
        }
    }
    signal(SIGPIPE, SIG_IGN);
    struct sigaction sa = {.sa_handler = on_term};
    sigaction(SIGTERM, &sa, NULL);
    sigaction(SIGINT, &sa, NULL);
    mkdir(S.dir, 0755);
    S.started = time(NULL);

    if (table_init(&S.table) != 0 || recover() != 0)
        return 1;
    char path[512];
    wal_path(path, sizeof path, S.gen);
    if (wal_open(&S.wal, path, policy) != 0) {
        logmsg("cannot open %s: %s", path, strerror(errno));
        return 1;
    }

    int lfd = socket(AF_INET, SOCK_STREAM | SOCK_NONBLOCK | SOCK_CLOEXEC, 0), one = 1;
    setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &one, sizeof one);
    struct sockaddr_in addr = {.sin_family = AF_INET, .sin_port = htons((uint16_t)atoi(port)),
                               .sin_addr.s_addr = htonl(INADDR_LOOPBACK)};
    if (bind(lfd, (struct sockaddr *)&addr, sizeof addr) != 0 || listen(lfd, SOMAXCONN) != 0) {
        logmsg("cannot listen on %s: %s", port, strerror(errno));
        return 1;
    }
    S.ep = epoll_create1(EPOLL_CLOEXEC);
    struct epoll_event lev = {.events = EPOLLIN, .data.ptr = NULL};
    epoll_ctl(S.ep, EPOLL_CTL_ADD, lfd, &lev);
    logmsg("listening on 127.0.0.1:%s, fsync=%s, dir=%s", port,
           policy == FSYNC_ALWAYS ? "always" : policy == FSYNC_NO ? "no" : "everysec", S.dir);

    time_t last_tick = time(NULL);
    while (!stop) {
        struct epoll_event ev[128];
        int n = epoll_wait(S.ep, ev, 128, 200);
        for (int i = 0; i < n; i++) {
            Conn *c = ev[i].data.ptr;
            if (c == NULL) on_accept(lfd);
            else if (ev[i].events & (EPOLLERR | EPOLLHUP)) conn_close(c);
            else if (ev[i].events & EPOLLOUT) flush_out(c);
            else if (ev[i].events & EPOLLIN) on_readable(c);
        }
        time_t now = time(NULL);
        if (now != last_tick) {             /* once per second */
            last_tick = now;
            if (S.wal.policy == FSYNC_EVERYSEC && wal_sync(&S.wal) != 0)
                logmsg("fsync failed: %s", strerror(errno));
            reap_bgsave();
            for (Conn *c = S.conns, *next; c; c = next) {
                next = c->next;
                if (now - c->last_active > IDLE_SECONDS)
                    conn_close(c);
            }
        }
    }
    logmsg("shutting down: %ld commands served", S.commands);
    while (S.conns)
        conn_close(S.conns);
    if (S.snap_child > 0)
        waitpid(S.snap_child, NULL, 0);
    wal_close(&S.wal);
    table_free(&S.table);
    close(lfd);
    return 0;
}
```

```make
# file: kvd/Makefile
CC      := gcc
CFLAGS  := -std=gnu17 -Wall -Wextra -O2 -g -MMD -MP
SRCS    := kvd.c table.c wal.c snapshot.c
OBJS    := $(SRCS:.c=.o)

.PHONY: all clean asan
all: kvd kvload

kvd: $(OBJS)
	$(CC) $(CFLAGS) $^ -o $@

kvload: kvload.c
	$(CC) $(CFLAGS) -pthread $< -o $@

asan: $(SRCS)
	$(CC) -std=gnu17 -Wall -Wextra -O1 -g -fsanitize=address,undefined $^ -o kvd_asan

clean:
	rm -f kvd kvd_asan kvload *.o *.d

-include $(wildcard *.d)
```

The order of operations in `SET` is the core of the durability guarantee: **log first, then apply, then reply**. If the log write fails, the client gets an error and the table is unchanged. If the process dies after the log write but before the reply, the client never got `+OK`, and on restart the write is present — which is allowed (the client must treat an unanswered request as "unknown", as with any network service).

<!-- expanded -->
### Reading the server code

The server is one thread and one `epoll` loop. Follow one request through it:

1. **Accept.** The listening socket is in the epoll set with `data.ptr = NULL`, which is how the loop tells it apart from connections. `on_accept` accepts in a loop until `EAGAIN` (the socket is non-blocking), allocates a `Conn` for each new client, and registers it for `EPOLLIN` with `data.ptr` pointing at the `Conn` — so an event leads straight to its connection without a lookup.
2. **Read.** `on_readable` appends whatever arrived to the connection's input buffer (`IN_MAX` bytes; a client that sends a longer line without a newline gets `-ERR line too long` and is disconnected, rather than being allowed to grow the server's memory without limit). Then it executes every *complete* line in the buffer and moves the incomplete rest to the front — TCP gives a byte stream (Unit 10.3), so one read may contain half a command or ten of them.
3. **Execute.** `execute` splits the line into tokens and runs the command against the table. Every command that changes data is first appended to the write-ahead log (Unit 13.3) — the *log first, then the table* order is what makes recovery possible.
4. **Reply.** `reply` formats into a per-connection output buffer (allocated on first use, `OUT_MAX` bytes). `flush_out` writes as much as the socket accepts; if the kernel's send buffer is full, it switches the connection's interest to `EPOLLOUT` and continues when the socket becomes writable, instead of blocking the whole server on one slow client. Once the buffer is empty, interest goes back to `EPOLLIN`.
5. **Snapshot.** On the `BGSAVE` command, `start_bgsave` `fork`s: the child writes the table to a snapshot file while the parent goes on serving — copy-on-write (Unit 8.3 and the Linux book's Chapter 11) gives the child a frozen image of the table at no up-front cost. `reap_bgsave` collects the child with `waitpid(..., WNOHANG)` from the event loop and deletes the log files the snapshot made unnecessary.
6. **Stop.** `SIGTERM` only sets a `volatile sig_atomic_t` flag (the only thing a signal handler may safely do here, Unit 8.5); the loop sees it after `epoll_wait` returns, flushes the log, and exits cleanly.

The design trades simplicity for limits: one CPU core does all the work, and one expensive request (a 64 KiB value written with `--fsync always`, which waits for the disk) delays every client behind it. Redis made the same trade for years and showed how far it goes; Unit 13.8 discusses what changes when it is no longer enough.

## Topic 13C — Prove and operate

## Unit 13.6 — Testing correctness and crash recovery

**One-line summary:** Test the protocol with scripted sessions, the durability claims with real `kill -9` crashes, recovery with deliberately torn and corrupted logs, and the whole server under AddressSanitizer. Durability bugs hide until a crash: so crash it on purpose.

```console
$ cd kvd && make clean > /dev/null && make 2>&1 | grep -E 'error|warning' ; make asan 2>&1 | grep -E 'error|warning'; ls kvd kvd_asan
kvd  kvd_asan
```

### Protocol

```console
$ cd kvd && rm -rf t1 && (./kvd_asan --port 7401 --dir t1 2> t1.log & echo $! > t1.pid) && sleep 0.5 && exec 3<>/dev/tcp/127.0.0.1/7401 && printf 'PING\nSET greeting hello world\nGET greeting\nGET missing\nINCR counter\nINCR counter\nSET n abc\nINCR n\nDEL greeting\nDEL greeting\nDBSIZE\nBOGUS\nset lower case works\nget lower\nQUIT\n' >&3 && cat <&3
+PONG
+OK
$hello world
(nil)
:1
:2
+OK
-ERR value is not an integer
:1
:0
:2
-ERR unknown command 'BOGUS'
+OK
$case works
+BYE
```

Every reply matches the specification, including errors: `INCR` on a non-integer, an unknown command, and commands accepted case-insensitively. All fifteen commands arrived in one write (pipelining) and were answered in order.

### Crash recovery

The test writes 2,000 keys with `--fsync always`, kills the server with `SIGKILL` (no chance to clean up), restarts it, and checks every acknowledged key:

```sh
# file: kvd/crashtest.sh
#!/bin/bash
# crashtest.sh POLICY: write keys, kill -9 the server, restart, verify every acknowledged key.
set -u
POLICY=$1; PORT=7402; DIR=crash-$POLICY
rm -rf $DIR
./kvd --port $PORT --dir $DIR --fsync $POLICY 2> $DIR.log & PID=$!
sleep 0.3
exec 3<>/dev/tcp/127.0.0.1/$PORT
acked=0
for i in $(seq 1 2000); do
    printf 'SET key%d value-%d\n' $i $i >&3
    read -r line <&3
    [ "$line" = "+OK" ] && acked=$i
done
kill -9 $PID; wait $PID 2>/dev/null
exec 3>&-
./kvd --port $PORT --dir $DIR --fsync $POLICY 2>> $DIR.log & PID=$!
sleep 0.5
exec 3<>/dev/tcp/127.0.0.1/$PORT
missing=0
for i in $(seq 1 $acked); do
    printf 'GET key%d\n' $i >&3
    read -r line <&3
    [ "$line" = "\$value-$i" ] || missing=$((missing + 1))
done
echo "fsync=$POLICY: $acked writes acknowledged, $missing missing after kill -9 and restart"
grep -E 'replayed|recovered' $DIR.log | tail -n 2
kill $PID; wait $PID 2>/dev/null
```

```console
$ cd kvd && chmod +x crashtest.sh && ./crashtest.sh always && ./crashtest.sh no
fsync=always: 2000 writes acknowledged, 0 missing after kill -9 and restart
kvd: replayed crash-always/wal.1: 2000 records
kvd: recovered 2000 keys
fsync=no: 2000 writes acknowledged, 0 missing after kill -9 and restart
kvd: replayed crash-no/wal.1: 2000 records
kvd: recovered 2000 keys
```

Both policies lost nothing — and for `kill -9`, both are expected to: a process crash cannot lose data that already reached the kernel with `write`. The difference between policies appears only when the *machine* fails (power loss, kernel panic), where `--fsync no` can lose whatever the kernel had not yet written back (typically up to 30 seconds), `everysec` about a second, and `always` nothing that was acknowledged (DT-91). That claim cannot be tested with `kill -9`; testing it properly requires power-cut testing on real hardware or a fault-injecting storage layer such as `dm-flakey` or a virtual machine snapshot — an honest limit of this test.

### Torn and corrupted logs

```console
$ cd kvd && rm -rf t3 && (./kvd --port 7403 --dir t3 2> t3.log & echo $! > t3.pid) && sleep 0.3 && exec 3<>/dev/tcp/127.0.0.1/7403 && printf 'SET a 1\nSET b 2\nSET c 3\nQUIT\n' >&3 && cat <&3 > /dev/null && kill $(cat t3.pid) && sleep 0.5 && ls -l t3/wal.1 | awk '{print "log size before:", $5}'
log size before: 45
$ cd kvd && printf '\x12\x34\x56\x78S\x01\x00\x00\x00\x05\x00\x00\x00dhel' >> t3/wal.1 && (./kvd --port 7403 --dir t3 2> t3b.log & echo $! > t3.pid) && sleep 0.5 && exec 3<>/dev/tcp/127.0.0.1/7403 && printf 'GET a\nGET c\nGET d\nDBSIZE\nQUIT\n' >&3 && cat <&3 && kill $(cat t3.pid) && sleep 0.3 && cat t3b.log | grep -E 'replayed|discarded|recovered' && ls -l t3/wal.1 | awk '{print "log size after recovery:", $5}'
$1
$3
(nil)
:3
+BYE
kvd: replayed t3/wal.1: 3 records (torn tail truncated)
kvd:   discarded 17 bytes of incomplete or corrupt log
kvd: recovered 3 keys
log size after recovery: 45
$ cd kvd && printf 'X' | dd of=t3/wal.1 bs=1 seek=20 conv=notrunc 2>/dev/null && (./kvd --port 7403 --dir t3 2> t3c.log & echo $! > t3.pid) && sleep 0.5 && kill $(cat t3.pid) && sleep 0.3 && grep -E 'replayed|discarded|recovered' t3c.log
kvd: replayed t3/wal.1: 1 records (torn tail truncated)
kvd:   discarded 30 bytes of incomplete or corrupt log
kvd: recovered 1 keys
```

The first experiment appended half of a record — exactly what a crash during `write` leaves behind. Recovery replayed the three complete records, detected the torn one (its declared lengths run past the end of the file), discarded its 17 bytes, and truncated the log back to the last good record. The second flipped one byte inside the second record: its CRC no longer matched, so recovery kept only the first record and discarded the rest. That is the conservative choice — a log is a sequence, and records after a corrupt one cannot be trusted to be in order — and it is why databases also keep snapshots and replicas.

### Snapshots and log rotation

```console
$ cd kvd && rm -rf t4 && (./kvd --port 7404 --dir t4 2> t4.log & echo $! > t4.pid) && sleep 0.3 && exec 3<>/dev/tcp/127.0.0.1/7404 && (for i in $(seq 1 5000); do echo "SET k$i v$i"; done; echo BGSAVE; echo "SET after-snapshot yes"; echo QUIT) >&3 && cat <&3 | sort | uniq -c && sleep 1.5 && ls t4 && kill $(cat t4.pid) && sleep 0.3
      1 +BYE
      1 +Background save started
   5001 +OK
snapshot  wal.2
$ cd kvd && (./kvd --port 7404 --dir t4 2> t4b.log & echo $! > t4.pid) && sleep 0.5 && exec 3<>/dev/tcp/127.0.0.1/7404 && printf 'GET k4999\nGET after-snapshot\nDBSIZE\nQUIT\n' >&3 && cat <&3 && kill $(cat t4.pid) && sleep 0.3 && grep -E 'snapshot:|replayed|recovered' t4b.log
$v4999
$yes
:5001
+BYE
kvd: snapshot: 5000 keys, generation 2
kvd: replayed t4/wal.2: 1 records
kvd: recovered 5001 keys
```

After `BGSAVE`, the directory holds the snapshot (generation 2) and the new log `wal.2`; `wal.1` was deleted once the child reported success. The write made *after* `BGSAVE` went to `wal.2`, and on restart the server loaded 5,000 keys from the snapshot and replayed the one later record from `wal.2`.

### Sanitizers

```console
$ cd kvd && kill -TERM $(cat t1.pid) && sleep 1 && grep -cE 'ERROR|runtime error|leak' t1.log; tail -n 1 t1.log
0
kvd: shutting down: 15 commands served
```

(The two `-Wempty-body` warnings in the first build came from a leftover `if` with an empty body in `on_readable` — harmless, but a warning-free build is the standard, so it was removed; the listing above is the corrected version.) The AddressSanitizer build that served the protocol test shut down cleanly on `SIGTERM` with no memory errors, undefined behaviour, or leaks.

## Unit 13.7 — Load testing and performance

**One-line summary:** Measure throughput and latency under concurrent load for each durability policy, explain the results in terms of what the system waits for, and find the bottleneck before optimising.

```c
// file: kvd/kvload.c
/* kvload: C connections, each doing SET then GET in a loop for D seconds (one
   request in flight per connection), reporting throughput and latency. */
#define _GNU_SOURCE
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <time.h>
#include <unistd.h>

static int port;
static double duration;

static double now(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec / 1e9;
}

typedef struct { int id; long ops, errors; double lat_sum, lat_max; } Stats;

static int roundtrip(int fd, const char *req, size_t n, char *buf, size_t bufsz) {
    if (write(fd, req, n) != (ssize_t)n)
        return -1;
    size_t got = 0;
    while (got == 0 || buf[got - 1] != '\n') {
        ssize_t r = read(fd, buf + got, bufsz - got - 1);
        if (r <= 0) return -1;
        got += (size_t)r;
    }
    buf[got] = '\0';
    return 0;
}

static void *worker(void *arg) {
    Stats *s = arg;
    int fd = socket(AF_INET, SOCK_STREAM, 0), one = 1;
    setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof one);
    struct sockaddr_in a = {.sin_family = AF_INET, .sin_port = htons((uint16_t)port),
                            .sin_addr.s_addr = htonl(INADDR_LOOPBACK)};
    if (connect(fd, (struct sockaddr *)&a, sizeof a) != 0) { s->errors++; return NULL; }
    char req[128], buf[256];
    double end = now() + duration;
    for (long i = 0; now() < end; i++) {
        int n = (i & 1) ? snprintf(req, sizeof req, "GET w%d-k%ld\n", s->id, (i - 1) % 1000)
                        : snprintf(req, sizeof req, "SET w%d-k%ld value-%ld\n", s->id, i % 1000, i);
        double t0 = now();
        if (roundtrip(fd, req, (size_t)n, buf, sizeof buf) != 0 || buf[0] == '-') { s->errors++; break; }
        double lat = now() - t0;
        s->ops++;
        s->lat_sum += lat;
        if (lat > s->lat_max) s->lat_max = lat;
    }
    close(fd);
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 4) { fprintf(stderr, "usage: kvload PORT CONNECTIONS SECONDS\n"); return 2; }
    port = atoi(argv[1]);
    int conns = atoi(argv[2]);
    duration = atof(argv[3]);
    pthread_t *t = calloc((size_t)conns, sizeof *t);
    Stats *st = calloc((size_t)conns, sizeof *st);
    for (int i = 0; i < conns; i++) {
        st[i].id = i;
        pthread_create(&t[i], NULL, worker, &st[i]);
    }
    Stats total = {0};
    for (int i = 0; i < conns; i++) {
        pthread_join(t[i], NULL);
        total.ops += st[i].ops;
        total.errors += st[i].errors;
        total.lat_sum += st[i].lat_sum;
        if (st[i].lat_max > total.lat_max) total.lat_max = st[i].lat_max;
    }
    printf("%3d conns: %8.0f ops/s, mean latency %7.3f ms, max %6.1f ms, errors %ld\n", conns,
           total.ops / duration, total.ops ? total.lat_sum / total.ops * 1e3 : 0, total.lat_max * 1e3,
           total.errors);
    return 0;
}
```

```console
$ cd kvd && for policy in no everysec always; do rm -rf perf-$policy; (./kvd --port 7410 --dir perf-$policy --fsync $policy 2> /dev/null & echo $! > perf.pid); sleep 0.3; echo "== fsync $policy"; for c in 1 16 64; do ./kvload 7410 $c 3; done; kill $(cat perf.pid); sleep 0.5; done
== fsync no
  1 conns:    28190 ops/s, mean latency   0.035 ms, max   12.6 ms, errors 0
 16 conns:   137520 ops/s, mean latency   0.116 ms, max    7.3 ms, errors 0
 64 conns:   111568 ops/s, mean latency   0.573 ms, max   14.7 ms, errors 0
== fsync everysec
  1 conns:    28909 ops/s, mean latency   0.034 ms, max    6.2 ms, errors 0
 16 conns:   108396 ops/s, mean latency   0.147 ms, max    6.9 ms, errors 0
 64 conns:   126021 ops/s, mean latency   0.508 ms, max   13.0 ms, errors 0
== fsync always
  1 conns:     8913 ops/s, mean latency   0.112 ms, max   19.0 ms, errors 0
 16 conns:    11418 ops/s, mean latency   1.401 ms, max   13.0 ms, errors 0
 64 conns:    10670 ops/s, mean latency   6.001 ms, max   29.1 ms, errors 0
```

How to read it:

- **`no` and `everysec`** perform alike (the differences are within run-to-run noise): a `SET` costs one `write` into the page cache. Throughput grows from 1 to 16 connections, as the server and clients overlap their work, and then flattens while latency keeps rising — the single-threaded server is saturated, and on this 2-core machine it shares the CPU with the load generator's threads.
- **`always`** is limited to about 10,000 operations per second *whatever the number of connections*. Half of those operations are `GET`s, so the server performs about 5,000 `fdatasync` calls per second — roughly 0.1–0.2 ms each on this machine's virtual disk. (An SSD that honestly flushes its cache is often slower than that; a spinning disk allows only about 100 per second.) Adding connections only adds waiting: latency grows in proportion while throughput stays flat, because each record is synced individually, inside the event loop, while every other client waits. That is the bottleneck.

The standard fix is **group commit**: execute all commands that arrived in one loop iteration, append their records, issue *one* `fdatasync`, and only then send all their replies. Throughput then scales with the number of concurrent writers while every acknowledged write remains durable — the technique every serious database uses. It is the first improvement to make in your capstone (Extension 1), and this measurement is the evidence that justifies it.

## Unit 13.8 — Operations and architecture review

**One-line summary:** A system is finished when it can be operated: configured, observed, backed up, upgraded, and reasoned about under failure — and when its designer can defend each decision with evidence and say plainly what it does not do.

### Operating kvd

- **Configuration:** port, data directory, fsync policy on the command line; a config file and `maxmemory` are natural additions.
- **Observability:** `INFO` exposes keys, commands, connections, log records, generation, and uptime; logs record recovery details, snapshots, and failures. Add latency histograms and a Prometheus-format endpoint for production.
- **Backups:** after `BGSAVE` completes, `snapshot` is a consistent, checksummed file that can be copied anywhere while the server runs.
- **Shutdown and restart:** `SIGTERM` stops accepting work, closes connections, syncs the log; recovery handles everything else.
- **Security:** the server binds to 127.0.0.1 only; exposing it needs authentication, TLS (or a tunnel), and connection limits.

### Architecture review: questions to answer with evidence

**Correctness and durability**
- Which writes survive `kill -9`? Power loss? A full disk? (Unit 13.6's tests; DT-91 for the limits of what was tested.)
- What happens if the snapshot child crashes, or the disk fills during `BGSAVE`? (Old logs are kept; the temporary file never replaces the snapshot.)
- How is a torn record distinguished from a valid one? What if a bit flips in the middle of the log? (CRC per record; conservative truncation.)

**Performance**
- What is the throughput and latency at each fsync policy, and what limits it? (Unit 13.7: the per-record fsync in the loop; group commit is the fix.)
- What does `BGSAVE` cost? (A `fork` of a large process copies page tables — milliseconds per gigabyte — and copy-on-write duplicates every page written during the snapshot; measure memory use during snapshots under write load.)

**Design judgement**
- Why one thread? (Simplicity, no locks; one core handles the target load. Scaling options: shard keys across processes, or I/O threads with a single execution thread, as Redis 6 does.)
- What would you change first? (Group commit; a binary-safe protocol such as RESP; `maxmemory` with eviction; replication for availability.)

A good answer to "what does it not do?" is part of the design, not a confession: kvd has no replication, so a disk failure loses data; no authentication; values are limited to one line of text; and recovery time grows with the log until the next snapshot.

### Extensions (your capstone)

1. **Group commit** for `--fsync always`, with a before/after load test.
2. **Automatic snapshots** when the log exceeds N records, so recovery time stays bounded.
3. **Key expiry** (`EXPIRE`, `TTL`) with lazy deletion plus a periodic sampling sweep.
4. **RESP protocol** compatibility, so `redis-cli` and `redis-benchmark` work against kvd.
5. **Replication:** a follower that connects, receives the snapshot, then streams the log.

## Where to go from here

You started this book with the four stages of compilation and ended with a database server that survives crashes and corrupted files. Along the way the same habit appeared in every chapter: the first version of an example was often wrong — a missing feature-test macro, a buffer one byte short, a benchmark the optimiser deleted, a fuzzer that lost its crash files, a 40-millisecond Nagle stall — and the tools and measurements found it. That is what writing C professionally is like. Expect the mistakes; build the warnings, sanitizers, tests, fuzzers, and measurements that catch them.

Ways to keep growing:

- **Read great C code:** SQLite (exceptionally tested), Redis, the Linux kernel's `lib/` and `mm/`, musl libc, Lua, curl.
- **Read the standard:** the C23 draft (N3220) is freely available; the CERT C Secure Coding Standard is online.
- **Go lower or higher:** the companion book on **Linux kernel and OS development** builds on Chapters 0, 3, 8, 9, and 11; the **C++** and **Rust** books show how newer languages address the memory-safety problems of Chapters 3, 5, and 12.
- **Contribute:** fix a bug with a test in an open-source C project. Code review by experienced maintainers teaches more than any book.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | "It worked when I ran it" proves nothing in C. | Unit 0.1 |
| DT-2 | The standard describes C; your compiler implements C plus extensions. | Unit 0.1 |
| DT-3 | The compiler sees one file at a time. | Unit 0.3 |
| DT-4 | `gcc` is a driver, not a compiler. | Unit 0.3 |
| DT-5 | Mismatched declarations across files are invisible to both compiler and linker. | Unit 0.4 |
| DT-6 | A leading zero makes an integer literal octal. | Unit 0.5 |
| DT-7 | Signed integer overflow is undefined; unsigned overflow wraps. | Unit 0.5 |
| DT-8 | Converting an out-of-range value to a signed type was implementation-defined until C23. | Unit 0.5 |
| DT-9 | Returning the address of a local variable creates a dangling pointer. | Unit 0.6 |
| DT-10 | Uninitialised automatic variables do not contain zero. | Unit 0.6 |
| DT-11 | A character constant has type `int` in C (but `char` in C++). | Unit 1.1 |
| DT-12 | `const` objects are not constant expressions (before C23). | Unit 1.1 |
| DT-13 | `unsigned short` × `unsigned short` can be signed overflow. | Unit 1.3 |
| DT-14 | The type of an expression decides overflow, not the variable it is stored in. | Unit 1.3 |
| DT-15 | `x == x` is false for NaN, so NaN breaks sorting and searching. | Unit 1.4 |
| DT-16 | `-ffast-math` makes the compiler assume no NaNs or infinities. | Unit 1.4 |
| DT-17 | Converting a floating-point value to an integer type that cannot hold it is undefined behaviour — it does not saturate and it does not wrap. | Unit 1.4 |
| DT-18 | `1 << 31` is undefined for a 32-bit `int`; `1u << 31` is fine. | Unit 1.5 |
| DT-19 | `%` of a negative number is negative. | Unit 1.5 |
| DT-20 | Precedence is not evaluation order. | Unit 1.6 |
| DT-21 | "Unspecified" and "undefined" are different promises: an unspecified order (function arguments, operands of `+`) makes the program choose one of several valid behaviours, possibly a different one each time; unsequenced side effects on the same object (`i = i++ + 1`) make the whole program undefined. | Unit 1.6 |
| DT-22 | A loop with no observable side effects that does not terminate is undefined behaviour in C11 and later, unless its controlling expression is a constant expression — so the compiler may assume that such a loop ends, and delete it. | Unit 1.7 |
| DT-23 | `int f();` and `int f(void);` are different declarations before C23. | Unit 1.8 |
| DT-24 | Arrays are not copied when passed — they become pointers. | Unit 1.8 |
| DT-25 | C does not guarantee tail-call elimination, so the maximum recursion depth of a program can change with the optimisation level, the compiler, and the stack limit of the thread it runs on. | Unit 1.8 |
| DT-26 | `va_arg(ap, T)` with a type that does not match the promoted type of the actual argument is undefined behaviour — and `va_arg(ap, char)`, `va_arg(ap, short)` and `va_arg(ap, float)` are always wrong, because those types never arrive in a variadic call. | Unit 1.10 |
| DT-27 | `int* a, b;` declares one pointer and one `int`. | Unit 2.1 |
| DT-28 | A null pointer is not necessarily all-zero bits, but `0` in source code is always a null pointer constant. | Unit 2.1 |
| DT-29 | Comparing or subtracting pointers into *different* objects is undefined (for `<`, `>`, `-`) even when both pointers are valid — so `if (p >= buf && p < buf + size)` is not a portable way to test whether an arbitrary pointer points into `buf`. | Unit 2.2 |
| DT-30 | `sizeof` on an array parameter gives the pointer size. | Unit 2.3 |
| DT-31 | `int a[3][4]` decays to `int (*)[4]`, not `int  | Unit 2.3 |
| DT-32 | `strncpy` is not a safe `strcpy`. | Unit 2.4 |
| DT-33 | Comparing strings with `==` compares addresses. | Unit 2.4 |
| DT-34 | `<ctype.h>` functions need `unsigned char` values. | Unit 2.4 |
| DT-35 | `char  | Unit 2.5 |
| DT-36 | `return a - b;` is a broken comparator. | Unit 2.7 |
| DT-37 | A function pointer cannot portably be stored in a `void *`. | Unit 2.7 |
| DT-38 | Calling a function through a pointer of a different function type is undefined behaviour, even when the parameter types "look compatible" — so passing `int cmp(const int *, const int *)` to `qsort` with a cast is wrong. | Unit 2.7 |
| DT-39 | `= {0}` zeroes every member, but in C17 the empty initialiser `= {}` is not valid C (it is a GCC extension and standard only from C23); and neither form promises that *padding bytes* are zero for an automatic struct. | Unit 3.1 |
| DT-40 | `p = realloc(p, n)` leaks on failure. | Unit 3.2 |
| DT-41 | On Linux, `malloc` rarely returns `NULL` — overcommit. | Unit 3.2 |
| DT-42 | A use-after-free often "works" — until the block is reused. | Unit 3.3 |
| DT-43 | Padding bytes have unspecified values, so `memcmp` on structs is unreliable. | Unit 3.5 |
| DT-44 | Strict-aliasing violations show up only under optimisation. | Unit 3.6 |
| DT-45 | Whether a plain `int` bit-field is signed or unsigned is implementation-defined, and almost everything else about bit-field layout is too: the order of fields within a unit, whether a field may straddle a boundary, and the size of the storage unit. | Unit 3.7 |
| DT-46 | Global variables are shared mutable state across the whole program. | Unit 4.1 |
| DT-47 | A plain `inline` function can fail to link at `-O0`. | Unit 4.2 |
| DT-48 | `#` does not expand its argument; you need two levels to stringise a macro's value. | Unit 4.3 |
| DT-49 | Code inside `assert` disappears in release builds. | Unit 4.3 |
| DT-50 | Without header dependencies, Make silently links stale objects. | Unit 4.4 |
| DT-51 | Library order on the command line matters for static libraries. | Unit 4.5 |
| DT-52 | Changing a struct in a shared library's public header breaks every program compiled against the old one. | Unit 4.5 |
| DT-53 | A cache variable keeps the value from the first configure: changing the `option()` default or a `set(... CACHE ...)` default in `CMakeLists.txt` has no effect on an existing build directory, and a `-D` given once stays until it is changed or the cache is deleted. | Unit 4.6 |
| DT-54 | Undefined behaviour can affect code that runs *before* it. | Unit 5.1 |
| DT-55 | A null check after a dereference can be deleted. | Unit 5.1 |
| DT-56 | Sanitizers only see the paths you run. | Unit 5.3 |
| DT-57 | AddressSanitizer does not detect reads of uninitialised memory at all; Valgrind's Memcheck does (at the point where the value influences behaviour), and Clang's MemorySanitizer does at compile-instrumented speed — so "clean under ASan" says nothing about uninitialised data. | Unit 5.4 |
| DT-58 | After `longjmp`, the values of local variables of the function that called `setjmp` are indeterminate if they are not `volatile` and were changed between the `setjmp` and the `longjmp`. | Unit 5.6 |
| DT-59 | After `fork`, unflushed stdio buffers are duplicated. | Unit 6.1 |
| DT-60 | `%f` in `printf` is `double`, but in `scanf` it is `float`. | Unit 6.2 |
| DT-61 | `fclose` can fail, and that is where write errors appear. | Unit 6.3 |
| DT-62 | Measure elapsed time with `CLOCK_MONOTONIC`, never with `time()`, `gettimeofday()` or `CLOCK_REALTIME`: the wall clock can jump backwards or forwards at any moment — NTP corrections, a manual change, a leap-second smear — so durations computed from it can be negative or wildly wrong. | Unit 6.5 |
| DT-63 | `wchar_t` is not "a Unicode character": its size and encoding are implementation-defined — 32-bit UTF-32 on Linux and macOS, 16-bit UTF-16 code units on Windows, where characters outside the Basic Multilingual Plane (such as most emoji) take two `wchar_t` values. | Unit 6.8 |
| DT-64 | Big-O hides constants that can dominate. | Unit 7.1 |
| DT-65 | A linked-list benchmark that builds its list with consecutive `malloc` calls and walks it right away measures a best case that real programs rarely see: the allocator places the nodes next to each other, so the "pointer chase" streams through memory like an array. | Unit 7.2 |
| DT-66 | Free-running unsigned indices — `head` and `tail` that only ever increase and wrap at 2^32 — work only when the capacity divides 2^32, that is, when it is a power of two. With any other capacity, `index % cap` jumps when the counter wraps, and the ring silently reorders or overwrites elements. | Unit 7.3 |
| DT-67 | Never iterate a hash table and rely on the order. | Unit 7.4 |
| DT-68 | `read` and `write` may transfer less than requested — and it is not an error. | Unit 8.1 |
| DT-69 | Descriptors leak into child programs unless marked close-on-exec. | Unit 8.1 |
| DT-70 | Check-then-use on file names is a race (TOCTOU). | Unit 8.2 |
| DT-71 | A child that has called `fork` must leave with `_exit`, not `exit` or `return from main`, when it does not `exec` — and after a failed `exec` too. `exit` runs the parent's `atexit` handlers and flushes the stdio buffers the child inherited, so output written by the parent before the `fork` appears twice, and cleanup meant to run once (deleting a temporary file, sending a "goodbye" message) runs in both processes. | Unit 8.3 |
| DT-72 | A reader on a pipe sees end-of-file only when *every* write descriptor for that pipe is closed — including the copies inherited through `fork` by processes that never write. Forget to close one, and the reader blocks forever. | Unit 8.4 |
| DT-73 | `signal()` semantics vary; use `sigaction`. | Unit 8.5 |
| DT-74 | Accessing a page of a file mapping that lies beyond the current end of the file raises `SIGBUS`, not `SIGSEGV` — so if another process truncates a file while you have it mapped, your next read of the vanished part kills you. | Unit 8.6 |
| DT-75 | Passing `&i` — the address of a loop variable — as the argument of `pthread_create` is a data race: the thread reads `i` whenever it gets around to it, while the loop keeps changing it, so several threads may see the same value and some values may be skipped. | Unit 9.1 |
| DT-76 | `volatile` does not make code thread-safe. | Unit 9.2 |
| DT-77 | Changing the shared state and signalling must happen with the mutex held — or at least the state change must; signalling a condition variable without first changing the predicate under the mutex can wake nobody, because a thread that checked the predicate just before may not yet be waiting: the "lost wake-up". | Unit 9.3 |
| DT-78 | Plain `int` flags between threads are a data race, even on x86. | Unit 9.5 |
| DT-79 | Forgetting `htons` on a port number does not cause an error — the program binds or connects to a different port: on a little-endian machine port 8080 (0x1F90) becomes 36895 (0x901F). | Unit 10.1 |
| DT-80 | A UDP datagram larger than the receive buffer is truncated, and the rest is discarded silently: `recvfrom` returns the buffer size, not an error, unless you ask with `MSG_TRUNC`, which makes Linux return the datagram's real length. | Unit 10.4 |
| DT-81 | Writing to a TCP socket whose peer has closed the connection raises `SIGPIPE`, and the default action of `SIGPIPE` is to terminate the process — so one client that disconnects at the wrong moment can kill a whole server that never installed a handler. | Unit 10.5 |
| DT-82 | One slow operation in an event loop delays every client. | Unit 10.6 |
| DT-83 | Two small writes in a row can cost 40 milliseconds: Nagle's algorithm meets delayed ACK. | Unit 10.6 |
| DT-84 | `select` can only watch descriptors numbered below `FD_SETSIZE` (1024 on Linux), and `FD_SET` with a larger descriptor writes past the end of the `fd_set` — memory corruption, not an error. A long-running server that opens more than about a thousand files or sockets crosses that line without noticing. | Unit 10.8 |
| DT-85 | Calling a function through a wrong prototype breaks the ABI silently. | Unit 11.2 |
| DT-86 | A linked list of N elements can be 10–100× slower to traverse than an array of N elements. | Unit 11.4 |
| DT-87 | `-ffast-math` is not one optimisation but a bundle of promises you make to the compiler — no NaNs, no infinities, no signed zeros, no trapping, associativity allowed — and breaking a promise makes the program's results, not just its speed, change; it even affects other code, because linking with `-ffast-math` can switch the CPU into "flush denormals to zero" mode for the whole process. | Unit 11.6 |
| DT-88 | An off-by-one write of a single zero byte can be exploitable. | Unit 12.1 |
| DT-89 | `calloc(n, size)` must fail if `n * size` overflows, but `malloc(n * size)` cannot know that an overflow happened — the multiplication was done, and wrapped, before `malloc` was called. So replacing `calloc` with `malloc` "for speed" can turn an allocation failure into a heap overflow, and so can `realloc(p, n * size)`. | Unit 12.3 |
| DT-90 | Checking a path for `..` with string operations does not make it safe to open: symbolic links, absolute paths, encodings (Unit 6.8's overlong forms, `%2e%2e` in URLs), and a file replaced between the check and the `open` all bypass it. On Linux, the robust tool is `openat2` with `RESOLVE_BENEATH`, which makes the kernel refuse any path that escapes the given directory — during the lookup itself. | Unit 12.5 |
| DT-91 | `write` returning success does not mean the data is on disk. | Unit 13.3 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Compile every change with warnings and sanitizers; ship with warnings as errors. | Unit 0.7 |
| P-2 | Read lines, then parse; validate every conversion completely. | Unit 1.9 |
| P-3 | Every printf-like function you write gets `__attribute__((format(printf, N, M)))`, and forwards to the `v` version of the library function (`vfprintf`, `vsnprintf`) instead of re-parsing the format. | Unit 1.10 |
| P-4 | Pair every allocation with a documented owner and a single release point. | Unit 3.3 |
| P-5 | Opaque handle + constructor/destructor pair for every non-trivial type. | Unit 3.8 |
| P-6 | `static` by default; external linkage only for the public interface. | Unit 4.1 |
| P-7 | Check every return value; propagate errors with context; assert only invariants. | Unit 5.6 |
| P-8 | Generic containers: `void *` for cold code, generated code for hot code. | Unit 7.9 |
| P-9 | One lock per piece of shared state, acquired in a documented global order, held briefly. | Unit 9.4 |
| P-10 | Per-connection receive buffer + incremental parser + hard limits. | Unit 10.3 |
| P-11 | Profile, form a hypothesis about the hardware, change one thing, measure again. | Unit 11.7 |
| P-12 | Parse untrusted input with a small, fuzzed, bounds-checked parser that produces validated structures. | Unit 12.5 |

# Appendix B — Glossary

Short definitions of the terms used in this book. The unit given (where there is one) explains the term fully.

| Term | Meaning |
| --- | --- |
| ABI (application binary interface) | The binary-level contract: calling convention, type sizes, struct layout, symbol names (Unit 11.2) |
| Abstract machine | The model of execution the C standard describes; compilers must preserve only its observable behaviour (Unit 0.1) |
| Acquire / release | Memory orders that publish data from one thread to another (Unit 9.5) |
| Address space | The range of virtual addresses a process can use |
| AddressSanitizer (ASan) | Compiler instrumentation that detects out-of-bounds accesses, use-after-free, double free, and leaks at run time |
| Aliasing | Two pointers referring to the same memory; limits optimisation (Unit 11.3) |
| Alignment | The requirement that an object's address be a multiple of a value such as 4 or 8 (Unit 3.5) |
| Amortised cost | Average cost per operation over a sequence, e.g. dynamic-array push is O(1) amortised |
| Arena allocator | Allocates many objects from large chunks and frees them all at once (Chapter 3 project) |
| Array decay | Conversion of an array to a pointer to its first element in most expressions (Unit 2.3) |
| ASLR | Address space layout randomisation: random placement of stack, heap, and libraries |
| Assembler | The tool that turns assembly text into machine code in an object file |
| Atomic operation | An indivisible memory operation that is free of data races (`<stdatomic.h>`) |
| Big-endian / little-endian | Byte orders for multi-byte values: most significant first / least significant first |
| Bit-field | A struct member with an explicit width in bits (Unit 3.7) |
| Branch prediction | CPU guessing the direction of branches to execute ahead; mispredictions cost cycles (Unit 11.5) |
| BSS | The section of zero-initialised static data; takes no space in the executable file |
| Buffer overflow | Writing past the end of a buffer; the classic memory-safety vulnerability (Unit 12.1) |
| Cache line | The unit of transfer between memory and CPU caches, 64 bytes on x86-64 |
| Calling convention | Rules for passing arguments and results and preserving registers across calls |
| Canary (stack protector) | A random value checked before a function returns, detecting stack overflows |
| Compound literal | An unnamed object created in an expression: `(struct point){1, 2}` |
| Condition variable | Lets threads sleep until shared state changes; always wait in a loop (Unit 9.3) |
| Copy-on-write | Sharing pages between processes until one writes, then copying (fork, MAP_PRIVATE) |
| CRC-32 | A checksum used to detect corrupted or torn data (Unit 13.3) |
| CSR (compressed sparse row) | Graph layout in flat arrays of offsets and edges (Unit 7.8) |
| Dangling pointer | A pointer to an object whose lifetime has ended |
| Data race | Concurrent unsynchronised accesses to the same location, at least one a write: undefined behaviour |
| Deadlock | Threads waiting for each other forever, typically through opposite lock orders |
| Declaration / definition | Introducing a name and type / also creating the object or function body |
| Designated initialiser | Initialising named members or indexes: `{.y = 3}`, `{[2] = 5}` |
| Dynamic linking | Resolving library functions at load time from shared objects (`.so`) |
| ELF | Executable and Linkable Format: the object and executable format on Linux |
| epoll | Linux's scalable API for waiting on many file descriptors (Unit 10.6) |
| errno | Thread-local error code set by failing library functions and system calls |
| Event loop | A single thread that waits for ready descriptors and handles each without blocking |
| Exit status | The integer a process returns to its parent; 0 means success |
| False sharing | Independent variables on one cache line slowing threads that write them (Unit 9.7) |
| Feature-test macro | A macro like `_POSIX_C_SOURCE` defined before includes to request extra declarations |
| File descriptor | Small integer referring to an open file, pipe, socket, or device |
| Flexible array member | A last struct member `T name[];` sized at allocation time (Unit 3.8) |
| fork / exec | Create a copy of the process / replace the program running in a process (Unit 8.3) |
| Fortify source | `_FORTIFY_SOURCE`: glibc checks for overflows in common functions |
| Framing | Finding message boundaries in a byte stream (Unit 10.3) |
| fsync / fdatasync | Force file data to stable storage (DT-91) |
| Fuzzing | Automatically generating inputs to find crashes and bugs (Unit 12.6) |
| GDB | The GNU debugger (Unit 5.5) |
| Generic selection (`_Generic`) | Compile-time choice of an expression by type (C11) |
| Hash table | Key-value structure with O(1) average lookup by hashing (Unit 7.4) |
| Header file | A file of declarations shared between translation units |
| Heap (data structure) | Array-based tree giving a priority queue (Unit 7.6) |
| Heap (memory) | Memory obtained from `malloc` and released with `free` |
| Implementation-defined behaviour | Behaviour the compiler chooses and documents (e.g. size of `int`) |
| Include guard | `#ifndef X_H / #define X_H / #endif` preventing double inclusion |
| Indeterminate value | The value of an uninitialised automatic object; using it is a bug |
| Inline assembly | Assembly embedded in C with `__asm__` (Unit 11.2) |
| Integer promotion | Conversion of types narrower than `int` to `int` before arithmetic (Unit 1.3) |
| Internal / external linkage | Name visible only in its file (`static`) / across the program |
| Intrusive list | A list whose links are embedded in the elements, used with `container_of` |
| libFuzzer | Clang's in-process coverage-guided fuzzer |
| Linker | Combines object files and libraries into an executable, resolving symbols |
| Load factor | Fraction of a hash table's slots in use |
| Lock-free | Algorithms that make progress without locks, using atomics |
| LTO | Link-time optimisation: optimising across translation units |
| Macro | A preprocessor text substitution (`#define`) |
| Memory leak | Allocated memory that is never freed |
| Memory model | Rules for visibility and ordering of memory operations between threads |
| Memory order | `relaxed`, `acquire`, `release`, `acq_rel`, `seq_cst` (Unit 9.5) |
| MemorySanitizer (MSan) | Clang tool detecting reads of uninitialised memory |
| mmap | Map files or anonymous memory into the address space (Unit 8.6) |
| Mutex | Lock providing mutual exclusion (`pthread_mutex_t`) |
| Nagle's algorithm | TCP batching of small segments; combined with delayed ACK can add 40 ms (DT-83) |
| NaN | Not a Number: IEEE 754 result of undefined operations; unequal to itself |
| Non-blocking I/O | I/O calls that return `EAGAIN` instead of waiting |
| Null pointer | A pointer value pointing to no object (`NULL`, C23 `nullptr`) |
| Object file | Compiled machine code plus symbols, before linking (`.o`) |
| Opaque type | A struct declared but not defined in a header, hiding its layout (Unit 3.8) |
| Ownership | Which part of a program is responsible for freeing an object (Unit 3.3) |
| Padding | Unused bytes inserted in structs for alignment |
| PGO | Profile-guided optimisation using data from a training run |
| PIE | Position-independent executable, loadable at any address |
| Pipe | A one-way kernel channel between processes (Unit 8.4) |
| Pointer arithmetic | Adding integers to pointers moves by whole elements (Unit 2.2) |
| POSIX | The standard API of Unix-like operating systems |
| Preprocessor | The first build stage: includes, macros, conditional compilation |
| Process | A running program with its own address space |
| pthreads | POSIX threads API |
| Race condition | Outcome depending on timing between threads or processes |
| RELRO | Making dynamic-linking tables read-only after start-up |
| restrict | Qualifier promising a pointer is the only way to access its object (Unit 11.3) |
| Ring buffer | Fixed array used as a queue with wrapping indices (Unit 7.3) |
| Sanitizer | Compiler instrumentation that detects classes of bugs at run time |
| Sequence point / sequencing | Rules for the order of side effects within expressions (Unit 1.6) |
| Shared library | Library loaded at run time (`.so`), shared by processes (Unit 4.5) |
| Signal | Asynchronous notification to a process (Unit 8.5) |
| SIMD | Single instruction, multiple data: vector instructions (Unit 11.6) |
| size_t | Unsigned type of sizes and `sizeof` results |
| Snapshot | A complete copy of a database's state at a point in time (Unit 13.4) |
| Socket | Endpoint for network communication |
| soname | A shared library's versioned name recorded by programs that use it |
| Stack (memory) | Per-thread memory for function frames and automatic variables |
| Static library | Archive of object files copied into executables (`.a`) |
| Storage duration | Static, automatic, allocated, or thread lifetime of an object |
| Strict aliasing | Rule that objects may be accessed only through compatible types (Unit 3.6) |
| String literal | Text in double quotes; a read-only array of `char` ending in `'\0'` |
| Struct | Aggregate type of named members |
| System call | Request from a process to the kernel |
| Tagged union | A union plus a field saying which member is valid |
| Tentative definition | A file-scope declaration without initialiser that may become a definition |
| ThreadSanitizer (TSan) | Tool detecting data races and lock-order inversions |
| Tombstone | Marker for a deleted slot in an open-addressing hash table |
| Translation unit | A source file after preprocessing, compiled independently |
| Two's complement | The representation of signed integers on all modern machines (required in C23) |
| Undefined behaviour (UB) | Behaviour for which the standard imposes no requirements (Unit 5.1) |
| Union | Type whose members share the same storage |
| Unspecified behaviour | Behaviour with several allowed outcomes, not necessarily documented |
| Usual arithmetic conversions | Rules bringing operands to a common type (Unit 1.3) |
| Valgrind | A dynamic analysis framework; Memcheck finds memory errors without recompiling |
| Variable-length array (VLA) | Array whose size is determined at run time (optional since C11) |
| volatile | Qualifier forcing every access to happen; not a synchronisation tool (DT-76) |
| Write-ahead log (WAL) | An append-only record of changes written before they are acknowledged (Unit 13.3) |
| Zombie process | An exited child not yet waited for |

# Appendix C — C standards and toolchain support

C is defined by ISO/IEC 9899. Each revision is mostly backwards compatible; select one with `-std=`. The `gnu` variants (`-std=gnu17`) add GCC extensions; this book uses the strict `c17` mode so that non-standard code is reported.

| Standard | Year | Features used or discussed in this book |
| --- | --- | --- |
| C89/C90 | 1989/1990 | the core language: types, pointers, structs, the preprocessor, `<stdio.h>`, `<stdlib.h>`, `<string.h>` |
| C99 | 1999 | `//` comments; declarations anywhere; `long long`; `<stdint.h>`/`<inttypes.h>`; `_Bool`/`<stdbool.h>`; designated initialisers; compound literals; flexible array members; variable-length arrays; `inline`; `restrict`; `snprintf`; `__func__` |
| C11 | 2011 | `_Generic`; `_Static_assert`; `_Alignas`/`_Alignof`; anonymous structs and unions; `_Noreturn`; `<threads.h>`; `<stdatomic.h>` and the memory model; `_Thread_local`; `aligned_alloc`; `fopen` mode `x`; `quick_exit`; removal of `gets`; VLAs made optional |
| C17 | 2018 | defect fixes only — **the baseline of this book** |
| C23 | 2024 | `bool`, `true`, `false`, `nullptr`, `static_assert`, `alignas`, `thread_local` as keywords; `constexpr` objects; `typeof`; `auto` type inference; `[[attributes]]` (`nodiscard`, `maybe_unused`, `fallthrough`, `noreturn`, `deprecated`); binary literals and digit separators; `= {}` initialisers; enums with fixed underlying types; `<stdckdint.h>` (`ckd_add`, …); `<stdbit.h>`; `#embed`; `memset_explicit`; `strdup`/`strndup`; two's complement required; `()` means `(void)` |

## Compiler support used in this book

| Feature set | GCC 13 | GCC 14 | Clang 18 |
| --- | --- | --- | --- |
| C17 | full | full | full |
| C23 core keywords, `typeof`, attributes, binary literals | yes (`-std=c2x`) | yes (`-std=c23`) | mostly (`-std=c23`) |
| C23 `constexpr` objects | no | yes | no (Clang 19+) |
| `<stdckdint.h>` | no (use `__builtin_add_overflow`) | yes | yes |
| `#embed` | no | no (GCC 15) | no (Clang 19) |

`<stdbit.h>` is provided by the C library (glibc 2.39+), not the compiler.

## Common extensions used and marked in the text

`__attribute__((...))` (`noinline`, `noipa`, `packed`, `aligned`, `target`, `warn_unused_result`, `fallthrough`, `visibility`), statement expressions `({ ... })`, `__typeof__`, `__builtin_expect`, `__builtin_cpu_supports`, `__builtin_*_overflow`, and extended inline assembly — all supported by both GCC and Clang.
