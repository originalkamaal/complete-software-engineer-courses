---
title: "Java from Zero to Expert"
subtitle: "A complete course in modern Java (Java 21) — from the JVM and bytecode to a production-style REST service"
lang: en
---

# Preface

Java runs a large share of the world's backend software: banks, e-commerce, telecoms, big-data systems, and Android's ecosystem all grew on it. It has also changed a great deal. Modern Java has records, sealed types, pattern matching, lambdas and streams, `var`, text blocks, and virtual threads — a far more expressive language than the Java of older tutorials.

This book teaches **Java 21**, the current long-term-support baseline, the way professionals use it: immutable data by default, interfaces over inheritance, explicit error handling, tested code, and decisions checked with the JDK's own tools — the compiler's lint, `javap`, GC logs, `jcmd`, and Java Flight Recorder.

It begins below the language, with the JVM, bytecode, class loading, and the JIT compiler. It then covers the core language, classes and encapsulation, inheritance and sealed types, exceptions, generics, collections, lambdas and streams, I/O, build and test, concurrency, JVM internals, networking and JDBC, and service design. It ends with a capstone: a REST service with a SQL database, connection pool, cache, rate limiter, metrics, and graceful shutdown.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the Java Language Specification, the library's documented behaviour, HotSpot's implementation, or established practice.
2. **Evidence.** Every program shown with output was compiled and run on OpenJDK 21, and the output is real — including compiler errors, exceptions, and timings. When a first version of an example was wrong, or a measurement contradicted folklore, the book says so.
3. **Order.** Nothing is used before it is taught; later chapters refer back by unit number.

## What could not be run

The build machine for this book had no access to Maven Central. The consequences are stated where they arise: tests run with **JUnit 4.13.2** (bundled with Gradle) while JUnit 5 is shown; JDBC runs against **HSQLDB 2.2.8**, an old release that was available locally; and **Spring Boot, Mockito, Jackson, JMH, and Maven** are shown as source marked *not run here*. Everything else — which is nearly everything — uses only the JDK.

# How to use this book

## Who it is for

- **Beginners:** start at Chapter 0 and read in order. No previous programming is assumed, though it helps.
- **Programmers from other languages:** read Chapters 0–1 quickly; slow down for Chapters 2–5 and every *Deep & Tricky* entry.
- **Developers who learned Java 8 or earlier:** Chapters 3 (sealed types, patterns), 7 (streams), and 10 (virtual threads) cover the largest changes.
- **Working Java developers:** use Appendix A to test yourself, then focus on Chapters 5, 10, 11, and 14.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | The rules, with small complete programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, labelled with their source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions; answers at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a specification, a complete reference solution with real output, and a review checklist.

## How to study

1. **Type the programs and run them.** Single files run directly: `java Hello.java`.
2. **Predict before you run.** When you are wrong, find out why.
3. **Compile with `-Xlint:all`** and read every warning.
4. **Break things on purpose:** remove a `volatile`, mutate a map key, drop a `flip()` — and see what happens.
5. **Revise with Appendix A,** which lists every DT and P entry by title.

## How to read the labels

| Label | Meaning |
| --- | --- |
| *Guarantee: JLS / Java SE specification* | Required by the language or platform specification; true on every conforming JVM. |
| *Library behaviour* | Documented behaviour of the standard library. |
| *Implementation behaviour* | What OpenJDK's HotSpot JVM or `javac` does; may differ on other JVMs or change between versions. |
| *Tool behaviour* | What Gradle or Maven do. |
| *Design fact*, *Practice* | Engineering consequences and professional practice. |

## Toolchain and versions

All code was run on **Linux x86-64 (Ubuntu 24.04)** with **OpenJDK 21.0.12** (HotSpot, G1 by default), **Gradle 8.14.3** (offline), JUnit 4.13.2, and HSQLDB 2.2.8. Measurements were taken on a 2-core virtual machine with a 1.5 GB default maximum heap; compare ratios and shapes, not absolute numbers, and expect run-to-run variation (the JIT makes Java timings noisier than C's).

## Setting up

1. Install a JDK 21 or later: Eclipse Temurin (adoptium.net), or `sudo apt install openjdk-21-jdk`, `brew install openjdk@21`, or SDKMAN (`sdk install java 21-tem`).
2. Check: `java -version`, `javac -version`.
3. Use IntelliJ IDEA (the Community edition is free), or VS Code with the Java extension pack, or Eclipse.
4. For Chapters 9 and 13 with internet access, use the standard Gradle or Maven set-up shown there (Maven Central, JUnit 5).

# Roadmap

| Chapter | Title | You will be able to… |
| --- | --- | --- |
| 0 | The JVM, Bytecode, Class Loading, and the JDK | explain what runs your program, and inspect it |
| 1 | Core Language | use primitives, strings, arrays, control flow, and methods precisely |
| 2 | Classes, Objects, and Encapsulation | design immutable, well-encapsulated classes; use records and enums |
| 3 | Inheritance, Interfaces, Sealed Types, Pattern Matching | choose between open and closed hierarchies; match exhaustively |
| 4 | Exceptions | handle, translate, and design failures; manage resources |
| 5 | Generics in Depth | use bounds and wildcards; understand erasure |
| 6 | The Collections Framework | choose and use collections; know how they work inside |
| 7 | Lambdas, Streams, and Functional Style | write pipelines and collectors; know when not to |
| 8 | I/O, NIO, Files, and Serialization | read and write text and binary data safely and fast |
| 9 | Build and Test | build with Gradle/Maven; write unit tests with test doubles |
| 10 | Concurrency | write correct concurrent code; use virtual threads |
| 11 | JVM Internals | read GC logs, find leaks, profile with JFR |
| 12 | Networking, HTTP, and JDBC | write clients, servers, and database code with transactions |
| 13 | Building Services | apply patterns and layers; map plain Java to Spring Boot |
| 14 | Capstone: A Production-Style REST Service | build, test, and measure a complete service |
| A | Index of Deep & Tricky entries and Patterns | revise every entry by title |
| B | Glossary | look up any term |
| C | Java versions and features | see which release introduced each feature |

Chapters 0–7 are the **foundation**; read them in order. Chapters 8–12 can be read in any order after that (11 uses 10). Chapters 13 and 14 use the whole book.

# Chapter 0 — The JVM, Bytecode, Class Loading, and the JDK

*Goal: know what happens between `javac` and a running program — bytecode, class loading, interpretation, just-in-time compilation, garbage collection — and use the JDK's tools to see each step.*

Java source code is not compiled to machine code. `javac` compiles it to **bytecode** for the **Java Virtual Machine**; the JVM loads classes on demand, verifies them, interprets them, and compiles the hot parts to machine code while the program runs, guided by what it observes. Much of Java's behaviour — fast steady-state speed after a slow start, garbage collection pauses, `ClassNotFoundException` at run time, the cost of the first call to a method — follows from this design.

**What you will be able to do after this chapter**

- Compile and run Java with `javac` and `java`, run single-file programs directly, and experiment in `jshell`.
- Read bytecode with `javap`, and explain the JVM's stack machine and constant pool.
- Explain class loading and initialisation, and observe them.
- Explain tiered JIT compilation and warm-up, and observe them.
- Package code into a JAR, and inspect dependencies with `jdeps`.

## Topic 0A — Java and its platform

## Unit 0.1 — What Java is

**One-line summary:** Java is a statically typed, object-oriented, garbage-collected language that compiles to platform-independent bytecode, run by the JVM. The **JDK** (Java Development Kit) contains the compiler, the JVM, the standard library, and tools; **OpenJDK** is the open-source reference implementation, distributed by many vendors (Oracle, Eclipse Temurin, Amazon Corretto, Microsoft, Azul, Ubuntu's packages).

A new Java version is released every six months; every two years one is a **long-term support** (LTS) release: 8, 11, 17, **21** (this book's baseline), and 25. Features arrive first as **previews** (`--enable-preview`) before becoming permanent, so the language changes steadily but carefully.

### Deep & Tricky

**DT-1. Java's backwards compatibility is strong at the bytecode level: classes compiled for Java 8 run on Java 21 unchanged — but not the other way round.** *(Guarantee: Java SE specification; class file versions)* Each class file records a version (Java 21 is 65.0); a JVM refuses to load classes newer than itself with `UnsupportedClassVersionError`. To build for an older JVM, use `javac --release 17`, which checks both the language level *and* that you only use APIs that existed in that release — unlike the older `-source`/`-target` flags, which let a call to a newer API compile and then fail at run time with `NoSuchMethodError`.

## Unit 0.2 — Compiling and running

**One-line summary:** `javac Hello.java` produces `Hello.class`; `java Hello` starts a JVM that loads the class and calls its `public static void main(String[] args)`. Since Java 11, `java Hello.java` compiles a single source file in memory and runs it; `jshell` evaluates declarations and expressions interactively.

```java
// file: Hello.java
import java.util.List;

public class Hello {
    public static void main(String[] args) {
        String who = args.length > 0 ? args[0] : "world";
        System.out.println("Hello, " + who + "!");
        List<Integer> squares = List.of(1, 2, 3, 4).stream().map(n -> n * n).toList();
        System.out.println("squares " + squares + ", running on Java " + Runtime.version().feature()
                + " (" + System.getProperty("java.vm.name") + ")");
    }
}
```

```console
$ javac Hello.java && ls Hello.* && java Hello && java Hello.java JVM
Hello.class  Hello.java
Hello, world!
squares [1, 4, 9, 16], running on Java 21 (OpenJDK 64-Bit Server VM)
Hello, JVM!
squares [1, 4, 9, 16], running on Java 21 (OpenJDK 64-Bit Server VM)
$ printf 'int sq(int x) { return x * x; }\nsq(12)\n"abc".repeat(3)\n/exit\n' | jshell --feedback normal 2>&1 | sed -E 's/(jshell> )+//' | grep -vE '^\s*$|Welcome|For an introduction|Goodbye'
 int sq(int x) { return x * x; }|  created method sq(int)
 $2 ==> 144
 $3 ==> "abcabcabc"
```

<!-- expanded -->
### Going deeper: `javac` + `java`, single-file mode, and `jshell`

Java offers three ways to run code, and each fits a different job. `javac Hello.java` compiles to `Hello.class` (bytecode) and `java Hello` runs that class: the normal two-step build, which any number of source files can go through. `java Hello.java` (**source-file mode**, Java 11+) compiles in memory and runs immediately, with no `.class` file left behind — this book uses it for single-file examples. `jshell` is an interactive shell (a REPL) for trying expressions.

```java
// file: Args.java
public class Args {
    public static void main(String[] args) {
        System.out.println("received " + args.length + " argument(s)");
        for (int i = 0; i < args.length; i++) {
            System.out.println("  args[" + i + "] = " + args[i]);
        }
        System.out.println("java.version = " + System.getProperty("java.version").split("\\.")[0]);
    }
}
```

```console
$ javac Args.java && ls Args.class && java Args one "two words" && rm Args.class
Args.class
received 2 argument(s)
  args[0] = one
  args[1] = two words
java.version = 21
$ java Args.java x y
received 2 argument(s)
  args[0] = x
  args[1] = y
java.version = 21
$ printf 'int x = 6 * 7\nSystem.out.println("x is " + x)\n/exit\n' | jshell -q 2>&1 | grep -E "^x|x is"
jshell> jshell> x is 42
```

Unlike C's `argv`, `args[0]` is the first real argument, not the program's name. Two traps: in source-file mode the *first* class in the file is the one that runs, and the file may not depend on other source files you have not compiled.

## Topic 0B — Inside the JVM

## Unit 0.3 — Bytecode

**One-line summary:** Bytecode is the instruction set of a **stack machine**: instructions push values onto an operand stack, operate on the top of it, and store results into numbered **local variable** slots. Each class file also has a **constant pool** holding the names of classes, methods, and fields it refers to, and literal constants. `javap -c` disassembles a class; `-v` shows the constant pool.

```java
// file: Calc.java
public class Calc {
    static int sumOfSquares(int n) {
        int total = 0;
        for (int i = 1; i <= n; i++) {
            total += i * i;
        }
        return total;
    }

    static String label(String name, int n) {
        return name + "=" + n;
    }

    public static void main(String[] args) {
        System.out.println(label("sumOfSquares(10)", sumOfSquares(10)));
    }
}
```

```console
$ javac Calc.java && java Calc && javap -c Calc | sed -n '/sumOfSquares(int)/,/^$/p'
sumOfSquares(10)=385
  static int sumOfSquares(int);
    Code:
       0: iconst_0
       1: istore_1
       2: iconst_1
       3: istore_2
       4: iload_2
       5: iload_0
       6: if_icmpgt     21
       9: iload_1
      10: iload_2
      11: iload_2
      12: imul
      13: iadd
      14: istore_1
      15: iinc          2, 1
      18: goto          4
      21: iload_1
      22: ireturn
$ javap -c Calc | sed -n '/label(java.lang.String, int)/,/^$/p'
  static java.lang.String label(java.lang.String, int);
    Code:
       0: aload_0
       1: iload_1
       2: invokedynamic #7,  0              // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;I)Ljava/lang/String;
       7: areturn
$ javap -v Calc | grep -E '^ +minor|^ +major'
  minor version: 0
  major version: 65
```

The loop compiles to exactly the stack operations you would expect: `iload`/`istore` move ints between local slots and the stack, `imul` and `iadd` work on the top two stack values, and `if_icmpgt` and `goto` form the loop. String concatenation in `label`, however, compiles to a single `invokedynamic` instruction.

### Deep & Tricky

**DT-2. Since Java 9, `+` on strings compiles to `invokedynamic`, and the JVM generates the concatenation code at run time.** *(Implementation behaviour: JEP 280, `StringConcatFactory`)* Before Java 9, `javac` emitted a `StringBuilder` chain. Now it emits one `invokedynamic` call with a "recipe" (`\u0001=\u0001`), and on the first execution the JVM's `StringConcatFactory` builds an optimised method for exactly that shape — sizing the result exactly and avoiding intermediate copies. The first call is slower (the bootstrap runs once); later calls are faster than the old `StringBuilder` code. `invokedynamic` is also how lambdas are compiled (Chapter 7). This is why "always use `StringBuilder` instead of `+`" is outdated advice — except inside loops, where each iteration would still create a new string.

## Unit 0.4 — Class loading and initialisation

**One-line summary:** Classes are loaded **lazily**, the first time they are needed, by a hierarchy of **class loaders**: the *bootstrap* loader (core `java.*` classes), the *platform* loader, and the *application* loader (your classpath), each delegating to its parent first. A class is **initialised** — its static fields set and `static { }` blocks run — on its first *active use*: creating an instance, calling a static method, or reading a non-constant static field.

```java
// file: Lazy.java
public class Lazy {
    static class Config {
        static final int CONSTANT = 42;                       // a compile-time constant: inlined by javac
        static final long LOADED_AT;
        static {
            LOADED_AT = System.nanoTime();
            System.out.println("  [Config initialised]");
        }
        static String describe() { return "config loaded at " + LOADED_AT; }
    }

    static class Unused {
        static { System.out.println("  [Unused initialised]"); }   // never printed
    }

    public static void main(String[] args) {
        System.out.println("main starts");
        System.out.println("reading a constant: " + Config.CONSTANT);   // does NOT initialise Config
        System.out.println("calling a static method:");
        System.out.println(Config.describe().startsWith("config") ? "  ok" : "  ?");
        System.out.println("Config's loader: " + Config.class.getClassLoader().getName()
                + ", String's loader: " + String.class.getClassLoader());
    }
}
```

```console
$ javac Lazy.java && java Lazy
main starts
reading a constant: 42
calling a static method:
  [Config initialised]
  ok
Config's loader: app, String's loader: null
$ java -Xlog:class+load:stdout Lazy | grep -E '\] Lazy' | sed -E 's/^\[[^]]*\]\[[^]]*\]\[[^]]*\] //'
Lazy source: file:
Lazy$Config source: file:
```

The class-loading log shows `Lazy` and `Lazy$Config` being loaded — and `Lazy$Unused` never: it was never used, so it was never even read from disk. `String`'s class loader prints as `null`, which is how Java represents the bootstrap loader.

### Deep & Tricky

**DT-3. Reading a `static final` compile-time constant does not initialise its class, because `javac` copies the value into the caller.** *(Guarantee: Java Language Specification §12.4.1 and §13.1)* `Config.CONSTANT` was compiled into `Lazy` as the literal 42; at run time `Config` is not touched. Two consequences: static initialisation order surprises (a `static { }` block you expected to run did not), and **binary compatibility** — if `Config.CONSTANT` changes to 43 and only `Config` is recompiled, `Lazy` keeps printing 42 until it, too, is recompiled. Constants that may change between releases should be accessed through a method or made non-constant (e.g. initialised from a method call).

<!-- expanded -->
### Going deeper: the `ClassNotFoundException` / `NoClassDefFoundError` difference

Classes are loaded the first time they are needed, from the **class path** (`-cp`). Two errors mean "could not find a class", at different times. `ClassNotFoundException` is thrown when code *asks by name* (`Class.forName("x.Y")`) and the class is missing — a checked exception you can catch. `NoClassDefFoundError` means a class that *was present when the code was compiled* is missing at run time — usually a JAR left off the class path.

```java
// file: cp/app/Main.java
package app;

import lib.Helper;

public class Main {
    public static void main(String[] args) throws Exception {
        try {
            Class.forName("lib.Missing");
        } catch (ClassNotFoundException e) {
            System.out.println("ClassNotFoundException: " + e.getMessage());
        }
        System.out.println(Helper.greet("class path"));
    }
}
```

```java
// file: cp/lib/Helper.java
package lib;

public class Helper {
    public static String greet(String who) { return "hello from " + who; }
}
```

```console
$ cd cp && javac -d out app/Main.java lib/Helper.java && java -cp out app.Main
ClassNotFoundException: lib.Missing
hello from class path
$ cd cp && rm -f out/lib/Helper.class && java -cp out app.Main 2>&1 | grep -E "ClassNotFound|NoClassDefFound|Caused" | head -3
ClassNotFoundException: lib.Missing
Exception in thread "main" java.lang.NoClassDefFoundError: lib/Helper
Caused by: java.lang.ClassNotFoundException: lib.Helper
```

The second run compiled fine yesterday and fails today because one `.class` file disappeared: the classic symptom of a missing dependency on a server.

## Unit 0.5 — The JIT compiler and warm-up

**One-line summary:** HotSpot (OpenJDK's JVM) starts by **interpreting** bytecode, counts how often each method runs and each loop iterates, and compiles hot methods with the fast **C1** compiler, then the most frequent ones with the optimising **C2** compiler, using run-time profile data (which branches are taken, which types appear). This **tiered compilation** gives quick start-up and high peak speed — and means the first iterations of any code are much slower than later ones.

```java
// file: Warmup.java
public class Warmup {
    static long work(int n) {
        long h = 0;
        for (int i = 0; i < n; i++) {
            h = h * 31 + (i ^ (h >>> 7));
        }
        return h;
    }

    public static void main(String[] args) {
        long sink = 0;
        for (int round = 1; round <= 8; round++) {
            long t = System.nanoTime();
            for (int k = 0; k < 400; k++) {
                sink += work(5_000);                          // 400 calls of a small method per round
            }
            System.out.printf("round %d: %6.2f ms%n", round, (System.nanoTime() - t) / 1e6);
        }
        System.out.println("(checksum " + sink + ")");
    }
}
```

```console
$ javac Warmup.java && java Warmup
round 1:   7.75 ms
round 2:   1.98 ms
round 3:   1.88 ms
round 4:   2.16 ms
round 5:   1.82 ms
round 6:   1.89 ms
round 7:   1.82 ms
round 8:   2.08 ms
(checksum -5347466402619837696)
$ java -Xint Warmup | sed -n '1p;8p'
round 1:  29.02 ms
round 8:  26.23 ms
$ java -XX:+PrintCompilation Warmup | grep -E 'Warmup::work' | tr -s ' ' | head -n 6
32 5 % 3 Warmup::work @ 4 (31 bytes)
32 6 3 Warmup::work (31 bytes)
33 7 % 4 Warmup::work @ 4 (31 bytes)
34 5 % 3 Warmup::work @ 4 (31 bytes) made not entrant
35 8 4 Warmup::work (31 bytes)
36 6 3 Warmup::work (31 bytes) made not entrant
```

The first round is slower: `work` starts in the interpreter, and the JVM compiles it (first with C1, then C2) while round 1 is still running. From round 2 on, every call runs compiled code. With `-Xint` (interpreter only) every round stays many times slower than the compiled ones. `-XX:+PrintCompilation` shows `work` compiled at tier 3 (C1 with profiling) and then tier 4 (C2); entries marked `%` are **on-stack replacement** — swapping a loop that is *already running* from interpreted to compiled code mid-execution. (Each line gives the time in milliseconds since start-up, a compilation id, flags, the tier, and the method.) When the tier-4 version is ready, the tier-3 versions are **made not entrant**: new calls go to the better code. Benchmarks that ignore warm-up measure the interpreter and the compiler at work; this is why Java benchmarks use JMH, which warms up before measuring (Chapter 11).

## Unit 0.6 — Packages, JARs, and the module path

**One-line summary:** Classes live in **packages** that mirror directories (`com.example.app.Main` in `com/example/app/Main.java`). A **JAR** is a zip of class files plus a manifest; `java -jar app.jar` runs the class named in the manifest's `Main-Class`. Since Java 9 the JDK itself is split into **modules** (`java.base`, `java.net.http`, …); applications may declare modules too (`module-info.java`), and `jdeps` reports which modules and packages a JAR depends on.

### Pattern

**P-1. Pin the Java version, compile with `--release`, and treat warnings as errors from the start.** *(Practice)* Declare the target version once (the `release` setting in Gradle/Maven, or `javac --release 21`), so builds are reproducible on any newer JDK and never silently use an API the target lacks (DT-1). Compile with `-Xlint:all -Werror` (unchecked casts, deprecation, missing `serialVersionUID`, fall-through, and more) so problems are fixed while they are cheap. *Trade-off:* `-Xlint:all` reports a few warnings teams sometimes accept (e.g. `serial`); disable those specific categories rather than the whole check.

<!-- expanded -->
### A modular application, built by hand

A **module** (Java 9+) is a JAR with a `module-info.java` that names what it `requires` and which packages it `exports`. Packages that are not exported cannot be used from other modules even if their classes are `public` — encapsulation at the level of libraries, checked by both the compiler and the JVM.

```java
// file: mods/greeter/module-info.java
module greeter {
    exports com.example.greeter;          // the API
    // com.example.greeter.internal is NOT exported
}
```

```java
// file: mods/greeter/com/example/greeter/Greeter.java
package com.example.greeter;

import com.example.greeter.internal.Format;

public class Greeter {
    public static String greet(String name) { return Format.decorate("Hello, " + name); }
}
```

```java
// file: mods/greeter/com/example/greeter/internal/Format.java
package com.example.greeter.internal;

public class Format {
    public static String decorate(String s) { return "*** " + s + " ***"; }
}
```

```java
// file: mods/app/module-info.java
module app {
    requires greeter;
}
```

```java
// file: mods/app/com/example/app/Main.java
package com.example.app;

import com.example.greeter.Greeter;

public class Main {
    public static void main(String[] args) {
        System.out.println(Greeter.greet("modules"));
        // com.example.greeter.internal.Format.decorate("x");   // error: package is not exported
    }
}
```

```console
$ cd mods && javac -d out --module-source-path . -m greeter,app && java -p out -m app/com.example.app.Main
*** Hello, modules ***
$ cd mods && rm -rf bad && mkdir -p bad && cp -r greeter app bad/ && sed -i 's|// com.example|com.example|' bad/app/com/example/app/Main.java && javac -d bad/out --module-source-path bad -m greeter,app 2>&1 | grep -m1 -oE "error: .*"
error: package com.example.greeter.internal is not visible
```

The `public` class `Format` cannot be reached from `app`: the compiler reports that the package is not visible, and its note explains that `greeter` does not export it. Most applications still run on the class path, where none of this is enforced; libraries and the JDK itself use modules — which is why reflection into JDK internals fails with `InaccessibleObjectException` since Java 16.

### Mastery check

1. **Predict.** In `Lazy`, if `CONSTANT` were declared `static final Integer CONSTANT = 42;` (boxed), would reading it initialise `Config`?
2. **Predict.** A class compiled with `javac --release 21` is run on a Java 17 JVM. What happens?

## Chapter 0 Project — Building and inspecting an application JAR

**Goal:** build a small two-package application into an executable JAR by hand — compile into a separate output directory, write a manifest, package, run — and then inspect what was built: the JAR's contents, the bytecode of a method, the class file version, the module dependencies, and the start-up cost of the JVM.

```java
// file: app/src/com/example/text/WordStats.java
package com.example.text;

import java.util.Map;
import java.util.TreeMap;

/** Counts words in text. */
public final class WordStats {
    private WordStats() {}

    public static Map<String, Integer> count(String text) {
        Map<String, Integer> counts = new TreeMap<>();
        for (String w : text.toLowerCase().split("[^a-z']+")) {
            if (!w.isEmpty()) {
                counts.merge(w, 1, Integer::sum);
            }
        }
        return counts;
    }
}
```

```java
// file: app/src/com/example/app/Main.java
package com.example.app;

import com.example.text.WordStats;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Map;

public class Main {
    public static void main(String[] args) throws IOException {
        String text = args.length > 0 ? Files.readString(Path.of(args[0])) : "to be or not to be";
        Map<String, Integer> counts = WordStats.count(text);
        counts.entrySet().stream()
              .sorted(Map.Entry.<String, Integer>comparingByValue().reversed().thenComparing(Map.Entry.comparingByKey()))
              .limit(5)
              .forEach(e -> System.out.printf("%-6s %d%n", e.getKey(), e.getValue()));
    }
}
```

```text
<!-- file: app/manifest.txt -->
Main-Class: com.example.app.Main
```

```console
$ cd app && rm -rf out && javac --release 21 -Xlint:all -Werror -d out $(find src -name '*.java') && find out -name '*.class' | sort
out/com/example/app/Main.class
out/com/example/text/WordStats.class
$ cd app && jar --create --file app.jar --manifest manifest.txt -C out . && jar --list --file app.jar && java -jar app.jar
META-INF/
META-INF/MANIFEST.MF
com/
com/example/
com/example/app/
com/example/app/Main.class
com/example/text/
com/example/text/WordStats.class
be     2
to     2
not    1
or     1
$ cd app && javap -cp app.jar -c com.example.text.WordStats | sed -n '/count(java.lang.String)/,/^$/p' | grep -E 'invoke|new ' 
       0: new           #7                  // class java/util/TreeMap
       4: invokespecial #9                  // Method java/util/TreeMap."<init>":()V
       9: invokevirtual #10                 // Method java/lang/String.toLowerCase:()Ljava/lang/String;
      14: invokevirtual #18                 // Method java/lang/String.split:(Ljava/lang/String;)[Ljava/lang/String;
      38: invokevirtual #22                 // Method java/lang/String.isEmpty:()Z
      48: invokestatic  #26                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      51: invokedynamic #32,  0             // InvokeDynamic #0:apply:()Ljava/util/function/BiFunction;
      56: invokeinterface #36,  4           // InterfaceMethod java/util/Map.merge:(Ljava/lang/Object;Ljava/lang/Object;Ljava/util/function/BiFunction;)Ljava/lang/Object;
$ cd app && jdeps -s app.jar && jdeps -verbose:package app.jar | grep -E '^ +com' | sort -u
app.jar -> java.base
   com.example.app                                    -> com.example.text                                   app.jar
   com.example.app                                    -> java.io                                            java.base
   com.example.app                                    -> java.lang                                          java.base
   com.example.app                                    -> java.lang.invoke                                   java.base
   com.example.app                                    -> java.nio.file                                      java.base
   com.example.app                                    -> java.util                                          java.base
   com.example.app                                    -> java.util.function                                 java.base
   com.example.app                                    -> java.util.stream                                   java.base
   com.example.text                                   -> java.lang                                          java.base
   com.example.text                                   -> java.lang.invoke                                   java.base
   com.example.text                                   -> java.util                                          java.base
   com.example.text                                   -> java.util.function                                 java.base
$ cd app && TIMEFORMAT='%R s wall-clock'; for i in 1 2 3; do time java -jar app.jar > /dev/null; done 2>&1 && java -Xlog:class+load -jar app.jar | grep -c 'class,load' 
0.079 s wall-clock
0.082 s wall-clock
0.069 s wall-clock
868
```

What the inspection shows:

- **The layout:** `javac -d out` placed each class in a directory matching its package. The JAR is just those files plus `META-INF/MANIFEST.MF`, whose `Main-Class` makes `java -jar` work.
- **The bytecode of `count`:** method calls appear as `invokevirtual` (instance methods, dispatched on the object's class), `invokestatic`, `invokeinterface` (calls through the `Map` interface), and an `invokedynamic` — the method reference `Integer::sum`, compiled like a lambda.
- **The dependencies:** `jdeps` reports that the application needs only `java.base`. A JAR that depends on few modules can be shipped with a custom minimal runtime built by `jlink`.
- **Start-up cost:** each run starts a fresh JVM — loading hundreds of JDK classes (the last command counts them), interpreting, then compiling — which for a small program dominates its run time. Long-running servers amortise this; short command-line tools are where Java's start-up shows, and where Class Data Sharing archives (`-XX:SharedArchiveFile`) or ahead-of-time compilation (GraalVM Native Image) help.

### Review checklist

- Why must `WordStats.java` be in `src/com/example/text/`? What happens if the directory and the package declaration disagree?
- What does `java -cp app.jar com.example.app.Main` do differently from `java -jar app.jar`?
- Which `-Xlint` warnings would appear if `WordStats` had a public constructor but no instance methods? (None — but should it be instantiable? The private constructor documents that it is not.)

## Answers to the Chapter 0 prediction questions

**0.6 Q1.** Yes. Only primitive and `String` constants initialised with constant expressions are compile-time constants. A boxed `Integer` field is read with `getstatic` at run time, which initialises `Config` first, so `[Config initialised]` would print before the value.

**0.6 Q2.** The JVM refuses to load the class: `UnsupportedClassVersionError: ... class file version 65.0 ... this version of the Java Runtime only recognizes class file versions up to 61.0`. Compile with `--release 17` to run on Java 17.

# Chapter 1 — Core Language

*Goal: write correct small Java programs — primitive types and their exact arithmetic rules, strings, arrays, control flow including switch expressions, and methods — and know the places where Java's rules surprise programmers from other languages.*

**What you will be able to do after this chapter**

- Use the eight primitive types and predict overflow, division, casting, and floating-point behaviour.
- Use `String`, text blocks, `StringBuilder`, and formatting, and handle Unicode correctly.
- Use `var`, arrays, enhanced `for`, labelled loops, and `switch` expressions.
- Write methods with overloading and varargs, and explain why Java is always pass-by-value.
- Use `BigDecimal` for money.

## Topic 1A — Values

## Unit 1.1 — Primitive types and arithmetic

**One-line summary:** Java has eight **primitive** types with sizes fixed by the specification on every platform: `byte` (8 bits), `short` (16), `int` (32), `long` (64), `float` (32-bit IEEE 754), `double` (64), `char` (16-bit unsigned UTF-16 code unit), and `boolean`. Integer arithmetic **wraps** silently on overflow (two's complement, never undefined); integer division truncates toward zero; there are no unsigned types except `char`, but `Integer.toUnsignedString`, `Integer.divideUnsigned`, and friends treat bits as unsigned.

```java
// file: Primitives.java
public class Primitives {
    public static void main(String[] args) {
        int big = Integer.MAX_VALUE;
        System.out.println("MAX_VALUE + 1 = " + (big + 1));                  // wraps to MIN_VALUE
        try {
            System.out.println(Math.addExact(big, 1));
        } catch (ArithmeticException e) {
            System.out.println("Math.addExact: " + e.getMessage());
        }
        System.out.println("-7 / 2 = " + (-7 / 2) + ", -7 % 2 = " + (-7 % 2) + ", Math.floorMod(-7, 2) = " + Math.floorMod(-7, 2));
        System.out.println("-8 >> 1 = " + (-8 >> 1) + ", -8 >>> 28 = " + (-8 >>> 28));      // arithmetic vs logical shift
        System.out.println("1 << 33 = " + (1 << 33) + " (shift distance is taken mod 32)");
        System.out.println("(byte) 200 = " + (byte) 200 + ", (int) 3.99 = " + (int) 3.99 + ", (int) -3.99 = " + (int) -3.99);
        System.out.println("(int) 1e10 = " + (int) 1e10 + ", (int) Double.NaN = " + (int) Double.NaN);
        System.out.println("0.1 + 0.2 = " + (0.1 + 0.2) + ", 1.0 / 0 = " + (1.0 / 0) + ", 0.0 / 0 = " + (0.0 / 0));
        long bytes = 3_000_000_000L;                                          // L suffix: a long literal
        System.out.println("unsigned view of -1: " + Integer.toUnsignedString(-1) + ", bytes = " + bytes);
        char c = 'A';
        c += 2;                                                               // compound assignment casts back to char
        System.out.println("char arithmetic: " + c + " " + (c + 1) + " " + (char) (c + 1));
    }
}
```

```console
$ java Primitives.java
MAX_VALUE + 1 = -2147483648
Math.addExact: integer overflow
-7 / 2 = -3, -7 % 2 = -1, Math.floorMod(-7, 2) = 1
-8 >> 1 = -4, -8 >>> 28 = 15
1 << 33 = 2 (shift distance is taken mod 32)
(byte) 200 = -56, (int) 3.99 = 3, (int) -3.99 = -3
(int) 1e10 = 2147483647, (int) Double.NaN = 0
0.1 + 0.2 = 0.30000000000000004, 1.0 / 0 = Infinity, 0.0 / 0 = NaN
unsigned view of -1: 4294967295, bytes = 3000000000
char arithmetic: C 68 D
```

`c + 1` is an `int` (arithmetic promotes `byte`, `short`, and `char` to `int`), so it prints a number; `c += 2` contains a hidden cast back to `char`. Float-to-int casts are fully defined: they round toward zero and saturate, and NaN becomes 0.

### Deep & Tricky

**DT-4. Integer overflow in Java is defined — it wraps — which makes it silent rather than catastrophic.** *(Guarantee: Java Language Specification §15.18.2, §4.2.2)* Unlike C and C++, where signed overflow is undefined behaviour, Java defines it as two's-complement wrapping, so the JIT cannot "optimise away" an overflow check. But nothing reports it: `Integer.MAX_VALUE + 1` is a large negative number, and a computation like `(low + high) / 2` in binary search silently goes wrong for large arrays — a bug that lived in the JDK's own `Arrays.binarySearch` for nine years. Use `Math.addExact`/`multiplyExact`/`toIntExact` where overflow must be detected, `(low + high) >>> 1` for midpoints, and `long` when values can grow.

<!-- expanded -->
### Going deeper: integer division, `%`, and the `Math.*Exact` methods

Integer division truncates toward zero, and `%` takes the sign of the left operand, so `-7 / 2` is `-3` and `-7 % 2` is `-1` — which breaks "is it odd?" checks written as `n % 2 == 1`. `Math.floorMod` gives the always-non-negative remainder. Overflow wraps silently (DT-4); the `Math.addExact` family throws `ArithmeticException` instead.

```java
// file: IntOps.java
public class IntOps {
    public static void main(String[] args) {
        System.out.println("-7 / 2 = " + (-7 / 2) + ", -7 % 2 = " + (-7 % 2));
        System.out.println("floorDiv = " + Math.floorDiv(-7, 2) + ", floorMod = " + Math.floorMod(-7, 2));
        int n = -3;
        System.out.println("n % 2 == 1 says odd? " + (n % 2 == 1) + "; n % 2 != 0 says odd? " + (n % 2 != 0));
        System.out.println("Integer.MAX_VALUE + 1 = " + (Integer.MAX_VALUE + 1));
        try {
            Math.addExact(Integer.MAX_VALUE, 1);
        } catch (ArithmeticException e) {
            System.out.println("addExact: " + e.getMessage());
        }
        long big = 3_000_000_000L;
        System.out.println("(int) 3e9 = " + (int) big + ", Math.toIntExact throws: " + throwsOn(() -> Math.toIntExact(big)));
        System.out.println("1 / 0.0 = " + (1 / 0.0) + ", 0.0 / 0.0 = " + (0.0 / 0.0));
        System.out.println("1 / 0 throws: " + throwsOn(() -> { int zero = 0; return 1 / zero; }));
        System.out.println("Math.abs(Integer.MIN_VALUE) = " + Math.abs(Integer.MIN_VALUE));
    }
    interface IntCall { int call(); }
    static boolean throwsOn(IntCall c) {
        try { c.call(); return false; } catch (ArithmeticException e) { return true; }
    }
}
```

```console
$ java IntOps.java
-7 / 2 = -3, -7 % 2 = -1
floorDiv = -4, floorMod = 1
n % 2 == 1 says odd? false; n % 2 != 0 says odd? true
Integer.MAX_VALUE + 1 = -2147483648
addExact: integer overflow
(int) 3e9 = -1294967296, Math.toIntExact throws: true
1 / 0.0 = Infinity, 0.0 / 0.0 = NaN
1 / 0 throws: true
Math.abs(Integer.MIN_VALUE) = -2147483648
```

The last line surprises everyone once: `Math.abs(Integer.MIN_VALUE)` is negative, because +2147483648 does not fit in an `int`.

## Unit 1.2 — Boxing and the equality trap

**One-line summary:** Each primitive has a **wrapper class** (`Integer`, `Long`, `Double`, `Character`, `Boolean`, …) used where an object is required — in collections and generics. The compiler converts automatically (**autoboxing** and **unboxing**). `==` on wrapper objects compares **references**, not values; use `equals`. Unboxing `null` throws `NullPointerException`.

```java
// file: Boxing.java
import java.util.HashMap;
import java.util.Map;

public class Boxing {
    public static void main(String[] args) {
        Integer a = 127, b = 127;
        Integer c = 1000, d = 1000;
        System.out.println("127 == 127: " + (a == b) + ", 1000 == 1000: " + (c == d) + ", 1000 equals 1000: " + c.equals(d));
        Long l = 1000L;
        System.out.println("Integer(1000).equals(Long(1000)): " + c.equals(l) + ", but c == 1000L: " + (c == 1000L));

        Map<String, Integer> stock = new HashMap<>();
        stock.put("apples", 3);
        int apples = stock.get("apples");                   // unboxing: fine
        System.out.println("apples = " + apples);
        int pears = stock.get("pears");                     // get returns null: unboxing throws
        System.out.println("pears = " + pears);
    }
}
```

```console
$ java Boxing.java
127 == 127: true, 1000 == 1000: false, 1000 equals 1000: true
Integer(1000).equals(Long(1000)): false, but c == 1000L: true
apples = 3
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "java.util.Map.get(Object)" is null
	at Boxing.main(Boxing.java:17)
```

### Deep & Tricky

**DT-5. `Integer` values from −128 to 127 are cached, so `==` happens to work for small numbers and fails for large ones.** *(Guarantee: JLS §5.1.7 requires the cache for −128..127; Implementation behaviour beyond that, `-XX:AutoBoxCacheMax`)* Autoboxing calls `Integer.valueOf`, which returns a shared instance for small values. Tests with small numbers pass; production data with larger numbers fails. Mixed comparisons (`c == 1000L`) unbox and compare values, which works — adding to the confusion. Always compare wrapper objects with `equals` (or `Objects.equals` when either may be null), or unbox explicitly.

## Unit 1.3 — Strings

**One-line summary:** `String` is an **immutable** sequence of UTF-16 `char`s. Every "modifying" method returns a new string. Compare strings with `equals`, never `==`. **Text blocks** (`"""…"""`) write multi-line strings with incidental indentation removed. Build strings in loops with `StringBuilder`; format with `String.format`, `formatted`, or `printf`. A `char` is a UTF-16 code unit, so characters outside the Basic Multilingual Plane (most emoji) take **two** `char`s — a surrogate pair.

```java
// file: Strings.java
public class Strings {
    public static void main(String[] args) {
        String s1 = "hello";
        String s2 = "hel" + "lo";                         // compile-time constant: the same interned object
        String part = "hel";
        String s3 = part + "lo";                          // built at run time: a new object
        System.out.println("s1 == s2: " + (s1 == s2) + ", s1 == s3: " + (s1 == s3) + ", s1.equals(s3): " + s1.equals(s3)
                + ", s1 == s3.intern(): " + (s1 == s3.intern()));

        String json = """
                {
                  "name": "Ada",
                  "langs": ["Java", "Rust"]
                }
                """;                                      // indentation up to the closing quotes is removed
        System.out.print(json);
        System.out.println("%s has %d lines, %d chars".formatted("json", json.lines().count(), json.length()));

        String word = "naïve🦀";
        System.out.println("length() = " + word.length() + ", code points = " + word.codePointCount(0, word.length())
                + ", charAt(5) = " + (int) word.charAt(5) + " and charAt(6) = " + (int) word.charAt(6) + " (the two halves of a surrogate pair)");
        word.codePoints().forEach(cp -> System.out.print(Character.toString(cp) + "|"));
        System.out.println();
        System.out.println("reversed with StringBuilder: " + new StringBuilder(word).reverse());

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 5; i++) {
            sb.append(i).append(i < 4 ? "," : "");
        }
        System.out.println(sb + " | " + String.join("-", "a", "b", "c") + " | " + "  trim me  ".strip() + "|" + " x".repeat(3));
    }
}
```

```console
$ java Strings.java
s1 == s2: true, s1 == s3: false, s1.equals(s3): true, s1 == s3.intern(): true
{
  "name": "Ada",
  "langs": ["Java", "Rust"]
}
json has 4 lines, 49 chars
length() = 7, code points = 6, charAt(5) = 55358 and charAt(6) = 56704 (the two halves of a surrogate pair)
n|a|ï|v|e|🦀|
reversed with StringBuilder: 🦀evïan
0,1,2,3,4 | a-b-c | trim me| x x x
```

`StringBuilder.reverse` handles surrogate pairs correctly — the crab survives reversal. Code that walks a string with `charAt(i)` treats each half of the pair as a separate, meaningless character; use `codePoints()` when characters matter.

<!-- expanded -->
### Going deeper: `StringBuilder`, text blocks, and useful `String` methods

Strings are immutable, so `s += x` in a loop creates a new string each time — quadratic work for large loops. `StringBuilder` appends into one growing buffer. Java 15's **text blocks** (`"""`) write multi-line strings without `\n` and escapes, with the common indentation removed. Java 11 added methods that remove most hand-written helpers: `isBlank`, `strip`, `lines`, `repeat`.

```java
// file: Strings2.java
import java.util.stream.Collectors;

public class Strings2 {
    public static void main(String[] args) {
        long t0 = System.nanoTime();
        String s = "";
        for (int i = 0; i < 20_000; i++) s += i % 10;
        long concat = System.nanoTime() - t0;
        t0 = System.nanoTime();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 20_000; i++) sb.append(i % 10);
        long builder = System.nanoTime() - t0;
        System.out.println("same result: " + s.equals(sb.toString()) + ", StringBuilder faster: " + (builder < concat));

        String json = """
            {
              "name": "ada",
              "langs": ["java", "c"]
            }
            """;
        System.out.print(json);
        System.out.println("lines: " + json.lines().count() + ", first line stripped: " + json.lines().findFirst().get().strip());
        System.out.println("'   '.isBlank() = " + "   ".isBlank() + ", '-'.repeat(5) = " + "-".repeat(5));
        System.out.println(String.join("|", "a", "b", "c") + " " + "a,b,,c".split(",").length
                           + " " + "a,b,,c,,".split(",").length);
        System.out.println("%s has %d chars".formatted("hello", "hello".length()));
        System.out.println("hello".chars().mapToObj(c -> String.valueOf((char) c)).collect(Collectors.joining(".")));
    }
}
```

```console
$ java Strings2.java
same result: true, StringBuilder faster: true
{
  "name": "ada",
  "langs": ["java", "c"]
}
lines: 4, first line stripped: {
'   '.isBlank() = true, '-'.repeat(5) = -----
a|b|c 4 4
hello has 5 chars
h.e.l.l.o
```

Note the `split` trap at the end: `"a,b,,c,,".split(",")` drops the *trailing* empty strings (4 elements, not 6), while empty strings in the middle are kept. Use `split(",", -1)` to keep them all — important when parsing CSV, where a trailing empty field is data.

## Unit 1.4 — `var` and arrays

**One-line summary:** `var` declares a local variable whose type is inferred from its initialiser (Java 10+); the type is still static. Arrays are objects with a fixed length (`a.length`), elements initialised to zero/`false`/`null`, bounds-checked access (`ArrayIndexOutOfBoundsException`), and **reference semantics**: assigning an array copies the reference, not the contents. `java.util.Arrays` provides `toString`, `sort`, `fill`, `copyOf`, `equals`, `binarySearch`, and `stream`.

```java
// file: Arrays1.java
import java.util.Arrays;

public class Arrays1 {
    public static void main(String[] args) {
        var counts = new int[5];                          // var: inferred int[]
        var names = new String[3];                        // elements are null
        int[] primes = {2, 3, 5, 7, 11};
        int[] alias = primes;                             // same array
        int[] copy = Arrays.copyOf(primes, primes.length);
        System.out.println("primes == copy: " + (primes == copy) + ", Arrays.equals: " + Arrays.equals(primes, copy)
                + ", primes.equals(copy): " + primes.equals(copy));
        alias[0] = 99;                                    // changes primes too: same array
        System.out.println(Arrays.toString(counts) + " " + Arrays.toString(names) + " primes=" + Arrays.toString(primes) + " copy=" + Arrays.toString(copy));

        int[][] grid = new int[3][4];                     // an array of 3 arrays of 4 ints
        grid[1][2] = 7;
        int[][] jagged = {{1}, {2, 3}, {4, 5, 6}};
        System.out.println(Arrays.deepToString(grid) + " " + Arrays.deepToString(jagged));
        Integer[] boxed = {5, 3, 9, 1};
        Arrays.sort(boxed, (x, y) -> y - x);              // descending, with a comparator
        System.out.println(Arrays.toString(boxed) + ", sum of primes via stream: " + Arrays.stream(copy).sum());
        System.out.println(primes[primes.length]);        // one past the end
    }
}
```

```console
$ java Arrays1.java 2>&1 | grep -v '^\s*at '
primes == copy: false, Arrays.equals: true, primes.equals(copy): false
[0, 0, 0, 0, 0] [null, null, null] primes=[99, 3, 5, 7, 11] copy=[2, 3, 5, 7, 11]
[[0, 0, 0, 0], [0, 0, 7, 0], [0, 0, 0, 0]] [[1], [2, 3], [4, 5, 6]]
[9, 5, 3, 1], sum of primes via stream: 28
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 5
```

`primes.equals(copy)` is `false`: arrays inherit `Object.equals`, which compares references. This is a common bug in tests; use `Arrays.equals` (or `assertArrayEquals`).

<!-- expanded -->
### Going deeper: `Arrays` utilities, and arrays are objects

An array is an object on the heap: `==` compares references, `equals` is inherited from `Object` (so also compares references), and `toString` prints a type code and hash. The `java.util.Arrays` class supplies the operations you actually want, and multi-dimensional arrays are arrays of arrays — rows can have different lengths.

```java
// file: Arrays2.java
import java.util.Arrays;

public class Arrays2 {
    public static void main(String[] args) {
        int[] a = {5, 3, 9, 1};
        int[] b = {5, 3, 9, 1};
        System.out.println("a == b: " + (a == b) + ", a.equals(b): " + a.equals(b) + ", Arrays.equals: " + Arrays.equals(a, b));
        System.out.println("toString on an array starts with: " + a.toString().substring(0, 3));
        System.out.println("Arrays.toString: " + Arrays.toString(a));
        int[] copy = Arrays.copyOf(a, 6);                  // padded with zeros
        Arrays.sort(a);
        System.out.println("sorted " + Arrays.toString(a) + ", copy " + Arrays.toString(copy)
                           + ", binarySearch(9) = " + Arrays.binarySearch(a, 9));
        int[][] tri = new int[4][];
        for (int i = 0; i < tri.length; i++) {
            tri[i] = new int[i + 1];
            tri[i][0] = tri[i][i] = 1;
            for (int j = 1; j < i; j++) tri[i][j] = tri[i - 1][j - 1] + tri[i - 1][j];
        }
        System.out.println("Pascal rows: " + Arrays.deepToString(tri));
        Object[] objs = new String[1];
        try {
            objs[0] = 42;                                  // compiles: arrays are covariant
        } catch (ArrayStoreException e) {
            System.out.println("ArrayStoreException: " + e.getMessage());
        }
    }
}
```

```console
$ java Arrays2.java
a == b: false, a.equals(b): false, Arrays.equals: true
toString on an array starts with: [I@
Arrays.toString: [5, 3, 9, 1]
sorted [1, 3, 5, 9], copy [5, 3, 9, 1, 0, 0], binarySearch(9) = 3
Pascal rows: [[1], [1, 1], [1, 2, 1], [1, 3, 3, 1]]
ArrayStoreException: java.lang.Integer
```

The last example is a design flaw kept for compatibility: a `String[]` can be used as an `Object[]`, so storing an `Integer` compiles and fails at run time. Generic collections (`List<String>`) do not have this hole (Unit 5.3).

## Topic 1B — Code

## Unit 1.5 — Control flow and `switch` expressions

**One-line summary:** `if`, `while`, `do`/`while`, `for`, and the enhanced `for (T x : iterable)` work as in C, but conditions must be `boolean`. Labelled `break` and `continue` exit nested loops. The modern **`switch` expression** (Java 14) uses `case A, B ->` arrows, does not fall through, yields a value, and must be **exhaustive**; `yield` returns a value from a block arm.

```java
// file: Switches.java
public class Switches {
    enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

    static int letters(Day d) {
        return switch (d) {                               // exhaustive over the enum: no default needed
            case MON, FRI, SUN -> 6;
            case TUE -> 7;
            case THU, SAT -> 8;
            case WED -> {
                int n = "WEDNESDAY".length();
                yield n;                                  // value of a block arm
            }
        };
    }

    static String classic(int month) {
        String season;
        switch (month) {                                  // old-style statement: falls through!
            case 12: case 1: case 2: season = "winter"; break;
            case 3: case 4: case 5: season = "spring";    // missing break...
            case 6: case 7: case 8: season = "summer"; break;
            default: season = "autumn";
        }
        return season;
    }

    public static void main(String[] args) {
        for (Day d : Day.values()) {
            System.out.print(d + "=" + letters(d) + " ");
        }
        System.out.println();
        System.out.println("April is " + classic(4) + " (the fall-through bug)");
        outer:
        for (int i = 2; i < 50; i++) {
            for (int j = 2; j * j <= i; j++) {
                if (i % j == 0) continue outer;           // not prime: next i
            }
            System.out.print(i + " ");
        }
        System.out.println();
    }
}
```

```console
$ java Switches.java
MON=6 TUE=7 WED=9 THU=8 FRI=6 SAT=8 SUN=6 
April is summer (the fall-through bug)
2 3 5 7 11 13 17 19 23 29 31 37 41 43 47 
$ javac -Xlint:fallthrough -d /tmp Switches.java 2>&1 | head -n 3
Switches.java:22: warning: [fallthrough] possible fall-through into case
            case 6: case 7: case 8: season = "summer"; break;
            ^
```

The old-style `switch` statement falls through from one case to the next unless you `break`, and `-Xlint:fallthrough` warns about it. Prefer the arrow form, which never falls through and makes the compiler check that every case is handled.

<!-- expanded -->
### Deep & Tricky

**DT-6. In an old-style `switch` *statement*, cases fall through to the next one unless you write `break`. The arrow form (`case X ->`), in both statements and expressions, never falls through, and a `switch` *expression* over an enum or sealed type must be exhaustive — the compiler rejects a missing case.** *(Guarantee: JLS §14.11, §15.28)* Prefer arrows in new code; a missing `break` is one of the oldest bugs in C-family languages.

```java
// file: Fallthrough.java
public class Fallthrough {
    enum Level { LOW, MEDIUM, HIGH }

    public static void main(String[] args) {
        int alerts = 0;
        Level l = Level.MEDIUM;
        switch (l) {
            case LOW:
                alerts += 1;
            case MEDIUM:
                alerts += 10;                 // forgot break
            case HIGH:
                alerts += 100;
        }
        System.out.println("old-style switch, MEDIUM: alerts = " + alerts);

        int arrow = switch (l) {
            case LOW -> 1;
            case MEDIUM -> 10;
            case HIGH -> 100;                 // remove this line: compile error, not exhaustive
        };
        System.out.println("switch expression, MEDIUM: " + arrow);
    }
}
```

```console
$ java Fallthrough.java
old-style switch, MEDIUM: alerts = 110
switch expression, MEDIUM: 10
$ sed '/case HIGH -> 100;/d' Fallthrough.java > NotExhaustive.java && sed -i 's/class Fallthrough/class NotExhaustive/' NotExhaustive.java && javac NotExhaustive.java 2>&1 | grep -m1 -oE "error: .*"
error: the switch expression does not cover all possible input values
```

## Unit 1.6 — Methods: pass-by-value, overloading, varargs

**One-line summary:** Methods have a return type, parameters, and a body; `static` methods belong to the class, instance methods to an object. Java passes **every** argument **by value** — for object types, the value passed is a *reference*, so a method can modify the object it refers to but cannot make the caller's variable refer to a different object. **Overloading** chooses among same-named methods by parameter types at compile time; **varargs** (`int... xs`) accept any number of arguments as an array.

```java
// file: Methods.java
import java.util.ArrayList;
import java.util.List;

public class Methods {
    static void reassign(List<String> list) {
        list = new ArrayList<>(List.of("new"));          // changes the local copy of the reference only
    }

    static void mutate(List<String> list) {
        list.add("added");                                // changes the shared object
    }

    static void increment(int x) {
        x++;                                              // a copy of the int
    }

    static String describe(int x) { return "int"; }
    static String describe(long x) { return "long"; }
    static String describe(Integer x) { return "Integer"; }
    static String describe(Object x) { return "Object"; }
    static String describe(int... xs) { return "varargs[" + xs.length + "]"; }

    static double average(double first, double... rest) {   // at least one argument required
        double sum = first;
        for (double r : rest) sum += r;
        return sum / (1 + rest.length);
    }

    public static void main(String[] args) {
        List<String> items = new ArrayList<>(List.of("original"));
        reassign(items);
        mutate(items);
        int n = 5;
        increment(n);
        System.out.println(items + ", n = " + n);
        short s = 1;
        System.out.println(describe(1) + " " + describe(s) + " " + describe(1L) + " " + describe(Integer.valueOf(1))
                + " " + describe("x") + " " + describe() + " " + describe(1, 2));
        System.out.println(average(4) + " " + average(1, 2, 3, 4));
    }
}
```

```console
$ java Methods.java
[original, added], n = 5
int int long Integer Object varargs[0] varargs[2]
4.0 2.5
```

Overload resolution prefers, in order: an exact or widening primitive match (`short` → `int`), then boxing (`int` → `Integer` → `Object`), then varargs. These rules are why adding an overload to a widely used method can silently change which method existing calls select.

### Deep & Tricky

**DT-7. `double` cannot represent most decimal fractions, so money must use `BigDecimal` (or integer cents) — and `new BigDecimal(0.1)` is already wrong.** *(IEEE 754 behaviour; Library behaviour: `BigDecimal`)* `0.1 + 0.2` is `0.30000000000000004` because 0.1 has no exact binary representation. `BigDecimal` stores a decimal number exactly as an unscaled integer and a scale. But the constructor `new BigDecimal(0.1)` faithfully converts the inexact `double` — giving `0.1000000000000000055511151231257827…` — so create values from strings (`new BigDecimal("0.1")`) or with `BigDecimal.valueOf(0.1)`, which goes through `Double.toString`. Also, `BigDecimal.equals` compares scale (`2.0` is not equal to `2.00`); use `compareTo` for numeric equality.

### Mastery check

1. **Predict.** What does `System.out.println((byte) (127 + 1) + " " + (char) 65 + 1);` print?
2. **Predict.** `Integer x = 100, y = 100; Long z = 100L;` — what are `x == y`, `x.equals(z)`, and `z.equals(100L)`?

<!-- expanded -->
## Unit 1.7 — Reading input: `Scanner`, `BufferedReader`, and the console

**One-line summary:** `Scanner` reads typed tokens (`nextInt`, `nextDouble`, `next`, `nextLine`) from any text source and is convenient but slow; `BufferedReader.readLine` reads whole lines quickly; mixing `nextInt` with `nextLine` is the classic trap, because `nextInt` leaves the end-of-line behind.

Every interactive program, every coding-interview harness, and every small tool that reads a file of numbers needs input. Java offers two standard tools with different strengths.

```java
// file: ReadInput.java
import java.util.Locale;
import java.util.Scanner;

public class ReadInput {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in).useLocale(Locale.ROOT);   // '.' as the decimal point everywhere
        System.out.print("How many items? ");
        int n = in.nextInt();
        double total = 0;
        for (int i = 0; i < n; i++) {
            String name = in.next();                 // one whitespace-separated token
            double price = in.nextDouble();
            System.out.printf("%n  %s costs %.2f", name, price);
            total += price;
        }
        in.nextLine();                               // consume the rest of the current line!
        System.out.print("%nYour name? ".formatted());
        String who = in.nextLine();
        System.out.printf("%nTotal for %s: %.2f%n", who, total);
        if (!in.hasNextInt()) System.out.println("no more numbers: hasNextInt() is false");
    }
}
```

```console
$ printf '3\napple 0.50 pen 1.25\nbook 12.00\nAda Lovelace\n' | java ReadInput.java
How many items? 
  apple costs 0.50
  pen costs 1.25
  book costs 12.00
Your name? 
Total for Ada Lovelace: 13.75
no more numbers: hasNextInt() is false
```

Line by line: `nextInt` and `nextDouble` skip whitespace (spaces *and* newlines) and parse one token, so the items can be spread over lines in any way; `next` reads one token as a `String`; `useLocale(Locale.ROOT)` matters because a `Scanner` otherwise uses the computer's locale, and on a German system `nextDouble` would expect `0,50`. The explicit `in.nextLine()` after the loop is the important line: the last `nextDouble` stopped right after `12.00`, *before* the newline, and without that call the next `nextLine()` would return the empty rest of that line instead of the name.

### When `Scanner` is too slow

`Scanner` parses with regular expressions internally, which costs a lot for large inputs. For a million numbers, read lines with `BufferedReader` and split them yourself:

```java
// file: FastSum.java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;

public class FastSum {
    public static void main(String[] args) throws IOException {
        long t0 = System.nanoTime(), sum = 0;
        if (args.length > 0 && args[0].equals("scanner")) {
            Scanner sc = new Scanner(System.in);
            while (sc.hasNextLong()) sum += sc.nextLong();
        } else {
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            String line;
            while ((line = br.readLine()) != null) {
                for (String tok : line.trim().split("\\s+")) if (!tok.isEmpty()) sum += Long.parseLong(tok);
            }
        }
        long ms = (System.nanoTime() - t0) / 1_000_000;
        System.out.println("sum = " + sum + (ms < 100_000 ? "" : " (slow)"));
        System.err.println((args.length > 0 ? args[0] : "bufferedreader") + ": " + ms + " ms");
    }
}
```

```console
$ seq 1 1000000 > nums.txt && java FastSum.java < nums.txt 2>/dev/null && java FastSum.java scanner < nums.txt 2>/dev/null
sum = 500000500000
sum = 500000500000
$ for m in bufferedreader scanner; do java FastSum.java $m < nums.txt 2>&1 >/dev/null; done | awk '{t[NR]=$2} END {print (t[2] > 1.5*t[1]) ? "Scanner was clearly slower than BufferedReader" : "no clear difference on this run"}'
no clear difference on this run
```

For interactive programs and small inputs, `Scanner`'s convenience wins; for large inputs, `BufferedReader` (or reading the whole file with `Files.readAllLines` / `Files.lines`, Unit 8.2) is the tool. `System.console()` gives access to the terminal itself, including `readPassword`, which does not echo the characters — but it returns `null` when input is redirected from a file or pipe, so programs must check for that.

### Deep & Tricky

**DT-8. Closing a `Scanner` (or `BufferedReader`) that wraps `System.in` closes `System.in` itself, and it cannot be reopened: a second `new Scanner(System.in)` later in the program finds no input and throws `NoSuchElementException`. Create one `Scanner` for standard input for the whole program and do not close it — even though IDEs warn "resource leak".** *(Library behaviour: `Scanner.close()` closes its source if the source implements `Closeable`)*

```java
// file: ClosedStdin.java
import java.util.NoSuchElementException;
import java.util.Scanner;

public class ClosedStdin {
    static int readOne() {
        try (Scanner sc = new Scanner(System.in)) {   // try-with-resources closes System.in on exit
            return sc.nextInt();
        }
    }
    public static void main(String[] args) {
        System.out.println("first: " + readOne());
        try {
            System.out.println("second: " + readOne());
        } catch (NoSuchElementException e) {
            System.out.println("second read failed: " + e.getClass().getSimpleName());
        }
    }
}
```

```console
$ printf '1\n2\n' | java ClosedStdin.java
first: 1
second read failed: NoSuchElementException
```

The `2` was still waiting in the pipe, but `System.in` had been closed by the first `Scanner`. (A first `Scanner` may also have *buffered* input beyond what it parsed, so even without closing, two `Scanner`s on the same stream can lose data.)

### Mastery check

1. **Predict.** Input `5\nhello world\n`; the program does `int n = sc.nextInt(); String s = sc.nextLine();`. What is `s`? Fix it.
2. **Write.** A program that reads lines of `name,score` until end of input and prints the average score, ignoring blank lines and reporting (with the line number) any line it cannot parse.
3. **Explain.** Why should a `Scanner` that parses numbers usually be given an explicit `Locale`?

## Chapter 1 Project — A loan amortisation schedule, exactly

**Goal:** a program that prints the monthly schedule for a fixed-rate loan — payment, interest, principal, remaining balance — computed with `BigDecimal` and banker's rounding to the cent, so that the final payment brings the balance to exactly zero. Run the same calculation with `double` alongside, and report how far it drifts.

```java
// file: loan/Amortize.java
import java.math.BigDecimal;
import java.math.MathContext;
import java.math.RoundingMode;

public class Amortize {
    static final MathContext MC = new MathContext(34, RoundingMode.HALF_EVEN);   // decimal128 precision

    /** The fixed monthly payment: P * r / (1 - (1 + r)^-n), rounded to the cent. */
    static BigDecimal payment(BigDecimal principal, BigDecimal monthlyRate, int months) {
        BigDecimal onePlusR = BigDecimal.ONE.add(monthlyRate);
        BigDecimal factor = BigDecimal.ONE.subtract(BigDecimal.ONE.divide(onePlusR.pow(months, MC), MC));
        return principal.multiply(monthlyRate).divide(factor, MC).setScale(2, RoundingMode.HALF_EVEN);
    }

    public static void main(String[] args) {
        BigDecimal principal = new BigDecimal(args.length > 0 ? args[0] : "250000.00");
        BigDecimal annualRate = new BigDecimal(args.length > 1 ? args[1] : "0.0675");
        int months = args.length > 2 ? Integer.parseInt(args[2]) : 360;
        BigDecimal r = annualRate.divide(BigDecimal.valueOf(12), MC);
        BigDecimal pay = payment(principal, r, months);
        System.out.printf("loan %s at %s%% for %d months: payment %s%n", principal, annualRate.movePointRight(2), months, pay);
        System.out.printf("%5s %12s %12s %12s %14s%n", "month", "payment", "interest", "principal", "balance");

        BigDecimal balance = principal, totalInterest = BigDecimal.ZERO;
        double dBalance = principal.doubleValue(), dRate = annualRate.doubleValue() / 12, dPay = pay.doubleValue(), dInterestTotal = 0;
        for (int m = 1; m <= months; m++) {
            BigDecimal interest = balance.multiply(r).setScale(2, RoundingMode.HALF_EVEN);
            BigDecimal thisPay = (m == months) ? balance.add(interest) : pay;      // last payment clears the balance
            BigDecimal toPrincipal = thisPay.subtract(interest);
            balance = balance.subtract(toPrincipal);
            totalInterest = totalInterest.add(interest);

            double dInterest = dBalance * dRate;                                  // the same steps in double
            dInterestTotal += dInterest;
            dBalance -= ((m == months) ? dBalance + dInterest : dPay) - dInterest;

            if (m <= 3 || m > months - 3 || m % 60 == 0) {
                System.out.printf("%5d %12s %12s %12s %14s%n", m, thisPay, interest, toPrincipal, balance);
            } else if (m == 4) {
                System.out.printf("%5s%n", "...");
            }
        }
        System.out.println("final balance (BigDecimal): " + balance + ", total interest " + totalInterest);
        System.out.printf("final balance (double):     %.10f, total interest %.10f%n", dBalance, dInterestTotal);
        System.out.println("new BigDecimal(0.1) = " + new BigDecimal(0.1));
        System.out.println("BigDecimal.valueOf(0.1) = " + BigDecimal.valueOf(0.1)
                + "; 2.0 equals 2.00? " + new BigDecimal("2.0").equals(new BigDecimal("2.00"))
                + "; compareTo == 0? " + (new BigDecimal("2.0").compareTo(new BigDecimal("2.00")) == 0));
    }
}
```

```console
$ cd loan && java Amortize.java
loan 250000.00 at 6.75% for 360 months: payment 1621.50
month      payment     interest    principal        balance
    1      1621.50      1406.25       215.25      249784.75
    2      1621.50      1405.04       216.46      249568.29
    3      1621.50      1403.82       217.68      249350.61
  ...
   60      1621.50      1321.81       299.69      234688.91
  120      1621.50      1201.90       419.60      213251.66
  180      1621.50      1034.01       587.49      183237.07
  240      1621.50       798.95       822.55      141213.13
  300      1621.50       469.84      1151.66       82374.79
  358      1621.50        27.03      1594.47        3210.27
  359      1621.50        18.06      1603.44        1606.83
  360      1615.87         9.04      1606.83           0.00
final balance (BigDecimal): 0.00, total interest 333734.37
final balance (double):     0.0000000000, total interest 333734.4730724015
new BigDecimal(0.1) = 0.1000000000000000055511151231257827021181583404541015625
BigDecimal.valueOf(0.1) = 0.1; 2.0 equals 2.00? false; compareTo == 0? true
$ cd loan && java Amortize.java 18500 0.049 48 | sed -n '/^   47/,/final balance (double)/p'
   47       425.20         3.45       421.75         423.67
   48       425.40         1.73       423.67           0.00
final balance (BigDecimal): 0.00, total interest 1909.80
final balance (double):     0.0000000000, total interest 1909.8332412812
```

The `BigDecimal` schedule rounds each month's interest to the cent exactly as a bank does (with half-even "banker's" rounding to avoid a systematic bias), and the last payment is adjusted to absorb the accumulated rounding, ending at exactly `0.00`. The `double` version also ends at zero — its last step is forced to, by the same rule — but its total interest is 333,734.47 against the bank's 333,734.37: ten cents out over 30 years. Most of the difference comes from never rounding each month's interest to the cent (which is what the bank actually charges, so the `double` figure is simply not the amount anyone pays); the rest is binary representation error accumulated over 360 steps. Tiny amounts — but in accounting, reconciliation fails on any difference at all.

### Review checklist

- Why does the program pass amounts as strings to `new BigDecimal`? What would `new BigDecimal(250000.00)` give? (Exact for this value — try `0.0675`.)
- Which values need `MathContext` (unbounded precision would throw for `1/3`), and which need `setScale(2, …)`?
- How would you handle a zero interest rate? (The payment formula divides by zero; the case needs its own branch.)

## Answers to the Chapter 1 prediction questions

**1.6 Q1.** `-128 A1`. `(byte) 128` wraps to −128. Then string concatenation proceeds left to right: `"-128 " + 'A'` is `"-128 A"`, and `+ 1` appends `"1"` — the `char` cast happened before concatenation, but the `+ 1` is string concatenation, not arithmetic.

**1.6 Q2.** `true` (100 is in the cache, DT-5), `false` (`Integer.equals` requires an `Integer`; a `Long` is never equal to an `Integer`), and `true` (`100L` is boxed to a `Long` with the same value).

- **Unit 1.7, Q1:** `s` is `""` — the rest of the line after `5`. Call `sc.nextLine()` once after `nextInt()` to discard it, then read the real line (or read every line with `nextLine()` and parse it with `Integer.parseInt`).

# Chapter 2 — Classes, Objects, and Encapsulation

*Goal: design classes that protect their invariants — constructors, encapsulated fields, immutability, the `equals`/`hashCode` contract, records, enums, and nested classes — and know the rules behind the bugs that come from getting them wrong.*

**What you will be able to do after this chapter**

- Write classes with constructors, fields, and methods, and predict initialisation order.
- Encapsulate state, validate invariants, and make objects immutable with defensive copies.
- Implement `equals`, `hashCode`, and `toString` correctly, and know what breaks when you do not.
- Use records for data carriers and enums with fields and behaviour.
- Choose between static nested classes, inner classes, local classes, and anonymous classes.

## Topic 2A — Classes

## Unit 2.1 — Classes, objects, and initialisation order

**One-line summary:** A class declares **fields** (state) and **methods** (behaviour); `new` allocates an object on the heap, sets every field to its default (0, `false`, `null`), runs field initialisers and instance initialiser blocks in source order, then the constructor body. Constructors can call each other with `this(...)` (first statement only). If you write no constructor, the compiler supplies a no-argument one.

```java
// file: InitOrder.java
public class InitOrder {
    static int counter = 0;

    static class Widget {
        private final int id = ++counter;                       // 1. field initialisers, in order
        private String name;
        private int size = log("field initialiser for size", 10);

        {                                                       // 2. instance initialiser block
            log("instance initialiser block, name is " + name, 0);
        }

        Widget(String name) {                                   // 3. constructor body
            this(name, 0);                                      // delegate to the other constructor
            log("Widget(String) body", 0);
        }

        Widget(String name, int extra) {
            this.name = name;                                   // `this` disambiguates field from parameter
            this.size += extra;
            log("Widget(String, int) body, name is " + this.name, 0);
        }

        static int log(String what, int v) {
            System.out.println("  " + what);
            return v;
        }

        @Override
        public String toString() {
            return "Widget#" + id + "(" + name + ", size " + size + ")";
        }
    }

    public static void main(String[] args) {
        System.out.println("new Widget(\"knob\"):");
        Widget w = new Widget("knob");
        System.out.println(w + " and " + new Widget("dial", 5));
    }
}
```

```console
$ java InitOrder.java
new Widget("knob"):
  field initialiser for size
  instance initialiser block, name is null
  Widget(String, int) body, name is knob
  Widget(String) body
  field initialiser for size
  instance initialiser block, name is null
  Widget(String, int) body, name is dial
Widget#1(knob, size 10) and Widget#2(dial, size 15)
```

<!-- expanded -->
### Going deeper: constructors, `this(...)`, and static factories

A class can have several constructors; one can call another with `this(...)` as its first statement, so the real work lives in one place. Many modern APIs prefer **static factory methods** — `List.of`, `Integer.valueOf`, `Path.of` — because, unlike constructors, they have names, can return a cached instance or a subclass, and can refuse to create an object by returning `Optional` or throwing with a clear message.

```java
// file: Factories.java
import java.util.HashMap;
import java.util.Map;

public class Factories {
    static final class Color {
        private static final Map<String, Color> NAMED = new HashMap<>();
        final int r, g, b;

        private Color(int r, int g, int b) {                     // private: use the factories
            if ((r | g | b) >>> 8 != 0) throw new IllegalArgumentException("component out of 0..255");
            this.r = r; this.g = g; this.b = b;
        }
        private Color(int grey) { this(grey, grey, grey); }      // delegates to the main constructor

        static Color rgb(int r, int g, int b) { return new Color(r, g, b); }
        static Color grey(int level) { return new Color(level); }
        static Color hex(String s) {
            int v = Integer.parseInt(s.replace("#", ""), 16);
            return new Color(v >> 16 & 0xff, v >> 8 & 0xff, v & 0xff);
        }
        static Color named(String name) {                        // cached: one instance per name
            return NAMED.computeIfAbsent(name, n -> switch (n) {
                case "red" -> new Color(255, 0, 0);
                case "white" -> new Color(255);
                default -> throw new IllegalArgumentException("unknown colour " + n);
            });
        }
        @Override public String toString() { return "Color(%d,%d,%d)".formatted(r, g, b); }
    }

    public static void main(String[] args) {
        System.out.println(Color.rgb(10, 20, 30) + " " + Color.grey(128) + " " + Color.hex("#1e90ff"));
        System.out.println("same cached instance: " + (Color.named("red") == Color.named("red")));
        try { Color.rgb(300, 0, 0); } catch (IllegalArgumentException e) { System.out.println(e.getMessage()); }
    }
}
```

```console
$ java Factories.java
Color(10,20,30) Color(128,128,128) Color(30,144,255)
same cached instance: true
component out of 0..255
```

`(r | g | b) >>> 8 != 0` checks all three components at once: if any has a bit above the lowest eight, the OR has one too.

## Unit 2.2 — Encapsulation and invariants

**One-line summary:** Make fields `private` and expose operations, not data. The constructor establishes the class's **invariant** (the conditions that must always hold) and every method preserves it, validating arguments and throwing `IllegalArgumentException` or `IllegalStateException` otherwise. `Objects.requireNonNull` rejects `null` early, at the boundary, instead of failing later somewhere unrelated.

```java
// file: Temperature.java
import java.util.Objects;

/** A temperature reading. Invariant: kelvin >= 0, unit is not null. */
public final class Temperature {
    public enum Unit { CELSIUS, FAHRENHEIT, KELVIN }

    private final double kelvin;
    private final Unit preferred;

    private Temperature(double kelvin, Unit preferred) {
        if (Double.isNaN(kelvin) || kelvin < 0) {
            throw new IllegalArgumentException("below absolute zero: " + kelvin + " K");
        }
        this.kelvin = kelvin;
        this.preferred = Objects.requireNonNull(preferred, "unit");
    }

    public static Temperature of(double value, Unit unit) {              // a static factory with a clear name
        double k = switch (Objects.requireNonNull(unit, "unit")) {
            case KELVIN -> value;
            case CELSIUS -> value + 273.15;
            case FAHRENHEIT -> (value - 32) * 5 / 9 + 273.15;
        };
        return new Temperature(k, unit);
    }

    public double in(Unit unit) {
        return switch (unit) {
            case KELVIN -> kelvin;
            case CELSIUS -> kelvin - 273.15;
            case FAHRENHEIT -> (kelvin - 273.15) * 9 / 5 + 32;
        };
    }

    public Temperature plusKelvin(double delta) {                         // "modifiers" return new objects
        return new Temperature(kelvin + delta, preferred);
    }

    @Override
    public String toString() {
        return "%.2f %s".formatted(in(preferred), preferred);
    }

    public static void main(String[] args) {
        Temperature body = Temperature.of(98.6, Unit.FAHRENHEIT);
        System.out.println(body + " = " + "%.2f C".formatted(body.in(Unit.CELSIUS)) + ", warmer: " + body.plusKelvin(1.5));
        for (Object[] bad : new Object[][] {{-300.0, Unit.CELSIUS}, {20.0, null}}) {
            try {
                Temperature.of((Double) bad[0], (Unit) bad[1]);
            } catch (IllegalArgumentException | NullPointerException e) {
                System.out.println(e.getClass().getSimpleName() + ": " + e.getMessage());
            }
        }
    }
}
```

```console
$ java Temperature.java
98.60 FAHRENHEIT = 37.00 C, warmer: 101.30 FAHRENHEIT
IllegalArgumentException: below absolute zero: -26.850000000000023 K
NullPointerException: unit
```

The class is `final` (no subclass can break the invariant), the fields are `private final`, construction goes through one checking constructor, and "modification" returns a new object. Such an **immutable** object is automatically thread-safe and can be shared freely.

### Deep & Tricky

**DT-9. `final` makes a *reference* unchangeable, not the object it refers to — immutability needs defensive copies.** *(Guarantee: JLS §4.12.4)* A `private final List<String> tags` field still allows `tags.add(...)`, and if the constructor stored the caller's list, the caller can change the "immutable" object after construction; if a getter returns the field, every caller can. Copy mutable inputs in the constructor and return unmodifiable views or copies: `this.tags = List.copyOf(tags)` (which also rejects `null` elements) and `return tags;` is then safe, because `List.copyOf` returns an unmodifiable list. Arrays cannot be made unmodifiable at all — copy them in and out (`array.clone()`).

```java
// file: DefensiveCopy.java
import java.util.ArrayList;
import java.util.List;

public class DefensiveCopy {
    record Leaky(List<String> tags) {}                                  // stores the caller's list as-is

    record Safe(List<String> tags) {
        Safe {
            tags = List.copyOf(tags);                                   // compact constructor: copy on the way in
        }
    }

    public static void main(String[] args) {
        List<String> input = new ArrayList<>(List.of("a", "b"));
        Leaky leaky = new Leaky(input);
        Safe safe = new Safe(input);
        input.add("sneaky");                                            // the caller changes its list later
        System.out.println("leaky sees " + leaky.tags() + ", safe sees " + safe.tags());
        try {
            safe.tags().add("x");
        } catch (UnsupportedOperationException e) {
            System.out.println("safe.tags() is unmodifiable: " + e.getClass().getSimpleName());
        }
    }
}
```

```console
$ java DefensiveCopy.java
leaky sees [a, b, sneaky], safe sees [a, b]
safe.tags() is unmodifiable: UnsupportedOperationException
```

## Unit 2.3 — `equals`, `hashCode`, and `toString`

**One-line summary:** `Object.equals` compares identity; override it for **value** semantics. The contract: `equals` is reflexive, symmetric, transitive, consistent, and `x.equals(null)` is `false`; and **equal objects must have equal hash codes**. `HashMap` and `HashSet` find objects by `hashCode` first, then `equals` — so overriding one without the other, or changing a field used by either while the object is in a hash collection, breaks them silently.

```java
// file: HashContract.java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

public class HashContract {
    static class PointNoHash {                                         // equals without hashCode: broken
        final int x, y;
        PointNoHash(int x, int y) { this.x = x; this.y = y; }
        @Override public boolean equals(Object o) {
            return o instanceof PointNoHash p && p.x == x && p.y == y;
        }
    }

    static class MutablePoint {                                        // correct methods, but mutable
        int x, y;
        MutablePoint(int x, int y) { this.x = x; this.y = y; }
        @Override public boolean equals(Object o) {
            return o instanceof MutablePoint p && p.x == x && p.y == y;
        }
        @Override public int hashCode() { return Objects.hash(x, y); }
    }

    record Point(int x, int y) {}                                     // records do it right

    public static void main(String[] args) {
        Set<PointNoHash> s1 = new HashSet<>();
        s1.add(new PointNoHash(1, 2));
        System.out.println("no hashCode: contains equal point? " + s1.contains(new PointNoHash(1, 2))
                + ", equals says " + new PointNoHash(1, 2).equals(new PointNoHash(1, 2)));

        Set<MutablePoint> s2 = new HashSet<>();
        MutablePoint p = new MutablePoint(1, 2);
        s2.add(p);
        p.x = 99;                                                     // mutate a key inside the set
        System.out.println("mutated key: contains(p)? " + s2.contains(p) + ", size " + s2.size()
                + ", but iterating finds it: " + s2.iterator().next().equals(p));

        Set<Point> s3 = new HashSet<>();
        s3.add(new Point(1, 2));
        System.out.println("record: contains? " + s3.contains(new Point(1, 2)) + ", toString " + new Point(1, 2));
    }
}
```

```console
$ java HashContract.java
no hashCode: contains equal point? false, equals says true
mutated key: contains(p)? false, size 1, but iterating finds it: true
record: contains? true, toString Point[x=1, y=2]
```

Two silent failures: the set "loses" an object equal to one it contains, because the default `hashCode` (derived from identity) sends the lookup to a different bucket; and a mutated key sits in the bucket of its *old* hash, unreachable by lookup but still counted and iterated. Use immutable keys — records are ideal.

### Deep & Tricky

**DT-10. `equals` and inheritance do not mix: there is no way to add a value field in a subclass and keep `equals` symmetric and transitive.** *(Design fact; Effective Java, Item 10)* If `ColorPoint extends Point` adds a colour and compares it, then `point.equals(colorPoint)` is true while `colorPoint.equals(point)` is false. Using `getClass() != o.getClass()` instead of `instanceof` restores symmetry but breaks substitutability (a subclass instance never equals a base instance). The practical answers: make value classes `final` (records are), and prefer composition — a `ColorPoint` that *has* a `Point` — over inheritance for value types.

## Topic 2B — Kinds of classes

## Unit 2.4 — Records

**One-line summary:** A **record** (Java 16) is a final class whose state is a fixed list of **components**: `record Point(int x, int y) {}` declares private final fields, a canonical constructor, accessors `x()` and `y()`, and value-based `equals`, `hashCode`, and `toString`. A **compact constructor** validates or normalises the arguments. Records may have static fields, static and instance methods, and implement interfaces, but no other instance fields.

```java
// file: Records.java
import java.util.Comparator;
import java.util.List;

public class Records {
    record Range(int lo, int hi) implements Comparable<Range> {
        Range {                                                  // compact constructor: checks before fields are set
            if (lo > hi) throw new IllegalArgumentException("lo > hi: " + lo + " > " + hi);
        }
        static Range of(int a, int b) { return new Range(Math.min(a, b), Math.max(a, b)); }
        int length() { return hi - lo; }
        boolean overlaps(Range o) { return lo < o.hi && o.lo < hi; }
        Range span(Range o) { return new Range(Math.min(lo, o.lo), Math.max(hi, o.hi)); }
        @Override public int compareTo(Range o) {
            return Comparator.comparingInt(Range::lo).thenComparingInt(Range::hi).compare(this, o);
        }
    }

    public static void main(String[] args) {
        var ranges = List.of(Range.of(9, 5), new Range(1, 3), new Range(2, 4), new Range(8, 10));
        System.out.println(ranges.stream().sorted().toList());
        System.out.println("(1,3) overlaps (2,4): " + ranges.get(1).overlaps(ranges.get(2))
                + ", span " + ranges.get(1).span(ranges.get(2)) + ", equal to new Range(1,3)? " + ranges.get(1).equals(new Range(1, 3)));
        try {
            new Range(5, 1);
        } catch (IllegalArgumentException e) {
            System.out.println("rejected: " + e.getMessage());
        }
    }
}
```

```console
$ java Records.java
[Range[lo=1, hi=3], Range[lo=2, hi=4], Range[lo=5, hi=9], Range[lo=8, hi=10]]
(1,3) overlaps (2,4): true, span Range[lo=1, hi=4], equal to new Range(1,3)? true
rejected: lo > hi: 5 > 1
```

<!-- expanded -->
### Going deeper: compact constructors, and what records do not do

A record's **compact constructor** — `record Range(int lo, int hi) { Range { ... } }` — runs before the fields are assigned, so it can validate or normalise the parameters. Records are shallowly immutable: the fields are `final`, but a record holding a `List` holds a reference to a list that can still change, unless the constructor copies it.

```java
// file: Records2.java
import java.util.ArrayList;
import java.util.List;

public class Records2 {
    record Range(int lo, int hi) {
        Range {                                                   // compact constructor
            if (lo > hi) throw new IllegalArgumentException("lo > hi: " + lo + " > " + hi);
        }
        int length() { return hi - lo; }
        static Range of(int a, int b) { return new Range(Math.min(a, b), Math.max(a, b)); }
    }

    record Team(String name, List<String> members) {}            // shallow: shares the caller's list
    record SafeTeam(String name, List<String> members) {
        SafeTeam { members = List.copyOf(members); }              // defensive, unmodifiable copy
    }

    public static void main(String[] args) {
        System.out.println(Range.of(9, 2) + " length " + Range.of(9, 2).length());
        System.out.println("equal by value: " + new Range(1, 2).equals(new Range(1, 2)));
        try { new Range(5, 1); } catch (IllegalArgumentException e) { System.out.println(e.getMessage()); }

        List<String> people = new ArrayList<>(List.of("ada"));
        Team t = new Team("core", people);
        SafeTeam s = new SafeTeam("core", people);
        people.add("mallory");                                    // changes the list after construction
        System.out.println("Team sees " + t.members() + ", SafeTeam sees " + s.members());
        try { s.members().add("x"); } catch (UnsupportedOperationException e) { System.out.println("SafeTeam list is unmodifiable"); }
    }
}
```

```console
$ java Records2.java
Range[lo=2, hi=9] length 7
equal by value: true
lo > hi: 5 > 1
Team sees [ada, mallory], SafeTeam sees [ada]
SafeTeam list is unmodifiable
```

## Unit 2.5 — Enums

**One-line summary:** An `enum` is a class with a fixed set of instances. Enum constants can have fields, constructors, methods, and even per-constant method bodies; they are compared with `==`, used in `switch`, and have `values()`, `valueOf`, `name()`, and `ordinal()`. `EnumMap` and `EnumSet` are compact, fast collections keyed by enum constants.

```java
// file: Planets.java
import java.util.EnumMap;
import java.util.EnumSet;
import java.util.Map;

public class Planets {
    enum Planet {
        MERCURY(3.303e+23, 2.4397e6), EARTH(5.976e+24, 6.37814e6), JUPITER(1.9e+27, 7.1492e7);

        static final double G = 6.67300E-11;
        private final double mass, radius;

        Planet(double mass, double radius) {
            this.mass = mass;
            this.radius = radius;
        }
        double surfaceGravity() { return G * mass / (radius * radius); }
    }

    enum Op {
        ADD("+") { int apply(int a, int b) { return a + b; } },
        MUL("*") { int apply(int a, int b) { return a * b; } };
        final String symbol;
        Op(String symbol) { this.symbol = symbol; }
        abstract int apply(int a, int b);                     // each constant supplies its own body
    }

    public static void main(String[] args) {
        double earthWeight = 70;
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        Map<Planet, Double> weights = new EnumMap<>(Planet.class);
        for (Planet p : Planet.values()) {
            weights.put(p, mass * p.surfaceGravity());
        }
        weights.forEach((p, w) -> System.out.printf("%-8s %6.1f (ordinal %d)%n", p, w, p.ordinal()));
        for (Op op : Op.values()) {
            System.out.print("6 " + op.symbol + " 7 = " + op.apply(6, 7) + "; ");
        }
        System.out.println(EnumSet.complementOf(EnumSet.of(Planet.EARTH)) + " " + Planet.valueOf("JUPITER").name());
    }
}
```

```console
$ java Planets.java
MERCURY    26.4 (ordinal 0)
EARTH      70.0 (ordinal 1)
JUPITER   177.1 (ordinal 2)
6 + 7 = 13; 6 * 7 = 42; [MERCURY, JUPITER] JUPITER
```

<!-- expanded -->
### Going deeper: enums with behaviour, `EnumMap`, and `EnumSet`

Enums are full classes: they can have fields, constructors, methods, and even a different method body per constant. `EnumMap` and `EnumSet` are collections specialised for enum keys — internally an array and a bit set — and are faster and more compact than `HashMap`/`HashSet`.

```java
// file: Enums2.java
import java.util.EnumMap;
import java.util.EnumSet;
import java.util.Map;

public class Enums2 {
    enum Op {
        ADD("+") { long apply(long a, long b) { return a + b; } },
        SUB("-") { long apply(long a, long b) { return a - b; } },
        MUL("*") { long apply(long a, long b) { return a * b; } };

        final String symbol;
        Op(String symbol) { this.symbol = symbol; }
        abstract long apply(long a, long b);                      // each constant implements it
    }

    enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

    public static void main(String[] args) {
        for (Op op : Op.values()) System.out.println("7 " + op.symbol + " 3 = " + op.apply(7, 3));
        System.out.println("valueOf(\"MUL\") = " + Op.valueOf("MUL") + ", ordinal " + Op.MUL.ordinal());

        EnumSet<Day> weekend = EnumSet.of(Day.SAT, Day.SUN);
        EnumSet<Day> weekdays = EnumSet.complementOf(weekend);
        System.out.println("weekdays " + weekdays + ", MON..WED " + EnumSet.range(Day.MON, Day.WED));
        Map<Day, Integer> hours = new EnumMap<>(Day.class);
        hours.put(Day.FRI, 6);
        hours.put(Day.MON, 8);
        System.out.println("EnumMap keeps declaration order: " + hours);
        try { Day.valueOf("Monday"); } catch (IllegalArgumentException e) { System.out.println(e.getMessage()); }
    }
}
```

```console
$ java Enums2.java
7 + 3 = 10
7 - 3 = 4
7 * 3 = 21
valueOf("MUL") = MUL, ordinal 2
weekdays [MON, TUE, WED, THU, FRI], MON..WED [MON, TUE, WED]
EnumMap keeps declaration order: {MON=8, FRI=6}
No enum constant Enums2.Day.Monday
```

Do not persist `ordinal()` (to a database or file): inserting a constant in the middle of the enum renumbers every later one. Store `name()` instead.

## Unit 2.6 — Nested, inner, local, and anonymous classes

**One-line summary:** A **static nested class** is an ordinary class scoped inside another (use it by default). A non-static **inner class** has an implicit reference to an instance of its enclosing class, so it can use that instance's fields — and keeps it alive. **Local** classes are declared inside a method; **anonymous** classes are declared and instantiated in one expression (mostly replaced by lambdas, Chapter 7).

### Deep & Tricky

**DT-11. Every instance of a non-static inner class holds a hidden reference to its outer object — a classic source of memory leaks.** *(Guarantee: JLS §8.1.3; Implementation behaviour: the synthetic `this$0` field)* An inner-class object (including an anonymous class, or a lambda that uses `this`) stored somewhere long-lived — a listener list, a cache, a thread — keeps its entire enclosing object reachable, with everything *it* references. Declare nested classes `static` unless they genuinely need the outer instance; `javap -p` shows the hidden field.

```java
// file: Outer.java
public class Outer {
    private final byte[] bigBuffer = new byte[10_000_000];       // 10 MB owned by each Outer

    class Inner { int get() { return bigBuffer.length; } }       // holds Outer.this
    static class Nested { int get() { return 42; } }             // holds nothing

    public static void main(String[] args) {
        Inner i = new Outer().new Inner();                        // the Outer is otherwise unreachable...
        Nested n = new Nested();
        System.gc();
        System.out.println("inner still reaches the buffer: " + i.get() + " bytes; nested: " + n.get());
    }
}
```

```console
$ javac -d out Outer.java && javap -p -cp out 'Outer$Inner' 'Outer$Nested' | grep -E 'class|this'
class Outer$Inner {
  final Outer this$0;
class Outer$Nested {
```

### Pattern

**P-2. Make classes immutable unless there is a good reason not to; make value types records; make everything else `private` by default.** *(Practice; Effective Java, Items 15–17)* Immutable objects are simple (one state, set at construction), thread-safe without locks, safe to share and cache, and safe as map keys. Minimise accessibility: `private` fields, package-private helper classes, `public` only for the API you commit to. *Trade-off:* immutable objects mean allocating new objects for changes; for large, frequently modified state, use a mutable class with a well-guarded interface, or a builder that produces immutable snapshots.

<!-- expanded -->
### Going deeper: when to use which nested class

| Kind | Has an outer instance? | Typical use |
| --- | --- | --- |
| `static` nested class | No | Helper types tied to the outer class: `Map.Entry`, a builder, a node of a linked list |
| Inner class (non-static) | Yes, hidden `this$0` | An iterator that needs the collection's fields |
| Local class | Yes, if declared in an instance method | A small helper used in one method only |
| Anonymous class | Same as local | Mostly replaced by lambdas; still used when you need state or several methods |

The rule of thumb: make nested classes `static` unless they genuinely need the outer object. A `static` nested class cannot accidentally keep a large outer object alive (DT-11), and it is cheaper to create.

### Mastery check

1. **Predict.** What does `new Records.Range(1, 3).equals(new Records.Range(1, 3))` return, and what does `==` return for the same two expressions?
2. **Predict.** A class overrides `hashCode` but not `equals`. Two distinct instances with the same field values are added to a `HashSet`. What is its size?

## Chapter 2 Project — A bank account model with enforced invariants

**Goal:** a small banking domain — money, account identifiers, transactions, accounts — designed so that invalid states are hard or impossible: money is an immutable value with a currency, amounts are exact cents, account balances never go negative, transaction history is append-only and cannot be modified from outside, and every identifier is validated. A driver exercises valid operations and each kind of invalid one.

```java
// file: bank/Money.java
import java.util.Objects;

/** An exact amount of money in minor units (cents), with an ISO currency code. */
public record Money(long cents, String currency) implements Comparable<Money> {
    public Money {
        Objects.requireNonNull(currency, "currency");
        if (!currency.matches("[A-Z]{3}")) throw new IllegalArgumentException("bad currency code: " + currency);
    }

    public static Money of(String amount, String currency) {          // "12.34" -> 1234 cents, exactly
        var bd = new java.math.BigDecimal(amount).movePointRight(2);
        return new Money(bd.longValueExact(), currency);               // throws if it has fractions of a cent
    }

    public Money plus(Money o) { requireSame(o); return new Money(Math.addExact(cents, o.cents), currency); }
    public Money minus(Money o) { requireSame(o); return new Money(Math.subtractExact(cents, o.cents), currency); }
    public boolean isNegative() { return cents < 0; }

    private void requireSame(Money o) {
        if (!currency.equals(o.currency)) throw new IllegalArgumentException("currency mismatch: " + currency + " vs " + o.currency);
    }

    @Override public int compareTo(Money o) { requireSame(o); return Long.compare(cents, o.cents); }
    @Override public String toString() {                              // sign handled separately: -50 cents is "-0.50"
        return "%s %s%d.%02d".formatted(currency, cents < 0 ? "-" : "", Math.abs(cents / 100), Math.abs(cents % 100));
    }
}
```

```java
// file: bank/AccountId.java
/** An account identifier: "AC-" followed by 6 digits, with a mod-97 check. */
public record AccountId(String value) {
    public AccountId {
        if (value == null || !value.matches("AC-\\d{6}")) throw new IllegalArgumentException("malformed account id: " + value);
        if (Integer.parseInt(value.substring(3)) % 97 != 1) throw new IllegalArgumentException("check digit failed: " + value);
    }
}
```

```java
// file: bank/Transaction.java
import java.time.Instant;

/** One immutable entry in an account's history. */
public record Transaction(long sequence, Instant at, Kind kind, Money amount, Money balanceAfter, String memo) {
    public enum Kind { DEPOSIT, WITHDRAWAL, TRANSFER_IN, TRANSFER_OUT }
}
```

```java
// file: bank/Account.java
import java.time.Clock;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

/** An account. Invariants: balance >= 0, balance == sum of history, history is append-only. */
public final class Account {
    private final AccountId id;
    private final String owner;
    private final Clock clock;                                   // injected, so tests can control time
    private Money balance;
    private final List<Transaction> history = new ArrayList<>();

    public Account(AccountId id, String owner, String currency, Clock clock) {
        this.id = Objects.requireNonNull(id);
        this.owner = Objects.requireNonNull(owner).strip();
        if (this.owner.isEmpty()) throw new IllegalArgumentException("owner must not be blank");
        this.clock = Objects.requireNonNull(clock);
        this.balance = new Money(0, currency);
    }

    public AccountId id() { return id; }
    public Money balance() { return balance; }                   // Money is immutable: safe to return
    public List<Transaction> history() { return Collections.unmodifiableList(history); }   // read-only view

    public void deposit(Money amount, String memo) { apply(Transaction.Kind.DEPOSIT, amount, memo); }

    public void withdraw(Money amount, String memo) { apply(Transaction.Kind.WITHDRAWAL, amount, memo); }

    /** Moves money between accounts; either both sides happen or neither. */
    public void transferTo(Account other, Money amount) {
        if (other == this) throw new IllegalArgumentException("cannot transfer to the same account");
        Money after = checkedAfter(Transaction.Kind.TRANSFER_OUT, amount);        // validate everything first...
        other.checkedAfter(Transaction.Kind.TRANSFER_IN, amount);
        record(Transaction.Kind.TRANSFER_OUT, amount, after, "to " + other.id.value());   // ...then change state
        other.record(Transaction.Kind.TRANSFER_IN, amount, other.balance.plus(amount), "from " + id.value());
    }

    private void apply(Transaction.Kind kind, Money amount, String memo) {
        record(kind, amount, checkedAfter(kind, amount), memo);
    }

    private Money checkedAfter(Transaction.Kind kind, Money amount) {
        Objects.requireNonNull(amount, "amount");
        if (amount.cents() <= 0) throw new IllegalArgumentException("amount must be positive: " + amount);
        boolean outgoing = kind == Transaction.Kind.WITHDRAWAL || kind == Transaction.Kind.TRANSFER_OUT;
        Money after = outgoing ? balance.minus(amount) : balance.plus(amount);   // also checks the currency
        if (after.isNegative()) throw new IllegalStateException("insufficient funds in " + id.value() + ": balance " + balance + ", requested " + amount);
        return after;
    }

    private void record(Transaction.Kind kind, Money amount, Money after, String memo) {
        balance = after;
        history.add(new Transaction(history.size() + 1, clock.instant(), kind, amount, after, memo));
    }

    @Override public String toString() { return id.value() + " (" + owner + "): " + balance; }
}
```

```java
// file: bank/Main.java
import java.time.Clock;
import java.time.Instant;
import java.time.ZoneOffset;

public class Main {
    static void attempt(String what, Runnable action) {
        try {
            action.run();
            System.out.println("ok:       " + what);
        } catch (RuntimeException e) {
            System.out.println("rejected: " + what + " -> " + e.getClass().getSimpleName() + ": " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        Clock fixed = Clock.fixed(Instant.parse("2026-11-15T09:00:00Z"), ZoneOffset.UTC);
        Account alice = new Account(new AccountId("AC-000001"), "Alice", "EUR", fixed);
        Account bob = new Account(new AccountId("AC-000098"), "Bob", "EUR", fixed);
        alice.deposit(Money.of("1000.00", "EUR"), "salary");
        alice.withdraw(Money.of("120.50", "EUR"), "rent share");
        alice.transferTo(bob, Money.of("300", "EUR"));

        attempt("withdraw more than the balance", () -> bob.withdraw(Money.of("300.01", "EUR"), "too much"));
        attempt("deposit a fraction of a cent", () -> alice.deposit(Money.of("0.005", "EUR"), "dust"));
        attempt("deposit in the wrong currency", () -> alice.deposit(Money.of("5", "USD"), "dollars"));
        attempt("deposit a negative amount", () -> alice.deposit(new Money(-500, "EUR"), "sneaky"));
        attempt("transfer exceeding funds", () -> bob.transferTo(alice, Money.of("1000", "EUR")));
        attempt("open account with bad check digit", () -> new AccountId("AC-000002"));
        attempt("modify history from outside", () -> alice.history().clear());

        System.out.println(alice + "\n" + bob);
        alice.history().forEach(t -> System.out.printf("  #%d %-12s %12s -> %12s  %s%n", t.sequence(), t.kind(), t.amount(), t.balanceAfter(), t.memo()));
        System.out.println("bob's failed transfer left both balances unchanged: " + bob.balance() + ", " + alice.balance());
    }
}
```

```console
$ cd bank && rm -rf out && javac -Xlint:all -d out *.java && java -cp out Main
rejected: withdraw more than the balance -> IllegalStateException: insufficient funds in AC-000098: balance EUR 300.00, requested EUR 300.01
rejected: deposit a fraction of a cent -> ArithmeticException: Rounding necessary
rejected: deposit in the wrong currency -> IllegalArgumentException: currency mismatch: EUR vs USD
rejected: deposit a negative amount -> IllegalArgumentException: amount must be positive: EUR -5.00
rejected: transfer exceeding funds -> IllegalStateException: insufficient funds in AC-000098: balance EUR 300.00, requested EUR 1000.00
rejected: open account with bad check digit -> IllegalArgumentException: check digit failed: AC-000002
rejected: modify history from outside -> UnsupportedOperationException: null
AC-000001 (Alice): EUR 579.50
AC-000098 (Bob): EUR 300.00
  #1 DEPOSIT       EUR 1000.00 ->  EUR 1000.00  salary
  #2 WITHDRAWAL     EUR 120.50 ->   EUR 879.50  rent share
  #3 TRANSFER_OUT   EUR 300.00 ->   EUR 579.50  to AC-000098
bob's failed transfer left both balances unchanged: EUR 300.00, EUR 579.50
```

The design applies every unit of the chapter:

- **Value objects are records**: `Money`, `AccountId`, and `Transaction` are immutable, compare by value, and validate in compact constructors, so an invalid `AccountId` or a lowercase currency code cannot exist.
- **Exact money**: `Money` stores whole cents in a `long`, parses decimal strings with `BigDecimal`, refuses fractions of a cent (`longValueExact` throws), and uses `Math.addExact` so even overflow cannot pass silently.
- **The entity is a class with private state**: `Account` exposes operations, never its mutable list; `history()` returns an unmodifiable view, so the attempt to `clear()` it failed.
- **Validate, then mutate**: `transferTo` checks both sides before changing either, so the failed transfer left both balances untouched — a small taste of the "all or nothing" property that Chapter 4's exception safety and Chapter 12's database transactions formalise.
- **Injected clock**: `Clock.fixed` makes timestamps deterministic, so the program (and its tests) produce the same output every time.

### Review checklist

- Which invariant would break if `history()` returned `history` directly? Which if `Money` had a public setter?
- `Account` does not override `equals`. Should two accounts with the same id be equal? What does that depend on? (Entities are usually compared by identity or id, values by content.)
- Is `Account` thread-safe? (No: two threads withdrawing concurrently can both pass the balance check. Chapter 10 fixes this.)

## Answers to the Chapter 2 prediction questions

**2.6 Q1.** `equals` returns `true` (records compare components); `==` returns `false` (two different objects).

**2.6 Q2.** 2. The equal hash codes put both in the same bucket, but the default `equals` (identity) says they differ, so both are kept. Both methods must be overridden together.

# Chapter 3 — Inheritance, Interfaces, Sealed Types, and Pattern Matching

*Goal: use Java's two models of polymorphism — open class hierarchies with virtual dispatch, and closed hierarchies of sealed interfaces and records taken apart with pattern matching — and know which to choose.*

**What you will be able to do after this chapter**

- Extend classes, override methods correctly, and chain constructors.
- Explain dynamic dispatch, and what is *not* polymorphic: fields and static methods.
- Choose between abstract classes and interfaces with default methods.
- Model closed sets of alternatives with sealed interfaces and records.
- Use pattern matching for `instanceof` and `switch`, record patterns, and guards, with compiler-checked exhaustiveness.
- Recognise the fragile-base-class problem and prefer composition.

## Topic 3A — Open hierarchies

## Unit 3.1 — Inheritance and overriding

**One-line summary:** `class B extends A` inherits A's fields and methods; B can **override** instance methods (same signature; return type may be a subtype — covariant; access may widen, never narrow; checked exceptions may only narrow). `@Override` makes the compiler check that you really override. `super.m()` calls the parent's version; every constructor first calls a superclass constructor (`super(...)`, implicitly `super()` if omitted). `final` methods cannot be overridden; `final` classes cannot be extended.

```java
// file: Shapes.java
import java.util.List;

public class Shapes {
    abstract static class Shape {
        private final String name;
        Shape(String name) { this.name = name; }
        abstract double area();                                   // subclasses must implement
        String describe() { return "%s with area %.2f".formatted(name, area()); }   // dispatches to the subclass
    }

    static class Rect extends Shape {
        final double w, h;
        Rect(double w, double h) { this("rectangle", w, h); }
        protected Rect(String name, double w, double h) { super(name); this.w = w; this.h = h; }
        @Override double area() { return w * h; }
    }

    static final class Square extends Rect {
        Square(double side) { super("square", side, side); }
        @Override String describe() { return super.describe() + " (side " + w + ")"; }
    }

    static class Circle extends Shape {
        final double r;
        Circle(double r) { super("circle"); this.r = r; }
        @Override double area() { return Math.PI * r * r; }
    }

    public static void main(String[] args) {
        List<Shape> shapes = List.of(new Rect(2, 3), new Square(2), new Circle(1));
        for (Shape s : shapes) {
            System.out.println(s.describe() + "  [" + s.getClass().getSimpleName() + "]");
        }
    }
}
```

```console
$ java Shapes.java
rectangle with area 6.00  [Rect]
square with area 4.00 (side 2.0)  [Square]
circle with area 3.14  [Circle]
```

### Deep & Tricky

**DT-12. Calling an overridable method from a constructor runs the subclass's override before the subclass's fields are initialised.** *(Guarantee: JLS §12.5)* The superclass constructor runs first; if it calls a method that a subclass overrides, the override runs while the subclass's fields still hold their defaults — `null`, 0 — even `final` ones. The result is a `NullPointerException` or wrong value in code that looks correct. Constructors should call only `private`, `static`, or `final` methods. (A curious detail from writing this example: with `label = "derived"` the trap did *not* fire, because a `final` field initialised with a constant is a compile-time constant that `javac` inlines into `describe` — the same rule as DT-3. Any non-constant initialiser, such as the `toUpperCase()` call here, exposes the bug.)

```java
// file: CtorTrap.java
public class CtorTrap {
    static class Base {
        Base() { System.out.println("Base constructor sees: " + describe()); }
        String describe() { return "base"; }
    }

    static class Derived extends Base {
        private final String label = "derived".toUpperCase();   // not yet assigned when Base() runs
        Derived() { super(); System.out.println("Derived constructor sees: " + describe()); }
        @Override String describe() { return "label=" + label + " (length " + (label == null ? "NPE!" : label.length()) + ")"; }
    }

    public static void main(String[] args) {
        new Derived();
    }
}
```

```console
$ java CtorTrap.java
Base constructor sees: label=null (length NPE!)
Derived constructor sees: label=DERIVED (length 7)
```

## Unit 3.2 — Dynamic dispatch, and what is not polymorphic

**One-line summary:** Instance method calls are dispatched on the object's **run-time class** (virtual dispatch). Three things are *not*: **fields** (accessed by the variable's compile-time type), **static methods** (they are hidden, not overridden: chosen by compile-time type), and **overload selection** (chosen at compile time from the argument's static type).

```java
// file: Dispatch.java
public class Dispatch {
    static class Animal {
        String name = "animal";
        static String kind() { return "Animal.kind"; }
        String speak() { return "..."; }
    }

    static class Dog extends Animal {
        String name = "dog";                                   // hides Animal.name: a second field
        static String kind() { return "Dog.kind"; }            // hides, does not override
        @Override String speak() { return "woof"; }
    }

    static String meet(Animal a) { return "meet(Animal)"; }
    static String meet(Dog d) { return "meet(Dog)"; }

    public static void main(String[] args) {
        Animal a = new Dog();                                  // static type Animal, run-time class Dog
        System.out.println("a.speak() = " + a.speak() + "   (virtual: the Dog's method)");
        System.out.println("a.name = " + a.name + "   (field: chosen by static type)");
        System.out.println("((Dog) a).name = " + ((Dog) a).name);
        System.out.println("meet(a) = " + meet(a) + "   (overload: chosen by static type at compile time)");
        System.out.println("Dog.kind() = " + Dog.kind() + ", Animal.kind() = " + Animal.kind());
    }
}
```

```console
$ java Dispatch.java
a.speak() = woof   (virtual: the Dog's method)
a.name = animal   (field: chosen by static type)
((Dog) a).name = dog
meet(a) = meet(Animal)   (overload: chosen by static type at compile time)
Dog.kind() = Dog.kind, Animal.kind() = Animal.kind
```

<!-- expanded -->
### Going deeper: static methods and fields are not overridden

Instance methods are dispatched on the object's *run-time* class. Static methods, fields, and overloads are all chosen by the *compile-time* type of the expression. A subclass field with the same name **hides** the superclass field — both exist in the object — and a static method with the same signature hides rather than overrides.

```java
// file: Hiding.java
public class Hiding {
    static class Animal {
        String name = "animal";
        static String kind() { return "Animal.kind"; }
        String describe() { return "Animal.describe"; }
    }
    static class Cat extends Animal {
        String name = "cat";                                   // hides Animal.name
        static String kind() { return "Cat.kind"; }            // hides, does not override
        @Override String describe() { return "Cat.describe"; }
    }
    static void feed(Animal a) { System.out.println("feed(Animal)"); }
    static void feed(Cat c) { System.out.println("feed(Cat)"); }

    public static void main(String[] args) {
        Animal a = new Cat();
        System.out.println(a.describe());          // run-time type: Cat
        System.out.println(a.name);                // compile-time type: Animal
        System.out.println(((Cat) a).name);
        System.out.println(Animal.kind() + " / " + Cat.kind());
        feed(a);                                    // overload chosen at compile time
    }
}
```

```console
$ java Hiding.java
Cat.describe
animal
cat
Animal.kind / Cat.kind
feed(Animal)
```

## Unit 3.3 — Interfaces, abstract classes, and default methods

**One-line summary:** An **interface** declares abstract methods a class promises to implement; a class can implement many interfaces but extend only one class. Since Java 8 interfaces may have **default** methods (with bodies, inherited by implementers), **static** methods, and (Java 9) **private** helper methods — but no instance state. Use an interface to define a capability; use an abstract class when implementations share *state* and constructor logic.

```java
// file: Interfaces.java
public class Interfaces {
    interface Greeter {
        String name();
        default String greet() { return "Hello from " + name() + suffix(); }
        private String suffix() { return "!"; }
        static Greeter of(String n) { return () -> n; }          // a functional interface: a lambda implements it
    }

    interface Polite {
        default String greet() { return "Good day"; }
    }

    static class Butler implements Greeter, Polite {
        public String name() { return "Jeeves"; }
        @Override public String greet() {                        // must resolve the conflict explicitly
            return Polite.super.greet() + ". " + Greeter.super.greet();
        }
    }

    public static void main(String[] args) {
        System.out.println(Greeter.of("a lambda").greet());
        System.out.println(new Butler().greet());
    }
}
```

```console
$ java Interfaces.java
Hello from a lambda!
Good day. Hello from Jeeves!
```

When two interfaces supply conflicting default methods, the class must override and choose (`X.super.m()`); a method from a *class* in the hierarchy always wins over an interface default.

<!-- expanded -->
### Going deeper: the diamond problem with default methods

When a class inherits the same default method from two interfaces, Java refuses to guess: the class must override it, and may call either version explicitly with `InterfaceName.super.method()`. A method inherited from a *class* always wins over a default method from an interface.

```java
// file: Diamond.java
public class Diamond {
    interface Flyer { default String move() { return "fly"; } }
    interface Swimmer { default String move() { return "swim"; } }

    static class Duck implements Flyer, Swimmer {
        @Override public String move() {                       // required: the two defaults conflict
            return Flyer.super.move() + " and " + Swimmer.super.move();
        }
    }
    static class Base { public String move() { return "walk (from the class)"; } }
    static class Penguin extends Base implements Swimmer {}     // the class method wins, no conflict

    public static void main(String[] args) {
        System.out.println(new Duck().move());
        System.out.println(new Penguin().move());
    }
}
```

```console
$ java Diamond.java
fly and swim
walk (from the class)
$ sed 's/@Override public String move() {                       \/\/ required: the two defaults conflict/String other() {/' Diamond.java > Diamond2.java && sed -i 's/class Diamond /class Diamond2 /' Diamond2.java && javac Diamond2.java 2>&1 | grep -m1 -oE "error: .*"
error: types Flyer and Swimmer are incompatible;
```

## Topic 3B — Closed hierarchies

## Unit 3.4 — Sealed interfaces and records

**One-line summary:** A **sealed** class or interface (Java 17) lists exactly which classes may extend or implement it: `sealed interface Shape permits Circle, Square {}`. Each permitted subclass must be `final`, `sealed`, or `non-sealed`. Combined with records, sealed interfaces express **algebraic data types**: "a `Shape` is exactly a `Circle` or a `Square`", which the compiler can then check `switch` statements against.

<!-- expanded -->
### Exhaustive `switch` over a sealed hierarchy

A `sealed` interface lists every permitted implementation. A `switch` over it with type patterns needs no `default`, and when a new record is added to the `permits` list, every `switch` that does not handle it stops compiling — the compiler finds every place that needs updating.

```java
// file: Sealed.java
public class Sealed {
    sealed interface Payment permits Card, Upi, Cash {}
    record Card(String last4, long paise) implements Payment {}
    record Upi(String vpa, long paise) implements Payment {}
    record Cash(long paise) implements Payment {}

    static long fee(Payment p) {
        return switch (p) {                                    // exhaustive: no default needed
            case Card c when c.paise() > 100_000 -> c.paise() / 100;   // 1% on large card payments
            case Card c -> 200;
            case Upi u -> 0;
            case Cash c -> 0;
        };
    }

    public static void main(String[] args) {
        Payment[] ps = { new Card("4242", 250_000), new Card("1111", 5_000), new Upi("ada@bank", 99_900), new Cash(1_000) };
        for (Payment p : ps) System.out.println(p + " -> fee " + fee(p) + " paise");
    }
}
```

```console
$ java Sealed.java
Card[last4=4242, paise=250000] -> fee 2500 paise
Card[last4=1111, paise=5000] -> fee 200 paise
Upi[vpa=ada@bank, paise=99900] -> fee 0 paise
Cash[paise=1000] -> fee 0 paise
$ sed 's/permits Card, Upi, Cash {}/permits Card, Upi, Cash, Wallet {}\n    record Wallet(long paise) implements Payment {}/; s/class Sealed /class Sealed2 /' Sealed.java > Sealed2.java && javac Sealed2.java 2>&1 | grep -m1 -oE "error: .*"
error: the switch expression does not cover all possible input values
```

Adding `Wallet` made `fee` fail to compile until it handles the new case — exactly the help you want when the set of variants changes. A `default` branch would have silently swallowed `Wallet`, so leave it out on purpose.

## Unit 3.5 — Pattern matching

**One-line summary:** `obj instanceof Point p` tests and binds in one step (Java 16). `switch` accepts **type patterns** (`case Circle c ->`), **record patterns** that destructure (`case Point(var x, var y) ->`, nested to any depth), **guards** (`case Circle c when c.r() > 10 ->`), and `case null` (Java 21). A `switch` over a sealed type with patterns for every permitted subtype is **exhaustive** without `default` — and adding a new subtype makes every non-exhaustive switch a compile error.

```java
// file: Patterns.java
public class Patterns {
    sealed interface Shape permits Circle, Square, Rect {}
    record Circle(double r) implements Shape {}
    record Square(double side) implements Shape {}
    record Rect(Point topLeft, Point bottomRight) implements Shape {}
    record Point(double x, double y) {}

    static double area(Shape s) {
        return switch (s) {                                             // exhaustive: no default needed
            case Circle c -> Math.PI * c.r() * c.r();
            case Square(double side) -> side * side;                    // record pattern
            case Rect(Point(var x1, var y1), Point(var x2, var y2)) ->  // nested record patterns
                Math.abs(x2 - x1) * Math.abs(y2 - y1);
        };
    }

    static String classify(Object o) {
        return switch (o) {
            case null -> "null!";
            case Integer i when i < 0 -> "negative int " + i;
            case Integer i -> "int " + i;
            case String s when s.isBlank() -> "blank string";
            case String s -> "string of length " + s.length();
            case Shape s -> "a shape with area %.1f".formatted(area(s));
            case int[] arr -> "int array of " + arr.length;
            default -> "something else: " + o.getClass().getSimpleName();
        };
    }

    public static void main(String[] args) {
        Object[] things = {42, -7, "hi", "   ", new Circle(1), new Rect(new Point(0, 0), new Point(3, 4)), new int[3], 2.5, null};
        for (Object o : things) {
            System.out.println(classify(o));
        }
        Object maybe = new Square(3);
        if (maybe instanceof Square(double side) && side > 2) {
            System.out.println("big square, side " + side);
        }
    }
}
```

```console
$ java Patterns.java
int 42
negative int -7
string of length 2
blank string
a shape with area 3.1
a shape with area 12.0
int array of 3
something else: Double
null!
big square, side 3.0
```

Add a new permitted shape and forget to handle it, and `area` stops compiling:

```console
$ sed -e 's/permits Circle, Square, Rect/permits Circle, Square, Rect, Triangle/' -e 's/^    record Point/    record Triangle(double b, double h) implements Shape {}\n    record Point/' Patterns.java > Patterns2.java && sed -i 's/class Patterns /class Patterns2 /' Patterns2.java && javac -d /tmp Patterns2.java 2>&1 | head -n 3
Patterns2.java:11: error: the switch expression does not cover all possible input values
        return switch (s) {                                             // exhaustive: no default needed
               ^
```

### Deep & Tricky

**DT-13. Order matters in pattern `switch`: a more general pattern before a more specific one is a compile error ("dominated"), and guards are checked in order.** *(Guarantee: JLS §14.11.1)* `case Integer i -> …` followed by `case Integer i when i < 0 -> …` does not compile, because the first case matches everything the second would. Unlike a chain of `if`s, where the unreachable branch would silently never run, the compiler reports it. Also note that a pattern `switch` with no `case null` throws `NullPointerException` for a `null` selector, as old `switch` always did — `case null` is the opt-in.

### Pattern

**P-3. Use sealed interfaces with records and pattern matching for data with a fixed set of shapes; use open class hierarchies with virtual methods for behaviour that new types will extend.** *(Practice; the "expression problem")* With a sealed hierarchy, adding a new *operation* is easy (write one more `switch`), and adding a new *type* is checked (every `switch` fails to compile until it handles it). With an open hierarchy, adding a new *type* is easy (one more subclass), but adding an *operation* means touching every class. Pick the axis along which your code will change. *Trade-off:* sealed hierarchies must live in one module (or package, without modules); third parties cannot add cases.

## Topic 3C — When not to inherit

## Unit 3.6 — Composition over inheritance

**One-line summary:** Inheriting from a concrete class couples you to its implementation details, not just its documented behaviour — the **fragile base class** problem. Prefer **composition**: hold an instance and **forward** to it, implementing the same interface.

```java
// file: Fragile.java
import java.util.Collection;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class Fragile {
    /** Counts insertion attempts by extending HashSet. */
    static class InstrumentedHashSet<E> extends HashSet<E> {
        int attempts = 0;
        @Override public boolean add(E e) { attempts++; return super.add(e); }
        @Override public boolean addAll(Collection<? extends E> c) { attempts += c.size(); return super.addAll(c); }
    }

    /** The same, by composition: wraps any Set and forwards to it. */
    static class InstrumentedSet<E> extends java.util.AbstractSet<E> {
        private final Set<E> inner;
        int attempts = 0;
        InstrumentedSet(Set<E> inner) { this.inner = inner; }
        @Override public boolean add(E e) { attempts++; return inner.add(e); }
        @Override public boolean addAll(Collection<? extends E> c) { attempts += c.size(); return inner.addAll(c); }
        @Override public java.util.Iterator<E> iterator() { return inner.iterator(); }
        @Override public int size() { return inner.size(); }
    }

    public static void main(String[] args) {
        var inherited = new InstrumentedHashSet<String>();
        inherited.addAll(List.of("a", "b", "c"));
        var composed = new InstrumentedSet<String>(new HashSet<>());
        composed.addAll(List.of("a", "b", "c"));
        System.out.println("inheritance counted " + inherited.attempts + ", composition counted " + composed.attempts);
    }
}
```

```console
$ java Fragile.java
inheritance counted 6, composition counted 3
```

Through inheritance the count is 6, not 3, because `HashSet.addAll` (inherited from `AbstractCollection`) happens to call `add` for each element — which the subclass overrode — so every element is counted twice. That self-use is an implementation detail; another JDK version could change it. The composed version forwards to *any* `Set` and depends only on the `Set` interface.

<!-- expanded -->
### Deep & Tricky

**DT-14. `protected` in Java is *wider* than in C++: a `protected` member is visible to every class in the same package, not only to subclasses. Package-private (no modifier) is visible to the package only. A class in another package that extends yours can use your `protected` members only through references of its own type (or subtypes).** *(Guarantee: JLS §6.6.2)* Design for inheritance with `protected` deliberately, or not at all: mark classes `final` (or sealed) when subclassing is not part of the design, as *Effective Java* item 19 advises.

### Mastery check

1. **Predict.** In `Dispatch`, what does `((Animal) new Dog()).speak()` return, and what does `((Animal) new Dog()).name` return?
2. **Predict.** Does a `switch` over a `sealed interface` whose cases cover every permitted record still compile if one of those records is later changed to a non-sealed class with subclasses?

## Chapter 3 Project — Symbolic differentiation with sealed types

**Goal:** represent arithmetic expressions over one variable `x` as a sealed hierarchy of records; implement evaluation, printing with minimal parentheses, algebraic simplification, and symbolic differentiation — each as a single exhaustive `switch` with record patterns. Verify the derivatives numerically.

```java
// file: sym/Expr.java
/** An expression in one variable, x. */
public sealed interface Expr {
    record Num(double value) implements Expr {}
    record X() implements Expr {}
    record Add(Expr left, Expr right) implements Expr {}
    record Mul(Expr left, Expr right) implements Expr {}
    record Pow(Expr base, int exponent) implements Expr {}          // integer powers only
    record Sin(Expr arg) implements Expr {}
    record Cos(Expr arg) implements Expr {}
    record Neg(Expr arg) implements Expr {}

    static Expr num(double v) { return new Num(v); }
    Expr x = new X();
    static Expr add(Expr a, Expr b) { return new Add(a, b); }
    static Expr mul(Expr a, Expr b) { return new Mul(a, b); }
    static Expr pow(Expr a, int n) { return new Pow(a, n); }
    static Expr sin(Expr a) { return new Sin(a); }
    static Expr cos(Expr a) { return new Cos(a); }
}
```

```java
// file: sym/Algebra.java
import static java.lang.Math.*;

public final class Algebra {
    private Algebra() {}

    public static double eval(Expr e, double x) {
        return switch (e) {
            case Expr.Num(double v) -> v;
            case Expr.X() -> x;
            case Expr.Add(Expr a, Expr b) -> eval(a, x) + eval(b, x);
            case Expr.Mul(Expr a, Expr b) -> eval(a, x) * eval(b, x);
            case Expr.Pow(Expr b, int n) -> pow(eval(b, x), n);
            case Expr.Sin(Expr a) -> sin(eval(a, x));
            case Expr.Cos(Expr a) -> cos(eval(a, x));
            case Expr.Neg(Expr a) -> -eval(a, x);
        };
    }

    /** d/dx, by the sum, product, power (with chain), and trigonometric rules. */
    public static Expr derive(Expr e) {
        return switch (e) {
            case Expr.Num n -> new Expr.Num(0);
            case Expr.X x -> new Expr.Num(1);
            case Expr.Add(Expr a, Expr b) -> new Expr.Add(derive(a), derive(b));
            case Expr.Mul(Expr a, Expr b) -> new Expr.Add(new Expr.Mul(derive(a), b), new Expr.Mul(a, derive(b)));
            case Expr.Pow(Expr b, int n) -> new Expr.Mul(new Expr.Mul(new Expr.Num(n), new Expr.Pow(b, n - 1)), derive(b));
            case Expr.Sin(Expr a) -> new Expr.Mul(new Expr.Cos(a), derive(a));
            case Expr.Cos(Expr a) -> new Expr.Mul(new Expr.Neg(new Expr.Sin(a)), derive(a));
            case Expr.Neg(Expr a) -> new Expr.Neg(derive(a));
        };
    }

    /** One bottom-up pass of simplification rules; repeated until nothing changes. */
    public static Expr simplify(Expr e) {
        Expr prev;
        do {
            prev = e;
            e = step(e);
        } while (!e.equals(prev));                       // records compare structurally
        return e;
    }

    private static Expr step(Expr e) {
        return switch (e) {
            case Expr.Add(Expr a, Expr b) -> {
                Expr l = step(a), r = step(b);
                yield switch (new Expr.Add(l, r)) {
                    case Expr.Add(Expr.Num(double p), Expr.Num(double q)) -> new Expr.Num(p + q);
                    case Expr.Add(Expr.Num(double z), Expr o) when z == 0 -> o;
                    case Expr.Add(Expr o, Expr.Num(double z)) when z == 0 -> o;
                    case Expr.Add(Expr p, Expr q) when p.equals(q) -> new Expr.Mul(new Expr.Num(2), p);
                    case Expr.Add sum -> sum;
                };
            }
            case Expr.Mul(Expr a, Expr b) -> {
                Expr l = step(a), r = step(b);
                yield switch (new Expr.Mul(l, r)) {
                    case Expr.Mul(Expr.Num(double p), Expr.Num(double q)) -> new Expr.Num(p * q);
                    case Expr.Mul(Expr.Num(double z), Expr o) when z == 0 -> new Expr.Num(0);
                    case Expr.Mul(Expr o, Expr.Num(double z)) when z == 0 -> new Expr.Num(0);
                    case Expr.Mul(Expr.Num(double one), Expr o) when one == 1 -> o;
                    case Expr.Mul(Expr o, Expr.Num(double one)) when one == 1 -> o;
                    case Expr.Mul(Expr o, Expr.Num n) -> new Expr.Mul(n, o);                 // constants to the left
                    case Expr.Mul(Expr.Num(double p), Expr.Mul(Expr.Num(double q), Expr o)) -> new Expr.Mul(new Expr.Num(p * q), o);
                    case Expr.Mul(Expr p, Expr q) when p.equals(q) -> new Expr.Pow(p, 2);
                    case Expr.Mul m -> m;
                };
            }
            case Expr.Pow(Expr b, int n) when n == 0 -> new Expr.Num(1);
            case Expr.Pow(Expr b, int n) when n == 1 -> step(b);
            case Expr.Pow(Expr b, int n) -> new Expr.Pow(step(b), n);
            case Expr.Neg(Expr.Neg(Expr a)) -> step(a);
            case Expr.Neg(Expr.Num(double v)) -> new Expr.Num(-v);
            case Expr.Neg(Expr a) -> new Expr.Neg(step(a));
            case Expr.Sin(Expr a) -> new Expr.Sin(step(a));
            case Expr.Cos(Expr a) -> new Expr.Cos(step(a));
            case Expr.Num n -> n;
            case Expr.X x -> x;
        };
    }

    /** Prints with parentheses only where precedence requires them. */
    public static String show(Expr e) { return show(e, 0); }

    private static String show(Expr e, int outer) {                // precedence: 1 add, 2 mul, 3 neg, 4 pow
        return switch (e) {
            case Expr.Num(double v) -> v == Math.rint(v) ? Long.toString((long) v) : Double.toString(v);
            case Expr.X() -> "x";
            case Expr.Add(Expr a, Expr b) -> wrap(show(a, 1) + " + " + show(b, 1), 1, outer);
            case Expr.Mul(Expr a, Expr b) -> wrap(show(a, 2) + "·" + show(b, 2), 2, outer);
            case Expr.Neg(Expr a) -> wrap("-" + show(a, 3), 3, outer);
            case Expr.Pow(Expr b, int n) -> wrap(show(b, 5) + "^" + n, 4, outer);
            case Expr.Sin(Expr a) -> "sin(" + show(a, 0) + ")";
            case Expr.Cos(Expr a) -> "cos(" + show(a, 0) + ")";
        };
    }

    private static String wrap(String s, int prec, int outer) { return prec < outer ? "(" + s + ")" : s; }
}
```

```java
// file: sym/Main.java
import static java.lang.Math.abs;

public class Main {
    public static void main(String[] args) {
        Expr x = Expr.x;
        Expr[] tests = {
            Expr.add(Expr.mul(Expr.num(3), Expr.pow(x, 2)), Expr.mul(Expr.num(2), x)),                    // 3x^2 + 2x
            Expr.mul(x, Expr.sin(x)),                                                                    // x sin x
            Expr.sin(Expr.pow(x, 3)),                                                                    // sin(x^3)
            Expr.pow(Expr.add(Expr.cos(x), x), 2),                                                       // (cos x + x)^2
        };
        for (Expr f : tests) {
            Expr raw = Algebra.derive(f);
            Expr d = Algebra.simplify(raw);
            double worst = 0;
            for (double at = -2; at <= 2; at += 0.25) {                  // check against a central difference
                double h = 1e-5;
                double numeric = (Algebra.eval(f, at + h) - Algebra.eval(f, at - h)) / (2 * h);
                worst = Math.max(worst, abs(numeric - Algebra.eval(d, at)));
            }
            System.out.printf("f(x)  = %s%nf'(x) = %s%n        (unsimplified: %d chars; simplified: %d; max numeric error %.1e)%n",
                    Algebra.show(f), Algebra.show(d), Algebra.show(raw).length(), Algebra.show(d).length(), worst);
        }
    }
}
```

```console
$ cd sym && rm -rf out && javac -Xlint:all -d out *.java && java -cp out Main
f(x)  = 3·x^2 + 2·x
f'(x) = 6·x + 2
        (unsimplified: 29 chars; simplified: 7; max numeric error 1.2e-10)
f(x)  = x·sin(x)
f'(x) = sin(x) + x·cos(x)
        (unsimplified: 21 chars; simplified: 17; max numeric error 5.5e-11)
f(x)  = sin(x^3)
f'(x) = cos(x^3)·3·x^2
        (unsimplified: 16 chars; simplified: 14; max numeric error 5.5e-09)
f(x)  = (cos(x) + x)^2
f'(x) = 2·(cos(x) + x)·(-sin(x) + 1)
        (unsimplified: 32 chars; simplified: 28; max numeric error 1.9e-10)
```

Each operation is one `switch` that the compiler checks for exhaustiveness: if `Expr` gains a `Log` record, `eval`, `derive`, `step`, and `show` all stop compiling until they handle it (P-3). The simplifier shows guards and nested record patterns doing real work — `Mul(Num(p), Mul(Num(q), o))` folds constants across nested products — and terminates because each rule makes the tree smaller or moves it toward a canonical form, with records' structural `equals` detecting the fixed point. The numeric check compares each symbolic derivative with a central-difference estimate at 17 points.

### Review checklist

- Why can `simplify` use `equals` to detect that nothing changed, without writing any `equals` method?
- The rule `Add(p, q) when p.equals(q) → 2·p` is applied before constants are folded inside `p`. Can you construct an input where a simplification is missed because rules run in the "wrong" order? How would you make simplification confluent?
- Add `Exp(Expr)` (e^x). Which methods must change, and does the compiler find them all?

## Answers to the Chapter 3 prediction questions

**3.6 Q1.** `"woof"` (methods dispatch on the run-time class) and `"animal"` (fields are chosen by the static type, `Animal`).

**3.6 Q2.** Yes. A `non-sealed` permitted class can have subclasses the compiler does not know, but the `case` for that class matches all of them, so the `switch` is still exhaustive. What breaks exhaustiveness is adding a new *permitted* subtype without a case for it — as the `Patterns2` experiment showed.

# Chapter 4 — Exceptions

*Goal: use Java's exception system well — checked versus unchecked exceptions, `try`/`catch`/`finally`, try-with-resources and suppressed exceptions, chaining and translation across layers — know what exceptions cost, and choose between exceptions, `Optional`, and result types.*

**What you will be able to do after this chapter**

- Navigate the `Throwable` hierarchy and decide between checked and unchecked exceptions.
- Write `try`/`catch`/`finally` correctly, including the cases where `finally` changes the outcome.
- Manage resources with try-with-resources, and read suppressed exceptions.
- Define exception types, chain causes, and translate exceptions at layer boundaries.
- Measure the cost of exceptions, and use `Optional` or result types where failure is expected.

## Topic 4A — The mechanism

## Unit 4.1 — The exception hierarchy

**One-line summary:** Everything thrown is a `Throwable`. **`Error`** (e.g. `OutOfMemoryError`, `StackOverflowError`) signals problems an application should not try to handle. **`Exception`** subclasses are **checked**: a method that may throw one must declare it (`throws IOException`), and callers must catch or declare it. **`RuntimeException`** subclasses (`NullPointerException`, `IllegalArgumentException`, `IndexOutOfBoundsException`, …) are **unchecked**: they usually indicate programming errors and need not be declared.

```java
// file: Hierarchy.java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class Hierarchy {
    static String firstLine(Path p) throws IOException {      // checked: must be declared
        return Files.readAllLines(p).get(0);                  // get(0) may throw an unchecked exception
    }

    static int depth(int n) {
        return depth(n + 1) + 1;                              // never returns
    }

    public static void main(String[] args) throws Exception {
        Files.writeString(Path.of("empty.txt"), "");
        for (String name : new String[] {"Hierarchy.java", "missing.txt", "empty.txt"}) {
            try {
                System.out.println("first line: " + firstLine(Path.of(name)).strip());
            } catch (IOException e) {
                System.out.println("checked   " + e.getClass().getName() + ": " + e.getMessage());
            } catch (IndexOutOfBoundsException e) {
                System.out.println("unchecked " + e.getClass().getName() + ": " + e.getMessage());
            }
        }
        try {
            depth(0);
        } catch (StackOverflowError e) {
            System.out.println("Error     " + e.getClass().getName() + " (stack trace has " + e.getStackTrace().length + " frames)");
        }
        for (Class<?> c = NumberFormatException.class; c != null; c = c.getSuperclass()) {
            System.out.print(c.getSimpleName() + (c.getSuperclass() != null ? " -> " : "\n"));
        }
    }
}
```

```console
$ java Hierarchy.java
first line: // file: Hierarchy.java
checked   java.nio.file.NoSuchFileException: missing.txt
unchecked java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
Error     java.lang.StackOverflowError (stack trace has 1024 frames)
NumberFormatException -> IllegalArgumentException -> RuntimeException -> Exception -> Throwable -> Object
```

Catching `StackOverflowError` is shown only to make it visible; in real code, an `Error` means the JVM is in trouble, and the right response is usually to let the thread die and be restarted by something above it. The stack trace was cut at 1,024 frames (the JVM's default `-XX:MaxJavaStackTraceDepth`).

<!-- expanded -->
### Going deeper: checked or unchecked?

Java is unusual in having **checked exceptions** (subclasses of `Exception` but not `RuntimeException`): the compiler forces callers to catch them or declare them with `throws`. The guideline used by most modern code: use checked exceptions for conditions a *reasonable caller can recover from* and must think about (a file that may not exist, a network call that may fail), and unchecked exceptions for *programming errors* (a null argument, an index out of range, a broken invariant). Lambdas and streams do not get along with checked exceptions — a `Function` cannot throw `IOException` — which is one reason many libraries wrap them in `UncheckedIOException`.

```java
// file: CheckedLambda.java
import java.io.IOException;
import java.io.UncheckedIOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

public class CheckedLambda {
    static long size(Path p) throws IOException { return Files.size(p); }

    public static void main(String[] args) throws IOException {
        Path a = Files.writeString(Path.of("a.txt"), "hello");
        List<Path> paths = List.of(a, Path.of("missing.txt"));
        // paths.stream().map(p -> size(p))        // error: unreported exception IOException
        try {
            long total = paths.stream().mapToLong(p -> {
                try { return size(p); }
                catch (IOException e) { throw new UncheckedIOException(e); }   // wrap at the lambda
            }).sum();
            System.out.println("total " + total);
        } catch (UncheckedIOException e) {
            System.out.println("failed: " + e.getCause().getClass().getSimpleName() + " " + e.getCause().getMessage());
        }
    }
}
```

```console
$ java CheckedLambda.java
failed: NoSuchFileException missing.txt
```

## Unit 4.2 — `try`, `catch`, `finally`

**One-line summary:** `catch` clauses are tried in order, the first matching one runs (a more specific type must come before a more general one); **multi-catch** (`catch (A | B e)`) handles several types identically. A `finally` block runs whether the `try` completes normally, returns, or throws — and a `return` or `throw` inside `finally` *replaces* whatever the `try` was doing.

```java
// file: Finally.java
public class Finally {
    static int returnsFromTry() {
        try {
            return 1;
        } finally {
            System.out.println("  finally runs before the method returns");
        }
    }

    @SuppressWarnings("finally")
    static int finallyOverridesReturn() {
        try {
            return 1;
        } finally {
            return 2;                                          // replaces the pending return
        }
    }

    @SuppressWarnings("finally")
    static int finallySwallowsException() {
        try {
            throw new IllegalStateException("important failure");
        } finally {
            return 3;                                          // the exception is silently lost
        }
    }

    static int finallyCannotChangeReturnedValue() {
        int x = 10;
        try {
            return x;                                          // the value 10 is saved here
        } finally {
            x = 20;                                            // too late
        }
    }

    public static void main(String[] args) {
        System.out.println("returnsFromTry: " + returnsFromTry());
        System.out.println("finallyOverridesReturn: " + finallyOverridesReturn());
        System.out.println("finallySwallowsException: " + finallySwallowsException());
        System.out.println("finallyCannotChangeReturnedValue: " + finallyCannotChangeReturnedValue());
        try {
            Object o = "text";
            Integer i = (Integer) o;
            System.out.println(i);
        } catch (ClassCastException | NullPointerException e) {    // multi-catch
            System.out.println("multi-catch: " + e.getClass().getSimpleName());
        }
    }
}
```

```console
$ java Finally.java
  finally runs before the method returns
returnsFromTry: 1
finallyOverridesReturn: 2
finallySwallowsException: 3
finallyCannotChangeReturnedValue: 10
multi-catch: ClassCastException
$ javac -Xlint:finally -d /tmp Finally.java 2>&1 | head -n 2; sed -i 's/    @SuppressWarnings("finally")//' Finally.java && javac -Xlint:finally -d /tmp Finally.java 2>&1 | grep warning
Finally.java:17: warning: [finally] finally clause cannot complete normally
Finally.java:26: warning: [finally] finally clause cannot complete normally
2 warnings
```

### Deep & Tricky

**DT-15. A `return` (or `throw`) in `finally` silently discards an exception thrown by the `try` block.** *(Guarantee: JLS §14.20.2)* `finallySwallowsException` returns 3 and the `IllegalStateException` vanishes without a trace — no log, no stack trace. `javac -Xlint:finally` warns ("finally clause cannot complete normally"); IDEs flag it too. Never return or throw from `finally`; use it only for cleanup that cannot fail — and for resources, prefer try-with-resources, which handles a failing cleanup correctly (Unit 4.3).

## Unit 4.3 — try-with-resources and suppressed exceptions

**One-line summary:** `try (var r = open()) { ... }` closes `r` automatically when the block exits, normally or by exception, for any `AutoCloseable`. Several resources are closed in **reverse** order of opening. If the body throws *and* `close()` throws, the body's exception propagates and the close exception is attached to it as a **suppressed** exception — nothing is lost.

```java
// file: Resources.java
public class Resources {
    record Res(String name, boolean failOnClose) implements AutoCloseable {
        Res {
            System.out.println("  open " + name);
        }
        @Override public void close() {
            System.out.println("  close " + name);
            if (failOnClose) throw new IllegalStateException("close failed: " + name);
        }
    }

    public static void main(String[] args) {
        System.out.println("normal exit:");
        try (var a = new Res("A", false); var b = new Res("B", false)) {
            System.out.println("  body uses " + a.name() + " and " + b.name());
        }

        System.out.println("body throws, and B's close throws too:");
        try (var a = new Res("A", false); var b = new Res("B", true)) {
            throw new RuntimeException("body failed");
        } catch (RuntimeException e) {
            System.out.println("caught: " + e.getMessage());
            for (Throwable s : e.getSuppressed()) {
                System.out.println("  suppressed: " + s.getMessage());
            }
        }
    }
}
```

```console
$ java Resources.java
normal exit:
  open A
  open B
  body uses A and B
  close B
  close A
body throws, and B's close throws too:
  open A
  open B
  close B
  close A
caught: body failed
  suppressed: close failed: B
```

The hand-written equivalent with `finally { b.close(); a.close(); }` gets this wrong in two ways: if `b.close()` throws, `a` is never closed, and the exception from `close` *replaces* the body's exception — the more informative one — exactly the problem of DT-15.

<!-- expanded -->
### Going deeper: the order of closing, and resources declared outside

Resources in one `try (...)` are closed in reverse order of declaration — the last opened is closed first, as with nested blocks. Since Java 9, an existing *effectively final* variable can be listed too: `try (in) { ... }`.

```java
// file: CloseOrder.java
public class CloseOrder {
    record Res(String name) implements AutoCloseable {
        Res { System.out.println("open " + name); }
        @Override public void close() { System.out.println("close " + name); }
    }

    public static void main(String[] args) {
        Res db = new Res("database");
        try (db; Res file = new Res("file"); Res lock = new Res("lock")) {
            System.out.println("working with " + db.name() + ", " + file.name() + ", " + lock.name());
        }
        System.out.println("after try");
    }
}
```

```console
$ java CloseOrder.java
open database
open file
open lock
working with database, file, lock
close lock
close file
close database
after try
```

## Topic 4B — Designing with exceptions

## Unit 4.4 — Custom exceptions, chaining, and translation

**One-line summary:** Define an exception type when callers need to distinguish a failure or when it carries useful data. Always pass the underlying exception as the **cause** (`new MyException("context", cause)`), so the stack trace shows the whole chain ("Caused by: …"). At a layer boundary, **translate** low-level exceptions into the layer's own vocabulary rather than leaking implementation details — and wrap checked exceptions that cannot be handled in unchecked ones (e.g. `UncheckedIOException`) when an interface does not allow `throws`.

```java
// file: Chaining.java
import java.io.IOException;
import java.io.UncheckedIOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

public class Chaining {
    /** The repository layer's own exception: callers need not know about files. */
    static class RepositoryException extends Exception {
        private final String key;
        RepositoryException(String message, String key, Throwable cause) {
            super(message, cause);
            this.key = key;
        }
        String key() { return key; }
    }

    static String load(String key) throws RepositoryException {
        try {
            return Files.readString(Path.of("store", key + ".txt"));
        } catch (IOException e) {
            throw new RepositoryException("cannot load " + key, key, e);       // translate, keep the cause
        }
    }

    public static void main(String[] args) {
        try {
            load("customer-42");
        } catch (RepositoryException e) {
            System.out.println(e.getMessage() + " (key " + e.key() + ")");
            System.out.println("cause: " + e.getCause());
        }
        // inside a lambda, checked exceptions are not allowed: wrap in an unchecked one
        try {
            List.of("a", "b").forEach(k -> {
                try {
                    load(k);
                } catch (RepositoryException e) {
                    throw new UncheckedIOException((IOException) e.getCause());
                }
            });
        } catch (UncheckedIOException e) {
            System.out.println("unchecked wrapper: " + e.getCause().getClass().getSimpleName());
        }
    }
}
```

```console
$ java Chaining.java
cannot load customer-42 (key customer-42)
cause: java.nio.file.NoSuchFileException: store/customer-42.txt
unchecked wrapper: NoSuchFileException
```

### Pattern

**P-4. Use checked exceptions for recoverable conditions the caller is expected to handle; use unchecked exceptions for programming errors; translate at boundaries; never swallow.** *(Practice; Effective Java, Items 70–77)* A missing file a user named is recoverable (checked); a `null` passed where the documentation forbids it is a bug (unchecked, thrown early with `Objects.requireNonNull`). An empty `catch` block, or one that only prints the stack trace and continues, turns a loud failure into a silent wrong result — catch only what you can handle, and let the rest propagate to a place that can log it and decide. *Trade-off:* checked exceptions do not compose with lambdas and streams (Unit 4.4's wrapper), which is why many modern libraries use unchecked exceptions throughout and document them instead.

## Unit 4.5 — What exceptions cost

**One-line summary:** Creating an exception captures a **stack trace** — walking every frame of the current stack — which is the expensive part; throwing and catching are cheap when the JIT can see both ends. Exceptions for *exceptional* situations cost nothing on the normal path; exceptions used for *control flow* (e.g. `Integer.parseInt` in a validation loop where most inputs are invalid) can dominate a program's run time.

```java
// file: ExceptionCost.java
public class ExceptionCost {
    static Integer parseOrNull(String s) {
        try {
            return Integer.parseInt(s);
        } catch (NumberFormatException e) {                       // exceptions as control flow
            return null;
        }
    }

    static Integer parseCheckFirst(String s) {
        if (s.isEmpty() || s.length() > 10) return null;
        int start = s.charAt(0) == '-' ? 1 : 0;
        if (start == s.length()) return null;
        for (int i = start; i < s.length(); i++) {
            if (!Character.isDigit(s.charAt(i))) return null;
        }
        long v = Long.parseLong(s);                                 // safe now: digits only, at most 10
        return (v < Integer.MIN_VALUE || v > Integer.MAX_VALUE) ? null : (int) v;
    }

    static class Cheap extends RuntimeException {
        Cheap() { super(null, null, false, false); }               // no stack trace, no suppression
    }

    static int recurse(int depth, boolean cheap) {
        if (depth == 0) throw cheap ? new Cheap() : new RuntimeException();
        return recurse(depth - 1, cheap) + 1;
    }

    public static void main(String[] args) {
        String[] inputs = new String[1_000_000];
        for (int i = 0; i < inputs.length; i++) inputs[i] = (i % 10 == 0) ? Integer.toString(i) : "x" + i;   // 90% invalid
        for (int round = 0; round < 3; round++) {
            long t = System.nanoTime();
            int a = 0;
            for (String s : inputs) if (parseOrNull(s) != null) a++;
            long t1 = System.nanoTime();
            int b = 0;
            for (String s : inputs) if (parseCheckFirst(s) != null) b++;
            long t2 = System.nanoTime();
            System.out.printf("round %d: try/catch %5.0f ms, check first %4.0f ms (valid: %d = %d)%n", round, (t1 - t) / 1e6, (t2 - t1) / 1e6, a, b);
        }
        for (boolean cheap : new boolean[] {false, true}) {
            long t = System.nanoTime();
            for (int i = 0; i < 100_000; i++) {
                try { recurse(50, cheap); } catch (RuntimeException e) { /* expected */ }
            }
            System.out.printf("throw from 50 frames deep x100k, %s: %.0f ms%n", cheap ? "no stack trace" : "with stack trace", (System.nanoTime() - t) / 1e6);
        }
    }
}
```

```console
$ java ExceptionCost.java
round 0: try/catch  1516 ms, check first   27 ms (valid: 100000 = 100000)
round 1: try/catch   782 ms, check first   10 ms (valid: 100000 = 100000)
round 2: try/catch   636 ms, check first   10 ms (valid: 100000 = 100000)
throw from 50 frames deep x100k, with stack trace: 306 ms
throw from 50 frames deep x100k, no stack trace: 115 ms
```

With 90% of inputs invalid, parsing by exception was about seventy times slower than checking first, even after warm-up. The second measurement isolates the stack trace: the same throw from 50 frames deep cost more than twice as much when the exception captured its stack trace. An exception type that is thrown often on purpose can disable trace capture through the four-argument `Throwable` constructor, as `Cheap` does — at the price of useless traces if one ever escapes.

### Deep & Tricky

**DT-16. HotSpot may throw pre-allocated exceptions *without* stack traces for frequently thrown built-in exceptions.** *(Implementation behaviour: `-XX:+OmitStackTraceInFastThrow`, on by default)* When compiled code throws the same implicit `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`, or `ClassCastException` many times at one site, C2 replaces it with a preallocated instance that has **no message and no stack trace**. Production logs then show a bare `java.lang.NullPointerException` with nothing after it. The first occurrences did have traces — search earlier in the log — or run with `-XX:-OmitStackTraceInFastThrow` while diagnosing.

## Unit 4.6 — `Optional` and result types

**One-line summary:** For "no value" that is a normal outcome, return `Optional<T>` rather than `null` or throwing: `Optional.empty()`, `Optional.of(v)`, and methods `map`, `flatMap`, `filter`, `orElse`, `orElseGet`, `orElseThrow`, `ifPresentOrElse`. For failures that carry information and are expected (validation), a small **sealed result type** makes the outcome explicit in the signature, like Rust's `Result`. Use `Optional` only as a return type — not for fields, parameters, or collection elements.

```java
// file: Results.java
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class Results {
    static final Map<String, String> EMAILS = Map.of("ada", "ada@example.com", "alan", "");

    static Optional<String> email(String user) {
        return Optional.ofNullable(EMAILS.get(user)).filter(e -> !e.isBlank());
    }

    sealed interface Parsed<T> permits Ok, Err {}
    record Ok<T>(T value) implements Parsed<T> {}
    record Err<T>(String reason, int position) implements Parsed<T> {}

    static Parsed<Integer> parsePort(String s) {
        for (int i = 0; i < s.length(); i++) {
            if (!Character.isDigit(s.charAt(i))) return new Err<>("not a digit: '" + s.charAt(i) + "'", i);
        }
        if (s.isEmpty()) return new Err<>("empty", 0);
        int v = Integer.parseInt(s.length() > 5 ? "99999" : s);
        return v >= 1 && v <= 65535 ? new Ok<>(v) : new Err<>("out of range 1..65535", 0);
    }

    public static void main(String[] args) {
        for (String user : List.of("ada", "alan", "grace")) {
            String domain = email(user).map(e -> e.substring(e.indexOf('@') + 1)).orElse("(none)");
            System.out.println(user + " -> " + domain);
        }
        for (String s : List.of("8080", "80a0", "", "70000")) {
            String msg = switch (parsePort(s)) {
                case Ok<Integer>(Integer port) -> "port " + port;
                case Err<Integer>(String why, int at) -> "invalid at " + at + ": " + why;
            };
            System.out.println("\"" + s + "\" -> " + msg);
        }
    }
}
```

```console
$ java Results.java
ada -> example.com
alan -> (none)
grace -> (none)
"8080" -> port 8080
"80a0" -> invalid at 2: not a digit: 'a'
"" -> invalid at 0: empty
"70000" -> invalid at 0: out of range 1..65535
```

<!-- expanded -->
### Deep & Tricky

**DT-17. `Optional.orElse(x)` always evaluates `x`, even when the `Optional` holds a value — so `opt.orElse(loadFromDatabase())` hits the database every time. Use `orElseGet(() -> loadFromDatabase())` for anything expensive or with side effects.** *(Library behaviour: `orElse` is an ordinary method, and Java evaluates arguments before the call)*

```java
// file: OrElse.java
import java.util.Optional;

public class OrElse {
    static int calls = 0;
    static String expensiveDefault() { calls++; return "default"; }

    public static void main(String[] args) {
        Optional<String> present = Optional.of("cached");
        String a = present.orElse(expensiveDefault());
        System.out.println(a + ", expensiveDefault calls with orElse: " + calls);
        calls = 0;
        String b = present.orElseGet(OrElse::expensiveDefault);
        System.out.println(b + ", expensiveDefault calls with orElseGet: " + calls);

        Optional<String> name = Optional.of("  Ada ");
        System.out.println(name.map(String::strip).filter(s -> !s.isEmpty()).map(String::toUpperCase).orElse("?"));
        System.out.println(Optional.empty().or(() -> Optional.of("fallback")).get());
    }
}
```

```console
$ java OrElse.java
cached, expensiveDefault calls with orElse: 1
cached, expensiveDefault calls with orElseGet: 0
ADA
fallback
```

Two more rules from the API's designers: return `Optional` from methods that may have no result, but do not use it for fields or parameters, and never return `null` from a method declared to return `Optional`.

### Mastery check

1. **Predict.** In try-with-resources with two resources, `A` opened first and `B` second, the body completes normally but both `close` calls throw. Which exception propagates, and where is the other?
2. **Predict.** What does `Optional.of(null)` do? And `Optional.ofNullable(null).map(String::valueOf).isPresent()`?

## Chapter 4 Project — An all-or-nothing CSV importer

**Goal:** import a CSV file of products into a data file, validating every row. Errors in rows must all be reported together, with line and column. The import is **all or nothing**: if any row is invalid, or if writing fails halfway, the existing data file is left exactly as it was. Resources are closed on every path, and failures are translated into the importer's own exception type with causes preserved.

```java
// file: importer/Importer.java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.math.BigDecimal;
import java.nio.charset.StandardCharsets;
import java.nio.file.AtomicMoveNotSupportedException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardCopyOption;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class Importer {
    public record Product(String sku, String name, BigDecimal price, int stock) {}

    public record RowError(int line, int column, String message) {
        @Override public String toString() { return "line " + line + ", column " + column + ": " + message; }
    }

    /** The importer's own failure type: either the input is invalid, or I/O failed. */
    public static class ImportException extends Exception {
        private final List<RowError> errors;
        ImportException(String message, List<RowError> errors, Throwable cause) {
            super(message, cause);
            this.errors = List.copyOf(errors);
        }
        public List<RowError> errors() { return errors; }
    }

    static List<Product> parse(BufferedReader in) throws IOException, ImportException {
        List<Product> products = new ArrayList<>();
        List<RowError> errors = new ArrayList<>();
        Set<String> seen = new HashSet<>();
        String header = in.readLine();
        if (!"sku,name,price,stock".equals(header)) {
            throw new ImportException("unexpected header: " + header, List.of(new RowError(1, 1, "expected sku,name,price,stock")), null);
        }
        String line;
        int lineNo = 1;
        while ((line = in.readLine()) != null) {
            lineNo++;
            if (line.isBlank()) continue;
            String[] f = line.split(",", -1);                  // -1: keep trailing empty fields
            if (f.length != 4) {
                errors.add(new RowError(lineNo, 1, "expected 4 fields, found " + f.length));
                continue;
            }
            int before = errors.size();
            String sku = f[0].strip();
            if (!sku.matches("[A-Z]{3}-\\d{4}")) errors.add(new RowError(lineNo, 1, "bad SKU '" + sku + "'"));
            else if (!seen.add(sku)) errors.add(new RowError(lineNo, 1, "duplicate SKU " + sku));
            String name = f[1].strip();
            if (name.isEmpty()) errors.add(new RowError(lineNo, 2, "empty name"));
            BigDecimal price = null;
            try {
                price = new BigDecimal(f[2].strip());
                if (price.signum() < 0 || price.scale() > 2) errors.add(new RowError(lineNo, 3, "price must be >= 0 with at most 2 decimals: " + price));
            } catch (NumberFormatException e) {
                errors.add(new RowError(lineNo, 3, "not a number '" + f[2].strip() + "'"));
            }
            int stock = 0;
            try {
                stock = Integer.parseInt(f[3].strip());
                if (stock < 0) errors.add(new RowError(lineNo, 4, "negative stock " + stock));
            } catch (NumberFormatException e) {
                errors.add(new RowError(lineNo, 4, "not an integer '" + f[3].strip() + "'"));
            }
            if (errors.size() == before) products.add(new Product(sku, name, price, stock));
        }
        if (!errors.isEmpty()) {
            throw new ImportException(errors.size() + " invalid value(s); nothing imported", errors, null);
        }
        return products;
    }

    /** Validates `csv` completely, then replaces `data` atomically. On any failure, `data` is untouched. */
    public static int importFile(Path csv, Path data, int failAfterRows) throws ImportException {
        List<Product> products;
        try (BufferedReader in = Files.newBufferedReader(csv, StandardCharsets.UTF_8)) {
            products = parse(in);
        } catch (IOException e) {
            throw new ImportException("cannot read " + csv, List.of(), e);
        }
        Path tmp = data.resolveSibling(data.getFileName() + ".tmp");
        try {
            try (BufferedWriter out = Files.newBufferedWriter(tmp, StandardCharsets.UTF_8)) {
                int written = 0;
                for (Product p : products) {
                    if (written++ == failAfterRows) throw new IOException("simulated disk full after " + failAfterRows + " rows");
                    out.write(p.sku() + "\t" + p.name() + "\t" + p.price() + "\t" + p.stock());
                    out.newLine();
                }
            }
            try {
                Files.move(tmp, data, StandardCopyOption.REPLACE_EXISTING, StandardCopyOption.ATOMIC_MOVE);
            } catch (AtomicMoveNotSupportedException e) {
                Files.move(tmp, data, StandardCopyOption.REPLACE_EXISTING);
            }
            return products.size();
        } catch (IOException e) {
            try {
                Files.deleteIfExists(tmp);
            } catch (IOException cleanup) {
                e.addSuppressed(cleanup);                  // keep the original failure, attach the cleanup failure
            }
            throw new ImportException("cannot write " + data + "; previous data kept", List.of(), e);
        }
    }

    public static void main(String[] args) throws IOException {
        Path data = Path.of("products.tsv");
        Files.writeString(data, "OLD-0001\tprevious catalogue\t1.00\t1\n");
        String[][] runs = {
            {"good.csv", "-1"}, {"bad.csv", "-1"}, {"good.csv", "2"}, {"missing.csv", "-1"},
        };
        for (String[] run : runs) {
            System.out.println("== import " + run[0] + (run[1].equals("-1") ? "" : " (failing after " + run[1] + " rows)"));
            try {
                System.out.println("imported " + importFile(Path.of(run[0]), data, Integer.parseInt(run[1])) + " products");
            } catch (ImportException e) {
                System.out.println("FAILED: " + e.getMessage());
                e.errors().forEach(err -> System.out.println("  " + err));
                if (e.getCause() != null) System.out.println("  cause: " + e.getCause());
            }
            System.out.println("data file now starts with: " + Files.readAllLines(data).get(0).replace('\t', '|')
                    + " (" + Files.readAllLines(data).size() + " lines), temp file left behind: " + Files.exists(Path.of("products.tsv.tmp")));
        }
    }
}
```

```text
<!-- file: importer/good.csv -->
sku,name,price,stock
ABC-0001,Blue pen,1.20,500
ABC-0002,Red pen,1.20,350
XYZ-1000,Notebook A5,4.95,120
XYZ-1001,Notebook A4,6.50,80
```

```text
<!-- file: importer/bad.csv -->
sku,name,price,stock
ABC-0001,Blue pen,1.20,500
abc-0002,Red pen,1.20,350
ABC-0001,Duplicate,2.00,1
XYZ-1000,,4.995,-3
XYZ-1001,Notebook A4,six,80
XYZ-1002,Too,many,fields,here
```

```console
$ cd importer && java Importer.java
== import good.csv
imported 4 products
data file now starts with: ABC-0001|Blue pen|1.20|500 (4 lines), temp file left behind: false
== import bad.csv
FAILED: 7 invalid value(s); nothing imported
  line 3, column 1: bad SKU 'abc-0002'
  line 4, column 1: duplicate SKU ABC-0001
  line 5, column 2: empty name
  line 5, column 3: price must be >= 0 with at most 2 decimals: 4.995
  line 5, column 4: negative stock -3
  line 6, column 3: not a number 'six'
  line 7, column 1: expected 4 fields, found 5
data file now starts with: ABC-0001|Blue pen|1.20|500 (4 lines), temp file left behind: false
== import good.csv (failing after 2 rows)
FAILED: cannot write products.tsv; previous data kept
  cause: java.io.IOException: simulated disk full after 2 rows
data file now starts with: ABC-0001|Blue pen|1.20|500 (4 lines), temp file left behind: false
== import missing.csv
FAILED: cannot read missing.csv
  cause: java.nio.file.NoSuchFileException: missing.csv
data file now starts with: ABC-0001|Blue pen|1.20|500 (4 lines), temp file left behind: false
```

The importer separates **validation** (collect every problem, then fail once with the full list) from **effects** (write a temporary file, then replace the data file with one atomic rename). That gives the all-or-nothing behaviour: the invalid file changed nothing; the simulated disk failure halfway through writing left the previous catalogue in place and cleaned up its temporary file; and the missing input was reported with its cause. Every reader and writer is closed by try-with-resources, and the cleanup failure path keeps the original exception and attaches the cleanup's as suppressed.

### Review checklist

- Which exceptions are checked here, and where are they translated into `ImportException`? Why does `ImportException` carry a list of `RowError` *and* an optional cause?
- What would happen if the program crashed (power loss) between writing the temp file and the move? (The old file is intact; a stale `.tmp` remains for the next run to overwrite.)
- Is `split(",")` a correct CSV parser? (No: quoted fields may contain commas. What would you use?)

## Answers to the Chapter 4 prediction questions

**4.6 Q1.** Resources close in reverse order: `B.close()` throws first; then `A.close()` still runs and throws. The first exception — `B`'s — propagates, and `A`'s is attached to it as suppressed.

**4.6 Q2.** `Optional.of(null)` throws `NullPointerException` (use `ofNullable` when the value may be null). The second expression is `false`: mapping an empty `Optional` gives an empty `Optional`.

# Chapter 5 — Generics in Depth

*Goal: write and use generic types and methods confidently — bounded types, wildcards and the PECS rule, generic methods with recursive bounds — and understand type erasure well enough to predict exactly what generics can and cannot do at run time.*

**What you will be able to do after this chapter**

- Write generic classes, interfaces, and methods, with bounds.
- Explain type erasure, bridge methods, raw types, and heap pollution.
- Use `? extends` and `? super` correctly (PECS), and explain why `List<Integer>` is not a `List<Number>`.
- Work around erasure with class tokens and type tokens.
- Use generics with varargs safely.

## Topic 5A — Generic types and methods

## Unit 5.1 — Generic classes and methods

**One-line summary:** A **generic class** `class Box<T>` is parameterised by a type; `Box<String>` and `Box<Integer>` are checked separately at compile time. A **generic method** `static <T> T first(List<T> xs)` declares its own type parameter, usually inferred from the arguments. A **bounded** parameter `<T extends Comparable<? super T>>` restricts which types are allowed and makes their methods available. The diamond `new Box<>()` lets the compiler infer the arguments.

```java
// file: GenericBasics.java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.Optional;

public class GenericBasics {
    /** An immutable pair. */
    record Pair<A, B>(A first, B second) {
        <C> Pair<A, C> withSecond(C c) { return new Pair<>(first, c); }
    }

    /** The largest element, for any type that can be compared with itself (or a supertype). */
    static <T extends Comparable<? super T>> Optional<T> max(Collection<? extends T> items) {
        T best = null;
        for (T x : items) {
            if (best == null || x.compareTo(best) > 0) best = x;
        }
        return Optional.ofNullable(best);
    }

    /** A growable stack backed by an ArrayList. */
    static class Stack<E> {
        private final List<E> items = new ArrayList<>();
        void push(E e) { items.add(e); }
        E pop() {
            if (items.isEmpty()) throw new IllegalStateException("empty stack");
            return items.remove(items.size() - 1);
        }
        boolean isEmpty() { return items.isEmpty(); }
    }

    public static void main(String[] args) {
        Pair<String, Integer> p = new Pair<>("age", 36);
        Pair<String, Double> q = p.withSecond(36.5);
        System.out.println(p + " " + q);
        System.out.println(max(List.of(3, 9, 4)) + " " + max(List.of("pear", "apple")) + " " + max(List.<Integer>of()));
        Stack<String> s = new Stack<>();
        for (String w : "generics are checked at compile time".split(" ")) s.push(w);
        StringBuilder sb = new StringBuilder();
        while (!s.isEmpty()) sb.append(s.pop()).append(' ');
        System.out.println(sb.toString().strip());
        // s.push(42);                       // compile error: incompatible types: int cannot be converted to String
    }
}
```

```console
$ java GenericBasics.java
Pair[first=age, second=36] Pair[first=age, second=36.5]
Optional[9] Optional[pear] Optional.empty
time compile at checked are generics
```

The bound `T extends Comparable<? super T>` (rather than `Comparable<T>`) lets `max` work for types that inherit their `compareTo` from a supertype — `java.sql.Timestamp` compares as a `java.util.Date`, for example.

<!-- expanded -->
### Going deeper: bounded type parameters and generic methods

`<T extends Comparable<T>>` restricts `T` to types that can be compared, which lets the method call `compareTo`. Static generic methods declare their own type parameters before the return type; the compiler infers them from the arguments, and they can be given explicitly (`Util.<String>first(list)`) when inference needs help.

```java
// file: Bounded.java
import java.util.List;

public class Bounded {
    static <T extends Comparable<? super T>> T max(List<? extends T> xs) {
        if (xs.isEmpty()) throw new IllegalArgumentException("empty list");
        T best = xs.get(0);
        for (T x : xs) if (x.compareTo(best) > 0) best = x;
        return best;
    }

    static <K, V extends Number> double sumValues(java.util.Map<K, V> m) {
        double s = 0;
        for (V v : m.values()) s += v.doubleValue();         // Number's method is available
        return s;
    }

    public static void main(String[] args) {
        System.out.println(max(List.of(3, 9, 4)) + " " + max(List.of("pear", "apple", "zebra")));
        System.out.println(sumValues(java.util.Map.of("a", 1, "b", 2.5)));
        System.out.println(Bounded.<Integer>max(List.of(1, 2)));
        // max(List.of(new Object()));                        // error: Object is not Comparable
    }
}
```

```console
$ java Bounded.java
9 zebra
3.5
2
```

The bound `Comparable<? super T>` (rather than `Comparable<T>`) accepts types whose `compareTo` is inherited from a superclass — `java.sql.Timestamp` compares as a `java.util.Date`, for example. It is the signature `Collections.max` uses.

## Unit 5.2 — Type erasure

**One-line summary:** Generics exist only at compile time. The compiler checks types, then **erases** them: `List<String>` and `List<Integer>` are the same class at run time (`List`), type parameters become their bound (`Object`, or `Comparable` for `<T extends Comparable<T>>`), and casts are inserted where values come out. Consequences: no `new T()`, no `new T[n]`, no `instanceof List<String>`, no overloading on `List<String>` versus `List<Integer>`, and no generic types with primitives (`List<int>`).

```java
// file: Erasure.java
import java.util.ArrayList;
import java.util.List;

public class Erasure {
    interface Shape extends Comparable<Shape> { double area(); }

    record Square(double side) implements Shape {
        public double area() { return side * side; }
        public int compareTo(Shape o) { return Double.compare(area(), o.area()); }   // erased signature differs
    }

    @SuppressWarnings({"unchecked", "rawtypes"})
    public static void main(String[] args) throws Exception {
        List<String> strings = new ArrayList<>();
        List<Integer> ints = new ArrayList<>();
        System.out.println("same class at run time: " + (strings.getClass() == ints.getClass()) + " (" + strings.getClass().getName() + ")");

        List raw = strings;                     // a raw type: generics switched off
        raw.add(42);                            // compiles (with a warning): heap pollution
        System.out.println("list now: " + strings);
        try {
            String s = strings.get(0);          // the compiler-inserted cast fails HERE, far from the bug
            System.out.println(s);
        } catch (ClassCastException e) {
            System.out.println("ClassCastException at the read: " + e.getMessage());
        }

        for (var m : Square.class.getDeclaredMethods()) {
            if (m.getName().equals("compareTo")) {
                System.out.println("compareTo(" + m.getParameterTypes()[0].getSimpleName() + ")" + (m.isBridge() ? "  <- bridge method" : ""));
            }
        }
    }
}
```

```console
$ java Erasure.java
same class at run time: true (java.util.ArrayList)
list now: [42]
ClassCastException at the read: class java.lang.Integer cannot be cast to class java.lang.String (java.lang.Integer and java.lang.String are in module java.base of loader 'bootstrap')
compareTo(Object)  <- bridge method
compareTo(Shape)
$ javac -Xlint:all -d /tmp Erasure.java 2>&1 | head -n 1; sed 's/    @SuppressWarnings({"unchecked", "rawtypes"})//' Erasure.java > E2.java && sed -i 's/class Erasure/class E2/' E2.java && javac -Xlint:all -d /tmp E2.java 2>&1 | grep warning
E2.java:19: warning: [rawtypes] found raw type: List
E2.java:20: warning: [unchecked] unchecked call to add(E) as a member of the raw type List
        raw.add(42);                            // compiles (with a warning): heap pollution
2 warnings
```

### Deep & Tricky

**DT-18. Heap pollution: through raw types or unchecked casts, a `List<String>` can hold an `Integer`, and the error surfaces only when — and where — the value is read.** *(Guarantee: JLS §4.12.2; erasure)* The JVM never checks generic types; the compiler inserts a cast at each read. So the `ClassCastException` appears in code that did nothing wrong, possibly much later and in another module, while the actual bug — the raw-type `add` — compiled with only a warning. Treat unchecked warnings as errors (`-Xlint:all -Werror`), never use raw types in new code, and use `Collections.checkedList(list, String.class)` when passing a collection to legacy code you do not trust: it checks every insertion at run time.

**DT-19. The compiler generates *bridge methods* so that erased generic overrides still override.** *(Implementation behaviour, required by JLS §15.12.4.5)* `Square.compareTo(Shape)` must override `Comparable.compareTo(Object)` after erasure, so `javac` adds a synthetic `compareTo(Object)` that casts its argument and calls `compareTo(Shape)`. The reflection output above shows both. Bridges are invisible in source but appear in stack traces, reflection, and `javap -p`, and are the reason a class cannot declare both `compareTo(Shape)` and `compareTo(Object)` of its own.

## Topic 5B — Variance

## Unit 5.3 — Wildcards and PECS

**One-line summary:** Generic types are **invariant**: `List<Integer>` is not a `List<Number>`, even though `Integer` is a `Number` — otherwise you could add a `Double` to a list of integers. **Wildcards** recover flexibility: `List<? extends Number>` is a list of *some* subtype of `Number` — you can read `Number`s from it but not add; `List<? super Integer>` is a list of *some* supertype of `Integer` — you can add `Integer`s but read only `Object`s. The rule: **Producer `extends`, Consumer `super`** (PECS).

```java
// file: Variance.java
import java.util.ArrayList;
import java.util.List;

public class Variance {
    static double sum(List<? extends Number> numbers) {          // a producer of Numbers
        double total = 0;
        for (Number n : numbers) total += n.doubleValue();
        // numbers.add(1);                                       // compile error: capture of ? extends Number
        return total;
    }

    static void fillWithSquares(List<? super Integer> sink, int n) {   // a consumer of Integers
        for (int i = 1; i <= n; i++) sink.add(i * i);
    }

    static <T> void copy(List<? super T> dst, List<? extends T> src) {  // the signature of Collections.copy
        dst.addAll(src);
    }

    public static void main(String[] args) {
        List<Integer> ints = List.of(1, 2, 3);
        List<Double> doubles = List.of(0.5, 1.5);
        System.out.println("sum(ints) = " + sum(ints) + ", sum(doubles) = " + sum(doubles));

        List<Number> numbers = new ArrayList<>();
        List<Object> objects = new ArrayList<>();
        fillWithSquares(numbers, 3);
        fillWithSquares(objects, 2);
        copy(objects, doubles);
        System.out.println("numbers " + numbers + ", objects " + objects);

        Object[] array = new Integer[2];                         // arrays ARE covariant...
        try {
            array[0] = "a string";                               // ...so this is checked at run time
        } catch (ArrayStoreException e) {
            System.out.println("ArrayStoreException: " + e.getMessage());
        }
    }
}
```

```console
$ java Variance.java
sum(ints) = 6.0, sum(doubles) = 2.0
numbers [1, 4, 9], objects [1, 4, 0.5, 1.5]
ArrayStoreException: java.lang.String
```

### Deep & Tricky

**DT-20. Arrays are covariant and checked at run time; generics are invariant and checked at compile time — so they do not mix.** *(Guarantee: JLS §10.5, §4.10)* `Integer[]` is a subtype of `Object[]`, and every store into an array is checked against its real element type (`ArrayStoreException`). Generic types carry no run-time element type to check against (erasure), so Java forbids creating arrays of non-reifiable types: `new List<String>[10]` does not compile. The generic varargs method `static <T> List<T> listOf(T... xs)` creates exactly such an array behind the scenes — hence the "possible heap pollution" warning, which `@SafeVarargs` suppresses for methods that only *read* the array.

### Pattern

**P-5. In APIs, use wildcards in parameters (PECS) and never in return types; prefer `List<E>` over arrays of generics.** *(Practice; Effective Java, Items 28–31)* A parameter type `Collection<? extends E>` accepts more callers than `Collection<E>` at no cost. A wildcard in a *return* type forces every caller to deal with it. If a type parameter appears only once in a signature, replace it with a wildcard (`void swap(List<?> list, …)`). *Trade-off:* wildcard-heavy signatures are harder to read; document the intent, and keep the simplest form that callers need.

## Unit 5.4 — Working around erasure: class tokens and type tokens

**One-line summary:** When code needs a type at run time — to create instances, cast safely, or deserialize — pass a **class token** (`Class<T>`), which carries `T` into the method: `type.cast(obj)`, `type.isInstance(obj)`, `type.getDeclaredConstructor().newInstance()`. A `Class` cannot represent `List<String>`, though; for full generic types, a **super type token** (an anonymous subclass of a generic class) captures the type in its `getGenericSuperclass()`, which reflection *can* read — the technique used by Jackson's `TypeReference` and Guice's `TypeLiteral`.

```java
// file: Tokens.java
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;

public class Tokens {
    static <T> T create(Class<T> type) throws ReflectiveOperationException {
        return type.getDeclaredConstructor().newInstance();       // `new T()` is impossible; this works
    }

    static <T> List<T> onlyOfType(List<?> mixed, Class<T> type) {
        return mixed.stream().filter(type::isInstance).map(type::cast).toList();
    }

    /** A super type token: subclassing it records the full generic type. */
    abstract static class TypeRef<T> {
        final Type type;
        protected TypeRef() {
            this.type = ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0];
        }
    }

    public static void main(String[] args) throws Exception {
        StringBuilder sb = create(StringBuilder.class).append("made by reflection");
        System.out.println(sb);
        List<Object> mixed = List.of(1, "two", 3.0, "four", 5);
        System.out.println(onlyOfType(mixed, String.class) + " " + onlyOfType(mixed, Integer.class));
        Type t = new TypeRef<Map<String, List<Integer>>>() {}.type;   // note the {}: an anonymous subclass
        System.out.println("captured at run time: " + t.getTypeName());
    }
}
```

```console
$ java Tokens.java
made by reflection
[two, four] [1, 5]
captured at run time: java.util.Map<java.lang.String, java.util.List<java.lang.Integer>>
```

<!-- expanded -->
### Deep & Tricky

**DT-21. Raw types turn off generic checking: assigning a `List<String>` to a raw `List`, adding an `Integer`, and reading it back as a `String` compiles with only an "unchecked" warning and fails with `ClassCastException` far from the bad `add` — at the line that reads the element. This is called *heap pollution*.** *(Guarantee: JLS §4.12.2; the compiler's unchecked warnings, shown with `-Xlint:unchecked`, mark every place it can happen)* Never use raw types in new code, and treat unchecked warnings as errors.

```java
// file: Pollution.java
import java.util.ArrayList;
import java.util.List;

public class Pollution {
    @SuppressWarnings({"rawtypes", "unchecked"})
    static void legacyAdd(List list) { list.add(42); }       // old pre-generics code

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("ada");
        legacyAdd(names);                                      // no error: the list now holds an Integer
        System.out.println("size " + names.size());
        try {
            for (String n : names) System.out.println(n.toUpperCase());
        } catch (ClassCastException e) {
            System.out.println("ClassCastException at the read, not at the add");
        }
    }
}
```

```console
$ java Pollution.java
size 2
ADA
ClassCastException at the read, not at the add
$ sed 's/@SuppressWarnings({"rawtypes", "unchecked"})//; s/class Pollution/class Pollution2/' Pollution.java > Pollution2.java && javac -Xlint:all Pollution2.java 2>&1 | grep -oE "warning: \[[a-z]+\]" | sort | uniq -c
      1 warning: [rawtypes]
      1 warning: [unchecked]
```

`Collections.checkedList(names, String.class)` wraps a list so that the bad `add` itself throws — useful when passing collections to legacy code.

### Mastery check

1. **Predict.** Does `List<Object> objs = new ArrayList<String>();` compile? Does `List<? extends Object> objs = new ArrayList<String>();`?
2. **Predict.** Can a class contain both `void f(List<String> x)` and `void f(List<Integer> x)`?

## Chapter 5 Project — A type-safe configuration registry

**Goal:** a configuration registry in which each setting is a typed key — `Key<Integer> PORT`, `Key<Duration> TIMEOUT`, `Key<List<String>> HOSTS` — so that `config.get(PORT)` returns an `int`-compatible `Integer` with no cast at the call site, parsing is done by type-specific parsers, defaults and validation are part of the key, and errors report the key and the raw text. It is a **typesafe heterogeneous container**: one map holding values of different types, with the type relationship enforced by the API.

```java
// file: config/Key.java
import java.util.Objects;
import java.util.function.Function;
import java.util.function.Predicate;

/** A typed configuration key: name, parser, default, and validation. */
public final class Key<T> {
    private final String name;
    private final Function<String, ? extends T> parser;
    private final T defaultValue;
    private final Predicate<? super T> valid;
    private final String rule;

    private Key(String name, Function<String, ? extends T> parser, T defaultValue, Predicate<? super T> valid, String rule) {
        this.name = Objects.requireNonNull(name);
        this.parser = Objects.requireNonNull(parser);
        this.defaultValue = defaultValue;
        this.valid = valid;
        this.rule = rule;
    }

    public static <T> Key<T> of(String name, Function<String, ? extends T> parser, T defaultValue) {
        return new Key<>(name, parser, defaultValue, x -> true, "");
    }

    /** Returns a key with an extra validity rule; the predicate may accept any supertype of T (PECS). */
    public Key<T> requiring(Predicate<? super T> rule, String description) {
        Predicate<T> both = x -> valid.test(x) && rule.test(x);      // both accept a T
        return new Key<>(name, parser, defaultValue, both, description);
    }

    public String name() { return name; }
    T defaultValue() { return defaultValue; }

    T parse(String raw) {
        T value = parser.apply(raw.strip());
        if (!valid.test(value)) throw new IllegalArgumentException("must be " + rule);
        return value;
    }

    @Override public String toString() { return "Key(" + name + ")"; }
}
```

```java
// file: config/Config.java
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/** A typesafe heterogeneous container of settings. */
public final class Config {
    private final Map<Key<?>, Object> values = new HashMap<>();   // the API guarantees value matches key type

    public record Problem(String key, String raw, String message) {}

    public static final class Builder {
        private final Map<String, Key<?>> known = new HashMap<>();
        Builder register(Key<?>... keys) {
            for (Key<?> k : keys) known.put(k.name(), k);
            return this;
        }
        Config load(Map<String, String> raw, List<Problem> problems) {
            Config c = new Config();
            raw.forEach((name, text) -> {
                Key<?> key = known.get(name);
                if (key == null) {
                    problems.add(new Problem(name, text, "unknown key"));
                    return;
                }
                try {
                    c.putParsed(key, text);
                } catch (RuntimeException e) {
                    problems.add(new Problem(name, text, e.getClass().getSimpleName() + ": " + e.getMessage()));
                }
            });
            return c;
        }
    }

    private <T> void putParsed(Key<T> key, String text) {        // a helper method captures the wildcard as T
        values.put(key, key.parse(text));
    }

    /** No cast needed by callers: the key's type parameter says what comes back. */
    public <T> T get(Key<T> key) {
        @SuppressWarnings("unchecked")                            // safe: only putParsed stores values, always as T
        T v = (T) values.get(key);
        return v != null ? v : key.defaultValue();
    }

    public Map<String, Object> snapshot(Key<?>... keys) {
        Map<String, Object> m = new java.util.TreeMap<>();
        for (Key<?> k : keys) m.put(k.name(), get(k));
        return Collections.unmodifiableMap(m);
    }

    static List<String> csv(String s) {
        List<String> out = new ArrayList<>();
        for (String part : s.split(",")) if (!part.isBlank()) out.add(part.strip());
        return List.copyOf(out);
    }
}
```

```java
// file: config/Main.java
import java.time.Duration;
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public class Main {
    static final Key<Integer> PORT = Key.of("server.port", Integer::valueOf, 8080).requiring(p -> p >= 1 && p <= 65535, "in 1..65535");
    static final Key<Duration> TIMEOUT = Key.of("server.timeout", Duration::parse, Duration.ofSeconds(30))
            .requiring(d -> !d.isNegative() && !d.isZero(), "positive");
    static final Key<List<String>> HOSTS = Key.of("cluster.hosts", Config::csv, List.of("localhost"))
            .requiring(l -> !l.isEmpty(), "non-empty");
    static final Key<Boolean> DEBUG = Key.of("debug", Boolean::parseBoolean, false);
    static final Key<Number> RATIO = Key.<Number>of("cache.ratio", Double::valueOf, 0.5)       // a Key<Number> parsed as Double
            .requiring((Object o) -> o != null, "present");                                     // Predicate<Object> accepted: ? super T

    public static void main(String[] args) {
        Map<String, String> file = new LinkedHashMap<>();
        file.put("server.port", " 9443 ");
        file.put("server.timeout", "PT2.5S");
        file.put("cluster.hosts", "db1, db2,,db3");
        file.put("cache.ratio", "0.75");
        List<Config.Problem> problems = new ArrayList<>();
        Config ok = new Config.Builder().register(PORT, TIMEOUT, HOSTS, DEBUG, RATIO).load(file, problems);

        int port = ok.get(PORT);                                  // no casts anywhere at the call site
        Duration timeout = ok.get(TIMEOUT);
        List<String> hosts = ok.get(HOSTS);
        System.out.println("port " + port + ", timeout " + timeout.toMillis() + " ms, hosts " + hosts + ", debug " + ok.get(DEBUG)
                + ", ratio " + ok.get(RATIO) + " problems " + problems);
        System.out.println(ok.snapshot(PORT, TIMEOUT, HOSTS, DEBUG, RATIO));

        Map<String, String> bad = Map.of("server.port", "70000", "server.timeout", "5 seconds", "cluster.hosts", " , ", "colour", "blue");
        List<Config.Problem> errs = new ArrayList<>();
        Config c = new Config.Builder().register(PORT, TIMEOUT, HOSTS, DEBUG).load(bad, errs);
        errs.stream().sorted(java.util.Comparator.comparing(Config.Problem::key))
            .forEach(p -> System.out.println("  " + p.key() + " = '" + p.raw() + "': " + p.message()));
        System.out.println("defaults still apply where values were rejected: port " + c.get(PORT) + ", hosts " + c.get(HOSTS));
    }
}
```

```console
$ cd config && rm -rf out && javac -Xlint:all -Werror -d out *.java && java -cp out Main
port 9443, timeout 2500 ms, hosts [db1, db2, db3], debug false, ratio 0.75 problems []
{cache.ratio=0.75, cluster.hosts=[db1, db2, db3], debug=false, server.port=9443, server.timeout=PT2.5S}
  cluster.hosts = ' , ': IllegalArgumentException: must be non-empty
  colour = 'blue': unknown key
  server.port = '70000': IllegalArgumentException: must be in 1..65535
  server.timeout = '5 seconds': DateTimeParseException: Text cannot be parsed to a Duration
defaults still apply where values were rejected: port 8080, hosts [localhost]
```

The generics carry the design:

- **`Key<T>` ties a name to a type**, so `get(Key<T>) : T` needs no cast at the call site and a mismatch (`String s = config.get(PORT);`) is a compile error. The one unchecked cast is inside `Config.get`, justified by the invariant that only `putParsed` stores values — and `putParsed` is a generic method that **captures** the wildcard `Key<?>` as a named `T`, so the compiler checks that the parsed value matches its key.
- **PECS in the API:** the parser is a `Function<String, ? extends T>` (a producer of `T`s: `Double::valueOf` can serve a `Key<Number>`), the validity rule is a `Predicate<? super T>` (a consumer: a `Predicate<Object>` works for any key).
- **Erasure is respected:** nothing needs `T` at run time, because each key carries its own parser function.
- **A capture error on the way.** The first version of `requiring` combined the rules with `valid.and(rule)`, and did not compile: `valid` is a `Predicate<? super T>` and `rule` is another `Predicate<? super T>`, but the two `?` are *different unknown types* (the compiler calls them `CAP#1` and `CAP#2`), so one cannot be passed to the other's `and`. Writing the combination as a lambda over `T` — which both predicates accept — fixed it.
- The build uses `-Xlint:all -Werror`, so the single justified `@SuppressWarnings("unchecked")` is the only unchecked operation in the program.

### Review checklist

- Where would a `ClassCastException` appear if `Config` exposed a `put(Key<?> key, Object value)` method? (DT-18.)
- Why is `Key` final with a private constructor? What could a subclass break?
- Add a `Key<Path>` that must point to an existing directory. Which parts change?

## Answers to the Chapter 5 prediction questions

**5.4 Q1.** No — generics are invariant, so `ArrayList<String>` is not a `List<Object>` (otherwise one could add an `Integer` to a list of strings). Yes for `List<? extends Object>` — but then nothing except `null` can be added through that reference.

**5.4 Q2.** No. Both erase to `f(List)`, a "name clash: … have the same erasure" compile error.

# Chapter 6 — The Collections Framework

*Goal: choose and use Java's collections well — lists, sets, maps, queues, and deques; how `HashMap` and `ArrayList` work inside; ordering with `Comparable` and `Comparator`; immutable collections; and the failure modes: `ConcurrentModificationException`, broken keys, and accidental quadratic time.*

**What you will be able to do after this chapter**

- Pick the right interface (`List`, `Set`, `Map`, `Queue`, `Deque`) and implementation for a job.
- Explain how `ArrayList`, `LinkedList`, `HashMap`, `TreeMap`, and `PriorityQueue` work and what each operation costs.
- Sort with `Comparable` and composed `Comparator`s.
- Create unmodifiable collections, and tell them apart from unmodifiable *views*.
- Iterate and modify safely.

## Topic 6A — The core collections

## Unit 6.1 — The map of the framework

**One-line summary:** Program to the **interfaces**: `Collection` → `List` (ordered, indexed), `Set` (no duplicates), `Queue`/`Deque` (ends); and `Map` (key → value), which is not a `Collection`. Choose the implementation by the operations you need.

| Need | Use | Costs |
| --- | --- | --- |
| Indexed list, append | `ArrayList` | get O(1), append amortised O(1), insert/remove in middle O(n) |
| Stack or queue | `ArrayDeque` | push/pop/offer/poll at both ends O(1) |
| Unique elements, fast lookup | `HashSet` | add/contains O(1) average; no order |
| Unique, insertion order | `LinkedHashSet` | as `HashSet`, iteration in insertion order |
| Unique, sorted | `TreeSet` | O(log n); `first`, `ceiling`, `subSet` |
| Key → value | `HashMap` | get/put O(1) average |
| Key → value, insertion or access order | `LinkedHashMap` | basis of LRU caches |
| Key → value, sorted keys, range queries | `TreeMap` | O(log n); `floorKey`, `headMap` |
| Priority queue | `PriorityQueue` | offer/poll O(log n), peek O(1) |
| Enum keys | `EnumMap`, `EnumSet` | array / bit-set backed |

`Vector`, `Stack`, and `Hashtable` are legacy synchronised classes; do not use them in new code. `LinkedList` is almost never the right choice — measured below.

## Unit 6.2 — Lists: `ArrayList` versus `LinkedList`

**One-line summary:** `ArrayList` is a resizable array: contiguous references, grows by 50% when full. `LinkedList` is a doubly linked list: one node object per element. `LinkedList` wins in textbooks for insertion in the middle; in practice it loses almost everywhere, because reaching the middle is O(n) pointer-chasing and each node is a separate allocation.

```java
// file: Lists.java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.ListIterator;

public class Lists {
    static long time(Runnable r) {
        long t = System.nanoTime();
        r.run();
        return (System.nanoTime() - t) / 1_000_000;
    }

    static void fill(List<Integer> l, int n) { for (int i = 0; i < n; i++) l.add(i); }

    static long sum(List<Integer> l) { long s = 0; for (int x : l) s += x; return s; }

    static void insertInMiddle(List<Integer> l, int n) { for (int i = 0; i < n; i++) l.add(l.size() / 2, i); }

    static void insertWithIterator(List<Integer> l, int n) {       // LinkedList's one real strength
        ListIterator<Integer> it = l.listIterator(l.size() / 2);
        for (int i = 0; i < n; i++) it.add(i);
    }

    public static void main(String[] args) {
        for (int round = 0; round < 2; round++) {                   // round 0 warms up the JIT
            int n = 3_000_000;
            List<Integer> a = new ArrayList<>(), l = new LinkedList<>();
            long fa = time(() -> fill(a, n)), fl = time(() -> fill(l, n));
            long sa = time(() -> sum(a)), sl = time(() -> sum(l));
            List<Integer> a2 = new ArrayList<>(), l2 = new LinkedList<>();
            fill(a2, 100_000); fill(l2, 100_000);
            long ma = time(() -> insertInMiddle(a2, 20_000)), ml = time(() -> insertInMiddle(l2, 20_000));
            List<Integer> l3 = new LinkedList<>(); fill(l3, 100_000);
            long il = time(() -> insertWithIterator(l3, 20_000));
            if (round == 1) {
                System.out.printf("append 3M:              ArrayList %4d ms, LinkedList %4d ms%n", fa, fl);
                System.out.printf("iterate 3M:             ArrayList %4d ms, LinkedList %4d ms%n", sa, sl);
                System.out.printf("20k inserts at middle:  ArrayList %4d ms, LinkedList %4d ms (via index), %d ms (via ListIterator)%n", ma, ml, il);
            }
        }
    }
}
```

```console
$ java -Xmx2g Lists.java
append 3M:              ArrayList  130 ms, LinkedList  249 ms
iterate 3M:             ArrayList   13 ms, LinkedList   22 ms
20k inserts at middle:  ArrayList  212 ms, LinkedList 2340 ms (via index), 3 ms (via ListIterator)
```

The results are less one-sided than folklore says, and worth reading carefully. Appending and iterating took about the same time for both lists in this run: the `LinkedList`'s nodes were allocated one after another, so they sit next to each other in memory and iteration is still sequential, and both loops are dominated by unboxing `Integer` objects. (A list built up over time, with nodes scattered across the heap, iterates much more slowly; `LinkedList` also uses several times the memory — a 24-byte node per element on top of the element itself.) The decisive line is the third: inserting in the middle **by index** was about 35 times slower with `LinkedList`, because `add(index, x)` must first walk to the index — O(n) pointer-chasing — while `ArrayList` shifts its tail with one fast `arraycopy`. `LinkedList` is only fast when you already hold a `ListIterator` at the position, which real code rarely does. Default to `ArrayList`; use `ArrayDeque` for queues and stacks.

<!-- expanded -->
### Going deeper: `ArrayDeque` for stacks and queues

`Stack` (a synchronised subclass of `Vector`) and using `LinkedList` as a queue are both legacy habits. `ArrayDeque` is a resizable circular array that supports adding and removing at both ends in amortised constant time; it is the standard choice for stacks, queues, and breadth-first searches. `PriorityQueue` is a binary heap: `poll()` always returns the smallest element (by natural order or a `Comparator`).

```java
// file: Deques.java
import java.util.ArrayDeque;
import java.util.Comparator;
import java.util.Deque;
import java.util.PriorityQueue;

public class Deques {
    record Task(String name, int priority) {}

    public static void main(String[] args) {
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(1); stack.push(2); stack.push(3);
        System.out.print("stack pops: ");
        while (!stack.isEmpty()) System.out.print(stack.pop() + " ");

        Deque<String> queue = new ArrayDeque<>();
        queue.offer("a"); queue.offer("b"); queue.offer("c");
        System.out.print("\nqueue polls: ");
        while (!queue.isEmpty()) System.out.print(queue.poll() + " ");

        PriorityQueue<Task> pq = new PriorityQueue<>(Comparator.comparingInt(Task::priority).reversed());
        pq.add(new Task("email", 1)); pq.add(new Task("outage", 9)); pq.add(new Task("review", 5));
        System.out.print("\nby priority: ");
        while (!pq.isEmpty()) System.out.print(pq.poll().name() + " ");
        System.out.println();
        try { new ArrayDeque<String>().offer(null); } catch (NullPointerException e) { System.out.println("ArrayDeque rejects null"); }
    }
}
```

```console
$ java Deques.java
stack pops: 3 2 1 
queue polls: a b c 
by priority: outage review email 
ArrayDeque rejects null
```

Iterating a `PriorityQueue` (or printing it) does *not* give sorted order — only repeated `poll()` does, because the heap is only partially ordered.

## Unit 6.3 — How `HashMap` works

**One-line summary:** A `HashMap` is an array of **buckets**. `put(k, v)` computes `k.hashCode()`, mixes its bits, and uses the low bits as the bucket index; colliding entries form a linked list in the bucket, which becomes a **red-black tree** when it grows past 8 entries (if keys are `Comparable`, lookups in a bad bucket stay O(log n)). When the number of entries exceeds capacity × load factor (0.75), the table doubles and entries are redistributed. Iteration order is unspecified and can change after a resize.

```java
// file: HashMapInside.java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.TreeMap;

public class HashMapInside {
    record BadKey(int id) {
        @Override public int hashCode() { return 1; }               // every key in one bucket
    }
    record BadComparableKey(int id) implements Comparable<BadComparableKey> {
        @Override public int hashCode() { return 1; }
        @Override public int compareTo(BadComparableKey o) { return Integer.compare(id, o.id); }
    }
    record GoodKey(int id) {}

    static long timeLookups(Map<Object, Integer> m, java.util.function.IntFunction<Object> key, int n) {
        for (int i = 0; i < n; i++) m.put(key.apply(i), i);
        long t = System.nanoTime();
        long sum = 0;
        for (int i = 0; i < n; i++) sum += m.get(key.apply(i));
        return (System.nanoTime() - t) / 1_000_000;
    }

    public static void main(String[] args) {
        int n = 20_000;
        System.out.println("20k lookups, good hashCode:                 " + timeLookups(new HashMap<>(), GoodKey::new, n) + " ms");
        System.out.println("20k lookups, constant hashCode, Comparable: " + timeLookups(new HashMap<>(), BadComparableKey::new, n) + " ms (bucket became a tree)");
        System.out.println("20k lookups, constant hashCode:             " + timeLookups(new HashMap<>(), BadKey::new, n) + " ms (tree cannot order the keys)");

        Map<String, Integer> hash = new HashMap<>(), linked = new LinkedHashMap<>(), tree = new TreeMap<>();
        for (String w : "the quick brown fox jumps over the lazy dog".split(" ")) {
            for (Map<String, Integer> m : java.util.List.of(hash, linked, tree)) m.merge(w, 1, Integer::sum);
        }
        System.out.println("HashMap:       " + hash + "\nLinkedHashMap: " + linked + "\nTreeMap:       " + tree);
        System.out.println("getOrDefault: " + hash.getOrDefault("cat", 0) + ", computeIfAbsent: "
                + new HashMap<String, java.util.List<Integer>>().computeIfAbsent("k", k -> new java.util.ArrayList<>()));
    }
}
```

```console
$ java HashMapInside.java
20k lookups, good hashCode:                 7 ms
20k lookups, constant hashCode, Comparable: 30 ms (bucket became a tree)
20k lookups, constant hashCode:             1851 ms (tree cannot order the keys)
HashMap:       {over=1, the=2, quick=1, lazy=1, jumps=1, brown=1, dog=1, fox=1}
LinkedHashMap: {the=2, quick=1, brown=1, fox=1, jumps=1, over=1, lazy=1, dog=1}
TreeMap:       {brown=1, dog=1, fox=1, jumps=1, lazy=1, over=1, quick=1, the=2}
getOrDefault: 0, computeIfAbsent: []
```

### Deep & Tricky

**DT-22. A poor `hashCode` turns a `HashMap` into a list — or, since Java 8, into a tree if the keys are `Comparable`.** *(Implementation behaviour: `HashMap` treeification, JEP 180)* With every key in one bucket, each operation scans the bucket. Treeification limits the damage to O(log n) per operation when keys can be ordered, which was added specifically to blunt **hash-collision denial-of-service attacks** (an attacker sending many `String` keys with equal hash codes). Keys that are not `Comparable` still degrade badly. Write good `hashCode` methods (records and `Objects.hash` do), and prefer `String`, boxed numbers, enums, and records as keys.

**DT-23. Modifying a collection while iterating over it throws `ConcurrentModificationException` — on a best-effort basis, even in a single thread.** *(Library behaviour: fail-fast iterators)* Each collection counts structural modifications; an iterator checks the count on `next()`. Removing inside a for-each loop therefore throws (or, for the second-to-last element, silently skips the last one, because `hasNext()` returns false first). Use `Iterator.remove()`, `removeIf(predicate)`, or collect changes and apply them afterwards. The check is not a thread-safety mechanism; concurrent access needs concurrent collections (Chapter 10).

```java
// file: Cme.java
import java.util.ArrayList;
import java.util.List;

public class Cme {
    public static void main(String[] args) {
        List<Integer> a = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
        try {
            for (Integer x : a) if (x % 2 == 0) a.remove(x);
        } catch (java.util.ConcurrentModificationException e) {
            System.out.println("for-each + remove: ConcurrentModificationException, list left as " + a);
        }
        List<Integer> b = new ArrayList<>(List.of(1, 2, 3, 4));
        for (Integer x : b) if (x == 3) b.remove(x);              // second-to-last: no exception, 4 is skipped
        System.out.println("removing the second-to-last element silently skipped the last: " + b);
        List<Integer> c = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
        c.removeIf(x -> x % 2 == 0);
        System.out.println("removeIf: " + c);
        List<Integer> d = new ArrayList<>(List.of(10, 20, 30));
        d.remove(1);                                              // remove(int index)!
        d.remove(Integer.valueOf(10));                            // remove(Object)
        System.out.println("remove(1) removed index 1, remove(Integer.valueOf(10)) removed the value: " + d);
    }
}
```

```console
$ java Cme.java
for-each + remove: ConcurrentModificationException, list left as [1, 3, 4, 5, 6]
removing the second-to-last element silently skipped the last: [1, 2, 4]
removeIf: [1, 3, 5]
remove(1) removed index 1, remove(Integer.valueOf(10)) removed the value: [30]
```

## Topic 6B — Order and immutability

## Unit 6.4 — `Comparable` and `Comparator`

**One-line summary:** A class with one natural order implements `Comparable<T>` (`compareTo`). Any other order is a `Comparator<T>`, built by composition: `Comparator.comparing(Person::lastName).thenComparing(Person::firstName).reversed()`, with `nullsFirst`, `comparingInt`, and `Comparator.naturalOrder()`. Sorting (`List.sort`, `Collections.sort`, `Arrays.sort` for objects) is a **stable** merge sort (TimSort). Never write a comparator as `a - b`: it overflows.

```java
// file: Sorting.java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.PriorityQueue;
import java.util.TreeMap;

public class Sorting {
    record Employee(String name, String dept, int salary, Integer bonus) {}

    public static void main(String[] args) {
        List<Employee> staff = new ArrayList<>(List.of(
            new Employee("Mia", "eng", 120, 10), new Employee("Raj", "ops", 90, null), new Employee("Zoe", "eng", 120, 5),
            new Employee("Ben", "ops", 95, 7), new Employee("Ann", "eng", 150, null)));
        staff.sort(Comparator.comparing(Employee::dept).thenComparing(Employee::salary, Comparator.reverseOrder()).thenComparing(Employee::name));
        staff.forEach(e -> System.out.print(e.dept() + "/" + e.salary() + "/" + e.name() + "  "));
        System.out.println();
        staff.sort(Comparator.comparing(Employee::bonus, Comparator.nullsLast(Comparator.naturalOrder())));
        System.out.println("by bonus, nulls last: " + staff.stream().map(e -> e.name() + "=" + e.bonus()).toList());

        Comparator<Integer> broken = (a, b) -> a - b;              // overflows for distant values
        System.out.println("a - b comparator says MIN_VALUE vs 1: " + broken.compare(Integer.MIN_VALUE, 1) + " (should be negative); Integer.compare: "
                + Integer.compare(Integer.MIN_VALUE, 1));

        PriorityQueue<Employee> bySalary = new PriorityQueue<>(Comparator.comparingInt(Employee::salary).reversed());
        bySalary.addAll(staff);
        System.out.println("top 2 earners: " + bySalary.poll().name() + ", " + bySalary.poll().name());
        TreeMap<Integer, String> grades = new TreeMap<>(java.util.Map.of(90, "A", 80, "B", 70, "C", 0, "F"));
        System.out.println("score 85 -> " + grades.floorEntry(85).getValue() + ", scores below 80: " + grades.headMap(80));
    }
}
```

```console
$ java Sorting.java
eng/150/Ann  eng/120/Mia  eng/120/Zoe  ops/95/Ben  ops/90/Raj  
by bonus, nulls last: [Zoe=5, Ben=7, Mia=10, Ann=null, Raj=null]
a - b comparator says MIN_VALUE vs 1: 2147483647 (should be negative); Integer.compare: -1
top 2 earners: Ann, Zoe
score 85 -> B, scores below 80: {0=F, 70=C}
```

<!-- expanded -->
### Going deeper: building comparators

`Comparator.comparing(keyExtractor)` builds a comparator from a key; `thenComparing` adds tie-breakers; `reversed()` flips one; `nullsFirst`/`nullsLast` decide where `null` goes. Writing `(a, b) -> a.x - b.x` by hand is a known bug: subtraction overflows for large or negative values and gives the wrong sign.

```java
// file: Comparators.java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class Comparators {
    record Employee(String name, String dept, int salary) {}

    public static void main(String[] args) {
        List<Employee> staff = new ArrayList<>(List.of(
            new Employee("ravi", "eng", 90), new Employee("ada", "eng", 120),
            new Employee("lin", "ops", 90), new Employee("bo", "eng", 90)));
        staff.sort(Comparator.comparing(Employee::dept)
                             .thenComparing(Employee::salary, Comparator.reverseOrder())
                             .thenComparing(Employee::name));
        staff.forEach(e -> System.out.println(e.dept() + " " + e.salary() + " " + e.name()));

        Comparator<Integer> bySubtraction = (a, b) -> a - b;           // the classic bug
        int a = Integer.MIN_VALUE, b = 1;
        System.out.println("subtraction says MIN_VALUE > 1: " + (bySubtraction.compare(a, b) > 0)
                           + "; Integer.compare says: " + (Integer.compare(a, b) > 0));
        List<String> withNull = new ArrayList<>(java.util.Arrays.asList("b", null, "a"));
        withNull.sort(Comparator.nullsLast(Comparator.naturalOrder()));
        System.out.println(withNull);
    }
}
```

```console
$ java Comparators.java
eng 120 ada
eng 90 bo
eng 90 ravi
ops 90 lin
subtraction says MIN_VALUE > 1: true; Integer.compare says: false
[a, b, null]
```

## Unit 6.5 — Unmodifiable collections and views

**One-line summary:** `List.of`, `Set.of`, `Map.of`, and `List.copyOf` create **unmodifiable** collections: mutators throw `UnsupportedOperationException`, and `null` elements are rejected. `Collections.unmodifiableList(list)` is only a read-only **view**: changes to the underlying list show through. `Arrays.asList` is a fixed-size view of an array; `subList` is a view of a range.

```java
// file: Unmodifiable.java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Unmodifiable {
    public static void main(String[] args) {
        List<String> source = new ArrayList<>(List.of("a", "b"));
        List<String> view = Collections.unmodifiableList(source);
        List<String> copy = List.copyOf(source);
        source.add("c");
        System.out.println("view sees the change: " + view + ", copy does not: " + copy);

        String[] array = {"x", "y", "z"};
        List<String> asList = Arrays.asList(array);
        asList.set(0, "changed");                                   // writes through to the array
        System.out.println("array[0] = " + array[0]);
        for (Runnable r : new Runnable[] {() -> asList.add("w"), () -> copy.add("d"), () -> List.of(1, null)}) {
            try {
                r.run();
            } catch (UnsupportedOperationException | NullPointerException e) {
                System.out.println(e.getClass().getSimpleName());
            }
        }
        List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
        nums.subList(1, 4).clear();                                 // a view: clearing it removes the range
        System.out.println("after subList(1,4).clear(): " + nums);
    }
}
```

```console
$ java Unmodifiable.java
view sees the change: [a, b, c], copy does not: [a, b]
array[0] = changed
UnsupportedOperationException
UnsupportedOperationException
NullPointerException
after subList(1,4).clear(): [1, 5, 6]
```

### Pattern

**P-6. Declare variables and parameters by interface, return unmodifiable collections, and default to `ArrayList`, `HashMap`, `HashSet`, and `ArrayDeque`.** *(Practice; Effective Java, Item 64)* `List<String> names = new ArrayList<>()` lets the implementation change in one place. Returning `List.copyOf(internal)` (or an unmodifiable view, if callers should see updates) protects invariants (DT-9). The four defaults are the fastest general-purpose choices; switch to a `Tree*`, `Linked*`, or `PriorityQueue` only for the property it adds. *Trade-off:* defensive copies cost O(n); for large collections on hot paths return a view and document it.

<!-- expanded -->
### Deep & Tricky

**DT-24. `Arrays.asList` returns a fixed-size list *backed by the array*: `set` writes through to the array, `add` and `remove` throw `UnsupportedOperationException`. And `List.of(...)` is fully immutable *and* rejects `null` elements — even `contains(null)` throws `NullPointerException`.** *(Library behaviour: `Arrays.asList` and `List.of` Javadoc)* Wrap in `new ArrayList<>(...)` when you need a normal, growable list.

```java
// file: AsList.java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class AsList {
    public static void main(String[] args) {
        String[] arr = {"a", "b", "c"};
        List<String> view = Arrays.asList(arr);
        view.set(0, "z");
        System.out.println("array after set on the list: " + Arrays.toString(arr));
        try { view.add("d"); } catch (UnsupportedOperationException e) { System.out.println("add: UnsupportedOperationException"); }
        List<String> fixed = List.of("x", "y");
        try { fixed.contains(null); } catch (NullPointerException e) { System.out.println("List.of(...).contains(null): NullPointerException"); }
        List<String> growable = new ArrayList<>(view);
        growable.add("d");
        System.out.println("copy is growable: " + growable);
    }
}
```

```console
$ java AsList.java
array after set on the list: [z, b, c]
add: UnsupportedOperationException
List.of(...).contains(null): NullPointerException
copy is growable: [z, b, c, d]
```

### Mastery check

1. **Predict.** What does `new ArrayList<>(List.of(5, 6, 7)).remove(Integer.valueOf(1))` return, and what does `.remove(1)` return?
2. **Predict.** After `Map<String,Integer> m = new HashMap<>(); m.put("a", 1); m.merge("a", 5, Integer::sum); m.computeIfAbsent("b", k -> 10); m.putIfAbsent("a", 99);` what is `m`?

## Chapter 6 Project — An LRU cache and an inverted index

**Goal:** two classic collection-based components. (1) An **LRU cache** — fixed capacity, evicting the least recently used entry — written twice: in six lines with `LinkedHashMap`, and from scratch with a `HashMap` plus a hand-made doubly linked list, to show how the six-line version works. (2) An **inverted index** over documents — word → sorted set of document ids — answering AND queries by intersecting posting sets.

```java
// file: lru/Lru.java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class Lru {
    /** LRU in six lines: LinkedHashMap in access order, evicting in removeEldestEntry. */
    static class SimpleLru<K, V> extends LinkedHashMap<K, V> {
        private final int capacity;
        SimpleLru(int capacity) {
            super(16, 0.75f, true);                       // true = access order
            this.capacity = capacity;
        }
        @Override protected boolean removeEldestEntry(Map.Entry<K, V> eldest) { return size() > capacity; }
    }

    /** The same from scratch: a hash map for lookup plus a doubly linked list for recency. */
    static class HandLru<K, V> {
        private final class Node {
            K key; V value; Node prev, next;
        }
        private final Map<K, Node> map = new HashMap<>();
        private final Node head = new Node(), tail = new Node();   // sentinels: head.next is most recent
        private final int capacity;
        int evictions = 0;

        HandLru(int capacity) {
            this.capacity = capacity;
            head.next = tail;
            tail.prev = head;
        }

        V get(K key) {
            Node n = map.get(key);
            if (n == null) return null;
            unlink(n);
            linkFirst(n);                                   // O(1) move to front
            return n.value;
        }

        void put(K key, V value) {
            Node n = map.get(key);
            if (n == null) {
                if (map.size() == capacity) {               // evict the least recently used: tail.prev
                    Node lru = tail.prev;
                    unlink(lru);
                    map.remove(lru.key);
                    evictions++;
                }
                n = new Node();
                n.key = key;
                map.put(key, n);
            } else {
                unlink(n);
            }
            n.value = value;
            linkFirst(n);
        }

        private void unlink(Node n) { n.prev.next = n.next; n.next.prev = n.prev; }
        private void linkFirst(Node n) { n.next = head.next; n.prev = head; head.next.prev = n; head.next = n; }

        String order() {
            StringBuilder sb = new StringBuilder("[");
            for (Node n = tail.prev; n != head; n = n.prev) sb.append(n.key).append(n.prev != head ? ", " : "");
            return sb.append("]").toString();
        }
    }

    public static void main(String[] args) {
        SimpleLru<String, Integer> simple = new SimpleLru<>(3);
        HandLru<String, Integer> hand = new HandLru<>(3);
        String[] ops = {"put a", "put b", "put c", "get a", "put d", "get b", "put e", "get c", "put a"};
        for (String op : ops) {
            String key = op.substring(4);
            if (op.startsWith("put")) { simple.put(key, 1); hand.put(key, 1); }
            else { simple.get(key); hand.get(key); }
            System.out.printf("%-6s LinkedHashMap %-10s hand-made %s%n", op, simple.keySet(), hand.order());
        }
        // agreement on a long random workload
        SimpleLru<Integer, Integer> s2 = new SimpleLru<>(100);
        HandLru<Integer, Integer> h2 = new HandLru<>(100);
        java.util.Random rnd = new java.util.Random(7);
        int mismatches = 0;
        for (int i = 0; i < 200_000; i++) {
            int k = (int) Math.abs(rnd.nextGaussian() * 80);
            if (rnd.nextInt(3) == 0) { s2.put(k, i); h2.put(k, i); }
            else if (!java.util.Objects.equals(s2.get(k), h2.get(k))) mismatches++;
        }
        System.out.println("200,000 random operations: " + mismatches + " mismatches, " + h2.evictions + " evictions, same final order: "
                + s2.keySet().toString().equals(h2.order()));
    }
}
```

```java
// file: lru/Index.java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.TreeSet;

public class Index {
    private final Map<String, TreeSet<Integer>> postings = new HashMap<>();
    private final List<String> docs = new ArrayList<>();

    int add(String text) {
        int id = docs.size();
        docs.add(text);
        for (String w : text.toLowerCase().split("[^a-z0-9]+")) {
            if (!w.isEmpty()) postings.computeIfAbsent(w, k -> new TreeSet<>()).add(id);
        }
        return id;
    }

    /** Documents containing every word: intersect posting sets, smallest first. */
    List<Integer> search(String query) {
        List<TreeSet<Integer>> sets = new ArrayList<>();
        for (String w : query.toLowerCase().split("\\s+")) {
            TreeSet<Integer> s = postings.get(w);
            if (s == null) return List.of();                     // a word that appears nowhere
            sets.add(s);
        }
        sets.sort(Comparator.comparingInt(TreeSet::size));       // start from the rarest word
        TreeSet<Integer> result = new TreeSet<>(sets.get(0));
        for (int i = 1; i < sets.size(); i++) result.retainAll(sets.get(i));
        return List.copyOf(result);
    }

    public static void main(String[] args) {
        Index idx = new Index();
        for (String d : List.of("The quick brown fox", "A quick study of Java collections", "Brown bears and the quick river",
                "Java maps: HashMap, TreeMap, LinkedHashMap", "The fox studies Java")) {
            idx.add(d);
        }
        for (String q : List.of("quick", "quick brown", "java", "java fox", "the quick", "python")) {
            List<Integer> hits = idx.search(q);
            System.out.printf("%-12s -> %s %s%n", q, hits, hits.stream().map(i -> "\"" + idx.docs.get(i) + "\"").toList());
        }
        System.out.println("distinct words indexed: " + idx.postings.size());
    }
}
```

```console
$ cd lru && java Lru.java
put a  LinkedHashMap [a]        hand-made [a]
put b  LinkedHashMap [a, b]     hand-made [a, b]
put c  LinkedHashMap [a, b, c]  hand-made [a, b, c]
get a  LinkedHashMap [b, c, a]  hand-made [b, c, a]
put d  LinkedHashMap [c, a, d]  hand-made [c, a, d]
get b  LinkedHashMap [c, a, d]  hand-made [c, a, d]
put e  LinkedHashMap [a, d, e]  hand-made [a, d, e]
get c  LinkedHashMap [a, d, e]  hand-made [a, d, e]
put a  LinkedHashMap [d, e, a]  hand-made [d, e, a]
200,000 random operations: 0 mismatches, 21048 evictions, same final order: true
$ cd lru && java Index.java
quick        -> [0, 1, 2] ["The quick brown fox", "A quick study of Java collections", "Brown bears and the quick river"]
quick brown  -> [0, 2] ["The quick brown fox", "Brown bears and the quick river"]
java         -> [1, 3, 4] ["A quick study of Java collections", "Java maps: HashMap, TreeMap, LinkedHashMap", "The fox studies Java"]
java fox     -> [4] ["The fox studies Java"]
the quick    -> [0, 2] ["The quick brown fox", "Brown bears and the quick river"]
python       -> [] []
distinct words indexed: 17
```

The two caches behave identically on a hand-traced sequence and on 200,000 random operations. The hand-made version shows what `LinkedHashMap` does inside: a hash table for O(1) lookup, with every entry also on a doubly linked list so that "move to most recent" and "evict least recent" are O(1) pointer updates. The index uses `computeIfAbsent` for the map-of-sets idiom, `TreeSet` for sorted postings, and `retainAll` for intersection — starting from the rarest word, so the working set only shrinks.

### Review checklist

- Neither cache is thread-safe. What goes wrong if two threads call `get` on `SimpleLru` at once? (In access order, even `get` modifies the list.)
- Why do the sentinels `head` and `tail` simplify `unlink` and `linkFirst`?
- How would the index support OR queries and phrase queries? Which collection would hold word positions?

## Answers to the Chapter 6 prediction questions

**6.5 Q1.** `remove(Integer.valueOf(1))` calls `remove(Object)`: 1 is not in the list, so it returns `false`. `remove(1)` calls `remove(int index)` and returns the removed element, `6`.

**6.5 Q2.** `{a=6, b=10}`. `merge` combines 1 and 5 with `Integer::sum`; `computeIfAbsent` adds `b`; `putIfAbsent` does nothing because `a` is present.

# Chapter 7 — Lambdas, Streams, and Functional Style

*Goal: write Java in its modern functional style — lambdas and method references, the standard functional interfaces, stream pipelines with collectors, `Optional` chains — and know how lambdas are implemented, what laziness means, and when streams (and parallel streams) help or hurt.*

**What you will be able to do after this chapter**

- Write lambdas and method references, and explain capture and "effectively final".
- Use `Function`, `Predicate`, `Supplier`, `Consumer`, `BiFunction`, `UnaryOperator`, and compose them.
- Build stream pipelines: `filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `reduce`, `collect`.
- Use collectors: `groupingBy`, `partitioningBy`, `toMap`, `joining`, `teeing`, and downstream collectors.
- Explain laziness and short-circuiting, avoid side effects, and decide when a parallel stream is appropriate.

## Topic 7A — Lambdas

## Unit 7.1 — Lambdas and functional interfaces

**One-line summary:** A **functional interface** has exactly one abstract method; a **lambda** `(args) -> body` or a **method reference** (`String::length`, `this::save`, `ArrayList::new`) supplies its implementation. The standard ones live in `java.util.function`: `Function<T,R>`, `BiFunction<T,U,R>`, `Predicate<T>`, `Supplier<T>`, `Consumer<T>`, `UnaryOperator<T>`, `BinaryOperator<T>`, plus primitive versions (`IntPredicate`, `ToIntFunction`, …) that avoid boxing. Lambdas can use local variables only if they are **effectively final**.

```java
// file: Lambdas.java
import java.util.ArrayList;
import java.util.List;
import java.util.function.BiFunction;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.function.UnaryOperator;

public class Lambdas {
    @FunctionalInterface
    interface Validator<T> {
        boolean check(T value);                                    // the single abstract method
        default Validator<T> and(Validator<T> other) { return v -> check(v) && other.check(v); }
    }

    static <T, R> List<R> mapAll(List<T> in, Function<? super T, ? extends R> f) {
        List<R> out = new ArrayList<>();
        for (T t : in) out.add(f.apply(t));
        return out;
    }

    public static void main(String[] args) {
        Function<String, Integer> length = String::length;                 // method reference: instance method of the argument
        Function<Integer, Integer> square = x -> x * x;
        Function<String, Integer> lengthSquared = length.andThen(square);   // composition
        Predicate<String> isBlank = String::isBlank;
        Predicate<String> longWord = s -> s.length() > 4;
        Supplier<List<String>> newList = ArrayList::new;                    // constructor reference
        BiFunction<Integer, Integer, Integer> max = Math::max;              // static method reference
        UnaryOperator<String> shout = s -> s.toUpperCase() + "!";

        List<String> words = List.of("lambda", "is", "", "an", "anonymous", "function");
        System.out.println(mapAll(words, lengthSquared) + " " + words.stream().filter(isBlank.negate().and(longWord)).toList());
        List<String> made = newList.get();
        made.add(shout.apply("made"));
        System.out.println(made + " max=" + max.apply(3, 9));

        Validator<String> notEmpty = s -> !s.isEmpty();
        Validator<String> ascii = s -> s.chars().allMatch(c -> c < 128);
        System.out.println("validators: " + notEmpty.and(ascii).check("plain") + " " + notEmpty.and(ascii).check("naïve"));

        int base = 10;                                              // effectively final: never reassigned
        Function<Integer, Integer> addBase = x -> x + base;
        // base++;                                                  // would make the lambda above a compile error
        int[] counter = {0};                                        // the usual workaround: a mutable holder
        Runnable r = () -> counter[0]++;
        r.run(); r.run();
        System.out.println("addBase(5) = " + addBase.apply(5) + ", counter = " + counter[0]);
    }
}
```

```console
$ java Lambdas.java
[36, 4, 0, 4, 81, 64] [lambda, anonymous, function]
[MADE!] max=9
validators: true false
addBase(5) = 15, counter = 2
```

### Deep & Tricky

**DT-25. A lambda is not an anonymous inner class: it compiles to a private method plus an `invokedynamic` call site, and a non-capturing lambda is a single shared instance.** *(Implementation behaviour: JEP 126, `LambdaMetafactory`)* `javac` turns the lambda body into a synthetic method (`lambda$main$0`) and emits `invokedynamic`; at first execution the JVM spins a tiny class implementing the interface. A lambda that captures nothing is created once and reused — no allocation per evaluation — while a capturing lambda allocates an object each time it is evaluated (which escape analysis often removes). Unlike an inner class, a lambda has no `this` of its own (`this` means the enclosing instance) and no separate class file. Do not rely on lambda identity: whether two evaluations return the same object is unspecified.

```java
// file: LambdaIdentity.java
import java.util.function.Supplier;

public class LambdaIdentity {
    static Supplier<String> nonCapturing() { return () -> "constant"; }
    static Supplier<String> capturing(String s) { return () -> s; }

    public static void main(String[] args) {
        System.out.println("non-capturing: same instance each time? " + (nonCapturing() == nonCapturing()));
        System.out.println("capturing:     same instance each time? " + (capturing("x") == capturing("x")));
        System.out.println("a lambda's class is generated at run time: name contains $$Lambda? " + nonCapturing().getClass().getName().contains("$$Lambda"));
    }
}
```

```console
$ javac -d out LambdaIdentity.java && java -cp out LambdaIdentity && javap -p -cp out LambdaIdentity | grep lambda
non-capturing: same instance each time? true
capturing:     same instance each time? false
a lambda's class is generated at run time: name contains $$Lambda? true
  private static java.lang.String lambda$capturing$1(java.lang.String);
  private static java.lang.String lambda$nonCapturing$0();
```

<!-- expanded -->
### Going deeper: the four method-reference forms

A method reference is a short lambda. There are four kinds, and the third is the one that confuses people: `String::length` used as a `Function<String, Integer>` means "call `length` on the argument", so the first parameter becomes the receiver.

```java
// file: MethodRefs.java
import java.util.List;
import java.util.function.BiFunction;
import java.util.function.Function;
import java.util.function.Supplier;

public class MethodRefs {
    public static void main(String[] args) {
        Function<String, Integer> parse = Integer::parseInt;          // 1. static method
        String prefix = "id-";
        Function<String, String> addPrefix = prefix::concat;          // 2. method of a particular object
        Function<String, Integer> len = String::length;               // 3. method of the argument
        BiFunction<String, String, Boolean> starts = String::startsWith; //    two args: receiver + parameter
        Supplier<List<String>> make = java.util.ArrayList::new;       // 4. constructor

        System.out.println(parse.apply("42") + 1);
        System.out.println(addPrefix.apply("7"));
        System.out.println(len.apply("hello") + " " + starts.apply("prefix", "pre"));
        List<String> l = make.get();
        l.add("built by a constructor reference");
        System.out.println(l);
        System.out.println(List.of("b", "a").stream().map(String::toUpperCase).sorted().toList());
    }
}
```

```console
$ java MethodRefs.java
43
id-7
5 true
[built by a constructor reference]
[A, B]
```

## Topic 7B — Streams

## Unit 7.2 — Stream pipelines

**One-line summary:** A **stream** is a pipeline over a source (collection, array, generator, file lines): zero or more lazy **intermediate** operations (`filter`, `map`, `flatMap`, `distinct`, `sorted`, `peek`, `limit`, `skip`, `takeWhile`) and one **terminal** operation that runs it (`collect`, `toList`, `forEach`, `reduce`, `count`, `min`, `max`, `anyMatch`, `findFirst`). A stream can be consumed only once. `IntStream`, `LongStream`, and `DoubleStream` work on primitives without boxing.

```java
// file: Streams.java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class Streams {
    record Order(String customer, String product, int qty, double price) {
        double total() { return qty * price; }
    }

    public static void main(String[] args) {
        List<Order> orders = List.of(
            new Order("ada", "pen", 10, 1.5), new Order("bob", "ink", 2, 9.0), new Order("ada", "pad", 3, 4.0),
            new Order("cy", "pen", 100, 1.2), new Order("bob", "pad", 1, 4.0), new Order("ada", "ink", 1, 9.0));

        List<String> bigSpenders = orders.stream()
            .filter(o -> o.total() > 10)
            .sorted(Comparator.comparingDouble(Order::total).reversed())
            .map(o -> o.customer() + ":" + o.product())
            .toList();
        double revenue = orders.stream().mapToDouble(Order::total).sum();
        var stats = orders.stream().mapToInt(Order::qty).summaryStatistics();
        System.out.println(bigSpenders + " revenue " + revenue + " " + stats);

        System.out.println(IntStream.rangeClosed(1, 5).map(x -> x * x).boxed().toList()
                + " " + Stream.iterate(1, x -> x * 2).limit(10).reduce(0, Integer::sum)
                + " " + Stream.of("b,a", "c", "a,d").flatMap(s -> Arrays.stream(s.split(","))).distinct().sorted().collect(Collectors.joining("|")));

        // laziness and short-circuiting: watch which elements each stage sees
        var first = Stream.of(5, 8, 3, 12, 7, 20)
            .peek(x -> System.out.print(" filter?" + x))
            .filter(x -> x > 6)
            .peek(x -> System.out.print(" map:" + x))
            .map(x -> x * 10)
            .findFirst();
        System.out.println("  -> " + first);

        Stream<String> once = Stream.of("a", "b");
        once.forEach(s -> {});
        try {
            once.forEach(s -> {});
        } catch (IllegalStateException e) {
            System.out.println("second use: " + e.getMessage());
        }
    }
}
```

```console
$ java Streams.java
[cy:pen, bob:ink, ada:pen, ada:pad] revenue 178.0 IntSummaryStatistics{count=6, sum=117, min=1, average=19.500000, max=100}
[1, 4, 9, 16, 25] 1023 a|b|c|d
 filter?5 filter?8 map:8  -> Optional[80]
second use: stream has already been operated upon or closed
```

The trace shows **vertical** processing: element 5 is tested and dropped; element 8 passes the filter, is mapped, and `findFirst` stops the pipeline — 3, 12, 7, and 20 are never examined.

<!-- expanded -->
### Going deeper: `flatMap`, `takeWhile`, `iterate`, and primitive streams

`flatMap` turns each element into a stream and concatenates them — one order into its many lines. `takeWhile`/`dropWhile` cut a stream at the first element that fails a test. `Stream.iterate(seed, hasNext, next)` is a `for` loop as a stream. `IntStream`, `LongStream`, and `DoubleStream` avoid boxing and add `sum`, `average`, and `summaryStatistics`.

```java
// file: Streams2.java
import java.util.List;
import java.util.stream.IntStream;
import java.util.stream.Stream;

public class Streams2 {
    record Order(String id, List<String> items) {}

    public static void main(String[] args) {
        List<Order> orders = List.of(new Order("o1", List.of("pen", "ink")), new Order("o2", List.of("pad")),
                                     new Order("o3", List.of("pen", "pad", "clip")));
        System.out.println(orders.stream().flatMap(o -> o.items().stream()).distinct().sorted().toList());

        System.out.println(Stream.iterate(1, n -> n < 1000, n -> n * 3).toList());
        System.out.println(IntStream.of(2, 4, 6, 7, 8).takeWhile(n -> n % 2 == 0).boxed().toList()
                           + " " + IntStream.of(2, 4, 6, 7, 8).dropWhile(n -> n % 2 == 0).boxed().toList());
        var stats = IntStream.rangeClosed(1, 100).summaryStatistics();
        System.out.println(stats);
        System.out.println(IntStream.rangeClosed(1, 5).mapToObj(i -> "*".repeat(i)).reduce((a, b) -> a + " " + b).get());
    }
}
```

```console
$ java Streams2.java
[clip, ink, pad, pen]
[1, 3, 9, 27, 81, 243, 729]
[2, 4, 6] [7, 8]
IntSummaryStatistics{count=100, sum=5050, min=1, average=50.500000, max=100}
* ** *** **** *****
```

## Unit 7.3 — Collectors

**One-line summary:** `collect(collector)` folds a stream into a container. The essential collectors: `toList`/`toSet`/`toMap`, `joining`, `groupingBy(classifier, downstream)`, `partitioningBy`, `counting`, `summingInt`, `averagingDouble`, `mapping`, `filtering`, `collectingAndThen`, and `teeing` (two collectors in one pass). Downstream collectors nest to any depth.

```java
// file: Collecting.java
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.TreeMap;
import java.util.function.Function;
import java.util.stream.Collectors;

public class Collecting {
    record Emp(String name, String dept, String city, int salary) {}

    public static void main(String[] args) {
        List<Emp> staff = List.of(new Emp("Ann", "eng", "Oslo", 150), new Emp("Ben", "ops", "Rome", 95), new Emp("Mia", "eng", "Rome", 120),
                new Emp("Raj", "ops", "Oslo", 90), new Emp("Zoe", "eng", "Oslo", 120), new Emp("Kim", "hr", "Rome", 80));

        Map<String, List<String>> namesByDept = staff.stream()
            .collect(Collectors.groupingBy(Emp::dept, TreeMap::new, Collectors.mapping(Emp::name, Collectors.toList())));
        Map<String, Double> avgByDept = staff.stream().collect(Collectors.groupingBy(Emp::dept, TreeMap::new, Collectors.averagingInt(Emp::salary)));
        Map<String, Map<String, Long>> countByDeptCity = staff.stream()
            .collect(Collectors.groupingBy(Emp::dept, TreeMap::new, Collectors.groupingBy(Emp::city, TreeMap::new, Collectors.counting())));
        Map<Boolean, List<String>> highEarners = staff.stream()
            .collect(Collectors.partitioningBy(e -> e.salary() >= 120, Collectors.mapping(Emp::name, Collectors.toList())));
        Map<String, String> topByDept = staff.stream().collect(Collectors.toMap(Emp::dept, Emp::name,
                (a, b) -> a, TreeMap::new));                                    // merge function: keep the first
        Map<String, Emp> byName = staff.stream().collect(Collectors.toMap(Emp::name, Function.identity()));
        String range = staff.stream().collect(Collectors.teeing(
                Collectors.minBy(Comparator.comparingInt(Emp::salary)), Collectors.maxBy(Comparator.comparingInt(Emp::salary)),
                (min, max) -> min.get().name() + ".." + max.get().name()));

        System.out.println(namesByDept + "\n" + avgByDept + "\n" + countByDeptCity + "\n" + highEarners + "\n" + topByDept
                + " " + byName.get("Kim").city() + " " + range);
        try {
            staff.stream().collect(Collectors.toMap(Emp::dept, Emp::name));       // duplicate keys, no merge function
        } catch (IllegalStateException e) {
            System.out.println("toMap without a merge function: " + e.getMessage());
        }
    }
}
```

```console
$ java Collecting.java
{eng=[Ann, Mia, Zoe], hr=[Kim], ops=[Ben, Raj]}
{eng=130.0, hr=80.0, ops=92.5}
{eng={Oslo=2, Rome=1}, hr={Rome=1}, ops={Oslo=1, Rome=1}}
{false=[Ben, Raj, Kim], true=[Ann, Mia, Zoe]}
{eng=Ann, hr=Kim, ops=Ben} Rome Kim..Ann
toMap without a merge function: Duplicate key eng (attempted merging values Ann and Mia)
```

### Deep & Tricky

**DT-26. Streams are lazy, single-use, and must not have side effects that interfere with their source or depend on order.** *(Library behaviour: `java.util.stream` package specification)* Nothing runs until the terminal operation; a pipeline without one (`list.stream().map(this::save);`) silently does nothing. Lambdas passed to stream operations should be **non-interfering** (do not modify the source) and **stateless**; adding to an external `ArrayList` from `forEach` works sequentially by accident and loses or corrupts elements in parallel. Collect results with a collector instead. `peek` is for debugging only — a stream may skip it when it can compute the result without traversing (e.g. `count()` on a sized source).

<!-- expanded -->
### Deep & Tricky

**DT-27. `Collectors.toMap` throws `IllegalStateException` on a duplicate key (unless you pass a merge function), and throws `NullPointerException` if any *value* is `null` — even though `HashMap` allows null values. `groupingBy` and `toList` have no such limits.** *(Library behaviour: `Collectors.toMap` Javadoc; the null check comes from its use of `Map.merge`)*

```java
// file: ToMap.java
import java.util.List;
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;

public class ToMap {
    record City(String name, String state, Integer population) {}

    public static void main(String[] args) {
        List<City> cities = List.of(new City("pune", "MH", 7), new City("mumbai", "MH", 21), new City("delhi", "DL", 32));
        try {
            cities.stream().collect(Collectors.toMap(City::state, City::name));
        } catch (IllegalStateException e) {
            System.out.println("toMap: " + e.getMessage());
        }
        Map<String, String> merged = cities.stream()
            .collect(Collectors.toMap(City::state, City::name, (a, b) -> a + "+" + b, TreeMap::new));
        System.out.println("with merge function: " + merged);
        System.out.println("groupingBy: " + cities.stream().collect(Collectors.groupingBy(City::state, TreeMap::new,
                                Collectors.mapping(City::name, Collectors.toList()))));
        try {
            List.of(new City("new", "XX", null)).stream().collect(Collectors.toMap(City::name, City::population));
        } catch (NullPointerException e) {
            System.out.println("toMap with a null value: NullPointerException");
        }
    }
}
```

```console
$ java ToMap.java
toMap: Duplicate key MH (attempted merging values pune and mumbai)
with merge function: {DL=delhi, MH=pune+mumbai}
groupingBy: {DL=[delhi], MH=[pune, mumbai]}
toMap with a null value: NullPointerException
```

## Unit 7.4 — Parallel streams

**One-line summary:** `.parallel()` splits the source and runs the pipeline on the common **ForkJoinPool**. It helps when there is a lot of CPU work per element, the source splits cheaply (arrays, `ArrayList`, ranges — not `LinkedList` or `Stream.iterate`), and operations are stateless and associative. It hurts for small inputs, cheap operations, blocking I/O (it starves the shared pool), and ordered operations like `limit` on unsized sources.

```java
// file: Parallel.java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.stream.IntStream;
import java.util.stream.LongStream;

public class Parallel {
    static boolean isPrime(long n) {
        if (n < 2) return false;
        for (long d = 2; d * d <= n; d++) if (n % d == 0) return false;
        return true;
    }

    static long ms(Runnable r) { long t = System.nanoTime(); r.run(); return (System.nanoTime() - t) / 1_000_000; }
    static long us(Runnable r) { long t = System.nanoTime(); r.run(); return (System.nanoTime() - t) / 1_000; }

    public static void main(String[] args) {
        long[] result = new long[4];
        for (int round = 0; round < 2; round++) {                              // round 0 warms up
            long seq = ms(() -> result[0] = LongStream.range(0, 2_000_000).filter(Parallel::isPrime).count());
            long par = ms(() -> result[1] = LongStream.range(0, 2_000_000).parallel().filter(Parallel::isPrime).count());
            long seqCheap = us(() -> result[2] = IntStream.range(0, 2_000).map(x -> x + 1).sum());
            long parCheap = us(() -> result[3] = IntStream.range(0, 2_000).parallel().map(x -> x + 1).sum());
            if (round == 1) {
                System.out.printf("primes below 2M: sequential %d ms, parallel %d ms on %d cores (results %d = %d)%n",
                        seq, par, Runtime.getRuntime().availableProcessors(), result[0], result[1]);
                System.out.printf("tiny cheap task (2,000 additions): sequential %d µs, parallel %d µs%n", seqCheap, parCheap);
            }
        }
        List<Integer> unsafe = new ArrayList<>();                              // not thread-safe
        try {
            IntStream.range(0, 100_000).parallel().forEach(unsafe::add);       // side effect on shared state: a data race
            System.out.println("unsafe side effect: expected 100000 elements, got " + unsafe.size());
        } catch (RuntimeException e) {
            System.out.println("unsafe side effect: failed with " + e.getClass().getSimpleName());
        }
        List<Integer> safe = IntStream.range(0, 100_000).parallel().boxed().toList();   // let the collector do it
        List<Integer> sync = Collections.synchronizedList(new ArrayList<>());
        IntStream.range(0, 100_000).parallel().forEach(sync::add);
        System.out.println("collector: " + safe.size() + " in order: " + (safe.get(99_999) == 99_999) + "; synchronizedList: " + sync.size()
                + " in order: " + sync.equals(safe));
    }
}
```

```console
$ java Parallel.java
primes below 2M: sequential 481 ms, parallel 296 ms on 2 cores (results 148933 = 148933)
tiny cheap task (2,000 additions): sequential 129 µs, parallel 130 µs
unsafe side effect: expected 100000 elements, got 89465
collector: 100000 in order: true; synchronizedList: 100000 in order: false
```

Three results. The CPU-heavy prime count ran faster in parallel, though well short of 2× on two cores (the work is uneven: large numbers cost more, and the pool shares the machine). The tiny task took a hundred-odd microseconds either way — at that size neither version matters, and repeated runs put either one ahead. And the unsafe `forEach(unsafe::add)` **lost elements** — thousands of them in this run, with no exception: a data race on `ArrayList`'s internal array. The collector version is correct and ordered; the `synchronizedList` version is complete but in arbitrary order.

### Pattern

**P-7. Use streams to say *what* is computed; use loops when the logic needs early exits with state, checked exceptions, or index arithmetic; make streams parallel only after measuring.** *(Practice; Effective Java, Items 45–48)* A pipeline of `filter`/`map`/`collect` reads as a description of the result and composes with collectors. A loop is clearer when the body mutates several variables, must `break` across levels, or throws checked exceptions. Keep lambdas short — extract a named method and use a method reference. *Trade-off:* streams add some overhead per element (usually irrelevant) and make debugging by stepping harder; `parallel()` adds real risk for small or no gain unless the work is CPU-heavy and side-effect free.

### Mastery check

1. **Predict.** What does `Stream.of(1, 2, 3, 4).peek(System.out::print).map(x -> x * 2).count()` print on Java 21?
2. **Predict.** What does `List.of("a", "bb", "cc").stream().collect(Collectors.toMap(String::length, s -> s))` do?

## Chapter 7 Project — A sales-analytics pipeline

**Goal:** analyse 500,000 generated sales records with streams and collectors: revenue per region and month, top products by revenue, each customer's first and largest purchase, a histogram of order sizes, and a custom collector that computes mean and standard deviation in one pass. The same analysis is written with loops for one query, to compare clarity and speed, and run in parallel to see whether it helps.

```java
// file: sales/Sales.java
import java.time.LocalDate;
import java.time.YearMonth;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Random;
import java.util.TreeMap;
import java.util.stream.Collector;
import java.util.stream.Collectors;

public class Sales {
    record Sale(int id, LocalDate date, String region, String product, String customer, int qty, double unitPrice) {
        double amount() { return qty * unitPrice; }
    }

    /** Mean and standard deviation in one pass (mergeable, so it works in parallel). */
    static final class Stats {
        long n; double sum, sumSq;
        void add(double x) { n++; sum += x; sumSq += x * x; }
        Stats merge(Stats o) { n += o.n; sum += o.sum; sumSq += o.sumSq; return this; }
        double mean() { return sum / n; }
        double stddev() { return Math.sqrt(sumSq / n - mean() * mean()); }
        static Collector<Double, Stats, Stats> collector() {
            return Collector.of(Stats::new, Stats::add, Stats::merge, Collector.Characteristics.UNORDERED);
        }
    }

    static List<Sale> generate(int n) {
        Random r = new Random(2026);
        String[] regions = {"north", "south", "east", "west"};
        String[] products = {"laptop", "phone", "tablet", "monitor", "keyboard", "mouse", "cable"};
        double[] prices = {1200, 800, 450, 300, 80, 35, 9};
        List<Sale> out = new ArrayList<>(n);
        for (int i = 0; i < n; i++) {
            int p = (int) Math.min(products.length - 1, Math.abs(r.nextGaussian() * 2.5));       // skewed: low values are common...
            p = products.length - 1 - p;                                                          // ...so cheap items sell most often
            out.add(new Sale(i, LocalDate.of(2026, 1, 1).plusDays(r.nextInt(300)), regions[r.nextInt(4)], products[p],
                    "c" + (int) Math.abs(r.nextGaussian() * 2000), 1 + r.nextInt(5), prices[p] * (0.9 + r.nextDouble() * 0.2)));
        }
        return out;
    }

    static long ms(Runnable r) { long t = System.nanoTime(); r.run(); return (System.nanoTime() - t) / 1_000_000; }

    public static void main(String[] args) {
        List<Sale> sales = generate(500_000);

        Map<String, Double> byRegion = sales.stream().collect(Collectors.groupingBy(Sale::region, TreeMap::new, Collectors.summingDouble(Sale::amount)));
        System.out.println("revenue by region (millions): " + byRegion.entrySet().stream()
                .map(e -> "%s=%.2f".formatted(e.getKey(), e.getValue() / 1e6)).collect(Collectors.joining(", ")));

        Map<YearMonth, Double> byMonth = sales.stream()
            .collect(Collectors.groupingBy(s -> YearMonth.from(s.date()), TreeMap::new, Collectors.summingDouble(Sale::amount)));
        var best = byMonth.entrySet().stream().max(Map.Entry.comparingByValue()).orElseThrow();
        System.out.printf("months: %d, best month %s with %.2f M%n", byMonth.size(), best.getKey(), best.getValue() / 1e6);

        System.out.println("top 3 products: " + sales.stream()
            .collect(Collectors.groupingBy(Sale::product, Collectors.summingDouble(Sale::amount)))
            .entrySet().stream().sorted(Map.Entry.<String, Double>comparingByValue().reversed()).limit(3)
            .map(e -> "%s %.1fM".formatted(e.getKey(), e.getValue() / 1e6)).toList());

        Map<String, Sale> largestPerCustomer = sales.stream()
            .collect(Collectors.toMap(Sale::customer, s -> s, (a, b) -> a.amount() >= b.amount() ? a : b));
        Sale whale = largestPerCustomer.values().stream().max(Comparator.comparingDouble(Sale::amount)).orElseThrow();
        System.out.printf("%d customers; largest single purchase: %s bought %d x %s for %.2f%n",
                largestPerCustomer.size(), whale.customer(), whale.qty(), whale.product(), whale.amount());

        Map<Integer, Long> histogram = sales.stream().collect(Collectors.groupingBy(Sale::qty, TreeMap::new, Collectors.counting()));
        System.out.println("orders by quantity: " + histogram);

        Stats st = sales.stream().map(Sale::amount).collect(Stats.collector());
        Stats stPar = sales.parallelStream().map(Sale::amount).collect(Stats.collector());
        System.out.printf("order amount: mean %.2f, stddev %.2f (parallel run agrees: %b)%n", st.mean(), st.stddev(),
                Math.abs(st.mean() - stPar.mean()) < 1e-6 && st.n == stPar.n);

        // the same query three ways, timed after warm-up
        Map<String, Double> viaLoop = new HashMap<>();
        long tLoop = 0, tStream = 0, tPar = 0;
        for (int round = 0; round < 5; round++) {
            viaLoop.clear();
            tLoop = ms(() -> { for (Sale s : sales) viaLoop.merge(s.region(), s.amount(), Double::sum); });
            tStream = ms(() -> sales.stream().collect(Collectors.groupingBy(Sale::region, Collectors.summingDouble(Sale::amount))));
            tPar = ms(() -> sales.parallelStream().collect(Collectors.groupingBy(Sale::region, Collectors.summingDouble(Sale::amount))));
        }
        System.out.printf("revenue by region: loop %d ms, stream %d ms, parallel stream %d ms; loop result matches: %b%n", tLoop, tStream, tPar,
                viaLoop.keySet().equals(byRegion.keySet()) && Math.abs(viaLoop.get("north") - byRegion.get("north")) < 1e-3);
    }
}
```

```console
$ cd sales && java Sales.java
revenue by region (millions): east=51.19, north=51.06, south=51.12, west=50.78
months: 10, best month 2026-03 with 21.31 M
top 3 products: [monitor 54.4M, tablet 43.2M, phone 35.2M]
6832 customers; largest single purchase: c4517 bought 5 x laptop for 6599.33
orders by quantity: {1=99593, 2=100005, 3=100339, 4=100149, 5=99914}
order amount: mean 408.29, stddev 765.71 (parallel run agrees: true)
revenue by region: loop 23 ms, stream 15 ms, parallel stream 8 ms; loop result matches: true
```

Every query is a single expression that reads top to bottom: source, classification, aggregation. The custom `Stats` collector shows the four parts of any collector — supplier, accumulator, combiner, finisher — and because its combiner merges partial results, the same collector works unchanged in a parallel stream. The timing at the end is the honest part: for a cheap aggregation over half a million records, the loop, the sequential stream, and the parallel stream all finish in 10–20 ms; the differences are within what JIT timing and noise produce on this machine. Summing doubles is too cheap per element for two cores to matter much (P-7) — unlike the prime test in Unit 7.4, where each element costs real CPU time and the parallel version was clearly faster.

### Review checklist

- Which queries would change if `sales` were a `Stream<Sale>` read lazily from a 50 GB file instead of a `List`? (A stream can be traversed once; you would need one pass with `teeing` or a combined collector.)
- `Collectors.toMap` needs a merge function here. What happens without it?
- The parallel floating-point sums may differ from the sequential ones in the last digits. Why? (Addition order; floating-point addition is not associative.)

## Answers to the Chapter 7 prediction questions

**7.4 Q1.** Nothing (and it returns 4). The source has a known size and neither `peek` nor `map` changes the count, so since Java 9 `count()` does not traverse the stream at all — which is why `peek` must never be used for required side effects (DT-26).

**7.4 Q2.** It throws `IllegalStateException: Duplicate key 2 (attempted merging values bb and cc)`: two elements map to the key 2 and no merge function was given.

# Chapter 8 — I/O, NIO, Files, and Serialization

*Goal: read and write data correctly and efficiently — byte and character streams, encodings, buffering, the `java.nio.file` API, channels and memory-mapped files, binary formats with `ByteBuffer` — and understand why Java's built-in serialization is dangerous and what to use instead.*

**What you will be able to do after this chapter**

- Distinguish byte streams from character streams, and always specify a charset.
- Read and write files with `Files`, walk directory trees, and watch for changes.
- Explain what buffering saves, and measure it.
- Use `FileChannel`, `ByteBuffer`, and memory-mapped files for binary data.
- Design a binary file format with explicit byte order and checksums.
- Explain the risks of Java serialization and use safer alternatives.

## Topic 8A — Streams and files

## Unit 8.1 — Bytes, characters, and encodings

**One-line summary:** `InputStream`/`OutputStream` move **bytes**; `Reader`/`Writer` move **characters** and need a **charset** to convert. Since Java 18 the default charset is UTF-8 everywhere (JEP 400), but always state it (`StandardCharsets.UTF_8`) in code that must run on older JVMs or exchange data. Decoding bytes with the wrong charset silently produces garbage ("mojibake").

```java
// file: Encodings.java
import java.nio.charset.StandardCharsets;
import java.util.HexFormat;

public class Encodings {
    public static void main(String[] args) {
        String text = "naïve €5 🦀";
        byte[] utf8 = text.getBytes(StandardCharsets.UTF_8);
        byte[] utf16 = text.getBytes(StandardCharsets.UTF_16BE);
        byte[] latin1 = text.getBytes(StandardCharsets.ISO_8859_1);          // cannot represent € or 🦀
        System.out.println(text.length() + " chars -> UTF-8 " + utf8.length + " bytes, UTF-16 " + utf16.length + " bytes, Latin-1 " + latin1.length + " bytes");
        System.out.println("UTF-8 hex:  " + HexFormat.ofDelimiter(" ").formatHex(utf8));
        System.out.println("Latin-1 round trip: " + new String(latin1, StandardCharsets.ISO_8859_1) + "   (unmappable characters became '?')");
        System.out.println("UTF-8 bytes read as Latin-1: " + new String(utf8, StandardCharsets.ISO_8859_1).replaceAll("\\p{Cntrl}", "·"));
        System.out.println("default charset: " + java.nio.charset.Charset.defaultCharset());
    }
}
```

```console
$ java Encodings.java
11 chars -> UTF-8 16 bytes, UTF-16 22 bytes, Latin-1 10 bytes
UTF-8 hex:  6e 61 c3 af 76 65 20 e2 82 ac 35 20 f0 9f a6 80
Latin-1 round trip: naïve ?5 ?   (unmappable characters became '?')
UTF-8 bytes read as Latin-1: naÃ¯ve â¬5 ð¦
default charset: UTF-8
```

<!-- expanded -->
### Going deeper: the default charset, and decoding errors

Since Java 18 the default charset is UTF-8 on every platform (JEP 400); before that it was the operating system's, so the same program could read a file differently on Windows and Linux. Pass the charset explicitly anyway when reading external data — it documents the assumption. What happens to bytes that are not valid in the charset depends on the API: `new String(bytes, UTF_8)` and `Files.readString` disagree.

```java
// file: Decoding.java
import java.nio.charset.Charset;
import java.nio.charset.MalformedInputException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;

public class Decoding {
    public static void main(String[] args) throws Exception {
        System.out.println("default charset: " + Charset.defaultCharset());
        byte[] bad = {'c', 'a', 'f', (byte) 0xE9};                 // "café" in Latin-1, invalid as UTF-8
        System.out.println("new String(.., UTF_8): " + new String(bad, StandardCharsets.UTF_8).replace('�', '?')
                           + " (bad byte replaced)");
        System.out.println("new String(.., ISO_8859_1): " + new String(bad, StandardCharsets.ISO_8859_1));
        Path p = Files.write(Path.of("latin1.txt"), bad);
        try {
            Files.readString(p);                                    // strict: refuses invalid UTF-8
        } catch (MalformedInputException e) {
            System.out.println("Files.readString: MalformedInputException, input length " + e.getInputLength());
        }
        System.out.println("Files.readString(p, ISO_8859_1): " + Files.readString(p, StandardCharsets.ISO_8859_1));
    }
}
```

```console
$ java Decoding.java
default charset: UTF-8
new String(.., UTF_8): caf? (bad byte replaced)
new String(.., ISO_8859_1): café
Files.readString: MalformedInputException, input length 1
Files.readString(p, ISO_8859_1): café
```

The lenient constructor silently turned the byte into U+FFFD (the replacement character, printed here as `?`); `Files.readString` refused. Silent replacement is how corrupted names end up in databases.

## Unit 8.2 — The `Files` API

**One-line summary:** `java.nio.file.Path` names a file; `Files` does the work: `readString`, `writeString`, `readAllLines`, `lines` (a lazy stream — close it), `newBufferedReader`/`Writer`, `copy`, `move` (optionally atomic), `delete`, `exists`, `size`, `createDirectories`, `createTempFile`, `walk`, `list`, and attribute access. Prefer it to the old `java.io.File`, whose methods return `false` instead of explaining failures.

```java
// file: FilesDemo.java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.NoSuchFileException;
import java.nio.file.Path;
import java.nio.file.StandardCopyOption;
import java.nio.file.StandardOpenOption;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Stream;

public class FilesDemo {
    public static void main(String[] args) throws IOException {
        Path root = Path.of("demo-tree");
        Files.createDirectories(root.resolve("src/main"));
        Files.createDirectories(root.resolve("docs"));
        Files.writeString(root.resolve("src/main/App.java"), "class App {}\n");
        Files.writeString(root.resolve("src/main/Util.java"), "class Util {}\n// helper\n");
        Files.write(root.resolve("docs/notes.txt"), List.of("first line", "second line"));
        Files.writeString(root.resolve("docs/notes.txt"), "appended\n", StandardOpenOption.APPEND);

        try (Stream<Path> walk = Files.walk(root)) {                           // close the stream: it holds directory handles
            walk.filter(Files::isRegularFile).sorted().forEach(p -> {
                try {
                    System.out.printf("%-34s %3d bytes%n", root.relativize(p), Files.size(p));
                } catch (IOException e) {
                    throw new java.io.UncheckedIOException(e);
                }
            });
        }
        try (Stream<String> lines = Files.lines(root.resolve("docs/notes.txt"))) {
            System.out.println("lines containing 'line': " + lines.filter(l -> l.contains("line")).count());
        }
        Path p = root.resolve("src/main/../main/./App.java");
        System.out.println("normalize: " + p.normalize() + ", file name: " + p.getFileName() + ", parent: " + p.normalize().getParent());
        Files.move(root.resolve("docs/notes.txt"), root.resolve("docs/NOTES.md"), StandardCopyOption.ATOMIC_MOVE);
        try {
            Files.delete(root.resolve("docs/notes.txt"));
        } catch (NoSuchFileException e) {
            System.out.println("Files.delete explains failures: NoSuchFileException " + e.getMessage()
                    + "  (java.io.File.delete() just returns " + new java.io.File("demo-tree/docs/notes.txt").delete() + ")");
        }
        try (Stream<Path> walk = Files.walk(root)) {                           // delete a tree: children before parents
            walk.sorted(Comparator.reverseOrder()).forEach(q -> q.toFile().delete());
        }
        System.out.println("tree removed: " + !Files.exists(root));
    }
}
```

```console
$ java FilesDemo.java
docs/notes.txt                      32 bytes
src/main/App.java                   13 bytes
src/main/Util.java                  24 bytes
lines containing 'line': 2
normalize: demo-tree/src/main/App.java, file name: App.java, parent: demo-tree/src/main
Files.delete explains failures: NoSuchFileException demo-tree/docs/notes.txt  (java.io.File.delete() just returns false)
tree removed: true
```

<!-- expanded -->
### Going deeper: walking a directory tree, and `Files.lines` must be closed

`Files.walk(root)` streams every path under a directory, `Files.find` adds a filter on attributes, and `Files.list` streams one level. These streams hold open directory handles, and `Files.lines` holds an open file — so all of them must be closed, which in practice means using them in try-with-resources.

```java
// file: Walk.java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Map;
import java.util.TreeMap;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class Walk {
    public static void main(String[] args) throws IOException {
        Path root = Files.createDirectories(Path.of("tree/src/util"));
        Files.writeString(root.resolve("Str.java"), "class Str {}\n");
        Files.writeString(Path.of("tree/src/Main.java"), "class Main {\n}\n");
        Files.writeString(Path.of("tree/README.md"), "# demo\nline 2\nline 3\n");

        try (Stream<Path> all = Files.walk(Path.of("tree"))) {
            Map<String, Long> byExt = all.filter(Files::isRegularFile)
                .collect(Collectors.groupingBy(p -> { String n = p.getFileName().toString(); return n.substring(n.lastIndexOf('.') + 1); },
                                               TreeMap::new, Collectors.counting()));
            System.out.println("files by extension: " + byExt);
        }
        try (Stream<String> lines = Files.lines(Path.of("tree/README.md"))) {
            System.out.println("README lines: " + lines.count());
        }
        try (Stream<Path> javaFiles = Files.find(Path.of("tree"), 10, (p, attr) -> attr.isRegularFile() && p.toString().endsWith(".java"))) {
            System.out.println("java files: " + javaFiles.map(p -> p.getFileName().toString()).sorted().toList());
        }
        System.out.println("relativize: " + Path.of("tree").relativize(Path.of("tree/src/util/Str.java"))
                           + ", normalize: " + Path.of("tree/src/../README.md").normalize());
    }
}
```

```console
$ java Walk.java
files by extension: {java=2, md=1}
README lines: 3
java files: [Main.java, Str.java]
relativize: src/util/Str.java, normalize: tree/README.md
```

A long-running server that calls `Files.lines` without closing the stream leaks a file descriptor per call and eventually fails with "Too many open files".

## Unit 8.3 — Buffering

**One-line summary:** Each `read()` or `write()` on a raw `FileInputStream`/`FileOutputStream` is a **system call**. `BufferedInputStream`/`BufferedOutputStream`/`BufferedReader`/`BufferedWriter` batch them through an 8 KiB buffer; `Files.newBufferedReader` and `newBufferedWriter` are buffered already. Always buffer byte-at-a-time or line-at-a-time I/O, and `flush()` (or close) a buffered writer before relying on its output.

```java
// file: Buffering.java
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.nio.file.Files;
import java.nio.file.Path;

public class Buffering {
    static long writeBytes(OutputStream out, int n) throws IOException {
        long t = System.nanoTime();
        try (out) {
            for (int i = 0; i < n; i++) out.write(i);                  // one byte at a time
        }
        return (System.nanoTime() - t) / 1_000_000;
    }

    public static void main(String[] args) throws IOException {
        int n = 2_000_000;
        long raw = writeBytes(new FileOutputStream("raw.bin"), n);
        long buffered = writeBytes(new BufferedOutputStream(new FileOutputStream("buf.bin")), n);
        System.out.printf("%,d single-byte writes: unbuffered %d ms, buffered %d ms (%.0fx); files identical: %b%n", n, raw, buffered,
                (double) raw / Math.max(1, buffered), Files.mismatch(Path.of("raw.bin"), Path.of("buf.bin")) == -1);
    }
}
```

```console
$ java Buffering.java
2,000,000 single-byte writes: unbuffered 561 ms, buffered 45 ms (12x); files identical: true
$ strace -f -c -e trace=write java Buffering.java 2>&1 | awk '$NF == "write" {print "write system calls for both versions together:", $4}'
write system calls for both versions together: 2000265
```

<!-- expanded -->
### Going deeper: what buffering saves

An unbuffered `FileOutputStream.write(int)` makes one system call per byte. `BufferedOutputStream` collects writes in an 8 KiB array and makes one system call per 8 KiB. On the machine used for this book, a million single-byte writes took about 300 ms unbuffered and 25 ms buffered; the system-call count below shows why.

```java
// file: BufferCost.java
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;

public class BufferCost {
    static long timeMs(boolean buffered) throws IOException {
        long t0 = System.nanoTime();
        try (OutputStream raw = new FileOutputStream("bytes.bin");
             OutputStream out = buffered ? new BufferedOutputStream(raw) : raw) {
            for (int i = 0; i < 1_000_000; i++) out.write(i);
        }
        return (System.nanoTime() - t0) / 1_000_000;
    }
    public static void main(String[] args) throws IOException {
        long unbuffered = timeMs(false), buffered = timeMs(true);
        System.out.println("1,000,000 single-byte writes: buffered is " + (unbuffered > 5 * buffered ? "more than 5x" : "less than 5x") + " faster");
        System.err.println("unbuffered " + unbuffered + " ms, buffered " + buffered + " ms");
    }
}
```

```console
$ java BufferCost.java 2>/dev/null && strace -f -c -e trace=write java BufferCost.java 2>&1 >/dev/null | grep -E " write$" | awk '{print "write system calls (both runs, all threads): " $4}'
1,000,000 single-byte writes: buffered is more than 5x faster
write system calls (both runs, all threads): 1000135
```

The `strace` count is dominated by the million unbuffered writes; the buffered run adds only about 123 (1,000,000 / 8192).

## Topic 8B — NIO

## Unit 8.4 — `ByteBuffer`, channels, and memory-mapped files

**One-line summary:** A `ByteBuffer` is a fixed block of bytes with a **position**, **limit**, and **capacity**, typed accessors (`putInt`, `getLong`, …), and an explicit **byte order** (big-endian by default). You write into it, `flip()` it, then read from it. A `FileChannel` reads and writes buffers at any file position, can `force()` data to disk, transfer between channels in the kernel, and **memory-map** a file so that its contents appear as a buffer backed by the page cache.

```java
// file: Buffers.java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.HexFormat;

public class Buffers {
    static String state(ByteBuffer b) { return "position=" + b.position() + " limit=" + b.limit() + " capacity=" + b.capacity(); }

    public static void main(String[] args) throws IOException {
        ByteBuffer buf = ByteBuffer.allocate(16);
        System.out.println("new:        " + state(buf));
        buf.putInt(0xCAFEBABE).putShort((short) 258).put((byte) 7);
        System.out.println("after puts: " + state(buf));
        buf.flip();                                                    // switch from writing to reading
        System.out.println("after flip: " + state(buf) + "  bytes: " + HexFormat.ofDelimiter(" ").formatHex(buf.array(), 0, buf.limit()));
        System.out.printf("read back: %x %d %d, remaining %d%n", buf.getInt(), buf.getShort(), buf.get(), buf.remaining());
        ByteBuffer le = ByteBuffer.allocate(4).order(ByteOrder.LITTLE_ENDIAN).putInt(0xCAFEBABE);
        System.out.println("same int little-endian: " + HexFormat.ofDelimiter(" ").formatHex(le.array()) + "  (native order here: " + ByteOrder.nativeOrder() + ")");

        Path file = Path.of("mapped.bin");
        try (FileChannel ch = FileChannel.open(file, StandardOpenOption.CREATE, StandardOpenOption.READ, StandardOpenOption.WRITE)) {
            MappedByteBuffer map = ch.map(FileChannel.MapMode.READ_WRITE, 0, 8L * 1_000_000);   // 8 MB of longs
            for (int i = 0; i < 1_000_000; i++) map.putLong(i * 8, (long) i * i);
            map.force();                                               // flush dirty pages to the file
            ByteBuffer one = ByteBuffer.allocate(8);
            ch.read(one, 8L * 999);                                    // positional read through the channel
            System.out.println("file size " + ch.size() + " bytes; the 999th long read via the channel = " + one.flip().getLong()
                    + ", via the mapping = " + map.getLong(8 * 999));
        }
    }
}
```

```console
$ java Buffers.java
new:        position=0 limit=16 capacity=16
after puts: position=7 limit=16 capacity=16
after flip: position=0 limit=7 capacity=16  bytes: ca fe ba be 01 02 07
read back: cafebabe 258 7, remaining 0
same int little-endian: be ba fe ca  (native order here: LITTLE_ENDIAN)
file size 8000000 bytes; the 999th long read via the channel = 998001, via the mapping = 998001
```

### Deep & Tricky

**DT-28. Forgetting `flip()` is the classic `ByteBuffer` bug: after writing, position is at the end, so reading (or writing the buffer to a channel) sees nothing.** *(Library behaviour: `java.nio.Buffer`)* A buffer has one position used for both writing and reading. `flip()` sets limit = position and position = 0, making exactly the written bytes readable; `clear()` prepares for writing again (it resets the markers, not the contents); `compact()` keeps unread bytes and prepares to append. `channel.write(buf)` on an unflipped buffer writes the *unwritten remainder* — usually zeros or stale data.

**DT-29. A memory-mapped file cannot be unmapped explicitly; the mapping lives until the buffer is garbage collected.** *(Implementation behaviour: `MappedByteBuffer`; fixed by the Foreign Function & Memory API)* There is no `unmap()`. Until the GC collects the `MappedByteBuffer`, the mapping (and on Windows, a lock on the file that prevents deleting it) remains. Java 22's `Arena`/`MemorySegment` API (final in JEP 454) provides deterministic unmapping: `try (Arena arena = Arena.ofConfined()) { MemorySegment seg = channel.map(mode, 0, size, arena); … }`. On Java 21 that API is still a preview; plan mapped-file lifetimes accordingly.

## Unit 8.5 — Serialization

**One-line summary:** Java's built-in **serialization** (`implements Serializable`, `ObjectOutputStream`) writes an object graph to bytes and reconstructs it — *without calling constructors*, so invariants are bypassed, and deserializing untrusted bytes can execute attacker-chosen code paths ("gadget chains"). It is also fragile across class changes (`serialVersionUID`). For data exchange use explicit formats: JSON, Protocol Buffers, or a hand-written binary format; if you must deserialize, apply an `ObjectInputFilter` allow-list.

```java
// file: Serial.java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.InvalidClassException;
import java.io.ObjectInputFilter;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class Serial {
    static class Range implements Serializable {
        private static final long serialVersionUID = 1L;
        final int lo, hi;
        transient int cachedLength;                                   // transient: not serialized
        Range(int lo, int hi) {
            if (lo > hi) throw new IllegalArgumentException("lo > hi");   // the invariant
            this.lo = lo;
            this.hi = hi;
            this.cachedLength = hi - lo;
        }
        @Override public String toString() { return "Range[" + lo + ".." + hi + ", cachedLength=" + cachedLength + "]"; }
    }

    static byte[] serialize(Object o) throws Exception {
        var bytes = new ByteArrayOutputStream();
        try (var out = new ObjectOutputStream(bytes)) {
            out.writeObject(o);
        }
        return bytes.toByteArray();
    }

    public static void main(String[] args) throws Exception {
        byte[] data = serialize(new Range(1, 5));
        System.out.println(data.length + " bytes for two ints");
        try (var in = new ObjectInputStream(new ByteArrayInputStream(data))) {
            System.out.println("round trip: " + in.readObject() + "   (transient field lost: the constructor did not run)");
        }
        // an attacker edits the bytes. The last 8 bytes are the int fields, in alphabetical order: hi, then lo.
        byte[] forged = data.clone();
        forged[forged.length - 1] = 9;                                 // lo = 9, hi stays 5
        try (var in = new ObjectInputStream(new ByteArrayInputStream(forged))) {
            System.out.println("forged:     " + in.readObject() + "   (invariant lo <= hi bypassed)");
        }
        try (var in = new ObjectInputStream(new ByteArrayInputStream(data))) {
            in.setObjectInputFilter(ObjectInputFilter.Config.createFilter("java.base/*;!*"));   // allow only JDK core classes
            in.readObject();
        } catch (InvalidClassException e) {
            System.out.println("with an allow-list filter: " + e.getMessage());
        }
    }
}
```

```console
$ java Serial.java
51 bytes for two ints
round trip: Range[1..5, cachedLength=0]   (transient field lost: the constructor did not run)
forged:     Range[9..5, cachedLength=0]   (invariant lo <= hi bypassed)
with an allow-list filter: filter status: REJECTED
```

### Pattern

**P-8. Never deserialize untrusted data with Java serialization; define explicit formats with versions and checksums.** *(Practice; Effective Java, Items 85–90; OWASP)* Records serialize more safely (they are reconstructed through their canonical constructor, so invariants hold), but the format remains Java-only and opaque. For interchange use JSON (Jackson), Protocol Buffers, or Avro; for your own files, write fields explicitly with `DataOutputStream` or `ByteBuffer`, include a magic number, a format version, and a checksum, and validate everything on read. *Trade-off:* explicit formats need code for every field; that code is also where versioning and validation live.

<!-- expanded -->
### Deep & Tricky

**DT-30. Java deserialisation runs code: `ObjectInputStream.readObject` instantiates whatever classes the byte stream names (if they are on the class path) and calls their `readObject`/`readResolve` methods, so deserialising untrusted data has led to remote code execution in many products ("gadget chains"). Never deserialise untrusted input with `ObjectInputStream`; if you must, install an `ObjectInputFilter` that allow-lists the expected classes.** *(Library behaviour: Java Serialization Specification; `ObjectInputFilter`, JEP 290 and JEP 415)* Use a data format such as JSON or Protocol Buffers for anything that crosses a trust boundary.

```java
// file: SerialFilter.java
import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class SerialFilter {
    record Point(int x, int y) implements Serializable {}

    static byte[] serialise(Object o) throws IOException {
        var bos = new ByteArrayOutputStream();
        try (var out = new ObjectOutputStream(bos)) { out.writeObject(o); }
        return bos.toByteArray();
    }

    static Object read(byte[] data) throws IOException, ClassNotFoundException {
        try (var in = new ObjectInputStream(new ByteArrayInputStream(data))) {
            in.setObjectInputFilter(ObjectInputFilter.Config.createFilter("SerialFilter$Point;java.lang.*;!*"));
            return in.readObject();
        }
    }

    public static void main(String[] args) throws Exception {
        System.out.println("allowed: " + read(serialise(new Point(1, 2))));
        try {
            read(serialise(new ArrayList<>(List.of(1))));          // not on the allow-list
        } catch (InvalidClassException e) {
            System.out.println("rejected: " + e.getMessage());
        }
    }
}
```

```console
$ java SerialFilter.java
allowed: Point[x=1, y=2]
rejected: filter status: REJECTED
```

### Mastery check

1. **Predict.** A `ByteBuffer.allocate(8)` has `putInt(1)` called once. Without `flip()`, what does `buf.remaining()` return, and what does `buf.getInt()` return?
2. **Predict.** A program writes 1,000 lines to a `BufferedWriter` wrapping a `FileWriter`, never closes or flushes it, and exits normally. How much is in the file?

<!-- expanded -->
## Unit 8.6 — Regular expressions

**One-line summary:** `Pattern.compile` turns a regex into a reusable, thread-safe matcher factory; `Matcher` finds matches (`matches` for the whole input, `find` for each occurrence), exposes numbered and named groups, and supports replacement — while `String.matches`, `split`, and `replaceAll` are shortcuts that recompile the pattern on every call.

Log parsing, input validation, extracting fields from text: regular expressions are the standard tool, and Java's `java.util.regex` is a full Perl-style engine.

```java
// file: Regex.java
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Regex {
    private static final Pattern LOG = Pattern.compile(
        "(?<date>\\d{4}-\\d{2}-\\d{2}) (?<level>ERROR|WARN|INFO) \\[(?<thread>[^\\]]+)] (?<msg>.*)");
    private static final Pattern EMAIL = Pattern.compile("[\\w.+-]+@[\\w-]+(\\.[\\w-]+)+");

    public static void main(String[] args) {
        List<String> lines = List.of(
            "2026-11-16 ERROR [worker-3] disk full on /var",
            "garbage line",
            "2026-11-16 INFO [main] started in 412 ms");
        for (String line : lines) {
            Matcher m = LOG.matcher(line);
            if (m.matches()) {
                System.out.printf("%-5s %-8s %s%n", m.group("level"), m.group("thread"), m.group("msg"));
            } else {
                System.out.println("no match: " + line);
            }
        }

        String text = "write to ada@example.org or b.smith+java@mail.co.in today";
        Matcher m = EMAIL.matcher(text);
        while (m.find()) System.out.println("found " + m.group() + " at " + m.start() + "-" + m.end());
        System.out.println(EMAIL.matcher(text).replaceAll("<hidden>"));
        System.out.println(EMAIL.matcher(text).replaceAll(r -> r.group().replaceAll("(?<=.).(?=[^@]*@)", "*")));

        System.out.println("2026-11-16".replaceAll("(\\d+)-(\\d+)-(\\d+)", "$3/$2/$1"));
        System.out.println(List.of("a1b22c333".split("\\d+")));
        System.out.println("case-insensitive: " + Pattern.compile("^java", Pattern.CASE_INSENSITIVE).matcher("JAVA 21").find());
        System.out.println("literal dot: " + "a.b".matches("a\\.b") + ", any char: " + "axb".matches("a.b")
                           + ", Pattern.quote: " + "a.b".split(Pattern.quote(".")).length);
    }
}
```

```console
$ java Regex.java
ERROR worker-3 disk full on /var
no match: garbage line
INFO  main     started in 412 ms
found ada@example.org at 9-24
found b.smith+java@mail.co.in at 28-51
write to <hidden> or <hidden> today
write to a**@example.org or b***********@mail.co.in today
16/11/2026
[a, b, c]
case-insensitive: true
literal dot: true, any char: true, Pattern.quote: 2
```

Line by line, the points to remember: in a Java string literal every regex backslash is written twice (`\\d` is the regex `\d`); named groups `(?<name>...)` make code readable and survive adding groups; `matches()` must match the *whole* input while `find()` searches for the next occurrence; in a replacement string `$1` refers to a group (so a literal `$` must be escaped, or passed through `Matcher.quoteReplacement`); `replaceAll` with a function (Java 9+) computes each replacement; and `split` takes a regex, so splitting on `"."` needs `Pattern.quote(".")` or `"\\."`.

### Performance and safety

`Pattern.compile` is the expensive step. `String.matches(regex)`, `String.split(regex)` (except for single-character literals, which have a fast path), and `String.replaceAll` compile their pattern *every time* — inside a loop over a million lines that is a million compilations. Keep compiled patterns in `static final` fields; a `Pattern` is immutable and safe to share between threads (a `Matcher` is not).

### Deep & Tricky

**DT-31. Java's regex engine backtracks, and some patterns take *exponential* time on input that almost matches — ReDoS (regular-expression denial of service). Since JDK 9 the engine remembers failed positions inside simple nested loops, so the textbook example `(a+)+$` is fast on Java 21; but that optimisation does not cover every pattern — patterns with backreferences, for example — and those still explode, adding a factor of about eight for every three extra characters below.** *(Library behaviour: `java.util.regex` is a backtracking implementation with no documented complexity bound; the memoisation was added in JDK 9, JDK-6328855)* Do not rely on the engine to save you: avoid ambiguous nested repetition, bound input length before matching, and use a linear-time engine such as RE2/J when patterns or inputs come from users.

```java
// file: Redos.java
import java.util.regex.Pattern;

public class Redos {
    static String time(Pattern p, String input) {
        long t0 = System.nanoTime();
        boolean found = p.matcher(input).find();
        long ms = (System.nanoTime() - t0) / 1_000_000;
        return (found ? "match" : "no match") + (ms < 20 ? " (under 20 ms)" : " (" + (ms < 200 ? "tens" : ms < 2000 ? "hundreds" : "thousands") + " of ms)");
    }
    public static void main(String[] args) {
        Pattern textbook = Pattern.compile("(a+)+$");
        Pattern backref = Pattern.compile("^((a+)+)\\2$");          // "some a's, then the same group again"
        for (int n = 18; n <= 27; n += 3) {
            String input = "a".repeat(n) + "!";                    // almost matches
            System.out.printf("n=%d: (a+)+$ %s; ^((a+)+)\\2$ %s%n", n, time(textbook, input), time(backref, input));
        }
    }
}
```

```console
$ java Redos.java
n=18: (a+)+$ no match (under 20 ms); ^((a+)+)\2$ no match (under 20 ms)
n=21: (a+)+$ no match (under 20 ms); ^((a+)+)\2$ no match (tens of ms)
n=24: (a+)+$ no match (under 20 ms); ^((a+)+)\2$ no match (hundreds of ms)
n=27: (a+)+$ no match (under 20 ms); ^((a+)+)\2$ no match (thousands of ms)
```

The textbook pattern stays instant because of the JDK 9 memoisation; the backreference version grows by roughly an order of magnitude every three characters, and the same engine would need hours for a few dozen. Running the textbook pattern on Java 8 shows the original explosion.

### Mastery check

1. **Write.** A `static final Pattern` that validates Indian PIN codes (six digits, first digit 1–9) and a method that extracts every PIN code from a paragraph of text.
2. **Predict.** What does `"a,b;c d".split("[,; ]")` return? And `"x".split(",")`?
3. **Explain.** Why is `line.matches("\\d+")` inside a loop over a large file slow, and how do you fix it?

## Unit 8.7 — Dates and times with `java.time`

**One-line summary:** `java.time` separates *machine time* (`Instant`, a point on the UTC timeline) from *human time* (`LocalDate`, `LocalDateTime`, `ZonedDateTime` with a `ZoneId`), and *amounts* (`Duration` for seconds, `Period` for calendar units); all types are immutable, and `DateTimeFormatter` is thread-safe — fixing the problems of `java.util.Date` and `SimpleDateFormat`.

Getting time wrong produces bugs that appear twice a year (daylight-saving changes) or once a year (31 December). `java.time` (Java 8, designed by the author of Joda-Time) makes the different meanings of "time" different types, so the compiler catches many mistakes.

```java
// file: Times.java
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAdjusters;

public class Times {
    public static void main(String[] args) {
        LocalDate d = LocalDate.of(2024, 1, 31);
        System.out.println("Jan 31 + 1 month = " + d.plusMonths(1) + " (clamped to the month's end)");
        System.out.println("2024 is a leap year: " + d.isLeapYear() + ", day of week " + d.getDayOfWeek());
        System.out.println("last day of Feb 2025: " + LocalDate.of(2025, 2, 1).with(TemporalAdjusters.lastDayOfMonth()));
        Period age = Period.between(LocalDate.of(1990, 8, 15), LocalDate.of(2026, 11, 16));
        System.out.println("age: " + age.getYears() + " years " + age.getMonths() + " months " + age.getDays() + " days");
        System.out.println("days between: " + ChronoUnit.DAYS.between(LocalDate.of(2026, 1, 1), LocalDate.of(2026, 11, 16)));

        ZoneId india = ZoneId.of("Asia/Kolkata"), berlin = ZoneId.of("Europe/Berlin");
        ZonedDateTime meeting = ZonedDateTime.of(2026, 3, 29, 14, 0, 0, 0, india);
        System.out.println("meeting in India: " + meeting);
        System.out.println("same instant in Berlin: " + meeting.withZoneSameInstant(berlin));
        Instant i = meeting.toInstant();
        System.out.println("as an Instant (UTC): " + i + ", epoch seconds " + i.getEpochSecond());

        ZonedDateTime beforeDst = ZonedDateTime.of(2026, 3, 29, 1, 30, 0, 0, berlin);   // clocks jump 02:00 -> 03:00
        System.out.println("Berlin 01:30 + 1 hour (Duration) = " + beforeDst.plus(Duration.ofHours(1)).toLocalTime()
                           + ", + 1 day (Period) = " + beforeDst.plusDays(1).toLocalTime());
        System.out.println("Berlin 02:30 does not exist; of() gives " + ZonedDateTime.of(2026, 3, 29, 2, 30, 0, 0, berlin).toLocalTime());

        DateTimeFormatter f = DateTimeFormatter.ofPattern("dd MMM uuuu, HH:mm");
        System.out.println(f.format(meeting) + " | parsed back: " + LocalDateTime.parse("16 Nov 2026, 09:05", f));
        System.out.println("ISO: " + LocalDate.parse("2026-11-16").atStartOfDay());
    }
}
```

```console
$ java Times.java
Jan 31 + 1 month = 2024-02-29 (clamped to the month's end)
2024 is a leap year: true, day of week WEDNESDAY
last day of Feb 2025: 2025-02-28
age: 36 years 3 months 1 days
days between: 319
meeting in India: 2026-03-29T14:00+05:30[Asia/Kolkata]
same instant in Berlin: 2026-03-29T10:30+02:00[Europe/Berlin]
as an Instant (UTC): 2026-03-29T08:30:00Z, epoch seconds 1774773000
Berlin 01:30 + 1 hour (Duration) = 03:30, + 1 day (Period) = 01:30
Berlin 02:30 does not exist; of() gives 03:30
29 Mar 2026, 14:00 | parsed back: 2026-11-16T09:05
ISO: 2026-11-16T00:00
```

Read the daylight-saving lines carefully. Berlin moved its clocks from 02:00 to 03:00 on 29 March 2026: adding a `Duration` of one hour to 01:30 gives 03:30 (one real hour later), adding a `Period` of one day keeps the wall-clock time 01:30 on the next day, and asking for 02:30 — a time that never happened that night — gives 03:30. Which one you want depends on the meaning: a timeout is a `Duration`, "same time tomorrow" is a `Period`.

### Which type to use

| You are storing… | Use |
| --- | --- |
| When something happened (logs, database timestamps) | `Instant` (store in UTC) |
| A birthday, a due date | `LocalDate` |
| A meeting that happens at a local time in a city | `ZonedDateTime` (or `LocalDateTime` + `ZoneId`) |
| A fixed offset from an external API | `OffsetDateTime` |
| How long something took | `Duration` |
| "3 months" in calendar terms | `Period` |

For measuring elapsed time inside a program, `System.nanoTime()` is still right: `Instant.now()` follows the wall clock, which can jump when the system time is corrected.

### Deep & Tricky

**DT-32. In `DateTimeFormatter` patterns, `yyyy` is the *year of era* and `YYYY` is the *week-based year*: for 29–31 December in some years, `YYYY` prints the *next* year. `Y` in place of `y` (and `D` — day of year — in place of `d`) produces dates that are correct for most of the year and wrong for a few days, which is why the bug survives testing.** *(Library behaviour: `DateTimeFormatter` pattern letters; `u` is the proleptic year and avoids era issues)*

```java
// file: WeekYear.java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.Locale;

public class WeekYear {
    public static void main(String[] args) {
        LocalDate d = LocalDate.of(2025, 12, 29);
        for (String p : new String[] {"yyyy-MM-dd", "YYYY-MM-dd", "uuuu-MM-dd", "yyyy-MM-DD"}) {
            System.out.println(p + " -> " + DateTimeFormatter.ofPattern(p, Locale.ROOT).format(d));
        }
    }
}
```

```console
$ java WeekYear.java
yyyy-MM-dd -> 2025-12-29
YYYY-MM-dd -> 2026-12-29
uuuu-MM-dd -> 2025-12-29
yyyy-MM-DD -> 2025-12-363
```

`YYYY` gave 2026 for a date in December 2025, and `DD` (day of year) gave 363 as the "day".

### Mastery check

1. **Write.** Given a list of `Instant` login times and a user's `ZoneId`, count logins per local calendar day.
2. **Predict.** `LocalDate.of(2025, 3, 31).minusMonths(1)` — what date?
3. **Explain.** Why is it wrong to store "the meeting is at 10:00 in Mumbai every Monday" as an `Instant`?

## Chapter 8 Project — A binary event log with an index

**Goal:** design and implement a compact binary file format for time-stamped events — a header with a magic number and version; fixed-layout records with a CRC-32 each; an append-only writer using `FileChannel`; a reader that validates checksums and stops cleanly at a truncated or corrupt record; and a sparse in-memory index for fast time-range queries by seeking. Compare size and speed with a text (CSV) version of the same data.

```java
// file: evlog/EventLog.java
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.util.ArrayList;
import java.util.List;
import java.util.TreeMap;
import java.util.zip.CRC32;

/**
 * File:   magic "EVLG" (4) | version u16 | reserved u16
 * Record: timestampMillis i64 | sensor u16 | kind u8 | payloadLen u8 | value f64 | payload bytes | crc32 u32
 * All integers big-endian. The CRC covers the record from the timestamp to the end of the payload.
 */
public class EventLog {
    static final int MAGIC = 0x45564C47, HEADER = 8, FIXED = 8 + 2 + 1 + 1 + 8;
    public record Event(long timestamp, int sensor, int kind, double value, String note) {}

    static final class Writer implements AutoCloseable {
        private final FileChannel ch;
        private final ByteBuffer buf = ByteBuffer.allocate(1 << 16);
        private final CRC32 crc = new CRC32();

        Writer(Path path) throws IOException {
            ch = FileChannel.open(path, StandardOpenOption.CREATE, StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING);
            buf.putInt(MAGIC).putShort((short) 1).putShort((short) 0);
        }

        void append(Event e) throws IOException {
            byte[] note = e.note().getBytes(StandardCharsets.UTF_8);
            if (note.length > 255) throw new IllegalArgumentException("note too long");
            if (buf.remaining() < FIXED + note.length + 4) flush();
            int start = buf.position();
            buf.putLong(e.timestamp()).putShort((short) e.sensor()).put((byte) e.kind()).put((byte) note.length).putDouble(e.value()).put(note);
            crc.reset();
            crc.update(buf.array(), start, buf.position() - start);
            buf.putInt((int) crc.getValue());
        }

        void flush() throws IOException {
            buf.flip();                                              // DT-28
            while (buf.hasRemaining()) ch.write(buf);
            buf.clear();
        }

        @Override public void close() throws IOException {
            flush();
            ch.force(true);                                          // durable before close
            ch.close();
        }
    }

    static final class Reader implements AutoCloseable {
        private final FileChannel ch;
        final TreeMap<Long, Long> index = new TreeMap<>();           // timestamp -> file offset, every 1024th record
        long validRecords, validBytes;
        String stopReason = "end of file";

        Reader(Path path) throws IOException {
            ch = FileChannel.open(path, StandardOpenOption.READ);
            ByteBuffer h = ByteBuffer.allocate(HEADER);
            ch.read(h, 0);
            h.flip();
            if (h.remaining() < HEADER || h.getInt() != MAGIC) throw new IOException("not an event log: bad magic");
            int version = h.getShort();
            if (version != 1) throw new IOException("unsupported version " + version);
            scan();
        }

        /** One pass over the file: validates every record and builds the sparse index. */
        private void scan() throws IOException {
            long pos = HEADER, size = ch.size();
            ByteBuffer b = ByteBuffer.allocate(FIXED + 255 + 4);
            CRC32 crc = new CRC32();
            while (pos < size) {
                b.clear();
                ch.read(b, pos);
                b.flip();
                if (b.remaining() < FIXED) { stopReason = "truncated record header at offset " + pos; break; }
                int len = b.get(11) & 0xFF;
                int total = FIXED + len + 4;
                if (b.remaining() < total) { stopReason = "truncated record at offset " + pos; break; }
                crc.reset();
                crc.update(b.array(), 0, FIXED + len);
                if ((int) crc.getValue() != b.getInt(FIXED + len)) { stopReason = "checksum mismatch at offset " + pos; break; }
                if (validRecords % 1024 == 0) index.put(b.getLong(0), pos);
                validRecords++;
                pos += total;
            }
            validBytes = pos;
        }

        private Event readAt(long pos, ByteBuffer b) throws IOException {
            b.clear();
            ch.read(b, pos);
            b.flip();
            long ts = b.getLong();
            int sensor = b.getShort() & 0xFFFF, kind = b.get() & 0xFF, len = b.get() & 0xFF;
            double value = b.getDouble();
            String note = new String(b.array(), FIXED, len, StandardCharsets.UTF_8);
            return new Event(ts, sensor, kind, value, note);
        }

        /** Events with from <= timestamp < to, found by seeking to the nearest indexed offset. */
        List<Event> range(long from, long to) throws IOException {
            List<Event> out = new ArrayList<>();
            var floor = index.floorEntry(from);
            long pos = floor == null ? HEADER : floor.getValue();
            ByteBuffer b = ByteBuffer.allocate(FIXED + 255 + 4);
            while (pos < validBytes) {
                Event e = readAt(pos, b);
                if (e.timestamp() >= to) break;
                if (e.timestamp() >= from) out.add(e);
                pos += FIXED + e.note().getBytes(StandardCharsets.UTF_8).length + 4;
            }
            return out;
        }

        @Override public void close() throws IOException { ch.close(); }
    }

    public static void main(String[] args) throws IOException {
        Path bin = Path.of("events.evl"), csv = Path.of("events.csv");
        int n = 1_000_000;
        java.util.Random r = new java.util.Random(11);
        String[] notes = {"", "", "", "ok", "retry", "sensor warmed up", "calibration drift détecté"};
        long t0 = 1_780_000_000_000L;

        long t = System.nanoTime();
        try (Writer w = new Writer(bin)) {
            for (int i = 0; i < n; i++) w.append(new Event(t0 + i * 250L, r.nextInt(500), r.nextInt(4), 20 + r.nextGaussian() * 3, notes[r.nextInt(notes.length)]));
        }
        long binWrite = (System.nanoTime() - t) / 1_000_000;
        r = new java.util.Random(11);
        t = System.nanoTime();
        try (BufferedWriter w = Files.newBufferedWriter(csv)) {
            for (int i = 0; i < n; i++) {
                w.write((t0 + i * 250L) + "," + r.nextInt(500) + "," + r.nextInt(4) + "," + (20 + r.nextGaussian() * 3) + "," + notes[r.nextInt(notes.length)] + "\n");
            }
        }
        long csvWrite = (System.nanoTime() - t) / 1_000_000;
        System.out.printf("wrote %,d events: binary %,d bytes in %d ms; CSV %,d bytes in %d ms%n", n, Files.size(bin), binWrite, Files.size(csv), csvWrite);

        t = System.nanoTime();
        try (Reader rd = new Reader(bin)) {
            long scan = (System.nanoTime() - t) / 1_000_000;
            long from = t0 + 600_000L * 250, to = from + 2_000;                 // a 2-second window deep in the file
            t = System.nanoTime();
            List<Event> hits = rd.range(from, to);
            long query = (System.nanoTime() - t) / 1_000;
            System.out.printf("scan + CRC check of %,d records: %d ms; index entries: %d; range query: %d events in %d µs%n",
                    rd.validRecords, scan, rd.index.size(), hits.size(), query);
            System.out.println("first hit: " + hits.get(0));
        }
        t = System.nanoTime();
        long from = t0 + 600_000L * 250, to = from + 2_000;
        long csvHits;
        try (var lines = Files.lines(csv)) {
            csvHits = lines.filter(l -> { long ts = Long.parseLong(l, 0, l.indexOf(','), 10); return ts >= from && ts < to; }).count();
        }
        System.out.printf("same query on CSV (full scan): %d events in %d ms%n", csvHits, (System.nanoTime() - t) / 1_000_000);

        // damage the file: truncate mid-record, then flip one bit
        long size = Files.size(bin);
        try (FileChannel ch = FileChannel.open(bin, StandardOpenOption.WRITE)) {
            ch.truncate(size - 7);
        }
        try (Reader rd = new Reader(bin)) {
            System.out.printf("after truncating 7 bytes: %,d valid records, stopped: %s%n", rd.validRecords, rd.stopReason);
        }
        try (FileChannel ch = FileChannel.open(bin, StandardOpenOption.READ, StandardOpenOption.WRITE)) {
            ByteBuffer one = ByteBuffer.allocate(1);
            ch.read(one, 5_000_000);
            one.flip();
            one.put(0, (byte) (one.get(0) ^ 0x10));
            ch.write(one, 5_000_000);
        }
        try (Reader rd = new Reader(bin)) {
            System.out.printf("after flipping one bit at offset 5,000,000: %,d valid records, stopped: %s%n", rd.validRecords, rd.stopReason);
        }
        Files.writeString(Path.of("fake.evl"), "this is not a log");
        try (Reader rd = new Reader(Path.of("fake.evl"))) {
            System.out.println("?");
        } catch (IOException e) {
            System.out.println("wrong file: " + e.getMessage());
        }
    }
}
```

```console
$ cd evlog && java EventLog.java
wrote 1,000,000 events: binary 31,133,086 bytes in 187 ms; CSV 46,535,019 bytes in 370 ms
scan + CRC check of 1,000,000 records: 333 ms; index entries: 977; range query: 8 events in 2121 µs
first hit: Event[timestamp=1780150000000, sensor=377, kind=3, value=23.62154946352011, note=calibration drift détecté]
same query on CSV (full scan): 8 events in 179 ms
after truncating 7 bytes: 999,999 valid records, stopped: truncated record header at offset 31133062
after flipping one bit at offset 5,000,000: 160,606 valid records, stopped: checksum mismatch at offset 4999999
wrong file: not an event log: bad magic
```

The binary format is about two-thirds the size of the CSV, and — more importantly — it is *seekable*: the sparse index (one entry per 1,024 records) lets a time-range query jump to within 1,024 records of its start, so the query took a millisecond or two (on its first, unwarmed execution) where the CSV needed a full scan of about 200 ms — roughly a hundred times longer, and growing with the file. Every design rule of the chapter is visible: explicit byte order and field widths (no dependence on the platform), a magic number and version (so a wrong or future file is rejected with a clear message), a checksum per record (so truncation and bit rot are *detected* and reading stops at the last good record), buffered positional I/O through a `FileChannel`, and `force(true)` before the writer reports success.

### Review checklist

- Why does the record store the payload length *before* the payload, and why must the reader bound it (here: one byte, at most 255)?
- After the bit flip, records after the corrupt one are not read. How could the format allow resynchronising (e.g. a sync marker every N records)?
- What would change to support appending to an existing log safely after a crash? (Scan, truncate to `validBytes`, then append — the technique of the Rust book's capstone.)

## Answers to the Chapter 8 prediction questions

**8.5 Q1.** `remaining()` is 4 (limit 8 − position 4), and `getInt()` reads the *next* four bytes — the unwritten zeros — returning 0, not 1. After `flip()`, `remaining()` would be 4 and `getInt()` would return 1.

**8.5 Q2.** Possibly nothing, or a multiple of the 8 KiB buffer: only full buffers were written; the last partial buffer is lost because nothing flushed it. The JVM does not flush or close streams at exit. Use try-with-resources.

- **Unit 8.6, Q2:** `[a, b, c, d]`; and `[x]` — a string without the delimiter gives a one-element array.
- **Unit 8.7, Q2:** `2025-02-28` — there is no 31 February, so the result is clamped to the last day of the month.

# Chapter 9 — Build and Test: Gradle, Maven, JUnit, and Test Doubles

*Goal: build Java projects the way teams do — a build tool that compiles, tests, and packages reproducibly; unit tests that are fast, isolated, and readable; test doubles for dependencies; and a feel for what makes code testable.*

> **A note on what was run.** This book's build machine has no access to Maven Central, the repository from which Gradle and Maven download libraries. Everything in this chapter that is shown with output was run **offline**: Gradle 8.14 itself, and **JUnit 4.13.2** (whose jar ships inside the Gradle distribution). **JUnit 5, AssertJ, Mockito, and Maven** are shown as source and configuration marked *not run here*. The concepts — and nearly all of the code — are the same; the units give the mapping.

**What you will be able to do after this chapter**

- Explain what a build tool does: dependency resolution, the build lifecycle, incremental builds.
- Write Gradle and Maven build files for a Java project, with tests.
- Write unit tests with JUnit: assertions, fixtures, parameterised tests, expected exceptions.
- Replace dependencies with hand-written fakes and stubs, and with Mockito.
- Design code for testability: dependency injection, clocks, pure functions.

## Topic 9A — Build tools

## Unit 9.1 — What a build tool does

**One-line summary:** A build tool turns sources into artifacts: it **resolves dependencies** (downloads libraries, and *their* dependencies, by `group:artifact:version` coordinates from a repository such as Maven Central), runs the build steps in order (compile → process resources → compile tests → run tests → package), and skips steps whose inputs have not changed. **Maven** describes the build declaratively in `pom.xml` with a fixed lifecycle; **Gradle** describes it in a Groovy or Kotlin script with a graph of tasks and an aggressive build cache.

A Maven build for the chapter's project (*not run here*):

```xml
<!-- pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>pricing</artifactId>
  <version>1.0.0</version>
  <properties>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.11.3</version>
      <scope>test</scope>            <!-- on the classpath for tests only -->
    </dependency>
  </dependencies>
</project>
```

`mvn test` compiles and runs tests; `mvn package` builds the JAR into `target/`; `mvn dependency:tree` shows the resolved dependency graph. The standard layout — `src/main/java`, `src/main/resources`, `src/test/java` — is shared by Maven and Gradle.

### Deep & Tricky

**DT-33. When two dependencies need different versions of the same library, Maven picks the *nearest* declaration and Gradle picks the *highest* version — and either can break at run time.** *(Tool behaviour: dependency mediation)* Only one version of a class can be on the classpath. Maven's "nearest wins" may silently choose an *older* version than some library was compiled against (`NoSuchMethodError` at run time); Gradle's "highest wins" may choose a newer, incompatible one. Inspect the graph (`mvn dependency:tree`, `gradle dependencies`), pin versions centrally (`<dependencyManagement>`, Gradle version catalogs or a BOM), and fail the build on conflicts in critical projects.

## Unit 9.2 — A Gradle project

**One-line summary:** A Gradle build is defined by `settings.gradle` (the project's name and modules) and `build.gradle` (plugins, repositories, dependencies, task configuration). The `java` plugin adds `compileJava`, `test`, `jar`, and `build`; a **toolchain** fixes the JDK version regardless of which JDK runs Gradle.

```groovy
// file: pricing/settings.gradle
rootProject.name = 'pricing'
```

```groovy
// file: pricing/build.gradle
plugins {
    id 'java'
}

group = 'com.example'
version = '1.0.0'

java {
    toolchain { languageVersion = JavaLanguageVersion.of(21) }
}

repositories {
    // normally: mavenCentral()
    flatDir { dirs 'libs' }                    // offline: jars copied into libs/
}

dependencies {
    // normally: testImplementation 'org.junit.jupiter:junit-jupiter:5.11.3'  and  test { useJUnitPlatform() }
    testImplementation files('libs/junit-4.13.2.jar', 'libs/hamcrest-core-1.3.jar')
}

tasks.withType(JavaCompile).configureEach {
    options.compilerArgs += ['-Xlint:all', '-Werror']
    options.encoding = 'UTF-8'
}

test {
    useJUnit()
    testLogging { events 'failed'; exceptionFormat 'short' }
}

jar {
    manifest { attributes 'Main-Class': 'com.example.pricing.Main' }
}
```

## Topic 9B — Unit testing

## Unit 9.3 — Writing tests with JUnit

**One-line summary:** A test is a method annotated `@Test` that sets up inputs (**arrange**), calls the code (**act**), and checks results with assertions (**assert**). Each test method runs on a **fresh instance** of the test class, so tests cannot depend on each other through fields. Good tests are fast, independent, deterministic, and named after the behaviour they check.

| | JUnit 4 (run in this chapter) | JUnit 5 "Jupiter" (current) |
| --- | --- | --- |
| Test annotation | `org.junit.Test` | `org.junit.jupiter.api.Test` |
| Fixtures | `@Before`, `@After`, `@BeforeClass` | `@BeforeEach`, `@AfterEach`, `@BeforeAll` |
| Assertions | `org.junit.Assert.assertEquals(expected, actual)` | `Assertions.assertEquals(expected, actual)`, `assertAll` |
| Expected exception | `assertThrows` (4.13+) | `assertThrows` |
| Parameterised | `@RunWith(Parameterized.class)` | `@ParameterizedTest` + `@CsvSource`, `@MethodSource` |
| Skip | `@Ignore` | `@Disabled` |
| Visibility | test classes and methods must be `public` | package-private is fine |

The code under test is a small pricing library:

```java
// file: pricing/src/main/java/com/example/pricing/Money.java
package com.example.pricing;

import java.math.BigDecimal;
import java.math.RoundingMode;

/** An amount in cents. */
public record Money(long cents) implements Comparable<Money> {
    public static final Money ZERO = new Money(0);

    public static Money of(String amount) {
        return new Money(new BigDecimal(amount).movePointRight(2).longValueExact());
    }
    public Money plus(Money o) { return new Money(Math.addExact(cents, o.cents)); }
    public Money minus(Money o) { return new Money(Math.subtractExact(cents, o.cents)); }
    public Money times(int n) { return new Money(Math.multiplyExact(cents, n)); }
    /** Applies a percentage, rounding half-up to the cent. */
    public Money percent(BigDecimal pct) {
        return new Money(BigDecimal.valueOf(cents).multiply(pct).divide(BigDecimal.valueOf(100), 0, RoundingMode.HALF_UP).longValueExact());
    }
    @Override public int compareTo(Money o) { return Long.compare(cents, o.cents); }
    @Override public String toString() { return "%s%d.%02d".formatted(cents < 0 ? "-" : "", Math.abs(cents / 100), Math.abs(cents % 100)); }
}
```

```java
// file: pricing/src/main/java/com/example/pricing/PriceCalculator.java
package com.example.pricing;

import java.math.BigDecimal;
import java.time.Clock;
import java.time.DayOfWeek;
import java.time.LocalDate;
import java.util.List;
import java.util.Objects;

/** Computes order totals: volume discount, weekend promotion, tax, shipping. */
public final class PriceCalculator {
    public record Line(String sku, Money unitPrice, int quantity) {
        public Line {
            Objects.requireNonNull(sku);
            if (quantity <= 0) throw new IllegalArgumentException("quantity must be positive: " + quantity);
            if (unitPrice.cents() < 0) throw new IllegalArgumentException("negative price for " + sku);
        }
    }
    public record Quote(Money subtotal, Money discount, Money tax, Money shipping, Money total) {}

    private final TaxRates taxRates;                       // a dependency: looked up by country
    private final Clock clock;                             // a dependency: "today"

    public PriceCalculator(TaxRates taxRates, Clock clock) {
        this.taxRates = Objects.requireNonNull(taxRates);
        this.clock = Objects.requireNonNull(clock);
    }

    public Quote quote(List<Line> lines, String country) {
        if (lines.isEmpty()) throw new IllegalArgumentException("empty order");
        Money subtotal = Money.ZERO;
        int items = 0;
        for (Line l : lines) {
            subtotal = subtotal.plus(l.unitPrice().times(l.quantity()));
            items += l.quantity();
        }
        BigDecimal pct = items >= 100 ? BigDecimal.TEN : items >= 10 ? new BigDecimal("5") : BigDecimal.ZERO;
        DayOfWeek day = LocalDate.now(clock).getDayOfWeek();
        if (day == DayOfWeek.SATURDAY || day == DayOfWeek.SUNDAY) pct = pct.add(new BigDecimal("2.5"));   // weekend promotion
        Money discount = subtotal.percent(pct);
        Money net = subtotal.minus(discount);
        Money tax = net.percent(taxRates.rateFor(country));
        Money shipping = net.compareTo(Money.of("50.00")) >= 0 ? Money.ZERO : Money.of("4.99");
        return new Quote(subtotal, discount, tax, shipping, net.plus(tax).plus(shipping));
    }
}
```

```java
// file: pricing/src/main/java/com/example/pricing/TaxRates.java
package com.example.pricing;

import java.math.BigDecimal;

/** Where tax rates come from: a database or a remote service in production. */
public interface TaxRates {
    /** @throws UnknownCountryException if there is no rate for the country */
    BigDecimal rateFor(String country);

    class UnknownCountryException extends RuntimeException {
        private static final long serialVersionUID = 1L;      // exceptions are Serializable; -Xlint:all insists
        public UnknownCountryException(String country) { super("no tax rate for " + country); }
    }
}
```

```java
// file: pricing/src/main/java/com/example/pricing/Main.java
package com.example.pricing;

import java.math.BigDecimal;
import java.time.Clock;
import java.util.List;
import java.util.Map;

public class Main {
    public static void main(String[] args) {
        Map<String, BigDecimal> rates = Map.of("DE", new BigDecimal("19"), "FR", new BigDecimal("20"));
        TaxRates taxRates = c -> {
            BigDecimal r = rates.get(c);
            if (r == null) throw new TaxRates.UnknownCountryException(c);
            return r;
        };
        var calc = new PriceCalculator(taxRates, Clock.systemDefaultZone());
        var quote = calc.quote(List.of(new PriceCalculator.Line("PEN", Money.of("1.20"), 12), new PriceCalculator.Line("PAD", Money.of("4.50"), 3)), "DE");
        System.out.println("total " + quote.total() + " (discount and total depend on the day of the week)");
    }
}
```

`PriceCalculator` depends on two things it does not control — tax rates and today's date. Both are passed in through the constructor (**dependency injection**), which is what makes it testable: a test supplies a fixed clock and a stub `TaxRates`.

```java
// file: pricing/src/test/java/com/example/pricing/MoneyTest.java
package com.example.pricing;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertThrows;

import java.math.BigDecimal;
import org.junit.Test;

public class MoneyTest {
    @Test
    public void parsesDecimalStringsExactly() {
        assertEquals(1234, Money.of("12.34").cents());
        assertEquals(500, Money.of("5").cents());
        assertEquals(-50, Money.of("-0.50").cents());
    }

    @Test
    public void rejectsFractionsOfACent() {
        assertThrows(ArithmeticException.class, () -> Money.of("0.005"));
    }

    @Test
    public void percentRoundsHalfUpToTheCent() {
        assertEquals(Money.of("0.13"), Money.of("2.50").percent(new BigDecimal("5")));      // 12.5 cents -> 13
        assertEquals(Money.of("0.12"), Money.of("2.49").percent(new BigDecimal("5")));      // 12.45 cents -> 12
    }

    @Test
    public void overflowIsAnErrorNotAWrongAnswer() {
        assertThrows(ArithmeticException.class, () -> new Money(Long.MAX_VALUE).plus(new Money(1)));
    }

    @Test
    public void formatsNegativeAmounts() {
        assertEquals("-0.50", new Money(-50).toString());
        assertEquals("12.05", new Money(1205).toString());
    }
}
```

## Unit 9.4 — Test doubles: stubs, fakes, and mocks

**One-line summary:** A **test double** stands in for a real dependency. A **stub** returns canned answers; a **fake** is a simple working implementation (an in-memory map instead of a database); a **mock** records calls so the test can verify interactions. Interfaces with one or two methods are easy to stub with a lambda or a tiny class; mocking libraries (Mockito) generate doubles at run time.

```java
// file: pricing/src/test/java/com/example/pricing/PriceCalculatorTest.java
package com.example.pricing;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertThrows;

import com.example.pricing.PriceCalculator.Line;
import com.example.pricing.PriceCalculator.Quote;
import java.math.BigDecimal;
import java.time.Clock;
import java.time.Instant;
import java.time.ZoneOffset;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters;

public class PriceCalculatorTest {
    static final Clock WEDNESDAY = Clock.fixed(Instant.parse("2026-11-18T10:00:00Z"), ZoneOffset.UTC);
    static final Clock SATURDAY = Clock.fixed(Instant.parse("2026-11-21T10:00:00Z"), ZoneOffset.UTC);

    /** A fake that also records calls, so tests can verify interactions. */
    static class RecordingTaxRates implements TaxRates {
        final List<String> calls = new ArrayList<>();
        @Override public BigDecimal rateFor(String country) {
            calls.add(country);
            return switch (country) {
                case "DE" -> new BigDecimal("19");
                case "CH" -> new BigDecimal("8.1");
                case "XX" -> BigDecimal.ZERO;
                default -> throw new UnknownCountryException(country);
            };
        }
    }

    private RecordingTaxRates taxRates;
    private PriceCalculator calc;

    @Before
    public void setUp() {                                  // runs before every test, on a fresh instance
        taxRates = new RecordingTaxRates();
        calc = new PriceCalculator(taxRates, WEDNESDAY);
    }

    @Test
    public void smallOrderPaysTaxAndShipping() {
        Quote q = calc.quote(List.of(new Line("PEN", Money.of("1.20"), 5)), "DE");
        assertEquals(Money.of("6.00"), q.subtotal());
        assertEquals(Money.ZERO, q.discount());
        assertEquals(Money.of("1.14"), q.tax());
        assertEquals(Money.of("4.99"), q.shipping());
        assertEquals(Money.of("12.13"), q.total());
    }

    @Test
    public void weekendPromotionAddsToVolumeDiscount() {
        PriceCalculator weekend = new PriceCalculator(taxRates, SATURDAY);
        Quote q = weekend.quote(List.of(new Line("PAD", Money.of("4.00"), 10)), "XX");
        assertEquals(Money.of("3.00"), q.discount());      // 5% volume + 2.5% weekend of 40.00
    }

    @Test
    public void freeShippingIsDecidedAfterDiscount() {
        // 52.00 gross, 5% off -> 49.40 net: below the 50.00 threshold, so shipping applies
        Quote q = calc.quote(List.of(new Line("INK", Money.of("5.20"), 10)), "XX");
        assertEquals(Money.of("4.99"), q.shipping());
    }

    @Test
    public void asksForTheTaxRateExactlyOnce() {
        calc.quote(List.of(new Line("A", Money.of("1.00"), 1), new Line("B", Money.of("2.00"), 2)), "CH");
        assertEquals(List.of("CH"), taxRates.calls);       // interaction verified with a hand-written recorder
    }

    @Test
    public void unknownCountryPropagatesAndEmptyOrderIsRejected() {
        var e = assertThrows(TaxRates.UnknownCountryException.class, () -> calc.quote(List.of(new Line("A", Money.of("1.00"), 1)), "ZZ"));
        assertEquals("no tax rate for ZZ", e.getMessage());
        assertThrows(IllegalArgumentException.class, () -> calc.quote(List.of(), "DE"));
        assertThrows(IllegalArgumentException.class, () -> new Line("A", Money.of("1.00"), 0));
    }

    /** One test body, many rows of data: the volume-discount boundaries. */
    @RunWith(Parameterized.class)
    public static class VolumeDiscountTest {
        @Parameters(name = "{0} items -> discount {1}")
        public static Collection<Object[]> data() {
            return Arrays.asList(new Object[][] {{1, "0.00"}, {9, "0.00"}, {10, "0.50"}, {99, "4.95"}, {100, "10.00"}, {250, "25.00"}});
        }
        private final int items;
        private final String expected;
        public VolumeDiscountTest(int items, String expected) { this.items = items; this.expected = expected; }

        @Test
        public void discountAtBoundary() {
            PriceCalculator c = new PriceCalculator(country -> BigDecimal.ZERO, WEDNESDAY);     // a stub as a lambda
            assertEquals(Money.of(expected), c.quote(List.of(new Line("X", Money.of("1.00"), items)), "any").discount());
        }
    }
}
```

```console
$ cd pricing && mkdir -p libs && cp /opt/gradle-8.14.3/lib/junit-4.13.2.jar /opt/gradle-8.14.3/lib/hamcrest-core-1.3.jar libs/ && gradle --offline --no-daemon --console=plain clean build 2>&1 | grep -E '^> Task|BUILD|tests completed|FAILED' 
> Task :clean
> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes
> Task :jar
> Task :assemble
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> Task :check
> Task :build
BUILD SUCCESSFUL in 9s
$ cd pricing && grep -ho 'testsuite name="[^"]*" tests="[0-9]*" skipped="[0-9]*" failures="[0-9]*" errors="[0-9]*"' build/test-results/test/*.xml | sed 's/testsuite name="com.example.pricing.//; s/"//g' && ls build/libs && java -jar build/libs/pricing-1.0.0.jar | cut -c1-12
MoneyTest tests=5 skipped=0 failures=0 errors=0
PriceCalculatorTest$VolumeDiscountTest tests=6 skipped=0 failures=0 errors=0
PriceCalculatorTest tests=5 skipped=0 failures=0 errors=0
pricing-1.0.0.jar
total 35.70 
$ cd pricing && gradle --offline --no-daemon --console=plain build 2>&1 | grep -E '^> Task|BUILD'
> Task :compileJava UP-TO-DATE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE
> Task :jar UP-TO-DATE
> Task :assemble UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test UP-TO-DATE
> Task :check UP-TO-DATE
> Task :build UP-TO-DATE
BUILD SUCCESSFUL in 7s
```

(The very first attempt at this build failed, usefully: `-Xlint:all -Werror` rejected `UnknownCountryException` for lacking a `serialVersionUID` — every exception is `Serializable`. That is the `serial` lint mentioned in P-1; the one-line constant fixed it.) The first build ran every task; the second, with nothing changed, reports each task `UP-TO-DATE` and runs nothing — **incremental builds**. The test report lists three suites, including the six parameterised rows.

To see what a failure looks like, break the code — change the free-shipping threshold from "net" to "gross" — and run the tests again:

```console
$ cd pricing && sed -i 's/Money shipping = net.compareTo/Money shipping = subtotal.compareTo/' src/main/java/com/example/pricing/PriceCalculator.java && gradle --offline --no-daemon --console=plain test 2>&1 | grep -E 'FAILED|AssertionError|expected|tests completed|BUILD' ; sed -i 's/Money shipping = subtotal.compareTo/Money shipping = net.compareTo/' src/main/java/com/example/pricing/PriceCalculator.java
> Task :test FAILED
PriceCalculatorTest > freeShippingIsDecidedAfterDiscount FAILED
    java.lang.AssertionError at PriceCalculatorTest.java:71
16 tests completed, 1 failed
BUILD FAILED in 7s
```

One test of sixteen failed, and its name says which behaviour broke (`freeShippingIsDecidedAfterDiscount`), with the line of the failed assertion; the HTML report under `build/reports/tests` shows the expected and actual values. A test suite is a safety net exactly to the extent that its tests pin down decisions like this one.

**The same tests in JUnit 5 with Mockito and AssertJ** (*not run here*):

```java
@ExtendWith(MockitoExtension.class)
class PriceCalculatorTest {
    @Mock TaxRates taxRates;                                   // Mockito generates the double

    @Test
    void asksForTheTaxRateExactlyOnce() {
        when(taxRates.rateFor("CH")).thenReturn(new BigDecimal("8.1"));
        var calc = new PriceCalculator(taxRates, WEDNESDAY);

        Quote q = calc.quote(List.of(new Line("A", Money.of("1.00"), 1)), "CH");

        assertThat(q.tax()).isEqualTo(Money.of("0.08"));       // AssertJ: fluent assertions
        verify(taxRates, times(1)).rateFor("CH");              // interaction verification
        verifyNoMoreInteractions(taxRates);
    }

    @ParameterizedTest(name = "{0} items -> discount {1}")
    @CsvSource({"1, 0.00", "9, 0.00", "10, 0.50", "99, 4.95", "100, 10.00"})
    void volumeDiscount(int items, String expected) {
        var calc = new PriceCalculator(c -> BigDecimal.ZERO, WEDNESDAY);
        assertThat(calc.quote(List.of(new Line("X", Money.of("1.00"), items)), "any").discount()).isEqualTo(Money.of(expected));
    }
}
```

### Deep & Tricky

**DT-34. JUnit creates a new instance of the test class for every test method — so instance fields are never shared between tests, but `static` fields are.** *(Library behaviour: JUnit 4 and JUnit 5 default lifecycle)* This isolation is deliberate: a test that passes alone must pass in any order. State that leaks between tests comes from `static` fields, singletons, files, databases, system properties, and the system clock. Tests that depend on execution order or on each other's leftovers ("flaky tests") are among the most expensive problems in large codebases; inject clocks and other sources of nondeterminism, and clean shared resources in `@After`/`@AfterEach`.

### Pattern

**P-9. Make code testable by injecting what it depends on — collaborators, time, randomness, configuration — and prefer fakes over mocks.** *(Practice)* Constructor injection lets a test pass a fixed `Clock`, a seeded `Random`, an in-memory repository. A hand-written fake behaves like the real thing and survives refactoring; a mock that verifies exact call sequences couples the test to the implementation and breaks when the code changes without changing behaviour. Verify *outcomes* first, *interactions* only when the interaction is the requirement (e.g. "the audit log is written once"). *Trade-off:* fakes are code that must be kept faithful to the real dependency; for wide interfaces a mocking library is quicker.

### Mastery check

1. **Predict.** A test class has a field `private int counter = 0;`, and three `@Test` methods that each do `counter++` and assert `counter == 1`. Do they pass?
2. **Predict.** After a successful `gradle build`, you change only a comment in a test file and run `gradle build` again. Which tasks run?

## Chapter 9 Project — Test-driving a rate limiter

**Goal:** develop a token-bucket rate limiter test-first, with an injected clock so that every timing behaviour is tested deterministically and instantly — no `Thread.sleep` in any test. Build and test it with Gradle.

```groovy
// file: limiter/settings.gradle
rootProject.name = 'limiter'
```

```groovy
// file: limiter/build.gradle
plugins { id 'java' }
java { toolchain { languageVersion = JavaLanguageVersion.of(21) } }
repositories { flatDir { dirs 'libs' } }
dependencies { testImplementation files('libs/junit-4.13.2.jar', 'libs/hamcrest-core-1.3.jar') }
tasks.withType(JavaCompile).configureEach { options.compilerArgs += ['-Xlint:all', '-Werror'] }
test { useJUnit(); testLogging { events 'failed'; exceptionFormat 'short' } }
```

```java
// file: limiter/src/main/java/limiter/TokenBucket.java
package limiter;

import java.util.function.LongSupplier;

/**
 * A token bucket: holds up to `capacity` tokens, refilled continuously at `refillPerSecond`.
 * Each permitted request takes one token. Not thread-safe (Chapter 10 makes it so).
 */
public final class TokenBucket {
    private final long capacity;
    private final double refillPerNano;
    private final LongSupplier nanoClock;                     // injected: System::nanoTime in production
    private double tokens;
    private long lastRefill;

    public TokenBucket(long capacity, double refillPerSecond, LongSupplier nanoClock) {
        if (capacity <= 0) throw new IllegalArgumentException("capacity must be positive");
        if (refillPerSecond <= 0) throw new IllegalArgumentException("refill rate must be positive");
        this.capacity = capacity;
        this.refillPerNano = refillPerSecond / 1e9;
        this.nanoClock = nanoClock;
        this.tokens = capacity;                               // starts full
        this.lastRefill = nanoClock.getAsLong();
    }

    /** Takes one token if available. */
    public boolean tryAcquire() { return tryAcquire(1); }

    public boolean tryAcquire(int permits) {
        if (permits <= 0 || permits > capacity) throw new IllegalArgumentException("permits out of range: " + permits);
        refill();
        if (tokens >= permits) {
            tokens -= permits;
            return true;
        }
        return false;
    }

    /** Nanoseconds until `permits` tokens will be available (0 if they are now). */
    public long nanosUntilAvailable(int permits) {
        refill();
        return tokens >= permits ? 0 : (long) Math.ceil((permits - tokens) / refillPerNano);
    }

    private void refill() {
        long now = nanoClock.getAsLong();
        long elapsed = now - lastRefill;                      // subtraction is safe across nanoTime overflow
        if (elapsed > 0) {
            tokens = Math.min(capacity, tokens + elapsed * refillPerNano);
            lastRefill = now;
        }
    }
}
```

```java
// file: limiter/src/test/java/limiter/TokenBucketTest.java
package limiter;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertThrows;
import static org.junit.Assert.assertTrue;

import org.junit.Before;
import org.junit.Test;

public class TokenBucketTest {
    /** A clock the test moves by hand. */
    static final class FakeClock {
        long nanos;
        void advanceMillis(long ms) { nanos += ms * 1_000_000; }
    }

    private FakeClock clock;
    private TokenBucket bucket;                               // 5 tokens, 10 per second

    @Before
    public void setUp() {
        clock = new FakeClock();
        bucket = new TokenBucket(5, 10, () -> clock.nanos);
    }

    private int drain() {
        int n = 0;
        while (bucket.tryAcquire()) n++;
        return n;
    }

    @Test
    public void startsFullAndAllowsABurstUpToCapacity() {
        assertEquals(5, drain());
        assertFalse(bucket.tryAcquire());
    }

    @Test
    public void refillsContinuouslyAtTheConfiguredRate() {
        drain();
        clock.advanceMillis(99);
        assertFalse("0.99 tokens is not enough", bucket.tryAcquire());
        clock.advanceMillis(1);
        assertTrue("one token after 100 ms at 10/s", bucket.tryAcquire());
        assertFalse(bucket.tryAcquire());
        clock.advanceMillis(250);
        assertEquals(2, drain());                             // 2.5 tokens -> 2 whole ones
    }

    @Test
    public void neverHoldsMoreThanCapacity() {
        clock.advanceMillis(60_000);                          // a minute idle
        assertEquals(5, drain());
    }

    @Test
    public void partialTokensAreKept() {
        drain();
        clock.advanceMillis(50);                              // 0.5
        assertFalse(bucket.tryAcquire());
        clock.advanceMillis(50);                              // 1.0: the earlier half was not lost
        assertTrue(bucket.tryAcquire());
    }

    @Test
    public void multiPermitRequestsAreAllOrNothing() {
        assertTrue(bucket.tryAcquire(3));
        assertFalse(bucket.tryAcquire(3));                    // only 2 left: takes none
        assertEquals(2, drain());
    }

    @Test
    public void reportsTimeUntilAvailable() {
        drain();
        assertEquals(100_000_000L, bucket.nanosUntilAvailable(1));
        clock.advanceMillis(40);
        assertEquals(60_000_000L, bucket.nanosUntilAvailable(1));
        assertEquals(0, new TokenBucket(1, 1, () -> 0).nanosUntilAvailable(1));
    }

    @Test
    public void survivesClockWrapAround() {
        clock.nanos = Long.MAX_VALUE - 10;                    // System.nanoTime may be near overflow
        TokenBucket b = new TokenBucket(2, 1000, () -> clock.nanos);
        assertTrue(b.tryAcquire(2));
        clock.nanos += 1_000_020;                             // wraps to a large negative number
        assertTrue("1 ms passed across the wrap: one token", b.tryAcquire());
        assertFalse(b.tryAcquire());
    }

    @Test
    public void rejectsInvalidArguments() {
        assertThrows(IllegalArgumentException.class, () -> new TokenBucket(0, 1, () -> 0));
        assertThrows(IllegalArgumentException.class, () -> new TokenBucket(1, 0, () -> 0));
        assertThrows(IllegalArgumentException.class, () -> bucket.tryAcquire(6));
        assertThrows(IllegalArgumentException.class, () -> bucket.tryAcquire(0));
    }
}
```

```console
$ cd limiter && mkdir -p libs && cp /opt/gradle-8.14.3/lib/junit-4.13.2.jar /opt/gradle-8.14.3/lib/hamcrest-core-1.3.jar libs/ && gradle --offline --no-daemon --console=plain clean test 2>&1 | grep -E 'FAILED|expected|BUILD' ; grep -o 'testsuite name="[^"]*" tests="[0-9]*" skipped="[0-9]*" failures="[0-9]*" errors="[0-9]*"' build/test-results/test/*.xml | sed 's/.*testsuite //'; grep -o 'time="[0-9.]*"' build/test-results/test/*.xml | head -n 1
BUILD SUCCESSFUL in 8s
name="limiter.TokenBucketTest" tests="8" skipped="0" failures="0" errors="0"
time="0.027"
```

Eight tests cover a minute of simulated idle time, sub-token accumulation, and a clock wrap-around that would take 292 years to occur naturally — and the whole suite runs in milliseconds, because time is a value the test controls. The injected `LongSupplier` is the entire trick: production passes `System::nanoTime`; tests pass a field they advance by hand.

### Review checklist

- Which test would fail if `refill` rounded tokens down to whole numbers on every call?
- The class is documented as not thread-safe. What could two threads calling `tryAcquire` at once do to `tokens`?
- Write the test first for a new feature: `acquire()` that blocks until a token is available. How do you test blocking without sleeping? (Inject a "sleeper" too.)

## Answers to the Chapter 9 prediction questions

**9.4 Q1.** Yes, all three pass: JUnit creates a fresh instance of the test class for each test method, so each sees `counter == 0` (DT-34). With a `static` counter, two of the three would fail.

**9.4 Q2.** `compileTestJava` runs (its input changed), but its output — the class file — is identical, so `test` is `UP-TO-DATE` and the tests do not run again. Gradle compares task inputs and outputs, not timestamps.

# Chapter 10 — Concurrency: Threads, the Memory Model, `java.util.concurrent`, and Virtual Threads

*Goal: write correct concurrent Java — understand what can go wrong (races, visibility, deadlock), what the Java Memory Model guarantees, and how to use the tools that make correctness practical: `synchronized`, `volatile`, atomics, locks, concurrent collections, executors, `CompletableFuture`, and virtual threads.*

Unlike Rust, the Java compiler does not stop you from sharing mutable data between threads without synchronisation; unlike C and C++, a data race in Java is not undefined behaviour — the program stays memory-safe — but it can read stale or inconsistent values forever. Correctness is your job, and the Java Memory Model defines the rules.

**What you will be able to do after this chapter**

- Create threads and explain interruption.
- Recognise race conditions and visibility bugs; fix them with `synchronized`, `volatile`, or atomics.
- State the *happens-before* rules of the Java Memory Model.
- Use `ReentrantLock`, `ReadWriteLock`, conditions, `ConcurrentHashMap`, blocking queues, latches, and semaphores.
- Run tasks with `ExecutorService`, compose them with `CompletableFuture`, and scale I/O with virtual threads.
- Diagnose deadlocks with thread dumps.

## Topic 10A — Threads and what goes wrong

## Unit 10.1 — Threads and interruption

**One-line summary:** A `Thread` runs a `Runnable`; `start()` begins it, `join()` waits for it. Java threads are OS threads ("platform threads") unless created as virtual threads (Unit 10.7). There is no safe way to kill a thread; instead, `interrupt()` sets a flag that blocking methods (`sleep`, `wait`, `join`, queue operations) turn into an `InterruptedException` — a polite request to stop that the thread's code must honour.

```java
// file: Threads.java
public class Threads {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            int ticks = 0;
            try {
                while (true) {
                    Thread.sleep(20);
                    ticks++;
                }
            } catch (InterruptedException e) {                    // the interrupt arrives as an exception while blocked
                System.out.println("worker interrupted after " + (ticks >= 3 ? "some" : "few") + " ticks; cleaning up");
            }
        }, "worker-1");
        worker.start();
        Thread.sleep(100);
        worker.interrupt();
        worker.join();

        Thread spinner = Thread.ofPlatform().name("spinner").start(() -> {
            long n = 0;
            while (!Thread.currentThread().isInterrupted()) n++;   // CPU-bound code must poll the flag itself
            System.out.println("spinner saw the flag");
        });
        Thread.sleep(50);
        spinner.interrupt();
        spinner.join();
        System.out.println("main: " + Thread.currentThread().getName() + ", worker state: " + worker.getState()
                + ", cores: " + Runtime.getRuntime().availableProcessors());
    }
}
```

```console
$ java Threads.java
worker interrupted after some ticks; cleaning up
spinner saw the flag
main: main, worker state: TERMINATED, cores: 2
```

Never swallow `InterruptedException` with an empty `catch`: either let it propagate, or restore the flag with `Thread.currentThread().interrupt()` so code further up can see it.

<!-- expanded -->
### Going deeper: interruption is a request, and swallowing it is a bug

`thread.interrupt()` does not stop a thread. It sets a flag, and blocking methods (`sleep`, `wait`, `join`, `BlockingQueue.take`) respond by throwing `InterruptedException` *and clearing the flag*. Code that catches the exception and carries on has thrown the request away; a pool shutting down will then wait forever. If you cannot propagate the exception, restore the flag with `Thread.currentThread().interrupt()`.

```java
// file: Interrupts.java
public class Interrupts {
    static void politeSleep() {
        try { Thread.sleep(10_000); }
        catch (InterruptedException e) { Thread.currentThread().interrupt(); }   // restore the flag
    }
    static void rudeSleep() {
        try { Thread.sleep(10_000); }
        catch (InterruptedException e) { /* swallowed */ }
    }
    static Thread worker(String name, Runnable sleeper) {
        return Thread.ofPlatform().name(name).start(() -> {
            int rounds = 0;
            while (!Thread.currentThread().isInterrupted() && rounds < 3) { sleeper.run(); rounds++; }
            System.out.println(name + " stopped after " + rounds + " round(s), interrupted flag " + Thread.currentThread().isInterrupted());
        });
    }
    public static void main(String[] args) throws InterruptedException {
        Thread polite = worker("polite", Interrupts::politeSleep);
        Thread rude = worker("rude", Interrupts::rudeSleep);
        Thread.sleep(100);
        polite.interrupt();
        rude.interrupt();
        polite.join();
        rude.join(1_000);
        System.out.println("rude still running a second later: " + rude.isAlive());
        rude.interrupt(); rude.interrupt();
        rude.join();
    }
}
```

```console
$ java Interrupts.java
polite stopped after 1 round(s), interrupted flag true
rude still running a second later: true
rude stopped after 3 round(s), interrupted flag false
```

The polite worker stopped at once. The rude one lost the interruption and went to sleep again — in a real program that is a thread pool that never shuts down. (It is interrupted twice more at the end only so that the demonstration finishes.)

## Unit 10.2 — Race conditions and `synchronized`

**One-line summary:** `count++` is three steps (read, add, write); two threads interleaving them lose updates. A **`synchronized`** block or method takes the object's **monitor** (intrinsic lock): only one thread at a time can hold it, so the steps become atomic with respect to other code synchronised on the *same* object. `AtomicInteger`/`AtomicLong`/`LongAdder` provide lock-free atomic updates for single variables.

```java
// file: Race.java
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAdder;

public class Race {
    static long plain = 0;
    static long locked = 0;
    static final Object lock = new Object();
    static final AtomicLong atomic = new AtomicLong();
    static final LongAdder adder = new LongAdder();

    static long timed(Runnable body) throws InterruptedException {
        Thread[] ts = new Thread[4];
        long t = System.nanoTime();
        for (int i = 0; i < ts.length; i++) {
            ts[i] = new Thread(() -> { for (int k = 0; k < 2_000_000; k++) body.run(); });
            ts[i].start();
        }
        for (Thread th : ts) th.join();
        return (System.nanoTime() - t) / 1_000_000;
    }

    public static void main(String[] args) throws InterruptedException {
        long t1 = timed(() -> plain++);
        long t2 = timed(() -> { synchronized (lock) { locked++; } });
        long t3 = timed(atomic::incrementAndGet);
        long t4 = timed(adder::increment);
        System.out.printf("expected 8000000%nplain++       %8d  (%d ms)  <- lost updates%nsynchronized  %8d  (%d ms)%nAtomicLong    %8d  (%d ms)%nLongAdder     %8d  (%d ms)%n",
                plain, t1, locked, t2, atomic.get(), t3, adder.sum(), t4);
    }
}
```

```console
$ java Race.java
expected 8000000
plain++        7762846  (31 ms)  <- lost updates
synchronized   8000000  (172 ms)
AtomicLong     8000000  (113 ms)
LongAdder      8000000  (63 ms)
```

The unsynchronised counter lost about 40,000 of 8 million updates — and was the fastest, as wrong code often is. `synchronized` is correct but serialises the threads; `AtomicLong` uses a compare-and-swap instruction instead of a lock. `LongAdder` spreads the count over several cells to avoid contention; it wins when many cores update one counter, and showed no advantage on this two-core machine.

<!-- expanded -->
### Going deeper: atomics and `LongAdder`

For a single counter, a lock is more than you need. `AtomicLong.incrementAndGet` uses a CPU compare-and-swap instruction; under heavy contention many threads keep retrying the same memory location. `LongAdder` keeps several counters (one per contended thread, roughly) and adds them up when you read — much faster for "many writers, few readers" statistics.

```java
// file: Counters.java
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.atomic.LongAdder;

public class Counters {
    interface Counter { void inc(); long get(); }

    static long run(Counter c) throws InterruptedException {
        Thread[] ts = new Thread[4];
        long t0 = System.nanoTime();
        for (int i = 0; i < ts.length; i++) ts[i] = Thread.ofPlatform().start(() -> { for (int k = 0; k < 2_000_000; k++) c.inc(); });
        for (Thread t : ts) t.join();
        long ms = (System.nanoTime() - t0) / 1_000_000;
        System.out.println("  total " + c.get());
        return ms;
    }

    public static void main(String[] args) throws InterruptedException {
        long[] plain = {0};
        System.out.println("plain long (racy):");
        run(new Counter() { public void inc() { plain[0]++; } public long get() { return plain[0]; } });
        AtomicLong al = new AtomicLong();
        System.out.println("AtomicLong:");
        long atomicMs = run(new Counter() { public void inc() { al.incrementAndGet(); } public long get() { return al.get(); } });
        LongAdder adder = new LongAdder();
        System.out.println("LongAdder:");
        long adderMs = run(new Counter() { public void inc() { adder.increment(); } public long get() { return adder.sum(); } });
        System.out.println("LongAdder faster than AtomicLong here: " + (adderMs < atomicMs));
    }
}
```

```console
$ java Counters.java
plain long (racy):
  total 7912368
AtomicLong:
  total 8000000
LongAdder:
  total 8000000
LongAdder faster than AtomicLong here: true
```

The racy counter lost updates; both atomic versions counted all 8,000,000.

## Unit 10.3 — Visibility and the Java Memory Model

**One-line summary:** Without synchronisation, one thread's write to a field **may never become visible** to another: the JIT may keep the value in a register or hoist the read out of a loop. The **Java Memory Model** (JMM) defines when a write is guaranteed visible, through **happens-before** edges: unlocking a monitor → a later lock of it; writing a `volatile` → a later read of it; `Thread.start()` → the thread's actions; a thread's actions → another's `join()` on it; and `final` fields are visible once the constructor finishes. Anything not connected by happens-before is a **data race**.

```java
// file: Visibility.java
public class Visibility {
    static boolean stopPlain = false;
    static volatile boolean stopVolatile = false;

    static boolean stopsWithin(Runnable loop, Runnable setFlag, long millis) throws InterruptedException {
        Thread t = new Thread(loop);
        t.setDaemon(true);                                        // so a stuck thread does not keep the JVM alive
        t.start();
        Thread.sleep(200);                                        // let the JIT compile the hot loop
        setFlag.run();
        t.join(millis);
        return !t.isAlive();
    }

    public static void main(String[] args) throws InterruptedException {
        boolean a = stopsWithin(() -> { long n = 0; while (!stopPlain) n++; }, () -> stopPlain = true, 1000);
        boolean b = stopsWithin(() -> { long n = 0; while (!stopVolatile) n++; }, () -> stopVolatile = true, 1000);
        System.out.println("plain flag:    loop stopped within 1 s? " + a);
        System.out.println("volatile flag: loop stopped within 1 s? " + b);
    }
}
```

```console
$ java Visibility.java
plain flag:    loop stopped within 1 s? true
volatile flag: loop stopped within 1 s? true
```

The loop reading the plain `boolean` **never stops**: after compiling the loop, the JIT saw that nothing inside it writes `stopPlain`, read it once, and turned the loop into `while (true)`. That is a legal optimisation under the JMM, because no happens-before edge connects the main thread's write to the loop's reads. `volatile` adds the edge: every read sees the latest write.

### Deep & Tricky

**DT-35. `volatile` guarantees visibility and ordering, not atomicity: `volatileCounter++` is still a lost-update race.** *(Guarantee: JLS §17.4, Java Memory Model)* A `volatile` read or write is a single atomic action with a happens-before edge, but `x++` is a read *and* a write with a gap between them. Use `volatile` for flags and for publishing immutable snapshots written by one thread; use atomics, locks, or `synchronized` for read-modify-write. Likewise, `volatile long`/`double` are atomic, while non-volatile `long` and `double` writes may be split into two 32-bit halves on some JVMs (JLS §17.7).

**DT-36. Double-checked locking is correct only if the field is `volatile`.** *(Guarantee: JMM, since Java 5)* The lazy-singleton idiom — check the field, synchronise, check again, assign — is broken with a plain field: another thread can see a non-null reference to an object whose constructor's writes are not yet visible. Declaring the field `volatile` fixes it. Simpler and equally lazy: the **initialisation-on-demand holder** (`private static class Holder { static final Foo INSTANCE = new Foo(); }`), which relies on class initialisation being thread-safe (Unit 0.4), or an enum singleton.

## Topic 10B — The toolbox

## Unit 10.4 — Locks, conditions, and deadlock

**One-line summary:** `ReentrantLock` is an explicit lock with features `synchronized` lacks: `tryLock` (with timeout), interruptible locking, fairness, and multiple `Condition`s. `ReentrantReadWriteLock` allows many readers or one writer; `StampedLock` adds optimistic reads. Always unlock in `finally`. **Deadlock** occurs when threads wait for each other's locks in a cycle; prevent it by acquiring locks in a fixed global order, or by using `tryLock` with back-off.

```java
// file: Deadlock.java
import java.lang.management.ManagementFactory;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class Deadlock {
    static final class Account {
        final int id;
        long balance = 1000;
        final ReentrantLock lock = new ReentrantLock();
        Account(int id) { this.id = id; }
    }

    static void transferNaive(Account from, Account to, long amount) {
        synchronized (from) {
            Thread.yield();                                       // widen the window
            synchronized (to) { from.balance -= amount; to.balance += amount; }
        }
    }

    static void transferOrdered(Account from, Account to, long amount) {
        Account first = from.id < to.id ? from : to, second = first == from ? to : from;   // global order by id
        synchronized (first) {
            synchronized (second) { from.balance -= amount; to.balance += amount; }
        }
    }

    static boolean transferTryLock(Account from, Account to, long amount) throws InterruptedException {
        if (from.lock.tryLock(10, TimeUnit.MILLISECONDS)) {
            try {
                if (to.lock.tryLock(10, TimeUnit.MILLISECONDS)) {
                    try { from.balance -= amount; to.balance += amount; return true; } finally { to.lock.unlock(); }
                }
            } finally {
                from.lock.unlock();
            }
        }
        return false;                                             // caller may retry
    }

    interface Transfer { void run(Account a, Account b) throws InterruptedException; }

    static String race(String name, Transfer transfer) throws InterruptedException {
        Account a = new Account(1), b = new Account(2);
        Thread t1 = new Thread(() -> { try { for (int i = 0; i < 100_000; i++) transfer.run(a, b); } catch (InterruptedException e) { } }, name + "-A");
        Thread t2 = new Thread(() -> { try { for (int i = 0; i < 100_000; i++) transfer.run(b, a); } catch (InterruptedException e) { } }, name + "-B");
        t1.setDaemon(true); t2.setDaemon(true);
        t1.start(); t2.start();
        t1.join(3000); t2.join(3000);
        long[] dead = ManagementFactory.getThreadMXBean().findDeadlockedThreads();
        return dead != null ? "DEADLOCK detected among " + dead.length + " threads" : "finished, total balance " + (a.balance + b.balance);
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("ordered locks: " + race("ordered", (x, y) -> transferOrdered(x, y, 1)));
        System.out.println("tryLock:       " + race("trylock", (x, y) -> { while (!transferTryLock(x, y, 1)) Thread.onSpinWait(); }));
        System.out.println("naive nesting: " + race("naive", (x, y) -> transferNaive(x, y, 1)));
    }
}
```

```console
$ java Deadlock.java
ordered locks: finished, total balance 2000
tryLock:       finished, total balance 2000
naive nesting: DEADLOCK detected among 2 threads
```

In production, a thread dump (`jcmd <pid> Thread.print`, or `jstack`) prints every thread's stack and ends with "Found one Java-level deadlock", naming the threads and the locks each holds and waits for — the first tool to reach for when a Java process hangs.

<!-- expanded -->
### Going deeper: `tryLock` with a timeout breaks deadlocks

`ReentrantLock.tryLock(timeout)` gives up after a while instead of waiting forever. A thread that cannot get its second lock releases its first, backs off, and retries — the deadlock from this unit's example becomes, at worst, a short delay.

```java
// file: TryLock.java
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class TryLock {
    static final ReentrantLock a = new ReentrantLock(), b = new ReentrantLock();
    static int retries = 0;

    static void both(ReentrantLock first, ReentrantLock second) throws InterruptedException {
        while (true) {
            if (first.tryLock(10, TimeUnit.MILLISECONDS)) {
                try {
                    if (second.tryLock(10, TimeUnit.MILLISECONDS)) {
                        try { return; } finally { second.unlock(); }   // got both: do the work
                    }
                } finally { first.unlock(); }
            }
            synchronized (TryLock.class) { retries++; }
            Thread.sleep(ThreadLocalRandom.current().nextInt(5));     // back off a random amount
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = Thread.ofPlatform().start(() -> { try { for (int i = 0; i < 2000; i++) both(a, b); } catch (InterruptedException e) {} });
        Thread t2 = Thread.ofPlatform().start(() -> { try { for (int i = 0; i < 2000; i++) both(b, a); } catch (InterruptedException e) {} });
        t1.join(); t2.join();
        System.out.println("4000 critical sections completed in opposite lock orders, no deadlock"
                           + (retries > 0 ? " (some retries needed)" : ""));
    }
}
```

```console
$ java TryLock.java
4000 critical sections completed in opposite lock orders, no deadlock
```

The random back-off matters: two threads that retry in lock-step can keep colliding forever (*livelock*).

## Unit 10.5 — Concurrent collections and coordination

**One-line summary:** `ConcurrentHashMap` allows concurrent reads and writes with per-bin locking and **atomic compound operations** (`compute`, `merge`, `computeIfAbsent`, `putIfAbsent`). `CopyOnWriteArrayList` suits read-mostly lists. `BlockingQueue` (`ArrayBlockingQueue`, `LinkedBlockingQueue`) hands work between producers and consumers with back-pressure. `CountDownLatch`, `Semaphore`, `CyclicBarrier`, and `Phaser` coordinate threads.

```java
// file: ConcurrentTools.java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.Semaphore;
import java.util.concurrent.atomic.AtomicInteger;

public class ConcurrentTools {
    static void hammer(Map<Integer, Integer> map, boolean atomicMerge) throws InterruptedException {
        Thread[] ts = new Thread[4];
        for (int i = 0; i < 4; i++) {
            ts[i] = new Thread(() -> {
                for (int k = 0; k < 200_000; k++) {
                    int key = k % 100;
                    if (atomicMerge) map.merge(key, 1, Integer::sum);          // atomic on ConcurrentHashMap
                    else map.put(key, map.getOrDefault(key, 0) + 1);            // check-then-act: a race even on a concurrent map
                }
            });
            ts[i].start();
        }
        for (Thread t : ts) t.join();
    }

    public static void main(String[] args) throws InterruptedException {
        Map<Integer, Integer> chmMerge = new ConcurrentHashMap<>(), chmRacy = new ConcurrentHashMap<>();
        hammer(chmMerge, true);
        hammer(chmRacy, false);
        System.out.println("ConcurrentHashMap.merge:       total " + chmMerge.values().stream().mapToInt(i -> i).sum() + " of 800000");
        System.out.println("ConcurrentHashMap get-then-put: total " + chmRacy.values().stream().mapToInt(i -> i).sum() + " of 800000  <- compound action not atomic");
        try {
            Map<Integer, Integer> hm = new HashMap<>();
            hammer(hm, true);
            System.out.println("plain HashMap, 4 threads:      total " + hm.values().stream().mapToInt(i -> i).sum() + " of 800000  <- corrupted");
        } catch (RuntimeException e) {
            System.out.println("plain HashMap, 4 threads:      " + e.getClass().getSimpleName());
        }

        // producer/consumer with a bounded queue, a latch, and a semaphore
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);            // put() blocks when full
        CountDownLatch done = new CountDownLatch(3);
        Semaphore permits = new Semaphore(2);                                   // at most 2 consumers work at once
        AtomicInteger maxConcurrent = new AtomicInteger(), current = new AtomicInteger(), sum = new AtomicInteger();
        for (int c = 0; c < 3; c++) {
            new Thread(() -> {
                try {
                    for (int v; (v = queue.take()) >= 0; ) {                    // negative value: poison pill
                        permits.acquire();
                        try {
                            maxConcurrent.accumulateAndGet(current.incrementAndGet(), Math::max);
                            sum.addAndGet(v);
                            Thread.sleep(1);
                        } finally {
                            current.decrementAndGet();
                            permits.release();
                        }
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();
                }
            }).start();
        }
        for (int i = 1; i <= 100; i++) queue.put(i);
        for (int c = 0; c < 3; c++) queue.put(-1);
        done.await();
        System.out.println("3 consumers processed sum " + sum.get() + " (expected 5050); most working at once: " + maxConcurrent.get());
    }
}
```

```console
$ java ConcurrentTools.java
ConcurrentHashMap.merge:       total 800000 of 800000
ConcurrentHashMap get-then-put: total 788705 of 800000  <- compound action not atomic
plain HashMap, 4 threads:      total 796255 of 800000  <- corrupted
3 consumers processed sum 5050 (expected 5050); most working at once: 2
```

### Deep & Tricky

**DT-37. A thread-safe collection makes each *method* atomic, not sequences of methods: check-then-act on a `ConcurrentHashMap` is still a race.** *(Library behaviour: `java.util.concurrent`)* `if (!map.containsKey(k)) map.put(k, v)` and `map.put(k, map.get(k) + 1)` interleave between the calls. Use the atomic compound methods — `putIfAbsent`, `computeIfAbsent`, `compute`, `merge` — whose function runs while the bin is locked (so keep it short, and never touch the same map from inside it).

## Unit 10.6 — Executors and `CompletableFuture`

**One-line summary:** Do not create threads by hand; submit tasks to an **`ExecutorService`** — a thread pool with a work queue. `submit` returns a `Future`; `invokeAll` runs a batch; the executor must be shut down (`close()` since Java 19 waits for tasks). **`CompletableFuture`** composes asynchronous steps without blocking: `supplyAsync`, `thenApply`, `thenCompose`, `thenCombine`, `allOf`, `exceptionally`, `orTimeout`.

```java
// file: Executors1.java
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.stream.IntStream;

public class Executors1 {
    static int slowSquare(int x) {
        try { Thread.sleep(100); } catch (InterruptedException e) { throw new IllegalStateException(e); }
        if (x == 13) throw new IllegalArgumentException("unlucky " + x);
        return x * x;
    }

    public static void main(String[] args) throws Exception {
        long t = System.nanoTime();
        try (ExecutorService pool = Executors.newFixedThreadPool(8)) {            // close() waits for submitted tasks
            List<Callable<Integer>> tasks = IntStream.rangeClosed(1, 16).<Callable<Integer>>mapToObj(i -> () -> slowSquare(i)).toList();
            int sum = 0, failed = 0;
            for (Future<Integer> f : pool.invokeAll(tasks)) {
                try {
                    sum += f.get();
                } catch (ExecutionException e) {                                  // the task's exception, wrapped
                    failed++;
                    System.out.println("task failed: " + e.getCause());
                }
            }
            System.out.printf("16 tasks of 100 ms on 8 threads: sum %d, %d failed, %d ms%n", sum, failed, (System.nanoTime() - t) / 1_000_000 / 10 * 10);
        }

        t = System.nanoTime();
        CompletableFuture<Integer> price = CompletableFuture.supplyAsync(() -> slowSquare(7));
        CompletableFuture<Integer> stock = CompletableFuture.supplyAsync(() -> slowSquare(3));
        CompletableFuture<String> report = price.thenCombine(stock, (p, s) -> "price " + p + ", stock " + s)   // both, in parallel
            .thenApply(String::toUpperCase);
        CompletableFuture<String> recovered = CompletableFuture.supplyAsync(() -> slowSquare(13)).thenApply(v -> "value " + v)
            .exceptionally(ex -> "fallback (" + ex.getCause().getMessage() + ")");
        CompletableFuture<String> timedOut = CompletableFuture.supplyAsync(() -> { slowSquare(1); slowSquare(2); slowSquare(4); return "late"; })
            .orTimeout(150, TimeUnit.MILLISECONDS).exceptionally(ex -> "timed out: " + ex.getClass().getSimpleName());
        System.out.println(report.get() + " | " + recovered.get() + " | " + timedOut.get() + " | " + (System.nanoTime() - t) / 1_000_000 / 50 * 50 + " ms");
    }
}
```

```console
$ java Executors1.java
task failed: java.lang.IllegalArgumentException: unlucky 13
16 tasks of 100 ms on 8 threads: sum 1327, 1 failed, 200 ms
PRICE 49, STOCK 9 | fallback (unlucky 13) | timed out: TimeoutException | 150 ms
```

<!-- expanded -->
### Going deeper: composing `CompletableFuture`s

`thenApply` transforms a result, `thenCompose` chains another asynchronous step, `thenCombine` joins two independent ones, `allOf` waits for many, `exceptionally`/`handle` recover from failures, and `orTimeout` (Java 9) fails a stage that takes too long.

```java
// file: Futures.java
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

public class Futures {
    static CompletableFuture<Integer> price(String item, int ms) {
        return CompletableFuture.supplyAsync(() -> {
            try { Thread.sleep(ms); } catch (InterruptedException e) { throw new RuntimeException(e); }
            if (item.equals("broken")) throw new IllegalStateException("no price for " + item);
            return item.length() * 100;
        });
    }

    public static void main(String[] args) {
        long t0 = System.nanoTime();
        CompletableFuture<Integer> total = price("pen", 200).thenCombine(price("notebook", 200), Integer::sum);
        System.out.println("combined: " + total.join() + " in " + ((System.nanoTime() - t0) / 100_000_000 * 100) + "ms (parallel)");

        String r = price("broken", 10).thenApply(p -> "price " + p)
                                       .exceptionally(e -> "recovered from " + e.getCause().getMessage())
                                       .join();
        System.out.println(r);

        List<CompletableFuture<Integer>> all = List.of(price("a", 50), price("bb", 60), price("ccc", 70));
        CompletableFuture.allOf(all.toArray(CompletableFuture[]::new)).join();
        System.out.println("allOf: " + all.stream().map(CompletableFuture::join).toList());

        try {
            price("slow", 2_000).orTimeout(100, TimeUnit.MILLISECONDS).join();
        } catch (Exception e) {
            System.out.println("orTimeout: " + e.getCause().getClass().getSimpleName());
        }
    }
}
```

```console
$ java Futures.java
combined: 1100 in 200ms (parallel)
recovered from no price for broken
allOf: [100, 200, 300]
orTimeout: TimeoutException
```

`supplyAsync` without an executor runs on the shared `ForkJoinPool.commonPool()`; blocking calls (like this `sleep`, or JDBC) should be given their own executor — or, since Java 21, a virtual-thread executor (Unit 10.7).

## Unit 10.7 — Virtual threads and structured concurrency

**One-line summary:** A **virtual thread** (Java 21, JEP 444) is a thread scheduled by the JVM onto a few carrier OS threads. When it blocks on I/O, a lock, or `sleep`, it is **unmounted** and its carrier runs another virtual thread; its stack lives on the heap and costs kilobytes, not megabytes. So blocking code scales: one virtual thread per request or per task, with `Executors.newVirtualThreadPerTaskExecutor()`. Do not pool them, and do not use them to speed up CPU-bound work.

```java
// file: VirtualThreads.java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.IntStream;

public class VirtualThreads {
    static void blockingCall() {
        try { Thread.sleep(200); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }   // stands for a network call
    }

    static long run(ExecutorService pool, int tasks) {
        long t = System.nanoTime();
        AtomicInteger done = new AtomicInteger();
        try (pool) {
            IntStream.range(0, tasks).forEach(i -> pool.submit(() -> { blockingCall(); done.incrementAndGet(); }));
        }                                                             // close() waits for all tasks
        if (done.get() != tasks) throw new AssertionError();
        return (System.nanoTime() - t) / 1_000_000;
    }

    public static void main(String[] args) throws Exception {
        int tasks = 10_000;
        System.out.printf("%,d blocking tasks of 200 ms:%n", tasks);
        System.out.printf("  200 platform threads (pool):  %,6d ms%n", run(Executors.newFixedThreadPool(200), tasks));
        System.out.printf("  one virtual thread per task:  %,6d ms%n", run(Executors.newVirtualThreadPerTaskExecutor(), tasks));
        Thread v = Thread.ofVirtual().name("my-virtual").start(() -> System.out.println("  running in " + Thread.currentThread()));
        v.join();
        long t = System.nanoTime();
        Thread[] many = new Thread[100_000];
        for (int i = 0; i < many.length; i++) many[i] = Thread.startVirtualThread(VirtualThreads::blockingCall);
        for (Thread th : many) th.join();
        System.out.printf("  100,000 virtual threads each sleeping 200 ms: %d ms total%n", (System.nanoTime() - t) / 1_000_000);
    }
}
```

```console
$ java VirtualThreads.java
10,000 blocking tasks of 200 ms:
  200 platform threads (pool):  10,052 ms
  one virtual thread per task:     340 ms
  running in VirtualThread[#10222,my-virtual]/runnable@ForkJoinPool-1-worker-2
  100,000 virtual threads each sleeping 200 ms: 611 ms total
```

Ten thousand blocking tasks took as long as fifty rounds on the 200-thread pool, and barely longer than a single task with virtual threads; a hundred thousand concurrent sleepers finished in well under a second on two cores. **Structured concurrency** (`StructuredTaskScope`, a preview API in Java 21 and later) builds on this: subtasks forked in a scope are joined, cancelled, and their errors propagated as a unit, so a failed or cancelled request never leaks threads.

### Deep & Tricky

**DT-38. On Java 21–23, a virtual thread that blocks while inside a `synchronized` block *pins* its carrier thread.** *(Implementation behaviour: JEP 444; removed by JEP 491 in Java 24)* While pinned, the carrier OS thread cannot run other virtual threads, so heavy blocking I/O inside `synchronized` can starve the scheduler (carriers default to the number of cores). On Java 21, guard blocking sections with `ReentrantLock` instead, and find pinning with `-Djdk.tracePinnedThreads=full`. Java 24 changed monitors so that virtual threads no longer pin in `synchronized`. Also avoid `ThreadLocal` caches sized for pooled threads: with millions of virtual threads, per-thread caches multiply.

### Pattern

**P-10. Share nothing if you can; share immutable data if you must share; guard mutable shared state with one clear lock — and prefer `java.util.concurrent` to `wait`/`notify`.** *(Practice; Java Concurrency in Practice)* Confine mutable state to one thread (or one task), pass immutable records between threads through queues or futures, and when state must be shared, encapsulate it in a class that documents which lock guards which fields. Reach for `ConcurrentHashMap`, atomics, `BlockingQueue`, executors, and `CompletableFuture` before hand-written locking; use virtual threads for blocking I/O at scale and platform-thread pools sized to the cores for CPU work. *Trade-off:* higher-level tools hide costs (queue growth, pool starvation); bound your queues and time out your waits.

### Mastery check

1. **Predict.** A field is `volatile int hits;` and 4 threads each run `hits++` a million times. Is the final value exactly 4,000,000?
2. **Predict.** `map.computeIfAbsent(key, k -> new ArrayList<>()).add(x)` on a `ConcurrentHashMap<String, List<Integer>>` from many threads: is the map structure safe? Are the lists?

## Chapter 10 Project — A concurrent web crawler over a simulated web

**Goal:** crawl a simulated web of 3,000 pages, where fetching a page blocks for a few milliseconds like a network call, without visiting any page twice, with a per-host limit on simultaneous requests, and with a deterministic result that can be checked exactly. Implement it three ways — single-threaded, a fixed pool of platform threads, and virtual threads — and compare.

```java
// file: crawler/Crawler.java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Phaser;
import java.util.concurrent.Semaphore;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.LongAdder;

public class Crawler {
    /** A deterministic fake web: page i links to a few pseudo-random pages; fetching blocks. */
    static final class Web {
        final int pages, hosts;
        final AtomicInteger inFlight = new AtomicInteger(), maxInFlight = new AtomicInteger();
        final Map<Integer, AtomicInteger> perHost = new ConcurrentHashMap<>();
        final AtomicInteger maxPerHost = new AtomicInteger();
        final LongAdder fetches = new LongAdder();

        Web(int pages, int hosts) { this.pages = pages; this.hosts = hosts; }

        int hostOf(int page) { return page % hosts; }

        List<Integer> fetch(int page) {
            fetches.increment();
            maxInFlight.accumulateAndGet(inFlight.incrementAndGet(), Math::max);
            AtomicInteger h = perHost.computeIfAbsent(hostOf(page), k -> new AtomicInteger());
            maxPerHost.accumulateAndGet(h.incrementAndGet(), Math::max);
            try {
                Thread.sleep(2 + page % 5);                              // 2-6 ms of "network"
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                h.decrementAndGet();
                inFlight.decrementAndGet();
            }
            List<Integer> links = new ArrayList<>();
            long x = page * 2654435761L + 12345;
            int n = 1 + (int) (x >>> 7) % 4;
            for (int i = 0; i < n; i++) {
                x = x * 6364136223846793005L + 1442695040888963407L;
                links.add((int) ((x >>> 33) % pages));
            }
            if (page + 1 < pages && page % 3 == 0) links.add(page + 1);  // keeps most of the web reachable
            return links;
        }
    }

    record Result(int visited, long checksum, long millis) {}

    static Result sequential(Web web, int start) {
        long t = System.nanoTime();
        Set<Integer> seen = new java.util.HashSet<>(Set.of(start));
        ArrayDeque<Integer> queue = new ArrayDeque<>(List.of(start));
        long checksum = 0;
        while (!queue.isEmpty()) {
            int page = queue.poll();
            checksum += page;
            for (int link : web.fetch(page)) if (seen.add(link)) queue.add(link);
        }
        return new Result(seen.size(), checksum, (System.nanoTime() - t) / 1_000_000);
    }

    /** Concurrent crawl: one task per page; `seen` decides atomically who fetches a page. */
    static Result concurrent(Web web, int start, ExecutorService executor, int perHostLimit) {
        long t = System.nanoTime();
        Set<Integer> seen = ConcurrentHashMap.newKeySet();
        Map<Integer, Semaphore> hostLimits = new ConcurrentHashMap<>();
        LongAdder checksum = new LongAdder();
        Phaser pending = new Phaser(1);                                  // counts unfinished tasks; main is party 1
        class Task implements Runnable {
            final int page;
            Task(int page) { this.page = page; pending.register(); }     // registered before submission
            @Override public void run() {
                try {
                    Semaphore limit = hostLimits.computeIfAbsent(web.hostOf(page), h -> new Semaphore(perHostLimit));
                    limit.acquireUninterruptibly();
                    List<Integer> links;
                    try {
                        links = web.fetch(page);
                    } finally {
                        limit.release();
                    }
                    checksum.add(page);
                    for (int link : links) {
                        if (seen.add(link)) executor.execute(new Task(link));   // add() is atomic: exactly one thread wins
                    }
                } finally {
                    pending.arriveAndDeregister();
                }
            }
        }
        try (executor) {
            seen.add(start);
            executor.execute(new Task(start));
            pending.arriveAndAwaitAdvance();                             // wait until every registered task has finished
        }
        return new Result(seen.size(), checksum.sum(), (System.nanoTime() - t) / 1_000_000);
    }

    public static void main(String[] args) {
        int pages = 3000, hosts = 30;
        Web w1 = new Web(pages, hosts);
        Result seq = sequential(w1, 0);
        System.out.printf("sequential:            %d pages, checksum %d, %5d ms, max in flight %d%n", seq.visited(), seq.checksum(), seq.millis(), w1.maxInFlight.get());
        Web w2 = new Web(pages, hosts);
        Result pool = concurrent(w2, 0, Executors.newFixedThreadPool(32), 4);
        System.out.printf("32 platform threads:   %d pages, checksum %d, %5d ms, max in flight %d, max per host %d, fetches %d%n",
                pool.visited(), pool.checksum(), pool.millis(), w2.maxInFlight.get(), w2.maxPerHost.get(), w2.fetches.sum());
        Web w3 = new Web(pages, hosts);
        Result virt = concurrent(w3, 0, Executors.newVirtualThreadPerTaskExecutor(), 4);
        System.out.printf("virtual threads:       %d pages, checksum %d, %5d ms, max in flight %d, max per host %d, fetches %d%n",
                virt.visited(), virt.checksum(), virt.millis(), w3.maxInFlight.get(), w3.maxPerHost.get(), w3.fetches.sum());
        System.out.println("all three agree: " + (seq.visited() == pool.visited() && pool.visited() == virt.visited()
                && seq.checksum() == pool.checksum() && pool.checksum() == virt.checksum())
                + "; no page fetched twice: " + (w2.fetches.sum() == pool.visited() && w3.fetches.sum() == virt.visited()));
    }
}
```

```console
$ cd crawler && java Crawler.java
sequential:            2286 pages, checksum 3430198,  9475 ms, max in flight 1
32 platform threads:   2286 pages, checksum 3430198,   328 ms, max in flight 32, max per host 4, fetches 2286
virtual threads:       2286 pages, checksum 3430198,   195 ms, max in flight 118, max per host 4, fetches 2286
all three agree: true; no page fetched twice: true
```

The three crawls visit exactly the same pages (same count, same checksum), and the concurrent ones fetched each page exactly once. The pieces that make that true:

- **`seen.add(link)` is the only decision point**: on a concurrent set it is atomic, so of all threads that discover a link, exactly one gets `true` and submits the task (DT-37's lesson applied correctly — no check-then-act).
- **Per-host politeness** is a `Semaphore` per host, created atomically with `computeIfAbsent`; the measured maximum per host never exceeds the limit of 4.
- **Completion** is the subtle part of any crawler: tasks spawn tasks, so "the queue is empty" does not mean "done". A `Phaser` counts registered-but-unfinished tasks; each task registers in its constructor (before it is submitted) and deregisters in `finally`, and the main thread waits for the count to reach zero.
- **Platform pool versus virtual threads:** with 32 platform threads, at most 32 fetches are in flight; with virtual threads, concurrency is limited only by the per-host semaphores (30 hosts × 4 = 120), so the same code finished in a little over half the time — with no thread-pool sizing at all. The semaphore, not the pool size, is now what expresses the real limit.

### Review checklist

- What would go wrong if `Task` registered with the `Phaser` at the start of `run()` instead of in its constructor?
- Replace `seen.add(link)` with `if (!seen.contains(link)) { seen.add(link); ... }`. Which of the final checks fails?
- On Java 21, `acquireUninterruptibly` blocks a virtual thread without pinning. What if the per-host limit were a `synchronized` block with a `sleep` inside (DT-38)?

## Answers to the Chapter 10 prediction questions

**10.7 Q1.** No — almost certainly less. `volatile` makes each read and each write visible, but `hits++` is a read followed by a write, and threads interleave between them (DT-35). Use `AtomicInteger` or `LongAdder`.

**10.7 Q2.** The map is safe: `computeIfAbsent` is atomic, so every thread gets the same list for a key. The lists are not: `ArrayList.add` from several threads is a data race. Use a thread-safe list, or do the whole update inside `compute`.

# Chapter 11 — JVM Internals: Memory, Garbage Collection, the JIT, and Profiling

*Goal: understand what the JVM does with your program at run time — how objects are laid out, how the heap is organised and collected, how the JIT optimises — and use the JDK's own tools (GC logs, `jcmd`, heap histograms, Java Flight Recorder) to find memory leaks and performance problems.*

**What you will be able to do after this chapter**

- Describe the JVM's memory areas and estimate the size of objects.
- Explain generational garbage collection, and read a G1 GC log.
- Choose a collector and heap size for a workload.
- Explain escape analysis, inlining, and why micro-benchmarks mislead.
- Find a memory leak with a heap histogram, and a CPU or allocation hot spot with Java Flight Recorder.

## Topic 11A — Memory

## Unit 11.1 — Memory areas and object layout

**One-line summary:** The JVM's memory is divided into the **heap** (all objects and arrays; garbage collected), per-thread **stacks** (frames with locals and operand stacks), **metaspace** (class metadata), the **code cache** (JIT-compiled code), and native memory used by the JVM itself and by direct buffers. Every object starts with a **header** — 12 bytes on a 64-bit JVM with compressed class pointers — followed by its fields, padded to a multiple of 8 bytes. References are 4 bytes with **compressed oops** (heaps under 32 GB).

```java
// file: ObjectSizes.java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ObjectSizes {
    static class Empty {}
    static class OneByte { byte b; }
    static class Mixed { byte a; long b; byte c; int d; }          // the JVM reorders fields to pack them
    record Point(int x, int y) {}

    static long used() {
        Runtime rt = Runtime.getRuntime();
        for (int i = 0; i < 3; i++) System.gc();
        return rt.totalMemory() - rt.freeMemory();
    }

    static double bytesPer(java.util.function.IntFunction<Object> make) {
        int n = 1_000_000;
        Object[] keep = new Object[n];
        long before = used();
        for (int i = 0; i < n; i++) keep[i] = make.apply(i);
        long after = used();
        if (keep[n - 1] == null) throw new AssertionError();
        return (after - before) / (double) n;
    }

    public static void main(String[] args) {
        System.out.printf("(measured from heap growth, so within a byte or so) Object %.0f, Empty %.0f, OneByte %.0f, Mixed %.0f, Point %.0f bytes%n",
                bytesPer(i -> new Object()), bytesPer(i -> new Empty()), bytesPer(i -> new OneByte()), bytesPer(i -> new Mixed()), bytesPer(i -> new Point(i, i)));
        System.out.printf("Integer %.0f, Long %.0f, int[0] %.0f, int[4] %.0f, String of 5 ASCII chars %.0f bytes%n",
                bytesPer(i -> Integer.valueOf(i + 1000)), bytesPer(i -> Long.valueOf(i + 1000L)), bytesPer(i -> new int[0]), bytesPer(i -> new int[4]),
                bytesPer(i -> String.valueOf(10000 + i % 80000)));
        int n = 1_000_000;
        long b0 = used();
        int[] prim = new int[n];
        long b1 = used();
        List<Integer> al = new ArrayList<>(n);
        for (int i = 0; i < n; i++) al.add(i + 1000);
        long b2 = used();
        List<Integer> ll = new LinkedList<>();
        for (int i = 0; i < n; i++) ll.add(i + 1000);
        long b3 = used();
        System.out.printf("one million ints: int[] %.1f MB, ArrayList<Integer> %.1f MB, LinkedList<Integer> %.1f MB%n",
                (b1 - b0) / 1e6, (b2 - b1) / 1e6, (b3 - b2) / 1e6);
        if (prim.length + al.size() + ll.size() == 0) throw new AssertionError();
    }
}
```

```console
$ java -Xmx1g -XX:+UseSerialGC ObjectSizes.java
(measured from heap growth, so within a byte or so) Object 15, Empty 16, OneByte 16, Mixed 32, Point 24 bytes
Integer 16, Long 24, int[0] 16, int[4] 32, String of 5 ASCII chars 48 bytes
one million ints: int[] 4.0 MB, ArrayList<Integer> 20.0 MB, LinkedList<Integer> 40.0 MB
```

A boxed `Integer` costs 16 bytes to hold 4 bytes of data, plus the 4-byte reference to it: an `ArrayList<Integer>` uses five times the memory of an `int[]`, and a `LinkedList<Integer>` ten times. The 5-character `String` costs 48 bytes: a 24-byte `String` object plus a 24-byte `byte[]` (since Java 9, strings containing only Latin-1 characters store one byte per character — "compact strings").

### Deep & Tricky

**DT-39. Java has no value types yet: every object has a header and identity, and a collection of objects is a collection of *pointers* to objects scattered across the heap.** *(Implementation behaviour: HotSpot object model; Project Valhalla)* That is why `int[]` is so much denser and faster to scan than `List<Integer>`, and why performance-sensitive Java code uses primitive arrays, parallel arrays instead of arrays of small objects, and primitive-specialised collections (Eclipse Collections, fastutil). Project Valhalla's *value classes* (in preview in recent JDKs) let classes give up identity so the JVM can flatten them into arrays and fields.

<!-- expanded -->
### Going deeper: escape analysis — objects that never reach the heap

The JIT's **escape analysis** checks whether an object created in a method can be seen outside it. If not, it can *scalar-replace* the object — keep its fields in registers and never allocate it. Short-lived helper objects (a `Point` returned and immediately unpacked, an iterator over an array) often cost nothing once the code is hot.

```java
// file: Escape.java
import java.lang.management.ManagementFactory;

public class Escape {
    record Point(double x, double y) { double len() { return Math.sqrt(x * x + y * y); } }

    static double sum(int n) {
        double s = 0;
        for (int i = 0; i < n; i++) s += new Point(i, i + 1).len();   // the Point never escapes
        return s;
    }

    static long allocatedBytes() {
        return ((com.sun.management.ThreadMXBean) ManagementFactory.getThreadMXBean()).getCurrentThreadAllocatedBytes();
    }

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) sum(100_000);                     // warm up: let the JIT compile sum
        long before = allocatedBytes();
        double r = sum(10_000_000);
        long after = allocatedBytes();
        System.out.printf("result %.3e, bytes allocated for 10 million Points: %s%n", r,
                          (after - before) < 1_000_000 ? "almost none (scalar-replaced)" : (after - before) / 1_000_000 + " MB");
    }
}
```

```console
$ javac Escape.java && java Escape && java -XX:-DoEscapeAnalysis Escape
result 7.071e+13, bytes allocated for 10 million Points: almost none (scalar-replaced)
result 7.071e+13, bytes allocated for 10 million Points: 320 MB
```

Note that the class was compiled with `javac` first: run in source-file mode (`java Escape.java`), the same program allocated the full 320 MB on the book's machine even with escape analysis on — a reminder that the optimisation depends on how and when the JIT compiles the code. With escape analysis switched off, the same loop allocates 10 million 32-byte objects (about 320 MB) for the garbage collector to clean up. You cannot rely on the optimisation — it depends on inlining and on the code being hot — but it is why "avoid creating small objects" is weaker advice in Java than it once was.

## Unit 11.2 — Garbage collection

**One-line summary:** An object is garbage when it is no longer **reachable** from the **GC roots** (thread stacks, static fields, JNI references). Collectors exploit the **generational hypothesis** — most objects die young: new objects are allocated in the **young generation** (eden + survivor spaces) by bumping a pointer; a **young collection** copies the few survivors and frees the rest at once; objects that survive several collections are **promoted** to the **old generation**, collected less often. **G1**, the default collector, divides the heap into regions and collects those with the most garbage first, aiming for a pause-time goal (`-XX:MaxGCPauseMillis`, 200 ms by default).

| Collector | Flag | Character |
| --- | --- | --- |
| Serial | `-XX:+UseSerialGC` | single-threaded, smallest footprint; small heaps and containers with 1 CPU |
| Parallel | `-XX:+UseParallelGC` | maximum throughput, longer pauses; batch jobs |
| G1 | `-XX:+UseG1GC` (default) | balanced; pause-time goal; most services |
| ZGC | `-XX:+UseZGC` (generational: `-XX:+ZGenerational` on 21, default from 23) | pauses under a millisecond at any heap size, some throughput cost |
| Shenandoah | `-XX:+UseShenandoahGC` | concurrent compaction, low pauses |

```java
// file: GcDemo.java
import java.util.ArrayList;
import java.util.List;

public class GcDemo {
    static byte[] last;                                     // storing here makes each array escape (see Unit 11.3)

    public static void main(String[] args) {
        List<byte[]> longLived = new ArrayList<>();
        long checksum = 0;
        for (int i = 0; i < 3_000_000; i++) {
            byte[] temp = new byte[256];                    // short-lived garbage: dies in eden
            temp[0] = (byte) i;
            last = temp;
            checksum += temp[0];
            if (i % 100 == 0) longLived.add(new byte[1024]);   // 1% survive and get promoted: ~30 MB in total
        }
        System.out.println("allocated about " + (3_000_000L * 272 / 1_000_000) + " MB of short-lived objects; kept " + longLived.size() + " KiB-sized arrays; checksum " + checksum);
    }
}
```

```console
$ javac -d out GcDemo.java && java -cp out -Xmx256m -Xlog:gc:stdout GcDemo | sed -E 's/^\[[0-9.]+s\]\[info\]\[gc\] //' | grep -vE 'Using G1' | awk 'NR<=4 || /Pause Full|Concurrent/ {print} END {print "(" NR " lines in total)"}'
GC(0) Pause Young (Normal) (G1 Evacuation Pause) 7M->1M(98M) 3.289ms
GC(1) Pause Young (Normal) (G1 Evacuation Pause) 25M->2M(98M) 4.116ms
GC(2) Pause Young (Normal) (G1 Evacuation Pause) 57M->4M(98M) 3.411ms
GC(3) Pause Young (Normal) (G1 Evacuation Pause) 57M->6M(98M) 3.032ms
(13 lines in total)
$ java -cp out -Xmx256m -Xlog:gc:stdout GcDemo | grep -c 'Pause Young'; for gc in UseSerialGC UseParallelGC UseG1GC UseZGC; do /bin/bash -c "TIMEFORMAT='%R s'; time java -cp out -Xmx256m -XX:+$gc GcDemo > /dev/null" 2>&1 | sed "s/^/$gc: /"; done
13
UseSerialGC: 0.200 s
UseParallelGC: 0.233 s
UseG1GC: 0.277 s
UseZGC: 0.448 s
```

Each `Pause Young` line reads *heap before → heap after (heap capacity) pause time*. The program allocated about 800 MB in a 256 MB heap with thirteen young collections of one to a few milliseconds each: the short-lived arrays died in eden, and only the 1% of long-lived arrays accumulated (the "after" figure creeps up by a few megabytes per collection). For this half-second program the simplest collector was the fastest and ZGC the slowest — collectors differ in start-up and concurrent overhead, and their advantages only appear on large heaps and long runs. (The first version of this demo did not store `temp` in the static field — and showed only two collections: escape analysis, Unit 11.3, had removed the allocations entirely.)

### Deep & Tricky

**DT-40. A "memory leak" in Java is an object that is still reachable but will never be used again.** *(Design fact)* The GC frees only unreachable objects. Leaks come from references that outlive their usefulness: entries never removed from a `static` map or cache, listeners never unregistered, `ThreadLocal` values in pooled threads, inner-class instances pinning their outer object (DT-11), and unclosed resources. The heap grows until `OutOfMemoryError: Java heap space`. The fix is to remove the reference — or hold it weakly (`WeakHashMap`, `WeakReference`), or bound the cache (an LRU, Chapter 6). The chapter project finds one.

## Topic 11B — Execution

## Unit 11.3 — What the JIT does

**One-line summary:** C2's most important optimisation is **inlining**: replacing a call with the callee's body, which then exposes further optimisations. Virtual calls are inlined speculatively when profiling shows one or two receiver types (**monomorphic** or **bimorphic** call sites), with a guard and **deoptimisation** back to the interpreter if the guess fails. **Escape analysis** proves that an object never leaves a method, so it can be **scalar-replaced**: its fields live in registers and no allocation happens at all.

```java
// file: Jit.java
public class Jit {
    record Vec(double x, double y) {
        Vec plus(Vec o) { return new Vec(x + o.x, y + o.y); }
        Vec scale(double k) { return new Vec(x * k, y * k); }
        double norm2() { return x * x + y * y; }
    }

    static double work(int n) {
        double acc = 0;
        for (int i = 0; i < n; i++) {
            Vec v = new Vec(i, i + 1).plus(new Vec(1, 1)).scale(0.5);     // three temporary objects per iteration
            acc += v.norm2();
        }
        return acc;
    }

    interface Shape { double area(); }
    record Sq(double s) implements Shape { public double area() { return s * s; } }
    record Circ(double r) implements Shape { public double area() { return 3.14159 * r * r; } }
    record Tri(double b, double h) implements Shape { public double area() { return 0.5 * b * h; } }

    static double total(Shape[] shapes) { double t = 0; for (Shape s : shapes) t += s.area(); return t; }

    static long allocatedBytes() {
        return ((com.sun.management.ThreadMXBean) java.lang.management.ManagementFactory.getThreadMXBean()).getCurrentThreadAllocatedBytes();
    }

    public static void main(String[] args) {
        for (int i = 0; i < 20; i++) work(100_000);                        // warm up: C2 compiles work()
        long a0 = allocatedBytes(), t0 = System.nanoTime();
        double r = work(50_000_000);
        long t1 = System.nanoTime(), a1 = allocatedBytes();
        System.out.printf("50M iterations creating 150M Vec objects in source: %d ms, %,d bytes actually allocated (result %.3e)%n", (t1 - t0) / 1_000_000, a1 - a0, r);

        Shape[] mono = new Shape[3000], mega = new Shape[3000];
        for (int i = 0; i < 3000; i++) {
            mono[i] = new Sq(i);
            mega[i] = i % 3 == 0 ? new Sq(i) : i % 3 == 1 ? new Circ(i) : new Tri(i, 2);
        }
        double sink = 0;
        long tm = 0, tg = 0;
        for (int round = 0; round < 6; round++) {
            long s = System.nanoTime();
            for (int k = 0; k < 20_000; k++) sink += total(mono);
            long m = System.nanoTime();
            for (int k = 0; k < 20_000; k++) sink += total(mega);
            tm = (m - s) / 1_000_000;
            tg = (System.nanoTime() - m) / 1_000_000;
        }
        System.out.printf("60M virtual calls: one receiver type %d ms, three receiver types %d ms (sink %.1e)%n", tm, tg, sink);
    }
}
```

```console
$ javac -d out Jit.java && java -cp out Jit
50M iterations creating 150M Vec objects in source: 84 ms, 952 bytes actually allocated (result 2.083e+22)
60M virtual calls: one receiver type 33 ms, three receiver types 119 ms (sink 2.6e+15)
$ java -cp out -XX:-DoEscapeAnalysis Jit | head -n 1
50M iterations creating 150M Vec objects in source: 1428 ms, 6,400,000,952 bytes actually allocated (result 2.083e+22)
```

With escape analysis, the loop that "creates" 150 million objects allocated essentially nothing: the JIT inlined `plus`, `scale`, and `norm2`, saw that no `Vec` escapes, and kept the fields in registers. With escape analysis switched off, the same code allocates gigabytes and runs several times slower. This is why idiomatic Java — small immutable objects, short methods — is not as costly as its source suggests, and why "avoid allocation" advice must be checked with measurements, not assumed.

### Deep & Tricky

**DT-41. Micro-benchmarks written by hand measure the JIT's cleverness, not your code — use JMH.** *(Practice; Implementation behaviour)* Hand-rolled timing loops fall into predictable traps: no warm-up (measuring the interpreter); **dead-code elimination** (the JIT deletes work whose result is unused); **constant folding** (inputs known at compile time); on-stack replacement compiling a different code shape than a real call; and profile pollution between benchmarks in one JVM (the type-profile experiment above depends on what ran before it). **JMH** (the OpenJDK Java Microbenchmark Harness) handles forking, warm-up, `Blackhole`s, and statistics. This book's timing loops warm up, consume results, and report ratios — enough to show effects of 2× or more, not to settle 5% questions. (JMH is distributed through Maven Central, so it could not be run in this book's build environment.)

## Unit 11.4 — Observing a running JVM

**One-line summary:** `jcmd <pid> <command>` talks to a live JVM: `GC.class_histogram` (instance counts and bytes per class), `Thread.print` (thread dump), `GC.heap_dump` (for analysis in Eclipse MAT or VisualVM), `VM.flags`, `VM.native_memory`, `JFR.start`. **Java Flight Recorder** (JFR) is a profiler built into the JVM, cheap enough for production: it records CPU samples, allocations, GC pauses, lock contention, I/O, and exceptions into a `.jfr` file, read with `jfr print`, `jfr summary`, `jfr view` or JDK Mission Control.

### Pattern

**P-11. Diagnose before tuning: GC log first, then a heap histogram or dump for memory, and a JFR recording for CPU, allocation, and latency.** *(Practice)* Most JVM "tuning" problems are application problems — a leak, an oversized cache, an allocation storm, a lock. Turn on `-Xlog:gc*` in production from day one (it is cheap), set `-Xmx` explicitly (in containers the default is a quarter of the memory limit), and add `-XX:+HeapDumpOnOutOfMemoryError`. Change collector flags only after the evidence points to the collector. *Trade-off:* a bigger heap means fewer collections but longer worst-case pauses and more memory cost; ZGC trades some throughput for very short pauses.

<!-- expanded -->
### Going deeper: a thread dump and a heap histogram from the command line

`jcmd` talks to a running JVM by process ID. Two commands answer most "what is it doing?" questions: `Thread.print` (a stack trace of every thread — run it twice a few seconds apart, and threads stuck in the same place are your problem) and `GC.class_histogram` (how many objects of each class are on the heap — a growing count is a leak).

```java
// file: Busy.java
import java.util.ArrayList;
import java.util.List;

public class Busy {
    static final List<byte[]> cache = new ArrayList<>();               // a "leak"
    static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread.ofPlatform().name("stuck-worker").start(() -> {
            synchronized (lock) { try { lock.wait(); } catch (InterruptedException e) {} }
        });
        for (int i = 0; i < 2000; i++) cache.add(new byte[1024]);
        System.out.println("ready");
        Thread.sleep(30_000);
    }
}
```

```console
$ javac Busy.java && (java Busy > busy.out &) ; sleep 2; pid=$(jcmd | awk '/ Busy$/{print $1}'); jcmd $pid Thread.print | grep -A2 '"stuck-worker"' | sed -E 's/#[0-9]+ .*//; s/0x[0-9a-f]+/ADDR/g' | head -3; jcmd $pid GC.class_histogram | awk 'NR>2 && $4=="[B" {print "byte[] instances: " ($2 >= 2000 ? "2000 or more" : $2)}'; kill $pid
"stuck-worker" 
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait0(java.base@21.0.12.1/Native Method)
byte[] instances: 2000 or more
```

The thread dump shows `stuck-worker` in `Object.wait`, and the histogram shows the 2000 `byte[]` arrays the cache is holding. In production, `jcmd PID JFR.start duration=60s filename=rec.jfr` records a Flight Recorder profile with almost no overhead, to open later in JDK Mission Control.

### Mastery check

1. **Predict.** Roughly how much heap does `new long[1_000_000]` use, and how much does a `List<Long>` with a million elements (values above 127) use?
2. **Predict.** A method allocates a `StringBuilder`, appends three strings, and returns `sb.toString()`. Can escape analysis remove the `StringBuilder` allocation? Can it remove the returned `String`?

## Chapter 11 Project — Finding a leak and a hot spot

**Goal:** a small "session service" has two production problems: its memory grows until it dies, and its report endpoint is slow. Diagnose both using only JDK tools — GC logs and a class histogram for the leak, Java Flight Recorder for the hot spot — then fix them and verify with the same tools.

```java
// file: diag/SessionService.java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public class SessionService {
    record Session(long id, String user, byte[] state, List<String> events) {}

    private final Map<Long, Session> sessions;
    private final boolean fixed;
    private long nextId = 1;

    SessionService(boolean fixed) {
        this.fixed = fixed;
        this.sessions = fixed
            ? new LinkedHashMap<>(1024, 0.75f, true) {                                  // bounded LRU (Chapter 6)
                @Override protected boolean removeEldestEntry(Map.Entry<Long, Session> e) { return size() > 5_000; }
            }
            : new HashMap<>();                                                           // BUG 1: grows forever
    }

    long login(String user) {
        long id = nextId++;
        sessions.put(id, new Session(id, user, new byte[2048], new ArrayList<>(List.of("login"))));
        return id;
    }

    void logout(long id) {
        if (fixed) sessions.remove(id);
        else sessions.get(id).events().add("logout");                                    // BUG 1: marks, never removes
    }

    /** A report of users by number of sessions. */
    String report() {
        return fixed ? reportFast() : reportSlow();
    }

    private String reportSlow() {
        String out = "";
        List<String> users = new ArrayList<>();
        for (Session s : sessions.values()) {
            if (!users.contains(s.user())) users.add(s.user());                          // BUG 2a: O(n) lookup in a list
        }
        for (String u : users) {
            int count = 0;
            for (Session s : sessions.values()) if (s.user().equals(u)) count++;         // BUG 2b: rescans all sessions per user
            out += u + "=" + count + ";";                                                // BUG 2c: quadratic string building
        }
        return out;
    }

    private String reportFast() {
        Map<String, Integer> counts = new java.util.TreeMap<>();
        for (Session s : sessions.values()) counts.merge(s.user(), 1, Integer::sum);
        StringBuilder sb = new StringBuilder();
        counts.forEach((u, c) -> sb.append(u).append('=').append(c).append(';'));
        return sb.toString();
    }

    public static void main(String[] args) throws Exception {
        boolean fixed = args.length > 0 && args[0].equals("fixed");
        int rounds = args.length > 1 ? Integer.parseInt(args[1]) : 60;
        SessionService svc = new SessionService(fixed);
        long reportNanos = 0;
        int reportLength = 0;
        for (int round = 1; round <= rounds; round++) {
            for (int i = 0; i < 2_000; i++) {
                long id = svc.login("user" + (i * 7919 + round) % 800);
                if (i % 10 != 0) svc.logout(id);                                         // 90% log out again
            }
            long t = System.nanoTime();
            reportLength = svc.report().length();
            reportNanos += System.nanoTime() - t;
            if (round % 10 == 0) {
                Runtime rt = Runtime.getRuntime();
                System.gc();
                System.out.printf("round %3d: %,7d sessions held, heap used after GC %4d MB, report() avg %.1f ms%n",
                        round, svc.sessions.size(), (rt.totalMemory() - rt.freeMemory()) >> 20, reportNanos / 1e6 / round);
            }
            if (round == rounds && args.length > 2) {                                    // pause so jcmd can inspect the live process
                System.out.println("ready " + ProcessHandle.current().pid());
                Thread.sleep(Long.parseLong(args[2]));
            }
        }
        System.out.println("report length " + reportLength);
    }
}
```

**Step 1 — the symptom.** Run with a small heap and watch:

```console
$ cd diag && javac -d out SessionService.java && java -cp out -Xmx400m SessionService leaky 30 2>&1 | grep -vE '^\s+at ' | tail -n 5
round  10:  20,000 sessions held, heap used after GC   44 MB, report() avg 268.8 ms
round  20:  40,000 sessions held, heap used after GC   88 MB, report() avg 537.3 ms
round  30:  60,000 sessions held, heap used after GC  131 MB, report() avg 729.9 ms
report length 8690
```

Sessions held and heap after GC climb by 20,000 sessions and about 44 MB every ten rounds, and the report slows in step — already around 0.7 s per call. With a smaller heap and more rounds, the process dies:

```console
$ cd diag && java -cp out -Xmx160m -Xlog:gc:file=gc.log SessionService leaky 400 2>&1 | grep -E 'OutOfMemoryError|round +(10|20|30):' | head -n 4; echo "full collections: $(grep -c 'Pause Full' gc.log)"; grep 'Pause Full' gc.log | tail -n 4 | head -n 3 | sed -E 's/^\[[0-9.]+s\]\[info\]\[gc\] //'
round  10:  20,000 sessions held, heap used after GC   44 MB, report() avg 207.5 ms
round  20:  40,000 sessions held, heap used after GC   87 MB, report() avg 408.6 ms
round  30:  60,000 sessions held, heap used after GC  131 MB, report() avg 611.5 ms
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
full collections: 13
GC(45) Pause Full (G1 Compaction Pause) 159M->158M(160M) 54.612ms
GC(47) Pause Full (G1 Compaction Pause) 159M->159M(160M) 40.307ms
GC(48) Pause Full (G1 Compaction Pause) 159M->159M(160M) 50.584ms
```

The GC log shows the signature of a leak: full collections that free almost nothing ("159M->159M" in a 160 MB heap), back to back, until allocation fails.

**Step 2 — what is filling the heap?** Ask the live process for a class histogram:

```console
$ cd diag && (java -cp out -Xmx400m SessionService leaky 20 8000 > live.log 2>&1 &) ; while ! grep -q ready live.log; do sleep 0.5; done; PID=$(grep -o 'ready [0-9]*' live.log | cut -d' ' -f2); jcmd $PID GC.class_histogram | sed -n '2,7p' | awk '{printf "%-5s %10s %12s  %s\n", $1, $2, $3, $4}'; sleep 4
num   #instances       #bytes  class
-------------------------------------------------------                          
1:         89485     83923112  [B
2:         42413      1357216  java.util.HashMap$Node
3:         40000      1280000  SessionService$Session
4:         49401      1185624  java.lang.String
```

The top of the histogram answers the question: 84 MB of `byte[]` (`[B`) — the 2 KiB session states, plus string contents — and exactly 40,000 `SessionService$Session` objects, which is every login so far (20 rounds × 2,000), although 90% of them logged out. Sessions are never removed. Reading `logout` confirms it: it appends an event instead of removing the session.

**Step 3 — where does `report()` spend its time?** Record with JFR and print the hottest stack frames:

```console
$ cd diag && java -cp out -Xmx400m -XX:StartFlightRecording=filename=leaky.jfr,settings=profile SessionService leaky 20 > /dev/null 2>&1; jfr print --events jdk.ExecutionSample leaky.jfr | grep -A1 'stackTrace = \[' | grep -vE 'stackTrace|^--' | sed -E 's/^ +//; s/line:.*//' | sort | uniq -c | sort -rn | head -n 4
    623 SessionService.reportSlow() 
    196 java.util.HashMap$HashIterator.nextNode() 
      2 java.util.HashMap.put(Object, Object) 
      1 jdk.internal.misc.Unsafe.allocateUninitializedArray0(Class, int) 
```

Nearly all CPU samples have their top frame in `reportSlow` itself (with `String.equals` inlined into it) or in the `HashMap` iterator it drives: the method rescans every session once per user — 800 users × 40,000 sessions per call. It also does a `List.contains` per session and concatenates strings in a loop; those are smaller costs here, but the same quadratic shape.

**Step 4 — fix and verify.** The fixed version removes sessions on logout, bounds the map as an LRU as a safety net, and computes the report in one pass with a map and a `StringBuilder`:

```console
$ cd diag && java -cp out -Xmx160m -Xlog:gc:file=gc2.log SessionService fixed 400 | grep -E 'round +(10|200|400):|report length'; echo "full GCs: $(grep -c 'Pause Full' gc2.log)"
round  10:   2,000 sessions held, heap used after GC    6 MB, report() avg 1.1 ms
round 200:   4,999 sessions held, heap used after GC   12 MB, report() avg 0.9 ms
round 400:   4,999 sessions held, heap used after GC   12 MB, report() avg 0.9 ms
report length 7890
full GCs: 40
```

Memory is flat at 12 MB over 400 rounds in the same 160 MB heap that the leaky version exhausted. The sessions held level off at the LRU bound — the 10% of users who never log out accumulate until the cap evicts the oldest. The 40 full collections are exactly the explicit `System.gc()` calls the program makes to measure itself, and the report takes about a millisecond instead of most of a second.

### Review checklist

- Which evidence distinguished "the heap is too small" from "there is a leak"? (Heap after full GC growing without bound.)
- The histogram showed *what* is retained. What tool shows *who* retains it? (A heap dump analysed for the path to GC roots: `jcmd <pid> GC.heap_dump file.hprof`, opened in Eclipse MAT or VisualVM.)
- The fixed service caps sessions at 5,000 with an LRU. What user-visible behaviour does that introduce, and how would you monitor it?

## Answers to the Chapter 11 prediction questions

**11.4 Q1.** The `long[]` uses 8 MB (plus a 16-byte header). The `List<Long>` uses about 20 MB: 4 MB for the array of compressed references plus a million `Long` objects of 16 bytes each.

**11.4 Q2.** The `StringBuilder` does not escape, so in principle it can be scalar-replaced (in practice C2 has a dedicated string-concatenation optimisation, and since Java 9 `+` uses `invokedynamic` instead, DT-2). The returned `String` escapes to the caller, so it must be allocated — unless the method is inlined into a caller where the string itself does not escape.

# Chapter 12 — Networking, HTTP, and Databases with JDBC

*Goal: connect Java programs to the world — TCP sockets, the JDK's HTTP client and HTTP server, and relational databases through JDBC: prepared statements, transactions, batching, and the concurrency problems (lost updates) that transactions and optimistic locking solve.*

> **A note on what was run.** Everything networked in this chapter uses only the JDK and the loopback interface. For JDBC, the build machine could not download a current driver, so the examples run against **HSQLDB 2.2.8**, an older release of a pure-Java SQL database whose jar was available locally. The JDBC API is identical for PostgreSQL, MySQL, H2, or any other database; only the URL and the driver jar change.

**What you will be able to do after this chapter**

- Write TCP clients and servers, one virtual thread per connection.
- Call HTTP services with `java.net.http.HttpClient`, synchronously and asynchronously, with timeouts.
- Serve HTTP with the JDK's built-in `HttpServer`.
- Use JDBC correctly: `PreparedStatement`, try-with-resources, transactions, batches, generated keys.
- Explain SQL injection and prevent it.
- Detect and prevent lost updates with optimistic locking.

## Topic 12A — Networking

## Unit 12.1 — TCP sockets

**One-line summary:** `ServerSocket.accept()` returns a `Socket` per client; each socket has an `InputStream` and an `OutputStream`. With virtual threads (Unit 10.7), the simplest design — one blocking thread per connection — scales to very many connections. Always set timeouts (`setSoTimeout`, connect timeout), and frame messages yourself: TCP is a byte stream with no message boundaries.

```java
// file: Echo.java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketTimeoutException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.atomic.AtomicInteger;

public class Echo {
    public static void main(String[] args) throws Exception {
        AtomicInteger served = new AtomicInteger();
        try (ServerSocket server = new ServerSocket(0, 50, InetAddress.getLoopbackAddress())) {   // port 0: the OS picks a free port
            int port = server.getLocalPort();
            Thread acceptor = Thread.ofVirtual().start(() -> {
                while (!server.isClosed()) {
                    try {
                        Socket client = server.accept();
                        Thread.ofVirtual().start(() -> handle(client, served));    // one virtual thread per connection
                    } catch (IOException e) {
                        break;                                                       // server closed
                    }
                }
            });

            try (Socket s = new Socket()) {
                s.connect(new InetSocketAddress(InetAddress.getLoopbackAddress(), port), 1000);   // connect timeout
                s.setSoTimeout(500);                                                              // read timeout
                var out = new PrintWriter(s.getOutputStream(), true, StandardCharsets.UTF_8);
                var in = new BufferedReader(new InputStreamReader(s.getInputStream(), StandardCharsets.UTF_8));
                for (String msg : new String[] {"hello", "línea two", "quit"}) {
                    out.println(msg);
                    System.out.println("sent " + msg + " -> " + in.readLine());
                }
                System.out.println("after quit, readLine() = " + in.readLine() + " (end of stream)");
            }
            Thread[] clients = new Thread[500];                                    // 500 concurrent clients
            AtomicInteger ok = new AtomicInteger();
            for (int i = 0; i < clients.length; i++) {
                int id = i;
                clients[i] = Thread.ofVirtual().start(() -> {
                    try (Socket s = new Socket(InetAddress.getLoopbackAddress(), port)) {
                        var out = new PrintWriter(s.getOutputStream(), true, StandardCharsets.UTF_8);
                        var in = new BufferedReader(new InputStreamReader(s.getInputStream(), StandardCharsets.UTF_8));
                        out.println("client " + id);
                        if (("ECHO client " + id).equals(in.readLine())) ok.incrementAndGet();
                    } catch (IOException e) { /* counted as a failure */ }
                });
            }
            for (Thread t : clients) t.join();
            System.out.println("500 concurrent clients: " + ok.get() + " correct echoes; connections served: " + served.get());

            try (Socket idle = new Socket(InetAddress.getLoopbackAddress(), port)) {
                idle.setSoTimeout(200);
                idle.getInputStream().read();                                       // the server sends nothing unprompted
            } catch (SocketTimeoutException e) {
                System.out.println("read with no data: SocketTimeoutException after 200 ms (without a timeout it would block forever)");
            }
        }
    }

    static void handle(Socket client, AtomicInteger served) {
        served.incrementAndGet();
        try (client;
             var in = new BufferedReader(new InputStreamReader(client.getInputStream(), StandardCharsets.UTF_8));
             var out = new PrintWriter(client.getOutputStream(), true, StandardCharsets.UTF_8)) {
            for (String line; (line = in.readLine()) != null && !line.equals("quit"); ) {
                out.println("ECHO " + line);
            }
            out.println("bye");
        } catch (IOException e) {
            // client went away
        }
    }
}
```

```console
$ java Echo.java
sent hello -> ECHO hello
sent línea two -> ECHO línea two
sent quit -> bye
after quit, readLine() = null (end of stream)
500 concurrent clients: 500 correct echoes; connections served: 501
read with no data: SocketTimeoutException after 200 ms (without a timeout it would block forever)
```

<!-- expanded -->
### Going deeper: timeouts on sockets

A `Socket` created without timeouts can block forever: `connect` to an address that drops packets, or `read` from a peer that went silent. Production code sets both a connect timeout and a read timeout (`setSoTimeout`), and treats `SocketTimeoutException` as an ordinary, expected failure.

```java
// file: Timeouts.java
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketTimeoutException;

public class Timeouts {
    public static void main(String[] args) throws Exception {
        try (ServerSocket silent = new ServerSocket(0)) {             // accepts, never sends anything
            Thread.ofVirtual().start(() -> { try { silent.accept(); Thread.sleep(5_000); } catch (Exception e) {} });
            try (Socket s = new Socket()) {
                s.connect(new InetSocketAddress("127.0.0.1", silent.getLocalPort()), 1_000);
                s.setSoTimeout(300);                                   // read timeout
                InputStream in = s.getInputStream();
                long t0 = System.nanoTime();
                try {
                    in.read();
                } catch (SocketTimeoutException e) {
                    System.out.println("read timed out after about " + Math.round((System.nanoTime() - t0) / 1e8) * 100 + " ms: " + e.getMessage());
                }
            }
        }
        try (Socket s = new Socket()) {
            s.connect(new InetSocketAddress("127.0.0.1", 1), 1_000);   // nothing listens on port 1
        } catch (java.net.ConnectException e) {
            System.out.println("connect refused: " + e.getMessage());
        }
    }
}
```

```console
$ java Timeouts.java
read timed out after about 300 ms: Read timed out
connect refused: Connection refused
```

`java.net.http.HttpClient` has the same two settings: `connectTimeout` on the client builder and `timeout` on each request.

## Unit 12.2 — HTTP: client and server

**One-line summary:** `java.net.http.HttpClient` (Java 11) speaks HTTP/1.1 and HTTP/2, synchronously (`send`) or asynchronously (`sendAsync` → `CompletableFuture`), with connect and request timeouts, redirects, and pluggable body handlers. The JDK also contains a small HTTP server, `com.sun.net.httpserver.HttpServer` — enough for tools, tests, and small services; production services usually use a framework (Chapter 13).

```java
// file: Http.java
import com.sun.net.httpserver.HttpServer;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.http.HttpTimeoutException;
import java.nio.charset.StandardCharsets;
import java.time.Duration;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executors;

public class Http {
    public static void main(String[] args) throws Exception {
        HttpServer server = HttpServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), 0), 0);
        server.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        server.createContext("/hello", ex -> {
            String name = ex.getRequestURI().getQuery() == null ? "world" : ex.getRequestURI().getQuery().replace("name=", "");
            byte[] body = ("{\"greeting\":\"hello, " + name + "\"}").getBytes(StandardCharsets.UTF_8);
            ex.getResponseHeaders().add("Content-Type", "application/json");
            ex.sendResponseHeaders(200, body.length);
            try (var os = ex.getResponseBody()) { os.write(body); }
        });
        server.createContext("/echo", ex -> {
            byte[] in = ex.getRequestBody().readAllBytes();
            String reply = ex.getRequestMethod() + " " + in.length + " bytes: " + new String(in, StandardCharsets.UTF_8).toUpperCase();
            byte[] body = reply.getBytes(StandardCharsets.UTF_8);
            ex.sendResponseHeaders(ex.getRequestMethod().equals("POST") ? 201 : 405, body.length);
            try (var os = ex.getResponseBody()) { os.write(body); }
        });
        server.createContext("/slow", ex -> {
            try { Thread.sleep(300); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            ex.sendResponseHeaders(204, -1);
            ex.close();
        });
        server.start();
        String base = "http://127.0.0.1:" + server.getAddress().getPort();

        HttpClient client = HttpClient.newBuilder().connectTimeout(Duration.ofSeconds(2)).build();
        HttpResponse<String> r = client.send(HttpRequest.newBuilder(URI.create(base + "/hello?name=Java")).GET().build(), HttpResponse.BodyHandlers.ofString());
        System.out.println(r.statusCode() + " " + r.headers().firstValue("content-type").orElse("?") + " " + r.body() + " via " + r.version());

        HttpRequest post = HttpRequest.newBuilder(URI.create(base + "/echo")).header("Content-Type", "text/plain")
            .POST(HttpRequest.BodyPublishers.ofString("some payload")).build();
        System.out.println(client.send(post, HttpResponse.BodyHandlers.ofString()).statusCode() + " " + client.send(post, HttpResponse.BodyHandlers.ofString()).body());
        System.out.println("missing path: " + client.send(HttpRequest.newBuilder(URI.create(base + "/nope")).build(), HttpResponse.BodyHandlers.ofString()).statusCode());

        long t = System.nanoTime();
        List<CompletableFuture<Integer>> calls = java.util.stream.IntStream.range(0, 20)
            .mapToObj(i -> client.sendAsync(HttpRequest.newBuilder(URI.create(base + "/slow")).build(), HttpResponse.BodyHandlers.discarding())
                .thenApply(HttpResponse::statusCode)).toList();
        CompletableFuture.allOf(calls.toArray(CompletableFuture[]::new)).join();
        System.out.println("20 async calls to a 300 ms endpoint: all " + calls.get(0).join() + ", total " + (System.nanoTime() - t) / 1_000_000 / 100 * 100 + " ms");

        try {
            client.send(HttpRequest.newBuilder(URI.create(base + "/slow")).timeout(Duration.ofMillis(100)).build(), HttpResponse.BodyHandlers.discarding());
        } catch (HttpTimeoutException e) {
            System.out.println("request timeout: " + e.getMessage());
        }
        server.stop(0);
    }
}
```

```console
$ java Http.java
200 application/json {"greeting":"hello, Java"} via HTTP_1_1
201 POST 12 bytes: SOME PAYLOAD
missing path: 404
20 async calls to a 300 ms endpoint: all 204, total 300 ms
request timeout: request timed out
```

### Deep & Tricky

**DT-42. A network call without a timeout can block forever — and by default, Java's sockets and `HttpClient` requests have none.** *(Library behaviour: `Socket`, `HttpClient`)* A peer that accepts the connection and then sends nothing holds your thread (or virtual thread, and whatever resources it owns) indefinitely. Set a connect timeout *and* a read/request timeout on every outbound call, bound retries, and treat timeouts as an expected failure. `HttpClient.connectTimeout` covers only connection establishment; `HttpRequest.timeout` covers the wait for the response.

## Topic 12B — Databases

## Unit 12.3 — JDBC basics

**One-line summary:** JDBC is Java's database API: a `Connection` (from `DriverManager` or, in real applications, a pooled `DataSource`), `PreparedStatement` for SQL with `?` parameters, and `ResultSet` for rows. All three are `AutoCloseable` — use try-with-resources. Column and parameter indexes start at **1**. Never build SQL by concatenating user input: that is **SQL injection**.

```java
// file: jdbc/Basics.java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

public class Basics {
    record User(long id, String name, String email, boolean admin) {}

    static List<User> query(Connection c, String sql, Object... params) throws SQLException {
        try (PreparedStatement ps = c.prepareStatement(sql)) {
            for (int i = 0; i < params.length; i++) ps.setObject(i + 1, params[i]);       // parameters are 1-based
            try (ResultSet rs = ps.executeQuery()) {
                List<User> out = new ArrayList<>();
                while (rs.next()) out.add(new User(rs.getLong("id"), rs.getString("name"), rs.getString("email"), rs.getBoolean("admin")));
                return out;
            }
        }
    }

    public static void main(String[] args) throws SQLException {
        try (Connection c = DriverManager.getConnection("jdbc:hsqldb:mem:basics", "SA", "")) {
            try (Statement s = c.createStatement()) {
                s.execute("create table users (id bigint generated by default as identity primary key, name varchar(50) not null, "
                        + "email varchar(100) unique, admin boolean default false not null)");
            }
            try (PreparedStatement ins = c.prepareStatement("insert into users (name, email, admin) values (?, ?, ?)", Statement.RETURN_GENERATED_KEYS)) {
                for (Object[] u : new Object[][] {{"Ada", "ada@example.com", true}, {"Bob", "bob@example.com", false}, {"O'Brien", "ob@example.com", false}}) {
                    ins.setString(1, (String) u[0]);
                    ins.setString(2, (String) u[1]);
                    ins.setBoolean(3, (Boolean) u[2]);
                    ins.executeUpdate();
                    try (ResultSet keys = ins.getGeneratedKeys()) {
                        keys.next();
                        System.out.println("inserted " + u[0] + " with id " + keys.getLong(1));
                    }
                }
            }
            System.out.println(query(c, "select * from users where name = ?", "O'Brien"));       // the quote is just data

            String evil = "nobody' or '1'='1";                                                  // attacker-controlled input
            try (Statement s = c.createStatement();
                 ResultSet rs = s.executeQuery("select count(*) from users where name = '" + evil + "'")) {   // NEVER do this
                rs.next();
                System.out.println("concatenated SQL with malicious input matched " + rs.getInt(1) + " users  <- SQL injection");
            }
            System.out.println("prepared statement with the same input matched " + query(c, "select * from users where name = ?", evil).size() + " users");

            try (PreparedStatement dup = c.prepareStatement("insert into users (name, email) values (?, ?)")) {
                dup.setString(1, "Dup");
                dup.setString(2, "ada@example.com");                                            // email must be unique
                dup.executeUpdate();
            } catch (SQLException e) {
                System.out.println("constraint violation: SQLState " + e.getSQLState() + " (" + e.getClass().getSimpleName() + ")");
            }
        }
    }
}
```

```console
$ cd jdbc && java -cp ../../../libs/hsqldb-2.2.8.jar Basics.java
inserted Ada with id 0
inserted Bob with id 1
inserted O'Brien with id 2
[User[id=2, name=O'Brien, email=ob@example.com, admin=false]]
concatenated SQL with malicious input matched 3 users  <- SQL injection
prepared statement with the same input matched 0 users
constraint violation: SQLState 23505 (SQLIntegrityConstraintViolationException)
```

The concatenated query turned the attacker's text into SQL (`… where name = 'nobody' or '1'='1'`), matching every row; with a `PreparedStatement`, the same text is sent as a *value* and matches nothing.

<!-- expanded -->
### Deep & Tricky

**DT-43. Building SQL by concatenating strings (`"... WHERE name = '" + name + "'"`) allows *SQL injection*: an input such as `' OR '1'='1` changes the query's meaning. `PreparedStatement` sends the SQL and the values separately, so a value can never become SQL — and it is also faster for repeated queries.** *(Guarantee: JDBC `PreparedStatement` parameters are bound as values, not parsed as SQL)* Identifiers (table and column names) cannot be parameters; when those come from input, check them against an allow-list.

```java
// file: jdbc/Injection.java
import java.sql.*;

public class Injection {
    public static void main(String[] args) throws SQLException {
        try (Connection c = DriverManager.getConnection("jdbc:hsqldb:mem:inj", "SA", "")) {
            try (Statement s = c.createStatement()) {
                s.execute("CREATE TABLE users (name VARCHAR(20), secret VARCHAR(20))");
                s.execute("INSERT INTO users VALUES ('ada', 'a-secret'), ('bob', 'b-secret')");
            }
            String input = "nobody' OR '1'='1";
            try (Statement s = c.createStatement();
                 ResultSet rs = s.executeQuery("SELECT secret FROM users WHERE name = '" + input + "'")) {
                int n = 0;
                while (rs.next()) n++;
                System.out.println("concatenated query returned " + n + " secret(s)");
            }
            try (PreparedStatement ps = c.prepareStatement("SELECT secret FROM users WHERE name = ?")) {
                ps.setString(1, input);
                try (ResultSet rs = ps.executeQuery()) {
                    int n = 0;
                    while (rs.next()) n++;
                    System.out.println("prepared statement returned " + n + " secret(s)");
                }
            }
        }
    }
}
```

```console
$ cd jdbc && java -cp ../../../libs/hsqldb-2.2.8.jar Injection.java
concatenated query returned 2 secret(s)
prepared statement returned 0 secret(s)
```

## Unit 12.4 — Transactions and batching

**One-line summary:** By default a JDBC connection is in **auto-commit** mode: each statement is its own transaction. For a unit of work that must succeed or fail as a whole, call `setAutoCommit(false)`, do the work, then `commit()` — or `rollback()` on any failure. **Batching** (`addBatch`/`executeBatch`) sends many statements in one round trip. The four **isolation levels** (read uncommitted, read committed, repeatable read, serializable) trade consistency against concurrency.

```java
// file: jdbc/Transactions.java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class Transactions {
    static long balance(Connection c, int id) throws SQLException {
        try (PreparedStatement ps = c.prepareStatement("select balance from accounts where id = ?")) {
            ps.setInt(1, id);
            try (ResultSet rs = ps.executeQuery()) { rs.next(); return rs.getLong(1); }
        }
    }

    /** Moves money atomically: both updates commit, or neither does. */
    static void transfer(Connection c, int from, int to, long amount) throws SQLException {
        boolean old = c.getAutoCommit();
        c.setAutoCommit(false);
        try (PreparedStatement debit = c.prepareStatement("update accounts set balance = balance - ? where id = ?");
             PreparedStatement credit = c.prepareStatement("update accounts set balance = balance + ? where id = ?")) {
            debit.setLong(1, amount); debit.setInt(2, from);
            if (debit.executeUpdate() != 1) throw new SQLException("no such account " + from);
            credit.setLong(1, amount); credit.setInt(2, to);
            if (credit.executeUpdate() != 1) throw new SQLException("no such account " + to);
            c.commit();
        } catch (SQLException e) {
            c.rollback();                                             // undo the debit if the credit failed
            throw e;
        } finally {
            c.setAutoCommit(old);
        }
    }

    public static void main(String[] args) throws SQLException {
        try (Connection c = DriverManager.getConnection("jdbc:hsqldb:mem:tx", "SA", "")) {
            try (Statement s = c.createStatement()) {
                s.execute("create table accounts (id int primary key, balance bigint not null, check (balance >= 0))");
                s.execute("insert into accounts values (1, 1000)");
                s.execute("insert into accounts values (2, 500)");
            }
            transfer(c, 1, 2, 300);
            System.out.println("after transfer 300: " + balance(c, 1) + " / " + balance(c, 2));
            for (Object[] bad : new Object[][] {{1, 99, 100L, "to a missing account"}, {2, 1, 5000L, "more than the balance"}}) {
                try {
                    transfer(c, (Integer) bad[0], (Integer) bad[1], (Long) bad[2]);
                } catch (SQLException e) {
                    System.out.println("transfer " + bad[3] + " failed (" + e.getMessage().lines().findFirst().orElse("").replaceAll(" ?SYS_CT_\\d+", "") + "); balances: "
                            + balance(c, 1) + " / " + balance(c, 2));
                }
            }

            try (Statement s = c.createStatement()) { s.execute("create table events (id int primary key, payload varchar(40))"); }
            int n = 20_000;
            long t = System.nanoTime();
            try (PreparedStatement ps = c.prepareStatement("insert into events values (?, ?)")) {
                for (int i = 0; i < n; i++) { ps.setInt(1, i); ps.setString(2, "event-" + i); ps.executeUpdate(); }      // auto-commit: one transaction each
            }
            long single = (System.nanoTime() - t) / 1_000_000;
            c.setAutoCommit(false);
            t = System.nanoTime();
            try (PreparedStatement ps = c.prepareStatement("insert into events values (?, ?)")) {
                for (int i = n; i < 2 * n; i++) {
                    ps.setInt(1, i); ps.setString(2, "event-" + i); ps.addBatch();
                    if (i % 1000 == 999) ps.executeBatch();                // send 1,000 rows per round trip
                }
            }
            c.commit();
            long batched = (System.nanoTime() - t) / 1_000_000;
            try (Statement s = c.createStatement(); ResultSet rs = s.executeQuery("select count(*) from events")) {
                rs.next();
                System.out.printf("%d rows; %d single auto-commit inserts: %d ms; %d batched in one transaction: %d ms%n", rs.getInt(1), n, single, n, batched);
            }
        }
    }
}
```

```console
$ cd jdbc && java -cp ../../../libs/hsqldb-2.2.8.jar Transactions.java
after transfer 300: 700 / 800
transfer to a missing account failed (no such account 99); balances: 700 / 800
transfer more than the balance failed (integrity constraint violation: check constraint; table: ACCOUNTS); balances: 700 / 800
40000 rows; 20000 single auto-commit inserts: 85 ms; 20000 batched in one transaction: 22 ms
```

Both failed transfers left the balances exactly as they were: in the first, the debit had already been applied when the credit found no row, and `rollback()` undid it. (Against an in-memory database in the same process, batching in one transaction was about twice as fast; against a database across a network, where each statement is a round trip and each commit a disk sync, it is commonly ten to a hundred times faster.)

### Deep & Tricky

**DT-44. A connection returned to a pool keeps its state: an unfinished transaction, a changed isolation level, or `autoCommit=false` leaks to the next user.** *(Library behaviour: connection pools — HikariCP, etc.)* Opening a database connection is expensive (TCP, TLS, authentication), so applications borrow connections from a **pool**; `close()` returns the connection rather than closing it. Good pools reset state and roll back open transactions on return, but locks held by an uncommitted transaction block other users until then. Always finish with `commit` or `rollback` in `finally` (or use a framework's transaction management), keep transactions short, and never hold a transaction open across a network call or user think-time.

## Unit 12.5 — Lost updates and optimistic locking

**One-line summary:** Two transactions that both *read* a row, compute a new value, and *write* it back can overwrite each other — a **lost update** — even though each transaction is individually correct. Fixes: do the arithmetic in SQL (`set stock = stock - ?`), lock the row when reading (`select … for update` — **pessimistic** locking), or add a **version column** and update `where id = ? and version = ?`, retrying when no row matches — **optimistic** locking, the default in most web applications and ORMs.

The chapter project measures all three.

### Pattern

**P-12. Hide JDBC behind a repository interface; use `PreparedStatement` always; one transaction per unit of work; optimistic locking for concurrent edits.** *(Practice)* A `ProductRepository` interface with `findById`, `save`, and domain-specific queries keeps SQL in one place and lets tests use an in-memory fake (P-9). Libraries reduce the boilerplate at increasing levels of abstraction: Spring's `JdbcClient`, jOOQ (type-safe SQL), and JPA/Hibernate (object-relational mapping). *Trade-off:* ORMs save code for simple CRUD and hide the SQL that matters for performance (the "N+1 queries" problem); learn JDBC and SQL first so you can see what the ORM does.

### Mastery check

1. **Predict.** With auto-commit on, a method runs two `update` statements and the second throws. Is the first update undone?
2. **Predict.** `ps.setString(1, "x'; drop table users; --")` on `select * from users where name = ?` — what does the database execute?

## Chapter 12 Project — An inventory service that does not oversell

**Goal:** a product inventory in a SQL database, with a repository class; a "purchase" operation run by 16 concurrent threads against the same product, implemented four ways — naive read-then-write, atomic SQL update, pessimistic locking, and optimistic locking with retry — and a check of which ones oversell.

```java
// file: inventory/Inventory.java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class Inventory {
    static final String URL = "jdbc:hsqldb:mem:shop;hsqldb.tx=locks";           // two-phase locking (this database's default)

    record Product(int id, String name, int stock, int version) {}

    /** All SQL for products lives here. */
    static final class ProductRepository {
        private final Connection c;
        ProductRepository(Connection c) { this.c = c; }

        Product find(int id, boolean forUpdate) throws SQLException {
            try (PreparedStatement ps = c.prepareStatement("select id, name, stock, version from products where id = ?" + (forUpdate ? " for update" : ""))) {
                ps.setInt(1, id);
                try (ResultSet rs = ps.executeQuery()) {
                    if (!rs.next()) throw new SQLException("no product " + id);
                    return new Product(rs.getInt(1), rs.getString(2), rs.getInt(3), rs.getInt(4));
                }
            }
        }

        void setStock(int id, int stock) throws SQLException {
            try (PreparedStatement ps = c.prepareStatement("update products set stock = ? where id = ?")) {
                ps.setInt(1, stock); ps.setInt(2, id);
                ps.executeUpdate();
            }
        }

        /** Atomic in SQL: decrements only if enough stock remains. Returns whether a row changed. */
        boolean decrementIfAvailable(int id, int qty) throws SQLException {
            try (PreparedStatement ps = c.prepareStatement("update products set stock = stock - ? where id = ? and stock >= ?")) {
                ps.setInt(1, qty); ps.setInt(2, id); ps.setInt(3, qty);
                return ps.executeUpdate() == 1;
            }
        }

        /** Optimistic: succeeds only if nobody changed the row since we read it. */
        boolean updateIfUnchanged(Product read, int newStock) throws SQLException {
            try (PreparedStatement ps = c.prepareStatement("update products set stock = ?, version = version + 1 where id = ? and version = ?")) {
                ps.setInt(1, newStock); ps.setInt(2, read.id()); ps.setInt(3, read.version());
                return ps.executeUpdate() == 1;
            }
        }
    }

    interface Purchase { boolean buy(ProductRepository repo, Connection c, AtomicInteger retries) throws SQLException; }

    static boolean naive(ProductRepository repo, Connection c, AtomicInteger retries) throws SQLException {
        Product p = repo.find(1, false);                                      // read...
        if (p.stock() < 1) return false;
        Thread.onSpinWait();
        repo.setStock(1, p.stock() - 1);                                      // ...then write: another thread may have written in between
        return true;
    }

    static boolean atomicSql(ProductRepository repo, Connection c, AtomicInteger retries) throws SQLException {
        return repo.decrementIfAvailable(1, 1);
    }

    static boolean pessimistic(ProductRepository repo, Connection c, AtomicInteger retries) throws SQLException {
        c.setAutoCommit(false);
        try {
            Product p = repo.find(1, true);                                   // select ... for update: locks the row until commit
            boolean ok = p.stock() >= 1;
            if (ok) repo.setStock(1, p.stock() - 1);
            c.commit();
            return ok;
        } catch (SQLException e) {
            c.rollback();
            throw e;
        } finally {
            c.setAutoCommit(true);
        }
    }

    static boolean optimistic(ProductRepository repo, Connection c, AtomicInteger retries) throws SQLException {
        while (true) {
            Product p = repo.find(1, false);
            if (p.stock() < 1) return false;
            if (repo.updateIfUnchanged(p, p.stock() - 1)) return true;        // version matched: our write won
            retries.incrementAndGet();                                        // someone else got there first: re-read and retry
        }
    }

    static void run(String name, Purchase purchase) throws Exception {
        int stock = 500, threads = 16, attemptsEach = 50;                     // 800 attempts for 500 items
        try (Connection admin = DriverManager.getConnection(URL, "SA", ""); Statement s = admin.createStatement()) {
            s.execute("drop table products if exists");
            s.execute("create table products (id int primary key, name varchar(40), stock int not null, version int default 0 not null)");
            s.execute("insert into products values (1, 'limited edition', " + stock + ", 0)");
        }
        AtomicInteger sold = new AtomicInteger(), retries = new AtomicInteger(), errors = new AtomicInteger();
        CountDownLatch start = new CountDownLatch(1);
        long t = System.nanoTime();
        try (ExecutorService pool = Executors.newFixedThreadPool(threads)) {
            for (int i = 0; i < threads; i++) {
                pool.submit(() -> {
                    try (Connection c = DriverManager.getConnection(URL, "SA", "")) {     // one connection per thread
                        ProductRepository repo = new ProductRepository(c);
                        start.await();
                        for (int k = 0; k < attemptsEach; k++) {
                            try {
                                if (purchase.buy(repo, c, retries)) sold.incrementAndGet();
                            } catch (SQLException e) {
                                errors.incrementAndGet();                                 // e.g. serialization failure: counted, not sold
                            }
                        }
                    } catch (Exception e) {
                        errors.incrementAndGet();
                    }
                    return null;
                });
            }
            start.countDown();
        }
        long ms = (System.nanoTime() - t) / 1_000_000;
        try (Connection c = DriverManager.getConnection(URL, "SA", "")) {
            int left = new ProductRepository(c).find(1, false).stock();
            int oversold = sold.get() - (stock - left);
            System.out.printf("%-12s sold %3d, stock left %3d, %s, retries %d, errors %d, %d ms%n", name, sold.get(), left,
                    oversold == 0 && sold.get() <= stock ? "consistent" : "INCONSISTENT: " + oversold + " sales not reflected in stock", retries.get(), errors.get(), ms);
        }
    }

    public static void main(String[] args) throws Exception {
        run("naive", Inventory::naive);
        run("atomic SQL", Inventory::atomicSql);
        run("pessimistic", Inventory::pessimistic);
        run("optimistic", Inventory::optimistic);
    }
}
```

```console
$ cd inventory && java -cp ../../../libs/hsqldb-2.2.8.jar Inventory.java
naive        sold 800, stock left 443, INCONSISTENT: 743 sales not reflected in stock, retries 0, errors 0, 449 ms
atomic SQL   sold 500, stock left   0, consistent, retries 0, errors 0, 104 ms
pessimistic  sold 500, stock left   0, consistent, retries 0, errors 0, 107 ms
optimistic   sold 500, stock left   0, consistent, retries 7309, errors 0, 830 ms
```

Sixteen customers made 800 attempts to buy 500 items:

- **Naive read-then-write oversold badly.** It confirmed all 800 purchases — 300 more than the 500 items that existed — while the stock column fell by only a few dozen: threads read the same stock value, each wrote "stock − 1", and most decrements overwrote one another. Each statement was correct; the *sequence* was not atomic. In a real shop this is selling items that do not exist.
- **Atomic SQL** let the database do the check and the decrement in one statement: exactly 500 sold, stock 0. When the operation fits in one statement, this is the simplest and fastest fix.
- **Pessimistic locking** (`select … for update`) made each purchase wait for the row: correct, at the price of holding a lock for the whole transaction. A warning from writing this project: the first version ran the database in its multi-version (`mvcc`) mode, and there this old HSQLDB release did **not** block on `select … for update` — the "pessimistic" variant oversold just like the naive one. Locking behaviour differs between databases, versions, and transaction modes; a concurrency test like this one is the only way to know what yours does.
- **Optimistic locking** never blocks: each purchase re-reads and retries when its version check fails. It is correct, but the retry count — a thousand or more retries for 500 sales — shows the cost when everyone fights over one row: it was the slowest correct variant here. Under the low contention typical of web applications (different users editing different rows), retries are rare and optimistic locking is the usual choice.

### Review checklist

- In `naive`, which exact interleaving of two threads loses a sale? Write it as four steps.
- Why does each thread need its own `Connection`? (A JDBC connection is one session with one transaction; sharing it interleaves statements.)
- The optimistic version loops without limit. What should a production version do after N failed retries, and what should the user see?

## Answers to the Chapter 12 prediction questions

**12.5 Q1.** No. With auto-commit, the first update was committed as its own transaction when it completed. To make the pair atomic, turn auto-commit off and commit after both.

**12.5 Q2.** Exactly `select * from users where name = ?` with one parameter whose value is that whole string. The statement was parsed before the value was supplied, so the value can never become SQL; no table is dropped, and no user has that name.

# Chapter 13 — Building Services: Design Patterns, Layered Architecture, and Spring Boot

*Goal: structure a Java service the way production systems are structured — the design patterns that still matter in modern Java, a layered architecture with dependency injection, a REST API with validation and consistent errors — first in plain Java so that nothing is hidden, then mapped to Spring Boot.*

> **A note on what was run.** The plain-Java service in this chapter uses only the JDK and was run and tested as shown. **Spring Boot** needs libraries from Maven Central, which the build machine cannot reach; the Spring code is shown side by side with the plain-Java code it corresponds to, marked *not run here*.

**What you will be able to do after this chapter**

- Apply the classic patterns in their modern Java forms: strategy, factory, builder, decorator, observer, template method, adapter.
- Separate a service into API, service, and repository layers, wired by constructor injection.
- Design a REST API: resources, status codes, validation, error format, idempotency.
- Test a service at two levels: fast unit tests and HTTP-level tests.
- Read and write the equivalent Spring Boot application.

## Topic 13A — Patterns

## Unit 13.1 — Design patterns in modern Java

**One-line summary:** Design patterns name recurring solutions. Lambdas, records, sealed types, and default methods have shrunk many of them: a **strategy** is a functional interface; a **factory** is a static method or a `Supplier`; a **template method** is a function parameter; a **visitor** is a `switch` over a sealed type (Chapter 3). **Builder**, **decorator**, **adapter**, and **observer** keep their shape but lose their boilerplate.

```java
// file: PatternsDemo.java
import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Supplier;

public class PatternsDemo {
    // Strategy: behaviour passed as a function
    interface Pricing { long priceCents(long base, int quantity); }
    static final Pricing REGULAR = (base, q) -> base * q;
    static final Pricing BULK = (base, q) -> q >= 10 ? base * q * 90 / 100 : base * q;

    // Factory: a named static method chooses the implementation
    static Pricing pricingFor(String customerType) {
        return switch (customerType) {
            case "wholesale" -> BULK;
            case "retail" -> REGULAR;
            default -> throw new IllegalArgumentException("unknown customer type " + customerType);
        };
    }

    // Builder: readable construction of an object with many optional parts
    record HttpCall(String method, String url, Map<String, String> headers, Duration timeout, int retries) {
        static Builder to(String url) { return new Builder(url); }
        static final class Builder {
            private final String url;
            private String method = "GET";
            private final Map<String, String> headers = new java.util.TreeMap<>();
            private Duration timeout = Duration.ofSeconds(10);
            private int retries = 0;
            private Builder(String url) { this.url = url; }
            Builder method(String m) { this.method = m; return this; }
            Builder header(String k, String v) { headers.put(k, v); return this; }
            Builder timeout(Duration d) { this.timeout = d; return this; }
            Builder retries(int n) { this.retries = n; return this; }
            HttpCall build() {
                if (retries < 0) throw new IllegalStateException("retries must be >= 0");
                return new HttpCall(method, url, Map.copyOf(headers), timeout, retries);
            }
        }
    }

    // Decorator: wrap a function to add behaviour (caching, timing, retrying) without changing it
    static <K, V> Function<K, V> cached(Function<K, V> f) {
        Map<K, V> cache = new ConcurrentHashMap<>();
        return k -> cache.computeIfAbsent(k, f);
    }
    static <K, V> Function<K, V> counted(Function<K, V> f, int[] calls) {
        return k -> { calls[0]++; return f.apply(k); };
    }

    // Observer: subscribers are Consumers
    static final class EventBus<E> {
        private final List<Consumer<? super E>> subscribers = new ArrayList<>();
        Runnable subscribe(Consumer<? super E> s) { subscribers.add(s); return () -> subscribers.remove(s); }   // returns "unsubscribe"
        void publish(E event) { subscribers.forEach(s -> s.accept(event)); }
    }

    // Template method as a higher-order function: the fixed skeleton takes the varying step
    static <T> T timed(String label, Supplier<T> step) {
        long t = System.nanoTime();
        try {
            return step.get();
        } finally {
            System.out.println("  [" + label + " took " + (System.nanoTime() - t) / 1_000_000 + " ms]");
        }
    }

    static long slowLookup(String key) {
        try { Thread.sleep(50); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        return key.length() * 100L;
    }

    public static void main(String[] args) {
        System.out.println("strategy + factory: retail " + pricingFor("retail").priceCents(250, 12) + ", wholesale " + pricingFor("wholesale").priceCents(250, 12));
        System.out.println("builder: " + HttpCall.to("https://example.org/api").method("POST").header("Accept", "application/json").retries(2).build());
        int[] calls = {0};
        Function<String, Long> lookup = cached(counted(PatternsDemo::slowLookup, calls));
        long total = timed("three lookups, two of them cached", () -> lookup.apply("alpha") + lookup.apply("alpha") + lookup.apply("alpha"));
        System.out.println("decorator: total " + total + ", underlying function called " + calls[0] + " time(s)");
        EventBus<String> bus = new EventBus<>();
        List<String> audit = new ArrayList<>();
        Runnable unsubscribe = bus.subscribe(audit::add);
        bus.subscribe(e -> System.out.println("  observer got: " + e));
        bus.publish("order-created");
        unsubscribe.run();
        bus.publish("order-shipped");
        System.out.println("observer: audit saw " + audit);
    }
}
```

```console
$ java PatternsDemo.java
strategy + factory: retail 3000, wholesale 2700
builder: HttpCall[method=POST, url=https://example.org/api, headers={Accept=application/json}, timeout=PT10S, retries=2]
  [three lookups, two of them cached took 50 ms]
decorator: total 1500, underlying function called 1 time(s)
  observer got: order-created
  observer got: order-shipped
observer: audit saw [order-created]
```

### Deep & Tricky

**DT-45. Dependency injection is a pattern, not a framework: it means a class receives its collaborators instead of creating or looking them up.** *(Design fact)* `new OrderService(repository, clock, mailer)` *is* dependency injection. A framework (Spring, Guice, CDI) only automates building the object graph — finding implementations, calling constructors in the right order, managing lifecycles. Understanding the plain form explains the framework's errors ("no qualifying bean", circular dependencies) and its rules (prefer constructor injection: dependencies are explicit, fields can be `final`, objects are never half-built).

## Topic 13B — A service in layers

## Unit 13.2 — Layers and REST design

**One-line summary:** A typical service has three layers. The **API layer** translates HTTP to method calls and back (routing, parsing, status codes); it contains no business rules. The **service layer** holds the business logic and transaction boundaries, and knows nothing about HTTP. The **repository layer** stores and retrieves domain objects, behind an interface. Dependencies point inward: API → service → repository interface.

REST conventions used below:

| Request | Meaning | Success | Failure |
| --- | --- | --- | --- |
| `GET /tasks` | list (with `?done=` filter) | 200 + array | |
| `POST /tasks` | create | 201 + `Location` header + body | 400 invalid or not JSON |
| `GET /tasks/{id}` | read one | 200 | 404 |
| `PUT /tasks/{id}` | replace (idempotent) | 200 | 400, 404, 409 version conflict |
| `DELETE /tasks/{id}` | delete (idempotent) | 204 | 404 |

Errors use one JSON shape everywhere — `{"error": "...", "message": "...", "details": [...]}` — so clients can handle them uniformly.

## Chapter 13 Project — A task service in plain Java

**Goal:** a REST service for tasks, built from the JDK alone: a tiny JSON reader and writer, a router, three layers wired by constructor injection in `main`, validation with field-level error messages, optimistic concurrency on update, and tests at the HTTP level using `HttpClient`. Then the same service as a Spring Boot application.

### JSON in 120 lines

Real projects use Jackson. Writing a small JSON library once is worth it: it is a clean example of a **recursive-descent parser**, and it makes clear what a JSON library does.

```java
// file: tasks/Json.java
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

/** A minimal JSON reader/writer. Objects are Maps, arrays are Lists, numbers are Long or Double. */
public final class Json {
    private Json() {}

    public static final class ParseException extends RuntimeException {
        private static final long serialVersionUID = 1L;
        ParseException(String message, int position) { super(message + " at position " + position); }
    }

    public static String write(Object value) {
        StringBuilder sb = new StringBuilder();
        write(value, sb);
        return sb.toString();
    }

    private static void write(Object v, StringBuilder sb) {
        switch (v) {
            case null -> sb.append("null");
            case String s -> writeString(s, sb);
            case Number n -> sb.append(n);
            case Boolean b -> sb.append(b);
            case Map<?, ?> m -> {
                sb.append('{');
                int i = 0;
                for (var e : m.entrySet()) {
                    if (i++ > 0) sb.append(',');
                    writeString(String.valueOf(e.getKey()), sb);
                    sb.append(':');
                    write(e.getValue(), sb);
                }
                sb.append('}');
            }
            case Iterable<?> it -> {
                sb.append('[');
                int i = 0;
                for (Object x : it) {
                    if (i++ > 0) sb.append(',');
                    write(x, sb);
                }
                sb.append(']');
            }
            default -> throw new IllegalArgumentException("cannot serialise " + v.getClass());
        }
    }

    private static void writeString(String s, StringBuilder sb) {
        sb.append('"');
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            switch (c) {
                case '"' -> sb.append("\\\"");
                case '\\' -> sb.append("\\\\");
                case '\n' -> sb.append("\\n");
                case '\r' -> sb.append("\\r");
                case '\t' -> sb.append("\\t");
                default -> { if (c < 0x20) sb.append(String.format("\\u%04x", (int) c)); else sb.append(c); }
            }
        }
        sb.append('"');
    }

    public static Object parse(String text) {
        Parser p = new Parser(text);
        Object v = p.value();
        p.skipSpace();
        if (p.pos != text.length()) throw new ParseException("unexpected trailing characters", p.pos);
        return v;
    }

    private static final class Parser {
        final String s;
        int pos = 0;
        int depth = 0;
        Parser(String s) { this.s = s; }

        void skipSpace() { while (pos < s.length() && " \t\r\n".indexOf(s.charAt(pos)) >= 0) pos++; }

        char peek() {
            skipSpace();
            if (pos >= s.length()) throw new ParseException("unexpected end of input", pos);
            return s.charAt(pos);
        }

        void expect(char c) {
            if (peek() != c) throw new ParseException("expected '" + c + "'", pos);
            pos++;
        }

        Object value() {
            char c = peek();
            if (c == '{') return object();
            if (c == '[') return array();
            if (c == '"') return string();
            if (s.startsWith("true", pos)) { pos += 4; return Boolean.TRUE; }
            if (s.startsWith("false", pos)) { pos += 5; return Boolean.FALSE; }
            if (s.startsWith("null", pos)) { pos += 4; return null; }
            if (c == '-' || Character.isDigit(c)) return number();
            throw new ParseException("unexpected character '" + c + "'", pos);
        }

        Map<String, Object> object() {
            if (++depth > 64) throw new ParseException("nesting too deep", pos);       // a parser must bound recursion
            expect('{');
            Map<String, Object> m = new LinkedHashMap<>();
            if (peek() == '}') { pos++; depth--; return m; }
            do {
                if (peek() != '"') throw new ParseException("expected a string key", pos);
                String key = string();
                expect(':');
                m.put(key, value());
            } while (tryConsume(','));
            expect('}');
            depth--;
            return m;
        }

        List<Object> array() {
            if (++depth > 64) throw new ParseException("nesting too deep", pos);
            expect('[');
            List<Object> l = new ArrayList<>();
            if (peek() == ']') { pos++; depth--; return l; }
            do { l.add(value()); } while (tryConsume(','));
            expect(']');
            depth--;
            return l;
        }

        boolean tryConsume(char c) { if (peek() == c) { pos++; return true; } return false; }

        String string() {
            pos++;                                                                  // opening quote
            StringBuilder sb = new StringBuilder();
            while (true) {
                if (pos >= s.length()) throw new ParseException("unterminated string", pos);
                char c = s.charAt(pos++);
                if (c == '"') return sb.toString();
                if (c != '\\') { sb.append(c); continue; }
                if (pos >= s.length()) throw new ParseException("unterminated escape", pos);
                char e = s.charAt(pos++);
                switch (e) {
                    case '"', '\\', '/' -> sb.append(e);
                    case 'n' -> sb.append('\n');
                    case 't' -> sb.append('\t');
                    case 'r' -> sb.append('\r');
                    case 'b' -> sb.append('\b');
                    case 'f' -> sb.append('\f');
                    case 'u' -> {
                        if (pos + 4 > s.length()) throw new ParseException("bad \\u escape", pos);
                        try { sb.append((char) Integer.parseInt(s.substring(pos, pos + 4), 16)); }
                        catch (NumberFormatException ex) { throw new ParseException("bad \\u escape", pos); }
                        pos += 4;
                    }
                    default -> throw new ParseException("bad escape \\" + e, pos - 1);
                }
            }
        }

        Number number() {
            int start = pos;
            if (s.charAt(pos) == '-') pos++;
            while (pos < s.length() && (Character.isDigit(s.charAt(pos)) || ".eE+-".indexOf(s.charAt(pos)) >= 0)) pos++;
            String t = s.substring(start, pos);
            try {
                return t.matches("-?\\d{1,18}") ? (Number) Long.valueOf(t) : (Number) Double.valueOf(t);
            } catch (NumberFormatException e) {
                throw new ParseException("bad number '" + t + "'", start);
            }
        }
    }
}
```

### Domain, repository, service

```java
// file: tasks/Task.java
import java.time.Instant;
import java.util.LinkedHashMap;
import java.util.Map;

/** A task. `version` increases on every update (optimistic concurrency, Unit 12.5). */
public record Task(long id, String title, boolean done, int priority, Instant createdAt, int version) {
    Map<String, Object> toJson() {
        Map<String, Object> m = new LinkedHashMap<>();
        m.put("id", id);
        m.put("title", title);
        m.put("done", done);
        m.put("priority", priority);
        m.put("createdAt", createdAt.toString());
        m.put("version", version);
        return m;
    }
}
```

```java
// file: tasks/TaskRepository.java
import java.util.List;
import java.util.Optional;

/** Storage for tasks. The service layer depends on this interface, not on an implementation. */
public interface TaskRepository {
    Task insert(String title, int priority);
    Optional<Task> find(long id);
    List<Task> findAll();
    /** Replaces the task if its stored version equals `expected.version()`. */
    boolean update(Task expected, Task replacement);
    boolean delete(long id);
}
```

```java
// file: tasks/InMemoryTaskRepository.java
import java.time.Clock;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

public final class InMemoryTaskRepository implements TaskRepository {
    private final Map<Long, Task> tasks = new ConcurrentHashMap<>();
    private final AtomicLong nextId = new AtomicLong(1);
    private final Clock clock;

    public InMemoryTaskRepository(Clock clock) { this.clock = clock; }

    @Override public Task insert(String title, int priority) {
        Task t = new Task(nextId.getAndIncrement(), title, false, priority, clock.instant(), 1);
        tasks.put(t.id(), t);
        return t;
    }
    @Override public Optional<Task> find(long id) { return Optional.ofNullable(tasks.get(id)); }
    @Override public List<Task> findAll() { return tasks.values().stream().sorted(Comparator.comparingLong(Task::id)).toList(); }
    @Override public boolean update(Task expected, Task replacement) { return tasks.replace(expected.id(), expected, replacement); }   // atomic compare-and-set
    @Override public boolean delete(long id) { return tasks.remove(id) != null; }
}
```

```java
// file: tasks/TaskService.java
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Optional;

/** Business rules. Knows nothing about HTTP or JSON syntax. */
public final class TaskService {
    public static final class ValidationException extends RuntimeException {
        private static final long serialVersionUID = 1L;
        final transient List<String> problems;                 // never serialised; -Xlint:serial would object otherwise
        ValidationException(List<String> problems) { super("validation failed"); this.problems = List.copyOf(problems); }
    }
    public static final class NotFoundException extends RuntimeException {
        private static final long serialVersionUID = 1L;
        NotFoundException(long id) { super("no task with id " + id); }
    }
    public static final class ConflictException extends RuntimeException {
        private static final long serialVersionUID = 1L;
        ConflictException(String message) { super(message); }
    }

    private final TaskRepository repository;

    public TaskService(TaskRepository repository) { this.repository = repository; }

    public Task create(Map<String, Object> input) {
        List<String> problems = new ArrayList<>();
        String title = title(input, problems);
        int priority = priority(input, problems, 3);
        if (!problems.isEmpty()) throw new ValidationException(problems);
        return repository.insert(title, priority);
    }

    public Task get(long id) { return repository.find(id).orElseThrow(() -> new NotFoundException(id)); }

    public List<Task> list(Optional<Boolean> done) {
        return repository.findAll().stream().filter(t -> done.isEmpty() || t.done() == done.get()).toList();
    }

    public Task replace(long id, Map<String, Object> input) {
        Task current = get(id);
        List<String> problems = new ArrayList<>();
        String title = title(input, problems);
        int priority = priority(input, problems, current.priority());
        Object done = input.getOrDefault("done", current.done());
        if (!(done instanceof Boolean)) problems.add("done: must be true or false");
        Object version = input.get("version");
        if (!(version instanceof Long)) problems.add("version: required (the version you last read)");
        if (!problems.isEmpty()) throw new ValidationException(problems);
        if ((Long) version != current.version()) {
            throw new ConflictException("task " + id + " was modified: you sent version " + version + ", current is " + current.version());
        }
        Task updated = new Task(id, title, (Boolean) done, priority, current.createdAt(), current.version() + 1);
        if (!repository.update(current, updated)) throw new ConflictException("task " + id + " was modified concurrently");
        return updated;
    }

    public void delete(long id) {
        if (!repository.delete(id)) throw new NotFoundException(id);
    }

    private static String title(Map<String, Object> input, List<String> problems) {
        if (!(input.get("title") instanceof String s) || s.isBlank()) {
            problems.add("title: required, a non-blank string");
            return "";
        }
        if (s.length() > 100) problems.add("title: at most 100 characters");
        return s.strip();
    }

    private static int priority(Map<String, Object> input, List<String> problems, int defaultValue) {
        Object p = input.getOrDefault("priority", (long) defaultValue);
        if (p instanceof Long l && l >= 1 && l <= 5) return l.intValue();
        problems.add("priority: an integer from 1 to 5");
        return defaultValue;
    }
}
```

### The API layer

```java
// file: tasks/TaskApi.java
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpServer;
import java.io.IOException;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.Executors;

/** HTTP in, HTTP out. Translates requests to service calls and exceptions to status codes. */
public final class TaskApi {
    private final TaskService service;

    public TaskApi(TaskService service) { this.service = service; }

    public HttpServer start(int port) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), port), 0);
        server.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        server.createContext("/tasks", this::handle);
        server.start();
        return server;
    }

    private record Response(int status, Object body, String location) {}

    private void handle(HttpExchange ex) throws IOException {
        Response r;
        try {
            r = route(ex);
        } catch (TaskService.ValidationException e) {
            r = error(400, "validation_failed", e.getMessage(), e.problems);
        } catch (Json.ParseException e) {
            r = error(400, "malformed_json", e.getMessage(), List.of());
        } catch (TaskService.NotFoundException e) {
            r = error(404, "not_found", e.getMessage(), List.of());
        } catch (TaskService.ConflictException e) {
            r = error(409, "conflict", e.getMessage(), List.of());
        } catch (RuntimeException e) {                                       // a bug: log it, reveal nothing
            System.err.println("internal error: " + e);
            r = error(500, "internal_error", "unexpected error", List.of());
        }
        byte[] bytes = r.body() == null ? new byte[0] : Json.write(r.body()).getBytes(StandardCharsets.UTF_8);
        if (r.location() != null) ex.getResponseHeaders().add("Location", r.location());
        if (bytes.length > 0) ex.getResponseHeaders().add("Content-Type", "application/json; charset=utf-8");
        ex.sendResponseHeaders(r.status(), bytes.length == 0 ? -1 : bytes.length);
        try (var os = ex.getResponseBody()) { os.write(bytes); }
    }

    private Response route(HttpExchange ex) throws IOException {
        String method = ex.getRequestMethod();
        String path = ex.getRequestURI().getPath();
        if (path.equals("/tasks") || path.equals("/tasks/")) {
            return switch (method) {
                case "GET" -> {
                    String q = ex.getRequestURI().getQuery();
                    Optional<Boolean> done = q != null && q.startsWith("done=") ? Optional.of(Boolean.parseBoolean(q.substring(5))) : Optional.empty();
                    yield new Response(200, service.list(done).stream().map(Task::toJson).toList(), null);
                }
                case "POST" -> {
                    Task t = service.create(body(ex));
                    yield new Response(201, t.toJson(), "/tasks/" + t.id());
                }
                default -> error(405, "method_not_allowed", method + " is not supported on /tasks", List.of());
            };
        }
        long id;
        try {
            id = Long.parseLong(path.substring("/tasks/".length()));
        } catch (NumberFormatException e) {
            return error(404, "not_found", "no such resource: " + path, List.of());
        }
        return switch (method) {
            case "GET" -> new Response(200, service.get(id).toJson(), null);
            case "PUT" -> new Response(200, service.replace(id, body(ex)).toJson(), null);
            case "DELETE" -> { service.delete(id); yield new Response(204, null, null); }
            default -> error(405, "method_not_allowed", method + " is not supported on " + path, List.of());
        };
    }

    @SuppressWarnings("unchecked")
    private static Map<String, Object> body(HttpExchange ex) throws IOException {
        String type = ex.getRequestHeaders().getFirst("Content-Type");
        if (type == null || !type.startsWith("application/json")) throw new TaskService.ValidationException(List.of("Content-Type: must be application/json"));
        byte[] raw = ex.getRequestBody().readNBytes(64 * 1024 + 1);          // bound the request size
        if (raw.length > 64 * 1024) throw new TaskService.ValidationException(List.of("body: larger than 64 KiB"));
        Object parsed = Json.parse(new String(raw, StandardCharsets.UTF_8));
        if (!(parsed instanceof Map)) throw new TaskService.ValidationException(List.of("body: must be a JSON object"));
        return (Map<String, Object>) parsed;
    }

    private static Response error(int status, String code, String message, List<String> details) {
        Map<String, Object> m = new LinkedHashMap<>();
        m.put("error", code);
        m.put("message", message);
        if (!details.isEmpty()) m.put("details", details);
        return new Response(status, m, null);
    }
}
```

### Wiring and HTTP-level tests

`main` is the **composition root**: the only place that knows which implementations are used.

```java
// file: tasks/Main.java
import com.sun.net.httpserver.HttpServer;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Clock;
import java.time.Instant;
import java.time.ZoneOffset;
import java.util.List;
import java.util.Map;

public class Main {
    static HttpClient client = HttpClient.newHttpClient();
    static String base;
    static int passed = 0, failed = 0;

    static HttpResponse<String> call(String method, String path, String json) throws Exception {
        HttpRequest.Builder b = HttpRequest.newBuilder(URI.create(base + path));
        if (json != null) b.header("Content-Type", "application/json");
        b.method(method, json == null ? HttpRequest.BodyPublishers.noBody() : HttpRequest.BodyPublishers.ofString(json));
        return client.send(b.build(), HttpResponse.BodyHandlers.ofString());
    }

    static void check(String what, boolean ok) {
        if (ok) passed++; else { failed++; System.out.println("FAILED: " + what); }
    }

    @SuppressWarnings("unchecked")
    public static void main(String[] args) throws Exception {
        // composition root: build the object graph by hand (this is what a DI container automates)
        Clock clock = Clock.fixed(Instant.parse("2026-11-15T12:00:00Z"), ZoneOffset.UTC);
        TaskRepository repository = new InMemoryTaskRepository(clock);
        TaskService service = new TaskService(repository);
        HttpServer server = new TaskApi(service).start(0);
        base = "http://127.0.0.1:" + server.getAddress().getPort();

        var r = call("POST", "/tasks", "{\"title\": \"write chapter 13\", \"priority\": 1}");
        System.out.println("POST /tasks -> " + r.statusCode() + " Location: " + r.headers().firstValue("location").orElse("?") + "\n  " + r.body());
        check("create returns 201", r.statusCode() == 201);
        call("POST", "/tasks", "{\"title\": \"review \\\"quotes\\\" and unicode \\u00e9\"}");
        call("POST", "/tasks", "{\"title\": \"ship it\", \"priority\": 5}");

        r = call("GET", "/tasks/2", null);
        System.out.println("GET /tasks/2 -> " + r.statusCode() + " " + r.body());
        check("title round-trips with escapes", ((Map<String, Object>) Json.parse(r.body())).get("title").equals("review \"quotes\" and unicode é"));

        r = call("PUT", "/tasks/1", "{\"title\": \"write chapter 13\", \"done\": true, \"version\": 1}");
        System.out.println("PUT /tasks/1 (version 1) -> " + r.statusCode() + " " + r.body());
        check("update returns 200 and bumps version", r.statusCode() == 200 && r.body().contains("\"version\":2"));
        r = call("PUT", "/tasks/1", "{\"title\": \"stale edit\", \"version\": 1}");
        System.out.println("PUT /tasks/1 (stale version 1) -> " + r.statusCode() + " " + r.body());
        check("stale update is a 409", r.statusCode() == 409);

        r = call("GET", "/tasks?done=false", null);
        check("filter by done", ((List<Object>) Json.parse(r.body())).size() == 2);
        System.out.println("GET /tasks?done=false -> " + ((List<Object>) Json.parse(r.body())).size() + " tasks");

        String[][] bad = {
            {"POST", "/tasks", "{\"title\": \"  \", \"priority\": 9}"}, {"POST", "/tasks", "{\"title\": \"x\""}, {"POST", "/tasks", "[1, 2]"},
            {"GET", "/tasks/999", null}, {"GET", "/tasks/abc", null}, {"PATCH", "/tasks/1", "{}"}, {"DELETE", "/tasks/3", null}, {"DELETE", "/tasks/3", null},
        };
        int[] expected = {400, 400, 400, 404, 404, 405, 204, 404};
        for (int i = 0; i < bad.length; i++) {
            r = call(bad[i][0], bad[i][1], bad[i][2]);
            System.out.printf("%-6s %-10s %-34s -> %d %s%n", bad[i][0], bad[i][1], bad[i][2] == null ? "" : bad[i][2], r.statusCode(), r.body());
            check(bad[i][0] + " " + bad[i][1] + " expected " + expected[i], r.statusCode() == expected[i]);
        }
        var noType = client.send(HttpRequest.newBuilder(URI.create(base + "/tasks")).POST(HttpRequest.BodyPublishers.ofString("{}")).build(), HttpResponse.BodyHandlers.ofString());
        check("missing content type rejected", noType.statusCode() == 400);
        String deep = "[".repeat(100) + "]".repeat(100);
        check("deeply nested JSON rejected, not a stack overflow", call("POST", "/tasks", deep).statusCode() == 400);

        server.stop(0);
        System.out.println(passed + " checks passed, " + failed + " failed");
    }
}
```

```console
$ cd tasks && rm -rf out && javac -Xlint:all -Werror -d out *.java && java -cp out Main
POST /tasks -> 201 Location: /tasks/1
  {"id":1,"title":"write chapter 13","done":false,"priority":1,"createdAt":"2026-11-15T12:00:00Z","version":1}
GET /tasks/2 -> 200 {"id":2,"title":"review \"quotes\" and unicode é","done":false,"priority":3,"createdAt":"2026-11-15T12:00:00Z","version":1}
PUT /tasks/1 (version 1) -> 200 {"id":1,"title":"write chapter 13","done":true,"priority":1,"createdAt":"2026-11-15T12:00:00Z","version":2}
PUT /tasks/1 (stale version 1) -> 409 {"error":"conflict","message":"task 1 was modified: you sent version 1, current is 2"}
GET /tasks?done=false -> 2 tasks
POST   /tasks     {"title": "  ", "priority": 9}     -> 400 {"error":"validation_failed","message":"validation failed","details":["title: required, a non-blank string","priority: an integer from 1 to 5"]}
POST   /tasks     {"title": "x"                      -> 400 {"error":"malformed_json","message":"unexpected end of input at position 13"}
POST   /tasks     [1, 2]                             -> 400 {"error":"validation_failed","message":"validation failed","details":["body: must be a JSON object"]}
GET    /tasks/999                                    -> 404 {"error":"not_found","message":"no task with id 999"}
GET    /tasks/abc                                    -> 404 {"error":"not_found","message":"no such resource: /tasks/abc"}
PATCH  /tasks/1   {}                                 -> 405 {"error":"method_not_allowed","message":"PATCH is not supported on /tasks/1"}
DELETE /tasks/3                                      -> 204 
DELETE /tasks/3                                      -> 404 {"error":"not_found","message":"no task with id 3"}
15 checks passed, 0 failed
```

Every layer has one job. The service can be unit-tested with a fake repository and no HTTP at all (P-9); the HTTP-level tests above check what unit tests cannot: status codes, headers, JSON round-trips with escapes, and that hostile input (a hundred nested brackets) is rejected rather than crashing the parser.

### The same service in Spring Boot (*not run here*)

Spring Boot supplies what we wrote by hand: the HTTP server and router (Spring MVC on embedded Tomcat), JSON (Jackson), validation (Jakarta Bean Validation), the object graph (the Spring container), and error mapping.

```java
// Task.java — a record again; Jackson serialises records directly
public record Task(long id, String title, boolean done, int priority, Instant createdAt, int version) {}

// The request body, with declarative validation instead of hand-written checks
public record TaskInput(
        @NotBlank @Size(max = 100) String title,
        @Min(1) @Max(5) Integer priority,
        Boolean done,
        Integer version) {}

@Service                                            // a bean: the container creates it and injects the repository
public class TaskService {
    private final TaskRepository repository;
    public TaskService(TaskRepository repository) { this.repository = repository; }   // constructor injection
    // create, get, list, replace, delete: the same logic as in the plain-Java version
}

@RestController
@RequestMapping("/tasks")
public class TaskController {
    private final TaskService service;
    public TaskController(TaskService service) { this.service = service; }

    @GetMapping
    public List<Task> list(@RequestParam Optional<Boolean> done) { return service.list(done); }

    @PostMapping
    public ResponseEntity<Task> create(@Valid @RequestBody TaskInput input) {       // 400 automatically if invalid
        Task t = service.create(input);
        return ResponseEntity.created(URI.create("/tasks/" + t.id())).body(t);
    }

    @GetMapping("/{id}")
    public Task get(@PathVariable long id) { return service.get(id); }

    @PutMapping("/{id}")
    public Task replace(@PathVariable long id, @Valid @RequestBody TaskInput input) { return service.replace(id, input); }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable long id) { service.delete(id); }
}

@RestControllerAdvice                               // one place that maps exceptions to responses
public class ApiErrors {
    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ProblemDetail> notFound(NotFoundException e) {
        return ResponseEntity.status(404).body(ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage()));
    }
    @ExceptionHandler(ConflictException.class)
    public ResponseEntity<ProblemDetail> conflict(ConflictException e) {
        return ResponseEntity.status(409).body(ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, e.getMessage()));
    }
}

@SpringBootApplication                              // component scan + auto-configuration + embedded server
public class TasksApplication {
    public static void main(String[] args) { SpringApplication.run(TasksApplication.class, args); }
}
```

With Spring Data JPA, the repository becomes an interface whose implementation is generated — `public interface TaskRepository extends JpaRepository<TaskEntity, Long> { List<TaskEntity> findByDone(boolean done); }` — with `@Version` on the entity's version field providing the optimistic locking we wrote by hand. Tests use `@WebMvcTest` with `MockMvc` for the controller layer and `@SpringBootTest` for the whole application.

| Concern | Plain Java (this chapter) | Spring Boot |
| --- | --- | --- |
| Object graph | `new` calls in `main` | component scan, constructor injection |
| Routing | a `switch` on method and path | `@GetMapping`, `@PathVariable` |
| JSON | `Json.parse` / `Json.write` | Jackson, automatic |
| Validation | hand-written checks collecting messages | `@Valid`, `@NotBlank`, `@Size` |
| Errors | `catch` blocks in the handler | `@RestControllerAdvice`, `ProblemDetail` (RFC 9457) |
| Configuration | constructor arguments | `application.yml`, `@ConfigurationProperties`, profiles |
| Persistence | `TaskRepository` implementation | Spring Data, `@Transactional` |
| Operations | (Chapter 14 builds it) | Actuator: health, metrics, tracing |

### Pattern

**P-13. Keep business logic in plain classes with constructor-injected dependencies; let the framework stay at the edges.** *(Practice: hexagonal / "ports and adapters" architecture)* A `TaskService` that depends only on a repository interface runs in a unit test, a command-line tool, or any framework. Controllers, repositories, and configuration are adapters around it. *Trade-off:* a few more interfaces and mapping code between layers; in a small CRUD service the layers can feel ceremonial — keep them thin rather than removing them.

### Mastery check

1. **Predict.** A client sends the same `PUT /tasks/1` request (with `"version": 1`) twice. What are the two responses, and is the operation idempotent?
2. **Predict.** In Spring, two beans each take the other as a constructor parameter. What happens at start-up?

### Review checklist

- Where would you add authentication? Which layer should know who the user is, and which should decide what they may do?
- The repository's `update` is a compare-and-set. Which HTTP status does a lost race produce, and what should a client do next?
- `Json` bounds nesting depth and the API bounds body size. What other limits does a public endpoint need? (Rate limits, timeouts — Chapter 14.)

## Answers to the Chapter 13 prediction questions

**13.2 Q1.** The first returns 200 with version 2; the second returns 409, because version 1 is now stale. The *effect* is idempotent — the task ends in the same state however many times the request is sent — but the responses differ. Clients treat the 409 as "re-read, then decide".

**13.2 Q2.** The application fails to start with `BeanCurrentlyInCreationException` ("requested bean is currently in creation: is there an unresolvable circular reference?"). Constructor injection makes cycles impossible to build — which is a feature: break the cycle by extracting the shared logic into a third class.

# Chapter 14 — Capstone: A Production-Style REST Service

*Goal: design, build, test, and measure a complete service — a URL shortener with a REST API, a SQL database behind a connection pool, a read-through cache, write-behind hit counting, rate limiting, metrics, health checks, structured logs, and graceful shutdown — using everything in this book.*

This chapter adds no new language features. The system is **linkd**: clients create short links, visitors are redirected through them, and each link counts its visits. It is small in features and deliberately complete in the things that separate a program from a service: it survives restarts, protects itself from overload, and can be observed while it runs.

Like the rest of the book, it is built on the JDK alone (plus the HSQLDB jar from Chapter 12 for JDBC), so that every moving part is visible — about 600 lines. Unit 14.7 maps each part to the library a production team would use instead.

**Capstone deliverables**

1. A design note: requirements, architecture, decisions (Unit 14.1).
2. Persistence: schema, repository, connection pool (Unit 14.2).
3. The cache and the write-behind counter (Unit 14.3).
4. Protection and observability: rate limiter, metrics, health, access log (Unit 14.4).
5. The service and API layers, and graceful shutdown (Unit 14.5).
6. A self-test over HTTP, a restart test, and a load test with and without the cache (Unit 14.6).
7. A design review (Unit 14.7).

## Topic 14A — Design

## Unit 14.1 — Requirements and architecture

### Requirements

- **R1 API:** `POST /links {"url": …}` → 201 with a short code; `GET /r/{code}` → 302 redirect; `GET /links/{code}` → link details and hit count; consistent JSON errors (Chapter 13).
- **R2 Persistence:** links and hit counts are stored in a SQL database and survive a restart.
- **R3 Performance:** redirects — by far the most frequent request — do not query the database when the link was used recently; counting a hit never delays the redirect.
- **R4 Protection:** per-client rate limiting (429 with `Retry-After`); bounded request size; bounded database concurrency; timeouts.
- **R5 Observability:** `GET /metrics` (request counts by route and status, latency histogram, cache and pool statistics), `GET /health`, and a structured access log.
- **R6 Graceful shutdown:** stop accepting requests, finish in-flight ones, flush pending hit counts, close the database cleanly.

### Architecture

```text
            HTTP (virtual thread per request)
                        |
   +--------------------v--------------------+
   | Api: routing, JSON, status codes,       |    Metrics  <---- every layer reports
   |      rate limit, access log             |
   +--------------------+--------------------+
                        |
   +--------------------v--------------------+
   | LinkService: validation, code creation, |
   |   resolve = cache -> repository         |----> HitCounter (in memory)
   +--------------------+--------------------+            |  flushed in batches
                        |                                 |  every 200 ms
   +--------------------v--------------------+            |
   | LinkRepository (JDBC) <- ConnectionPool |<-----------+
   +--------------------+--------------------+
                        |
                   SQL database
```

### Key decisions

| Decision | Choice | Reason, and the alternative |
|---|---|---|
| Threads | one virtual thread per request | blocking JDBC and simple code scale (Unit 10.7); the pool, not the thread count, limits database concurrency |
| Reads | read-through LRU cache with TTL | links are immutable once created, so caching is safe; TTL bounds staleness if that ever changes |
| Hit counts | accumulate in memory, flush in batches | a redirect must not wait for a database write; cost: up to 200 ms of counts lost in a crash |
| Codes | base-62 of a scrambled sequence number | short, unique without a lookup; not secret |
| Negative results | cached too (briefly) | a flood of requests for unknown codes must not reach the database each time |

## Topic 14B — Build

## Unit 14.2 — Persistence: connection pool and repository

```java
// file: linkd/ConnectionPool.java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.LongAdder;

/** A fixed-size JDBC connection pool. Borrowers wait, up to a timeout, when all connections are in use. */
public final class ConnectionPool implements AutoCloseable {
    public interface Work<T> { T run(Connection c) throws SQLException; }

    private final BlockingQueue<Connection> idle;
    private final int size;
    private final long timeoutMillis;
    final LongAdder borrows = new LongAdder(), waits = new LongAdder(), timeouts = new LongAdder();

    public ConnectionPool(String url, int size, long timeoutMillis) throws SQLException {
        this.size = size;
        this.timeoutMillis = timeoutMillis;
        this.idle = new ArrayBlockingQueue<>(size);
        for (int i = 0; i < size; i++) idle.add(DriverManager.getConnection(url, "SA", ""));
    }

    /** Runs `work` with a pooled connection and always returns the connection, rolled back to a clean state on failure. */
    public <T> T with(Work<T> work) throws SQLException {
        Connection c = idle.poll();
        if (c == null) {
            waits.increment();
            try {
                c = idle.poll(timeoutMillis, TimeUnit.MILLISECONDS);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new SQLException("interrupted while waiting for a connection");
            }
            if (c == null) {
                timeouts.increment();
                throw new SQLException("no database connection available within " + timeoutMillis + " ms");
            }
        }
        borrows.increment();
        try {
            return work.run(c);
        } catch (SQLException | RuntimeException e) {
            try {
                if (!c.getAutoCommit()) { c.rollback(); c.setAutoCommit(true); }      // DT-44: never return a dirty connection
            } catch (SQLException cleanup) {
                e.addSuppressed(cleanup);
            }
            throw e;
        } finally {
            idle.add(c);
        }
    }

    public int idleCount() { return idle.size(); }
    public int size() { return size; }

    @Override public void close() throws SQLException {
        for (int i = 0; i < size; i++) {
            Connection c = idle.poll();
            if (c != null) c.close();
        }
    }
}
```

```java
// file: linkd/LinkRepository.java
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.time.Instant;
import java.util.Map;
import java.util.Optional;

/** All SQL for links. */
public final class LinkRepository {
    public record Link(long id, String code, String url, Instant createdAt, long hits) {}

    private final ConnectionPool pool;

    public LinkRepository(ConnectionPool pool) throws SQLException {
        this.pool = pool;
        pool.with(c -> {
            try (Statement s = c.createStatement()) {
                s.execute("create cached table if not exists links (id bigint generated by default as identity (start with 1) primary key, "
                        + "code varchar(16) unique, url varchar(2000) not null, created_at bigint not null, hits bigint default 0 not null)");
            }
            return null;
        });
    }

    /** Inserts the link, then stores the code derived from its generated id — in one transaction. */
    public Link insert(String url, Instant now, java.util.function.LongFunction<String> codeForId) throws SQLException {
        return pool.with(c -> {
            c.setAutoCommit(false);
            long id;
            try (PreparedStatement ins = c.prepareStatement("insert into links (url, created_at) values (?, ?)", Statement.RETURN_GENERATED_KEYS)) {
                ins.setString(1, url);
                ins.setLong(2, now.toEpochMilli());
                ins.executeUpdate();
                try (ResultSet keys = ins.getGeneratedKeys()) { keys.next(); id = keys.getLong(1); }
            }
            String code = codeForId.apply(id);
            try (PreparedStatement upd = c.prepareStatement("update links set code = ? where id = ?")) {
                upd.setString(1, code);
                upd.setLong(2, id);
                upd.executeUpdate();
            }
            c.commit();
            c.setAutoCommit(true);
            return new Link(id, code, url, now, 0);
        });
    }

    public Optional<Link> findByCode(String code) throws SQLException {
        return pool.with(c -> {
            try (PreparedStatement ps = c.prepareStatement("select id, code, url, created_at, hits from links where code = ?")) {
                ps.setString(1, code);
                try (ResultSet rs = ps.executeQuery()) {
                    if (!rs.next()) return Optional.<Link>empty();
                    return Optional.of(new Link(rs.getLong(1), rs.getString(2), rs.getString(3), Instant.ofEpochMilli(rs.getLong(4)), rs.getLong(5)));
                }
            }
        });
    }

    /** Adds the given hit counts in one batch and one transaction. */
    public void addHits(Map<String, Long> hitsByCode) throws SQLException {
        if (hitsByCode.isEmpty()) return;
        pool.with(c -> {
            c.setAutoCommit(false);
            try (PreparedStatement ps = c.prepareStatement("update links set hits = hits + ? where code = ?")) {
                for (var e : hitsByCode.entrySet()) {
                    ps.setLong(1, e.getValue());
                    ps.setString(2, e.getKey());
                    ps.addBatch();
                }
                ps.executeBatch();
            }
            c.commit();
            c.setAutoCommit(true);
            return null;
        });
    }

    public long count() throws SQLException {
        return pool.with(c -> {
            try (Statement s = c.createStatement(); ResultSet rs = s.executeQuery("select count(*) from links")) { rs.next(); return rs.getLong(1); }
        });
    }

    /** Asks the database to write everything to disk and stop (HSQLDB needs this for file databases). */
    public void shutdownDatabase() throws SQLException {
        pool.with(c -> { try (Statement s = c.createStatement()) { s.execute("shutdown"); } return null; });
    }
}
```

## Unit 14.3 — The cache and the write-behind counter

```java
// file: linkd/Cache.java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.atomic.LongAdder;
import java.util.function.LongSupplier;

/** A thread-safe LRU cache with per-entry expiry. Capacity 0 disables it. */
public final class Cache<K, V> {
    private record Entry<V>(V value, long expiresAtNanos) {}

    private final int capacity;
    private final LongSupplier nanoClock;
    private final LinkedHashMap<K, Entry<V>> map;
    final LongAdder hits = new LongAdder(), misses = new LongAdder(), evictions = new LongAdder();

    public Cache(int capacity, LongSupplier nanoClock) {
        this.capacity = capacity;
        this.nanoClock = nanoClock;
        this.map = new LinkedHashMap<>(16, 0.75f, true) {                      // access order (Chapter 6)
            @Override protected boolean removeEldestEntry(Map.Entry<K, Entry<V>> eldest) {
                boolean evict = size() > Cache.this.capacity;
                if (evict) evictions.increment();
                return evict;
            }
        };
    }

    public Optional<V> get(K key) {
        if (capacity == 0) { misses.increment(); return Optional.empty(); }
        synchronized (map) {                                                    // even get() reorders an access-order map
            Entry<V> e = map.get(key);
            if (e != null && e.expiresAtNanos() - nanoClock.getAsLong() > 0) {
                hits.increment();
                return Optional.of(e.value());
            }
            if (e != null) map.remove(key);                                     // expired
        }
        misses.increment();
        return Optional.empty();
    }

    public void put(K key, V value, long ttlNanos) {
        if (capacity == 0) return;
        synchronized (map) { map.put(key, new Entry<>(value, nanoClock.getAsLong() + ttlNanos)); }
    }

    public int size() { synchronized (map) { return map.size(); } }
}
```

```java
// file: linkd/HitCounter.java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.LongAdder;

/** Counts hits in memory; a background thread writes them to the database in batches (write-behind). */
public final class HitCounter implements AutoCloseable {
    private final ConcurrentHashMap<String, LongAdder> pending = new ConcurrentHashMap<>();
    private final LinkRepository repository;
    private final Thread flusher;
    private volatile boolean running = true;
    final LongAdder flushes = new LongAdder(), flushedHits = new LongAdder(), flushFailures = new LongAdder();

    public HitCounter(LinkRepository repository, long intervalMillis) {
        this.repository = repository;
        this.flusher = Thread.ofPlatform().name("hit-flusher").daemon(true).start(() -> {
            while (running) {
                try {
                    Thread.sleep(intervalMillis);
                } catch (InterruptedException e) {
                    break;                                                      // close() interrupts to stop promptly
                }
                flush();
            }
        });
    }

    public void record(String code) {
        pending.computeIfAbsent(code, k -> new LongAdder()).increment();        // no lock, no I/O on the request path
    }

    /** Hits recorded but not yet in the database, for one code. */
    public long pendingFor(String code) {
        LongAdder a = pending.get(code);
        return a == null ? 0 : a.sum();
    }

    public synchronized void flush() {
        Map<String, Long> batch = new HashMap<>();
        pending.forEach((code, adder) -> {
            long n = adder.sumThenReset();
            if (n > 0) batch.put(code, n);
        });
        if (batch.isEmpty()) return;
        try {
            repository.addHits(batch);
            flushes.increment();
            flushedHits.add(batch.values().stream().mapToLong(Long::longValue).sum());
        } catch (java.sql.SQLException e) {
            flushFailures.increment();
            batch.forEach((code, n) -> pending.computeIfAbsent(code, k -> new LongAdder()).add(n));   // put them back: retry next time
        }
    }

    @Override public void close() {
        running = false;
        flusher.interrupt();
        try { flusher.join(2000); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        flush();                                                                // the final flush on shutdown
    }
}
```

### Deep & Tricky

**DT-46. A cache is a second copy of the truth; decide what happens when the copies differ before adding one.** *(Design fact)* Caching is safe here because a link never changes after creation, and unknown codes are cached only briefly (a link created a moment later would otherwise stay "not found"). For mutable data the questions are: how stale may a read be (TTL)? Who invalidates on write — and what if the invalidation is lost? What happens when a popular entry expires and a thousand requests miss at once (a **cache stampede**; mitigated by per-key loading locks, as Caffeine's `LoadingCache` does)? A cache that returns wrong data quickly is worse than no cache.

## Unit 14.4 — Protection and observability

```java
// file: linkd/RateLimiter.java
import java.util.concurrent.ConcurrentHashMap;
import java.util.function.LongSupplier;

/** One token bucket per client (Chapter 9's project, made thread-safe). */
public final class RateLimiter {
    private static final class Bucket {
        double tokens;
        long last;
    }
    private final ConcurrentHashMap<String, Bucket> buckets = new ConcurrentHashMap<>();
    private final double capacity, refillPerNano;
    private final LongSupplier nanoClock;

    public RateLimiter(double capacity, double refillPerSecond, LongSupplier nanoClock) {
        this.capacity = capacity;
        this.refillPerNano = refillPerSecond / 1e9;
        this.nanoClock = nanoClock;
    }

    /** Returns 0 if the request is allowed, otherwise the number of milliseconds until a token is available. */
    public long tryAcquire(String client) {
        long now = nanoClock.getAsLong();
        Bucket b = buckets.computeIfAbsent(client, k -> { Bucket nb = new Bucket(); nb.tokens = capacity; nb.last = now; return nb; });
        synchronized (b) {                                                     // one small lock per client
            b.tokens = Math.min(capacity, b.tokens + (now - b.last) * refillPerNano);
            b.last = now;
            if (b.tokens >= 1) { b.tokens -= 1; return 0; }
            return (long) Math.ceil((1 - b.tokens) / refillPerNano / 1e6);
        }
    }
}
```

```java
// file: linkd/Metrics.java
import java.util.Map;
import java.util.TreeMap;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.LongAdder;
import java.util.function.LongSupplier;

/** Counters, gauges, and one latency histogram, rendered in the Prometheus text format. */
public final class Metrics {
    private static final double[] BUCKETS_MS = {0.25, 0.5, 1, 2.5, 5, 10, 25, 50, 100, 250, 1000};
    private final ConcurrentHashMap<String, LongAdder> counters = new ConcurrentHashMap<>();
    private final ConcurrentHashMap<String, LongSupplier> gauges = new ConcurrentHashMap<>();
    private final LongAdder[] latency = new LongAdder[BUCKETS_MS.length + 1];
    private final LongAdder latencySumMicros = new LongAdder();

    public Metrics() { for (int i = 0; i < latency.length; i++) latency[i] = new LongAdder(); }

    public void count(String name) { counters.computeIfAbsent(name, k -> new LongAdder()).increment(); }
    public void gauge(String name, LongSupplier value) { gauges.put(name, value); }
    public long counter(String name) { LongAdder a = counters.get(name); return a == null ? 0 : a.sum(); }

    public void observeLatency(long nanos) {
        double ms = nanos / 1e6;
        int i = 0;
        while (i < BUCKETS_MS.length && ms > BUCKETS_MS[i]) i++;
        latency[i].increment();
        latencySumMicros.add(nanos / 1000);
    }

    /** An upper bound for the given quantile, from the histogram buckets. */
    public double latencyQuantileMs(double q) {
        long total = 0;
        for (LongAdder a : latency) total += a.sum();
        long target = (long) Math.ceil(total * q), seen = 0;
        for (int i = 0; i < latency.length; i++) {
            seen += latency[i].sum();
            if (seen >= target) return i < BUCKETS_MS.length ? BUCKETS_MS[i] : Double.POSITIVE_INFINITY;
        }
        return 0;
    }

    public String render() {
        StringBuilder sb = new StringBuilder();
        new TreeMap<>(counters).forEach((k, v) -> sb.append(k).append(' ').append(v.sum()).append('\n'));
        for (Map.Entry<String, LongSupplier> g : new TreeMap<>(gauges).entrySet()) sb.append(g.getKey()).append(' ').append(g.getValue().getAsLong()).append('\n');
        long cumulative = 0;
        for (int i = 0; i < BUCKETS_MS.length; i++) {
            cumulative += latency[i].sum();
            sb.append("http_request_duration_ms_bucket{le=\"").append(BUCKETS_MS[i]).append("\"} ").append(cumulative).append('\n');
        }
        cumulative += latency[BUCKETS_MS.length].sum();
        sb.append("http_request_duration_ms_bucket{le=\"+Inf\"} ").append(cumulative).append('\n');
        sb.append("http_request_duration_ms_count ").append(cumulative).append('\n');
        sb.append("http_request_duration_ms_sum ").append(latencySumMicros.sum() / 1000.0).append('\n');
        return sb.toString();
    }
}
```

A histogram with fixed buckets is how production metrics systems record latency: cheap to update (one counter), mergeable across servers, and good enough to answer "what is p99?" to within a bucket. Averages are not recorded at all — they hide exactly the slow requests that matter.

## Unit 14.5 — Service, API, and shutdown

```java
// file: linkd/LinkService.java
import java.net.URI;
import java.sql.SQLException;
import java.time.Clock;
import java.util.Optional;

/** Business logic: validation, code generation, cached resolution. */
public final class LinkService {
    public static final class InvalidUrlException extends RuntimeException {
        private static final long serialVersionUID = 1L;
        InvalidUrlException(String message) { super(message); }
    }

    private static final String ALPHABET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    private static final long SPACE = 62L * 62 * 62 * 62 * 62 * 62;             // 6 characters: about 56.8 billion codes
    private static final long LINK_TTL = 60_000_000_000L, NEGATIVE_TTL = 2_000_000_000L;

    private final LinkRepository repository;
    private final Cache<String, Optional<String>> cache;                        // code -> url, or empty for "no such code"
    private final HitCounter hits;
    private final Clock clock;
    private final Metrics metrics;

    public LinkService(LinkRepository repository, Cache<String, Optional<String>> cache, HitCounter hits, Clock clock, Metrics metrics) {
        this.repository = repository;
        this.cache = cache;
        this.hits = hits;
        this.clock = clock;
        this.metrics = metrics;
    }

    /** Maps a sequence number to a 6-character code. Multiplying by a number coprime to SPACE is a bijection, so codes never collide. */
    static String codeFor(long id) {
        long n = Math.floorMod(id * 31_337_317L, SPACE);
        char[] out = new char[6];
        for (int i = 5; i >= 0; i--) { out[i] = ALPHABET.charAt((int) (n % 62)); n /= 62; }
        return new String(out);
    }

    public LinkRepository.Link create(String url) throws SQLException {
        if (url == null || url.isBlank()) throw new InvalidUrlException("url: required");
        if (url.length() > 2000) throw new InvalidUrlException("url: longer than 2000 characters");
        URI uri;
        try {
            uri = URI.create(url);
        } catch (IllegalArgumentException e) {
            throw new InvalidUrlException("url: not a valid URI");
        }
        if (uri.getScheme() == null || !(uri.getScheme().equals("http") || uri.getScheme().equals("https")) || uri.getHost() == null) {
            throw new InvalidUrlException("url: must be an absolute http or https URL");      // no javascript:, file:, or relative URLs
        }
        LinkRepository.Link link = repository.insert(url, clock.instant(), LinkService::codeFor);
        cache.put(link.code(), Optional.of(url), LINK_TTL);
        return link;
    }

    /** The redirect path: cache first; on a miss, one database read; then count the hit in memory. */
    public Optional<String> resolve(String code) throws SQLException {
        Optional<Optional<String>> cached = cache.get(code);
        Optional<String> url;
        if (cached.isPresent()) {
            url = cached.get();
        } else {
            metrics.count("linkd_db_lookups_total");
            url = repository.findByCode(code).map(LinkRepository.Link::url);
            cache.put(code, url, url.isPresent() ? LINK_TTL : NEGATIVE_TTL);
        }
        if (url.isPresent()) hits.record(code);
        return url;
    }

    /** Link details; the hit count includes hits not yet flushed to the database. */
    public Optional<LinkRepository.Link> details(String code) throws SQLException {
        return repository.findByCode(code).map(l -> new LinkRepository.Link(l.id(), l.code(), l.url(), l.createdAt(), l.hits() + hits.pendingFor(code)));
    }
}
```

```java
// file: linkd/Api.java
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpServer;
import java.io.IOException;
import java.io.PrintStream;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.nio.charset.StandardCharsets;
import java.sql.SQLException;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;

public final class Api {
    private final LinkService service;
    private final RateLimiter limiter;
    private final Metrics metrics;
    private final PrintStream accessLog;
    private final AtomicBoolean healthy = new AtomicBoolean(true);
    private HttpServer server;

    public Api(LinkService service, RateLimiter limiter, Metrics metrics, PrintStream accessLog) {
        this.service = service;
        this.limiter = limiter;
        this.metrics = metrics;
        this.accessLog = accessLog;
    }

    public int start(int port) throws IOException {
        server = HttpServer.create(new InetSocketAddress(InetAddress.getLoopbackAddress(), port), 1024);
        server.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
        server.createContext("/", this::handle);
        server.start();
        return server.getAddress().getPort();
    }

    /** Stops accepting connections and waits up to `seconds` for in-flight requests. */
    public void stop(int seconds) {
        healthy.set(false);
        server.stop(seconds);
    }

    private record Response(int status, String contentType, byte[] body, Map<String, String> headers) {
        static Response json(int status, Object value) { return new Response(status, "application/json", Json.write(value).getBytes(StandardCharsets.UTF_8), Map.of()); }
        static Response error(int status, String code, String message) {
            Map<String, Object> m = new LinkedHashMap<>();
            m.put("error", code);
            m.put("message", message);
            return json(status, m);
        }
    }

    private void handle(HttpExchange ex) throws IOException {
        long start = System.nanoTime();
        String method = ex.getRequestMethod(), path = ex.getRequestURI().getPath();
        String route = path.startsWith("/r/") ? "/r/{code}" : path.startsWith("/links/") ? "/links/{code}" : path;
        String client = Optional.ofNullable(ex.getRequestHeaders().getFirst("X-Client-Id")).orElse(ex.getRemoteAddress().getAddress().getHostAddress());
        Response r;
        try {
            long waitMs = route.equals("/metrics") || route.equals("/health") ? 0 : limiter.tryAcquire(client);
            if (waitMs > 0) {
                r = new Response(429, "application/json", Json.write(Map.of("error", "rate_limited", "message", "too many requests")).getBytes(StandardCharsets.UTF_8),
                        Map.of("Retry-After", String.valueOf(Math.max(1, (waitMs + 999) / 1000))));
            } else {
                r = route(method, path, route, ex);
            }
        } catch (LinkService.InvalidUrlException | Json.ParseException e) {
            r = Response.error(400, "bad_request", e.getMessage());
        } catch (SQLException e) {                                            // database unavailable or pool exhausted
            metrics.count("linkd_db_errors_total");
            r = new Response(503, "application/json", Json.write(Map.of("error", "unavailable", "message", "try again shortly")).getBytes(StandardCharsets.UTF_8),
                    Map.of("Retry-After", "1"));
        } catch (RuntimeException e) {
            System.err.println("internal error on " + method + " " + path + ": " + e);
            r = Response.error(500, "internal_error", "unexpected error");
        }
        r.headers().forEach((k, v) -> ex.getResponseHeaders().add(k, v));
        if (r.body().length > 0) ex.getResponseHeaders().add("Content-Type", r.contentType());
        ex.sendResponseHeaders(r.status(), r.body().length == 0 ? -1 : r.body().length);
        try (var os = ex.getResponseBody()) { os.write(r.body()); }
        long nanos = System.nanoTime() - start;
        metrics.count("http_requests_total{route=\"" + route + "\",status=\"" + r.status() + "\"}");
        metrics.observeLatency(nanos);
        if (accessLog != null) {                                              // one JSON object per line: easy to ship and query
            Map<String, Object> line = new LinkedHashMap<>();
            line.put("method", method);
            line.put("path", path);
            line.put("status", r.status());
            line.put("micros", nanos / 1000);
            line.put("client", client);
            accessLog.println(Json.write(line));
        }
    }

    @SuppressWarnings("unchecked")
    private Response route(String method, String path, String route, HttpExchange ex) throws IOException, SQLException {
        switch (route) {
            case "/health":
                return healthy.get() ? Response.json(200, Map.of("status", "up")) : Response.json(503, Map.of("status", "shutting down"));
            case "/metrics":
                return new Response(200, "text/plain; version=0.0.4", metrics.render().getBytes(StandardCharsets.UTF_8), Map.of());
            case "/links": {
                if (!method.equals("POST")) return Response.error(405, "method_not_allowed", method + " not supported");
                byte[] raw = ex.getRequestBody().readNBytes(8 * 1024 + 1);
                if (raw.length > 8 * 1024) return Response.error(413, "too_large", "body larger than 8 KiB");
                Object parsed = Json.parse(new String(raw, StandardCharsets.UTF_8));
                if (!(parsed instanceof Map<?, ?> m) || !(m.get("url") instanceof String url)) return Response.error(400, "bad_request", "expected {\"url\": \"...\"}");
                LinkRepository.Link link = service.create(url);
                Map<String, Object> body = new LinkedHashMap<>();
                body.put("code", link.code());
                body.put("url", link.url());
                body.put("short", "/r/" + link.code());
                return new Response(201, "application/json", Json.write(body).getBytes(StandardCharsets.UTF_8), Map.of("Location", "/links/" + link.code()));
            }
            case "/r/{code}": {
                if (!method.equals("GET") && !method.equals("HEAD")) return Response.error(405, "method_not_allowed", method + " not supported");
                Optional<String> url = service.resolve(path.substring(3));
                if (url.isEmpty()) return Response.error(404, "not_found", "no such link");
                return new Response(302, "", new byte[0], Map.of("Location", url.get(), "Cache-Control", "private, max-age=0"));
            }
            case "/links/{code}": {
                Optional<LinkRepository.Link> link = service.details(path.substring(7));
                if (link.isEmpty()) return Response.error(404, "not_found", "no such link");
                Map<String, Object> body = new LinkedHashMap<>();
                body.put("code", link.get().code());
                body.put("url", link.get().url());
                body.put("createdAt", link.get().createdAt().toString());
                body.put("hits", link.get().hits());
                return Response.json(200, body);
            }
            default:
                return Response.error(404, "not_found", "no such resource");
        }
    }
}
```

`Main` is the composition root (DT-45), and holds the three ways to run the program: `serve`, `selftest`, and `load`.

```java
// file: linkd/Main.java
import java.io.PrintStream;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.file.Files;
import java.nio.file.Path;
import java.time.Clock;
import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.LongAdder;

public class Main {
    /** The assembled application. */
    record App(Api api, int port, LinkRepository repository, HitCounter hits, ConnectionPool pool, Metrics metrics, Cache<String, Optional<String>> cache, PrintStream log)
            implements AutoCloseable {
        /** Graceful shutdown, in dependency order (R6). */
        @Override public void close() throws java.sql.SQLException {
            api.stop(2);                 // 1. stop accepting, let in-flight requests finish
            hits.close();                // 2. flush pending hit counts
            repository.shutdownDatabase();   // 3. database writes its files
            pool.close();                // 4. release connections
            if (log != null) log.close();
        }
    }

    static App start(String dbUrl, int port, int cacheCapacity, double ratePerSecond, Path accessLog) throws Exception {
        Metrics metrics = new Metrics();
        ConnectionPool pool = new ConnectionPool(dbUrl, 8, 1000);
        LinkRepository repository = new LinkRepository(pool);
        Cache<String, Optional<String>> cache = new Cache<>(cacheCapacity, System::nanoTime);
        HitCounter hits = new HitCounter(repository, 200);
        LinkService service = new LinkService(repository, cache, hits, Clock.systemUTC(), metrics);
        RateLimiter limiter = new RateLimiter(ratePerSecond, ratePerSecond, System::nanoTime);
        PrintStream log = accessLog == null ? null : new PrintStream(Files.newOutputStream(accessLog), false, "UTF-8");
        Api api = new Api(service, limiter, metrics, log);
        metrics.gauge("linkd_cache_hits_total", cache.hits::sum);
        metrics.gauge("linkd_cache_misses_total", cache.misses::sum);
        metrics.gauge("linkd_cache_size", cache::size);
        metrics.gauge("linkd_pool_idle_connections", pool::idleCount);
        metrics.gauge("linkd_pool_waits_total", pool.waits::sum);
        metrics.gauge("linkd_hits_flushed_total", hits.flushedHits::sum);
        return new App(api, api.start(port), repository, hits, pool, metrics, cache, log);
    }

    static final HttpClient CLIENT = HttpClient.newBuilder().followRedirects(HttpClient.Redirect.NEVER).connectTimeout(Duration.ofSeconds(2)).build();

    static HttpResponse<String> call(int port, String method, String path, String json, String clientId) throws Exception {
        HttpRequest.Builder b = HttpRequest.newBuilder(URI.create("http://127.0.0.1:" + port + path)).timeout(Duration.ofSeconds(5)).header("X-Client-Id", clientId);
        if (json != null) b.header("Content-Type", "application/json");
        b.method(method, json == null ? HttpRequest.BodyPublishers.noBody() : HttpRequest.BodyPublishers.ofString(json));
        return CLIENT.send(b.build(), HttpResponse.BodyHandlers.ofString());
    }

    @SuppressWarnings("unchecked")
    static String field(String json, String name) { return String.valueOf(((Map<String, Object>) Json.parse(json)).get(name)); }

    static int checks = 0, failures = 0;
    static void check(String what, boolean ok) {
        checks++;
        if (!ok) { failures++; System.out.println("  FAILED: " + what); }
    }

    static void selftest(String dbUrl) throws Exception {
        String code;
        try (App app = start(dbUrl, 0, 1000, 50, Path.of("access.log"))) {
            int port = app.port();
            var created = call(port, "POST", "/links", "{\"url\": \"https://example.org/a/long/path?q=1\"}", "tester");
            code = field(created.body(), "code");
            System.out.println("POST /links -> " + created.statusCode() + " " + created.body());
            check("create is 201 with a Location", created.statusCode() == 201 && created.headers().firstValue("location").orElse("").equals("/links/" + code));
            var redirect = call(port, "GET", "/r/" + code, null, "tester");
            System.out.println("GET /r/" + code + " -> " + redirect.statusCode() + " Location: " + redirect.headers().firstValue("location").orElse("?"));
            check("redirect is 302 to the stored URL", redirect.statusCode() == 302 && redirect.headers().firstValue("location").orElse("").equals("https://example.org/a/long/path?q=1"));
            for (int i = 0; i < 9; i++) call(port, "GET", "/r/" + code, null, "tester");
            check("hits are visible at once (pending included)", field(call(port, "GET", "/links/" + code, null, "tester").body(), "hits").equals("10"));
            check("only the first lookup could miss the cache; create() primed it", app.metrics().counter("linkd_db_lookups_total") == 0);
            check("unknown code is 404", call(port, "GET", "/r/zzzzzz", null, "tester").statusCode() == 404);
            for (String bad : List.of("{\"url\": \"javascript:alert(1)\"}", "{\"url\": \"/relative\"}", "{\"url\": \"\"}", "{\"nope\": 1}", "{not json", "{\"url\": \"http://x.org/" + "a".repeat(2100) + "\"}")) {
                check("rejected: " + bad.substring(0, Math.min(30, bad.length())), call(port, "POST", "/links", bad, "tester").statusCode() == 400);
            }
            check("oversized body is 413", call(port, "POST", "/links", "{\"url\": \"http://x.org/" + "a".repeat(9000) + "\"}", "tester").statusCode() == 413);
            check("wrong method is 405", call(port, "DELETE", "/links", null, "tester").statusCode() == 405);

            int limited = 0;
            String retryAfter = "";
            for (int i = 0; i < 120; i++) {                                    // a greedy client: bucket of 50, refilled at 50/s
                var r = call(port, "GET", "/r/" + code, null, "greedy");
                if (r.statusCode() == 429) { limited++; retryAfter = r.headers().firstValue("retry-after").orElse(""); }
            }
            System.out.println("greedy client: " + (120 - limited) + " of 120 allowed, " + limited + " rejected with 429, Retry-After: " + retryAfter);
            check("rate limiter rejected a greedy client", limited > 5 && !retryAfter.isEmpty());
            check("other clients are unaffected", call(port, "GET", "/r/" + code, null, "polite").statusCode() == 302);
            check("health is up", call(port, "GET", "/health", null, "tester").body().contains("up"));
            String metrics = call(port, "GET", "/metrics", null, "tester").body();
            System.out.println("a few lines of /metrics:");
            metrics.lines().filter(l -> l.contains("/r/{code}") || l.startsWith("linkd_cache_hits") || l.contains("le=\"1.0\"") || l.contains("_count")).forEach(l -> System.out.println("  " + l));
        }                                                                      // graceful shutdown: flush hits, close database
        System.out.println("access log: " + Files.readAllLines(Path.of("access.log")).size() + " lines; first: " + Files.readAllLines(Path.of("access.log")).get(0));

        try (App app = start(dbUrl, 0, 1000, 50, null)) {                       // restart on the same database files
            var details = call(app.port(), "GET", "/links/" + code, null, "tester");
            System.out.println("after restart: GET /links/" + code + " -> " + details.body());
            check("link and all hits survived the restart", details.statusCode() == 200 && Long.parseLong(field(details.body(), "hits")) >= 11);
            check("redirect works after restart (one database lookup, then cached)", call(app.port(), "GET", "/r/" + code, null, "tester").statusCode() == 302
                    && call(app.port(), "GET", "/r/" + code, null, "tester").statusCode() == 302 && app.metrics().counter("linkd_db_lookups_total") == 1);
        }
        System.out.println(checks + " checks, " + failures + " failed");
    }

    static void load(int cacheCapacity, int links, int requests, int concurrency) throws Exception {
        try (App app = start("jdbc:hsqldb:mem:load" + cacheCapacity, 0, cacheCapacity, 1e9, null)) {
            int port = app.port();
            List<String> codes = new ArrayList<>();
            for (int i = 0; i < links; i++) codes.add(field(call(port, "POST", "/links", "{\"url\": \"https://example.org/page/" + i + "\"}", "loader").body(), "code"));
            long lookupsBefore = app.metrics().counter("linkd_db_lookups_total");
            AtomicInteger next = new AtomicInteger();
            LongAdder ok = new LongAdder(), notFound = new LongAdder(), other = new LongAdder();
            long[] latencies = new long[requests];
            long t0 = System.nanoTime();
            try (var pool = Executors.newVirtualThreadPerTaskExecutor()) {
                for (int w = 0; w < concurrency; w++) {
                    pool.submit(() -> {
                        java.util.Random rnd = new java.util.Random(Thread.currentThread().threadId());
                        for (int i; (i = next.getAndIncrement()) < requests; ) {
                            // 80% of requests go to the most popular 5% of links; 2% ask for codes that do not exist
                            String code = rnd.nextInt(100) < 2 ? "nope" + rnd.nextInt(50) : codes.get(rnd.nextInt(100) < 80 ? rnd.nextInt(Math.max(1, links / 20)) : rnd.nextInt(links));
                            long t = System.nanoTime();
                            int status = call(port, "GET", "/r/" + code, null, "loader").statusCode();
                            latencies[i] = System.nanoTime() - t;
                            (status == 302 ? ok : status == 404 ? notFound : other).increment();
                        }
                        return null;
                    });
                }
            }
            double seconds = (System.nanoTime() - t0) / 1e9;
            java.util.Arrays.sort(latencies);
            app.hits().flush();
            long counted = 0;
            for (String c : codes) counted += Long.parseLong(field(call(port, "GET", "/links/" + c, null, "loader").body(), "hits"));
            System.out.printf("cache capacity %5d: %,d requests in %.1f s = %,.0f req/s; latency p50 %.2f ms, p99 %.2f ms, max %.1f ms%n", cacheCapacity, requests, seconds,
                    requests / seconds, latencies[requests / 2] / 1e6, latencies[(int) (requests * 0.99)] / 1e6, latencies[requests - 1] / 1e6);
            System.out.printf("  302: %d, 404: %d, other: %d; database lookups for redirects: %,d; cache hit ratio %.1f%%; pool waits %d; hits counted in database: %d (= 302s: %b)%n",
                    ok.sum(), notFound.sum(), other.sum(), app.metrics().counter("linkd_db_lookups_total") - lookupsBefore,
                    100.0 * app.cache().hits.sum() / Math.max(1, app.cache().hits.sum() + app.cache().misses.sum()), app.pool().waits.sum(), counted, counted == ok.sum());
        }
    }

    public static void main(String[] args) throws Exception {
        switch (args.length > 0 ? args[0] : "") {
            case "selftest" -> selftest("jdbc:hsqldb:file:data/linkd;hsqldb.tx=mvcc");
            case "load" -> {
                load(0, 2000, 30_000, 64);
                load(5000, 2000, 30_000, 64);
            }
            case "serve" -> {
                App app = start("jdbc:hsqldb:file:data/linkd;hsqldb.tx=mvcc", 8080, 10_000, 20, Path.of("access.log"));
                Runtime.getRuntime().addShutdownHook(new Thread(() -> {       // SIGTERM / Ctrl-C: shut down gracefully
                    try { app.close(); } catch (Exception e) { System.err.println("shutdown: " + e); }
                }));
                System.out.println("linkd listening on http://127.0.0.1:" + app.port());
                Thread.currentThread().join();
            }
            default -> System.out.println("usage: Main serve | selftest | load");
        }
    }
}
```

## Topic 14C — Verify and measure

## Unit 14.6 — Self-test, restart test, and load test

`Json.java` is Chapter 13's file, copied unchanged.

```console
$ cd linkd && cp ../../ch13/tasks/Json.java . && rm -rf out data access.log && javac -Xlint:all -Werror -cp ../../../libs/hsqldb-2.2.8.jar -d out *.java && java -cp out:../../../libs/hsqldb-2.2.8.jar Main selftest 2>&1 | grep -v 'org.hsqldb\|^INFO'
POST /links -> 201 {"code":"027UGb","url":"https://example.org/a/long/path?q=1","short":"/r/027UGb"}
GET /r/027UGb -> 302 Location: https://example.org/a/long/path?q=1
greedy client: 102 of 120 allowed, 18 rejected with 429, Retry-After: 1
a few lines of /metrics:
  http_requests_total{route="/r/{code}",status="302"} 113
  http_requests_total{route="/r/{code}",status="404"} 1
  http_requests_total{route="/r/{code}",status="429"} 18
  linkd_cache_hits_total 113
  http_request_duration_ms_bucket{le="1.0"} 115
  http_request_duration_ms_count 143
access log: 144 lines; first: {"method":"POST","path":"/links","status":201,"micros":60232,"client":"tester"}
after restart: GET /links/027UGb -> {"code":"027UGb","url":"https://example.org/a/long/path?q=1","createdAt":"2026-11-15T11:34:05.954Z","hits":113}
18 checks, 0 failed
$ cd linkd && ls data | sort | tr '\n' ' '
linkd.data linkd.properties linkd.script 
$ cd linkd && java -cp out:../../../libs/hsqldb-2.2.8.jar Main load 2>&1 | grep -v 'org.hsqldb\|^INFO'
cache capacity     0: 30,000 requests in 7.5 s = 4,023 req/s; latency p50 12.45 ms, p99 61.36 ms, max 130.3 ms
  302: 29433, 404: 567, other: 0; database lookups for redirects: 30,000; cache hit ratio 0.0%; pool waits 0; hits counted in database: 29433 (= 302s: true)
cache capacity  5000: 30,000 requests in 2.8 s = 10,838 req/s; latency p50 4.38 ms, p99 47.76 ms, max 65.8 ms
  302: 29376, 404: 624, other: 0; database lookups for redirects: 754; cache hit ratio 97.5%; pool waits 0; hits counted in database: 29376 (= 302s: true)
$ cd linkd && (java -cp out:../../../libs/hsqldb-2.2.8.jar Main serve > serve.log 2>&1 & echo $! > linkd.pid) && sleep 3 && curl -s -X POST -H 'Content-Type: application/json' -d '{"url":"https://openjdk.org/"}' http://127.0.0.1:8080/links && echo && CODE=$(curl -s -X POST -H 'Content-Type: application/json' -d '{"url":"https://dev.java/"}' http://127.0.0.1:8080/links | sed -E 's/.*"code":"([^"]+)".*/\1/') && curl -s -o /dev/null -w 'redirect: %{http_code} -> %{redirect_url}\n' http://127.0.0.1:8080/r/$CODE && curl -s http://127.0.0.1:8080/health && echo && kill -TERM $(cat linkd.pid) && sleep 3 && echo "process gone: $(! kill -0 $(cat linkd.pid) 2>/dev/null && echo yes)" && tail -n 2 access.log
{"code":"04EyXC","url":"https://openjdk.org/","short":"/r/04EyXC"}
redirect: 302 -> https://dev.java/
{"status":"up"}
process gone: yes
{"method":"GET","path":"/r/06MSnn","status":302,"micros":714,"client":"127.0.0.1"}
{"method":"GET","path":"/health","status":200,"micros":375,"client":"127.0.0.1"}
```

What the runs establish, requirement by requirement:

- **R1, R4 (API and protection).** The self-test exercised every route and every rejection: non-HTTP URLs (`javascript:`), relative URLs, missing fields, malformed JSON, over-long URLs, oversized bodies, wrong methods. The greedy client was cut off with 429 and a `Retry-After` header after exhausting its bucket, while another client was served normally.
- **R2 (persistence).** After a graceful shutdown and a restart on the same database files, the link was still there, with all of its hits — including those that were only in memory until the final flush.
- **R3 (performance).** The load test is the same workload twice. Without the cache, every one of the 30,000 redirects was a database lookup; with it, fewer than a thousand were — all for *unknown* codes, whose "not found" entries deliberately expire after two seconds (existing links were already cached when they were created). Throughput rose about 2.6 times and median latency fell to a third. The lookup count is slightly higher than the number of 404 responses: several concurrent requests for the same uncached code each queried the database before the first answer was cached — a small, real cache stampede (DT-46). In both runs the number of hits counted in the database equals the number of 302 responses exactly: write-behind counting lost nothing.
- **R5 (observability).** `/metrics` reports requests by route and status, a latency histogram, and cache statistics; the access log has one JSON object per request.
- **R6 (graceful shutdown).** `SIGTERM` to the running server triggered the shutdown hook; the process exited after flushing and closing the database.

### Deep & Tricky

**DT-47. A shutdown hook is the last code your service runs — keep it short, ordered, and bounded.** *(Guarantee: `Runtime.addShutdownHook`; OS behaviour)* Hooks run on `SIGTERM`, `Ctrl-C`, and `System.exit`, but not on `SIGKILL` or a JVM crash, and orchestrators (Kubernetes, systemd) send `SIGKILL` after a grace period (30 seconds by default in Kubernetes). So shutdown must be: stop accepting work; finish or abandon in-flight work within a deadline; flush buffers; close resources — in dependency order, each step with a timeout. Anything that *must* not be lost (payments, orders) cannot live only in a memory buffer waiting for a hook; this service accepts losing up to 200 ms of hit counts in a crash, and says so in its design.

## Unit 14.7 — Design review and where to go next

| Requirement | Status | Evidence |
|---|---|---|
| R1 API and errors | met | self-test over HTTP |
| R2 persistence across restart | met | restart test on the same files |
| R3 no database read on hot redirects; hits never block | met | database-lookup counter and timings in the load test |
| R4 rate limit, size limits, bounded pool, timeouts | met | 429 test, 413 test, pool with borrow timeout |
| R5 metrics, health, access log | met | `/metrics`, `/health`, `access.log` |
| R6 graceful shutdown | met | `SIGTERM` test; hits present after restart |

**What a production team would use instead of each hand-written part:**

| This chapter | In production |
| --- | --- |
| `HttpServer` + `switch` routing | Spring Boot (MVC or WebFlux), Quarkus, Micronaut, Helidon, Javalin |
| `Json` | Jackson |
| `ConnectionPool` | HikariCP |
| JDBC by hand | Spring `JdbcClient`, jOOQ, or JPA/Hibernate; schema migrations with Flyway or Liquibase |
| `Cache` | Caffeine (in-process), Redis (shared between instances) |
| `RateLimiter` | Bucket4j, Resilience4j, or the API gateway |
| `Metrics` | Micrometer + Prometheus; OpenTelemetry for traces |
| access log by `println` | SLF4J + Logback with a JSON encoder |
| `Main` wiring | the framework's dependency-injection container |
| tests in `main` | JUnit 5, Testcontainers (real databases in Docker), REST Assured |

**Known limitations and next steps** (pick at least two):

1. **More than one instance.** The cache, the rate limiter, and the hit buffer are per process. Behind a load balancer, move rate limiting and caching to Redis (or accept per-instance limits), and make hit flushing safe when several instances write.
2. **Cache stampede protection** (DT-46): when a popular code expires, only one request should load it.
3. **Custom aliases and expiry**, with conflict handling (409) and a uniqueness constraint doing the real work.
4. **Authentication and ownership:** API keys or OAuth 2 tokens; only the owner may see statistics.
5. **Abuse prevention:** blocklists for malicious destinations; limits per account, not just per IP.
6. **Tracing:** propagate a request id through logs and to the database (OpenTelemetry).
7. **Migrate to Spring Boot** using the mapping in Chapter 13, and compare lines of code, start-up time, and memory.
8. **A real load test** from another machine with a tool such as k6 or Gatling; the in-process test here shares CPU between client and server.

### Mastery check

1. **Predict.** The process is killed with `SIGKILL` 100 ms after a burst of 1,000 redirects. What is lost, and what is not?
2. **Predict.** The cache capacity is set to 10 while 2,000 links are in use with the load test's access pattern. What happens to the hit ratio, to correctness, and to the hit counts?

## Answers to the Chapter 14 prediction questions

**14.7 Q1.** The links themselves are safe — each was committed to the database when it was created. Hit counts recorded since the last flush (up to 200 ms of them, so possibly most of the 1,000) are lost: they existed only in `HitCounter`'s memory. The shutdown hook does not run on `SIGKILL`.

**14.7 Q2.** The hit ratio collapses — only the handful of most recently used codes are cached — so nearly every redirect reads the database and throughput approaches the no-cache run. Correctness is unaffected: the cache only ever stores what the database says. Hit counts are also unaffected: they go through `HitCounter`, not the cache.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | Java's backwards compatibility is strong at the bytecode level: classes compiled for Java 8 run on Java 21 unchanged — but not the other way round. | Unit 0.1 |
| DT-2 | Since Java 9, `+` on strings compiles to `invokedynamic`, and the JVM generates the concatenation code at run time. | Unit 0.3 |
| DT-3 | Reading a `static final` compile-time constant does not initialise its class, because `javac` copies the value into the caller. | Unit 0.4 |
| DT-4 | Integer overflow in Java is defined — it wraps — which makes it silent rather than catastrophic. | Unit 1.1 |
| DT-5 | `Integer` values from −128 to 127 are cached, so `==` happens to work for small numbers and fails for large ones. | Unit 1.2 |
| DT-6 | In an old-style `switch` *statement*, cases fall through to the next one unless you write `break`. The arrow form (`case X ->`), in both statements and expressions, never falls through, and a `switch` *expression* over an enum or sealed type must be exhaustive — the compiler rejects a missing case. | Unit 1.5 |
| DT-7 | `double` cannot represent most decimal fractions, so money must use `BigDecimal` (or integer cents) — and `new BigDecimal(0.1)` is already wrong. | Unit 1.6 |
| DT-8 | Closing a `Scanner` (or `BufferedReader`) that wraps `System.in` closes `System.in` itself, and it cannot be reopened: a second `new Scanner(System.in)` later in the program finds no input and throws `NoSuchElementException`. Create one `Scanner` for standard input for the whole program and do not close it — even though IDEs warn "resource leak". | Unit 1.7 |
| DT-9 | `final` makes a *reference* unchangeable, not the object it refers to — immutability needs defensive copies. | Unit 2.2 |
| DT-10 | `equals` and inheritance do not mix: there is no way to add a value field in a subclass and keep `equals` symmetric and transitive. | Unit 2.3 |
| DT-11 | Every instance of a non-static inner class holds a hidden reference to its outer object — a classic source of memory leaks. | Unit 2.6 |
| DT-12 | Calling an overridable method from a constructor runs the subclass's override before the subclass's fields are initialised. | Unit 3.1 |
| DT-13 | Order matters in pattern `switch`: a more general pattern before a more specific one is a compile error ("dominated"), and guards are checked in order. | Unit 3.5 |
| DT-14 | `protected` in Java is *wider* than in C++: a `protected` member is visible to every class in the same package, not only to subclasses. Package-private (no modifier) is visible to the package only. A class in another package that extends yours can use your `protected` members only through references of its own type (or subtypes). | Unit 3.6 |
| DT-15 | A `return` (or `throw`) in `finally` silently discards an exception thrown by the `try` block. | Unit 4.2 |
| DT-16 | HotSpot may throw pre-allocated exceptions *without* stack traces for frequently thrown built-in exceptions. | Unit 4.5 |
| DT-17 | `Optional.orElse(x)` always evaluates `x`, even when the `Optional` holds a value — so `opt.orElse(loadFromDatabase())` hits the database every time. Use `orElseGet(() -> loadFromDatabase())` for anything expensive or with side effects. | Unit 4.6 |
| DT-18 | Heap pollution: through raw types or unchecked casts, a `List<String>` can hold an `Integer`, and the error surfaces only when — and where — the value is read. | Unit 5.2 |
| DT-19 | The compiler generates *bridge methods* so that erased generic overrides still override. | Unit 5.2 |
| DT-20 | Arrays are covariant and checked at run time; generics are invariant and checked at compile time — so they do not mix. | Unit 5.3 |
| DT-21 | Raw types turn off generic checking: assigning a `List<String>` to a raw `List`, adding an `Integer`, and reading it back as a `String` compiles with only an "unchecked" warning and fails with `ClassCastException` far from the bad `add` — at the line that reads the element. This is called *heap pollution*. | Unit 5.4 |
| DT-22 | A poor `hashCode` turns a `HashMap` into a list — or, since Java 8, into a tree if the keys are `Comparable`. | Unit 6.3 |
| DT-23 | Modifying a collection while iterating over it throws `ConcurrentModificationException` — on a best-effort basis, even in a single thread. | Unit 6.3 |
| DT-24 | `Arrays.asList` returns a fixed-size list *backed by the array*: `set` writes through to the array, `add` and `remove` throw `UnsupportedOperationException`. And `List.of(...)` is fully immutable *and* rejects `null` elements — even `contains(null)` throws `NullPointerException`. | Unit 6.5 |
| DT-25 | A lambda is not an anonymous inner class: it compiles to a private method plus an `invokedynamic` call site, and a non-capturing lambda is a single shared instance. | Unit 7.1 |
| DT-26 | Streams are lazy, single-use, and must not have side effects that interfere with their source or depend on order. | Unit 7.3 |
| DT-27 | `Collectors.toMap` throws `IllegalStateException` on a duplicate key (unless you pass a merge function), and throws `NullPointerException` if any *value* is `null` — even though `HashMap` allows null values. `groupingBy` and `toList` have no such limits. | Unit 7.3 |
| DT-28 | Forgetting `flip()` is the classic `ByteBuffer` bug: after writing, position is at the end, so reading (or writing the buffer to a channel) sees nothing. | Unit 8.4 |
| DT-29 | A memory-mapped file cannot be unmapped explicitly; the mapping lives until the buffer is garbage collected. | Unit 8.4 |
| DT-30 | Java deserialisation runs code: `ObjectInputStream.readObject` instantiates whatever classes the byte stream names (if they are on the class path) and calls their `readObject`/`readResolve` methods, so deserialising untrusted data has led to remote code execution in many products ("gadget chains"). Never deserialise untrusted input with `ObjectInputStream`; if you must, install an `ObjectInputFilter` that allow-lists the expected classes. | Unit 8.5 |
| DT-31 | Java's regex engine backtracks, and some patterns take *exponential* time on input that almost matches — ReDoS (regular-expression denial of service). Since JDK 9 the engine remembers failed positions inside simple nested loops, so the textbook example `(a+)+$` is fast on Java 21; but that optimisation does not cover every pattern — patterns with backreferences, for example — and those still explode, adding a factor of about eight for every three extra characters below. | Unit 8.6 |
| DT-32 | In `DateTimeFormatter` patterns, `yyyy` is the *year of era* and `YYYY` is the *week-based year*: for 29–31 December in some years, `YYYY` prints the *next* year. `Y` in place of `y` (and `D` — day of year — in place of `d`) produces dates that are correct for most of the year and wrong for a few days, which is why the bug survives testing. | Unit 8.7 |
| DT-33 | When two dependencies need different versions of the same library, Maven picks the *nearest* declaration and Gradle picks the *highest* version — and either can break at run time. | Unit 9.1 |
| DT-34 | JUnit creates a new instance of the test class for every test method — so instance fields are never shared between tests, but `static` fields are. | Unit 9.4 |
| DT-35 | `volatile` guarantees visibility and ordering, not atomicity: `volatileCounter++` is still a lost-update race. | Unit 10.3 |
| DT-36 | Double-checked locking is correct only if the field is `volatile`. | Unit 10.3 |
| DT-37 | A thread-safe collection makes each *method* atomic, not sequences of methods: check-then-act on a `ConcurrentHashMap` is still a race. | Unit 10.5 |
| DT-38 | On Java 21–23, a virtual thread that blocks while inside a `synchronized` block *pins* its carrier thread. | Unit 10.7 |
| DT-39 | Java has no value types yet: every object has a header and identity, and a collection of objects is a collection of *pointers* to objects scattered across the heap. | Unit 11.1 |
| DT-40 | A "memory leak" in Java is an object that is still reachable but will never be used again. | Unit 11.2 |
| DT-41 | Micro-benchmarks written by hand measure the JIT's cleverness, not your code — use JMH. | Unit 11.3 |
| DT-42 | A network call without a timeout can block forever — and by default, Java's sockets and `HttpClient` requests have none. | Unit 12.2 |
| DT-43 | Building SQL by concatenating strings (`"... WHERE name = '" + name + "'"`) allows *SQL injection*: an input such as `' OR '1'='1` changes the query's meaning. `PreparedStatement` sends the SQL and the values separately, so a value can never become SQL — and it is also faster for repeated queries. | Unit 12.3 |
| DT-44 | A connection returned to a pool keeps its state: an unfinished transaction, a changed isolation level, or `autoCommit=false` leaks to the next user. | Unit 12.4 |
| DT-45 | Dependency injection is a pattern, not a framework: it means a class receives its collaborators instead of creating or looking them up. | Unit 13.1 |
| DT-46 | A cache is a second copy of the truth; decide what happens when the copies differ before adding one. | Unit 14.3 |
| DT-47 | A shutdown hook is the last code your service runs — keep it short, ordered, and bounded. | Unit 14.6 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Pin the Java version, compile with `--release`, and treat warnings as errors from the start. | Unit 0.6 |
| P-2 | Make classes immutable unless there is a good reason not to; make value types records; make everything else `private` by default. | Unit 2.6 |
| P-3 | Use sealed interfaces with records and pattern matching for data with a fixed set of shapes; use open class hierarchies with virtual methods for behaviour that new types will extend. | Unit 3.5 |
| P-4 | Use checked exceptions for recoverable conditions the caller is expected to handle; use unchecked exceptions for programming errors; translate at boundaries; never swallow. | Unit 4.4 |
| P-5 | In APIs, use wildcards in parameters (PECS) and never in return types; prefer `List<E>` over arrays of generics. | Unit 5.3 |
| P-6 | Declare variables and parameters by interface, return unmodifiable collections, and default to `ArrayList`, `HashMap`, `HashSet`, and `ArrayDeque`. | Unit 6.5 |
| P-7 | Use streams to say *what* is computed; use loops when the logic needs early exits with state, checked exceptions, or index arithmetic; make streams parallel only after measuring. | Unit 7.4 |
| P-8 | Never deserialize untrusted data with Java serialization; define explicit formats with versions and checksums. | Unit 8.5 |
| P-9 | Make code testable by injecting what it depends on — collaborators, time, randomness, configuration — and prefer fakes over mocks. | Unit 9.4 |
| P-10 | Share nothing if you can; share immutable data if you must share; guard mutable shared state with one clear lock — and prefer `java.util.concurrent` to `wait`/`notify`. | Unit 10.7 |
| P-11 | Diagnose before tuning: GC log first, then a heap histogram or dump for memory, and a JFR recording for CPU, allocation, and latency. | Unit 11.4 |
| P-12 | Hide JDBC behind a repository interface; use `PreparedStatement` always; one transaction per unit of work; optimistic locking for concurrent edits. | Unit 12.5 |
| P-13 | Keep business logic in plain classes with constructor-injected dependencies; let the framework stay at the edges. | Chapter 13 Project |

# Appendix B — Glossary

| Term | Meaning |
| --- | --- |
| Autoboxing | Automatic conversion between a primitive and its wrapper object (Unit 1.2) |
| Bridge method | A synthetic method that makes an erased generic override work (DT-19) |
| Bytecode | The JVM's instruction set, produced by `javac` (Unit 0.3) |
| Checked exception | An exception the compiler requires callers to handle or declare (Unit 4.1) |
| Class loader | The JVM component that finds and loads classes on demand (Unit 0.4) |
| Collector | A recipe for folding a stream into a result, e.g. `groupingBy` (Unit 7.3) |
| Compact constructor | A record constructor that validates or normalises its components (Unit 2.4) |
| Composition root | The one place where an application's object graph is assembled (Chapter 13) |
| Connection pool | A set of reusable database connections (DT-44, Unit 14.2) |
| Data race | Unsynchronised access to shared mutable data from several threads (Unit 10.3) |
| Deadlock | Threads waiting for each other's locks in a cycle (Unit 10.4) |
| Default method | An interface method with a body (Unit 3.3) |
| Dependency injection | Giving an object its collaborators instead of letting it create them (DT-45) |
| Erasure | Removal of generic type information at compile time (Unit 5.2) |
| Escape analysis | A JIT analysis that removes allocations of objects that never leave a method (Unit 11.3) |
| Executor | An object that runs submitted tasks, usually on a thread pool (Unit 10.6) |
| Functional interface | An interface with one abstract method; the target type of a lambda (Unit 7.1) |
| Garbage collection | Automatic reclamation of unreachable objects (Unit 11.2) |
| G1 | The default, region-based, pause-targeting garbage collector (Unit 11.2) |
| Happens-before | The Java Memory Model's guarantee that one action's effects are visible to another (Unit 10.3) |
| Heap pollution | A variable of a parameterised type referring to an object of the wrong type (DT-18) |
| HotSpot | OpenJDK's JVM implementation |
| Immutable object | An object whose state cannot change after construction (Unit 2.2) |
| `invokedynamic` | A bytecode whose target is linked at run time; used for lambdas and string concatenation (DT-2, DT-25) |
| JAR | A zip archive of class files and a manifest (Unit 0.6) |
| JDBC | Java's API for relational databases (Unit 12.3) |
| JFR | Java Flight Recorder, the JVM's built-in profiler (Unit 11.4) |
| JIT | The just-in-time compiler that turns hot bytecode into machine code (Unit 0.5) |
| JMM | The Java Memory Model (Unit 10.3) |
| Lambda | An anonymous function implementing a functional interface (Unit 7.1) |
| LTS | A long-term-support release: 8, 11, 17, 21, 25 (Appendix C) |
| Memory leak | Objects that stay reachable though they will never be used (DT-40) |
| Monitor | The intrinsic lock of every object, used by `synchronized` (Unit 10.2) |
| Optimistic locking | Detecting concurrent modification with a version check instead of holding a lock (Unit 12.5) |
| Optional | A container for a value that may be absent (Unit 4.6) |
| Pattern matching | Testing a value's shape and binding its parts: `instanceof`, `switch`, record patterns (Unit 3.5) |
| PECS | Producer `extends`, consumer `super`: the wildcard rule (Unit 5.3) |
| Raw type | A generic type used without type arguments; disables generic checking (Unit 5.2) |
| Record | A concise, immutable data class with value semantics (Unit 2.4) |
| Sealed type | A class or interface with a fixed list of permitted subtypes (Unit 3.4) |
| Stream | A lazy pipeline of operations over a sequence of elements (Unit 7.2) |
| Test double | A stand-in for a dependency in a test: stub, fake, or mock (Unit 9.4) |
| Transaction | A unit of database work that commits or rolls back as a whole (Unit 12.4) |
| try-with-resources | A statement that closes its resources automatically (Unit 4.3) |
| Type token | A `Class<T>` (or super type token) carrying a type to run time (Unit 5.4) |
| Unchecked exception | A `RuntimeException`; need not be declared (Unit 4.1) |
| Virtual thread | A lightweight thread scheduled by the JVM (Unit 10.7) |
| `volatile` | A field modifier guaranteeing visibility of writes across threads (Unit 10.3) |
| Wildcard | `?` in a generic type: an unknown type argument (Unit 5.3) |
| Young generation | The heap area where new objects are allocated and most die (Unit 11.2) |

# Appendix C — Java versions and features

A feature release ships every six months (March and September); every two years one is designated **long-term support** (LTS) by vendors. New language features usually appear first as **preview** features (enabled with `--enable-preview`) for one or more releases.

| Version | Year | Features used or discussed in this book |
| --- | --- | --- |
| 5 | 2004 | generics, annotations, enums, autoboxing, varargs, enhanced `for`, `java.util.concurrent`, the revised Java Memory Model |
| 7 | 2011 | try-with-resources, diamond `<>`, multi-catch, `java.nio.file` (NIO.2), strings in `switch`, `invokedynamic`, Fork/Join |
| 8 (LTS) | 2014 | lambdas, method references, streams, default methods, `Optional`, `java.time`, `CompletableFuture`, `HashMap` treeification |
| 9 | 2017 | modules (JPMS), `jshell`, `List.of`/`Map.of`, private interface methods, compact strings, `invokedynamic` string concatenation, G1 as default |
| 10 | 2018 | `var` for local variables |
| 11 (LTS) | 2018 | `java.net.http.HttpClient`, single-file source launch (`java Hello.java`), `String.strip`/`isBlank`/`repeat`/`lines`, `Files.readString`, ZGC (experimental), Flight Recorder open-sourced |
| 14 | 2020 | `switch` expressions, helpful `NullPointerException` messages |
| 15 | 2020 | text blocks, ZGC and Shenandoah production-ready |
| 16 | 2021 | records, pattern matching for `instanceof`, `Stream.toList()` |
| 17 (LTS) | 2021 | sealed classes and interfaces, strong encapsulation of JDK internals, `HexFormat`, enhanced pseudo-random generators |
| 18 | 2022 | UTF-8 as the default charset, simple web server (`jwebserver`) |
| 19–20 | 2022–23 | previews of virtual threads, record patterns, pattern `switch`, structured concurrency |
| **21 (LTS)** | 2023 | **the baseline of this book.** Virtual threads, pattern matching for `switch`, record patterns, sequenced collections, generational ZGC, key encapsulation API; previews: string templates (later withdrawn), unnamed patterns, structured concurrency, scoped values |
| 22 | 2024 | unnamed variables and patterns (`_`), Foreign Function & Memory API final, multi-file source launch; `stream gatherers` preview |
| 23 | 2024 | Markdown documentation comments, generational ZGC by default |
| 24 | 2025 | stream gatherers final, virtual threads no longer pin in `synchronized` (JEP 491), class-file API, ahead-of-time class loading and linking |
| 25 (LTS) | 2025 | scoped values final, compact source files and instance `main` methods, flexible constructor bodies, module import declarations, compact object headers, generational Shenandoah |

## Choosing a version

Use the newest LTS your organisation supports (21 or 25). Compile with `--release N` so that the same build works on any newer JDK (DT-1). Preview features are not for production: code compiled with `--enable-preview` runs only on that exact release.

## Tools in the JDK

| Tool | Purpose | Chapter |
| --- | --- | --- |
| `javac`, `java`, `jshell` | compile, run, experiment | 0 |
| `javap` | disassemble class files | 0 |
| `jar`, `jdeps`, `jlink`, `jpackage` | package, analyse dependencies, build custom runtimes and installers | 0 |
| `jcmd` | send diagnostic commands to a running JVM (histograms, thread dumps, JFR) | 10, 11 |
| `jfr` | print and summarise Flight Recorder files | 11 |
| `jstack`, `jmap`, `jstat` | thread dumps, heap dumps, GC statistics (older single-purpose tools) | 11 |
| `javadoc` | generate API documentation | — |
