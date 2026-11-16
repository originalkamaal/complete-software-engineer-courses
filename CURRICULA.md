# Six companion books — curricula

Same method as *Go from Zero to Production*: every unit has a one-line summary, mental model, lesson with runnable code and real output, *Deep & Tricky* entries labelled by source of truth (standard / implementation / platform), patterns with trade-offs, common mistakes, and a mastery check; every chapter ends with a verified project; appendices give an index, glossary, and version table.

Writing order (each builds on the one before): **C → C++ → Rust → Java → Linux Kernel & OS Development → Swift**.

Verification toolchains available here: GCC 13.3, Clang 18 (with ASan/UBSan/TSan), Valgrind, GDB, NASM, QEMU 8.2 + GRUB (for booting the OS book's kernel), OpenJDK 21, Rust 1.97. **No Swift toolchain can be installed in this workspace**, so the Swift book's code will be marked "not compiled" unless a toolchain becomes available.

---

## 0. The Go Apprenticeship (Go) — delivered first

0 Computing foundations and toolchain · 1 Core language fundamentals · 2 Composite data and memory · 3 Types, methods, interfaces, errors, packages, generics · 4 Testing, tooling, debugging · 5 Standard library and I/O · 6 Data structures, algorithms, complexity · 7 Concurrency · 8 Networking, HTTP, RPC · 9 Databases, persistence, caching · 10 Performance engineering and runtime internals · 11 Production engineering · 12 Distributed systems and system design · 13 Geospatial computing and routing algorithms · 14 Capstone: a production routing backend

## 1. C from Zero to Systems (C17, with C23 marked)

| Ch | Title                                                                                                                                   | Project                                    |
| -- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| 0  | How C becomes a running program: preprocess, compile, assemble, link; object files, ELF, the C abstract machine                         | Inspect every stage of a build by hand     |
| 1  | Core language: types, integer promotions and conversions, operators, control flow, functions                                            | Number-format converter                    |
| 2  | Pointers and arrays: addresses, pointer arithmetic, array decay, strings, `const`                                                     | String utility library with tests          |
| 3  | Memory and data layout: storage durations, `malloc`/`free`, structs, unions, alignment, padding, enums, bit-fields, ownership rules | Dynamic array and arena allocator          |
| 4  | Program structure: headers, linkage, the preprocessor, multi-file builds, Make and CMake, static and shared libraries                   | A reusable library + CLI, built three ways |
| 5  | Undefined behaviour and correctness: the standard, warnings, sanitizers, Valgrind, GDB, unit testing                                    | Find and fix 10 planted bugs               |
| 6  | Standard library and I/O: stdio, buffering, files, `errno`, safe string handling, time, C11–C23 features                             | `wc`/`grep` clones                     |
| 7  | Data structures and algorithms in C: lists, hash tables, trees, heaps, graphs; generic code with `void *` and macros                  | Generic hash map + benchmark               |
| 8  | POSIX systems programming: processes, `fork`/`exec`, file descriptors, pipes, signals, `mmap`                                     | A Unix shell                               |
| 9  | Concurrency: pthreads, mutexes, condition variables, C11 atomics, memory ordering, thread pools                                         | Parallel file indexer                      |
| 10 | Networking: sockets, TCP/UDP, non-blocking I/O, `epoll`, protocol parsing                                                             | Event-driven HTTP server                   |
| 11 | Performance and the machine: reading assembly, calling conventions/ABI, caches, SIMD intrinsics, profiling                              | Optimise a hot loop 10× and prove it      |
| 12 | Security and robustness: overflows, format strings, integer bugs, CERT C, fuzzing                                                       | Fuzz and harden a parser                   |
| 13 | Capstone: an in-memory key-value database server with persistence (write-ahead log, snapshots, `epoll`)                               | Design doc, build, load test, review       |

## 2. C++ from Zero to Expert (C++20, with C++23/26 marked)

0 Toolchain and the C++ model (translation units, ODR, compilation model, modules preview) · 1 Core language and types (references, `auto`, `constexpr`, initialisation rules) · 2 Classes and RAII (constructors, destructors, rule of 0/3/5) · 3 Value semantics and move semantics (lvalues/rvalues, perfect forwarding) · 4 The standard library: containers, iterators, algorithms, ranges · 5 Smart pointers, ownership, and resource management · 6 Errors: exceptions, exception safety, `std::expected` · 7 Templates I: function and class templates, specialisation, CTAD · 8 Templates II: concepts, variadic templates, metaprogramming, `constexpr`/`consteval` · 9 Object-oriented design: inheritance, virtual dispatch, vtables, type erasure · 10 Concurrency: threads, atomics, memory model, coroutines · 11 Build, test, tooling: CMake, sanitizers, GoogleTest/Catch2, static analysis · 12 Performance: allocators, data-oriented design, cache, benchmarking · 13 Undefined behaviour, lifetime bugs, and safe C++ · 14 Capstone: a high-performance order-matching engine (or a ray tracer)

## 3. Rust from Zero to Expert (Rust 2024 edition)

0 Toolchain, Cargo, how Rust compiles · 1 Core syntax and types · 2 Ownership, borrowing, lifetimes · 3 Structs, enums, pattern matching · 4 Error handling · 5 Traits and generics · 6 Collections, iterators, closures · 7 Modules, crates, workspaces, testing, docs · 8 Smart pointers and interior mutability · 9 Concurrency: threads, `Send`/`Sync`, channels, atomics · 10 Async Rust: futures, executors, Tokio · 11 Unsafe Rust, FFI with C, the memory model · 12 Macros: declarative and procedural · 13 Performance and systems programming · 14 Capstone: a networked, persistent key-value store (or a Git implementation)

## 4. Java from Zero to Expert (Java 21 LTS, with 25 marked)

0 JVM, bytecode, class loading, the JDK · 1 Core language · 2 Classes, objects, encapsulation · 3 Inheritance, interfaces, records, sealed types, pattern matching · 4 Exceptions · 5 Generics in depth (erasure, wildcards) · 6 Collections framework · 7 Lambdas, streams, functional style · 8 I/O, NIO, files, serialization · 9 Build and test: Maven/Gradle, JUnit 5, Mockito · 10 Concurrency: threads, the Java Memory Model, `java.util.concurrent`, virtual threads · 11 JVM internals: memory, GC, JIT, profiling (JFR) · 12 Networking, HTTP, JDBC, databases · 13 Building services (plain Java, then Spring Boot), design patterns · 14 Capstone: a production REST service with persistence, caching, observability

## 5. Linux Kernel & OS Development from Zero

Part I — Operating-system concepts and a hobby kernel (x86-64, verified in QEMU): 0 How computers boot (firmware, BIOS/UEFI, bootloaders, GRUB/Multiboot2) · 1 x86-64 architecture for OS developers (modes, registers, privilege, assembly with NASM) · 2 A freestanding kernel in C (no libc, linker scripts, VGA/serial output) · 3 Interrupts and exceptions (IDT, PIC/APIC, timers, keyboard) · 4 Memory management (physical frame allocator, paging, virtual memory, kernel heap) · 5 Processes and scheduling (context switching, user mode, system calls) · 6 Filesystems and drivers (block devices, a simple FS, ELF loading) · 7 Concurrency in kernels (spinlocks, interrupts vs locks, SMP basics)
Part II — The Linux kernel: 8 Linux architecture and the source tree; building and booting your own kernel in QEMU · 9 Kernel modules and the kernel API · 10 Character drivers, `ioctl`, `/proc` and `sysfs` · 11 Linux memory management, processes, and the scheduler (CFS/EEVDF) · 12 Synchronisation, RCU, and debugging (printk, ftrace, kgdb, KASAN) · 13 Networking stack and eBPF · 14 Capstone: boot your hobby OS to a userland shell, and contribute-style work on a Linux driver

## 6. Swift from Zero to Expert (Swift 6)

0 Toolchain and how Swift compiles · 1 Core language · 2 Optionals and error handling · 3 Structs, classes, enums, value vs reference semantics · 4 Protocols and protocol-oriented programming · 5 Generics, opaque and existential types · 6 Collections and functional style · 7 Memory management: ARC, retain cycles, ownership (`borrowing`/`consuming`) · 8 Concurrency: async/await, tasks, actors, `Sendable`, strict concurrency · 9 Packages, testing (Swift Testing), tooling · 10 Macros and result builders · 11 SwiftUI and app architecture · 12 Server-side Swift and C/C++ interop · 13 Performance · 14 Capstone: a full app with a Swift backend

## 7. Mathematics from Zero to Advanced — for AI and Deep Learning

Purpose: every piece of mathematics that AI and deep learning use, from school level to graduate level, in the order it is needed. Every formula is computed in Python (NumPy/SciPy) with real output; every chapter ends with a project that is a real piece of ML.

**Part I — Foundations (school level).** 0 Numbers, arithmetic, fractions, powers, roots, scientific notation; how computers store numbers · 1 Algebra: expressions, equations, inequalities, functions and graphs · 2 Exponentials, logarithms, and why ML uses log everywhere · 3 Trigonometry, angles, the unit circle, waves · 4 Sets, logic, proofs, summation and product notation, sequences and series · 5 Counting: permutations, combinations, the binomial theorem

**Part II — Linear algebra.** 6 Vectors: geometry, dot product, norms, cosine similarity · 7 Matrices: multiplication as transformation, systems of equations, elimination · 8 Vector spaces, basis, rank, the four subspaces · 9 Orthogonality, projections, least squares, QR · 10 Determinants, eigenvalues and eigenvectors, diagonalisation · 11 SVD, PCA, low-rank approximation, pseudo-inverse · 12 Positive-definite matrices, quadratic forms, matrix norms, conditioning · 13 Tensors, broadcasting, Einstein summation

**Part III — Calculus.** 14 Limits and continuity · 15 Derivatives and the rules of differentiation · 16 Integrals and the fundamental theorem · 17 Taylor series and approximation · 18 Multivariable calculus: partial derivatives, gradient, directional derivative · 19 Jacobians, Hessians, the multivariable chain rule · 20 Matrix calculus: derivatives with respect to vectors and matrices · 21 Automatic differentiation and backpropagation, built from scratch

**Part IV — Probability and statistics.** 22 Probability: events, conditional probability, Bayes' theorem · 23 Random variables, expectation, variance, covariance · 24 The common distributions (Bernoulli to Gaussian to Dirichlet) · 25 Multivariate Gaussians, joint/marginal/conditional · 26 Law of large numbers, central limit theorem, concentration inequalities · 27 Estimation: maximum likelihood, MAP, bias and variance · 28 Hypothesis tests, confidence intervals, bootstrap, A/B testing · 29 Bayesian inference, conjugate priors, Monte Carlo and MCMC · 30 Markov chains and stochastic processes

**Part V — Optimisation.** 31 Convex sets and functions · 32 Gradient descent, convergence, learning rates · 33 SGD, momentum, RMSProp, Adam, schedules · 34 Second-order methods: Newton, quasi-Newton, natural gradient · 35 Constrained optimisation: Lagrange multipliers, KKT, duality · 36 Loss landscapes of neural networks: saddle points, sharpness, initialisation

**Part VI — Information theory and numerical computing.** 37 Entropy, cross-entropy, KL divergence, mutual information · 38 Floating point, numerical stability, log-sum-exp, conditioning · 39 Numerical linear algebra in practice

**Part VII — Advanced topics used in modern AI.** 40 Graphs and spectral graph theory · 41 Fourier analysis and convolution · 42 Kernels, RKHS, Gaussian processes · 43 Variational inference and the ELBO · 44 Differential equations, SDEs, and the mathematics of diffusion models · 45 Statistical learning theory: generalisation, VC dimension, PAC, double descent · 46 Measure-theoretic probability in brief, optimal transport, information geometry · 47 Capstone: derive and implement a neural network, a Gaussian process, and a small diffusion model from the mathematics alone

## 8. AI and Deep Learning from Zero to Research and Production

Purpose: one book that takes a school student to the level of a strong AI researcher and applied AI engineer. It is deliberately large and will be delivered in **volumes**. Every model is first built from scratch (NumPy), then with a framework (PyTorch); every chapter has a runnable project. Code that can run on a CPU here is run and its output is real; code that needs a GPU, a paid API, or a large download is clearly marked "not run here".

**Volume 1 — Foundations of machine learning.** 0 What AI is; history; the map of the field · 1 Python, NumPy, pandas, plotting for ML · 2 The ML workflow: data, splits, metrics, leakage · 3 Linear and logistic regression from scratch · 4 Regularisation, bias–variance, cross-validation · 5 Trees, random forests, gradient boosting · 6 SVMs, kNN, naive Bayes · 7 Clustering, dimensionality reduction, anomaly detection · 8 Feature engineering, imbalanced data, tabular practice · 9 Project: an end-to-end tabular ML system

**Volume 2 — Deep learning core.** 10 The neuron, MLPs, backpropagation from scratch · 11 An autograd engine and a tiny framework · 12 PyTorch in depth · 13 Training well: initialisation, normalisation, optimisers, schedules, regularisation · 14 Debugging and tuning neural networks · 15 CNNs · 16 RNNs, LSTMs, sequence models · 17 Attention and the Transformer, built from scratch · 18 Efficient training: mixed precision, distributed training, compilation · 19 Project: train a small GPT

**Volume 3 — Language models (LLMs).** 20 Tokenisation and embeddings · 21 Language modelling, pre-training objectives, BERT/GPT/T5 families · 22 Modern LLM architecture: RoPE, RMSNorm, KV cache, flash attention, MoE · 23 Decoding and sampling · 24 Fine-tuning: full, LoRA/QLoRA, adapters, instruction tuning · 25 Alignment training: RLHF, DPO, constitutional approaches · 26 Prompt engineering and in-context learning · 27 Structured output, tool calling, function schemas · 28 Reasoning models and test-time compute · 29 Evaluation of LLMs · 30 Inference and serving: quantisation, distillation, batching, vLLM/llama.cpp, cost · 31 Project: fine-tune and serve a domain model

**Volume 4 — Vision, speech, multimodal, generative models.** 32 Vision: classification, detection, segmentation · 33 Vision Transformers, self-supervised learning (SimCLR, DINO, MAE) · 34 CLIP and vision–language models · 35 Autoencoders, VAEs, GANs, normalising flows · 36 Diffusion and flow matching; latent diffusion; guidance · 37 Speech: ASR, TTS, audio models · 38 Video and 3D (NeRF, Gaussian splatting) · 39 Multimodal LLMs · 40 Project: a multimodal search and captioning system

**Volume 5 — Applied AI engineering.** 41 Embeddings and vector search (HNSW, IVF, PQ) · 42 RAG: chunking, retrieval, reranking, hybrid search, GraphRAG, evaluation · 43 Agents: the loop, planning, memory, tools, MCP, multi-agent systems · 44 Agent reliability: evals, tracing, cost and latency control · 45 Guardrails: input/output validation, prompt-injection defence, PII, policy · 46 AI security and red teaming · 47 Explainability and interpretability: SHAP, LIME, saliency, probing, mechanistic interpretability, sparse autoencoders · 48 Fairness, privacy (differential privacy, federated learning), governance and regulation · 49 MLOps/LLMOps: data and model versioning, CI/CD, monitoring, drift, feedback loops · 50 System design for AI products; case studies (search, support bot, coding assistant, document processing, recommendation) · 51 Recommender systems, time series, graph neural networks · 52 Reinforcement learning: bandits, Q-learning, policy gradients, PPO, model-based RL

**Volume 6 — Research level.** 53 Theory of deep learning: generalisation, NTK, implicit bias, scaling laws · 54 Reading, reproducing, and writing papers; experiment design · 55 Open problems and frontiers (continual learning, world models, AI for science, safety and alignment) · 56 Capstone: a production-grade agentic RAG system with guardrails, evals, and observability; and a research reproduction

## 9. Data Structures and Algorithms: Patterns for Hard Problems

Purpose: cover practically every data structure and algorithm pattern used in interviews (including FAANG-level) and competitive programming, and — more important — train the skill of *recognising* which pattern a new problem needs. Each pattern has: the idea, the invariant that makes it correct, a template, complexity, "how to recognise it" signals, common traps, and worked problems from easy to very hard, including problems that combine patterns. Code in Python (primary) with C++/Java notes; every solution is run against tests and a brute-force checker.

**Part I — Foundations.** 0 How to solve problems: a method; reading constraints to guess complexity · 1 Complexity analysis, amortisation, recurrences, Master theorem · 2 Arrays and strings: prefix sums, difference arrays, in-place tricks · 3 Hashing: maps, sets, counting, rolling hash · 4 Two pointers · 5 Sliding window (fixed, variable, monotonic) · 6 Binary search: on index, on answer, on real numbers · 7 Sorting: comparison sorts, counting/radix, custom orders, sweeps · 8 Linked lists: fast/slow pointers, reversal, merge · 9 Stacks and queues: monotonic stack, monotonic deque, expression parsing · 10 Recursion and backtracking: subsets, permutations, pruning, constraint search · 11 Bit manipulation and bitmasks · 12 Math and number theory: gcd, modular arithmetic, primes and sieves, combinatorics, fast power, inclusion–exclusion

**Part II — Trees and heaps.** 13 Binary trees: traversals, recursion patterns, LCA · 14 Binary search trees, balanced trees (AVL, red-black, treap), order statistics · 15 Heaps: top-k, two heaps, k-way merge, lazy deletion · 16 Tries, binary tries, Aho–Corasick · 17 Union–find (DSU): by rank, path compression, rollback, weighted · 18 Segment trees: point/range, lazy propagation, persistent, beats · 19 Fenwick trees (BIT), sparse tables, sqrt decomposition, Mo's algorithm

**Part III — Graphs.** 20 Representation, BFS, DFS, components, grids · 21 Topological sort, DAG DP, cycle detection · 22 Shortest paths: Dijkstra, Bellman–Ford, SPFA, Floyd–Warshall, 0-1 BFS, A* · 23 Minimum spanning trees: Kruskal, Prim · 24 Strongly connected components, bridges, articulation points, 2-SAT · 25 Trees as graphs: rerooting, binary lifting, Euler tour, heavy–light and centroid decomposition · 26 Network flow, bipartite matching, min-cut · 27 Eulerian paths, Hamiltonian paths with bitmask DP, graph colouring

**Part IV — Dynamic programming.** 28 DP thinking: states, transitions, order; top-down vs bottom-up · 29 1-D and 2-D DP: paths, LIS, LCS, edit distance · 30 Knapsack family · 31 Interval DP · 32 DP on trees · 33 Bitmask DP and DP over subsets (SOS) · 34 Digit DP · 35 Probability and expectation DP; game theory (Sprague–Grundy, minimax) · 36 DP optimisations: monotonic queue, convex hull trick, divide and conquer, Knuth, matrix exponentiation

**Part V — Greedy, strings, geometry, advanced.** 37 Greedy: exchange arguments, intervals, scheduling, Huffman · 38 Intervals and sweep line · 39 String algorithms: KMP, Z, Manacher, suffix array, suffix automaton · 40 Computational geometry: orientation, convex hull, line intersection, closest pair · 41 Divide and conquer; meet in the middle · 42 Randomised algorithms: reservoir sampling, quickselect, hashing tricks, Bloom filters · 43 Design problems: LRU/LFU, iterators, rate limiters, consistent hashing · 44 Concurrency interview problems

**Part VI — Mastery.** 45 Pattern recognition: a decision guide from problem statement and constraints to technique · 46 Combined-pattern problems (binary search + greedy, DP + segment tree, graph + DSU + sorting, …) · 47 100 hard problems with full reasoning · 48 Interview craft: communication, testing, complexity, follow-ups; system-design-flavoured DSA · 49 Mock interviews and a study plan

## 10. Python from Zero to Advanced

0 How Python runs: interpreter, bytecode, CPython · 1 Core language · 2 Data structures: list, tuple, dict, set, and their internals · 3 Functions, closures, decorators · 4 Classes, data model (dunder methods), dataclasses · 5 Iterators, generators, comprehensions · 6 Errors and context managers · 7 Modules, packages, virtual environments, packaging (uv/pip, pyproject) · 8 Type hints and static checking · 9 Files, text, bytes, serialization · 10 Testing (pytest), debugging, profiling · 11 Concurrency: threads and the GIL (and free-threaded builds), multiprocessing, asyncio · 12 Standard library tour · 13 Performance: internals, NumPy, C extensions, Cython · 14 Metaprogramming: descriptors, metaclasses, AST · 15 Databases and web services (FastAPI) · 16 Capstone

## 11. HTML, CSS, and JavaScript from Zero to Advanced

0 How the web works: HTTP, browsers, rendering pipeline · 1 HTML: semantics, forms, accessibility · 2 CSS: cascade, box model, selectors · 3 Layout: flexbox, grid, responsive design · 4 Modern CSS: custom properties, container queries, animation · 5 JavaScript core · 6 Functions, closures, `this`, prototypes, classes · 7 The event loop, promises, async/await · 8 The DOM, events, browser APIs · 9 Modules, tooling (Vite), npm · 10 TypeScript · 11 Networking: fetch, CORS, WebSockets · 12 Storage, service workers, PWAs · 13 Performance and Core Web Vitals · 14 Security: XSS, CSRF, CSP · 15 Testing · 16 Node.js basics · 17 Capstone: a framework-free single-page app

## 12. React and Next.js from Zero to Advanced

0 Why React; JSX; rendering model · 1 Components, props, state · 2 Hooks in depth · 3 Effects and data fetching · 4 Forms and controlled inputs · 5 Context, reducers, state management (Zustand/Redux Toolkit) · 6 Performance: reconciliation, memoisation, the React Compiler · 7 Routing · 8 Server state (TanStack Query) · 9 TypeScript with React · 10 Styling (CSS modules, Tailwind) · 11 Testing (Vitest, Testing Library, Playwright) · 12 Next.js App Router: server components, server actions, caching, streaming · 13 Authentication, databases, deployment · 14 Accessibility and internationalisation · 15 Capstone: a full-stack Next.js application

## 13. Dart from Zero to Advanced

0 Toolchain, how Dart compiles (JIT/AOT) · 1 Core language · 2 Null safety · 3 Collections · 4 Functions and closures · 5 Classes, mixins, extensions · 6 Generics · 7 Records, patterns, sealed classes · 8 Errors · 9 Async: futures, streams, isolates · 10 Packages, testing, tooling · 11 FFI and performance · 12 Capstone

## 14. Flutter from Zero to Advanced

0 Architecture: widgets, elements, render objects · 1 Layout · 2 Stateless and stateful widgets · 3 Navigation and routing · 4 State management (Provider, Riverpod, Bloc) · 5 Forms and input · 6 Networking and JSON · 7 Persistence · 8 Animations · 9 Custom painting and rendering · 10 Platform channels and plugins · 11 Testing · 12 Performance · 13 Adaptive and responsive UI, accessibility · 14 Release and CI/CD · 15 Capstone app

## 15. macOS and iOS App Development with Swift

(Builds on book 6. Code marked "not compiled" unless a toolchain becomes available.) 0 Xcode and project anatomy · 1 SwiftUI fundamentals · 2 State and data flow (Observation) · 3 Navigation · 4 Lists, forms, layout · 5 Networking and concurrency · 6 Persistence: SwiftData, Core Data, files, Keychain · 7 UIKit and AppKit interop · 8 macOS specifics: windows, menus, sandboxing · 9 Animations and graphics · 10 System frameworks: notifications, widgets, App Intents · 11 Testing and debugging (Instruments) · 12 Architecture · 13 Accessibility and localisation · 14 Distribution: signing, TestFlight, App Store · 15 Capstone apps (one iOS, one macOS)

## 16. Android Development with Java

(Builds on book 4. Code marked "not built" where the Android SDK is unavailable.) 0 Android platform and Gradle build · 1 Activities and lifecycle · 2 Layouts and views · 3 Fragments and navigation · 4 RecyclerView and adapters · 5 Intents, permissions · 6 ViewModel, LiveData, architecture · 7 Background work: threads, WorkManager, services · 8 Networking (Retrofit/OkHttp) · 9 Persistence: Room, DataStore · 10 Dependency injection (Hilt) · 11 Notifications, sensors, media, location · 12 Testing · 13 Performance and security · 14 Release · 15 Capstone app. (A short bridge chapter on Kotlin and Jetpack Compose, since new Android code is mostly Kotlin.)

## 17. Digital Design and Computer Architecture

An original book that follows the topic order of Prof. Onur Mutlu's ETH Zürich "Digital Design and Computer Architecture" lectures (publicly available on YouTube), so that it can be read alongside them; the text, examples, and exercises are written fresh, and each chapter names the lectures it accompanies. 0 Why architecture matters; the transformation hierarchy · 1 Binary, Boolean algebra, combinational logic · 2 Sequential logic, FSMs · 3 Hardware description (Verilog), timing and verification · 4 Von Neumann model, ISA, assembly (MIPS/RISC-V/LC-3) · 5 Single-cycle and multi-cycle microarchitecture · 6 Pipelining and hazards · 7 Precise exceptions, out-of-order execution · 8 Branch prediction · 9 Superscalar, SIMD, GPUs · 10 Memory hierarchy and caches · 11 Virtual memory · 12 Memory technology: DRAM, RowHammer, emerging memories · 13 Storage and I/O · 14 Multiprocessors, cache coherence · 15 Projects: a CPU in Verilog, simulated

---

# Recommended reading order

One path that builds each skill on the previous one (✔ = already written):

1. **Python** ✔ — the gentlest first language; needed for DSA, Mathematics, and AI.
2. **HTML, CSS, JavaScript** ✔
3. **React and Next.js** ✔ — if you want to build for the web early (can be moved later).
6. **Data Structures and Algorithms** ✔ — after one language is solid; keep practising it in parallel with everything below.
7. **Go** ✔ — a simple, productive backend language.
8. **Java** ✔ — the JVM and enterprise backends
10. **C++** ✔
11. **Rust** ✔ — systems languages in depth.
13. **Swift**
15. **Dart**
9. **Android with Java**.
14. **macOS and iOS development**.
16. **Flutter**.
17. **Mathematics for AI**
18. **AI and Deep Learning** — needs Python.
4. **C** ✔ — how the machine really works.
5. **Digital Design and Computer Architecture** ✔ — read alongside or just after C.
12. **Linux Kernel and OS Development** ✔ — needs C and Computer Architecture.

Shorter tracks: **Interview/FAANG:** Python → DSA → (C or Java) → Go system-design chapters. **AI engineer:** Python → Mathematics → AI and Deep Learning → DSA. **Systems engineer:** C → Computer Architecture → C++ → Rust → Linux Kernel. **Mobile:** Dart → Flutter, or Swift → iOS, or Java → Android. **Web:** HTML/CSS/JS → React/Next.js → Go or Java for the backend.

**Writing order (what I will write next):** Go ✔ → C ✔ → C++ ✔ → Rust ✔ → Java (finishing now) → **Data Structures and Algorithms** (moved up: you marked it most important) → Python → Linux Kernel & OS Development → Digital Design and Computer Architecture → HTML/CSS/JavaScript → React and Next.js → Mathematics → AI and Deep Learning → Swift → macOS/iOS → Dart → Flutter → Android. Tell me if you want a different order.
