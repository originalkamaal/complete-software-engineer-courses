---
title: "C++ from Zero to Expert"
subtitle: "A complete course in modern C++ (C++20 and C++23) — from the compilation model to a tested, measured order-matching engine"
lang: en
---

# Preface

C++ is the language of performance-critical software: browsers, game engines, databases, compilers, trading systems, robotics, machine-learning runtimes, and much of the operating system's user space. It gives you the machine-level control of C, plus abstractions — classes, templates, RAII, the standard library — that cost nothing at run time when used well.

C++ is also large, and much of what is written about it is out of date. This book teaches **modern C++**: C++20 as the baseline, with C++23 features where today's compilers support them. It treats the language the way experienced engineers do: value semantics and RAII by default; ownership expressed in types; the standard library before hand-written code; templates constrained with concepts; and every claim about correctness or speed checked with tools — warnings, sanitizers, debuggers, static analysis, fuzzers, benchmarks, and profilers.

The book begins with what a C++ compiler and linker actually do, then covers the core language, classes and RAII, move semantics, the standard library, smart pointers, error handling, templates and concepts, polymorphism, concurrency and coroutines, build and test tooling, performance engineering, and undefined behaviour and safety. It ends with a capstone: a limit order book and matching engine, verified against a reference model and measured to nanosecond percentiles.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the C++ standard, the standard library's specification, a particular implementation, the compiler, or established practice.
2. **Evidence.** Every program in this book was compiled and run, and every output shown is real — including compiler errors, warnings, sanitizer reports, and benchmark numbers. When the first version of an example was wrong, or an optimisation did not pay off (several did not), the book shows that too, because that is how real C++ development goes.
3. **Order.** Nothing is used before it is taught, and later chapters refer back by unit number.

This book is a companion to *C from Zero to Systems*. You do not need to read the C book first — Chapter 0 and Chapter 1 cover what you need — but where a topic is the same in both languages (the machine model, the ABI, POSIX, caches, fuzzing), this book gives the C++ view and refers to the C book for depth.

# How to use this book

## Who it is for

- **Beginners** who are willing to work carefully: start at Chapter 0 and read in order. No previous programming is assumed, though it helps.
- **C programmers:** read Chapters 0–1 quickly, then slow down at Chapter 2: RAII, value semantics, and move semantics (Chapters 2–3) are where C++ thinking differs most from C.
- **Programmers who learned "C++98 with classes":** start at Chapter 3 and do every *Deep & Tricky* entry; Chapters 3, 5, 8, and 13 cover what has changed most.
- **Working C++ programmers:** use Appendix A to test yourself, then focus on Chapters 8, 10, 12, and 13.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | The rules, with small complete programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, each labelled with its source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Explain*, *Predict* (decide the output before running), *Write* (code it). Answers to prediction questions are at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a specification, a complete reference solution with real output, and a review checklist. Try the project yourself first.

## How to study

1. **Type the programs and run them,** with the flags shown. Compare your output with the book's.
2. **Predict before you run.** When you are wrong, find out why before moving on.
3. **Read compiler errors fully.** Template errors are long but structured; Chapters 7 and 8 show how to read them, and concepts make them short.
4. **Break things on purpose.** Remove a `std::move`, return a reference to a local, share data between threads without a lock — and watch which tool catches it.
5. **Use the tools from day one:** `-Wall -Wextra`, `-fsanitize=address,undefined`, and a debugger. Chapter 0 shows how; Chapters 11 and 13 explain them fully.
6. **Revise with Appendix A,** which lists every DT and P entry by title.

## How to read the labels

| Label | Meaning |
| --- | --- |
| *Guarantee: C++ standard* (and variants naming a rule, e.g. *special member generation rules*) | Required by ISO C++. True on every conforming implementation. |
| *Guarantee: UB* (… *on dangling reference*, *double delete*, …) | The standard says the behaviour is undefined: no requirements at all. |
| *ODR violation, no diagnostic required* | The program is ill-formed, but the implementation need not tell you. |
| *Library guarantee*, *Library behaviour*, *Library precondition* | Specified by the standard library's specification, or how libstdc++ behaves where the standard leaves freedom. |
| *Compiler behaviour*, *Optimiser behaviour*, *Implementation behaviour* | What GCC, Clang, or the library implementation do. Stable in practice; may change between versions. |
| *Language rules*, *Standard, [section]* | A rule of the language, with the section of the standard where it is specified. |
| *Practice*, *Core Guidelines* | Professional practice, or a rule from the C++ Core Guidelines — experience, not language rules. |

## Toolchain and versions

All code was compiled and run on **Linux x86-64 (Ubuntu 24.04)** with:

- **GCC 14.2** (`g++-14`, the default compiler in this book) and **GCC 13.3**,
- **Clang 18** (`clang++`: sanitizers including ThreadSanitizer, libFuzzer, `clang-tidy`, `clang-format`),
- **libstdc++** from GCC 14, **CMake 3.28**, **GoogleTest 1.14**, **Google Benchmark 1.8**, **GDB 15**, **Valgrind 3.22**, **perf** (Linux 6.8 tools), **oneTBB** (for parallel algorithms).

The language baseline is **C++20** (`-std=c++20`). **C++23** features are marked and compiled with `-std=c++23`. Some C++23 library features are not yet complete in GCC 14's libstdc++ — notably formatting of ranges (`std::println("{}", vec)`) — and the text says so where it matters. Appendix C lists which standard introduced each feature and what each compiler supports.

Measurements (Chapters 6, 10, 11, 12, 14) were taken on a 2-core virtual machine. Your numbers will differ; compare the ratios and the shapes, and expect some run-to-run variation, which the text reports where it is large. Hardware performance counters were not available in the virtual machine, so profiles use software clock sampling.

## Setting up

1. Install a compiler and tools. On Debian/Ubuntu 24.04: `sudo apt install g++-14 clang clang-tidy clang-format gdb valgrind cmake libgtest-dev libbenchmark-dev libtbb-dev`. On Fedora: `sudo dnf install gcc-c++ clang clang-tools-extra gdb valgrind cmake gtest-devel google-benchmark-devel tbb-devel`. On macOS: install Xcode command-line tools for Apple Clang, or `brew install llvm gcc cmake` for current compilers. On Windows: Visual Studio 2022 supports nearly everything here; use WSL2 for the POSIX, sanitizer, and `perf` sections.
2. Check: `g++-14 --version`, `clang++ --version`, `cmake --version`.
3. Use an editor with **clangd** (VS Code, CLion, Neovim, Emacs, Zed). CMake's `compile_commands.json` (Chapter 11) makes it understand your project.
4. Create a working folder with one subfolder per chapter, and keep a journal of the predictions you got wrong.

# Roadmap

| Chapter | Title | You will be able to… |
| --- | --- | --- |
| 0 | The C++ Toolchain and Compilation Model | follow C++ through compiling and linking; explain the ODR, name mangling, `extern "C"`, and modules |
| 1 | Core Language | use types, initialisation, references, `const`/`constexpr`, functions, lambdas, strings, and formatted output |
| 2 | Classes and RAII | design classes with invariants, constructors, destructors, copying, and operators; tie resources to lifetimes |
| 3 | Value Semantics and Move Semantics | explain value categories; write moves; use perfect forwarding; pass and return values efficiently |
| 4 | The Standard Library | choose containers; use iterators, algorithms, ranges, and views; know the invalidation rules |
| 5 | Smart Pointers and Ownership | express ownership with `unique_ptr`, `shared_ptr`, `weak_ptr`, and scope guards |
| 6 | Error Handling | use exceptions with the safety guarantees, `noexcept`, and `std::expected`; know what each costs |
| 7 | Templates I | write function and class templates; understand deduction, CTAD, specialisation, and instantiation |
| 8 | Templates II | constrain templates with concepts; write variadic templates and compile-time code |
| 9 | Polymorphism | use inheritance and virtual dispatch correctly; use `std::variant`, CRTP, and type erasure |
| 10 | Concurrency | use threads, locks, condition variables, atomics, the memory model, tasks, parallel algorithms, and coroutines |
| 11 | Building, Testing, and Tooling | build with modern CMake; test with GoogleTest; benchmark; lint; run sanitizers in CI |
| 12 | Performance Engineering | remove hidden allocations and copies; use `pmr`; design cache-friendly data; optimise by measurement |
| 13 | Undefined Behaviour and Safe C++ | prevent dangling and invalidation bugs; use hardened libraries, sanitizers, and fuzzing |
| 14 | Capstone: An Order-Matching Engine | design, build, verify, and measure a performance-critical C++ system |
| A | Index of Deep & Tricky entries and Patterns | revise every entry by title |
| B | Glossary | look up any term |
| C | C++ standards and compiler support | see which standard introduced each feature, and what compilers support |

Chapters 0–6 are the **foundation**; read them in order. Chapters 7 and 8 belong together. After that, Chapters 9–13 can be read in any order (Chapter 12 uses Chapter 11's benchmarking, and Chapter 13 uses Chapter 11's sanitizers). Chapter 14 uses the whole book.

# Chapter 0 — The C++ Toolchain and Compilation Model

*Goal: understand how C++ source becomes a running program — and the rules (translation units, the One Definition Rule, name mangling) that explain most confusing build errors.*

C++ is a compiled, statically typed language designed by Bjarne Stroustrup in 1979 as "C with Classes", standardised by ISO in 1998, and modernised every three years since 2011. It keeps C's model — separate compilation, a linker, direct access to memory, no garbage collector — and adds classes with constructors and destructors, templates, exceptions, a large standard library, and compile-time computation. The result is the language of game engines, browsers, databases, trading systems, compilers, and most performance-critical software.

This chapter sets up the toolchain and explains the compilation model. If you have read *C from Zero to Systems*, much of it will be familiar; the C++-specific parts — name mangling, the One Definition Rule, inline and template definitions in headers, and modules — are new.

**What you will be able to do after this chapter**

- Compile, link, and run C++ programs with GCC and Clang, with a professional set of warning and sanitizer flags.
- Explain the stages of a C++ build and identify which stage produced an error.
- Explain the One Definition Rule and why templates and `inline` functions live in headers.
- Read mangled names, and call C code from C++ (and C++ from C) with `extern "C"`.
- Know which C++ standard you are using and what each revision added.

## Topic 0A — C++ and its tools

## Unit 0.1 — What C++ is

**One-line summary:** C++ is a multi-paradigm language — procedural, object-oriented, generic, and functional — whose core design principle is the **zero-overhead abstraction**: what you don't use, you don't pay for, and what you do use, you couldn't hand-code any better. It is defined by the ISO standard; this book targets **C++20** and marks **C++23** features.

### Versions that matter

| Standard | Highlights |
| --- | --- |
| C++98/03 | classes, templates, exceptions, the STL (containers, iterators, algorithms) |
| **C++11** | the "modern C++" revolution: move semantics, `auto`, lambdas, `unique_ptr`/`shared_ptr`, range-`for`, `constexpr`, `nullptr`, threads and the memory model, variadic templates, `enum class` |
| C++14 | generic lambdas, `make_unique`, relaxed `constexpr` |
| C++17 | structured bindings, `if constexpr`, `std::optional`, `std::variant`, `std::string_view`, fold expressions, parallel algorithms, guaranteed copy elision, `std::filesystem` |
| **C++20** | **concepts**, **ranges**, **coroutines**, **modules**, `std::format`, `std::span`, three-way comparison `<=>`, `consteval`, `std::jthread` — **this book's baseline** |
| C++23 | `std::expected`, `std::print`/`println`, `std::generator`, deducing `this`, `std::mdspan`, `std::flat_map`, `import std;` |
| C++26 | (finalised 2026) static reflection, contracts, `std::execution`, hardened standard library |

### Undefined behaviour, again

C++ inherits C's **undefined behaviour** (out-of-bounds access, signed overflow, use after free, data races) and adds its own (using a moved-from object in ways its type does not allow is usually fine, but dangling references to temporaries, iterator invalidation, and violating a class's invariants through casts are common UB sources). Chapter 13 is devoted to them; every chapter points them out.

### Deep & Tricky

**DT-1. "C++ is a superset of C" is false.** *(Guarantee: both standards)* Valid C can be invalid or behave differently in C++: `int *p = malloc(4);` needs a cast in C++; `sizeof('a')` is 1 in C++ but `sizeof(int)` in C; C++ has no variable-length arrays or designated initialisers in arbitrary order; C++ reserves keywords like `class`, `new`, `template`. Write each language as itself, and connect them with `extern "C"` (Unit 0.4).

### Mastery check

1. **Explain.** What does "zero-overhead abstraction" promise, and what does it not promise?

## Unit 0.2 — Building and running C++ programs

**One-line summary:** `g++` (GCC) or `clang++` compile and link C++; always choose a standard (`-std=c++20`), enable warnings (`-Wall -Wextra -pedantic`), and during development use sanitizers (`-fsanitize=address,undefined`) and the standard library's debug assertions (`-D_GLIBCXX_ASSERTIONS`).

```cpp
// file: hello.cpp
#include <iostream>
#include <string>
#include <vector>

int main() {
    std::vector<std::string> words{"Hello", "C++", "world"};
    for (const auto& w : words)
        std::cout << w << ' ';
    std::cout << '\n';
    return 0;
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -pedantic hello.cpp -o hello && ./hello
Hello C++ world 
```

Already this tiny program shows C++'s character: `std::vector` is a growable array that frees its memory automatically when it goes out of scope; `std::string` owns its characters; `const auto&` binds a reference to each element without copying; `std::cout <<` is type-safe output. There is no `malloc`, no `free`, and no buffer size to get wrong.

### The flags this book uses

```sh
CXXFLAGS="-std=c++20 -Wall -Wextra -pedantic -g -fsanitize=address,undefined -D_GLIBCXX_ASSERTIONS"
```

- `-D_GLIBCXX_ASSERTIONS` turns on bounds checks in libstdc++'s `operator[]`, `front()`, `back()`, and similar functions — cheap, and it turns many silent bugs into clear aborts.
- Release builds: `-O2` (and often `-DNDEBUG`), without sanitizers.
- GCC 14 (`g++-14`) is used throughout because it supports almost all of C++20 and much of C++23; Clang 18 is used where its diagnostics or tools are better.

```cpp
// file: bounds.cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3};
    std::cout << v[3] << '\n';     // out of bounds: undefined behaviour
}
```

```console
$ g++-14 -std=c++20 bounds.cpp -o bounds_plain && ./bounds_plain
0
$ g++-14 -std=c++20 -D_GLIBCXX_ASSERTIONS bounds.cpp -o bounds_checked && ./bounds_checked 2>&1 | head -n 2; echo "exit status: ${PIPESTATUS[0]}"
/usr/include/c++/14/bits/stl_vector.h:1130: constexpr std::vector<_Tp, _Alloc>::reference std::vector<_Tp, _Alloc>::operator[](size_type) [with _Tp = int; _Alloc = std::allocator<int>; reference = int&; size_type = long unsigned int]: Assertion '__n < this->size()' failed.
exit status: 134
```

Without assertions the program printed whatever followed the array in memory and exited normally. With assertions, the library caught the bad index and aborted with the exact precondition that failed.

<!-- expanded -->
### Going deeper: `g++` versus `gcc`

`g++` and `gcc` are the same compiler driver with different defaults. Both compile `.cpp` files as C++, but only `g++` adds the C++ standard library (`libstdc++`) and its runtime support when it links. Compiling a C++ program with `gcc` therefore succeeds and linking fails, with dozens of "undefined reference" errors for names that live in the library:

```cpp
// file: which_driver.cpp
#include <iostream>
#include <string>

int main() {
    std::string s = "linked by the right driver";
    std::cout << s << '\n';
}
```

```console
$ gcc-14 -std=c++20 which_driver.cpp -o which_driver 2>&1 | grep -c "undefined reference"; gcc-14 -std=c++20 which_driver.cpp -o which_driver 2>&1 | grep -m2 -oE "undefined reference to \`[^']*'" | cut -c1-90
12
undefined reference to `std::cout'
undefined reference to `std::basic_ostream<char, std::char_traits<char> >& std::operator<<
$ g++-14 -std=c++20 which_driver.cpp -o which_driver && ./which_driver && ldd which_driver | grep -oE "libstdc\+\+\S*|libc\.so\S*|libm\.so\S*|libgcc_s\S*" | sort -u
linked by the right driver
libc.so.6
libgcc_s.so.1
libm.so.6
libstdc++.so.6
```

The executable depends on `libstdc++.so.6` (the C++ library), `libm` (which `libstdc++` uses), `libgcc_s` (stack unwinding for exceptions), and `libc`. A C++ program is a C program plus these — which is also why mixing code built by two different C++ standard libraries (`libstdc++` and LLVM's `libc++`) in one program does not work: their `std::string` and `std::vector` have different layouts.

### Mastery check

1. **Explain.** Why is `v.at(3)` different from `v[3]` even without `_GLIBCXX_ASSERTIONS`? (Find out: what does `at` do on a bad index?)

## Unit 0.3 — The build stages and the One Definition Rule

**One-line summary:** Like C, C++ preprocesses, compiles each **translation unit** separately, and links. The **One Definition Rule (ODR)** says every function and object used must be defined exactly once in the program — except that `inline` functions, templates, and class definitions may (and must) appear identically in every translation unit that uses them. Violations of the "identically" part are undefined behaviour the toolchain often cannot detect.

```cpp
// file: odr/shape.h
#pragma once
#include <string>

struct Shape {                       // a class definition may appear in many TUs
    std::string name;
    double area() const;             // declared here, defined once in shape.cpp
    double scaled(double k) const {  // defined in the class: implicitly inline
        return area() * k * k;
    }
};

inline int shape_api_version() { return 2; }   // inline: may be defined in every TU

template <typename T>
T twice(T x) { return x + x; }       // templates: the definition must be visible to every user
```

```cpp
// file: odr/shape.cpp
#include "shape.h"

double Shape::area() const { return 42.0; }
```

```cpp
// file: odr/main.cpp
#include <iostream>
#include "shape.h"

int main() {
    Shape s{"square"};
    std::cout << s.name << " area " << s.area() << ", scaled " << s.scaled(2)
              << ", api v" << shape_api_version() << ", twice(21) = " << twice(21) << '\n';
}
```

```console
$ cd odr && g++-14 -std=c++20 -Wall -c shape.cpp && g++-14 -std=c++20 -Wall -c main.cpp && g++-14 shape.o main.o -o app && ./app
square area 42, scaled 168, api v2, twice(21) = 42
$ cd odr && nm -C main.o | grep -E 'twice|api_version|area|scaled'
0000000000000000 W shape_api_version()
0000000000000000 W int twice<int>(int)
                 U Shape::area() const
0000000000000000 W Shape::scaled(double) const
```

`nm -C` (demangled) shows the rules at work in `main.o`: `Shape::area()` is `U` — used but defined elsewhere, exactly once, in `shape.o`. `twice<int>`, `shape_api_version`, and `Shape::scaled` are `W` (**weak**) symbols: every translation unit that uses them gets its own copy, and the linker keeps one and discards the rest. That is how the ODR's "may appear in every TU" is implemented.

### Why templates live in headers

A template is a recipe, not code. The compiler generates `twice<int>` only when it sees a use *and* the definition in the same translation unit. If `twice` were only declared in the header and defined in a `.cpp` file, `main.cpp` could not instantiate it, and linking would fail with an undefined reference to `twice<int>(int)` — one of the most common C++ build errors. (Explicit instantiation and C++20 modules offer alternatives; Units 0.5 and 7.6.)

### Deep & Tricky

**DT-2. Two different definitions of the same inline function or class across TUs: undefined behaviour, usually silent.** *(Guarantee: ODR violation, no diagnostic required)* If one `.cpp` file sees `struct Config { int a; };` and another (through a different header version or `#define`) sees `struct Config { int a, b; };`, both compile, the program links, and code in each file uses a different layout for the same type — memory corruption with no error. The linker keeps one copy of each `inline` function arbitrarily, so even "which version runs" is unpredictable. Keep one definition in one header; rebuild everything when headers change (Make dependencies, Chapter 11); `-flto` with `-Wodr` detects some cases.

### Mastery check

1. **Explain.** Why can a class definition appear in many translation units, but a non-inline function definition only in one?

## Unit 0.4 — Name mangling and `extern "C"`

**One-line summary:** C++ allows overloading (several functions with one name), namespaces, and templates, so the compiler encodes each function's full signature into its linker symbol name: **name mangling**. C does not mangle. To call C from C++ or to export C++ functions to C (and to other languages), declare them `extern "C"`.

```cpp
// file: mangle/overloads.cpp
#include <cstdio>

namespace geo {
int area(int side) { return side * side; }
double area(double w, double h) { return w * h; }
}

extern "C" int c_compatible(int x) { return x + 1; }    // plain C symbol name

int main() {
    std::printf("%d %.1f %d\n", geo::area(3), geo::area(2.0, 1.5), c_compatible(41));
}
```

```console
$ cd mangle && g++-14 -std=c++20 -c overloads.cpp && nm overloads.o | grep -E 'area|c_compatible' && echo --- && nm -C overloads.o | grep -E 'area|c_compatible'
0000000000000013 T _ZN3geo4areaEdd
0000000000000000 T _ZN3geo4areaEi
0000000000000031 T c_compatible
---
0000000000000013 T geo::area(double, double)
0000000000000000 T geo::area(int)
0000000000000031 T c_compatible
```

`_ZN3geo4areaEi` encodes "namespace `geo`, function `area`, parameter `int`"; `_ZN3geo4areaEdd` takes two `double`s. `c_compatible` keeps its plain name. Use `c++filt` or `nm -C` to demangle names in linker errors and crash reports.

### Calling C from C++

C headers meant for both languages wrap their declarations:

```c
#ifdef __cplusplus
extern "C" {
#endif
int legacy_checksum(const unsigned char *data, unsigned long n);
#ifdef __cplusplus
}
#endif
```

Without `extern "C"`, C++ code would look for a mangled `_Z15legacy_checksumPKhm` and the link would fail, because the C library defines plain `legacy_checksum`. All C standard headers (`<cstdio>`, `<cstring>`, …) already handle this.

<!-- expanded -->
### Going deeper: reading mangled names, and what may cross the `extern "C"` line

Mangled names look random but are a precise encoding of the namespace, name, and parameter types (the Itanium C++ ABI, used by GCC and Clang on every platform except Windows). `c++filt` turns them back into C++:

```cpp
// file: mangle2.cpp
#include <string>
#include <vector>

namespace geo {
struct Point { double x, y; };
double dist(const Point& a, const Point& b) { return (a.x - b.x) * (a.x - b.x) + (a.y - b.y) * (a.y - b.y); }
}
int count(const std::vector<std::string>& v) { return static_cast<int>(v.size()); }
extern "C" int plain_c_name(int x) { return x + 1; }
```

```console
$ g++-14 -std=c++20 -c mangle2.cpp -o mangle2.o && nm mangle2.o | grep " T " | awk '{print $3}' | sort > names.txt && c++filt < names.txt | paste -d'|' names.txt - | sed 's/|/   ->   /'
_Z5countRKSt6vectorINSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEESaIS5_EE   ->   count(std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > const&)
_ZN3geo4distERKNS_5PointES2_   ->   geo::dist(geo::Point const&, geo::Point const&)
plain_c_name   ->   plain_c_name
```

`_ZN3geo4distERKNS_5PointES2_` reads as: `_Z` (a mangled name), `N3geo4distE` (nested name `geo::dist`), `RK` (reference to const), `NS_5PointE` (`geo::Point`, abbreviated with `S_`), and `S2_` (the same type again). The parameter types are part of the symbol, which is how overloading survives separate compilation — and why a changed parameter type in a header without recompiling every user gives an "undefined reference" at link time instead of a silent mismatch. The `extern "C"` function keeps its plain name.

### Deep & Tricky

**DT-3. An exception must not propagate out of a function called from C code: C frames have no unwind information to run destructors with, so the result ranges from `std::terminate` to skipped cleanup — and the C++ standard leaves crossing language boundaries entirely to the implementation.** *(Implementation-defined: C++ [dcl.link] says nothing about exceptions through other languages; GCC documents that C code must be compiled with `-fexceptions` for exceptions to pass through it, which C code normally is not)* Every `extern "C"` function that will be called from C should be `noexcept` and catch everything inside: `try { ... } catch (...) { return -1; }`, translating exceptions into C-style error codes at the boundary, as this chapter's project does.

### Mastery check

1. **Explain.** Why can't `extern "C"` functions be overloaded?

## Unit 0.5 — Headers, modules, and build speed

**One-line summary:** `#include` pastes headers textually, so large C++ projects recompile the same standard-library headers thousands of times. **C++20 modules** (`export module`, `import`) are compiled once into a binary interface and imported, with no macro leakage. Compiler support is maturing; this book uses headers by default and shows modules for completeness.

```cpp
// file: modules/math.cppm
export module math;            // a named module

export int gcd(int a, int b) {
    while (b != 0) {
        int t = a % b;
        a = b;
        b = t;
    }
    return a;
}

int helper() { return 7; }     // not exported: invisible to importers
```

```cpp
// file: modules/main.cpp
import math;
#include <cstdio>

int main() {
    std::printf("gcd(84, 36) = %d\n", gcd(84, 36));
    // helper();  error: not visible
}
```

```console
$ cd modules && g++-14 -std=c++20 -fmodules-ts -c -x c++ math.cppm && g++-14 -std=c++20 -fmodules-ts main.cpp math.o -o app && ./app && ls gcm.cache
gcd(84, 36) = 12
math.gcm
```

The compiler wrote the module's compiled interface (`gcm.cache/math.gcm`) once; every importer reads it instead of re-parsing source. Build systems must compile modules in dependency order — CMake 3.28+ supports this for GCC 14 and Clang 17+. Until module support is universal in your toolchain, the classic header model (with `#pragma once` or include guards, and precompiled headers for speed) remains the norm.

### Measuring the header cost

```console
$ printf '#include <%s>\n' iostream vector string map algorithm ranges format > heavy.cpp && g++-14 -std=c++20 -E heavy.cpp | wc -l && /bin/bash -c 'TIMEFORMAT="compile time: %R s"; time g++-14 -std=c++20 -c heavy.cpp -o heavy.o'
85003
compile time: 0.631 s
```

Seven common includes expand to about 85,000 lines — most of a second of compile time before a single line of your own code — and the compiler must parse them again in every translation unit that includes them — the main reason C++ builds are slow, and the problem modules solve.

### Mastery check

1. **Explain.** What does a module stop that `#include` allows: list two things (hint: macros, and non-exported names).

## Chapter 0 Project — A two-language build: a C library used from C++

**Goal:** build a small C library and a C++ program that uses it — and a C++ function exported back to C — inspecting symbols, fixing the classic link errors, and using the professional flag set.

### Requirements

1. `crc.h`/`crc.c`: a C function `uint32_t crc32_compute(const void *data, size_t n)`, with a header usable from both C and C++.
2. `wrap.hpp`/`wrap.cpp`: a C++ wrapper `std::uint32_t crc32(std::string_view s)` and an `extern "C"` function `int cpp_word_count(const char *text)` callable from C.
3. A C++ `main` using the wrapper, and a C `main` using `cpp_word_count`.
4. Demonstrate the link error you get without `extern "C"`, and fix it.

### Reference solution

```c
// file: twolang/crc.h
#ifndef CRC_H
#define CRC_H
#include <stddef.h>
#include <stdint.h>

#ifdef __cplusplus
extern "C" {
#endif

/* crc32_compute returns the IEEE CRC-32 of n bytes. */
uint32_t crc32_compute(const void *data, size_t n);

#ifdef __cplusplus
}
#endif
#endif
```

```c
// file: twolang/crc.c
#include "crc.h"

uint32_t crc32_compute(const void *data, size_t n) {
    const unsigned char *p = data;
    uint32_t c = 0xFFFFFFFFu;
    for (size_t i = 0; i < n; i++) {
        c ^= p[i];
        for (int k = 0; k < 8; k++)
            c = (c & 1) ? 0xEDB88320u ^ (c >> 1) : c >> 1;
    }
    return c ^ 0xFFFFFFFFu;
}
```

```cpp
// file: twolang/wrap.hpp
#pragma once
#include <cstdint>
#include <string_view>

std::uint32_t crc32(std::string_view s);

extern "C" int cpp_word_count(const char *text);   // usable from C
```

```cpp
// file: twolang/wrap.cpp
#include "wrap.hpp"

#include <sstream>
#include <string>
#include "crc.h"

std::uint32_t crc32(std::string_view s) { return crc32_compute(s.data(), s.size()); }

extern "C" int cpp_word_count(const char *text) {
    std::istringstream in{text};
    std::string word;
    int n = 0;
    while (in >> word)
        ++n;
    return n;
}
```

```cpp
// file: twolang/main.cpp
#include <cstdio>
#include "wrap.hpp"

int main() {
    std::printf("crc32(\"123456789\") = %08X (expected CBF43926)\n", crc32("123456789"));
    std::printf("words: %d\n", cpp_word_count("the quick  brown\tfox"));
}
```

```c
// file: twolang/cmain.c
#include <stdio.h>

int cpp_word_count(const char *text);   /* implemented in C++, exported with extern "C" */

int main(void) {
    printf("C calling C++: %d words\n", cpp_word_count("one two three"));
    return 0;
}
```

```console
$ cd twolang && gcc -std=c17 -Wall -c crc.c && g++-14 -std=c++20 -Wall -Wextra -c wrap.cpp main.cpp && g++-14 crc.o wrap.o main.o -o app && ./app
crc32("123456789") = CBF43926 (expected CBF43926)
words: 4
$ cd twolang && gcc -std=c17 -Wall -c cmain.c && g++-14 crc.o wrap.o cmain.o -o capp && ./capp
C calling C++: 3 words
$ cd twolang && sed 's/extern "C" {//; s/^}$//' crc.h > crc_noextern.h && sed 's/crc.h/crc_noextern.h/' wrap.cpp > wrap_bad.cpp && g++-14 -std=c++20 -c wrap_bad.cpp && g++-14 crc.o wrap_bad.o main.o -o bad 2>&1 | grep -E 'undefined reference' | head -n 1
wrap_bad.cpp:(.text+0x48): undefined reference to `crc32_compute(void const*, unsigned long)'
```

The C++ program called the C CRC (matching the standard check value `CBF43926`), and the C program called a C++ function that uses `std::istringstream` internally. Notice that the C program was **linked with `g++`**: the C++ code needs the C++ standard library (`libstdc++`) and runtime, which `g++` adds automatically. Without `extern "C"` in the header, C++ looked for the mangled `crc32_compute(void const*, unsigned long)` and the link failed.

### Review checklist

- Does every header shared with C use the `#ifdef __cplusplus` / `extern "C"` guard?
- Is the final link done by the C++ driver when any object is C++?
- Do the builds pass with `-Wall -Wextra -pedantic` and no warnings?

## Answers to the Chapter 0 prediction questions

There are no prediction questions in this chapter.

# Chapter 1 — Core Language: Types, Initialisation, and Functions

*Goal: write correct everyday C++ — with the right types, initialisation that cannot go wrong, references, compile-time constants, lambdas, strings, and formatted output.*

C++ shares C's fundamental types and operators (if you need the details of integer promotion, overflow, and floating point, *C from Zero to Systems* Chapter 1 covers them, and they are identical in C++). This chapter covers what C++ adds or changes in the core: type deduction with `auto`, the several forms of initialisation (and why braces are safest), references, `const` and `constexpr`, scoped enums and namespaces, overloading and lambdas, the standard string and array types, structured bindings, and modern formatted output.

**What you will be able to do after this chapter**

- Choose the right initialisation syntax and explain what each form does.
- Use references, `const`, `constexpr`, and `consteval` correctly.
- Use `enum class`, namespaces, overloading, default arguments, and lambdas.
- Use `std::string`, `std::string_view`, `std::vector`, `std::array`, and `std::span` — and avoid `string_view`'s lifetime trap.
- Use range-`for`, `if` with initialisers, and structured bindings.
- Format output with `std::format` and `std::print`.

## Topic 1A — Values and types

## Unit 1.1 — Types, `auto`, and initialisation

**One-line summary:** C++ has several initialisation syntaxes. **Brace initialisation** (`T x{…}`) works for every type, never narrows silently, and never parses as a function declaration — prefer it. `auto` deduces a variable's type from its initialiser (dropping references and top-level `const`), and an uninitialised local of a built-in type still holds garbage, exactly as in C.

```cpp
// file: init.cpp
#include <iostream>
#include <string>
#include <vector>

struct Point { int x, y; };

int main() {
    int a = 7;                 // copy-initialisation
    int b(7);                  // direct-initialisation
    int c{7};                  // direct-list-initialisation (braces): preferred
    int d{};                   // value-initialisation: zero
    // int e{7.5};             // error: narrowing conversion from double to int
    int f = 7.9;               // allowed (truncates to 7), only a warning with -Wconversion

    std::vector<int> v1(3, 1); // three elements, each 1
    std::vector<int> v2{3, 1}; // two elements: 3 and 1 (initializer_list wins)
    Point p{1, 2};             // aggregate initialisation
    Point q{.x = 5, .y = 6};   // designated initialisers (C++20, must be in member order)

    auto i = 42;               // int
    auto s = std::string{"hi"};// std::string
    auto x = 1.5f;             // float
    const int& r = a;
    auto copy = r;             // int: auto drops the reference and top-level const

    std::cout << a << b << c << d << ' ' << f << '\n';
    std::cout << "v1 size " << v1.size() << ", v2 size " << v2.size() << '\n';
    std::cout << p.x << p.y << q.x << q.y << ' ' << i << s << x << copy << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra init.cpp -o init && ./init
7770 7
v1 size 3, v2 size 2
1256 42hi1.57
```

### The pitfalls braces avoid

- **Narrowing:** `int e{7.5};` and `char c{300};` are compile errors; `int e = 7.5;` silently truncates.
- **The most vexing parse:** `Widget w();` declares a *function* named `w` returning `Widget`, not an object. `Widget w{};` is always an object.
- **Uninitialised variables:** `int n;` inside a function is indeterminate; `int n{};` is zero.

The one surprise of braces: a type with an `std::initializer_list` constructor (like `std::vector`) prefers it, so `std::vector<int>{3, 1}` holds the two values 3 and 1, while `std::vector<int>(3, 1)` holds three ones. Use parentheses when you mean "size and value".

### `auto` rules

`auto` deduces like a template parameter (Chapter 7): references and top-level `const` are dropped. Write `const auto&` to bind without copying, `auto&` to modify in place, `auto&&` in generic code. Use `auto` when the type is obvious from the right-hand side or unwieldy to write (iterators, lambdas); spell the type out when it documents intent (`std::uint32_t crc = …`).

### Deep & Tricky

**DT-4. `auto x{1, 2};` is an error, but `auto x = {1, 2};` is an `std::initializer_list<int>`.** *(Guarantee: C++17 rules)* Copy-list-initialisation with `auto` deduces `initializer_list`, a surprising type with reference semantics to a temporary array. Direct-list-initialisation (`auto x{1};`) deduces `int` and allows only one element. Avoid `auto` with braces.

### Mastery check

1. **Predict.** What are the sizes of `std::vector<int>(5)`, `std::vector<int>{5}`, `std::vector<std::string>{5}`? (The last one is a trick — `5` cannot be a string.)
2. **Explain.** Why is `Widget w();` not an object?

## Unit 1.2 — References

**One-line summary:** A **reference** is another name for an existing object: it must be initialised, cannot be re-seated, and cannot be null. Use `const T&` to pass large objects without copying, `T&` for parameters the function modifies. A `const` reference can bind to a temporary and extends its lifetime — but only in that one direct binding.

```cpp
// file: refs.cpp
#include <iostream>
#include <string>

void increment(int& n) { ++n; }                          // modifies the caller's object
std::size_t length(const std::string& s) { return s.size(); }   // no copy, no modification

const std::string& longer(const std::string& a, const std::string& b) {
    return a.size() >= b.size() ? a : b;                  // returns a reference to an argument
}

int main() {
    int x = 1;
    int& alias = x;            // alias is x
    alias = 5;
    increment(x);
    std::cout << "x = " << x << '\n';

    std::cout << "length = " << length("temporary string") << '\n';   // binds to a temporary

    const std::string& lifetime_extended = std::string("I live as long as the reference");
    std::cout << lifetime_extended << '\n';

    std::string s1 = "short", s2 = "much longer";
    std::cout << "longer: " << longer(s1, s2) << '\n';                // fine: s1, s2 outlive the call
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra refs.cpp -o refs && ./refs
x = 6
length = 16
I live as long as the reference
longer: much longer
```

### References versus pointers

| | Reference `T&` | Pointer `T*` |
| --- | --- | --- |
| Can be null | no | yes |
| Can be re-pointed | no | yes |
| Must be initialised | yes | no (but should be) |
| Syntax | used like the object | `*p`, `p->m` |
| Use for | parameters, return values referring to existing objects | optional relationships, arrays, ownership via smart pointers (Chapter 5) |

### Deep & Tricky

**DT-5. Lifetime extension applies only to a reference bound *directly* to a temporary.** *(Guarantee: C++ standard)* `const auto& s = std::string("x");` extends the temporary's life to the reference's scope. But `const auto& r = longer(std::string("a"), std::string("bb"));` binds `r` to a reference returned by a function — the temporaries die at the end of the full expression, and `r` dangles. GCC 13+ warns (`-Wdangling-reference`) in many such cases. Returning references to parameters is safe only when the caller passes objects that outlive the use.

### Mastery check

1. **Predict.** Does `const std::string& r = longer("a", "bb"); std::cout << r;` have defined behaviour? Why?

## Unit 1.3 — `const`, `constexpr`, `consteval`, `constinit`

**One-line summary:** `const` means "not modified through this name". `constexpr` means "usable in constant expressions": a `constexpr` variable is a compile-time constant, and a `constexpr` function *can* run at compile time when its arguments are constants. `consteval` functions *must* run at compile time; `constinit` guarantees a static variable is initialised at compile time.

```cpp
// file: constexpr.cpp
#include <array>
#include <iostream>

constexpr unsigned long long factorial(int n) {
    unsigned long long r = 1;
    for (int i = 2; i <= n; ++i)
        r *= i;
    return r;
}

consteval int square(int x) { return x * x; }       // compile time only

constexpr std::array<int, 10> make_squares() {
    std::array<int, 10> a{};
    for (int i = 0; i < 10; ++i)
        a[i] = i * i;
    return a;
}

constinit int global_counter = square(4);          // initialised at compile time, still mutable

int main() {
    constexpr auto f20 = factorial(20);             // computed by the compiler
    static_assert(f20 == 2432902008176640000ULL);   // checked at compile time
    constexpr auto squares = make_squares();        // a whole table at compile time
    static_assert(squares[7] == 49);

    int n;
    std::cin >> n;
    std::cout << "factorial(" << n << ") at run time = " << factorial(n) << '\n';
    // square(n);   error: n is not a constant expression
    std::cout << "square(12) = " << square(12) << ", global_counter = " << ++global_counter << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra constexpr.cpp -o constexpr && echo 10 | ./constexpr
factorial(10) at run time = 3628800
square(12) = 144, global_counter = 17
```

The same `factorial` ran at compile time (for `f20`, checked by `static_assert`) and at run time (for input `n`). Compile-time evaluation catches undefined behaviour: a `constexpr` computation that overflowed a signed integer or read out of bounds would be a compile error, not silent UB — a useful way to test pure functions.

### `const` member functions and `mutable`

A member function marked `const` promises not to modify the object (Chapter 2). `const` objects can only call `const` member functions. Mark every member function that doesn't modify the object `const`, as you would mark pointer parameters `const` in C.

<!-- expanded -->
### Going deeper: when does a `constexpr` function run at compile time?

`constexpr` on a function means *may* be evaluated at compile time, not *will*. It is evaluated at compile time only where the language requires a constant — an array bound, a template argument, a `static_assert`, a `constexpr` variable — and otherwise it is an ordinary function call. `consteval` makes compile-time evaluation mandatory: calling it with a run-time value is an error. `constinit` (C++20) does something different again: it requires that a variable with static storage is initialised at compile time, without making it `const`.

```cpp
// file: when_ct.cpp
#include <iostream>

constexpr long fib(int n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
consteval long fib_ct(int n) { return fib(n); }

constinit long counter = fib(10);          // initialised at compile time, but may change later

int main(int argc, char**) {
    constexpr long a = fib(30);            // a constant is required: computed by the compiler
    long b = fib(argc + 29);               // depends on run-time input: computed when the program runs
    static_assert(fib(10) == 55);
    long c = fib_ct(30);                   // must be computed at compile time
    counter += 1;
    std::cout << a << ' ' << b << ' ' << c << ' ' << counter << '\n';
    // long d = fib_ct(argc);              // error: argc is not a constant expression
}
```

```console
$ g++-14 -std=c++20 -O0 -Wall -Wextra when_ct.cpp -o when_ct && ./when_ct && objdump -d --no-show-raw-insn when_ct | grep -c "call.*fib"
832040 832040 832040 56
3
```

At `-O0` the disassembly still contains calls to `fib` — the one for `b`, plus the recursive calls inside `fib` itself — but none for `a` or `c`: those values were computed by the compiler and appear in the code as the constant 832040.

### Deep & Tricky

**DT-6. The *static initialisation order fiasco*: global objects in different translation units are initialised in an unspecified order, so a global whose constructor uses another file's global may see it before it has been constructed. `constinit` turns the bug into a compile error by insisting that the initialiser is a constant expression — run before any dynamic initialisation at all.** *(Guarantee: [basic.start.dynamic] — dynamic initialisation of non-local variables in different translation units is "indeterminately sequenced"; [dcl.constinit])* The program below links the same two files in two orders and gets two different answers.

```cpp
// file: fiasco/registry.cpp
#include <cstdlib>
#include <string>
static std::string make_name() {                           // cannot run at compile time: reads the environment
    const char* e = std::getenv("SERVICE_NAME");
    return e ? e : "service";
}
std::string default_name = make_name();                    // dynamic initialisation
```

```cpp
// file: fiasco/main.cpp
#include <iostream>
#include <string>
extern std::string default_name;
std::string greeting = "hello, " + default_name;          // uses another file's global during start-up
int main() { std::cout << "[" << greeting << "]\n"; }
```

```console
$ cd fiasco && g++-14 -std=c++20 registry.cpp main.cpp -o first_registry && ./first_registry && g++-14 -std=c++20 main.cpp registry.cpp -o first_main && ./first_main
[hello, service]
[hello, ]
```

Linked with `registry.cpp` first, `default_name` was ready; linked the other way round, `greeting` read an empty, not-yet-constructed `std::string` (technically undefined behaviour — it happened to read zero bytes). The usual cure is a function that returns a reference to a local `static` ("construct on first use"), which is initialised exactly once, the first time it is needed, and thread-safely since C++11.

### Mastery check

1. **Explain.** What is the difference between `const int n = f();` and `constexpr int n = f();`?

## Topic 1B — Organising code

## Unit 1.4 — `enum class`, namespaces, and type aliases

**One-line summary:** `enum class` creates **scoped, strongly typed** enumerations that do not convert to `int` implicitly and do not leak their names into the enclosing scope. **Namespaces** group names and prevent collisions. `using` creates type aliases (the modern `typedef`).

```cpp
// file: enums.cpp
#include <cstdint>
#include <iostream>
#include <string_view>

enum class Color : std::uint8_t { Red, Green, Blue };   // scoped, 1 byte
enum class Light { Red, Yellow, Green };                // no clash with Color::Red

namespace app::net {                                   // nested namespace (C++17)
    using Port = std::uint16_t;                         // type alias
    constexpr Port default_port = 8080;
}

std::string_view to_string(Color c) {
    switch (c) {                                        // -Wswitch warns about missing cases
    case Color::Red: return "red";
    case Color::Green: return "green";
    case Color::Blue: return "blue";
    }
    return "?";
}

int main() {
    Color c = Color::Green;
    // int i = c;                        error: no implicit conversion
    int i = static_cast<int>(c);         // explicit
    std::cout << to_string(c) << " = " << i << ", sizeof(Color) = " << sizeof(Color) << '\n';
    std::cout << "port " << app::net::default_port << '\n';

    using enum Light;                    // C++20: bring the enumerators into scope
    Light l = Yellow;
    std::cout << "light " << static_cast<int>(l) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra enums.cpp -o enums && ./enums
green = 1, sizeof(Color) = 1
port 8080
light 1
```

Rules for namespaces: never write `using namespace std;` in a header (it forces every name in `std` on every includer); in `.cpp` files, prefer specific `using std::string;` declarations or just write `std::`. Names in an unnamed namespace (`namespace { … }`) have internal linkage — the C++ equivalent of `static` functions in C.

<!-- expanded -->
### Going deeper: argument-dependent lookup

When an unqualified function name is called, C++ looks for it not only in the scopes that are visible, but also in the namespaces of the *arguments'* types. This **argument-dependent lookup** (ADL, or Koenig lookup) is what lets you write `std::cout << x` without `std::operator<<`, and `swap(a, b)` that finds a type's own `swap`. It is also why a call can silently pick a function you never meant to call.

```cpp
// file: adl.cpp
#include <iostream>

namespace shapes {
struct Circle { double r; };
void print(const Circle& c) { std::cout << "shapes::print: circle of radius " << c.r << '\n'; }
}

namespace app {
void print(double) { std::cout << "app::print(double)\n"; }
void draw() {
    shapes::Circle c{2.0};
    print(c);         // not visible in app ... found through the argument's namespace
    print(3.5);       // ordinary lookup: app::print
}
}

namespace util {
template <typename T> void log(const T&) { std::cout << "util::log (generic)\n"; }
}
namespace shapes {
void log(const Circle&) { std::cout << "shapes::log (via ADL!)\n"; }
}

int main() {
    app::draw();
    shapes::Circle c{1.0};
    using util::log;
    log(c);            // both util::log<Circle> and shapes::log are candidates; the non-template wins
    util::log(c);      // qualified name: no ADL
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra adl.cpp -o adl && ./adl
shapes::print: circle of radius 2
app::print(double)
shapes::log (via ADL!)
util::log (generic)
```

Qualifying a call (`util::log(c)`) turns ADL off. The standard library relies on ADL deliberately in one idiom worth knowing: `using std::swap; swap(a, b);` calls a type's own `swap` if its namespace has one, and `std::swap` otherwise. C++20 adds `using enum` (bring an `enum class`'s names into scope, for example inside a `switch`), and `inline namespace` (used by libraries to version their ABI, as libstdc++ does with `std::__cxx11`).

### Mastery check

1. **Explain.** Why does `enum class` prevent bugs that plain `enum` allows? Give an example comparison that compiles with plain enums and should not.

## Unit 1.5 — Functions, overloading, and lambdas

**One-line summary:** C++ functions can be **overloaded** (same name, different parameter types — the compiler picks the best match), have **default arguments**, and be written inline as **lambdas**: unnamed function objects that can **capture** variables from their scope by value or by reference.

```cpp
// file: lambdas.cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

void print(int x) { std::cout << "int " << x << '\n'; }
void print(double x) { std::cout << "double " << x << '\n'; }
void print(const std::string& s, int times = 1) {          // default argument
    for (int i = 0; i < times; ++i) std::cout << "string " << s << '\n';
}

int main() {
    print(3);
    print(3.5);
    print(std::string("hi"), 2);
    // print(3L);   error: ambiguous (long -> int and long -> double are equally good)

    std::vector<int> v{5, 3, 9, 1, 7};
    int threshold = 4;
    auto count = std::count_if(v.begin(), v.end(), [threshold](int x) { return x > threshold; });
    std::cout << count << " values above " << threshold << '\n';

    int calls = 0;
    std::sort(v.begin(), v.end(), [&calls](int a, int b) { ++calls; return a > b; });  // by reference
    std::cout << "sorted descending with " << calls << " comparisons:";
    for (int x : v) std::cout << ' ' << x;
    std::cout << '\n';

    auto make_counter = [] {
        int n = 0;
        return [n]() mutable { return ++n; };          // owns its own copy of n
    };
    auto c1 = make_counter(), c2 = make_counter();
    std::cout << c1() << c1() << c1() << ' ' << c2() << '\n';

    auto add = [](auto a, auto b) { return a + b; };   // generic lambda
    std::cout << add(1, 2) << ' ' << add(std::string("a"), "b") << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra lambdas.cpp -o lambdas && ./lambdas
int 3
double 3.5
string hi
string hi
3 values above 4
sorted descending with 10 comparisons: 9 7 5 3 1
123 1
3 ab
```

### Captures

| Capture | Meaning |
| --- | --- |
| `[]` | nothing |
| `[x]`, `[=]` | copy of `x` / copies of everything used |
| `[&x]`, `[&]` | reference to `x` / references to everything used |
| `[this]`, `[*this]` | the enclosing object by pointer / by copy |
| `[y = std::move(x)]` | init-capture: a new member initialised by any expression |
| `mutable` | allows modifying by-value captures |

A lambda is an object of a unique class type with an `operator()`; captures are its data members. That is why lambdas compose with algorithms at zero cost: the compiler sees the whole function and inlines it.

### Deep & Tricky

**DT-7. A lambda that captures by reference must not outlive what it captured.** *(Guarantee: UB on dangling reference)* Returning `[&n] { return ++n; }` from a function where `n` is local creates a lambda holding a reference to a dead variable. `[&]` in a lambda that is stored, returned, or run on another thread is a common source of crashes. Capture by value (or `std::move` into an init-capture) for anything that escapes the scope.

### Mastery check

1. **Predict.** In `lambdas.cpp`, why does `c2()` print 1 rather than 4?
2. **Write.** A function `std::function<int()> make_accumulator(int start)` returning a lambda that adds 1 to its own total each call; test two independent accumulators.

## Topic 1C — Standard types and syntax you use every day

## Unit 1.6 — Strings, views, and arrays

**One-line summary:** `std::string` owns a growable sequence of characters; `std::string_view` is a **non-owning** view of characters stored elsewhere (cheap to pass, dangerous to store); `std::vector<T>` is the default container; `std::array<T, N>` is a fixed-size array with value semantics; `std::span<T>` is a non-owning view of contiguous elements of any of them.

```cpp
// file: strings.cpp
#include <array>
#include <iostream>
#include <span>
#include <string>
#include <string_view>
#include <vector>

std::string_view first_word(std::string_view s) {          // accepts literals, strings, substrings
    auto end = s.find(' ');
    return s.substr(0, end);                                // no allocation: a view into s
}

int sum(std::span<const int> values) {                      // any contiguous ints
    int total = 0;
    for (int v : values) total += v;
    return total;
}

int main() {
    std::string s = "hello";
    s += ", world";
    s.insert(0, ">> ");
    std::cout << s << " (size " << s.size() << ")\n";
    std::cout << "find world at " << s.find("world") << ", substr: " << s.substr(3, 5) << '\n';
    std::cout << "starts_with \">>\": " << s.starts_with(">>") << '\n';        // C++20

    std::cout << "first word: " << first_word("alpha beta gamma") << '\n';

    std::array<int, 4> arr{1, 2, 3, 4};
    std::vector<int> vec{10, 20, 30};
    int c_array[] = {100, 200};
    std::cout << sum(arr) << ' ' << sum(vec) << ' ' << sum(c_array) << '\n';
    std::cout << "number as text: " << std::to_string(3.25) << ", parsed: " << std::stoi("42") + 1 << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra strings.cpp -o strings && ./strings
>> hello, world (size 15)
find world at 10, substr: hello
starts_with ">>": 1
first word: alpha
10 60 300
number as text: 3.250000, parsed: 43
```

### The `string_view` lifetime trap

```cpp
// file: view_trap.cpp
#include <iostream>
#include <string>
#include <string_view>

std::string make_name() { return "a temporary string long enough to be on the heap"; }

int main() {
    std::string_view v = make_name();   // the temporary std::string dies at the end of this line
    std::cout << v << '\n';             // reads freed memory
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -fsanitize=address view_trap.cpp -o view_trap && ./view_trap 2>&1 | grep -E 'ERROR|#[0-9] .*main' | head -n 2
==6475==ERROR: AddressSanitizer: heap-use-after-free on address 0x506000000020 at pc 0x7f08cc87f05f bp 0x7ffe9eb9e5c0 sp 0x7ffe9eb9dd68
    #3 0x5653044666e0 in main view_trap.cpp:10
```

AddressSanitizer reports a heap-use-after-free: `v` points into a string that was destroyed at the semicolon. (Short strings are stored inside the `std::string` object itself — the small-string optimisation — so this bug can appear only for longer strings, making it even harder to spot.) Rules: use `string_view` for **parameters** and short-lived local views; never return a `string_view` to a local or temporary string; store `std::string` in data structures.

### Deep & Tricky

**DT-8. `std::string_view` is not null-terminated.** *(Library behaviour)* A view of `"hello world"` from 0 to 5 is `"hello"` with no `'\0'` after it. Passing `v.data()` to a C function that expects a C string reads past the view. Convert with `std::string{v}` (or keep a `std::string`) when calling C APIs.

<!-- expanded -->
### Going deeper: the small-string optimisation

A `std::string` object is 32 bytes in libstdc++ (24 in libc++). Short strings are stored *inside* the object — no heap allocation at all — and only longer strings get a heap buffer. This **small-string optimisation** (SSO) makes most strings in real programs (names, keys, short messages) cheap to create and copy.

```cpp
// file: sso.cpp
#include <cstdio>
#include <cstdlib>
#include <new>
#include <string>

static int allocations = 0;
void* operator new(std::size_t n) {                        // count every heap allocation in the program
    ++allocations;
    if (void* p = std::malloc(n)) return p;
    throw std::bad_alloc();
}
void operator delete(void* p) noexcept { std::free(p); }
void operator delete(void* p, std::size_t) noexcept { std::free(p); }

int main() {
    std::printf("sizeof(std::string) = %zu\n", sizeof(std::string));
    for (std::size_t len : {0u, 15u, 16u, 100u}) {
        int before = allocations;
        std::string s(len, 'x');
        const void* inside = &s;
        bool in_object = static_cast<const void*>(s.data()) >= inside &&
                         static_cast<const void*>(s.data()) < static_cast<const char*>(inside) + sizeof s;
        std::printf("length %3zu: capacity %3zu, heap allocations %d, characters stored %s\n", len, s.capacity(),
                    allocations - before, in_object ? "inside the string object" : "on the heap");
    }
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra sso.cpp -o sso && ./sso
sizeof(std::string) = 32
length   0: capacity  15, heap allocations 0, characters stored inside the string object
length  15: capacity  15, heap allocations 0, characters stored inside the string object
length  16: capacity  16, heap allocations 1, characters stored on the heap
length 100: capacity 100, heap allocations 1, characters stored on the heap
```

Up to 15 characters (plus the terminating `'\0'`) fit inside the object; the 16th forces an allocation. A consequence that surprises people: moving a short string copies its characters (there is no heap pointer to steal), and `std::string_view`s into a short string point into the string *object* — so they dangle as soon as the object moves, even though no heap memory was freed.

### Deep & Tricky

**DT-9. `"abc" + 1` and `'a' + "bc"` compile — as pointer arithmetic, not string concatenation — because string literals are arrays of `char`, not `std::string`s. `std::string s = 'a' + "bc";` adds 97 to the address of `"bc"` and reads far past it.** *(Guarantee: [expr.add], a literal decays to `const char*`; the result is undefined because it points outside the array)* Concatenation needs at least one `std::string` operand: `std::string("a") + "bc"`, or `"a"s + "bc"` with `using namespace std::string_literals;`. GCC warns with `-Wall` for the obvious cases ("adding 'char' to a string does not append to the string" from Clang; GCC's `-Warray-bounds` at `-O2`); compile both ways.

```cpp
// file: plus.cpp
#include <iostream>
#include <string>
using namespace std::string_literals;

int main() {
    std::string ok = "a"s + "bc";
    std::cout << "\"a\"s + \"bc\" = " << ok << '\n';
    const char* p = "hello, world" + 7;              // legal: a pointer to "world"
    std::cout << "\"hello, world\" + 7 = " << p << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra plus.cpp -o plus && ./plus
"a"s + "bc" = abc
"hello, world" + 7 = world
$ printf '#include <string>\nstd::string f() { return %s; }\n' "'a' + \"bc\"" > charplus.cpp && clang++ -std=c++20 -Wall -c charplus.cpp 2>&1 | grep -E "warning" | sed 's/^charplus.cpp:[0-9:]* //'
warning: adding 'char' to a string does not append to the string [-Wstring-plus-int]
note: use array indexing to silence this warning
warning: adding 'char' to a string pointer does not append to the string [-Wstring-plus-char]
note: use array indexing to silence this warning
2 warnings generated.
```

### Mastery check

1. **Explain.** When should a function take `std::string_view`, `const std::string&`, or `std::string` (by value)?

## Unit 1.7 — Modern control flow: range-`for`, initialisers, structured bindings

**One-line summary:** Range-`for` iterates any container; `if` and `switch` can declare a variable scoped to the statement; **structured bindings** unpack structs, pairs, tuples, and arrays into named variables.

```cpp
// file: modern.cpp
#include <iostream>
#include <map>
#include <string>
#include <tuple>

struct Stats { double mean; double max; int count; };

Stats summarise(const int* data, int n) {
    double sum = 0, max = data[0];
    for (int i = 0; i < n; ++i) {
        sum += data[i];
        if (data[i] > max) max = data[i];
    }
    return {sum / n, max, n};
}

std::tuple<int, int> divmod(int a, int b) { return {a / b, a % b}; }

int main() {
    int data[] = {4, 8, 15, 16, 23, 42};
    auto [mean, max, count] = summarise(data, 6);              // structured binding
    std::cout << "mean " << mean << ", max " << max << ", n " << count << '\n';

    auto [q, r] = divmod(17, 5);
    std::cout << "17 = 5*" << q << " + " << r << '\n';

    std::map<std::string, int> ages{{"ada", 36}, {"alan", 41}};
    for (const auto& [name, age] : ages)                       // bindings in range-for
        std::cout << name << " is " << age << '\n';

    if (auto it = ages.find("grace"); it == ages.end())        // if with initialiser
        std::cout << "grace not found\n";
    else
        std::cout << "grace is " << it->second << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra modern.cpp -o modern && ./modern
mean 18, max 42, n 6
17 = 5*3 + 2
ada is 36
alan is 41
grace not found
```

The `if (init; condition)` form keeps `it` scoped to the `if`/`else` — no leftover variable after it. Range-`for` works with anything that has `begin()` and `end()`.

<!-- expanded -->
### Deep & Tricky

**DT-10. `auto [a, b] = expr;` does not bind `a` and `b` to the members of `expr`: it makes one hidden *copy* of the whole object and binds the names to that copy's members. Modifying `a` changes nothing outside; to modify the original, write `auto& [a, b]`, and to avoid the copy while only reading, `const auto& [a, b]`.** *(Guarantee: [dcl.struct.bind] — an unnamed variable `e` is introduced, initialised from the initialiser, and the names refer to its members)* The `auto` before the brackets applies to the hidden object, not to each name — which is also why the names cannot be declared with different types or qualifiers.

```cpp
// file: bindcopy.cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> stock{{"apples", 3}, {"pears", 5}};
    for (auto [name, count] : stock) count += 10;          // modifies copies
    std::cout << "after auto:  apples " << stock["apples"] << '\n';
    for (auto& [name, count] : stock) count += 10;         // modifies the map's values
    std::cout << "after auto&: apples " << stock["apples"] << '\n';
    // name is const std::string here either way: map keys cannot be changed in place
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra bindcopy.cpp -o bindcopy && ./bindcopy
after auto:  apples 3
after auto&: apples 13
```

### Mastery check

1. **Write.** A function returning `std::pair<std::string, std::string>` that splits `"key=value"` at the first `=`; use it with structured bindings.

## Unit 1.8 — Formatted output: iostreams, `std::format`, `std::print`

**One-line summary:** iostreams (`std::cout <<`) are type-safe but verbose and stateful; C++20's **`std::format`** brings Python-like format strings with compile-time checking; C++23's **`std::print`/`std::println`** write formatted output directly. Prefer `std::format`/`std::print` in new code.

```cpp
// file: format.cpp
#include <format>
#include <iomanip>
#include <iostream>
#include <print>
#include <string>

struct Point { int x, y; };

template <>
struct std::formatter<Point> : std::formatter<std::string> {     // make Point formattable
    auto format(const Point& p, std::format_context& ctx) const {
        return std::formatter<std::string>::format(std::format("({}, {})", p.x, p.y), ctx);
    }
};

int main() {
    double pi = 3.14159265;
    std::cout << "iostream: " << std::fixed << std::setprecision(2) << pi << " [" << std::setw(6) << 42 << "]\n";

    std::string s = std::format("format: {:.3f} [{:>6}] [{:<6}] [{:^6}] {:#x} {:08b}", pi, 42, 42, 42, 255, 5);
    std::cout << s << '\n';

    std::println("print (C++23): {} + {} = {}", 2, 3, 2 + 3);
    std::println("{0} {1} {0}", "positional", "args");
    std::println("custom type: {}", Point{3, 4});
    // std::println("{:d}", "text");   compile error: format string checked at compile time
}
```

```console
$ g++-14 -std=c++23 -Wall -Wextra format.cpp -o format && ./format
iostream: 3.14 [    42]
format: 3.142 [    42] [42    ] [  42  ] 0xff 00000101
print (C++23): 2 + 3 = 5
positional args positional
custom type: (3, 4)
```

The format string is checked at compile time against the argument types (`std::format_string`), so the `printf` class of bugs (Unit 6.2 of the C book) cannot happen. C++23 also lets you format whole ranges (`std::println("{}", vec)` prints `[1, 2, 3]`), but libstdc++ added that only in GCC 15: the first version of this program used it and failed to compile with GCC 14 with several screens of template errors whose key line was "std::formatter must be specialized for each type being formatted". When a new library feature fails to compile, check your standard library's support tables (cppreference lists them per feature) before suspecting your code. `std::format` returns a `std::string`; `std::format_to` writes to any output iterator without allocating a temporary.

<!-- expanded -->
### Going deeper: format strings are checked at compile time

`std::format` and `std::print` parse the format string *while compiling* when it is a literal: a wrong placeholder or a type that cannot be formatted with the given specification is a compile error, not a run-time surprise as with `printf`. The price is that the format string must be a constant expression; a format string read from a file or a translation catalogue must go through `std::vformat` and `std::make_format_args`, which check at run time and throw `std::format_error`.

```cpp
// file: fmtcheck.cpp
#include <format>
#include <iostream>
#include <string>

int main() {
    std::cout << std::format("{:>8.2f}|{:<6}|{:#x}\n", 3.14159, "ab", 255);
    std::string runtime_fmt = "{} items, {:.1f}% done\n";             // e.g. read from a config file
    int items = 12; double pct = 37.5;                                 // make_format_args takes lvalues
    std::cout << std::vformat(runtime_fmt, std::make_format_args(items, pct));
    try {
        std::string bad = "{:d}\n";                                    // 'd' is not valid for a string
        std::string arg = "text";
        std::cout << std::vformat(bad, std::make_format_args(arg));
    } catch (const std::format_error& e) {
        std::cout << "format_error: " << e.what() << '\n';
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra fmtcheck.cpp -o fmtcheck && ./fmtcheck
    3.14|ab    |0xff
12 items, 37.5% done
format_error: format error: failed to parse format-spec
$ printf '#include <format>\n#include <string>\nstd::string f() { return std::format("{:d}", "text"); }\n' > fmtbad.cpp && g++-14 -std=c++20 -c fmtbad.cpp 2>&1 | grep -m1 -oE "error: [^']*" 
error: call to consteval function ‘std::basic_format_string<char, const char (&)[5]>("{:d}")’ is not a constant expression
```

The same mistake — `{:d}` for a string — is a compile-time error with a literal format string and a run-time exception with a run-time one. `printf` with a literal format gets only a *warning* for such mistakes, and none at all with a run-time format.

### Mastery check

1. **Write.** Print a table of names and scores with right-aligned scores to one decimal place using `std::println`.

## Chapter 1 Project — A grade report from CSV

**Goal:** read lines of `name,score1,score2,...` from standard input, validate and parse them without exceptions or `atoi`, and print a formatted report — using `string_view` parsing, `std::from_chars`, structured bindings, lambdas, and `std::print`.

### Specification

- Each line: a name, then one or more integer scores 0–100, comma-separated. Blank lines are skipped.
- Malformed lines (missing scores, non-numbers, out-of-range values) are reported on standard error with their line number and skipped.
- Output: one row per student — name, number of scores, mean (1 decimal), best — sorted by mean descending, then a class mean.
- Exit status 0 if all lines were valid, 1 otherwise.

### Reference solution

```cpp
// file: grades/grades.cpp
#include <algorithm>
#include <charconv>
#include <iostream>
#include <optional>
#include <print>
#include <string>
#include <string_view>
#include <vector>

struct Student {
    std::string name;
    std::vector<int> scores;
    double mean() const {
        double s = 0;
        for (int x : scores) s += x;
        return s / static_cast<double>(scores.size());
    }
};

// parse_score converts a field to 0..100 with no allocation and no exceptions.
std::optional<int> parse_score(std::string_view field) {
    while (!field.empty() && field.front() == ' ') field.remove_prefix(1);
    while (!field.empty() && field.back() == ' ') field.remove_suffix(1);
    int value{};
    auto [ptr, ec] = std::from_chars(field.data(), field.data() + field.size(), value);
    if (ec != std::errc{} || ptr != field.data() + field.size() || value < 0 || value > 100)
        return std::nullopt;
    return value;
}

// parse_line returns a Student, or an error message.
std::optional<Student> parse_line(std::string_view line, std::string& error) {
    auto comma = line.find(',');
    if (comma == std::string_view::npos || comma == 0) {
        error = "expected name,score[,score...]";
        return std::nullopt;
    }
    Student st{std::string{line.substr(0, comma)}, {}};
    line.remove_prefix(comma + 1);
    while (true) {
        auto next = line.find(',');
        auto field = line.substr(0, next);
        auto score = parse_score(field);
        if (!score) {
            error = std::format("bad score '{}'", field);
            return std::nullopt;
        }
        st.scores.push_back(*score);
        if (next == std::string_view::npos) break;
        line.remove_prefix(next + 1);
    }
    return st;
}

int main() {
    std::vector<Student> students;
    std::string line, error;
    int lineno = 0, bad = 0;
    while (std::getline(std::cin, line)) {
        ++lineno;
        if (line.find_first_not_of(" \t\r") == std::string::npos) continue;
        if (auto st = parse_line(line, error))
            students.push_back(std::move(*st));
        else {
            std::println(stderr, "line {}: {}", lineno, error);
            ++bad;
        }
    }
    std::ranges::sort(students, [](const Student& a, const Student& b) {
        return a.mean() != b.mean() ? a.mean() > b.mean() : a.name < b.name;
    });
    std::println("{:<10} {:>6} {:>6} {:>5}", "name", "scores", "mean", "best");
    double total = 0;
    for (const auto& [name, scores] : students) {
        Student tmp{name, scores};
        total += tmp.mean();
        std::println("{:<10} {:>6} {:>6.1f} {:>5}", name, scores.size(), tmp.mean(), std::ranges::max(scores));
    }
    if (!students.empty())
        std::println("class mean: {:.1f} over {} students", total / static_cast<double>(students.size()), students.size());
    return bad == 0 ? 0 : 1;
}
```

```text
# file: grades/class.csv
ada,95,88,92
alan, 78 , 85
grace,100
bob,70,x,80
linus,101
dennis

margaret,90,91,89
```

```console
$ cd grades && g++-14 -std=c++23 -Wall -Wextra -pedantic -g -fsanitize=address,undefined grades.cpp -o grades && ./grades < class.csv; echo "exit status: $?"
line 1: expected name,score[,score...]
line 5: bad score 'x'
line 6: bad score '101'
line 7: expected name,score[,score...]
name       scores   mean  best
grace           1  100.0   100
ada             3   91.7    95
margaret        3   90.0    91
alan            2   81.5    85
class mean: 90.8 over 4 students
exit status: 1
```

Design notes: `std::from_chars` is the fastest standard conversion, never allocates, never throws, and reports exactly where parsing stopped — unlike `std::stoi`, which throws and accepts trailing garbage. `std::optional` represents "a value or nothing" without sentinel values (Chapter 6 compares it with `std::expected`, which also carries the error). The `# file:` comment line in the data is itself reported as malformed — a good reminder that parsers see everything.

### Review checklist

- Does any input crash the program or produce UB? (Try empty lines, trailing commas, huge numbers, a line of only commas, under the sanitizers.)
- Are all string views used only while the line they view is alive?
- Does the program avoid exceptions for expected input errors?

## Answers to the Chapter 1 prediction questions

**1.1 Q1.** `std::vector<int>(5)`: five zeros. `std::vector<int>{5}`: one element, 5. `std::vector<std::string>{5}`: `5` cannot initialise a string, so the `initializer_list` constructor is not viable and the compiler falls back to the size constructor — five empty strings.

**1.2 Q1.** No. The two `std::string` temporaries created from the literals die at the end of the full expression; `r` refers to one of them and dangles when it is used on the next line.

**1.5 Q1.** Each call to `make_counter` creates a new lambda with its own copy of `n`; `c1` and `c2` are independent objects.

# Chapter 2 — Classes and RAII

*Goal: design classes that keep their invariants, manage resources automatically through constructors and destructors, copy correctly, and behave like built-in types through operator overloading.*

The most important idea in C++ is not object orientation. It is **RAII** — *Resource Acquisition Is Initialisation*: a resource (memory, a file, a lock, a socket) is acquired in an object's constructor and released in its destructor, and the language guarantees that destructors run when objects go out of scope — on every path, including early returns and exceptions. RAII is why idiomatic C++ has no `free` calls, no `goto cleanup` ladders, and far fewer leaks than C. This chapter teaches classes as the vehicle for RAII and for **value types** that are as easy to use as `int`.

**What you will be able to do after this chapter**

- Write classes with private data, invariants, and `const`-correct member functions.
- Write constructors with member initialiser lists, and know initialisation order.
- Write RAII wrappers whose destructors release resources reliably.
- Implement correct copy semantics and know the Rule of Zero, Three, and Five.
- Overload operators, including C++20's defaulted three-way comparison.

## Topic 2A — Classes

## Unit 2.1 — Classes, access, and invariants

**One-line summary:** A class bundles data with the functions that operate on it. `private` data can only be changed through member functions, which lets the class maintain an **invariant** — a condition that is always true for every valid object. `struct` and `class` differ only in default access (public vs private).

```cpp
// file: account.cpp
#include <iostream>
#include <stdexcept>
#include <string>

class Account {
public:
    Account(std::string owner, long long cents) : owner_{std::move(owner)}, balance_{cents} {
        if (cents < 0)
            throw std::invalid_argument("opening balance must not be negative");
    }

    void deposit(long long cents) {
        if (cents <= 0) throw std::invalid_argument("deposit must be positive");
        balance_ += cents;
    }

    bool withdraw(long long cents) {        // invariant: balance never negative
        if (cents <= 0 || cents > balance_)
            return false;
        balance_ -= cents;
        return true;
    }

    long long balance() const { return balance_; }      // const: callable on const objects
    const std::string& owner() const { return owner_; }

private:
    std::string owner_;
    long long balance_;                     // in cents: never use floating point for money
};

int main() {
    Account a{"ada", 10'000};
    a.deposit(2'550);
    std::cout << a.owner() << " has " << a.balance() << " cents\n";
    std::cout << "withdraw 20000: " << (a.withdraw(20'000) ? "ok" : "refused") << '\n';
    const Account& view = a;
    std::cout << "via const reference: " << view.balance() << '\n';
    // view.deposit(1);   error: deposit is not const
    // a.balance_ = -5;   error: balance_ is private
    try {
        Account bad{"eve", -1};
    } catch (const std::invalid_argument& e) {
        std::cout << "construction failed: " << e.what() << '\n';
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra account.cpp -o account && ./account
ada has 12550 cents
withdraw 20000: refused
via const reference: 12550
construction failed: opening balance must not be negative
```

The class guarantees that no `Account` with a negative balance can exist: the constructor refuses to create one (by throwing — Chapter 6 covers exceptions), and the only ways to change the balance check the rule. That guarantee is what encapsulation buys. The trailing-underscore naming for members is a common convention (others use `m_balance`).

<!-- expanded -->
### Going deeper: `struct` versus `class`, and what an invariant costs

`struct` and `class` are the same thing in C++ except for the default access (`public` for `struct`, `private` for `class`). The convention used by most codebases and the C++ Core Guidelines: use `struct` for a plain bundle of data whose members can take any values independently, and `class` when there is an **invariant** — a rule linking the members that the class's functions maintain, and that outside code must not be able to break.

```cpp
// file: invariant.cpp
#include <iostream>
#include <stdexcept>

struct Point { double x, y; };                 // no invariant: any x and y are fine

class Range {                                  // invariant: lo <= hi, always
public:
    Range(int lo, int hi) : lo_(lo), hi_(hi) {
        if (lo > hi) throw std::invalid_argument("Range: lo > hi");
    }
    int lo() const { return lo_; }
    int hi() const { return hi_; }
    int width() const { return hi_ - lo_; }    // can rely on the invariant: never negative
    void shift(int d) { lo_ += d; hi_ += d; }  // every member function keeps the invariant
    void set_hi(int h) {
        if (h < lo_) throw std::invalid_argument("Range: hi < lo");
        hi_ = h;
    }
private:
    int lo_, hi_;
};

int main() {
    Point p{1.5, -2};
    p.x = 100;                                 // fine: Point promises nothing
    Range r(3, 10);
    r.shift(5);
    std::cout << "p.x " << p.x << ", range [" << r.lo() << ", " << r.hi() << "] width " << r.width() << '\n';
    try { r.set_hi(0); } catch (const std::exception& e) { std::cout << "rejected: " << e.what() << '\n'; }
    try { Range bad(5, 1); } catch (const std::exception& e) { std::cout << "rejected: " << e.what() << '\n'; }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra invariant.cpp -o invariant && ./invariant
p.x 100, range [8, 15] width 7
rejected: Range: hi < lo
rejected: Range: lo > hi
```

A getter/setter pair for every member (`get_lo`/`set_lo` that accept anything) protects nothing — it is a `struct` with extra typing. The value of `private` is in the functions you *do not* offer.

### Mastery check

1. **Explain.** Why does `balance()` need `const` for `view.balance()` to compile?

## Unit 2.2 — Constructors

**One-line summary:** Constructors initialise objects. Initialise members in the **member initialiser list** (`: a_{x}, b_{y}`), not by assignment in the body; members are initialised in the order they are **declared** in the class, not the order in the list. Mark single-argument constructors `explicit` to prevent unintended implicit conversions.

```cpp
// file: ctors.cpp
#include <iostream>
#include <string>

class Temperature {
public:
    Temperature() = default;                                   // uses the default member initialisers
    explicit Temperature(double celsius) : celsius_{celsius} {}
    static Temperature from_fahrenheit(double f) { return Temperature{(f - 32) * 5 / 9}; }  // named constructor
    double celsius() const { return celsius_; }
private:
    double celsius_ = 20.0;                                    // default member initialiser
};

void report(Temperature t) { std::cout << t.celsius() << " C\n"; }

class Range {
public:
    Range(int lo, int hi) : hi_{hi}, lo_{lo}, width_{hi_ - lo_} {}   // list order is irrelevant ...
    int width() const { return width_; }
private:
    int lo_, hi_, width_;                                      // ... declaration order is what counts
};

int main() {
    Temperature room;
    report(room);
    report(Temperature{25});
    report(Temperature::from_fahrenheit(212));
    // report(30.0);   error with explicit; without it, 30.0 would silently become a Temperature

    Range r{10, 15};
    std::cout << "width " << r.width() << " (expected 5)\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra ctors.cpp -o ctors && ./ctors
ctors.cpp: In constructor ‘Range::Range(int, int)’:
ctors.cpp:22:14: warning: ‘Range::hi_’ will be initialized after [-Wreorder]
   22 |     int lo_, hi_, width_;                                      // ... declaration order is what counts
      |              ^~~
ctors.cpp:22:9: warning:   ‘int Range::lo_’ [-Wreorder]
   22 |     int lo_, hi_, width_;                                      // ... declaration order is what counts
      |         ^~~
ctors.cpp:19:5: warning:   when initialized here [-Wreorder]
   19 |     Range(int lo, int hi) : hi_{hi}, lo_{lo}, width_{hi_ - lo_} {}   // list order is irrelevant ...
      |     ^~~~~
20 C
25 C
100 C
width 5 (expected 5)
```

The `Range` class has a bug that the compiler diagnoses: `lo_` is declared first, so it is initialised first; then `hi_`; then `width_` — and `-Wall` (`-Wreorder`) warns that the initialiser list is written in a different order than execution. Here the order happens to work because `width_` is declared last; had `width_` been declared before `lo_`, it would have read uninitialised members. Always write initialiser lists in declaration order.

### Kinds of constructors

| Constructor | Signature | Called when |
| --- | --- | --- |
| Default | `T()` | `T x;`, `T x{};`, arrays and containers of `T` |
| Parameterised | `T(args…)` | `T x{args…}` |
| Copy | `T(const T&)` | `T y = x;`, pass/return by value (unless elided) |
| Move | `T(T&&)` | initialising from a temporary or `std::move(x)` (Chapter 3) |
| Delegating | `T() : T(0, 0) {}` | one constructor reusing another |
| Converting | non-`explicit` single-argument | implicit conversions — usually unwanted |

If you declare *any* constructor, the compiler no longer generates the default constructor; write `T() = default;` to get it back.

<!-- expanded -->
### Going deeper: members are initialised in declaration order

The member initialiser list does not decide the order in which members are initialised: they are always initialised in the order they are *declared* in the class. Writing the list in a different order compiles, and when one member's initialiser reads another, it can read a member that has not been initialised yet.

```cpp
// file: initorder.cpp
#include <iostream>
#include <string>

class Message {
public:
    explicit Message(const std::string& text) : text_(text), length_(text_.size()) {}
    // looks fine, but length_ is declared first, so it is initialised first,
    // from text_ — which has not been constructed yet
    std::size_t length() const { return length_; }
private:
    std::size_t length_;
    std::string text_;
};

int main() {
    Message m("hello");
    std::cout << "length " << m.length() << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra initorder.cpp -o initorder 2>&1 | grep -oE "warning: .*\[-W[a-z-]+\]" | head -2
warning: ‘Message::text_’ will be initialized after [-Wreorder]
warning:   ‘std::size_t Message::length_’ [-Wreorder]
$ g++-14 -std=c++20 -g -fsanitize=address,undefined initorder.cpp -o initorder_asan && ./initorder_asan 2>&1 | head -1
length 0
```

The program printed `length 0`, not 5: `text_.size()` was called on a `std::string` that had not been constructed yet — undefined behaviour that here happened to read zero bytes (another build could read garbage or crash). `-Wall` catches this as `-Wreorder` — the initialiser list order does not match the declaration order — and the fix is to initialise from the constructor *parameter* (`length_(text.size())`) or reorder the declarations.

### Deep & Tricky

**DT-11. A constructor with one parameter (and no `explicit`) is an implicit conversion: `void send(Packet p); send(42);` silently builds a `Packet` from 42. Mark single-argument constructors `explicit` unless the conversion is genuinely meant to be invisible (like `std::string` from `const char*`).** *(Guarantee: [class.conv.ctor] — a non-explicit constructor is a converting constructor)* The C++ Core Guidelines (C.46) make `explicit` the default; `clang-tidy`'s `google-explicit-constructor` check flags the omission.

```cpp
// file: explicit.cpp
#include <iostream>

struct Buffer { Buffer(int size) : size(size) {} int size; };              // converting constructor
struct SafeBuffer { explicit SafeBuffer(int size) : size(size) {} int size; };

void upload(const Buffer& b) { std::cout << "uploading a buffer of " << b.size << " bytes\n"; }
void upload_safe(const SafeBuffer& b) { std::cout << "uploading " << b.size << " bytes\n"; }

int main() {
    upload(1024);                   // compiles: an int silently became a Buffer
    upload('A');                    // compiles too: char -> int -> Buffer, 65 bytes
    upload_safe(SafeBuffer(1024));  // the conversion must be written out
    // upload_safe(1024);           // error: no implicit conversion
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra explicit.cpp -o explicit && ./explicit
uploading a buffer of 1024 bytes
uploading a buffer of 65 bytes
uploading 1024 bytes
```

### Mastery check

1. **Predict.** If `Range` declared its members as `int width_, lo_, hi_;`, what would `width()` return?

## Unit 2.3 — Destructors and RAII

**One-line summary:** A **destructor** (`~T()`) runs automatically when an object's lifetime ends: at the end of its scope for locals (in reverse order of construction), when `delete` is called, when its owner is destroyed. Put resource release in destructors and the language guarantees it happens — even when a function returns early or an exception propagates.

```cpp
// file: raii.cpp
#include <cstdio>
#include <iostream>
#include <stdexcept>
#include <string>

class File {                                 // RAII wrapper for a C FILE*
public:
    File(const std::string& path, const char* mode) : f_{std::fopen(path.c_str(), mode)}, path_{path} {
        if (f_ == nullptr)
            throw std::runtime_error("cannot open " + path);
        std::cout << "  [open " << path_ << "]\n";
    }
    ~File() {
        if (f_) {
            std::fclose(f_);
            std::cout << "  [close " << path_ << "]\n";
        }
    }
    File(const File&) = delete;              // a file handle must not be copied (Unit 2.4)
    File& operator=(const File&) = delete;

    void write_line(const std::string& s) {
        if (std::fprintf(f_, "%s\n", s.c_str()) < 0)
            throw std::runtime_error("write failed");
    }
private:
    std::FILE* f_;
    std::string path_;
};

void process(bool fail_midway) {
    File a{"a.txt", "w"};
    File b{"b.txt", "w"};
    a.write_line("first");
    if (fail_midway)
        throw std::runtime_error("something went wrong");   // both files still closed
    b.write_line("second");
}                                            // destructors run here: b, then a

int main() {
    std::cout << "normal run:\n";
    process(false);
    std::cout << "failing run:\n";
    try {
        process(true);
    } catch (const std::exception& e) {
        std::cout << "caught: " << e.what() << '\n';
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra raii.cpp -o raii && ./raii
normal run:
  [open a.txt]
  [open b.txt]
  [close b.txt]
  [close a.txt]
failing run:
  [open a.txt]
  [open b.txt]
  [close b.txt]
  [close a.txt]
caught: something went wrong
```

On both paths, both files were closed, in reverse order of opening, with no cleanup code in `process` at all. Compare the `goto` ladder of C (Unit 1.7 of the C book): the destructor is that ladder, written once in the class and applied automatically everywhere the class is used. The standard library is built this way: `std::vector` frees its memory, `std::fstream` closes its file, `std::lock_guard` unlocks its mutex, `std::unique_ptr` deletes its object.

### Pattern

**P-1. Every resource is owned by exactly one RAII object.** *(Practice)* Never hold a raw resource (a `FILE*`, a file descriptor, a `new`ed pointer, a locked mutex) in a plain variable across code that might return or throw. Wrap it immediately in an object whose destructor releases it — a standard type when one exists (`std::unique_ptr` with a custom deleter, `std::fstream`, `std::lock_guard`), or a small class like `File`. *Trade-off:* a little wrapper code in exchange for leak-free, exception-safe code everywhere the resource is used.

### Deep & Tricky

**DT-12. A destructor must not throw.** *(Guarantee: destructors are `noexcept` by default since C++11)* If a destructor throws while another exception is already propagating (during stack unwinding), the program calls `std::terminate`. Since C++11, destructors are implicitly `noexcept`, so any exception escaping one terminates the program immediately. Report errors from cleanup in another way (log, or offer an explicit `close()` that can throw, with the destructor as a silent fallback).

### Mastery check

1. **Explain.** In which order were the four `[open]`/`[close]` lines printed in the failing run, and why?

## Topic 2B — Copying and operators

## Unit 2.4 — Copy semantics and the Rule of Three

**One-line summary:** By default, copying an object copies each member. For a class that owns a resource through a raw pointer or handle, that **shallow copy** leaves two objects owning one resource — a double free waiting to happen. Either forbid copying (`= delete`), or write a **deep copy**: if a class needs a custom destructor, copy constructor, or copy assignment, it needs all three (the **Rule of Three**).

```cpp
// file: buffer.cpp
#include <algorithm>
#include <cstddef>
#include <iostream>

class Buffer {                          // owns a heap array: needs the Rule of Three
public:
    explicit Buffer(std::size_t n) : size_{n}, data_{new int[n]{}} {}
    ~Buffer() { delete[] data_; }

    Buffer(const Buffer& other) : size_{other.size_}, data_{new int[other.size_]} {   // deep copy
        std::copy(other.data_, other.data_ + size_, data_);
    }

    Buffer& operator=(const Buffer& other) {        // copy-and-swap: strong exception safety
        Buffer tmp{other};                          // may throw: *this is untouched if it does
        std::swap(size_, tmp.size_);
        std::swap(data_, tmp.data_);
        return *this;                               // tmp's destructor frees our old array
    }

    int& operator[](std::size_t i) { return data_[i]; }
    std::size_t size() const { return size_; }

private:
    std::size_t size_;
    int* data_;
};

int main() {
    Buffer a{3};
    a[0] = 1; a[1] = 2; a[2] = 3;
    Buffer b = a;           // copy constructor
    b[0] = 100;
    Buffer c{1};
    c = a;                  // copy assignment
    c = c;                  // self-assignment must be safe
    std::cout << "a[0]=" << a[0] << " b[0]=" << b[0] << " c[2]=" << c[2] << " c.size=" << c.size() << std::endl;   // endl flushes: see below
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -fsanitize=address buffer.cpp -o buffer && ./buffer
a[0]=1 b[0]=100 c[2]=3 c.size=3
```

Each `Buffer` owns its own array, so modifying `b` did not affect `a`, and AddressSanitizer found no double frees or leaks. The **copy-and-swap** idiom makes assignment simple and safe: build the new state in a temporary first (if that throws, nothing changed), then swap it in, and let the temporary's destructor free the old state — self-assignment works automatically.

Now delete the copy constructor and assignment from `Buffer` and let the compiler generate them:

```console
$ sed -e '/Buffer(const Buffer& other)/,/^    }$/d' -e '/Buffer& operator=(const Buffer& other)/,/^    }$/d' buffer.cpp > buffer_shallow.cpp && g++-14 -std=c++20 -g -fsanitize=address buffer_shallow.cpp -o buffer_shallow && ./buffer_shallow 2>&1 | grep -E 'ERROR|a\[0\]' | head -n 2
a[0]=100 b[0]=100 c[2]=3 c.size=3
==7367==ERROR: AddressSanitizer: attempting double-free on 0x502000000010 in thread T0:
```

The generated copy shares the pointer: `b[0] = 100` changed `a` as well (`a[0]=100`), and at the end of `main` the same array is deleted twice — AddressSanitizer reports the double free. (The output line uses `std::endl` rather than `'\n'` so that it is flushed before the program aborts; when output goes to a pipe, `std::cout` is fully buffered, and an abort loses whatever is still in the buffer — the first run of this example printed only the error.)

### Rule of Zero

The best fix is not to write the three at all: hold resources in members that already manage themselves. A class with `std::vector<int> data_;` instead of `int* data_;` gets correct copying, moving, and destruction **generated by the compiler** — the **Rule of Zero**. Write custom copy/move/destructor only in small classes whose entire job is managing one resource (like `File`), and build everything else from those.

<!-- expanded -->
### Going deeper: copy-and-swap

Writing a correct copy assignment operator by hand is surprisingly hard: it must handle self-assignment (`a = a`), and it must not leave the object half-changed if allocating the new storage throws. The **copy-and-swap** idiom gets both right in three lines: take the argument *by value* (the copy constructor makes the copy, and if it throws, `*this` has not been touched), then swap the copy's contents with `*this`, and let the parameter's destructor free the old contents.

```cpp
// file: copyswap.cpp
#include <algorithm>
#include <cstddef>
#include <iostream>
#include <utility>

class IntArray {
public:
    explicit IntArray(std::size_t n) : size_(n), data_(new int[n]{}) {}
    IntArray(const IntArray& o) : size_(o.size_), data_(new int[o.size_]) {
        std::copy(o.data_, o.data_ + size_, data_);
    }
    IntArray& operator=(IntArray other) {        // by value: the copy happens here
        swap(*this, other);                      // cannot fail
        return *this;                            // other's destructor frees our old array
    }
    ~IntArray() { delete[] data_; }
    friend void swap(IntArray& a, IntArray& b) noexcept {
        using std::swap;
        swap(a.size_, b.size_);
        swap(a.data_, b.data_);
    }
    int& operator[](std::size_t i) { return data_[i]; }
    std::size_t size() const { return size_; }
private:
    std::size_t size_;
    int* data_;
};

int main() {
    IntArray a(3), b(5);
    a[0] = 7;
    b = a;                     // b now has 3 elements
    a = a;                     // self-assignment: harmless
    b[1] = 9;
    std::cout << "a: " << a[0] << ' ' << a[1] << " (size " << a.size() << "), b: " << b[0] << ' ' << b[1]
              << " (size " << b.size() << ")\n";
}
```

```console
$ g++-14 -std=c++20 -g -fsanitize=address -Wall -Wextra copyswap.cpp -o copyswap && ./copyswap
a: 7 0 (size 3), b: 7 9 (size 3)
```

Taking the parameter by value also makes the same operator serve as the *move* assignment once a move constructor exists: `b = std::move(a)` move-constructs the parameter, then swaps. The cost is one extra swap compared with a hand-tuned version, and losing the chance to reuse `*this`'s existing buffer when it is already big enough — which is why `std::vector` does not use this idiom, and most application classes should.

### Mastery check

1. **Explain.** Why is self-assignment (`c = c`) dangerous in a naive assignment operator that first does `delete[] data_`?

## Unit 2.5 — Operator overloading

**One-line summary:** Operators are functions named `operator+`, `operator==`, `operator<<`, and so on. Overload them when a type has a natural meaning for them (numbers, containers, iterators), keep their usual semantics, and use C++20's **defaulted comparisons** (`operator==` and `operator<=>`) to get all six relational operators from one line.

```cpp
// file: money.cpp
#include <compare>
#include <iostream>

class Money {
public:
    constexpr explicit Money(long long cents = 0) : cents_{cents} {}
    constexpr long long cents() const { return cents_; }

    Money& operator+=(Money o) { cents_ += o.cents_; return *this; }     // compound: member
    friend Money operator+(Money a, Money b) { return a += b; }         // binary: non-member (friend)
    friend Money operator*(Money a, long long k) { return Money{a.cents_ * k}; }
    Money operator-() const { return Money{-cents_}; }                  // unary minus

    auto operator<=>(const Money&) const = default;   // generates <, <=, >, >= (and == below)
    bool operator==(const Money&) const = default;

    friend std::ostream& operator<<(std::ostream& os, Money m) {        // streaming
        long long c = m.cents_ < 0 ? -m.cents_ : m.cents_;
        return os << (m.cents_ < 0 ? "-" : "") << '$' << c / 100 << '.' << (c % 100 < 10 ? "0" : "") << c % 100;
    }

private:
    long long cents_;
};

int main() {
    Money price{1999}, tax{160};
    Money total = price + tax;
    std::cout << price << " + " << tax << " = " << total << '\n';
    std::cout << "x3: " << total * 3 << ", negated: " << -total << '\n';
    std::cout << std::boolalpha << "price < total: " << (price < total)
              << ", total == $21.59: " << (total == Money{2159}) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra money.cpp -o money && ./money
$19.99 + $1.60 = $21.59
x3: $64.77, negated: -$21.59
price < total: true, total == $21.59: true
```

Guidelines:

- Implement `+=` as a member and `+` as a non-member in terms of it; non-member binary operators treat both operands symmetrically (implicit conversions apply to both).
- Defining operators as `friend`s inside the class (the "hidden friend" idiom) makes them visible only through argument-dependent lookup — fewer overloads for the compiler to consider, faster builds, clearer errors.
- `= default` comparison compares members in declaration order. `<=>` returns `std::strong_ordering` for types like integers; `std::partial_ordering` for floating point (because of NaN).
- Do not overload `&&`, `||`, or `,` — overloaded versions lose short-circuit evaluation and sequencing.

<!-- expanded -->
### Going deeper: defaulted comparisons and the spaceship operator

C++20 can write comparison operators for you. `bool operator==(const T&) const = default;` compares members in order; `auto operator<=>(const T&) const = default;` (the **three-way comparison**, or "spaceship" operator) gives all of `<`, `<=`, `>`, `>=` comparing member by member, like a dictionary orders words. The result type tells you what kind of ordering it is: `std::strong_ordering` for integers, `std::partial_ordering` for floating point (because `NaN` is not ordered with anything).

```cpp
// file: spaceship.cpp
#include <compare>
#include <iostream>
#include <set>
#include <string>

struct Version {
    int major, minor, patch;
    auto operator<=>(const Version&) const = default;   // compares major, then minor, then patch
    bool operator==(const Version&) const = default;
};

struct Reading {
    std::string sensor;
    double value;
    auto operator<=>(const Reading&) const = default;  // partial_ordering because of the double
};

int main() {
    Version a{1, 4, 2}, b{1, 10, 0};
    std::cout << std::boolalpha << "1.4.2 < 1.10.0: " << (a < b) << '\n';
    std::set<Version> vs{{2, 0, 0}, {1, 10, 0}, {1, 4, 2}};
    for (auto& v : vs) std::cout << v.major << '.' << v.minor << '.' << v.patch << ' ';
    std::cout << '\n';

    Reading r1{"t1", 0.0 / 0.0}, r2{"t1", 1.0};                 // a NaN reading
    auto c = r1 <=> r2;
    std::cout << "NaN vs 1.0 unordered: " << (c == std::partial_ordering::unordered)
              << ", r1 < r2: " << (r1 < r2) << ", r1 > r2: " << (r1 > r2) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra spaceship.cpp -o spaceship && ./spaceship
1.4.2 < 1.10.0: true
1.4.2 1.10.0 2.0.0 
NaN vs 1.0 unordered: true, r1 < r2: false, r1 > r2: false
```

Comparing versions as strings would have put `"1.10.0"` before `"1.4.2"`; comparing as members gets it right. With a `NaN` member, neither `r1 < r2` nor `r1 > r2` is true — which is exactly why putting such values in a `std::set` or sorting them is dangerous (DT-20).

### Mastery check

1. **Write.** A `Vec2` with `+`, `-`, scalar `*`, `==` (defaulted), and `<<`, plus a `length()` member.

## Unit 2.6 — Static members, `this`, and special member summary

**One-line summary:** `static` data members belong to the class, not to each object (one copy shared by all); `static` member functions have no `this`. `inline static` members (C++17) can be defined in the class. The compiler can generate six **special member functions** — default constructor, destructor, copy constructor, copy assignment, move constructor, move assignment — and the rules for when it does are worth knowing by heart.

```cpp
// file: statics.cpp
#include <iostream>

class Widget {
public:
    Widget() : id_{++next_id_} { ++alive_; }
    Widget(const Widget&) : id_{++next_id_} { ++alive_; }   // copies get new ids
    ~Widget() { --alive_; }
    Widget& operator=(const Widget&) { return *this; }      // keep our own id
    int id() const { return id_; }
    static int alive() { return alive_; }                   // no this: class-level query
private:
    int id_;
    inline static int next_id_ = 0;                         // one shared counter (C++17 inline)
    inline static int alive_ = 0;
};

int main() {
    Widget a, b;
    {
        Widget c = a;
        std::cout << "ids " << a.id() << b.id() << c.id() << ", alive " << Widget::alive() << '\n';
    }
    std::cout << "after scope, alive " << Widget::alive() << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra statics.cpp -o statics && ./statics
ids 123, alive 3
after scope, alive 2
```

### When the compiler generates special members

| You declare… | Default ctor | Destructor | Copy ctor/assign | Move ctor/assign |
| --- | --- | --- | --- | --- |
| nothing | generated | generated | generated | generated |
| any constructor | **not** generated | generated | generated | generated |
| destructor | generated | yours | generated (deprecated) | **not** generated (copies used instead) |
| copy ctor or copy assign | (if no ctor) | generated | yours / generated (deprecated) | **not** generated |
| move ctor or move assign | (if no ctor) | generated | **deleted** | yours |

The practical rule (**Rule of Five**): if you write any of destructor, copy constructor, copy assignment, move constructor, move assignment, think about all five — and preferably write none (Rule of Zero).

### Deep & Tricky

**DT-13. Declaring a destructor silently disables move operations.** *(Guarantee: special member generation rules)* Adding `~Widget() {}` for logging makes the compiler stop generating the move constructor and move assignment; "moves" of `Widget` then fall back to copies, which can make code dramatically slower without any error. If you need a destructor and want moves, declare them: `Widget(Widget&&) = default; Widget& operator=(Widget&&) = default;` (and then the copies too).

### Mastery check

1. **Predict.** In `statics.cpp`, what ids do `a`, `b`, `c` get, and why does `c` not share `a`'s id?

## Chapter 2 Project — A rational number type and a line-reading RAII file

**Goal:** two classes that show the two faces of C++ classes: `Rational`, a **value type** with an invariant (always in lowest terms, positive denominator) and a full set of operators; and `LineReader`, an **RAII resource** wrapper with deleted copying. Both tested with assertions under sanitizers.

### Reference solution

```cpp
// file: rational/rational.hpp
#pragma once
#include <compare>
#include <cstdint>
#include <numeric>
#include <ostream>
#include <stdexcept>

class Rational {
public:
    constexpr Rational(std::int64_t num = 0, std::int64_t den = 1) : num_{num}, den_{den} {
        if (den_ == 0)
            throw std::domain_error("zero denominator");
        normalise();
    }
    constexpr std::int64_t num() const { return num_; }
    constexpr std::int64_t den() const { return den_; }

    constexpr Rational& operator+=(const Rational& o) { return *this = Rational{num_ * o.den_ + o.num_ * den_, den_ * o.den_}; }
    constexpr Rational& operator-=(const Rational& o) { return *this = Rational{num_ * o.den_ - o.num_ * den_, den_ * o.den_}; }
    constexpr Rational& operator*=(const Rational& o) { return *this = Rational{num_ * o.num_, den_ * o.den_}; }
    constexpr Rational& operator/=(const Rational& o) { return *this = Rational{num_ * o.den_, den_ * o.num_}; }

    friend constexpr Rational operator+(Rational a, const Rational& b) { return a += b; }
    friend constexpr Rational operator-(Rational a, const Rational& b) { return a -= b; }
    friend constexpr Rational operator*(Rational a, const Rational& b) { return a *= b; }
    friend constexpr Rational operator/(Rational a, const Rational& b) { return a /= b; }
    constexpr Rational operator-() const { return Rational{-num_, den_}; }

    friend constexpr bool operator==(const Rational&, const Rational&) = default;   // normalised: memberwise is right
    friend constexpr std::strong_ordering operator<=>(const Rational& a, const Rational& b) {
        return a.num_ * b.den_ <=> b.num_ * a.den_;     // dens are positive: cross-multiply
    }
    friend std::ostream& operator<<(std::ostream& os, const Rational& r) {
        os << r.num_;
        if (r.den_ != 1) os << '/' << r.den_;
        return os;
    }
    constexpr explicit operator double() const { return static_cast<double>(num_) / static_cast<double>(den_); }

private:
    constexpr void normalise() {                         // establishes the invariant
        std::int64_t g = std::gcd(num_, den_);
        num_ /= g;
        den_ /= g;
        if (den_ < 0) { num_ = -num_; den_ = -den_; }
    }
    std::int64_t num_, den_;
};
```

(Overflow of the 64-bit products is not handled — a real library would use 128-bit intermediates or arbitrary precision; the exercises ask you to add checks with `__builtin_mul_overflow`.)

```cpp
// file: rational/line_reader.hpp
#pragma once
#include <cstdio>
#include <optional>
#include <stdexcept>
#include <string>

class LineReader {                        // owns a FILE*; yields lines one at a time
public:
    explicit LineReader(const std::string& path) : f_{std::fopen(path.c_str(), "r")} {
        if (!f_) throw std::runtime_error("cannot open " + path);
    }
    ~LineReader() { if (f_) std::fclose(f_); }
    LineReader(const LineReader&) = delete;
    LineReader& operator=(const LineReader&) = delete;

    std::optional<std::string> next() {
        std::string line;
        int c;
        while ((c = std::fgetc(f_)) != EOF && c != '\n')
            line.push_back(static_cast<char>(c));
        if (c == EOF && line.empty())
            return std::nullopt;
        ++count_;
        return line;
    }
    int lines_read() const { return count_; }
private:
    std::FILE* f_;
    int count_ = 0;
};
```

```cpp
// file: rational/test.cpp
#include <cassert>
#include <fstream>
#include <iostream>
#include <sstream>
#include "line_reader.hpp"
#include "rational.hpp"

int main() {
    // Rational: invariant and arithmetic
    static_assert(Rational(6, -8) == Rational(-3, 4));          // checked at compile time
    static_assert(Rational(1, 3) + Rational(1, 6) == Rational(1, 2));
    static_assert(Rational(1, 2) < Rational(2, 3));
    Rational r{2, 4};
    assert(r.num() == 1 && r.den() == 2);
    r *= Rational{4};
    assert(r == Rational{2});
    assert(-Rational(1, 3) < Rational(0));
    assert(Rational(7, 3) / Rational(7, 6) == Rational(2));
    std::ostringstream os;
    os << Rational(-10, 4) << ' ' << Rational(8, 4);
    assert(os.str() == "-5/2 2");
    bool threw = false;
    try { Rational bad{1, 0}; } catch (const std::domain_error&) { threw = true; }
    assert(threw);

    // harmonic number H(10) as an exact fraction
    Rational h;
    for (int k = 1; k <= 10; ++k) h += Rational(1, k);
    std::cout << "H(10) = " << h << " ~= " << static_cast<double>(h) << '\n';

    // LineReader: RAII and reading
    { std::ofstream out{"lines.txt"}; out << "alpha\nbeta\n\ngamma"; }   // last line has no newline
    LineReader reader{"lines.txt"};
    std::string all;
    while (auto line = reader.next())
        all += "[" + *line + "]";
    assert(all == "[alpha][beta][][gamma]");
    assert(reader.lines_read() == 4);
    threw = false;
    try { LineReader missing{"no-such-file.txt"}; } catch (const std::runtime_error&) { threw = true; }
    assert(threw);
    std::cout << "lines: " << all << "\nall tests passed\n";
}
```

```console
$ cd rational && g++-14 -std=c++20 -Wall -Wextra -pedantic -g -fsanitize=address,undefined test.cpp -o test && ./test
H(10) = 7381/2520 ~= 2.92897
lines: [alpha][beta][][gamma]
all tests passed
```

The `static_assert`s run the `Rational` arithmetic **inside the compiler**: because the class is `constexpr` throughout, its core behaviour is verified before the program is even built — and any undefined behaviour in those paths would be a compile error. H(10) = 7381/2520 is computed exactly, which floating point cannot do.

### Review checklist

- Can any `Rational` violate the invariant (zero or negative denominator, non-reduced form)? Through which member function?
- Is `LineReader` impossible to copy, and does its file close on every path (including the constructor throwing)?
- Would `Rational` work correctly if its `==` were defaulted but the invariant were not maintained? (No — that is why the invariant matters.)

## Answers to the Chapter 2 prediction questions

**2.2 Q1.** Unspecified garbage (formally, undefined behaviour): `width_` would be initialised first, from `hi_` and `lo_`, which have not been initialised yet. `-Wall` warns.

**2.6 Q1.** `a` gets 1, `b` gets 2, and `c` gets 3: the copy constructor is user-defined and takes a new id from the shared counter rather than copying `a`'s.

### Answers for this chapter's expansions

- **Unit 2.2, `initorder.cpp`:** the fix is `length_(text.size())`, or declaring `text_` before `length_`. Either way, `-Wall` is silent afterwards and the program prints `length 5`.

# Chapter 3 — Value Semantics and Move Semantics

*Goal: understand how C++ moves data instead of copying it — value categories, rvalue references, move constructors, `std::move` and `std::forward`, copy elision — and design types and functions that are both safe and fast.*

C++ objects are **values**: assigning or passing one copies it, just like an `int`. Value semantics make programs easy to reason about — no hidden sharing — but copying a large vector or string is expensive. C++11 solved this with **move semantics**: when the source of a copy is about to disappear anyway (a temporary, or an object you explicitly give up), its resources can be *transferred* instead of duplicated. This chapter explains the machinery precisely, because it underlies everything from returning containers by value to `std::unique_ptr`.

**What you will be able to do after this chapter**

- Classify expressions as lvalues, prvalues, and xvalues, and explain which overloads they bind to.
- Write move constructors and move assignment operators, and explain why they should be `noexcept`.
- Use `std::move` and `std::forward` correctly, and write perfect-forwarding functions.
- Rely on copy elision, and know when *not* to write `std::move`.
- Choose parameter-passing styles: by value, by `const&`, by `&&`.

## Topic 3A — The machinery

## Unit 3.1 — Value categories and rvalue references

**One-line summary:** Every expression is an **lvalue** (has an identity you can refer to again: a named variable), a **prvalue** (a pure temporary value: `42`, `f()` returning by value, `std::string("x")`), or an **xvalue** (an expiring object: the result of `std::move(x)`). An **rvalue reference** `T&&` binds only to rvalues (prvalues and xvalues) — that is, to objects it is safe to steal from.

```cpp
// file: categories.cpp
#include <iostream>
#include <string>
#include <utility>

void take(const std::string& s) { std::cout << "const& (lvalue or anything): " << s << '\n'; }
void take(std::string&& s) { std::cout << "&& (rvalue: may steal): " << s << '\n'; }

std::string make() { return "from a function"; }

int main() {
    std::string name = "named";
    take(name);                       // lvalue -> const&
    take(std::string("temporary"));   // prvalue -> &&
    take(make());                     // prvalue -> &&
    take(std::move(name));            // xvalue -> &&  (std::move is just a cast)
    take("literal");                  // creates a temporary std::string -> &&

    std::string&& r = make();         // an rvalue reference variable ...
    take(r);                          // ... is itself an lvalue (it has a name): const&
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra categories.cpp -o categories && ./categories
const& (lvalue or anything): named
&& (rvalue: may steal): temporary
&& (rvalue: may steal): from a function
&& (rvalue: may steal): named
&& (rvalue: may steal): literal
const& (lvalue or anything): from a function
```

The last two lines contain the most important rule of this chapter: **a named rvalue reference is an lvalue**. `r` has type `std::string&&`, but the expression `r` names an object you could use again on the next line, so it is not treated as expiring. To pass it on as an rvalue you must say so again with `std::move(r)`.

### `std::move` moves nothing

`std::move(x)` is `static_cast<T&&>(x)`: it only changes the expression's category to xvalue, which lets overload resolution pick the `&&` overload. The actual transfer happens in the move constructor or move assignment that gets selected. If the type has no move operations, `std::move` silently results in a copy.

<!-- expanded -->
### Going deeper: `decltype` shows the value category

The type `decltype` reports for an expression encodes its value category: for an lvalue expression it gives `T&`, for an xvalue (such as `std::move(x)`) `T&&`, and for a prvalue plain `T`. This is the most reliable way to check your understanding of categories — let the compiler tell you.

```cpp
// file: categories2.cpp
#include <iostream>
#include <string>
#include <type_traits>
#include <utility>

template <typename T> const char* category() {
    if constexpr (std::is_lvalue_reference_v<T>) return "lvalue";
    else if constexpr (std::is_rvalue_reference_v<T>) return "xvalue";
    else return "prvalue";
}
#define SHOW(expr) std::cout << #expr << ": " << category<decltype((expr))>() << '\n'

std::string make() { return "temp"; }
std::string& ref(std::string& s) { return s; }

int main() {
    std::string s = "x";
    int a[3]{};
    SHOW(s);
    SHOW(s + "y");
    SHOW(std::move(s));
    SHOW(make());
    SHOW(ref(s));
    SHOW(a[1]);
    SHOW(42);
    SHOW("literal");
    SHOW(++a[0]);
    SHOW(a[0]++);
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra categories2.cpp -o categories2 && ./categories2
s: lvalue
s + "y": prvalue
std::move(s): xvalue
make(): prvalue
ref(s): lvalue
a[1]: lvalue
42: prvalue
"literal": lvalue
++a[0]: lvalue
a[0]++: prvalue
```

Two results surprise people. A string literal is an *lvalue* (it is an array that lives for the whole program), while `42` is a prvalue. And `++x` is an lvalue (it returns the variable itself) while `x++` is a prvalue (it returns a copy of the old value) — one reason `++it` is the habit for iterators.

### Mastery check

1. **Classify.** `x`, `x + 1`, `std::move(x)`, `"abc"`, `std::string("abc")`, `v[0]` (for a `std::vector<int> v`), `f()` where `f` returns `int&`.

## Unit 3.2 — Move constructors and move assignment

**One-line summary:** A move constructor `T(T&& other) noexcept` takes over `other`'s resources and leaves `other` in a **valid but unspecified** state (usually empty). Moves should be `noexcept`, because containers like `std::vector` will only move elements during reallocation if moving cannot throw — otherwise they copy, to keep their strong exception guarantee.

```cpp
// file: moves.cpp
#include <cstring>
#include <iostream>
#include <utility>
#include <vector>

static int copies, moves;

class Text {
public:
    explicit Text(const char* s = "") : len_{std::strlen(s)}, data_{new char[len_ + 1]} {
        std::memcpy(data_, s, len_ + 1);
    }
    ~Text() { delete[] data_; }

    Text(const Text& o) : len_{o.len_}, data_{new char[o.len_ + 1]} {      // copy: allocate
        std::memcpy(data_, o.data_, len_ + 1);
        ++copies;
    }
    Text(Text&& o) noexcept : len_{o.len_}, data_{o.data_} {               // move: steal
        o.len_ = 0;
        o.data_ = nullptr;                                                  // leave o destructible
        ++moves;
    }
    Text& operator=(Text o) noexcept {      // copy-and-swap, unified: o is copied or moved in
        std::swap(len_, o.len_);
        std::swap(data_, o.data_);
        return *this;
    }
    const char* c_str() const { return data_ ? data_ : ""; }

private:
    std::size_t len_;
    char* data_;
};

int main() {
    std::vector<Text> v;
    for (int i = 0; i < 1000; ++i)
        v.push_back(Text{"some text that needs a heap allocation"});   // temporaries: moved in
    std::cout << "push_back of temporaries: copies=" << copies << " moves=" << moves << '\n';

    copies = moves = 0;
    Text a{"source"};
    Text b = a;              // copy
    Text c = std::move(a);   // move: a is now empty but valid
    std::cout << "b=" << b.c_str() << " c=" << c.c_str() << " a=\"" << a.c_str() << "\"\n";
    a = Text{"reassigned"};  // a moved-from object can be assigned to
    std::cout << "a=" << a.c_str() << ", copies=" << copies << " moves=" << moves << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -fsanitize=address moves.cpp -o moves && ./moves
push_back of temporaries: copies=0 moves=2023
b=source c=source a=""
a=reassigned, copies=1 moves=1
```

1,000 `push_back` calls produced **zero copies**: each temporary was moved into the vector, and every time the vector reallocated (about ten times while growing to 1,000), its existing elements were moved, not copied, to the new storage — about 2,000 moves in total. Now remove the `noexcept`:

```console
$ sed 's/Text(Text&& o) noexcept/Text(Text\&\& o)/' moves.cpp > moves_throwing.cpp && g++-14 -std=c++20 -Wall moves_throwing.cpp -o moves_throwing && ./moves_throwing | head -n 1
push_back of temporaries: copies=1023 moves=1000
```

Without `noexcept`, `std::vector` copied every existing element on each reallocation — over a thousand extra heap allocations — because if a move threw halfway through, the old elements would already be gutted and the vector could not roll back. `std::move_if_noexcept` implements that decision.

### The moved-from state

After `Text c = std::move(a);`, `a` is valid but unspecified. Valid means its invariants hold and it can be destroyed or assigned to; unspecified means you should not read its value. Standard library types are more specific in practice (a moved-from `std::string` or `std::vector` is empty in every major implementation, but the standard only promises "valid but unspecified" for most types; `std::unique_ptr` *is* guaranteed null).

### Deep & Tricky

**DT-14. A move constructor that is not `noexcept` makes `std::vector` copy instead of move.** *(Library behaviour: strong exception guarantee)* Measured above: a single missing keyword turned zero copies into over a thousand. Compiler-generated moves are `noexcept` when all members' moves are. Write `noexcept` on hand-written moves (they almost never need to throw: they only shuffle pointers), and check with `static_assert(std::is_nothrow_move_constructible_v<T>)`.

<!-- expanded -->
### Deep & Tricky

**DT-15. A moved-from standard library object is in a "valid but unspecified state": you may assign to it, destroy it, or call functions without preconditions (`size()`, `clear()`, `empty()`), but you must not assume it is empty. A moved-from `std::string` holding a short string usually still contains its characters.** *(Guarantee: [lib.types.movedfrom] — "valid but unspecified")* With libstdc++, moving from a short (SSO) string copies the characters and then sets the source to empty, but no rule requires that; code that reads a moved-from object and expects `""` is relying on one implementation. `clang-tidy`'s `bugprone-use-after-move` check catches reads after `std::move`.

```cpp
// file: movedfrom.cpp
#include <iostream>
#include <string>
#include <utility>
#include <vector>

int main() {
    std::string shortstr = "hi", longstr(40, 'x');
    std::vector<int> v{1, 2, 3};
    std::string a = std::move(shortstr), b = std::move(longstr);
    std::vector<int> w = std::move(v);
    std::cout << "moved-from short string size " << shortstr.size() << ", long string size " << longstr.size()
              << ", vector size " << v.size() << '\n';
    shortstr = "reused";                 // assigning gives it a known value again: always fine
    std::cout << shortstr << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra movedfrom.cpp -o movedfrom && ./movedfrom
moved-from short string size 0, long string size 0, vector size 0
reused
$ clang-tidy --checks='-*,bugprone-use-after-move' movedfrom.cpp -- -std=c++20 2>/dev/null | grep -m1 -oE "warning: .*"
warning: 'shortstr' used after it was moved [bugprone-use-after-move]
```

libstdc++ leaves all three empty; the standard only promises they are usable.

### Mastery check

1. **Explain.** Why does the move constructor set `o.data_ = nullptr`? What would happen without it?

## Unit 3.3 — Forwarding references and perfect forwarding

**One-line summary:** In a template, `T&&` where `T` is deduced is a **forwarding reference** (not an rvalue reference): it binds to anything and remembers whether the argument was an lvalue or an rvalue. `std::forward<T>(arg)` passes the argument on with its original category — **perfect forwarding** — which is how `std::make_unique`, `emplace_back`, and wrappers pass arguments through without extra copies.

```cpp
// file: forwarding.cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

struct Tracer {
    std::string name;
    explicit Tracer(std::string n) : name{std::move(n)} {}
    Tracer(const Tracer& o) : name{o.name} { std::cout << "  copy " << name << '\n'; }
    Tracer(Tracer&& o) noexcept : name{std::move(o.name)} { std::cout << "  move " << name << '\n'; }
};

struct Box {
    Tracer item;
    explicit Box(const Tracer& t) : item{t} {}            // copies
    explicit Box(Tracer&& t) : item{std::move(t)} {}      // moves
};

template <typename T, typename... Args>
std::unique_ptr<T> make(Args&&... args) {                 // forwarding references
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

template <typename T, typename... Args>
std::unique_ptr<T> make_broken(Args&&... args) {          // forgets std::forward
    return std::unique_ptr<T>(new T(args...));            // args are lvalues here: always copies
}

int main() {
    Tracer t{"lvalue-tracer"};
    std::cout << "make<Box>(t):\n";
    auto b1 = make<Box>(t);                               // lvalue in -> copy
    std::cout << "make<Box>(Tracer{...}):\n";
    auto b2 = make<Box>(Tracer{"temp-tracer"});           // rvalue in -> move
    std::cout << "make_broken<Box>(Tracer{...}):\n";
    auto b3 = make_broken<Box>(Tracer{"temp-tracer-2"});  // rvalue in -> copied anyway
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra forwarding.cpp -o forwarding && ./forwarding
make<Box>(t):
  copy lvalue-tracer
make<Box>(Tracer{...}):
  move temp-tracer
make_broken<Box>(Tracer{...}):
  copy temp-tracer-2
```

`make` preserved each argument's category: the named `t` was copied, the temporary was moved. `make_broken` copied the temporary, because inside the function the parameter has a name and is therefore an lvalue (Unit 3.1) — the information that the caller passed an rvalue is lost unless `std::forward` restores it.

### The deduction rule behind it

For a forwarding reference parameter `T&&`: passing an lvalue of type `X` deduces `T = X&`, and `X& &&` collapses to `X&` (**reference collapsing**); passing an rvalue deduces `T = X`, giving `X&&`. `std::forward<T>(a)` casts to `T&&` — an rvalue only if `T` was deduced as a non-reference.

Not every `&&` is a forwarding reference: `std::vector<T>&&`, `const T&&`, and `T&&` in a non-template member of a class template (where `T` is already fixed) are plain rvalue references.

### Mastery check

1. **Explain.** Why is `std::forward` used with forwarding references and `std::move` with rvalue references, and what goes wrong if you swap them?

## Topic 3B — Using it well

## Unit 3.4 — Copy elision and returning by value

**One-line summary:** Returning a local object by value does not copy it: C++17 **guarantees** elision for prvalues (`return T{…};`), and compilers perform **named return value optimisation** (NRVO) for named locals in practice. When elision does not apply, a returned local is automatically treated as an rvalue and moved. Therefore: return by value, and never write `return std::move(local);`.

```cpp
// file: elision.cpp
#include <iostream>
#include <string>
#include <utility>
#include <vector>

struct Tracer {
    Tracer() { std::cout << "  construct\n"; }
    Tracer(const Tracer&) { std::cout << "  copy\n"; }
    Tracer(Tracer&&) noexcept { std::cout << "  move\n"; }
};

Tracer prvalue() { return Tracer{}; }              // guaranteed elision (C++17)
Tracer named() { Tracer t; return t; }            // NRVO (allowed, done by GCC/Clang)
Tracer pessimised() { Tracer t; return std::move(t); }   // prevents NRVO: forces a move
Tracer two_paths(bool b) {                        // NRVO impossible: which object is returned?
    Tracer x, y;
    if (b) return x;
    return y;                                     // implicitly moved, not copied
}

std::vector<int> big() {
    std::vector<int> v(1'000'000, 7);
    return v;                                     // no copy of a million ints
}

int main() {
    std::cout << "prvalue:\n"; [[maybe_unused]] auto a = prvalue();
    std::cout << "named:\n"; [[maybe_unused]] auto b = named();
    std::cout << "pessimised:\n"; [[maybe_unused]] auto c = pessimised();
    std::cout << "two_paths:\n"; [[maybe_unused]] auto d = two_paths(true);
    auto v = big();
    std::cout << "big() returned " << v.size() << " elements\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -Wpessimizing-move elision.cpp -o elision && ./elision
elision.cpp: In function ‘Tracer pessimised()’:
elision.cpp:15:49: warning: moving a local object in a return statement prevents copy elision [-Wpessimizing-move]
   15 | Tracer pessimised() { Tracer t; return std::move(t); }   // prevents NRVO: forces a move
      |                                        ~~~~~~~~~^~~
elision.cpp:15:49: note: remove ‘std::move’ call
prvalue:
  construct
named:
  construct
pessimised:
  construct
  move
two_paths:
  construct
  construct
  move
big() returned 1000000 elements
```

GCC even warns (`-Wpessimizing-move`, enabled by `-Wall`) that `std::move` on the return prevents elision. Returning large containers by value is free; output parameters (`void fill(std::vector<int>& out)`) are no longer needed for performance.

<!-- expanded -->
### Going deeper: guaranteed elision versus NRVO

Since C++17, returning a **prvalue** of the same type (`return T(args);`) is *guaranteed* to construct the object directly in the caller's storage: no copy, no move, and the type does not even need a copy or move constructor. Returning a *named* local (`T t; ...; return t;`) is the **named return value optimisation** (NRVO), which compilers usually perform but are not required to — and they cannot when different `return` statements return different named objects.

```cpp
// file: nrvo.cpp
#include <iostream>

struct Tracer {
    Tracer() { std::cout << "  construct\n"; }
    Tracer(const Tracer&) { std::cout << "  copy\n"; }
    Tracer(Tracer&&) noexcept { std::cout << "  move\n"; }
};

Tracer prvalue() { return Tracer(); }                // guaranteed: no copy, no move
Tracer named() { Tracer t; return t; }               // NRVO: usually no copy, no move
Tracer two_names(bool f) {                           // NRVO impossible: which one to build in place?
    Tracer a, b;
    if (f) return a;
    return b;
}
struct NoMove { NoMove() = default; NoMove(const NoMove&) = delete; NoMove(NoMove&&) = delete; };
NoMove factory() { return NoMove(); }                // compiles only because of C++17 guaranteed elision

int main() {
    std::cout << "prvalue:\n";   Tracer a = prvalue();
    std::cout << "named:\n";     Tracer b = named();
    std::cout << "two names:\n"; Tracer c = two_names(true);
    NoMove n = factory();
    (void)a; (void)b; (void)c; (void)n;
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra nrvo.cpp -o nrvo && ./nrvo
prvalue:
  construct
named:
  construct
two names:
  construct
  construct
  move
$ g++-14 -std=c++20 -fno-elide-constructors nrvo.cpp -o nrvo_noelide && ./nrvo_noelide | tr '\n' ' '; echo
prvalue:   construct named:   construct   move two names:   construct   construct   move 
```

`-fno-elide-constructors` turns NRVO off but cannot turn off the guaranteed case: the prvalue call still builds one object, while `named()` now moves. `return std::move(t);` is a pessimisation: it turns an NRVO candidate into a guaranteed move (GCC warns with `-Wpessimizing-move`), and since C++11 a returned local is moved automatically anyway when elision does not happen.

### Mastery check

1. **Predict.** How many "copy" lines would `two_paths` print if `Tracer` had no move constructor (only a copy constructor)?

## Unit 3.5 — Passing parameters

**One-line summary:** Pass cheap-to-copy types (`int`, `double`, small structs, `string_view`, `span`) **by value**; pass objects you only read **by `const&`**; pass objects you will **store** ("sink" parameters) **by value and `std::move` them** into place; use `&&` parameters mainly in move constructors and in overloads optimised for rvalues.

```cpp
// file: params.cpp
#include <iostream>
#include <string>
#include <utility>
#include <vector>

static int allocations;
struct Str {                                   // instrumented string wrapper
    std::string s;
    Str(const char* p) : s{p} { ++allocations; }
    Str(const Str& o) : s{o.s} { ++allocations; }
    Str(Str&& o) noexcept = default;
    Str& operator=(Str&&) noexcept = default;
};

class Person {
public:
    explicit Person(Str name) : name_{std::move(name)} {}   // sink: by value, then move
private:
    Str name_;
};

int main() {
    Str n{"a long name that does not fit in the small string buffer"};
    allocations = 0;
    Person p1{n};                     // lvalue: one copy (into the parameter), then a move
    std::cout << "from lvalue: " << allocations << " allocation(s)\n";
    allocations = 0;
    Person p2{Str{"another long name that will not fit in SSO storage"}};   // temporary: zero copies
    std::cout << "from temporary: " << allocations << " allocation(s) (the temporary itself)\n";
    allocations = 0;
    Person p3{std::move(n)};          // explicitly given up: zero copies
    std::cout << "from std::move: " << allocations << " allocation(s)\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra params.cpp -o params && ./params
from lvalue: 1 allocation(s)
from temporary: 1 allocation(s) (the temporary itself)
from std::move: 0 allocation(s)
```

One constructor taking `Str` by value handles all three cases optimally (or within one cheap move of optimal), where the alternative is two overloads (`const Str&` and `Str&&`) per sink parameter. This is the standard advice for constructors and setters that store their argument.

### Pattern

**P-2. Return by value; take sinks by value and move; read through `const&`.** *(Practice)* Let functions return their results by value (elision makes it free), take parameters they store by value and `std::move` them into members, take parameters they only read by `const T&` (or `string_view`/`span`), and pass small trivially copyable types by value. *Trade-off:* sink-by-value costs one extra move compared with hand-written overload pairs, in exchange for half the code and no chance of forgetting an overload.

<!-- expanded -->
### Deep & Tricky

**DT-16. "Pass by value and move" (`void set(std::string s) { name_ = std::move(s); }`) costs one extra move compared with two overloads (`const std::string&` and `std::string&&`), and it allocates on every call where the overloads would reuse the existing buffer. For constructors it is the right default; for a setter called in a loop on a long-lived object, the `const&` overload can be much cheaper.** *(Library behaviour: `std::string::operator=(const std::string&)` reuses its capacity when the new value fits)* The measurement below calls a setter one million times with a 40-character string.

```cpp
// file: setter_cost.cpp
#include <chrono>
#include <cstdio>
#include <string>
#include <utility>

struct ByValue { std::string name; void set(std::string s) { name = std::move(s); } };
struct ByRef { std::string name; void set(const std::string& s) { name = s; } };

template <typename T> double run(const char* label) {
    T obj;
    std::string input(40, 'n');
    auto t0 = std::chrono::steady_clock::now();
    for (int i = 0; i < 1'000'000; ++i) { input[0] = char('a' + i % 26); obj.set(input); }
    double ms = std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
    std::printf("%-9s %6.1f ms (last: %c)\n", label, ms, obj.name[0]);
    return ms;
}

int main() {
    double v = run<ByValue>("by value");
    double r = run<ByRef>("const&");
    std::printf("by value / const& = %.1fx\n", v / r);
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra setter_cost.cpp -o setter_cost && ./setter_cost | awk '/by value \//{print ($6+0 > 1.3) ? "by value is clearly slower (ratio above 1.3)" : "no clear difference"}'
by value is clearly slower (ratio above 1.3)
```

The by-value setter must copy `input` into a fresh heap buffer (40 characters is past the small-string limit) every call; the `const&` setter copies into `name`'s existing buffer. On the machine used for this book the by-value setter took about 1.9 times as long (13 ms against 7 ms for a million calls); exact times vary, so the command prints only the verdict.

### Mastery check

1. **Write.** A `setName` member for `Person` following the pattern, and count allocations for an lvalue and an rvalue argument.

## Chapter 3 Project — A move-aware dynamic array

**Goal:** implement `DynArray<T>`, a simplified `std::vector`, with all five special members, `push_back` for lvalues and rvalues, `emplace_back` with perfect forwarding, and reallocation that moves elements when their move constructor is `noexcept` and copies otherwise. Verify behaviour by counting operations, and test under sanitizers.

### Reference solution

```cpp
// file: dynarray/dynarray.hpp
#pragma once
#include <cstddef>
#include <memory>
#include <new>
#include <stdexcept>
#include <type_traits>
#include <utility>

template <typename T>
class DynArray {
public:
    DynArray() = default;
    ~DynArray() { clear(); ::operator delete(data_, std::align_val_t{alignof(T)}); }

    DynArray(const DynArray& o) {                          // copy: deep, element by element
        reserve(o.size_);
        for (std::size_t i = 0; i < o.size_; ++i)
            emplace_back(o.data_[i]);
    }
    DynArray(DynArray&& o) noexcept                        // move: steal the buffer
        : data_{std::exchange(o.data_, nullptr)}, size_{std::exchange(o.size_, 0)}, cap_{std::exchange(o.cap_, 0)} {}
    DynArray& operator=(DynArray o) noexcept {             // copy-and-swap covers both assignments
        swap(o);
        return *this;
    }
    void swap(DynArray& o) noexcept {
        std::swap(data_, o.data_);
        std::swap(size_, o.size_);
        std::swap(cap_, o.cap_);
    }

    void push_back(const T& v) { emplace_back(v); }
    void push_back(T&& v) { emplace_back(std::move(v)); }

    template <typename... Args>
    T& emplace_back(Args&&... args) {                      // construct in place
        if (size_ == cap_)
            reserve(cap_ ? cap_ * 2 : 4);
        T* p = std::construct_at(data_ + size_, std::forward<Args>(args)...);
        ++size_;
        return *p;
    }

    void reserve(std::size_t n) {
        if (n <= cap_) return;
        T* fresh = static_cast<T*>(::operator new(n * sizeof(T), std::align_val_t{alignof(T)}));
        std::size_t i = 0;
        try {
            for (; i < size_; ++i)                         // move if noexcept, else copy
                std::construct_at(fresh + i, std::move_if_noexcept(data_[i]));
        } catch (...) {
            std::destroy_n(fresh, i);                      // roll back: old buffer untouched
            ::operator delete(fresh, std::align_val_t{alignof(T)});
            throw;
        }
        std::destroy_n(data_, size_);
        ::operator delete(data_, std::align_val_t{alignof(T)});
        data_ = fresh;
        cap_ = n;
    }

    void clear() noexcept { std::destroy_n(data_, size_); size_ = 0; }
    T& operator[](std::size_t i) { return data_[i]; }
    const T& operator[](std::size_t i) const { return data_[i]; }
    T& at(std::size_t i) { if (i >= size_) throw std::out_of_range("DynArray::at"); return data_[i]; }
    std::size_t size() const { return size_; }
    std::size_t capacity() const { return cap_; }
    T* begin() { return data_; }
    T* end() { return data_ + size_; }

private:
    T* data_ = nullptr;
    std::size_t size_ = 0, cap_ = 0;
};
```

Key techniques: memory is allocated **uninitialised** with `::operator new` and objects are constructed in it with `std::construct_at` (placement construction) — capacity beyond `size()` holds no objects, exactly like `std::vector`. `std::move_if_noexcept` gives the strong exception guarantee: if copying throws mid-reallocation, the partially built new buffer is destroyed and the old one is untouched. `std::exchange` makes the move constructor a one-liner per member.

```cpp
// file: dynarray/test.cpp
#include <cassert>
#include <iostream>
#include <string>
#include "dynarray.hpp"

struct Counted {
    static inline int copies = 0, moves = 0, alive = 0;
    std::string s;
    explicit Counted(std::string v) : s{std::move(v)} { ++alive; }
    Counted(const Counted& o) : s{o.s} { ++copies; ++alive; }
    Counted(Counted&& o) noexcept : s{std::move(o.s)} { ++moves; ++alive; }
    ~Counted() { --alive; }
};

struct ThrowingMove {                       // move not noexcept: DynArray must copy on growth
    static inline int copies = 0, moves = 0;
    int v;
    explicit ThrowingMove(int x) : v{x} {}
    ThrowingMove(const ThrowingMove& o) : v{o.v} { ++copies; }
    ThrowingMove(ThrowingMove&& o) : v{o.v} { ++moves; }
};

int main() {
    {
        DynArray<Counted> a;
        for (int i = 0; i < 100; ++i)
            a.emplace_back("item " + std::to_string(i));       // constructed in place
        std::cout << "100 emplace_backs: copies=" << Counted::copies << " moves=" << Counted::moves
                  << " capacity=" << a.capacity() << '\n';
        assert(Counted::copies == 0);

        DynArray<Counted> b = a;                                 // deep copy
        assert(Counted::copies == 100 && b[42].s == "item 42");
        DynArray<Counted> c = std::move(a);                      // steals: no element touched
        assert(Counted::copies == 100 && a.size() == 0 && c.size() == 100);
        b = c;                                                   // copy-assign
        c = DynArray<Counted>{};                                 // move-assign an empty one
        assert(c.size() == 0 && b.size() == 100);
        assert(Counted::alive == 100);                           // only b's elements are alive
    }
    assert(Counted::alive == 0);                                 // nothing leaked

    DynArray<ThrowingMove> t;
    for (int i = 0; i < 100; ++i)
        t.emplace_back(i);
    std::cout << "throwing-move type, 100 emplace_backs: copies=" << ThrowingMove::copies
              << " moves=" << ThrowingMove::moves << '\n';
    assert(ThrowingMove::moves == 0);

    bool threw = false;
    try { t.at(100); } catch (const std::out_of_range&) { threw = true; }
    assert(threw);
    std::cout << "all DynArray tests passed\n";
}
```

```console
$ cd dynarray && g++-14 -std=c++20 -Wall -Wextra -pedantic -g -fsanitize=address,undefined test.cpp -o test && ./test
100 emplace_backs: copies=0 moves=124 capacity=128
throwing-move type, 100 emplace_backs: copies=124 moves=0
all DynArray tests passed
```

With a `noexcept` move, growth to 100 elements moved existing elements (124 moves: 4 + 8 + 16 + 32 + 64) and copied none; with a potentially throwing move, the same growth copied all 124 instead. The test checks every special member and proves through the `alive` counter — under AddressSanitizer — that no element is leaked or destroyed twice.

### Review checklist

- Do the special members leave every object (including moved-from ones) destructible?
- Does `reserve` give the strong exception guarantee? (Write a test type whose copy constructor throws on the 50th copy.)
- Is `emplace_back` correct when an argument refers to an element of the same array (`a.emplace_back(a[0])`) during reallocation? (Hint: the standard `vector` handles it; this version does not — fix it by constructing the new element before moving the old ones.)

## Answers to the Chapter 3 prediction questions

**3.1 Q1.** `x`: lvalue. `x + 1`: prvalue. `std::move(x)`: xvalue. `"abc"`: lvalue (string literals are arrays with static storage). `std::string("abc")`: prvalue. `v[0]`: lvalue (returns `int&`). `f()` returning `int&`: lvalue.

**3.4 Q1.** One: `return y;` would try to move, find no move constructor, and copy instead. (Unless NRVO applies — it cannot here, because there are two candidate objects.)

# Chapter 4 — The Standard Library: Containers, Iterators, Algorithms, and Ranges

*Goal: solve everyday problems with the standard library instead of hand-written loops — choosing the right container, using algorithms and C++20 ranges, and knowing exactly when iterators and references stay valid.*

The C++ standard library's containers and algorithms (descended from Alexander Stepanov's STL) are one of the language's great strengths: generic, efficient, and composable. Most loops in professional C++ are calls to algorithms; most data lives in a `std::vector`, `std::unordered_map`, or `std::map`. This chapter covers the containers and their complexity guarantees, iterators and the rules for invalidation (a major source of bugs), the algorithms, and C++20 ranges — then builds a log analyser with them.

**What you will be able to do after this chapter**

- Choose among `vector`, `deque`, `list`, `array`, `map`, `set`, `unordered_map`, and the adaptors by complexity and memory layout.
- Explain iterator categories and the invalidation rules of each container.
- Replace loops with algorithms: searching, sorting, transforming, partitioning, removing.
- Compose lazy pipelines with C++20 ranges and views, using projections.
- Use `optional`, `pair`, `tuple`, and priority queues fluently.

## Topic 4A — Containers

## Unit 4.1 — Sequence containers

**One-line summary:** `std::vector` (contiguous, growable) is the default container: fastest for iteration and appending, cache-friendly, and the only one compatible with C arrays. `std::array` is a fixed-size array. `std::deque` adds fast insertion at the front; `std::list` gives stable element addresses and O(1) splicing at the cost of a heap node per element. Choose anything but `vector` only for a reason you can state.

| Container | Layout | Access by index | Insert/erase at end | at front | in middle | Iterators invalidated by insertion |
| --- | --- | --- | --- | --- | --- | --- |
| `vector` | one contiguous block | O(1) | O(1) amortised | O(n) | O(n) | all, if it reallocates; else those after the point |
| `array<T,N>` | inline, fixed | O(1) | — | — | — | — |
| `deque` | blocks of elements | O(1) | O(1) | O(1) | O(n) | all iterators (references stay valid at the ends) |
| `list` | doubly linked nodes | O(n) | O(1) | O(1) | O(1) given an iterator | none |
| `forward_list` | singly linked nodes | O(n) | — | O(1) | O(1) after an iterator | none |

```cpp
// file: seq.cpp
#include <chrono>
#include <deque>
#include <iostream>
#include <list>
#include <numeric>
#include <vector>

template <typename C>
long long time_sum(const C& c) {
    auto t0 = std::chrono::steady_clock::now();
    long long s = 0;
    for (int rep = 0; rep < 20; ++rep)
        s += std::accumulate(c.begin(), c.end(), 0LL);
    auto t1 = std::chrono::steady_clock::now();
    std::cout << std::chrono::duration_cast<std::chrono::milliseconds>(t1 - t0).count() << " ms ";
    return s;
}

int main() {
    const int n = 2'000'000;
    std::vector<int> v(n, 1);
    std::deque<int> d(n, 1);
    std::list<int> l(n, 1);
    std::cout << "sum 20x over 2M ints: vector "; auto a = time_sum(v);
    std::cout << "| deque "; auto b = time_sum(d);
    std::cout << "| list "; auto c = time_sum(l);
    std::cout << "(checksums " << a << ' ' << b << ' ' << c << ")\n";   // using the results keeps the work

    std::vector<int> w;
    std::cout << "capacity growth:";
    std::size_t last = 0;
    for (int i = 0; i < 100; ++i) {
        w.push_back(i);
        if (w.capacity() != last) { last = w.capacity(); std::cout << ' ' << last; }
    }
    std::cout << '\n';
    w.reserve(1000);                           // pre-allocate when the size is known
    w.shrink_to_fit();                         // non-binding request to release spare capacity
    std::cout << "after reserve(1000) and shrink_to_fit: capacity " << w.capacity() << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall seq.cpp -o seq && ./seq
sum 20x over 2M ints: vector 21 ms | deque 20 ms | list 176 ms (checksums 40000000 40000000 40000000)
capacity growth: 1 2 4 8 16 32 64 128
after reserve(1000) and shrink_to_fit: capacity 100
```

Iterating a `list` is many times slower than a `vector` holding the same values: every element is a separate heap node reached through a pointer (the same lesson as Unit 11.4 of the C book). libstdc++'s `vector` doubles its capacity when full, giving amortised O(1) `push_back`.

<!-- expanded -->
### Going deeper: how `std::vector` grows, and `reserve`

`push_back` on a full vector allocates a bigger buffer, moves every element into it, and frees the old one. To keep `push_back` cheap *on average* (amortised constant time), the capacity grows by a constant factor: libstdc++ doubles it, MSVC multiplies by 1.5. When you know the final size, `reserve` allocates once.

```cpp
// file: growth.cpp
#include <cstdio>
#include <vector>

int main() {
    std::vector<int> v;
    std::size_t last = v.capacity();
    int reallocations = 0;
    std::printf("capacity after push_back: ");
    for (int i = 0; i < 1000; ++i) {
        v.push_back(i);
        if (v.capacity() != last) { last = v.capacity(); ++reallocations; std::printf("%zu ", last); }
    }
    std::printf("\nreallocations for 1000 elements: %d\n", reallocations);

    std::vector<int> w;
    w.reserve(1000);
    const int* before = w.data();
    for (int i = 0; i < 1000; ++i) w.push_back(i);
    std::printf("with reserve(1000): buffer moved? %s\n", w.data() == before ? "no" : "yes");

    v.clear();
    std::printf("after clear(): size %zu, capacity %zu\n", v.size(), v.capacity());
    v.shrink_to_fit();
    std::printf("after shrink_to_fit(): capacity %zu\n", v.capacity());
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra growth.cpp -o growth && ./growth
capacity after push_back: 1 2 4 8 16 32 64 128 256 512 1024 
reallocations for 1000 elements: 11
with reserve(1000): buffer moved? no
after clear(): size 0, capacity 1024
after shrink_to_fit(): capacity 0
```

Each reallocation invalidates every pointer, reference, and iterator into the vector (Unit 4.3). `clear()` keeps the memory, which is what you want for a buffer reused in a loop; `shrink_to_fit()` is a non-binding request to release it.

### Deep & Tricky

**DT-17. `std::vector<bool>` is not a container of `bool`s: it packs bits, so `v[i]` returns a proxy object, not a `bool&`. `auto x = v[0];` gives a proxy that still refers into the vector, `bool* p = &v[0];` does not compile, and two threads writing different elements race on the same byte.** *(Guarantee: [vector.bool] — an explicitly specialised, space-optimised partial specialisation)* When you need real `bool`s, use `std::vector<char>`, `std::deque<bool>`, or `std::vector<std::uint8_t>`; for a fixed-size set of flags, `std::bitset`.

```cpp
// file: vecbool.cpp
#include <iostream>
#include <type_traits>
#include <vector>

int main() {
    std::vector<bool> flags(8, false);
    auto first = flags[0];                       // a proxy, not a copy of the bool
    flags[0] = true;
    std::cout << std::boolalpha << "auto copy sees the later change: " << bool(first) << '\n';
    std::cout << "flags[0] is a bool&: " << std::is_same_v<decltype(flags[0]), bool&> << '\n';
    std::cout << "sizeof(vector<bool>) " << sizeof(flags) << ", and 8 flags fit in one byte of storage\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra vecbool.cpp -o vecbool && ./vecbool
auto copy sees the later change: true
flags[0] is a bool&: false
sizeof(vector<bool>) 40, and 8 flags fit in one byte of storage
```

### Mastery check

1. **Explain.** Why does inserting into a `vector` invalidate *all* iterators when it reallocates?

## Unit 4.2 — Associative containers

**One-line summary:** `std::map` and `std::set` are sorted, balanced binary trees (red-black trees): O(log n) operations and ordered iteration. `std::unordered_map` and `std::unordered_set` are hash tables: O(1) average operations, no order. Use the unordered versions for lookups unless you need ordering, range queries, or worst-case guarantees.

```cpp
// file: assoc.cpp
#include <iostream>
#include <map>
#include <set>
#include <string>
#include <unordered_map>

struct Point {
    int x, y;
    bool operator==(const Point&) const = default;
};

struct PointHash {                                   // a hash for a user-defined key
    std::size_t operator()(const Point& p) const noexcept {
        std::size_t h = std::hash<int>{}(p.x);
        return h ^ (std::hash<int>{}(p.y) + 0x9e3779b97f4a7c15ULL + (h << 6) + (h >> 2));   // hash_combine
    }
};

int main() {
    std::map<std::string, int> stock{{"pear", 3}, {"apple", 5}};
    stock["banana"] += 7;                             // operator[] inserts a default value if absent
    stock.insert({"apple", 100});                     // insert does NOT overwrite
    stock.insert_or_assign("pear", 9);                // this does
    if (auto [it, inserted] = stock.try_emplace("kiwi", 2); inserted)
        std::cout << "added " << it->first << '\n';
    for (const auto& [fruit, n] : stock)              // sorted by key
        std::cout << fruit << '=' << n << ' ';
    std::cout << '\n';

    if (stock.contains("plum") == false)              // C++20; count() before that
        std::cout << "no plums; lookups with find() don't insert: size still " << stock.size() << '\n';

    auto lo = stock.lower_bound("b"), hi = stock.upper_bound("k");   // range query: keys in [b, k]
    std::cout << "keys from b to k:";
    for (auto it = lo; it != hi; ++it) std::cout << ' ' << it->first;
    std::cout << '\n';

    std::unordered_map<Point, std::string, PointHash> grid{{{0, 0}, "origin"}, {{3, 4}, "target"}};
    std::cout << "grid[3,4] = " << grid.at({3, 4}) << ", buckets " << grid.bucket_count()
              << ", load factor " << grid.load_factor() << '\n';

    std::set<int> s{5, 1, 4, 1, 3};                   // unique, sorted
    std::multiset<int> ms{5, 1, 4, 1, 3};             // duplicates allowed
    std::cout << "set size " << s.size() << ", multiset size " << ms.size() << ", count(1) " << ms.count(1) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra assoc.cpp -o assoc && ./assoc
added kiwi
apple=5 banana=7 kiwi=2 pear=9 
no plums; lookups with find() don't insert: size still 4
keys from b to k: banana
grid[3,4] = target, buckets 13, load factor 0.153846
set size 4, multiset size 5, count(1) 2
```

### Deep & Tricky

**DT-18. `map[key]` on a missing key inserts it.** *(Library behaviour)* `if (m[k] == 0)` adds `k` with a default value when it is absent — growing the map during what looks like a read, and failing to compile for `const` maps. Use `find`, `contains`, or `at` (which throws) for lookups; use `operator[]` only when inserting a default is what you want (counting is the classic case: `++counts[word]`).

**DT-19. References and pointers to `unordered_map` elements survive rehashing; iterators do not.** *(Library guarantee)* Node-based containers (`map`, `set`, `unordered_*`, `list`) never move their elements, so `&m[k]` stays valid across inserts. But a rehash in an unordered container invalidates its iterators. Keep references, not iterators, across insertions into unordered containers.

<!-- expanded -->
### Going deeper: `try_emplace`, `insert_or_assign`, and node extraction

C++17 added map operations that say exactly what you mean and avoid hidden work. `try_emplace(k, args...)` constructs the value only if `k` is absent (and leaves the arguments untouched if it is present — `emplace` may consume a moved argument even when it inserts nothing). `insert_or_assign(k, v)` overwrites without requiring a default constructor, unlike `m[k] = v`. `extract(k)` unlinks a node and hands it to you, so a key can be changed — or a node moved to another map — without copying or reallocating.

```cpp
// file: mapops.cpp
#include <iostream>
#include <map>
#include <memory>
#include <string>

int main() {
    std::map<std::string, std::unique_ptr<int>> m;
    auto p = std::make_unique<int>(1);
    auto [it, inserted] = m.try_emplace("a", std::move(p));
    auto q = std::make_unique<int>(2);
    auto [it2, inserted2] = m.try_emplace("a", std::move(q));   // "a" exists: q is NOT moved from
    std::cout << std::boolalpha << "first insert " << inserted << ", second insert " << inserted2
              << ", q still owns its int: " << (q != nullptr) << '\n';

    m.insert_or_assign("a", std::make_unique<int>(10));
    std::cout << "a = " << *m["a"] << '\n';

    std::map<std::string, int> ages{{"alice", 30}, {"bob", 25}};
    auto node = ages.extract("bob");
    node.key() = "robert";                                      // change a key without copying the value
    ages.insert(std::move(node));
    for (auto& [name, age] : ages) std::cout << name << ' ' << age << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra mapops.cpp -o mapops && ./mapops
first insert true, second insert false, q still owns its int: true
a = 10
alice 30
robert 25
```

### Mastery check

1. **Write.** Count word frequencies from standard input with an `unordered_map<std::string, int>` and print the ten most frequent (Unit 4.4 shows `partial_sort`).

## Topic 4B — Iterators and algorithms

## Unit 4.3 — Iterators and invalidation

**One-line summary:** Iterators generalise pointers: `*it` accesses an element, `++it` advances, `it1 == it2` compares. Their **category** (input, forward, bidirectional, random access, contiguous) determines which algorithms can use them. Using an iterator, pointer, or reference after an operation that **invalidates** it is undefined behaviour — the most common container bug in C++.

```cpp
// file: invalidation.cpp
#include <iostream>
#include <vector>

int main() {
    std::cout << std::unitbuf;         // flush after every output: the program aborts below
    std::vector<int> v{1, 2, 3};
    int& first = v[0];                 // reference into the vector's buffer
    auto it = v.begin();
    std::cout << "capacity before " << v.capacity() << '\n';
    v.push_back(4);                    // capacity 3 -> 6: reallocation moves every element
    std::cout << "capacity after " << v.capacity() << '\n';
    std::cout << first << ' ' << *it << '\n';   // both dangle: reading freed memory
}
```

```console
$ g++-14 -std=c++20 -g -fsanitize=address invalidation.cpp -o invalidation && ./invalidation 2>&1 | grep -E 'capacity|ERROR' | head -n 3
capacity before 3
capacity after 6
==8782==ERROR: AddressSanitizer: heap-use-after-free on address 0x502000000010 at pc 0x5565161ef73a bp 0x7fff7bb9a640 sp 0x7fff7bb9a630
```

The classic loop-erase bug has the same cause:

```cpp
// file: erase.cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5, 6};
    for (auto it = v.begin(); it != v.end();) {
        if (*it % 2 == 0)
            it = v.erase(it);          // erase returns the next valid iterator: use it
        else
            ++it;
    }
    std::vector<int> w{1, 2, 3, 4, 5, 6};
    std::erase_if(w, [](int x) { return x % 2 == 0; });   // C++20: one line, O(n)
    for (int x : v) std::cout << x << ' ';
    std::cout << "| ";
    for (int x : w) std::cout << x << ' ';
    std::cout << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra erase.cpp -o erase && ./erase
1 3 5 | 1 3 5 
```

The manual loop is correct only because it uses the iterator `erase` returns; writing `v.erase(it); ++it;` would increment an invalidated iterator. It is also O(n²) for many erasures, since each `erase` shifts the tail. `std::erase_if` (or the older erase–remove idiom: `v.erase(std::remove_if(…), v.end())`) is O(n) and has no iterator to get wrong.

### Mastery check

1. **Predict.** Is `for (int x : v) if (x == 3) v.push_back(4);` safe? Why or why not?

## Unit 4.4 — Algorithms

**One-line summary:** `<algorithm>` and `<numeric>` provide over a hundred tested, optimised algorithms that work on iterator ranges: search (`find`, `find_if`, `binary_search`, `lower_bound`), sort (`sort`, `stable_sort`, `partial_sort`, `nth_element`), transform (`transform`, `accumulate`, `reduce`), rearrange (`partition`, `rotate`, `unique`, `reverse`), and more. Prefer them to raw loops: they state intent, and they are hard to get wrong.

```cpp
// file: algos.cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

struct Employee { std::string name; std::string dept; int salary; };

int main() {
    std::vector<Employee> staff{
        {"ada", "eng", 120}, {"bob", "ops", 80}, {"cy", "eng", 95}, {"di", "ops", 80},
        {"ed", "eng", 150}, {"flo", "sales", 70}, {"gus", "sales", 90},
    };

    auto rich = std::count_if(staff.begin(), staff.end(), [](const Employee& e) { return e.salary >= 100; });
    auto total = std::accumulate(staff.begin(), staff.end(), 0, [](int s, const Employee& e) { return s + e.salary; });
    std::cout << rich << " earn >= 100; payroll " << total << '\n';

    // stable_sort keeps the original order among equal keys
    std::stable_sort(staff.begin(), staff.end(), [](const auto& a, const auto& b) { return a.salary > b.salary; });
    for (const auto& e : staff) std::cout << e.name << ':' << e.salary << ' ';
    std::cout << '\n';

    // top 3 by salary without sorting everything: O(n log k)
    std::vector<Employee> top(3, Employee{});
    std::partial_sort_copy(staff.begin(), staff.end(), top.begin(), top.end(),
                           [](const auto& a, const auto& b) { return a.salary > b.salary; });
    std::cout << "top 3: " << top[0].name << ' ' << top[1].name << ' ' << top[2].name << '\n';

    // median salary in O(n): nth_element partially orders around the middle
    std::vector<int> sal;
    std::transform(staff.begin(), staff.end(), std::back_inserter(sal), [](const auto& e) { return e.salary; });
    std::nth_element(sal.begin(), sal.begin() + sal.size() / 2, sal.end());
    std::cout << "median " << sal[sal.size() / 2] << '\n';

    // partition: engineers first
    auto mid = std::stable_partition(staff.begin(), staff.end(), [](const auto& e) { return e.dept == "eng"; });
    std::cout << "engineers: " << (mid - staff.begin()) << ", first non-engineer: " << mid->name << '\n';

    // binary search on sorted data
    std::vector<int> sorted{2, 3, 5, 7, 11, 13, 17};
    auto lb = std::lower_bound(sorted.begin(), sorted.end(), 10);
    std::cout << "first prime >= 10: " << *lb << ", is 13 present: " << std::binary_search(sorted.begin(), sorted.end(), 13) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra algos.cpp -o algos && ./algos
2 earn >= 100; payroll 685
ed:150 ada:120 cy:95 gus:90 bob:80 di:80 flo:70 
top 3: ed ada cy
median 90
engineers: 3, first non-engineer: gus
first prime >= 10: 11, is 13 present: 1
```

### Requirements algorithms place on you

- `sort` needs random-access iterators (not `list` — use `list::sort`) and a comparator that is a **strict weak ordering**: irreflexive (`!(a < a)`), transitive, with transitive equivalence. Using `<=` breaks irreflexivity; libstdc++'s debug mode and `-D_GLIBCXX_ASSERTIONS` can catch it, but in release builds it can read out of bounds.
- Binary search algorithms require the range to be sorted by the same ordering.
- Algorithms never change a container's size: `std::remove` only moves the kept elements forward and returns the new logical end — hence erase–remove or `std::erase_if`.

### Deep & Tricky

**DT-20. An invalid comparator can make `std::sort` crash, not just mis-sort.** *(Guarantee: UB on violated preconditions; implementation behaviour)* Introsort's partitioning loops rely on the comparator to stop at sentinels; a comparator like `[](int a, int b) { return a <= b; }` lets them run off the ends of the array. Real-world crashes in sorting usually trace back to comparators that are not strict weak orderings — including ones that compare floating-point values containing NaN.

<!-- expanded -->
### Going deeper: algorithms that need sorted input

Several algorithms have the precondition "the range is sorted (by the same comparison)": `binary_search`, `lower_bound`, `upper_bound`, `equal_range`, `merge`, `set_intersection`, `set_difference`, `includes`, `unique` (which only removes *adjacent* duplicates). Called on unsorted data they do not fail; they return wrong answers.

```cpp
// file: sortedalgos.cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> v{5, 1, 4, 1, 5, 9, 2, 6};
    std::cout << std::boolalpha << "binary_search(2) on unsorted data: " << std::binary_search(v.begin(), v.end(), 2) << '\n';
    std::vector<int> u = v;
    u.erase(std::unique(u.begin(), u.end()), u.end());
    std::cout << "unique on unsorted data leaves " << u.size() << " of 8 elements\n";

    std::sort(v.begin(), v.end());
    std::cout << "binary_search(2) after sort: " << std::binary_search(v.begin(), v.end(), 2) << '\n';
    v.erase(std::unique(v.begin(), v.end()), v.end());
    std::cout << "unique after sort: ";
    std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " "));
    auto [lo, hi] = std::equal_range(v.begin(), v.end(), 5);
    std::cout << "\nposition of 5: " << (lo - v.begin()) << ", count " << (hi - lo) << '\n';

    std::vector<int> a{1, 3, 5, 7, 9}, b{3, 4, 5, 6, 7}, both;
    std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(both));
    std::cout << "intersection: ";
    for (int x : both) std::cout << x << ' ';
    std::cout << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra sortedalgos.cpp -o sortedalgos && ./sortedalgos
binary_search(2) on unsorted data: false
unique on unsorted data leaves 8 of 8 elements
binary_search(2) after sort: true
unique after sort: 1 2 4 5 6 9 
position of 5: 3, count 1
intersection: 3 5 7 
$ g++-14 -std=c++20 -D_GLIBCXX_DEBUG sortedalgos.cpp -o sortedalgos_dbg && ./sortedalgos_dbg 2>&1 | grep -m1 -oE "Error: .*"
Error: elements in iterator range [first, last) are not partitioned by the 
```

Building with libstdc++'s debug mode (`-D_GLIBCXX_DEBUG`, Unit 13.3) turns the silently wrong `binary_search` (2 is in the vector, yet the search said `false`) into an immediate abort with a message naming the violated precondition.

### Mastery check

1. **Write.** Using algorithms only (no raw loops), check whether a vector of strings has duplicates, and print them.

## Unit 4.5 — Ranges and views (C++20)

**One-line summary:** C++20 **ranges** let algorithms take a whole container (`std::ranges::sort(v)`) with optional **projections** (`sort(v, {}, &Employee::salary)`), and **views** build lazy pipelines (`v | views::filter(…) | views::transform(…) | views::take(5)`) that compute elements on demand without intermediate containers.

```cpp
// file: ranges.cpp
#include <algorithm>
#include <iostream>
#include <ranges>
#include <string>
#include <vector>

struct Employee { std::string name; int salary; };

int main() {
    std::vector<Employee> staff{{"ada", 120}, {"bob", 80}, {"cy", 95}, {"ed", 150}, {"flo", 70}};

    std::ranges::sort(staff, std::ranges::greater{}, &Employee::salary);   // projection: sort by salary
    for (const auto& e : staff) std::cout << e.name << ' ';
    std::cout << '\n';

    auto it = std::ranges::find(staff, "cy", &Employee::name);              // project, then compare
    std::cout << "cy earns " << it->salary << '\n';

    namespace vw = std::views;
    auto squares_of_odds = vw::iota(1)                         // 1, 2, 3, ... (infinite, lazy)
                         | vw::filter([](int n) { return n % 2 == 1; })
                         | vw::transform([](int n) { return n * n; })
                         | vw::take(5);
    for (int x : squares_of_odds) std::cout << x << ' ';      // computed only as needed
    std::cout << '\n';

    for (const auto& name : staff | vw::transform(&Employee::name) | vw::reverse)
        std::cout << name << ' ';
    std::cout << '\n';

    std::string csv = "red,green,,blue";
    for (auto part : csv | vw::split(','))                     // lazy split
        std::cout << '[' << std::string_view(part.begin(), part.end()) << ']';
    std::cout << '\n';

    for (auto [i, e] : vw::enumerate(staff) | vw::take(2))     // C++23 enumerate
        std::cout << i << ':' << e.name << ' ';
    std::cout << '\n';
}
```

```console
$ g++-14 -std=c++23 -Wall -Wextra ranges.cpp -o ranges && ./ranges
ed ada cy bob flo 
cy earns 95
1 9 25 49 81 
flo bob cy ada ed 
[red][green][][blue]
0:ed 1:ada 
```

Views are **lazy** and **non-owning**: `squares_of_odds` is a small object describing a computation over an infinite sequence; iterating it does exactly the work needed for five results. Because views refer to their underlying range, the same lifetime rules as `string_view` apply: a view of a temporary container dangles. The ranges algorithms protect against the simplest case — calling `std::ranges::find` on a temporary vector returns `std::ranges::dangling`, a type you cannot dereference, so the bug is a compile error.

<!-- expanded -->
### Deep & Tricky

**DT-21. Some views cache their `begin()` (for example `filter_view`, so that repeated iteration stays cheap). Iterating a cached view, then modifying the underlying container, then iterating the view again gives results based on the stale cached position — and a view that caches is not usable through a `const&`.** *(Guarantee: [range.filter.view] — `begin()` is amortised constant time and must cache; modifying elements so that the predicate result changes is undefined behaviour)* Build views where you use them, iterate them once, and do not keep them across modifications of what they look at.

```cpp
// file: viewcache.cpp
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5, 6};
    auto even = v | std::views::filter([](int x) { return x % 2 == 0; });
    std::cout << "first pass: ";
    for (int x : even) std::cout << x << ' ';
    v[0] = 100;                                   // now the first even element is at index 0
    std::cout << "\nsecond pass (cached begin skips the new 100): ";
    for (int x : even) std::cout << x << ' ';
    auto fresh = v | std::views::filter([](int x) { return x % 2 == 0; });
    std::cout << "\nfresh view: ";
    for (int x : fresh) std::cout << x << ' ';
    std::cout << '\n';
    // void print(const auto& r); print(even);   // error: filter_view has no const begin()
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra viewcache.cpp -o viewcache && ./viewcache
first pass: 2 4 6 
second pass (cached begin skips the new 100): 2 4 6 
fresh view: 100 2 4 6 
```

### Mastery check

1. **Write.** A pipeline that reads integers from `std::views::istream<int>(std::cin)`, keeps the positive ones, and prints the first 10 squared.

## Unit 4.6 — Adaptors and vocabulary types

**One-line summary:** `std::stack`, `std::queue`, and `std::priority_queue` adapt a container to a restricted interface; `std::pair`, `std::tuple`, and `std::optional` are **vocabulary types** — standard ways to return two values, several values, or maybe-a-value.

```cpp
// file: adaptors.cpp
#include <iostream>
#include <optional>
#include <queue>
#include <string>
#include <vector>

struct Task { int priority; std::string name; };

std::optional<int> find_index(const std::vector<std::string>& v, const std::string& key) {
    for (std::size_t i = 0; i < v.size(); ++i)
        if (v[i] == key) return static_cast<int>(i);
    return std::nullopt;
}

int main() {
    auto cmp = [](const Task& a, const Task& b) { return a.priority < b.priority; };   // max-heap
    std::priority_queue<Task, std::vector<Task>, decltype(cmp)> pq{cmp};
    pq.push({2, "write tests"});
    pq.push({5, "fix outage"});
    pq.push({1, "refactor"});
    while (!pq.empty()) {
        std::cout << pq.top().priority << ' ' << pq.top().name << '\n';
        pq.pop();
    }

    std::vector<std::string> names{"ada", "alan", "grace"};
    if (auto i = find_index(names, "alan")) std::cout << "alan at " << *i << '\n';
    std::cout << "bob at " << find_index(names, "bob").value_or(-1) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra adaptors.cpp -o adaptors && ./adaptors
5 fix outage
2 write tests
1 refactor
alan at 1
bob at -1
```

`std::optional<T>` stores the `T` inline (no heap allocation) plus a flag. Test it with `if (opt)`, read it with `*opt` (undefined if empty) or `opt.value()` (throws if empty) or `value_or(default)`. Chapter 6 introduces `std::expected`, which carries an error instead of just "nothing"; Chapter 9 covers `std::variant`.

<!-- expanded -->
### Going deeper: `std::span` and `std::array`

`std::array<T, N>` is a C array that behaves like a value: it knows its size, it can be copied and returned, and `at()` checks bounds. `std::span<T>` (C++20) is a *non-owning* view over contiguous elements — any vector, array, or C array — the C++ equivalent of passing a pointer and a length together, and it is what a function should take when it only reads or writes a run of elements and does not care who owns them.

```cpp
// file: span.cpp
#include <array>
#include <iostream>
#include <numeric>
#include <span>
#include <vector>

double average(std::span<const int> xs) {
    return xs.empty() ? 0.0 : std::accumulate(xs.begin(), xs.end(), 0.0) / static_cast<double>(xs.size());
}
void zero_first(std::span<int> xs, std::size_t n) {
    for (int& x : xs.first(std::min(n, xs.size()))) x = 0;
}

int main() {
    std::vector<int> v{2, 4, 6, 8};
    std::array<int, 3> a{1, 2, 3};
    int c[] = {10, 20, 30, 40, 50};
    std::cout << average(v) << ' ' << average(a) << ' ' << average(c) << ' '
              << average(std::span(c).subspan(1, 3)) << '\n';
    zero_first(c, 2);
    std::cout << c[0] << ' ' << c[1] << ' ' << c[2] << '\n';
    std::array<int, 3> b = a;                       // arrays copy like values
    b[0] = 99;
    std::cout << "a[0] " << a[0] << ", b[0] " << b[0] << ", sizeof(span<int>) " << sizeof(std::span<int>) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra span.cpp -o span && ./span
5 2 30 30
0 0 30
a[0] 1, b[0] 99, sizeof(span<int>) 16
```

A `span` is two words (pointer and length), so pass it by value. Like `string_view`, it dangles if the container it views is destroyed or reallocates.

<!-- expanded -->
## Unit 4.7 — Files and directories: `<filesystem>` and file streams

**One-line summary:** `std::filesystem::path` is a portable file path type, and `<filesystem>` lists directories, checks and changes files, and reports errors either by throwing `filesystem_error` or through an `std::error_code` argument; `std::ifstream` and `std::ofstream` read and write the files themselves.

Before C++17, listing a directory needed POSIX `opendir` or Windows `FindFirstFile`. `<filesystem>` (based on Boost.Filesystem) does it portably. The key types: `path` (joins with `/`, splits into `parent_path()`, `filename()`, `stem()`, `extension()`), `directory_entry`, and the iterators `directory_iterator` (one level) and `recursive_directory_iterator` (the whole tree).

```cpp
// file: fsdemo.cpp
#include <filesystem>
#include <fstream>
#include <iostream>
#include <map>
#include <string>
#include <system_error>

namespace fs = std::filesystem;

int main() {
    fs::path root = fs::temp_directory_path() / "fsdemo";
    fs::remove_all(root);                                          // start clean; no error if missing
    fs::create_directories(root / "src" / "util");
    for (auto name : {"src/main.cpp", "src/util/str.cpp", "src/util/str.hpp", "README.md"}) {
        std::ofstream(root / name) << "// " << name << '\n';      // a temporary stream: written and closed
    }

    fs::path p = root / "src" / "util" / "str.cpp";
    std::cout << "filename " << p.filename() << ", stem " << p.stem() << ", extension " << p.extension()
              << ", parent ends in " << p.parent_path().filename() << '\n';

    std::map<std::string, int> by_ext;
    std::uintmax_t bytes = 0;
    for (const auto& e : fs::recursive_directory_iterator(root)) {
        if (!e.is_regular_file()) continue;
        ++by_ext[e.path().extension().string()];
        bytes += e.file_size();
    }
    for (const auto& [ext, n] : by_ext) std::cout << ext << ": " << n << '\n';
    std::cout << "total bytes " << bytes << '\n';

    std::ifstream in(root / "README.md");
    std::string line;
    std::getline(in, line);
    std::cout << "README first line: " << line << '\n';

    std::error_code ec;                                            // the non-throwing overloads
    auto size = fs::file_size(root / "missing.txt", ec);
    std::cout << "missing.txt: " << (ec ? ec.message() : std::to_string(size)) << '\n';
    try {
        (void)fs::file_size(root / "missing.txt");                 // the throwing overload
    } catch (const fs::filesystem_error& e) {
        std::cout << std::boolalpha << "filesystem_error mentions the path: " << (std::string(e.what()).find("missing.txt") != std::string::npos) << '\n';
    }
    std::cout << "removed " << fs::remove_all(root) << " entries\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra fsdemo.cpp -o fsdemo && ./fsdemo
filename "str.cpp", stem "str", extension ".cpp", parent ends in "util"
.cpp: 2
.hpp: 1
.md: 1
total bytes 69
README first line: // README.md
missing.txt: No such file or directory
filesystem_error mentions the path: true
removed 7 entries
```

Line by line, the parts that matter: `operator/` joins path parts with the right separator for the platform; `std::ofstream(path) << ...` creates a temporary stream that is flushed and closed at the end of the statement; `recursive_directory_iterator` visits every entry under `root`, in an unspecified order (so the program counts into a `std::map` rather than printing in visiting order); and every operation that can fail comes in two forms — one throws `fs::filesystem_error`, and one takes a `std::error_code&` and never throws.

### File streams: what to check

A stream does not throw by default when opening or reading fails; it sets a state flag. `if (!in)` after opening, and `while (std::getline(in, line))` for reading, are the idioms. Opening with `std::ios::binary` stops Windows from translating line endings; on Linux and macOS it changes nothing, but portable code that reads raw bytes always sets it.

### Deep & Tricky

**DT-22. Checking and then acting on a file is a race: between `fs::exists(p)` and `std::ofstream(p)`, another process can create, delete, or replace `p` (a *time-of-check to time-of-use* bug, TOCTOU). Do the operation and handle its failure instead of checking first; for "create only if absent", `std::fstream` has no exclusive mode before C++23's `std::ios::noreplace`, so use POSIX `open(O_CREAT | O_EXCL)` or C11 `fopen(p, "wx")`.** *(Library behaviour: [fs.race.behavior] — the standard explicitly says file system races are the program's problem and may cause undefined behaviour)* The same applies to `fs::remove_all` on a directory that someone else can write to: it follows the tree as it finds it.

### Mastery check

1. **Write.** A program `du_ext DIR` that prints, for each file extension under `DIR`, the total size in bytes, largest first. Use the `error_code` overloads so that an unreadable subdirectory is skipped (`directory_options::skip_permission_denied`) instead of ending the program.
2. **Explain.** Why does `fsdemo.cpp` count into a `std::map` instead of printing entries as it finds them?
3. **Predict.** What does `fs::path("/a/b/") / "c"` give? And `fs::path("/a/b") / "/c"`? (Try it: an absolute right-hand side *replaces* the left.)

## Unit 4.8 — Text and time: `<regex>`, `<chrono>`, and `<random>`

**One-line summary:** `std::regex` matches and searches with regular expressions (correct but slow — compile each pattern once); `<chrono>` represents durations, clocks, and, since C++20, calendar dates and time zones with type safety; `<random>` separates the random *engine* from the *distribution* that shapes its output.

### Regular expressions

```cpp
// file: regexdemo.cpp
#include <iostream>
#include <regex>
#include <string>

int main() {
    const std::regex line_re(R"((\d{4})-(\d{2})-(\d{2}) (ERROR|WARN|INFO) (.*))");   // compiled once
    for (std::string line : {"2026-11-16 ERROR disk full", "garbage", "2026-11-16 INFO started"}) {
        std::smatch m;
        if (std::regex_match(line, m, line_re))
            std::cout << "year " << m[1] << ", level " << m[4] << ", message '" << m[5] << "'\n";
        else
            std::cout << "no match: " << line << '\n';
    }
    std::string text = "mail ada@example.org or alan@example.net today";
    const std::regex email(R"([\w.]+@[\w.]+)");
    for (std::sregex_iterator it(text.begin(), text.end(), email), end; it != end; ++it)
        std::cout << "found " << it->str() << " at " << it->position() << '\n';
    std::cout << std::regex_replace(text, email, "<hidden>") << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra regexdemo.cpp -o regexdemo && ./regexdemo
year 2026, level ERROR, message 'disk full'
no match: garbage
year 2026, level INFO, message 'started'
found ada@example.org at 5
found alan@example.net at 24
mail <hidden> or <hidden> today
```

`regex_match` must match the *whole* string; `regex_search` finds a match anywhere; `sregex_iterator` walks all matches. Raw string literals (`R"(...)"`) save you from doubling every backslash.

### Durations, clocks, and dates

```cpp
// file: chronodemo.cpp
#include <chrono>
#include <iostream>
#include <thread>

using namespace std::chrono_literals;

int main() {
    std::chrono::milliseconds timeout = 2s + 500ms;           // converted exactly: 2500ms
    auto minutes = std::chrono::duration_cast<std::chrono::minutes>(90s);   // truncates: lossy, so explicit
    std::cout << timeout << ", 90s is " << minutes << " when truncated\n";
    // std::chrono::seconds s = 1500ms;                        // error: would lose precision implicitly

    auto t0 = std::chrono::steady_clock::now();               // steady: for measuring intervals
    std::this_thread::sleep_for(20ms);
    auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now() - t0);
    std::cout << "slept at least 20ms: " << std::boolalpha << (elapsed >= 20ms) << '\n';

    using namespace std::chrono;
    year_month_day d = 2024y / February / 29;                  // C++20 calendar types
    std::cout << d << " valid: " << d.ok() << ", 2023-02-29 valid: " << (2023y / February / 29).ok() << '\n';
    sys_days next = sys_days(d) + days(1);
    std::cout << "day after: " << year_month_day(next) << ", weekday " << weekday(next) << '\n';
    std::cout << "days from 2024-01-01 to 2026-11-16: " << (sys_days(2026y / November / 16) - sys_days(2024y / January / 1)).count() << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra chronodemo.cpp -o chronodemo && ./chronodemo
2500ms, 90s is 1min when truncated
slept at least 20ms: true
2024-02-29 valid: true, 2023-02-29 valid: false
day after: 2024-03-01, weekday Fri
days from 2024-01-01 to 2026-11-16: 1050
```

Durations carry their unit in the type, so `2s + 500ms` is exactly `2500ms` and a conversion that would lose precision does not compile without `duration_cast` — the class of bug where one function takes seconds and its caller passes milliseconds disappears. Use `steady_clock` for timing (it never jumps) and `system_clock` for wall-clock time (it can be adjusted).

### Random numbers

```cpp
// file: randomdemo.cpp
#include <iostream>
#include <map>
#include <random>

int main() {
    std::mt19937 engine(42);                                  // fixed seed: the same sequence every run
    std::uniform_int_distribution<int> die(1, 6);
    std::normal_distribution<double> height(170.0, 10.0);
    std::map<int, int> counts;
    for (int i = 0; i < 60000; ++i) ++counts[die(engine)];
    for (auto [face, n] : counts) std::cout << face << ": " << n << '\n';
    double sum = 0;
    for (int i = 0; i < 10000; ++i) sum += height(engine);
    std::cout << "average height " << sum / 10000 << '\n';
    std::random_device rd;                                    // non-deterministic seed source
    std::mt19937 seeded(rd());
    std::cout << "a seeded die roll is between 1 and 6: " << std::boolalpha << (die(seeded) >= 1) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra randomdemo.cpp -o randomdemo && ./randomdemo
1: 9890
2: 10037
3: 9991
4: 10075
5: 10096
6: 9911
average height 169.958
a seeded die roll is between 1 and 6: true
```

The engine (`mt19937`, the Mersenne Twister) produces raw bits; the distribution turns them into the numbers you want, without the bias of `rand() % 6`. With a fixed seed, the engine's sequence is specified exactly by the standard — but the *distributions* are not, so the same seed gives the same die rolls on GCC and different ones on MSVC. Tests that need identical numbers across platforms must implement their own distribution.

### Deep & Tricky

**DT-23. `std::regex` in libstdc++ uses a recursive backtracking matcher: a long input against a pattern with nested repetition can take exponential time or overflow the stack and crash. It is also several times slower than RE2, PCRE2, or a hand-written parser even on simple patterns.** *(Implementation behaviour: libstdc++ bug 86164 and others; the standard does not specify complexity)* Never run `std::regex` on untrusted input of unbounded length; for anything performance-sensitive, use a dedicated library or the CTRE (compile-time regular expressions) library.

```cpp
// file: regexdepth.cpp
#include <iostream>
#include <regex>
#include <string>

int main(int argc, char** argv) {
    std::size_t n = argc > 1 ? std::stoul(argv[1]) : 1000;
    std::string input(n, 'a');
    std::cout << "length " << n << ": " << std::flush;
    bool ok = std::regex_match(input, std::regex("(a|b)*"));
    std::cout << (ok ? "matched" : "no match") << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 regexdepth.cpp -o regexdepth && ./regexdepth 1000 && (ulimit -s 8192; ./regexdepth 200000); echo "exit status $?"
length 1000: matched
length 200000: bash: line 1:  8867 Segmentation fault      ( ulimit -s 8192; ./regexdepth 200000 )
exit status 139
```

A trivial pattern on a 200,000-character string overflowed the 8 MiB stack (status 139 is 128 + `SIGSEGV`), because each repetition of the group adds stack frames.

### Mastery check

1. **Write.** Using `<regex>`, parse lines of the form `key = value` (spaces optional, `#` comments ignored) into a `std::map<std::string, std::string>`. Compile the pattern once.
2. **Explain.** Why does `std::chrono::seconds s = 1500ms;` fail to compile while `std::chrono::milliseconds m = 2s;` compiles?
3. **Predict.** Two programs on two different compilers seed `std::mt19937` with 7 and print `engine()` five times. Same output? Now they print `std::uniform_int_distribution<int>(1, 100)(engine)` five times. Same output?

## Chapter 4 Project — An access-log analyser

**Goal:** analyse a web server access log — request counts per path and status, the top paths, latency percentiles, and traffic per hour — using containers, algorithms, and ranges instead of hand-written loops; and measure `std::map` against `std::unordered_map` for the counting step.

### Specification

Input lines: `2026-11-15T10:23:45Z GET /api/users 200 12.5` (timestamp, method, path, status, latency in ms). Output: total requests; count by status class (2xx, 3xx, 4xx, 5xx); top 5 paths by count; p50, p90, p99 latency; requests per hour; malformed line count.

### Reference solution

```cpp
// file: logstat/gen.cpp
// Generates a reproducible synthetic access log.
#include <cstdio>
#include <random>

int main(int argc, char* argv[]) {
    int n = argc > 1 ? std::atoi(argv[1]) : 100000;
    std::mt19937 rng{2026};
    const char* paths[] = {"/", "/login", "/api/users", "/api/orders", "/static/app.js", "/static/style.css",
                           "/api/search", "/health", "/admin", "/api/orders/42"};
    std::discrete_distribution<int> path_dist{30, 5, 15, 10, 12, 12, 8, 4, 1, 3};    // skewed popularity
    std::discrete_distribution<int> status_dist{90, 3, 5, 2};
    const int statuses[] = {200, 304, 404, 500};
    std::lognormal_distribution<double> latency{2.5, 0.7};                           // long tail
    for (int i = 0; i < n; ++i) {
        long long sec = static_cast<long long>(i) * 86400 / n;   // i * 86400 overflows int
        if (i % 997 == 0) { std::puts("garbage line"); continue; }
        std::printf("2026-11-15T%02lld:%02lld:%02lldZ GET %s %d %.1f\n", sec / 3600, sec / 60 % 60, sec % 60,
                    paths[path_dist(rng)], statuses[status_dist(rng)], latency(rng));
    }
}
```

```cpp
// file: logstat/logstat.cpp
#include <algorithm>
#include <array>
#include <charconv>
#include <chrono>
#include <iostream>
#include <map>
#include <optional>
#include <print>
#include <ranges>
#include <string>
#include <string_view>
#include <unordered_map>
#include <vector>

struct Entry {
    int hour;
    std::string_view path;
    int status;
    double latency_ms;
};

// parse splits on spaces; returns nullopt for malformed lines. Views point into 'line'.
std::optional<Entry> parse(std::string_view line) {
    std::array<std::string_view, 5> f;
    std::size_t n = 0;
    for (auto part : line | std::views::split(' ')) {
        if (n == f.size()) return std::nullopt;
        f[n++] = std::string_view(part.begin(), part.end());
    }
    if (n != 5 || f[0].size() < 13) return std::nullopt;
    Entry e{};
    auto ok = [](auto r, std::string_view s) { return r.ec == std::errc{} && r.ptr == s.data() + s.size(); };
    if (!ok(std::from_chars(f[0].data() + 11, f[0].data() + 13, e.hour), f[0].substr(11, 2)) ||
        !ok(std::from_chars(f[3].data(), f[3].data() + f[3].size(), e.status), f[3]) ||
        !ok(std::from_chars(f[4].data(), f[4].data() + f[4].size(), e.latency_ms), f[4]))
        return std::nullopt;
    e.path = f[2];
    return e;
}

template <typename Map>
double count_paths(const std::vector<Entry>& entries, Map& counts) {
    auto t0 = std::chrono::steady_clock::now();
    for (const auto& e : entries)
        ++counts[e.path];
    return std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
}

int main() {
    std::vector<std::string> lines;                    // owns the text the views point into
    for (std::string line; std::getline(std::cin, line);)
        lines.push_back(std::move(line));
    std::vector<Entry> entries;
    entries.reserve(lines.size());
    for (const auto& l : lines)
        if (auto e = parse(l)) entries.push_back(*e);
    const auto malformed = lines.size() - entries.size();

    std::unordered_map<std::string_view, long> by_path;
    std::map<std::string_view, long> by_path_ordered;
    double t_hash = count_paths(entries, by_path);
    double t_tree = count_paths(entries, by_path_ordered);

    std::array<long, 6> by_class{};
    for (const auto& e : entries) ++by_class[std::clamp(e.status / 100, 0, 5)];

    std::vector<std::pair<std::string_view, long>> top(by_path.begin(), by_path.end());
    auto k = std::min<std::size_t>(5, top.size());
    std::ranges::partial_sort(top, top.begin() + static_cast<long>(k), std::ranges::greater{}, &std::pair<std::string_view, long>::second);

    std::vector<double> lat;
    std::ranges::transform(entries, std::back_inserter(lat), &Entry::latency_ms);
    auto pct = [&lat](double p) {
        auto idx = static_cast<std::size_t>(p * static_cast<double>(lat.size() - 1));
        std::ranges::nth_element(lat, lat.begin() + static_cast<long>(idx));
        return lat[idx];
    };

    std::array<long, 24> per_hour{};
    for (int h : entries | std::views::transform(&Entry::hour)) ++per_hour[static_cast<std::size_t>(h) % 24];
    auto busiest = std::ranges::max_element(per_hour) - per_hour.begin();

    std::println("requests: {} parsed, {} malformed", entries.size(), malformed);
    std::println("status: 2xx={} 3xx={} 4xx={} 5xx={}", by_class[2], by_class[3], by_class[4], by_class[5]);
    std::println("top paths:");
    for (const auto& [path, count] : top | std::views::take(k))
        std::println("  {:<20} {:>7}", path, count);
    std::println("latency ms: p50={:.1f} p90={:.1f} p99={:.1f}", pct(0.50), pct(0.90), pct(0.99));
    std::println("busiest hour: {:02}:00 with {} requests", busiest, per_hour[static_cast<std::size_t>(busiest)]);
    std::println("counting {} entries: unordered_map {:.1f} ms, map {:.1f} ms", entries.size(), t_hash, t_tree);
}
```

```console
$ cd logstat && g++-14 -std=c++23 -O2 -Wall -Wextra gen.cpp -o gen && g++-14 -std=c++23 -O2 -Wall -Wextra logstat.cpp -o logstat && ./gen 1000000 > access.log && head -n 3 access.log && ./logstat < access.log
garbage line
2026-11-15T00:00:00Z GET / 200 12.6
2026-11-15T00:00:00Z GET /static/style.css 200 10.7
requests: 998996 parsed, 1004 malformed
status: 2xx=898372 3xx=30044 4xx=50223 5xx=20357
top paths:
  /                     299570
  /api/users            149561
  /static/app.js        120345
  /static/style.css     120202
  /api/orders            99914
latency ms: p50=12.2 p90=29.8 p99=62.3
busiest hour: 04:00 with 41626 requests
counting 998996 entries: unordered_map 21.6 ms, map 36.5 ms
$ cd logstat && g++-14 -std=c++23 -g -fsanitize=address,undefined -D_GLIBCXX_ASSERTIONS logstat.cpp -o logstat_san && ./gen 20000 | ./logstat_san | head -n 2
requests: 19979 parsed, 21 malformed
status: 2xx=17884 3xx=651 4xx=1034 5xx=410
```

Design notes:

- **A bug found by reading the output.** The first run of this project reported that the busiest hour was 00:00 with half of all requests — implausible for a generator that spreads requests evenly over the day. The cause was in the *generator*: `int sec = i * 86400 / n;` overflows `int` once `i` exceeds 24,855, which is undefined behaviour (signed overflow); in practice the products wrapped negative and produced garbage timestamps. Computing in `long long` fixed it. Two lessons: sanity-check results against what you know about the input, and run test-data generators under UBSan too.
- The program keeps every line in `lines` and stores only `string_view`s into them — no per-entry string copies. The views stay valid because `lines` is never modified after reading (moving strings into a vector that reallocates would be fine too: moving a `std::string` keeps its heap buffer, but short strings stored inline *would* move — another reason to finish filling `lines` before taking views).
- Percentiles use `nth_element` — O(n) each — instead of a full sort.
- With only ten distinct paths, both maps are fast; the hash table wins by avoiding O(log n) string comparisons per lookup. With millions of distinct keys the gap grows (Chapter 12 measures hash-table design further).

### Review checklist

- Are all `string_view`s used only while the strings they point into are alive and unmodified?
- Are malformed lines (wrong field count, bad numbers, short timestamps) counted rather than crashing the parser?
- Is every loop that could be an algorithm written as one?

## Answers to the Chapter 4 prediction questions

**4.3 Q1.** No. If `push_back` reallocates, the range-`for`'s hidden iterators (into the old buffer) are invalidated, and the loop continues reading freed memory. Even without reallocation, the loop's end iterator was computed once at the start and would not include the new element.

- **Unit 4.7, Q3:** `"/a/b/c"`; and `"/c"` — appending an absolute path replaces the whole left-hand side.
- **Unit 4.8, Q3:** the raw engine output is the same everywhere (the algorithm and its parameters are fixed by the standard); the distribution's output may differ, because each library implements the distribution its own way.

# Chapter 5 — Smart Pointers and Ownership

*Goal: never write `delete` again — express ownership in types with `std::unique_ptr`, `std::shared_ptr`, and `std::weak_ptr`, use raw pointers and references only as non-owning observers, and design object graphs without leaks or cycles.*

In C, every `malloc` needs a `free`, and the rules for who frees what live in comments (Chapter 3 of the C book). Modern C++ moves those rules into the type system. A `std::unique_ptr<T>` *is* the statement "I am the only owner of this object, and it dies with me"; a `std::shared_ptr<T>` is "ownership is shared, and the last owner deletes it"; a raw `T*` or `T&` is "I am only looking at it". With these types, leaks and double deletes become compile errors or simply impossible.

**What you will be able to do after this chapter**

- Explain why naked `new` and `delete` are error-prone, and replace them.
- Use `std::unique_ptr` for exclusive ownership, including custom deleters and polymorphic objects.
- Use `std::shared_ptr` and `std::weak_ptr`, know their costs, and break reference cycles.
- Choose function parameter types that state ownership precisely.
- Write RAII scope guards for any cleanup action.

## Topic 5A — Exclusive ownership

## Unit 5.1 — Why not `new` and `delete`

**One-line summary:** `new` allocates and constructs; `delete` destroys and frees. Pairing them manually fails in the same ways `malloc`/`free` do — leaks on early returns and exceptions, double deletes, mismatched `delete`/`delete[]` — so modern C++ code almost never writes either: containers and smart pointers do it.

```cpp
// file: leak.cpp
#include <iostream>
#include <stdexcept>
#include <string>

struct Widget {
    std::string name;
    explicit Widget(std::string n) : name{std::move(n)} {}
};

void risky(bool fail) {
    Widget* w = new Widget{"manual"};
    if (fail)
        throw std::runtime_error("oops");   // leak: delete is never reached
    delete w;
}

int main() {
    try {
        risky(true);
    } catch (const std::exception& e) {
        std::cout << "caught " << e.what() << '\n';
    }
    int* arr = new int[10];
    delete arr;                             // wrong form: must be delete[]
}
```

```console
$ g++-14 -std=c++20 -g -fsanitize=address leak.cpp -o leak && ./leak 2>&1 | grep -E 'caught|ERROR|SUMMARY' | head -n 3
==7618==ERROR: AddressSanitizer: alloc-dealloc-mismatch (operator new [] vs operator delete) on 0x504000000010
SUMMARY: AddressSanitizer: alloc-dealloc-mismatch ../../../../src/libsanitizer/asan/asan_new_delete.cpp:164 in operator delete(void*, unsigned long)
```

AddressSanitizer stops at the `new[]`/`delete` mismatch (`alloc-dealloc-mismatch`); with that fixed, LeakSanitizer would report the `Widget` leaked by the exception. Both bugs are impossible with `std::unique_ptr<Widget>` and `std::vector<int>`.

<!-- expanded -->
### Going deeper: what `new` does, in three steps

`new T(args)` does two separate things: it calls `operator new(sizeof(T))` to get raw memory, then runs `T`'s constructor in that memory. `delete p` runs the destructor, then calls `operator delete`. **Placement new** — `new (address) T(args)` — runs only the constructor, in memory you already have; it is how `std::vector`, `std::optional`, and memory pools build objects inside their own buffers. Mixing the forms is undefined behaviour: memory from `new[]` must go to `delete[]`, from `malloc` to `free`.

```cpp
// file: placement.cpp
#include <cstdio>
#include <memory>
#include <new>
#include <string>

struct Loud {
    std::string name;
    explicit Loud(std::string n) : name(std::move(n)) { std::printf("construct %s\n", name.c_str()); }
    ~Loud() { std::printf("destroy %s\n", name.c_str()); }
};

int main() {
    alignas(Loud) unsigned char storage[sizeof(Loud)];   // raw bytes, no object yet
    Loud* p = new (storage) Loud("in-place");             // construct inside our buffer
    std::printf("object lives at the start of the buffer: %s\n", static_cast<void*>(p) == storage ? "yes" : "no");
    p->~Loud();                                           // destroy explicitly; nothing to free
    std::destroy_at(std::construct_at(p, "again"));       // C++20 spelling of the same two steps
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra placement.cpp -o placement && ./placement
construct in-place
object lives at the start of the buffer: yes
destroy in-place
construct again
destroy again
$ printf '#include <string>\nint main(){ auto* p = new std::string[3]; delete p; }\n' > mismatch.cpp && g++-14 -std=c++20 -g -fsanitize=address mismatch.cpp -o mismatch 2>/dev/null; ./mismatch 2>&1 | grep -m1 -oE "ERROR: AddressSanitizer: [^:]*on address"
ERROR: AddressSanitizer: attempting free on address
```

AddressSanitizer catches the `new[]`/`delete` mismatch: `delete` passed the allocator an address it never handed out, because `new[]` for a type with a destructor stores the element count just *before* the array and returns a pointer past it. Without it, the program runs only one of the three destructors, and on many implementations it corrupts the heap.

### Mastery check

1. **Explain.** Why is `delete` (instead of `delete[]`) on an array undefined behaviour even for `int`?

## Unit 5.2 — `std::unique_ptr`

**One-line summary:** `std::unique_ptr<T>` owns one object and deletes it in its destructor. It cannot be copied — only moved — so ownership is always unique and transfers are explicit. It has the size of a raw pointer and costs nothing at run time. Create it with `std::make_unique<T>(args…)`; give it a custom deleter to manage non-memory resources.

```cpp
// file: unique.cpp
#include <cstdio>
#include <iostream>
#include <memory>
#include <string>
#include <vector>

struct Shape {
    virtual ~Shape() = default;                       // virtual destructor: deleting via Shape* is safe
    virtual double area() const = 0;
};
struct Circle : Shape { double r; explicit Circle(double r) : r{r} {} double area() const override { return 3.14159 * r * r; } };
struct Square : Shape { double s; explicit Square(double s) : s{s} {} double area() const override { return s * s; } };

std::unique_ptr<Shape> make_shape(char kind, double size) {   // factory: returns ownership
    if (kind == 'c') return std::make_unique<Circle>(size);
    return std::make_unique<Square>(size);
}

void consume(std::unique_ptr<Shape> s) {                     // sink: takes ownership
    std::cout << "consumed shape with area " << s->area() << '\n';
}                                                            // deleted here

struct FileCloser { void operator()(std::FILE* f) const { if (f) { std::fclose(f); std::cout << "  file closed\n"; } } };
using FilePtr = std::unique_ptr<std::FILE, FileCloser>;      // RAII for FILE* in one line

int main() {
    std::vector<std::unique_ptr<Shape>> shapes;              // a container of owned, polymorphic objects
    shapes.push_back(make_shape('c', 1.0));
    shapes.push_back(make_shape('s', 2.0));
    for (const auto& s : shapes) std::cout << s->area() << ' ';
    std::cout << '\n';

    auto mine = make_shape('s', 3.0);
    // auto copy = mine;            error: unique_ptr cannot be copied
    consume(std::move(mine));       // explicit transfer
    std::cout << "after move, mine is " << (mine ? "non-null" : "null") << '\n';

    {
        FilePtr f{std::fopen("unique_demo.txt", "w")};
        std::fprintf(f.get(), "hello\n");                    // get(): the raw pointer, non-owning
    }
    std::cout << "sizeof(unique_ptr<Shape>) = " << sizeof(std::unique_ptr<Shape>)
              << ", sizeof(FilePtr) = " << sizeof(FilePtr) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -fsanitize=address unique.cpp -o unique && ./unique
3.14159 4 
consumed shape with area 9
after move, mine is null
  file closed
sizeof(unique_ptr<Shape>) = 8, sizeof(FilePtr) = 8
```

Both unique pointers are the size of one raw pointer: the empty deleter type takes no space. Things to remember:

- **Moved-from `unique_ptr`s are guaranteed null.**
- **`get()`** returns the raw pointer without giving up ownership — for passing to functions that only use the object. **`release()`** gives up ownership (and returns the pointer, which you must then manage) — rarely needed. **`reset(p)`** deletes the current object and takes `p`.
- **Polymorphic deletion needs a virtual destructor** in the base class (Chapter 9); otherwise deleting a `Circle` through a `unique_ptr<Shape>` is undefined behaviour.
- **`std::unique_ptr<T[]>`** manages arrays with `delete[]`, though `std::vector<T>` is almost always better.

<!-- expanded -->
### Going deeper: custom deleters

A `unique_ptr` can own anything that needs a cleanup call, not just memory from `new`: a `FILE*` closed with `fclose`, a C library handle freed with its own function, memory from `malloc`. The deleter is part of the type; a stateless deleter (an empty struct, or a lambda with no captures used through `decltype`) costs no space at all, while a function pointer deleter doubles the `unique_ptr`'s size.

```cpp
// file: deleters.cpp
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <memory>

struct FileCloser { void operator()(std::FILE* f) const { std::printf("fclose\n"); std::fclose(f); } };
using File = std::unique_ptr<std::FILE, FileCloser>;

struct FreeDeleter { void operator()(void* p) const { std::free(p); } };

int main() {
    {
        File f(std::fopen("/tmp/deleters_demo.txt", "w"));
        if (!f) return 1;
        std::fputs("written through a unique_ptr\n", f.get());
    }                                                            // fclose runs here

    std::unique_ptr<char, FreeDeleter> buf(static_cast<char*>(std::malloc(32)));
    std::strcpy(buf.get(), "malloc'd, freed with free");
    std::printf("%s\n", buf.get());

    auto fnptr = std::unique_ptr<std::FILE, int (*)(std::FILE*)>(std::fopen("/dev/null", "r"), &std::fclose);
    std::printf("sizes: plain %zu, empty-struct deleter %zu, function-pointer deleter %zu\n",
                sizeof(std::unique_ptr<int>), sizeof(File), sizeof(fnptr));
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra deleters.cpp -o deleters && ./deleters && cat /tmp/deleters_demo.txt
fclose
malloc'd, freed with free
sizes: plain 8, empty-struct deleter 8, function-pointer deleter 16
written through a unique_ptr
```

### Mastery check

1. **Write.** A `using FdPtr = …` for POSIX file descriptors (`int`, closed with `close`) — hint: a deleter can't hold an `int` in a `unique_ptr` directly; write a small `UniqueFd` class instead and compare.

## Topic 5B — Shared ownership

## Unit 5.3 — `std::shared_ptr`

**One-line summary:** `std::shared_ptr<T>` keeps a **reference count** in a separately allocated **control block**; copying increments it, destruction decrements it, and the object is deleted when it reaches zero. Use it only when ownership is genuinely shared and lifetimes cannot be ordered. It is twice the size of a raw pointer, and its count updates are atomic — measurably slower than `unique_ptr`.

```cpp
// file: shared.cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

struct Texture {
    std::string file;
    explicit Texture(std::string f) : file{std::move(f)} { std::cout << "  load " << file << '\n'; }
    ~Texture() { std::cout << "  unload " << file << '\n'; }
};

struct Sprite { std::shared_ptr<const Texture> tex; int x, y; };

int main() {
    std::vector<Sprite> sprites;
    {
        auto grass = std::make_shared<const Texture>("grass.png");   // one allocation: object + control block
        std::cout << "use_count after make_shared: " << grass.use_count() << '\n';
        for (int i = 0; i < 3; ++i)
            sprites.push_back({grass, i * 10, 0});
        std::cout << "use_count with 3 sprites: " << grass.use_count() << '\n';
    }                                                                 // 'grass' gone; sprites keep it alive
    std::cout << "local handle gone, sprites remain: " << sprites.size() << '\n';
    sprites.pop_back();
    sprites.pop_back();
    std::cout << "one sprite left, use_count " << sprites[0].tex.use_count() << '\n';
    sprites.clear();                                                  // last owner: texture unloaded
    std::cout << "sizeof(shared_ptr) = " << sizeof(std::shared_ptr<Texture>)
              << ", sizeof(unique_ptr) = " << sizeof(std::unique_ptr<Texture>) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra shared.cpp -o shared && ./shared
  load grass.png
use_count after make_shared: 1
use_count with 3 sprites: 4
local handle gone, sprites remain: 3
one sprite left, use_count 1
  unload grass.png
sizeof(shared_ptr) = 16, sizeof(unique_ptr) = 8
```

`std::make_shared` allocates the object and the control block in **one** allocation (instead of two for `std::shared_ptr<T>(new T)`), and is exception-safe. The reference count is updated with atomic instructions so that `shared_ptr` copies in different threads are safe — but the *object* it points to is not made thread-safe by this (Chapter 10).

### Deep & Tricky

**DT-24. Two `shared_ptr`s created separately from the same raw pointer will both delete it.** *(Guarantee: UB, double delete)* `T* p = new T; std::shared_ptr<T> a(p), b(p);` creates two independent control blocks, each with count 1. Always create shared ownership exactly once (`make_shared`), then copy the `shared_ptr`. Inside a class that needs a `shared_ptr` to itself, inherit `std::enable_shared_from_this<T>` and call `shared_from_this()` instead of `std::shared_ptr<T>(this)`.

<!-- expanded -->
### Going deeper: `make_shared` and the control block

Every `shared_ptr` points to a **control block** holding the strong count, the weak count, and the deleter. `std::shared_ptr<T>(new T)` makes two allocations — the object and then the control block. `std::make_shared<T>()` makes one allocation holding both, which is faster and keeps them on the same cache line. The trade-off: with `make_shared`, the object's memory cannot be returned until the last `weak_ptr` is gone too, because it shares an allocation with the control block that the weak pointers still need.

```cpp
// file: makeshared.cpp
#include <cstdio>
#include <cstdlib>
#include <memory>
#include <new>

static int allocations = 0;
void* operator new(std::size_t n) { ++allocations; if (void* p = std::malloc(n)) return p; throw std::bad_alloc(); }
void operator delete(void* p) noexcept { std::free(p); }
void operator delete(void* p, std::size_t) noexcept { std::free(p); }

struct Big { char data[1024]; };

int main() {
    int before = allocations;
    std::shared_ptr<Big> a(new Big);
    std::printf("shared_ptr(new Big): %d allocations\n", allocations - before);
    before = allocations;
    auto b = std::make_shared<Big>();
    std::printf("make_shared<Big>():  %d allocation\n", allocations - before);
    std::printf("sizeof(shared_ptr) = %zu (object pointer + control block pointer)\n", sizeof(a));
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra -Wno-mismatched-new-delete makeshared.cpp -o makeshared && ./makeshared
shared_ptr(new Big): 2 allocations
make_shared<Big>():  1 allocation
sizeof(shared_ptr) = 16 (object pointer + control block pointer)
```

### Deep & Tricky

**DT-25. `shared_ptr`'s reference count is thread-safe; the object it points to is not, and neither is one `shared_ptr` *variable* shared between threads. Two threads copying *their own* `shared_ptr`s to the same object is fine; one thread assigning to a `shared_ptr` while another reads that same variable is a data race.** *(Guarantee: [util.smartptr.shared]/4 — only the control block's counts are synchronised; C++20 adds `std::atomic<std::shared_ptr<T>>` for the shared-variable case)* A configuration pointer swapped by one thread and read by others is the classic case: make the variable `std::atomic<std::shared_ptr<Config>>`, and let readers `load()` a copy they then use freely.

### Mastery check

1. **Explain.** Why does `std::make_shared` need only one allocation, and what is the downside when `weak_ptr`s outlive all `shared_ptr`s? (Hint: Unit 5.4.)

## Unit 5.4 — `std::weak_ptr` and reference cycles

**One-line summary:** Reference counting cannot free a **cycle**: A owns B and B owns A, so neither count reaches zero. `std::weak_ptr` refers to a shared object *without* owning it; `lock()` returns a `shared_ptr` if the object still exists, or an empty one if it was destroyed. Use it for back-references, observers, and caches.

```cpp
// file: cycles.cpp
#include <iostream>
#include <memory>
#include <string>

struct NodeStrong {
    std::string name;
    std::shared_ptr<NodeStrong> partner;      // strong both ways: a cycle
    ~NodeStrong() { std::cout << "  ~" << name << '\n'; }
};

struct NodeWeak {
    std::string name;
    std::shared_ptr<NodeWeak> child;          // owner -> owned: strong
    std::weak_ptr<NodeWeak> parent;           // back-reference: weak
    ~NodeWeak() { std::cout << "  ~" << name << '\n'; }
};

int main() {
    std::cout << std::unitbuf;               // flush each line: LeakSanitizer ends the process at exit
    std::cout << "strong cycle:\n";
    {
        auto a = std::make_shared<NodeStrong>();
        auto b = std::make_shared<NodeStrong>();
        a->name = "A";
        b->name = "B";
        a->partner = b;
        b->partner = a;
    }                                         // no destructor runs: leaked
    std::cout << "weak back-reference:\n";
    {
        auto parent = std::make_shared<NodeWeak>();
        parent->name = "parent";
        parent->child = std::make_shared<NodeWeak>();
        parent->child->name = "child";
        parent->child->parent = parent;
        if (auto p = parent->child->parent.lock())
            std::cout << "  child sees parent " << p->name << '\n';
    }                                         // both destroyed
    std::weak_ptr<int> observer;
    {
        auto value = std::make_shared<int>(42);
        observer = value;
        std::cout << "observer expired? " << observer.expired() << '\n';
    }
    std::cout << "after owner gone, expired? " << observer.expired() << ", lock() is "
              << (observer.lock() ? "non-null" : "null") << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -fsanitize=address cycles.cpp -o cycles && ./cycles 2>&1 | grep -E '^[a-z]|^  [^ #]|^SUMMARY'
strong cycle:
weak back-reference:
  child sees parent parent
  ~parent
  ~child
observer expired? 0
after owner gone, expired? 1, lock() is null
SUMMARY: AddressSanitizer: 128 byte(s) leaked in 2 allocation(s).
```

The strong cycle leaked silently in normal operation — no destructor ran — and LeakSanitizer reports it at exit. With a weak back-reference, both nodes were destroyed. (A first version of this example created the nodes with `std::make_shared<NodeStrong>(NodeStrong{"A", nullptr})` and printed `~A` and `~B` in the strong-cycle case — not the leaked nodes, but the *temporaries* passed to `make_shared`, which were copied rather than moved because the user-declared destructor suppresses the implicit move constructor: DT-13 in action. Constructing in place avoids the temporaries.)

<!-- expanded -->
### Going deeper: `enable_shared_from_this`

An object managed by `shared_ptr` sometimes needs to hand out a `shared_ptr` to *itself* — for example, to register a callback that keeps it alive. `std::shared_ptr<T>(this)` would create a second control block and a double delete (DT-24). Deriving from `std::enable_shared_from_this<T>` lets the object call `shared_from_this()`, which returns a `shared_ptr` sharing the existing control block.

```cpp
// file: fromthis.cpp
#include <functional>
#include <iostream>
#include <memory>
#include <vector>

std::vector<std::function<void()>> pending;           // a simple "event loop" queue

class Session : public std::enable_shared_from_this<Session> {
public:
    explicit Session(int id) : id_(id) {}
    ~Session() { std::cout << "session " << id_ << " destroyed\n"; }
    void start() {
        auto self = shared_from_this();               // keeps the session alive inside the callback
        pending.push_back([self] { std::cout << "callback for session " << self->id_ << '\n'; });
    }
private:
    int id_;
};

int main() {
    std::make_shared<Session>(1)->start();            // the caller's pointer dies immediately ...
    std::cout << "caller dropped its pointer\n";
    for (auto& f : pending) f();                      // ... but the session is still alive here
    pending.clear();                                  // last owner gone: destroyed now
    std::cout << "done\n";
    try {
        Session on_stack(2);
        on_stack.start();                             // not owned by any shared_ptr
    } catch (const std::bad_weak_ptr& e) {
        std::cout << "bad_weak_ptr: " << e.what() << '\n';
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra fromthis.cpp -o fromthis && ./fromthis
caller dropped its pointer
callback for session 1
session 1 destroyed
done
session 2 destroyed
bad_weak_ptr: bad_weak_ptr
```

`shared_from_this()` on an object not owned by any `shared_ptr` throws `std::bad_weak_ptr` (since C++17). The usual protection is to make the constructor private and offer a static `create()` that returns `std::make_shared`.

### Mastery check

1. **Write.** A tree where parents own children (`std::vector<std::unique_ptr<Node>>`) and children point back to their parent with a raw `Node*`. Why is a raw pointer safe here, when the cycle example needed `weak_ptr`?

## Topic 5C — Designing with ownership

## Unit 5.5 — Ownership in interfaces

**One-line summary:** Parameter and return types should say who owns what. **Return** `std::unique_ptr<T>` from factories. **Take** `T&`/`const T&` (or `T*` if it may be null) when the function only uses the object; `std::unique_ptr<T>` by value when it takes ownership; `std::shared_ptr<T>` by value only when it will share ownership (store a copy). Never take a smart pointer by reference just to call a method on the object.

| Signature | Means |
| --- | --- |
| `void use(const Widget& w)` | reads `w`; caller keeps ownership; `w` must exist |
| `void use(Widget* w)` | may receive null; does not own |
| `void keep(std::unique_ptr<Widget> w)` | takes ownership (caller must `std::move`) |
| `void share(std::shared_ptr<Widget> w)` | will co-own (stores the copy) |
| `void reseat(std::unique_ptr<Widget>& w)` | may replace the caller's pointer (rare) |
| `std::unique_ptr<Widget> make()` | caller receives ownership |
| `Widget& get()` / `Widget* find()` | non-owning access to an object owned elsewhere |

### Pattern

**P-3. Make ownership visible in types; observe with references.** *(C++ Core Guidelines R.20–R.37)* Owning pointers are always smart pointers (or containers); raw pointers and references never own. Prefer `unique_ptr`; use `shared_ptr` only for genuinely shared, unordered lifetimes; use `weak_ptr` to observe or to break cycles. *Trade-off:* occasional verbosity (`std::move`, `.get()`) in exchange for leaks, double frees, and ownership confusion becoming impossible or compile errors.

### Deep & Tricky

**DT-26. Passing `shared_ptr` by value in hot code costs two atomic operations per call.** *(Implementation behaviour)* Copying a `shared_ptr` increments the reference count atomically, and destroying the copy decrements it. In a function called millions of times that only reads the object, that is pure overhead and cross-core cache traffic. Pass `const T&` (or `const std::shared_ptr<T>&` if the callee might decide to keep a copy).

## Unit 5.6 — RAII for everything: scope guards

**One-line summary:** When no wrapper type exists for a cleanup action, a **scope guard** — an object holding a lambda that runs in its destructor — gives the same guarantee for one-off cleanup: restoring state, rolling back, removing a temporary file.

```cpp
// file: scopeguard.cpp
#include <cstdio>
#include <iostream>
#include <stdexcept>
#include <utility>

template <typename F>
class ScopeGuard {
public:
    explicit ScopeGuard(F f) : f_{std::move(f)} {}
    ~ScopeGuard() { if (active_) f_(); }
    void dismiss() { active_ = false; }               // commit: don't run the cleanup
    ScopeGuard(const ScopeGuard&) = delete;
    ScopeGuard& operator=(const ScopeGuard&) = delete;
private:
    F f_;
    bool active_ = true;
};

bool write_report(bool fail) {
    std::FILE* f = std::fopen("report.tmp", "w");
    if (!f) return false;
    ScopeGuard close_file{[f] { std::fclose(f); }};
    ScopeGuard remove_tmp{[] { std::remove("report.tmp"); std::cout << "  rolled back: temp file removed\n"; }};
    std::fputs("data\n", f);
    if (fail)
        throw std::runtime_error("validation failed");
    std::fflush(f);
    if (std::rename("report.tmp", "report.txt") != 0) return false;
    remove_tmp.dismiss();                              // success: keep the result
    std::cout << "  committed report.txt\n";
    return true;
}

int main() {
    write_report(false);
    try { write_report(true); } catch (const std::exception& e) { std::cout << "  error: " << e.what() << '\n'; }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra scopeguard.cpp -o scopeguard && ./scopeguard && ls report*
  committed report.txt
  rolled back: temp file removed
  error: validation failed
report.txt
```

The "dismiss on success" form implements **commit-or-rollback**: every early exit undoes the partial work automatically. (C++ library proposals call this `scope_exit`/`scope_fail`/`scope_success`; until it is standard, the dozen lines above are common in codebases.)

<!-- expanded -->
### Going deeper: guards that know whether you are unwinding

A plain scope guard runs its action on every exit. Transactions need two different actions: roll back if the scope is left by an exception, and do nothing (or commit) if it ends normally. `std::uncaught_exceptions()` returns how many exceptions are currently in flight; comparing the count at construction with the count in the destructor tells the guard which kind of exit it is — the idea behind `scope_fail` and `scope_success` in the Library Fundamentals TS.

```cpp
// file: scopefail.cpp
#include <exception>
#include <iostream>
#include <stdexcept>
#include <utility>
#include <vector>

template <typename F> class ScopeFail {
public:
    explicit ScopeFail(F f) : f_(std::move(f)), count_(std::uncaught_exceptions()) {}
    ~ScopeFail() { if (std::uncaught_exceptions() > count_) f_(); }
    ScopeFail(const ScopeFail&) = delete;
    ScopeFail& operator=(const ScopeFail&) = delete;
private:
    F f_;
    int count_;
};

void add_all(std::vector<int>& v, const std::vector<int>& items) {
    std::size_t old_size = v.size();
    ScopeFail rollback([&] { v.resize(old_size); std::cout << "  rolled back to " << old_size << " items\n"; });
    for (int x : items) {
        if (x < 0) throw std::invalid_argument("negative item");
        v.push_back(x);
    }
}

int main() {
    std::vector<int> v{1, 2};
    add_all(v, {3, 4});
    std::cout << "after good batch: " << v.size() << " items\n";
    try { add_all(v, {5, 6, -1, 7}); } catch (const std::exception& e) { std::cout << "caught: " << e.what() << '\n'; }
    std::cout << "after bad batch: " << v.size() << " items\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra scopefail.cpp -o scopefail && ./scopefail
after good batch: 4 items
  rolled back to 4 items
caught: negative item
after bad batch: 4 items
```

The bad batch had already pushed 5 and 6 when it hit -1; the guard saw an exception in flight and undid both, giving the function the strong exception guarantee (Unit 6.2).

## Chapter 5 Project — An asset cache and a scene graph

**Goal:** model the ownership of a small game scene: a **scene graph** where nodes own their children exclusively (`unique_ptr`) and children observe their parent (raw pointer), and an **asset cache** that hands out shared textures (`shared_ptr`) while holding only weak references, so unused textures are freed automatically. Prove with counters and LeakSanitizer that nothing leaks and nothing is freed early.

### Reference solution

```cpp
// file: scene/scene.cpp
#include <cassert>
#include <iostream>
#include <memory>
#include <string>
#include <unordered_map>
#include <vector>

struct Texture {
    static inline int alive = 0, loads = 0;
    std::string path;
    explicit Texture(std::string p) : path{std::move(p)} { ++alive; ++loads; }
    ~Texture() { --alive; }
};

class AssetCache {                                  // observes textures; never keeps them alive
public:
    std::shared_ptr<const Texture> get(const std::string& path) {
        if (auto it = cache_.find(path); it != cache_.end())
            if (auto tex = it->second.lock())       // still alive somewhere: reuse it
                return tex;
        auto tex = std::make_shared<const Texture>(path);
        cache_[path] = tex;                         // store a weak reference only
        return tex;
    }
    std::size_t purge() {                           // forget entries whose textures died
        return std::erase_if(cache_, [](const auto& kv) { return kv.second.expired(); });
    }
private:
    std::unordered_map<std::string, std::weak_ptr<const Texture>> cache_;
};

class Node {
public:
    static inline int alive = 0;
    explicit Node(std::string name, std::shared_ptr<const Texture> tex = nullptr)
        : name_{std::move(name)}, tex_{std::move(tex)} { ++alive; }
    ~Node() { --alive; }
    Node(const Node&) = delete;
    Node& operator=(const Node&) = delete;

    Node& add_child(std::unique_ptr<Node> child) {   // takes ownership
        child->parent_ = this;                       // non-owning back pointer
        children_.push_back(std::move(child));
        return *children_.back();
    }
    std::unique_ptr<Node> detach(const std::string& name) {   // gives ownership back
        for (auto it = children_.begin(); it != children_.end(); ++it)
            if ((*it)->name_ == name) {
                auto node = std::move(*it);
                children_.erase(it);
                node->parent_ = nullptr;
                return node;
            }
        return nullptr;
    }
    std::string path() const { return parent_ ? parent_->path() + "/" + name_ : name_; }
    int count() const {
        int n = 1;
        for (const auto& c : children_) n += c->count();
        return n;
    }
    const Node* find(const std::string& name) const {
        if (name_ == name) return this;
        for (const auto& c : children_)
            if (auto f = c->find(name)) return f;
        return nullptr;
    }
private:
    std::string name_;
    std::shared_ptr<const Texture> tex_;
    Node* parent_ = nullptr;                          // safe: a parent always outlives its children
    std::vector<std::unique_ptr<Node>> children_;
};

int main() {
    AssetCache cache;
    {
        Node root{"world"};
        auto& level = root.add_child(std::make_unique<Node>("level1"));
        auto& house = level.add_child(std::make_unique<Node>("house", cache.get("brick.png")));
        house.add_child(std::make_unique<Node>("door", cache.get("wood.png")));
        house.add_child(std::make_unique<Node>("wall", cache.get("brick.png")));   // reused
        level.add_child(std::make_unique<Node>("tree", cache.get("wood.png")));    // reused

        std::cout << "nodes: " << root.count() << ", textures loaded: " << Texture::loads
                  << ", alive: " << Texture::alive << '\n';
        std::cout << "path of door: " << root.find("door")->path() << '\n';

        auto tree = level.detach("tree");             // ownership moves out of the graph
        std::cout << "after detach: nodes in graph " << root.count() << ", detached path " << tree->path() << '\n';
        tree.reset();                                 // tree destroyed; wood.png still used by the door
        std::cout << "textures alive after tree destroyed: " << Texture::alive << '\n';
    }                                                 // whole graph destroyed: all textures released
    std::cout << "after scene destroyed: nodes alive " << Node::alive << ", textures alive " << Texture::alive
              << ", stale cache entries purged " << cache.purge() << '\n';
    auto again = cache.get("brick.png");              // must reload: the old one is gone
    std::cout << "reloaded brick.png: total loads " << Texture::loads << '\n';
    assert(Node::alive == 0 && Texture::alive == 1);
}
```

```console
$ cd scene && g++-14 -std=c++20 -Wall -Wextra -pedantic -g -fsanitize=address,undefined scene.cpp -o scene && ./scene
nodes: 6, textures loaded: 2, alive: 2
path of door: world/level1/house/door
after detach: nodes in graph 5, detached path tree
textures alive after tree destroyed: 2
after scene destroyed: nodes alive 0, textures alive 0, stale cache entries purged 2
reloaded brick.png: total loads 3
```

Every lifetime in the output follows from the types: textures were loaded twice (two distinct files) despite four requests; detaching a node moved its ownership out; destroying the scene destroyed all nodes recursively and released all textures; the cache's weak references expired instead of keeping textures alive; and a later request reloaded. LeakSanitizer found nothing — and there is not a single `new` or `delete` in the program.

### Review checklist

- Is every owning relationship a `unique_ptr`, `shared_ptr`, or container, and every non-owning one a reference, raw pointer, or `weak_ptr`?
- Can any raw `parent_` pointer outlive its parent? (Only if a node is detached without resetting it — `detach` does reset it.)
- Does the cache ever keep an unused texture alive?

## Answers to the Chapter 5 prediction questions

There are no prediction questions in this chapter.

# Chapter 6 — Error Handling: Exceptions, Safety Guarantees, and `std::expected`

*Goal: handle failures correctly in C++ — know how exceptions propagate and what they cost, write code with precise exception-safety guarantees, use `noexcept` deliberately, and represent expected failures as values with `std::expected`.*

C++ offers two families of error handling. **Exceptions** separate the error path from the normal path and propagate automatically until something catches them, running destructors along the way; they suit errors that are rare and that the immediate caller usually cannot fix. **Error values** — return codes, `std::optional`, and C++23's `std::expected` — make failure part of the function's type; they suit errors that are common, expected, and handled close to where they occur. Professional C++ uses both, deliberately. This chapter teaches the mechanics, the guarantees, the costs (measured), and the judgement.

**What you will be able to do after this chapter**

- Throw, catch, and propagate exceptions, and explain stack unwinding.
- Classify code by exception-safety guarantee (no-throw, strong, basic) and write strongly safe operations.
- Use `noexcept` correctly and know what happens when it is violated.
- Measure the cost of exceptions versus error values.
- Use `std::expected` with its monadic operations, `std::optional`, and `std::error_code`.
- Choose an error strategy for a library and for an application.

## Topic 6A — Exceptions

## Unit 6.1 — Throwing, catching, and stack unwinding

**One-line summary:** `throw` creates an exception object and starts **stack unwinding**: the runtime destroys every local object of every function between the `throw` and the matching `catch`, in reverse order, then runs the handler. Catch by `const&`, from most specific to most general; derive your own exception types from `std::exception` (or its subclasses) so `what()` works everywhere.

```cpp
// file: unwind.cpp
#include <iostream>
#include <stdexcept>
#include <string>

struct Noisy {
    std::string name;
    explicit Noisy(std::string n) : name{std::move(n)} { std::cout << "  acquire " << name << '\n'; }
    ~Noisy() { std::cout << "  release " << name << '\n'; }
};

class ConfigError : public std::runtime_error {        // a domain-specific exception type
public:
    ConfigError(const std::string& key, const std::string& why)
        : std::runtime_error("config key '" + key + "': " + why), key_{key} {}
    const std::string& key() const { return key_; }
private:
    std::string key_;
};

int parse_port(const std::string& text) {
    Noisy guard{"parse buffer"};
    int port = std::stoi(text);                        // throws std::invalid_argument on "abc"
    if (port < 1 || port > 65535)
        throw ConfigError("port", "out of range: " + text);
    return port;
}

void load(const std::string& value) {
    Noisy file{"config file"};
    int port = parse_port(value);
    std::cout << "  port = " << port << '\n';
}

int main() {
    for (std::string v : {"8080", "99999", "abc"}) {
        std::cout << "load(\"" << v << "\"):\n";
        try {
            load(v);
        } catch (const ConfigError& e) {               // most specific first
            std::cout << "  ConfigError for key " << e.key() << ": " << e.what() << '\n';
        } catch (const std::exception& e) {            // anything standard
            std::cout << "  other error: " << e.what() << '\n';
        }
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra unwind.cpp -o unwind && ./unwind
load("8080"):
  acquire config file
  acquire parse buffer
  release parse buffer
  port = 8080
  release config file
load("99999"):
  acquire config file
  acquire parse buffer
  release parse buffer
  release config file
  ConfigError for key port: config key 'port': out of range: 99999
load("abc"):
  acquire config file
  acquire parse buffer
  release parse buffer
  release config file
  other error: stoi
```

On both error paths, the "parse buffer" and "config file" objects were released in reverse order before the handler ran — RAII (Chapter 2) is what makes exceptions safe. Rules:

- **Catch by `const&`.** Catching by value copies the exception and **slices** derived types to the base (losing `ConfigError::key`).
- **Order handlers from derived to base;** the first match wins.
- **`throw;`** inside a handler rethrows the *same* exception object (preserving its dynamic type); `throw e;` throws a sliced copy.
- **Uncaught exceptions** call `std::terminate`, which aborts the program; whether the stack is unwound first is implementation-defined.
- **Don't use exceptions for control flow** (ending loops, normal "not found" results): they are expensive when thrown (Unit 6.4) and obscure the logic.

### The standard exception hierarchy

`std::exception` → `std::logic_error` (`invalid_argument`, `domain_error`, `length_error`, `out_of_range`) for errors in program logic, and `std::runtime_error` (`range_error`, `overflow_error`, `system_error`, …) for errors only detectable at run time; plus `std::bad_alloc` (allocation failure), `std::bad_cast`, `std::bad_optional_access`, and others.

<!-- expanded -->
### Going deeper: catch by reference, rethrow with `throw;`

Catching by value copies the exception and **slices** it: a `catch (std::exception e)` that receives a `std::runtime_error` keeps only the base part, and `e.what()` may lose the message. Always catch by `const&`. To rethrow the *same* exception from a handler, write `throw;` — `throw e;` throws a new copy of the static type, slicing again. And `std::exception_ptr` lets you carry an exception out of the handler, for example from a worker thread to the thread that waits for it.

```cpp
// file: catching.cpp
#include <exception>
#include <iostream>
#include <stdexcept>

struct ParseError : std::runtime_error {
    int line;
    ParseError(const std::string& m, int l) : std::runtime_error(m), line(l) {}
};

void parse() { throw ParseError("unexpected '}'", 42); }

void layer_bad() {
    try { parse(); }
    catch (std::exception& e) { throw e; }        // throws a copy of type std::exception: sliced
}
void layer_good() {
    try { parse(); }
    catch (std::exception&) { throw; }            // rethrows the original ParseError
}

int main() {
    for (auto f : {layer_bad, layer_good}) {
        try { f(); }
        catch (const ParseError& e) { std::cout << "ParseError at line " << e.line << ": " << e.what() << '\n'; }
        catch (const std::exception& e) { std::cout << "only std::exception left: " << e.what() << '\n'; }
    }
    std::exception_ptr saved;
    try { parse(); } catch (...) { saved = std::current_exception(); }
    try { if (saved) std::rethrow_exception(saved); }
    catch (const ParseError& e) { std::cout << "rethrown later, still a ParseError (line " << e.line << ")\n"; }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra catching.cpp -o catching && ./catching
only std::exception left: std::exception
ParseError at line 42: unexpected '}'
rethrown later, still a ParseError (line 42)
```

`layer_bad` turned a `ParseError` into a plain `std::exception` whose `what()` is the generic `std::exception`; the caller's specific handler never ran.

### Mastery check

1. **Predict.** What does `catch (std::exception e)` print for a `ConfigError` via `e.what()`? What is lost?

## Unit 6.2 — Exception safety guarantees

**One-line summary:** Every function offers one of three guarantees if an exception escapes it: **no-throw** (it never throws), **strong** (commit-or-rollback: state is exactly as before the call), or **basic** (no leaks, all invariants hold, but state may have changed). A function that offers none can leak or corrupt objects. The standard technique for the strong guarantee: do all work that might throw on the side, then commit with operations that cannot throw.

```cpp
// file: safety.cpp
#include <iostream>
#include <stdexcept>
#include <string>
#include <utility>
#include <vector>

class Inventory {
public:
    // Basic guarantee only: if the second push_back throws, the first change stays.
    void add_pair_basic(std::string a, std::string b) {
        items_.push_back(std::move(a));
        maybe_fail(b);
        items_.push_back(std::move(b));
    }
    // Strong guarantee: build the new state on the side, then commit with a no-throw swap.
    void add_pair_strong(std::string a, std::string b) {
        std::vector<std::string> next = items_;         // may throw: items_ untouched
        next.push_back(std::move(a));
        maybe_fail(b);
        next.push_back(std::move(b));
        items_.swap(next);                               // no-throw commit
    }
    std::size_t size() const { return items_.size(); }
private:
    static void maybe_fail(const std::string& s) {
        if (s == "bad") throw std::runtime_error("cannot add 'bad'");
    }
    std::vector<std::string> items_;
};

int main() {
    Inventory inv;
    inv.add_pair_strong("apple", "pear");
    try { inv.add_pair_basic("plum", "bad"); } catch (const std::exception& e) {
        std::cout << "basic: " << e.what() << ", size now " << inv.size() << " (half-applied)\n";
    }
    try { inv.add_pair_strong("fig", "bad"); } catch (const std::exception& e) {
        std::cout << "strong: " << e.what() << ", size now " << inv.size() << " (unchanged)\n";
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra safety.cpp -o safety && ./safety
basic: cannot add 'bad', size now 3 (half-applied)
strong: cannot add 'bad', size now 3 (unchanged)
```

The strong version pays for its guarantee with a copy of the whole vector; when that cost matters, choose the basic guarantee and document it — the standard library does exactly this (`vector::push_back` is strong; `vector::insert` in the middle is only basic for types whose moves can throw). The copy-and-swap assignment of Unit 2.4 and the `noexcept`-move rule of Unit 3.2 are both applications of this unit.

### Pattern

**P-4. Prepare on the side, commit with no-throw operations.** *(Practice)* For operations that must be all-or-nothing, perform every step that can fail on temporary objects; then publish the result with operations that cannot fail (`swap`, `std::move` of `noexcept`-movable types, pointer assignment). *Trade-off:* extra copies or allocations in exchange for operations that never leave objects half-modified.

<!-- expanded -->
### Deep & Tricky

**DT-27. An exception thrown while a constructor is running means the object never existed: its destructor does *not* run. Members and base classes that were already fully constructed *are* destroyed — so a class that owns two raw resources and acquires them in its constructor body leaks the first if the second throws.** *(Guarantee: [except.ctor] — destructors are invoked for fully constructed subobjects only)* The fix is not a `try` block in the constructor but one RAII member per resource: each finished member cleans up after itself.

```cpp
// file: ctorthrow.cpp
#include <iostream>
#include <memory>
#include <stdexcept>

struct Resource {
    int id;
    explicit Resource(int i) : id(i) {
        if (i == 2) throw std::runtime_error("cannot acquire resource 2");
        std::cout << "  acquire " << id << '\n';
    }
    ~Resource() { std::cout << "  release " << id << '\n'; }
};

struct RawOwner {
    Resource* a; Resource* b;
    RawOwner() : a(new Resource(1)), b(new Resource(2)) {}   // b throws: a leaks
    ~RawOwner() { delete a; delete b; }                       // never runs
};
struct RaiiOwner {
    std::unique_ptr<Resource> a, b;
    RaiiOwner() : a(std::make_unique<Resource>(1)), b(std::make_unique<Resource>(2)) {}
};

int main() {
    std::cout << "raw pointers:\n";
    try { RawOwner r; } catch (const std::exception& e) { std::cout << "  caught: " << e.what() << '\n'; }
    std::cout << "unique_ptr members:\n";
    try { RaiiOwner r; } catch (const std::exception& e) { std::cout << "  caught: " << e.what() << '\n'; }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra ctorthrow.cpp -o ctorthrow && ./ctorthrow
raw pointers:
  acquire 1
  caught: cannot acquire resource 2
unique_ptr members:
  acquire 1
  release 1
  caught: cannot acquire resource 2
```

Resource 1 is released only in the `unique_ptr` version.

### Mastery check

1. **Classify.** Which guarantee does Unit 2.4's copy-and-swap assignment give, and why?

## Unit 6.3 — `noexcept`

**One-line summary:** `noexcept` declares that a function does not throw. It is a promise checked at run time, not compile time: if an exception escapes a `noexcept` function, `std::terminate` is called immediately. Mark destructors (automatic), move operations, `swap`, and simple non-allocating functions `noexcept`; it enables optimisations and lets the standard library choose faster algorithms (Unit 3.2).

```cpp
// file: noexcept.cpp
#include <iostream>
#include <stdexcept>
#include <type_traits>
#include <vector>

void promise_broken() noexcept {
    throw std::runtime_error("escaped a noexcept function");   // GCC warns: will call terminate
}

template <typename T>
void swap_values(T& a, T& b) noexcept(std::is_nothrow_move_constructible_v<T> &&
                                      std::is_nothrow_move_assignable_v<T>) {   // conditional noexcept
    T tmp = std::move(a);
    a = std::move(b);
    b = std::move(tmp);
}

int main() {
    std::vector<int> a{1}, b{2};
    std::cout << std::boolalpha << "swap_values<vector> noexcept: " << noexcept(swap_values(a, b)) << '\n';
    std::cout << "vector move ctor noexcept: " << std::is_nothrow_move_constructible_v<std::vector<int>> << '\n';
    try {
        promise_broken();
    } catch (...) {
        std::cout << "never reached\n";
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra noexcept.cpp -o noexcept && ./noexcept; echo "exit status: $?"
noexcept.cpp: In function ‘void promise_broken()’:
noexcept.cpp:8:5: warning: ‘throw’ will always call ‘terminate’ [-Wterminate]
    8 |     throw std::runtime_error("escaped a noexcept function");   // GCC warns: will call terminate
      |     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
swap_values<vector> noexcept: true
vector move ctor noexcept: true
terminate called after throwing an instance of 'std::runtime_error'
  what():  escaped a noexcept function
bash: line 1:  8378 Aborted                 ./noexcept
exit status: 134
```

The `catch (...)` never ran: the exception reached the `noexcept` boundary and the program terminated. `noexcept(expr)` as an *operator* asks whether an expression is declared non-throwing — the basis of conditional `noexcept` in generic code.

### Deep & Tricky

**DT-28. `noexcept` is not checked at compile time.** *(Guarantee: C++ standard)* A `noexcept` function may call functions that throw; the compiler only warns in obvious cases (a direct `throw`). Marking a function `noexcept` that calls `std::vector::push_back` turns an out-of-memory condition into an immediate `terminate`. Mark `noexcept` only what genuinely cannot fail.

<!-- expanded -->
### Going deeper: `noexcept` is part of the type, and is queryable

`noexcept(expr)` used as an *operator* asks the compiler whether an expression can throw, without evaluating it. Library code uses this to choose strategies — `std::vector` moves its elements on reallocation only when the move constructor is `noexcept` (DT-14), which it checks with `std::is_nothrow_move_constructible`. Since C++17, `noexcept` is also part of a function's type: a pointer to a `noexcept` function cannot point to a function that may throw.

```cpp
// file: noexceptop.cpp
#include <iostream>
#include <string>
#include <type_traits>
#include <vector>

struct Safe { Safe(Safe&&) noexcept = default; Safe() = default; };
struct Risky { Risky(Risky&&) {} Risky() = default; };
void may_throw();
void never_throws() noexcept;

int main() {
    std::cout << std::boolalpha
              << "noexcept(may_throw()): " << noexcept(may_throw()) << '\n'
              << "noexcept(never_throws()): " << noexcept(never_throws()) << '\n'
              << "noexcept(1 + 2): " << noexcept(1 + 2) << '\n'
              << "string move ctor nothrow: " << std::is_nothrow_move_constructible_v<std::string> << '\n'
              << "Safe nothrow: " << std::is_nothrow_move_constructible_v<Safe> << '\n'
              << "Risky nothrow: " << std::is_nothrow_move_constructible_v<Risky> << '\n'
              << "vector<int>::push_back noexcept: " << noexcept(std::declval<std::vector<int>&>().push_back(1)) << '\n';
    void (*p)() noexcept = never_throws;
    // void (*q)() noexcept = may_throw;     // error: cannot convert
    (void)p;
}
void may_throw() {}
void never_throws() noexcept {}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra noexceptop.cpp -o noexceptop && ./noexceptop
noexcept(may_throw()): false
noexcept(never_throws()): true
noexcept(1 + 2): true
string move ctor nothrow: true
Safe nothrow: true
Risky nothrow: false
vector<int>::push_back noexcept: false
```

`push_back` is not `noexcept` because it may allocate, and allocation may throw `std::bad_alloc`.

## Unit 6.4 — What exceptions cost

**One-line summary:** Modern compilers use **table-based ("zero-cost") exceptions**: code that doesn't throw runs at full speed, with no checks on the normal path, but *throwing* is expensive — typically microseconds, involving table lookups and dynamic allocation. Error values cost a little on every call but almost nothing to "throw". Choose by how often failure happens.

```cpp
// file: cost.cpp
#include <chrono>
#include <expected>
#include <iostream>
#include <stdexcept>
#include <string>

[[gnu::noinline]] int with_exception(int x) {
    if (x % 100 == 0) throw std::runtime_error("bad");
    return x;
}

[[gnu::noinline]] std::expected<int, int> with_expected(int x) {
    if (x % 100 == 0) return std::unexpected(-1);
    return x;
}

int main() {
    using clock = std::chrono::steady_clock;
    const int n = 1'000'000;
    for (int fail_every : {0, 100, 1}) {
        long long sum = 0;
        auto t0 = clock::now();
        for (int i = 1; i <= n; ++i) {
            int x = fail_every == 0 ? 1 : (fail_every == 1 ? i * 100 : i);
            try { sum += with_exception(x); } catch (const std::exception&) { --sum; }
        }
        auto t1 = clock::now();
        for (int i = 1; i <= n; ++i) {
            int x = fail_every == 0 ? 1 : (fail_every == 1 ? i * 100 : i);
            auto r = with_expected(x);
            sum += r ? *r : -1;
        }
        auto t2 = clock::now();
        auto ns = [&](auto a, auto b) { return std::chrono::duration<double, std::nano>(b - a).count() / n; };
        const char* label = fail_every == 0 ? "never fails " : fail_every == 100 ? "fails 1%    " : "always fails";
        std::cout << label << ": exceptions " << ns(t0, t1) << " ns/call, expected " << ns(t1, t2)
                  << " ns/call (checksum " << sum << ")\n";
    }
}
```

```console
$ g++-14 -std=c++23 -O2 -Wall cost.cpp -o cost && ./cost
never fails : exceptions 0.692496 ns/call, expected 1.04306 ns/call (checksum 2000000)
fails 1%    : exceptions 10.929 ns/call, expected 1.3537 ns/call (checksum 989999980000)
always fails: exceptions 971.862 ns/call, expected 1.21163 ns/call (checksum -2000000)
```

When nothing fails, the two are within noise of each other (both a few nanoseconds, mostly the call). As failures become common, exceptions grow to hundreds of nanoseconds or more per throw, while `std::expected` stays flat. That is the quantitative basis for the rule: **exceptions for exceptional failures, error values for expected ones** (parsing user input, lookups that may miss, network timeouts in a retry loop).

Some environments disable exceptions entirely (`-fno-exceptions`): many game engines, embedded systems, and Google's C++ style guide, for reasons of binary size, predictability, or legacy code. Libraries that may be used there should offer non-throwing APIs.

<!-- expanded -->
### Going deeper: `-fno-exceptions`

Some codebases — game engines, much embedded code, parts of Google's C++ — compile with exceptions disabled. With GCC's `-fno-exceptions`, `throw` and `try` are compile errors, and the program cannot catch anything: an exception thrown inside the (precompiled) standard library finds no handler and calls `std::terminate`, which aborts. Code that may be used in both worlds therefore reports errors with return values (`std::expected`, error codes), and libraries document what happens on failure when exceptions are off.

```cpp
// file: noexc.cpp
#include <cstdio>
#include <vector>

int main() {
    std::vector<int> v(3);
    std::printf("v.at(1) = %d\n", v.at(1));
    std::printf("v.at(10) = %d\n", v.at(10));    // throws out_of_range normally; aborts with -fno-exceptions
}
```

```console
$ g++-14 -std=c++20 -fno-exceptions noexc.cpp -o noexc && ./noexc; echo "exit status $?"
v.at(1) = 0
terminate called after throwing an instance of 'std::out_of_range'
  what():  vector::_M_range_check: __n (which is 10) >= this->size() (which is 3)
bash: line 1:  8397 Aborted                 ./noexc
exit status 134
$ printf 'int main(){ try { throw 1; } catch (int) {} }\n' > tryoff.cpp && g++-14 -fno-exceptions -c tryoff.cpp 2>&1 | grep -m1 -oE "error: .*"
error: exception handling disabled, use ‘-fexceptions’ to enable
```

The program aborted (status 134 is 128 + `SIGABRT`) at the first out-of-range `at()`. The `std::out_of_range` was still thrown — by code inside the compiled `libstdc++` — but nothing in our program could catch it, so `std::terminate` ran.

## Topic 6B — Errors as values

## Unit 6.5 — `std::expected`, `std::optional`, and `std::error_code`

**One-line summary:** `std::expected<T, E>` (C++23) holds either a value `T` or an error `E`, making failure part of the return type; its monadic operations (`and_then`, `transform`, `or_else`) chain fallible steps without nested `if`s. `std::optional<T>` is for "maybe a value" without a reason. `std::error_code` carries OS and library error numbers portably.

```cpp
// file: expected.cpp
#include <charconv>
#include <expected>
#include <fstream>
#include <iostream>
#include <string>
#include <string_view>
#include <system_error>

enum class ParseError { empty, not_a_number, out_of_range };

std::string_view describe(ParseError e) {
    switch (e) {
    case ParseError::empty: return "empty input";
    case ParseError::not_a_number: return "not a number";
    case ParseError::out_of_range: return "out of range";
    }
    return "?";
}

std::expected<int, ParseError> parse_int(std::string_view s) {
    if (s.empty()) return std::unexpected(ParseError::empty);
    int v{};
    auto [p, ec] = std::from_chars(s.data(), s.data() + s.size(), v);
    if (ec == std::errc::result_out_of_range) return std::unexpected(ParseError::out_of_range);
    if (ec != std::errc{} || p != s.data() + s.size()) return std::unexpected(ParseError::not_a_number);
    return v;
}

std::expected<int, ParseError> check_percent(int v) {
    if (v < 0 || v > 100) return std::unexpected(ParseError::out_of_range);
    return v;
}

std::expected<std::string, std::error_code> read_file(const std::string& path) {
    std::ifstream in{path};
    if (!in) return std::unexpected(std::make_error_code(std::errc::no_such_file_or_directory));
    return std::string{std::istreambuf_iterator<char>{in}, {}};
}

int main() {
    for (std::string_view input : {"42", "", "4x2", "250", "99999999999"}) {
        auto result = parse_int(input)
                          .and_then(check_percent)                       // runs only on success
                          .transform([](int p) { return p / 100.0; });  // maps the value
        if (result)
            std::cout << '"' << input << "\" -> " << *result << '\n';
        else
            std::cout << '"' << input << "\" -> error: " << describe(result.error()) << '\n';
    }
    auto f = read_file("/no/such/file");
    std::cout << "read_file: " << (f ? "ok" : f.error().message()) << '\n';
}
```

```console
$ g++-14 -std=c++23 -Wall -Wextra expected.cpp -o expected && ./expected
"42" -> 0.42
"" -> error: empty input
"4x2" -> error: not a number
"250" -> error: out of range
"99999999999" -> error: out of range
read_file: No such file or directory
```

The pipeline `parse_int(...).and_then(check_percent).transform(...)` expresses "parse, then validate, then convert" with the error short-circuiting automatically — the value-based equivalent of exceptions propagating, but visible in the types and cheap.

### Choosing a strategy

| Situation | Use |
| --- | --- |
| Failure is rare and the caller can't do much about it (out of memory, broken invariant, corrupt internal state) | exception (or `assert`/`terminate` for bugs) |
| Failure is a normal outcome that the caller will check (parse user input, file not found, lookup miss) | `std::expected<T, E>` (or `std::optional<T>` if no reason is needed) |
| Constructors that can fail | exception (constructors cannot return values), or a factory returning `expected` |
| Interfacing with C, OS APIs | `std::error_code` / `errno` wrapped in `expected` |
| Code built with `-fno-exceptions` | `expected` everywhere |
| Programming errors (preconditions violated) | `assert` in debug builds; contracts (C++26) |

### Deep & Tricky

**DT-29. Dereferencing an `expected` or `optional` that holds no value is undefined behaviour.** *(Library precondition)* `*opt` and `*exp` do not check; `.value()` checks and throws (`bad_optional_access`, `bad_expected_access`). Test first (`if (r)`), use `.value_or(x)`, or build with `-D_GLIBCXX_ASSERTIONS`, which turns the unchecked accesses into aborts during testing.

<!-- expanded -->
### Going deeper: chaining `std::expected` (C++23 monadic operations)

Checking an `expected` after every step reads like C error handling. C++23 adds `and_then` (run the next step only on success; the step itself returns an `expected`), `transform` (apply a plain function to the value), `or_else` (handle or replace the error), and `transform_error`. A pipeline then reads top to bottom, and the first error skips the rest.

```cpp
// file: chain.cpp
#include <charconv>
#include <expected>
#include <iostream>
#include <string>
#include <string_view>

enum class Err { empty, not_a_number, out_of_range };
std::string_view name(Err e) {
    switch (e) { case Err::empty: return "empty"; case Err::not_a_number: return "not a number";
                 case Err::out_of_range: return "out of range"; }
    return "?";
}

std::expected<std::string_view, Err> nonempty(std::string_view s) {
    if (s.empty()) return std::unexpected(Err::empty);
    return s;
}
std::expected<int, Err> to_int(std::string_view s) {
    int v{};
    auto [p, ec] = std::from_chars(s.data(), s.data() + s.size(), v);
    if (ec != std::errc{} || p != s.data() + s.size()) return std::unexpected(Err::not_a_number);
    return v;
}
std::expected<int, Err> port(int v) {
    if (v < 1 || v > 65535) return std::unexpected(Err::out_of_range);
    return v;
}

int main() {
    for (std::string_view input : {"8080", "", "80a", "70000"}) {
        auto result = nonempty(input).and_then(to_int).and_then(port).transform([](int p) { return p + 1; });
        std::cout << '"' << input << "\" -> ";
        if (result) std::cout << "next port " << *result << '\n';
        else std::cout << "error: " << name(result.error()) << '\n';
    }
}
```

```console
$ g++-14 -std=c++23 -Wall -Wextra chain.cpp -o chain && ./chain
"8080" -> next port 8081
"" -> error: empty
"80a" -> error: not a number
"70000" -> error: out of range
```

### Mastery check

1. **Write.** `std::expected<Config, std::string> load_config(path)` chaining `read_file`, a parse step, and a validation step with `and_then`, and printing the first error.

## Chapter 6 Project — An expression evaluator with precise errors

**Goal:** a calculator that parses and evaluates arithmetic expressions (`+ - * / ^`, unary minus, parentheses, numbers with decimals) by **recursive descent**, reporting every error with its position — division by zero, unexpected characters, missing parentheses, trailing input — using `std::expected` throughout, and tested on valid and invalid inputs.

### Grammar

```
expr   := term (('+' | '-') term)*
term   := factor (('*' | '/') factor)*
factor := unary ('^' factor)?          -- right-associative power
unary  := '-' unary | primary
primary:= number | '(' expr ')'
```

### Reference solution

```cpp
// file: calc/calc.hpp
#pragma once
#include <cmath>
#include <cstddef>
#include <expected>
#include <string>
#include <string_view>

struct CalcError {
    std::size_t pos;          // offset in the input where the error was detected
    std::string message;
};

class Parser {
public:
    explicit Parser(std::string_view text) : s_{text} {}

    std::expected<double, CalcError> parse() {
        auto v = expr();
        if (!v) return v;
        skip_ws();
        if (i_ != s_.size()) return fail("unexpected '" + std::string(1, s_[i_]) + "'");
        return v;
    }

private:
    std::expected<double, CalcError> fail(std::string msg) const { return std::unexpected(CalcError{i_, std::move(msg)}); }
    void skip_ws() { while (i_ < s_.size() && s_[i_] == ' ') ++i_; }
    bool eat(char c) { skip_ws(); if (i_ < s_.size() && s_[i_] == c) { ++i_; return true; } return false; }

    std::expected<double, CalcError> expr() {
        auto lhs = term();
        while (lhs) {
            if (eat('+')) { auto r = term(); if (!r) return r; *lhs += *r; }
            else if (eat('-')) { auto r = term(); if (!r) return r; *lhs -= *r; }
            else break;
        }
        return lhs;
    }

    std::expected<double, CalcError> term() {
        auto lhs = factor();
        while (lhs) {
            if (eat('*')) { auto r = factor(); if (!r) return r; *lhs *= *r; }
            else if (eat('/')) {
                skip_ws();
                std::size_t at = i_;                   // position of the divisor
                auto r = factor();
                if (!r) return r;
                if (*r == 0.0) return std::unexpected(CalcError{at, "division by zero"});
                *lhs /= *r;
            } else break;
        }
        return lhs;
    }

    std::expected<double, CalcError> factor() {
        auto base = unary();
        if (base && eat('^'))
            return factor().and_then([&](double e) -> std::expected<double, CalcError> {
                double r = std::pow(*base, e);
                if (!std::isfinite(r)) return fail("result too large");
                return r;
            });
        return base;
    }

    std::expected<double, CalcError> unary() {
        if (eat('-')) return unary().transform([](double v) { return -v; });
        return primary();
    }

    std::expected<double, CalcError> primary() {
        skip_ws();
        if (i_ == s_.size()) return fail("unexpected end of input");
        if (eat('(')) {
            auto v = expr();
            if (!v) return v;
            if (!eat(')')) return fail("expected ')'");
            return v;
        }
        std::size_t start = i_;
        while (i_ < s_.size() && (std::isdigit(static_cast<unsigned char>(s_[i_])) || s_[i_] == '.')) ++i_;
        if (start == i_) return fail("expected a number");
        try {
            std::size_t used = 0;
            double v = std::stod(std::string(s_.substr(start, i_ - start)), &used);
            if (used != i_ - start) { i_ = start + used; return fail("malformed number"); }
            return v;
        } catch (const std::exception&) {      // stod throws: convert at the boundary
            i_ = start;
            return fail("malformed number");
        }
    }

    std::string_view s_;
    std::size_t i_ = 0;
};
```

Design notes: every grammar rule returns `std::expected<double, CalcError>`, and errors propagate by early return — or by `and_then`/`transform` where that reads better. The one place where a library function throws (`std::stod`) is wrapped and converted to an error value at the boundary, so no exception can escape the parser.

```cpp
// file: calc/test.cpp
#include <cassert>
#include <cmath>
#include <iostream>
#include <string>
#include "calc.hpp"

static int failures = 0;

void ok(const char* in, double want) {
    auto r = Parser{in}.parse();
    if (!r || std::fabs(*r - want) > 1e-9) {
        ++failures;
        std::cout << "FAIL: \"" << in << "\" expected " << want << '\n';
    }
}

void err(const char* in, std::size_t pos, const std::string& msg) {
    auto r = Parser{in}.parse();
    if (r || r.error().pos != pos || r.error().message != msg) {
        ++failures;
        std::cout << "FAIL: \"" << in << "\" expected error at " << pos << ": " << msg;
        if (!r) std::cout << " (got " << r.error().pos << ": " << r.error().message << ")";
        std::cout << '\n';
    }
}

int main() {
    ok("1 + 2 * 3", 7);
    ok("(1 + 2) * 3", 9);
    ok("2 ^ 3 ^ 2", 512);              // right-associative: 2^(3^2)
    ok("-2 ^ 2", 4);                   // unary minus binds tighter here: (-2)^2
    ok("10 / 4 - 0.5", 2);
    ok("--3", 3);
    ok("  42  ", 42);
    err("1 / 0", 4, "division by zero");
    err("1 / (2 - 2)", 4, "division by zero");
    err("2 *", 3, "unexpected end of input");
    err("(1 + 2", 6, "expected ')'");
    err("1 + 2)", 5, "unexpected ')'");
    err("3 $ 4", 2, "unexpected '$'");
    err("1.2.3", 3, "malformed number");
    err("10 ^ 1000", 9, "result too large");
    err("", 0, "unexpected end of input");

    for (std::string line : {"(2 + 3) * 4 ^ 0.5", "1 / (3 - 3)"}) {
        auto r = Parser{line}.parse();
        if (r) std::cout << line << " = " << *r << '\n';
        else std::cout << line << "\n" << std::string(r.error().pos, ' ') << "^ " << r.error().message << '\n';
    }
    std::cout << (failures == 0 ? "all calculator tests passed\n" : "SOME TESTS FAILED\n");
    return failures == 0 ? 0 : 1;
}
```

```console
$ cd calc && g++-14 -std=c++23 -Wall -Wextra -pedantic -g -fsanitize=address,undefined test.cpp -o test && ./test
(2 + 3) * 4 ^ 0.5 = 10
1 / (3 - 3)
    ^ division by zero
all calculator tests passed
```

The first run of these tests failed twice: the division-by-zero errors were reported at position 3 — the space after the `/` — instead of at the divisor. The position was taken before skipping whitespace; adding `skip_ws()` first fixed it. Tests that check error *details*, not just "it failed", catch this kind of off-by-one.

The last part shows the user-facing value of positioned errors: a caret under the exact character where the problem was detected.

### Review checklist

- Can any input make an exception escape `parse()`? (Fuzz it: Chapter 11 of this book and Chapter 12 of the C book.)
- Does every error carry the position where it was *detected*, and is that position useful to a user?
- Does deep nesting (`((((…))))` 100,000 levels) overflow the stack? (It does — recursive descent uses stack per level. Add a depth limit and an error for it.)

## Answers to the Chapter 6 prediction questions

**6.1 Q1.** It prints `std::exception`. Catching by value copies only the `std::exception` base part of the `ConfigError` (slicing), so the copy's dynamic type is plain `std::exception`, whose `what()` in libstdc++ returns `"std::exception"`. The message and `key()` are lost. Catch by `const&`.

# Chapter 7 — Templates I: Generic Functions and Classes

*Goal: write generic code that works for many types without losing type safety or speed — function and class templates, argument deduction, non-type parameters, specialisation, and compile-time branching.*

Templates are how C++ achieves "write once, use for any type" with **zero** run-time cost: the compiler generates a separate, fully optimised version of the code for each set of template arguments. The whole standard library — `vector<T>`, `sort`, `unique_ptr<T>`, `format` — is templates. This chapter covers templates as most code uses them; Chapter 8 covers concepts, variadic templates, and metaprogramming.

**What you will be able to do after this chapter**

- Write function templates and predict how their arguments are deduced.
- Write class templates, and use class template argument deduction (CTAD).
- Use non-type template parameters, including values of class type (C++20).
- Specialise templates fully and partially, and know why functions are overloaded instead.
- Explain instantiation, its effect on build time and code size, and how to control it.
- Use type traits, `static_assert`, and `if constexpr` to adapt code to types at compile time.

## Topic 7A — Function templates

## Unit 7.1 — Function templates and deduction

**One-line summary:** `template <typename T> T max_of(T a, T b)` defines a family of functions. Calling `max_of(3, 7)` makes the compiler **deduce** `T = int` and **instantiate** `max_of<int>`. Deduction requires every use of `T` to agree; when it can't, pass the type explicitly (`max_of<double>(3, 7.5)`) or use separate parameters.

```cpp
// file: functmpl.cpp
#include <iostream>
#include <string>
#include <vector>

template <typename T>
T max_of(T a, T b) { return a < b ? b : a; }

template <typename Container>
auto sum(const Container& c) {
    typename Container::value_type total{};     // 'typename': value_type names a type
    for (const auto& x : c) total += x;
    return total;
}

template <typename T, typename U>
auto mixed_add(T a, U b) { return a + b; }       // return type deduced from the expression

int main() {
    std::cout << max_of(3, 7) << ' ' << max_of(2.5, 1.5) << ' ' << max_of<std::string>("pear", "apple") << '\n';
    // max_of(3, 7.5);   error: deduced conflicting types for T ('int' and 'double')
    std::cout << max_of<double>(3, 7.5) << '\n';  // explicit argument: both convert to double
    std::cout << sum(std::vector<int>{1, 2, 3}) << ' ' << sum(std::vector<std::string>{"a", "b", "c"}) << '\n';
    std::cout << mixed_add(1, 2.5) << ' ' << mixed_add(std::string("x"), "y") << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra functmpl.cpp -o functmpl && ./functmpl
7 2.5 pear
7.5
6 abc
3.5 xy
```

Note `max_of<std::string>("pear", "apple")`: without the explicit argument, `T` would be deduced as `const char*` and the function would compare **pointer addresses**, not strings — an example of deduction doing exactly what the rules say and not what you meant.

### `typename` in dependent names

Inside a template, a name that depends on a template parameter (`Container::value_type`) is assumed to be a *value* unless you write `typename` before it. (`auto` and `std::ranges::range_value_t<Container>` often avoid the need.)

<!-- expanded -->
### Going deeper: two-phase lookup

A template is checked twice. When it is *defined*, names that do not depend on the template parameters are looked up and must exist. When it is *instantiated* with real types, the dependent names are looked up. Two consequences trip people up: inside a class template that derives from a template base, members of the base are not found without `this->` (the base is dependent, so it is not searched in the first phase), and a dependent name that is a type needs `typename`.

```cpp
// file: twophase.cpp
#include <iostream>
#include <vector>

template <typename T> struct Base {
    int count = 3;
    void hello() const { std::cout << "Base::hello, count " << count << '\n'; }
};

template <typename T> struct Derived : Base<T> {
    void run() const {
        // hello();               // error: there are no arguments to 'hello' that depend on a template parameter
        this->hello();            // OK: 'this' is dependent, so lookup waits until instantiation
        std::cout << "count via Base<T>:: " << Base<T>::count << '\n';
    }
};

template <typename C> typename C::value_type first(const C& c) {   // C::value_type is a dependent type
    typename C::const_iterator it = c.begin();
    return *it;
}

int main() {
    Derived<int>{}.run();
    std::cout << "first: " << first(std::vector<int>{7, 8}) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra twophase.cpp -o twophase && ./twophase
Base::hello, count 3
count via Base<T>:: 3
first: 7
$ sed 's|// hello();|hello();|' twophase.cpp > twophase_bad.cpp && g++-14 -std=c++20 -c twophase_bad.cpp 2>&1 | grep -m1 -oE "error: .*"
error: there are no arguments to ‘hello’ that depend on a template parameter, so a declaration of ‘hello’ must be available [-fpermissive]
```

(MSVC historically did not do phase one and accepted such code; its `/permissive-` mode — the default in new projects — follows the standard.)

### Mastery check

1. **Predict.** What type is deduced for `T` in `max_of('a', 'b')`? In `max_of(1u, 2u)`?

## Unit 7.2 — The deduction rules

**One-line summary:** Template parameter deduction is the same machinery as `auto`. For a by-value parameter `T`, references and top-level `const` are dropped and arrays/functions **decay** to pointers. For `T&`, the argument's `const` is kept and arrays are not decayed. For `T&&`, lvalues deduce `T` as an lvalue reference (forwarding references, Unit 3.3 of this book).

```cpp
// file: deduction.cpp
#include <iostream>
#include <type_traits>
#include <typeinfo>

template <typename T> const char* by_value(T)        { return std::is_pointer_v<T> ? "pointer (decayed)" : std::is_const_v<T> ? "const" : "plain"; }
template <typename T> const char* by_ref(T&)         { return std::is_array_v<T> ? "array (not decayed)" : std::is_const_v<T> ? "const" : "plain"; }
template <typename T> const char* by_fwd(T&&)        { return std::is_lvalue_reference_v<T> ? "T is an lvalue reference" : "T is a plain type"; }
template <typename T, std::size_t N> std::size_t array_size(T (&)[N]) { return N; }   // deduces N

int main() {
    const int ci = 1;
    int arr[5] = {};
    std::cout << "by_value(ci): " << by_value(ci) << '\n';     // top-level const dropped
    std::cout << "by_value(arr): " << by_value(arr) << '\n';
    std::cout << "by_ref(ci): " << by_ref(ci) << '\n';         // const kept
    std::cout << "by_ref(arr): " << by_ref(arr) << '\n';
    std::cout << "by_fwd(ci): " << by_fwd(ci) << '\n';
    std::cout << "by_fwd(42): " << by_fwd(42) << '\n';
    std::cout << "array_size(arr) = " << array_size(arr) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra deduction.cpp -o deduction && ./deduction
by_value(ci): plain
by_value(arr): pointer (decayed)
by_ref(ci): const
by_ref(arr): array (not decayed)
by_fwd(ci): T is an lvalue reference
by_fwd(42): T is a plain type
array_size(arr) = 5
```

`array_size` shows a classic technique: a reference-to-array parameter keeps the array type, so `N` can be deduced — the safe replacement for the C `COUNT` macro (and the basis of `std::size`).

<!-- expanded -->
### Deep & Tricky

**DT-30. Template argument deduction never considers implicit conversions: `std::max(1, 2.0)` does not compile, because `T` would have to be both `int` and `double`. And `T` deduced from an array argument passed by value decays to a pointer, while `T&` keeps the array type and its size.** *(Guarantee: [temp.deduct.call])* Fix the first by naming the type (`std::max<double>(1, 2.0)`); use the second to write functions that know an array's length.

```cpp
// file: deduce2.cpp
#include <algorithm>
#include <cstddef>
#include <iostream>

template <typename T> void by_value(T) { std::cout << "by value: sizeof(T) = " << sizeof(T) << '\n'; }
template <typename T> void by_ref(T&) { std::cout << "by ref:   sizeof(T) = " << sizeof(T) << '\n'; }
template <typename T, std::size_t N> constexpr std::size_t length(const T (&)[N]) { return N; }

int main() {
    int arr[10]{};
    by_value(arr);                       // T = int*
    by_ref(arr);                         // T = int[10]
    std::cout << "length(arr) = " << length(arr) << '\n';
    std::cout << "max = " << std::max<double>(1, 2.5) << '\n';
    // std::max(1, 2.5);                 // error: deduced conflicting types for 'T' ('int' and 'double')
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra deduce2.cpp -o deduce2 && ./deduce2
by value: sizeof(T) = 8
by ref:   sizeof(T) = 40
length(arr) = 10
max = 2.5
```

### Mastery check

1. **Explain.** Why can `by_ref` see that its argument is an array while `by_value` cannot?

## Topic 7B — Class templates

## Unit 7.3 — Class templates and CTAD

**One-line summary:** A class template (`template <typename T> class Stack`) generates a class per type argument. Member functions are instantiated only if used, so a `Stack<T>` works with types that support only the operations actually called. **Class template argument deduction** (C++17) lets you write `Stack s{1, 2, 3};` and have `T` deduced from the constructor arguments, guided by **deduction guides** when needed.

```cpp
// file: classtmpl.cpp
#include <iostream>
#include <stdexcept>
#include <string>
#include <utility>
#include <vector>

template <typename T>
class Stack {
public:
    Stack() = default;
    Stack(std::initializer_list<T> init) : items_{init} {}
    template <typename It>
    Stack(It first, It last) : items_(first, last) {}

    void push(T value) { items_.push_back(std::move(value)); }
    T pop();                                          // defined outside the class below
    const T& top() const { return items_.back(); }
    bool empty() const { return items_.empty(); }
    std::size_t size() const { return items_.size(); }
    void print() const {                              // requires operator<< for T, but only if called
        for (const auto& x : items_) std::cout << x << ' ';
        std::cout << '\n';
    }
private:
    std::vector<T> items_;
};

template <typename T>
T Stack<T>::pop() {                                    // out-of-class member definition
    if (items_.empty()) throw std::out_of_range("pop on empty stack");
    T v = std::move(items_.back());
    items_.pop_back();
    return v;
}

template <typename It>
Stack(It, It) -> Stack<typename std::iterator_traits<It>::value_type>;   // deduction guide

struct NoPrint { int x; };

int main() {
    Stack s{1, 2, 3};                        // CTAD: Stack<int>
    s.push(4);
    std::cout << "pop " << s.pop() << ", size " << s.size() << '\n';
    s.print();

    std::vector<std::string> words{"a", "b", "c"};
    Stack w(words.begin(), words.end());     // the guide deduces Stack<std::string>
    std::cout << "top of w: " << w.top() << '\n';

    Stack<NoPrint> np;                       // fine: print() is never instantiated for NoPrint
    np.push({7});
    std::cout << "NoPrint stack size " << np.size() << '\n';
    // np.print();   error only if called: no operator<< for NoPrint
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra classtmpl.cpp -o classtmpl && ./classtmpl
pop 4, size 3
1 2 3 
top of w: c
NoPrint stack size 1
```

Without the deduction guide, `Stack w(words.begin(), words.end())` would try to deduce `T` from the iterator constructor — impossible, since `T` doesn't appear in its parameters. The guide says how to compute `T` from the iterator type. The standard library has guides like this for `vector`, `pair`, `optional`, and more.

<!-- expanded -->
### Going deeper: deduction guides

Class template argument deduction (CTAD) deduces template arguments from constructor arguments. When the constructors do not say enough — typically, a constructor taking an iterator pair, from which the element type must be computed — a **deduction guide** tells the compiler what to deduce.

```cpp
// file: guides.cpp
#include <iostream>
#include <iterator>
#include <list>
#include <string>
#include <vector>

template <typename T> class Bag {
public:
    template <typename It> Bag(It first, It last) : items_(first, last) {}
    std::size_t size() const { return items_.size(); }
    const T& front() const { return items_.front(); }
private:
    std::vector<T> items_;
};
template <typename It> Bag(It, It) -> Bag<typename std::iterator_traits<It>::value_type>;   // the guide

template <typename T> struct Named { std::string name; T value; };
Named(const char*, double) -> Named<double>;                                              // guide for an aggregate

int main() {
    std::list<std::string> words{"alpha", "beta"};
    Bag b(words.begin(), words.end());           // deduces Bag<std::string> from the guide
    std::cout << b.size() << " items, first " << b.front() << '\n';
    Named n{"pi", 3.14};
    std::cout << n.name << " = " << n.value << '\n';
    std::vector v{1, 2, 3};                      // CTAD in the standard library: vector<int>
    std::vector w{v.begin(), v.end()};           // careful: vector<vector<int>::iterator>, two elements!
    std::cout << "v has " << v.size() << " ints; w has " << w.size() << " iterators\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra guides.cpp -o guides && ./guides
2 items, first alpha
pi = 3.14
v has 3 ints; w has 2 iterators
```

The last line is a well-known trap: braces prefer the `initializer_list` constructor, so `std::vector w{v.begin(), v.end()}` is a vector of two iterators. With parentheses, `std::vector w(v.begin(), v.end())` uses the standard library's own iterator deduction guide and gives a `vector<int>`.

### Mastery check

1. **Explain.** Why does `Stack<NoPrint>` compile even though `print()` uses `operator<<`?

## Unit 7.4 — Non-type template parameters

**One-line summary:** Template parameters can be **values**, not just types: integers (`std::array<int, 5>`), pointers, enums, and in C++20 floating-point numbers and structural class types. Each distinct value produces a distinct type, so dimensions, sizes, and units become part of the type and are checked at compile time.

```cpp
// file: nttp.cpp
#include <algorithm>
#include <array>
#include <cstddef>
#include <iostream>
#include <string_view>

template <std::size_t N>
struct FixedString {                                    // a string usable as a template argument (C++20)
    char data[N]{};
    constexpr FixedString(const char (&s)[N]) { std::copy_n(s, N, data); }
    constexpr std::string_view view() const { return {data, N - 1}; }
};

template <FixedString Name, typename T = int, T Default = T{}>
struct Setting {
    T value = Default;
    static constexpr std::string_view name() { return Name.view(); }
};

template <int Base>
constexpr int digits(unsigned v) {                       // one function per base, computed at compile time
    int n = 1;
    while (v >= static_cast<unsigned>(Base)) { v /= Base; ++n; }
    return n;
}

int main() {
    Setting<"timeout", int, 30> timeout;
    Setting<"retries"> retries;                          // T = int, Default = 0
    std::cout << timeout.name() << '=' << timeout.value << ", " << retries.name() << '=' << retries.value << '\n';
    static_assert(digits<10>(12345) == 5 && digits<2>(255) == 8 && digits<16>(255) == 2);
    std::array<int, digits<10>(1000000)> buf{};          // array size computed from a template
    std::cout << "buffer for 1000000 has " << buf.size() << " digits\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra nttp.cpp -o nttp && ./nttp
timeout=30, retries=0
buffer for 1000000 has 7 digits
```

Strings as template arguments (`Setting<"timeout">`) became possible in C++20 through structural class types like `FixedString` — the technique behind compile-time format strings and some reflection-style libraries.

### Mastery check

1. **Explain.** Are `Setting<"timeout">` and `Setting<"retries">` the same type? What follows from your answer?

## Unit 7.5 — Specialisation and overloading

**One-line summary:** A **full specialisation** (`template <> class Hash<std::string>`) replaces a template for one specific set of arguments; a **partial specialisation** (`template <typename T> class Printer<T*>`) replaces it for a pattern of arguments — class templates only. Function templates cannot be partially specialised: **overload** them instead.

```cpp
// file: special.cpp
#include <cstring>
#include <iostream>
#include <string>
#include <vector>

template <typename T>
struct TypeName { static std::string get() { return "unknown"; } };          // primary template

template <> struct TypeName<int> { static std::string get() { return "int"; } };              // full
template <> struct TypeName<double> { static std::string get() { return "double"; } };
template <typename T> struct TypeName<T*> { static std::string get() { return TypeName<T>::get() + "*"; } };   // partial
template <typename T> struct TypeName<std::vector<T>> { static std::string get() { return "vector<" + TypeName<T>::get() + ">"; } };

// Functions: overloads instead of partial specialisation
template <typename T> bool equal(const T& a, const T& b) { return a == b; }
inline bool equal(const char* a, const char* b) { return std::strcmp(a, b) == 0; }   // compare contents

int main() {
    std::cout << TypeName<int>::get() << ", " << TypeName<double**>::get() << ", "
              << TypeName<std::vector<int*>>::get() << ", " << TypeName<char>::get() << '\n';
    char a[] = "hi", b[] = "hi";
    std::cout << std::boolalpha << "equal(a, b) = " << equal<const char*>(a, b) << " via template with T = const char*? no: "
              << equal(static_cast<const char*>(a), static_cast<const char*>(b)) << " via the overload\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra special.cpp -o special && ./special
int, double**, vector<int*>, unknown
equal(a, b) = false via template with T = const char*? no: true via the overload
```

The output reveals the subtle part: `equal<const char*>(a, b)` explicitly selected the *template* with `T = const char*`, which compares pointers (two different arrays: `false`); the plain call picked the more specialised non-template overload, which compares contents (`true`). Overload resolution prefers a non-template function over a template when both match equally well.

<!-- expanded -->
### Deep & Tricky

**DT-31. Function templates cannot be partially specialised, and a full specialisation does not take part in overload resolution: the compiler first picks the best *primary template* among the overloads, and only then looks at that template's specialisations. A specialisation can therefore be skipped for a call that seems to match it exactly.** *(Guarantee: [temp.expl.spec], overload resolution considers only primary templates — the "Dimov/Abrahams example")* Prefer overloading functions to specialising them.

```cpp
// file: specorder.cpp
#include <iostream>

template <typename T> void f(T)  { std::cout << "f(T)\n"; }          // primary 1
template <> void f<int*>(int*)   { std::cout << "f<int*> specialisation of f(T)\n"; }
template <typename T> void f(T*) { std::cout << "f(T*)\n"; }         // primary 2, declared later

int main() {
    int x = 0;
    f(&x);       // primary 2 is more specialised than primary 1, so it wins; its own specialisations: none
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra specorder.cpp -o specorder && ./specorder
f(T*)
```

The explicit specialisation `f<int*>` belongs to primary 1, which lost; the call prints `f(T*)` even though a function for exactly `int*` was written.

### Mastery check

1. **Write.** A partial specialisation `TypeName<const T>` that prints `const ` before the inner name.

## Topic 7C — Compile-time techniques

## Unit 7.6 — Instantiation, build times, and code size

**One-line summary:** Each distinct set of template arguments instantiates new code in every translation unit that uses it; the linker merges duplicates (weak symbols, Unit 0.3). Heavy template use therefore costs compile time and can grow binaries. `extern template` and explicit instantiation let you compile common instantiations once.

```cpp
// file: inst/big.hpp
#pragma once
#include <algorithm>
#include <vector>

template <typename T>
T median(std::vector<T> v) {                          // defined in the header: usable with any T
    std::nth_element(v.begin(), v.begin() + v.size() / 2, v.end());
    return v[v.size() / 2];
}

extern template double median<double>(std::vector<double>);   // "instantiated elsewhere: don't instantiate here"
```

```cpp
// file: inst/big.cpp
#include "big.hpp"
template double median<double>(std::vector<double>);           // explicit instantiation: the one copy
```

```cpp
// file: inst/main.cpp
#include <iostream>
#include "big.hpp"

int main() {
    std::cout << median(std::vector<double>{3, 1, 2}) << ' ' << median(std::vector<int>{9, 7, 8}) << '\n';
}
```

```console
$ cd inst && g++-14 -std=c++20 -O2 -c big.cpp main.cpp && g++-14 big.o main.o -o app && ./app && nm -C main.o | grep -E 'median'
2 8
                 U double median<double>(std::vector<double, std::allocator<double> >)
```

`main.o` instantiated `median<int>` itself (at `-O2` it was inlined into `main`, so no separate symbol remains; without optimisation it would appear as a weak `W` symbol), but holds only an undefined reference (`U`) to `median<double>`, whose single copy lives in `big.o`. In a large project, this technique keeps commonly used instantiations from being compiled in hundreds of files.

### Deep & Tricky

**DT-32. Template errors are reported deep inside the instantiation.** *(Compiler behaviour)* Calling `std::sort` on a `std::list` produces dozens of lines about `operator-` on iterators inside `<bits/stl_algo.h>`, not "list iterators are not random access". Read the first "required from here" line that points to *your* code. C++20 concepts (Chapter 8) fix this by checking requirements at the call site.

<!-- expanded -->
### Going deeper: `extern template`

Every translation unit that uses `std::vector<Widget>` instantiates its member functions; the linker later throws away the duplicates. For heavy templates used everywhere with the same arguments, an **explicit instantiation declaration** — `extern template class Big<int>;` in the header — tells every user "do not instantiate this here", and one `.cpp` file provides the single **explicit instantiation definition** — `template class Big<int>;`. This chapter's `inst/` example already measures the cost; the fix is two lines.

```cpp
// file: externtmpl/heavy.hpp
#pragma once
#include <algorithm>
#include <vector>
template <typename T> struct Heavy {
    std::vector<T> data;
    void process() { for (int i = 0; i < 50; ++i) std::sort(data.begin(), data.end()); std::reverse(data.begin(), data.end()); }
};
#ifdef USE_EXTERN
extern template struct Heavy<int>;      // users must not instantiate Heavy<int>
#endif
```

```cpp
// file: externtmpl/heavy.cpp
#include "heavy.hpp"
template struct Heavy<int>;             // the one place it is instantiated
```

```cpp
// file: externtmpl/user.cpp
#include "heavy.hpp"
void use() { Heavy<int> h{{3, 1, 2}}; h.process(); }
```

```console
$ cd externtmpl && g++-14 -std=c++20 -O0 -c user.cpp -o plain.o && g++-14 -std=c++20 -O0 -DUSE_EXTERN -c user.cpp -o ext.o && for o in plain.o ext.o; do echo "$o: $(nm -C $o | grep -c ' W ') template functions defined here, Heavy<int>::process is $(nm -C $o | grep 'Heavy<int>::process' | awk '{print ($1=="U") ? "only referenced (U)" : "defined (W)"}')"; done
plain.o: 112 template functions defined here, Heavy<int>::process is defined (W)
ext.o: 54 template functions defined here, Heavy<int>::process is only referenced (U)
```

Built at `-O0` so that nothing is inlined: without `extern template`, `user.o` carries its own weak (`W`) copy of `Heavy<int>::process` and every `std::sort` helper it pulls in; with it, `process` is only referenced (`U`) and the code lives once, in `heavy.o`. (The compiler may still inline small members — `extern template` forbids *emitting* them, not using them.)

## Unit 7.7 — Type traits, `static_assert`, and `if constexpr`

**One-line summary:** `<type_traits>` answers questions about types at compile time (`std::is_integral_v<T>`, `std::is_same_v<T, U>`, `std::remove_cvref_t<T>`); `static_assert` turns a false answer into a clear compile error; `if constexpr` discards the branch that doesn't apply, so one template can contain code that would not compile for every type.

```cpp
// file: traits.cpp
#include <iostream>
#include <string>
#include <type_traits>
#include <vector>

template <typename T>
std::string describe(const T& value) {
    if constexpr (std::is_integral_v<T>)                  // only the matching branch is compiled
        return "integer " + std::to_string(value) + (value % 2 == 0 ? " (even)" : " (odd)");
    else if constexpr (std::is_floating_point_v<T>)
        return "floating " + std::to_string(value);
    else if constexpr (std::is_convertible_v<T, std::string>)
        return "string \"" + std::string(value) + "\"";
    else
        return "something with sizeof " + std::to_string(sizeof(T));
}

template <typename T>
T safe_average(const std::vector<T>& v) {
    static_assert(std::is_arithmetic_v<T>, "safe_average needs a numeric element type");
    T sum{};
    for (const T& x : v) sum += x;
    return v.empty() ? T{} : sum / static_cast<T>(v.size());
}

int main() {
    std::cout << describe(42) << '\n' << describe(2.5) << '\n' << describe("text") << '\n'
              << describe(std::vector<int>{}) << '\n';
    std::cout << safe_average(std::vector<double>{1, 2, 4}) << '\n';
    // safe_average(std::vector<std::string>{"a"});   error: "safe_average needs a numeric element type"
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra traits.cpp -o traits && ./traits
integer 42 (even)
floating 2.500000
string "text"
something with sizeof 24
2.33333
```

Without `if constexpr`, the `value % 2` in the first branch would be compiled for `std::vector<int>` too, and fail. `if constexpr` is the modern replacement for most of the old SFINAE and tag-dispatch tricks.

<!-- expanded -->
### Going deeper: `if constexpr` discards, `if` does not

In an ordinary `if`, both branches must compile for every type, even if one can never run. `if constexpr` discards the untaken branch inside a template, so it may contain code that would not compile for the current type. Outside a template the discarded branch is still checked, and inside one a `static_assert(false)` in a discarded branch was ill-formed until C++23 (GCC 13+ accepts it, implementing that change as a defect report).

```cpp
// file: ifconstexpr.cpp
#include <iostream>
#include <string>
#include <type_traits>

template <typename T> std::string describe(const T& x) {
    if constexpr (std::is_arithmetic_v<T>)
        return "number " + std::to_string(x);       // would not compile for std::string
    else if constexpr (requires { x.size(); })
        return "sized thing of size " + std::to_string(x.size());
    else
        return "something else";
}

int main() {
    std::cout << describe(42) << '\n' << describe(2.5) << '\n' << describe(std::string("hey")) << '\n'
              << describe(nullptr) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra ifconstexpr.cpp -o ifconstexpr && ./ifconstexpr
number 42
number 2.500000
sized thing of size 3
something else
$ sed 's/if constexpr/if/' ifconstexpr.cpp > plainif.cpp && g++-14 -std=c++20 -c plainif.cpp 2>&1 | grep -m1 -oE "error: [^(]*"
error: request for member ‘size’ in ‘x’, which is of non-class type ‘const int’
```

### Mastery check

1. **Write.** `template <typename T> void serialize(std::ostream&, const T&)` writing integers in decimal, floating-point with 17 significant digits, strings quoted, and rejecting other types with a `static_assert`.

## Chapter 7 Project — Matrices with compile-time dimensions

**Goal:** a `Matrix<T, R, C>` class template whose dimensions are template parameters, so that dimension mismatches — adding a 2×3 to a 3×2, multiplying 2×3 by 2×3 — are **compile errors**, while correct code runs as fast as hand-written loops. Support construction, element access, addition, multiplication, transposition, and printing, mostly `constexpr`.

### Reference solution

```cpp
// file: matrix/matrix.hpp
#pragma once
#include <array>
#include <cstddef>
#include <initializer_list>
#include <ostream>
#include <stdexcept>
#include <type_traits>

template <typename T, std::size_t R, std::size_t C>
class Matrix {
    static_assert(std::is_arithmetic_v<T>, "Matrix elements must be numbers");
    static_assert(R > 0 && C > 0, "Matrix dimensions must be positive");
public:
    constexpr Matrix() = default;
    constexpr Matrix(std::initializer_list<std::initializer_list<T>> rows) {
        if (rows.size() != R) throw std::invalid_argument("wrong number of rows");
        std::size_t r = 0;
        for (const auto& row : rows) {
            if (row.size() != C) throw std::invalid_argument("wrong number of columns");
            std::size_t c = 0;
            for (T v : row) data_[r * C + c++] = v;
            ++r;
        }
    }
    static constexpr Matrix identity() requires (R == C) {        // only square matrices
        Matrix m;
        for (std::size_t i = 0; i < R; ++i) m(i, i) = T{1};
        return m;
    }

    constexpr T& operator()(std::size_t r, std::size_t c) { return data_[r * C + c]; }
    constexpr const T& operator()(std::size_t r, std::size_t c) const { return data_[r * C + c]; }
    static constexpr std::size_t rows() { return R; }
    static constexpr std::size_t cols() { return C; }

    constexpr Matrix<T, C, R> transposed() const {
        Matrix<T, C, R> t;
        for (std::size_t r = 0; r < R; ++r)
            for (std::size_t c = 0; c < C; ++c) t(c, r) = (*this)(r, c);
        return t;
    }

    friend constexpr Matrix operator+(const Matrix& a, const Matrix& b) {   // same R, C enforced by the type
        Matrix out;
        for (std::size_t i = 0; i < R * C; ++i) out.data_[i] = a.data_[i] + b.data_[i];
        return out;
    }
    friend constexpr bool operator==(const Matrix&, const Matrix&) = default;

    friend std::ostream& operator<<(std::ostream& os, const Matrix& m) {
        for (std::size_t r = 0; r < R; ++r) {
            os << (r == 0 ? "[" : " ");
            for (std::size_t c = 0; c < C; ++c) os << (c ? " " : "") << m(r, c);
            os << (r + 1 == R ? "]" : "\n");
        }
        return os;
    }
private:
    std::array<T, R * C> data_{};
};

// (R x K) * (K x C) -> (R x C): the shared K must match, or this template doesn't apply
template <typename T, std::size_t R, std::size_t K, std::size_t C>
constexpr Matrix<T, R, C> operator*(const Matrix<T, R, K>& a, const Matrix<T, K, C>& b) {
    Matrix<T, R, C> out;
    for (std::size_t r = 0; r < R; ++r)
        for (std::size_t k = 0; k < K; ++k)              // i-k-j order: cache-friendly
            for (std::size_t c = 0; c < C; ++c) out(r, c) += a(r, k) * b(k, c);
    return out;
}
```

```cpp
// file: matrix/test.cpp
#include <iostream>
#include "matrix.hpp"

constexpr auto rotation90 = Matrix<int, 2, 2>{{0, -1}, {1, 0}};
static_assert(rotation90 * rotation90 * rotation90 * rotation90 == Matrix<int, 2, 2>::identity());   // compile time

int main() {
    Matrix<double, 2, 3> a{{1, 2, 3}, {4, 5, 6}};
    Matrix<double, 3, 2> b{{7, 8}, {9, 10}, {11, 12}};
    auto c = a * b;                                       // 2x3 * 3x2 = 2x2
    std::cout << "a * b =\n" << c << '\n';
    std::cout << "a + a =\n" << a + a << '\n';
    std::cout << "transpose(a) =\n" << a.transposed() << '\n';
    std::cout << "dimensions of a * b: " << c.rows() << 'x' << c.cols() << ", sizeof(c) = " << sizeof(c) << '\n';
    // auto bad = a + b;   error: no match for operator+ (Matrix<double,2,3> and Matrix<double,3,2>)
    // auto bad2 = a * a;  error: no match for operator* (inner dimensions 3 and 2 differ)
    // Matrix<double, 2, 3>::identity();   error: constraints not satisfied (R == C)
}
```

```console
$ cd matrix && g++-14 -std=c++20 -Wall -Wextra -pedantic test.cpp -o test && ./test
a * b =
[58 64
 139 154]
a + a =
[2 4 6
 8 10 12]
transpose(a) =
[1 4
 2 5
 3 6]
dimensions of a * b: 2x2, sizeof(c) = 32
$ cd matrix && printf '#include "matrix.hpp"\nint main() { Matrix<double, 2, 3> a; auto bad = a * a; (void)bad; }\n' > bad.cpp && g++-14 -std=c++20 -c bad.cpp 2>&1 | grep -E 'error' | head -n 2  
bad.cpp:2:51: error: no match for ‘operator*’ (operand types are ‘Matrix<double, 2, 3>’ and ‘Matrix<double, 2, 3>’)
```

The `static_assert` at the top of `test.cpp` multiplies a rotation matrix by itself four times **during compilation** and checks that the result is the identity. Dimension errors are compile errors; `sizeof(c)` is exactly four `double`s — no dimensions or pointers stored at run time, because the dimensions live in the type. (`requires (R == C)` on `identity` is a C++20 constraint: Chapter 8 explains it.)

### Review checklist

- Which mistakes does the type system now catch, and which does it not (e.g., indexing out of range at run time)?
- Is every operation usable in `constexpr` contexts?
- What happens to compile times and binary size if a program uses 50 different matrix shapes?

## Answers to the Chapter 7 prediction questions

**7.1 Q1.** `char` for `max_of('a', 'b')`; `unsigned int` for `max_of(1u, 2u)`.

**7.4 Q1.** No — different template arguments make different types. They cannot be assigned to each other or stored in the same container without a common base or type erasure, and each gets its own instantiated code.

# Chapter 8 — Templates II: Concepts, Variadic Templates, and Compile-Time Programming

*Goal: write generic code that states its requirements clearly (concepts), accepts any number of arguments (variadic templates and fold expressions), and computes at compile time — the tools behind modern C++ libraries.*

Chapter 7's templates accept any type and fail — with long, obscure errors — when a type doesn't support what the template does. C++20 **concepts** let you state requirements up front, producing clear errors at the call site and allowing overloading by capability. **Variadic templates** handle argument lists of any length (`std::make_unique`, `std::format`, `std::tuple`). **Compile-time programming** with `constexpr` and `consteval` moves computation and checking into the compiler. Together they make it possible to build libraries that are both very general and very safe — this chapter's project is a units-of-measure library in which adding metres to seconds does not compile.

**What you will be able to do after this chapter**

- Constrain templates with standard and custom concepts, and overload by constraints.
- Write variadic function and class templates, and expand parameter packs with fold expressions.
- Write `constexpr` and `consteval` code, including compile-time containers and tables.
- Write type traits, and read legacy SFINAE code.
- Use templated lambdas.

## Topic 8A — Concepts

## Unit 8.1 — Constraining templates with concepts

**One-line summary:** A **concept** is a named compile-time predicate on types (`std::integral<T>`, `std::sortable<It>`). Constraining a template with one (`template <std::integral T>`, `requires` clauses, or `void f(std::integral auto x)`) checks the requirement at the call site, gives readable errors, and allows overloads for different capabilities — the most constrained matching overload wins.

```cpp
// file: concepts.cpp
#include <algorithm>
#include <concepts>
#include <iostream>
#include <list>
#include <string>
#include <vector>

template <typename T>
concept Shape = requires(const T& s) {               // a custom concept: what a Shape must support
    { s.area() } -> std::convertible_to<double>;
    { s.name() } -> std::convertible_to<std::string>;
};

struct Circle { double r; double area() const { return 3.14159 * r * r; } std::string name() const { return "circle"; } };
struct Square { double s; double area() const { return s * s; } std::string name() const { return "square"; } };
struct Blob { int size; };

void describe(const Shape auto& s) {                 // abbreviated function template with a constraint
    std::cout << s.name() << " with area " << s.area() << '\n';
}

// Overloading by constraint: the more specific concept wins.
template <std::integral T> std::string kind(T) { return "integral"; }
template <std::signed_integral T> std::string kind(T) { return "signed integral (more constrained)"; }
template <std::floating_point T> std::string kind(T) { return "floating point"; }

template <typename C>
    requires std::ranges::random_access_range<C>     // requires clause
void sort_container(C& c) { std::sort(c.begin(), c.end()); }

int main() {
    describe(Circle{1});
    describe(Square{2});
    // describe(Blob{3});   error: the constraint 'Shape<Blob>' is not satisfied
    std::cout << kind(1u) << " | " << kind(-1) << " | " << kind(2.5) << '\n';
    std::vector<int> v{3, 1, 2};
    sort_container(v);
    std::cout << v[0] << v[1] << v[2] << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra concepts.cpp -o concepts && ./concepts
circle with area 3.14159
square with area 4
integral | signed integral (more constrained) | floating point
123
```

Compare the error messages for calling a sort on a `std::list`:

```console
$ printf '#include <algorithm>\n#include <list>\nint main() { std::list<int> l{3,1,2}; std::sort(l.begin(), l.end()); }\n' > old.cpp && g++-14 -std=c++20 -c old.cpp 2>&1 | wc -l && g++-14 -std=c++20 -c old.cpp 2>&1 | grep -m1 'error'
28
/usr/include/c++/14/bits/stl_algo.h:1906:50: error: no match for ‘operator-’ (operand types are ‘std::_List_iterator<int>’ and ‘std::_List_iterator<int>’)
$ printf '#include <algorithm>\n#include <list>\nint main() { std::list<int> l{3,1,2}; std::ranges::sort(l); }\n' > new.cpp && g++-14 -std=c++20 -c new.cpp 2>&1 | grep -E 'error|note: the required|not satisfied' | head -n 3
new.cpp:3:56: error: no match for call to ‘(const std::ranges::__sort_fn) (std::__cxx11::list<int>&)’
/usr/include/c++/14/bits/ranges_algo.h:1792:7: note: constraints not satisfied
```

The unconstrained `std::sort` produces a long error from deep inside the library about `operator-` on list iterators. The constrained `std::ranges::sort` fails at the call site — *your* line — with "no match" and "constraints not satisfied"; the notes that follow name `random_access_range` and `random_access_iterator`, the actual reason.

### Writing concepts

A `requires` expression lists valid expressions, their result types (`{ expr } -> concept;`), nested requirements, and required member types. Build concepts from standard ones (`<concepts>`: `std::same_as`, `std::convertible_to`, `std::equality_comparable`, `std::regular`, `std::invocable`; `<iterator>` and `<ranges>`: `std::input_iterator`, `std::ranges::range`, …). Concepts check **syntax**, not semantics: `std::totally_ordered` cannot verify that `<` is actually transitive.

### Deep & Tricky

**DT-33. Overloads constrained by unrelated concepts are ambiguous; subsumption only works through named concepts.** *(Guarantee: constraint normalisation rules)* `std::signed_integral` is defined in terms of `std::integral`, so the compiler knows it is more specific (it *subsumes* it). Two concepts that test overlapping conditions with separately written `requires` expressions are unrelated as far as the compiler is concerned, so a type satisfying both makes the call ambiguous. Build specific concepts from general ones with `&&`.

<!-- expanded -->
### Going deeper: what a concept error looks like, compared with an unconstrained template

The practical gain from concepts is the error message. An unconstrained template fails deep inside its body, with a message about the implementation; a constrained one fails at the call, naming the requirement that was not met.

```cpp
// file: concepterr.cpp
#include <algorithm>
#include <concepts>
#include <list>
#include <vector>

template <typename C> void sort_all(C& c) { std::sort(c.begin(), c.end()); }

template <typename C>
    requires std::random_access_iterator<typename C::iterator>
void sort_all_c(C& c) { std::sort(c.begin(), c.end()); }

int main() {
    std::vector<int> v{3, 1, 2};
    sort_all(v);
    sort_all_c(v);
#ifdef BAD_PLAIN
    std::list<int> l{3, 1, 2};
    sort_all(l);
#endif
#ifdef BAD_CONCEPT
    std::list<int> l{3, 1, 2};
    sort_all_c(l);
#endif
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra concepterr.cpp -o concepterr && echo compiled
compiled
$ g++-14 -std=c++20 -DBAD_PLAIN -c concepterr.cpp 2>&1 | wc -l | sed 's/$/ lines of errors without a concept/'; g++-14 -std=c++20 -DBAD_PLAIN -c concepterr.cpp 2>&1 | grep -m1 -oE "error: .*" | cut -c1-90
31 lines of errors without a concept
error: no match for ‘operator-’ (operand types are ‘std::_List_iterator<int>’ and 
$ g++-14 -std=c++20 -DBAD_CONCEPT -c concepterr.cpp 2>&1 | wc -l | sed 's/$/ lines of errors with a concept/'; g++-14 -std=c++20 -DBAD_CONCEPT -c concepterr.cpp 2>&1 | grep -m2 -oE "(error|note): (no matching|the required|constraints not satisfied).*" | cut -c1-90
23 lines of errors with a concept
error: no matching function for call to ‘sort_all_c(std::__cxx11::list<int>&)’
note: constraints not satisfied
```

The unconstrained error starts inside `std::sort`'s implementation (`operator-` on list iterators); the constrained one says there is no matching `sort_all_c` and that its constraint `random_access_iterator` was not satisfied.

### Mastery check

1. **Write.** A concept `Container` requiring `begin()`, `end()`, `size()`, and a `value_type`, and a function `print_all(const Container auto&)`.

## Topic 8B — Variadic templates

## Unit 8.2 — Parameter packs and fold expressions

**One-line summary:** `template <typename... Ts> void f(Ts... args)` accepts any number of arguments of any types: `Ts` is a **type parameter pack**, `args` a **function parameter pack**. Packs are expanded with `...` in patterns (`g(args)...`, `std::forward<Ts>(args)...`), and **fold expressions** (`(args + ...)`, `(std::cout << ... << args)`) apply a binary operator across them.

```cpp
// file: variadic.cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <utility>

template <typename... Ts>
auto sum(Ts... xs) { return (xs + ... + 0); }                 // binary right fold with an initial value

template <typename... Ts>
void print_all(const Ts&... xs) {
    ((std::cout << xs << ' '), ...);                         // comma fold: one output per argument
    std::cout << "(" << sizeof...(xs) << " args)\n";
}

template <typename T, typename... Ts>
bool all_equal(const T& first, const Ts&... rest) { return ((first == rest) && ...); }   // && fold

template <typename F, typename... Args>
auto call_logged(const std::string& name, F&& f, Args&&... args) {   // perfect forwarding of a pack
    std::cout << "calling " << name << " with " << sizeof...(Args) << " argument(s)\n";
    return std::forward<F>(f)(std::forward<Args>(args)...);
}

// Recursive style (pre-C++17), still useful when each step differs:
void print_typed() { std::cout << '\n'; }
template <typename T, typename... Rest>
void print_typed(const T& x, const Rest&... rest) {
    if constexpr (std::is_same_v<T, std::string>) std::cout << '"' << x << "\" ";
    else std::cout << x << ' ';
    print_typed(rest...);
}

int main() {
    std::cout << sum(1, 2, 3, 4) << ' ' << sum(1.5, 2) << ' ' << sum() << '\n';
    print_all("mixed", 42, 3.5, 'c');
    std::cout << std::boolalpha << all_equal(7, 7, 7) << ' ' << all_equal(1, 1, 2) << '\n';
    auto r = call_logged("max", [](int a, int b, int c) { return std::max({a, b, c}); }, 4, 9, 2);
    std::cout << "result " << r << '\n';
    print_typed(std::string("name"), 3, std::string("x"), 2.5);
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra variadic.cpp -o variadic && ./variadic
10 3.5 0
mixed 42 3.5 c (4 args)
true false
calling max with 3 argument(s)
result 9
"name" 3 "x" 2.5 
```

### Fold forms

| Fold | Expands to (for args a, b, c) |
| --- | --- |
| `(args op ...)` | `a op (b op c)` |
| `(... op args)` | `(a op b) op c` |
| `(args op ... op init)` | `a op (b op (c op init))` |
| `(init op ... op args)` | `((init op a) op b) op c` |

A unary fold over an **empty** pack is allowed only for `&&` (gives `true`), `||` (`false`), and `,` (`void()`); for other operators, give an initial value, as `sum` does.

<!-- expanded -->
### Going deeper: forwarding a pack, and indexing into it

A pack of forwarding references, `Args&&... args`, is forwarded with `std::forward<Args>(args)...` — the `...` expands the whole pattern, once per element. `sizeof...(args)` counts the elements. Getting the *N*th element was awkward before C++26; the usual way is to forward the pack into a tuple and use `std::get<N>`.

```cpp
// file: packs.cpp
#include <iostream>
#include <memory>
#include <string>
#include <tuple>
#include <utility>

struct Widget {
    Widget(std::string n, int w, int h) : name(std::move(n)), area(w * h) {}
    std::string name; int area;
};

template <typename T, typename... Args> std::unique_ptr<T> make(Args&&... args) {
    std::cout << "make with " << sizeof...(args) << " arguments\n";
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

template <std::size_t N, typename... Ts> decltype(auto) nth(Ts&&... ts) {
    return std::get<N>(std::forward_as_tuple(std::forward<Ts>(ts)...));
}

template <typename... Ts> void print_all(const Ts&... ts) {
    std::size_t i = 0;
    ((std::cout << (i++ ? ", " : "") << ts), ...);       // fold over the comma operator
    std::cout << '\n';
}

int main() {
    auto w = make<Widget>(std::string("panel"), 3, 4);
    std::cout << w->name << " area " << w->area << '\n';
    std::cout << "nth<1>: " << nth<1>("a", 2.5, 'c') << '\n';
    print_all(1, "two", 3.0, '4');
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra packs.cpp -o packs && ./packs
make with 3 arguments
panel area 12
nth<1>: 2.5
1, two, 3, 4
```

### Mastery check

1. **Write.** `bool any_of_values(const T& x, const Ts&... candidates)` returning whether `x` equals any candidate, with a fold.

## Unit 8.3 — Variadic class templates

**One-line summary:** Class templates can take packs too: `std::tuple<Ts...>`, `std::variant<Ts...>`, `std::function<R(Args...)>`. Index sequences (`std::index_sequence<0, 1, 2>`) let you expand over the positions of a tuple, and `std::apply` calls a function with a tuple's elements as arguments.

```cpp
// file: tuples.cpp
#include <iostream>
#include <string>
#include <tuple>
#include <utility>

template <typename Tuple, std::size_t... I>
void print_tuple_impl(const Tuple& t, std::index_sequence<I...>) {
    ((std::cout << (I == 0 ? "(" : ", ") << std::get<I>(t)), ...);
    std::cout << ")\n";
}

template <typename... Ts>
void print_tuple(const std::tuple<Ts...>& t) {
    print_tuple_impl(t, std::index_sequence_for<Ts...>{});     // 0, 1, ..., N-1
}

template <typename... Ts>
class TypeList {                                               // a compile-time list of types
public:
    static constexpr std::size_t size = sizeof...(Ts);
    template <typename T>
    static constexpr bool contains = (std::is_same_v<T, Ts> || ...);
};

int main() {
    auto record = std::make_tuple(std::string("ada"), 36, 1.75);
    print_tuple(record);
    auto [name, age, height] = record;                         // structured bindings work on tuples
    std::cout << name << " is " << age << '\n';
    auto total = std::apply([](const auto&... xs) { return sizeof...(xs); }, record);
    std::cout << "apply sees " << total << " elements\n";

    using Numbers = TypeList<int, long, double>;
    static_assert(Numbers::size == 3 && Numbers::contains<long> && !Numbers::contains<char>);
    std::cout << "TypeList checks passed at compile time\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra tuples.cpp -o tuples && ./tuples
(ada, 36, 1.75)
ada is 36
apply sees 3 elements
TypeList checks passed at compile time
```

<!-- expanded -->
### Going deeper: a variadic class template, built by recursion

Before fold expressions, variadic class templates were built by peeling off the first type and inheriting from (or holding) the rest. `std::tuple` is implemented this way. Writing a tiny one shows the technique — and what `std::tuple` saves you from.

```cpp
// file: minituple.cpp
#include <iostream>
#include <string>

template <typename... Ts> struct Tuple {};                       // empty case ends the recursion

template <typename Head, typename... Tail>
struct Tuple<Head, Tail...> {
    Head head;
    Tuple<Tail...> tail;
};

template <std::size_t I, typename Head, typename... Tail>
auto& get(Tuple<Head, Tail...>& t) {
    if constexpr (I == 0) return t.head;
    else return get<I - 1>(t.tail);
}

int main() {
    Tuple<int, std::string, double> t{1, {"two", {3.0, {}}}};
    get<1>(t) += "!";
    std::cout << get<0>(t) << ' ' << get<1>(t) << ' ' << get<2>(t) << '\n';
    std::cout << "sizeof(Tuple<int, std::string, double>) = " << sizeof(t)
              << ", sizeof(std::tuple<...>) = " << sizeof(std::tuple<int, std::string, double>) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra minituple.cpp -o minituple && ./minituple
1 two! 3
sizeof(Tuple<int, std::string, double>) = 56, sizeof(std::tuple<...>) = 48
```

The hand-made tuple is bigger because each empty `Tuple<>` at the end of the chain still occupies a byte (plus padding). `std::tuple` avoids that by inheriting from its empty parts, which lets the *empty base optimisation* give them zero size; C++20's `[[no_unique_address]]` attribute does the same for members.

## Topic 8C — Compile-time programming

## Unit 8.4 — `constexpr` and `consteval` everything

**One-line summary:** Since C++20, `constexpr` functions can use loops, local variables, `try` blocks, virtual calls, and even `std::vector` and `std::string` (as long as memory allocated during constant evaluation is freed before it ends). `consteval` forces evaluation at compile time. Compile-time computation turns run-time work into constants and run-time bugs into compile errors.

```cpp
// file: compiletime.cpp
#include <algorithm>
#include <array>
#include <iostream>
#include <string_view>
#include <vector>

constexpr std::array<bool, 100> sieve() {               // primes below 100, at compile time
    std::array<bool, 100> prime{};
    prime.fill(true);
    prime[0] = prime[1] = false;
    for (std::size_t i = 2; i * i < prime.size(); ++i)
        if (prime[i])
            for (std::size_t j = i * i; j < prime.size(); j += i) prime[j] = false;
    return prime;
}

constexpr int count_unique(std::initializer_list<int> values) {
    std::vector<int> v(values);                         // a std::vector inside a constant expression (C++20)
    std::sort(v.begin(), v.end());
    return static_cast<int>(std::unique(v.begin(), v.end()) - v.begin());
}

consteval std::size_t checked_index(std::string_view key) {
    constexpr std::string_view keys[] = {"red", "green", "blue"};
    for (std::size_t i = 0; i < 3; ++i)
        if (keys[i] == key) return i;
    throw "unknown key";                                // in consteval: becomes a compile error
}

constexpr auto PRIMES = sieve();
static_assert(PRIMES[97] && !PRIMES[91]);
static_assert(count_unique({3, 1, 3, 2, 1}) == 3);

int main() {
    int n = 0;
    for (bool p : PRIMES) n += p;
    std::cout << n << " primes below 100 (table built by the compiler)\n";
    std::cout << "index of green: " << checked_index("green") << '\n';
    // checked_index("purple");   error: 'throw' is not a constant expression: caught at compile time
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra compiletime.cpp -o compiletime && ./compiletime
25 primes below 100 (table built by the compiler)
index of green: 1
$ printf '#include <string_view>\nconsteval int f(std::string_view k) { if (k == "ok") return 1; throw "bad key"; }\nint main() { return f("typo"); }\n' > ce.cpp && g++-14 -std=c++20 -c ce.cpp 2>&1 | grep -E 'error' | head -n 2
ce.cpp:3:22: error: call to consteval function ‘f(std::basic_string_view<char>(((const char*)"typo")))’ is not a constant expression
ce.cpp:2:64: error: expression ‘<throw-expression>’ is not a constant expression
```

`checked_index("purple")` would be rejected by the compiler: lookups by name are verified when the program is built. This is how `std::format` checks format strings — the format string is a `consteval`-constructed object.

### Deep & Tricky

**DT-34. Undefined behaviour in a constant expression is a compile error.** *(Guarantee: C++ standard)* The compiler must diagnose UB (signed overflow, out-of-bounds access, use of an uninitialised value, dangling pointers) that occurs during constant evaluation. Making a function `constexpr` and testing it with `static_assert` is therefore a cheap UB detector for pure code — the same function might silently misbehave at run time.

### Mastery check

1. **Write.** A `constexpr` CRC-32 table (256 entries) generated at compile time, and a `static_assert` that the CRC of `"123456789"` is `0xCBF43926`.

## Unit 8.5 — Type traits, `decltype`, and SFINAE

**One-line summary:** Type traits are templates that compute properties of types; you can write your own with specialisation. `decltype(expr)` gives an expression's type; `std::declval<T>()` pretends to produce a `T` in unevaluated contexts. Before concepts, libraries constrained templates with **SFINAE** ("substitution failure is not an error") — you will still read it in existing code.

```cpp
// file: mytraits.cpp
#include <iostream>
#include <list>
#include <string>
#include <type_traits>
#include <vector>

template <typename T> struct is_vector : std::false_type {};               // primary: false
template <typename T, typename A> struct is_vector<std::vector<T, A>> : std::true_type {};   // partial: true
template <typename T> constexpr bool is_vector_v = is_vector<T>::value;

template <typename T, typename = void>
struct has_size : std::false_type {};
template <typename T>
struct has_size<T, std::void_t<decltype(std::declval<const T&>().size())>> : std::true_type {};   // detection idiom

// Legacy SFINAE: enable_if removes the overload when the condition is false
template <typename T, std::enable_if_t<std::is_integral_v<T>, int> = 0>
std::string old_style(T) { return "integral (enable_if)"; }
template <typename T, std::enable_if_t<!std::is_integral_v<T>, int> = 0>
std::string old_style(T) { return "other (enable_if)"; }

// The same with a concept-style requires clause
template <typename T> requires std::is_integral_v<T> std::string new_style(T) { return "integral (requires)"; }
template <typename T> std::string new_style(T) { return "other (unconstrained)"; }

int main() {
    std::cout << std::boolalpha << is_vector_v<std::vector<int>> << ' ' << is_vector_v<std::list<int>> << '\n';
    std::cout << has_size<std::string>::value << ' ' << has_size<int>::value << '\n';
    std::cout << old_style(1) << ", " << old_style(1.5) << '\n';
    std::cout << new_style(1) << ", " << new_style(1.5) << '\n';
    decltype(1 + 2.0) d = 0;                                                // double
    std::cout << "decltype(1 + 2.0) is double: " << std::is_same_v<decltype(d), double> << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra mytraits.cpp -o mytraits && ./mytraits
true false
true false
integral (enable_if), other (enable_if)
integral (requires), other (unconstrained)
decltype(1 + 2.0) is double: true
```

In new code, write concepts; recognise `enable_if`, `void_t`, and `decltype(…)` return types as their older equivalents.

<!-- expanded -->
### Going deeper: SFINAE versus `requires`, side by side

**SFINAE** ("substitution failure is not an error") means that when substituting template arguments into a function's *signature* produces invalid code, that function is quietly dropped from the overload set instead of causing an error. Before C++20 it was the only way to constrain templates, through `std::enable_if`. The same constraint written three ways:

```cpp
// file: sfinae.cpp
#include <iostream>
#include <string>
#include <type_traits>

// C++11: enable_if in the return type
template <typename T>
std::enable_if_t<std::is_integral_v<T>, std::string> kind11(T) { return "integral"; }
template <typename T>
std::enable_if_t<!std::is_integral_v<T>, std::string> kind11(T) { return "not integral"; }

// C++11: expression SFINAE — this overload exists only if t.size() is valid
template <typename T> auto size_or(const T& t, int) -> decltype(t.size(), std::size_t{}) { return t.size(); }
template <typename T> std::size_t size_or(const T&, long) { return 0; }

// C++20: the same with requires
template <typename T> std::size_t size20(const T& t) {
    if constexpr (requires { t.size(); }) return t.size(); else return 0;
}

int main() {
    std::cout << kind11(3) << ", " << kind11(3.5) << '\n';
    std::cout << size_or(std::string("abcd"), 0) << ", " << size_or(42, 0) << '\n';
    std::cout << size20(std::string("abcd")) << ", " << size20(42) << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra sfinae.cpp -o sfinae && ./sfinae
integral, not integral
4, 0
4, 0
```

The `int`/`long` trick in `size_or` ranks the overloads: passing `0` (an `int`) prefers the first when it exists. You will meet all of this in existing code; for new code, write `requires`.

## Unit 8.6 — Templated lambdas

**One-line summary:** Lambdas can have explicit template parameter lists (`[]<typename T>(const std::vector<T>& v) {…}`, C++20) in addition to `auto` parameters, which lets them name the deduced types and constrain them.

```cpp
// file: tlambda.cpp
#include <concepts>
#include <iostream>
#include <string>
#include <vector>

int main() {
    auto first_or = []<typename T>(const std::vector<T>& v, T fallback) {   // names T: both args must agree
        return v.empty() ? fallback : v.front();
    };
    auto twice = []<std::integral T>(T x) { return 2 * x; };                  // constrained
    std::cout << first_or(std::vector<int>{}, -1) << ' ' << first_or(std::vector<std::string>{"a"}, std::string("z"))
              << ' ' << twice(21) << '\n';
    // twice(1.5);   error: constraints not satisfied
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra tlambda.cpp -o tlambda && ./tlambda
-1 a 42
```

<!-- expanded -->
### Going deeper: what templated lambdas add

A generic lambda (`[](auto x)`) gives each parameter its own invented type. The C++20 syntax `[]<typename T>(T a, T b)` names the type, so two parameters can be required to have the *same* type, the type can be used in the body, and the lambda can take a `std::vector<T>` and know `T`.

```cpp
// file: tlambda2.cpp
#include <iostream>
#include <string>
#include <vector>

int main() {
    auto same_type_max = []<typename T>(const T& a, const T& b) { return a < b ? b : a; };
    auto element_size = []<typename T>(const std::vector<T>&) { return sizeof(T); };
    auto loose = [](auto a, auto b) { return a < b ? b : a; };

    std::cout << same_type_max(3, 9) << ' ' << same_type_max(std::string("a"), std::string("b")) << '\n';
    std::cout << element_size(std::vector<double>{}) << ' ' << element_size(std::vector<char>{}) << '\n';
    std::cout << loose(1, 2.5) << '\n';          // int and double mixed: compiles, result type double
    // same_type_max(1, 2.5);                    // error: T deduced as both int and double
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra tlambda2.cpp -o tlambda2 && ./tlambda2
9 b
8 1
2.5
```

## Chapter 8 Project — A compile-time units library

**Goal:** a small dimensional-analysis library: `Quantity<Dim>` values whose physical dimension (length, mass, time exponents) is part of the type, so that `metres + seconds` does not compile, `metres / seconds` has type velocity, and conversions are explicit — with zero run-time overhead compared with plain `double`s.

### Reference solution

```cpp
// file: units/units.hpp
#pragma once
#include <concepts>
#include <ostream>
#include <ratio>

template <int L, int M, int T>            // exponents of length, mass, time
struct Dim {
    static constexpr int length = L, mass = M, time = T;
};

template <typename D1, typename D2>
using dim_multiply = Dim<D1::length + D2::length, D1::mass + D2::mass, D1::time + D2::time>;
template <typename D1, typename D2>
using dim_divide = Dim<D1::length - D2::length, D1::mass - D2::mass, D1::time - D2::time>;

template <typename D>
class Quantity {
public:
    constexpr explicit Quantity(double v) : value_{v} {}
    constexpr double value() const { return value_; }      // in SI base units

    constexpr Quantity& operator+=(Quantity o) { value_ += o.value_; return *this; }
    friend constexpr Quantity operator+(Quantity a, Quantity b) { return a += b; }   // same D only
    friend constexpr Quantity operator-(Quantity a, Quantity b) { return Quantity{a.value_ - b.value_}; }
    friend constexpr auto operator<=>(Quantity, Quantity) = default;
    friend constexpr Quantity operator*(double k, Quantity q) { return Quantity{k * q.value_}; }
    friend constexpr Quantity operator*(Quantity q, double k) { return Quantity{q.value_ * k}; }

    template <typename D2>
    friend constexpr auto operator*(Quantity a, Quantity<D2> b) { return Quantity<dim_multiply<D, D2>>{a.value_ * b.value()}; }
    template <typename D2>
    friend constexpr auto operator/(Quantity a, Quantity<D2> b) { return Quantity<dim_divide<D, D2>>{a.value_ / b.value()}; }

private:
    double value_;
};

using Dimensionless = Dim<0, 0, 0>;
using Length = Quantity<Dim<1, 0, 0>>;
using Mass = Quantity<Dim<0, 1, 0>>;
using Time = Quantity<Dim<0, 0, 1>>;
using Velocity = Quantity<Dim<1, 0, -1>>;
using Acceleration = Quantity<Dim<1, 0, -2>>;
using Force = Quantity<Dim<1, 1, -2>>;
using Energy = Quantity<Dim<2, 1, -2>>;

namespace literals {
constexpr Length operator""_m(long double v) { return Length{static_cast<double>(v)}; }
constexpr Length operator""_km(long double v) { return Length{static_cast<double>(v) * 1000}; }
constexpr Mass operator""_kg(long double v) { return Mass{static_cast<double>(v)}; }
constexpr Time operator""_s(long double v) { return Time{static_cast<double>(v)}; }
constexpr Time operator""_h(long double v) { return Time{static_cast<double>(v) * 3600}; }
}

template <typename D>
concept PhysicalDim = requires { D::length; D::mass; D::time; };

template <PhysicalDim D>
std::ostream& operator<<(std::ostream& os, Quantity<D> q) {
    os << q.value();
    auto unit = [&os](const char* sym, int e) { if (e) { os << ' ' << sym; if (e != 1) os << '^' << e; } };
    unit("m", D::length);
    unit("kg", D::mass);
    unit("s", D::time);
    return os;
}

template <typename D>
constexpr double in(Quantity<D> q, Quantity<D> unit) { return q.value() / unit.value(); }   // explicit conversion
```

```cpp
// file: units/test.cpp
#include <iostream>
#include "units.hpp"

using namespace literals;

constexpr Energy kinetic_energy(Mass m, Velocity v) { return 0.5 * (m * v * v); }

static_assert(std::is_same_v<decltype(1.0_m / 1.0_s), Velocity>);
static_assert(std::is_same_v<decltype(1.0_kg * (1.0_m / (1.0_s * 1.0_s))), Force>);
static_assert(kinetic_energy(2.0_kg, Velocity{3.0}).value() == 9.0);       // checked by the compiler
static_assert(sizeof(Length) == sizeof(double));                          // zero overhead in size

int main() {
    Length trip = 42.195_km;
    Time duration = 2.0_h + 1.0_s * 2340;                       // 2:39:00
    Velocity pace = trip / duration;
    std::cout << "marathon distance: " << trip << '\n';
    std::cout << "average speed: " << pace << " = " << in(pace, Velocity{1000.0 / 3600}) << " km/h\n";

    Mass m = 70.0_kg;
    Acceleration g{9.81};
    Force weight = m * g;
    std::cout << "weight: " << weight << '\n';
    std::cout << "kinetic energy: " << kinetic_energy(m, pace) << '\n';
    std::cout << "is the trip longer than 40 km? " << std::boolalpha << (trip > 40.0_km) << '\n';
    // auto nonsense = trip + duration;   error: no match for 'operator+' (Length and Time)
    // Velocity wrong = trip * duration;  error: cannot convert Quantity<Dim<1,0,1>> to Velocity
}
```

```console
$ cd units && g++-14 -std=c++20 -Wall -Wextra -pedantic test.cpp -o test && ./test
marathon distance: 42195 m
average speed: 4.42296 m s^-1 = 15.9226 km/h
weight: 686.7 m kg s^-2
kinetic energy: 684.689 m^2 kg s^-2
is the trip longer than 40 km? true
$ cd units && printf '#include "units.hpp"\nusing namespace literals;\nint main() { auto x = 1.0_m + 1.0_s; (void)x; }\n' > bad.cpp && g++-14 -std=c++20 -c bad.cpp 2>&1 | grep -m1 error
bad.cpp:3:29: error: no match for ‘operator+’ (operand types are ‘Length’ {aka ‘Quantity<Dim<1, 0, 0> >’} and ‘Time’ {aka ‘Quantity<Dim<0, 0, 1> >’})
$ cd units && printf '#include "units.hpp"\ndouble f(double a, double b) { return a / b; }\nauto g(Length a, Time b) { return a / b; }\n' > cmp.cpp && g++-14 -std=c++20 -O2 -fcf-protection=none -c cmp.cpp && objdump -d --no-show-raw-insn -C cmp.o | grep -A3 -E '^[0-9a-f]+ <(f|g)\('
0000000000000000 <f(double, double)>:
   0:	divsd  %xmm1,%xmm0
   4:	ret
   5:	data16 cs nopw 0x0(%rax,%rax,1)
--
0000000000000010 <g(Quantity<Dim<1, 0, 0> >, Quantity<Dim<0, 0, 1> >)>:
  10:	divsd  %xmm1,%xmm0
  14:	ret
```

The last command compares the generated code for dividing two plain `double`s and dividing a `Length` by a `Time`: the same single `divsd` instruction. The dimensions exist only in the type system — **zero-overhead abstraction** in its purest form. The physics is checked by the compiler, including in `static_assert`s.

### Review checklist

- Which errors does the library turn into compile errors? Which conversions remain possible by mistake (e.g., `Length{3600}` meaning seconds)?
- How would you add temperature, current, or angles? (More exponents in `Dim`, or a `std::ratio`-based scale.)
- Are all operations `constexpr` and free of run-time overhead (check the assembly)?

## Answers to the Chapter 8 prediction questions

There are no prediction questions in this chapter.

# Chapter 9 — Polymorphism: Inheritance, Virtual Functions, Variants, and Type Erasure

*Goal: model "one interface, many implementations" in C++ — with virtual functions when the set of types is open, with `std::variant` when it is closed, with templates when it is known at compile time, and with type erasure when you want value semantics — and know what each costs.*

Object-oriented C++ is built on inheritance and virtual functions, and it remains the right tool for many designs: plugin systems, GUI widgets, interfaces between modules. But modern C++ offers alternatives that are often simpler or faster — `std::variant` with `std::visit`, the CRTP for compile-time polymorphism, and type erasure (the technique behind `std::function`). This chapter teaches the classic mechanics precisely (including the pitfalls), then the alternatives, then measures them against each other.

**What you will be able to do after this chapter**

- Use inheritance with correct construction order, access control, and virtual destructors.
- Use `virtual`, `override`, `final`, and abstract classes, and explain vtables.
- Avoid slicing, name hiding, and virtual calls during construction.
- Use `std::variant` and `std::visit` for closed sets of types.
- Implement static polymorphism with the CRTP, and type erasure with value semantics.
- Use C++'s casts correctly, including `dynamic_cast` and `std::bit_cast`.

## Topic 9A — Classic object orientation

## Unit 9.1 — Inheritance

**One-line summary:** A derived class inherits the members of its base classes. Bases are constructed before the derived class and destroyed after it; members are constructed in declaration order. Use public inheritance only for "is-a" relationships where every derived object can be used wherever a base is expected (the Liskov substitution principle); for "has-a" or code reuse, prefer composition.

```cpp
// file: inherit.cpp
#include <iostream>
#include <string>

struct Logger {
    std::string tag;
    explicit Logger(std::string t) : tag{std::move(t)} { std::cout << "  Logger(" << tag << ")\n"; }
    ~Logger() { std::cout << "  ~Logger(" << tag << ")\n"; }
};

class Animal {
public:
    explicit Animal(std::string name) : name_{std::move(name)}, log_{"animal"} { std::cout << "  Animal\n"; }
    ~Animal() { std::cout << "  ~Animal\n"; }
    const std::string& name() const { return name_; }
protected:                                      // visible to derived classes, not to users
    int legs_ = 4;
private:
    std::string name_;
    Logger log_;
};

class Bird : public Animal {
public:
    explicit Bird(std::string name) : Animal{std::move(name)}, log_{"bird"} {   // base initialised first
        legs_ = 2;
        std::cout << "  Bird\n";
    }
    ~Bird() { std::cout << "  ~Bird\n"; }
    int legs() const { return legs_; }
private:
    Logger log_;
};

int main() {
    std::cout << "constructing:\n";
    {
        Bird b{"robin"};
        std::cout << b.name() << " has " << b.legs() << " legs\ndestroying:\n";
    }
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra inherit.cpp -o inherit && ./inherit
constructing:
  Logger(animal)
  Animal
  Logger(bird)
  Bird
robin has 2 legs
destroying:
  ~Bird
  ~Logger(bird)
  ~Animal
  ~Logger(animal)
```

Construction runs from the base outwards (base members, base body, derived members, derived body); destruction is the exact reverse. `protected` data like `legs_` is a weak form of encapsulation — every derived class can break the base's invariants — so prefer `protected` *functions* and keep data `private`.

<!-- expanded -->
### Going deeper: object slicing

Copying a derived object into a base *value* keeps only the base part: the derived members are cut off and the virtual functions of the copy are the base's. This is **slicing**, and it happens silently in assignments, in pass-by-value, and in containers of base values (`std::vector<Shape>`).

```cpp
// file: slicing.cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

struct Animal {
    virtual ~Animal() = default;
    virtual std::string sound() const { return "..."; }
};
struct Dog : Animal {
    std::string name = "Rex";
    std::string sound() const override { return "woof (" + name + ")"; }
};

void by_value(Animal a) { std::cout << "by value:     " << a.sound() << '\n'; }
void by_ref(const Animal& a) { std::cout << "by reference: " << a.sound() << '\n'; }

int main() {
    Dog d;
    by_value(d);
    by_ref(d);
    std::vector<Animal> values{d};                         // sliced on the way in
    std::vector<std::unique_ptr<Animal>> owners;
    owners.push_back(std::make_unique<Dog>());
    std::cout << "vector<Animal>: " << values[0].sound() << ", vector<unique_ptr<Animal>>: " << owners[0]->sound() << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra slicing.cpp -o slicing && ./slicing
by value:     ...
by reference: woof (Rex)
vector<Animal>: ..., vector<unique_ptr<Animal>>: woof (Rex)
```

The usual defence is to make polymorphic base classes non-copyable (delete the copy constructor and assignment, or make them `protected`), so that slicing becomes a compile error; the Core Guidelines rule C.67 says exactly this.

## Unit 9.2 — Virtual functions and dynamic dispatch

**One-line summary:** A `virtual` member function is dispatched on the object's **dynamic** type: calling it through a base pointer or reference runs the most-derived override. Mark overrides `override` (the compiler checks the signature), prevent further overriding with `final`, make a class abstract with pure virtual functions (`= 0`), and give every polymorphic base a **virtual destructor**.

```cpp
// file: virtual.cpp
#include <iostream>
#include <memory>
#include <numbers>
#include <vector>

class Shape {
public:
    virtual ~Shape() = default;                        // required for deleting through Shape*
    virtual double area() const = 0;                   // pure virtual: Shape is abstract
    virtual const char* name() const { return "shape"; }
    void describe() const {                            // non-virtual, calls virtuals (template method)
        std::cout << name() << " with area " << area() << '\n';
    }
};

class Circle final : public Shape {                    // final: no further derivation
public:
    explicit Circle(double r) : r_{r} {}
    double area() const override { return std::numbers::pi * r_ * r_; }
    const char* name() const override { return "circle"; }
private:
    double r_;
};

class Rect : public Shape {
public:
    Rect(double w, double h) : w_{w}, h_{h} {}
    double area() const override { return w_ * h_; }
    const char* name() const override { return "rect"; }
protected:
    double w_, h_;
};

class Square : public Rect {
public:
    explicit Square(double s) : Rect{s, s} {}
    const char* name() const override { return "square"; }   // area() inherited from Rect
};

int main() {
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(1));
    shapes.push_back(std::make_unique<Rect>(2, 3));
    shapes.push_back(std::make_unique<Square>(2));
    for (const auto& s : shapes) s->describe();       // dynamic dispatch through the base

    std::cout << "sizeof(Rect) = " << sizeof(Rect) << " (two doubles + a vtable pointer)\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra virtual.cpp -o virtual && ./virtual
circle with area 3.14159
rect with area 6
square with area 4
sizeof(Rect) = 24 (two doubles + a vtable pointer)
```

### How dispatch works

Each class with virtual functions has a **vtable** — a static table of function pointers, one per virtual function. Each object holds a hidden **vptr** pointing to its class's vtable (the extra 8 bytes in `sizeof(Rect)`). A virtual call loads the vptr, loads the function pointer from the table, and calls it indirectly: two dependent loads and an indirect branch, which also prevents inlining unless the compiler can prove the dynamic type (as it can for `final` classes). The cost is small per call but matters in hot loops over millions of objects (Unit 9.5 measures it).

### Deep & Tricky

**DT-35. Without `override`, a typo creates a new function instead of overriding.** *(Guarantee: C++ rules; `override` is the fix)* `double area() const;` in the base and `double area();` (missing `const`) in the derived class are different functions; calls through the base silently use the base version. With `override`, the mismatch is a compile error. Compile with `-Wsuggest-override` to find overrides missing the keyword.

**DT-36. Deleting a derived object through a base pointer without a virtual destructor is undefined behaviour.** *(Guarantee: C++ standard)* In practice only the base destructor runs, leaking whatever the derived class owns. GCC warns (`-Wdelete-non-virtual-dtor`, in `-Wall`) when it can see the problem. Rule: a base class's destructor is either public and virtual, or protected and non-virtual.

<!-- expanded -->
### Going deeper: virtual calls inside constructors and destructors

While a base class constructor runs, the derived part of the object does not exist yet, so the object *is* a base object: virtual calls go to the base's version, never to the derived override. The same happens in reverse in destructors. Calling a pure virtual function this way is undefined behaviour; GCC's runtime then aborts with "pure virtual method called".

```cpp
// file: ctorvirtual.cpp
#include <iostream>

struct Base {
    Base() { std::cout << "Base ctor calls: "; describe(); }
    virtual ~Base() { std::cout << "Base dtor calls: "; describe(); }
    virtual void describe() const { std::cout << "Base::describe\n"; }
};
struct Derived : Base {
    int value = 42;
    Derived() { std::cout << "Derived ctor calls: "; describe(); }
    void describe() const override { std::cout << "Derived::describe, value " << value << '\n'; }
};

int main() { Derived d; std::cout << "--- end of main\n"; }
```

```console
$ g++-14 -std=c++20 -Wall -Wextra ctorvirtual.cpp -o ctorvirtual && ./ctorvirtual
Base ctor calls: Base::describe
Derived ctor calls: Derived::describe, value 42
--- end of main
Base dtor calls: Base::describe
```

If `Base`'s constructor *could* call `Derived::describe`, that function would read `value` before `Derived`'s constructor initialised it. When a base class needs a derived class's behaviour during construction, pass it in as a constructor argument, or use a two-step `create()` that constructs fully and then calls an `init()`.

### Mastery check

1. **Explain.** Why does `Square` print "square" but use `Rect`'s `area`?

## Unit 9.3 — Pitfalls of inheritance

**One-line summary:** **Slicing** copies only the base part of a derived object when it is passed or stored by value as a base. **Virtual calls in constructors and destructors** do not dispatch to the derived class (it doesn't exist yet, or anymore). **Name hiding** makes a derived member hide all base overloads with the same name. Know all three by sight.

```cpp
// file: pitfalls.cpp
#include <iostream>
#include <string>

class Base {
public:
    Base() { std::cout << "  Base() calls who(): " << who() << '\n'; }   // virtual call in a constructor
    virtual ~Base() = default;
    virtual std::string who() const { return "Base"; }
    void f(int) const { std::cout << "  Base::f(int)\n"; }
    void f(double) const { std::cout << "  Base::f(double)\n"; }
};

class Derived : public Base {
public:
    Derived() { std::cout << "  Derived() calls who(): " << who() << '\n'; }
    std::string who() const override { return "Derived"; }
    void f(const std::string&) const { std::cout << "  Derived::f(string)\n"; }   // hides BOTH Base::f
};

void by_value(Base b) { std::cout << "  by value: " << b.who() << '\n'; }      // slices
void by_ref(const Base& b) { std::cout << "  by reference: " << b.who() << '\n'; }

int main() {
    std::cout << "construction:\n";
    Derived d;
    std::cout << "slicing:\n";
    by_value(d);
    by_ref(d);
    std::cout << "hiding:\n";
    d.f(std::string("x"));
    // d.f(42);              error: Base::f(int) is hidden by Derived::f
    d.Base::f(42);          // still reachable explicitly ('using Base::f;' in Derived would unhide)
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra pitfalls.cpp -o pitfalls && ./pitfalls
construction:
  Base() calls who(): Base
  Derived() calls who(): Derived
slicing:
  by value: Base
  by reference: Derived
hiding:
  Derived::f(string)
  Base::f(int)
```

During `Base`'s constructor the object is still a `Base` — the `Derived` part has not been constructed — so `who()` dispatches to `Base::who`. Passing `d` by value to `by_value` constructed a brand-new `Base` from it (slicing), so the derived behaviour was lost. Defences: make polymorphic bases non-copyable (`Base(const Base&) = delete;` or abstract), pass them by reference or pointer, and add `using Base::f;` when a derived class adds overloads.

<!-- expanded -->
### Deep & Tricky

**DT-37. A derived class function with the same *name* as a base class function hides *every* base overload of that name, even ones with different parameters: `d.f(1)` stops finding `Base::f(int)` once `Derived` declares `f(std::string)`. Bring the base overloads back with `using Base::f;`.** *(Guarantee: [class.member.lookup] — name lookup stops at the first scope that declares the name; overload resolution happens only afterwards)* GCC warns with `-Woverloaded-virtual` for the virtual case (in `-Wall` since GCC 13).

```cpp
// file: hiding.cpp
#include <iostream>
#include <string>

struct Base {
    void f(int) { std::cout << "Base::f(int)\n"; }
    void f(double) { std::cout << "Base::f(double)\n"; }
};
struct Hides : Base {
    void f(const std::string&) { std::cout << "Hides::f(string)\n"; }
};
struct Unhides : Base {
    using Base::f;                                        // all Base::f overloads join this scope
    void f(const std::string&) { std::cout << "Unhides::f(string)\n"; }
};

int main() {
    Hides h;
    h.f("text");
    // h.f(1);                                            // error: no matching function, Base::f is hidden
    h.Base::f(1);                                         // still reachable when named explicitly
    Unhides u;
    u.f(1); u.f(2.5); u.f("text");
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra hiding.cpp -o hiding && ./hiding
Hides::f(string)
Base::f(int)
Base::f(int)
Base::f(double)
Unhides::f(string)
```

## Topic 9B — Alternatives to inheritance

## Unit 9.4 — `std::variant` and `std::visit`

**One-line summary:** `std::variant<A, B, C>` is a type-safe tagged union: it holds exactly one of its alternatives, stored inline (no heap allocation), and `std::visit` calls the right overload for whatever it holds — with the compiler checking that every alternative is handled. Use it when the set of types is **closed** (known and fixed); use virtual functions when it is **open** (others may add types).

```cpp
// file: variant.cpp
#include <iostream>
#include <numbers>
#include <string>
#include <variant>
#include <vector>

struct Circle { double r; };
struct Rect { double w, h; };
struct Triangle { double base, height; };
using Shape = std::variant<Circle, Rect, Triangle>;

template <typename... Fs> struct overloaded : Fs... { using Fs::operator()...; };   // the "overloaded" idiom

double area(const Shape& s) {
    return std::visit(overloaded{
        [](const Circle& c) { return std::numbers::pi * c.r * c.r; },
        [](const Rect& r) { return r.w * r.h; },
        [](const Triangle& t) { return 0.5 * t.base * t.height; },
    }, s);                                            // omit a case: compile error
}

int main() {
    std::vector<Shape> shapes{Circle{1}, Rect{2, 3}, Triangle{4, 5}};
    for (const auto& s : shapes)
        std::cout << "index " << s.index() << " area " << area(s) << '\n';
    if (const Rect* r = std::get_if<Rect>(&shapes[1])) std::cout << "shapes[1] is a Rect of width " << r->w << '\n';
    try {
        std::get<Circle>(shapes[1]);                  // wrong alternative: throws
    } catch (const std::bad_variant_access& e) {
        std::cout << "bad_variant_access: " << e.what() << '\n';
    }
    std::cout << "sizeof(Shape) = " << sizeof(Shape) << " (largest alternative + index)\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra variant.cpp -o variant && ./variant
index 0 area 3.14159
index 1 area 6
index 2 area 10
shapes[1] is a Rect of width 2
bad_variant_access: std::get: wrong index for variant
sizeof(Shape) = 24 (largest alternative + index)
```

The trade-off between the two designs is known as the **expression problem**: with virtual functions, adding a new *type* is easy (write a class) but adding a new *operation* means touching every class; with variants, adding a new *operation* is easy (write a visitor) but adding a new *type* means touching every visitor — which the compiler then lists for you.

<!-- expanded -->
### Going deeper: the overload pattern, and `valueless_by_exception`

`std::visit` needs one callable that accepts every alternative. The **overloaded** idiom builds it from several lambdas by inheriting all of their call operators: two lines of code that appear in most codebases using `variant`.

```cpp
// file: overloaded.cpp
#include <iostream>
#include <string>
#include <variant>
#include <vector>

template <typename... Fs> struct overloaded : Fs... { using Fs::operator()...; };

using Json = std::variant<std::nullptr_t, bool, double, std::string>;

int main() {
    std::vector<Json> values{nullptr, true, 3.5, std::string("hi")};
    for (const auto& v : values)
        std::visit(overloaded{
                       [](std::nullptr_t) { std::cout << "null\n"; },
                       [](bool b) { std::cout << (b ? "true" : "false") << '\n'; },
                       [](double d) { std::cout << d << '\n'; },
                       [](const std::string& s) { std::cout << '"' << s << "\"\n"; },
                   },
                   v);
    std::cout << "sizeof(Json) = " << sizeof(Json) << " (largest alternative std::string is " << sizeof(std::string)
              << ", plus the index)\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra overloaded.cpp -o overloaded && ./overloaded
null
true
3.5
"hi"
sizeof(Json) = 40 (largest alternative std::string is 32, plus the index)
```

A trap with `variant<bool, std::string>`: assigning `"text"` (a `const char*`) used to pick `bool`, because pointer-to-`bool` is a standard conversion and `const char*`-to-`std::string` is a user-defined one. C++20 (P0608) fixed this by forbidding narrowing and `bool` conversions in that choice; older compilers still pick `bool`. And if assigning a new alternative throws halfway, a variant can be left `valueless_by_exception()`, holding nothing — `std::visit` on it throws `std::bad_variant_access`.

## Unit 9.5 — Static polymorphism (CRTP) and type erasure

**One-line summary:** The **Curiously Recurring Template Pattern** (`class Derived : public Base<Derived>`) lets a base class call derived-class functions with no virtual dispatch, resolved at compile time. **Type erasure** wraps any type that supports an interface in a single value type with an internal virtual call — giving polymorphism *without* inheritance in user types, and with value semantics (copyable, storable by value). `std::function` and `std::any` are type erasure.

```cpp
// file: erasure.cpp
#include <chrono>
#include <iostream>
#include <memory>
#include <numbers>
#include <random>
#include <variant>
#include <vector>

// --- 1. classic virtual hierarchy
struct VShape { virtual ~VShape() = default; virtual double area() const = 0; };
struct VCircle : VShape { double r; explicit VCircle(double r) : r{r} {} double area() const override { return std::numbers::pi * r * r; } };
struct VRect : VShape { double w, h; VRect(double w, double h) : w{w}, h{h} {} double area() const override { return w * h; } };

// --- 2. variant
struct Circle { double r; double area() const { return std::numbers::pi * r * r; } };
struct Rect { double w, h; double area() const { return w * h; } };
using VarShape = std::variant<Circle, Rect>;

// --- 3. type erasure: any type with area() const, held by value
class AnyShape {
public:
    template <typename T>
    AnyShape(T shape) : self_{std::make_unique<Model<T>>(std::move(shape))} {}   // implicit: wraps anything shape-like
    AnyShape(const AnyShape& o) : self_{o.self_->clone()} {}
    AnyShape(AnyShape&&) noexcept = default;
    AnyShape& operator=(AnyShape o) noexcept { self_ = std::move(o.self_); return *this; }
    double area() const { return self_->area(); }
private:
    struct Concept { virtual ~Concept() = default; virtual double area() const = 0; virtual std::unique_ptr<Concept> clone() const = 0; };
    template <typename T> struct Model final : Concept {
        T obj;
        explicit Model(T o) : obj{std::move(o)} {}
        double area() const override { return obj.area(); }
        std::unique_ptr<Concept> clone() const override { return std::make_unique<Model>(obj); }
    };
    std::unique_ptr<Concept> self_;
};

// --- 4. CRTP: static dispatch to the derived class
template <typename Derived>
struct ShapeBase {
    double scaled_area(double k) const { return k * k * static_cast<const Derived&>(*this).area(); }
};
struct CCircle : ShapeBase<CCircle> { double r; explicit CCircle(double r) : r{r} {} double area() const { return std::numbers::pi * r * r; } };

template <typename F>
void bench(const char* name, F&& f) {
    auto t0 = std::chrono::steady_clock::now();
    double s = f();
    auto ms = std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
    std::cout << name << ms << " ms (sum " << s << ")\n";
}

int main() {
    const int n = 2'000'000;
    std::mt19937 rng{1};
    std::vector<std::unique_ptr<VShape>> vs;
    std::vector<VarShape> var;
    std::vector<AnyShape> anys;
    for (int i = 0; i < n; ++i) {
        double x = 1 + rng() % 10;
        if (rng() % 2) { vs.push_back(std::make_unique<VCircle>(x)); var.push_back(Circle{x}); anys.push_back(Circle{x}); }
        else { vs.push_back(std::make_unique<VRect>(x, x)); var.push_back(Rect{x, x}); anys.push_back(Rect{x, x}); }
    }
    bench("virtual (heap objects): ", [&] { double s = 0; for (const auto& p : vs) s += p->area(); return s; });
    bench("variant (contiguous):   ", [&] { double s = 0; for (const auto& v : var) s += std::visit([](const auto& x) { return x.area(); }, v); return s; });
    bench("type erasure:           ", [&] { double s = 0; for (const auto& a : anys) s += a.area(); return s; });
    std::cout << "CRTP scaled_area: " << CCircle{1}.scaled_area(2) << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra erasure.cpp -o erasure && ./erasure
virtual (heap objects): 21.5493 ms (sum 1.59235e+08)
variant (contiguous):   11.4784 ms (sum 1.59235e+08)
type erasure:           24.197 ms (sum 1.59235e+08)
CRTP scaled_area: 12.5664
```

The variant version is fastest here: its objects are stored contiguously in the vector (no pointer per element, no heap scatter), and `std::visit` dispatches with a jump on the index. The virtual and type-erased versions both chase a pointer to a separately allocated object per element, then make an indirect call — the memory layout, more than the dispatch itself, makes the difference (the cache lessons of the C book's Chapter 11 apply). With a few hundred objects the differences vanish; with millions in a hot loop they matter.

`AnyShape` shows type erasure's appeal: `Circle` and `Rect` know nothing about it — no base class, no `virtual` — yet any type with `area() const` can be stored, copied, and called through it. This is the design of `std::function`, `std::any`, `std::move_only_function` (C++23), and Sean Parent's "runtime polymorphism without inheritance".

### Pattern

**P-5. Choose the polymorphism mechanism by how the set of types changes.** *(Practice)* Open set extended by other code (plugins, library users): virtual interfaces (or type erasure, for value semantics). Closed set you control, many operations: `std::variant` + `std::visit`. Known at compile time, performance-critical: templates, concepts, CRTP. *Trade-off:* flexibility of adding types versus adding operations, compile-time versus run-time binding, and memory layout.

## Topic 9C — Types at run time

## Unit 9.6 — Casts and RTTI

**One-line summary:** C++ replaces C's all-powerful `(T)x` cast with four named casts that say what you mean: `static_cast` (well-defined conversions, including up/down a hierarchy without checks), `dynamic_cast` (checked downcast of polymorphic types), `const_cast` (remove `const` — only safe if the object wasn't originally `const`), `reinterpret_cast` (reinterpret bits: almost never right). Use `std::bit_cast` (C++20) to reinterpret an object's bytes safely.

```cpp
// file: casts.cpp
#include <bit>
#include <cstdint>
#include <iostream>
#include <memory>
#include <typeinfo>

struct Base { virtual ~Base() = default; };
struct Dog : Base { void bark() const { std::cout << "  woof\n"; } };
struct Cat : Base {};

void maybe_bark(const Base& b) {
    if (auto d = dynamic_cast<const Dog*>(&b))        // checked at run time: null if not a Dog
        d->bark();
    else
        std::cout << "  not a dog: " << typeid(b).name() << '\n';
}

int main() {
    Dog dog;
    Cat cat;
    maybe_bark(dog);
    maybe_bark(cat);

    double d = 3.9;
    int i = static_cast<int>(d);                      // explicit, visible conversion
    float f = 1.0f;
    auto bits = std::bit_cast<std::uint32_t>(f);      // the bytes of f as an integer: no aliasing UB
    std::cout << "static_cast<int>(3.9) = " << i << ", bits of 1.0f = 0x" << std::hex << bits << std::dec << '\n';

    const int frozen = 5;
    int* p = const_cast<int*>(&frozen);               // compiles ...
    (void)p;                                          // ... but *p = 6 would be undefined behaviour
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra casts.cpp -o casts && ./casts
  woof
  not a dog: 3Cat
static_cast<int>(3.9) = 3, bits of 1.0f = 0x3f800000
```

`dynamic_cast` uses **RTTI** (run-time type information) stored alongside the vtable; it costs a string or pointer comparison walk up the hierarchy, and it is a design smell when used to branch on type repeatedly (that is what virtual functions or `std::visit` are for). `typeid(x).name()` returns an implementation-specific mangled name (`3Cat` above; `c++filt -t` demangles it). Some projects disable RTTI (`-fno-rtti`) for size, which disables `dynamic_cast` and `typeid` on polymorphic types.

### Deep & Tricky

**DT-38. A C-style cast can silently become a `reinterpret_cast` or `const_cast`.** *(Guarantee: C++ cast rules)* `(Derived*)base_ptr` performs a `static_cast` if `Derived` is a known derived class — but if the class is only forward-declared at that point, the same syntax performs a `reinterpret_cast`, producing a wrong pointer under multiple inheritance. Named casts make the intent explicit and fail to compile in the dangerous cases. Compile with `-Wold-style-cast`.

<!-- expanded -->
### Going deeper: what `dynamic_cast` costs

`dynamic_cast` walks the RTTI (run-time type information) of the object's class to check the cast, which for deep hierarchies means string comparisons of type names in some ABIs. `static_cast` does no check at all. When the type is known from elsewhere — a tag field, a `kind()` virtual — the check is wasted work; when it is not known, `dynamic_cast` is the only safe option.

```cpp
// file: dyncost.cpp
#include <chrono>
#include <cstdio>
#include <memory>
#include <vector>

struct Base { virtual ~Base() = default; };
struct Mid : Base {};
struct Leaf : Mid { int v = 1; };
struct Other : Base {};

int main() {
    std::vector<std::unique_ptr<Base>> objs;
    for (int i = 0; i < 1000; ++i) {
        if (i % 2) objs.push_back(std::make_unique<Leaf>()); else objs.push_back(std::make_unique<Other>());
    }
    long hits = 0;
    auto t0 = std::chrono::steady_clock::now();
    for (int rep = 0; rep < 10000; ++rep)
        for (auto& p : objs) if (auto* l = dynamic_cast<Leaf*>(p.get())) hits += l->v;
    double ns = std::chrono::duration<double, std::nano>(std::chrono::steady_clock::now() - t0).count() / 1e7;
    std::printf("dynamic_cast: %ld hits, %s\n", hits, ns > 1.0 ? "more than 1 ns per cast" : "under 1 ns per cast");
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra dyncost.cpp -o dyncost && ./dyncost
dynamic_cast: 5000000 hits, more than 1 ns per cast
```

Each cast takes more than a nanosecond here — cheap once, noticeable in an inner loop over millions of objects. Compiling with `-fno-rtti` removes RTTI entirely (some embedded and game codebases do); `dynamic_cast` and `typeid` then stop compiling.

## Chapter 9 Project — An expression tree, three ways

**Goal:** build an arithmetic expression tree (numbers, variables, `+ - * /`, and function calls like `sqrt`) and evaluate and print it — implemented with a **virtual class hierarchy** and with **`std::variant`** — then compare the code, the extensibility, and the evaluation speed on a large expression.

### Reference solution: the virtual hierarchy

```cpp
// file: exprtree/expr_virtual.hpp
#pragma once
#include <cmath>
#include <map>
#include <memory>
#include <string>

namespace oo {
using Env = std::map<std::string, double>;

struct Expr {
    virtual ~Expr() = default;
    virtual double eval(const Env& env) const = 0;
    virtual std::string str() const = 0;
};
using ExprPtr = std::unique_ptr<Expr>;

struct Num final : Expr {
    double v;
    explicit Num(double v) : v{v} {}
    double eval(const Env&) const override { return v; }
    std::string str() const override { auto s = std::to_string(v); s.erase(s.find_last_not_of('0') + 1); if (s.back() == '.') s.pop_back(); return s; }
};
struct Var final : Expr {
    std::string name;
    explicit Var(std::string n) : name{std::move(n)} {}
    double eval(const Env& env) const override { return env.at(name); }
    std::string str() const override { return name; }
};
struct Bin final : Expr {
    char op;
    ExprPtr l, r;
    Bin(char o, ExprPtr a, ExprPtr b) : op{o}, l{std::move(a)}, r{std::move(b)} {}
    double eval(const Env& env) const override {
        double a = l->eval(env), b = r->eval(env);
        switch (op) { case '+': return a + b; case '-': return a - b; case '*': return a * b; default: return a / b; }
    }
    std::string str() const override { return "(" + l->str() + " " + op + " " + r->str() + ")"; }
};
struct Sqrt final : Expr {
    ExprPtr arg;
    explicit Sqrt(ExprPtr a) : arg{std::move(a)} {}
    double eval(const Env& env) const override { return std::sqrt(arg->eval(env)); }
    std::string str() const override { return "sqrt(" + arg->str() + ")"; }
};
}  // namespace oo
```

### Reference solution: the variant

```cpp
// file: exprtree/expr_variant.hpp
#pragma once
#include <cmath>
#include <map>
#include <memory>
#include <string>
#include <variant>

namespace vt {
using Env = std::map<std::string, double>;
struct Num; struct Var; struct Bin; struct Sqrt;
using Expr = std::variant<Num, Var, Bin, Sqrt>;     // closed set of node types

struct Num { double v; };
struct Var { std::string name; };
struct Bin { char op; std::unique_ptr<Expr> l, r; };  // recursion needs indirection
struct Sqrt { std::unique_ptr<Expr> arg; };

template <typename... Fs> struct overloaded : Fs... { using Fs::operator()...; };

inline double eval(const Expr& e, const Env& env) {
    return std::visit(overloaded{
        [](const Num& n) { return n.v; },
        [&](const Var& v) { return env.at(v.name); },
        [&](const Bin& b) {
            double a = eval(*b.l, env), c = eval(*b.r, env);
            switch (b.op) { case '+': return a + c; case '-': return a - c; case '*': return a * c; default: return a / c; }
        },
        [&](const Sqrt& s) { return std::sqrt(eval(*s.arg, env)); },
    }, e);
}

inline std::string str(const Expr& e) {                 // a second operation: just another visitor
    return std::visit(overloaded{
        [](const Num& n) { auto s = std::to_string(n.v); s.erase(s.find_last_not_of('0') + 1); if (s.back() == '.') s.pop_back(); return s; },
        [](const Var& v) { return v.name; },
        [](const Bin& b) { return "(" + str(*b.l) + " " + b.op + " " + str(*b.r) + ")"; },
        [](const Sqrt& s) { return "sqrt(" + str(*s.arg) + ")"; },
    }, e);
}

inline std::unique_ptr<Expr> num(double v) { return std::make_unique<Expr>(Num{v}); }
inline std::unique_ptr<Expr> var(std::string n) { return std::make_unique<Expr>(Var{std::move(n)}); }
inline std::unique_ptr<Expr> bin(char op, std::unique_ptr<Expr> a, std::unique_ptr<Expr> b) { return std::make_unique<Expr>(Bin{op, std::move(a), std::move(b)}); }
inline std::unique_ptr<Expr> sqrt_(std::unique_ptr<Expr> a) { return std::make_unique<Expr>(Sqrt{std::move(a)}); }
}  // namespace vt
```

```cpp
// file: exprtree/main.cpp
#include <chrono>
#include <iostream>
#include "expr_variant.hpp"
#include "expr_virtual.hpp"

// Build the same balanced tree of 'depth' levels in both representations.
oo::ExprPtr build_oo(int depth, int& leaf) {
    if (depth == 0) return (leaf++ % 3 == 0) ? oo::ExprPtr{std::make_unique<oo::Var>("x")} : oo::ExprPtr{std::make_unique<oo::Num>(leaf % 7 + 1)};
    const char ops[] = "+-*+";
    auto l = build_oo(depth - 1, leaf);
    auto r = build_oo(depth - 1, leaf);
    return std::make_unique<oo::Bin>(ops[depth % 4], std::move(l), std::move(r));
}
std::unique_ptr<vt::Expr> build_vt(int depth, int& leaf) {
    if (depth == 0) return (leaf++ % 3 == 0) ? vt::var("x") : vt::num(leaf % 7 + 1);
    const char ops[] = "+-*+";
    auto l = build_vt(depth - 1, leaf);
    auto r = build_vt(depth - 1, leaf);
    return vt::bin(ops[depth % 4], std::move(l), std::move(r));
}

template <typename F>
double timed(const char* label, F f) {
    auto t0 = std::chrono::steady_clock::now();
    double r = 0;
    for (int rep = 0; rep < 20; ++rep) r += f();
    std::cout << label << std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count() << " ms\n";
    return r;
}

int main() {
    // A small expression in both forms: sqrt((x * x) + 16) at x = 3
    oo::Sqrt e1{std::make_unique<oo::Bin>('+', std::make_unique<oo::Bin>('*', std::make_unique<oo::Var>("x"), std::make_unique<oo::Var>("x")), std::make_unique<oo::Num>(16))};
    auto e2 = vt::sqrt_(vt::bin('+', vt::bin('*', vt::var("x"), vt::var("x")), vt::num(16)));
    oo::Env env{{"x", 3}};
    std::cout << e1.str() << " = " << e1.eval(env) << " (virtual)\n";
    std::cout << vt::str(*e2) << " = " << vt::eval(*e2, env) << " (variant)\n";

    int leaf1 = 0, leaf2 = 0;
    auto big_oo = build_oo(18, leaf1);                  // 2^18 leaves, ~524k nodes
    auto big_vt = build_vt(18, leaf2);
    double a = timed("virtual tree, 20 evaluations: ", [&] { return big_oo->eval(env); });
    double b = timed("variant tree, 20 evaluations: ", [&] { return vt::eval(*big_vt, env); });
    std::cout << "results agree: " << std::boolalpha << (a == b) << '\n';
}
```

```console
$ cd exprtree && g++-14 -std=c++20 -O2 -Wall -Wextra main.cpp -o main && ./main
sqrt(((x * x) + 16)) = 5 (virtual)
sqrt(((x * x) + 16)) = 5 (variant)
virtual tree, 20 evaluations: 108.554 ms
variant tree, 20 evaluations: 170.406 ms
results agree: true
$ cd exprtree && g++-14 -std=c++20 -g -fsanitize=address,undefined main.cpp -o main_san && ./main_san | head -n 2
sqrt(((x * x) + 16)) = 5 (virtual)
sqrt(((x * x) + 16)) = 5 (variant)
```

Both representations print and evaluate identically. On the large tree their speeds are close: here, unlike the flat array of Unit 9.5, *both* trees allocate every node separately on the heap, so both pay the same pointer-chasing cost and the dispatch mechanism is a small part of the total. The real differences are in the code:

- **Adding an operation** (say, symbolic differentiation): the variant needs one new visitor function; the hierarchy needs a new virtual function in every class.
- **Adding a node type** (say, `Pow`): the hierarchy needs one new class; the variant needs the type added to the `std::variant` list, after which the compiler points to every visitor that doesn't handle it.
- **Memory layout:** an arena or a vector of nodes with indices instead of pointers (as in compilers' AST implementations) would speed up either version more than the choice between them.

### Review checklist

- Which representation would you choose for a compiler's AST (closed set of nodes, many passes)? For a plugin system?
- Does every node own its children exclusively, and is the tree freed without leaks (check under ASan)?
- What happens to evaluation on a degenerate, deeply nested (linked-list-shaped) tree of a million nodes? (Recursion depth: stack overflow — the same limit as Chapter 6's parser.)

## Answers to the Chapter 9 prediction questions

There are no prediction questions in this chapter.

# Chapter 10 — Concurrency in Modern C++

*Goal: write correct, efficient concurrent C++ with the standard library — threads that stop cleanly, locks that cannot leak or deadlock, condition variables, atomics, futures, parallel algorithms, and coroutines.*

C++11 gave C++ a memory model and a portable threading library; C++17 added parallel algorithms; C++20 added `std::jthread`, stop tokens, latches, barriers, semaphores, atomic waiting, and coroutines. The rules about data races and memory ordering are the same as in C (Chapter 9 of the C book explains them in depth with ThreadSanitizer); what C++ adds is RAII for every synchronisation primitive and higher-level abstractions that remove whole classes of mistakes.

**What you will be able to do after this chapter**

- Start threads with `std::jthread` and stop them cooperatively with `std::stop_token`.
- Protect shared data with mutexes and RAII locks, including deadlock-free multi-lock with `std::scoped_lock`.
- Coordinate threads with condition variables, latches, barriers, and semaphores.
- Use `std::atomic` and the memory orders, including C++20 `wait`/`notify`.
- Run tasks with `std::async`, futures, and promises, and know their pitfalls.
- Parallelise standard algorithms with execution policies.
- Explain C++20 coroutines and use `std::generator`.

Programs are compiled with `-pthread`; thread-safety claims are checked with `-fsanitize=thread`.

## Topic 10A — Threads and locks

## Unit 10.1 — `std::thread` and `std::jthread`

**One-line summary:** `std::thread` runs a callable on a new thread and must be joined or detached before it is destroyed (otherwise `std::terminate`). C++20's `std::jthread` joins automatically in its destructor and carries a **stop token** for cooperative cancellation — use it by default.

```cpp
// file: jthread.cpp
#include <chrono>
#include <iostream>
#include <stop_token>
#include <thread>
#include <vector>

using namespace std::chrono_literals;

void worker(std::stop_token st, int id, int& progress) {     // stop_token injected by jthread
    while (!st.stop_requested()) {
        ++progress;
        std::this_thread::sleep_for(10ms);
    }
    std::cout << "  worker " << id << " stopping after " << progress << " steps\n";
}

int main() {
    std::vector<int> progress(3, 0);                         // each thread writes only its own slot
    {
        std::vector<std::jthread> threads;
        for (int i = 0; i < 3; ++i)
            threads.emplace_back(worker, i, std::ref(progress[i]));   // std::ref: pass by reference
        std::this_thread::sleep_for(55ms);
        std::cout << "requesting stop\n";
    }                                                          // ~jthread: request_stop() + join()
    std::cout << "all joined; hardware threads: " << std::thread::hardware_concurrency() << '\n';

    std::thread plain{[] {}};
    plain.join();                                              // forgetting this would terminate the program
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -pthread jthread.cpp -o jthread && ./jthread
requesting stop
  worker 0 stopping after 6 steps
  worker 1 stopping after 6 steps
  worker 2 stopping after 7 steps
all joined; hardware threads: 2
```

Arguments to thread constructors are **copied** into the new thread; wrap references in `std::ref` (and make sure the referenced object outlives the thread). Stop tokens can also wake blocked waits: `std::condition_variable_any::wait(lock, stop_token, pred)` returns when a stop is requested.

<!-- expanded -->
### Going deeper: arguments are copied into the thread

`std::thread t(f, a, b)` *copies* (or moves) `a` and `b` into storage owned by the new thread, even when `f` takes references. To pass a real reference, wrap it with `std::ref`; and then the referenced object must outlive the thread. A `std::thread` that is still joinable when destroyed calls `std::terminate` — `std::jthread` joins in its destructor instead.

```cpp
// file: threadargs.cpp
#include <functional>
#include <iostream>
#include <thread>

void add_ten(int& x) { x += 10; }

int main() {
    int counter = 0;
    // std::thread bad(add_ten, counter);       // error: cannot bind a copied int to int&
    std::jthread t(add_ten, std::ref(counter));  // a reference to main's counter
    t.join();
    std::cout << "counter = " << counter << '\n';

    try {
        std::thread forgot([] {});
        forgot.detach();                         // detached threads may not touch main's locals after main ends
    } catch (...) {}
    std::cout << "hardware threads: " << (std::thread::hardware_concurrency() > 0 ? "reported" : "unknown") << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra threadargs.cpp -o threadargs && ./threadargs
counter = 10
hardware threads: reported
$ printf '#include <thread>\nint main(){ std::thread t([]{}); }\n' > nojoin.cpp && g++-14 -std=c++20 nojoin.cpp -o nojoin && ./nojoin; echo "exit status $?"
terminate called without an active exception
bash: line 1: 17552 Aborted                 ./nojoin
exit status 134
```

The second program destroyed a joinable `std::thread` and was terminated (status 134). Prefer `std::jthread`.

### Mastery check

1. **Explain.** What happens if a `std::thread` object is destroyed while still joinable, and why did the standard choose that behaviour?

## Unit 10.2 — Mutexes and RAII locks

**One-line summary:** Protect shared data with `std::mutex`, and never call `lock`/`unlock` directly: `std::lock_guard` (simplest), `std::unique_lock` (movable, unlockable, needed for condition variables), and `std::scoped_lock` (locks several mutexes at once without deadlock). `std::shared_mutex` with `std::shared_lock` allows concurrent readers.

```cpp
// file: locks.cpp
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <string>
#include <thread>
#include <unordered_map>
#include <vector>

class Account {
public:
    explicit Account(long long b) : balance_{b} {}
    friend void transfer(Account& from, Account& to, long long amount) {
        std::scoped_lock lock{from.m_, to.m_};          // both, in a deadlock-free order
        from.balance_ -= amount;
        to.balance_ += amount;
    }
    long long balance() const { std::lock_guard g{m_}; return balance_; }
private:
    mutable std::mutex m_;                              // mutable: lockable in const functions
    long long balance_;
};

class Config {                                          // read-mostly data
public:
    std::string get(const std::string& k) const {
        std::shared_lock lock{m_};                      // many readers at once
        auto it = values_.find(k);
        return it == values_.end() ? "" : it->second;
    }
    void set(const std::string& k, std::string v) {
        std::unique_lock lock{m_};                      // one writer, no readers
        values_[k] = std::move(v);
    }
private:
    mutable std::shared_mutex m_;
    std::unordered_map<std::string, std::string> values_;
};

int main() {
    Account a{1000}, b{1000};
    {
        std::vector<std::jthread> ts;
        for (int i = 0; i < 4; ++i)
            ts.emplace_back([&, i] {
                for (int k = 0; k < 10000; ++k)
                    i % 2 ? transfer(a, b, 1) : transfer(b, a, 1);   // opposite directions: classic deadlock risk
            });
    }
    std::cout << "a=" << a.balance() << " b=" << b.balance() << " total=" << a.balance() + b.balance() << '\n';

    Config cfg;
    cfg.set("mode", "fast");
    std::vector<std::jthread> readers;
    for (int i = 0; i < 4; ++i) readers.emplace_back([&] { for (int k = 0; k < 1000; ++k) (void)cfg.get("mode"); });
    readers.clear();
    std::cout << "mode=" << cfg.get("mode") << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -pthread -fsanitize=thread locks.cpp -o locks && ./locks
a=1000 b=1000 total=2000
mode=fast
```

Transfers in opposite directions from four threads never deadlocked, because `std::scoped_lock` acquires multiple mutexes with a deadlock-avoidance algorithm (the "lock ordering" of the C book's Unit 9.4, done for you). ThreadSanitizer reported nothing.

### Deep & Tricky

**DT-39. `std::lock_guard(m_);` without a variable name locks and immediately unlocks.** *(Guarantee: C++ parsing rules)* Writing `std::lock_guard<std::mutex>(m_);` declares a temporary (or, depending on syntax, a new variable named `m_`!) that is destroyed at the end of the statement — the critical section is not protected at all. Always name the guard: `std::lock_guard g{m_};`. Clang warns in some cases; code review must catch the rest.

<!-- expanded -->
### Going deeper: locking two mutexes without deadlock

Transferring between two accounts needs both accounts' mutexes. If one thread locks A then B while another locks B then A, each can hold one and wait forever for the other: a **deadlock**. `std::scoped_lock(a, b)` locks any number of mutexes using a deadlock-avoidance algorithm (try, back off, retry), so the order you list them in does not matter.

```cpp
// file: transfer.cpp
#include <iostream>
#include <mutex>
#include <thread>
#include <vector>

struct Account { std::mutex m; long balance = 1000; };

void transfer(Account& from, Account& to, long amount) {
    std::scoped_lock lock(from.m, to.m);       // both, deadlock-free, whatever the order
    from.balance -= amount;
    to.balance += amount;
}

int main() {
    Account a, b;
    std::vector<std::jthread> ts;
    for (int i = 0; i < 4; ++i) {
        ts.emplace_back([&] { for (int k = 0; k < 100000; ++k) transfer(a, b, 1); });
        ts.emplace_back([&] { for (int k = 0; k < 100000; ++k) transfer(b, a, 1); });
    }
    ts.clear();                                // joins every thread
    std::cout << "a " << a.balance << ", b " << b.balance << ", total " << a.balance + b.balance << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra transfer.cpp -o transfer && timeout 20 ./transfer
a 1000, b 1000, total 2000
```

Replacing the `scoped_lock` with two `lock_guard`s (`from.m` first, then `to.m`) makes this program hang within moments on a multi-core machine. Another standard tool for read-mostly data is `std::shared_mutex`: many readers hold `std::shared_lock` at once, a writer takes `std::unique_lock` alone.

### Mastery check

1. **Explain.** Why is `m_` declared `mutable`?

## Unit 10.3 — Condition variables, latches, barriers, and semaphores

**One-line summary:** `std::condition_variable` lets threads wait for a condition on shared state; always use the overload with a **predicate** (`cv.wait(lock, [&]{ return ready; })`), which handles spurious wake-ups. C++20 adds simpler tools for common patterns: `std::latch` (wait until N events happen once), `std::barrier` (reusable phase synchronisation), and `std::counting_semaphore` (limit concurrent access).

```cpp
// file: sync.cpp
#include <barrier>
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <latch>
#include <mutex>
#include <queue>
#include <semaphore>
#include <thread>
#include <vector>

template <typename T>
class BlockingQueue {
public:
    void push(T v) {
        { std::lock_guard g{m_}; q_.push(std::move(v)); }
        cv_.notify_one();                                   // notify after unlocking: the waiter can run at once
    }
    bool pop(T& out) {                                      // false when closed and empty
        std::unique_lock lock{m_};
        cv_.wait(lock, [&] { return !q_.empty() || closed_; });   // predicate: no lost or spurious wake-ups
        if (q_.empty()) return false;
        out = std::move(q_.front());
        q_.pop();
        return true;
    }
    void close() { { std::lock_guard g{m_}; closed_ = true; } cv_.notify_all(); }
private:
    std::mutex m_;
    std::condition_variable cv_;
    std::queue<T> q_;
    bool closed_ = false;
};

int main() {
    BlockingQueue<int> q;
    long long sum = 0;
    std::mutex sum_m;
    {
        std::vector<std::jthread> consumers;
        for (int i = 0; i < 3; ++i)
            consumers.emplace_back([&] { int v; long long local = 0; while (q.pop(v)) local += v; std::lock_guard g{sum_m}; sum += local; });
        for (int v = 1; v <= 1000; ++v) q.push(v);
        q.close();
    }
    std::cout << "queue: sum " << sum << " (expected 500500)\n";

    std::latch ready{3};                                     // one-shot: count down to zero
    std::barrier phase{3, []() noexcept { std::cout << "  -- phase complete --\n"; }};
    std::counting_semaphore<2> slots{2};                     // at most 2 threads inside at once
    std::atomic<int> inside{0}, max_inside{0};
    {
        std::vector<std::jthread> ts;
        for (int id = 0; id < 3; ++id)
            ts.emplace_back([&, id] {
                ready.count_down();
                ready.wait();                                // everyone starts together
                for (int p = 0; p < 2; ++p) {
                    slots.acquire();
                    int now = ++inside;
                    for (int m = max_inside; now > m && !max_inside.compare_exchange_weak(m, now);) {}
                    std::this_thread::sleep_for(std::chrono::milliseconds(5));
                    --inside;
                    slots.release();
                    phase.arrive_and_wait();                 // nobody starts phase p+1 early
                }
                (void)id;
            });
    }
    std::cout << "max threads inside the semaphore at once: " << max_inside << " (limit 2)\n";
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -pthread -fsanitize=thread sync.cpp -o sync && ./sync
queue: sum 500500 (expected 500500)
  -- phase complete --
  -- phase complete --
max threads inside the semaphore at once: 2 (limit 2)
```

<!-- expanded -->
### Going deeper: spurious wake-ups, and why the predicate matters

`condition_variable::wait` may return even though nobody called `notify` (a **spurious wake-up**, allowed by the standard and by POSIX), and a notification sent before anyone waits is lost. Both problems disappear with the predicate form `cv.wait(lock, [&]{ return condition; })`, which checks the condition before sleeping and again after every wake-up. Here is a bounded queue built that way — the core of every producer/consumer program.

```cpp
// file: boundedq.cpp
#include <condition_variable>
#include <deque>
#include <iostream>
#include <mutex>
#include <optional>
#include <thread>
#include <vector>

template <typename T> class BoundedQueue {
public:
    explicit BoundedQueue(std::size_t cap) : cap_(cap) {}
    void push(T v) {
        std::unique_lock lk(m_);
        not_full_.wait(lk, [&] { return q_.size() < cap_; });
        q_.push_back(std::move(v));
        not_empty_.notify_one();
    }
    std::optional<T> pop() {                   // empty optional once closed and drained
        std::unique_lock lk(m_);
        not_empty_.wait(lk, [&] { return !q_.empty() || closed_; });
        if (q_.empty()) return std::nullopt;
        T v = std::move(q_.front());
        q_.pop_front();
        not_full_.notify_one();
        return v;
    }
    void close() { { std::lock_guard lk(m_); closed_ = true; } not_empty_.notify_all(); }
private:
    std::mutex m_;
    std::condition_variable not_full_, not_empty_;
    std::deque<T> q_;
    std::size_t cap_;
    bool closed_ = false;
};

int main() {
    BoundedQueue<int> q(4);
    long total = 0;
    std::mutex tm;
    {
        std::vector<std::jthread> consumers;
        for (int c = 0; c < 3; ++c)
            consumers.emplace_back([&] {
                long local = 0;
                while (auto v = q.pop()) local += *v;
                std::lock_guard lk(tm);
                total += local;
            });
        for (int i = 1; i <= 10000; ++i) q.push(i);
        q.close();
    }                                          // consumers join here
    std::cout << "sum " << total << " (expected " << 10000L * 10001 / 2 << ")\n";
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra boundedq.cpp -o boundedq && ./boundedq
sum 50005000 (expected 50005000)
$ g++-14 -std=c++20 -O1 -g -fsanitize=thread boundedq.cpp -o boundedq_tsan && ./boundedq_tsan 2>&1 | grep -c "WARNING: ThreadSanitizer" | sed 's/$/ ThreadSanitizer warnings/'
0 ThreadSanitizer warnings
```

## Topic 10B — Lock-free tools and tasks

## Unit 10.4 — `std::atomic` and the memory model

**One-line summary:** `std::atomic<T>` gives race-free reads, writes, and read-modify-write operations on a single variable, with the memory orders of the C++ memory model (`relaxed`, `acquire`, `release`, `acq_rel`, `seq_cst` — identical to C11's, Unit 9.5 of the C book). C++20 adds `wait`/`notify_one` on atomics (efficient blocking without a mutex) and `std::atomic_ref` for atomic access to ordinary objects.

```cpp
// file: atomics.cpp
#include <atomic>
#include <iostream>
#include <thread>
#include <vector>

int main() {
    std::atomic<long> hits{0};
    {
        std::vector<std::jthread> ts;
        for (int i = 0; i < 4; ++i)
            ts.emplace_back([&] { for (int k = 0; k < 100000; ++k) hits.fetch_add(1, std::memory_order_relaxed); });
    }
    std::cout << "hits " << hits << ", lock-free: " << std::boolalpha << hits.is_lock_free() << '\n';

    std::atomic<int> state{0};
    std::jthread waiter{[&] {
        state.wait(0);                                   // sleeps until state != 0 (futex on Linux, no spinning)
        std::cout << "waiter woke with state " << state.load() << '\n';
    }};
    state.store(42);
    state.notify_one();

    int plain_counter = 0;
    {
        std::vector<std::jthread> ts;
        for (int i = 0; i < 4; ++i)
            ts.emplace_back([&] { std::atomic_ref ref{plain_counter}; for (int k = 0; k < 1000; ++k) ++ref; });
    }
    std::cout << "plain int updated atomically via atomic_ref: " << plain_counter << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra -g -pthread -fsanitize=thread atomics.cpp -o atomics && ./atomics
hits 400000, lock-free: true
waiter woke with state 42
plain int updated atomically via atomic_ref: 4000
```

### Deep & Tricky

**DT-40. `std::atomic<T>` is not lock-free for every `T`.** *(Implementation behaviour)* For large or oddly sized types, the library falls back to an internal lock (`is_lock_free()` returns false), and on some platforms that requires linking `-latomic`. Atomics of `std::shared_ptr` (`std::atomic<std::shared_ptr<T>>`, C++20) are usually implemented with a lock. Check `is_always_lock_free` where it matters (signal handlers, lock-free algorithms).

<!-- expanded -->
### Going deeper: release and acquire, demonstrated

`memory_order_release` on a store and `memory_order_acquire` on a load of the same atomic create a *happens-before* edge: everything the writing thread did before the release store is visible to a reading thread after its acquire load sees the stored value. That is all a one-shot "data is ready" flag needs, and it is cheaper than the default `seq_cst` on ARM (on x86 the two compile to the same instructions for loads and stores, but the compiler is still forbidden from reordering).

```cpp
// file: relacq.cpp
#include <atomic>
#include <cassert>
#include <iostream>
#include <string>
#include <thread>

std::string payload;                       // ordinary, non-atomic data
std::atomic<bool> ready{false};

int main() {
    int checks = 0;
    for (int round = 0; round < 2000; ++round) {
        payload.clear();
        ready.store(false, std::memory_order_relaxed);
        std::jthread producer([] {
            payload = "result of a long computation";
            ready.store(true, std::memory_order_release);      // publishes payload
        });
        std::jthread consumer([&] {
            while (!ready.load(std::memory_order_acquire)) {}  // spin until published
            assert(payload == "result of a long computation"); // guaranteed to see it
            ++checks;
        });
    }
    std::cout << checks << " rounds, payload always visible\n";
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra relacq.cpp -o relacq && ./relacq
2000 rounds, payload always visible
$ g++-14 -std=c++20 -O1 -g -fsanitize=thread relacq.cpp -o relacq_tsan && ./relacq_tsan 2>&1 | grep -c "data race" | sed 's/$/ data races reported/'
0 data races reported
$ sed 's/memory_order_acquire/memory_order_relaxed/' relacq.cpp > relaxed.cpp && g++-14 -std=c++20 -O1 -g -fsanitize=thread relaxed.cpp -o relaxed_tsan && ./relaxed_tsan 2>&1 | grep -m1 -oE "WARNING: ThreadSanitizer: data race"
WARNING: ThreadSanitizer: data race
```

ThreadSanitizer accepts the release/acquire version and reports a data race on `payload` as soon as the load is weakened to `relaxed` — even on x86, where the relaxed version would probably "work", because the C++ memory model, not the CPU, decides what is correct.

## Unit 10.5 — Tasks: `std::async`, futures, and promises

**One-line summary:** `std::async` runs a function (on a new thread with `std::launch::async`) and returns a `std::future` from which the result — or the exception — can be retrieved with `get()`. `std::promise` lets any code fulfil a future. Pitfalls: a future from `std::async` **blocks in its destructor** until the task finishes, and the default launch policy may defer execution until `get()`.

```cpp
// file: futures.cpp
#include <chrono>
#include <future>
#include <iostream>
#include <stdexcept>
#include <thread>
#include <vector>

long long count_primes(int from, int to) {
    long long n = 0;
    for (int x = std::max(2, from); x < to; ++x) {
        bool prime = true;
        for (int d = 2; d * d <= x && prime; ++d) prime = x % d != 0;
        n += prime;
    }
    return n;
}

int main() {
    using clock = std::chrono::steady_clock;
    const int limit = 3'000'000;
    auto t0 = clock::now();
    long long serial = count_primes(0, limit);
    auto t1 = clock::now();

    std::vector<std::future<long long>> parts;
    const int chunks = 8;
    for (int c = 0; c < chunks; ++c)
        parts.push_back(std::async(std::launch::async, count_primes, limit / chunks * c, limit / chunks * (c + 1)));
    long long parallel = 0;
    for (auto& f : parts) parallel += f.get();             // get() waits; rethrows exceptions
    auto t2 = clock::now();
    auto ms = [](auto a, auto b) { return std::chrono::duration<double, std::milli>(b - a).count(); };
    std::cout << "primes below " << limit << ": serial " << serial << " in " << ms(t0, t1) << " ms, async "
              << parallel << " in " << ms(t1, t2) << " ms\n";

    auto failing = std::async(std::launch::async, [] () -> int { throw std::runtime_error("task failed"); });
    try { failing.get(); } catch (const std::exception& e) { std::cout << "exception from task: " << e.what() << '\n'; }

    std::promise<std::string> p;
    std::future<std::string> answer = p.get_future();
    std::jthread producer{[&p] { p.set_value("value set by another thread"); }};
    std::cout << answer.get() << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra -pthread futures.cpp -o futures && ./futures
primes below 3000000: serial 216816 in 545.525 ms, async 216816 in 284.389 ms
exception from task: task failed
value set by another thread
```

On this 2-core virtual machine the eight async tasks finished faster than the serial loop, but by less than the ideal 2×: across repeated runs the speed-up varied from about 1.2× to 1.9×, because a cloud VM's cores are shared with other workloads and the chunks are unequal (larger numbers take longer to test, so the last chunks finish last). Measure several runs before drawing conclusions about parallel speed-up. Exceptions thrown inside a task are captured and rethrown by `get()`, so error handling works across threads. `std::async` creates a thread per call; for many small tasks, use a thread pool (this chapter's project).

<!-- expanded -->
### Deep & Tricky

**DT-41. The `std::future` returned by `std::async(std::launch::async, ...)` blocks in its destructor until the task finishes. Ignoring the return value therefore runs the "asynchronous" tasks one after another: `std::async(std::launch::async, f); std::async(std::launch::async, g);` waits for `f` before starting `g`.** *(Guarantee: [futures.async] — the shared state's last future "blocks until the asynchronous operation completes"; this applies only to futures from `std::async`)* Keep the futures in variables, or use `std::jthread`.

```cpp
// file: asyncblock.cpp
#include <chrono>
#include <future>
#include <iostream>
#include <thread>

using namespace std::chrono;

void work() { std::this_thread::sleep_for(200ms); }

int main() {
    auto t0 = steady_clock::now();
    std::async(std::launch::async, work);          // temporary future: destructor waits here
    std::async(std::launch::async, work);
    auto serial = duration_cast<milliseconds>(steady_clock::now() - t0).count();

    t0 = steady_clock::now();
    {
        auto f1 = std::async(std::launch::async, work);
        auto f2 = std::async(std::launch::async, work);
    }
    auto parallel = duration_cast<milliseconds>(steady_clock::now() - t0).count();
    std::cout << "discarded futures: " << (serial >= 390 ? "about 400 ms (serial)" : "?") << '\n'
              << "kept futures:      " << (parallel < 390 ? "about 200 ms (parallel)" : "?") << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra asyncblock.cpp -o asyncblock 2>&1 | grep -c "warning: ignoring return value" | sed 's/$/ warnings about the discarded futures/'; ./asyncblock
2 warnings about the discarded futures
discarded futures: about 400 ms (serial)
kept futures:      about 200 ms (parallel)
```

`std::async`'s result is marked `[[nodiscard]]` in libstdc++, so GCC warns about both discarded calls.

## Unit 10.6 — Parallel algorithms

**One-line summary:** Most standard algorithms accept an **execution policy** (C++17): `std::execution::seq`, `par` (parallel), `par_unseq` (parallel and vectorised), `unseq` (C++20, vectorised). With libstdc++, parallel policies are implemented with Intel TBB, so link with `-ltbb`. Element access functions must be free of data races and, for `par_unseq`, must not take locks.

```cpp
// file: parallel.cpp
#include <algorithm>
#include <chrono>
#include <execution>
#include <iostream>
#include <numeric>
#include <random>
#include <vector>

template <typename F>
double ms(F f) {
    auto t0 = std::chrono::steady_clock::now();
    f();
    return std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
}

int main() {
    std::vector<double> data(20'000'000);
    std::mt19937_64 rng{7};
    std::uniform_real_distribution<double> dist{0, 1};
    for (auto& x : data) x = dist(rng);
    auto a = data, b = data;

    double t_seq = ms([&] { std::sort(std::execution::seq, a.begin(), a.end()); });
    double t_par = ms([&] { std::sort(std::execution::par, b.begin(), b.end()); });
    double s1 = 0, s2 = 0;
    double r_seq = ms([&] { s1 = std::transform_reduce(std::execution::seq, data.begin(), data.end(), 0.0, std::plus<>{}, [](double x) { return x * x; }); });
    double r_par = ms([&] { s2 = std::transform_reduce(std::execution::par_unseq, data.begin(), data.end(), 0.0, std::plus<>{}, [](double x) { return x * x; }); });
    std::cout << "sort 20M doubles: seq " << t_seq << " ms, par " << t_par << " ms; sorted equal: " << std::boolalpha << (a == b) << '\n';
    std::cout << "sum of squares: seq " << r_seq << " ms, par_unseq " << r_par << " ms; |difference| = " << std::abs(s1 - s2) << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra parallel.cpp -o parallel -ltbb && ./parallel
sort 20M doubles: seq 1593.8 ms, par 1084.11 ms; sorted equal: true
sum of squares: seq 23.3609 ms, par_unseq 6.63611 ms; |difference| = 1.41561e-07
```

The parallel sort gained little on two cores (sorting is limited by memory bandwidth and TBB's merging overhead), while the parallel, vectorised reduction was about twice as fast. The two sums differ in the last digits: a parallel reduction adds in a different order, and floating-point addition is not associative (the C book, Chapter 11) — `std::reduce` and `transform_reduce` are allowed to reorder, unlike `std::accumulate`.

## Topic 10C — Coroutines

## Unit 10.7 — Coroutines and `std::generator`

**One-line summary:** A **coroutine** is a function that can suspend (`co_await`, `co_yield`) and later resume, keeping its local state in a heap-allocated frame. C++20 provides the language machinery but almost no library types; C++23 adds `std::generator<T>` for lazy sequences. Asynchronous I/O frameworks (Asio, cppcoro, folly) supply task types for `co_await`-based concurrency.

```cpp
// file: coro.cpp
#include <generator>
#include <utility>
#include <iostream>
#include <ranges>
#include <string>
#include <string_view>

std::generator<long long> fibonacci() {                  // an infinite lazy sequence
    long long a = 0, b = 1;
    while (true) {
        co_yield a;                                       // suspend, hand out a, resume here next time
        a = std::exchange(b, a + b);
    }
}

std::generator<std::string_view> words(std::string_view text) {
    std::size_t i = 0;
    while (i < text.size()) {
        while (i < text.size() && text[i] == ' ') ++i;
        std::size_t j = i;
        while (j < text.size() && text[j] != ' ') ++j;
        if (j > i) co_yield text.substr(i, j - i);
        i = j;
    }
}

int main() {
    for (long long f : fibonacci() | std::views::take(15)) std::cout << f << ' ';
    std::cout << '\n';
    for (auto w : words("  coroutines suspend   and resume ")) std::cout << '[' << w << ']';
    std::cout << '\n';
}
```

```console
$ g++-14 -std=c++23 -Wall -Wextra coro.cpp -o coro && ./coro
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 
[coroutines][suspend][and][resume]
```

Each call to `fibonacci()` creates a coroutine frame holding `a` and `b`; `co_yield` suspends it and hands a value to the consumer; the next iteration resumes it exactly where it stopped. Writing the same as an iterator class would need explicit state and a `next()` method. The machinery underneath — promise types, awaitables, `coroutine_handle` — is what library authors implement; most programmers use the resulting types.

### Deep & Tricky

**DT-42. A coroutine's reference parameters can dangle.** *(Guarantee: coroutine parameter rules)* Parameters are copied into the coroutine frame, but a *reference* parameter copies only the reference. `words(std::string("temp"))` would store a `string_view` into a temporary that dies before the first resume. Take parameters by value in coroutines unless the caller's object definitely outlives the coroutine.

<!-- expanded -->
### Going deeper: what a coroutine is made of

A function is a coroutine if its body uses `co_await`, `co_yield`, or `co_return`. The compiler turns it into a state machine whose local variables live in a heap-allocated **coroutine frame**, and asks the *return type* — through its nested `promise_type` — what to do at each step. Writing a minimal generator by hand shows every piece that `std::generator` (C++23) packages up for you.

```cpp
// file: minigen.cpp
#include <coroutine>
#include <exception>
#include <iostream>
#include <utility>

struct IntGen {
    struct promise_type {
        int current = 0;
        IntGen get_return_object() { return IntGen{std::coroutine_handle<promise_type>::from_promise(*this)}; }
        std::suspend_always initial_suspend() noexcept { return {}; }   // start suspended
        std::suspend_always final_suspend() noexcept { return {}; }     // keep the frame until destroyed
        std::suspend_always yield_value(int v) noexcept { current = v; return {}; }
        void return_void() noexcept {}
        void unhandled_exception() { std::terminate(); }
    };
    explicit IntGen(std::coroutine_handle<promise_type> h) : h_(h) {}
    IntGen(IntGen&& o) noexcept : h_(std::exchange(o.h_, {})) {}
    ~IntGen() { if (h_) h_.destroy(); }                                  // frees the coroutine frame
    bool next() { h_.resume(); return !h_.done(); }
    int value() const { return h_.promise().current; }
private:
    std::coroutine_handle<promise_type> h_;
};

IntGen fib(int n) {
    int a = 0, b = 1;                     // these locals live in the coroutine frame
    for (int i = 0; i < n; ++i) {
        co_yield a;
        int t = a + b; a = b; b = t;
    }
}

int main() {
    auto g = fib(10);
    while (g.next()) std::cout << g.value() << ' ';
    std::cout << '\n';
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra minigen.cpp -o minigen && ./minigen
0 1 1 2 3 5 8 13 21 34 
$ g++-14 -std=c++20 -g -fsanitize=address minigen.cpp -o minigen_asan && ./minigen_asan | wc -w | sed 's/$/ numbers, no leak reported/'
10 numbers, no leak reported
```

The move constructor uses `std::exchange` (from `<utility>`) so that the moved-from generator no longer owns the frame. Forgetting `h_.destroy()` leaks the frame — AddressSanitizer would report it.

## Chapter 10 Project — A thread pool with futures and cancellation

**Goal:** a reusable `ThreadPool` that runs submitted callables on a fixed set of `std::jthread` workers and returns `std::future`s (with results and exceptions), supports waiting for all work, and shuts down promptly — then use it to parallelise a computation and prove correctness under ThreadSanitizer.

### Reference solution

```cpp
// file: pool/thread_pool.hpp
#pragma once
#include <condition_variable>
#include <deque>
#include <functional>
#include <future>
#include <mutex>
#include <thread>
#include <type_traits>
#include <vector>

class ThreadPool {
public:
    explicit ThreadPool(unsigned n = std::thread::hardware_concurrency()) {
        for (unsigned i = 0; i < (n ? n : 1); ++i)
            workers_.emplace_back([this](std::stop_token st) { run(st); });
    }
    ~ThreadPool() {
        for (auto& w : workers_) w.request_stop();
        cv_.notify_all();                                     // wake idle workers so they see the stop
    }                                                         // jthreads join here
    ThreadPool(const ThreadPool&) = delete;
    ThreadPool& operator=(const ThreadPool&) = delete;

    template <typename F, typename... Args>
    auto submit(F&& f, Args&&... args) -> std::future<std::invoke_result_t<F, Args...>> {
        using R = std::invoke_result_t<F, Args...>;
        std::packaged_task<R()> task{[f = std::forward<F>(f), ... a = std::forward<Args>(args)]() mutable {
            return std::invoke(std::move(f), std::move(a)...);
        }};
        auto fut = task.get_future();
        {
            std::lock_guard g{m_};
            tasks_.emplace_back(std::move(task));             // std::move_only_function would avoid the wrapper
            ++pending_;
        }
        cv_.notify_one();
        return fut;
    }

    void wait_idle() {                                        // block until every submitted task finished
        std::unique_lock lock{m_};
        idle_cv_.wait(lock, [&] { return pending_ == 0; });
    }

private:
    void run(std::stop_token st) {
        while (true) {
            std::packaged_task<void()> job;
            {
                std::unique_lock lock{m_};
                cv_.wait(lock, [&] { return st.stop_requested() || !tasks_.empty(); });
                if (tasks_.empty()) return;                   // stop requested and nothing left
                job = std::move(tasks_.front());
                tasks_.pop_front();
            }
            job();                                            // run outside the lock; exceptions go to the future
            {
                std::lock_guard g{m_};
                if (--pending_ == 0) idle_cv_.notify_all();
            }
        }
    }

    std::mutex m_;
    std::condition_variable cv_, idle_cv_;
    std::deque<std::packaged_task<void()>> tasks_;
    std::size_t pending_ = 0;
    std::vector<std::jthread> workers_;                       // declared last: destroyed (joined) first
};
```

Two details carry the design. **`std::packaged_task<R()>`** connects a callable to a `std::future<R>`: when the task runs, its return value or exception is stored in the future. Tasks of every result type are stored in one queue of `std::packaged_task<void()>`, which can hold a `packaged_task<R()>` because invoking it returns nothing. And **member declaration order** matters: `workers_` is declared after the mutex, condition variables, and queue, so it is destroyed *first* — the threads are joined while the state they use still exists.

```cpp
// file: pool/test.cpp
#include <atomic>
#include <cassert>
#include <chrono>
#include <iostream>
#include <numeric>
#include <stdexcept>
#include <string>
#include "thread_pool.hpp"

long long count_primes(int from, int to) {
    long long n = 0;
    for (int x = std::max(2, from); x < to; ++x) {
        bool prime = true;
        for (int d = 2; d * d <= x && prime; ++d) prime = x % d != 0;
        n += prime;
    }
    return n;
}

int main() {
    ThreadPool pool{4};

    auto sq = pool.submit([](int x) { return x * x; }, 12);
    auto text = pool.submit([](std::string a, std::string b) { return a + b; }, std::string("thread "), std::string("pool"));
    auto boom = pool.submit([]() -> int { throw std::runtime_error("task error"); });
    assert(sq.get() == 144);
    assert(text.get() == "thread pool");
    try { boom.get(); assert(false); } catch (const std::runtime_error& e) { std::cout << "exception propagated: " << e.what() << '\n'; }

    std::atomic<int> done{0};
    for (int i = 0; i < 1000; ++i) pool.submit([&done] { done.fetch_add(1, std::memory_order_relaxed); });
    pool.wait_idle();
    assert(done == 1000);
    std::cout << "1000 small tasks completed\n";

    const int limit = 2'000'000, chunks = 64;
    auto t0 = std::chrono::steady_clock::now();
    std::vector<std::future<long long>> parts;
    for (int c = 0; c < chunks; ++c) parts.push_back(pool.submit(count_primes, limit / chunks * c, limit / chunks * (c + 1)));
    long long total = 0;
    for (auto& f : parts) total += f.get();
    double ms = std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
    assert(total == count_primes(0, limit));
    std::cout << "primes below " << limit << ": " << total << " (" << chunks << " tasks, " << ms << " ms)\n";
    std::cout << "all thread pool tests passed\n";
}
```

```console
$ cd pool && g++-14 -std=c++20 -O1 -g -Wall -Wextra -pthread -fsanitize=thread test.cpp -o test_tsan && ./test_tsan
exception propagated: task error
1000 small tasks completed
primes below 2000000: 148933 (64 tasks, 148.111 ms)
all thread pool tests passed
$ cd pool && g++-14 -std=c++20 -O2 -Wall -Wextra -pthread test.cpp -o test && ./test | tail -n 2
primes below 2000000: 148933 (64 tasks, 138.219 ms)
all thread pool tests passed
```

The pool passed every check under ThreadSanitizer — results, exception propagation, waiting for idleness, a thousand tiny tasks, and a parallel computation verified against the serial answer — and shut down cleanly when it went out of scope.

### Review checklist

- Is every access to the queue and `pending_` under the mutex? Does any task run while the lock is held?
- Can a task submitted during destruction be lost or run after the pool's state is destroyed?
- What happens if a task submits another task and waits for it on a pool with one thread? (Deadlock — document it, or support work-stealing.)

## Answers to the Chapter 10 prediction questions

There are no prediction questions in this chapter.

# Chapter 11 — Building, Testing, and Tooling

*Goal: set up C++ projects the way professional teams do — modern CMake with targets, unit tests with GoogleTest, micro-benchmarks with Google Benchmark, static analysis with clang-tidy, formatting with clang-format, and sanitizer builds — so that quality is checked automatically on every change.*

C++ has no single official build tool or package manager, but the ecosystem has converged: **CMake** for builds, **GoogleTest** (or Catch2) for tests, **Google Benchmark** for micro-benchmarks, **clang-tidy** and **clang-format** for analysis and style, and **vcpkg** or **Conan** for dependencies. This chapter teaches each tool through a running project and ends with a complete, reusable project template.

**What you will be able to do after this chapter**

- Write modern, target-based CMake with warnings, sanitizer options, and dependencies.
- Write unit tests with GoogleTest: assertions, fixtures, parameterised tests, and death tests.
- Write micro-benchmarks with Google Benchmark that measure what you think they measure.
- Use clang-tidy and clang-format, and choose useful warning flags.
- Add sanitizer and hardened builds to a project, and explain what a CI pipeline should run.

## Topic 11A — Building

## Unit 11.1 — Modern CMake for C++

**One-line summary:** Describe **targets** (libraries and executables) and their **usage requirements** (include directories, compile features, link dependencies) with `target_*` commands and `PUBLIC`/`PRIVATE`/`INTERFACE` visibility; let CMake propagate them. Require the language standard with `cxx_std_20`, keep warnings and sanitizers in options, and get dependencies with `find_package` or `FetchContent`.

The essentials (the C book's Unit 4.6 introduced CMake; this unit applies it to C++):

```cmake
cmake_minimum_required(VERSION 3.24)
project(textkit VERSION 1.0 LANGUAGES CXX)

add_library(textkit src/split.cpp)                       # the library target
target_include_directories(textkit PUBLIC include)        # users get the include path too
target_compile_features(textkit PUBLIC cxx_std_20)        # users must compile as C++20 as well

add_library(project_warnings INTERFACE)                   # a "flags-only" target
target_compile_options(project_warnings INTERFACE -Wall -Wextra -Wpedantic -Wshadow -Wconversion)
target_link_libraries(textkit PRIVATE project_warnings)   # warnings for us, not imposed on users

option(ENABLE_SANITIZERS "Build with ASan and UBSan" OFF)
if(ENABLE_SANITIZERS)
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
  add_link_options(-fsanitize=address,undefined)
endif()
```

### Dependencies

- **`find_package(GTest)`** finds an installed package (system package manager, vcpkg, Conan) through its CMake config files and gives you imported targets like `GTest::gtest_main`.
- **`FetchContent`** downloads a dependency's source at configure time and builds it as part of your project. With `FIND_PACKAGE_ARGS` (CMake 3.24+), it first tries `find_package` and only downloads when nothing is installed — the best of both.
- **vcpkg** and **Conan** manage binary packages for many libraries across platforms; both integrate through `find_package`. (They are not demonstrated here; the project in this chapter uses the system packages installed with `apt` and falls back to downloading.)

### Presets

`CMakePresets.json` records named configurations (`debug`, `release`, `asan`) so that every developer and the CI system build the same way: `cmake --preset asan && cmake --build --preset asan && ctest --preset asan`. The chapter project includes one.

### Mastery check

1. **Explain.** Why are warning flags attached through a `PRIVATE` interface target rather than with `add_compile_options` or `PUBLIC`?

## Topic 11B — Testing

## Unit 11.2 — GoogleTest

**One-line summary:** GoogleTest organises tests as `TEST(Suite, Name) { … }` functions using `EXPECT_*` (record a failure, continue) and `ASSERT_*` (record and stop this test) macros; **fixtures** (`TEST_F`) share setup between tests; **parameterised tests** (`TEST_P`) run one test body over many inputs; **death tests** check that code aborts. `ctest` runs the test executables.

```cpp
// file: gtest_demo/stack_test.cpp
#include <gtest/gtest.h>
#include <stdexcept>
#include <vector>

class BoundedStack {
public:
    explicit BoundedStack(std::size_t cap) : cap_{cap} {}
    void push(int v) { if (v_.size() == cap_) throw std::overflow_error("full"); v_.push_back(v); }
    int pop() { if (v_.empty()) throw std::underflow_error("empty"); int x = v_.back(); v_.pop_back(); return x; }
    std::size_t size() const { return v_.size(); }
    int top_unchecked() const { return v_.back(); }       // precondition: not empty
private:
    std::size_t cap_;
    std::vector<int> v_;
};

TEST(BoundedStack, PushPopIsLifo) {
    BoundedStack s{3};
    s.push(1);
    s.push(2);
    EXPECT_EQ(s.pop(), 2);
    EXPECT_EQ(s.pop(), 1);
    EXPECT_EQ(s.size(), 0u);
}

TEST(BoundedStack, ThrowsWhenFullOrEmpty) {
    BoundedStack s{1};
    s.push(7);
    EXPECT_THROW(s.push(8), std::overflow_error);
    EXPECT_EQ(s.pop(), 7);
    EXPECT_THROW(s.pop(), std::underflow_error);
}

class FilledStack : public ::testing::Test {              // fixture: shared setup
protected:
    void SetUp() override { for (int i = 1; i <= 5; ++i) s.push(i * 10); }
    BoundedStack s{5};
};

TEST_F(FilledStack, IsFull) { EXPECT_THROW(s.push(60), std::overflow_error); }
TEST_F(FilledStack, PopsInReverse) { for (int i = 5; i >= 1; --i) ASSERT_EQ(s.pop(), i * 10) << "at step " << i; }

class CapacityTest : public ::testing::TestWithParam<std::size_t> {};
TEST_P(CapacityTest, HoldsExactlyCapacityItems) {          // parameterised over capacities
    BoundedStack s{GetParam()};
    for (std::size_t i = 0; i < GetParam(); ++i) s.push(static_cast<int>(i));
    EXPECT_EQ(s.size(), GetParam());
    EXPECT_ANY_THROW(s.push(0));
}
INSTANTIATE_TEST_SUITE_P(Sizes, CapacityTest, ::testing::Values(1, 2, 10, 1000));

TEST(BoundedStackDeathTest, TopOnEmptyAbortsWithAssertions) {
    BoundedStack s{1};
    EXPECT_DEATH((void)s.top_unchecked(), "");             // with _GLIBCXX_ASSERTIONS, back() on empty aborts
}

TEST(Demo, ShowsAFailure) {
    EXPECT_EQ(2 + 2, 5) << "a deliberately failing check, to show the report format";
}
```

```console
$ cd gtest_demo && g++-14 -std=c++20 -D_GLIBCXX_ASSERTIONS stack_test.cpp -o stack_test -lgtest -lgtest_main -pthread && ./stack_test --gtest_brief=1 2>&1 | grep -vE '^\s*$'; echo "exit status: $?"
Running main() from ./googletest/src/gtest_main.cc
stack_test.cpp:59: Failure
Expected equality of these values:
  2 + 2
    Which is: 4
  5
a deliberately failing check, to show the report format
[  FAILED  ] Demo.ShowsAFailure (0 ms)
[==========] 10 tests from 5 test suites ran. (0 ms total)
[  PASSED  ] 9 tests.
exit status: 0
```

The run reports one failure — the deliberate one — with file, line, both values, and the custom message; every other test passed, including the parameterised suite (four instances) and the death test (which forks a child process and checks that it dies). (`echo $?` prints `grep`'s status here; the test binary itself exits non-zero whenever any test fails, which is what CI checks.)

### What makes tests valuable

- **Test behaviour, not implementation:** assert on observable results, so refactoring doesn't break tests.
- **One reason to fail per test,** with a name that says what is expected.
- **Fast and deterministic:** no sleeps, no real network, no wall-clock dependence; seed random generators.
- **Run them under sanitizers in CI** — a test suite under ASan/UBSan/TSan finds bugs the assertions don't check for.

## Unit 11.3 — Google Benchmark

**One-line summary:** Google Benchmark runs a function enough times to measure it reliably, reports time per iteration with variance, supports argument ranges and complexity estimation, and provides `benchmark::DoNotOptimize` and `ClobberMemory` to stop the optimiser from deleting the work being measured.

```cpp
// file: bench_demo/bench.cpp
#include <benchmark/benchmark.h>
#include <algorithm>
#include <map>
#include <numeric>
#include <random>
#include <unordered_map>
#include <vector>

static std::vector<int> random_keys(std::size_t n) {
    std::vector<int> v(n);
    std::iota(v.begin(), v.end(), 0);
    std::shuffle(v.begin(), v.end(), std::mt19937{42});
    return v;
}

template <typename Map>
static void BM_Lookup(benchmark::State& state) {
    auto keys = random_keys(static_cast<std::size_t>(state.range(0)));
    Map m;
    for (int k : keys) m[k] = k;
    std::size_t i = 0;
    for (auto _ : state) {                                // the timed loop
        auto it = m.find(keys[i++ % keys.size()]);
        benchmark::DoNotOptimize(it);                     // the result "escapes": cannot be optimised away
    }
    state.SetComplexityN(state.range(0));
}
BENCHMARK_TEMPLATE(BM_Lookup, std::map<int, int>)->RangeMultiplier(10)->Range(1000, 1000000)->Complexity();
BENCHMARK_TEMPLATE(BM_Lookup, std::unordered_map<int, int>)->RangeMultiplier(10)->Range(1000, 1000000)->Complexity();

static void BM_SortedVectorLookup(benchmark::State& state) {
    auto keys = random_keys(static_cast<std::size_t>(state.range(0)));
    std::vector<int> sorted = keys;
    std::sort(sorted.begin(), sorted.end());
    std::size_t i = 0;
    for (auto _ : state)
        benchmark::DoNotOptimize(std::binary_search(sorted.begin(), sorted.end(), keys[i++ % keys.size()]));
}
BENCHMARK(BM_SortedVectorLookup)->RangeMultiplier(10)->Range(1000, 1000000);

BENCHMARK_MAIN();
```

```console
$ cd bench_demo && g++-14 -std=c++20 -O2 bench.cpp -o bench -lbenchmark -pthread && ./bench --benchmark_min_time=0.2s 2>/dev/null | grep -E 'BM_|Benchmark' | sed -E 's/ +[0-9]+$//'
Benchmark                                                Time             CPU   Iterations
BM_Lookup<std::map<int, int>>/1000                    36.0 ns         34.6 ns
BM_Lookup<std::map<int, int>>/10000                   84.2 ns         76.7 ns
BM_Lookup<std::map<int, int>>/100000                   222 ns          209 ns
BM_Lookup<std::map<int, int>>/1000000                  828 ns          795 ns
BM_Lookup<std::map<int, int>>_BigO                    0.00 N          0.00 N    
BM_Lookup<std::map<int, int>>_RMS                       28 %            27 %    
BM_Lookup<std::unordered_map<int, int>>/1000          6.64 ns         6.21 ns
BM_Lookup<std::unordered_map<int, int>>/10000         5.50 ns         5.21 ns
BM_Lookup<std::unordered_map<int, int>>/100000        9.90 ns         9.86 ns
BM_Lookup<std::unordered_map<int, int>>/1000000       15.8 ns         15.6 ns
BM_Lookup<std::unordered_map<int, int>>_BigO          0.65 lgN        0.64 lgN  
BM_Lookup<std::unordered_map<int, int>>_RMS             23 %            24 %    
BM_SortedVectorLookup/1000                            39.5 ns         37.4 ns
BM_SortedVectorLookup/10000                           85.0 ns         76.1 ns
BM_SortedVectorLookup/100000                           100 ns         96.6 ns
BM_SortedVectorLookup/1000000                          182 ns          182 ns
```

The table compares the three lookup structures at four sizes. From 1,000 to 1,000,000 keys, `std::map` slowed by about 20× — far more than the 2× its O(log n) suggests, because once the tree outgrows the CPU caches every level of the search is a cache miss — while the hash map stayed in the tens of nanoseconds and binary search over a sorted vector (contiguous memory) stayed several times faster than the tree. The `_BigO` and `_RMS` rows are Google Benchmark's attempt to fit a complexity curve; with only four sizes and cache effects dominating, the fit is poor (35% RMS error for the map), a reminder to read the raw numbers. Run benchmarks on an optimised build, on a quiet machine, and compare runs with the `compare.py` tool shipped with the library rather than by eye.

### Deep & Tricky

**DT-43. A benchmark whose result is unused may measure nothing.** *(Optimiser behaviour)* If the compiler can see that a computation's result is never used and has no side effects, it removes it — and the benchmark reports a beautifully fast, meaningless time (the C book's Chapter 11 benchmarks hit exactly this). `benchmark::DoNotOptimize(value)` forces the value to be materialised; `benchmark::ClobberMemory()` forces pending memory writes to happen.

## Topic 11C — Quality tools

## Unit 11.4 — Warnings, clang-tidy, and clang-format

**One-line summary:** Beyond `-Wall -Wextra`, C++ code benefits from `-Wshadow -Wconversion -Wsign-conversion -Wnon-virtual-dtor -Wold-style-cast -Woverloaded-virtual -Wnull-dereference -Wdouble-promotion -Wformat=2 -Wimplicit-fallthrough`. **clang-tidy** adds hundreds of checks (bug-prone patterns, modernisation, C++ Core Guidelines), many with automatic fixes. **clang-format** removes formatting from code review.

```cpp
// file: tidy/legacy.cpp
#include <cstdio>
#include <cstring>
#include <vector>

class Widget {
public:
    Widget(int n) : n(n) {}                              // implicit conversion from int
    virtual void draw() { std::printf("%d\n", n); }
    ~Widget() {}                                          // non-virtual destructor in a polymorphic class
    int n;
};

int sum(std::vector<int> v) {                             // copies the vector
    int total = 0;
    for (std::size_t i = 0; i < v.size(); i++) total += v[i];
    return total;
}

int main() {
    Widget* w = new Widget(3);                            // owning raw pointer
    w->draw();
    char buf[8];
    std::strcpy(buf, "hello");
    int* p = NULL;
    if (p == NULL) std::printf("%d %s\n", sum({1, 2, 3}), buf);
    delete w;
}
```

```console
$ cd tidy && clang-tidy -quiet --checks='-*,modernize-*,bugprone-*,performance-*,cppcoreguidelines-*,-cppcoreguidelines-avoid-magic-numbers,-modernize-use-trailing-return-type' legacy.cpp -- -std=c++20 2>/dev/null | grep -oE 'warning: .*\[[a-z-]+\.?[a-z-]*\]' | sed -E 's/warning: //' | sort -u | head -n 14
class 'Widget' defines a non-default destructor but does not define a copy constructor, a copy assignment operator, a move constructor or a move assignment operator [cppcoreguidelines-special-member-functions]
deleting a pointer through a type that is not marked 'gsl::owner<>'; consider using a smart pointer instead [cppcoreguidelines-owning-memory]
destructor of 'Widget' is public and non-virtual [cppcoreguidelines-virtual-class-destructor]
do not call c-style vararg functions [cppcoreguidelines-pro-type-vararg]
do not implicitly decay an array into a pointer; consider using gsl::array_view or an explicit cast instead [cppcoreguidelines-pro-bounds-array-to-pointer-decay]
initializing non-owner 'Widget *' with a newly created 'gsl::owner<>' [cppcoreguidelines-owning-memory]
use '= default' to define a trivial destructor [modernize-use-equals-default]
use auto when initializing with new to avoid duplicating the type name [modernize-use-auto]
use nullptr [modernize-use-nullptr]
use range-based for loop instead [modernize-loop-convert]
```

clang-tidy found the problems a reviewer would raise: a public non-virtual destructor in a polymorphic class, a class that declares a destructor but none of the other special members (Rule of Five), owning raw pointers from `new` and `delete`, C varargs (`printf`), an array decaying to a pointer (`strcpy` into `buf`), `NULL` instead of `nullptr`, an index loop that could be a range-`for`, and a destructor that should be `= default`. Other check groups find more — `google-explicit-constructor` would flag the implicit conversion from `int`, `performance-unnecessary-value-param` the vector copied into `sum` (it is only reported in some configurations). Many checks offer `--fix`. Adopt it gradually: enable a small set of checks, fix the code, and add checks over time; a `.clang-tidy` file in the repository records the choice.

**clang-format** reformats code to a style described in `.clang-format` (for example `BasedOnStyle: Google` with a few overrides). Run it in an editor on save and in CI (`clang-format --dry-run --Werror`), and formatting stops being a topic of review comments.

## Unit 11.5 — Sanitizers, hardening, and CI

**One-line summary:** Every C++ project should have, besides the normal debug and release builds, an **ASan+UBSan** build, a **TSan** build for concurrent code, and a build with library assertions (`-D_GLIBCXX_ASSERTIONS`, or libc++'s hardening modes) — and CI should build and test all of them, plus run clang-tidy and clang-format checks, on every change.

A minimal CI pipeline for C++:

1. Configure and build with GCC and with Clang, warnings as errors.
2. Run the unit tests in a Debug build with ASan+UBSan and `_GLIBCXX_ASSERTIONS`.
3. Run concurrency tests under TSan.
4. Run clang-tidy on changed files and clang-format in check mode.
5. Run fuzz targets for a fixed time on parsers (Chapter 12 of the C book, Chapter 13 here).
6. Build Release and run benchmarks on a dedicated machine for performance-sensitive code, comparing against the previous run.

### Pattern

**P-6. Make every quality check a command that CI runs, not a habit people must remember.** *(Practice)* Warnings as errors, sanitizer test runs, static analysis, formatting, and fuzzing all run automatically per change, with presets so developers can run the same commands locally. *Trade-off:* longer CI times and some initial clean-up, in exchange for defects caught minutes after they are written instead of months later.

## Chapter 11 Project — A professional project template: an LRU cache library

**Goal:** a header-only generic **LRU (least-recently-used) cache** — the data structure behind web caches, database buffer pools, and CPU caches — packaged as a complete CMake project with an interface library target, GoogleTest tests (including a differential test), Google Benchmark benchmarks, sanitizer and release presets, and a clang-tidy configuration. This layout can be copied for any library you write.

### Layout

```
lru/
  CMakeLists.txt  CMakePresets.json  .clang-tidy
  include/lru/lru_cache.hpp
  tests/lru_test.cpp
  bench/lru_bench.cpp
```

### The library

```cpp
// file: lru/include/lru/lru_cache.hpp
#pragma once
#include <cstddef>
#include <functional>
#include <list>
#include <optional>
#include <stdexcept>
#include <unordered_map>
#include <utility>

namespace lru {

// LruCache maps keys to values, holding at most capacity() entries. Every get or
// put marks the entry most recently used; inserting into a full cache evicts the
// least recently used entry. All operations are O(1) on average.
template <typename K, typename V, typename Hash = std::hash<K>>
class LruCache {
public:
    explicit LruCache(std::size_t capacity) : capacity_{capacity} {
        if (capacity == 0) throw std::invalid_argument("LruCache capacity must be positive");
        index_.reserve(capacity);
    }

    // get returns a pointer to the value (valid until the next modification), or nullptr.
    V* get(const K& key) {
        auto it = index_.find(key);
        if (it == index_.end()) { ++misses_; return nullptr; }
        order_.splice(order_.begin(), order_, it->second);   // move to front: O(1), no allocation
        ++hits_;
        return &it->second->second;
    }

    // put inserts or replaces; returns the evicted key, if any.
    std::optional<K> put(K key, V value) {
        if (auto it = index_.find(key); it != index_.end()) {
            it->second->second = std::move(value);
            order_.splice(order_.begin(), order_, it->second);
            return std::nullopt;
        }
        std::optional<K> evicted;
        if (order_.size() == capacity_) {
            evicted = std::move(order_.back().first);
            index_.erase(*evicted);
            order_.pop_back();
        }
        order_.emplace_front(std::move(key), std::move(value));
        index_.emplace(order_.front().first, order_.begin());
        return evicted;
    }

    bool erase(const K& key) {
        auto it = index_.find(key);
        if (it == index_.end()) return false;
        order_.erase(it->second);
        index_.erase(it);
        return true;
    }

    std::size_t size() const { return index_.size(); }
    std::size_t capacity() const { return capacity_; }
    std::size_t hits() const { return hits_; }
    std::size_t misses() const { return misses_; }

    template <typename F> void for_each_mru(F&& f) const {   // most recent first
        for (const auto& [k, v] : order_) f(k, v);
    }

private:
    using Entry = std::pair<K, V>;
    std::size_t capacity_;
    std::list<Entry> order_;                                  // front = most recently used
    std::unordered_map<K, typename std::list<Entry>::iterator, Hash> index_;
    std::size_t hits_ = 0, misses_ = 0;
};

}  // namespace lru
```

The classic design combines a doubly linked list (recency order; `splice` moves an element in O(1) without invalidating iterators — DT-19 again) with a hash map from keys to list iterators.

### Tests: unit tests plus a differential test against a simple model

```cpp
// file: lru/tests/lru_test.cpp
#include <gtest/gtest.h>
#include <algorithm>
#include <random>
#include <string>
#include <vector>
#include "lru/lru_cache.hpp"

using lru::LruCache;

TEST(LruCache, RejectsZeroCapacity) { EXPECT_THROW((LruCache<int, int>{0}), std::invalid_argument); }

TEST(LruCache, EvictsLeastRecentlyUsed) {
    LruCache<std::string, int> c{2};
    c.put("a", 1);
    c.put("b", 2);
    ASSERT_NE(c.get("a"), nullptr);          // a is now most recent
    auto evicted = c.put("c", 3);            // evicts b
    ASSERT_TRUE(evicted.has_value());
    EXPECT_EQ(*evicted, "b");
    EXPECT_EQ(c.get("b"), nullptr);
    EXPECT_EQ(*c.get("a"), 1);
    EXPECT_EQ(*c.get("c"), 3);
}

TEST(LruCache, PutExistingKeyUpdatesAndRefreshes) {
    LruCache<int, std::string> c{2};
    c.put(1, "one");
    c.put(2, "two");
    EXPECT_FALSE(c.put(1, "uno").has_value());   // update, no eviction
    c.put(3, "three");                            // evicts 2, not 1
    EXPECT_EQ(*c.get(1), "uno");
    EXPECT_EQ(c.get(2), nullptr);
}

TEST(LruCache, CountsHitsAndMisses) {
    LruCache<int, int> c{4};
    c.put(1, 1);
    c.get(1);
    c.get(2);
    c.get(1);
    EXPECT_EQ(c.hits(), 2u);
    EXPECT_EQ(c.misses(), 1u);
}

// Differential test: compare with an obviously correct (slow) model on random operations.
TEST(LruCache, MatchesReferenceModel) {
    const std::size_t cap = 8;
    LruCache<int, int> cache{cap};
    std::vector<std::pair<int, int>> model;      // front = most recent
    std::mt19937 rng{123};
    for (int step = 0; step < 100000; ++step) {
        int key = static_cast<int>(rng() % 20), op = static_cast<int>(rng() % 3);
        auto it = std::find_if(model.begin(), model.end(), [&](auto& e) { return e.first == key; });
        if (op < 2) {                            // get
            int* got = cache.get(key);
            if (it == model.end()) { ASSERT_EQ(got, nullptr) << "step " << step; }
            else {
                ASSERT_NE(got, nullptr) << "step " << step;
                ASSERT_EQ(*got, it->second);
                std::rotate(model.begin(), it, it + 1);
            }
        } else {                                 // put
            if (it != model.end()) model.erase(it);
            else if (model.size() == cap) model.pop_back();
            model.insert(model.begin(), {key, step});
            cache.put(key, step);
        }
        ASSERT_EQ(cache.size(), model.size());
    }
    std::vector<int> order;
    cache.for_each_mru([&](int k, int) { order.push_back(k); });
    for (std::size_t i = 0; i < model.size(); ++i) ASSERT_EQ(order[i], model[i].first);
}
```

### Benchmark

```cpp
// file: lru/bench/lru_bench.cpp
#include <benchmark/benchmark.h>
#include <random>
#include "lru/lru_cache.hpp"

static void BM_LruMixed(benchmark::State& state) {
    const auto cap = static_cast<std::size_t>(state.range(0));
    lru::LruCache<int, int> cache{cap};
    std::mt19937 rng{1};
    std::geometric_distribution<int> skewed{0.001};               // popular keys are small numbers
    for (auto _ : state) {
        int key = skewed(rng);
        if (int* v = cache.get(key)) benchmark::DoNotOptimize(*v);
        else cache.put(key, key);
    }
    state.counters["hit_rate"] = static_cast<double>(cache.hits()) / static_cast<double>(cache.hits() + cache.misses());
}
BENCHMARK(BM_LruMixed)->Arg(100)->Arg(1000)->Arg(10000);
BENCHMARK_MAIN();
```

The key distribution is skewed (a geometric distribution: small keys are much more popular), as real cache workloads are; a uniform distribution would make every cache size look equally bad.

### Build files

```cmake
# file: lru/CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(lru VERSION 1.0 LANGUAGES CXX)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)                        # compile_commands.json for clang-tidy and editors

add_library(lru INTERFACE)                                   # header-only library
add_library(lru::lru ALIAS lru)
target_include_directories(lru INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_features(lru INTERFACE cxx_std_20)

add_library(lru_warnings INTERFACE)
target_compile_options(lru_warnings INTERFACE -Wall -Wextra -Wpedantic -Wshadow -Wconversion
                       -Wnon-virtual-dtor -Wold-style-cast -Woverloaded-virtual)

option(LRU_SANITIZE "Build tests with ASan and UBSan" OFF)
option(LRU_BENCH "Build benchmarks" ON)

include(FetchContent)
FetchContent_Declare(googletest                              # installed copy if present, else download
  GIT_REPOSITORY https://github.com/google/googletest.git GIT_TAG v1.15.2
  FIND_PACKAGE_ARGS NAMES GTest)
FetchContent_MakeAvailable(googletest)

enable_testing()
add_executable(lru_test tests/lru_test.cpp)
target_link_libraries(lru_test PRIVATE lru::lru lru_warnings GTest::gtest_main)
if(LRU_SANITIZE)
  target_compile_options(lru_test PRIVATE -fsanitize=address,undefined -fno-omit-frame-pointer -D_GLIBCXX_ASSERTIONS)
  target_link_options(lru_test PRIVATE -fsanitize=address,undefined)
endif()
include(GoogleTest)
gtest_discover_tests(lru_test)

if(LRU_BENCH)
  find_package(benchmark REQUIRED)
  add_executable(lru_bench bench/lru_bench.cpp)
  target_link_libraries(lru_bench PRIVATE lru::lru benchmark::benchmark)
endif()
```

```json
// file: lru/CMakePresets.json
{
  "version": 6,
  "configurePresets": [
    {"name": "debug-asan", "binaryDir": "build/asan",
     "cacheVariables": {"CMAKE_BUILD_TYPE": "Debug", "LRU_SANITIZE": "ON", "LRU_BENCH": "OFF"}},
    {"name": "release", "binaryDir": "build/release",
     "cacheVariables": {"CMAKE_BUILD_TYPE": "Release"}}
  ],
  "buildPresets": [{"name": "debug-asan", "configurePreset": "debug-asan"}, {"name": "release", "configurePreset": "release"}],
  "testPresets": [{"name": "debug-asan", "configurePreset": "debug-asan", "output": {"outputOnFailure": true}}]
}
```

(The `// file:` line is for this book's tooling; JSON has no comments, so it is removed before use, as the commands below do.)

```yaml
# file: lru/.clang-tidy
Checks: >
  -*,bugprone-*,performance-*,modernize-*,cppcoreguidelines-*,
  -modernize-use-trailing-return-type,-cppcoreguidelines-avoid-magic-numbers,
  -bugprone-easily-swappable-parameters
WarningsAsErrors: ''
HeaderFilterRegex: 'include/lru/.*'
```

### Building, testing, analysing

```console
$ cd lru && sed -i '1d' CMakePresets.json && rm -rf build && cmake --preset debug-asan > /dev/null && cmake --build --preset debug-asan 2>&1 | grep -E 'warning|error|Built target' ; ctest --preset debug-asan 2>&1 | grep -E 'tests passed|Failed|Passed' | tail -n 3
[100%] Built target lru_test
4/5 Test #4: LruCache.CountsHitsAndMisses .................   Passed    0.01 sec
5/5 Test #5: LruCache.MatchesReferenceModel ...............   Passed    0.22 sec
100% tests passed, 0 tests failed out of 5
$ cd lru && cmake --preset release > /dev/null && cmake --build --preset release 2>&1 | grep -E 'warning|error|Built target'; ./build/release/lru_bench --benchmark_min_time=0.2s 2>/dev/null | grep -E 'BM_' | awk '{print $1, $2, $3, $NF}'
[ 50%] Built target lru_test
[100%] Built target lru_bench
BM_LruMixed/100 101 ns hit_rate=0.0494671
BM_LruMixed/1000 86.5 ns hit_rate=0.449903
BM_LruMixed/10000 32.6 ns hit_rate=0.998866
$ cd lru && clang-tidy -quiet -p build/asan include/lru/lru_cache.hpp tests/lru_test.cpp 2>/dev/null | grep -E 'warning:' | sed -E 's|^.*/(include\|tests)/|\1/|' | sort -u | head -n 6
include/lru/lru_cache.hpp:59:5: warning: function 'size' should be marked [[nodiscard]] [modernize-use-nodiscard]
include/lru/lru_cache.hpp:60:5: warning: function 'capacity' should be marked [[nodiscard]] [modernize-use-nodiscard]
include/lru/lru_cache.hpp:61:5: warning: function 'hits' should be marked [[nodiscard]] [modernize-use-nodiscard]
include/lru/lru_cache.hpp:62:5: warning: function 'misses' should be marked [[nodiscard]] [modernize-use-nodiscard]
include/lru/lru_cache.hpp:64:49: warning: forwarding reference parameter 'f' is never forwarded inside the function body [cppcoreguidelines-missing-std-forward]
tests/lru_test.cpp:20:16: warning: unchecked access to optional value [bugprone-unchecked-optional-access]
```

The sanitizer build compiled without warnings under a strict warning set, and every test passed under AddressSanitizer, UndefinedBehaviorSanitizer, and library assertions — including the differential test's 100,000 random operations. The benchmark shows each operation costs on the order of tens of nanoseconds and that hit rate rises with cache size. clang-tidy, configured from the repository's `.clang-tidy` and reading the compile commands CMake exported, made reasonable suggestions even on this small, carefully written library: mark the getters `[[nodiscard]]`, forward (or take by `const&`) the callable in `for_each_mru`, and — in a test — dereference an `optional` only after a check the tool can see. (A first version of this project forgot `CMAKE_EXPORT_COMPILE_COMMANDS`, and clang-tidy silently analysed nothing; always confirm a tool actually ran.) Fixing such findings, or consciously disabling a check in `.clang-tidy`, is the routine that keeps a codebase clean.

### Review checklist

- Can a user of the library build it with nothing but CMake and a compiler (FetchContent fallback), and does it impose no warning flags on them?
- Do the tests cover eviction order, updates, capacity one, and agreement with a model?
- Does CI run the `debug-asan` preset and clang-tidy on every change?

## Answers to the Chapter 11 prediction questions

There are no prediction questions in this chapter.

# Chapter 12 — Performance Engineering in C++

*Goal: make C++ code fast deliberately — avoid hidden allocations and copies, choose cache-friendly data layouts, use allocators and heterogeneous lookup, and prove each improvement with measurements.*

C++ gives you control over memory, layout, and code generation, but its abstractions can also hide costs: a `std::string` parameter that allocates, a `std::map` whose every node is a cache miss, an `std::function` that heap-allocates its target, a `std::istringstream` that locks a locale on every word. Performance work in C++ is mostly about seeing those costs and removing the ones that matter — measured, one at a time. (The C book's Chapter 11 covers the hardware side — caches, branch prediction, SIMD, profiling — and applies unchanged; this chapter focuses on what is specific to C++.)

**What you will be able to do after this chapter**

- Find allocation and copy costs in C++ code, and remove them with `reserve`, `string_view`, moves, and heterogeneous lookup.
- Use polymorphic memory resources (`std::pmr`) for arena and pool allocation.
- Choose data layouts — contiguous containers, values over pointers, struct-of-arrays — for cache efficiency.
- Explain the small-buffer optimisation in `std::string` and `std::function`.
- Use `[[likely]]`, `constexpr` tables, and devirtualisation where they help.
- Take a realistic program through a measured, step-by-step optimisation.

## Topic 12A — Hidden costs

## Unit 12.1 — Counting allocations

**One-line summary:** Heap allocation is one of the most common hidden costs in C++: each `new` costs tens to hundreds of nanoseconds and scatters data across memory. Replace the global `operator new` in a test program to **count** allocations, and you will find them in surprising places.

```cpp
// file: allocs.cpp
#include <cstdio>
#include <cstdlib>
#include <functional>
#include <map>
#include <new>
#include <string>
#include <vector>

static long g_allocs = 0;
void* operator new(std::size_t n) {                       // count every allocation in the program
    ++g_allocs;
    if (void* p = std::malloc(n)) return p;
    throw std::bad_alloc{};
}
void operator delete(void* p) noexcept { std::free(p); }
void operator delete(void* p, std::size_t) noexcept { std::free(p); }

template <typename F>
void count(const char* label, F f) {
    long before = g_allocs;
    f();
    std::printf("%-48s %4ld allocation(s)\n", label, g_allocs - before);
}

void by_value(std::string s) { (void)s; }
void by_cref(const std::string& s) { (void)s; }

int main() {
    std::string long_str(64, 'x');
    count("std::string of 15 chars (fits SSO)", [] { std::string s(15, 'a'); });
    count("std::string of 16 chars (heap)", [] { std::string s(16, 'a'); });
    count("pass long string by value", [&] { by_value(long_str); });
    count("pass long string by const&", [&] { by_cref(long_str); });
    count("call const std::string& with a literal", [] { by_cref("a literal longer than fifteen chars"); });
    count("push_back 1000 ints, no reserve", [] { std::vector<int> v; for (int i = 0; i < 1000; ++i) v.push_back(i); });
    count("push_back 1000 ints, reserve first", [] { std::vector<int> v; v.reserve(1000); for (int i = 0; i < 1000; ++i) v.push_back(i); });
    count("insert 100 ints into std::map", [] { std::map<int, int> m; for (int i = 0; i < 100; ++i) m[i] = i; });
    count("std::function with small lambda (SBO)", [] { int a = 1; std::function<int()> f = [a] { return a; }; (void)f; });
    count("std::function with large capture", [] { std::array<long, 8> big{}; std::function<long()> f = [big] { return big[0]; }; (void)f; });
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall allocs.cpp -o allocs && ./allocs
std::string of 15 chars (fits SSO)                  0 allocation(s)
std::string of 16 chars (heap)                      1 allocation(s)
pass long string by value                           1 allocation(s)
pass long string by const&                          0 allocation(s)
call const std::string& with a literal              1 allocation(s)
push_back 1000 ints, no reserve                    11 allocation(s)
push_back 1000 ints, reserve first                  1 allocation(s)
insert 100 ints into std::map                     100 allocation(s)
std::function with small lambda (SBO)               0 allocation(s)
std::function with large capture                    1 allocation(s)
```

Each line is a lesson:

- **Small-string optimisation (SSO):** libstdc++'s `std::string` stores up to 15 characters inside the object itself; the 16th character forces a heap allocation.
- **Pass-by-value of a string copies it** (an allocation) unless the argument is a temporary that can be moved; `const&` and `string_view` never allocate.
- **A `const std::string&` parameter called with a literal creates a temporary `std::string`** — an allocation for a long literal. Taking `std::string_view` avoids it.
- **`vector` growth** reallocates about log₂ n times; `reserve` makes it one.
- **Node-based containers** (`map`, `set`, `list`, `unordered_map`) allocate per element.
- **`std::function`** stores small callables inline (small-buffer optimisation, 16 bytes in libstdc++) and heap-allocates larger ones.

### Mastery check

1. **Predict.** How many allocations does `std::vector<std::string> v{"a", "b", "c"};` make? (Hint: the strings are short; count the vector's buffer and the initializer_list's temporaries.)

## Unit 12.2 — Heterogeneous lookup and avoiding temporary keys

**One-line summary:** Looking up a `std::string`-keyed map with a `string_view` or a literal normally constructs a temporary `std::string` — an allocation per lookup for long keys. **Transparent comparators and hashes** (`std::less<>`, and a hash with `is_transparent` for unordered containers, C++20) let the container compare directly with the lookup type.

```cpp
// file: hetero.cpp
#include <chrono>
#include <cstdio>
#include <cstdlib>
#include <new>
#include <string>
#include <string_view>
#include <unordered_map>
#include <vector>

static long g_allocs = 0;
void* operator new(std::size_t n) { ++g_allocs; if (void* p = std::malloc(n)) return p; throw std::bad_alloc{}; }
void operator delete(void* p) noexcept { std::free(p); }
void operator delete(void* p, std::size_t) noexcept { std::free(p); }

struct StringHash {                                        // transparent: hashes string, string_view, const char*
    using is_transparent = void;
    std::size_t operator()(std::string_view s) const noexcept { return std::hash<std::string_view>{}(s); }
};

int main() {
    std::unordered_map<std::string, int> plain;
    std::unordered_map<std::string, int, StringHash, std::equal_to<>> transparent;
    std::vector<std::string> keys;
    for (int i = 0; i < 1000; ++i) {
        keys.push_back("a fairly long key that does not fit in SSO #" + std::to_string(i));
        plain[keys.back()] = i;
        transparent[keys.back()] = i;
    }
    std::vector<std::string_view> views(keys.begin(), keys.end());   // lookups arrive as views (e.g. parsed input)

    long sum = 0;
    long a0 = g_allocs;
    auto t0 = std::chrono::steady_clock::now();
    for (int r = 0; r < 200; ++r)
        for (auto v : views) sum += plain.find(std::string(v))->second;     // must build a std::string
    auto t1 = std::chrono::steady_clock::now();
    long a1 = g_allocs;
    for (int r = 0; r < 200; ++r)
        for (auto v : views) sum += transparent.find(v)->second;           // no temporary
    auto t2 = std::chrono::steady_clock::now();
    long a2 = g_allocs;
    auto ms = [](auto a, auto b) { return std::chrono::duration<double, std::milli>(b - a).count(); };
    std::printf("plain:       %ld allocations, %.1f ms\ntransparent: %ld allocations, %.1f ms (sum %ld)\n",
                a1 - a0, ms(t0, t1), a2 - a1, ms(t1, t2), sum);
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall hetero.cpp -o hetero && ./hetero
plain:       200000 allocations, 8.7 ms
transparent: 0 allocations, 4.8 ms (sum 199800000)
```

200,000 lookups with the plain map made 200,000 allocations; the transparent map made none and ran faster. For ordered containers, `std::map<std::string, V, std::less<>>` enables the same (since C++14).

<!-- expanded -->
### Going deeper: heterogeneous lookup in `unordered_map`

`std::map<std::string, V, std::less<>>` accepts a `std::string_view` or `const char*` key in `find` without building a `std::string`. For `std::unordered_map` the same needs C++20 and *two* transparent pieces: a hash with `using is_transparent = void;` and a transparent equality (`std::equal_to<>`). Without them every lookup with a string literal allocates a temporary `std::string` once the literal is longer than the small-string buffer.

```cpp
// file: hetero_um.cpp
#include <cstdio>
#include <cstdlib>
#include <new>
#include <string>
#include <string_view>
#include <unordered_map>

static int allocations = 0;
void* operator new(std::size_t n) { ++allocations; if (void* p = std::malloc(n)) return p; throw std::bad_alloc(); }
void operator delete(void* p) noexcept { std::free(p); }
void operator delete(void* p, std::size_t) noexcept { std::free(p); }

struct StringHash {
    using is_transparent = void;
    std::size_t operator()(std::string_view s) const { return std::hash<std::string_view>{}(s); }
};

int main() {
    const char* key = "a-rather-long-configuration-key";
    std::unordered_map<std::string, int> plain{{key, 1}};
    std::unordered_map<std::string, int, StringHash, std::equal_to<>> transparent{{key, 1}};

    int before = allocations;
    for (int i = 0; i < 1000; ++i) (void)plain.find(key);
    std::printf("plain unordered_map:       %d allocations for 1000 lookups\n", allocations - before);
    before = allocations;
    for (int i = 0; i < 1000; ++i) (void)transparent.find(key);
    std::printf("transparent unordered_map: %d allocations for 1000 lookups\n", allocations - before);
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra -Wno-mismatched-new-delete hetero_um.cpp -o hetero_um && ./hetero_um
plain unordered_map:       1000 allocations for 1000 lookups
transparent unordered_map: 0 allocations for 1000 lookups
```

## Unit 12.3 — Allocators and `std::pmr`

**One-line summary:** **Polymorphic memory resources** (`<memory_resource>`, C++17) let containers allocate from a resource you choose at run time: `std::pmr::monotonic_buffer_resource` hands out memory from a buffer and frees it all at once (an arena — the C book's Chapter 3 project), and pool resources recycle fixed-size blocks. `std::pmr::vector`, `std::pmr::string`, and `std::pmr::unordered_map` use them.

```cpp
// file: pmr.cpp
#include <array>
#include <chrono>
#include <cstddef>
#include <iostream>
#include <list>
#include <memory_resource>
#include <string>

template <typename F>
double ms(F f) {
    auto t0 = std::chrono::steady_clock::now();
    f();
    return std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
}

int main() {
    const int n = 1'000'000;
    long checksum = 0;
    double t_default = ms([&] {
        for (int r = 0; r < 5; ++r) {
            std::list<int> l;
            for (int i = 0; i < n; ++i) l.push_back(i);
            checksum += l.back();
        }
    });
    double t_arena = ms([&] {
        for (int r = 0; r < 5; ++r) {
            std::pmr::monotonic_buffer_resource arena{std::pmr::new_delete_resource()};   // grows in big chunks
            std::pmr::list<int> l{&arena};
            for (int i = 0; i < n; ++i) l.push_back(i);
            checksum += l.back();
        }                                          // the whole arena released at once
    });
    std::cout << "1M-node lists x5: default allocator " << t_default << " ms, monotonic arena " << t_arena
              << " ms (checksum " << checksum << ")\n";

    std::array<std::byte, 1024> stack_buf;         // allocation from a stack buffer, no heap at all
    std::pmr::monotonic_buffer_resource local{stack_buf.data(), stack_buf.size(), std::pmr::null_memory_resource()};
    std::pmr::string s{"pmr strings propagate their allocator", &local};
    std::cout << s << " (allocated inside a 1 KiB stack buffer)\n";
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall pmr.cpp -o pmr && ./pmr
1M-node lists x5: default allocator 139.558 ms, monotonic arena 42.0175 ms (checksum 9999990)
pmr strings propagate their allocator (allocated inside a 1 KiB stack buffer)
```

Arena allocation is a pointer bump, and freeing is one operation for everything — ideal for per-request or per-frame data with a common lifetime. The `null_memory_resource` upstream makes the stack-buffer example throw `bad_alloc` if the buffer is exhausted, instead of silently falling back to the heap.

### Deep & Tricky

**DT-44. Objects allocated from a `monotonic_buffer_resource` must not outlive it.** *(Library behaviour)* Destroying the resource releases all its memory, whether or not the containers using it are still alive; and `deallocate` is a no-op, so memory freed by containers is not reused until the resource dies. Use monotonic resources for bounded lifetimes; use `unsynchronized_pool_resource` when objects come and go.

## Topic 12B — Layout and code generation

## Unit 12.4 — Values over pointers, and data-oriented design

**One-line summary:** A `std::vector<T>` of values is one contiguous block; a `std::vector<std::unique_ptr<T>>` (or of `shared_ptr`) is an array of pointers to objects scattered over the heap. Store values when you can, group the fields a hot loop uses together (struct of arrays), and keep rarely used data out of hot structs.

```cpp
// file: layout.cpp
#include <chrono>
#include <iostream>
#include <memory>
#include <random>
#include <string>
#include <vector>

struct Particle {                       // hot fields and cold fields together
    float x, y, vx, vy;
    std::string name;                   // 32 bytes of rarely used data
    double created_at;
};

struct ParticlesSoA {                   // struct of arrays: one array per field
    std::vector<float> x, y, vx, vy;
};

template <typename F>
double ms(F f) {
    auto t0 = std::chrono::steady_clock::now();
    for (int r = 0; r < 20; ++r) f();
    return std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
}

int main() {
    const int n = 1'000'000;
    std::vector<Particle> values(n);
    std::vector<std::unique_ptr<Particle>> pointers;
    for (int i = 0; i < n; ++i) pointers.push_back(std::make_unique<Particle>());
    std::shuffle(pointers.begin(), pointers.end(), std::mt19937{1});   // realistic: allocation order != iteration order
    ParticlesSoA soa{std::vector<float>(n), std::vector<float>(n), std::vector<float>(n, 1), std::vector<float>(n, 1)};

    double t_ptr = ms([&] { for (auto& p : pointers) { p->x += p->vx; p->y += p->vy; } });
    double t_val = ms([&] { for (auto& p : values) { p.x += p.vx; p.y += p.vy; } });
    double t_soa = ms([&] { for (int i = 0; i < n; ++i) { soa.x[i] += soa.vx[i]; soa.y[i] += soa.vy[i]; } });
    std::cout << "update 1M particles x20: vector<unique_ptr> " << t_ptr << " ms, vector<Particle> " << t_val
              << " ms, struct of arrays " << t_soa << " ms\n";
    std::cout << "sizeof(Particle) = " << sizeof(Particle) << " bytes, of which the loop uses 16\n";
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall layout.cpp -o layout && ./layout
update 1M particles x20: vector<unique_ptr> 235.748 ms, vector<Particle> 64.2811 ms, struct of arrays 14.859 ms
sizeof(Particle) = 56 bytes, of which the loop uses 16
```

Same arithmetic, three layouts, very different speeds: pointer-chasing through shuffled heap objects is slowest; contiguous values are faster but still drag 56 unused bytes per particle through the cache; the struct-of-arrays layout streams exactly the needed data and lets the compiler vectorise the loop. This is the core idea of data-oriented design, used throughout game engines and high-performance systems.

<!-- expanded -->
### Going deeper: false sharing

Two threads writing *different* variables can still slow each other down badly when the variables sit in the same 64-byte cache line: every write takes the line away from the other core. This is **false sharing**. Padding each thread's data to its own line — `alignas(std::hardware_destructive_interference_size)`, or simply `alignas(64)` — removes it.

```cpp
// file: falseshare.cpp
#include <atomic>
#include <chrono>
#include <cstdio>
#include <thread>
#include <vector>

struct Packed { std::atomic<long> v{0}; };
struct alignas(64) Padded { std::atomic<long> v{0}; };

template <typename Slot> double run() {
    Slot slots[4];
    auto t0 = std::chrono::steady_clock::now();
    {
        std::vector<std::jthread> ts;
        for (int t = 0; t < 4; ++t)
            ts.emplace_back([&slots, t] { for (int i = 0; i < 20'000'000; ++i) slots[t].v.fetch_add(1, std::memory_order_relaxed); });
    }
    return std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
}

int main() {
    double packed = run<Packed>(), padded = run<Padded>();
    std::printf("sizeof Packed %zu, Padded %zu\n", sizeof(Packed), sizeof(Padded));
    std::printf("same cache line is %s than separate lines\n", packed > 1.5 * padded ? "clearly slower" : "not clearly slower");
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra falseshare.cpp -o falseshare && nproc | awk '{print ($1 >= 4) ? "4+ cores available" : "fewer than 4 cores: the effect may not show"}' && ./falseshare
fewer than 4 cores: the effect may not show
sizeof Packed 8, Padded 64
same cache line is clearly slower than separate lines
```

The result depends on the machine having several cores running the threads at the same time; even on the small virtual machine used for this book, which reports fewer than four cores, the packed version was clearly slower. `std::hardware_destructive_interference_size` (in `<new>`) gives the line size the compiler targets, but GCC warns that using it in a header can change the layout between compiler flags, so many codebases write 64 explicitly.

## Unit 12.5 — Helping the compiler: inlining, devirtualisation, `[[likely]]`

**One-line summary:** The optimiser's best tool is **inlining**, which exposes more code to every other optimisation; templates and lambdas inline naturally, virtual calls and `std::function` usually do not. `final` classes and LTO let the compiler **devirtualise**; `[[likely]]`/`[[unlikely]]` (C++20) guide code layout; `constexpr` moves work to compile time.

```cpp
// file: inline.cpp
#include <algorithm>
#include <chrono>
#include <functional>
#include <iostream>
#include <random>
#include <vector>

template <typename F>
double ms(F f) {
    auto t0 = std::chrono::steady_clock::now();
    f();
    return std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
}

bool less_fn(int a, int b) { return a < b; }

int main() {
    std::vector<int> base(5'000'000);
    std::mt19937 rng{3};
    for (auto& x : base) x = static_cast<int>(rng());
    auto a = base, b = base, c = base, d = base;
    double t_lambda = ms([&] { std::sort(a.begin(), a.end(), [](int x, int y) { return x < y; }); });
    double t_fnptr = ms([&] { std::sort(b.begin(), b.end(), less_fn); });                     // through a pointer
    std::function<bool(int, int)> f = [](int x, int y) { return x < y; };
    double t_stdfn = ms([&] { std::sort(c.begin(), c.end(), f); });                           // type-erased
    double t_qsort = ms([&] { std::qsort(d.data(), d.size(), sizeof(int), [](const void* p, const void* q) {
                                   int x = *static_cast<const int*>(p), y = *static_cast<const int*>(q);
                                   return (x > y) - (x < y); }); });
    std::cout << "sort 5M ints: lambda " << t_lambda << " ms, function pointer " << t_fnptr << " ms, std::function "
              << t_stdfn << " ms, C qsort " << t_qsort << " ms; equal: " << (a == b && b == c && c == d) << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall inline.cpp -o inline && ./inline
sort 5M ints: lambda 315.193 ms, function pointer 441.247 ms, std::function 565.828 ms, C qsort 621.371 ms; equal: 1
```

The comparator passed as a lambda is part of the `std::sort` instantiation's type, so it is inlined into the sorting loop. A function pointer is only a value of type `bool(*)(int, int)`, so the instantiation calls through it — here about 1.7× slower. Through `std::function` every comparison is an indirect call plus the wrapper's dispatch, and C's `qsort` pays for an indirect call and generic byte-wise element moves. This is the "zero-overhead abstraction" promise made concrete: the most abstract version is the fastest.

### Pattern

**P-7. Optimise in this order: algorithm, allocations, layout, then code generation — measuring each step.** *(Practice)* A better algorithm or data structure beats everything else; then remove unnecessary allocations and copies; then make memory access contiguous and compact; only then tune inlining, branches, and vectorisation. Keep each change only if a benchmark shows it matters. *Trade-off:* time spent measuring in exchange for changes that are real and code that stays readable where speed doesn't matter.

<!-- expanded -->
### Deep & Tricky

**DT-45. `[[likely]]` and `[[unlikely]]` only change code layout, never correctness, and a wrong hint makes code slower. Profile-guided optimisation (`-fprofile-generate`, run, `-fprofile-use`) usually beats hand-written hints, because it uses measured branch frequencies for every branch, not just the ones you annotated.** *(Compiler behaviour: the attributes are hints the standard lets implementations ignore)* Use the attributes for branches whose frequency you *know* (error paths), and PGO for the rest.

```console
$ printf '#include <cstdio>\nint main(int c,char**){long s=0;for(long i=0;i<200000000;++i){if(i%%97==c)s+=i;else s^=i;}std::printf("%%ld\\n",s);}\n' > pgo.cpp && g++-14 -O2 -fprofile-generate pgo.cpp -o pgo && ./pgo > /dev/null && ls *.gcda | sed 's/$/ (the recorded profile)/' && g++-14 -O2 -fprofile-use pgo.cpp -o pgo && ./pgo
pgo.gcda (the recorded profile)
206185689703408
```

## Chapter 12 Project — Making a word-frequency counter 10× faster

**Goal:** a program that reads a large text and prints the 10 most frequent words is written the "obvious" way — `std::istringstream`, `std::map<std::string, int>`, lowercasing with a copy — then optimised step by step, measuring each version, until it is at least ten times faster. Every version must produce identical output.

### The versions

```cpp
// file: wordfreq/wordfreq.cpp
#include <algorithm>
#include <array>
#include <cctype>
#include <chrono>
#include <cstdint>
#include <cstdio>
#include <fstream>
#include <functional>
#include <iostream>
#include <map>
#include <memory_resource>
#include <sstream>
#include <string>
#include <string_view>
#include <unordered_map>
#include <vector>

using Counts = std::vector<std::pair<std::string, long>>;

Counts top10(std::vector<std::pair<std::string, long>> all) {
    std::partial_sort(all.begin(), all.begin() + std::min<std::size_t>(10, all.size()), all.end(),
                      [](const auto& a, const auto& b) { return a.second != b.second ? a.second > b.second : a.first < b.first; });
    all.resize(std::min<std::size_t>(10, all.size()));
    return all;
}

// v1: the obvious version
Counts v1_naive(const std::string& text) {
    std::map<std::string, long> counts;
    std::istringstream in{text};
    std::string word;
    while (in >> word) {
        std::string clean;
        for (char c : word)
            if (std::isalpha(static_cast<unsigned char>(c))) clean += static_cast<char>(std::tolower(static_cast<unsigned char>(c)));
        if (!clean.empty()) ++counts[clean];
    }
    return top10({counts.begin(), counts.end()});
}

// v2: hash map instead of a tree
Counts v2_unordered(const std::string& text) {
    std::unordered_map<std::string, long> counts;
    std::istringstream in{text};
    std::string word;
    while (in >> word) {
        std::string clean;
        for (char c : word)
            if (std::isalpha(static_cast<unsigned char>(c))) clean += static_cast<char>(std::tolower(static_cast<unsigned char>(c)));
        if (!clean.empty()) ++counts[clean];
    }
    return top10({counts.begin(), counts.end()});
}

// v3: scan the text directly (no istringstream), reuse one buffer for the word
Counts v3_scan(const std::string& text) {
    std::unordered_map<std::string, long> counts;
    std::string word;
    for (std::size_t i = 0, n = text.size(); i <= n; ++i) {
        unsigned char c = i < n ? static_cast<unsigned char>(text[i]) : ' ';
        if (std::isalpha(c)) word += static_cast<char>(std::tolower(c));
        else if (std::isspace(c) || i == n) { if (!word.empty()) { ++counts[word]; word.clear(); } }
    }
    return top10({counts.begin(), counts.end()});
}

// v4: transparent lookup with string_view: allocate only for NEW words; reserve the table
struct SvHash { using is_transparent = void; std::size_t operator()(std::string_view s) const noexcept { return std::hash<std::string_view>{}(s); } };
Counts v4_transparent(const std::string& text) {
    std::unordered_map<std::string, long, SvHash, std::equal_to<>> counts;
    counts.reserve(1 << 16);
    std::string word;
    for (std::size_t i = 0, n = text.size(); i <= n; ++i) {
        unsigned char c = i < n ? static_cast<unsigned char>(text[i]) : ' ';
        if (std::isalpha(c)) word += static_cast<char>(std::tolower(c));
        else if (std::isspace(c) || i == n) {
            if (!word.empty()) {
                if (auto it = counts.find(std::string_view{word}); it != counts.end()) ++it->second;
                else counts.emplace(word, 1);
                word.clear();
            }
        }
    }
    return top10({counts.begin(), counts.end()});
}

// v5: one lowercase copy of all letters, an open-addressing table of string_views into it, a lookup table
Counts v5_flat(const std::string& text) {
    static constexpr auto kind = [] {                        // compile-time classification: 0 other, 1 space, else letter
        std::array<unsigned char, 256> t{};
        for (int c = 'a'; c <= 'z'; ++c) t[static_cast<std::size_t>(c)] = static_cast<unsigned char>(c);
        for (int c = 'A'; c <= 'Z'; ++c) t[static_cast<std::size_t>(c)] = static_cast<unsigned char>(c - 'A' + 'a');
        for (char c : {' ', '\t', '\n', '\v', '\f', '\r'}) t[static_cast<unsigned char>(c)] = 1;
        return t;
    }();
    std::string letters(text.size(), '\0');                  // one allocation holds every word's characters
    struct Slot { std::string_view key; long count; std::uint64_t hash; };
    std::vector<Slot> slots(1 << 17);                        // power of two, kept sparse
    const std::size_t mask = slots.size() - 1;
    std::size_t w = 0, start = 0;                            // write cursor and start of the current word
    std::uint64_t h = 14695981039346656037ull;               // FNV-1a hash, computed while scanning
    auto flush = [&] {
        if (w == start) return;
        std::string_view word{letters.data() + start, w - start};
        for (std::size_t i = h & mask;; i = (i + 1) & mask) {
            if (slots[i].key.empty()) { slots[i] = {word, 1, h}; break; }
            if (slots[i].hash == h && slots[i].key == word) { ++slots[i].count; w = start; break; }   // reuse the space
        }
        start = w;
        h = 14695981039346656037ull;
    };
    for (unsigned char c : text) {
        unsigned char k = kind[c];
        if (k > 1) { letters[w++] = static_cast<char>(k); h = (h ^ k) * 1099511628211ull; }   // letter: append, hash
        else if (k == 1) flush();                            // whitespace ends a word; other characters are dropped
    }
    flush();
    std::vector<std::pair<std::string, long>> all;
    for (const auto& s : slots) if (!s.key.empty()) all.emplace_back(std::string{s.key}, s.count);
    return top10(std::move(all));
}

template <typename F>
Counts run(const char* name, F f, const std::string& text, double base_ms, double& out_ms) {
    auto t0 = std::chrono::steady_clock::now();
    Counts c = f(text);
    out_ms = std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
    std::printf("%-34s %8.1f ms  %5.1fx\n", name, out_ms, base_ms > 0 ? base_ms / out_ms : 1.0);
    return c;
}

int main(int argc, char* argv[]) {
    std::ifstream in{argc > 1 ? argv[1] : "corpus.txt"};
    std::string text{std::istreambuf_iterator<char>{in}, {}};
    std::printf("input: %.1f MB\n", text.size() / 1e6);
    double base = 0, t = 0;
    Counts ref = run("v1 istringstream + map", v1_naive, text, 0, base);
    bool same = true;
    same &= run("v2 unordered_map", v2_unordered, text, base, t) == ref;
    same &= run("v3 direct scan, reused buffer", v3_scan, text, base, t) == ref;
    same &= run("v4 transparent lookup + reserve", v4_transparent, text, base, t) == ref;
    same &= run("v5 flat table, LUT, inline hash", v5_flat, text, base, t) == ref;
    std::printf("all versions agree: %s\ntop words:", same ? "yes" : "NO");
    for (const auto& [w, n] : ref) std::printf(" %s(%ld)", w.c_str(), n);
    std::printf("\n");
}
```

A generator produces a reproducible 50 MB corpus with a natural-language-like (Zipf) word distribution, mixed case, and punctuation:

```cpp
// file: wordfreq/gen.cpp
#include <cmath>
#include <cstdio>
#include <random>
#include <string>
#include <vector>

int main() {
    std::mt19937 rng{2026};
    std::vector<std::string> vocab;
    std::uniform_int_distribution<int> len{2, 9}, letter{0, 25};
    for (int i = 0; i < 50000; ++i) {
        std::string w;
        for (int k = len(rng); k > 0; --k) w += static_cast<char>('a' + letter(rng));
        vocab.push_back(w);
    }
    std::vector<double> weights(vocab.size());
    for (std::size_t i = 0; i < weights.size(); ++i) weights[i] = 1.0 / static_cast<double>(i + 1);   // Zipf
    std::discrete_distribution<std::size_t> pick{weights.begin(), weights.end()};
    std::uniform_int_distribution<int> style{0, 19};
    std::size_t bytes = 0;
    while (bytes < 50'000'000) {
        std::string w = vocab[pick(rng)];
        int s = style(rng);
        if (s == 0) w[0] = static_cast<char>(w[0] - 32);             // Capitalised
        if (s == 1) w += ",";
        if (s == 2) w = "\"" + w + "\"";
        if (s == 3) w.insert(w.size() / 2, "'");                      // inner apostrophe
        w += (style(rng) == 0 ? "\n" : " ");
        bytes += w.size();
        std::fputs(w.c_str(), stdout);
    }
}
```

```console
$ cd wordfreq && g++-14 -std=c++20 -O2 gen.cpp -o gen && ./gen > corpus.txt && g++-14 -std=c++20 -O2 -Wall -Wextra wordfreq.cpp -o wordfreq && ./wordfreq corpus.txt
input: 50.0 MB
v1 istringstream + map               1633.5 ms    1.0x
v2 unordered_map                      705.5 ms    2.3x
v3 direct scan, reused buffer         548.4 ms    3.0x
v4 transparent lookup + reserve       629.4 ms    2.6x
v5 flat table, LUT, inline hash       157.0 ms   10.4x
all versions agree: yes
top words: tky(645727) zcsmazqfu(321918) hobuthqkr(214262) oxwb(161180) lvoejish(129030) caaddgbqy(107780) wzw(92446) sbo(80604) arzwr(71357) tnnmagt(64564)
$ cd wordfreq && g++-14 -std=c++20 -O1 -g -fsanitize=address,undefined wordfreq.cpp -o wordfreq_san && head -c 2000000 corpus.txt > small.txt && ./wordfreq_san small.txt | tail -n 2
all versions agree: yes
top words: tky(25792) zcsmazqfu(12933) hobuthqkr(8637) oxwb(6488) lvoejish(5091) caaddgbqy(4326) wzw(3639) sbo(3111) arzwr(2806) tnnmagt(2622)
```

What each step removed:

- **v1 → v2 (`std::map` → `std::unordered_map`):** O(log n) string comparisons per word, each a likely cache miss in a 50,000-node tree, became one hash and one or two probes.
- **v2 → v3 (no `istringstream`):** stream extraction is slow — sentry objects, locale-aware `isspace` through a facet, and a fresh `std::string` per token. Scanning the buffer directly and reusing one word buffer removes all of it.
- **v3 → v4 (transparent lookup, `reserve`): no gain — v4 was *slower* than v3.** This step was expected to help and did not, and it is kept in the program as a lesson. The word is already in a reused `std::string`, so `counts[word]` never built a temporary key; transparent lookup had nothing to remove. And `reserve(1 << 16)` gave the table fewer buckets (67,307) than it reached by growing on its own (85,229), so chains got longer. Measured in isolation over several runs, the four combinations differed by less than the run-to-run noise, with the reserved table slightly behind. Heterogeneous lookup pays when lookups arrive as `string_view`s or literals (Unit 12.2), not here.
- **v3 → v5 (flat table of views, lookup table, inline hash):** words live in one buffer; the hash table stores views and cached hashes in one contiguous array (open addressing, Chapter 7 of the C book); character classification is one table lookup instead of two `<cctype>` calls; and the FNV-1a hash is computed while the letters are copied, so each word is read once instead of being re-read by `std::hash`. No allocation happens per word at all. The first version of v5 used `std::hash<std::string_view>` after the word ended and reached only 9.5–9.9× in repeated runs, just short of the target; moving the hash into the scanning loop is what crossed 10×.

All five versions produce the same top-ten list, and the sanitizer build ran cleanly. The final version meets the 10× target (timings vary by about 10% between runs on this machine), and it is still ordinary, readable C++. The biggest wins came from data-structure choice, from not going through streams, and from removing per-word allocations; one plausible-looking step (v4) gained nothing, which is exactly why P-7 says to keep a change only if a measurement supports it.

### Review checklist

- Does every version produce identical results on the same input, including mixed case, punctuation inside words, and the last word without a trailing space?
- Which step gave the largest gain, and can you explain it in terms of allocations, cache misses, or instructions per byte?
- Is the fastest version's table guaranteed not to fill up? (With 50,000 distinct words in 131,072 slots, yes; a production version would grow it.)

## Answers to the Chapter 12 prediction questions

**12.1 Q1.** One for the vector's buffer (three elements). The three short strings fit in SSO, so the temporary `std::string`s in the `initializer_list` and the copies in the vector allocate nothing.

# Chapter 13 — Undefined Behaviour and Safe C++

*Goal: recognise the ways C++ code goes wrong silently — dangling references and views, invalidated iterators, data races, out-of-bounds access — and build the defences professional teams use: lifetime-aware design, compiler warnings, hardened standard libraries, sanitizers, and fuzzing.*

C++ inherits all of C's undefined behaviour (the C book's Chapter 12) and adds its own: references and views that outlive what they refer to, iterators invalidated by container operations, temporaries destroyed sooner than expected. None of these reliably crash; they produce programs that work in testing and fail in production. Modern C++ cannot make them impossible, but the combination of value semantics, RAII, warnings, hardened libraries, sanitizers, and fuzzing makes them rare and quickly found.

**What you will be able to do after this chapter**

- Recognise dangling references, dangling `string_view`/`span`, and the temporary-lifetime rules that cause them.
- Know which container operations invalidate which iterators, references, and pointers.
- Turn on the compiler warnings and library hardening modes that catch these bugs.
- Use AddressSanitizer, UndefinedBehaviorSanitizer, and ThreadSanitizer on C++ code.
- Fuzz a C++ parser with libFuzzer and fix what it finds.
- Apply the C++ Core Guidelines' main lifetime and safety rules.

## Topic 13A — Lifetimes

## Unit 13.1 — Dangling references and views

**One-line summary:** A reference, pointer, iterator, `std::string_view`, or `std::span` does not own what it refers to; if the object dies first, using the reference is undefined behaviour. The common sources are returning a reference to a local or a temporary, binding a reference to a function result that refers to a temporary argument, and storing a view of a temporary string.

```cpp
// file: dangle.cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <string_view>

std::string make_name() { return "a name long enough to be on the heap"; }

int main() {
    const int& m = std::max(1, 2);               // max returns a reference to one of its (temporary) arguments
    std::cout << "max: " << m << '\n';           // the temporaries died at the end of the previous line

    std::string_view v = make_name();            // view of a temporary std::string, destroyed at the ';'
    std::cout << "view: " << v << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall -Wextra dangle.cpp -o dangle
dangle.cpp: In function ‘int main()’:
dangle.cpp:10:16: warning: possibly dangling reference to a temporary [-Wdangling-reference]
   10 |     const int& m = std::max(1, 2);               // max returns a reference to one of its (temporary) arguments
      |                ^
dangle.cpp:10:28: note: the temporary was destroyed at the end of the full expression ‘std::max<int>(1, 2)’
   10 |     const int& m = std::max(1, 2);               // max returns a reference to one of its (temporary) arguments
      |                    ~~~~~~~~^~~~~~
dangle.cpp:11:34: warning: ‘<anonymous>’ is used uninitialized [-Wuninitialized]
   11 |     std::cout << "max: " << m << '\n';           // the temporaries died at the end of the previous line
      |                                  ^~~~
dangle.cpp:10:28: note: ‘<anonymous>’ was declared here
   10 |     const int& m = std::max(1, 2);               // max returns a reference to one of its (temporary) arguments
      |                    ~~~~~~~~^~~~~~
$ clang++ -std=c++20 -O2 -Wall dangle.cpp -o dangle_clang
dangle.cpp:13:26: warning: object backing the pointer will be destroyed at the end of the
      full-expression [-Wdangling-gsl]
   13 |     std::string_view v = make_name();            // view of a temporary std::string, destroy...
      |                          ^~~~~~~~~~~
1 warning generated.
$ clang++ -std=c++20 -O1 -g -w -fsanitize=address dangle.cpp -o dangle_asan && ./dangle_asan 2>&1 | grep -E 'ERROR|READ of|#[0-9] .*main' | head -n 3
==26098==ERROR: AddressSanitizer: heap-use-after-free on address 0x504000000050 at pc 0x56101cc8acec bp 0x7fffdae78880 sp 0x7fffdae78048
READ of size 36 at 0x504000000050 thread T0
    #3 0x56101cd47e5d in main dangle.cpp:14:27
```

GCC 14's `-Wdangling-reference` (enabled by `-Wextra`) flags the `std::max` binding, and its optimiser then notices the read of a dead object (`-Wuninitialized`); Clang misses that one but flags the `string_view` of a temporary through its `-Wdangling-gsl` analysis (on by default), which GCC misses. AddressSanitizer catches the `string_view` read at run time (`heap-use-after-free`: the string's buffer had been freed); the dead `int` was not reported, because at `-O1` the compiler had already folded the value `2` into the output statement. Neither compiler catches everything — warnings are heuristics, not proofs — but both are free, and a warning-clean build is the first line of defence.

The rules that prevent dangling:

- **Return by value** unless you are returning a reference to something that clearly outlives the call (a member of `*this`, an element of a container argument).
- **Do not store views** (`string_view`, `span`, iterators, raw pointers) in long-lived objects unless the owner's lifetime is guaranteed; prefer owning types for members.
- **Lifetime extension applies only to a reference bound directly to a temporary** (`const std::string& s = make_name();` is fine), not through a function that returns a reference.

### Deep & Tricky

**DT-46. `std::string_view` parameters are safe; `std::string_view` return values and members are where dangling happens.** *(Core Guidelines F.42, Library semantics)* A view parameter cannot outlive the call, so the caller's argument is alive while the function runs. A function returning a view must refer to something that outlives the call — never to a local or to a by-value parameter — and a class holding a view member must document whose data it views. Return `std::string` when in doubt.

## Unit 13.2 — Iterator invalidation

**One-line summary:** Operations that change a container's structure can **invalidate** iterators, pointers, and references into it: `vector::push_back` invalidates everything if it reallocates; `vector::erase` invalidates everything at and after the erased point; `unordered_map` insertion invalidates iterators on rehash (but not references); `map`/`list` insertion invalidates nothing. Using an invalidated iterator is UB.

```cpp
// file: invalidate.cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3};
    int& first = v[0];                      // reference into the vector's buffer
    for (int i = 0; i < 10; ++i) v.push_back(i);   // reallocation moves the elements
    std::cout << "first = " << first << '\n';      // use-after-free
}
```

```console
$ clang++ -std=c++20 -O1 -g -fsanitize=address invalidate.cpp -o invalidate && ./invalidate 2>&1 | grep -E 'ERROR|READ of|#0 .*main|freed by' | head -n 3
==26106==ERROR: AddressSanitizer: heap-use-after-free on address 0x502000000010 at pc 0x557ef59cbf56 bp 0x7ffd174a23b0 sp 0x7ffd174a23a8
READ of size 4 at 0x502000000010 thread T0
    #0 0x557ef59cbf55 in main invalidate.cpp:9:32
```

The classic loop bug — erasing while iterating — has a standard fix in C++20, `std::erase_if`, and a manual form that uses `erase`'s return value:

```cpp
// file: erase.cpp
#include <iostream>
#include <map>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5, 6};
    std::erase_if(v, [](int x) { return x % 2 == 0; });          // C++20: one call, linear time
    std::map<int, char> m{{1, 'a'}, {2, 'b'}, {3, 'c'}, {4, 'd'}};
    for (auto it = m.begin(); it != m.end();)                      // manual: erase returns the next iterator
        it = (it->first % 2 == 0) ? m.erase(it) : std::next(it);
    for (int x : v) std::cout << x << ' ';
    for (auto& [k, c] : m) std::cout << k << c << ' ';
    std::cout << '\n';
}
```

```console
$ g++-14 -std=c++20 -O2 -Wall erase.cpp -o erase && ./erase
1 3 5 1a 3c 
```

The invalidation rules (summary):

| Container | Insert | Erase |
|---|---|---|
| `vector`, `string` | all, if capacity grows; else at and after the point | at and after the point |
| `deque` | all iterators (references survive at the ends) | all, unless at an end |
| `list`, `map`, `set` | none | only the erased element |
| `unordered_*` | iterators on rehash; references never | only the erased element |

### Mastery check

1. **Predict.** `std::unordered_map<int, std::string> m{{1, "one"}}; std::string& r = m[1];` — then 10,000 more keys are inserted, forcing several rehashes. Is `r` still valid? Is an iterator `it = m.find(1)` taken before the inserts still valid?

### Deep & Tricky

**DT-47. The range-for over a temporary's member dangles (before C++23).** *(Standard, [stmt.ranged]; fixed by P2718 in C++23)* `for (int x : make_holder().items())` binds the range to the reference that `items()` returns, but the `Holder` temporary is destroyed before the loop body runs, so the loop iterates over a destroyed vector. C++23 extends the lifetime of all temporaries in the range expression, but GCC 14 and Clang 18 do not implement that yet. Bind the owner to a named variable first.

```cpp
// file: rangefor.cpp
#include <iostream>
#include <vector>

struct Holder {
    std::vector<int> v{1, 2, 3};
    const std::vector<int>& items() const { return v; }
};
Holder make_holder() { return {}; }

int main() {
    int sum = 0;
    for (int x : make_holder().items()) sum += x;      // Holder destroyed before the first iteration
    std::cout << sum << '\n';
}
```

```console
$ clang++ -std=c++23 -O1 -g -fsanitize=address rangefor.cpp -o rangefor && ./rangefor 2>&1 | grep -E 'ERROR|READ of' | head -n 2
==26120==ERROR: AddressSanitizer: heap-use-after-free on address 0x502000000010 at pc 0x564e4698ee07 bp 0x7ffee307d130 sp 0x7ffee307d128
READ of size 4 at 0x502000000010 thread T0
```

## Topic 13B — Defences

## Unit 13.3 — Hardened standard library modes

**One-line summary:** `std::vector::operator[]` does not check bounds — by design. But libstdc++ has a **hardening mode**, `-D_GLIBCXX_ASSERTIONS`, that adds cheap precondition checks (bounds in `operator[]`, non-empty in `front()`/`back()`/`pop_back()`, valid `optional` access) and aborts with a message; and a **debug mode**, `-D_GLIBCXX_DEBUG`, that also tracks iterator validity (much slower, and changes the ABI, so everything must be built with it). libc++ has the equivalent `_LIBCPP_HARDENING_MODE`.

```cpp
// file: harden.cpp
#include <iostream>
#include <vector>

int main(int argc, char*[]) {
    std::vector<int> v{1, 2, 3};
    std::cout << "reading v[" << argc + 2 << "]" << std::endl;
    std::cout << v[static_cast<std::size_t>(argc + 2)] << std::endl;        // out of bounds
    for (auto it = v.begin(); it != v.end(); ++it)
        if (*it == 2) v.push_back(4);                                         // invalidates it
    std::cout << "done" << std::endl;
}
```

```console
$ g++-14 -std=c++20 -O2 harden.cpp -o harden_plain && ./harden_plain | head -n 2
reading v[3]
0
$ g++-14 -std=c++20 -O2 -D_GLIBCXX_ASSERTIONS harden.cpp -o harden_assert && ./harden_assert 2>&1 | head -n 3
reading v[3]
/usr/include/c++/14/bits/stl_vector.h:1130: constexpr std::vector<_Tp, _Alloc>::reference std::vector<_Tp, _Alloc>::operator[](size_type) [with _Tp = int; _Alloc = std::allocator<int>; reference = int&; size_type = long unsigned int]: Assertion '__n < this->size()' failed.
$ g++-14 -std=c++20 -O2 -D_GLIBCXX_DEBUG harden.cpp -o harden_debug && ./harden_debug 2>&1 | grep -A1 '^Error'
Error: attempt to subscript container with out-of-bounds index 3, but 
container only holds 3 elements.
```

Without hardening, the out-of-bounds read silently returned whatever was in memory (here 0). With `_GLIBCXX_ASSERTIONS` the program stops at the bad access with the failed precondition; distributions such as Fedora build their packages with it, because the cost is typically a few percent. Debug mode explains the error in words, and would also have reported the invalidated iterator in the loop if execution had got that far.

The full set of compiler hardening flags recommended for production C++ builds (from the OpenSSF compiler-options guide) extends the C list in the C book's Chapter 12: `-D_GLIBCXX_ASSERTIONS -D_FORTIFY_SOURCE=3 -fstack-protector-strong -fstack-clash-protection -fcf-protection -ftrivial-auto-var-init=zero -Wl,-z,relro,-z,now -fPIE -pie`.

<!-- expanded -->
### Going deeper: what `_GLIBCXX_ASSERTIONS` catches, and what it costs

`-D_GLIBCXX_ASSERTIONS` turns on cheap precondition checks in libstdc++ — `operator[]` bounds, `front()` on an empty container, `pop_back()` on an empty vector — without changing any type's layout, so it can be used in release builds and mixed with libraries built without it (Fedora and Ubuntu build many packages this way). `-D_GLIBCXX_DEBUG` adds expensive checks (iterator validity, sortedness) and changes layouts, so everything must be built with it.

```cpp
// file: assertions.cpp
#include <chrono>
#include <cstdio>
#include <vector>

int main(int argc, char**) {
    std::vector<int> v(10'000'000, 1);
    long sum = 0;
    auto t0 = std::chrono::steady_clock::now();
    for (int rep = 0; rep < 20; ++rep)
        for (std::size_t i = 0; i < v.size(); ++i) sum += v[i];
    double ms = std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - t0).count();
    std::printf("sum %ld (%s)\n", sum, ms < 1000 ? "under a second" : "slow");
    if (argc > 1) std::printf("v[v.size()] = %d\n", v[v.size()]);    // one past the end
}
```

```console
$ g++-14 -std=c++20 -O2 assertions.cpp -o plain && g++-14 -std=c++20 -O2 -D_GLIBCXX_ASSERTIONS assertions.cpp -o checked && ./plain x | head -2 && ./checked x 2>&1 | grep -oE "Assertion '[^']*' failed"
sum 200000000 (under a second)
v[v.size()] = 0
Assertion '__n < this->size()' failed
```

The unchecked build read past the end and printed whatever was there; the checked one stopped with the failed precondition. In this tight loop the checked build was no slower on the machine used for this book — about 80 ms for both, because the compiler proves most checks redundant and removes them — measure your own hot paths, but the default for production C++ should be *on*.

## Unit 13.4 — Sanitizers for C++

**One-line summary:** The sanitizers from the C book work identically for C++, with C++-aware extras: ASan detects container overflows (`-D_GLIBCXX_SANITIZE_VECTOR` annotates `vector` capacity), `new`/`delete` mismatches, and use-after-scope; UBSan checks the dynamic type in virtual calls and downcasts (`-fsanitize=vptr`); ThreadSanitizer understands `std::thread`, `std::mutex`, and atomics.

```cpp
// file: race.cpp
#include <iostream>
#include <thread>
#include <vector>

struct Stats { long hits = 0; };

int main() {
    Stats s;
    std::vector<std::jthread> ts;
    for (int t = 0; t < 4; ++t)
        ts.emplace_back([&s] { for (int i = 0; i < 100000; ++i) ++s.hits; });   // unsynchronised
    ts.clear();                                                               // join
    std::cout << "hits = " << s.hits << " (expected 400000)\n";
}
```

```console
$ g++-14 -std=c++20 -O2 race.cpp -o race && ./race
hits = 400000 (expected 400000)
$ g++-14 -std=c++20 -O1 -g -fsanitize=thread race.cpp -o race_tsan && ./race_tsan 2>&1 | grep -E 'WARNING|Write of|Previous write|#0' | head -n 4
WARNING: ThreadSanitizer: data race (pid=26179)
    #0 operator() race.cpp:12 (race_tsan+0x2500) (BuildId: 666b1682d3111928e5287026f3ef02c5f99c3e45)
  Previous write of size 8 at 0x7ffe1b1a8f50 by thread T1:
    #0 operator() race.cpp:12 (race_tsan+0x250f) (BuildId: 666b1682d3111928e5287026f3ef02c5f99c3e45)
```

The unsynchronised program printed the *correct* total. That does not mean it is correct: at `-O2` GCC turned each thread's loop into a single addition, so the window for the threads to collide was tiny. The race is still undefined behaviour, a different compiler or a busier machine can lose updates, and ThreadSanitizer reports it on every run regardless of what the output happens to be. A test that checks only output cannot prove the absence of a race.

```cpp
// file: badcast.cpp
#include <iostream>

struct Shape { virtual ~Shape() = default; virtual double area() const = 0; };
struct Circle : Shape { double r = 1; double area() const override { return 3.14159 * r * r; } };
struct Square : Shape { double s = 2; double area() const override { return s * s; } long extra = 42; };

int main() {
    Circle c;
    Shape* p = &c;
    auto* sq = static_cast<Square*>(p);            // wrong: p points to a Circle
    std::cout << sq->extra << '\n';
}
```

```console
$ clang++ -std=c++20 -O1 -g -fsanitize=undefined,vptr -fno-sanitize-recover=all badcast.cpp -o badcast && ./badcast 2>&1 | head -n 2
badcast.cpp:11:16: runtime error: downcast of address 0x7ffcd2d6a588 with insufficient space for an object of type 'Square'
0x7ffcd2d6a588: note: pointer points here
```

`static_cast` down a hierarchy is unchecked. UBSan stops at the cast itself: here the first check to fail was that a 16-byte `Circle` is too small to be a `Square`; when sizes happen to fit, the `vptr` check compares the object's real dynamic type instead. In code, use `dynamic_cast` (or better, a virtual function) when the type is not certain.

### Pattern

**P-8. Run every test suite in three build configurations: warnings-as-errors release, ASan+UBSan, and TSan for concurrent code.** *(Practice; Google, Chromium, LLVM CI)* The sanitizers cannot be combined in one binary (TSan excludes ASan), and each finds bugs the others miss. CMake presets (Chapter 11) make each configuration one command. *Trade-off:* CI time (sanitized tests run 2–15× slower) in exchange for catching lifetime, bounds, and race bugs before they ship.

## Unit 13.5 — The Core Guidelines and safety profiles

**One-line summary:** The **C++ Core Guidelines** (Stroustrup and Sutter, isocpp.github.io/CppCoreGuidelines) are the community's consensus rules for safe, modern C++. clang-tidy implements many as `cppcoreguidelines-*` checks. The most valuable lifetime and safety rules:

- **R.1 / R.11:** manage resources with RAII; avoid naked `new` and `delete`.
- **F.43 / F.42:** never return a pointer or reference to a local; return views only of things that outlive the call.
- **ES.46 / ES.49:** avoid narrowing conversions; if you must cast, use a named cast.
- **ES.65:** don't dereference an invalid pointer — the rule behind all of this chapter.
- **Bounds (Bounds.1–4):** no pointer arithmetic, use `span` and checked access instead of raw arrays.
- **CP.2:** avoid data races; **CP.20:** use RAII locks, never plain `lock()`/`unlock()`.
- **C.21:** the rule of zero/five; **C.35:** a base class destructor is public and virtual, or protected and non-virtual.

```console
$ clang-tidy --checks='-*,cppcoreguidelines-pro-bounds-pointer-arithmetic,cppcoreguidelines-owning-memory,cppcoreguidelines-narrowing-conversions,cppcoreguidelines-pro-type-static-cast-downcast' badcast.cpp invalidate.cpp -- -std=c++20 2>/dev/null | grep -E 'warning' || true
badcast.cpp:11:16: warning: do not use static_cast to downcast from a base to a derived class; use dynamic_cast instead [cppcoreguidelines-pro-type-static-cast-downcast]
```

On these two files, clang-tidy flags the unchecked downcast in `badcast.cpp`; `invalidate.cpp` has nothing these checks can see — the bug there is a lifetime problem that only a run-time tool caught. Static analysis and dynamic analysis find different bugs, which is why P-8 and these checks are used together.

Work on **safety profiles** — compiler-enforced subsets of C++ that reject lifetime and bounds errors — is in progress in the standards committee, alongside **contracts** (C++26, P2900), which give preconditions and postconditions a standard syntax and checked/unchecked build modes. Until those are widely available, the guidelines plus tools are how teams get most of the benefit today.

<!-- expanded -->
### Going deeper: `gsl::not_null` and `gsl::narrow`, in plain C++

The Guidelines Support Library (GSL) provides small types the Core Guidelines rely on. Two are worth knowing even without the library, because they are easy to write: `not_null<T*>`, a pointer that cannot hold `nullptr` (checked once, at construction, so callees need not check), and `narrow<T>(x)`, a conversion that throws if the value changes.

```cpp
// file: gsl_lite.cpp
#include <iostream>
#include <stdexcept>
#include <type_traits>

template <typename P> class not_null {
public:
    not_null(P p) : p_(p) { if (!p_) throw std::invalid_argument("not_null constructed from null"); }
    not_null(std::nullptr_t) = delete;              // a literal nullptr is a compile error
    auto operator->() const { return p_; }
    auto& operator*() const { return *p_; }
private:
    P p_;
};

template <typename To, typename From> To narrow(From x) {
    To y = static_cast<To>(x);
    if (static_cast<From>(y) != x || (std::is_signed_v<From> != std::is_signed_v<To> && ((x < From{}) != (y < To{}))))
        throw std::range_error("narrow: value changed");
    return y;
}

struct Engine { int rpm = 900; };
int read_rpm(not_null<Engine*> e) { return e->rpm; }         // no null check needed here

int main() {
    Engine e;
    std::cout << read_rpm(&e) << '\n';
    Engine* missing = nullptr;
    try { read_rpm(missing); } catch (const std::exception& ex) { std::cout << ex.what() << '\n'; }
    std::cout << narrow<unsigned char>(200) << " fits\n";
    try { narrow<unsigned char>(300); } catch (const std::exception& ex) { std::cout << ex.what() << " (300)\n"; }
    try { narrow<unsigned>(-1); } catch (const std::exception& ex) { std::cout << ex.what() << " (-1 to unsigned)\n"; }
    // read_rpm(nullptr);                            // compile error
}
```

```console
$ g++-14 -std=c++20 -Wall -Wextra gsl_lite.cpp -o gsl_lite && ./gsl_lite | cat -v
900
not_null constructed from null
M-H fits
narrow: value changed (300)
narrow: value changed (-1 to unsigned)
```

`narrow<unsigned char>(200)` prints as the character with code 200 (shown by `cat -v` as `M-H`); the point is that it did not throw. The real GSL (Microsoft's `gsl` or `gsl-lite`) adds `span` (now `std::span`), `owner<T*>` (a marker for static analysis), and `Expects`/`Ensures` contract macros.

## Topic 13C — Fuzzing

## Unit 13.6 — Fuzzing C++ with libFuzzer

**One-line summary:** A fuzzer feeds generated inputs to the code under test and keeps the inputs that reach new code (C book, Chapter 12). For C++ the entry point is the same `extern "C" int LLVMFuzzerTestOneInput(const uint8_t*, size_t)`; inside it, wrap the bytes in a `std::string_view` and call your API. Besides crashes, a fuzz target can check **properties** — invariants that must hold for every input, such as "parsing what you serialised gives back the same value" — and call `abort()` (or `__builtin_trap()`) when one fails.

The project below applies both.

<!-- expanded -->
### Going deeper: a fuzzer finding a real bug in seconds

The fuzzing harness is one function, `LLVMFuzzerTestOneInput(const uint8_t* data, size_t size)`, which libFuzzer calls millions of times with mutated inputs, guided by which code each input reaches. Combined with AddressSanitizer, it turns "this parser might mishandle some input" into a concrete crashing input within seconds.

```cpp
// file: fuzzdemo/parse.cpp
#include <cstddef>
#include <cstdint>
#include <cstring>

// Parses "LEN:BYTES" where LEN is one decimal digit. Bug: trusts LEN.
int parse_record(const uint8_t* data, size_t size, char* out /* 8 bytes */) {
    if (size < 2 || data[1] != ':' || data[0] < '0' || data[0] > '9') return -1;
    size_t len = data[0] - '0';
    if (len > 8) return -1;                       // checks the output buffer ...
    std::memcpy(out, data + 2, len);              // ... but not that the input has len bytes
    return static_cast<int>(len);
}

volatile char sink;                               // keeps the result alive, so the optimiser cannot drop the call

extern "C" int LLVMFuzzerTestOneInput(const uint8_t* data, size_t size) {
    char out[8];
    int n = parse_record(data, size, out);
    if (n > 0) sink = out[n - 1];
    return 0;
}
```

```console
$ cd fuzzdemo && clang++ -std=c++20 -g -O1 -fsanitize=fuzzer,address parse.cpp -o fuzz_parse && rm -f crash-*; ./fuzz_parse -max_total_time=60 -seed=1 > fuzz.log 2>&1; grep -m1 -oE "ERROR: AddressSanitizer: [a-z-]+" fuzz.log; ls crash-* | wc -l | sed 's/$/ crashing input saved/'; od -c crash-* | head -1
ERROR: AddressSanitizer: heap-buffer-overflow
1 crashing input saved
0000000   2   :
```

The fuzzer found an input like `2:` — a length larger than the bytes that follow (here, none) — and saved it to a `crash-<hash>` file, which becomes a regression test: `./fuzz_parse crash-<hash>` reproduces it, and after the fix (`if (len > 8 || size - 2 < len) return -1;`) it belongs in the unit tests, as this chapter's project does with `test_regressions.cpp`.

## Chapter 13 Project — Fuzzing a URL query-string library

**Goal:** a small library parses URL query strings (`name=Ada%20L&lang=c%2B%2B&x`) into key/value pairs, percent-decoding as it goes, and serialises pairs back. It ships with unit tests that pass. Two libFuzzer targets then check two things on arbitrary input — no memory errors (ASan + UBSan), and the round-trip property `parse(serialize(parse(s))) == parse(s)` — and we fix whatever it finds.

### The library

```cpp
// file: query/query.hpp
#pragma once
#include <string>
#include <string_view>
#include <utility>
#include <vector>

namespace query {
using Pairs = std::vector<std::pair<std::string, std::string>>;
Pairs parse(std::string_view s);          // "a=1&b=two%20words" -> {{"a","1"},{"b","two words"}}
std::string serialize(const Pairs& ps);   // inverse of parse
}
```

```cpp
// file: query/query.cpp
#include "query.hpp"
#include <cctype>

namespace query {
namespace {
int hexval(char c) {
    if (c >= '0' && c <= '9') return c - '0';
    if (c >= 'a' && c <= 'f') return c - 'a' + 10;
    if (c >= 'A' && c <= 'F') return c - 'A' + 10;
    return -1;
}

std::string decode(std::string_view s) {
    std::string out;
    for (std::size_t i = 0; i < s.size(); ++i) {
        if (s[i] == '+') {
            out += ' ';
        } else if (s[i] == '%') {
            int hi = hexval(s[i + 1]), lo = hexval(s[i + 2]);       // BUG? reads past the end for "%" or "%4"
            if (hi >= 0 && lo >= 0) { out += static_cast<char>(hi * 16 + lo); i += 2; }
            else out += '%';
        } else {
            out += s[i];
        }
    }
    return out;
}

std::string encode(std::string_view s) {
    static constexpr char hex[] = "0123456789ABCDEF";
    std::string out;
    for (unsigned char c : s) {
        if (std::isalnum(c) || c == '-' || c == '_' || c == '.' || c == '~') out += static_cast<char>(c);
        else if (c == ' ') out += '+';
        else { out += '%'; out += hex[c >> 4]; out += hex[c & 15]; }
    }
    return out;
}
}  // namespace

Pairs parse(std::string_view s) {
    Pairs out;
    while (!s.empty()) {
        std::size_t amp = s.find('&');
        std::string_view item = s.substr(0, amp);
        s = amp == std::string_view::npos ? std::string_view{} : s.substr(amp + 1);
        if (item.empty()) continue;
        std::size_t eq = item.find('=');
        if (eq == std::string_view::npos) out.emplace_back(decode(item), "");
        else out.emplace_back(decode(item.substr(0, eq)), decode(item.substr(eq + 1)));
    }
    return out;
}

std::string serialize(const Pairs& ps) {
    std::string out;
    for (const auto& [k, v] : ps) {
        if (!out.empty()) out += '&';
        out += encode(k);
        if (!v.empty()) { out += '='; out += encode(v); }
    }
    return out;
}
}  // namespace query
```

```cpp
// file: query/test_query.cpp
#include <cassert>
#include <iostream>
#include "query.hpp"

int main() {
    using query::Pairs;
    assert((query::parse("a=1&b=two%20words") == Pairs{{"a", "1"}, {"b", "two words"}}));
    assert((query::parse("name=Ada+L&lang=c%2B%2B&x") == Pairs{{"name", "Ada L"}, {"lang", "c++"}, {"x", ""}}));
    assert((query::parse("&&a=&") == Pairs{{"a", ""}}));
    assert((query::parse("bad=%zz") == Pairs{{"bad", "%zz"}}));
    assert(query::serialize({{"q", "x & y"}, {"n", "1"}}) == "q=x+%26+y&n=1");
    Pairs p{{"k", "v=w"}, {"empty", ""}};
    assert(query::parse(query::serialize(p)) == p);
    std::cout << "all unit tests passed\n";
}
```

Two fuzz targets: the first only exercises `parse` (any memory error or UB is reported by the sanitizers), the second adds the round-trip property:

```cpp
// file: query/fuzz_parse.cpp
#include <cstdint>
#include <string_view>
#include "query.hpp"

extern "C" int LLVMFuzzerTestOneInput(const std::uint8_t* data, std::size_t size) {
    std::string_view input{reinterpret_cast<const char*>(data), size};   // exact length, not NUL-terminated
    (void)query::parse(input);
    return 0;
}
```

```cpp
// file: query/fuzz_roundtrip.cpp
#include <cstdint>
#include <cstdio>
#include <cstdlib>
#include <string_view>
#include "query.hpp"

extern "C" int LLVMFuzzerTestOneInput(const std::uint8_t* data, std::size_t size) {
    std::string_view input{reinterpret_cast<const char*>(data), size};
    query::Pairs once = query::parse(input);
    query::Pairs twice = query::parse(query::serialize(once));
    if (once != twice) {                                                   // the round-trip property
        std::fprintf(stderr, "round-trip mismatch: %zu pairs became %zu\n", once.size(), twice.size());
        std::abort();
    }
    return 0;
}
```

### Running it

The unit tests pass, with and without sanitizers:

```console
$ cd query && g++-14 -std=c++20 -O2 -Wall -Wextra query.cpp test_query.cpp -o test_query && ./test_query
all unit tests passed
$ cd query && clang++ -std=c++20 -O1 -g -fsanitize=address,undefined query.cpp test_query.cpp -o test_query_san && ./test_query_san
all unit tests passed
```

Now fuzz it. The output goes to a log first, then is filtered — a fuzzer piped directly into `grep -m1` or `head` can be killed by `SIGPIPE` before it writes its crash file, which is exactly what happened while the C book's fuzzing examples were being written:

```console
$ cd query && rm -rf corpus found* && mkdir -p corpus && printf 'a=1&b=two%%20words' > corpus/seed && clang++ -std=c++20 -O1 -g -fsanitize=fuzzer,address,undefined query.cpp fuzz_parse.cpp -o fuzz_parse && ./fuzz_parse corpus -seed=1 -max_total_time=60 -artifact_prefix=found1- > fuzz1.log 2>&1; grep -E 'ERROR|READ of|#[0-9] .*query.cpp|round-trip|Test unit written' fuzz1.log | sed 's/found1-[a-z]*-[0-9a-f]*/found1-<hash>/' | head -n 5
==26239==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x502000002260 at pc 0x5631708f690a bp 0x7ffd876edd40 sp 0x7ffd876edd38
READ of size 1 at 0x502000002260 thread T0
    #0 0x5631708f6909 in query::(anonymous namespace)::decode[abi:cxx11](std::basic_string_view<char, std::char_traits<char>>) query/query.cpp:20:29
    #1 0x5631708f5a5f in query::parse[abi:cxx11](std::basic_string_view<char, std::char_traits<char>>) query/query.cpp:50:60
artifact_prefix='found1-'; Test unit written to found1-<hash>
```

Within seconds, AddressSanitizer stopped the fuzzer with a `heap-buffer-overflow` in `decode`, at the line that reads `s[i + 1]` and `s[i + 2]`. libFuzzer saved the input that triggered it; it is short:

```console
$ cd query && od -c found1-crash-* | head -n 2
0000000   a   a   =   1   &   b   w   o   %   8   w 244   b   w   o   %
0000020
```

The input is mostly junk, but it ends in a lone `%`, so `s[i + 1]` is one past the end — exactly the read at `query.cpp:20` that ASan reported. The unit tests never caught this, for a subtle reason: they pass string literals, and every `std::string_view` made from a literal is followed in memory by the literal's NUL terminator, so `s[i + 1]` read a `'\0'` that happened to be there. `std::string_view` makes no such promise, and the fuzzer's buffer has nothing after its last byte. The fix checks the bounds, and the failing input becomes a unit test:

```diff
-            int hi = hexval(s[i + 1]), lo = hexval(s[i + 2]);       // BUG? reads past the end for "%" or "%4"
+            int hi = i + 2 < s.size() ? hexval(s[i + 1]) : -1;      // fixed: never read past the end
+            int lo = i + 2 < s.size() ? hexval(s[i + 2]) : -1;
```

```console
$ cd query && sed -i 's|            int hi = hexval(s\[i + 1\]), lo = hexval(s\[i + 2\]);.*|            int hi = i + 2 < s.size() ? hexval(s[i + 1]) : -1;      // fixed: never read past the end\n            int lo = i + 2 < s.size() ? hexval(s[i + 2]) : -1;|' query.cpp && grep -n 'i + 2 <' query.cpp
20:            int hi = i + 2 < s.size() ? hexval(s[i + 1]) : -1;      // fixed: never read past the end
21:            int lo = i + 2 < s.size() ? hexval(s[i + 2]) : -1;
```

With the bounds bug fixed, `fuzz_parse` runs cleanly, so we move to the second target. The round-trip property failed almost immediately:

```console
$ cd query && clang++ -std=c++20 -O1 -g -fsanitize=fuzzer,address,undefined query.cpp fuzz_roundtrip.cpp -o fuzz_roundtrip && ./fuzz_roundtrip corpus -seed=1 -max_total_time=60 -artifact_prefix=found2- > fuzz2.log 2>&1; grep -E 'ERROR|round-trip|deadly signal' fuzz2.log | head -n 3; od -c found2-crash-* | head -n 2
round-trip mismatch: 3 pairs became 2
==26255== ERROR: libFuzzer: deadly signal
SUMMARY: libFuzzer: deadly signal
0000000   =   &   B   B   =   t   w   =   b   t   =   0   &   =   w   d
0000020 017
```

The input begins `=&`: the pair `("", "")`, followed by other pairs. `serialize` wrote the first pair as nothing at all — empty key, and no `=` because the value is empty — and then decided whether to write `&` by checking `out.empty()`, which was still true. So the first two pairs serialised as one, and three pairs came back as two. Two bugs, both in `serialize`: an empty key with an empty value must still produce `=`, and the separator must depend on the pair's position, not on what has been written so far:

```diff
-        if (!out.empty()) out += '&';
+        if (&p != ps.data()) out += '&';                     // fixed: separator by position
         out += encode(k);
-        if (!v.empty()) { out += '='; out += encode(v); }
+        if (!v.empty() || k.empty()) { out += '='; out += encode(v); }   // fixed: ("", "") -> "="
```

```console
$ cd query && sed -i -e 's|    for (const auto& \[k, v\] : ps) {|    for (const auto\& p : ps) {\n        const auto\& [k, v] = p;|' -e 's|        if (!out.empty()) out += .&.;|        if (\&p != ps.data()) out += \x27\&\x27;                     // fixed: separator by position|' -e 's|        if (!v.empty()) { out += .=.; out += encode(v); }|        if (!v.empty() \|\| k.empty()) { out += \x27=\x27; out += encode(v); }   // fixed: ("", "") -> "="|' query.cpp && sed -n '/^std::string serialize/,/^}/p' query.cpp
std::string serialize(const Pairs& ps) {
    std::string out;
    for (const auto& p : ps) {
        const auto& [k, v] = p;
        if (&p != ps.data()) out += '&';                     // fixed: separator by position
        out += encode(k);
        if (!v.empty() || k.empty()) { out += '='; out += encode(v); }   // fixed: ("", "") -> "="
    }
    return out;
}
```

Both crashing inputs become unit tests, so these bugs can never come back unnoticed:

```cpp
// file: query/test_regressions.cpp
#include <cassert>
#include <iostream>
#include <string>
#include "query.hpp"

int main() {
    using query::Pairs;
    std::string trailing = "a=%2";                                  // found1: a truncated escape at the very end
    assert((query::parse(std::string_view{trailing.data(), trailing.size()}) == Pairs{{"a", "%2"}}));
    assert((query::parse("x=%") == Pairs{{"x", "%"}}));
    Pairs p{{"", ""}, {"u", ""}};                                    // found2: empty key and value, then another pair
    assert(query::parse(query::serialize(p)) == p);
    std::cout << "regression tests passed\n";
}
```

```console
$ cd query && clang++ -std=c++20 -O1 -g -fsanitize=address,undefined query.cpp test_query.cpp -o test_query_san && ./test_query_san && clang++ -std=c++20 -O1 -g -fsanitize=address,undefined query.cpp test_regressions.cpp -o test_reg && ./test_reg
all unit tests passed
regression tests passed
$ cd query && clang++ -std=c++20 -O1 -g -fsanitize=fuzzer,address,undefined query.cpp fuzz_parse.cpp -o fuzz_parse && ./fuzz_parse corpus -seed=1 -max_total_time=60 -artifact_prefix=found3- > fuzz3a.log 2>&1; clang++ -std=c++20 -O1 -g -fsanitize=fuzzer,address,undefined query.cpp fuzz_roundtrip.cpp -o fuzz_roundtrip && ./fuzz_roundtrip corpus -seed=1 -max_total_time=60 -artifact_prefix=found3- > fuzz3.log 2>&1; grep -hE 'ERROR|round-trip|Done' fuzz3a.log fuzz3.log | head -n 3; ls found3-* 2>/dev/null | wc -l
Done 2926099 runs in 61 second(s)
Done 996896 runs in 61 second(s)
0
```

After the two fixes, a minute of each target — over three million generated inputs, starting from the corpus the earlier runs had built up — produced no memory error and no round-trip failure.

What this project shows:

- **Unit tests passed while two bugs lived in 40 lines of code.** Both were in edge cases nobody thought to write down: a truncated escape at the very end of the input, and an empty key with an empty value.
- **Tests with string literals hid a memory bug** because literals are NUL-terminated. Fuzz targets receive exact-length buffers, which is what real network input looks like.
- **Property checks find logic bugs, not just crashes.** The round-trip assertion turned "serialisation loses data" into a crash the fuzzer could find and minimise.
- **Every fuzzer finding becomes a regression test**, and the fuzz targets themselves are kept and run in CI (OSS-Fuzz runs targets like this one continuously for thousands of open-source projects).

### Review checklist

- Does `parse` treat every possible byte sequence safely, including empty input, a lone `%`, `%` at the end, and `=` with no key?
- Does `serialize` produce output that `parse` maps back to the identical pairs, for every possible list of pairs — including empty strings and strings containing `&`, `=`, `%`, `+`, and bytes ≥ 0x80?
- Do the regression tests construct inputs that are *not* NUL-terminated, the way the fuzzer did?

## Answers to the Chapter 13 prediction questions

**13.2 Q1.** `r` is still valid: unordered containers never move their elements, so rehashing invalidates iterators but not references or pointers to elements. The iterator `it` is invalidated by any insertion that causes a rehash and must not be used afterwards — look the key up again.

# Chapter 14 — Capstone: An Order-Matching Engine

*Goal: design, build, test, and measure a real piece of performance-critical C++ — the matching engine at the heart of every stock exchange — using everything in this book.*

Like the C book's capstone, this chapter introduces no new language features. It combines them. The system is **ob**, a **limit order book** with a **matching engine**: traders send orders to buy or sell at a price; the engine matches buyers with sellers by strict price-time priority and reports trades; unmatched orders rest in the book until they are matched or cancelled. Exchanges, crypto venues, and trading firms run engines like this, where correctness is non-negotiable and latency is measured in nanoseconds.

The reference implementation below was built and tested while writing this book. It is small enough to read in full (about 450 lines including tests and tools); your own capstone should go further — the last unit lists directions.

**Capstone deliverables**

1. A design note: requirements, data structures, and their trade-offs (Unit 14.1).
2. The library: order book and matching engine (Units 14.2–14.3).
3. A test suite: unit tests, a **differential test** against an obviously correct reference model, and sanitizer runs (Unit 14.4).
4. A replay tool and a latency report with percentiles, comparing two allocation strategies (Units 14.5–14.6).
5. A design review (Unit 14.7).

## Topic 14A — Design

## Unit 14.1 — Requirements and design

### The domain in one paragraph

A **limit order** says "buy (or sell) *quantity* at *price* or better". The **book** holds resting orders on two sides: **bids** (buy orders, best = highest price) and **asks** (sell orders, best = lowest price). When a new order arrives whose price **crosses** the best opposite price (a buy at or above the best ask, or a sell at or below the best bid), it **trades** against resting orders: best price first, and at the same price, oldest first (**price-time priority**). Trades execute at the **resting** order's price. Whatever quantity remains rests in the book. An **immediate-or-cancel (IOC)** order trades what it can and never rests. A **cancel** removes a resting order.

### Requirements

- **R1 Correctness:** strict price-time priority; trades at the maker's price; no crossed book after any operation (best bid < best ask).
- **R2 Operations:** add limit, add IOC, cancel by id; query best bid/ask, the quantity at a price level, and the number of resting orders.
- **R3 Validation:** reject non-positive quantities and duplicate ids of resting orders without changing the book.
- **R4 Performance:** add and cancel in O(log L) for L price levels (O(1) at the best level), cancel by id in O(1) lookup; report p50/p99/p99.9 latency over a million-operation replay.
- **R5 Determinism:** the same input produces the same trades, byte for byte — so results can be compared across versions and machines.

### Key decisions

| Decision | Choice | Alternative and why not (yet) |
|---|---|---|
| Prices | integers in ticks (`int64_t`) | `double`: rounding makes equal prices unequal (C book, Unit 1.4) |
| Price levels | `std::map<Price, Level>`, bids with `std::greater<>` | flat array indexed by price: faster, but needs a bounded price range (Unit 14.7) |
| Orders at a level | `std::list` (FIFO) | `vector`/`deque`: cancel from the middle would be O(n) or need tombstones |
| Cancel by id | `unordered_map<OrderId, Locator>` holding a list iterator | search the book: O(n) |
| Allocation | everything through a `std::pmr::memory_resource*` | fixed `std::allocator`: cannot try a pool without rewriting (Unit 14.6) |
| Errors | return codes (`AddResult`), no exceptions on the hot path | exceptions: invalid orders are expected input, not exceptional |

`std::list` iterators stay valid when other elements are inserted or erased (Unit 13.2's table), which is exactly what the id index needs: it can point straight at an order's node and cancel it in O(1).

## Topic 14B — Build

## Unit 14.2 — Types and the book's interface

```cpp
// file: ob/include/ob/order_book.hpp
#pragma once
#include <cstdint>
#include <functional>
#include <list>
#include <map>
#include <memory_resource>
#include <optional>
#include <unordered_map>
#include <vector>

namespace ob {

using OrderId = std::uint64_t;
using Price = std::int64_t;                       // in ticks: never a floating-point price
using Qty = std::int64_t;

enum class Side : std::uint8_t { Buy, Sell };
enum class Kind : std::uint8_t { Limit, IOC };    // IOC: immediate-or-cancel, never rests

struct Order {
    OrderId id;
    Side side;
    Price price;
    Qty qty;
    Kind kind = Kind::Limit;
};

struct Trade {
    OrderId taker;                                // the incoming order
    OrderId maker;                                // the resting order it matched
    Price price;                                  // always the maker's price
    Qty qty;
    friend bool operator==(const Trade&, const Trade&) = default;
};

enum class AddResult : std::uint8_t { Ok, DuplicateId, InvalidQty };

class OrderBook {
public:
    explicit OrderBook(std::pmr::memory_resource* mr = std::pmr::get_default_resource());
    OrderBook(const OrderBook&) = delete;         // the index holds iterators into the book: no copies
    OrderBook& operator=(const OrderBook&) = delete;

    AddResult add(const Order& order, std::vector<Trade>& trades);   // appends any trades
    bool cancel(OrderId id);

    [[nodiscard]] std::optional<Price> best_bid() const;
    [[nodiscard]] std::optional<Price> best_ask() const;
    [[nodiscard]] Qty depth(Side side, Price price) const;
    [[nodiscard]] std::size_t order_count() const { return index_.size(); }

private:
    struct Resting { OrderId id; Qty qty; };
    struct Level {
        std::pmr::list<Resting> queue;            // FIFO: front is the oldest order
        Qty total = 0;
        explicit Level(std::pmr::memory_resource* mr) : queue{mr} {}
    };
    using Bids = std::pmr::map<Price, Level, std::greater<>>;   // begin() = highest bid
    using Asks = std::pmr::map<Price, Level, std::less<>>;      // begin() = lowest ask
    struct Locator { Side side; Price price; std::pmr::list<Resting>::iterator it; };

    template <typename Book> void match(Book& opposite, Order& incoming, std::vector<Trade>& trades);
    template <typename Book> void rest(Book& book, const Order& order);
    template <typename Book> void remove(Book& book, const Locator& loc);

    std::pmr::memory_resource* mr_;
    Bids bids_;
    Asks asks_;
    std::pmr::unordered_map<OrderId, Locator> index_;
};

}  // namespace ob
```

Two details matter. Bids and asks are different `map` types — only the comparator differs — so that `begin()` is the best price on both sides; the private member templates let one piece of matching code serve both. And `OrderBook` is non-copyable: a copy would duplicate the maps but its index would still hold iterators into the *original's* lists (Rule of Five thinking from Chapter 2, and DT-48 below).

### Deep & Tricky

**DT-48. A class that stores iterators or pointers into its own members must not use the default copy and move.** *(Language rules, [class.copy]; Core Guidelines C.21)* The defaulted copy constructor copies the containers *and* the iterators, which still point into the source object's containers — a perfect recipe for a use-after-free when the source dies. Moving `std::map` and `std::list` keeps node addresses (so their iterators stay valid in the new owner), but that is a property of node-based containers, not a general rule; for `vector` or a small-buffer type it would be wrong. Delete copy (and move, unless you have reasoned it through) or write them to rebuild the internal pointers. Here copy is deleted, and since a user-declared copy constructor suppresses the implicit move, move is unavailable too.

## Unit 14.3 — Matching

```cpp
// file: ob/src/order_book.cpp
#include "ob/order_book.hpp"
#include <algorithm>
#include <iterator>

namespace ob {

OrderBook::OrderBook(std::pmr::memory_resource* mr) : mr_{mr}, bids_{mr}, asks_{mr}, index_{mr} {}

template <typename Book>
void OrderBook::match(Book& opposite, Order& in, std::vector<Trade>& trades) {
    while (in.qty > 0 && !opposite.empty()) {
        auto level = opposite.begin();                               // best opposite price
        bool crosses = in.side == Side::Buy ? level->first <= in.price : level->first >= in.price;
        if (!crosses) break;
        Level& lvl = level->second;
        while (in.qty > 0 && !lvl.queue.empty()) {
            Resting& maker = lvl.queue.front();                      // oldest order at this price
            Qty q = std::min(in.qty, maker.qty);
            trades.push_back({in.id, maker.id, level->first, q});   // at the maker's price
            in.qty -= q;
            maker.qty -= q;
            lvl.total -= q;
            if (maker.qty == 0) {
                index_.erase(maker.id);
                lvl.queue.pop_front();
            }
        }
        if (lvl.queue.empty()) opposite.erase(level);
    }
}

template <typename Book>
void OrderBook::rest(Book& book, const Order& o) {
    auto [level, inserted] = book.try_emplace(o.price, mr_);
    Level& lvl = level->second;
    lvl.queue.push_back({o.id, o.qty});
    lvl.total += o.qty;
    index_.emplace(o.id, Locator{o.side, o.price, std::prev(lvl.queue.end())});
}

template <typename Book>
void OrderBook::remove(Book& book, const Locator& loc) {
    auto level = book.find(loc.price);
    Level& lvl = level->second;
    lvl.total -= loc.it->qty;
    lvl.queue.erase(loc.it);                                         // O(1): we hold the iterator
    if (lvl.queue.empty()) book.erase(level);
}

AddResult OrderBook::add(const Order& order, std::vector<Trade>& trades) {
    if (order.qty <= 0) return AddResult::InvalidQty;
    if (index_.contains(order.id)) return AddResult::DuplicateId;
    Order in = order;
    if (in.side == Side::Buy) match(asks_, in, trades);
    else match(bids_, in, trades);
    if (in.qty > 0 && in.kind == Kind::Limit) {
        if (in.side == Side::Buy) rest(bids_, in);
        else rest(asks_, in);
    }
    return AddResult::Ok;
}

bool OrderBook::cancel(OrderId id) {
    auto it = index_.find(id);
    if (it == index_.end()) return false;
    Locator loc = it->second;
    index_.erase(it);
    if (loc.side == Side::Buy) remove(bids_, loc);
    else remove(asks_, loc);
    return true;
}

std::optional<Price> OrderBook::best_bid() const {
    if (bids_.empty()) return std::nullopt;
    return bids_.begin()->first;
}

std::optional<Price> OrderBook::best_ask() const {
    if (asks_.empty()) return std::nullopt;
    return asks_.begin()->first;
}

Qty OrderBook::depth(Side side, Price price) const {
    auto total = [price](const auto& book) -> Qty {
        auto level = book.find(price);
        return level == book.end() ? 0 : level->second.total;
    };
    return side == Side::Buy ? total(bids_) : total(asks_);
}

}  // namespace ob
```

The matching loop is the whole engine: take the best opposite level; while it crosses, trade against its oldest order; remove fully filled orders from the queue and the index; remove empty levels. Because levels are removed as soon as they empty, `begin()` is always a real price — which is what keeps R1's "no crossed book" invariant: an incoming order rests only once nothing on the other side crosses it.

### Deep & Tricky

**DT-49. Erasing the element a reference points to, then using the reference, is the most common bug in code like this.** *(Language rules: lifetime; Unit 13.2)* In the matching loop, `maker` is a reference into the list node; `index_.erase(maker.id)` must come *before* `lvl.queue.pop_front()`, because after `pop_front` the node — and `maker.id` with it — is gone. Swapping those two lines compiles cleanly, usually passes simple tests (freed memory still holds the old value), and is caught at once by AddressSanitizer as a `heap-use-after-free`. This is one reason the test suite in Unit 14.4 always runs under ASan.

## Topic 14C — Verify

## Unit 14.4 — Tests, including a differential test

Unit tests check the cases you can think of. A **differential test** checks the cases you cannot: it runs thousands of random operations through both the real engine and a deliberately simple **reference model** — a flat vector searched exhaustively, obviously correct and hopelessly slow — and requires them to agree on every trade and on the best prices after every operation. (It is the same idea as the fuzzer's round-trip property in Chapter 13: an oracle that needs no hand-written expected values.)

```cpp
// file: ob/tests/order_book_test.cpp
#include <gtest/gtest.h>
#include <algorithm>
#include <random>
#include <vector>
#include "ob/order_book.hpp"

using namespace ob;

namespace {
std::vector<Trade> add(OrderBook& b, OrderId id, Side s, Price p, Qty q, Kind k = Kind::Limit) {
    std::vector<Trade> t;
    EXPECT_EQ(b.add({id, s, p, q, k}, t), AddResult::Ok);
    return t;
}
}  // namespace

TEST(OrderBook, RestsWhenNotCrossing) {
    OrderBook b;
    EXPECT_TRUE(add(b, 1, Side::Buy, 99, 10).empty());
    EXPECT_TRUE(add(b, 2, Side::Sell, 101, 5).empty());
    EXPECT_EQ(b.best_bid(), 99);
    EXPECT_EQ(b.best_ask(), 101);
    EXPECT_EQ(b.order_count(), 2u);
}

TEST(OrderBook, TradesAtMakerPrice) {
    OrderBook b;
    add(b, 1, Side::Sell, 100, 5);
    auto t = add(b, 2, Side::Buy, 105, 5);                // willing to pay 105, pays 100
    ASSERT_EQ(t.size(), 1u);
    EXPECT_EQ(t[0], (Trade{2, 1, 100, 5}));
    EXPECT_FALSE(b.best_ask());
    EXPECT_FALSE(b.best_bid());
}

TEST(OrderBook, PartialFillRestsRemainder) {
    OrderBook b;
    add(b, 1, Side::Sell, 100, 3);
    auto t = add(b, 2, Side::Buy, 100, 10);
    EXPECT_EQ(t, (std::vector<Trade>{{2, 1, 100, 3}}));
    EXPECT_EQ(b.best_bid(), 100);
    EXPECT_EQ(b.depth(Side::Buy, 100), 7);
}

TEST(OrderBook, PriceThenTimePriority) {
    OrderBook b;
    add(b, 1, Side::Sell, 100, 1);
    add(b, 2, Side::Sell, 100, 1);
    add(b, 3, Side::Sell, 99, 1);                         // better price, arrived last
    auto t = add(b, 4, Side::Buy, 100, 3);
    EXPECT_EQ(t, (std::vector<Trade>{{4, 3, 99, 1}, {4, 1, 100, 1}, {4, 2, 100, 1}}));
}

TEST(OrderBook, SweepsLevelsAndStopsAtLimit) {
    OrderBook b;
    add(b, 1, Side::Buy, 100, 2);
    add(b, 2, Side::Buy, 99, 2);
    add(b, 3, Side::Buy, 98, 2);
    auto t = add(b, 4, Side::Sell, 99, 10);               // takes 100 and 99, not 98
    EXPECT_EQ(t, (std::vector<Trade>{{4, 1, 100, 2}, {4, 2, 99, 2}}));
    EXPECT_EQ(b.best_bid(), 98);
    EXPECT_EQ(b.best_ask(), 99);                          // remainder rests at its limit
    EXPECT_EQ(b.depth(Side::Sell, 99), 6);
}

TEST(OrderBook, CancelRemovesOrderAndEmptyLevel) {
    OrderBook b;
    add(b, 1, Side::Buy, 100, 5);
    add(b, 2, Side::Buy, 100, 7);
    EXPECT_TRUE(b.cancel(1));
    EXPECT_EQ(b.depth(Side::Buy, 100), 7);
    EXPECT_TRUE(b.cancel(2));
    EXPECT_FALSE(b.best_bid());
    EXPECT_FALSE(b.cancel(2));                            // already gone
    EXPECT_FALSE(b.cancel(42));                           // never existed
}

TEST(OrderBook, RejectsInvalidOrdersWithoutSideEffects) {
    OrderBook b;
    std::vector<Trade> t;
    add(b, 1, Side::Sell, 100, 5);
    EXPECT_EQ(b.add({1, Side::Buy, 100, 5}, t), AddResult::DuplicateId);
    EXPECT_EQ(b.add({2, Side::Buy, 100, 0}, t), AddResult::InvalidQty);
    EXPECT_EQ(b.add({3, Side::Buy, 100, -4}, t), AddResult::InvalidQty);
    EXPECT_TRUE(t.empty());
    EXPECT_EQ(b.depth(Side::Sell, 100), 5);
}

TEST(OrderBook, IocNeverRests) {
    OrderBook b;
    add(b, 1, Side::Sell, 100, 2);
    auto t = add(b, 2, Side::Buy, 100, 5, Kind::IOC);
    EXPECT_EQ(t, (std::vector<Trade>{{2, 1, 100, 2}}));
    EXPECT_FALSE(b.best_bid());                           // the remaining 3 were cancelled
    EXPECT_EQ(b.order_count(), 0u);
}

// ---- differential test against a reference model ----
namespace {
class ReferenceBook {                                     // obviously correct, O(n) everything
public:
    AddResult add(Order o, std::vector<Trade>& trades) {
        if (o.qty <= 0) return AddResult::InvalidQty;
        for (const auto& r : rest_) if (r.o.id == o.id) return AddResult::DuplicateId;
        while (o.qty > 0) {
            Entry* best = nullptr;
            for (auto& r : rest_) {
                if (r.o.side == o.side) continue;
                bool crosses = o.side == Side::Buy ? r.o.price <= o.price : r.o.price >= o.price;
                bool better = !best || (o.side == Side::Buy ? r.o.price < best->o.price : r.o.price > best->o.price) ||
                              (r.o.price == best->o.price && r.seq < best->seq);
                if (crosses && better) best = &r;
            }
            if (!best) break;
            Qty q = std::min(o.qty, best->o.qty);
            trades.push_back({o.id, best->o.id, best->o.price, q});
            o.qty -= q;
            best->o.qty -= q;
            std::erase_if(rest_, [](const Entry& e) { return e.o.qty == 0; });
        }
        if (o.qty > 0 && o.kind == Kind::Limit) rest_.push_back({o, seq_++});
        return AddResult::Ok;
    }
    bool cancel(OrderId id) { return std::erase_if(rest_, [id](const Entry& e) { return e.o.id == id; }) > 0; }
    std::optional<Price> best(Side s) const {
        std::optional<Price> p;
        for (const auto& r : rest_)
            if (r.o.side == s && (!p || (s == Side::Buy ? r.o.price > *p : r.o.price < *p))) p = r.o.price;
        return p;
    }
    std::size_t size() const { return rest_.size(); }

private:
    struct Entry { Order o; std::uint64_t seq; };
    std::vector<Entry> rest_;
    std::uint64_t seq_ = 0;
};
}  // namespace

TEST(OrderBook, MatchesReferenceModelOnRandomOperations) {
    std::mt19937_64 rng{12345};
    for (int round = 0; round < 20; ++round) {
        OrderBook real;
        ReferenceBook ref;
        std::vector<OrderId> ids;
        OrderId next = 1;
        for (int op = 0; op < 2000; ++op) {
            std::vector<Trade> t1, t2;
            unsigned r = rng() % 100;
            if (r < 25 && !ids.empty()) {                           // cancel (possibly already filled)
                OrderId id = ids[rng() % ids.size()];
                ASSERT_EQ(real.cancel(id), ref.cancel(id)) << "op " << op;
            } else {
                OrderId id = r < 28 && !ids.empty() ? ids[rng() % ids.size()] : next++;   // some duplicate ids
                Order o{id, rng() % 2 ? Side::Buy : Side::Sell, 95 + static_cast<Price>(rng() % 11),
                        static_cast<Qty>(rng() % 6), rng() % 10 == 0 ? Kind::IOC : Kind::Limit};  // qty 0 sometimes
                ASSERT_EQ(real.add(o, t1), ref.add(o, t2)) << "op " << op;
                ASSERT_EQ(t1, t2) << "op " << op;
                ids.push_back(id);
            }
            ASSERT_EQ(real.best_bid(), ref.best(Side::Buy)) << "op " << op;
            ASSERT_EQ(real.best_ask(), ref.best(Side::Sell)) << "op " << op;
            ASSERT_EQ(real.order_count(), ref.size()) << "op " << op;
            if (real.best_bid() && real.best_ask()) ASSERT_LT(*real.best_bid(), *real.best_ask());   // never crossed
        }
    }
}
```

The random operations use a narrow price band (95–105) and small quantities so that orders cross constantly, levels empty and refill, cancels hit filled orders, and duplicate ids and zero quantities appear — the edge cases a hand-written test list tends to miss. Twenty rounds of 2,000 operations each are 40,000 operations checked trade by trade.

```cmake
# file: ob/CMakeLists.txt
cmake_minimum_required(VERSION 3.24)
project(ob VERSION 1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

option(OB_SANITIZE "Build everything with ASan, UBSan and libstdc++ assertions" OFF)
if(OB_SANITIZE)
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer -fno-sanitize-recover=all -D_GLIBCXX_ASSERTIONS)
  add_link_options(-fsanitize=address,undefined)
endif()

add_library(ob src/order_book.cpp)
target_include_directories(ob PUBLIC include)
target_compile_options(ob PRIVATE -Wall -Wextra -Wpedantic -Wshadow -Wconversion -Werror)

add_executable(replay tools/replay.cpp)
target_link_libraries(replay PRIVATE ob)
add_executable(gen tools/gen.cpp)

include(FetchContent)
FetchContent_Declare(googletest
  GIT_REPOSITORY https://github.com/google/googletest.git GIT_TAG v1.15.2
  FIND_PACKAGE_ARGS NAMES GTest)
FetchContent_MakeAvailable(googletest)
enable_testing()
add_executable(ob_test tests/order_book_test.cpp)
target_link_libraries(ob_test PRIVATE ob GTest::gtest_main)
include(GoogleTest)
gtest_discover_tests(ob_test)
```

## Topic 14D — Measure

## Unit 14.5 — The replay tool

Real matching engines are tested and tuned by **replaying** recorded order flow. The replay tool reads a file of operations — `A <id> <B|S> <price> <qty> [I]` to add (with `I` for IOC), `X <id>` to cancel — parses all of it *before* the clock starts (I/O is not what we are measuring), then applies every operation, timing each one individually to build a latency distribution. A checksum over all trades makes R5's determinism checkable: two runs, or two versions of the engine, produce the same trades exactly when their checksums match.

```cpp
// file: ob/tools/replay.cpp
#include <algorithm>
#include <charconv>
#include <chrono>
#include <cstdio>
#include <fstream>
#include <iterator>
#include <memory_resource>
#include <string>
#include <string_view>
#include <vector>
#include "ob/order_book.hpp"

namespace {
struct Op { bool cancel; ob::Order order; };

template <typename T>
bool number(std::string_view& s, T& out) {                       // parse one field, advance past it
    while (!s.empty() && s.front() == ' ') s.remove_prefix(1);
    auto [p, ec] = std::from_chars(s.data(), s.data() + s.size(), out);
    if (ec != std::errc{}) return false;
    s.remove_prefix(static_cast<std::size_t>(p - s.data()));
    return true;
}

std::vector<Op> load(const char* path) {
    std::ifstream in{path};
    std::string text{std::istreambuf_iterator<char>{in}, {}};
    std::vector<Op> ops;
    std::string_view rest{text};
    while (!rest.empty()) {
        auto nl = rest.find('\n');
        std::string_view line = rest.substr(0, nl);
        rest = nl == std::string_view::npos ? std::string_view{} : rest.substr(nl + 1);
        if (line.size() < 3) continue;
        Op op{line[0] == 'X', {}};
        std::string_view f = line.substr(1);
        if (!number(f, op.order.id)) continue;
        if (!op.cancel) {
            while (!f.empty() && f.front() == ' ') f.remove_prefix(1);
            if (f.empty()) continue;
            op.order.side = f.front() == 'B' ? ob::Side::Buy : ob::Side::Sell;
            f.remove_prefix(1);
            if (!number(f, op.order.price) || !number(f, op.order.qty)) continue;
            op.order.kind = f.find('I') != std::string_view::npos ? ob::Kind::IOC : ob::Kind::Limit;
        }
        ops.push_back(op);
    }
    return ops;
}

void run(const std::vector<Op>& ops, std::pmr::memory_resource* mr, const char* label) {
    ob::OrderBook book{mr};
    std::vector<ob::Trade> trades;
    trades.reserve(64);
    std::vector<std::uint32_t> ns(ops.size());
    std::uint64_t checksum = 1469598103934665603ull, ntrades = 0, volume = 0, rejected = 0;
    auto start = std::chrono::steady_clock::now();
    for (std::size_t i = 0; i < ops.size(); ++i) {
        auto t0 = std::chrono::steady_clock::now();
        if (ops[i].cancel) book.cancel(ops[i].order.id);
        else if (book.add(ops[i].order, trades) != ob::AddResult::Ok) ++rejected;
        auto t1 = std::chrono::steady_clock::now();
        ns[i] = static_cast<std::uint32_t>(std::chrono::duration_cast<std::chrono::nanoseconds>(t1 - t0).count());
        for (const auto& t : trades) {                                 // FNV-1a over every trade
            for (std::uint64_t v : {t.taker, t.maker, static_cast<std::uint64_t>(t.price), static_cast<std::uint64_t>(t.qty)})
                checksum = (checksum ^ v) * 1099511628211ull;
            ++ntrades;
            volume += static_cast<std::uint64_t>(t.qty);
        }
        trades.clear();
    }
    double total_ms = std::chrono::duration<double, std::milli>(std::chrono::steady_clock::now() - start).count();
    std::sort(ns.begin(), ns.end());
    auto pct = [&](double p) { return ns[static_cast<std::size_t>(p * static_cast<double>(ns.size() - 1))]; };
    std::printf("%-8s %zu ops in %.0f ms (%.2f M ops/s), %llu trades, volume %llu, rejected %llu\n", label, ops.size(),
                total_ms, static_cast<double>(ops.size()) / total_ms / 1000.0, static_cast<unsigned long long>(ntrades),
                static_cast<unsigned long long>(volume), static_cast<unsigned long long>(rejected));
    std::printf("         latency ns: p50 %u  p90 %u  p99 %u  p99.9 %u  max %u\n", pct(0.5), pct(0.9), pct(0.99),
                pct(0.999), ns.back());
    std::printf("         resting orders %zu, best bid %lld, best ask %lld, trade checksum %016llx\n",
                book.order_count(), static_cast<long long>(book.best_bid().value_or(-1)),
                static_cast<long long>(book.best_ask().value_or(-1)), static_cast<unsigned long long>(checksum));
}
}  // namespace

int main(int argc, char* argv[]) {
    if (argc < 2) { std::fprintf(stderr, "usage: replay FILE [default|pool|both]\n"); return 2; }
    std::vector<Op> ops = load(argv[1]);
    std::string_view mode = argc > 2 ? argv[2] : "default";
    if (mode == "default" || mode == "both") run(ops, std::pmr::new_delete_resource(), "default");
    if (mode == "pool" || mode == "both") {
        std::pmr::unsynchronized_pool_resource pool;                   // recycles list/map/hash nodes
        run(ops, &pool, "pool");
    }
}
```

The generator produces a reproducible order flow shaped like a real market's: prices cluster around a slowly wandering mid-price; most orders are passive limit orders near the top of the book; a minority are aggressive and cross; and a large fraction of all messages are cancels, as on real exchanges.

```cpp
// file: ob/tools/gen.cpp
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <random>
#include <vector>

int main(int argc, char* argv[]) {
    long n = argc > 1 ? std::atol(argv[1]) : 1'000'000;
    std::mt19937_64 rng{42};
    std::normal_distribution<double> offset{0.0, 6.0};
    std::uniform_int_distribution<int> qty{1, 100}, pct{0, 99};
    std::vector<unsigned long> live;                     // ids that may still be resting
    double mid = 10'000;
    unsigned long next = 1;
    for (long i = 0; i < n; ++i) {
        mid += (pct(rng) - 49.5) * 0.002;                // slow random walk
        int r = pct(rng);
        if (r < 40 && !live.empty()) {                   // 40%: cancel a random earlier order
            std::size_t k = rng() % live.size();
            std::printf("X %lu\n", live[k]);
            live[k] = live.back();
            live.pop_back();
            continue;
        }
        bool buy = rng() % 2;
        long away = std::lround(std::abs(offset(rng)));
        bool aggressive = r >= 92;                       // 8%: crosses the spread
        long price = std::lround(mid) + (buy == aggressive ? away : -away) + (buy ? -1 : 1);
        std::printf("A %lu %c %ld %d%s\n", next, buy ? 'B' : 'S', price, qty(rng), r >= 97 ? " I" : "");
        live.push_back(next++);
    }
}
```

## Unit 14.6 — Building, testing, and measuring

First the sanitizer build, which runs the whole test suite — unit tests and the 40,000-operation differential test — under ASan, UBSan, and the libstdc++ assertions:

```console
$ cd ob && rm -rf build-asan && cmake -S . -B build-asan -DCMAKE_BUILD_TYPE=Debug -DOB_SANITIZE=ON > /dev/null && cmake --build build-asan -j2 2>&1 | grep -E 'warning|error' ; ./build-asan/ob_test --gtest_brief=1 2>&1 | tail -n 2
[==========] 9 tests from 1 test suite ran. (675 ms total)
[  PASSED  ] 9 tests.
```

Then the release build, a million-operation flow, and the replay with both allocation strategies:

```console
$ cd ob && rm -rf build && cmake -S . -B build > /dev/null && cmake --build build -j2 2>&1 | grep -E 'warning|error' ; ./build/ob_test --gtest_brief=1 2>&1 | tail -n 1
[  PASSED  ] 9 tests.
$ cd ob && ./build/gen 1000000 > flow.txt && wc -l < flow.txt && head -n 4 flow.txt
1000000
A 1 S 10002 38
X 1
A 2 S 10006 83
A 3 S 10005 43
$ cd ob && ./build/replay flow.txt both
default  1000000 ops in 226 ms (4.42 M ops/s), 326452 trades, volume 8327171, rejected 0
         latency ns: p50 134  p90 350  p99 595  p99.9 957  max 2538947
         resting orders 73936, best bid 10031, best ask 10032, trade checksum b11e1a24fd22ee20
pool     1000000 ops in 247 ms (4.05 M ops/s), 326452 trades, volume 8327171, rejected 0
         latency ns: p50 179  p90 358  p99 627  p99.9 1110  max 1992016
         resting orders 73936, best bid 10031, best ask 10032, trade checksum b11e1a24fd22ee20
$ cd ob && ./build/replay flow.txt both
default  1000000 ops in 227 ms (4.40 M ops/s), 326452 trades, volume 8327171, rejected 0
         latency ns: p50 132  p90 344  p99 577  p99.9 932  max 2592669
         resting orders 73936, best bid 10031, best ask 10032, trade checksum b11e1a24fd22ee20
pool     1000000 ops in 240 ms (4.17 M ops/s), 326452 trades, volume 8327171, rejected 0
         latency ns: p50 176  p90 348  p99 597  p99.9 1035  max 1889143
         resting orders 73936, best bid 10031, best ask 10032, trade checksum b11e1a24fd22ee20
```

Where does the time go? A CPU profile with `perf` (software clock sampling; the C book's Chapter 11):

```console
$ cd ob && perf record -q -e cpu-clock -o perf.data ./build/replay flow.txt default > /dev/null 2>&1; perf report -i perf.data --stdio --no-children 2>/dev/null | grep -E '^ +[0-9]' | head -n 7 | sed -E 's/\(.*//; s/std::_Hashtable<[^>]*>.*::_M_/std::_Hashtable<...>::_M_/; s/std::__introsort_loop<.*/std::__introsort_loop<...>/' | cut -c1-100
    14.19%  replay   replay                [.] ob::OrderBook::cancel
    13.53%  replay   [vdso]                [.] __vdso_clock_gettime
    11.60%  replay   replay                [.] main
     8.43%  replay   replay                [.] void std::__introsort_loop<...>
     5.62%  replay   replay                [.] ob::OrderBook::add
     3.92%  replay   [kernel.kallsyms]     [k] exc_page_fault
     3.92%  replay   replay                [.] std::_Hashtable<...>::_M_erase
```

### Reading the results

- **Correctness first.** All nine tests pass in both builds, including 40,000 operations compared trade by trade against the reference model, under ASan, UBSan, and the hardened library. The two replays of the same flow produced the same trade checksum, with both allocators — R5 holds.
- **Throughput:** roughly 4 to 4.5 million operations per second on one core of this small VM, and the median operation takes on the order of 100–150 ns. Part of that is the measurement itself: each operation calls `steady_clock::now()` twice, and the profile shows `__vdso_clock_gettime` as one of the top entries. The profile also shows `main` (the file parser, inlined into it) and `std::sort` (the replay tool sorting a million latencies afterwards) — neither is the engine.
- **The engine's own hot spots** are `cancel`, `add`, and the hash-table operations of the id index (`_M_erase` is in the top seven; the insert follows just below). A cancel looks up the id (a hash-table probe, likely a cache miss with ~74,000 resting orders), finds the price level in the `std::map` (more pointer-chasing), and unlinks a list node. That is where a faster design would start (Unit 14.7).
- **The tail:** p99.9 is several times the median, and the maximum is in the milliseconds. The maximum single operation is dominated by events outside the algorithm — page faults as memory is first touched (`exc_page_fault` appears in the profile), the scheduler, and occasional rehashes of the id index as it grows. Production engines pre-fault and reserve memory at start-up, pin threads to isolated cores, and report p99.9 and p99.99 rather than the mean, because the tail is what traders feel.
- **The pool allocator did not help — it was slower.** This was the experiment's hypothesis (Chapter 12's arena result suggested it), and it failed: `std::pmr::unsynchronized_pool_resource` was consistently slower than the default `new`/`delete`, in throughput and at the median. glibc's `malloc` already keeps per-thread caches of recently freed small blocks (tcache), which is close to what a pool does, and the pool resource adds its own bookkeeping and an extra virtual call through `memory_resource`. Because the allocator is a constructor argument, trying the hypothesis cost one line, and rejecting it costs nothing — which is P-7 in practice, and the reason the design made allocation pluggable.

### Pattern

**P-9. Test a fast implementation against a slow, obviously correct one.** *(Practice: differential testing; used for compilers, databases, and exchange matching engines)* Write the simplest possible model of the behaviour — linear scans, no clever data structures — and drive both implementations with the same random operations, comparing every observable result. The model is easy to get right by inspection; the comparison then extends that confidence to the fast version. Keep the random seed fixed so failures reproduce, and narrow the input space (few prices, small quantities) so that interesting interactions happen often. *Trade-off:* a second implementation to maintain, in exchange for tests that find the bugs nobody thought to write a case for.

## Unit 14.7 — Design review and where to go next

A review of the reference implementation against its requirements:

| Requirement | Status | Evidence |
|---|---|---|
| R1 price-time priority, maker price, never crossed | met | unit tests; differential test asserts `best_bid < best_ask` after every op |
| R2 operations | met | unit tests for each operation |
| R3 validation without side effects | met | `RejectsInvalidOrdersWithoutSideEffects`; random zero quantities and duplicate ids in the differential test |
| R4 complexity and latency report | met | O(log L) map, O(1) index; replay percentiles |
| R5 determinism | met | identical checksums across runs and allocators |

**Known limitations and next steps** (pick at least two for your own capstone):

1. **A flat price ladder.** Replace the `std::map` of levels with an array indexed by `price - base` over a bounded band, plus a bitmap or cached best price for finding the next non-empty level. Typical engines get several-fold speed-ups here; measure it with the replay tool and the same checksum.
2. **Intrusive order nodes from a pre-allocated pool.** Store orders in one `std::vector` (indices instead of pointers), with the FIFO as an intrusive doubly linked list of indices, and the id index as an open-addressing table (Chapter 12's project). No allocation at all on the hot path.
3. **Modify orders.** Reducing quantity keeps time priority; changing price loses it (cancel + add). Add both, with tests and model support.
4. **Market orders and self-trade prevention**, both common exchange features.
5. **An event log and snapshots** so the book survives restarts (the C capstone's write-ahead log, applied here), with recovery tested by replaying the log and comparing checksums.
6. **A network front end** — the C capstone's event loop, or Asio — with one matching thread fed by a lock-free single-producer/single-consumer queue (Chapter 10).
7. **Fuzz the replay parser** (Chapter 13) and **benchmark individual operations** with Google Benchmark (Chapter 11).

### Mastery check

1. **Predict.** In `OrderBook::match`, `index_.erase(maker.id)` comes before `lvl.queue.pop_front()`. What happens if the two lines are swapped, in a release build and in the ASan build?
2. **Predict.** If `OrderBook`'s copy constructor were not deleted, and a test copied a book with resting orders and then cancelled an order in the copy, what would happen?

## Answers to the Chapter 14 prediction questions

**14.7 Q1.** After `pop_front()` the list node holding `maker` is freed, so `index_.erase(maker.id)` reads freed memory. In a release build it usually still "works" — the freed node typically still holds the old id — so every test may pass. The ASan build reports a `heap-use-after-free` in `match` on the first fully filled order (DT-49). This was checked while writing the book: with the two lines swapped, the release build passed all nine tests, and the ASan build failed at once.

**14.7 Q2.** The defaulted copy would copy the maps and the index, but the copied `Locator`s would hold iterators into the *original* book's lists. `cancel` on the copy would erase a node from the original's list through the copy's iterator — corrupting the original (wrong totals, a list whose size no longer matches) and leaving the copy's own node in place; when either book is destroyed, the result is a double free or a use-after-free (DT-48).

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | "C++ is a superset of C" is false. | Unit 0.1 |
| DT-2 | Two different definitions of the same inline function or class across TUs: undefined behaviour, usually silent. | Unit 0.3 |
| DT-3 | An exception must not propagate out of a function called from C code: C frames have no unwind information to run destructors with, so the result ranges from `std::terminate` to skipped cleanup — and the C++ standard leaves crossing language boundaries entirely to the implementation. | Unit 0.4 |
| DT-4 | `auto x{1, 2};` is an error, but `auto x = {1, 2};` is an `std::initializer_list<int>`. | Unit 1.1 |
| DT-5 | Lifetime extension applies only to a reference bound *directly* to a temporary. | Unit 1.2 |
| DT-6 | The *static initialisation order fiasco*: global objects in different translation units are initialised in an unspecified order, so a global whose constructor uses another file's global may see it before it has been constructed. `constinit` turns the bug into a compile error by insisting that the initialiser is a constant expression — run before any dynamic initialisation at all. | Unit 1.3 |
| DT-7 | A lambda that captures by reference must not outlive what it captured. | Unit 1.5 |
| DT-8 | `std::string_view` is not null-terminated. | Unit 1.6 |
| DT-9 | `"abc" + 1` and `'a' + "bc"` compile — as pointer arithmetic, not string concatenation — because string literals are arrays of `char`, not `std::string`s. `std::string s = 'a' + "bc";` adds 97 to the address of `"bc"` and reads far past it. | Unit 1.6 |
| DT-10 | `auto [a, b] = expr;` does not bind `a` and `b` to the members of `expr`: it makes one hidden *copy* of the whole object and binds the names to that copy's members. Modifying `a` changes nothing outside; to modify the original, write `auto& [a, b]`, and to avoid the copy while only reading, `const auto& [a, b]`. | Unit 1.7 |
| DT-11 | A constructor with one parameter (and no `explicit`) is an implicit conversion: `void send(Packet p); send(42);` silently builds a `Packet` from 42. Mark single-argument constructors `explicit` unless the conversion is genuinely meant to be invisible (like `std::string` from `const char*`). | Unit 2.2 |
| DT-12 | A destructor must not throw. | Unit 2.3 |
| DT-13 | Declaring a destructor silently disables move operations. | Unit 2.6 |
| DT-14 | A move constructor that is not `noexcept` makes `std::vector` copy instead of move. | Unit 3.2 |
| DT-15 | A moved-from standard library object is in a "valid but unspecified state": you may assign to it, destroy it, or call functions without preconditions (`size()`, `clear()`, `empty()`), but you must not assume it is empty. A moved-from `std::string` holding a short string usually still contains its characters. | Unit 3.2 |
| DT-16 | "Pass by value and move" (`void set(std::string s) { name_ = std::move(s); }`) costs one extra move compared with two overloads (`const std::string&` and `std::string&&`), and it allocates on every call where the overloads would reuse the existing buffer. For constructors it is the right default; for a setter called in a loop on a long-lived object, the `const&` overload can be much cheaper. | Unit 3.5 |
| DT-17 | `std::vector<bool>` is not a container of `bool`s: it packs bits, so `v[i]` returns a proxy object, not a `bool&`. `auto x = v[0];` gives a proxy that still refers into the vector, `bool* p = &v[0];` does not compile, and two threads writing different elements race on the same byte. | Unit 4.1 |
| DT-18 | `map[key]` on a missing key inserts it. | Unit 4.2 |
| DT-19 | References and pointers to `unordered_map` elements survive rehashing; iterators do not. | Unit 4.2 |
| DT-20 | An invalid comparator can make `std::sort` crash, not just mis-sort. | Unit 4.4 |
| DT-21 | Some views cache their `begin()` (for example `filter_view`, so that repeated iteration stays cheap). Iterating a cached view, then modifying the underlying container, then iterating the view again gives results based on the stale cached position — and a view that caches is not usable through a `const&`. | Unit 4.5 |
| DT-22 | Checking and then acting on a file is a race: between `fs::exists(p)` and `std::ofstream(p)`, another process can create, delete, or replace `p` (a *time-of-check to time-of-use* bug, TOCTOU). Do the operation and handle its failure instead of checking first; for "create only if absent", `std::fstream` has no exclusive mode before C++23's `std::ios::noreplace`, so use POSIX `open(O_CREAT ¦ O_EXCL)` or C11 `fopen(p, "wx")`. | Unit 4.7 |
| DT-23 | `std::regex` in libstdc++ uses a recursive backtracking matcher: a long input against a pattern with nested repetition can take exponential time or overflow the stack and crash. It is also several times slower than RE2, PCRE2, or a hand-written parser even on simple patterns. | Unit 4.8 |
| DT-24 | Two `shared_ptr`s created separately from the same raw pointer will both delete it. | Unit 5.3 |
| DT-25 | `shared_ptr`'s reference count is thread-safe; the object it points to is not, and neither is one `shared_ptr` *variable* shared between threads. Two threads copying *their own* `shared_ptr`s to the same object is fine; one thread assigning to a `shared_ptr` while another reads that same variable is a data race. | Unit 5.3 |
| DT-26 | Passing `shared_ptr` by value in hot code costs two atomic operations per call. | Unit 5.5 |
| DT-27 | An exception thrown while a constructor is running means the object never existed: its destructor does *not* run. Members and base classes that were already fully constructed *are* destroyed — so a class that owns two raw resources and acquires them in its constructor body leaks the first if the second throws. | Unit 6.2 |
| DT-28 | `noexcept` is not checked at compile time. | Unit 6.3 |
| DT-29 | Dereferencing an `expected` or `optional` that holds no value is undefined behaviour. | Unit 6.5 |
| DT-30 | Template argument deduction never considers implicit conversions: `std::max(1, 2.0)` does not compile, because `T` would have to be both `int` and `double`. And `T` deduced from an array argument passed by value decays to a pointer, while `T&` keeps the array type and its size. | Unit 7.2 |
| DT-31 | Function templates cannot be partially specialised, and a full specialisation does not take part in overload resolution: the compiler first picks the best *primary template* among the overloads, and only then looks at that template's specialisations. A specialisation can therefore be skipped for a call that seems to match it exactly. | Unit 7.5 |
| DT-32 | Template errors are reported deep inside the instantiation. | Unit 7.6 |
| DT-33 | Overloads constrained by unrelated concepts are ambiguous; subsumption only works through named concepts. | Unit 8.1 |
| DT-34 | Undefined behaviour in a constant expression is a compile error. | Unit 8.4 |
| DT-35 | Without `override`, a typo creates a new function instead of overriding. | Unit 9.2 |
| DT-36 | Deleting a derived object through a base pointer without a virtual destructor is undefined behaviour. | Unit 9.2 |
| DT-37 | A derived class function with the same *name* as a base class function hides *every* base overload of that name, even ones with different parameters: `d.f(1)` stops finding `Base::f(int)` once `Derived` declares `f(std::string)`. Bring the base overloads back with `using Base::f;`. | Unit 9.3 |
| DT-38 | A C-style cast can silently become a `reinterpret_cast` or `const_cast`. | Unit 9.6 |
| DT-39 | `std::lock_guard(m_);` without a variable name locks and immediately unlocks. | Unit 10.2 |
| DT-40 | `std::atomic<T>` is not lock-free for every `T`. | Unit 10.4 |
| DT-41 | The `std::future` returned by `std::async(std::launch::async, ...)` blocks in its destructor until the task finishes. Ignoring the return value therefore runs the "asynchronous" tasks one after another: `std::async(std::launch::async, f); std::async(std::launch::async, g);` waits for `f` before starting `g`. | Unit 10.5 |
| DT-42 | A coroutine's reference parameters can dangle. | Unit 10.7 |
| DT-43 | A benchmark whose result is unused may measure nothing. | Unit 11.3 |
| DT-44 | Objects allocated from a `monotonic_buffer_resource` must not outlive it. | Unit 12.3 |
| DT-45 | `[[likely]]` and `[[unlikely]]` only change code layout, never correctness, and a wrong hint makes code slower. Profile-guided optimisation (`-fprofile-generate`, run, `-fprofile-use`) usually beats hand-written hints, because it uses measured branch frequencies for every branch, not just the ones you annotated. | Unit 12.5 |
| DT-46 | `std::string_view` parameters are safe; `std::string_view` return values and members are where dangling happens. | Unit 13.1 |
| DT-47 | The range-for over a temporary's member dangles (before C++23). | Unit 13.2 |
| DT-48 | A class that stores iterators or pointers into its own members must not use the default copy and move. | Unit 14.2 |
| DT-49 | Erasing the element a reference points to, then using the reference, is the most common bug in code like this. | Unit 14.3 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Every resource is owned by exactly one RAII object. | Unit 2.3 |
| P-2 | Return by value; take sinks by value and move; read through `const&`. | Unit 3.5 |
| P-3 | Make ownership visible in types; observe with references. | Unit 5.5 |
| P-4 | Prepare on the side, commit with no-throw operations. | Unit 6.2 |
| P-5 | Choose the polymorphism mechanism by how the set of types changes. | Unit 9.5 |
| P-6 | Make every quality check a command that CI runs, not a habit people must remember. | Unit 11.5 |
| P-7 | Optimise in this order: algorithm, allocations, layout, then code generation — measuring each step. | Unit 12.5 |
| P-8 | Run every test suite in three build configurations: warnings-as-errors release, ASan+UBSan, and TSan for concurrent code. | Unit 13.4 |
| P-9 | Test a fast implementation against a slow, obviously correct one. | Unit 14.6 |

# Appendix B — Glossary

Short definitions of the terms used in this book. The unit given (where there is one) explains the term fully.

| Term | Meaning |
| --- | --- |
| ABI (application binary interface) | The binary-level contract between compiled pieces of code: calling convention, object layout, name mangling, vtable layout (Unit 0.4) |
| Abstract class | A class with at least one pure virtual function; cannot be instantiated (Unit 9.2) |
| ADL (argument-dependent lookup) | Lookup of an unqualified function name in the namespaces of its arguments' types; how `swap` and operators are found |
| Aggregate | A class or array with no user-declared constructors, no private members, and no virtual functions; initialised member by member with braces |
| Allocator | An object that supplies memory to a container; `std::pmr` makes it a run-time choice (Unit 12.3) |
| Arena (monotonic buffer) | An allocator that hands out memory by bumping a pointer and frees everything at once (Unit 12.3) |
| Atomic | An object whose operations are indivisible and free of data races (Unit 10.4) |
| `auto` | Lets the compiler deduce a variable's or return type from its initialiser (Unit 1.1) |
| Benchmark | A program that measures performance under controlled conditions; Google Benchmark automates repetition and statistics (Unit 11.3) |
| CMake | The de-facto build system generator for C++; describes targets and their usage requirements (Unit 11.1) |
| Coroutine | A function that can suspend and resume (`co_await`, `co_yield`, `co_return`) (Unit 10.7) |
| Concept | A named compile-time predicate on template arguments, used to constrain templates (Unit 8.1) |
| `const` correctness | Marking everything that should not change as `const`, so the compiler enforces it (Unit 1.3) |
| `constexpr` / `consteval` / `constinit` | May run at compile time / must run at compile time / must be initialised at compile time (Unit 1.3) |
| Copy elision | Constructing a returned or temporary object directly in its final place; guaranteed for prvalues since C++17 (Unit 3.4) |
| CRTP | Curiously recurring template pattern: a base class template parameterised by its derived class, for static polymorphism (Unit 9.5) |
| CTAD | Class template argument deduction: `std::vector v{1, 2}` deduces `vector<int>` (Unit 7.3) |
| Dangling reference | A reference, pointer, iterator, or view whose object has been destroyed; using it is UB (Unit 13.1) |
| Data race | Two threads accessing the same memory without synchronisation, at least one writing; UB (Unit 10.2) |
| Data-oriented design | Organising data by how it is accessed (e.g. struct of arrays) for cache efficiency (Unit 12.4) |
| Deduction | The process by which the compiler works out template arguments from function arguments (Unit 7.2) |
| Destructor | A member function that runs automatically when an object's lifetime ends; the basis of RAII (Unit 2.3) |
| Devirtualisation | The compiler replacing a virtual call with a direct call when it can prove the dynamic type (Unit 12.5) |
| Differential testing | Testing an implementation against a simpler reference implementation on random inputs (Unit 14.4) |
| Dynamic dispatch | Choosing which function to call at run time from the object's dynamic type, via the vtable (Unit 9.2) |
| Exception safety guarantees | No-throw, strong (commit or roll back), basic (valid but unspecified state) (Unit 6.2) |
| `explicit` | Prevents a constructor or conversion operator from being used for implicit conversions (Unit 2.2) |
| `std::expected` | A value or an error, returned instead of throwing (C++23) (Unit 6.5) |
| Fold expression | Applies a binary operator across a parameter pack: `(args + ...)` (Unit 8.2) |
| Forwarding reference | A `T&&` parameter where `T` is deduced; binds to anything and preserves value category (Unit 3.3) |
| Fuzzing | Feeding generated inputs to code to find crashes and property violations (Unit 13.6) |
| Hardening (library) | Precondition checks in the standard library, e.g. `_GLIBCXX_ASSERTIONS` (Unit 13.3) |
| Heterogeneous lookup | Looking up a container by a type other than its key type without converting (Unit 12.2) |
| Header-only library | A library whose code is entirely in headers (often templates), needing no separate compilation (Chapter 11 project) |
| Inlining | Replacing a call with the body of the called function; the optimisation that enables most others (Unit 12.5) |
| Instantiation | Generating concrete code from a template for particular arguments (Unit 7.6) |
| Invariant | A condition a class guarantees is always true between member-function calls (Unit 2.1) |
| Iterator | An object that refers to a position in a sequence; the glue between containers and algorithms (Unit 4.3) |
| Iterator invalidation | An iterator, reference, or pointer becoming unusable after a container operation (Units 4.3, 13.2) |
| `std::jthread` | A thread that joins automatically in its destructor and supports cooperative cancellation (Unit 10.1) |
| Lambda | An anonymous function object, optionally capturing variables (Unit 1.5) |
| Lifetime extension | Binding a temporary directly to a `const&` or `&&` reference extends its life to the reference's (Unit 13.1) |
| Linker | Combines object files and libraries into an executable, resolving symbols (Unit 0.3) |
| lvalue / rvalue / prvalue / xvalue | Value categories: has identity / can be moved from; prvalues are pure temporaries, xvalues are expiring objects (Unit 3.1) |
| Mangling | Encoding namespaces, classes, and parameter types into symbol names (Unit 0.4) |
| Memory order | The ordering constraints on atomic operations: `relaxed`, `acquire`, `release`, `seq_cst` (Unit 10.4) |
| Module | A C++20 unit of code compiled once and imported, replacing textual inclusion (Unit 0.5) |
| Move semantics | Transferring a resource from an object that is about to die instead of copying it (Unit 3.2) |
| `noexcept` | Declares that a function does not throw; enables optimisations such as moving in `vector` growth (Unit 6.3) |
| Non-type template parameter | A template parameter that is a value rather than a type: `std::array<int, 4>` (Unit 7.4) |
| ODR (One Definition Rule) | Every entity has exactly one definition in the program (inline ones: identical in every translation unit) (Unit 0.3) |
| Order book | The set of resting buy and sell orders in a market, organised by price (Unit 14.1) |
| Overload resolution | The rules that choose among functions with the same name (Unit 1.5) |
| `override` / `final` | Ask the compiler to check that a function overrides a virtual / forbid further overriding or derivation (Unit 9.2) |
| Parameter pack | A template parameter that accepts any number of arguments: `typename... Ts` (Unit 8.2) |
| Perfect forwarding | Passing arguments on with their value category preserved, using `std::forward` (Unit 3.3) |
| PMR | Polymorphic memory resources (`<memory_resource>`): allocators chosen at run time (Unit 12.3) |
| Polymorphism | One interface, many implementations: dynamic (virtual), static (templates, CRTP), or closed (`std::variant`) (Chapter 9) |
| Price-time priority | The matching rule: best price first, then earliest arrival (Unit 14.1) |
| RAII | Resource acquisition is initialisation: a resource's lifetime is tied to an object's (Unit 2.3) |
| Ranges and views | C++20 algorithms over whole ranges, and lazy, composable adaptors such as `views::filter` (Unit 4.5) |
| Reference | An alias for an existing object; cannot be null or reseated (Unit 1.2) |
| Rule of Zero / Three / Five | Define none of the special members, or all of the copy ones, or all five including the moves (Units 2.4, 3.2) |
| Sanitizer | Compiler instrumentation that detects bugs at run time: ASan, UBSan, TSan (Units 11.5, 13.4) |
| `shared_ptr` / `weak_ptr` | Shared ownership by reference counting / a non-owning observer of a shared object (Units 5.3, 5.4) |
| SFINAE | Substitution failure is not an error: removing overloads whose template substitution fails; largely replaced by concepts (Unit 8.5) |
| Slicing | Copying a derived object into a base object, losing the derived part (Unit 9.3) |
| Small-buffer / small-string optimisation | Storing small contents inside the object to avoid heap allocation (Unit 12.1) |
| Special member functions | Default constructor, destructor, copy and move constructors, copy and move assignment (Unit 2.6) |
| Specialisation | A custom version of a template for particular arguments, full or partial (Unit 7.5) |
| `std::span` | A non-owning view of a contiguous sequence (Unit 4.6) |
| Stack unwinding | Destroying local objects in reverse order as an exception propagates (Unit 6.1) |
| `std::string_view` | A non-owning view of a sequence of characters (Units 1.6, 13.1) |
| Structured bindings | `auto [a, b] = pair;` — naming the parts of an aggregate, pair, or tuple (Unit 1.7) |
| Template | A pattern from which the compiler generates functions or classes for given types or values (Chapter 7) |
| Translation unit | A source file after preprocessing; the unit of compilation (Unit 0.3) |
| Type erasure | Hiding different concrete types behind one non-template interface, as `std::function` does (Unit 9.5) |
| Type trait | A compile-time query or transformation of types, in `<type_traits>` (Unit 7.7) |
| Undefined behaviour (UB) | Behaviour for which the standard imposes no requirements (Chapter 13) |
| `unique_ptr` | Exclusive ownership of a heap object, freed automatically; zero overhead (Unit 5.2) |
| `std::variant` | A type-safe union holding one of a fixed set of types; used with `std::visit` (Unit 9.4) |
| Virtual function / vtable | A function dispatched on the dynamic type; implemented with a per-class table of function pointers (Unit 9.2) |
| Zero-overhead principle | What you don't use, you don't pay for; what you do use, you couldn't hand-code any better (Unit 0.1) |

# Appendix C — C++ standards and compiler support

C++ is defined by ISO/IEC 14882, revised every three years. Each revision is almost entirely backwards compatible; select one with `-std=`. The `gnu++` variants (`-std=gnu++20`) add GCC extensions; this book uses the strict modes so that non-standard code is reported.

| Standard | Year | Features used or discussed in this book |
| --- | --- | --- |
| C++98/03 | 1998/2003 | classes, inheritance, virtual functions, templates, exceptions, namespaces, the STL (containers, iterators, algorithms), `std::string`, iostreams, RTTI and the named casts |
| C++11 | 2011 | move semantics and rvalue references; `auto` and `decltype`; lambdas; range-`for`; `nullptr`; `enum class`; `constexpr`; `noexcept`; `override`/`final`; `= default`/`= delete`; uniform initialisation and `initializer_list`; variadic templates; `static_assert`; `unique_ptr`, `shared_ptr`, `weak_ptr`; `unordered_*` containers; `std::array`; `std::thread`, `mutex`, `condition_variable`, `atomic`, `future`; the memory model; `<chrono>`, `<random>`, `<regex>`; user-defined literals |
| C++14 | 2014 | generic lambdas; return type deduction; `make_unique`; relaxed `constexpr`; variable templates; digit separators and binary literals; heterogeneous lookup in ordered containers (`std::less<>`); `shared_timed_mutex` |
| C++17 | 2017 | structured bindings; `if constexpr`; `if`/`switch` with initialisers; guaranteed copy elision; class template argument deduction (CTAD); fold expressions; inline variables; `std::optional`, `std::variant`, `std::any`, `std::string_view`; `<filesystem>`; parallel algorithms (execution policies); `<memory_resource>` (`pmr`); `[[nodiscard]]`, `[[maybe_unused]]`, `[[fallthrough]]`; `std::scoped_lock`; `try_emplace`; `std::from_chars`/`to_chars` |
| C++20 | 2020 | **the baseline of this book.** Concepts and `requires`; ranges and views; coroutines; modules; three-way comparison `<=>` and defaulted `==`; `consteval`, `constinit`; designated initialisers; templated lambdas; `std::format`; `std::span`; `std::jthread` and stop tokens; `latch`, `barrier`, `counting_semaphore`; `std::atomic_ref` and `atomic::wait`; `[[likely]]`/`[[unlikely]]`; `std::erase_if`; `contains` on associative containers; heterogeneous lookup for unordered containers; `std::bit_cast`; `std::source_location`; `using enum` |
| C++23 | 2024 | `std::expected`; `std::print`/`println`; `std::generator`; deducing `this`; `if consteval`; multidimensional `operator[]`; `std::mdspan`; `std::flat_map`/`flat_set`; formatting of ranges; `ranges::to`, `views::zip`, `enumerate`, `chunk`, `slide`, `stride`; `std::unreachable`; `std::to_underlying`; `import std;`; lifetime extension of temporaries in range-`for` (P2718) |
| C++26 | expected 2026 | contracts (P2900: `pre`, `post`, `contract_assert`); static reflection (P2996); `std::execution` (senders/receivers); `std::inplace_vector`; hardened standard library preconditions; erroneous behaviour for uninitialised reads |

## Compiler support used in this book

| Feature | GCC 13 | GCC 14 | Clang 18 (with libstdc++) |
| --- | --- | --- | --- |
| C++20 language (concepts, coroutines, `<=>`, `consteval`) | yes | yes | yes |
| C++20 named modules (`export module`) | partial (`-fmodules-ts`) | yes (`-fmodules-ts`; Unit 0.5) | yes (`.cppm` files; CMake 3.28+ automates the build order) |
| `std::format` | yes | yes | yes |
| `std::print`/`println` | no | yes | yes (`-std=c++23`) |
| `std::expected` | yes | yes | no: libstdc++'s `<expected>` needs C++20 concepts at a level Clang 19 first reports |
| `std::generator` | no | yes | no (Clang 19) |
| Formatting of ranges (`println("{}", v)`) | no | no (GCC 15) | no |
| `std::flat_map`, `std::mdspan` | no | no (GCC 15) | no |
| Deducing `this` | no | yes | yes (without the feature-test macro) |
| Range-`for` temporary lifetime extension (P2718) | no | no (GCC 15) | no (Clang 19) |
| `import std;` | no | no (GCC 15) | no (Clang 19, with libc++) |

Microsoft Visual C++ (Visual Studio 2022, 17.10 and later) supports all the C++20 features in this book and most of C++23, including `import std;`. Apple Clang lags upstream Clang by roughly a year.

Every row above was checked with the compilers used for this book. Library support changes quickly: before relying on a newer feature, check the compiler-support tables on cppreference.com, and the "Status" pages of libstdc++ and libc++.

## Feature-test macros

Every library and language feature since C++20 has a **feature-test macro** in `<version>` (`__cpp_lib_expected`, `__cpp_lib_generator`, `__cpp_concepts`, …). Code that must build on several compilers can test them instead of compiler versions:

```text
#include <version>
#if defined(__cpp_lib_print) && __cpp_lib_print >= 202207L
    std::println("hello");
#else
    std::cout << "hello\n";
#endif
```
