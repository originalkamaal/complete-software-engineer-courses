---
title: "Python from Zero to Advanced"
subtitle: "How Python really works — from the interpreter and the data model to concurrency, performance, packaging, and a tested, typed, production-style application"
lang: en
---

# Preface

Python is easy to start and surprisingly deep. Many people write it for years on top of a mental model that is almost right: variables as boxes, `is` as a stricter `==`, threads as parallelism, type hints as checks, `eval` with empty builtins as a sandbox. Almost-right models produce bugs that are hard to see, because the code runs.

This book builds the accurate model from the ground up. It begins below the language — how CPython compiles source to bytecode, how names refer to objects, how memory is reclaimed — and climbs through the core language, the built-in data structures and their internals, functions and closures, classes and the data model, iterators and generators, errors and context managers, modules and packaging, type hints, files and formats, testing and profiling, concurrency, the standard library, performance with NumPy and compiled code, and metaprogramming. It ends with databases, web services, and a capstone: a small search engine with an asynchronous crawler, a SQLite index, a FastAPI service, and a command-line tool, checked by `mypy --strict` and a pytest suite.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth: the Python Language Reference, the library documentation, a PEP, CPython's implementation, or a library's documented behaviour.
2. **Evidence.** Every program shown with output was run with CPython 3.13, and the output is real — including tracebacks, type-checker errors, and timings. Several examples were wrong in their first draft; where that taught something (a file that shadowed a standard module, a connection used across threads by a web framework, an upsert that orphaned rows), the book shows what happened.
3. **Order.** Nothing is used before it is taught; later chapters refer back by unit number.

# How to use this book

## Who it is for

- **Beginners:** start at Chapter 0 and read in order. Type every program; run it; change it.
- **Programmers from other languages:** read Chapters 0–2 quickly but carefully — names and objects (Unit 0.3), integer division (DT-3), and containers (Chapter 2) differ from most languages — then continue in order.
- **Working Python developers:** skim the summaries and read every *Deep & Tricky* entry; Chapters 5, 8, 10, 11, 13, and 14 usually contain the most new material.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | Small complete programs and their real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced developers, labelled with their source of truth. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions; answers at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a realistic program with complete code, real output, and a review checklist.

## Running the code

The source archive has one directory per chapter. Programs run with `python3 file.py` from their chapter directory; the console blocks show the exact commands. You need Python 3.12 or later (3.13 was used). Third-party packages used: `pytest`, `hypothesis`, `mypy`, `numpy`, `cython`, `pydantic`, `fastapi`, `uvicorn`, and `httpx` — Chapters 7 and 16 show how to install them into a virtual environment with `uv` or `pip`. Chapter 13 also needs a C compiler.

# Chapter 0 — How Python Runs

*Goal: understand what happens between typing `python3 hello.py` and seeing output — the interpreter, compilation to bytecode, the evaluation loop, objects and references, and memory management — so that every later chapter rests on an accurate mental model rather than on folklore.*

**What you will be able to do after this chapter**

- Run Python code three ways (REPL, script, `-c`) and know which interpreter you are using.
- Disassemble a function and read its bytecode.
- Explain names, objects, references, `id`, `is`, and `==`.
- Explain reference counting, the cyclic garbage collector, and weak references.
- Explain `.pyc` files and what "compiled" means for Python.

## Topic 0A — The interpreter

## Unit 0.1 — Running Python

**One-line summary:** "Python" is a language; **CPython** is the reference implementation, a program written in C that reads your source code and executes it. You can run code interactively in the **REPL** (Read–Eval–Print Loop), as a **script** (`python3 file.py`), or as a one-liner (`python3 -c "..."`). Always know *which* interpreter and version you are running.

```python
# file: hello.py
import platform
import sys

print("Hello from Python", platform.python_version())
print("implementation:", platform.python_implementation())
print("executable:", sys.executable)
print("running as a script, __name__ =", __name__)
```

```console
$ python3 hello.py
Hello from Python 3.13.18
implementation: CPython
executable: /usr/local/bin/python3
running as a script, __name__ = __main__
$ python3 -c "print(2 ** 100)"
1267650600228229401496703205376
```

`2 ** 100` prints all 31 digits: Python integers have no fixed size (Chapter 1). Other implementations exist — **PyPy** (a JIT compiler, often much faster for pure-Python loops), **MicroPython** (microcontrollers), **GraalPy** — but this book uses CPython 3.13 and says so whenever a behaviour is CPython-specific.

## Unit 0.2 — Source → bytecode → evaluation loop

**One-line summary:** CPython does not execute your text directly. It **parses** the source into a syntax tree, **compiles** the tree into **bytecode** — a compact list of instructions for a virtual stack machine — and runs the bytecode in an **evaluation loop**. The `dis` module shows the bytecode.

```python
# file: bytecode.py
import ast
import dis

src = "total = price * qty + tax"
print(ast.dump(ast.parse(src).body[0], indent=1))     # 1) the syntax tree

def total(price, qty, tax):
    return price * qty + tax

dis.dis(total)                                        # 2) the bytecode
print("code object:", total.__code__.co_varnames, "stack size", total.__code__.co_stacksize)
```

```console
$ python3 bytecode.py
Assign(
 targets=[
  Name(id='total', ctx=Store())],
 value=BinOp(
  left=BinOp(
   left=Name(id='price', ctx=Load()),
   op=Mult(),
   right=Name(id='qty', ctx=Load())),
  op=Add(),
  right=Name(id='tax', ctx=Load())))
  8           RESUME                   0

  9           LOAD_FAST_LOAD_FAST      1 (price, qty)
              BINARY_OP                5 (*)
              LOAD_FAST                2 (tax)
              BINARY_OP                0 (+)
              RETURN_VALUE
code object: ('price', 'qty', 'tax') stack size 2
```

Read the bytecode like a recipe for a stack machine: `LOAD_FAST` pushes a local variable, `BINARY_OP` pops two values and pushes the result, `RETURN_VALUE` pops the result and returns it. Every Python operation — even `a + b` — becomes a few such instructions, each executed by a C function inside the interpreter. This is the main reason pure-Python loops are slower than C: the per-instruction overhead is paid on every step.

### Deep & Tricky

**DT-1. Bytecode is an implementation detail of CPython: it changes between versions (3.11 introduced specialising "adaptive" instructions, 3.12 and 3.13 changed many opcodes), it is not portable between versions, and other implementations do not use it at all.** *(CPython documentation, `dis` module: "Bytecode is an implementation detail of the CPython interpreter… no guarantees are made that bytecode will not be added, removed, or changed between versions")* Use `dis` to understand and compare, never as an interface. Since 3.11, CPython also rewrites instructions at run time into specialised forms (for example a `BINARY_OP` that has only seen integers); `dis.dis(f, adaptive=True)` shows them after the function has run a few times.

```python
# file: adaptive.py
import dis

def add(a, b):
    return a + b

for _ in range(1000):                                # warm up with integers
    add(1, 2)
dis.dis(add, adaptive=True)
```

```console
$ python3 adaptive.py
  4           RESUME_CHECK             0

  5           LOAD_FAST_LOAD_FAST      1 (a, b)
              BINARY_OP_ADD_INT        0 (+)
              RETURN_VALUE
```

## Topic 0B — Objects and names

## Unit 0.3 — Everything is an object; variables are names

**One-line summary:** Every value in Python — numbers, strings, functions, classes, modules — is an **object** with an identity (`id`), a type, and a value. A **variable** is a **name bound to an object**, not a box containing a value. Assignment `b = a` binds a second name to the **same** object. `is` compares identities; `==` compares values.

```python
# file: names.py
a = [1, 2, 3]
b = a                       # a second name for the same list
b.append(4)
print(a, a is b, id(a) == id(b))

c = [1, 2, 3, 4]
print(a == c, a is c)       # equal values, different objects

def show(x):
    print(f"{x!r:>12}  type={type(x).__name__:<8} id={id(x)}")
for v in (42, "hi", 3.5, None, len, show):
    show(v)

x = 10
y = x
x = x + 1                   # rebinding x does not affect y: ints are immutable
print(x, y)
```

```console
$ python3 names.py
[1, 2, 3, 4] True True
True False
          42  type=int      id=20101984
        'hi'  type=str      id=20144664
         3.5  type=float    id=140664472194384
        None  type=NoneType id=20086032
<built-in function len>  type=builtin_function_or_method id=140664473511520
<function show at 0x7feeffe271a0>  type=function id=140664471974304
11 10
```

The last three lines show the difference between **mutating** an object (`b.append` changes the one list both names see) and **rebinding** a name (`x = x + 1` makes `x` refer to a new int object). Whether an operation mutates or rebinds is the key to understanding most "surprising" Python behaviour.

### Deep & Tricky

**DT-2. `is` on numbers and strings is unreliable for value comparison: CPython caches small integers (−5 to 256) and interns some strings, so `a is b` can be `True` for equal small values and `False` for equal large ones — and the result can differ between the REPL, a script, and constant folding.** *(CPython implementation detail; `PyLong` small-int cache; Python Language Reference: "the `is` operator… compares object identity")* Use `==` for values and reserve `is` for singletons (`None`, `True`, `False`, sentinels). Since 3.8, `x is 5` even triggers a `SyntaxWarning`.

```python
# file: identity.py
import sys

a = int("256"); b = int("256")
c = int("257"); d = int("257")
print("256 is 256:", a is b, "| 257 is 257:", c is d)
s1 = "".join(["hello", "_", "world"]); s2 = "hello_world"
print("equal strings, same object?", s1 is s2, "| after sys.intern:", sys.intern(s1) is sys.intern(s2))
```

```console
$ python3 identity.py
256 is 256: True | 257 is 257: False
equal strings, same object? False | after sys.intern: True
```

Building the numbers with `int("...")` at run time avoids the compiler merging identical constants, which would otherwise hide the effect inside a single script.

## Topic 0C — Memory

## Unit 0.4 — Reference counting, the cycle collector, and weak references

**One-line summary:** Each CPython object has a **reference count**; when it drops to zero, the object is freed **immediately**. Reference counting cannot free **cycles** (a list containing itself), so a separate **cyclic garbage collector** (`gc` module) periodically finds unreachable cycles. A **weak reference** refers to an object without keeping it alive.

```python
# file: memory.py
import gc
import sys
import weakref

class Node:
    def __init__(self, name):
        self.name, self.other = name, None
    def __del__(self):
        print(f"  freeing {self.name}")

print("refcount of a fresh object:", sys.getrefcount(object()))   # only the temporary argument reference
x = Node("x")
print("refcount of x:", sys.getrefcount(x))
print("del x ->")
del x                                                 # count drops to 0: freed immediately

a, b = Node("a"), Node("b")
a.other, b.other = b, a                               # a cycle
print("del a, b (cycle) ->")
del a, b
print("  ...nothing freed yet; running gc.collect():")
print("  collected", gc.collect(), "objects")

target = Node("target")
w = weakref.ref(target)
print("weakref alive:", w() is target)
del target
print("after del, weakref returns:", w())
```

```console
$ python3 memory.py
refcount of a fresh object: 1
refcount of x: 2
del x ->
  freeing x
del a, b (cycle) ->
  ...nothing freed yet; running gc.collect():
  freeing a
  freeing b
  collected 9 objects
weakref alive: True
  freeing target
after del, weakref returns: None
```

`sys.getrefcount` reports one more than you might expect, because passing the object to the function creates a temporary reference.

### Pattern

**P-1. Think of Python variables as labels on objects: ask "does this operation mutate the object, or rebind the name?" and "who else holds a reference to this object?"** *(Practice)* Most aliasing bugs — a default list shared across calls (Chapter 3), a nested list built with `*` (Chapter 2), a function that modifies its argument — are answered by these two questions. *Trade-off:* reference semantics make passing large objects cheap (no copying), at the cost of having to copy explicitly (`list(a)`, `copy.deepcopy`) when independence is needed.

## Unit 0.5 — `.pyc` files and imports

**One-line summary:** When you import a module, CPython compiles it and caches the bytecode in `__pycache__/name.cpython-313.pyc`, keyed by the source's modification time (or hash). The next import skips parsing and compiling. The script you run directly is compiled every time and not cached. "Compiled" Python is still interpreted bytecode — it is not machine code.

```python
# file: mymodule.py
def greet(name):
    return f"hello, {name}"
```

```python
# file: use_module.py
import importlib.util
import os
import time

import mymodule
print(mymodule.greet("pyc"))
print("cached bytecode:", sorted(os.listdir("__pycache__")))
print("cache file for mymodule:", importlib.util.cache_from_source("mymodule.py"))
```

```console
$ rm -rf __pycache__ && python3 use_module.py
hello, pyc
cached bytecode: ['mymodule.cpython-313.pyc']
cache file for mymodule: __pycache__/mymodule.cpython-313.pyc
```

### Mastery check

1. **Predict.** After `a = [1]; b = a; a = a + [2]`, what is `b`? What if the second statement were `a += [2]`?
2. **Predict.** Does `del x` always free the object that `x` referred to?

## Chapter 0 Project — A bytecode explorer

**Goal:** write a tool that, for a few ways of computing the same thing, prints the number of bytecode instructions executed per loop iteration (static count inside the loop body) and the measured time — making visible why "do it in C" (built-ins) beats "do it in a Python loop".

```python
# file: explorer/explorer.py
import dis
import timeit

def loop_sum(n):
    total = 0
    for i in range(n):
        total += i
    return total

def builtin_sum(n):
    return sum(range(n))

def formula_sum(n):
    return n * (n - 1) // 2

def count_instructions(f):
    ins = list(dis.get_instructions(f))
    return len(ins), sorted({i.opname for i in ins})

N = 1_000_000
assert loop_sum(N) == builtin_sum(N) == formula_sum(N)
for f in (loop_sum, builtin_sum, formula_sum):
    count, ops = count_instructions(f)
    t = min(timeit.repeat(lambda: f(N), number=3, repeat=3)) / 3
    print(f"{f.__name__:>12}: {count:>2} bytecode instructions, {t * 1000:8.2f} ms for n = {N:,}")
print("\nselected instructions of loop_sum (those from FOR_ITER to JUMP_BACKWARD run once per iteration):")
for ins in dis.get_instructions(loop_sum):
    if ins.opname in ("FOR_ITER", "STORE_FAST", "LOAD_FAST", "LOAD_FAST_LOAD_FAST", "BINARY_OP", "JUMP_BACKWARD"):
        print(f"  {ins.opname:<22} {ins.argrepr}")
```

```console
$ cd explorer && python3 explorer.py
    loop_sum: 17 bytecode instructions,    21.44 ms for n = 1,000,000
 builtin_sum:  7 bytecode instructions,    11.01 ms for n = 1,000,000
 formula_sum:  8 bytecode instructions,     0.00 ms for n = 1,000,000

selected instructions of loop_sum (those from FOR_ITER to JUMP_BACKWARD run once per iteration):
  STORE_FAST             total
  LOAD_FAST              n
  FOR_ITER               to L2
  STORE_FAST             i
  LOAD_FAST_LOAD_FAST    total, i
  BINARY_OP              +=
  STORE_FAST             total
  JUMP_BACKWARD          to L1
  LOAD_FAST              total
```

`loop_sum` executes its handful of loop-body instructions a million times; `builtin_sum` runs a short bytecode sequence once and does the loop in C; `formula_sum` does no loop at all. This ladder — Python loop, C loop, no loop — reappears throughout the book as the main lever for performance (Chapter 13).

### Review checklist

- Which instructions in `loop_sum` run once per iteration?
- Why does `builtin_sum` still take time proportional to n?
- Would PyPy change the ranking? (It compiles the hot loop to machine code, narrowing the gap.)

## Answers to the Chapter 0 prediction questions

**0.5 Q1.** `a = a + [2]` builds a **new** list and rebinds `a`, so `b` is still `[1]`. With `a += [2]`, lists implement `+=` as in-place extension (`__iadd__`), so the shared object is mutated and `b` becomes `[1, 2]`.

**0.5 Q2.** No. `del x` removes the **name** and decrements the reference count; the object is freed only if no other reference remains (and, for cycles, only when the cycle collector runs).

# Chapter 1 — The Core Language

*Goal: write correct small programs with Python's built-in types and control flow — integers, floats, booleans, strings, `None`; operators and their precedence; `if`, `while`, `for`, `match`; and simple functions — knowing exactly where Python's numbers and comparisons differ from other languages.*

**What you will be able to do after this chapter**

- Use integers (unbounded), floats (IEEE 754 doubles), and their division and rounding rules.
- Format output with f-strings, and handle text as Unicode.
- Write conditions using truthiness, chained comparisons, and short-circuit operators.
- Control flow with `if`/`elif`, `while`, `for` over iterables, `break`/`continue`, and the loop `else` clause.
- Dispatch on the shape of data with `match` (structural pattern matching).
- Define and call functions with positional, keyword, and default arguments.

## Topic 1A — Values

## Unit 1.1 — Numbers

**One-line summary:** `int` has **arbitrary precision** (it grows as needed); `float` is a 64-bit IEEE 754 double (about 15–17 significant digits); `/` always produces a float, `//` is **floor** division, `%` has the sign of the divisor, and `round` rounds halves to **even**. Use `fractions.Fraction` or `decimal.Decimal` when exactness matters.

```python
# file: number_demo.py
from decimal import Decimal
from fractions import Fraction

print(2 ** 200)                                   # no overflow
print(7 / 2, 7 // 2, -7 // 2, 7 % 3, -7 % 3)      # floor division rounds toward minus infinity
print(divmod(-7, 3), (-7 // 3) * 3 + (-7 % 3))    # invariant: (a // b) * b + a % b == a
print(0.1 + 0.2, 0.1 + 0.2 == 0.3)
print(round(0.5), round(1.5), round(2.5), round(2.675, 2))   # banker's rounding; 2.675 is really 2.67499...
print(Fraction(1, 10) + Fraction(2, 10), Decimal("0.1") + Decimal("0.2"))
print(int("ff", 16), hex(255), bin(5), 1_000_000, 1e3, int(3.99), int(-3.99))
import math
print(math.isclose(0.1 + 0.2, 0.3), math.inf > 10**1000, math.nan == math.nan)
```

```console
$ python3 number_demo.py
1606938044258990275541962092341162602522202993782792835301376
3.5 3 -4 1 2
(-3, 2) -7
0.30000000000000004 False
0 2 2 2.67
3/10 0.3
255 0xff 0b101 1000000 1000.0 3 -3
True True False
```

### Deep & Tricky

**DT-3. Python's `//` and `%` follow floor semantics (−7 // 2 == −4, −7 % 3 == 2), unlike C, Java, Go, and Rust, which truncate toward zero (−7 / 2 == −3, −7 % 3 == −1); and `int(x)` truncates toward zero, so `int(-3.99) == -3` while `-7 // 2 == -4`.** *(Python Language Reference, "Binary arithmetic operations": "the result is that of mathematical division with the floor function applied")* The payoff is the invariant `(a // b) * b + a % b == a` with `0 <= a % b < b` for positive b — ideal for indexing circular buffers (`i % n` is never negative). The trap is porting code between languages, or mixing `int()` and `//` and expecting the same rounding.

## Unit 1.2 — Strings and formatting

**One-line summary:** `str` is an immutable sequence of Unicode **code points**. Index and slice with `s[i]`, `s[a:b:step]`; common methods: `split`, `join`, `strip`, `replace`, `find`, `startswith`, `upper`. **f-strings** (`f"{expr:format}"`) embed expressions with format specifications. `len` counts code points, not bytes or visible characters.

```python
# file: strings.py
name, price, qty = "Widget", 3.5, 12
print(f"{name:<10}|{price:>8.2f}|{qty:04d}|{price * qty:,.1f}")
print(f"{qty = }, {price * qty = }")              # debugging form: shows the expression too
s = "  Hello, World  "
print(repr(s.strip()), s.strip().lower().split(", "), "-".join(["a", "b", "c"]))
t = "Python"
print(t[0], t[-1], t[1:4], t[::-1], t[::2], "th" in t)
word = "naïve café 🐍"
print(len(word), len(word.encode("utf-8")), [hex(ord(c)) for c in "é🐍"])
try:
    t[0] = "J"
except TypeError as e:
    print("strings are immutable:", e)
```

```console
$ python3 strings.py
Widget    |    3.50|0012|42.0
qty = 12, price * qty = 42.0
'Hello, World' ['hello', 'world'] a-b-c
P n yth nohtyP Pto True
12 17 ['0xe9', '0x1f40d']
strings are immutable: 'str' object does not support item assignment
```

<!-- expanded -->
### Going deeper: f-string format specifications, and string methods that matter

The part after `:` in an f-string replacement field is a small language: `{x:>10}` right-aligns in 10 columns, `{x:^10}` centres, `{x:08.3f}` zero-pads a float to width 8 with 3 decimals, `{n:,}` adds thousands separators, `{n:_x}` prints hex with `_` grouping, `{p:.1%}` prints a percentage, and `{x!r}` uses `repr` instead of `str`. Since Python 3.8, `{x=}` prints the expression and its value — the fastest debugging print there is.

```python
# file: fstrings.py
items = [("apples", 3, 0.5), ("kiwis", 12, 0.25), ("watermelon", 1, 3.0)]
print(f"{'item':<12}|{'qty':>5}|{'cost':>8}")
for name, qty, price in items:
    print(f"{name:<12}|{qty:>5}|{qty * price:>8.2f}")

n, ratio, name = 1234567, 0.4567, "Ada"
print(f"{n:,} {n:_x} {ratio:.1%} {3.14159:08.3f} {name!r} {name:*^9}")
width = 6
print(f"[{42:>{width}}] [{42:<{width}}]")      # width from a variable
total = sum(q * p for _, q, p in items)
print(f"{total=}, {len(items)=}, {total / len(items)=:.3f}")

s = "  Hello, World  "
print(repr(s.strip()), s.split(","), "-".join(["a", "b", "c"]))
print("hello world".title(), "Straße".casefold(), "abc".startswith(("x", "a")))
print("report.final.pdf".rsplit(".", 1), "a=b=c".partition("="), "tab\tsep".expandtabs(4))
print("café".encode("utf-8"), "naïve".isascii(), "١٢٣".isdigit(), int("١٢٣"))
```

```console
$ python3 fstrings.py
item        |  qty|    cost
apples      |    3|    1.50
kiwis       |   12|    3.00
watermelon  |    1|    3.00
1,234,567 12_d687 45.7% 0003.142 'Ada' ***Ada***
[    42] [42    ]
total=7.5, len(items)=3, total / len(items)=2.500
'Hello, World' ['  Hello', ' World  '] a-b-c
Hello World strasse True
['report.final', 'pdf'] ('a', '=', 'b=c') tab sep
b'caf\xc3\xa9' False True 123
```

Two surprises in the last line: `str.isdigit()` is true for digits in any script (here Arabic-Indic), and `int()` accepts them — so `"١٢٣".isdigit()` passing a validation check does not mean the user typed ASCII digits. Use `s.isascii() and s.isdigit()` when you need `0`–`9` only.

## Unit 1.3 — Truthiness, comparisons, and logic

**One-line summary:** In a condition, `False`, `None`, `0`, `0.0`, empty strings and empty containers are **falsy**; everything else is truthy. Comparisons **chain** (`0 < x < 10` means `0 < x and x < 10`). `and`/`or` **short-circuit** and return one of their **operands**, not necessarily a bool.

```python
# file: logic.py
for v in (0, 1, -1, "", "0", [], [0], None, 0.0, {}):
    print(f"{v!r:>5} -> {bool(v)}")
x = 5
print(0 < x < 10, 1 < x == 5 > 3)                 # chained: (1 < x) and (x == 5) and (5 > 3)
print("" or "default", 0 or None, 3 and "yes", [] and "never evaluated")
def expensive():
    print("  (expensive called)")
    return True
print("short-circuit:", False and expensive(), True or expensive())
print(None is None, [] == [], [] is [])
```

```console
$ python3 logic.py
    0 -> False
    1 -> True
   -1 -> True
   '' -> False
  '0' -> True
   [] -> False
  [0] -> True
 None -> False
  0.0 -> False
   {} -> False
True True
default None yes []
short-circuit: False True
True True False
```

<!-- expanded -->
### Going deeper: chained comparisons, `and`/`or` return values, and `is` versus `==`

`a < b < c` means `a < b and b < c` (with `b` evaluated once). `and` and `or` do not return `True`/`False` — they return one of their operands, which is why `name = given or "anonymous"` works as a default. `is` compares identity and should be used only with singletons (`None`, `True`, `False`, sentinel objects); for values use `==`.

```python
# file: logic2.py
x = 5
print(1 < x < 10, 1 < x > 3, (1 < x) < 10)        # the last is (True) < 10, i.e. 1 < 10
print(0 or "default", "" or [] or "last", "a" and "b", None and 1)
given = ""
name = given or "anonymous"                         # careful: "" and 0 and [] are all replaced
count = 0
print(name, count or 10, count if count is not None else 10)

a = 1000
b = int("1000")
print(a == b, a is b)                               # equal values, different objects
print([] == [], [] is [])
SENTINEL = object()
def get(d, key, default=SENTINEL):
    v = d.get(key, SENTINEL)
    if v is SENTINEL:                                # identity check against a unique object
        if default is SENTINEL:
            raise KeyError(key)
        return default
    return v
print(get({"k": None}, "k"), get({}, "k", 7))
print(float("nan") == float("nan"), [float("nan")] == [float("nan")])
nan = float("nan")
print([nan] == [nan])                               # containers check identity first
```

```console
$ python3 logic2.py
True True True
default last b None
anonymous 10 0
True False
True False
None 7
False False
True
```

The last three lines show a corner of the language: `nan != nan`, yet a list containing the *same* `nan` object equals itself, because list comparison tries `is` before `==` for each element.

## Topic 1B — Control flow and functions

## Unit 1.4 — `if`, `while`, `for`, and loop `else`

**One-line summary:** `for` iterates over any **iterable** (lists, strings, ranges, files, dict keys) — there is no C-style index loop; use `range(n)` for indices and `enumerate` when you need both. `while` repeats while a condition holds. `break` leaves a loop, `continue` skips to the next iteration, and a loop's `else` block runs only if the loop ended **without** `break`.

```python
# file: loops.py
def find_divisor(n):
    for d in range(2, int(n ** 0.5) + 1):
        if n % d == 0:
            print(f"{n} = {d} * {n // d}")
            break
    else:                                          # no break happened: no divisor found
        print(f"{n} is prime")

for n in (91, 97):
    find_divisor(n)

fruits = ["apple", "banana", "cherry"]
for i, f in enumerate(fruits, start=1):
    print(i, f)
for f, price in zip(fruits, [1.2, 0.5, 3.0]):
    print(f"{f}: {price}")

n, steps = 27, 0                                   # Collatz: a while loop with an unknown count
while n != 1:
    n = 3 * n + 1 if n % 2 else n // 2
    steps += 1
print("Collatz steps from 27:", steps)
```

```console
$ python3 loops.py
91 = 7 * 13
97 is prime
1 apple
2 banana
3 cherry
apple: 1.2
banana: 0.5
cherry: 3.0
Collatz steps from 27: 111
```

<!-- expanded -->
### Going deeper: `enumerate`, `zip`, `range`, and the walrus operator

Index-based loops (`for i in range(len(xs))`) are rarely needed. `enumerate` gives index and item together, `zip` walks several sequences in step (with `strict=True` since 3.10 to catch length mismatches), and `reversed` and `sorted` return new iteration orders. The **walrus operator** `:=` (3.8) assigns inside an expression, which tidies "read, test, use" loops.

```python
# file: loops2.py
import io

names = ["ada", "alan", "grace"]
scores = [91, 78, 88]
for i, (n, s) in enumerate(zip(names, scores), start=1):
    print(f"{i}. {n:6} {s}")
try:
    list(zip(names, [1, 2], strict=True))
except ValueError as e:
    print("zip strict:", e)
print(list(range(10, 0, -3)), list(reversed(names)), range(0, 10**12, 7)[-1])

stream = io.StringIO("alpha\nbeta\n\ngamma\n")
while (line := stream.readline()):                  # assign and test in one expression
    print(repr(line.rstrip("\n")))

data = [3, 8, 1, 9, 4]
if (big := [x for x in data if x > 5]):
    print("big values:", big)

for n in range(2, 20):
    for d in range(2, int(n ** 0.5) + 1):
        if n % d == 0:
            break
    else:                                           # runs only when the inner loop did not break
        print(n, end=" ")
print()
```

```console
$ python3 loops2.py
1. ada    91
2. alan   78
3. grace  88
zip strict: zip() argument 2 is shorter than argument 1
[10, 7, 4, 1] ['grace', 'alan', 'ada'] 999999999999
'alpha'
'beta'
''
'gamma'
big values: [8, 9]
2 3 5 7 11 13 17 19 
```

`range` objects are lazy and support indexing and `len` in constant time — `range(0, 10**12, 7)[-1]` computes the last element without building a trillion-element list.

## Unit 1.5 — Structural pattern matching

**One-line summary:** `match` compares a value against **patterns** that can test literals, bind names, destructure sequences and mappings, check types (`int()`), and add guards (`if ...`). Cases are tried in order; `case _` matches anything. It is not a C `switch`: a bare name in a pattern **binds**, it does not compare.

```python
# file: matching.py
def describe(cmd):
    match cmd:
        case ["go", direction]:
            return f"move {direction}"
        case ["take", *items] if items:
            return f"pick up {', '.join(items)}"
        case {"action": "say", "text": str(text)}:
            return f"say {text!r}"
        case int(n) | float(n) if n < 0:
            return f"negative number {n}"
        case "quit" | "exit":
            return "goodbye"
        case _:
            return f"unknown: {cmd!r}"

for c in (["go", "north"], ["take", "key", "lamp"], ["take"], {"action": "say", "text": "hi"}, -3, 2.5, "quit"):
    print(f"{c!r:>35} -> {describe(c)}")

RED = "red"
def wrong(colour):
    match colour:
        case RED:                                  # binds a NEW name RED: matches everything!
            return "red?"
print(wrong("blue"))
```

```console
$ python3 matching.py
                    ['go', 'north'] -> move north
            ['take', 'key', 'lamp'] -> pick up key, lamp
                           ['take'] -> unknown: ['take']
    {'action': 'say', 'text': 'hi'} -> say 'hi'
                                 -3 -> negative number -3
                                2.5 -> unknown: 2.5
                             'quit' -> goodbye
red?
```

The last example compiles only because `case RED:` is the final case; Python rejects a capture pattern that would make later cases unreachable. To compare against a constant, use a dotted name (`Colour.RED`) or a literal.

<!-- expanded -->
### Going deeper: class patterns, guards, and capture versus constant

`match` patterns can destructure objects by attribute (`Point(x=0, y=y)`), nest sequences and mappings, bind sub-patterns with `as`, and add `if` guards. One rule catches everyone once: a bare name in a pattern is a *capture* that matches anything, so `case RED:` with a variable `RED` captures instead of comparing. Constants must be dotted names (`Color.RED`) or literals.

```python
# file: matching2.py
from dataclasses import dataclass
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2

@dataclass
class Point:
    x: float
    y: float

def describe(obj):
    match obj:
        case Point(x=0, y=0):
            return "origin"
        case Point(x=0, y=y) | Point(x=y, y=0):
            return f"on an axis at {y}"
        case Point(x=x, y=y) if x == y:
            return f"on the diagonal at {x}"
        case {"type": "circle", "r": float(r) | int(r)} if r > 0:
            return f"circle of radius {r}"
        case [Point() as first, *rest]:
            return f"path starting at {first} with {len(rest)} more"
        case Color.RED:
            return "the colour red"
        case str() as s if s.isdigit():
            return f"digit string {s}"
        case _:
            return f"unknown: {obj!r}"

for o in [Point(0, 0), Point(0, 5), Point(3, 3), {"type": "circle", "r": 2}, [Point(1, 2), Point(3, 4)],
          Color.RED, "42", 3.5]:
    print(describe(o))

RED = "red"
def bad(c):
    match c:
        case RED:            # a capture pattern: matches anything and rebinds RED
            return "matched"
print(bad("blue"))
```

```console
$ python3 matching2.py 2>&1
origin
on an axis at 5
on the diagonal at 3
circle of radius 2
path starting at Point(x=1, y=2) with 1 more
the colour red
digit string 42
unknown: 3.5
matched
```

Python accepts `bad`: the capture pattern matches anything and rebinds `RED`, so `bad("blue")` returns `"matched"`. Had another `case` followed it, compilation would fail with "name capture 'RED' makes remaining patterns unreachable". The fix is a dotted name (`case Color.RED:`) or a literal (`case "red":`).

## Unit 1.6 — Functions

**One-line summary:** `def` creates a function object. Parameters can be passed **positionally** or **by keyword**; parameters with defaults are optional; `*args` collects extra positional arguments into a tuple and `**kwargs` extra keyword arguments into a dict; `/` and `*` in the parameter list mark positional-only and keyword-only parameters. A function without `return` returns `None`.

```python
# file: functions.py
def area(width, height=1.0, *, unit="m"):         # unit is keyword-only
    return f"{width * height:g} {unit}^2"

print(area(3), area(3, 4), area(height=2, width=5), area(2, 2, unit="cm"))
try:
    area(2, 2, "cm")
except TypeError as e:
    print("TypeError:", e)

def stats(*nums, **options):
    precision = options.get("precision", 2)
    mean = sum(nums) / len(nums)
    return round(mean, precision)

print(stats(1, 2, 4), stats(1, 2, 4, precision=4))
values = [3, 5, 8]
print(stats(*values))                              # unpack a list into positional arguments

def no_return():
    pass
print(no_return())
```

```console
$ python3 functions.py
3 m^2 12 m^2 10 m^2 4 cm^2
TypeError: area() takes from 1 to 2 positional arguments but 3 were given
2.33 2.3333
5.33
None
```

### Deep & Tricky

**DT-4. Floats cannot represent most decimal fractions exactly: 0.1 is stored as 0.1000000000000000055511151231257827…, so `0.1 + 0.2 != 0.3`, `round(2.675, 2) == 2.67`, and repeated addition accumulates error; compare with `math.isclose`, sum with `math.fsum` (or `sum`, which uses compensated summation for floats since Python 3.12), and use `Decimal` for money.** *(IEEE 754; Python Tutorial, "Floating-Point Arithmetic: Issues and Limitations")* Python prints the *shortest* decimal string that round-trips to the same float (`repr(0.1) == '0.1'`), which hides the representation error until arithmetic exposes it. `Decimal('0.1')` is exact; `Decimal(0.1)` converts the binary float and shows its true value.

```python
# file: floats.py
import math
from decimal import Decimal
print(Decimal(0.1))
print(Decimal("0.1"))
total = 0.0
for _ in range(10):
    total += 0.1                                  # naive accumulation: error builds up
print(total, total == 1.0)
print(sum([0.1] * 10), math.fsum([0.1] * 10))      # sum() of floats is compensated since 3.12
```

```console
$ python3 floats.py
0.1000000000000000055511151231257827021181583404541015625
0.1
0.9999999999999999 False
1.0 1.0
```

**DT-5. The directory of the script you run is put first on `sys.path`, so a file of yours named like a standard-library module (`numbers.py`, `random.py`, `email.py`, `queue.py`) shadows it — not only for your imports but for every library module that imports it.** *(Python documentation, `sys.path`: "the first item… is the directory containing the script")* While writing this chapter, the first draft of Unit 1.1's example was called `numbers.py`; `from decimal import Decimal` then failed with a circular-import error, because `decimal` (in its pure-Python fallback) imports the standard `numbers` module and got the draft file instead. The demonstration below reproduces it with `fractions`, which also imports `numbers`:

```python
# file: shadow/numbers.py
print("this is MY numbers.py, not the standard library's")
```

```python
# file: shadow/use.py
from fractions import Fraction
print(Fraction(1, 3))
```

```console
$ cd shadow && python3 use.py
this is MY numbers.py, not the standard library's
Traceback (most recent call last):
  File "shadow/use.py", line 2, in <module>
    from fractions import Fraction
  File "/usr/local/lib/python3.13/fractions.py", line 6, in <module>
    from decimal import Decimal
  File "/usr/local/lib/python3.13/decimal.py", line 102, in <module>
    from _decimal import *
AttributeError: module 'numbers' has no attribute 'Number' (consider renaming 'shadow/numbers.py' since it has the same name as the standard library module named 'numbers' and prevents importing that standard library module)
```

Python 3.13 even recognises the situation and suggests renaming the file. Name your files after what they do in *your* project (`price_math.py`), and when an import error mentions a "partially initialized module", look for a file of yours with a standard name.

### Pattern

**P-2. Write conditions and loops the Pythonic way: iterate directly over items (`for x in items`), use `enumerate`/`zip` instead of index arithmetic, rely on truthiness for emptiness checks (`if not items:`), and use `==` for values and `is` only for `None` and other singletons.** *(Practice; PEP 8)* These idioms are shorter and avoid off-by-one errors. *Trade-off:* truthiness can hide intent when 0 or an empty string is a *valid* value — write `if x is None:` when that is what you mean.

### Mastery check

1. **Predict.** What do `-17 // 5`, `-17 % 5`, and `int(-17 / 5)` print?
2. **Predict.** What does `[] or {} or 0 or "end"` evaluate to?

## Chapter 1 Project — A calculator with pattern matching

**Goal:** a small command-line calculator that reads lines from standard input, tokenises them, and evaluates arithmetic with `+ − * / // % **`, parentheses, unary minus, variables (`x = 3 * 4`), and the commands `vars` and `quit` — using a recursive-descent parser and `match` for dispatch. Errors (division by zero, unknown variable, bad syntax) are reported without crashing.

```python
# file: calc/calc.py
import re
import sys

TOKEN = re.compile(r"\s*(?:(\d+\.\d*|\d+)|(\*\*|//|[-+*/%()=])|([A-Za-z_]\w*))")

def tokenize(text):
    pos, out = 0, []
    text = text.rstrip()
    while pos < len(text):
        m = TOKEN.match(text, pos)
        if not m:
            raise SyntaxError(f"unexpected character {text[pos]!r} at {pos}")
        num, op, name = m.groups()
        out.append(("num", float(num) if "." in num else int(num)) if num else ("op", op) if op else ("name", name))
        pos = m.end()
    return out

class Parser:
    def __init__(self, tokens, env):
        self.toks, self.i, self.env = tokens, 0, env
    def peek(self):
        return self.toks[self.i] if self.i < len(self.toks) else ("end", None)
    def take(self, kind=None, value=None):
        tok = self.peek()
        if (kind and tok[0] != kind) or (value and tok[1] != value):
            raise SyntaxError(f"expected {value or kind}, got {tok[1]!r}")
        self.i += 1
        return tok
    def expr(self):                                 # expr := term (('+'|'-') term)*
        v = self.term()
        while self.peek() in (("op", "+"), ("op", "-")):
            op = self.take()[1]
            v = v + self.term() if op == "+" else v - self.term()
        return v
    def term(self):                                 # term := unary (('*'|'/'|'//'|'%') unary)*
        v = self.unary()
        while self.peek()[0] == "op" and self.peek()[1] in ("*", "/", "//", "%"):
            op, rhs = self.take()[1], None
            rhs = self.unary()
            match op:
                case "*": v *= rhs
                case "/": v /= rhs
                case "//": v //= rhs
                case "%": v %= rhs
        return v
    def unary(self):                                # unary := '-' unary | power
        if self.peek() == ("op", "-"):
            self.take()
            return -self.unary()
        return self.power()
    def power(self):                                # power := atom ('**' unary)?   (right-associative)
        base = self.atom()
        if self.peek() == ("op", "**"):
            self.take()
            return base ** self.unary()
        return base
    def atom(self):
        match self.peek():
            case ("num", value):
                self.take(); return value
            case ("name", name):
                self.take()
                if name not in self.env:
                    raise NameError(f"unknown variable {name!r}")
                return self.env[name]
            case ("op", "("):
                self.take(); v = self.expr(); self.take("op", ")"); return v
            case ("end", _):
                raise SyntaxError("unexpected end of input")
            case tok:
                raise SyntaxError(f"unexpected {tok[1]!r}")

def run_line(line, env):
    tokens = tokenize(line)
    match tokens:
        case []:
            return None
        case [("name", "quit")]:
            raise SystemExit
        case [("name", "vars")]:
            return ", ".join(f"{k}={v}" for k, v in sorted(env.items())) or "(none)"
        case [("name", target), ("op", "="), *rest]:
            p = Parser(rest, env); value = p.expr()
            if p.i != len(rest):
                raise SyntaxError("trailing input")
            env[target] = value
            return f"{target} = {value}"
        case _:
            p = Parser(tokens, env); value = p.expr()
            if p.i != len(tokens):
                raise SyntaxError("trailing input")
            return value

def main():
    env = {}
    for line in sys.stdin:
        try:
            out = run_line(line, env)
        except SystemExit:
            break
        except (SyntaxError, NameError, ZeroDivisionError) as e:
            out = f"error: {type(e).__name__}: {e}"
        if out is not None:
            print(f"> {line.strip():<22} {out}")

if __name__ == "__main__":
    main()
```

```console
$ cd calc && printf '%s\n' '2 + 3 * 4' '(2 + 3) * 4' '-2 ** 2' '2 ** 3 ** 2' '7 // 2 + 7 % 3' 'x = 10 / 4' 'y = x * 2' 'x + y' 'vars' '1 / 0' 'z + 1' '3 +' 'quit' '99' | python3 calc.py
> 2 + 3 * 4              14
> (2 + 3) * 4            20
> -2 ** 2                -4
> 2 ** 3 ** 2            512
> 7 // 2 + 7 % 3         4
> x = 10 / 4             x = 2.5
> y = x * 2              y = 5.0
> x + y                  7.5
> vars                   x=2.5, y=5.0
> 1 / 0                  error: ZeroDivisionError: division by zero
> z + 1                  error: NameError: unknown variable 'z'
> 3 +                    error: SyntaxError: unexpected end of input
```

The grammar encodes precedence by **levels** (expression → term → unary → power → atom): each level handles one precedence class and calls the next, which is why `2 + 3 * 4` is 14 and `2 ** 3 ** 2` is 512 (power is right-associative). `-2 ** 2` gives −4, as in Python itself, because unary minus binds less tightly than `**`.

### Review checklist

- Why does `power` call `unary` (not `atom`) for its right operand?
- Which input shows that errors do not stop the loop, and which shows that `quit` does?
- How would you add functions like `sqrt(x)` to the grammar?

## Answers to the Chapter 1 prediction questions

**1.6 Q1.** `-17 // 5` is −4 (floor of −3.4), `-17 % 5` is 3 (since −4·5 + 3 = −17), and `int(-17 / 5)` is −3 (truncation of −3.4).

**1.6 Q2.** `"end"`: `or` returns the first truthy operand; `[]`, `{}`, and `0` are all falsy.

# Chapter 2 — Data Structures: list, tuple, dict, set — and Their Internals

*Goal: use Python's four core containers fluently and choose between them by their costs, which follow directly from how CPython implements them — lists as over-allocated arrays of pointers, dicts and sets as open-addressing hash tables.*

**What you will be able to do after this chapter**

- Use lists, tuples, dicts, and sets with their main operations and idioms.
- Predict the cost of each operation (O(1), O(n)) from the implementation.
- Avoid aliasing bugs: nested lists, shallow versus deep copies.
- Explain hashability and why dict keys and set elements must be immutable.
- Sort with keys, and pick the right container for a task.

## Topic 2A — Sequences

## Unit 2.1 — Lists: dynamic arrays of references

**One-line summary:** A `list` is a resizable array of **pointers** to objects. Indexing and `append` are O(1) (append is **amortised** O(1): the array over-allocates and occasionally grows); inserting or deleting at the front or middle is O(n) because later elements shift; `x in list` is O(n). Slicing creates a new list (a shallow copy).

```python
# file: lists.py
import sys

a = []
last = sys.getsizeof(a)
print(f"len  0: {last} bytes")
for i in range(1, 33):
    a.append(i)
    size = sys.getsizeof(a)
    if size != last:                                  # print only when the array was reallocated
        print(f"len {i:>2}: {size} bytes  (capacity ~{(size - sys.getsizeof([])) // 8} slots)")
        last = size

b = [5, 3, 8, 1]
b.append(9); b.insert(0, 0); b.extend([7, 7]); b.remove(7)   # remove deletes the FIRST 7 only
print(b)
last, first = b.pop(), b.pop(0)
print(last, first, b, b.index(8), b.count(7))
print(sorted(b), sorted(b, reverse=True), b[::-1], b[1:3])
c = b[:]                                              # a copy: c and b are different lists
c.append(100)
print(b, c)
```

```console
$ python3 lists.py
len  0: 56 bytes
len  1: 88 bytes  (capacity ~4 slots)
len  5: 120 bytes  (capacity ~8 slots)
len  9: 184 bytes  (capacity ~16 slots)
len 17: 248 bytes  (capacity ~24 slots)
len 25: 312 bytes  (capacity ~32 slots)
[0, 5, 3, 8, 1, 9, 7]
7 0 [5, 3, 8, 1, 9] 2 0
[1, 3, 5, 8, 9] [9, 8, 5, 3, 1] [9, 1, 8, 3, 5] [3, 8]
[5, 3, 8, 1, 9] [5, 3, 8, 1, 9, 100]
```

The capacities jump in steps (4, 8, 16, 24, 32, …): CPython grows the array by about 1/8 plus a small constant, so the total copying over n appends is O(n): each append costs O(1) **on average** (amortised), even though an individual append that triggers a reallocation copies the whole array.

### Deep & Tricky

**DT-6. `[[0] * 3] * 3` creates **three references to one inner list**, so assigning `grid[0][0] = 1` changes all three rows; `[x] * n` copies references, never objects.** *(Python Language Reference, "Common Sequence Operations", note 2: "items in the sequence s are not copied; they are referenced multiple times")* The fix is a comprehension that evaluates the inner expression once per row: `[[0] * 3 for _ in range(3)]`. The outer `* 3` is fine for immutable elements (numbers, strings), which is why `[0] * 3` behaves as expected.

```python
# file: aliasing.py
grid = [[0] * 3] * 3
grid[0][0] = 1
print(grid, [id(row) for row in grid])
grid = [[0] * 3 for _ in range(3)]
grid[0][0] = 1
print(grid, len({id(row) for row in grid}), "distinct rows")
```

```console
$ python3 aliasing.py
[[1, 0, 0], [1, 0, 0], [1, 0, 0]] [139964559742784, 139964559742784, 139964559742784]
[[1, 0, 0], [0, 0, 0], [0, 0, 0]] 3 distinct rows
```

## Unit 2.2 — Tuples, packing, and unpacking

**One-line summary:** A `tuple` is an immutable sequence — use it for fixed records (a point, a database row) and as dict keys. Commas make tuples; **unpacking** assigns elements to names (`x, y = point`), with `*rest` collecting the remainder. `collections.namedtuple` and `typing.NamedTuple` add field names.

```python
# file: tuples.py
from typing import NamedTuple

point = 3, 4                                         # parentheses are optional: the comma makes the tuple
x, y = point
first, *middle, last = [1, 2, 3, 4, 5]
print(point, x, y, first, middle, last)
a, b = 1, 2
a, b = b, a                                          # swap via packing and unpacking
print(a, b, (1,), type((1)), type((1,)))

class City(NamedTuple):
    name: str
    lat: float
    lon: float
paris = City("Paris", 48.86, 2.35)
print(paris, paris.name, paris[1], paris._replace(name="Lutetia"))
locations = {(48.86, 2.35): "Paris"}                 # tuples can be dict keys
print(locations[(48.86, 2.35)])
t = ([1, 2], 3)
t[0].append(99)                                      # the tuple is immutable; the list inside is not
print(t)
```

```console
$ python3 tuples.py
(3, 4) 3 4 1 [2, 3, 4] 5
2 1 (1,) <class 'int'> <class 'tuple'>
City(name='Paris', lat=48.86, lon=2.35) Paris 48.86 City(name='Lutetia', lat=48.86, lon=2.35)
Paris
([1, 2, 99], 3)
```

<!-- expanded -->
### Going deeper: star-unpacking, named tuples, and tuples as keys

Unpacking works on any iterable and supports one starred target that collects "the rest". Tuples are hashable when their elements are, which makes them natural dictionary keys for coordinates and composite keys. `collections.namedtuple` and `typing.NamedTuple` add field names without losing tuple behaviour.

```python
# file: tuples2.py
from typing import NamedTuple

first, *middle, last = [1, 2, 3, 4, 5]
print(first, middle, last)
head, *tail = "abc"
print(head, tail)
(a, b), c = (1, 2), 3
print(a, b, c)
a, b = b, a                                     # swap without a temporary
print(a, b)

grid = {(0, 0): "start", (2, 3): "treasure"}
print(grid[(2, 3)], grid.get((9, 9), "empty"))

class Point(NamedTuple):
    x: float
    y: float = 0.0
    def dist(self) -> float:
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p, p.x, p[1], p.dist(), p._replace(y=0), Point(5))
x, y = p                                        # still a tuple
print(x + y, p == (3, 4))
t = (1, [2, 3])
t[1].append(4)                                  # the tuple is immutable; the list inside is not
print(t)
try:
    hash(t)
except TypeError as e:
    print("unhashable:", e)
```

```console
$ python3 tuples2.py
1 [2, 3, 4] 5
a ['b', 'c']
1 2 3
2 1
treasure empty
Point(x=3, y=4) 3 4 5.0 Point(x=3, y=0) Point(x=5, y=0.0)
7 True
(1, [2, 3, 4])
unhashable: unhashable type: 'list'
```

`p == (3, 4)` is `True`: a named tuple is still a tuple and compares equal to a plain one with the same values — convenient, and occasionally a source of bugs when two different named tuple types compare equal.

## Topic 2B — Hash tables

## Unit 2.3 — Dicts

**One-line summary:** A `dict` maps hashable keys to values with O(1) average lookup, insertion, and deletion, using an **open-addressing hash table**; since Python 3.7 it preserves **insertion order** (a language guarantee). Useful methods: `get`, `setdefault`, `pop`, `items`, `update`, `|` (merge, 3.9+); `collections.defaultdict` and `Counter` cover common patterns.

```python
# file: dicts.py
from collections import Counter, defaultdict

stock = {"apple": 3, "pear": 0}
stock["kiwi"] = 7
print(stock, stock.get("plum", 0), "pear" in stock, list(stock))
for fruit, qty in stock.items():
    print(f"  {fruit}: {qty}")
print({k: v for k, v in stock.items() if v})          # dict comprehension
print(stock | {"apple": 10, "fig": 1})                # merge: right side wins
words = "the cat and the hat and the bat".split()
by_letter = defaultdict(list)
for w in words:
    by_letter[w[0]].append(w)
print(dict(by_letter))
print(Counter(words).most_common(2))
try:
    {["a", "list"]: 1}
except TypeError as e:
    print("TypeError:", e)
```

```console
$ python3 dicts.py
{'apple': 3, 'pear': 0, 'kiwi': 7} 0 True ['apple', 'pear', 'kiwi']
  apple: 3
  pear: 0
  kiwi: 7
{'apple': 3, 'kiwi': 7}
{'apple': 10, 'pear': 0, 'kiwi': 7, 'fig': 1}
{'t': ['the', 'the', 'the'], 'c': ['cat'], 'a': ['and', 'and'], 'h': ['hat'], 'b': ['bat']}
[('the', 3), ('and', 2)]
TypeError: unhashable type: 'list'
```

### Deep & Tricky

**DT-7. A dict key's hash must never change while it is in the dict, which is why mutable built-ins (list, dict, set) are unhashable; a user-defined class whose `__hash__` depends on mutable fields can be inserted and then become unfindable.** *(Python Data Model, `object.__hash__`: "the only required property is that objects which compare equal have the same hash value"; Python Glossary, "hashable")* The table stores each key at a slot derived from its hash; if the hash changes after insertion, lookups probe the wrong slot. CPython also caches string hashes and randomises them per process (`PYTHONHASHSEED`), so iteration order of a *set* of strings can differ between runs — dicts are ordered by insertion, sets are not.

```python
# file: bad_hash.py
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)
    def __hash__(self):
        return hash((self.x, self.y))                 # depends on mutable fields: dangerous

p = Point(1, 2)
d = {p: "here"}
p.x = 100                                             # mutate the key after insertion
print("p in d:", p in d, "| Point(1, 2) in d:", Point(1, 2) in d, "| len(d):", len(d))
print("but iterating finds it:", [(k.x, k.y) for k in d])
```

```console
$ python3 bad_hash.py
p in d: False | Point(1, 2) in d: False | len(d): 1
but iterating finds it: [(100, 2)]
```

The key is still in the table, but neither the mutated object nor an equal fresh object can find it: one hashes to a different slot, the other finds the slot but fails the equality check.

## Unit 2.4 — Sets

**One-line summary:** A `set` is an unordered collection of unique hashable elements with O(1) average membership tests, built on the same hashing as dicts. Set algebra: `|` union, `&` intersection, `-` difference, `^` symmetric difference, `<=` subset. `frozenset` is the immutable, hashable variant.

```python
# file: sets.py
import time

a, b = {1, 2, 3, 4}, {3, 4, 5}
print(a | b, a & b, a - b, a ^ b, {3, 4} <= a)
emails = ["x@a.com", "y@b.com", "x@a.com"]
print(len(set(emails)), "unique;", list(dict.fromkeys(emails)), "(dedupe keeping order)")
print({frozenset({1, 2}): "a frozenset can be a key"})

n = 50_000
as_list, as_set = list(range(n)), set(range(n))
probes = range(n - 1000, n)                           # values near the end: worst case for the list
for name, container in (("list", as_list), ("set", as_set)):
    t = time.perf_counter()
    hits = sum(1 for p in probes if p in container)
    print(f"1,000 membership tests in a {name} of {n:,}: {(time.perf_counter() - t) * 1000:8.2f} ms ({hits} hits)")
```

```console
$ python3 sets.py
{1, 2, 3, 4, 5} {3, 4} {1, 2} {1, 2, 5} True
2 unique; ['x@a.com', 'y@b.com'] (dedupe keeping order)
{frozenset({1, 2}): 'a frozenset can be a key'}
1,000 membership tests in a list of 50,000:   215.33 ms (1000 hits)
1,000 membership tests in a set of 50,000:     0.05 ms (1000 hits)
```

<!-- expanded -->
### Going deeper: set algebra, frozensets, and when sets beat lists

Sets support the mathematical operations as operators: `|` union, `&` intersection, `-` difference, `^` symmetric difference, `<=` subset. Membership tests are O(1) on average, against O(n) for lists — the most common easy speed-up in Python code. `frozenset` is the immutable, hashable version, usable as a dictionary key or a member of another set.

```python
# file: sets2.py
import timeit

python = {"ada", "alan", "grace", "linus"}
rust = {"grace", "linus", "ken"}
print("both:", sorted(python & rust), "either:", sorted(python | rust))
print("only python:", sorted(python - rust), "exactly one:", sorted(python ^ rust))
print({"grace"} <= python, python.isdisjoint({"x"}))

seen, dupes = set(), []
for w in "the cat and the hat and the bat".split():
    if w in seen:
        dupes.append(w)
    seen.add(w)
print("duplicates in order:", dupes, "| first occurrences:", list(dict.fromkeys("the cat and the hat".split())))

pairs = {frozenset({"a", "b"}): 1}
print(pairs[frozenset({"b", "a"})])            # order does not matter

big_list = list(range(100_000))
big_set = set(big_list)
t_list = timeit.timeit(lambda: 99_999 in big_list, number=200)
t_set = timeit.timeit(lambda: 99_999 in big_set, number=200)
print("set membership faster by more than 100x:", t_list > 100 * t_set)
```

```console
$ python3 sets2.py
both: ['grace', 'linus'] either: ['ada', 'alan', 'grace', 'ken', 'linus']
only python: ['ada', 'alan'] exactly one: ['ada', 'alan', 'ken']
True True
duplicates in order: ['the', 'and', 'the'] | first occurrences: ['the', 'cat', 'and', 'hat']
1
set membership faster by more than 100x: True
```

`dict.fromkeys(items)` is the idiomatic way to remove duplicates *while keeping order*; `set(items)` loses the order.

## Topic 2C — Copying and ordering

## Unit 2.5 — Shallow and deep copies; sorting with keys

**One-line summary:** A **shallow copy** (`list(a)`, `a[:]`, `dict(d)`, `copy.copy`) creates a new outer container holding the **same** inner objects; a **deep copy** (`copy.deepcopy`) recursively copies everything. `sorted(iterable, key=...)` returns a new sorted list; `list.sort` sorts in place; both are **stable**, so sorting by a secondary key and then by the primary key gives a multi-key order (or use a tuple key).

```python
# file: copying.py
import copy
from operator import itemgetter

orig = {"name": "team", "members": ["ana", "ben"]}
shallow, deep = copy.copy(orig), copy.deepcopy(orig)
orig["members"].append("cy")
print("shallow sees the change:", shallow["members"], "| deep does not:", deep["members"])

people = [("ana", 31, "Pune"), ("ben", 25, "Delhi"), ("cy", 31, "Agra"), ("dev", 25, "Pune")]
print(sorted(people, key=itemgetter(1)))                      # stable: ties keep original order
print(sorted(people, key=lambda p: (-p[1], p[0])))            # age descending, then name
print(max(people, key=lambda p: p[1]), min(people, key=len))
by_city = sorted(sorted(people, key=itemgetter(0)), key=itemgetter(2))   # two stable passes
print([p[0] for p in by_city])
```

```console
$ python3 copying.py
shallow sees the change: ['ana', 'ben', 'cy'] | deep does not: ['ana', 'ben']
[('ben', 25, 'Delhi'), ('dev', 25, 'Pune'), ('ana', 31, 'Pune'), ('cy', 31, 'Agra')]
[('ana', 31, 'Pune'), ('cy', 31, 'Agra'), ('ben', 25, 'Delhi'), ('dev', 25, 'Pune')]
('ana', 31, 'Pune') ('ana', 31, 'Pune')
['cy', 'ben', 'ana', 'dev']
```

### Pattern

**P-3. Choose the container by its operations: list for ordered sequences with index access and appends; tuple for fixed records and keys; dict for lookup by key; set for membership and de-duplication; deque for both ends (Chapter 12).** *(Practice)* Replace `if x in some_list` inside a loop with a set built once; replace parallel lists with a list of tuples or dataclasses (Chapter 4). *Trade-off:* hash tables use several times more memory than lists (empty slots, stored hashes) and do not keep elements sorted — when you need order by value, sort, or use `bisect` (Chapter 12).

<!-- expanded -->
### Deep & Tricky

**DT-8. `sort()` and `sorted()` are *stable*: items with equal keys keep their original order. That makes multi-key sorting possible by sorting several times, *least important key first* — or, more simply, by sorting once with a tuple key. And `key=` functions are called once per element, while a `cmp`-style comparison would be called O(n log n) times.** *(Guarantee: Python documentation, "Sorting Techniques" — sorts are guaranteed stable; the algorithm is Timsort, Powersort since 3.11)*

```python
# file: stable.py
from operator import attrgetter, itemgetter

staff = [("ravi", "eng", 90), ("ada", "eng", 120), ("lin", "ops", 90), ("bo", "eng", 90)]
by_two_passes = sorted(sorted(staff, key=itemgetter(0)), key=itemgetter(1))   # dept, then name
by_tuple = sorted(staff, key=lambda e: (e[1], -e[2], e[0]))                     # dept, salary desc, name
print(by_two_passes)
print(by_tuple)

calls = 0
def key(s):
    global calls
    calls += 1
    return s.lower()
words = ["b", "A", "c", "B", "a"] * 200
sorted(words, key=key)
print("key called", calls, "times for", len(words), "items")
print(sorted(["b", "A", "c", "B", "a"]), sorted(["b", "A", "c", "B", "a"], key=str.lower))
```

```console
$ python3 stable.py
[('ada', 'eng', 120), ('bo', 'eng', 90), ('ravi', 'eng', 90), ('lin', 'ops', 90)]
[('ada', 'eng', 120), ('bo', 'eng', 90), ('ravi', 'eng', 90), ('lin', 'ops', 90)]
key called 1000 times for 1000 items
['A', 'B', 'a', 'b', 'c'] ['A', 'a', 'b', 'B', 'c']
```

Note `sorted(["b","A",...])` puts uppercase first, because it compares code points (`"A"` is 65, `"a"` is 97). Human-friendly ordering needs a key such as `str.casefold` — and for real names in other languages, `locale.strxfrm` or the PyICU library.

### Mastery check

1. **Predict.** After `a = [1, [2, 3]]; b = a[:]; b[1].append(4); b[0] = 9`, what are `a` and `b`?
2. **Predict.** What does `dict.fromkeys("hello")` return, and in what order?

## Chapter 2 Project — A text index

**Goal:** build an index over the text of this book's own Chapter 0 (its Markdown source): word frequencies, the words appearing only once, an **anagram** index (words grouped by their sorted letters, a canonical key), the **inverted index** "word → line numbers", and a query "which lines contain all of these words?" answered by set intersection. Each structure is the right container for its job.

```python
# file: textindex/textindex.py
import re
from collections import Counter, defaultdict
from pathlib import Path

text = Path(__file__).resolve().parents[3].joinpath("src", "ch00.md").read_text()
lines = text.splitlines()
WORD = re.compile(r"[a-z]+")

freq = Counter()
where = defaultdict(set)                                   # word -> set of line numbers
for no, line in enumerate(lines, 1):
    for w in WORD.findall(line.lower()):
        freq[w] += 1
        where[w].add(no)

print(f"{len(lines)} lines, {sum(freq.values()):,} words, {len(freq):,} distinct")
print("most common:", freq.most_common(8))
hapax = sorted(w for w, c in freq.items() if c == 1 and len(w) > 9)
print("long words used once:", hapax[:8])

anagrams = defaultdict(set)
for w in freq:
    anagrams["".join(sorted(w))].add(w)
groups = sorted((sorted(g) for g in anagrams.values() if len(g) > 1), key=len, reverse=True)
print("anagram groups:", groups[:6])

def lines_with_all(*words):
    sets = [where.get(w, set()) for w in words]
    return sorted(set.intersection(*sets)) if sets else []

for query in (("reference", "count"), ("bytecode", "loop"), ("object", "identity", "type")):
    hits = lines_with_all(*query)
    print(f"lines containing {' + '.join(query)}: {hits[:10]}")
```

```console
$ cd textindex && python3 textindex.py
336 lines, 2,004 words, 533 distinct
most common: [('a', 87), ('the', 63), ('python', 42), ('is', 40), ('and', 38), ('for', 30), ('print', 29), ('x', 27)]
long words used once: ['assignment', 'comparison', 'compilation', 'decrements', 'difference', 'disassemble', 'documentation', 'everything']
anagram groups: [['how', 'who'], ['os', 'so'], ['no', 'on'], ['means', 'names']]
lines containing reference + count: [179, 336]
lines containing bytecode + loop: [3, 42, 44, 271, 308, 324]
lines containing object + identity + type: [114]
```

Each container matches its job: a `Counter` for frequencies, a `defaultdict(set)` for "word → lines" (sets because a line counts once and intersections are needed), a dict keyed by a **canonical form** (sorted letters) for anagrams, and a list for the lines themselves.

### Review checklist

- Why are line numbers stored in sets rather than lists?
- What is the canonical key for anagram grouping, and why does it work?
- How would you make the query rank lines by how many query words they contain?

## Answers to the Chapter 2 prediction questions

**2.5 Q1.** `a` is `[1, [2, 3, 4]]` and `b` is `[9, [2, 3, 4]]`: the slice is a shallow copy, so the inner list is shared, while rebinding `b[0]` affects only `b`.

**2.5 Q2.** `{'h': None, 'e': None, 'l': None, 'o': None}` — duplicates collapse, and dicts keep first-insertion order.

# Chapter 3 — Functions, Closures, and Decorators

*Goal: treat functions as values — pass them, return them, store them — understand exactly how Python looks up names (LEGB), how closures capture variables, and how decorators wrap functions; and avoid the two classic traps: mutable default arguments and late-binding closures.*

**What you will be able to do after this chapter**

- Pass functions as arguments, return them, and use `lambda` where it helps.
- Explain name resolution (Local, Enclosing, Global, Built-in) and use `global` and `nonlocal`.
- Write closures, and explain late binding in loops.
- Avoid mutable default arguments.
- Write decorators, with and without arguments, preserving metadata with `functools.wraps`.
- Use `functools.cache`, `partial`, and `singledispatch`.

## Topic 3A — Functions as values

## Unit 3.1 — First-class functions

**One-line summary:** A function is an object like any other: it can be assigned to a name, stored in a list or dict, passed to another function (a **higher-order** function such as `sorted(key=...)`, `map`, `filter`), and returned. `lambda args: expr` creates a small anonymous function — useful for short keys and callbacks, never for anything that needs a name or statements.

```python
# file: first_class.py
def shout(s):
    return s.upper() + "!"

def whisper(s):
    return s.lower() + "..."

speak = shout                                           # just another name for the same function
print(speak("hi"), speak is shout, shout.__name__)
for style in (shout, whisper):
    print(style("Hello"))

ops = {"+": lambda a, b: a + b, "-": lambda a, b: a - b, "max": max}   # dispatch table
print([ops[k](7, 3) for k in ("+", "-", "max")])

def apply_twice(f, x):
    return f(f(x))
print(apply_twice(lambda n: n * 3, 2), apply_twice(shout, "go"))
words = ["banana", "Apple", "cherry", "date"]
print(sorted(words), sorted(words, key=str.lower), sorted(words, key=len))
print(list(map(len, words)), [w for w in words if len(w) > 4])   # comprehension preferred over filter
```

```console
$ python3 first_class.py
HI! True shout
HELLO!
hello...
[10, 4, 7]
18 GO!!
['Apple', 'banana', 'cherry', 'date'] ['Apple', 'banana', 'cherry', 'date'] ['date', 'Apple', 'banana', 'cherry']
[6, 5, 6, 4] ['banana', 'Apple', 'cherry']
```

<!-- expanded -->
### Going deeper: `*args`, `**kwargs`, keyword-only and positional-only parameters

A function signature can say exactly how each parameter may be passed. Parameters after `*` (or `*args`) are **keyword-only** — the caller must name them, which makes calls such as `connect(host, timeout=5)` self-explaining. Parameters before `/` (3.8) are **positional-only** — the name is not part of the API and can be changed later.

```python
# file: params.py
def connect(host, port=5432, /, *, timeout=10.0, retries=3):
    return f"{host}:{port} timeout={timeout} retries={retries}"

print(connect("db.local"))
print(connect("db.local", 6543, timeout=2))
for bad in ['connect(host="db")', 'connect("db", 1, 2)']:
    try:
        eval(bad)
    except TypeError as e:
        print(f"{bad}: {e}")

def log(msg, *args, level="INFO", **fields):
    extra = " ".join(f"{k}={v!r}" for k, v in fields.items())
    print(f"[{level}] {msg % args if args else msg} {extra}".rstrip())

log("started")
log("user %s logged in from %s", "ada", "10.0.0.7", level="DEBUG", session=42)
opts = {"level": "WARN", "disk": "/var"}
log("low disk", **opts)                                 # unpack a dict into keyword arguments
nums = [3, 9, 4]
print(max(*nums), max(nums), f"{len(nums)=}")
```

```console
$ python3 params.py
db.local:5432 timeout=10.0 retries=3
db.local:6543 timeout=2 retries=3
connect(host="db"): connect() got some positional-only arguments passed as keyword arguments: 'host'
connect("db", 1, 2): connect() takes from 1 to 2 positional arguments but 3 were given
[INFO] started
[DEBUG] user ada logged in from 10.0.0.7 session=42
[WARN] low disk disk='/var'
9 9 len(nums)=3
```

`print`, `sorted`, and most of the standard library use keyword-only parameters (`sorted(xs, key=..., reverse=True)`); `len` and `abs` are positional-only. Keyword-only flags are the cure for the "boolean trap": `send(msg, True, False)` says nothing; `send(msg, urgent=True, retry=False)` says everything.

## Unit 3.2 — Scope: LEGB, `global`, `nonlocal`

**One-line summary:** Python resolves a name by searching **L**ocal (the current function), **E**nclosing (outer functions), **G**lobal (the module), then **B**uilt-in scopes. Whether a name is local is decided at **compile time**: any assignment to it inside a function makes it local for the **whole** function body. `global x` and `nonlocal x` declare that assignments refer to the module-level or enclosing variable.

```python
# file: scope.py
x = "global"

def outer():
    x = "enclosing"
    def inner():
        print("inner sees:", x)                     # E: found in outer
    inner()

outer()

count = 0
def broken():
    try:
        count += 1                                  # assignment makes count local -> read before assignment
    except UnboundLocalError as e:
        print("UnboundLocalError:", e)
broken()

def fixed():
    global count
    count += 1
fixed(); fixed()
print("count =", count)

def counter():
    n = 0
    def step():
        nonlocal n
        n += 1
        return n
    return step
c = counter()
print(c(), c(), c())
print(len("built-in len"), "<- len came from the built-in scope")
```

```console
$ python3 scope.py
inner sees: enclosing
UnboundLocalError: cannot access local variable 'count' where it is not associated with a value
count = 2
1 2 3
12 <- len came from the built-in scope
```

<!-- expanded -->
### Going deeper: `UnboundLocalError`

Python decides at compile time whether a name in a function is local: if the function *assigns* to it anywhere, it is local everywhere in that function. Reading it before the assignment then fails — even if a global of the same name exists.

```python
# file: unbound.py
counter = 0

def broken():
    print("counter is", counter)       # counter is local here, because of the line below
    counter = 1

def fixed():
    global counter
    counter += 1
    return counter

try:
    broken()
except UnboundLocalError as e:
    print("UnboundLocalError:", e)
print("fixed:", fixed(), fixed())

def make_counter():
    count = 0
    def inc():
        nonlocal count                  # rebinding an enclosing function's variable
        count += 1
        return count
    return inc
c = make_counter()
print(c(), c(), c())
print(broken.__code__.co_varnames, fixed.__code__.co_varnames)
```

```console
$ python3 unbound.py
UnboundLocalError: cannot access local variable 'counter' where it is not associated with a value
fixed: 1 2
1 2 3
('counter',) ()
```

`co_varnames` shows the compiler's decision: `broken` has a local `counter`, `fixed` has none.

## Topic 3B — Closures

## Unit 3.3 — Closures and late binding

**One-line summary:** A **closure** is an inner function that remembers variables of the enclosing function after it has returned — it captures the **variable** (a cell), not its current value. So functions created in a loop all see the loop variable's **final** value when called later (**late binding**). Bind the value at creation time with a default argument or `functools.partial`.

```python
# file: closures.py
from functools import partial

def make_multiplier(k):
    def multiply(x):
        return x * k                                # k lives on in the closure's cell
    return multiply
double, triple = make_multiplier(2), make_multiplier(3)
print(double(5), triple(5), double.__closure__[0].cell_contents)

late = [lambda: i for i in range(3)]
print("late binding:", [f() for f in late])         # all see the final i
early = [lambda i=i: i for i in range(3)]           # default evaluated at definition time
print("default-arg binding:", [f() for f in early])
with_partial = [partial(lambda i: i, i) for i in range(3)]
print("partial binding:", [f() for f in with_partial])
```

```console
$ python3 closures.py
10 15 2
late binding: [2, 2, 2]
default-arg binding: [0, 1, 2]
partial binding: [0, 1, 2]
```

### Deep & Tricky

**DT-9. Closures capture variables, not values: all lambdas created in `[lambda: i for i in range(3)]` share the comprehension's single variable `i`, and when called afterwards they all return its last value, 2.** *(Python Language Reference, "Resolution of names": free variables are looked up when the function is called; CPython implements them as shared cell objects)* The same happens with callbacks registered in a loop (GUI buttons, tasks scheduled with `asyncio`, threads). The bug is invisible until the functions are called later; tests that call each function immediately inside the loop pass.

## Unit 3.4 — Default arguments are evaluated once

**One-line summary:** Default values are evaluated **once, when the `def` statement runs**, and stored on the function. A mutable default (`def f(x, acc=[])`) is therefore **shared** across all calls that use the default. Use `None` as the default and create the object inside the function.

```python
# file: defaults.py
import time

def add_item_bad(item, basket=[]):
    basket.append(item)
    return basket

print(add_item_bad("apple"), add_item_bad("pear"))      # the same list, growing across calls
print("stored default:", add_item_bad.__defaults__)

def add_item(item, basket=None):
    if basket is None:
        basket = []                                       # a fresh list per call
    basket.append(item)
    return basket

print(add_item("apple"), add_item("pear"))

def stamp(when=time.time()):                              # evaluated once at definition!
    return when
a = stamp(); time.sleep(0.01); b = stamp()
print("same timestamp both times:", a == b)
```

```console
$ python3 defaults.py
['apple', 'pear'] ['apple', 'pear']
stored default: (['apple', 'pear'],)
['apple'] ['pear']
same timestamp both times: True
```

### Deep & Tricky

**DT-10. A mutable default argument is created once at function definition and shared by every call that omits the argument — the list (or dict) accumulates state across calls; likewise `def f(t=time.time())` freezes a single timestamp.** *(Python Language Reference, "Function definitions": "Default parameter values are evaluated from left to right when the function definition is executed… the same 'pre-computed' value is used for each call")* Linters flag it (Ruff `B006`, Pylint `dangerous-default-value`). The occasional deliberate use — a cheap per-function cache — is better written with `functools.cache` (Unit 3.6).

## Topic 3C — Decorators

## Unit 3.5 — Writing decorators

**One-line summary:** A **decorator** is a function that takes a function and returns a replacement, usually a wrapper that adds behaviour before/after the call. `@deco` above `def f` means `f = deco(f)`. Use `functools.wraps` so the wrapper keeps the original name, docstring, and signature. A decorator **with arguments** is a function that returns a decorator (three levels).

```python
# file: decorators.py
import functools
import time

def timed(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            print(f"  {func.__name__} took {(time.perf_counter() - start) * 1000:.1f} ms")
    return wrapper

def repeat(times):                                  # decorator factory: repeat(3) returns the decorator
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            return [func(*args, **kwargs) for _ in range(times)]
        return wrapper
    return decorator

@timed
def slow_square(x):
    """Square x, slowly."""
    time.sleep(0.02)
    return x * x

@repeat(3)
def greet(name):
    return f"hi {name}"

print(slow_square(7))
print(greet("ana"))
print("metadata kept:", slow_square.__name__, "|", slow_square.__doc__, "| original:", slow_square.__wrapped__)

def no_wraps(func):
    def wrapper(*args):
        return func(*args)
    return wrapper
@no_wraps
def documented():
    """I have a docstring."""
print("without functools.wraps:", documented.__name__, documented.__doc__)
```

```console
$ python3 decorators.py
  slow_square took 20.1 ms
49
['hi ana', 'hi ana', 'hi ana']
metadata kept: slow_square | Square x, slowly. | original: <function slow_square at 0x7f5f14ca1940>
without functools.wraps: wrapper None
```

<!-- expanded -->
### Going deeper: decorators with arguments, and `functools.wraps`

A decorator is a function that takes a function and returns a replacement. A decorator *with arguments* (`@retry(times=3)`) is a function that returns a decorator — three levels of nesting. `functools.wraps` copies the original's name, docstring, and signature information onto the wrapper; without it, every decorated function shows up as `wrapper` in tracebacks, logs, and documentation.

```python
# file: decorators2.py
import functools
import time

def retry(times=3, exceptions=(Exception,), delay=0.01):
    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except exceptions as e:
                    print(f"  {fn.__name__} attempt {attempt} failed: {e}")
                    if attempt == times:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

calls = {"n": 0}

@retry(times=3, exceptions=(ConnectionError,))
def flaky_fetch(url):
    """Fetch a URL (fails twice, then works)."""
    calls["n"] += 1
    if calls["n"] < 3:
        raise ConnectionError("timeout")
    return f"content of {url}"

print(flaky_fetch("https://example.org"))
print(flaky_fetch.__name__, "|", flaky_fetch.__doc__, "|", flaky_fetch.__wrapped__.__name__)

def no_wraps(fn):
    def wrapper(*a, **k):
        return fn(*a, **k)
    return wrapper

@no_wraps
def important():
    """Docs that will be lost."""
print(important.__name__, important.__doc__)
```

```console
$ python3 decorators2.py
  flaky_fetch attempt 1 failed: timeout
  flaky_fetch attempt 2 failed: timeout
content of https://example.org
flaky_fetch | Fetch a URL (fails twice, then works). | flaky_fetch
wrapper None
```

`__wrapped__` (set by `wraps`) gives access to the undecorated function — useful in tests that want to call the logic without the retry delays.

## Unit 3.6 — `functools`: cache, partial, singledispatch

**One-line summary:** `functools.cache` (and `lru_cache(maxsize=…)`) memoises a function on its (hashable) arguments; `partial` fixes some arguments of a function; `singledispatch` chooses an implementation by the **type of the first argument** — a lightweight alternative to `if isinstance` chains.

```python
# file: tools.py
import functools
from fractions import Fraction

@functools.cache
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
print(fib(200), fib.cache_info())

int_from_hex = functools.partial(int, base=16)
print(int_from_hex("ff"), int_from_hex("10"))

@functools.singledispatch
def describe(x):
    return f"something: {x!r}"
@describe.register
def _(x: int):
    return f"an int with {x.bit_length()} bits"
@describe.register
def _(x: list):
    return f"a list of {len(x)} items"
@describe.register(Fraction)
def _(x):
    return f"a fraction {x.numerator}/{x.denominator}"
for v in (255, [1, 2], Fraction(3, 4), "text", True):
    print(describe(v))
```

```console
$ python3 tools.py
280571172992510140037611932413038677189525 CacheInfo(hits=198, misses=201, maxsize=None, currsize=201)
255 16
an int with 8 bits
a list of 2 items
a fraction 3/4
something: 'text'
an int with 1 bits
```

`describe(True)` uses the `int` implementation because `bool` is a subclass of `int` — dispatch follows the type's inheritance chain (method resolution order, Chapter 4).

### Pattern

**P-4. Put cross-cutting behaviour (timing, logging, retries, caching, access checks) in decorators, always with `functools.wraps`; keep the decorated function focused on its job.** *(Practice)* Prefer the standard library's decorators (`functools.cache`, `contextlib.contextmanager`, `dataclasses.dataclass`) before writing your own. *Trade-off:* decorators hide control flow — a reader of `@retry` must know it can call the function several times; keep them few, well named, and documented, and make the wrapped original reachable via `__wrapped__` for tests.

### Mastery check

1. **Predict.** What does `[f(10) for f in [lambda x: x * i for i in range(3)]]` return?
2. **Predict.** If a decorator forgets `return wrapper`, what does the decorated name refer to?

## Chapter 3 Project — A small decorator toolkit

**Goal:** implement and test three production-style decorators: `@retry(times, exceptions, backoff)` that re-calls a failing function with exponential backoff; `@memoize(maxsize)` with an LRU policy and hit/miss statistics (built on an `OrderedDict`, then compared with `functools.lru_cache`); and `@validate` that checks arguments against the function's **type annotations** at call time.

```python
# file: toolkit/toolkit.py
import functools
import inspect
import random
import time
from collections import OrderedDict

def retry(times=3, exceptions=(Exception,), backoff=0.01):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            delay = backoff
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise                                    # give up: re-raise the last error
                    print(f"  attempt {attempt} failed ({e}); retrying in {delay * 1000:.0f} ms")
                    time.sleep(delay)
                    delay *= 2
        return wrapper
    return decorator

def memoize(maxsize=128):
    def decorator(func):
        cache, stats = OrderedDict(), {"hits": 0, "misses": 0}
        @functools.wraps(func)
        def wrapper(*args):
            if args in cache:
                stats["hits"] += 1
                cache.move_to_end(args)                         # most recently used
                return cache[args]
            stats["misses"] += 1
            result = cache[args] = func(*args)
            if len(cache) > maxsize:
                cache.popitem(last=False)                       # evict least recently used
            return result
        wrapper.stats = stats
        return wrapper
    return decorator

def validate(func):
    sig = inspect.signature(func)
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        for name, value in bound.arguments.items():
            expected = sig.parameters[name].annotation
            if expected is not inspect.Parameter.empty and not isinstance(value, expected):
                raise TypeError(f"{func.__name__}(): {name} must be {expected.__name__}, got {type(value).__name__}")
        return func(*args, **kwargs)
    return wrapper

rng = random.Random(3)
@retry(times=4, exceptions=(ConnectionError,))
def flaky_fetch(url):
    if rng.random() < 0.6:
        raise ConnectionError("network blip")
    return f"<html from {url}>"
print(flaky_fetch("example.org"))

@memoize(maxsize=2)
def slow_double(x):
    time.sleep(0.005)
    return 2 * x
for x in (1, 2, 1, 3, 2, 1):                                     # 3 evicts 2 (least recent), so 2 misses again
    slow_double(x)
print("memoize stats:", slow_double.stats)

@functools.lru_cache(maxsize=2)
def slow_double_std(x):
    return 2 * x
for x in (1, 2, 1, 3, 2, 1):
    slow_double_std(x)
print("lru_cache   :", slow_double_std.cache_info())

@validate
def transfer(amount: int, to: str, note: str = ""):
    return f"sent {amount} to {to}"
print(transfer(100, "ana"))
for bad in ((100.5, "ana"), (100, 42)):
    try:
        transfer(*bad)
    except TypeError as e:
        print("TypeError:", e)
```

```console
$ cd toolkit && python3 toolkit.py
  attempt 1 failed (network blip); retrying in 10 ms
  attempt 2 failed (network blip); retrying in 20 ms
  attempt 3 failed (network blip); retrying in 40 ms
<html from example.org>
memoize stats: {'hits': 1, 'misses': 5}
lru_cache   : CacheInfo(hits=1, misses=5, maxsize=2, currsize=2)
sent 100 to ana
TypeError: transfer(): amount must be int, got float
TypeError: transfer(): to must be str, got int
```

The hand-written LRU and `functools.lru_cache` report the same hits and misses for the same access sequence — a cheap correctness check against a reference implementation. `@validate` is a teaching example: real projects check types statically with a type checker (Chapter 8) or validate data at boundaries with a library such as pydantic (Chapter 15).

### Review checklist

- Why does `retry` re-raise with a bare `raise` on the last attempt, and what does the caller see?
- Why does `memoize` accept only positional arguments, and how would you support keyword arguments?
- What does `validate` do with annotations like `list[int]`? (`isinstance` cannot check parameterised generics — a reason to prefer static checking.)

## Answers to the Chapter 3 prediction questions

**3.6 Q1.** `[20, 20, 20]`: all three lambdas share the variable `i`, whose final value is 2 (DT-9).

**3.6 Q2.** The decorator returns `None`, so the decorated name becomes `None`, and calling it raises `TypeError: 'NoneType' object is not callable`.

# Chapter 4 — Classes, the Data Model, and Dataclasses

*Goal: define your own types that behave like built-in ones — with readable representations, equality and hashing, ordering, arithmetic operators, iteration and indexing — understand inheritance and the method resolution order, and use dataclasses and enums to remove boilerplate.*

**What you will be able to do after this chapter**

- Write classes with instance and class attributes, methods, `classmethod` and `staticmethod`.
- Use inheritance and `super()`, and read the method resolution order (MRO).
- Implement the data model ("dunder") methods: `__repr__`, `__eq__`, `__hash__`, ordering, arithmetic, `__len__`, `__getitem__`, `__iter__`, `__bool__`.
- Control attributes with properties and `__slots__`.
- Use `dataclasses` (including frozen and slotted ones) and `enum`.

## Topic 4A — Classes

## Unit 4.1 — Classes, instances, and attributes

**One-line summary:** A `class` statement creates a type; calling it creates an **instance** and runs `__init__` to initialise it. Methods receive the instance as their first parameter, conventionally `self`. Attributes set on `self` belong to the instance; attributes in the class body belong to the **class** and are shared by all instances. `@classmethod` receives the class (for alternative constructors); `@staticmethod` receives neither.

```python
# file: classes.py
class Account:
    bank = "PyBank"                                   # class attribute: shared
    count = 0

    def __init__(self, owner, balance=0):
        self.owner = owner                            # instance attributes
        self.balance = balance
        Account.count += 1

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("deposit must be positive")
        self.balance += amount
        return self

    @classmethod
    def from_csv(cls, line):                          # alternative constructor
        owner, balance = line.split(",")
        return cls(owner, int(balance))

    @staticmethod
    def fee(amount):
        return round(amount * 0.01, 2)

a = Account("ana", 100).deposit(50)
b = Account.from_csv("ben,20")
print(a.owner, a.balance, b.owner, b.balance, Account.count, a.bank, Account.fee(250))
print(vars(a), type(a).__name__, isinstance(a, Account))
a.bank = "MyBank"                                     # creates an INSTANCE attribute that shadows the class one
print(a.bank, b.bank, Account.bank)
```

```console
$ python3 classes.py
ana 150 ben 20 2 PyBank 2.5
{'owner': 'ana', 'balance': 150} Account True
MyBank PyBank PyBank
```

### Deep & Tricky

**DT-11. A mutable class attribute is shared by all instances: `items = []` in the class body is one list, and `self.items.append(x)` mutates it for everyone — while `self.items = [...]` would create an instance attribute that shadows it.** *(Python Tutorial, §9.3.5 "Class and Instance Variables")* Attribute *lookup* falls back from the instance to the class, but *assignment* always writes to the instance. That asymmetry is why `a.bank = "MyBank"` above changed only `a`, and why mutating a class-level list changes everyone. Initialise per-instance state in `__init__` (or with `field(default_factory=list)` in a dataclass, Unit 4.5).

```python
# file: shared_attr.py
class Cart:
    items = []                                        # shared by every Cart!
    def add(self, x):
        self.items.append(x)
c1, c2 = Cart(), Cart()
c1.add("apple")
print(c2.items, c1.items is c2.items is Cart.items)
```

```console
$ python3 shared_attr.py
['apple'] True
```

## Unit 4.2 — Inheritance, `super()`, MRO, and abstract base classes

**One-line summary:** A subclass inherits attributes and methods and can **override** them; `super()` calls the next implementation in the **method resolution order** (MRO), which Python computes by C3 linearisation so that each class appears once and before its bases — making cooperative multiple inheritance work. `abc.ABC` with `@abstractmethod` defines interfaces that cannot be instantiated until every abstract method is implemented.

```python
# file: inheritance.py
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self): ...
    def describe(self):
        return f"{type(self).__name__} with area {self.area():.2f}"

class Rect(Shape):
    def __init__(self, w, h):
        self.w, self.h = w, h
    def area(self):
        return self.w * self.h

class Square(Rect):
    def __init__(self, side):
        super().__init__(side, side)

print(Square(3).describe(), isinstance(Square(1), Shape))
try:
    Shape()
except TypeError as e:
    print("TypeError:", e)

class Base:
    def hello(self): return ["Base"]
class Left(Base):
    def hello(self): return ["Left"] + super().hello()
class Right(Base):
    def hello(self): return ["Right"] + super().hello()
class Child(Left, Right):
    def hello(self): return ["Child"] + super().hello()
print(Child().hello())
print([c.__name__ for c in Child.__mro__])
```

```console
$ python3 inheritance.py
Square with area 9.00 True
TypeError: Can't instantiate abstract class Shape without an implementation for abstract method 'area'
['Child', 'Left', 'Right', 'Base']
['Child', 'Left', 'Right', 'Base', 'object']
```

In the diamond, `super()` inside `Left` calls **`Right`**, not `Base` — `super()` means "next in the MRO of the actual object", which is how each class runs exactly once.

<!-- expanded -->
### Going deeper: the MRO and cooperative `super()`

With multiple inheritance, Python computes one **method resolution order** (MRO) for each class with the C3 algorithm, and `super()` means "the next class in the MRO of the *object's* class", not "my parent". When every class in a hierarchy calls `super().__init__(**kwargs)`, each `__init__` runs exactly once, even in a diamond.

```python
# file: mro.py
class Base:
    def __init__(self, **kwargs):
        print("Base.__init__")
        super().__init__(**kwargs)           # object.__init__ at the end of the chain

class Logged(Base):
    def __init__(self, *, log_name="app", **kwargs):
        print(f"Logged.__init__ ({log_name})")
        super().__init__(**kwargs)

class Cached(Base):
    def __init__(self, *, cache_size=128, **kwargs):
        print(f"Cached.__init__ ({cache_size})")
        super().__init__(**kwargs)

class Service(Logged, Cached):
    def __init__(self, **kwargs):
        print("Service.__init__")
        super().__init__(**kwargs)

print([c.__name__ for c in Service.__mro__])
Service(log_name="svc", cache_size=10)
```

```console
$ python3 mro.py
['Service', 'Logged', 'Cached', 'Base', 'object']
Service.__init__
Logged.__init__ (svc)
Cached.__init__ (10)
Base.__init__
```

`Logged`'s `super()` goes to `Cached` — a class `Logged` knows nothing about — because that is next in `Service`'s MRO. This is why cooperative classes pass along `**kwargs` they do not use: a sibling further along the chain may need them. And `Base.__init__` runs once, not twice.

## Topic 4B — The data model

## Unit 4.3 — Dunder methods: making objects behave like built-ins

**One-line summary:** Python's syntax calls special methods: `repr(x)` → `x.__repr__()`, `a == b` → `a.__eq__(b)`, `a + b` → `a.__add__(b)` (falling back to `b.__radd__(a)`), `len(x)` → `__len__`, `x[i]` → `__getitem__`, `for v in x` → `__iter__`, `bool(x)` → `__bool__` or `__len__`. Return `NotImplemented` from a binary operator for unsupported types so Python can try the other operand.

```python
# file: vector.py
import math
from functools import total_ordering

@total_ordering                                      # derives <=, >, >= from __eq__ and __lt__
class Vector:
    def __init__(self, *coords):
        self.coords = tuple(coords)
    def __repr__(self):
        return f"Vector{self.coords}"
    def __len__(self):
        return len(self.coords)
    def __getitem__(self, i):
        return self.coords[i]                        # also makes the object iterable and sliceable
    def __eq__(self, other):
        return isinstance(other, Vector) and self.coords == other.coords
    def __hash__(self):
        return hash(self.coords)                     # immutable coords: safe to hash
    def __lt__(self, other):
        return abs(self) < abs(other)
    def __abs__(self):
        return math.hypot(*self.coords)
    def __bool__(self):
        return any(self.coords)
    def __add__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(*(a + b for a, b in zip(self, other, strict=True)))
    def __mul__(self, k):
        if not isinstance(k, (int, float)):
            return NotImplemented
        return Vector(*(a * k for a in self))
    __rmul__ = __mul__                               # 3 * v works too

v, w = Vector(3, 4), Vector(1, 2)
print(v + w, 2 * v, v * 0.5, abs(v), len(v), v[0], list(v), v[::-1])
print(v == Vector(3, 4), v > w, sorted([v, w, Vector(0, 1)]), bool(Vector(0, 0)))
print({v: "hashable"}, max([v, w]))
try:
    v + 1
except TypeError as e:
    print("TypeError:", e)
```

```console
$ python3 vector.py
Vector(4, 6) Vector(6, 8) Vector(1.5, 2.0) 5.0 2 3 [3, 4] (4, 3)
True True [Vector(0, 1), Vector(1, 2), Vector(3, 4)] False
{Vector(3, 4): 'hashable'} Vector(3, 4)
TypeError: unsupported operand type(s) for +: 'Vector' and 'int'
```

### Deep & Tricky

**DT-12. Defining `__eq__` without `__hash__` makes a class unhashable: Python sets `__hash__ = None` implicitly, because the default identity hash would break the rule "equal objects must have equal hashes".** *(Python Data Model, `object.__hash__`: "A class that overrides `__eq__()` and does not define `__hash__()` will have its `__hash__()` implicitly set to None")* Instances then cannot be dict keys or set members (`TypeError: unhashable type`). If the object is immutable, define `__hash__` from the same fields as `__eq__`; if it is mutable, leaving it unhashable is correct (DT-7).

```python
# file: eq_hash.py
class P:
    def __init__(self, x): self.x = x
    def __eq__(self, o): return isinstance(o, P) and self.x == o.x
print(P.__hash__)
try:
    {P(1)}
except TypeError as e:
    print("TypeError:", e)
```

```console
$ python3 eq_hash.py
None
TypeError: unhashable type: 'P'
```

## Unit 4.4 — Properties and `__slots__`

**One-line summary:** A `@property` makes a method look like an attribute, so you can add validation or computation later **without changing callers** — Python needs no getter/setter boilerplate up front. `__slots__ = ("x", "y")` replaces the per-instance `__dict__` with fixed storage: less memory, faster attribute access, and no accidental new attributes.

```python
# file: properties.py
import sys

class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius                         # goes through the setter
    @property
    def celsius(self):
        return self._c
    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("below absolute zero")
        self._c = value
    @property
    def fahrenheit(self):                              # computed, read-only
        return self._c * 9 / 5 + 32

t = Temperature(25)
t.celsius = 30
print(t.celsius, t.fahrenheit)
for bad in (-300,):
    try:
        t.celsius = bad
    except ValueError as e:
        print("ValueError:", e)

class PointDict:
    def __init__(self, x, y): self.x, self.y = x, y
class PointSlots:
    __slots__ = ("x", "y")
    def __init__(self, x, y): self.x, self.y = x, y
d, s = PointDict(1, 2), PointSlots(1, 2)
print("with __dict__:", sys.getsizeof(d) + sys.getsizeof(d.__dict__), "bytes; with __slots__:", sys.getsizeof(s), "bytes")
try:
    s.z = 3
except AttributeError as e:
    print("AttributeError:", e)
```

```console
$ python3 properties.py
30 86.0
ValueError: below absolute zero
with __dict__: 344 bytes; with __slots__: 48 bytes
AttributeError: 'PointSlots' object has no attribute 'z' and no __dict__ for setting new attributes
```

<!-- expanded -->
### Going deeper: what `__slots__` saves, and what it costs

By default every instance has a `__dict__` holding its attributes. Declaring `__slots__ = ("x", "y")` replaces it with fixed storage for exactly those attributes — less memory and slightly faster access — at the cost of not being able to add other attributes, and (unless you add `"__weakref__"` to the slots) not supporting weak references.

```python
# file: slots.py
import sys
import tracemalloc

class PlainPoint:
    def __init__(self, x, y):
        self.x, self.y = x, y

class SlotPoint:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x, self.y = x, y

def measure(cls, n=100_000):
    tracemalloc.start()
    pts = [cls(i, i) for i in range(n)]
    current, _ = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    return current / n, pts

plain_bytes, _ = measure(PlainPoint)
slot_bytes, _ = measure(SlotPoint)
print(f"per instance: plain ~{plain_bytes:.0f} bytes, slots ~{slot_bytes:.0f} bytes")
p = SlotPoint(1, 2)
try:
    p.z = 3
except AttributeError as e:
    print("AttributeError:", e)
print(hasattr(PlainPoint(1, 2), "__dict__"), hasattr(p, "__dict__"))
```

```console
$ python3 slots.py
per instance: plain ~128 bytes, slots ~88 bytes
AttributeError: 'SlotPoint' object has no attribute 'z' and no __dict__ for setting new attributes
True False
```

The numbers include the two integer objects each point refers to (small integers are cached, larger ones are not), so they are measurements of a realistic workload rather than of the bare instances. Since Python 3.10, `@dataclass(slots=True)` generates `__slots__` for you.

## Topic 4C — Less boilerplate

## Unit 4.5 — Dataclasses and enums

**One-line summary:** `@dataclass` generates `__init__`, `__repr__`, and `__eq__` from annotated fields; options add ordering (`order=True`), immutability and hashing (`frozen=True`), and `__slots__` (`slots=True`). Use `field(default_factory=list)` for mutable defaults. `enum.Enum` gives named constants with identity, iteration, and exhaustive `match` cases.

```python
# file: dataclasses_demo.py
from dataclasses import dataclass, field, asdict, replace
from enum import Enum, auto

class Status(Enum):
    TODO = auto()
    DOING = auto()
    DONE = auto()

@dataclass(order=True)
class Task:
    priority: int
    title: str = field(compare=False)
    status: Status = field(default=Status.TODO, compare=False)
    tags: list[str] = field(default_factory=list, compare=False)

@dataclass(frozen=True, slots=True)
class Money:
    amount: int                                       # in cents
    currency: str = "EUR"
    def __add__(self, other):
        if other.currency != self.currency:
            raise ValueError("currency mismatch")
        return Money(self.amount + other.amount, self.currency)

tasks = [Task(2, "write"), Task(1, "plan", tags=["urgent"]), Task(3, "ship")]
print(sorted(tasks)[0])
t = tasks[0]
t.status = Status.DONE
print(asdict(t), replace(t, title="rewrite").title)
m = Money(150) + Money(250)
print(m, {m: "frozen dataclasses are hashable"})
try:
    m.amount = 0
except Exception as e:
    print(type(e).__name__, e)
print([s.name for s in Status], Status["DONE"] is Status.DONE, Status.DONE.value)
```

```console
$ python3 dataclasses_demo.py
Task(priority=1, title='plan', status=<Status.TODO: 1>, tags=['urgent'])
{'priority': 2, 'title': 'write', 'status': <Status.DONE: 3>, 'tags': []} rewrite
Money(amount=400, currency='EUR') {Money(amount=400, currency='EUR'): 'frozen dataclasses are hashable'}
FrozenInstanceError cannot assign to field 'amount'
['TODO', 'DOING', 'DONE'] True 3
```

### Pattern

**P-5. Model data with dataclasses (frozen when the values should not change), behaviour with methods, and integration with the language through the data model — implement `__repr__` always, `__eq__`/`__hash__` together, and operators only when their meaning is obvious.** *(Practice)* Prefer composition (an object holding others) to deep inheritance; use ABCs or `typing.Protocol` (Chapter 8) to describe interfaces. *Trade-off:* dunder methods make code concise but can surprise readers when the meaning is not conventional — `+` on vectors is clear, `+` on database connections is not.

<!-- expanded -->
### Going deeper: dataclass options — `frozen`, `order`, `field`, and `__post_init__`

`@dataclass` writes `__init__`, `__repr__`, and `__eq__`. Options add more: `frozen=True` makes instances immutable (and hashable), `order=True` adds `<` and friends comparing fields in order, `kw_only=True` makes every field keyword-only, and `field(default_factory=list)` gives each instance its own mutable default. `__post_init__` runs after the generated `__init__`, for validation and derived fields.

```python
# file: dataclasses2.py
from dataclasses import dataclass, field, asdict, replace

@dataclass(frozen=True, order=True)
class Version:
    major: int
    minor: int = 0
    patch: int = 0
    def __str__(self):
        return f"{self.major}.{self.minor}.{self.patch}"

@dataclass(kw_only=True)
class Order:
    id: str
    items: list[str] = field(default_factory=list)       # a new list per instance
    total_paise: int = 0
    note: str = field(default="", repr=False)
    def __post_init__(self):
        if self.total_paise < 0:
            raise ValueError("total_paise must be >= 0")

versions = sorted([Version(1, 10), Version(1, 4, 2), Version(2)])
print([str(v) for v in versions], Version(1, 4) < Version(1, 10))
v = Version(1, 2, 3)
print(replace(v, patch=4), {v: "release"}[Version(1, 2, 3)])
try:
    v.major = 9
except Exception as e:
    print(type(e).__name__, e)

a, b = Order(id="o1"), Order(id="o2")
a.items.append("pen")
print(a, b, asdict(a))
try:
    Order(id="o3", total_paise=-5)
except ValueError as e:
    print("ValueError:", e)
```

```console
$ python3 dataclasses2.py
['1.4.2', '1.10.0', '2.0.0'] True
1.2.4 release
FrozenInstanceError cannot assign to field 'major'
Order(id='o1', items=['pen'], total_paise=0) Order(id='o2', items=[], total_paise=0) {'id': 'o1', 'items': ['pen'], 'total_paise': 0, 'note': ''}
ValueError: total_paise must be >= 0
```

`field(default_factory=list)` exists because of DT on default arguments (Unit 3.4): `items: list[str] = []` is rejected by `@dataclass` with a `ValueError`, precisely to stop every instance sharing one list.

### Mastery check

1. **Predict.** For `class A: pass` and `class B(A): pass`, what does `B.__mro__` contain?
2. **Predict.** Is an instance of `@dataclass class C: x: int` (no options) hashable?

## Chapter 4 Project — An exact polynomial type

**Goal:** a `Polynomial` class with `Fraction` coefficients that supports `+`, `-`, `*`, `**` (non-negative integer powers), calling (`p(x)` evaluates with Horner's rule), equality and hashing, a mathematical `repr`, derivative, and iteration over coefficients. **Verify** the arithmetic property-style: for random polynomials p, q and random x, check `(p * q)(x) == p(x) * q(x)`, `(p + q)(x) == p(x) + q(x)`, and the derivative against a finite-difference identity that is exact for polynomials.

```python
# file: poly/poly.py
import random
from fractions import Fraction
from itertools import zip_longest

class Polynomial:
    __slots__ = ("c",)
    def __init__(self, *coeffs):                       # c[i] is the coefficient of x^i
        c = [Fraction(x) for x in coeffs]
        while c and c[-1] == 0:
            c.pop()                                    # normalise: no trailing zeros
        self.c = tuple(c)
    @property
    def degree(self):
        return len(self.c) - 1                         # -1 for the zero polynomial
    def __call__(self, x):
        acc = Fraction(0)
        for coef in reversed(self.c):                  # Horner: ((a_n x + a_{n-1}) x + ...)
            acc = acc * x + coef
        return acc
    def __add__(self, o):
        o = _lift(o)
        return Polynomial(*(a + b for a, b in zip_longest(self.c, o.c, fillvalue=0))) if o is not NotImplemented else o
    __radd__ = __add__
    def __neg__(self):
        return Polynomial(*(-a for a in self.c))
    def __sub__(self, o):
        o = _lift(o)
        return self + (-o) if o is not NotImplemented else o
    def __rsub__(self, o):
        return (-self) + o
    def __mul__(self, o):
        o = _lift(o)
        if o is NotImplemented:
            return o
        out = [Fraction(0)] * max(0, len(self.c) + len(o.c) - 1)
        for i, a in enumerate(self.c):
            for j, b in enumerate(o.c):
                out[i + j] += a * b
        return Polynomial(*out)
    __rmul__ = __mul__
    def __pow__(self, n):
        if not isinstance(n, int) or n < 0:
            return NotImplemented
        result, base = Polynomial(1), self
        while n:                                       # binary exponentiation
            if n & 1:
                result *= base
            base, n = base * base, n >> 1
        return result
    def __eq__(self, o):
        o = _lift(o)
        return o is not NotImplemented and self.c == o.c
    def __hash__(self):
        return hash(self.c)
    def __iter__(self):
        return iter(self.c)
    def __bool__(self):
        return bool(self.c)
    def derivative(self):
        return Polynomial(*(i * a for i, a in enumerate(self.c) if i))
    def __repr__(self):
        if not self.c:
            return "0"
        terms = []
        for i in range(len(self.c) - 1, -1, -1):
            a = self.c[i]
            if a == 0:
                continue
            sign = "-" if a < 0 else "+"
            a = abs(a)
            coef = "" if a == 1 and i else str(a)
            var = "" if i == 0 else "x" if i == 1 else f"x^{i}"
            terms.append((sign, coef + var))
        s = ("-" if terms[0][0] == "-" else "") + terms[0][1]
        return s + "".join(f" {sg} {t}" for sg, t in terms[1:])

def _lift(o):
    if isinstance(o, Polynomial):
        return o
    if isinstance(o, (int, Fraction)):
        return Polynomial(o)
    return NotImplemented

x = Polynomial(0, 1)
p = (x + 1) ** 3
print("(x + 1)^3 =", p, "| derivative:", p.derivative(), "| p(2) =", p(2))
print("(x - 1)(x + 1) =", (x - 1) * (x + 1), "| equal to x^2 - 1:", (x - 1) * (x + 1) == x ** 2 - 1)
print(Polynomial(Fraction(1, 2), 0, -3), "| zero:", x - x, "| hashable:", len({x ** 2 - 1, (x - 1) * (x + 1)}))

rng = random.Random(4)
rand_poly = lambda: Polynomial(*(rng.randint(-5, 5) for _ in range(rng.randint(0, 5))))
for _ in range(2000):
    a, b, v = rand_poly(), rand_poly(), Fraction(rng.randint(-9, 9), rng.randint(1, 4))
    assert (a * b)(v) == a(v) * b(v) and (a + b)(v) == a(v) + b(v) and (a - b)(v) == a(v) - b(v)
    assert (a ** 2)(v) == a(v) ** 2
    # exact identity for polynomials: d/dx a at v equals the coefficient of h in a(v + h)
    shifted = Polynomial(0) + a(Polynomial(v, 1))   # a(v + h) as a polynomial in h (lifted: a may be zero)
    assert a.derivative()(v) == (shifted.c[1] if len(shifted.c) > 1 else 0)
print("2,000 random checks of +, -, *, **, and derivative passed")
```

```console
$ cd poly && python3 poly.py
(x + 1)^3 = x^3 + 3x^2 + 3x + 1 | derivative: 3x^2 + 6x + 3 | p(2) = 27
(x - 1)(x + 1) = x^2 - 1 | equal to x^2 - 1: True
-3x^2 + 1/2 | zero: 0 | hashable: 1
2,000 random checks of +, -, *, **, and derivative passed
```

The derivative check evaluates `a` at the **polynomial** `v + h` — Horner's rule works for any values supporting `*` and `+`, including Polynomials, so `a(v + h)` is a polynomial in h whose linear coefficient is exactly a′(v). Duck typing (Chapter 8) makes the class compose with itself. The first version of this check crashed on the zero polynomial: with no coefficients, Horner's loop never runs and returns its starting value `Fraction(0)` rather than a Polynomial — so the result is lifted with `Polynomial(0) + …`. Random tests find such empty cases quickly.

### Review checklist

- Why does `__init__` strip trailing zero coefficients, and what would break in `__eq__` and `__hash__` without it?
- Why do `__radd__` and `__rmul__` exist, and which expression in the program needs them?
- `Polynomial` defines `__eq__` and `__hash__` — what makes it safe to hash (DT-7)?

## Answers to the Chapter 4 prediction questions

**4.5 Q1.** `(B, A, object)` — every class ultimately inherits from `object`.

**4.5 Q2.** No. A plain dataclass generates `__eq__` (so `__hash__` becomes `None`, DT-12) and is mutable. With `frozen=True` (or `unsafe_hash=True`) it gets a generated `__hash__`.

# Chapter 5 — Iterators, Generators, and Comprehensions

*Goal: understand the iteration protocol that every `for` loop uses, write comprehensions that are clear and fast, build lazy pipelines with generators that process data larger than memory, and use `itertools` to combine iterators without writing loops.*

**What you will be able to do after this chapter**

- Explain iterables versus iterators, and implement both.
- Write list, dict, and set comprehensions and generator expressions.
- Write generator functions with `yield` and `yield from`, and chain them into pipelines.
- Use `itertools` (`islice`, `chain`, `groupby`, `accumulate`, `batched`, `product`, …).
- Measure the memory difference between eager lists and lazy generators.

## Topic 5A — The protocol

## Unit 5.1 — Iterables and iterators

**One-line summary:** An **iterable** is anything with `__iter__` that returns an **iterator**; an iterator has `__next__`, which returns the next value or raises `StopIteration`, and its own `__iter__` returns itself. `for x in obj` calls `iter(obj)` once and then `next()` until `StopIteration`. Lists are iterables (you can loop over them many times); iterators are **one-shot**.

```python
# file: protocol.py
class Countdown:                                      # an iterable: each iter() gives a fresh iterator
    def __init__(self, start):
        self.start = start
    def __iter__(self):
        return CountdownIterator(self.start)

class CountdownIterator:
    def __init__(self, n):
        self.n = n
    def __iter__(self):
        return self
    def __next__(self):
        if self.n <= 0:
            raise StopIteration
        self.n -= 1
        return self.n + 1

c = Countdown(3)
print(list(c), list(c))                               # iterable: can be looped twice
it = iter([10, 20, 30])
print(next(it), next(it), next(it), next(it, "done"))  # next with a default instead of StopIteration

loop_it = iter("ab")                                  # what a for loop does, by hand
while True:
    try:
        ch = next(loop_it)
    except StopIteration:
        break
    print("got", ch)
```

```console
$ python3 protocol.py
[3, 2, 1] [3, 2, 1]
10 20 30 done
got a
got b
```

### Deep & Tricky

**DT-13. Iterators are exhausted after one pass: a generator, `map`, `zip`, `filter`, a file object, or `dict.items()`'s iterator yields nothing the second time — silently, with no error — so code that loops twice over a parameter works for lists and breaks for iterators.** *(Python Glossary, "iterator": "Once an iterator's `__next__()` method raises StopIteration… subsequent calls just raise StopIteration again")* Typical symptom: `max(data)` then `sum(data)` where `data` is a generator returns the right maximum and a sum of 0. If a function needs several passes, materialise with `list(data)` once, or accept an iterable and call `iter()` on it per pass.

```python
# file: exhausted.py
squares = (x * x for x in range(5))
print("max:", max(squares), "| sum afterwards:", sum(squares))
pairs = zip("abc", [1, 2, 3])
print(list(pairs), list(pairs))
```

```console
$ python3 exhausted.py
max: 16 | sum afterwards: 0
[('a', 1), ('b', 2), ('c', 3)] []
```

<!-- expanded -->
### Going deeper: writing an iterable class with `__iter__` as a generator

The simplest way to make a class iterable is to write its `__iter__` as a generator function: each call returns a fresh iterator, so the object can be looped over many times — unlike an iterator, which is exhausted after one pass (DT-earlier in this unit).

```python
# file: iterable_class.py
class Countdown:
    def __init__(self, start):
        self.start = start
    def __iter__(self):                 # a generator: a new iterator each time
        n = self.start
        while n > 0:
            yield n
            n -= 1
    def __len__(self):
        return self.start

c = Countdown(3)
print(list(c), list(c), len(c), 2 in c, sorted(c))
it = iter(c)
print(next(it), next(it), list(it), list(it))   # the iterator is used up
```

```console
$ python3 iterable_class.py
[3, 2, 1] [3, 2, 1] 3 True [1, 2, 3]
3 2 [1] []
```

## Unit 5.2 — Comprehensions and generator expressions

**One-line summary:** `[expr for x in it if cond]` builds a list; `{k: v for ...}` a dict; `{expr for ...}` a set; `(expr for ...)` a **generator expression** that produces items lazily. Multiple `for` clauses nest left to right. Comprehensions have their **own scope** (the loop variable does not leak), and are usually faster than the equivalent `for` loop with `append`.

```python
# file: comprehensions.py
import sys
import timeit

nums = range(1, 11)
print([n * n for n in nums if n % 2 == 0])
print({n: n * n for n in range(4)}, {c.lower() for c in "Hello"})
print([(r, c) for r in range(2) for c in "ab"])         # nested: r outer, c inner
matrix = [[1, 2, 3], [4, 5, 6]]
print([[row[i] for row in matrix] for i in range(3)])   # transpose
x = "outer"
_ = [x for x in range(3)]
print("loop variable did not leak:", x)

big_list = [n for n in range(1_000_000)]
big_gen = (n for n in range(1_000_000))
print(f"list: {sys.getsizeof(big_list):,} bytes; generator: {sys.getsizeof(big_gen)} bytes")

def with_loop():
    out = []
    for n in range(10_000):
        out.append(n * 2)
    return out
def with_comp():
    return [n * 2 for n in range(10_000)]
for f in (with_loop, with_comp):
    print(f"{f.__name__}: {min(timeit.repeat(f, number=200, repeat=3)) / 200 * 1e6:.0f} µs")
```

```console
$ python3 comprehensions.py
[4, 16, 36, 64, 100]
{0: 0, 1: 1, 2: 4, 3: 9} {'h', 'e', 'o', 'l'}
[(0, 'a'), (0, 'b'), (1, 'a'), (1, 'b')]
[[1, 4], [2, 5], [3, 6]]
loop variable did not leak: outer
list: 8,448,728 bytes; generator: 192 bytes
with_loop: 248 µs
with_comp: 239 µs
```

<!-- expanded -->
### Going deeper: comprehension scope, nesting order, and memory

A comprehension has its own scope: its loop variable does not leak (unlike a `for` loop's). Nested `for` clauses read in the same order as nested loops — outermost first. And a list comprehension builds the whole list, while a generator expression produces items on demand, which matters for large data.

```python
# file: comprehensions2.py
import sys

x = "unchanged"
squares = [x * x for x in range(5)]
print(squares, x)                                    # x did not leak

pairs = [(r, c) for r in range(3) for c in range(r)]  # like: for r ...: for c ...:
print(pairs)
matrix = [[1, 2, 3], [4, 5, 6]]
print([n for row in matrix for n in row], [list(col) for col in zip(*matrix)])
words = ["apple", "bob", "civic", "dad", "eagle"]
print({w: len(w) for w in words if w == w[::-1]}, {len(w) for w in words})

big_list = [n * n for n in range(1_000_000)]
big_gen = (n * n for n in range(1_000_000))
print(f"list: {sys.getsizeof(big_list):,} bytes, generator: {sys.getsizeof(big_gen)} bytes")
print(sum(n * n for n in range(1_000_000)) == sum(big_list))   # no brackets needed as the only argument
```

```console
$ python3 comprehensions2.py
[0, 1, 4, 9, 16] unchanged
[(1, 0), (2, 0), (2, 1)]
[1, 2, 3, 4, 5, 6] [[1, 4], [2, 5], [3, 6]]
{'bob': 3, 'civic': 5, 'dad': 3} {3, 5}
list: 8,448,728 bytes, generator: 200 bytes
True
```

`sys.getsizeof` of the list counts only its array of 8-byte pointers (8 MB); the million integer objects it refers to are extra. The generator is a couple of hundred bytes whatever the range.

## Topic 5B — Generators

## Unit 5.3 — Generator functions and lazy pipelines

**One-line summary:** A function containing `yield` is a **generator function**: calling it returns a generator object without running the body; each `next()` runs until the next `yield`, which produces a value and **suspends** the function, keeping its local state. Chained generators form **pipelines** that process one item at a time — constant memory, regardless of input size — and can even be infinite.

```python
# file: generators.py
def countdown(n):
    print("  (starting)")
    while n > 0:
        yield n
        n -= 1
    print("  (finished)")

g = countdown(2)
print("created:", g)
print(next(g)); print(next(g))
print(next(g, "exhausted"))

def naturals():                                       # infinite
    n = 1
    while True:
        yield n
        n += 1

def take(n, it):
    for i, x in enumerate(it):
        if i == n:
            return
        yield x

def evens(it):
    return (x for x in it if x % 2 == 0)

def squared(it):
    for x in it:
        yield x * x

print(list(take(5, squared(evens(naturals())))))      # lazy: only 10 naturals are ever produced
```

```console
$ python3 generators.py
created: <generator object countdown at 0x7f79b14ddb40>
  (starting)
2
1
  (finished)
exhausted
[4, 16, 36, 64, 100]
```

<!-- expanded -->
### Going deeper: a lazy pipeline over a large file

Generators chain into pipelines where each stage pulls one item at a time from the previous one. Processing a log file this way uses constant memory however big the file is, and stops reading as soon as the consumer has enough.

```python
# file: pipeline.py
import itertools

def generate_log(path, n):
    with open(path, "w") as f:
        for i in range(n):
            level = "ERROR" if i % 1000 == 7 else "INFO"
            f.write(f"2026-11-16T10:{i // 60 % 60:02}:{i % 60:02} {level} request {i} took {i % 97} ms\n")

def read_lines(path):
    with open(path) as f:
        for line in f:                      # files are iterators: one line at a time
            yield line.rstrip("\n")

def parse(lines):
    for line in lines:
        ts, level, *rest = line.split(" ")
        yield {"ts": ts, "level": level, "ms": int(rest[-2])}

def errors(records):
    return (r for r in records if r["level"] == "ERROR")

generate_log("big.log", 200_000)
lines_read = 0
def counting(lines):
    global lines_read
    for l in lines:
        lines_read += 1
        yield l

first_three = list(itertools.islice(errors(parse(counting(read_lines("big.log")))), 3))
print(first_three)
print("lines read to find them:", lines_read, "of 200000")
```

```console
$ python3 pipeline.py
[{'ts': '2026-11-16T10:00:07', 'level': 'ERROR', 'ms': 7}, {'ts': '2026-11-16T10:16:47', 'level': 'ERROR', 'ms': 37}, {'ts': '2026-11-16T10:33:27', 'level': 'ERROR', 'ms': 67}]
lines read to find them: 2008 of 200000
```

The pipeline read only as far as the third error (line 2008) and stopped: nothing upstream runs until something downstream asks for an item.

## Unit 5.4 — `yield from`, `send`, and closing

**One-line summary:** `yield from sub` delegates to a sub-iterator, yielding all its values (and passing `send`/`throw` through) — the clean way to write recursive generators. `gen.send(value)` resumes a generator and makes the paused `yield` expression evaluate to `value`; `gen.close()` raises `GeneratorExit` inside it so `finally` blocks run. (Coroutines in `asyncio` grew out of this machinery, Chapter 11.)

```python
# file: delegation.py
def flatten(nested):
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)                  # delegate recursively
        else:
            yield item
print(list(flatten([1, [2, [3, (4, 5)]], [], 6])))

def running_average():
    total = count = 0
    average = None
    try:
        while True:
            value = yield average                     # receives values via send()
            total += value; count += 1
            average = total / count
    finally:
        print("  generator closed: cleanup runs")

avg = running_average()
next(avg)                                             # "prime" it: run to the first yield
print([avg.send(v) for v in (10, 20, 60)])
avg.close()
```

```console
$ python3 delegation.py
[1, 2, 3, 4, 5, 6]
[10.0, 15.0, 30.0]
  generator closed: cleanup runs
```

<!-- expanded -->
### Deep & Tricky

**DT-14. A generator's `finally` block (and a `with` inside it) runs only when the generator finishes, is closed, or is garbage-collected. A generator abandoned halfway keeps its file open until then — in CPython usually at once (reference counting), but later or never in PyPy or when a reference cycle holds it.** *(Guarantee: PEP 342 — `close()` raises `GeneratorExit` inside the generator; when finalisation happens is implementation-dependent)* Close generators explicitly when you stop early, with `contextlib.closing` or by calling `.close()`.

```python
# file: genclose.py
from contextlib import closing

def numbers():
    print("  open resource")
    try:
        for i in range(10):
            yield i
    finally:
        print("  close resource")

gen = numbers()
print("first two:", next(gen), next(gen))
print("abandoned; nothing closed yet")
gen.close()
print("after close()")

with closing(numbers()) as g:
    for n in g:
        if n == 1:
            break
print("after with closing(...)")
```

```console
$ python3 genclose.py
  open resource
first two: 0 1
abandoned; nothing closed yet
  close resource
after close()
  open resource
  close resource
after with closing(...)
```

## Unit 5.5 — `itertools`

**One-line summary:** `itertools` provides fast, lazy building blocks: infinite (`count`, `cycle`, `repeat`), slicing and filtering (`islice`, `takewhile`, `dropwhile`, `compress`), combining (`chain`, `zip_longest`, `product`, `permutations`, `combinations`), grouping and accumulating (`groupby`, `accumulate`, `pairwise`, `batched` — 3.12+), and splitting (`tee`).

```python
# file: itertools_demo.py
from itertools import accumulate, batched, chain, count, groupby, islice, pairwise, product, takewhile, zip_longest
import operator

print(list(islice(count(10, 5), 4)), list(takewhile(lambda x: x < 20, count(15))))
print(list(chain("ab", [1, 2])), list(zip_longest("abc", [1], fillvalue="-")))
print(list(accumulate([3, 1, 4, 1, 5])), list(accumulate([3, 1, 4, 1, 5], max)), list(accumulate([1, 2, 3, 4], operator.mul)))
print(list(pairwise("abcd")), list(batched(range(7), 3)))
print(["".join(p) for p in product("ab", repeat=2)])
data = ["apple", "avocado", "banana", "blueberry", "cherry", "apricot"]
print({k: list(g) for k, g in groupby(data, key=lambda w: w[0])})          # NOT sorted: 'a' appears twice
print({k: list(g) for k, g in groupby(sorted(data), key=lambda w: w[0])})  # sorted first: one group per key
```

```console
$ python3 itertools_demo.py
[10, 15, 20, 25] [15, 16, 17, 18, 19]
['a', 'b', 1, 2] [('a', 1), ('b', '-'), ('c', '-')]
[3, 4, 8, 9, 14] [3, 3, 4, 4, 5] [1, 2, 6, 24]
[('a', 'b'), ('b', 'c'), ('c', 'd')] [(0, 1, 2), (3, 4, 5), (6,)]
['aa', 'ab', 'ba', 'bb']
{'a': ['apricot'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}
{'a': ['apple', 'apricot', 'avocado'], 'b': ['banana', 'blueberry'], 'c': ['cherry']}
```

### Deep & Tricky

**DT-15. `itertools.groupby` groups only **consecutive** equal keys — like Unix `uniq`, not SQL `GROUP BY` — so unsorted input produces several groups for the same key, and building a dict from it silently keeps only the last group per key.** *(Python documentation, `itertools.groupby`: "Generally, the iterable needs to already be sorted on the same key function")* In the demo, the unsorted version loses `apple` and `avocado`: the third group with key `a` overwrote the first. Sort by the same key first, or use a `defaultdict(list)` when the input should not be sorted.

### Pattern

**P-6. Process streams lazily: write each step as a generator (or generator expression), chain them, and materialise only at the end — or never. Use comprehensions for building collections, `itertools` instead of hand-written index loops.** *(Practice)* Lazy pipelines use constant memory and can start producing results immediately. *Trade-off:* generators are single-pass (DT-13), harder to debug (no `len`, no indexing, values computed on demand), and slightly slower per item than a list comprehension for small data; materialise when you need several passes or random access.

### Mastery check

1. **Predict.** What does `g = (print(x) for x in range(3))` print by itself, before anything iterates over `g`?
2. **Predict.** What does `list(zip(*[iter(range(6))] * 2))` produce, and why?

## Chapter 5 Project — A lazy log-analysis pipeline

**Goal:** analyse a 1,000,000-line web-server log (generated deterministically to a file) with a **lazy pipeline** — read lines, parse, filter errors, group by endpoint, keep running counts and the slowest requests — and compare the **peak memory** (measured with `tracemalloc`) against an eager version that reads everything into lists first. Both must produce identical results.

```python
# file: logs/logs.py
import heapq
import random
import time
import tracemalloc
from collections import Counter
from pathlib import Path

LOG = Path("access.log")
ENDPOINTS = ["/", "/login", "/api/items", "/api/orders", "/static/app.js", "/search"]

def make_log(n):
    rng = random.Random(5)
    with LOG.open("w") as f:
        for i in range(n):
            status = rng.choices([200, 304, 404, 500], weights=[85, 8, 5, 2])[0]
            ms = int(rng.expovariate(1 / 40))
            f.write(f"2026-11-15T10:{i // 60000 % 60:02d}:{i // 1000 % 60:02d} {rng.choice(ENDPOINTS)} {status} {ms}\n")

def parse(lines):
    for line in lines:
        ts, path, status, ms = line.split()
        yield path, int(status), int(ms)

def analyse_lazy():
    with LOG.open() as f:
        records = parse(f)                              # nothing is read yet
        errors, slowest = Counter(), []
        for path, status, ms in records:
            if status >= 500:
                errors[path] += 1
            if len(slowest) < 5:
                heapq.heappush(slowest, ms)
            elif ms > slowest[0]:
                heapq.heapreplace(slowest, ms)
    return errors.most_common(3), sorted(slowest, reverse=True)

def analyse_eager():
    lines = LOG.read_text().splitlines()                # everything in memory
    records = [tuple(l.split()) for l in lines]
    parsed = [(p, int(s), int(m)) for _, p, s, m in records]
    errors = Counter(p for p, s, _ in parsed if s >= 500)
    slowest = sorted((m for _, _, m in parsed), reverse=True)[:5]
    return errors.most_common(3), slowest

make_log(1_000_000)
print(f"log file: {LOG.stat().st_size / 1e6:.1f} MB")
results = {}
for name, f in (("lazy pipeline", analyse_lazy), ("eager lists", analyse_eager)):
    tracemalloc.start()
    t = time.perf_counter()
    results[name] = f()
    el = time.perf_counter() - t
    _, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    print(f"{name:>14}: peak memory {peak / 1e3:>9,.0f} kB, {el:.1f} s")
print("same results:", results["lazy pipeline"] == results["eager lists"])
print("top 5xx endpoints:", results["lazy pipeline"][0], "| slowest (ms):", results["lazy pipeline"][1])
LOG.unlink()
```

```console
$ cd logs && python3 logs.py
log file: 36.0 MB
 lazy pipeline: peak memory        23 kB, 2.3 s
   eager lists: peak memory   433,701 kB, 5.1 s
same results: True
top 5xx endpoints: [('/login', 3430), ('/api/orders', 3411), ('/', 3400)] | slowest (ms): [668, 571, 555, 511, 509]
```

The lazy version's memory does not depend on the file size: one line, one parsed record, a counter, and a five-element heap are alive at any moment. The eager version holds every line, every tuple, and every integer at once — here about 23 kB versus 434 MB, a factor of nearly 20,000, for the same answer. (`tracemalloc` measures Python allocations and slows execution; compare the timings only with each other.)

### Review checklist

- Where in `analyse_lazy` does reading from the file actually happen?
- Why is a heap of size 5 used for the slowest requests instead of sorting?
- What would you change to process a log that is still being written (`tail -f`)?

## Answers to the Chapter 5 prediction questions

**5.5 Q1.** Nothing: a generator expression does not run until iterated; the `print` calls happen only when items are requested.

**5.5 Q2.** `[(0, 1), (2, 3), (4, 5)]`: the list holds **the same iterator twice**, so `zip` takes alternately from one iterator — consecutive pairs. (`itertools.batched` does this more readably.)

# Chapter 6 — Errors and Context Managers

*Goal: handle errors deliberately — catch what you can handle, let the rest propagate with useful context — design your own exception types, use exception groups for concurrent failures, and guarantee cleanup with context managers so that files, locks, and transactions are never left half-done.*

**What you will be able to do after this chapter**

- Use `try` / `except` / `else` / `finally` correctly, and read tracebacks.
- Raise and define exceptions; chain them with `raise ... from ...`; add notes.
- Choose between "easier to ask forgiveness" (EAFP) and "look before you leap" (LBYL).
- Handle several errors at once with `ExceptionGroup` and `except*`.
- Write context managers as classes and with `contextlib.contextmanager`; use `ExitStack` and `suppress`.

## Topic 6A — Exceptions

## Unit 6.1 — `try`, `except`, `else`, `finally`

**One-line summary:** Code in `try` runs; if an exception is raised, the first matching `except` clause handles it (matching by class, including subclasses); `else` runs only if no exception occurred; `finally` **always** runs — after success, after a handled exception, and even while an unhandled one propagates. Catch the **narrowest** exception type you can actually handle.

```python
# file: basics.py
def parse_age(text):
    try:
        age = int(text)
    except ValueError:
        print(f"  {text!r} is not a number")
        return None
    else:                                              # only when int() succeeded
        if age < 0:
            raise ValueError(f"age cannot be negative: {age}")
        return age
    finally:
        print(f"  (parse_age({text!r}) done)")

for s in ("42", "forty", "-3"):
    try:
        print("result:", parse_age(s))
    except ValueError as e:
        print("caller caught:", e)

print([cls.__name__ for cls in ZeroDivisionError.__mro__])     # exceptions form a class hierarchy
try:
    1 / 0
except ArithmeticError as e:                           # a base class catches its subclasses
    print("caught via base class:", type(e).__name__, "-", e)
```

```console
$ python3 basics.py
  (parse_age('42') done)
result: 42
  'forty' is not a number
  (parse_age('forty') done)
result: None
  (parse_age('-3') done)
caller caught: age cannot be negative: -3
['ZeroDivisionError', 'ArithmeticError', 'Exception', 'BaseException', 'object']
caught via base class: ZeroDivisionError - division by zero
```

### Deep & Tricky

**DT-16. A `return` (or `break`/`continue`) inside `finally` silently discards an exception that is propagating — the error simply disappears; since Python 3.14 this is a `SyntaxWarning` (PEP 765), but on 3.13 it is legal and silent.** *(Python Language Reference, "The try statement": "If the finally clause executes a return, break or continue statement, the saved exception is discarded")* Use `finally` only for cleanup that cannot fail, never to produce a return value; put the return value in `try` or `else`.

```python
# file: finally_return.py
def risky():
    try:
        raise RuntimeError("important failure")
    finally:
        return "everything is fine"                   # swallows the RuntimeError!
print(risky())
```

```console
$ python3 finally_return.py
everything is fine
```

## Unit 6.2 — Raising, custom exceptions, chaining, and notes

**One-line summary:** Raise with `raise SomeError("message")`. Define a small hierarchy for your library (`class AppError(Exception)`, then specific subclasses) so callers can catch broadly or narrowly. When converting one error into another, use `raise NewError(...) from original` to keep the cause in the traceback (`from None` hides it deliberately). `e.add_note(...)` (3.11+) attaches context without changing the type.

```python
# file: custom.py
import json
import traceback

class ConfigError(Exception):
    """Base class for configuration problems."""

class MissingKey(ConfigError):
    def __init__(self, key):
        super().__init__(f"missing required key {key!r}")
        self.key = key

def load(text):
    try:
        cfg = json.loads(text)
    except json.JSONDecodeError as e:
        raise ConfigError("config is not valid JSON") from e      # keep the cause
    for key in ("host", "port"):
        if key not in cfg:
            raise MissingKey(key)
    return cfg

for text in ('{"host": "db", "port": 5432}', '{"host": "db"}', "{not json"):
    try:
        print("loaded:", load(text))
    except MissingKey as e:
        print("MissingKey for", e.key)
    except ConfigError as e:
        print("ConfigError:", e, "| caused by:", repr(e.__cause__))

try:
    try:
        load('{"host": "db"}')
    except ConfigError as e:
        e.add_note("while loading /etc/app.json")
        raise
except ConfigError:
    print(traceback.format_exc().strip().splitlines()[-2:])
```

```console
$ python3 custom.py
loaded: {'host': 'db', 'port': 5432}
MissingKey for port
ConfigError: config is not valid JSON | caused by: JSONDecodeError('Expecting property name enclosed in double quotes: line 1 column 2 (char 1)')
["MissingKey: missing required key 'port'", 'while loading /etc/app.json']
```

<!-- expanded -->
### Going deeper: exception chaining and `add_note`

`raise NewError(...) from err` records the original as `__cause__`, and the traceback shows both with "The above exception was the direct cause of the following exception". Raising inside an `except` block *without* `from` sets `__context__` instead ("During handling of the above exception, another exception occurred") — which usually means a bug in the handler. `raise ... from None` hides the original. Python 3.11 added `e.add_note(text)` to attach context without changing the exception's type.

```python
# file: chaining.py
import traceback

class ConfigError(Exception):
    pass

def load_port(text):
    try:
        return int(text)
    except ValueError as err:
        raise ConfigError(f"port must be a number, got {text!r}") from err

for fn in (lambda: load_port("80a"),):
    try:
        fn()
    except ConfigError as e:
        print("message:", e, "| cause:", repr(e.__cause__))
        lines = traceback.format_exception(e)
        print("traceback mentions the cause:", any("direct cause" in l for l in lines))

def parse_rows(rows):
    for i, row in enumerate(rows, 1):
        try:
            yield int(row)
        except ValueError as e:
            e.add_note(f"while parsing row {i} of the input")
            raise

try:
    list(parse_rows(["1", "2", "x"]))
except ValueError as e:
    print(type(e).__name__, e, e.__notes__)
```

```console
$ python3 chaining.py
message: port must be a number, got '80a' | cause: ValueError("invalid literal for int() with base 10: '80a'")
traceback mentions the cause: True
ValueError invalid literal for int() with base 10: 'x' ['while parsing row 3 of the input']
```

## Unit 6.3 — EAFP and LBYL

**One-line summary:** **LBYL** ("look before you leap") checks preconditions first (`if key in d: ...`); **EAFP** ("easier to ask forgiveness than permission") just tries and handles the failure (`try: d[key] except KeyError`). Python favours EAFP: it avoids duplicated work and, crucially, **race conditions** between the check and the use (a file can disappear after `os.path.exists` returns True).

```python
# file: eafp.py
import os
import tempfile
import timeit

d = {"a": 1}
def lbyl(k):
    return d[k] if k in d else None
def eafp(k):
    try:
        return d[k]
    except KeyError:
        return None
for name, f in (("LBYL", lbyl), ("EAFP", eafp)):
    hit = min(timeit.repeat(lambda: f("a"), number=200_000, repeat=3))
    miss = min(timeit.repeat(lambda: f("zz"), number=200_000, repeat=3))
    print(f"{name}: key present {hit * 5:.2f} µs/op, key missing {miss * 5:.2f} µs/op")

path = os.path.join(tempfile.gettempdir(), "maybe.txt")
try:                                                    # EAFP for files: no window between check and use
    with open(path) as f:
        print(f.read())
except FileNotFoundError:
    print("no such file - handled without a separate exists() check")
```

```console
$ python3 eafp.py
LBYL: key present 0.05 µs/op, key missing 0.03 µs/op
EAFP: key present 0.03 µs/op, key missing 0.11 µs/op
no such file - handled without a separate exists() check
```

Since 3.11, entering a `try` block costs essentially nothing ("zero-cost" exceptions); raising and catching is the expensive part. So EAFP is fastest when failures are rare, LBYL when they are common — but the correctness argument (no race) usually decides.

<!-- expanded -->
### Going deeper: why EAFP is safer with files

"Look before you leap" (`if os.path.exists(p): open(p)`) has a race: the file can disappear between the check and the open. "Easier to ask forgiveness" (`try: open(p) except FileNotFoundError:`) asks the operating system once, atomically. The same holds for dictionaries in threaded code and for any check-then-act on shared state.

```python
# file: eafp2.py
import os
import threading
import time

path = "volatile.txt"

def deleter():
    for _ in range(2000):
        try:
            open(path, "w").close()
            os.remove(path)
        except FileNotFoundError:
            pass

lbyl_failures = 0
t = threading.Thread(target=deleter)
t.start()
for _ in range(2000):
    if os.path.exists(path):
        try:
            with open(path) as f:
                f.read()
        except FileNotFoundError:
            lbyl_failures += 1        # existed when checked, gone when opened
    time.sleep(0)
t.join()
print("LBYL: check said 'exists' but open failed", "at least once" if lbyl_failures else "zero times this run")
```

```console
$ python3 eafp2.py
LBYL: check said 'exists' but open failed at least once
```

The check was true and the open still failed: the window between them is small but real. Only the `try` around the `open` handles it correctly.

## Unit 6.4 — Exception groups and `except*`

**One-line summary:** When several operations run concurrently (or validation collects every error), more than one exception can occur. `ExceptionGroup("msg", [e1, e2])` bundles them; `except* ValueError:` handles the matching **subset** and lets the rest continue propagating. `asyncio.TaskGroup` (Chapter 11) raises exception groups.

```python
# file: groups.py
def validate(user):
    errors = []
    if not user.get("name"):
        errors.append(ValueError("name is required"))
    if not isinstance(user.get("age"), int):
        errors.append(TypeError("age must be an int"))
    if "@" not in user.get("email", ""):
        errors.append(ValueError("email is invalid"))
    if errors:
        raise ExceptionGroup("invalid user", errors)

try:
    validate({"name": "", "age": "ten", "email": "nope"})
except* ValueError as eg:
    print("value problems:", [str(e) for e in eg.exceptions])
except* TypeError as eg:
    print("type problems:", [str(e) for e in eg.exceptions])
```

```console
$ python3 groups.py
value problems: ['name is required', 'email is invalid']
type problems: ['age must be an int']
```

Both `except*` clauses run — unlike plain `except`, where only the first match runs — because each handles its own part of the group.

<!-- expanded -->
### Going deeper: where exception groups come from

`ExceptionGroup` exists because concurrent code can fail in several places at once. `asyncio.TaskGroup` (Unit 11.4) raises one when more than one task fails, and `except*` handles each kind separately — every matching handler runs, unlike plain `except`, where only the first matching clause runs.

```python
# file: groups2.py
import asyncio

async def fetch(name, fail_with=None):
    await asyncio.sleep(0.01)
    if fail_with:
        raise fail_with(f"{name} failed")
    return name

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(fetch("a"))
            tg.create_task(fetch("b", TimeoutError))
            tg.create_task(fetch("c", ValueError))
            tg.create_task(fetch("d", TimeoutError))
    except* TimeoutError as eg:
        print("timeouts:", sorted(str(e) for e in eg.exceptions))
    except* ValueError as eg:
        print("bad values:", [str(e) for e in eg.exceptions])

asyncio.run(main())
```

```console
$ python3 groups2.py
timeouts: ['b failed', 'd failed']
bad values: ['c failed']
```

## Topic 6B — Context managers

## Unit 6.5 — `with`, `__enter__`/`__exit__`, and `contextlib`

**One-line summary:** `with cm as x:` calls `cm.__enter__()` (its return value becomes `x`), runs the block, then **always** calls `cm.__exit__(exc_type, exc, tb)` — with the exception details if one occurred; returning `True` from `__exit__` suppresses it. `@contextlib.contextmanager` turns a generator with one `yield` into a context manager. `ExitStack` manages a dynamic number of them; `suppress(...)` ignores specific exceptions.

```python
# file: managers.py
import contextlib
import time

class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    def __exit__(self, exc_type, exc, tb):
        self.elapsed = time.perf_counter() - self.start
        print(f"  block took {self.elapsed * 1000:.1f} ms (exception: {exc_type.__name__ if exc_type else None})")
        return False                                   # do not suppress exceptions

with Timer() as t:
    sum(range(1_000_000))
try:
    with Timer():
        raise KeyError("boom")
except KeyError:
    print("KeyError still propagated after __exit__ ran")

@contextlib.contextmanager
def tag(name):
    print(f"<{name}>")
    try:
        yield name.upper()                             # value bound by 'as'
    finally:
        print(f"</{name}>")                            # runs even on errors

with tag("div") as t:
    with tag("p"):
        print("  hello", t)

with contextlib.ExitStack() as stack:                  # a variable number of resources
    names = [stack.enter_context(tag(n)) for n in ("a", "b", "c")]
    print("  inside all three:", names)

with contextlib.suppress(FileNotFoundError):
    open("/definitely/missing.txt")
print("suppress: continued after FileNotFoundError")
```

```console
$ python3 managers.py
  block took 10.9 ms (exception: None)
  block took 0.0 ms (exception: KeyError)
KeyError still propagated after __exit__ ran
<div>
<p>
  hello DIV
</p>
</div>
<a>
<b>
<c>
  inside all three: ['A', 'B', 'C']
</c>
</b>
</a>
suppress: continued after FileNotFoundError
```

### Deep & Tricky

**DT-17. A bare `except:` (or `except BaseException:`) also catches `KeyboardInterrupt` and `SystemExit`, so Ctrl-C and `sys.exit()` stop working inside it; use `except Exception:` for "anything that is an error", because those two (and `GeneratorExit`) inherit from `BaseException` directly, precisely so that error handlers do not catch them.** *(Python documentation, "Exception hierarchy"; PEP 352)* The demo shows a "catch-everything" loop that cannot be stopped by `sys.exit`, next to the correct version.

```python
# file: bare_except.py
import sys

def run(handler_catches_everything):
    for attempt in range(3):
        try:
            sys.exit(f"exit requested on attempt {attempt}")
        except BaseException if handler_catches_everything else Exception as e:
            print(f"  swallowed {type(e).__name__}: {e}")
    return "loop finished - the exit request was ignored"

print("bare except:", run(True))
print("except Exception:")
run(False)
print("never printed")
```

```console
$ python3 bare_except.py
  swallowed SystemExit: exit requested on attempt 0
  swallowed SystemExit: exit requested on attempt 1
  swallowed SystemExit: exit requested on attempt 2
bare except: loop finished - the exit request was ignored
except Exception:
exit requested on attempt 0
```

### Pattern

**P-7. Handle an exception only where you can do something meaningful (retry, fall back, add context, translate it for your caller); otherwise let it propagate. Release resources with `with`, never with manual `close()` calls in every branch.** *(Practice)* Define one base exception per library; chain with `from`; never swallow silently — at minimum log it. *Trade-off:* catching too broadly hides bugs (a `KeyError` from a typo becomes "item not found"); catching too narrowly crashes on recoverable problems. Keep `try` blocks small so the handled exception can only come from the line you expect.

### Mastery check

1. **Predict.** In `try: return 1` / `finally: print("x")`, is `"x"` printed, and what is returned?
2. **Predict.** Does `except (ValueError, TypeError) as e:` catch a `KeyError`?

## Chapter 6 Project — Atomic file writes and a transactional store

**Goal:** two context managers that make operations **all-or-nothing**. (1) `atomic_write(path)`: write to a temporary file in the same directory, `fsync`, then `os.replace` it over the target — readers see either the old file or the complete new one, never a half-written file, even if the writer crashes mid-way. (2) `Store.transaction()`: changes to an in-memory key-value store are applied only if the block finishes; on any exception they are rolled back. Verify both with **fault injection**.

```python
# file: atomic/atomic.py
import contextlib
import os
import random
import tempfile
from pathlib import Path

@contextlib.contextmanager
def atomic_write(path, mode="w"):
    path = Path(path)
    fd, tmp = tempfile.mkstemp(dir=path.parent, prefix=f".{path.name}.", suffix=".tmp")
    try:
        with os.fdopen(fd, mode) as f:
            yield f
            f.flush()
            os.fsync(f.fileno())                        # data is on disk before the rename
        os.replace(tmp, path)                           # atomic on POSIX and Windows
    except BaseException:
        with contextlib.suppress(FileNotFoundError):
            os.unlink(tmp)                              # leave no temporary debris
        raise

class Store:
    def __init__(self):
        self.data = {}
    @contextlib.contextmanager
    def transaction(self):
        snapshot = dict(self.data)                      # shallow copy is enough for immutable values
        try:
            yield self.data
        except BaseException:
            self.data.clear()
            self.data.update(snapshot)                  # roll back
            raise

class InjectedFault(Exception):
    pass

target = Path("settings.txt")
target.write_text("version=1\n" * 3)
rng = random.Random(6)
ok = failed = 0
for trial in range(200):
    lines = [f"version={trial}\n"] * 3
    try:
        with atomic_write(target) as f:
            for line in lines:
                if rng.random() < 0.2:
                    raise InjectedFault("crash while writing")
                f.write(line)
        ok += 1
    except InjectedFault:
        failed += 1
    content = target.read_text().splitlines()
    assert len(content) == 3 and len(set(content)) == 1, f"torn write detected: {content}"
leftovers = [p.name for p in Path(".").glob(".settings.txt.*")]
print(f"atomic_write: {ok} completed, {failed} crashed mid-write, never a torn file; temp files left: {leftovers}")

store = Store()
with store.transaction() as d:
    d["alice"], d["bob"] = 100, 50
committed = rolled_back = 0
for trial in range(1000):
    before = dict(store.data)
    try:
        with store.transaction() as d:
            amount = rng.randint(1, 30)
            src, dst = rng.sample(["alice", "bob"], 2)
            d[src] -= amount                            # debit first...
            if d[src] < 0 or rng.random() < 0.1:
                raise InjectedFault("insufficient funds or crash")
            d[dst] += amount                            # ...then credit
        committed += 1
    except InjectedFault:
        rolled_back += 1
        assert store.data == before, "partial transaction leaked"
    assert sum(store.data.values()) == 150 and min(store.data.values()) >= 0
print(f"transactions: {committed} committed, {rolled_back} rolled back; total money conserved: {sum(store.data.values())}")
target.unlink()
```

```console
$ cd atomic && python3 atomic.py
atomic_write: 90 completed, 110 crashed mid-write, never a torn file; temp files left: []
transactions: 805 committed, 195 rolled back; total money conserved: 150
```

The invariants are checked after **every** trial: the file always has three identical lines (no torn writes), no temporary files remain, money is conserved, and no balance goes negative — even though each line write is crashed with probability 0.2 (so more than half of the file writes fail part-way) and a fifth of the transactions are aborted after the debit.

### Review checklist

- Why must the temporary file be in the **same directory** as the target for `os.replace` to be atomic?
- Why do the cleanup handlers catch `BaseException` but re-raise it?
- The store's rollback copies the dict on every transaction — how would you avoid that for a large store? (Log the changes made inside the block and undo them.)

## Answers to the Chapter 6 prediction questions

**6.5 Q1.** Yes, `"x"` is printed (finally always runs), and the function returns 1 — the `return` value is saved while `finally` executes.

**6.5 Q2.** No. `KeyError` is a subclass of `LookupError`, not of `ValueError` or `TypeError`, so it propagates.

# Chapter 7 — Modules, Packages, Virtual Environments, and Packaging

*Goal: organise code into modules and packages, understand exactly how `import` finds and caches modules, isolate each project's dependencies in a virtual environment, and turn a project into an installable, versioned package with a command-line entry point using `pyproject.toml`.*

**What you will be able to do after this chapter**

- Explain what `import` does: search `sys.path`, execute the module once, cache it in `sys.modules`.
- Write packages with `__init__.py`, absolute and relative imports, and runnable `__main__` modules.
- Recognise and fix circular imports.
- Create virtual environments with `venv` and `uv`, and install dependencies into them.
- Write a `pyproject.toml`, build a wheel, install it, and run its console script.

## Topic 7A — The import system

## Unit 7.1 — Modules, `sys.path`, and `sys.modules`

**One-line summary:** A **module** is a `.py` file (or a compiled extension). `import m` looks up `sys.modules` first (a cache); if absent, it searches the directories in `sys.path` in order, **executes the module's code once** in a new module object, and stores it in `sys.modules`. The module's top-level code therefore runs only on the first import. `if __name__ == "__main__":` guards code that should run only when the file is executed directly.

```python
# file: greetings.py
print(f"  [greetings module body runs, __name__ = {__name__!r}]")
DEFAULT = "hello"

def greet(name, word=DEFAULT):
    return f"{word}, {name}"

if __name__ == "__main__":
    print(greet("from the command line"))
```

```python
# file: importer.py
import sys
print("first import:")
import greetings
print("second import (cached, body does not run again):")
import greetings as g2
print(greetings is g2, "greetings" in sys.modules, greetings.greet("ana"))
from greetings import greet
print(greet("ben", word="hi"))
print("search path starts with the script's directory:", sys.path[0].rsplit("/", 1)[-1])
```

```console
$ python3 greetings.py
  [greetings module body runs, __name__ = '__main__']
hello, from the command line
$ python3 importer.py
first import:
  [greetings module body runs, __name__ = 'greetings']
second import (cached, body does not run again):
True True hello, ana
hi, ben
search path starts with the script's directory: ch07
```

<!-- expanded -->
### Going deeper: `if __name__ == "__main__"` and import-time side effects

Importing a module runs its top-level code once. Code that should run only when the file is executed as a script goes under `if __name__ == "__main__":` — otherwise importing the module (for tests, or by another module) runs it too. Keep top-level code to definitions and cheap constants: slow work at import time slows every program that imports you, and `python -X importtime` shows who is responsible.

```python
# file: tool.py
import time

print(f"tool.py top level runs, __name__ = {__name__!r}")
CONFIG = {"retries": 3}

def main():
    print("main() runs: doing the script's work")

if __name__ == "__main__":
    main()
```

```python
# file: uses_tool.py
import tool
import tool                      # cached in sys.modules: the top level does not run again
print("imported; CONFIG =", tool.CONFIG)
```

```console
$ python3 tool.py && python3 uses_tool.py
tool.py top level runs, __name__ = '__main__'
main() runs: doing the script's work
tool.py top level runs, __name__ = 'tool'
imported; CONFIG = {'retries': 3}
$ python3 -X importtime -c "import json" 2>&1 | tail -1 | sed -E 's/[0-9]+ \|/N |/g'
import time:       N |       N | json
```

The `-X importtime` report lists every module imported, with the time spent in it (self and cumulative, in microseconds); the last line is the module you asked for.

## Unit 7.2 — Packages, relative imports, and `python -m`

**One-line summary:** A **package** is a directory of modules with an `__init__.py` (run when the package is first imported). Inside a package, use **absolute** imports (`from shop.pricing import tax`) or **relative** ones (`from .pricing import tax`). Run a module inside a package with `python -m package.module`, which sets up `sys.path` and the package context correctly; a `__main__.py` makes `python -m package` work.

```python
# file: shop/__init__.py
"""A tiny package."""
from .pricing import total                              # re-export: users can write `from shop import total`
__version__ = "1.0"
```

```python
# file: shop/pricing.py
from .tax import VAT                                    # relative import within the package

def total(net):
    return round(net * (1 + VAT), 2)
```

```python
# file: shop/tax.py
VAT = 0.20
```

```python
# file: shop/__main__.py
import sys
from shop import total, __version__

amount = float(sys.argv[1]) if len(sys.argv) > 1 else 100.0
print(f"shop {__version__}: total for {amount} is {total(amount)}")
```

```console
$ python3 -m shop 59.99
shop 1.0: total for 59.99 is 71.99
$ python3 -c "import shop, shop.pricing; print(shop.total(10), shop.pricing.__name__, shop.__file__.split('/')[-2:])"
12.0 shop.pricing ['shop', '__init__.py']
$ python3 shop/pricing.py
Traceback (most recent call last):
  File "shop/pricing.py", line 2, in <module>
    from .tax import VAT                                    # relative import within the package
    ^^^^^^^^^^^^^^^^^^^^
ImportError: attempted relative import with no known parent package
```

Running `pricing.py` directly fails: as a script it has no parent package, so the relative import has nothing to be relative to. `python -m shop` (or importing) works.

<!-- expanded -->
### Deep & Tricky

**DT-18. Running a module inside a package as a script (`python shop/pricing.py`) puts `shop/` itself on `sys.path` and makes the module `__main__`, so its relative imports fail ("attempted relative import with no known parent package") and the module can be imported a second time under its real name, giving two copies of its globals. Run it as `python -m shop.pricing` from the project root.** *(Guarantee: Python docs, "The import system" and `-m` — `-m` sets `__package__` so relative imports work)*

```python
# file: pkgdemo/app/__init__.py
```

```python
# file: pkgdemo/app/util.py
def double(x):
    return 2 * x
```

```python
# file: pkgdemo/app/cli.py
from .util import double
print("double(21) =", double(21), "| __name__ =", __name__, "| __package__ =", __package__)
```

```console
$ cd pkgdemo && python3 -m app.cli
double(21) = 42 | __name__ = __main__ | __package__ = app
$ cd pkgdemo && python3 app/cli.py 2>&1 | tail -1
ImportError: attempted relative import with no known parent package
```

## Unit 7.3 — Circular imports

**One-line summary:** If module A imports B while B (during its own first execution) imports A, B receives a **partially initialised** A — the names A defines after its `import B` line do not exist yet. Fixes, in order of preference: move the shared code into a third module; import the **module** (`import a`) and access attributes at call time instead of `from a import name` at import time; or move the import inside the function that needs it.

```python
# file: circ/models.py
from circ.services import audit                       # models imports services...

class User:
    def __init__(self, name):
        self.name = name
        audit(f"created {name}")
```

```python
# file: circ/services.py
from circ.models import User                          # ...and services imports models: a cycle

def audit(msg):
    print("audit:", msg)

def make_admin():
    return User("admin")
```

```python
# file: circ/__init__.py
```

```console
$ python3 -c "import circ.models"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    import circ.models
  File "circ/models.py", line 2, in <module>
    from circ.services import audit                       # models imports services...
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "circ/services.py", line 2, in <module>
    from circ.models import User                          # ...and services imports models: a cycle
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
ImportError: cannot import name 'User' from partially initialized module 'circ.models' (most likely due to a circular import) (circ/models.py)
```

```python
# file: circ_fixed/models.py
from circ_fixed import services                       # import the module, not the name

class User:
    def __init__(self, name):
        self.name = name
        services.audit(f"created {name}")             # attribute looked up at call time
```

```python
# file: circ_fixed/services.py
from circ_fixed import models

def audit(msg):
    print("audit:", msg)

def make_admin():
    return models.User("admin")
```

```python
# file: circ_fixed/__init__.py
```

```console
$ python3 -c "from circ_fixed import services; print(services.make_admin().name)"
audit: created admin
admin
```

### Deep & Tricky

**DT-19. `from module import name` copies the binding at import time; with a circular import it fails if `name` is not yet defined, and even without a cycle, later rebinding `module.name` (for example by monkey-patching in tests) does not affect the copied name — while `import module` and `module.name` always look up the current value.** *(Python Language Reference, "The import statement": `from` imports bind names "in the local namespace" to the objects found at import time)* This is why test doubles must patch the name *where it is used* (`unittest.mock.patch("app.views.send_email")`), not where it is defined, and why `import module` is the robust choice inside import cycles.

## Topic 7B — Environments and packaging

## Unit 7.4 — Virtual environments

**One-line summary:** A **virtual environment** is a directory with its own `site-packages` and a `python` that uses it, so each project has its own dependency versions, independent of the system Python and of other projects. Create one with `python -m venv .venv` (standard library) or `uv venv` (much faster); install with `pip install` or `uv pip install`; record exact versions in a lock file (`uv lock`, `pip freeze`).

```console
$ rm -rf demo-env && python3 -m venv demo-env && demo-env/bin/python -c "import sys; print('prefix:', sys.prefix.split('/')[-1], '| base:', sys.base_prefix)"
prefix: demo-env | base: /usr/local
$ demo-env/bin/python -c "import sys; print([p.split('/')[-1] for p in sys.path if 'site-packages' in p])"
['site-packages']
$ demo-env/bin/python -m pip --version | cut -d' ' -f1-2
pip 26.2.2
$ rm -rf uv-env && uv venv uv-env -q && uv pip install --python uv-env/bin/python -q "humanize==4.9.0" && uv-env/bin/python -c "import humanize; print(humanize.naturalsize(123456789), '| installed in', humanize.__file__.split('/')[-6])"
123.5 MB | installed in uv-env
$ python3 -c "import humanize" 2>&1 | tail -1
ModuleNotFoundError: No module named 'humanize'
```

The last line shows the isolation: `humanize` was installed only inside `uv-env`, so the system interpreter cannot import it.

<!-- expanded -->
### Going deeper: what a virtual environment is

A virtual environment is a directory with its own `site-packages` and a `python` that uses it. "Activating" only puts its `bin/` first on `PATH`; you can skip activation and run `venv/bin/python` directly, which is what scripts and services should do.

```console
$ python3 -m venv demo-env && demo-env/bin/python -c "import sys; print(sys.prefix != sys.base_prefix, sys.prefix.endswith('demo-env'))" && ls demo-env && cat demo-env/pyvenv.cfg | grep -E "^(include-system-site-packages|version)"
True True
bin  include  lib  lib64  pyvenv.cfg
include-system-site-packages = false
version = 3.13.18
$ demo-env/bin/python -m pip --version | sed -E 's/from .*demo-env/from demo-env/; s/ \(python.*//'
pip 26.2.2 from demo-env/lib/python3.13/site-packages/pip
```

`sys.prefix != sys.base_prefix` is how code detects it is running in a virtual environment. The `pyvenv.cfg` file points back at the base interpreter; nothing is copied except a few scripts and links, which is why a venv breaks when the base Python is upgraded or moved — recreate it rather than repair it. Tools such as `uv` create and fill environments much faster, but the result is the same layout.

## Unit 7.5 — `pyproject.toml`, wheels, and entry points

**One-line summary:** A modern package is described by **`pyproject.toml`**: the `[build-system]` table names the build backend (setuptools, hatchling, …); `[project]` holds the name, version, Python requirement, dependencies, and `[project.scripts]` **console entry points** that become executables on install. Build a **wheel** (`.whl`, a zip with the files and metadata) with `uv build` or `python -m build`; install it with `pip`/`uv pip`. Use the `src/` layout so tests import the installed package, not the working directory.

```text
# file: wordstat/pyproject.toml
[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.build_meta"

[project]
name = "wordstat"
version = "0.1.0"
description = "Count words in text files"
requires-python = ">=3.10"
dependencies = []

[project.scripts]
wordstat = "wordstat.cli:main"
```

```python
# file: wordstat/src/wordstat/__init__.py
from .core import top_words
__all__ = ["top_words"]
```

```python
# file: wordstat/src/wordstat/core.py
import re
from collections import Counter

WORD = re.compile(r"[A-Za-z']+")

def top_words(text, n=5):
    return Counter(w.lower() for w in WORD.findall(text)).most_common(n)
```

```python
# file: wordstat/src/wordstat/cli.py
import argparse
import sys
from importlib.metadata import version
from .core import top_words

def main(argv=None):
    p = argparse.ArgumentParser(prog="wordstat", description="Show the most common words.")
    p.add_argument("files", nargs="*", help="files to read (default: stdin)")
    p.add_argument("-n", type=int, default=5, help="how many words")
    p.add_argument("--version", action="version", version=f"%(prog)s {version('wordstat')}")
    args = p.parse_args(argv)
    text = "".join(open(f).read() for f in args.files) if args.files else sys.stdin.read()
    for word, count in top_words(text, args.n):
        print(f"{count:>6}  {word}")
    return 0
```

```console
$ cd wordstat && rm -rf dist && uv build -q --wheel && ls dist
wordstat-0.1.0-py3-none-any.whl
$ rm -rf pkg-env && uv venv pkg-env -q && uv pip install --python pkg-env/bin/python -q wordstat/dist/*.whl && ls pkg-env/bin | grep wordstat
wordstat
$ pkg-env/bin/wordstat --version
wordstat 0.1.0
$ echo "the cat and the hat and the bat sat" | pkg-env/bin/wordstat -n 3
     3  the
     2  and
     1  cat
$ unzip -l wordstat/dist/*.whl | awk 'NR>3 {print $4}' | grep -v '^$' | head -8
wordstat/__init__.py
wordstat/cli.py
wordstat/core.py
wordstat-0.1.0.dist-info/METADATA
wordstat-0.1.0.dist-info/WHEEL
wordstat-0.1.0.dist-info/entry_points.txt
wordstat-0.1.0.dist-info/top_level.txt
wordstat-0.1.0.dist-info/RECORD
```

The wheel contains the package's modules plus a `.dist-info` directory with the metadata (`METADATA`, `entry_points.txt`, `RECORD` with file hashes); installing it created the `wordstat` executable from `[project.scripts]`.

### Pattern

**P-8. One virtual environment per project; dependencies declared in `pyproject.toml` and pinned in a lock file; code in a `src/` layout; executables as entry points; and `python -m` (never `python path/to/module.py`) to run modules inside packages.** *(Practice; PyPA packaging guide)* `uv` combines environment creation, installation, locking, and building in one fast tool; the standard `venv` + `pip` + `build` remains the universal fallback. *Trade-off:* strict pinning makes builds reproducible but requires deliberate upgrades; for **libraries**, declare compatible ranges (`>=2,<3`) and leave exact pins to applications.

### Mastery check

1. **Predict.** If `a.py` contains `print("loading a")` and is imported three times by different modules in one program, how many times is the line printed?
2. **Predict.** Why does `python3 shop/pricing.py` fail while `python3 -m shop` works?

## Chapter 7 Project — Ship a tool with tests

**Goal:** extend `wordstat` into a properly structured project: add a pytest test suite that runs against the **installed** package (not the source directory), install it in **editable** mode for development (`uv pip install -e`), and run the tests and the CLI from the environment.

```python
# file: wordstat/tests/test_core.py
import subprocess
import sys

import wordstat
from wordstat.cli import main

def test_counts_case_insensitively():
    assert wordstat.top_words("A a B", 2) == [("a", 2), ("b", 1)]

def test_apostrophes_are_part_of_words():
    assert wordstat.top_words("don't don't do", 1) == [("don't", 2)]

def test_cli_reads_files(tmp_path, capsys):
    f = tmp_path / "t.txt"
    f.write_text("x y x")
    assert main([str(f), "-n", "1"]) == 0
    assert capsys.readouterr().out.split() == ["2", "x"]

def test_module_comes_from_src_layout():
    assert "/src/wordstat/" in wordstat.__file__        # editable install points at src/
```

```console
$ rm -rf dev-env && uv venv dev-env -q && uv pip install --python dev-env/bin/python -q -e ./wordstat pytest
$ cd wordstat && ../dev-env/bin/python -m pytest -q tests 2>&1 | tail -3
....                                                                                         [100%]
4 passed in 0.03s
$ echo "to be or not to be" | dev-env/bin/wordstat -n 2
     2  to
     2  be
```

The editable install links the environment to `wordstat/src`, so edits take effect without reinstalling, while the `src/` layout guarantees that tests import the package through the environment, exactly as users will — a missing file in the package would fail the tests instead of being silently picked up from the working directory.

### Review checklist

- What does `[project.scripts]` generate on installation, and where?
- Why is the `src/` layout safer than putting `wordstat/` next to `tests/`?
- What would you add to publish this to PyPI? (A license, a README, classifiers, then `uv publish` or `twine upload`.)

## Answers to the Chapter 7 prediction questions

**7.5 Q1.** Once: the first import executes the module and caches it in `sys.modules`; later imports return the cached module.

**7.5 Q2.** As a script, `pricing.py` runs as `__main__` with no parent package, so `from .tax import VAT` has no package to be relative to. `python -m shop` imports `shop` as a package, so its modules know their parent.

# Chapter 8 — Type Hints and Static Checking

*Goal: annotate Python code with types that document intent and let a static checker (mypy) find bugs before the code runs — generics, unions and `None`, callables, protocols for duck typing, `TypedDict`, `Literal`, overloads — and know where static types stop and runtime validation must begin.*

**What you will be able to do after this chapter**

- Annotate functions, variables, and classes; run `mypy` and read its errors.
- Use generics (`list[int]`, `dict[str, X]`, type parameters), unions, and `Optional`.
- Describe duck-typed interfaces with `typing.Protocol`.
- Use `TypedDict`, `Literal`, `Final`, `NewType`, and `@overload`.
- Validate untrusted data at run time (pydantic) while keeping static types inside.

## Topic 8A — Annotations and the checker

## Unit 8.1 — Annotations are hints; mypy checks them

**One-line summary:** Annotations (`def f(x: int) -> str:`) are stored in `__annotations__` but **not enforced** by the interpreter: calling `f("oops")` runs anyway. A static checker such as **mypy** reads the annotations and reports inconsistencies without running the code. Gradual typing lets you add types incrementally; `--strict` turns on the strongest checks.

```python
# file: basics.py
def greeting(name: str, times: int = 1) -> str:
    return ", ".join([f"hello {name}"] * times)

print(greeting("ana", 2))
print(greeting.__annotations__)
print(greeting(42))                                   # runs fine: annotations are not enforced
total: int = "not an int"                             # also runs fine
print(total)
```

```console
$ python3 basics.py
hello ana, hello ana
{'name': <class 'str'>, 'times': <class 'int'>, 'return': <class 'str'>}
hello 42
not an int
$ mypy basics.py
basics.py:7: error: Argument 1 to "greeting" has incompatible type "int"; expected "str"  [arg-type]
basics.py:8: error: Incompatible types in assignment (expression has type "str", variable has type "int")  [assignment]
Found 2 errors in 1 file (checked 1 source file)
```

### Deep & Tricky

**DT-20. By default mypy does not type-check the *bodies* of functions that have no annotations at all, so an unannotated helper can contain obvious type errors that mypy silently ignores; `--check-untyped-defs` (or `--strict`) closes the gap.** *(mypy documentation, "Untyped definitions and calls")* This is the "gradual" in gradual typing: unannotated code is treated as dynamically typed (`Any`). A project that "passes mypy" may therefore be checking much less than it seems — the demo shows the same bug reported only with the stricter flag.

```python
# file: untyped.py
def add_tax(price):                                   # no annotations: body not checked by default
    rate = 1.2
    label = rate + " EUR"                             # float + str: certain TypeError at run time
    return price * rate, label

def typed_caller(p: float) -> object:
    return add_tax(p)
```

```console
$ mypy untyped.py
Success: no issues found in 1 source file
$ mypy --check-untyped-defs untyped.py
untyped.py:4: error: Unsupported operand types for + ("float" and "str")  [operator]
Found 1 error in 1 file (checked 1 source file)
```

## Unit 8.2 — Generics, unions, `None`, and callables

**One-line summary:** Containers are parameterised (`list[int]`, `dict[str, list[float]]`, `tuple[int, str]`, `tuple[int, ...]`). `X | None` (or `Optional[X]`) marks values that may be `None`, and mypy forces you to handle the `None` case before use (**narrowing** with `if x is not None`, `isinstance`, early returns). Python 3.12 type-parameter syntax writes generic functions and classes directly: `def first[T](xs: list[T]) -> T`. `Callable[[int, str], bool]` types functions.

```python
# file: generics.py
from collections.abc import Callable, Iterable

def first[T](items: list[T]) -> T | None:             # PEP 695 syntax (3.12+)
    return items[0] if items else None

def apply_all[T, R](f: Callable[[T], R], xs: Iterable[T]) -> list[R]:
    return [f(x) for x in xs]

class Stack[T]:
    def __init__(self) -> None:
        self._items: list[T] = []
    def push(self, item: T) -> None:
        self._items.append(item)
    def pop(self) -> T:
        return self._items.pop()

def shout(name: str | None) -> str:
    if name is None:                                  # narrowing: after this, name is str
        return "NOBODY"
    return name.upper()

s: Stack[int] = Stack()
s.push(3)
n = first([10, 20])
lengths = apply_all(len, ["a", "bcd"])
print(s.pop(), n, lengths, shout(None), shout("ana"))
```

```python
# file: generics_errors.py
from generics import Stack, first

s: Stack[int] = Stack()
s.push("three")                                       # wrong item type
value = first(["a", "b"])
print(value.upper())                                  # value may be None
```

```console
$ mypy generics.py generics_errors.py
generics_errors.py:5: error: Argument 1 to "push" of "Stack" has incompatible type "str"; expected "int"  [arg-type]
generics_errors.py:7: error: Item "None" of "str | None" has no attribute "upper"  [union-attr]
Found 2 errors in 1 file (checked 2 source files)
```

<!-- expanded -->
### Going deeper: type parameters (3.12) and `TypeVar` bounds

Python 3.12 added a direct syntax for generics: `def first[T](xs: list[T]) -> T` and `class Stack[T]:`, replacing explicit `TypeVar` declarations. Bounds (`[T: Comparable]`) and constraints (`[T: (int, float)]`) restrict what `T` can be, and `type` statements declare aliases.

```python
# file: generics312.py
from collections.abc import Callable, Iterable
from typing import Any, Protocol

class SupportsLessThan(Protocol):
    def __lt__(self, other: Any, /) -> bool: ...

def biggest[T: SupportsLessThan](xs: Iterable[T]) -> T:
    it = iter(xs)
    best = next(it)
    for x in it:
        if best < x:
            best = x
    return best

class Stack[T]:
    def __init__(self) -> None:
        self._items: list[T] = []
    def push(self, x: T) -> None:
        self._items.append(x)
    def pop(self) -> T:
        return self._items.pop()

type Handler = Callable[[str], None]

def run(h: Handler) -> None:
    h("event")

s: Stack[int] = Stack()
s.push(1)
print(biggest([3, 9, 4]), biggest(["pear", "apple"]), s.pop())
run(lambda e: print("handled", e))
print(Stack.__type_params__, Handler.__value__)
```

```console
$ python3 generics312.py
9 pear 1
handled event
(T,) collections.abc.Callable[[str], None]
$ python3 -m mypy --strict generics312.py 2>&1 | tail -1
Success: no issues found in 1 source file
```

## Unit 8.3 — Protocols: typed duck typing

**One-line summary:** Python code usually depends on what an object **can do**, not what class it is ("duck typing"). `typing.Protocol` describes that capability — any class with matching methods is accepted, **without inheriting** from the protocol (structural subtyping). Mark it `@runtime_checkable` to allow `isinstance` checks of method presence.

```python
# file: protocols.py
from typing import Protocol, runtime_checkable

@runtime_checkable
class SupportsArea(Protocol):
    def area(self) -> float: ...

class Circle:                                          # note: no inheritance from SupportsArea
    def __init__(self, r: float) -> None:
        self.r = r
    def area(self) -> float:
        return 3.14159 * self.r ** 2

class Square:
    def __init__(self, s: float) -> None:
        self.s = s
    def area(self) -> float:
        return self.s * self.s

class Label:
    text = "no area here"

def total_area(shapes: list[SupportsArea]) -> float:
    return sum(s.area() for s in shapes)

print(total_area([Circle(1), Square(2)]))
print(isinstance(Circle(1), SupportsArea), isinstance(Label(), SupportsArea))
```

```python
# file: protocols_error.py
from protocols import Label, Square, total_area
total_area([Square(1), Label()])
```

```console
$ python3 protocols.py
7.14159
True False
$ mypy protocols.py protocols_error.py
protocols_error.py:3: error: List item 1 has incompatible type "Label"; expected "SupportsArea"  [list-item]
Found 1 error in 1 file (checked 2 source files)
```

<!-- expanded -->
### Going deeper: `runtime_checkable`, and why it is shallow

A `Protocol` is checked by the type checker only. Decorating it with `@runtime_checkable` makes `isinstance` work too — but the run-time check only tests that the *methods exist*, not their signatures or return types.

```python
# file: protocols2.py
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None: ...

class File:
    def close(self) -> None:
        print("file closed")

class Liar:
    close = 42                        # an attribute named close, not a method

def shutdown(*things: object) -> None:
    for t in things:
        if isinstance(t, Closeable):
            print(type(t).__name__, "passes isinstance")

shutdown(File(), Liar(), "text")
```

```console
$ python3 protocols2.py
File passes isinstance
Liar passes isinstance
```

`Liar` passes the `isinstance` check because it has *an attribute* called `close`; calling it would fail. Use run-time protocol checks for dispatch hints, not for validation.

## Unit 8.4 — `TypedDict`, `Literal`, `Final`, `NewType`, `@overload`

**One-line summary:** `TypedDict` types dictionaries with known keys (JSON-like data); `Literal["r", "w"]` restricts values to constants; `Final` marks names that must not be reassigned; `NewType("UserId", int)` creates a distinct type for the checker at zero runtime cost; `@overload` declares several signatures when the return type depends on the arguments.

```python
# file: precise.py
from typing import Final, Literal, NewType, TypedDict, overload

class Movie(TypedDict):
    title: str
    year: int

UserId = NewType("UserId", int)
MAX_RETRIES: Final = 3
Mode = Literal["read", "write"]

def open_db(path: str, mode: Mode) -> str:
    return f"{path} opened for {mode}"

def lookup(uid: UserId) -> str:
    return f"user #{uid}"

@overload
def parse(x: str) -> int: ...
@overload
def parse(x: bytes) -> str: ...
def parse(x: str | bytes) -> int | str:
    return int(x) if isinstance(x, str) else x.decode()

m: Movie = {"title": "Arrival", "year": 2016}
print(open_db("app.db", "read"), lookup(UserId(7)), parse("42") + 1, parse(b"ok").upper(), m["title"])
```

```python
# file: precise_errors.py
from precise import Movie, UserId, lookup, open_db, MAX_RETRIES

m: Movie = {"title": "Arrival", "yeer": 2016}         # typo in a key
open_db("app.db", "delete")                           # not an allowed mode
lookup(7)                                             # plain int is not a UserId
MAX_RETRIES = 5                                       # reassigning a Final
```

```console
$ python3 precise.py
app.db opened for read user #7 43 OK Arrival
$ mypy precise.py precise_errors.py
precise_errors.py:4: error: Missing key "year" for TypedDict "Movie"  [typeddict-item]
precise_errors.py:4: error: Extra key "yeer" for TypedDict "Movie"  [typeddict-unknown-key]
precise_errors.py:5: error: Argument 2 to "open_db" has incompatible type "Literal['delete']"; expected "Literal['read', 'write']"  [arg-type]
precise_errors.py:6: error: Argument 1 to "lookup" has incompatible type "int"; expected "UserId"  [arg-type]
precise_errors.py:7: error: Cannot assign to final name "MAX_RETRIES"  [misc]
Found 5 errors in 1 file (checked 2 source files)
```

### Deep & Tricky

**DT-21. `list[int]` is not a subtype of `list[float]` (even though `int` is accepted where `float` is expected): lists are **invariant**, because a function receiving `list[float]` could append a float into what the caller believes is a list of ints. Read-only types such as `Sequence[float]` are **covariant** and do accept `list[int]`.** *(PEP 484, "Covariance and contravariance"; mypy documentation, "Invariance vs covariance")* The fix is usually to annotate parameters with the least capable abstract type that works — `Sequence`, `Iterable`, `Mapping` from `collections.abc` — which is also better design.

```python
# file: variance.py
from collections.abc import Sequence

def mean_list(xs: list[float]) -> float:
    return sum(xs) / len(xs)

def mean_seq(xs: Sequence[float]) -> float:
    return sum(xs) / len(xs)

ints: list[int] = [1, 2, 3]
print(mean_seq(ints))                                  # fine: Sequence is read-only (covariant)
print(mean_list(ints))                                 # mypy error: list is invariant
```

```console
$ mypy variance.py
variance.py:12: error: Argument 1 to "mean_list" has incompatible type "list[int]"; expected "list[float]"  [arg-type]
variance.py:12: note: "list" is invariant -- see https://mypy.readthedocs.io/en/stable/common_issues.html#variance
variance.py:12: note: Consider using "Sequence" instead, which is covariant
Found 1 error in 1 file (checked 1 source file)
```

<!-- expanded -->
### Going deeper: `@overload`, `TypeGuard`, and `TypeIs`

`@overload` describes a function whose return type depends on its argument types, so the checker knows `parse("1")` returns `int` and `parse(b"1")` returns `bytes`. `TypeGuard` (3.10) and `TypeIs` (3.13) let a function you write narrow a type the way `isinstance` does. They differ in one rule: `TypeIs[T]` requires `T` to be a subtype of the parameter's type, and because `list` is *invariant* (a `list[str]` is not a `list[object]` — you could append an `int` to the latter), `TypeIs[list[str]]` on a `list[object]` parameter is rejected. `TypeGuard` has no such rule.

```python
# file: narrowing.py
from typing import TYPE_CHECKING, TypeGuard, TypeIs, overload

@overload
def double(x: int) -> int: ...
@overload
def double(x: str) -> str: ...
def double(x: int | str) -> int | str:
    return x * 2

def is_str_list(xs: list[object]) -> TypeGuard[list[str]]:   # TypeIs here: mypy error (invariance)
    return all(isinstance(x, str) for x in xs)

def is_int(x: object) -> TypeIs[int]:                         # fine: int is a subtype of object
    return isinstance(x, int)

def shout(xs: list[object]) -> str:
    if is_str_list(xs):
        return " ".join(x.upper() for x in xs)     # the checker knows x is str here
    return "not all strings"

n: int = double(21)
s: str = double("ab")
print(n, s, shout(["a", "b"]), shout(["a", 1]), [v for v in [1, "x", 2] if is_int(v)])
if TYPE_CHECKING:
    reveal_type(double(21))                          # only the type checker sees this
```

```console
$ python3 -m mypy --strict narrowing.py 2>&1 | head -2
narrowing.py:26: note: Revealed type is "int"
Success: no issues found in 1 source file
$ python3 narrowing.py
42 abab A B not all strings [1, 2]
$ sed 's/-> TypeGuard\[list\[str\]\]/-> TypeIs[list[str]]/' narrowing.py > narrowing_typeis.py && python3 -m mypy --strict narrowing_typeis.py 2>&1 | grep -oE "error: .*" | head -1
error: Narrowed type "list[str]" is not a subtype of input type "list[object]"  [narrowed-type-not-subtype]
```

`reveal_type` asks the type checker to print what it inferred; guarding it with `if TYPE_CHECKING:` keeps it out of the running program.

## Topic 8B — Static and runtime

## Unit 8.5 — Validating untrusted data at run time

**One-line summary:** Static types describe what *your* code passes around; they cannot check data arriving from outside (JSON from an API, a config file, user input) — at run time, a `TypedDict` is just a dict. Validate at the **boundary** with a runtime library such as **pydantic**, which parses and converts input into typed objects (or raises detailed errors); inside the boundary, rely on static checking.

```python
# file: boundary.py
import json
from pydantic import BaseModel, Field, ValidationError

class Order(BaseModel):
    id: int
    customer: str = Field(min_length=1)
    quantity: int = Field(gt=0)
    unit_price: float

    @property
    def total(self) -> float:
        return self.quantity * self.unit_price

good = json.loads('{"id": "17", "customer": "ana", "quantity": 3, "unit_price": "2.5"}')
order = Order.model_validate(good)                     # strings are converted where it is safe
print(order, "| total:", order.total, "| id type:", type(order.id).__name__)
bad = json.loads('{"id": "x", "customer": "", "quantity": 0}')
try:
    Order.model_validate(bad)
except ValidationError as e:
    for err in e.errors():
        print(f"  {'.'.join(map(str, err['loc']))}: {err['msg']}")
```

```console
$ python3 boundary.py
id=17 customer='ana' quantity=3 unit_price=2.5 | total: 7.5 | id type: int
  id: Input should be a valid integer, unable to parse string as an integer
  customer: String should have at least 1 character
  quantity: Input should be greater than 0
  unit_price: Field required
```

### Pattern

**P-9. Annotate public functions and class attributes; run mypy in CI with strictness increased over time (`--strict` for new code); use abstract parameter types (`Iterable`, `Sequence`, `Mapping`) and `Protocol`s for duck-typed interfaces; validate external data at the boundary with a runtime validator.** *(Practice)* Types are documentation the checker verifies — they pay off most in large codebases and refactorings. *Trade-off:* typing dynamic patterns (decorators, metaprogramming, heterogeneous dicts) can be laborious; use `Any` or `cast` sparingly at those seams, and prefer simpler designs that type naturally.

### Mastery check

1. **Predict.** Does `def f(x: int) -> int: return x` raise an error when called as `f("a")` at run time?
2. **Predict.** Will mypy accept passing a `dict[str, int]` to a parameter annotated `Mapping[str, float]`?

## Chapter 8 Project — Let the checker find the bugs

**Goal:** an inventory module written first **without** types contains three bugs that tests did not catch. Add annotations, run `mypy --strict`, and let the checker point at each bug; then fix them and show the strict run passing and the program producing correct output.

```python
# file: inventory/inventory_untyped.py
def load(rows):
    stock = {}
    for name, qty in rows:
        stock[name] = qty                              # bug 1: qty stays a string ("3")
    return stock

def restock(stock, name, amount):
    stock[name] = stock.get(name) + amount             # bug 2: get() may return None

def report(stock):
    lines = []
    for name, qty in sorted(stock.items()):
        lines.append(name + ": " + qty)                # bug 3 (masked by bug 1): concatenates qty
    return "\n".join(lines)

stock = load([("apple", "3"), ("pear", "5")])
restock(stock, "apple", "2")
print(report(stock))
```

```python
# file: inventory/inventory_typed.py
from collections.abc import Iterable

Stock = dict[str, int]

def load(rows: Iterable[tuple[str, str]]) -> Stock:
    stock: Stock = {}
    for name, qty in rows:
        stock[name] = qty
    return stock

def restock(stock: Stock, name: str, amount: int) -> None:
    stock[name] = stock.get(name) + amount

def report(stock: Stock) -> str:
    lines = []
    for name, qty in sorted(stock.items()):
        lines.append(name + ": " + qty)
    return "\n".join(lines)
```

```python
# file: inventory/inventory_fixed.py
from collections.abc import Iterable

Stock = dict[str, int]

def load(rows: Iterable[tuple[str, str]]) -> Stock:
    stock: Stock = {}
    for name, qty in rows:
        stock[name] = int(qty)                         # fix 1: parse at the boundary
    return stock

def restock(stock: Stock, name: str, amount: int) -> None:
    stock[name] = stock.get(name, 0) + amount          # fix 2: default for new items

def report(stock: Stock) -> str:
    lines = []
    for name, qty in sorted(stock.items()):
        lines.append(f"{name}: {qty}")                 # fix 3: format, do not concatenate
    return "\n".join(lines)

if __name__ == "__main__":
    stock = load([("apple", "3"), ("pear", "5")])
    restock(stock, "apple", 2)
    restock(stock, "kiwi", 4)
    print(report(stock))
```

```console
$ cd inventory && python3 inventory_untyped.py
apple: 32
pear: 5
$ cd inventory && mypy --strict inventory_typed.py
inventory_typed.py:9: error: Incompatible types in assignment (expression has type "str", target has type "int")  [assignment]
inventory_typed.py:13: error: Unsupported operand types for + ("None" and "int")  [operator]
inventory_typed.py:13: note: Left operand is of type "int | None"
inventory_typed.py:18: error: Unsupported operand types for + ("str" and "int")  [operator]
Found 3 errors in 1 file (checked 1 source file)
$ cd inventory && mypy --strict inventory_fixed.py && python3 inventory_fixed.py
Success: no issues found in 1 source file
apple: 5
kiwi: 4
pear: 5
```

The untyped version **runs** and prints plausible-looking but wrong output — `"3" + "2"` is `"32"`, string concatenation instead of addition — the most dangerous kind of bug. Annotating the intent (`Stock = dict[str, int]`) is enough for mypy to report every place where the code contradicts it.

### Review checklist

- Which of the three mypy errors corresponds to the bug that produced "32"?
- Why is `int(qty)` placed in `load` rather than in `report`?
- What would `reveal_type(stock.get(name))` show, and why does mypy insist on handling it?

## Answers to the Chapter 8 prediction questions

**8.5 Q1.** No. Annotations are not enforced; `f("a")` returns `"a"`. Only a static checker (or a runtime validator) would object.

**8.5 Q2.** Yes. `Mapping` is read-only, and covariant in its value type, so `dict[str, int]` is accepted where `Mapping[str, float]` is expected (int is compatible with float). A parameter typed `dict[str, float]` would be rejected (DT-21).

# Chapter 9 — Files, Text, Bytes, and Serialization

*Goal: move data reliably between programs and storage — understand the difference between text (`str`) and bytes, encode and decode with explicit encodings, read and write files with `pathlib`, use CSV and JSON correctly, pack binary records with `struct`, and know why `pickle` must never read untrusted data.*

**What you will be able to do after this chapter**

- Convert between `str` and `bytes` with explicit encodings; handle decoding errors; normalise Unicode.
- Read and write text and binary files, line by line or in chunks, with `pathlib`.
- Read and write CSV and JSON, including types JSON does not support.
- Pack and unpack binary data with `struct`, choosing the byte order.
- Explain the security risk of `pickle` and choose safer formats.

## Topic 9A — Text and bytes

## Unit 9.1 — `str`, `bytes`, and encodings

**One-line summary:** `str` holds Unicode code points; `bytes` holds raw 8-bit values. **Encoding** converts text to bytes (`s.encode("utf-8")`), **decoding** converts back (`b.decode("utf-8")`). UTF-8 uses 1–4 bytes per character and is the default almost everywhere; decoding bytes with the wrong encoding gives an error or **mojibake** (garbled text). Always state the encoding when opening text files.

```python
# file: encodings.py
s = "café ☕ 🐍"
for enc in ("utf-8", "utf-16-le", "latin-1"):
    try:
        b = s.encode(enc)
        print(f"{enc:>9}: {len(b):>2} bytes {b[:12]!r}...")
    except UnicodeEncodeError as e:
        print(f"{enc:>9}: cannot encode {e.object[e.start]!r} ({e.reason})")
raw = "café".encode("utf-8")
print("decoded with latin-1 (mojibake):", raw.decode("latin-1"))
try:
    b"\xff\xfeabc".decode("utf-8")
except UnicodeDecodeError as e:
    print("UnicodeDecodeError:", e)
print("errors='replace':", b"ok \xff bad".decode("utf-8", errors="replace"))
print("bytes are integers:", raw[0], list(raw), bytes([99, 97, 102, 195, 169]).decode())
```

```console
$ python3 encodings.py
    utf-8: 14 bytes b'caf\xc3\xa9 \xe2\x98\x95 \xf0\x9f'...
utf-16-le: 18 bytes b'c\x00a\x00f\x00\xe9\x00 \x00\x15&'...
  latin-1: cannot encode '☕' (ordinal not in range(256))
decoded with latin-1 (mojibake): cafÃ©
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 0: invalid start byte
errors='replace': ok � bad
bytes are integers: 99 [99, 97, 102, 195, 169] café
```

### Deep & Tricky

**DT-22. Two strings that look identical can differ: "é" can be one code point (U+00E9) or two ("e" + combining accent U+0301), so `==`, `len`, dictionary lookups, and set membership treat them as different until both are normalised with `unicodedata.normalize("NFC", s)`.** *(Unicode Standard Annex #15, "Unicode Normalization Forms"; Python documentation, `unicodedata.normalize`)* Text typed on different systems (macOS file names are stored decomposed, for example) or pasted from documents mixes the forms. Normalise at the boundary — when reading input or keys — before comparing, hashing, or counting characters; `casefold()` (not `lower()`) is the corresponding tool for case-insensitive comparison.

```python
# file: normalize.py
import unicodedata
a = "café"                 # é as one code point
b = "café"                # e + combining acute accent
print(a, b, a == b, len(a), len(b))
nfc = lambda s: unicodedata.normalize("NFC", s)
print("after NFC:", nfc(a) == nfc(b), len(nfc(b)))
print("Straße".lower() == "STRASSE".lower(), "Straße".casefold() == "STRASSE".casefold())
```

```console
$ python3 normalize.py
café café False 4 5
after NFC: True 4
False True
```

## Unit 9.2 — Files and `pathlib`

**One-line summary:** `pathlib.Path` represents paths with operators (`base / "data" / "x.csv"`) and methods (`exists`, `glob`, `read_text`, `write_bytes`, `stat`, `mkdir`). `open(path, mode, encoding=...)`: `"r"`/`"w"`/`"a"`/`"x"` for text, add `"b"` for bytes. Iterate over a text file to read it **line by line** (constant memory); read binary files in **chunks**. Text mode translates line endings (`newline=` controls it).

```python
# file: files.py
import hashlib
from pathlib import Path

base = Path("demo_dir")
base.mkdir(exist_ok=True)
notes = base / "notes.txt"
notes.write_text("first line\nsecond line\nthird line\n", encoding="utf-8")
with notes.open("a", encoding="utf-8") as f:
    f.write("appended line\n")
with notes.open(encoding="utf-8") as f:
    for no, line in enumerate(f, 1):                    # streaming, one line at a time
        print(no, line.rstrip("\n"))

big = base / "blob.bin"
big.write_bytes(bytes(range(256)) * 4096)              # 1 MiB
h = hashlib.sha256()
with big.open("rb") as f:
    while chunk := f.read(64 * 1024):                  # walrus: read until b""
        h.update(chunk)
print(f"{big.name}: {big.stat().st_size:,} bytes, sha256 {h.hexdigest()[:16]}...")
print(sorted(p.name for p in base.glob("*")), notes.suffix, notes.stem, notes.parent, notes.resolve().is_absolute())
try:
    with notes.open("x") as f:                         # "x": create, fail if it exists
        pass
except FileExistsError as e:
    print("FileExistsError:", e.filename)
for p in base.iterdir():
    p.unlink()
base.rmdir()
```

```console
$ python3 files.py
1 first line
2 second line
3 third line
4 appended line
blob.bin: 1,048,576 bytes, sha256 fbbab289f7f94b25...
['blob.bin', 'notes.txt'] .txt notes demo_dir True
FileExistsError: demo_dir/notes.txt
```

<!-- expanded -->
### Going deeper: atomic writes and temporary files

Writing a file in place (`open(p, "w")`) truncates it first; a crash halfway leaves a half-written file, and a reader can see it half-written. The safe pattern: write to a temporary file in the *same directory*, flush and `fsync` it, then `os.replace` it over the target — a rename, which is atomic on POSIX and Windows.

```python
# file: atomic.py
import json
import os
import tempfile
from pathlib import Path

def write_atomic(path: Path, data: str) -> None:
    fd, tmp = tempfile.mkstemp(dir=path.parent, prefix=path.name + ".", suffix=".tmp")
    try:
        with os.fdopen(fd, "w", encoding="utf-8") as f:
            f.write(data)
            f.flush()
            os.fsync(f.fileno())               # the data is on disk before the rename
        os.replace(tmp, path)                  # atomic: readers see old or new, never half
    except BaseException:
        os.unlink(tmp)
        raise

cfg = Path("settings.json")
write_atomic(cfg, json.dumps({"theme": "dark", "size": 14}))
print(cfg.read_text(), sorted(p.name for p in Path(".").glob("settings.json*")))

with tempfile.TemporaryDirectory() as d:
    p = Path(d) / "scratch.txt"
    p.write_text("temporary")
    print("inside:", p.exists(), p.read_text())
print("after the with block:", p.exists())
print(Path("data/raw/2026/file.csv").parts, Path("report.tar.gz").suffixes, Path("a/b/c.txt").with_suffix(".md"))
```

```console
$ python3 atomic.py
{"theme": "dark", "size": 14} ['settings.json']
inside: True temporary
after the with block: False
('data', 'raw', '2026', 'file.csv') ['.tar', '.gz'] a/b/c.md
```

No `.tmp` file is left behind, and at no moment did `settings.json` contain partial JSON.

## Topic 9B — Formats

## Unit 9.3 — CSV and JSON

**One-line summary:** Use the `csv` module (never `line.split(",")`): it handles quoting, embedded commas, and newlines; open CSV files with `newline=""`. `csv.DictReader`/`DictWriter` map rows to dicts. JSON maps to dicts, lists, strings, numbers, booleans, and `null` — **not** dates, `Decimal`, sets, or tuples (which become lists); convert them explicitly with `default=` when writing and an `object_hook` (or explicit parsing) when reading.

```python
# file: formats.py
import csv
import io
import json
from datetime import date
from decimal import Decimal

rows = [{"name": "Ana", "city": "Pune, MH", "note": 'says "hi"'}, {"name": "Ben", "city": "Delhi", "note": "line1\nline2"}]
buf = io.StringIO(newline="")
w = csv.DictWriter(buf, fieldnames=["name", "city", "note"])
w.writeheader(); w.writerows(rows)
text = buf.getvalue()
print(text)
print("naive split breaks:", text.splitlines()[1].split(","))
print("csv module round-trips:", list(csv.DictReader(io.StringIO(text, newline=""))) == rows)

def encode(obj):
    if isinstance(obj, (date, Decimal)):
        return {"__type__": type(obj).__name__, "value": str(obj)}
    raise TypeError(f"not JSON serializable: {type(obj).__name__}")

def decode(d):
    match d:
        case {"__type__": "date", "value": v}:
            return date.fromisoformat(v)
        case {"__type__": "Decimal", "value": v}:
            return Decimal(v)
    return d

invoice = {"id": 7, "issued": date(2026, 11, 15), "amount": Decimal("19.99"), "tags": ("a", "b")}
s = json.dumps(invoice, default=encode, indent=None)
print(s)
back = json.loads(s, object_hook=decode)
print(back, "| tuple came back as", type(back["tags"]).__name__)
try:
    json.dumps({"when": date.today()})
except TypeError as e:
    print("TypeError:", e)
```

```console
$ python3 formats.py
name,city,note
Ana,"Pune, MH","says ""hi"""
Ben,Delhi,"line1
line2"

naive split breaks: ['Ana', '"Pune', ' MH"', '"says ""hi"""']
csv module round-trips: True
{"id": 7, "issued": {"__type__": "date", "value": "2026-11-15"}, "amount": {"__type__": "Decimal", "value": "19.99"}, "tags": ["a", "b"]}
{'id': 7, 'issued': datetime.date(2026, 11, 15), 'amount': Decimal('19.99'), 'tags': ['a', 'b']} | tuple came back as list
TypeError: Object of type date is not JSON serializable
```

<!-- expanded -->
### Going deeper: CSV dialects, and JSON for your own types

`csv` handles quoting, embedded commas, and embedded newlines — never split CSV lines with `str.split(",")`. Open CSV files with `newline=""`, as the documentation requires, so that newlines inside quoted fields survive. `json` handles only dicts, lists, strings, numbers, booleans, and `None`; for dates, `Decimal`, or dataclasses, supply a `default=` function when writing and convert explicitly when reading.

```python
# file: formats2.py
import csv
import io
import json
from dataclasses import dataclass, asdict
from datetime import date
from decimal import Decimal

text = 'name,comment,amount\nada,"says ""hi"", then leaves",10.50\nbob,"two\nlines",3\n'
rows = list(csv.DictReader(io.StringIO(text, newline="")))
for r in rows:
    print(r)
print("naive split gets", len(text.splitlines()[1].split(",")), "fields on line 2 instead of 3")

@dataclass
class Invoice:
    number: int
    issued: date
    total: Decimal

def encode(o):
    if isinstance(o, date):
        return o.isoformat()
    if isinstance(o, Decimal):
        return str(o)                             # keep exact decimal digits
    raise TypeError(f"cannot serialise {type(o).__name__}")

inv = Invoice(7, date(2026, 11, 16), Decimal("1999.90"))
s = json.dumps(asdict(inv), default=encode)
print(s)
back = json.loads(s, parse_float=Decimal)
print(Invoice(back["number"], date.fromisoformat(back["issued"]), Decimal(back["total"])) == inv)
try:
    json.dumps({"when": date.today()})
except TypeError as e:
    print("TypeError:", e)
print(json.dumps({"price": 0.1 + 0.2}), json.loads("1e400"), json.dumps(float("nan")))
```

```console
$ python3 formats2.py
{'name': 'ada', 'comment': 'says "hi", then leaves', 'amount': '10.50'}
{'name': 'bob', 'comment': 'two\nlines', 'amount': '3'}
naive split gets 4 fields on line 2 instead of 3
{"number": 7, "issued": "2026-11-16", "total": "1999.90"}
True
TypeError: Object of type date is not JSON serializable
{"price": 0.30000000000000004} inf NaN
```

The last line shows three JSON surprises: floats are written with full precision (`0.30000000000000004`), a too-large number becomes `inf` silently, and Python writes `NaN` — which is not valid JSON and is rejected by many other parsers (`json.dumps(..., allow_nan=False)` raises instead).

## Unit 9.4 — Binary data with `struct`

**One-line summary:** `struct.pack(fmt, ...)` converts values to bytes according to a format string — byte order (`<` little-endian, `>` big-endian/network, `=` native) followed by type codes (`i` int32, `q` int64, `d` float64, `H` uint16, `10s` 10 bytes, …); `unpack` reverses it. Fixed-size records allow random access by offset. `memoryview` slices bytes without copying.

```python
# file: binary.py
import struct

record = struct.Struct("<I10sd")                       # uint32 id, 10-byte name, float64 price
print("record size:", record.size)
packed = record.pack(42, "widget".encode(), 3.5)
print(packed.hex(" "))
rid, name, price = record.unpack(packed)
print(rid, name.rstrip(b"\0").decode(), price)
print("big-endian 1:", struct.pack(">I", 1).hex(), "| little-endian 1:", struct.pack("<I", 1).hex())
data = b"".join(record.pack(i, f"item{i}".encode(), i * 1.5) for i in range(5))
view = memoryview(data)
third = record.unpack(view[2 * record.size:3 * record.size])   # random access, no copy of data
print("third record:", third[0], third[1].rstrip(b"\0").decode(), third[2])
try:
    struct.pack("<H", 70000)
except struct.error as e:
    print("struct.error:", e)
```

```console
$ python3 binary.py
record size: 22
2a 00 00 00 77 69 64 67 65 74 00 00 00 00 00 00 00 00 00 00 0c 40
42 widget 3.5
big-endian 1: 00000001 | little-endian 1: 01000000
third record: 2 item2 3.0
struct.error: 'H' format requires 0 <= number <= 65535
```

<!-- expanded -->
### Going deeper: byte order and alignment in `struct`

`struct` format strings start with a byte-order character: `<` little-endian, `>` or `!` big-endian (network order), `=` native order with no padding, and `@` (the default) native order *with* C alignment padding. Forgetting it means your file format changes between machines and gains invisible padding bytes.

```python
# file: struct2.py
import struct

print(struct.calcsize("@BI"), struct.calcsize("<BI"))        # native alignment pads after the byte
print(struct.pack("<I", 1).hex(), struct.pack(">I", 1).hex())

header = struct.Struct("<4sHHI")                              # magic, version, flags, length
blob = header.pack(b"BOOK", 2, 0b101, 1234) + b"payload..."
magic, version, flags, length = header.unpack_from(blob)
print(magic, version, bin(flags), length, header.size)
for rec in struct.iter_unpack("<hh", bytes([1, 0, 255, 255, 0, 1, 2, 0])):
    print(rec)
print(int.from_bytes(b"\x01\x00", "little"), (258).to_bytes(2, "big"))
```

```console
$ python3 struct2.py
8 5
01000000 00000001
b'BOOK' 2 0b101 1234 12
(1, -1)
(256, 2)
1 b'\x01\x02'
```

## Unit 9.5 — `pickle`: convenient and dangerous

**One-line summary:** `pickle` serialises almost any Python object graph (including classes and shared references) — but **unpickling executes code**: a pickle can instruct the loader to call any importable function with any arguments. Never unpickle data from an untrusted source; use JSON, CSV, or a schema-based format (protobuf, msgpack with plain types) for data exchange, and pickle only for trusted, internal, short-lived data.

```python
# file: pickle_danger.py
import pickle

class Payload:
    def __reduce__(self):                                # tells pickle how to "rebuild" this object...
        import os
        return (os.system, ("echo '  !!! arbitrary command executed during pickle.loads'",))

evil = pickle.dumps(Payload())
print("pickle bytes look harmless:", evil[:40], "...")
pickle.loads(evil)                                       # ...and loading runs the command
safe = pickle.loads(pickle.dumps({"a": [1, 2], "b": (3, 4)}))
print("ordinary round trip:", safe)
```

```console
$ python3 pickle_danger.py
pickle bytes look harmless: b"\x80\x04\x95V\x00\x00\x00\x00\x00\x00\x00\x8c\x05posix\x94\x8c\x06system\x94\x93\x94\x8c;echo '  " ...
  !!! arbitrary command executed during pickle.loads
ordinary round trip: {'a': [1, 2], 'b': (3, 4)}
```

### Deep & Tricky

**DT-23. `pickle.loads` on attacker-controlled bytes is remote code execution: the format includes opcodes that import a callable and call it (`__reduce__`), so no "safe" subset exists in the standard `pickle` module; the documentation's warning is literal.** *(Python documentation, `pickle`: "Warning: The pickle module is not secure. Only unpickle data you trust.")* The same applies to formats built on pickle — `shelve`, many ML model files (`.pkl`, older PyTorch `.pt` loaded with `weights_only=False`), `multiprocessing` messages from untrusted peers. Signing pickles with HMAC (`hmac.compare_digest`) protects integrity if both ends share a secret, but the simplest defence is not to use pickle across trust boundaries.

### Pattern

**P-10. Decode bytes to text at the input boundary, work with `str` inside, encode at the output boundary — always naming the encoding (UTF-8). Use the format's library (csv, json, struct), never hand-written splitting; choose human-readable formats for interchange and binary fixed-size records only when you need speed or random access.** *(Practice: "the Unicode sandwich")* *Trade-off:* JSON is portable and safe but verbose and lossy for types like dates and decimals; binary formats are compact and fast but need a documented schema, byte order, and versioning.

### Mastery check

1. **Predict.** How many bytes does `"€".encode("utf-8")` have, and how many does `"€".encode("utf-16-le")` have?
2. **Predict.** What does `json.loads(json.dumps({1: "a"}))` return?

## Chapter 9 Project — A binary record store with an index

**Goal:** convert a 100,000-row CSV of products into a **binary file of fixed-size records** with a header (magic bytes, version, record count), then answer "find product by id" with **binary search directly on the file** (`seek` to `offset = header + i × record_size`), compare file sizes and lookup speed with scanning the CSV, and verify that a round trip CSV → binary → JSON preserves every row.

```python
# file: recstore/recstore.py
import csv
import json
import random
import struct
import time
from pathlib import Path

HEADER = struct.Struct("<4sHI")                        # magic, version, count
RECORD = struct.Struct("<I24sdI")                      # id, name (24 bytes UTF-8), price, stock
MAGIC, VERSION = b"PRDB", 1

def make_csv(path, n, rng):
    ids = sorted(rng.sample(range(1, 10 * n), n))
    with path.open("w", newline="", encoding="utf-8") as f:
        w = csv.writer(f)
        w.writerow(["id", "name", "price", "stock"])
        for i in ids:
            w.writerow([i, f"Produkt-{i}-é", f"{rng.uniform(1, 500):.2f}", rng.randint(0, 999)])
    return ids

def csv_to_bin(src, dst):
    with src.open(newline="", encoding="utf-8") as f:
        rows = list(csv.DictReader(f))
    with dst.open("wb") as out:
        out.write(HEADER.pack(MAGIC, VERSION, len(rows)))
        for r in rows:                                  # rows are sorted by id: enables binary search
            name = r["name"].encode("utf-8")
            if len(name) > 24:
                raise ValueError(f"name too long for the format: {r['name']!r}")
            out.write(RECORD.pack(int(r["id"]), name, float(r["price"]), int(r["stock"])))
    return len(rows)

def decode(rec):
    rid, name, price, stock = RECORD.unpack(rec)
    return {"id": rid, "name": name.rstrip(b"\0").decode("utf-8"), "price": price, "stock": stock}

def find(f, count, target):                             # binary search with seek
    lo, hi = 0, count - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        f.seek(HEADER.size + mid * RECORD.size)
        rec = f.read(RECORD.size)
        rid = struct.unpack_from("<I", rec)[0]
        if rid == target:
            return decode(rec)
        lo, hi = (mid + 1, hi) if rid < target else (lo, mid - 1)
    return None

def find_in_csv(path, target):
    with path.open(newline="", encoding="utf-8") as f:
        for r in csv.DictReader(f):
            if int(r["id"]) == target:
                return r
    return None

rng = random.Random(9)
src, dst = Path("products.csv"), Path("products.bin")
ids = make_csv(src, 100_000, rng)
n = csv_to_bin(src, dst)
print(f"{n:,} rows: CSV {src.stat().st_size:,} bytes, binary {dst.stat().st_size:,} bytes "
      f"(header {HEADER.size} + {n:,} x {RECORD.size})")

with dst.open("rb") as f:
    magic, version, count = HEADER.unpack(f.read(HEADER.size))
    assert magic == MAGIC and version == VERSION and count == n
    probes = rng.sample(ids, 200) + [0, 10**9]        # 200 hits and 2 misses
    t = time.perf_counter()
    found = [find(f, count, p) for p in probes]
    tb = time.perf_counter() - t
print(f"binary search on file: 202 lookups in {tb * 1000:.1f} ms, {sum(r is not None for r in found)} found")
t = time.perf_counter()
csv_found = [find_in_csv(src, p) for p in probes[:20]]
tc = (time.perf_counter() - t) / 20 * 202
print(f"scanning the CSV: ~{tc * 1000:,.0f} ms for the same 202 lookups (estimated from 20)")
print("example:", found[0])

with dst.open("rb") as f:                              # round trip: binary -> JSON -> compare with CSV
    f.seek(HEADER.size)
    from_bin = [decode(f.read(RECORD.size)) for _ in range(count)]
from_json = json.loads(json.dumps(from_bin))
with src.open(newline="", encoding="utf-8") as f:
    from_csv = [{"id": int(r["id"]), "name": r["name"], "price": float(r["price"]), "stock": int(r["stock"])} for r in csv.DictReader(f)]
print("round trip CSV -> binary -> JSON preserves all rows:", from_json == from_csv)
src.unlink(); dst.unlink()
```

```console
$ cd recstore && python3 recstore.py
100,000 rows: CSV 3,645,060 bytes, binary 4,000,010 bytes (header 10 + 100,000 x 40)
binary search on file: 202 lookups in 2.9 ms, 200 found
scanning the CSV: ~9,436 ms for the same 202 lookups (estimated from 20)
example: {'id': 829391, 'name': 'Produkt-829391-é', 'price': 392.34, 'stock': 975}
round trip CSV -> binary -> JSON preserves all rows: True
```

The binary file is not even smaller here — padding every name to 24 bytes wastes more space than the CSV's separators — but it allows O(log n) lookups with about 17 `seek`+`read` calls each, because every record has the same size and the ids are sorted: thousands of times faster than scanning the CSV. The price: a fixed schema (24 bytes of name — the converter refuses longer names rather than truncating silently), an explicit byte order, and a version number in the header so the format can evolve.

### Review checklist

- Why does the header store a version, and what would a reader do with an unknown version?
- Why is the name limit checked in **bytes** after encoding rather than in characters?
- How would you support variable-length names while keeping random access? (An offset table, or a separate string heap.)

## Answers to the Chapter 9 prediction questions

**9.5 Q1.** "€" (U+20AC) needs 3 bytes in UTF-8 and 2 bytes in UTF-16 (it is in the Basic Multilingual Plane).

**9.5 Q2.** `{'1': 'a'}`: JSON object keys are always strings, so the integer key comes back as the string `"1"`.

# Chapter 10 — Testing, Debugging, and Profiling

*Goal: prove that code works with automated tests (pytest fixtures, parametrisation, mocks, and property-based tests with Hypothesis), find bugs efficiently with the debugger and logging, and find performance problems by measuring (cProfile, timeit, tracemalloc) instead of guessing.*

**What you will be able to do after this chapter**

- Write pytest tests with fixtures, `parametrize`, `tmp_path`, and `pytest.raises`.
- Replace slow or external dependencies with mocks and `monkeypatch`, patching the right name.
- Write property-based tests that generate inputs and shrink failures.
- Debug with `breakpoint()`/`pdb` and structured logging.
- Profile CPU time with cProfile and memory with tracemalloc; micro-benchmark with `timeit`.

## Topic 10A — Tests

## Unit 10.1 — pytest essentials

**One-line summary:** pytest discovers `test_*.py` files and `test_*` functions; a plain `assert` is enough — pytest rewrites it to show the compared values on failure. **Fixtures** (functions decorated with `@pytest.fixture`, requested by parameter name) provide setup and teardown; `@pytest.mark.parametrize` runs one test over many inputs; built-in fixtures such as `tmp_path`, `capsys`, and `monkeypatch` cover common needs; `pytest.raises` checks exceptions.

```python
# file: shopping/cart.py
class Cart:
    def __init__(self, prices):
        self.prices = prices
        self.items = {}
    def add(self, sku, qty=1):
        if sku not in self.prices:
            raise KeyError(f"unknown sku {sku!r}")
        if qty <= 0:
            raise ValueError("quantity must be positive")
        self.items[sku] = self.items.get(sku, 0) + qty
    def total(self, discount=0.0):
        gross = sum(self.prices[s] * q for s, q in self.items.items())
        return round(gross * (1 - discount), 2)
    def save(self, path):
        path.write_text("\n".join(f"{s},{q}" for s, q in sorted(self.items.items())))
```

```python
# file: shopping/test_cart.py
import pytest
from cart import Cart

@pytest.fixture
def cart():                                            # fresh object for every test
    return Cart({"apple": 0.5, "pear": 0.75, "melon": 3.0})

def test_empty_total(cart):
    assert cart.total() == 0

@pytest.mark.parametrize("items, discount, expected", [
    ({"apple": 2}, 0.0, 1.0),
    ({"apple": 2, "melon": 1}, 0.0, 4.0),
    ({"pear": 4}, 0.1, 2.7),
])
def test_totals(cart, items, discount, expected):
    for sku, q in items.items():
        cart.add(sku, q)
    assert cart.total(discount) == expected

def test_unknown_sku(cart):
    with pytest.raises(KeyError, match="kiwi"):
        cart.add("kiwi")

def test_bad_quantity(cart):
    with pytest.raises(ValueError):
        cart.add("apple", 0)

def test_save(cart, tmp_path):                         # tmp_path: a fresh temporary directory
    cart.add("pear"); cart.add("apple", 3)
    f = tmp_path / "cart.csv"
    cart.save(f)
    assert f.read_text().splitlines() == ["apple,3", "pear,1"]

def test_deliberately_failing(cart):                   # shows pytest's assertion introspection
    cart.add("melon", 2)
    assert cart.total() == 5.0
```

```console
$ cd shopping && python3 -m pytest -q -p no:cacheprovider 2>&1 | sed -n '1,25p'
.......F                                                                                     [100%]
============================================= FAILURES =============================================
____________________________________ test_deliberately_failing _____________________________________

cart = <cart.Cart object at 0x7f86abf92be0>

    def test_deliberately_failing(cart):                   # shows pytest's assertion introspection
        cart.add("melon", 2)
>       assert cart.total() == 5.0
E       assert 6.0 == 5.0
E        +  where 6.0 = total()
E        +    where total = <cart.Cart object at 0x7f86abf92be0>.total

test_cart.py:38: AssertionError
===================================== short test summary info ======================================
FAILED test_cart.py::test_deliberately_failing - assert 6.0 == 5.0
1 failed, 7 passed in 0.11s
```

The failure report shows the values on both sides of `==` (`6.0 == 5.0`) and the source line — no `assertEqual` boilerplate needed. One test failing does not stop the others: 7 passed, 1 failed.

<!-- expanded -->
### Going deeper: fixtures with teardown, parametrisation, and `tmp_path`

A pytest **fixture** is a function that prepares something a test needs; a test asks for it by naming it as a parameter. A fixture that `yield`s runs the code after the `yield` as teardown, even when the test fails. `@pytest.mark.parametrize` runs one test function over a table of inputs, each reported separately. Built-in fixtures such as `tmp_path` (a fresh temporary directory) and `capsys` (captured output) cover common needs.

```python
# file: fixtures/inventory.py
import json
from pathlib import Path

class Inventory:
    def __init__(self, path: Path):
        self.path = path
        self.items = json.loads(path.read_text()) if path.exists() else {}
    def add(self, name: str, qty: int) -> None:
        if qty <= 0:
            raise ValueError("qty must be positive")
        self.items[name] = self.items.get(name, 0) + qty
    def save(self) -> None:
        self.path.write_text(json.dumps(self.items, sort_keys=True))
```

```python
# file: fixtures/test_inventory.py
import pytest
from inventory import Inventory

@pytest.fixture
def inv(tmp_path):
    path = tmp_path / "inv.json"
    path.write_text('{"pen": 2}')
    inventory = Inventory(path)
    yield inventory                       # the test runs here
    print(f"\nteardown: file still exists = {path.exists()}")

def test_loads_existing(inv):
    assert inv.items == {"pen": 2}

def test_save_round_trip(inv):
    inv.add("ink", 3)
    inv.save()
    assert Inventory(inv.path).items == {"ink": 3, "pen": 2}

@pytest.mark.parametrize("qty", [0, -1, -100])
def test_rejects_non_positive(inv, qty):
    with pytest.raises(ValueError, match="positive"):
        inv.add("pad", qty)

@pytest.mark.parametrize(("start", "add", "expected"), [(0, 1, 1), (2, 3, 5)], ids=["new", "existing"])
def test_add_cases(tmp_path, start, add, expected):
    inv = Inventory(tmp_path / "x.json")
    if start:
        inv.add("pen", start)
    inv.add("pen", add)
    assert inv.items["pen"] == expected
```

```console
$ cd fixtures && python3 -m pytest -q -p no:cacheprovider 2>&1 | tail -1
7 passed in 0.09s
$ cd fixtures && python3 -m pytest -v -p no:cacheprovider 2>&1 | grep -oE "test_[a-z_]+(\[[^]]+\])? PASSED" 
test_loads_existing PASSED
test_save_round_trip PASSED
test_rejects_non_positive[0] PASSED
test_rejects_non_positive[-1] PASSED
test_rejects_non_positive[-100] PASSED
test_add_cases[new] PASSED
test_add_cases[existing] PASSED
```

The verbose listing shows each parametrised case as its own test, named by its parameters or by the `ids` you gave — so a failure report says exactly which input broke.

## Unit 10.2 — Mocks and monkeypatching

**One-line summary:** Replace dependencies you do not want to call in a unit test — network, clock, randomness, email — with **test doubles**. `monkeypatch.setattr(module, "name", fake)` swaps an attribute for the duration of a test; `unittest.mock.Mock`/`patch` create objects that record how they were called. Patch the name **where it is looked up** (DT-19), not where it was defined.

```python
# file: weather/service.py
import time
import urllib.request

def fetch_temperature(city):
    with urllib.request.urlopen(f"https://example.invalid/weather?city={city}") as r:   # never called in tests
        return float(r.read())

def advice(city):
    t = fetch_temperature(city)
    hour = time.localtime().tm_hour
    if t < 5:
        return "wear a coat"
    return "take sunglasses" if 10 <= hour < 17 and t > 25 else "dress normally"
```

```python
# file: weather/test_service.py
from unittest import mock
import service

def test_cold(monkeypatch):
    monkeypatch.setattr(service, "fetch_temperature", lambda city: 2.0)
    assert service.advice("Oslo") == "wear a coat"

def test_sunny_afternoon(monkeypatch):
    monkeypatch.setattr(service, "fetch_temperature", lambda city: 31.0)
    fake_time = mock.Mock()
    fake_time.localtime.return_value = mock.Mock(tm_hour=14)
    monkeypatch.setattr(service, "time", fake_time)      # replace the module as seen by service
    assert service.advice("Jaipur") == "take sunglasses"
    fake_time.localtime.assert_called_once()

def test_records_calls():
    with mock.patch("service.fetch_temperature", return_value=20.0) as fake:
        assert service.advice("Pune") == "dress normally"
    fake.assert_called_once_with("Pune")
```

```console
$ cd weather && python3 -m pytest -q -p no:cacheprovider 2>&1 | tail -2
...                                                                                          [100%]
3 passed in 0.09s
```

<!-- expanded -->
### Going deeper: patch where the name is *looked up*

`unittest.mock.patch("module.name")` replaces an attribute on a module for the duration of a test. The classic mistake is patching where a function is *defined* instead of where it is *used*: if `service.py` does `from clock import now`, it has its own reference called `service.now`, and patching `clock.now` changes nothing that `service` sees.

```python
# file: patching/clock.py
import datetime

def now() -> datetime.datetime:
    return datetime.datetime.now()
```

```python
# file: patching/greeter.py
from clock import now                   # greeter now holds its own reference

def greeting() -> str:
    return "good morning" if now().hour < 12 else "good afternoon"
```

```python
# file: patching/test_greeter.py
import datetime
from unittest.mock import patch
import greeter

MORNING = datetime.datetime(2026, 11, 16, 9, 0)
EVENING = datetime.datetime(2026, 11, 16, 18, 0)

def test_wrong_target_has_no_effect():
    with patch("clock.now", return_value=EVENING):
        with patch("greeter.now", return_value=MORNING):
            assert greeter.greeting() == "good morning"     # only greeter.now matters

def test_right_target():
    with patch("greeter.now", return_value=EVENING) as fake:
        assert greeter.greeting() == "good afternoon"
        fake.assert_called_once_with()
```

```console
$ cd patching && python3 -m pytest -q -p no:cacheprovider 2>&1 | tail -1
2 passed in 0.09s
```

A design that avoids patching entirely is often better: pass the clock in (`def greeting(now=datetime.datetime.now)`), and the test passes a lambda.

## Unit 10.3 — Property-based testing with Hypothesis

**One-line summary:** Instead of hand-picking examples, state a **property** that must hold for all inputs ("decoding an encoding returns the original"; "sorting is idempotent and preserves elements") and let **Hypothesis** generate hundreds of inputs — including nasty edge cases — and **shrink** any failure to a minimal counterexample.

```python
# file: props/rle.py
def encode(s):
    """Run-length encoding: 'aaab' -> [('a', 3), ('b', 1)]."""
    out = []
    for ch in s:
        if out and out[-1][0] == ch:
            out[-1] = (ch, out[-1][1] + 1)
        else:
            out.append((ch, 1))
    return out

def decode(pairs):
    return "".join(ch * n for ch, n in pairs)

def encode_buggy(s):                                     # "optimised": skips single characters' counts
    return [(ch, n) for ch, n in encode(s) if n > 1] or [(s[:1], 1)] if s else []
```

```python
# file: props/test_rle.py
from hypothesis import given, strategies as st
from rle import decode, encode, encode_buggy

@given(st.text())
def test_roundtrip(s):
    assert decode(encode(s)) == s

@given(st.text())
def test_no_adjacent_equal_runs(s):
    runs = encode(s)
    assert all(a[0] != b[0] for a, b in zip(runs, runs[1:]))

@given(st.text(alphabet="ab"))
def test_buggy_roundtrip(s):
    assert decode(encode_buggy(s)) == s
```

```console
$ cd props && python3 -m pytest -q -p no:cacheprovider 2>&1 | grep -E -A2 "Failing test case" | sed 's/^E *//'
Failing test case: test_buggy_roundtrip(
s='ab',
)
$ cd props && python3 -m pytest -q -p no:cacheprovider 2>&1 | tail -1
1 failed, 2 passed in 0.16s
```

Hypothesis found the bug and shrank the input to a tiny string — the same idea as the stress-testing harness of the *Data Structures and Algorithms* book, packaged as a library with smarter generation and a database of past failures.

### Deep & Tricky

**DT-24. `assert (condition, "message")` is always true — a non-empty tuple is truthy — so the test can never fail; and `assert` statements are removed entirely when Python runs with `-O`, so they must never be used for input validation in production code.** *(Python Language Reference, "The assert statement": `assert expr1, expr2` is equivalent to `if __debug__: if not expr1: raise AssertionError(expr2)`, and `__debug__` is False under `-O`)* CPython emits a `SyntaxWarning` ("assertion is always true, perhaps remove parentheses?") for the tuple form. In tests, plain `assert x == y` is right; in application code, raise explicit exceptions (`ValueError`, `TypeError`).

```python
# file: assert_traps.py
def withdraw(balance, amount):
    assert amount <= balance, "insufficient funds"     # WRONG place for validation
    return balance - amount

if __name__ == "__main__":
    try:
        withdraw(10, 50)
    except AssertionError as e:
        print("normal run: AssertionError:", e)
```

```console
$ python3 assert_traps.py
normal run: AssertionError: insufficient funds
$ python3 -O -c "import assert_traps; print('with -O, withdraw(10, 50) =', assert_traps.withdraw(10, 50))"
with -O, withdraw(10, 50) = -40
$ python3 -c "assert (1 == 2, 'never fails')" 2>&1 | grep -o "SyntaxWarning.*"
SyntaxWarning: assertion is always true, perhaps remove parentheses?
```

## Topic 10B — Debugging and profiling

## Unit 10.4 — The debugger and logging

**One-line summary:** `breakpoint()` (or `python -m pdb script.py`) stops execution and opens **pdb**: `n` next line, `s` step into, `c` continue, `p expr` print, `l` list source, `w` stack, `u`/`d` move up/down frames, `q` quit; `python -m pdb -c continue` stops at the point of an unhandled exception (post-mortem). For running systems, use **`logging`** with levels and context instead of `print`.

```python
# file: buggy_avg.py
def average(values):
    total = 0
    for v in values:
        total += v
    return total / len(values)

def report(groups):
    return {name: average(vals) for name, vals in groups.items()}

print(report({"a": [1, 2, 3], "b": []}))
```

```console
$ printf '%s\n' 'p values' 'up' 'p groups' 'q' | python3 -m pdb -c continue buggy_avg.py 2>&1 | grep -E "^ZeroDivision|^\(Pdb\)|^> |^->" | sed 's|/.*/ch10/||'
ZeroDivisionError: division by zero
> buggy_avg.py(6)average()
-> return total / len(values)
(Pdb) []
(Pdb) > buggy_avg.py(9)report()
-> return {name: average(vals) for name, vals in groups.items()}
(Pdb) {'a': [1, 2, 3], 'b': []}
(Pdb) 
```

The post-mortem session stops inside `average` at the failing line; `p values` shows `[]`; `up` moves to the calling frame (`report`), where `p groups` shows that group `"b"` is empty — the culprit, found without adding a single print statement.

```python
# file: logging_demo.py
import logging

logging.basicConfig(level=logging.INFO, format="%(levelname)-7s %(name)s: %(message)s")
log = logging.getLogger("orders")

def process(order):
    log.debug("processing %s", order)                  # below INFO: not shown
    if order["qty"] <= 0:
        log.warning("rejecting order %s: qty=%d", order["id"], order["qty"])
        return False
    log.info("accepted order %s", order["id"])
    return True

for o in ({"id": 1, "qty": 2}, {"id": 2, "qty": 0}):
    process(o)
try:
    1 / 0
except ZeroDivisionError:
    log.exception("unexpected failure")                # logs the traceback too
```

```console
$ python3 logging_demo.py 2>&1 | head -8
INFO    orders: accepted order 1
WARNING orders: rejecting order 2: qty=0
ERROR   orders: unexpected failure
Traceback (most recent call last):
  File "logging_demo.py", line 18, in <module>
    1 / 0
    ~~^~~
ZeroDivisionError: division by zero
```

<!-- expanded -->
### Going deeper: structured logging configuration

Libraries should create loggers with `logging.getLogger(__name__)` and never configure handlers; applications configure logging once, at start-up. `logging.config.dictConfig` sets levels per logger name, so you can turn one noisy module down without touching the rest. Use `%`-style arguments (`log.info("user %s", uid)`) rather than f-strings: the message is formatted only if the record is actually emitted.

```python
# file: logconfig.py
import logging
import logging.config

logging.config.dictConfig({
    "version": 1,
    "formatters": {"brief": {"format": "%(levelname)-7s %(name)s: %(message)s"}},
    "handlers": {"console": {"class": "logging.StreamHandler", "formatter": "brief", "stream": "ext://sys.stdout"}},
    "loggers": {"app.db": {"level": "WARNING"}},            # quiet one subsystem
    "root": {"level": "DEBUG", "handlers": ["console"]},
})

log = logging.getLogger("app.web")
db = logging.getLogger("app.db")

class Expensive:
    calls = 0
    def __str__(self):
        Expensive.calls += 1
        return "expensive repr"

log.debug("request %s took %d ms", "/home", 42)
db.debug("query plan: %s", Expensive())                     # filtered: never formatted
db.warning("slow query: %s", "SELECT ...")
try:
    1 / 0
except ZeroDivisionError:
    log.exception("handler failed")                         # includes the traceback
print("Expensive.__str__ calls:", Expensive.calls)
```

```console
$ python3 logconfig.py 2>&1 | grep -v '^  ' | grep -v '^Traceback\|^    \|^ZeroDivision'
DEBUG   app.web: request /home took 42 ms
WARNING app.db: slow query: SELECT ...
ERROR   app.web: handler failed
Expensive.__str__ calls: 0
```

`Expensive.__str__` was never called: the `app.db` logger dropped the debug record before formatting it.

## Unit 10.5 — Profiling: measure, do not guess

**One-line summary:** **cProfile** records how many times each function is called and how much time it takes (total and cumulative); sort by `cumulative` to find the expensive call trees, by `tottime` to find hot functions. **`timeit`** measures small snippets reliably (many repetitions, take the minimum). **`tracemalloc`** shows which lines allocate memory.

```python
# file: profiling.py
import cProfile
import pstats
import io

def slow_unique(words):
    seen = []
    for w in words:
        if w not in seen:                              # O(n) membership in a list
            seen.append(w)
    return seen

def normalise(words):
    return [w.strip().lower() for w in words]

def pipeline(n):
    words = [f" Word{i % 3000} " for i in range(n)]
    return len(slow_unique(normalise(words)))

pr = cProfile.Profile()
pr.enable()
pipeline(60_000)
pr.disable()
out = io.StringIO()
pstats.Stats(pr, stream=out).sort_stats("cumulative").print_stats(6)
print("\n".join(l for l in out.getvalue().splitlines() if l.strip())[:1400])
```

```console
$ python3 profiling.py
         123005 function calls in 0.512 seconds
   Ordered by: cumulative time
   List reduced from 8 to 6 due to restriction <6>
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.006    0.006    0.512    0.512 profiling.py:16(pipeline)
        1    0.479    0.479    0.480    0.480 profiling.py:6(slow_unique)
        1    0.014    0.014    0.026    0.026 profiling.py:13(normalise)
    60000    0.007    0.000    0.007    0.000 {method 'lower' of 'str' objects}
    60000    0.006    0.000    0.006    0.000 {method 'strip' of 'str' objects}
     3000    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
```

The profile points straight at `slow_unique` — almost all of the time is its own (`tottime`), spent in list membership tests. Replacing the list by a set (or `dict.fromkeys(words)`) fixes it; this chapter's project does exactly that, with tests guarding the behaviour.

### Pattern

**P-11. Test behaviour, not implementation: fixtures for setup, parametrisation for tables of cases, property-based tests for invariants, mocks only at real boundaries (network, clock, randomness). When something is slow, profile first; when something is wrong, reproduce it in a failing test before fixing it.** *(Practice)* A failing test that reproduces a bug is the best debugging tool — it becomes a regression test once fixed. *Trade-off:* heavy mocking couples tests to the implementation (they break on refactoring while the behaviour is unchanged); prefer fakes with real behaviour (an in-memory store) over call-recording mocks where possible.

### Mastery check

1. **Predict.** In Unit 10.2, why does `monkeypatch.setattr(service, "time", fake_time)` work, while patching `time.localtime` globally would also affect pytest itself?
2. **Predict.** What does `assert []` do?

## Chapter 10 Project — Profile, fix, and prove it

**Goal:** a log-deduplication tool is too slow. (1) Write tests (example-based and property-based) that pin down its behaviour: keep the first occurrence of each normalised line, in order. (2) Profile it and find the hot spot. (3) Fix it, run the tests again, and measure the speed-up — showing that the optimised version behaves identically.

```python
# file: dedupe/dedupe.py
import re

def normalise(line):
    line = line.strip().lower()
    return re.sub(r"\s+", " ", line)                   # recompiles via the cache on every call

def dedupe_slow(lines):
    seen, out = [], []
    for line in lines:
        key = normalise(line)
        if key not in seen:                            # O(n) per line
            seen.append(key)
            out.append(line)
    return out

SPACES = re.compile(r"\s+")

def dedupe_fast(lines):
    seen, out = set(), []
    for line in lines:
        key = SPACES.sub(" ", line.strip().lower())
        if key not in seen:                            # O(1) average
            seen.add(key)
            out.append(line)
    return out
```

```python
# file: dedupe/test_dedupe.py
import pytest
from hypothesis import given, strategies as st
from dedupe import dedupe_fast, dedupe_slow

@pytest.mark.parametrize("impl", [dedupe_slow, dedupe_fast])
@pytest.mark.parametrize("lines, expected", [
    ([], []),
    (["a", "A ", " a"], ["a"]),
    (["x  y", "x y", "X\tY", "z"], ["x  y", "z"]),
    (["b", "a", "b"], ["b", "a"]),
])
def test_examples(impl, lines, expected):
    assert impl(lines) == expected

@given(st.lists(st.text(alphabet="ab \t", max_size=6), max_size=30))
def test_fast_matches_slow(lines):
    assert dedupe_fast(lines) == dedupe_slow(lines)
```

```python
# file: dedupe/bench.py
import cProfile
import io
import pstats
import random
import time
from dedupe import dedupe_fast, dedupe_slow

rng = random.Random(10)
lines = [f"  Event {rng.randint(0, 5000)}   from   host{rng.randint(0, 3)} " for _ in range(30_000)]
pr = cProfile.Profile()
pr.enable(); dedupe_slow(lines); pr.disable()
s = io.StringIO()
pstats.Stats(pr, stream=s).sort_stats("tottime").print_stats(3)
print("\n".join(l for l in s.getvalue().splitlines() if "function calls" in l or "dedupe" in l or "normalise" in l or "sub" in l))
for f in (dedupe_slow, dedupe_fast):
    t = time.perf_counter()
    out = f(lines)
    print(f"{f.__name__}: {len(out):,} unique of {len(lines):,} in {(time.perf_counter() - t) * 1000:,.0f} ms")
```

```console
$ cd dedupe && python3 -m pytest -q -p no:cacheprovider 2>&1 | tail -1
9 passed in 0.24s
$ cd dedupe && python3 bench.py
         241114 function calls in 1.656 seconds
        1    1.575    1.575    1.656    1.656 dedupe/dedupe.py:8(dedupe_slow)
    30000    0.022    0.000    0.022    0.000 {method 'sub' of 're.Pattern' objects}
    30000    0.018    0.000    0.077    0.000 dedupe/dedupe.py:4(normalise)
dedupe_slow: 15,556 unique of 30,000 in 1,580 ms
dedupe_fast: 15,556 unique of 30,000 in 17 ms
```

The tests run both implementations against the same examples and let Hypothesis compare them on random inputs, so the optimisation is protected by more than the handful of cases written by hand. The profile shows where the time went before any code was changed — which is the order that keeps optimisation honest.

### Review checklist

- Which profile column (`tottime` or `cumtime`) identifies `dedupe_slow` itself as the hot spot?
- Why does the property test compare against the slow version instead of restating the specification?
- What other inputs should the example table include (Unicode whitespace? empty strings?)

## Answers to the Chapter 10 prediction questions

**10.5 Q1.** `service.py` looks up `time` in its own module namespace each call, so replacing `service.time` affects only that module. Patching `time.localtime` on the real `time` module would change it for every module in the process, including pytest's own timing code.

**10.5 Q2.** It raises `AssertionError`: an empty list is falsy.

# Chapter 11 — Concurrency: Threads, the GIL, Processes, and asyncio

*Goal: choose the right concurrency model for the work — threads for blocking I/O, processes for CPU-bound computation, `asyncio` for many simultaneous network operations — understand what the global interpreter lock (GIL) does and does not protect, and write concurrent code that is correct, cancellable, and measurably faster.*

**What you will be able to do after this chapter**

- Explain the GIL, why it limits CPU-bound threads, and what free-threaded builds change.
- Use `concurrent.futures` thread and process pools, and measure their effect.
- Protect shared state with locks and prefer queues and immutable messages.
- Write `asyncio` programs with `async`/`await`, `gather`, `TaskGroup`, timeouts, and semaphores.
- Avoid blocking the event loop, and run blocking code from async code safely.

## Topic 11A — Threads and processes

## Unit 11.1 — The GIL: threads for I/O, not for CPU

**One-line summary:** In standard CPython, the **global interpreter lock** lets only one thread execute Python bytecode at a time. Threads still give real speed-ups for **I/O-bound** work, because a thread waiting on the network, disk, or `time.sleep` releases the GIL; they give **no** speed-up (often a slow-down) for pure-Python **CPU-bound** work.

```python
# file: gil.py
import os
import time
from concurrent.futures import ThreadPoolExecutor

def io_task(_):
    time.sleep(0.1)                                    # stands in for a network call: releases the GIL

def cpu_task(_):
    return sum(i * i for i in range(1_500_000))       # pure Python: holds the GIL

print("CPUs:", os.cpu_count())
for name, task, n in (("I/O-bound", io_task, 8), ("CPU-bound", cpu_task, 4)):
    t = time.perf_counter()
    for i in range(n):
        task(i)
    seq = time.perf_counter() - t
    t = time.perf_counter()
    with ThreadPoolExecutor(max_workers=n) as pool:
        list(pool.map(task, range(n)))
    thr = time.perf_counter() - t
    print(f"{name}: {n} tasks sequential {seq:.2f} s, {n} threads {thr:.2f} s  (speed-up {seq / thr:.1f}x)")
```

```console
$ python3 gil.py
CPUs: 2
I/O-bound: 8 tasks sequential 0.80 s, 8 threads 0.10 s  (speed-up 7.9x)
CPU-bound: 4 tasks sequential 0.22 s, 4 threads 0.21 s  (speed-up 1.0x)
```

### Deep & Tricky

**DT-25. The GIL makes individual bytecode instructions atomic, not your operations — threads still need locks for compound updates — and since Python 3.13 an optional **free-threaded** build (`python3.13t`, PEP 703) removes the GIL entirely, so CPU-bound threads can run in parallel and code that "worked thanks to the GIL" may break.** *(PEP 703; Python 3.13 "What's New": experimental free-threaded CPython; `sys._is_gil_enabled()`)* Check the build with `sys._is_gil_enabled()` (this book's interpreter reports `True`). Write thread-safe code as if no GIL existed — locks for shared mutable state, or no shared mutable state at all — and it will be correct on both builds.

## Unit 11.2 — Shared state: locks, queues, and races

**One-line summary:** When threads share mutable data, a "read–modify–write" sequence can interleave and lose updates. Protect it with a `threading.Lock` (`with lock:`), or avoid sharing: pass work and results through a thread-safe `queue.Queue`, or let `concurrent.futures` return results instead of writing to shared variables.

```python
# file: shared.py
import sys
import threading
from concurrent.futures import ThreadPoolExecutor

sys.setswitchinterval(1e-6)                            # switch threads very often to expose races

def pause():
    pass

class Counter:
    def __init__(self):
        self.value, self.lock = 0, threading.Lock()
    def unsafe_add(self, n):
        for _ in range(n):
            v = self.value
            pause()                                     # a Python call: a point where CPython may switch threads
            self.value = v + 1
    def safe_add(self, n):
        for _ in range(n):
            with self.lock:
                self.value += 1

for method in ("unsafe_add", "safe_add"):
    c = Counter()
    threads = [threading.Thread(target=getattr(c, method), args=(100_000,)) for _ in range(4)]
    for t in threads: t.start()
    for t in threads: t.join()
    print(f"{method}: expected 400000, got {c.value}")

def work(chunk):                                       # no shared state: return results instead
    return sum(chunk)
chunks = [range(i, i + 250_000) for i in range(0, 1_000_000, 250_000)]
with ThreadPoolExecutor(4) as pool:
    print("results via futures:", sum(pool.map(work, chunks)) == sum(range(1_000_000)))
```

```console
$ python3 shared.py
unsafe_add: expected 400000, got 168367
safe_add: expected 400000, got 400000
results via futures: True
```

<!-- expanded -->
### Going deeper: `+=` on a shared integer is not atomic

`counter += 1` compiles to several bytecode instructions (load, add, store), and a thread switch can happen between them, so two threads can both read the same old value. Whether you *see* lost updates depends on the interpreter version and on luck; the code is wrong either way. A `Lock` or a thread-safe structure fixes it.

```python
# file: notatomic.py
import dis
import threading
import time

counter = 0
lock = threading.Lock()

def bump_unsafe(n):
    global counter
    for _ in range(n):
        v = counter          # read
        time.sleep(0)        # give up the CPU here: another thread may run
        counter = v + 1      # write: overwrites whatever that thread wrote

def bump_safe(n):
    global counter
    for _ in range(n):
        with lock:
            counter += 1

for fn in (bump_unsafe, bump_safe):
    counter = 0
    ts = [threading.Thread(target=fn, args=(20_000,)) for _ in range(4)]
    for t in ts: t.start()
    for t in ts: t.join()
    print(f"{fn.__name__}: {counter} of 80000", "(updates lost)" if counter < 80_000 else "")

ops = [i.opname for i in dis.get_instructions("counter += 1")]
print("counter += 1 is", len([o for o in ops if o not in ("RESUME", "RETURN_CONST")]), "bytecode instructions:", ops[1:4])
```

```console
$ python3 notatomic.py
bump_unsafe: 20000 of 80000 (updates lost)
bump_safe: 80000 of 80000 
counter += 1 is 4 bytecode instructions: ['LOAD_NAME', 'LOAD_CONST', 'BINARY_OP']
```

The read-modify-write is spelled out in `bump_unsafe`, with a `time.sleep(0)` that invites a thread switch, to make the window wide enough to see on every run; `counter += 1` has the same window, only narrower — without the `sleep`, this program happened to lose no updates on the book's machine, which is exactly why such bugs survive testing. With the free-threaded build of Python 3.13+ (no GIL), such races become far more frequent.

## Unit 11.3 — Processes for CPU-bound work

**One-line summary:** `ProcessPoolExecutor` (or `multiprocessing`) runs tasks in separate **processes**, each with its own interpreter and GIL — true parallelism on multiple cores. The cost: arguments and results are **pickled** and sent between processes, and starting workers takes time, so give each task enough work and little data. Guard the entry point with `if __name__ == "__main__":` (required on platforms that start workers with "spawn").

```python
# file: processes.py
import os
import time
from concurrent.futures import ProcessPoolExecutor

def count_primes(bounds):
    lo, hi = bounds
    count = 0
    for n in range(max(lo, 2), hi):
        if all(n % d for d in range(2, int(n ** 0.5) + 1)):
            count += 1
    return count

if __name__ == "__main__":
    N, workers = 300_000, os.cpu_count()
    chunks = [(i, i + N // 8) for i in range(0, N, N // 8)]
    t = time.perf_counter()
    seq = sum(map(count_primes, chunks))
    ts = time.perf_counter() - t
    t = time.perf_counter()
    with ProcessPoolExecutor(max_workers=workers) as pool:
        par = sum(pool.map(count_primes, chunks))
    tp = time.perf_counter() - t
    print(f"primes below {N:,}: {seq} sequential in {ts:.2f} s; {par} with {workers} processes in {tp:.2f} s "
          f"(speed-up {ts / tp:.1f}x on {workers} cores)")

    big = list(range(2_000_000))
    t = time.perf_counter()
    with ProcessPoolExecutor(max_workers=workers) as pool:
        lengths = list(pool.map(len, [big] * 8))       # tiny work, huge arguments: pickling dominates
    print(f"sending a 2M-element list to workers 8 times for len(): {time.perf_counter() - t:.2f} s (len itself: microseconds)")
```

```console
$ python3 processes.py
primes below 300,000: 25997 sequential in 0.42 s; 25997 with 2 processes in 0.23 s (speed-up 1.8x on 2 cores)
sending a 2M-element list to workers 8 times for len(): 0.50 s (len itself: microseconds)
```

The speed-up is bounded by the number of cores (two on the machine that ran this book); the second experiment shows the opposite case — work so small that moving the data costs far more than computing on it.

<!-- expanded -->
### Going deeper: `concurrent.futures` — one interface for threads and processes

`ThreadPoolExecutor` and `ProcessPoolExecutor` share one API: `submit` returns a `Future`, `map` maps a function over inputs, and `as_completed` yields futures as they finish. Switching between threads (for I/O) and processes (for CPU work) is a one-word change.

```python
# file: executors.py
import time
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor, as_completed

def cpu_work(n):
    return sum(i * i for i in range(n))

def io_work(name):
    time.sleep(0.2)                         # stands in for a network call
    return name

if __name__ == "__main__":
    t0 = time.perf_counter()
    with ThreadPoolExecutor(max_workers=8) as pool:
        futures = [pool.submit(io_work, f"url{i}") for i in range(8)]
        done = [f.result() for f in as_completed(futures)]
    print(f"8 I/O tasks with threads: {len(done)} done in about {round(time.perf_counter() - t0, 1)} s")

    times = {}
    for Executor in (ThreadPoolExecutor, ProcessPoolExecutor):
        t0 = time.perf_counter()
        with Executor(max_workers=2) as pool:
            results = list(pool.map(cpu_work, [3_000_000] * 4))
        times[Executor.__name__] = time.perf_counter() - t0
    ratio = times["ThreadPoolExecutor"] / times["ProcessPoolExecutor"]
    print("4 CPU tasks: processes faster than threads:", ratio > 1.3)
```

```console
$ python3 executors.py
8 I/O tasks with threads: 8 done in about 0.2 s
4 CPU tasks: processes faster than threads: True
```

The `if __name__ == "__main__":` guard is required with process pools: on macOS and Windows (and with the `spawn` start method generally) each worker process imports the main module, and without the guard every worker would start its own pool.

## Topic 11B — asyncio

## Unit 11.4 — `async`/`await`, tasks, gather, and TaskGroup

**One-line summary:** `async def` defines a **coroutine function**; `await` suspends the coroutine until the awaited operation completes, letting the **event loop** run other coroutines in the same thread meanwhile. `asyncio.run(main())` starts the loop. Run coroutines concurrently with `asyncio.gather(...)` or, better, `async with asyncio.TaskGroup() as tg: tg.create_task(...)` (3.11+), which cancels the siblings if one fails. Bound waiting with `asyncio.timeout(...)` and concurrency with `asyncio.Semaphore`.

```python
# file: async_basics.py
import asyncio
import time

async def fetch(name, delay):
    await asyncio.sleep(delay)                        # non-blocking wait: other tasks run meanwhile
    return f"{name} after {delay}s"

async def failing():
    await asyncio.sleep(0.05)
    raise ValueError("service B is down")

async def main():
    t = time.perf_counter()
    results = await asyncio.gather(fetch("A", 0.3), fetch("B", 0.2), fetch("C", 0.1))
    print(results, f"in {time.perf_counter() - t:.2f} s (not 0.6)")

    try:
        async with asyncio.TaskGroup() as tg:
            slow = tg.create_task(fetch("slow", 5))
            tg.create_task(failing())
    except* ValueError as eg:
        print("TaskGroup:", [str(e) for e in eg.exceptions], "| sibling cancelled:", slow.cancelled())

    try:
        async with asyncio.timeout(0.1):
            await fetch("too slow", 1)
    except TimeoutError:
        print("timed out after 0.1 s")

    limit = asyncio.Semaphore(2)
    async def limited(i):
        async with limit:                              # at most 2 at a time
            await asyncio.sleep(0.1)
            return i
    t = time.perf_counter()
    await asyncio.gather(*(limited(i) for i in range(6)))
    print(f"6 tasks, 2 at a time: {time.perf_counter() - t:.2f} s")

asyncio.run(main())
```

```console
$ python3 async_basics.py
['A after 0.3s', 'B after 0.2s', 'C after 0.1s'] in 0.30 s (not 0.6)
TaskGroup: ['service B is down'] | sibling cancelled: True
timed out after 0.1 s
6 tasks, 2 at a time: 0.30 s
```

<!-- expanded -->
### Going deeper: timeouts and cancellation in asyncio

`asyncio.timeout(seconds)` (3.11) cancels the code inside it when the deadline passes, raising `TimeoutError` outside. Cancellation is delivered as `CancelledError` *inside* the task at its current `await`, so `finally` blocks and `async with` exits run — and code must not swallow `CancelledError`.

```python
# file: async_timeouts.py
import asyncio

async def slow_query(name, seconds):
    try:
        await asyncio.sleep(seconds)
        return f"{name} done"
    except asyncio.CancelledError:
        print(f"  {name}: cancelled, cleaning up")
        raise                                   # always re-raise

async def main():
    try:
        async with asyncio.timeout(0.1):
            await slow_query("report", 5)
    except TimeoutError:
        print("report timed out after 0.1 s")

    task = asyncio.create_task(slow_query("export", 5))
    await asyncio.sleep(0.05)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("export task:", "cancelled" if task.cancelled() else "?")

    results = await asyncio.gather(slow_query("a", 0.01), asyncio.wait_for(slow_query("b", 1), 0.05),
                                   return_exceptions=True)
    print([type(r).__name__ if isinstance(r, BaseException) else r for r in results])

asyncio.run(main())
```

```console
$ python3 async_timeouts.py
  report: cancelled, cleaning up
report timed out after 0.1 s
  export: cancelled, cleaning up
export task: cancelled
  b: cancelled, cleaning up
['a done', 'TimeoutError']
```

`gather(..., return_exceptions=True)` collects failures as values instead of cancelling the rest at the first error.

## Unit 11.5 — Never block the event loop

**One-line summary:** asyncio is **cooperative**: a coroutine gives up control only at `await`. A blocking call (`time.sleep`, `requests.get`, a long CPU loop, a synchronous database driver) inside a coroutine freezes **every** task. Use async libraries, or hand blocking work to a thread with `await asyncio.to_thread(func, ...)` (or to a process pool via `loop.run_in_executor`).

```python
# file: blocking.py
import asyncio
import time

async def heartbeat(log):
    for _ in range(5):
        log.append(round(time.perf_counter() - START, 2))
        await asyncio.sleep(0.1)

def blocking_io():
    time.sleep(0.4)                                    # e.g. a synchronous HTTP library

async def bad():
    blocking_io()                                      # blocks the whole loop

async def good():
    await asyncio.to_thread(blocking_io)               # runs in a worker thread

async def run(job):
    log = []
    await asyncio.gather(heartbeat(log), job())
    return log

START = time.perf_counter()
print("blocking call in a coroutine, heartbeat times:", asyncio.run(run(bad)))
START = time.perf_counter()
print("asyncio.to_thread,           heartbeat times:", asyncio.run(run(good)))
```

```console
$ python3 blocking.py
blocking call in a coroutine, heartbeat times: [0.0, 0.4, 0.5, 0.6, 0.7]
asyncio.to_thread,           heartbeat times: [0.0, 0.1, 0.2, 0.3, 0.4]
```

### Deep & Tricky

**DT-26. A task created with `asyncio.create_task` and not stored anywhere can be garbage-collected before it finishes — the event loop keeps only a weak reference to tasks — and exceptions in fire-and-forget tasks are reported only when the task is destroyed ("Task exception was never retrieved"), often far from their cause.** *(Python documentation, `asyncio.create_task`: "Save a reference to the result of this function, to avoid a task disappearing mid-execution")* `TaskGroup` solves both problems by owning its tasks and propagating their exceptions; for long-lived background tasks, keep them in a set and remove them in a done-callback.

### Pattern

**P-12. I/O-bound with a few blocking calls → threads (`ThreadPoolExecutor`); I/O-bound with thousands of connections → asyncio; CPU-bound → processes (or native code that releases the GIL: NumPy, Chapter 13). Share data by passing messages (futures, queues), not by sharing mutable objects; when you must share, lock.** *(Practice)* Measure the speed-up — the overhead of starting workers, pickling, and switching can outweigh the gain. *Trade-off:* asyncio gives the most concurrency per thread but "colours" functions (async code calls async code) and needs async-aware libraries throughout; threads work with ordinary libraries but scale to hundreds, not tens of thousands, of concurrent operations.

### Mastery check

1. **Predict.** Ten coroutines each `await asyncio.sleep(1)` and are run with `gather`. About how long does it take? What if each calls `time.sleep(1)` instead?
2. **Predict.** Why can a `ProcessPoolExecutor` not run a `lambda` as its task function?

## Chapter 11 Project — Fetching 200 URLs three ways

**Goal:** a local TCP "web service" (an asyncio server that replies after a 50 ms delay, simulating network latency) serves 200 requests. Fetch them (1) sequentially with blocking sockets, (2) with a thread pool of 20, and (3) with asyncio and a semaphore of 50 — verify that all three get identical responses and compare wall times.

```python
# file: fetch/fetch.py
import asyncio
import socket
import threading
import time
from concurrent.futures import ThreadPoolExecutor

HOST, PORT, DELAY, N = "127.0.0.1", 8765, 0.05, 200

async def handle(reader, writer):
    req = (await reader.readline()).decode().strip()
    await asyncio.sleep(DELAY)                         # simulated network/processing latency
    writer.write(f"OK {req.upper()}\n".encode())
    await writer.drain()
    writer.close()

def run_server(ready):
    async def serve():
        server = await asyncio.start_server(handle, HOST, PORT, backlog=512)
        ready.set()
        async with server:
            await server.serve_forever()
    asyncio.run(serve())

def fetch_blocking(path):
    with socket.create_connection((HOST, PORT)) as s:
        s.sendall(f"{path}\n".encode())
        return s.makefile().readline().strip()

async def fetch_async(path, sem):
    async with sem:
        reader, writer = await asyncio.open_connection(HOST, PORT)
        writer.write(f"{path}\n".encode())
        await writer.drain()
        line = (await reader.readline()).decode().strip()
        writer.close()
        await writer.wait_closed()
        return line

async def fetch_all_async(paths):
    sem = asyncio.Semaphore(50)
    return await asyncio.gather(*(fetch_async(p, sem) for p in paths))

ready = threading.Event()
threading.Thread(target=run_server, args=(ready,), daemon=True).start()
ready.wait()
paths = [f"/item/{i}" for i in range(N)]
results = {}

t = time.perf_counter()
results["sequential"] = [fetch_blocking(p) for p in paths]
times = {"sequential": time.perf_counter() - t}
t = time.perf_counter()
with ThreadPoolExecutor(20) as pool:
    results["20 threads"] = list(pool.map(fetch_blocking, paths))
times["20 threads"] = time.perf_counter() - t
t = time.perf_counter()
results["asyncio, 50 at a time"] = asyncio.run(fetch_all_async(paths))
times["asyncio, 50 at a time"] = time.perf_counter() - t

for name, el in times.items():
    print(f"{name:>22}: {N} requests in {el:.2f} s  (lower bound {N * DELAY / {'sequential': 1, '20 threads': 20}.get(name, 50):.2f} s)")
print("identical responses:", results["sequential"] == results["20 threads"] == results["asyncio, 50 at a time"], "| e.g.", results["sequential"][7])
```

```console
$ cd fetch && python3 fetch.py
            sequential: 200 requests in 10.20 s  (lower bound 10.00 s)
            20 threads: 200 requests in 0.54 s  (lower bound 0.50 s)
 asyncio, 50 at a time: 200 requests in 0.26 s  (lower bound 0.20 s)
identical responses: True | e.g. OK /ITEM/7
```

Each request spends almost all its time waiting, so concurrency multiplies throughput by roughly the number of requests in flight — until something else (the server, the operating system, the event loop's own overhead) becomes the bottleneck. The lower bounds printed beside each time (N × delay / concurrency) show how close each approach gets.

### Review checklist

- Why does the server itself handle all 200 connections in one thread without becoming the bottleneck?
- What would happen to the asyncio version if `fetch_async` used `fetch_blocking` internally?
- How would you add a per-request timeout and retry to the asyncio client?

## Answers to the Chapter 11 prediction questions

**11.5 Q1.** About 1 second with `asyncio.sleep`: the sleeps overlap. With `time.sleep(1)` each call blocks the event loop, so they run one after another: about 10 seconds.

**11.5 Q2.** Tasks are sent to worker processes by pickling, and pickle stores functions by their importable name; a lambda has no importable name, so it cannot be pickled (`PicklingError`). Use a module-level function.

# Chapter 12 — A Tour of the Standard Library

*Goal: know the "batteries included" modules that professional Python code uses daily — specialised containers, heaps and bisection, dates and time zones, regular expressions, processes and the file system, command-line parsing, randomness and statistics — and the traps that make each of them bite.*

**What you will be able to do after this chapter**

- Use `collections` (`deque`, `Counter`, `ChainMap`), `heapq`, and `bisect` for common data tasks.
- Handle dates, times, durations, and time zones correctly with `datetime` and `zoneinfo`.
- Write, test, and debug regular expressions, and avoid catastrophic backtracking.
- Run external commands with `subprocess`, manage files with `shutil`, and build CLIs with `argparse`.
- Choose between `random`, `secrets`, and `statistics`.

## Topic 12A — Data helpers

## Unit 12.1 — `collections`, `heapq`, and `bisect`

**One-line summary:** `deque` is a double-ended queue with O(1) appends and pops at both ends (and `maxlen` for sliding windows); `Counter` counts and supports arithmetic on counts; `ChainMap` layers dictionaries (defaults ← config file ← command line) without copying. `heapq` keeps a list as a min-heap; `bisect` searches and inserts into sorted lists.

```python
# file: containers.py
import bisect
import heapq
from collections import ChainMap, Counter, deque

recent = deque(maxlen=3)                              # keeps only the last 3
for event in ["a", "b", "c", "d", "e"]:
    recent.append(event)
print(recent)
oldest = recent.popleft()
print(oldest, recent)

inventory, sold = Counter(apples=5, pears=2), Counter(apples=3, kiwis=1)
print(inventory - sold, inventory + sold, (inventory - sold).total())

defaults = {"theme": "light", "lang": "en", "debug": False}
config_file = {"lang": "hi"}
cli = {"debug": True}
settings = ChainMap(cli, config_file, defaults)        # first mapping wins
print(settings["theme"], settings["lang"], settings["debug"], dict(settings))

tasks = []
for prio, name in [(3, "deploy"), (1, "fix bug"), (2, "review")]:
    heapq.heappush(tasks, (prio, name))
print([heapq.heappop(tasks)[1] for _ in range(3)], heapq.nlargest(2, [5, 1, 9, 7]))

grades = [60, 70, 80, 90]                              # boundaries
letters = "FDCBA"
print([letters[bisect.bisect_right(grades, s)] for s in (55, 60, 75, 99)])
sorted_list = [10, 20, 30]
bisect.insort(sorted_list, 25)
print(sorted_list)
```

```console
$ python3 containers.py
deque(['c', 'd', 'e'], maxlen=3)
c deque(['d', 'e'], maxlen=3)
Counter({'apples': 2, 'pears': 2}) Counter({'apples': 8, 'pears': 2, 'kiwis': 1}) 4
light hi True {'theme': 'light', 'lang': 'hi', 'debug': True}
['fix bug', 'review', 'deploy'] [9, 7]
['F', 'D', 'C', 'A']
[10, 20, 25, 30]
```

<!-- expanded -->
### Going deeper: `Counter`, `defaultdict`, `deque`, and `heapq` in real tasks

```python
# file: containers2.py
import bisect
import heapq
from collections import Counter, defaultdict, deque, OrderedDict

words = "the cat sat on the mat the end".split()
c = Counter(words)
print(c.most_common(2), c["dog"], (c + Counter(["dog"]))["dog"])

graph = defaultdict(list)                        # adjacency list without key checks
for a, b in [("a", "b"), ("a", "c"), ("b", "d"), ("c", "d"), ("d", "e")]:
    graph[a].append(b)
def bfs(start):
    seen, q, order = {start}, deque([start]), []
    while q:
        node = q.popleft()                       # O(1); list.pop(0) is O(n)
        order.append(node)
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt)
                q.append(nxt)
    return order
print("bfs:", bfs("a"))

last3 = deque(maxlen=3)                          # a sliding window
for x in range(6):
    last3.append(x)
print("last three:", list(last3))

scores = [("ada", 91), ("bo", 78), ("cy", 99), ("di", 85)]
print("top 2:", heapq.nlargest(2, scores, key=lambda s: s[1]))
tasks = []
for pri, name in [(3, "email"), (1, "outage"), (2, "review")]:
    heapq.heappush(tasks, (pri, name))
print("by priority:", [heapq.heappop(tasks)[1] for _ in range(3)])

grades = [60, 70, 80, 90]
print("grade for 85:", "FDCBA"[bisect.bisect(grades, 85)])

lru = OrderedDict()
for key in ["a", "b", "c", "a", "d"]:
    lru[key] = True
    lru.move_to_end(key)
    if len(lru) > 3:
        lru.popitem(last=False)                  # evict the least recently used
print("LRU keeps:", list(lru))
```

```console
$ python3 containers2.py
[('the', 3), ('cat', 1)] 0 1
bfs: ['a', 'b', 'c', 'd', 'e']
last three: [3, 4, 5]
top 2: [('cy', 99), ('ada', 91)]
by priority: ['outage', 'review', 'email']
grade for 85: B
LRU keeps: ['c', 'a', 'd']
```

## Unit 12.2 — Dates, times, and time zones

**One-line summary:** A `datetime` is **naive** (no time zone) or **aware** (`tzinfo` set). Store and compute in **UTC**; convert to local time only for display, with `zoneinfo.ZoneInfo("Asia/Kolkata")`, which knows each zone's daylight-saving rules. `timedelta` represents durations; parse ISO 8601 with `fromisoformat`, format with `isoformat` or `strftime`.

```python
# file: times.py
from datetime import date, datetime, timedelta, timezone
from zoneinfo import ZoneInfo

utc_now = datetime(2026, 11, 15, 9, 30, tzinfo=timezone.utc)
for zone in ("Asia/Kolkata", "Europe/Berlin", "America/New_York", "Australia/Sydney"):
    local = utc_now.astimezone(ZoneInfo(zone))
    print(f"{zone:>18}: {local:%Y-%m-%d %H:%M} (UTC{local:%z})")
print(datetime.fromisoformat("2026-11-15T14:00:00+05:30").astimezone(timezone.utc).isoformat())
print(date(2026, 12, 25) - date(2026, 11, 15), timedelta(hours=36).days, (date(2024, 3, 1) - date(2024, 2, 1)).days)

ny = ZoneInfo("America/New_York")
start = datetime(2026, 11, 1, 0, 30, tzinfo=ny)        # the night clocks fall back (02:00 EDT -> 01:00 EST)
later = start + timedelta(hours=2)                     # wall-clock arithmetic
elapsed = later.astimezone(timezone.utc) - start.astimezone(timezone.utc)
print(f"{start:%H:%M %Z} + 2h = {later:%H:%M %Z}; real time elapsed: {elapsed}")
utc_later = (start.astimezone(timezone.utc) + timedelta(hours=2)).astimezone(ny)
print(f"adding 2h in UTC instead: {utc_later:%H:%M %Z}")
gap = datetime(2026, 3, 8, 2, 30, tzinfo=ny)           # this wall time never happens (clocks jump 02:00 -> 03:00)
print("nonexistent 02:30 on 8 March round-trips to:", gap.astimezone(timezone.utc).astimezone(ny).strftime("%H:%M %Z"))
```

```console
$ python3 times.py
      Asia/Kolkata: 2026-11-15 15:00 (UTC+0530)
     Europe/Berlin: 2026-11-15 10:30 (UTC+0100)
  America/New_York: 2026-11-15 04:30 (UTC-0500)
  Australia/Sydney: 2026-11-15 20:30 (UTC+1100)
2026-11-15T08:30:00+00:00
40 days, 0:00:00 1 29
00:30 EDT + 2h = 02:30 EST; real time elapsed: 3:00:00
adding 2h in UTC instead: 01:30 EST
nonexistent 02:30 on 8 March round-trips to: 03:30 EDT
```

### Deep & Tricky

**DT-27. Arithmetic on aware datetimes in the same time zone is **wall-clock** arithmetic: in New York on the night clocks fall back, 00:30 + 2 hours gives 02:30 local time — but 3 real hours have passed; and wall times inside the spring-forward gap (02:30 on 8 March 2026) do not exist at all and are silently shifted when converted. Compute durations and add intervals in UTC; convert to local time only for display.** *(Python documentation, `datetime`: "If both are aware and have the same tzinfo attribute, the tzinfo attributes are ignored, and the result is computed on the naive datetimes"; PEP 495 on `fold`)* Comparing naive and aware datetimes raises `TypeError`; `datetime.now()` without an argument is naive local time — prefer `datetime.now(timezone.utc)`.

## Unit 12.3 — Regular expressions

**One-line summary:** `re` matches patterns in text: `re.search` finds the first match anywhere, `re.fullmatch` requires the whole string, `re.findall`/`finditer` find all, `re.sub` replaces. Use raw strings (`r"\d+"`), **named groups** `(?P<name>...)`, `re.VERBOSE` for readable patterns, and compile patterns used repeatedly. Quantifiers are greedy (`.*`) unless made lazy (`.*?`).

```python
# file: regex.py
import re

LOG = re.compile(r"""
    (?P<ip>\d{1,3}(?:\.\d{1,3}){3}) \s+
    \[(?P<when>[^\]]+)\] \s+
    "(?P<method>GET|POST|PUT|DELETE) \s (?P<path>\S+)" \s+
    (?P<status>\d{3})
""", re.VERBOSE)
line = '10.0.0.7 [15/Nov/2026:09:30:00] "GET /api/items?id=3" 404'
m = LOG.search(line)
print(m.group("ip"), m["method"], m["path"], int(m["status"]), m.groupdict()["when"])
print(re.findall(r"\b\w+@\w+\.\w+\b", "mail ana@ex.com or ben@ex.org now"))
html = "<b>bold</b> and <i>italic</i>"
print(re.findall(r"<.*>", html), re.findall(r"<.*?>", html))       # greedy vs lazy
print(re.sub(r"(\d{4})-(\d{2})-(\d{2})", r"\3/\2/\1", "due 2026-11-15, paid 2026-12-01"))
print(bool(re.fullmatch(r"[A-Z]{2}\d{4}", "AB1234")), bool(re.fullmatch(r"[A-Z]{2}\d{4}", "AB12345")))
```

```console
$ python3 regex.py
10.0.0.7 GET /api/items?id=3 404 15/Nov/2026:09:30:00
['ana@ex.com', 'ben@ex.org']
['<b>bold</b> and <i>italic</i>'] ['<b>', '</b>', '<i>', '</i>']
due 15/11/2026, paid 01/12/2026
True False
```

### Deep & Tricky

**DT-28. Nested quantifiers such as `(a+)+$` or `(\w+\s?)*$` can take **exponential** time on inputs that almost match, because Python's backtracking engine tries every way of dividing the input among the repetitions — a single 30-character string can hang a server ("ReDoS").** *(Python `re` uses a backtracking matcher; OWASP, "Regular expression Denial of Service")* Avoid ambiguous nested repetition, anchor patterns, limit input length, and in 3.11+ use **atomic groups** `(?>...)` or **possessive quantifiers** `a++` to forbid backtracking into a group.

```python
# file: redos.py
import re
import time
for n in (16, 18, 20, 22):
    s = "a" * n + "!"                                 # almost matches: the final "!" forces full backtracking
    for name, pat in (("(a+)+$", r"(a+)+$"), ("(?>a+)+$", r"(?>a+)+$")):
        t = time.perf_counter()
        re.match(pat, s)
        el = time.perf_counter() - t
        print(f"n={n}  {name:<9} {el * 1000:9.2f} ms")
```

```console
$ python3 redos.py
n=16  (a+)+$         2.03 ms
n=16  (?>a+)+$       0.05 ms
n=18  (a+)+$         7.72 ms
n=18  (?>a+)+$       0.00 ms
n=20  (a+)+$        30.27 ms
n=20  (?>a+)+$       0.01 ms
n=22  (a+)+$       121.26 ms
n=22  (?>a+)+$       0.01 ms
```

Each extra character roughly doubles the time of the nested pattern; the atomic version is constant.

## Topic 12B — The operating system

## Unit 12.4 — `subprocess`, `shutil`, `os`, and `argparse`

**One-line summary:** Run external programs with `subprocess.run([...], capture_output=True, text=True, check=True)` — pass a **list** of arguments, not a shell string, to avoid shell injection. `shutil` copies, moves, archives, and finds executables; `os.environ` reads the environment; `argparse` builds command-line interfaces with types, defaults, choices, and generated help.

```python
# file: system.py
import argparse
import os
import shutil
import subprocess
import tempfile
from pathlib import Path

r = subprocess.run(["python3", "-c", "import sys; print('child says hi'); sys.exit(3)"], capture_output=True, text=True)
print("stdout:", r.stdout.strip(), "| return code:", r.returncode)
try:
    subprocess.run(["false"], check=True)
except subprocess.CalledProcessError as e:
    print("CalledProcessError, exit status", e.returncode)
user_input = "file.txt; echo INJECTED"
print("list args are not interpreted by a shell:", subprocess.run(["echo", user_input], capture_output=True, text=True).stdout.strip())

with tempfile.TemporaryDirectory() as d:
    src = Path(d, "src"); src.mkdir()
    (src / "a.txt").write_text("alpha")
    shutil.copytree(src, Path(d, "copy"))
    archive = shutil.make_archive(str(Path(d, "backup")), "zip", root_dir=d, base_dir="src")
    print(sorted(p.name for p in Path(d).iterdir()), Path(archive).suffix, shutil.which("python3") is not None)
print("HOME is set:", "HOME" in os.environ, "| PATH entries:", len(os.environ["PATH"].split(os.pathsep)))

parser = argparse.ArgumentParser(prog="resize", description="Resize images.")
parser.add_argument("files", nargs="+")
parser.add_argument("--width", type=int, required=True)
parser.add_argument("--format", choices=["png", "jpg"], default="png")
parser.add_argument("-v", "--verbose", action="store_true")
args = parser.parse_args(["a.png", "b.png", "--width", "640", "-v"])
print(args)
try:
    parser.parse_args(["a.png", "--width", "wide"])
except SystemExit as e:
    print("argparse exited with status", e.code)
```

```console
$ python3 system.py
stdout: child says hi | return code: 3
CalledProcessError, exit status 1
list args are not interpreted by a shell: file.txt; echo INJECTED
['backup.zip', 'copy', 'src'] .zip True
HOME is set: True | PATH entries: 16
Namespace(files=['a.png', 'b.png'], width=640, format='png', verbose=True)
usage: resize [-h] --width WIDTH [--format {png,jpg}] [-v] files [files ...]
resize: error: argument --width: invalid int value: 'wide'
argparse exited with status 2
```

<!-- expanded -->
### Going deeper: `subprocess.run` safely, and `argparse` subcommands

`subprocess.run([...])` with a *list* of arguments runs the program directly — no shell, so no quoting problems and no shell injection. `check=True` raises on a non-zero exit, `capture_output=True` collects output, `text=True` decodes it, and `timeout=` kills a hung child. `shell=True` with a string built from input is how command-injection bugs are written.

```python
# file: system2.py
import argparse
import subprocess
import sys

r = subprocess.run(["printf", "%s\n", "a file name; rm -rf /"], capture_output=True, text=True, check=True)
print("argument passed literally:", repr(r.stdout.strip()))
try:
    subprocess.run([sys.executable, "-c", "import sys; sys.exit(3)"], check=True)
except subprocess.CalledProcessError as e:
    print("CalledProcessError, returncode", e.returncode)
try:
    subprocess.run(["sleep", "5"], timeout=0.2)
except subprocess.TimeoutExpired:
    print("TimeoutExpired after 0.2 s (the child was killed)")

parser = argparse.ArgumentParser(prog="notes", description="A tiny notes tool.")
parser.add_argument("-v", "--verbose", action="count", default=0)
sub = parser.add_subparsers(dest="command", required=True)
add = sub.add_parser("add", help="add a note")
add.add_argument("text")
add.add_argument("--tag", action="append", default=[])
ls = sub.add_parser("list", help="list notes")
def limit(text):
    n = int(text)
    if not 1 <= n <= 100:
        raise argparse.ArgumentTypeError(f"{n} is not in 1..100")
    return n
ls.add_argument("--limit", type=limit, default=10)

print(parser.parse_args(["-vv", "add", "buy milk", "--tag", "home", "--tag", "shop"]))
print(parser.parse_args(["list", "--limit", "5"]))
try:
    parser.parse_args(["list", "--limit", "500"])
except SystemExit as e:
    print("argparse exited with status", e.code)
```

```console
$ python3 system2.py 2>&1 | grep -v "^usage:"
argument passed literally: 'a file name; rm -rf /'
CalledProcessError, returncode 3
TimeoutExpired after 0.2 s (the child was killed)
Namespace(verbose=2, command='add', text='buy milk', tag=['home', 'shop'])
Namespace(verbose=0, command='list', limit=5)
notes list: error: argument --limit: 500 is not in 1..100
argparse exited with status 2
```

`argparse` prints a usage message and exits with status 2 on bad input, like most Unix tools; in tests, catch `SystemExit` or use `parser.exit_on_error = False` (3.9+).

## Unit 12.5 — `random`, `secrets`, and `statistics`

**One-line summary:** `random` is a fast **pseudo-random** generator (Mersenne Twister) — reproducible with a seed, perfect for simulations and tests, and **predictable**, so never for passwords or tokens. `secrets` draws from the operating system's cryptographically secure source (`token_hex`, `token_urlsafe`, `choice`, `compare_digest`). `statistics` computes mean, median, standard deviation, quantiles, and correlation correctly.

```python
# file: chance.py
import random
import secrets
import statistics

rng = random.Random(42)                                # seeded: reproducible
print([rng.randint(1, 6) for _ in range(10)], rng.choice("ABC"), rng.sample(range(100), 3))
data = [rng.gauss(170, 8) for _ in range(10_000)]
print(f"mean {statistics.fmean(data):.2f}, median {statistics.median(data):.2f}, stdev {statistics.stdev(data):.2f}")
print("quartiles:", [round(q, 1) for q in statistics.quantiles(data, n=4)])
print("token:", len(secrets.token_urlsafe(32)), "chars | secure pick:", secrets.choice(["heads", "tails"]) in ("heads", "tails"))
print("constant-time compare:", secrets.compare_digest("s3cret", "s3cret"))
```

```console
$ python3 chance.py
[6, 1, 1, 6, 3, 2, 2, 2, 6, 1] C [94, 69, 11]
mean 170.02, median 169.97, stdev 7.96
quartiles: [164.5, 170.0, 175.4]
token: 43 chars | secure pick: True
constant-time compare: True
```

### Pattern

**P-13. Before writing a utility, search the standard library: `collections`, `itertools`, `functools`, `pathlib`, `datetime`/`zoneinfo`, `re`, `json`/`csv`, `subprocess`, `argparse`, `logging`, `statistics`, `secrets`, `concurrent.futures`, `contextlib`, `dataclasses`. Standard modules are documented, tested on every platform, and known to every reader.** *(Practice)* Store times in UTC, pass subprocess arguments as lists, compile hot regular expressions, and seed randomness in tests. *Trade-off:* some standard modules have better third-party successors for heavy use (`httpx`/`requests` over `urllib`, `regex` over `re` for Unicode-heavy work, `click`/`typer` over `argparse` for large CLIs) — adopt them deliberately, not by default.

### Mastery check

1. **Predict.** What does `Counter("banana").most_common(1)` return?
2. **Predict.** Is `datetime(2026, 1, 1) < datetime(2026, 1, 1, tzinfo=timezone.utc)` `True`, `False`, or an error?

## Chapter 12 Project — A time-zone-aware meeting planner

**Goal:** given participants in different time zones, each available 09:00–17:00 **in their own local time** on working days, find every 30-minute slot in a given week when **all** are available — correctly across a daylight-saving change that happens during the week for some of them. Compute in UTC with `zoneinfo`; verify against a brute-force check of every minute.

```python
# file: planner/planner.py
from datetime import date, datetime, time, timedelta, timezone
from zoneinfo import ZoneInfo

PEOPLE = {"Alan (London)": "Europe/London", "Bruno (Berlin)": "Europe/Berlin", "Carla (New York)": "America/New_York"}
WORK_START, WORK_END = time(9), time(17)

def available_utc(zone, day):
    """The person's working hours on a local calendar day, as a UTC interval (or None on weekends)."""
    if day.weekday() >= 5:
        return None
    tz = ZoneInfo(zone)
    start = datetime.combine(day, WORK_START, tzinfo=tz).astimezone(timezone.utc)
    end = datetime.combine(day, WORK_END, tzinfo=tz).astimezone(timezone.utc)
    return start, end

def intervals(zone, first, last):
    d = first - timedelta(days=1)                      # local days near the UTC range boundaries
    out = []
    while d <= last + timedelta(days=1):
        iv = available_utc(zone, d)
        if iv:
            out.append(iv)
        d += timedelta(days=1)
    return out

def common_slots(first, last, minutes=30):
    lists = [intervals(z, first, last) for z in PEOPLE.values()]
    common = lists[0]
    for other in lists[1:]:                            # intersect sorted interval lists pairwise
        merged = []
        for a0, a1 in common:
            for b0, b1 in other:
                lo, hi = max(a0, b0), min(a1, b1)
                if lo < hi:
                    merged.append((lo, hi))
        common = merged
    step = timedelta(minutes=minutes)
    slots = []
    for lo, hi in common:
        t = lo
        while t + step <= hi:
            slots.append(t)
            t += step
    return slots

def brute_free(t):                                     # check one instant directly in every local zone
    for zone in PEOPLE.values():
        local = t.astimezone(ZoneInfo(zone))
        if local.weekday() >= 5 or not (WORK_START <= local.time() < WORK_END):
            return False
    return True

first, last = date(2026, 10, 26), date(2026, 11, 6)    # US clocks change on Sunday Nov 1, 2026; Europe's on Oct 25
slots = common_slots(first, last)
for s in slots[:3] + slots[-3:]:
    locals_ = "  ".join(f"{name.split()[0]} {s.astimezone(ZoneInfo(z)):%a %H:%M}" for name, z in PEOPLE.items())
    print(f"{s:%a %d %b %H:%M} UTC | {locals_}")
by_day = {}
for s in slots:
    by_day.setdefault(s.date(), []).append(s)
print({str(d): len(v) for d, v in by_day.items()})
t = datetime(2026, 10, 26, tzinfo=timezone.utc)
in_slots = free = mismatches = 0
while t < datetime(2026, 11, 7, tzinfo=timezone.utc):  # brute force: every minute of the two weeks
    covered = any(s <= t < s + timedelta(minutes=30) for s in by_day.get(t.date(), []))
    is_free = brute_free(t)
    in_slots += covered; free += is_free; mismatches += covered != is_free
    t += timedelta(minutes=1)
print(f"brute force over every minute: {free} free minutes, {in_slots} minutes in proposed slots, mismatches: {mismatches}")
```

```console
$ cd planner && python3 planner.py
Mon 26 Oct 13:00 UTC | Alan Mon 13:00  Bruno Mon 14:00  Carla Mon 09:00
Mon 26 Oct 13:30 UTC | Alan Mon 13:30  Bruno Mon 14:30  Carla Mon 09:30
Mon 26 Oct 14:00 UTC | Alan Mon 14:00  Bruno Mon 15:00  Carla Mon 10:00
Fri 06 Nov 14:30 UTC | Alan Fri 14:30  Bruno Fri 15:30  Carla Fri 09:30
Fri 06 Nov 15:00 UTC | Alan Fri 15:00  Bruno Fri 16:00  Carla Fri 10:00
Fri 06 Nov 15:30 UTC | Alan Fri 15:30  Bruno Fri 16:30  Carla Fri 10:30
{'2026-10-26': 6, '2026-10-27': 6, '2026-10-28': 6, '2026-10-29': 6, '2026-10-30': 6, '2026-11-02': 4, '2026-11-03': 4, '2026-11-04': 4, '2026-11-05': 4, '2026-11-06': 4}
brute force over every minute: 1500 free minutes, 1500 minutes in proposed slots, mismatches: 0
```

Europe left summer time on 25 October, the US only on 1 November. In the first week New York is UTC−4 and the three-way overlap is 13:00–16:00 UTC (six slots per day); after 1 November New York is UTC−5, its day starts an hour later in UTC, and the overlap shrinks to 14:00–16:00 (four slots) — the per-day counts show the change, and the minute-by-minute brute force agrees exactly. Hard-coding UTC offsets ("New York is UTC−5") would have produced a week of wrong meeting times.

### Review checklist

- Why does `intervals` look at local days one day before and after the UTC range?
- Which participant determines the start of the common window, and which the end?
- How would you exclude public holidays per person?

## Answers to the Chapter 12 prediction questions

**12.5 Q1.** `[('a', 3)]` — "banana" has three a's.

**12.5 Q2.** An error: `TypeError: can't compare offset-naive and offset-aware datetimes`.

# Chapter 13 — Performance: Internals, NumPy, C Extensions, and Cython

*Goal: make Python code fast in the right order — measure first, then choose better algorithms and data structures, then remove interpreter overhead with idiomatic built-ins, then move hot loops into compiled code with NumPy, Cython, or C — and verify at every step that the faster code computes the same results.*

**What you will be able to do after this chapter**

- Measure reliably with `timeit` and `perf_counter`, and know what the numbers mean.
- Apply interpreter-level optimisations that matter (and recognise the ones that do not).
- Vectorise numeric code with NumPy arrays, broadcasting, and boolean masks.
- Call C code from Python with `ctypes`, and compile typed Python-like code with Cython.
- Compare four implementations of the same kernel, checking that their results agree.

## Topic 13A — Python-level speed

## Unit 13.1 — Where the time goes, and cheap wins

**One-line summary:** Each Python-level operation costs tens of nanoseconds of interpreter work (dispatch, reference counting, dynamic lookups — Chapter 0). So: first use a better **algorithm** (O(n) instead of O(n²)); then push loops into **built-ins written in C** (`sum`, `sorted`, `str.join`, `set` operations, comprehensions); avoid repeated work inside loops (attribute lookups, string concatenation in a loop). Micro-tricks like caching a method in a local variable give small gains — measure before keeping them.

```python
# file: cheap_wins.py
import timeit

words = [f"w{i}" for i in range(20_000)]

def concat_loop():
    s = ""
    for w in words:
        s += w + ","                                    # may copy the growing string
    return s

def concat_join():
    return ",".join(words) + ","

def squares_loop():
    out = []
    for i in range(20_000):
        out.append(i * i)
    return out

def squares_comp():
    return [i * i for i in range(20_000)]

def squares_local_append():
    out = []
    append = out.append                                 # hoist the attribute lookup
    for i in range(20_000):
        append(i * i)
    return out

def total_loop():
    t = 0
    for i in range(200_000):
        t += i
    return t

def total_builtin():
    return sum(range(200_000))

assert concat_loop() == concat_join() and squares_loop() == squares_comp() == squares_local_append()
for f in (concat_loop, concat_join, squares_loop, squares_local_append, squares_comp, total_loop, total_builtin):
    us = min(timeit.repeat(f, number=20, repeat=5)) / 20 * 1e6
    print(f"{f.__name__:>22}: {us:9,.0f} µs")
```

```console
$ python3 cheap_wins.py
           concat_loop:       772 µs
           concat_join:        89 µs
          squares_loop:       495 µs
  squares_local_append:       564 µs
          squares_comp:       451 µs
            total_loop:     4,125 µs
         total_builtin:     2,021 µs
```

The big wins come from moving the loop into C (`join`, `sum`); comprehensions beat `append` loops modestly; hoisting `out.append` helps less than folklore suggests on modern CPython, whose specialising interpreter (Chapter 0) already speeds up attribute access.

<!-- expanded -->
### Going deeper: local variable lookups and built-in functions

In CPython, a local variable is an index into an array, a global is a dictionary lookup, and an attribute (`obj.method`) is a lookup on every access. In a hot loop, built-ins written in C (`sum`, `map`, `sorted`, `str.join`) beat Python-level loops because the loop itself runs in C.

```python
# file: cheap_wins2.py
import timeit

data = list(range(1_000_000))

def loop_sum():
    total = 0
    for x in data:
        total += x
    return total

def append_loop():
    out = []
    for x in data:
        out.append(x * 2)
    return out

def comprehension():
    return [x * 2 for x in data]

t = {name: timeit.timeit(fn, number=5) for name, fn in
     [("for loop sum", loop_sum), ("built-in sum", lambda: sum(data)),
      ("append loop", append_loop), ("comprehension", comprehension)]}
print("sum() faster than the for loop:", t["built-in sum"] < t["for loop sum"] / 2)
print("comprehension faster than append loop:", t["comprehension"] < t["append loop"])
```

```console
$ python3 cheap_wins2.py
sum() faster than the for loop: True
comprehension faster than append loop: True
```

Python 3.11's specialising interpreter narrowed many of these gaps (Unit 0.2), so measure rather than rewrite on principle: the built-in `sum` is still several times faster, the comprehension only modestly.

## Topic 13B — NumPy

## Unit 13.2 — Vectorisation, broadcasting, and masks

**One-line summary:** A NumPy `ndarray` is a block of **unboxed, fixed-type** numbers (`int64`, `float64`, …) with operations implemented in C over the whole array. Replace Python loops by **array expressions** (`a * b + c`), **broadcasting** (combining arrays of compatible shapes without copying), **boolean masks** (`a[a > 0]`), and **reductions** (`sum`, `mean`, `argmax` along an axis).

```python
# file: numpy_basics.py
import time
import numpy as np

rng = np.random.default_rng(13)
x = rng.random(1_000_000)
t = time.perf_counter(); py = sum(v * v for v in x.tolist()); tp = time.perf_counter() - t
t = time.perf_counter(); vec = float(np.dot(x, x)); tv = time.perf_counter() - t
print(f"sum of squares: Python {tp * 1000:.0f} ms, NumPy {tv * 1000:.2f} ms, relative difference {abs(py - vec) / vec:.1e}")

m = np.arange(12).reshape(3, 4)
print(m, m.shape, m.dtype, sep="\n")
print("column means:", m.mean(axis=0), "| row sums:", m.sum(axis=1))
print("broadcast add a row vector:\n", m + np.array([100, 200, 300, 400]))
print("mask:", m[m % 3 == 0], "| where:", np.where(m > 6, m, 0)[1])

points = rng.random((5, 2))
d = np.sqrt(((points[:, None, :] - points[None, :, :]) ** 2).sum(axis=2))   # all pairwise distances
print("pairwise distance matrix shape:", d.shape, "| symmetric:", np.allclose(d, d.T), "| zero diagonal:", np.allclose(np.diag(d), 0))
```

```console
$ python3 numpy_basics.py
sum of squares: Python 50 ms, NumPy 0.82 ms, relative difference 3.5e-16
[[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]
(3, 4)
int64
column means: [4. 5. 6. 7.] | row sums: [ 6 22 38]
broadcast add a row vector:
 [[100 201 302 403]
 [104 205 306 407]
 [108 209 310 411]]
mask: [0 3 6 9] | where: [0 0 0 7]
pairwise distance matrix shape: (5, 5) | symmetric: True | zero diagonal: True
```

### Deep & Tricky

**DT-29. NumPy integers have fixed width and **wrap around silently** on overflow in array arithmetic (`np.int64` max + 1 becomes the most negative value), unlike Python's unbounded `int`; default integer dtypes can also differ by platform (32-bit `int` on Windows for NumPy < 2.0).** *(NumPy documentation, "Overflow errors": "NumPy… integer types… overflow… without warning" for array operations; NumPy 2.0 release notes on default integer size)* Scalar operations on NumPy integers may warn, but vectorised operations do not. Choose dtypes deliberately (`int64`, or `float64`/`object` when values can grow), and check ranges when converting results back to Python.

```python
# file: overflow.py
import numpy as np
a = np.array([2**62, 2**62], dtype=np.int64)
print("NumPy int64:", a + a, "| sum:", a.sum())
print("Python int: ", 2**62 + 2**62)
small = np.array([200, 100], dtype=np.uint8)
print("uint8 200 + 100 =", small[0] + small[1], "| as Python ints:", int(small[0]) + int(small[1]))
```

```console
$ python3 overflow.py
NumPy int64: [-9223372036854775808 -9223372036854775808] | sum: -9223372036854775808
Python int:  9223372036854775808
overflow.py:7: RuntimeWarning: overflow encountered in scalar add
  print("uint8 200 + 100 =", small[0] + small[1], "| as Python ints:", int(small[0]) + int(small[1]))
uint8 200 + 100 = 44 | as Python ints: 300
```

## Topic 13C — Compiled code

## Unit 13.3 — Calling C with `ctypes`

**One-line summary:** Compile C code into a shared library (`gcc -O2 -shared -fPIC`) and load it with `ctypes.CDLL`; declare each function's `argtypes` and `restype`; pass NumPy arrays through `ndarray.ctypes.data_as(...)` pointers. No Python headers or build system are needed — but you are responsible for matching types exactly and for memory safety.

```c
// file: fastsum.c
#include <stdint.h>
#include <stddef.h>

int64_t sum_squares(const int64_t *a, size_t n) {
    int64_t s = 0;
    for (size_t i = 0; i < n; i++) s += a[i] * a[i];
    return s;
}
```

```python
# file: call_c.py
import ctypes
import time
import numpy as np

lib = ctypes.CDLL("./libfastsum.so")
lib.sum_squares.argtypes = [ctypes.POINTER(ctypes.c_int64), ctypes.c_size_t]
lib.sum_squares.restype = ctypes.c_int64

a = np.arange(1_000_000, dtype=np.int64)
t = time.perf_counter(); c_result = lib.sum_squares(a.ctypes.data_as(ctypes.POINTER(ctypes.c_int64)), a.size); tc = time.perf_counter() - t
t = time.perf_counter(); py_result = sum(int(v) * int(v) for v in range(1_000_000)); tp = time.perf_counter() - t
print(f"C: {c_result} in {tc * 1000:.2f} ms | Python: {py_result} in {tp * 1000:.0f} ms | equal: {c_result == py_result}")
```

```console
$ gcc -O2 -shared -fPIC -o libfastsum.so fastsum.c && python3 call_c.py
C: 333332833333500000 in 1.31 ms | Python: 333332833333500000 in 45 ms | equal: True
```

<!-- expanded -->
### Going deeper: passing arrays and structs to C with `ctypes`

`ctypes` describes C types in Python: `c_int * n` is an array type, a `Structure` subclass with `_fields_` mirrors a C struct, and `argtypes`/`restype` tell `ctypes` how to convert arguments — without them, it assumes every argument and the return value is an `int`, and a `double` result comes back as garbage.

```c
// file: geom.c
#include <math.h>
typedef struct { double x, y; } Point;

double path_length(const Point *pts, int n) {
    double total = 0;
    for (int i = 1; i < n; i++) total += hypot(pts[i].x - pts[i - 1].x, pts[i].y - pts[i - 1].y);
    return total;
}
void scale(Point *pts, int n, double k) {
    for (int i = 0; i < n; i++) { pts[i].x *= k; pts[i].y *= k; }
}
```

```python
# file: call_geom.py
import ctypes

lib = ctypes.CDLL("./libgeom.so")

class Point(ctypes.Structure):
    _fields_ = [("x", ctypes.c_double), ("y", ctypes.c_double)]

lib.path_length.argtypes = [ctypes.POINTER(Point), ctypes.c_int]
lib.path_length.restype = ctypes.c_double
lib.scale.argtypes = [ctypes.POINTER(Point), ctypes.c_int, ctypes.c_double]
lib.scale.restype = None

pts = (Point * 3)(Point(0, 0), Point(3, 4), Point(6, 8))
print("length:", lib.path_length(pts, len(pts)))
lib.scale(pts, len(pts), 0.5)                      # C modifies our array in place
print("scaled:", [(p.x, p.y) for p in pts])

raw = ctypes.CDLL("./libgeom.so")                  # same library, no restype declared
print("without restype the double comes back as:", type(raw.path_length(pts, 3)).__name__)
```

```console
$ gcc -O2 -shared -fPIC geom.c -o libgeom.so -lm && python3 call_geom.py
length: 10.0
scaled: [(0.0, 0.0), (1.5, 2.0), (3.0, 4.0)]
without restype the double comes back as: int
```

## Unit 13.4 — Cython

**One-line summary:** **Cython** compiles Python-like code to a C extension module. Unmodified Python code gets modest gains; adding **static C types** (`cdef int i`, typed memoryviews `double[:, :]`) turns loops into plain C. Build with `cythonize -i module.pyx` (or through `setup.py`/`pyproject.toml`); import the result like any module. `cython -a` produces an annotated HTML report showing which lines still use the Python API.

```python
# file: primes_cy.pyx
# cython: language_level=3, boundscheck=False, wraparound=False
def count_primes(int limit):
    cdef int n, d, count = 0
    cdef bint is_prime
    for n in range(2, limit):
        is_prime = True
        d = 2
        while d * d <= n:
            if n % d == 0:
                is_prime = False
                break
            d += 1
        if is_prime:
            count += 1
    return count
```

```python
# file: primes_py.py
def count_primes(limit):
    count = 0
    for n in range(2, limit):
        d, is_prime = 2, True
        while d * d <= n:
            if n % d == 0:
                is_prime = False
                break
            d += 1
        count += is_prime
    return count
```

```python
# file: compare_primes.py
import time
import primes_cy
import primes_py

for name, mod in (("Python", primes_py), ("Cython (typed)", primes_cy)):
    t = time.perf_counter()
    r = mod.count_primes(200_000)
    print(f"{name:>15}: {r} primes below 200,000 in {(time.perf_counter() - t) * 1000:,.0f} ms")
```

```console
$ cythonize -i -q primes_cy.pyx > /dev/null 2>&1 && ls primes_cy*.so | sed 's/\.cpython.*//' && python3 compare_primes.py
primes_cy
         Python: 17984 primes below 200,000 in 316 ms
 Cython (typed): 17984 primes below 200,000 in 12 ms
```

### Pattern

**P-14. Optimise in this order: measure → better algorithm → better data structure → built-ins and comprehensions → vectorise with NumPy → compile the remaining hot loop (Cython, C, Rust via PyO3, or Numba). Keep the slow version as a reference and check that every faster version produces the same results.** *(Practice)* Most programs spend most of their time in a few lines; optimise only those, and stop when the program is fast enough. *Trade-off:* each step down the list costs readability, portability, or build complexity — NumPy adds a dependency; C extensions add a compiler, platform-specific wheels, and memory-safety risks.

### Mastery check

1. **Predict.** Why is `np.dot(x, x)` on a million floats hundreds of times faster than `sum(v * v for v in x)`?
2. **Predict.** What happens if `restype` is not set for a `ctypes` function that returns a 64-bit integer?

## Chapter 13 Project — One kernel, four implementations

**Goal:** compute the Mandelbrot set's escape counts on a 400 × 300 grid (max 200 iterations) four ways — pure Python, NumPy (vectorised over the whole grid with masks), Cython with typed loops, and C called via `ctypes` — verify that all four produce **identical** integer arrays, and time them.

```python
# file: mandel/mandel_py.py
def mandel_py(w, h, max_iter, x0=-2.0, x1=1.0, y0=-1.2, y1=1.2):
    out = [[0] * w for _ in range(h)]
    for j in range(h):
        ci = y0 + (y1 - y0) * j / (h - 1)
        for i in range(w):
            cr = x0 + (x1 - x0) * i / (w - 1)
            zr = zi = 0.0
            n = 0
            while n < max_iter and zr * zr + zi * zi <= 4.0:
                zr, zi = zr * zr - zi * zi + cr, 2.0 * zr * zi + ci
                n += 1
            out[j][i] = n
    return out
```

```python
# file: mandel/mandel_np.py
import numpy as np

def mandel_np(w, h, max_iter, x0=-2.0, x1=1.0, y0=-1.2, y1=1.2):
    cr = x0 + (x1 - x0) * np.arange(w) / (w - 1)
    ci = y0 + (y1 - y0) * np.arange(h) / (h - 1)
    CR, CI = np.meshgrid(cr, ci)
    ZR, ZI = np.zeros_like(CR), np.zeros_like(CI)
    counts = np.zeros(CR.shape, dtype=np.int64)
    active = np.ones(CR.shape, dtype=bool)
    for _ in range(max_iter):
        active &= ZR * ZR + ZI * ZI <= 4.0                # points still iterating
        if not active.any():
            break
        zr, zi = ZR[active], ZI[active]
        ZR[active], ZI[active] = zr * zr - zi * zi + CR[active], 2.0 * zr * zi + CI[active]
        counts[active] += 1
    return counts
```

```python
# file: mandel/mandel_cy.pyx
# cython: language_level=3, boundscheck=False, wraparound=False, cdivision=True
import numpy as np

def mandel_cy(int w, int h, int max_iter, double x0=-2.0, double x1=1.0, double y0=-1.2, double y1=1.2):
    out = np.zeros((h, w), dtype=np.int64)
    cdef long long[:, :] o = out
    cdef int i, j, n
    cdef double cr, ci, zr, zi, t
    for j in range(h):
        ci = y0 + (y1 - y0) * j / (h - 1)
        for i in range(w):
            cr = x0 + (x1 - x0) * i / (w - 1)
            zr = 0.0; zi = 0.0; n = 0
            while n < max_iter and zr * zr + zi * zi <= 4.0:
                t = zr * zr - zi * zi + cr
                zi = 2.0 * zr * zi + ci
                zr = t
                n += 1
            o[j, i] = n
    return out
```

```c
// file: mandel/mandel.c
#include <stdint.h>

void mandel_c(int w, int h, int max_iter, double x0, double x1, double y0, double y1, int64_t *out) {
    for (int j = 0; j < h; j++) {
        double ci = y0 + (y1 - y0) * j / (h - 1);
        for (int i = 0; i < w; i++) {
            double cr = x0 + (x1 - x0) * i / (w - 1);
            double zr = 0.0, zi = 0.0;
            int n = 0;
            while (n < max_iter && zr * zr + zi * zi <= 4.0) {
                double t = zr * zr - zi * zi + cr;
                zi = 2.0 * zr * zi + ci;
                zr = t;
                n++;
            }
            out[j * w + i] = n;
        }
    }
}
```

```python
# file: mandel/bench.py
import ctypes
import time
import numpy as np
from mandel_py import mandel_py
from mandel_np import mandel_np
from mandel_cy import mandel_cy

lib = ctypes.CDLL("./libmandel.so")
lib.mandel_c.argtypes = [ctypes.c_int] * 3 + [ctypes.c_double] * 4 + [ctypes.POINTER(ctypes.c_int64)]
lib.mandel_c.restype = None

def mandel_c(w, h, max_iter, x0=-2.0, x1=1.0, y0=-1.2, y1=1.2):
    out = np.zeros((h, w), dtype=np.int64)
    lib.mandel_c(w, h, max_iter, x0, x1, y0, y1, out.ctypes.data_as(ctypes.POINTER(ctypes.c_int64)))
    return out

W, H, IT = 400, 300, 200
results, times = {}, {}
for name, f in (("pure Python", mandel_py), ("NumPy", mandel_np), ("Cython", mandel_cy), ("C via ctypes", mandel_c)):
    t = time.perf_counter()
    results[name] = np.asarray(f(W, H, IT))
    times[name] = time.perf_counter() - t
base = times["pure Python"]
for name in results:
    print(f"{name:>13}: {times[name] * 1000:8,.1f} ms  ({base / times[name]:6.1f}x)  identical to Python: {np.array_equal(results[name], results['pure Python'])}")
inside = (results["C via ctypes"] == IT).sum()
print(f"{inside:,} of {W * H:,} points never escaped (in the set)")
rows = results["C via ctypes"][::30, ::10]
print("\n".join("".join(" .:-=+*#%@"[min(9, v // 20)] for v in row) for row in rows))
```

```console
$ cd mandel && cythonize -i -q mandel_cy.pyx > /dev/null 2>&1 && gcc -O2 -shared -fPIC -o libmandel.so mandel.c && python3 bench.py
  pure Python:    375.9 ms  (   1.0x)  identical to Python: True
        NumPy:    129.3 ms  (   2.9x)  identical to Python: True
       Cython:     13.0 ms  (  28.8x)  identical to Python: True
 C via ctypes:     13.0 ms  (  29.0x)  identical to Python: True
25,300 of 120,000 points never escaped (in the set)
                                        
                                        
                        @@@             
                  .@@@@@@@@@@@@         
           :@@@. =@@@@@@@@@@@@@@@       
   @ . .-@@@@@@@@@@@@@@@@@@@@@@         
           -.@@. =@@@@@@@@@@@@@@        
                  .=@@@@@@@@@@@         
                        @@@             
                                        
```

All four produce bit-identical counts because every version performs the same floating-point operations in the same order (the C and Cython versions use a temporary exactly like the tuple assignment in Python). NumPy is faster than pure Python but loses to the compiled loops here: the masked updates allocate temporary arrays on every iteration, and points that escape early still cost mask operations — vectorisation is not always the end of the road.

### Review checklist

- Why must the compiled versions compute `zr` and `zi` in the same order as Python's tuple assignment to get identical results?
- Which costs dominate the NumPy version, and how could you reduce them? (Compact the active points into smaller arrays every few iterations.)
- What would you need to ship the Cython version to users who have no compiler? (Pre-built wheels per platform and Python version.)

## Answers to the Chapter 13 prediction questions

**13.5 Q1.** `np.dot` runs a compiled loop (often SIMD-vectorised, via BLAS) over raw `float64` values; the generator expression creates a Python float object for every element and executes several bytecode instructions per element.

**13.5 Q2.** `ctypes` assumes the return type is a C `int` (usually 32 bits), so the 64-bit result is truncated — wrong values with no error.

# Chapter 14 — Metaprogramming: Attribute Access, Descriptors, Metaclasses, and the AST

*Goal: understand the machinery underneath Python's object model — how attribute lookup really works, how descriptors implement methods and properties, how classes are created (and customised with `__init_subclass__`, class decorators, and metaclasses), and how to read and transform code as data with the `ast` module — and know when *not* to use these powers.*

**What you will be able to do after this chapter**

- Trace an attribute lookup through `__getattribute__`, descriptors, the instance dict, and `__getattr__`.
- Write descriptors with `__get__`, `__set__`, and `__set_name__`; explain how methods bind.
- Register and customise subclasses with `__init_subclass__` and class decorators.
- Explain what a metaclass is, create classes with `type(...)`, and recognise when a metaclass is justified.
- Parse, inspect, transform, and compile Python code with `ast`.

## Topic 14A — Attributes

## Unit 14.1 — How attribute lookup works

**One-line summary:** `obj.name` calls `type(obj).__getattribute__(obj, "name")`, which checks, in order: a **data descriptor** on the class (one with `__set__`, e.g. a property), then the **instance `__dict__`**, then a **non-data descriptor** or plain attribute on the class (functions, class variables), and finally calls `__getattr__` if defined. `getattr`, `setattr`, `hasattr`, and `delattr` do the same with names chosen at run time.

```python
# file: lookup.py
class Config:
    defaults = {"timeout": 30, "retries": 3}
    def __init__(self, **overrides):
        self.__dict__.update(overrides)
    def __getattr__(self, name):                       # called only when normal lookup fails
        try:
            return self.defaults[name]
        except KeyError:
            raise AttributeError(f"no setting {name!r}") from None

c = Config(timeout=5)
print(c.timeout, c.retries, vars(c))
print(getattr(c, "retries"), getattr(c, "colour", "default-colour"), hasattr(c, "colour"))
setattr(c, "verbose", True)
print(vars(c))

class Traced:
    def __getattribute__(self, name):                  # called for EVERY attribute access
        if not name.startswith("__"):
            print(f"  looking up {name!r}")
        return super().__getattribute__(name)
    x = 1
t = Traced()
print(t.x + t.x)
```

```console
$ python3 lookup.py
5 3 {'timeout': 5}
3 default-colour False
{'timeout': 5, 'verbose': True}
  looking up 'x'
  looking up 'x'
2
```

<!-- expanded -->
### Going deeper: `__getattr__` versus `__getattribute__`

`__getattribute__` runs on *every* attribute access; overriding it is rarely right and easy to make recursive. `__getattr__` runs only when normal lookup *fails*, which makes it the tool for proxies, lazy attributes, and friendly error messages.

```python
# file: getattr_demo.py
class Config:
    def __init__(self, **values):
        self._values = values
    def __getattr__(self, name):                     # only for names not found normally
        try:
            return self._values[name]
        except KeyError:
            close = [k for k in self._values if k[0] == name[0]]
            raise AttributeError(f"no setting {name!r}" + (f"; did you mean {close[0]!r}?" if close else "")) from None

class Proxy:
    def __init__(self, target):
        self._target = target
        self.calls = []
    def __getattr__(self, name):
        attr = getattr(self._target, name)
        if callable(attr):
            def wrapper(*a, **k):
                self.calls.append(name)
                return attr(*a, **k)
            return wrapper
        return attr

cfg = Config(timeout=5, retries=3)
print(cfg.timeout, cfg._values is not None)
try:
    cfg.retry
except AttributeError as e:
    print("AttributeError:", e)
p = Proxy([3, 1, 2])
p.sort(); p.append(4)
print(p._target, p.calls, len(p._target))
```

```console
$ python3 getattr_demo.py
5 True
AttributeError: no setting 'retry'; did you mean 'retries'?
[1, 2, 3, 4] ['sort', 'append'] 4
```

## Unit 14.2 — Descriptors

**One-line summary:** A **descriptor** is an object stored as a **class** attribute that defines `__get__` (and optionally `__set__`/`__delete__`); accessing the attribute through an instance calls these methods. `__set_name__` tells the descriptor the attribute name it was assigned to. Properties, methods (functions are descriptors that return **bound methods**), `classmethod`, `staticmethod`, and `__slots__` are all built from descriptors.

```python
# file: descriptors.py
class Typed:
    """A reusable validated field."""
    def __init__(self, kind, minimum=None):
        self.kind, self.minimum = kind, minimum
    def __set_name__(self, owner, name):
        self.name = name
        self.storage = f"_{name}"
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self                                # accessed on the class itself
        return getattr(obj, self.storage)
    def __set__(self, obj, value):
        if not isinstance(value, self.kind):
            raise TypeError(f"{self.name} must be {self.kind.__name__}, got {type(value).__name__}")
        if self.minimum is not None and value < self.minimum:
            raise ValueError(f"{self.name} must be >= {self.minimum}")
        setattr(obj, self.storage, value)

class Product:
    name = Typed(str)
    price = Typed(float, minimum=0.0)
    stock = Typed(int, minimum=0)
    def __init__(self, name, price, stock):
        self.name, self.price, self.stock = name, price, stock

p = Product("pen", 1.5, 10)
print(p.name, p.price, p.stock, Product.price.name)
for field, bad in (("price", -1.0), ("stock", "ten")):
    try:
        setattr(p, field, bad)
    except (TypeError, ValueError) as e:
        print(type(e).__name__, "-", e)

def greet(self):
    return f"hi from {self.name}"
Product.greet = greet                                  # a plain function stored on the class...
print(type(Product.__dict__["greet"]), "->", type(p.greet), p.greet())   # ...becomes a bound method via __get__
print(greet.__get__(p, Product)())                     # what p.greet does under the hood
```

```console
$ python3 descriptors.py
pen 1.5 10 price
ValueError - price must be >= 0.0
TypeError - stock must be int, got str
<class 'function'> -> <class 'method'> hi from pen
hi from pen
```

### Deep & Tricky

**DT-30. Lookup precedence depends on whether a descriptor defines `__set__`: a **data descriptor** (property, a descriptor with `__set__`) on the class wins over the instance `__dict__`, while a **non-data descriptor** (a plain function, or `__get__` only) loses to it — which is why you can shadow a method by assigning `obj.method = something`, but cannot bypass a property that way.** *(Python "Descriptor HowTo Guide", "Invocation from an instance")* This rule is also why `functools.cached_property` (a non-data descriptor) works: the first access stores the value in the instance dict under the same name, and later accesses find it there without calling the descriptor again.

```python
# file: precedence.py
from functools import cached_property

class Demo:
    @property
    def p(self): return "property (data descriptor)"
    def m(self): return "method (non-data descriptor)"
    @cached_property
    def heavy(self):
        print("  computing heavy once")
        return 42

d = Demo()
d.__dict__["p"] = "instance dict"
d.__dict__["m"] = lambda: "instance dict"
print(d.p, "|", d.m())
print(d.heavy, d.heavy, "| stored in instance dict:", "heavy" in vars(d))
```

```console
$ python3 precedence.py
property (data descriptor) | instance dict
  computing heavy once
42 42 | stored in instance dict: True
```

## Topic 14B — Creating classes

## Unit 14.3 — `__init_subclass__` and class decorators

**One-line summary:** Most class customisation does **not** need a metaclass. `__init_subclass__` (defined on a base class) runs every time a subclass is created — ideal for registries and validation of subclasses. A **class decorator** receives the finished class and can modify or register it (`@dataclass` is one).

```python
# file: registry.py
class Exporter:
    registry = {}
    def __init_subclass__(cls, *, fmt, **kwargs):
        super().__init_subclass__(**kwargs)
        if not hasattr(cls, "export"):
            raise TypeError(f"{cls.__name__} must define export()")
        Exporter.registry[fmt] = cls

class CsvExporter(Exporter, fmt="csv"):
    def export(self, rows): return "\n".join(",".join(map(str, r)) for r in rows)

class JsonExporter(Exporter, fmt="json"):
    def export(self, rows):
        import json
        return json.dumps(rows)

rows = [[1, "a"], [2, "b"]]
for fmt, cls in Exporter.registry.items():
    print(fmt, "->", cls().export(rows).replace("\n", " | "))
try:
    class BrokenExporter(Exporter, fmt="xml"):
        pass
except TypeError as e:
    print("TypeError:", e)

def add_repr(cls):                                     # a class decorator
    fields = list(vars(cls).get("__annotations__", {}))
    cls.__repr__ = lambda self: f"{cls.__name__}(" + ", ".join(f"{f}={getattr(self, f)!r}" for f in fields) + ")"
    return cls

@add_repr
class Point:
    x: int
    y: int
    def __init__(self, x, y): self.x, self.y = x, y
print(Point(1, 2))
```

```console
$ python3 registry.py
csv -> 1,a | 2,b
json -> [[1, "a"], [2, "b"]]
TypeError: BrokenExporter must define export()
Point(x=1, y=2)
```

<!-- expanded -->
### Going deeper: a plugin registry with `__init_subclass__`

`__init_subclass__` runs on the parent whenever a subclass is *defined*. It replaces most metaclass uses: registering plugins, validating that subclasses define required attributes, or accepting class-level keyword arguments.

```python
# file: plugins.py
class Exporter:
    registry: dict[str, type["Exporter"]] = {}

    def __init_subclass__(cls, *, fmt: str, **kwargs):
        super().__init_subclass__(**kwargs)
        if not hasattr(cls, "export"):
            raise TypeError(f"{cls.__name__} must define export()")
        Exporter.registry[fmt] = cls

class CsvExporter(Exporter, fmt="csv"):
    def export(self, rows):
        return "\n".join(",".join(map(str, r)) for r in rows)

class JsonExporter(Exporter, fmt="json"):
    def export(self, rows):
        import json
        return json.dumps(rows)

print(sorted(Exporter.registry))
print(Exporter.registry["csv"]().export([(1, 2), (3, 4)]))
try:
    class Broken(Exporter, fmt="xml"):
        pass
except TypeError as e:
    print("TypeError:", e)
```

```console
$ python3 plugins.py
['csv', 'json']
1,2
3,4
TypeError: Broken must define export()
```

## Unit 14.4 — Metaclasses and `type`

**One-line summary:** Classes are objects too; their type is a **metaclass**, by default `type`. `type(name, bases, namespace)` creates a class dynamically. A custom metaclass (`class Meta(type)` with `__new__`/`__call__`) controls class creation and instantiation for a whole hierarchy — needed rarely (ORMs, ABCs, enums use them); prefer `__init_subclass__` and decorators when they suffice.

```python
# file: metaclasses.py
Point = type("Point", (), {"x": 0, "norm": lambda self: abs(self.x)})
p = Point(); p.x = -3
print(Point, type(Point), p.norm(), type(type))

class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):                # controls Class(...) for classes using this metaclass
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Settings(metaclass=Singleton):
    def __init__(self):
        print("  Settings.__init__ runs once")
print(Settings() is Settings(), type(Settings).__name__)

class UpperAttrs(type):
    def __new__(mcls, name, bases, ns):
        ns = {k if k.startswith("__") else k.upper(): v for k, v in ns.items()}
        return super().__new__(mcls, name, bases, ns)
class Constants(metaclass=UpperAttrs):
    pi = 3.14159
    answer = 42
print(Constants.PI, Constants.ANSWER, hasattr(Constants, "pi"))
```

```console
$ python3 metaclasses.py
<class '__main__.Point'> <class 'type'> 3 <class 'type'>
  Settings.__init__ runs once
True Singleton
3.14159 42 False
```

<!-- expanded -->
### Going deeper: `type()` builds classes, and when metaclasses are still needed

A `class` statement is sugar for calling the metaclass — by default `type(name, bases, namespace)`. Metaclasses remain the tool when you must control the class object itself: changing how the class namespace is collected (`__prepare__`), or customising behaviour on the class (not its instances), as `enum.Enum` does to make `Color["RED"]` and `len(Color)` work.

```python
# file: type_demo.py
def area(self):
    return self.w * self.h

Rect = type("Rect", (), {"__init__": lambda self, w, h: setattr(self, "w", w) or setattr(self, "h", h), "area": area})
print(Rect(3, 4).area(), type(Rect), Rect.__name__)

class CountingMeta(type):
    def __call__(cls, *args, **kwargs):              # runs when the CLASS is called: making an instance
        cls.instances = getattr(cls, "instances", 0) + 1
        return super().__call__(*args, **kwargs)
    def __len__(cls):                                # len() on the class itself
        return cls.instances

class Session(metaclass=CountingMeta):
    pass

Session(); Session(); Session()
print("sessions created:", len(Session))
```

```console
$ python3 type_demo.py
12 <class 'type'> Rect
sessions created: 3
```

## Topic 14C — Code as data

## Unit 14.5 — The `ast` module

**One-line summary:** `ast.parse(source)` returns the syntax tree; walk it with `ast.walk` or an `ast.NodeVisitor`; rewrite it with an `ast.NodeTransformer`; turn it back into code with `compile(...)` and run it, or into text with `ast.unparse`. Linters, formatters, test tools (pytest's assertion rewriting), and code generators all work this way.

```python
# file: ast_demo.py
import ast

src = '''
def area(w, h):
    print("computing")
    return w * h

total = area(3, 4) + area(1, 2)
'''
tree = ast.parse(src)
calls = [n.func.id for n in ast.walk(tree) if isinstance(n, ast.Call) and isinstance(n.func, ast.Name)]
print("function calls:", calls)
print("names assigned:", [t.id for n in ast.walk(tree) if isinstance(n, ast.Assign) for t in n.targets])

class StripPrints(ast.NodeTransformer):                # remove print(...) statements
    def visit_Expr(self, node):
        if isinstance(node.value, ast.Call) and getattr(node.value.func, "id", None) == "print":
            return None
        return node

class MulToAdd(ast.NodeTransformer):                   # a silly rewrite to show transformation
    def visit_BinOp(self, node):
        self.generic_visit(node)
        if isinstance(node.op, ast.Mult):
            node.op = ast.Add()
        return node

new = ast.fix_missing_locations(MulToAdd().visit(StripPrints().visit(tree)))
print(ast.unparse(new))
ns = {}
exec(compile(new, "<rewritten>", "exec"), ns)
print("total after rewrite:", ns["total"])
```

```console
$ python3 ast_demo.py
function calls: ['print', 'area', 'area']
names assigned: ['total']
def area(w, h):
    return w + h
total = area(3, 4) + area(1, 2)
total after rewrite: 10
```

### Deep & Tricky

**DT-31. `eval(expr, {"__builtins__": {}})` is not a sandbox: attribute access on literals reaches the whole object graph — `().__class__.__base__.__subclasses__()` lists every loaded class, including ones that can open files or run processes — so untrusted input must never be passed to `eval`/`exec`; use `ast.literal_eval` for data, or parse and whitelist AST nodes.** *(Python documentation, `eval`, and the security notes on `ast.literal_eval`)* The demo uses an empty builtins dict and still walks from a tuple to `object`, to all its subclasses, to the built-in importer — and imports the operating-system module, from which running commands is one more attribute away.

```python
# file: eval_escape.py
import ast
import warnings
warnings.simplefilter("ignore", DeprecationWarning)
payload = ("[c for c in ().__class__.__base__.__subclasses__() if c.__name__ == 'BuiltinImporter'][0]"
           ".load_module('posix').uname().sysname")      # import the OS module with no builtins at all
print("eval with empty __builtins__ reached the OS:", eval(payload, {"__builtins__": {}}))
print("literal_eval of data:", ast.literal_eval("{'a': [1, 2.5, (3, None)]}"))
try:
    ast.literal_eval(payload)
except ValueError as e:
    print("literal_eval refuses code:", type(e).__name__)
```

```console
$ python3 eval_escape.py
eval with empty __builtins__ reached the OS: Linux
literal_eval of data: {'a': [1, 2.5, (3, None)]}
literal_eval refuses code: ValueError
```

### Pattern

**P-15. Reach for the least powerful tool: a function or class first; then a decorator, `property`, or `dataclass`; then `__init_subclass__` or a descriptor for reusable per-class behaviour; a metaclass only when you must control class creation for an entire hierarchy; AST manipulation for tools that analyse code.** *(Practice: "explicit is better than implicit")* Metaprogramming saves repetition but hides control flow from readers, debuggers, and type checkers. *Trade-off:* frameworks (ORMs, serialisers, validators) justify the complexity because thousands of users benefit; application code usually does not.

### Mastery check

1. **Predict.** If a class defines both a `property` named `x` and `__getattr__`, and the property's getter raises `AttributeError`, what happens on `obj.x`?
2. **Predict.** What is `type(int)`, and what is `type(type)`?

## Chapter 14 Project — A safe expression evaluator

**Goal:** evaluate user-supplied formulas such as `price * qty * (1 - discount) if qty > 10 else price * qty` — arithmetic, comparisons, `and`/`or`/`not`, conditional expressions, named variables, and a whitelist of functions (`min`, `max`, `abs`, `round`) — **without** `eval`. Parse with `ast.parse(mode="eval")`, walk the tree, and evaluate only whitelisted node types; reject everything else (attribute access, subscripts, lambdas, comprehensions, calls to unknown names). Test against Python's own `eval` on random safe expressions, and against an attack list.

```python
# file: safeeval/safeeval.py
import ast
import operator
import random

BIN = {ast.Add: operator.add, ast.Sub: operator.sub, ast.Mult: operator.mul, ast.Div: operator.truediv,
       ast.FloorDiv: operator.floordiv, ast.Mod: operator.mod, ast.Pow: operator.pow}
UNARY = {ast.USub: operator.neg, ast.UAdd: operator.pos, ast.Not: operator.not_}
CMP = {ast.Lt: operator.lt, ast.LtE: operator.le, ast.Gt: operator.gt, ast.GtE: operator.ge,
       ast.Eq: operator.eq, ast.NotEq: operator.ne}
FUNCS = {"min": min, "max": max, "abs": abs, "round": round}

class UnsafeExpression(ValueError):
    pass

def evaluate(expr, variables, max_len=500):
    if len(expr) > max_len:
        raise UnsafeExpression("expression too long")
    tree = ast.parse(expr, mode="eval")

    def ev(node):
        match node:
            case ast.Expression(body=b):
                return ev(b)
            case ast.Constant(value=v) if isinstance(v, (int, float, bool)):
                return v
            case ast.Name(id=name) if name in variables:
                return variables[name]
            case ast.BinOp(left=l, op=op, right=r) if type(op) in BIN:
                left, right = ev(l), ev(r)
                if isinstance(op, ast.Pow) and abs(right) > 100:
                    raise UnsafeExpression("exponent too large")      # avoid CPU/memory bombs like 9**9**9
                return BIN[type(op)](left, right)
            case ast.UnaryOp(op=op, operand=x) if type(op) in UNARY:
                return UNARY[type(op)](ev(x))
            case ast.BoolOp(op=ast.And(), values=vs):
                result = True
                for v in vs:                                          # short-circuit like Python
                    result = ev(v)
                    if not result:
                        return result
                return result
            case ast.BoolOp(op=ast.Or(), values=vs):
                result = False
                for v in vs:
                    result = ev(v)
                    if result:
                        return result
                return result
            case ast.Compare(left=l, ops=ops, comparators=cs) if all(type(o) in CMP for o in ops):
                left = ev(l)
                for o, c in zip(ops, cs):                             # chained comparisons
                    right = ev(c)
                    if not CMP[type(o)](left, right):
                        return False
                    left = right
                return True
            case ast.IfExp(test=t, body=b, orelse=e):
                return ev(b) if ev(t) else ev(e)
            case ast.Call(func=ast.Name(id=fname), args=args, keywords=[]) if fname in FUNCS:
                return FUNCS[fname](*(ev(a) for a in args))
            case _:
                raise UnsafeExpression(f"not allowed: {ast.dump(node)[:60]}")
    return ev(tree)

env = {"price": 12.5, "qty": 14, "discount": 0.1}
formula = "price * qty * (1 - discount) if qty > 10 else price * qty"
print(f"{formula}  =>  {evaluate(formula, env)}")
print("max(price, 20) // 3 =", evaluate("max(price, 20) // 3", env), "| 1 < qty <= 20 =", evaluate("1 < qty <= 20", env))

attacks = ["__import__('os').system('echo pwned')", "().__class__.__base__.__subclasses__()", "open('/etc/passwd').read()",
           "[x for x in range(10)]", "(lambda: 1)()", "price.__class__", "9 ** 9 ** 9", "exec('1')", "env['price']"]
for a in attacks:
    try:
        evaluate(a, env)
        print("NOT BLOCKED:", a)
    except (UnsafeExpression, SyntaxError) as e:
        print(f"blocked: {a[:42]:<42} ({type(e).__name__})")

rng = random.Random(14)
def rand_expr(depth=0):
    if depth > 3 or rng.random() < 0.3:
        return rng.choice(["price", "qty", "discount", str(rng.randint(-5, 9)), "2.5", "True"])
    kind = rng.random()
    if kind < 0.45:
        return f"({rand_expr(depth + 1)} {rng.choice(['+', '-', '*', '<', '>=', '==', 'and', 'or'])} {rand_expr(depth + 1)})"
    if kind < 0.6:
        return f"(not {rand_expr(depth + 1)})"
    if kind < 0.8:
        return f"({rand_expr(depth + 1)} if {rand_expr(depth + 1)} else {rand_expr(depth + 1)})"
    return f"{rng.choice(['min', 'max'])}({rand_expr(depth + 1)}, {rand_expr(depth + 1)})"
agree = 0
for _ in range(3000):
    e = rand_expr()
    try:
        expected = eval(e, {"__builtins__": {}, "min": min, "max": max}, env)   # trusted, generated input only
    except Exception as ex:
        expected = type(ex)
    try:
        got = evaluate(e, env)
    except Exception as ex:
        got = type(ex)
    agree += got == expected
print(f"3,000 random expressions: agreement with Python's eval on {agree}")
```

```console
$ cd safeeval && python3 safeeval.py
price * qty * (1 - discount) if qty > 10 else price * qty  =>  157.5
max(price, 20) // 3 = 6 | 1 < qty <= 20 = True
blocked: __import__('os').system('echo pwned')      (UnsafeExpression)
blocked: ().__class__.__base__.__subclasses__()     (UnsafeExpression)
blocked: open('/etc/passwd').read()                 (UnsafeExpression)
blocked: [x for x in range(10)]                     (UnsafeExpression)
blocked: (lambda: 1)()                              (UnsafeExpression)
blocked: price.__class__                            (UnsafeExpression)
blocked: 9 ** 9 ** 9                                (UnsafeExpression)
blocked: exec('1')                                  (UnsafeExpression)
blocked: env['price']                               (UnsafeExpression)
3,000 random expressions: agreement with Python's eval on 3000
```

The evaluator accepts exactly the node types it understands — a **whitelist** — so anything new or unexpected is rejected by default; a blacklist ("forbid `__`, forbid `import`") always misses something. The differential test against `eval` (used here only on expressions the test itself generated) checks that the whitelisted subset has Python's semantics, including short-circuiting and chained comparisons.

### Review checklist

- Why must `and`/`or` evaluate their operands lazily rather than evaluating all of them first?
- What resource-exhaustion attacks remain (very deep nesting, huge multiplications), and how would you bound them?
- Why is `ast.parse` itself safe to call on untrusted input, while `compile` + `exec` is not?

## Answers to the Chapter 14 prediction questions

**14.5 Q1.** `__getattr__` is called: when `__getattribute__` (here, via the property) raises `AttributeError`, Python falls back to `__getattr__` — which can hide bugs inside property getters.

**14.5 Q2.** `type(int)` is `type` (classes are instances of their metaclass), and `type(type)` is `type` itself.

# Chapter 15 — Databases and Web Services

*Goal: store data in a relational database with `sqlite3` — safely parameterised queries, transactions, indexes — and expose it as an HTTP JSON API with FastAPI and pydantic: request validation, status codes, dependency injection, automated tests with a test client, and a real server process called by a client.*

**What you will be able to do after this chapter**

- Create SQLite schemas, insert and query with parameters, and use transactions correctly.
- Explain SQL injection and prevent it; read a query plan and add an index.
- Build a FastAPI application with typed request and response models.
- Inject dependencies (a database connection per request) and return proper HTTP errors.
- Test the API in-process, and run it with uvicorn and call it over HTTP.

## Topic 15A — SQL with `sqlite3`

## Unit 15.1 — Connections, queries, and parameters

**One-line summary:** `sqlite3.connect(path)` opens (or creates) a database file (`":memory:"` for a temporary one). Execute SQL with `conn.execute(sql, params)`; always pass values as **parameters** (`?` or `:name` placeholders), never by formatting them into the SQL string. Set `conn.row_factory = sqlite3.Row` to access columns by name.

```python
# file: sql_basics.py
import sqlite3

conn = sqlite3.connect(":memory:")
conn.row_factory = sqlite3.Row
conn.executescript("""
    CREATE TABLE books (id INTEGER PRIMARY KEY, title TEXT NOT NULL, author TEXT NOT NULL,
                        year INTEGER, price REAL CHECK (price >= 0));
""")
books = [("Dune", "Herbert", 1965, 9.99), ("Neuromancer", "Gibson", 1984, 7.5),
         ("Hyperion", "Simmons", 1989, 8.25), ("Foundation", "Asimov", 1951, 6.0)]
conn.executemany("INSERT INTO books (title, author, year, price) VALUES (?, ?, ?, ?)", books)
conn.commit()
for row in conn.execute("SELECT title, year FROM books WHERE year > ? ORDER BY year", (1960,)):
    print(dict(row))
r = conn.execute("SELECT COUNT(*) AS n, ROUND(AVG(price), 2) AS avg_price FROM books").fetchone()
print("count:", r["n"], "| average price:", r["avg_price"])
try:
    conn.execute("INSERT INTO books (title, author, price) VALUES (?, ?, ?)", ("Free?", "X", -1))
except sqlite3.IntegrityError as e:
    print("IntegrityError:", e)
```

```console
$ python3 sql_basics.py
{'title': 'Dune', 'year': 1965}
{'title': 'Neuromancer', 'year': 1984}
{'title': 'Hyperion', 'year': 1989}
count: 4 | average price: 7.94
IntegrityError: CHECK constraint failed: price >= 0
```

### Deep & Tricky

**DT-32. Building SQL with f-strings or `%` lets input change the query's structure — SQL injection — and escaping by hand is never complete; parameter binding sends values separately from the SQL text, so they can never be interpreted as SQL. Identifiers (table and column names) cannot be parameters: validate them against a whitelist.** *(Python documentation, `sqlite3`: "How to use placeholders to bind values in SQL queries… This is the only safe way"; OWASP SQL Injection Prevention Cheat Sheet)*

```python
# file: injection.py
import sqlite3

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE users (name TEXT, is_admin INTEGER)")
conn.executemany("INSERT INTO users VALUES (?, ?)", [("ana", 1), ("ben", 0)])
attack = "nobody' OR '1'='1"
unsafe = conn.execute(f"SELECT name FROM users WHERE name = '{attack}'").fetchall()
safe = conn.execute("SELECT name FROM users WHERE name = ?", (attack,)).fetchall()
print("f-string query returned:", unsafe)
print("parameterised query returned:", safe)
```

```console
$ python3 injection.py
f-string query returned: [('ana',), ('ben',)]
parameterised query returned: []
```

## Unit 15.2 — Transactions and indexes

**One-line summary:** A **transaction** makes a group of statements atomic: all succeed (`COMMIT`) or none take effect (`ROLLBACK`). Using the connection as a context manager (`with conn:`) commits on success and rolls back on an exception. An **index** lets the database find rows without scanning the whole table; `EXPLAIN QUERY PLAN` shows whether a query uses one.

```python
# file: transactions.py
import sqlite3
import time

conn = sqlite3.connect(":memory:")
conn.execute("CREATE TABLE accounts (name TEXT PRIMARY KEY, balance INTEGER CHECK (balance >= 0))")
conn.executemany("INSERT INTO accounts VALUES (?, ?)", [("ana", 100), ("ben", 50)])
conn.commit()

def transfer(src, dst, amount):
    try:
        with conn:                                       # one transaction
            conn.execute("UPDATE accounts SET balance = balance + ? WHERE name = ?", (amount, dst))
            conn.execute("UPDATE accounts SET balance = balance - ? WHERE name = ?", (amount, src))
        return "ok"
    except sqlite3.IntegrityError as e:
        return f"rolled back ({e})"

print(transfer("ana", "ben", 30), dict(conn.execute("SELECT * FROM accounts")))
print(transfer("ben", "ana", 500), dict(conn.execute("SELECT * FROM accounts")))   # the credit to ana is undone too

conn.execute("CREATE TABLE events (id INTEGER PRIMARY KEY, user_id INTEGER, kind TEXT)")
conn.executemany("INSERT INTO events (user_id, kind) VALUES (?, ?)", ((i % 5000, "click") for i in range(300_000)))
query = "SELECT COUNT(*) FROM events WHERE user_id = ?"
for label in ("without index", "with index"):
    if label == "with index":
        conn.execute("CREATE INDEX idx_events_user ON events(user_id)")
    plan = conn.execute("EXPLAIN QUERY PLAN " + query, (42,)).fetchall()[0][-1]
    t = time.perf_counter()
    for uid in range(200):
        conn.execute(query, (uid,)).fetchone()
    print(f"{label:>14}: plan '{plan}', 200 queries in {(time.perf_counter() - t) * 1000:.0f} ms")
```

```console
$ python3 transactions.py
ok {'ana': 70, 'ben': 80}
rolled back (CHECK constraint failed: balance >= 0) {'ana': 70, 'ben': 80}
 without index: plan 'SCAN events', 200 queries in 1201 ms
    with index: plan 'SEARCH events USING COVERING INDEX idx_events_user (user_id=?)', 200 queries in 0 ms
```

<!-- expanded -->
### Going deeper: `executemany`, one transaction, and what an index changes

Each `commit` makes SQLite wait for the disk. Inserting rows one transaction at a time is therefore slow; putting them all in one transaction with `executemany` is the standard fix. And `EXPLAIN QUERY PLAN` shows whether a query scans the whole table or uses an index.

```python
# file: bulk.py
import sqlite3
import time

db = sqlite3.connect("bulk.db")
db.execute("DROP TABLE IF EXISTS events")
db.execute("CREATE TABLE events (id INTEGER PRIMARY KEY, user TEXT, kind TEXT, ts INTEGER)")
rows = [(f"user{i % 500}", "click" if i % 3 else "view", i) for i in range(20_000)]

t0 = time.perf_counter()
for r in rows[:500]:
    db.execute("INSERT INTO events (user, kind, ts) VALUES (?, ?, ?)", r)
    db.commit()                                         # one transaction per row
per_row = (time.perf_counter() - t0) / 500

t0 = time.perf_counter()
with db:                                                # one transaction for everything
    db.executemany("INSERT INTO events (user, kind, ts) VALUES (?, ?, ?)", rows[500:])
batched = (time.perf_counter() - t0) / (len(rows) - 500)
print(f"per-row commits are more than 10x slower per row: {per_row > 10 * batched}")

query = "SELECT count(*) FROM events WHERE user = ?"
print("plan before index:", db.execute("EXPLAIN QUERY PLAN " + query, ("user7",)).fetchone()[-1])
db.execute("CREATE INDEX idx_events_user ON events(user)")
print("plan after index: ", db.execute("EXPLAIN QUERY PLAN " + query, ("user7",)).fetchone()[-1])
db.row_factory = sqlite3.Row                            # rows usable by column name
row = db.execute("SELECT user, count(*) AS n FROM events GROUP BY user ORDER BY n DESC, user LIMIT 1").fetchone()
print(dict(row))
```

```console
$ python3 bulk.py
per-row commits are more than 10x slower per row: True
plan before index: SCAN events
plan after index:  SEARCH events USING COVERING INDEX idx_events_user (user=?)
{'user': 'user0', 'n': 40}
```

`SCAN events` means every row is read; `SEARCH events USING COVERING INDEX idx_events_user (user=?)` means SQLite jumps straight to the matching rows — and "covering" means the index alone answers the query, without touching the table.

## Topic 15B — HTTP APIs

## Unit 15.3 — A FastAPI application

**One-line summary:** FastAPI maps **path operations** (`@app.get("/items/{id}")`) to Python functions; parameters are taken from the path, query string, or JSON body according to their type annotations, and **pydantic models** validate request bodies and shape responses. Invalid input gets an automatic `422` response describing the problem; raise `HTTPException(status_code=404, ...)` for application errors.

```python
# file: api/app.py
import sqlite3
from fastapi import Depends, FastAPI, HTTPException, Query
from pydantic import BaseModel, Field

DB_PATH = "notes.db"

class NoteIn(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    body: str = ""
    tags: list[str] = []

class Note(NoteIn):
    id: int

app = FastAPI(title="Notes API")

def get_db():                                          # a dependency: one connection per request
    # FastAPI may run this generator's setup, the endpoint, and the cleanup in DIFFERENT pool threads,
    # so sqlite3's same-thread check must be disabled; the connection is still used by one request at a time.
    conn = sqlite3.connect(DB_PATH, check_same_thread=False)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()

def init_db(path=DB_PATH):
    with sqlite3.connect(path) as conn:
        conn.execute("CREATE TABLE IF NOT EXISTS notes (id INTEGER PRIMARY KEY, title TEXT, body TEXT, tags TEXT)")

def to_note(row):
    return Note(id=row["id"], title=row["title"], body=row["body"], tags=[t for t in row["tags"].split(",") if t])

@app.post("/notes", response_model=Note, status_code=201)
def create_note(note: NoteIn, db: sqlite3.Connection = Depends(get_db)):
    with db:
        cur = db.execute("INSERT INTO notes (title, body, tags) VALUES (?, ?, ?)", (note.title, note.body, ",".join(note.tags)))
    return Note(id=cur.lastrowid, **note.model_dump())

@app.get("/notes/{note_id}", response_model=Note)
def read_note(note_id: int, db: sqlite3.Connection = Depends(get_db)):
    row = db.execute("SELECT * FROM notes WHERE id = ?", (note_id,)).fetchone()
    if row is None:
        raise HTTPException(status_code=404, detail=f"note {note_id} not found")
    return to_note(row)

@app.get("/notes", response_model=list[Note])
def list_notes(tag: str | None = None, limit: int = Query(10, ge=1, le=100), db: sqlite3.Connection = Depends(get_db)):
    rows = db.execute("SELECT * FROM notes ORDER BY id").fetchall()
    notes = [to_note(r) for r in rows]
    if tag:
        notes = [n for n in notes if tag in n.tags]
    return notes[:limit]

@app.delete("/notes/{note_id}", status_code=204)
def delete_note(note_id: int, db: sqlite3.Connection = Depends(get_db)):
    with db:
        if db.execute("DELETE FROM notes WHERE id = ?", (note_id,)).rowcount == 0:
            raise HTTPException(status_code=404, detail=f"note {note_id} not found")
```

## Unit 15.4 — Testing the API in-process

**One-line summary:** `fastapi.testclient.TestClient` sends requests to the application **without a network** — fast and deterministic. Override dependencies (`app.dependency_overrides`) or configuration to point tests at a temporary database, and assert on status codes and JSON bodies.

```python
# file: api/test_app.py
import pytest
from fastapi.testclient import TestClient
import app as app_module

@pytest.fixture
def client(tmp_path, monkeypatch):
    db = tmp_path / "test.db"
    monkeypatch.setattr(app_module, "DB_PATH", str(db))   # get_db reads the module global at call time
    app_module.init_db(str(db))
    return TestClient(app_module.app)

def test_create_and_read(client):
    r = client.post("/notes", json={"title": "Buy milk", "tags": ["home"]})
    assert r.status_code == 201
    note = r.json()
    assert note["id"] == 1 and note["tags"] == ["home"]
    assert client.get(f"/notes/{note['id']}").json()["title"] == "Buy milk"

def test_validation_error(client):
    r = client.post("/notes", json={"title": ""})
    assert r.status_code == 422
    assert r.json()["detail"][0]["loc"] == ["body", "title"]

def test_not_found_and_delete(client):
    assert client.get("/notes/99").status_code == 404
    nid = client.post("/notes", json={"title": "temp"}).json()["id"]
    assert client.delete(f"/notes/{nid}").status_code == 204
    assert client.delete(f"/notes/{nid}").status_code == 404

def test_filter_and_limit(client):
    for i in range(5):
        client.post("/notes", json={"title": f"n{i}", "tags": ["even"] if i % 2 == 0 else []})
    assert [n["title"] for n in client.get("/notes", params={"tag": "even"}).json()] == ["n0", "n2", "n4"]
    assert len(client.get("/notes", params={"limit": 2}).json()) == 2
    assert client.get("/notes", params={"limit": 0}).status_code == 422
```

```console
$ cd api && python3 -m pytest -q -p no:cacheprovider test_app.py 2>&1 | tail -1
4 passed in 0.49s
```

### Deep & Tricky

**DT-33. In FastAPI, a path operation defined with plain `def` runs in a **thread pool**, while one defined with `async def` runs **on the event loop** — so calling blocking code (such as `sqlite3` or `requests`) inside an `async def` endpoint stalls every request in the process (Chapter 11, Unit 11.5), whereas the same code in a plain `def` endpoint is fine.** *(FastAPI documentation, "Concurrency and async / await": "When you declare a path operation function with normal def instead of async def, it is run in an external threadpool")* Use `async def` only when everything awaited inside is truly asynchronous (an async database driver, `httpx.AsyncClient`); otherwise use `def`. This chapter's endpoints use `def` because `sqlite3` is blocking. A related surprise, found while running this chapter's project: a `def` **generator dependency** such as `get_db` may run its setup, the endpoint, and its cleanup in *different* worker threads, so a connection opened with the default `check_same_thread=True` fails with "SQLite objects created in a thread can only be used in that same thread" — but only under a real server with concurrent requests; the in-process tests passed. Opening the connection with `check_same_thread=False` is correct here because each connection still serves one request at a time.

### Pattern

**P-16. Validate at the edges and keep the core typed: pydantic models for HTTP input and output, parameterised SQL for every query, one transaction per business operation, dependencies for resources (connections, settings, the current user), and in-process tests with dependency or configuration overrides.** *(Practice)* Return the right status codes (201 created, 204 no content, 404 not found, 422 invalid input) — clients depend on them. *Trade-off:* SQLite is excellent for single-machine applications, tests, and prototypes; for many concurrent writers or multiple servers, use a client–server database (PostgreSQL) with a connection pool — the code structure stays the same.

### Mastery check

1. **Predict.** What does `conn.execute("SELECT ?", ("1; DROP TABLE users",)).fetchone()` return?
2. **Predict.** What status code does FastAPI return when `/notes/abc` is requested for a path parameter typed `int`?

## Chapter 15 Project — Run the service for real

**Goal:** start the Notes API in a real **uvicorn** server process, wait until it is up, exercise it from a separate client over HTTP with `httpx` — create notes concurrently, read, filter, delete, and trigger validation and not-found errors — print the exchanges, and shut the server down cleanly.

```python
# file: api/client.py
import concurrent.futures
import os
import subprocess
import sys
import time
import httpx

PORT = 8799
BASE = f"http://127.0.0.1:{PORT}"
if os.path.exists("notes.db"):
    os.remove("notes.db")
subprocess.run([sys.executable, "-c", "import app; app.init_db()"], check=True)
server = subprocess.Popen([sys.executable, "-m", "uvicorn", "app:app", "--port", str(PORT), "--log-level", "warning"])
try:
    for _ in range(100):                                  # wait until the server answers
        try:
            httpx.get(f"{BASE}/notes", timeout=0.2)
            break
        except httpx.TransportError:
            time.sleep(0.1)
    with httpx.Client(base_url=BASE) as c:
        with concurrent.futures.ThreadPoolExecutor(8) as pool:
            created = list(pool.map(lambda i: c.post("/notes", json={"title": f"note {i}", "tags": ["work" if i % 3 == 0 else "home"]}),
                                    range(12)))
        print("12 concurrent POSTs:", sorted({r.status_code for r in created}), "| ids:", sorted(r.json()["id"] for r in created))
        print("GET /notes/5      ->", c.get("/notes/5").json())
        print("GET /notes?tag=work ->", [n["title"] for n in c.get("/notes", params={"tag": "work", "limit": 50}).json()])
        r = c.post("/notes", json={"title": "x" * 200})
        print("POST too-long title ->", r.status_code, r.json()["detail"][0]["msg"])
        print("DELETE /notes/5   ->", c.delete("/notes/5").status_code, "| again ->", c.delete("/notes/5").status_code)
        print("GET /notes/abc    ->", c.get("/notes/abc").status_code)
        print("OpenAPI schema paths:", sorted(c.get("/openapi.json").json()["paths"]))
finally:
    server.terminate()
    server.wait(timeout=10)
    print("server stopped with exit code", server.returncode)
    os.remove("notes.db")
```

```console
$ cd api && python3 client.py
12 concurrent POSTs: [201] | ids: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
GET /notes/5      -> {'title': 'note 6', 'body': '', 'tags': ['work'], 'id': 5}
GET /notes?tag=work -> ['note 3', 'note 6', 'note 0', 'note 9']
POST too-long title -> 422 String should have at most 100 characters
DELETE /notes/5   -> 204 | again -> 404
GET /notes/abc    -> 422
OpenAPI schema paths: ['/notes', '/notes/{note_id}']
server stopped with exit code -15
```

The twelve concurrent requests run in uvicorn's thread pool (the endpoints are plain `def`), each with its own SQLite connection from the `get_db` dependency; SQLite serialises the writes, so every insert gets a distinct id. FastAPI also generated an OpenAPI description of the API from the type annotations — the same annotations that drive validation.

### Review checklist

- Why does the client wait for the server with a retry loop instead of a fixed `sleep`?
- What would change if `create_note` were declared `async def` (see DT-33)?
- How would you add authentication as a dependency that every endpoint requires?

## Answers to the Chapter 15 prediction questions

**15.5 Q1.** A one-column row containing the string `'1; DROP TABLE users'`: the parameter is a value, never SQL.

**15.5 Q2.** `422 Unprocessable Entity` (newer Starlette versions call it "Unprocessable Content"), with a body explaining that the path parameter is not a valid integer.

# Chapter 16 — Capstone: A Small Search Engine

*Goal: build a complete, professional-quality Python application that uses the whole book — a package with a `src/` layout and `pyproject.toml`, an asynchronous web crawler, an HTML parser, an inverted index in SQLite with TF-IDF ranking, a FastAPI search service, a command-line interface — guarded by type checking (`mypy --strict`) and a pytest suite, and exercised end to end against a local website.*

**What you will build**

`minisearch` crawls a website, indexes its pages, and answers queries:

```text
minisearch crawl http://127.0.0.1:8901/ --db site.db --max-pages 200 --concurrency 8
minisearch search "binary search tree" --db site.db
minisearch serve --db site.db --port 8902          # GET /search?q=...
```

| Component | Module | Book chapters |
| --- | --- | --- |
| Data types | `models.py` (frozen dataclasses) | 4, 8 |
| HTML parsing, URL normalisation | `parse.py` (`html.parser`, `urllib.parse`) | 9, 12 |
| Asynchronous crawler | `crawl.py` (`asyncio`, `httpx`, semaphore, TaskGroup) | 11 |
| Index and ranking | `index.py` (`sqlite3`, transactions, TF-IDF) | 2, 15 |
| HTTP API | `api.py` (FastAPI, pydantic) | 15 |
| CLI and packaging | `cli.py`, `pyproject.toml`, entry point | 7, 12 |
| Tests and types | `tests/`, `mypy --strict` | 8, 10 |
| Test website | `testsite.py` (generated pages, local server) | 9, 11 |

## Topic 16A — The pieces

## Unit 16.1 — Project layout, models, and parsing

**One-line summary:** Start with the data: frozen dataclasses for a fetched `Page` and a search `Hit`. Parsing extracts the title, visible text, and links from HTML with the standard library's `HTMLParser`, and **normalises** each link (resolve relative URLs, drop fragments, keep only http(s) on the same host) so that the crawler visits every page exactly once.

```text
# file: minisearch/pyproject.toml
[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.build_meta"

[project]
name = "minisearch"
version = "1.0.0"
requires-python = ">=3.12"
dependencies = ["httpx>=0.27", "fastapi>=0.110", "uvicorn>=0.29"]

[project.scripts]
minisearch = "minisearch.cli:main"

[tool.setuptools.package-data]
minisearch = ["py.typed"]

[tool.mypy]
strict = true
```

```python
# file: minisearch/src/minisearch/__init__.py
"""minisearch: crawl, index, and search a website."""
__version__ = "1.0.0"
```

```text
# file: minisearch/src/minisearch/py.typed
```

The empty `py.typed` file is the PEP 561 marker that tells type checkers "this installed package ships its own type information"; without it, `mypy` treats the installed `minisearch` as untyped and refuses to analyse imports of it from the tests (a failure this chapter's first run produced).

```python
# file: minisearch/src/minisearch/models.py
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class Page:
    url: str
    title: str
    text: str
    links: tuple[str, ...]


@dataclass(frozen=True, slots=True)
class Hit:
    url: str
    title: str
    score: float
    snippet: str
```

```python
# file: minisearch/src/minisearch/parse.py
import re
from html.parser import HTMLParser
from urllib.parse import urldefrag, urljoin, urlsplit

from .models import Page

WORD = re.compile(r"[a-z0-9]+")
SKIP_TAGS = {"script", "style", "noscript"}


def tokenize(text: str) -> list[str]:
    return WORD.findall(text.lower())


def normalise_link(base: str, href: str) -> str | None:
    """Absolute http(s) URL on the same host as base, without fragment; None if not crawlable."""
    url, _fragment = urldefrag(urljoin(base, href.strip()))
    parts, base_parts = urlsplit(url), urlsplit(base)
    if parts.scheme not in ("http", "https") or parts.netloc != base_parts.netloc:
        return None
    path = parts.path or "/"
    return f"{parts.scheme}://{parts.netloc}{path}" + (f"?{parts.query}" if parts.query else "")


class _Extractor(HTMLParser):
    def __init__(self) -> None:
        super().__init__(convert_charrefs=True)
        self.title_parts: list[str] = []
        self.text_parts: list[str] = []
        self.hrefs: list[str] = []
        self._in_title = False
        self._skip_depth = 0
        self._in_link = 0

    def handle_starttag(self, tag: str, attrs: list[tuple[str, str | None]]) -> None:
        if tag in SKIP_TAGS:
            self._skip_depth += 1
        elif tag == "title":
            self._in_title = True
        elif tag == "a":
            self._in_link += 1
            href = dict(attrs).get("href")
            if href:
                self.hrefs.append(href)

    def handle_endtag(self, tag: str) -> None:
        if tag in SKIP_TAGS and self._skip_depth:
            self._skip_depth -= 1
        elif tag == "title":
            self._in_title = False
        elif tag == "a" and self._in_link:
            self._in_link -= 1

    def handle_data(self, data: str) -> None:
        if self._skip_depth or self._in_link:           # skip scripts and navigation (anchor) text
            return
        (self.title_parts if self._in_title else self.text_parts).append(data)


def parse_page(url: str, html: str) -> Page:
    ex = _Extractor()
    ex.feed(html)
    ex.close()
    links: list[str] = []
    for href in ex.hrefs:
        link = normalise_link(url, href)
        if link and link not in links:
            links.append(link)
    text = " ".join(" ".join(ex.text_parts).split())
    title = " ".join("".join(ex.title_parts).split()) or url
    return Page(url=url, title=title, text=text, links=tuple(links))
```

### Deep & Tricky

**DT-34. `urljoin` follows RFC 3986 exactly, which surprises people: a base URL without a trailing slash is treated as a *file*, so `urljoin("http://h/docs", "intro.html")` is `http://h/intro.html`, not `http://h/docs/intro.html`; and `#section` links, `?query` variants, `mailto:`, and `javascript:` hrefs must be normalised or filtered, or a crawler will fetch the same page under many URLs or try to fetch things that are not pages.** *(RFC 3986 §5.2 "Relative Resolution"; Python documentation, `urllib.parse.urljoin`)* The test suite below pins these cases down, because a crawler's correctness — every page once, nothing outside the site — depends on them.

## Unit 16.2 — The asynchronous crawler

**One-line summary:** Breadth-first crawl with `asyncio`: a queue of URLs, a `seen` set, a semaphore limiting concurrent requests, an `httpx.AsyncClient` with a timeout, and a `TaskGroup` of workers. Only `text/html` responses with status 200 are parsed; failures are recorded, not fatal. The crawl stops at `max_pages`.

```python
# file: minisearch/src/minisearch/crawl.py
import asyncio
from dataclasses import dataclass, field

import httpx

from .models import Page
from .parse import normalise_link, parse_page


@dataclass
class CrawlResult:
    pages: list[Page] = field(default_factory=list)
    errors: dict[str, str] = field(default_factory=dict)


async def crawl(start: str, max_pages: int = 100, concurrency: int = 8, timeout: float = 5.0) -> CrawlResult:
    root = normalise_link(start, start)
    if root is None:
        raise ValueError(f"not an http(s) URL: {start!r}")
    result = CrawlResult()
    seen: set[str] = {root}
    queue: asyncio.Queue[str] = asyncio.Queue()
    queue.put_nowait(root)

    async with httpx.AsyncClient(timeout=timeout, follow_redirects=True) as client:

        async def worker() -> None:
            while len(result.pages) < max_pages:
                try:
                    url = await asyncio.wait_for(queue.get(), timeout=0.5)
                except TimeoutError:
                    return                                  # queue stayed empty: crawl is finished
                try:
                    resp = await client.get(url)
                    if resp.status_code != 200 or "text/html" not in resp.headers.get("content-type", ""):
                        result.errors[url] = f"HTTP {resp.status_code} {resp.headers.get('content-type', '')}"
                        continue
                    page = parse_page(str(resp.url), resp.text)
                    if len(result.pages) < max_pages:
                        result.pages.append(page)
                    for link in page.links:
                        if link not in seen:
                            seen.add(link)
                            queue.put_nowait(link)
                except httpx.HTTPError as e:
                    result.errors[url] = type(e).__name__
                finally:
                    queue.task_done()

        async with asyncio.TaskGroup() as tg:
            for _ in range(concurrency):
                tg.create_task(worker())
    return result
```

## Unit 16.3 — The index and TF-IDF ranking

**One-line summary:** Store pages and an **inverted index** (`term → (page, count)`) in SQLite, written in one transaction. Rank with **TF-IDF**: a term's weight in a page grows with its frequency there (normalised by page length) and with its rarity across pages (`idf = log(N / df)`); a page's score is the sum over query terms, with a bonus for title matches. Snippets show the first occurrence of a query term.

```python
# file: minisearch/src/minisearch/index.py
import math
import sqlite3
from collections import Counter
from collections.abc import Iterable

from .models import Hit, Page
from .parse import tokenize

SCHEMA = """
CREATE TABLE IF NOT EXISTS pages (id INTEGER PRIMARY KEY, url TEXT UNIQUE, title TEXT, text TEXT, length INTEGER);
CREATE TABLE IF NOT EXISTS postings (term TEXT, page_id INTEGER, count INTEGER, PRIMARY KEY (term, page_id));
CREATE INDEX IF NOT EXISTS idx_postings_term ON postings(term);
"""


def connect(path: str) -> sqlite3.Connection:
    conn = sqlite3.connect(path, check_same_thread=False)
    conn.executescript(SCHEMA)
    return conn


def add_pages(conn: sqlite3.Connection, pages: Iterable[Page]) -> int:
    n = 0
    with conn:                                               # one transaction for the whole batch
        for page in pages:
            tokens = tokenize(page.title + " " + page.text)
            # upsert that KEEPS the row id (INSERT OR REPLACE would delete the row and assign a new id,
            # orphaning the old postings - the test suite caught exactly that)
            page_id = conn.execute(
                "INSERT INTO pages (url, title, text, length) VALUES (?, ?, ?, ?) "
                "ON CONFLICT(url) DO UPDATE SET title = excluded.title, text = excluded.text, length = excluded.length "
                "RETURNING id", (page.url, page.title, page.text, len(tokens))).fetchone()[0]
            conn.execute("DELETE FROM postings WHERE page_id = ?", (page_id,))
            conn.executemany("INSERT INTO postings (term, page_id, count) VALUES (?, ?, ?)",
                             ((t, page_id, c) for t, c in Counter(tokens).items()))
            n += 1
    return n


def snippet(text: str, terms: list[str], width: int = 80) -> str:
    low = text.lower()
    pos = min((p for t in terms if (p := low.find(t)) >= 0), default=0)
    start = max(0, pos - width // 4)
    return ("…" if start else "") + text[start:start + width].strip() + ("…" if start + width < len(text) else "")


def search(conn: sqlite3.Connection, query: str, limit: int = 10) -> list[Hit]:
    terms = list(dict.fromkeys(tokenize(query)))
    if not terms:
        return []
    total = conn.execute("SELECT COUNT(*) FROM pages").fetchone()[0]
    scores: Counter[int] = Counter()
    matched: Counter[int] = Counter()                       # how many distinct query terms each page contains
    for term in terms:
        rows = conn.execute("SELECT p.page_id, p.count, pg.length FROM postings p JOIN pages pg ON pg.id = p.page_id "
                            "WHERE p.term = ?", (term,)).fetchall()
        if not rows:
            continue
        idf = math.log(total / len(rows)) + 1.0
        for page_id, count, length in rows:
            scores[page_id] += (count / max(length, 1)) * idf
            matched[page_id] += 1
    hits: list[Hit] = []
    for page_id, score in scores.most_common():
        url, title, text = conn.execute("SELECT url, title, text FROM pages WHERE id = ?", (page_id,)).fetchone()
        coverage = matched[page_id] / len(terms)            # pages matching more of the query rank higher
        bonus = 0.25 * sum(t in tokenize(title) for t in terms)
        final = score * coverage ** 2 * (1 + bonus)
        hits.append(Hit(url=url, title=title, score=round(final, 6), snippet=snippet(text, terms)))
    hits.sort(key=lambda h: (-h.score, h.url))
    return hits[:limit]
```

## Unit 16.4 — The API and the command line

**One-line summary:** The FastAPI app opens the index named by an environment variable and exposes `GET /search?q=&limit=` with validated parameters and a typed response model. The CLI uses `argparse` sub-commands — `crawl`, `search`, `serve` — and returns an exit status.

```python
# file: minisearch/src/minisearch/api.py
import os
from typing import Annotated

from fastapi import FastAPI, Query
from pydantic import BaseModel

from .index import connect, search

app = FastAPI(title="minisearch")


class HitOut(BaseModel):
    url: str
    title: str
    score: float
    snippet: str


@app.get("/search", response_model=list[HitOut])
def search_endpoint(q: Annotated[str, Query(min_length=1, max_length=200)],
                    limit: Annotated[int, Query(ge=1, le=50)] = 10) -> list[HitOut]:
    conn = connect(os.environ.get("MINISEARCH_DB", "minisearch.db"))
    try:
        return [HitOut(url=h.url, title=h.title, score=h.score, snippet=h.snippet) for h in search(conn, q, limit)]
    finally:
        conn.close()
```

```python
# file: minisearch/src/minisearch/cli.py
import argparse
import asyncio
import os
import sys
import time

from .crawl import crawl
from .index import add_pages, connect, search


def main(argv: list[str] | None = None) -> int:
    p = argparse.ArgumentParser(prog="minisearch", description="Crawl, index, and search a website.")
    sub = p.add_subparsers(dest="command", required=True)
    c = sub.add_parser("crawl", help="crawl a site and index it")
    c.add_argument("url")
    c.add_argument("--db", default="minisearch.db")
    c.add_argument("--max-pages", type=int, default=100)
    c.add_argument("--concurrency", type=int, default=8)
    s = sub.add_parser("search", help="query the index")
    s.add_argument("query")
    s.add_argument("--db", default="minisearch.db")
    s.add_argument("-n", type=int, default=5)
    v = sub.add_parser("serve", help="run the HTTP API")
    v.add_argument("--db", default="minisearch.db")
    v.add_argument("--port", type=int, default=8000)
    args = p.parse_args(argv)

    if args.command == "crawl":
        t = time.perf_counter()
        result = asyncio.run(crawl(args.url, args.max_pages, args.concurrency))
        n = add_pages(connect(args.db), result.pages)
        print(f"crawled {n} pages ({len(result.errors)} errors) in {time.perf_counter() - t:.2f} s")
        for url, err in sorted(result.errors.items())[:5]:
            print(f"  error: {url} -> {err}")
        return 0
    if args.command == "search":
        hits = search(connect(args.db), args.query, args.n)
        if not hits:
            print("no results")
            return 1
        for i, h in enumerate(hits, 1):
            print(f"{i}. {h.title}  [{h.score:.4f}]\n   {h.url}\n   {h.snippet}")
        return 0
    os.environ["MINISEARCH_DB"] = args.db
    import uvicorn
    uvicorn.run("minisearch.api:app", port=args.port, log_level="warning")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

## Topic 16B — Proving it works

## Unit 16.5 — A test website, tests, and type checking

**One-line summary:** Generate a deterministic test website (topic pages that link to each other, plus traps: a broken link, a fragment link, an external link, a PDF, a script block containing a decoy word), serve it from a local `http.server` thread in the test fixture, and test every layer: parsing and URL normalisation, crawling (every page once, nothing external), ranking (the obviously most relevant page first), the API (status codes and shape), and the CLI.

```python
# file: minisearch/tests/testsite.py
"""A deterministic website for tests, served from a temporary directory."""
import functools
import http.server
import threading
from pathlib import Path

TOPICS = {
    "binary-search": "Binary search halves a sorted range at each step until the target is found.",
    "binary-tree": "A binary tree is a tree in which each node has at most two children.",
    "binary-search-tree": "A binary search tree keeps keys ordered so that search, insert and delete follow one path.",
    "hash-table": "A hash table maps keys to buckets with a hash function for constant time lookups.",
    "heap": "A heap is a tree where every parent is ordered before its children; heaps implement priority queues.",
    "graph": "A graph has vertices and edges; breadth first search explores it level by level.",
}


def build(root: Path) -> None:
    names = list(TOPICS)
    for i, name in enumerate(names):
        links = "".join(f'<a href="{other}.html#top">{other}</a> ' for other in names if other != name)
        (root / f"{name}.html").write_text(
            f"<html><head><title>{name.replace('-', ' ').title()}</title>"
            f"<script>var decoy = 'zebra';</script></head><body><h1 id=top>{name}</h1>"
            f"<p>{TOPICS[name]}</p>{links}"
            f'<a href="missing.html">broken</a> <a href="https://example.com/x">external</a> '
            f'<a href="paper.pdf">pdf</a> <a href="mailto:a@b.c">mail</a></body></html>')
    (root / "index.html").write_text("<html><head><title>Home</title></head><body>"
                                     + "".join(f'<a href="{n}.html">{n}</a>' for n in names) + "</body></html>")
    (root / "paper.pdf").write_bytes(b"%PDF-1.4 not html")


class _Quiet(http.server.SimpleHTTPRequestHandler):
    def log_message(self, *args: object) -> None:
        pass


def serve(root: Path) -> tuple[str, http.server.ThreadingHTTPServer]:
    handler = functools.partial(_Quiet, directory=str(root))
    server = http.server.ThreadingHTTPServer(("127.0.0.1", 0), handler)
    threading.Thread(target=server.serve_forever, daemon=True).start()
    return f"http://127.0.0.1:{server.server_address[1]}/", server
```

```python
# file: minisearch/tests/test_minisearch.py
import asyncio
import sqlite3
from collections.abc import Iterator
from pathlib import Path

import pytest
from fastapi.testclient import TestClient

from minisearch.cli import main
from minisearch.crawl import crawl
from minisearch.index import add_pages, connect, search
from minisearch.parse import normalise_link, parse_page
from testsite import TOPICS, build, serve


@pytest.fixture(scope="module")
def site(tmp_path_factory: pytest.TempPathFactory) -> Iterator[str]:
    root = tmp_path_factory.mktemp("site")
    build(root)
    url, server = serve(root)
    yield url
    server.shutdown()


@pytest.fixture(scope="module")
def db(site: str, tmp_path_factory: pytest.TempPathFactory) -> str:
    path = str(tmp_path_factory.mktemp("db") / "index.db")
    result = asyncio.run(crawl(site, max_pages=50, concurrency=4))
    add_pages(connect(path), result.pages)
    return path


@pytest.mark.parametrize("base, href, expected", [
    ("http://h/docs/", "intro.html", "http://h/docs/intro.html"),
    ("http://h/docs", "intro.html", "http://h/intro.html"),               # DT-34: no trailing slash
    ("http://h/a.html", "b.html#sec", "http://h/b.html"),                 # fragment dropped
    ("http://h/a.html", "?page=2", "http://h/a.html?page=2"),
    ("http://h/a.html", "https://other.org/", None),                      # external host
    ("http://h/a.html", "mailto:x@y.z", None),
    ("http://h/a.html", "javascript:void(0)", None),
])
def test_normalise_link(base: str, href: str, expected: str | None) -> None:
    assert normalise_link(base, href) == expected


def test_parse_ignores_scripts() -> None:
    page = parse_page("http://h/", "<title> T </title><script>secret()</script><p>Hello <b>world</b></p><a href='x'>x</a>")
    assert page.title == "T" and page.text == "Hello world" and page.links == ("http://h/x",)


def test_crawl_visits_each_page_once(site: str) -> None:
    result = asyncio.run(crawl(site, max_pages=50, concurrency=4))
    urls = sorted(p.url.rsplit("/", 1)[-1] for p in result.pages)
    assert urls == sorted([""] + [f"{n}.html" for n in TOPICS])          # index + topic pages, no duplicates
    assert set(u.rsplit("/", 1)[-1] for u in result.errors) == {"missing.html", "paper.pdf"}


def test_max_pages_is_respected(site: str) -> None:
    assert len(asyncio.run(crawl(site, max_pages=3)).pages) == 3


def test_ranking(db: str) -> None:
    hits = search(connect(db), "binary search tree")
    assert hits[0].title == "Binary Search Tree"
    assert {h.title for h in hits[:3]} == {"Binary Search Tree", "Binary Search", "Binary Tree"}
    assert search(connect(db), "zebra") == []                             # script text is not indexed
    assert "priority queues" in search(connect(db), "priority")[0].snippet


def test_reindex_is_idempotent(db: str, site: str) -> None:
    conn = connect(db)
    before = conn.execute("SELECT COUNT(*) FROM postings").fetchone()[0]
    add_pages(conn, asyncio.run(crawl(site, max_pages=50)).pages)
    assert conn.execute("SELECT COUNT(*) FROM postings").fetchone()[0] == before


def test_api(db: str, monkeypatch: pytest.MonkeyPatch) -> None:
    monkeypatch.setenv("MINISEARCH_DB", db)
    from minisearch.api import app
    client = TestClient(app)
    r = client.get("/search", params={"q": "hash", "limit": 1})
    assert r.status_code == 200 and r.json()[0]["title"] == "Hash Table"
    assert client.get("/search", params={"q": ""}).status_code == 422
    assert client.get("/search", params={"q": "heap", "limit": 99}).status_code == 422


def test_cli(db: str, capsys: pytest.CaptureFixture[str]) -> None:
    assert main(["search", "breadth first", "--db", db]) == 0
    assert "Graph" in capsys.readouterr().out
    assert main(["search", "nonexistentword", "--db", db]) == 1
```

```console
$ rm -rf cap-env && uv venv cap-env -q && uv pip install --python cap-env/bin/python -q -e ./minisearch pytest mypy && echo installed
installed
$ cd minisearch && ../cap-env/bin/python -m mypy src && ../cap-env/bin/python -m mypy --strict tests 2>&1 | tail -1
Success: no issues found in 7 source files
Success: no issues found in 2 source files
$ cd minisearch && ../cap-env/bin/python -m pytest -q -p no:cacheprovider tests 2>&1 | tail -2
-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
14 passed, 1 warning in 3.20s
```

### Pattern

**P-17. Structure an application as small typed modules around plain data (dataclasses), with I/O at the edges (HTTP, database, CLI) and pure logic in the middle (parsing, ranking); test each layer against something independent of its implementation — a generated fixture whose structure you know, RFC-derived cases, obviously relevant results — and gate every change on `mypy --strict` and the test suite.** *(Practice)* The pure middle is easy to test exhaustively; the edges are tested with in-process clients and a few end-to-end runs. *Trade-off:* this structure costs some indirection in a small program, and pays for itself the first time a requirement changes — a new storage backend or a new interface touches one edge, not the whole program.

## Chapter 16 Project — End to end

**Goal:** run the installed tool exactly as a user would: serve the generated website, crawl it with the `minisearch` command, query from the command line, start the HTTP API as a separate process with `minisearch serve`, and query it with `httpx`.

```python
# file: e2e.py
import os
import subprocess
import sys
import tempfile
import time
from pathlib import Path

import httpx

sys.path.insert(0, "minisearch/tests")
from testsite import build, serve                     # noqa: E402

BIN = Path("cap-env/bin")
with tempfile.TemporaryDirectory() as d:
    root = Path(d, "site"); root.mkdir()
    build(root)
    url, server = serve(root)
    db = str(Path(d, "site.db"))
    run = lambda *args: subprocess.run([str(BIN / "minisearch"), *args], capture_output=True, text=True)
    print("> minisearch crawl ...\n" + run("crawl", url, "--db", db, "--concurrency", "4").stdout)
    print("> minisearch search 'binary search tree' -n 2\n" + run("search", "binary search tree", "--db", db, "-n", "2").stdout)
    api = subprocess.Popen([str(BIN / "minisearch"), "serve", "--db", db, "--port", "8902"])
    try:
        for _ in range(100):
            try:
                r = httpx.get("http://127.0.0.1:8902/search", params={"q": "tree children"})
                break
            except httpx.TransportError:
                time.sleep(0.1)
        print("GET /search?q=tree children ->", r.status_code, [(h["title"], h["score"]) for h in r.json()][:3])
        print("GET /search?q= ->", httpx.get("http://127.0.0.1:8902/search", params={"q": ""}).status_code)
    finally:
        api.terminate(); api.wait(10)
        server.shutdown()
```

```console
$ cap-env/bin/python e2e.py
> minisearch crawl ...
crawled 7 pages (2 errors) in 0.61 s
  error: http://127.0.0.1:46203/missing.html -> HTTP 404 text/html;charset=utf-8
  error: http://127.0.0.1:46203/paper.pdf -> HTTP 200 application/pdf

> minisearch search 'binary search tree' -n 2
1. Binary Search Tree  [1.4694]
   http://127.0.0.1:46203/binary-search-tree.html
   binary-search-tree A binary search tree keeps keys ordered so that search, inser…
2. Binary Tree  [0.4537]
   http://127.0.0.1:46203/binary-tree.html
   binary-tree A binary tree is a tree in which each node has at most two children.

GET /search?q=tree children -> 200 [('Binary Tree', 0.634339), ('Heap', 0.215793), ('Binary Search Tree', 0.07872)]
GET /search?q= -> 422
```

The test suite earned its keep while this chapter was written. Its first run reported three real defects: re-indexing doubled the postings, because `INSERT OR REPLACE` deletes and re-inserts a row with a **new** id, orphaning the old postings (fixed with an `ON CONFLICT … DO UPDATE … RETURNING id` upsert); `mypy --strict` on the tests refused to analyse the installed package because the `py.typed` marker was missing; and the ranking test failed because navigation links — every page lists every topic — made the words "binary", "search", and "tree" appear everywhere, so anchor text is now excluded from a page's indexed text.

Every layer was built from the book's chapters, and each is checked by something independent of the code under test: URL rules by RFC-derived cases, the crawler by the known structure of a generated site (every page once; exactly the broken link and the PDF reported as errors; the script's decoy word never indexed), the ranking by obviously relevant pages, and the whole program by `mypy --strict` and an end-to-end run of the installed command.

### Review checklist

- How would you make the crawler polite (respect `robots.txt`, rate-limit per host)?
- Which part of `search` will become slow first as the index grows, and how would you fix it (precomputed document frequencies, a single SQL query with `GROUP BY`)?
- What would you change to index pages that are added or modified over time (store `ETag`/`Last-Modified`, re-crawl incrementally)?
- Where would `async def` endpoints be appropriate here, and what would they require (DT-33)?

## Answers to the Chapter 16 prediction questions

This chapter has no mastery questions; its test suite is the check. If you have read the whole book, the remaining exercise is to extend `minisearch` with one of the review-checklist items — with a test first.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | Bytecode is an implementation detail of CPython: it changes between versions (3.11 introduced specialising "adaptive" instructions, 3.12 and 3.13 changed many opcodes), it is not portable between versions, and other implementations do not use it at all. | Unit 0.2 |
| DT-2 | `is` on numbers and strings is unreliable for value comparison: CPython caches small integers (−5 to 256) and interns some strings, so `a is b` can be `True` for equal small values and `False` for equal large ones — and the result can differ between the REPL, a script, and constant folding. | Unit 0.3 |
| DT-3 | Python's `//` and `%` follow floor semantics (−7 // 2 == −4, −7 % 3 == 2), unlike C, Java, Go, and Rust, which truncate toward zero (−7 / 2 == −3, −7 % 3 == −1); and `int(x)` truncates toward zero, so `int(-3.99) == -3` while `-7 // 2 == -4`. | Unit 1.1 |
| DT-4 | Floats cannot represent most decimal fractions exactly: 0.1 is stored as 0.1000000000000000055511151231257827…, so `0.1 + 0.2 != 0.3`, `round(2.675, 2) == 2.67`, and repeated addition accumulates error; compare with `math.isclose`, sum with `math.fsum` (or `sum`, which uses compensated summation for floats since Python 3.12), and use `Decimal` for money. | Unit 1.6 |
| DT-5 | The directory of the script you run is put first on `sys.path`, so a file of yours named like a standard-library module (`numbers.py`, `random.py`, `email.py`, `queue.py`) shadows it — not only for your imports but for every library module that imports it. | Unit 1.6 |
| DT-6 | `[[0] * 3] * 3` creates  | Unit 2.1 |
| DT-7 | A dict key's hash must never change while it is in the dict, which is why mutable built-ins (list, dict, set) are unhashable; a user-defined class whose `__hash__` depends on mutable fields can be inserted and then become unfindable. | Unit 2.3 |
| DT-8 | `sort()` and `sorted()` are *stable*: items with equal keys keep their original order. That makes multi-key sorting possible by sorting several times, *least important key first* — or, more simply, by sorting once with a tuple key. And `key=` functions are called once per element, while a `cmp`-style comparison would be called O(n log n) times. | Unit 2.5 |
| DT-9 | Closures capture variables, not values: all lambdas created in `[lambda: i for i in range(3)]` share the comprehension's single variable `i`, and when called afterwards they all return its last value, 2. | Unit 3.3 |
| DT-10 | A mutable default argument is created once at function definition and shared by every call that omits the argument — the list (or dict) accumulates state across calls; likewise `def f(t=time.time())` freezes a single timestamp. | Unit 3.4 |
| DT-11 | A mutable class attribute is shared by all instances: `items = []` in the class body is one list, and `self.items.append(x)` mutates it for everyone — while `self.items = [...]` would create an instance attribute that shadows it. | Unit 4.1 |
| DT-12 | Defining `__eq__` without `__hash__` makes a class unhashable: Python sets `__hash__ = None` implicitly, because the default identity hash would break the rule "equal objects must have equal hashes". | Unit 4.3 |
| DT-13 | Iterators are exhausted after one pass: a generator, `map`, `zip`, `filter`, a file object, or `dict.items()`'s iterator yields nothing the second time — silently, with no error — so code that loops twice over a parameter works for lists and breaks for iterators. | Unit 5.1 |
| DT-14 | A generator's `finally` block (and a `with` inside it) runs only when the generator finishes, is closed, or is garbage-collected. A generator abandoned halfway keeps its file open until then — in CPython usually at once (reference counting), but later or never in PyPy or when a reference cycle holds it. | Unit 5.4 |
| DT-15 | `itertools.groupby` groups only  | Unit 5.5 |
| DT-16 | A `return` (or `break`/`continue`) inside `finally` silently discards an exception that is propagating — the error simply disappears; since Python 3.14 this is a `SyntaxWarning` (PEP 765), but on 3.13 it is legal and silent. | Unit 6.1 |
| DT-17 | A bare `except:` (or `except BaseException:`) also catches `KeyboardInterrupt` and `SystemExit`, so Ctrl-C and `sys.exit()` stop working inside it; use `except Exception:` for "anything that is an error", because those two (and `GeneratorExit`) inherit from `BaseException` directly, precisely so that error handlers do not catch them. | Unit 6.5 |
| DT-18 | Running a module inside a package as a script (`python shop/pricing.py`) puts `shop/` itself on `sys.path` and makes the module `__main__`, so its relative imports fail ("attempted relative import with no known parent package") and the module can be imported a second time under its real name, giving two copies of its globals. Run it as `python -m shop.pricing` from the project root. | Unit 7.2 |
| DT-19 | `from module import name` copies the binding at import time; with a circular import it fails if `name` is not yet defined, and even without a cycle, later rebinding `module.name` (for example by monkey-patching in tests) does not affect the copied name — while `import module` and `module.name` always look up the current value. | Unit 7.3 |
| DT-20 | By default mypy does not type-check the *bodies* of functions that have no annotations at all, so an unannotated helper can contain obvious type errors that mypy silently ignores; `--check-untyped-defs` (or `--strict`) closes the gap. | Unit 8.1 |
| DT-21 | `list[int]` is not a subtype of `list[float]` (even though `int` is accepted where `float` is expected): lists are  | Unit 8.4 |
| DT-22 | Two strings that look identical can differ: "é" can be one code point (U+00E9) or two ("e" + combining accent U+0301), so `==`, `len`, dictionary lookups, and set membership treat them as different until both are normalised with `unicodedata.normalize("NFC", s)`. | Unit 9.1 |
| DT-23 | `pickle.loads` on attacker-controlled bytes is remote code execution: the format includes opcodes that import a callable and call it (`__reduce__`), so no "safe" subset exists in the standard `pickle` module; the documentation's warning is literal. | Unit 9.5 |
| DT-24 | `assert (condition, "message")` is always true — a non-empty tuple is truthy — so the test can never fail; and `assert` statements are removed entirely when Python runs with `-O`, so they must never be used for input validation in production code. | Unit 10.3 |
| DT-25 | The GIL makes individual bytecode instructions atomic, not your operations — threads still need locks for compound updates — and since Python 3.13 an optional  | Unit 11.1 |
| DT-26 | A task created with `asyncio.create_task` and not stored anywhere can be garbage-collected before it finishes — the event loop keeps only a weak reference to tasks — and exceptions in fire-and-forget tasks are reported only when the task is destroyed ("Task exception was never retrieved"), often far from their cause. | Unit 11.5 |
| DT-27 | Arithmetic on aware datetimes in the same time zone is  | Unit 12.2 |
| DT-28 | Nested quantifiers such as `(a+)+$` or `(\w+\s?)*$` can take  | Unit 12.3 |
| DT-29 | NumPy integers have fixed width and  | Unit 13.2 |
| DT-30 | Lookup precedence depends on whether a descriptor defines `__set__`: a  | Unit 14.2 |
| DT-31 | `eval(expr, {"__builtins__": {}})` is not a sandbox: attribute access on literals reaches the whole object graph — `().__class__.__base__.__subclasses__()` lists every loaded class, including ones that can open files or run processes — so untrusted input must never be passed to `eval`/`exec`; use `ast.literal_eval` for data, or parse and whitelist AST nodes. | Unit 14.5 |
| DT-32 | Building SQL with f-strings or `%` lets input change the query's structure — SQL injection — and escaping by hand is never complete; parameter binding sends values separately from the SQL text, so they can never be interpreted as SQL. Identifiers (table and column names) cannot be parameters: validate them against a whitelist. | Unit 15.1 |
| DT-33 | In FastAPI, a path operation defined with plain `def` runs in a  | Unit 15.4 |
| DT-34 | `urljoin` follows RFC 3986 exactly, which surprises people: a base URL without a trailing slash is treated as a *file*, so `urljoin("http://h/docs", "intro.html")` is `http://h/intro.html`, not `http://h/docs/intro.html`; and `#section` links, `?query` variants, `mailto:`, and `javascript:` hrefs must be normalised or filtered, or a crawler will fetch the same page under many URLs or try to fetch things that are not pages. | Unit 16.1 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Think of Python variables as labels on objects: ask "does this operation mutate the object, or rebind the name?" and "who else holds a reference to this object?" | Unit 0.4 |
| P-2 | Write conditions and loops the Pythonic way: iterate directly over items (`for x in items`), use `enumerate`/`zip` instead of index arithmetic, rely on truthiness for emptiness checks (`if not items:`), and use `==` for values and `is` only for `None` and other singletons. | Unit 1.6 |
| P-3 | Choose the container by its operations: list for ordered sequences with index access and appends; tuple for fixed records and keys; dict for lookup by key; set for membership and de-duplication; deque for both ends (Chapter 12). | Unit 2.5 |
| P-4 | Put cross-cutting behaviour (timing, logging, retries, caching, access checks) in decorators, always with `functools.wraps`; keep the decorated function focused on its job. | Unit 3.6 |
| P-5 | Model data with dataclasses (frozen when the values should not change), behaviour with methods, and integration with the language through the data model — implement `__repr__` always, `__eq__`/`__hash__` together, and operators only when their meaning is obvious. | Unit 4.5 |
| P-6 | Process streams lazily: write each step as a generator (or generator expression), chain them, and materialise only at the end — or never. Use comprehensions for building collections, `itertools` instead of hand-written index loops. | Unit 5.5 |
| P-7 | Handle an exception only where you can do something meaningful (retry, fall back, add context, translate it for your caller); otherwise let it propagate. Release resources with `with`, never with manual `close()` calls in every branch. | Unit 6.5 |
| P-8 | One virtual environment per project; dependencies declared in `pyproject.toml` and pinned in a lock file; code in a `src/` layout; executables as entry points; and `python -m` (never `python path/to/module.py`) to run modules inside packages. | Unit 7.5 |
| P-9 | Annotate public functions and class attributes; run mypy in CI with strictness increased over time (`--strict` for new code); use abstract parameter types (`Iterable`, `Sequence`, `Mapping`) and `Protocol`s for duck-typed interfaces; validate external data at the boundary with a runtime validator. | Unit 8.5 |
| P-10 | Decode bytes to text at the input boundary, work with `str` inside, encode at the output boundary — always naming the encoding (UTF-8). Use the format's library (csv, json, struct), never hand-written splitting; choose human-readable formats for interchange and binary fixed-size records only when you need speed or random access. | Unit 9.5 |
| P-11 | Test behaviour, not implementation: fixtures for setup, parametrisation for tables of cases, property-based tests for invariants, mocks only at real boundaries (network, clock, randomness). When something is slow, profile first; when something is wrong, reproduce it in a failing test before fixing it. | Unit 10.5 |
| P-12 | I/O-bound with a few blocking calls → threads (`ThreadPoolExecutor`); I/O-bound with thousands of connections → asyncio; CPU-bound → processes (or native code that releases the GIL: NumPy, Chapter 13). Share data by passing messages (futures, queues), not by sharing mutable objects; when you must share, lock. | Unit 11.5 |
| P-13 | Before writing a utility, search the standard library: `collections`, `itertools`, `functools`, `pathlib`, `datetime`/`zoneinfo`, `re`, `json`/`csv`, `subprocess`, `argparse`, `logging`, `statistics`, `secrets`, `concurrent.futures`, `contextlib`, `dataclasses`. Standard modules are documented, tested on every platform, and known to every reader. | Unit 12.5 |
| P-14 | Optimise in this order: measure → better algorithm → better data structure → built-ins and comprehensions → vectorise with NumPy → compile the remaining hot loop (Cython, C, Rust via PyO3, or Numba). Keep the slow version as a reference and check that every faster version produces the same results. | Unit 13.4 |
| P-15 | Reach for the least powerful tool: a function or class first; then a decorator, `property`, or `dataclass`; then `__init_subclass__` or a descriptor for reusable per-class behaviour; a metaclass only when you must control class creation for an entire hierarchy; AST manipulation for tools that analyse code. | Unit 14.5 |
| P-16 | Validate at the edges and keep the core typed: pydantic models for HTTP input and output, parameterised SQL for every query, one transaction per business operation, dependencies for resources (connections, settings, the current user), and in-process tests with dependency or configuration overrides. | Unit 15.4 |
| P-17 | Structure an application as small typed modules around plain data (dataclasses), with I/O at the edges (HTTP, database, CLI) and pure logic in the middle (parsing, ranking); test each layer against something independent of its implementation — a generated fixture whose structure you know, RFC-derived cases, obviously relevant results — and gate every change on `mypy --strict` and the test suite. | Unit 16.5 |

# Appendix B — What Changed in Recent Python Versions

Features used in this book, by the version that introduced them. Code in this book needs Python 3.12 or later (PEP 695 type-parameter syntax in Chapter 8); it was run on 3.13.

| Version | Feature | Where in this book |
| --- | --- | --- |
| 3.6 | f-strings; `secrets` module | Units 1.2, 12.5 |
| 3.7 | dict insertion order guaranteed; `dataclasses`; `breakpoint()` | Units 2.3, 4.5, 10.4 |
| 3.8 | walrus operator `:=`; positional-only parameters `/`; `f"{x=}"`; `functools.cached_property` | Units 9.2, 1.6, 1.2, DT-30 |
| 3.9 | dict merge `|`; built-in generics `list[int]`; `zoneinfo`; `str.removeprefix` | Units 2.3, 8.2, 12.2 |
| 3.10 | structural pattern matching `match`; `X | Y` union types; `zip(strict=True)`; `int.bit_count` | Units 1.5, 8.2, 4.3 |
| 3.11 | `ExceptionGroup` and `except*`; `asyncio.TaskGroup` and `asyncio.timeout`; `add_note`; specialising adaptive interpreter; zero-cost `try`; atomic groups in `re` | Units 6.4, 11.4, 6.2, DT-1, 6.3, DT-28 |
| 3.12 | type-parameter syntax `def f[T]`, `class C[T]`; `itertools.batched`; `sum()` of floats uses compensated summation; f-string grammar relaxed | Units 8.2, 5.5, DT-4 |
| 3.13 | experimental free-threaded build (no GIL, PEP 703); experimental JIT; improved REPL and error messages (e.g. "consider renaming" for shadowed modules); `copy.replace` | DT-25, DT-5 |
| 3.14 | `return`/`break`/`continue` in `finally` warns (PEP 765); deferred evaluation of annotations (PEP 649) | DT-16, Chapter 8 |

## Habits that changed with these versions

- Use `list[int]`, `dict[str, int]`, and `X | None` instead of `typing.List`, `typing.Dict`, and `Optional` (3.9/3.10+).
- Use `TaskGroup` instead of bare `gather` + `create_task` when tasks must be cancelled together (3.11+).
- Prefer `zoneinfo` over third-party time-zone libraries for IANA zones (3.9+).
- Expect `dis` output and micro-benchmark results to change between versions (DT-1); re-measure after upgrading.
