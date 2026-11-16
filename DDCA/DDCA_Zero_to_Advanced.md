---
title: "Digital Design and Computer Architecture from Zero to Advanced"
subtitle: "From transistors and logic gates to a pipelined RISC-V processor in Verilog, branch predictors, caches, virtual memory, DRAM, and multicore coherence — every circuit simulated, every number measured"
lang: en
---

# Preface

A computer is a stack of promises. A program promises the compiler to follow the rules of its language; the compiler promises the processor an instruction stream that follows the rules of its **instruction set architecture** (ISA); the processor promises that, however it reorders and speculates inside, the result will look as if those instructions ran one after another; and the circuits underneath promise that a 1 stays a 1 long enough to be captured by the next clock edge. This book is about how each promise is kept — and about the cases where one is quietly broken, which is where most of the interesting engineering, and most of the security problems of the last decade, come from.

The book follows the topic order of Prof. Onur Mutlu's *Digital Design and Computer Architecture* course at ETH Zürich, whose lectures are freely available on YouTube, so that you can watch a lecture and then work through the matching chapter. Each chapter names the lectures it accompanies. The text, the circuits, the programs, and the exercises are written fresh for this book; the course is the map, not the source.

**Part I (Chapters 0–3)** goes from the transformation hierarchy and how to measure a computer, through Boolean algebra and combinational circuits, sequential circuits and finite state machines, to hardware description in Verilog, timing, and verification. **Part II (Chapters 4–9)** defines an ISA (RISC-V, compared throughout with MIPS and LC-3), writes assembly that runs, and then builds the processor: single-cycle, multi-cycle, pipelined with forwarding and stalls, precise exceptions, out-of-order execution, branch prediction, and the ways of finding more parallelism — superscalar, VLIW, SIMD, and GPUs. **Part III (Chapters 10–14)** is the memory system: caches, virtual memory, DRAM and its reliability and security problems such as RowHammer, storage and I/O, and multiprocessors with cache coherence and memory consistency. **Chapter 15** puts it together: a pipelined RV32I processor in Verilog that runs C programs compiled with GCC, with caches and a branch predictor, measured against the simulators you built along the way.

Three promises shape every page:

1. **Precision.** Each *Deep & Tricky* fact is labelled with its source of truth — the RISC-V ISA manual, the IEEE Verilog standard (IEEE 1364-2005 / 1800-2017), a JEDEC DRAM standard, a published paper, or a vendor's manual.
2. **Evidence.** Every circuit in the book was simulated with Icarus Verilog or Verilator, and every program was run; the waveforms, gate counts, cycle counts, miss rates, and timings shown are real outputs. When a first version of an example behaved differently from what the text claimed, the text was changed to match the hardware — and some of those surprises are kept, because they teach something.
3. **Order.** Nothing is used before it is taught.

# How to use this book

## Who it is for

- **Programmers who want to know what their code turns into:** read in order. The C book of this series is the best preparation; you should be able to read C and use the command line.
- **Students taking a computer architecture course:** read each chapter alongside the lectures it names, and do every *Predict* question before running the code.
- **Engineers preparing for hardware, performance, or systems roles:** the *Deep & Tricky* entries and the chapter projects are the parts interviews draw from.

No electronics background is needed: Chapter 1 starts from what a transistor does as a switch.

## The shape of a unit

| Part | What it gives you |
| --- | --- |
| **One-line summary** | The whole idea in one sentence. |
| **Lesson** | Circuits in Verilog, programs in C and assembly, simulators in Python — complete, with real output. |
| **Deep & Tricky (DT-n)** | Precise facts that surprise experienced engineers, labelled with their source of truth, usually with an experiment that shows the failure. |
| **Patterns (P-n)** | Engineering practices with their trade-offs. |
| **Mastery check** | *Predict* questions; answers at the end of each chapter. |

Each chapter ends with a **Chapter Project**: a realistic piece of work with complete code, real output, and a review checklist.

## Running the code

The source archive has one directory per chapter, with every file exactly as listed (each starts with a one-line `// file:` or `# file:` comment that names it). Console blocks show the exact commands, run from the chapter's directory.

You need a Linux machine (or macOS with Homebrew equivalents) with: `gcc`, `make`, Python 3, **Icarus Verilog** (`iverilog`, `vvp`) for simulation, **Verilator** for fast simulation and linting, **Yosys** for synthesis and gate counts, the **RISC-V GNU toolchain** (`riscv64-linux-gnu-gcc`, which also builds 32-bit RV32I code with `-march=rv32i -mabi=ilp32`), **QEMU user mode** (`qemu-riscv64`) to run RISC-V Linux programs, and **Valgrind** (its Cachegrind tool counts instructions and simulates caches). On Debian or Ubuntu:

```text
apt install build-essential python3 iverilog verilator yosys gcc-riscv64-linux-gnu qemu-user valgrind gtkwave
```

GTKWave is optional; it shows the `.vcd` waveform files the testbenches write. Everything was run on Ubuntu 24.04 with Icarus Verilog 12.0, Verilator 5.020, Yosys 0.33, GCC 13.3 (native and RISC-V), and QEMU 8.2, on a two-core x86-64 virtual machine. Timings and cache sizes are from that machine; yours will differ in the numbers and agree in the shape.

## Reading order

| Part | Chapters | Builds on |
| --- | --- | --- |
| I — Digital design | 0 Why architecture · 1 Combinational logic · 2 Sequential logic · 3 Verilog, timing, verification | C (helpful) |
| II — The processor | 4 ISA and assembly · 5 Single- and multi-cycle · 6 Pipelining · 7 Exceptions and out-of-order · 8 Branch prediction · 9 Superscalar, VLIW, SIMD, GPUs | Part I |
| III — Memory and beyond | 10 Caches · 11 Virtual memory · 12 DRAM · 13 Storage and I/O · 14 Multiprocessors and coherence | Part II |
| Project | 15 A RISC-V CPU in Verilog | Everything |

# Chapter 0 — Why Computer Architecture Matters

*Goal: see the whole machine before taking it apart. A computer is a stack of layers, from the problem you want solved down to the electrons in a transistor, and each layer is defined by an interface to the one above. This chapter walks down the stack with one small C program, measures where its time goes, and shows three "mysteries" — effects you cannot explain without knowing the hardware — that the rest of the book explains.*

*Accompanies the lectures "Introduction and Basics", "Tradeoffs, Metrics, Mindset", "Mysteries in Computer Architecture", and the first lecture on "Memory Systems".*

**What you will be able to do after this chapter**

- Name the levels of the transformation hierarchy and say which interface separates each pair of levels.
- Compile one C function for two ISAs (x86-64 and RISC-V) and read the difference in the assembly.
- Apply the "iron law" of performance — time = instructions × cycles per instruction × time per cycle — and Amdahl's law.
- Measure memory latency with a pointer-chasing benchmark and read the cache levels of your machine from the result.
- Explain, at a high level, how a hardware effect (a cache) can leak secrets between programs.

## Topic 0A — The stack of abstractions

## Unit 0.1 — The transformation hierarchy

**One-line summary:** Solving a problem with a computer means transforming it through a stack of levels — problem, algorithm, program, ISA, microarchitecture, logic, circuits, devices — where each level hides the details of the one below behind an interface; the ISA is the interface between software and hardware.

| Level | What it is | Example |
| --- | --- | --- |
| Problem | What we want | "Sum the numbers in a list" |
| Algorithm | A finite, definite procedure | Add them one by one into an accumulator |
| Program / language | The algorithm in a language | A C `for` loop |
| Runtime / OS | Services the program uses | Memory allocation, files |
| **ISA** | The contract between software and hardware: instructions, registers, memory model | x86-64, RISC-V, ARMv8 |
| Microarchitecture | One implementation of an ISA | A five-stage pipeline; an out-of-order core |
| Logic | Gates and flip-flops | Adders, multiplexers, registers |
| Circuits | Transistors wired into gates | CMOS NAND gate |
| Devices | The physics | Silicon transistors |

The most important line in the table is the ISA. Everything above it is *software* and sees only the ISA: a compiled program does not know whether it runs on a cheap in-order core or a large out-of-order one, as long as both implement the same ISA. Everything below is *hardware*, free to change completely between generations — and it does: Intel and AMD have implemented the x86 ISA with very different microarchitectures for forty years, and a program compiled in 1990 still runs.

Let us follow one function down the stack. Here it is in C:

```c
// file: sum.c
#include <stdio.h>

long sum(const long *a, long n) {
    long s = 0;
    for (long i = 0; i < n; i++)
        s += a[i];
    return s;
}

int main(void) {
    long data[] = {3, 1, 4, 1, 5, 9, 2, 6};
    printf("sum = %ld\n", sum(data, 8));
    return 0;
}
```

The compiler turns `sum` into instructions of the target ISA. For x86-64 and for 64-bit RISC-V, with optimisation on:

```console
$ gcc -O2 -fno-tree-vectorize -c sum.c -o sum_x86.o && objdump -d --no-show-raw-insn sum_x86.o | awk '/<sum>:/,/ret/' | sed -E 's/^ +[0-9a-f]+:\t//'
0000000000000000 <sum>:
endbr64
test   %rsi,%rsi
jle    20 <sum+0x20>
lea    (%rdi,%rsi,8),%rdx
xor    %eax,%eax
nop
add    (%rdi),%rax
add    $0x8,%rdi
cmp    %rdx,%rdi
jne    10 <sum+0x10>
ret
$ riscv64-linux-gnu-gcc -O2 -c sum.c -o sum_rv.o && riscv64-linux-gnu-objdump -d --no-show-raw-insn sum_rv.o | awk '/<sum>:/,/ret/' | sed -E 's/^ +[0-9a-f]+:\t//'
0000000000000000 <sum>:
blez	a1,1a <.L4>
slli	a1,a1,0x3
mv	a5,a0
add	a3,a0,a1
li	a0,0

000000000000000e <.L3>:
ld	a4,0(a5)
addi	a5,a5,8
add	a0,a0,a4
bne	a5,a3,e <.L3>
ret
```

(`-fno-tree-vectorize` stops GCC from using x86's SIMD instructions here, so the two listings do the same thing one element at a time; Chapter 9 turns it back on.)

The two listings are the same algorithm under two contracts. Some differences are visible at once:

- **x86-64 can use memory as an operand.** `add (%rdi),%rax` loads from memory and adds in one instruction. RISC-V is a **load/store** architecture: only `ld` touches memory, and `add` works on registers only, so the same step is two instructions.
- **Instruction sizes differ.** x86 instructions are 1 to 15 bytes long; RISC-V instructions are 4 bytes (or 2 with the compressed extension, which this toolchain enables by default).
- **Different register names and calling conventions**: arguments arrive in `rdi`, `rsi` on x86-64 (System V ABI) and in `a0`, `a1` on RISC-V; the result goes back in `rax` and `a0`.

The program itself runs on either machine. QEMU's user mode emulates a RISC-V CPU running Linux programs, so we can run the RISC-V build on an x86 computer:

```console
$ gcc -O2 sum.c -o sum_x86 && ./sum_x86 && riscv64-linux-gnu-gcc -O2 -static sum.c -o sum_rv && qemu-riscv64 ./sum_rv && file sum_rv | cut -d, -f1-2
sum = 31
sum = 31
sum_rv: ELF 64-bit LSB executable, UCB RISC-V
```

Same source, same output, different machine code: that is what the ISA abstraction buys.

### Deep & Tricky

**DT-1. The ISA specifies *what* each instruction does, not *how fast*: two correct implementations of the same ISA can differ in speed by 10× or more on the same program, and a program's speed can depend on details (alignment, branch patterns, memory layout) that the ISA does not mention at all.** *(RISC-V Unprivileged ISA Specification, Ch. 1: the ISA "is separated into a small base integer ISA … and optional extensions" and specifies architectural state and semantics; timing is outside the specification)* This is why "the same code" can be fast on one laptop and slow on another, and why this book spends Part II on microarchitecture: performance is a property of the implementation. It is also why timing can leak information that the ISA says is invisible (Unit 0.5).

### Mastery check

1. **Explain.** Why can Apple run x86 programs on ARM chips (Rosetta 2), and what layer does such a translator work at?
2. **Predict.** Will `sum_rv` run on an x86-64 Linux machine without QEMU? What error do you expect?
3. **Explain.** In the RISC-V listing, why is there a separate `ld` before the `add`?

## Unit 0.2 — Measuring performance: the iron law

**One-line summary:** Execution time = instructions × (cycles per instruction) × (seconds per cycle); every performance idea in this book changes one of these three factors, usually at the expense of another.

```
         instructions     cycles         seconds
Time  =  ------------  ×  -----------  ×  -------
           program        instruction     cycle
```

- **Instruction count** depends on the algorithm, the compiler, and the ISA (Unit 0.1's RISC-V loop has more instructions than the x86 one).
- **CPI** (cycles per instruction) depends on the microarchitecture and on the program's behaviour: cache misses and mispredicted branches add cycles.
- **Cycle time** (1 / clock frequency) depends on the circuit design and the technology: deeper pipelines allow shorter cycles (Chapter 6).

Hardware performance counters measure the first two directly (`perf stat`), but they are often unavailable inside virtual machines — including the one this book was built on. Valgrind's **Cachegrind** counts instructions exactly by simulating the program, which is enough to see the first factor at work:

```c
// file: iron.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define N 2000000

static double now(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec * 1e-9;
}

static long work_branchy(const int *a, long n) {
    long s = 0;
    for (long i = 0; i < n; i++)
        if (a[i] >= 128) {                   // a data-dependent branch
            s += a[i];
            __asm__ volatile("");            // keeps it a branch (see below)
        }
    return s;
}

int main(int argc, char **argv) {
    int sorted = argc > 1;
    int *a = malloc(N * sizeof *a);
    srand(1);
    for (long i = 0; i < N; i++) a[i] = rand() % 256;
    if (sorted) {                              // counting sort: same values, in order
        long count[256] = {0};
        for (long i = 0; i < N; i++) count[a[i]]++;
        long k = 0;
        for (int v = 0; v < 256; v++) while (count[v]--) a[k++] = v;
    }
    long s = 0;
    double t0 = now();
    for (int rep = 0; rep < 20; rep++) s += work_branchy(a, N);
    printf("%s: s = %ld, %.0f ms\n", sorted ? "sorted" : "random", s, (now() - t0) * 1e3);
    free(a);
    return 0;
}
```

```console
$ gcc -O2 iron.c -o iron && ./iron && ./iron sorted
random: s = 3832063160, 160 ms
sorted: s = 3832063160, 17 ms
$ for mode in "" sorted; do valgrind --tool=cachegrind --cache-sim=no --branch-sim=yes --cachegrind-out-file=/dev/null ./iron $mode 2>&1 | grep -E "I *refs|Mispred rate" | sed -E 's/==[0-9]+== //'; done
I refs:        398,230,218
Mispred rate:         19.7% (      20.1%     +       0.0%   )
I refs:        413,235,466
Mispred rate:          0.1% (        0.1%     +       0.0%   )
```

Both runs compute the same sum over the same numbers, and both execute almost the same number of instructions (the sorted run executes a few million more, for the sorting). Yet the random run takes nearly ten times longer. The instruction count did not change; the **CPI** did. Cachegrind's branch simulator shows why: with random data about one conditional branch in five is mispredicted — almost all of them the `a[i] >= 128` test, which is half of the branches executed and cannot be predicted better than a coin toss — while with sorted data almost none are. Each misprediction throws away the work of tens of instructions that the processor had already started (Chapter 8). The empty `asm volatile` statement inside the `if` is there for the experiment: without it, GCC turns this `if` into branch-free code (a conditional move), there is nothing left to mispredict, and the difference disappears — try it.

### Amdahl's law

If a fraction `p` of the time is sped up by a factor `s`, the overall speedup is

```
speedup = 1 / ((1 − p) + p / s)
```

and it can never exceed `1 / (1 − p)` however large `s` is. Speeding up 90% of a program infinitely gives at most 10×; the remaining 10% becomes the bottleneck. This is why architects measure *before* optimising, and why a 2× faster floating-point unit helps a program that spends 5% of its time in floating point by 2.6% — not by 2×.

```python
# file: amdahl.py
def speedup(p, s):
    return 1 / ((1 - p) + p / s)

for p in (0.5, 0.9, 0.99):
    row = "  ".join(f"s={s:>4}: {speedup(p, s):6.2f}x" for s in (2, 10, 100, 1_000_000))
    print(f"p={p:<4} {row}   limit {1 / (1 - p):.0f}x")
print(f"FPU 2x faster on a program 5% FP: {speedup(0.05, 2):.3f}x")
```

```console
$ python3 amdahl.py
p=0.5  s=   2:   1.33x  s=  10:   1.82x  s= 100:   1.98x  s=1000000:   2.00x   limit 2x
p=0.9  s=   2:   1.82x  s=  10:   5.26x  s= 100:   9.17x  s=1000000:  10.00x   limit 10x
p=0.99 s=   2:   1.98x  s=  10:   9.17x  s= 100:  50.25x  s=1000000:  99.99x   limit 100x
FPU 2x faster on a program 5% FP: 1.026x
```

### Deep & Tricky

**DT-2. Clock frequency alone does not determine performance, and neither does instruction count: a "faster" chip with a higher clock can be slower if its CPI is worse, and a compiler change that *adds* instructions can make a program faster if it removes stalls.** *(Hennessy & Patterson, "Computer Architecture: A Quantitative Approach", §1.8–1.9, the processor performance equation)* The Pentium 4 (2000–2008) raised clock rates to 3.8 GHz with a 31-stage pipeline, and was outperformed by the Pentium M and Core designs at much lower clocks because each misprediction cost so many cycles. Always compare *time* on the workload you care about; the three factors explain the time, they do not replace it.

### Pattern

**P-1. Measure the end-to-end time on a representative workload first; then use instruction counts, CPI, and miss rates to explain it.** *(Practice)* Counters and simulators (Cachegrind here, `perf` on bare metal) tell you *why* something is slow, but only time tells you *whether* it is. *Trade-off:* representative workloads are larger and noisier than micro-benchmarks; repeat runs and report the spread.

### Mastery check

1. **Calculate.** A program runs 10⁹ instructions at CPI 1.5 on a 2 GHz clock. How long does it take? A new design has CPI 1.2 but a 1.8 GHz clock — faster or slower?
2. **Predict.** Delete the `asm volatile` line, compile with `-O2`, and run both modes. What happens to the difference, and why?
3. **Calculate.** 40% of a program is sped up 4×. What is the total speedup?

## Topic 0B — The memory wall

## Unit 0.3 — Latency, measured: pointer chasing

**One-line summary:** A load that hits in the first-level cache takes a few nanoseconds; one that goes to DRAM takes around 100 nanoseconds — enough time for hundreds of instructions — so where data lives matters more than how many instructions touch it.

Processor speed grew much faster than DRAM speed for three decades; the gap is called the **memory wall**. Caches — small, fast memories close to the processor that hold recently used data — hide it when a program's accesses have **locality**: *temporal* (the same data again soon) or *spatial* (nearby data soon).

To measure the latency of each level, a benchmark must defeat everything that hides latency: prefetchers that predict the next address, and out-of-order execution that overlaps independent loads. **Pointer chasing** does both: each load's address comes from the previous load's value, so loads cannot overlap, and the chain visits memory in a random order that no prefetcher can predict.

```c
// file: chase.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

static double now(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec * 1e-9;
}

int main(void) {
    printf("%10s  %8s\n", "size", "ns/load");
    for (size_t bytes = 4 << 10; bytes <= (256u << 20); bytes *= 2) {
        size_t n = bytes / sizeof(void *);
        void **mem = malloc(n * sizeof(void *));
        size_t *perm = malloc(n * sizeof(size_t));
        for (size_t i = 0; i < n; i++) perm[i] = i;
        srand(42);
        for (size_t i = n - 1; i > 0; i--) {               // random cyclic order (Fisher–Yates)
            size_t j = (size_t)rand() % (i + 1);
            size_t t = perm[i]; perm[i] = perm[j]; perm[j] = t;
        }
        for (size_t i = 0; i < n; i++) mem[perm[i]] = &mem[perm[(i + 1) % n]];
        void **p = &mem[perm[0]];
        long steps = 20 * 1000 * 1000;
        double t0 = now();
        for (long i = 0; i < steps; i++) p = (void **)*p;  // each load depends on the last
        double ns = (now() - t0) * 1e9 / steps;
        printf("%8zu KiB  %8.1f%s\n", bytes >> 10, ns, p == NULL ? "!" : "");
        free(mem);
        free(perm);
    }
    return 0;
}
```

```console
$ gcc -O2 chase.c -o chase && ./chase
      size   ns/load
       4 KiB       1.3
       8 KiB       1.3
      16 KiB       1.3
      32 KiB       1.3
      64 KiB       2.2
     128 KiB       3.4
     256 KiB       4.2
     512 KiB       4.8
    1024 KiB       5.8
    2048 KiB      16.5
    4096 KiB      41.0
    8192 KiB      58.5
   16384 KiB     139.3
   32768 KiB     167.2
   65536 KiB     178.8
  131072 KiB     188.9
  262144 KiB     195.4
```

Read the table from the top. While the working set fits in the 48 KiB L1 data cache, each load takes about a nanosecond — a handful of cycles at 2.3 GHz. Past L1, the time steps up to the L2 level; past the 2 MiB L2, it rises again; and for working sets of hundreds of megabytes, every load is a trip to DRAM — more than a hundred times slower than L1. Part of that last figure is not DRAM itself but address translation: with 4 KiB pages, a random access to a 256 MiB array also misses in the TLB and walks the page table (Chapter 11); on bare metal with huge pages, DRAM latency is typically 80–100 ns. (On this virtual machine the hypervisor reports a 504 MiB "L3", which is not a real cache size, so the last steps are less sharp than on bare metal.) The `p == NULL` test in the `printf` exists only so the compiler cannot delete the loop as useless: the final pointer is "used".

### Deep & Tricky

**DT-3. A benchmark that walks memory sequentially, or in any regular pattern, measures the prefetcher, not the memory: hardware prefetchers detect constant strides and fetch ahead, so sequential latency looks like L1 latency even for gigabyte arrays. Random dependent loads are the only way to see true latency.** *(Intel 64 and IA-32 Architectures Optimization Reference Manual, "Hardware Prefetching" — the L1 and L2 streamers and the IP-based stride prefetcher)* Change the shuffle in `chase.c` to the identity permutation (each element points to the next) and the "DRAM" rows drop to a few nanoseconds — not because DRAM got faster, but because the prefetcher had each line in cache before the load asked for it. Chapter 10 builds caches and prefetchers in simulation and shows the same effect from the inside.

### Mastery check

1. **Explain.** Why must each load's address depend on the previous load's value?
2. **Predict.** If the array is visited sequentially instead of randomly, what happens to the 256 MiB row, and why?
3. **Calculate.** At 2.3 GHz, how many cycles is a 90 ns DRAM access? How many simple instructions could a core that completes 3 per cycle have run in that time?

## Unit 0.4 — Bandwidth versus latency

**One-line summary:** Latency is how long one access takes; bandwidth is how many bytes per second flow when many accesses are in flight at once — and a program that issues independent loads gets bandwidth, while a program whose loads depend on each other gets latency.

Pointer chasing measured latency by making every load wait for the previous one. Streaming over an array does the opposite: the loads are independent, the prefetcher runs ahead, and the out-of-order core keeps many misses in flight. The memory system then delivers data at its *bandwidth*.

```c
// file: stream.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

static double now(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec * 1e-9;
}

int main(void) {
    size_t n = (256u << 20) / sizeof(long);               // 256 MiB: far beyond the caches
    long *a = malloc(n * sizeof(long));
    for (size_t i = 0; i < n; i++) a[i] = (long)i;
    long s = 0;
    double best = 1e9;
    for (int rep = 0; rep < 5; rep++) {
        double t0 = now();
        for (size_t i = 0; i < n; i++) s += a[i];
        double t = now() - t0;
        if (t < best) best = t;
    }
    double gbs = n * sizeof(long) / best / 1e9;
    printf("sequential read: %.1f GB/s (checksum %ld)\n", gbs, s & 1);
    printf("time per 8-byte element: %.2f ns\n", best * 1e9 / n);
    free(a);
    return 0;
}
```

```console
$ gcc -O2 stream.c -o stream && ./stream
sequential read: 10.0 GB/s (checksum 0)
time per 8-byte element: 0.80 ns
```

Each 8-byte element took a fraction of a nanosecond — two orders of magnitude less than one random DRAM access in Unit 0.3 — although both programs read the same 256 MiB from DRAM. A cache line (64 bytes on this machine) holds 8 elements, so one miss serves 8 loads; the prefetcher fetches lines before they are needed; and many fetches overlap. **Little's law** connects the three quantities: *bytes in flight = bandwidth × latency*. To sustain 9 GB/s with 100 ns latency, about 900 bytes — 14 cache lines — must be in flight at all times, which is why cores track many outstanding misses at once (Chapter 10).

### Mastery check

1. **Calculate.** With 100 ns latency and 64-byte lines, what bandwidth does a program get if only one miss can be outstanding at a time?
2. **Explain.** Why does a linked list traversal get latency-bound performance while an array sum gets bandwidth-bound performance, even if both visit every element once?

## Topic 0C — Mysteries

## Unit 0.5 — A cache can leak a secret

**One-line summary:** Whether an address is in the cache changes how long a load takes; an attacker who can measure that time can learn which addresses another piece of code touched — the basis of cache side-channel attacks, Spectre among them.

The ISA promises that a program's result does not depend on the cache. It promises nothing about *time* (DT-1). The demonstration below uses two x86 instructions: `clflush` evicts a cache line from every level, and `rdtscp` reads a cycle counter. A "victim" function touches one of 16 cache lines depending on a secret value; the "attacker" then times a load from each line. The one that loads quickly is the one the victim touched. This is **Flush+Reload**.

```c
// file: flushreload.c
#include <stdint.h>
#include <stdio.h>
#include <x86intrin.h>

#define LINES 16
#define STRIDE 4096                       // one line per page: defeats the adjacent-line prefetcher

static uint8_t probe[LINES * STRIDE];

static void victim(int secret) {
    volatile uint8_t x = probe[secret * STRIDE];   // the only trace of the secret: one cache line
    (void)x;
}

static uint64_t time_load(volatile uint8_t *p) {
    unsigned aux;
    _mm_mfence();
    uint64_t t0 = __rdtscp(&aux);
    (void)*p;
    uint64_t t1 = __rdtscp(&aux);
    return t1 - t0;
}

static int recover(int secret) {
    int votes[LINES] = {0};
    for (int trial = 0; trial < 200; trial++) {
        for (int i = 0; i < LINES; i++) _mm_clflush(&probe[i * STRIDE]);   // flush
        _mm_mfence();
        victim(secret);                                                   // victim runs
        int best = 0;
        uint64_t best_t = UINT64_MAX;
        for (int k = 0; k < LINES; k++) {                                 // reload, in a scrambled order
            int i = (k * 7 + trial) % LINES;
            uint64_t t = time_load(&probe[i * STRIDE]);
            if (t < best_t) { best_t = t; best = i; }
        }
        votes[best]++;
    }
    int guess = 0;
    for (int i = 1; i < LINES; i++) if (votes[i] > votes[guess]) guess = i;
    return guess;
}

int main(void) {
    for (int i = 0; i < LINES * STRIDE; i++) probe[i] = 1;          // make the pages real
    int secrets[] = {3, 11, 7, 0, 15};
    for (int k = 0; k < 5; k++)
        printf("victim's secret %2d -> attacker recovered %2d\n", secrets[k], recover(secrets[k]));
    return 0;
}
```

```console
$ gcc -O2 flushreload.c -o flushreload && ./flushreload
victim's secret  3 -> attacker recovered  3
victim's secret 11 -> attacker recovered 11
victim's secret  7 -> attacker recovered  7
victim's secret  0 -> attacker recovered  0
victim's secret 15 -> attacker recovered 15
```

The attacker never reads the secret; it reads *time*. In this toy, attacker and victim are one program for clarity; real attacks run them as separate processes sharing memory (a shared library) or as JavaScript in a browser timing its own array. **Spectre** (2018) added the decisive step: it makes the processor *speculatively* execute victim code that it will later throw away — for example a bounds check that is predicted to pass when it should fail — and the discarded instructions still leave cache footprints. Chapter 7 explains speculation; Chapter 8 explains the predictor that is trained to mispredict.

### Deep & Tricky

**DT-4. Instructions that are executed speculatively and then squashed leave no trace in architectural state (registers, memory) — exactly as the ISA promises — but they do change microarchitectural state such as cache contents, which persists after the squash and can be measured. Spectre and Meltdown exploit this gap between the architectural and the microarchitectural.** *(Kocher et al., "Spectre Attacks: Exploiting Speculative Execution", IEEE S&P 2019; Lipp et al., "Meltdown: Reading Kernel Memory from User Space", USENIX Security 2018)* Mitigations work at every layer of the hierarchy: compilers insert speculation barriers (`lfence`) or masks after bounds checks, operating systems separate kernel and user page tables (KPTI), and newer processors change how speculation interacts with caches — each with a performance cost.

### Mastery check

1. **Explain.** Why does the probe array use a stride of 4096 bytes instead of 64?
2. **Predict.** What does the attacker recover if the `clflush` loop is removed, and why?
3. **Explain.** Which layer of the transformation hierarchy "promises" that the cache is invisible, and which layer breaks that promise?

## Unit 0.6 — Other mysteries the book will explain

**One-line summary:** Many effects that look like software bugs or magic — bit flips from reading memory, programs slowing each other down on a multicore chip, a sorted array summing faster — have hardware explanations, and the rest of this book gives them.

| Mystery | Where it is explained |
| --- | --- |
| Reading a DRAM row many times can flip bits in *neighbouring* rows (RowHammer, 2014) | Chapter 12 |
| Two programs on different cores slow each other down although they share no data | Chapters 10, 12, 14 (shared caches, memory bandwidth, false sharing) |
| Sorting data before processing makes a loop several times faster | Unit 0.2, Chapter 8 (branch prediction) |
| A one-line change in array layout makes code 10× faster | Chapter 10 (spatial locality) |
| Two threads both see the "old" value of a variable the other just wrote | Chapter 14 (memory consistency) |
| A design passes simulation and fails on the real chip | Chapter 3 (timing, metastability) |

RowHammer deserves a sentence now, because it changed how the field thinks about reliability. DRAM stores bits as charge in tiny capacitors packed so densely that repeatedly activating one row disturbs the charge in adjacent rows. Kim et al. showed in 2014 that most DRAM modules then on sale could be made to flip bits this way, purely from software, and later work turned it into privilege-escalation attacks. A hardware reliability problem became a security problem — an example of why architects cannot ignore the layers below them.

### Mastery check

1. **Explain.** Pick one row of the table and write, in two sentences, which layer of the transformation hierarchy you think is responsible.

## Chapter 0 Project — `machprobe`: measure your machine

### Requirements

Write a small tool that reports, for the machine it runs on: the latency of each level of the memory hierarchy (from a pointer-chasing sweep), the sizes where latency jumps (an estimate of the cache sizes), the sequential read bandwidth, and the cost of a mispredicted branch (from the difference between random and sorted data). Print a short report. The numbers must come from measurement, not from `/proc` or `lscpu`; those are printed alongside for comparison.

### Reference solution

```c
// file: machprobe/machprobe.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

static double now(void) {
    struct timespec t;
    clock_gettime(CLOCK_MONOTONIC, &t);
    return t.tv_sec + t.tv_nsec * 1e-9;
}

static double chase_ns(size_t bytes) {
    size_t n = bytes / sizeof(void *);
    void **mem = malloc(n * sizeof(void *));
    size_t *perm = malloc(n * sizeof(size_t));
    for (size_t i = 0; i < n; i++) perm[i] = i;
    srand(7);
    for (size_t i = n - 1; i > 0; i--) {
        size_t j = (size_t)rand() % (i + 1);
        size_t t = perm[i]; perm[i] = perm[j]; perm[j] = t;
    }
    for (size_t i = 0; i < n; i++) mem[perm[i]] = &mem[perm[(i + 1) % n]];
    void **p = &mem[perm[0]];
    long steps = 5 * 1000 * 1000;
    double best = 1e9;
    for (int rep = 0; rep < 3; rep++) {
        double t0 = now();
        for (long i = 0; i < steps; i++) p = (void **)*p;
        double ns = (now() - t0) * 1e9 / steps;
        if (ns < best) best = ns;
    }
    if (p == NULL) puts("");
    free(mem);
    free(perm);
    return best;
}

static double branch_ns(int sorted) {
    enum { N = 1 << 20 };
    static int a[N];
    srand(3);
    for (int i = 0; i < N; i++) a[i] = rand() % 256;
    if (sorted) {
        int count[256] = {0}, k = 0;
        for (int i = 0; i < N; i++) count[a[i]]++;
        for (int v = 0; v < 256; v++) while (count[v]--) a[k++] = v;
    }
    volatile long sink = 0;
    double best = 1e9;
    for (int rep = 0; rep < 5; rep++) {
        double t0 = now();
        long s = 0;
        for (int i = 0; i < N; i++)
            if (a[i] >= 128) {
                s += a[i];
                __asm__ volatile("");              // keep the branch: no conditional-move rewrite
            }
        sink = s;
        double t = (now() - t0) * 1e9 / N;
        if (t < best) best = t;
    }
    (void)sink;
    return best;
}

static double stream_gbs(void) {
    size_t n = (128u << 20) / sizeof(long);
    long *a = malloc(n * sizeof(long));
    for (size_t i = 0; i < n; i++) a[i] = (long)i;
    double best = 1e9;
    volatile long sink;
    for (int rep = 0; rep < 3; rep++) {
        double t0 = now();
        long s = 0;
        for (size_t i = 0; i < n; i++) s += a[i];
        sink = s;
        double t = now() - t0;
        if (t < best) best = t;
    }
    (void)sink;
    free(a);
    return n * sizeof(long) / best / 1e9;
}

int main(void) {
    printf("== memory latency (pointer chasing) ==\n");
    double prev = 0;
    for (size_t kib = 8; kib <= 128 * 1024; kib *= 2) {
        double ns = chase_ns(kib << 10);
        const char *mark = (prev > 0 && ns > 1.6 * prev) ? "  <- jump" : "";
        printf("%8zu KiB %7.1f ns%s\n", kib, ns, mark);
        prev = ns;
    }
    printf("== bandwidth ==\nsequential read %.1f GB/s\n", stream_gbs());
    double r = branch_ns(0), s = branch_ns(1);
    printf("== branches ==\nrandom %.2f ns/elem, sorted %.2f ns/elem\n", r, s);
    printf("estimated misprediction penalty: %.1f ns (half of the random branches mispredict)\n", (r - s) * 2);
    return 0;
}
```

```console
$ cd machprobe && gcc -O2 machprobe.c -o machprobe && ./machprobe
== memory latency (pointer chasing) ==
       8 KiB     1.3 ns
      16 KiB     1.3 ns
      32 KiB     1.3 ns
      64 KiB     2.1 ns  <- jump
     128 KiB     3.2 ns
     256 KiB     4.1 ns
     512 KiB     4.6 ns
    1024 KiB     5.1 ns
    2048 KiB    14.3 ns  <- jump
    4096 KiB    38.1 ns  <- jump
    8192 KiB    53.6 ns
   16384 KiB    99.1 ns  <- jump
   32768 KiB   155.8 ns
   65536 KiB   171.7 ns
  131072 KiB   178.1 ns
== bandwidth ==
sequential read 10.2 GB/s
== branches ==
random 3.80 ns/elem, sorted 0.33 ns/elem
estimated misprediction penalty: 6.9 ns (half of the random branches mispredict)
$ lscpu | grep -E "^(L1d|L2|L3) cache"
L1d cache:                               96 KiB (2 instances)
L2 cache:                                4 MiB (2 instances)
L3 cache:                                504 MiB (1 instance)
```

Design notes: the latency sweep marks a "jump" where latency grows by more than 60% from the previous size — a crude but robust way to find cache boundaries. The branch test multiplies the per-element difference by two because, with random data, about half the branches mispredict; dividing by the clock period turns the result into cycles. The empty `asm volatile` statement inside the `if` stops the compiler from turning the branch into a conditional move, which would hide the effect (Unit 0.2): a compiler may not execute a `volatile` asm speculatively, so the branch must stay.

### Review checklist

- Do the latency jumps line up with the cache sizes `lscpu` reports? Where they do not, which is more trustworthy?
- What would change if the machine had transparent huge pages disabled? (Hint: TLB misses — Chapter 11 — add to the latency of large working sets.)
- Is the misprediction estimate consistent with a pipeline depth of 15–20 stages at 2.3 GHz?

## Answers to the Chapter 0 prediction questions

**Unit 0.1, Q2.** No. The kernel refuses to execute it: the ELF header says the machine is RISC-V, so `execve` fails with "Exec format error" (unless `binfmt_misc` is configured to run it through QEMU automatically, as some distributions do when `qemu-user-binfmt` is installed).

**Unit 0.2, Q1.** 10⁹ × 1.5 / (2·10⁹) = 0.75 s. New design: 10⁹ × 1.2 / (1.8·10⁹) = 0.667 s — faster despite the slower clock.

**Unit 0.2, Q2.** Without the `asm` statement, GCC replaces the branch with a conditional move (or a masked add), so there is nothing to mispredict: both modes run at about the same speed.

**Unit 0.2, Q3.** 1 / (0.6 + 0.4 / 4) = 1 / 0.7 ≈ 1.43×.

**Unit 0.3, Q2.** It drops to a few nanoseconds per load: the prefetcher detects the sequential stride and fetches lines before they are needed.

**Unit 0.3, Q3.** 90 ns × 2.3 cycles/ns ≈ 207 cycles; at 3 instructions per cycle, about 620 instructions.

**Unit 0.4, Q1.** 64 bytes per 100 ns = 0.64 GB/s.

**Unit 0.5, Q2.** Every line is in the cache already (from earlier reloads), so all loads are fast and the attacker's choice is noise: the recovered values become essentially random.

# Chapter 1 — Combinational Logic

*Goal: build every circuit a processor needs that has no memory — gates, multiplexers, decoders, adders, comparators, an ALU — starting from transistors used as switches. A **combinational** circuit's outputs depend only on its current inputs; there is no state and no clock. Along the way you learn to describe circuits in Verilog, test them exhaustively against a reference model, and let a synthesis tool count the gates and measure the longest path.*

*Accompanies the lectures "Combinational Logic I" and "Combinational Logic II" (and the first part of "Hardware Description Languages and Verilog", which Chapter 3 completes).*

**What you will be able to do after this chapter**

- Explain how CMOS transistors make NOT, NAND, and NOR gates, and why NAND is cheaper than AND.
- Move between truth tables, Boolean expressions (sum-of-products), and circuits, and simplify with Boolean algebra.
- Write combinational Verilog with `assign` and gate instances, and test it exhaustively with a self-checking testbench in Icarus Verilog.
- Build decoders, multiplexers, and adders, and compare ripple-carry with carry-lookahead using Yosys's gate counts and path lengths.
- Represent signed numbers in two's complement, detect overflow, and build an ALU.

**How this chapter's code is organised.** Each circuit is one `.v` file with its testbench (`module tb`) at the bottom. `iverilog -g2012 -o sim file.v && vvp -n sim` compiles and runs it; `yosys -p "read_verilog file.v; synth -top NAME; stat"` synthesises it to generic gates and counts them. Yosys defines the macro `SYNTHESIS`, so each testbench is wrapped in `` `ifndef SYNTHESIS `` … `` `endif `` and the synthesis tool skips it (it cannot build hardware for `$display`).

## Topic 1A — From switches to gates

## Unit 1.1 — Transistors as switches: CMOS gates

**One-line summary:** A MOS transistor is a voltage-controlled switch — an nMOS conducts when its gate is 1, a pMOS when its gate is 0 — and a CMOS gate pairs a pull-up network of pMOS transistors with a complementary pull-down network of nMOS transistors so that the output is always connected to exactly one of the supply (1) or ground (0).

A digital circuit represents 1 and 0 as two voltage ranges (for example, near 1 V and near 0 V). The building block is the **metal-oxide-semiconductor field-effect transistor** (MOSFET), which for digital purposes is a switch between two terminals (source and drain) controlled by a third (the gate):

| Transistor | Conducts when gate is | Good at passing |
| --- | --- | --- |
| nMOS | 1 | 0 (connects to ground) |
| pMOS | 0 | 1 (connects to the supply) |

**CMOS** (complementary MOS) builds each gate from two networks. The **pull-down network** of nMOS transistors connects the output to ground when the output should be 0; the **pull-up network** of pMOS transistors connects it to the supply when the output should be 1. The two are complements, so for every input exactly one network conducts — no current flows from supply to ground except briefly while switching, which is why CMOS uses so little power when idle.

- **NOT** (inverter): one pMOS from supply to output, one nMOS from output to ground, both gated by the input.
- **NAND**: two pMOS in *parallel* (either input 0 pulls the output up), two nMOS in *series* (both inputs 1 pull it down).
- **NOR**: two pMOS in *series*, two nMOS in *parallel*.

Verilog can describe circuits at this switch level with the `nmos` and `pmos` primitives (`nmos name(drain, source, gate)`), and Icarus Verilog simulates them:

```verilog
// file: cmos.v
module not_cmos(output y, input a);
    supply1 vdd;
    supply0 gnd;
    pmos p(y, vdd, a);           // a = 0: y connected to vdd
    nmos n(y, gnd, a);           // a = 1: y connected to gnd
endmodule

module nand_cmos(output y, input a, input b);
    supply1 vdd;
    supply0 gnd;
    wire mid;
    pmos pa(y, vdd, a);          // pull-up: parallel
    pmos pb(y, vdd, b);
    nmos na(y, mid, a);          // pull-down: series
    nmos nb(mid, gnd, b);
endmodule

module nor_cmos(output y, input a, input b);
    supply1 vdd;
    supply0 gnd;
    wire mid;
    pmos pa(mid, vdd, a);        // pull-up: series
    pmos pb(y, mid, b);
    nmos na(y, gnd, a);          // pull-down: parallel
    nmos nb(y, gnd, b);
endmodule

module and_cmos(output y, input a, input b);   // AND = NAND followed by NOT
    wire n;
    nand_cmos g1(n, a, b);
    not_cmos  g2(y, n);
endmodule

module tb;
    reg a, b;
    wire y_not, y_nand, y_nor, y_and;
    not_cmos  u0(y_not, a);
    nand_cmos u1(y_nand, a, b);
    nor_cmos  u2(y_nor, a, b);
    and_cmos  u3(y_and, a, b);
    integer i;
    initial begin
        $display("a b | NOT(a) NAND NOR AND");
        for (i = 0; i < 4; i = i + 1) begin
            {a, b} = i[1:0];
            #1 $display("%b %b |   %b     %b    %b   %b", a, b, y_not, y_nand, y_nor, y_and);
        end
    end
endmodule
```

```console
$ iverilog -g2012 -o cmos cmos.v && vvp -n cmos
a b | NOT(a) NAND NOR AND
0 0 |   1     1    1   0
0 1 |   1     1    0   0
1 0 |   0     1    0   0
1 1 |   0     0    0   1
```

Line by line: `supply1` and `supply0` are nets permanently at 1 and 0; each `pmos`/`nmos` instance lists drain, source, and gate; the testbench assigns the 2-bit loop counter to `{a, b}` (the braces concatenate signals into one vector) and waits one time unit (`#1`) so the switches settle before printing.

Counting transistors: NOT uses 2, NAND and NOR use 4 each, and AND — which needs a NAND *and* a NOT — uses 6. In CMOS, the inverting gates are the natural ones. That is why logic is often designed with NANDs and NORs, and why a synthesis tool's gate library is full of inverting cells.

### Deep & Tricky

**DT-5. A CMOS gate's output is *inverting* by construction: a single stage of nMOS pull-down and pMOS pull-up can only produce functions that go from 1 to 0 when inputs rise (NOT, NAND, NOR, AND-OR-INVERT…). Non-inverting functions such as AND, OR, and the plain buffer need two stages.** *(Weste & Harris, "CMOS VLSI Design", §1.4: "a single CMOS gate is inherently inverting")* The reason is in the transistors: nMOS devices pass a strong 0 but a weak 1, and pMOS devices the reverse, so the pull-down network must sit next to ground and the pull-up next to the supply. One consequence you will meet in the gate counts below: Yosys's generic library offers `$_ANDNOT_` and `$_ORNOT_` cells, and real standard-cell libraries are built around NAND, NOR, and AOI cells because those are single stages.

### Mastery check

1. **Explain.** Why must the pull-up network of a NOR gate have its pMOS transistors in series?
2. **Predict.** In `nand_cmos`, what happens to `y` if the pull-up were removed (only the nMOS pair left), when `a = 0`? (Icarus prints a special value — which one?)
3. **Design.** How many transistors does a 3-input NAND need? Draw (on paper) its pull-up and pull-down networks.

## Unit 1.2 — Boolean algebra and the sum-of-products form

**One-line summary:** Any combinational function can be written from its truth table as a **sum of products** — an OR of one AND term (a *minterm*) per row where the output is 1 — and the axioms of Boolean algebra (distributivity, De Morgan's laws, absorption, the consensus theorem) simplify such expressions into smaller circuits.

Boolean algebra works on values 0 and 1 with three operations: AND (written `·` or juxtaposition, `AB`), OR (`+`), and NOT (an overbar or prime, `A'`). The laws that matter most in practice:

| Law | AND form | OR form |
| --- | --- | --- |
| Identity | `A·1 = A` | `A + 0 = A` |
| Null | `A·0 = 0` | `A + 1 = 1` |
| Complement | `A·A' = 0` | `A + A' = 1` |
| Distributive | `A·(B + C) = AB + AC` | `A + BC = (A + B)(A + C)` |
| Absorption | `A·(A + B) = A` | `A + AB = A` |
| De Morgan | `(AB)' = A' + B'` | `(A + B)' = A'B'` |
| Combining | `AB + AB' = A` | `(A + B)(A + B') = A` |

The second distributive law, `A + BC = (A + B)(A + C)`, has no counterpart in ordinary arithmetic — a classic source of mistakes.

**From a truth table to a circuit.** Take the **majority** function of three inputs: the output is 1 when at least two inputs are 1. Its truth table has 1s in rows 011, 101, 110, and 111, so its sum-of-products form is

```
Y = A'BC + AB'C + ABC' + ABC
```

Using `ABC = ABC + ABC + ABC` (idempotence) and the combining law three times gives `Y = AB + BC + AC`: three 2-input ANDs and one 3-input OR instead of four 3-input ANDs and a 4-input OR.

Two descriptions of one function must agree on every input. For a few inputs, the testbench can simply try them all — an **exhaustive** test — which proves equivalence:

```verilog
// file: majority.v
module maj_sop(output y, input a, input b, input c);            // straight from the truth table
    assign y = (~a & b & c) | (a & ~b & c) | (a & b & ~c) | (a & b & c);
endmodule

module maj_min(output y, input a, input b, input c);            // simplified
    assign y = (a & b) | (b & c) | (a & c);
endmodule

module maj_gates(output y, input a, input b, input c);          // the same, as gate instances
    wire ab, bc, ac;
    and g1(ab, a, b);
    and g2(bc, b, c);
    and g3(ac, a, c);
    or  g4(y, ab, bc, ac);
endmodule

`ifndef SYNTHESIS                                // Yosys defines SYNTHESIS: it skips the testbench
module tb;
    reg [2:0] in;
    wire y1, y2, y3;
    maj_sop   u1(y1, in[2], in[1], in[0]);
    maj_min   u2(y2, in[2], in[1], in[0]);
    maj_gates u3(y3, in[2], in[1], in[0]);
    integer i, errors = 0;
    initial begin
        for (i = 0; i < 8; i = i + 1) begin
            in = i;
            #1;
            if (y1 !== y2 || y2 !== y3) begin
                $display("MISMATCH at %b: %b %b %b", in, y1, y2, y3);
                errors = errors + 1;
            end
            $display("%b -> %b", in, y2);
        end
        $display("%0d mismatches over all 8 inputs", errors);
    end
endmodule
`endif
```

```console
$ iverilog -g2012 -o majority majority.v && vvp -n majority
000 -> 0
001 -> 0
010 -> 0
011 -> 1
100 -> 0
101 -> 1
110 -> 1
111 -> 1
0 mismatches over all 8 inputs
$ for top in maj_sop maj_min; do echo -n "$top: "; yosys -p "read_verilog majority.v; synth -top $top; abc -g AND,OR; stat" | grep -E "Number of cells" | tail -1 | tr -s ' '; done
maj_sop:  Number of cells: 4
maj_min:  Number of cells: 4
```

The `assign` statement describes a continuous connection: whenever an input changes, the expression is re-evaluated, like a wire driven by gates. `~`, `&`, `|`, and `^` are bitwise NOT, AND, OR, and XOR. The comparison uses `!==` (case inequality), which treats the unknown value `x` as a value of its own; DT-6 explains why that matters.

Yosys, asked to map both versions onto AND and OR gates only (`abc -g AND,OR`), finds the same small circuit for both: synthesis tools minimise automatically, so in practice you write the clearest description and let the tool simplify. Knowing the algebra still matters — to read what the tool produced, to check it, and to reason about circuits too large to simplify by hand.

### Deep & Tricky

**DT-6. Verilog signals have four values — 0, 1, `x` (unknown), and `z` (high impedance, not driven) — and the ordinary equality operators propagate unknowns: `x == 1` is `x`, and an `if` whose condition is `x` takes the `else` branch. A testbench that checks `if (y != expected)` therefore silently *passes* when `y` is `x`. Use the case-equality operators `===` and `!==`, which compare all four values literally.** *(IEEE 1364-2005 §5.1.8 "Equality operators": "For the logical equality and logical inequality operators… if… the relation is ambiguous, then the result shall be a one-bit unknown value (x)"; §9.4: an `if` with an unknown condition executes the `else` statement)* The experiment below leaves one input of a gate unconnected and checks it both ways.

```verilog
// file: xcheck.v
module half_built(output y, input a, input b);
    wire unconnected;                    // forgot to connect: stays z
    assign y = a & unconnected;
endmodule

module tb;
    reg a = 1, b = 1;
    wire y;
    half_built u(y, a, b);
    initial begin
        #1;
        $display("y = %b", y);
        if (y != 1'b1) $display("== check: caught the bug");
        else           $display("== check: PASSED (wrongly)");
        if (y !== 1'b1) $display("!== check: caught the bug");
    end
endmodule
```

```console
$ iverilog -g2012 -o xcheck xcheck.v && vvp -n xcheck
y = x
== check: PASSED (wrongly)
!== check: caught the bug
```

The `!=` comparison of `x` with 1 is itself `x`, which is not "true", so the error branch did not run.

### Mastery check

1. **Simplify.** `Y = A'B'C + A'BC + AB'C` — write it in the smallest sum-of-products form.
2. **Prove.** Use De Morgan's laws to show that a NAND gate with both inputs tied together is a NOT gate.
3. **Predict.** What does `x & 0` evaluate to in Verilog? And `x | 1`?

## Topic 1B — Building blocks

## Unit 1.3 — Decoders, multiplexers, and look-up tables

**One-line summary:** A **decoder** turns an n-bit number into 2ⁿ one-hot lines (exactly one is 1); a **multiplexer** selects one of 2ⁿ inputs by an n-bit select signal; together they can implement any function, and an FPGA is built from small look-up tables that are multiplexers whose data inputs are configuration bits.

A 2:1 multiplexer computes `y = s ? d1 : d0`, which as gates is `(s' · d0) + (s · d1)`. Wider multiplexers are trees of 2:1 muxes, or AND-OR structures driven by a decoder. In Verilog, the conditional operator and array indexing describe them directly:

```verilog
// file: blocks.v
module dec2to4(output [3:0] y, input [1:0] a);
    assign y = 4'b0001 << a;                         // one-hot
endmodule

module mux4(output y, input [3:0] d, input [1:0] s);
    assign y = d[s];                                 // indexing a vector with a signal = a mux
endmodule

module mux4_tree(output y, input [3:0] d, input [1:0] s);
    wire lo = s[0] ? d[1] : d[0];
    wire hi = s[0] ? d[3] : d[2];
    assign y = s[1] ? hi : lo;
endmodule

module lut3(output y, input [2:0] x, input [7:0] config_bits);   // any 3-input function
    assign y = config_bits[x];
endmodule

`ifndef SYNTHESIS                                // Yosys defines SYNTHESIS: it skips the testbench
module tb;
    reg [1:0] a;
    wire [3:0] onehot;
    reg [3:0] d;
    wire y1, y2, maj, xor3;
    reg [2:0] x;
    dec2to4   dec(onehot, a);
    mux4      m1(y1, d, a);
    mux4_tree m2(y2, d, a);
    lut3 l_maj(maj, x, 8'b1110_1000);                // bit i = majority(i)
    lut3 l_xor(xor3, x, 8'b1001_0110);               // bit i = parity(i)
    integer i, j, errors = 0;
    initial begin
        for (i = 0; i < 4; i = i + 1) begin
            a = i;
            #1 $display("decoder %b -> %b", a, onehot);
        end
        for (i = 0; i < 16; i = i + 1)
            for (j = 0; j < 4; j = j + 1) begin
                d = i; a = j;
                #1 if (y1 !== d[a] || y2 !== d[a]) errors = errors + 1;
            end
        $display("mux checks over 64 cases: %0d errors", errors);
        for (i = 0; i < 8; i = i + 1) begin
            x = i;
            #1 $display("lut input %b: majority %b, parity %b", x, maj, xor3);
        end
    end
endmodule
`endif
```

```console
$ iverilog -g2012 -o blocks blocks.v && vvp -n blocks
decoder 00 -> 0001
decoder 01 -> 0010
decoder 10 -> 0100
decoder 11 -> 1000
mux checks over 64 cases: 0 errors
lut input 000: majority 0, parity 0
lut input 001: majority 0, parity 1
lut input 010: majority 0, parity 1
lut input 011: majority 1, parity 0
lut input 100: majority 0, parity 1
lut input 101: majority 1, parity 0
lut input 110: majority 1, parity 0
lut input 111: majority 1, parity 1
$ for top in dec2to4 mux4 lut3; do echo -n "$top: "; yosys -p "read_verilog blocks.v; synth -top $top; stat" | grep "Number of cells" | tail -1 | tr -s ' '; done
dec2to4:  Number of cells: 4
mux4:  Number of cells: 3
lut3:  Number of cells: 7
```

The two `lut3` instances are the same circuit with different configuration bits. Bit `i` of the configuration is the output for input `i`, so the configuration *is* the truth table written as a number: `1110_1000` has ones at positions 3, 5, 6, and 7 — the majority function. A field-programmable gate array (FPGA) is a grid of such look-up tables (typically with 6 inputs) plus programmable wiring; "programming" it means loading the configuration bits. This is how course labs build processors on real hardware without manufacturing chips.

### Pattern

**P-2. Test every combinational block exhaustively against an independent reference whenever the input space is small (up to about 2²⁰ combinations), and randomly plus at the corners when it is not.** *(Practice)* The reference must be written differently from the design — here, `d[a]` in the testbench checks two mux implementations. Writing the reference the same way as the design copies its bugs. *Trade-off:* exhaustive tests grow as 2ⁿ: a 32-bit adder has 2⁶⁴ input pairs, so it is tested on random values plus carefully chosen edge cases (all ones, alternating bits, carry chains), or proved correct with formal tools (Chapter 3).

### Mastery check

1. **Write.** The configuration bits of `lut3` for `y = x[2] & ~x[0]`.
2. **Explain.** How many 2:1 multiplexers does a 16:1 multiplexer tree need?
3. **Predict.** What does `dec2to4` output if `a` is `2'bx1`?

## Unit 1.4 — Adders: ripple-carry and carry-lookahead

**One-line summary:** A full adder adds three bits into a sum and a carry; chaining n of them gives a **ripple-carry adder** whose delay grows linearly with n, while a **carry-lookahead** adder computes carries from *generate* and *propagate* signals in parallel and brings the delay down to logarithmic, at the cost of more gates.

A **half adder** adds two bits: `s = a ⊕ b`, `c = a · b`. A **full adder** also takes a carry in: `s = a ⊕ b ⊕ cin`, `cout = ab + cin(a ⊕ b)`. Chaining full adders, carry out to carry in, adds n-bit numbers; but bit i cannot finish until the carry from bit i − 1 arrives, so the worst-case delay — a carry rippling from bit 0 to bit n − 1 — is proportional to n.

**Carry-lookahead** rewrites the carry using two signals per bit: **generate** `g_i = a_i b_i` (this bit produces a carry regardless) and **propagate** `p_i = a_i ⊕ b_i` (this bit passes an incoming carry on). Then `c_{i+1} = g_i + p_i c_i`, and unrolling it:

```
c1 = g0 + p0·c0
c2 = g1 + p1·g0 + p1·p0·c0
c3 = g2 + p2·g1 + p2·p1·g0 + p2·p1·p0·c0
```

Each carry is now a two-level expression of the inputs, and a *prefix tree* (Kogge–Stone, Brent–Kung) combines (g, p) pairs across groups in log₂ n levels.

```verilog
// file: adders.v
module ripple #(parameter N = 16)(input [N-1:0] a, b, input cin, output [N-1:0] s, output cout);
    wire [N:0] c;
    assign c[0] = cin;
    genvar i;
    generate
        for (i = 0; i < N; i = i + 1) begin : fa
            assign s[i]     = a[i] ^ b[i] ^ c[i];
            assign c[i + 1] = (a[i] & b[i]) | (c[i] & (a[i] ^ b[i]));
        end
    endgenerate
    assign cout = c[N];
endmodule

module kogge_stone #(parameter N = 16)(input [N-1:0] a, b, input cin, output [N-1:0] s, output cout);
    localparam L = $clog2(N);
    wire [N-1:0] g [0:L];
    wire [N-1:0] p [0:L];
    // level 0: per-bit generate and propagate; fold cin into bit 0's generate
    assign g[0] = (a & b) | {{(N-1){1'b0}}, (a[0] ^ b[0]) & cin};
    assign p[0] = a ^ b;
    genvar k, i;
    generate
        for (k = 0; k < L; k = k + 1) begin : level
            for (i = 0; i < N; i = i + 1) begin : node
                if (i >= (1 << k)) begin
                    assign g[k + 1][i] = g[k][i] | (p[k][i] & g[k][i - (1 << k)]);
                    assign p[k + 1][i] = p[k][i] & p[k][i - (1 << k)];
                end else begin
                    assign g[k + 1][i] = g[k][i];
                    assign p[k + 1][i] = p[k][i];
                end
            end
        end
    endgenerate
    wire [N:0] c = {g[L], cin};                  // c[i+1] = group generate of bits i..0
    assign s = (a ^ b) ^ c[N-1:0];
    assign cout = c[N];
endmodule

`ifndef SYNTHESIS                                // Yosys defines SYNTHESIS: it skips the testbench
module tb;
    localparam N = 16;
    reg [N-1:0] a, b;
    reg cin;
    wire [N-1:0] s1, s2;
    wire c1, c2;
    ripple      #(N) r(a, b, cin, s1, c1);
    kogge_stone #(N) k(a, b, cin, s2, c2);
    integer i, errors = 0;
    reg [N:0] want;
    initial begin
        for (i = 0; i < 200000; i = i + 1) begin
            a = $random; b = $random; cin = $random;
            if (i < 4) begin a = {N{1'b1}}; b = i; cin = i[0]; end      // carry-chain corner cases
            #1 want = a + b + cin;
            if ({c1, s1} !== want || {c2, s2} !== want) errors = errors + 1;
        end
        $display("%0d random + corner additions, %0d errors", i, errors);
    end
endmodule
`endif
```

```console
$ iverilog -g2012 -o adders adders.v && vvp -n adders
200000 random + corner additions, 0 errors
$ for top in ripple kogge_stone; do yosys -p "read_verilog adders.v; synth -top $top -noabc; stat; ltp -noff" | grep -E "Number of cells|Longest topological" | tail -2 | tr -s ' ' | tr '\n' ' ' | sed "s/^/$top: /"; echo; done
ripple:  Number of cells: 80 Longest topological path in ripple (length=33): 
kogge_stone:  Number of cells: 182 Longest topological path in kogge_stone (length=11): 
```

`ltp` reports the longest topological path — the number of gates on the deepest path from an input to an output — which is a proxy for delay before real timing analysis (Chapter 3). For 16 bits, the ripple adder's path is 33 gates (two per bit, plus one) and grows linearly with the width; the Kogge–Stone adder's path is 11 gates and grows with log₂ of the width, for more than twice the gates. `-noabc` tells Yosys to keep the circuit as written and only map it to simple gates; the next paragraph shows why that matters here.

Run the same comparison *with* Yosys's default logic optimiser (ABC), and both adders come out with about 80 gates and a path of 33: the optimiser, minimising area with no timing goal, rebuilt the Kogge–Stone adder as a ripple adder. Logic synthesis optimises for whatever goal it is given; an adder architecture survives only if the tool is told the delay matters (with real timing constraints and a cell library, Chapter 3), or if the structure is preserved explicitly. `generate` loops instantiate hardware repeatedly at elaboration time; they are not loops that run in time.

### Deep & Tricky

**DT-7. A Verilog expression is sized by its *context*: in `assign s = a + b;` with 8-bit `a`, `b`, and `s`, the carry out is silently discarded, while `assign {cout, s} = a + b;` makes the context 9 bits wide and keeps it. The same rule makes `(a + b) >> 1` lose the carry before shifting when the result is assigned to an 8-bit variable — the average of 200 and 100 becomes 22.** *(IEEE 1364-2005 §5.4.1 "Rules for expression bit lengths": the operands of `+` are extended to the size of the largest operand *and the left-hand side* of the assignment)* Linters warn about width mismatches (`verilator --lint-only -Wall`); the experiment below shows the wrong and the right average.

```verilog
// file: widths.v
module tb;
    reg [7:0] a = 200, b = 100;
    reg [7:0] avg_wrong, avg_right;
    reg [8:0] sum9;
    initial begin
        avg_wrong = (a + b) >> 1;                    // context is 8 bits: 300 wraps to 44, then 22
        sum9 = a + b;                                // 9-bit context keeps the carry
        avg_right = sum9 >> 1;
        $display("avg_wrong = %0d, avg_right = %0d", avg_wrong, avg_right);
        $display("in a 9-bit context directly: %0d", ({1'b0, a} + {1'b0, b}) >> 1);
    end
endmodule
```

```console
$ iverilog -g2012 -o widths widths.v && vvp -n widths
avg_wrong = 22, avg_right = 150
in a 9-bit context directly: 150
```

The first line shows the bug: in the 8-bit context of the assignment, 200 + 100 = 300 wraps to 44 before the shift, and 44 / 2 = 22. Widening the context — a 9-bit intermediate, or zero-extending the operands with `{1'b0, a}` — keeps the carry. Note that the shift *amount* (`1`, a 32-bit integer) does not widen the expression: a shift's right operand is self-determined (§5.4.1, Table 5-22), a detail that makes width rules subtle enough for real projects to run a linter on every file. But look at what the linter reports:

```console
$ verilator --lint-only -Wall widths.v 2>&1 | grep -E "^%Warning" | sed -E 's/: Operator/:\n    Operator/' | cut -c1-110
%Warning-DECLFILENAME: widths.v:2:8: Filename 'widths' does not match MODULE name: 'tb'
%Warning-WIDTHTRUNC: widths.v:9:19:
    Operator ASSIGN expects 8 bits on the Assign RHS, but Assign RHS's SHIFTR generates 9 bits.
```

### Mastery check

1. **Calculate.** In `ripple` with N = 16, how many full adders does the worst-case carry pass through?
2. **Explain.** Why does folding `cin` into `g[0][0]` make the Kogge–Stone adder handle a carry in?
3. **Predict.** In Verilog, what is `4'b1111 + 4'b0001` assigned to a 4-bit `reg`? To a 5-bit one?

## Topic 1C — Numbers and the ALU

## Unit 1.5 — Two's complement, overflow, and comparison

**One-line summary:** In n-bit two's complement, the top bit has weight −2ⁿ⁻¹, so the same adder adds signed and unsigned numbers; overflow means the true result does not fit, which for signed addition happens exactly when both operands have the same sign and the result's sign differs.

An n-bit pattern `b_{n−1} … b_0` means `Σ b_i 2^i` as an **unsigned** number and `−b_{n−1} 2^{n−1} + Σ_{i<n−1} b_i 2^i` as a **signed** (two's complement) number. For 4 bits, `1111` is 15 unsigned and −1 signed; `1000` is 8 unsigned and −8 signed. Negation is "invert and add one": `−x = ~x + 1`. Subtraction is addition of the negation: `a − b = a + ~b + 1`, which an adder does by inverting `b` and setting the carry in to 1 — one adder serves for both.

Overflow is different for the two interpretations:

- **Unsigned** overflow: the carry out of the top bit is 1 (for addition), or 0 (for subtraction — a *borrow*).
- **Signed** overflow: the two inputs to the top bit's addition have the same sign and the result has the other sign. Equivalently, the carry *into* the top bit differs from the carry *out of* it.

Comparison falls out of subtraction. For unsigned `a < b`: compute `a − b`; there was a borrow. For signed `a < b`: the sign of `a − b` XOR the signed overflow flag — the overflow corrects the sign when the subtraction wrapped.

```verilog
// file: compare.v
module flags #(parameter N = 8)(input [N-1:0] a, b, output lt_signed, lt_unsigned, output ovf_sub);
    wire [N:0] diff = {1'b0, a} + {1'b0, ~b} + 1;          // a - b with the carry kept
    wire sign = diff[N-1];
    assign ovf_sub    = (a[N-1] != b[N-1]) && (sign != a[N-1]);  // signed overflow of a - b
    assign lt_signed  = sign ^ ovf_sub;
    assign lt_unsigned = ~diff[N];                          // no carry out = a borrow happened
endmodule

module tb;
    reg [7:0] a, b;
    wire lts, ltu, ovf;
    flags #(8) f(a, b, lts, ltu, ovf);
    integer i, j, errors = 0;
    initial begin
        for (i = 0; i < 256; i = i + 1)
            for (j = 0; j < 256; j = j + 1) begin
                a = i; b = j;
                #1;
                if (lts !== ($signed(a) < $signed(b)) || ltu !== (a < b)) errors = errors + 1;
            end
        $display("exhaustive 8-bit compare: %0d errors over 65536 pairs", errors);
        a = 8'd100; b = -8'sd100;
        #1 $display("100 - (-100): signed overflow %b, 100 <s -100: %b, 100 <u 156: %b", ovf, lts, ltu);
    end
endmodule
```

```console
$ iverilog -g2012 -o compare compare.v && vvp -n compare
exhaustive 8-bit compare: 0 errors over 65536 pairs
100 - (-100): signed overflow 1, 100 <s -100: 0, 100 <u 156: 1
```

`$signed(a)` reinterprets a vector as signed for the comparison in the testbench — the independent reference. 100 − (−100) = 200, which does not fit in a signed 8-bit number (maximum 127), so the subtraction overflows: the raw difference `11001000` looks negative (−56), and without the overflow correction the comparison would say 100 < −100.

### Deep & Tricky

**DT-8. In Verilog, if *any* operand of an expression is unsigned, the whole expression is evaluated as unsigned — so comparing a `signed` register with an unsigned one, or with a part-select (which is always unsigned), silently becomes an unsigned comparison, and −1 < 0 is false.** *(IEEE 1364-2005 §5.5.1 "Rules for expression types": "If any operand is unsigned, the result is unsigned, regardless of the operator"; part-selects are unsigned, §5.5.1)* The ISA has the same split for the same reason: RISC-V has both `slt` and `sltu`, and `blt` and `bltu` (Chapter 4), because the bits do not say which interpretation is meant.

```verilog
// file: signedness.v
module tb;
    reg signed [7:0] x = -1;
    reg        [7:0] zero_u = 0;
    reg signed [7:0] zero_s = 0;
    initial begin
        $display("signed -1 < signed 0:   %b", x < zero_s);
        $display("signed -1 < unsigned 0: %b  (whole comparison became unsigned: 255 < 0)", x < zero_u);
        $display("signed -1 < 0:          %b  (an unsized literal is a signed integer)", x < 0);
        $display("signed -1 < 8'd0:       %b  (a sized literal like 8'd0 is unsigned)", x < 8'd0);
    end
endmodule
```

```console
$ iverilog -g2012 -o signedness signedness.v && vvp -n signedness
signed -1 < signed 0:   1
signed -1 < unsigned 0: 0  (whole comparison became unsigned: 255 < 0)
signed -1 < 0:          1  (an unsized literal is a signed integer)
signed -1 < 8'd0:       0  (a sized literal like 8'd0 is unsigned)
```

### Mastery check

1. **Calculate.** What is `11110110` as an unsigned number? As a signed one? What is its negation, in binary?
2. **Predict.** 8-bit signed: does `-128 - 1` overflow? `-128 + -1`? What bits does the adder produce?
3. **Explain.** Why is signed overflow impossible when adding a positive and a negative number?

## Unit 1.6 — An ALU, and shifters

**One-line summary:** An arithmetic-logic unit computes one of several operations — add, subtract, AND, OR, XOR, shifts, set-less-than — on two operands, selected by a control input; it is a set of functional units in parallel followed by a multiplexer, and it is the heart of every processor's execute stage.

The ALU below implements the operations RISC-V's integer instructions need (Chapter 4 uses it unchanged). A **barrel shifter** shifts by any amount in one step: log₂ n levels of 2:1 multiplexers, level k shifting by 2ᵏ or not.

```verilog
// file: alu.v
module alu #(parameter N = 32)(
    input  [N-1:0] a, b,
    input  [3:0]   op,
    output reg [N-1:0] y,
    output zero
);
    localparam ADD = 4'd0, SUB = 4'd1, SLL = 4'd2, SLT = 4'd3, SLTU = 4'd4,
               XOR = 4'd5, SRL = 4'd6, SRA = 4'd7, OR = 4'd8, AND = 4'd9;
    wire [4:0] shamt = b[4:0];
    always @(*) begin
        case (op)
            ADD:  y = a + b;
            SUB:  y = a - b;
            SLL:  y = a << shamt;
            SLT:  y = {{(N-1){1'b0}}, $signed(a) < $signed(b)};
            SLTU: y = {{(N-1){1'b0}}, a < b};
            XOR:  y = a ^ b;
            SRL:  y = a >> shamt;
            SRA:  y = $signed(a) >>> shamt;
            OR:   y = a | b;
            AND:  y = a & b;
            default: y = {N{1'bx}};
        endcase
    end
    assign zero = (y == 0);
endmodule

`ifndef SYNTHESIS                                // Yosys defines SYNTHESIS: it skips the testbench
module tb;
    reg [31:0] a, b;
    reg [3:0] op;
    wire [31:0] y;
    wire zero;
    alu #(32) dut(a, b, op, y, zero);
    reg [31:0] want;
    integer i, errors = 0;
    initial begin
        for (i = 0; i < 100000; i = i + 1) begin
            a = $random; b = $random; op = $urandom % 10;
            if (i % 7 == 0) b = b % 40;                       // small shift amounts and edge values
            #1;
            case (op)
                0: want = a + b;                      1: want = a - b;
                2: want = a << b[4:0];                3: want = ($signed(a) < $signed(b)) ? 1 : 0;
                4: want = (a < b) ? 1 : 0;            5: want = a ^ b;
                6: want = a >> b[4:0];                7: want = $signed(a) >>> b[4:0];
                8: want = a | b;                      9: want = a & b;
            endcase
            if (y !== want) begin
                errors = errors + 1;
                if (errors < 4) $display("op %0d a=%h b=%h: got %h expected %h", op, a, b, y, want);
            end
        end
        $display("%0d random ALU operations, %0d errors", i, errors);
        a = 32'h8000_0000; b = 4; op = 7; #1 $display("SRA 0x80000000 by 4 = %h, SRL = %h", y, a >> 4);
    end
endmodule
`endif
```

```console
$ iverilog -g2012 -o alu alu.v && vvp -n alu
100000 random ALU operations, 0 errors
SRA 0x80000000 by 4 = f8000000, SRL = 08000000
$ yosys -p "read_verilog alu.v; synth -top alu; stat" | grep -E "Number of cells" | tail -1 | tr -s ' '
 Number of cells: 1432
```

The `always @(*)` block with a `case` statement describes combinational logic procedurally: whenever any input changes, the block runs and computes `y`. Every path must assign `y` — otherwise the synthesis tool infers a *latch* to remember the old value, which is a different (sequential) circuit; the `default` branch prevents that. Chapter 3 returns to this rule. `>>>` is the arithmetic shift: on a signed operand it copies the sign bit into the vacated positions, so `0x80000000 >>> 4` is `0xF8000000`, while the logical shift gives `0x08000000`.

### Mastery check

1. **Explain.** Why does the ALU take only the low 5 bits of `b` for shifts?
2. **Predict.** What is `zero` when `op = SLTU`, `a = 5`, `b = 3`?
3. **Explain.** Which single ALU operation, followed by a test of `zero`, implements "branch if equal"?

## Chapter 1 Project — A 4-bit calculator circuit, verified

### Requirements

Build `calc4`, a combinational circuit with two 4-bit signed inputs, a 2-bit operation select (add, subtract, multiply, compare), an 8-bit signed output, and a 2-bit status (overflow of the 4-bit result for add and subtract; "less than" for compare). Implement the multiplier as an array of AND gates and adders (no `*` operator). Verify it exhaustively against a reference model that does use Verilog's `*` and signed arithmetic, report the gate count and the longest path, and drive a seven-segment display decoder from the low hex digit of the result.

### Reference solution

```verilog
// file: calc/calc4.v
module mul4s(input signed [3:0] a, b, output signed [7:0] p);   // signed 4x4 array multiplier
    wire [7:0] ae = {{4{a[3]}}, a};                             // sign-extend to 8 bits
    wire [7:0] be = {{4{b[3]}}, b};
    wire [7:0] pp [0:7];
    genvar i;
    generate
        for (i = 0; i < 8; i = i + 1) begin : row
            assign pp[i] = be[i] ? (ae << i) : 8'd0;            // AND gates: partial product i
        end
    endgenerate
    assign p = pp[0] + pp[1] + pp[2] + pp[3] + pp[4] + pp[5] + pp[6] + pp[7];   // modulo 2^8
endmodule

module calc4(input signed [3:0] a, b, input [1:0] op, output reg signed [7:0] y, output reg [1:0] status);
    wire signed [7:0] prod;
    mul4s m(a, b, prod);
    wire signed [4:0] sum = a + b, diff = a - b;
    always @(*) begin
        status = 2'b00;
        case (op)
            2'd0: begin y = sum;  status[0] = (sum  > 7 || sum  < -8); end
            2'd1: begin y = diff; status[0] = (diff > 7 || diff < -8); end
            2'd2: y = prod;
            default: begin y = 0; status[1] = a < b; end
        endcase
    end
endmodule

module hex7seg(input [3:0] h, output reg [6:0] seg);           // segments gfedcba, 1 = lit
    always @(*)
        case (h)
            4'h0: seg = 7'b0111111; 4'h1: seg = 7'b0000110; 4'h2: seg = 7'b1011011; 4'h3: seg = 7'b1001111;
            4'h4: seg = 7'b1100110; 4'h5: seg = 7'b1101101; 4'h6: seg = 7'b1111101; 4'h7: seg = 7'b0000111;
            4'h8: seg = 7'b1111111; 4'h9: seg = 7'b1101111; 4'hA: seg = 7'b1110111; 4'hB: seg = 7'b1111100;
            4'hC: seg = 7'b0111001; 4'hD: seg = 7'b1011110; 4'hE: seg = 7'b1111001; default: seg = 7'b1110001;
        endcase
endmodule
```

```verilog
// file: calc/tb_calc4.v
module tb;
    reg signed [3:0] a, b;
    reg [1:0] op;
    wire signed [7:0] y;
    wire [1:0] status;
    wire [6:0] seg;
    calc4 dut(a, b, op, y, status);
    hex7seg disp(y[3:0], seg);
    integer i, j, k, errors = 0;
    integer ref_y, ref_ovf, ref_lt;
    initial begin
        for (k = 0; k < 4; k = k + 1)
            for (i = -8; i < 8; i = i + 1)
                for (j = -8; j < 8; j = j + 1) begin
                    a = i; b = j; op = k;
                    #1;
                    case (k)
                        0: begin ref_y = i + j; ref_ovf = (i + j > 7) || (i + j < -8); ref_lt = 0; end
                        1: begin ref_y = i - j; ref_ovf = (i - j > 7) || (i - j < -8); ref_lt = 0; end
                        2: begin ref_y = i * j; ref_ovf = 0; ref_lt = 0; end
                        3: begin ref_y = 0;     ref_ovf = 0; ref_lt = i < j; end
                    endcase
                    if (y !== ref_y[7:0] || status !== {ref_lt[0], ref_ovf[0]}) begin
                        errors = errors + 1;
                        if (errors < 5) $display("op %0d %0d,%0d: got y=%0d status=%b", k, i, j, y, status);
                    end
                end
        $display("calc4: %0d of 1024 cases wrong", errors);
        a = -3; b = 5; op = 2; #1 $display("-3 * 5 = %0d (hex %h), segments %b", y, y, seg);
        a = 7;  b = 1; op = 0; #1 $display("7 + 1 = %0d, overflow flag %b", y, status[0]);
    end
endmodule
```

```console
$ cd calc && iverilog -g2012 -o calc calc4.v tb_calc4.v && vvp -n calc
calc4: 0 of 1024 cases wrong
-3 * 5 = -15 (hex f1), segments 0000110
7 + 1 = 8, overflow flag 1
$ cd calc && yosys -p "read_verilog calc4.v; synth -top calc4 -flatten; abc -g AND,OR,XOR,MUX; stat; ltp -noff" | grep -E "Number of cells|Longest" | tail -2 | tr -s ' '
 Number of cells: 204
Longest topological path in calc4 (length=16):
```

Design notes: the multiplier sign-extends both operands to 8 bits and adds eight shifted partial products modulo 2⁸; because two's complement arithmetic is modular, the low 8 bits of the product of the sign-extended operands are exactly the 8-bit signed product. Status bits use 5-bit intermediate results, so a 4-bit overflow is visible as a value outside −8..7. The testbench loops over *integers* and compares against Verilog's own `+`, `-`, `*`, and `<` on integers — an independent reference.

### Review checklist

- Is every output assigned on every path of every `always @(*)` block (no inferred latches)? Run `yosys ... ; stat` and check that no `$dlatch` cells appear.
- Does the overflow flag behave correctly for −8 − 1 and for 7 + 1?
- How would the multiplier's gate count grow for 8-bit or 16-bit inputs?

## Answers to the Chapter 1 prediction questions

**Unit 1.1, Q2.** `y` is not driven by anything when the nMOS path is off: Icarus prints `z` (high impedance) — the output is floating.

**Unit 1.2, Q3.** `x & 0` is `0` (a 0 forces AND to 0 whatever the other input), and `x | 1` is `1`. Verilog's tables return a known value whenever one operand decides the result.

**Unit 1.3, Q3.** `4'b0001 << x` with an unknown shift amount gives `xxxx` — every output unknown.

**Unit 1.4, Q3.** `0000` in a 4-bit `reg` (the carry is lost); `10000` (16) in a 5-bit one, because the 5-bit context widens the addition.

**Unit 1.5, Q2.** `-128 - 1` overflows (the true result −129 does not fit): `10000000 + 11111111 = 1_01111111`, giving `01111111` = 127. `-128 + -1` is the same computation and overflows the same way.

**Unit 1.6, Q2.** `y = 0` because 5 < 3 is false, so `zero = 1`.

# Chapter 2 — Sequential Logic and Finite State Machines

*Goal: give circuits memory. A **sequential** circuit's outputs depend on its inputs and on what it has seen before, stored in latches and flip-flops. This chapter builds storage from gates with feedback, then adopts the discipline every real design uses — synchronous logic, where all state changes on the edge of one clock — and uses it to build registers, counters, shift registers, memories, and finite state machines.*

*Accompanies the lecture "Sequential Logic Design" (and the parts of "Hardware Description Languages and Verilog" about sequential logic).*

**What you will be able to do after this chapter**

- Explain how cross-coupled gates store a bit, and why an SR latch has a forbidden input.
- Tell a level-sensitive latch from an edge-triggered flip-flop, and build a flip-flop from two latches.
- Write sequential Verilog with `always @(posedge clk)` and non-blocking assignments, and say why blocking assignments there cause simulation races.
- Design Moore and Mealy finite state machines, choose a state encoding, and test them with a clocked testbench.
- Describe register files, RAMs, and ROMs in Verilog.

## Topic 2A — Storing a bit

## Unit 2.1 — Feedback: the SR latch and the D latch

**One-line summary:** Two cross-coupled NOR gates form an **SR latch** that holds a bit indefinitely and can be set or reset; adding gating logic gives a **D latch**, which copies its input D to its output while a clock signal is high and holds the value while the clock is low.

A combinational circuit has no loops. Add one — feed a gate's output back to its own input through another gate — and the circuit can have two stable states. Two NOR gates, each fed by the other's output, form the **SR latch** (set/reset):

| S | R | Q next | Meaning |
| --- | --- | --- | --- |
| 0 | 0 | Q (unchanged) | hold |
| 1 | 0 | 1 | set |
| 0 | 1 | 0 | reset |
| 1 | 1 | 0, and Q̄ = 0 too | forbidden: Q and Q̄ are no longer complements |

The last row is "forbidden" for a practical reason: if S and R both fall from 1 to 0 at the same time, both NOR gates see 0 and 0, both outputs rise, which turns both off again — the latch can **oscillate** or settle in an unpredictable state. The **D latch** removes the problem by deriving S and R from one data input: `S = D·CLK`, `R = D'·CLK`. When CLK is 1 the latch is *transparent* (Q follows D); when CLK is 0 it holds.

```verilog
// file: latches.v
`timescale 1ns/1ps
module sr_latch(output q, output qn, input s, input r);
    nor #1 g1(q,  r, qn);           // #1: each gate takes 1 ns
    nor #1 g2(qn, s, q);
endmodule

module d_latch(output q, input d, input clk);
    wire s = d & clk;
    wire r = ~d & clk;
    wire qn;
    sr_latch core(q, qn, s, r);
endmodule

module tb;
    reg s = 0, r = 1;                                 // start by resetting, so Q is known
    wire q, qn;
    sr_latch l(q, qn, s, r);
    reg d = 0, clk = 0;
    wire dq;
    d_latch dl(dq, d, clk);
    initial begin
        $timeformat(-9, 0, "", 5);                    // print times in ns
        $display(" time S R | Q Qn");
        $monitor("%5t %b %b | %b %b", $time, s, r, q, qn);
        #5 r = 0;                  // hold 0
        #5 s = 1;                  // set
        #5 s = 0;                  // hold 1: the loop remembers
        #5 r = 1;                  // reset
        #5 r = 0;
        #5 s = 1; r = 1;           // forbidden: both outputs 0
        #5 s = 0; r = 0;           // release both at once
        #20 $monitoroff;
        $display("D latch: transparent while clk = 1, holds while clk = 0");
        clk = 1; d = 1; #5 $display("  clk=1 d=1 -> q=%b", dq);
        d = 0;          #5 $display("  clk=1 d=0 -> q=%b (follows)", dq);
        d = 1; #5 clk = 0; #5 $display("  clk=0 after d=1 -> q=%b", dq);
        d = 0;          #5 $display("  clk=0 d=0 -> q=%b (holds)", dq);
        $finish;
    end
endmodule
```

```console
$ iverilog -g2012 -o latches latches.v && vvp -n latches | head -40
 time S R | Q Qn
    0 0 1 | x x
    1 0 1 | 0 x
    2 0 1 | 0 1
    5 0 0 | 0 1
   10 1 0 | 0 1
   11 1 0 | 0 0
   12 1 0 | 1 0
   15 0 0 | 1 0
   20 0 1 | 1 0
   21 0 1 | 0 0
   22 0 1 | 0 1
   25 0 0 | 0 1
   30 1 1 | 0 1
   31 1 1 | 0 0
   35 0 0 | 0 0
   36 0 0 | 1 1
   37 0 0 | 0 0
   38 0 0 | 1 1
   39 0 0 | 0 0
   40 0 0 | 1 1
   41 0 0 | 0 0
   42 0 0 | 1 1
   43 0 0 | 0 0
   44 0 0 | 1 1
   45 0 0 | 0 0
   46 0 0 | 1 1
   47 0 0 | 0 0
   48 0 0 | 1 1
   49 0 0 | 0 0
   50 0 0 | 1 1
   51 0 0 | 0 0
   52 0 0 | 1 1
   53 0 0 | 0 0
   54 0 0 | 1 1
D latch: transparent while clk = 1, holds while clk = 0
  clk=1 d=1 -> q=1
  clk=1 d=0 -> q=0 (follows)
  clk=0 after d=1 -> q=1
  clk=0 d=0 -> q=1 (holds)
```

The `$monitor` prints a line every time a signal changes, so the trace shows the gate delays (the D-latch test waits 5 ns after changing `d` before lowering the clock; with only 1 ns — less than the two gate delays around the loop — the latch captured a half-finished transition, which is a small preview of Chapter 3's *setup time*): after S rises at 10 ns, Qn falls at 11 ns and Q rises at 12 ns — the change propagates around the loop. When S and R are released together at 35 ns, the latch **oscillates**: in this simulation, both gates switch at exactly the same time forever (until the trace is cut off), because the model has perfectly equal delays. A real latch would eventually fall to one side, at a time and in a direction no one can predict — the same phenomenon, **metastability**, that Chapter 3 meets when an input changes too close to a clock edge.

### Mastery check

1. **Explain.** Why does the D latch never see the forbidden S = R = 1 input?
2. **Predict.** In the SR latch, if S rises to 1 while R is already 1, what are Q and Qn after the gates settle?
3. **Explain.** Why is a latch called "transparent" while its clock is high?

## Unit 2.2 — The D flip-flop and registers

**One-line summary:** A **D flip-flop** samples D only at the rising edge of the clock and holds the sampled value for the whole cycle; it is two D latches in series with opposite clock phases (master–slave), and a row of flip-flops sharing a clock is a **register**.

A latch is transparent for half the clock period, so a value can race through several latches in one phase. A **flip-flop** closes that window to an instant. Put two D latches in series: the first (the *master*) is transparent while CLK is 0, the second (the *slave*) while CLK is 1. While CLK is 0 the master follows D and the slave holds; when CLK rises, the master freezes the value D had just before the edge and the slave passes it to Q. At no time is there a transparent path from D to Q.

In Verilog, a flip-flop is an `always` block triggered by the clock edge:

```verilog
// file: flops.v
`timescale 1ns/1ps
module dff_ms(output q, input d, input clk);          // master-slave from two latches
    reg m, s;
    always @(*) if (!clk) m = d;                       // master: transparent while clk = 0
    always @(*) if (clk)  s = m;                       // slave:  transparent while clk = 1
    assign q = s;
endmodule

module dff(output reg q, input d, input clk);         // the usual way to write a flip-flop
    always @(posedge clk) q <= d;
endmodule

module reg_en_rst #(parameter N = 8)(output reg [N-1:0] q, input [N-1:0] d,
                                     input clk, input rst, input en);
    always @(posedge clk)
        if (rst)     q <= 0;                           // synchronous reset
        else if (en) q <= d;                           // load only when enabled
endmodule

module tb;
    reg clk = 0, d = 0;
    always #5 clk = ~clk;                              // 10 ns period
    wire q1, q2;
    dff_ms a(q1, d, clk);
    dff    b(q2, d, clk);
    reg rst = 1, en = 0;
    reg [7:0] din = 8'hAA;
    wire [7:0] r;
    reg_en_rst #(8) rg(r, din, clk, rst, en);
    initial begin
        $timeformat(-9, 0, "", 5);
        $display(" time clk d | ms dff | reg");
        $monitor("%5t  %b  %b |  %b   %b  | %h", $time, clk, d, q1, q2, r);
        #12 d = 1;          // changes mid-cycle: both flops wait for the next rising edge
        #4  d = 0;          // a short pulse between edges is never seen
        #8  d = 1;
        rst = 0;
        #10 en = 1;
        #10 din = 8'h55; en = 0;
        #20 $finish;
    end
endmodule
```

```console
$ iverilog -g2012 -o flops flops.v && vvp -n flops
 time clk d | ms dff | reg
    0  0  0 |  x   x  | xx
    5  1  0 |  x   0  | 00
   10  0  0 |  x   0  | 00
   12  0  1 |  x   0  | 00
   15  1  1 |  1   1  | 00
   16  1  0 |  1   1  | 00
   20  0  0 |  1   1  | 00
   24  0  1 |  1   1  | 00
   25  1  1 |  1   1  | 00
   30  0  1 |  1   1  | 00
   35  1  1 |  1   1  | aa
   40  0  1 |  1   1  | aa
   45  1  1 |  1   1  | aa
   50  0  1 |  1   1  | aa
   55  1  1 |  1   1  | aa
   60  0  1 |  1   1  | aa
flops.v:41: $finish called at 64000 (1ps)
```

Follow `d` and the two flip-flop outputs: `d` rises at 12 ns, but `q` changes only at the rising clock edge at 15 ns. The pulse from 12 ns to 16 ns is caught because it was present at the edge at 15 ns; had it ended before 15 ns, no flip-flop would ever have seen it. The register ignores `din` until `en` is 1 at a rising edge, then loads `AA`, and keeps `AA` when `din` changes to `55` with `en` back at 0.

### Deep & Tricky

**DT-9. In sequential `always @(posedge clk)` blocks, use non-blocking assignments (`<=`); in combinational `always @(*)` blocks, use blocking ones (`=`). With blocking assignments, two flip-flops clocked by the same edge can read each other's *new* value or old value depending on which block the simulator happens to run first — a simulation race that does not exist in hardware.** *(IEEE 1364-2005 §11.4 "Scheduling semantics": non-blocking assignments evaluate their right-hand sides immediately but update in the NBA region after all active events; the order in which `always` blocks triggered by the same event execute is not defined; Cummings, "Nonblocking Assignments in Verilog Synthesis, Coding Styles That Kill!", SNUG 2000)* A two-register swap shows it:

```verilog
// file: race.v
module swap_blocking(input clk, output reg [3:0] a, output reg [3:0] b);
    initial begin a = 1; b = 2; end
    always @(posedge clk) a = b;        // blocking: updates immediately
    always @(posedge clk) b = a;
endmodule

module swap_nonblocking(input clk, output reg [3:0] a, output reg [3:0] b);
    initial begin a = 1; b = 2; end
    always @(posedge clk) a <= b;       // non-blocking: reads old values, updates together
    always @(posedge clk) b <= a;
endmodule

module tb;
    reg clk = 0;
    wire [3:0] a1, b1, a2, b2;
    swap_blocking    s1(clk, a1, b1);
    swap_nonblocking s2(clk, a2, b2);
    initial begin
        #1 clk = 1; #1 clk = 0;
        $display("after one edge: blocking a=%0d b=%0d   non-blocking a=%0d b=%0d", a1, b1, a2, b2);
        #1 clk = 1; #1 clk = 0;
        $display("after two edges: blocking a=%0d b=%0d   non-blocking a=%0d b=%0d", a1, b1, a2, b2);
    end
endmodule
```

```console
$ iverilog -g2012 -o race race.v && vvp -n race
after one edge: blocking a=1 b=1   non-blocking a=2 b=1
after two edges: blocking a=1 b=1   non-blocking a=1 b=2
$ verilator --lint-only -Wall --top-module swap_blocking race.v 2>&1 | grep -oE "%Warning-[A-Z]+" | sort -u
%Warning-BLKSEQ
%Warning-DECLFILENAME
```

The non-blocking pair swaps on every edge, as two real flip-flops would. The blocking pair copies one value into both registers — and which value depends on the simulator's internal order, so a different simulator (or a different version) may produce the other answer. Verilator's lint reports blocking assignments in clocked blocks (`BLKSEQ`).

### Mastery check

1. **Explain.** Why does a master–slave flip-flop not have a transparent path from D to Q?
2. **Predict.** In `tb` of `flops.v`, if `d` had been a pulse from 16 ns to 24 ns, would `q` ever show it?
3. **Explain.** What is the difference between a synchronous and an asynchronous reset in terms of *when* the register clears?

## Topic 2B — Synchronous design

## Unit 2.3 — Counters, shift registers, and the LFSR

**One-line summary:** In a synchronous circuit, combinational logic computes each register's next value from the current values and the inputs, and every register updates at the same clock edge; counters, shift registers, and linear-feedback shift registers are the simplest examples, and the same pattern scales to whole processors.

Every synchronous design has the same shape:

```
            +--------------------+
inputs ---->|   combinational    |----> outputs
      +---->|   next-state logic |--+
      |     +--------------------+  |
      |     +-----------+           |
      +-----| registers |<----------+   (all clocked by the same edge)
            +-----------+
```

The rule that makes it work: no loop goes through combinational logic alone — every feedback path passes through a register. Then the circuit's behaviour is a sequence of states, one per clock cycle, and the only timing question is whether the combinational logic settles within one clock period (Chapter 3).

```verilog
// file: counters.v
module counter #(parameter N = 4)(input clk, input rst, output reg [N-1:0] q);
    always @(posedge clk) q <= rst ? 0 : q + 1;
endmodule

module shift_reg #(parameter N = 8)(input clk, input din, output reg [N-1:0] q);
    always @(posedge clk) q <= {q[N-2:0], din};         // shift left, new bit at the bottom
endmodule

module lfsr8(input clk, input rst, output reg [7:0] q);  // maximal-length: taps 8,6,5,4
    wire fb = q[7] ^ q[5] ^ q[4] ^ q[3];
    always @(posedge clk) q <= rst ? 8'h01 : {q[6:0], fb};
endmodule

`ifndef SYNTHESIS
module tb;
    reg clk = 0, rst = 1, din = 0;
    always #5 clk = ~clk;
    wire [3:0] c;
    wire [7:0] sr, l;
    counter #(4) cnt(clk, rst, c);
    shift_reg #(8) sh(clk, din, sr);
    lfsr8 lf(clk, rst, l);
    integer cycle, period;
    reg [255:0] seen;
    initial begin
        #12 rst = 0;
        for (cycle = 0; cycle < 18; cycle = cycle + 1) begin
            din = (cycle % 3 == 0);
            @(posedge clk) #1 $display("cycle %2d: counter %2d  shift %b  lfsr %h", cycle, c, sr, l);
        end
        seen = 0; period = 0;
        while (!seen[l]) begin seen[l] = 1; period = period + 1; @(posedge clk) #1; end
        $display("LFSR period: %0d distinct states before repeating (2^8 - 1 = 255)", period);
        $finish;
    end
endmodule
`endif
```

```console
$ iverilog -g2012 -o counters counters.v && vvp -n counters | tail -8
cycle 12: counter 13  shift 01001001  lfsr 89
cycle 13: counter 14  shift 10010010  lfsr 12
cycle 14: counter 15  shift 00100100  lfsr 25
cycle 15: counter  0  shift 01001001  lfsr 4b
cycle 16: counter  1  shift 10010010  lfsr 97
cycle 17: counter  2  shift 00100100  lfsr 2e
LFSR period: 255 distinct states before repeating (2^8 - 1 = 255)
counters.v:35: $finish called at 2736 (1s)
$ yosys -p "read_verilog counters.v; synth -top lfsr8" | sed -n '/Printing statistics/,$p' | grep -E '\$_' | tr -s ' '
 $_SDFF_PP0_ 7
 $_SDFF_PP1_ 1
 $_XNOR_ 2
 $_XOR_ 1
```

The 4-bit counter wraps from 15 to 0 — arithmetic modulo 2⁴. The LFSR (linear-feedback shift register) shifts left and feeds back the XOR of a few bits ("taps"); with the right taps it visits all 255 non-zero states before repeating, so with 8 flip-flops and three XOR-type gates it generates a pseudo-random sequence. (Yosys reports the flip-flops as `$_SDFF_PP0_` and `$_SDFF_PP1_` — flip-flops with a synchronous reset to 0 or to 1 built in — and uses XNOR where it is cheaper; the reset value `01` explains the one flip-flop that resets to 1.) LFSRs are everywhere: scrambling data on serial links, generating test patterns, computing CRC checksums.

### Mastery check

1. **Predict.** An 8-bit LFSR is reset to `00`. What sequence does it produce?
2. **Design.** Write a counter that counts 0, 1, …, 9, 0, … (a decade counter). What extra logic does it need compared with a 4-bit binary counter?
3. **Explain.** Why is a combinational loop (`assign a = ~a;`) not allowed in synchronous design?

## Unit 2.4 — Finite state machines

**One-line summary:** A finite state machine (FSM) is a state register plus next-state logic plus output logic; in a **Moore** machine the outputs depend only on the state, in a **Mealy** machine also on the current inputs — which makes Mealy outputs respond one cycle earlier, and makes them combinational paths from input to output.

Designing an FSM has four steps: list the states, draw the transitions, choose an encoding, and write the logic. Example: detect the input sequence `1101` on a serial line (overlapping matches allowed, so `1101101` contains two).

Moore version: the states record how much of the pattern has been seen: S0 (nothing), S1 (`1`), S11 (`11`), S110 (`110`), and S1101 (match — output 1 in this state).

| State | in = 0 | in = 1 | Output |
| --- | --- | --- | --- |
| S0 | S0 | S1 | 0 |
| S1 | S0 | S11 | 0 |
| S11 | S110 | S11 | 0 |
| S110 | S0 | S1101 | 0 |
| S1101 | S0 | S11 | 1 |

From S1101 on input 1, the last `1` plus this `1` make `11`: the next state is S11, which is how overlapping matches are found.

```verilog
// file: fsm.v
module detect_moore(input clk, input rst, input in, output out);
    localparam S0 = 3'd0, S1 = 3'd1, S11 = 3'd2, S110 = 3'd3, S1101 = 3'd4;
    reg [2:0] state, next;
    always @(posedge clk) state <= rst ? S0 : next;          // state register
    always @(*) begin                                        // next-state logic
        case (state)
            S0:    next = in ? S1    : S0;
            S1:    next = in ? S11   : S0;
            S11:   next = in ? S11   : S110;
            S110:  next = in ? S1101 : S0;
            S1101: next = in ? S11   : S0;
            default: next = S0;
        endcase
    end
    assign out = (state == S1101);                            // output logic: state only
endmodule

module detect_mealy(input clk, input rst, input in, output out);
    localparam S0 = 2'd0, S1 = 2'd1, S11 = 2'd2, S110 = 2'd3;
    reg [1:0] state, next;
    always @(posedge clk) state <= rst ? S0 : next;
    always @(*) begin
        case (state)
            S0:   next = in ? S1  : S0;
            S1:   next = in ? S11 : S0;
            S11:  next = in ? S11 : S110;
            S110: next = in ? S1  : S0;        // after a match, the last 1 starts a new "1"
        endcase
    end
    assign out = (state == S110) && in;      // output depends on the input too
endmodule

`ifndef SYNTHESIS
module tb;
    reg clk = 0, rst = 1, in = 0;
    always #5 clk = ~clk;
    wire mo, me;
    detect_moore u1(clk, rst, in, mo);
    detect_mealy u2(clk, rst, in, me);
    reg [0:15] stream = 16'b1101101_0011_01101;
    integer i, moore_hits = 0, mealy_hits = 0;
    initial begin
        @(posedge clk) #1 rst = 0;
        $display("cycle in | mealy moore");
        for (i = 0; i < 16; i = i + 1) begin
            in = stream[i];
            #1 $display("%5d  %b |   %b     %b", i, in, me, mo);   // sample just after the input changes
            mealy_hits = mealy_hits + me;
            @(posedge clk) #1;
            moore_hits = moore_hits + mo;
        end
        $display("matches: mealy %0d, moore %0d", mealy_hits, moore_hits);
        $finish;
    end
endmodule
`endif
```

```console
$ iverilog -g2012 -o fsm fsm.v && vvp -n fsm
cycle in | mealy moore
    0  1 |   0     0
    1  1 |   0     0
    2  0 |   0     0
    3  1 |   1     0
    4  1 |   0     1
    5  0 |   0     0
    6  1 |   1     0
    7  0 |   0     1
    8  0 |   0     0
    9  1 |   0     0
   10  1 |   0     0
   11  0 |   0     0
   12  1 |   1     0
   13  1 |   0     1
   14  0 |   0     0
   15  1 |   1     0
matches: mealy 4, moore 4
fsm.v:54: $finish called at 166 (1s)
```

Both machines find the same four matches in `1101101 0011 01101` (the overlapping pair in the first seven bits, and two more in the last five bits, which overlap too), but the Mealy output rises in the *same* cycle as the final `1` arrives, while the Moore output rises one cycle later, when the machine has entered S1101. The Mealy machine needs one state fewer. The price is that its output is a combinational function of the input: a glitch on the input can appear on the output, and in a large design such paths chain through several modules and lengthen the critical path. Many design teams prefer Moore outputs, or register Mealy outputs, for that reason.

### State encoding

The machine's states need not be numbered 0, 1, 2, … . **One-hot** encoding uses one flip-flop per state (exactly one is 1): more flip-flops, simpler next-state logic, and on FPGAs, where flip-flops are plentiful, often faster. Yosys recognises FSMs and re-encodes them; `-nofsm` keeps the encoding as written:

```console
$ for opt in -nofsm ""; do echo -n "synth $opt: "; yosys -p "read_verilog fsm.v; synth -top detect_moore $opt; stat" | sed -n '/Printing statistics/,$p' | grep -E "Number of cells|DFF" | tail -2 | tr -s ' ' | tr '\n' ' '; echo; done
synth -nofsm:  Number of cells: 22  $_SDFF_PP0_ 3 
synth :  Number of cells: 18  $_DFF_P_ 5 
```

As written (binary, `-nofsm`), the machine uses 3 flip-flops; Yosys's FSM pass re-encodes it as one-hot with 5 flip-flops and *fewer* cells in total, because each state's next-state logic becomes a small OR of the states that lead to it.

### Pattern

**P-3. Write every FSM in three parts — a clocked state register, an `always @(*)` next-state `case` with a `default`, and output logic — and give the states names with `localparam`.** *(Practice)* The separation keeps the clocked part trivial and makes the combinational part easy to review against the state table; the `default` both prevents latches and recovers from an illegal state (after a radiation upset, for example). *Trade-off:* it is more lines than a single clocked block that assigns state and outputs together, and outputs written in a clocked block become registered — one cycle later — which may or may not be what you want.

### Mastery check

1. **Design.** Draw the Moore state table for detecting `101` with overlapping matches.
2. **Predict.** In one-hot encoding, how many flip-flops does `detect_moore` need? In binary?
3. **Explain.** Why does the Mealy detector need only four states?

## Unit 2.5 — Memory arrays: register files, RAM, and ROM

**One-line summary:** A memory is an array of words addressed by a number; a **register file** has several read ports and a write port and is read combinationally, while larger RAMs are usually read synchronously (the data appears after the clock edge), which is what FPGA block RAMs and SRAM macros provide.

A processor's register file — RISC-V has 32 registers of 32 bits — is read by two instructions operands every cycle and written by one result. In Verilog, a memory is an array of vectors:

```verilog
// file: memories.v
module regfile(input clk, input we, input [4:0] ra1, ra2, wa, input [31:0] wd,
               output [31:0] rd1, rd2);
    reg [31:0] r [0:31];
    integer i;
    initial for (i = 0; i < 32; i = i + 1) r[i] = 0;
    always @(posedge clk) if (we && wa != 0) r[wa] <= wd;     // register x0 stays 0
    assign rd1 = r[ra1];                                       // combinational reads
    assign rd2 = r[ra2];
endmodule

module sync_ram #(parameter AW = 10, DW = 32)(input clk, input we, input [AW-1:0] addr,
                                                input [DW-1:0] wd, output reg [DW-1:0] rd);
    reg [DW-1:0] mem [0:(1 << AW) - 1];
    always @(posedge clk) begin
        if (we) mem[addr] <= wd;
        rd <= mem[addr];                                       // data appears after the edge
    end
endmodule

module rom_squares(input [3:0] addr, output [7:0] data);
    reg [7:0] table_ [0:15];
    initial $readmemh("squares.hex", table_);                  // contents from a file
    assign data = table_[addr];
endmodule

`ifndef SYNTHESIS
module tb;
    reg clk = 0;
    always #5 clk = ~clk;
    reg we = 0;
    reg [4:0] ra1, ra2, wa;
    reg [31:0] wd;
    wire [31:0] rd1, rd2;
    regfile rf(clk, we, ra1, ra2, wa, wd, rd1, rd2);
    reg rwe = 0;
    reg [9:0] addr;
    reg [31:0] rwd;
    wire [31:0] rrd;
    sync_ram #(10, 32) ram(clk, rwe, addr, rwd, rrd);
    wire [7:0] sq;
    reg [3:0] n;
    rom_squares rom(n, sq);
    initial begin
        @(negedge clk) we = 1; wa = 5; wd = 32'hCAFE;
        @(negedge clk) wa = 0; wd = 32'hDEAD;                  // attempt to write x0
        @(negedge clk) we = 0; ra1 = 5; ra2 = 0;
        #1 $display("x5 = %h, x0 = %h (writes to x0 are ignored)", rd1, rd2);
        @(negedge clk) rwe = 1; addr = 3; rwd = 32'h1234;
        @(negedge clk) rwe = 0; addr = 3;
        #1 $display("sync RAM right after setting the address: %h (the previous read, of an unwritten address)", rrd);
        @(posedge clk) #1 $display("one clock edge later: %h", rrd);
        n = 12; #1 $display("ROM: 12 squared = %0d", sq);
        $finish;
    end
endmodule
`endif
```

```text
// file: squares.hex
00 01 04 09 10 19 24 31 40 51 64 79 90 a9 c4 e1
```

```console
$ iverilog -g2012 -o memories memories.v && vvp -n memories
x5 = 0000cafe, x0 = 00000000 (writes to x0 are ignored)
sync RAM right after setting the address: xxxxxxxx (the previous read, of an unwritten address)
one clock edge later: 00001234
ROM: 12 squared = 144
memories.v:54: $finish called at 57 (1s)
$ yosys -p "read_verilog memories.v; synth -top regfile; stat" | grep -E "Number of cells|DFF" | tail -2 | tr -s ' '
 Number of cells: 3093
 $_DFFE_PP_ 1024
```

The `squares.hex` file starts with a `// file:` line like every listing; `$readmemh` accepts `//` comments, so it is valid as it stands. Synthesising the register file with generic gates gives 1024 flip-flops and some two thousand gates for the two 32-way read multiplexers. All 32 registers became flip-flops: the `wa != 0` guard keeps `r[0]` at 0, but the tool did not prove it, so it kept 32 flip-flops that never change — writing `rd1 = (ra1 == 0) ? 0 : r[ra1]` and dropping `r[0]` would save them. Numbers like these are why real processors use custom register-file arrays, and why FPGAs provide dedicated RAM blocks that synthesis tools map arrays like `sync_ram` into.

### Deep & Tricky

**DT-10. A RAM written and read at the same address in the same clock cycle returns *either* the old or the new data, depending on how it is written in Verilog (or how the RAM macro is configured): "read-first" returns the old value, "write-first" the new one. A design that assumes one and is mapped to a memory configured for the other works in simulation and fails in hardware.** *(Xilinx UG901 "Vivado Design Suite User Guide: Synthesis", "RAM HDL Coding Techniques": read-first, write-first, and no-change modes and the coding style that infers each)* In `sync_ram` above, the non-blocking `rd <= mem[addr]` reads the old contents, so a write and a read of the same address in one cycle return the *old* value — read-first. Forwarding logic around the RAM, or a deliberate choice of mode, makes the behaviour explicit.

### Mastery check

1. **Explain.** Why do most processors make register x0 (or its equivalent) constant zero?
2. **Predict.** In `sync_ram`, if the testbench writes `5678` to address 3 and reads address 3 in the same cycle, what does `rrd` show after that edge?
3. **Calculate.** How many flip-flops does a 32 × 32-bit register file need if x0 is hard-wired to zero?

## Chapter 2 Project — A UART transmitter and receiver

### Requirements

A UART (universal asynchronous receiver-transmitter) sends bytes over one wire: the line idles at 1; a frame is a **start bit** (0), eight data bits least-significant first, and a **stop bit** (1), each lasting one *bit time*. Build a transmitter FSM that sends a byte when `start` is pulsed and raises `busy` meanwhile, and a receiver FSM that detects the start bit's falling edge, samples each bit in the **middle** of its bit time, and outputs the byte with a `valid` pulse (and a `frame_error` flag if the stop bit is 0). Parameterise the number of clock cycles per bit. Test the pair in loopback (transmitter wired to receiver) with every byte value 0–255.

### Reference solution

```verilog
// file: uart/uart.v
module uart_tx #(parameter CPB = 16)(input clk, input rst, input start, input [7:0] data,
                                     output reg tx, output busy);
    localparam IDLE = 2'd0, START = 2'd1, DATA = 2'd2, STOP = 2'd3;
    reg [1:0] state;
    reg [$clog2(CPB)-1:0] tick;
    reg [2:0] bitn;
    reg [7:0] shift;
    assign busy = (state != IDLE);
    always @(posedge clk) begin
        if (rst) begin
            state <= IDLE; tx <= 1; tick <= 0;
        end else case (state)
            IDLE: if (start) begin state <= START; shift <= data; tick <= 0; tx <= 0; end
            START: if (tick == CPB - 1) begin state <= DATA; tick <= 0; bitn <= 0; tx <= shift[0]; end
                   else tick <= tick + 1;
            DATA: if (tick == CPB - 1) begin
                      tick <= 0;
                      if (bitn == 7) begin state <= STOP; tx <= 1; end
                      else begin bitn <= bitn + 1; shift <= shift >> 1; tx <= shift[1]; end
                  end else tick <= tick + 1;
            STOP: if (tick == CPB - 1) begin state <= IDLE; tick <= 0; end
                  else tick <= tick + 1;
        endcase
    end
endmodule

module uart_rx #(parameter CPB = 16)(input clk, input rst, input rx,
                                     output reg [7:0] data, output reg valid, output reg frame_error);
    localparam IDLE = 2'd0, START = 2'd1, DATA = 2'd2, STOP = 2'd3;
    reg [1:0] state;
    reg [$clog2(CPB)-1:0] tick;
    reg [2:0] bitn;
    reg rx_s1, rx_s;                                       // two-flop synchroniser (Chapter 3)
    always @(posedge clk) begin rx_s1 <= rx; rx_s <= rx_s1; end
    always @(posedge clk) begin
        valid <= 0;
        if (rst) begin
            state <= IDLE; tick <= 0; frame_error <= 0;
        end else case (state)
            IDLE: if (!rx_s) begin state <= START; tick <= 0; end
            START: if (tick == CPB / 2 - 1) begin          // middle of the start bit
                       if (!rx_s) begin state <= DATA; tick <= 0; bitn <= 0; end
                       else state <= IDLE;                  // a glitch, not a start bit
                   end else tick <= tick + 1;
            DATA: if (tick == CPB - 1) begin                // middle of each data bit
                      tick <= 0;
                      data <= {rx_s, data[7:1]};            // LSB first: shift in from the top
                      if (bitn == 7) state <= STOP; else bitn <= bitn + 1;
                  end else tick <= tick + 1;
            STOP: if (tick == CPB - 1) begin
                      state <= IDLE; tick <= 0;
                      valid <= 1;
                      frame_error <= !rx_s;
                  end else tick <= tick + 1;
        endcase
    end
endmodule
```

```verilog
// file: uart/tb_uart.v
`timescale 1ns/1ps
module tb;
    localparam CPB = 16;
    reg clk = 0, rst = 1, start = 0;
    always #5 clk = ~clk;
    reg [7:0] data;
    wire line, busy, valid, ferr;
    wire [7:0] got;
    uart_tx #(CPB) tx(clk, rst, start, data, line, busy);
    uart_rx #(CPB) rx(clk, rst, line, got, valid, ferr);
    integer i, errors = 0, received = 0, frame_errors = 0, cycles = 0;
    always @(posedge clk) cycles = cycles + 1;
    always @(posedge clk) if (valid) begin
        received = received + 1;
        if (ferr) frame_errors = frame_errors + 1;
        if (got !== received - 1) begin
            errors = errors + 1;
            if (errors < 4) $display("byte %0d received as %0d", received - 1, got);
        end
    end
    initial begin
        $dumpfile("uart.vcd");
        $dumpvars(0, tb);
        repeat (3) @(posedge clk);
        rst = 0;
        for (i = 0; i < 256; i = i + 1) begin
            @(negedge clk) data = i; start = 1;
            @(negedge clk) start = 0;
            wait (!busy);
        end
        repeat (4 * CPB) @(posedge clk);
        $display("sent 256 bytes, received %0d, %0d wrong, %0d frame errors", received, errors, frame_errors);
        $display("one frame = 10 bits x %0d cycles = %0d cycles; whole test %0d cycles", CPB, 10 * CPB, cycles);
        $finish;
    end
endmodule
```

```console
$ cd uart && iverilog -g2012 -o uart uart.v tb_uart.v && vvp -n uart | grep -v VCD && ls -la uart.vcd | awk '{print "waveform file:", $5, "bytes"}'
sent 256 bytes, received 256, 0 wrong, 0 frame errors
one frame = 10 bits x 16 cycles = 160 cycles; whole test 41283 cycles
tb_uart.v:35: $finish called at 412825000 (1ps)
waveform file: 2588527 bytes
$ cd uart && yosys -p "read_verilog uart.v; synth -top uart_rx; stat" | grep -E "Number of cells" | tail -1 | tr -s ' '
 Number of cells: 118
```

Design notes: the receiver samples in the middle of each bit — half a bit time after the falling edge of the start bit, then every full bit time — which tolerates clock differences between sender and receiver of a few percent. The two flip-flops on `rx` (`rx_s1`, `rx_s`) are a **synchroniser**: the line comes from another device with its own clock, so it can change at any time, including right at our clock edge; Chapter 3 explains why one flip-flop is not enough. `$dumpvars` writes every signal to `uart.vcd`, which GTKWave displays as waveforms.

### Review checklist

- What happens if a glitch of a few cycles pulls the line low while the receiver is idle? (Trace the START state.)
- How far can the transmitter's clock differ from the receiver's before the eighth data bit is sampled in the wrong bit time?
- Is every register reset? Which ones need not be?

## Answers to the Chapter 2 prediction questions

**Unit 2.1, Q2.** Both Q and Qn become 0 (each NOR has a 1 input) — the forbidden state.

**Unit 2.2, Q2.** No. Rising edges are at 5, 15, 25 ns, …; a pulse from 16 to 24 ns contains no rising edge, so no flip-flop ever samples it and `q` never shows it.

**Unit 2.3, Q1.** All zeros forever: the XOR of zeros is zero, so the all-zero state is a fixed point — which is why an LFSR must be reset to a non-zero value.

**Unit 2.4, Q2.** One-hot needs 5 flip-flops (one per state); binary needs 3 (⌈log₂ 5⌉).

**Unit 2.5, Q2.** The old contents of address 3 (read-first): the read samples `mem[3]` before the non-blocking write updates it. One cycle later, a read of address 3 returns `5678`.

**Unit 2.5, Q3.** 31 × 32 = 992.

# Chapter 3 — Hardware Description, Timing, and Verification

*Goal: write Verilog that synthesises into the circuit you meant, find out how fast that circuit can be clocked, and convince yourself — by simulation, by random testing against a model, and by formal proof — that it is correct. A design that simulates correctly can still fail in silicon if it infers latches you did not intend, violates setup or hold time, or samples a signal from another clock domain; this chapter is about each of those failures.*

*Accompanies the lectures "Hardware Description Languages and Verilog" and "Timing and Verification".*

**What you will be able to do after this chapter**

- Separate the synthesisable subset of Verilog from simulation-only constructs, and recognise code that infers latches.
- Write self-checking testbenches with random stimulus, a reference model, assertions, and coverage counters; run them on both Icarus Verilog and Verilator.
- Compute the critical path and maximum clock frequency of a circuit from propagation delay, setup time, and clock-to-Q delay — and watch a too-fast clock produce wrong results in simulation.
- Explain metastability, compute a synchroniser's mean time between failures, and build a two-flip-flop synchroniser.
- Prove two circuits equivalent with a SAT solver in Yosys.

## Topic 3A — Describing hardware

## Unit 3.1 — Synthesisable Verilog, and the accidental latch

**One-line summary:** Verilog describes hardware, not a program: `assign` and `always @(*)` describe combinational logic, `always @(posedge clk)` describes flip-flops, and any combinational block that does not assign an output on every path describes a **latch** — usually a bug, which synthesis reports and simulation does not.

Verilog was designed for simulation first; only a subset can be turned into gates. The rules of thumb:

| Construct | Synthesisable? | Notes |
| --- | --- | --- |
| `module`, ports, `parameter`, `generate` | Yes | Structure and repetition at elaboration time |
| `assign`, `always @(*)` with `=` | Yes | Combinational logic |
| `always @(posedge clk)` with `<=` | Yes | Flip-flops |
| `for` loops with constant bounds | Yes | Unrolled into parallel hardware |
| `#delay`, `initial` (except for memory contents on FPGAs), `$display`, `wait`, `forever` | No | Simulation only — testbenches |
| `*`, `/`, `%` | `*` yes; `/` and `%` by constants or small widths | Large dividers are big and slow |

The most common synthesis bug is the unintended latch. A combinational block must give every output a value in every case; if some path leaves an output unassigned, the language says the output *keeps its old value* — and keeping a value requires memory, so the tool builds a latch:

```verilog
// file: latch_bug.v
module priority_bad(input [3:0] req, output reg [1:0] grant, output reg valid);
    always @(*) begin
        valid = 1;
        if      (req[3]) grant = 3;
        else if (req[2]) grant = 2;
        else if (req[1]) grant = 1;
        else if (req[0]) grant = 0;
        else valid = 0;                 // grant not assigned here: a latch
    end
endmodule

module priority_good(input [3:0] req, output reg [1:0] grant, output reg valid);
    always @(*) begin
        valid = 1;
        grant = 0;                      // default first: every path assigns grant
        if      (req[3]) grant = 3;
        else if (req[2]) grant = 2;
        else if (req[1]) grant = 1;
        else if (req[0]) grant = 0;
        else valid = 0;
    end
endmodule
```

```console
$ for top in priority_bad priority_good; do echo -n "$top: "; yosys -p "read_verilog latch_bug.v; synth -top $top" 2>&1 | grep -c "Latch inferred" || true; done
priority_bad: 1
priority_good: 0
$ yosys -p "read_verilog latch_bug.v; synth -top priority_bad" | grep -m1 -E "Latch inferred" | sed 's/^ *//'
Latch inferred for signal `\priority_bad.\grant' from process `\priority_bad.$proc$latch_bug.v:3$1': $auto$proc_dlatch.cc:427:proc_dlatch$145
$ verilator --lint-only -Wall latch_bug.v 2>&1 | grep -m1 -oE "%Warning-LATCH.{0,70}"
%Warning-LATCH: latch_bug.v:3:5: Latch inferred for signal 'priority_bad.grant' (not
```

Yosys and Verilator both report the latch; a simulator does not, because in simulation "keep the old value" is perfectly well defined. In hardware the latch is transparent whenever its enable is active, which creates timing loops and glitches, and most FPGA and ASIC flows treat an inferred latch as an error. Assigning defaults at the top of every `always @(*)` block is the standard cure (P-4 below).

### Pattern

**P-4. In every `always @(*)` block, assign a default value to every output first, then override it in the branches; in every `case`, include a `default`.** *(Practice; Verilator's `LATCH` and `CASEINCOMPLETE` warnings check it)* Defaults make "no latch" true by construction instead of by careful reading. *Trade-off:* a default can hide a missing case that should have been an error — use `x` as the default (`grant = 2'bx`) during development if you want simulation to expose unintended paths, because synthesis treats `x` as "don't care" and still builds no latch.

### Mastery check

1. **Predict.** Does `always @(*) if (en) y = d;` infer a latch? What does it describe?
2. **Explain.** Why does a `for` loop in synthesisable Verilog describe parallel hardware rather than repetition in time?
3. **Explain.** Why is `initial` allowed to set memory contents on FPGAs but not in ASIC designs?

## Unit 3.2 — Verification: testbenches, models, assertions, coverage

**One-line summary:** A good testbench drives the design with directed and random inputs, compares every output with an independent **reference model**, stops on the first mismatch with a useful message, and measures **coverage** — which interesting situations actually happened — because a test that never exercised a case proves nothing about it.

Verification takes more effort than design in real projects — often twice as much. The parts of a self-checking testbench:

- **Stimulus**: directed tests for the cases you thought of, random tests for those you did not.
- **Reference model**: the same function computed differently — in Verilog behavioural code, or in C or Python linked to the simulator.
- **Checker**: compares design and model every cycle; **assertions** check properties that must always hold. SystemVerilog has a dedicated statement for this, `assert (condition) else action;`, but this book's testbenches write their checks as plain `if` statements, for a reason DT-11 explains.
- **Coverage**: counters for interesting events (the FIFO became full; a carry rippled through all bits), reported at the end.

Here is a testbench for an 8-bit saturating adder — `a + b`, clamped to 255 — with all four parts. It is written in the SystemVerilog subset that both Icarus Verilog and Verilator accept, so the same file runs on both:

```verilog
// file: verify/satadd.v
module satadd(input [7:0] a, b, output [7:0] y, output sat);
    wire [8:0] s = a + b;
    assign sat = s[8];
    assign y = sat ? 8'hFF : s[7:0];
endmodule
```

```verilog
// file: verify/tb_satadd.sv
module tb;
    logic [7:0] a, b, y;
    logic sat;
    satadd dut(.a(a), .b(b), .y(y), .sat(sat));
    int errors = 0, cov_sat = 0, cov_exact255 = 0, cov_zero = 0;
    function automatic logic [7:0] model(input logic [7:0] x, z);   // reference: integers, min()
        int sum = int'(x) + int'(z);
        return (sum > 255) ? 8'd255 : 8'(sum);
    endfunction
    task automatic check();
        #1;
        if (y !== model(a, b)) begin                                   // the checker
            errors = errors + 1;
            if (errors < 4) $display("ERROR a=%0d b=%0d: y=%0d expected %0d", a, b, y, model(a, b));
        end
        if (sat !== (int'(a) + int'(b) > 255)) errors = errors + 1;
        if (sat) cov_sat++;
        if (!sat && y == 255) cov_exact255++;      // the boundary: exactly 255, no saturation
        if (y == 0) cov_zero++;
    endtask
    initial begin
        a = 0;   b = 0;   check();                  // directed corners
        a = 255; b = 255; check();
        a = 128; b = 127; check();
        a = 128; b = 128; check();
        repeat (20000) begin a = 8'($urandom); b = 8'($urandom); check(); end
        $display("errors %0d | coverage: saturated %0d, exactly 255 %0d, zero %0d",
                 errors, cov_sat, cov_exact255, cov_zero);
        $finish;
    end
endmodule
```

```console
$ cd verify && iverilog -g2012 -o sim_icarus satadd.v tb_satadd.sv && vvp -n sim_icarus | grep -v finish
errors 0 | coverage: saturated 9888, exactly 255 76, zero 1
$ cd verify && verilator --binary -j 2 -Wall -Wno-DECLFILENAME --top-module tb satadd.v tb_satadd.sv -o sim_verilator > /dev/null && ./obj_dir/sim_verilator | grep -v -i finish
errors 0 | coverage: saturated 9911, exactly 255 74, zero 1
```

The coverage line is the important one. The "exactly 255" case — the boundary where an off-by-one saturation bug (`s > 255` written as `s >= 255`) would show — happened only about 75 times in 20,000 random tests (for each sum, 1 pair in 256 adds up to exactly 255), and the "zero" case only once, because it was directed: random 8-bit pairs rarely sum to zero. Without the coverage counters you would not know how thin that evidence is. When coverage is low, add directed tests or *constrain* the random stimulus toward the interesting region.

**Two simulators.** Icarus Verilog interprets the design event by event; Verilator compiles it to C++ and is typically 10–100× faster on large designs, at the cost of supporting fewer simulation-only constructs (it is cycle-based and handles `#` delays only in its `--timing` mode). Running a testbench on both catches bugs that depend on one simulator's event ordering (Chapter 2's DT-9).

### Deep & Tricky

**DT-11. Simulators disagree, and simulators have bugs. Verilator ignores immediate `assert` statements unless it is run with `--assert`; in version 5.020 an `assert … else begin errors++; … end` then *executed the action block on every check* (the FIFO testbench of this chapter's project reported 115,698 "errors" against Icarus's 0), and with `--assert` the C++ that Verilator generated for the same block failed to compile. Running every testbench on two simulators, and starting each verification effort by checking that a deliberately broken design fails, catches tool problems as well as design problems.** *(Verilator 5.020 manual, "--assert": "Enable all assertions"; the behaviour described was observed while writing this chapter)* The checks in this book are plain `if` statements, which every simulator handles identically.

### Mastery check

1. **Explain.** Why must the reference model be written differently from the design?
2. **Predict.** If the design had the bug `assign sat = s > 255;` replaced by `s >= 255`, which coverage bucket would the failures fall into?
3. **Design.** Name three coverage events worth counting for a FIFO.

## Unit 3.3 — Formal equivalence: proving two circuits identical

**One-line summary:** A **miter** combines two circuits with the same inputs and outputs "1 when their outputs agree"; asking a SAT solver to find inputs that make it 0 either finds a counterexample or proves that none exists — equivalence over *all* inputs, which no amount of random simulation can show for wide designs.

Random simulation of a 32-bit adder tries a vanishing fraction of its 2⁶⁴ input pairs. **Formal verification** reasons about all of them symbolically. The simplest and most widely used form is **equivalence checking**: after a synthesis tool optimises a circuit, or an engineer rewrites it for speed, prove the new version computes the same function as the old. Yosys includes a SAT solver:

```verilog
// file: equiv.v
module add_ref(input [15:0] a, b, output [16:0] s);
    assign s = a + b;
endmodule

module add_ripple(input [15:0] a, b, output [16:0] s);
    wire [16:0] c;
    assign c[0] = 1'b0;
    genvar i;
    generate for (i = 0; i < 16; i = i + 1) begin : fa
        assign s[i] = a[i] ^ b[i] ^ c[i];
        assign c[i + 1] = (a[i] & b[i]) | (c[i] & (a[i] ^ b[i]));
    end endgenerate
    assign s[16] = c[16];
endmodule

module add_buggy(input [15:0] a, b, output [16:0] s);      // wrong carry in one rare case
    wire [16:0] c;
    assign c[0] = 1'b0;
    genvar i;
    generate for (i = 0; i < 16; i = i + 1) begin : fa
        assign s[i] = a[i] ^ b[i] ^ c[i];
        assign c[i + 1] = ((a[i] & b[i]) | (c[i] & (a[i] ^ b[i])))
                          & ~(i == 11 && a[i] && !b[i] && c[i] && a[3]);
    end endgenerate
    assign s[16] = c[16];
endmodule

module miter_ok(input [15:0] a, b, output ok);
    wire [16:0] s1, s2;
    add_ref r(a, b, s1);
    add_ripple d(a, b, s2);
    assign ok = (s1 == s2);
endmodule

module miter_bug(input [15:0] a, b, output ok);
    wire [16:0] s1, s2;
    add_ref r(a, b, s1);
    add_buggy d(a, b, s2);
    assign ok = (s1 == s2);
endmodule
```

```console
$ yosys -p "read_verilog equiv.v; prep -top miter_ok; flatten; sat -prove ok 1" | grep -E "SUCCESS|FAIL"
SAT proof finished - no model found: SUCCESS!
$ yosys -p "read_verilog equiv.v; prep -top miter_bug; flatten; sat -prove ok 1 -show-inputs" | grep -E "SUCCESS|FAIL|\\\\[ab] " | tr -s ' '
SAT proof finished - model found: FAIL!
 \a 7932 1efc 0001111011111100
 \b 10060 274c 0010011101001100
```

The first proof succeeds in a fraction of a second: for all 2³² input pairs, the ripple adder equals `a + b`. The second finds a counterexample — specific values of `a` and `b`, printed in decimal, hex, and binary — for a bug that fires only when bit 11 of `a` is 1, bit 11 of `b` is 0, a carry arrives at bit 11, and bit 3 of `a` is 1: about one input pair in sixteen, so random tests would find this one quickly — but change the condition to depend on eight more bits and random testing needs millions of tries, while the SAT solver's effort barely changes.

### Deep & Tricky

**DT-12. Formal tools prove exactly the property you wrote, over exactly the inputs you allowed, and nothing else: a miter that compares only some outputs, a property with an unintended `x` or an over-constrained assumption, or a design with uninitialised state can "pass" a proof that says nothing about the bug you care about. A proof that succeeds too easily deserves suspicion — check it by planting a bug and confirming the proof fails.** *(Clarke, Grumberg, Peled, "Model Checking", §1: correctness is relative to the specification; the same point is made in every formal tool's documentation, e.g. SymbiYosys "Reference" on assumptions)* The second command above is that check: the same proof setup, with a known bug, must fail. Formal verification of *sequential* circuits — properties that must hold over many cycles, such as "the FIFO never reports full and empty at once" — uses bounded model checking and induction (SymbiYosys); the principle is the same.

### Mastery check

1. **Explain.** What does it mean for `sat -prove ok 1` to report "no model found"?
2. **Predict.** If `miter_ok` compared only `s[15:0]` and ignored `s[16]`, which bug could slip through?

## Topic 3B — Timing

## Unit 3.4 — Delays, the critical path, and the maximum clock frequency

**One-line summary:** A synchronous circuit works if, in every clock cycle, the slowest path from one flip-flop through combinational logic to the next arrives a **setup time** before the next edge: `T_clk ≥ t_clk→Q + t_logic,max + t_setup`; the slowest such path is the **critical path**, and it sets the maximum clock frequency.

Every gate takes time to respond. Two numbers describe it: the **propagation delay** `t_pd` (the longest time from an input change until the output is stable) and the **contamination delay** `t_cd` (the shortest time until the output *starts* to change). A flip-flop adds its own: **clock-to-Q** delay `t_ccq`/`t_pcq` (from the edge until Q changes), **setup time** `t_setup` (D must be stable this long *before* the edge), and **hold time** `t_hold` (D must stay stable this long *after* it).

Two constraints follow for a path from register R1 through logic to register R2:

```
setup:  T_clk ≥ t_pcq + t_pd(logic, longest path) + t_setup       (data arrives in time)
hold:   t_ccq + t_cd(logic, shortest path) ≥ t_hold               (data does not arrive too early)
```

Violate setup, and R2 captures a value that is still changing; slow the clock down and the circuit works. Violate hold, and the *new* value races through and corrupts the value R2 is capturing — and no clock frequency fixes it, which is why hold violations are fixed by adding delay to short paths.

The simulation below gives every gate a 1 ns delay (`#1`), builds an 8-bit ripple-carry adder whose worst path is about 17 gates long, registers its inputs and outputs, and runs it at two clock periods:

```verilog
// file: timing/timing.v
`timescale 1ns/1ps
module fa(input a, b, cin, output s, cout);              // full adder: every gate takes 1 ns
    wire t1, t2, t3;
    xor #1 x1(t1, a, b);
    xor #1 x2(s, t1, cin);
    and #1 a1(t2, a, b);
    and #1 a2(t3, t1, cin);
    or  #1 o1(cout, t2, t3);
endmodule

module pipe_adder(input clk, input [7:0] a, b, output reg [8:0] sum);
    reg [7:0] ra, rb;
    wire [8:0] c;
    wire [7:0] s;
    assign c[0] = 1'b0;
    genvar i;
    generate for (i = 0; i < 8; i = i + 1) begin : bits
        fa f(ra[i], rb[i], c[i], s[i], c[i + 1]);
    end endgenerate
    always @(posedge clk) begin
        ra <= a; rb <= b;                       // launch registers
        sum <= {c[8], s};                       // capture register: one cycle later
    end
endmodule

module tb;
    reg clk = 0;
    real half;
    always #(half) clk = ~clk;
    reg [7:0] a, b;
    wire [8:0] sum;
    pipe_adder dut(clk, a, b, sum);
    reg [8:0] expected [0:1];
    integer i, errors;
    task run(input real period);
        begin
            half = period / 2; errors = 0;
            a = 0; b = 0;
            repeat (3) @(posedge clk);
            for (i = 0; i < 2000; i = i + 1) begin
                @(negedge clk);
                a = $random; b = $random;
                expected[i % 2] = a + b;
                @(posedge clk);                 // inputs captured here
                @(posedge clk); #0.01;          // result captured one edge later
                if (sum !== expected[i % 2]) errors = errors + 1;
            end
            $display("clock period %4.1f ns (%5.1f MHz): %0d of 2000 sums wrong", period, 1000.0 / period, errors);
        end
    endtask
    initial begin
        run(30.0);
        run(18.0);
        run(10.0);
        run(6.0);
        $finish;
    end
endmodule
```

```console
$ cd timing && iverilog -g2012 -o timing timing.v && vvp -n timing | grep -v finish
clock period 30.0 ns ( 33.3 MHz): 0 of 2000 sums wrong
clock period 18.0 ns ( 55.6 MHz): 0 of 2000 sums wrong
clock period 10.0 ns (100.0 MHz): 125 of 2000 sums wrong
clock period  6.0 ns (166.7 MHz): 800 of 2000 sums wrong
```

At a 30 ns period every sum is right. As the period shrinks below the worst-case path, more and more sums are wrong — not all of them, because most random additions do not ripple a carry through every bit: the critical path matters only for the inputs that exercise it. That is exactly why timing cannot be verified by functional simulation with random inputs: the failing inputs are rare. Real flows use **static timing analysis** (STA), which computes the worst path through the circuit from a delay model of every cell and wire, without simulating any input at all.

### Deep & Tricky

**DT-13. A functional (zero-delay) simulation says nothing about timing: the RTL simulation of a design passes at any clock frequency, because `#` delays are absent and every assignment completes "instantly". Timing is checked separately, by static timing analysis on the synthesised netlist, and the clock constraint you give the tool is part of the design.** *(IEEE 1364-2005 §11 — the simulation scheduling model has no notion of physical delay unless delays are specified; Synopsys/Cadence STA documentation; Harris & Harris, "Digital Design and Computer Architecture", §3.5)* The example above put explicit gate delays in the netlist to make timing visible in simulation; with plain `assign s = a + b;` the same testbench reports zero errors at any period. Gate-level simulation with delays extracted from a real layout (SDF back-annotation) exists, but it is slow and only as complete as the test vectors — STA is the tool of record.

### Mastery check

1. **Calculate.** `t_pcq` = 0.1 ns, `t_setup` = 0.05 ns, and the longest logic path is 1.35 ns. What is the maximum clock frequency?
2. **Explain.** Why can a hold violation not be fixed by slowing the clock?
3. **Predict.** In `timing.v`, if the adder were replaced by a Kogge–Stone adder with the same gate delays, at roughly what period would errors start?

## Unit 3.5 — Metastability and synchronisers

**One-line summary:** A flip-flop whose input changes inside its setup/hold window may enter a **metastable** state — neither 0 nor 1 — and take an unbounded time to resolve; signals from another clock domain must therefore pass through a **synchroniser** (usually two flip-flops in series), which reduces the failure probability to an acceptable mean time between failures.

Inside a flip-flop is a cross-coupled loop like Chapter 2's latch. If D changes right at the clock edge, the loop can be left balanced near the middle voltage, and it leaves that point only gradually, as noise and the loop's gain push it to one side. The probability that it is still unresolved after time `t` falls exponentially: `P(t) = (T₀ / T_c) · e^(−t/τ)`, where `τ` is the loop's time constant (tens of picoseconds in modern processes), `T₀` a technology constant, and `T_c` the clock period.

A **synchroniser** gives the first flip-flop almost a whole clock period to resolve before a second flip-flop samples it. The mean time between failures for a signal that changes `f_d` times per second, sampled at clock frequency `f_c`:

```
MTBF = e^(t_r / τ) / (T₀ · f_c · f_d)        with t_r ≈ T_c − t_setup  (time allowed to resolve)
```

```python
# file: mtbf.py
import math

tau, T0 = 20e-12, 150e-12            # 20 ps time constant, 150 ps window: illustrative values
fd = 10e6                            # the asynchronous input changes 10 million times a second
for fc_mhz in (100, 500, 1000, 2000):
    fc = fc_mhz * 1e6
    t_setup = 50e-12
    one = 0.0                        # one flip-flop: the logic after it uses the value at once
    two = 1 / fc - t_setup           # two flip-flops: almost a full cycle to resolve
    for name, tr in (("1 flop ", 0.2 / fc), ("2 flops", two)):
        mtbf = math.exp(tr / tau) / (T0 * fc * fd)
        print(f"{fc_mhz:5d} MHz {name}: MTBF = {mtbf:9.3g} s ({mtbf / 3.15e7:9.3g} years)")
```

```console
$ python3 mtbf.py
  100 MHz 1 flop : MTBF =  1.79e+38 s ( 5.69e+30 years)
  100 MHz 2 flops: MTBF = 7.68e+210 s (2.44e+203 years)
  500 MHz 1 flop : MTBF =       647 s ( 2.05e-05 years)
  500 MHz 2 flops: MTBF =  2.94e+36 s ( 9.34e+28 years)
 1000 MHz 1 flop : MTBF =    0.0147 s ( 4.66e-10 years)
 1000 MHz 2 flops: MTBF =  2.84e+14 s ( 9.01e+06 years)
 2000 MHz 1 flop : MTBF =  4.95e-05 s ( 1.57e-12 years)
 2000 MHz 2 flops: MTBF =  1.97e+03 s ( 6.25e-05 years)
```

The numbers span dozens of orders of magnitude, which is the point: one flip-flop, whose output feeds logic that leaves it only a fraction of the cycle to settle, fails every few microseconds to seconds; a second flip-flop that gives a full cycle pushes the MTBF beyond the age of the universe at moderate clock rates — and at 2 GHz, where a cycle is only 25 time constants, even two flip-flops fail every few days, which is why very fast designs use three. The UART receiver in Chapter 2 used two flip-flops on its input for this reason.

### Deep & Tricky

**DT-14. A two-flip-flop synchroniser is correct for a *single* bit. Synchronising a multi-bit value (a counter, an address) bit by bit is wrong: each bit may resolve differently at the same edge, so the receiving domain can see a value that never existed — half old bits, half new. Multi-bit values cross clock domains as Gray codes (only one bit changes per step), through handshakes, or through asynchronous FIFOs, which use exactly those Gray-coded pointers.** *(Cummings, "Clock Domain Crossing (CDC) Design & Verification Techniques Using SystemVerilog", SNUG 2008, §5; "Simulation and Synthesis Techniques for Asynchronous FIFO Design", SNUG 2002)* A binary counter going from 0111 to 1000 changes all four bits at once; sampled mid-change, it can read as any of the sixteen values. In Gray code the same step changes one bit, so the sample is either the old or the new value — both legitimate.

```python
# file: gray.py
def gray(n):
    return n ^ (n >> 1)

def gray_to_bin(g):
    n = 0
    while g:
        n ^= g
        g >>= 1
    return n

print("n  binary  gray   bits changed (binary vs gray)")
for n in range(8):
    b_change = bin(n ^ (n + 1) & 0xF).count("1")
    g_change = bin(gray(n) ^ gray((n + 1) % 16)).count("1")
    print(f"{n}  {n:04b}    {gray(n):04b}   {b_change} vs {g_change}")
assert all(gray_to_bin(gray(n)) == n for n in range(1 << 12))
print("gray_to_bin(gray(n)) == n for all 12-bit n")
```

```console
$ python3 gray.py
n  binary  gray   bits changed (binary vs gray)
0  0000    0000   1 vs 1
1  0001    0001   2 vs 1
2  0010    0011   1 vs 1
3  0011    0010   3 vs 1
4  0100    0110   1 vs 1
5  0101    0111   2 vs 1
6  0110    0101   1 vs 1
7  0111    0100   4 vs 1
gray_to_bin(gray(n)) == n for all 12-bit n
```

### Mastery check

1. **Calculate.** Using `mtbf.py`'s formula, how does MTBF change if `τ` doubles (a slower technology)?
2. **Explain.** Why is a Gray-coded pointer safe to synchronise bit by bit, while a binary pointer is not?

## Chapter 3 Project — A verified synchronous FIFO

### Requirements

Build a parameterised synchronous FIFO (first in, first out queue): `DEPTH` entries of `WIDTH` bits, a write port (`wr_en`, `wdata`), a read port (`rd_en`, `rdata`, valid in the same cycle as the data is at the head), and `full`, `empty`, and `count` outputs. Writes when full and reads when empty are ignored. Verify it with a randomised testbench that keeps a reference queue in the testbench, checks every read, asserts that `full` and `empty` are never both 1, and reports coverage: times full, times empty, simultaneous read and write while full, and while empty. Lint it with Verilator and synthesise it with Yosys.

### Reference solution

```verilog
// file: fifo/fifo.v
module fifo #(parameter WIDTH = 8, DEPTH = 8)(
    input clk, input rst,
    input wr_en, input [WIDTH-1:0] wdata,
    input rd_en, output [WIDTH-1:0] rdata,
    output full, output empty, output [$clog2(DEPTH):0] count);
    localparam AW = $clog2(DEPTH);
    reg [WIDTH-1:0] mem [0:DEPTH-1];
    reg [AW:0] wptr, rptr;                     // one extra bit: tells full from empty
    wire do_wr = wr_en && !full;
    wire do_rd = rd_en && !empty;
    always @(posedge clk) begin
        if (rst) begin
            wptr <= 0; rptr <= 0;
        end else begin
            if (do_wr) begin mem[wptr[AW-1:0]] <= wdata; wptr <= wptr + 1; end
            if (do_rd) rptr <= rptr + 1;
        end
    end
    assign rdata = mem[rptr[AW-1:0]];          // head of the queue, combinational
    assign empty = (wptr == rptr);
    assign full  = (wptr[AW-1:0] == rptr[AW-1:0]) && (wptr[AW] != rptr[AW]);
    assign count = wptr - rptr;
endmodule
```

```verilog
// file: fifo/tb_fifo.sv
module tb;
    localparam W = 8, D = 8;
    logic clk = 0, rst = 1, wr_en = 0, rd_en = 0;
    logic [W-1:0] wdata, rdata;
    logic full, empty;
    logic [$clog2(D):0] count;
    fifo #(W, D) dut(.*);
    always #5 clk = ~clk;
    logic [W-1:0] model [$];                       // reference: a SystemVerilog queue
    int errors = 0, reads = 0, cov_full = 0, cov_empty = 0, cov_rw_full = 0, cov_rw_empty = 0;
    int cycle = 0;
    logic phase;
    initial begin
        repeat (2) @(posedge clk);
        rst = 0;
        repeat (50000) begin
            @(negedge clk);
            // bias the traffic: phases that fill and phases that drain
            phase = (cycle / 200) % 2 == 1;
            wr_en = ($urandom % 100) < (phase ? 70 : 30);
            rd_en = ($urandom % 100) < (phase ? 30 : 70);
            wdata = 8'($urandom);
            cycle = cycle + 1;
            #1;
            if (full && empty) errors = errors + 1;                     // assertion: never both
            if (int'(count) != model.size()) begin
                errors = errors + 1;
                $display("ERROR count %0d model %0d", count, model.size());
            end
            if (full) cov_full = cov_full + 1;
            if (empty) cov_empty = cov_empty + 1;
            if (full && wr_en && rd_en) cov_rw_full = cov_rw_full + 1;
            if (empty && wr_en && rd_en) cov_rw_empty = cov_rw_empty + 1;
            if (rd_en && !empty) begin
                reads = reads + 1;
                if (rdata !== model[0]) begin
                    errors = errors + 1;
                    $display("ERROR read %h expected %h", rdata, model[0]);
                end
                void'(model.pop_front());
            end
            if (wr_en && !full) model.push_back(wdata);
        end
        $display("%0d reads checked, %0d errors", reads, errors);
        $display("coverage: full %0d, empty %0d, read+write while full %0d, while empty %0d",
                 cov_full, cov_empty, cov_rw_full, cov_rw_empty);
        $finish;
    end
endmodule
```

```console
$ cd fifo && iverilog -g2012 -o sim_icarus fifo.v tb_fifo.sv && vvp -n sim_icarus | grep -v finish
15980 reads checked, 0 errors
coverage: full 12985, empty 13208, read+write while full 2749, while empty 2753
$ cd fifo && verilator --binary -j 2 -Wall -Wno-DECLFILENAME -Wno-BLKSEQ --top-module tb fifo.v tb_fifo.sv -o sim > /dev/null && ./obj_dir/sim | grep -v -i finish
15706 reads checked, 0 errors
coverage: full 13216, empty 13369, read+write while full 2772, while empty 2774
$ cd fifo && verilator --lint-only -Wall fifo.v 2>&1 | grep -c "%Warning" | sed 's/$/ lint warnings in fifo.v/'
0 lint warnings in fifo.v
$ cd fifo && yosys -p "read_verilog fifo.v; synth -top fifo; stat" | sed -n '/Printing statistics/,$p' | grep -E "Number of cells|DFF" | tail -4 | tr -s ' '
 Number of cells: 190
 $_DFFE_PP_ 64
 $_SDFFE_PP0P_ 8
 $_SDFF_PP0_ 3
```

Design notes: the pointers have one bit more than the address needs. When the address bits are equal, the extra bit says whether the write pointer has lapped the read pointer (full) or not (empty) — no separate counter or "last operation was a write" flag is needed. The testbench's traffic alternates between phases that favour writing and phases that favour reading, every 200 cycles; with unbiased 50/50 traffic the FIFO would almost never fill, and the coverage counters would show it. The read data is combinational (a "first-word fall-through" FIFO); a FIFO built on a synchronous RAM would deliver data one cycle after `rd_en`, and the testbench would need to account for that.

### Review checklist

- Does the coverage report show every bucket hit at least a few hundred times? If one is zero, which stimulus change fixes it?
- What happens on a simultaneous read and write when the FIFO is full? When it is empty? Is that what the specification says?
- Which signals would need a synchroniser if the write side and the read side used different clocks? (Answer: the pointers — as Gray codes, DT-14.)

## Answers to the Chapter 3 prediction questions

**Unit 3.1, Q1.** Yes: when `en` is 0, `y` keeps its value — a D latch with enable `en`. That is the only correct way to describe a latch on purpose, and it should be done only on purpose.

**Unit 3.2, Q2.** Into "exactly 255": with `>=`, a sum of exactly 255 would set `sat` although no saturation happened (the output value is still right, but the flag is wrong), so the failures appear only in the cases the boundary counter counts.

**Unit 3.3, Q2.** Any bug that affects only the carry out (bit 16), such as a wrong final carry, would pass.

**Unit 3.4, Q1.** T_min = 0.1 + 1.35 + 0.05 = 1.5 ns, so f_max ≈ 667 MHz.

**Unit 3.4, Q3.** Its path is about 11 gates instead of about 17, so errors would start below a period of roughly 12–13 ns instead of about 18–19 ns (plus the register overheads, which are zero in this simulation).

# Chapter 4 — The Von Neumann Model, the ISA, and Assembly

*Goal: define the contract between software and hardware precisely enough to build hardware for it. This chapter describes the stored-program computer, then the RISC-V RV32I instruction set — registers, memory, instruction formats, and their binary encodings — and programs it in assembly: arithmetic, memory, branches, loops, and function calls with a stack. The project is an instruction-set simulator in Python that runs programs compiled by GCC; it becomes the reference model that the processors of the next chapters are checked against.*

*Accompanies the lectures "Von Neumann Model and Instruction Set Architecture" and "Assembly Programming".*

**What you will be able to do after this chapter**

- Explain the Von Neumann model: memory holding both instructions and data, and the fetch–decode–execute cycle.
- Read and write RV32I assembly, and encode and decode its six instruction formats by hand.
- Follow the RISC-V calling convention: argument registers, the return address, the stack, saved and temporary registers.
- Read compiler output for C code at `-O0` and `-O2`.
- Compare RISC-V with MIPS, LC-3, and x86, and explain the trade-offs behind each design choice.

**How this chapter's code is organised.** Programs are built for 32-bit RISC-V (`-march=rv32i -mabi=ilp32`) without the C library (`-nostdlib`), and run with `qemu-riscv32`, which emulates a RISC-V CPU running Linux. A program talks to the outside world through the `ecall` instruction with Linux system-call numbers: 64 is `write`, 93 is `exit`.

## Topic 4A — The stored-program computer

## Unit 4.1 — The Von Neumann model

**One-line summary:** A Von Neumann computer keeps instructions and data in the same memory, and a control unit repeatedly fetches the instruction at the address in the **program counter** (PC), decodes it, executes it, and updates the PC — sequential execution of a stored program, which every general-purpose processor still *appears* to do.

John von Neumann's 1945 report on the EDVAC described a machine with five parts that every computer since has had in some form:

| Part | Role | In RV32I |
| --- | --- | --- |
| **Memory** | Holds instructions *and* data, addressed by number | 2³² bytes, byte-addressed |
| **Processing unit** | Arithmetic and logic on words held in fast storage | ALU + 32 registers |
| **Control unit** | Fetches and decodes instructions, sequences execution | The PC and the decoder |
| **Input** / **Output** | Communication with the outside | Memory-mapped devices; here, `ecall` to the OS |

The **instruction cycle** is the model's heartbeat: *fetch* the instruction at `PC`; *decode* it; *fetch operands*; *execute*; *store the result*; set `PC` to the next instruction (`PC + 4`, or a branch target). Two properties define the model: the **stored program** (a program is data in memory, so programs can load, generate, or modify programs — compilers, loaders, and JIT compilers depend on it), and **sequential execution** (instructions take effect one at a time, in program order).

The second property is a promise about what software *observes*, not about what the hardware does. Chapters 6–9 build processors that overlap, reorder, and speculate on instructions — while keeping the promise that the result is as if they ran one by one. The alternative to the Von Neumann model, **dataflow**, executes an instruction as soon as its operands are ready, with no program counter at all; out-of-order processors (Chapter 7) are dataflow machines hidden behind a Von Neumann interface.

A whole computer in a few dozen lines shows the cycle. This toy machine has a 16-word memory, one accumulator, and four instructions; its program — instructions and data in the same memory — adds the numbers 5, 7, and 9:

```python
# file: toy.py
LOAD, ADD, STORE, HALT = 1, 2, 3, 0          # opcode in the high byte, address in the low byte

mem = [0] * 16
program = [(LOAD, 12), (ADD, 13), (ADD, 14), (STORE, 15), (HALT, 0)]
for i, (op, addr) in enumerate(program):
    mem[i] = op << 8 | addr                    # instructions are numbers in memory
mem[12:15] = [5, 7, 9]                         # data, in the same memory

pc, acc, cycles = 0, 0, 0
while True:
    instr = mem[pc]                            # fetch
    op, addr = instr >> 8, instr & 0xFF        # decode
    pc += 1
    cycles += 1
    if op == LOAD:    acc = mem[addr]          # execute
    elif op == ADD:   acc += mem[addr]
    elif op == STORE: mem[addr] = acc
    elif op == HALT:  break
    print(f"pc={pc - 1:2} instr={instr:#06x} acc={acc}")
print(f"halted after {cycles} instructions; mem[15] = {mem[15]}")
print("memory:", mem)
```

```console
$ python3 toy.py
pc= 0 instr=0x010c acc=5
pc= 1 instr=0x020d acc=12
pc= 2 instr=0x020e acc=21
pc= 3 instr=0x030f acc=21
halted after 5 instructions; mem[15] = 21
memory: [268, 525, 526, 783, 0, 0, 0, 0, 0, 0, 0, 0, 5, 7, 9, 21]
```

The memory dump at the end shows the point of the model: `0x10c` (268) and `5` sit in the same array; nothing but the PC distinguishes an instruction from a number.

### Mastery check

1. **Explain.** What would happen in `toy.py` if the program stored a value into address 2 before reaching it?
2. **Explain.** In what sense is an out-of-order processor still a Von Neumann machine?

## Topic 4B — RISC-V RV32I

## Unit 4.2 — Registers, memory, and the instruction formats

**One-line summary:** RV32I has 32 general-purpose 32-bit registers (`x0` is always zero), a byte-addressed little-endian memory accessed only by loads and stores, and fixed 32-bit instructions in six formats — R, I, S, B, U, J — that keep the register fields in the same bit positions so the decoder can read registers before it knows the instruction.

**Registers.** `x0`–`x31`, each 32 bits. `x0` reads as 0 and ignores writes, which gives many instructions for free: `mv rd, rs` is `addi rd, rs, 0`, `nop` is `addi x0, x0, 0`, `j label` is `jal x0, label`. The assembler accepts ABI names that describe each register's role in the calling convention (Unit 4.4): `zero`, `ra` (return address), `sp` (stack pointer), `a0`–`a7` (arguments and results), `t0`–`t6` (temporaries), `s0`–`s11` (saved).

**Memory.** Byte-addressed: every byte has its own address, and a 32-bit word occupies four consecutive addresses. RISC-V is **little-endian**: the least significant byte is stored at the lowest address. Loads and stores come in widths of 8 (`lb`, `lbu`, `sb`), 16 (`lh`, `lhu`, `sh`), and 32 bits (`lw`, `sw`); the `u` variants zero-extend, the others sign-extend.

**Formats.** Every instruction is 32 bits. The fields:

```
 31        25 24    20 19    15 14  12 11       7 6       0
| funct7     | rs2    | rs1    |funct3| rd        | opcode  |  R-type  (add, sub, and, sll, slt…)
| imm[11:0]           | rs1    |funct3| rd        | opcode  |  I-type  (addi, lw, jalr, …)
| imm[11:5]  | rs2    | rs1    |funct3| imm[4:0]  | opcode  |  S-type  (sw, sh, sb)
| imm[12|10:5]| rs2   | rs1    |funct3|imm[4:1|11]| opcode  |  B-type  (beq, bne, blt, …)
| imm[31:12]                           | rd        | opcode  |  U-type  (lui, auipc)
| imm[20|10:1|11|19:12]                | rd        | opcode  |  J-type  (jal)
```

`rs1`, `rs2`, and `rd` are always in the same places, and the sign bit of every immediate is always bit 31. The scrambled immediate bits of the B and J formats look strange, but they keep as many immediate bits as possible in the same positions across formats, which saves multiplexers in the decoder; branch and jump offsets are multiples of 2, so bit 0 is not stored.

Let us check the formats against the assembler. The program below assembles a few instructions and decodes them with a Python decoder written from the table above:

```asm
# file: formats.s
    .text
    .globl _start
_start:
    add  x5, x6, x7          # R
    addi x5, x6, -3          # I
    lw   x8, 12(x2)          # I (load)
    sw   x8, -4(x2)          # S
    beq  x5, x6, _start      # B (backwards: negative offset)
    lui  x9, 0x12345         # U
    jal  x1, _start          # J
```

```python
# file: decode.py
import subprocess, sys

def sext(v, bits):
    return v - (1 << bits) if v >> (bits - 1) & 1 else v

def decode(w):
    op, rd, f3, rs1, rs2, f7 = w & 0x7F, w >> 7 & 31, w >> 12 & 7, w >> 15 & 31, w >> 20 & 31, w >> 25
    if op == 0x33:
        return f"R  add-family f7={f7:#x} f3={f3} rd=x{rd} rs1=x{rs1} rs2=x{rs2}"
    if op in (0x13, 0x03, 0x67):
        return f"I  op={op:#04x} f3={f3} rd=x{rd} rs1=x{rs1} imm={sext(w >> 20, 12)}"
    if op == 0x23:
        imm = (w >> 25) << 5 | (w >> 7 & 31)
        return f"S  f3={f3} rs1=x{rs1} rs2=x{rs2} imm={sext(imm, 12)}"
    if op == 0x63:
        imm = (w >> 31) << 12 | (w >> 7 & 1) << 11 | (w >> 25 & 0x3F) << 5 | (w >> 8 & 0xF) << 1
        return f"B  f3={f3} rs1=x{rs1} rs2=x{rs2} offset={sext(imm, 13)}"
    if op in (0x37, 0x17):
        return f"U  rd=x{rd} imm={w >> 12:#x}"
    if op == 0x6F:
        imm = (w >> 31) << 20 | (w >> 12 & 0xFF) << 12 | (w >> 20 & 1) << 11 | (w >> 21 & 0x3FF) << 1
        return f"J  rd=x{rd} offset={sext(imm, 21)}"
    return "?"

dump = subprocess.run(["riscv64-linux-gnu-objdump", "-d", sys.argv[1]], capture_output=True, text=True).stdout
for line in dump.splitlines():
    parts = line.split("\t")
    if len(parts) >= 3 and parts[0].strip().endswith(":") and len(parts[1].strip()) == 8:
        word = int(parts[1], 16)
        print(f"{word:08x}  {parts[2]:<6} {parts[3] if len(parts) > 3 else '':<18} -> {decode(word)}")
```

```console
$ riscv64-linux-gnu-as -march=rv32i -mabi=ilp32 formats.s -o formats.o && python3 decode.py formats.o
007302b3  add    t0,t1,t2           -> R  add-family f7=0x0 f3=0 rd=x5 rs1=x6 rs2=x7
ffd30293  addi   t0,t1,-3           -> I  op=0x13 f3=0 rd=x5 rs1=x6 imm=-3
00c12403  lw     s0,12(sp)          -> I  op=0x03 f3=2 rd=x8 rs1=x2 imm=12
fe812e23  sw     s0,-4(sp)          -> S  f3=2 rs1=x2 rs2=x8 imm=-4
fe6288e3  beq    t0,t1,0 <_start>   -> B  f3=0 rs1=x5 rs2=x6 offset=-16
123454b7  lui    s1,0x12345         -> U  rd=x9 imm=0x12345
fe9ff0ef  jal    0 <_start>         -> J  rd=x1 offset=-24
```

The decoded fields match the assembler's view, including the negative offsets of the backward branch and jump (−16 and −24 bytes: four and six instructions back). The `lw x8, 12(x2)` line shows the only addressing mode RV32I has: *base register + 12-bit signed offset*. Every other addressing mode — absolute addresses, array indexing, PC-relative data — is built from it with `lui`, `auipc`, and `add`.

### Deep & Tricky

**DT-15. RISC-V has no "load a 32-bit constant" instruction: `li x9, 0x12345678` is a pseudo-instruction that the assembler expands to `lui` + `addi`, and because `addi` sign-extends its 12-bit immediate, the `lui` value must be *rounded up* by one whenever bit 11 of the constant is 1. Hand-written `lui`/`addi` pairs that forget this produce constants that are off by 4096.** *(RISC-V Unprivileged ISA Specification v20191213, §2.4: "LUI … places the 20-bit U-immediate into bits 31–12 … ADDI adds the sign-extended 12-bit immediate"; the assembler pseudo-instruction table in the RISC-V Assembly Programmer's Manual)*

```asm
# file: li.s
    .globl _start
_start:
    li   a0, 0x12345FFF       # bit 11 is 1: the assembler compensates
    lui  a1, 0x12345           # naive pair: lui of the top 20 bits...
    addi a1, a1, -1            # ...plus 0xFFF, which as a 12-bit signed value is -1
```

```console
$ riscv64-linux-gnu-as -march=rv32i -mabi=ilp32 li.s -o li.o && riscv64-linux-gnu-objdump -d -M no-aliases li.o | grep -E "^\s+[0-9a-f]+:" | cut -f3- | tr '\t' ' '
lui a0,0x12346
addi a0,a0,-1 # 12345fff <_start+0x12345fff>
lui a1,0x12345
addi a1,a1,-1 # 12344fff <_start+0x12344fff>
$ python3 -c "print(hex(0x12345 << 12 + 0) , hex((0x12345 << 12) - 1), hex((0x12346 << 12) - 1))"
0x12345000 0x12344fff 0x12345fff
```

The assembler emitted `lui a0, 0x12346` and `addi a0, a0, -1`: 0x12346000 − 1 = 0x12345FFF. The naive pair computes 0x12345000 − 1 = 0x12344FFF, off by 4096.

### Mastery check

1. **Encode.** Write the 32-bit encoding of `addi x10, x0, 42` (opcode `0010011`, funct3 `000`).
2. **Predict.** After `li t0, 0x80` and `sb t0, 0(sp)`, what does `lb t1, 0(sp)` load? And `lbu t2, 0(sp)`?
3. **Explain.** Why does keeping `rs1` and `rs2` in fixed bit positions help the hardware?

## Unit 4.3 — Assembly programming: arithmetic, branches, loops, memory

**One-line summary:** Assembly programs build every control structure from conditional branches and jumps, every data structure from base-plus-offset loads and stores, and every larger value from 32-bit registers — so an `if` is a branch around code, a loop is a backward branch, and an array access is an address calculation.

A program that sums an array and prints the result in decimal needs all of it: a loop over memory, a division-free conversion to decimal digits (RV32I has no divide instruction; the `M` extension adds one), and system calls. Here it is, with the C it corresponds to in the comments:

```asm
# file: sumprint.s
    .data
arr:    .word 3, 1, 4, 1, 5, 9, 2, 6, 5, 3     # int arr[10]
n:      .word 10
buf:    .space 12                              # char buf[12]

    .text
    .globl _start
_start:
    .option push
    .option norelax
    lla  gp, __global_pointer$   # set up gp: the linker may address data relative to it
    .option pop
    lla  t0, arr                 # t0 = &arr[0]  (lla: PC-relative address)
    lw   t1, n                   # t1 = n        (lw with a symbol: auipc + lw)
    li   a0, 0                   # sum = 0
loop:                            # while (n > 0)
    beqz t1, done
    lw   t2, 0(t0)               #   sum += *p
    add  a0, a0, t2
    addi t0, t0, 4               #   p++   (4 bytes per word)
    addi t1, t1, -1              #   n--
    j    loop
done:
    # convert a0 to decimal, right to left, by repeated subtraction of powers of ten
    lla  t0, buf + 11
    li   t1, 10
    sb   t1, 0(t0)               # buf[11] = '\n'
digits:
    addi t0, t0, -1
    li   t2, 0                   # quotient
div10:                           # while (a0 >= 10) { a0 -= 10; q++; }
    blt  a0, t1, divdone
    addi a0, a0, -10
    addi t2, t2, 1
    j    div10
divdone:
    addi a0, a0, '0'             # remainder -> ASCII digit
    sb   a0, 0(t0)
    mv   a0, t2                  # continue with the quotient
    bnez a0, digits
    # write(1, t0, buf + 12 - t0)
    lla  a2, buf + 12
    sub  a2, a2, t0
    mv   a1, t0
    li   a0, 1
    li   a7, 64
    ecall
    li   a0, 0                   # exit(0)
    li   a7, 93
    ecall
```

```console
$ riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -nostdlib -static sumprint.s -o sumprint && qemu-riscv32 ./sumprint; echo "exit status $?"
39
exit status 0
```

The sum is 39. Some things to notice:

- **The first instructions set up `gp`.** The linker *relaxes* accesses to small global variables — `lw t1, n` was assembled as `auipc` + `lw`, and the linker rewrote it into one `lw` relative to the **global pointer** register `gp`, which it assumes points into the data section. Normally the C runtime's start-up code sets `gp`; a program with its own `_start` must do it, with relaxation turned off for that one instruction (otherwise the linker would "relax" the setup of `gp` relative to `gp`). The first version of this program omitted those lines and crashed with a segmentation fault at the `lw`.
- **Conditions are branches.** `beqz t1, done` is `beq t1, x0, done`. RISC-V has `beq`, `bne`, `blt`, `bge`, `bltu`, `bgeu`; the others (`bgt`, `ble`) are the same instructions with operands swapped.
- **Arrays are address arithmetic.** `p++` on an `int *` adds 4 to the address, because each word is 4 bytes.
- **Symbols are addresses.** `lla t0, arr` (load local address) expands to `auipc` + `addi`: an address computed relative to the PC, so the program works wherever it is loaded. The more common `la` means "load address" *through the global offset table* when the toolchain builds position-independent executables, as this one does by default — and the linker rejected `la t0, buf + 11` with "dangerous relocation: The addend isn't allowed for R_RISCV_GOT_HI20" when this example was first written with it. `lla` always computes the address directly.

### Mastery check

1. **Write.** Assembly that finds the maximum of the array instead of the sum.
2. **Explain.** Why does the decimal conversion produce digits right to left?
3. **Predict.** If `addi t0, t0, 4` were `addi t0, t0, 1`, what would the program sum?

## Unit 4.4 — Functions, the stack, and the calling convention

**One-line summary:** A function call is `jal ra, f` (jump and save the return address in `ra`) and a return is `jalr x0, 0(ra)`; because there is only one `ra` and a fixed set of registers, functions agree on a **calling convention** — arguments in `a0`–`a7`, result in `a0`, `s` registers preserved by the callee, `t` and `a` registers free to be overwritten — and save what they must on the **stack**.

A function that calls another function overwrites `ra`, so it must save its own return address first; a function that needs a value to survive a call must keep it in an `s` register (and save the caller's value of that register) or on the stack. The stack grows downward from high addresses; `sp` points to its lowest used word and must stay 16-byte aligned.

Recursive factorial shows all of it:

```asm
# file: fact.s
    .text
    .globl _start
# unsigned fact(unsigned n) { return n <= 1 ? 1 : n * fact(n - 1); }
# RV32I has no multiply: mul(a, b) below is shift-and-add.
fact:
    li   t0, 1
    bgtu a0, t0, recurse
    li   a0, 1
    ret                          # = jalr x0, 0(ra)
recurse:
    addi sp, sp, -16             # stack frame: 16 bytes
    sw   ra, 12(sp)              # save the return address
    sw   s0, 8(sp)               # save the caller's s0
    mv   s0, a0                  # keep n across the call
    addi a0, a0, -1
    call fact                    # = jal ra, fact   (a0 = fact(n - 1))
    mv   a1, s0
    call mul                     # a0 = fact(n - 1) * n
    lw   s0, 8(sp)
    lw   ra, 12(sp)
    addi sp, sp, 16
    ret

mul:                             # a0 = a0 * a1, leaf function: no frame needed
    li   t0, 0
mul_loop:
    beqz a1, mul_done
    andi t1, a1, 1
    beqz t1, mul_skip
    add  t0, t0, a0
mul_skip:
    slli a0, a0, 1
    srli a1, a1, 1
    j    mul_loop
mul_done:
    mv   a0, t0
    ret

_start:
    li   a0, 10
    call fact                    # fact(10) = 3628800
    li   t0, 3628800
    sub  a0, a0, t0              # exit status 0 if correct
    li   a7, 93
    ecall
```

```console
$ riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -nostdlib -static fact.s -o fact && qemu-riscv32 ./fact; echo "exit status $? (0 = fact(10) was 3628800)"
exit status 0 (0 = fact(10) was 3628800)
```

`mul` is a **leaf** function — it calls nothing — so it uses only temporaries and needs no stack frame. `fact` saves `ra` and `s0` because it calls other functions and needs `n` afterwards. Each recursive level has its own 16-byte frame; at the deepest point, ten frames are on the stack.

### Deep & Tricky

**DT-16. The calling convention is not part of the ISA: the hardware treats `x1` and `x8` like any other register, and only the ABI document makes `x1` the return address and `x8`–`x9`, `x18`–`x27` callee-saved. Code that breaks the convention runs perfectly — until it is linked with code that follows it, at which point a callee-saved register silently changes across a call.** *(RISC-V ELF psABI Specification, "Integer Register Convention"; the ISA manual itself only recommends x1 and x5 as link registers, §2.5)* Compilers generate both sides of every call they compile, so violations come from hand-written assembly — a common source of bugs in interrupt handlers, bootloaders, and JIT compilers.

### Mastery check

1. **Explain.** Why does `fact` save `s0` before using it, while `mul` uses `t0` and `t1` without saving them?
2. **Predict.** What happens if `fact` forgets `sw ra, 12(sp)` and the matching `lw`?
3. **Calculate.** How many bytes of stack does `fact(10)` use at its deepest point?

## Unit 4.5 — From C to assembly

**One-line summary:** A compiler maps C variables to registers or stack slots, expressions to instruction sequences, and control flow to branches; at `-O0` every variable lives in memory and each statement is compiled alone, while at `-O2` values stay in registers, loops are restructured, and the assembly may no longer follow the source line by line.

```c
// file: strlen_c.c
unsigned my_strlen(const char *s) {
    unsigned n = 0;
    while (s[n] != '\0')
        n++;
    return n;
}
```

```console
$ for opt in -O0 -O2 "-O2 -ffreestanding"; do echo "== $opt"; riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 $opt -c strlen_c.c -o s.o && riscv64-linux-gnu-objdump -dr --no-show-raw-insn s.o | grep -E "^\s+[0-9a-f]+:" | cut -f2- | tr '\t' ' ' | grep -vE "RELAX|R_RISCV_(BRANCH|JAL|RVC)"; done
== -O0
addi sp,sp,-48
sw s0,44(sp)
addi s0,sp,48
sw a0,-36(s0)
sw zero,-20(s0)
j 24 <.L2>
lw a5,-20(s0)
addi a5,a5,1
sw a5,-20(s0)
lw a4,-36(s0)
lw a5,-20(s0)
add a5,a4,a5
lbu a5,0(a5)
bnez a5,18 <.L3>
lw a5,-20(s0)
mv a0,a5
lw s0,44(sp)
addi sp,sp,48
ret
== -O2
lbu a5,0(a0)
beqz a5,2c <.L3>
addi sp,sp,-16
addi a0,a0,1
sw ra,12(sp)
auipc ra,0x0
  14: R_RISCV_CALL_PLT strlen
jalr ra # 14 <my_strlen+0x14>
lw ra,12(sp)
addi a0,a0,1
addi sp,sp,16
ret
li a0,0
ret
== -O2 -ffreestanding
lbu a5,0(a0)
mv a4,a0
li a0,0
beqz a5,24 <.L4>
addi a0,a0,1
add a5,a4,a0
lbu a5,0(a5)
bnez a5,10 <.L3>
ret
ret
```

At `-O0`, `n` and `s` live on the stack: every use loads them, every update stores them back — eight memory accesses per character. At plain `-O2`, the loop is gone: GCC recognised that it computes the length of a string and replaced it with a **call to the library's `strlen`** (the relocation `R_RISCV_CALL_PLT strlen` shows the call target the linker must fill in), plus 1 because the loop is entered after the first character has been checked. With `-ffreestanding` — telling the compiler there may be no C library — it compiles the loop as written: four instructions per character, with `n` in a register. Reading `-O0` output teaches how C maps to the machine; reading `-O2` output teaches what the machine actually runs, which may be someone else's code.

### Deep & Tricky

**DT-17. An optimising compiler may replace your loops with calls to library functions it recognises — `memset`, `memcpy`, `strlen` — even when your source never mentions them, and even with `-fno-builtin`; in freestanding code (kernels, bootloaders, firmware, this book's test programs) that turns into a link error or, worse, a call to a different implementation than you think.** *(GCC manual, "-ffreestanding": "a freestanding environment … implies -fno-builtin"; "-ftree-loop-distribute-patterns": "perform loop distribution of patterns that can be code generated with calls to a library", enabled at -O2; GCC's documentation of `-nostdlib` notes that "GCC requires the freestanding environment provide memcpy, memmove, memset and memcmp")* That is why the project's runtime below defines `memset` and `memcpy` even though no test calls them: with `-O2`, GCC turns the zero-initialisation of a local array into a `memset` call.

### Mastery check

1. **Explain.** Why does `-O0` code store every variable to the stack after each change?
2. **Predict.** If `n` were declared `volatile`, how would the `-O2` loop change?

## Unit 4.6 — ISA design: RISC-V, MIPS, LC-3, and x86

**One-line summary:** ISAs differ in instruction length (fixed or variable), operand model (load/store or memory operands), number of registers, condition codes versus compare-and-branch, and how much they expose of the microarchitecture (MIPS's branch delay slot) — and each choice trades simplicity of hardware against code density, compatibility, and performance.

| | RISC-V RV32I | MIPS32 | LC-3 | x86-64 |
| --- | --- | --- | --- | --- |
| Instruction length | 32 bits (16 with C extension) | 32 bits | 16 bits | 1–15 bytes |
| Registers | 32 × 32 bits, x0 = 0 | 32 × 32 bits, $0 = 0 | 8 × 16 bits | 16 × 64 bits |
| Memory operands | loads and stores only | loads and stores only | loads and stores only (plus indirect) | most instructions |
| Branches | compare two registers and branch | compare and branch (`beq`, `bne`) + `slt` | condition codes N, Z, P set by results | condition flags set by arithmetic |
| Exposes the pipeline? | No | Yes: the branch delay slot | No | No |
| Addressing modes | base + offset | base + offset | PC-relative, base + offset, indirect | many (base + index × scale + offset, …) |

- **MIPS** (1985), RISC-V's closest ancestor, executes the instruction *after* a branch whether or not the branch is taken — the **delay slot** — because its five-stage pipeline (Chapter 6) had already fetched it. It made the first implementation simpler and every later, deeper or out-of-order implementation more complicated: the slot's behaviour is part of the ISA forever. RISC-V deliberately exposes no microarchitectural detail.
- **LC-3** is the teaching ISA of Patt and Patel's *Introduction to Computing Systems*: 16-bit words, 8 registers, 15 instructions, and **condition codes** — every instruction that writes a register sets N, Z, or P, and branches test them. Condition codes create a hidden dependence between consecutive instructions, which out-of-order processors must track; RISC-V avoids them by comparing registers in the branch itself.
- **x86** began in 1978 with variable-length instructions designed for memory-starved machines; compatibility has kept them. Modern x86 processors translate them internally into RISC-like micro-operations — the ISA as a compatibility layer above a different internal machine.

### Deep & Tricky

**DT-18. Fixed-length instructions make decoding easy but cost code size: the same program compiled for RV32I is typically 20–30% larger than with the 16-bit compressed extension (RV32IC), and often larger than x86 — which matters for instruction caches and embedded flash, and is why almost every RISC-V core implements the C extension.** *(RISC-V Unprivileged ISA Specification, Ch. 16 "C Standard Extension": "typically, 50%–60% of the RISC-V instructions in a program can be replaced with RVC instructions, resulting in a 25%–30% code-size reduction")* The measurement below compiles this chapter's C example for both.

```console
$ for march in rv32i rv32ic; do riscv64-linux-gnu-gcc -march=$march -mabi=ilp32 -O2 -c strlen_c.c -o $march.o && echo "$march: $(riscv64-linux-gnu-size $march.o | awk 'NR==2{print $1}') bytes of code"; done
rv32i: 100 bytes of code
rv32ic: 80 bytes of code
```

### Mastery check

1. **Explain.** Why did MIPS's designers include a delay slot, and why was it a burden later?
2. **Compare.** How does LC-3 implement "branch if the last result was zero", and how does RISC-V?

## Chapter 4 Project — `rvsim`: an RV32I instruction-set simulator

### Requirements

Write an instruction-set simulator (ISS) in Python for RV32I: load a 32-bit RISC-V ELF executable, initialise the stack pointer, and execute instructions until `exit` (system call 93), supporting `write` (system call 64) to standard output. Implement all 37 RV32I user-level instructions (the base set without `fence`'s ordering effects). Count instructions executed, and optionally print a trace of PC and instruction. Check it against QEMU on C programs compiled with GCC: the outputs and exit statuses must match. This simulator is the **golden model** that Chapters 5, 6, and 15 compare their Verilog processors with.

### Reference solution

```python
# file: rvsim/rvsim.py
"""rvsim: an RV32I instruction-set simulator that runs static ELF32 programs (Linux write/exit ecalls)."""
import struct
import sys

MASK = 0xFFFFFFFF

def sext(v, bits):
    v &= (1 << bits) - 1
    return v - (1 << bits) if v >> (bits - 1) else v

class CPU:
    def __init__(self, mem_size=1 << 24):
        self.x = [0] * 32
        self.pc = 0
        self.mem = bytearray(mem_size)
        self.count = 0
        self.out = bytearray()
        self.exit_code = None

    def load_elf(self, path):
        data = open(path, "rb").read()
        assert data[:4] == b"\x7fELF" and data[4] == 1, "need a 32-bit ELF"
        entry, phoff = struct.unpack_from("<II", data, 0x18)
        phentsize, phnum = struct.unpack_from("<HH", data, 0x2A)
        for i in range(phnum):
            p_type, off, vaddr, _, filesz, memsz = struct.unpack_from("<IIIIII", data, phoff + i * phentsize)
            if p_type == 1:                                          # PT_LOAD
                self.mem[vaddr:vaddr + filesz] = data[off:off + filesz]
                self.mem[vaddr + filesz:vaddr + memsz] = bytes(memsz - filesz)
        self.pc = entry
        self.x[2] = len(self.mem) - 16                               # sp at the top of memory

    def ld(self, addr, size, signed):
        v = int.from_bytes(self.mem[addr:addr + size], "little")
        return sext(v, size * 8) & MASK if signed else v

    def st(self, addr, size, v):
        self.mem[addr:addr + size] = (v & ((1 << size * 8) - 1)).to_bytes(size, "little")

    def step(self):
        w = self.ld(self.pc, 4, False)
        op, rd, f3 = w & 0x7F, w >> 7 & 31, w >> 12 & 7
        a, b = self.x[w >> 15 & 31], self.x[w >> 20 & 31]
        f7 = w >> 25
        immI = sext(w >> 20, 12)
        next_pc, val = self.pc + 4, None
        if op == 0x37:   val = w & 0xFFFFF000                                      # LUI
        elif op == 0x17: val = (self.pc + (w & 0xFFFFF000)) & MASK                 # AUIPC
        elif op == 0x6F:                                                           # JAL
            imm = (w >> 31) << 20 | (w >> 12 & 0xFF) << 12 | (w >> 20 & 1) << 11 | (w >> 21 & 0x3FF) << 1
            val, next_pc = next_pc, (self.pc + sext(imm, 21)) & MASK
        elif op == 0x67:                                                           # JALR
            val, next_pc = next_pc, (a + immI) & MASK & ~1
        elif op == 0x63:                                                           # branches
            imm = (w >> 31) << 12 | (w >> 7 & 1) << 11 | (w >> 25 & 0x3F) << 5 | (w >> 8 & 0xF) << 1
            sa, sb = sext(a, 32), sext(b, 32)
            taken = {0: a == b, 1: a != b, 4: sa < sb, 5: sa >= sb, 6: a < b, 7: a >= b}[f3]
            if taken:
                next_pc = (self.pc + sext(imm, 13)) & MASK
        elif op == 0x03:                                                           # loads
            addr = (a + immI) & MASK
            val = {0: (1, True), 1: (2, True), 2: (4, False), 4: (1, False), 5: (2, False)}[f3]
            val = self.ld(addr, val[0], val[1])
        elif op == 0x23:                                                           # stores
            imm = sext((w >> 25) << 5 | (w >> 7 & 31), 12)
            self.st((a + imm) & MASK, 1 << f3, b)
        elif op in (0x13, 0x33):                                                   # ALU
            imm_mode = op == 0x13
            y = immI & MASK if imm_mode else b
            sh = y & 31
            if f3 == 0:   val = a - y if (not imm_mode and f7 == 0x20) else a + y
            elif f3 == 1: val = a << sh
            elif f3 == 2: val = int(sext(a, 32) < sext(y, 32))
            elif f3 == 3: val = int(a < y)
            elif f3 == 4: val = a ^ y
            elif f3 == 5: val = (sext(a, 32) >> sh) if (f7 & 0x20) else a >> sh
            elif f3 == 6: val = a | y
            else:         val = a & y
            val &= MASK
        elif op == 0x73:                                                           # ECALL
            self.syscall()
        elif op == 0x0F:                                                           # FENCE: no-op here
            pass
        else:
            raise RuntimeError(f"illegal instruction {w:08x} at {self.pc:08x}")
        if val is not None and rd:
            self.x[rd] = val
        self.pc = next_pc
        self.count += 1

    def syscall(self):
        n, a0, a1, a2 = self.x[17], self.x[10], self.x[11], self.x[12]
        if n == 64:                                          # write(fd, buf, len)
            self.out += self.mem[a1:a1 + a2]
            self.x[10] = a2
        elif n == 93:                                        # exit(code)
            self.exit_code = a0 & 0xFF
        else:
            raise RuntimeError(f"unsupported syscall {n}")

    def run(self, limit=50_000_000, trace=False):
        while self.exit_code is None and self.count < limit:
            if trace:
                print(f"{self.count:6d} pc={self.pc:08x} instr={self.ld(self.pc, 4, False):08x}", file=sys.stderr)
            self.step()
        return self.exit_code

if __name__ == "__main__":
    cpu = CPU()
    cpu.load_elf(sys.argv[1])
    code = cpu.run(trace="--trace" in sys.argv)
    sys.stdout.write(cpu.out.decode(errors="replace"))
    print(f"[rvsim: {cpu.count} instructions, exit {code}]", file=sys.stderr)
    sys.exit(code)
```

The test programs are ordinary C, compiled freestanding: a tiny runtime provides `_start` (which sets up `gp`, as Unit 4.3 explained, calls `main`, and passes its result to `exit`), `write`, `exit`, and a `putint` helper, and the compiler supplies `memset`/`memcpy` calls that the runtime also defines.

```c
// file: rvsim/rt.h
#ifndef RT_H
#define RT_H
void rt_write(const char *s, unsigned n);
void rt_exit(int code) __attribute__((noreturn));
void puts_(const char *s);
void putint(int v);
#endif
```

```c
// file: rvsim/rt.c
#include "rt.h"

int main(void);

void rt_write(const char *s, unsigned n) {
    register long a0 __asm__("a0") = 1, a1 __asm__("a1") = (long)s, a2 __asm__("a2") = n;
    register long a7 __asm__("a7") = 64;
    __asm__ volatile("ecall" : "+r"(a0) : "r"(a1), "r"(a2), "r"(a7) : "memory");
}

void rt_exit(int code) {
    register long a0 __asm__("a0") = code, a7 __asm__("a7") = 93;
    __asm__ volatile("ecall" : : "r"(a0), "r"(a7));
    for (;;) {}
}

void puts_(const char *s) {
    unsigned n = 0;
    while (s[n]) n++;
    rt_write(s, n);
}

void putint(int v) {                              // no division in RV32I: subtract powers of ten
    static const int pow10[] = {1000000000, 100000000, 10000000, 1000000, 100000, 10000, 1000, 100, 10, 1};
    char buf[12];
    unsigned n = 0, u = v < 0 ? -(unsigned)v : (unsigned)v;
    int started = 0;
    if (v < 0) buf[n++] = '-';
    for (int i = 0; i < 10; i++) {
        char d = '0';
        while (u >= (unsigned)pow10[i]) { u -= pow10[i]; d++; }
        if (d != '0' || started || i == 9) { buf[n++] = d; started = 1; }
    }
    rt_write(buf, n);
}

void *memset(void *p, int c, unsigned n) { char *q = p; while (n--) *q++ = (char)c; return p; }
void *memcpy(void *d, const void *s, unsigned n) { char *a = d; const char *b = s; while (n--) *a++ = *b++; return d; }

__asm__(".section .text._start\n"            // the entry point, in assembly: set gp, call main, exit
        ".globl _start\n"
        "_start:\n"
        ".option push\n.option norelax\n"
        "lla gp, __global_pointer$\n"
        ".option pop\n"
        "call main\n"
        "tail rt_exit\n");
```

```c
// file: rvsim/tests.c
#include "rt.h"

static unsigned mul(unsigned a, unsigned b) { unsigned r = 0; while (b) { if (b & 1) r += a; a <<= 1; b >>= 1; } return r; }

static int fib(int n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }

static void bubble(int *a, int n) {
    for (int i = 0; i < n; i++)
        for (int j = 0; j + 1 < n - i; j++)
            if (a[j] > a[j + 1]) { int t = a[j]; a[j] = a[j + 1]; a[j + 1] = t; }
}

static unsigned crc32(const unsigned char *p, unsigned n) {
    unsigned c = 0xFFFFFFFFu;
    while (n--) {
        c ^= *p++;
        for (int k = 0; k < 8; k++) c = (c >> 1) ^ (0xEDB88320u & -(c & 1));
    }
    return ~c;
}

int main(void) {
    puts_("fib(20) = "); putint(fib(20)); puts_("\n");
    int a[12] = {5, -3, 9, 0, 12, -7, 3, 3, 100, -50, 8, 1};
    bubble(a, 12);
    puts_("sorted:");
    for (int i = 0; i < 12; i++) { puts_(" "); putint(a[i]); }
    puts_("\n");
    puts_("12345 * 6789 = "); putint((int)mul(12345, 6789)); puts_("\n");
    const char *msg = "The quick brown fox jumps over the lazy dog";
    unsigned n = 0; while (msg[n]) n++;
    puts_("crc32 = "); putint((int)crc32((const unsigned char *)msg, n)); puts_("\n");
    signed char sc = -100; unsigned char uc = 200;
    puts_("sign tests: "); putint(sc >> 2); puts_(" "); putint(uc >> 2); puts_(" "); putint(-17 >> 1); puts_("\n");
    return 42;
}
```

```console
$ cd rvsim && riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -O2 -nostdlib -static -ffreestanding -fno-builtin rt.c tests.c -o tests && qemu-riscv32 ./tests; echo "qemu exit status $?"
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
qemu exit status 42
$ cd rvsim && python3 rvsim.py tests; echo "rvsim exit status $?"
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[rvsim: 222847 instructions, exit 42]
rvsim exit status 42
$ cd rvsim && diff <(qemu-riscv32 ./tests) <(python3 rvsim.py tests 2>/dev/null) && echo "outputs identical"
outputs identical
$ cd rvsim && python3 rvsim.py ../fact; echo "fact.s under rvsim: exit status $?"
[rvsim: 357 instructions, exit 0]
fact.s under rvsim: exit status 0
```

Design notes: decoding follows the format table of Unit 4.2 directly — each format's immediate is reassembled from its scattered bits and sign-extended with `sext`. All register values are kept as unsigned 32-bit integers (`& MASK` after every operation) and converted with `sext` only where the instruction is signed (`slt`, `blt`, `bge`, `sra`, signed loads). The CRC check value 0x414FA339 (printed as the signed integer 1095738169) is the published CRC-32 of the "quick brown fox" sentence, a good end-to-end test of shifts, XOR, and unsigned comparisons. Python's speed — around a million instructions per second — is enough for the test programs of this book; Chapter 15 runs longer programs on the Verilog processor through Verilator.

### Review checklist

- Does `rvsim` reject an illegal instruction rather than silently skipping it? Try a word of zeros.
- Are writes to `x0` discarded? (`if val is not None and rd:`)
- What would it take to add the `M` extension (multiply/divide)? Which test programs would get faster?

## Answers to the Chapter 4 prediction questions

**Unit 4.1, Q1.** The stored value would overwrite the instruction at address 2: when the PC reached it, the machine would decode the stored number as an instruction — self-modifying code.

**Unit 4.2, Q1.** imm = 42 = `000000101010`, rs1 = 0, funct3 = 000, rd = 10 = `01010`, opcode `0010011`: `000000101010 00000 000 01010 0010011` = `0x02A00513`.

**Unit 4.2, Q2.** `lb` sign-extends 0x80: −128 (0xFFFFFF80). `lbu` zero-extends: 128.

**Unit 4.3, Q3.** Each "word" would start one byte later, so the loop would read overlapping, misaligned words assembled from the bytes of neighbouring elements — a nonsense sum (and on some RISC-V implementations, misaligned loads trap).

**Unit 4.4, Q2.** The `call mul` overwrites `ra` with the address after it, so `fact`'s final `ret` jumps back into `fact` itself instead of to its caller — an infinite loop (or a crash).

**Unit 4.4, Q3.** Nine frames of 16 bytes (fact(10) down to fact(2); fact(1) returns without a frame): 144 bytes.

**Unit 4.5, Q2.** With `volatile`, every read and write of `n` must go to memory: `n` gets a stack slot, and each iteration loads and stores it, like `-O0` — and the compiler can no longer replace the loop with a `strlen` call.

# Chapter 5 — Microarchitecture: Single-Cycle and Multi-Cycle Processors

*Goal: build a processor. This chapter implements the RV32I instruction set of Chapter 4 twice in Verilog — first as a single-cycle machine in which every instruction finishes in one long clock cycle, then as a multi-cycle machine that splits each instruction into short steps and reuses one ALU — and runs real C programs compiled by GCC on both, checking every output against the instruction-set simulator `rvsim`. Then it measures what each design costs and how fast each would be.*

*Accompanies the lectures "Microarchitecture Fundamentals" (single-cycle) and "Multi-Cycle Microarchitectures".*

**What you will be able to do after this chapter**

- Separate a processor into **datapath** (registers, ALU, memories, multiplexers) and **control** (the decoder or FSM that drives them).
- Read and modify a complete single-cycle RV32I processor in about 150 lines of Verilog.
- Run GCC-compiled programs on a Verilog processor and compare it, instruction for instruction, with a golden model.
- Build a multi-cycle processor with an FSM controller and measure its CPI and instruction mix.
- Compare designs with the iron law, using synthesised gate counts and path lengths, and explain why neither design is used in fast processors.

**How this chapter's code is organised.** `single/` holds the single-cycle CPU, `multi/` the multi-cycle one, `sw/` the build files for programs, and `project/` the RV32M extension. Programs are the C test program and runtime from Chapter 4 (`../ch04/rvsim/`), linked at address 0 and turned into a memory image of 32-bit words that `$readmemh` loads.

## Topic 5A — The single-cycle processor

## Unit 5.1 — Datapath, control, and architectural state

**One-line summary:** A microarchitecture implements the ISA's **architectural state** — the PC, the 32 registers, and memory — plus a **datapath** that computes the next state and a **control unit** that tells the datapath what to do for each instruction; the single-cycle design computes the entire next state of one instruction combinationally and commits it at one clock edge.

Everything a program can observe is architectural state: `pc`, `x0`–`x31`, and memory. The ISA defines, for each instruction, a function from the current state to the next. A microarchitecture is any circuit that computes that function. The **single-cycle** design does it in the most direct way:

```
          +--------+   instr   +---------+  a,b  +-----+  result  +-------+
pc ------>| instr  |---------->| decode, |------>| ALU |--------->| data  |---> writeback
  ^       | memory |           | reg file|       +-----+  address | memory|      to reg file
  |       +--------+           +---------+                        +-------+
  +--- next pc (pc + 4, branch target, or jump target) <---------------------------
```

In one clock period, the instruction is read from memory, its registers are read, the ALU computes, memory is read or written, and the result is ready to be written to the register file. At the clock edge, the PC, the destination register, and (for a store) memory are updated together. The **CPI is exactly 1**; the price is that the clock period must accommodate the slowest instruction — a load, which uses every stage in sequence.

The **control** in a single-cycle design is purely combinational: a decoder turns the opcode and function fields into signals — "write the register file", "the ALU's second operand is the immediate", "the result comes from memory", "the next PC is the branch target". In the design below these are the `is_*` wires and the expressions that use them.

### Mastery check

1. **Explain.** Which parts of the processor are state, and which are combinational?
2. **Explain.** Why must instruction memory and data memory be readable in the same cycle in a single-cycle design?

## Unit 5.2 — The single-cycle RV32I processor in Verilog

**One-line summary:** The whole processor is the five steps of the instruction cycle written as combinational logic — fetch, decode with an immediate generator, an ALU and branch comparator, a memory read path with sub-word extraction, a writeback multiplexer and next-PC logic — followed by one clocked block that commits the next state.

```verilog
// file: single/sc_cpu.v
// A single-cycle RV32I processor: every instruction completes in one clock cycle.
module sc_cpu #(parameter MEMWORDS = 16384, parameter HEX = "prog.hex", parameter M_EXT = 0)(input clk, input rst, output [31:0] a0);
    // ---------------- state: PC, registers, memory ----------------
    reg  [31:0] pc;
    reg  [31:0] rf [0:31];
    reg  [31:0] mem [0:MEMWORDS-1];
    integer k;
    initial begin
        for (k = 0; k < 32; k = k + 1) rf[k] = 0;
        rf[2] = MEMWORDS * 4;                         // sp: top of memory
`ifndef SYNTHESIS
        $readmemh(HEX, mem);                          // program image (simulation only)
`endif
    end

    // ---------------- fetch and decode ----------------
    wire [31:0] instr  = mem[pc[31:2]];
    wire [6:0]  opcode = instr[6:0];
    wire [4:0]  rd = instr[11:7], rs1 = instr[19:15], rs2 = instr[24:20];
    wire [2:0]  funct3 = instr[14:12];
    wire        funct7b5 = instr[30];
    wire [31:0] a = rf[rs1], b = rf[rs2];

    // immediate generator: one immediate per format, sign bit always instr[31]
    wire [31:0] immI = {{20{instr[31]}}, instr[31:20]};
    wire [31:0] immS = {{20{instr[31]}}, instr[31:25], instr[11:7]};
    wire [31:0] immB = {{19{instr[31]}}, instr[31], instr[7], instr[30:25], instr[11:8], 1'b0};
    wire [31:0] immU = {instr[31:12], 12'b0};
    wire [31:0] immJ = {{11{instr[31]}}, instr[31], instr[19:12], instr[20], instr[30:21], 1'b0};

    // main decoder
    wire is_lui = opcode == 7'b0110111, is_auipc = opcode == 7'b0010111;
    wire is_jal = opcode == 7'b1101111, is_jalr  = opcode == 7'b1100111;
    wire is_br  = opcode == 7'b1100011, is_load  = opcode == 7'b0000011;
    wire is_st  = opcode == 7'b0100011, is_opimm = opcode == 7'b0010011;
    wire is_op  = opcode == 7'b0110011, is_ecall = opcode == 7'b1110011;

    // ---------------- execute: the ALU ----------------
    // RV32M hook: a multiply/divide unit, present only when M_EXT = 1 (the chapter project)
    wire is_mext = M_EXT && is_op && instr[25];
    wire [31:0] mext;
    generate if (M_EXT) begin : m
        muldiv md(a, b, funct3, mext);
    end else begin : no_m
        assign mext = 32'b0;
    end endgenerate

    wire [31:0] alu_b = (is_op || is_br) ? b : (is_st ? immS : immI);
    wire [4:0]  shamt = alu_b[4:0];
    reg  [31:0] alu;
    always @(*) begin
        case (funct3)
            3'd0: alu = (is_op && funct7b5) ? a - alu_b : a + alu_b;
            3'd1: alu = a << shamt;
            3'd2: alu = {31'b0, $signed(a) < $signed(alu_b)};
            3'd3: alu = {31'b0, a < alu_b};
            3'd4: alu = a ^ alu_b;
            3'd5: alu = funct7b5 ? $signed(a) >>> shamt : a >> shamt;
            3'd6: alu = a | alu_b;
            default: alu = a & alu_b;
        endcase
        if (is_load || is_st) alu = a + alu_b;          // address calculation
        if (is_mext) alu = mext;                         // RV32M (the chapter project)
    end


    // branch condition
    reg take;
    always @(*) begin
        case (funct3)
            3'd0: take = a == b;            3'd1: take = a != b;
            3'd4: take = $signed(a) < $signed(b);   3'd5: take = $signed(a) >= $signed(b);
            3'd6: take = a < b;             3'd7: take = a >= b;
            default: take = 0;
        endcase
    end

    // ---------------- memory ----------------
    wire [31:0] word = mem[alu[31:2]];
    wire [1:0]  off = alu[1:0];
    wire [7:0]  byte_ = word >> (8 * off);
    wire [15:0] half = word >> (8 * off);
    reg  [31:0] load_val;
    always @(*) begin
        case (funct3)
            3'd0: load_val = {{24{byte_[7]}}, byte_};
            3'd1: load_val = {{16{half[15]}}, half};
            3'd4: load_val = {24'b0, byte_};
            3'd5: load_val = {16'b0, half};
            default: load_val = word;
        endcase
    end

    // ---------------- writeback and next PC ----------------
    wire [31:0] pc4 = pc + 4;
    reg  [31:0] wb;
    always @(*) begin
        if (is_lui) wb = immU;
        else if (is_auipc) wb = pc + immU;
        else if (is_jal || is_jalr) wb = pc4;
        else if (is_load) wb = load_val;
        else wb = alu;
    end
    wire reg_write = is_lui | is_auipc | is_jal | is_jalr | is_load | is_opimm | is_op;
    wire [31:0] next_pc = is_jal ? pc + immJ :
                          is_jalr ? ((a + immI) & ~32'd1) :
                          (is_br && take) ? pc + immB : pc4;

    // the clock edge: every state element updates at once
    always @(posedge clk) begin
        if (rst) pc <= 0;
        else begin
            pc <= next_pc;
            if (reg_write && rd != 0) rf[rd] <= wb;
            if (is_st) case (funct3)
                3'd0: mem[alu[31:2]] <= (word & ~(32'hFF   << (8 * off))) | ({24'b0, b[7:0]}  << (8 * off));
                3'd1: mem[alu[31:2]] <= (word & ~(32'hFFFF << (8 * off))) | ({16'b0, b[15:0]} << (8 * off));
                default: mem[alu[31:2]] <= b;
            endcase
        end
    end

    assign a0 = rf[10];                               // an output, so synthesis keeps the logic

`ifndef SYNTHESIS
    // simulation only: system calls (write, exit) and statistics
    integer cycles = 0, j;
    always @(posedge clk) if (!rst) begin
        cycles = cycles + 1;
        if (is_ecall) begin
            if (rf[17] == 64)
                for (j = 0; j < rf[12]; j = j + 1)
                    $write("%c", mem[(rf[11] + j) >> 2] >> (8 * ((rf[11] + j) & 3)));
            else if (rf[17] == 93) begin
                $display("[sc_cpu: exit %0d after %0d cycles]", rf[10], cycles);
                $finish;
            end
        end
    end
`endif
endmodule
```

Read it in the order the data flows:

- **State** (`pc`, `rf`, `mem`) and an `initial` block that clears the registers, points `sp` at the top of memory (normally the C runtime's job) and loads the program image. Memory is a single array for instructions and data — a Von Neumann memory with two read ports (one for the instruction, one for data), which simulation provides for free and real hardware does not (Unit 5.4).
- **Decode** splits the instruction into fields and builds all five immediates in parallel; the right one is selected later. The `is_*` wires are the main decoder.
- **Execute**: one `case` on `funct3` implements every ALU operation, with `funct7` bit 30 (`funct7b5`) choosing `sub` over `add` and `sra` over `srl`. The second operand is a register for R-type and branches, an immediate otherwise. Loads and stores override the operation to an addition: their `funct3` encodes the access *width*, not an ALU operation.
- **Memory**: memory is word-addressed (`alu[31:2]`), so byte and half-word loads extract their part from the word with a shift (`word >> (8 * off)`) and sign- or zero-extend it; stores do a read-modify-write of the word.
- **Writeback and next PC**: a priority chain picks what goes to `rd`, and `next_pc` is the jump target, the branch target if the branch is taken, or `pc + 4`.
- **Commit**: one `always @(posedge clk)` block updates the PC, the destination register (never `x0`), and memory — the only sequential logic in the processor.
- **Simulation only**: `ecall` is handled by the testbench side of the model, which prints the bytes of a `write` and stops at `exit` — standing in for an operating system.

### Mastery check

1. **Trace.** For `sw x8, -4(x2)`, list the value of `alu_b`, `alu`, and `reg_write`.
2. **Explain.** Why does `jalr` clear bit 0 of its target (`& ~32'd1`)?
3. **Predict.** What breaks if the commit block wrote `rf[rd]` without checking `rd != 0`?

## Unit 5.3 — Running C programs, checked against a golden model

**One-line summary:** A Verilog processor is verified the way a real one is: by running real programs and comparing everything observable — output, exit status, and the number of instructions — with an independent model of the ISA, here the Python simulator from Chapter 4.

Programs for the Verilog CPU are built with a linker script that puts `_start` at address 0 and discards sections the CPU does not need, and a small script that writes the binary image as one hexadecimal word per line:

```text
/* file: sw/link.ld */
ENTRY(_start)
SECTIONS {
  . = 0;
  .text   : { *(.text._start) *(.text*) }
  .rodata : { *(.rodata*) *(.srodata*) }
  .data   : { *(.data*) }
  __global_pointer$ = . + 0x800;
  .sdata  : { *(.sdata*) }
  .bss    : { *(.sbss*) *(.bss*) *(COMMON) }
  _end = .;
  /DISCARD/ : { *(.note*) *(.comment*) *(.riscv.attributes) *(.eh_frame*) }
}
```

```python
# file: sw/mkhex.py
"""Write a flat binary image as 32-bit little-endian words, one hex word per line (for $readmemh)."""
import sys
data = open(sys.argv[1], "rb").read()
data += bytes(-len(data) % 4)
with open(sys.argv[2], "w") as f:
    for i in range(0, len(data), 4):
        f.write(f"{int.from_bytes(data[i:i + 4], 'little'):08x}\n")
print(f"{sys.argv[2]}: {len(data) // 4} words")
```

```verilog
// file: single/tb.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    sc_cpu #(.HEX("../sw/tests.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd sw && riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -O2 -nostdlib -ffreestanding -fno-builtin -static -T link.ld -Wl,--build-id=none -Wl,--no-warn-rwx-segments ../../ch04/rvsim/rt.c ../../ch04/rvsim/tests.c -o tests.elf && riscv64-linux-gnu-objcopy -O binary tests.elf tests.bin && python3 mkhex.py tests.bin tests.hex
tests.hex: 592 words
$ cd single && iverilog -g2012 -o sim sc_cpu.v tb.v && vvp -n sim | grep -v WARNING
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[sc_cpu: exit 42 after 222847 cycles]
sc_cpu.v:137: $finish called at 2228475 (1s)
$ python3 ../ch04/rvsim/rvsim.py sw/tests.elf; echo "rvsim exit status $?"
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[rvsim: 222847 instructions, exit 42]
rvsim exit status 42
```

The Verilog processor prints exactly what QEMU and `rvsim` printed in Chapter 4, exits with 42, and takes 222,847 cycles — the same number as `rvsim`'s instruction count, as a CPI of 1 requires. (`$readmemh` warns that the image is smaller than the 64 KiB memory; that warning is filtered out above.) When the outputs differ, the next step is a **lockstep** comparison: print the PC and the register written at every instruction from both models and find the first line where they disagree. The fib, sort, multiply, CRC, and shift tests between them exercise every instruction class, and the CRC alone executes tens of thousands of shifts, XORs, and branches whose results feed back into later ones — one wrong bit anywhere changes the final number.

### Mastery check

1. **Explain.** Why is matching the instruction count a stronger check than matching the printed output alone?
2. **Design.** Describe the lockstep comparison you would build if the output differed.

## Unit 5.4 — What the single-cycle design costs

**One-line summary:** Synthesis shows the single-cycle processor's real costs — every instruction pays for the longest path (instruction memory, register read, ALU, data memory, writeback in series), and it needs separate instruction and data read ports — which is why no fast processor is built this way.

Yosys can synthesise the processor with a small memory (64 words, set with `chparam`) to show the size and depth of the logic. The output port `a0` exists only so that synthesis keeps the logic; a circuit with no outputs is optimised away completely.

```console
$ cd single && yosys -p "read_verilog sc_cpu.v; chparam -set MEMWORDS 64 sc_cpu; synth -top sc_cpu -flatten; stat; ltp -noff" > synth.log && grep -E "Number of cells|DFF|Longest" synth.log | tail -5 | tr -s ' '
 Number of cells: 15739
 $_DFFE_PP_ 3072
 $_SDFFE_PP0P_ 1
 $_SDFF_PP0_ 30
Longest topological path in sc_cpu (length=41):
```

About 3,000 flip-flops: 2,048 for the 64-word memory, 1,024 for the register file (all 32 registers; Chapter 2 showed why `x0` is not removed), and the PC. The longest path through the combinational logic is 41 gates deep: instruction-memory read multiplexer, register-file read multiplexer, ALU, data-memory read multiplexer, load extraction, writeback multiplexer — one long chain that every instruction, even an `addi`, must fit into one clock period.

Two more problems are not visible in the gate count:

- **Two memory reads per cycle.** The instruction fetch and a load both read memory in the same cycle. Real memories have one or two ports, so single-cycle designs need separate instruction and data memories (a **Harvard** organisation) — or, in modern processors, separate instruction and data *caches* (Chapter 10).
- **The register file needs two read ports and one write port** every cycle — acceptable, and every processor in this book keeps that.

### Mastery check

1. **Explain.** Why does every instruction pay for the load's path in a single-cycle design?
2. **Predict.** If the memory were 16 K words, as in simulation, roughly how would the gate count and the path length change?

## Topic 5B — The multi-cycle processor

## Unit 5.5 — Splitting instructions into steps

**One-line summary:** A multi-cycle processor breaks each instruction into steps — fetch, decode/register read, execute, memory, writeback — each taking one short clock cycle, with registers between the steps and an FSM sequencing them; instructions use only the steps they need, so their CPI varies from 3 to 5, and one ALU and one memory port are reused across steps.

Instead of one long cycle, the multi-cycle design adds **internal registers** (`ir`, `A`, `B`, `alu_out`, `mdr`) that hold intermediate results between short cycles, and replaces the combinational control with an **FSM**:

| Step | State | Work | Used by |
| --- | --- | --- | --- |
| 1 | FETCH | `ir <= mem[pc]` | all |
| 2 | DECODE | `A <= rf[rs1]`, `B <= rf[rs2]` | all |
| 3 | EXEC | ALU operation; branches, jumps, `lui`, `auipc` finish here | all |
| 4 | MEM | load into `mdr`, or store (stores finish here) | loads, stores |
| 5 | WB | write `rd` | ALU instructions, loads |

So branches and jumps take 3 cycles, stores and ALU instructions 4, and loads 5.

```verilog
// file: multi/mc_cpu.v
// A multi-cycle RV32I processor: each instruction takes 3 to 5 short cycles, sequenced by an FSM.
module mc_cpu #(parameter MEMWORDS = 16384, parameter HEX = "prog.hex")(input clk, input rst, output [31:0] a0);
    reg  [31:0] pc, ir, A, B, alu_out, mdr;
    reg  [31:0] rf [0:31];
    reg  [31:0] mem [0:MEMWORDS-1];
    integer k;
    initial begin
        for (k = 0; k < 32; k = k + 1) rf[k] = 0;
        rf[2] = MEMWORDS * 4;
`ifndef SYNTHESIS
        $readmemh(HEX, mem);                          // program image (simulation only)
`endif
    end

    localparam FETCH = 3'd0, DECODE = 3'd1, EXEC = 3'd2, MEM = 3'd3, WB = 3'd4;
    reg [2:0] state;

    // decode fields from the instruction register (stable for the whole instruction)
    wire [6:0] opcode = ir[6:0];
    wire [4:0] rd = ir[11:7];
    wire [2:0] funct3 = ir[14:12];
    wire funct7b5 = ir[30];
    wire [31:0] immI = {{20{ir[31]}}, ir[31:20]};
    wire [31:0] immS = {{20{ir[31]}}, ir[31:25], ir[11:7]};
    wire [31:0] immB = {{19{ir[31]}}, ir[31], ir[7], ir[30:25], ir[11:8], 1'b0};
    wire [31:0] immU = {ir[31:12], 12'b0};
    wire [31:0] immJ = {{11{ir[31]}}, ir[31], ir[19:12], ir[20], ir[30:21], 1'b0};
    wire is_lui = opcode == 7'b0110111, is_auipc = opcode == 7'b0010111;
    wire is_jal = opcode == 7'b1101111, is_jalr  = opcode == 7'b1100111;
    wire is_br  = opcode == 7'b1100011, is_load  = opcode == 7'b0000011;
    wire is_st  = opcode == 7'b0100011, is_opimm = opcode == 7'b0010011;
    wire is_op  = opcode == 7'b0110011, is_ecall = opcode == 7'b1110011;

    // one ALU, shared by every step (the point of multi-cycle: reuse hardware across cycles)
    wire [31:0] alu_b = (is_op || is_br) ? B : (is_st ? immS : immI);
    wire [4:0]  shamt = alu_b[4:0];
    reg  [31:0] alu;
    always @(*) begin
        case (funct3)
            3'd0: alu = (is_op && funct7b5) ? A - alu_b : A + alu_b;
            3'd1: alu = A << shamt;
            3'd2: alu = {31'b0, $signed(A) < $signed(alu_b)};
            3'd3: alu = {31'b0, A < alu_b};
            3'd4: alu = A ^ alu_b;
            3'd5: alu = funct7b5 ? $signed(A) >>> shamt : A >> shamt;
            3'd6: alu = A | alu_b;
            default: alu = A & alu_b;
        endcase
        if (is_load || is_st) alu = A + alu_b;
    end
    reg take;
    always @(*) begin
        case (funct3)
            3'd0: take = A == B;   3'd1: take = A != B;
            3'd4: take = $signed(A) < $signed(B);   3'd5: take = $signed(A) >= $signed(B);
            3'd6: take = A < B;    3'd7: take = A >= B;
            default: take = 0;
        endcase
    end

    wire [31:0] word = mem[alu_out[31:2]];
    wire [1:0]  off = alu_out[1:0];
    wire [7:0]  byte_ = word >> (8 * off);
    wire [15:0] half = word >> (8 * off);
    reg  [31:0] load_val;
    always @(*) case (funct3)
        3'd0: load_val = {{24{byte_[7]}}, byte_};
        3'd1: load_val = {{16{half[15]}}, half};
        3'd4: load_val = {24'b0, byte_};
        3'd5: load_val = {16'b0, half};
        default: load_val = word;
    endcase

    always @(posedge clk) begin
        if (rst) begin
            pc <= 0; state <= FETCH;
        end else case (state)
            FETCH:  begin ir <= mem[pc[31:2]]; state <= DECODE; end
            DECODE: begin A <= rf[ir[19:15]]; B <= rf[ir[24:20]]; state <= EXEC; end
            EXEC: begin
                state <= FETCH;                                        // short instructions end here
                if (is_br)       pc <= take ? pc + immB : pc + 4;
                else if (is_jal) begin if (rd != 0) rf[rd] <= pc + 4; pc <= pc + immJ; end
                else if (is_jalr) begin if (rd != 0) rf[rd] <= pc + 4; pc <= (A + immI) & ~32'd1; end
                else if (is_lui)  begin if (rd != 0) rf[rd] <= immU; pc <= pc + 4; end
                else if (is_auipc) begin if (rd != 0) rf[rd] <= pc + immU; pc <= pc + 4; end
                else if (is_load || is_st) begin alu_out <= alu; state <= MEM; end
                else if (is_op || is_opimm) begin alu_out <= alu; state <= WB; end
                else pc <= pc + 4;                                     // ecall, fence
            end
            MEM: begin
                if (is_load) begin mdr <= load_val; state <= WB; end
                else begin
                    case (funct3)
                        3'd0: mem[alu_out[31:2]] <= (word & ~(32'hFF   << (8 * off))) | ({24'b0, B[7:0]}  << (8 * off));
                        3'd1: mem[alu_out[31:2]] <= (word & ~(32'hFFFF << (8 * off))) | ({16'b0, B[15:0]} << (8 * off));
                        default: mem[alu_out[31:2]] <= B;
                    endcase
                    pc <= pc + 4; state <= FETCH;
                end
            end
            WB: begin
                if (rd != 0) rf[rd] <= is_load ? mdr : alu_out;
                pc <= pc + 4; state <= FETCH;
            end
        endcase
    end

    assign a0 = rf[10];

`ifndef SYNTHESIS
    integer cycles = 0, instrs = 0, j;
    integer n_load = 0, n_store = 0, n_alu = 0, n_branch = 0, n_jump = 0, n_other = 0;
    always @(posedge clk) if (!rst) begin
        cycles = cycles + 1;
        if (state == EXEC) begin
            instrs = instrs + 1;
            if (is_load) n_load = n_load + 1;
            else if (is_st) n_store = n_store + 1;
            else if (is_op || is_opimm) n_alu = n_alu + 1;
            else if (is_br) n_branch = n_branch + 1;
            else if (is_jal || is_jalr) n_jump = n_jump + 1;
            else n_other = n_other + 1;
            if (is_ecall) begin
                if (rf[17] == 64)
                    for (j = 0; j < rf[12]; j = j + 1)
                        $write("%c", mem[(rf[11] + j) >> 2] >> (8 * ((rf[11] + j) & 3)));
                else if (rf[17] == 93) begin
                    $display("[mc_cpu: exit %0d after %0d instructions, %0d cycles, CPI %0.2f]",
                             rf[10], instrs, cycles, 1.0 * cycles / instrs);
                    $display("[mix: load %0d store %0d alu %0d branch %0d jump %0d other %0d]",
                             n_load, n_store, n_alu, n_branch, n_jump, n_other);
                    $finish;
                end
            end
        end
    end
`endif
endmodule
```

```verilog
// file: multi/tb.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    mc_cpu #(.HEX("../sw/tests.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd multi && iverilog -g2012 -o sim mc_cpu.v tb.v && vvp -n sim | grep -v WARNING
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[mc_cpu: exit 42 after 222847 instructions, 908489 cycles, CPI 4.08]
[mix: load 43690 store 42010 alu 110558 branch 20430 jump 6078 other 81]
mc_cpu.v:134: $finish called at 9084895 (1s)
```

Same output, same 222,847 instructions — and 908,489 cycles, a CPI of 4.08. The instruction mix explains it: half the instructions are ALU operations (4 cycles), 38% are loads or stores (5 and 4), and only about 12% are branches and jumps (3). With these counts the CPI can be checked by hand: (43,690 × 5 + 42,010 × 4 + 110,558 × 4 + 20,430 × 3 + 6,078 × 3 + 81 × 3) / 222,847 ≈ 4.08.

### Mastery check

1. **Calculate.** A program is 30% loads, 10% stores, 45% ALU, 15% branches. What is its CPI on `mc_cpu`?
2. **Explain.** Why can the multi-cycle design use a single memory port for both instructions and data?
3. **Design.** Which instructions could finish one cycle earlier if the register file were written in the EXEC state? What would that cost?

## Unit 5.6 — Single-cycle versus multi-cycle, by the iron law

**One-line summary:** The multi-cycle design wins on hardware (one memory port, a shared ALU) and loses on time here: its clock period is not short enough to make up for four times as many cycles, because the steps are unbalanced and each adds register overhead — which is the problem pipelining solves by overlapping the steps instead of serialising them.

```console
$ cd multi && yosys -p "read_verilog mc_cpu.v; chparam -set MEMWORDS 64 mc_cpu; synth -top mc_cpu -flatten; stat; ltp -noff" > synth.log && grep -E "Number of cells|Longest" synth.log | tail -2 | tr -s ' '
 Number of cells: 12759
Longest topological path in mc_cpu (length=29):
```

Using the longest topological path as a stand-in for the clock period (a gate count, not nanoseconds — Chapter 3's caveats apply):

| Design | Cycles for `tests.c` | Path (gates) | Time ∝ cycles × path |
| --- | --- | --- | --- |
| Single-cycle | 222,847 | 41 | 9.1 million |
| Multi-cycle | 908,489 | 29 | 26.3 million |

The multi-cycle processor is almost three times *slower*. Its longest step is still 29 gates deep — the steps are not balanced: the EXEC state alone contains the ALU, the branch comparator, and the next-PC adders — so splitting the work into five steps shortened the cycle by only 30% while multiplying the cycle count by four. A real design would also pay a register's setup and clock-to-Q delay in every short cycle. The multi-cycle design's real advantages are elsewhere: one memory port, one ALU and adder instead of several, and the ability to give a long instruction (a multiply or divide, a floating-point operation) as many cycles as it needs without slowing down every other instruction.

The way out is to keep the multi-cycle design's short steps and *overlap* them: while one instruction is in EXEC, the next is in DECODE and the one after that in FETCH. That is **pipelining**, Chapter 6's subject — the CPI returns towards 1 while the clock period stays short.

### Deep & Tricky

**DT-19. A microarchitecture is correct only if it is correct for *every* instruction in *every* context, and a test program exercises a fraction of them: both processors in this chapter pass the test programs, yet neither implements `fence.i`, misaligned accesses (which RV32I allows implementations to trap or handle), or any exception — an illegal instruction simply executes as something else. Commercial processors are checked against the ISA with randomly generated instruction streams (e.g. Google's RISCV-DV) and formal proofs (e.g. the riscv-formal framework) in addition to programs.** *(RISC-V Unprivileged ISA Specification, §2.6: misaligned accesses "may be handled in hardware or by an invisible trap"; RISCV-DV and riscv-formal project documentation)* The first versions of many hobby RISC-V cores were found to mishandle `sra` on negative numbers, `jalr` with an odd target, or loads that sign-extend — exactly the cases this chapter's test program includes on purpose ("sign tests"), and which random generators find without being told to look.

### Mastery check

1. **Calculate.** If the multi-cycle design's steps were perfectly balanced, each 41/5 ≈ 8 gates deep plus 2 gates of register overhead, how would its time compare with the single-cycle design's?
2. **Explain.** Why does a multi-cycle design make it easy to add a slow multi-cycle divide instruction, while a single-cycle design does not?

## Chapter 5 Project — Adding the M extension

### Requirements

Extend the single-cycle processor with RV32M: `mul`, `mulh`, `mulhsu`, `mulhu`, `div`, `divu`, `rem`, `remu`. RISC-V defines the edge cases precisely: division by zero returns −1 (all ones) as the quotient and the dividend as the remainder, and the signed overflow case `−2³¹ / −1` returns −2³¹ with remainder 0 — no exceptions. Extend `rvsim` the same way as the golden model. Verify with a C program compiled with `-march=rv32im` that exercises the edge cases, comparing QEMU, `rvsim_m`, and the Verilog processor; then measure how many instructions the M extension saves on a matrix multiplication against the same program using software multiplication on RV32I.

### Reference solution

The CPU already has the hook (Unit 5.2): with `M_EXT = 1` it instantiates `muldiv` and, for R-type instructions with `funct7 = 0000001`, uses its result instead of the ALU's.

```verilog
// file: project/muldiv.v
// RV32M: multiply, divide, and remainder — combinational (one cycle in sc_cpu).
module muldiv(input [31:0] a, input [31:0] b, input [2:0] funct3, output reg [31:0] y);
    wire signed [63:0] p_ss = $signed(a) * $signed(b);                    // MUL, MULH
    wire signed [63:0] p_su = $signed({{32{a[31]}}, a}) * $signed({32'b0, b});   // MULHSU
    wire        [63:0] p_uu = {32'b0, a} * {32'b0, b};                    // MULHU
    wire div_by_zero = (b == 0);
    wire overflow = (a == 32'h8000_0000) && (b == 32'hFFFF_FFFF);         // -2^31 / -1
    wire [31:0] abs_a = a[31] ? -a : a, abs_b = b[31] ? -b : b;
    wire [31:0] uq = abs_a / abs_b, ur = abs_a % abs_b;                  // magnitudes
    wire [31:0] sq = (a[31] ^ b[31]) ? -uq : uq;                          // quotient rounds toward zero
    wire [31:0] sr = a[31] ? -ur : ur;                                    // remainder has the dividend's sign
    always @(*) begin
        case (funct3)
            3'd0: y = p_ss[31:0];                                         // MUL
            3'd1: y = p_ss[63:32];                                        // MULH
            3'd2: y = p_su[63:32];                                        // MULHSU
            3'd3: y = p_uu[63:32];                                        // MULHU
            3'd4: y = div_by_zero ? 32'hFFFF_FFFF : overflow ? a : sq;    // DIV
            3'd5: y = div_by_zero ? 32'hFFFF_FFFF : a / b;                // DIVU
            3'd6: y = div_by_zero ? a : overflow ? 32'd0 : sr;            // REM
            default: y = div_by_zero ? a : a % b;                         // REMU
        endcase
    end
endmodule
```

```python
# file: project/rvsim_m.py
"""rvsim with the RV32M extension: the golden model for sc_cpu with M_EXT = 1."""
import os
import sys
sys.path.insert(0, os.path.join(os.path.dirname(os.path.abspath(__file__)), "../../ch04/rvsim"))
from rvsim import CPU, sext, MASK

def tdiv(a, b):                       # signed division rounding toward zero, as RISC-V requires
    q = abs(a) // abs(b)
    return q if (a < 0) == (b < 0) else -q

class CPUM(CPU):
    def step(self):
        w = self.ld(self.pc, 4, False)
        if w & 0x7F != 0x33 or w >> 25 != 1:
            return super().step()                               # not an M instruction
        rd, f3 = w >> 7 & 31, w >> 12 & 7
        a, b = self.x[w >> 15 & 31], self.x[w >> 20 & 31]
        sa, sb = sext(a, 32), sext(b, 32)
        if f3 == 0:   v = a * b
        elif f3 == 1: v = (sa * sb) >> 32
        elif f3 == 2: v = (sa * b) >> 32
        elif f3 == 3: v = (a * b) >> 32
        elif f3 == 4: v = -1 if b == 0 else (sa if (sa == -2**31 and sb == -1) else tdiv(sa, sb))
        elif f3 == 5: v = MASK if b == 0 else a // b
        elif f3 == 6: v = sa if b == 0 else (0 if (sa == -2**31 and sb == -1) else sa - tdiv(sa, sb) * sb)
        else:         v = a if b == 0 else a % b
        if rd:
            self.x[rd] = v & MASK
        self.pc = (self.pc + 4) & MASK
        self.count += 1

if __name__ == "__main__":
    cpu = CPUM()
    cpu.load_elf(sys.argv[1])
    code = cpu.run()
    sys.stdout.write(cpu.out.decode(errors="replace"))
    print(f"[rvsim_m: {cpu.count} instructions, exit {code}]", file=sys.stderr)
    sys.exit(code)
```

```c
// file: project/mtest.c
// Multiply/divide workload. Built two ways: -march=rv32im (hardware M) and with -DSOFT for rv32i.
#include "rt.h"

#ifdef SOFT
static unsigned umul(unsigned a, unsigned b) { unsigned r = 0; while (b) { if (b & 1) r += a; a <<= 1; b >>= 1; } return r; }
#define MUL(a, b) umul((a), (b))
#else
#define MUL(a, b) ((unsigned)(a) * (unsigned)(b))
#endif

static unsigned matmul_checksum(void) {             // 12x12 integer matrix product
    enum { N = 12 };
    static unsigned A[N][N], B[N][N], C[N][N];
    unsigned seed = 12345;
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            seed = MUL(seed, 1103515245u) + 12345u; A[i][j] = seed >> 20;
            seed = MUL(seed, 1103515245u) + 12345u; B[i][j] = seed >> 20;
        }
    unsigned sum = 0;
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            unsigned s = 0;
            for (int k = 0; k < N; k++) s += MUL(A[i][k], B[k][j]);
            C[i][j] = s;
            sum ^= s + (unsigned)i;
        }
    return sum;
}

int main(void) {
    puts_("matmul checksum: "); putint((int)matmul_checksum()); puts_("\n");
#ifndef SOFT
    volatile int x = 7, y = -2, z = 0, m = (int)0x80000000, n1 = -1;   // volatile: computed at run time
    puts_("7/-2 = "); putint(x / y); puts_(", 7%-2 = "); putint(x % y);
    puts_(", -7/2 = "); putint(-x / -y); puts_(", -7%2 = "); putint(-x % -y); puts_("\n");
    puts_("7/0 = "); putint(x / z); puts_(", 7%0 = "); putint(x % z); puts_("\n");
    puts_("INT_MIN/-1 = "); putint(m / n1); puts_(", INT_MIN%-1 = "); putint(m % n1); puts_("\n");
    volatile unsigned u = 0xFFFFFFFFu;
    puts_("mulhu(0xFFFFFFFF, 0xFFFFFFFF) = "); putint((int)(((unsigned long long)u * u) >> 32)); puts_("\n");
#endif
    return 0;
}
```

```verilog
// file: project/tb_m.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    sc_cpu #(.HEX("mtest_m.hex"), .M_EXT(1)) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd project && CF="-mabi=ilp32 -O2 -nostdlib -ffreestanding -fno-builtin -static -T ../sw/link.ld -Wl,--build-id=none -Wl,--no-warn-rwx-segments -I../../ch04/rvsim" && riscv64-linux-gnu-gcc -march=rv32im $CF ../../ch04/rvsim/rt.c mtest.c -o mtest_m.elf && riscv64-linux-gnu-gcc -march=rv32i -DSOFT $CF ../../ch04/rvsim/rt.c mtest.c -o mtest_soft.elf && riscv64-linux-gnu-objcopy -O binary mtest_m.elf m.bin && python3 ../sw/mkhex.py m.bin mtest_m.hex
mtest_m.hex: 302 words
$ cd project && qemu-riscv32 ./mtest_m.elf
matmul checksum: 81203238
7/-2 = -3, 7%-2 = 1, -7/2 = -3, -7%2 = -1
7/0 = -1, 7%0 = 7
INT_MIN/-1 = -2147483648, INT_MIN%-1 = 0
mulhu(0xFFFFFFFF, 0xFFFFFFFF) = -2
$ cd project && python3 rvsim_m.py mtest_m.elf 2>&1 | tail -4 && python3 ../../ch04/rvsim/rvsim.py mtest_soft.elf
7/0 = -1, 7%0 = 7
INT_MIN/-1 = -2147483648, INT_MIN%-1 = 0
mulhu(0xFFFFFFFF, 0xFFFFFFFF) = -2
[rvsim_m: 16981 instructions, exit 0]
matmul checksum: 81203238
[rvsim: 179453 instructions, exit 0]
$ cd project && iverilog -g2012 -o sim ../single/sc_cpu.v muldiv.v tb_m.v && vvp -n sim | grep -v WARNING
matmul checksum: 81203238
7/-2 = -3, 7%-2 = 1, -7/2 = -3, -7%2 = -1
7/0 = -1, 7%0 = 7
INT_MIN/-1 = -2147483648, INT_MIN%-1 = 0
mulhu(0xFFFFFFFF, 0xFFFFFFFF) = -2
[sc_cpu: exit 0 after 16981 cycles]
../single/sc_cpu.v:137: $finish called at 169815 (1s)
$ cd project && riscv64-linux-gnu-objdump -d mtest_m.elf | grep -oE "\s(mul|mulh|mulhu|mulhsu|div|divu|rem|remu)\s" | sort | uniq -c
      4 	div	
      3 	mul	
      1 	mulhu	
      4 	rem	
```

The three models agree on every edge case: 7 / −2 = −3 with remainder 1 (division rounds toward zero, and the remainder takes the dividend's sign), division by zero gives −1 and the dividend, and `INT_MIN / −1` gives `INT_MIN` with remainder 0. The matrix product with hardware multiply takes about 17,000 instructions against about 179,000 with shift-and-add software multiplication — more than ten times fewer, and the M build's count even includes the edge-case tests. On the single-cycle processor each `mul` takes one cycle, which is unrealistic: a 32 × 32 multiplier is large and slow (Yosys turns the `*` operators into thousands of gates), so real processors pipeline it over three or four cycles and implement division iteratively over 32 or more — another reason processors moved beyond single-cycle designs.

### Review checklist

- Do `rvsim_m`, QEMU, and `sc_cpu` agree on `mulhsu` with a negative first operand? (Add a test: the objdump count shows which M instructions GCC actually emitted.)
- Is `M_EXT = 0` still bit-for-bit the processor of Unit 5.2? (Rerun Unit 5.3.)
- How would you implement `div` in the multi-cycle processor without a combinational divider?

## Answers to the Chapter 5 prediction questions

**Unit 5.2, Q1.** `alu_b = immS` (−4), `alu = x2 − 4` (the address), `reg_write = 0` (stores write memory, not a register).

**Unit 5.2, Q3.** An instruction with `rd = x0` — for example `jal x0, label` (a plain jump) or `addi x0, x0, 0` (`nop`) — would write `x0`, and later reads of `x0` would no longer return 0, breaking every instruction that relies on it.

**Unit 5.4, Q2.** The memory flip-flops would grow 256-fold (to over 500,000) and the memory read multiplexers would gain 8 levels, lengthening the path — which is why real memories are dense SRAM arrays, not flip-flops and multiplexers.

**Unit 5.5, Q1.** 0.30 × 5 + 0.10 × 4 + 0.45 × 4 + 0.15 × 3 = 1.5 + 0.4 + 1.8 + 0.45 = 4.15.

**Unit 5.6, Q1.** Path ≈ 10 gates; time ∝ 908,489 × 10 ≈ 9.1 million — about equal to the single-cycle design. Balancing alone does not win; overlapping (pipelining) does.

# Chapter 6 — Pipelining

*Goal: get the short clock cycle of the multi-cycle design and a CPI close to 1 at the same time, by overlapping instructions — while one executes, the next is decoded and the one after that is fetched. This chapter builds a five-stage pipelined RV32I processor in Verilog, then deals with what overlapping breaks: data hazards (an instruction needs a result that is still in the pipeline), handled by forwarding and stalls, and control hazards (the pipeline fetched past a branch before knowing where it goes), handled by flushing. It runs the same programs as Chapter 5, watches the pipeline cycle by cycle, and measures what each hazard costs.*

*Accompanies the lectures "Pipelining" and "Pipelined Processor Design" (data and control dependence handling).*

**What you will be able to do after this chapter**

- Explain pipelining in terms of throughput and latency, and say why stage balance and register overhead limit it.
- Read a five-stage pipelined processor in Verilog: pipeline registers, forwarding multiplexers, a hazard detection unit, and flush logic.
- Identify RAW data hazards and the cases forwarding cannot remove (load-use), and control hazards and their flush penalty.
- Visualise a pipeline cycle by cycle and verify it with a hazard "torture test" against a golden model.
- Compare single-cycle, multi-cycle, and pipelined processors with measured CPI and synthesised path lengths.

**How this chapter's code is organised.** `pipe/` holds the pipelined CPU and its testbench (the test program image is the one built in Chapter 5, `../ch05/sw/tests.hex`); `torture/` holds the hazard test and the pipeline viewer; `compare/` re-synthesises all three processors with the same memory size.

## Topic 6A — Overlapping instructions

## Unit 6.1 — The pipelining idea

**One-line summary:** Pipelining divides the work of an instruction into stages separated by registers and starts a new instruction every cycle, so *throughput* approaches one instruction per (short) cycle even though each instruction's *latency* — the time from fetch to writeback — does not shrink and even grows a little.

A laundry with a washer (30 minutes), a dryer (40), and a folding table (20) can do one load in 90 minutes. Doing four loads one after another takes 360 minutes. Starting the next load in the washer as soon as the previous one moves to the dryer, the loads overlap, and after the first one finishes, a load comes out every 40 minutes — the time of the *slowest* stage. Two lessons are already there: the speedup is limited by the slowest stage (balance matters), and each load still takes at least 90 minutes (latency does not improve).

The five-stage RISC pipeline uses the multi-cycle design's steps as stages:

| Stage | Work | Pipeline register after it |
| --- | --- | --- |
| IF | fetch the instruction at `pc` | IF/ID: instruction, its PC |
| ID | decode, read registers, detect hazards | ID/EX: operands, control signals, register numbers |
| EX | ALU operation, branch decision, target address | EX/MEM: result or address, store data |
| MEM | data memory read or write | MEM/WB: result |
| WB | write the register file | — |

Each pipeline register holds everything the later stages need about *its* instruction — including control signals decoded in ID, and the instruction's own register numbers — because by the time an instruction reaches MEM, the decoder is busy with an instruction three places younger.

Ideal speedup with *k* balanced stages is *k*. In practice: the stages are unequal (the clock must fit the slowest); each stage adds a register's setup time and clock-to-Q delay; and **hazards** — situations where the next instruction cannot proceed in the next cycle — add stall cycles. The rest of this chapter is about hazards.

### Mastery check

1. **Calculate.** Stages of 300, 400, 350, 400, and 250 ps, plus 50 ps register overhead per stage. What are the pipelined cycle time and the latency of one instruction? What would the single-cycle cycle time be?
2. **Explain.** Why must the destination register number travel down the pipeline with the instruction?

## Unit 6.2 — A five-stage pipeline in Verilog

**One-line summary:** The pipelined processor is the single-cycle processor cut into five pieces by four pipeline registers, plus three additions: a **hazard unit** in ID that stalls, **forwarding multiplexers** in EX that take operands from younger pipeline registers, and **flush** logic that turns wrong-path instructions into bubbles when a branch or jump redirects the PC.

```verilog
// file: pipe/pipe_cpu.v
// A five-stage pipelined RV32I processor: IF, ID, EX, MEM, WB.
// Hazards: forwarding (FORWARD = 1) or stalling (FORWARD = 0) for data hazards,
// a one-cycle stall for load-use, and predict-not-taken with a two-instruction flush.
module pipe_cpu #(parameter MEMWORDS = 16384, parameter HEX = "prog.hex", parameter FORWARD = 1)
                 (input clk, input rst, output [31:0] a0);
    reg [31:0] rf [0:31];
    reg [31:0] mem [0:MEMWORDS-1];
    integer k;
    initial begin
        for (k = 0; k < 32; k = k + 1) rf[k] = 0;
        rf[2] = MEMWORDS * 4;
`ifndef SYNTHESIS
        $readmemh(HEX, mem);
`endif
    end
    localparam NOP = 32'h0000_0013;                       // addi x0, x0, 0: a bubble

    // ===================== IF =====================
    reg  [31:0] pc;
    wire        stall;                                    // from the hazard unit (ID)
    wire        redirect;                                 // from EX: taken branch or jump
    wire [31:0] redirect_pc;
    always @(posedge clk)
        if (rst) pc <= 0;
        else if (redirect) pc <= redirect_pc;
        else if (!stall) pc <= pc + 4;

    reg [31:0] d_instr, d_pc;                             // IF/ID pipeline register
    always @(posedge clk)
        if (rst || redirect) begin d_instr <= NOP; d_pc <= 0; end   // flush the wrong-path fetch
        else if (!stall) begin d_instr <= mem[pc[31:2]]; d_pc <= pc; end

    // ===================== ID =====================
    wire [6:0] d_op = d_instr[6:0];
    wire [4:0] d_rs1 = d_instr[19:15], d_rs2 = d_instr[24:20], d_rd = d_instr[11:7];
    wire d_uses_rs1 = !(d_op == 7'b0110111 || d_op == 7'b0010111 || d_op == 7'b1101111);   // not lui/auipc/jal
    wire d_uses_rs2 = (d_op == 7'b0110011 || d_op == 7'b0100011 || d_op == 7'b1100011);    // R, store, branch
    wire d_writes = !(d_op == 7'b0100011 || d_op == 7'b1100011 || d_op == 7'b1110011) && d_rd != 0;

    // register read with write-through: a value written by WB this cycle is read in the same cycle
    wire [31:0] d_a = (w_we && w_rd == d_rs1 && d_rs1 != 0) ? w_val : rf[d_rs1];
    wire [31:0] d_b = (w_we && w_rd == d_rs2 && d_rs2 != 0) ? w_val : rf[d_rs2];

    reg [31:0] e_instr, e_pc, e_a, e_b;                   // ID/EX pipeline register
    reg        e_we, e_load;
    reg [4:0]  e_rd, e_rs1, e_rs2;

    // hazard detection
    wire hit_e = e_we && ((d_uses_rs1 && e_rd == d_rs1) || (d_uses_rs2 && e_rd == d_rs2));
    wire hit_m = m_we && ((d_uses_rs1 && m_rd == d_rs1) || (d_uses_rs2 && m_rd == d_rs2));
    assign stall = FORWARD ? (e_load && hit_e)            // load-use: the value exists only after MEM
                           : (hit_e || hit_m);           // no forwarding: wait until WB writes it

    always @(posedge clk)
        if (rst || redirect || stall) begin               // insert a bubble
            e_instr <= NOP; e_we <= 0; e_load <= 0; e_rd <= 0; e_rs1 <= 0; e_rs2 <= 0; e_pc <= 0;
        end else begin
            e_instr <= d_instr; e_pc <= d_pc; e_a <= d_a; e_b <= d_b;
            e_we <= d_writes; e_load <= (d_op == 7'b0000011);
            e_rd <= d_rd; e_rs1 <= d_rs1; e_rs2 <= d_rs2;
        end

    // ===================== EX =====================
    // forwarding: the newest value of a register may be in EX/MEM or MEM/WB
    wire [31:0] fa = (FORWARD && m_we && m_rd == e_rs1 && e_rs1 != 0) ? m_val :
                     (FORWARD && w_we && w_rd == e_rs1 && e_rs1 != 0) ? w_val : e_a;
    wire [31:0] fb = (FORWARD && m_we && m_rd == e_rs2 && e_rs2 != 0) ? m_val :
                     (FORWARD && w_we && w_rd == e_rs2 && e_rs2 != 0) ? w_val : e_b;
    wire [6:0] op = e_instr[6:0];
    wire [2:0] f3 = e_instr[14:12];
    wire f7b5 = e_instr[30];
    wire [31:0] immI = {{20{e_instr[31]}}, e_instr[31:20]};
    wire [31:0] immS = {{20{e_instr[31]}}, e_instr[31:25], e_instr[11:7]};
    wire [31:0] immB = {{19{e_instr[31]}}, e_instr[31], e_instr[7], e_instr[30:25], e_instr[11:8], 1'b0};
    wire [31:0] immU = {e_instr[31:12], 12'b0};
    wire [31:0] immJ = {{11{e_instr[31]}}, e_instr[31], e_instr[19:12], e_instr[20], e_instr[30:21], 1'b0};
    wire is_lui = op == 7'b0110111, is_auipc = op == 7'b0010111, is_jal = op == 7'b1101111;
    wire is_jalr = op == 7'b1100111, is_br = op == 7'b1100011, is_load = op == 7'b0000011;
    wire is_st = op == 7'b0100011, is_r = op == 7'b0110011;
    wire [31:0] y = (is_r || is_br) ? fb : (is_st ? immS : immI);
    reg  [31:0] alu;
    always @(*) begin
        case (f3)
            3'd0: alu = (is_r && f7b5) ? fa - y : fa + y;
            3'd1: alu = fa << y[4:0];
            3'd2: alu = {31'b0, $signed(fa) < $signed(y)};
            3'd3: alu = {31'b0, fa < y};
            3'd4: alu = fa ^ y;
            3'd5: alu = f7b5 ? $signed(fa) >>> y[4:0] : fa >> y[4:0];
            3'd6: alu = fa | y;
            default: alu = fa & y;
        endcase
        if (is_load || is_st) alu = fa + y;
        if (is_lui) alu = immU;
        if (is_auipc) alu = e_pc + immU;
        if (is_jal || is_jalr) alu = e_pc + 4;            // link value
    end
    reg take;
    always @(*) case (f3)
        3'd0: take = fa == fb;   3'd1: take = fa != fb;
        3'd4: take = $signed(fa) < $signed(fb);   3'd5: take = $signed(fa) >= $signed(fb);
        3'd6: take = fa < fb;    3'd7: take = fa >= fb;
        default: take = 0;
    endcase
    assign redirect = (is_br && take) || is_jal || is_jalr;
    assign redirect_pc = is_jalr ? ((fa + immI) & ~32'd1) : e_pc + (is_jal ? immJ : immB);

    reg [31:0] m_instr, m_val, m_b, m_pc;                 // EX/MEM pipeline register
    reg        m_we, m_load, m_st;
    reg [4:0]  m_rd;
    always @(posedge clk)
        if (rst) begin m_instr <= NOP; m_we <= 0; m_load <= 0; m_st <= 0; m_rd <= 0; end
        else begin
            m_instr <= e_instr; m_val <= alu; m_b <= fb; m_pc <= e_pc;
            m_we <= e_we; m_load <= is_load; m_st <= is_st; m_rd <= e_rd;
        end

    // ===================== MEM =====================
    wire [2:0]  mf3 = m_instr[14:12];
    wire [31:0] word = mem[m_val[31:2]];
    wire [1:0]  off = m_val[1:0];
    wire [7:0]  byte_ = word >> (8 * off);
    wire [15:0] half = word >> (8 * off);
    reg  [31:0] load_val;
    always @(*) case (mf3)
        3'd0: load_val = {{24{byte_[7]}}, byte_};
        3'd1: load_val = {{16{half[15]}}, half};
        3'd4: load_val = {24'b0, byte_};
        3'd5: load_val = {16'b0, half};
        default: load_val = word;
    endcase
    always @(posedge clk)
        if (!rst && m_st) case (mf3)
            3'd0: mem[m_val[31:2]] <= (word & ~(32'hFF   << (8 * off))) | ({24'b0, m_b[7:0]}  << (8 * off));
            3'd1: mem[m_val[31:2]] <= (word & ~(32'hFFFF << (8 * off))) | ({16'b0, m_b[15:0]} << (8 * off));
            default: mem[m_val[31:2]] <= m_b;
        endcase

    reg [31:0] w_instr, w_val, w_pc;                      // MEM/WB pipeline register
    reg        w_we;
    reg [4:0]  w_rd;
    always @(posedge clk)
        if (rst) begin w_instr <= NOP; w_we <= 0; w_rd <= 0; end
        else begin
            w_instr <= m_instr; w_val <= m_load ? load_val : m_val; w_pc <= m_pc;
            w_we <= m_we; w_rd <= m_rd;
        end

    // ===================== WB =====================
    always @(posedge clk) if (!rst && w_we) rf[w_rd] <= w_val;
    assign a0 = rf[10];

`ifndef SYNTHESIS
    integer cycles = 0, retired = 0, stalls = 0, flushes = 0, j;
    reg trace;
    initial trace = $test$plusargs("trace");
    always @(posedge clk) if (!rst) begin
        cycles = cycles + 1;
        if (trace && cycles <= 60)                        // which instruction is in each stage
            $display("T %0d %h %h %h %h %h %0d %0d", cycles, pc,
                     d_instr == NOP ? 32'hFFFFFFFF : d_pc, e_instr == NOP ? 32'hFFFFFFFF : e_pc,
                     m_instr == NOP ? 32'hFFFFFFFF : m_pc, w_instr == NOP ? 32'hFFFFFFFF : w_pc, stall, redirect);
        if (w_instr != NOP) retired = retired + 1;         // bubbles are NOPs (and so is a real nop)
        if (stall && !redirect) stalls = stalls + 1;
        if (redirect) flushes = flushes + 1;
        if (w_instr[6:0] == 7'b1110011) begin             // ecall reaches WB: older instructions are done
            if (rf[17] == 64)
                for (j = 0; j < rf[12]; j = j + 1)
                    $write("%c", mem[(rf[11] + j) >> 2] >> (8 * ((rf[11] + j) & 3)));
            else if (rf[17] == 93) begin
                $display("[pipe_cpu FORWARD=%0d: exit %0d, %0d instructions, %0d cycles, CPI %0.3f]",
                         FORWARD, rf[10], retired, cycles, 1.0 * cycles / retired);
                $display("[stall cycles %0d, redirects %0d (each flushes 2 instructions)]", stalls, flushes);
                $finish;
            end
        end
    end
`endif
endmodule
```

Reading guide:

- **Stage by stage.** Each stage's combinational logic reads the pipeline register *before* it (`d_*` for ID, `e_*` for EX, `m_*` for MEM, `w_*` for WB) and the clocked block at its end writes the next one. The instruction travels with its control information: `e_we` says it will write a register, `e_load` that it is a load, `e_rd` which register.
- **Bubbles are `nop`s.** Flushing or stalling writes `addi x0, x0, 0` (0x00000013) into a pipeline register with all write enables off. An instruction that writes nothing and stores nothing is harmless wherever it is.
- **Register read with write-through.** In the same cycle, WB writes a register and ID may read it. Real register files write in the first half of the cycle and read in the second; here the ID read multiplexer simply takes WB's value when the register numbers match.
- **The hazard unit** (`hit_e`, `hit_m`, `stall`) compares the source registers of the instruction in ID with the destinations of the instructions in EX and MEM (Unit 6.3).
- **Forwarding** (`fa`, `fb`) replaces a stale register value with the newer one from EX/MEM or MEM/WB, preferring the youngest (EX/MEM) when both match.
- **Redirect.** Branches and jumps are resolved in EX. When the PC must change, the two younger instructions already fetched (in IF/ID and being fetched) are on the wrong path: the IF/ID and ID/EX registers are loaded with bubbles (Unit 6.4).
- **Statistics.** A `retired` counter counts non-`nop` instructions reaching WB; the `ecall` handling moves to WB, where every older instruction has finished.

```verilog
// file: pipe/tb.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_cpu #(.HEX("../../ch05/sw/tests.hex"), .FORWARD(1)) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd pipe && iverilog -g2012 -o sim pipe_cpu.v tb.v && vvp -n sim | grep -v WARNING
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[pipe_cpu FORWARD=1: exit 42, 222847 instructions, 255469 cycles, CPI 1.146]
[stall cycles 1376, redirects 15622 (each flushes 2 instructions)]
pipe_cpu.v:175: $finish called at 2554695 (1s)
```

The same program, the same output, the same 222,847 instructions — in 255,469 cycles: CPI 1.146, against 1 for the single-cycle and 4.08 for the multi-cycle processor. The 32,622 extra cycles are exactly accounted for: 1,376 load-use stalls, 15,622 redirects that each flush two instructions (31,244 cycles), and 2 cycles to fill the pipeline after reset (the ecall's own trip to WB is counted inside the instructions).

### Mastery check

1. **Explain.** Why does the hazard unit compare the *source* registers in ID with the *destination* registers further down?
2. **Predict.** If a program had no branches, jumps, or loads, what CPI would this pipeline achieve?

## Topic 6B — Hazards

## Unit 6.3 — Data hazards: forwarding and the load-use stall

**One-line summary:** A read-after-write (RAW) hazard occurs when an instruction needs a register that an older instruction in the pipeline has not yet written back; **forwarding** sends the result from the pipeline register where it already exists to the ALU input, eliminating the wait for ALU results, but a load's result exists only after MEM, so an instruction that uses it immediately must **stall** one cycle.

Consider `addi t1, t0, 1` followed by `add t2, t1, t0`. When `add` is in ID reading `t1`, `addi` is in EX — its result does not even exist yet. Without forwarding, `add` must wait until `addi` has written the register file: two stall cycles (with the write-through register file). With forwarding, `add` reads the stale `t1` in ID, and one cycle later, in EX, the forwarding multiplexer replaces it with `addi`'s result from the EX/MEM register.

A load is different: `lw t4, 0(t6)` followed by `add t5, t4, t4`. When `add` reaches EX, the load is in MEM and the data is still being read from memory. The value exists at the end of MEM, too late for the `add` that is executing at the same time. The hazard unit detects this case — instruction in EX is a load, and its destination is a source of the instruction in ID — and stalls: the PC and IF/ID hold their values, and a bubble enters EX. One cycle later the load is in WB and its value is forwarded from MEM/WB.

The processor has a parameter to turn forwarding off, so the difference can be measured:

```verilog
// file: pipe/tb_nofwd.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_cpu #(.HEX("../../ch05/sw/tests.hex"), .FORWARD(0)) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd pipe && iverilog -g2012 -o sim_nofwd pipe_cpu.v tb_nofwd.v && vvp -n sim_nofwd | grep -E "^\[pipe"
[pipe_cpu FORWARD=0: exit 42, 222847 instructions, 296029 cycles, CPI 1.328]
```

Without forwarding, the program stalls for 41,936 cycles instead of 1,376, and the CPI rises from 1.146 to 1.328. Compilers help too: GCC's instruction scheduler moves an independent instruction between a load and its use whenever it can, which is why only 1,376 of the 43,690 loads in this program cause a stall.

### Deep & Tricky

**DT-20. Forwarding must take the *youngest* matching value: if both the instruction in MEM and the one in WB write the register being read, the one in MEM is newer and must win. Reversing the priority of the two forwarding paths passes many tests and silently computes with stale values when the same register is written twice in a row — which compilers do constantly (`addi a0, a0, 1; addi a0, a0, 1`).** *(Harris & Harris, "Digital Design and Computer Architecture: RISC-V Edition", §7.5.3 "Data Hazards": "if both the Memory and Writeback stages contain matching destination registers, the Memory stage should have priority, because it contains the more recently executed instruction")* In `pipe_cpu.v` the priority is the order of the conditional operators in `fa` and `fb`: the EX/MEM test comes first. The hazard torture test in this chapter's Unit 6.5 contains such chains on purpose.

### Mastery check

1. **Count.** In `add t2, t1, t0` following `addi t1, t0, 1` following `li t0, 5`, from which pipeline registers are `t1` and `t0` forwarded?
2. **Explain.** Why can a store's *data* operand (`rs2`) sometimes be forwarded later than its address operand? (This pipeline does not exploit it.)
3. **Predict.** Without forwarding, how many stall cycles does a dependent pair of ALU instructions cost in this pipeline, and why not three?

## Unit 6.4 — Control hazards: branches and the flush

**One-line summary:** The pipeline fetches the next instruction before it knows whether the previous one is a taken branch; this design predicts "not taken" by fetching sequentially, resolves branches in EX, and flushes the two wrong-path instructions when a branch is taken or a jump executes — a two-cycle penalty that is the largest single loss in this pipeline.

When a branch reaches EX and is taken, the two instructions behind it (in ID and IF) were fetched from the fall-through path. They have not written anything yet — writes happen only in MEM and WB — so cancelling them is easy: replace them with bubbles, and fetch from the target in the next cycle. The cost is two cycles per taken branch and per jump.

In `tests.c` there were 15,622 redirects: 31,244 lost cycles — 12% of the run, and 96% of all lost cycles. Options for reducing it, all used in real processors:

- **Resolve earlier.** Comparing registers and computing the target in ID cuts the penalty to one cycle, but puts a comparator and an adder in ID, after the register read, and creates new hazards (a branch that depends on the instruction just before it needs forwarding *into ID*).
- **Delay slots.** MIPS defined the instruction after a branch to execute anyway (Chapter 4), turning one wasted cycle into useful work when the compiler can fill it.
- **Prediction.** Guess the outcome *and the target* at fetch time, from history. A good predictor removes most of the penalty — at the cost of a flush when it is wrong. Chapter 8 is about this.

### Deep & Tricky

**DT-21. A flushed instruction must have *no* architectural effect — no register write, no memory write, no exception, no system call. In a five-stage pipeline that resolves branches in EX this is automatic, because the wrong-path instructions are still in IF and ID; in deeper pipelines and out-of-order processors, wrong-path instructions reach execution, and the hardware must suppress or undo all of their effects — which is exactly the property Spectre (Chapter 0) showed does not extend to microarchitectural state such as the cache.** *(RISC-V Unprivileged ISA Specification, §1.6: exceptions and the architectural state must be as if instructions executed in order; Kocher et al., 2019)* In `pipe_cpu.v`, an `ecall` fetched after a taken branch is flushed before it reaches WB, which is the only stage that acts on it — moving the `ecall` handling to an earlier stage would print output for system calls that never architecturally happened.

### Mastery check

1. **Calculate.** A program has 20% branches, of which 60% are taken, and 5% jumps. What CPI does this pipeline achieve from control hazards alone?
2. **Explain.** Why is "predict not taken" free in this pipeline, while "predict taken" is not?

## Topic 6C — Verification and performance

## Unit 6.5 — Watching and torturing the pipeline

**One-line summary:** Pipelines fail at hazard corner cases — a load feeding a branch, a jump through a register written one instruction earlier, a store of a just-loaded value — so they are verified with a directed "torture test" that packs these cases together and compares the result with a golden model, and debugged with a cycle-by-cycle view of which instruction is in which stage.

```asm
# file: torture/hazards.s
# Back-to-back dependences of every kind; the checksum must match rvsim's.
    .text
    .globl _start
_start:
    lla  t6, data
    li   s0, 0                  # checksum
    li   t0, 5
    addi t1, t0, 1              # EX->EX forwarding (distance 1)
    add  t2, t1, t0             # distance 1 and 2 at once
    sub  t3, t2, t1             # chain
    add  s0, s0, t3
    lw   t4, 0(t6)              # load ...
    add  t5, t4, t4             # ... used at once: load-use stall
    add  s0, s0, t5
    lw   t4, 4(t6)
    nop
    add  t5, t4, t2             # load, then use at distance 2: forwarded from WB
    add  s0, s0, t5
    sw   t5, 8(t6)              # store data forwarded (rs2)
    lw   a1, 8(t6)              # load what was just stored
    add  s0, s0, a1
    lw   a2, 0(t6)
    sw   a2, 12(t6)             # load-use into a store's data
    lw   a3, 12(t6)
    add  s0, s0, a3
    lw   a4, 0(t6)
    beq  a4, t0, skip1          # load-use into a branch (7 != 5: not taken)
    addi s0, s0, 100
skip1:
    li   a5, 3
loop:                           # a taken backward branch three times
    add  s0, s0, a5
    addi a5, a5, -1
    bnez a5, loop
    lla  a6, target
    jalr ra, 0(a6)              # jalr with a register written two instructions earlier
    addi s0, s0, 1000           # executed after target returns here through ra
    j    finish
target:
    addi s0, s0, 7
    ret                         # jalr through ra written by the jalr above
finish:
    add  s0, s0, x0
    addi x0, s0, 1              # writes to x0 are ignored ...
    add  s0, s0, x0             # ... and x0 still reads 0
    mv   a0, s0
    andi a0, a0, 255
    li   a7, 93
    ecall
    .data
data: .word 7, 11, 0, 0
```

```python
# file: torture/pipeview.py
"""Print which instruction occupies each pipeline stage, cycle by cycle, from pipe_cpu's +trace output."""
import re
import subprocess
import sys

elf, trace = sys.argv[1], sys.argv[2]
names = {}
dump = subprocess.run(["riscv64-linux-gnu-objdump", "-d", elf], capture_output=True, text=True).stdout
for line in dump.splitlines():
    m = re.match(r"\s+([0-9a-f]+):\s+[0-9a-f]{8}\s+(\S+)\s*(\S*)", line)
    if m:
        names[int(m.group(1), 16)] = (m.group(2) + " " + m.group(3).split(",")[0])[:12]

def show(pc):
    return "-" if pc == 0xFFFFFFFF else names.get(pc, f"{pc:x}")

print(f"{'cyc':>3} | {'IF':12} | {'ID':12} | {'EX':12} | {'MEM':12} | {'WB':12} | note")
for line in open(trace):
    if not line.startswith("T "):
        continue
    _, cyc, f, d, e, m, w, stall, redirect = line.split()
    note = ("stall" if stall == "1" else "") + (" redirect: flush IF, ID" if redirect == "1" else "")
    cols = [show(int(x, 16)) for x in (f, d, e, m, w)]
    print(f"{int(cyc):>3} | " + " | ".join(f"{c:12}" for c in cols) + f" | {note}")
```

```verilog
// file: torture/tb.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_cpu #(.HEX("hazards.hex"), .FORWARD(1)) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #1000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd torture && riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -nostdlib -static -T ../../ch05/sw/link.ld -Wl,--build-id=none -Wl,--no-warn-rwx-segments hazards.s -o hazards.elf && riscv64-linux-gnu-objcopy -O binary hazards.elf h.bin && python3 ../../ch05/sw/mkhex.py h.bin hazards.hex
hazards.hex: 46 words
$ cd torture && python3 ../../ch04/rvsim/rvsim.py hazards.elf; echo "rvsim exit status $?"; qemu-riscv32 ./hazards.elf; echo "qemu exit status $?"
[rvsim: 48 instructions, exit 159]
rvsim exit status 159
qemu exit status 159
$ cd torture && iverilog -g2012 -o sim ../pipe/pipe_cpu.v tb.v && vvp -n sim | grep "^\[pipe"
[pipe_cpu FORWARD=1: exit 159, 47 instructions, 67 cycles, CPI 1.426]
$ cd torture && vvp -n sim +trace | grep "^T " > trace.txt && python3 pipeview.py hazards.elf trace.txt | head -31
cyc | IF           | ID           | EX           | MEM          | WB           | note
  1 | li t6        | -            | -            | -            | -            | 
  2 | li s0        | li t6        | -            | -            | -            | 
  3 | li t0        | li s0        | li t6        | -            | -            | 
  4 | addi t1      | li t0        | li s0        | li t6        | -            | 
  5 | add t2       | addi t1      | li t0        | li s0        | li t6        | 
  6 | sub t3       | add t2       | addi t1      | li t0        | li s0        | 
  7 | add s0       | sub t3       | add t2       | addi t1      | li t0        | 
  8 | lw t4        | add s0       | sub t3       | add t2       | addi t1      | 
  9 | add t5       | lw t4        | add s0       | sub t3       | add t2       | 
 10 | add s0       | add t5       | lw t4        | add s0       | sub t3       | stall
 11 | add s0       | add t5       | -            | lw t4        | add s0       | 
 12 | lw t4        | add s0       | add t5       | -            | lw t4        | 
 13 | nop          | lw t4        | add s0       | add t5       | -            | 
 14 | add t5       | -            | lw t4        | add s0       | add t5       | 
 15 | add s0       | add t5       | -            | lw t4        | add s0       | 
 16 | sw t5        | add s0       | add t5       | -            | lw t4        | 
 17 | lw a1        | sw t5        | add s0       | add t5       | -            | 
 18 | add s0       | lw a1        | sw t5        | add s0       | add t5       | 
 19 | lw a2        | add s0       | lw a1        | sw t5        | add s0       | stall
 20 | lw a2        | add s0       | -            | lw a1        | sw t5        | 
 21 | sw a2        | lw a2        | add s0       | -            | lw a1        | 
 22 | lw a3        | sw a2        | lw a2        | add s0       | -            | stall
 23 | lw a3        | sw a2        | -            | lw a2        | add s0       | 
 24 | add s0       | lw a3        | sw a2        | -            | lw a2        | 
 25 | lw a4        | add s0       | lw a3        | sw a2        | -            | stall
 26 | lw a4        | add s0       | -            | lw a3        | sw a2        | 
 27 | beq a4       | lw a4        | add s0       | -            | lw a3        | 
 28 | addi s0      | beq a4       | lw a4        | add s0       | -            | stall
 29 | addi s0      | beq a4       | -            | lw a4        | add s0       | 
 30 | li a5        | addi s0      | beq a4       | -            | lw a4        | 
```

All three agree on the checksum 159. (The pipeline reports 47 instructions against `rvsim`'s 48 because the program contains one real `nop`, which the pipeline's statistics cannot tell from a bubble — it shows as `-` in the view too.) The view shows every mechanism of this chapter: in cycle 10, `add t5` is in ID and needs `t4`, which `lw t4` in EX will only produce after MEM — the hazard unit stalls, `add t5` stays in ID for a second cycle, and a bubble (`-`) enters EX. Cycles 19, 22, 25, and 28 show the same stall for a load feeding an `add`, a store's data, another `add`, and a branch. Back-to-back ALU dependences (cycles 6–9) flow without a single stall — that is forwarding.

### Mastery check

1. **Read.** In the pipeline view, in which cycle does `lw t4` (the first one) write the register file, and in which does `add t5` use the value?
2. **Design.** Add a case to `hazards.s` for a load whose result is used as the *base address* of the next load.

## Unit 6.6 — Three processors, compared

**One-line summary:** With the same memory size, synthesis shows the pipeline's longest stage is shorter than the single-cycle processor's whole path but longer than the multi-cycle design's — because forwarding multiplexers sit in front of the ALU — and the iron law then puts the pipeline first; its remaining losses are the branch flushes, which prediction attacks next.

Chapter 5 synthesised its processors with a 64-word memory, which flatters the single-cycle design: its critical path runs through *two* memory reads (instruction and data), and real memories are much larger. Here all three are synthesised with 4,096 words (16 KiB), with the memory still built from flip-flops and multiplexers (each synthesis takes a couple of minutes):

```console
$ mkdir -p compare && cd compare && for top in sc_cpu mc_cpu pipe_cpu; do d=$([ $top = sc_cpu ] && echo ../../ch05/single || ([ $top = mc_cpu ] && echo ../../ch05/multi || echo ../pipe)); yosys -p "read_verilog $d/$top.v; chparam -set MEMWORDS 4096 $top; synth -top $top -flatten; stat; ltp -noff" > $top.log 2>&1; echo "$top: $(grep 'Number of cells' $top.log | tail -1 | tr -s ' ') | $(grep -o 'length=[0-9]*' $top.log | tail -1)"; done
sc_cpu:  Number of cells: 540070 | length=52
mc_cpu:  Number of cells: 408064 | length=29
pipe_cpu:  Number of cells: 408361 | length=36
```

| Design | Cycles (`tests.c`) | CPI | Path (gates) | Time ∝ cycles × path |
| --- | --- | --- | --- | --- |
| Single-cycle | 222,847 | 1.00 | 52 | 11.6 million |
| Multi-cycle | 908,489 | 4.08 | 29 | 26.3 million |
| Pipelined | 255,469 | 1.15 | 36 | 9.2 million |

The pipeline wins — by 26% over the single-cycle design, not the 5× that five stages promise. The reasons are the ones Unit 6.1 listed. Its stages are unbalanced: Yosys's `ltp` report (in `pipe_cpu.log`) shows the longest path starts at the MEM/WB destination-register field, runs through the forwarding comparison and multiplexer (8 gates), and then through the 32-bit ALU to the EX/MEM register (28 gates) — the EX stage carries both. And the single-cycle path would be much longer with realistic memories: a real 16 KiB SRAM read takes a large fraction of a nanosecond, and the single-cycle design has two in series in every cycle, while the pipeline has one per stage.

### Deep & Tricky

**DT-22. Deeper pipelines raise the clock frequency only until the per-stage register overhead and the growing hazard penalties win: each extra stage adds a fixed setup + clock-to-Q delay, and lengthens the distance (in cycles) between a branch's fetch and its resolution, so mispredictions cost more. Studies of the trade-off put the optimum for general-purpose code at roughly 15–25 FO4 gate delays of logic per stage; the Pentium 4's 31-stage Prescott core went past it and was abandoned for shorter pipelines.** *(Hrishikesh et al., "The Optimal Logic Depth Per Pipeline Stage is 6 to 8 FO4 Inverter Delays", ISCA 2002; Hartstein & Puzak, "The Optimum Pipeline Depth for a Microprocessor", ISCA 2002; Intel's move from NetBurst to the Core microarchitecture, 2006)* The two ISCA 2002 papers disagree on the exact number because they weigh power and workloads differently — but both show a U-shaped curve with an optimum, not "deeper is faster".

### Mastery check

1. **Calculate.** If branch prediction removed 90% of this pipeline's flush cycles, what would the CPI and the relative time become?
2. **Explain.** Why is the forwarding logic on the pipeline's critical path, and how could a design move it off?

## Chapter 6 Project — Resolving branches in ID

### Requirements

Modify `pipe_cpu.v` (as a new module `pipe_id`) to resolve conditional branches and `jal` in the ID stage, reducing their penalty from two cycles to one: compare the two source registers and compute the target in ID, and flush only IF/ID on a redirect. Branches in ID read registers that may still be in flight, so: forward from EX/MEM and MEM/WB *into ID*, and stall a branch whose source is being computed by the instruction in EX (one cycle) or loaded by a load in EX (two cycles) or in MEM (one cycle). Keep `jalr` in EX. Verify with `tests.c` and the torture test against `rvsim`, and report the new CPI and the change in the synthesised path length. Decide whether the change is worth it.

### Reference solution

Rather than copying the 190-line processor, the solution is a script that applies each change to `pipe_cpu.v` as an exact text edit and writes `pipe_id.v` — every edit fails loudly if its target text is not found exactly once, so the derivation cannot silently drift from the original.

```python
# file: project/make_pipe_id.py
"""Derive pipe_id.v from pipe_cpu.v: resolve conditional branches and jal in ID (penalty 1 cycle)."""
import sys

src, dst = sys.argv[1], sys.argv[2]
v = open(src).read()

def edit(old, new):
    global v
    assert v.count(old) == 1, f"edit target not found exactly once: {old[:60]!r}"
    v = v.replace(old, new)

edit("module pipe_cpu", "module pipe_id")
edit('$display("[pipe_cpu FORWARD=%0d', '$display("[pipe_id FORWARD=%0d')

# 1. Branch comparison and target in ID, with forwarding from EX/MEM into ID.
edit("""    reg [31:0] e_instr, e_pc, e_a, e_b;                   // ID/EX pipeline register""",
"""    wire d_is_br = d_op == 7'b1100011, d_is_jal = d_op == 7'b1101111;
    wire [31:0] d_fa = (m_we && m_rd == d_rs1 && d_rs1 != 0) ? m_val : d_a;   // forward into ID
    wire [31:0] d_fb = (m_we && m_rd == d_rs2 && d_rs2 != 0) ? m_val : d_b;
    reg d_take;
    always @(*) case (d_instr[14:12])
        3'd0: d_take = d_fa == d_fb;   3'd1: d_take = d_fa != d_fb;
        3'd4: d_take = $signed(d_fa) < $signed(d_fb);   3'd5: d_take = $signed(d_fa) >= $signed(d_fb);
        3'd6: d_take = d_fa < d_fb;    3'd7: d_take = d_fa >= d_fb;
        default: d_take = 0;
    endcase
    wire [31:0] d_immB = {{19{d_instr[31]}}, d_instr[31], d_instr[7], d_instr[30:25], d_instr[11:8], 1'b0};
    wire [31:0] d_immJ = {{11{d_instr[31]}}, d_instr[31], d_instr[19:12], d_instr[20], d_instr[30:21], 1'b0};
    wire id_redirect = !stall && ((d_is_br && d_take) || d_is_jal);
    wire [31:0] id_target = d_pc + (d_is_jal ? d_immJ : d_immB);

    reg [31:0] e_instr, e_pc, e_a, e_b;                   // ID/EX pipeline register""")

# 2. Hazards: a branch in ID needs its operands *now*.
edit("""    assign stall = FORWARD ? (e_load && hit_e)            // load-use: the value exists only after MEM
                           : (hit_e || hit_m);           // no forwarding: wait until WB writes it""",
"""    wire br_hazard = d_is_br && (hit_e || (m_load && hit_m));   // value not yet available in ID
    assign stall = (e_load && hit_e) || br_hazard;""")

# 3. Redirects: jalr still from EX (flush 2); branches and jal from ID (flush IF/ID only).
edit("""    assign redirect = (is_br && take) || is_jal || is_jalr;
    assign redirect_pc = is_jalr ? ((fa + immI) & ~32'd1) : e_pc + (is_jal ? immJ : immB);""",
"""    assign redirect = is_jalr;
    assign redirect_pc = (fa + immI) & ~32'd1;""")
edit("""        else if (redirect) pc <= redirect_pc;
        else if (!stall) pc <= pc + 4;""",
"""        else if (redirect) pc <= redirect_pc;
        else if (id_redirect) pc <= id_target;
        else if (!stall) pc <= pc + 4;""")
edit("""        if (rst || redirect) begin d_instr <= NOP; d_pc <= 0; end   // flush the wrong-path fetch""",
"""        if (rst || redirect || id_redirect) begin d_instr <= NOP; d_pc <= 0; end""")
edit("""        if (redirect) flushes = flushes + 1;""",
"""        if (redirect) flushes = flushes + 1;
        if (id_redirect && !redirect) id_flushes = id_flushes + 1;""")
edit("""    integer cycles = 0, retired = 0, stalls = 0, flushes = 0, j;""",
"""    integer cycles = 0, retired = 0, stalls = 0, flushes = 0, id_flushes = 0, j;""")
edit("""                $display("[stall cycles %0d, redirects %0d (each flushes 2 instructions)]", stalls, flushes);""",
"""                $display("[stall cycles %0d, EX redirects %0d (flush 2), ID redirects %0d (flush 1)]",
                         stalls, flushes, id_flushes);""")
open(dst, "w").write(v)
print(f"wrote {dst}")
```

```verilog
// file: project/tb.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_id #(.HEX("../../ch05/sw/tests.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```verilog
// file: project/tb_torture.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_id #(.HEX("../torture/hazards.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #1000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd project && python3 make_pipe_id.py ../pipe/pipe_cpu.v pipe_id.v && iverilog -g2012 -o sim pipe_id.v tb.v && vvp -n sim | grep -v WARNING
wrote pipe_id.v
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[pipe_id FORWARD=1: exit 42, 222847 instructions, 247240 cycles, CPI 1.109]
[stall cycles 6721, EX redirects 2047 (flush 2), ID redirects 13576 (flush 1)]
pipe_id.v:193: $finish called at 2472405 (1s)
$ cd project && iverilog -g2012 -o sim_t pipe_id.v tb_torture.v && vvp -n sim_t | grep "^\[pipe"
[pipe_id FORWARD=1: exit 159, 47 instructions, 68 cycles, CPI 1.447]
$ cd project && yosys -p "read_verilog pipe_id.v; chparam -set MEMWORDS 4096 pipe_id; synth -top pipe_id -flatten; ltp -noff" 2>&1 | grep -o "length=[0-9]*" | tail -1
length=36
```

Both programs still produce the golden results. The CPI drops from 1.146 to 1.109: 13,576 branches and `jal`s now cost one cycle instead of two, and only the 2,047 `jalr`s (function returns) still pay two. But stalls grew from 1,376 to 6,721 cycles, because a branch that compares a value computed by the instruction right before it must now wait a cycle — a pattern compilers produce constantly (`addi a5, a5, -1; bnez a5, loop`). Net: 3.2% fewer cycles. Generic synthesis reports the same longest path (36 gates, still through forwarding into the ALU), so by this chapter's metric the change is a 3% win. In a real design the ID stage now holds the register read, a forwarding multiplexer, a 32-bit comparator, and the PC multiplexer in series — with real timing analysis it may well become the critical stage, which is why many designs keep branch resolution in EX and rely on prediction instead (Chapter 8).

### Review checklist

- What happens when a `jalr` in EX and a taken branch in ID redirect in the same cycle? Which target must win, and why? (The EX redirect: the branch in ID is younger, on the wrong path.)
- Does `br_hazard` cover a branch that depends on a load two instructions earlier (load in MEM)? Check the torture test's `lw a4; beq a4, ...` case.
- Would the design still work with `FORWARD = 0`? (No: the script replaced the stall logic; the parameter is now ignored. Say so in a comment, or restore it.)

## Answers to the Chapter 6 prediction questions

**Unit 6.1, Q1.** The cycle must fit the slowest stage plus overhead: 400 + 50 = 450 ps. Latency = 5 × 450 = 2,250 ps. A single-cycle design would need 300 + 400 + 350 + 400 + 250 = 1,700 ps per instruction.

**Unit 6.2, Q2.** CPI → 1 (plus the few fill cycles): without loads there are no load-use stalls, and without branches and jumps there are no flushes.

**Unit 6.3, Q1.** `t1` comes from EX/MEM (the `addi` is one instruction ahead); `t0` comes from MEM/WB (the `li` is two ahead).

**Unit 6.3, Q3.** Two cycles: the consumer reads the register file in ID in the same cycle that the producer writes it in WB (write-through), so it need not wait for a third.

**Unit 6.4, Q1.** (0.20 × 0.60 + 0.05) × 2 = 0.34 extra cycles per instruction: CPI 1.34.

**Unit 6.4, Q2.** Fetching sequentially needs nothing extra — PC + 4 is already computed. Predicting "taken" needs the target address at fetch time, before the instruction has even been decoded — which requires a branch target buffer (Chapter 8).

**Unit 6.6, Q1.** Flush cycles 31,244 → 3,124: cycles ≈ 227,349, CPI ≈ 1.02; time ∝ 227,349 × 36 ≈ 8.2 million, about 11% less than now.

# Chapter 7 — Precise Exceptions and Out-of-Order Execution

*Goal: understand how a processor can execute instructions in a different order from the program — whenever their operands are ready — and still behave, for software, exactly like the sequential Von Neumann machine of Chapter 4, including when an instruction faults. This chapter starts from precise exceptions, builds the reorder buffer that guarantees them, adds register renaming and dataflow scheduling, and measures with a trace-driven model how much instruction-level parallelism a real program offers and what limits it.*

*Accompanies the lectures "Precise Exceptions", "Out-of-Order Execution", and "Out-of-Order Execution and Data Flow".*

**What you will be able to do after this chapter**

- Define a precise exception and explain why pipelines with multi-cycle units break it unless they take care.
- Explain the reorder buffer (ROB): out-of-order completion, in-order commit, and flushing on an exception.
- Explain how register renaming removes write-after-read and write-after-write dependences, and measure what that is worth.
- Describe dataflow scheduling — reservation stations, wake-up, select — and the role of the instruction window.
- Use a trace-driven model to measure the instruction-level parallelism of a program and how branches and cache misses limit it.

**How this chapter's code is organised.** `model/` holds a tracer that runs a program on Chapter 4's `rvsim` and records what each instruction reads and writes, and a cycle-level model of an out-of-order core driven by that trace; `trap/` holds a real exception demonstrated under QEMU; `project/` adds a cache to the model.

## Topic 7A — Precise exceptions

## Unit 7.1 — Exceptions, interrupts, and what "precise" means

**One-line summary:** An exception (a fault caused by an instruction) or an interrupt (an external event) must stop the program at a single instruction boundary — every older instruction has completed, no younger instruction has changed anything, and the faulting instruction's address is saved — so that a handler can fix the cause and resume, or report exactly where the program failed.

Exceptions are caused by an instruction: an illegal instruction, a page fault on a load, a misaligned access, a division by zero on ISAs that trap on it (RISC-V does not), a system call. Interrupts come from outside: a timer, a network card. Either way, the processor must transfer control to a handler, and the handler must see a consistent state. An exception is **precise** if:

1. All instructions older than the faulting one have completed and their effects are visible.
2. No instruction younger than it has changed architectural state (registers, memory, PC).
3. The PC of the faulting instruction is saved (in RISC-V, in the `mepc` or `sepc` CSR; the cause goes to `mcause`/`scause`, and the processor jumps to the address in `mtvec`/`stvec`).

Precision is what makes demand paging (Chapter 11) possible: the OS loads the missing page and re-executes the same load as if nothing had happened. It is also what makes debugging possible.

On Linux, the kernel turns a hardware exception in user mode into a **signal**. The program below installs a handler for `SIGILL` and `SIGSEGV`, stores to a variable, executes an all-zeros word (which RISC-V defines as an illegal instruction), and would store to a second variable after it:

```c
// file: trap/trap.c
#define _GNU_SOURCE
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ucontext.h>

static volatile int before = 0, after = 0;

static void handler(int sig, siginfo_t *info, void *uc_) {
    ucontext_t *uc = uc_;
    unsigned long pc = uc->uc_mcontext.__gregs[REG_PC];
    printf("signal %s at pc %#lx, si_addr %p\n", sig == SIGILL ? "SIGILL" : "SIGSEGV", pc, info->si_addr);
    printf("before = %d (older instruction completed), after = %d (younger instruction did not)\n", before, after);
    exit(0);
}

int main(void) {
    struct sigaction sa;
    memset(&sa, 0, sizeof sa);
    sa.sa_sigaction = handler;
    sa.sa_flags = SA_SIGINFO;
    sigaction(SIGILL, &sa, NULL);
    sigaction(SIGSEGV, &sa, NULL);
    extern char faulting_instruction[];
    printf("the faulting instruction is at %p\n", (void *)faulting_instruction);
    __asm__ volatile(
        "li t0, 1\n"
        "sw t0, %0\n"                      /* before = 1 : older than the fault */
        ".globl faulting_instruction\n"
        "faulting_instruction:\n"
        ".word 0x00000000\n"               /* an illegal instruction (all zeros) */
        "sw t0, %1\n"                      /* after = 1  : younger: must not happen */
        : "=m"(before), "=m"(after) : : "t0");
    return 1;
}
```

```console
$ cd trap && riscv64-linux-gnu-gcc -O2 -static trap.c -o trap && qemu-riscv64 ./trap
the faulting instruction is at 0x104fa
signal SIGILL at pc 0x104fa, si_addr 0x104fa
before = 1 (older instruction completed), after = 0 (younger instruction did not)
```

(This example uses 64-bit RISC-V Linux with the C library, because it needs `sigaction`.) The signal reports the exact address of the illegal word; the store before it happened, the store after it did not. That is a precise exception, delivered all the way up to a C program — and the property every mechanism in this chapter must preserve.

### The problem pipelines create

In Chapter 6's pipeline every instruction takes the same path and writes registers only in WB, in program order, so precision is free. Give the pipeline a 20-cycle divider and a 1-cycle adder, and a younger `add` finishes and writes its register long before an older `div` discovers it must trap. Four solutions exist: wait (allow only one long instruction in flight — slow); a **history buffer** (write early but keep old values to undo); a **future file** (keep a speculative and an architectural register file); and the **reorder buffer**, which all modern out-of-order processors use.

### Mastery check

1. **Explain.** Why must a page-fault handler be able to re-execute the faulting load, and what would go wrong if a younger load had already updated a register?
2. **Explain.** Is a timer interrupt "precise" with respect to a particular instruction? What does the processor guarantee?

## Unit 7.2 — The reorder buffer: complete out of order, commit in order

**One-line summary:** The reorder buffer is a queue of in-flight instructions in program order; instructions may execute and complete in any order, writing their results into their ROB entry, but they update architectural state only when they reach the head of the queue (**commit**, or **retire**) — so an exception at the head simply discards everything behind it.

Each ROB entry holds an instruction's destination register, its result when complete, and its exception status. Every cycle, the processor commits the instruction at the head *if it has completed*: writes its result to the architectural register file (or, for a store, sends it to memory) and removes it. If the head instruction completed *with an exception*, the processor flushes the whole ROB — all younger instructions, whether finished or not — and jumps to the handler. Because nothing younger ever touched architectural state, the exception is precise by construction.

The model in this chapter records each instruction's register reads and writes by running the program on `rvsim`:

```python
# file: model/trace.py
"""Run a program on rvsim and record, per instruction, what an out-of-order model needs."""
import os
import pickle
import sys
sys.path.insert(0, os.path.join(os.path.dirname(os.path.abspath(__file__)), "../../ch04/rvsim"))
from rvsim import CPU

def classify(w):
    op = w & 0x7F
    return {0x03: "load", 0x23: "store", 0x63: "branch", 0x6F: "jump", 0x67: "jump",
            0x33: "alu", 0x13: "alu", 0x37: "alu", 0x17: "alu"}.get(op, "other")

def record(elf):
    cpu = CPU()
    cpu.load_elf(elf)
    out = []
    while cpu.exit_code is None:
        pc = cpu.pc
        w = cpu.ld(pc, 4, False)
        kind = classify(w)
        rd, rs1, rs2 = w >> 7 & 31, w >> 15 & 31, w >> 20 & 31
        op = w & 0x7F
        srcs = []
        if op not in (0x37, 0x17, 0x6F):                 # lui, auipc, jal read no register
            srcs.append(rs1)
        if op in (0x33, 0x23, 0x63):                     # R-type, store, branch read rs2
            srcs.append(rs2)
        dst = rd if op not in (0x23, 0x63, 0x73) else 0
        addr = None
        if kind in ("load", "store"):
            imm = (w >> 20) if kind == "load" else ((w >> 25) << 5 | (w >> 7 & 31))
            imm = imm - 4096 if imm & 0x800 else imm
            addr = (cpu.x[rs1] + imm) & 0xFFFFFFFF
        cpu.step()
        taken = cpu.pc != pc + 4
        out.append((pc, kind, dst, tuple(s for s in srcs if s), addr, taken, cpu.pc))
    return out

if __name__ == "__main__":
    t = record(sys.argv[1])
    pickle.dump(t, open(sys.argv[2], "wb"))
    kinds = {}
    for x in t:
        kinds[x[1]] = kinds.get(x[1], 0) + 1
    print(f"{len(t)} instructions:", ", ".join(f"{k} {v}" for k, v in sorted(kinds.items())))
```

and simulates a core cycle by cycle: **commit** up to `width` completed instructions from the ROB head, **issue** up to `width` instructions whose operands are ready, and **fetch** up to `width` instructions into the ROB while it has room. Each instruction records which older instructions produce its operands; it may issue when they have completed.

```python
# file: model/ooo.py
"""A trace-driven out-of-order core model: fetch, rename/dispatch into a reorder buffer,
dataflow issue to functional units, out-of-order completion, in-order commit."""
import pickle
import sys
from collections import deque

LAT = {"alu": 1, "branch": 1, "jump": 1, "load": 3, "store": 1, "other": 1}

def simulate(trace, width=1, rob_size=1, in_order=False, branches="perfect", penalty=0,
             fault_at=None, rename=True, load_latency=None):
    """branches: "perfect" (never stall), "stall" (fetch waits for every branch and jump to execute),
    or a predictor object with .predict_and_update(pc, taken, target) -> bool (True = mispredicted)."""
    n = len(trace)
    rob = deque()                      # entries: [index, ready_cycle or None, done_cycle or None]
    last_writer = {}                   # arch register -> trace index of its youngest in-flight writer
    done_at = {}                       # trace index -> cycle its result is available
    last_store = {}                    # address -> index of youngest store to it
    readers = {}                       # arch register -> indices that read it since its last write
    fetch_i, committed, cycle = 0, 0, 0
    fetch_blocked_until = 0            # cycle at which fetch may resume after a branch
    completed_not_committed_at_fault = None
    while committed < n:
        cycle += 1
        # 1. commit: in order, up to `width` per cycle, only finished instructions
        for _ in range(width):
            if not rob:
                break
            i, _, done = rob[0]
            if done is None or done > cycle:
                break
            if fault_at is not None and i == fault_at:
                completed_not_committed_at_fault = sum(1 for e in rob if e[2] is not None and e[2] <= cycle)
                return {"cycles": cycle, "committed": committed, "fault_at": i,
                        "rob_occupancy": len(rob),
                        "younger_already_executed": completed_not_committed_at_fault - 1}
            rob.popleft()
            committed += 1
        # 2. issue: oldest-first, up to `width` instructions whose operands are ready
        issued = 0
        for e in rob:
            if issued == width:
                break
            if e[2] is not None:
                continue
            i = e[0]
            stop_here = in_order                               # in-order issue: never pass a waiting instruction
            _, kind, dst, srcs, addr, taken, nxt = trace[i]
            deps = [done_at.get(p) for p in e[1]]
            if any(d is None or d > cycle for d in deps):
                if stop_here:
                    break
                continue                                   # an operand is not ready yet
            lat = load_latency(addr) if (kind == "load" and load_latency) else LAT[kind]
            e[2] = cycle + lat
            done_at[i] = e[2]
            issued += 1
            if kind in ("branch", "jump") and fetch_blocked_until == ("wait", i):
                fetch_blocked_until = e[2] + penalty
        # 3. fetch + rename + dispatch into the ROB
        for _ in range(width):
            if fetch_i >= n or len(rob) >= rob_size:
                break
            if isinstance(fetch_blocked_until, tuple) or cycle < fetch_blocked_until:
                break
            pc, kind, dst, srcs, addr, taken, nxt = trace[fetch_i]
            producers = [last_writer[s] for s in srcs if s in last_writer]   # true (RAW) dependences
            if not rename and dst:                                          # without renaming, the one
                if dst in last_writer:                                      # register name is shared:
                    producers.append(last_writer[dst])                      # WAW: wait for the old writer
                producers += readers.get(dst, [])                           # WAR: wait for older readers
            for s in srcs:
                readers.setdefault(s, []).append(fetch_i)
            if kind == "load" and addr in last_store:
                producers.append(last_store[addr])                         # memory dependence
            rob.append([fetch_i, producers, None])
            if dst:
                last_writer[dst] = fetch_i
                readers[dst] = []
            if kind == "store":
                last_store[addr] = fetch_i
            if kind in ("branch", "jump"):
                if branches == "perfect":
                    mispredicted = False
                elif branches == "stall":
                    mispredicted = True
                else:
                    mispredicted = branches.predict_and_update(pc, taken, nxt)
                if mispredicted:
                    fetch_blocked_until = ("wait", fetch_i)      # resume after it executes
                    fetch_i += 1
                    break
            fetch_i += 1
    return {"cycles": cycle, "committed": committed, "ipc": committed / cycle}

class Bimodal:
    """2-bit saturating counters indexed by PC; jumps and targets assumed predicted correctly."""
    def __init__(self, bits=10):
        self.table = [2] * (1 << bits)
        self.mask = (1 << bits) - 1
        self.misses = 0
    def predict_and_update(self, pc, taken, target):
        i = (pc >> 2) & self.mask
        guess = self.table[i] >= 2
        self.table[i] = min(3, self.table[i] + 1) if taken else max(0, self.table[i] - 1)
        miss = guess != taken
        self.misses += miss
        return miss

if __name__ == "__main__":
    trace = pickle.load(open(sys.argv[1], "rb"))
    print(f"{'machine':42} {'branches':>9} {'cycles':>9} {'IPC':>6}")
    runs = [("in-order pipeline, 1-wide", dict(width=1, rob_size=8, in_order=True)),
            ("out-of-order, 1-wide, ROB 8", dict(width=1, rob_size=8)),
            ("in-order, 4-wide, ROB 32", dict(width=4, rob_size=32, in_order=True)),
            ("out-of-order, 4-wide, ROB 32", dict(width=4, rob_size=32)),
            ("out-of-order, 4-wide, ROB 128", dict(width=4, rob_size=128)),
            ("out-of-order, 8-wide, ROB 256", dict(width=8, rob_size=256))]
    for name, cfg in runs:
        for br in ("perfect", "stall", "bimodal"):
            b = Bimodal() if br == "bimodal" else br
            res = simulate(trace, branches=b, penalty=10 if br == "bimodal" else 0, **cfg)
            print(f"{name:42} {br:>9} {res['cycles']:9d} {res['ipc']:6.2f}")
```

```python
# file: model/fault.py
"""Inject a fault into one instruction and show what the ROB guarantees at that moment."""
import pickle
import sys
from ooo import simulate

t = pickle.load(open(sys.argv[1], "rb"))
r = simulate(t, width=4, rob_size=128, fault_at=100000)
print(f"fault at instruction {r['fault_at']}: committed {r['committed']}, ROB holds {r['rob_occupancy']},"
      f" younger instructions already executed: {r['younger_already_executed']}")
```

```console
$ cd model && python3 trace.py ../../ch05/sw/tests.elf tests.trace
222847 instructions: alu 110596, branch 20430, jump 6078, load 43690, other 43, store 42010
$ cd model && python3 fault.py tests.trace
fault at instruction 100000: committed 100000, ROB holds 40, younger instructions already executed: 20
```

The model injects a fault into instruction 100,000. When it reaches the head of the ROB, exactly 100,000 instructions have committed — every older one, and no younger one — although 20 younger instructions had *already executed*. Their results die in the ROB. Without the ROB, those 20 would have written their registers out of order, and the state at the exception would match no point in the sequential program.

### Mastery check

1. **Explain.** Why can a store not write memory when it completes, but only when it commits?
2. **Predict.** In a 4-wide core with a 128-entry ROB, how many instructions can be "in flight" at most, and why does a long-latency instruction at the head limit that?

## Topic 7B — Out-of-order execution

## Unit 7.3 — Register renaming

**One-line summary:** Only read-after-write dependences carry data; write-after-read and write-after-write "dependences" exist only because two instructions happen to use the same register *name*, and renaming each destination to a fresh physical register removes them, so instructions wait only for the values they actually need.

Three kinds of register dependence between an older instruction A and a younger B:

| Kind | Example | Real? |
| --- | --- | --- |
| RAW (true) | A: `add t0, …`  B: `sub …, t0, …` | Yes: B needs A's value |
| WAR (anti) | A: `sub …, t0, …`  B: `add t0, …` | No: B must not overwrite `t0` before A reads it — only because they share a name |
| WAW (output) | A: `add t0, …`  B: `lw t0, …` | No: the later write must win — only because they share a name |

With 32 architectural registers and loops that reuse them every iteration, WAR and WAW conflicts are everywhere. **Renaming** gives each new value its own physical register: a **register alias table** (RAT) maps each architectural register to the physical register holding its newest value; each instruction's sources are looked up in the RAT, and its destination gets a free physical register. Two writes to `t0` become writes to, say, `p37` and `p52`, and neither waits for the other. Modern cores have 200–300 physical registers for 32 architectural ones.

In the model, renaming is the default: each source is linked only to its youngest producer (`last_writer`). With `rename=False`, an instruction also waits for the previous writer of its destination (WAW) and for every older instruction that reads it (WAR):

```python
# file: model/rename.py
import pickle
import sys
from ooo import simulate

t = pickle.load(open(sys.argv[1], "rb"))
for on in (True, False):
    ipc = simulate(t, width=4, rob_size=128, rename=on)["ipc"]
    print(f"4-wide, ROB 128, renaming {'on ' if on else 'off'}: IPC {ipc:.2f}")
```

```console
$ cd model && python3 rename.py tests.trace
4-wide, ROB 128, renaming on : IPC 3.94
4-wide, ROB 128, renaming off: IPC 3.20
```

Renaming is worth about 23% here (IPC 3.94 against 3.20), with perfect branch prediction — and much more in floating-point loops, where the same few registers are rewritten every iteration.

### Mastery check

1. **Classify.** `lw t0, 0(a0)`; `add t1, t0, t2`; `addi t0, a0, 4`; `lw t2, 0(t0)` — list every dependence between these four instructions and its kind.
2. **Explain.** Why does the ROB need to remember, for each instruction, the physical register its destination *previously* mapped to?

## Unit 7.4 — Dataflow scheduling and the limits of ILP

**One-line summary:** An out-of-order core is a dataflow machine inside a window: instructions wait in reservation stations until their operands' producers broadcast results (**wake-up**), and each cycle the scheduler picks ready instructions for the functional units (**select**); the parallelism it finds is bounded by the window size, the machine width, true dependences — and above all by branches, because the window can only be filled with instructions from the predicted path.

Tomasulo's algorithm (IBM 360/91, 1967) is the ancestor: each reservation station holds an instruction with either its operand values or the *tags* of the instructions that will produce them; a common data bus broadcasts each result with its tag; matching stations capture the value. Modern cores do the same with physical register numbers as tags. The model above abstracts this into "issue when all producers have completed", oldest first.

The experiment runs `tests.c` on six machines, each with three branch models: **perfect** prediction (fetch never stops), **stall** (fetch waits for every branch and jump to execute — no prediction), and **bimodal** (the simple 2-bit predictor Chapter 8 starts with, with a 10-cycle penalty per misprediction):

```console
$ cd model && python3 ooo.py tests.trace
machine                                     branches    cycles    IPC
in-order pipeline, 1-wide                    perfect    227557   0.98
in-order pipeline, 1-wide                      stall    254063   0.88
in-order pipeline, 1-wide                    bimodal    288640   0.77
out-of-order, 1-wide, ROB 8                  perfect    222853   1.00
out-of-order, 1-wide, ROB 8                    stall    254063   0.88
out-of-order, 1-wide, ROB 8                  bimodal    286459   0.78
in-order, 4-wide, ROB 32                     perfect     79108   2.82
in-order, 4-wide, ROB 32                       stall    116765   1.91
in-order, 4-wide, ROB 32                     bimodal    142673   1.56
out-of-order, 4-wide, ROB 32                 perfect     56792   3.92
out-of-order, 4-wide, ROB 32                   stall    112076   1.99
out-of-order, 4-wide, ROB 32                 bimodal    126290   1.76
out-of-order, 4-wide, ROB 128                perfect     56494   3.94
out-of-order, 4-wide, ROB 128                  stall    112076   1.99
out-of-order, 4-wide, ROB 128                bimodal    126278   1.76
out-of-order, 8-wide, ROB 256                perfect     28970   7.69
out-of-order, 8-wide, ROB 256                  stall     98822   2.26
out-of-order, 8-wide, ROB 256                bimodal    100075   2.23
```

Read the table in three steps.

- **With perfect branches**, the dataflow parallelism is large: a 4-wide out-of-order core reaches IPC 3.9, an 8-wide one 7.7 — this program's dependence chains are short and many are independent (each `fib` call is a separate subtree). An in-order core of the same width gets 2.8: it stops at the first instruction that is not ready.
- **Without prediction**, every machine collapses towards 2: each branch or jump drains the front end, and a basic block in this program is only about 8 instructions long. The 8-wide core gains almost nothing over the 4-wide one.
- **With a simple predictor**, results land near the no-prediction line, because this program's branches are hard for a bimodal predictor (about 27% mispredicted — recursion and data-dependent comparisons) and each miss costs 10 cycles. Better predictors are worth more than wider machines; that is Chapter 8.

This is the classic result of limit studies (Wall, "Limits of Instruction-Level Parallelism", 1991): with oracle knowledge, programs offer tens of instructions per cycle; real branch prediction, memory disambiguation, and finite windows bring it down to a few.

### Deep & Tricky

**DT-23. An out-of-order core does not make a single dependent chain faster: its benefit is overlapping *independent* work — other chains, other loop iterations, cache misses behind each other. A linked-list traversal, where each load's address comes from the previous load, runs at the same speed on a 4-wide out-of-order core as on a simple in-order one; Unit 0.3's pointer-chasing benchmark relied on exactly this.** *(Hennessy & Patterson, "Computer Architecture: A Quantitative Approach", §3.10 "Limitations of ILP"; Mutlu et al., "Runahead Execution", HPCA 2003, on why long-latency dependent misses defeat the window)* Techniques that do help dependent chains change what is computed or fetched — value prediction, prefetching (Chapter 10), and runahead execution, which keeps executing past a cache miss *only* to generate prefetches.

### Mastery check

1. **Explain.** Why is the in-order 4-wide core so much worse than the out-of-order 4-wide core with perfect branches, but almost as good with no prediction?
2. **Calculate.** With 20% branches, a 4-wide machine, and fetch stopping at every branch, what is the maximum number of instructions fetched per cycle on average if each fetch group ends at a branch?

## Unit 7.5 — Loads, stores, and memory disambiguation

**One-line summary:** Register dependences are known at rename time, but memory dependences are not — a load cannot know whether an older store with a not-yet-computed address writes the same location — so out-of-order cores keep loads and stores in a **load/store queue**, forward data from older stores to younger loads, and speculate that loads do not depend on unknown stores, replaying them when that turns out wrong.

The model uses an *oracle* for memory: a load depends exactly on the youngest older store to the same address, which it knows from the trace. Real hardware has three options:

- **Conservative:** a load waits until every older store's address is known. Safe, and slow: it serialises loads behind stores.
- **Speculative with check:** a load executes as soon as its own address is ready; each store, when its address resolves, searches the load queue for younger loads to the same address that already executed, and if it finds one, flushes from that load on (a *memory-order violation*).
- **Predicted:** a **memory dependence predictor** (for example *store sets*, Chrysos & Emer, 1998) learns which loads have conflicted with which stores and makes only those wait.

When a load's address matches an older store still in the queue, the store's data is forwarded to the load directly (**store-to-load forwarding**), without going through the cache — which the model represents as the load depending on the store.

### Mastery check

1. **Explain.** Why is store-to-load forwarding necessary for correctness, not just speed, in an out-of-order core whose stores write the cache only at commit?
2. **Design.** What must happen when a store's address resolves and matches a younger load that has already executed?

## Chapter 7 Project — How much of a cache miss can out-of-order execution hide?

### Requirements

Give loads a realistic latency: model a direct-mapped data cache (a number of lines of 16 bytes; 3 cycles on a hit, 40 on a miss), pass its latency function to the core model, and compare an in-order core with out-of-order cores of different window sizes as the cache shrinks. Report the miss rate and the IPC as a fraction of the ideal (no-miss) IPC.

### Reference solution

```python
# file: project/cache_lat.py
"""Chapter 7 project: give loads a realistic latency from a small cache, and measure how much
an out-of-order core hides it compared with an in-order one."""
import os
import pickle
import sys
sys.path.insert(0, os.path.join(os.path.dirname(os.path.abspath(__file__)), "../model"))
from ooo import simulate, Bimodal

class DirectMapped:
    """A direct-mapped cache: `lines` lines of `line_bytes`; hit and miss latencies in cycles."""
    def __init__(self, lines=64, line_bytes=16, hit=3, miss=40):
        self.tags = [None] * lines
        self.lines, self.line_bytes, self.hit, self.miss = lines, line_bytes, hit, miss
        self.hits = self.misses = 0
    def latency(self, addr):
        block = addr // self.line_bytes
        idx, tag = block % self.lines, block // self.lines
        if self.tags[idx] == tag:
            self.hits += 1
            return self.hit
        self.tags[idx] = tag
        self.misses += 1
        return self.miss

trace = pickle.load(open(sys.argv[1], "rb"))
print(f"{'machine':30} {'cache':>18} {'IPC':>6} {'vs ideal':>9}")
for name, cfg in [("in-order, 1-wide", dict(width=1, rob_size=8, in_order=True)),
                  ("out-of-order, 4-wide, ROB 32", dict(width=4, rob_size=32)),
                  ("out-of-order, 4-wide, ROB 128", dict(width=4, rob_size=128))]:
    ideal = simulate(trace, **cfg)["ipc"]
    for lines in (64, 16, 4):
        c = DirectMapped(lines=lines)
        ipc = simulate(trace, load_latency=c.latency, **cfg)["ipc"]
        rate = c.misses / (c.hits + c.misses)
        print(f"{name:30} {lines:3d} lines, {rate:5.1%} miss {ipc:6.2f} {ipc / ideal:8.0%}")
```

```console
$ cd project && python3 cache_lat.py ../model/tests.trace
machine                                     cache    IPC  vs ideal
in-order, 1-wide                64 lines,  0.1% miss   0.98     100%
in-order, 1-wide                16 lines,  0.1% miss   0.97      99%
in-order, 1-wide                 4 lines, 18.1% miss   0.57      58%
out-of-order, 4-wide, ROB 32    64 lines,  0.1% miss   3.88      99%
out-of-order, 4-wide, ROB 32    16 lines,  0.1% miss   3.87      99%
out-of-order, 4-wide, ROB 32     4 lines, 18.1% miss   1.66      42%
out-of-order, 4-wide, ROB 128   64 lines,  0.1% miss   3.93     100%
out-of-order, 4-wide, ROB 128   16 lines,  0.1% miss   3.93     100%
out-of-order, 4-wide, ROB 128    4 lines, 23.3% miss   2.23      56%
```

`tests.c` has a tiny working set: with 16 or more lines (256 bytes) almost every load hits, and every machine runs at its ideal speed. With 4 lines the miss rate jumps to about 20% and every machine loses roughly half of its speed. The out-of-order cores keep far higher *absolute* IPC (1.7–2.2 against 0.57), but they hide no larger a *fraction* of the misses, because most of this program's misses are on dependence chains — the reloads of `ra` and saved registers in recursive `fib` feed the very next return — exactly DT-23's case. Notice also that the miss rate itself depends on the machine (18% or 23% with 4 lines): an out-of-order core accesses the cache in a different order, which changes which lines evict which. Chapter 10 builds caches properly, with associativity and replacement policies, and measures them on larger programs.

### Review checklist

- The cache is accessed when a load *issues*. In a real core, could a load that is later flushed still change the cache? (Yes — and that is Spectre's side channel, DT-21 and Chapter 0.)
- How would you add a limit of one load per cycle to the model? Which machines would it slow down?
- Why does the model need the trace's actual addresses, not just the dependence structure?

## Answers to the Chapter 7 prediction questions

**Unit 7.1, Q2.** No instruction caused it; the processor picks a boundary — typically it lets the instructions before some point commit, discards the rest, saves the PC of the first discarded one, and takes the interrupt there. The state is still precise: it corresponds to a point in the sequential program.

**Unit 7.2, Q2.** At most 128 (the ROB size). If the head instruction waits 100 cycles for a cache miss, the ROB fills behind it within about 32 cycles at 4 instructions per cycle, and fetch stops until the head commits.

**Unit 7.3, Q1.** RAW: (1)→(2) on `t0`; RAW: (3)→(4) on `t0`. WAR: (2)→(3) on `t0` (3 must not overwrite `t0` before 2 reads it); WAR: (2)→(4) on `t2`. WAW: (1)→(3) on `t0`. With renaming, only the two RAW dependences remain.

**Unit 7.4, Q2.** About 1/0.2 = 5 instructions between branches, so at most 4 per cycle (the width) — but each group of 4 ends early at a branch about 4 in 5 cycles; with fetch stopping until each branch resolves, the average falls well below 4, as the "stall" column shows.

**Unit 7.5, Q1.** Because the store's data is not in the cache until the store commits, a younger load to the same address that reads the cache would get the old value — a wrong result, not just a slow one.

# Chapter 8 — Branch Prediction

*Goal: keep the pipeline full across branches. Every taken branch or jump costs a pipelined processor several cycles unless it knows, at fetch time, that the instruction is a branch, whether it will be taken, and where it goes. This chapter measures how often each kind of predictor is right — static rules, 2-bit counters, local and global history, tournament and perceptron predictors — on programs chosen for their different branch behaviour, then predicts targets with a branch target buffer and a return address stack, and finally builds a predictor into Chapter 6's Verilog pipeline.*

*Accompanies the lectures "Branch Prediction I" and "Branch Prediction II".*

**What you will be able to do after this chapter**

- Compute the cost of branches from misprediction rate, branch frequency, and penalty (MPKI × penalty).
- Explain static prediction, 2-bit saturating counters, and why hysteresis helps loops.
- Explain local and global history predictors, which patterns each can learn, and why correlation between branches matters.
- Predict branch targets with a BTB and returns with a return address stack, and measure the difference.
- Add a BTB and a 2-bit predictor to a Verilog pipeline and measure the CPI improvement.

**How this chapter's code is organised.** `progs/` holds a benchmark program whose functions have deliberately different branch behaviour; `bp/` holds a predictor study that runs programs on `rvsim`, collects every branch, and replays the stream through each predictor; `project/` adds a predictor to the Chapter 6 pipeline.

## Topic 8A — Why predict, and what

## Unit 8.1 — The branch problem

**One-line summary:** A pipeline fetches every cycle, so it must decide the next PC before the current instruction is even decoded; each wrong decision throws away the work fetched since, and the cost — mispredictions per thousand instructions (MPKI) times the penalty in cycles — grows with pipeline depth and width.

Chapter 6's pipeline always fetched `pc + 4` and paid 2 cycles for every taken branch and jump: 12% of its time on `tests.c`. Deeper and wider machines pay much more: a modern core resolves a branch 15–20 cycles after fetching it and fetches 4–8 instructions per cycle, so one misprediction can cost 100 instruction slots. Chapter 7's model showed the result: with perfect prediction an 8-wide out-of-order core ran at IPC 7.7; with no prediction, 2.3.

The cost model is simple:

```
extra CPI from branches = (mispredictions per instruction) × (penalty in cycles)
                        = MPKI / 1000 × penalty
```

With 20 MPKI and a 15-cycle penalty, that is 0.3 extra cycles per instruction — for a 4-wide core whose ideal CPI is 0.25, more than doubling the run time. So the front end must answer three questions about the instruction at `pc`, in the same cycle it is fetched, before it is decoded:

1. **Is it a branch?** — Answered by the **branch target buffer** (BTB): a cache indexed by PC that remembers which addresses held taken branches.
2. **Will it be taken?** — The **direction predictor**, this chapter's main subject.
3. **Where to?** — The BTB's stored target, or a **return address stack** for returns.

### Mastery check

1. **Calculate.** A program has 20% branches; the predictor is 95% accurate; the penalty is 12 cycles. What is the MPKI and the extra CPI?
2. **Explain.** Why can the front end not wait for decode to learn that an instruction is a branch?

## Topic 8B — Direction prediction

## Unit 8.2 — A benchmark with different branch personalities

**One-line summary:** Branch behaviour depends on the program: loop branches are almost always taken, data-dependent branches on random data are unpredictable, some branches follow short repeating patterns, and some are decided by an earlier branch — so a predictor study needs programs that contain each kind, and must report results per kind.

```python
# file: progs/gen_data.py
"""Write data.h: 400 random bytes from Python's Mersenne Twister, which no branch predictor can learn."""
import random
random.seed(2026)
vals = [random.randrange(256) for _ in range(400)]
with open("data.h", "w") as f:
    f.write("// generated by gen_data.py\nstatic const unsigned char DATA[400] = {\n")
    for i in range(0, 400, 20):
        f.write("    " + ", ".join(map(str, vals[i:i + 20])) + ",\n")
    f.write("};\n")
print("data.h written: 400 bytes")
```

```c
// file: progs/bench.c
// Kernels with different branch behaviour; the predictor study attributes each branch to its function.
#include "rt.h"
#include "data.h"

#define NOINLINE __attribute__((noipa))   /* no inlining, no interprocedural shortcuts */
#define KEEP __asm__ volatile("")             /* keeps an if as a real branch (Chapter 0, Unit 0.2) */
static unsigned seed;
static unsigned rnd(void) { seed ^= seed << 13; seed ^= seed >> 17; seed ^= seed << 5; return seed; }   // xorshift32

#define N 400
static int a[N];

#define SUM_BIG(s) do {                       /* the branch from Chapter 0's sorted/random experiment */ \
        for (int rep = 0; rep < 5; rep++)                                                           \
            for (int i = 0; i < N; i++)                                                             \
                if (a[i] >= 128) { s += a[i]; KEEP; }                                               \
    } while (0)
static NOINLINE int threshold_random(void) {
    for (int i = 0; i < N; i++) a[i] = DATA[i];
    int s = 0;
    SUM_BIG(s);
    return s;
}
static NOINLINE int threshold_sorted(void) {
    for (int i = 0; i < N; i++) a[i] = DATA[i];
    for (int i = 1; i < N; i++) { int v = a[i], j = i - 1; while (j >= 0 && a[j] > v) { a[j + 1] = a[j]; j--; } a[j + 1] = v; }
    int s = 0;
    SUM_BIG(s);
    return s;
}
static NOINLINE int pattern(void) {           // a branch with a period-3 pattern: taken, taken, not taken
    int s = 0, m = 0;
    for (int i = 0; i < 6000; i++) {
        if (m != 2) { s += i; KEEP; } else s -= 1;
        if (++m == 3) m = 0;
    }
    return s;
}
static NOINLINE int correlated(void) {        // the second branch is decided by the first
    int s = 0;
    seed = 7;
    for (int i = 0; i < 4000; i++) {
        int x = rnd() & 1, y = x;
        __asm__ volatile("" : "+r"(y));       // hide that y == x, so GCC keeps two separate branches
        if (x) { s += 3; KEEP; }              // random: unpredictable on its own
        if (!y) { s += 5; KEEP; }             // predictable only from the previous branch's outcome
    }
    return s;
}
static NOINLINE int bsearch_(int key) {
    int lo = 0, hi = N - 1;
    while (lo <= hi) { int mid = (lo + hi) >> 1; if (a[mid] < key) lo = mid + 1; else if (a[mid] > key) hi = mid - 1; else return mid; }
    return -1;
}
static NOINLINE int search(void) {
    for (int i = 0; i < N; i++) a[i] = 3 * i;
    int found = 0;
    seed = 3;
    for (int k = 0; k < 1500; k++) found += bsearch_(rnd() & 1023) >= 0;
    return found;
}
static NOINLINE int depth(int n) {
    __asm__ volatile("" : "+r"(n));       // GCC can otherwise prove depth(n) == n and delete the recursion
    if (n == 0) return 0;
    return 1 + depth(n - 1);
}
static NOINLINE int calls(void) { int s = 0, d = 0; for (int i = 0; i < 200; i++) { s += depth(d); if (++d == 23) d = 0; } return s; }

int main(void) {
    puts_("threshold random "); putint(threshold_random()); puts_("\n");
    puts_("threshold sorted "); putint(threshold_sorted()); puts_("\n");
    puts_("pattern "); putint(pattern()); puts_("\n");
    puts_("correlated "); putint(correlated()); puts_("\n");
    puts_("search "); putint(search()); puts_("\n");
    puts_("calls "); putint(calls()); puts_("\n");
    return 0;
}
```

Each kernel is a function, so the study can attribute every branch to its kernel by address:

- **`threshold_random` / `threshold_sorted`** — Chapter 0's experiment: `a[i] >= 128` on random data, then the same data sorted. The data comes from Python's Mersenne Twister through `data.h`, so no predictor can find a formula behind it. The empty `asm volatile` (`KEEP`) forces GCC to keep each `if` as a real branch, as in Chapter 0.
- **`pattern`** — one branch that goes taken, taken, not taken, forever.
- **`correlated`** — the first branch is random; the second is the exact opposite of the first. The extra `asm` statement hides from GCC that `y == x`; otherwise it merges the two `if`s into one branch (it did, in the first version of this program).
- **`bsearch_`** — binary search: two data-dependent branches per step.
- **`depth`/`calls`** — GCC turned `depth`'s recursion into a loop, so these are loop branches with varying trip counts.

```console
$ cd progs && python3 gen_data.py && riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -O2 -nostdlib -ffreestanding -fno-builtin -static -T ../../ch05/sw/link.ld -Wl,--build-id=none -Wl,--no-warn-rwx-segments -I../../ch04/rvsim ../../ch04/rvsim/rt.c bench.c -o bench.elf && riscv64-linux-gnu-objcopy -O binary bench.elf b.bin && python3 ../../ch05/sw/mkhex.py b.bin bench.hex && qemu-riscv32 ./bench.elf
data.h written: 400 bytes
bench.hex: 493 words
threshold random 200325
threshold sorted 200325
pattern 11994000
correlated 16024
search 501
calls 2144
```

### Mastery check

1. **Explain.** Why would a branch study with only one aggregate number per predictor hide most of what is interesting?

## Unit 8.3 — Static prediction and 2-bit counters

**One-line summary:** Static rules (always taken; backward taken, forward not taken) cost nothing and predict loops well; a table of 2-bit saturating counters indexed by the branch address learns each branch's usual direction, and its hysteresis — two wrong guesses are needed to change its mind — keeps a loop's single exit from spoiling the next entry.

**Static.** "Always taken" exploits the fact that loops dominate. "Backward taken, forward not taken" (BTFN) refines it: a branch to a lower address is usually a loop's back edge. Compilers help static schemes by laying code out so that the likely path falls through, and GCC accepts hints (`__builtin_expect`, `[[likely]]` in C++20).

**Dynamic.** A **bimodal** predictor keeps a table of 2-bit counters indexed by low bits of the PC. Values 0 and 1 predict not taken, 2 and 3 taken; a taken branch increments (saturating at 3), a not-taken one decrements. A loop branch that is taken 99 times and not taken once sits at 3; the single not-taken outcome moves it to 2, which still predicts taken next time — so a 1-bit predictor would mispredict twice per loop execution (the exit and the next first iteration), a 2-bit one only once.

The study replays every conditional branch through each predictor:

```python
# file: bp/bp.py
"""Branch predictor study: run a program on rvsim, collect every control-transfer instruction,
and replay the stream through several direction predictors and target predictors."""
import os
import subprocess
import sys
from bisect import bisect_right
sys.path.insert(0, os.path.join(os.path.dirname(os.path.abspath(__file__)), "../../ch04/rvsim"))
from rvsim import CPU

def collect(elf):
    cpu = CPU()
    cpu.load_elf(elf)
    events = []                            # (pc, kind, taken, target)
    while cpu.exit_code is None:
        pc = cpu.pc
        w = cpu.ld(pc, 4, False)
        op, rd, rs1 = w & 0x7F, w >> 7 & 31, w >> 15 & 31
        cpu.step()
        if op == 0x63:
            events.append((pc, "cond", cpu.pc != pc + 4, cpu.pc))
        elif op == 0x6F:
            events.append((pc, "call" if rd == 1 else "jump", True, cpu.pc))
        elif op == 0x67:
            kind = "ret" if (rd == 0 and rs1 == 1) else ("icall" if rd == 1 else "ijump")
            events.append((pc, kind, True, cpu.pc))
    return events, cpu.count

def functions(elf):
    out = subprocess.run(["riscv64-linux-gnu-nm", "-n", elf], capture_output=True, text=True).stdout
    syms = [(int(a, 16), n) for a, t, n in (l.split() for l in out.splitlines() if len(l.split()) == 3) if t in "tT"]
    return [a for a, _ in syms], [n for _, n in syms]

# ---------------- direction predictors: predict(pc) -> bool, then update(pc, taken) ----------------
class Static:
    def __init__(self, rule): self.rule = rule
    def predict(self, pc, target): return self.rule(pc, target)
    def update(self, pc, taken): pass

class Bimodal:
    def __init__(self, bits=12):
        self.t, self.m = [1] * (1 << bits), (1 << bits) - 1      # 2-bit counters, start weakly not-taken
    def predict(self, pc, target): return self.t[(pc >> 2) & self.m] >= 2
    def update(self, pc, taken):
        i = (pc >> 2) & self.m
        self.t[i] = min(3, self.t[i] + 1) if taken else max(0, self.t[i] - 1)

class GShare:
    """2-bit counters indexed by PC XOR the global history of recent branch outcomes."""
    def __init__(self, bits=12, hist=12):
        self.t, self.m, self.h, self.hm = [1] * (1 << bits), (1 << bits) - 1, 0, (1 << hist) - 1
    def idx(self, pc): return ((pc >> 2) ^ self.h) & self.m
    def predict(self, pc, target): return self.t[self.idx(pc)] >= 2
    def update(self, pc, taken):
        i = self.idx(pc)
        self.t[i] = min(3, self.t[i] + 1) if taken else max(0, self.t[i] - 1)
        self.h = ((self.h << 1) | taken) & self.hm

class Local:
    """Two-level local predictor: each branch's own recent history selects a 2-bit counter."""
    def __init__(self, bits=10, hist=10):
        self.bht, self.bm = [0] * (1 << bits), (1 << bits) - 1
        self.pht, self.hm = [1] * (1 << hist), (1 << hist) - 1
    def predict(self, pc, target): return self.pht[self.bht[(pc >> 2) & self.bm]] >= 2
    def update(self, pc, taken):
        b = (pc >> 2) & self.bm
        h = self.bht[b]
        self.pht[h] = min(3, self.pht[h] + 1) if taken else max(0, self.pht[h] - 1)
        self.bht[b] = ((h << 1) | taken) & self.hm

class Tournament:
    """A chooser of 2-bit counters picks, per branch, whichever of two predictors has been right."""
    def __init__(self, a, b, bits=12):
        self.a, self.b, self.c, self.m = a, b, [2] * (1 << bits), (1 << bits) - 1
    def predict(self, pc, target):
        self.pa, self.pb = self.a.predict(pc, target), self.b.predict(pc, target)
        return self.pb if self.c[(pc >> 2) & self.m] >= 2 else self.pa
    def update(self, pc, taken):
        i = (pc >> 2) & self.m
        if self.pa != self.pb:
            self.c[i] = min(3, self.c[i] + 1) if self.pb == taken else max(0, self.c[i] - 1)
        self.a.update(pc, taken)
        self.b.update(pc, taken)

class Perceptron:
    """Jimenez & Lin (2001): one weight vector per branch; predict taken if bias + sum(w_i * h_i) >= 0,
    where h_i is +1/-1 for the i-th most recent global outcome. Learns linear functions of long histories."""
    def __init__(self, rows=256, hist=24):
        self.w = [[0] * (hist + 1) for _ in range(rows)]
        self.rows, self.hist = rows, hist
        self.h = [1] * hist
        self.theta = int(1.93 * hist + 14)                   # training threshold from the paper
    def predict(self, pc, target):
        w = self.w[(pc >> 2) % self.rows]
        self.y = w[0] + sum(wi * hi for wi, hi in zip(w[1:], self.h))
        return self.y >= 0
    def update(self, pc, taken):
        w = self.w[(pc >> 2) % self.rows]
        t = 1 if taken else -1
        if (self.y >= 0) != taken or abs(self.y) <= self.theta:
            w[0] = max(-128, min(127, w[0] + t))
            for i in range(self.hist):
                w[i + 1] = max(-128, min(127, w[i + 1] + t * self.h[i]))
        self.h = [t] + self.h[:-1]

def run(events, pred, names_at):
    starts, names = names_at
    total, miss, per = 0, 0, {}
    for pc, kind, taken, target in events:
        if kind != "cond":
            continue
        guess = pred.predict(pc, target)
        pred.update(pc, taken)
        fn = names[bisect_right(starts, pc) - 1]
        t, m = per.get(fn, (0, 0))
        per[fn] = (t + 1, m + (guess != taken))
        total += 1
        miss += guess != taken
    return total, miss, per

if __name__ == "__main__":
    elf = sys.argv[1]
    events, ninstr = collect(elf)
    syms = functions(elf)
    kernels = ["threshold_random", "threshold_sorted", "pattern", "correlated", "bsearch_", "depth"]
    preds = [("always taken", lambda: Static(lambda pc, t: True)),
             ("BTFN", lambda: Static(lambda pc, t: t < pc)),
             ("bimodal 4K", lambda: Bimodal(12)),
             ("local 1K x 10", lambda: Local(10, 10)),
             ("gshare 4K, h12", lambda: GShare(12, 12)),
             ("tournament", lambda: Tournament(Bimodal(12), GShare(12, 12))),
             ("perceptron h24", lambda: Perceptron(256, 24))]
    print(f"{ninstr} instructions, {sum(e[1] == 'cond' for e in events)} conditional branches")
    print(f"{'predictor':16} {'MPKI':>6} " + " ".join(f"{k[:10]:>10}" for k in kernels))
    for name, make in preds:
        total, miss, per = run(events, make(), syms)
        cells = []
        for k in kernels:
            t, m = per.get(k, (0, 0))
            cells.append(f"{100 * m / t:9.1f}%" if t else f"{'-':>10}")
        print(f"{name:16} {1000 * miss / ninstr:6.2f} " + " ".join(cells))
```

```console
$ cd bp && python3 bp.py ../progs/bench.elf
524092 instructions, 153981 conditional branches
predictor          MPKI threshold_ threshold_    pattern correlated   bsearch_      depth
always taken     132.14      24.0%      48.7%      60.0%      33.3%      41.7%      16.3%
BTFN              27.94      21.6%       1.0%      20.0%      33.3%      21.3%       0.4%
bimodal 4K        33.80      21.7%       0.5%      40.0%      34.2%      25.2%       9.3%
local 1K x 10     25.32       9.5%       0.5%       0.1%      33.4%      25.6%       7.7%
gshare 4K, h12    23.38      21.3%       0.6%       0.1%      17.5%      26.4%       9.2%
tournament        21.88      19.8%       0.5%       0.1%      17.6%      24.7%       8.4%
perceptron h24    21.93      22.2%       0.5%       0.1%      17.0%      24.7%       8.7%
```

Read the static and bimodal rows first:

- **`threshold_sorted`** is easy for everyone except "always taken" (the branch is not taken for the first half of the sorted array): ~1% wrong with BTFN or counters.
- **`threshold_random`** is about 22% wrong for BTFN and bimodal. That is the average over the data branch (about 50% wrong — a coin toss) and the loop branches (almost always right).
- **`pattern`**: bimodal gets 40% wrong — worse than BTFN's 20% — because with a taken, taken, not-taken pattern the counter oscillates between 2 and 3 and predicts taken, wrong one time in three, plus the counter's warm-up; counters cannot learn sequences.
- **`correlated`**: about a third wrong for every predictor without history: the first branch is random, and the second, looked at alone, is random too.

### Mastery check

1. **Trace.** A loop runs 4 iterations, is exited, and is entered again. Write the 2-bit counter's values and the predictions over the 10 branch executions, starting from 1.
2. **Explain.** Why is BTFN right on `threshold_sorted` almost always, but wrong on `pattern` 20% of the time?

## Unit 8.4 — History: local, global, and beyond

**One-line summary:** Branches are predictable from history — a branch's own recent outcomes (local history) capture repeating patterns, the recent outcomes of all branches (global history) capture correlation between branches — and modern predictors combine many history lengths, as tournament, perceptron, and TAGE predictors do.

- **Local (two-level) prediction** (Yeh & Patt, 1991): a table records each branch's last *h* outcomes; that history pattern selects a 2-bit counter. A branch that goes T, T, N, T, T, N … produces the histories `TTN`, `TNT`, `NTT` repeatedly, each always followed by the same outcome, and the counters learn it: `pattern` drops to 0.1% wrong.
- **Global history** (`gshare`, McFarling 1993): one shift register of the last *h* outcomes of *all* branches, XORed with the PC to index the counters. In `correlated`, when the second branch is predicted, the newest bit in the global history is the first branch's outcome — which decides it completely. gshare gets `correlated` to 17.5% wrong — the coin-toss first branch alone, averaged with a perfectly predicted second branch and the loop branch.
- **Tournament** (Alpha 21264): a chooser table learns, per branch, whether a local-style or a global predictor has been right more often. **Perceptron** predictors (Jiménez & Lin, 2001) learn a weight per history bit, which lets them use long histories (24 here) with tables that grow linearly rather than exponentially. **TAGE** (Seznec, 2006), the basis of most current high-end predictors, keeps several tables indexed by geometrically increasing history lengths (for example 5, 15, 44, 130 branches) with partial tags, and predicts with the longest history that has a matching entry.

The table shows each predictor's strengths and blind spots. No single scheme wins every row: local history wins `pattern` and, strikingly, `threshold_random`; global history wins `correlated`; the hybrids are best overall (lowest MPKI) without being best anywhere. And nothing predicts `bsearch_` much better than 25% — some branches are genuinely unpredictable.

### Deep & Tricky

**DT-24. A history-based predictor can *memorise* "random" data if the same data is processed repeatedly: in `threshold_random` the 400 random values are scanned five times, and with 10 bits of local history almost every position in the sequence has a unique context, so the predictor learns each outcome during the first pass and repeats it afterwards — 9.5% wrong where a coin toss is 50%. Benchmarks that loop over a fixed input overstate predictor accuracy for this reason, and predictor studies use inputs larger than the predictor's state or different data per pass.** *(Yeh & Patt, "Two-Level Adaptive Training Branch Prediction", MICRO 1991: a pattern history table learns any outcome that is a function of the last k outcomes; the memorisation effect is visible in this chapter's measurement)* gshare does not memorise it as well, because its 12 bits of global history are half loop-branch outcomes (always taken), leaving only about six data bits of context — too few for 400 positions. Change the repetition count from 5 to 1 in `SUM_BIG` and rerun: the local predictor then gets 15.8% of `threshold_random`'s branches wrong — exactly as many as BTFN, and no better than bimodal (16.1%) — its advantage was memory, not insight.

### Mastery check

1. **Explain.** Why does global history help `correlated` but not `pattern`, while local history does the opposite?
2. **Calculate.** How many 2-bit counters does a local predictor with 16-bit histories need for its pattern table? Why does that motivate perceptron predictors?

## Topic 8C — Targets

## Unit 8.5 — The branch target buffer and the return address stack

**One-line summary:** The BTB remembers, for each recently taken branch, its target, which is right for direct branches (their target never changes) and for indirect jumps that usually go to the same place; returns go to a different place depending on the caller, so they are predicted with a **return address stack** that pushes the return address on every call and pops it on every return.

```python
# file: bp/targets.py
"""Target prediction: a branch target buffer (last target per PC) versus a return address stack for returns."""
import sys
from bp import collect

def study(elf, btb_entries=64, ras_depth=8):
    events, n = collect(elf)
    btb = {}
    ras = []
    stats = {}
    def count(kind, ok):
        t, m = stats.get(kind, (0, 0))
        stats[kind] = (t + 1, m + (not ok))
    for pc, kind, taken, target in events:
        if not taken:
            continue                                       # a not-taken branch needs no target
        slot = (pc >> 2) % btb_entries
        entry = btb.get(slot)
        btb_ok = entry is not None and entry == (pc, target)
        if kind == "ret":
            ras_ok = bool(ras) and ras[-1] == target
            if ras:
                ras.pop()
            count("ret via BTB", btb_ok)
            count("ret via RAS", ras_ok)
        else:
            count(kind + " via BTB", btb_ok)
        if kind in ("call", "icall"):
            ras.append(pc + 4)                             # push the return address
            if len(ras) > ras_depth:
                ras.pop(0)                                 # overflow: the oldest entry is lost
        btb[slot] = (pc, target)
    return stats

if __name__ == "__main__":
    for elf in sys.argv[1:]:
        print(f"== {elf}")
        for kind, (t, m) in sorted(study(elf).items()):
            print(f"  {kind:16} {t:7d} taken transfers, {100 * m / t:5.1f}% wrong target")
```

```console
$ cd bp && python3 targets.py ../../ch05/sw/tests.elf ../progs/bench.elf
== ../../ch05/sw/tests.elf
  call via BTB        2047 taken transfers,   1.1% wrong target
  cond via BTB        9543 taken transfers,   5.9% wrong target
  jump via BTB        1984 taken transfers,   3.7% wrong target
  ret via BTB         2047 taken transfers,  25.4% wrong target
  ret via RAS         2047 taken transfers,   0.0% wrong target
== ../progs/bench.elf
  call via BTB        1725 taken transfers,   1.6% wrong target
  cond via BTB       84728 taken transfers,   0.0% wrong target
  jump via BTB        3057 taken transfers,   0.2% wrong target
  ret via BTB         1725 taken transfers,   1.6% wrong target
  ret via RAS         1725 taken transfers,   0.0% wrong target
```

For direct branches and calls, a 64-entry BTB is right almost always (the misses are first encounters and conflicts between branches mapping to the same entry). Returns are the interesting row. In `tests.c`, `putint` and `puts_` are called from many places, and recursive `fib` returns alternately to two call sites, so "the last target" is wrong a quarter of the time; the return address stack is never wrong, because calls and returns nest. In `bench.c`, almost every function is called from a single site, so the BTB alone does nearly as well — another reason to measure more than one program.

### Deep & Tricky

**DT-25. Returns are the most frequent indirect branches in most programs, and a BTB predicts them poorly whenever a function has more than one caller; a small return address stack (8–32 entries) predicts them almost perfectly, failing only when call depth overflows it or when the program does not return the way it called — `longjmp`, exceptions, coroutines, and stack-switching code.** *(Kaeli & Emma, "Branch History Table Prediction of Moving Target Branches due to Subroutine Returns", ISCA 1991)* The RAS is also an attack surface: speculative execution through a mispredicted return target ("ret2spec", Spectre-RSB, 2018) led to mitigations such as filling the RAS on context switches.

### Mastery check

1. **Predict.** With an 8-entry RAS, what happens to the return predictions of `fib(20)` as the recursion goes deeper than 8 calls and comes back up?
2. **Explain.** Why does a BTB entry need a tag, when a 2-bit counter table does not?

## Chapter 8 Project — A predictor in the Verilog pipeline

### Requirements

Add branch prediction to Chapter 6's five-stage pipeline: in IF, look up a 64-entry direct-mapped BTB (valid bit, tag, target, and whether the entry is a conditional branch) and a 256-entry table of 2-bit counters; fetch from the predicted target when the BTB hits and the branch is predicted taken (jumps are always predicted taken). Carry the predicted next PC down the pipeline; in EX, compare it with the actual next PC, redirect and flush on a mismatch, and train the BTB and counters. Verify that `tests.c`, the hazard torture test, and `bench.c` still produce their golden results, and compare the CPI with Chapter 6's pipeline.

### Reference solution

As in Chapter 6, the predictor is added by a script of exact edits to `pipe_cpu.v`:

```python
# file: project/make_pipe_bp.py
"""Derive pipe_bp.v from Chapter 6's pipe_cpu.v: predict branches and jumps in IF with a
64-entry branch target buffer and a 256-entry table of 2-bit counters, check in EX."""
import sys

src, dst = sys.argv[1], sys.argv[2]
v = open(src).read()

def edit(old, new):
    global v
    assert v.count(old) == 1, f"edit target not found exactly once: {old[:60]!r}"
    v = v.replace(old, new)

edit("module pipe_cpu", "module pipe_bp")
edit('$display("[pipe_cpu FORWARD=%0d', '$display("[pipe_bp FORWARD=%0d')

# 1. Predictor state and the prediction in IF.
edit("""    always @(posedge clk)
        if (rst) pc <= 0;
        else if (redirect) pc <= redirect_pc;
        else if (!stall) pc <= pc + 4;""",
"""    reg        btb_valid [0:63];
    reg [23:0] btb_tag   [0:63];
    reg [31:0] btb_target[0:63];
    reg        btb_cond  [0:63];                          // conditional branch (consult the counter)?
    reg [1:0]  bht       [0:255];
    integer p;
    initial for (p = 0; p < 256; p = p + 1) begin bht[p] = 2'd1; if (p < 64) btb_valid[p] = 0; end
    wire [5:0] bi = pc[7:2];
    wire btb_hit = btb_valid[bi] && btb_tag[bi] == pc[31:8];
    wire f_pred = btb_hit && (!btb_cond[bi] || bht[pc[9:2]][1]);   // predicted taken?
    wire [31:0] f_next = f_pred ? btb_target[bi] : pc + 4;
    always @(posedge clk)
        if (rst) pc <= 0;
        else if (redirect) pc <= redirect_pc;
        else if (!stall) pc <= f_next;""")

# 2. The prediction travels with the instruction.
edit("""    reg [31:0] d_instr, d_pc;                             // IF/ID pipeline register""",
"""    reg [31:0] d_instr, d_pc, d_pnext;                    // IF/ID pipeline register (+ predicted next PC)""")
edit("""        else if (!stall) begin d_instr <= mem[pc[31:2]]; d_pc <= pc; end""",
"""        else if (!stall) begin d_instr <= mem[pc[31:2]]; d_pc <= pc; d_pnext <= f_next; end""")
edit("""    reg [31:0] e_instr, e_pc, e_a, e_b;                   // ID/EX pipeline register""",
"""    reg [31:0] e_instr, e_pc, e_a, e_b, e_pnext;          // ID/EX pipeline register""")
edit("""            e_instr <= d_instr; e_pc <= d_pc; e_a <= d_a; e_b <= d_b;""",
"""            e_instr <= d_instr; e_pc <= d_pc; e_a <= d_a; e_b <= d_b; e_pnext <= d_pnext;""")

# 3. EX checks the prediction: redirect only when the predicted next PC was wrong.
edit("""    assign redirect = (is_br && take) || is_jal || is_jalr;
    assign redirect_pc = is_jalr ? ((fa + immI) & ~32'd1) : e_pc + (is_jal ? immJ : immB);""",
"""    wire ctrl = is_br || is_jal || is_jalr;
    wire [31:0] actual_next = is_jalr ? ((fa + immI) & ~32'd1) :
                              (is_jal || (is_br && take)) ? e_pc + (is_jal ? immJ : immB) : e_pc + 4;
    assign redirect = (e_instr != NOP) && (ctrl ? (actual_next != e_pnext) : (e_pnext != e_pc + 4));
    assign redirect_pc = actual_next;
    always @(posedge clk) if (!rst && ctrl) begin           // train the predictor
        if (is_br) bht[e_pc[9:2]] <= take ? (bht[e_pc[9:2]] == 3 ? 3 : bht[e_pc[9:2]] + 1)
                                          : (bht[e_pc[9:2]] == 0 ? 0 : bht[e_pc[9:2]] - 1);
        if (actual_next != e_pc + 4) begin                  // taken: remember the target
            btb_valid[e_pc[7:2]] <= 1; btb_tag[e_pc[7:2]] <= e_pc[31:8];
            btb_target[e_pc[7:2]] <= actual_next; btb_cond[e_pc[7:2]] <= is_br;
        end
    end""")
edit("""                $display("[stall cycles %0d, redirects %0d (each flushes 2 instructions)]", stalls, flushes);""",
"""                $display("[stall cycles %0d, mispredictions %0d (each flushes 2 instructions)]", stalls, flushes);""")
open(dst, "w").write(v)
print(f"wrote {dst}")
```

```verilog
// file: project/tb_tests.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_bp #(.HEX("../../ch05/sw/tests.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```verilog
// file: project/tb_bench.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_bp #(.HEX("../progs/bench.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```verilog
// file: project/tb_torture.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_bp #(.HEX("../../ch06/torture/hazards.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #1000000 $display("timeout");
        $finish;
    end
endmodule
```

```verilog
// file: project/tb_bench_nopred.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_cpu #(.HEX("../progs/bench.hex")) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #100000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd project && python3 make_pipe_bp.py ../../ch06/pipe/pipe_cpu.v pipe_bp.v && iverilog -g2012 -o sim_tests pipe_bp.v tb_tests.v && vvp -n sim_tests | grep -v -E "WARNING|finish"
wrote pipe_bp.v
fib(20) = 6765
sorted: -50 -7 -3 0 1 3 3 5 8 9 12 100
12345 * 6789 = 83810205
crc32 = 1095738169
sign tests: -25 50 -9
[pipe_bp FORWARD=1: exit 42, 222847 instructions, 237039 cycles, CPI 1.064]
[stall cycles 1376, mispredictions 6407 (each flushes 2 instructions)]
$ cd project && iverilog -g2012 -o sim_t pipe_bp.v tb_torture.v && vvp -n sim_t | grep "^\[pipe"
[pipe_bp FORWARD=1: exit 159, 47 instructions, 67 cycles, CPI 1.426]
$ cd project && iverilog -g2012 -o sim_b0 ../../ch06/pipe/pipe_cpu.v tb_bench_nopred.v && vvp -n sim_b0 | grep "^\[pipe"
[pipe_cpu FORWARD=1: exit 0, 524092 instructions, 768234 cycles, CPI 1.466]
$ cd project && iverilog -g2012 -o sim_b pipe_bp.v tb_bench.v && vvp -n sim_b | grep -v -E "WARNING|finish"
threshold random 200325
threshold sorted 200325
pattern 11994000
correlated 16024
search 501
calls 2144
[pipe_bp FORWARD=1: exit 0, 524092 instructions, 621316 cycles, CPI 1.186]
[stall cycles 61668, mispredictions 17777 (each flushes 2 instructions)]
```

All three programs produce their golden results. On `tests.c` the CPI falls from 1.146 to 1.064: redirects drop from 15,622 to 6,407. On `bench.c` — far more branches — from 1.466 to 1.186: 91,236 redirects become 17,777. Most remaining mispredictions are the genuinely unpredictable branches of `threshold_random` and `bsearch_`, and returns, which this BTB predicts as "the last target" (a RAS would fix those). The hazard torture test is too short for the predictor to warm up and runs exactly as before.

Note what the counters cost: about 4.5 KB of state (64 × (1 + 24 + 32 + 1) bits of BTB plus 256 × 2 bits), read in IF in parallel with the instruction memory — the predictor's lookup must fit in the fetch cycle, which is why real BTBs are often a cycle late for the *next* fetch and use a small, fast first-level predictor in front of a large, slow, accurate one.

### Review checklist

- When a non-branch instruction hits in the BTB (aliasing after code changes, or a tag collision), what does the EX check do? (It redirects to `pc + 4`: the last line of `redirect`.)
- Why is the predictor trained in EX, and what goes wrong if two branches in flight index the same counter?
- Add a 4-entry return address stack. Which program improves, and by how much?

## Answers to the Chapter 8 prediction questions

**Unit 8.1, Q1.** 0.2 branches per instruction × 5% = 0.01 mispredictions per instruction = 10 MPKI; extra CPI = 0.01 × 12 = 0.12.

**Unit 8.3, Q1.** Outcomes T, T, T, N, T, T, T, N, …: counter 1 (predict N, wrong) → 2 (T, right) → 3 → 3 (T, right) → exit N: predicted T, wrong, counter 2 → re-entry T: predicted T, right → 3 … — one misprediction per loop execution after warm-up.

**Unit 8.4, Q2.** 2¹⁶ = 65,536 counters (131,072 bits) per pattern table — exponential in history length; a perceptron needs one weight per history bit per row, growing linearly.

**Unit 8.5, Q1.** Calls deeper than 8 push the oldest entries off the stack; the first 8 returns on the way back up are predicted correctly, and the returns from the deeper frames below that find the stack empty or holding the wrong (overwritten) addresses — mispredicted — unless the predictor falls back to the BTB.

**Unit 8.5, Q2.** A BTB must not redirect fetch for an instruction that is not a taken branch at all; without a tag, any instruction whose PC maps to the entry would be treated as a branch to the stored target. A direction counter that is wrong only costs a misprediction for an instruction already known to be a branch.

# Chapter 9 — Superscalar, VLIW, SIMD, GPUs, and Systolic Arrays

*Goal: survey the other ways of doing more work per cycle. Out-of-order superscalar cores find parallelism dynamically among scalar instructions; VLIW machines leave that to the compiler; SIMD and vector instructions apply one instruction to many data elements; GPUs run thousands of threads in lock-step groups; and systolic arrays — the core of today's machine-learning accelerators — pump data through a grid of simple processing elements. Each is measured or simulated here: a VLIW scheduler, SIMD timings on the host, RISC-V vector code under QEMU, a SIMT warp model, and a systolic array in Verilog.*

*Accompanies the lectures "Superscalar Execution", "VLIW", "SIMD Processors and GPUs", and "Systolic Arrays".*

**What you will be able to do after this chapter**

- Classify parallel machines (Flynn's taxonomy) and say what kind of parallelism each exploits: instruction-level, data-level, thread-level.
- Explain the trade-off between superscalar (hardware scheduling) and VLIW (compiler scheduling), and schedule a loop into VLIW bundles.
- Measure SIMD speedups on a real CPU and explain why floating-point reductions do not vectorise by default.
- Write vector-length-agnostic RISC-V vector code.
- Explain SIMT execution and branch divergence on GPUs, and build and verify a systolic array.

## Topic 9A — Parallelism and its kinds

## Unit 9.1 — Three kinds of parallelism

**One-line summary:** Instruction-level parallelism (independent instructions in one thread), data-level parallelism (the same operation on many elements), and thread-level parallelism (independent instruction streams) are exploited by different machines — superscalar/VLIW, SIMD/vector/GPU, and multicore — and Flynn's taxonomy (SISD, SIMD, MISD, MIMD) classifies them by instruction and data streams.

| Flynn class | Meaning | Examples |
| --- | --- | --- |
| SISD | One instruction stream, one data stream | A scalar processor (even a superscalar one, as far as the ISA shows) |
| SIMD | One instruction, many data elements | x86 SSE/AVX, ARM NEON/SVE, RISC-V V, vector supercomputers, GPU warps |
| MISD | Many instructions, one data stream | Rare; systolic arrays are sometimes put here |
| MIMD | Many instruction streams | Multicore processors, clusters |

The kinds of parallelism are not exclusive: a modern CPU core is an out-of-order superscalar (ILP) with SIMD units (DLP) and two hardware threads (TLP), and there are many cores. What differs is *who finds* the parallelism: hardware at run time (out-of-order), the compiler (VLIW, auto-vectorisation), or the programmer (SIMD intrinsics, GPU kernels, threads).

### Mastery check

1. **Classify.** A GPU running 10,000 threads in groups of 32 that execute in lock-step: which Flynn categories apply, and at which level?

## Unit 9.2 — Superscalar and VLIW

**One-line summary:** A superscalar processor fetches several instructions per cycle and decides in hardware which can execute together; a VLIW (very long instruction word) processor executes a fixed bundle of operations per cycle that the compiler packed, trading hardware complexity for compiler responsibility and binary compatibility.

An in-order superscalar must check, every cycle, the dependences among the instructions it fetched together and with those in flight; an out-of-order one (Chapter 7) does far more. **VLIW** moves this work to the compiler: each wide instruction has one slot per functional unit, and the compiler guarantees that the operations in a bundle are independent and that results are not used before their latency has elapsed. The hardware is simple; the compiler must find the parallelism statically, without knowing cache-miss latencies or branch outcomes, and code compiled for one machine width does not run efficiently (or at all) on another. Intel's Itanium (EPIC) was the large-scale attempt; VLIW survives in digital signal processors and some accelerators.

The scheduler below packs a SAXPY loop body into bundles for a machine with two memory slots, two floating-point slots, and one integer slot:

```python
# file: vliw/vliw.py
"""Static scheduling for a VLIW machine: pack the operations of a loop body into wide instructions
(bundles) of fixed slots, respecting dependences and latencies — what a VLIW compiler does."""

SLOTS = {"mem": 2, "fp": 2, "int": 1}                    # functional units per bundle
LAT = {"load": 3, "fmul": 4, "fadd": 3, "store": 1, "add": 1}
UNIT = {"load": "mem", "store": "mem", "fmul": "fp", "fadd": "fp", "add": "int"}

def saxpy_body(unroll):
    """y[i] = a * x[i] + y[i], unrolled `unroll` times; returns (name, op, sources)."""
    ops = []
    for k in range(unroll):
        ops += [(f"lx{k}", "load", []), (f"ly{k}", "load", []),
                (f"m{k}", "fmul", [f"lx{k}"]), (f"a{k}", "fadd", [f"m{k}", f"ly{k}"]),
                (f"s{k}", "store", [f"a{k}"])]
    ops.append(("inc", "add", []))                        # pointer/counter update, once per body
    return ops

def schedule(ops):
    ready_at, placed, bundles = {}, set(), []
    cycle = 0
    while len(placed) < len(ops):
        free = dict(SLOTS)
        bundle = []
        for name, op, srcs in ops:                        # list scheduling, in program order
            if name in placed or free[UNIT[op]] == 0:
                continue
            if all(s in ready_at and ready_at[s] <= cycle for s in srcs):
                bundle.append(name)
                free[UNIT[op]] -= 1
                placed.add(name)
                ready_at[name] = cycle + LAT[op]
        bundles.append(bundle)
        cycle += 1
    while any(t > len(bundles) for t in ready_at.values()):
        bundles.append([])                                # wait for the last results
    return bundles

for unroll in (1, 4):
    ops = saxpy_body(unroll)
    b = schedule(ops)
    used = sum(len(x) for x in b)
    total = len(b) * sum(SLOTS.values())
    print(f"unroll {unroll}: {len(ops)} operations in {len(b)} bundles, "
          f"{len(b) / unroll:.2f} cycles per element, {100 * used / total:.0f}% of slots used")
    for i, x in enumerate(b):
        print(f"   bundle {i:2d}: " + (", ".join(x) if x else "(empty: waiting for latency)"))
```

```console
$ cd vliw && python3 vliw.py
unroll 1: 6 operations in 11 bundles, 11.00 cycles per element, 11% of slots used
   bundle  0: lx0, ly0, inc
   bundle  1: (empty: waiting for latency)
   bundle  2: (empty: waiting for latency)
   bundle  3: m0
   bundle  4: (empty: waiting for latency)
   bundle  5: (empty: waiting for latency)
   bundle  6: (empty: waiting for latency)
   bundle  7: a0
   bundle  8: (empty: waiting for latency)
   bundle  9: (empty: waiting for latency)
   bundle 10: s0
unroll 4: 21 operations in 14 bundles, 3.50 cycles per element, 30% of slots used
   bundle  0: lx0, ly0, inc
   bundle  1: lx1, ly1
   bundle  2: lx2, ly2
   bundle  3: m0, lx3, ly3
   bundle  4: m1
   bundle  5: m2
   bundle  6: m3
   bundle  7: a0
   bundle  8: a1
   bundle  9: a2
   bundle 10: s0, a3
   bundle 11: s1
   bundle 12: s2
   bundle 13: s3
```

One iteration alone fills 11% of the slots: the chain load → multiply → add → store is 11 cycles of latency with almost nothing to do in parallel. Unrolling four iterations gives the scheduler independent work to interleave, and the time per element falls from 11 to 3.5 cycles. The remaining limit is the two memory slots (each element needs two loads and a store). A real VLIW compiler goes further with **software pipelining**, overlapping iterations so that each bundle contains the store of iteration *i*, the add of *i* + 1, the multiply of *i* + 2, and the loads of *i* + 3 — one element per 1.5 cycles here, limited only by memory slots.

### Mastery check

1. **Explain.** Why can a VLIW compiler not schedule around a cache miss, while an out-of-order core can?
2. **Calculate.** With software pipelining, what is the best possible cycles per element for this body on this machine, and which resource limits it?

## Topic 9B — Data-level parallelism

## Unit 9.3 — SIMD instructions on a real CPU

**One-line summary:** SIMD extensions (SSE: 128-bit, AVX2: 256-bit registers) apply one instruction to 4 or 8 floats at once; compilers auto-vectorise simple loops, and the measured speedup on SAXPY is about 4× with SSE and 8× with AVX2 — but a floating-point *reduction* such as a dot product is not vectorised unless the compiler is allowed to reorder additions.

```c
// file: simd/simd.c
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <time.h>
#define N 4096
static float x[N], y[N];
static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }
__attribute__((noipa)) void saxpy(int n, float a, const float *restrict xs, float *restrict ys) {
    for (int i = 0; i < n; i++) ys[i] = a * xs[i] + ys[i];
}
__attribute__((noipa)) float dot(int n, const float *xs, const float *ys) {
    float s = 0;
    for (int i = 0; i < n; i++) s += xs[i] * ys[i];
    return s;
}
int main(void) {
    for (int i = 0; i < N; i++) { x[i] = i * 0.001f; y[i] = 1.0f; }
    double t0 = now();
    for (int r = 0; r < 100000; r++) saxpy(N, 1e-7f, x, y);
    double t_saxpy = now() - t0;
    t0 = now();
    float s = 0;
    for (int r = 0; r < 20000; r++) s += dot(N, x, y);
    double t_dot = now() - t0;
    printf("saxpy: %.2f ns per element   dot: %.2f ns per element   (check %.3f %.1f)\n",
           t_saxpy * 1e9 / (100000.0 * N), t_dot * 1e9 / (20000.0 * N), y[N - 1], s / 20000);
    return 0;
}
```

```console
$ cd simd && for flags in "-O2 -fno-tree-vectorize" "-O3" "-O3 -march=x86-64-v3" "-O3 -march=x86-64-v3 -ffast-math"; do gcc $flags simd.c -o simd && echo -n "$flags: " && ./simd; done
-O2 -fno-tree-vectorize: saxpy: 0.29 ns per element   dot: 0.52 ns per element   (check 1.036 8612.7)
-O3: saxpy: 0.08 ns per element   dot: 0.51 ns per element   (check 1.036 8612.7)
-O3 -march=x86-64-v3: saxpy: 0.04 ns per element   dot: 0.53 ns per element   (check 1.036 8612.7)
-O3 -march=x86-64-v3 -ffast-math: saxpy: 0.08 ns per element   dot: 0.07 ns per element   (check 1.036 8612.7)
$ cd simd && gcc -O3 -march=x86-64-v3 -S -o - simd.c | grep -m3 -oE "\s(vfmadd[0-9a-z]+|vmulps|vaddps|vmovups)\s+[^,]+,[^,]+(,.*)?"
	vmovups	(%rcx,%rax), %ymm1
	vfmadd213ps	(%rdx,%rax), %ymm2, %ymm1
	vmovups	%ymm1, (%rdx,%rax)
```

`-O3` turns on GCC's loop vectoriser (SSE2, the x86-64 baseline: 4 floats per instruction); `-march=x86-64-v3` allows AVX2 and FMA (8 floats per instruction, and fused multiply-add). SAXPY speeds up about 4× and then 8×, with the arrays in the L1 cache. The dot product does not move until `-ffast-math`. (These are single runs of a few microseconds each; the SAXPY figures wobble by a few hundredths of a nanosecond from run to run, so the change in the last row's SAXPY is noise.) The `restrict` qualifiers on SAXPY's pointers tell the compiler that `x` and `y` do not overlap; without them it must generate a run-time overlap check before using vector code.

### Deep & Tricky

**DT-26. Floating-point addition is not associative — `(a + b) + c` can differ from `a + (b + c)` in the last bits, or by much more with mixed magnitudes — so a compiler may not reorder a sum into vector partial sums without permission; `-ffast-math` (or `-fassociative-math`) grants it, and with it the result may change slightly. The same rule makes parallel reductions on GPUs and multicores give run-to-run differences.** *(IEEE 754-2019 §10.4: language standards should not allow "value-changing optimizations" by default; ISO C17 §5.1.2.3 and Annex F; GCC manual, "-fassociative-math": "allow re-association of operands in series of floating-point operations… may change computation results")* Integer sums vectorise at `-O3` without permission, because wrapping integer addition *is* associative.

### Mastery check

1. **Predict.** If the dot product used `int` instead of `float`, which build would vectorise it?
2. **Explain.** Why does SAXPY on arrays of 64 million floats (far larger than the caches) speed up much less than 8× with AVX2?

## Unit 9.4 — Vector processors and the RISC-V V extension

**One-line summary:** A vector ISA lets the *hardware* say how many elements one instruction processes (`vsetvl` returns the vector length), so one binary runs on machines with 128-bit or 1024-bit vector registers, and loops are "strip-mined" into chunks of whatever size the hardware supports — the opposite of fixed-width SIMD, where AVX2 code must be rewritten for AVX-512.

Classic vector supercomputers (the Cray-1, 1976) had long vector registers, pipelined functional units, and a vector-length register. RISC-V V follows them: `vsetvli` takes the remaining element count and returns how many the hardware will process in this iteration (at most the register length), and every vector instruction obeys that length — so there is no separate "remainder" loop for the last few elements.

```c
// file: vector/saxpy_rvv.c
// SAXPY with the RISC-V vector extension: a vector-length-agnostic "strip-mined" loop.
#include <riscv_vector.h>
#include <stdio.h>

static int strips;

void saxpy_rvv(size_t n, float a, const float *x, float *y) {
    while (n > 0) {
        size_t vl = __riscv_vsetvl_e32m1(n);                  // hardware says how many it will do
        vfloat32m1_t vx = __riscv_vle32_v_f32m1(x, vl);
        vfloat32m1_t vy = __riscv_vle32_v_f32m1(y, vl);
        vy = __riscv_vfmacc_vf_f32m1(vy, a, vx, vl);           // vy += a * vx, vl elements at once
        __riscv_vse32_v_f32m1(y, vy, vl);
        x += vl; y += vl; n -= vl;
        strips++;
    }
}

int main(void) {
    enum { N = 100 };
    float x[N], y[N];
    for (int i = 0; i < N; i++) { x[i] = i; y[i] = 1; }
    saxpy_rvv(N, 2.0f, x, y);
    int ok = 1;
    for (int i = 0; i < N; i++) ok &= (y[i] == 2.0f * i + 1);
    printf("VLEN %4zu bits: %2zu floats per vector, %2d strips for %d elements, results %s\n",
           __riscv_vsetvlmax_e32m1() * 32, __riscv_vsetvlmax_e32m1(), strips, N, ok ? "correct" : "WRONG");
    return 0;
}
```

```console
$ cd vector && riscv64-linux-gnu-gcc -O2 -march=rv64gcv -static saxpy_rvv.c -o saxpy_rvv && for v in 128 256 512 1024; do qemu-riscv64 -cpu rv64,v=true,vext_spec=v1.0,vlen=$v ./saxpy_rvv; done
VLEN  128 bits:  4 floats per vector, 25 strips for 100 elements, results correct
VLEN  256 bits:  8 floats per vector, 13 strips for 100 elements, results correct
VLEN  512 bits: 16 floats per vector,  7 strips for 100 elements, results correct
VLEN 1024 bits: 32 floats per vector,  4 strips for 100 elements, results correct
$ cd vector && riscv64-linux-gnu-objdump -d saxpy_rvv | sed -n '/<saxpy_rvv>:/,/ret/p' | grep -E "vsetvli|vle32|vfmacc|vse32" | cut -f3-
vsetvli	a5,a0,e32,m1,ta,ma
vle32.v	v25,(a1)
vle32.v	v24,(a2)
vfmacc.vf	v24,fa0,v25
vse32.v	v24,(a2)
```

The same binary, run on four emulated machines with vector registers from 128 to 1024 bits, processes 4 to 32 floats per instruction, needs 25 down to 4 trips through the loop, and is correct on all of them — including the last, partial strip (100 is not a multiple of 32). The loop is the four vector instructions shown, plus pointer updates. (The intrinsics come from `<riscv_vector.h>`; GCC 13 does not yet auto-vectorise for RVV, GCC 14 and later do.)

### Mastery check

1. **Explain.** Why does fixed-width SIMD code need a scalar "epilogue" loop, and why does RVV not?
2. **Calculate.** For 1,000 elements and VLEN = 512, how many strips? How long is the last one?

## Unit 9.5 — GPUs: SIMT and branch divergence

**One-line summary:** A GPU runs thousands of threads, grouped into warps (32 threads on NVIDIA GPUs) that share one instruction stream and execute in lock-step on SIMD lanes — single instruction, multiple threads (SIMT); when threads of a warp take different paths at a branch, the warp executes both paths with lanes masked off, wasting lanes, and reconverges afterwards.

GPUs hide memory latency not with caches and out-of-order execution but with **multithreading**: each streaming multiprocessor holds dozens of warps, and every cycle it issues an instruction from a warp that is ready, switching for free. A warp waiting 400 cycles for memory is simply not picked until its data arrives. The programmer writes a scalar kernel for one thread (CUDA, OpenCL); the hardware groups threads into warps.

Divergence is the price of lock-step execution. The model below runs a kernel with a 20-instruction "then" path and a 10-instruction "else" path over 4,096 threads, with different branch conditions, and reports **SIMD efficiency** — the fraction of lane-slots doing useful work:

```python
# file: gpu/simt.py
"""A SIMT model: a warp of 32 threads shares one program counter. When threads disagree at a
branch, the warp runs each path in turn with the other threads masked off, then reconverges."""

WARP = 32

def run_kernel(n_threads, branch_of):
    """Each thread runs: 4 instructions; if branch_of(tid): 20 instructions else 10; then 4 more.
    Returns (issue cycles, useful thread-instructions, total lane-slots)."""
    issue = useful = 0
    for w in range(0, n_threads, WARP):
        tids = list(range(w, min(w + WARP, n_threads)))
        takes = [branch_of(t) for t in tids]
        issue += 4; useful += 4 * len(tids)                       # common prefix: all lanes active
        n_then = sum(takes)
        if n_then:                                                # "then" path, other lanes masked
            issue += 20; useful += 20 * n_then
        if n_then < len(tids):                                    # "else" path
            issue += 10; useful += 10 * (len(tids) - n_then)
        issue += 4; useful += 4 * len(tids)                       # after reconvergence
    return issue, useful, issue * WARP

cases = [
    ("no branch taken by anyone", lambda t: False),
    ("whole warps agree (tid / 32 even)", lambda t: (t // 32) % 2 == 0),
    ("half of each warp (tid even)", lambda t: t % 2 == 0),
    ("one thread per warp (tid % 32 == 0)", lambda t: t % 32 == 0),
    ("data-dependent, about half", lambda t: (t * 2654435761 >> 7) & 1 == 1),
]
print(f"{'branch pattern':38} {'issue cycles':>12} {'SIMD efficiency':>16}")
for name, f in cases:
    issue, useful, slots = run_kernel(4096, f)
    print(f"{name:38} {issue:12d} {100 * useful / slots:15.0f}%")
```

```console
$ cd gpu && python3 simt.py
branch pattern                         issue cycles  SIMD efficiency
no branch taken by anyone                      2304             100%
whole warps agree (tid / 32 even)              2944             100%
half of each warp (tid even)                   4864              61%
one thread per warp (tid % 32 == 0)            4864              48%
data-dependent, about half                     4864              61%
```

When whole warps agree — even if half the warps take each path — efficiency stays at 100%: each warp runs only its own path. When threads *within* a warp disagree, every warp runs both paths, and efficiency drops to 61%; with a single diverging thread per warp, the other 31 lanes idle through the 20-instruction "then" path, and it falls to 48%. The lesson for GPU programmers: arrange data so that threads with the same behaviour land in the same warp (sort or bin work first), and keep divergent paths short. The second rule, for memory, is **coalescing**: when the 32 threads of a warp access 32 consecutive words, the hardware combines them into one or two wide memory transactions; scattered addresses cost one transaction each.

### Mastery check

1. **Calculate.** A warp with 8 threads on the then-path (20 instructions) and 24 on the else-path (10 instructions), plus 8 common instructions: what is its SIMD efficiency?
2. **Explain.** Why do GPUs need far more registers per core than CPUs?

## Topic 9C — Systolic arrays

## Unit 9.6 — A systolic array for matrix multiplication

**One-line summary:** A systolic array is a grid of identical processing elements, each connected only to its neighbours, through which data flows rhythmically ("like blood pumped by the heart"); for matrix multiplication, rows of A flow right and columns of B flow down, each element multiplies and accumulates what passes through it, and N² multiply-accumulates happen every cycle with no shared memory or long wires.

H. T. Kung and Charles Leiserson introduced systolic arrays in 1978 to make computation-heavy, data-light kernels fast with simple, regular hardware. The idea returned at scale in Google's Tensor Processing Unit (2016), whose core is a 256 × 256 systolic array of 8-bit multiply-accumulate units. In the **output-stationary** design below, PE (i, j) computes C[i][j]: A[i][k] arrives from the left and B[k][j] from above in the same cycle, because row i of A is fed i cycles late and column j of B is fed j cycles late — the input **skew**.

```verilog
// file: systolic/systolic.v
// An N x N output-stationary systolic array computing C = A * B.
// Row i of A enters from the left, delayed by i cycles; column j of B enters from the top,
// delayed by j cycles. Each processing element multiplies what passes through it and accumulates.
module pe(input clk, input rst, input signed [7:0] a_in, input signed [7:0] b_in,
          output reg signed [7:0] a_out, output reg signed [7:0] b_out, output reg signed [31:0] acc);
    always @(posedge clk)
        if (rst) begin a_out <= 0; b_out <= 0; acc <= 0; end
        else begin
            acc   <= acc + a_in * b_in;     // multiply-accumulate
            a_out <= a_in;                  // pass A to the right
            b_out <= b_in;                  // pass B down
        end
endmodule

module systolic #(parameter N = 4)(input clk, input rst,
                  input [8*N-1:0] a_left,           // one A value per row, entering at the left edge
                  input [8*N-1:0] b_top,            // one B value per column, entering at the top
                  output [32*N*N-1:0] c);           // C[i][j] at bits (i*N + j)*32
    wire signed [7:0] a [0:N-1][0:N];               // a[i][j]: A value entering PE (i, j)
    wire signed [7:0] b [0:N][0:N-1];               // b[i][j]: B value entering PE (i, j)
    genvar i, j;
    generate
        for (i = 0; i < N; i = i + 1) begin : edges
            assign a[i][0] = a_left[8*i +: 8];
            assign b[0][i] = b_top[8*i +: 8];
        end
        for (i = 0; i < N; i = i + 1) begin : row
            for (j = 0; j < N; j = j + 1) begin : col
                pe p(clk, rst, a[i][j], b[i][j], a[i][j+1], b[i+1][j], c[32*(i*N+j) +: 32]);
            end
        end
    endgenerate
endmodule
```

```verilog
// file: systolic/tb.v
module tb;
    localparam N = 4;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    reg  [8*N-1:0] a_left, b_top;
    wire [32*N*N-1:0] c;
    systolic #(N) dut(clk, rst, a_left, b_top, c);
    integer A [0:N-1][0:N-1], B [0:N-1][0:N-1], C [0:N-1][0:N-1];
    integer i, j, k, t, errors = 0, cycles = 0;
    initial begin
        for (i = 0; i < N; i = i + 1)
            for (j = 0; j < N; j = j + 1) begin
                A[i][j] = $random % 100; B[i][j] = $random % 100;
            end
        for (i = 0; i < N; i = i + 1)                       // reference result
            for (j = 0; j < N; j = j + 1) begin
                C[i][j] = 0;
                for (k = 0; k < N; k = k + 1) C[i][j] = C[i][j] + A[i][k] * B[k][j];
            end
        a_left = 0; b_top = 0;
        @(negedge clk) rst = 0;
        for (t = 0; t < 3 * N - 1; t = t + 1) begin          // feed skewed inputs
            for (i = 0; i < N; i = i + 1) begin
                k = t - i;                                   // row i is delayed by i cycles
                a_left[8*i +: 8] = (k >= 0 && k < N) ? A[i][k] : 0;
                b_top[8*i +: 8]  = (k >= 0 && k < N) ? B[k][i] : 0;   // column i, delayed by i
            end
            @(negedge clk) cycles = cycles + 1;
        end
        for (i = 0; i < N; i = i + 1)
            for (j = 0; j < N; j = j + 1)
                if ($signed(c[32*(i*N+j) +: 32]) !== C[i][j]) errors = errors + 1;
        $display("%0dx%0d systolic matrix multiply: %0d cycles, %0d of %0d results wrong", N, N, cycles, errors, N * N);
        $display("C[0][0] = %0d (reference %0d), C[%0d][%0d] = %0d (reference %0d)", $signed(c[31:0]), C[0][0],
                 N - 1, N - 1, $signed(c[32*(N*N-1) +: 32]), C[N-1][N-1]);
        $finish;
    end
endmodule
```

```console
$ cd systolic && iverilog -g2012 -o sim systolic.v tb.v && vvp -n sim | grep -v finish
4x4 systolic matrix multiply: 11 cycles, 0 of 16 results wrong
C[0][0] = -7048 (reference -7048), C[3][3] = -8976 (reference -8976)
$ cd systolic && yosys -p "read_verilog systolic.v; synth -top systolic; stat" | grep -E "Number of cells" | tail -1 | tr -s ' '
 Number of cells: 17712
```

All 16 results match the reference in 11 cycles (3N − 1 for N = 4: N cycles of data for the last PE, plus 2(N − 1) cycles of skew). Each PE talks only to its right and lower neighbours, so the wiring stays short however large the array grows, and every PE does a multiply-accumulate in every cycle once the pipeline is full. Synthesis shows where the area goes: almost all of the 17,000 cells are the sixteen 8 × 8 multipliers and 32-bit accumulators.

### Mastery check

1. **Explain.** Why does row i of A have to enter i cycles later than row 0?
2. **Calculate.** How many cycles does an N × N array need for one N × N product, as a function of N? What fraction of that time is every PE busy?

## Chapter 9 Project — Tiled matrix multiplication on a fixed-size array

### Requirements

A real accelerator has a fixed array (here 4 × 4) and must multiply matrices of any size. Write a testbench that multiplies a 12 × 16 matrix by a 16 × 8 matrix on the 4 × 4 systolic array by **tiling**: compute the output one 4 × 4 tile at a time, streaming the full inner dimension (K = 16) through the array for each tile, clearing the accumulators between tiles. Check every result against a reference, and report the total cycles and the array **utilisation** (useful multiply-accumulates divided by array capacity × cycles).

### Reference solution

```verilog
// file: project/tb_tiled.v
// Multiply an M x K matrix by a K x P matrix on the 4 x 4 systolic array, one 4 x 4 output tile at a time.
module tb;
    localparam N = 4, M = 12, K = 16, P = 8;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    reg  [8*N-1:0] a_left, b_top;
    wire [32*N*N-1:0] c;
    systolic #(N) dut(clk, rst, a_left, b_top, c);
    integer A [0:M-1][0:K-1], B [0:K-1][0:P-1], C [0:M-1][0:P-1], R [0:M-1][0:P-1];
    integer i, j, k, t, ti, tj, errors = 0, cycles = 0;
    initial begin
        for (i = 0; i < M; i = i + 1) for (k = 0; k < K; k = k + 1) A[i][k] = $random % 50;
        for (k = 0; k < K; k = k + 1) for (j = 0; j < P; j = j + 1) B[k][j] = $random % 50;
        for (i = 0; i < M; i = i + 1) for (j = 0; j < P; j = j + 1) begin
            R[i][j] = 0;
            for (k = 0; k < K; k = k + 1) R[i][j] = R[i][j] + A[i][k] * B[k][j];
        end
        a_left = 0; b_top = 0;
        for (ti = 0; ti < M; ti = ti + N)
            for (tj = 0; tj < P; tj = tj + N) begin
                @(negedge clk) rst = 1;                          // clear the accumulators
                @(negedge clk) rst = 0; cycles = cycles + 1;
                for (t = 0; t < K + 2 * N - 2; t = t + 1) begin  // stream K products through the tile
                    for (i = 0; i < N; i = i + 1) begin
                        k = t - i;
                        a_left[8*i +: 8] = (k >= 0 && k < K) ? A[ti + i][k] : 0;
                        b_top[8*i +: 8]  = (k >= 0 && k < K) ? B[k][tj + i] : 0;
                    end
                    @(negedge clk) cycles = cycles + 1;
                end
                for (i = 0; i < N; i = i + 1)
                    for (j = 0; j < N; j = j + 1) C[ti + i][tj + j] = $signed(c[32*(i*N+j) +: 32]);
            end
        for (i = 0; i < M; i = i + 1) for (j = 0; j < P; j = j + 1) if (C[i][j] !== R[i][j]) errors = errors + 1;
        $display("(%0dx%0d) x (%0dx%0d) on a %0dx%0d array: %0d tiles, %0d cycles, %0d of %0d results wrong",
                 M, K, K, P, N, N, (M / N) * (P / N), cycles, errors, M * P);
        $display("multiply-accumulates %0d; array capacity %0d per cycle; utilisation %0d%%",
                 M * K * P, N * N, 100 * M * K * P / (cycles * N * N));
        $finish;
    end
endmodule
```

```console
$ cd project && iverilog -g2012 -o sim ../systolic/systolic.v tb_tiled.v && vvp -n sim | grep -v finish
(12x16) x (16x8) on a 4x4 array: 6 tiles, 138 cycles, 0 of 96 results wrong
multiply-accumulates 1536; array capacity 16 per cycle; utilisation 69%
```

All 96 results are right, using 6 tiles in 138 cycles: 1,536 multiply-accumulates on an array capable of 16 per cycle, 69% utilisation. The losses are the skew — each tile spends 2(N − 1) = 6 of its 22 streaming cycles filling and draining the array — and the reset cycle between tiles. Longer inner dimensions amortise them (K = 256 would give over 97%), which is why accelerators favour large matrices and batch small ones together; real designs also overlap the drain of one tile with the fill of the next (double-buffered accumulators).

### Review checklist

- Where would weight-stationary dataflow (B held in the PEs, A streaming through) reduce data movement? What does the TPU use?
- What limits the array's clock rate: the multiplier, the accumulator adder, or the wires? Which would you pipeline first?
- How many bytes per cycle must the memory system deliver to keep the 4 × 4 array busy? A 256 × 256 one?

## Answers to the Chapter 9 prediction questions

**Unit 9.1, Q1.** Each warp is SIMD (one instruction, 32 lanes); the GPU as a whole is MIMD (many warps, each with its own program counter, on many multiprocessors).

**Unit 9.2, Q2.** Each element needs 2 loads and 1 store = 3 memory operations; with 2 memory slots per bundle, at best 1.5 cycles per element.

**Unit 9.3, Q1.** With `int`, the dot product vectorises at `-O3` (SSE) and `-O3 -march=x86-64-v3` (AVX2) — integer addition is associative, so no `-ffast-math` is needed.

**Unit 9.3, Q2.** It becomes limited by memory bandwidth (Chapter 0's ~10 GB/s): 12 bytes moved per element, so about 1 ns per element whatever the instruction set.

**Unit 9.4, Q2.** ⌈1000 / 16⌉ = 63 strips (16 floats per vector at VLEN 512); the last strip has 1000 − 62 × 16 = 8 elements.

**Unit 9.5, Q1.** Useful: 32 × 8 + 8 × 20 + 24 × 10 = 256 + 160 + 240 = 656. Slots: (8 + 20 + 10) × 32 = 1,216. Efficiency 54%.

**Unit 9.6, Q2.** 3N − 1 cycles (counting from the first input); every PE is busy for N of them, so N / (3N − 1) — about a third for large N, for a single product. Streaming many products back to back raises it towards 100%.

# Chapter 10 — Caches

*Goal: understand the memory hierarchy as an engineer who must both build caches and write software that uses them well. The chapter builds a trace-driven cache simulator to study organisation, the three kinds of misses, replacement, and write policy; measures on the host CPU how loop order and a single unlucky array size change a matrix multiply's time by 8×; and ends with a working set-associative write-back cache in Verilog, verified against a reference memory.*

*Accompanies the lectures "Memory Hierarchy and Caches", "More Caches", and "Prefetching".*

**What you will be able to do after this chapter**

- Explain why caches work (locality) and compute average memory access time.
- Split an address into tag, index, and offset for any cache geometry, and say where a block can live.
- Classify misses as compulsory, capacity, or conflict, and pick the change that removes each kind.
- Compare replacement and write policies by simulation.
- Restructure loops for locality, recognise power-of-two conflict traps, and design a cache controller in RTL.

## Topic 10A — The memory hierarchy

## Unit 10.1 — Locality and average memory access time

**One-line summary:** Memory that is both big and fast cannot be built, so computers use a hierarchy — small fast SRAM caches in front of large slow DRAM — that works because programs reuse recently used data (temporal locality) and use data near recently used data (spatial locality); its performance is summarised by the average memory access time, AMAT = hit time + miss rate × miss penalty.

Chapter 0 measured the hierarchy of this machine with a pointer chase: about 1 ns for data in the L1 cache, a few ns in L2, tens in L3, and around 100 ns from DRAM. A load that misses everything costs as much as a few hundred instructions. Caches hide this by keeping copies of recently used **blocks** (also called lines; 64 bytes on almost all current CPUs) close to the processor:

- **Temporal locality:** a loop counter, a stack frame, the code of a loop — used again soon.
- **Spatial locality:** the next array element, the next instruction — fetching a whole 64-byte block for one load brings its neighbours for free.

For a multi-level hierarchy the formula nests: AMAT = t(L1) + m(L1) × (t(L2) + m(L2) × (t(L3) + m(L3) × t(DRAM))), where m(L2) is the *local* miss rate — the fraction of L1 misses that also miss in L2.

**Worked example.** L1 hit 1 ns, miss rate 5%; L2 hit 4 ns, local miss rate 20%; DRAM 100 ns. AMAT = 1 + 0.05 × (4 + 0.2 × 100) = 1 + 0.05 × 24 = 2.2 ns. Halving the L1 miss rate gives 1.6 ns; halving DRAM latency gives 1.7 ns.

### Mastery check

1. **Calculate.** In the worked example, what is the *global* miss rate of L2 (the fraction of all accesses that go to DRAM)?
2. **Explain.** Why is the block size 64 bytes rather than 4 bytes (one word) or 4 KiB?

## Unit 10.2 — Cache organisation: tag, index, offset

**One-line summary:** A cache of S sets, each with W ways (blocks), and blocks of B bytes splits every address into an offset (log₂ B bits: which byte in the block), an index (log₂ S bits: which set), and a tag (the rest: which of the many blocks that map to this set is stored); direct-mapped caches have one way, fully associative caches have one set.

Size = S × W × B. For the 8 KiB, 64-byte-block caches simulated below:

| Organisation | Ways W | Sets S | Offset bits | Index bits | Tag bits (32-bit address) |
| --- | --- | --- | --- | --- | --- |
| Direct-mapped | 1 | 128 | 6 | 7 | 19 |
| 4-way set-associative | 4 | 32 | 6 | 5 | 21 |
| Fully associative | 128 | 1 | 6 | 0 | 26 |

**Worked example.** Address 0x0000_3A7C in the 4-way cache: offset = low 6 bits = 0x3C = 60; index = next 5 bits = (0x3A7C >> 6) & 31 = 0xE9 & 31 = 9; tag = 0x3A7C >> 11 = 7. The block can live in any of the 4 ways of set 9; a lookup compares 7 against the 4 tags stored there, in parallel, and checks each valid bit.

Each way needs a comparator, so associativity costs area, energy, and hit time (the way multiplexer sits on the critical path). L1 caches are typically 8- to 12-way, L2 and L3 caches 16-way or more. The simulator below models any geometry; Python's `OrderedDict` keeps each set in recency order.

```python
# file: sim/cachesim.py
"""A trace-driven cache simulator: size, block size, associativity, replacement policy,
write policy, and classification of misses into compulsory, capacity, and conflict (the 3 Cs)."""
import random
from collections import OrderedDict

class Cache:
    def __init__(self, size, block, ways, policy="lru", write_back=True, seed=1):
        self.block, self.ways = block, ways
        self.sets = size // (block * ways)
        assert self.sets * block * ways == size and self.sets & (self.sets - 1) == 0
        self.data = [OrderedDict() for _ in range(self.sets)]   # tag -> dirty, in LRU order
        self.policy, self.write_back = policy, write_back
        self.rng = random.Random(seed)
        self.hits = self.misses = self.writebacks = self.mem_writes = 0

    def split(self, addr):
        blk = addr // self.block
        return blk % self.sets, blk // self.sets            # index, tag

    def access(self, addr, write=False):
        index, tag = self.split(addr)
        s = self.data[index]
        if tag in s:
            self.hits += 1
            if self.policy == "lru":
                s.move_to_end(tag)                           # most recently used goes last
            hit = True
        else:
            self.misses += 1
            if write and not self.write_back:                # write-through, no write-allocate
                self.mem_writes += 1
                return False
            if len(s) == self.ways:                          # evict a victim
                if self.policy == "random":
                    victim = self.rng.choice(list(s))
                else:                                        # lru and fifo both evict the first
                    victim = next(iter(s))
                if s.pop(victim):
                    self.writebacks += 1
            s[tag] = False
            hit = False
        if write:
            if self.write_back:
                s[tag] = True                                # mark dirty; memory updated on eviction
            else:
                self.mem_writes += 1                         # write-through: every store goes to memory
        return hit

def classify(trace, size, block, ways):
    """3C model: compulsory = first touch of a block; capacity = also misses in a fully associative
    LRU cache of the same size; conflict = the rest."""
    real = Cache(size, block, ways)
    full = Cache(size, block, size // block)
    seen = set()
    comp = cap = conf = 0
    for addr in trace:
        blk = addr // block
        h_real, h_full = real.access(addr), full.access(addr)
        if not h_real:
            if blk not in seen:
                comp += 1
            elif not h_full:
                cap += 1
            else:
                conf += 1
        seen.add(blk)
    return comp, cap, conf, len(trace)
```

### Mastery check

1. **Calculate.** A 48 KiB, 12-way cache with 64-byte blocks: how many sets, and how many index bits?
2. **Calculate.** For the 4-way 8 KiB cache, which set does address 0x0000_2000 map to? And 0x0000_4000? What does that suggest about arrays 8 KiB apart?

## Topic 10B — Misses and policies

## Unit 10.3 — The three Cs

**One-line summary:** Misses are compulsory (first touch of a block — only prefetching or larger blocks help), capacity (the working set does not fit — only a bigger cache or better locality helps), or conflict (too many hot blocks map to the same set — more associativity or a different layout helps); simulating a fully associative cache of the same size separates the last two.

```python
# file: sim/threecs.py
from cachesim import Cache, classify
import random

def matrix_walk(n, by_row, base=0, elem=4, reps=2):
    """Read an n x n int matrix stored row-major, `reps` times, by rows or by columns."""
    return [base + elem * (i * n + j if by_row else j * n + i)
            for _ in range(reps) for i in range(n) for j in range(n)]

def ping_pong(stride, n=256, reps=4):
    """Alternate between two arrays whose starts are `stride` bytes apart."""
    return [base + 4 * i for _ in range(reps) for i in range(n) for base in (0, stride)]

rng = random.Random(7)
traces = {
    "row-major walk of 64x64 ints": matrix_walk(64, True),
    "column walk of 64x64 ints": matrix_walk(64, False),
    "two arrays 8 KiB apart": ping_pong(8192),
    "two arrays 8 KiB + 64 B apart": ping_pong(8192 + 64),
    "random in 32 KiB": [rng.randrange(0, 32768, 4) for _ in range(20000)],
}
print("8 KiB cache, 64-byte blocks")
print(f"{'trace':32} {'ways':>4} {'miss %':>7} {'compulsory':>11} {'capacity':>9} {'conflict':>9}")
for name, t in traces.items():
    for ways in (1, 4):
        comp, cap, conf, n = classify(t, 8192, 64, ways)
        print(f"{name:32} {ways:4d} {100 * (comp + cap + conf) / n:6.1f}% {comp:11d} {cap:9d} {conf:9d}")
```

```console
$ cd sim && python3 threecs.py
8 KiB cache, 64-byte blocks
trace                            ways  miss %  compulsory  capacity  conflict
row-major walk of 64x64 ints        1    6.2%         256       256         0
row-major walk of 64x64 ints        4    6.2%         256       256         0
column walk of 64x64 ints           1  100.0%         256       256      7680
column walk of 64x64 ints           4  100.0%         256       256      7680
two arrays 8 KiB apart              1  100.0%          32         0      2016
two arrays 8 KiB apart              4    1.6%          32         0         0
two arrays 8 KiB + 64 B apart       1    6.0%          32         0        90
two arrays 8 KiB + 64 B apart       4    1.6%          32         0         0
random in 32 KiB                    1   75.2%         512     12967      1560
random in 32 KiB                    4   75.3%         512     13718       828
```

Read the table row by row:

- **Row-major walk** of a 16 KiB matrix through an 8 KiB cache: one miss per 64-byte block (16 ints), so 6.25%; the first pass misses are compulsory, the second pass's are capacity (the matrix does not fit). Associativity does not matter.
- **Column walk** of the same matrix: every access misses. Each column touches 64 blocks 256 bytes apart. A fully associative cache would keep them, and the next 15 columns would hit — but 256-byte strides use only every fourth set, and in the 4-way cache only 8 of the 32 sets, so 64 blocks compete for 32 slots. The misses are conflict misses, and they do not go away with 4 ways.
- **Two arrays exactly 8 KiB apart:** in the direct-mapped cache `x[i]` and `y[i]` share a set and evict each other on every access — 100% misses on 512 bytes of data. Four ways solve it completely; so does moving one array by a single block (64 bytes).
- **Random accesses in 32 KiB:** mostly capacity misses; 4 ways change little.

### Deep & Tricky

**DT-27. The three-Cs split is defined relative to a fully associative LRU cache, and LRU is not optimal, so the "conflict" count can be negative or mislabel misses: in the random trace above, the 4-way cache shows *more* capacity misses (13,718) than the direct-mapped one (12,967), with the same total — the classification moved misses between columns, not the cache.** *(Hill & Smith, "Evaluating Associativity in CPU Caches", IEEE Trans. Computers 1989, which defined the model and notes that a set-associative cache can occasionally beat the fully associative one)* Use the three Cs to choose the *direction* of a fix, not to account for every miss.

### Mastery check

1. **Predict.** In the column walk, would doubling the cache to 16 KiB (4-way) help? Would 16-way at 8 KiB?
2. **Explain.** Why does moving the second array by 64 bytes fix the direct-mapped case almost completely?

## Unit 10.4 — Replacement policies

**One-line summary:** When a set is full, the cache must evict a block; LRU (least recently used) is usually good but fails catastrophically on loops slightly larger than the cache, where random replacement does far better; Belady's OPT (evict the block used furthest in the future) is the unattainable optimum used as a yardstick, and real caches use cheap LRU approximations and scan-resistant policies.

```python
# file: sim/replace.py
"""Replacement policies on a fully associative cache of 64 blocks: LRU, FIFO, random,
and Belady's OPT (evict the block used furthest in the future: the unreachable ideal)."""
from cachesim import Cache
import random

BLOCKS, BLOCK = 64, 64

def opt_misses(trace):
    blocks = [a // BLOCK for a in trace]
    next_use, last = [0] * len(blocks), {}
    for i in range(len(blocks) - 1, -1, -1):             # when is each block used next?
        next_use[i] = last.get(blocks[i], float("inf"))
        last[blocks[i]] = i
    cache, misses = {}, 0                                # block -> its next use
    for i, b in enumerate(blocks):
        if b not in cache:
            misses += 1
            if len(cache) == BLOCKS:
                del cache[max(cache, key=cache.get)]
        cache[b] = next_use[i]
    return misses

def run(trace):
    out = []
    for policy in ("lru", "fifo", "random"):
        c = Cache(BLOCKS * BLOCK, BLOCK, BLOCKS, policy)
        for a in trace:
            c.access(a)
        out.append(c.misses)
    out.append(opt_misses(trace))
    return [100 * m / len(trace) for m in out]

rng = random.Random(3)
hot = [rng.randrange(0, 48 * BLOCK) for _ in range(20000)]
traces = {
    "loop over 60 blocks": [b * BLOCK for _ in range(100) for b in range(60)],
    "loop over 65 blocks": [b * BLOCK for _ in range(100) for b in range(65)],
    "loop over 96 blocks": [b * BLOCK for _ in range(100) for b in range(96)],
    "hot set of 48 + a scan": [x for i, h in enumerate(hot) for x in ((h,) if i % 4 else (h, (10000 + i) * BLOCK))],
    "random over 128 blocks": [rng.randrange(0, 128 * BLOCK) for _ in range(20000)],
}
print(f"{'access pattern (64-block cache)':32} {'LRU':>6} {'FIFO':>6} {'random':>7} {'OPT':>6}   (miss %)")
for name, t in traces.items():
    print(f"{name:32} " + " ".join(f"{m:6.1f}" for m in run(t)))
```

```console
$ cd sim && python3 replace.py
access pattern (64-block cache)     LRU   FIFO  random    OPT   (miss %)
loop over 60 blocks                 1.0    1.0    1.0    1.0
loop over 65 blocks               100.0  100.0    3.8    2.5
loop over 96 blocks               100.0  100.0   59.1   34.3
hot set of 48 + a scan             31.7   43.0   43.0   20.2
random over 128 blocks             50.3   50.1   50.0   20.8
```

The loop rows are the important ones. A loop over 60 blocks fits: only compulsory misses. A loop over 65 blocks — one more than the cache holds — makes LRU and FIFO miss on *every* access: each block is evicted just before it is needed again. Random replacement keeps most of the loop by accident and misses 3.8%; OPT 2.5%. With 96 blocks, OPT still hits a third of the time by keeping a fixed subset. In the "hot set + scan" trace, a stream of blocks used only once (a scan) pushes the hot blocks out of LRU, which is why modern last-level caches insert new blocks at low priority (Qureshi et al.'s insertion policies; Intel's and ARM's RRIP-style policies) so that one-time blocks leave first. True LRU needs log₂(W!) bits per set and updates on every hit; hardware uses approximations such as tree pseudo-LRU (W − 1 bits per set).

### Mastery check

1. **Explain.** Why does random replacement beat LRU on the 65-block loop?
2. **Calculate.** How many bits per set does exact LRU need for 8 ways? Tree pseudo-LRU?

## Unit 10.5 — Write policies

**One-line summary:** A write-through cache sends every store to the next level (simple, memory always up to date, lots of traffic); a write-back cache marks the block dirty and writes it only when it is evicted (much less traffic, but evictions are slower and memory is stale); on a write miss, write-allocate fetches the block first, no-write-allocate sends the write past the cache.

```python
# file: sim/writes.py
"""Memory traffic of write-through versus write-back caches for three store patterns."""
from cachesim import Cache
import random

rng = random.Random(5)
patterns = {
    "counter updated 10,000 times": [(0, True)] * 10000,
    "array of 4096 ints written once": [(4 * i, True) for i in range(4096)],
    "read-mostly, 10% stores, 4 KiB": [(rng.randrange(0, 4096, 4), rng.random() < 0.1) for _ in range(10000)],
}
print(f"{'pattern (8 KiB, 4-way, 64 B blocks)':34} {'write-through: words to memory':>31} {'write-back: blocks to memory':>29}")
for name, trace in patterns.items():
    wt, wb = Cache(8192, 64, 4, write_back=False), Cache(8192, 64, 4)
    for a, w in trace:
        wt.access(a, w); wb.access(a, w)
    dirty = sum(d for s in wb.data for d in s.values())     # still dirty: written back at the end
    print(f"{name:34} {wt.mem_writes:31d} {wb.writebacks + dirty:29d}")
```

```console
$ cd sim && python3 writes.py
pattern (8 KiB, 4-way, 64 B blocks)  write-through: words to memory  write-back: blocks to memory
counter updated 10,000 times                                 10000                             1
array of 4096 ints written once                               4096                           256
read-mostly, 10% stores, 4 KiB                                 979                            64
```

A counter updated 10,000 times costs 10,000 memory writes with write-through and a single block write-back with write-back. Writing an array once costs 4,096 word writes versus 256 block writes (the same bytes, but in 64-byte bursts, which DRAM handles far better). Write-through caches therefore sit only where the next level is close — some L1 data caches write through to an L2 — and use a **write buffer** so that the processor does not wait for each store. Write-back caches need a dirty bit per block and an extra state in the controller (this chapter's project).

### Mastery check

1. **Explain.** Why is a write-through cache simpler to keep coherent between processors (Chapter 14)?
2. **Predict.** In the "array written once" row, how many blocks would a write-back cache *read* from memory with write-allocate? Why is that wasteful, and what do processors do about it?

## Topic 10C — Software and caches

## Unit 10.6 — Loop order, blocking, and the power-of-two trap

**One-line summary:** The order of nested loops decides whether the innermost loop walks memory sequentially (one miss per block) or with a large stride (one miss per access); on real hardware the difference in a matrix multiply is about 10×, and choosing a matrix dimension that is a power of two can make it worse still by mapping a whole column into a handful of cache sets.

```c
// file: host/matmul.c
// Matrix multiply three ways: the naive ijk order, the cache-friendly ikj order, and blocked (tiled).
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#ifndef N
#define N 1024
#endif
static double A[N][N], B[N][N], C[N][N];

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

__attribute__((noipa)) static void ijk(void) {           // inner loop walks B down a column
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            double s = 0;
            for (int k = 0; k < N; k++) s += A[i][k] * B[k][j];
            C[i][j] = s;
        }
}
__attribute__((noipa)) static void ikj(void) {           // inner loop walks B and C along rows
    for (int i = 0; i < N; i++)
        for (int k = 0; k < N; k++) {
            double a = A[i][k];
            for (int j = 0; j < N; j++) C[i][j] += a * B[k][j];
        }
}
__attribute__((noipa)) static void blocked(int T) {      // T x T tiles that stay in the cache
    for (int ii = 0; ii < N; ii += T)
        for (int kk = 0; kk < N; kk += T)
            for (int jj = 0; jj < N; jj += T)
                for (int i = ii; i < ii + T && i < N; i++)
                    for (int k = kk; k < kk + T && k < N; k++) {
                        double a = A[i][k];
                        for (int j = jj; j < jj + T && j < N; j++) C[i][j] += a * B[k][j];
                    }
}
static double check(void) { double s = 0; for (int i = 0; i < N; i++) s += C[i][(i * 7) % N]; return s; }
static void clear(void) { for (int i = 0; i < N; i++) for (int j = 0; j < N; j++) C[i][j] = 0; }

int main(void) {
    for (int i = 0; i < N; i++) for (int j = 0; j < N; j++) { A[i][j] = (i + j) % 7; B[i][j] = (i * j) % 5; }
    double t;
    t = now(); ijk(); printf("ijk          %6.2f s  check %.0f\n", now() - t, check());
    clear(); t = now(); ikj(); printf("ikj          %6.2f s  check %.0f\n", now() - t, check());
    for (int T = 16; T <= 256; T *= 4) {
        clear(); t = now(); blocked(T); printf("blocked %3d  %6.2f s  check %.0f\n", T, now() - t, check());
    }
    return 0;
}
```

```console
$ cd host && gcc -O2 matmul.c -o matmul && ./matmul
ijk            2.92 s  check 5030602
ikj            0.37 s  check 5030602
blocked  16    0.71 s  check 5030602
blocked  64    0.63 s  check 5030602
blocked 256    0.48 s  check 5030602
```

The `ijk` order computes each `C[i][j]` as a dot product: its inner loop walks `B` down a column, 8 KiB between consecutive elements, so every access is to a new block. The `ikj` order makes the inner loop walk a row of `B` and a row of `C` — consecutive addresses, which the hardware prefetcher also streams in early — and is about eight times faster. (Timings on this shared machine vary by up to a factor of two between runs; the ratios are stable.) Blocking (tiling) the loops so that sub-matrices stay in the cache is the textbook next step, but here it gains nothing and small tiles even lose: `ikj` already streams well, this processor's 2 MiB L2 and its prefetchers absorb the rest, and the extra loop levels cost time. Blocking matters most where the reuse distance would otherwise exceed the last-level cache, or for the register-level tiling inside optimised BLAS libraries, which reach ten times the speed of `ikj`.

Now change only the size:

```console
$ cd host && for n in 1000 1024 1040; do gcc -O2 -DN=$n matmul.c -o m$n && echo "N=$n" && ./m$n | head -2; done
N=1000
ijk            0.50 s  check 4800051
ikj            0.34 s  check 4800051
N=1024
ijk            3.48 s  check 5030602
ikj            0.57 s  check 5030602
N=1040
ijk            0.68 s  check 5191721
ikj            0.33 s  check 5191721
```

With N = 1000 or 1040, `ijk` takes well under a second; with N = 1024 it takes five to seven times as long, while `ikj` changes much less. The simulator can explain it, and Cachegrind confirms it on the real binary at a smaller size, simulating a 32 KiB, 8-way L1:

```c
// file: host/ijk.c
// Only the naive ijk loop, for counting cache misses with perf.
#include <stdio.h>
#ifndef N
#define N 1024
#endif
static double A[N][N], B[N][N], C[N][N];
int main(void) {
    for (int i = 0; i < N; i++) for (int j = 0; j < N; j++) { A[i][j] = (i + j) % 7; B[i][j] = (i * j) % 5; }
    for (int i = 0; i < N; i++)
        for (int j = 0; j < N; j++) {
            double s = 0;
            for (int k = 0; k < N; k++) s += A[i][k] * B[k][j];
            C[i][j] = s;
        }
    double check = 0;
    for (int i = 0; i < N; i++) check += C[i][i];
    printf("N=%d check %.0f\n", N, check);
    return 0;
}
```

```console
$ cd host && for n in 250 256; do gcc -O2 -DN=$n ijk.c -o ijk$n && valgrind --tool=cachegrind --cache-sim=yes --cachegrind-out-file=/dev/null --I1=32768,8,64 --D1=32768,8,64 --LL=1048576,16,64 ./ijk$n 2>&1 | grep -E "check|D1  miss rate"; done
N=250 check 300045
==9195== D1  miss rate:       12.5% (      12.5%     +    14.1%  )
N=256 check 312120
==9202== D1  miss rate:       50.1% (      50.1%     +    44.8%  )
```

### Deep & Tricky

**DT-28. A power-of-two row length turns a column walk into a conflict-miss machine: with 256 doubles per row, consecutive elements of a column are 2,048 bytes apart, and in a 32 KiB 8-way cache (64 sets of 64-byte blocks, so addresses 4,096 bytes apart share a set) the whole column maps to only 2 sets — 16 blocks — instead of the 256 it needs; with 250 doubles per row the same column spreads over all 64 sets. The miss rate jumps from 12.5% to 50%, and on the real machine the N = 1024 multiply takes five to seven times as long as N = 1000, which does only 7% fewer operations.** *(Cache indexing by low address bits: Hennessy & Patterson, §B.1; the effect is the reason for array padding in Intel's optimisation manual, "Avoid cache conflicts…", and the classic "leading dimension" advice in LAPACK)* The fix is padding: declare `B[N][N + 8]` and use only N columns. Large power-of-two strides also collide in TLBs and DRAM banks, which is why FFT and image libraries pad.

### Pattern

**P-5. Make the innermost loop walk memory with stride 1; when it cannot, block the loops so that the data reused by the inner loops fits in a cache level, and never let a hot stride be a large power of two.** *(Practice; Lam, Rothberg & Wolf, "The Cache Performance and Optimizations of Blocked Algorithms", ASPLOS 1991)* *Trade-off:* blocking adds loop overhead and tuning parameters that depend on the machine's caches; libraries (BLAS, FFTW) choose them automatically.

### Mastery check

1. **Explain.** At N = 250, why is the miss rate 12.5% — which accesses miss and which hit?
2. **Predict.** In `ijk`, would storing `B` transposed (`Bt[j][k]`) help? Why?

## Unit 10.7 — Multi-level caches, inclusion, and prefetching

**One-line summary:** Real processors stack two or three cache levels — small fast private L1s split into instructions and data, a larger private L2, a big shared L3 — each with its own trade-off between hit time and miss rate; prefetchers (hardware stride and stream detectors, or software prefetch instructions) turn compulsory and capacity misses into hits by fetching blocks before they are asked for.

Design choices that recur:

- **Split L1 (I and D)** so that the pipeline can fetch and load in the same cycle; unified L2 and L3.
- **Inclusive** L3 (every block in an L1 or L2 is also in L3: simple coherence lookups, wasted capacity), **exclusive** (no duplication), or non-inclusive.
- **Non-blocking caches** with miss status holding registers (MSHRs) keep serving hits while several misses are outstanding — essential for the out-of-order cores of Chapter 7, which overlap misses (memory-level parallelism).
- **Prefetchers** detect strides per instruction (a table indexed by the load's PC, like a branch predictor) or streams of consecutive blocks. They are why the pointer chase of Chapter 0 had to be random, and why `ikj` above is so fast. A good prefetcher is timely (early enough), accurate (does not fetch unused blocks), and does not evict useful data (pollution).

### Mastery check

1. **Explain.** Why does a non-blocking cache matter more for an out-of-order core than for the in-order pipeline of Chapter 6?
2. **Explain.** Why can no hardware prefetcher help the linked-list walk of Chapter 0, and what could software do?

## Chapter 10 Project — A set-associative write-back cache in Verilog

### Requirements

Design a 2-way set-associative, write-back, write-allocate cache with LRU replacement, 16-byte blocks of four 32-bit words, and a parameter for the number of sets. The CPU side holds a request (`req`, `we`, `addr`, `wdata`) until `ready`; a hit answers in the same cycle. The memory side transfers whole blocks and acknowledges each request after a latency. Write a testbench that issues 20,000 random loads and stores with locality, checks every load against a flat reference memory, and reports the miss rate and the average cycles per access; run it for two sizes and two memory latencies, and synthesise the design.

### Reference solution

The controller has three states. IDLE serves hits; on a miss it goes to WB if the victim way is valid and dirty, else straight to FILL. WB writes the victim block to memory; FILL reads the new block, installs it, and returns to IDLE, where the still-held request now hits. One LRU bit per set names the way to evict next.

```verilog
// file: rtl/cache.v
// A 2-way set-associative, write-back, write-allocate cache with LRU replacement.
// 32-bit words, 4-word (16-byte) blocks, SETS sets. CPU side: hold req until ready.
// Memory side: block-wide reads and writes, each acknowledged by mem_ack after some cycles.
module cache #(parameter SETS = 32) (
    input  wire         clk, rst,
    input  wire         req, we,
    input  wire [31:0]  addr, wdata,
    output reg          ready,
    output reg  [31:0]  rdata,
    output reg          mem_req, mem_we,
    output reg  [31:0]  mem_addr,              // byte address of a block
    output reg  [127:0] mem_wdata,
    input  wire [127:0] mem_rdata,
    input  wire         mem_ack
);
    localparam IB = $clog2(SETS);
    localparam TB = 32 - 4 - IB;
    wire [1:0]    word  = addr[3:2];
    wire [IB-1:0] index = addr[4 +: IB];
    wire [TB-1:0] tag   = addr[31 -: TB];

    reg [TB-1:0]  tags  [0:1][0:SETS-1];
    reg [127:0]   data  [0:1][0:SETS-1];
    reg           valid [0:1][0:SETS-1];
    reg           dirty [0:1][0:SETS-1];
    reg           lru   [0:SETS-1];            // the way to evict next

    wire hit0 = valid[0][index] && tags[0][index] == tag;
    wire hit1 = valid[1][index] && tags[1][index] == tag;
    wire hit  = hit0 || hit1;
    wire hway = hit1;
    wire victim = lru[index];

    localparam IDLE = 0, WB = 1, FILL = 2;
    reg [1:0] state;
    integer s;

    always @(*) begin                          // CPU-side outputs: a hit answers in the same cycle
        ready = req && state == IDLE && hit;
        rdata = data[hway][index][32 * word +: 32];
        mem_req = state != IDLE;
        mem_we = state == WB;
        mem_addr = state == WB ? {tags[victim][index], index, 4'b0} : {tag, index, 4'b0};
        mem_wdata = data[victim][index];
    end

    always @(posedge clk) begin
        if (rst) begin
            state <= IDLE;
            for (s = 0; s < SETS; s = s + 1) begin
                valid[0][s] <= 0; valid[1][s] <= 0; dirty[0][s] <= 0; dirty[1][s] <= 0; lru[s] <= 0;
            end
        end else case (state)
            IDLE: if (req) begin
                if (hit) begin
                    lru[index] <= ~hway;                       // the other way is now least recent
                    if (we) begin
                        data[hway][index][32 * word +: 32] <= wdata;
                        dirty[hway][index] <= 1;
                    end
                end else
                    state <= valid[victim][index] && dirty[victim][index] ? WB : FILL;
            end
            WB: if (mem_ack) state <= FILL;                    // victim written back
            FILL: if (mem_ack) begin                           // block arrives: install it
                data[victim][index]  <= mem_rdata;
                tags[victim][index]  <= tag;
                valid[victim][index] <= 1;
                dirty[victim][index] <= 0;
                state <= IDLE;                                 // the retried access now hits
            end
        endcase
    end
endmodule
```

```verilog
// file: rtl/tb_cache.v
// Random loads and stores with locality, checked against a flat reference memory.
`ifndef SYNTHESIS
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    reg req = 0, we = 0;
    reg [31:0] addr = 0, wdata = 0;
    wire ready, mem_req, mem_we, mem_ack;
    wire [31:0] rdata, mem_addr;
    wire [127:0] mem_wdata;
    reg  [127:0] mem_rdata;
    parameter SETS = 32;
    cache #(SETS) dut(clk, rst, req, we, addr, wdata, ready, rdata,
                    mem_req, mem_we, mem_addr, mem_wdata, mem_rdata, mem_ack);

    // main memory: 64 KiB, answers a block request after LAT cycles
    parameter LAT = 20;
    reg [31:0] mem [0:16383];
    reg [31:0] ref_mem [0:16383];
    integer wait_cnt = 0;
    assign mem_ack = mem_req && wait_cnt == LAT;
    always @(posedge clk) begin
        wait_cnt <= mem_req && wait_cnt < LAT ? wait_cnt + 1 : 0;
        if (mem_ack && mem_we) {mem[mem_addr[15:2] + 3], mem[mem_addr[15:2] + 2],
                                 mem[mem_addr[15:2] + 1], mem[mem_addr[15:2]]} <= mem_wdata;
    end
    always @(*) mem_rdata = {mem[mem_addr[15:2] + 3], mem[mem_addr[15:2] + 2],
                             mem[mem_addr[15:2] + 1], mem[mem_addr[15:2]]};

    integer i, errors = 0, accesses = 0, misses = 0, cycles = 0, region, seed = 1;
    integer lastaddr;
    always @(posedge clk) if (!rst) cycles = cycles + 1;
    always @(posedge clk) if (dut.state == 0 && req && !dut.hit) misses = misses + 1;

    task access(input w, input [31:0] a, input [31:0] d);
        begin
            @(negedge clk) req = 1; we = w; addr = a; wdata = d;
            @(posedge clk); while (!ready) @(posedge clk);
            if (!w && rdata !== ref_mem[a[15:2]]) begin
                errors = errors + 1;
                if (errors < 5) $display("addr %h: read %h, expected %h", a, rdata, ref_mem[a[15:2]]);
            end
            if (w) ref_mem[a[15:2]] = d;
            accesses = accesses + 1;
        end
    endtask

    initial begin
        for (i = 0; i < 16384; i = i + 1) begin mem[i] = i * 3 + 7; ref_mem[i] = i * 3 + 7; end
        repeat (2) @(negedge clk); rst = 0;
        lastaddr = 0;
        for (i = 0; i < 20000; i = i + 1) begin
            // mostly near the previous address (spatial locality), sometimes a jump in 4 KiB,
            // rarely anywhere in 64 KiB (forces evictions of dirty blocks)
            region = $urandom(seed) % 100;  seed = seed + 1;
            if (region < 80)      lastaddr = (lastaddr + 4 * ($urandom % 4)) & 16'hfffc;
            else if (region < 97) lastaddr = $urandom % 4096 & 32'hfffc;
            else                  lastaddr = $urandom % 65536 & 32'hfffc;
            access($urandom % 4 == 0, lastaddr, $urandom);
        end
        $display("%5d-byte cache, memory latency %2d: %0d accesses, miss rate %0d.%0d%%, %0d.%02d cycles per access, %0d errors",
                 SETS * 32, LAT, accesses, 100 * misses / accesses, 1000 * misses / accesses % 10,
                 cycles / accesses, 100 * cycles / accesses % 100, errors);
        $finish;
    end
endmodule
`endif
```

```console
$ cd rtl && for p in "-Ptb.SETS=32" "-Ptb.SETS=128" "-Ptb.SETS=128 -Ptb.LAT=60"; do iverilog -g2012 $p -o sim cache.v tb_cache.v && vvp -n sim | grep -v finish; done
 1024-byte cache, memory latency 20: 20000 accesses, miss rate 40.6%, 13.60 cycles per access, 0 errors
 4096-byte cache, memory latency 20: 20000 accesses, miss rate 15.3%, 6.11 cycles per access, 0 errors
 4096-byte cache, memory latency 60: 20000 accesses, miss rate 15.3%, 15.57 cycles per access, 0 errors
$ cd rtl && yosys -p "read_verilog cache.v; synth -top cache; stat" | grep -E "Number of cells" | tail -1 | tr -s ' '
 Number of cells: 21077
```

Every load returns the value the reference memory holds, through thousands of dirty evictions. The numbers obey AMAT: quadrupling the cache from 1 KiB to 4 KiB cuts the miss rate from about 41% to 15% and the average access from about 14 cycles to 6; tripling the memory latency to 60 cycles with the same miss rate pushes it back up to about 16. The synthesised design is about 21,000 cells, almost all of them the data array built from flip-flops; a real cache uses SRAM macros for data and tags, and the controller is a few hundred cells.

### Deep & Tricky

**DT-29. A cache testbench that never evicts a dirty block cannot find write-back bugs, and one that compares loads only against values the testbench itself just wrote cannot find stale-data bugs: the reference model must be a complete memory image, the address stream must overflow every set (here the rare jumps across 64 KiB do that), and stores must be followed by loads of the same address after an eviction.** *(Practice: coverage of the controller's state transitions — IDLE→WB→FILL is the interesting path; Bergeron, "Writing Testbenches", ch. 6 on self-checking with reference models)* Try it: make the miss path go straight to FILL, so dirty victims are dropped, and the 1 KiB run reports 8,596 errors. Confine the addresses to 2 KiB and run the 4 KiB cache, and the same broken design reports 0 errors — nothing is ever evicted, so the missing write-back is never exercised.

### Review checklist

- `ready` depends combinationally on `req` and the tag comparison: what does that do to the CPU's critical path, and how would a pipelined cache split it?
- How would you add a write buffer so that WB and FILL overlap?
- What changes for a 4-way cache: how many LRU bits, and what replaces the single `lru` bit?

## Answers to the Chapter 10 prediction questions

**Unit 10.1, Q1.** 5% × 20% = 1% of all accesses.

**Unit 10.2, Q1.** 48 KiB / (12 × 64) = 64 sets, 6 index bits. **Q2.** Both map to set 0 (bits 6–10 are 0 for both): arrays 8 KiB apart collide in every set.

**Unit 10.3, Q1.** 16 KiB, 4-way (64 sets): a column's 64 blocks (4 blocks apart) use 16 sets × 4 ways = 64 slots, which just fit, and the whole 16 KiB matrix now fits too — the simulator reports only the 256 compulsory misses (3.1%). 16-way at 8 KiB does *not* help: 8 sets, of which a stride of 4 blocks uses only 2, so 32 slots for 64 blocks — still 100% misses. Only full associativity (128 ways) brings it down to 6.25%. Associativity cannot fix a stride that uses a fraction of the sets; a different layout can.

**Unit 10.4, Q2.** ⌈log₂ 8!⌉ = ⌈log₂ 40320⌉ = 16 bits; tree pseudo-LRU uses 7.

**Unit 10.5, Q2.** 256 blocks read, only to be completely overwritten. Processors detect full-block writes (streaming stores, `rep movs`, `dc zva` on ARM) and allocate without reading.

**Unit 10.6, Q1.** `A[i][k]` walks a row: one miss per 8 doubles. `B[k][j]` misses once per block when the column is first walked, but the next 7 columns (`j + 1`…) reuse those blocks because the 250 blocks fit in the cache — so half the accesses (to `B`) miss 1 in 8 times, and A misses 1 in 8 times: 12.5% overall. **Q2.** Yes: the inner loop then walks rows of both `A` and `Bt`, as `ikj` does.

# Chapter 11 — Virtual Memory

*Goal: understand how every load and store in a modern system is translated from a virtual to a physical address, and what that costs. The chapter implements RISC-V's Sv32 page tables and hardware page-table walker exactly as the privileged specification describes them, measures TLB reach in simulation and on the host CPU (where huge pages make random accesses up to three times faster), watches demand paging and copy-on-write through the page-fault counter, and ends with a small operating-system memory manager that implements both on top of the walker.*

*Accompanies the lecture "Virtual Memory" and the virtual-memory parts of "Memory Hierarchy".*

**What you will be able to do after this chapter**

- Explain the three jobs of virtual memory: isolation, relocation, and the illusion of more memory than exists.
- Translate an address through a two-level page table by hand and in code, including permission checks and superpages.
- Compute TLB reach and explain why huge pages and ASIDs matter.
- Explain demand paging and copy-on-write, and measure their cost.
- Explain how caches and translation interact (VIPT caches) and why the L1 cache size is tied to the page size.

## Topic 11A — Translation

## Unit 11.1 — Why virtual memory

**One-line summary:** Each process sees its own private virtual address space; hardware translates every virtual address to a physical one using page tables that only the operating system can change, which isolates processes from each other, lets programs be loaded anywhere, and lets the OS keep only the pages in use in physical memory.

Memory is divided into fixed-size **pages** (4 KiB on RISC-V, x86, and most ARM systems). A virtual address is split into a virtual page number (VPN) and an offset within the page; translation replaces the VPN by a physical page number (PPN, the number of a **frame**) and keeps the offset. Each page-table entry also carries permissions (read, write, execute, user-accessible), so the same mechanism enforces protection: a store to a read-only page or a user access to kernel memory raises a **page fault**, an exception handled by the OS.

| Job | How paging does it |
| --- | --- |
| Isolation | Each process has its own page table; a page not mapped in it cannot be reached at all |
| Relocation | Every program can be linked at the same virtual addresses; the frames can be anywhere |
| More memory than exists | Pages not in use need no frame; the OS fills them on first use, or brings them back from disk |
| Sharing | Two page tables can point to the same frame: shared libraries, shared memory, copy-on-write |

### Mastery check

1. **Calculate.** With 4 KiB pages and 32-bit virtual addresses, how many bits are the offset and the VPN? How many entries would a single flat page table need, and how big would it be with 4-byte entries?

## Unit 11.2 — Multi-level page tables: RISC-V Sv32

**One-line summary:** A flat page table for every process would be huge, so page tables are trees: Sv32 splits the 20-bit VPN into two 10-bit halves, the first indexing a 1,024-entry root table whose entries point to second-level tables, each a 4 KiB page; unused regions of the address space need no second-level tables, and a leaf at the first level maps a 4 MiB "megapage".

An Sv32 page-table entry (PTE) is 32 bits: bits 0–7 are the flags **V**alid, **R**ead, **W**rite, e**X**ecute, **U**ser, **G**lobal, **A**ccessed, **D**irty; bits 8–9 are reserved for software; bits 10–31 are the PPN. A PTE with R = W = X = 0 is a *pointer* to the next level; any other valid PTE is a *leaf*. The root table's physical page number is held in the `satp` register, together with the address-space identifier (ASID).

The walker below follows the specification's algorithm step by step: read the PTE, check it is valid and well-formed, stop at a leaf or descend, check the permissions, check superpage alignment, set the A and D bits, and form the physical address. The `AddressSpace` class is the operating system's side — building the tables — and `TLB` caches translations (Unit 11.3).

```python
# file: mmu/sv32.py
"""RISC-V Sv32 virtual memory: page tables in simulated physical memory, a hardware page-table
walker following the privileged specification (section 4.3.2), and a TLB with ASIDs."""
from collections import OrderedDict

V, R, W, X, U, G, A, D = (1 << i for i in range(8))
PAGE = 4096

class PageFault(Exception):
    pass

class Memory:
    def __init__(self, size):
        self.b = bytearray(size)
        self.next_free = PAGE                 # frame 0 is never handed out
        self.reads = 0
    def alloc_frame(self):
        f, self.next_free = self.next_free, self.next_free + PAGE
        return f
    def r32(self, a):
        self.reads += 1
        return int.from_bytes(self.b[a:a + 4], "little")
    def w32(self, a, v):
        self.b[a:a + 4] = v.to_bytes(4, "little")

class AddressSpace:
    """What an operating system does: build a two-level page table."""
    def __init__(self, mem, asid):
        self.mem, self.asid = mem, asid
        self.root = mem.alloc_frame()
    def satp(self):
        return 1 << 31 | self.asid << 22 | self.root >> 12          # MODE = Sv32
    def map(self, va, pa, flags, mega=False):
        vpn1, vpn0 = va >> 22, va >> 12 & 0x3FF
        l1 = self.root + 4 * vpn1
        if mega:                                                    # a 4 MiB leaf at level 1
            self.mem.w32(l1, (pa >> 12) << 10 | flags | V)
            return
        pte = self.mem.r32(l1)
        if not pte & V:                                             # allocate a level-0 table
            pte = (self.mem.alloc_frame() >> 12) << 10 | V          # pointer: V=1, R=W=X=0
            self.mem.w32(l1, pte)
        self.mem.w32((pte >> 10 << 12) + 4 * vpn0, (pa >> 12) << 10 | flags | V)

def walk(mem, satp, va, access, user):
    """The hardware walker. access is 'r', 'w' or 'x'. Returns (physical address, level, pte flags)."""
    a = (satp & 0x3FFFFF) << 12
    vpn = [va >> 12 & 0x3FF, va >> 22]
    for i in (1, 0):
        pte_addr = a + 4 * vpn[i]
        pte = mem.r32(pte_addr)
        if not pte & V or (not pte & R and pte & W):
            raise PageFault(f"invalid PTE at level {i}")
        if pte & (R | X):
            break                                                   # a leaf
        if i == 0:
            raise PageFault("pointer PTE at level 0")
        a = (pte >> 10) << 12
    need = {"r": R, "w": W, "x": X}[access]
    if not pte & need:
        raise PageFault(f"no {access} permission")
    if user and not pte & U:
        raise PageFault("user access to a supervisor page")
    if not user and pte & U:
        raise PageFault("supervisor access to a user page (SUM=0)")
    if i == 1 and (pte >> 10) & 0x3FF:
        raise PageFault("misaligned megapage")
    new = pte | A | (D if access == "w" else 0)                     # hardware A/D update (Svadu)
    if new != pte:
        mem.w32(pte_addr, new)
    ppn = pte >> 10
    if i == 1:
        pa = (ppn >> 10) << 22 | va & 0x3FFFFF                      # superpage: keep VPN[0] and offset
    else:
        pa = ppn << 12 | va & 0xFFF
    return pa, i, new & 0xFF

class TLB:
    def __init__(self, entries):
        self.n, self.e = entries, OrderedDict()                     # (asid, vpn, level) -> (ppn base, flags)
        self.hits = self.misses = 0
    def translate(self, mem, satp, va, access, user):
        asid = satp >> 22 & 0x1FF
        for level, shift in ((0, 12), (1, 22)):
            for key in ((asid, va >> shift, level), (None, va >> shift, level)):   # None = global
                if key in self.e:
                    base, flags = self.e[key]
                    if {"r": R, "w": W, "x": X}[access] & flags and bool(flags & U) == user \
                            and (access != "w" or flags & D):
                        self.hits += 1
                        self.e.move_to_end(key)
                        return base | va & ((1 << shift) - 1)
        self.misses += 1
        pa, level, flags = walk(mem, satp, va, access, user)
        shift = 22 if level else 12
        key = (None if flags & G else asid, va >> shift, level)
        if len(self.e) == self.n:
            self.e.popitem(last=False)                               # evict least recently used
        self.e[key] = (pa >> shift << shift, flags)
        return pa
```

```python
# file: mmu/demo.py
from sv32 import *

mem = Memory(64 << 20)
p1 = AddressSpace(mem, asid=1)
code, data, kern = mem.alloc_frame(), mem.alloc_frame(), mem.alloc_frame()
p1.map(0x0001_0000, code, R | X | U)                    # user code: read + execute
p1.map(0x0002_0000, data, R | W | U)                    # user data: read + write
p1.map(0xC000_0000, kern, R | W | G)                    # kernel data, global, supervisor only
p1.map(0x8000_0000, 0x0040_0000, R | W | X | G, mega=True)  # 4 MiB kernel megapage at PA 4 MiB

tests = [("user fetch from code page", 0x0001_0ABC, "x", True),
         ("user store to data page", 0x0002_0010, "w", True),
         ("user store to code page", 0x0001_0000, "w", True),
         ("user load from kernel page", 0xC000_0000, "r", True),
         ("kernel load from kernel page", 0xC000_0123, "r", False),
         ("kernel load inside megapage", 0x8012_3456, "r", False),
         ("user load from unmapped page", 0x0003_0000, "r", True)]
for name, va, acc, user in tests:
    before = mem.reads
    try:
        pa, level, flags = walk(mem, p1.satp(), va, acc, user)
        print(f"{name:30} VA {va:08x} -> PA {pa:08x}  level {level}, "
              f"{mem.reads - before} PTE reads, A={flags >> 6 & 1} D={flags >> 7 & 1}")
    except PageFault as e:
        print(f"{name:30} VA {va:08x} -> page fault: {e}")
```

```console
$ cd mmu && python3 demo.py
user fetch from code page      VA 00010abc -> PA 00002abc  level 0, 2 PTE reads, A=1 D=0
user store to data page        VA 00020010 -> PA 00003010  level 0, 2 PTE reads, A=1 D=1
user store to code page        VA 00010000 -> page fault: no w permission
user load from kernel page     VA c0000000 -> page fault: user access to a supervisor page
kernel load from kernel page   VA c0000123 -> PA 00004123  level 0, 2 PTE reads, A=1 D=0
kernel load inside megapage    VA 80123456 -> PA 00523456  level 1, 1 PTE reads, A=1 D=0
user load from unmapped page   VA 00030000 -> page fault: invalid PTE at level 0
```

**Worked example.** VA 0x0001_0ABC: VPN[1] = 0x0001_0ABC >> 22 = 0; VPN[0] = (0x0001_0ABC >> 12) & 0x3FF = 0x10; offset 0xABC. The walker reads root[0] (a pointer), then entry 0x10 of the level-0 table (a leaf with PPN 2), and produces PA = 2 × 4096 + 0xABC = 0x2ABC — two memory reads before the actual access. The megapage translation needs only one: root[0x200] is a leaf, and the PA keeps all 22 low bits of the VA.

### Deep & Tricky

**DT-30. The hardware sets the Accessed and Dirty bits — or, on implementations without hardware update, raises a page fault so that software can set them — and the OS depends on them: A tells the page-replacement algorithm which pages were used recently, and D tells it which pages must be written to disk before their frame is reused. A walker that sets D on a load, or forgets it on a store, silently corrupts files.** *(RISC-V Privileged Specification, §4.3.1–4.3.2 and the Svade/Svadu extensions; Intel SDM Vol. 3A §4.8)* Note in the output: the store set D=1 on the data page; the instruction fetch and the loads set only A.

### Mastery check

1. **Calculate.** Translate VA 0xC000_0123 by hand in the demo's address space. Which PTEs are read?
2. **Explain.** Why must a megapage's PPN[0] be zero? What would the alternative force the hardware to do?
3. **Calculate.** A process uses 1 MiB of code at 0x0001_0000, 1 MiB of heap at 0x1000_0000, and 64 KiB of stack just below 0x8000_0000. How many page-table pages does Sv32 need?

## Unit 11.3 — TLBs and TLB reach

**One-line summary:** A page-table walk costs extra memory accesses on every load, so processors cache translations in a translation lookaside buffer (TLB) — a small, highly associative cache of PTEs; its *reach* (entries × page size) is tiny with 4 KiB pages (64 entries cover 256 KiB), which is why large-memory programs benefit from huge pages, and ASIDs let the TLB keep the entries of several processes.

```python
# file: mmu/reach.py
"""TLB reach: how much memory a TLB covers, and what megapages and ASIDs change."""
import random
from sv32 import *

def setup(mega, span, asid=1, mem=None):
    mem = mem or Memory(96 << 20)
    asp = AddressSpace(mem, asid)
    if mega:
        for off in range(0, span, 1 << 22):
            asp.map(0x0040_0000 + off, 0x0100_0000 + off, R | W | U, mega=True)
    else:
        for off in range(0, span, PAGE):
            asp.map(0x0040_0000 + off, 0x0100_0000 + off, R | W | U)
    return mem, asp

rng = random.Random(1)
print(f"{'working set':>12} {'pages':>8} {'TLB entries':>12} {'TLB miss %':>11} {'PTE reads per access':>21}")
for span in (64 << 10, 1 << 20, 16 << 20):
    trace = [0x0040_0000 + rng.randrange(span) & ~3 for _ in range(20000)]
    for mega in (False, True):
        for entries in (16, 64):
            mem, asp = setup(mega, span)
            tlb, before = TLB(entries), mem.reads
            for va in trace:
                tlb.translate(mem, asp.satp(), va, "r", True)
            print(f"{span >> 10:9d} KiB {'4 MiB' if mega else '4 KiB':>8} {entries:12d} "
                  f"{100 * tlb.misses / len(trace):10.1f}% {(mem.reads - before) / len(trace):21.2f}")

# two processes take turns, 200 accesses each, 50 times
mem, p1 = setup(False, 128 << 10, asid=1)
mem, p2 = setup(False, 128 << 10, asid=2, mem=mem)
for use_asid in (False, True):
    tlb = TLB(64)
    for turn in range(100):
        p = (p1, p2)[turn % 2]
        if not use_asid:
            tlb.e.clear()                         # without ASIDs the OS must flush on every switch
        for _ in range(200):
            tlb.translate(mem, p.satp(), 0x0040_0000 + rng.randrange(128 << 10) & ~3, "r", True)
    print(f"two processes, 32 pages each, 64-entry TLB, {'ASIDs   ' if use_asid else 'flushing'}: "
          f"{100 * tlb.misses / (tlb.hits + tlb.misses):.1f}% TLB misses")
```

```console
$ cd mmu && python3 reach.py
 working set    pages  TLB entries  TLB miss %  PTE reads per access
       64 KiB    4 KiB           16        0.1%                  0.00
       64 KiB    4 KiB           64        0.1%                  0.00
       64 KiB    4 MiB           16        0.0%                  0.00
       64 KiB    4 MiB           64        0.0%                  0.00
     1024 KiB    4 KiB           16       93.6%                  1.87
     1024 KiB    4 KiB           64       74.6%                  1.49
     1024 KiB    4 MiB           16        0.0%                  0.00
     1024 KiB    4 MiB           64        0.0%                  0.00
    16384 KiB    4 KiB           16       99.5%                  1.99
    16384 KiB    4 KiB           64       98.4%                  1.97
    16384 KiB    4 MiB           16        0.0%                  0.00
    16384 KiB    4 MiB           64        0.0%                  0.00
two processes, 32 pages each, 64-entry TLB, flushing: 16.0% TLB misses
two processes, 32 pages each, 64-entry TLB, ASIDs   : 0.3% TLB misses
```

With 4 KiB pages, a 64-entry TLB covers 256 KiB: a random working set of 1 MiB misses 75% of the time and needs about 1.5 extra memory reads per access; at 16 MiB nearly every access walks. With 4 MiB megapages, all three working sets fit in a handful of entries. Without ASIDs the OS must flush the TLB on every process switch, and each process starts cold (16% misses); with ASIDs the entries of both processes survive.

The host CPU tells the same story. The program below follows a random chain through an increasing number of pages, one 64-byte line per page, once with 4 KiB pages and once with 2 MiB transparent huge pages:

```c
// file: host/tlb.c
// Random accesses touching one int per page versus the same number of accesses in few pages,
// with 4 KiB pages and (if the kernel grants them) 2 MiB transparent huge pages.
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <sys/mman.h>
#include <time.h>

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

static double walk(char *buf, size_t pages, size_t step, int iters) {
    // a random cyclic permutation of pages; each visit loads one int at a fixed offset of the page
    size_t *order = malloc(pages * sizeof *order);
    for (size_t i = 0; i < pages; i++) order[i] = i;
    srand(1);
    for (size_t i = pages - 1; i > 0; i--) { size_t j = rand() % (i + 1); size_t t = order[i]; order[i] = order[j]; order[j] = t; }
    // a random 64-byte line in each page, so that the lines spread over all cache sets
    size_t *line = malloc(pages * sizeof *line);
    for (size_t i = 0; i < pages; i++) line[i] = (rand() % (step / 64)) * 64;
#define AT(k) (buf + order[k] * step + line[k])
    for (size_t i = 0; i < pages; i++)                       // store "next" pointers: a dependent chain
        *(char **)AT(i) = AT((i + 1) % pages);
    char *p = AT(0);
    free(order); free(line);
    double t = now();
    for (int i = 0; i < iters; i++) p = *(char **)p;
    t = now() - t;
    if (p == 0) puts("");                                    // keep the chain alive
    return t / iters * 1e9;
}

int main(void) {
    const size_t SIZE = 1UL << 30;                           // 1 GiB
    for (int huge = 0; huge <= 1; huge++) {
        char *buf = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        madvise(buf, SIZE, huge ? MADV_HUGEPAGE : MADV_NOHUGEPAGE);
        memset(buf, 1, SIZE);
        long anon_huge = 0;
        FILE *sm = fopen("/proc/self/smaps_rollup", "r");
        char l[256];
        while (sm && fgets(l, sizeof l, sm)) sscanf(l, "AnonHugePages: %ld", &anon_huge);
        if (sm) fclose(sm);
        printf("%s pages requested (%ld MiB of huge pages granted):\n", huge ? "2 MiB huge" : "4 KiB", anon_huge >> 10);
        for (size_t pages = 16; pages <= 65536 * 4; pages *= 4)
            printf("  %7zu blocks, one per 4 KiB page (%5zu MiB span): %5.1f ns per load\n",
                   pages, pages * 4096 >> 20, walk(buf, pages, 4096, 20000000));
        munmap(buf, SIZE);
    }
    FILE *f = fopen("/sys/kernel/mm/transparent_hugepage/enabled", "r");
    char line[128] = "";
    if (f) { if (!fgets(line, sizeof line, f)) line[0] = 0; fclose(f); }
    printf("transparent_hugepage: %s", line);
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall tlb.c -o tlb && ./tlb
4 KiB pages requested (0 MiB of huge pages granted):
       16 blocks, one per 4 KiB page (    0 MiB span):   1.3 ns per load
       64 blocks, one per 4 KiB page (    0 MiB span):   1.3 ns per load
      256 blocks, one per 4 KiB page (    1 MiB span):   3.2 ns per load
     1024 blocks, one per 4 KiB page (    4 MiB span):   5.6 ns per load
     4096 blocks, one per 4 KiB page (   16 MiB span):  13.0 ns per load
    16384 blocks, one per 4 KiB page (   64 MiB span):  19.0 ns per load
    65536 blocks, one per 4 KiB page (  256 MiB span): 240.9 ns per load
   262144 blocks, one per 4 KiB page ( 1024 MiB span): 319.5 ns per load
2 MiB huge pages requested (1024 MiB of huge pages granted):
       16 blocks, one per 4 KiB page (    0 MiB span):   1.5 ns per load
       64 blocks, one per 4 KiB page (    0 MiB span):   1.3 ns per load
      256 blocks, one per 4 KiB page (    1 MiB span):   1.3 ns per load
     1024 blocks, one per 4 KiB page (    4 MiB span):   4.2 ns per load
     4096 blocks, one per 4 KiB page (   16 MiB span):   5.1 ns per load
    16384 blocks, one per 4 KiB page (   64 MiB span):  10.2 ns per load
    65536 blocks, one per 4 KiB page (  256 MiB span): 107.4 ns per load
   262144 blocks, one per 4 KiB page ( 1024 MiB span): 234.9 ns per load
transparent_hugepage: always [madvise] never
```

Up to 64 pages, both fit in the first-level TLB and the lines fit in L1. From 256 pages on, the 4 KiB version pays for second-level TLB lookups and then page walks, on top of the cache misses that both versions have; with huge pages the same chain is two to three times faster until the data itself falls out of the caches, where DRAM latency dominates both. (Huge pages must be requested with `madvise` here, because this system's setting is `[madvise]`; the program checks `/proc/self/smaps_rollup` to confirm the kernel granted them.)

### Deep & Tricky

**DT-31. A page walk's memory accesses go through the data caches, so a TLB miss whose PTEs are cached costs a few nanoseconds, while one whose PTEs are in DRAM costs several DRAM latencies in series — and in a virtual machine translation is two-dimensional: each of the guest's page-table pointers is itself a guest-physical address that must be translated through the hypervisor's tables, so a 4-level guest walk on a 4-level host can take up to 24 memory accesses.** *(Bhargava et al., "Accelerating Two-Dimensional Page Walks for Virtualized Systems", ASPLOS 2008; Intel SDM Vol. 3C ch. 29, EPT)* This book's measurements run inside a virtual machine; the hardware's page-walk caches and the hypervisor's own huge pages hide much of the cost, but not all — the huge-page speedup above includes both dimensions.

### Mastery check

1. **Calculate.** What is the reach of a 1,536-entry second-level TLB with 4 KiB pages? With 2 MiB pages?
2. **Explain.** Why does the program put each chain element on a *random* line of its page, rather than always at offset 0? (Hint: Chapter 10, DT-28.)

## Topic 11B — The operating system's side

## Unit 11.4 — Page faults, demand paging, and copy-on-write

**One-line summary:** The OS fills page tables lazily: `mmap` and `malloc` only reserve virtual addresses, the first touch of each page faults and the kernel allocates a frame (demand paging), and `fork` copies no memory at all — parent and child share every page read-only, and the first write to a shared page faults and copies just that page (copy-on-write).

```c
// file: host/faults.c
// Demand paging and copy-on-write, observed through the page-fault counter.
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/resource.h>
#include <sys/wait.h>
#include <time.h>
#include <unistd.h>

static long minor_faults(void) { struct rusage u; getrusage(RUSAGE_SELF, &u); return u.ru_minflt; }
static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

int main(void) {
    const size_t SIZE = 64UL << 20, PAGE = 4096;               // 64 MiB = 16,384 pages
    long f0 = minor_faults();
    char *buf = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    madvise(buf, SIZE, MADV_NOHUGEPAGE);
    printf("after mmap of 64 MiB:          %6ld page faults\n", minor_faults() - f0);

    f0 = minor_faults();
    volatile char sum = 0;
    for (size_t i = 0; i < SIZE; i += PAGE) sum += buf[i];      // read every page
    printf("after reading every page:      %6ld page faults\n", minor_faults() - f0);

    f0 = minor_faults();
    double t = now();
    for (size_t i = 0; i < SIZE; i += PAGE) buf[i] = 1;         // write every page
    t = now() - t;
    printf("after writing every page:      %6ld page faults (%.2f us each)\n", minor_faults() - f0,
           t / (SIZE / PAGE) * 1e6);

    f0 = minor_faults();
    for (size_t i = 0; i < SIZE; i += PAGE) buf[i] = 2;         // write again: pages are present
    printf("after writing every page again: %5ld page faults\n", minor_faults() - f0);

    fflush(stdout);
    pid_t pid = fork();
    if (pid == 0) {                                              // child shares the pages, read-only
        long c0 = minor_faults();
        for (size_t i = 0; i < SIZE / 2; i += PAGE) buf[i] = 3;  // writing half of them copies them
        printf("child, writing half the pages:  %5ld page faults (copy-on-write)\n", minor_faults() - c0);
        fflush(stdout);
        _exit(0);
    }
    waitpid(pid, NULL, 0);
    printf("parent still sees buf[0] = %d\n", buf[0]);
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall faults.c -o faults && ./faults
after mmap of 64 MiB:               0 page faults
after reading every page:       16384 page faults
after writing every page:       16386 page faults (2.07 us each)
after writing every page again:     0 page faults
child, writing half the pages:   8192 page faults (copy-on-write)
parent still sees buf[0] = 2
```

Reserving 64 MiB costs nothing. Reading every page faults once per page — and then *writing* every page faults again, once per page, at about 2 µs each. The second write pass costs nothing; the child's writes after `fork` fault once per page written, and the parent's data is untouched.

### Deep & Tricky

**DT-32. Reading a never-written anonymous page does not allocate memory: Linux maps it to a single shared, read-only page of zeros, so the read faults, and the first write faults *again* to replace the zero page with a private frame — which is why the program above takes 16,384 faults for the reads and another 16,384 for the writes. Benchmarks that "warm up" a buffer by reading it measure the zero page, not their memory.** *(Linux kernel, mm/memory.c, `do_anonymous_page`: "Use the zero-page for reads"; Gorman, "Understanding the Linux Virtual Memory Manager", ch. 4)* Warm buffers by writing them (or `memset`), or with `MAP_POPULATE`.

### Mastery check

1. **Explain.** Why does `fork` followed immediately by `exec` (how shells start programs) cost almost nothing, even for a parent using gigabytes?
2. **Predict.** If the parent wrote to every page *after* the fork, while the child was still alive, how many faults would it take?

## Unit 11.5 — Caches and translation together

**One-line summary:** Translating before the L1 cache lookup would add the TLB to every load's critical path, so L1 caches are virtually indexed and physically tagged (VIPT): the set index comes from the page-offset bits, which translation does not change, so the TLB lookup and the cache lookup run in parallel — and this limits an L1 cache to (page size × associativity) bytes.

```console
$ for i in /sys/devices/system/cpu/cpu0/cache/index*; do echo "L$(cat $i/level) $(cat $i/type): $(cat $i/size), $(cat $i/ways_of_associativity)-way, $(cat $i/number_of_sets) sets, $(cat $i/coherency_line_size)-byte lines"; done
L1 Data: 48K, 12-way, 64 sets, 64-byte lines
L1 Instruction: 64K, 16-way, 64 sets, 64-byte lines
L2 Unified: 2048K, 16-way, 2048 sets, 64-byte lines
L3 Unified: 516096K, 16-way, 516096 sets, 64-byte lines
```

The L1 caches of this processor have exactly 64 sets of 64-byte lines: 64 × 64 = 4,096 bytes per way — one page. Index and offset come entirely from the 12 page-offset bits. To grow the L1 data cache from 32 KiB (8-way) to 48 KiB, Intel had to go to 12 ways; Apple's M1 has 128 KiB L1 data caches because its pages are 16 KiB (16 KiB × 8 ways). The L2 cache (2,048 sets) is physically indexed and is accessed after translation.

### Deep & Tricky

**DT-33. If a virtually indexed cache used index bits above the page offset, two virtual addresses that map to the same physical address (synonyms, created by shared memory or `mmap`ing a file twice) could sit in two different sets, and a write through one would not be seen through the other. Keeping index bits inside the page offset makes synonyms impossible by construction; caches that break the rule must detect synonyms in hardware or rely on the OS to colour pages.** *(Hennessy & Patterson, §B.3 "Avoiding address translation during indexing of the cache"; Cekleov & Dubois, "Virtual-Address Caches", IEEE Micro 1997)*

### Mastery check

1. **Calculate.** A designer wants a 64 KiB, VIPT L1 cache with 4 KiB pages. What associativity is required? What does that cost?

## Unit 11.6 — Protection and its failures

**One-line summary:** The user/supervisor bit stops user code from *architecturally* reading kernel memory, but Meltdown (2018) showed that some out-of-order processors performed the load speculatively before checking the permission, leaving the data's trace in the cache; the fix, kernel page-table isolation, removes the kernel from the user page table entirely, at the cost of a page-table switch on every system call.

Chapter 0's flush+reload measurement showed how a cache line's presence can be read through timing; Chapter 7 showed that faults are taken only when the faulting instruction retires, by which time younger instructions may have run. Meltdown combined the two: a user load from a kernel address faults at retirement, but its value was forwarded to dependent instructions first, and one of them touched a cache line indexed by the secret. Processors since 2019 stop the forwarding in hardware. KPTI made system calls measurably slower on affected processors — a direct cost of keeping isolation correct — and PCIDs (x86's ASIDs) limit that cost by avoiding a full TLB flush on each switch.

### Mastery check

1. **Explain.** Why does KPTI make system-call-heavy workloads slower, and how do ASIDs/PCIDs reduce the cost?

## Chapter 11 Project — A copy-on-write memory manager

### Requirements

Using the Sv32 walker of Unit 11.2, write the memory-management part of a tiny OS kernel:

- A process has a list of permitted regions (start, end, permissions); nothing is mapped at first.
- `access(process, va, kind)` uses the hardware walker; on a page fault the kernel handles it and retries once.
- First touch of a permitted page allocates a zeroed frame (demand paging).
- `fork` shares every mapped page between parent and child, read-only, marking formerly writable pages copy-on-write with a software PTE bit and counting references per frame.
- A write to a copy-on-write page copies it if the frame is still shared, or simply restores write permission if this process is the last user.
- Accesses outside the regions, or writes to read-only regions, are segmentation faults.

Demonstrate each case and count frames.

### Reference solution

```python
# file: project/kernel.py
"""A tiny operating-system memory manager on top of the Sv32 walker: demand paging,
fork with copy-on-write, and segmentation faults — all driven by page faults."""
import sys
sys.path.insert(0, "../mmu")
from sv32 import *

COW = 1 << 8                     # one of the two PTE bits reserved for software (RSW)

class Process:
    def __init__(self, kernel, asid):
        self.aspace = AddressSpace(kernel.mem, asid)
        self.regions = []        # (start, end, flags) the program is allowed to use
        self.pid = asid

class Kernel:
    def __init__(self):
        self.mem = Memory(32 << 20)
        self.refs = {}           # frame -> number of page-table entries that point to it
        self.stats = {"demand": 0, "cow_copy": 0, "cow_reuse": 0, "segv": 0}

    def pte_addr(self, proc, va):
        """Software walk to the level-0 PTE's address (allocating tables), as an OS does."""
        l1 = proc.aspace.root + 4 * (va >> 22)
        pte = self.mem.r32(l1)
        if not pte & V:
            pte = (self.mem.alloc_frame() >> 12) << 10 | V
            self.mem.w32(l1, pte)
        return (pte >> 10 << 12) + 4 * (va >> 12 & 0x3FF)

    def page_fault(self, proc, va, access):
        region = next((r for r in proc.regions if r[0] <= va < r[1]), None)
        if region is None or (access == "w" and not region[2] & W):
            self.stats["segv"] += 1
            raise PageFault(f"segmentation fault: pid {proc.pid} {access} at {va:08x}")
        pa_pte = self.pte_addr(proc, va)
        pte = self.mem.r32(pa_pte)
        if not pte & V:                                       # first touch: demand paging
            frame = self.mem.alloc_frame()                    # (frames start zeroed)
            self.refs[frame] = 1
            self.mem.w32(pa_pte, (frame >> 12) << 10 | region[2] | U | V)
            self.stats["demand"] += 1
        elif access == "w" and pte & COW:                     # a write to a shared page
            frame = pte >> 10 << 12
            if self.refs[frame] > 1:                          # still shared: copy it
                new = self.mem.alloc_frame()
                self.mem.b[new:new + PAGE] = self.mem.b[frame:frame + PAGE]
                self.refs[frame] -= 1
                self.refs[new] = 1
                frame = new
                self.stats["cow_copy"] += 1
            else:                                             # last user: just make it writable
                self.stats["cow_reuse"] += 1
            self.mem.w32(pa_pte, (frame >> 12) << 10 | (pte & 0xFF) | W)
        else:
            raise PageFault(f"unexpected fault at {va:08x}")
        # a real kernel would now execute sfence.vma to drop stale TLB entries

    def access(self, proc, va, access, value=None):
        for _ in range(2):                                    # try, fault, retry once
            try:
                pa, _, _ = walk(self.mem, proc.aspace.satp(), va, access, user=True)
                break
            except PageFault:
                self.page_fault(proc, va, access)
        if access == "w":
            self.mem.w32(pa, value)
        return self.mem.r32(pa)

    def fork(self, parent, asid):
        child = Process(self, asid)
        child.regions = list(parent.regions)
        for start, end, _ in parent.regions:
            for va in range(start, end, PAGE):
                p = self.pte_addr(parent, va)
                pte = self.mem.r32(p)
                if pte & V:                                   # share the frame, read-only, both sides
                    shared = (pte & ~W) | (COW if pte & W else 0)
                    self.mem.w32(p, shared)
                    self.mem.w32(self.pte_addr(child, va), shared)
                    self.refs[pte >> 10 << 12] += 1
        return child
```

```python
# file: project/run.py
from kernel import *

k = Kernel()
parent = Process(k, asid=1)
HEAP = 0x1000_0000
parent.regions.append((HEAP, HEAP + 16 * PAGE, R | W))          # a 64 KiB heap, nothing mapped yet
parent.regions.append((0x0001_0000, 0x0001_0000 + PAGE, R))     # a read-only page

def frames():
    return (k.mem.next_free - PAGE) // PAGE

for i in range(8):                                               # touch 8 of the 16 heap pages
    k.access(parent, HEAP + i * PAGE, "w", 100 + i)
print(f"parent wrote 8 pages: {k.stats}, frames in use {frames()}")

child = k.fork(parent, asid=2)
print(f"after fork:           {k.stats}, frames in use {frames()}")
for i in (0, 1):
    k.access(child, HEAP + i * PAGE, "w", 900 + i)               # child writes 2 shared pages
print(f"child wrote 2 pages:  {k.stats}, frames in use {frames()}")
print("parent reads", [k.access(parent, HEAP + i * PAGE, "r") for i in range(4)],
      " child reads", [k.access(child, HEAP + i * PAGE, "r") for i in range(4)])

k.access(parent, HEAP + 0 * PAGE, "w", 7)                        # parent is now the only user
k.access(parent, HEAP + 5 * PAGE, "w", 8)                        # still shared with the child
print(f"parent wrote 2 pages: {k.stats}, frames in use {frames()}")
print("child still reads page 5 as", k.access(child, HEAP + 5 * PAGE, "r"))

for va, acc in ((HEAP + 20 * PAGE, "r"), (0x0001_0000, "w")):
    try:
        k.access(parent, va, acc, 1)
    except PageFault as e:
        print(e)
print(f"final: {k.stats}")
```

```console
$ cd project && python3 run.py
parent wrote 8 pages: {'demand': 8, 'cow_copy': 0, 'cow_reuse': 0, 'segv': 0}, frames in use 10
after fork:           {'demand': 8, 'cow_copy': 0, 'cow_reuse': 0, 'segv': 0}, frames in use 13
child wrote 2 pages:  {'demand': 8, 'cow_copy': 2, 'cow_reuse': 0, 'segv': 0}, frames in use 15
parent reads [100, 101, 102, 103]  child reads [900, 901, 102, 103]
parent wrote 2 pages: {'demand': 8, 'cow_copy': 3, 'cow_reuse': 1, 'segv': 0}, frames in use 16
child still reads page 5 as 105
segmentation fault: pid 1 r at 10014000
segmentation fault: pid 1 w at 00010000
final: {'demand': 8, 'cow_copy': 3, 'cow_reuse': 1, 'segv': 2}
```

Eight writes cause eight demand-paging faults; the 10 frames in use are those 8 pages plus the root table and one level-0 table. `fork` copies no data: the three new frames are page tables (the child's root and level-0 table, and the parent's level-0 table for the read-only region, created while walking it). The child's two writes copy two pages; each side then sees its own values. When the parent writes page 0 — which the child has already copied away — the frame has one user left and is reused without copying (`cow_reuse`); page 5 is still shared and is copied. Both illegal accesses become segmentation faults.

### Review checklist

- Where must the kernel execute `sfence.vma` (flush stale TLB entries), and for which ASIDs, after `fork` and after each fault?
- What happens to the reference counts when a process exits? Write `exit()`.
- How would you add swapping: choose a victim using the A bits, write it out if D is set, and clear V?

## Answers to the Chapter 11 prediction questions

**Unit 11.1, Q1.** 12 offset bits, 20 VPN bits: 2²⁰ = 1,048,576 entries × 4 bytes = 4 MiB per process.

**Unit 11.2, Q1.** VPN[1] = 0x300, VPN[0] = 0; root[0x300] is a pointer, then entry 0 of that level-0 table is the kernel leaf (G set, U clear): PA 0x4123 as printed. **Q3.** The root, plus one level-0 table per 4 MiB region in use: code (0x0001_0000…) and heap (0x1000_0000…) and stack (just below 0x8000_0000) are in three different 4 MiB regions — 4 pages, 16 KiB.

**Unit 11.3, Q1.** 1,536 × 4 KiB = 6 MiB; 1,536 × 2 MiB = 3 GiB. **Q2.** At offset 0 every element maps to the same cache sets (4 KiB apart), so the measurement would show cache conflict misses rather than TLB effects — the first version of this program did exactly that.

**Unit 11.4, Q2.** One per page written: 16,384, since every page is shared until one side writes it.

**Unit 11.5, Q1.** 64 KiB / 4 KiB = 16 ways: 16 tag comparators and a 16-way multiplexer on the load path, more energy per access.

# Chapter 12 — DRAM, Memory Controllers, and RowHammer

*Goal: open the box labelled "main memory". DRAM is not a flat array with one latency: it is a hierarchy of channels, ranks, banks, rows, and columns, its latency depends on what the last access to the same bank was, it must be refreshed, and — as RowHammer showed — it can be corrupted by nothing more than reading it too often. This chapter builds a DRAM timing model and memory controller, measures memory-level parallelism on the host, shows how a scheduler that maximises throughput can starve a program, and simulates RowHammer and its mitigations.*

*Accompanies the lectures "Main Memory and DRAM Fundamentals", "Memory Controllers", "RowHammer", and "Memory Interference and QoS".*

**What you will be able to do after this chapter**

- Explain DRAM's organisation and the three cases of an access: row hit, row empty, and row conflict.
- Compute access latencies from timing parameters and refresh overhead from datasheet values.
- Explain open-page versus closed-page policy and FR-FCFS scheduling, and measure their effect.
- Explain inter-application interference in the memory controller and evaluate a fairness-aware scheduler.
- Explain RowHammer, why it is a reliability *and* security problem, and how mitigations work and fail.

## Topic 12A — How DRAM works

## Unit 12.1 — Cells, rows, banks, and the row buffer

**One-line summary:** A DRAM cell is one transistor and one capacitor; cells sit in a two-dimensional array of rows and columns, and reading any bit first *activates* its whole row — 8 KiB moved into the bank's row buffer of sense amplifiers — after which columns are read from the row buffer quickly, until the bank is *precharged* to open another row.

Accessing a column involves up to three commands, each with a minimum delay:

| Command | What it does | Timing parameter (DDR4-3200, typical) |
| --- | --- | --- |
| PRECHARGE | Close the open row, restoring it and preparing the bit lines | tRP ≈ 14 ns |
| ACTIVATE | Copy a row into the row buffer (sense amplifiers) | tRCD ≈ 14 ns until a column command |
| READ / WRITE | Transfer one burst (64 bytes from a 64-bit channel) from the row buffer | tCL ≈ 14 ns until data, then 4 bus cycles of data |

So an access is a **row hit** (row already open: tCL), a **row empty** (bank precharged: tRCD + tCL), or a **row conflict** (another row open: tRP + tRCD + tCL — three times a hit). These numbers have barely changed in twenty years: DRAM has become denser and its interface faster, but the latency of the cell array has stayed near 14 ns per step, because it is set by charging long, heavily loaded bit lines.

The organisation above the bank provides parallelism: a chip has 16 banks (DDR4) or 32 (DDR5) that can each have a different row open and work on different requests at once; several chips form a **rank** that works in lock-step to fill a 64-bit channel; a system has one or more **channels**, each with its own controller and bus. The memory controller maps physical addresses to channel, rank, bank, row, and column — and that mapping decides how much of the parallelism a program gets.

### Mastery check

1. **Calculate.** Using the table, what are the latencies (until the data starts) of a row hit, row empty, and row conflict?
2. **Explain.** Why does every activation have to rewrite the row back into the cells?

## Unit 12.2 — Row-buffer locality and bank-level parallelism

**One-line summary:** Streaming through a row gets mostly row hits and is limited only by the data bus; random accesses get row conflicts, and the only way to make them fast is to spread them over many banks so that their activations overlap; whether to leave a row open after an access (open-page) or close it at once (closed-page) depends on which of the two a workload has.

The model below simulates one DRAM channel with 8 banks and a memory controller, driven by "cores" that each keep a fixed number of requests in flight:

```python
# file: model/dram.py
"""A DRAM timing model: banks with row buffers, the three access cases (hit, empty, conflict),
and a memory controller with a pluggable scheduling policy, driven by closed-loop cores."""
import random

T_RCD, T_CL, T_RP, T_BURST = 14, 14, 14, 4        # nanoseconds, roughly DDR4-3200 22-22-22

class Bank:
    def __init__(self):
        self.open_row, self.free_at = None, 0

class Core:
    """Keeps `mlp` requests in flight; after each completes, computes for `think` ns, then issues another."""
    def __init__(self, cid, gen, mlp, think):
        self.cid, self.gen, self.mlp, self.think = cid, gen, mlp, think
        self.done = 0
        self.latency = 0

def simulate(cores, policy, banks=8, until=200_000, open_page=True, seed=1):
    rng = random.Random(seed)
    bank = [Bank() for _ in range(banks)]
    queue = []                                     # (arrival, core, bank, row, seq)
    seq = 0
    pending_issue = []                             # (time, core) when a core will issue
    for c in cores:
        for _ in range(c.mlp):
            pending_issue.append((0, c))
    bus_free = 0
    t = 0
    stats = {"hit": 0, "empty": 0, "conflict": 0}
    while t < until:
        for item in [p for p in pending_issue if p[0] <= t]:
            pending_issue.remove(item)
            b, r = item[1].gen(rng)
            queue.append((t, item[1], b, r, seq)); seq += 1
        ready = [q for q in queue if bank[q[2]].free_at <= t]
        if ready and bus_free <= t:
            q = policy(ready, bank, t)
            queue.remove(q)
            arr, core, b, r, _ = q
            bk = bank[b]
            if bk.open_row == r:
                lat, kind = T_CL, "hit"
            elif bk.open_row is None:
                lat, kind = T_RCD + T_CL, "empty"
            else:
                lat, kind = T_RP + T_RCD + T_CL, "conflict"
            stats[kind] += 1
            finish = t + lat + T_BURST
            row_time = lat - T_CL                          # precharge and activate, if needed
            bk.free_at = t + row_time + T_BURST            # next column command to this bank
            if open_page:
                bk.open_row = r                            # leave the row open for future hits
            else:
                bk.open_row = None                         # close it now: precharge after the access
                bk.free_at += T_CL + T_RP
            bus_free = t + T_BURST                         # one burst per slot on the shared bus
            core.done += 1
            core.latency += finish - arr
            pending_issue.append((finish + core.think, core))
        t += 1
    return stats

# --- scheduling policies: choose one of the requests whose bank is free ---
def fcfs(ready, bank, t):
    return min(ready, key=lambda q: q[4])                      # oldest first

def fr_fcfs(ready, bank, t):
    hits = [q for q in ready if bank[q[2]].open_row == q[3]]   # row hits first, then oldest
    return min(hits or ready, key=lambda q: q[4])
```

```python
# file: model/rowbuf.py
"""Row-buffer locality and bank-level parallelism: one core, different access patterns."""
from dram import *

def streaming():
    state = {"i": 0}
    def gen(rng):
        state["i"] += 1
        return 0, state["i"] // 128                 # 128 consecutive blocks per 8 KiB row, one bank
    return gen

def random_one_bank(rng):
    return 0, rng.randrange(65536)

def random_all_banks(rng):
    return rng.randrange(8), rng.randrange(65536)

print(f"{'pattern':34} {'policy':6} {'requests/us':>11} {'avg latency':>12}   hits / empty / conflicts")
for name, gen, mlp in (("streaming, one bank", streaming(), 4),
                       ("random rows, one bank", random_one_bank, 4),
                       ("random rows, 8 banks", random_all_banks, 4),
                       ("random rows, 8 banks, 1 in flight", random_all_banks, 1)):
    for open_page in (True, False):
        c = Core(0, gen, mlp, think=0)
        s = simulate([c], fr_fcfs, open_page=open_page, until=100_000)
        print(f"{name:34} {'open' if open_page else 'closed':6} {c.done / 100:11.1f} "
              f"{c.latency / c.done:9.1f} ns   {s['hit']} / {s['empty']} / {s['conflict']}")
```

```console
$ cd model && python3 rowbuf.py
pattern                            policy requests/us  avg latency   hits / empty / conflicts
streaming, one bank                open         211.9      18.9 ns   21027 / 1 / 165
streaming, one bank                closed        21.7     183.8 ns   0 / 2174 / 0
random rows, one bank              open          31.3     127.9 ns   0 / 1 / 3125
random rows, one bank              closed        21.7     183.8 ns   0 / 2174 / 0
random rows, 8 banks               open          78.7      50.9 ns   0 / 8 / 7857
random rows, 8 banks               closed        83.5      47.9 ns   0 / 8352 / 0
random rows, 8 banks, 1 in flight  open          21.8      45.9 ns   0 / 8 / 2169
random rows, 8 banks, 1 in flight  closed        29.7      33.6 ns   0 / 2974 / 0
```

- **Streaming** through one bank: almost all row hits, about 210 requests per microsecond — one 4 ns burst after another, limited by the bus. With closed-page policy the same stream pays tRCD + tCL + tRP for every block: ten times slower.
- **Random rows in one bank:** every access is a row conflict; nothing overlaps; about 31 per microsecond. Closed-page does not help, since every access must activate anyway, and adds the precharge time to each.
- **Random rows in 8 banks:** the activations of different banks overlap, and throughput rises 2.5×. Here closed-page wins slightly, because the precharge happens right after each access rather than when the next request arrives.
- **One request in flight:** no overlap is possible, and latency matters most; closed-page cuts the average latency from 46 to 34 ns because the next request finds its bank already precharged.

Real controllers use adaptive policies (close a row after a timeout, or predict per bank).

On real hardware, the only way to use bank-level parallelism from one core is to have several misses outstanding — memory-level parallelism (Chapter 10, Unit 10.7). The program below follows 1 to 16 independent random pointer chains through 1 GiB, with huge pages so that TLB misses do not interfere:

```c
// file: host/mlp.c
// Memory-level parallelism: follow K independent random pointer chains at once through 1 GiB.
// One chain waits for each DRAM access in turn; several chains keep several banks busy.
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <time.h>

#define SIZE (1UL << 30)
#define LINE 64
#define NLINES (SIZE / LINE)
#define STEPS 4000000

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

int main(void) {
    char *buf = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    madvise(buf, SIZE, MADV_HUGEPAGE);                         // keep TLB misses out of the picture
    size_t *perm = malloc(NLINES * sizeof *perm);
    for (size_t i = 0; i < NLINES; i++) perm[i] = i;
    srand(2);
    for (size_t i = NLINES - 1; i > 0; i--) {
        size_t j = ((size_t)rand() << 31 | rand()) % (i + 1);
        size_t t = perm[i]; perm[i] = perm[j]; perm[j] = t;
    }
    for (size_t i = 0; i < NLINES; i++)                        // one random cycle through all lines
        *(char **)(buf + perm[i] * LINE) = buf + perm[(i + 1) % NLINES] * LINE;

    for (int k = 1; k <= 16; k *= 2) {
        char *p[16];
        for (int c = 0; c < k; c++) p[c] = buf + perm[c * (NLINES / 16)] * LINE;   // far-apart starts
        double t = now();
        for (int s = 0; s < STEPS / k; s++)
            for (int c = 0; c < k; c++) p[c] = *(char **)p[c];
        t = now() - t;
        size_t sum = 0;
        for (int c = 0; c < k; c++) sum += (size_t)p[c] & 1;
        printf("%2d chains: %6.1f ns per load, %5.2f GB/s of 64-byte lines%s\n", k, t / STEPS * 1e9,
               STEPS * 64.0 / t / 1e9, sum == 99 ? "!" : "");
    }
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall mlp.c -o mlp && ./mlp
 1 chains:  226.7 ns per load,  0.28 GB/s of 64-byte lines
 2 chains:  114.1 ns per load,  0.56 GB/s of 64-byte lines
 4 chains:   61.7 ns per load,  1.04 GB/s of 64-byte lines
 8 chains:   30.4 ns per load,  2.10 GB/s of 64-byte lines
16 chains:   16.6 ns per load,  3.85 GB/s of 64-byte lines
```

One chain sees the full DRAM latency of about 200 ns per load. With 16 chains, each load still takes as long, but 16 are in flight at once, across banks and channels, and the time per load falls more than tenfold. This is why out-of-order execution, prefetching, and multithreading help so much with memory-bound code, and why a linked list is so much slower than an array that the hardware can fetch ahead.

### Mastery check

1. **Predict.** In `rowbuf.py`, what happens to "random rows, 8 banks" if the core keeps 16 requests in flight instead of 4? What limits it then?
2. **Explain.** Why does an address mapping that puts consecutive cache lines in *different* banks help random-access workloads but can hurt streaming ones?

## Unit 12.3 — Refresh

**One-line summary:** A DRAM capacitor leaks its charge in tens of milliseconds, so every row must be refreshed (read and rewritten) at least every 64 ms; the controller issues a REFRESH command every 7.8 µs (DDR4), and during the following tRFC — 350 ns for an 8 Gb chip, 550 ns for 16 Gb — the rank cannot be used, so refresh costs a growing share of time as chips grow.

**Worked example.** The fraction of time a DDR4 rank is busy refreshing is tRFC / tREFI:

| Chip density | tRFC | tREFI | Time refreshing |
| --- | --- | --- | --- |
| 8 Gb | 350 ns | 7.8 µs | 350 / 7800 = 4.5% |
| 16 Gb | 550 ns | 7.8 µs | 550 / 7800 = 7.1% |
| 16 Gb, above 85 °C | 550 ns | 3.9 µs | 550 / 3900 = 14.1% |

Refresh also consumes energy and delays the requests that arrive during it. Since most cells retain their charge far longer than 64 ms, proposals such as RAIDR (Liu et al., ISCA 2012) refresh most rows less often, binning them by measured retention time; DDR5 adds same-bank refresh so that the other banks stay available.

### Mastery check

1. **Calculate.** A system has 16 Gb chips and must refresh at 3.9 µs. If a rank is otherwise busy 50% of the time, how much of its idle time is consumed by refresh?

## Topic 12B — Sharing memory

## Unit 12.4 — Memory scheduling and interference

**One-line summary:** FR-FCFS (first-ready, first-come-first-served: row hits first, then oldest) maximises DRAM throughput and is used by almost every controller, but it is unfair: a program with high row-buffer locality is served again and again while a program with random accesses waits, so in a multicore system one program can slow another down by several times without slowing down itself.

```python
# file: model/fairness.py
"""Two programs share one memory: a streaming 'hog' with high row-buffer locality and a program
with random accesses. Slowdown = throughput alone / throughput when sharing."""
from dram import *

def make_stream():
    state = {"i": 0}
    def gen(rng):
        state["i"] += 1
        return state["i"] // 1024 % 8, state["i"] // 128          # long runs of hits, moving across banks
    return gen

def rand_gen(rng):
    return rng.randrange(8), rng.randrange(65536)

def cores():
    return [Core(0, make_stream(), mlp=8, think=0), Core(1, rand_gen, mlp=2, think=20)]

def compare(policies):
    alone = []
    for i in range(2):
        c = cores()[i]
        simulate([c], fr_fcfs, until=100_000)
        alone.append(c.done)
    print(f"{'scheduler':10} {'stream slowdown':>16} {'random slowdown':>16} {'unfairness':>11} {'total requests':>15}")
    for name, policy in policies:
        cs = cores()
        simulate(cs, policy, until=100_000)
        sd = [alone[i] / cs[i].done for i in range(2)]
        print(f"{name:10} {sd[0]:16.2f} {sd[1]:16.2f} {max(sd) / min(sd):11.2f} {cs[0].done + cs[1].done:15d}")

if __name__ == "__main__":
    compare([("FCFS", fcfs), ("FR-FCFS", fr_fcfs)])
```

```console
$ cd model && python3 fairness.py
scheduler   stream slowdown  random slowdown  unfairness  total requests
FCFS                   1.33             1.25        1.06           20212
FR-FCFS                1.03             7.94        7.72           23437
```

The streaming program alone is about as fast as when it shares with the random program under FR-FCFS (slowdown 1.03), while the random program becomes almost eight times slower: its requests are always older but never row hits, so they are passed over while the stream keeps its rows open. FCFS is fair but gives up 14% of the total throughput. Mutlu and Moscibroda called this a **memory performance hog** (USENIX Security 2007): a program can deny memory service to others simply by having good locality, and in the cloud, where tenants share machines, this is both a performance and a security issue.

### Deep & Tricky

**DT-34. Throughput and fairness are different goals: the scheduler that maximises total requests served (FR-FCFS) gives the worst slowdown to the program with the least row-buffer locality, so in a shared system "the memory is fast" can be true on average while one application runs eight times slower than alone. Evaluating a multicore memory system therefore needs multi-program metrics — weighted speedup (system throughput) and maximum slowdown (unfairness) — not average latency.** *(Mutlu & Moscibroda, "Stall-Time Fair Memory Access Scheduling", MICRO 2007; Eyerman & Eeckhout, "System-Level Performance Metrics for Multiprogram Workloads", IEEE Micro 2008)*

### Mastery check

1. **Explain.** Why does the random program in this model suffer far more under FR-FCFS than under FCFS, when FR-FCFS still serves it in age order whenever there is no row hit?

## Topic 12C — RowHammer

## Unit 12.5 — RowHammer: when reading memory corrupts it

**One-line summary:** Activating one DRAM row many times within a refresh interval disturbs the cells in physically adjacent rows — charge leaks faster through the coupling between word lines — and once a neighbour has seen a few thousand to a hundred thousand activations before its refresh, its bits flip; any program that can make many uncached accesses can corrupt memory it cannot write, which attackers turned into privilege escalation.

Kim et al. (ISCA 2014) showed bit flips in DRAM modules from all three major manufacturers, needing 139,000 activations in their worst case; repeated studies found the threshold falling as cells shrink — around 10,000 in recent DDR4 and below 5,000 in some LPDDR4 chips (Kim et al., ISCA 2020). Seaborn and Dullien (Google Project Zero, 2015) used flips in page-table entries to gain kernel privileges from an unprivileged process; later attacks worked from JavaScript, over the network (Throwhammer), and on phones (Drammer).

The model below counts, for one 64 ms window in one bank, how many rows receive enough neighbouring activations to flip — for ordinary random accesses, for single-sided hammering (one aggressor alternating with a far row, which forces the aggressor to be re-activated each time), for double-sided hammering (both neighbours of a victim), many-sided hammering, and a pattern with decoy rows; and with three mitigations:

```python
# file: model/rowhammer.py
"""RowHammer: activating a DRAM row many times disturbs its physical neighbours; if a neighbour
is not refreshed before it has seen HC_FIRST nearby activations, its cells flip. This model runs one
64 ms refresh window of activations on one bank and counts victim rows that flip, with and without
mitigations."""
import random

ROWS = 1024
HC_FIRST = 10_000          # activations of adjacent rows before the first flip (modern DDR4: ~10K)
ACTS = 1_360_000           # one 64 ms window at one activation every 47 ns
REFS = 8192                # regular refresh commands per window (one every 7.8 us)

def run(pattern, mitigation, seed=1):
    rng = random.Random(seed)
    disturb = [0] * ROWS
    flipped = set()
    tracker = []           # TRR's small table of [row, count]
    acts_per_ref = ACTS // REFS
    refresh = lambda r: disturb.__setitem__(r, 0) if 0 <= r < ROWS else None
    for i in range(ACTS):
        row = pattern(i, rng)
        for v in (row - 1, row + 1):                      # charge leaks from both neighbours
            if 0 <= v < ROWS:
                disturb[v] += 1
                if disturb[v] >= HC_FIRST:
                    flipped.add(v)
        if mitigation[0] == "para" and rng.random() < mitigation[1]:
            refresh(row + rng.choice((-1, 1)))           # PARA: probabilistic adjacent row activation
        if mitigation[0] == "trr":
            for e in tracker:
                if e[0] == row:
                    e[1] += 1
                    break
            else:
                if len(tracker) == mitigation[1]:
                    tracker.pop(0)                        # table full: forget the oldest entry
                tracker.append([row, 1])
        if i % acts_per_ref == acts_per_ref - 1:          # a regular refresh command
            r = (i // acts_per_ref) * ROWS // REFS        # refreshes rows in turn, each once per window
            refresh(r)
            if mitigation[0] == "trr" and tracker:        # TRR: also refresh the top aggressor's neighbours
                top = max(tracker, key=lambda e: e[1])
                refresh(top[0] - 1); refresh(top[0] + 1)
                tracker.remove(top)
    return len(flipped)

patterns = {
    "normal program (random rows)": lambda i, rng: rng.randrange(ROWS),
    "single-sided (row 100 + far row)": lambda i, rng: (100, 700)[i % 2],
    "double-sided (rows 100 and 102)": lambda i, rng: (100, 102)[i % 2],
    "many-sided, 10 aggressor pairs": lambda i, rng: 100 + 2 * (i % 20),
    "double-sided + 4 decoy rows": lambda i, rng: (100, 102, 500, 500, 600, 600, 700, 700, 800, 800)[i % 10],
}
mitigations = [("none", 0), ("para", 0.0005), ("para", 0.005), ("trr", 4)]
names = ["none", "PARA p=0.0005", "PARA p=0.005", "TRR, 4 entries"]
print(f"{'access pattern':34}" + "".join(f"{n:>16}" for n in names) + "   (victim rows that flip)")
for pname, pat in patterns.items():
    print(f"{pname:34}" + "".join(f"{run(pat, m):16d}" for m in mitigations))
```

```console
$ cd model && python3 rowhammer.py
access pattern                                none   PARA p=0.0005    PARA p=0.005  TRR, 4 entries   (victim rows that flip)
normal program (random rows)                     0               0               0               0
single-sided (row 100 + far row)                 4               4               0               0
double-sided (rows 100 and 102)                  3               3               0               0
many-sided, 10 aggressor pairs                  21              20               0               1
double-sided + 4 decoy rows                     11              11               0               5
```

Random accesses never come near the threshold: a row is activated on average 1,300 times per window, spread over all rows. All hammering patterns flip bits without a mitigation. **PARA** (probabilistic adjacent row activation, proposed with the original paper) refreshes a random neighbour of each activated row with a small probability p; with p = 0.0005 the expected gap between refreshes of a victim (about 4,000 of its neighbour's activations) is too close to the threshold, and victims still flip; with p = 0.005 every pattern is stopped, at a cost of 0.5% extra activations. **TRR** (target row refresh, the mitigation DDR4 vendors actually shipped) tracks a few frequently activated rows and refreshes their neighbours during regular refresh commands; it stops single- and double-sided hammering and all but one victim of the many-sided pattern, but a pattern with more rows than its table — here two aggressors hidden among decoys activated twice as often — steers it to refresh the decoys' neighbours while the real victims flip.

### Deep & Tricky

**DT-35. A mitigation that tracks a fixed number of aggressor rows can be defeated by an access pattern with more aggressors or decoys than its table: TRRespass (Frigo et al., IEEE S&P 2020) found that DDR4 modules advertised as RowHammer-free flipped bits with many-sided patterns, and Blacksmith (Jattke et al., IEEE S&P 2022) used non-uniform frequency patterns to flip bits on all 40 DDR4 modules tested. Deterministic, small-state defences need a proof that no pattern evades them; probabilistic ones (PARA) and counter-per-row designs with guaranteed bounds (Graphene, MICRO 2020) give one.** *(Frigo et al., "TRRespass: Exploiting the Many Sides of Target Row Refresh", S&P 2020; Kim et al., "Flipping Bits in Memory Without Accessing Them", ISCA 2014; Park et al., "Graphene", MICRO 2020)* ECC alone does not suffice either: ECCploit (S&P 2019) produced multi-bit flips that ECC miscorrects or misses.

### Pattern

**P-6. Treat reliability failures as security problems: any fault an attacker can induce on purpose — a bit flip from RowHammer, a timing fault from undervolting (Plundervolt), a speculative side effect — is an attack primitive, so mitigations must hold against the worst access pattern, not the typical one.** *(Mutlu & Kim, "RowHammer: A Retrospective", IEEE TCAD 2020)* *Trade-off:* worst-case guarantees cost area, energy, or performance (extra refreshes, counters) even for workloads that would never trigger the fault.

### Mastery check

1. **Explain.** Why must the single-sided pattern alternate with a far row rather than simply reading the aggressor row repeatedly? And why must a real attack use `clflush` or uncached accesses?
2. **Calculate.** With p = 0.005 and two aggressors around a victim, each activation has a 0.0025 chance of refreshing the victim. What is the probability that the victim survives 10,000 activations without being refreshed? (Use (1 − p)ⁿ ≈ e^(−pn).)

## Chapter 12 Project — A fairness-aware memory scheduler

### Requirements

Implement the BLISS scheduler (Subramanian et al., "The Blacklisting Memory Scheduler", ICCD 2014) for the model of Unit 12.2: it serves requests FR-FCFS, but counts how many requests in a row it has served from the same core; a core that reaches the threshold is blacklisted, and the requests of non-blacklisted cores are served first; the blacklist is cleared every 10,000 ns. Compare it with FCFS and FR-FCFS on the workload of Unit 12.4, and study the threshold.

### Reference solution

```python
# file: project/bliss.py
"""BLISS: the Blacklisting memory scheduler (Subramanian et al., ICCD 2014)."""
import sys
sys.path.insert(0, "../model")
from dram import fcfs, fr_fcfs
from fairness import compare

class Bliss:
    """Blacklisting (Subramanian et al., ICCD 2014): a core served `limit` times in a row is
    deprioritised for a while; otherwise FR-FCFS."""
    def __init__(self, limit=4, clear=10_000):
        self.limit, self.clear = limit, clear
        self.last, self.streak, self.black, self.cleared_at = None, 0, set(), 0
    def __call__(self, ready, bank, t):
        if t - self.cleared_at >= self.clear:
            self.black.clear(); self.cleared_at = t
        good = [q for q in ready if q[1].cid not in self.black] or ready
        q = fr_fcfs(good, bank, t)
        self.streak = self.streak + 1 if q[1].cid == self.last else 1
        self.last = q[1].cid
        if self.streak >= self.limit:
            self.black.add(q[1].cid)
        return q

compare([("FCFS", fcfs), ("FR-FCFS", fr_fcfs), ("BLISS", Bliss())])
for limit in (2, 8, 32):
    compare([(f"BLISS {limit}", Bliss(limit=limit))])
```

```console
$ cd project && python3 bliss.py | grep -v "^scheduler"
FCFS                   1.33             1.25        1.06           20212
FR-FCFS                1.03             7.94        7.72           23437
BLISS                  1.44             1.04        1.39           19444
BLISS 2                1.03             7.40        7.18           23445
BLISS 8                1.44             1.04        1.39           19413
BLISS 32               1.44             1.05        1.37           19408
```

With a threshold of 4, BLISS cuts the random program's slowdown from 7.9 to 1.04 and the unfairness from 7.7 to 1.4; the price is the stream's row hits, so total throughput falls back to about FCFS's level. A threshold of 2 is *worse* than no blacklist: the random program is also served twice in a row often enough to be blacklisted, and when both cores are on the list the scheduler falls back to plain FR-FCFS. Larger thresholds (8, 32) behave like 4, because the streaming core's runs of hits are far longer.

### Review checklist

- How does BLISS's state compare with that of schedulers that rank applications by measured slowdown (STFM, ATLAS, TCM)? Why does that matter in a controller that must decide every few nanoseconds?
- What would a priority-based scheme do with a latency-critical program that also has high row-buffer locality?
- Add a second channel and an address mapping. Does interference fall?

## Answers to the Chapter 12 prediction questions

**Unit 12.1, Q1.** 14 ns, 28 ns, 42 ns. **Q2.** Activation is destructive: sensing a cell shares its tiny charge with the bit line, so the sense amplifier must drive the value back into the cell.

**Unit 12.2, Q1.** It rises until the bus (one 4 ns burst at a time) or the banks' row-cycle time limits it: with 8 banks each busy about 30 ns per conflict, at most about 8 / 32 ns ≈ 250 per microsecond; the bus allows 250 per microsecond as well.

**Unit 12.3, Q1.** 14.1% of all time; the rank is idle 50%, so refresh takes about 28% of the idle time.

**Unit 12.4, Q1.** Under FR-FCFS, whenever the stream has a row hit waiting in a bank — almost always — that hit goes first, so the random program's request in the same bank waits for the whole run of hits; under FCFS it waits only for older requests.

**Unit 12.5, Q2.** e^(−0.0025 × 10,000) = e^(−25) ≈ 1.4 × 10⁻¹¹.

# Chapter 13 — I/O, Storage, and the Cost of Moving Data

*Goal: follow data out of the processor and memory to devices and back. The chapter explains how the processor talks to devices (memory-mapped registers, polling, interrupts, DMA), measures the layers of the storage stack on the host — page cache, direct device access, queue depth, and the price of durability — builds a flash translation layer to see why SSDs write more than they are asked to, and ends with the argument that drives much of current architecture research: moving data costs far more than computing on it.*

*Accompanies the lectures on I/O and storage, "Flash Memory and SSDs", and "Processing in Memory".*

**What you will be able to do after this chapter**

- Explain memory-mapped I/O, polling, interrupts, and DMA, and compute the processor time each costs.
- Measure and explain the gap between cached and device I/O, and how queue depth trades latency for throughput (Little's law).
- Explain flash memory's erase-before-write constraint, what an FTL does, and calculate write amplification.
- Explain why data movement dominates energy and what processing in or near memory tries to do about it.

## Topic 13A — Talking to devices

## Unit 13.1 — Memory-mapped I/O, polling, interrupts, and DMA

**One-line summary:** Devices expose control and status registers at physical addresses, so the processor drives them with ordinary loads and stores (memory-mapped I/O); it learns that a device is ready either by repeatedly reading a status register (polling) or by being interrupted, and for bulk data a DMA engine copies between the device and memory without the processor.

A UART like the one in Chapter 2 would appear to software as a few registers: a data register (a store sends a byte), a status register (a bit says "transmitter busy" or "byte received"), and a control register (enable interrupts). Those addresses must be **uncached** — the page table or a region register marks them so — because a cached status register would never change, and **ordered**, because a store that starts the device must not overtake the stores that set it up (RISC-V uses `fence` with the I/O bits; x86's uncached memory type is strongly ordered).

**Worked example: what does each method cost?** A processor at 2 GHz; a poll costs 400 cycles (an uncached device read); an interrupt costs 2,000 cycles to enter, handle, and return.

| Device | Events per second | Polling (must poll at event rate) | Interrupts |
| --- | --- | --- | --- |
| Keyboard | 30 (poll 100×/s to feel responsive) | 100 × 400 = 40,000 cycles/s: 0.002% | 30 × 2,000: 0.003% |
| Network, 1 Gb/s, 1,500-byte packets | 83,000 | 83,000 × 400 = 3.3 × 10⁷: 1.7% | 83,000 × 2,000 = 1.7 × 10⁸: 8.3% |
| NVMe SSD, 1 million I/Os per second | 10⁶ | 4 × 10⁸: 20% | 2 × 10⁹: 100% — one core does nothing else |

Interrupts win when events are rare and unpredictable; polling wins when they are frequent, which is why high-speed network and storage stacks (DPDK, SPDK, Linux's NAPI and io_uring polling modes) poll, and why devices **coalesce** interrupts (one interrupt per many completions). Data itself is never moved by these loads and stores: the driver gives the device's **DMA** engine a list of memory buffers, and the device reads or writes them directly, raising one interrupt (or completion-queue entry) when done.

### Deep & Tricky

**DT-36. DMA writes memory behind the processor's back, so on systems whose caches are not coherent with DMA, a driver must flush cache lines before the device reads a buffer and invalidate them before the processor reads data the device wrote — and must not let the buffer share a cache line with other data, or invalidating it destroys the neighbour's dirty bytes. Many embedded RISC-V and ARM systems are like this; x86 and server ARM make DMA cache-coherent in hardware.** *(Linux kernel documentation, "Dynamic DMA mapping Guide" (DMA-API-HOWTO), on streaming mappings and `ARCH_DMA_MINALIGN`; ARM Architecture Reference Manual on cache maintenance)*

### Mastery check

1. **Calculate.** With interrupt coalescing of one interrupt per 32 completions, what fraction of the core does the NVMe example spend on interrupts?
2. **Explain.** Why must a device's status register be mapped uncached? What would a cached poll loop do?

## Topic 13B — Storage

## Unit 13.2 — The storage stack, measured

**One-line summary:** A file read that hits the operating system's page cache costs about a microsecond (a system call and a 4 KiB copy); one that goes to the device costs tens of microseconds even on a fast SSD; and making a write *durable* with `fsync` costs a device round trip on every call — so durability, not bandwidth, usually limits databases and logs.

```c
// file: host/io.c
// What the storage stack costs: the page cache, direct I/O, sequential versus random, and fsync.
#define _GNU_SOURCE
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>

#define FILE_SIZE (256L << 20)
#define BLK 4096

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

static void reads(const char *what, int flags, int random, long count) {
    int fd = open("data.bin", O_RDONLY | flags);
    char *buf = aligned_alloc(BLK, BLK);                 // O_DIRECT needs aligned buffers
    srand(3);
    double t = now();
    for (long i = 0; i < count; i++) {
        long blk = random ? rand() % (FILE_SIZE / BLK) : i % (FILE_SIZE / BLK);
        if (pread(fd, buf, BLK, blk * BLK) != BLK) { perror("pread"); exit(1); }
    }
    t = now() - t;
    printf("%-34s %8.1f us per 4 KiB read, %7.0f MB/s\n", what, t / count * 1e6, count * (double)BLK / t / 1e6);
    free(buf);
    close(fd);
}

int main(void) {
    char *buf = aligned_alloc(BLK, BLK);
    memset(buf, 'x', BLK);
    int fd = open("data.bin", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    for (long i = 0; i < FILE_SIZE / BLK; i++) if (write(fd, buf, BLK) != BLK) return 1;
    fsync(fd);
    close(fd);

    reads("page cache, sequential", 0, 0, 65536);
    reads("page cache, random", 0, 1, 65536);
    reads("O_DIRECT (device), sequential", O_DIRECT, 0, 8192);
    reads("O_DIRECT (device), random", O_DIRECT, 1, 8192);

    fd = open("log.bin", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    for (int sync = 0; sync <= 1; sync++) {
        double t = now();
        for (int i = 0; i < 200; i++) {
            if (write(fd, buf, 512) != 512) return 1;     // append a 512-byte "log record"
            if (sync) fdatasync(fd);                      // ...and wait until it is on the device
        }
        t = now() - t;
        printf("append 512 bytes%-18s %8.1f us each\n", sync ? ", fdatasync each" : "", t / 200 * 1e6);
    }
    close(fd);
    unlink("data.bin"); unlink("log.bin");
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall io.c -o io && ./io
page cache, sequential                  0.9 us per 4 KiB read,    4322 MB/s
page cache, random                      1.8 us per 4 KiB read,    2259 MB/s
O_DIRECT (device), sequential          32.9 us per 4 KiB read,     125 MB/s
O_DIRECT (device), random              34.3 us per 4 KiB read,     119 MB/s
append 512 bytes                        0.6 us each
append 512 bytes, fdatasync each      153.4 us each
```

(The storage of this machine is a virtual disk; on a laptop NVMe SSD the direct reads take 10–80 µs and `fdatasync` from 20 µs to several milliseconds, depending on whether the drive has power-loss protection for its write cache.) Reads served from the page cache cost one or two microseconds, sequential or random. With `O_DIRECT` the page cache is bypassed and every read is a device request: 20 to 35 times slower. (Device timings on this shared virtual machine vary by tens of percent between runs.) An append that only reaches the page cache costs under a microsecond; waiting for the device each time costs over 100 µs.

### Deep & Tricky

**DT-37. `write` returning success means only that the data reached the operating system's page cache; it reaches the device later, and a crash or power failure before then loses it. Durability needs `fsync`/`fdatasync` (and, for a newly created file, an `fsync` of its directory), and the error from a failed write-back may be reported only by the next `fsync` — once: PostgreSQL retried a failed `fsync`, which then "succeeded", and silently lost data until 2018.** *(POSIX `fsync`; Rebello et al., "Can Applications Recover from fsync Failures?", USENIX ATC 2020; the PostgreSQL "fsyncgate" discussion, 2018)*

### Mastery check

1. **Explain.** A database commits 10,000 transactions per second on this machine, each needing one `fdatasync`. How? (Hint: group commit.)

## Unit 13.3 — Queue depth and Little's law

**One-line summary:** An SSD contains many flash chips that work in parallel, so it delivers its full throughput only with many requests outstanding; Little's law — requests in flight = throughput × latency — ties the three together, and pushing beyond the device's parallelism raises latency without raising throughput.

```c
// file: host/qd.c
// Random 4 KiB direct reads from 1, 4, 16 and 64 threads: an SSD (or a virtual disk backed by
// one) serves many requests in parallel, so throughput grows with the queue depth.
#define _GNU_SOURCE
#include <fcntl.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>

#define FILE_SIZE (256L << 20)
#define BLK 4096
#define PER_THREAD_MS 1500

static int fd;
static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }

static void *worker(void *arg) {
    unsigned seed = (unsigned)(size_t)arg;
    char *buf = aligned_alloc(BLK, BLK);
    long n = 0;
    double end = now() + PER_THREAD_MS / 1000.0;
    while (now() < end) {
        long blk = rand_r(&seed) % (FILE_SIZE / BLK);
        if (pread(fd, buf, BLK, blk * BLK) != BLK) { perror("pread"); exit(1); }
        n++;
    }
    free(buf);
    return (void *)n;
}

int main(void) {
    char *buf = aligned_alloc(BLK, BLK);
    memset(buf, 'y', BLK);
    int w = open("qd.bin", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    for (long i = 0; i < FILE_SIZE / BLK; i++) if (write(w, buf, BLK) != BLK) return 1;
    fsync(w); close(w);
    fd = open("qd.bin", O_RDONLY | O_DIRECT);
    for (int threads = 1; threads <= 64; threads *= 4) {
        pthread_t t[64];
        double start = now();
        for (int i = 0; i < threads; i++) pthread_create(&t[i], NULL, worker, (void *)(size_t)(i + 1));
        long total = 0;
        for (int i = 0; i < threads; i++) { void *r; pthread_join(t[i], &r); total += (long)r; }
        double secs = now() - start;
        printf("%2d requests in flight: %7.0f reads/s (%5.0f MB/s), %6.1f us average latency\n",
               threads, total / secs, total * (double)BLK / secs / 1e6, threads * secs / total * 1e6);
    }
    close(fd); unlink("qd.bin");
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall -pthread qd.c -o qd && ./qd
 1 requests in flight:   24809 reads/s (  102 MB/s),   40.3 us average latency
 4 requests in flight:   72645 reads/s (  298 MB/s),   55.1 us average latency
16 requests in flight:  148602 reads/s (  609 MB/s),  107.7 us average latency
64 requests in flight:  157310 reads/s (  644 MB/s),  406.8 us average latency
```

One request at a time gives 25,000–30,000 reads per second at 35–40 µs each. Sixteen in flight give six to seven times the throughput at two to three times the latency. At 64, throughput has saturated and Little's law forces the latency up: 64 divided by the saturated rate (about 160,000–220,000 per second, depending on the run) is 300–400 µs, as measured. This is the storage version of Chapter 12's memory-level parallelism, and why modern I/O interfaces (NVMe with up to 64K queues, io_uring) are built around deep queues.

### Mastery check

1. **Calculate.** A device serves 400,000 I/Os per second at 100 µs latency at saturation. How many requests must be in flight to reach that throughput?

## Unit 13.4 — Flash memory and the flash translation layer

**One-line summary:** NAND flash is read and programmed in pages (4–16 KiB) but erased only in blocks of hundreds of pages, and a page cannot be overwritten until its block is erased; an SSD therefore writes every update to a fresh page and keeps a logical-to-physical map — the flash translation layer — whose garbage collector must copy still-valid pages out of a block before erasing it, so the drive writes more than the host does (write amplification).

Flash cells also wear out: each block survives a limited number of program/erase cycles (a few thousand for TLC NAND), so the FTL must also spread erases evenly (wear levelling). The FTL below implements the core of it: out-of-place writes, a page map, and greedy garbage collection; drives reserve **over-provisioning** — physical capacity hidden from the host — to give the collector room.

```python
# file: ssd/ftl.py
"""A page-mapped flash translation layer (FTL): out-of-place writes, a logical-to-physical map,
garbage collection, and the resulting write amplification."""
import random

class FTL:
    def __init__(self, blocks=256, pages=64, op=0.125, victim="greedy", streams=1, hot=None, wear_limit=0):
        self.B, self.P = blocks, pages
        self.logical = int(blocks * pages * (1 - op))          # what the host is told it has
        self.l2p = {}                                          # logical page -> (block, page)
        self.owner = [[None] * pages for _ in range(blocks)]   # physical page -> logical page or None
        self.valid = [0] * blocks
        self.erases = [0] * blocks
        self.free = list(range(blocks))
        self.age = {}                                          # block -> when it was filled
        self.victim, self.hot, self.wear_limit = victim, hot, wear_limit
        self.open = [self.free.pop(0) for _ in range(streams)] # one open block per stream
        self.next = [0] * streams
        self.host_writes = self.flash_writes = 0
        self.clock = 0

    def _program(self, lpn, stream):
        b = self.open[stream]
        p = self.next[stream]
        self.owner[b][p] = lpn
        self.valid[b] += 1
        self.l2p[lpn] = (b, p)
        self.flash_writes += 1
        self.next[stream] += 1
        if self.next[stream] == self.P:                        # block full: open a fresh one
            self.age[b] = self.clock
            self.open[stream] = self.free.pop(0)
            self.next[stream] = 0

    def _invalidate(self, lpn):
        if lpn in self.l2p:
            b, p = self.l2p.pop(lpn)
            self.owner[b][p] = None
            self.valid[b] -= 1

    def _gc(self):
        while len(self.free) < 2:
            full = [b for b in self.age]                       # only full, closed blocks
            coldest = min(full, key=lambda b: self.erases[b])
            if self.wear_limit and max(self.erases) - self.erases[coldest] > self.wear_limit:
                v = coldest                                    # static wear levelling: move cold data
            elif self.victim == "greedy":
                v = min(full, key=lambda b: self.valid[b])     # fewest valid pages to copy
            else:
                v = min(full, key=lambda b: self.age[b])       # FIFO: oldest block
            del self.age[v]
            for p in range(self.P):                            # relocate the still-valid pages
                lpn = self.owner[v][p]
                if lpn is not None:
                    self._invalidate(lpn)
                    self._program(lpn, self._stream(lpn))      # keep hot with hot, cold with cold
            self.owner[v] = [None] * self.P
            self.erases[v] += 1
            self.free.append(v)

    def _stream(self, lpn):
        return 0 if self.hot is None or self.hot(lpn) else len(self.open) - 1

    def write(self, lpn):
        self.clock += 1
        self.host_writes += 1
        self._invalidate(lpn)
        self._program(lpn, self._stream(lpn))
        self._gc()

    def wa(self):
        return self.flash_writes / self.host_writes

def run(ftl, pattern, n_writes, seed=1):
    rng = random.Random(seed)
    for lpn in range(ftl.logical):                             # fill the drive once
        ftl.write(lpn)
    ftl.host_writes = ftl.flash_writes = 0                     # measure the steady state
    for i in range(n_writes):
        ftl.write(pattern(i, rng, ftl.logical))
    return ftl

sequential = lambda i, rng, n: i % n
uniform = lambda i, rng, n: rng.randrange(n)
def hot_cold(i, rng, n):                                       # 80% of writes go to 20% of the data
    return rng.randrange(n // 5) if rng.random() < 0.8 else n // 5 + rng.randrange(n - n // 5)

if __name__ == "__main__":
    print(f"{'workload':12} {'over-provisioning':>18} {'write amplification':>20}")
    for name, pat in (("sequential", sequential), ("uniform", uniform), ("hot/cold", hot_cold)):
        for op in (0.07, 0.125, 0.28):
            f = run(FTL(op=op), pat, 100_000)
            print(f"{name:12} {100 * op:17.0f}% {f.wa():20.2f}")
```

```console
$ cd ssd && python3 ftl.py
workload      over-provisioning  write amplification
sequential                   7%                 1.00
sequential                  12%                 1.00
sequential                  28%                 1.00
uniform                      7%                 7.23
uniform                     12%                 4.11
uniform                     28%                 1.95
hot/cold                     7%                 7.47
hot/cold                    12%                 4.35
hot/cold                    28%                 2.17
```

Sequential overwrites invalidate whole blocks in order, so the collector always finds an empty victim: write amplification 1. Uniform random writes leave every block partly valid; with 7% over-provisioning the drive writes 7 pages for each page the host writes. The simulation matches the classic approximation for greedy collection under uniform random writes, WA ≈ (1 + ρ) / (2ρ), where ρ is the spare capacity divided by the user capacity: for 7%, ρ = 0.07 / 0.93 = 0.075, and WA ≈ 1.075 / 0.15 ≈ 7.1. More over-provisioning buys lower write amplification — which is why enterprise drives hide 28% or more of their flash.

### Deep & Tricky

**DT-38. An SSD's speed and lifetime depend on its history, not just the current request: a fresh or freshly trimmed drive writes at full speed with write amplification near 1, but after the whole capacity has been written randomly, garbage collection runs alongside every write, and sustained random-write throughput can drop by a factor of several. Benchmarks must precondition the drive (fill it, then write randomly until steady state) — this simulation fills the drive once before measuring for the same reason — and file systems should issue TRIM so that the FTL knows which pages are free.** *(SNIA, "Solid State Storage Performance Test Specification", preconditioning and steady state; Hu et al., "Write Amplification Analysis in Flash-Based Solid State Drives", SYSTOR 2009)*

### Mastery check

1. **Calculate.** Using the approximation, what write amplification do you expect with 28% over-provisioning? Compare with the simulation, and explain the difference.
2. **Explain.** Why does the hot/cold workload give *higher* write amplification than uniform random writes with greedy collection?

## Topic 13C — Data movement

## Unit 13.5 — Moving data costs more than computing on it

**One-line summary:** At current technology, fetching 64 bits from DRAM costs about a thousand times the energy of a 32-bit floating-point addition, and data movement dominates both the time and the energy of data-intensive workloads; processing in memory (PIM) and near-data processing put computation where the data is, from simple bulk operations inside DRAM arrays to processors in the logic layer of stacked memory.

Horowitz's widely used estimates for a 45 nm process (ISSCC 2014):

| Operation | Energy |
| --- | --- |
| 32-bit integer add | 0.1 pJ |
| 32-bit floating-point add | 0.9 pJ |
| 32-bit floating-point multiply | 3.7 pJ |
| 64-bit read from an 8 KiB SRAM | 10 pJ |
| 64-bit read from a 1 MiB SRAM | 100 pJ |
| 64-bit read from DRAM | 1,300–2,600 pJ |

The measurements in this book agree from the time side: the SAXPY of Chapter 9 ran at 0.04 ns per element in the L1 cache and would run at about 1 ns per element from DRAM; the memory-bound pointer chase of Chapter 0 waited 100–200 ns for each load. Studies of consumer workloads (Boroumand et al., ASPLOS 2018) found that more than 60% of the system energy of tasks such as web browsing and video playback on a phone went to moving data between memory and the processor. Processing in memory attacks this directly: RowClone copies a DRAM row to another inside the chip, Ambit performs bulk bitwise AND/OR by activating three rows at once, and HBM-based designs (and commercial devices such as UPMEM's DRAM with embedded processors and Samsung's HBM-PIM) put simple cores or multiply-accumulate units next to the arrays. The systolic arrays of Chapter 9 are the same idea at another scale: reuse each operand many times once it is on chip.

### Mastery check

1. **Calculate.** A loop reads two 64-bit values from DRAM and does one floating-point multiply and add on them. Using the table, what fraction of its energy is computation?
2. **Explain.** Why does a dot product benefit far less from a faster processor core than a matrix multiply of the same data size?

## Chapter 13 Project — Taming write amplification

### Requirements

Using the FTL of Unit 13.4 and the hot/cold workload (80% of writes to 20% of the data), with 12.5% over-provisioning, compare: FIFO and greedy victim selection; separating hot and cold data into different open blocks (with the host telling the FTL which data is hot, as NVMe "streams" and zoned storage allow), with garbage-collected pages kept in their own temperature class; and adding static wear levelling, which takes the least-erased block as the victim whenever the gap between the most and least erased blocks exceeds a limit. Report write amplification and the spread of erase counts.

### Reference solution

The extensions are already part of `ftl.py`: the `streams` and `hot` parameters, `_stream`, and `wear_limit`. The experiment:

```python
# file: project/separate.py
"""Reducing write amplification: victim selection, and separating hot from cold data."""
import sys
sys.path.insert(0, "../ssd")
from ftl import FTL, run, hot_cold

N = 100_000
is_hot = lambda lpn, n=int(256 * 64 * (1 - 0.125)): lpn < n // 5     # the FTL is told which data is hot

configs = [
    ("FIFO victim, one stream", dict(victim="fifo")),
    ("greedy victim, one stream", dict(victim="greedy")),
    ("greedy, hot/cold streams", dict(victim="greedy", streams=2, hot=is_hot)),
    ("greedy, hot/cold streams, wear levelling", dict(victim="greedy", streams=2, hot=is_hot, wear_limit=8)),
]
print(f"{'hot/cold workload, 12.5% over-provisioning':44} {'WA':>5} {'erases: min':>12} {'max':>5}")
for name, kw in configs:
    f = run(FTL(op=0.125, **kw), hot_cold, N)
    print(f"{name:44} {f.wa():5.2f} {min(f.erases):12d} {max(f.erases):5d}")
```

```console
$ cd project && python3 separate.py
hot/cold workload, 12.5% over-provisioning      WA  erases: min   max
FIFO victim, one stream                       4.69           28    29
greedy victim, one stream                     4.35           20    37
greedy, hot/cold streams                      3.92            5    53
greedy, hot/cold streams, wear levelling      4.14           24    32
```

Greedy beats FIFO (4.35 against 4.69), but at the cost of uneven wear: blocks that happen to hold cold data are rarely chosen and rarely erased. Separating hot and cold data lowers write amplification further, to 3.92 — hot blocks invalidate almost completely before they are collected — but makes wear far more uneven, from 5 to 53 erases, because the cold blocks are left alone. Static wear levelling moves cold data onto worn blocks and brings the spread back to 24–32, giving back part of the gain. Every FTL balances these three: write amplification, wear, and the memory and time its decisions cost.

### Review checklist

- The map here is a Python dictionary; a real 1 TB drive with 4 KiB pages needs a 1 GB map. Where does it live, and what do DRAM-less drives do?
- What happens to the map on a sudden power loss, and how does the FTL rebuild it?
- How does TRIM change write amplification for a drive that is only half full of live data?

## Answers to the Chapter 13 prediction questions

**Unit 13.1, Q1.** 10⁶ / 32 × 2,000 = 6.25 × 10⁷ cycles per second, about 3% of a 2 GHz core. **Q2.** The first read would bring the register into the cache, and every later read would hit there and never see the device change it — the loop would spin forever.

**Unit 13.2, Q1.** Group commit: transactions that commit while one `fdatasync` is in progress wait for the next one, which makes all of them durable together; one device round trip covers many commits.

**Unit 13.3, Q1.** 400,000 × 100 × 10⁻⁶ = 40 requests in flight.

**Unit 13.4, Q1.** ρ = 0.28 / 0.72 = 0.39; WA ≈ 1.39 / 0.78 ≈ 1.8. The simulation gives about 1.95: with only 256 blocks the victims are less empty than the approximation (which assumes very many blocks) predicts. **Q2.** Greedy picks blocks with few valid pages; blocks that mix hot and cold pages keep their cold pages valid, so every collection copies cold data that will not change, again and again.

**Unit 13.5, Q1.** Computation 0.9 + 3.7 = 4.6 pJ; data 2 × 1,300 to 2 × 2,600 pJ — computation is 0.1–0.2% of the energy. **Q2.** A dot product does one multiply-add per element loaded and is limited by memory bandwidth; a matrix multiply does N multiply-adds per element and, with blocking, keeps the operands in caches, so the core's speed matters.

# Chapter 14 — Multiprocessors: Coherence, Consistency, and Synchronisation

*Goal: understand what changes when several processors share one memory. Caches give each core its own copy of data, so the hardware must keep the copies coherent; store buffers and out-of-order execution reorder memory operations, so the architecture must say which orders other cores can observe; and software builds locks out of atomic instructions whose cost is coherence traffic. The chapter measures a store-buffer reordering on the host, measures false sharing, simulates the MSI and MESI protocols, and compares spin locks by their bus traffic.*

*Accompanies the lectures "Multiprocessors", "Memory Ordering (Consistency)", and "Cache Coherence".*

**What you will be able to do after this chapter**

- Explain why multiprocessor speedup is limited (Amdahl's law with parallel overheads) and what thread-level parallelism costs.
- Explain sequential consistency and total store order, run a litmus test, and place fences correctly.
- Trace the MSI and MESI protocols and count their bus transactions.
- Recognise and fix false sharing.
- Explain how spin locks interact with coherence and choose a scalable one.

## Topic 14A — Parallel machines

## Unit 14.1 — Multiprocessors and their limits

**One-line summary:** A shared-memory multiprocessor runs several threads on several cores that all see one address space; its speedup is limited by the serial fraction of the program (Amdahl's law), by the overhead of communication and synchronisation, and by load imbalance — and growing the problem with the machine (Gustafson's law) is often how parallel speedups are actually obtained.

With a parallel fraction p on N cores, Amdahl's speedup is 1 / ((1 − p) + p / N): with p = 0.95, 8 cores give 5.9×, 64 give 15.4×, and no number of cores gives more than 20×. Communication makes things worse — every time a core uses data another core has written, it pays a cache-to-cache transfer of tens of nanoseconds (the next units measure it) — and synchronisation serialises the cores that wait. Gustafson's observation is that users of bigger machines usually run bigger problems, whose serial part grows more slowly: the scaled speedup with serial fraction s of the *parallel* run is N − s(N − 1).

The rest of this chapter is about the hardware mechanisms underneath shared memory. This machine has only two cores, which is enough to see each effect.

### Mastery check

1. **Calculate.** A program spends 10% of its time in serial code. What is its speedup on 16 cores? What serial fraction would you need to reach 12×?

## Topic 14B — Memory consistency

## Unit 14.2 — Sequential consistency and total store order

**One-line summary:** Sequential consistency (SC) — the result is as if all cores' operations were interleaved in one order that respects each core's program order — is what programmers intuitively expect, but no mainstream processor provides it by default: x86's total store order (TSO) lets a load overtake an earlier store to a different address (the store waits in a store buffer), and ARM and RISC-V (RVWMO) allow almost any reordering of independent accesses; fences restore order where it is needed.

The **store-buffering** litmus test makes the difference visible. Thread 0 runs `x = 1; r0 = y;`, thread 1 runs `y = 1; r1 = x;`, with x and y initially 0. Under SC, whichever store comes first in the interleaving is seen by the other thread's load, so r0 = r1 = 0 is impossible. Under TSO each store can still be in its core's store buffer when the other core's load reads memory:

```c
// file: host/sb.c
// The "store buffering" litmus test. Thread 0: x = 1; r0 = y.  Thread 1: y = 1; r1 = x.
// Under sequential consistency at least one thread must see the other's store, so r0 = r1 = 0
// is impossible. On x86 (TSO) each store can wait in a store buffer while the later load runs.
// Each round uses fresh variables; the two threads meet at a spin barrier before each round.
#include <pthread.h>
#include <stdatomic.h>
#include <stdio.h>
#include <stdlib.h>

#define ROUNDS 200000
typedef struct { _Alignas(64) volatile int v; } padded;   // one variable per cache line
static padded *x, *y;
static int r0[ROUNDS], r1[ROUNDS];
static atomic_int arrived;
static int fence;

static void barrier(int round) {                           // both threads wait for each other
    atomic_fetch_add(&arrived, 1);
    while (atomic_load(&arrived) < 2 * (round + 1)) ;
}
static void *t0(void *a) {
    for (int i = 0; i < ROUNDS; i++) {
        barrier(i);
        x[i].v = 1;
        if (fence) atomic_thread_fence(memory_order_seq_cst);   // mfence on x86
        r0[i] = y[i].v;
    }
    return a;
}
static void *t1(void *a) {
    for (int i = 0; i < ROUNDS; i++) {
        barrier(i);
        y[i].v = 1;
        if (fence) atomic_thread_fence(memory_order_seq_cst);
        r1[i] = x[i].v;
    }
    return a;
}

int main(void) {
    x = aligned_alloc(64, ROUNDS * sizeof *x);
    y = aligned_alloc(64, ROUNDS * sizeof *y);
    for (fence = 0; fence <= 1; fence++) {
        for (int i = 0; i < ROUNDS; i++) x[i].v = y[i].v = 0;
        atomic_store(&arrived, 0);
        pthread_t a, b;
        pthread_create(&a, NULL, t0, NULL);
        pthread_create(&b, NULL, t1, NULL);
        pthread_join(a, NULL); pthread_join(b, NULL);
        long both_zero = 0;
        for (int i = 0; i < ROUNDS; i++) both_zero += (r0[i] == 0 && r1[i] == 0);
        printf("%s: r0 = r1 = 0 in %ld of %d rounds\n", fence ? "with a fence " : "without fence", both_zero, ROUNDS);
    }
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall -pthread sb.c -o sb && ./sb
without fence: r0 = r1 = 0 in 6194 of 200000 rounds
with a fence : r0 = r1 = 0 in 0 of 200000 rounds
```

The "impossible" outcome occurs in a few rounds in a thousand up to a few in a hundred — the rate depends on how closely the two threads leave the barrier together, and varies from run to run; a full fence (`mfence`, from `atomic_thread_fence(memory_order_seq_cst)`) between the store and the load makes the core drain its store buffer first, and it never occurs. TSO forbids the other reorderings — on x86, two stores are seen in order and two loads are performed in order — so the classic "message passing" idiom (write data, then write a flag; read the flag, then read data) works on x86 without fences, but not on ARM or RISC-V, where it needs a release store and an acquire load.

### Deep & Tricky

**DT-39. `volatile` in C stops the *compiler* from removing or reordering accesses to that variable, but emits no fence, so it says nothing about the order in which other cores see them; the code above with `volatile` and no fence produces the non-SC outcome on x86, and on ARM or RISC-V even the message-passing idiom breaks. Portable code uses C11/C++11 atomics with the right memory order, which compile to the fences each architecture needs — `memory_order_seq_cst` stores become `xchg` on x86, and release/acquire become `fence rw,w` / `fence r,rw` (or `.aqrl` bits) on RISC-V.** *(ISO C17 §5.1.2.4 and §7.17; "Mappings from C/C++ primitives to RISC-V primitives", RISC-V Unprivileged Specification, Appendix A; Sewell et al., "x86-TSO", CACM 2010)*

### Mastery check

1. **Predict.** Which of these outcomes can x86 produce in the message-passing test (thread 0: `data = 1; flag = 1;` thread 1: `r0 = flag; r1 = data;`): r0 = 1, r1 = 0? And RISC-V without fences?
2. **Explain.** Why do processors relax consistency at all? What would a strictly SC out-of-order core have to do with every load?

## Topic 14C — Cache coherence

## Unit 14.3 — The coherence problem and the MSI/MESI protocols

**One-line summary:** When several caches hold copies of the same line, a write by one core must make the others' copies invalid (or update them) so that every core eventually sees the latest value; snooping protocols do this on a shared bus with per-line states — Modified, Shared, Invalid, plus Exclusive in MESI, which lets a core that has the only copy write it without telling anyone.

The single-writer, multiple-reader invariant: at any time a line is either writable by one cache (M) or readable by any number (S). A read miss broadcasts **BusRd**; a cache holding the line in M supplies it (a cache-to-cache transfer) and drops to S. A write to a line not held broadcasts **BusRdX** (read for ownership); a write to a line in S broadcasts **BusUpgr**; both invalidate every other copy. MESI's E state is granted when a read miss finds no other copy: a later write turns E into M silently.

```python
# file: coh/mesi.py
"""A snooping cache-coherence simulator: private caches on a shared bus, with the MSI or MESI
protocol. Counts bus transactions — the cost of coherence."""
from collections import Counter

class System:
    def __init__(self, cores, protocol="MESI"):
        self.n, self.protocol = cores, protocol
        self.state = [dict() for _ in range(cores)]        # line -> 'M', 'E', 'S' (absent = I)
        self.bus = Counter()

    def _others(self, core, line):
        return [c for c in range(self.n) if c != core and line in self.state[c]]

    def read(self, core, line):
        st = self.state[core]
        if line in st:
            return                                         # hit in M, E or S: no bus traffic
        others = self._others(core, line)
        self.bus["BusRd"] += 1
        for c in others:
            if self.state[c][line] == "M":
                self.bus["flush"] += 1                     # owner supplies the dirty line
            self.state[c][line] = "S"
        st[line] = "S" if others or self.protocol == "MSI" else "E"

    def write(self, core, line):
        st = self.state[core]
        s = st.get(line)
        if s == "M":
            return
        if s == "E":
            st[line] = "M"                                 # silent upgrade: nobody else has it
            return
        others = self._others(core, line)
        if s == "S":
            self.bus["BusUpgr"] += 1                       # invalidate the other copies
        else:
            self.bus["BusRdX"] += 1                        # read for ownership
            if any(self.state[c][line] == "M" for c in others):
                self.bus["flush"] += 1
        self.bus["invalidations"] += len(others)
        for c in others:
            del self.state[c][line]
        st[line] = "M"

    def total(self):
        return self.bus["BusRd"] + self.bus["BusRdX"] + self.bus["BusUpgr"]
```

```python
# file: coh/patterns.py
from mesi import System

def private(s):                     # each core reads then writes its own 100 lines, 10 times
    for _ in range(10):
        for c in range(s.n):
            for l in range(100):
                s.read(c, (c, l)); s.write(c, (c, l))

def producer_consumer(s):           # core 0 writes 100 lines, then cores 1..3 read them; 10 rounds
    for _ in range(10):
        for l in range(100):
            s.write(0, l)
        for c in range(1, s.n):
            for l in range(100):
                s.read(c, l)

def false_sharing(s):               # four cores each update "their own" counter — in one line
    for _ in range(1000):
        for c in range(s.n):
            s.read(c, "line"); s.write(c, "line")

def read_shared(s):                 # everybody reads a shared table of 100 lines, 10 times
    for _ in range(10):
        for c in range(s.n):
            for l in range(100):
                s.read(c, ("table", l))

print(f"{'pattern (4 cores)':20} {'protocol':8} {'bus transactions':>17} {'invalidations':>14} {'cache-to-cache':>15}")
for name, f in (("private data", private), ("producer-consumer", producer_consumer),
                ("false sharing", false_sharing), ("read-only sharing", read_shared)):
    for proto in ("MSI", "MESI"):
        s = System(4, proto)
        f(s)
        print(f"{name:20} {proto:8} {s.total():17d} {s.bus['invalidations']:14d} {s.bus['flush']:15d}")
```

```console
$ cd coh && python3 patterns.py
pattern (4 cores)    protocol  bus transactions  invalidations  cache-to-cache
private data         MSI                    800              0               0
private data         MESI                   400              0               0
producer-consumer    MSI                   4000           2700            1000
producer-consumer    MESI                  4000           2700            1000
false sharing        MSI                   8000           3999            3999
false sharing        MESI                  7999           3999            3999
read-only sharing    MSI                    400              0               0
read-only sharing    MESI                   400              0               0
```

- **Private data:** MESI halves the traffic — every line is read in E and upgraded silently; MSI needs a BusUpgr for each first write. Most data in real programs is private, which is why every modern protocol has an E state (or equivalent).
- **Producer-consumer:** every round, the producer invalidates three copies of each line and each consumer then misses and gets the data cache-to-cache: communication is unavoidable and costs a miss per line per consumer.
- **False sharing:** four cores update *different* counters in one line; every update needs the line exclusively, so 4,000 updates cause about 4,000 invalidations and 4,000 transfers — as much traffic as if they shared one variable.
- **Read-only sharing:** only compulsory misses; S copies coexist forever.

Large machines replace the broadcast bus with a **directory** that records, per line, which caches hold it, so that invalidations go only to them; the states and the invariant are the same.

### Mastery check

1. **Trace.** Starting with all caches empty, give the states of the line in cores 0 and 1 and the bus transactions for: core 0 reads, core 0 writes, core 1 reads, core 1 writes, core 0 reads — under MESI.
2. **Explain.** Why is a directory needed beyond a few dozen cores?

## Unit 14.4 — False sharing, measured

**One-line summary:** Coherence works on whole cache lines, so two threads writing different variables that share a 64-byte line force the line to bounce between their caches; on the host, two threads incrementing adjacent atomic counters run two to five times slower than with the counters on separate lines.

```c
// file: host/falseshare.c
// Two threads each increment their own counter 100 million times. The counters are either
// adjacent (same 64-byte cache line) or 64 bytes apart. No data is shared — only a line.
// Plain increments (load, add, store) and atomic ones (lock add) are both measured.
#include <pthread.h>
#include <stdatomic.h>
#include <stdio.h>
#include <time.h>

#define N 100000000
static struct { volatile long a; volatile long b; } near_;                         // same line
static struct { volatile long a; char pad[56]; volatile long b; } far_ __attribute__((aligned(64)));
static volatile long *ca, *cb;
static int atomic_ops;

static double now(void) { struct timespec t; clock_gettime(CLOCK_MONOTONIC, &t); return t.tv_sec + t.tv_nsec * 1e-9; }
static void inc(volatile long *c) {
    if (atomic_ops)
        for (long i = 0; i < N; i++) atomic_fetch_add_explicit((_Atomic long *)c, 1, memory_order_relaxed);
    else
        for (long i = 0; i < N; i++) (*c)++;
}
static void *inc_a(void *p) { inc(ca); return p; }
static void *inc_b(void *p) { inc(cb); return p; }

static void run(const char *what, volatile long *a, volatile long *b, int threads) {
    ca = a; cb = b;
    pthread_t t1, t2;
    double t = now();
    pthread_create(&t1, NULL, inc_a, NULL);
    if (threads == 2) pthread_create(&t2, NULL, inc_b, NULL);
    pthread_join(t1, NULL);
    if (threads == 2) pthread_join(t2, NULL);
    printf("%-8s %-36s %5.2f s\n", atomic_ops ? "atomic" : "plain", what, now() - t);
}

int main(void) {
    for (atomic_ops = 0; atomic_ops <= 1; atomic_ops++) {
        run("one thread, one counter", &far_.a, &far_.b, 1);
        run("two threads, counters 64 bytes apart", &far_.a, &far_.b, 2);
        run("two threads, counters in one line", &near_.a, &near_.b, 2);
    }
    return 0;
}
```

```console
$ cd host && gcc -O2 -Wall -pthread falseshare.c -o fs && ./fs
plain    one thread, one counter               0.03 s
plain    two threads, counters 64 bytes apart  0.04 s
plain    two threads, counters in one line     0.04 s
atomic   one thread, one counter               0.56 s
atomic   two threads, counters 64 bytes apart  0.71 s
atomic   two threads, counters in one line     1.93 s
```

Plain increments are cheap (the core forwards each store to the next load from its store buffer), and with the counters on separate lines two threads take no longer than one. In the same line, the stores must still win the line in M before they leave the store buffer, but the buffer hides much of that: across runs on this machine the plain case ranged from no visible effect to three times slower. Atomic increments (`lock add`) cannot hide it — each must own the line and drain the store buffer before it completes — so they are slow even alone, and false sharing makes them another two to five times slower (the factor varies between runs): the line moves between the cores on almost every operation. The fix is layout: pad or align per-thread data to 64 bytes (C11 `_Alignas(64)`, C++17 `std::hardware_destructive_interference_size`), or accumulate privately and combine at the end.

### Mastery check

1. **Explain.** Why do the two separate counters, with two threads, take the same time as one thread with one counter, rather than half as long?

## Topic 14D — Synchronisation

## Unit 14.5 — Atomic operations and locks

**One-line summary:** Locks are built from atomic read-modify-write instructions (swap, compare-and-swap, fetch-and-add; on RISC-V, `amoswap` or load-reserved/store-conditional), each of which must own the line; a naive test-and-set spin lock floods the interconnect because every waiting core keeps stealing the line, while test-and-test-and-set and ticket locks spin on a shared read-only copy and generate traffic only when the lock is released.

RISC-V gives two styles. **AMOs** perform the operation at the memory system (`amoswap.w.aq t0, t1, (a0)`: swap and acquire). **LR/SC** pairs implement anything: `lr.w` loads and reserves the line; `sc.w` stores only if no other core wrote the line in between, and reports failure otherwise, so a compare-and-swap is a short loop. The `.aq` and `.rl` bits add the acquire/release ordering a lock needs (Unit 14.2): nothing in the critical section may move above the acquire or below the release.

The project measures what the different spin loops cost in coherence traffic.

### Deep & Tricky

**DT-40. A lock-free algorithm built on compare-and-swap can be fooled by the ABA problem: a thread reads A, another thread changes the location to B and back to A, and the first thread's CAS succeeds although the structure changed underneath — for example, a node was popped from a lock-free stack, freed, and reallocated at the same address. LR/SC does not have this flaw, because any intervening write breaks the reservation; with CAS, the fixes are version counters in a double-width CAS, hazard pointers, or epoch-based reclamation.** *(Herlihy & Shavit, "The Art of Multiprocessor Programming", §10.6; Michael, "Hazard Pointers", IEEE TPDS 2004; RISC-V Unprivileged Specification, "A" extension, LR/SC)*

### Mastery check

1. **Write.** Write a compare-and-swap loop that atomically increments a counter using `lr.w`/`sc.w` in RISC-V assembly.

## Chapter 14 Project — Spin locks and coherence traffic

### Requirements

On the MESI simulator, model 2 to 16 cores that each acquire a lock 20 times, hold it for 20 steps, and release it, with every core performing one memory operation per step. Compare: a test-and-set (TAS) lock that spins with an atomic swap; a test-and-test-and-set (TTAS) lock that spins reading the lock and attempts the swap only when it looks free; and a ticket lock (fetch-and-increment a ticket, spin reading `now_serving`). Report bus transactions per acquisition.

### Reference solution

```python
# file: project/locks.py
"""Spin locks on the MESI simulator: every core wants the lock 20 times. Each simulation step,
every core performs one memory operation; we count the bus traffic per acquisition."""
import sys
sys.path.insert(0, "../coh")
from mesi import System

HOLD = 20                           # steps the critical section takes
ROUNDS = 20

def simulate(kind, cores):
    s = System(cores)
    mem = {"lock": 0, "next": 0, "serving": 0}
    state = [{"phase": "try", "left": ROUNDS, "hold": 0, "ticket": None} for _ in range(cores)]
    acquisitions = steps = 0
    while any(st["left"] for st in state):
        steps += 1
        for c, st in enumerate(state):
            if not st["left"]:
                continue
            if st["phase"] == "hold":                        # in the critical section
                st["hold"] -= 1
                if st["hold"] == 0:                           # release
                    if kind == "ticket":
                        s.write(c, "serving"); mem["serving"] += 1
                    else:
                        s.write(c, "lock"); mem["lock"] = 0
                    st["left"] -= 1
                    st["phase"] = "try"
                continue
            if kind == "TAS":                                 # test-and-set: an atomic swap every time
                s.write(c, "lock")
                got = mem["lock"] == 0
                mem["lock"] = 1
            elif kind == "TTAS":                              # test-and-test-and-set: spin on a read
                s.read(c, "lock")
                got = False
                if mem["lock"] == 0:
                    s.write(c, "lock")                        # looks free: now try the swap
                    got = mem["lock"] == 0
                    mem["lock"] = 1
            else:                                             # ticket lock: take a number, wait for it
                if st["ticket"] is None:
                    s.write(c, "next")                        # atomic fetch-and-increment
                    st["ticket"] = mem["next"]; mem["next"] += 1
                s.read(c, "serving")
                got = mem["serving"] == st["ticket"]
                if got:
                    st["ticket"] = None
            if got:
                st["phase"], st["hold"] = "hold", HOLD
                acquisitions += 1
    return s.total() / acquisitions, steps

print(f"{'lock':6} " + "".join(f"{f'{n} cores':>16}" for n in (2, 4, 8, 16)) + "   (bus transactions per acquisition)")
for kind in ("TAS", "TTAS", "ticket"):
    print(f"{kind:6} " + "".join(f"{simulate(kind, n)[0]:16.1f}" for n in (2, 4, 8, 16)))
```

```console
$ cd project && python3 locks.py
lock            2 cores         4 cores         8 cores        16 cores   (bus transactions per acquisition)
TAS                 3.0            60.2           138.4           294.4
TTAS                3.9             5.9             9.8            17.6
ticket              3.0             5.0             8.9            16.7
```

With two cores the TAS lock looks harmless: the single waiter keeps the line in M and its repeated swaps hit in its own cache, silently. With four or more waiters, every swap steals the line from the previous spinner, and the traffic per acquisition grows to almost 300 transactions at 16 cores — while the holder, who needs the line to release, competes with them. TTAS and the ticket lock spin on S copies and cost traffic only at each release, when every waiter's copy is invalidated and re-read: still O(N) per handoff, about 17 at 16 cores. The ticket lock is also fair (first come, first served), which TAS and TTAS are not. Queue locks (MCS, CLH) bring it to O(1) by giving each waiter its own line to spin on, and Linux's `qspinlock` is one.

### Review checklist

- Add exponential backoff to TAS (wait 1, 2, 4… steps after a failed swap). How does its traffic compare with TTAS?
- Implement an MCS lock in the simulator: each waiter spins on a flag in its own node. Count the traffic.
- What happens to the ticket lock if a waiting thread is descheduled by the OS?

## Answers to the Chapter 14 prediction questions

**Unit 14.1, Q1.** 1 / (0.1 + 0.9 / 16) = 1 / 0.15625 = 6.4×. For 12× on 16 cores: 1 / (s + (1 − s) / 16) = 12 → s + (1 − s) / 16 = 0.0833 → 15s = 0.333 → s ≈ 2.2%.

**Unit 14.2, Q1.** x86 (TSO) keeps both stores in order and both loads in order: r0 = 1, r1 = 0 is impossible. RISC-V (RVWMO) without fences may reorder either pair, so it is allowed. **Q2.** Performance: an SC core would have to complete stores before later loads (stalling on the store buffer) or speculate and check every load against other cores' invalidations, squashing on conflict — which high-end cores do (and use for TSO's load-load ordering).

**Unit 14.3, Q1.** Core 0 read: BusRd, 0 = E. Core 0 write: silent, 0 = M. Core 1 read: BusRd, 0 flushes, both S. Core 1 write: BusUpgr, 0 = I, 1 = M. Core 0 read: BusRd, 1 flushes, both S. **Q2.** Broadcasting every miss to every cache uses bus bandwidth and snoop bandwidth that grow with the number of cores; a directory sends messages only to the caches that hold the line.

**Unit 14.4, Q1.** Each thread does the same 10⁸ increments; with no interference they run in parallel on two cores, so the elapsed time is that of one thread.

**Unit 14.5, Q1.**

```asm
increment:            # a0 = address of the counter
1:  lr.w    t0, (a0)
    addi    t0, t0, 1
    sc.w    t1, t0, (a0)   # t1 = 0 on success
    bnez    t1, 1b
    ret
```

# Chapter 15 — Capstone: A Complete Processor with Caches

*Goal: put the book together. The pipelined RV32I processor of Chapters 6 and 8 — forwarding, hazard detection, a branch target buffer and 2-bit predictors — gets instruction and data caches in front of a slow memory, runs a C program compiled with GCC that stresses the memory system, and is simulated at full speed with Verilator. Every number it produces is then predicted independently, by the instruction-set simulator of Chapter 4 driving the cache simulator of Chapter 10, and the two agree — to the miss.*

*Accompanies the course's closing lectures, and serves as the final project.*

**What you will be able to do after this chapter**

- Integrate caches into a pipeline with a global stall ("freeze") and reason about its correctness.
- Use a fast functional simulator and a trace-driven model as golden references for an RTL design, and explain every remaining difference.
- Use Verilator for fast, cycle-accurate simulation.
- Read a performance result as a chain of causes: instructions, CPI from hazards, misses, and memory latency.

## Topic 15A — The system

## Unit 15.1 — The workload

**One-line summary:** A capstone needs a program whose behaviour you can predict: four small kernels over 16 KiB data structures — a row-order sum, a column-order sum, a matrix transpose, and a linked-list walk in random order — each exercising a different kind of locality, compiled for RV32I and checked on QEMU and on the instruction-set simulator before any hardware runs it.

```c
// file: sw/memtest.c
// Memory-system workloads for the capstone CPU (RV32I only: no multiply or divide).
#include "rt.h"

#define NOINLINE __attribute__((noipa))
#define R 64                                  // 64 x 64 ints = 16 KiB per matrix
static int A[R][R], B[R][R];
static unsigned seed = 1;
static unsigned rnd(void) { seed ^= seed << 13; seed ^= seed >> 17; seed ^= seed << 5; return seed; }

static NOINLINE int sum_rows(void) {          // walks A in memory order
    int s = 0;
    for (int i = 0; i < R; i++) for (int j = 0; j < R; j++) s += A[i][j];
    return s;
}
static NOINLINE int sum_cols(void) {          // the same sum, 256 bytes between accesses
    int s = 0;
    for (int j = 0; j < R; j++) for (int i = 0; i < R; i++) s += A[i][j];
    return s;
}
static NOINLINE void transpose(void) {        // reads rows of A, writes columns of B
    for (int i = 0; i < R; i++) for (int j = 0; j < R; j++) B[j][i] = A[i][j];
}

#define NODES 2048                            // 2048 nodes of 8 bytes = 16 KiB
struct node { struct node *next; int val; };
static struct node pool[NODES];
static int order[NODES];
static NOINLINE int chase(void) {             // follows the list in a random order
    int s = 0;
    for (struct node *p = &pool[order[0]]; p; p = p->next) s += p->val;
    return s;
}

static void say(const char *what, int v) { puts_(what); putint(v); puts_("\n"); }

int main(void) {
    for (int i = 0; i < R; i++) for (int j = 0; j < R; j++) A[i][j] = (int)(rnd() & 255);
    for (int i = 0; i < NODES; i++) { order[i] = i; pool[i].val = i & 7; }
    for (int i = NODES - 1; i > 0; i--) {     // shuffle with rejection sampling instead of %
        unsigned j;
        do j = rnd() & (NODES - 1); while (j > (unsigned)i);
        int t = order[i]; order[i] = order[j]; order[j] = t;
    }
    for (int i = 0; i < NODES - 1; i++) pool[order[i]].next = &pool[order[i + 1]];
    pool[order[NODES - 1]].next = 0;

    say("rows ", sum_rows());
    say("cols ", sum_cols());
    transpose();
    say("trans ", B[5][7] - A[7][5]);
    say("chase ", chase());
    return 0;
}
```

```console
$ cd sw && riscv64-linux-gnu-gcc -march=rv32i -mabi=ilp32 -O2 -nostdlib -ffreestanding -fno-builtin -static -T ../../ch05/sw/link.ld -Wl,--build-id=none -Wl,--no-warn-rwx-segments -I../../ch04/rvsim ../../ch04/rvsim/rt.c memtest.c -o memtest.elf && riscv64-linux-gnu-objcopy -O binary memtest.elf m.bin && python3 ../../ch05/sw/mkhex.py m.bin memtest.hex && qemu-riscv32 ./memtest.elf
memtest.hex: 297 words
rows 524737
cols 524737
trans 0
chase 7168
$ cd sw && python3 ../../ch04/rvsim/rvsim.py memtest.elf
rows 524737
cols 524737
trans 0
chase 7168
[rvsim: 270767 instructions, exit 0]
$ cd sw && riscv64-linux-gnu-nm -n memtest.elf | grep -E " [bB] (A|B|pool|order)$"
000004a4 b order
000024a4 b pool
000064a4 b B
0000a4a4 b A
```

QEMU and the Chapter 4 simulator agree on the output, and the simulator counts 270,767 instructions. The arrays occupy most of the processor's 64 KiB memory: `order` (8 KiB), `pool` (16 KiB), `B` and `A` (16 KiB each) — far larger than a small cache.

### Mastery check

1. **Predict.** For a direct-mapped data cache of 1 KiB with 16-byte blocks, roughly what miss rate do you expect for `sum_rows`? For `sum_cols`?

## Unit 15.2 — Caches in the pipeline

**One-line summary:** The simplest correct way to add blocking caches to a pipeline is to freeze it: whenever the instruction in MEM misses in the data cache, or the fetch in IF misses in the instruction cache, every pipeline register, the PC, the register file, the memory, and the predictor hold their values until the block arrives — so the pipeline's behaviour is exactly that of the perfect-memory pipeline, stretched in time.

The design is derived from Chapter 8's `pipe_bp.v` by an edit script, as Chapters 6 and 8 did: every clocked block gets the condition `if (!freeze)`, and a cache controller is added. The caches are direct-mapped with 16-byte blocks; the data cache is write-back and write-allocate, so a miss that evicts a dirty block costs two memory latencies. A data miss is served before an instruction miss, because the instruction in MEM is older.

```python
# file: rtl/make_pipe_cache.py
"""Derive pipe_cache.v from Chapter 8's pipe_bp.v: add direct-mapped instruction and data caches
(16-byte blocks, write-back, write-allocate) in front of a memory with a fixed latency. A miss
freezes the whole pipeline until the block has arrived. The caches hold tags, valid and dirty bits;
the data stays in the pipeline's memory array, so the caches decide *when* an access completes,
not *what* it returns (Chapter 10's project built the data path)."""
import sys

src, dst = sys.argv[1], sys.argv[2]
v = open(src).read()

def edit(old, new, count=1):
    global v
    assert v.count(old) == count, f"edit target found {v.count(old)} times: {old[:60]!r}"
    v = v.replace(old, new)

edit("module pipe_bp #(parameter MEMWORDS = 16384, parameter HEX = \"prog.hex\", parameter FORWARD = 1)",
     "module pipe_cache #(parameter MEMWORDS = 16384, parameter HEX = \"prog.hex\", parameter FORWARD = 1,\n"
     "                    parameter IC_SETS = 64, parameter DC_SETS = 64, parameter LAT = 20)")
edit('$display("[pipe_bp FORWARD=%0d', '$display("[pipe_cache FORWARD=%0d')

# every pipeline register, the PC, the register file, memory writes and predictor training
# hold their values while the pipeline is frozen
n = v.count("always @(posedge clk)")
edit("always @(posedge clk)", "always @(posedge clk) if (!freeze)", n)
edit("    localparam NOP = 32'h0000_0013;                       // addi x0, x0, 0: a bubble",
     "    localparam NOP = 32'h0000_0013;                       // addi x0, x0, 0: a bubble\n"
     "    wire freeze;                                          // a cache miss is being served")

caches = """
    // ===================== caches =====================
    localparam IS = IC_SETS == 0 ? 1 : IC_SETS, DS = DC_SETS == 0 ? 1 : DC_SETS;
    reg        i_valid [0:IS-1];
    reg [27:0] i_tag   [0:IS-1];                          // block address (addr >> 4) as the tag
    reg        d_valid [0:DS-1], d_dirty [0:DS-1];
    reg [27:0] d_tag   [0:DS-1];
    wire [27:0] i_blk = pc[31:4], d_blk = m_val[31:4];
    wire [31:0] ii = i_blk & (IS - 1), di = d_blk & (DS - 1);
    wire i_hit = i_valid[ii] && i_tag[ii] == i_blk;
    wire d_hit = d_valid[di] && d_tag[di] == d_blk;
    wire d_miss = DC_SETS != 0 && !rst && (m_load || m_st) && !d_hit;
    wire i_miss = IC_SETS != 0 && !rst && !i_hit;
    assign freeze = d_miss || i_miss;
    wire [8:0] d_cost = d_valid[di] && d_dirty[di] ? 2 * LAT : LAT;   // write the victim back first
    reg  [8:0] wait_cnt;
    integer s, i_misses = 0, d_misses = 0, writebacks = 0, frozen = 0;
    initial for (s = 0; s < DS || s < IS; s = s + 1) begin
        if (s < IS) i_valid[s] = 0;
        if (s < DS) begin d_valid[s] = 0; d_dirty[s] = 0; end
    end
    always @(posedge clk)
        if (rst) wait_cnt <= 0;
        else if (d_miss) begin                            // the older instruction goes first
            if (wait_cnt == d_cost - 1) begin
                wait_cnt <= 0;
                if (d_cost != LAT) writebacks = writebacks + 1;
                d_valid[di] <= 1; d_tag[di] <= d_blk; d_dirty[di] <= 0;
                d_misses = d_misses + 1;
            end else wait_cnt <= wait_cnt + 1;
        end else if (i_miss) begin
            if (wait_cnt == LAT - 1) begin
                wait_cnt <= 0;
                i_valid[ii] <= 1; i_tag[ii] <= i_blk;
                i_misses = i_misses + 1;
            end else wait_cnt <= wait_cnt + 1;
        end else if (DC_SETS != 0 && m_st)
            d_dirty[di] <= 1;                             // a store hit dirties the block
    always @(posedge clk) if (!rst && freeze) frozen = frozen + 1;

    // ===================== WB ====================="""
edit("\n    // ===================== WB =====================", caches)
edit("""                $display("[stall cycles %0d, mispredictions %0d (each flushes 2 instructions)]", stalls, flushes);""",
     """                $display("[stall cycles %0d, mispredictions %0d (each flushes 2 instructions)]", stalls, flushes);
                $display("[caches %0d + %0d sets: %0d I-misses, %0d D-misses, %0d write-backs; %0d cycles frozen; %0d cycles in all, CPI %0.3f]",
                         IC_SETS, DC_SETS, i_misses, d_misses, writebacks, frozen, cycles + frozen,
                         1.0 * (cycles + frozen) / retired);""")
open(dst, "w").write(v)
```

```verilog
// file: rtl/tb.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_cache #(.HEX("../sw/memtest.hex"), .IC_SETS(`IC), .DC_SETS(`DC), .LAT(`LAT)) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #200000000 $display("timeout");
        $finish;
    end
endmodule
```

```console
$ cd rtl && python3 make_pipe_cache.py ../../ch08/project/pipe_bp.v pipe_cache.v && for s in 0 64 256 1024 4096; do iverilog -g2012 -DIC=$s -DDC=$s -DLAT=20 -o sim pipe_cache.v tb.v && vvp -n sim | grep caches; done
[caches 0 + 0 sets: 0 I-misses, 0 D-misses, 0 write-backs; 0 cycles frozen; 274702 cycles in all, CPI 1.015]
[caches 64 + 64 sets: 76 I-misses, 20255 D-misses, 10536 write-backs; 617340 cycles frozen; 892042 cycles in all, CPI 3.295]
[caches 256 + 256 sets: 67 I-misses, 18610 D-misses, 9358 write-backs; 560700 cycles frozen; 835402 cycles in all, CPI 3.085]
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 259300 cycles frozen; 534002 cycles in all, CPI 1.972]
[caches 4096 + 4096 sets: 67 I-misses, 3593 D-misses, 0 write-backs; 73200 cycles frozen; 347902 cycles in all, CPI 1.285]
$ cd rtl && iverilog -g2012 -DIC=1024 -DDC=1024 -DLAT=20 -o sim pipe_cache.v tb.v && vvp -n sim | grep -v -E "WARNING|finish"
rows 524737
cols 524737
trans 0
chase 7168
[pipe_cache FORWARD=1: exit 0, 270767 instructions, 274702 cycles, CPI 1.015]
[stall cycles 29, mispredictions 1952 (each flushes 2 instructions)]
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 259300 cycles frozen; 534002 cycles in all, CPI 1.972]
```

With perfect memory (0 sets: no caches) the pipeline runs at CPI 1.015 — the program's branches are mostly loop branches, well predicted, and it has few load-use stalls. With 1 KiB caches the CPI is 3.3: two thirds of all cycles are spent frozen. Growing the caches to 16 KiB cuts the data misses by more than half; at 64 KiB, the whole data set fits, only compulsory misses remain, and no block is ever written back during the run. The program's output is the same in every configuration — the freeze changes timing, never results.

### Deep & Tricky

**DT-41. Freezing a pipeline is correct only if *everything* with state freezes — including side effects that are easy to forget: the branch predictor's training, the store to memory, the register-file write, the performance counters (here, `retired` would otherwise count the instruction in WB once per frozen cycle), and the simulation's own `ecall` handler (which would print the output many times). The edit script freezes every clocked block in the design and then adds the cache controller with the raw clock; a design that freezes "the pipeline registers" by hand misses one sooner or later.** *(Practice: the "stall-everything" discipline of in-order pipelines, Hennessy & Patterson, Appendix C; verified here by the identical output and instruction count in every configuration)*

### Mastery check

1. **Explain.** Why does the 4,096-set (64 KiB) data cache show no write-backs at all?
2. **Explain.** Why must the data miss be served before the instruction miss when both happen in the same cycle?

## Topic 15B — Checking the hardware against models

## Unit 15.3 — Predicting the hardware from software models

**One-line summary:** Because the frozen pipeline is the perfect-memory pipeline stretched in time, its total cycle count should be the perfect-memory count plus the memory latency times (instruction misses + data misses + write-backs) — and those miss counts can be computed by feeding the instruction-set simulator's address trace through the trace-driven cache simulator, without any RTL.

```python
# file: model/predict.py
"""Predict the capstone CPU's cache behaviour from the instruction-set simulator of Chapter 4
and the cache simulator of Chapter 10: run the program, collect every instruction fetch and data
access, and feed them to two direct-mapped caches with 16-byte blocks."""
import sys
sys.path.insert(0, "../../ch04/rvsim")
sys.path.insert(0, "../../ch10/sim")
from rvsim import CPU
from cachesim import Cache

class TracingCPU(CPU):
    def __init__(self):
        super().__init__(mem_size=1 << 16)
        self.itrace, self.dtrace, self.fetching = [], [], False
    def step(self):
        self.itrace.append(self.pc)
        self.fetching = True                     # step()'s first ld is the instruction fetch
        super().step()
    def ld(self, addr, size, signed):
        if self.fetching:
            self.fetching = False
        else:
            self.dtrace.append((addr, False))
        return super().ld(addr, size, signed)
    def st(self, addr, size, v):
        self.dtrace.append((addr, True))
        super().st(addr, size, v)

cpu = TracingCPU()
cpu.load_elf("../sw/memtest.elf")
cpu.run()
LAT, BASE = 20, 274702                           # memory latency; cycles of the CPU with perfect memory
print(f"{cpu.count} instructions, {len(cpu.dtrace)} data accesses")
for sets in (64, 256, 1024, 4096):
    ic, dc = Cache(16 * sets, 16, 1), Cache(16 * sets, 16, 1)
    for a in cpu.itrace:
        ic.access(a)
    for a, w in cpu.dtrace:
        dc.access(a, w)
    cycles = BASE + LAT * (ic.misses + dc.misses + dc.writebacks)
    print(f"{sets:5d} sets ({16 * sets:6d} bytes each): {ic.misses:5d} I-misses, {dc.misses:6d} D-misses, "
          f"{dc.writebacks:6d} write-backs -> predicted {cycles} cycles, CPI {cycles / cpu.count:.3f}")
```

```console
$ cd model && python3 predict.py
270767 instructions, 41073 data accesses
   64 sets (  1024 bytes each):    68 I-misses,  20255 D-misses,  10536 write-backs -> predicted 891882 cycles, CPI 3.294
  256 sets (  4096 bytes each):    64 I-misses,  18610 D-misses,   9358 write-backs -> predicted 835342 cycles, CPI 3.085
 1024 sets ( 16384 bytes each):    64 I-misses,   7955 D-misses,   4943 write-backs -> predicted 533942 cycles, CPI 1.972
 4096 sets ( 65536 bytes each):    64 I-misses,   3593 D-misses,      0 write-backs -> predicted 347842 cycles, CPI 1.285
```

The data-side predictions match the RTL **exactly** at every size — every miss and every write-back. The instruction side differs slightly: the RTL reports 76 instruction misses with 1 KiB caches where the model predicts 68, and 67 where the model predicts 64 for the larger caches. The model sees only the instructions that execute; the pipeline also fetches the two instructions after each mispredicted branch, before the branch resolves in EX, and occasionally those wrong-path fetches miss — costing cycles and, in the small cache, evicting a useful block. The total differs by exactly 20 cycles per extra miss.

### Deep & Tricky

**DT-42. A trace-driven model is only as faithful as its trace: the ISA-level trace contains the correct path only, so it misses wrong-path instruction fetches (and, in an out-of-order core, wrong-path loads, which also bring lines into the cache — the effect Spectre exploits), prefetches, and page-table walks. When model and hardware disagree, the first question is not "which is wrong?" but "what is in one trace and not the other?" — here, 8 instruction fetches after mispredicted branches.** *(Practice; Uhlig & Mudge, "Trace-driven memory simulation: a survey", ACM Computing Surveys 1997)*

### Mastery check

1. **Calculate.** From the 1 KiB numbers, verify that 274,702 + 20 × (76 + 20,255 + 10,536) equals the RTL's total cycle count.

## Unit 15.4 — Fast simulation with Verilator

**One-line summary:** Verilator compiles synthesizable Verilog into C++ and runs it tens to hundreds of times faster than an event-driven simulator such as Icarus, at the cost of being a two-state, cycle-based simulator — which is why this book has run every design on both.

```console
$ cd rtl && time (iverilog -g2012 -DIC=1024 -DDC=1024 -DLAT=20 -o sim pipe_cache.v tb.v && vvp -n sim | grep caches)
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 259300 cycles frozen; 534002 cycles in all, CPI 1.972]

real	0m7.169s
user	0m7.143s
sys	0m0.005s
$ cd rtl && verilator --binary --timing -Wno-fatal -Wno-lint -Wno-style -O3 --top-module tb -DIC=1024 -DDC=1024 -DLAT=20 -o vsim pipe_cache.v tb.v > build.log 2>&1 && time ./obj_dir/vsim | grep caches
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 259300 cycles frozen; 534002 cycles in all, CPI 1.972]

real	0m0.118s
user	0m0.120s
sys	0m0.000s
$ cd rtl && for lat in 20 50 100; do verilator --binary --timing -Wno-fatal -Wno-lint -Wno-style -O3 --top-module tb -DIC=1024 -DDC=1024 -DLAT=$lat -Mdir obj_$lat -o vsim pipe_cache.v tb.v > /dev/null 2>&1 && ./obj_$lat/vsim | grep caches; done
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 259300 cycles frozen; 534002 cycles in all, CPI 1.972]
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 648250 cycles frozen; 922952 cycles in all, CPI 3.409]
[caches 1024 + 1024 sets: 67 I-misses, 7955 D-misses, 4943 write-backs; 1296500 cycles frozen; 1571202 cycles in all, CPI 5.803]
```

Both simulators produce the same line, cycle for cycle; the compiled simulation runs in about a tenth of a second against several seconds for Icarus (plus a few seconds of C++ compilation, paid once per configuration). The latency sweep shows how quickly a blocking cache loses: at 100 cycles, the program that runs at CPI 1.015 with perfect memory runs at CPI 5.8 even with 16 KiB caches — the case for the non-blocking caches, prefetching, and out-of-order execution of the earlier chapters, and for bigger caches.

### Mastery check

1. **Calculate.** Using the miss and write-back counts, predict the 100-cycle result before looking at it.

## Chapter 15 Project — Associativity in the capstone

### Requirements

Make the capstone's data cache two-way set-associative with LRU replacement, keeping its total size (half as many sets), by writing an edit script for `pipe_cache.v`. Predict its misses with the Chapter 10 simulator first, then run the RTL with Verilator for 1, 4, and 16 KiB data caches and check that the two agree exactly. Does associativity help this program? Explain from the three Cs.

### Reference solution

```python
# file: project/make_pipe_cache2.py
"""Make the data cache of pipe_cache.v two-way set-associative with LRU replacement:
each set gets a second way, a hit may be in either, and a miss replaces the least recently used."""
import sys

src, dst = sys.argv[1], sys.argv[2]
v = open(src).read()

def edit(old, new):
    global v
    assert v.count(old) == 1, f"edit target not found exactly once: {old[:60]!r}"
    v = v.replace(old, new)

edit("module pipe_cache #", "module pipe_cache2 #")
edit('$display("[pipe_cache FORWARD', '$display("[pipe_cache2 FORWARD')
edit("""    reg        d_valid [0:DS-1], d_dirty [0:DS-1];
    reg [27:0] d_tag   [0:DS-1];""",
"""    reg        d_valid [0:1][0:DS-1], d_dirty [0:1][0:DS-1];
    reg [27:0] d_tag   [0:1][0:DS-1];
    reg        d_lru   [0:DS-1];                          // the way to replace next""")
edit("""    wire d_hit = d_valid[di] && d_tag[di] == d_blk;""",
"""    wire d_hit0 = d_valid[0][di] && d_tag[0][di] == d_blk;
    wire d_hit1 = d_valid[1][di] && d_tag[1][di] == d_blk;
    wire d_hit = d_hit0 || d_hit1;
    wire d_way = d_hit1;                                  // the way that hit
    wire d_vic = d_lru[di];                               // the way to fill on a miss""")
edit("""    wire [8:0] d_cost = d_valid[di] && d_dirty[di] ? 2 * LAT : LAT;""",
"""    wire [8:0] d_cost = d_valid[d_vic][di] && d_dirty[d_vic][di] ? 2 * LAT : LAT;""")
edit("""        if (s < DS) begin d_valid[s] = 0; d_dirty[s] = 0; end""",
"""        if (s < DS) begin d_valid[0][s] = 0; d_valid[1][s] = 0; d_dirty[0][s] = 0; d_dirty[1][s] = 0; d_lru[s] = 0; end""")
edit("""                d_valid[di] <= 1; d_tag[di] <= d_blk; d_dirty[di] <= 0;""",
"""                d_valid[d_vic][di] <= 1; d_tag[d_vic][di] <= d_blk; d_dirty[d_vic][di] <= 0;""")
edit("""        end else if (DC_SETS != 0 && m_st)
            d_dirty[di] <= 1;                             // a store hit dirties the block""",
"""        end else if (DC_SETS != 0 && (m_load || m_st)) begin   // a hit: update LRU, maybe dirty
            d_lru[di] <= ~d_way;
            if (m_st) d_dirty[d_way][di] <= 1;
        end""")
open(dst, "w").write(v)
```

```verilog
// file: project/tb2.v
module tb;
    reg clk = 0, rst = 1;
    always #5 clk = ~clk;
    wire [31:0] a0;
    pipe_cache2 #(.HEX("../sw/memtest.hex"), .IC_SETS(`IC), .DC_SETS(`DC), .LAT(`LAT)) cpu(clk, rst, a0);
    initial begin
        #12 rst = 0;
        #200000000 $display("timeout");
        $finish;
    end
endmodule
```

```python
# file: project/predict2.py
"""The model's prediction for a two-way LRU data cache with the same total size."""
import sys
sys.path.insert(0, "../model")
sys.argv = ["predict2"]
import io, contextlib
with contextlib.redirect_stdout(io.StringIO()):
    from predict import cpu, Cache, LAT, BASE        # reuses the traces collected by predict.py
for sets in (32, 128, 512):                          # two ways: same bytes as 64, 256, 1024 sets
    ic, dc = Cache(32 * sets, 16, 1), Cache(32 * sets, 16, 2)
    for a in cpu.itrace:
        ic.access(a)
    for a, w in cpu.dtrace:
        dc.access(a, w)
    cycles = BASE + LAT * (ic.misses + dc.misses + dc.writebacks)
    print(f"2-way, {sets:4d} sets ({32 * sets:5d} bytes): {dc.misses:6d} D-misses, {dc.writebacks:6d} write-backs, "
          f"predicted CPI {cycles / cpu.count:.3f}")
```

```console
$ cd project && python3 make_pipe_cache2.py ../rtl/pipe_cache.v pipe_cache2.v && for s in 32 128 512; do verilator --binary --timing -Wno-fatal -Wno-lint -Wno-style -O3 --top-module tb -DIC=$((2*s)) -DDC=$s -DLAT=20 -Mdir obj_$s -o vsim pipe_cache2.v tb2.v > /dev/null 2>&1 && ./obj_$s/vsim | grep caches; done
[caches 64 + 32 sets: 76 I-misses, 20031 D-misses, 10427 write-backs; 610680 cycles frozen; 885382 cycles in all, CPI 3.270]
[caches 256 + 128 sets: 67 I-misses, 18462 D-misses, 9300 write-backs; 556580 cycles frozen; 831282 cycles in all, CPI 3.070]
[caches 1024 + 512 sets: 67 I-misses, 7727 D-misses, 4812 write-backs; 252120 cycles frozen; 526822 cycles in all, CPI 1.946]
$ cd project && python3 predict2.py
2-way,   32 sets ( 1024 bytes):  20031 D-misses,  10427 write-backs, predicted CPI 3.269
2-way,  128 sets ( 4096 bytes):  18462 D-misses,   9300 write-backs, predicted CPI 3.070
2-way,  512 sets (16384 bytes):   7727 D-misses,   4812 write-backs, predicted CPI 1.945
```

Model and RTL agree on every data miss and every write-back. Associativity barely helps — 1 to 3% fewer data misses — because this program's misses are capacity misses: each kernel sweeps 16 KiB or more through a cache of at most 16 KiB, and no arrangement of the blocks within the cache can keep data that does not fit. The predicted CPI differs from the RTL in the third decimal place only, through the same wrong-path instruction fetches as before.

### Review checklist

- Make the instruction cache 2-way as well. How much does it matter for this program, and why?
- Add a one-entry "next block" prefetcher to the data cache: on a miss to block b, also fetch b + 1 into a buffer. Predict its effect on `sum_rows` and `chase` before you build it.
- Replace the freeze with a non-blocking data cache that lets independent instructions continue past a load miss. What new hazards must the pipeline detect?

## Answers to the Chapter 15 prediction questions

**Unit 15.1, Q1.** `sum_rows` reads 4 ints per 16-byte block: one miss per 4 loads, 25%. `sum_cols` jumps 256 bytes per access: with 64 sets of 16 bytes (1 KiB), addresses 256 bytes apart map to only 4 different sets, so almost every access misses — close to 100%.

**Unit 15.2, Q1.** All of the program's data (about 58 KiB including the stack) fits in 64 KiB of cache, so no block is ever evicted; dirty blocks would be written back only when evicted, which never happens before the program ends. **Q2.** It is older: if the instruction cache were filled first, the pipeline would still be frozen for the data miss — and in a design that let the front end proceed, serving the younger instruction first could reorder memory effects.

**Unit 15.3, Q1.** 20 × (76 + 20,255 + 10,536) = 20 × 30,867 = 617,340 frozen cycles; 274,702 + 617,340 = 892,042. ✓

**Unit 15.4, Q1.** 274,702 + 100 × (67 + 7,955 + 4,943) = 274,702 + 1,296,500 = 1,571,202 cycles; CPI = 1,571,202 / 270,767 ≈ 5.80.

# Appendix A — Index of Deep & Tricky entries and Patterns

Every *Deep & Tricky* entry (DT) and *Pattern* (P) in the book, in number order, with the unit where it is explained. Use it for revision: read each title and explain the entry before looking it up.

## Deep & Tricky

| # | Entry | Where |
| --- | --- | --- |
| DT-1 | The ISA specifies *what* each instruction does, not *how fast*: two correct implementations of the same ISA can differ in speed by 10× or more on the same program, and a program's speed can depend on details (alignment, branch patterns, memory layout) that the ISA does not mention at all. | Unit 0.1 |
| DT-2 | Clock frequency alone does not determine performance, and neither does instruction count: a "faster" chip with a higher clock can be slower if its CPI is worse, and a compiler change that *adds* instructions can make a program faster if it removes stalls. | Unit 0.2 |
| DT-3 | A benchmark that walks memory sequentially, or in any regular pattern, measures the prefetcher, not the memory: hardware prefetchers detect constant strides and fetch ahead, so sequential latency looks like L1 latency even for gigabyte arrays. Random dependent loads are the only way to see true latency. | Unit 0.3 |
| DT-4 | Instructions that are executed speculatively and then squashed leave no trace in architectural state (registers, memory) — exactly as the ISA promises — but they do change microarchitectural state such as cache contents, which persists after the squash and can be measured. Spectre and Meltdown exploit this gap between the architectural and the microarchitectural. | Unit 0.5 |
| DT-5 | A CMOS gate's output is *inverting* by construction: a single stage of nMOS pull-down and pMOS pull-up can only produce functions that go from 1 to 0 when inputs rise (NOT, NAND, NOR, AND-OR-INVERT…). Non-inverting functions such as AND, OR, and the plain buffer need two stages. | Unit 1.1 |
| DT-6 | Verilog signals have four values — 0, 1, `x` (unknown), and `z` (high impedance, not driven) — and the ordinary equality operators propagate unknowns: `x == 1` is `x`, and an `if` whose condition is `x` takes the `else` branch. A testbench that checks `if (y != expected)` therefore silently *passes* when `y` is `x`. Use the case-equality operators `===` and `!==`, which compare all four values literally. | Unit 1.2 |
| DT-7 | A Verilog expression is sized by its *context*: in `assign s = a + b;` with 8-bit `a`, `b`, and `s`, the carry out is silently discarded, while `assign {cout, s} = a + b;` makes the context 9 bits wide and keeps it. The same rule makes `(a + b) >> 1` lose the carry before shifting when the result is assigned to an 8-bit variable — the average of 200 and 100 becomes 22. | Unit 1.4 |
| DT-8 | In Verilog, if *any* operand of an expression is unsigned, the whole expression is evaluated as unsigned — so comparing a `signed` register with an unsigned one, or with a part-select (which is always unsigned), silently becomes an unsigned comparison, and −1 < 0 is false. | Unit 1.5 |
| DT-9 | In sequential `always @(posedge clk)` blocks, use non-blocking assignments (`<=`); in combinational `always @(*)` blocks, use blocking ones (`=`). With blocking assignments, two flip-flops clocked by the same edge can read each other's *new* value or old value depending on which block the simulator happens to run first — a simulation race that does not exist in hardware. | Unit 2.2 |
| DT-10 | A RAM written and read at the same address in the same clock cycle returns *either* the old or the new data, depending on how it is written in Verilog (or how the RAM macro is configured): "read-first" returns the old value, "write-first" the new one. A design that assumes one and is mapped to a memory configured for the other works in simulation and fails in hardware. | Unit 2.5 |
| DT-11 | Simulators disagree, and simulators have bugs. Verilator ignores immediate `assert` statements unless it is run with `--assert`; in version 5.020 an `assert … else begin errors++; … end` then *executed the action block on every check* (the FIFO testbench of this chapter's project reported 115,698 "errors" against Icarus's 0), and with `--assert` the C++ that Verilator generated for the same block failed to compile. Running every testbench on two simulators, and starting each verification effort by checking that a deliberately broken design fails, catches tool problems as well as design problems. | Unit 3.2 |
| DT-12 | Formal tools prove exactly the property you wrote, over exactly the inputs you allowed, and nothing else: a miter that compares only some outputs, a property with an unintended `x` or an over-constrained assumption, or a design with uninitialised state can "pass" a proof that says nothing about the bug you care about. A proof that succeeds too easily deserves suspicion — check it by planting a bug and confirming the proof fails. | Unit 3.3 |
| DT-13 | A functional (zero-delay) simulation says nothing about timing: the RTL simulation of a design passes at any clock frequency, because `#` delays are absent and every assignment completes "instantly". Timing is checked separately, by static timing analysis on the synthesised netlist, and the clock constraint you give the tool is part of the design. | Unit 3.4 |
| DT-14 | A two-flip-flop synchroniser is correct for a *single* bit. Synchronising a multi-bit value (a counter, an address) bit by bit is wrong: each bit may resolve differently at the same edge, so the receiving domain can see a value that never existed — half old bits, half new. Multi-bit values cross clock domains as Gray codes (only one bit changes per step), through handshakes, or through asynchronous FIFOs, which use exactly those Gray-coded pointers. | Unit 3.5 |
| DT-15 | RISC-V has no "load a 32-bit constant" instruction: `li x9, 0x12345678` is a pseudo-instruction that the assembler expands to `lui` + `addi`, and because `addi` sign-extends its 12-bit immediate, the `lui` value must be *rounded up* by one whenever bit 11 of the constant is 1. Hand-written `lui`/`addi` pairs that forget this produce constants that are off by 4096. | Unit 4.2 |
| DT-16 | The calling convention is not part of the ISA: the hardware treats `x1` and `x8` like any other register, and only the ABI document makes `x1` the return address and `x8`–`x9`, `x18`–`x27` callee-saved. Code that breaks the convention runs perfectly — until it is linked with code that follows it, at which point a callee-saved register silently changes across a call. | Unit 4.4 |
| DT-17 | An optimising compiler may replace your loops with calls to library functions it recognises — `memset`, `memcpy`, `strlen` — even when your source never mentions them, and even with `-fno-builtin`; in freestanding code (kernels, bootloaders, firmware, this book's test programs) that turns into a link error or, worse, a call to a different implementation than you think. | Unit 4.5 |
| DT-18 | Fixed-length instructions make decoding easy but cost code size: the same program compiled for RV32I is typically 20–30% larger than with the 16-bit compressed extension (RV32IC), and often larger than x86 — which matters for instruction caches and embedded flash, and is why almost every RISC-V core implements the C extension. | Unit 4.6 |
| DT-19 | A microarchitecture is correct only if it is correct for *every* instruction in *every* context, and a test program exercises a fraction of them: both processors in this chapter pass the test programs, yet neither implements `fence.i`, misaligned accesses (which RV32I allows implementations to trap or handle), or any exception — an illegal instruction simply executes as something else. Commercial processors are checked against the ISA with randomly generated instruction streams (e.g. Google's RISCV-DV) and formal proofs (e.g. the riscv-formal framework) in addition to programs. | Unit 5.6 |
| DT-20 | Forwarding must take the *youngest* matching value: if both the instruction in MEM and the one in WB write the register being read, the one in MEM is newer and must win. Reversing the priority of the two forwarding paths passes many tests and silently computes with stale values when the same register is written twice in a row — which compilers do constantly (`addi a0, a0, 1; addi a0, a0, 1`). | Unit 6.3 |
| DT-21 | A flushed instruction must have *no* architectural effect — no register write, no memory write, no exception, no system call. In a five-stage pipeline that resolves branches in EX this is automatic, because the wrong-path instructions are still in IF and ID; in deeper pipelines and out-of-order processors, wrong-path instructions reach execution, and the hardware must suppress or undo all of their effects — which is exactly the property Spectre (Chapter 0) showed does not extend to microarchitectural state such as the cache. | Unit 6.4 |
| DT-22 | Deeper pipelines raise the clock frequency only until the per-stage register overhead and the growing hazard penalties win: each extra stage adds a fixed setup + clock-to-Q delay, and lengthens the distance (in cycles) between a branch's fetch and its resolution, so mispredictions cost more. Studies of the trade-off put the optimum for general-purpose code at roughly 15–25 FO4 gate delays of logic per stage; the Pentium 4's 31-stage Prescott core went past it and was abandoned for shorter pipelines. | Unit 6.6 |
| DT-23 | An out-of-order core does not make a single dependent chain faster: its benefit is overlapping *independent* work — other chains, other loop iterations, cache misses behind each other. A linked-list traversal, where each load's address comes from the previous load, runs at the same speed on a 4-wide out-of-order core as on a simple in-order one; Unit 0.3's pointer-chasing benchmark relied on exactly this. | Unit 7.4 |
| DT-24 | A history-based predictor can *memorise* "random" data if the same data is processed repeatedly: in `threshold_random` the 400 random values are scanned five times, and with 10 bits of local history almost every position in the sequence has a unique context, so the predictor learns each outcome during the first pass and repeats it afterwards — 9.5% wrong where a coin toss is 50%. Benchmarks that loop over a fixed input overstate predictor accuracy for this reason, and predictor studies use inputs larger than the predictor's state or different data per pass. | Unit 8.4 |
| DT-25 | Returns are the most frequent indirect branches in most programs, and a BTB predicts them poorly whenever a function has more than one caller; a small return address stack (8–32 entries) predicts them almost perfectly, failing only when call depth overflows it or when the program does not return the way it called — `longjmp`, exceptions, coroutines, and stack-switching code. | Unit 8.5 |
| DT-26 | Floating-point addition is not associative — `(a + b) + c` can differ from `a + (b + c)` in the last bits, or by much more with mixed magnitudes — so a compiler may not reorder a sum into vector partial sums without permission; `-ffast-math` (or `-fassociative-math`) grants it, and with it the result may change slightly. The same rule makes parallel reductions on GPUs and multicores give run-to-run differences. | Unit 9.3 |
| DT-27 | The three-Cs split is defined relative to a fully associative LRU cache, and LRU is not optimal, so the "conflict" count can be negative or mislabel misses: in the random trace above, the 4-way cache shows *more* capacity misses (13,718) than the direct-mapped one (12,967), with the same total — the classification moved misses between columns, not the cache. | Unit 10.3 |
| DT-28 | A power-of-two row length turns a column walk into a conflict-miss machine: with 256 doubles per row, consecutive elements of a column are 2,048 bytes apart, and in a 32 KiB 8-way cache (64 sets of 64-byte blocks, so addresses 4,096 bytes apart share a set) the whole column maps to only 2 sets — 16 blocks — instead of the 256 it needs; with 250 doubles per row the same column spreads over all 64 sets. The miss rate jumps from 12.5% to 50%, and on the real machine the N = 1024 multiply takes five to seven times as long as N = 1000, which does only 7% fewer operations. | Unit 10.6 |
| DT-29 | A cache testbench that never evicts a dirty block cannot find write-back bugs, and one that compares loads only against values the testbench itself just wrote cannot find stale-data bugs: the reference model must be a complete memory image, the address stream must overflow every set (here the rare jumps across 64 KiB do that), and stores must be followed by loads of the same address after an eviction. | Chapter 10 Project |
| DT-30 | The hardware sets the Accessed and Dirty bits — or, on implementations without hardware update, raises a page fault so that software can set them — and the OS depends on them: A tells the page-replacement algorithm which pages were used recently, and D tells it which pages must be written to disk before their frame is reused. A walker that sets D on a load, or forgets it on a store, silently corrupts files. | Unit 11.2 |
| DT-31 | A page walk's memory accesses go through the data caches, so a TLB miss whose PTEs are cached costs a few nanoseconds, while one whose PTEs are in DRAM costs several DRAM latencies in series — and in a virtual machine translation is two-dimensional: each of the guest's page-table pointers is itself a guest-physical address that must be translated through the hypervisor's tables, so a 4-level guest walk on a 4-level host can take up to 24 memory accesses. | Unit 11.3 |
| DT-32 | Reading a never-written anonymous page does not allocate memory: Linux maps it to a single shared, read-only page of zeros, so the read faults, and the first write faults *again* to replace the zero page with a private frame — which is why the program above takes 16,384 faults for the reads and another 16,384 for the writes. Benchmarks that "warm up" a buffer by reading it measure the zero page, not their memory. | Unit 11.4 |
| DT-33 | If a virtually indexed cache used index bits above the page offset, two virtual addresses that map to the same physical address (synonyms, created by shared memory or `mmap`ing a file twice) could sit in two different sets, and a write through one would not be seen through the other. Keeping index bits inside the page offset makes synonyms impossible by construction; caches that break the rule must detect synonyms in hardware or rely on the OS to colour pages. | Unit 11.5 |
| DT-34 | Throughput and fairness are different goals: the scheduler that maximises total requests served (FR-FCFS) gives the worst slowdown to the program with the least row-buffer locality, so in a shared system "the memory is fast" can be true on average while one application runs eight times slower than alone. Evaluating a multicore memory system therefore needs multi-program metrics — weighted speedup (system throughput) and maximum slowdown (unfairness) — not average latency. | Unit 12.4 |
| DT-35 | A mitigation that tracks a fixed number of aggressor rows can be defeated by an access pattern with more aggressors or decoys than its table: TRRespass (Frigo et al., IEEE S&P 2020) found that DDR4 modules advertised as RowHammer-free flipped bits with many-sided patterns, and Blacksmith (Jattke et al., IEEE S&P 2022) used non-uniform frequency patterns to flip bits on all 40 DDR4 modules tested. Deterministic, small-state defences need a proof that no pattern evades them; probabilistic ones (PARA) and counter-per-row designs with guaranteed bounds (Graphene, MICRO 2020) give one. | Unit 12.5 |
| DT-36 | DMA writes memory behind the processor's back, so on systems whose caches are not coherent with DMA, a driver must flush cache lines before the device reads a buffer and invalidate them before the processor reads data the device wrote — and must not let the buffer share a cache line with other data, or invalidating it destroys the neighbour's dirty bytes. Many embedded RISC-V and ARM systems are like this; x86 and server ARM make DMA cache-coherent in hardware. | Unit 13.1 |
| DT-37 | `write` returning success means only that the data reached the operating system's page cache; it reaches the device later, and a crash or power failure before then loses it. Durability needs `fsync`/`fdatasync` (and, for a newly created file, an `fsync` of its directory), and the error from a failed write-back may be reported only by the next `fsync` — once: PostgreSQL retried a failed `fsync`, which then "succeeded", and silently lost data until 2018. | Unit 13.2 |
| DT-38 | An SSD's speed and lifetime depend on its history, not just the current request: a fresh or freshly trimmed drive writes at full speed with write amplification near 1, but after the whole capacity has been written randomly, garbage collection runs alongside every write, and sustained random-write throughput can drop by a factor of several. Benchmarks must precondition the drive (fill it, then write randomly until steady state) — this simulation fills the drive once before measuring for the same reason — and file systems should issue TRIM so that the FTL knows which pages are free. | Unit 13.4 |
| DT-39 | `volatile` in C stops the *compiler* from removing or reordering accesses to that variable, but emits no fence, so it says nothing about the order in which other cores see them; the code above with `volatile` and no fence produces the non-SC outcome on x86, and on ARM or RISC-V even the message-passing idiom breaks. Portable code uses C11/C++11 atomics with the right memory order, which compile to the fences each architecture needs — `memory_order_seq_cst` stores become `xchg` on x86, and release/acquire become `fence rw,w` / `fence r,rw` (or `.aqrl` bits) on RISC-V. | Unit 14.2 |
| DT-40 | A lock-free algorithm built on compare-and-swap can be fooled by the ABA problem: a thread reads A, another thread changes the location to B and back to A, and the first thread's CAS succeeds although the structure changed underneath — for example, a node was popped from a lock-free stack, freed, and reallocated at the same address. LR/SC does not have this flaw, because any intervening write breaks the reservation; with CAS, the fixes are version counters in a double-width CAS, hazard pointers, or epoch-based reclamation. | Unit 14.5 |
| DT-41 | Freezing a pipeline is correct only if *everything* with state freezes — including side effects that are easy to forget: the branch predictor's training, the store to memory, the register-file write, the performance counters (here, `retired` would otherwise count the instruction in WB once per frozen cycle), and the simulation's own `ecall` handler (which would print the output many times). The edit script freezes every clocked block in the design and then adds the cache controller with the raw clock; a design that freezes "the pipeline registers" by hand misses one sooner or later. | Unit 15.2 |
| DT-42 | A trace-driven model is only as faithful as its trace: the ISA-level trace contains the correct path only, so it misses wrong-path instruction fetches (and, in an out-of-order core, wrong-path loads, which also bring lines into the cache — the effect Spectre exploits), prefetches, and page-table walks. When model and hardware disagree, the first question is not "which is wrong?" but "what is in one trace and not the other?" — here, 8 instruction fetches after mispredicted branches. | Unit 15.3 |

## Patterns

| # | Pattern | Where |
| --- | --- | --- |
| P-1 | Measure the end-to-end time on a representative workload first; then use instruction counts, CPI, and miss rates to explain it. | Unit 0.2 |
| P-2 | Test every combinational block exhaustively against an independent reference whenever the input space is small (up to about 2²⁰ combinations), and randomly plus at the corners when it is not. | Unit 1.3 |
| P-3 | Write every FSM in three parts — a clocked state register, an `always @(*)` next-state `case` with a `default`, and output logic — and give the states names with `localparam`. | Unit 2.4 |
| P-4 | In every `always @(*)` block, assign a default value to every output first, then override it in the branches; in every `case`, include a `default`. | Unit 3.1 |
| P-5 | Make the innermost loop walk memory with stride 1; when it cannot, block the loops so that the data reused by the inner loops fits in a cache level, and never let a hot stride be a large power of two. | Unit 10.6 |
| P-6 | Treat reliability failures as security problems: any fault an attacker can induce on purpose — a bit flip from RowHammer, a timing fault from undervolting (Plundervolt), a speculative side effect — is an attack primitive, so mitigations must hold against the worst access pattern, not the typical one. | Unit 12.5 |
